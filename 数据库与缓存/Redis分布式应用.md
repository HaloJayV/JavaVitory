# 分布式解决方案redis cluster

### 使用原因

1.主从复制不能实现高可用 

2.随着公司发展，用户数量增多，并发越来越多，业务需要更高的QPS，而主从复制中单机的QPS可能无法满足业务需求 

3.数据量的考虑，现有服务器内存不能满足业务数据的需要时，单纯向服务器添加内存不能达到要求，此时需要考虑分布式需求，把数据分布到不同服务器上 

4.网络流量需求：业务的流量已经超过服务器的网卡的上限值，可以考虑使用分布式来进行分流 

5.离线计算，需要中间环节缓冲等别的需求

### 数据分布

#### 顺序分布

比如：1到100个数字，要保存在3个节点上，按照顺序分区，把数据平均分配三个节点上 1号到33号数据保存到节点1上，34号到66号数据保存到节点2上，67号到100号数据保存到节点3上。一般常用在关系型数据库

#### 哈希分布

##### 1、节点取余分区：

比如有100个数据，对每个数据进行hash运算之后，与节点数进行取余运算，根据余数不同保存在不同的节点上

节点取余分区方式有一个问题：即当增加或减少节点时，原来节点中的80%的数据会进行迁移操作，对所有数据重新进行分布

节点取余分区方式建议使用多倍扩容的方式，例如以前用3个节点保存数据，扩容为比以前多一倍的节点即6个节点来保存数据，这样只需要适移50%的数据。数据迁移之后，第一次无法从缓存中读取数据，必须先从数据库中读取数据，然后回写到缓存中，然后才能从缓存中读取迁移之后的数据

![image-20201103122709166](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201103122709166.png)

##### 2、一致性哈希分区

原理：将所有的数据当做一个token环，token环中的数据范围是0到2的32次方。然后为每一个数据节点分配一个token范围值，这个节点就负责保存这个范围内的数据。一致性哈希一般用在节点比较多的时候

![image-20201103134222480](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201103134222480.png)

对每一个key进行hash运算，被哈希后的结果在哪个token的范围内，则按顺时针去找最近的节点，这个key将会被保存在这个节点上。

![image-20201103134245440](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201103134245440.png)

在上面的图中，有4个key被hash之后的值在在n1节点和n2节点之间，按照顺时针规则，这4个key都会被保存在n2节点上，
如果在n1节点和n2节点之间添加n5节点，当下次有key被hash之后的值在n1节点和n5节点之间，这些key就会被保存在n5节点上面了
在上面的例子里，添加n5节点之后，数据迁移会在n1节点和n2节点之间进行，n3节点和n4节点不受影响，数据迁移范围被缩小很多

同理，如果有1000个节点，此时添加一个节点，受影响的节点范围最多只有千分之2
一致性哈希一般用在节点比较多的时候

一致性哈希分区优点：

```
采用客户端分片方式：哈希 + 顺时针(优化取余)
节点伸缩时，只影响邻近节点，但是还是有数据迁移
```

一致性哈希分区缺点：

```
翻倍伸缩，保证最小迁移数据和负载均衡
```

![image-20201103134757904](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201103134757904.png)

##### 3、虚拟槽分区

###### 虚拟槽分区是Redis Cluster采用的分区方式

预设虚拟槽，每个槽就相当于一个数字，有一定范围。每个槽映射一个数据子集，一般比节点数大

> Redis Cluster中预设虚拟槽的范围为0到16383

![image-20201103134646713](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201103134646713.png)

步骤：

```
1.把16384槽按照节点数量进行平均分配，由节点进行管理
2.对每个key按照CRC16规则进行hash运算
3.把hash结果对16383进行取余
4.把余数发送给Redis节点
5.节点接收到数据，验证是否在自己管理的槽编号的范围
    如果在自己管理的槽编号范围内，则把数据保存到数据槽中，然后返回执行结果
    如果在自己管理的槽编号范围外，则会把数据发送给正确的节点，由正确的节点来把数据保存在对应的槽中
```

> 需要注意的是：Redis Cluster的节点之间会共享消息，每个节点都会知道是哪个节点负责哪个范围内的数据槽

虚拟槽分布方式中，由于每个节点管理一部分数据槽，数据保存到数据槽中。当节点扩容或者缩容时，对数据槽进行重新分配迁移即可，数据不会丢失。
虚拟槽分区特点：

```
使用服务端管理节点，槽，数据：例如Redis Cluster
可以对数据打散，又可以保证数据分布均匀
```

### 基本架构

#### 节点

Redis Cluster是分布式架构：即Redis Cluster中有多个节点，每个节点都负责进行数据读写操作

每个节点之间会进行通信。

#### meet操作

节点之间会相互通信，每个节点里都有对应的一定数量的槽

meet操作是节点之间完成相互通信的基础，meet操作有一定的频率和规则

![image-20201103135152945](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201103135152945.png)

#### 分配槽

把16384个槽平均分配给节点进行管理，每个节点只能对自己负责的槽进行读写操作

由于每个节点之间都彼此通信，每个节点都知道另外节点负责管理的槽范围

客户端访问任意节点时，对数据key按照CRC16规则进行hash运算，然后对运算结果对16383进行取余，如果余数在当前访问的节点管理的槽范围内，则直接返回对应的数据
如果不在当前节点负责管理的槽范围内，则会告诉客户端去哪个节点获取数据，由客户端去正确的节点获取数据

#### 复制

保证高可用，每个主节点都有一个从节点，当主节点故障，Cluster会按照规则实现主备的高可用性

对于节点来说，有一个配置项：cluster-enabled，即是否以集群模式启动

### 客户端路由

#### moved重定向

```
1.每个节点通过通信都会共享Redis Cluster中槽和集群中对应节点的关系
2.客户端向Redis Cluster的任意节点发送命令，接收命令的节点会根据CRC16规则进行hash运算与16383取余，计算自己的槽和对应节点
3.如果保存数据的槽被分配给当前节点，则去槽中执行命令，并把命令执行结果返回给客户端
4.如果保存数据的槽不在当前节点的管理范围内，则向客户端返回moved重定向异常
5.客户端接收到节点返回的结果，如果是moved异常，则从moved异常中获取目标节点的信息
6.客户端向目标节点发送命令，获取命令执行结果
```

![image-20201103200723150](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201103200723150.png)

需要注意的是：客户端不会自动找到目标节点执行命令

槽命中：直接返回

```
[root@mysql ~]# redis-cli -p 9002 cluster keyslot hello
(integer) 866
```

槽不命中：moved异常

```
[root@mysql ~]# redis-cli -p 9002 cluster keyslot php
(integer) 9244
```

```
[root@mysql ~]# redis-cli -c -p 9002
127.0.0.1:9002> cluster keyslot hello
(integer) 866
127.0.0.1:9002> set hello world
-> Redirected to slot [866] located at 192.168.81.100:9003
OK
192.168.81.100:9003> cluster keyslot python
(integer) 7252
192.168.81.100:9003> set python best
-> Redirected to slot [7252] located at 192.168.81.101:9002
OK
192.168.81.101:9002> get python
"best"
192.168.81.101:9002> get hello
-> Redirected to slot [866] located at 192.168.81.100:9003
"world"
192.168.81.100:9003> exit
[root@mysql ~]# redis-cli -p 9002
127.0.0.1:9002> cluster keyslot python
(integer) 7252
127.0.0.1:9002> set python best
OK
127.0.0.1:9002> set hello world
(error) MOVED 866 192.168.81.100:9003
127.0.0.1:9002> exit
```

#### ask重定向

![image-20201103201212975](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201103201212975.png)

在对集群进行扩容和缩容时，需要对槽及槽中数据进行迁移

当客户端向某个节点发送命令，节点向客户端返回moved异常，告诉客户端数据对应的槽的节点信息

如果此时正在进行集群扩展或者缩空操作，当客户端向正确的节点发送命令时，槽及槽中数据已经被迁移到别的节点了，就会返回ask，这就是ask重定向机制

![image-20201103201452505](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201103201452505.png)

步骤：

```
1.客户端向目标节点发送命令，目标节点中的槽已经迁移支别的节点上了，此时目标节点会返回ask转向给客户端
2.客户端向新的节点发送Asking命令给新的节点，然后再次向新节点发送命令
3.新节点执行命令，把命令执行结果返回给客户端
```

moved异常与ask异常的相同点和不同点

```
两者都是客户端重定向
moved异常：槽已经确定迁移，即槽已经不在当前节点
ask异常：槽还在迁移中
```

#### smart智能客户端

使用智能客户端的首要目标：追求性能

从集群中选一个可运行节点，使用Cluster slots初始化槽和节点映射

将Cluster slots的结果映射在本地，为每个节点创建JedisPool，相当于为每个redis节点都设置一个JedisPool，然后就可以进行数据读写操作

读写数据时的注意事项：

```
每个JedisPool中缓存了slot和节点node的关系
key和slot的关系：对key进行CRC16规则进行hash后与16383取余得到的结果就是槽
JedisCluster启动时，已经知道key,slot和node之间的关系，可以找到目标节点
JedisCluster对目标节点发送命令，目标节点直接响应给JedisCluster
如果JedisCluster与目标节点连接出错，则JedisCluster会知道连接的节点是一个错误的节点
此时JedisCluster会随机节点发送命令，随机节点返回moved异常给JedisCluster
JedisCluster会重新初始化slot与node节点的缓存关系，然后向新的目标节点发送命令，目标命令执行命令并向JedisCluster响应
如果命令发送次数超过5次，则抛出异常"Too many cluster redirection!"
```

![image-20201104132845788](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201104132845788.png)

#### 多节点命令实现

Redis Cluster不支持使用scan命令扫描所有节点，多节点命令就是在在所有节点上都执行一条命令，批量操作优化

##### 1、串行mget：定义for循环，遍历所有的key，分别去所有的Redis节点中获取值并进行汇总，简单，但是效率不高，需要n次网络时间

![image-20201104133000732](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201104133000732.png)

##### 2、串行IO：

对串行mget进行优化，在客户端本地做内聚，对每个key进行CRC16hash，然后与16383取余，就可以知道哪个key对应的是哪个槽

本地已经缓存了槽与节点的对应关系，然后对key按节点进行分组，成立子集，然后使用pipeline把命令发送到对应的node，需要nodes次网络时间，大大减少了网络时间开销

![image-20201104133028690](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201104133028690.png)

##### 3、并行IO

并行IO是对串行IO的一个优化，把key分组之后，根据节点数量启动对应的线程数，根据多线程模式并行向node节点请求数据，只需要1次网络时间

![image-20201104133050177](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201104133050177.png)

##### 4、hash_tag

将key进行hash_tag的包装，然后把tag用大括号括起来，保证所有的key只向一个node请求数据，这样执行类似mget命令只需要去一个节点获取数据即可，效率更高

![image-20201104133150340](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201104133150340.png)

![image-20201104133200343](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201104133200343.png)

### 3.7 故障发现

Redis Cluster通过ping/pong消息实现故障发现：不需要sentinel

ping/pong不仅能传递节点与槽的对应消息，也能传递其他状态，比如：节点主从状态，节点故障等

故障发现就是通过这种模式来实现，分为主观下线和客观下线

#### 3.7.1 主观下线

某个节点认为另一个节点不可用，'偏见'，只代表一个节点对另一个节点的判断，不代表所有节点的认知

主观下线流程：

```
1.节点1定期发送ping消息给节点2
2.如果发送成功，代表节点2正常运行，节点2会响应PONG消息给节点1，节点1更新与节点2的最后通信时间
3.如果发送失败，则节点1与节点2之间的通信异常判断连接，在下一个定时任务周期时，仍然会与节点2发送ping消息
4.如果节点1发现与节点2最后通信时间超过node-timeout，则把节点2标识为pfail状态
        
```

![img](https://img2018.cnblogs.com/blog/1133627/201810/1133627-20181027173946714-700297693.png)

#### 3.7.2 客观下线

当半数以上持有槽的主节点都标记某节点主观下线时，可以保证判断的公平性

集群模式下，只有主节点(master)才有读写权限和集群槽的维护权限，从节点(slave)只有复制的权限

客观下线流程：

```
1.某个节点接收到其他节点发送的ping消息，如果接收到的ping消息中包含了其他pfail节点，这个节点会将主观下线的消息内容添加到自身的故障列表中，故障列表中包含了当前节点接收到的每一个节点对其他节点的状态信息
2.当前节点把主观下线的消息内容添加到自身的故障列表之后，会尝试对故障节点进行客观下线操作
```

> 故障列表的周期为：集群的node-timeout * 2，保证以前的故障消息不会对周期内的故障消息造成影响，保证客观下线的公平性和有效性

![img](https://img2018.cnblogs.com/blog/1133627/201810/1133627-20181027173956048-817167341.png)

![img](https://img2018.cnblogs.com/blog/1133627/201810/1133627-20181027174002279-818812119.png)

### 3.8 故障恢复

#### 3.8.1 资格检查

```
对从节点的资格进行检查，只有难过检查的从节点才可以开始进行故障恢复
每个从节点检查与故障主节点的断线时间
超过cluster-node-timeout * cluster-slave-validity-factor数字，则取消资格
cluster-node-timeout默认为15秒，cluster-slave-validity-factor默认值为10
如果这两个参数都使用默认值，则每个节点都检查与故障主节点的断线时间，如果超过150秒，则这个节点就没有成为替换主节点的可能性
```

#### 3.9.2 准备选举时间

```
使偏移量最大的从节点具备优先级成为主节点的条件
```

![img](https://img2018.cnblogs.com/blog/1133627/201810/1133627-20181027174013055-1564035725.png)

#### 3.8.3 选举投票

```
对选举出来的多个从节点进行投票，选出新的主节点
```

![img](https://img2018.cnblogs.com/blog/1133627/201810/1133627-20181027174022694-1531606344.png)

#### 3.8.4 替换主节点

```
当前从节点取消复制变成离节点(slaveof no one)
执行cluster del slot撤销故障主节点负责的槽，并执行cluster add slot把这些槽分配给自己
向集群广播自己的pong消息，表明已经替换了故障从节点
```

#### 3.8.5 故障转移演练

```
对某一个主节点执行kill -9 {pid}来模拟宕机的情况
```

### 3.9 Redis Cluster的缺点

```
当节点数量很多时，性能不会很高
解决方式：使用智能客户端。智能客户端知道由哪个节点负责管理哪个槽，而且当节点与槽的映射关系发生改变时，客户端也会知道这个改变，这是一种非常高效的方式
```

# 使用

## 4.搭建Redis Cluster

搭建Redis Cluster有两种安装方式

- 1.[原生命令安装](https://www.cnblogs.com/renpingsheng/p/9813959.html)

- 2.[官方工具安装](https://www.cnblogs.com/renpingsheng/p/9833740.html)

- 3.集群配置：https://www.jianshu.com/p/813a79ddf932

  ## 5.开发运维常见的问题

  ### 5.1 集群完整性

cluster-require-full-coverage默认为yes，即是否集群中的所有节点都是在线状态且16384个槽都处于服务状态时，集群才会提供服务

集群中16384个槽全部处于服务状态，保证集群完整性

当某个节点故障或者正在故障转移时获取数据会提示：(error)CLUSTERDOWN The cluster is down

> 建议把cluster-require-full-coverage设置为no

### 5.2 带宽消耗

Redis Cluster节点之间会定期交换Gossip消息，以及做一些心跳检测

官方建议Redis Cluster节点数量不要超过1000个,当集群中节点数量过多时，会产生不容忽视的带宽消耗

消息发送频率：节点发现与其他节点最后通信时间超过cluster-node-timeout /2时，会直接发送PING消息

消息数据量：slots槽数组(2kb空间)和整个集群1/10的状态数据(10个节点状态数据约为1kb)

节点部署的机器规模：集群分布的机器越多且每台机器划分的节点数越均匀，则集群内整体的可用带宽越高

带宽优化：

```
避免使用'大'集群：避免多业务使用一个集群，大业务可以多集群
cluster-node-timeout:带宽和故障转移速度的均衡
尽量均匀分配到多机器上：保证高可用和带宽
```

### 5.3 Pub/Sub广播

在任意一个cluster节点执行publish，则发布的消息会在集群中传播，集群中的其他节点都会订阅到消息，这样节点的带宽的开销会很大

publish在集群每个节点广播，加重带宽

解决办法：需要使用Pub/Sub时，为了保证高可用，可以单独开启一套Redis Sentinel

### 5.4 集群倾斜

对于分布式数据库来说，存在倾斜问题是比较常见的

集群倾斜也就是各个节点使用的内存不一致

#### 5.4.1 数据倾斜原因

1.节点和槽分配不均，如果使用redis-trib.rb工具构建集群，则出现这种情况的机会不多

```
redis-trib.rb info ip:port查看节点，槽，键值分布
redis-trib.rb rebalance ip:port进行均衡(谨慎使用)
```

2.不同槽对应键值数量差异比较大

```
CRC16算法正常情况下比较均匀
可能存在hash_tag
cluster countkeysinslot {slot}获取槽对应键值个数
```

3.包含bigkey：例如大字符串，几百万的元素的hash,set等

```
在从节点：redis-cli --bigkeys
优化：优化数据结构
```

4.内存相关配置不一致

```
hash-max-ziplist-value：满足一定条件情况下，hash可以使用ziplist
set-max-intset-entries：满足一定条件情况下，set可以使用intset
在一个集群内有若干个节点，当其中一些节点配置上面两项优化，另外一部分节点没有配置上面两项优化
当集群中保存hash或者set时，就会造成节点数据不均匀
优化：定期检查配置一致性
```

5.请求倾斜：热点key

```
重要的key或者bigkey
Redis Cluster某个节点有一个非常重要的key，就会存在热点问题
```

#### 5.4.2 集群倾斜优化：

```
避免bigkey
热键不要用hash_tag
当一致性不高时，可以用本地缓存+ MQ(消息队列)
```

### 5.5 集群读写分离

只读连接：集群模式下，从节点不接受任何读写请求

当向从节点执行读请求时，重定向到负责槽的主节点

readonly命令可以读：连接级别命令，当连接断开之后，需要再次执行readonly命令

读写分离：

```
同样的问题：复制延迟，读取过期数据，从节点故障
修改客户端：cluster slaves {nodeId}
```

### 5.6 数据迁移

官方迁移工具：redis-trib.rb和import

只能从单机迁移到集群

不支持在线迁移：source需要停写

不支持断点续传

单线程迁移：影响深度

在线迁移：

```
唯品会：redis-migrate-tool
豌豆荚：redis-port
```

### 5.7 集群VS单机

集群的限制:

```
key批量操作支持有限：例如mget,mset必须在一个slot
key事务和Lua支持有限：操作的key必须在一个节点
key是数据分区的最小粒度：不支持bigkey分区
不支持多个数据库：集群模式下只有一个db0
复制只支持一层：不支持树形复制结构
Redis Cluster满足容量和性能的扩展性，很多业务'不需要'
大多数时客户端性能会'降低'
命令无法跨节点使用：mget,keys,scan,flush,sinter等
Lua和事务无法跨节点使用
客户端维护更复杂：SDK和应用本身消耗(例如更多的连接池)
```

> 很多场景Redis Sentinel已经够用了

### 6.Redis Cluster总结：

```
1.Redis Cluster数据分区规则采用虚拟槽方式(16384个槽)，每个节点负责一部分槽和相关数据，实现数据和请求的负载均衡
2.搭建Redis Cluster划分四个步骤：准备节点，meet操作，分配槽，复制数据。
3.Redis官方推荐使用redis-trib.rb工具快速搭建Redis Cluster
4.集群伸缩通过在节点之间移动槽和相关数据实现
    扩容时根据槽迁移计划把槽从源节点迁移到新节点
    收缩时如果下线的节点有负责的槽需要迁移到其他节点，再通过cluster forget命令让集群内所有节点忘记被下线节点
5.使用smart客户端操作集群过到通信效率最大化，客户端内部负责计算维护键，槽以及节点的映射，用于快速定位到目标节点
6.集群自动故障转移过程分为故障发现和节点恢复。节点下线分为主观下线和客观下线，当超过半数节点认为故障节点为主观下线时，标记这个节点为客观下线状态。从节点负责对客观下线的主节点触发故障恢复流程，保证集群的可用性
7.开发运维常见问题包括：超大规模集群带席消耗，pub/sub广播问题，集群倾斜问题，单机和集群对比等
```

 

# Redis cluster搭建

https://www.jianshu.com/p/813a79ddf932

> redis最开始使用主从模式做集群，若master宕机需要手动配置slave转为master；后来为了高可用提出来**哨兵**模式，该模式下有一个哨兵监视master和slave，若master宕机可自动将slave转为master，但它也有一个问题，就是不能动态扩充；所以在3.x提出cluster集群模式。

**一、redis-cluster设计**
 Redis-Cluster采用无中心结构，每个节点保存数据和整个集群状态,每个节点都和其他所有节点连接。

![img](https:////upload-images.jianshu.io/upload_images/12185313-0f55e1cc574cae70.png?imageMogr2/auto-orient/strip|imageView2/2/w/275/format/webp)

 其结构特点：
 1、所有的redis节点彼此互联(PING-PONG机制),内部使用二进制协议优化传输速度和带宽。
 2、节点的fail是通过集群中超过半数的节点检测失效时才生效。
 3、客户端与redis节点直连,不需要中间proxy层.客户端不需要连接集群所有节点,连接集群中任何一个可用节点即可。
 4、redis-cluster把所有的物理节点映射到[0-16383]slot上（不一定是平均分配）,cluster 负责维护node<->slot<->value。
 5、Redis集群预分好16384个桶，当需要在 Redis 集群中放置一个 key-value 时，根据 CRC16(key) mod 16384的值，决定将一个key放到哪个桶中。

**a.redis cluster节点分配**
 现在我们是三个主节点分别是：A, B, C 三个节点，它们可以是一台机器上的三个端口，也可以是三台不同的服务器。那么，采用哈希槽 (hash slot)的方式来分配16384个slot 的话，它们三个节点分别承担的slot 区间是：

- 节点A覆盖0－5460;

- 节点B覆盖5461－10922;

- 节点C覆盖10923－16383.

  获取数据:
   如果存入一个值，按照redis cluster哈希槽的[算法](http://lib.csdn.net/base/datastructure)： CRC16('key')384 = 6782。 那么就会把这个key 的存储分配到 B 上了。同样，当我连接(A,B,C)任何一个节点想获取'key'这个key时，也会这样的算法，然后内部跳转到B节点上获取数据

  新增一个主节点:
   新增一个节点D，redis cluster的这种做法是从各个节点的前面各拿取一部分slot到D上，我会在接下来的实践中实验。大致就会变成这样：

- 节点A覆盖1365-5460

- 节点B覆盖6827-10922

- 节点C覆盖12288-16383

- 节点D覆盖0-1364,5461-6826,10923-12287

同样删除一个节点也是类似，移动完成后就可以删除这个节点了。

**b.Redis Cluster主从模式**
 redis cluster 为了保证数据的高可用性，加入了主从模式，一个主节点对应一个或多个从节点，主节点提供数据存取，从节点则是从主节点拉取数据备份，当这个主节点挂掉后，就会有这个从节点选取一个来充当主节点，从而保证集群不会挂掉

上面那个例子里, 集群有ABC三个主节点, 如果这3个节点都没有加入从节点，如果B挂掉了，我们就无法访问整个集群了。A和C的slot也无法访问。

所以我们在集群建立的时候，一定要为每个主节点都添加了从节点, 比如像这样, 集群包含主节点A、B、C, 以及从节点A1、B1、C1, 那么即使B挂掉系统也可以继续正确工作。

B1节点替代了B节点，所以Redis集群将会选择B1节点作为新的主节点，集群将会继续正确地提供服务。 当B重新开启后，它就会变成B1的从节点。

不过需要注意，如果节点B和B1同时挂了，Redis集群就无法继续正确地提供服务了。

**二、redis集群的搭建**
 集群中至少应该有奇数个节点，所以至少有三个节点，每个节点至少有一个备份节点，所以下面使用6节点（主节点、备份节点由redis-cluster集群确定）
 下载[redis](https://redis.io/download)
 **1、安装redis节点指定端口**
 解压redis压缩包，编译安装



```csharp
[root@localhost redis-3.2.0]# tar xzf redis-3.2.0.tar.gz
[root@localhost redis-3.2.0]# cd redis-3.2.0
[root@localhost redis-3.2.0]# make
[root@localhost redis01]# make install PREFIX=/usr/andy/redis-cluster
```

在redis-cluster下 修改bin文件夹为redis01,复制redis.conf配置文件
 创建目录redis-cluster并在此目录下再创建7000 7001 7002 7003 7004 7005共6个目录，在7000中创建配置文件redis.conf，内容如下：



```bash
        daemonize yes #后台启动
        port 7001 #修改端口号，从7001到7006
        cluster-enabled yes #开启cluster，去掉注释
        cluster-config-file nodes.conf #自动生成
        cluster-node-timeout 15000 #节点通信时间
        appendonly yes #持久化方式
```

同时把redis.conf复制到其它目录中

**2、安装redis-trib所需的 ruby脚本**
 注意：centos7默认的ruby版本太低(2.0)，要卸载重装(最低2.2)



```csharp
yum remove ruby
yum install ruby
yum install rubygems
```

复制redis解压文件src下的redis-trib.rb文件到redis-cluster目录并安装gem



```css
gem install redis-3.x.x.gem
```

若不想安装src目录下的gem，也可以直接`gem install redis`。

注意，gem install可能会报错
 Unable to require openssl,install OpenSSL and rebuild ruby (preferred) or use ....
 解决步骤：

1. yum install openssl-devel -y
2. 在ruby安装包/root/ruby-x.x.x/ext/openssl，执行ruby ./extconf.rb
3. 执行make,若出现make: *** No rule to make target `/include/ruby.h', needed by`ossl.o'.  Stop.;在Makefile顶部中的增加`top_srcdir = ../..`
4. 执行make install

**3、启动所有的redis节点**
 可以写一个命令脚本start-all.sh



```bash
cd 7000
redis-server redis.conf
cd ..
cd 7001
redis-server redis.conf
cd ..
cd 7002
redis-server redis.conf
cd ..
cd 7003
redis-server redis.conf
cd ..
cd 7004
redis-server redis.conf
cd ..
cd 7005
redis-server redis.conf
cd ..
```

设置权限启动



```csharp
[root@localhost redis-cluster]# chmod 777 start-all.sh 
[root@localhost redis-cluster]# ./start-all.sh 
```

查看redis进程启动状态



```cpp
[root@localhost redis-4.0.2]# ps -ef|grep cluster
root      54956      1  0 19:17 ?        00:00:00 redis-server *:7000 [cluster]
root      54961      1  0 19:17 ?        00:00:00 redis-server *:7001 [cluster]
root      54966      1  0 19:17 ?        00:00:00 redis-server *:7002 [cluster]
root      54971      1  0 19:17 ?        00:00:00 redis-server *:7003 [cluster]
root      54976      1  0 19:17 ?        00:00:00 redis-server *:7004 [cluster]
root      54981      1  0 19:17 ?        00:00:00 redis-server *:7005 [cluster]
root      55071  24089  0 19:24 pts/0    00:00:00 grep --color=auto cluster
```

可以看到redis的6个节点已经启动成功
 **注意：这里并没有创建集群**

**4、使用redis-trib.rb创建集群**
 注意：redis-trib.rb在redis/src目录下。



```undefined
./redis-trib.rb create --replicas 1 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7000
```

使用create命令 --replicas 1 参数表示为每个主节点创建一个从节点，其他参数是实例的地址集合。



```ruby
[root@localhost redis]# ./src/redis-trib.rb create --replicas 1 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7000
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
127.0.0.1:7001
127.0.0.1:7002
127.0.0.1:7003
Adding replica 127.0.0.1:7004 to 127.0.0.1:7001
Adding replica 127.0.0.1:7005 to 127.0.0.1:7002
Adding replica 127.0.0.1:7000 to 127.0.0.1:7003
M: f4ee0a501f9aaf11351787a46ffb4659d45b7bd7 127.0.0.1:7001
   slots:0-5460 (5461 slots) master
M: 671a0524a616da8b2f50f3d11a74aaf563578e41 127.0.0.1:7002
   slots:5461-10922 (5462 slots) master
M: 18948dab5b07e3726afd1b6a42d5bf6e2f411ba1 127.0.0.1:7003
   slots:10923-16383 (5461 slots) master
S: 34e322ca50a2842e9f3664442cb11c897defba06 127.0.0.1:7004
   replicates f4ee0a501f9aaf11351787a46ffb4659d45b7bd7
S: 62a00566233fbff4467c4031345b1db13cf12b46 127.0.0.1:7005
   replicates 671a0524a616da8b2f50f3d11a74aaf563578e41
S: 2cb649ad3584370c960e2036fb01db834a546114 127.0.0.1:7000
   replicates 18948dab5b07e3726afd1b6a42d5bf6e2f411ba1
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join...
>>> Performing Cluster Check (using node 127.0.0.1:7001)
M: f4ee0a501f9aaf11351787a46ffb4659d45b7bd7 127.0.0.1:7001
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: 671a0524a616da8b2f50f3d11a74aaf563578e41 127.0.0.1:7002
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: 2cb649ad3584370c960e2036fb01db834a546114 127.0.0.1:7000
   slots: (0 slots) slave
   replicates 18948dab5b07e3726afd1b6a42d5bf6e2f411ba1
S: 34e322ca50a2842e9f3664442cb11c897defba06 127.0.0.1:7004
   slots: (0 slots) slave
   replicates f4ee0a501f9aaf11351787a46ffb4659d45b7bd7
M: 18948dab5b07e3726afd1b6a42d5bf6e2f411ba1 127.0.0.1:7003
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
S: 62a00566233fbff4467c4031345b1db13cf12b46 127.0.0.1:7005
   slots: (0 slots) slave
   replicates 671a0524a616da8b2f50f3d11a74aaf563578e41
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

上面显示创建成功，有3个主节点，3个从节点，每个节点都是成功连接状态。

**三、redis集群的测试**
 测试存取值，客户端连接集群redis-cli需要带上 -c ，redis-cli -c -p 端口号



```ruby
[root@localhost redis]# ./redis-cli -c -p 7001
127.0.0.1:7001> set name andy
-> Redirected to slot [5798] located at 127.0.0.1:7002
OK
127.0.0.1:7002> get name
"andy"
127.0.0.1:7002> 
```

根据redis-cluster的key值分配，name应该分配到节点7002[5461-10922]上，上面显示redis cluster自动从7001跳转到了7002节点。

测试一下7000从节点获取name值



```ruby
[root@localhost redis]# ./redis-cli -c -p 7000
127.0.0.1:7000> get name
-> Redirected to slot [5798] located at 127.0.0.1:7002
"andy"
127.0.0.1:7002> 
```

**四、集群节点选举**
 现在模拟将7002节点挂掉，按照redis-cluster原理会选举会将 7002的从节点7005选举为主节点。



```csharp
[root@localhost redis-cluster]# ps -ef | grep redis
root       7966      1  0 12:50 ?        00:00:29 ./redis-server 127.0.0.1:7000 [cluster]
root       7950      1  0 12:50 ?        00:00:28 ./redis-server 127.0.0.1:7001 [cluster]
root       7952      1  0 12:50 ?        00:00:29 ./redis-server 127.0.0.1:7002 [cluster]
root       7956      1  0 12:50 ?        00:00:29 ./redis-server 127.0.0.1:7003 [cluster]
root       7960      1  0 12:50 ?        00:00:29 ./redis-server 127.0.0.1:7004 [cluster]
root       7964      1  0 12:50 ?        00:00:29 ./redis-server 127.0.0.1:7005 [cluster]
root      11346  10581  0 14:57 pts/2    00:00:00 grep --color=auto redis
[root@localhost redis-cluster]# kill 7952
```

在查看集群中的7002节点

```ruby
[root@localhost src]# ./redis-trib.rb check 127.0.0.1:7002
>>> Performing Cluster Check (using node 127.0.0.1:7002)
S: 671a0524a616da8b2f50f3d11a74aaf563578e41 127.0.0.1:7002
   slots: (0 slots) slave
   replicates 62a00566233fbff4467c4031345b1db13cf12b46
M: 18948dab5b07e3726afd1b6a42d5bf6e2f411ba1 127.0.0.1:7003
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
M: 62a00566233fbff4467c4031345b1db13cf12b46 127.0.0.1:7005
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
M: f4ee0a501f9aaf11351787a46ffb4659d45b7bd7 127.0.0.1:7001
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
S: 34e322ca50a2842e9f3664442cb11c897defba06 127.0.0.1:7004
   slots: (0 slots) slave
   replicates f4ee0a501f9aaf11351787a46ffb4659d45b7bd7
S: 2cb649ad3584370c960e2036fb01db834a546114 127.0.0.1:7000
   slots: (0 slots) slave
   replicates 18948dab5b07e3726afd1b6a42d5bf6e2f411ba1
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

可以看到集群连接不了7002节点，而7005有原来的S转换为M节点，代替了原来的7002节点。我们可以获取name值：



```ruby
[root@localhost redis]# ./redis-cli -c -p 7001
127.0.0.1:7001> get name
-> Redirected to slot [5798] located at 127.0.0.1:7005
"andy"
127.0.0.1:7005> 
127.0.0.1:7005> 
```

从7001节点连入，自动跳转到7005节点，并且获取name值。

现在我们将7002节点恢复，看是否会自动加入集群中以及充当的M还是S节点。



```csharp
[root@localhost redis-cluster]# cd 7002
[root@localhost 7002]# ./redis-server redis.conf 
[root@localhost 7002]# 
```

再check一下7002节点，可以看到7002节点变成了7005的从节点。

# 缓存读写模式/更新策略

## Cache Aside Pattern

**Cache Aside Pattern，即失效模式，是我们平时使用比较多的一个缓存读写模式，比较适合读请求比较多的场景。**

Cache Aside Pattern 中服务端需要同时维系 DB 和 cache，并且是以 DB 的结果为准。

下面我们来看一下这个策略模式下的缓存读写步骤。

**写** ：

- 先更新 DB
- 然后直接删除 cache 。

**读** :

- 从 cache 中读取数据，读取到就直接返回
- cache中读取不到的话，就从 DB 中读取数据返回
- 再把数据放到 cache 中。

比如说面试官很可能会追问：“**在写数据的过程中，可以先删除 cache ，后更新 DB 么？**”

**答案：** 那肯定是不行的！因为这样可能会造成**数据库（DB）和缓存（Cache）数据不一致**的问题。为什么呢？比如说请求1 先写数据A，请求2随后读数据A的话就很有可能产生数据不一致性的问题。这个过程可以简单描述为：

> 请求1先把cache中的A数据删除 -> 请求2从DB中读取数据->请求1再把DB中的A数据更新。

当你这样回答之后，面试官可能会紧接着就追问：“**在写数据的过程中，先更新DB，后删除cache就没有问题了么？**”

**答案：** 理论上来说还是可能会出现数据不一致性的问题，不过概率非常小，因为缓存的写入速度是比数据库的写入速度快很多！

比如请求1先读数据 A，请求2随后写数据A，并且数据A不在缓存中的话也有可能产生数据不一致性的问题。这个过程可以简单描述为：

> 请求1从DB读数据A->请求2写更新数据 A 到数据库并把删除cache中的A数据->请求1将数据A写入cache。

现在我们再来分析一下 **Cache Aside Pattern 的缺陷**。

**缺陷1：首次请求数据一定不在 cache 的问题**

解决办法：可以将热点数据可以提前放入cache 中。

**缺陷2：写操作比较频繁的话导致cache中的数据会被频繁被删除，这样会影响缓存命中率 。**

解决办法：

- 数据库和缓存数据强一致场景 ：更新DB的时候同样更新cache，不过我们需要加一个锁/分布式锁来保证更新cache的时候不存在线程安全问题。
- 可以短暂地允许数据库和缓存数据不一致的场景 ：更新DB的时候同样更新cache，但是给缓存加一个比较短的过期时间，这样的话就可以保证即使数据不一致的话影响也比较小。

## Read/Write Through Pattern（读写穿透）

Read/Write Through Pattern 中服务端把 cache 视为主要数据存储，从中读取数据并将数据写入其中。cache 服务负责将此数据读取和写入 DB，从而减轻了应用程序的职责。分布式缓存 Redis 并没有提供 cache 将数据写入DB的功能，因此很少用。

**写（Write Through）：**

- 先查 cache，cache 中不存在，直接更新 DB。
- cache 中存在，则先更新 cache，然后 cache 服务自己更新 DB（**同步更新 cache 和 DB**）

**读(Read Through)：**

- 从 cache 中读取数据，读取到就直接返回 。
- 读取不到的话，先从 DB 加载，写入到 cache 后返回响应。

Read-Through Pattern 实际只是在 Cache-Aside Pattern 之上进行了封装。在 Cache-Aside Pattern 下，发生读请求的时候，如果 cache 中不存在对应的数据，是由客户端自己负责把数据写入 cache，而 Read Through Pattern 则是 cache 服务自己来写入缓存的，这对客户端是透明的。

和 Cache Aside Pattern 一样， Read-Through Pattern 也有首次请求数据一定不再 cache 的问题，对于热点数据可以提前放入缓存中。

## Write Behind Pattern（异步缓存写入）

Write Behind Pattern 和 Read/Write Through Pattern 很相似，两者都是由 cache 服务来负责 cache 和 DB 的读写。

但是，两个又有很大的不同：**Read/Write Through 是同步更新 cache 和 DB，而 Write Behind Caching 则是只更新缓存，不直接更新 DB，而是改为异步批量的方式来更新 DB。**

很明显，这种方式对数据一致性带来了更大的挑战，比如cache数据可能还没异步更新DB的话，cache服务可能就就挂掉了。

这种策略在我们平时开发过程中也非常非常少见，但是不代表它的应用场景少，比如消息队列中消息的异步写入磁盘、MySQL 的 InnoDB Buffer Pool 机制都用到了这种策略。

Write Behind Pattern 下 DB 的写性能非常高，非常适合一些数据经常变化又对数据一致性要求没那么高的场景，比如浏览量、点赞量。

###  Redis 和 Memcached 的区别和共同点

**共同点** ：

1. 都是基于内存的数据库，一般都用来当做缓存使用。
2. 都有过期策略。
3. 两者的性能都非常高。

**区别** ：

1. **Redis 支持更丰富的数据类型（支持更复杂的应用场景）**。Redis 不仅仅支持简单的 k/v 类型的数据，同时还提供 list，set，zset，hash 等数据结构的存储。Memcached 只支持最简单的 k/v 数据类型。
2. **Redis 支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用,而 Memecache 把数据全部存在内存之中。**
3. **Redis 有灾难恢复机制。** 因为可以把缓存中的数据持久化到磁盘上。
4. **Redis 在服务器内存使用完之后，可以将不用的数据放到磁盘上。但是，Memcached 在服务器内存使用完之后，就会直接报异常。**
5. **Memcached 没有原生的集群模式，需要依靠客户端来实现往集群中分片写入数据；但是 Redis 目前是原生支持 cluster 模式的.**
6. **Memcached 是多线程，非阻塞 IO 复用的网络模型；Redis 使用单线程的多路 IO 复用模型。** （Redis 6.0 引入了多线程 IO ）
7. **Redis 支持发布订阅模型、Lua 脚本、事务等功能，而 Memcached 不支持。并且，Redis 支持更多的编程语言。**
8. **Memcached过期数据的删除策略只用了惰性删除，而 Redis 同时使用了惰性删除与定期删除。**





















