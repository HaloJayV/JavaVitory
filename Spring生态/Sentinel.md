[TOC]

# Sentinel实现熔断与限流 

* 下载安装Sentiel控制台：https://github.com/alibaba/Sentinel/releases

* 8080端口不能被占用

* pom依赖：spring-cloud-starter-alibaba-sentinel   、 sentinel-datasource-nacos

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

![image-20200905125829275](D:/JAVA/Java_Learning/Software/Typora/Picture/image-20200905125829275.png)

### Warm Up 预热：

公式:阈值除以coldFactor(默认值为3)，经过预热时长后才会达到阈值

默认coldFactor为3，即请求QPS从threshold/3开始，经预热时长逐渐升至设定的QPS阈值

* 应用场景：

如：秒杀系统在开启瞬间，会有很多流量上来，很可能把系统打死，预热方式就是为了保护系统，可慢慢的把流量放进来，慢慢的把阈值增长到设置的阈值。

### 排队等待： 

一秒钟只能通过阈值的请求数量

## 降级规则

![image-20200905162717618](D:/JAVA/Java_Learning/Software/Typora/Picture/image-20200905162717618.png)

### RT

![image-20200905170828023](D:/JAVA/Java_Learning/Software/Typora/Picture/image-20200905170828023.png)

### 异常比例

![image-20200905172323176](D:/JAVA/Java_Learning/Software/Typora/Picture/image-20200905172323176.png)

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
*  配置降级处理类中的处理方法： `@SentinelRe`source(value = "xxx",`blockHandlerClass = xxx.class, blockHandler = "xxx")`

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

![image-20200905212129737](D:/JAVA/Java_Learning/Software/Typora/Picture/image-20200905212129737.png)

![image-20200905212152816](D:/JAVA/Java_Learning/Software/Typora/Picture/image-20200905212152816.png)

* 重启sentinel微服务后，刷新sentinel发现nacos刚配置的流控规则还存在



