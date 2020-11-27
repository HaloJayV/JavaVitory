

[TOC]

# Seata处理分布式事务

### 分布式事务问题

一次业务操作需要跨多个数据源或需要垮多个系统进行远程调用, 就会产生分布式事务问题

##### 即每个服务内部的数据一致性由本地事务保证，但是全局的数据一致性问题无法保证

##### Seata是一款开源的分布式事务解决方案,  致力于在微服务架构下提供高性能和简单易用的分布式事务服务

## 一个典型的分布式事务过程

### （1+3套件）分布式事务处理过程-ID+三组件模型

#### Transaction ID(XID)

全局唯一的事务id，类似一个班学生的班级号是全局唯一的

#### 三组件概念

* Transaction Coordinator(TC) ： 事务协调器。维护全局事务的运行状态, 负责协调并驱动全局事务的提交或回滚
* Transaction Manager(TM) ： 事务管理器。控制全局事务的边界,负责开启一个全局事务,并最终发起全局提交或全局回滚的决议（到TC）
* Resource Manager(RM)：资源管理器。控制分支事务,负责分支注册、状态汇报,并接受事务协调的指令,驱动分支(本地)事务的提交和回滚

TC：讲课老师     TM：班主任       RM：学生       XID：班级号

![image-20200905215924096](D:/JAVA/Java_Learning/Software/Typora/Picture/image-20200905215924096.png)

### 玩法

本地@Transational、全局@GlobalTranstional

* 1、seata-server-0.9.0.zip解压到指定目录，备份后修改conf目录下的file.conf配置文件。主要修改:自定义事务组名称+事务日志存储模式为db+数据库连接

  * service模块：自定义事务组名称， 在service内添加：vgroup_mapping_test_tx_group = ‘xxx_tx_group’

    ![image-20200905223846013](D:/JAVA/Java_Learning/Software/Typora/Picture/image-20200905223846013.png)

  * store模块：事务日志存储模式为db+数据库连接

    ```java
    store{
    	mode: 'db'
    }
    db {  #修改数据连接信息
    }
    ```

* 2、mysql5.7数据库新建库seata。 建表db_store.sql在seata-server-0.9.0\seata\conf目录里面

* 3、修改seata-server-0.9.0\seata\conf目录下的registry.conf目录下的registry.conf配置文件![image-20200905224729726](D:/JAVA/Java_Learning/Software/Typora/Picture/image-20200905224729726.png)

## 下单案例

### 订单/库存/账户业务数据库准备

#### 创建业务数据库

* seata_order:存储订单的数据库、seata_order库下新建t_order表

  ```
  DROP TABLE IF EXISTS `t_order`;
  CREATE TABLE `t_order`  (
    `int` bigint(11) NOT NULL AUTO_INCREMENT,
    `user_id` bigint(20) DEFAULT NULL COMMENT '用户id',
    `product_id` bigint(11) DEFAULT NULL COMMENT '产品id',
    `count` int(11) DEFAULT NULL COMMENT '数量',
    `money` decimal(11, 0) DEFAULT NULL COMMENT '金额',
    `status` int(1) DEFAULT NULL COMMENT '订单状态:  0:创建中 1:已完结',
    PRIMARY KEY (`int`) USING BTREE
  ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '订单表' ROW_FORMAT = Dynamic;
  ```

* seata_storage:存储库存的数据库、seata_storage库下新建t_storage表

  ```
  DROP TABLE IF EXISTS `t_storage`;
  CREATE TABLE `t_storage`  (
    `int` bigint(11) NOT NULL AUTO_INCREMENT,
    `product_id` bigint(11) DEFAULT NULL COMMENT '产品id',
    `total` int(11) DEFAULT NULL COMMENT '总库存',
    `used` int(11) DEFAULT NULL COMMENT '已用库存',
    `residue` int(11) DEFAULT NULL COMMENT '剩余库存',
    PRIMARY KEY (`int`) USING BTREE
  ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '库存' ROW_FORMAT = Dynamic;
  INSERT INTO `t_storage` VALUES (1, 1, 100, 0, 100);
  
  ```

* seata_account:存储账户信息的数据库、seata_account库下新建t_account表

  ```
  CREATE TABLE `t_account`  (
    `id` bigint(11) NOT NULL COMMENT 'id',
    `user_id` bigint(11) DEFAULT NULL COMMENT '用户id',
    `total` decimal(10, 0) DEFAULT NULL COMMENT '总额度',
    `used` decimal(10, 0) DEFAULT NULL COMMENT '已用余额',
    `residue` decimal(10, 0) DEFAULT NULL COMMENT '剩余可用额度',
    PRIMARY KEY (`id`) USING BTREE
  ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '账户表' ROW_FORMAT = Dynamic;
   
  INSERT INTO `t_account` VALUES (1, 1, 1000, 0, 1000);
  
  ```

* 按照上述3库分别建立对应的回滚日志表,   

  订单-库存-账户3个库下都需要建各自独立的回滚日志表 : seata-server-0.9.0\seata\conf\目录下的db_undo_log.sql

### 订单/库存/账户业务微服务准备

##### 下订单->减库存->扣余额->改(订单)状态。若其中一环出错，数据需要回滚

* 新建订单、库存、账户三个微服务。以订单为例

* pom：nacos、seata、openfeign、web、actuator、mysql、druid、mybatis、test、lombok

  ```xml
  		<dependency>
              <groupId>com.alibaba.cloud</groupId>
              <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
              <exclusions>
                  <exclusion>
                      <artifactId>seata-all</artifactId>
                      <groupId>io.seata</groupId>
                  </exclusion>
              </exclusions>
          </dependency>
          <dependency>
              <groupId>io.seata</groupId>
              <artifactId>seata-all</artifactId>
              <version>0.9.0</version>
          </dependency>
  ```

* yml

  ```yaml
  server:
    port: 2001
  
  spring:
    application:
      name: seata-order-service
    cloud:
      alibaba:
        seata:
          #自定义事务组名称需要与seata-server中的对应
          tx-service-group: fsp_tx_group
      nacos:
        discovery:
          server-addr: localhost:8848
    datasource:
      driver-class-name: com.mysql.jdbc.Driver
      url: jdbc:mysql://localhost:3306/seata_order
      username: root
      password: 123456
  
  feign:
    hystrix:
      enabled: false
  
  logging:
    level:
      io:
        seata: info
  
  mybatis:
    mapperLocations: classpath:mapper/*.xml
  ```

* file.conf

  ```properties
  transport {
    # tcp udt unix-domain-socket
    type = "TCP"
    #NIO NATIVE
    server = "NIO"
    #enable heartbeat
    heartbeat = true
    #thread factory for netty
    thread-factory {
      boss-thread-prefix = "NettyBoss"
      worker-thread-prefix = "NettyServerNIOWorker"
      server-executor-thread-prefix = "NettyServerBizHandler"
      share-boss-worker = false
      client-selector-thread-prefix = "NettyClientSelector"
      client-selector-thread-size = 1
      client-worker-thread-prefix = "NettyClientWorkerThread"
      # netty boss thread size,will not be used for UDT
      boss-thread-size = 1
      #auto default pin or 8
      worker-thread-size = 8
    }
    shutdown {
      # when destroy server, wait seconds
      wait = 3
    }
    serialization = "seata"
    compressor = "none"
  }
  
  service {
  
    vgroup_mapping.fsp_tx_group = "default" # 需要修改自定义事务组名称
  
    default.grouplist = "127.0.0.1:8091"
    enableDegrade = false
    disable = false
    max.commit.retry.timeout = "-1"
    max.rollback.retry.timeout = "-1"
    disableGlobalTransaction = false
  }
  
  
  client {
    async.commit.buffer.limit = 10000
    lock {
      retry.internal = 10
      retry.times = 30
    }
    report.retry.count = 5
    tm.commit.retry.count = 1
    tm.rollback.retry.count = 1
  }
  
  ## transaction log store
  store {
    ## store mode: file、db
    mode = "db"
  
    ## file store
    file {
      dir = "sessionStore"
  
      # branch session size , if exceeded first try compress lockkey, still exceeded throws exceptions
      max-branch-session-size = 16384
      # globe session size , if exceeded throws exceptions
      max-global-session-size = 512
      # file buffer size , if exceeded allocate new buffer
      file-write-buffer-cache-size = 16384
      # when recover batch read size
      session.reload.read_size = 100
      # async, sync
      flush-disk-mode = async
    }
  
    ## database store
    db {
      ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp) etc.
      datasource = "dbcp"
      ## mysql/oracle/h2/oceanbase etc.
      db-type = "mysql"
      driver-class-name = "com.mysql.jdbc.Driver"
      url = "jdbc:mysql://127.0.0.1:3306/seata"  # 需要修改数据源连接配置
      user = "root"
      password = "123456"
      min-conn = 1
      max-conn = 3
      global.table = "global_table"
      branch.table = "branch_table"
      lock-table = "lock_table"
      query-limit = 100
    }
  }
  lock {
    ## the lock store mode: local、remote
    mode = "remote"
  
    local {
      ## store locks in user's database
    }
  
    remote {
      ## store locks in the seata's server
    }
  }
  recovery {
    #schedule committing retry period in milliseconds
    committing-retry-period = 1000
    #schedule asyn committing retry period in milliseconds
    asyn-committing-retry-period = 1000
    #schedule rollbacking retry period in milliseconds
    rollbacking-retry-period = 1000
    #schedule timeout retry period in milliseconds
    timeout-retry-period = 1000
  }
  
  transaction {
    undo.data.validation = true
    undo.log.serialization = "jackson"
    undo.log.save.days = 7
    #schedule delete expired undo_log in milliseconds
    undo.log.delete.period = 86400000
    undo.log.table = "undo_log"
  }
  
  ## metrics settings
  metrics {
    enabled = false
    registry-type = "compact"
    # multi exporters use comma divided
    exporter-list = "prometheus"
    exporter-prometheus-port = 9898
  }
  
  support {
    ## spring
    spring {
      # auto proxy the DataSource bean
      datasource.autoproxy = false
    }
  }
  
  
  ```

* register.conf

  ```properties
  registry {
    # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
    type = "nacos"
  
    nacos {
      serverAddr = "localhost:8848"
      namespace = ""
      cluster = "default"
    }
    eureka {
      serviceUrl = "http://localhost:8761/eureka"
      application = "default"
      weight = "1"
    }
    redis {
      serverAddr = "localhost:6379"
      db = "0"
    }
    zk {
      cluster = "default"
      serverAddr = "127.0.0.1:2181"
      session.timeout = 6000
      connect.timeout = 2000
    }
    consul {
      cluster = "default"
      serverAddr = "127.0.0.1:8500"
    }
    etcd3 {
      cluster = "default"
      serverAddr = "http://localhost:2379"
    }
    sofa {
      serverAddr = "127.0.0.1:9603"
      application = "default"
      region = "DEFAULT_ZONE"
      datacenter = "DefaultDataCenter"
      cluster = "default"
      group = "SEATA_GROUP"
      addressWaitTime = "3000"
    }
    file {
      name = "file.conf"
    }
  }
  
  config {
    # file、nacos 、apollo、zk、consul、etcd3
    type = "file"
  
    nacos {
      serverAddr = "localhost"
      namespace = ""
    }
    consul {
      serverAddr = "127.0.0.1:8500"
    }
    apollo {
      app.id = "seata-server"
      apollo.meta = "http://192.168.1.204:8801"
    }
    zk {
      serverAddr = "127.0.0.1:2181"
      session.timeout = 6000
      connect.timeout = 2000
    }
    etcd3 {
      serverAddr = "http://localhost:2379"
    }
    file {
      name = "file.conf"
    }
  }    
  ```

* domain：Order.class

  ```java
  @Data
  @AllArgsConstructor
  @NoArgsConstructor
  public class Order
  {
      private Long id;
      private Long userId;
      private Long productId;
      private Integer count;
      private BigDecimal money;
      private Integer status; //订单状态：0：创建中；1：已完结
  }
  
  ```

* OrderDao.class

  ```
  @Mapper
  public interface OrderDao
  {
      //1 新建订单
      void create(Order order);
      //2 修改订单状态，从零改为1
      void update(@Param("userId") Long userId,@Param("status") Integer status);
  }
  ```

* OrderMapper.xml

  ```xml
  <mapper namespace="com.atguigu.springcloud.alibaba.dao.OrderDao">
      <resultMap id="BaseResultMap" type="com.atguigu.springcloud.alibaba.domain.Order">
          <id column="id" property="id" jdbcType="BIGINT"/>
          <result column="user_id" property="userId" jdbcType="BIGINT"/>
          <result column="product_id" property="productId" jdbcType="BIGINT"/>
          <result column="count" property="count" jdbcType="INTEGER"/>
          <result column="money" property="money" jdbcType="DECIMAL"/>
          <result column="status" property="status" jdbcType="INTEGER"/>
      </resultMap>
  
      <insert id="create">
          insert into t_order (id,user_id,product_id,count,money,status)
          values (null,#{userId},#{productId},#{count},#{money},0);
      </insert>
      
      <update id="update">
          update t_order set status = 1
          where user_id=#{userId} and status = #{status};
      </update>
  </mapper>
  ```

* 3个service，分别是订单service、远程调用账户微服务service、远程调用库存微服务service

* AccountService

  ```java
  @FeignClient(value = "seata-account-service")
  public interface AccountService
  {
      @PostMapping(value = "/account/decrease")
      CommonResult decrease(@RequestParam("userId") Long userId, @RequestParam("money") BigDecimal money);
  }
  ```

* StorageService

  ```java
  @FeignClient(value = "seata-storage-service")
  public interface StorageService
  {
      @PostMapping(value = "/storage/decrease")
      CommonResult decrease(@RequestParam("productId") Long productId, @RequestParam("count") Integer count);
  }
  ```

* OrderService

  ```
  public interface OrderService
  {
      void create(Order order);
  }
  ```

* MyBatisConfig

  ```
  @Configuration
  @MapperScan({"com.atguigu.springcloud.alibaba.dao"})
  public class MyBatisConfig {
  }
  ```

* DataSourceProxyConfig

  ```java
  // 使用Seata对数据源进行代理
  @Configuration
  public class DataSourceProxyConfig {
  
      @Value("${mybatis.mapperLocations}")
      private String mapperLocations;
  
      @Bean
      @ConfigurationProperties(prefix = "spring.datasource")
      public DataSource druidDataSource(){
          return new DruidDataSource();
      }
  
      @Bean
      public DataSourceProxy dataSourceProxy(DataSource dataSource) {
          return new DataSourceProxy(dataSource);
      }
  
      @Bean
      public SqlSessionFactory sqlSessionFactoryBean(DataSourceProxy dataSourceProxy) throws Exception {
          SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
          sqlSessionFactoryBean.setDataSource(dataSourceProxy);
          sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(mapperLocations));
          sqlSessionFactoryBean.setTransactionFactory(new SpringManagedTransactionFactory());
          return sqlSessionFactoryBean.getObject();
      }
  }
  ```

* 启动类

  ```java
  @EnableDiscoveryClient
  @EnableFeignClients
  @SpringBootApplication(exclude = DataSourceAutoConfiguration.class)//取消数据源的自动创建
  public class SeataOrderMainApp2001
  {
      public static void main(String[] args)
      {
          SpringApplication.run(SeataOrderMainApp2001.class, args);
      }
  }
  ```

### @GlobalTransactional

* serviceImpl

  ```java
      @Override   // 整个业务一发生异常，全局回滚
      @GlobalTransactional(name = "fsp-create-order",rollbackFor = Exception.class)
      public void create(Order order)
      {
          log.info("----->开始新建订单");
          //1 新建订单
          orderDao.create(order);
  
          //2 扣减库存
          log.info("----->订单微服务开始调用库存，做扣减Count");
          storageService.decrease(order.getProductId(),order.getCount());
          log.info("----->订单微服务开始调用库存，做扣减end");
  
          //3 扣减账户
          log.info("----->订单微服务开始调用账户，做扣减Money");
          accountService.decrease(order.getUserId(),order.getMoney());
          log.info("----->订单微服务开始调用账户，做扣减end");
  
          //4 修改订单状态，从零到1,1代表已经完成
          log.info("----->修改订单状态开始");
          orderDao.update(order.getUserId(),0);
          log.info("----->修改订单状态结束");
  
          log.info("----->下订单结束了，O(∩_∩)O哈哈~");
      }
  ```

  

## Seata原理

* 哪个业务service加了@GlobalTransactional注解，就是事务发起方TM， TC就是seata服务器 ，一个数据库就是一个事务参与方RM



### 分布式事务的执行流程

* TM开启分布式事务(TM向TC注册全局事务记录)
* 按业务场景,编排数据库、服务等事务内资源(RM向TC汇报资源准备状态)
* TM结束分布式事务,事务一阶段结束(TM通知TC提交/回滚分布式事务)
* TC汇报事务信息,决定分布式事务是提交还是回滚
* TC通知所有RM提交/回滚资源,事务二阶段结束

### SEATA四大模式

AT模式（默认）、TCC模式、SAGA模式、XA模式

### AT模式

能做到对业务无侵入

##### 前提：基于支持本地ACID事务的关系型数据库，通过JDBC访问数据库

### 整体机制

* 一阶段：业务数据和回滚日志记录在同一个本地事务中提交，释放本地和连接资源

* 二阶段：
  * 提交异步化，非常快速地完成
  * 回滚通过阶段一的回滚日志进行反向补偿

### 一阶段加载

![image-20200906123244462](D:/JAVA/Java_Learning/Software/Typora/Picture/image-20200906123244462.png)

 ### 二阶段提交

1.收到TC的分支提交请求，把请求放入一个异步任务的队列中，马上返回提交成功的结果给TC

2.异步任务阶段的分支提交请求将异步和批量地删除相应的UNDO LOG记录 

![image-20200906123548027](D:/JAVA/Java_Learning/Software/Typora/Picture/image-20200906123548027.png)



### 二阶段回滚

![image-20200906123615888](D:/JAVA/Java_Learning/Software/Typora/Picture/image-20200906123615888.png)







