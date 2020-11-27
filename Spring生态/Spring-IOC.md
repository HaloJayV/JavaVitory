[TOC]

# Spring（IOC + AOP）

### IOC：

* 基于XML配置

  * ioc容器一旦启动，除了多实例bean对象propotype之外的所有bean对象都会被创建

  * `ApplicationContext ioc = new ClassPathXmlApplicationContext("ioc.xml"); // 从xml获取容器`   `ioc.getBean(xxx); // 获取bean对象`

  * 如果ioc容器中按类型查找的bean有多个会报错，只能通过对象id或者通过id+类型查找都可行

  * ```
    在ioc创建有参构造器：<bean id="bean对象id" class="全类名"> <constructor-arg name="参数名" value="值"></constructor-arg> </bean>
    ```

  * 通过p名称空间为bean赋值，加前缀防止标签重复： `<bean id="bean对象id" class="全类名" p:age="15" p:name="a">`   调用的是set方法

  * ref引用外部bean的值: `<bean> <property name="外部bean的属性名" ref="外部bean的id"> </bean>` ，也可以直接在property标签内引入外部class

  * 实例工厂的使用：`<bean id="" class="" factory-bean="工厂实例对象" factory-method="工厂方法"></bean>`

  * bean作用域：`<bean id="" class="" scope="xxx">` xxx：prototype/singleton/request/session分别是多实例(ioc启动默认不会创建)，单实例，同一次请求和会话

  * 组件扫描：

    ```
    <context:component-scan base-package="至少是二级类名" use-default-filters="true/false">
    	<!-- type：annotion/assignable/aspectj/custom/regex   分别是按注解、类、aspectj表达式、自定义TypeFilter、正则表达式排除-->
    	<context:exclude-filter type="annotion/assignable" expression="要排除的组件全类名">
    	<context:include-filter type="annotion/assignable" expression="要扫描的组件全类名,use-default-filters=false才生效">
    </context:component-scan>
    ```

* Spring注解

  * `@Scope(value="prototype/singleton/request/session")` 可以修改作用域
  * @Qualifier("xxx"): 指定一个名作为id，让spriing别使用变量名作为id而用自定义id找
  * @Autowired(required=false)  找不到bean不会报错，自动装配null，默认required=true
  * `@ContextConfiguration(locations="classpath:xxx")`  用来指定Spring的配置文件的位置，即导入配置文件
  * `<context:property-placeholder localtation="classpath:dbconfig.properties">` 引入外部配置文件
  * @RunWith(SpringJUnit4ClassRunner)  // 使用特定的单元测试模块

* @Autowired（自动装配）：底层是ioc.getBean(xxx) 

  * 1.先按照**注解类型**去容器中找到对应的组件：ioc.getBean(BookService);   如果找到就赋值，没找到抛异常；
  * 2.如果找到多个：按照**变量名**作bookService为id 继续匹配，匹配上就装配，没有匹配就报错，可以使用@Qualifier("bookServiceExt")指定新id 

  * 3.@Autowired 标注的自动装配属性默认是一定装配上的，找到就装配，找不到拉倒：@Autowired(required=false)

  * 和@Resource，@Inject的区别：都能自动装配，@Autowired最强大但依赖Spring，Spring自己的注解，@Resource是Javax，即J2ee的标准，扩展性强

  * @Autowired底层有@Target，表示可以写在构造器上、属性字段上、方法上、注解上
  
  * 若子类加了注解成为组件，那么所继承的父类也会被自动注入到ioc中。泛型依赖注入会根据子类调用者的类型进行调用对应的对象和方法

* 容器底层是map，这个map中保存所有创建好的bean，并提供外界获取功能
  

# Spring源码分析：

## IOC源码解析

* ### ioc = new ClassPathXmlApplicationContext("ioc.xml");

* 断点1：进入 **ClassPathXmlApplicationContext.class**

* ```
  	private static ApplicationContext ioc;
   	public static void main(String[] args) {
   		ioc = new ClassPathXmlApplicationContext("ioc.xml"); // 打断点1：进入创建ioc的jdk源码
   		Object bean = ioc.getBean("person03");
   		System.out.println(bean);
   	}
  ```

  ![image-20200706161712300](../../../../Software/Typora/Picture/image-20200706161712300.png)

  * 断点2：

  ```java
  // ioc = new ClassPathXmlApplicationContext("ioc.xml"); // 调用的底层是：
  public ClassPathXmlApplicationContext(String configLocation) throws BeansException {
  	this(new String[] {configLocation}, true, null);  // 断点2：进入this，发现调用带有3个参数的构造器
  }
  ```
  
* 通过进入源码可以发现几乎所有构造方法都调用那个带有三个参数的构造器
  
* ```java
  public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent)
    			throws BeansException {
    		super(parent);
    		setConfigLocations(configLocations);
    		if (refresh) {  
    			refresh();   // 断点3：refresh为true，所有单实例bean对象被创建，进入断点3
    		}
    	}
  ```
  
  * 断点3：断点3进入 **AbstractApplicationContext.class**，   实现所有单实例bean对象的创建
  
  ```java
     	@Override   // 实现所有单实例bean对象的创建
    	public void refresh() throws BeansException, IllegalStateException {
    		synchronized (this.startupShutdownMonitor) {  // 锁，保证多线程情况下ioc容器只会被创建一次
    			// Prepare this context for refreshing.
    // 断点4：准备刷新、启动 ClassPathXmlApplicationContext 对象
    // 控制台打印信息: Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@edf4efb
    			prepareRefresh(); 
    
    			// Tell the subclass to refresh the internal bean factory.
    // 断点5：ConfigurableListableBeanFactory 和 ClassPathXmlApplicationContext 同属于BeanFactory下
  // 控制台打印信息: Loading XML bean definitions from class path resource [ioc.xml]
    			// 即从类路径下的 ioc.xml 配置文件中加载将要创建的所有bean配置信息保存在 beanFactory
    			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory(); 
    
    			// Prepare the bean factory for use in this context.
    			prepareBeanFactory(beanFactory);  // beanFactory的准备
    
    			try {
    				// Allows post-processing of the bean factory in context subclasses.
    				postProcessBeanFactory(beanFactory); // beanFactory 后置处理
    
    				// Invoke factory processors registered as beans in the context.
    				invokeBeanFactoryPostProcessors(beanFactory); // 执行beanFactory的后置处理器
    
    				// Register bean processors that intercept bean creation. 
    				registerBeanPostProcessors(beanFactory);  // 注册 beanFactory 内部组件的后置处理器
    
    				// Initialize message source for this context.
    				initMessageSource();  // 初始化消息源，用于支持国际化功能
    
    				// Initialize event multicaster for this context.
    				initApplicationEventMulticaster(); // 初始化事件转换器
    
    				// Initialize other special beans in specific context subclasses.
    				onRefresh();  // 里面是空方法，专门留给子类的方法
    
    				// Check for listener beans and register them.
    				registerListeners(); // 注册监听器
    
    				// Instantiate all remaining (non-lazy-init) singletons.
    // 断点6：完成BeanFactory 的初始化，完成所有非懒加载的、单实例Bean的创建
    				finishBeanFactoryInitialization(beanFactory); 
    
    				// Last step: publish corresponding event.
    				finishRefresh();
    			}
    			catch (BeansException ex) {
    				// Destroy already created singletons to avoid dangling resources.
    				destroyBeans();
    
    				// Reset 'active' flag.
    				cancelRefresh(ex);
    
    				// Propagate exception to caller.
    				throw ex;
    			}
    		}
    	}
  ```
  
    * 断点5：ConfigurableListableBeanFactory
  
    ![image-20200706164325723](../../../../Software/Typora/Picture/image-20200706164325723.png)
  
    和ClassPathXmlApplicationContext同属于BeanFactory（Bean工厂）下，是BeanFactory的子接口
  
    * 断点5：
  
    ```
    ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
    ```
  
  ![image-20200706165912900](../../../../Software/Typora/Picture/image-20200706165912900.png)
  
  * ##### 断点6：finishBeanFactoryInitialization完成了单例对象的创建
  
    * ![image-20200706171401065](../../../../Software/Typora/Picture/image-20200706171401065.png)
  
    ```java
    protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
      		// Initialize conversion service for this context. 初始化类型转换组件服务
    		if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
      				beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
    			beanFactory.setConversionService(
      					beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
      		}
      		
    		// 加载需要 @Autowire 的组件
      		// Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
    		String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
      		for (String weaverAwareName : weaverAwareNames) {
    			getBean(weaverAwareName);
      		}
    
      		// Stop using the temporary ClassLoader for type matching.
      		beanFactory.setTempClassLoader(null);  // 设置beanFactory 的类加载器
      		
      		// Allow for caching all bean definition metadata, not expecting further changes.
      		beanFactory.freezeConfiguration(); // 冻结beanFactory的配置
      
      		// Instantiate all remaining (non-lazy-init) singletons.
          // 断点7：预初始化所有非懒加载的单实例bean，进入断点
      		beanFactory.preInstantiateSingletons(); 
      	}
    ```
    
      * 断点7：进入到 **DefaultListableBeanFactory.class** 的 preInstantiateSingletons方法
    
        ```java
        	public void preInstantiateSingletons() throws BeansException { // 准备实例化单实例bean
        		if (this.logger.isDebugEnabled()) {
        			this.logger.debug("Pre-instantiating singletons in " + this);
        		}
        		
        // 断点8：按顺序拿到在ioc中的所有非懒加载的单实例bean名
        		List<String> beanNames;
        		synchronized (this.beanDefinitionMap) {
        			// Iterate over a copy to allow for init methods which in turn register new bean definitions.
        			// While this may not be part of the regular factory bootstrap, it does otherwise work fine.
        			beanNames = new ArrayList<String>(this.beanDefinitionNames); 
        		}
        // 断点9：按顺序创建bean
        		for (String beanName : beanNames) {
        			// 根据BeanId获取到bean的定义信息
        			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
        // 断点10：只允许 非抽象、单例的、非懒加载的 bean 创建
        			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
        // 断点11：判断是否是通过BeanFactory接口实现的、创建的Bean
        				if (isFactoryBean(beanName)) { 
        					final FactoryBean<?> factory = (FactoryBean<?>) getBean(FACTORY_BEAN_PREFIX + beanName);
        					boolean isEagerInit;
        					if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
        						isEagerInit = AccessController.doPrivileged(new PrivilegedAction<Boolean>() {
        							@Override
        							public Boolean run() {
        								return ((SmartFactoryBean<?>) factory).isEagerInit();
        							}
        						}, getAccessControlContext());
        					}
        					else {
        						isEagerInit = (factory instanceof SmartFactoryBean &&
        								((SmartFactoryBean<?>) factory).isEagerInit());
        					}
        					if (isEagerInit) {
        						getBean(beanName);
        					}
        				}
        				else {
         // 断点12：bean创建的具体实现，进入断点12, 进入了AbstractBeanFactory.class
        					getBean(beanName);			
        				}
        			}
        		}
        	}
        ```
    
      * 断点8：
    
        * ![image-20200706174022240](../../../../Software/Typora/Picture/image-20200706174022240.png)
    
      * 进入断点12：进入了AbstractBeanFactory.class 的 getBean方法
    
        ```java
        	// 该getBean方法在 ioc.getBean("xxx"); 时会被调用
        	public Object getBean(String name) throws BeansException {
        	// 断点13：进入断点
        		return doGetBean(name, null, null, false); 
        	}
        ```
    
      * ##### 进入断点13：doGetBean方法 
  
    ```java
    	@SuppressWarnings("unchecked")
      	protected <T> T doGetBean(
    			final String name, final Class<T> requiredType, final Object[] args, boolean typeCheckOnly)
      			throws BeansException {
    
      		final String beanName = transformedBeanName(name);
      		Object bean;
      
      		// Eagerly check singleton cache for manually registered singletons.
      // 断点14：根据bean名，从缓存中的已注册单实例bean里获取单实例，第一次为null
      		Object sharedInstance = getSingleton(beanName); 
      		if (sharedInstance != null && args == null) { // 判断缓存中有没有已注册单实例bean，第一次创建bean没有
    			if (logger.isDebugEnabled()) {
      				if (isSingletonCurrentlyInCreation(beanName)) {
    					logger.debug("Returning eagerly cached instance of singleton bean '" + beanName +
      							"' that is not fully initialized yet - a consequence of a circular reference");
      				}
      				else {
      					logger.debug("Returning cached instance of singleton bean '" + beanName + "'");
      				}
      			}
      			bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
      		}
      // 断点15：第一次创建bean
      		else { 
      			// Fail if we're already creating this bean instance:
      			// We're assumably within a circular reference.
      			if (isPrototypeCurrentlyInCreation(beanName)) {
      				throw new BeanCurrentlyInCreationException(beanName);
      			}
      
      			// Check if bean definition exists in this factory.
      			BeanFactory parentBeanFactory = getParentBeanFactory();
      			if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
      				// Not found -> check parent.
      				String nameToLookup = originalBeanName(name);
      				if (args != null) {
      					// Delegation to parent with explicit args.
      					return (T) parentBeanFactory.getBean(nameToLookup, args);
      				}
      				else {
      					// No args -> delegate to standard getBean method.
      					return parentBeanFactory.getBean(nameToLookup, requiredType);
      				}
      			}
      
      			if (!typeCheckOnly) { // 传过来的参数，默认为false
      				markBeanAsCreated(beanName); // 标记该bean已经被创建
      			}
      
      			try {
      			// 根据bean名获取本地合并的bean定义信息
      				final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
      				checkMergedBeanDefinition(mbd, beanName, args);
      
      				// Guarantee initialization of beans that the current bean depends on.
      // 断点16： 获取当前bean所依赖的bean、depends-on属性
      				String[] dependsOn = mbd.getDependsOn(); 
      				if (dependsOn != null) {
      					for (String dependsOnBean : dependsOn) { // 创建当前bean前需要先创建依赖的所有bean
      						if (isDependent(beanName, dependsOnBean)) {
      							throw new BeanCreationException("Circular depends-on relationship between '" +
      									beanName + "' and '" + dependsOnBean + "'");
      						}
      						registerDependentBean(dependsOnBean, beanName);
      						getBean(dependsOnBean); // 循环创建依赖bean
      					}
      				}
      				
      // 断点17 开始创建bean实例
      				// Create bean instance.
      				if (mbd.isSingleton()) {  
      // 断点18： 创建单实例bean，进入 DefaultSingletonBeanRegistry 的 getSingleton方法
      					sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() {
      						@Override
      						public Object getObject() throws BeansException {
      							try {
      								return createBean(beanName, mbd, args); 
      							}
      							catch (BeansException ex) {
      								// Explicitly remove instance from singleton cache: It might have been put there
      								// eagerly by the creation process, to allow for circular reference resolution.
      								// Also remove any beans that received a temporary reference to the bean.
      								destroySingleton(beanName);
      								throw ex;
      							}
      						}
      					});
      					bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
      				}
      
      				else if (mbd.isPrototype()) {
      					// It's a prototype -> create a new instance.
      					Object prototypeInstance = null;
      					try {
      						beforePrototypeCreation(beanName);
      						prototypeInstance = createBean(beanName, mbd, args);
      					}
      					finally {
      						afterPrototypeCreation(beanName);
      					}
      					bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
      				}
      
      				else {
      					String scopeName = mbd.getScope();
      					final Scope scope = this.scopes.get(scopeName);
      					if (scope == null) {
      						throw new IllegalStateException("No Scope registered for scope '" + scopeName + "'");
      					}
      					try {
      						Object scopedInstance = scope.get(beanName, new ObjectFactory<Object>() {
      							@Override
      							public Object getObject() throws BeansException {
      								beforePrototypeCreation(beanName);
      								try {
      									return createBean(beanName, mbd, args);
      								}
      								finally {
      									afterPrototypeCreation(beanName);
      								}
      							}
      						});
      						bean = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
      					}
      					catch (IllegalStateException ex) {
      						throw new BeanCreationException(beanName,
      								"Scope '" + scopeName + "' is not active for the current thread; " +
      								"consider defining a scoped proxy for this bean if you intend to refer to it from a singleton", ex);
      					}
      				}
      			}
      			catch (BeansException ex) {
      				cleanupAfterBeanCreationFailure(beanName);
      				throw ex;
      			}
      		}
      
      		// Check if required type matches the type of the actual bean instance.
      		if (requiredType != null && bean != null && !requiredType.isAssignableFrom(bean.getClass())) {
      			try {
      				return getTypeConverter().convertIfNecessary(bean, requiredType);
      			}
      			catch (TypeMismatchException ex) {
      				if (logger.isDebugEnabled()) {
      					logger.debug("Failed to convert bean '" + name + "' to required type [" +
      							ClassUtils.getQualifiedNam*e(requiredType) + "]", ex);
      				}
      				throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
      			}
      		}
      		return (T) bean;
      	}
    ```
    
    * 断点18： DefaultSingletonBeanRegistry 的 getSingleton 方法
    
      ```java
      /** Cache of singleton objects: bean name --> bean instance */
      // 按照bean名缓存相应的单实例对象
      private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(64);
      	
      	public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
      		Assert.notNull(beanName, "'beanName' must not be null");
      		synchronized (this.singletonObjects) {
      		// 先从一个地方将这个bean Get出来
      			Object singletonObject = this.singletonObjects.get(beanName); // 按照bean名缓存相应的单实例对象
      			if (singletonObject == null) {
      				if (this.singletonsCurrentlyInDestruction) {
      					throw new BeanCreationNotAllowedException(beanName,
    							"Singleton bean creation not allowed while the singletons of this factory are in destruction " +
      							"(Do not request a bean from a BeanFactory in a destroy method implementation!)");
    				}
      				if (logger.isDebugEnabled()) {
      					logger.debug("Creating shared instance of singleton bean '" + beanName + "'");
      				}
      				beforeSingletonCreation(beanName);
      				boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
      				if (recordSuppressedExceptions) {
      					this.suppressedExceptions = new LinkedHashSet<Exception>();
      				}
      				try {
      // 断点19： 利用反射创建单实例bean，此刻创建好了单实例bean
      					singletonObject = singletonFactory.getObject(); 
      				}
      				catch (BeanCreationException ex) {
      					if (recordSuppressedExceptions) {
      						for (Exception suppressedException : this.suppressedExceptions) {
      							ex.addRelatedCause(suppressedException);
      						}
      					}
      					throw ex;
      				}
      				finally {
      					if (recordSuppressedExceptions) {
      						this.suppressedExceptions = null;
      					}
      					afterSingletonCreation(beanName);
      				}
      // 断点20：添加保存刚创建好的单实例bean，进入断点
      				addSingleton(beanName, singletonObject);
      			}
      			return (singletonObject != NULL_OBJECT ? singletonObject : null);
      		}
      	}
      ```
    
    * 断点20：进入DefaultSingletonBeanRegistry 的 addSingleton方法
    
      ```java
      	protected void addSingleton(String beanName, Object singletonObject) {
      		synchronized (this.singletonObjects) {
      		 // 创建好的单实例bean最终保存在该map：DefaultSingletonBeanRegistry 的 singletonObjects中
      		 // singletonObjects只是ioc容器之一，保存单实例bean的地方
      		 // ioc是一个容器，单实例bean保存在一个 concurrentHashMap 中
      			this.singletonObjects.put(beanName, (singletonObject != null ? singletonObject : NULL_OBJECT));
      			this.singletonFactories.remove(beanName);
      			this.earlySingletonObjects.remove(beanName);
      			this.registeredSingletons.add(beanName);
      		}
      	}
      ```

##### 创建bean总结：

底层调用refresh方法开始实现bean单例对象的创建，refresh里完成beanFactory的注册与加载，以及各种后置处理器、监听器的注册等等操作，最后refresh下的finishBeanFactoryInitialization方法最终完成了bean单例对象的创建，但具体的创建是getBean下的doGetBean方法完成

  ### getBean获取bean;

  * 进入ioc.getBean 断点：AbstractBeanFactory.getBean()

    * ```java
      	@Override
       	public Object getBean(String name) throws BeansException {
       		assertBeanFactoryActive();
       		return getBeanFactory().getBean(name); // 进入断点1：getBeanFactory().getBean(name)
       	}
      ```

    * 进入断点1：AbstractBeanFactory 的 getBeanFactory().getBean(name)：

      ```java
      	@Override
       	public Object getBean(String name) throws BeansException {
       		return doGetBean(name, null, null, false); // 进入断点2
       	}
      ```

    * 进入断点2：进入到AbstractBeanFactory的 doGetBean

      ```java
@SuppressWarnings("unchecked")
      protected <T> T doGetBean(
      		final String name, final Class<T> requiredType, final Object[] args, boolean typeCheckOnly)
      		throws BeansException {
              final String beanName = transformedBeanName(name);
              Object bean;
              // Eagerly check singleton cache for manually registered singletons.
              Object sharedInstance = getSingleton(beanName);  // 断点3
      }
      ```
      
    * 进入断点3：进入到 DefaultSingletonBeanRegistry.getSingleton(String, boolean)

    ```java
	protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    		Object singletonObject = this.singletonObjects.get(beanName);   // 还是从 singletonObjects 这个map获取bean
    		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
    			synchronized (this.singletonObjects) {
    				singletonObject = this.earlySingletonObjects.get(beanName);
    				if (singletonObject == null && allowEarlyReference) {
    					ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
    					if (singletonFactory != null) {
    						singletonObject = singletonFactory.getObject();
    						this.earlySingletonObjects.put(beanName, singletonObject);
    						this.singletonFactories.remove(beanName);
    					}
    				}
    			}
    		}
    		return (singletonObject != NULL_OBJECT ? singletonObject : null);
    }
    ```


#### 总结：

##### 创建好的单实例bean最终保存在DefaultSingletonBeanRegistry 的 singletonObjects 这个**ConcurrentHashMap**中，ioc.getBean("")也是在这获取

* 1、Spring容器通过BeanDefinition对象来表示Bean, BeanDefinition描述了Bean的配置信息。而BeanDefinitionRegistry向容器注入，删除，获取BeanDefinition对象的方法，即BeanDefinitionRegistry可以用来管理BeanDefinition。

##### 　　1）：ConfigurationClassPostProcessor。

​    ConfigurationClassPostProcessor是一个BeanFactory和BeanDefinitionRegistry处理器，BeanDefinitionRegistry处理方法能处理@Configuration等注解。ConfigurationClassPostProcessor的postProcessBeanDefinitionRegistry()方法内部处理@Configuration，@Import，@ImportResource和类内部的@Bean。

​    ConfigurationClassPostProcessor类继承了BeanDefinitionRegistryPostProcessor继承了BeanFactoryPostProcessor。通过BeanDefinitionRegistryPostProcessor可以创建一个特别后置处理器来将BeanDefinition添加到BeanDefinitionRegistry中。它和BeanPostProcessor不同，BeanPostProcessor只是在Bean初始化的时候有个钩子让我们加入一些自定义操作；而BeanDefinitionRegistryPostProcessor可以让我们在BeanDefinition中添加一些自定义操作。在Mybatis与Spring的整合中，就利用到了BeanDefinitionRegistryPostProcessor来对Mapper的BeanDefinition进行了后置的自定义处理。

#####     2）AutowiredAnnotationBeanPostProcessor

AutowiredAnnotationBeanPostProcessor是一个BeanPostProcessor，注意BeanPostProcessor和BeanFactoryPostProcessor的区别。AutowiredAnnotationBeanPostProcessor是用来处理@Autowired注解和@Value注解的。

#####     3）：RequiredAnnotationBeanPostProcessor。这是用来处理@Required注解。

#####     4）：CommonAnnotationBeanPostProcessor

CommonAnnotationBeanPostProcessor提供对JSR-250规范注解的支持@javax.annotation.Resource、@javax.annotation.PostConstruct和@javax.annotation.PreDestroy等的支持。

#####     5）：PersistenceAnnotationBeanPostProcessor

##### EventListenerMethodProcessor提供@PersistenceContext的支持。

##### 	6）：EventListenerMethodProcessor

EventListenerMethodProcessor提供@ EventListener  的支持。@ EventListener实在spring4.2之后出现的，可以在一个Bean的方法上使用@EventListener注解来自动注册一个ApplicationListener。

* 2、**ResourcePatternResolver**是一个接口，继承了ResourceLoader，可以用来获取Resource 实例。ResourcePatternResolver中申明它默认的扫描文件 Pattern 为"**/*.class"，我们使用的AnnotationConfigApplicationContext或者说是任何的ApplicationContext都实现了这个接口，所以这里将AnnotationConfigApplicationContext传给了ResourcePatternResolver。

* 3、**ClassPathBeanDefinitionScanner**是一个扫描指定类路径中注解Bean定义的[扫描器](https://www.baidu.com/s?wd=扫描器&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)，在它初始化的时候，会初始化一些需要被扫描的注解，初始化用于加载包下的资源的Loader。



# 面试

### **什么是Spring框架？**

Spring是一种轻量级框架，旨在提高开发人员的开发效率以及系统的可维护性。

我们一般说的Spring框架就是Spring Framework，它是很多模块的集合，使用这些模块可以很方便地协助我们进行开发。这些模块是核心容器、数据访问/集成、Web、AOP（面向切面编程）、工具、消息和测试模块。比如Core Container中的Core组件是Spring所有组件的核心，Beans组件和Context组件是实现IOC和DI的基础，AOP组件用来实现面向切面编程。

Spring官网（https://spring.io/）列出的Spring的6个特征：

核心技术：依赖注入（DI），AOP，事件（Events），资源，i18n，验证，数据绑定，类型转换，SpEL。

测试：模拟对象，TestContext框架，Spring MVC测试，WebTestClient。

数据访问：事务，DAO支持，JDBC，ORM，编组XML。

Web支持：Spring MVC和Spring WebFlux Web框架。

集成：远程处理，JMS，JCA，JMX，电子邮件，任务，调度，缓存。

语言：Kotlin，Groovy，动态语言。

### **列举一些重要的Spring模块？**

下图对应的是Spring 4.x的版本，目前最新的5.x版本中Web模块的Portlet组件已经被废弃掉，同时增加了用于异步响应式处理的WebFlux组件。

![img](../../../../Software/Typora/Picture/842514-20190611160935478-279758854.png)

Spring Core：基础，可以说Spring其他所有的功能都依赖于该类库。主要提供IOC和DI功能。

Spring Aspects：该模块为与AspectJ的集成提供支持。

Spring AOP：提供面向方面的编程实现。

Spring JDBC：Java数据库连接。

Spring JMS：Java消息服务。

Spring ORM：用于支持Hibernate等ORM工具。

Spring Web：为创建Web应用程序提供支持。

Spring Test：提供了对JUnit和TestNG测试的支持。

### **谈谈自己对于Spring IOC和AOP的理解**

IOC

IOC（Inversion Of Controll，控制反转）是一种设计思想，就是将原本在程序中手动创建对象的控制权，交由给Spring框架来管理。IOC在其他语言中也有应用，并非Spring特有。IOC容器是Spring用来实现IOC的载体，IOC容器实际上就是一个Map(key, value)，Map中存放的是各种对象。

将对象之间的相互依赖关系交给IOC容器来管理，并由IOC容器完成对象的注入。这样可以很大程度上简化应用的开发，把应用从复杂的依赖关系中解放出来。IOC容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。在实际项目中一个Service类可能由几百甚至上千个类作为它的底层，假如我们需要实例化这个Service，可能要每次都搞清楚这个Service所有底层类的构造函数，这可能会把人逼疯。如果利用IOC的话，你只需要配置好，然后在需要的地方引用就行了，大大增加了项目的可维护性且降低了开发难度。

Spring时代我们一般通过XML文件来配置Bean，后来开发人员觉得用XML文件来配置不太好，于是Sprng Boot注解配置就慢慢开始流行起来。

![img](../../../../Software/Typora/Picture/842514-20190611212352385-156216982.png)

上图是Spring IOC的初始化过程，IOC的源码阅读：https://javadoop.com/post/spring-ioc。

AOP

AOP（Aspect-Oriented Programming，面向切面编程）能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如**事务处理、日志管理、权限控制**等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可扩展性和可维护性。

Spring AOP是基于动态代理的，如果要代理的对象实现了某个接口，那么Spring AOP就会使用JDK动态代理去创建代理对象；而对于没有实现接口的对象，就无法使用JDK动态代理，转而使用CGlib动态代理生成一个被代理对象的子类来作为代理。

![img](../../../../Software/Typora/Picture/842514-20190611212955176-1474324049.png)

当然也可以使用AspectJ，Spring AOP中已经集成了AspectJ，AspectJ应该算得上是Java生态系统中最完整的AOP框架了。使用AOP之后我们可以把一些通用功能抽象出来，在需要用到的地方直接使用即可，这样可以大大简化代码量。我们需要增加新功能也方便，提高了系统的扩展性。日志功能、事务管理和权限管理等场景都用到了AOP。

### **Spring AOP和AspectJ AOP有什么区别？**

Spring AOP是属于运行时增强，而AspectJ是编译时增强。Spring AOP基于代理（Proxying），而AspectJ基于字节码操作（Bytecode Manipulation）。

Spring AOP已经集成了AspectJ，AspectJ应该算得上是Java生态系统中最完整的AOP框架了。AspectJ相比于Spring AOP功能更加强大，但是Spring AOP相对来说更简单。

如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择AspectJ，它比SpringAOP快很多。

### **Spring中的bean的作用域有哪些？**

1.singleton：唯一bean单实例，Spring中的bean默认都是单例的。

2.prototype：每次请求都会创建一个新的bean实例。

3.request：每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效。

4.session：每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP session内有效。

5.global-session：全局session作用域，仅仅在基于Portlet的Web应用中才有意义，Spring5中已经没有了。Portlet是能够生成语义代码（例如HTML）片段的小型Java Web插件。它们基于Portlet容器，可以像Servlet一样处理HTTP请求。但是与Servlet不同，每个Portlet都有不同的会话。

### **Spring中的单例bean的线程安全问题了解吗？**

单例bean存在线程问题，主要是因为当多个线程操作同一个对象的时候，对这个对象的非静态成员变量的写操作会存在线程安全问题。

有两种常见的解决方案：

1.在bean对象中尽量避免定义可变的成员变量（不太现实）。

2.在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在ThreadLocal中（推荐的一种方式）。

### 容器启动关键的几个类

- AbstractApplicationContext
  ApplicationContext接口的抽象实现类,   能够自动检测并注册各种后置处理器(PostProcessor)和事件监听器(Listener),  以模板方法模式定义了一些容器的通用方法,  比如启动容器的真正方法refresh()就是在该类中定义的。
- AbstractRefreshableApplicationContext
  继承AbstractApplicationContext的抽象类。内部持有一个DefaultListableBeanFactory 的实例,使得继承AbstractRefreshableApplicationContext的Spring的应用容器内部默认有一个Spring的核心容器,那么Spring容器的一些核心功能就可以委托给内部的核心容器去完成。AbstractRefreshableApplicationContext在内部定义了创建,销毁以及刷新核心容器BeanFactory的方法。
- ClassPathXmlApplicationContext
  最常用的Spring的应用容器之一。在启动时会加载类路径下的xml文件作为容器的配置信息。

### **Spring中的bean生命周期？**

###### 首先说一下Servlet的生命周期：实例化，初始init，接收请求service，销毁destroy；

 Spring上下文中的Bean生命周期也类似，如下：

##### （1）实例化Bean：（工厂模式）

对于**BeanFactory**容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用**createBean**进行实例化。

对于**ApplicationContext**容器，当容器启动结束后，通过获取**BeanDefinition**对象中的信息，实例化所有的bean。

##### （2）设置对象属性（依赖注入）：（包装模式）

实例化后的对象被封装在**BeanWrapper**对象中，紧接着，Spring根据**BeanDefinition**中的信息 以及 通过**BeanWrapper**提供的设置属性的接口完成依赖注入。

##### （3）处理Aware接口：（模板模式）

接着，Spring会检测该对象是否实现了**xxxAware**接口，并将相关的xxxAware实例注入给Bean：

①如果这个Bean已经实现了**BeanNameAware**接口，会调用它实现的**setBeanName**(String beanId)方法，此处传递的就是Spring配置文件中Bean的id值；

②如果这个Bean已经实现了**BeanFactoryAware**接口，会调用它实现的**setBeanFactory**()方法，传递的是Spring工厂自身。

③如果这个Bean已经实现了**ApplicationContextAware**接口，会调用**setApplicationContext**(ApplicationContext)方法，传入Spring上下文；

##### （4）BeanPostProcessor：

如果想对Bean进行一些自定义的处理，那么可以让Bean实现了**BeanPostProcessor**接口，那将会调用**postProcessBeforeInitialization**(Object obj, String s)方法。

##### （5）InitializingBean 与 init-method：

如果Bean在Spring配置文件中配置了 **init-method 属性**，则会自动调用其配置的初始化方法。

（6）如果这个Bean实现了**BeanPostProcessor**接口，将会调用**postProcessAfterInitialization**(Object obj, String s)方法；由于这个方法是在Bean初始化结束时调用的，所以可以被应用于**内存或缓存技术**；

> 以上几个步骤完成后，Bean就已经被正确创建了，之后就可以使用这个Bean了。

##### （7）DisposableBean：

当Bean不再需要时，会经过清理阶段，如果Bean实现了**DisposableBean**这个接口，会调用其实现的**destroy**()方法；

##### （8）destroy-method：

最后，如果这个Bean的Spring配置中配置了**destroy-method**属性，会自动调用其配置的销毁方法。

![img](../../../../Software/Typora/Picture/842514-20190611220522364-1747090465.png)

![img](../../../../Software/Typora/Picture/842514-20190611220552139-1446928813.png)

### Spring循环依赖

https://zhuanlan.zhihu.com/p/84267654

一个完整的对象包含两部分：当前对象实例化和对象属性的实例化。

在Spring中，对象的实例化是通过反射实现的，而对象的属性则是在对象实例化之后通过一定的方式设置的。

```java
@Component
public class A {
  private B b;
  public void setB(B b) {
    this.b = b;
  }
}
@Component
public class B {
  private A a;
  public void setA(A a) {
    this.a = a;
  }
}
```

![image-20201114111520876](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201114111520876.png)

#### 解决：

- Spring是通过**递归**的方式获取目标bean及其所依赖的bean的；
- Spring实例化一个bean的时候，是分两步进行的，首先**实例化目标bean**，然后为其**注入属性**。

结合这两点，也就是说，Spring在实例化一个bean的时候，是首先**递归实例化**其所依赖的所有bean，直到某个bean没有依赖其他bean，此时就会将该实例返回，然后**反递归**的将获取到的bean设置为各个上层bean的属性的。

###### 构造器注入的不能循环依赖，对象都还在创建中，vm都还没返回给你实例，怎么注入？

spring不解决循环依赖，setter一样注入会失败。spring是有意解决循环依赖，使用了一个预创建池子存放半成品。而构造器注入解决不了是原理问题

##### 用字段 **setter** 注入的就可以循环依赖，因为都是先初始化了实例再填充字段值的。

构造注入无法解决循环依赖，而spring的三级缓存的存在使得setter注入不存在循环依赖。三级缓存的实现在代码中的 SingletonBeanRegistry 中

### 什么是三级缓存

##### 1，第一级缓存：单例缓存池 singletonObjects（concurrentHashMap）

##### 2，第二级缓存：早期提前暴露的对象缓存 earlySingletonObjects。

##### 3，第三级缓存：singletonFactories 单例对象工厂缓存

![image-20201127154218214](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201127154218214.png)

bean初始化是一个相当复杂的过程，假如**初始化A bean时，发现A bean依赖B bean**,即A初始化执行到了第2步，此时B还没有初始化，则需要暂停A，先去初始化B，那么此时new出来的A对象放哪里，直接放在容器Map里显然不合适，半残品怎么能用，所以需要提供一个可以**标记创建中bean(A)的Map，可以提前暴露正在创建的bean供其他bean依赖**，如果在初始化A所依赖的bean B时，发现B也需要注入一个A的依赖，则**B可以从创建中的beanMap中直接获取A对象（创建中）注入A**，然后完成B的初始化，返回给正在注入属性的A，最终A也完成初始化，皆大欢喜。

如果配置不允许循环依赖，则上述缓存就用不到了，A 依赖B，就是创建B，B依赖C就去创建C，创建完了逐级返回就行，所以，**一级缓存之后的其他缓存(二三级缓存)就是为了解决循环依赖**！而配置支持循环依赖后，就一定要解决循环依赖吗？肯定不是！循环依赖在实际应用中也有，但不会太多，简单的应用场景是： controller注入service，service注入mapper，只有复杂的业务，可能service互相引用，有可能出现循环依赖，所以为了**出现循环依赖才去解决，不出现就不解决，虽然支持循环依赖，但是只有在出现循环依赖时才真正暴露早期对象，否则只暴露个获取bean的方法，并没有真正暴露bean，因为这个方法不会被执行到，这块的实现就是三级缓存（singletonFactories），只缓存了一个单例bean工厂**。

这个bean工厂不仅可以暴露早期bean还可以暴露代理bean，如果存在aop代理，则依赖的应该是代理对象，而不是原始的bean。而暴露原始bean是在单例bean初始化的第2步，填充属性第3步，生成代理对象第4步，这就矛盾了，A依赖到B并去解决B依赖时，要去初始化B，然后B又回来依赖A，而此时A还没有执行代理的过程，所以，需要在填充属性前就生成A的代理并暴露出去，第2步时机就刚刚好。

三级缓存的bean工厂getObject方式，实际执行的是getEarlyBeanReference，如果对象需要被代理(存在beanPostProcessors -> SmartInstantiationAwareBeanPostProcessor)，则提前生成代理对象。

### **说说自己对于Spring MVC的了解？**

谈到这个问题，我们不得不提提之前Model1和Model2这两个没有Spring MVC的时代。

Model1时代：很多学Java比较晚的后端程序员可能并没有接触过Model1模式下的JavaWeb应用开发。在Model1模式下，整个Web应用几乎全部用JSP页面组成，只用少量的JavaBean来处理数据库连接，访问等操作。这个模式下JSP即是控制层又是表现层。显而易见，这种模式存在很多问题。比如将控制逻辑和表现逻辑混杂在一起，导致代码重用率极低；又比如前端和后端相互依赖，难以进行测试并且开发效率极低。

Model2时代：学过Servlet并做过相关Demo的朋友应该了解Java Bean（Model）+JSP（View）+Servlet（Controller）这种开发模式，这就是早期的Java Web MVC开发模式。Model是系统中涉及的数据，也就是dao和bean；View是用来展示模型中的数据，只是用来展示；Controller是将用户请求都发送给Servlet做处理，返回数据给JSP并展示给用户。

Model2模式下还存在很多问题，Model2的抽象和封装程度还远远不够，使用Model2进行开发时不可避免地会重复造轮子，这就大大降低了程序的可维护性和可复用性。于是很多Java Web开发相关的MVC框架应运而生，比如Struts2，但是由于Struts2比较笨重，随着Spring轻量级开发框架的流行，Spring生态圈出现了Spring MVC框架。Spring MVC是当前最优秀的MVC框架，相比于Struts2，Spring MVC使用更加简单和方便，开发效率更高，并且Spring MVC运行速度更快。

###### MVC是一种设计模式，Spring MVC是一款很优秀的MVC框架。Spring MVC可以帮助我们进行更简洁的Web层的开发，并且它天生与Spring框架集成。Spring MVC下我们一般把后端项目分为Service层（处理业务）、Dao层（数据库操作）、Entity层（实体类）、Controller层（控制层，返回数据给前台页面）。

Spring MVC的简单原理图如下：

![img](../../../../Software/Typora/Picture/842514-20190611223228942-985610072.png)

### **Spring MVC的工作原理了解嘛？**

![img](../../../../Software/Typora/Picture/842514-20190612121947431-1483101551.png)

流程说明：

1.客户端（浏览器）发送请求，直接请求到**DispatcherServlet**。

2.DispatcherServlet根据请求信息调用**HandlerMapping**，解析请求对应的Handler。

3.解析到对应的**Handler**（也就是我们平常说的Controller控制器）。

4.**HandlerAdapter**会根据Handler来调用真正的处理器(**service**)来处理请求和执行相对应的业务逻辑。

5.处理器处理完业务后，会返回一个**ModelAndView**对象，Model是返回的数据对象，View是逻辑上的View。

6.**ViewResolver**会根据逻辑View去查找实际的View。

7.DispatcherServlet把返回的Model传给View（**视图渲染**）。

8.把View返回给请求者（浏览器）。

### **Spring框架中用到了哪些设计模式**

1.**工厂**模式：Spring使用工厂模式通过BeanFactory和ApplicationContext创建bean对象。

2.**代理**模式：Spring AOP功能的实现。

3.**单例**模式：Spring中的bean默认都是单例的。

4.**模板**方法模式：Spring中的jdbcTemplate、hibernateTemplate等以Template结尾的对数据库操作的类，它们就使用到了模板模式。

5.**包装器**设计模式：我们的项目需要**连接多个数据库**，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的**数据源**。

6.**观察者**模式：Spring**事件驱动模型**就是观察者模式很经典的一个应用。

7.**适配器**模式：**Spring AOP**的增强或通知（**Advice**）使用到了适配器模式、**Spring MVC**中也是用到了适配器模式适配Controller。

### **@Component和@Bean的区别是什么**

1.作用对象不同。@Component注解作用于类，而@Bean注解作用于**方法**。

2.@Component注解通常是通过**类路径扫描**来自动侦测以及自动装配到Spring容器中（我们可以使用@ComponentScan注解定义要扫描的路径）。@Bean注解通常是在标有该注解的方法中定义产生这个bean，告诉Spring这是某个类的实例，当我需要用它的时候还给我。

3.@Bean注解比@Component注解的自定义性更强，而且很多地方只能通过@Bean注解来注册bean。比如当引用第三方库的类需要装配到Spring容器的时候，就只能通过@Bean注解来实现。

@Bean注解的使用示例：

```
@Configuration
public class AppConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }
}
```

上面的代码相当于下面的XML配置：

```
<beans>
    <bean id="transferService" class="com.yanggb.TransferServiceImpl"/>
</beans>
```

下面这个例子是无法通过@Component注解实现的：

```
@Bean
public OneService getService(status) {
    case (status)  {
        when 1:
                return new serviceImpl1();
        when 2:
                return new serviceImpl2();
        when 3:
                return new serviceImpl3();
    }
}
```

### **将一个类声明为Spring的bean的注解有哪些？**

我们一般使用@Autowired注解去自动装配bean。而想要把一个类标识为可以用@Autowired注解自动装配的bean，可以采用以下的注解实现：

1.@**Component**注解。通用的注解，可标注任意类为Spring组件。如果一个Bean不知道属于哪一个层，可以使用@Component注解标注。

2.@**Repository**注解。对应持久层，即Dao层，主要用于数据库相关操作。

3.@**Service**注解。对应服务层，即Service层，主要涉及一些复杂的逻辑，需要用到Dao层（注入）。

4.@**Controller**注解。对应Spring MVC的控制层，即Controller层，主要用于接受用户请求并调用Service层的方法返回数据给前端页面。

### **Spring事务管理的方式有几种？**

1.编程式事务：在代码中硬编码（不推荐使用）。

2.声明式事务：在配置文件中配置（推荐使用），分为基于XML的声明式事务和基于注解的声明式事务。

### **Spring事务中的隔离级别有哪几种？**

在TransactionDefinition接口中定义了**五**个表示隔离级别的常量：

**ISOLATION_DEFAULT**：使用后端数据库默认的隔离级别，Mysql默认采用的**REPEATABLE_READ**隔离级别；Oracle默认采用的READ_COMMITTED隔离级别。

**ISOLATION_READ_UNCOMMITTED**：最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读。

**ISOLATION_READ_COMMITTED**：允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生

**ISOLATION_REPEATABLE_READ**：对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。

**ISOLATION_SERIALIZABLE**：最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

### **Spring事务中有哪几种事务传播行为？**

在TransactionDefinition接口中定义了**7**个表示事务传播行为的常量。

支持当前事务的情况：

**PROPAGATION_REQUIRED**：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。

**PROPAGATION_SUPPORTS**： 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。

**PROPAGATION_MANDATORY**： 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）。

不支持当前事务的情况：

**PROPAGATION_REQUIRES_NEW**： 创建一个新的事务，如果当前存在事务，则把当前事务挂起。

**PROPAGATION_NOT_SUPPORTED**： 以非事务方式运行，如果当前存在事务，则把当前事务挂起。

**PROPAGATION_NEVER**： 以非事务方式运行，如果当前存在事务，则抛出异常。

其他情况：

**PROPAGATION_NESTED**： 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价PROPAGATION_REQUIRED

### Spring的优点

（1）spring属于低侵入式设计，代码的污染极低；

（2）spring的DI机制将对象之间的依赖关系交由框架处理，减低组件的耦合性；

（3）Spring提供了AOP技术，支持将一些通用任务，如安全、事务、日志、权限等进行集中式管理，从而提供更好的复用。

（4）spring对于主流的应用框架提供了集成支持。

### **BeanFactory和ApplicationContext有什么区别？**

 BeanFactory和ApplicationContext是Spring的两大核心接口，都可以当做Spring的容器。其中ApplicationContext是BeanFactory的子接口。

（1）BeanFactory：是Spring里面最底层的接口，包含了各种Bean的定义，读取bean配置文档，管理bean的加载、实例化，控制bean的生命周期，维护bean之间的依赖关系。ApplicationContext接口作为BeanFactory的派生，除了提供BeanFactory所具有的功能外，还提供了更完整的框架功能：

①继承MessageSource，因此支持国际化。

②统一的资源文件访问方式。

③提供在监听器中注册bean的事件。

④同时加载多个配置文件。

⑤载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层。

（2）①BeanFactroy采用的是**延迟加载**形式来注入Bean的，即只有在使用到某个Bean时(调用getBean())，才对该Bean进行加载实例化。这样，我们就不能发现一些存在的Spring的配置问题。如果Bean的某一个属性没有注入，BeanFacotry加载后，直至第一次使用调用getBean方法才会抛出异常。

​    ②ApplicationContext，它是在容器启动时，一次性创建了所有的Bean。这样，在容器启动时，我们就可以发现Spring中存在的配置错误，这样有利于检查所依赖属性是否注入。 ApplicationContext启动后预载入所有的单实例Bean，通过预载入单实例bean ,确保当你需要的时候，你就不用等待，因为它们已经创建好了。

​    ③相对于基本的BeanFactory，ApplicationContext 唯一的不足是占用内存空间。当应用程序配置Bean较多时，程序启动较慢。

（3）BeanFactory通常以编程的方式被创建，ApplicationContext还能以声明的方式创建，如使用ContextLoader。

（4）BeanFactory和ApplicationContext都支持BeanPostProcessor、BeanFactoryPostProcessor的使用，但两者之间的区别是：BeanFactory需要手动注册，而ApplicationContext则是自动注册。

### **Spring如何处理线程并发问题？**

在一般情况下，只有无状态的Bean才可以在多线程环境下共享，在Spring中，绝大部分Bean都可以声明为singleton作用域，因为Spring对一些Bean中非线程安全状态采用ThreadLocal进行处理，解决线程安全问题。

ThreadLocal和线程同步机制都是为了解决多线程中相同变量的访问冲突问题。同步机制采用了“时间换空间”的方式，仅提供一份变量，不同的线程在访问前需要获取锁，没获得锁的线程则需要排队。而ThreadLocal采用了“空间换时间”的方式。

### **Spring基于xml注入bean的几种方式：**

（1）Set方法注入；

（2）构造器注入：①通过index设置参数的位置；②通过type设置参数类型；

（3）静态工厂注入；

（4）实例工厂；

### **Spring的自动装配：**

在spring中，对象无需自己查找或创建与其关联的其他对象，由容器负责把需要相互协作的对象引用赋予各个对象，使用autowire来配置自动装载模式。

在Spring框架xml配置中共有5种自动装配：

（1）no：默认的方式是不进行自动装配的，通过手工设置ref属性来进行装配bean。

（2）byName：通过bean的名称进行自动装配，如果一个bean的 property 与另一bean 的name 相同，就进行自动装配。 

（3）byType：通过参数的数据类型进行自动装配。

（4）constructor：利用构造函数进行装配，并且构造函数的参数通过byType进行装配。

（5）autodetect：自动探测，如果有构造方法，通过 construct的方式自动装配，否则使用 byType的方式自动装配。

基于注解的方式：

使用@Autowired注解来自动装配指定的bean。在使用@Autowired注解之前需要在Spring配置文件进行配置，<context:annotation-config />。在启动spring IoC时，容器自动装载了一个AutowiredAnnotationBeanPostProcessor后置处理器，当容器扫描到@Autowied、@Resource或@Inject时，就会在IoC容器自动查找需要的bean，并装配给该对象的属性。在使用@Autowired时，首先在容器中查询对应类型的bean：

如果查询结果刚好为一个，就将该bean装配给@Autowired指定的数据；

如果查询的结果不止一个，那么@Autowired会根据名称来查找；

如果上述查找的结果为空，那么会抛出异常。解决方法时，使用required=false。

> @Autowired可用于：构造函数、成员变量、Setter方法
>
> 注：@Autowired和@Resource之间的区别
>
> (1) @Autowired默认是按照类型装配注入的，默认情况下它要求依赖对象必须存在（可以设置它required属性为false）。
>
> (2) @Resource默认是按照名称来装配注入的，只有当找不到与名称匹配的bean才会按照类型来装配注入。

### **Spring事务的实现方式和实现原理：**

Spring事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，spring是无法提供事务功能的。真正的数据库层的事务提交和回滚是通过binlog或者redo log实现的。

### **Spring事务的种类：**

spring支持编程式事务管理和声明式事务管理两种方式：

①编程式事务管理使用TransactionTemplate。

②声明式事务管理建立在AOP之上的。其本质是通过AOP功能，对方法前后进行拦截，将事务处理的功能编织到拦截的方法中，也就是在目标方法开始之前加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。

> 声明式事务最大的优点就是不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明或通过@Transactional注解的方式，便可以将事务规则应用到业务逻辑中。
>
> 声明式事务管理要优于编程式事务管理，这正是spring倡导的非侵入式的开发方式，使业务代码不受污染，只要加上注解就可以获得完全的事务支持。唯一不足地方是，最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。

### **Spring框架中有哪些不同类型的事件？**

Spring 提供了以下5种标准的事件：

（1）上下文更新事件（ContextRefreshedEvent）：在调用ConfigurableApplicationContext 接口中的**refresh**()方法时被触发。

（2）上下文开始事件（ContextStartedEvent）：当容器调用ConfigurableApplicationContext的**Start**()方法开始/重新开始容器时触发该事件。

（3）上下文停止事件（ContextStoppedEvent）：当容器调用ConfigurableApplicationContext的**Stop**()方法停止容器时触发该事件。

（4）上下文关闭事件（ContextClosedEvent）：当**ApplicationContext**被**关闭**时触发该事件。容器被关闭时，其管理的所有单例Bean都被销毁。

（5）请求处理事件（RequestHandledEvent）：在Web应用中，当一个http请求（**request**）结束触发该事件。

如果一个bean实现了ApplicationListener接口，当一个ApplicationEvent 被**发布**以后，bean会自动被通知。

### **Spring通知有哪些类型？**

（1）前置通知（Before advice）：在某连接点（join point）之前执行的通知，但这个通知不能阻止连接点前的执行（除非它抛出一个异常）。

（2）返回后通知（After returning advice）：在某连接点（join point）正常完成后执行的通知：例如，一个方法没有抛出任何异常，正常返回。 

（3）抛出异常后通知（After throwing advice）：在方法抛出异常退出时执行的通知。 

（4）后通知（After (finally) advice）：当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。 

（5）环绕通知（Around Advice）：包围一个连接点（join point）的通知，如方法调用。这是最强大的一种通知类型。 环绕通知可以在方法调用前后完成自定义的行为。它也会选择是否继续执行连接点或直接返回它们自己的返回值或抛出异常来结束执行。 环绕通知是最常用的一种通知类型。大部分基于拦截的AOP框架，例如Nanning和JBoss4，都只提供环绕通知。 

> 同一个aspect，不同advice的执行顺序：
>
> ①没有异常情况下的执行顺序：
>
> around before advice
> before advice
> target method 执行
> around after advice
> after advice
> afterReturning
>
> ②有异常情况下的执行顺序：
>
> around before advice
> before advice
> target method 执行
> around after advice
> after advice
> afterThrowing:异常发生
> java.lang.RuntimeException: 异常发生

