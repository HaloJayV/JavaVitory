[TOC]

# 一、Spring Boot与缓存

## Spring缓存抽象

```
Spring从3.1开始定义了org.springframework.cache.Cache

和org.springframework.cache.CacheManager接口来统一不同的缓存技术；

并支持使用JCache（JSR-107）注解简化我们开发；

•Cache接口为缓存的组件规范定义，包含缓存的各种操作集合；  

•Cache接口下Spring提供了各种xxxCache的实现；如RedisCache，EhCacheCache , ConcurrentMapCache等；


•每次调用需要缓存功能的方法时，Spring会检查检查指定参数的指定的目标方法是否已经被调用过；如果有就直接从缓存中获取方法调用后的结果，如果没有就调用方法并缓存结果后返回给用户。下次调用直接从缓存中获取。

•使用Spring缓存抽象时我们需要关注以下两点；

1、确定方法需要被缓存以及他们的缓存策略

2、从缓存中读取之前缓存存储的数据
```

### 重要的缓存注解

| **Cache**          | **缓存接口，定义缓存操作。实现有：****RedisCache****、****EhCacheCache****、****ConcurrentMapCache****等** |
| ------------------ | ------------------------------------------------------------ |
| **CacheManager**   | **缓存管理器，管理各种缓存（****Cache****）组件**            |
| **@Cacheable**     | **主要针对service方法配置，能够根据方法的请求参数对其结果进行缓存** |
| **@CacheEvict**    | **清空缓存**                                                 |
| **@CachePut**      | **保证方法被调用，又希望结果被缓存。**                       |
| **@EnableCaching** | **在启动类开启基于注解的缓存**                               |
| **keyGenerator**   | **缓存数据时**key生成策略                                    |
| **serialize**      | **缓存数据时**value序列化策略                                |

### 使用

| **@Cacheable/@CachePut/@CacheEvict** **主要的参数** |                                                              |                                                              |
| --------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| value                                               | 缓存的名称，在  spring 配置文件中定义，必须指定至少一个      | 例如：     @Cacheable(value=”mycache”) 或者      @Cacheable(value={”cache1”,”cache2”} |
| key                                                 | 缓存的  key，可以为空，如果指定要按照  SpEL 表达式编写，如果不指定，则缺省按照方法的所有参数进行组合 | 例如：     @Cacheable(value=”testcache”,key=”#userName”)     |
| condition                                           | 缓存的条件，可以为空，使用  SpEL 编写，返回  true 或者 false，只有为  true 才进行缓存/清除缓存，在调用方法之前之后都能判断 | 例如：     @Cacheable(value=”testcache”,condition=”#userName.length()>2”) |
| allEntries  (**@CacheEvict**  )                     | 是否清空所有缓存内容，缺省为  false，如果指定为 true，则方法调用后将立即清空所有缓存 | 例如：     @CachEvict(value=”testcache”,allEntries=true)     |
| beforeInvocation  **(@CacheEvict)**                 | 是否在方法执行前就清空，缺省为  false，如果指定为 true，则在方法还没有执行的时候就清空缓存，缺省情况下，如果方法执行抛出异常，则不会清空缓存 | 例如：  @CachEvict(value=”testcache”，beforeInvocation=true) |
| unless  **(@CachePut)**  **(@Cacheable)**           | 用于否决缓存的，不像condition，该表达式只在方法执行之后判断，此时可以拿到返回值result进行判断。条件为true不会缓存，fasle才缓存 | 例如：     @Cacheable(value=”testcache”,unless=”#result  == null”) |

| **名字**            | **位置**               | **描述**                                                     | **示例**                |
| ------------------- | ---------------------- | ------------------------------------------------------------ | ----------------------- |
| methodName          | root object            | 当前被调用的方法名                                           | #root.methodName        |
| method              | root object            | 当前被调用的方法                                             | #root.method.name       |
| target              | root object            | 当前被调用的目标对象                                         | #root.target            |
| targetClass         | root object            | 当前被调用的目标对象类                                       | #root.targetClass       |
| args                | root object            | 当前被调用的方法的参数列表                                   | #root.args[0]           |
| caches              | root object            | 当前方法调用使用的缓存列表（如@Cacheable(value={"cache1",  "cache2"})），则有两个cache | #root.caches[0].name    |
| ***argument name*** | **evaluation context** | **方法参数的名字. 可以直接 #参数名 ，也可以使用 #p0或#a0 的形式，0代表参数的索引；** | **#iban 、 #a0 、 #p0** |
| result              | evaluation context     | 方法执行后的返回值（仅当方法执行之后的判断有效，如‘unless’，’cache put’的表达式 ’cache evict’的表达式beforeInvocation=false） | #result                 |

## 源码

```java
@CacheConfig(cacheNames="emp"/*,cacheManager = "employeeCacheManager"*/) //抽取缓存的公共配置
@Service
public class EmployeeService {

    @Autowired      
    EmployeeMapper employeeMapper;
    /**
     * 将方法的运行结果进行缓存；以后再要相同的数据，直接从缓存中获取，不用调用方法；
     * CacheManager管理多个Cache组件的，对缓存的真正CRUD操作在Cache组件中，每一个缓存组件有自己唯一一个名字；
     *
     * 原理：
     *   1、自动配置类；CacheAutoConfiguration
     *   2、缓存的配置类
     *   org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.JCacheCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.EhCacheCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.HazelcastCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.InfinispanCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.CouchbaseCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.CaffeineCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.GuavaCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration【默认】
     *   org.springframework.boot.autoconfigure.cache.NoOpCacheConfiguration
     *   3、哪个配置类默认生效：SimpleCacheConfiguration；
     *
     *   4、给容器中注册了一个CacheManager：ConcurrentMapCacheManager
     *   5、可以获取和创建ConcurrentMapCache类型的缓存组件；他的作用将数据保存在ConcurrentMap中；
     *
     *   运行流程：
     *   @Cacheable：
     *   1、方法运行之前，先去查询Cache（缓存组件），按照cacheNames指定的名字获取；
     *      （CacheManager先获取相应的缓存），第一次获取缓存如果没有Cache组件会自动创建。
     *   2、去Cache中查找缓存的内容，使用一个key，默认就是方法的参数；
     *      key是按照某种策略生成的；默认是使用keyGenerator生成的，默认使用SimpleKeyGenerator生成key；
     *          SimpleKeyGenerator生成key的默认策略；
     *                  如果没有参数；key=new SimpleKey()；
     *                  如果有一个参数：key=参数的值
     *                  如果有多个参数：key=new SimpleKey(params)；
     *   3、没有查到缓存就调用目标方法；
     *   4、将目标方法返回的结果，放进缓存中
     *
     *   @Cacheable标注的方法执行之前先来检查缓存中有没有这个数据，默认按照参数的值作为key去查询缓存，
     *   如果没有就运行方法并将结果放入缓存；以后再来调用就可以直接使用缓存中的数据；
     *
     *   核心：
     *      1）、使用CacheManager【ConcurrentMapCacheManager】按照名字得到Cache【ConcurrentMapCache】组件
     *      2）、key使用keyGenerator生成的，默认是SimpleKeyGenerator
     *
     *
     *   几个属性：
     *      cacheNames/value：指定缓存组件的名字;将方法的返回结果放在哪个缓存中，是数组的方式，可以指定多个缓存；
     *
     *      key：缓存数据使用的key；可以用它来指定。默认是使用方法参数的值  1-方法的返回值
     *              编写SpEL； #i d;参数id的值   #a0  #p0  #root.args[0]
     *              getEmp[2]
     *
     *      keyGenerator：key的生成器；可以自己指定key的生成器的组件id
     *              key/keyGenerator：二选一使用;
     *
     *
     *      cacheManager：指定缓存管理器；或者cacheResolver指定获取解析器
     *
     *      condition：指定符合条件的情况下才缓存；
     *              ,condition = "#id>0"
     *          condition = "#a0>1"：第一个参数的值》1的时候才进行缓存
     *
     *      unless:否定缓存；当unless指定的条件为true，方法的返回值就不会被缓存；可以获取到结果进行判断
     *              unless = "#result == null"
     *              unless = "#a0==2":如果第一个参数的值是2，结果不缓存；
     *      sync：是否使用异步模式
     * @param id
     * @return
     *
     */
    @Cacheable(value = {"emp"}/*,keyGenerator = "myKeyGenerator",condition = "#a0>1",unless = "#a0==2"*/)
    public Employee getEmp(Integer id){
        System.out.println("查询"+id+"号员工");
        Employee emp = employeeMapper.getEmpById(id);
        return emp;
    }

    /**
     * @CachePut：既调用方法，又更新缓存数据；同步更新缓存
     * 修改了数据库的某个数据，同时更新缓存；
     * 运行时机：
     *  1、先调用目标方法
     *  2、将目标方法的结果缓存起来
     *
     * 测试步骤：
     *  1、查询1号员工；查到的结果会放在缓存中；
     *          key：1  value：lastName：张三
     *  2、以后查询还是之前的结果
     *  3、更新1号员工；【lastName:zhangsan；gender:0】
     *          将方法的返回值也放进缓存了；
     *          key：传入的employee对象  值：返回的employee对象；
     *  4、查询1号员工？
     *      应该是更新后的员工；
     *          key = "#employee.id":使用传入的参数的员工id；
     *          key = "#result.id"：使用返回后的id
     *             @Cacheable的key是不能用#result
     *      为什么是没更新前的？【1号员工没有在缓存中更新】
     *
     */
    @CachePut(/*value = "emp",*/key = "#result.id")
    public Employee updateEmp(Employee employee){
        System.out.println("updateEmp:"+employee);
        employeeMapper.updateEmp(employee);
        return employee;
    }

    /**
     * @CacheEvict：缓存清除
     *  key：指定要清除的数据
     *  allEntries = true：指定清除这个缓存中所有的数据
     *  beforeInvocation = false：缓存的清除是否在方法之前执行
     *      默认代表缓存清除操作是在方法执行之后执行;如果出现异常缓存就不会清除
     *
     *  beforeInvocation = true：
     *      代表清除缓存操作是在方法运行之前执行，无论方法是否出现异常，缓存都清除
     */
    @CacheEvict(value="emp",beforeInvocation = true/*key = "#id",*/)
    public void deleteEmp(Integer id){
        System.out.println("deleteEmp:"+id);
        //employeeMapper.deleteEmpById(id);
        int i = 10/0;
    }

    // @Caching 定义复杂的缓存规则
    @Caching(
         cacheable = {
             @Cacheable(/*value="emp",*/key = "#lastName")
         },
         put = {
             @CachePut(/*value="emp",*/key = "#result.id"),
             @CachePut(/*value="emp",*/key = "#result.email")
         }
    )
    public Employee getEmpByLastName(String lastName){
        return employeeMapper.getEmpByLastName(lastName);
    }


```

### 整合redis

```
1.引入spring-boot-starter-data-redis

2.application.yml配置redis连接地址

3.使用RestTemplate操作redis

    1.redisTemplate.opsForValue();//操作字符串

    2.redisTemplate.opsForHash();//操作hash

    3.redisTemplate.opsForList();//操作list

    4.redisTemplate.opsForSet();//操作set

    5.redisTemplate.opsForZSet();//操作有序set

4.配置缓存、CacheManagerCustomizers

5.测试使用缓存、切换缓存、 CompositeCacheManager
```

* 代码

```java

@Configuration
public class MyRedisConfig {

    // 改变默认序列化规则为json格式，默认是序列化为jdk
    @Bean
    public RedisTemplate<Object, Employee> empRedisTemplate(
            RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<Object, Employee> template = new RedisTemplate<Object, Employee>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Employee> ser = new Jackson2JsonRedisSerializer<Employee>(Employee.class);
        template.setDefaultSerializer(ser);
        return template;
    }
    @Bean
    public RedisTemplate<Object, Department> deptRedisTemplate(
            RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<Object, Department> template = new RedisTemplate<Object, Department>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Department> ser = new Jackson2JsonRedisSerializer<Department>(Department.class);
        template.setDefaultSerializer(ser);
        return template;
    }

    //CacheManagerCustomizers可以来定制缓存的一些规则
    @Primary  //将某个缓存管理器作为默认的
    @Bean
    public RedisCacheManager employeeCacheManager(RedisTemplate<Object, Employee> empRedisTemplate){
        RedisCacheManager cacheManager = new RedisCacheManager(empRedisTemplate);
        //key多了一个前缀
        //使用前缀，默认会将CacheName作为key的前缀
        cacheManager.setUsePrefix(true);
        return cacheManager;
    }

    @Bean
    public RedisCacheManager deptCacheManager(RedisTemplate<Object, Department> deptRedisTemplate){
        RedisCacheManager cacheManager = new RedisCacheManager(deptRedisTemplate);
        //key多了一个前缀
        //使用前缀，默认会将CacheName作为key的前缀
        cacheManager.setUsePrefix(true);
        return cacheManager;
    }
}
```

```java

@RunWith(SpringRunner.class)
@SpringBootTest
public class Springboot01CacheApplicationTests {
	@Autowired
	EmployeeMapper employeeMapper;

	@Autowired
	StringRedisTemplate stringRedisTemplate;  //操作k-v都是字符串的

	@Autowired
	RedisTemplate redisTemplate;  //k-v都是对象的

	@Autowired
	RedisTemplate<Object, Employee> empRedisTemplate;

	/**
	 * Redis常见的五大数据类型
	 *  String（字符串）、List（列表）、Set（集合）、Hash（散列）、ZSet（有序集合）
	 *  stringRedisTemplate.opsForValue()[String（字符串）]
	 *  stringRedisTemplate.opsForList()[List（列表）]
	 *  stringRedisTemplate.opsForSet()[Set（集合）]
	 *  stringRedisTemplate.opsForHash()[Hash（散列）]
	 *  stringRedisTemplate.opsForZSet()[ZSet（有序集合）]
	 */
	@Test
	public void test01(){
		//给redis中保存数据
	    //stringRedisTemplate.opsForValue().append("msg","hello");
//		String msg = stringRedisTemplate.opsForValue().get("msg");
//		System.out.println(msg);

//		stringRedisTemplate.opsForList().leftPush("mylist","1");
//		stringRedisTemplate.opsForList().leftPush("mylist","2");
	}

	//测试保存对象
	@Test
	public void test02(){
		Employee empById = employeeMapper.getEmpById(1);
		//默认如果保存对象，使用jdk序列化机制，序列化后的数据保存到redis中
		//redisTemplate.opsForValue().set("emp-01",empById);
		//1、将数据以json的方式保存
		 //(1)自己将对象转为json
		 //(2)redisTemplate默认的序列化规则；改变默认的序列化规则；
		empRedisTemplate.opsForValue().set("emp-01",empById);
	}

	@Test
	public void contextLoads() {
		Employee empById = employeeMapper.getEmpById(1);
		System.out.println(empById);
	}
}
```

```java

/**
 * 一、搭建基本环境
 * 1、导入数据库文件 创建出department和employee表
 * 2、创建javaBean封装数据
 * 3、整合MyBatis操作数据库
 * 		1.配置数据源信息
 * 		2.使用注解版的MyBatis；
 * 			1）、@MapperScan指定需要扫描的mapper接口所在的包
 * 二、快速体验缓存
 * 		步骤：
 * 			1、开启基于注解的缓存 @EnableCaching
 * 			2、标注缓存注解即可
 * 				@Cacheable
 * 				@CacheEvict
 * 				@CachePut
 * 默认使用的是ConcurrentMapCacheManager==ConcurrentMapCache；将数据保存在	ConcurrentMap<Object, Object>中
 * 开发中使用缓存中间件；redis、memcached、ehcache；
 * 三、整合redis作为缓存
 * Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。
 * 	1、安装redis：使用docker；
 * 	2、引入redis的starter
 * 	3、配置redis
 * 	4、测试缓存
 * 		原理：CacheManager===Cache 缓存组件来实际给缓存中存取数据
 *		1）、引入redis的starter，容器中保存的是 RedisCacheManager；
 *		2）、RedisCacheManager 帮我们创建 RedisCache 来作为缓存组件；RedisCache通过操作redis缓存数据的
 *		3）、默认保存数据 k-v 都是Object；利用序列化保存；如何保存为json
 *   			1、引入了redis的starter，cacheManager变为 RedisCacheManager；
 *   			2、默认创建的 RedisCacheManager 操作redis的时候使用的是 RedisTemplate<Object, Object>
 *   			3、RedisTemplate<Object, Object> 是 默认使用jdk的序列化机制
 *      4）、自定义CacheManager；
 *
 */
@MapperScan("com.atguigu.cache.mapper")
@SpringBootApplication
@EnableCaching
public class Springboot01CacheApplication {

	public static void main(String[] args) {
		SpringApplication.run(Springboot01CacheApplication.class, args);
	}
}

```

```java
@Service
public class DeptService {

    @Autowired
    DepartmentMapper departmentMapper;

    @Qualifier("deptCacheManager")
    @Autowired
    RedisCacheManager deptCacheManager;

    /**
     *  缓存的数据能存入redis；
     *  第二次从缓存中查询就不能反序列化回来；
     *  存的是dept的json数据;CacheManager默认使用RedisTemplate<Object, Employee>操作Redis
     *
     * @param id
     * @return
     */
//    @Cacheable(cacheNames = "dept",cacheManager = "deptCacheManager")
//    public Department getDeptById(Integer id){
//        System.out.println("查询部门"+id);
//        Department department = departmentMapper.getDeptById(id);
//        return department;
//    }

    // 使用缓存管理器得到缓存，进行api调用
    public Department getDeptById(Integer id){
        System.out.println("查询部门"+id);
        Department department = departmentMapper.getDeptById(id);

        //获取某个缓存
        Cache dept = deptCacheManager.getCache("dept");
        dept.put("dept:1",department);

        return department;
    }
}
```



# 二、Spring Boot与消息

### 概述

1.大多应用中，可通过消息服务中间件来提升系统异步通信、扩展解耦能力

2.消息服务中两个重要概念：

​    消息代理（message broker）和目的地（destination）

​	当消息发送者发送消息以后，将由消息代理接管，消息代理保证消息传递到指定目的地。

3.消息队列主要有两种形式的目的地

​		1.队列（queue）：点对点消息通信（point-to-point）

​		2.主题（topic）：发布（publish）/订阅（subscribe）消息通信

4.点对点式：

​		–消息发送者发送消息，消息代理将其放入一个队列中，消息接收者从队列中获取消息内容，消息读取后被移出队列

​		–消息只有唯一的发送者和接受者，但并不是说只能有一个接收者

5.发布订阅式：

​		–发送者（发布者）发送消息到主题，多个接收者（订阅者）监听（订阅）这个主题，那么就会在消息到达时同时收到消息

6.JMS（Java Message Service）JAVA消息服务：

​		–基于JVM消息代理的规范。ActiveMQ、HornetMQ是JMS实现

7.AMQP（Advanced Message Queuing Protocol）

​		–高级消息队列协议，也是一个消息代理的规范，兼容JMS

​		–RabbitMQ是AMQP的实现

#### 8.Spring支持

–**spring-****jms****提供了对****JMS****的支持**

–**spring-rabbit****提供了对****AMQP****的支持**

–**需要****ConnectionFactory****的实现来连接消息代理**

–**提供****JmsTemplate****、****RabbitTemplate****来发送消息**

–**@JmsListener****（****JMS****）、****@RabbitListener****（****AMQP****）注解在方法上监听消息代理发布的消息**

–**@EnableJms****、****@EnableRabbit****开启支持**

9.Spring Boot自动配置

–**JmsAutoConfiguration**

–**RabbitAutoConfiguration**

|              | JMS                                                          | AMQP                                                         |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 定义         | Java  api                                                    | 网络线级协议                                                 |
| 跨语言       | 否                                                           | 是                                                           |
| 跨平台       | 否                                                           | 是                                                           |
| Model        | 提供两种消息模型：  （1）、Peer-2-Peer  （2）、Pub/sub       | 提供了五种消息模型：  （1）、direct  exchange  （2）、fanout  exchange  （3）、topic  change  （4）、headers  exchange  （5）、system  exchange  本质来讲，后四种和JMS的pub/sub模型没有太大差别，仅是在路由机制上做了更详细的划分； |
| 支持消息类型 | 多种消息类型：  TextMessage  MapMessage  BytesMessage  StreamMessage  ObjectMessage  Message  （只有消息头和属性） | byte[]  当实际应用时，有复杂的消息，可以将消息序列化后发送。 |
| 综合评价     | JMS  定义了JAVA  API层面的标准；在java体系中，多个client均可以通过JMS进行交互，不需要应用修改代码，但是其对跨平台的支持较差； | AMQP定义了wire-level层的协议标准；天然具有跨平台、跨语言特性。 |



## RabbitMQ

**RabbitMQ**简介：

RabbitMQ是一个由erlang开发的AMQP(Advanved Message Queue Protocol)的开源实现。



**Message**

消息，消息是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出该消息可能需要持久性存储）等。

**Publisher**

消息的生产者，也是一个向交换器发布消息的客户端应用程序。

**Exchange**

交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。

Exchange有4种类型：direct(默认)，fanout, topic, 和headers，不同类型的Exchange转发消息的策略有所区别

**Queue**

消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。

**Binding**

绑定，用于消息队列和交换器之间的关联。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。

Exchange 和Queue的绑定可以是多对多的关系。

**Connection**

网络连接，比如一个TCP连接。

**Channel**

信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP连接内的虚拟连接，AMQP 命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁 TCP 都是非常昂贵的开销，所以引入了信道的概念，以复用一条 TCP 连接。

Exchange 和Queue的绑定可以是多对多的关系。

**Consumer**

消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。



**Virtual Host**

虚拟主机，表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个 vhost 本质上就是一个 mini 版的 RabbitMQ 服务器，拥有自己的队列、交换器、绑定和权限机制。vhost 是 AMQP 概念的基础，必须在连接时指定，RabbitMQ 默认的 vhost 是 / 。

**Broker**

表示消息队列服务器实体

![image-20200814115406619](../../../../Software/Typora/Picture/image-20200814115406619.png)

### AMQP 中的消息路由

AMQP 中消息的路由过程和 Java 开发者熟悉的 JMS 存在一些差别，AMQP 中增加了 **Exchange** 和 **Binding** 的角色。生产者把消息发布到 Exchange 上，消息最终到达队列并被消费者接收，而 Binding 决定交换器的消息应该发送到那个队列。

![image-20200814120619752](../../../../Software/Typora/Picture/image-20200814120619752.png)





#### Exchange 类型

•**Exchange**分发消息时根据类型的不同分发策略有区别，目前共四种类型：**direct****、****fanout****、****topic****、****headers** 。headers 匹配 AMQP 消息的 header 而不是路由键， headers 交换器和 direct 交换器完全一致，但性能差很多，目前几乎用不到了，所以直接看另外三种类型：

消息中的路由键（routing key）如果和 Binding 中的 binding key 一致， 交换器就将消息发到对应的队列中。路由键与队列名完全匹配，如果一个队列绑定到交换机要求路由键为“dog”，则只转发 routing key 标记为“dog”的消息，不会转发“dog.puppy”，也不会转发“dog.guard”等等。它是完全匹配、单播的模式。

![image-20200814125433211](../../../../Software/Typora/Picture/image-20200814125433211.png)

#### Fanout Exchange

![image-20200814122646736](../../../../Software/Typora/Picture/image-20200814122646736.png)

每个发到 fanout 类型交换器的消息都会分到所有绑定的队列上去。fanout 交换器不处理路由键，只是简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。很像子网广播，每台子网内的主机都获得了一份复制的消息。fanout 类型转发消息是最快的。



##### Topic Exchange

![image-20200814122957081](../../../../Software/Typora/Picture/image-20200814122957081.png)

topic 交换器通过模式匹配分配消息的路由键属性，将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上。它将路由键和绑定键的字符串切分成单词，这些**单词之间用点隔开**。它同样也会识别两个通配符：符号“#”和符号“**”**。**#**匹配**0**个或多个单词**，****匹配一个单词。

### RabbitMQ 整合

* 启动类加上注解 @EnableRabbit

  ```java
  /**
   * 自动配置
   *  1、RabbitAutoConfiguration
   *  2、有自动配置了连接工厂ConnectionFactory；
   *  3、RabbitProperties 封装了 RabbitMQ的配置
   *  4、 RabbitTemplate ：给RabbitMQ发送和接受消息；
   *  5、 AmqpAdmin ： RabbitMQ系统管理功能组件;
   *  	AmqpAdmin：创建和删除 Queue，Exchange，Binding
   *  6、@EnableRabbit +  @RabbitListener 监听消息队列的内容
   *
   */
  @EnableRabbit  //开启基于注解的RabbitMQ模式
  @SpringBootApplication
  public class Springboot02AmqpApplication {
  
  	public static void main(String[] args) {
  		SpringApplication.run(Springboot02AmqpApplication.class, args);
  	}
  }
  ```

* 配置

  ```
  spring.rabbitmq.host=118.24.44.169
  spring.rabbitmq.username=guest
  spring.rabbitmq.password=guest
  #spring.rabbitmq.virtual-host=
  ```

* 测试

  ```java
  @RunWith(SpringRunner.class)
  @SpringBootTest
  public class Springboot02AmqpApplicationTests {
  
  	@Autowired
  	RabbitTemplate rabbitTemplate;
  
  	@Autowired
  	AmqpAdmin amqpAdmin;
  
  	@Test
  	public void createExchange(){
  
  //		amqpAdmin.declareExchange(new DirectExchange("amqpadmin.exchange"));
  //		amqpAdmin.declareQueue(new Queue("amqpadmin.queue",true));
  		//创建绑定规则
  //		amqpAdmin.declareBinding(new Binding("amqpadmin.queue", Binding.DestinationType.QUEUE,"amqpadmin.exchange","amqp.haha",null));
  
  	}
  
  	/**
  	 * 1、单播（点对点）
  	 */
  	@Test
  	public void contextLoads() {
  		//Message需要自己构造一个;定义消息体内容和消息头
  		//rabbitTemplate.send(exchage,routeKey,message);
  
  		//object默认当成消息体，只需要传入要发送的对象，自动序列化发送给rabbitmq；常用
  		//rabbitTemplate.convertAndSend(exchage,routeKey,object);
  		Map<String,Object> map = new HashMap<>();
  		map.put("msg","这是第一个消息");
  		map.put("data", Arrays.asList("helloworld",123,true));
  		//对象被默认序列化以后发送出去
  		rabbitTemplate.convertAndSend("exchange.direct","atguigu.news",new Book("西游记","吴承恩"));
  	}
  
  	//接受数据,如何将数据自动的转为json发送出去
  	@Test
  	public void receive(){
  		Object o = rabbitTemplate.receiveAndConvert("atguigu.news");
  		System.out.println(o.getClass());
  		System.out.println(o);
  	}
  
  	/**
  	 * 广播
  	 */
  	@Test
  	public void sendMsg(){
  		rabbitTemplate.convertAndSend("exchange.fanout","",new Book("红楼梦","曹雪芹"));
  	}
  }
  ```
  
* 设置消息的格式为json

```java
@Configuration
public class MyAMQPConfig {
    @Bean
    public MessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }
}
```

### 监听消息

 

```java

@Service
public class BookService {

    @RabbitListener(queues = "atguigu.news") // 监听队列
    public void receive(Book book){
        System.out.println("收到消息："+book);
    }

    @RabbitListener(queues = "atguigu")
    public void receive02(Message message){
        System.out.println(message.getBody());
        System.out.println(message.getMessageProperties());
    }
}

```



# 三、Spring Boot与检索

* Elasticsearch是一个分布式搜索服务，提供Restful API，底层基于Lucene，采用多shard（分片）的方式保证数据安全，并且提供自动resharding的功能，github等大型的站点也是采用了ElasticSearch作为其搜索服务，

* 一个 ElasticSearch 集群可以 包含多个 *索引* ，相应的每个索引可以包含多个 *类型* 。 这些不同的类型存储着多个 *文档* ，每个文档又有 多个 *属性* 。

* •类似关系：

  ​		–索引-数据库

  ​		–类型-表

  ​		–文档-表中的记录

  ​		–属性-列

![image-20200814155409529](../../../../Software/Typora/Picture/image-20200814155409529.png)

### 整合Elasticsearch

* pom

  ```
  	<!--SpringBoot默认使用SpringData ElasticSearch模块进行操作-->
  		<dependency>
  			<groupId>org.springframework.boot</groupId>
  			<artifactId>spring-boot-starter-data-elasticsearch</artifactId>
  		</dependency>
  ```

* properties

  ```
  spring.data.elasticsearch.cluster-name=elasticsearch
  spring.data.elasticsearch.cluster-nodes=118.24.44.169:9301
  ```

* SpringData ElasticSearch

  ```java
  /**
   * SpringBoot默认支持两种技术来和ES交互；
   * 1、Jest（默认不生效）
   * 	需要导入jest的工具包（io.searchbox.client.JestClient）
   * 2、SpringData ElasticSearch【ES版本有可能不合适】
   * 		版本适配说明：https://github.com/spring-projects/spring-data-elasticsearch
   *		如果版本不适配：2.4.6
   *			1）、升级SpringBoot版本
   *			2）、安装对应版本的ES
   *
   * 		1）、Client 节点信息clusterNodes；clusterName
   * 		2）、ElasticsearchTemplate 操作es
   *		3）、编写一个 ElasticsearchRepository 的子接口来操作ES；
   *	两种用法：https://github.com/spring-projects/spring-data-elasticsearch
   *	1）、编写一个 ElasticsearchRepository
   */
  ```

  ```java
  public interface BookRepository extends ElasticsearchRepository<Book,Integer> {
      //参照
      // https://docs.spring.io/spring-data/elasticsearch/docs/3.0.6.RELEASE/reference/html/
     public List<Book> findByBookNameLike(String bookName); // 自定义索引方法，不需要写实现，按照书名模糊查询
  }
  ```

  ```java
  @Document(indexName = "atguigu",type = "book")  // 保存在哪个索引indexName和类型type
  public class Book {...}
  ```

* 测试

  ```java
  	@Test
  	public void test02(){
  //		Book book = new Book();
  //		book.setId(1);
  //		book.setBookName("西游记");
  //		bookRepository.index(book);   // 索引到es里
  		for (Book book : bookRepository.findByBookNameLike("游")) { // 自定义模糊查询方法
  			System.out.println(book);
  		}
  	}
  ```

* 然在浏览器输入，查询所有索引_：[IP]:[port]/atguigu/book/_search  

# 四、Spring Boot与任务

### 异步任务

在Java应用中，绝大多数情况下都是通过同步的方式来实现交互处理的；但是在处理与第三方系统交互的时候，容易造成响应迟缓的情况，之前大部分都是使用多线程来完成此类任务，其实，在Spring 3.x之后，就已经内置了@Async来完美解决这个问题。

* 在启动类加上注解@EnableAysnc，在目标方法加上注解@Async， 才能开启异步功能

### 定时任务

项目开发中经常需要执行一些定时任务，比如需要在每天凌晨时候，分析一次前一天的日志信息。Spring为我们提供了异步执行任务调度的方式，提供TaskExecutor 、TaskScheduler 接口。

* 同样是2个注解@EnableScheduling、@Scheduled

* 实例

  ```java
  @Service
  public class ScheduledService {
  
      /**
       * second(秒), minute（分）, hour（时）, day of month（日）, month（月）, day of week（周几）.
       * 0 * * * * MON-FRI （4个*分别表示：分、时、日、月，意思是从周一到周五每次0秒启动任务）
       *  【0 0/5 14,18 * * ?】 每天14点整，和18点整，每隔5分钟执行一次
       *  【0 15 10 ? * 1-6】 每个月的周一至周六10:15分执行一次
       *  【0 0 2 ? * 6L】每个月的最后一个周六凌晨2点执行一次
       *  【0 0 2 LW * ?】每个月的最后一个工作日凌晨2点执行一次
       *  【0 0 2-4 ? * 1#1】每个月的第一个周一凌晨2点到4点期间，每个整点都执行一次；
       */
     // @Scheduled(cron = "0 * * * * MON-SAT")
      //@Scheduled(cron = "0,1,2,3,4 * * * * MON-SAT")
     // @Scheduled(cron = "0-4 * * * * MON-SAT")
      @Scheduled(cron = "0/4 * * * * MON-SAT")  //每4秒执行一次
      public void hello(){
          System.out.println("hello ... ");
      }
  }
  ```


### 邮件任务

•邮件发送需要引入spring-boot-starter-mail

•Spring Boot 自动配置MailSenderAutoConfiguration

•定义MailProperties内容，配置在application.yml中

```properties
spring.mail.username=534096094@qq.com
spring.mail.password=gtstkoszjelabijb
spring.mail.host=smtp.qq.com
spring.mail.properties.mail.smtp.ssl.enable=true
```

•自动装配JavaMailSender

•简单邮件发送

```java
	@Autowired
	JavaMailSenderImpl mailSender;

	@Test
	public void contextLoads() {
		SimpleMailMessage message = new SimpleMailMessage();
		//邮件设置
		message.setSubject("通知-今晚开会");
		message.setText("今晚7:30开会");

		message.setTo("17512080612@163.com");
		message.setFrom("534096094@qq.com");

		mailSender.send(message);
	}
```

* 复杂邮件发送

  ```java
  	@Test
  	public void test02() throws  Exception{
  		//1、创建一个复杂的消息邮件
  		MimeMessage mimeMessage = mailSender.createMimeMessage();
  		MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true);
  		//邮件设置
  		helper.setSubject("通知-今晚开会");
  		helper.setText("<b style='color:red'>今天 7:30 开会</b>",true);
  		helper.setTo("17512080612@163.com");
  		helper.setFrom("534096094@qq.com");
  		//上传文件
  		helper.addAttachment("1.jpg",new File("C:\\Users\\lfy\\Pictures\\Saved Pictures\\1.jpg"));
  		helper.addAttachment("2.jpg",new File("C:\\Users\\lfy\\Pictures\\Saved Pictures\\2.jpg"));
  		mailSender.send(mimeMessage);
  	}
  ```

# 五、Spring Boot与安全

### 概述

Spring Security是针对Spring项目的安全框架，也是Spring Boot底层安全模块默认的技术选型。他可以实现强大的web安全控制。对于安全控制，我们仅需引入spring-boot-starter-security模块，进行少量的配置，即可实现强大的安全管理。

* 几个类：

WebSecurityConfigurerAdapter：自定义Security策略

AuthenticationManagerBuilder：自定义认证策略

@EnableWebSecurity：开启WebSecurity模式

•应用程序的两个主要区域是“认证”和“授权”（或者访问控制）。这两个主要区域是Spring Security 的两个目标。

•“认证”（Authentication），是建立一个他声明的主体的过程（一个“主体”一般是指用户，设备或一些可以在你的应用程序中执行动作的其他系统）。

•“授权”（Authorization）指确定一个主体是否允许在你的应用程序执行一个动作的过程。为了抵达需要授权的店，主体的身份已经有认证过程建立。

•这个概念是通用的而不只在Spring Security中。

### Web安全

1.登陆/注销

​	–HttpSecurity配置登陆、注销功能

2.Thymeleaf提供的SpringSecurity标签支持

​	–需要引入thymeleaf-extras-springsecurity4

​	–sec:authentication=“name”获得当前用户的用户名

​	–sec:authorize=“hasRole(‘ADMIN’)”当前用户必须拥有ADMIN权限时才会显示标签内容

3.remember me

​	–表单添加remember-me的checkbox

​	–配置启用remember-me功能

4.CSRF（Cross-site request forgery）跨站请求伪造

​	–HttpSecurity启用csrf功能，会为表单添加_csrf的值，提交携带来预防CSRF；

### 配置类

```java
@EnableWebSecurity
public class MySecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //super.configure(http);
        //定制请求的授权规则
        http.authorizeRequests().antMatchers("/").permitAll()
                .antMatchers("/level1/**").hasRole("VIP1")	
                .antMatchers("/level2/**").hasRole("VIP2")
                .antMatchers("/level3/**").hasRole("VIP3");
        //开启自动配置的登陆功能，效果，如果没有登陆，没有权限就会来到登陆页面
        http.formLogin().usernameParameter("user").passwordParameter("pwd") // 用户名密码参数与themleaf输入的参数绑定
                .loginPage("/userlogin"); // 修改默认登录请求，即未登录时转到登录页面时的请求
        //1、/login来到登陆页
        //2、重定向到/login?error表示登陆失败
        //3、更多详细规定
        //4、默认post形式的 /login代表处理登陆
        //5、一但定制loginPage；那么 loginPage的post请求就是登陆
        //开启自动配置的注销功能。
        http.logout().logoutSuccessUrl("/");//注销成功以后来到首页
        //1、访问 /logout 表示用户注销，清空session
        //2、注销成功会返回 /login?logout 页面；

        //开启记住我功能,默认过期时间为14天，用户信息放在cookie中
        http.rememberMe().rememberMeParameter("remeber"); // remember为前端的checkbox的name
        //登陆成功以后，将cookie发给浏览器保存，以后访问页面带上这个cookie，只要通过检查就可以免登录
        //点击注销会删除cookie
    }

    //定义认证规则
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        //super.configure(auth);
        auth.inMemoryAuthentication()
                .withUser("zhangsan").password("123456").roles("VIP1","VIP2")
                .and()
                .withUser("lisi").password("123456").roles("VIP2","VIP3")
                .and()
                .withUser("wangwu").password("123456").roles("VIP1","VIP3");
    }
}
```

* security整合themlef

  * 登录页面

    ```html
    <!DOCTYPE html>
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
    <meta charset="UTF-8">
    <title>Insert title here</title>
    </head>
    <body>
    	<h1 align="center">欢迎登陆武林秘籍管理系统</h1>
    	<hr>
    	<div align="center">
    		<form th:action="@{/userlogin}" method="post">
    			用户名:<input name="user"/><br>
    			密码:<input name="pwd"><br/>
    			<input type="checkbox" name="remeber"> 记住我<br/>
    			<input type="submit" value="登陆">
    		</form>
    	</div>
    </body>
    </html>
    ```

    

  ```html
  <html xmlns:th="http://www.thymeleaf.org"
  	  xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity4">
  <head>
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
  <title>Insert title here</title>
  </head>
  <body>
  <h1 align="center">欢迎光临武林秘籍管理系统</h1>
  <div sec:authorize="!isAuthenticated()">
  	<h2 align="center">游客您好，如果想查看武林秘籍 <a th:href="@{/userlogin}">请登录</a></h2>
  </div>
  <div sec:authorize="isAuthenticated()">
  	<h2><span sec:authentication="name"></span>，您好,您的角色有：
  		<span sec:authentication="principal.authorities"></span></h2>
  	<form th:action="@{/logout}" method="post">
  		<input type="submit" value="注销"/>
  	</form>
  </div>
  <hr>
  <div sec:authorize="hasRole('VIP1')">
  	<h3>普通武功秘籍</h3>
  	<ul>
  		<li><a th:href="@{/level1/1}">罗汉拳</a></li>
  		<li><a th:href="@{/level1/2}">武当长拳</a></li>
  		<li><a th:href="@{/level1/3}">全真剑法</a></li>
  	</ul>
  </div>
  <div sec:authorize="hasRole('VIP2')">
  	<h3>高级武功秘籍</h3>
  	<ul>
  		<li><a th:href="@{/level2/1}">太极拳</a></li>
  		<li><a th:href="@{/level2/2}">七伤拳</a></li>
  		<li><a th:href="@{/level2/3}">梯云纵</a></li>
  	</ul>
  </div>
  
  <div sec:authorize="hasRole('VIP3')">
  	<h3>绝世武功秘籍</h3>
  	<ul>
  		<li><a th:href="@{/level3/1}">葵花宝典</a></li>
  		<li><a th:href="@{/level3/2}">龟派气功</a></li>
  		<li><a th:href="@{/level3/3}">独孤九剑</a></li>
  	</ul>
  </div>
  </body>
  </html>
  ```

  

# 六、Spring Boot与分布式

•**ZooKeeper**

ZooKeeper 是一个分布式的，开放源码的分布式应用程序协调服务。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。

•**Dubbo**

Dubbo是Alibaba开源的分布式服务框架，它最大的特点是按照分层的方式来架构，使用这种方式可以使各个层之间解耦合（或者最大限度地松耦合）。从服务模型的角度来看，Dubbo采用的是一种非常简单的模型，要么是提供方提供服务，要么是消费方消费服务，所以基于这一点可以抽象出服务提供方（Provider）和服务消费方（Consumer）两个角色。

![image-20200816124836747](../../../../Software/Typora/Picture/image-20200816124836747.png)

### 服务提供者

* pom

  ```
  <!--引入zookeeper的客户端工具-->
  		<!-- https://mvnrepository.com/artifact/com.github.sgroschupf/zkclient -->
  		<dependency>
  			<groupId>com.github.sgroschupf</groupId>
  			<artifactId>zkclient</artifactId>
  			<version>0.1</version>
  		</dependency>
  ```

* application.properties

  ```
  dubbo.application.name=provider-ticket
  
  dubbo.registry.address=zookeeper://118.24.44.169:2181
  
  dubbo.scan.base-packages=com.atguigu.ticket.service
  ```

* 启动类

  ```java
  /**
   * 1、将服务提供者注册到注册中心
   * 	    1、引入dubbo和zkclient相关依赖
   * 	    2、配置dubbo的扫描包和注册中心地址
   * 	    3、使用@Service发布服务
   */
  @SpringBootApplication
  public class ProviderTicketApplication {
  
  	public static void main(String[] args) {
  		SpringApplication.run(ProviderTicketApplication.class, args);
  	}
  }
  ```

* service

  ```
  public interface TicketService {
      public String getTicket();
  }
  
  ```

* serviceImpl

  ```java
  @Component
  @Service //将服务发布出去
  public class TicketServiceImpl implements TicketService {
      @Override
      public String getTicket() {
          return "《厉害了，我的国》";
      }
  }
  ```

### 服务消费者

* pom

  ```
  <!--引入zookeeper的客户端工具-->
  		<!-- https://mvnrepository.com/artifact/com.github.sgroschupf/zkclient -->
  		<dependency>
  			<groupId>com.github.sgroschupf</groupId>
  			<artifactId>zkclient</artifactId>
  			<version>0.1</version>
  		</dependency>
  ```

* application.properties

  ```
  dubbo.application.name=consumer-user
  
  dubbo.registry.address=zookeeper://118.24.44.169:2181
  ```

* 启动类

  ```java
  /**
   * 1、引入依赖‘
   * 2、配置dubbo的注册中心地址
   * 3、引用服务
   */
  @SpringBootApplication
  public class ConsumerUserApplication {
  
  	public static void main(String[] args) {
  		SpringApplication.run(ConsumerUserApplication.class, args);
  	}
  }
  
  ```

* 调用注册中心的服务接口

  ```
  public interface TicketService {
      public String getTicket();
  }
  ```

* 使用接口方法

  ```java
  @Service
  public class UserService{
  
      @Reference
      TicketService ticketService;
  
      public void hello(){
          String ticket = ticketService.getTicket();
          System.out.println("买到票了："+ticket);
      }
  }
  ```


## SpringCloud

### eureka注册中心

* yml 

  ```yaml
  server:
    port: 8761
  eureka:
    instance:
      hostname: eureka-server  # eureka实例的主机名
    client:
      register-with-eureka: false #不把自己注册到eureka上
      fetch-registry: false #不从eureka上来获取服务的注册信息
      service-url:
        defaultZone: http://localhost:8761/eureka/
  ```

* 启动类注解：

  ```
  @EnableEurekaServer
  @SpringBootApplication
  ```

### 服务提供者 

* yml

  ```yaml
  server:
    port: 8002
  spring:
    application:
      name: provider-ticket
  
  eureka:
    instance:
      prefer-ip-address: true # 注册服务的时候使用服务的ip地址
    client:
      service-url:
        defaultZone: http://localhost:8761/eureka/
  ```

* serviceImpl类标注@Service时已经注册到注册中心

### 服务消费者

* yml 

  ```yaml
  spring:
    application:
      name: consumer-user
  server:
    port: 8200
  
  eureka:
    instance:
      prefer-ip-address: true # 注册服务的时候使用服务的ip地址
    client:
      service-url:
        defaultZone: http://localhost:8761/eureka/
  ```

* 启动类加上注解：@EnableDiscoveryClient //开启发现服务功能

  ```java
  @EnableDiscoveryClient //开启发现服务功能
  @SpringBootApplication
  public class ConsumerUserApplication {
  
  	public static void main(String[] args) {
  		SpringApplication.run(ConsumerUserApplication.class, args);
  	}
  
  	@LoadBalanced //使用负载均衡机制
  	@Bean
  	public RestTemplate restTemplate(){
  		return new RestTemplate();
  	}
  }
  ```

* controller 

  ```java
  @RestController
  public class UserController {
  
      @Autowired
      RestTemplate restTemplate;
  
      @GetMapping("/buy")
      public String buyTicket(String name){
          String s = restTemplate.getForObject("http://PROVIDER-TICKET/ticket", String.class);
          return name+"购买了"+s;
      }
  }
  ```

# 七、热部署

在开发中我们修改一个Java文件后想看到效果不得不重启应用，这导致大量时间花费，我们希望不重启应用的情况下，程序可以自动部署（热部署）。有以下四种情况，如何能实现热部署。

### Spring Boot Devtools（推荐）

```
<dependency>  
       <groupId>org.springframework.boot</groupId>  
       <artifactId>spring-boot-devtools</artifactId>   
</dependency> 
```

# 八、Spring Boot与监控管理

通过引入spring-boot-starter-actuator，可以使用Spring Boot为我们提供的准生产环境下的应用监控和管理功能。我们可以通过HTTP，JMX，SSH协议来进行操作，自动得到审计、健康及指标信息等

•步骤：

​	–引入spring-boot-starter-actuator

​	–通过http方式访问监控端点；

​	–可进行shutdown（POST 提交，此端点默认关闭）

* properties

  ```properties
  management.security.enabled=false   # 禁用安全防护，才能查看各种监控信息
  spring.redis.host=118.24.44.169
  info.app.id=hello  # 自定义端点info信息
  info.app.version=1.0.0
  #endpoints.metrics.enabled=false
  endpoints.shutdown.enabled=true
  ```

  

* 浏览器输入：localhost:【端口】/【以下端点名】

| **端点名**   | **描述**                    |
| ------------ | --------------------------- |
| *autoconfig* | 所有自动配置信息            |
| auditevents  | 审计事件                    |
| beans        | 所有Bean的信息              |
| configprops  | 所有配置属性                |
| dump         | 线程状态信息                |
| env          | 当前环境信息                |
| health       | 应用健康状况                |
| info         | 当前应用信息                |
| metrics      | 应用的各项指标              |
| mappings     | 应用@RequestMapping映射路径 |
| shutdown     | 关闭当前应用（默认关闭）    |
| trace        | 追踪信息（最新的http请求）  |

* 源码托管到github，需要配置git.properties ，git继承了info

  ```
  git.branch=master
  git.commit.id=xjkd33s
  git.commit.time=2017-12-12 12:12:56
  ```


### 定制端点信息

```
定制端点一般通过endpoints+端点名+属性名来设置。
修改端点id（endpoints.beans.id=mybeans）
开启远程应用关闭功能（endpoints.shutdown.enabled=true）
关闭端点（endpoints.beans.enabled=false）
开启所需端点
endpoints.enabled=false
endpoints.beans.enabled=true
定制端点访问根路径
management.context-path=/manage
关闭http端点，也可以设置管理端点
management.port=-1
```

* 实例：

```properties
management.security.enabled=false   # 禁用安全防护，才能查看各种监控信息

spring.redis.host=118.24.44.169

info.app.id=hello  # 自定义端点info信息
info.app.version=1.0.0

#endpoints.metrics.enabled=false
endpoints.shutdown.enabled=true

#endpoints.beans.id=mybean   # 修改端点名，访问路径默认为该端点名
#endpoints.beans.path=/bean # 修改访问路径
#endpoints.beans.enabled=false  # 禁用该端点

#endpoints.enabled=false  # 关闭所有端点访问
#endpoints.beans.enabled=true

management.context-path=/manage
management.port=8181
```

### 自定义健康状态指示器

```java
/**
 * 自定义健康状态指示器
 * 1、编写一个指示器 实现 HealthIndicator 接口
 * 2、指示器的名字 xxxxHealthIndicator
 * 3、加入容器中
 */
@SpringBootApplication
public class Springboot08ActuatorApplication {

	public static void main(String[] args) {
		SpringApplication.run(Springboot08ActuatorApplication.class, args);
	}
}

```

```java
@Component
public class MyAppHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {

        //自定义的检查方法
        //Health.up().build()代表健康
        return Health.down().withDetail("msg","服务异常").build();
    }
}
```















