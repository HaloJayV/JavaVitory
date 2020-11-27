

### UNION

现在表的数据量有 800万。

一般的使用语句是：ELECT * FROM A WHERE c_a IN(955555, 955556, 955557, 955558, 955559);

上面语句会执行的很快，知道使用 explain 的都明白这样一般都是会使用索引的，并且是所有范围扫描。

MySQL不会从 1 开始 扫描 800万，而是从555555 扫描到 555559(只要扫描5行数据)。

在一般情况下是没有什么问题的。但是如果 IN 里面的数据是不连续的就有很大问题了。

使用 in 查询不连续的数

```sql
SELECT *
FROM t
WHERE cid IN(1, 5000, 50000, 500000, 955559);
+---------+--------+-----------------------------------
| id      | cid    | c1                              
+---------+--------+-----------------------------------
|      1 |  341702 |      1 | aaaaaaaaaaaaaaaaaaaaaaaaa
|      1 | 1045176 |      1 | aaaaaaaaaaaaaaaaaaaaaaaaa
......
| 955559 | 8733757 | 955559 | aaaaaaaaaaaaaaaaaaaaaaaaa
| 955559 | 8796305 | 955559 | aaaaaaaaaaaaaaaaaaaaaaaaa
+--------+---------+--------+--------------------------
41 rows in set (4.34 sec)
```

使用 UNION 优化

原理是使用UNION生成一个临时表作为关联的主表

```sql
SELECT *
FROM (
    SELECT 1 AS cid UNION ALL
    SELECT 5000 UNION ALL
    SELECT 50000 UNION ALL
    SELECT 500000 UNION ALL
    SELECT 955559
) AS tmp, t
WHERE tmp.cid = t.cid;
+---------+--------+-----------------------------------
| id      | cid    | c1                              
+---------+--------+-----------------------------------
|      1 |  341702 |      1 | aaaaaaaaaaaaaaaaaaaaaaaaa
|      1 | 1045176 |      1 | aaaaaaaaaaaaaaaaaaaaaaaaa
......
| 955559 | 8733757 | 955559 | aaaaaaaaaaaaaaaaa	aaaaaaaa
| 955559 | 8796305 | 955559 | aaaaaaaaaaaaaaaaaaaaaaaaa
+--------+---------+--------+--------------------------
41 rows in set (0.01 sec)
```

### SELECT * 

```
SELECT COUNT(*) FROM SomeTable
SELECT COUNT(1) FROM SomeTable
```

以上SQL语句在MySQL5.6之前会造成全表扫描，实际上针对无 where  的 **COUNT(\*)**，MySQL 在5.6 版本之后是有优化的，优化器会选择成本最小的辅助索引查询计数，其实反而性能最高

通过explain，发现Extra是Using index，表明使用了覆盖索引，即只需要在一棵索引树上就能获取SQL所需的所有列数据，无需回表，即使用了聚集索引

#### 原理

在 InnoDB 存储引擎中，count(*) 函数是先内存中读取数据到内存缓冲区，然后进行扫描获得行记录数。这里 InnoDB 会优先走二级索引；如果同时存在多个二级索引，会选择key_len 最小的二级索引；如果不存在二级索引（普通索引），那么会走主键索引；如果连主键都不存在，那么就走全表扫描！

因此对于数据量巨大的情况，如果走的是主键索引，所以 MySQL 需要先把整个主键索引读取到内存缓冲区，这是个从磁盘读写到内存的过程，而且主键索引基本等于整个表数据量（10GB+），所以非常耗时！解决方法便是使用建立并使用指定使用二级缓存

因为二级索引只包含对应的索引列及主键列，所以体积非常小。在 select count(*) 的查询过程中，只需要将二级索引读取到内存缓冲区，只有几十 MB 的数据量，所以速度会非常快。

举个形象的比喻，我们想知道一本书的页数：

- 走聚集索引：从第一页翻到最后一页，知道总页数；
- 走二级索引：通过目录直接知道总页数。

如果 select count(*) 走的是主键索引，那么会缓存整个表数据，大量查询时间会花费在读取表数据到缓冲区。

如果存在二级索引，那么只需要读取索引页到缓冲区即可，速度自然快。

#### 优化

避免直接 select count(*) from table 来解决，例如：

1. 使用 MySQL 触发器 + 统计表实时计算表数据量；
2. 使用 MyISAM 替换 InnoDB，因为 MyISAM 自带计数器，坏处就不多说了；
3. 通过 ETL 导入表数据到其他更高效的异构环境中进行计算；
4. 升级到 MySQL 8 中，使用并行查询，加快检索速度。



### explain extra

using index ：使用覆盖索引的时候就会出现

using where：在查找使用索引的情况下，需要回表去查询所需的数据

using index condition：查找使用了索引，但是需要回表查询数据

using index & using where：查找使用了索引，但是需要的数据都在索引列中能找到，所以不需要回表查询数据

### SQL选用索引

在有多个索引的情况下， 在查询数据前，MySQL 会选择成本最小原则来选择使用对应的索引，这里的成本主要包含两个方面。

- IO 成本: 即从磁盘把数据加载到内存的成本，默认情况下，读取数据页的 IO 成本是 1，MySQL 是以页的形式读取数据的，每页16Kb，即当用到某个数据时，并不会只读取这个数据，而会把这个数据相邻的数据也一起读到内存中，这就是有名的程序局部性原理，所以 MySQL 每次会读取一整页，一页的成本就是 1。所以 IO 的成本主要和页的大小有关
- CPU 成本：将数据读入内存后，还要检测数据是否满足条件和排序等 CPU 操作的成本，显然它与行数有关，默认情况下，检测记录的成本是 0.2。

#### 实例

为了根据以上两个成本来算出使用索引的最终成本，我们先准备一个表（以下操作基于 MySQL 5.7.18）

```
CREATE TABLE person (
  id bigint(20) NOT NULL AUTO_INCREMENT,
  name varchar(255) NOT NULL,
  score int(11) NOT NULL,
  create_time timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (id),
  KEY name_score (name(191),score),
  KEY create_time (create_time)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

这个表除了主键索引之外，还有另外两个索引, name_score 及 create_time。然后我们在此表中插入 10 w 行数据，只要写一个存储过程调用即可，如下:

```sql
CREATE PROCEDURE insert_person()
begin
declare c_id integer default 1;
    while c_id<=100000 do
insert into person values(c_id, concat('name',c_id), c_id+100, date_sub(NOW(), interval c_id second));
set c_id=c_id+1;
end while;
end
```

插入之后我们现使用 EXPLAIN 来计算下统计总行数到底使用的是哪个索引

[![我说 SELECT COUNT(*) 会造成全表扫描，面试官让我回去等通知](https://www.iwmyx.cn/wp-content/uploads/2020/05/c1e48dfe842a.png)](https://www.iwmyx.cn/wp-content/uploads/2020/05/c1e48dfe842a.png)

从结果上看它选择了 create_time 辅助索引，显然 MySQL 认为使用此索引进行查询成本最小，这也是符合我们的预期，使用辅助索引来查询确实是性能最高的！

我们再来看以下 SQL 会使用哪个索引

```
SELECT * FROM person WHERE NAME >'name84059' AND create_time>'2020-05-23 14:39:18'
```

type：ALL，即用了全表扫描！理论上应该用 name_score 或者 create_time 索引才对，从 WHERE 的查询条件来看确实都能命中索引，那是否是使用 **SELECT \*** 造成的回表代价太大所致呢，我们改成覆盖索引的形式试一下

```
SELECT create_time FROM person WHERE NAME >'name84059' AND create_time > '2020-05-23 14:39:18'
```

结果 MySQL 依然选择了全表扫描！这就比较有意思了，理论上采用了覆盖索引的方式进行查找性能肯定是比全表扫描更好的，为啥 MySQL 选择了全表扫描呢，既然它认为全表扫描比使用覆盖索引的形式性能更好，那我们分别用这两者执行来比较下查询时间吧

```sql
-- 全表扫描执行时间: 4.0 ms
SELECT create_time FROM person WHERE NAME >'name84059' AND create_time>'2020-05-23 14:39:18'
-- 使用覆盖索引执行时间: 2.0 ms
SELECT create_time FROM person force index(create_time) WHERE NAME >'name84059' AND create_time>'2020-05-23 14:39:18' 
```

从实际执行的效果看使用覆盖索引查询比使用全表扫描执行的时间快了一倍！说明 MySQL 在查询前做的成本估算不准！我们先来看看 MySQL 做全表扫描的成本有多少。

前面我们说了成本主要 IO 成本和 CPU 成本有关，对于全表扫描来说也就是分别和聚簇索引占用的页面数和表中的记录数。执行以下命令

```
SHOW TABLE STATUS LIKE 'person'
```

[![我说 SELECT COUNT(*) 会造成全表扫描，面试官让我回去等通知](https://www.iwmyx.cn/wp-content/uploads/2020/05/34d0020c2171.png)](https://www.iwmyx.cn/wp-content/uploads/2020/05/34d0020c2171.png)

可以发现

1. 行数是 100264，我们不是插入了 10 w 行的数据了吗，怎么算出的数据反而多了，其实这里的计算是**估算**，也有可能这里的行数统计出来比 10 w 少了，估算方式有兴趣大家去网上查找，这里不是本文重点，就不展开了。得知行数，那我们知道 CPU 成本是 100264 * 0.2 = 20052.8。
2. 数据长度是 5783552，InnoDB 每个页面的大小是 16 KB，可以算出页面数量是 353。

也就是说全表扫描的成本是 20052.8 + 353 =  20406。

这个结果对不对呢，我们可以用一个工具验证一下。在 MySQL 5.6 及之后的版本中，我们可以用 optimizer trace 功能来查看优化器生成计划的整个过程 ，它列出了选择每个索引的执行计划成本以及最终的选择结果，我们可以依赖这些信息来进一步优化我们的 SQL。

optimizer_trace 功能使用如下

```
SET optimizer_trace="enabled=on";
SELECT create_time FROM person WHERE NAME >'name84059' AND create_time > '2020-05-23 14:39:18';
SELECT * FROM information_schema.OPTIMIZER_TRACE;
SET optimizer_trace="enabled=off";
```

执行之后我们主要观察使用 name_score，create_time 索引及全表扫描的成本。

先来看下使用 name_score 索引执行的的预估执行成本:

```
{
"index": "name_score",
"ranges": [
"name84059 <= name"
    ],
"index_dives_for_eq_ranges": true,
"rows": 25372,
"cost": 30447
}
```

可以看到执行成本为 30447，高于我们之前算出来的全表扫描成本：20406。所以没选择此索引执行

注意：这里的 30447 是查询二级索引的 IO 成本和 CPU 成本之和，再加上回表查询聚簇索引的 IO 成本和 CPU 成本之和。

再来看下使用 create_time 索引执行的的预估执行成本:

```
{
    "index": "create_time",
    "ranges": [
      "0x5ec8c516 < create_time"
    ],
    "index_dives_for_eq_ranges": true,
    "rows": 50132,
    "cost": 60159,
    "cause": "cost"
}
```

可以看到成本是 60159,远大于全表扫描成本 20406，自然也没选择此索引。

再来看计算出的全表扫描成本：

```
{
    "considered_execution_plans": [
      {
        "plan_prefix": [
        ],
        "table": "`person`",
        "best_access_path": {
          "considered_access_paths": [
            {
              "rows_to_scan": 100264,
              "access_type": "scan",
              "resulting_rows": 100264,
              "cost": 20406,
              "chosen": true
            }
          ]
        },
        "condition_filtering_pct": 100,
        "rows_for_plan": 100264,
        "cost_for_plan": 20406,
        "chosen": true
      }
    ]
}
```

注意看 cost：20406，与我们之前算出来的完全一样！这个值在以上三者算出的执行成本中最小，所以最终 MySQL 选择了用全表扫描的方式来执行此 SQL。

实际上 optimizer trace 详细列出了覆盖索引，回表的成本统计情况，有兴趣的可以去研究一下。

从以上分析可以看出， MySQL 选择的执行计划未必是最佳的，原因有挺多，就比如上文说的行数统计信息不准，再比如 MySQL 认为的最优跟我们认为不一样，我们可以认为执行时间短的是最优的，但 MySQL 认为的成本小未必意味着执行时间短。

## Redis计数器

https://www.cnblogs.com/ShenJunHui6/p/11127737.html

可以使用redis原生方法**INCR key**
将 key 中储存的数字值增一。

如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 INCR 操作。

如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。

**这是一个针对字符串的操作，因为 Redis 没有专用的整数类型，所以 key 内储存的字符串被解释为十进制 64 位有符号整数来执行 INCR 操作。**

### 计数器的实现

计数器是 Redis 的原子性自增操作可实现的最直观的模式了，它的想法相当简单：每当某个操作发生时，向 Redis 发送一个 INCR 命令。

比如在一个 web 应用程序中，如果想知道用户在一年中每天的点击量，那么只要将用户 ID 以及相关的日期信息作为键，并在每次用户点击页面时，执行一次自增操作即可。

比如用户名是 peter ，点击时间是 2012 年 3 月 22 日，那么执行命令 INCR peter::2012.3.22 。

```
$redisKey = “api_name_” + $api;
$count = $this->redis->incr($redisKey);
if ($count == 1) {
    //设置有效期一s
    $this->redis->expire($redisKey,1);//设置过期时间
}
if (count > 200) {//防止刷单的安全拦截
	return false;//超过就返回false
}
//后续处理*
```

这就简单的实现了redis计数器的应用，另外还有以下方法：

可以通过组合使用 INCR 和 EXPIRE ，来达到只在规定的生存时间内进行计数(counting)的目的。
客户端可以通过使用 GETSET 命令原子性地获取计数器的当前值并将计数器清零，更多信息请参考 GETSET 命令。
使用其他自增/自减操作，比如 DECR 和 INCRBY ，用户可以通过执行不同的操作增加或减少计数器的值，比如在游戏中的记分器就可能用到这些命令。

### 应用和优化

https://www.cnblogs.com/ShenJunHui6/p/11127737.html

社交产品业务里有很多统计计数的功能，比如：

- 用户： 总点赞数，关注数，粉丝数
- 帖子： 点赞数，评论数，热度
- 消息： 已读，未读，红点消息数
- 话题： 阅读数，帖子数，收藏数

#### 方案一：hash结构的counter

```
//用户
counter:user:{userID}
                        ->  praiseCnt: 100      //点赞数
                        ->  hostCnt: 200        //热度
                        ->  followCnt: 332      //关注数
                        ->  fansCnt: 123        //粉丝数


//帖子
counter:topic:{topicID}
                        -> praiseCnt: 100       //点赞数
                        -> commentCnt: 322      //评论数


//话题
counter:subject:{subjectID}
                            -> favoCnt: 312     //收藏数
                            -> viewCnt: 321     //阅读数
                            -> searchCnt: 212   //搜索进入次数
                            -> topicCnt: 312    //话题中帖子数 
```

相关命令

```
//获取指定userID的所有计数器
HGETALL counter:user:{userID}   

//获取指定userID的指定计数器
HMGET counter:user:{userID}  praiseCnt hostCnt 

//指定userID点赞数+1
HINCRBY counter:user:{userID}   praiseCnt 
```

缺点:这样设计，如果要批量查询多个用户的数据，就比较麻烦，例如一次要查指定20个userID的计数器？只能循环执行 HGETALL counter:user:{userID}。

优点:以实体聚合数据，方便数据管理

#### 方案二

依然是采用hash

```
counter:user:praiseCnt
                        ->  userID_1001: 100
                        ->  userID_1002: 200
                        ->  userID_1003: 332
                        ->  userID_1004: 123
                                .......
                        ->  userID_9999: 213



counter:user:hostCnt
                        ->  userID_1001: 10
                        ->  userID_1002: 290
                        ->  userID_1003: 322
                        ->  userID_1004: 143
                                .......
                        ->  userID_9999: 213


counter:user:followCnt
                        ->  userID_1001: 21
                        ->  userID_1002: 10
                        ->  userID_1003: 32
                        ->  userID_1004: 203
                                .......
                        ->  userID_9999: 130
```

获取多个指定userID的点赞数的命令变成这样了

```
HMGET counter:user:praiseCnt userID_1001 userID_1002
```

上面命令可以批量获取多个用户的点赞数，时间复杂度为O(n)，n为指定userID的数量。

优点:解决了批量操作的问题

缺点:当要获取多个计数器，比如同时需要praiseCnt，hostCnt时，要读多次，不过要比第一种方案读的次数要少。一个hash里的字段将会非常宠大，HMGET也许会有性能瓶颈。

**用redis管道（Pipelining)来优化方案一**

对于第一种方案的缺点，可以通过redis管道来优化，一次性发送多个命令给redis执行：

```lua
$userIDArray = array(1001, 1002, 1003, 1009);

$pipe = $redis->multi(Redis::PIPELINE);
foreach ($userIDArray as $userID) {
    $pipe->hGetAll('counter:user:' . $userID);
}

$replies = $pipe->exec();
print_r($replies); 
```

还有一种方式是在redis上执行lua脚本，前提是你必须要学会写lua。

### concat

SELECT CONCAT(xxx, xxx) 可将两个字段的查询结果通过字符串连接，并返回连接后的结果











