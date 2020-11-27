[TOC]

## Dubbo

### 基础

* Dubbo是一个RPC框架，用于服务通信。而SpringCloud是一个微服务生态

![image-20200714110437587](../../../../Software/Typora/Picture/image-20200714110437587.png)

* 服务器端是提供者Provider，客户端是消费者Consumer

#### 使用

![image-20200714145605515](../../../../Software/Typora/Picture/image-20200714145605515.png)

* 启动zookeeper：运行zookeeper的bin目录下的zkServer.cmd和zkCli.cmd

* 安装图形化服务管理页面dubbo-admin：编译运行 dubbo-admin-0.0.1-SNAPSHOT.jar

* 进入图形化界面：localhost:7007 进入dubbo客户端，账号密码都是root

* 1.将服务提供者的接口和bean都放在一个单独的API工程gmail-interface的包里，供其他工程导入该API工程依赖从而进行远程调用

* 2.将服务提供者注册到注册中心（暴露服务）

* 服务提供方设置的的超时时间高于消费方设置的超时事件时会调用失败，抛异常

  * 1）导入dubbo依赖

  ```
  		<dependency>
  			<groupId>com.alibaba</groupId>
  			<artifactId>dubbo</artifactId>
  			<version>2.6.2</version>
  		</dependency>
  		<!-- 注册中心使用的是zookeeper，引入操作zookeeper的客户端 -->
  		<dependency>
  			<groupId>org.apache.curator</groupId>
  			<artifactId>curator-framework</artifactId>
  			<version>2.12.0</version>
  		</dependency>
  ```

  * 2）服务提供者配置provider.xml

  ```
  <!-- 1、指定当前服务/应用的名字（同样的服务名字相同，不要和别的服务同名） -->
  	<dubbo:application name="user-service-provider"></dubbo:application>
  	<!-- 2、指定注册中心的位置 -->
  	<!-- <dubbo:registry address="zookeeper://127.0.0.1:2181"></dubbo:registry> -->
  	<dubbo:registry protocol="zookeeper" address="127.0.0.1:2181"></dubbo:registry>
  	<!-- 3、指定通信规则（通信协议？通信端口） -->
  	<dubbo:protocol name="dubbo" port="20882"></dubbo:protocol>
  	<!-- 4、暴露服务   ref：指向服务的真正的实现对象 -->
  	<dubbo:service interface="com.atguigu.gmall.service.UserService" 
  		ref="userServiceImpl01" timeout="1000" version="1.0.0">
  		<dubbo:method name="getUserAddressList" timeout="1000"></dubbo:method>
  	</dubbo:service>
  	<!--统一设置服务提供方的规则  -->
  	<dubbo:provider timeout="1000"></dubbo:provider>
  	<!-- 服务的实现 -->
  	<bean id="userServiceImpl01" class="com.atguigu.gmall.service.impl.UserServiceImpl"></bean>
  	<dubbo:service interface="com.atguigu.gmall.service.UserService" 
  		ref="userServiceImpl02" timeout="1000" version="2.0.0">
  		<dubbo:method name="getUserAddressList" timeout="1000"></dubbo:method>
  	</dubbo:service>
  	<bean id="userServiceImpl02" class="com.atguigu.gmall.service.impl.UserServiceImpl2"></bean>
  	<!-- 连接监控中心 -->
  	<dubbo:monitor protocol="registry"></dubbo:monitor>
  ```

  

* 3.让消费者去注册中心订阅服务提供者的服务地址

```
	<context:component-scan base-package="com.atguigu.gmall.service.impl"></context:component-scan>
	<dubbo:application name="order-service-consumer"></dubbo:application>
	<dubbo:registry address="zookeeper://127.0.0.1:2181"></dubbo:registry>
	<!--  配置本地存根-->
	<!--声明需要调用的远程服务的接口；生成远程服务代理  -->
	<!-- 
		1）、精确优先 (方法级优先，接口级次之，全局配置再次之)
		2）、消费者设置优先(如果级别一样，则消费方优先，提供方次之)
	-->
	<!-- timeout="0" 默认是1000ms-->
	<!-- retries="":重试次数，不包含第一次调用，0代表不重试-->
	<!-- 幂等（设置重试次数）【查询、删除、修改】、非幂等（不能设置重试次数）【新增】 -->
	<dubbo:reference interface="com.atguigu.gmall.service.UserService" 
		id="userService" timeout="5000" retries="3" version="*">
		<!-- <dubbo:method name="getUserAddressList" timeout="1000"></dubbo:method> -->
	</dubbo:reference>
	<!-- 配置当前消费者的统一规则：所有的服务都不检查 -->
	<dubbo:consumer check="false" timeout="5000"></dubbo:consumer>
	<dubbo:monitor protocol="registry"></dubbo:monitor>
	<!-- <dubbo:monitor address="127.0.0.1:7070"></dubbo:monitor> -->
```

```
@Service
public class OrderServiceImpl implements OrderService {
	@Autowired
	UserService userService;
	@Override
	public List<UserAddress> initOrder(String userId) {
		// TODO Auto-generated method stub
		System.out.println("用户id："+userId);
		//1、查询用户的收货地址
		List<UserAddress> addressList = userService.getUserAddressList(userId);
		for (UserAddress userAddress : addressList) {
			System.out.println(userAddress.getUserAddress());
		}
		return addressList;
	}
}
```

* 安装监控中心 dubbo-monitor-simple
  * http://localhost:8080/  ： 进入监控中心管理界面
  * 服务消费者

```
	<dubbo:monitor protocol="registry"></dubbo:monitor>    从注册中心发现并连上监控中心
	<!-- <dubbo:monitor address="127.0.0.1:7070"></dubbo:monitor> -->   直连监控中心
```

​		    服务提供者

```
	<!-- 从注册中心连接监控中心 -->
	<dubbo:monitor protocol="registry"></dubbo:monitor>
```



#### Dubbo整合SpringBoot

* 服务提供者

  * 1.导入依赖

  ```
  <dependency>
      <groupId>com.alibaba.boot</groupId>
      <artifactId>dubbo-spring-boot-starter</artifactId>
      <version>0.2.0</version>
  </dependency>
  ```

  * 2.配置文件

  ```
  dubbo.application.name=user-service-provider
  dubbo.registry.address=127.0.0.1:2181
  dubbo.registry.protocol=zookeeper
  dubbo.protocol.name=dubbo
  dubbo.protocol.port=20881
  dubbo.monitor.protocol=registry
  dubbo.scan.base-packages=com.atguigu.gmall 
  ```

  * 3.暴露服务：在需要被RPC的ServiceImpl上标上Dubbo的注解@Service，以及在启动类上注解：@EnableDubbo，开启基于注解的Dubbo功能
  * 若是基于xml配置，则需要在service类加上注释：@ImportResource(localtion="classpath:provider.xml")

* 服务消费者

  * 导入依赖

  ```
  <dependency>
      <groupId>com.alibaba.boot</groupId>
      <artifactId>dubbo-spring-boot-starter</artifactId>
  	<version>0.2.0</version>
  </dependency>
  ```

  * 2.配置文件

  ```
  # 修改tomcat端口号
  server.port=8081
  
  dubbo.application.name=boot-order-service-consumer
  dubbo.registry.address=zookeeper://127.0.0.1:2181
  dubbo.monitor.protocol=registry
  ```

  * 3.消费服务：将远程调用的Service上的@Autowire改为：@Reference 。以及在启动类上注解：@EnableDubbo

  

#### dubbo配置细节

* Dubbo配置加载顺序优先级：

  ![image-20200714163547817](../../../../Software/Typora/Picture/image-20200714163547817.png)

* 启动检查：消费者启动之前会先检查是否有可用的服务提供者，没有则会报错，可以在消费者xml文件修改配置，关闭启动检查：

  ```
  <dubbo:reference interface="com.atguigu.gmall.service.UserService" id="userService" check="false">
  ```

  还可以统一关闭启动检查

  ```
  <!-- 配置当前消费者的统一规则：所有的服务都不检查 -->
  <dubbo:consumer check="false" timeout="5000"></dubbo:consumer>
  ```

* 超时&配置覆盖关系

  * 远程调用服务时超过5秒则报错，防止阻塞

  ```
  <dubbo:reference interface="com.atguigu.gmall.service.UserService" id="userService" timeout="5000" retries="3" version="*">   
  ```

  * 也可以设置某个方法的超时属性，精确优先级：方法>接口>全局配置。消费方> 提供方。如果级别一样，则消费方优先

  ```
  <dubbo:method name="getUserAddressList" timeout="1000"></dubbo:method>
  ```

* 重试次数：（第一次调用不包括在重试次数里，超过次数就报错）

  * 全局配置：（幂等（设置重试次数）【查询、删除、修改】、非幂等（不能设置重试次数）【新增】 ）

  ```
  <dubbo:reference interface="com.atguigu.gmall.service.UserService" id="userService"  retries="3">
  ```

* 配置多版本（类似灰度测试）

  * 在消费者配置 version设置版本号，会执行服务提供方对应版本号的服务，可用于灰度测试，新版本测试稳定再上线。设置 * 表示随机一个版本。

  ```
  <dubbo:reference interface="com.atguigu.gmall.service.UserService" id="userService" version="1.0.0">
  ```

* 本地存根：用于正式远程调用前的测试

* 与SpringBoot整合的三种方式

  * 1）、导入dubbo-starter，在application.properties配置属性，使用@Service【暴露服务】使用@Reference【引用服务】

   * 2）、保留dubbo xml配置文件：导入dubbo-starter，使用@ImportResource导入dubbo的配置文件即可

   * 3）、使用注解API的方式：将每一个组件手动创建到容器中,让dubbo来扫描其他的组件

      * ```
        @Configuration
        public class MyDubboConfig {
        	@Bean
        	public ApplicationConfig applicationConfig() {
        		ApplicationConfig applicationConfig = new ApplicationConfig();
        		applicationConfig.setName("boot-user-service-provider");
        		return applicationConfig;
        	}
        	//<dubbo:registry protocol="zookeeper" address="127.0.0.1:2181"></dubbo:registry>
        	@Bean
        	public RegistryConfig registryConfig() {
        		RegistryConfig registryConfig = new RegistryConfig();
        		registryConfig.setProtocol("zookeeper");
        		registryConfig.setAddress("127.0.0.1:2181");
        		return registryConfig;
        	}
        	//<dubbo:protocol name="dubbo" port="20882"></dubbo:protocol>
        	@Bean
        	public ProtocolConfig protocolConfig() {
        		ProtocolConfig protocolConfig = new ProtocolConfig();
        		protocolConfig.setName("dubbo");
        		protocolConfig.setPort(20882);
        		return protocolConfig;
        	}
        	/**
        	 *<dubbo:service interface="com.atguigu.gmall.service.UserService" 
        		ref="userServiceImpl01" timeout="1000" version="1.0.0">
        		<dubbo:method name="getUserAddressList" timeout="1000"></dubbo:method>
        	</dubbo:service>
        	 */
        	@Bean
        	public ServiceConfig<UserService> userServiceConfig(UserService userService){
        		ServiceConfig<UserService> serviceConfig = new ServiceConfig<>();
        		serviceConfig.setInterface(UserService.class);
        		serviceConfig.setRef(userService);
        		serviceConfig.setVersion("1.0.0");
        		//配置每一个method的信息
        		MethodConfig methodConfig = new MethodConfig();
        		methodConfig.setName("getUserAddressList");
        		methodConfig.setTimeout(1000);
        		//将method的设置关联到service配置中
        		List<MethodConfig> methods = new ArrayList<>();
        		methods.add(methodConfig);
        		serviceConfig.setMethods(methods);
        		//ProviderConfig
        		//MonitorConfig
        		return serviceConfig;
        	}
        }
        ```

        

#### 高可用

* 通过设计，减少系统不能提供服务的事件

* 当zookeeper注册中心宕机，可以直连dubbo，消费暴露的服务。做法是在服务消费方的serviceImpl下的引用远程调用注入加注解

  * ```
    	@Reference(url="127.0.0.1:20882") //dubbo直连
      	UserService userService;
    ```

* 四大负载均衡机制（默认随机轮询）

  * Random LoadBalance：基于权重的随机负载均衡机制（dubbo默认机制），可以在dubbo图形化页面动态调节权重

  ```
  @Reference(loadbalance="random") //dubbo直连（需要在服务消费方设置）
  UserService userService;
  ```

  * RoundRobin LoadBalance：基于权重的轮询加载均衡机制

  ```
  @Reference(loadbalance="roundrobin") //dubbo直连（需要在服务消费方设置）
  UserService userService;
  ```

  * LastActive LoadBalance：最少活跃数-负载均衡机制：每次使用在上一次响应最快的服务器

  ```
  @Reference(loadbalance="lastactive ") //dubbo直连（需要在服务消费方设置）
  UserService userService;
  ```

  * ConsistenHash LoadBalance：一致性hash-负载均衡机制：根据参数的hash值进行选择调度

  ```
  @Reference(loadbalance="consistenhash ") //dubbo直连（需要在服务消费方设置）
  UserService userService;
  ```

#### 服务降级

* 服务器压力剧增情况下，对一些服务和页面有策略地不处理或换种简单的方式处理，从而释放服务器资源以保证核心交易正常运作或高效运作
* 可以通过服务降级功能临时屏蔽某个出错的非关键服务，并定义降级后的返回策略
* 向注册中心写入动态配置覆盖规则：（可以在dubbo图形化页面设置）
  * mock=force:return+null (屏蔽)：消费方对该服务的调用都返回null，不发起远程调用。用来屏蔽不重要服务不可用时对调用方的影响
  * mock=fail:return+null(容错) ：消费方对该服务的方法调用失败后，再返回null，不抛异常，用来容忍不重要服务不稳定时对调用方的影响



#### 集群容错&Hystrix

* Failfast Cluster：快速失败，只发起一次调用，失败立即报错，通常用于非幂等性（新增 ）的写操作
* Failsave Cluster：失败安全，出现异常直接忽略，通常用于写入审计日志等操作
* Failback Cluster：失败自动恢复，后台记录失败请求，定时重发，通常用于消息通知操作
* Forking Cluster：并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，通过forks=""设置最大并行数
* Broadcast Cluster：广播调用所有提供者，逐个调用，任意一台报错则报错，通常用于通知所有提供者更新缓存或日志等本地资源信息
* 集群模式配置：`<dubbo:service cluster="failsave" />` 或 `<dubbo;reference cluster="failsave" />`

#### 整合Hystrix熔断

```
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
		</dependency>
```

* 服务提供者的serviceImpl加上注解@HystrixCommand()即可开启容错

* 消费方的启动类需要加上注解：@EnableHystrix。
* 消费方远程调用时，在类上加上注解@HystrixCommand(fallbackMethod="hello")，表示远程调用失败就执行hello方法

#### RPC原理

![image-20200714221133403](../../../../Software/Typora/Picture/image-20200714221133403.png)

一次完整的RPC调用流程（同步调用，异步另说）如下：

**1**）服务消费方（client）调用以本地调用方式调用服务；

2）client stub接收到调用后负责将方法、参数等组装成能够进行网络传输的消息体； 

3）client stub找到服务地址，并将消息发送到服务端； 

4）server stub收到消息后进行解码； 

5）server stub根据解码结果调用本地的服务； 

6）本地服务执行并将结果返回给server stub； 

7）server stub将返回结果打包成消息并发送至消费方； 

8）client stub接收到消息，并进行解码； 

9）服务消费方得到最终结果。

RPC框架的目标就是要2~8这些步骤都封装起来，这些细节对用户来说是透明的，不可见的。而Dubbo只需要做第1步和第9步即可

#### Netty通信原理

* Netty是一个异步事件驱动的网络应用程序框架， 用于快速开发可维护的高性能协议服务器和客户端。它极大地简化并简化了TCP和UDP套接字服务器等网络编程。

* BIO：(Blocking IO)

![image-20200714221606460](../../../../Software/Typora/Picture/image-20200714221606460.png)

* NIO (Non-Blocking IO)

  * Selector 一般称 为**选择器** ，也可以翻译为 **多路复用器，**

    Connect（连接就绪）、Accept（接受就绪）、Read（读就绪）、Write（写就绪）

![image-20200714221659269](../../../../Software/Typora/Picture/image-20200714221659269.png)



## Dubbo原理

* Netty就是基于NIO的多路复用模型来做的：

![image-20200714222004822](../../../../Software/Typora/Picture/image-20200714222004822.png)



* 框架设计

![image-20200714222453650](../../../../Software/Typora/Picture/image-20200714222453650.png)

#### 标签解析

* 配置文件中标签解析
  * Spring解析xml配置文件中的所有标签，这些标签有一个总接口BeanDefinitionParser（定义解析器）
  * 这个总接口其中有一个子接口DubboBeanDefinitionParser，该接口里就是是所有关于dubbo标签的实现
  * DubboBeanDefinitionParser的parse方法就是具体解析标签的实现，下面只摘抄一部分源码展示

```
public class DubboBeanDefinitionParser implements BeanDefinitionParser {
    public BeanDefinition parse(Element element, ParserContext parserContext) {
    	return parse(element, parserContext, beanClass, required);
    }

	public DubboBeanDefinitionParser(Class<?> beanClass, boolean required){
		this.beanClass = beanClass;
		this.required = required;
	}

	// 解析所有标签里的属性
	// beanClass参数都需要先注册标签，执行DubboNamespaceHandler的init方法
    @SuppressWarnings("unchecked")
    private static BeanDefinition parse(Element element,ParserContext parserContext,Class<?> beanClass,boolean required) {
        RootBeanDefinition beanDefinition = new RootBeanDefinition();
        beanDefinition.setBeanClass(beanClass);
        beanDefinition.setLazyInit(false);
        //......省略
        if (ProtocolConfig.class.equals(beanClass)) {
            for (String name : parserContext.getRegistry().getBeanDefinitionNames()) {
                BeanDefinition definition = parserContext.getRegistry().getBeanDefinition(name);
                PropertyValue property = definition.getPropertyValues().getPropertyValue("protocol");
                if (property != null) {
                    Object value = property.getValue();
                    if (value instanceof ProtocolConfig && id.equals(((ProtocolConfig) value).getName())) {
                        definition.getPropertyValues().addPropertyValue("protocol", new RuntimeBeanReference(id));
                    }
                }
            }
        } else if (ServiceBean.class.equals(beanClass)) {
            String className = element.getAttribute("class");
            if(className != null && className.length() > 0) {
                RootBeanDefinition classDefinition = new RootBeanDefinition();
                classDefinition.setBeanClass(ReflectUtils.forName(className));
                classDefinition.setLazyInit(false);
                parseProperties(element.getChildNodes(), classDefinition);
                beanDefinition.getPropertyValues().addPropertyValue("ref", new BeanDefinitionHolder(classDefinition, id + "Impl"));
            }
        } else if (ProviderConfig.class.equals(beanClass)) {
            parseNested(element, parserContext, ServiceBean.class, true, "service", "provider", id, beanDefinition);
        } else if (ConsumerConfig.class.equals(beanClass)) {
            parseNested(element, parserContext, ReferenceBean.class, false, "reference", "consumer", id, beanDefinition);
        }
        //......省略
        return beanDefinition;
    }
}
```

* 在执行DubboBeanDefinitionParser构造器之前，执行了Dubbo名称空间处理器DubboNamespaceHandler的init方法，注册所有标签解析器

  其中在DubboNamespaceHandler的init方法里，ServiceBean.class比较特殊

```
public class DubboNamespaceHandler extends NamespaceHandlerSupport {
 
	static {
		Version.checkDuplicate(DubboNamespaceHandler.class);
	}
 	// 解析每个标签并保存到对应的class
	public void init() {
		//配置<dubbo:application>标签解析器，解析并保存到ApplicationConfig.class
	    registerBeanDefinitionParser("application", new DubboBeanDefinitionParser(ApplicationConfig.class, true));
        //配置<dubbo:module>标签解析器
	    registerBeanDefinitionParser("module", new DubboBeanDefinitionParser(ModuleConfig.class, true));
        //配置<dubbo:registry>标签解析器
	    registerBeanDefinitionParser("registry", new DubboBeanDefinitionParser(RegistryConfig.class, true));
        //配置<dubbo:monitor>标签解析器
	    registerBeanDefinitionParser("monitor", new DubboBeanDefinitionParser(MonitorConfig.class, true));
        //配置<dubbo:provider>标签解析器
	    registerBeanDefinitionParser("provider", new DubboBeanDefinitionParser(ProviderConfig.class, true));
        //配置<dubbo:consumer>标签解析器
	    registerBeanDefinitionParser("consumer", new DubboBeanDefinitionParser(ConsumerConfig.class, true));
        //配置<dubbo:protocol>标签解析器
	    registerBeanDefinitionParser("protocol", new DubboBeanDefinitionParser(ProtocolConfig.class, true));
	    //配置<dubbo:service>标签解析器
	    registerBeanDefinitionParser("service", new DubboBeanDefinitionParser(ServiceBean.class, true));// 断点1：ServiceBean
	    //配置<dubbo:refenrence>标签解析器
	    registerBeanDefinitionParser("reference", new DubboBeanDefinitionParser(ReferenceBean.class, false));
	    //配置<dubbo:annotation>标签解析器
	    registerBeanDefinitionParser("annotation", new DubboBeanDefinitionParser(AnnotationBean.class, true));
    }
 }
```

#### 服务暴露过程，即注册到服务中心

![image-20200715124702296](../../../../Software/Typora/Picture/image-20200715124702296.png)

* 进入断点1：ServiceBean.class

```
// InitializingBean:Spring里的接口，当组件完成创建对象后，会回调该接口的afterPropertiesSet()方法
// ApplicationListener：Spring里的接口，监听ContextRefreshedEvent事件，即当ioc容器所有对象创建完成后，会回调该方法中的onApplicationEvent方法
// 
public class ServiceBean<T> extends ServiceConfig<T> implements InitializingBean, DisposableBean, ApplicationContextAware, ApplicationListener<ContextRefreshedEvent>, BeanNameAware {
	。。。
    
	@Override
    @SuppressWarnings({"unchecked", "deprecation"})
    public void afterPropertiesSet() throws Exception {
        if (getProvider() == null) { // getProvider()获得每个解析标签后的值
            Map<String, ProviderConfig> providerConfigMap = applicationContext == null ? null : BeanFactoryUtils.beansOfTypeIncludingAncestors(applicationContext, ProviderConfig.class, false, false);
            if (providerConfigMap != null && providerConfigMap.size() > 0) {
                Map<String, ProtocolConfig> protocolConfigMap = applicationContext == null ? null : BeanFactoryUtils.beansOfTypeIncludingAncestors(applicationContext, ProtocolConfig.class, false, false);
                if ((protocolConfigMap == null || protocolConfigMap.size() == 0)
                        && providerConfigMap.size() > 1) { // backward compatibility
                    List<ProviderConfig> providerConfigs = new ArrayList<ProviderConfig>();
                    for (ProviderConfig config : providerConfigMap.values()) {
                        if (config.isDefault() != null && config.isDefault().booleanValue()) {
                            providerConfigs.add(config);
                        }
                    }
                    if (!providerConfigs.isEmpty()) {
                        setProviders(providerConfigs);  // 保存解析标签后的属性配置信息
                    }
                } else {
                    ProviderConfig providerConfig = null;
                    for (ProviderConfig config : providerConfigMap.values()) {
                        if (config.isDefault() == null || config.isDefault().booleanValue()) {
                            if (providerConfig != null) {
                                throw new IllegalStateException("Duplicate provider configs: " + providerConfig + " and " + config);
                            }
                            providerConfig = config;
                        }
                    }
                    if (providerConfig != null) {
                        setProvider(providerConfig); // 保存解析标签后的属性配置信息
                    }
                }
            }
        }
        if (getApplication() == null
                && (getProvider() == null || getProvider().getApplication() == null)) {
            Map<String, ApplicationConfig> applicationConfigMap = applicationContext == null ? null : BeanFactoryUtils.beansOfTypeIncludingAncestors(applicationContext, ApplicationConfig.class, false, false);
            if (applicationConfigMap != null && applicationConfigMap.size() > 0) {
                ApplicationConfig applicationConfig = null;
                for (ApplicationConfig config : applicationConfigMap.values()) {
                    if (config.isDefault() == null || config.isDefault().booleanValue()) {
                        if (applicationConfig != null) {
                            throw new IllegalStateException("Duplicate application configs: " + applicationConfig + " and " + config);
                        }
                        applicationConfig = config;
                    }
                }
                if (applicationConfig != null) {
                    setApplication(applicationConfig); // 保存Application应用信息
                }
            }
        }
        if (getModule() == null
                && (getProvider() == null || getProvider().getModule() == null)) {
            Map<String, ModuleConfig> moduleConfigMap = applicationContext == null ? null : BeanFactoryUtils.beansOfTypeIncludingAncestors(applicationContext, ModuleConfig.class, false, false);
            if (moduleConfigMap != null && moduleConfigMap.size() > 0) {
                ModuleConfig moduleConfig = null;
                for (ModuleConfig config : moduleConfigMap.values()) {
                    if (config.isDefault() == null || config.isDefault().booleanValue()) {
                        if (moduleConfig != null) {
                            throw new IllegalStateException("Duplicate module configs: " + moduleConfig + " and " + config);
                        }
                        moduleConfig = config;
                    }
                }
                if (moduleConfig != null) {
                    setModule(moduleConfig); // 保存Module信息
                }
            }
        }
        if ((getRegistries() == null || getRegistries().isEmpty())
                && (getProvider() == null || getProvider().getRegistries() == null || getProvider().getRegistries().isEmpty())
                && (getApplication() == null || getApplication().getRegistries() == null || getApplication().getRegistries().isEmpty())) {
            Map<String, RegistryConfig> registryConfigMap = applicationContext == null ? null : BeanFactoryUtils.beansOfTypeIncludingAncestors(applicationContext, RegistryConfig.class, false, false);
            if (registryConfigMap != null && registryConfigMap.size() > 0) {
                List<RegistryConfig> registryConfigs = new ArrayList<RegistryConfig>();
                for (RegistryConfig config : registryConfigMap.values()) {
                    if (config.isDefault() == null || config.isDefault().booleanValue()) {
                        registryConfigs.add(config);
                    }
                }
                if (registryConfigs != null && !registryConfigs.isEmpty()) {
                    super.setRegistries(registryConfigs); // 保存注册中心信息
                }
            }
        }
        if (getMonitor() == null
                && (getProvider() == null || getProvider().getMonitor() == null)
                && (getApplication() == null || getApplication().getMonitor() == null)) {
            Map<String, MonitorConfig> monitorConfigMap = applicationContext == null ? null : BeanFactoryUtils.beansOfTypeIncludingAncestors(applicationContext, MonitorConfig.class, false, false);
            if (monitorConfigMap != null && monitorConfigMap.size() > 0) {
                MonitorConfig monitorConfig = null;
                for (MonitorConfig config : monitorConfigMap.values()) {
                    if (config.isDefault() == null || config.isDefault().booleanValue()) {
                        if (monitorConfig != null) {
                            throw new IllegalStateException("Duplicate monitor configs: " + monitorConfig + " and " + config);
                        }
                        monitorConfig = config;
                    }
                }
                if (monitorConfig != null) {
                    setMonitor(monitorConfig); // 保存监控中心信息
                }
            }
        }
        if ((getProtocols() == null || getProtocols().isEmpty())
                && (getProvider() == null || getProvider().getProtocols() == null || getProvider().getProtocols().isEmpty())) {
            Map<String, ProtocolConfig> protocolConfigMap = applicationContext == null ? null : BeanFactoryUtils.beansOfTypeIncludingAncestors(applicationContext, ProtocolConfig.class, false, false);
            if (protocolConfigMap != null && protocolConfigMap.size() > 0) {
                List<ProtocolConfig> protocolConfigs = new ArrayList<ProtocolConfig>();
                for (ProtocolConfig config : protocolConfigMap.values()) {
                    if (config.isDefault() == null || config.isDefault().booleanValue()) {
                        protocolConfigs.add(config);
                    }
                }
                if (protocolConfigs != null && !protocolConfigs.isEmpty()) {
                    super.setProtocols(protocolConfigs); // 保存协议配置信息
                }
            }
        }
        if (getPath() == null || getPath().length() == 0) {
            if (beanName != null && beanName.length() > 0
                    && getInterface() != null && getInterface().length() > 0
                    && beanName.startsWith(getInterface())) {
                setPath(beanName);
            }
        }
        if (!isDelay()) {
            export();
        }
    }
}

	// 容器创建完成后，触发事件回调，执行服务暴露
	@Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        if (isDelay() && !isExported() && !isUnexported()) {
            if (logger.isInfoEnabled()) {
                logger.info("The service ready on spring started. service: " + getInterface());
            }
            export(); // 断点2：执行暴露服务功能
        }
    }
```

* 断点2：进入到ServiceConfig的export方法，执行暴露服务功能

```
    public synchronized void export() {
        if (provider != null) {
            if (export == null) {
                export = provider.getExport();
            }
            if (delay == null) {
                delay = provider.getDelay();
            }
        }
        if (export != null && !export) {
            return;
        }

        if (delay != null && delay > 0) {
            delayExportExecutor.schedule(new Runnable() {
                @Override
                public void run() {
                    doExport(); // 断点3：执行暴露服务
                }
            }, delay, TimeUnit.MILLISECONDS);
        } else {
            doExport();
        }
    }
```

* 断点3：进入doExport方法。执行暴露服务

```
protected synchronized void doExport() {
        if (unexported) {
            throw new IllegalStateException("Already unexported!");
        }
        if (exported) {
            return;
        }
        exported = true;
        if (interfaceName == null || interfaceName.length() == 0) {
            throw new IllegalStateException("<dubbo:service interface=\"\" /> interface not allow null!");
        }
        checkDefault();
        if (provider != null) {
            if (application == null) {
                application = provider.getApplication();
            }
            if (module == null) {
                module = provider.getModule();
            }
            if (registries == null) {
                registries = provider.getRegistries();
            }
            if (monitor == null) {
                monitor = provider.getMonitor();
            }
            if (protocols == null) {
                protocols = provider.getProtocols();
            }
        }
        if (module != null) {
            if (registries == null) {
                registries = module.getRegistries();
            }
            if (monitor == null) {
                monitor = module.getMonitor();
            }
        }
        if (application != null) {
            if (registries == null) {
                registries = application.getRegistries();
            }
            if (monitor == null) {
                monitor = application.getMonitor();
            }
        }
        if (ref instanceof GenericService) {
            interfaceClass = GenericService.class;
            if (StringUtils.isEmpty(generic)) {
                generic = Boolean.TRUE.toString();
            }
        } else {
            try {
                interfaceClass = Class.forName(interfaceName, true, Thread.currentThread()
                        .getContextClassLoader());
            } catch (ClassNotFoundException e) {
                throw new IllegalStateException(e.getMessage(), e);
            }
            checkInterfaceAndMethods(interfaceClass, methods);
            checkRef();
            generic = Boolean.FALSE.toString();
        }
        if (local != null) {
            if ("true".equals(local)) {
                local = interfaceName + "Local";
            }
            Class<?> localClass;
            try {
                localClass = ClassHelper.forNameWithThreadContextClassLoader(local);
            } catch (ClassNotFoundException e) {
                throw new IllegalStateException(e.getMessage(), e);
            }
            if (!interfaceClass.isAssignableFrom(localClass)) {
                throw new IllegalStateException("The local implementation class " + localClass.getName() + " not implement interface " + interfaceName);
            }
        }
        if (stub != null) {
            if ("true".equals(stub)) {
                stub = interfaceName + "Stub";
            }
            Class<?> stubClass;
            try {
                stubClass = ClassHelper.forNameWithThreadContextClassLoader(stub);
            } catch (ClassNotFoundException e) {
                throw new IllegalStateException(e.getMessage(), e);
            }
            if (!interfaceClass.isAssignableFrom(stubClass)) {
                throw new IllegalStateException("The stub implementation class " + stubClass.getName() + " not implement interface " + interfaceName);
            }
        }
        checkApplication();
        checkRegistry();
        checkProtocol();
        appendProperties(this);
        checkStubAndMock(interfaceClass);
        if (path == null || path.length() == 0) {
            path = interfaceName;
        }
        doExportUrls();  // 断点4：执行暴露URL地址
        ProviderModel providerModel = new ProviderModel(getUniqueServiceName(), this, ref);
        ApplicationModel.initProviderModel(getUniqueServiceName(), providerModel);
    }
```

* 断点4：执行暴露URL地址

```
	@SuppressWarnings({"unchecked", "rawtypes"})
    private void doExportUrls() {
        List<URL> registryURLs = loadRegistries(true); // 加载并读取注册中心的信息
        for (ProtocolConfig protocolConfig : protocols) {  // 遍历protocal协议标签，即除了dubbo外还可以有其他协议
            doExportUrlsFor1Protocol(protocolConfig, registryURLs); // 断点5：暴露dubbo协议的服务
        }
    }
```

* 断点5：暴露dubbo协议的服务

```
	private void doExportUrlsFor1Protocol(ProtocolConfig protocolConfig, List<URL> registryURLs) {
        String name = protocolConfig.getName();
        if (name == null || name.length() == 0) {
            name = "dubbo";
        }

        Map<String, String> map = new HashMap<String, String>();
        map.put(Constants.SIDE_KEY, Constants.PROVIDER_SIDE);
        map.put(Constants.DUBBO_VERSION_KEY, Version.getVersion());
        map.put(Constants.TIMESTAMP_KEY, String.valueOf(System.currentTimeMillis()));
        if (ConfigUtils.getPid() > 0) {
            map.put(Constants.PID_KEY, String.valueOf(ConfigUtils.getPid()));
        }
        appendParameters(map, application);
        appendParameters(map, module);
        appendParameters(map, provider, Constants.DEFAULT_KEY);
        appendParameters(map, protocolConfig);
        appendParameters(map, this);
        if (methods != null && !methods.isEmpty()) {
            for (MethodConfig method : methods) {
                appendParameters(map, method, method.getName());
                String retryKey = method.getName() + ".retry";
                if (map.containsKey(retryKey)) {
                    String retryValue = map.remove(retryKey);
                    if ("false".equals(retryValue)) {
                        map.put(method.getName() + ".retries", "0");
                    }
                }
                List<ArgumentConfig> arguments = method.getArguments();
                if (arguments != null && !arguments.isEmpty()) {
                    for (ArgumentConfig argument : arguments) {
                        // convert argument type
                        if (argument.getType() != null && argument.getType().length() > 0) {
                            Method[] methods = interfaceClass.getMethods();
                            // visit all methods
                            if (methods != null && methods.length > 0) {
                                for (int i = 0; i < methods.length; i++) {
                                    String methodName = methods[i].getName();
                                    // target the method, and get its signature
                                    if (methodName.equals(method.getName())) {
                                        Class<?>[] argtypes = methods[i].getParameterTypes();
                                        // one callback in the method
                                        if (argument.getIndex() != -1) {
                                            if (argtypes[argument.getIndex()].getName().equals(argument.getType())) {
                                                appendParameters(map, argument, method.getName() + "." + argument.getIndex());
                                            } else {
                                                throw new IllegalArgumentException("argument config error : the index attribute and type attribute not match :index :" + argument.getIndex() + ", type:" + argument.getType());
                                            }
                                        } else {
                                            // multiple callbacks in the method
                                            for (int j = 0; j < argtypes.length; j++) {
                                                Class<?> argclazz = argtypes[j];
                                                if (argclazz.getName().equals(argument.getType())) {
                                                    appendParameters(map, argument, method.getName() + "." + j);
                                                    if (argument.getIndex() != -1 && argument.getIndex() != j) {
                                                        throw new IllegalArgumentException("argument config error : the index attribute and type attribute not match :index :" + argument.getIndex() + ", type:" + argument.getType());
                                                    }
                                                }
                                            }
                                        }
                                    }
                                }
                            }
                        } else if (argument.getIndex() != -1) {
                            appendParameters(map, argument, method.getName() + "." + argument.getIndex());
                        } else {
                            throw new IllegalArgumentException("argument config must set index or type attribute.eg: <dubbo:argument index='0' .../> or <dubbo:argument type=xxx .../>");
                        }

                    }
                }
            } // end of methods for
        }

        if (ProtocolUtils.isGeneric(generic)) {
            map.put(Constants.GENERIC_KEY, generic);
            map.put(Constants.METHODS_KEY, Constants.ANY_VALUE);
        } else {
            String revision = Version.getVersion(interfaceClass, version);
            if (revision != null && revision.length() > 0) {
                map.put("revision", revision);
            }

            String[] methods = Wrapper.getWrapper(interfaceClass).getMethodNames();
            if (methods.length == 0) {
                logger.warn("NO method found in service interface " + interfaceClass.getName());
                map.put(Constants.METHODS_KEY, Constants.ANY_VALUE);
            } else {
                map.put(Constants.METHODS_KEY, StringUtils.join(new HashSet<String>(Arrays.asList(methods)), ","));
            }
        }
        if (!ConfigUtils.isEmpty(token)) {
            if (ConfigUtils.isDefault(token)) {
                map.put(Constants.TOKEN_KEY, UUID.randomUUID().toString());
            } else {
                map.put(Constants.TOKEN_KEY, token);
            }
        }
        if (Constants.LOCAL_PROTOCOL.equals(protocolConfig.getName())) {
            protocolConfig.setRegister(false);
            map.put("notify", "false");
        }
        // export service
        String contextPath = protocolConfig.getContextpath();
        if ((contextPath == null || contextPath.length() == 0) && provider != null) {
            contextPath = provider.getContextpath();
        }

        String host = this.findConfigedHosts(protocolConfig, registryURLs, map);
        Integer port = this.findConfigedPorts(protocolConfig, name, map);
        URL url = new URL(name, host, port, (contextPath == null || contextPath.length() == 0 ? "" : contextPath + "/") + path, map);

        if (ExtensionLoader.getExtensionLoader(ConfiguratorFactory.class)
                .hasExtension(url.getProtocol())) {
            url = ExtensionLoader.getExtensionLoader(ConfiguratorFactory.class)
                    .getExtension(url.getProtocol()).getConfigurator(url).configure(url);
        }

        String scope = url.getParameter(Constants.SCOPE_KEY);
        // don't export when none is configured
        if (!Constants.SCOPE_NONE.toString().equalsIgnoreCase(scope)) {

            // export to local if the config is not remote (export to remote only when config is remote)
            if (!Constants.SCOPE_REMOTE.toString().equalsIgnoreCase(scope)) {
                exportLocal(url);
            }
            // export to remote if the config is not local (export to local only when config is local)
            if (!Constants.SCOPE_LOCAL.toString().equalsIgnoreCase(scope)) {
                if (logger.isInfoEnabled()) {
                    logger.info("Export dubbo service " + interfaceClass.getName() + " to url " + url);
                }
                if (registryURLs != null && !registryURLs.isEmpty()) {
                    for (URL registryURL : registryURLs) {
                        url = url.addParameterIfAbsent(Constants.DYNAMIC_KEY, registryURL.getParameter(Constants.DYNAMIC_KEY));
                        URL monitorUrl = loadMonitor(registryURL);
                        if (monitorUrl != null) {
                            url = url.addParameterAndEncoded(Constants.MONITOR_KEY, monitorUrl.toFullString());
                        }
                        if (logger.isInfoEnabled()) {
                            logger.info("Register dubbo service " + interfaceClass.getName() + " url " + url + " to registry " + registryURL);
                        }
                        // 从代理工厂获取执行器，即包装需要暴露的serviceImpl实现类从而得到执行器invoker 
                        Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class) interfaceClass, registryURL.addParameterAndEncoded(Constants.EXPORT_KEY, url.toFullString())); 
                        // 继续包装执行器
                        DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);
						// 断点6：继续执行暴露服务，进入到DubboProtocal的export
                        Exporter<?> exporter = protocol.export(wrapperInvoker); 
                        exporters.add(exporter);
                    }
                } else {
                    Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class) interfaceClass, url);
                    DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);

                    Exporter<?> exporter = protocol.export(wrapperInvoker);
                    exporters.add(exporter);
                }
            }
        }
        this.urls.add(url);
    }

```

* 断点6：继续执行暴露服务，进入到RegistryProtocol的export方法

```
@Override
public <T> Exporter<T> export(final Invoker<T> originInvoker) throws RpcException {
    //export invoker
    // 会进入DubboProtocal的export方法获取暴露执行器，即暴露器
    final ExporterChangeableWrapper<T> exporter = doLocalExport(originInvoker); 

    URL registryUrl = getRegistryUrl(originInvoker);
    
    //registry provider
    final Registry registry = getRegistry(originInvoker);
    final URL registedProviderUrl = getRegistedProviderUrl(originInvoker);
    
    //to judge to delay publish whether or not
    boolean register = registedProviderUrl.getParameter("register", true);
// 端口7：注册服务提供者和消费者到注册表
    ProviderConsumerRegTable.registerProvider(originInvoker, registryUrl, registedProviderUrl);
    
    if (register) {
        register(registryUrl, registedProviderUrl); // 注册中心注册服务
        ProviderConsumerRegTable.getProviderWrapper(originInvoker).setReg(true);
    }
    
    // Subscribe the override data
    // FIXME When the provider subscribes, it will affect the scene : a certain JVM exposes the service and call the same service. Because the subscribed is cached key with the name of the service, it causes the subscription information to cover.
    final URL overrideSubscribeUrl = getSubscribedOverrideUrl(registedProviderUrl);
    final OverrideListener overrideSubscribeListener = new OverrideListener(overrideSubscribeUrl, originInvoker);
    overrideListeners.put(overrideSubscribeUrl, overrideSubscribeListener);
    registry.subscribe(overrideSubscribeUrl, overrideSubscribeListener);
    //Ensure that a new exporter instance is returned every time export
    return new DestroyableExporter<T>(exporter, originInvoker, overrideSubscribeUrl, registedProviderUrl);

}
```

* 在执行RegistryProtocol的export方法时，方法里的doLocalExport方法会进入到DubboProtocal的export方法。会启动Netty服务器监听服务端口

```
	@Override
    public <T> Exporter<T> export(Invoker<T> invoker) throws RpcException {
        URL url = invoker.getUrl();

        // export service.
        String key = serviceKey(url);
        DubboExporter<T> exporter = new DubboExporter<T>(invoker, key, exporterMap);
        exporterMap.put(key, exporter);

        //export an stub service for dispatching event
        Boolean isStubSupportEvent = url.getParameter(Constants.STUB_EVENT_KEY, Constants.DEFAULT_STUB_EVENT);
        Boolean isCallbackservice = url.getParameter(Constants.IS_CALLBACK_SERVICE, false);
        if (isStubSupportEvent && !isCallbackservice) {
            String stubServiceMethods = url.getParameter(Constants.STUB_EVENT_METHODS_KEY);
            if (stubServiceMethods == null || stubServiceMethods.length() == 0) {
                if (logger.isWarnEnabled()) {
                    logger.warn(new IllegalStateException("consumer [" + url.getParameter(Constants.INTERFACE_KEY) +
                            "], has set stubproxy support event ,but no stub methods founded."));
                }
            } else {
                stubServiceMethodsMap.put(url.getServiceKey(), stubServiceMethods);
            }
        }

        openServer(url);  // 打开服务器，启动Netty服务器，监听dubbo服务端口
        optimizeSerialization(url);
        return exporter;
    }
    
    // 打开服务器，启动Netty服务器，监听dubbo服务端口
    private void openServer(URL url) {
        // find server.
        String key = url.getAddress();
        //client can export a service which's only for server to invoke
        boolean isServer = url.getParameter(Constants.IS_SERVER_KEY, true);
        if (isServer) {
            ExchangeServer server = serverMap.get(key);
            if (server == null) {
                serverMap.put(key, createServer(url)); // 创建服务器
            } else {
                // server supports reset, use together with override
                server.reset(url);
            }
        }
    }
```

* 进入断点7：来到ProviderConsumerRegTable的registerProvider方法，注册服务提供者和消费者到注册表

```
// 保存到ConcurrentHashMap。key保存服务提供者的每个URL地址，value保存对应的服务提供者执行器，执行器里才有真正的服务对象、servieImpl服务实现对象
   public static ConcurrentHashMap<String, Set<ProviderInvokerWrapper>> providerInvokers = new ConcurrentHashMap<String, Set<ProviderInvokerWrapper>>();
    public static ConcurrentHashMap<String, Set<ConsumerInvokerWrapper>> consumerInvokers = new ConcurrentHashMap<String, Set<ConsumerInvokerWrapper>>();

	public static void registerProvider(Invoker invoker, URL registryUrl, URL providerUrl) {
        ProviderInvokerWrapper wrapperInvoker = new ProviderInvokerWrapper(invoker, registryUrl, providerUrl);
        String serviceUniqueName = providerUrl.getServiceKey();
        Set<ProviderInvokerWrapper> invokers = providerInvokers.get(serviceUniqueName); // 获取服务提供者执行器
        if (invokers == null) {
            providerInvokers.putIfAbsent(serviceUniqueName, new ConcurrentHashSet<ProviderInvokerWrapper>());
            invokers = providerInvokers.get(serviceUniqueName);
           
        invokers.add(wrapperInvoker); // 添加需要暴露服务的所有信息，至此服务就暴露完成
    }
```

#### 远程服务调用源码

* 解析远程调用，也就是解析 dubbo:reference 标签，进入ReferenceBean.class

```
public class ReferenceBean<T> extends ReferenceConfig<T> implements FactoryBean, ApplicationContextAware, InitializingBean, DisposableBean {
	...
	// 调用getObject方法从Bean工厂FactoryBean获取bean
	@Override
    public Object getObject() throws Exception {
        return get(); // 断点1
    }
}
```

* 断点1：进入ReferenceConfig的get方法

```
    public synchronized T get() {
        if (destroyed) {
            throw new IllegalStateException("Already destroyed!");
        }
        if (ref == null) {
            init();  // 断点2：引用为空，则初始化 
        }
        return ref;
    }
```

* 进入断点2：进入init方法实现

```
 //attributes are stored by system context.
 StaticContext.getSystemContext().putAll(attributes);
 ref = createProxy(map); // 断点3：创建代理对象
 ConsumerModel consumerModel = new ConsumerModel(getUniqueServiceName(), this, ref, interfaceClass.getMethods());
 ApplicationModel.initConsumerModel(getUniqueServiceName(), consumerModel);
```

* 断点3：进入createProxy方法

```
@SuppressWarnings({"unchecked", "rawtypes", "deprecation"})
    private T createProxy(Map<String, String> map) {
        URL tmpUrl = new URL("temp", "localhost", 0, map);
        final boolean isJvmRefer;
        if (isInjvm() == null) {
            if (url != null && url.length() > 0) { // if a url is specified, don't do local reference
                isJvmRefer = false;
            } else if (InjvmProtocol.getInjvmProtocol().isInjvmRefer(tmpUrl)) {
                // by default, reference local service if there is
                isJvmRefer = true;
            } else {
                isJvmRefer = false;
            }
        } else {
            isJvmRefer = isInjvm().booleanValue();
        }

        if (isJvmRefer) {
            URL url = new URL(Constants.LOCAL_PROTOCOL, NetUtils.LOCALHOST, 0, interfaceClass.getName()).addParameters(map);
            invoker = refprotocol.refer(interfaceClass, url);
            if (logger.isInfoEnabled()) {
                logger.info("Using injvm service " + interfaceClass.getName());
            }
        } else {
            if (url != null && url.length() > 0) { // user specified URL, could be peer-to-peer address, or register center's address.
                String[] us = Constants.SEMICOLON_SPLIT_PATTERN.split(url);
                if (us != null && us.length > 0) {
                    for (String u : us) {
                        URL url = URL.valueOf(u);
                        if (url.getPath() == null || url.getPath().length() == 0) {
                            url = url.setPath(interfaceName);
                        }
                        if (Constants.REGISTRY_PROTOCOL.equals(url.getProtocol())) {
                            urls.add(url.addParameterAndEncoded(Constants.REFER_KEY, StringUtils.toQueryString(map)));
                        } else {
                            urls.add(ClusterUtils.mergeUrl(url, map));
                        }
                    }
                }
            } else { // assemble URL from register center's configuration
                List<URL> us = loadRegistries(false);
                if (us != null && !us.isEmpty()) {
                    for (URL u : us) {
                        URL monitorUrl = loadMonitor(u);
                        if (monitorUrl != null) {
                            map.put(Constants.MONITOR_KEY, URL.encode(monitorUrl.toFullString()));
                        }
                        urls.add(u.addParameterAndEncoded(Constants.REFER_KEY, StringUtils.toQueryString(map)));
                    }
                }
                if (urls == null || urls.isEmpty()) {
                    throw new IllegalStateException("No such any registry to reference " + interfaceName + " on the consumer " + NetUtils.getLocalHost() + " use dubbo version " + Version.getVersion() + ", please config <dubbo:registry address=\"...\" /> to your spring config.");
                }
            }

            if (urls.size() == 1) {
 // 断点4：远程引用service接口
                invoker = refprotocol.refer(interfaceClass, urls.get(0));
            } else {
                List<Invoker<?>> invokers = new ArrayList<Invoker<?>>();
                URL registryURL = null;
                for (URL url : urls) {
                    invokers.add(refprotocol.refer(interfaceClass, url));
                    if (Constants.REGISTRY_PROTOCOL.equals(url.getProtocol())) {
                        registryURL = url; // use last registry url
                    }
                }
                if (registryURL != null) { // registry url is available
                    // use AvailableCluster only when register's cluster is available
                    URL u = registryURL.addParameter(Constants.CLUSTER_KEY, AvailableCluster.NAME);
                    invoker = cluster.join(new StaticDirectory(u, invokers));
                } else { // not a registry url
                    invoker = cluster.join(new StaticDirectory(invokers));
                }
            }
        }

        Boolean c = check;
        if (c == null && consumer != null) {
            c = consumer.isCheck();
        }
        if (c == null) {
            c = true; // default true
        }
        if (c && !invoker.isAvailable()) {
            throw new IllegalStateException("Failed to check the status of the service " + interfaceName + ". No provider available for the service " + (group == null ? "" : group + "/") + interfaceName + (version == null ? "" : ":" + version) + " from the url " + invoker.getUrl() + " to the consumer " + NetUtils.getLocalHost() + " use dubbo version " + Version.getVersion());
        }
        if (logger.isInfoEnabled()) {
            logger.info("Refer dubbo service " + interfaceClass.getName() + " from url " + invoker.getUrl());
        }
        // create service proxy
        return (T) proxyFactory.getProxy(invoker);
    }
```

* 断点4：进入RegistryProtocol的refer方法，远程引用service接口

```
@Override
    @SuppressWarnings("unchecked")
    public <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException {
        url = url.setProtocol(url.getParameter(Constants.REGISTRY_KEY, 	               Constants.DEFAULT_REGISTRY)).removeParameter(Constants.REGISTRY_KEY);
        
        Registry registry = registryFactory.getRegistry(url);  // 根据注册中心地址url得到注册中心信息
        if (RegistryService.class.equals(type)) {
            return proxyFactory.getInvoker((T) registry, type, url);
        }

        // group="a,b" or group="*"
        Map<String, String> qs = StringUtils.parseQueryString(url.getParameterAndDecoded(Constants.REFER_KEY));
        String group = qs.get(Constants.GROUP_KEY);
        if (group != null && group.length() > 0) {
            if ((Constants.COMMA_SPLIT_PATTERN.split(group)).length > 1
                    || "*".equals(group)) {
                return doRefer(getMergeableCluster(), registry, type, url);
            }
        }
        // 断点5：引用服务
        return doRefer(cluster, registry, type, url);
    }
```

* 断点5：引用服务，进入RegistryProtocol 的 doRefer方法

```
	private <T> Invoker<T> doRefer(Cluster cluster, Registry registry, Class<T> type, URL url) {
        RegistryDirectory<T> directory = new RegistryDirectory<T>(type, url);
        directory.setRegistry(registry);
        directory.setProtocol(protocol);
        // all attributes of REFER_KEY
        Map<String, String> parameters = new HashMap<String, String>(directory.getUrl().getParameters());
        URL subscribeUrl = new URL(Constants.CONSUMER_PROTOCOL, parameters.remove(Constants.REGISTER_IP_KEY), 0, type.getName(), parameters);
        if (!Constants.ANY_VALUE.equals(url.getServiceInterface())
                && url.getParameter(Constants.REGISTER_KEY, true)) {
            registry.register(subscribeUrl.addParameters(Constants.CATEGORY_KEY, Constants.CONSUMERS_CATEGORY,
                    Constants.CHECK_KEY, String.valueOf(false)));
        }
        // 断点6：开始向注册中心订阅服务提供者对外提供的服务，进入到DubboProtocol的refer方法
        directory.subscribe(subscribeUrl.addParameter(Constants.CATEGORY_KEY,
                Constants.PROVIDERS_CATEGORY
                        + "," + Constants.CONFIGURATORS_CATEGORY
                        + "," + Constants.ROUTERS_CATEGORY));

        Invoker invoker = cluster.join(directory);
        ProviderConsumerRegTable.registerConsumer(invoker, url, subscribeUrl, directory);
        return invoker;
    }
```

* 断点6：进入到DubboProtocol的refer方法，开始向注册中心订阅服务提供者对外提供的服务

```
    @Override
    public <T> Invoker<T> refer(Class<T> serviceType, URL url) throws RpcException {
        optimizeSerialization(url);
        // create rpc invoker.
        // 断点7：getClients(url)：获取客户端
        DubboInvoker<T> invoker = new DubboInvoker<T>(serviceType, url, getClients(url), invokers);
        invokers.add(invoker);
        return invoker;
    }
```

* 断点7：进入getClients(url)获取客户端

```
	private ExchangeClient[] getClients(URL url) {
        // whether to share connection
        boolean service_share_connect = false;
        int connections = url.getParameter(Constants.CONNECTIONS_KEY, 0);
        // if not configured, connection is shared, otherwise, one connection for one service
        if (connections == 0) {
            service_share_connect = true;
            connections = 1;
        }
		// 创建客户端
        ExchangeClient[] clients = new ExchangeClient[connections];
        for (int i = 0; i < clients.length; i++) {
            if (service_share_connect) {
                clients[i] = getSharedClient(url); // 断点8：获取共享客户端
            } else {
                clients[i] = initClient(url);
            }
        }
        return clients;
    }
```

* 断点8：获取共享客户端

```
private ExchangeClient getSharedClient(URL url) {
        String key = url.getAddress();
        ReferenceCountExchangeClient client = referenceClientMap.get(key);
        if (client != null) {
            if (!client.isClosed()) {
                client.incrementAndGetCount();
                return client;
            } else {
                referenceClientMap.remove(key);
            }
        }
        synchronized (key.intern()) {
        // 断点9：初始化客户端
            ExchangeClient exchangeClient = initClient(url); 
            client = new ReferenceCountExchangeClient(exchangeClient, ghostClientMap);
            referenceClientMap.put(key, client);
            ghostClientMap.remove(key);
            return client;
        }
    }
```

* 断点9：初始化客户端，进入initClient方法

```
private ExchangeClient initClient(URL url) {

        // client type setting.
        String str = url.getParameter(Constants.CLIENT_KEY, url.getParameter(Constants.SERVER_KEY, Constants.DEFAULT_REMOTING_CLIENT));

        url = url.addParameter(Constants.CODEC_KEY, DubboCodec.NAME);
        // enable heartbeat by default
        url = url.addParameterIfAbsent(Constants.HEARTBEAT_KEY, String.valueOf(Constants.DEFAULT_HEARTBEAT));

        // BIO is not allowed since it has severe performance issue.
        if (str != null && str.length() > 0 && !ExtensionLoader.getExtensionLoader(Transporter.class).hasExtension(str)) {
            throw new RpcException("Unsupported client type: " + str + "," +
                    " supported client type is " + StringUtils.join(ExtensionLoader.getExtensionLoader(Transporter.class).getSupportedExtensions(), " "));
        }

        ExchangeClient client;
        try {
            // connection should be lazy
            if (url.getParameter(Constants.LAZY_CONNECT_KEY, false)) {
                client = new LazyConnectExchangeClient(url, requestHandler);
            } else {
            // 断点10：进行连接
                client = Exchangers.connect(url, requestHandler);
            }
        } catch (RemotingException e) {
            throw new RpcException("Fail to create remoting client for service(" + url + "): " + e.getMessage(), e);
        }
        return client;
    }
```

* 断点10：进入Exchangers的connect方法进行连接

```
	public static ExchangeClient connect(URL url, ExchangeHandler handler) throws RemotingException {
        if (url == null) {
            throw new IllegalArgumentException("url == null");
        }
        if (handler == null) {
            throw new IllegalArgumentException("handler == null");
        }
        url = url.addParameterIfAbsent(Constants.CODEC_KEY, "exchange");
// 断点11：进入Exchanger 的connect方法进行传输器连接     
        return getExchanger(url).connect(url, handler);
    }
```

* 断点11：进入Exchanger 的connect方法进行传输器连接

```
	@Override
    public ExchangeClient connect(URL url, ExchangeHandler handler) throws RemotingException {
    // 断点12：进入Transporters的connect方法
        return new HeaderExchangeClient(Transporters.connect(url, new DecodeHandler(new HeaderExchangeHandler(handler))), true);
    }
```

* 断点12：进入Transporters的connect方法

```
	public static Client connect(URL url, ChannelHandler... handlers) throws RemotingException {
        if (url == null) {
            throw new IllegalArgumentException("url == null");
        }
        ChannelHandler handler;
        if (handlers == null || handlers.length == 0) {
            handler = new ChannelHandlerAdapter();
        } else if (handlers.length == 1) {
            handler = handlers[0];
        } else {
            handler = new ChannelHandlerDispatcher(handlers);
        }
        // 断点13：进入NettyTransporter的connect方法
        return getTransporter().connect(url, handler);
    }
```

* 断点13：进入NettyTransporter的connect方法，创建Netty客户端，并通过Netty底层监听url的端口，至此创建连接。

  返回断点5：进入到RegistryProtocol 的doRefer方法

```
	@Override
    public Client connect(URL url, ChannelHandler listener) throws RemotingException {
        return new NettyClient(url, listener);
    }
```

* 返回断点5：进入到RegistryProtocol 的doRefer方法

```
	private <T> Invoker<T> doRefer(Cluster cluster, Registry registry, Class<T> type, URL url) {
        RegistryDirectory<T> directory = new RegistryDirectory<T>(type, url);
        directory.setRegistry(registry);
        directory.setProtocol(protocol);
        // all attributes of REFER_KEY
        Map<String, String> parameters = new HashMap<String, String>(directory.getUrl().getParameters());
        URL subscribeUrl = new URL(Constants.CONSUMER_PROTOCOL, parameters.remove(Constants.REGISTER_IP_KEY), 0, type.getName(), parameters);
        if (!Constants.ANY_VALUE.equals(url.getServiceInterface())
                && url.getParameter(Constants.REGISTER_KEY, true)) {
            registry.register(subscribeUrl.addParameters(Constants.CATEGORY_KEY, Constants.CONSUMERS_CATEGORY,
                    Constants.CHECK_KEY, String.valueOf(false)));
        }
        // 断点6：开始向注册中心订阅服务提供者对外提供的服务，进入到DubboProtocol的refer方法
        directory.subscribe(subscribeUrl.addParameter(Constants.CATEGORY_KEY,
                Constants.PROVIDERS_CATEGORY
                        + "," + Constants.CONFIGURATORS_CATEGORY
                        + "," + Constants.ROUTERS_CATEGORY));

		// 封装目标服务的url等配置信息
        Invoker invoker = cluster.join(directory);
        // 同服务暴露流程中，将invoker注册到注册表中
        ProviderConsumerRegTable.registerConsumer(invoker, url, subscribeUrl, directory);
        return invoker;
    }
```

* 保存每一个消费者和提供者的映射信息：url对应的的代理对象invoker，保存在ConcurrentHashMap中

```
public class ProviderConsumerRegTable {
    public static ConcurrentHashMap<String, Set<ProviderInvokerWrapper>> providerInvokers = new ConcurrentHashMap<String, Set<ProviderInvokerWrapper>>();
    public static ConcurrentHashMap<String, Set<ConsumerInvokerWrapper>> consumerInvokers = new ConcurrentHashMap<String, Set<ConsumerInvokerWrapper>>();
```

* 返回断点2：返回到ReferenceConfig的init方法

```
        //attributes are stored by system context.
        StaticContext.getSystemContext().putAll(attributes);
        ref = createProxy(map);  // 创建代理对象完成，封装了invoker
        ConsumerModel consumerModel = new ConsumerModel(getUniqueServiceName(), this, ref, interfaceClass.getMethods());
        ApplicationModel.initConsumerModel(getUniqueServiceName(), consumerModel);
```

![image-20200715220737829](../../../../Software/Typora/Picture/image-20200715220737829.png)

#### 代理对象调用远程服务的过程

![image-20200715230830524](../../../../Software/Typora/Picture/image-20200715230830524.png)

* 进入InvokerInvocationHandler的invoke方法

```
	@Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String methodName = method.getName();  // 方法名
        Class<?>[] parameterTypes = method.getParameterTypes(); // 参数类型信息
        if (method.getDeclaringClass() == Object.class) {
            return method.invoke(invoker, args);
        }
        if ("toString".equals(methodName) && parameterTypes.length == 0) {
            return invoker.toString();
        }
        if ("hashCode".equals(methodName) && parameterTypes.length == 0) {
            return invoker.hashCode();
        }
        if ("equals".equals(methodName) && parameterTypes.length == 1) {
            return invoker.equals(args[0]);
        }
        // 断点1：MockClusterInvoker<T>的invoke方法
        return invoker.invoke(new RpcInvocation(method, args)).recreate();
    }
```

* 断点1：进入MockClusterInvoker<T>的invoke方法

```
@Override
    public Result invoke(Invocation invocation) throws RpcException {
        Result result = null;

        String value = directory.getUrl().getMethodParameter(invocation.getMethodName(), Constants.MOCK_KEY, Boolean.FALSE.toString()).trim();
        if (value.length() == 0 || value.equalsIgnoreCase("false")) {
            //no mock
            // 断点2：invoker：FailoverClusterinvoker，集群容错invoker，集群容错失败后重试到其他服务器。
            result = this.invoker.invoke(invocation);
        } else if (value.startsWith("force")) {
            if (logger.isWarnEnabled()) {
                logger.info("force-mock: " + invocation.getMethodName() + " force-mock enabled , url : " + directory.getUrl());
            }
            //force:direct mock
            result = doMockInvoke(invocation, null);
        } else {
            //fail-mock
            try {
                result = this.invoker.invoke(invocation);
            } catch (RpcException e) {
                if (e.isBiz()) {
                    throw e;
                } else {
                    if (logger.isWarnEnabled()) {
                        logger.warn("fail-mock: " + invocation.getMethodName() + " fail-mock enabled , url : " + directory.getUrl(), e);
                    }
                    result = doMockInvoke(invocation, e);
                }
            }
        }
        return result;
    }
```

* 断点2：进入AbstractClusterInvoker的invoke方法

```
	@Override
    public Result invoke(final Invocation invocation) throws RpcException {
        checkWhetherDestroyed();
        LoadBalance loadbalance = null;
        List<Invoker<T>> invokers = list(invocation); // 从注册中心找到所有能执行的invoker
        if (invokers != null && !invokers.isEmpty()) {
        	// 获取到设置的负载均衡机制
        	on.getMethodName(), Constants.LOADBALANCE_KEY, Constants.DEFAULT_LOADBALANCE));
        }
        RpcUtils.attachInvocationIdIfAsync(getUrl(), invocation);
        // 断点3：进入到FailoverClusterInvoker的doInvoke方法
        return doInvoke(invocation, invokers, loadbalance);
    }	
```

* 断点3：进入到FailoverClusterInvoker的doInvoke方法

```
@Override
    @SuppressWarnings({"unchecked", "rawtypes"})
    public Result doInvoke(Invocation invocation, final List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
        List<Invoker<T>> copyinvokers = invokers;
        checkInvokers(copyinvokers, invocation);
        int len = getUrl().getMethodParameter(invocation.getMethodName(), Constants.RETRIES_KEY, Constants.DEFAULT_RETRIES) + 1;
        if (len <= 0) {
            len = 1;
        }
        // retry loop.
        RpcException le = null; // last exception.
        List<Invoker<T>> invoked = new ArrayList<Invoker<T>>(copyinvokers.size()); // invoked invokers.
        Set<String> providers = new HashSet<String>(len);
        for (int i = 0; i < len; i++) {
            //Reselect before retry to avoid a change of candidate `invokers`.
            //NOTE: if `invokers` changed, then `invoked` also lose accuracy.
            if (i > 0) {
                checkWhetherDestroyed();
                copyinvokers = list(invocation);
                // check again
                checkInvokers(copyinvokers, invocation);
            }
            // 根据负载均衡策略，选择一个invoker
            Invoker<T> invoker = select(loadbalance, invocation, copyinvokers, invoked);
            invoked.add(invoker);
            RpcContext.getContext().setInvokers((List) invoked);
            try {
            	// 断点4：封装Filter和invoker，最终封装为Dubboinvoker。最终来到DubboInvoker的doInvoke方法
                Result result = invoker.invoke(invocation);
                if (le != null && logger.isWarnEnabled()) {
                    logger.warn("Although retry the method " + invocation.getMethodName()
                            + " in the service " + getInterface().getName()
                            + " was successful by the provider " + invoker.getUrl().getAddress()
                            + ", but there have been failed providers " + providers
                            + " (" + providers.size() + "/" + copyinvokers.size()
                            + ") from the registry " + directory.getUrl().getAddress()
                            + " on the consumer " + NetUtils.getLocalHost()
                            + " using the dubbo version " + Version.getVersion() + ". Last error is: "
                            + le.getMessage(), le);
                }
                return result;
            } catch (RpcException e) {
                if (e.isBiz()) { // biz exception.
                    throw e;
                }
                le = e;
            } catch (Throwable e) {
                le = new RpcException(e.getMessage(), e);
            } finally {
                providers.add(invoker.getUrl().getAddress());
            }
        }
        throw new RpcException(le != null ? le.getCode() : 0, "Failed to invoke the method "
                + invocation.getMethodName() + " in the service " + getInterface().getName()
                + ". Tried " + len + " times of the providers " + providers
                + " (" + providers.size() + "/" + copyinvokers.size()
                + ") from the registry " + directory.getUrl().getAddress()
                + " on the consumer " + NetUtils.getLocalHost() + " using the dubbo version "
                + Version.getVersion() + ". Last error is: "
                + (le != null ? le.getMessage() : ""), le != null && le.getCause() != null ? le.getCause() : le);
    }
```

*  断点4：最终来到DubboInvoker的doInvoke方法

```
	@Override
    protected Result doInvoke(final Invocation invocation) throws Throwable {
        RpcInvocation inv = (RpcInvocation) invocation;
        final String methodName = RpcUtils.getMethodName(invocation);
        inv.setAttachment(Constants.PATH_KEY, getUrl().getPath());
        inv.setAttachment(Constants.VERSION_KEY, version);

        ExchangeClient currentClient;
        if (clients.length == 1) {
            currentClient = clients[0]; // 服务客户端，保存了要RPC的服务URL
        } else {
            currentClient = clients[index.getAndIncrement() % clients.length];
        }
        try {
            boolean isAsync = RpcUtils.isAsync(getUrl(), invocation);
            boolean isOneway = RpcUtils.isOneway(getUrl(), invocation);
            int timeout = getUrl().getMethodParameter(methodName, Constants.TIMEOUT_KEY, Constants.DEFAULT_TIMEOUT);
            if (isOneway) {
                boolean isSent = getUrl().getMethodParameter(methodName, Constants.SENT_KEY, false);
                currentClient.send(inv, isSent);
                RpcContext.getContext().setFuture(null);
                return new RpcResult();
            } else if (isAsync) {
                ResponseFuture future = currentClient.request(inv, timeout);
                RpcContext.getContext().setFuture(new FutureAdapter<Object>(future));
                return new RpcResult();
            } else {
                RpcContext.getContext().setFuture(null);
                // 断点5：进入底层通信，客户端发起请求，并将结果获取后返回。成功后进入断点6：AbstractInvoker的invoke方法
                return (Result) currentClient.request(inv, timeout).get();
            }
        } catch (TimeoutException e) {
            throw new RpcException(RpcException.TIMEOUT_EXCEPTION, "Invoke remote method timeout. method: " + invocation.getMethodName() + ", provider: " + getUrl() + ", cause: " + e.getMessage(), e);
        } catch (RemotingException e) {
            throw new RpcException(RpcException.NETWORK_EXCEPTION, "Failed to invoke remote method: " + invocation.getMethodName() + ", provider: " + getUrl() + ", cause: " + e.getMessage(), e);
        }
    }
```

* 断点5：进入底层通信，客户端发起请求，并将结果获取后返回。进入HeaderExchangeChannel的request方法

```
	@Override
    public ResponseFuture request(Object request, int timeout) throws RemotingException {
        if (closed) {
            throw new RemotingException(this.getLocalAddress(), null, "Failed to send request " + request + ", cause: The channel " + this + " is closed!");
        }
        // create request.
        Request req = new Request();
        req.setVersion("2.0.0");
        req.setTwoWay(true);
        req.setData(request);
        DefaultFuture future = new DefaultFuture(channel, req, timeout);
        try {
            channel.send(req);  // 发送通道。返回到断点4
        } catch (RemotingException e) {
            future.cancel();
            throw e;
        }
        return future;
    }
```

* 断点6：进入AbstractInvoker的invoke方法

```
@Override
    public Result invoke(Invocation inv) throws RpcException {
        if (destroyed.get()) {
            throw new RpcException("Rpc invoker for service " + this + " on consumer " + NetUtils.getLocalHost()
                    + " use dubbo version " + Version.getVersion()
                    + " is DESTROYED, can not be invoked any more!");
        }
        RpcInvocation invocation = (RpcInvocation) inv;
        invocation.setInvoker(this);
        if (attachment != null && attachment.size() > 0) {
            invocation.addAttachmentsIfAbsent(attachment);
        }
        Map<String, String> context = RpcContext.getContext().getAttachments();
        if (context != null) {
            invocation.addAttachmentsIfAbsent(context);
        }
        if (getUrl().getMethodParameter(invocation.getMethodName(), Constants.ASYNC_KEY, false)) {
            invocation.setAttachment(Constants.ASYNC_KEY, Boolean.TRUE.toString());
        }
        RpcUtils.attachInvocationIdIfAsync(getUrl(), invocation);


        try {
        	// 断点7：成功拿到响应结果，进入MonitorFilter的invoke方法
            return doInvoke(invocation);
        } catch (InvocationTargetException e) { // biz exception
            Throwable te = e.getTargetException();
            if (te == null) {
                return new RpcResult(e);
            } else {
                if (te instanceof RpcException) {
                    ((RpcException) te).setCode(RpcException.BIZ_EXCEPTION);
                }
                return new RpcResult(te);
            }
        } catch (RpcException e) {
            if (e.isBiz()) {
                return new RpcResult(e);
            } else {
                throw e;
            }
        } catch (Throwable e) {
            return new RpcResult(e);
        }
    }
```

*  断点7：成功拿到响应结果，进入MonitorFilter的invoke方法

```
	@Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        if (invoker.getUrl().hasParameter(Constants.MONITOR_KEY)) {
            RpcContext context = RpcContext.getContext(); // provider must fetch context before invoke() gets called
            String remoteHost = context.getRemoteHost();
            long start = System.currentTimeMillis(); // record start timestamp
            getConcurrent(invoker, invocation).incrementAndGet(); // count up
            try {
                Result result = invoker.invoke(invocation); // proceed invocation chain
                // 成功连接，拿到返回的结果对象
                collect(invoker, invocation, result, remoteHost, start, false);
                return result;
            } catch (RpcException e) {
                collect(invoker, invocation, null, remoteHost, start, true);
                throw e;
            } finally {
                getConcurrent(invoker, invocation).decrementAndGet(); // count down
            }
        } else {
            return invoker.invoke(invocation);
        }
    }
```

* 最后来到ProtocolFilterWrapper的buildInvokerChain方法，完毕

```
@Override
public Result invoke(Invocation invocation) throws RpcException {
	return filter.invoke(next, invocation);
}
```

