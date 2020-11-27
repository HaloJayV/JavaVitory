[TOC]

# 框架搭建

### cloud和boot版本依赖关系

##### 通过json工具查看：https://start.spring.io.actuator/info

### 服务提供者

* yml

  ```yaml
  server:
    port: 8001
  
  spring:
    application:
      name: cloud-payment-service
    datasource:
      type: com.alibaba.druid.pool.DruidDataSource    # 当前数据源操作类型
      driver-class-name: com.mysql.cj.jdbc.Driver   # mysql驱动包
      url: jdbc:mysql://localhost:3306/springcloud2?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=false
      username: root
      password: root
  
  mybatis:
    mapper-Locations: classpath:mapper/*.xml
    type-aliases-package: com.atguigu.springcloud.entities    # 所有Entity别名类所在包
  ```
  
* 通用返回结果类

  ```java
  @Data
  @AllArgsConstructor
  @NoArgsConstructor
  public class CommonResult<T> {
      private Integer code;
      private String message;
      private T data;
      public CommonResult(Integer code, String message){
          this(code, message, null);  // this表示有参构造方法
      }
  }
  ```

* mapper.xml

  ```xml
  <mapper namespace="com.atguigu.springcloud.dao.PaymentDao">
      <!-- useGeneratedKeys="true"表示如果插入的表id以自增列为主键，则允许 JDBC 支持自动生成主键，并可将自动生成的主键id返回。可用于判断是否插入成功 -->
      <insert id="create" parameterType="Payment" useGeneratedKeys="true" keyProperty="id">
        insert into payment(serial) values(#{serial});
      </insert>
      <resultMap id="BaseResultMap" type="com.atguigu.springcloud.entities.Payment">
          <id column="id" property="id" jdbcType="BIGINT" />
          <id column="serial" property="serial" jdbcType="VARCHAR" />
      </resultMap>
      <select id="getPaymentById" parameterType="Long" resultMap="BaseResultMap">
          select * from payment where id=#{id};
      </select>
  </mapper>
  ```


* serviceImpl

  ```java
  @Service  // 表明向外提供的服务
  public class PaymentServiceImpl implements PaymentService {
      @Resource
      private PaymentDao paymentDao;
  
      public int create(Payment payment){
          return paymentDao.create(payment);
      }
  
      public Payment getPaymentById(Long id){
          return paymentDao.getPaymentById(id);
      }
  }
  ```

### 服务消费者

* yml

  ```
  server:
    port: 80
  ```

* restTemplate配置类（封装了httpclient）

  ```java
  @Configuration
  public class ApplicationContextConfig {
      @Bean
      public RestTemplate getRestTemplate(){
          return new RestTemplate();
      }
  }
  ```

* controller

  ```java
  @RestController
  @Slf4j
  public class OrderController {
      public static final String PAYMENT_URL = "http://localhost:8001";
  
      @Resource
      private RestTemplate restTemplate;
  
      @GetMapping("/consumer/payment/create")
      public CommonResult<Payment> create(Payment payment){
          return restTemplate.postForObject(PAYMENT_URL + "/payment/create", payment, CommonResult.class);
      }
  
      @GetMapping("/consumer/payment/get/{id}")
      public CommonResult<Payment> getPayment(@PathVariable("id") Long id){
          return restTemplate.getForObject(PAYMENT_URL + "/payment/get/" + id, CommonResult.class);
      }
  }
  ```

# 服务注册 

## Eureka (满足AP)

### 注册中心

* yml

  ```yaml
  server:
    port: 7001
  
  eureka:
    instance:
      hostname: eureka7001.com #eureka服务端的实例名称
    client:
      register-with-eureka: false     #false表示不向注册中心注册自己。
      fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
      service-url:
      #集群指向其它eureka
        defaultZone: http://eureka7002.com:7002/eureka/
      #单机就是7001自己
  #      defaultZone: http://eureka7001.com:7001/eureka/
    #server:
      #关闭自我保护机制，保证不可用服务被及时踢除
      #enable-self-preservation: false
      #eviction-interval-timer-in-ms: 2000
  ```

* 启动类注解：@EnableEurekaServer

#### 服务提供者

* yml 

  ```yaml
  server:
    port: 8001
  
  spring:
    application:
      name: cloud-payment-service
    datasource:
      type: com.alibaba.druid.pool.DruidDataSource    # 当前数据源操作类型
      driver-class-name: com.mysql.cj.jdbc.Driver   # mysql驱动包
      url: jdbc:mysql://localhost:3306/springcloud2?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=false
      username: root
      password: root
  
  eureka:
    client:
      #表示是否将自己注册进EurekaServer默认为true。
      register-with-eureka: true
      #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
      fetchRegistry: true
      service-url:
        #单机版
  #      defaultZone: http://localhost:7001/eureka
        # 集群版
        defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
    instance:
        instance-id: payment8001
        #访问路径可以显示IP地址
        prefer-ip-address: true
        #Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
        #lease-renewal-interval-in-seconds: 1
        #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
        #lease-expiration-duration-in-seconds: 2
  
  
  mybatis:
    mapper-Locations: classpath:mapper/*.xml
    type-aliases-package: com.atguigu.springcloud.entities    # 所有Entity别名类所在包
  ```

* 启动类注解：@EnableEurekaClient

### 服务消费者

* yml

  ```yaml
  server:
    port: 80
  spring:
      application:
          name: cloud-order-service
      zipkin:
        base-url: http://localhost:9411
      sleuth:
        sampler:
          probability: 1
  eureka:
    client:
      #表示是否将自己注册进EurekaServer默认为true。
      register-with-eureka: false
      #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
      fetchRegistry: true
      service-url:
        #单机
        #defaultZone: http://localhost:7001/eureka
        # 集群
        defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  # 集群版
  ```

* 启动类注解：@EnableEurekaClient

* controller

  ```java
  @RestController
  @Slf4j
  public class OrderController {
      // 取自yml文件的spring.application.name
      public static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";
      
      @Resource
      private RestTemplate restTemplate;
      @GetMapping("/consumer/payment/create")
      public CommonResult<Payment> create(Payment payment){
          return restTemplate.postForObject(PAYMENT_URL + "/payment/create", payment, CommonResult.class);
      }
      @GetMapping("/consumer/payment/get/{id}")
      public CommonResult<Payment> getPayment(@PathVariable("id") Long id){
          return restTemplate.getForObject(PAYMENT_URL + "/payment/get/" + id, CommonResult.class);
      }
  }
  ```

* RestTemplateConfig类添加注解：@LoadBalanced ，开启根据eureka里的微服务名进行负载均衡

```java
 	@Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
```

#### 服务发现

* 在启动类加上注解：@EnableDiscoveryClient

* ```
  	@GetMapping("/payment/discovery")
      public Object discovery(){
          List<String> services = discoveryClient.getServices();
          List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
          for (ServiceInstance instance : instances) {
              log.info(instance.getServiceId()+"\t"+instance.getHost()+"\t"+instance.getPort()+"\t"+instance.getUri());
    
          }
          return this.discoveryClient;
      }
  ```

### 自我保护机制

主要用于一组客户端和Eureka Server之间存在网络分区场景下的保护，EurekaServer会尝试保护其服务注册表中的信息，不再删除服务注册表中的数据，不会注销任何微服务。

* 设置心跳

  ```yaml
  eureka: 
    instance:
        instance-id: payment8001
        #访问路径可以显示IP地址
        prefer-ip-address: true
        #Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
        #lease-renewal-interval-in-seconds: 1
        #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
        #lease-expiration-duration-in-seconds: 2
  ```

## Zookeeper (满足CP)

* pom

  ```xml
          <!-- SpringBoot整合zookeeper客户端 -->
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
              <!--先排除自带的zookeeper3.5.3-->
              <exclusions>
                  <exclusion>
                      <groupId>org.apache.zookeeper</groupId>
                      <artifactId>zookeeper</artifactId>
                  </exclusion>
              </exclusions>
          </dependency>
          <!--添加zookeeper3.4.9版本-->
          <dependency>
              <groupId>org.apache.zookeeper</groupId>
              <artifactId>zookeeper</artifactId>
              <version>3.4.9</version>
          </dependency>
  ```

* yml

  ```yaml
  #服务别名----注册zookeeper到注册中心名称
  spring:
    application:
      name: cloud-provider-payment
    cloud:
      zookeeper:
        connect-string: 192.168.137.133:2181
  ```

* 默认服务节点是临时节点

* 服务提供者

  ```yaml
  ###consul服务端口号
  server:
    port: 8006
  
  spring:
    application:
      name: consul-provider-payment
  ####consul注册中心地址
    cloud:
      consul:
        host: localhost
        port: 8500
        discovery:
          #hostname: 127.0.0.1
          service-name: ${spring.application.name}
  ```

  ```
      @RequestMapping(value = "/payment/consul")
      public String paymentConsul()
      {
          return "springcloud with consul: "+serverPort+"\t   "+ UUID.randomUUID().toString();
      }
  ```

* 服务消费者

  * config

  ```
  @Configuration
  public class ApplicationContextConfig
  {
      @Bean
      @LoadBalanced
      public RestTemplate getRestTemplate()
      {
          return new RestTemplate();
      }
  }
  
  ```

* controller

  ```java
  @RestController
  @Slf4j
  public class OrderConsulController
  {
      public static final String INVOKE_URL = "http://consul-provider-payment";
  
      @Resource
      private RestTemplate restTemplate;
  
      @GetMapping(value = "/consumer/payment/consul")
      public String paymentInfo()
      {
          String result = restTemplate.getForObject(INVOKE_URL+"/payment/consul",String.class);
          return result;
      }
  }
  ```

## consul (满足CP)

官网下载运行consul.exe文件，使用开发模式启动：consul agent -dev , 控制台首页：http://localhost:8500

* 服务提供者

  ```yaml
  ###consul服务端口号
  server:
    port: 8006
  
  spring:
    application:
      name: consul-provider-payment
  ####consul注册中心地址
    cloud:
      consul:
        host: localhost
        port: 8500
        discovery:
          #hostname: 127.0.0.1
          service-name: ${spring.application.name}
  ```

  ```
      @RequestMapping(value = "/payment/consul")
      public String paymentConsul()
      {
          return "springcloud with consul: "+serverPort+"\t   "+ UUID.randomUUID().toString();
      }
  ```

* 服务消费者

  * config

  ```
  @Configuration
  public class ApplicationContextConfig
  {
      @Bean
      @LoadBalanced
      public RestTemplate getRestTemplate()
      {
          return new RestTemplate();
      }
  }
  
  ```

* controller

  ```
  @RestController
  @Slf4j
  public class OrderConsulController
  {
      public static final String INVOKE_URL = "http://consul-provider-payment";
  
      @Resource
      private RestTemplate restTemplate;
  
      @GetMapping(value = "/consumer/payment/consul")
      public String paymentInfo()
      {
          String result = restTemplate.getForObject(INVOKE_URL+"/payment/consul",String.class);
          return result;
      }
  }
  
  ```

# 负载均衡 Ribbon/LoadBalancer

## Ribbon（进程内LB）

##### 负载均衡 + RestTemplate调用

* nginx是服务器负载均衡，而Ribbon本地负载均衡，在调用微服务接口时，会在注册中心获取注册信息服务列表后缓存到JVM本地，从而在本地实现RPC

* 集中式LB：在服务消费方和提供方之间使用独立LB设施（可以是硬件F5也可以是软件nginx等），由设施负责把访问请求通过某种策略转发到服务提供方

* 进程内LB：将LB逻辑集成到消费方，消费方从服务注册中心获取到可用的地址，然后从地址中选出一个合适的服务器；

  ​					Ribbon就是进程内LB，只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方地址

* 架构：Ribbon是一个软负载均衡的客户端组件，可以和其他所需请求的客户端结合使用

* pom  : netflix-eureka-client包含了netflix-Ribbon 依赖

### RestTemplate

* `restTemplate.getForObject(url, Result.class);`  返回对象为响应体中数据转化成的对象，可以理解为json
* `restTemplate.getForEntity(url, Result.class).getBody;` 返回对象为ResponseEntity对象，包含响应中一些重要信息，如响应头、响应状态码、响应体等。

* `restTemplate.postForObject(url, param, Result.class);` 提交请求，直接返回响应体，请求时通过泛型约束响应体的类型，但是这个方法无法得到状态头信息。

### Ribbon核心组件 IRule（七大负载均衡算法）

```java
public interface IRule {
    Server choose(Object var1); // 选择服务
    void setLoadBalancer(ILoadBalancer var1);  // 设置负载均衡策略
    ILoadBalancer getLoadBalancer();  // 获取负载均衡策略
}
```

* com.netflix.loadbalancer.RoundRobinRule       轮询   (默认)

  * 轮询算法：rest接口第几次请求数 % 服务器集群总数量 = 实际调用服务器位置下标, 每次服务重启后rest接口计数从1开始

  ```
  List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
  ```

* com.netflix.loadbalancer.RandomRule     随机  

* com.netflix.loadbalancer.RetryRule    先按照RoundRobinRule的策略获取服务,如果获取服务失败则在指定时间内进行重试, 获取可用的服务   

* WeightedResponseTimeRule     对RoundRobinRule的扩展,响应速度越快的实例选择权重越大, 越容易被选择

* BestAvailableRule     会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务,然后选择一个并发量最小的服务

* AvailabilityFilteringRule      先过滤掉故障实例,再选择并发较小的实例

* ZoneAvoidanceRule       默认规则, 复合判断server所在区域的性能和server的可用性来选择服务器

#### 如何替换

* 官方：创建自定义配置类，且不能放在@ComponentScan扫描的包下以及子包下，否则这个配置类就会被所有Ribbon客户端共享，达不到特殊化定制的目的

  ```java
  @Configuration
  public class MySelfRule
  {
      @Bean
      public IRule myRule()
      {
          return new RandomRule();//定义为随机
      }
  }
  ```

* 启动类加上注解： `@RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration=MySelfRule.class)` 

### 手写负载均衡算法

* 在服务消费方的配置类里去掉@LoadBalanced

* 新建接口LoadBalancer 

  ```
  public interface LoadBalancer{
      ServiceInstance instances(List<ServiceInstance> serviceInstances);
  }
  ```

* 实现类LoadBalancerImpl

  ```java
  @Component
  public class MyLB implements LoadBalancer
  {
      private AtomicInteger atomicInteger = new AtomicInteger(0);  // 初始为0的并发原子整型数，线程共享
      
      // 通过CAS自旋锁 + AtomicInteger获取到当前是第几次访问
      public final int getAndIncrement()  // 获取第几次访问
      {
          int current;
          int next;
          do {
              current = this.atomicInteger.get();
              next = current >= 2147483647 ? 0 : current + 1; // 轮询计数
               // 自旋，如果当前值next与期望值current一致表示有进程枪资源请求，返回false，此时取反后为true
              // 如果当前值next与期望值current不一样，则current设置为next的值，且取反后返回为false，退出循环，表示轮询成功
          }while(!this.atomicInteger.compareAndSet(current, next));  
          System.out.println("*****第几次访问，次数next: "+next);
          return next;
      }
  
      //负载均衡算法：rest接口第几次请求数 % 服务器集群总数量 = 实际调用服务器位置下标  ，每次服务重启动后rest接口计数从1开始。
      @Override
      public ServiceInstance instances(List<ServiceInstance> serviceInstances)
      {
          int index = getAndIncrement() % serviceInstances.size();
          return serviceInstances.get(index); // 决定哪个服务instance干活
      }
  }
  ```

* Controller使用自己手写的算法

  ```java
  	@Resource
      private LoadBalancer loadBalancer;
      @Resource
      private DiscoveryClient discoveryClient;
  	@GetMapping(value = "/consumer/payment/lb")
      public String getPaymentLB()
      {
          List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
          if(instances == null || instances.size() <= 0){
              return null;
          }
          ServiceInstance serviceInstance = loadBalancer.instances(instances);
          URI uri = serviceInstance.getUri(); // 获取服务请求的访问地址
          return restTemplate.getForObject(uri+"/payment/lb",String.class);
      }
  ```


# 服务调用 openfeign

Feign是一个声明式WebService客户端，使用方法是定义一个服务接口然后添加注解。

Feign支持可拔插式的编码器和解码器，支持SpringMVC标准注解和HttpMessageConverters, 可以与Ribbon组合使用以支持负载均衡

* 往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一些客户端来包装这些微服务的调用，Feign是在此基础上做了进一步的封装
* 我们只需创建一个接口并使用注解的方式来配置它（以前是Dao接口上标注Mapper注解，现在是微服务接口上标注一个Feign注解即可），完成对服务提供方的接口绑定，简化了使用Ribbon+RestTemplate， 自动封装服务调用客户端的并发量
* Feign继承了Ribbon，Feign只需要定义服务绑定接口且以声明式的方法，简化实现服务调用

#### Feign

Feign自带负载均衡配置项

轻量级Restful的HTTP服务客户端，内置了Ribbon，用来做客户端负载均衡，去调用注册中心的服务，使用方法是：使用Feign注解定义接口，调用这个接口就可以调用注册中心的服务

#### OpenFeign

在Feign基础上支持了SpringMVC的注解如@RequestMapping等，@FeignClient可以解析@RequestMapping注解下的接口，并通过动态代理的方法产生实现类，实现类中做负载均衡并调用其他服务

### 使用

* 启动类加上注解： @EnableFeignClients

* service接口加上注解：@FeignClient(value = "CLOUD-PAYMENT-SERVICE")，并调用服务提供方的接口：

  ```java
  @Component
  @FeignClient(value = "CLOUD-PAYMENT-SERVICE")
  public interface PaymentFeignService{
      @GetMapping(value = "/payment/get/{id}")
      // @PathVariable()里必须加上 "id" ，否则报错
      public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id);
      @GetMapping(value = "/payment/feign/timeout")
      public String paymentFeignTimeout();
  } 
  ```


### 设置超时时间，默认是3秒 

```yaml
#设置feign客户端超时时间(OpenFeign默认支持ribbon)
ribbon:
#指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
  ReadTimeout: 5000
#指的是建立连接后从服务器读取到可用资源所用的时间
  ConnectTimeout: 5000
```

### 日志增强

OpenFeign日志级别：NONE、BASIC（请求方法、URL、响应状态码、执行时间）、HEADERS（包含BASIC外，还有请求和响应的头信息）、FULL（包含HEADERS外还有请求和响应的正文以及元数据）

```yaml
logging:
  level:
    # feign日志以什么级别监控哪个接口
    com.atguigu.springcloud.service.PaymentFeignService: debug
```

配置类  

```java
@Configuration
public class FeignConfig{
    @Bean
    Logger.Level feignLoggerLevel(){
        return Logger.Level.FULL;
    }
}
```

# 服务降级 Hystrix/sentinel(推荐)

## Hystrix

当某个服务故障后，向调用方返回一个符合预期的、可处理的备选响应（Fallback），而不是长时间等待或者抛出调用方无法处理的异常，保证了服务调用方的线程不会被长时间不必要地调用，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性

#### 服务降级 fallback

服务器忙，不让客户端等待并立刻返回一个友好提示fallback，一般以下情况会发生服务降级

* 程序运行异常
* 超时
* 服务熔断器打开，触发服务降级
* 线程池/信号量打满

#### 服务熔断 break

达到最大访问后，直接拒绝访问，然后调用服务降级的方法并返回友好提示

过程：服务降级=》进而熔断 =》 恢复调用链路

高并发会卡死的原因：tomcat的默认工作线程数被打满了，没有多余的线程来分解压力和处理

#### 服务限流 flowlimit

适用于秒杀、高并发等操作， 防止服务熔断

### 1.服务降级

#### 服务消费方（一般hystrix配置在客户端）

* yml 开启hystrix

  ```
  feign:
    hystrix:
      enabled: true
  ```

* 启动类加上注解：@EnableHystrix

* controller

  ```java
  @RestController
  @Slf4j
  @DefaultProperties(defaultFallback = "payment_Global_FallbackMethod") // 没有fallbackMethod指明则用统一的全局defaultFallback
  public class OrderHystirxController
  {
      @Resource
      private PaymentHystrixService paymentHystrixService;
  
      @GetMapping("/consumer/payment/hystrix/timeout/{id}")
      @HystrixCommand(fallbackMethod = "paymentTimeOutFallbackMethod",commandProperties = {
              @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="1500")
      })
      //@HystrixCommand
      public String paymentInfo_TimeOut(@PathVariable("id") Integer id)
      {
          int age = 10/0;
          String result = paymentHystrixService.paymentInfo_TimeOut(id);
          return result;
      }
      public String paymentTimeOutFallbackMethod(@PathVariable("id") Integer id)
      {
          return "我是消费者80,对方支付系统繁忙请10秒钟后再试或者自己运行出错请检查自己,o(╥﹏╥)o";
      }
      // 下面是全局fallback方法
      public String payment_Global_FallbackMethod()
      {
          return "Global异常处理信息，请稍后再试，/(ㄒoㄒ)/~~";
      }
  }
  ```

* xxxService， 在service层配置，实现解耦

  ```
  @Component
  @FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT" ,fallback = PaymentFallbackService.class)
  public interface PaymentHystrixService
  {
      @GetMapping("/payment/hystrix/timeout/{id}")
      public String paymentInfo_TimeOut(@PathVariable("id") Integer id);
  }
  ```

* xxxFallbackService

  ```
  @Component
  public class PaymentFallbackService implements PaymentHystrixService
  {
      @Override
      public String paymentInfo_TimeOut(Integer id)
      {
          return "-----PaymentFallbackService fall back-paymentInfo_TimeOut ,o(╥﹏╥)o";
      }
  }
  
  ```

### 2.服务熔断

服务熔断会触发服务降级，及：服务降级=》进而熔断 =》 恢复调用链路

* 服务提供者service

  ```java
   //=====服务熔断
      @HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback",commandProperties = {
          @HystrixProperty(name = "circuitBreaker.enabled",value = "true"),// 是否开启断路器
          @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),// 请求次数达到10次才打开断路器
          @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"), // 时间窗口期，10秒
          @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60"),// 10内10次请求失败率达到多少后跳闸
          // 恢复调用链路；多次错误后，错误率高于60%，当开始恢复正确时还处于熔断状态，多次正确后错误率60%以下，才恢复正确
      })
      public String paymentCircuitBreaker(@PathVariable("id") Integer id)
      {
          if(id < 0)
          {
              throw new RuntimeException("******id 不能负数");
          }
          String serialNumber = IdUtil.simpleUUID();
  
          return Thread.currentThread().getName()+"\t"+"调用成功，流水号: " + serialNumber;
      }
      public String paymentCircuitBreaker_fallback(@PathVariable("id") Integer id)
      {
          return "id 不能负数，请稍后再试，/(ㄒoㄒ)/~~   id: " +id;
      }
  ```

![image-20200903204908803](../../../../Software/Typora/Picture/image-20200903204908803.png)

熔断类型：

* 熔断打开：请求不再调用当前服务,内部设置一般为MTTR(平均故障处理时间),当打开长达导所设时钟则进入半熔断状态
* 熔断关闭：熔断关闭后不会对服务进行熔断
* 熔断半开：部分请求根据规则调用当前服务,如果请求成功且符合规则则认为当前服务恢复正常,关闭熔断

![image-20200903205505197](../../../../Software/Typora/Picture/image-20200903205505197.png)

![image-20200903210000706](../../../../Software/Typora/Picture/image-20200903210000706.png)

### 3.服务限流

### 服务监控hystrixDashboard

* pom（所有Provider微服务提供类都需要spring-boot-starter-actuator监控依赖部署）

```xml
	<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        </dependency>
        <!-- 凡是被监控的都要有此依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

* 监控微服务的启动类加上注解：@EnableHystrixDashboard

* 服务提供者的启动类加上配置，固定写法配置

  ```java
      /**
       *此配置是为了服务监控而配置，与服务容错本身无关，springcloud升级后的坑
       *ServletRegistrationBean因为springboot的默认路径不是"/hystrix.stream"，
       *只要在自己的项目里配置上下面的servlet就可以了
       */
      @Bean
      public ServletRegistrationBean getServlet() {
          HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
          ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
          registrationBean.setLoadOnStartup(1);
          registrationBean.addUrlMappings("/hystrix.stream");
          registrationBean.setName("HystrixMetricsStreamServlet");
          return registrationBean;
      }
  ```

* 打开网址：http://;localhost:8001/hystrix.stream   , 8001为被监控的端口

![image-20200903214543397](../../../../Software/Typora/Picture/image-20200903214543397.png)



# 服务网关 Gateway(推荐)/Zuul

## Zuul

* pom

  ```
  <dependencies>
      <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
      </dependency>
      <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
      </dependency>
  </dependencies>
  ```

* 启动类注解：@EnableEurekaClient、@EnableZuulProxy

* yml

  ```
  server:
    port: 9013
  spring:
    application:
      name: tensquare-encrypt
  zuul:
    routes:
      tensquare-article: #文章
        path: /article/** #配置请求URL的请求规则
        serviceId: tensquare-article #指定Eureka注册中心中的服务id
        strip-prefix: true
        sentiviteHeaders:
        customSensitiveHeaders: true
  ​
  eureka:
    client:
      service-url:
        defaultZone: http://127.0.0.1:6868/eureka/
    instance:
      prefer-ip-address: true
  ```


## Gateway

Gateway是基于Webflux中的reactor-netty响应式编程组件实现的，而非阻塞异步的WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty, 

### 三大核心概念

#### Route(路由)

路由是构建网关的基本模块,它由ID,目标URI,一系列的断言和过滤器组成,如断言为true则匹配该路由

#### Predicate(断言)

参考的是Java8的java.util.function.Predicate
开发人员可以匹配HTTP请求中的所有内容(例如请求头或请求参数),如果请求与断言相匹配则进行路由，也就是匹配条件

#### Filter(过滤)

指的是Spring框架中GatewayFilter的实例,使用过滤器,可以在请求被路由前或者之后对请求进行修改

### 配置的方式（推荐）

* 移除spring-boot-starter-web 依赖，导入 spring-boot-starter-gateway

```yaml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_route #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址, 实现负载均衡
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由

        - id: payment_route2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://cloud-payment-service #匹配后提供服务的路由地址, 实现负载均衡
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
            #- After=2020-02-21T15:51:37.485+08:00[Asia/Shanghai] # 这个时间之后的请求访问才有效
            #- Cookie=username,zzyy  ## 请求时需要带上cookie：
            #- Header=X-Request-Id, \d+  # 请求头要有X-Request-Id属性并且值为整数的正则表达式
```

### 编码的方式

```java
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder routeLocatorBuilder) {
        RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();
        routes.route("path_route_atguigu",
                r -> r.path("/guonei")
                        .uri("http://news.baidu.com/guonei")).build();
        return routes.build();
    }
```

### 自定义Filter

```java
@Component
@Slf4j
public class MyLogGateWayFilter implements GlobalFilter,Ordered {  // 实现接口
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("***********come in MyLogGateWayFilter:  "+new Date());
        String uname = exchange.getRequest().getQueryParams().getFirst("uname"); // 请求时要求带有参数uname
        if(uname == null){
            log.info("*******用户名为null，非法用户，o(╥﹏╥)o");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE); // 返回错误信息给请求方
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange); // 请求通过，放行
    }
    @Override
    public int getOrder(){  // 设置filter顺序
        return 0;
    }
}
```

# 服务配置 Config

Config为微服务架构中的微服务提供集中化的外部配置支持，配置服务器为各个不同微服务应用的所有环境提供一个中心化的外部配置。

![image-20200904132739184](../../../../Software/Typora/Picture/image-20200904132739184.png)

### 服务端

* pom：spring-cloud-config-server

* yml 

  ```yaml
  server:
    port: 3344
  spring:
    application:
      name:  cloud-config-center #注册进Eureka服务器的微服务名
    cloud:
      config:
        server:
          git:
            uri: git@github.com:zzyybs/springcloud-config.git #GitHub上面的git仓库地址
          ####搜索目录
            search-paths:
              - springcloud-config   # github上配置文件名
        ####读取分支
        label: master
  ```

* 启动类加上注解：@EnableConfigServer

* windows下修改hosts文件，增加映射：127.0.0.1 config-3344.com

* 测试通过Config微服务可以从GitHub上获取配置内容：http://config-3344.com:3344/master/config-dev.yml

##### 配置文件读取规则

/{label}/{application}-{profile}.yml

### 客户端

* pom：spring-cloud-starter-config

* bootstrap.yml

  ```yaml
  spring:
    application:
      name: config-client
    cloud:
      #Config客户端配置
      config:
        label: master #分支名称
        name: config #配置文件名称
        profile: dev #读取后缀名称  
        # 上述3个综合：master分支上config-dev.yml的配置文件被读取http://config-3344.com:3344/master/config-dev.yml
        uri: http://localhost:3344 #配置中心地址                                                                              
  ```

* 读取配置中心配置文件内容

  ```java
  @RestController
  @RefreshScope  // 动态刷新
  public class ConfigClientController
  {
      @Value("${config.info}")
      private String configInfo;
  
      @GetMapping("/configInfo")
      public String getConfigInfo()
      {
          return configInfo;
      }
  }
  ```

### 动态刷新

* pom引入监控依赖：spring-boot-starter-actuator

* bootstrap.yml添加配置

```yaml
# 暴露监控端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

* controller添加注解：@RefreshScope

# 消息总线 BUS

分布式自动刷新配置功能，Spring Cloud Bus配合Spring Cloud Config使用可以实现配置的动态刷新

Bus支持两种消息代理:RabbitMQ和Kafka

### 使用

* server和client的yml都要添加rabbit连接配置

  ```yaml
  #rabbitmq相关配置 15672是Web管理界面的端口；5672是MQ访问的端口
    rabbitmq:
      host: localhost
      port: 5672
      username: guest
      password: guest
  ```

* 利用消息总线触发一个服务端ConfigServer的/bus/refresh端点，从而刷新所有客户端配置

* 添加消息总线支持,  server和client的pom都要添加依赖：spring-cloud-starter-bus-amqp

* 配置中心服务端yml 

  ```yaml
  ##rabbitmq相关配置,暴露bus刷新配置的端点
  management:
    endpoints: #暴露bus刷新配置的端点
      web:
        exposure:
          include: 'bus-refresh'
  ```

* 客户端bootstrap.yml

  ```yaml
  # 暴露监控端点
  management:
    endpoints:
      web:
        exposure:
          include: "*"
  ```

*  配置中心的配置文件修改后，运维工程师发送post请求：curl -X POST "http://localhost:3344/actuator/bus-refresh"    一次发送，处处生效

### Bus动态刷新定点通知

指定某一实例生效而不是全部：curl -X POST "http://localhost:3344/actuator/bus-refresh/config-client:3355"

# SpringCloud Stream消息驱动

屏蔽底层消息中间件的差异，降低切换成本，统一消息的编程模型（支持rabbitmq和kafka）

![image-20200904154743907](../../../../Software/Typora/Picture/image-20200904154743907.png)

![image-20200904163659546](../../../../Software/Typora/Picture/image-20200904163659546.png)

### 消息驱动之生产者

* pom依赖：spring-cloud-starter-stream-rabbit

* yml

  ```yaml
  spring:
    application:
      name: cloud-stream-provider
    cloud:
        stream:
          binders: # 在此处配置要绑定的rabbitmq的服务信息；
            defaultRabbit: # 表示定义的名称，用于于binding整合
              type: rabbit # 消息组件类型
              environment: # 设置rabbitmq的相关的环境配置
                spring:
                  rabbitmq:
                    host: localhost
                    port: 5672
                    username: guest
                    password: guest
          bindings: # 服务的整合处理
            output: # 这个名字是一个通道的名称, 生产者消息output发送
              destination: studyExchange # 表示要使用的Exchange名称定义
              content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
              binder: defaultRabbit # 设置要绑定的消息服务的具体设置
  ```

  

* MessageProviderImpl，自定义接口和实现类，发送消息

  ```java
  @EnableBinding(Source.class) //定义消息的推送管道
  public class MessageProviderImpl implements IMessageProvider{
      @Resource
      private MessageChannel output; // 消息发送管道        
      
      @Override
      public String send(){
          String serial = UUID.randomUUID().toString(); // 自定义消息
          output.send(MessageBuilder.withPayload(serial).build()); // 发送消息
          System.out.println("*****serial: "+serial);
          return null;
      }
  }
  ```

### 消息驱动之消费者

* yml

  ```yaml
  .
    cloud:
        stream:
          binders: # 在此处配置要绑定的rabbitmq的服务信息；
            defaultRabbit: # 表示定义的名称，用于于binding整合
              type: rabbit # 消息组件类型
              environment: # 设置rabbitmq的相关的环境配置
                spring:
                  rabbitmq:
                    host: localhost
                    port: 5672
                    username: guest
                    password: guest
          bindings: # 服务的整合处理
            input: # 这个名字是一个通道的名称
              destination: studyExchange # 表示要使用的Exchange名称定义
              content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
              binder: defaultRabbit # 设置要绑定的消息服务的具体设置
  ```

* controller接收消息

  ```java
  @Component
  @EnableBinding(Sink.class)
  public class ReceiveMessageListenerController
  {
      @Value("${server.port}")
      private String serverPort;
  
      @StreamListener(Sink.INPUT)   // 监听消息
      public void input(Message<String> message)
      {
          System.out.println("消费者1号,----->接受到的消息: "+message.getPayload()+"\t  port: "+serverPort);
      }
  }
  ```


### 重复消费

![image-20200904175756341](../../../../Software/Typora/Picture/image-20200904175756341.png)

* 默认每一个消费者都是不同组，因此会重复消费，根据业务要求可通过分组避免重复消费

### 分组和持久化

微服务应用放置于同一个group中，就能够保证消息只会被其中一个应用消费一次。不同的组是可以消费的，同一个组内会发生竞争关系，只有其中一个可以消费

* 在yml配置

  ```yaml
  spring:
  	cloud:
  		stream:
  			bindings: # 服务的整合处理
                  group: xxx  # 自定义分组，根据组名分组，组内服务进行轮询消费，且持久化消费
  ```

# SpringCloud Sleuth分布式请求链路跟踪

* Sleuth负责捕捉服务间的调用关系链路，zinkins负责前台展现

* 运行zinkinsjar包。浏览器进入：http://localhost:9411/zipkin/

* **Trace**：类似于树结构的Span结合，表示一条调用链路，存在唯一标识。例如，如果你要执行一个分布式大数据的存储操作，这个Trace也许会由你的PUT请求来形成。

* **Span** ---- 基本的工作单元。无论是发送一个RPC或是向RPC发送一个响应都是一个Span。每一个Span通过一个64位ID来进行唯一标识，并通过另一个64位ID对Span所在的Trace进行唯一标识。标识调用链路来源，通俗的理解span就是一次请求信息。在zinkins上的Span名称为请求requestMapping的url

  Span能够启动和停止，他们不断地追踪自身的时间信息，当你创建了一个Span，你必须在未来的某个时刻停止它。

  提示：启动一个Trace的初始化Span被叫作 Root Span ，它的 Span ID 和 Trace Id 相同。

* **Annotation**：用来及时记录一个事件的存在。通过引入 Brave 库，我们不用再去设置一系列的特别事件，从而让 Zipkin 能够知道客户端和服务器是谁、请求是从哪里开始的、又到哪里结束。出于学习的目的，还是把这些事件在这里列举一下：

  * **cs** （Client Sent） - 客户端发起一个请求，这个注释指示了一个Span的开始。

    **sr** （Server Received） - 服务端接收请求并开始处理它，如果用 sr 时间戳减去 cs 时间戳便能看出有多少网络延迟。

    **ss**（Server Sent）- 注释请求处理完成(响应已发送给客户端)，如果用 ss 时间戳减去sr 时间戳便可得出服务端处理请求耗费的时间。

    **cr**（Client Received）- 预示了一个 Span的结束，客户端成功地接收到了服务端的响应，如果用 cr 时间戳减去 cs 时间戳便可得出客户端从服务端获得响应所需耗费的整个时间。

![image-20200904202341053](../../../../Software/Typora/Picture/image-20200904202341053.png)

### 监控链路（服务提供者和消费者都一样的配置）

* pom : spring-cloud-starter-zipkin ，里面包含了sleuth + zipkin

  ```xml
  	<dependencies>
  		<!-- Zipkin 依赖 -->
  		<dependency>
  			<groupId>org.springframework.cloud</groupId>
  			<artifactId>spring-cloud-starter-zipkin</artifactId>
  		</dependency>
  	</dependencies>
   
  	<dependencyManagement>
  		<dependencies>
  			<!-- SpringCloud 版本控制依赖 -->
  			<dependency>
  				<groupId>org.springframework.cloud</groupId>
  				<artifactId>spring-cloud-dependencies</artifactId>
  				<version>${spring-cloud.version}</version>
  				<type>pom</type>
  				<scope>import</scope>
  			</dependency>
  		</dependencies>
  	</dependencyManagement>
  ```

* yml

  ```yaml
  spring:
    application:
      name: cloud-payment-service
    zipkin:
        base-url: http://localhost:9411  # 监控展示页面uri
    sleuth:
      sampler:
      # 采样率值介于 0 到 1 之间，1 则表示全部采集
      probability: 1
  ```

































