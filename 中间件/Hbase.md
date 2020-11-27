[TOC]

参考：https://www.jianshu.com/p/b23800d9b227

### HBase特征

- 强读写一致，但是不是“最终一致性”的数据存储，这使得它非常适合高速的计算聚合
- 自动分片，通过Region分散在集群中，当行数增长的时候，Region也会自动的切分和再分配
- 自动的故障转移
- Hadoop/HDFS集成，和HDFS开箱即用，不用太麻烦的衔接
- 丰富的“简洁，高效”API，Thrift/REST API，Java API
- 块缓存，布隆过滤器，可以高效的列查询优化
- 操作管理，Hbase提供了内置的web界面来操作，还可以监控JMX指标

### 应用场景

首先数据库量要足够多，如果有十亿及百亿行数据，那么Hbase是一个很好的选项，如果只有几百万行甚至不到的数据量，RDBMS是一个很好的选择。因为数据量小的话，真正能工作的机器量少，剩余的机器都处于空闲的状态

其次，如果你不需要辅助索引，静态类型的列，事务等特性，一个已经用RDBMS的系统想要切换到Hbase，则需要重新设计系统。

最后，保证硬件资源足够，每个HDFS集群在少于5个节点的时候，都不能表现的很好。因为HDFS默认的复制数量是3，再加上一个NameNode。

### 内部应用

- 存储业务数据 : 车辆GPS信息，司机点位信息，用户操作信息，设备访问信息。。。
- 存储日志数据 : 架构监控数据（登录日志，中间件访问日志，推送日志，短信邮件发送记录。。。），业务操作日志信息
- 存储业务附件：UDFS系统存储图像，视频，文档等附件信息

不过在公司使用的时候，一般不使用原生的Hbase API，使用原生的API会导致访问不可监控，影响系统稳定性，以致于版本升级的不可控。

## 架构体系

![image-20201023175525350](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201023175525350.png)

Zookeeper，作为分布式的协调。RegionServer也会把自己的信息写到ZooKeeper中。

HDFS是Hbase运行的底层文件系统

RegionServer，理解为数据节点，存储数据的。

Master：RegionServer要实时的向Master报告信息。Master知道全局的RegionServer运行情况，可以控制RegionServer的故障转移和Region的切分。

![image-20201023175851026](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201023175851026.png)

HMaster是Master Server的实现，负责监控集群中的RegionServer实例，同时是所有metadata改变的接口，在集群中，通常运行在NameNode上面

- HMasterInterface暴露的接口，Table(createTable, modifyTable, removeTable, enable, disable),ColumnFamily (addColumn, modifyColumn, removeColumn),Region (move, assign, unassign)
- Master运行的后台线程：LoadBalancer线程，控制region来平衡集群的负载。CatalogJanitor线程，周期性的检查hbase:meta表。

HRegionServer是RegionServer的实现，服务和管理Regions，集群中RegionServer运行在DataNode

- HRegionRegionInterface暴露接口：Data (get, put, delete, next, etc.)，Region (splitRegion, compactRegion, etc.)
- RegionServer后台线程：CompactSplitThread，MajorCompactionChecker，MemStoreFlusher，LogRoller

Regions，代表table，Region有多个Store(列簇)，Store有一个Memstore和多个StoreFiles(HFiles)，StoreFiles的底层是Block。





#### 存储设计

在Hbase中，表被分割成多个更小的块然后分散的存储在不同的服务器上，这些小块叫做Regions，存放Regions的地方叫做RegionServer。Master进程负责处理不同的RegionServer之间的Region的分发。在Hbase实现中HRegionServer和HRegion类代表RegionServer和Region。HRegionServer除了包含一些HRegions之外，还处理两种类型的文件用于数据存储

- HLog， 预写日志文件，也叫做WAL(write-ahead log)
- HFile 真实的数据存储文件

##### HLog

- MasterProcWAL：HMaster记录管理操作，比如解决冲突的服务器，表创建和其它DDLs等操作到它的WAL文件中，这个WALs存储在MasterProcWALs目录下，它不像RegionServer的WALs，HMaster的WAL也支持弹性操作，就是如果Master服务器挂了，其它的Master接管的时候继续操作这个文件。
- WAL记录所有的Hbase数据改变，如果一个RegionServer在MemStore进行FLush的时候挂掉了，WAL可以保证数据的改变被应用到。如果写WAL失败了，那么修改数据的完整操作就是失败的。
  - 通常情况，每个RegionServer只有一个WAL实例。在2.0之前，WAL的实现叫做HLog
  - WAL位于*/hbase/WALs/*目录下
  - MultiWAL: 如果每个RegionServer只有一个WAL，由于HDFS必须是连续的，导致必须写WAL连续的，然后出现性能问题。MultiWAL可以让RegionServer同时写多个WAL并行的，通过HDFS底层的多管道，最终提升总的吞吐量，但是不会提升单个Region的吞吐量。



##### HFile

HFile是Hbase在HDFS中存储数据的格式，它包含多层的索引，这样在Hbase检索数据的时候就不用完全的加载整个文件。索引的大小(keys的大小，数据量的大小)影响block的大小，在大数据集的情况下，block的大小设置为每个RegionServer 1GB也是常见的。

> 探讨数据库的数据存储方式，其实就是探讨数据如何在磁盘上进行有效的组织。因为我们通常以如何高效读取和消费数据为目的，而不是数据存储本身。

###### Hfile生成方式

起初，HFile中并没有任何Block，数据还存在于MemStore中。

Flush发生时，创建HFile Writer，第一个空的Data Block出现，初始化后的Data Block中为Header部分预留了空间，Header部分用来存放一个Data Block的元数据信息。

而后，位于MemStore中的KeyValues被一个个append到位于内存中的第一个Data Block中：

**注**：如果配置了Data Block Encoding，则会在Append KeyValue的时候进行同步编码，编码后的数据不再是单纯的KeyValue模式。Data Block Encoding是HBase为了降低KeyValue结构性膨胀而提供的内部编码机制。





















