[TOC]



## 一 容器

#### 组件注册

* 	```java
    @SuppressWarnings("resource")
    	@Test
    	public void test01(){
    		// MainConfig标注了@Configuration注解，获取标注了@Bean()的bean，以及@ComponentScans扫描组件的Bean。
    		AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
            // Map<String, Person> persons = applicationContext.getBeansOfType(Person.class); 获取该类下所有bean并保存在Map
    		String[] definitionNames = applicationContext.getBeanDefinitionNames(); // 容器内所有Bean名
    		for (String name : definitionNames) {
    			System.out.println(name);
		}
    	}
    ```
    
    
    
* ```java
  //@ComponentScan  value:指定要扫描的包
  //excludeFilters = Filter[] ：指定扫描的时候按照什么规则排除那些组件
  //includeFilters = Filter[] ：指定扫描的时候只需要包含哪些组件
  //FilterType.ANNOTATION：按照注解
  //FilterType.ASSIGNABLE_TYPE：按照给定的类型；
  //FilterType.ASPECTJ：使用ASPECTJ表达式
  //FilterType.REGEX：使用正则指定
  //FilterType.CUSTOM：使用自定义规则
  @ComponentScans(   / / 扫描要注入容器内的bean组件
  		value = {
  				@ComponentScan(value="com.atguigu",includeFilters = {   // 除了包含规则，还有排除规则excludeFilters
  /*					@Filter(type=FilterType.ANNOTATION,classes={Controller.class}),
  						@Filter(type=FilterType.ASSIGNABLE_TYPE,classes={BookService.class}),*/
  						@Filter(type=FilterType.CUSTOM,classes={MyTypeFilter.class}) // 自定义过滤规则，
  				},useDefaultFilters = false)	// useDefaultFilters = false:禁用默认过滤规则，只让自定义的规则生效
  		})
  ```

* 	TypeFilter：自定义过滤规则，

   ```java
   public class MyTypeFilter implements TypeFilter {
   	/**
   	 * metadataReader：读取到的当前正在扫描的类的信息
   	 * metadataReaderFactory:可以获取到其他任何类信息的
   	 */
   	@Override
   	public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory)
   			throws IOException {
   		// TODO Auto-generated method stub
   		//获取当前类注解的信息
   		AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
   		//获取当前正在扫描的类的类信息
   		ClassMetadata classMetadata = metadataReader.getClassMetadata();
   		//获取当前类资源（类的路径）
   		Resource resource = metadataReader.getResource();
   		
   		String className = classMetadata.getClassName();
   		System.out.println("--->"+className); 
   		if(className.contains("er")){ // 扫描带有‘er’的组件并加入容器
   			return true;
   		}
   		return false;
   	}
   }
   ```

* 	```java
   //默认是单实例的
   /**
    * ConfigurableBeanFactory#SCOPE_PROTOTYPE    
    * @see ConfigurableBeanFactory#SCOPE_SINGLETON  
    * @see org.springframework.web.context.WebApplicationContext#SCOPE_REQUEST  request
    * @see org.springframework.web.context.WebApplicationContext#SCOPE_SESSION	 sesssion
    * @return\
    * @Scope:调整作用域
    * prototype：多实例的：ioc容器启动并不会去调用方法创建对象放在容器中。
    * 					每次获取的时候才会调用方法创建对象；每次创建bean实例都是不一样的实例对象
    * singleton：单实例的（默认值）：ioc容器启动会调用方法创建对象放到ioc容器中。
    * 			以后每次获取就是直接从容器（map.get()）中拿，
    * request：同一次请求创建一个实例
    * session：同一个session创建一个实例
    
    * @Lazy懒加载：
    * 		单实例bean：默认在容器启动的时候创建对象；
    * 		懒加载：容器启动不创建对象。第一次使用(获取)Bean创建对象，并初始化；
    */
    * 	//	@Scope("prototype")
      	@Lazy
      	@Bean("person")
      	public Person person(){
      		System.out.println("给容器中添加Person....");
      		return new Person("张三", 25);
      	}
   
   	/**
   	 * @Conditional({Condition}) ： 按照一定的条件进行判断，满足条件给容器中注册bean
   	 * 如果系统是windows，给容器中注册("bill")
   	 * 如果是linux系统，给容器中注册("linus")
   	 */
   	@Conditional(LinuxCondition.class)
   	@Bean("linus")
   	public Person person02(){
   		return new Person("linus", 48);
   	}
   ```

   ```
   	/**
   	 * ConditionContext：判断条件能使用的上下文（环境）
   	 * AnnotatedTypeMetadata：注释信息
   	 */
   	@Override
   	public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
   		// TODO是否linux系统
   		//1、能获取到ioc使用的beanfactory
   		ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
   		//2、获取类加载器
   		ClassLoader classLoader = context.getClassLoader();
   		//3、获取当前环境信息
   		Environment environment = context.getEnvironment();
   		//4、获取到bean定义的注册类
   		BeanDefinitionRegistry registry = context.getRegistry();
   		
   		String property = environment.getProperty("os.name");
   		
   		//可以判断容器中的bean注册情况，也可以给容器中注册bean
   		boolean definition = registry.containsBeanDefinition("person");
   		if(property.contains("linux")){
   			return true;
   		}
   		return false;
   	}
   ```

   
   
* @Conditional（重要）：按照条件给容器中注册bean
* @Import（重要）

```java
//类中组件统一设置。满足当前条件，这个类中配置的所有bean注册才能生效；
@Conditional({WindowsCondition.class})
@Configuration
@Import({Color.class,Red.class,MyImportSelector.class,MyImportBeanDefinitionRegistrar.class})
//1. @Import导入组件，id默认是组件的全类名
public class MainConfig2 {...}

//判断是否windows系统
public class WindowsCondition implements Condition {
	@Override
	public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
		Environment environment = context.getEnvironment();
		String property = environment.getProperty("os.name");
		if(property.contains("Windows")){
			return true;
		}
		return false;
	}
}
/**
	 * 给容器中注册组件；
	 * 1）、包扫描+组件标注注解（@Controller/@Service/@Repository/@Component）[自己写的类]
	 * 2）、@Bean[导入的第三方包里面的组件]
	 * 3）、@Import[快速给容器中导入一个组件]
	 * 		1）、@Import(要导入到容器中的组件)；容器中就会自动注册这个组件，id默认是全类名
	 * 		2）、ImportSelector:(SpringBoot频繁使用)返回需要导入的组件的全类名数组；（需要实现ImportSelector）
	 * 		3）、ImportBeanDefinitionRegistrar:手动注册bean到容器中
	 * 4）、使用Spring提供的 FactoryBean（工厂Bean）（整合框架时使用频繁）;
	 * 		1）、默认获取到的是工厂bean调用getObject创建的对象
	 * 		2）、要获取工厂Bean本身，我们需要给id前面加一个&
	 * 			&colorFactoryBean
	 */
// 2.ImportSelector自定义逻辑返回需要导入的组件
public class MyImportSelector implements ImportSelector {

	//返回值，就是到导入到容器中的组件全类名
	//AnnotationMetadata:当前标注了@Import注解的类的所有注解信息
	@Override
	public String[] selectImports(AnnotationMetadata importingClassMetadata) {
		// TODO Auto-generated method stub
		//importingClassMetadata
		//方法不要返回null值
		return new String[]{"com.atguigu.bean.Blue","com.atguigu.bean.Yellow"};
	}
}
// 3.ImportBeanDefinitionRegistrar
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
	/**
	 * AnnotationMetadata：当前类的注解信息
	 * BeanDefinitionRegistry:BeanDefinition注册类；
	 * 		把所有需要添加到容器中的bean；调用
	 * 		BeanDefinitionRegistry.registerBeanDefinition手工注册进来
	 */
	@Override
	public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
		boolean definition = registry.containsBeanDefinition("com.atguigu.bean.Red");
		boolean definition2 = registry.containsBeanDefinition("com.atguigu.bean.Blue");
		if(definition && definition2){
			//指定Bean定义信息；（Bean的类型，Bean。。。）
			RootBeanDefinition beanDefinition = new RootBeanDefinition(RainBow.class);
			//注册一个Bean，指定bean名
			registry.registerBeanDefinition("rainBow", beanDefinition);
		}
	}
}
//4）创建一个Spring定义的FactoryBean
public class ColorFactoryBean implements FactoryBean<Color> {
	//返回一个Color对象，这个对象会添加到容器中
	@Override
	public Color getObject() throws Exception {
		System.out.println("ColorFactoryBean...getObject...");
		return new Color();
	}
	@Override
	public Class<?> getObjectType() {
		return Color.class;
	}
	//是单例？
	//true：这个bean是单实例，在容器中保存一份
	//false：多实例，每次获取都会创建一个新的bean；
	@Override
	public boolean isSingleton() {
		// TODO Auto-generated method stub
		return false;
	}
}


```



#### 生命周期

```java
 /* 1）、指定初始化和销毁方法；（执行顺序：无参构造、容器刚创建init、容器关闭destory
 * 		通过@Bean指定init-method和destroy-method；
 * 构造（对象创建）
 * 		单实例：在容器启动的时候创建对象
 * 		多实例：在每次获取的时候创建对象\
  * BeanPostProcessor.postProcessBeforeInitialization
  * 初始化：
 * 		对象创建完成，并赋值好，调用初始化方法。。。
 * BeanPostProcessor.postProcessAfterInitialization
 * 销毁：
 * 		单实例：容器关闭的时候
 * 		多实例：容器不会管理这个bean；容器不会调用销毁方法；
 
 * 2）、通过让Bean实现InitializingBean（定义初始化逻辑），
 * 				DisposableBean（定义销毁逻辑）;
 * 3）、可以使用JSR250；
 * 		@PostConstruct：在bean创建完成并且属性赋值完成；来执行初始化方法
 * 		@PreDestroy：在容器销毁bean之前通知我们进行清理工作
 * 4）、BeanPostProcessor【interface】：bean的后置处理器；（重要）
 * 		在bean初始化前后进行一些处理工作；
 * 		postProcessBeforeInitialization:在初始化之前工作
 * 		postProcessAfterInitialization:在初始化之后工作
 
 * BeanPostProcessor原理（生命周期注解执行的底层原理）
  遍历得到容器中所有的BeanPostProcessor；挨个执行beforeInitialization，
 * 一但返回null，跳出for循环，不会执行后面的BeanPostProcessor.postProcessorsBeforeInitialization
 * populateBean(beanName, mbd, instanceWrapper);给bean进行属性赋值
 * initializeBean
 * {
 * applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
 * invokeInitMethods(beanName, wrappedBean, mbd);执行自定义初始化
 * applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
 *}
  * Spring底层对 BeanPostProcessor 的使用：
  		bean赋值，注入其他组件，@Autowired，生命周期注解功能，@Async,等等都是以靠BeanPostProcessor完成的
 * 	
 * */
 
 // 1）
@ComponentScan("com.atguigu.bean")
@Configuration
public class MainConfigOfLifeCycle {
	//@Scope("prototype")
	@Bean(initMethod="init",destroyMethod="detory")
	public Car car(){
		return new Car();
	}
}
@Component
public class Car {
	public Car(){
		System.out.println("car constructor...");
	}
	public void init(){
		System.out.println("car ... init...");
	}
	public void detory(){
		System.out.println("car ... detory...");
	}
}

 // 2)
@Component
public class Cat implements InitializingBean,DisposableBean {
	public Cat(){
		System.out.println("cat constructor...");
	}
	@Override
	public void destroy() throws Exception {
		System.out.println("cat...destroy...");
	}
	@Override
	public void afterPropertiesSet() throws Exception {
		System.out.println("cat...afterPropertiesSet...");
	}
}
// 3)
@Component
public class Dog implements ApplicationContextAware {
	//@Autowired
	private ApplicationContext applicationContext;
	public Dog(){
		System.out.println("dog constructor...");
	}
	//对象创建并赋值之后调用
	@PostConstruct
	public void init(){
		System.out.println("Dog....@PostConstruct...");
	}
	//容器移除对象之前
	@PreDestroy
	public void detory(){
		System.out.println("Dog....@PreDestroy...");
	}
	@Override     // 自动执行设置容器
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		// TODO Auto-generated method stub
		this.applicationContext = applicationContext;
	}
}
// 4)
 @Component
public class MyBeanPostProcessor implements BeanPostProcessor {
	@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		System.out.println("postProcessBeforeInitialization..."+beanName+"=>"+bean);
		return bean;
	}
	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		System.out.println("postProcessAfterInitialization..."+beanName+"=>"+bean);
		return bean;
	}
}
 
 
 
 
 
```

* 属性赋值

```java
//使用@PropertySource读取外部配置文件中的k/v保存到运行的环境变量中;加载完外部的配置文件以后使用${}取出配置文件的值
@PropertySource(value={"classpath:/person.properties"})
// 需要再person.java的bean文件中使用@Value
@Value("${person.nickName}")
private String nickName;
// 也可以在运行环境变量中获取
ConfigurableEnvironment environment = applicationContext.getEnvironment();
String property = environment.getProperty("person.nickName");
```

#### 自动装配

```java
/**
 * 自动装配;
 * 		Spring利用依赖注入（DI），完成对IOC容器中中各个组件的依赖关系赋值；
 * 
 * 1）、@Autowired：自动注入：
 * 		1）、默认优先按照类型去容器中找对应的组件:applicationContext.getBean(BookDao.class);找到就赋值
 * 		2）、如果找到多个相同类型的组件，再将属性的名称作为组件的id去容器中查找
 * 							applicationContext.getBean("bookDao")
 * 		3）、@Qualifier("bookDao")：使用@Qualifier指定需要装配的组件的id，而不是使用属性名
 * 		4）、自动装配默认一定要将属性赋值好，没有就会报错；
 * 			可以使用@Autowired(required=false);
 * 		5）、@Primary：让Spring进行自动装配的时候，默认使用首选的bean；
 * 				也可以继续使用@Qualifier指定需要装配的bean的名字
 * 		BookService{
 * 			@Autowired
 * 			BookDao  bookDao;
 * 		}
 * 2）、Spring还支持使用@Resource(JSR250)和@Inject(JSR330)[java规范的注解]
 * 		@Resource:
 * 			可以和@Autowired一样实现自动装配功能；默认是按照组件名称进行装配的；
 * 			没有能支持@Primary功能没有支持@Autowired（reqiured=false）;
 * 		@Inject:
 * 			需要导入javax.inject的包，和Autowired的功能一样。没有required=false的功能；
 *  @Autowired:Spring定义的； @Resource、@Inject都是java规范
 * 	
 * AutowiredAnnotationBeanPostProcessor:解析完成自动装配功能；		
 * 
 * 3）、 @Autowired:构造器，参数，方法，属性；都是从容器中获取参数组件的值
 * 		1）、[标注在方法位置]：@Bean+方法参数；参数从容器中获取;默认不写@Autowired效果是一样的；都能自动装配
 * 		2）、[标在构造器上]：如果组件只有一个有参构造器，这个有参构造器的@Autowired可以省略，参数位置的组件还是可以自动从容器中获取
 * 		3）、放在参数位置：
 * 
 * 4）、自定义组件想要使用Spring容器底层的一些组件（ApplicationContext，BeanFactory，xxx）；
 * 		自定义组件实现xxxAware；在创建对象的时候，会调用接口规定的方法注入相关组件；Aware；
 * 		把Spring底层一些组件注入到自定义的Bean中；
 * 		xxxAware功能依靠xxxProcessor来处理；
 * 			ApplicationContextAware==》ApplicationContextAwareProcessor=》BeanPostProcessor； 
 
 // 自动创建该bean对象到ioc中，该对象需要用到的其他组件(看实现了哪些xxxAware)就会以这种接口方法回调的方式传入进来
@Component
public class Red implements ApplicationContextAware,BeanNameAware,EmbeddedValueResolverAware {
	private ApplicationContext applicationContext;
	@Override  // ApplicationContextAware=>传入ioc容器
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		System.out.println("传入的ioc："+application   ontext);
		this.applicationContext = applicationContext;
	}
	@Override // BeanNameAware=>传入bean名
	public void setBeanName(String name) {
		System.out.println("当前bean的名字："+name);
	}
	@Override  // EmbeddedValueResolverAware=>解析占位符，返回解析后的值
	public void setEmbeddedValueResolver(StringValueResolver resolver) {
		String resolveStringValue = resolver.resolveStringValue("你好 ${os.name} 我是 #{20*18}");
		System.out.println("解析的字符串："+resolveStringValue);
	}
}
 */
```

* Profile

  ```java
  /**
   * Profile：
   * 		Spring为我们提供的可以根据当前环境，动态的激活和切换一系列组件的功能；
   * 开发环境、测试环境、生产环境；
   * 数据源：(/A)(/B)(/C)；
   * @Profile：指定组件在哪个环境的情况下才能被注册到容器中，不指定，任何环境下都能注册这个组件
   * 1）、加了环境标识的bean，只有这个环境被激活的时候才能注册到容器中。默认是default环境
   * 2）、写在配置类上，只有是指定的环境的时候，整个配置类里面的所有配置才能开始生效
   * 3）、没有标注环境标识的bean在，任何环境下都是加载的；
   */
  @PropertySource("classpath:/dbconfig.properties")
  @Configuration
  public class MainConfigOfProfile implements EmbeddedValueResolverAware{
  		@Profile("test")
  	@Bean("testDataSource")
  	public DataSource dataSourceTest(@Value("${db.password}")String pwd) throws Exception{
  		ComboPooledDataSource dataSource = new ComboPooledDataSource();
  		dataSource.setUser(user);
  		dataSource.setPassword(pwd);
  		dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
  		dataSource.setDriverClass(driverClass);
  		return dataSource;
  	}
  }
  
  	//1、使用命令行动态参数: 在虚拟机参数位置加载 -Dspring.profiles.active=test
  	//2、代码的方式激活某种环境；
  	@Test
  	public void test01(){
          //1、创建一个applicationContext
  		AnnotationConfigApplicationContext applicationContext = 
  				new AnnotationConfigApplicationContext();
  		//2、设置需要激活的环境
  		applicationContext.getEnvironment().setActiveProfiles("test");
  		//3、注册主配置类
  		applicationContext.register(MainConfigOfProfile.class);
  		//4、启动刷新容器
  		applicationContext.refresh();
  		String[] namesForType = applicationContext.getBeanNamesForType(DataSource.class);
  		for (String string : namesForType) {
  			System.out.println(string);
  		}
  		Yellow bean = applicationContext.getBean(Yellow.class);
  		System.out.println(bean);
  		applicationContext.close();
  	}
  ```

  











## 二 扩展原理







## 三 WEB















































