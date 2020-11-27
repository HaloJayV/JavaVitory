#### **Spring是什么?**

​    Spring是一个轻量级的IoC和AOP容器框架。是为Java应用程序提供基础性服务的一套框架，目的是用于简化企业应用程序的开发，它使得开发者只需要关心业务需求。常见的配置方式有三种：基于XML的配置、基于注解的配置、基于Java的配置。

主要由以下几个模块组成：

Spring Core：核心类库，可以说 Spring 其他所有的功能都需要依赖于该类库。主要提供 IOC 依赖注入功能。

Spring Context：提供框架式的Bean访问方式，以及企业级功能（JNDI、定时任务等）；

Spring Aspects：该模块为与AspectJ的集成提供支持。

Spring AOP：AOP切面服务，体现了spring生命周期；

Spring DAO：对JDBC的抽象，简化了数据访问异常的处理；

Spring ORM：对现有的ORM框架的支持；

Spring Web：提供了基本的面向Web的综合特性，例如多方文件上传；

Spring MVC：提供面向Web应用的Model-View-Controller实现。

Spring Test：提供了对JUnit和TestNG测试的支持。

#### Spring 的 6 个特征:

- **核心技术** ：依赖注入(DI)，AOP，事件(events)，资源，i18n，验证，数据绑定，类型转换，SpEL。
- **测试** ：模拟对象，TestContext框架，Spring MVC 测试，WebTestClient。
- **数据访问** ：事务，DAO支持，JDBC，ORM，编组XML。
- **Web支持** : Spring MVC和Spring WebFlux Web框架。
- **集成** ：远程处理，JMS，JCA，JMX，电子邮件，任务，调度，缓存。
- **语言** ：Kotlin，Groovy，动态语言。

#### **Spring 的优点？**

1.降低了组件之间的耦合性 ，实现了软件各层之间的解耦
2.可以使用容易提供的众多服务，如事务管理，消息服务等
3.容器提供单例模式支持
4.容器提供了AOP技术，支持将一些通用任务，如安全、事务、日志、权限等进行集中式管理，从而提供更好的复用。
5.容器提供了众多的辅助类，能加快应用的开发
6.spring对于主流的应用框架提供了集成支持，如[hibernate](http://lib.csdn.net/base/17)，JPA，Struts等
7.spring属于低侵入式设计，代码的污染极低
8.独立于各种应用服务器
9.spring的DI机制将对象之间的依赖关系交由框架处理，减低组件的耦合性；
10.Spring的高度开放性，并不强制应用完全依赖于Spring，开发者可以自由选择spring的部分或全部

### IOC

（1）IOC就是控制反转，是指创建对象的控制权的转移，以前创建对象的主动权和时机是由自己把控的，而现在这种权力转移到Spring容器中，并由容器根据配置文件去创建实例和管理各个实例之间的依赖关系，对象与对象之间松散耦合，也利于功能的复用。DI依赖注入，和控制反转是同一个概念的不同角度的描述，即 应用程序在运行时依赖IoC容器来动态注入对象需要的外部资源。

（2）最直观的表达就是，IOC让对象的创建不用去new了，可以由spring自动生产，使用java的反射机制，根据配置文件在运行时动态的去创建对象以及管理对象，并调用对象的方法的。

（3）Spring的IOC有三种注入方式 ：构造器注入、setter方法注入、根据注解注入。

> IoC让相互协作的组件保持松散的耦合，而AOP编程允许你把遍布于应用各层的功能分离出来形成可重用的功能组件。



### AOP

OOP面向对象，允许开发者定义纵向的关系，但并适用于定义横向的关系，导致了大量代码的重复，而不利于各个模块的重用。

AOP，一般称为面向切面，作为面向对象的一种补充，用于将那些与业务无关，但却对多个对象产生影响的公共行为和逻辑，抽取并封装为一个可重用的模块，这个模块被命名为“切面”（Aspect），减少系统中的重复代码，降低了模块间的耦合度，同时提高了系统的可维护性。可用于权限认证、日志、事务处理。

AOP实现的关键在于 代理模式，AOP代理主要分为静态代理和动态代理。静态代理的代表为AspectJ；动态代理则以Spring AOP为代表。

（1）AspectJ是静态代理的增强，所谓静态代理，就是AOP框架会在编译阶段生成AOP代理类，因此也称为编译时增强，他会在编译阶段将AspectJ(切面)织入到Java字节码中，运行的时候就是增强之后的AOP对象。

（2）Spring AOP使用的动态代理，所谓的动态代理就是说AOP框架不会去修改字节码，而是每次运行时在内存中临时为方法生成一个AOP对象，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法。

Spring AOP中的动态代理主要有两种方式，JDK动态代理和CGLIB动态代理：

​    ①JDK动态代理只提供接口的代理，不支持类的代理。核心InvocationHandler接口和Proxy类，InvocationHandler 通过invoke()方法反射来调用目标类中的代码，动态地将横切逻辑和业务编织在一起；接着，Proxy利用 InvocationHandler动态创建一个符合某一接口的的实例, 生成目标类的代理对象。

​    ②如果代理类没有实现 InvocationHandler 接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成指定类的一个子类对象，并覆盖其中特定方法并添加增强代码，从而实现AOP。CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。

（3）静态代理与动态代理区别在于生成AOP代理对象的时机不同，相对来说AspectJ的静态代理方式具有更好的性能，但是AspectJ需要特定的编译器进行处理，而Spring AOP则无需特定的编译器处理。

>  InvocationHandler 的 invoke(Object proxy,Method method,Object[] args)：proxy是最终生成的代理实例; method 是被代理目标实例的某个具体方法; args 是被代理目标实例某个方法的具体入参, 在方法反射调用时使用。

### Spring AOP 和 AspectJ AOP 有什么区别？

**Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。** Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)。

Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ 相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，

如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择 AspectJ ，它比Spring AOP 快很多。

### Spring 中的 bean 的作用域有哪些?

- singleton : 唯一 bean 实例，Spring 中的 bean 默认都是单例的。
- prototype : 每次请求都会创建一个新的 bean 实例。
- request : 每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效。
- session : 每一次HTTP请求都会产生一个新的 bean，该bean仅在当前 HTTP session 内有效。
- global-session： 全局session作用域，仅仅在基于portlet的web应用中才有意义，Spring5已经没有了。Portlet是能够生成语义代码(例如：HTML)片段的小型Java Web插件。它们基于portlet容器，可以像servlet一样处理HTTP请求。但是，与 servlet 不同，每个 portlet 都有不同的会话

### Spring 中的单例 bean 的线程安全问题了解吗？

 Spring框架并没有对单例bean进行任何多线程的封装处理。关于单例bean的线程安全和并发问题需要开发者自行去搞定。但实际上，大部分的Spring bean并没有可变的状态(比如Serview类和DAO类)，所以在某种程度上说Spring的单例bean是线程安全的。如果你的bean有多种状态的话（比如 View Model 对象），就需要自行保证线程安全。最浅显的解决办法就是将多态bean的作用域由“singleton”变更为“prototype”。

常见的有两种解决办法：

1. 在Bean对象中尽量避免定义可变的成员变量（不太现实）。
2. 在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在 ThreadLocal 中（推荐的一种方式）。

### **BeanFactory和ApplicationContext有什么区别？**

​    BeanFactory和ApplicationContext是Spring的两大核心接口，都可以当做Spring的容器。其中ApplicationContext是BeanFactory的子接口。

（1）BeanFactory：是Spring里面最底层的接口，包含了各种Bean的定义，读取bean配置文档，管理bean的加载、实例化，控制bean的生命周期，维护bean之间的依赖关系。ApplicationContext接口作为BeanFactory的派生，除了提供BeanFactory所具有的功能外，还提供了更完整的框架功能：

①继承MessageSource，因此支持国际化。

②统一的资源文件访问方式。

③提供在监听器中注册bean的事件。

④同时加载多个配置文件。

⑤载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层。

（2）①BeanFactroy采用的是延迟加载形式来注入Bean的，即只有在使用到某个Bean时(调用getBean())，才对该Bean进行加载实例化。这样，我们就不能发现一些存在的Spring的配置问题。如果Bean的某一个属性没有注入，BeanFacotry加载后，直至第一次使用调用getBean方法才会抛出异常。

​    ②ApplicationContext，它是在容器启动时，一次性创建了所有的Bean。这样，在容器启动时，我们就可以发现Spring中存在的配置错误，这样有利于检查所依赖属性是否注入。 ApplicationContext启动后预载入所有的单实例Bean，通过预载入单实例bean ,确保当你需要的时候，你就不用等待，因为它们已经创建好了。

​    ③相对于基本的BeanFactory，ApplicationContext 唯一的不足是占用内存空间。当应用程序配置Bean较多时，程序启动较慢。

（3）BeanFactory通常以编程的方式被创建，ApplicationContext还能以声明的方式创建，如使用ContextLoader。

（4）BeanFactory和ApplicationContext都支持BeanPostProcessor、BeanFactoryPostProcessor的使用，但两者之间的区别是：BeanFactory需要手动注册，而ApplicationContext则是自动注册。

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

### **解释一下Spring AOP里面的几个名词：**

（1）切面（Aspect）：被抽取的公共模块，可能会横切多个对象。 在Spring AOP中，切面可以使用通用类（基于模式的风格） 或者在普通类中以 @AspectJ 注解来实现。

（2）连接点（Join point）：指方法，在Spring AOP中，一个连接点 总是 代表一个方法的执行。 

（3）通知（Advice）：在切面的某个特定的连接点（Join point）上执行的动作。通知有各种类型，其中包括“around”、“before”和“after”等通知。许多AOP框架，包括Spring，都是以拦截器做通知模型， 并维护一个以连接点为中心的拦截器链。

（4）切入点（Pointcut）：切入点是指 我们要对哪些Join point进行拦截的定义。通过切入点表达式，指定拦截的方法，比如指定拦截add*、search*。

（5）引入（Introduction）：（也被称为内部类型声明（inter-type declaration））。声明额外的方法或者某个类型的字段。Spring允许引入新的接口（以及一个对应的实现）到任何被代理的对象。例如，你可以使用一个引入来使bean实现 IsModified 接口，以便简化缓存机制。

（6）目标对象（Target Object）： 被一个或者多个切面（aspect）所通知（advise）的对象。也有人把它叫做 被通知（adviced） 对象。 既然Spring AOP是通过运行时代理实现的，这个对象永远是一个 被代理（proxied） 对象。

（7）织入（Weaving）：指把增强应用到目标对象来创建新的代理对象的过程。Spring是在运行时完成织入。

切入点（pointcut）和连接点（join point）匹配的概念是AOP的关键，这使得AOP不同于其它仅仅提供拦截功能的旧技术。 切入点使得定位通知（advice）可独立于OO层次。 例如，一个提供声明式事务管理的around通知可以被应用到一组横跨多个对象中的方法上（例如服务层的所有业务操作）。

### **Spring框架中有哪些不同类型的事件？**

Spring 提供了以下5种标准的事件：

（1）上下文更新事件（ContextRefreshedEvent）：在调用ConfigurableApplicationContext 接口中的refresh()方法时被触发。

（2）上下文开始事件（ContextStartedEvent）：当容器调用ConfigurableApplicationContext的Start()方法开始/重新开始容器时触发该事件。

（3）上下文停止事件（ContextStoppedEvent）：当容器调用ConfigurableApplicationContext的Stop()方法停止容器时触发该事件。

（4）上下文关闭事件（ContextClosedEvent）：当ApplicationContext被关闭时触发该事件。容器被关闭时，其管理的所有单例Bean都被销毁。

（5）请求处理事件（RequestHandledEvent）：在Web应用中，当一个http请求（request）结束触发该事件。

如果一个bean实现了ApplicationListener接口，当一个ApplicationEvent 被发布以后，bean会自动被通知。

### **Spring 框架中都用到了哪些设计模式？**

（1）工厂模式：BeanFactory就是简单工厂模式的体现，用来创建对象的实例；

（2）单例模式：Bean默认为单例模式。

（3）代理模式：Spring的AOP功能用到了JDK的动态代理和CGLIB字节码生成技术；

（4）模板方法：用来解决代码重复的问题。比如. RestTemplate, JmsTemplate, JpaTemplate。

（5）观察者模式：定义对象键一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都会得到通知被制动更新，如Spring中listener的实现--ApplicationListener。

### **请解释Spring Bean的生命周期？**

 首先说一下Servlet的生命周期：实例化，初始init，接收请求service，销毁destroy；

 Spring上下文中的Bean生命周期也类似，如下：

（1）实例化Bean：

对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用**createBean**进行实例化。对于ApplicationContext容器，当容器启动结束后，通过获取BeanDefinition对象中的信息，实例化所有的bean。

（2）设置对象属性（依赖注入）：

实例化后的对象被封装在BeanWrapper对象中，紧接着，Spring根据BeanDefinition中的信息 以及 通过BeanWrapper提供的设置属性的接口完成依赖注入。

（3）处理Aware接口：

接着，Spring会检测该对象是否实现了xxxAware接口，并将相关的xxxAware实例注入给Bean：

①如果这个Bean已经实现了BeanNameAware接口，会调用它实现的setBeanName(String beanId)方法，此处传递的就是Spring配置文件中Bean的id值；

②如果这个Bean已经实现了BeanFactoryAware接口，会调用它实现的setBeanFactory()方法，传递的是Spring工厂自身。

③如果这个Bean已经实现了ApplicationContextAware接口，会调用setApplicationContext(ApplicationContext)方法，传入Spring上下文；

（4）BeanPostProcessor：

如果想对Bean进行一些自定义的处理，那么可以让Bean实现了BeanPostProcessor接口，那将会调用postProcessBeforeInitialization(Object obj, String s)方法。

（5）InitializingBean 与 init-method：

如果Bean在Spring配置文件中配置了 init-method 属性，则会自动调用其配置的初始化方法。

（6）如果这个Bean实现了BeanPostProcessor接口，将会调用postProcessAfterInitialization(Object obj, String s)方法；由于这个方法是在Bean初始化结束时调用的，所以可以被应用于内存或缓存技术；

> 以上几个步骤完成后，Bean就已经被正确创建了，之后就可以使用这个Bean了。

（7）DisposableBean：

当Bean不再需要时，会经过清理阶段，如果Bean实现了DisposableBean这个接口，会调用其实现的destroy()方法；

（8）destroy-method：

最后，如果这个Bean的Spring配置中配置了destroy-method属性，会自动调用其配置的销毁方法。

### BeanDefinition

BeanDefinition 用于保存 Bean 的相关信息，包括属性、构造方法参数、依赖的 Bean 名称及是否单例、延迟加载等，它是实例化 Bean 的原材料，Spring 就是根据 BeanDefinition 中的信息实例化 Bean。

一个 BeanDefinition 描述了一个 Bean 实例，实例包含属性值、构造方法参数值以及更多实现信息。该 BeanDefinition 只是是一个最小的接口，主要目的是允许修改属性值和其他 Bean 元数据，这里列出几个核心方法。可以看到 BeanDefinition 接口提供了一系列操作 Bean 元数据的set、get方法，这些操作为 Bean 的描述定义了一套模板，具体的实现则交由子类。

```java
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {

	// 单例、原型标识符
	String SCOPE_SINGLETON = ConfigurableBeanFactory.SCOPE_SINGLETON;
	String SCOPE_PROTOTYPE = ConfigurableBeanFactory.SCOPE_PROTOTYPE;

    // 标识 Bean 的类别，分别对应 用户定义的 Bean、来源于配置文件的 Bean、Spring 内部的 Bean
	int ROLE_APPLICATION = 0;
	int ROLE_SUPPORT = 1;
	int ROLE_INFRASTRUCTURE = 2;

    // 设置、返回 Bean 的父类名称
	void setParentName(@Nullable String parentName);
	String getParentName();

    // 设置、返回 Bean 的 className
	void setBeanClassName(@Nullable String beanClassName);
	String getBeanClassName();

    // 设置、返回 Bean 的作用域
	void setScope(@Nullable String scope);
	String getScope();

    // 设置、返回 Bean 是否懒加载
	void setLazyInit(boolean lazyInit);
	boolean isLazyInit();
	
	// 设置、返回当前 Bean 所依赖的其它 Bean 名称。
	void setDependsOn(@Nullable String... dependsOn);
	String[] getDependsOn();
	
	// 设置、返回 Bean 是否可以自动注入。只对 @Autowired 注解有效
	void setAutowireCandidate(boolean autowireCandidate);
	boolean isAutowireCandidate();
	
	// 设置、返回当前 Bean 是否为主要候选 Bean 。
	// 当同一个接口有多个实现类时，通过该属性来配置某个 Bean 为主候选 Bean。
	void setPrimary(boolean primary);
	boolean isPrimary();

    // 设置、返回创建该 Bean 的工厂类。
	void setFactoryBeanName(@Nullable String factoryBeanName);
	String getFactoryBeanName();
	
	// 设置、返回创建该 Bean 的工厂方法
	void setFactoryMethodName(@Nullable String factoryMethodName);
	String getFactoryMethodName();
	
	// 返回该 Bean 构造方法参数值、所有属性
	ConstructorArgumentValues getConstructorArgumentValues();
	MutablePropertyValues getPropertyValues();

    // 返回该 Bean 是否是单例、是否是非单例、是否是抽象的
	boolean isSingleton();
	boolean isPrototype();
	boolean isAbstract();

    // 返回 Bean 的类别。类别对应上面的三个属性值。
	int getRole();

    ...
}
```

#### AnnotatedBeanDefinition 

AnnotatedBeanDefinition 是 BeanDefinition 子接口之一，该接口扩展了 BeanDefinition 的功能，其用来操作注解元数据。一般情况下，通过注解方式得到的 Bean（@Component、@Bean），其 BeanDefinition 类型都是该接口的实现类。

```java
public interface AnnotatedBeanDefinition extends BeanDefinition {

	// 获得当前 Bean 的注解元数据
	AnnotationMetadata getMetadata();

	// 获得当前 Bean 的工厂方法上的元数据
	MethodMetadata getFactoryMethodMetadata();
}
```

该接口可以返回两个元数据的类：

- AnnotationMetadata：主要对 Bean 的注解信息进行操作，如：获取当前 Bean 标注的所有注解、判断是否包含指定注解。
- MethodMetadata：方法的元数据类。提供获取方法名称、此方法所属类的全类名、是否是抽象方法、判断是否是静态方法、判断是否是final方法等。

#### AbstractBeanDefinition

AbstractBeanDefinition 是 BeanDefinition 的子抽象类，也是其他 BeanDefinition 类型的基类，其实现了接口中定义的一系列操作方法，并定义了一系列的常量属性，这些常量会直接影响到 Spring 实例化 Bean 时的策略。核心属性如下。

```java
public abstract class AbstractBeanDefinition extends BeanMetadataAttributeAccessor
		implements BeanDefinition, Cloneable {

    // 默认的 SCOPE，默认是单例
	public static final String SCOPE_DEFAULT = "";

	// 不进行自动装配
	public static final int AUTOWIRE_NO = AutowireCapableBeanFactory.AUTOWIRE_NO;
    // 根据 Bean 的名字进行自动装配，byName
	public static final int AUTOWIRE_BY_NAME = AutowireCapableBeanFactory.AUTOWIRE_BY_NAME;
	// 根据 Bean 的类型进行自动装配，byType
	public static final int AUTOWIRE_BY_TYPE = AutowireCapableBeanFactory.AUTOWIRE_BY_TYPE;
	// 根据构造器进行自动装配
	public static final int AUTOWIRE_CONSTRUCTOR = AutowireCapableBeanFactory.AUTOWIRE_CONSTRUCTOR;
	// 首先尝试按构造器自动装配。如果失败，再尝试使用 byType 进行自动装配。（Spring 3.0 之后已废除）
	public static final int AUTOWIRE_AUTODETECT = AutowireCapableBeanFactory.AUTOWIRE_AUTODETECT;

    // 通过依赖检查来查看 Bean 的每个属性是否都设置完成
    // 以下常量分别对应：不检查、对依赖对象检查、对基本类型，字符串和集合进行检查、对全部属性进行检查
	public static final int DEPENDENCY_CHECK_NONE = 0;
	public static final int DEPENDENCY_CHECK_OBJECTS = 1;
	public static final int DEPENDENCY_CHECK_SIMPLE = 2;
	public static final int DEPENDENCY_CHECK_ALL = 3;

	// 关闭应用上下文时需调用的方法名称
	public static final String INFER_METHOD = "(inferred)";

    // 存放 Bean 的 Class 对象
	private volatile Object beanClass;

	// Bean 的作用范围
	private String scope = SCOPE_DEFAULT;

    // 非抽象
	private boolean abstractFlag = false;
	// 非延迟加载
	private boolean lazyInit = false;
    // 默认不自动装配
	private int autowireMode = AUTOWIRE_NO;
    // 默认不依赖检查
	private int dependencyCheck = DEPENDENCY_CHECK_NONE;

	// 依赖的 Bean 列表
	private String[] dependsOn;

    // 可以作为自动装配的候选者，意味着可以自动装配到其他 Bean 的某个属性中
	private boolean autowireCandidate = true;
    
	// 创建当前 Bean 实例工厂类名称
	private String factoryBeanName;
    // 创建当前 Bean 实例工厂类中方法名称
	private String factoryMethodName;

	// 存储构造方法的参数
	private ConstructorArgumentValues constructorArgumentValues;
    // 存储 Bean 属性名称以及对应的值
	private MutablePropertyValues propertyValues;
    // 存储被覆盖的方法信息
	private MethodOverrides methodOverrides;

	// init、destroy 方法名称
	private String initMethodName;
	private String destroyMethodName;

    // 是否执行 init 和 destroy 方法
	private boolean enforceInitMethod = true;
	private boolean enforceDestroyMethod = true;

    // Bean 是否是用户定义的而不是应用程序本身定义的
	private boolean synthetic = false;

    // Bean 的身份类别，默认是用户定义的 Bean
	private int role = BeanDefinition.ROLE_APPLICATION;

	// Bean 的描述信息
	private String description;

	// Bean 定义的资源
	private Resource resource;
	
	...
}
```

以上是 AbstractBeanDefinition 中定义的一些常量和属性，该类中还有一部分是操作这些属性的 set 和 get 方法，这些方法都由子类来操作，且应用程序中真正使用的也是这些子类 BeanDefinition。

先来看 AbstractBeanDefinition 直接实现类：RootBeanDefinition、GenericBeanDefinition、ChildBeanDefinition。

#### RootBeanDefinition

该类继承自 AbstractBeanDefinition，它可以单独作为一个 BeanDefinition，也可以作为其他 BeanDefinition 的父类。

RootBeanDefinition 在 AbstractBeanDefinition 的基础上定义了更多属性。

```java
public class RootBeanDefinition extends AbstractBeanDefinition {

    // BeanDefinitionHolder 存储 Bean 的名称、别名、BeanDefinition
	private BeanDefinitionHolder decoratedDefinition;

	// AnnotatedElement 是java反射包的接口，通过它可以查看 Bean 的注解信息
	private AnnotatedElement qualifiedElement;

    // 允许缓存
	boolean allowCaching = true;
    
    // 工厂方法是否唯一
	boolean isFactoryMethodUnique = false;

	// 封装了 java.lang.reflect.Type，提供了泛型相关的操作
	volatile ResolvableType targetType;

	// 缓存 Class，表示 RootBeanDefinition 存储哪个类的信息
	volatile Class<?> resolvedTargetType;

	// 缓存工厂方法的返回类型
	volatile ResolvableType factoryMethodReturnType;

	// 这是以下四个构造方法字段的通用锁
	final Object constructorArgumentLock = new Object();
	// 用于缓存已解析的构造方法或工厂方法
	Executable resolvedConstructorOrFactoryMethod;
	// 将构造方法参数标记为已解析
	boolean constructorArgumentsResolved = false;
	// 用于缓存完全解析的构造方法参数
	Object[] resolvedConstructorArguments;
	// 缓存待解析的构造方法参数
	Object[] preparedConstructorArguments;

	// 这是以下两个后处理字段的通用锁
	final Object postProcessingLock = new Object();
	// 表明是否被 MergedBeanDefinitionPostProcessor 处理过
	boolean postProcessed = false;
	// 在生成代理的时候会使用，表明是否已经生成代理
	volatile Boolean beforeInstantiationResolved;

	// 实际缓存的类型是 Constructor、Field、Method 类型
	private Set<Member> externallyManagedConfigMembers;

	// InitializingBean中 的 init 回调函数名 afterPropertiesSet 会在这里记录，以便进行生命周期回调
	private Set<String> externallyManagedInitMethods;

	// DisposableBean 的 destroy 回调函数名 destroy 会在这里记录，以便进生命周期回调
	private Set<String> externallyManagedDestroyMethods;

    ...
}
```

#### ChildBeanDefinition

该类继承自 AbstractBeanDefinition。其相当于一个子类，不可以单独存在，必须依赖一个父 BeanDetintion，构造 ChildBeanDefinition 时，通过构造方法传入父 BeanDetintion 的名称或通过 setParentName 设置父名称。它可以从父类继承方法参数、属性值，并可以重写父类的方法，同时也可以增加新的属性或者方法。若重新定义 init 方法，destroy 方法或者静态工厂方法，ChildBeanDefinition 会重写父类的设置。

> 从 Spring 2.5 开始，以编程方式注册 Bean 定义的首选方法是 GenericBeanDefinition，GenericBeanDefinition 可以有效替代 ChildBeanDefinition 的绝大分部使用场合。

#### GenericBeanDefinition

GenericBeanDefinition 是 Spring 2.5 以后新引入的 BeanDefinition，是 ChildBeanDefinition 更好的替代者，它同样可以通过 setParentName 方法设置父 BeanDefinition。

------

最后三个 BeanDefinition 既实现了 AnnotatedBeanDefinition 接口，又间接继承 AbstractBeanDefinition 抽象类，这些 BeanDefinition 描述的都是注解形式的 Bean。

#### ConfigurationClassBeanDefinition

该类继承自 RootBeanDefinition ，并实现了 AnnotatedBeanDefinition 接口。这个 BeanDefinition 用来描述在标注 @Configuration 注解的类中，通过 @Bean 注解实例化的 Bean。

其功能特点如下：

1、如果 @Bean 注解没有指定 Bean 的名字，默认会用方法的名字命名 Bean。

2、标注 @Configuration 注解的类会成为一个工厂类，而标注 @Bean 注解的方法会成为工厂方法，通过工厂方法实例化 Bean，而不是直接通过构造方法初始化。

3、标注 @Bean 注解的类会使用构造方法自动装配

#### AnnotatedGenericBeanDefinition

该类继承自 GenericBeanDefinition ，并实现了 AnnotatedBeanDefinition 接口。这个 BeanDefinition 用来描述标注 @Configuration 注解的 Bean。

#### ScannedGenericBeanDefinition

该类继承自 GenericBeanDefinition ，并实现了 AnnotatedBeanDefinition 接口。这个 BeanDefinition 用来描述标注 @Component 注解的 Bean，其派生注解如 @Service、@Controller 也同理。

最后，我们来做个总结。BeanDefinition 主要是用来描述 Bean，其存储了 Bean 的相关信息，Spring 实例化 Bean 时需读取该 Bean 对应的 BeanDefinition。BeanDefinition 整体可以分为两类，一类是描述通用的 Bean，还有一类是描述注解形式的 Bean。一般前者在 XML 时期定义 <bean‘> 标签以及在 Spring 内部使用较多，而现今我们大都使用后者，通过注解形式加载 Bean。

### Spring循环依赖

两个对象相互引用，如相互set即造成循环依赖，通常通过3个Map来解决循环依赖

就Spring生态上而言，Spring的Bean是由一个建模的类BeanDefinition创建而来的，Bean有一系列复杂的生命周期。生命周期大概是这样的：

首先Spring容器启动，容器会进行扫描后会变成BeanDefinition后存放到一个BeanDefinition Map中，然后对这个map做一个遍历并做验证，如验证是否单例、原型、懒加载、DepensOn、抽象、factoryBean、名字符合等等，验证完后容器会获取当前实例话的这个类有没有存在单例池中、是否被提前暴露。如果没有被提前暴露，Bean就会去开始创建Bean：先通过推断构造方法，把当前这个Bean所代表的的类当中的构造方法，推断得到一个最佳的构造方法，然后通过反射去实例化这个Java对象，接着对这个Bean做一些初始化工作：比如是否需要做BeanDefinition的合并，是否当前容器是否支持循环依赖，如果支持循环依赖会提前表露当前Java对象所对应的ObjectFactory工厂类，暴露即存放到二级缓存中的map中，然后进行属性填充，即自动注入。接着执行各种Aware接口的回调，然后执行生命周期初始化的回调，如@PostConstruct、InitializerBean、xml等来执行生命周期的初始化回调方法。回调完成之后，如果有AOP，还会生成代理对象，如果没有则会进行事件发布等，此时生命周期就完成了，之后放进单例池中，就会在Spring中存在了。

#### 如何解决？

- Spring是通过递归的方式获取目标bean及其所依赖的bean的；
- Spring实例化一个bean的时候，是分两步进行的，首先实例化目标bean，然后为其注入属性。

结合这两点，也就是说，Spring在实例化一个bean的时候，是首先递归的实例化其所依赖的所有bean，直到某个bean没有依赖其他bean，此时就会将该实例返回，然后反递归的将获取到的bean设置为各个上层bean的属性的。

##### 简单来讲

spring官方推荐构造器注入，构造器注入的不能循环依赖，因为对象都还在创建中，Jvm都还没返回实例，所以注入不了
用字段/setter注入的就可以循环依赖，因为都是先初始化了实例再填充字段值的。即使不用构造器注入依然可能会有循环依赖的问题。例如@Async就是典型的例子，@Transactional是在从单例工厂获取时就实现代理了，而@Async则是在注入后才实现代理对象。而有时成功有时失败是由于操作系统不同导致读取jar时文件顺序不一样，最后实例创建循序不一样。



### **Spring如何处理线程并发问题？**

在一般情况下，只有无状态的Bean才可以在多线程环境下共享，在Spring中，绝大部分Bean都可以声明为singleton作用域，因为Spring对一些Bean中非线程安全状态采用ThreadLocal进行处理，解决线程安全问题。

ThreadLocal和线程同步机制都是为了解决多线程中相同变量的访问冲突问题。同步机制采用了“时间换空间”的方式，仅提供一份变量，不同的线程在访问前需要获取锁，没获得锁的线程则需要排队。而ThreadLocal采用了“空间换时间”的方式。

ThreadLocal会为每一个线程提供一个独立的变量副本，从而隔离了多个线程对数据的访问冲突。因为每一个线程都拥有自己的变量副本，从而也就没有必要对该变量进行同步了。ThreadLocal提供了线程安全的共享对象，在编写多线程代码时，可以把不安全的变量封装进ThreadLocal。

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

**（1）Spring事务的种类：**

spring支持编程式事务管理和声明式事务管理两种方式：

①编程式事务管理使用TransactionTemplate。

②声明式事务管理建立在AOP之上的。其本质是通过AOP功能，对方法前后进行拦截，将事务处理的功能编织到拦截的方法中，也就是在目标方法开始之前加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。

> 声明式事务最大的优点就是不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明或通过@Transactional注解的方式，便可以将事务规则应用到业务逻辑中。
>
> 声明式事务管理要优于编程式事务管理，这正是spring倡导的非侵入式的开发方式，使业务代码不受污染，只要加上注解就可以获得完全的事务支持。唯一不足地方是，最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。

**（2）spring的事务传播行为：**

spring事务的传播行为说的是，当多个事务同时存在的时候，spring如何处理这些事务的行为。

**TransactionDefinition 接口中定义了7 个表示传播行为的常量：**

> ① PROPAGATION_REQUIRED：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的设置。
>
> ② PROPAGATION_SUPPORTS：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。‘
>
> ③ PROPAGATION_MANDATORY：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。
>
> ④ PROPAGATION_REQUIRES_NEW：创建新事务，无论当前存不存在事务，都创建新事务。
>
> ⑤ PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
>
> ⑥ PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。
>
> ⑦ PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行。

**（3）Spring中的隔离级别：**

**TransactionDefinition 接口中定义了五个表示隔离级别的常量：**

> ① ISOLATION_DEFAULT：这是个 PlatfromTransactionManager 默认的隔离级别，使用数据库默认的事务隔离级别。
>
> ② ISOLATION_READ_UNCOMMITTED：读未提交，允许另外一个事务可以看到这个事务未提交的数据。
>
> ③ ISOLATION_READ_COMMITTED：读已提交，保证一个事务修改的数据提交后才能被另一事务读取，而且能看到该事务对已有记录的更新。
>
> ④ ISOLATION_REPEATABLE_READ：可重复读，保证一个事务修改的数据提交后才能被另一事务读取，但是不能看到该事务对已有记录的更新。
>
> ⑤ ISOLATION_SERIALIZABLE：一个事务在执行的过程中完全看不到其他事务对数据库所做的更新。

### Spring 管理事务的方式有几种？

1. 编程式事务，在代码中硬编码。(不推荐使用)
2. 声明式事务，在配置文件中配置（推荐使用）

**声明式事务又分为两种：**

1. 基于XML的声明式事务
2. 基于注解的声明式事务

### @Component 和 @Bean 的区别是什么？

1. 作用对象不同: `@Component` 注解作用于类，而`@Bean`注解作用于方法。
2. `@Component`通常是通过类路径扫描来自动侦测以及自动装配到Spring容器中（我们可以使用 `@ComponentScan` 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。`@Bean` 注解通常是我们在标有该注解的方法中定义产生这个 bean,`@Bean`告诉了Spring这是某个类的示例，当我需要用它的时候还给我。
3. `@Bean` 注解比 `Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册bean。比如当我们引用第三方库中的类需要装配到 `Spring`容器时，则只能通过 `@Bean`来实现。



### SpringMVC 工作原理了解吗?

**原理如下图所示：**
![SpringMVC运行原理](https://user-gold-cdn.xitu.io/2019/6/5/16b27eedc1634c3f?w=1015&h=466&f=png&s=48321)

上图的一个笔误的小问题：Spring MVC 的入口函数也就是前端控制器 `DispatcherServlet` 的作用是接收请求，响应结果。

**流程说明（重要）：**

1. 客户端（浏览器）发送请求，直接请求到 `DispatcherServlet`。
2. `DispatcherServlet` 根据请求信息调用 `HandlerMapping`，解析请求对应的 `Handler`。
3. 解析到对应的 `Handler`（也就是我们平常说的 `Controller` 控制器）后，开始由 `HandlerAdapter` 适配器处理。
4. `HandlerAdapter` 会根据 `Handler`来调用真正的处理器开处理请求，并处理相应的业务逻辑。
5. 处理器处理完业务后，会返回一个 `ModelAndView` 对象，`Model` 是返回的数据对象，`View` 是个逻辑上的 `View`。
6. `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`。
7. `DispaterServlet` 把返回的 `Model` 传给 `View`（视图渲染）。
8. 把 `View` 返回给请求者（浏览器）







































