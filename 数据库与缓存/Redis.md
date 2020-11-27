# Nosql：非关系型数据库

* 分表分库 + 水平拆分 + mysql集群：
  * 在Memcached的高速缓存，Mysql主从复制、读写分离的基础上，由于MyISAM使用表锁，高并发Mysql应用开始使用InnoDB引擎代替MyISAM。现如今分表分库 + 水平拆分 + mysql集群 已经成为解决缓解写压力和数据增长的问题的热门技术。


* NoSQL用于超大规模数据的存储。这些类型的数据存储不需要固定的模式，无需多余操作就可以横向扩展。

* \- 代表着不仅仅是SQL
  \- 没有声明性查询语言
  \- 没有预定义的模式
  -键 - 值对存储，列存储，文档存储，图形数据库
  \- 最终一致性，而非ACID属性
  \- 非结构化和不可预知的数据

  \- 高性能，高可用性和可伸缩性

  \- CAP定理

* CAP定理：对于一个分布式计算系统来说，不可能同时满足以下三点:

  * **一致性(Consistency)** (所有节点在同一时间具有相同的数据)
  * **可用性(Availability)** (保证每个请求不管成功或者失败都有响应)
  * **分区容忍(Partition tolerance)** (系统中任意信息的丢失或失败不会影响系统的继续运作)

  ​      CAP理论的核心是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，最多只能同时较好的满足两个。

  ​      因此，根据 CAP 原理将 NoSQL 数据库分成了满足 CA 原则、满足 CP 原则和满足 AP 原则三 大类：

  * CA - 单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大。
  * CP - 满足一致性，分区容忍性的系统，通常性能不是特别高。
  * AP - 满足可用性，分区容忍性的系统，通常可能对一致性要求低一些。

* NoSQL数据模型：

  * 聚合模型：（主要是前两个）
    * KV键值
    * Bson：类似JSON，可以在 Value中放 JSON， 可分可合
    * 列族：每个字段放在一行
    * 图形

* NoSQL数据库四大分类：


  * KV键值：典型介绍：阿里百度（memcache+redis），美团（redis+tair），新浪（BerkeleyDB+redis）
  * 文档型数据库：json格式比较多，如MongoDB(基于分布式文件存储数据库)
  * 列存储数据库：分布式文件系统、Cassandra、HBase
  * 图关系数据库：存放关系图，如：朋友圈社交网络、广告推荐系统、社交系统、推荐系统等，专注于构建关系图谱。Neo4J，InfoGrid

# Redis（Remote Dictionary Server（远程字典服务器））

* Redis = KV + Cache + persistence
* 3V+3高：海量Volume、多样Variety、实时Velocity、高并发、高可扩、高性能
* BASE（牺牲CAP 中的 C 换取 AP）
  * 基本可用（Basically Available）
  * 软状态（Soft state）
  * 最终一致（Eventually consistent）
* 分布式 + 集群：
  * 分布式：不同的多台服务器上部署不同的服务模块（工程），他们之间通过RPC/Rmi之间通信和调用，对外提供服务和组内工作
  * 集群：不同的多台服务器上面部署相同的服务模块，通过分布式调度软件进行统一的调度，对外提供服务和访问
* Redis支持数据持久化，数据类型包括：KV、list、set、zset、hash等数据结构的存储。Redis支持master-slave模式的数据备份
* Linux中Redis开启服务：`/usr/local/bin/redis-server /opt/myRedis/redis.conf`     
* 启动Redis客户端：`redis-cli -p 6379 `      6379是Redis默认端口    退出：exit 或 ctrl+c     关闭服务：`SHUTDOWN`
* ps -ef|grep redis ：查看redis服务的是否启动
* lsof -i :6379  ：根据redis 的端口查看redis进程
* redis默认安装在：/usr/local/bin 
  * ![image-20200712143609109](../../../../Software/Typora/Picture/image-20200712143609109.png)

#### Redis常用命令

* Redis默认在redis.conf中配置16个数据库0-15，  select 7  ：表示使用6号数据库,  

* set 和 get 能存放和取出数据， 其中set key val   时key已经存在，则覆盖

* `keys k*`  :  查询以 k 开头的键         FLUSH：删除           

* `keys *` ：查询所有key，`move k 2`：将 k 移动到2号数据库

* exists key：判断这个key是否在当前数据库中存在

* expire key 1 :  设置key过期时间为1秒  

* ttl key : 查看还有多少秒过期，-1表示永不过期，-2表示已经过期

* type key ： 查看key的类型

* del key：删除key


#### Redis五大value数据类型

* String：与Memcached一样的类型，即一个key对应一个value。String是二进制安全的，意思是可以包含任何数据类型。value最多可以是512M
  
  * **应用场景** ：一般常用在需要计数的场景，比如用户的访问次数、热点文章的点赞转发数量等等。
    
  * STRLEN key : 查看key的长度     INCR key：该key的值+1        DECR key则-1
  
  * DEL key
  
  * EXPIRE key 5 // 过期时间5秒
  
  * GETRANGE key 0 -1 :  获取key的value并截取索引长度，0 -1表示截取全部
  
  * SETRANGE key 0 val: 从该key的value的索引位置0开始插入值 val  
  
  * setex  key  10   val :  新增key-val 同时设置过期时间为10
  
  * setnx  key  val :  如果该key不存在，则新增key-val ，如果key存在则该命令不生效
  
  * xmset k1 v1 k2 v2 k3 v3: 插入多个数据 
  
  * msetnx k1 v1 k2 v2 k3 v3 ：若k1 k2 k3 中有一个已经存在，则全部新增失败
  
  * ##### MSET [表]:[id]:[key] val  如：MSET user1:name xxx user:1:balance 1999
  
  * ##### MGET [表]:[id]:[key] ，如：MGETuser1:name user:1:balance
  
  MSET和MGET比set、get性能更好
  
  原子加减
  
  * INCRBY key 3 和 DECRBY key 3：分别是每次+3和每次-3
  * INCR key 和 DECR key // 加1减1
  
* Hash：类似Java的Map，是一个String类型的field和value的映射表，hash特别适合用于存储对象
  
  * **应用场景:** 系统中对象数据的存储。
  * HSET user id 11：插入键值对 id-11
  * HGET user id：获取value
  * HMSET user id 11 name lisi age 18：一口气插入         HMGET user id name age
  * HGETALL user：查询所有映射关系
  * HDEL user name：删除
  * HLEN user：长度
  * HEXISTS user id：查询该表的id 这个key是否存在
  * HKEYS user：查询所有key         HVALS user：查询所有value
  * HINCRBY user age 2：key为age的value，每次 +2  
  * HINCRBYFLOAD user age 0.5：key为age的value，每次 +0.5
  * HSETNX user age 16： 不存在age这个key才插入有效

> **1、hset(name, key, value)**
>
> ```
> # name对应的hash中设置一个键值对（不存在，则创建；否则，修改）
>  
> # 参数：
>     # name，redis的name
>     # key，name对应的hash中的key
>     # value，name对应的hash中的value
>  
> # 注：
>     # hsetnx(name, key, value),当name对应的hash中不存在当前key时则创建（相当于添加）
> ```

> **2、hmset(name, mapping)**
>
> ```
> # 在name对应的hash中批量设置键值对
>  
> # 参数：
>     # name，redis的name
>     # mapping，字典，如：{'k1':'v1', 'k2': 'v2'}
>  
> # 如：
>     # r.hmset('xx', {'k1':'v1', 'k2': 'v2'})
> ```

> **3、hget(name,key)**
>
> ```
> # 在name对应的hash中获取根据key获取value
> ```

> **4、hmget(name, keys, \*args)**
>
> ```
> # 在name对应的hash中获取多个key的值
>  
> # 参数：
>     # name，reids对应的name
>     # keys，要获取key集合，如：['k1', 'k2', 'k3']
>     # *args，要获取的key，如：k1,k2,k3
>  
> # 如：
>     # r.mget('xx', ['k1', 'k2'])
>     # 或
>     # print r.hmget('xx', 'k1', 'k2')
> ```

> #### 5、**hgetall(name)**
>
> ```
> # 获取name对应hash的所有键值
> print(re.hgetall('xxx').get(b'name'))
> ```

> **6、hlen(name)**
>
> ```
> # 获取name对应的hash中键值对的个数
> ```

> **7、hkeys(name)**
>
> ```
> # 获取name对应的hash中所有的key的值
> ```

> **8、hvals(name)**
>
> ```
> # 获取name对应的hash中所有的value的值
> ```

> **9、hexists(name, key)**
>
> ```
> # 检查name对应的hash是否存在当前传入的key
> ```

> **10、hdel(name,\*keys)**
>
> ```
> # 将name对应的hash中指定key的键值对删除
> print(re.hdel('xxx','sex','name'))
> ```

> **11、hincrby(name, key, amount=1)**
>
> ```
> # 自增name对应的hash中的指定key的值，不存在则创建key=amount
> # 参数：
>     # name，redis中的name
>     # key， hash对应的key
>     # amount，自增数（整数）
> ```

> **12、hincrbyfloat(name, key, amount=1.0)**
>
> ```
> # 自增name对应的hash中的指定key的值，不存在则创建key=amount
>  
> # 参数：
>     # name，redis中的name
>     # key， hash对应的key
>     # amount，自增数（浮点数）
>  
> # 自增name对应的hash中的指定key的值，不存在则创建key=amount
> ```

> #### 13、**hscan(name, cursor=0, match=None, count=None)**
>
> ```
> # 增量式迭代获取，对于数据大的数据非常有用，hscan可以实现分片的获取数据，并非一次性将数据全部获取完，从而放置内存被撑爆
>  
> # 参数：
>     # name，redis的name
>     # cursor，游标（基于游标分批取获取数据）
>     # match，匹配指定key，默认None 表示所有的key
>     # count，每次分片最少获取个数，默认None表示采用Redis的默认分片个数
>  
> # 如：
>     # 第一次：cursor1, data1 = r.hscan('xx', cursor=0, match=None, count=None)
>     # 第二次：cursor2, data1 = r.hscan('xx', cursor=cursor1, match=None, count=None)
>     # ...
>     # 直到返回值cursor的值为0时，表示数据已经通过分片获取完毕
> ```

> **14、hscan_iter(name, match=None, count=None)**
>
> ```
> # 利用yield封装hscan创建生成器，实现分批去redis中获取数据
>  
> # 参数：
>     # match，匹配指定key，默认None 表示所有的key
>     # count，每次分片最少获取个数，默认None表示采用Redis的默认分片个数
>  
> # 如：
>     # for item in r.hscan_iter('xx'):
>     #     print item
> ```

* List（列表）：底层是个链表，是简单的字符串列表，按照插入顺序排序，可以头插也可以尾插。类似**双向循环链表LinkedList**
  * **应用场景:** 发布与订阅或者说消息队列、慢查询。
  * LPUSH list01 1 2 3 4 ：依次从最左边插入到ilist01，结果为4 3 2 1 ， RPUSH同理
  * LRANGE list01 0 -1: 查询 集合 list01所有value，0 和 -1 表示索引位置
  * lpop list01：表示list01的左边作为栈顶出栈一个值，同理rpop表示list01的右边作为栈顶出栈一个值
  * LINDEX list01 3 : 查询列表list01索引3的value
  * LLEN list01 ： 列表长度
  * LREM list01  2  3  : 删除2个索引为3的元素
  * LTRAM list01 3 5 ： 截取索引为3到5的元素并重新覆盖list01
  * RPOPLPUSH：list01 list02  ： 相当于 RPOP list01 的结果 LPUSH 进 list02
  * LSET list01 1 x：从左边起索引为1的位置上插入string类型的 x
  * LINSERT list01 before/after x java：在x的左边/右边插入元素 java
* Set (集合)：是string类型的无序集合，不允许重复，是通过**HashTable**实现的
  * **应用场景:** 需要存放的数据不能重复以及需要获取多个数据源交集和并集等场景
  * sadd set01 1 1 2 2 3 3：只添加 1 2 3
  * SMEMBERS set01 0  -1：查询set01集合，也可以用SMEMBERS set01
  * SISMEMBER set01 x：判断x是否存在
  * SCART set01：获取集合的元素个数
  * SREM set01 3：删除集合中值为 ‘3’ 的元素
  * SRANDMEMBER：set01 3 ：从set01中随机得到3个元素
  * SPOP set01：随机出栈一个元素
  * SMOVE set01 set02 5：将set01中 5 这个元素移动到set02
  * DEL set01：删除集合
  * SDIFF set01 set02：输出set01中set02所没有的元素，即差集
  * SINTER set01 set02：输出交集
  * SUNION set01 set02：输出并集，
* Zset (sortedSet有序集合)：是string类型的有序集合且不允许重复，每个元素都会关联一个**double**类型的分数，通过分数**从小到大**排序。zset成员是唯一的，但分数可以重复  
  * **应用场景：** 需要对数据根据某个权重进行排序的场景。比如在直播系统中，实时排行信息包含直播间在线用户列表，各种礼物排行榜，弹幕消息（可以理解为按消息维度的消息排行榜）等信息。
  * ZADD zset01 60 v1 70 v2：60和70表示分数，越小排序越前
  * ZRANGE zset01 0 -1：查询所有          ZRANGE zset01 0 -1 WITHSCOPES：查询所有且带分数
  * ZRANGEBYSCOPE zset01 60 90：查询分数60-90的元素          ZRANGEBYSCOPE zset01 60 (90 ： 查询分数60-90的元素但不包含90
  * ZRANGEBYSCOPE zset01 60 90 limit 2 3：从分数60-90的元素中从索引为2的位置截取3个
  * ZREM zset01 v1：删除
  * ZCARD zset01 ：查询元素个数              ZCOUNT zset01 60 80：查询分数在60-90元素个数
  * ZRANK zset01 v2：查询元素下标       ZSCOP zset01 v2：查询元素分数
  * ZREVRANK zset01 v2：逆序获得下标值                ZREVRANGE zset01 0 -1：逆序输出
  * ZREVRANGEBYSCOPE zset01 60-90：查询分数60-90的元素并逆序输出

#### String字符串

- string是redis最基本的类型，一个key对应一个value。
- string类型是二进制安全的。意思是redis的string可以包含任何数据。比如jpg图片或者序列化的对象 。
- string类型是Redis最基本的数据类型，一个键最大能存储512MB。
- [String类型的操作参考-](http://www.runoob.com/redis/redis-strings.html)

#### 链表list

- redis列表是简单的字符串列表，排序为插入的顺序。列表的最大长度为2^32-1。
- redis的列表是使用链表实现的，这意味着，即使列表中有上百万个元素，增加一个元素到列表的头部或尾部的操作都是在常量的时间完成。
- 可以用列表获取最新的内容（像帖子，微博等），用ltrim很容易就会获取最新的内容，并移除旧的内容。
- 用列表可以实现生产者消费者模式，生产者调用lpush添加项到列表中，消费者调用rpop从列表中提取，如果没有元素，则轮询去获取，或者使用brpop等待生产者添加项到列表中。
- [List类型的操作参考](http://www.runoob.com/redis/redis-lists.html)

#### 集合

- redis集合是无序的字符串集合，集合中的值是唯一的，无序的。可以对集合执行很多操作，例如，测试元素是否存在，对多个集合执行交集、并集和差集等等。
- 我们通常可以用集合存储一些无关顺序的，表达对象间关系的数据，例如用户的角色，可以用sismember很容易就判断用户是否拥有某个角色。
- 在一些用到随机值的场合是非常适合的，可以用 srandmember/spop 获取/弹出一个随机元素。
  同时，使用@EnableCaching开启声明式缓存支持，这样就可以使用基于注解的缓存技术。注解缓存是一个对缓存使用的抽象，通过在代码中添加下面的一些注解，达到缓存的效果。
- [Set类型的操作参考](http://www.runoob.com/redis/redis-sets.html)

#### ZSet 有序集合

- 有序集合由唯一的，不重复的字符串元素组成。有序集合中的每个元素都关联了一个浮点值，称为分数。可以把有序看成hash和集合的混合体，分数即为hash的key。
- 有序集合中的元素是按序存储的，不是请求时才排序的。
- [ZSet类型的操作类型](http://www.runoob.com/redis/redis-sorted-sets.html)

#### Hash-哈希

- redis的哈希值是字符串字段和字符串之间的映射，是表示对象的完美数据类型。
- 哈希中的字段数量没有限制，所以可以在你的应用程序以不同的方式来使用哈希。
- [Hash类型的操作参考](http://www.runoob.com/redis/redis-hashes.html)

#### redis.conf

* redis.conf 默认安装在：/opt/myRedis

* Units单位：配置大小单位，定义了一些基本的度量单位，只支持bytes不支持bit，对大小写不敏感

* INCLUDES：包含其他配置文件，作为总闸

* GENERAL：

  * daemonize yes：启用守护进程。设置redis关掉终端后，redis服务和进程还在执行。

  * port 6379：默认端口

  * tcp-backlog 511：配置tcp的backlog ，是一个连接队列，backlog 队列总和=未完成三次握手队列+已经完成三次握手队列，高并发环境下需要一个高backlog 值来避免客户端连接慢问题

  * #bind IP：设置连接的IP

  * timeout 0：设置空闲多少秒后关闭redis连接，0表示一直连接不关闭

  * Tcp-Keepalive：单位为秒，如果设置为0，则不会进行Keepalive检测，建议设置为60，作用类似心跳检测、判断每过一段时间检测是否还存活、使用中

  * loglevel  debug/verbose/notice/warning  ：设置log日志级别，从左到右级别依次变高

  * database 16：设置数据库数量，默认为16

  * Syslog-enabled：是否把日志输出到syslog中，默认不使用

  * Syslog-facility：指定syslog设备，值可以是USER或LOCAL0-LOCAL7

* SNAPSHOTTING快照（RDB）

  * dbfilename dump.rdb :  默认快照备份的文件为dump.rdb
  * stop-writes-on-bgsave-error  yes ：默认yes表示备份出错就立即停止写数据。如果配置成no，表示不在乎数据不一致
  * rdbcompression yes ： 对于存储到磁盘中的快照，可以设置是否进行压缩存储。默认yes表示采用LZF算法进行压缩 ，会消耗cpu，建议开启
  * rdbchecksum yes: 在存储快照后，可以使用CRC64算法来进行数据检验，但会增大约10%的性能消耗，建议开启
  * dir : 设置快照的备份文件dump.rdb 所在目录，config get dir 可以获得目录 

* REPLICATION复制

* SECURITY安全：

  *  config set requirepass "123456" : 设置密码，此时若直接ping，则会失败，需要先输入密码：auth 123456
  *  config get requirepass : 获取密码

* LIMITS限制： 

  * Maxclients：最大同时连接数，默认10000
  * Maxmemory：单位为字节，最大内存
  * Maxmemory-policy：缓存策略 (六大策略)。默认是noeviction永不过期策略，但一般不使用该策略
    * ![image-20200712231742607](../../../../Software/Typora/Picture/image-20200712231742607.png)
    * volatile-lru：使用LRU(最近最少使用)算法移除key，只对设置了过期时间的键有效
    * allkeys-lru：使用LRU算法移除key
    * volatile-random：在过期集合中移除随机的key，只对设置了过期时间的key有效
    * allkeys-random：移除随机TTL的key
    * volatile-ttl：移除那些TTL值最小的key，即那些最近要过期的key
    * noeviction：不进行移除，针对写操作，只是返回错误信息，一般不使用该策略
  * Maxmemory-samples：设置样本数量，LRU算法和最小TTL算法都不是最精确的算法，而是估算值，所以你可以设置样本大小，redis默认会检查这么多个key并选择其中LRU的那个。默认选取5个样本

* APPEND ONLY MODE追加（AOF）

  * appendonly no : 默认关闭AOF
  * appendfilename "appendonly.aof" ：日志记录保存到该文件, 数据修改一次，追加一次日志记录到该文件
  * appendfsync：3中AOF持久化策略
    * aways：同步持久化，每次发生数据变更会被立即记录到磁盘，性能较差但数据完整性较好
    * everysec：出厂默认推荐使用，异步操作，每秒记录，如果最后一秒内宕机，会有最后一秒还没持久化的数据将丢失。
    * no：从不同步
  * No-appendfsync-on-rewrite：重写时是否可以运用Appendfsync，默认no，保证数据安全性
  * auto-aof-rewrite-percentage  100：百分百情况下AOF文件大小是上次rewrite后大小的一倍
  * auto-aof-rewrite-min-size  64mb：AOF文件大于64M时触发重写机制。实际公司设置 3G 起步

#### 常见redis.conf配置

```
参数说明
redis.conf 配置项说明如下：
1. Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程
  daemonize no
2. 当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定
  pidfile /var/run/redis.pid
3. 指定Redis监听端口，默认端口为6379，作者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字
  port 6379
4. 绑定的主机地址
  bind 127.0.0.1
5.当 客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能
  timeout 300
6. 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose
  loglevel verbose
7. 日志记录方式，默认为标准输出，如果配置Redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给/dev/null
  logfile stdout
8. 设置数据库的数量，默认数据库为0，可以使用SELECT <dbid>命令在连接上指定数据库id
  databases 16
9. （RDB）指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合
  save <seconds> <changes>
  Redis默认配置文件中提供了三个条件：
  save 900 1
  save 300 10
  save 60 10000
  分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。
 
10. 指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大
  rdbcompression yes
11. 指定本地数据库文件名，默认值为dump.rdb
  dbfilename dump.rdb
12. 指定本地数据库存放目录
  dir ./
13. 设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步
  slaveof <masterip> <masterport>
14. 当master服务设置了密码保护时，slave服务连接master的密码
  masterauth <master-password>
15. 设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH <password>命令提供密码，默认关闭
  requirepass foobared
16. 设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回max number of clients reached错误信息
  maxclients 128
17. 指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区
  maxmemory <bytes>
  
18. 是否开启AOF，指定是否在每次更新操作后进行追加日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no。
  appendonly no   
19. 指定更新日志文件名，默认为appendonly.aof
   appendfilename appendonly.aof
20. 指定更新日志条件，共有3个可选值： 
  no：表示等操作系统进行数据缓存同步到磁盘（快） 
  always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全） 
  everysec：表示每秒同步一次（折衷，默认值）
  appendfsync everysec
 
21. 指定是否启用虚拟内存机制，默认值为no，VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中
   vm-enabled no
22. 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享
   vm-swap-file /tmp/redis.swap
23. 将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0
   vm-max-memory 0
24. Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不 确定，就使用默认值
   vm-page-size 32
25.设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，，在磁盘上每8个pages将消耗1byte的内存。
   vm-pages 134217728
26. 设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4
   vm-max-threads 4
27. 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启
  glueoutputbuf yes
28. 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法
  hash-max-zipmap-entries 64
  hash-max-zipmap-value 512
29. 指定是否激活重置哈希，默认为开启（后面在介绍Redis的哈希算法时具体介绍）
  activerehashing yes
30. 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件
  include /path/to/local.conf
```

#### 持久化RDB（Redis DataBase）

* Redis持久化包含RDB（Redis DataBase快照）和AOF（Append Only File日志）
* RDB：在指定时间间隔内将内存中的数据快照写入磁盘，是SnapShot快照，它恢复时是将快照文件直接读到内存里，实现持久化
* Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。在整个过程中**主线程**不进行任何IO操作，确保了极高的性能。
* 如果需要进行大规模数据恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加高效。
* RDB的缺点是最后一次持久化后的数据可能丢失

* Fork的作用是复制一个与当前进程一样的子进程。新进程的所有数据（变量、环境变量、程序计数器等）数值都和原进程一致

* RDB保存的是**dump.rdb**文件，命令是：save <seconds> <changes> .  默认有以下3种保存策略(可以自定义) ， save "" 则是禁用该功能
  * 三个策略分别为：15分钟内改了1次，5分钟内改了10次，1分钟内改了一万次，会触发快照条件从而备份文件，备份文件为dump.rdb
  * ![image-20200713122608467](../../../../Software/Typora/Picture/image-20200713122608467.png)
* 可以在任何一个目录下使用redis，所生成的db文件也保存在该目录
* 一般备份dump.rdb时需要拷贝文件到另外的备机上，以防物理损坏
* SHUTDOWN关闭redis的同时，默认会commit并备份到dump.rdb中。slushall命令也会产生dump.rdb，但空文件无意义
* 如何恢复数据？重启服务后，默认会将启动redis的当前目录下的备份文件dump.rdb 恢复到redis数据库中

* 手动备份命令：save 或者 bgsave。其中save只管保存，其他不管全部阻塞。BGSAVE：表示redis会在后台异步进行快照操作，快照同时还可以响应客户端请求，可以通过lastsave命令获取最后一次成功执行快照的时间。
* config get dir 可以获得快照的备份文件dump.rdb所在目录 
* 优势：适合大规模数据恢复，对数据完整性和一致性要求不高。
* 劣势：fork时内存的数据被克隆了一份，大致2倍的膨胀性需要考虑内存空间是否足够大。最后一次备份的数据可能丢失
* 动态停止所有RDB保存规则命令：redis-cli config set save ""。不建议使用
* 配置文件里的 SNAPSHOTTING快照（RDB）
  * dbfilename dump.rdb :  默认快照备份的文件为dump.rdb
  * stop-writes-on-bgsave-error  yes ：默认yes表示备份出错就立即停止写数据。如果配置成no，表示不在乎数据不一致
  * rdbcompression yes ： 对于存储到磁盘中的快照，可以设置是否进行压缩存储。默认yes表示采用LZF算法进行压缩 ，会消耗cpu，建议开启
  * rdbchecksum yes: 在存储快照后，可以使用CRC64算法来进行数据检验，但会增大约10%的性能消耗，建议开启
  * dir : 设置快照的备份文件dump.rdb 所在目录，config get dir 可以获得目录 
* ![image-20200713131151648](../../../../Software/Typora/Picture/image-20200713131151648.png)

#### 持久化AOF（Append Only File）

* AOF是在RDB之后产生，是以日志的形式记录每个写操作，将Redsi执行过的所有写指令记录，只需追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据。换言之，redis重启时会根据日志文件的内容将写指令从头到尾执行一次以完成数据的恢复工作

* AOF也是启动redis后自动执行日志文件appendonly.aof 从而恢复数据。

* `/usr/local/bin/redis-server /myredis/redis_aof.conf` ：启动Redis服务并以aof文件恢复数据

* dump.rdb和 appendonly.aof 可以同时存在，先加载appendonly.aof，若aof文件中有记录是错的，开启redis服务会失败。此时在redis的bin目录下使用命令：`redis-check-aof --fix appendonly.aof` 可查看appendonly.aof 的错误信息并消除其中的不规范指令，才能启动redis服务

* appendonly.aof 因为只追加写操作记录，因此容易内存膨胀，free命令用于查看内存使用情况，df -h 查看磁盘空间

* Rewrite：

  * AOF的重写机制，当AOF文件大小超过所设定的阈值时，会启动AOF文件的内容压缩，只保留可以不恢复数据的最小指令集，可以使用命令bgrewriteaof
  * 原理：AOF文件持续增长而过大时，会fork出一条新进程来将文件重写(先写临时文件最后再rename)，遍历新进程的内存中数据，每条记录有一条Set语句。重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方法重写了一个新aof文件，类似快照。 

  * 触发机制：redis会记录上次重写时的AOF大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发

* （优势）对应在redis.conf  的配置：APPEND ONLY MODE追加（AOF）

  * appendonly no : 默认关闭AOF
  * appendfilename "appendonly.aof" ：日志记录保存到该文件, 数据修改一次，追加一次日志记录到该文件
  * appendfsync：3中AOF持久化策略
    * aways：同步持久化，每次发生数据变更会被立即记录到磁盘，性能较差但数据完整性较好
    * everysec：出厂默认推荐使用，异步操作，每秒记录，如果一秒内宕机，会有数据丢失。
    * no：从不同步
  * No-appendfsync-on-rewrite：重写时是否可以运用Appendfsync，默认no，保证数据安全性
  * auto-aof-rewrite-percentage  100：百分百情况下AOF文件大小是上次rewrite后大小的一倍
  * auto-aof-rewrite-min-size  64mb：AOF文件大于64M时触发重写机制。实际公司设置 3G 起步

* 劣势：相同数据集的数据而言aof文件远大于rdb文件，恢复速度慢于rdb。运行效率也慢，但每秒同步策略效率较好，不同步效率和rdb相同..

* 如果Enalbe AOF，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件就可以了。代价一是带来了持续的IO，二是AOF rewrite的最后将rewrite过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘许可，应该尽量减少AOF rewrite的频率，AOF重写的基础大小默认值64M太小了，可以设到5G以上。默认超过原大小100%大小时重写可以改到适当的数值。

  如果不Enable AOF ，仅靠Master-Slave Replication 实现高可用性也可以。能省掉一大笔IO也减少了rewrite时带来的系统波动。代价是如果Master/Slave同时倒掉，会丢失十几分钟的数据，启动脚本也要比较两个Master/Slave中的RDB文件，载入较新的那个。新浪微博就选用了这种架构

#### Redis事务

* 可以一次执行多个命令，本质是一组命令的集合，一个事务中的所有命令都会序列化，按顺序地串行化执行而不会被其他命令插入，不许加塞
* 能干嘛？开启事务后每个指令都会加入一个队列中，一次性、顺序性、排他性地执行一系列命令
* 常用命令
  * DISCARD：取消事务，放弃执行事务块内的所有命令
  * EXEC：执行所有事务块的命令，执行之后会一次性显示该次事务块中所有命令的结果
  * MULTI：标记一个事务块的开始
  * UNWATCH：取消WATCH命令对**所有**key的监视
  * WATCH key  [key...] ：类似乐观锁，监视一个或多个key，如果在事务EXEC执行之前这些key被其他命令所改动，那么事务将被打断，该事务提交无效

* ##### 若事务块中有无效命令(错误指令)则全部命令不生效，若事务块中都是有效命令但有些命令无法执行成功，则只有这些命令执行失败，其他命令无影响

* watch监控
  * 事务提交时，如果key已被其他客户端改变，那么整个事务队列都不会被执行，同时返回Nullmulti-bulk应答以通知调用者事务执行失败
  * 乐观锁(常用)：开启WATCH后，若有命令要修改数据，则在要修改的该行数据加一个字段：版本号version，每次修改完成后缓存中该行的version+1，以后修改该行的请求所带的版本不等于缓存中version，则会报错。此时需要重新从缓存中获取最新数据后再修改才能提交成功，保证数据一致性
  * 悲观锁(不常用)：认为每次修改数据都会觉得会有别的请求也要修改，因此锁整张表
  * 一旦执行了exec，之前加的监控锁都会被取消掉
  * CAS(Check And Set) 

* 事务3阶段：MULTI开启事务=》命令入队，此时还未被执行=》EXEC执行，触发事务
* Redis事务3特性：
  * 单独的隔离操作：事务中的所有命令都会被序列化、按顺序执行，事务执行过程中不会被其他客户端的命令请求打断
  * 没有隔离级别的概念：因为事务提交前任何指令都不会被实际执行
  * 不保证原子性：同一事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚

#### 发布订阅机制

* 进程间的一种消息通信模式：发送者（pub）发布消息，订阅者（sub）接收消息

* 一次性订阅多个： SUBSCRIBE c1 c2 c3 c*

* 消息发布： PUBLISH c2 hello-redis


#### 主从复制（Master/Slave）

* 配从机（库）、主机（库）

* 从库配置：slaveof  主库IP  主库端口

* 配置从库时需修改配置文件，需修改：端口号、进程号pidfile、日志logfile、RDB快照文件dbfilename

* 通常主从复制有4招（一般都用哨兵模式）
  * 一主二从：一个主机2个从机
  * 薪火相传：主=》从=》从。。。。若中途变更转向，会清除之前的数据，重新建立拷贝最新的数据. 命令：Slaveof  新主库IP  新主库端口
  * 反客为主： SLAVEOF no one：使当前数据库停止与其他数据库的同步，转为主数据库
  * 哨兵模式：反客为主的自动版，能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库
  
* info replication可以查看当前redis的详细信息，也可查看是主库还是从库

* ##### 读写分离：默认只有主机可以写，从机只能读

* 默认当主机宕机或关闭Redis服务后，其他从机还是从机，即原地待命

* 当从机宕机或关闭Redis服务，重新启动Redis后，需要重新连接主库，除非redis.conf 已经有配置

* Slave启动成功连接到master后会发送一个sync命令


#### 哨兵模式

* 反客为主的自动版，能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库
* 使用步骤：
  * 自定义/myRedis目录下新建 sentinel.conf
  * 配置主机监控：在sentinel.conf配置：**sentinel monitor host6379 127.0.0.1 6379 1**  
  * 最后的数字1表示主机挂掉后slave投票，得票数的多少后成为主机
  * 启动哨兵：redis-sentinel /opt/myRedis/sentinel.conf
  * ##### 当旧主机修复后重新启动，此时哨兵监控到后会将该旧主机转换为新主库的从库

* 一组sentinel能同时监控多个master
* 主从复制的缺点：由于写操作都是先在Master进行，然后同步更新到slave上，所以主库同步到从库有一定延迟，Slave数量增加，延迟会更加严重

### Java使用Redis

#### key的存活时间：

无论什么时候，只要有可能就利用key超时的优势。一个很好的例子就是储存一些诸如临时认证key之类的东西。当你去查找一个授权key时——以OAUTH为例——通常会得到一个超时时间。
这样在设置key的时候，设成同样的超时时间，Redis就会自动为你清除。

#### 关系型数据库的redis

1: 把表名转换为key前缀 如  tag:
2: 第2段放置用于区分区key的字段--对应mysql中的主键的列名,如userid
3: 第3段放置主键值,如2,3,4...., a , b ,c
4: 第4段,写要存储的列名
例：user:userid:9:username

```
1. 连接池自动管理，提供了一个高度封装的“RedisTemplate”类

2. 针对jedis客户端中大量api进行了归类封装,将同一类型操作封装为operation接口

ValueOperations：简单K-V操作
SetOperations：set类型数据操作
ZSetOperations：zset类型数据操作
HashOperations：针对map类型的数据操作
ListOperations：针对list类型的数据操作

3. 提供了对key的“bound”(绑定)便捷化操作API，可以通过bound封装指定的key，然后进行一系列的操作而无须“显式”的再次指定Key，即BoundKeyOperations：

BoundValueOperations
BoundSetOperations
BoundListOperations
BoundSetOperations
BoundHashOperations

4. 将事务操作封装，有容器控制。

5. 针对数据的“序列化/反序列化”，提供了多种可选择策略(RedisSerializer)

JdkSerializationRedisSerializer：POJO对象的存取场景，使用JDK本身序列化机制，将pojo类通过ObjectInputStream/ObjectOutputStream进行序列化操作，最终redis-server中将存储字节序列。是目前最常用的序列化策略。

StringRedisSerializer：Key或者value为字符串的场景，根据指定的charset对数据的字节序列编码成string，是“new String(bytes, charset)”和“string.getBytes(charset)”的直接封装。是最轻量级和高效的策略。

JacksonJsonRedisSerializer：jackson-json工具提供了javabean与json之间的转换能力，可以将pojo实例序列化成json格式存储在redis中，也可以将json格式的数据转换成pojo实例。因为jackson工具在序列化和反序列化时，需要明确指定Class类型，因此此策略封装起来稍微复杂。【需要jackson-mapper-asl工具支持】

OxmSerializer：提供了将javabean与xml之间的转换能力，目前可用的三方支持包括jaxb，apache-xmlbeans；redis存储的数据将是xml工具。不过使用此策略，编程将会有些难度，而且效率最低；不建议使用。【需要spring-oxm模块的支持】
如果你的数据需要被第三方工具解析，那么数据应该使用StringRedisSerializer而不是JdkSerializationRedisSerializer。

```

* 连接使用

```java
	public static void main(String[] args) {
		Jedis jedis = new Jedis("127.0.0.1", 6379);
		jedis.set("k1", "v1");
		System.out.println(jedis.ping());
		Set<String> keys = jedis.keys("*");
	}
```

* 事务

```
	public static void main(String[] args) {
		Jedis jedis = new Jedis("127.0.0.1", 6379);
		Transaction transaction = jedis.multi();
		//transaction.discard(); // 事务取消
		transaction.exec();
	}
```

* 事务-watch监控-加锁

```java
public class TestTransaction {
 
  public boolean transMethod() {
     Jedis jedis = new Jedis("127.0.0.1", 6379);
     int balance;// 可用余额
     int debt;// 欠额
     int amtToSubtract = 10;// 实刷额度
     jedis.watch("balance");// watch监控某个字段
     //jedis.set("balance","5");//此句不该出现。模拟其他程序已经修改了该条目
     Thread.sleep(7000); // 延时7秒，模拟高并发或网络延迟，在7秒的过程中有其它程序改变了balance
     balance = Integer.parseInt(jedis.get("balance"));
     if (balance < amtToSubtract) {
       jedis.unwatch();
       System.out.println("modify");
       return false;
     } else {
       System.out.println("***********transaction");
       Transaction transaction = jedis.multi();
       transaction.decrBy("balance", amtToSubtract);
       transaction.incrBy("debt", amtToSubtract);
       transaction.exec();
       balance = Integer.parseInt(jedis.get("balance"));
       debt = Integer.parseInt(jedis.get("debt"));
 
       System.out.println("*******" + balance);
       System.out.println("*******" + debt);
       return true;
     }
  }
 
  /**
   * 通俗点讲，watch命令就是标记一个键，如果标记了一个键， 在提交事务前如果该键被别人修改过，那事务就会失败，这种情况通常可以在程序中
   * 重新再尝试一次。
   * 首先标记了键balance，然后检查余额是否足够，不足就取消标记，并不做扣减； 足够的话，就启动事务进行更新操作，
   * 如果在此期间键balance被其它人修改， 那在提交事务（执行exec）时就会报错， 程序中通常可以捕获这类错误再重新执行一次，直到成功。
   */
  public static void main(String[] args) {
     TestTransaction test = new TestTransaction();
     boolean retValue = test.transMethod();
     System.out.println("main retValue-------: " + retValue);
  }
}
```

* 主从复制（配从不配主）

```
public static void main(String[] args) throws InterruptedException 
  {
     Jedis jedis_M = new Jedis("127.0.0.1",6379);
     Jedis jedis_S = new Jedis("127.0.0.1",6380);
     
     jedis_S.slaveof("127.0.0.1",6379);
     
     jedis_M.set("k6","v6");
     Thread.sleep(500);
     System.out.println(jedis_S.get("k6"));
  }
```

### JedisPool

* JedisPoolUtil

```java
public class JedisPoolUtil {
  private static volatile JedisPool jedisPool = null;//被volatile修饰的变量不会被本地线程缓存，对该变量的读写都是直接操作共享内存。
  private JedisPoolUtil() {}  // 单例模式，构造方法私有化，无法new该实例对象
  
  public static JedisPool getJedisPoolInstance() // 对外提供获得单实例JedisPool的方法
 {
     if(null == jedisPool)
    {
       synchronized (JedisPoolUtil.class) // 单例模式的双重校验锁
      {
          if(null == jedisPool) 
         {
           JedisPoolConfig poolConfig = new JedisPoolConfig();
// pool可分配的jedis实例，pool.getResource()获取；如果为-1，则表示不限制；如果pool已经分配了maxActive个jedis实例，则此时pool的状态exhausted。
           poolConfig.setMaxActive(1000); 
           // 控制一个pool最多有多少个状态为idle(空闲)的jedis实例；
           poolConfig.setMaxIdle(32);
           // 表示当borrow一个jedis实例时，最大的等待时间，如果超过等待时间，则直接抛JedisConnectionException；
           poolConfig.setMaxWait(100*1000);  
           // 获得一个jedis实例的时候是否检查连接可用性（ping()）；如果为true，则得到的jedis实例均是可用的；
           poolConfig.setTestOnBorrow(true); 
		   // 带上配置连接redis
           jedisPool = new JedisPool(poolConfig,"127.0.0.1");
         }
      }
    }
     return jedisPool;
 }
  
  public static void release(JedisPool jedisPool,Jedis jedis)
 {
     if(null != jedis)
    {
      jedisPool.returnResourceObject(jedis);
    }
 }
```

### RedisTemplate

- spring-redis中使用了RedisTemplate来进行redis的操作，通过泛型的K和V设置键值对的对象类型。这里使用了string作为key的对象类型，值为Object。
- 对于Object，spring-redis默认使用了jdk自带的序列化，不推荐使用默认了。所以使用了json的序列化方式
- 对spring-redis对redis的五种数据类型也有支持
- HashOperations：对hash类型的数据操作
- ValueOperations：对redis字符串类型数据操作
- ListOperations：对链表类型的数据操作
- SetOperations：对无序集合类型的数据操作
- ZSetOperations：对有序集合类型的数据操作

##### pom：spring-boot-starter-data-redis

##### properties

```properties
# Redis数据库索引（默认为0）
spring.redis.database=0  
# Redis服务器地址
spring.redis.host=127.0.0.1
# Redis服务器连接端口
spring.redis.port=6379  
# Redis服务器连接密码（默认为空）
spring.redis.password=
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.pool.max-active=8  
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.pool.max-wait=-1  
# 连接池中的最大空闲连接
spring.redis.pool.max-idle=8  
# 连接池中的最小空闲连接
spring.redis.pool.min-idle=0  
# 连接超时时间（毫秒）
spring.redis.timeout=0  
```

##### 切换数据库

```
JedisConnectionFactory connectionFactory = (JedisConnectionFactory) redisTemplate.getConnectionFactory();
connectionFactory.setDatabase(9);//切换9号数据库
```

##### RedisConfig配置类

```java
/**
 * @author janti
 * reids 相关bean的配置
 */
@Configuration
@EnableCaching
public class RedisConfig extends CachingConfigurerSupport {

    /**
     * 选择redis作为默认缓存工具
     * @param redisTemplate
     * @return
     */
    @Bean
    public CacheManager cacheManager(RedisTemplate redisTemplate) {
        RedisCacheManager rcm = new RedisCacheManager(redisTemplate);
        return rcm;
    }

    /**
     * retemplate相关配置
     * @param factory
     * @return
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {

        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // 配置连接工厂
        template.setConnectionFactory(factory);

        //使用Jackson2JsonRedisSerializer来序列化和反序列化redis的value值（默认使用JDK的序列化方式）
        Jackson2JsonRedisSerializer jacksonSeial = new Jackson2JsonRedisSerializer(Object.class);

        ObjectMapper om = new ObjectMapper();
        // 指定要序列化的域，field,get和set,以及修饰符范围，ANY是都有包括private和public
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        // 指定序列化输入的类型，类必须是非final修饰的，final修饰的类，比如String,Integer等会跑出异常
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jacksonSeial.setObjectMapper(om);

        // 值采用json序列化
        template.setValueSerializer(jacksonSeial);
        //使用StringRedisSerializer来序列化和反序列化redis的key值
        template.setKeySerializer(new StringRedisSerializer());

        // 设置hash key 和value序列化模式
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(jacksonSeial);
        template.afterPropertiesSet();

        return template;
    }

    /**
     * 对hash类型的数据操作
     *
     * @param redisTemplate
     * @return
     */
    @Bean
    public HashOperations<String, String, Object> hashOperations(RedisTemplate<String, Object> redisTemplate) {
        return redisTemplate.opsForHash();
    }

    /**
     * 对redis字符串类型数据操作
     *
     * @param redisTemplate
     * @return
     */
    @Bean
    public ValueOperations<String, Object> valueOperations(RedisTemplate<String, Object> redisTemplate) {
        return redisTemplate.opsForValue();
    }

    /**
     * 对链表类型的数据操作
     *
     * @param redisTemplate
     * @return
     */
    @Bean
    public ListOperations<String, Object> listOperations(RedisTemplate<String, Object> redisTemplate) {
        return redisTemplate.opsForList();
    }

    /**
     * 对无序集合类型的数据操作
     *
     * @param redisTemplate
     * @return
     */
    @Bean
    public SetOperations<String, Object> setOperations(RedisTemplate<String, Object> redisTemplate) {
        return redisTemplate.opsForSet();
    }

    /**
     * 对有序集合类型的数据操作
     * @param redisTemplate
     * @return
     */
    @Bean
    public ZSetOperations<String, Object> zSetOperations(RedisTemplate<String, Object> redisTemplate) {
        return redisTemplate.opsForZSet();
    }
}
```

#### 操作工具类

```java
@Component
public class RedisService {
    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    /**
     * 默认过期时长，单位：秒
     */
    public static final long DEFAULT_EXPIRE = 60 * 60 * 24;

    /**
     * 不设置过期时长
     */
    public static final long NOT_EXPIRE = -1;

    public boolean existsKey(String key) {
        return redisTemplate.hasKey(key);
    }

    /**
     * 重名名key，如果newKey已经存在，则newKey的原值被覆盖
     *
     * @param oldKey
     * @param newKey
     */
    public void renameKey(String oldKey, String newKey) {
        redisTemplate.rename(oldKey, newKey);
    }

    /**
     * newKey不存在时才重命名
     *
     * @param oldKey
     * @param newKey
     * @return 修改成功返回true
     */
    public boolean renameKeyNotExist(String oldKey, String newKey) {
        return redisTemplate.renameIfAbsent(oldKey, newKey);
    }

    /**
     * 删除key
     *
     * @param key
     */
    public void deleteKey(String key) {
        redisTemplate.delete(key);
    }

    /**
     * 删除多个key
     *
     * @param keys
     */
    public void deleteKey(String... keys) {
        Set<String> kSet = Stream.of(keys).map(k -> k).collect(Collectors.toSet());
        redisTemplate.delete(kSet);
    }

    /**
     * 删除Key的集合
     *
     * @param keys
     */
    public void deleteKey(Collection<String> keys) {
        Set<String> kSet = keys.stream().map(k -> k).collect(Collectors.toSet());
        redisTemplate.delete(kSet);
    }

    /**
     * 设置key的生命周期
     *
     * @param key
     * @param time
     * @param timeUnit
     */
    public void expireKey(String key, long time, TimeUnit timeUnit) {
        redisTemplate.expire(key, time, timeUnit);
    }

    /**
     * 指定key在指定的日期过期
     *
     * @param key
     * @param date
     */
    public void expireKeyAt(String key, Date date) {
        redisTemplate.expireAt(key, date);
    }

    /**
     * 查询key的生命周期
     *
     * @param key
     * @param timeUnit
     * @return
     */
    public long getKeyExpire(String key, TimeUnit timeUnit) {
        return redisTemplate.getExpire(key, timeUnit);
    }

    /**
     * 将key设置为永久有效
     *
     * @param key
     */
    public void persistKey(String key) {
        redisTemplate.persist(key);
    }
}
```

#### redis的key工具类

```java
/**
 * redisKey设计
 */
public class RedisKeyUtil {

    /**
     * redis的key
     * 形式为：
     * 表名:主键名:主键值:列名
     *
     * @param tableName 表名
     * @param majorKey 主键名
     * @param majorKeyValue 主键值
     * @param column 列名
     * @return
     */
    public static String getKeyWithColumn(String tableName,String majorKey,String majorKeyValue,String column){
        StringBuffer buffer = new StringBuffer();
        buffer.append(tableName).append(":");
        buffer.append(majorKey).append(":");
        buffer.append(majorKeyValue).append(":");
        buffer.append(column);
        return buffer.toString();
    }
    /**
     * redis的key
     * 形式为：
     * 表名:主键名:主键值
     *
     * @param tableName 表名
     * @param majorKey 主键名
     * @param majorKeyValue 主键值
     * @return
     */
    public static String getKey(String tableName,String majorKey,String majorKeyValue){
        StringBuffer buffer = new StringBuffer();
        buffer.append(tableName).append(":");
        buffer.append(majorKey).append(":");
        buffer.append(majorKeyValue).append(":");
        return buffer.toString();
    }
}
```

#### 注解缓存的使用

- @Cacheable：在方法执行前Spring先查看缓存中是否有数据，如果有数据，则直接返回缓存数据；没有则调用方法并将方法返回值放进缓存。
- @CachePut：将方法的返回值放到缓存中。
- @CacheEvict：删除缓存中的数据。

### 批量操作

一、背景

- 需求：一次性获取redis缓存中多个key的value
- 潜在隐患：循环key，获取value，可能会造成连接池的连接数增多，连接的创建和摧毁，消耗性能
- 解决方法：根据项目中的缓存数据结构的实际情况，数据结构为string类型的，使用RedisTemplate的multiGet方法；数据结构为hash，使用Pipeline(管道)，组合命令，批量操作redis。

二、操作

1. RedisTemplate的multiGet的操作

   - 针对数据结构为String类型

   - 示例代码

     ```java
     List<String> keys = new ArrayList<>();
     for (Book e : booklist) {
        String key = generateKey.getKey(e);
        keys.add(key);
     }
     List<Serializable> resultStr = template.opsForValue().multiGet(keys);
     ```

     此方法还是比较好用，使用者注意封装。

2. RedisTemplate的Pipeline使用

   1）方式一 ： 基础方式

   - 使用类：StringRedisTemplate

   - 使用方法

     ```java
     public executePipelined(RedisCallback<?> action) {...}
     ```

   - 示例代码：批量获取value

     ```java
     List<Object> redisResult = redisTemplate.executePipelined(new RedisCallback<String>() {
        @Override
         public String doInRedis(RedisConnection redisConnection) throws DataAccessException {  
             for (BooK e : booklist) {
            StringRedisConnection stringRedisConnection =(StringRedisConnection)redisConnection;
             stringRedisConnection.get(e.getId());
             }
            return null;
         }
     });
     ```

   1. 方法二 :  使用自定义序列化方法

   - 使用类：RedisTemplate

   - 使用方法

     ```java
     public List<Object> executePipelined(final RedisCallback<?> action, final RedisSerializer<?> resultSerializer) {...}
     ```

   - 示例代码：批量获取hash数据结构value

     ```java
     List<Object> redisResult = redisTemplate.executePipelined(
       new RedisCallback<String>() {
         // 自定义序列化
         RedisSerializer keyS = redisTemplate.getKeySerializer();
         @Override
         public String doInRedis(RedisConnection redisConnection) throws DataAccessException {
             for (BooK e : booklist) {
                   redisConnection.hGet(keyS.serialize(e.getName()), keyS.serialize(e.getAuthor()));
             }
             return null;
         }
       }, redisTemplate.getValueSerializer()); // 自定义序列化
     ```

# 缓存问题

### 缓存穿透

​	缓存穿透是指缓存和数据库中都没有的数据，而用户不断发起请求，如发起为id为“-1”的数据或id为特别大不存在的数据。这时的用户很可能是攻击者，攻击会导致数据库压力过大。

 **解决方案：**

1. 接口层增加校验，如用户鉴权校验，id做基础校验，id<=0的直接拦截；
2. 从缓存取不到的数据，在数据库中也没有取到，这时也可以将key-value对写为key-null，缓存有效时间可以设置短点，如30秒（设置太长会导致正常情况也没法使用）。这样可以防止攻击用户反复用同一个id暴力攻击

### 缓存击穿

​	缓存击穿是指缓存中没有（一般是缓存时间到期）但数据库中有的数据，这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力

   **解决方案：**

1. 设置热点数据永远不过期。
2. 加互斥锁，互斥锁参考代码如下：

![image-20200916132938008](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916132938008.png)

### 缓存雪崩

​	缓存雪崩是指缓存中数据大批量到过期时间，而查询数据量巨大，引起数据库压力过大甚至down机。和缓存击穿不同的是，缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库。

   **解决方案**：

1. 缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。
2. 如果缓存数据库是分布式部署，将热点数据均匀分布在不同的缓存数据库中（采用一致性哈希）。
3. 设置热点数据永远不过期。

# 面试问题

### 热点数据需要大批量更新，怎么解决？

可以把这些热点数据写在一个内存的临时表里，因为innoDB会维护一个 buffer pool（数据库缓存池），如果直接把大量数据全部读进去，可能会造成flush操作（把脏页刷回 MySQL），从而造成线上业务的阻塞。 也可以使用Redis进行缓存

### 如果热点数据太猛，Redis可能因为带宽被打满，访问不到，如何解决？

* 如果要缓存的数据过多或者缓存空间不够，可以使用缓存淘汰策略
* 也可以使用本地缓存，在不考虑高并发情况下可以使用HashMap
* 如果是在高并发情况下，可以使用ConcurrentHashMap 作为缓存集合，再配合缓存淘汰策略。
* 还可以使用guava cache组件实现本地缓存，

### Redis 的持久化机制是什么？各自的优缺点？

Redis提供两种持久化机制 RDB 和 AOF 机制:

1、RDBRedis DataBase)持久化方式：

是指用数据集快照的方式半持久化模式)记录 redis 数据库的所有键值对,在某个时间点将数据写入一个临时文件，持久化结束后，用这个临时文件替换上次持久化的文件，达到数据恢复。

优点：

（1）只有一个文件 dump.rdb，方便持久化。

（2）容灾性好，一个文件可以保存到安全的磁盘。

（3）性能最大化，fork 子进程来完成写操作，让主进程继续处理命令，所以是 IO最大化。使用单独子进程来进行持久化，主进程不会进行任何 IO 操作，保证了 redis的高性能

（4）相对于数据集大时，比 AOF 的启动效率更高。

缺点：

数据安全性低。RDB 是间隔一段时间进行持久化，如果持久化之间 redis 发生故障，会发生数据丢失。所以这种方式更适合数据要求不严谨的时候

2、AOFAppend-only file)持久化方式：

是指所有的命令行记录以 redis 命令请求协议的格式完全持久化存储)保存为 aof 文件。

优点：

（1）数据安全，aof 持久化可以配置 appendfsync 属性，有 always，每进行一次命令操作就记录到 aof 文件中一次。

（2）通过 append 模式写文件，即使中途服务器宕机，可以通过 redis-check-aof工具解决数据一致性问题。

（3）AOF 机制的 rewrite 模式。AOF 文件没被 rewrite 之前（文件过大时会对命令进行合并重写），可以删除其中的某些命令（比如误操作的 flushall）)

缺点：

（1）AOF 文件比 RDB 文件大，且恢复速度慢。

（2）数据集大的时候，比 rdb 启动效率低。

### Redis为什么这么快?

Redis采用的是基于内存的采用的是**单进程单线程**模型的 **KV 数据库**，用”单线程-多路复用IO模型”来实现高性能的内存数据服务的，**由C语言编写**，官方提供的数据是可以达到100000+的QPS（每秒内查询次数）。这个数据不比采用单进程多线程的同样基于内存的 KV 数据库 Memcached 差

**横轴是连接数，纵轴是QPS**。

![image-20200916132943502](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916132943502.png)

##### 1、完全基于内存，绝大部分请求是纯粹的内存操作，非常快速。数据存在内存中，类似HashMap，HashMap的优势就是查找和操作的时间复杂度都是O(1)；

##### 2、数据结构简单，对数据操作也简单，Redis中的数据结构是专门进行设计的；

##### 3、采用单线程，避免了不必要的上下文切换和竞争条件，也避免了多线程之间的的切换而消耗 CPU，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗；

##### 4、使用多路I/O复用模型，非阻塞IO；

##### 多路 I/O 复用模型;

多路I/O复用模型是利用 select、poll、**epoll 可以同时监察多个流的 I/O 事件**的能力，在空闲的时候，会把当前线程阻塞掉，当有一个或多个流有 I/O 事件时，就从阻塞态中唤醒，于是程序就会轮询一遍所有的流（epoll 是只轮询那些真正发出了事件的流），并且只依次顺序的处理就绪的流，这种做法就避免了大量的无用操作。

**这里“多路”指的是多个网络连接，“复用”指的是复用同一个线程。**采用多路 I/O 复用技术可以让单个线程高效的处理多个连接请求（尽量减少网络 IO 的时间消耗），且 Redis 在内存中操作数据的速度非常快，也就是说内存内的操作不会成为影响Redis性能的瓶颈。

##### 5、底层实现方式以及与客户端之间通信的应用协议构建了VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求；

##### VM(virtual memory)机制：

Redis使用到了VM,在redis.conf设置vm-enabled yes 即开启VM功能。 通过VM功能可以实现冷热数据分离。使热数据仍在内存中，冷数据保存到磁盘。这样就可以避免因为内存不足而造成访问速度下降的问题。Redis并没有使用OS提供的Swap，而是自己实现。

相比于OS的交换方式。redis可以将被交换到磁盘的对象进行压缩,保存到磁盘的对象可以去除指针和对象元数据信息。一般压缩后对象会比内存中对象小10倍。这样redis的vm会比OS vm能少做很多io操作。OS交换的时候，是会阻塞线程的，而Redis可以设置让工作线程来完成，主线程仍可以继续接收client的请求。

当前VM的坏处： slow restart 重启太慢 slow saving 保存数据太慢 slow replication 上面两条导致 replication 太慢 complex code 代码过于复杂

### 为什么Redis是单线程？

因为Redis是基于内存的操作，CPU不是Redis的瓶颈，Redis的瓶颈最有可能是机器内存的大小或者网络带宽。既然单线程容易实现，而且CPU不会成为瓶颈，那就顺理成章地采用单线程的方案了，不采用多线程可以避免很多麻烦，如：避免了不必要的上下文切换和竞争条件，也避免了多线程之间的的切换而消耗 CPU，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗；

用”单线程-多路复用IO模型”来实现高性能的内存数据服务的，这种机制避免了使用锁，但是同时这种机制在进行sunion之类的比较耗时的命令时会使redis的并发下降。因为是单一线程，所以同一时刻只有一个操作在进行，所以，耗时的命令会导致并发的下降，不只是读并发，写并发也会下降。而单一线程也只能用到一个CPU核心，所以可以在同一个多核的服务器中，可以启动多个实例，组成master-master或者master-slave的主从复制形式，耗时的读命令可以完全在slave进行。可以通过修改redis.conf 启动多个实例

```
pidfile /var/run/redis/redis_6377.pid  #pidfile要加上端口号
port 6377  #这个是必须改的
logfile /var/log/redis/redis_6377.log #logfile的名称也加上端口号
dbfilename dump_6377.rdb  #rdbfile也加上端口号
```

同时有多个子系统去set一个key。这个时候要注意什么呢？ 不推荐使用redis的事务机制。因为我们的生产环境，基本都是redis集群环境，做了数据分片操作。你一个事务中有涉及到多个key操作的时候，这多个key不一定都存储在同一个redis-server上。因此，redis的事务机制，十分鸡肋。
(1)如果对这个key操作，不要求顺序： 准备一个分布式锁，大家去抢锁，抢到锁就做set操作即可
(2)如果对这个key操作，要求顺序： 分布式锁+时间戳。 假设这会系统B先抢到锁，将key1设置为{valueB 3:05}。接下来系统A抢到锁，发现自己的valueA的时间戳早于缓存中的时间戳，那就不做set操作了。以此类推。
(3) 利用队列，将set方法变成串行访问也可以redis遇到高并发，如果保证读写key的一致性
对redis的操作都是具有原子性的,是线程安全的操作,你不用考虑并发问题,redis内部已经帮你处理好并发的问题了。

#### 注意点：

“我们不能任由操作系统负载均衡，因为我们自己更了解自己的程序，所以，我们可以手动地为其分配CPU核，而不会过多地占用CPU，或是让我们关键进程和一堆别的进程挤在一起。”。 
CPU 是一个重要的影响因素，由于是单线程模型，Redis 更喜欢大缓存快速 CPU， 而不是多核

在多核 CPU 服务器上面，Redis 的性能还依赖NUMA 配置和处理器绑定位置。最明显的影响是 redis-benchmark 会随机使用CPU内核。为了获得精准的结果，需要使用固定处理器工具（在 Linux 上可以使用 taskset）。最有效的办法是将客户端和服务端分离到两个不同的 CPU 来高校使用三级缓存。

#### Memcache与Redis的区别都有哪些？

1)、存储方式 Memecache把数据全部存在内存之中，断电后会挂掉，数据不能超过内存大小。 Redis有部份存在硬盘上，redis可以持久化其数据
2)、数据支持类型 memcached所有的值均是简单的字符串，redis作为其替代者，支持更为丰富的数据类型 ，提供list，set，zset，hash等数据结构的存储
3)、使用底层模型不同 它们之间底层实现方式 以及与客户端之间通信的应用协议不一样。 Redis直接自己构建了VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求。
4). value 值大小不同：Redis 最大可以达到 512M；memcache 只有 1mb。
5）redis的速度比memcached快很多
6）Redis支持数据的备份，即master-slave模式的数据备份。

### redis的数据类型，以及每种数据类型的使用场景

回答：一共五种
(一)String
这个其实没啥好说的，最常规的set/get操作，value可以是String也可以是数字。一般做一些复杂的计数功能的缓存。
(二)hash
这里value存放的是结构化的对象，比较方便的就是操作其中的某个字段。博主在做单点登录的时候，就是用这种数据结构存储用户信息，以cookieId作为key，设置30分钟为缓存过期时间，能很好的模拟出类似session的效果。
(三)list
使用List的数据结构，可以做简单的消息队列的功能。另外还有一个就是，可以利用lrange命令，做基于redis的分页功能，性能极佳，用户体验好。本人还用一个场景，很合适—取行情信息。就也是个生产者和消费者的场景。LIST可以很好的完成排队，先进先出的原则。
(四)set
因为set堆放的是一堆不重复值的集合。所以可以做全局去重的功能。为什么不用JVM自带的Set进行去重？因为我们的系统一般都是集群部署，使用JVM自带的Set，比较麻烦，难道为了一个做一个全局去重，再起一个公共服务，太麻烦了。
另外，就是利用交集、并集、差集等操作，可以计算共同喜好，全部的喜好，自己独有的喜好等功能。
(五)sorted set（zset）
sorted set多了一个权重参数score,集合中的元素能够按score进行排列。可以做排行榜应用，取TOP N操作。

## Redis 内部结构

- dict：是一个用于维护key和value映射关系的数据结构，与很多语言中的Map或dictionary类似。 本质上是为了解决算法中的查找问题
- sds： 等同于char * 它可以存储任意二进制数据，不能像C语言字符串那样以字符’\0’来标识字符串的结 束，因此它必然有个长度字段。
- skiplist： （跳跃表） 跳表是一种实现起来很简单，单层多指针的链表，它查找效率很高，堪比优化过的二叉平衡树
- quicklist：采用双向链表sdlist和压缩表ziplist来实现快速列表quicklist的，其中sdlist充当map中控器的作用，ziplist充当占用连续内存空间数组的作用。quicklist本身是一个双向无环链表，它的每一个节点都是一个ziplist。
- ziplist： 压缩表 ziplist是一个编码后的列表，是由一系列特殊编码的连续内存块组成的顺序型数据结构，

## redis的过期策略和配置以及内存淘汰机制

redis采用的是**定期删除+惰性删除策略**。
为什么不用定时删除策略?
定时删除,用一个定时器来负责监视key,过期则自动删除。虽然内存及时释放，但是十分消耗CPU资源。在大并发请求下，CPU要将时间应用在处理请求，而不是删除key,因此没有采用这一策略.
**定期删除+惰性删除是如何工作的呢?**
定期删除，redis默认每个100ms检查，是否有过期的key,有过期key则删除。需要说明的是，redis不是每个100ms将所有的key检查一次，而是随机抽取进行检查(如果每隔100ms,全部key进行检查，redis岂不是卡死)。因此，如果只采用定期删除策略，会导致很多key到时间没有删除。
于是，惰性删除派上用场。也就是说在你获取某个key的时候，redis会检查一下，这个key如果设置了过期时间判断是否过期了？如果过期了此时就会删除。
采用定期删除+惰性删除就没其他问题了么?
不是的，如果定期删除没删除key。然后你也没即时去请求key，也就是说惰性删除也没生效。这样，redis的内存会越来越高。那么就应该采用内存淘汰机制。
在redis.conf中有一行配置内存淘汰策略

```
maxmemory-policy volatile-lru
```

该配置就是配内存淘汰策略的（6种）
**volatile-lru**：最近最少使用，从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
**volatile-ttl**：淘汰即将过期，从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
**volatile-random**：随机淘汰，从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
**allkeys-lru**：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰
**allkeys-random**：从数据集（server.db[i].dict）中任意选择数据淘汰
**no-enviction**（不淘汰）：禁止驱逐数据，新写入操作会报错
ps：如果没有设置 expire 的key, 不满足先决条件(prerequisites); 那么 volatile-lru, volatile-random 和 volatile-ttl 策略的行为, 和 noeviction(不删除) 基本上一致。

使用策略规则：

（1）如果数据呈现幂律分布，也就是一部分数据访问频率高，一部分数据访问频率低，则使用 allkeys-lru

（2）如果数据呈现平等分布，也就是所有的数据访问频率都相同，则使用allkeys-random

### Redis 集群方案应该怎么做？都有哪些方案？

1.twemproxy，大概概念是，它类似于一个代理方式， 使用时在本需要连接 redis 的地方改为连接 twemproxy， 它会以一个代理的身份接收请求并使用一致性 hash 算法，将请求转接到具体 redis，将结果再返回 twemproxy。
缺点： twemproxy 自身单端口实例的压力，使用一致性 hash 后，对 redis 节点数量改变时候的计算值的改变，数据无法自动移动到新的节点。

2.codis，目前用的最多的集群方案，基本和 twemproxy 一致的效果，但它支持在 节点数量改变情况下，旧节点数据可恢复到新 hash 节点

3.redis cluster3.0 自带的集群，特点在于他的分布式算法不是一致性 hash，而是 hash 槽的概念，以及自身支持节点设置从节点

### 有没有尝试进行多机redis 的部署？如何保证数据一致的？

主从复制，读写分离
一类是主数据库（master）一类是从数据库（slave），主数据库可以进行读写操作，当发生写操作的时候自动将数据同步到从数据库，而从数据库一般是只读的，并接收主数据库同步过来的数据，一个主数据库可以有多个从数据库，而一个从数据库只能有一个主数据库。

### 对于大量的请求怎么样处理

redis是一个单线程程序，也就说同一时刻它只能处理一个客户端请求；
对于大量请求，redis是通过IO多路复用（select，epoll, kqueue，依据不同的平台，采取不同的实现）来处理多个客户端请求的

### Redis 常见性能问题和解决方案？

(1) Master 最好不要做任何持久化工作，如 RDB 内存快照和 AOF 日志文件
(2) 如果数据比较重要，某个 Slave 开启 AOF 备份数据，策略设置为每秒同步一次
(3) 为了主从复制的速度和连接的稳定性， Master 和 Slave 最好在同一个局域网内
(4) 尽量避免在压力很大的主库上增加从库
（5）主从复制不要用图状结构，用单向链表结构更为稳定，即：Master <- Slave1<- Slave2 <- Slave3…这样的结构方便解决单点故障问题，实现 Slave 对 Master的替换。如果 Master 挂了，可以立刻启用 Slave1 做 Master，其他不变。

### Redis实现分布式锁

Redis为单进程单线程模式，采用队列模式将并发访问变成串行访问，且多客户端对Redis的连接并不存在竞争关系Redis中可以使用SETNX命令实现分布式锁。
将 key 的值设为 value ，当且仅当 key 不存在。 若给定的 key 已经存在，则 SETNX 不做任何动作
![image-20200916132948763](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916132948763.png)
解锁：使用 del key 命令就能释放锁
解决死锁：
1）通过Redis中expire()给锁设定最大持有时间，如果超过，则Redis来帮我们释放锁。
2） 使用 setnx key “当前系统时间+锁持有的时间”和getset key “当前系统时间+锁持有的时间”组合的命令就可以实现。

### Redis事务

Redis事务功能是通过MULTI、EXEC、DISCARD和WATCH 四个原语实现的
Redis会将一个事务中的所有命令序列化，然后按顺序执行。
1.redis 不支持回滚“Redis 在事务失败时不进行回滚，而是继续执行余下的命令”， 所以 Redis 的内部可以保持简单且快速。
2.如果在一个事务中的**命令**出现错误，那么**所有的命令**都不会执行；
3.如果在一个事务中出现**运行错误**，那么**正确的命令**会被执行。
注：redis的discard只是结束本次事务,正确命令造成的影响仍然存在.

1）MULTI命令用于开启一个事务，它总是返回OK。 MULTI执行之后，客户端可以继续向服务器发送任意多条命令，这些命令不会立即被执行，而是被放到一个队列中，当EXEC命令被调用时，所有队列中的命令才会被执行。
2）EXEC：执行所有事务块内的命令。返回事务块内所有命令的返回值，按命令执行的先后顺序排列。 当操作被打断时，返回空值 nil 。
3）通过调用DISCARD，客户端可以清空事务队列，并放弃执行事务， 并且客户端会从事务状态中退出。
4）WATCH 命令可以为 Redis 事务提供 check-and-set （CAS）行为。 可以监控一个或多个键，一旦其中有一个键被修改（或删除），之后的事务就不会执行，监控一直持续到EXEC命令。

### 为什么Redis的操作是原子性的，怎么保证原子性的？

对于Redis而言，命令的原子性指的是：一个操作的不可以再分，操作要么执行，要么不执行。
Redis的操作之所以是原子性的，是因为Redis是单线程的。
Redis本身提供的所有API都是原子操作，Redis中的事务其实是要保证批量操作的原子性。
多个命令在并发中也是原子性的吗？
不一定， 将get和set改成单命令操作，incr 。使用Redis的事务，或者使用Redis+Lua 的方式实现.

### 讲解下Redis线程模型

文件事件处理器包括分别是**套接字、 I/O 多路复用程序、 文件事件分派器（dispatcher）、 以及事件处理器**。使用 I/O 多路复用程序来同时监听多个套接字， 并根据套接字目前执行的任务来为套接字关联不同的事件处理器。当被监听的套接字准备好执行连接应答（accept）、读取（read）、写入（write）、关闭（close）等操作时， 与操作相对应的文件事件就会产生， 这时文件事件处理器就会调用套接字之前关联好的事件处理器来处理这些事件。
I/O 多路复用程序负责监听多个套接字， 并向文件事件分派器传送那些产生了事件的套接字。
**工作原理：**
1)I/O 多路复用程序负责监听多个套接字， 并向文件事件分派器传送那些产生了事件的套接字。
尽管多个文件事件可能会并发地出现， 但 I/O 多路复用程序总是会将所有产生事件的套接字都入队到一个队列里面， 然后通过这个队列， 以有序（sequentially）、同步（synchronously）、每次一个套接字的方式向文件事件分派器传送套接字： 当上一个套接字产生的事件被处理完毕之后（该套接字为事件所关联的事件处理器执行完毕）， I/O 多路复用程序才会继续向文件事件分派器传送下一个套接字。如果一个套接字又可读又可写的话， 那么服务器将先读套接字， 后写套接字.
![image-20200916132956250](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916132956250.png)

### 使用 Redis 有哪些好处？

（1）速度快，因为数据存在内存中，类似于 HashMap，HashMap 的优势就是查找和操作的时间复杂度都是 O1)

（2）支持丰富数据类型，支持 string，list，set，Zset，hash 等

（3）支持事务，操作都是原子性，所谓的原子性就是对数据的更改要么全部执行，要么全部不执行

（4）丰富的特性：可用于缓存，消息，按 key 设置过期时间，过期后将会自动删除

### Redis 相比 Memcached 有哪些优势？

（1）Memcached 所有的值均是简单的字符串，redis 作为其替代者，支持更为丰富的数据类

（2）Redis 的速度比 Memcached 快很多

（3）Redis 可以持久化其数据

### redis 过期键的删除策略？

（1）定时删除:在设置键的过期时间的同时，创建一个定时器 timer，让定时器在键的过期时间一到，立即执行对键的删除操作。

（2）惰性删除:放任键过期不管，但是每次从键空间中获取键时，都检查取得的键是否过期，如果过期的话，就删除该键;如果没有过期，就返回该键。

（3）定期删除:每隔一段时间程序就对数据库进行一次检查，删除里面的过期键。至于要删除多少过期键，以及要检查多少个数据库，则由算法决定。

### 为什么 Redis 需要把所有数据放到内存中？

答 ：Redis 为了达到最快的读写速度将数据都读到内存中，并通过异步的方式将数据写入磁盘。所以 redis 具有快速和数据持久化的特征。如果不将数据放在内存中，磁盘 I/O 速度会严重影响 redis 的性能。如果设置了最大使用的内存，则数据已有记录数达到内存限值后不能继续插入新值。

### Redis 的同步机制了解么？

答：Redis 可以使用主从同步，从从同步。第一次同步时，主节点做一次 **bgsave（另开子进程执行）**，并同时将后续修改操作记录到内存 **buffer**，待完成后将 rdb 文件全量同步到**复制节点**，复制节点接受完成后将 rdb 镜像加载到内存。加载完成后，再通知主节点将期间修改的操作记录从内存buffer，同步到复制节点进行重放就完成了同步过程。

#### Pipeline 有什么好处，为什么要用 pipeline？

答：**可以将多次 IO 往返的时间缩减为一次**，前提是 pipeline 执行的指令之间没有因果相关性。使用 redis-benchmark 进行压测的时候可以发现影响 redis 的 QPS峰值的一个重要因素是 pipeline 批次指令的数目。

### 是否使用过 Redis 集群，集群的原理是什么？

（1）Redis Sentinal 着眼于高可用，在 master 宕机时会自动将 slave 提升为master，继续提供服务。

（2）Redis Cluster 着眼于扩展性，在单个 redis 内存不足时，使用 Cluster 进行分片存储。

### Redis 集群方案什么情况下会导致整个集群不可用？

答：有 A，B，C 三个节点的集群,在没有复制模型的情况下,如果节点 B 失败了，那么整个集群就会以为缺少 5501-11000 这个范围的槽而不可用。

### Redis 支持的 Java 客户端都有哪些？官方推荐用哪个？

答：Redisson、Jedis、lettuce 等等，官方推荐使用 Redisson。

Jedis 是 Redis 的 Java 实现的客户端，其 API 提供了比较全面的 Redis 命令的支持；Redisson 实现了分布式和可扩展的 Java 数据结构，和 Jedis 相比，功能较为简单，不支持字符串操作，不支持排序、事务、管道、分区等 Redis 特性。

Redisson 的宗旨是促进使用者对 Redis 的关注分离，从而让使用者能够将精力更集中地放在处理业务逻辑上。

### 说说 Redis 哈希槽的概念？

答：Redis 集群没有使用一致性 hash,而是引入了哈希槽的概念，Redis 集群有16384 个哈希槽，每个 key 通过 CRC16 校验后对 16384 取模来决定放置哪个槽，集群的每个节点负责一部分 hash 槽。

### Redis 集群的主从复制模型是怎样的？

答：为了使在部分节点失败或者大部分节点无法通信的情况下集群仍然可用，所以集群使用了主从复制模型,每个节点都会有 N-1 个复制品

### Redis 集群会有写操作丢失吗？为什么？

答 ：Redis 并不能保证数据的强一致性，这意味这在实际中集群在特定的条件下可能会丢失写操作。

### Redis 集群之间是如何复制的？

答：异步复制

### Redis 集群最大节点个数是多少？

答：16384 个桶(哈希槽)。

Redis集群没有使用一致性hash,而是引入了哈希槽的概念，当需要在 Redis 集群中放置一个 key-value 时，根据 CRC16(key) mod 16384的值，决定将一个key放到哪个桶中。

### Redis key 的过期时间和永久有效分别怎么设置？

答：EXPIRE 和 PERSIST 命令。

### Redis 如何做内存优化？

答：尽可能使用散列表（hashes），散列表（是说散列表里面存储的数少）使用的内存非常小，所以你应该尽可能的将你的数据模型抽象到一个散列表里面。比如你的 web 系统中有一个用户对象，不要为这个用户的名称，姓氏，邮箱，密码设置单独的 key,而是应该把这个用户的所有信息存储到一张散列表里面。

### Redis 回收进程如何工作的？

答：一个客户端运行了新的命令，添加了新的数据。Redi 检查内存使用情况，如果大于 **maxmemory** 的限制, 则根据设定好的策略进行回收。一个新的命令被执行，等等。所以我们不断地穿越内存限制的边界，通过不断达到边界然后不断地回收回到边界以下。如果一个命令的结果导致大量内存被使用（例如很大的集合的交集保存到一个新的键），不用多久内存限制就会被这个内存使用量超越。

### 都有哪些办法可以降低 Redis 的内存使用情况呢？

答：如果你使用的是 32 位的 Redis 实例，可以好好利用 Hash,list,sorted set,set等集合类型数据，因为通常情况下很多小的 Key-Value 可以用更紧凑的方式存放到一起。

### Redis 的内存用完了会发生什么？

答：如果达到设置的上限，Redis 的写命令会返回错误信息（但是读命令还可以正常返回。）或者你可以将 Redis 当缓存来使用配置淘汰机制，当 Redis 达到内存上限时会冲刷掉旧的内容。

### 一个 Redis 实例最多能存放多少keys？List、Set、Sorted Set 最多能存放多少元素？

答：理论上 Redis 可以处理多达 232 的 keys，并且在实际中进行了测试，每个实例至少存放了 2 亿 5 千万的 keys。任何 list、set、和 sorted set 都可以放 232 个元素。换句话说，Redis 的存储极限是系统中的可用内存值。

### MySQL 里有2000w数据，redis 中只存 20w 的数据，如何保证 redis 中的数据都是热点数据？

答：Redis 内存数据集大小上升到一定大小的时候，就会执行数据淘汰策略。

Redis 提供 6 种数据淘汰策略：

volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰

volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰

volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰

allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰

allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰

no-enviction（驱逐）：禁止驱逐数据

### Redis 最适合的场景？

1、会话缓存（Session Cache）

最常用的一种使用 Redis 的情景是会话缓存（session cache）。用 Redis 缓存会话比其他存储（如 Memcached）的优势在于：Redis 提供持久化。当维护一个不是严格要求一致性的缓存时，如果用户的购物车信息全部丢失，大部分人都会不高兴的，现在，他们还会这样吗？ 幸运的是，随着 Redis 这些年的改进，很容易找到怎么恰当的使用 Redis 来缓存会话的文档。甚至广为人知的商业平台Magento 也提供 Redis 的插件。

2、全页缓存（FPC）

除基本的会话 token 之外，Redis 还提供很简便的 FPC 平台。回到一致性问题，即使重启了 Redis 实例，因为有磁盘的持久化，用户也不会看到页面加载速度的下降，这是一个极大改进，类似 PHP 本地 FPC。 再次以 Magento 为例，Magento提供一个插件来使用 Redis 作为全页缓存后端。 此外，对 WordPress 的用户来说，Pantheon 有一个非常好的插件 wp-redis，这个插件能帮助你以最快速度加载你曾浏览过的页面。

3、队列

Reids 在内存存储引擎领域的一大优点是提供 list 和 set 操作，这使得 Redis能作为一个很好的消息队列平台来使用。Redis 作为队列使用的操作，就类似于本地程序语言（如 Python）对 list 的 push/pop 操作。 如果你快速的在 Google中搜索“Redis queues”，你马上就能找到大量的开源项目，这些项目的目的就是利用 Redis 创建非常好的后端工具，以满足各种队列需求。例如，Celery 有一个后台就是使用 Redis 作为 broker，你可以从这里去查看。

4，排行榜/计数器

Redis 在内存中对数字进行递增或递减的操作实现的非常好。集合（Set）和有序集合（Sorted Set）也使得我们在执行这些操作的时候变的非常简单，Redis 只是正好提供了这两种数据结构。所以，我们要从排序集合中获取到排名最靠前的 10个用户–我们称之为“user_scores”，我们只需要像下面一样执行即可： 当然，这是假定你是根据你用户的分数做递增的排序。如果你想返回用户及用户的分数，你需要这样执行： ZRANGE user_scores 0 10 WITHSCORES Agora Games 就是一个很好的例子，用 Ruby 实现的，它的排行榜就是使用 Redis 来存储数据的，你可以在这里看到。

5、发布/订阅

Redis 的发布/订阅功能。发布/订阅的使用场景确实非常多。我已看见人们在社交网络连接中使用，还可作为基于发布/订阅的脚本触发器，甚至用 Redis 的发布/订阅功能来建立聊天系统！

### 假如 Redis 里面有 1 亿个 key，其中有 10w 个 key 是以某个固定的已知的前缀开头的，如何将它们全部找出来？

答：使用 keys 指令可以扫出指定模式的 key 列表。

对方接着追问：如果这个 redis 正在给线上的业务提供服务，那使用 keys 指令会有什么问题？

这个时候你要回答 redis 关键的一个特性：redis 的单线程的。keys 指令会导致线程阻塞一段时间，线上服务会停顿，直到指令执行完毕，服务才能恢复。这个时候可以使用 scan 指令，scan 指令可以无阻塞的提取出指定模式的 key 列表，但是会有一定的重复概率，在客户端做一次去重就可以了，但是整体所花费的时间会比直接用 keys 指令长。

### 如果有大量的 key 需要设置同一时间过期，一般需要注意什么？

答：如果大量的 key 过期时间设置的过于集中，到过期的那个时间点，redis 可能会出现短暂的卡顿现象。一般需要在时间上加一个随机值，使得过期时间分散一些。

### 使用过 Redis 做异步队列么，你是怎么用的？

答：一般使用 list 结构作为队列，rpush 生产消息，lpop 消费消息。当 lpop 没有消息的时候，要适当 sleep 一会再重试。如果对方追问可不可以不用 sleep 呢？list 还有个指令叫 blpop，在没有消息的时候，它会阻塞住直到消息到来。如果对方追问能不能生产一次消费多次呢？使用 pub/sub 主题订阅者模式，可以实现1:N 的消息队列。

如果对方追问 pub/sub 有什么缺点？

在消费者下线的情况下，生产的消息会丢失，得使用专业的消息队列如 RabbitMQ等。

如果对方追问 redis 如何实现延时队列？

##### 使用 zset，拿时间戳作为score，消息内容作为 key 调用 zadd 来生产消息，消费者用 zrangebyscore 指令获取 N 秒之前的数据轮询进行处理

### Redis 分布式锁

先拿 setnx 来争抢锁，抢到之后，再用 expire 给锁加一个过期时间防止锁忘记了释放。

然后接着问如果在 setnx 之后执行 expire之前进程意外 crash 或者要重启维护了，那会怎么样？

 set 指令有非常复杂的参数，这个应该是可以同时把 setnx 和expire 合成一条指令来用的

### Redis回收进程如何工作的？

一个客户端运行了新的命令，添加了新的数据。
Redi检查内存使用情况，如果大于maxmemory的限制, 则根据设定好的策略进行回收。回收使用的算法：LRU算法

### Redis常见性能问题和解决方案？

(1) Master最好不要做任何持久化工作，如RDB内存快照和AOF日志文件
(2) 如果数据比较重要，某个Slave开启AOF备份数据，策略设置为每秒同步一次
(3) 为了主从复制的速度和连接的稳定性，Master和Slave最好在同一个局域网内
(4) 尽量避免在压力很大的主库上增加从库
(5) 主从复制不要用图状结构，用单向链表结构更为稳定，即：Master <- Slave1 <- Slave2 <- Slave3...
这样的结构方便解决单点故障问题，实现Slave对Master的替换。如果Master挂了，可以立刻启用Slave1做Master，其他不变。

### Redis数据分片模型

为了解决读写分离模型的缺陷，可以将数据分片模型应用进来。
可以将每个节点看成都是独立的master，然后通过业务实现数据分片。
结合上面两种模型，可以将每个master设计成由一个master和多个slave组成的模型。

### Redis读写分离模型

通过增加Slave DB的数量，读的性能可以线性增长。为了避免Master DB的单点故障，集群一般都会采用两台Master DB做双机热备，所以整个集群的读和写的可用性都非常高。

读写分离架构的缺陷在于，不管是Master还是Slave，每个节点都必须保存完整的数据，如果在数据量很大的情况下，集群的扩展能力还是受限于单个节点的存储能力，而且对于Write-intensive类型的应用，读写分离架构并不适合。

## Redis集群方案应该怎么做？都有哪些方案？

1.twemproxy
2.codis，目前用的最多的集群方案，基本和twemproxy一致的效果，但它支持在 节点数量改变情况下，旧节点数据可恢复到新hash节点。
3.Redis cluster3.0自带的集，特点在于他的分布式算法不是一致性hash，而是hash槽的概念，以及自身支持节点设置从节点。

## Redis单点吞吐量

单点TPS达到8万/秒，QPS达到10万/秒，补充下TPS和QPS的概念

1.QPS: 应用系统每秒钟最大能接受的用户访问量
每秒钟处理完请求的次数，注意这里是处理完，具体是指发出请求到服务器处理完成功返回结果。可以理解在server中有个counter，每处理一个请求加1，1秒后counter=QPS。
2.TPS： 每秒钟最大能处理的请求数
每秒钟处理完的事务次数，一个应用系统1s能完成多少事务处理，一个事务在分布式处理中，可能会对应多个请求，对于衡量单个接口服务的处理能力，用QPS比较合理。

### Redis相比memcached有哪些优势？

1.memcached所有的值均是简单的字符串，Redis作为其替代者，支持更为丰富的数据类型
2.Redis的速度比memcached快很多
3.Redis可以持久化其数据
4.Redis支持数据的备份，即master-slave模式的数据备份。















