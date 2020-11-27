[TOC]

## InnoDB两大类索引

- 聚集索引(clustered index) 也叫聚簇索引
- 普通索引(secondary index)

简单来说，通常主键为聚集索引，其他属性为普通索引

#### InnoDB规定每个表都必须有聚集索引且只有一个：

1. 如果定义了主键，那么主键就是聚集索引
2. 如果没有定义，第一个非空 unique列就是聚集索引
3. 如果再没有，会自动生成一个隐藏的row-id作为聚集索引

## 结构

**索引的结构为B+树**

1. 聚集索引：非叶子节点存储key，叶子节点存储**行记录**
2. 普通索引：非叶子节点存储key，叶子节点存储**value值** + **主键值** （不是存储行记录头指针！MyISAM的索引叶子节点存储记录指针。）
3. 索引包含null：放在B+树的最左边

![image-20200916132258779](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916132258779.png)

### **回表查询**：

**普通索引无法直接定位行记录，所以对于普通索引，需要二次查找，先通过普通索引找到聚集索引，再通过聚集索引找到行记录。**

##### 先定位主键值，再定位行记录，它的性能较扫一遍索引树更低。



## 索引覆盖

在explain查询计划优化章节，即explain的输出结果Extra字段为**Using index**时，能够触发索引覆盖。

简单来讲，只需要在一棵索引树上就能获取SQL所需的所有列数据，无需回表，速度更快。

也就是说，根据索引查询时能一次性命中要查询的数据就是索引覆盖，无需会表再查一次，提高效率，此时explain后出现Extra：**Using index**



### 索引覆盖的实现

*select id,name from user where name='xxx';*      （*Extra：**Using index**。*）

能够命中name索引，索引叶子节点存储了主键id，通过name的索引树即可获取id和name，无需回表，符合索引覆盖，效率较高。

*select id,name**,sex** from user where name='shenjian';*      （Extra：**Using index condition**）

能够命中name索引，索引叶子节点存储了主键id，但sex字段必须回表查询才能获取到，不符合索引覆盖，需要再次通过id值扫码聚集索引获取sex字段，效率会降低。

此时如果使用联合索引 {index(name, sex)} ，就能避免回表查询：  *Extra：**Using index**。*

```sql
create table user (
    id int primary key,
    name varchar(20),
    sex varchar(5),
    index(name, sex)
)engine=innodb;
```

### limit、offset

```sql
//从表的第2条开始，读10条数据
select * from C limit 10 offset 1
//等价于
select * from C limit 1,10
```

如果要从第100000… 开始读数据，那么offset会特别大，必然慢SQL，可以通过以下方法解决：

* 延迟加载

* 使用联合索引 index(sex, score)

  ```
  //方式一：
  select <col> from A where sex = 'M' order by score limit 100000,10
  ```

* 使用覆盖索引：先从覆盖索引中获取100010个id，在丢掉前100000条id，保留最后10个；

  ```
  //方式二：
  select <col> from A inner join(select id from A where a.sex = 'M' order by score limit 100000,10) as a using(id)
  ```

## 索引覆盖优化SQL

* MySQL5.6之后对覆盖索引做了进一步的优化，支持索引下推的功能，把覆盖索引所覆盖的字段进一步筛选，尽量减少回表的次数，此时为 （Extra：**Using index condition**）。

* 如果使用的是机械硬盘的话，要进一步进行优化，随机硬盘很怕随机读写，有一个磁盘寻址的开销，可以打开Multi range read，可以在回表之前把ID读到buffer里，进行排序，把原来的一个随机操作变成一个顺序操作。

* 覆盖索引可以避免比如在排序时用到的一些临时文件，可以利用最左原则和覆盖索引配合，减少索引的维护
* 对于普通索引，如果是写多读少的服务，并且服务的唯一性要求没那么高，或者业务代码可以保证唯一性时，可以用普通索引。因为普通索引可以用到Change Buffer，可以把一些写操作缓存下来，在读的时候进行merge的操作，这样可以提高写入的速度和内存的命中率
* 如果索引走不上，可以考虑：
  * 是否是SQL有问题；
  * 或者对索引字段进行一些函数操作，在连接查询时两个表的编码不一样；
  * 或者字段类型不一样，比如String赋给它一个ID，在MySQL里默认会把String转为ID，用到了隐式的cast函数转换
  * 可能是索引统计信息有问题，可以Analyze table重新统计索引信息，因为索引信息不是一个准确值，是随机采样的过程 
  * 如果业务表增加太多，内存的空洞比较多，都有可能造成内存索引选择的问题   



### 场景1：全表count查询优化

添加索引：

*alter table user add key(name);*

然后进行查询：*select count(name) from user;*      此时就通过索引覆盖提高查询效率

### **场景2：列查询回表优化**

*select id,name,sex ... where name='shenjian';*

将单列索引(name)升级为联合索引 index(name, sex)，即可避免回表。

### **场景3：分页查询**

*select id,name,sex ... **order by** name limit 500,100;*

将单列索引(name)升级为联合索引 index(name, sex)，也可以避免回表



## MVCC机制

MVCC: Multiversion Concurrency Control,翻译为多版本并发控制，其目标就是为了提高数据库在高并发场景下的性能。

**MVCC最大的优势：读不加锁，读写不冲突。在读多写少的场景下极大的增加了系统的并发性能**

### MySQL基本结构

![image-20200916132304206](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916132304206.png)



##### MYISAM并不支持事务，所以InnoDB实现了MVCC的事务并发处理机制

### SQL事务隔离级别

`read uncommitted` 读未提交： 一个事务还没提交时，它做的变更就能被别的事务看到。

`read committed` 读已提交：一个事务提交之后，它做的变更才会被其他事务看到。

`repeatable read` 可重复读：一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的。在可重复读隔离级别下，未提交变更对其他事务也是不可见的。

`serializable` 串行化 ：对于同一行记录，“写”会加“写锁”，“读”会加“读锁”。

##### 而MVCC只在RC读提交和RR可重复度的隔离级别下才触发该机制

##### InnoDB相比与MYISAM的提升就是对于行级锁的支持和对事务的支持，而应对高并发事务, `MVCC` 比单纯的加行锁更有效, 开销更小。

##### 在不同的隔离级别下，数据库通过 `MVCC` 和隔离级别，让事务之间并行操作遵循了某种规则，来保证单个事务内前后数据的一致性。

### 并发事务可能出现的问题

`Lost Update`更新丢失: 多个事务对同一行数据进行读取初值更新时，由于每个事务对其他事务都未感知，会造成最后的更新覆盖了其他事务所做的更新。

`dirty read`脏读: 事务一个正在对一条记录进行修改，在完成并提交前事务二也来读取该条记录，事务二读取了事务一修改但未提交的数据，如果事务一回滚，那么事务二读取到的数据就成了“脏”数据。

`non-repeatable read`不可重复读: 事务在读取某些数据后的某个时间再次读取之前读取过的数据，发现读出的数据已经发生了改变或者删除，这种现象称为“不可重复读”

`phantom read`幻读: 事务按相同的查询条件重新读取以前检索过的数据，发现其他事务插入了满足查询条件的新数据，这种现象称为“幻读”

## MVCC原理

MVCC(Multi Version Concurrency Control的简称)，代表多版本并发控制。与MVCC相对的，是基于锁的并发控制，Lock-Based Concurrency Control)。
 MVCC最大的优势：读不加锁，读写不冲突。在读多写少的OLTP应用中，读写不冲突是非常重要的，极大的增加了系统的并发性能

MySQL从概念上可以分为四层，顶层是**接入层**，不同语言的客户端通过mysql的协议与mysql服务器进行连接通信，接入层进行权限验证、连接池管理、线程管理等。下面是mysql**服务层**，包括sql解析器、sql优化器、数据缓冲、缓存等。再下面是mysql中的**存储引擎层**，mysql中存储引擎是基于表的。最后是**系统文件层**，保存数据、索引、日志等。

大多数数据库系统的默认隔离级别都是READ COMMITTED（但MySQL不是)，InnoDB存储引擎默认隔离级别REPEATABLE READ，通过多版本并发控制（MVCC，Multiversion Concurrency Control）解决了幻读的问题。

在InnoDB中MVCC的实现通过两个重要的字段进行连接：`DB_TRX_ID`和`DB_ROLL_PT`，在多个事务并行操作某行数据的情况下，不同事务对该行数据的`UPDATE`会产生多个版本，数据库通过`DB_TRX_ID`来标记版本，然后用`DB_ROLL_PT`回滚指针将这些版本以先后顺序连接成一条 `Undo Log` 链。

* 表中每行记录

![image-20200916132308648](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916132308648.png)

##### `DB_TRX_ID`: 事务id，6byte，每处理一个事务，值自动加一。

##### `DB_ROLL_PT`: 回滚指针，7byte，指向当前记录的`ROLLBACK SEGMENT` 的undolog记录，通过这个指针获得之前版本的数据。该行记录上所有旧版本在 `undolog` 中都通过链表的形式组织。

##### 还有一个`DB_ROW_ID(隐含id,6byte，由innodb自动产生)`，InnoDB下聚簇索引B+Tree的构造规则:

如果声明了主键，InnoDB以用户指定的主键构建B+Tree，如果未声明主键，InnoDB 会自动生成一个隐藏主键，说的就是`DB_ROW_ID`。另外，每条记录的头信息（record header）里都有一个专门的`bit`（deleted_flag）来表示当前记录是否已经被删除

### Undo log链的构建

1. 事务A对DB_ROW_ID=1这一行加排它锁

2. 将修改行原本的值拷贝到Undo log中

3. 修改目标值产生一个新版本，将`DB_TRX_ID`设为当前事务ID，将`DB_ROLL_PT`指向拷贝到Undo log中的旧版本记录

4. 记录redo log， bin log

![image-20200916132313054](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916132313054.png)

##### INSERT: 产生一条新的记录，该记录的`DB_TRX_ID`为当前事务ID

##### DELETE: 特殊的UPDATE，在`DB_TRX_ID`上记录下当前事务的ID，同时将`delete_flag`设为true，在执行commit时才进行删除操作

MVCC的规则大概就是以上所述，那么它是如何实现高并发下`RC`和`RR`的隔离性呢，这就是在MVCC机制下基于生成的Undo log链和一致性视图ReadView来实现的。

## 一致性视图的生成 ReadView

要实现`read committed`在另一个事务提交之后其他事务可见和`repeatable read`在一个事务中SELECT操作一致，就是依靠ReadView，对于`read uncommitted`，直接读取最新值即可，而`serializable`采用加锁的策略通过牺牲并发能力而保证数据安全，因此只有`RC`和`RR`这两个级别需要在MVCC机制下通过ReadView来实现。

**在read committed级别下，readview会在事务中的每一个SELECT语句查询发送前生成**（也可以在声明事务时显式声明`START TRANSACTION WITH CONSISTENT SNAPSHOT`），因此每次SELECT都可以获取到当前已提交事务和自己修改的最新版本。而在`repeatable read`级别下，每个事务只会在第一个SELECT语句查询发送前或显式声明处生成，其他查询操作都会基于这个ReadView，这样就保证了一个事务中的多次查询结果都是相同的，因为他们都是基于同一个ReadView下进行MVCC机制的查询操作。

InnoDB为每一个事务构造了一个数组`m_ids`用于保存一致性视图生成瞬间当前所有`活跃事务`(开始但未提交事务)的ID，将数组中事务ID最小值记为低水位`m_up_limit_id`，当前系统中已创建事务ID最大值+1记为高水位`m_low_limit_id`，构成如图所示 : 

![image-20200916132318039](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916132318039.png)


 一致性视图下查询操作的流程如下:

1. 当查询发生时根据以上条件生成ReadView，该查询操作遍历Undo log链，根据当前被访问版本(可以理解为Undo log链中每一个记录即一个版本，遍历都是从最新版本向老版本遍历)的`DB_TRX_ID`，如果`DB_TRX_ID`小于`m_up_limit_id`,则该版本在ReadView生成前就已经完成提交，该版本可以被当前事务访问。**`DB_TRX_ID`在绿色范围内的可以被访问**
2. 若被访问版本的`DB_TRX_ID`大于`m_up_limit_id`，说明该版本在ReadView生成之后才生成，因此该版本不能被访问，根据当前版本指向上一版本的指针`DB_ROLL_PT`访问上一个版本，继续判断。**`DB_TRX_ID`在蓝色范围内的都不允许被访问**
3. 若被访问版本的`DB_TRX_ID`在[m_up_limit_id, m_low_limit_id)区间内，则判断`DB_TRX_ID`是否等于当前事务ID，等于则证明是当前事务做的修改，可以被访问，否则不可被访问, 继续向上寻找。**只有`DB_TRX_ID`等于当前事务ID才允许访问橙色范围内的版本**
4. 最后，还要确保满足以上要求的可访问版本的数据的`delete_flag`不为true，否则查询到的就会是删除的数据。

所以以上总结就是**只有当前事务修改的未commit版本和所有已提交事务版本允许被访问**。我想现在看文章的你应该是明白了(主要是说我自己)。

## 一致性读和当前读

前面说的都是查询相关，那么涉及到多个事务的查询同时还有更新操作时，MVCC机制如何保证在实现事务隔离级别的同时进行正确的数据更新操作，保证事务的正确性呢，我们可以看一个案例:

```sql
DROP TABLE IF EXISTS `mvccs`;
CREATE TABLE `mvccs`( `field` INT)ENGINE=InnoDB;
INSERT INTO `mvccs` VALUES(1); -- 插入一条数据
```

![image-20200916132324105](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916132324105.png)

假设在所有事务开始前当前有一个活跃事务10，且这三个事务期间没有其他并发事务:

1. 在操作1开始SELECT语句时，需要创建一致性视图，此时当前事务的一致性视图为[10, 100, 200，301), 事务100开始查询Undo log链，第一个查询到的版本为为事务200的操作4的更新操作， `DB_TRX_ID`在`m_ids`数组但并不等于当前事务ID， 不可被访问；
2. 向上查询下一个即事务300在操作6时生成的版本，小于高水位`m_up_limit_id`，且不在`m_ids`中，处于已提交状态，因此可被访问；
3. 综上在`RR`和`RC`下得到操作1查询的结果都是2

那么操作5查询到的field的值是多少呢？

在`RR`下，我们可以明确操作2和操作3查询field的值都是1，在`RC`下操作2为1，操作3的值为2，那么操作5的值呢？

答案在`RR`和`RC`下都是是3，我一开始以为`RR`下是2，因为这里如果按照一致性读的规则，事务300在操作2时都未提交，对于事务200来说应该时不可见状态，你看我说的是不是好像很有道理的样子？

上面的问题在于UPDATE操作都是读取**当前读(current read)**数据进行更新的，而不是一致性视图ReadView，因为如果读取的是ReadView，那么事务300的操作会丢失。当前读会读取记录中的最新数据，从而解决以上情形下的并发更新丢失问题。

## MySQL事务日志

事务日志可以帮助提高事务的效率。使用事务日志，存储引擎在修改表的数据时只需要修改其内存拷贝，再把该修改行为记录到持久在硬盘上的事务日志中，而不用每次都将修改的数据本身持久到磁盘。事务日志采用的是追加的方式，因此写日志的操作是磁盘上一小块区域内的顺序I/O，而不像随机I/O需要在磁盘的多个地方移动磁头，所以采用事务日志的方式相对来说要快得多。事务日志持久以后，内存中被修改的数据在后台可以慢慢地刷回到磁盘。目前大多数存储引擎都是这样实现的，我们通常称之为预写式日志（Write-Ahead Logging），修改数据需要写两次磁盘。
 如果数据的修改已经记录到事务日志并持久化，但数据本身还没有写回磁盘，此时系统崩溃，存储引擎在重启时能够自动恢复这部分修改的数据。

MySQL Innodb中跟数据持久性、一致性有关的日志，有以下几种：

- Bin Log: 是mysql服务层产生的日志，常用来进行数据恢复、数据库复制，常见的mysql主从架构，就是采用slave同步master的binlog实现的
- Redo Log: 记录了数据操作在物理层面的修改，mysql中使用了大量缓存，修改操作时会直接修改内存，而不是立刻修改磁盘，事务进行中时会不断的产生redo log，在事务提交时进行一次flush操作，保存到磁盘中。当数据库或主机失效重启时，会根据redo log进行数据的恢复，如果redo log中有事务提交，则进行事务提交修改数据。
- Undo Log: 除了记录redo log外，当进行数据修改时还会记录undo log，undo log用于数据的撤回操作，它记录了修改的反向操作，比如，插入对应删除，修改对应修改为原来的数据，通过undo log可以实现事务回滚，并且可以根据undo log回溯到某个特定的版本的数据，实现MVCC



## Explain分析索引

##### Explain分析出来的索引不一定是最优的，可能会选错，因为索引时可能会涉及到回表操作，或者排序操作，可能导致索引选错 

#### 索引查询速度慢的情况

* 首先可以考虑用force index，强制走一个索引，但只能作为一个应急预案，不推荐经常使用。因为一旦迁移到别的数据库里就不支持了，还需要做代码的重新发布 
* 可以用覆盖索引 + 最左前缀原则，考虑能否把原来的索引删掉重新分析选择索引

### 最左前缀原则

https://www.cnblogs.com/ljl150/p/12934071.html

在MySQL建立联合索引时会遵守最左前缀匹配原则，即最左优先，在检索数据时从联合索引的最左边开始匹配。

索引的底层原理。索引的底层是一颗B+树，那么联合索引的底层也就是一颗B+树，只不过联合索引的B+树节点中存储的是键值

即where后面的查询条件是从左到右匹配的

![image-20201115104939876](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201115104939876.png)

因为有查询优化器explane，所以sql语句中字段的顺序不需要和联合索引定义的字段顺序相同

但在匹配最左边的列时，如果查询条件不按照依次匹配就会造成Using where全索引扫描，如：

![image-20201115104935632](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201115104935632.png)

![image-20201115104931263](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201115104931263.png)

通过key字段可知，在搜索过程中也使用到了联合索引，但使用的是联合索引中的（id）索引，从key_len字段也可知。因为联合索引树是按照id字段创建的，但age相对于id来说是无序的，只有id只有序的，所以他只能使用联合索引中的id索引。而查询条件age没有用到索引



### change buffer

change buffer 是对数据进行修改的时候(insert、update、delete)的时候将修改优先写入change buffer，减少磁盘的随机IO消耗。但在内存中有拷贝，也会被写入到磁盘上。

当需要更新一个数据页时，如果数据页在内存中就直接更新，而如果这个数据页还没有在内存中的话，在不影响数据一致性的前提下，InnoDB 会将这些更新操作缓存在 change buffer 中，这样就不需要从磁盘中读入这个数据页了。在下次查询需要访问这个数据页的时候，将数据页读入内存，然后执行 change buffer 中与这个页有关的操作。通过这种方式就能保证这个数据逻辑的正确性。

对于唯一索引来说，所有的更新操作都要先判断这个操作是否违反唯一性约束。而这必须要将数据页读入内存才能判断。如果都已经读入到内存了，那直接更新内存会更快，就没必要使用 change buffer 了。

因此，唯一索引的更新就不能使用 change buffer，实际上也只有普通索引可以使用。



## MySQL数据页

数据页之间是以双向链表的形式存储，数据页里的记录以单链表的形式连接。所以这里可以**介绍全表扫描的方式**：即定位到第一个数据页，然后从第一条记录开始遍历，依次遍历完所有记录开始遍历第二个数据页，直到全部遍历完
  数据在innodb是以B+树(数据结构)的形式存储，MySQL会默认给每张表创建**聚蔟索引**，聚蔟索引的特点是根据主键大小按升序创建，在B+树里面分层，底部叶子层存储的是一张张的数据页(**数据页的记录包含所有数据，即所谓的索引即数据**)，非叶子层存储的则是目录结构(存储主键+数据页号)，用于快速定位对应的数据页。所以，**所谓的全表扫描，就是扫描聚蔟索引的叶子层数据！**
  因为innodb默认创建聚簇索引，所以当我们用主键来查找数据的时候就非常快了，能利用到索引目录的结构非常迅速定位到目标！

##### 数据页类型

  MySQL的数据页类型有十多种，包括：索引页，Undo页，Inode页，系统页，BloB页等，但我们最常用的就是索引页

##### 数据页头

  数据页的header即数据页头就是记载数据页的相关信息，可以说是数据页的核心部分(除了记录的真实数据)了，所以掌握了头部，就相当于掌握了数据页的结构。

![image-20201107170116020](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201107170116020.png)

### innodb数据页结构

页（Page）是 Innodb 存储引擎用于管理数据的最小磁盘单位。常见的页类型有数据页、Undo 页、系统页、事务数据页等，默认的页大小为 16KB，每个页中至少存储有 2 条或以上的行记录

下图是 Innodb 逻辑存储结构图，从上往下依次为：Tablespace、Segment、Extent、Page 以及 Row。本文关注的重点是 Page 和 Row 的数据结构。

![image-20201107165038832](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201107165038832.png)

#### Page

![image-20201107170151912](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201107170151912.png)

上图为 Page 数据结构，File Header 字段用于记录 Page 的头信息，其中比较重要的是 FIL_PAGE_PREV 和 FIL_PAGE_NEXT 字段，通过这两个字段，我们可以找到该页的上一页和下一页，实际上所有页通过两个字段可以形成一条双向链表。Page Header 字段用于记录 Page 的状态信息。接下来的 Infimum 和 Supremum 是两个伪行记录，Infimum（下确界）记录比该页中任何主键值都要小的值，Supremum （上确界）记录比该页中任何主键值都要大的值，这个伪记录分别构成了页中记录的边界。

![image-20201107170311943](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201107170311943.png)

User Records 中存放的是实际的数据行记录，具体的行记录结构将在本文的第二节中详细介绍。Free Space 中存放的是空闲空间，被删除的行记录会被记录成空闲空间。Page Directory 记录着与二叉查找相关的信息。File Trailer 存储用于检测数据完整性的校验和等数据



#### Row

Innodb 存储引擎提供了两种格式的行记录：Compact 和 Redundant。

### Compact 行记录

![img](https://images2018.cnblogs.com/blog/919737/201804/919737-20180408164927757-867511928.png)

变长字段长度列表：逆序记录每一个列的长度，如果列的长度小于 255 字节，则使用一个字节，否则使用 2 个字节。该字段的实际长度取决于列数和每一列的长度，因此是变长的。

NULL 标志位：一个字节，表示该行是否有 NULL 值（此处有疑问，8位，最多只能表示 8 列？）

记录头信息：五个字节，其中 next_record 记录了下一条记录的相对位置，一个页中的所有记录使用这个字段形成了一条单链表。

列数据部分：除了记录每一列对应的数据外，还有隐藏列，它们分别是 Transaction ID、Roll Pointer 以及 row_id（当没有指定主键）。

**注意**：此处需要注意固定长度 CHAR 数据类型和变长 VCHAR 数据类型在 Compact 记录下为 NULL 时不占用任何存储空间。

### Redundant 行记录

![img](https://images2018.cnblogs.com/blog/919737/201804/919737-20180408172411477-541622873.png)

字段长度偏移列表：与 Compact 中的变长字段长度列表相同的是它们都是按照列的逆序顺序设置值的，不同的是字段长度偏移列表记录的是偏移量，每一次都需要加上上一次的偏移，同时对于 CHAR 的 NULL 值，会直接按照最大空间记录，而对于 VCHAR 的 NULL 值不占用任何存储空间。

**注意**：此处需要注意 VCHAR 类型和 CHAR 类型在建表时传入的参数是字符长度而不是字节长度，实际的字节长度需要跟编码方式相关联，例如 UTF-8 一个中文字符需要 3 字节来表示，这样 CHAR(10) 以 UTF-8 来表示的话，它的字节长度在 10 - 30 之间。

### 行溢出

我们知道数据页的大小是 16KB，Innodb 存储引擎保证了每一页至少有两条记录，如果一页当中的记录过大，会截取前 768 个字节存入页中，其余的放入 BLOB Page。









