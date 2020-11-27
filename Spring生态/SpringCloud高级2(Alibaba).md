[TOC]

# Spring Cloud Alibaba

* 父工程pom

  ```xml
  <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-alibaba-dependencies</artifactId>
      <version>2.1.0.RELEASE</version>
      <type>pom</type>
      <scope>import</scope>
  </dependency>
  ```

# Nacos服务注册和配置中心

相当于Eureka + Config + Bus，  和Eureka一样都是AP模型（只支持注册临时实例+高可用+分区分治），但支持CP和AP切换

### 服务提供者

* pom依赖：spring-cloud-starter-alibaba-nacos-discovery

* 命令运行成功后直接访问http://localhost:8848/nacos

* yml

  ```yaml
  spring:
    application:
      name: nacos-payment-provider
    cloud:
      nacos:
        discovery:
          server-addr: localhost:8848 #配置Nacos地址
  ```

* 启动类注解：@EnableDiscoveryClient

### 服务消费者

* pom依赖：spring-cloud-starter-alibaba-nacos-discovery ， 包含了Ribbon

* yml

  ```yaml
  spring:
    application:
      name: nacos-order-consumer
    cloud:
      nacos:
        discovery:
          server-addr: localhost:8848
  
  #消费者将要去访问的微服务名称(注册成功进nacos的微服务提供者)
  service-url:
    nacos-user-service: http://nacos-payment-provider
  ```

* 可以用Ribbon + RestTemplate进行远程调用

## 配置中心

* pom依赖：spring-cloud-starter-alibaba-nacos-config

* 全局配置文件boostrap.yml

  ```yaml
spring:
    application:
      name: nacos-config-client
    cloud:
      nacos:
        discovery:
          server-addr: localhost:8848 #Nacos服务注册中心地址
        config:
          server-addr: localhost:8848 #Nacos作为配置中心地址
          file-extension: yaml #指定读取yaml格式的配置文件
          group: DEV_GROUP  
          namespace: 7d8f0f5a-6a53-4785-9686-dd460158e5d4
  
  # nacos上配置文件规则：${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension}
  # nacos-config-client-dev.yaml
  ```

![image-20200904231143004](../../../../Software/Typora/Picture/image-20200904231143004.png)

* application.yml

  ```yaml
  spring:
    profiles:
      active: dev # 表示开发环境
      #active: test # 表示测试环境
      #active: prod # 生成环境
  ```

* Nacos支持配置动态刷新：@RefreshScope //支持Nacos的动态刷新功能。

* 启动类注解：@EnableDiscoveryClient

## 分类配置

![image-20200905000009508](../../../../Software/Typora/Picture/image-20200905000009508.png)

![image-20200905000105474](../../../../Software/Typora/Picture/image-20200905000105474.png)

#### DataID方案

指定spring.profile.active和配置文件的DataID来使不同环境下读取不同的配置

 nacos上配置文件规则：${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension} => nacos-config-client-dev.yaml

#### Group方案

通过Group实现环境区分

#### Namespace方案

不同的Namespace之间是隔离的

* bootstrap.yml

  ```yaml
  spring:
    cloud:
      nacos:
        config:
          server-addr: localhost:8848 #Nacos作为配置中心地址
          file-extension: yaml #指定yaml格式的配置  DataID方案
          group: DEV_GROUP  # Group方案
          namespace: 7d8f0f5a-6a53-4785-9686-dd460158e5d4   # 在nacos中新建命名空间后Namespace的id
  ```

* yml选择读取的配置文件

  ```yaml
  spring:
    profiles:
      active: dev # 表示开发环境
      #active: test # 表示测试环境
      #active: info
  ```

## Nacos集群和持久化配置(重要)

![image-20200905110415450](../../../../Software/Typora/Picture/image-20200905110415450.png)

### derby到mysql切换配置步骤

Nacos默认自带的是嵌入式数据库derby，目前只支持mysql 数据源配置

* nacos\conf目录下找到sql脚本：nacos-mysql.sql ， 在数据库执行脚本，配置信息存储在数据库nacos-config
* nacos\conf目录下找到application.properties，修改该文件，修改内容在nacos官网，修改mysql用户名密码即可



## Nacos集群配置

* Linux服务器上nacos的集群配置cluster.conf， 先备份后配置, 根据自己的linux的ip配置

  ```
  192.168.xxx.xxx:3333
  192.168.xxx.xxx:4444
  192.168.xxx.xxx:5555
  ```

* 备份后编辑Nacos的启动脚本startup.sh,使他能够接受不同的启动端口![image-20200905122953529](../../../../Software/Typora/Picture/image-20200905122953529.png)

  ![image-20200905123237668](../../../../Software/Typora/Picture/image-20200905123237668.png)

* nginx.conf备份后修改 

  ![image-20200905123519126](../../../../Software/Typora/Picture/image-20200905123519126.png)

  按照指定配置文件启动nginx：./nginx -c /usr/local/nginx/conf/nginx.conf

* 根据端口启动3个命令： `./startup.sh -p 3333`  、  `./startup.sh -p 4444` 、  `./startup.sh -p 5555`
* 测试通过nginx访问nacos： http://192.168.111.144:1111/nacos/#/login
* 微服务springalibaba-provider-payment9002启动注册进nacos集群， 修改配置yml文件中的nacos地址：server-addr: 192.168.111.144.1111

# Sentinel实现熔断与限流 

* 下载安装Sentiel控制台：https://github.com/alibaba/Sentinel/releases

* 8080端口不能被占用

* pom依赖：spring-cloud-starter-alibaba-sentinel   、 sentinel-datasource-nacos

  ```
  <!-- https://mvnrepository.com/artifact/com.alibaba.cloud/spring-cloud-starter-alibaba-sentinel -->
  <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
      <version>2.2.1.RELEASE</version>
  </dependency>
  <!-- https://mvnrepository.com/artifact/com.alibaba.csp/sentinel-datasource-nacos -->
  <dependency>
      <groupId>com.alibaba.csp</groupId>
      <artifactId>sentinel-datasource-nacos</artifactId>
      <version>1.7.1</version>
      <scope>test</scope>
  </dependency>
  ```

* yml

  ```yaml
  spring:
    application:
      name: cloudalibaba-sentinel-service
    cloud:
      nacos:
        discovery:
          server-addr: localhost:8848 #Nacos服务注册中心地址
      sentinel:
        transport:
          dashboard: localhost:8080 #配置Sentinel dashboard地址
          port: 8719
        datasource:
          ds1:
            nacos:
              server-addr: localhost:8848
              dataId: cloudalibaba-sentinel-service
              groupId: DEFAULT_GROUP
              data-type: json
              rule-type: flow
  management:  # 开启监控
    endpoints:
      web:
        exposure:
          include: '*'
  feign:
    sentinel:
      enabled: true # 激活Sentinel对Feign的支持
  ```

* 访问控制台：http://localhost:8080      登录账号密码均为sentinel，sentinel控制台采用懒加载机制，需要先执行一次请求才会在控制台实时监控微服务

## 流控规则

![image-20200905125829275](../../../../Software/Typora/Picture/image-20200905125829275.png)

### Warm Up 预热：

公式:阈值除以coldFactor(默认值为3)，经过预热时长后才会达到阈值

默认coldFactor为3，即请求QPS从threshold/3开始，经预热时长逐渐升至设定的QPS阈值

* 应用场景：

如：秒杀系统在开启瞬间，会有很多流量上来，很可能把系统打死，预热方式就是为了保护系统，可慢慢的把流量放进来，慢慢的把阈值增长到设置的阈值。

### 排队等待： 

一秒钟只能通过阈值的请求数量

## 降级规则

![image-20200905162717618](../../../../Software/Typora/Picture/image-20200905162717618.png)

### RT

![image-20200905170828023](../../../../Software/Typora/Picture/image-20200905170828023.png)

### 异常比例

![image-20200905172323176](../../../../Software/Typora/Picture/image-20200905172323176.png)

## 热点key限流

官方文档：https://github.com/alibaba/Sentinel/wiki/%E7%83%AD%E7%82%B9%E5%8F%82%E6%95%B0%E9%99%90%E6%B5%81

```java
@GetMapping("/testHotKey")  // value = "testHotKey"表示该方法第一个请求参数只要QPS超过设置的阈值，马上降级处理
@SentinelResource(value = "testHotKey",blockHandler = "deal_testHotKey")  // 分别为资源名称（请求方法名）和降级处理方法
public String testHotKey(@RequestParam(value = "p1",required = false) String p1,
                         @RequestParam(value = "p2",required = false) String p2)
{
    return "------testHotKey";
}
public String deal_testHotKey (String p1, String p2, BlockException exception)
{
    return "------deal_testHotKey,o(╥﹏╥)o";  //sentinel系统默认的提示：Blocked by Sentinel (flow limiting)
}
```
#### 参数例外项:  我们期望p1参数当它是某个特殊值时，它的限流值和平时不一样

## @SentinelResource

相当于Hystrix的 @HystrixCommand，底层代码由try-catch-finally实现的

*  配置降级处理方法： `@SentinelResource(value = "xxx", blockHandler = "xx")`
*  配置降级处理类中的处理方法： `@SentinelResource(value = "xxx",blockHandlerClass = xxx.class, blockHandler = "xxx")`

##### Sentinel主要有三个核心Api：sphU定义资源、Tracer定义统计、ContextUtil定义了上下文

## 服务熔断功能

sentinel整合ribbon+openFeign+fallback

### fallback 和 blockHandler

##### fallback只负责业务异常、运行异常，而blockHandler负责sentinel配置违规、请求错误进行降级处理

```java
 @SentinelResource(value = "fallback",fallback = "handlerFallback",blockHandler = "blockHandler",
            exceptionsToIgnore = {IllegalArgumentException.class}) // 可以设置排除某个异常从而不进入fallback
```

### sentinel配合Openfeign

* yml 

  ```yaml
  # 激活Sentinel对Feign的支持
  feign:
    sentinel:
      enabled: true
  ```

## 持久化规则

##### 一旦我们重启应用,sentinel规则消失,生产环境需要将配置规则进行持久化

##### 解决：将限流规则持久进Nacos保存,只要刷新8401某个rest地址,sentinel控制台的流控规则就能看得到,只要Nacos里面的配置不删除,针对8401上的流控规则持续有效

* pom依赖：sentinel-datasource-nacos

* yml添加Nacos数据源配置，与Nacos连接绑定

  ```yaml
  spring:
    cloud:
      sentinel:
        datasource:
          ds1:
            nacos:
              server-addr: localhost:8848
              dataId: cloudalibaba-sentinel-service
              groupId: DEFAULT_GROUP
              data-type: json
              rule-type: flow
  ```

* 添加Nacos业务规则配置

![image-20200905212129737](../../../../Software/Typora/Picture/image-20200905212129737.png)

![image-20200905212152816](../../../../Software/Typora/Picture/image-20200905212152816.png)

* 重启sentinel微服务后，刷新sentinel发现nacos刚配置的流控规则还存在







# Seata处理分布式事务

全局唯一的事务id，类似一个班学生的班级号是全局唯一的

#### 三组件概念

* Transaction Coordinator(TC) ： 事务协调器。维护全局事务的运行状态, 负责协调并驱动全局事务的提交或回滚
* Transaction Manager(TM) ： 事务管理器。控制全局事务的边界,负责开启一个全局事务,并最终发起全局提交或全局回滚的决议（到TC）
* Resource Manager(RM)：资源管理器。控制分支事务,负责分支注册、状态汇报,并接受事务协调的指令,驱动分支(本地)事务的提交和回滚

TC：讲课老师     TM：班主任       RM：学生       XID：班级号

![image-20200905215924096](../../../../Software/Typora/Picture/image-20200905215924096.png)

### 玩法

本地@Transational、全局@GlobalTranstional

* 1、seata-server-0.9.0.zip解压到指定目录，备份后修改conf目录下的file.conf配置文件。主要修改:自定义事务组名称+事务日志存储模式为db+数据库连接

  * service模块：自定义事务组名称， 在service内添加：vgroup_mapping_test_tx_group = ‘xxx_tx_group’

    ![image-20200905223846013](../../../../Software/Typora/Picture/image-20200905223846013.png)

  * store模块：事务日志存储模式为db+数据库连接

    ```java
    store{
    	mode: 'db'
    }
    db {  #修改数据连接信息
    }
    ```

* 2、mysql5.7数据库新建库seata。 建表db_store.sql在seata-server-0.9.0\seata\conf目录里面

* 3、修改seata-server-0.9.0\seata\conf目录下的registry.conf目录下的registry.conf配置文件![image-20200905224729726](../../../../Software/Typora/Picture/image-20200905224729726.png)

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

![·](../../../../Software/Typora/Picture/image-20200906123244462.png)

 ### 二阶段提交

1.收到TC的分支提交请求，把请求放入一个异步任务的队列中，马上返回提交成功的结果给TC

2.异步任务阶段的分支提交请求将异步和批量地删除相应的UNDO LOG记录（快照数据）

![image-20200906123548027](../../../../Software/Typora/Picture/image-20200906123548027.png)

### 二阶段回滚

![image-20200906123615888](../../../../Software/Typora/Picture/image-20200906123615888.png)

# OSS

```java
/**
 * 1、引入oss-starter
 * 2、在application配置key，endpoint相关信息即可
     alicloud:
      access-key: LTAI4G3SehuZHENZA3FxE7ye
      secret-key: XuCsjuSidrTFu7uysdy3G9amFRCShV
      oss:
        endpoint: oss-cn-shenzhen.aliyuncs.com
 * 3、使用OSSClient 进行相关操作
 */
 	@Autowired
	OSSClient ossClient;

	@Test
	public void testUpload() throws FileNotFoundException {
        
// 上传文件流。
		InputStream inputStream = new FileInputStream("D:\\JAVA\\Java_Learning\\IDEA_Project02\\gulimall\\2b1837c6c50add30.jpg");
		ossClient.putObject("gulimall-javajayv", "2b1837c6c50add30.jpg", inputStream);

// 关闭OSSClient。
		ossClient.shutdown();
	}
```

### 服务端签名后直传

客户端请求服务端，服务端返回签名，客户端直接访问oss即可



































