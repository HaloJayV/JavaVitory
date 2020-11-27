[TOC]

# AOP（Aspect Oriented Programming）

* 面向切面编程 ，基于OOP基础上的编程思想，指程序运行时，将某段代码动态地切入到指定方法的指定位置进行运行的编程方式

* 1、目标对象 Target Object : 需要被代理的对象target。

  2、切面 (aspect )： 规则，一系列的方法统一的规则。

  3、切点 (PointCut)： 需要代理的具体方法，target的某一个方法。

  4、通知 Advice： 代理对象执行规则业务的方法。

* #### 应用场景：

  * 通过动态代理实现日志记录非常强大，而且与业务逻辑解耦，可以进行日志操作
  * 可以做权限判断，例如用JWT进入系统前，可以先用AOP拦截下来判断权限

* AOP底层是通过动态代理实现的，动态代理有JDK Proxy和CGLIB

  * JDK Proxy可以生成语言相同接口，需要实现类去实现一个接口才可以动态代理
  * 而CGLIB会使用一种ACM字节码编辑器，可以生成一个目标类的一个代理类作为Proxy子类，去实现方法增强的代理功能。性能上来说CGLIB在创建对象过程中速度慢，但在运行的时候效率更高

* JDK Proxy：` Object proxy = Proxy.newProxyInstance(loader,interfaces,h);  【动态代理对象】.getProxy(【被代理对象】)`

  * jdk默认的动态代理如果目标对象没有实现接口，是无法为他创建代理对象的。**因此Spring实现了AOP功能用来代替动态代理，底层就是动态代理**

  * ```java
    public static Calculator getProxy(final Calculator calculator) {
     		InvocationHandler h = new InvocationHandler() {
     			@Override
     			public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
     				Object result = null;
     				try{
     					LogUtils.logStart(method, args);
     					result = method.invoke(calculator, args); // 反射
     					LogUtils.logReturn(method, result);
     				} catch (Exception e) {
     					LogUtils.logException(method, e);
     				} finally{
     					LogUtils.logEnd(method);
     				}
     				return result; // 返回值必须返回
     			}
     		}; // 方法执行器
     		Class<?>[] interfaces = calculator.getClass().getInterfaces();//必须要有接口，唯一让代理对象proxy与被代理对象产生关联
     		ClassLoader loader = calculator.getClass().getClassLoader();
     		Object proxy = Proxy.newProxyInstance(loader, interfaces, h); // proxy.getClass() // com.sun.proxy.$Proxy2
     		return (Calculator)proxy;
     	}
    ```

* AOP日志：

  * ```java
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>   // 开启基于注解的AOP功能，aop的名称空间
    @Component   @Aspect // 日志上添加该注解表示开启AOP
    @Before("execution(方法全类名，带修饰符和返回值类型和参数)") // execution指定的方法执行前先执行该注解下的方法
    @AfterReturning 返回值返回时执行   	 @AfterThrowing：出现异常时执行	
    结束之后@After("execution(public int com.atguigu.impl.MyMathCalculator.*(int, int))")
    在使用时，先获取bean = ioc.getBean(必须是接口类型，不能是它本类)
    ```

* AOP细节1：IOC容器保存的是组件的代理对象

  * ##### 被代理对象的接口不需要加入到ioc中，只要它的实现类有注入到ioc即可，即接口一般不注入ioc中，虽然注入也不创建对象

  * 被代理对象通过 `bean=ioc.getBean(接口A.class)`获取ioc里的bean对象，而bean.getClass()则获取proxy代理对象，因为AOP底层就是动态代理，**容器中保存的组件是它的代理对象**，前提是被@Aspect的组件的动态方法运行切入。

  * 没有接口也可以创建代理对象，bean.getClass()输出的是本类类型, 底层是cglib帮助创建代理对象

  * 使用AOP日志时会自动创建代理对象，有接口转为接口类型，没接口转为本类类型

* AOP细节2：切入点表达式的写法

  * 格式：如@Before(excution(访问权限符 返回值类型 方法全类名(参数表)))  通配符：* 匹配一个或多个字符  .. 任意参数或多层路径

* AOP细节3：通知方法的执行顺序

  * @Before前置通知 =》 @After后置通知=》@AfterReturning正常返回 / @AfterThrowing 异常返回
  * 而@Around环绕通知则通过@Order(-1)设置优先级，参数 ProceedingJoinPoint的 point.proceed() 切入方法中。JoinPoint获取切入方法的信息

* AOP细节4,5：JoinPoint获取目标方法的信息，在@Aspect下的方法参数里加入JoinPoint便可获取该方法的信息

  * ```java
    @Pointcut("execution(public int com.atguigu.impl.类名.*(int, int))") // 定义切入方法，抽取切点表达式
    public void pointCut(){}; 
    
    @AfterThrowing(value="pointCut()",throwing="exception") // 直接写入方法名，也可以使用全类名作为value
    public static void logException(JoinPoint joinPoint, Exception exception) {
    	System.out.println("["+joinPoint.getSignature().getName()+"]方法执行异常,异常为：" + exception);
    }
    ```

* AOP细节6：方法的参数列表不能乱写，除了JoinPoint，其他参数需要在注解里添加该参数，Spring才能识别该参数

* AOP细节7：整合四大通知为环绕通知，

  * 环绕通知@Around，利用JoinPoint的子接口`Object proceed = ProceedingJoinPoint.proceed(args)`作用是推进目标方法的执行，利用反射调用目标方法，相当于`method.invoke(obj, args);`四合一就是环绕通知	

  * 在环绕通知中，顺序：@Before前置通知 =》@AfterReturning正常返回 / =》@AfterThrowing 异常返回=》@After后置通知

  * ```java
    @Around("pointCut()")  // ProceedingJoinPoint为JoinPoint子接口
       	public Object myAround(ProceedingJoinPoint pjp) throws Throwable{
       		Object[] args = pjp.getArgs();
       		String name = pjp.getSignature().getName();
       		Object proceed = null;
       		try {
       			System.out.println("1.【环绕前置通知】【"+name+"方法开始】");
       			// 就是利用反射调用目标方法，相当于method.invoke(obj, args)
       			proceed = pjp.proceed(); // 动态代理帮忙调用方法，即在这调用目标方法
       			System.out.println("2。【环绕返回通知】【"+name+"方法返回，返回值为：】" + proceed);
       		} catch (Exception e) {
       			System.out.println("【环绕异常通知】【"+name+"方法异常，异常为："+ e +"】");//消除掉异常通知
       			throw new RuntimeException(e);  // 重新将异常信息抛出，这样普通通知才能检测出异常
       		} finally {
       			System.out.println("3.【环绕后置通知】【"+name+"方法结束】");
       		}
       		return proceed; // 反射调用后的返回值也一定返回出去
       	}
    ```

* AOP细节9：环绕通知是优先于普通通知执行

  ![image-20200703165634907](../../../../Software/Typora/Picture/image-20200703165634907.png)

* AOP细节10：多切面运行顺序

  * 普通通知：按照切面类首字母顺序执行切面，先运行前置通知，最后才执行其他通知，@Order(x)：x越小优先级越高
  * 环绕通知：环绕通知优先于该切面的普通通知执行，但在其他切面下的顺序和普通通知一致，也是先进后出，但无法影响其他切面
  * 使用场景：AOP加日志保存到数据库、AOP做权限验证、AOP做安全检查、AOP做事务控制

* 基于XML配置的AOP：

  * 将目标类和切面类都加入到ioc容器中`<bean id="类命首字母小写" class="全类名"></beam>`

  * ```
    <aop:config>
    	<aop:aspect ref="切面类id" order="-1">   // 相当于加了@Aspect，优先级最高为-1
    		<aop:around method="切面类方法" pointcut-ref="目标类id"/>
    	</aop:aspect>
    </aop:config>
    ```

### JdbcTemplate

* ```java
  <context:property-placeholder localtation="classpath:dbconfig.properties">  <!--在xml中配置数据源，即导入配置文件-->
  <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
  	<property name="user" value="${jdbc.user}"></property>
  	<property name="password" value="${jdbc.password}"></property>
  	<property name="jdbcUrl" value="${jdbc.jdbcUrl}"></property>
  	<property name="driverClass" value="${jdbc.driverClass}"></property>
  </bean>
  ```

```java
  
JdbcTemplate jdbcTemplate = ioc.getBean(JdbcTemplate.class);  // 获取JdbcTemplate对象
修改：jdbcTemplate.update(sql, 参数)
String sql = "UPDATE employee SET name=? WHERE id=?";     // 修改
jdbcTemplate.update(sql, employee.getName(), employee.getId);
String sql = "INSERT INTO employee(name,salary) values(?,?)";     // 添加插入
jdbcTemplate.update(sql, employee.getName(), employee.getSalary);
```

* 批量插入（少用 ）

  ```java
    String sql = "INSERT INTO employee(emp_name, salary) VALUES(?, ?)";
  List<Object[]> batchArgs = new ArrayList<Object[]>();
    batchArgs.add(new Object[]{"张三",123.00});
    jdbcTemplate.batchUpdate(sql ,batchArgs);
  ```

    * 查询并返回java对象: `jdbcTemplate.queryForObject(sql语句, 返回值类型, 参数)`

  * ```
    String sql = "SELECT emp_id FROM employee WHERE emp_id=?";
    // queryForObject查询单个对象，query()查询多个对象，BeanPropertyRowMapper规定返回的Java对象类型
      jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<>(Employee.class), 5);
    ```

### 声明式事务：

* ```java
  try {  // AOP之前								
  	// 获取链接
  	// 设置非自动提交
  	chain.doFilter();
  	// 提交
  } catch (Exception e) {
  	// 回滚
  } finally{
  	// 关闭链接释放资源
  }
  ```

  <img src="../../../../Software/Typora/Picture/image-20200704130926076.png" alt="image-20200704130926076"  />                                                                     

  <img src="../../../../Software/Typora/Picture/image-20200704130941619.png" alt="image-20200704130941619"  />

* Mybatis使用DataSourceTransactionManager事务管理器

* 基于xml配置事务管理器： 

  ```java
  <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  	<property name="dataSource" ref="pooledDataSource"></property> <!-- 控制数据源 -->
  </bean>
  <tx:annotation-driven transaction-manager="transactionManager"/>    <!-- 开启基于注解的声明式事务：依赖tx名称空间 -->
  <!-- 在Service类的方法上标上注解 @Transactional 即可开启事务 -->
  ```


* 事务细节：

  ```
  @Transactional(xxx):
  isolation=Isolation:事务级别
  propagation=Propagation:事务的传播行为
  noRollbackFor=Class[] 和noRollbackFOrClassName-String[]：哪些异常事务可以不回滚
  rollbackFor=class[]和rollbackForClassName-String[]: 哪些异常事务需要回滚
  readOnly=true:设置事务位只读事务,可以进行事务优化，加快查询速度
  timeout=3: 超时：事务超出指定时长后自动终止并回滚，单位：秒
  ```

  * 异常分类：
    * 运行时异常（非检查异常）：可以不用处理，默认都回滚，如1/0异常
    * 编译时异常（受检查异常）：要么try-catch，要么声明throws，默认不回滚，如FileIOException
  * 数据库事务并发问题
    * 脏读：两次读取的过程中间有修改回滚，第一次读到的是无效的值，不允许发生
    * 不可重复读：两次读取的过程中间有修改提交，两次读取不一致，允许发生
    * 幻读；第二次读取比第一次读取表时多了一些行数据，允许发生
  * Spring事务隔离级别：@Transactional(isolation = Isolation.?)   
    * 读未提交：READ_UNCOMMITTED，出现脏读，不允许用
    * 读已提交：READ_COMMITTED，只能避免脏读

    * 可重复读：REPEATABLE_READ, 可以避免脏读和不可重复读，在mysql中还可以避免幻读。Mysql默认级别。在事务期间读到的数据始终不变
    * 串行化：SERIALIZABLE,  事务期间进入单线程模式，不允许进行修改操作，极少用到
  * 默认两个事务进行并发修改时，第二个事务会一直等待，直到第一个事务提交事务
  * 有事务的业务逻辑，容器中保存的是这个业务逻辑的cglib代理对象

* **事务的传播行为**： 如果有多个事务及逆行嵌套运行，子事务是否要和大事务共用一个事务。只需要掌握前两个

  * ![image-20200709231048024](../../../../Software/Typora/Picture/image-20200709231048024.png)

  * 如果REQUIRED，子事务的属性都是继承大事务的，而REQUIRED_NEW可以调整

* 如果在本Service类的方法开启事务，在事务中调用2个REQUIRED_NEW 修改子事务，若当前事务异常，则两个子事务都回滚，原因是都在当前Service类，即三个事务方法类型都是同一个代理对象，因此都是在同一个事务方法中

## AOP源码解析；

原文：https://www.cnblogs.com/liuyk-code/p/9886033.html

IOC的入口是ClassPathXmlApplicationContext.getBean()方法。

MVC 的入口是DispatcherServlet.doDispatch()方法。

AOP的入口是AbstractAutowireCapableBeanFactory.doCreateBean()方法。

AOP三个基本点：

1、Advice通知，即切入点具体要做的事情，分为BeforeAdvice、AfterAdvice以及ThrowsAdvice。

2、Pointcut：切点，@Pointcut(正则表达式)定义切入点的位置

3、Advisor: 通知器，

Spring的AOP采用的是动态代理模式，Spring在代理类中包裹切面，在运行期间把切面织入到Bean中，也就是说Spring用代理类封装了目标类，同时拦截了被通知方法的调用，处理完通知（Advice）后，再把调用转发给真正的目标Bean，因为是动态代理，所以Spring的AOP只支持到方法连接点而无法提供字段和构造器接入点（AspectJ和JBoss可以），所以Spring无法创建细粒度的通知。

根据Spring AOP的动态代理的过程，我们可以把AOP的设计分为两大块：第一，需要为目标对象建立代理对象；第二，需要启动代理对象的拦截器来完成各种横切面的织入

#### 生成代理对象

![image-20201110122615407](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201110122615407.png)

Spring的AspectJProxyFactory、ProxyFactoryBean和ProxyFactory封装了代理对象AopProxy的生成过程，代理对象的生成实现过程由JDK的Proxy和CGLIB第三方来实现。

在调用getBean方法->doGetBean方法中，我们可以看到无论是直接取单例的bean，还是创建单例、多例、自定义生命周期的bean，都会经过bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);这个方法，深入进去会发现最终会调用FactoryBean的getObject方法生产指定Bean的实例对象（Spring中具体的getObject的实现方法有70多个），也就是说对于FactoryBean而言，调用getBean(String BeanName)得到的是具体某个Bean对象（在getObject中具体实现的）而不是FactoryBean对象本身，如果想得到FactoryBean对象本身，只需要加上&符号即可。在FactoryBean的实现原理中我们可以发现工厂模式和装饰器模式的具体应用。

对于ProxyFactoryBean来说，生成代理对象，也是通过getObject方法封装（修饰）了对目标对象增加的增强处理

![image-20201110123255094](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201110123255094.png)

**如何拦截对目标对象方法的调用？**

对于JDK的代理对象，拦截使用的是InvocationHandler的invoke回调入口；对于CGLIB的代理对象，拦截是有设置好的回调callback方法（intercept方法）来完成的。首先会根据配置来判断拦截器是否与当前的调用方法相匹配（matches），如果当前的调用方法与配置的拦截器相匹配，那么相应的拦截器就开始发挥作用。这个过程是一个递归遍历的过程，它会遍历在代理对象中设置的拦截器链中所有的拦截器，被匹配的拦截器被逐一调用，直到所有的拦截器都被遍历完，才是对目标对象的方法调用，这样就完成了对目标对象方法调用的增强。

应该注意到，Advice通知不是直接对目标对象作用来完成增强的，而是对不同种类的通知通过AdviceAdapter适配器来实现的。





​	


### **一、准备工作**

 AOP：【动态代理】
          指在程序运行期间动态的将某段代码切入到指定方法指定位置进行运行的编程方式；

  1、导入aop模块；Spring AOP：(spring-aspects)
  2、定义一个业务逻辑类（MathCalculator）；在业务逻辑运行的时候将日志进行打印（方法之前、方法运行结束、方法出现异常，xxx）
  3、定义一个日志切面类（LogAspects）：切面类里面的方法需要动态感知MathCalculator.div运行到哪里然后执行；
          通知方法：
              前置通知(@Before)：logStart：在目标方法(div)运行之前运行
              后置通知(@After)：logEnd：在目标方法(div)运行结束之后运行（无论方法正常结束还是异常结束）
              返回通知(@AfterReturning)：logReturn：在目标方法(div)正常返回之后运行
              异常通知(@AfterThrowing)：logException：在目标方法(div)出现异常以后运行
              环绕通知(@Around)：动态代理，手动推进目标方法运行（joinPoint.procced()）
  4、给切面类的目标方法标注何时何地运行（通知注解）；
  5、将切面类和业务逻辑类（目标方法所在类）都加入到容器中;
  6、必须告诉Spring哪个类是切面类(给切面类上加一个注解：@Aspect)
  7、给配置类中加 @EnableAspectJAutoProxy 【开启基于注解的aop模式】
  三步：
      1）、将业务逻辑组件和切面类都加入到容器中；告诉Spring哪个是切面类（@Aspect）
      2）、在切面类上的每一个通知方法上标注通知注解，告诉Spring何时何地运行（切入点表达式）
      3）、开启基于注解的aop模式；@EnableAspectJAutoProxy

* 定义一个切面（包含目标方法）：

```java
public class MathCalculator {
	// 目标方法
    public int div(int i,int j){
        System.out.println("MathCalculator...div...");
        return i/j;    
    }
}
```

* 定义一个切面类 (通知器Advicer)：

```java
/**
 * 切面类
 * @author lfy
 * @Aspect： 告诉Spring当前类是一个切面类
 */
@Aspect
public class LogAspects {
    //抽取公共的切入点表达式
    //1、本类引用
    //2、其他的切面引用
    @Pointcut("execution(public int com.kun.aop.MathCalculator.*(..))")
    public void pointCut(){};
    
    //@Before在目标方法（切入点MathCalculator）之前切入；切入点表达式（指定在哪个方法切入）
    @Before("pointCut()")
    public void logStart(JoinPoint joinPoint){
        Object[] args = joinPoint.getArgs();
        System.out.println(""+joinPoint.getSignature().getName()+"运行。。。@Before:参数列表是：{"+Arrays.asList(args)+"}");
    }
    
    @After("com.kun.aop.LogAspects.pointCut()")
    public void logEnd(JoinPoint joinPoint){
        System.out.println(""+joinPoint.getSignature().getName()+"结束。。。@After");
    }
    
    //JoinPoint一定要出现在参数表的第一位
    @AfterReturning(value="pointCut()",returning="result")
    public void logReturn(JoinPoint joinPoint,Object result){
        System.out.println(""+joinPoint.getSignature().getName()+"正常返回。。。@AfterReturning:运行结果：{"+result+"}");
    }
    
    @AfterThrowing(value="pointCut()",throwing="exception")
    public void logException(JoinPoint joinPoint,Exception exception){
        System.out.println(""+joinPoint.getSignature().getName()+"异常。。。异常信息：{"+exception+"}");
    }
}
```

　　接下来是一个aop的配置：

```java
@EnableAspectJAutoProxy
@Configuration
public class MainConfigOfAOP {
     
    //业务逻辑类加入容器中
    @Bean
    public MathCalculator calculator(){
        return new MathCalculator();
    }

    //切面类加入到容器中
    @Bean
    public LogAspects logAspects(){
        return new LogAspects();
    }
}
```

### **二、从@EnableAspectJAutoProxy看起**

　　查看一下@EnableAspectJAutoProxy的定义：

```java
@Target(ElementType.TYPE) 
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AspectJAutoProxyRegistrar.class)  // 导入组件AspectJAutoProxyRegistrar
public @interface EnableAspectJAutoProxy {
    /**
     * Indicate whether subclass-based (CGLIB) proxies are to be created as opposed
     * to standard Java interface-based proxies. The default is {@code false}.
     */
    boolean proxyTargetClass() default false;
}
```

我们发现它导入了一个AspectJAutoProxyRegistrar组件，进一步查看其代码：

```java
class AspectJAutoProxyRegistrar implements ImportBeanDefinitionRegistrar {

    public void registerBeanDefinitions(
            AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

        AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);

        AnnotationAttributes enableAJAutoProxy =
                attributesFor(importingClassMetadata, EnableAspectJAutoProxy.class);
        if (enableAJAutoProxy.getBoolean("proxyTargetClass")) {
            AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
        }
    }
}
```

我们发现它实现了ImportBeanDefinitionRegistrar接口，这个接口可以向IOC容器中注册bean。 由此可以推测: 

**aop利用 AspectJAutoProxyRegistrar 自定义给容器中注册bean；BeanDefinition。**通过断点我们发现

![img](../../../../Software/Typora/Picture/1140836-20181031200151572-1248357045.png)

IOC容器中注入了一个 `internalAutoProxyCreator = AnnotationAwareAspectJAutoProxyCreator` 的bean，到此可以得出结论，**@EnableAspectJAutoProxy** 给容器中注册一个 **AnnotationAwareAspectJAutoProxyCreator**，即注解切面自动动态代理创建器

### **三、AnnotationAwareAspectJAutoProxyCreator创建过程**

　　首先查看类图：![img](../../../../Software/Typora/Picture/1140836-20181031201123887-743732675.png)

　　在此需要关注两点内容：

1）关注后置处理器SmartInstantiationAwareBeanPostProcessor（在bean初始化完成前后做事情）

2）关注自动装配BeanFactory。

　　通过代码查看，发现父类AbstractAutoProxyCreator中有后置处理器的内容；AbstactAdvisorAutoProxyCreator类中重写了其父类AbstractAutoProxyCreator中setBeanFactory()方法，在AnnotationAwareAspectJAutoProxyCreator类中initBeanFactory()方法完成了自动装配BeanFactory。分别在这两处关注点打断点来查看其流程：

```java
/**
     * Instantiate and invoke all registered BeanPostProcessor beans,
     * respecting explicit order if given.
     * <p>Must be called before any instantiation of application beans.
     */
    protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory) {
        String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);

        // Register BeanPostProcessorChecker that logs an info message when
        // a bean is created during BeanPostProcessor instantiation, i.e. when
        // a bean is not eligible for getting processed by all BeanPostProcessors.
        int beanProcessorTargetCount = beanFactory.getBeanPostProcessorCount() + 1 + postProcessorNames.length;
        beanFactory.addBeanPostProcessor(new BeanPostProcessorChecker(beanFactory, beanProcessorTargetCount));

        // Separate between BeanPostProcessors that implement PriorityOrdered,
        // Ordered, and the rest.
        List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList<BeanPostProcessor>();
        List<BeanPostProcessor> internalPostProcessors = new ArrayList<BeanPostProcessor>();
        List<String> orderedPostProcessorNames = new ArrayList<String>();
        List<String> nonOrderedPostProcessorNames = new ArrayList<String>();
        for (String ppName : postProcessorNames) {
            if (isTypeMatch(ppName, PriorityOrdered.class)) {
                BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
                priorityOrderedPostProcessors.add(pp);
                if (pp instanceof MergedBeanDefinitionPostProcessor) {
                    internalPostProcessors.add(pp);
                }
            }
            else if (isTypeMatch(ppName, Ordered.class)) {
                orderedPostProcessorNames.add(ppName);
            }
            else {
                nonOrderedPostProcessorNames.add(ppName);
            }
        }

        // First, register the BeanPostProcessors that implement PriorityOrdered.
        OrderComparator.sort(priorityOrderedPostProcessors);
        registerBeanPostProcessors(beanFactory, priorityOrderedPostProcessors);

        // Next, register the BeanPostProcessors that implement Ordered.
        List<BeanPostProcessor> orderedPostProcessors = new ArrayList<BeanPostProcessor>();
        for (String ppName : orderedPostProcessorNames) {
            BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
            orderedPostProcessors.add(pp);
            if (pp instanceof MergedBeanDefinitionPostProcessor) {
                internalPostProcessors.add(pp);
            }
        }
        OrderComparator.sort(orderedPostProcessors);
        registerBeanPostProcessors(beanFactory, orderedPostProcessors);

        // Now, register all regular BeanPostProcessors.
        List<BeanPostProcessor> nonOrderedPostProcessors = new ArrayList<BeanPostProcessor>();
        for (String ppName : nonOrderedPostProcessorNames) {
            BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
            nonOrderedPostProcessors.add(pp);
            if (pp instanceof MergedBeanDefinitionPostProcessor) {
                internalPostProcessors.add(pp);
            }
        }
        registerBeanPostProcessors(beanFactory, nonOrderedPostProcessors);
        // Finally, re-register all internal BeanPostProcessors.
        OrderComparator.sort(internalPostProcessors);
        registerBeanPostProcessors(beanFactory, internalPostProcessors);
        beanFactory.addBeanPostProcessor(new ApplicationListenerDetector());
    }
```

![img](../../../../Software/Typora/Picture/1140836-20181031204352324-1008707909.png)

![img](../../../../Software/Typora/Picture/1140836-20181031205212920-2020772773.png)

　　总结如下：

```
		  1）、传入配置类，创建ioc容器
          2）、注册配置类，调用refresh（）刷新容器；
          3）、registerBeanPostProcessors(beanFactory);注册bean的后置处理器来方便拦截bean的创建；
              1）、先获取ioc容器已经定义了的需要创建对象的所有BeanPostProcessor
              2）、给容器中加别的BeanPostProcessor
              3）、优先注册实现了PriorityOrdered接口的BeanPostProcessor；
              4）、再给容器中注册实现了Ordered接口的BeanPostProcessor；
              5）、注册没实现优先级接口的BeanPostProcessor；
              6）、注册BeanPostProcessor，实际上就是创建BeanPostProcessor对象，保存在容器中；
                  创建internalAutoProxyCreator的BeanPostProcessor【AnnotationAwareAspectJAutoProxyCreator】
                  1）、创建Bean的实例
                  2）、populateBean；给bean的各种属性赋值
                  3）、initializeBean：初始化bean；
                          1）、invokeAwareMethods()：处理Aware接口的方法回调
                          2）、applyBeanPostProcessorsBeforeInitialization()：应用后置处理器的postProcessBeforeInitialization（）
                          3）、invokeInitMethods()；执行自定义的初始化方法
                          4）、applyBeanPostProcessorsAfterInitialization()；执行后置处理器的postProcessAfterInitialization（）；
                  4）、BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)创建成功；--》aspectJAdvisorsBuilder
              7）、把BeanPostProcessor注册到BeanFactory中；
                  beanFactory.addBeanPostProcessor(postProcessor);  
=======以上是创建和注册AnnotationAwareAspectJAutoProxyCreator的过程========
```

###  **四、AnnotationAwareAspectJAutoProxyCreator的执行时机**

　　通过以上步骤我们发现在IOC容器启动时候，会通过一个@EnableAspectJAutoProxy注解注入AnnotationAwareAspectJAutoProxyCreator对象，并分析了该对象在IOC容器的启动时进行创建的过程。接下来我们重点来分析一下AnnotationAwareAspectJProxyCreator对象执行的时机。

　　之前分析到AnnotationAwareAspectJAutoProxyCreator是一个后置处理器，可以猜测它在其他bean的初始化前后进行了特殊处理。我在它父类的postProcessBeforeInstantiation方法进行了断点调试，其方法调用栈如下：![img](../../../../Software/Typora/Picture/1140836-20181104194632432-949327847.png)

　　通过对方法栈中源码的简单查看，我继续完善了流程：

```
		流程：
 　        1）、传入配置类，创建ioc容器
          2）、注册配置类，调用refresh（）刷新容器；
          3）、registerBeanPostProcessors(beanFactory);注册bean的后置处理器来方便拦截bean的创建；
              1）、先获取ioc容器已经定义了的需要创建对象的所有BeanPostProcessor
              2）、给容器中加别的BeanPostProcessor
              3）、优先注册实现了PriorityOrdered接口的BeanPostProcessor；
              4）、再给容器中注册实现了Ordered接口的BeanPostProcessor；
              5）、注册没实现优先级接口的BeanPostProcessor；
              6）、注册BeanPostProcessor，实际上就是创建BeanPostProcessor对象，保存在容器中；
                  创建internalAutoProxyCreator的BeanPostProcessor【AnnotationAwareAspectJAutoProxyCreator】
                  1）、创建Bean的实例
                  2）、populateBean；给bean的各种属性赋值
                  3）、initializeBean：初始化bean；
                          1）、invokeAwareMethods()：处理Aware接口的方法回调
                          2）、applyBeanPostProcessorsBeforeInitialization()：应用后置处理器的postProcessBeforeInitialization（）
                          3）、invokeInitMethods()；执行自定义的初始化方法
                          4）、applyBeanPostProcessorsAfterInitialization()；执行后置处理器的postProcessAfterInitialization（）；
                  4）、BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)创建成功；--》aspectJAdvisorsBuilder
              7）、把BeanPostProcessor注册到BeanFactory中；
                  beanFactory.addBeanPostProcessor(postProcessor);
  =======以上是创建和注册AnnotationAwareAspectJAutoProxyCreator的过程========
  
              AnnotationAwareAspectJAutoProxyCreator => InstantiationAwareBeanPostProcessor
          4）、finishBeanFactoryInitialization(beanFactory);完成BeanFactory初始化工作；创建剩下的单实例bean
              1）、遍历获取容器中所有的Bean，依次创建对象getBean(beanName);
                  getBean->doGetBean()->getSingleton()->
              2）、创建bean
                  【AnnotationAwareAspectJAutoProxyCreator在所有bean创建之前会有一个拦截，InstantiationAwareBeanPostProcessor，　　　　　　　　　　　　会调用postProcessBeforeInstantiation()】
                  1）、先从缓存中获取当前bean，如果能获取到，说明bean是之前被创建过的，直接使用，否则再创建；
                      只要创建好的Bean都会被缓存起来
                  2）、createBean（）;创建bean；
                      AnnotationAwareAspectJAutoProxyCreator 会在任何bean创建之前先尝试返回bean的实例
                      【BeanPostProcessor是在Bean对象创建完成初始化前后调用的】
                      【InstantiationAwareBeanPostProcessor是在创建Bean实例之前先尝试用后置处理器返回对象的】
                      1）、resolveBeforeInstantiation(beanName, mbdToUse);解析BeforeInstantiation
                          希望后置处理器在此能返回一个代理对象；如果能返回代理对象就使用，如果不能就继续
                          1）、后置处理器先尝试返回对象；
                              bean = applyBeanPostProcessorsBeforeInstantiation（）：
                                  拿到所有后置处理器，如果是InstantiationAwareBeanPostProcessor;
                                  就执行postProcessBeforeInstantiation
                              if (bean != null) {
                                bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
                            }
  
                      2）、doCreateBean(beanName, mbdToUse, args);真正的去创建一个bean实例；和3.6流程一样；
                      3）、
```



### **五、创建AOP代理**

* 如果被代理对象不是接口或者未继承接口，则是通过 cglib 创建代理对象，否则(基于接口生成代理对象)是通过 Java动态代理JDK Proxy创建该代理对象

* 在DefaultSingletonBeanRegistry 的 addSingleton方法里，this.singletonObjects.put(beanName, singletonObject); 底层创建代理对象

  * 进入put方法来到AbstractBeanFactory的put 方法=>AbstractBeanFactory 的 doCreateBean 方法，该方法里的initializeBean方法创建代理对象

  * 进入到initializeBean的applyBeanPostProcessorsAfterInitialization 方法，其方法里

  * 最后在DefaultAopProxyFactory的createAopProxy方法里找到 ：return new ObjectsisCglibAopProxy(config);// 创建代理对象

    如果是被代理对象是接口则通过Java动态代理JDK Proxy，则 return new JdkDynamicAopProxy(config);

    <img src="../../../../Software/Typora/Picture/image-20200711154648912.png" alt="image-20200711154648912" style="zoom:80%;" />

* 在DefaultSingletonBeanRegistry 的 getSingleton方法里，this.singletonObjects.get(beanName); 底层获取代理对象，是一个concurrentHashMap

　　上一步我发现AnnotationAwareAspectJAutoProxyCreator在所有bean创建时进行了拦截，执行其中的postProcessBeforeInstantiation方法，接下来我们继续通过断点调试查看该方法的进行的操作。

![img](../../../../Software/Typora/Picture/1140836-20181104200556508-982467155.png)

　　首先方法内部会进行一系列的判断，判    断当前bean是否在advisedBeans中（保存了所有需要增强bean）、判断当前bean是否是基础类型的Advice、Pointcut、Advisor、AopInfrastructureBean、判断是否是切面（是否实现了注解@Aspect）、判断是否需要跳过等。

![img](../../../../Software/Typora/Picture/1140836-20181104201043850-1949023986.png)

　　在判断的过程中会拿到增强bean的相关的通知方法，并通过这些切面进行逻辑判断。

　　执行完postProcessBeforeInstantiation方法进行通知方法的判断后，执行postProcessAfterInitialization方法。

![img](../../../../Software/Typora/Picture/1140836-20181104201705168-1205976536.png)

　　我们发现postProcessAfterInitialization方法会对切面进行一次包装的处理。

![img](../../../../Software/Typora/Picture/1140836-20181104201925094-1913383467.png)

　　在对对象包装的过程中创建了一个代理对象。

![img](../../../../Software/Typora/Picture/1140836-20181104203025925-1076514182.png)

 

![img](../../../../Software/Typora/Picture/1140836-20181104202505899-649621902.png)

　　我们细看创建代理对象的过程，发现在创建之前首先会根据切入点表达式对增强器进行一一匹配，最终拿到所有的增强器。

```
 protected Object createProxy(Class<?> beanClass, String beanName, Object[] specificInterceptors, TargetSource targetSource) {
        if (this.beanFactory instanceof ConfigurableListableBeanFactory) {
            AutoProxyUtils.exposeTargetClass((ConfigurableListableBeanFactory)this.beanFactory, beanName, beanClass);
        }

        ProxyFactory proxyFactory = new ProxyFactory();
        proxyFactory.copyFrom(this);
        if (!proxyFactory.isProxyTargetClass()) {
            if (this.shouldProxyTargetClass(beanClass, beanName)) {
                proxyFactory.setProxyTargetClass(true);
            } else {
                this.evaluateProxyInterfaces(beanClass, proxyFactory);
            }
        }

        Advisor[] advisors = this.buildAdvisors(beanName, specificInterceptors);
        proxyFactory.addAdvisors(advisors);
        proxyFactory.setTargetSource(targetSource);
        this.customizeProxyFactory(proxyFactory);
        proxyFactory.setFrozen(this.freezeProxy);
        if (this.advisorsPreFiltered()) {
            proxyFactory.setPreFiltered(true);
        }

        return proxyFactory.getProxy(this.getProxyClassLoader());
    }
```

　　创建代理对象过程中，会先创建一个代理工厂，获取到所有的增强器（通知方法），将这些增强器和目标类注入代理工厂，再用代理工厂创建对象。

```
public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
        if (!config.isOptimize() && !config.isProxyTargetClass() && !this.hasNoUserSuppliedProxyInterfaces(config)) {
            return new JdkDynamicAopProxy(config);
        } else {
            Class<?> targetClass = config.getTargetClass();
            if (targetClass == null) {
                throw new AopConfigException("TargetSource cannot determine target class: Either an interface or a target is required for proxy creation.");
            } else {
                return (AopProxy)(!targetClass.isInterface() && !Proxy.isProxyClass(targetClass) ? new ObjenesisCglibAopProxy(config) : new JdkDynamicAopProxy(config));
            }
        }
    }
```

　　代理工厂会选择JdkDynamicAopProxy或者CglibAopProxy，主要通过是否接口和是否配置cglib代理来选择。最终工厂会创建一个代理增强的对象。我们继续完善之前的流程.。

```
流程：
 　        1）、传入配置类，创建ioc容器
          2）、注册配置类，调用refresh（）刷新容器；
          3）、registerBeanPostProcessors(beanFactory);注册bean的后置处理器来方便拦截bean的创建；
              1）、先获取ioc容器已经定义了的需要创建对象的所有BeanPostProcessor
              2）、给容器中加别的BeanPostProcessor
              3）、优先注册实现了PriorityOrdered接口的BeanPostProcessor；
              4）、再给容器中注册实现了Ordered接口的BeanPostProcessor；
              5）、注册没实现优先级接口的BeanPostProcessor；
              6）、注册BeanPostProcessor，实际上就是创建BeanPostProcessor对象，保存在容器中；
                  创建internalAutoProxyCreator的BeanPostProcessor【AnnotationAwareAspectJAutoProxyCreator】
                  1）、创建Bean的实例
                  2）、populateBean；给bean的各种属性赋值
                  3）、initializeBean：初始化bean；
                          1）、invokeAwareMethods()：处理Aware接口的方法回调
                          2）、applyBeanPostProcessorsBeforeInitialization()：应用后置处理器的postProcessBeforeInitialization（）
                          3）、invokeInitMethods()；执行自定义的初始化方法
                          4）、applyBeanPostProcessorsAfterInitialization()；执行后置处理器的postProcessAfterInitialization（）；
                  4）、BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)创建成功；--》aspectJAdvisorsBuilder
              7）、把BeanPostProcessor注册到BeanFactory中；
                  beanFactory.addBeanPostProcessor(postProcessor);
  =======以上是创建和注册AnnotationAwareAspectJAutoProxyCreator的过程========
  
              AnnotationAwareAspectJAutoProxyCreator => InstantiationAwareBeanPostProcessor
          4）、finishBeanFactoryInitialization(beanFactory);完成BeanFactory初始化工作；创建剩下的单实例bean
              1）、遍历获取容器中所有的Bean，依次创建对象getBean(beanName);
                  getBean->doGetBean()->getSingleton()->
              2）、创建bean
                  【AnnotationAwareAspectJAutoProxyCreator在所有bean创建之前会有一个拦截，InstantiationAwareBeanPostProcessor，
　　　　　　　　　　　　会调用postProcessBeforeInstantiation()】
                  1）、先从缓存中获取当前bean，如果能获取到，说明bean是之前被创建过的，直接使用，否则再创建；
                      只要创建好的Bean都会被缓存起来
                  2）、createBean（）;创建bean；
                      AnnotationAwareAspectJAutoProxyCreator 会在任何bean创建之前先尝试返回bean的实例
                      【BeanPostProcessor是在Bean对象创建完成初始化前后调用的】
                      【InstantiationAwareBeanPostProcessor是在创建Bean实例之前先尝试用后置处理器返回对象的】
                      1）、resolveBeforeInstantiation(beanName, mbdToUse);解析BeforeInstantiation
                          希望后置处理器在此能返回一个代理对象；如果能返回代理对象就使用，如果不能就继续
                          1）、后置处理器先尝试返回对象；
                              bean = applyBeanPostProcessorsBeforeInstantiation（）：
                                  拿到所有后置处理器，如果是InstantiationAwareBeanPostProcessor;
                                  就执行postProcessBeforeInstantiation
                              if (bean != null) {
                                bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
                            }
  
                      2）、doCreateBean(beanName, mbdToUse, args);真正的去创建一个bean实例；和3.6流程一样；
                      3）、
AnnotationAwareAspectJAutoProxyCreator【InstantiationAwareBeanPostProcessor】    的作用：
 1）、每一个bean创建之前，调用postProcessBeforeInstantiation()；
          关心MathCalculator和LogAspect的创建
          1）、判断当前bean是否在advisedBeans中（保存了所有需要增强bean）
          2）、判断当前bean是否是基础类型的Advice、Pointcut、Advisor、AopInfrastructureBean，
              或者是否是切面（@Aspect）
          3）、是否需要跳过
              1）、获取候选的增强器（切面里面的通知方法）【List<Advisor> candidateAdvisors】
                  每一个封装的通知方法的增强器是 InstantiationModelAwarePointcutAdvisor；
                  判断每一个增强器是否是 AspectJPointcutAdvisor 类型的；返回true
              2）、永远返回false
 2）、创建对象
  postProcessAfterInitialization；
          return wrapIfNecessary(bean, beanName, cacheKey);//包装如果需要的情况下
          1）、获取当前bean的所有增强器（通知方法）  Object[]  specificInterceptors
             1、找到候选的所有的增强器（找哪些通知方法是需要切入当前bean方法的）
             2、获取到能在bean使用的增强器。
             3、给增强器排序
         2）、保存当前bean在advisedBeans中；
         3）、如果当前bean需要增强，创建当前bean的代理对象；
             1）、获取所有增强器（通知方法）
             2）、保存到proxyFactory
             3）、创建代理对象：Spring自动决定
                 JdkDynamicAopProxy(config);jdk动态代理；
                 ObjenesisCglibAopProxy(config);cglib的动态代理；
         4）、给容器中返回当前组件使用cglib增强了的代理对象；
         5）、以后容器中获取到的就是这个组件的代理对象，执行目标方法的时候，代理对象就会执行通知方法的流程；         
```

### **六、获取拦截器链**

　　上一步分析了目标方法被代理并创建的过程，接下来我们分析目标方法被拦截并执行的过程。

![img](../../../../Software/Typora/Picture/1140836-20181104210757487-747483612.png)

　　因为要查看目标方法的执行过程，所以我继续在目标方法上进行断点调试。通过对MathCalculator查看，可以发现它从IOC容器中取出已经是一个cglib代理对象了，其中包含增强方法和目标对象的一些详细信息。

　　![img](../../../../Software/Typora/Picture/1140836-20181104211556922-1272092409.png)

　　紧接着断点进入CglibAopProxy.intercept()拦截器，其中会获取即将执行的目标方法的拦截器链。

```java
public List<Object> getInterceptorsAndDynamicInterceptionAdvice(
            Advised config, Method method, Class targetClass) {

        // This is somewhat tricky... we have to process introductions first,
        // but we need to preserve order in the ultimate list.
        List<Object> interceptorList = new ArrayList<Object>(config.getAdvisors().length);
        boolean hasIntroductions = hasMatchingIntroductions(config, targetClass);
        AdvisorAdapterRegistry registry = GlobalAdvisorAdapterRegistry.getInstance();
        for (Advisor advisor : config.getAdvisors()) {
            if (advisor instanceof PointcutAdvisor) {
                // Add it conditionally.
                PointcutAdvisor pointcutAdvisor = (PointcutAdvisor) advisor;
                if (config.isPreFiltered() || pointcutAdvisor.getPointcut().getClassFilter().matches(targetClass)) {
                    MethodInterceptor[] interceptors = registry.getInterceptors(advisor);
                    MethodMatcher mm = pointcutAdvisor.getPointcut().getMethodMatcher();
                    if (MethodMatchers.matches(mm, method, targetClass, hasIntroductions)) {
                        if (mm.isRuntime()) {
                            // Creating a new object instance in the getInterceptors() method
                            // isn't a problem as we normally cache created chains.
                            for (MethodInterceptor interceptor : interceptors) {
                                interceptorList.add(new InterceptorAndDynamicMethodMatcher(interceptor, mm));
                            }
                        }
                        else {
                            interceptorList.addAll(Arrays.asList(interceptors));
                        }
                    }
                }
            }
            else if (advisor instanceof IntroductionAdvisor) {
                IntroductionAdvisor ia = (IntroductionAdvisor) advisor;
                if (config.isPreFiltered() || ia.getClassFilter().matches(targetClass)) {
                    Interceptor[] interceptors = registry.getInterceptors(advisor);
                    interceptorList.addAll(Arrays.asList(interceptors));
                }
            }
            else {
                Interceptor[] interceptors = registry.getInterceptors(advisor);
                interceptorList.addAll(Arrays.asList(interceptors));
            }
        }
        return interceptorList;
    }
```

　　通过registry.getInterceptors(advisor)方法获取所有的增强器，并将增强器转为List<MethodInterceptor>，最终返回。

```
public Object proceed() throws Throwable {
        if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size() - 1) {
            return this.invokeJoinpoint();
        } else {
            Object interceptorOrInterceptionAdvice = this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
            if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher) {
                InterceptorAndDynamicMethodMatcher dm = (InterceptorAndDynamicMethodMatcher)interceptorOrInterceptionAdvice;
                return dm.methodMatcher.matches(this.method, this.targetClass, this.arguments) ? dm.interceptor.invoke(this) : this.proceed();
            } else {
                return ((MethodInterceptor)interceptorOrInterceptionAdvice).invoke(this);
            }
        }
    }
```

　　之后，会将拦截器链和目标对象等传入methodInvocation，并调用proceed()方法。该方法执行也是拦截器的触发过程，也是目标方法的主要执行过程。

　　![img](../../../../Software/Typora/Picture/1140836-20181104214738637-382717242.png)

　　通过索引遍历拦截器链中的所有的拦截器（封装的增强器），并分别执行增强方法。每次执行拦截器方法索引自增，直至所有的增强方法执行完毕。

![img](../../../../Software/Typora/Picture/1140836-20181104220844941-633387789.png)

　　源码中有一个ExposeInvocationInterceptor对象会将`MethodInvocation放入到ThreadLocal进行线程共享，查看相关资料说方便同一线程中其他地方获取通知方法，具体哪里获得，我暂时没有找到，在后续工作学习中继续深入查看探究吧。`

　　完善流程如下：

```
流程：
 　        1）、传入配置类，创建ioc容器
          2）、注册配置类，调用refresh（）刷新容器；
          3）、registerBeanPostProcessors(beanFactory);注册bean的后置处理器来方便拦截bean的创建；
              1）、先获取ioc容器已经定义了的需要创建对象的所有BeanPostProcessor
              2）、给容器中加别的BeanPostProcessor
              3）、优先注册实现了PriorityOrdered接口的BeanPostProcessor；
              4）、再给容器中注册实现了Ordered接口的BeanPostProcessor；
              5）、注册没实现优先级接口的BeanPostProcessor；
              6）、注册BeanPostProcessor，实际上就是创建BeanPostProcessor对象，保存在容器中；
                  创建internalAutoProxyCreator的BeanPostProcessor【AnnotationAwareAspectJAutoProxyCreator】
                  1）、创建Bean的实例
                  2）、populateBean；给bean的各种属性赋值
                  3）、initializeBean：初始化bean；
                          1）、invokeAwareMethods()：处理Aware接口的方法回调
                          2）、applyBeanPostProcessorsBeforeInitialization()：应用后置处理器的postProcessBeforeInitialization（）
                          3）、invokeInitMethods()；执行自定义的初始化方法
                          4）、applyBeanPostProcessorsAfterInitialization()；执行后置处理器的postProcessAfterInitialization（）；
                  4）、BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)创建成功；--》aspectJAdvisorsBuilder
              7）、把BeanPostProcessor注册到BeanFactory中；
                  beanFactory.addBeanPostProcessor(postProcessor);
  =======以上是创建和注册AnnotationAwareAspectJAutoProxyCreator的过程========
  
              AnnotationAwareAspectJAutoProxyCreator => InstantiationAwareBeanPostProcessor
          4）、finishBeanFactoryInitialization(beanFactory);完成BeanFactory初始化工作；创建剩下的单实例bean
              1）、遍历获取容器中所有的Bean，依次创建对象getBean(beanName);
                  getBean->doGetBean()->getSingleton()->
              2）、创建bean
                  【AnnotationAwareAspectJAutoProxyCreator在所有bean创建之前会有一个拦截，InstantiationAwareBeanPostProcessor，
　　　　　　　　　　　　会调用postProcessBeforeInstantiation()】
                  1）、先从缓存中获取当前bean，如果能获取到，说明bean是之前被创建过的，直接使用，否则再创建；
                      只要创建好的Bean都会被缓存起来
                  2）、createBean（）;创建bean；
                      AnnotationAwareAspectJAutoProxyCreator 会在任何bean创建之前先尝试返回bean的实例
                      【BeanPostProcessor是在Bean对象创建完成初始化前后调用的】
                      【InstantiationAwareBeanPostProcessor是在创建Bean实例之前先尝试用后置处理器返回对象的】
                      1）、resolveBeforeInstantiation(beanName, mbdToUse);解析BeforeInstantiation
                          希望后置处理器在此能返回一个代理对象；如果能返回代理对象就使用，如果不能就继续
                          1）、后置处理器先尝试返回对象；
                              bean = applyBeanPostProcessorsBeforeInstantiation（）：
                                  拿到所有后置处理器，如果是InstantiationAwareBeanPostProcessor;
                                  就执行postProcessBeforeInstantiation
                              if (bean != null) {
                                bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
                            }
  
                      2）、doCreateBean(beanName, mbdToUse, args);真正的去创建一个bean实例；和3.6流程一样；
                      3）、
AnnotationAwareAspectJAutoProxyCreator【InstantiationAwareBeanPostProcessor】    的作用：
 1）、每一个bean创建之前，调用postProcessBeforeInstantiation()；
          关心MathCalculator和LogAspect的创建
          1）、判断当前bean是否在advisedBeans中（保存了所有需要增强bean）
          2）、判断当前bean是否是基础类型的Advice、Pointcut、Advisor、AopInfrastructureBean，
              或者是否是切面（@Aspect）
          3）、是否需要跳过
              1）、获取候选的增强器（切面里面的通知方法）【List<Advisor> candidateAdvisors】
                  每一个封装的通知方法的增强器是 InstantiationModelAwarePointcutAdvisor；
                  判断每一个增强器是否是 AspectJPointcutAdvisor 类型的；返回true
              2）、永远返回false
 2）、创建对象
  postProcessAfterInitialization；
          return wrapIfNecessary(bean, beanName, cacheKey);//包装如果需要的情况下
          1）、获取当前bean的所有增强器（通知方法）  Object[]  specificInterceptors
             1、找到候选的所有的增强器（找哪些通知方法是需要切入当前bean方法的）
             2、获取到能在bean使用的增强器。
             3、给增强器排序
         2）、保存当前bean在advisedBeans中；
         3）、如果当前bean需要增强，创建当前bean的代理对象；
             1）、获取所有增强器（通知方法）
             2）、保存到proxyFactory
             3）、创建代理对象：Spring自动决定
                 JdkDynamicAopProxy(config);jdk动态代理；
                 ObjenesisCglibAopProxy(config);cglib的动态代理；
         4）、给容器中返回当前组件使用cglib增强了的代理对象；
         5）、以后容器中获取到的就是这个组件的代理对象，执行目标方法的时候，代理对象就会执行通知方法的流程；       
  3）、目标方法执行；容器中保存了组件的代理对象（cglib增强后的对象），这个对象里面保存了详细信息（比如增强器，目标对象，xxx）；
         1）、CglibAopProxy.intercept();拦截目标方法的执行
         2）、根据ProxyFactory对象获取将要执行的目标方法拦截器链；
             List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
             1）、List<Object> interceptorList保存所有拦截器 5
                 一个默认的ExposeInvocationInterceptor 和 4个增强器；
             2）、遍历所有的增强器，将其转为Interceptor；
                 registry.getInterceptors(advisor);
             3）、将增强器转为List<MethodInterceptor>；
                 如果是MethodInterceptor，直接加入到集合中
                 如果不是，使用AdvisorAdapter将增强器转为MethodInterceptor；
                 转换完成返回MethodInterceptor数组；
		3）、如果没有拦截器链，直接执行目标方法;
        	拦截器链（每一个通知方法又被包装为方法拦截器，利用MethodInterceptor机制）
		4）、如果有拦截器链，把需要执行的目标对象，目标方法，
            拦截器链等信息传入创建一个 CglibMethodInvocation 对象，
            并调用 Object retVal =  mi.proceed();
		5）、拦截器链的触发过程;
            1)、如果没有拦截器执行执行目标方法，或者拦截器的索引和拦截器数组-1大小一样（指定到了最后一个拦截器）执行目标方法；
            2)、链式获取每一个拦截器，拦截器执行invoke方法，每一个拦截器等待下一个拦截器执行完成返回以后再来执行；
               拦截器链的机制，保证通知方法与目标方法的执行顺序；
```

###  **七、小结**

```
 1）、 @EnableAspectJAutoProxy 开启AOP功能
 2）、 @EnableAspectJAutoProxy 会给容器中注册一个组件 AnnotationAwareAspectJAutoProxyCreator
 3）、AnnotationAwareAspectJAutoProxyCreator是一个后置处理器；
 4）、容器的创建流程：
        1）、通过registerBeanPostProcessors（）注册后置处理器；然后创建AnnotationAwareAspectJAutoProxyCreator组件对象
        2）、finishBeanFactoryInitialization（）初始化剩下的单实例bean
            1）、创建业务逻辑组件和切面组件
            2）、AnnotationAwareAspectJAutoProxyCreator拦截组件的创建过程
            3）、组件创建完之后，判断组件是否需要增强
                     是：切面的通知方法，包装成增强器（Advisor）;给业务逻辑组件创建一个代理对象（cglib）；
 5）、执行目标方法：
        1）、代理对象执行目标方法
        2）、CglibAopProxy.intercept()；
             1）、得到目标方法的拦截器链（增强器包装成拦截器MethodInterceptor）
             2）、利用拦截器的链式机制，依次进入每一个拦截器进行执行；
             3）、效果：
                      正常执行：前置通知-》目标方法-》后置通知-》返回通知
                      出现异常：前置通知-》目标方法-》后置通知-》异常通知
```

# 面试

### 代理生成阶段

1、首先IOC会调用doCreateBean()来开始 Bean的创建，这里会完成bean的创建和Bean的依赖注入功能，然后会调用到initializeBean()方法初始化;

2、initializeBean 是在bean创建之后对Bean的信息进行一些业务初始化，这里会执行用户自定的业务如：

执行实现了Aware接口的业务。

执行实现了BeanPostProcessor接口的业务。

执行实现了InitializingBean 接口和initMethod的业务。

最后再调用到applyBeanPostProcessorsAfterInitialization方法，这个方法准备生成代理。

3、在bean的初始化业务都完成后，开始进行代理的生成。

这里经过了postProcessAfterInitialization()方法进入到wrapIfNecessary() 方法开始进入代理创建。

在创建代理之前首先会通过getAdvicesAndAdvisorsForBean（）来获得代理类的各个通知方法组成的方法链条，如：before、after 组成的方法顺序集合。

然后通过createProxy()方法来使用JDK或者Cglib的方式来生成代理对象。

整个代理生成方法调用流程如下：



![image-20201107141915878](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201107141915878.png)

### 代理执行阶段

代理生成后会保存到BeanWrappper包装类里面，我通过IOC容器getBean()方法获得的对象就是通过JDK或者Cglib生成的代理对象，那么其实我们每次调用bean的方法其实都和我们实际的对象没有关系了，我们使用的都是代理对象。

代理执行的逻辑很简单，两种代理对象执行的基本逻辑都差不多，这里是阅读的JDK的代理对象的执行过程。

1、代理对象每次执行方法首先会进入JdkDynamicAopProxy.invoke()方法.这个方法首先会过滤一些不需代理的一些对象原生方法，如：equals,hashcode。

*2、*然后会通过getInterceptorsAndDynamicInterceptionAdvice()方法 获取我们在代理生成阶段 生成的执行方法链，这里面包括了通知的方法，和他们执行的顺序。

3、拿到执行方法链之后就通过ReflectiveMethodInvocation类的proceed()方法来循环调用执行方法链的方法，执行完成之后整个流程结束。

整个代理执行调用流程如下：

![image-20201107141933741](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201107141933741.png)

### **解释一下Spring AOP里面的几个名词：**

1）切面（Aspect）：被抽取的公共模块，可能会横切多个对象。 在Spring AOP中，切面可以使用通用类（基于模式的风格） 或者在普通类中以 @AspectJ 注解来实现。

（2）连接点（Join point）：指方法，在Spring AOP中，一个连接点 总是 代表一个方法的执行。 

（3）通知（Advice）：在切面的某个特定的连接点（Join point）上执行的动作。通知有各种类型，其中包括“around”、“before”和“after”等通知。许多AOP框架，包括Spring，都是以拦截器做通知模型， 并维护一个以连接点为中心的拦截器链。

（4）切入点（Pointcut）：切入点是指 我们要对哪些Join point进行拦截的定义。通过切入点表达式，指定拦截的方法，比如指定拦截add*、search*。

（5）引入（Introduction）：（也被称为内部类型声明（inter-type declaration））。声明额外的方法或者某个类型的字段。Spring允许引入新的接口（以及一个对应的实现）到任何被代理的对象。例如，你可以使用一个引入来使bean实现 IsModified 接口，以便简化缓存机制。

（6）目标对象（Target Object）： 被一个或者多个切面（aspect）所通知（advise）的对象。也有人把它叫做 被通知（adviced） 对象。 既然Spring AOP是通过运行时代理实现的，这个对象永远是一个 被代理（proxied） 对象。

（7）织入（Weaving）：指把增强应用到目标对象来创建新的代理对象的过程。Spring是在运行时完成织入。

切入点（pointcut）和连接点（join point）匹配的概念是AOP的关键，这使得AOP不同于其它仅仅提供拦截功能的旧技术。 切入点使得定位通知（advice）可独立于OO层次。 例如，一个提供声明式事务管理的around通知可以被应用到一组横跨多个对象中的方法上（例如服务层的所有业务操作）。

![img](D:/JAVA/Java_Learning/Software/Typora/Picture/20180708154818891)











