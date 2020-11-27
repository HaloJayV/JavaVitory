[TOC]

# Nacos

## 1、基本概念

简单来讲就是 Nacos = Spring Cloud Eureka + Spring Cloud Config

#### 常见的注册中心：

\1. Eureka（原生，2.0遇到性能瓶颈，停止维护）

\2. Zookeeper（支持，专业的独立产品。例如：dubbo）

\3. Consul（原生，GO语言开发）

\4. Nacos

相对于 Spring Cloud Eureka 来说，Nacos 更强大。

 Nacos 可以与 Spring, Spring Boot, Spring Cloud 集成，并能代替 Spring Cloud Eureka, Spring Cloud Config

通过 Nacos Server 和 spring-cloud-starter-alibaba-nacos-discovery 实现服务的注册与发现。

**（3）**Nacos是以服务为主要服务对象的中间件，Nacos支持所有主流的服务发现、配置和管理。

Nacos主要提供以下四大功能：

\1. 服务发现和服务健康监测

\2. 动态配置服务

\3. 动态DNS服务

\4. 服务及其元数据管理

**（4）**Nacos结构图

![image-20200916191839059](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916191839059.png)

## 2、Nacos下载和安装

**（1）下载地址和版本**

下载地址：https://github.com/alibaba/nacos/releases

下载版本：nacos-server-1.1.4.tar.gz或nacos-server-1.1.4.zip，解压任意目录即可

## （2）启动nacos服务

\- Linux/Unix/Mac

启动命令(standalone代表着单机模式运行，非集群模式)

启动命令：sh startup.sh -m standalone



\- Windows

启动命令：cmd startup.cmd 或者双击startup.cmd运行文件。

访问：http://localhost:8848/nacos

用户名密码：nacos/nacos

![image-20200916191834640](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916191834640.png)

# 服务注册

把service-edu微服务注册到注册中心中，service-vod步骤相同

相当于Eureka + Config + Bus，  和Eureka一样都是AP模型（只支持注册临时实例+高可用+分区分治），但支持CP和AP切换

## 1、在service模块配置pom

配置Nacos客户端的pom依赖

```
<!--服务注册-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

## 2、添加服务配置信息

配置application.properties，在客户端微服务中添加注册Nacos服务的配置信息

```
# nacos服务地址
spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
```

**3、添加Nacos客户端注解**

在客户端微服务启动类中添加注解

```
@EnableDiscoveryClient
```

## **4、启动客户端微服务**

启动注册中心

启动已注册的微服务，可以在Nacos服务列表中看到被注册的微服务

![image-20200916191826462](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916191826462.png)



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

# Nacos服务注册与发现原理

- Provider APP：服务提供者
- Consumer APP：服务消费者
- Name Server：通过VIP（Virtual IP）或DNS的方式实现Nacos高可用集群的服务路由
- Nacos Server：Nacos服务提供者，里面包含的Open API是功能访问入口，Conig Service、Naming Service 是Nacos提供的配置服务、命名服务模块。Consitency Protocol是一致性协议，用来实现Nacos集群节点的数据同步，这里使用的是Raft算法（Etcd、Redis哨兵选举）
- Nacos Console：控制台

### Nacos 服务注册需要具备的能力：

- 服务提供者把自己的协议地址注册到Nacos server
- 服务消费者需要从Nacos Server上去查询服务提供者的地址（根据服务名称）
- Nacos Server需要感知到服务提供者的上下线的变化
- 服务消费者需要动态感知到Nacos Server端服务地址的变化（动态刷新）

## 服务注册流程

在基于Dubbo服务发布的过程中， 自动装配是走的事件监听机制，在 DubboServiceRegistrationNonWebApplicationAutoConfiguration 这个类中，这个类会监听 ApplicationStartedEvent 事件，这个事件是spring boot在2.0新增的，就是当spring boot应用启动完成之后会发布这个事件。而此时监听到这个事件之后，会触发注册的动作。

* this.serviceRegistry。 是spring-cloud提供的接口实现(org.springframework.cloud.client.serviceregistry.ServiceRegistry).很显然注入的实例是： NacosServiceRegistry。然后进入到实现类的注册方法
* 接下去就是开始注册实例
  * 如果当前注册的是临时节点，则构建心跳信息，通过beat反应堆来构建心跳任务.
  * 调用registerService发起服务注册
* 然后调用 NamingProxy 的注册方法进行注册，代码逻辑很简单，构建请求参数，发起请求。
* 往下走我们就会发现上面提到的，服务在进行注册的时候会轮询配置好的注册中心的地址：
* 最后通过 callServer(api, params, server, method) 发起调用，这里通过 JSK自带的 HttpURLConnection 进行发起调用。我们可以通过断点的方式来看到这里的请求参数：
* 期间可能会有多个 GET的请求获取服务列表，是正常的，会发现有如上的一个请求，会调用 http://192.168.200.1:8848/nacos/v1/ns/instance 这个地址。那么接下去就是Nacos Server 接受到服务端的注册请求的处理流程。

总结：

- 服务实例在启动时注册到服务注册表，并在关闭时注销
- 服务消费者查询服务注册表，获得可用实例
- 服务注册中心需要调用服务实例的健康检查API来验证它是否能够处理请求

　　![image-20200916191813135](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916191813135.png)

在Nacos 中，服务注册时在服务端本地会通过轮询注册中心集群节点地址进行服务的注册，在注册中心上，即Nacos Server上采用了concurrentHashMap保存实例信息，当然配置了持久化的服务会被保存到数据库中，在服务的调用方，为了保证本地服务实例列表的动态感知，Nacos与其他注册中心不同的是，采用了 Pull/Push同时运作的方式。

## Nacos注册服务端的处理

* 服务端提供了一个InstanceController类，在这个类中提供了服务注册相关的API，而服务端发起注册时，调用的接口是： [post]: /nacos/v1/ns/instance ，serviceName: 代表客户端的项目名称 ，namespace: nacos 的namespace。
* 然后调用 ServiceManager 进行服务的注册
* 在创建空的服务实例的时候我们发现了存储实例的map
* 在 getService 方法中我们发现了Map：
* 通过注释我们可以知道，Nacos是通过不同的 namespace 来维护服务的，而每个namespace下有不同的group，不同的group下才有对应的Service ，再通过这个 serviceName 来确定服务实例。
* 第一次进来则会进入初始化，初始化完会调用 putServiceAndInit
* 获取到服务以后把服务实例添加到集合中，然后基于一致性协议进行数据的同步。然后调用 addInstance
* 然后给服务注册方发送注册成功的响应

## SpringCloud集成Nacos

Nacos是通过Spring的事件机制继承到SpringCloud中去的。

AbstractAutoServiceRegistration实现了onApplicationEvent抽象方法,并且监听WebServerInitializedEvent事件(当Webserver初始化完成之后) , 调用this.bind ( event )方法。

## 心跳机制

从上述代码看,所谓心跳机制就是客户端通过schedule定时向服务端发送一个数据包 ,然后启动-个线程不断检测服务端的回应,如果在设定时间内没有收到服务端的回应,则认为服务器出现了故障。Nacos服务端会根据客户端的心跳包不断更新服务的状态。

## 注册原理

Nacos客户端通过Open API的形式发送服务注册请求

底层都是基于HTTP协议完成请求的。所以注册服务就是发送一个HTTP请求：

- 从请求参数汇总获得serviceName（服务名）和namespaceId（命名空间Id）
- 调用registerInstance注册实例
- 创建一个控服务（在Nacos控制台“服务列表”中展示的服务信息），实际上是初始化一个serviceMap，它是一个ConcurrentHashMap集合
- getService，从serviceMap中根据namespaceId和serviceName得到一个服务对象
- 调用addInstance添加服务实例
- 根据namespaceId、serviceName从缓存中获取Service实例
- 如果Service实例为空，则创建并保存到缓存中
- 通过putService()方法将服务缓存到内存
- service.init()建立心跳机制
- consistencyService.listen实现数据一致性监听
- 继续调用addInstance方法把当前注册的服务实例保存到Service中：

### 总结：

Nacos服务端收到请求后，做以下三件事：

1. 构建一个Service对象保存到ConcurrentHashMap集合中
2. 使用定时任务对当前服务下的所有实例建立心跳检测机制
3. 基于数据一致性协议服务数据进行同步



# 服务发现-消费

服务注册到注册中心后，服务的消费者就可以进行服务发现的流程了，消费者可以直接向注册中心发送获取某个服务实例的请求，这种情况下注册中心将返回所有可用的服务实例给消费者，但是一般不推荐这种情况。另一种方法就是服务的消费者向注册中心订阅某个服务，并提交一个监听器，当注册中心中服务发生变更时，监听器会收到通知，这时消费者更新本地的服务实例列表，以保证所有的服务均是可用的。

NacosDiscoveryClient 中提供了一个 getInstances 方法用来根据服务提供者名称获取服务提供者的url地址的方法.

### 客户端启动获取服务列表：

* 然后回调用 NacosServiceDiscovery 的 getInstances 方法，将我们所配置的 group 、serviceId 传过去，获取基于该serviceId的实例列表。

* 调用NamingService，通过updateServiceNow方法根据serviceId、group获得服务实例列表。然后把instance转化为ServiceInstance对象

* NacosNamingService.selectInstances 首先从 hostReactor 获取 serviceInfo，然后再从serviceInfo.getHosts()剔除非 healty、非enabled、weight小于等于0的 instance 再返回；如果subscribe为true，则执行 hostReactor.getServiceInfo获取serviceInfo，否则执行

  hostReactor.getServiceInfoDirectlyFromServer 获取serviceInfo

### Nacos Server 处理消费端请求：

* 通过上面消费端的请求 URL，我们可以定位到服务端源码的 InstanceController 的对应 GET请求的列表获取接口：.
* 就跟查询数据库一样，现在有参数了
* 经过这么一系列操作以后，服务消费者就能获取到相应的服务实例集合了。

### 服务动态更新：

![image-20200918171423629](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200918171423629.png)

* 基于上面的分析，服务消费者对于服务实例的动态更新主要来源于两个地方，第一个就是本地的定时任务，第二个就是采用服务端的 Push 机制

* 　**pull 定时任务请求更新服务信息：**

    　　在查询服务调用 getServiceInfo 方法的代码中，会开启一个定时任务，这个任务会在默认在1s之后开始执行。而任务的具体实现是一个UpdateTask。

* 　**push请求推送数据:**

    　　还记得在服务提供者发起服务注册时。在 createEmptyService 方法中，会创建一个空的服务.并且在这个创建过程中，调用了一个 putServiceAndInit ，这个方法中除了创建空的服务并且初始化，还会调用 service.init 方法进行服务的初始化。

*  service.init方法，会和当前服务提供者建立一个心跳检测机制，这个心跳检测会每5s执行一次。然后来看 ClientBeatCheckTask.run

* getPushService().serviceChanged(service) 会发布一个服务变更事件：

*  PushService 类实现了 ApplicationListener<ServiceChangeEvent> 所以本身又会取监听该事件，监听服务状态变更事件，然后遍历所有的客户端，通过udp协议进行消息的广播通知：

* 那么服务消费者此时应该是建立了一个udp服务的监听，否则服务端无法进行数据的推送。这个监听是在HostReactor的构造方法中初始化的

* new PushReceiver(this) 把this 传进去，初始化了一个DatagramSocket，这是一个Udp的socket连接，开启一个线程，定时执行当前任务

*  PushReceiver 的 Run 方法：在run方法中，不断循环监听服务端的push请求。然后调用 processServiceJSON 对服务端的数据进行*解析。*



## Nacos服务地址动态感知原理

Nacos客户端中有一个HostReactor类，它的功能是实现服务的动态更新，基本原理是：

- 客户端发起时间订阅后，在HostReactor中有一个UpdateTask线程，每10s发送一次Pull请求，获得服务端最新的地址列表

- 对于服务端，它和服务提供者的实例之间维持了心跳检测，一旦服务提供者出现异常，则会发送一个Push消息给Nacos客户端，也就是服务端消费者

- 服务消费者收到请求之后，使用HostReactor中提供的processServiceJSON解析消息，并更新本地服务地址列表

  



# 配置中心

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

![image-20200904231143004](D:/JAVA/Java_Learning/Software/Typora/Picture/image-20200904231143004.png)

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

![image-20200905000009508](D:/JAVA/Java_Learning/Software/Typora/Picture/image-20200905000009508.png)

![image-20200905000105474](D:/JAVA/Java_Learning/Software/Typora/Picture/image-20200905000105474.png)

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

* application.yml选择读取的配置文件

  ```yaml
  spring:
    profiles:
      active: dev # 表示开发环境
      #active: test # 表示测试环境
      #active: info
  ```

* bootstarp.properties，获取oss.yml的配置信息

  ```properties
  spring.application.name=gulimall-third-party
  spring.cloud.nacos.config.server-addr=127.0.0.1:8848
  spring.cloud.nacos.config.namespace=4380e303-f736-4fe8-99e9-b5e842506888
  
  
  spring.cloud.nacos.config.ext-config[0].data-id=oss.yml
  spring.cloud.nacos.config.ext-config[0].group=DEFAULT_GROUP
  spring.cloud.nacos.config.ext-config[0].refresh=true
  ```

### 从nacos获取配置信息

```java
@RestController
@RequestMapping("/demo")
public class DemoController {

    //动态更新需要用到这个对象
    @Autowired
    private ConfigurableApplicationContext applicationContext;

    //直接通过@Value注解就能获取nacos配置中心的数据，但这种写法不能实现动态更新
    @Value(value = "${name}")
    private String name;

    @GetMapping("/test")
    public String test(){
        return "test " + applicationContext.getEnvironment().getProperty("name");
    }
}
```



## Nacos集群和持久化配置(重要)

![image-20200905110415450](D:/JAVA/Java_Learning/Software/Typora/Picture/image-20200905110415450.png)

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

* 备份后编辑Nacos的启动脚本startup.sh,使他能够接受不同的启动端口![image-20200905122953529](D:/JAVA/Java_Learning/Software/Typora/Picture/image-20200905122953529.png)

  ![image-20200905123237668](D:/JAVA/Java_Learning/Software/Typora/Picture/image-20200905123237668.png)

* nginx.conf备份后修改 

  ![image-20200905123519126](D:/JAVA/Java_Learning/Software/Typora/Picture/image-20200905123519126.png)

  按照指定配置文件启动nginx：./nginx -c /usr/local/nginx/conf/nginx.conf

* 根据端口启动3个命令： `./startup.sh -p 3333`  、  `./startup.sh -p 4444` 、  `./startup.sh -p 5555`

* 测试通过nginx访问nacos： http://192.168.111.144:1111/nacos/#/login

* 微服务springalibaba-provider-payment9002启动注册进nacos集群， 修改配置yml文件中的nacos地址：server-addr: 192.168.111.144.1111



# Nacos配置中心原理

参考：https://www.cnblogs.com/wuzhenzhao/p/11385079.html

### 配置类型 ：

　　Spring Cloud Alibaba Nacos Config 目前提供了三种配置能力从 Nacos 拉取相关的配置。当三种方式共同使用时，他们的一个优先级关系是:A < B < C。

- A: 通过 spring.cloud.nacos.config.shared-configs[n].data-id 支持多个共享 Data Id 的配置。
- B: 通过 spring.cloud.nacos.config.extension-configs[n].data-id 的方式支持多个扩展Data Id 的配置。
- C: 通过内部相关规则(应用名、应用名+ Profile )自动生成相关的 Data Id 配置。

#### 基于dataid为 yaml 的文件扩展配置：

　　spring-cloud-starter-alibaba-nacos-config 默认支持的文件格式是 properties, 如果我们想用其他格式的文件，可以只需要完成以下两步：

1、在应用的 bootstrap.properties 配置文件中显示的声明 dataid 文件扩展名。如下所示 bootstrap.properties

```
spring.cloud.nacos.config.file-extension=yaml
```

2、在Nacos控制台，修改配置文件的类型，改成yml。

### 针对profile粒度配置：

　　spring-cloud-starter-alibaba-nacos-config 在加载配置的时候，不仅仅加载了以 dataid 为 ${spring.application.name}.${file-extension:properties} 为前缀的基础配置，还加载了dataid为 ${spring.application.name}-${profile}.${file-extension:properties} 的基础配置。在日常开发中如果遇到多套环境下的不同配置，可以通过Spring 提供的 ${spring.profiles.active} 这个配置项来配置。

1.在bootstrap.properties中添加profile

```
spring.profiles.active=develop
```

2. Nacos 上新增一个dataid为：wuzz-nacos-dubbo-consumer-develop.yaml 的基础配置，如下所示：

　如果需要切换到生产环境，只需要更改 ${spring.profiles.active} 参数配置即可。如下所示：

```
spring.profiles.active=product
```

　　此案例中我们通过 spring.profiles.active= 的方式写死在配置文件中，而在真正的项目实施过程中这个变量的值是需要不同环境而有不同的值。这个时候通常的做法是通过 -Dspring.profiles.active= 参数指定其配置来达到环境间灵活的切换。

### Nacos 中的Namespace和Group ：

　　在nacos中提供了namespace和group命名空间和分组的机制，它是Nacos提供的一种数据模型，也就是我们要去定位到一个配置，需要基于namespace- > group ->dataid来实现。

　　namespace可以解决多环境以及多用户数据的隔离问题。比如在多套环境下，可以根据指定环境创建不同的namespace，实现多环境隔离。或者在多租户的场景中，每个用户可以维护自己的namespace，实现每个用户的配置数据和注册数据的隔离。

　　group是分组机制，它的纬度是实现服务注册信息或者DataId的分组管理机制，对于group的用法，没有固定的规则，它也可以实现不同环境下的分组，也可以实现同一个应用下不同配置类型或者不同业务类型的分组。

　　官方建议是，namespace用来区分不同环境，group可以专注在业务层面的数据分组。实际上在使用过程中，最重要的是提前定要统一的口径和规定，避免不同的项目团队混用导致后期维护混乱的问题。

### 自定义namespace： 

 　在没有明确指定 ${spring.cloud.nacos.config.namespace} 配置的情况下， 默认使用的是 Nacos上 Public 这个namespae。如果需要使用自定义的命名空间，可以通过以下配置来实现：

```
spring.cloud.nacos.config.namespace=b3404bc0-d7dc-4855-b519-570ed34b62d7
```

　　该配置必须放在 bootstrap.properties 文件中。此外 spring.cloud.nacos.config.namespace 的值是 namespace 对应的 id，id 值可以在 Nacos的控制台获取。并且在添加配置时注意不要选择其他的 namespae，否则将会导致读取不到正确的配置。

### 自定义group ：

　　在没有明确指定 ${spring.cloud.nacos.config.group} 配置的情况下， 默认使用的是DEFAULT_GROUP 。如果需要自定义自己的 Group，可以通过以下配置来实现：

```
spring.cloud.nacos.config.group=DEVELOP_GROUP
```

 　该配置必须放在 bootstrap.properties 文件中。并且在添加配置时 Group 的值一定要和spring.cloud.nacos.config.group 的配置值一致

### 自定义扩展的DataId：

　　Spring Cloud Alibaba Nacos Config 从 0.2.1 版本后，可支持自定义 Data Id 的配置。关于这部分详细的设计可参考 [这里](https://github.com/alibaba/spring-cloud-alibaba/issues/141)。 bootstrap.properties:

```properties
# config external configuration
# 1、Data Id 在默认的组 DEFAULT_GROUP,不支持配置的动态刷新
spring.cloud.nacos.config.extension-configs[0].data-id=ext-config-common01.properties
# 2、Data Id 不在默认的组，不支持动态刷新
spring.cloud.nacos.config.extension-configs[1].data-id=ext-config-common02.properties
spring.cloud.nacos.config.extension-configs[1].group=GLOBALE_GROUP
# 3、Data Id 既不在默认的组，也支持动态刷新
spring.cloud.nacos.config.extension-configs[2].data-id=ext-config-common03.properties
spring.cloud.nacos.config.extension-configs[2].group=REFRESH_GROUP
spring.cloud.nacos.config.extension-configs[2].refresh=true
```

　　可以看到:

1. 通过 spring.cloud.nacos.config.extension-configs[n].data-id 的配置方式来支持多个Data Id 的配置。
2. 通过 spring.cloud.nacos.config.extension-configs[n].group 的配置方式自定义 Data Id所在的组，不明确配置的话，默认是 DEFAULT_GROUP。
3. 通过 spring.cloud.nacos.config.extension-configs[n].refresh 的配置方式来控制该Data Id 在配置变更时，是否支持应用中可动态刷新， 感知到最新的配置值。默认是不支持的。

　　多个 Data Id 同时配置时，他的优先级关系是 spring.cloud.nacos.config.extension-configs[n].data-id 其中 n 的值越大，优先级越高。

　　spring.cloud.nacos.config.extension-configs[n].data-id 的值必须带文件扩展名，文件扩展名既可支持 properties，又可以支持 yaml/yml。 此时spring.cloud.nacos.config.file-extension 的配置对自定义扩展配置的 Data Id 文件扩展名没有影响。

　　 通过自定义扩展的 Data Id 配置，既可以解决多个应用间配置共享的问题，又可以支持一个应用有多个配置文件。

　　为了更加清晰的在多个应用间配置共享的 Data Id ，你可以通过以下的方式来配置：通过自定义扩展的 Data Id 配置，既可以解决多个应用间配置共享的问题，又可以支持一个应用有多个配置文件。 bootstrap.properties:

```properties
# 配置支持共享的 Data Id
spring.cloud.nacos.config.shared-configs[0].data-id=common.yaml
# 配置 Data Id 所在分组，缺省默认 DEFAULT_GROUP
spring.cloud.nacos.config.shared-configs[0].group=GROUP_APP1
# 配置Data Id 在配置变更时，是否动态刷新，缺省默认 false
spring.cloud.nacos.config.shared-configs[0].refresh=true
```

　　可以看到：

1. 通过 spring.cloud.nacos.config.shared-configs[n].data-id 来支持多个共享 Data Id 的配置。
2. 通过 spring.cloud.nacos.config.shared-configs[n].group 来配置自定义 Data Id 所在的组，不明确配置的话，默认是 DEFAULT_GROUP。
3. 通过 spring.cloud.nacos.config.shared-configs[n].refresh 来控制该Data Id在配置变更时，是否支持应用中动态刷新，默认false。

## 源码解析：

　Environment，这个是非常重要的类，他负责管理spring的运行相关的配置信息，其中就包含application.properties。

　　而在Spring Cloud中，如果集成Nacos作为配置中心的话，那么意味着这部分配置是属于远程配置，也会作为配置源保存到Environment中，这样我们才能通过@value注解来注入配置中的属性。

　　Environment中所有外部化配置，针对不同类型的配置都会有与之对应的PropertySource，比如（SystemEnvironmentPropertySource、CommandLinePropertySource）。以及PropertySourcesPropertyResolver来进行解析。

　　那NacosClient在启动的时候，必然也会需要从远程服务器上获取配置加载到Environment中，这样才能使得应用程序通过@value进行属性的注入，而且我们一定可以猜测到的是，这块的工作一定又和spring中某个机制有关系。

　　在spring boot项目启动时，有一个prepareContext的方法，它会回调所有实现了 ApplicationContextInitializer 的实例，来做一些初始化工作。

　　Environment，这个是非常重要的类，他负责管理spring的运行相关的配置信息，其中就包含application.properties。

　　而在Spring Cloud中，如果集成Nacos作为配置中心的话，那么意味着这部分配置是属于远程配置，也会作为配置源保存到Environment中，这样我们才能通过@value注解来注入配置中的属性。

　　Environment中所有外部化配置，针对不同类型的配置都会有与之对应的PropertySource，比如（SystemEnvironmentPropertySource、CommandLinePropertySource）。以及PropertySourcesPropertyResolver来进行解析。

　　那NacosClient在启动的时候，必然也会需要从远程服务器上获取配置加载到Environment中，这样才能使得应用程序通过@value进行属性的注入，而且我们一定可以猜测到的是，这块的工作一定又和spring中某个机制有关系。

　　在spring boot项目启动时，有一个prepareContext的方法，它会回调所有实现了 ApplicationContextInitializer 的实例，来做一些初始化工作。



### 源码总结;

- 客户端发起长轮训请求，
- 服务端收到请求以后，先比较服务端缓存中的数据是否相同，如果不通，则直接返回
- 如果相同，则通过schedule延迟29.5s之后再执行比较
- 为了保证当服务端在29.5s之内发生数据变化能够及时通知给客户端，服务端采用事件订阅的方式来监听服务端本地数据变化的事件，一旦收到事件，则触发DataChangeTask的通知，并且遍历allStubs队列中的ClientLongPolling,把结果写回到客户端，就完成了一次数据的推送
- 如果 DataChangeTask 任务完成了数据的 “推送” 之后，ClientLongPolling 中的调度任务又开始执行了怎么办呢？很简单，只要在进行 “推送” 操作之前，先将原来等待执行的调度任务取消掉就可以了，这样就防止了推送操作写完响应数据之后，调度任务又去写响应数据，这时肯定会报错的。所以，在ClientLongPolling方法中，最开始的一个步骤就是删除订阅事件

　　所以总的来说，Nacos采用推+拉的形式，来解决最开始关于长轮训时间间隔的问题。当然，30s这个时间是可以设置的，而之所以定30s，应该是一个经验值。



# Nacos集群选举原理

Nacos的集群类似于zookeeper， 它分为leader角色和follower角色， 那么从这个角色的名字可以看出来，这个集群存在选举的机制。 因为如果自己不具备选举功能，角色的命名可能就是master/slave了.

　Nacos集群采用 **raft** 算法来实现，它是相对zookeeper的选举算法较为简单的一种。选举算法的核心在 RaftCore 中，包括数据的处理和数据同步。

### 选举算法 ：

　　Nacos集群采用 **raft** 算法来实现，它是相对zookeeper的选举算法较为简单的一种。选举算法的核心在 RaftCore 中，包括数据的处理和数据同步。

　　raft 算法演示地址 ：http://thesecretlivesofdata.com/raft/

　　在Raft中，节点有三种角色：

1. Leader：负责接收客户端的请求
2. Candidate：用于选举Leader的一种角色(竞选状态)
3. Follower：负责响应来自Leader或者Candidate的请求

　　选举分为两个节点

1. 服务启动的时候
2. leader挂了的时候

　　所有节点启动的时候，都是follower状态。 如果在一段时间内如果没有收到leader的心跳（可能是没有leader，也可能是leader挂了），那么follower会变成Candidate。然后发起选举，选举之前，会增加 term，这个 term 和 zookeeper 中的 epoch 的道理是一样的。

　　follower会投自己一票，并且给其他节点发送票据vote，等到其他节点回复在这个过程中，可能出现几种情况

1. 收到过半的票数通过，则成为leader
2. 被告知其他节点已经成为leader，则自己切换为follower
3. 一段时间内没有收到过半的投票，则重新发起选举

　　**约束条件在任一term中，单个节点最多只能投一票**

**选举的几种情况 :**

1. 第一种情况，赢得选举之后，leader会给所有节点发送消息，避免其他节点触发新的选举
2. 第二种情况，比如有三个节点A B C。A B同时发起选举，而A的选举消息先到达C，C给A投了一票，当B的消息到达C时，已经不能满足上面提到的**约束条件**，即C不会给B投票，而A和B显然都不会给对方投票。A胜出之后，会给B,C发心跳消息，节点B发现节点A的term不低于自己的term，知道有已经有Leader了，于是转换成follower
3. 第三种情况， 没有任何节点获得majority投票，可能是平票的情况。加入总共有四个节点（A/B/C/D），Node C、Node D同时成为了candidate，但Node A投了NodeD一票，NodeB投了Node C一票，这就出现了平票 split vote的情况。这个时候大家都在等啊等，直到超时后重新发起选举。如果出现平票的情况，那么就延长了系统不可用的时间,因此raft引入了 randomizedelection timeouts来尽量避免平票情况.



### 源码分析 ：

　　**RaftCore 初始化 ：Raft选举算法，是在RaftCore这个类中实现的。**

处理逻辑非常简单。

- 判断收到的请求的term是不是过期的数据，如果是，则认为对方的这个票据无效，直接告诉发送这个票据的节点，你应该选择当前收到请求的节点。
- 否则，当前收到请求的节点会自动接受对方的票据，并把自己设置成follower
- peers.decideLeader(peer) 表示用来决策谁能成为leader

### 数据同步：

　　在注册服务时，调用addInstance之后，最后会调用 consistencyService.put(key,instances); 这个方法，来实现数据一致性的同步。

　调用 consistencyService.put 用来发布内容，也就是实现数据的一致性同步。

其中 onPublish(datum, peers.local()); 发送数据到所有节点：











