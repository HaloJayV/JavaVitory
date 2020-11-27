[TOC]

## MySQL基础

* SELECT DISTINCT price FROM product;    // 去重查询
* truncate与delete的异同：
  * truncate是DDL，操作不会进行存储不能进行事务回滚，而delete是DML，会被回滚
  * truncate是删除整个表
  * truncate事务日志少，速度较快，delete则速度慢，但相对安全许多
  * delete数据不会对数据库的查询效率有所改变，而truncate会使数据库容量被重置，DML速度提高
* 修改表名：rename table [xxx] [old] to [new]; / alter table [xxx] rename [new]
* 模糊查询：
  * like "%x%" // 字符匹配     like "x___"  //前缀匹配     in(2,4,6,8) // 包含查询
  * between 1 and 5   // 范围查询
* 排序：  order by xxx asc/desc
* 聚合函数：sum, avg, max, min 
* 外键：

  * 主表：从表 = 一对多
  * alter table 从表 add constraint 外键名 foreign key(从表主键) references 主表名(主表主键);

  * 主表不能删除从表中已经使用的数据，应该先删除从表，再删除主表数据
  * 可以在创建表时添加外键，如 foreign key(iforeign_goodsid) references goods(goodsid);
* 多表查询：

  * 交叉查询(了解): select * from A  表名，B   表名  where  A.aid = B.a_id
  * 内连接查询:
    * 隐式内连接查询：select distinct cname from category c, product p where c.cateid=p.category_id
    * 显示内连接查询：select * from A 表名 inner join B 表名 on category.cateid=p.category_id
  * 外连接查询
  * 子查询：一条select语句的结果作为另一条select语句的一部分，如
    * select * from product where category_id=(select cateid from category where cname='服装')

# Mysql高级

* Linux查看是否已经安装Mysql `rpm -qa|grep -i mysql`   
* 安装Mysql-server ：`rpm -ivh MySQL-server-5.5.48-1.linux2.6.i386.rpm --nodeps`
* 安装MySql-client：`rpm -ivh MySQL-client-5.5.48-1.linux2.6.i386.rpm --nodeps`

* 查看linux用户组：`cat /etc/group|grep mysql`

* 查看服务是否启动： `ps -ef|grep mysql`
* Mysql服务命令： `systemctl start/restart/stop/status  mysqld.service`
* 设置MySQL开机自启动： `systemctl enable mysqld`    `systemctl list-unit-files |grep mysqld`

* 修改Mysql登录密码：`ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';`

* Mysql配置文件![image-20200711000848948](../../../../Software/Typora/Picture/image-20200711000848948.png)

* **Mysql逻辑架构：连接层—服务层—引擎层—存储层**

* ![image-20200916132501542](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916132501542.png)

* **插件式的存储引擎架构将查询处理和其它系统任务以及数据的存储提取相分离。这种架构可以根据业务需求和实际需要选择合适的存储引擎**

* 从5.5开始Mysql默认使用表(插件式)存储引擎 InnoDB ， InnoDB **支持事务、支持外键**，MyISAM 不支持

  * 1. InnoDB 支持外键，而 MyISAM 不支持。对一个包含外键的 InnoDB 表转为 MYISAM 会失败；  
2. InnoDB 是聚集索引，MyISAM 是非聚集索引。聚簇索引的文件存放在主键索引的叶子节点上，因此 InnoDB 必须要有主键，通过主键索引效率很高。但是辅助索引需要两次查询，先查询到主键，然后再通过主键查询到数据。因此，主键不应该过大，因为主键太大，其他索引也都会很大。而 MyISAM 是非聚集索引，数据文件是分离的，索引保存的是数据文件的指针。主键索引和辅助索引是独立的。 
    3. InnoDB 不保存表的具体行数，执行 select count(*) from table 时需要全表扫描。而MyISAM **用一个变量保存了整个表的行数**，执行上述语句时只需要读出该变量即可，速度很快
4. InnoDB 最小的锁粒度是**行锁**，MyISAM 最小的锁粒度是**表锁**。一个更新语句会锁住整张表，导致其他查询和更新都会被阻塞，因此并发访问受限。这也是 MySQL 将默认存储引擎从 MyISAM 变成 InnoDB 的重要原因之一；



![image-20200916132507983](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916132507983.png)

grant all privileges on *.* to 'root'@'% 'identified by 'xx' with  grant option;

#### 性能优化分析

* 创建索引：如  `create index idx_user_nameEmail on user(name, email);`  

* 查询效率低的原因：索引失效(单值、复合)、关联查询太多Join、服务器调优及各个参数设置（缓冲、线程数等）

#### Join查询

* SQL执行顺序：对于Mysql ，计算机读取SQL语句时，先从FROM开始

![image-20200916132511655](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916132511655.png)

​      ![image-20200916132514636](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916132514636.png)

* 7种Join关系

  * 从左到右从上到下分别为：左外连接、右外连接、内连接、左连接、右连接、全外连接、两表独有的数据集连接

  * 在Mysql种，全外连接和两表独有的数据集连接用法应为：

  * ```
    SELECT * FROM TableA A LEFT JOIN TableB B ON A.Key = B.Key
    	UNION
    	SELECT * FROM TableA A RIGHT JOIN TableB B ON A.Key = B.Key;
    	
    SELECT * FROM TableA A LEFT JOIN TableB B ON A.Key = B.Key WHERE B.Key is null;
    	UNION
    	SELECT * FROM TableA A RIGHT JOIN TableB B ON A.Key = B.Key WHERE A.Key is null;	
    ```

    

![image-20200916132524121](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916132524121.png)

#### 事务管理ACID

* 原子性（Atomicity）
  原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。
* 一致性（Consistency）
  事务前后数据的完整性必须保持一致。
* 隔离性（Isolation）
  事务的隔离性是多个用户并发访问数据库时，数据库为每一个用户开启的事务，不能被其他事务的操作数据所干扰，多个并发事务之间要相互隔离。
* 持久性（Durability）
  持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来即使数据库发生故障也不应该对其有任何影响

#### Mysql配置文件

* 配置文件是：/etc/my.cnf 文件，在windows里是在my.ini

* 二进制日志 log-bin：主从复制
* 错误日志log-error：存放错误信息，默认关闭。每次启动和关闭的详细信息等
* 查询日志log：默认关闭，记录查询的sql语句，如果开启会降低整体性能，因为记录日志会消耗资源
* 数据文件：
  * linux:  /var/lib/mysql
  * windows:  D:\devsoft\MySQLServer5.5\data
* frm文件：存放表结构
* myd文件：存放表数据
* myi文件：存放表索引

# 索引

.Mysql对索引的定义：索引是帮助Mysql高效获取数据的数据结构，可以得到索引的本质：索引是数据结构。类比字典，目的是提高查询效率

* 索引：排好序的快速查找数据结构。索引会影响到sql中 WHERE后面的条件过滤查找 和 ORDER BY 后面的排序。
* ![image-20200916132532643](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916132532643.png)
* 数据库维护着一个满足特定查找算法的数据结构，这些结构以某种方式指向数据，这样就可以在这些数据结构的基础上实现高级查找算法。这种数据结构就算索引
* 逻辑删除（惰性删除）：delete方法时最终底层都是调用update方法将数据的标志位从激活状态1变为非激活状态0。这样做一是为了保存数据浏览记录，为了记录分析。二是为了索引。
* 数据修改慢的原因：一是数据在改，二是索引也要改，需要重新指向正确的位置。所以经常修改的数据不适合建立索引。
* 一般来说索引本身也很大，不可能全部存储在内存中，因此索引往往以索引文件的形式存储在磁盘上
* 平常所说的索引如果没有特别说明，都是指 B 树（多路搜索树，不一定是二叉树）结构组织的索引。其中聚集索引、次要索引、覆盖索引、复合索引、前缀索引、唯一索引，默认都是使用B+树索引，统称索引。除了B+树这种类型的索引之外，还有哈希索引等。

* 索引优势：提高检索效率，降低数据库IO成本；降低数据排序成本，降低了CPU的消耗
* 劣势：索引列要占空间；降低更新表的速度

#### 索引分类

* 单值索引：一个索引只包含单个列，一个表可以有一个单列索引

* 唯一索引：索引列的值必须唯一，但允许有空值

* 复合索引：唯一索引包含多个列 

* 基本语法（最好使用复合索引。一张表一般不能超过5个索引）

  * 创建：CREATE [UNIQUE] INDEX indexName ON mytable(columnName(length)); 

    ​			或 ALTER mytable ADD [UNIQUE] IDNEX [indexName] ON (columnName(length))

  * 删除：DROP INDEX [indexName] ON mytable;

  * 查看：SHOW INDEX FROM table_name\G  

* 有四种方式来添加数据表的索引：
  ALTER TABLE tbl_name ADD PRIMARY KEY (column_list): 该语句添加一个主键，这意味着索引值必须是唯一的，且不能为NULL。

  ALTER TABLE tbl_name ADD UNIQUE index_name (column_list): 这条语句创建索引的值必须是唯一的（除了NULL外，NULL可能会出现多次）。

  ALTER TABLE tbl_name ADD INDEX index_name (column_list): 添加普通索引，索引值可出现多次。

  ALTER TABLE tbl_name ADD FULLTEXT index_name (column_list):该语句指定了索引为 FULLTEXT ，用于全文索引。


#### 索引结构

* Mysql索引结构有4种：BTree、Hash、full-text全文索引、R-Tree索引。Java开发只有BTree索引
* 真实的数据存在于叶子节点即3、5、9、10、13、15、28、29、36、60、75、79、90、99。
  非叶子节点不存储真实的数据，只存储指引搜索方向的数据项，如17、35并不真实存在于数据表中。
  * ![image-20200916132537577](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916132537577.png)

* 哪些情况需要创建索引：

  * 1.主键自动建议为唯一索引
  * 2.频繁作为查询条件的字段应该创建索引
  * 3.查询种与其他表关联的字段，外键关系建立索引

  * 4.单键/复合索引的选择问题，在高并发下倾向创建组合索引
  * 5.排序字段若通过索引去访问将大大提高排序速度
  * 6.查询中统计或者分组字段

* 不适合创建索引的情况：
  * 表记录太少，记录数在300万左右性能开始下降
  * 经常增删改的表
  * where条件里用不到的字段不创建索引
  * 某个数据列包含太多重复内容，建立索引后实际效果不大
    * ![image-20200916132540795](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916132540795.png)

#### 性能分析基础知识

* Mysql常见瓶颈：CPU、IO、锁、硬件性能：top、free、iostat、vmstat 可以查看系统的性能状态
* Mysql Query Optimizer：Mysql自带的查询优化器
* Explain：使用EXPLANE关键字可以模拟优化器执行SQL查询语句，分析该查询语句或是表结构的性能瓶颈

#### explane

* 使用EXPLANE关键字可以模拟优化器执行SQL查询语句，分析该查询语句或是表结构的性能瓶颈
* **用处：**
  * 表的读取顺序：由 id 决定，小表永远驱动大表
  * 数据读取操作的操作类型
  * 哪些索引可以使用
  * 哪些索引被实际使用
  * 表之间的引用
  * 每张表有多少行被优化器查询（rows ）

* 用法：explain + SQL语句
  * ![image-20200916132544615](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916132544615.png)
* **字段包含的信息**
  * id：select序列号，表示执行select子句操作表顺序。
    * id相同，执行顺序由上至下。
    * id不同，如果是子查询，id的序号会递增，**id值越大优先级越高，越先被执行。**
    * id如果相同，可以认为是一组，从上往下顺序执行；在所有组中，id值越大，优先级越高，越先执行
  * select_type：查询的类型，主要用于区别普通查询、联合查询、子查询等复杂查询
    * ![image-20200916132547844](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916132547844.png)
    * ![image-20200916132551460](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916132551460.png)
  * table：显示这一行是关于那张表的，其中derived2表示衍生表，可以理解为子查询后的虚表，2表示来自select_type为DERIVED的 id
  * type：显示查询使用了何种类型
    * ALL（全表扫描）、index（全索引扫描）、range（如between）、ref (非唯一性索引扫描，如姓名)、eq_ref（唯一索引扫描，如主键id）、const（如id）、system、NULL    
    * 最好到最差：system > const > eq_ref > ref > range > index > ALL
    * ![image-20200916132557001](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916132557001.png)
  * possible_keys :
    * 显示可能应用在这张表的索引，查询涉及到的字段上若存在索引，则该索引被列出。但不一定被查询的会实际使用到
  * key :
    * 实际使用的索引，如果为NULL表示没有索引。查询中若使用了覆盖索引，则该索引仅出现在key列表中
    * ![image-20200916132603319](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916132603319.png)
  * key_len :
    * 表示索引使用的字节数，通过该列计算查询中使用的索引长度。在不损失精确性前提下，长度越短越好
    * 显示的值为索引字段的最大可能长度，并非实际使用长度。即key_len是根据表定义计算而得，不是通过表内检索出的
  * ref :
    * 显示索引的哪一列被使用了，哪些列或常量被用于查找索引列上的值。如果可能，最好是常数const
    * ![image-20200716140230709](../../../../Software/Typora/Picture/image-20200716140230709.png)
  * rows :
    * 根据表统计信息及索引选用情况，大致估算出找到记录所需要读取的行数。行数越少优化越好
  * Extra :
    * 包含不合适在其他列中显示但十分重要的额外信息
    * ![image-20200716171017811](../../../../Software/Typora/Picture/image-20200716171017811.png)
    * 前三个最重要
    * ![image-20200716145318281](../../../../Software/Typora/Picture/image-20200716145318281.png)
    * 出现Using filesort、Using temporary时最好立即处理掉
    * 索引最好的情况：创建的索引 col 字段，和查的 col 字段个数和顺序刚好一致
    * 覆盖索引：①一个索引 ②包含了(或覆盖了)[select子句]与查询条件[Where子句]中 ③所有需要的字段就叫做覆盖索引。

* 案例1：单表查询优化
  * 如下图，索引顺序是从上到下
  * ![image-20200716182142326](../../../../Software/Typora/Picture/image-20200716182142326.png)
  * 索引失效：range类型查询字段后面的索引无效。解决方法可以是：range类型的字段不要建立索引，这样range后面的SQL语句能生效
  * ![image-20200716210032414](../../../../Software/Typora/Picture/image-20200716210032414.png)

* 案例2：多表查询索引优化

  * EXPLAIN SELECT * FROM class LEFT JOIN book ON class.card = book.card;
    type 变为了 ref,rows 也变成了优化比较明显。这是由左连接特性决定的。LEFT JOIN 条件用于确定如何从右表搜索行,左边一定都有, 所以右边是我们的关键点,一定需要建立索引。

### 索引优化

![image-20200722114702364](../../../../Software/Typora/Picture/image-20200722114702364.png)



* 1.最佳左前缀法则：
  * 如果索引了多列，要遵守最左前缀法则。即查询从索引的最左前列开始并且不跳过索引中的任何一列（索引的带头字段必须要有，否则失效）。中间索引列也不能断，否则只能让前部分索引有效。最好的做法是使用索引时，索引列从头开始按顺序不中断。

![image-20200722112515618](../../../../Software/Typora/Picture/image-20200722112515618.png)

* 即使顺序不对，也无影响，只要不出现断层，因为底层有optimizer进行优化，优化后查询的字段按照索引排序，符合最左前缀法则，AND不分先后，但最好按照索引顺序查询

![image-20200722204826718](../../../../Software/Typora/Picture/image-20200722204826718.png)

* 2.不能在索引列上做任何操作（计算、函数、（自动or手动）类型转换），否则会导致索引失效而转向全表扫描

* 如：`EXPLAIN SELECT * FROM staffs WHERE left(NAME,4) = 'July';`    将变为全表扫描

* 3.存储引擎不能使用索引中范围条件右边的列（范围之后全失效）

  ![image-20200722123158459](../../../../Software/Typora/Picture/image-20200722123158459.png)

* 4.尽量使用覆盖索引（只访问索引的查询 ( 索引列和查询列一致 ) ) ，减少使用 select *

  ![image-20200722133619941](../../../../Software/Typora/Picture/image-20200722133619941.png)

* 5.mysql在使用不等于 ( != 或者 <>) 时无法使用索引，会导致全表扫描

  * ![image-20200722134331532](../../../../Software/Typora/Picture/image-20200722134331532.png)

* 6.  is null 和 is not null 都无法使用索引，如：

  ![image-20200722145553437](../../../../Software/Typora/Picture/image-20200722145553437.png)

* 7.like以通配符开头，mysql索引失效会变成全表扫描的操作

  * like ‘%abc’ 和 like ‘%abc%’  type 类型会变成 all 
    like ‘abc%’ type 类型为 range ，算是范围，可以使用索引。之后的索引列也有效
  * ![image-20200722155620097](../../../../Software/Typora/Picture/image-20200722155620097.png)
  * 解决like ‘%abc%’ 时索引不被使用的方法：使用覆盖索引：`EXPLAIN SELECT id,NAME FROM tbl_user WHERE NAME LIKE '%aa%';`结果type都是index

  

* 8.字符串不加单引号索引会失败

![image-20200722155640328](../../../../Software/Typora/Picture/image-20200722155640328.png)



* 9.。少用or，用or来连接时会索引失效

![image-20200722155653840](../../../../Software/Typora/Picture/image-20200722155653840.png)



* 总结
  * 假设index(a,b,c)
    Where语句	索引是否被使用
    where a = 3	Y,使用到a
    where a = 3 and b = 5	Y,使用到a，b
    where a = 3 and b = 5 and c = 4	Y,使用到a,b,c
    where b = 3 或者 where b = 3 and c = 4  或者 where c = 4	N
    where a = 3 and c = 5	使用到a， 但是c不可以，b中间断了
    where a = 3 and b > 4 and c = 5	使用到a和b， c不能用在范围之后，b后断了
    **where a = 3 and b like 'kk%' and c = 4	Y,使用到a,b,c**
    where a = 3 and b like '%kk' and c = 4	Y,只用到a
    where a = 3 and b like '%kk%' and c = 4	Y,只用到a
    **where a = 3 and b like 'k%kk%' and c = 4	Y,使用到a,b,c**

![image-20200722235906221](../../../../Software/Typora/Picture/image-20200722235906221.png)

典型题目：create index idx_test03_c1234 on test03(c1,c2,c3,c4);

* 1）explain select * from test03 where c1='a1' and c5='a5' order by c2,c3; 

  只用c1一个字段索引，但是c2、c3用于排序,无filesort

*  2）explain select * from test03 where c1='a1' and c5='a5' order by c3,c2;

   出现了filesort，我们建的索引是1234，它没有按照顺序来，3 2 颠倒了，因此出现filesort， 只用c1一个字段索引

* 3）
  explain select * from test03 where c1='a1' and c2='a2' and c5='a5' order by c2,c3;       

  explain select * from test03 where c1='a1' and c2='a2' and c5='a5' order by c3,c2;      

  用c1、c2两个字段索引，但是c2、c3用于排序, 与题目 2）不同，无filesort，因为此时c2已经是固定常数，不需要排序，只有c3需要排序

* 4） explain select * from test03 where c1='a1' and c4='a4' group by c2,c3;     只有c1作为索引

  ​	 explain select * from test03 where c1='a1' and c4='a4' group by c3,c2;			Using where; Using temporary; Using filesort 

  ​	定值、范围还是排序，一般order by 是给个范围

  ​	group by 基本上都需要进行排序，会有临时表产生

* 5)  explain select * from test03 where c1='a1' and c2='a2' order by c4;      出现了filesort

#### 子查询EXISTS 和 IN 

* ![image-20200724113844671](../../../../Software/Typora/Picture/image-20200724113844671.png)
* EXISTS：将主查询的数据，放到子查询中做条件验证，根据验证结果（TRUE或FALSE）来决定主查询的数据结果是否得以保留

#### ORDER BY

* ![image-20200725110134775](../../../../Software/Typora/Picture/image-20200725110134775.png)

* ![image-20200725110148606](../../../../Software/Typora/Picture/image-20200725110148606.png)

* Mysql支持2种方式的排序，FileSort和Index，Index效率高，指Mysql扫描索引本身完成排序，FileSort效率低 

* ORDER BY语句使用索引最左前列

* 使用where子句与order by子句条件列组合满足索引最左前列

* ![image-20200725112144475](../../../../Software/Typora/Picture/image-20200725112144475.png)

  * 第二种中，where a = const and b > const order by b , c 不会出现 using filesort  b , c 两个衔接上了
    但是：where a = const and b > const order by  c 将会出现 using filesort 。因为 b 用了范围索引，断了；

    而上一个  order by 后的b 用到了索引，所以能衔接上 c 

* Using filesort两种算法：

  * 双路排序：Mysql4.1之前使用双路排序，扫描两次磁盘

  * 单路排序：磁盘读取查询需要的所有列，在内存中按照 order by列在bufe对它们进行排序，然后扫描排序后的列表进行输出。

    避免了第二次读取数据。并且把随机O变成了顺序，但是它会使用更多的空间，因为它把每一行都保存在内存中了

* 问题：

* 单路排序在内存进行排序时，有可能取出的数据的总大小超出了sort_buffer的容量，导致每次只能取sort_buffer容量大小的数据，进行排序（创建tmp文件，多路合并），排完再取取sort_buffer容量大小，再排……从而多次I/O。

* 优化策略：

  * 增大sort_buffer_size参数的设置：用于单路排序的内存大小
  * 增大max_length_for_sort_data参数的设置：单次排序字段大小。(单次排序请求)
  * 尽量不用select * ：select 后的多了，排序的时候也会带着一起，很占内存

#### GROUP BY

* where 高于having，能写where限定的条件就不去having限定
* 其余的与ORDER BY一样



### 查询截取分析

* 1. 慢查询 ( 设置查询时间阈值并抓取慢sql ) 的开启并捕获
  2. explain + 慢SQL分析
  3. show profile 查询SQL在Mysql服务器里面执行细节和生命周期情况
  4. SQL数据库服务器的参数调优

####  慢查询日志 

* 开启与设置

  * 默认情况下slow_query_log的值为OFF，表示慢查询日志是禁用的，
    可以通过设置slow_query_log的值来开启：SHOW VARIABLES LIKE '%slow_query_log%';

  * set global slow_query_log=1;  可以开启慢查询日志。只对当前数据库生效，如果MySQL重启后则会失效。

  * 如果要永久生效，就必须修改配置文件my.cnf（其它系统变量也是如此）。修改my.cnf文件，将如下两行配置进my.cnf文件然后重启MySQL服务器

    slow_query_log =1
    slow_query_log_file=/var/lib/mysql/atguigu-slow.log。系统默认会给一个缺省的文件host_name-slow.log（如果没有指定参数slow_query_log_file的话）

  * 关于慢查询的参数slow_query_log_file ，它指定慢查询日志文件的存放路径，系统默认会给一个缺省的文件host_name-slow.log（如果没有指定参数slow_query_log_file的话）

* long_query_time慢查询时间

  * SHOW VARIABLES LIKE 'long_query_time%';  查看允许最长查询时间，默认为10秒，即大于10秒的查询为慢查询
  * set global long_query_time=1
  * 设置完需要重新连接一个会话才能看到修改成功，命令：SHOW VARIABLES LIKE 'long_query_time%'; 

* 日志分析工具：mysqldumpslow

  * ![image-20200725155704400](../../../../Software/Typora/Picture/image-20200725155704400.png)

  * 工作中常用命令

    ```
    得到返回记录集最多的10个SQL
    mysqldumpslow -s r -t 10 /var/lib/mysql/atguigu-slow.log
    
    得到访问次数最多的10个SQL
    mysqldumpslow -s c -t 10 /var/lib/mysql/atguigu-slow.log
    
    得到按照时间排序的前10条里面含有左连接的查询语句
    mysqldumpslow -s t -t 10 -g "left join" /var/lib/mysql/atguigu-slow.log
    
    另外建议在使用这些命令时结合 | 和more 使用 ，否则有可能出现爆屏情况
    mysqldumpslow -s r -t 10 /var/lib/mysql/atguigu-slow.log | more
    ```

#### 批量插入数据脚本

* 创建函数，假如报错：This function has none of DETERMINISTIC......

  ##### 由于开启过慢查询日志，因为我们开启了 bin-log, 我们就必须为我们的function指定一个参数。

  show variables like 'log_bin_trust_function_creators';

  set global log_bin_trust_function_creators=1;

  ##### 这样添加了参数以后，如果mysqld重启，上述参数又会消失，永久方法：

  windows下my.ini[mysqld]加上log_bin_trust_function_creators=1 

  linux下    /etc/my.cnf下my.cnf[mysqld]加上log_bin_trust_function_creators=1

* 创建存储过程，可一次性执行多次insert

```mysql
DELIMITER $$			## '$$'是为了区分分号，作为结束符。参数：START为起始编号no，max_num为插入的个数
CREATE PROCEDURE insert_emp(IN START INT(10),IN max_num INT(10))  
BEGIN  
    DECLARE i INT DEFAULT 0;   
    #set autocommit =0 把autocommit设置成0，关闭自动提交；提高执行效率
     SET autocommit = 0;    
     REPEAT  ##重复循环
         SET i = i + 1;  
         INSERT INTO emp10000 (empno, ename ,job ,mgr ,hiredate ,sal ,comm ,deptno ) VALUES ((START+i) ,rand_string(6),'SALESMAN',0001,CURDATE(),FLOOR(1+RAND()*20000),FLOOR(1+RAND()*1000),rand_num());  
         UNTIL i = max_num   ## 循环一定次数，上面也是一个循环
     END REPEAT;  ##满足条件后结束循环
     COMMIT;   ##执行完成后一起提交
END $$  ## 创建存储结构完成
 
## 调用存储过程
DELIMITER ;
CALL CALL insert_emp(0,500000);  # 插入50万数据

 
#删除
# DELIMITER ;
# drop PROCEDURE insert_emp;
```



#### ShowProfile

* 是mysql提供可以用来分析当前会话中语句执行的资源消耗情况， 可以用于SQL的调优的测量

* 调优四层结构：连接、服务、引擎、存储

* 默认情况下，参数处于关闭状态，并保存最近15次的运行结果

* show variables like 'profiling';

  默认是关闭，使用前需开启：set profiling=1;    

* 查看被抓取的sql： show profiles

* 诊断SQL， show profile cpu, block io for query n（n为show profiles中所展示的SQL的ID）

* 开发需要诊断的问题：（出现就需要解决）

  * ![image-20200725180825038](../../../../Software/Typora/Picture/image-20200725180825038.png)

#### 全局查询日志

* 只能再测试环境开启这个功能

* 配置启用：

  * 在mysql的my.cnf中，设置如下：
    general_log=1     #开启

    general_log_file=/path/logfile   # 记录日志文件的路径
    log_output=FILE   #输出格式：文件

* 编码启用：

  * set global general_log=1;   #全局日志可以存放到日志文件中，也可以存放到Mysql系统表中。

    存放到日志中性能更好一些，存储到表中  ：set global log_output='TABLE';


     此后 ，编写的sql语句将会记录到mysql库里的general_log表，可以用下面的命令查看
    
    select * from mysql.general_log;

# Mysql锁机制

* 从数据操作的粒度上，分为表锁（偏读）和行锁。从类型上分为读锁（共享锁）和写锁（排它锁）

#### 表锁分析：

* 偏向MyISAM存储引擎，开销小，加锁块，无死锁，锁定粒度大，发生锁冲突的概率最高，并发度最低，
* show open tables：查看整个Mysql中所有数据库的表锁情况
* 手动添加表锁：lock table 表名 read(write), 表名2 read(write);

* unlock tables ：释放所有表锁

* 当加上表读锁时，
  * 当前session只能查询该表记录，不能修改当前表，且不能查询其它没有锁定的表。
  * 其他session能查询，但修改该表时，会阻塞直到该session释放锁
* 当加上表写锁时，
  * 当前session对锁定表的查询+更新+插入操作都可以执行：不能查询其它表
  * 其他session对锁定表的查询被阻塞，需要等待锁被释放：
  * 在锁表前，如果session2有数据缓存，锁表以后，在锁住的表不发生改变的情况下session2可以读出缓存数据，一旦数据发生改变，缓存将失效，操作将被阻塞住。
* MyISAM在执行查询语句（SELECT）前，会自动给涉及的所有表加读锁，在执行增删改操作前，会自动给涉及的表加写锁。 
* 表锁分析：
  *  通过检查table_locks_waited 和 table_locks_immediate状态变量俩分析系统上的表锁定：show status like ‘table%’;    
  *  table_locks_waited：产生表级锁定的次数，表示可以立即获取锁的查询次数，每立即获取锁值加1
  *  (关键) table_locks_immediate：出现表级锁定争用而发生等待的次数（不能立即获取锁的次数，每等待一次锁值加1），此值高则说明存在着较严重的表级锁争用情况。
  *  Myisam的读写锁调度是写优先，因此myisam不适合做写为主表的引擎，因为写锁后，其他线程不能做任何操作，大量更新使查询很难得到锁，造成阻塞

* 简而言之，就是读锁会阻塞写，但是不会堵塞读。而写锁则会把读和写都堵塞

#### 行锁分析

* 偏向InnoDB存储引擎，开销大加锁慢，会出现死锁。锁定粒度最小，发生锁冲突概率最低，并发度最高
* InnoDB与MyISAM不同点：（1）InnoDB支持事务（2）InnoDB采用行级锁，而MyISAM采用表级锁
* 事务ADCI属性：
  * l 原子性（Atomicity）：事务是一个原子操作单元，其对数据的修改，要么全都执行，要么全都不执行。 
    l 一致性（Consistent）：在事务开始和完成时，数据都必须保持一致状态。这意味着所有相关的数据规则都必须应用于事务的修改，以保持数据的完整性；事务结束时，所有的内部数据结构（如B树索引或双向链表）也都必须是正确的。 
    l 隔离性（Isolation）：数据库系统提供一定的隔离机制，保证事务在不受外部并发操作影响的“独立”环境执行。这意味着事务处理过程中的中间状态对外部是不可见的，反之亦然。 
    l 持久性（Durable）：事务完成之后，它对于数据的修改是永久性的，即使出现系统故障也能够保持。
* 并发事务处理带来的问题：
  * 更新丢失：最后的更新覆盖了由其他事务所做的更新。
  * 脏读：事务A读取到了事务B已修改但尚未提交的的数据
  * 不可重复读：一个事务范围内两个相同的查询却返回了不同数据。
  * 幻读：事务A 读取到了事务B提交的新增数据，不符合隔离性。（幻读是读到新增数据，脏读读到的是修改数据）
* 隔离级别：

![image-20200726124903882](../../../../Software/Typora/Picture/image-20200726124903882.png)

​		常看当前数据库的事务隔离级别：show variables like 'tx_isolation';   mysql默认是可重复读（默认在修改事务commit，其他事务自动commit）

* 当事务A修改某一行时，事务B如果也修改这一行，后修改的事务会阻塞直到修改的事务commit；若事务B修改不同行将不会阻塞。

* 如果索引失效，行锁变表锁，例如varchar类型没有加单引号导致索引自动类型转换从而索引失效

* select xxx …. for update：写时锁定某一行，其他对该行的操作会被阻塞，直到锁定行的会话提交 

* 检查InnoDB_row_lock状态变量：

  * **show status like 'innodb_row_lock%'** ： 通过检查InnoDB_row_lock状态变量来分析系统上的行锁的争夺情况，对各个状态量的说明如下：

    Innodb_row_lock_current_waits：当前正在等待锁定的数量；（重要）
    Innodb_row_lock_time：从系统启动到现在锁定总时间长度；（重要）
    Innodb_row_lock_time_avg：每次等待所花平均时间；
    Innodb_row_lock_time_max：从系统启动到现在等待最常的一次所花的时间；
    Innodb_row_lock_waits：系统启动后到现在总共等待的次数；（重要）

* 优化：![image-20200726153514523](../../../../Software/Typora/Picture/image-20200726153514523.png)

* 页锁：锁定粒度、开销和加锁时间介于表锁和行锁之间，会出现死锁，并发度一般（了解）

#### 间隙锁危害

* 当用范围条件而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据记录的索引项加锁；对于键值在条件范围内但并不存在的记录，叫做“间隙（GAP)”，InnoDB也会对这个“间隙”**加锁**，这种锁机制就是所谓的间隙锁（GAP Lock）。
* 因为Query执行过程中通过范围查找的话，他会锁定整个范围内所有的索引键值，即使这个键值并不存在。
* 间隙锁有致命的弱点，就是当锁定一个范围键值之后，即使某些不存在的键值也会被无辜的锁定，而造成在锁定的时候无法插入锁定键值范围内的任何数据。



# 主从复制

* slave会从master读取binlog进行数据同步

#### 三步骤+原理

* 1 master将改变记录到二进制日志（binary log）。这些记录过程叫做二进制日志事件，binary log events；
  2 slave将master的binary log events拷贝到它的中继日志（relay log）；
  3 slave重做中继日志中的事件，将改变应用到自己的数据库中。 MySQL复制是异步的且串行化的

![image-20200726172651677](../../../../Software/Typora/Picture/image-20200726172651677.png)



#### 复制的基本原则

* 每个slave只有一个master，只能有一个唯一服务器ID
* 每个master可以有多个slave

## 一主一从常见配置

* mysql版本一致且后台以服务运行，主从都配置在【mysqld】节点下。都是小写
* 主从机配置文件修改后都需要重启mysql服务
* 主从机都需要关闭防火墙，centos7：systemctl stop firewalld.service

#### （windows）主机配置（windows配置文件my.ini，linux配置文件my.cnf）

![image-20200726174612996](../../../../Software/Typora/Picture/image-20200726174612996.png)

#### 从机配置（linux）

* 【必须】从服务器唯一ID  ：server-id=2
* 【可选】启用二进制日志 ： log-bin=mysql-bin

#### 主机建立账户并授权slave

* `GRANT REPLICATION SLAVE ON *.* TO ‘账户名’ @ ‘从库IP’ INENTIFIED BY ‘账户密码’;` 
* flush privileges;   授权后刷新
* show master status      查询主机状态
* ![image-20200726182639923](../../../../Software/Typora/Picture/image-20200726182639923.png)
  * File：二进制日志文件
  * Position：从二进制日志文件的第几行开始复制 
  * Binlog_Do_DB：如果为空则表示需要复制主机的所有数据库
  * Binlog_Ignore_DB：标注哪些数据库不复制

#### 从机配置需要复制的主机

* 连接主机并复制

```sql
CHANGE MASTER TO MASTER_HOST='主机IP',
MASTER_USER='账户名',
MASTER_PASSWORD='账户密码',
MASTER_LOG_FILE='mysqlbin.具体数字',MASTER_LOG_POS=具体值;
```

* start slave    ： 启动从服务器的复制功能

* show slave status\G  ： 查看从机复制状态：若 

  * Slave_IO_Running: Yes
  * Slave_SQL_Running: Yes 

  同时为yes才表示主从配置成功

* stop slave  :  停止从服务复制功能，但需要注意的是，如果要重新连接主机并复制，需要先在主机查看主机状态，根据新的Position重新连接主机并复制

# MySQL事务日志

## redo log

- 用来保证事务的持久性，即用于数据库的崩溃恢复，是innodb特有的
- redolog是持久化在磁盘上的日志文件，记录的是对物理磁盘上数据的修改。
- 当数据发生修改时，innodb引擎会将记录写到redo log文件中，并更新内存，此时更新就算完成了，同时innodb引擎会在合适的时机记录操作到磁盘中。
- redo log 是固定大小的，是循环写过程。
- 有了redo log之后，innodb 就可以保证数据库即使宕机、发生异常，之前的记录也不会丢失。

简单来说，MySQL修改内存中的数据后，并不会立即写入到磁盘中进行持久化，而是随机或者按一定规律进行持久化，这就可能导致内存中的数据在未写入磁盘进行持久化前，如果发生异常故障，将会导致内存中的数据丢失。此时redo日志就可以解决这个问题，将redo里的修改操作持久化到MySQL中

## undo log

- 用于事务回滚
- undo log用来实现事务的原子性，在innnodb引擎中还用来实现事务的多版本并发控制
- 在操作数据库之前先将数据备份到一个地方，这个存储数据备份的地方就是undo log。然后再进行数据的修改，如果中途发生错误或者用户执行的rollback语句，系统利用undo log中的备份将数据回复到事务开始前的状态
- redo log 是逻辑日志，可以理解为
  - 当delete一条语句时，undo log 中会记录一条对应的insert语句
  - 当insert一条语句时，undo log 中会记录一条对应的delect语句
  - 当update一条语句时，undo log 中会记录一条相反的update语句

通俗来讲，MySQL从磁盘中读取数据到内存，对内存中的数据进行修改后，undo会 log保存修改前的数据。如果之后用户对数据进行rollback回滚，undo log就能将修改之前的旧数据写到磁盘中

## binlog

- binlog是所有索引引擎都可以使用的，是server层的日志，主要做mysql功能层的事情
- binlog会记录所有的逻辑，并采用追加写的方式。而redo log是采用循环写的方式，空间不够会覆盖之前的信息
- 一般企业中数据库会有备份系统，可以定期执行备份
- 利用binlog进行数据恢复
  1. 找到最近一次的全量备份数据
  2. 从备份的时间点将binlog取出来，重放到要恢复的时刻



# 面试

## MySQL动态扩容方案

### 目前可用方案

- MySQL的复制：
  - 一个Master数据库，多个Salve，然后利用MySQL的异步复制能力实现读写分离，这个方案目前应用比较广泛，这种技术对于以读为主的应用很有效。
- 数据切分（MySQL的Sharding策略）：
  - 垂直切分：一种是按照不同的表（或者Schema）来切分到不同的数据库（主机）之上，这种切可以称之为数据的垂直（纵向）切分；垂直切分的思路就是分析表间的聚合关系，把关系紧密的表放在一起。
  - 水平切分：另外一种则是根据表中的数据的逻辑关系，将同一个表中的数据按照某种条件拆分到多台数据库（主机）上面，这种切分称之为数据的水平（横向）切分。
- 通过集群扩展：MySQL Cluster(NDB Cluster)
  - 类似于MongoDB的动态扩容策略。
  - MySQL Cluster是一套具备可扩展能力、实时、内存内且符合ACID要求的事务型数据库，其将99.999%高可用性与低廉的开源总体拥有成本相结合。在设计思路方面，MySQL Cluster采用一套分布式多主架构并借此彻底消灭了单点故障问题。MySQL Cluster能够横向扩展至商用硬件之上，能够通过自动分区以承载读取与写入敏感型工作负载，并可通过SQL与NoSQL接口实现访问。
  - 采用NDB存储引擎，有数据节点，SQL节点，和管理节点（1个，配置要求低）
- 分库分表分区
- 开源解决方案
  - mycat:
    - 它是一个开源的分布式数据库系统，是一个实现了MySQL协议的服务器，前端用户可以把它看作是一个数据库代理，用MySQL客户端工具和命令行访问，而其后端可以用MySQL原生协议与多个MySQL服务器通信，也可以用JDBC协议与大多数主流数据库服务器通信，其核心功能是分表分库，即将一个大表水平分割为N个小表，存储在后端MySQL服务器里或者其他数据库里。
    - MyCAT的前身，是阿里巴巴于2012年6月19日，正式对外开源的数据库中间件Cobar，Cobar的前身是早已经开源的Amoeba，不过其作者陈思儒离职去盛大之后，阿里巴巴内部考虑到Amoeba的稳定性、性能和功能支持，以及其他因素，重新设立了一个项目组并且更换名称为Cobar。Cobar是由 Alibaba 开源的 MySQL 分布式处理中间件，它可以在分布式的环境下看上去像传统数据库一样提供海量数据服务。 
      Cobar自诞生之日起， 就受到广大程序员的追捧，但是自2013年后，几乎没有后续更新。在此情况下，MyCAT应运而生，它基于阿里开源的Cobar产品而研发，Cobar的稳定性、可靠性、优秀的架构和性能，以及众多成熟的使用案例使得MyCAT一开始就拥有一个很好的起点，站在巨人的肩膀上，MyCAT能看到更远。
    - 做分库分表，读写分离，不能动态扩展增加机器
  - cobar
    - Cobar 是由 Alibaba 开源的 MySQL 分布式处理中间件，它可以在分布式的环境下看上去像传统数据库一样提供海量数据服务
    - 提供分库分表，主备切换
  - MySQL Router 
    - 提供应用与任意 MySQL 服务器后端的透明路由
    - 可以快速实现一个简单的带有读写分离的高可用集群。写可进行主备的自动切换，实现高可用，读可提供类似于LVS形式的负载均衡。
  - MySQL Fabric
    - fabric是“织物”的意思，这意味着它是用来“织”起一片MySQL数据库。MySQL Fabric是一套数据库服务器场(Database Server Farm)的架构管理系统。
    - MySQL Fabric能“组织”多个MySQL数据库，是应用系统将大于几TB的表分散到多个数据库，即数据分片(Data Shard)。在同一个分片内又可以含有多个数据库，并且由Fabric自动挑选一个适合的作为主数据库，其他的数据库配置成从数据库，来做主从复制。在主数据库挂掉时，从各个从数据库中挑选一个提升为主数据库。之后，其他的从数据库转向新的主数据库复制新的数据。
    - [Fabric实现学习笔记](http://www.tuicool.com/articles/miuq63E)
  - Atlas
    - Qihoo 360公司Web平台部基础架构团队开发维护的一个基于MySQL协议的数据中间层项目。它在MySQL官方推出的MySQL-Proxy 0.8.2版本的基础上，修改了大量bug，添加了很多功能特性。
    - 读写分离， 从库负载均衡， 静态分
  - [谈谈MySQL水平扩展](http://note.youdao.com/)
  - [关于MySQL的在线扩容](http://www.cnblogs.com/netfocus/p/3582358.html)

### 高可用方案

- MMM 
  - MMM（Master-Master replication manager for MySQL） 是一套支持双主故障切换和双主日常管理的脚本程序。MMM使用Perl语言开发，主要用来监控和管理MySQL Master-Master（双主）复制，虽然叫做双主复制，但是业务上同一时刻只允许对一个主进行写入，另一台备选主上提供部分读服务，以加速在主主切换时刻备选主的预热，可以说MMM这套脚本程序一方面实现了故障切换的功能，另一方面其内部附加的工具脚本也可以实现多个slave的read负载均衡。
  - 由于MMM无法完全的保证数据一致性，所以MMM适用于对数据的一致性要求不是很高，但是又想最大程度的保证业务可用性的场景。对于那些对数据的一致性要求很高的业务，非常不建议采用MMM这种高可用架构
  - [MySQL 高可用架构之MMM](http://www.cnblogs.com/gomysql/p/3671896.html)
- MHA 
  - MHA（Master High Availability）目前在MySQL高可用方面是一个相对成熟的解决方案，它由日本DeNA 公司youshimaton（现就职于Facebook公司）开发，是一套优秀的作为MySQL高可用性环境下故障切换和主从提升的高可用软件。在MySQL故障切换过程中，MHA能做到在0~30秒之内自动完成数据库的故障切换操作，并且在进行故障切换的过程中，MHA能在最大程度上保证数据的一致性，以达到真正意义上的高可用。
- Percona XTRADB Cluste 
  - Percona XtraDB Cluster是MySQL高可用性和可扩展性的解决方案
  - 相当于多主同时读写

### 主从同步

![image-20200922182452759](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200922182452759.png)





[



.]