[TOC]

#### 基础

* Mybatis是和数据库交互的半自动化的持久化层框架（SQL映射框架），而Hibernate是数据交互框架（ORM对象关系映射框架）
* Mybatis底层就是对原生JDBC的一个简单封装，只抽取出【写SQL】过程供程序员使用，其他都被Mybatis封装
* 导包：mysql-connector-java-5.1.37-bin.jar  和 mybatis-3.4.1.jar

#### 基于XML配置文件

* mybatis-config.xml：mybatis全局配置文件

```xml
<configuration>
	<!-- 从类路径里，配置连接池的数据 -->
	<properties resource="dbconfig.properties"></properties>
	<settings>
		<!-- 开启延迟加载开关 -->
		<setting name="lazyLoadingEnabled" value="true"/>
		<!-- 开启属性按需加载 -->
		<setting name="aggressiveLazyLoading" value="false"/>
		
		 <!-- 开启驼峰命名规则:javaBean字段符合驼峰命名规则，sql字段以‘_’分隔，也可以在sql映射文件起别名-->
		<setting name="mapUnderscoreToCamelCase" value="true" /> 
	</settings>
	<environments default="development">
		<environment id="development">
			<transactionManager type="JDBC" />  <!--  除了查询其他操作都需要openSession.commmit()提交会话-->
			<dataSource type="POOLED">
				<property name="driver" value="${driverclass}" />
				<property name="url" value="${jdbcurl}" />
				<property name="username" value="${username}" />
				<property name="password" value="${password}" />
			</dataSource>
		</environment>
	</environments>
	// 提高数据库移植性
	<databaseIdProvider type="DB_VENDOR">
		<property name="MySQL" value="mysql"/>
	</databaseIdProvider>
	<mappers>  
		<package name="com.atguigu.dao"/> <!-- 批量注册SQL映射文件-->
		<mapper resource="Employee"/> <!-- resource:类路径下找该sql映射文件 -->
	</mappers>
</configuration>
```



#### SQL映射文件：相当于对Dao接口的一个实现描述

```xml
<mapper namespace="com.atguigu.dao.EmployeeDao">
<!--public Map<String, Object> getEmpByIdReturnMap(Integer id);  -->
<!-- id：跟该namespace接口对应的方法。参数id跟接口的方法同名 -->

<!-- 查询数据，resultType为返回值类型,Employee为在该bean类上标注@Alias(“Employee”)的别名 -->
<select id="getEmpByIdReturnMap" resultType="Employee" databaseId="mysql" order="BEFORE"> <!-- databaseId：使用mysql数据库-->
	select * from t_employee where id=#{id}
</select>

<!-- 插入数据。 useGeneratedKeys：自增id，后台可以直接获取到生成的自增id。-->
<insert id="insertEmployee" useGeneratedKeys="true" keyProperty="id">
    INSERT INTO t_employee(empname,gender,email)
    VALUES(#{empName},#{gender},#{email})
</insert>
```

* 测试

```java
	@Before   // 初始化一个sql会话工厂sqlSessionFactory
	public void initSqlSessionFactory() throws IOException {
		String resource = "mybatis-config.xml";
		InputStream inputStream = Resources.getResourceAsStream(resource);
		sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream); // sqlSessionFactory负责创建SqlSession对象 
	}
	
	public void test() {
		SqlSession openSession = sqlSessionFactory.openSession();
		try {
			// sql会话获取dao接口实现。employeeDao.getClass()是一个代理对象proxy。并没有给接口写实现类，而在sql映射文件里实现接口方法
			EmployeeDao employeeDao = openSession.getMapper(EmployeeDao.class); 
			Employee empById = employeeDao.getEmpById(1);  // 调用接口方法操作数据库
			System.out.println(empById);
		} finally {
			openSession.close();  // 关闭会话
		}
	}
```

![image-20200718170615445](../../../../Software/Typora/Picture/image-20200718170615445.png)

* #{属性名}：是参数预编译的方式，参数的位置都是用？替代，参数后来都是预编译设置进去的；安全，不会有sql注入问题
  ${属性名}：不是参数预编译，而是直接和sql语句进行拼串；不安全；一般用于往某个日志写数据

* 自定义映射规则：（作为sql语句结果的返回值）

  * ```xml-dtd
    <resultMap type="com.atguigu.bean.Cat" id="mycat">
        <!-- 指定主键列的对应规则；
        column="id":指定哪一列是主键列
        property="":指定cat的哪个属性封装id这一列数据
        -->
        <id property="id" column="id"/>
        <!-- 普通列 -->
        <result property="name" column="cName"/>
        <result property="age" column="cAge"/>
        <result property="gender" column="cgender"/>
    </resultMap>
    ```

* 联合查询

```xml
 	<select id="getKeyById" resultMap="mykey">
 		select k.id,k.`keyname`,k.`lockid`,
		       l.`id` lid,l.`lockName` from t_key k
			left join t_lock l on k.`lockid`=l.`id`
			where k.`id`=#{id}
 	</select>
 	
 	 <!-- 自定义封装规则：使用级联属性封装联合查询出的结果 -->
	<!--<resultMap type="com.atguigu.bean.Key" id="mykey">
 		<id property="id" column="id"/>
 		<result property="keyName" column="keyname"/>
 		<result property="lock.id" column="lid"/>
 		<result property="lock.lockName" column="lockName"/>
 	</resultMap> -->
 	
 	<!-- mybatis推荐的   <association property=""></association>-->
 	<resultMap type="com.atguigu.bean.Key" id="mykey">
 		<id property="id" column="id"/>
 		<result property="keyName" column="keyname"/>
 		<!-- 接下来的属性是一个对象，自定义这个对象的封装规则；使用association；表示联合了一个对象 -->
 		<!-- javaType：指定这个属性的类型 -->
 		<association property="lock" javaType="com.atguigu.bean.Lock">
 			<!-- 定义lock属性对应的这个Lock对象如何封装 -->
 			<id property="id" column="lid"/>
 			<result property="lockName" column="lockName"/>
 		</association>
 	</resultMap>
```

* 查询集合

```xml
 		<collection property="keys" ofType="com.atguigu.bean.Key">
 			<!-- 标签体中指定集合中这个元素的封装规则 -->
 			<id property="id" column="kid"/>
 			<result property="keyName" column="keyname"/>
 		</collection>
```

* 指定分布查询

```xml
 		 <!--告诉mybatis自己去调用一个查询查锁子
 		select=""：指定一个查询sql的唯一标识；mybatis自动调用指定的sql将查出的lock封装进来
 		public Lock getLockByIdSimple(Integer id);需要传入锁子id
 		告诉mybatis把哪一列的值传递过去。column：指定将哪一列的数据传递过去。多列则需要{key1=col1,key2=col2}-->
 		<association property="lock" 
 			select="com.atguigu.dao.LockDao.getLockByIdSimple"
 			column="lockid" fetchType="lazy"> // fetchType="lazy"：按需加载.
 		</association>
```

* 简单的sql语句可以在dao接口上直接@Select("sql语句")、@Update("") ,   @Delete("") ,   @Insert("")

* ```
  参数大于1，参数自动被封装为map，key是索引。@Param就是为了避免被自动封装为map时是索引作为key，而是自定义key。sql映射文件才能直接#{id}使用
  public Employee getEmpByIdAndEmpName(@Param("id")Integer id, String empName);
  ```

* @MapKey("id")   ：把查询出来的结果集中的id作为key，存放在map里

#### 动态SQL

```xml
<!-- if：判断 -->
	<!--public List<Teacher> getTeacherByCondition(Teacher teacher); -->
	<select id="getTeacherByCondition" resultMap="teacherMap">
		select * from t_teacher
		<!-- test=""：编写判断条件 id!=null：取出传入的javaBean属性中的id的值，判断其是否为空 -->
		<!-- where可以帮我们去除掉前面的and; -->
		<!-- trim：截取字符串 
			prefix=""：前缀；为我们下面的sql整体添加一个前缀 
			prefixOverrides=""： 取出整体字符串前面多余的字符 
			suffix=""：为整体添加一个后缀 
			suffixOverrides=""：后面哪个多了可以去掉; -->
		<!-- 我们的查询条件就放在where标签中；每个and写在前面，
			where帮我们自动取出前面多余的and -->
		<trim prefix="where" prefixOverrides="and" suffixOverrides="and">
			<if test="id!=null">
				id > #{id} and
			</if>
			<!-- 空串 "" and； && or: ||； if()：传入非常强大的判断条件；
			OGNL表达式；对象导航图
			
			方法、静态方法、构造器。xxx
			在mybatis中，传入的参数可以用来做判断；
			额外还有两个东西；
			_parameter：代表传入来的参数；
				1）、传入了单个参数：_parameter就代表这个参数
				2）、传入了多个参数：_parameter就代表多个参数集合起来的map
			_databaseId：代表当前环境
				如果配置了databaseIdProvider：_databaseId就有值
				
			 -->
			<!-- 绑定一个表达式的值到一个变量 -->
			<!-- <bind name="_name" value="'%'+name+'%'"/> -->
			<if test="name!=null &amp;&amp; !name.equals(&quot;&quot;)">
				teacherName like #{_name} and
			</if>
			<if test="birth!=null">
				birth_date &lt; #{birth} and
			</if>
		</trim>
	</select>
	
	<!-- public List<Teacher> getTeacherByIdIn(List<Integer> ids); -->
	<select id="getTeacherByIdIn" resultMap="teacherMap">
		
		SELECT * FROM t_teacher WHERE id IN
		<!-- 帮我们遍历集合的； collection=""：指定要遍历的集合的key 
		close=""：以什么结束 
		index="i"：索引； 
			如果遍历的是一个list； 
				index：指定的变量保存了当前索引 
				item：保存当前遍历的元素的值 
			如果遍历的是一个map： 
				index：指定的变量就是保存了当前遍历的元素的key 
				item：就是保存当前遍历的元素的值
		item="变量名"：每次遍历出的元素起一个变量名方便引用 
		open=""：以什么开始 
		separator=""：每次遍历的元素的分隔符 
			(#{id_item},#{id_item},#{id_item} -->
		<if test="ids.size >0">
			<foreach collection="ids" item="id_item" separator="," open="("
				close=")">
				#{id_item}
			</foreach>
		</if>
	</select>

	<!--public List<Teacher> getTeacherByConditionChoose(Teacher teacher); -->
	<select id="getTeacherByConditionChoose" resultMap="teacherMap">
		select * from t_teacher
		<where>
			<choose>
				<when test="id!=null">
					id=#{id}
				</when>
				<when test="name!=null and !name.equals(&quot;&quot;)">
					teacherName=#{name}
				</when>
				<when test="birth!=null">
					birth_date = #{birth}
				</when>
				<otherwise>
					1=1
				</otherwise>
			</choose>
		</where>
	</select>

	<!-- public int updateTeacher(Teacher teacher); -->
	<update id="updateTeacher" >
		UPDATE t_teacher
		<set>
			<if test="name!=null and !name.equals(&quot;&quot;)">
				teacherName=#{name},
			</if>
			<if test="course!=null and !course.equals(&quot;&quot;)">
				class_name=#{course},
			</if>
			<if test="address!=null and !address.equals(&quot;&quot;)">
				address=#{address},
			</if>
			<if test="birth!=null">
				birth_date=#{birth}
			</if>
		</set>
		<where>
			id=#{id}
		</where>
		 
	</update>
```

#### Mybatis缓存机制

* 缓存：暂时的存储一些数据；加快系统的查询速度...

* MyBatis缓存机制：**Map**；能保存查询出的一些数据；只要之前查询过的数据，mybatis就会保存在一个缓存中（Map）；下次获取直接从缓存中拿；

* 一级缓存：

  * 线程级别的缓存；本地缓存；SqlSession级别的缓存；默认存在；
  * SqlSession关闭或者提交以后，一级缓存的数据会放在二级缓存中；

* 二级缓存：

  * 全局范围的缓存；除过当前线程、SqlSession能用外其他也可以使用；

  * namespace级别的缓存；**POJO需要实现Serializable**

  * 二级缓存在SqlSession关闭或提交之后，一级缓存的数据才会放在二级缓存中

  * 1) 首先全局配置开启二级缓存: 

    ```
    <setting name="cacheEnabled" value="true"/>  <!-- 开启全局缓存开关； -->
    ```

  * 2) 然后配置某个dao.xml文件，让其使用二级缓存

    * 	```xml
        <!-- 使用mybatis默认二级缓存<cache></cache> 。在<select>里可以使用useCache=false可以设置该sql语句不使用二级缓存-->
        	<cache type="org.mybatis.caches.ehcache.EhcacheCache"></cache>
        ```

* 一级缓存失效的几种情况：
  * 1、不同的SqlSession对应不同的一级缓存
  * 2、同一个SqlSession但是查询条件不同
  * 3、同一个SqlSession两次查询期间执行了任何一次增删改操作
  * 4、同一个SqlSession两次查询期间手动清空了缓存：openSession.clearCache();

* 不会出现一级缓存和二级缓存中有同一个数据，二级缓存中，一级缓存关闭了就有了

* 一级缓存中：二级缓存中没有此数据，就会看一级缓存，一级缓存没有去查数据库；数据库的查询后的结果放在一级缓存中了；

* 因此任何时候，都会先到二级缓存查-》一级缓存查=》数据库

* 缓存有关设置

  - 1、全局setting的cacheEnable：配置二级缓存的开关。一级缓存一直是打开的。
  - 2、select标签的useCache属性：配置这个select是否使用二级缓存。一级缓存一直是使用的
  - 3、sql标签的flushCache属性：增删改默认flushCache=true。sql执行以后，会同时清空一级和二级缓存。查询默认flushCache=false。
  - 4、sqlSession.clearCache()：只是用来清除一级缓存。
  - 5、当在某一个作用域 (一级缓存Session/二级缓存Namespaces) 进行了 C/U/D 操作后，默认该作用域下所有 select 中的缓存将被clear。

#### 整合第三方缓存

* 整合ehcache；ehcache非常专业的java进程内的缓存框架；

* 1、导包

  ```
  ehcache-core-2.6.8.jar(ehcache核心包)
  mybatis-ehcache-1.0.3.jar(ehcache的整合包) 
  slf4j-api-1.7.21.jar
  slf4j-log4j12-1.7.21.jar
  ```

* 2、ehcache要工作有一个配置文件；   文件名叫ehcache.xml；放在类路径的根目录下

* 3、在mapper.xml（sql映射文件）中配置使用自定义的缓存

  ```
  <cache type="org.mybatis.caches.ehcache.EhcacheCache"></cache>
  ```

* 4、别的dao还要用这个缓存；缓存引用cache-ref

  ```
  <!-- 和别的dao共用一块缓存-->  <cache-ref namespace="com.atguigu.dao.TeacherDao"/>
  ```

* ```xml
  <cache-ref namespace="dao接口类的全类名">  // 和别的dao共用一块缓存
  <!-- 磁盘保存路径 -->
   <diskStore path="D:\44\ehcache" />
   <defaultCache 
     maxElementsInMemory="1" 
     maxElementsOnDisk="10000000"
     eternal="false" 
     overflowToDisk="true" 
     timeToIdleSeconds="120"
     timeToLiveSeconds="120" 
     diskExpiryThreadIntervalSeconds="120"
     memoryStoreEvictionPolicy="LRU">
   </defaultCache>
  </ehcache>
  <!-- 
  属性说明：
  l diskStore：指定数据在磁盘中的存储位置。
  l defaultCache：当借助CacheManager.add("demoCache")创建Cache时，EhCache便会采用<defalutCache/>指定的的管理策略
   
  以下属性是必须的：
  l maxElementsInMemory - 在内存中缓存的element的最大数目 
  l maxElementsOnDisk - 在磁盘上缓存的element的最大数目，若是0表示无穷大
  l eternal - 设定缓存的elements是否永远不过期。如果为true，则缓存的数据始终有效，如果为false那么还要根据timeToIdleSeconds，timeToLiveSeconds判断
  l overflowToDisk - 设定当内存缓存溢出的时候是否将过期的element缓存到磁盘上
   
  以下属性是可选的：
  l timeToIdleSeconds - 当缓存在EhCache中的数据前后两次访问的时间超过timeToIdleSeconds的属性取值时，这些数据便会删除，默认值是0,也就是可闲置时间无穷大
  l timeToLiveSeconds - 缓存element的有效生命期，默认是0.,也就是element存活时间无穷大
   diskSpoolBufferSizeMB 这个参数设置DiskStore(磁盘缓存)的缓存区大小.默认是30MB.每个Cache都应该有自己的一个缓冲区.
  l diskPersistent - 在VM重启的时候是否启用磁盘保存EhCache中的数据，默认是false。
  l diskExpiryThreadIntervalSeconds - 磁盘缓存的清理线程运行间隔，默认是120秒。每个120s，相应的线程会进行一次EhCache中数据的清理工作
  l memoryStoreEvictionPolicy - 当内存缓存达到最大，有新的element加入的时候， 移除缓存中element的策略。默认是LRU（最近最少使用），可选的有LFU（最不常使用）和FIFO（先进先出）
   -->
  ```

  

#### MBG逆向工程

* 正向：

  table----javaBean---BookDao---dao.xml---xxx

  逆向工程：

  根据数据表table，逆向分析数据表，自动生成javaBean---BookDao---dao.xml---xxx

* MBG：MyBatis Generator：代码生成器；MyBatis官方提供的代码生成器；帮我们逆向生成；

* 1、导包：mbg的核心包

* 2、编写mbg.xml配置文件

  * ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE generatorConfiguration
      PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
      "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
    
    <generatorConfiguration>
    
        <!--
        MyBatis3Simple：基础班CRUD
        MyBatis3：复杂版CRUD
         -->
        <context id="DB2Tables" targetRuntime="MyBatis3">
            <commentGenerator>
                <property name="suppressAllComments" value="true"/>
            </commentGenerator>
            <!-- jdbcConnection:指导连接到哪个数据库 -->
            <jdbcConnection
    
                driverClass="com.mysql.jdbc.Driver"
                connectionURL="jdbc:mysql://localhost:3306/mybatis_0325"
    
                userId="root"
    
                password="123456">
            </jdbcConnection>
    
            <javaTypeResolver>
                <property name="forceBigDecimals" value="false" />
            </javaTypeResolver>
    
            <!-- javaModelGenerator：生成pojo
    
            targetPackage：生成的pojo放在哪个包
            targetProject：放在哪个工程下
            -->
            <javaModelGenerator targetPackage="com.atguigu.bean"
                targetProject=".\src">
                <property name="enableSubPackages" value="true" />
                <property name="trimStrings" value="true" />
            </javaModelGenerator>
    
            <!--sqlMapGenerator：sql映射文件生成器；指定xml生成的地方  -->
            <sqlMapGenerator targetPackage="com.atguigu.dao"
                targetProject=".\conf">
                <property name="enableSubPackages" value="true" />
            </sqlMapGenerator>
    
            <!-- javaClientGenerator：dao接口生成的地方 -->
            <javaClientGenerator type="XMLMAPPER"
                targetPackage="com.atguigu.dao"
    
                targetProject=".\src">
                <property name="enableSubPackages" value="true" />
            </javaClientGenerator>
    
            <!-- table：指定要逆向生成哪个数据表
            tableName="t_cat"：表名
            domainObjectName=""：这个表对应的对象名
             -->
            <table tableName="t_cat" domainObjectName="Cat"></table>
            <table tableName="t_employee" domainObjectName="Employee"></table>
            <table tableName="t_teacher" domainObjectName="Teacher"></table>
    
        </context>
    </generatorConfiguration>
    ```

    

* 3、运行代码生成

  ```
  public class MBGTest {
  
      public static void main(String[] args) throws Exception {
          List<String> warnings = new ArrayList<String>();
          boolean overwrite = true;
          File configFile = new File("mbg.xml");
          ConfigurationParser cp = new ConfigurationParser(warnings);
          Configuration config = cp.parseConfiguration(configFile);
          DefaultShellCallback callback = new DefaultShellCallback(overwrite);
          MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config,
                  callback, warnings);
          //代码生成
          myBatisGenerator.generate(null);
          System.out.println("生成ok了！");
      }
  
  }
  ```
  
* 4、测试复杂查询

```java
public class MyBatisTest {

    // 工厂一个
    SqlSessionFactory sqlSessionFactory;


    @Test
    public void test02(){
        SqlSession sqlSession = sqlSessionFactory.openSession();
        //1、测试
        TeacherMapper mapper = sqlSession.getMapper(TeacherMapper.class);
        List<Teacher> teachers = new ArrayList<Teacher>();
        for (int i = 0; i < 1000; i++) {
            Teacher teacher = new Teacher();
            teacher.setTeachername(UUID.randomUUID().toString().substring(0, 5));
            teacher.setClassName(UUID.randomUUID().toString().substring(0, 5));
            teachers.add(teacher);
        }
        System.out.println("批量保存.....");
        mapper.insertBatch(teachers);
        sqlSession.commit();
        sqlSession.close();


    }

    /**
     * 测试代码生成器
     * @throws IOException
     */
    @Test
    public void test01(){
        SqlSession sqlSession = sqlSessionFactory.openSession();
        //1、测试
        TeacherMapper mapper = sqlSession.getMapper(TeacherMapper.class);
        //2、测试查询所有teacher
        List<Teacher> list = mapper.selectByExample(null);
        for (Teacher teacher : list) {
            System.out.println(teacher);
        }

        //3、带复杂条件的查询
        //select * from t_teacher id=? and teacherName like ?
        //封装查询条件的
        TeacherExample example = new TeacherExample();
        example.setOrderByClause("id DESC");
        //1、使用example创建一个Criteria（查询准则）
        Criteria criteria = example.createCriteria();
        criteria.andIdEqualTo(1);
        criteria.andTeachernameLike("%a%");

        System.out.println("======================");
        List<Teacher> list2 = mapper.selectByExample(example);
        for (Teacher teacher : list2) {
                System.out.println(teacher);
        }

        /**
         * 多个复杂条件
         * select * from t_teacher where  (id=? and teacherName like ?) or (address like ? and birth bet)
         */
        TeacherExample example2 = new TeacherExample();


        //一个Criteria能封装一整个条件
        Criteria criteria2 = example2.createCriteria();
        criteria2.andIdGreaterThan(1);
        criteria2.andTeachernameLike("%a%");

        //创建第二个查询条件
        Criteria criteria3 = example2.createCriteria();
        criteria3.andAddressLike("%%");
        criteria3.andBirthDateBetween(new Date(), new Date());

        example2.or(criteria3);
        System.out.println("=======-=-=-=-=-=-=-");
        mapper.selectByExample(example2);

    }

    @Before
    public void initSqlSessionFactory() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    }

}
```



# 原理剖析

**MyBatis** **流程图**

![image-20200916133200183](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916133200183.png)

 

**Configuration.xml** 

该配置文件是 MyBatis 的全局配置文件，在这个文件中可以配置诸多项目。常用的内容是别名设置，拦截器设置等。

**Properties**属性）

将数据库连接参数单独配置在 db.properties 中，放在类路径下。这样只需要在SqlMapConfig.xml 中加载 db.properties 的属性值。这样在 SqlMapConfig.xml 中就不需要对数据库连接参数硬编码。

将数据库连接参数只配置在 db.properties 中，原因：方便对参数进行统一管理.

**Settings**（全局配置参数）:

Mybatis 全局配置参数，全局参数将会影响 mybatis 的运行行为。比如：开启二级缓存、开启延迟加载。

**TypeAliase**（类型别名）：

类型别名是为 Java 类型命名一个短的名字。它只和 XML 配置有关, 只用来减少类完全限定名的多余部分。

**Plugin**（插件）：

MyBatis 允许你在某一点拦截已映射语句执行的调用。默认情况下,MyBatis 允许使用插件来拦截方法调用


**Environments*（环境集合属性对象）：

MyBatis 可以配置多种环境。这会帮助你将 SQL 映射应用于多种数据库之中。但是要记得一个很重要的问题：你可以配置多种环境，但每个数据库对应一个 SqlSessionFactory。

所以，如果你想连接两个数据库，你需要创建两个 SqlSessionFactory 实例，每个数据库对应一个。

**Environment**（环境子属性对象）

**TransactionManager****（事务管理）**:在 MyBatis 中有两种事务管理器类型(也就是 type=”[JDBC|MANAGED]”)

**DataSource（数据源）:**UNPOOLED|POOLED|JNDI

**Mappers**（映射器）

指定映射配置文件位置

```html
<mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
<mapper url="file:///var/mappers/AuthorMapper.xml"/>
<mapper class="org.mybatis.builder.AuthorMapper"/>
<package name="org.mybatis.builder"/>
```

*Mapper.xml*：

Mapper.xml 映射文件中定义了操作数据库的 sql，每个 sql 是一个 statement，映射文件是 mybatis 的核心

  **ResultMap** ：Mybatis 中可以使用 resultMap 完成高级输出结果映射。如果查询出来的列名和定义的pojo 属性名不一致，就可 以通过定义一个 resultMap 对列名和 pojo 属性名之间作一个映射关系。

  **Cache：**开启二级缓存

  **Sql：**可以重用的 SQL 块,也可以被其他语句引用

**Resources**

Resources 工具类会从路径中加载资源，并返回一个输入流对象，对于资源文件的加载提供了简易的使用方法。

加载一个资源有很多方式：

对于简单的只读文本数据，加载为 Reader。

对于简单的只读二进制或文本数据，加载为 Stream。

对于可读写的二进制或文本文件，加载为 File。

对于只读的配置属性文件，加载为 Properties。

对于只读的通用资源，加载为 URL。

按以上的顺序，Resources 类加载资源的方法如下：

**Reader getResourceAsReader(String resource);**

**Stream getResourceAsStream(String resource);**

**File getResourceAsFile(String resource);**

**Properties getResourceAsProperties(String resource);**

**Url getResourceAsUrl(String resource);**



**SqlSessionFactoryBuilder：**

该类是 SqlSessionFactory（会话工厂）的构建者类，之前描述的操作其实全是从这里面开启的，首先就是调用 XMLConfigBuilder 类的构造器来创建一个 XML 配置构建器对象， 利用这个构建器对象来调用其解析方法 parse()来完成 Configuration 对象的创建，之后以这个配置对象为参数调用会话工厂构建者类中的 build(Configuration config)方法来完成SqlSessionFactory（会话工厂）对象的构建。

**XMLConfigBuilder：**

该类是 XML 配置构建者类，是用来通过 XML 配置文件来构建 Configuration 对象实例，构建的过程就是解析 Configuration.xml 配置文件的过程，期间会将从配置文件中获取到的指定标签的值逐个添加到之前创建好的默认 Configuration 对象实例中

**Configuration**

该对象是 Mybatis 的上下文对象，实例化这个类的目的就是为了使用其对象作为项目全局配置对象，这样通过配置文件配置的信息可以保存在这个配置对象中，而这个配置对象在创建好之后是保存在 JVM 的 Heap 内存中的，方便随时读取。不然每次需要配置信息的时候都要临时从磁盘配置文件中获取，代码复用性差的同时，也不利于开发

**DefaultSqlSessionFactory ：**

SqlsessionFactory该接口是会话工厂，是用来生产会话的工厂接口，DefaultSqlSessionFactory是其实现类，是真正生产会话的工厂类，这个类的实例的生命周期是全局的，它只会在首次调用时生成一个实例（单例模式），就一直存在直到服务器关闭。

**Executor**

执行器接口，SqlSession 会话是面向程序员的，而内部真正执行数据库操作的却是Executor 执行器，可以将 Executor 看作是面向 MyBatis 执行环境的，SqlSession 就是门面货，Executor 才是实干家。通过 SqlSession 产生的数据库操作，全部是通过调用 Executor 执行器来完成的。Executor 是跟 SqlSession 绑定在一起的，每一个 SqlSession 都拥有一个新的 Executor 对象，由 Configuration 创建。

  **Executor** **继承结构 ：**

![image-20200916133207956](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916133207956.png)

**BaseExecutor：**

SimpleExecutor：每执行一次 update 或 select，就开启一个 Statement 对象，用完立刻关闭 Statement 对象。（可以是 Statement 或 PrepareStatement 对象）

ReuseExecutor：执行 update 或 select，以 sql 作为 key 查找 Statement 对象，存在就使用，不存在就创建，用完后，不关闭 Statement 对象，而是放置于 Map<String, Statement>内，供下一次使用。（可以是 Statement 或 PrepareStatement 对象）

BatchExecutor：执行 update（没有 select，JDBC 批处理不支持 select），将所有 sql 都添加到批处理中（addBatch()），等待统一执行（executeBatch()），它缓存了多个 Statement对象，每个 Statement 对象都是 addBatch()完毕后，等待逐一执行

**CachingExecutor：**先从缓存中获取查询结果，存在就返回，不存在，再委托给 Executor delegate 去数据库取，delegate 可以是上面任一的 SimpleExecutor、ReuseExecutor、BatchExecutor。

**StatementHandler**

该类是 Statement 处理器，封装了 Statement 的各种数据库操作方法 execute()，可见MyBatis 其实就是将操作数据库的 JDBC 操作封装起来的一个框架，同时还实现了 ORM 罢了。RoutingStatementHandler，这是一个封装类，它不提供具体的实现，只是根据 Executor的类型，创建不同的类型 StatementHandler。

**ResultSetHandler**

结果集处理器，如果是查询操作，必定会有返回结果，针对返回结果的操作，就要使用ResultSetHandler 来进行处理，这个是由 StatementHandler 来进行调用的。这个处理器的作用就是对返回结果进行处理。

# Mybatis缓存机制

参考：https://www.cnblogs.com/wuzhenzhao/p/11103043.html

缓存是一般的ORM 框架都会提供的功能，目的就是提升查询的效率和减少数据库的压力。跟Hibernate 一样，MyBatis 也有一级缓存和二级缓存，并且预留了集成第三方缓存的接口。

　MyBatis 跟缓存相关的类都在cache 包里面，其中有一个Cache 接口，只有一个默认的实现类 PerpetualCache，它是用HashMap 实现的。

　　除此之外，还有很多的装饰器，通过这些装饰器可以额外实现很多的功能：回收策略、日志记录、定时刷新等等。但是无论怎么装饰，经过多少层装饰，最后使用的还是基本的实现类（默认PerpetualCache）。可以通过 CachingExecutor 类 Debug 去查看。

![image-20200917213336125](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200917213336125.png)

## 一级缓存

即本地缓存，MyBatis 的一级缓存是在会话（SqlSession）层面进行缓存的。MyBatis 的一级缓存是默认开启的，不需要任何的配置。

如果要在同一个会话里面共享一级缓存，这个对象肯定是在SqlSession 里面创建的，作为SqlSession 的一个属性。

DefaultSqlSession 里面只有两个属性，Configuration 是全局的，所以缓存只可能放在Executor 里面维护

SimpleExecutor/ReuseExecutor/BatchExecutor 的父类BaseExecutor 的构造函数中持有了PerpetualCache。在同一个会话里面，多次执行相同的SQL 语句，会直接从内存取到缓存的结果，不会再发送SQL 到数据库。但是不同的会话里面，即使执行的SQL 一模一样（通过一个Mapper 的同一个方法的相同参数调用），也不能使用到一级缓存。

　　每当我们使用MyBatis开启一次和数据库的会话，MyBatis会创建出一个SqlSession对象表示一次数据库会话。

　　在对数据库的一次会话中，我们有可能会反复地执行完全相同的查询语句，如果不采取一些措施的话，每一次查询都会查询一次数据库,而我们在极短的时间内做了完全相同的查询，那么它们的结果极有可能完全相同，由于查询一次数据库的代价很大，这有可能造成很大的资源浪费。

　　为了解决这一问题，减少资源的浪费，MyBatis会在表示会话的SqlSession对象中建立一个简单的缓存，将每次查询到的结果结果缓存起来，当下次查询的时候，如果判断先前有个完全一样的查询，会直接从缓存中直接将结果取出，返回给用户，不需要再进行一次数据库查询了。

　　如下图所示，MyBatis会在一次会话的表示----一个SqlSession对象中创建一个本地缓存(local cache)，对于每一次查询，都会尝试根据查询的条件去本地缓存中查找是否在缓存中，如果在缓存中，就直接从缓存中取出，然后返回给用户；否则，从数据库读取数据，将查询结果存入缓存并返回给用户。

![image-20200918161810332](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200918161810332.png)

### 一级缓存的生命周期

1. MyBatis在开启一个数据库会话时，会 创建一个新的SqlSession对象，SqlSession对象中会有一个新的Executor对象，Executor对象中持有一个新的PerpetualCache对象；当会话结束时，SqlSession对象及其内部的Executor对象还有PerpetualCache对象也一并释放掉。
2. 如果SqlSession调用了close()方法，会释放掉一级缓存PerpetualCache对象，一级缓存将不可用；
3. 如果SqlSession调用了clearCache()，会清空PerpetualCache对象中的数据，但是该对象仍可使用；
4. SqlSession中执行了任何一个update操作(update()、delete()、insert()) ，都会清空PerpetualCache对象的数据，但是该对象可以继续使用；

#### SqlSession 一级缓存的工作流程：

1. 对于某个查询，根据statementId,params,rowBounds来构建一个key值，根据这个key值去缓存Cache中取出对应的key值存储的缓存结果
2. 判断从Cache中根据特定的key值取的数据数据是否为空，即是否命中；
3. 如果命中，则直接将缓存结果返回；
4. 如果没命中：

> ​	1.去数据库中查询数据，得到查询结果；
>
> ​	2.将key和查询到的结果分别作为key,value对存储到Cache中；
>
> ​	3.将查询结果返回

#### 一级缓存的不足：

　　使用一级缓存的时候，因为缓存不能跨会话共享，不同的会话之间对于相同的数据可能有不一样的缓存。在有多个会话或者分布式环境下，会存在脏数据的问题。如果要解决这个问题，就要用到二级缓存。MyBatis 一级缓存（MyBaits 称其为 Local Cache）无法关闭，但是有两种级别可选：

1. session 级别的缓存，在同一个 sqlSession 内，对同样的查询将不再查询数据库，直接从缓存中。
2. statement 级别的缓存，避坑： 为了避免这个问题，可以将一级缓存的级别设为 statement 级别的，这样每次查询结束都会清掉一级缓存。





## 二级缓存

　二级缓存是全局缓存，用来解决一级缓存不能跨会话共享的问题的，范围是namespace 级别的，可以被多个SqlSession 共享（只要是同一个接口里面的相同方法，都可以共享），生命周期和应用同步。如果你的MyBatis使用了二级缓存，并且你的Mapper和select语句也配置使用了二级缓存，那么在执行select查询的时候，MyBatis会先从二级缓存中取输入，其次才是一级缓存，即MyBatis查询数据的顺序是：二级缓存  —> 一级缓存 —> 数据库。

　　作为一个作用范围更广的缓存，它肯定是在SqlSession 的外层，否则不可能被多个SqlSession 共享。而一级缓存是在SqlSession 内部的，所以第一个问题，肯定是工作在一级缓存之前，也就是只有取不到二级缓存的情况下才到一个会话中去取一级缓存。第二个问题，二级缓存放在哪个对象中维护呢？ 要跨会话共享的话，SqlSession 本身和它里面的BaseExecutor 已经满足不了需求了，那我们应该在BaseExecutor 之外创建一个对象。

　　实际上MyBatis 用了一个装饰器的类来维护，就是CachingExecutor。如果启用了二级缓存，MyBatis 在创建Executor 对象的时候会对Executor 进行装饰。CachingExecutor 对于查询请求，会判断二级缓存是否有缓存结果，如果有就直接返回，如果没有委派交给真正的查询器Executor 实现类，比如SimpleExecutor 来执行查询，再走到一级缓存的流程。最后会把结果缓存起来，并且返回给用户。

![image-20200918161955085](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200918161955085.png)

### 开启二级缓存

第一步：配置 mybatis.configuration.cache-enabled=true，只要没有显式地设置cacheEnabled=false，都会用CachingExecutor 装饰基本的执行器。

第二步：在Mapper.xml 中配置<cache/>标签：

```xml
<cache type="org.apache.ibatis.cache.impl.PerpetualCache" 
size="1024"
eviction="LRU"
flushInterval="120000"
readOnly="false"/>
```

* 映射语句文件中的所有 select 语句的结果将会被缓存。
* 映射语句文件中的所有 insert、update 和 delete 语句会刷新缓存。
* 缓存会使用最近最少使用算法（LRU, Least Recently Used）算法来清除不需要的缓存。
* 缓存不会定时进行刷新（也就是说，没有刷新间隔）。
* 缓存会保存列表或对象（无论查询方法返回哪种）的 1024 个引用。
* 缓存会被视为读/写缓存，这意味着获取到的对象并不是共享的，可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改。

这个更高级的配置创建了一个 FIFO 缓存，每隔 60 秒刷新，最多可以存储结果对象或列表的 512 个引用，而且返回的对象被认为是只读的，因此对它们进行修改可能会在不同线程中的调用者产生冲突。可用的清除策略有：

- `LRU` – 最近最少使用：移除最长时间不被使用的对象。
- `FIFO` – 先进先出：按对象进入缓存的顺序来移除它们。
- `SOFT` – 软引用：基于垃圾回收器状态和软引用规则移除对象。
- `WEAK` – 弱引用：更积极地基于垃圾收集器状态和弱引用规则移除对象。

默认的清除策略是 LRU。

flushInterval（刷新间隔）属性可以被设置为任意的正整数，设置的值应该是一个以毫秒为单位的合理时间量。 默认情况是不设置，也就是没有刷新间隔，缓存仅仅会在调用语句时刷新。

size（引用数目）属性可以被设置为任意正整数，要注意欲缓存对象的大小和运行环境中可用的内存资源。默认值是 1024。

readOnly（只读）属性可以被设置为 true 或 false。只读的缓存会给所有调用者返回缓存对象的相同实例。 因此这些对象不能被修改。这就提供了可观的性能提升。而可读写的缓存会（通过序列化）返回缓存对象的拷贝。 速度上会慢一些，但是更安全，因此默认值是 false。

　　注：二级缓存是事务性的。这意味着，当 SqlSession 完成并提交时，或是完成并回滚，但没有执行 flushCache=true 的 insert/delete/update 语句时，缓存会获得更新。

　　Mapper.xml 配置了<cache>之后，select()会被缓存。update()、delete()、insert()会刷新缓存。：如果cacheEnabled=true，Mapper.xml 没有配置标签，还有二级缓存吗？（没有）还会出现CachingExecutor 包装对象吗？（会）

　　只要cacheEnabled=true 基本执行器就会被装饰。有没有配置<cache>，决定了在启动的时候会不会创建这个mapper 的Cache 对象，只是最终会影响到CachingExecutorquery 方法里面的判断。如果某些查询方法对数据的实时性要求很高，不需要二级缓存，怎么办？我们可以在单个Statement ID 上显式关闭二级缓存（默认是true）：

```
<select id="selectBlog" resultMap="BaseResultMap" useCache="false">
```

### 什么时候开启二级缓存？

1. 因为所有的增删改都会刷新二级缓存，导致二级缓存失效，所以适合在查询为主的应用中使用，比如历史交易、历史订单的查询。否则缓存就失去了意义。
2. 如果多个namespace 中有针对于同一个表的操作，比如Blog 表，如果在一个namespace 中刷新了缓存，另一个namespace 中没有刷新，就会出现读到脏数据的情况。所以，推荐在一个Mapper 里面只操作单表的情况使用。

### 第三方缓存做二级缓存

　　除了MyBatis 自带的二级缓存之外，我们也可以通过实现Cache 接口来自定义二级缓存。MyBatis 官方提供了一些第三方缓存集成方式，比如ehcache 和redis。当然，我们也可以使用独立的缓存服务，不使用MyBatis 自带的二级缓存。

### 自定义缓存：

　　除了上述自定义缓存的方式，你也可以通过实现你自己的缓存，或为其他第三方缓存方案创建适配器，来完全覆盖缓存行为。

```
<cache type="com.domain.something.MyCustomCache"/>
```

　这个示例展示了如何使用一个自定义的缓存实现。type 属性指定的类必须实现 org.mybatis.cache.Cache 接口，且提供一个接受 String 参数作为 id 的构造器。 这个接口是 MyBatis 框架中许多复杂的接口之一，但是行为却非常简单。



# 面试



