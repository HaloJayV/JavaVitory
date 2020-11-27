[TOC]

# 微服务

* 技术维度理解：
  * 微服务化的核心就是将传统的一站式应用，根据业务拆分成一个一个的服务，彻底地去耦合,每一个微服务提供单个业务功能的服务，一个服务做一件事，
    从技术角度看就是一种小而独立的处理过程，类似进程概念，能够自行单独启动或销毁，拥有自己独立的数据库。
* 但通常而言， 微服务架构是一种架构模式或者说是一种架构风格，它提倡将单一应用程序划分成一组小的服务，每个服务运行在其独立的自己的进程中，服务之间互相协调、互相配合，为用户提供最终价值。服务之间采用轻量级的通信机制互相沟通（**通常是基于HTTP的RESTful API**）。
* 每个服务都围绕着具体业务进行构建，并且能够被独立地部署到生产环境、类生产环境等。另外，应尽量避免统一的、集中式的服务管理机制，
* 对具体的一个服务而言，应根据业务上下文，选择合适的语言、工具对其进行构建，可以有一个非常轻量级的集中式管理来协调这些服务，可以使用不同的语言来编写服务，也可以使用不同的数据存储。
* Spring是基于HTTP的RESTful API ，Dubbo基于RPC
* 强调的是服务的大小，它关注的是某一个点，是具体解决某一个问题/提供落地对应服务的一个服务应用,
  狭意的看,可以看作Eclipse里面的一个个微服务工程/或者Module

#### 微服务架构

微服务架构是⼀种架构模式，它提倡将单⼀应⽤程序划分成⼀组⼩的服务，服务之间互相协调、互相配合，为⽤户提供最终价值。每个服务运⾏在其独⽴的进程中，服务与服务间采⽤轻量级的通信机制互相协作（通常是基于HTTP协议的RESTful API）。每个服务都围绕着具体业务进⾏构建，并且能够被独⽴的部署到⽣产环境、类⽣产环境等。另外，应当尽量避免统⼀的、集中式的服务管理机制，对具体的⼀个服务⽽⾔，应根据业务上下⽂，选择合适的语⾔、⼯具对其进⾏构建。



* SpringBoot 是Spring的一套快速配置脚手架，而SpringCloud是基于SpringBoot实现的开发工具，包含很多技术、框架集合



## Spring Cloud相关基础服务组件

服务发现——Netflix Eureka （Nacos）

服务调用——Netflix Feign 

熔断器——Netflix Hystrix 

服务网关——Spring Cloud GateWay 

分布式配置——Spring Cloud Config  （Nacos）

消息总线 —— Spring Cloud Bus （Nacos）

### 技术栈

微服务条目	落地技术	备注
服务开发	Springboot、Spring、SpringMVC	
服务配置与管理	Netflix公司的Archaius、阿里的Diamond等	
服务注册与发现	Eureka、Consul、Zookeeper等	
服务调用	Rest、RPC、gRPC	
服务熔断器	Hystrix、Envoy等	
负载均衡	Ribbon、Nginx等	
服务接口调用(客户端调用服务的简化工具)	Feign等	
消息队列	Kafka、RabbitMQ、ActiveMQ等	
服务配置中心管理	SpringCloudConfig、Chef等	
服务路由（API网关）	Zuul等	
服务监控	Zabbix、Nagios、Metrics、Spectator等	
全链路追踪	Zipkin，Brave、Dapper等	
服务部署	Docker、OpenStack、Kubernetes等	
数据流操作开发包	SpringCloud Stream（封装与Redis,Rabbit、Kafka等发送接收消息）	
事件消息总线	Spring Cloud Bus	



### SpringCloud与SpringBoot

SpringCloud，基于SpringBoot提供了一套微服务解决方案，包括服务注册与发现，配置中心，全链路监控，服务网关，负载均衡，熔断器等组件，除了基于NetFlix的开源组件做高度抽象封装之外，还有一些选型中立的开源组件。

SpringCloud利用SpringBoot的开发便利性巧妙地简化了分布式系统基础设施的开发，SpringCloud为开发人员提供了快速构建分布式系统的一些工具，包括配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话等等,它们都可以用SpringBoot的开发风格做到一键启动和部署。

SpringBoot并没有重复制造轮子，它只是将目前各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过SpringBoot风格进行再封装屏蔽掉了复杂的配置和实现原理，最终给开发者留出了一套简单易懂、易部署和易维护的分布式系统开发工具包

### 关系：

SpringBoot专注于快速方便的开发单个个体微服务。

SpringCloud是关注全局的微服务协调整理治理框架，它将SpringBoot开发的一个个单体微服务整合并管理起来，
为各个微服务之间提供，配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话等等集成服务

SpringBoot可以离开SpringCloud独立使用开发项目，但是SpringCloud离不开SpringBoot，属于依赖的关系.

SpringBoot专注于快速、方便的开发单个微服务个体，SpringCloud关注全局的服务治理框架。

### 技术选型方案

![image-20200809112852453](../../../../Software/Typora/Picture/image-20200809112852453.png)

##### 最大区别：SpringCloud抛弃了Dubbo的RPC通信，采用的是基于HTTP的REST方式。

[API文档中文]: https://www.springcloud.cc/spring-cloud-dalston.html



### 配置

```yaml
server:
  port: 8001
  
mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml        # mybatis配置文件所在路径
  type-aliases-package: com.atguigu.springcloud.entities    # 所有Entity别名类所在包
  mapper-locations:
  - classpath:mybatis/mapper/**/*.xml                       # mapper映射文件
    
spring:
   application:
    name: microservicecloud-dept 
   datasource:
    type: com.alibaba.druid.pool.DruidDataSource            # 当前数据源操作类型
    driver-class-name: org.gjt.mm.mysql.Driver              # mysql驱动包
    url: jdbc:mysql://localhost:3306/cloudDB01              # 数据库名称
    username: root
    password: 123456
    dbcp2:
      min-idle: 5                                           # 数据库连接池的最小维持连接数
      initial-size: 5                                       # 初始化连接数
      max-total: 5                                          # 最大连接数
      max-wait-millis: 200                                  # 等待连接获取的最大超时时间
 
 


```



### 服务调用RestTemplate

* RestTemplate提供了多种便捷访问远程Http服务的方法， 
  是一种简单便捷的访问restful服务模板类，是Spring提供的用于访问Rest服务的客户端模板工具集
* 而RPC是基于socket的远程调用

```java
@RestController
public class DeptController_Consumer
{
    private static final String REST_URL_PREFIX = "http://localhost:8001";
    
    @Autowired
    private RestTemplate restTemplate;
    
    @RequestMapping(value="/consumer/dept/add")
    public boolean add(Dept dept)
    {
         return restTemplate.postForObject(REST_URL_PREFIX+"/dept/add", dept, Boolean.class);
    }
    
    @RequestMapping(value="/consumer/dept/get/{id}")
    public Dept get(@PathVariable("id") Long id)
    {
         return restTemplate.getForObject(REST_URL_PREFIX+"/dept/get/"+id, Dept.class);
    }
    
    @SuppressWarnings("unchecked")
    @RequestMapping(value="/consumer/dept/list")
    public List<Dept> list()
    {
         return restTemplate.getForObject(REST_URL_PREFIX+"/dept/list", List.class);
    }   
}
```

# Eureka

* Eureka是Netflix的一个子模块，也是核心模块之一。Eureka是一个基于REST的服务，用于定位服务，以实现云端中间层服务发现和故障转移。服务注册与发现对于微服务架构来说是非常重要的，有了服务发现与注册，只需要使用服务的标识符，就可以访问到服务，而不需要修改服务调用的配置文件了。功能类似于dubbo的注册中心，比如Zookeeper。

### 注册中心Module

* pom

  ```
    <dependencies>
     <!--eureka-server服务端 -->
     <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-eureka-server</artifactId>
     </dependency>
     <!-- 修改后立即生效，热部署 -->
     <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>springloaded</artifactId>
     </dependency>
     <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-devtools</artifactId>
     </dependency>
    </dependencies>
  ```

  

* 服务注册中心配置

  ```yaml
  server: 
    port: 7001
   
  eureka:
    instance:
      hostname: localhost #eureka服务端的实例名称
    client:
      register-with-eureka: false #false表示不向注册中心注册自己。
      fetch-registry: false #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
      service-url:
        defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/        #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址。
  ```

* 启动类

  ```java
  @SpringBootApplication
  @EnableEurekaServer//EurekaServer服务器端启动类,接受其它微服务注册进来
  public class EurekaServer7001_App
  {
    public static void main(String[] args)
    {
     SpringApplication.run(EurekaServer7001_App.class, args);
    }
  }
  ```

  

### 服务注册

* pom

  ```
     <!-- 将微服务provider侧注册进eureka -->
     <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-eureka</artifactId>
     </dependency>
     <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-config</artifactId>
     </dependency>
     <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-actuator</artifactId>
     </dependency>
  ```

  

* Eureka服务提供者配置

  ```yaml
  server:
    port: 8001
    
  mybatis:
    config-location: classpath:mybatis/mybatis.cfg.xml  #mybatis所在路径
    type-aliases-package: com.atguigu.springcloud.entities #entity别名类
    mapper-locations:
    - classpath:mybatis/mapper/**/*.xml #mapper映射文件
      
  spring:
     application:
      name: microservicecloud-dept 
     datasource:
      type: com.alibaba.druid.pool.DruidDataSource
      driver-class-name: org.gjt.mm.mysql.Driver
      url: jdbc:mysql://localhost:3306/cloudDB01
      username: root
      password: 123456
      dbcp2:
        min-idle: 5
        initial-size: 5
        max-total: 5
        max-wait-millis: 200
        
  eureka:
    client: #客户端注册进eureka服务列表内
      service-url: 
        defaultZone: http://localhost:7001/eureka
    instance:
      instance-id: microservicecloud-dept8001   #自定义服务名称信息
      prefer-ip-address: true     #访问路径可以显示IP地址
       
  # 微服务页面内容详细信息     
  info:
    app.name: atguigu-microservicecloud
    company.name: www.atguigu.com
    build.artifactId: $project.artifactId$
    build.version: $project.version$
  ```

* 启动类

  ```
  @SpringBootApplication
  @EnableEurekaClient //本服务启动后会自动注册进eureka服务中
  public class DeptProvider8001_App
  {
    public static void main(String[] args)
    {
     SpringApplication.run(DeptProvider8001_App.class, args);
    }
  }
  ```

  

### 自我保护机制

默认情况下，如果EurekaServer在一定时间内没有接收到某个微服务实例的心跳，EurekaServer将会注销该实例（默认90秒）。但是当网络分区故障发生时，微服务与EurekaServer之间无法正常通信，以上行为可能变得非常危险了——因为微服务本身其实是健康的，此时本不应该注销这个微服务。Eureka通过“自我保护模式”来解决这个问题——当EurekaServer节点在短时间内丢失过多客户端时（可能发生了网络分区故障），那么这个节点就会进入自我保护模式。一旦进入该模式，EurekaServer就会保护服务注册表中的信息，不再删除服务注册表中的数据（也就是不会注销任何微服务）。当网络故障恢复后，该Eureka Server节点会自动退出自我保护模式。

在自我保护模式中，Eureka Server会保护服务注册表中的信息，不再注销任何服务实例。当它收到的心跳数重新恢复到阈值以上时，该Eureka Server节点就会自动退出自我保护模式。它的设计哲学就是宁可保留错误的服务注册信息，也不盲目注销任何可能健康的服务实例。一句话讲解：好死不如赖活着

综上，自我保护模式是一种应对网络异常的安全保护措施。它的架构哲学是宁可同时保留所有微服务（健康的微服务和不健康的微服务都会保留），也不盲目注销任何健康的微服务。使用自我保护模式，可以让Eureka集群更加的健壮、稳定。

在Spring Cloud中，可以使用eureka.server.enable-self-preservation = false 禁用自我保护模式。

### 服务发现（非重点）

* 启动类加上@EnableDiscoveryClient //服务发现

```java
  @Autowired
  private DiscoveryClient client;
  
  @RequestMapping(value = "/dept/discovery", method = RequestMethod.GET)
  public Object discovery()
  {
    List<String> list = client.getServices();
    System.out.println("**********" + list);
 
    List<ServiceInstance> srvList = client.getInstances("MICROSERVICECLOUD-DEPT");
    for (ServiceInstance element : srvList) {
     System.out.println(element.getServiceId() + "\t" + element.getHost() + "\t" + element.getPort() + "\t"
         + element.getUri());
    }
    retur
```

### Eureka服务端集群配置

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
      #单机 defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/       #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址（单机）。
      defaultZone: http://eureka7002.com:7002/eureka/
```

* 修改host文件添加映射配置

```
127.0.0.1 eureka7001.com
127.0.0.1 eureka7002.com
```



* eureka客户端

  ```yaml
  eureka:
    client: #客户端注册进eureka服务列表内
      service-url: 
        defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
    instance:
      instance-id: microservicecloud-dept8001   #自定义服务名称信息
      prefer-ip-address: true     #访问路径可以显示IP地址
  
  ```

*  作为服务注册中心，Eureka比Zookeeper好在哪里

  ​    著名的CAP理论指出，一个分布式系统不可能同时满足C(一致性)、A(可用性)和P(分区容错性)。由于分区容错性P在是分布式系统中必须要保证的，因此我们只能在A和C之间进行权衡。
  因此
  Zookeeper保证的是CP,
  Eureka则是AP。

  * 4.1 Zookeeper保证CP
    当向注册中心查询服务列表时，我们可以容忍注册中心返回的是几分钟以前的注册信息，但不能接受服务直接down掉不可用。也就是说，服务注册功能对可用性的要求要高于一致性。但是zk会出现这样一种情况，当master节点因为网络故障与其他节点失去联系时，剩余节点会重新进行leader选举。问题在于，选举leader的时间太长，30 ~ 120s, 且选举期间整个zk集群都是不可用的，这就导致在选举期间注册服务瘫痪。在云部署的环境下，因网络问题使得zk集群失去master节点是较大概率会发生的事，虽然服务能够最终恢复，但是漫长的选举时间导致的注册长期不可用是不能容忍的。

  * 4.2 Eureka保证AP
    Eureka看明白了这一点，因此在设计时就优先保证可用性。Eureka各个节点都是平等的，几个节点挂掉不会影响正常节点的工作，剩余的节点依然可以提供注册和查询服务。而Eureka的客户端在向某个Eureka注册或时如果发现连接失败，则会自动切换至其它节点，只要有一台Eureka还在，就能保证注册服务可用(保证可用性)，只不过查到的信息可能不是最新的(不保证强一致性)。除此之外，Eureka还有一种自我保护机制，如果在15分钟内超过85%的节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，此时会出现以下几种情况： 

  1. Eureka不再从注册列表中移除因为长时间没收到心跳而应该过期的服务 
  2. Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上(即保证当前节点依然可用) 
  3. 当网络稳定时，当前实例新的注册信息会被同步到其它节点中

  因此， Eureka可以很好的应对因网络故障导致部分节点失去联系的情况，而不会像zookeeper那样使整个注册服务瘫痪。

  

# Ribbon负载均衡


Spring Cloud Ribbon是基于Netflix Ribbon实现的一套客户端负载均衡的工具。

简单的说，Ribbon是Netflix发布的开源项目，主要功能是提供客户端的软件负载均衡算法，将Netflix的中间层服务连接在一起。Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，就是在配置文件中列出Load Balancer（简称LB）后面所有的机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器。我们也很容易使用Ribbon实现自定义的负载均衡算法。

负载均衡分为集中式LB（硬件）和进程内LB（软件） 

* pom

  ```xml
  <!-- Ribbon相关 -->
     <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-eureka</artifactId>
     </dependency>
     <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-ribbon</artifactId>
     </dependency>
     <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-config</artifactId>
     </dependency>
  
  ```

* 服务消费者的配置类上注解

  ```java
  @Configuration
  public class ConfigBean
  {
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate()
    {
     return new RestTemplate();
    }
  }
  ```

* 服务消费者调用服务提供者

  ```
  @SpringBootApplication
  @EnableEurekaClient
  public class DeptConsumer80_App
  {
    public static void main(String[] args)
    {
     SpringApplication.run(DeptConsumer80_App.class, args);
    }
  }
  ```

  ```
  @RestController
  public class DeptController_Consumer
  {
    //private static final String REST_URL_PREFIX = "http://localhost:8001";
    private static final String REST_URL_PREFIX = "http://MICROSERVICECLOUD-DEPT";
    
    @Autowired
    private RestTemplate restTemplate;
    
    //测试@EnableDiscoveryClient,消费端可以调用服务发现
    @RequestMapping(value="/consumer/dept/discovery") 
    public Object discovery()
    {
     return restTemplate.getForObject(REST_URL_PREFIX+"/dept/discovery", Object.class);
    }  
    
  }
   
  
  ```

### 负载均衡策略

```
IRule
这是所有负载均衡策略的父接口，里边的核心方法就是choose方法，用来选择一个服务实例。

AbstractLoadBalancerRule
AbstractLoadBalancerRule是一个抽象类，里边主要定义了一个ILoadBalancer，就是我们上文所说的负载均衡器，负载均衡器的功能我们在上文已经说的很详细了，这里就不再赘述，这里定义它的目的主要是辅助负责均衡策略选取合适的服务端实例。

1、RandomRule
看名字就知道，这种负载均衡策略就是随机选择一个服务实例，看源码我们知道，在RandomRule的无参构造方法中初始化了一个Random对象，然后在它重写的choose方法又调用了choose(ILoadBalancer lb, Object key)这个重载的choose方法，在这个重载的choose方法中，每次利用random对象生成一个不大于服务实例总数的随机数，并将该数作为下标所以获取一个服务实例。

2、RoundRobinRule(默认)
RoundRobinRule这种负载均衡策略叫做线性负载均衡策略，也就是我们在上文所说的BaseLoadBalancer负载均衡器中默认采用的负载均衡策略。这个类的choose(ILoadBalancer lb, Object key)函数整体逻辑是这样的：开启一个计数器count，在while循环中遍历服务清单，获取清单之前先通过incrementAndGetModulo方法获取一个下标，这个下标是一个不断自增长的数先加1然后和服务清单总数取模之后获取到的（所以这个下标从来不会越界），拿着下标再去服务清单列表中取服务，每次循环计数器都会加1，如果连续10次都没有取到服务，则会报一个警告No available alive servers after 10 tries from load balancer: XXXX。

3、RetryRule
看名字就知道这种负载均衡策略带有重试功能。首先RetryRule中又定义了一个subRule，它的实现类是RoundRobinRule，然后在RetryRule的choose(ILoadBalancer lb, Object key)方法中，每次还是采用RoundRobinRule中的choose规则来选择一个服务实例，如果选到的实例正常就返回，如果选择的服务实例为null或者已经失效，则在失效时间deadline之前不断的进行重试（重试时获取服务的策略还是RoundRobinRule中定义的策略），如果超过了deadline还是没取到则会返回一个null。

4、WeightedResponseTimeRule
WeightedResponseTimeRule是RoundRobinRule的一个子类，在WeightedResponseTimeRule中对RoundRobinRule的功能进行了扩展，WeightedResponseTimeRule中会根据每一个实例的运行情况来给计算出该实例的一个权重，然后在挑选实例的时候则根据权重进行挑选，这样能够实现更优的实例调用。WeightedResponseTimeRule中有一个名叫DynamicServerWeightTask的定时任务，默认情况下每隔30秒会计算一次各个服务实例的权重，权重的计算规则也很简单，如果一个服务的平均响应时间越短则权重越大，那么该服务实例被选中执行任务的概率也就越大。

5、ClientConfigEnabledRoundRobinRule
ClientConfigEnabledRoundRobinRule选择策略的实现很简单，内部定义了RoundRobinRule，choose方法还是采用了RoundRobinRule的choose方法，所以它的选择策略和RoundRobinRule的选择策略一致，不赘述。

6、BestAvailableRule
BestAvailableRule继承自ClientConfigEnabledRoundRobinRule，它在ClientConfigEnabledRoundRobinRule的基础上主要增加了根据loadBalancerStats中保存的服务实例的状态信息来过滤掉失效的服务实例的功能，然后顺便找出并发请求最小的服务实例来使用。然而loadBalancerStats有可能为null，如果loadBalancerStats为null，则BestAvailableRule将采用它的父类即ClientConfigEnabledRoundRobinRule的服务选取策略（线性轮询）。

7、PredicateBasedRule
PredicateBasedRule是ClientConfigEnabledRoundRobinRule的一个子类，它先通过内部定义的一个过滤器过滤出一部分服务实例清单，然后再采用线性轮询的方式从过滤出来的结果中选取一个服务实例。

8、ZoneAvoidanceRule
ZoneAvoidanceRule是PredicateBasedRule的一个实现类，只不过这里多一个过滤条件，ZoneAvoidanceRule中的过滤条件是以ZoneAvoidancePredicate为主过滤条件和以AvailabilityPredicate为次过滤条件组成的一个叫做CompositePredicate的组合过滤条件，过滤成功之后，继续采用线性轮询的方式从过滤结果中选择一个出来。
```

* 选择策略

  ```java
  @Configuration
  public class MySelfRule
  {
    @Bean
    public IRule myRule()
    {
     return new RandomRule();//Ribbon默认是轮询，自定义为随机，也可以自己定义其他策略
    }
  }
  ```

* 在服务消费方的启动类；

```
// 在启动该微服务的时候就能去加载我们的自定义Ribbon配置类，从而使配置生效，形如：
@RibbonClient(name="MICROSERVICECLOUD-DEPT",configuration=MySelfRule.class)
```

* 官方文档明确给出了警告：
  这个自定义配置类不能放在@ComponentScan所扫描的当前包下以及子包下，否则我们自定义的这个配置类就会被所有的Ribbon客户端所共享，也就是说我们达不到特殊化定制的目的了。（原因是@SpringBootApplication这个注解里有@ComponentScan）

* 也可以自定义LB策略 (自定义策略类和选择策略类都不能在启动类所在的包里)

  ```java
  public class RandomRule_ZY extends AbstractLoadBalancerRule {
   
    private int total = 0;    //总共被调用的次数，目前要求每台被调用5次
    private int currentIndex = 0;//当前提供服务的机器号
    
      public Server choose(ILoadBalancer lb, Object key) {
          if (lb == null) {
              return null;
          }
          Server server = null;
   
          while (server == null) {
              if (Thread.interrupted()) {
                  return null;
              }
              List<Server> upList = lb.getReachableServers();
              List<Server> allList = lb.getAllServers();
   
              int serverCount = allList.size();
              if (serverCount == 0) {
                  /*
                   * No servers. End regardless of pass, because subsequent passes
                   * only get more restrictive.
                   */
                  return null;
              }
    
  //            int index = rand.nextInt(serverCount);
  //            server = upList.get(index);
              if(total < 5)
              {
                  server = upList.get(currentIndex);
                  total++;
                  }else {
                  total = 0;
                  currentIndex++;
                  if(currentIndex >= upList.size())
                  {
                    currentIndex = 0;
                  }
              
              }
  
              if (server == null) {
                  /*
                   * The only time this should happen is if the server list were
                   * somehow trimmed. This is a transient condition. Retry after
                   * yielding.
                   */
                  Thread.yield();
                  continue;
              }
   
              if (server.isAlive()) {
                  return (server);
              }
   
              // Shouldn't actually happen.. but must be transient or a bug.
              server = null;
              Thread.yield();
          }
   
          return server;
   
      }
   
    @Override
    public Server choose(Object key) {
     return choose(getLoadBalancer(), key);
    }
   
    @Override
    public void initWithNiwsConfig(IClientConfig clientConfig) {
     
    }
  }
  ```

  ```java
  @Configuration
  public class MySelfRule
  {
    @Bean
    public IRule myRule()
    {
     //return new RandomRule();//Ribbon默认是轮询，我自定义为随机
     
     return new RandomRule_ZY();//我自定义为每个机器被访问5次
    }
  }
  
  ```

  

# Feign负载均衡

* Feign是一个声明式的Web服务客户端，使得编写Web服务客户端变得非常容易，
  只需要创建一个接口，然后在上面添加注解即可。
  参考官网：https://github.com/OpenFeign/feign 

### Feign能干什么
Feign旨在使编写Java Http客户端变得更容易。
前面在使用Ribbon+RestTemplate时，利用RestTemplate对http请求的封装处理，形成了一套模版化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用。所以，Feign在此基础上做了进一步封装，由他来帮助我们定义和实现依赖服务接口的定义。在Feign的实现下，我们只需创建一个接口并使用注解的方式来配置它(以前是Dao接口上面标注Mapper注解,现在是一个微服务接口上面标注一个Feign注解即可)，即可完成对服务提供方的接口绑定，简化了使用Spring cloud Ribbon时，自动封装服务调用客户端的开发量。

* Feign集成了Ribbon
  利用Ribbon维护了MicroServiceCloud-Dept的服务列表信息，并且通过轮询实现了客户端的负载均衡。而与Ribbon不同的是，通过feign只需要定义服务绑定接口且以声明式的方法，优雅而简单的实现了服务调用

 ### 使用步骤（面向接口编程 ，Ribbon+RestTemplate是面向微服务编程）

* pom

  ```
   
     <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-feign</artifactId>
     </dependency>
  ```

* api工程创建service接口 (需要有一个接口来声明服务提供者的方法)

  ```java
  
  @FeignClient(value = "MICROSERVICECLOUD-DEPT")  // 声明服务提供者的方法
  public interface DeptClientService
  {
    @RequestMapping(value = "/dept/get/{id}",method = RequestMethod.GET)
    public Dept get(@PathVariable("id") long id);
   
    @RequestMapping(value = "/dept/list",method = RequestMethod.GET)
    public List<Dept> list();
   
    @RequestMapping(value = "/dept/add",method = RequestMethod.POST)
    public boolean add(Dept dept);
  }
  ```

* 服务消费者调用api

  ```java
  
  @RestController
  public class DeptController_Feign
  {
    @Autowired
    private DeptClientService service = null;
   
    @RequestMapping(value = "/consumer/dept/get/{id}")
    public Dept get(@PathVariable("id") Long id)
    {
     return this.service.get(id);
    }
   
    @RequestMapping(value = "/consumer/dept/list")
    public List<Dept> list()
    {
     return this.service.list();
    }
   
    @RequestMapping(value = "/consumer/dept/add")
    public Object add(Dept dept)
    {
     return this.service.add(dept);
    }
  }
  
  ```

* 服务消费者修改启动类

  ```java
   
  @SpringBootApplication
  @EnableEurekaClient
  @EnableFeignClients(basePackages= {"com.atguigu.springcloud"}) // api工程service接口所在的包
  @ComponentScan("com.atguigu.springcloud")
  public class DeptConsumer80_Feign_App
  {
    public static void main(String[] args)
    {
     SpringApplication.run(DeptConsumer80_Feign_App.class, args);
    }
  }
   
  ```

* 总结


   Feign通过接口的方法调用Rest服务（之前是Ribbon+RestTemplate），
该请求发送给Eureka服务器（http://MICROSERVICECLOUD-DEPT/dept/list）,
通过Feign直接找到服务接口，由于在进行服务调用的时候融合了Ribbon技术，所以也支持负载均衡作用。

 

 # 熔断器Histrix

​		熔断机制是应对雪崩效应的一种微服务链路保护机制。
当扇出链路的某个微服务不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回"错误"的响应信息。当检测到该节点微服务调用响应正常后恢复调用链路。在SpringCloud框架里熔断机制通过Hystrix实现。Hystrix会监控微服务间调用的状况，当失败的调用到一定阈值，缺省是5秒内20次调用失败就会启动熔断机制。熔断机制的注解是@HystrixCommand。

 ### 服务降级

备注：Hystrix服务降级，其实就是线程池中单个线程障处理，防止单个线程请求时间太长，导致资源长期被占有而得不到释放，从而导致线程池被快速占用完，导致服务崩溃。
Hystrix能解决如下问题：
1.请求超时降级，线程资源不足降级，降级之后可以返回自定义数据
2.线程池隔离降级，分布式服务可以针对不同的服务使用不同的线程池，从而互不影响
3.自动触发降级与恢复
4.实现请求缓存和请求合并

 ### 服务熔断

备注：熔断模式：这种模式主要是参考电路熔断，如果一条线路电压过高，保险丝会熔断，防止火灾。放到我们的系统中，如果某个目标服务调用慢或者有大量超时，此时，熔断该服务的调用，对于后续调用请求，不在继续调用目标服务，直接返回，快速释放资源。如果目标服务情况好转则恢复调用。

 ### 服务限流

备注：限流模式主要是提前对各个类型的请求设置最高的QPS阈值，若高于设置的阈值则对该请求直接返回，不再调用后续资源。这种模式不能解决服务依赖的问题，只能解决系统整体资源分配问题，因为没有被限流的请求依然有可能造成雪崩效应。

 ### 服务熔断

* pom

  ```
  <!--  hystrix -->
     <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-hystrix</artifactId>
     </dependency>
  ```

* 服务提供者Controller

  ```java
  
  @RestController
  public class DeptController
  {
    @Autowired
    private DeptService service = null;
    
    @RequestMapping(value="/dept/get/{id}",method=RequestMethod.GET)
      @HystrixCommand(fallbackMethod = "processHystrix_Get")
    public Dept get(@PathVariable("id") Long id)
    {
     Dept dept =  this.service.get(id);
     if(null == dept)
     {
       throw new RuntimeException("该ID："+id+"没有没有对应的信息");
     }
     return dept;
    }
    
    public Dept processHystrix_Get(@PathVariable("id") Long id)
    {
     return new Dept().setDeptno(id)
             .setDname("该ID："+id+"没有没有对应的信息,null--@HystrixCommand")
             .setDb_source("no this database in MySQL");
    }
  }
  
  ```

### 服务降级

* 在api工程中新建接口实现FallbackFactory，专门来处理服务降级和熔断@HystrixCommand处理

  ```java
  @Component//不要忘记添加，不要忘记添加
  public class DeptClientServiceFallbackFactory implements FallbackFactory<DeptClientService> //解耦，相当于@HystrixCommand
  {
    @Override
    public DeptClientService create(Throwable throwable)
    {
     return new DeptClientService() {
       @Override
       public Dept get(long id)
       {
         return new Dept().setDeptno(id)
                 .setDname("该ID："+id+"没有没有对应的信息,Consumer客户端提供的降级信息,此刻服务Provider已经关闭")
                 .setDb_source("no this database in MySQL");
       }
   
       @Override
       public List<Dept> list()
       {
         return null;
       }
   
       @Override
       public boolean add(Dept dept)
       {
         return false;
       } 
     };
    }
  }
   
  ```

* 在api工程的clientService接口修改注解，用于绑定统一处理fallback的DeptClientServiceFallbackFactory类

  ```java
  
  @FeignClient(value = "MICROSERVICECLOUD-DEPT",fallbackFactory=DeptClientServiceFallbackFactory.class)
  public interface DeptClientService
  {
    @RequestMapping(value = "/dept/get/{id}",method = RequestMethod.GET)
    public Dept get(@PathVariable("id") long id);
   
    @RequestMapping(value = "/dept/list",method = RequestMethod.GET)
    public List<Dept> list();
   
    @RequestMapping(value = "/dept/add",method = RequestMethod.POST)
    public boolean add(Dept dept);
  }
  ```

### 准实时的调用监控（Hystrix Dashboard）


除了隔离依赖服务的调用以外，Hystrix还提供了准实时的调用监控（Hystrix Dashboard），Hystrix会持续地记录所有通过Hystrix发起的请求的执行信息，并以统计报表和图形的形式展示给用户，包括每秒执行多少请求多少成功，多少失败等。Netflix通过hystrix-metrics-event-stream项目实现了对以上指标的监控。Spring Cloud也提供了Hystrix Dashboard的整合，对监控内容转化成可视化界面。

* 服务消费方pom

  ```
     <!-- hystrix和 hystrix-dashboard相关-->
     <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-hystrix</artifactId>
     </dependency>
     <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
     </dependency> 
  
  ```

* 服务消费方的启动类加上 @EnableHystrixDashboard

* 服务提供方pom

  ```
  <!-- actuator监控信息完善 -->
     <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
     </dependency>
  ```

* 直接在浏览器地址栏输入 http://localhost:9001/hystrix   ，其中9001为服务消费方的port

* 在浏览器地址栏输入 http://localhost:8001/hystrix.stream   ，其中8001为服务消费方的port



# zuul路由网关

### 基本介绍

Zuul包含了对请求的路由和过滤两个最主要的功能：
其中路由功能负责将外部请求转发到具体的微服务实例上，是实现外部访问统一入口的基础而过滤器功能则负责对请求的处理过程进行干预，是实现请求校验、服务聚合等功能的基础.Zuul和Eureka进行整合，将Zuul自身注册为Eureka服务治理下的应用，同时从Eureka中获得其他微服务的消息，也即以后的访问微服务都是通过Zuul跳转后获得。

注意：Zuul服务最终还是会注册进Eureka

提供=代理+路由+过滤三大功能

 ### 使用

新建工程用于zuul网关路由

* pom

  ```
     <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-zuul</artifactId>
     </dependency>
  ```

* yml

  ```yaml
  server: 
    port: 9527
   
  spring: 
    application:
      name: microservicecloud-zuul-gateway
   
  eureka: 
    client: 
      service-url: 
        defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka  
    instance:
      instance-id: gateway-9527.com
      prefer-ip-address: true 
   
   
  info:
    app.name: atguigu-microcloud
    company.name: www.atguigu.com
    build.artifactId: $project.artifactId$
    build.version: $project.version$
  
  ```

* host文件修改：127.0.0.1   myzuul.com

* 启动类上加上

  ```
  @SpringBootApplication
  @EnableZuulProxy
  ```

![image-20200811205619580](../../../../Software/Typora/Picture/image-20200811205619580.png)

 ### 映射规则

* yml

  映射前：  http://myzuul.com:9527/microservicecloud-dept/dept/get/2

  ```yaml
  zuul: 
    prefix: /atguigu   # 设置统一公共前缀
    ignored-services: microservicecloud-dept   # 禁用原真实服务名， 也可以设置 '*' 
    routes: 
      mydept.serviceId: microservicecloud-dept
      mydept.path: /mydept/**
  ```

  映射后：  http://myzuul.com:9527/mydept/dept/get/1

 

 # SpringCloud Config 分布式配置中心

 ### 基本介绍

SpringCloud Config为微服务架构中的微服务提供集中化的外部配置支持，配置服务器为各个不同微服务应用的所有环境提供了一个中心化的外部配置。怎么玩

SpringCloud Config分为服务端和客户端两部分。

服务端也称为分布式配置中心，它是一个独立的微服务应用，用来连接配置服务器并为客户端提供获取配置信息，加密/解密信息等访问接口

客户端则是通过指定的配置中心来管理应用资源，以及与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息配置服务器默认采用git来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过git客户端工具来方便的管理和访问配置内容。

### 服务端配置

在github上新建仓库，并 git clone到本地文件夹，在改文件夹内新建application.yml，保存格式必须为utf-8

```yaml
spring:
  profiles:
    active:
    - dev
---
spring:
  profiles: dev     #开发环境   http://config-3344.com:3344/application-dev.yml    运行生成环境配置
  application: 
    name: microservicecloud-config-atguigu-dev
---
spring:
  profiles: test   #测试环境   http://config-3344.com:3344/application-test.yml   运行测试环境配置
  application: 
    name: microservicecloud-config-atguigu-test
#  请保存为UTF-8格式
 
 

```

* git到github：

  ```
  git add .
  git commit -m "init yml"
  git push origin master
  ```

* 新建module（Config Server），用于与github关联，管理配置信息

* pom

  ```
  <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-config-server</artifactId>
  </dependency> 
  ```

* 服务端工程的application.yml

  ```yaml
  server: 
    port: 3344 
    
  spring:
    application:
      name:  microservicecloud-config
    cloud:
      config:
        server:
          git:
            uri: git@github.com:zzyybs/microservicecloud-config.git #GitHub上面的git仓库名字
  ```

* 启动类加上注解 @EnableConfigServer

* 修改host文件：127.0.0.1  config-3344.com

##### 测试 (与github上的application.yml里的profile相关)

* http://config-3344.com:3344/application-dev.yml    运行生产环境配置
* http://config-3344.com:3344/application-test.yml   运行测试环境配置

 

### 客户端配置

* 需要提交到github上的microservicecloud-config-client.yml

  ```yaml
  spring:
    profiles:
      active:
      - dev
  ---
  server: 
    port: 8201 
  spring:
    profiles: dev
    application: 
      name: microservicecloud-config-client
  eureka: 
    client: 
      service-url: 
        defaultZone: http://eureka-dev.com:7001/eureka/   
  ---
  server: 
    port: 8202 
  spring:
    profiles: test
    application: 
      name: microservicecloud-config-client
  eureka: 
    client: 
      service-url: 
        defaultZone: http://eureka-test.com:7001/eureka/
  ```

* 新建工程

* pom

  ```
   
     <!-- SpringCloud Config客户端 -->
     <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-config</artifactId>
     </dependency> 
  
  ```

* bootstrap.yml

  ```yaml
  spring:
    cloud:
      config:
        name: microservicecloud-config-client #需要从github上读取的资源名称，注意没有yml后缀名
        profile: dev   #本次访问的配置项
        label: master   
        uri: http://config-3344.com:3344  #本微服务启动后先去找3344号服务，通过SpringCloudConfig获取GitHub的服务地址
  ```

* application.yml

  ```
  spring:
    application:
      name: microservicecloud-config-client
  ```

* 修改host文件

* 读取github上yml文件的配置信息

  ```java
  
  @RestController
  public class ConfigClientRest {
   
    @Value("${spring.application.name}")  // 取得github上的microservicecloud-config-client.yml配置信息
    private String applicationName;
    
    @Value("${eureka.client.service-url.defaultZone}")
    private String eurekaServers;
    
    @Value("${server.port}")
    private String port;
    
    @RequestMapping("/config")
    public String getConfig()
    {
     String str = "applicationName: "+applicationName+"\t eurekaServers:"+eurekaServers+"\t port: "+port;
     System.out.println("******str: "+ str);
     return "applicationName: "+applicationName+"\t eurekaServers:"+eurekaServers+"\t port: "+port;
    }
  }
  ```

  ### 架构

![image-20200812002338939](../../../../Software/Typora/Picture/image-20200812002338939.png)

 

 ![image-20200812002405226](../../../../Software/Typora/Picture/image-20200812002405226.png)

 













