[TOC]

### 官网介绍

- ZooKeeper主要**服务于分布式系统**，可以用ZooKeeper来做：统一配置管理、统一命名服务、分布式锁、集群管理。
- 使用分布式系统就无法避免对节点管理的问题(需要实时感知节点的状态、对节点进行统一管理等等)，而由于这些问题处理起来可能相对麻烦和提高了系统的复杂性，ZooKeeper作为一个能够**通用**解决这些问题的中间件就应运而生了。

### 必备知识

* Zookeeper = 文件系统 + 通知机制

* Apach Hbase和 Apache solr 以及 Dubbo等项目都采用了Zookeeper
* Zookeeper是一个分布式的、高性能的，开源的分布式系统的协调服务，是Google的Chubby一个开源的实现，是Hadoop 和 Hbase 的重要组件，是一个为分布式应用提供数据一致性服务的软件。具有严格顺序访问控制能力的分布式协调存储服务

* 是一个基于观察者模式设计的分布式服务管理框架，负责存储和管理大家都关心的数据，然后接收观察者Watch的注册，一旦这些数据的状态发生变化，Zookeeper负责通知已经在Zookeeper上注册的那些观察者做出相应的反应，从而实现集群中类似Master/Slave管理模式

* Zookeeper应用场景：
  * 维护配置信息：作为配置中心，保证配置项配置信息一致性，从而达到维护配置信息的效果
  * 分布式锁服务：多台服务器运行同一种服务，协调个服务的进度。为了保证当某个服务需要进行同步操作时，Zookeeper可以对该操作加锁
  * 集群管理：集群管理各种服务器，当一些服务器加入/移出时会通知给其他正常工作的服务器，以即使调整和分配任务。还对故障服务器诊断并尝试修复
  * 生成分布式唯一ID：分库分表后无法再以靠数据库的auto_increment自动生成唯一ID，此时Zookeeper可以在生成新id时创建持久顺序节点，创建操作返回的节点符号，即为新Id，然后把比自己节点小的删除即可。

* Zookeeper提供了一套分布式集群管理机制，基于层次性的目录树的数据结构(文件系统)，并对树中的结点进行有效管理，从而设计出各种分布式数据管理模型，作为分布式系统的沟通调度桥梁。
* Zookeeper Service看作是班主任，client看作学生，watch看作是微信，config Data看作是通知信息，一处更新处处更新

![image-20200713152246279](../../../../Software/Typora/Picture/image-20200713152246279.png)

#### 能干吗：

* 命名服务
* 配置维护
* 集群管理、
* 分布式消息同步和协调机制
* 负载均衡(大多还是用nginx)
* 对Dubbo的支持

#### 怎么玩

* 统一命名服务（Name Service 如 Dubbo服务注册中心）

* 配置管理（Configuration Management，如淘宝考员配置管理框架Diamond）

  * 可以将配置文件放在ZooKeeper的Znode节点中，系统A、B、C监听着这个Znode节点有无变更，如果变更了，**及时**响应。

* 集群管理（Group Membership， 如Hadoop分布式集群管理）

  * 以三个系统A、B、C为例，在ZooKeeper中创建**临时节点**即可：

  * 只要系统A挂了，那`/groupMember/A`这个节点就会删除，通过**监听**`groupMember`下的子节点，系统B和C就能够感知到系统A已经挂了。(新增也是同理)

  * 除了能够感知节点的上下线变化，ZooKeeper还可以实现**动态选举Master**的功能。(如果集群是主从架构模式下)

    原理也很简单，如果想要实现动态选举Master的功能，Znode节点的类型是带**顺序号的临时节点**(`EPHEMERAL_SEQUENTIAL`)就好了。

    - Zookeeper会每次选举最小编号的作为Master，如果Master挂了，自然对应的Znode节点就会删除。然后让**新的最小编号作为Master**，这样就可以实现动态选举的功能了。

### 分布式锁

系统A、B、C都去访问`/locks`节点，访问的时候会创建**带顺序号的临时/短暂**(`EPHEMERAL_SEQUENTIAL`)节点，比如，系统A创建了`id_000000`节点，系统B创建了`id_000002`节点，系统C创建了`id_000001`节点。接着，拿到`/locks`节点下的所有子节点(id_000000,id_000001,id_000002)，**判断自己创建的是不是最小的那个节点**

- 如果是，则拿到锁。

- - 释放锁：执行完操作后，把创建的节点给删掉

- 如果不是，则监听比自己要小1的节点变化

等到系统A执行完操作以后，将自己创建的节点删除(`id_000000`)。通过监听，系统B发现`id_000000`节点已经删除了，发现自己已经是最小的节点了，于是顺利拿到锁

### 选举机制

![image-20201113141626383](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201113141626383.png)

一、首先开始选举阶段，每个Server读取自身的zxid（数据ID，值越大说明数据越新，在选举算法中数据越新权重越大）

二、发送投票信息

  a、首先，每个Server第一轮都会投票给自己。

  b、投票信息包含 ：所选举leader的Serverid（服务器ID），Zxid，Epoch（投票的次数）。Epoch会随着选举轮数的增加而递增。

三、接收投票信息

 1、如果服务器B接收到服务器A的数据（服务器A处于选举状态(LOOKING 状态)

   1）首先，判断逻辑时钟值：

　　　　a）如果发送过来的逻辑时钟Epoch大于目前的逻辑时钟。首先，更新本逻辑时钟Epoch，同时清空本轮逻辑时钟收集到的来自其他server的选举数据。然后，判断是否需要更新当前自己的选举leader Serverid。判断规则rules judging：保存的zxid最大值和leader Serverid来进行判断的。先看数据zxid,数据zxid大者胜出;其次再判断leader Serverid,leader Serverid大者胜出；然后再将自身最新的选举结果(也就是上面提到的三种数据（leader Serverid，Zxid，Epoch）广播给其他server)

　　　　b）如果发送过来的逻辑时钟Epoch小于目前的逻辑时钟。说明对方server在一个相对较早的Epoch中，这里只需要将本机的三种数据（leader Serverid，Zxid，Epoch）发送过去就行。

　　　　c）如果发送过来的逻辑时钟Epoch等于目前的逻辑时钟。再根据上述判断规则rules judging来选举leader ，然后再将自身最新的选举结果(也就是上面提到的三种数据（leader  Serverid，Zxid，Epoch）广播给其他server)。

  2）其次，判断服务器是不是已经收集到了所有服务器的选举状态：若是，根据选举结果设置自己的角色(FOLLOWING还是LEADER)，退出选举过程就是了。

最后，若没有收到没有收集到所有服务器的选举状态：也可以判断一下根据以上过程之后最新的选举leader是不是得到了超过半数以上服务器的支持,如果是,那么尝试在200ms内接收一下数据,如果没有新的数据到来,说明大家都已经默认了这个结果,同样也设置角色退出选举过程。

 2、 如果所接收服务器A处在其它状态（FOLLOWING或者LEADING）。

　　　　a)逻辑时钟Epoch等于目前的逻辑时钟，将该数据保存到recvset。此时Server已经处于LEADING状态，说明此时这个server已经投票选出结果。若此时这个接收服务器宣称自己是leader, 那么将判断是不是有半数以上的服务器选举它，如果是则设置选举状态退出选举过程。
　　　　b) 否则这是一条与当前逻辑时钟不符合的消息，那么说明在另一个选举过程中已经有了选举结果，于是将该选举结果加入到outofelection集合中，再根据outofelection来判断是否可以结束选举,如果可以也是保存逻辑时钟，设置选举状态，退出选举过程。

四、默认是采用投票数大于半数则胜出的逻辑。



![image-20201113141022264](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201113141022264.png)

### 节点结构

ZooKeeper的数据结构，跟Unix文件系统非常类似，可以看做是一颗**树**，每个节点叫做**ZNode**。每一个节点可以通过**路径**来标识。znode有2种类型

- **短暂/临时(Ephemeral)**：当客户端和服务端断开连接后，所创建的Znode(节点)**会自动删除**
- **持久(Persistent)**：当客户端和服务端断开连接后，所创建的Znode(节点)**不会删除**

ZooKeeper和Redis一样，也是C/S结构(分成客户端和服务端)

### 监听器

**常见**的监听场景有以下两项：

- 监听Znode节点的**数据变化**
- 监听子节点的**增减变化**



#### zookeeper配置文件zoo.cfg

* ls -ltr ：该linux命令工作中很常用，作用是按创建顺序显示当前目录下的文件和目录

* 5大参数：
  * tickTime=2000：通信心跳数，Zookeeper服务器心跳时间，单位为毫秒。服务器与客户端或服务器之间维持心跳的时间间隔
  * initLimit=10：集群中follower跟随着服务器(F) 与leader领导者服务器(L)之间初始连接时能容忍的最多心跳书(tickTime的数量)。即主机和从机连接所需的时间
  * syncLimit=5：LF同步通信时限，Leader和Follower之间的最大响应时间单位，加入超过 syncLimit * tickTime，Leader认为Follower死掉，从服务器列表删除Follower
  * dataDir=/tmp/zookeeper：数据存放的目录，默认在临时目录temp，一般需要修改到指定目录。
  * clientPort=2182 ：默认端口为2182

#### 使用

* 进入bin：`./zkServer.sh start`，启动zookeeper服务
* `./zkCli.sh`开启客户端
* zookeeper服务器端的四字命令（在linux下使用而不是zookeeper下）：
  * `echo ruok | nc 127.0.0.1 2181` ：查看zookeeper的服务器端是否准备就绪
  * `echo stat | nc 127.0.0.1 2181` ：查看zookeeper服务器端状态
  * `echo envi | nc 127.0.0.1 2181` ：查看zookeeper服务器端环境
* create /testNode v1：创建节点       get /testNode ：获取节点的data。
* set /testNode  v2：覆盖 testNode的value
* get /testNode ：查询该节点的数据
* ls/ls2 /testNode ：查询该节点信息，ls只显示子节点，ls2显示所有信息
* 删除节点：delete/rmr   /testNode   

#### zookeeper数据模型

* Zookeeper所使用的数据模型风格很像文件系统的目录树结构。有点类似windows中注册表的结构
* 有名称、有树节点，有键值对的关系
* 可以看作是一个树形结构的数据库，但又不能存放数据，作用是对分布在不同机器上做名称管理
* Stat结构体：
  * ![image-20200713173217271](../../../../Software/Typora/Picture/image-20200713173217271.png)

#### 数据模型znode节点

* Zookeeper数据模型结构与Unix文件系统很类似，整体看作一棵树，自身维护一套存储数据结构，每个节点是一个ZNode
* 每一个znode默认能存储1MB数据，每个ZNode都可以通过其路径唯一标识
* create /node1/node2 val   ：报错，不支持多级创建节点

* Znode = path + nodeValue + Stat 。即 set /path nodeValue，自带Stat，存放该节点的一些信息

* znode中的存在类型：持久，临时。但细分要有四种
* create  -s  /myNode v2 : -s 表示节点自动从myNode后面添加序列号，确保该节点不重复
* create  -e  /myNode v2 :  -e 代表临时节点，重启将失效，默认-p(可不写)表示持久化。-s和-e可同时用。

* 一个节点对应一个应用，节点存储的数据就是应用所需要的配置信息

### zookeeper分布式锁

Zookeeper的数据存储结构就像一棵树，这棵树由节点组成，这种节点叫做Znode。

Znode分为四种类型：

**1.持久节点 （PERSISTENT）**

默认的节点类型。创建节点的客户端与zookeeper断开连接后，该节点依旧存在 。

**2.持久节点顺序节点（PERSISTENT_SEQUENTIAL）**

所谓顺序节点，就是在创建节点时，Zookeeper根据创建的时间顺序给该节点名称进行编号：

**3.临时节点（EPHEMERAL）**

和持久节点相反，当创建节点的客户端与zookeeper断开连接后，临时节点会被删除：

**4.临时顺序节点（EPHEMERAL_SEQUENTIAL）**

顾名思义，临时顺序节点结合和临时节点和顺序节点的特点：在创建节点时，Zookeeper根据创建的时间顺序给该节点名称进行编号；当创建节点的客户端与zookeeper断开连接后，临时节点会被删除。

### **Zookeeper分布式锁的原理**

Zookeeper分布式锁恰恰应用了临时顺序节点。具体如何实现呢？让我们来看一看详细步骤：

#### **获取锁**

首先，在Zookeeper当中创建一个持久节点ParentLock。当第一个客户端想要获得锁时，需要在ParentLock这个节点下面创建一个**临时顺序节点** Lock1。

之后，Client1查找ParentLock下面所有的临时顺序节点并排序，判断自己所创建的节点Lock1是不是顺序最靠前的一个。如果是第一个节点，则成功获得锁。

这时候，如果再有一个客户端 Client2 前来获取锁，则在ParentLock下载再创建一个临时顺序节点Lock2。

Client2查找ParentLock下面所有的临时顺序节点并排序，判断自己所创建的节点Lock2是不是顺序最靠前的一个，结果发现节点Lock2并不是最小的。

于是，Client2向排序仅比它靠前的节点Lock1注册**Watcher**，用于监听Lock1节点是否存在。这意味着Client2抢锁失败，进入了等待状态。

这时候，如果又有一个客户端Client3前来获取锁，则在ParentLock下载再创建一个临时顺序节点Lock3。

Client3查找ParentLock下面所有的临时顺序节点并排序，判断自己所创建的节点Lock3是不是顺序最靠前的一个，结果同样发现节点Lock3并不是最小的。

于是，Client3向排序仅比它靠前的节点**Lock2**注册Watcher，用于监听Lock2节点是否存在。这意味着Client3同样抢锁失败，进入了等待状态。

这样一来，Client1得到了锁，Client2监听了Lock1，Client3监听了Lock2。这恰恰形成了一个等待队列，很像是Java当中ReentrantLock所依赖的

### 释放锁

释放锁分为两种情况：

**1.任务完成，客户端显示释放**

当任务完成时，Client1会显示调用删除节点Lock1的指令。

**2.任务执行过程中，客户端崩溃**

获得锁的Client1在任务执行过程中，如果Duang的一声崩溃，则会断开与Zookeeper服务端的链接。根据临时节点的特性，相关联的节点Lock1会随之自动删除。

由于Client2一直监听着Lock1的存在状态，当Lock1节点被删除，Client2会立刻收到通知。这时候Client2会再次查询ParentLock下面的所有节点，确认自己创建的节点Lock2是不是目前最小的节点。如果是最小，则Client2顺理成章获得了锁。

同理，如果Client2也因为任务完成或者节点崩溃而删除了节点Lock2，那么Client3就会接到通知。最终，Client3成功得到了锁。

### 方案

可以直接使用zookeeper第三方库[Curator](https://curator.apache.org/)客户端，这个客户端中封装了一个可重入的锁服务。

Curator提供的InterProcessMutex是分布式锁的实现。acquire方法用户获取锁，release方法用于释放锁。

 ![image-20200920155839897](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200920155839897.png)

**缺点：**

  性能上可能并没有缓存服务那么高。因为每次在创建锁和释放锁的过程中，都要动态创建、销毁瞬时节点来实现锁功能。ZK中创建和删除节点只能通过Leader服务器来执行，然后将数据同不到所有的Follower机器上。

  其实，使用Zookeeper也有可能带来并发问题，只是并不常见而已。考虑这样的情况，由于网络抖动，客户端可ZK集群的session连接断了，那么zk以为客户端挂了，就会删除临时节点，这时候其他客户端就可以获取到分布式锁了。就可能产生并发问题。这个问题不常见是因为zk有重试机制，一旦zk集群检测不到客户端的心跳，就会重试，Curator客户端支持多种重试策略。多次重试之后还不行的话才会删除临时节点。（所以，选择一个合适的重试策略也比较重要，要在锁的粒度和并发之间找一个平衡。）

#### 通话机制：Session + Watch

* 客户端注册监听它关心的目录节点，当目录节点发生变化(数据改变，被删除、子目录节点增加删除)时，zookeeper会通知客户端。 

* Session ：通过Java建立Zookeeper会话，此时处于CONNECTING状态，当客户端连接Zookeeper服务成功后进入CONNECTED状态 ，结束状态CLOSED
* Watch（观察者）
  * 异步+回调+的触发机制
  * 异步和回调的理解：假如面试我一结束，面试官口渴托我买瓶可乐，如果在我去买可乐过程中面试官一直在等而不继续面试，这叫同步；若在我买水的过程中面试继续进行，这是异步；若买水时发现没有可乐，这时打电话询问面试官说明情况的这个过程，就叫异步回调。
  * 客户端 可以在每个znode节点上设置一个Watcher，如果被观察的服务端的znode节点有变更，watch会触发，这和watch所属的客户端将接收到一个通知包被告知节点已经变化，把响应的事件通知给设置Watcher的Client端
  * zookeeper里所有读取操作：getData() ,  getCildren() 和 exists()  都有设置watch的选项
* watch事件理解
  * 1.一次性触发：只监控一次，触发一次后不再有效。一般不多用
  * 2.发送客户端
  * 3.为数据设置watch
  * 4.时序性和一致性

## Java操作Zookeeper

#### Maven工程和配置POM

* 依赖

  ```
          <!-- 注册中心使用的是zookeeper，引入操作zookeeper的客户端 -->
  		<dependency>
  			<groupId>org.apache.curator</groupId>
  			<artifactId>curator-framework</artifactId>
  			<version>2.12.0</version>
  		</dependency>
          <dependency>
              <groupId>org.apache.zookeeper</groupId>
              <artifactId>zookeeper</artifactId>
              <version>RELEASE</version>
          </dependency>
  ```

* WatchOne 一次性触发watch，数据监控

```
public class WatchOne {
    private static final Logger logger = LoggerFactory.getLogger(WatchOne.class);
    private final static String CONNECTSTRING = "192.168.137.133:2181";
    private final static int SESSION_TIMEOUT = 50 * 1000;
    private final static String PATH = "/atguigu";
    private ZooKeeper zk = null;

    public ZooKeeper startZK() throws Exception {
        return new ZooKeeper(CONNECTSTRING, SESSION_TIMEOUT, new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {
            }
        });
    }

    public void stopZK() throws Exception {
        if(zk != null)  zk.close();
    }

    public void createZnode(String nodePath, String nodeValue) throws Exception {
        // OPEN_ACL_UNSAFE:类似关闭防火墙
        // CreateMode.PERSISTENT: 创建的节点由Java控制是持久的
        zk.create(nodePath, nodeValue.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
    }

    public String getZnode(String nodePath) throws Exception {
        String result = null;
        byte[] byteArray = zk.getData(PATH, new Watcher() { // 一次性触发
            @Override
            public void process(WatchedEvent event) {
                try {
                    trigerValue(PATH);
                }  catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }, new Stat());
        result = new String(byteArray); // 转为字符串
        return result;
    }

    public String trigerValue(String nodePath) throws Exception {
        String result = null;
        byte[] byteArray = zk.getData(PATH, false, new Stat());
        result = new String(byteArray); // 转为字符串
        return result;
    }

    public static void main(String[] args) throws Exception {
        WatchOne watchOne = new WatchOne();
        watchOne.setZk(watchOne.startZK());
        if(watchOne.getZk().exists(PATH, false) == null){ // 没有该节点才能创建，不然会报错，节点是不能覆盖的
            watchOne.createZnode( PATH, "AAA");
            String retValue = watchOne.getZnode(PATH);
            logger.info("retValue="+retValue);
            Thread.sleep(Long.MAX_VALUE);
        }else{
            logger.info("I have no znode");
        }
    }
    public ZooKeeper getZk() {
        return zk;
    }
    public void setZk(ZooKeeper zk) {
        this.zk = zk;
    }
}
```

#### WatchMore 长久数据监控（常用）

```
public class WatchMore {
    private static final Logger logger = LoggerFactory.getLogger(WatchMore.class);
    private final static String CONNECTSTRING = "192.168.137.133:2181";
    private final static int SESSION_TIMEOUT = 50 * 1000;
    private final static String PATH = "/atguigu";
    private ZooKeeper zk = null;
    private String oldValue = null;

    public ZooKeeper startZK() throws Exception {
        return new ZooKeeper(CONNECTSTRING, SESSION_TIMEOUT, new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {
            }
        });
    }

    public void stopZK() throws Exception {
        if(zk != null)  zk.close();
    }

    public void createZnode(String nodePath, String nodeValue) throws Exception {
        // OPEN_ACL_UNSAFE:类似关闭防火墙
        // CreateMode.PERSISTENT: 创建的节点由Java控制是持久的
        zk.create(nodePath, nodeValue.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
    }

    public String getZnode(String nodePath) throws Exception {
        String result = null;
        byte[] byteArray = zk.getData(PATH, new Watcher() { // 一次性触发
            @Override
            public void process(WatchedEvent event) {
                try {
                    trigerValue(PATH);
                }  catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }, new Stat());
        result = new String(byteArray); // 转为字符串
        oldValue = result; // 一开始初始值
        return result;
    }

	// 每次所观察的节点改变就触发 
    public boolean trigerValue(String nodePath) throws Exception {
        String result = null;
        byte[] byteArray = zk.getData(PATH, new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {
                try {
                    trigerValue(nodePath);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }, new Stat());
        result = new String(byteArray); // 转为字符串
        String newValue = result;
        if(oldValue.equals(newValue)){
            logger.info("*********no changes");
            return false;
        }else{
            logger.info("************oldValue:"+oldValue+"\t newValue:"+newValue);
            oldValue = newValue;
            return true;
        }
    }
	// 监控节点 /atguigu下的数据
    public static void main(String[] args) throws Exception {
        WatchMore watchOne = new WatchMore();
        watchOne.setZk(watchOne.startZK());
        if(watchOne.getZk().exists(PATH, false) == null){ // 没有该节点才能创建，不然会报错，节点是不能覆盖的
            watchOne.createZnode( PATH, "AAA");
            String retValue = watchOne.getZnode(PATH);
            logger.info("retValue="+retValue);
            Thread.sleep(Long.MAX_VALUE);
        }else{
            logger.info("I have no znode");
        }
    }
    public ZooKeeper getZk() {
        return zk;
    }
    public void setZk(ZooKeeper zk) {
        this.zk = zk;
    }

    public String getOldValue() {
        return oldValue;
    }

    public void setOldValue(String oldValue) {
        this.oldValue = oldValue;
    }
}
```



#### 子节点变化监控（不常用）

```
package com.xuecheng.manage_course;

import org.apache.zookeeper.*;
import org.apache.zookeeper.Watcher.Event.EventType;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.List;


public class WatchChild {
    private static final Logger logger = LoggerFactory.getLogger(WatchChild.class);

    private final static String CONNECTSTRING = "192.168.137.133:2181";
    private final static int SESSION_TIMEOUT = 50 * 1000;
    private final static String PATH = "/atguigu";
    private ZooKeeper zk = null;

    public ZooKeeper startZK() throws Exception {
        return new ZooKeeper(CONNECTSTRING, SESSION_TIMEOUT, new Watcher() { // 开启监控
            @Override
            public void process(WatchedEvent event) {
                // 子节点没改变
                if(event.getType() == EventType.NodeChildrenChanged && event.getPath().equals(PATH)){
                    showChildNode(PATH); // 子节点每次改变都会执行该方法
                }else{
                    showChildNode(PATH); // 一开始会注册父亲节点并打印初始子节点
                }
            }
        });
    }

    public void showChildNode(String nodePath) {
        List<String> list = null;
        try {
            // 获取该节点所有子节点
            list = zk.getChildren(PATH, true); // true表示该节点下全部监控，自带连续监控
            logger.info("***********"+list);
        } catch (KeeperException | InterruptedException e) {
            e.printStackTrace();
        }
    }

    // 监控子节点
    public static void main(String[] args) throws Exception {
        WatchChild watchChild = new WatchChild();
        watchChild.setZk(watchChild.startZK());
        Thread.sleep(Long.MAX_VALUE);

    }
    public ZooKeeper getZk() {
        return zk;
    }
    public void setZk(ZooKeeper zk) {
        this.zk = zk;
    }
}

```



#### Zookeeper集群

* 分为服务器端和客户端，客户端之连接到某个服务器上，客户端使用并维护一个TCP连接，第一次连接会建立会话，当这个客户端连接到另外服务器时，这个会话会被新的服务器重新建立

* 配置项书写格式：server.N = YYY:A:B，其中，N为服务器编号，YYY表示服务器IP地址，A为LF(主从机)通信端口，即该服务器与集群中的leader交换的信息的端口。B为选举端口，即选举新leader时服务器间相互通信的端口（当leader挂掉后，其余服务器会相互通信并选举出新的leader）

* 真集群中每个服务器A端口和B端口都是一样的。
* 伪集群中每个服务器A端口和B端口都是不一样的。但IP一样
* 伪集群做法：
  * 分别复制zookeeper目录为zk01、zk02、zk03并分别在配置文件修改以下：

![image-20200714010132087](../../../../Software/Typora/Picture/image-20200714010132087.png)

![image-20200714010140415](../../../../Software/Typora/Picture/image-20200714010140415.png)



​		此时使用zk01和zk02作为服务器，zk03连接server作为客户端：`./zkCli.sh -server 127.0.0.1:2193`。

​		此时使用zk03改变节点数据，zk01和zk02便可查到刚改变的数据，完成伪集群







