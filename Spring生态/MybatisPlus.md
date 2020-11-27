[TOC]

## 特性

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- **支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer2005、SQLServer 等多种数据库
- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- **支持 XML 热加载**：Mapper 对应的 XML 支持热加载，对于简单的 CRUD 操作，甚至可以无 XML 启动
- **支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
- **支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ）
- **支持关键词自动转义**：支持数据库关键词（order、key......）自动转义，还可自定义关键词
- **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
- **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- **内置性能分析插件**：可输出 Sql 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作
- **内置 Sql 注入剥离器**：支持 Sql 注入剥离，有效预防 Sql 注入攻击

## 使用

* ```xml
  <dependency>
     <groupId>com.baomidou</groupId>
     <artifactId>mybatis-plus-boot-starter</artifactId>
     <version>3.0.5</version>
  </dependency>
  ```

* 创建包 mapper 编写Mapper 接口： `UserMapper.java`

```java
@Repository   // 也可以在启动类扫描包下的mapper接口: @MapperScan()
public interface UserMapper extends BaseMapper<User> {
}
```

* 测试

  ```java
      @Test
      public void testSelectList() {
          System.out.println(("----- selectAll method test ------"));
          //UserMapper 中的 selectList() 方法的参数为 MP 内置的条件封装器 Wrapper
          //所以不填写就是无任何条件
          List<User> users = userMapper.selectList(null);
          users.forEach(System.out::println);
      }
  ```

* 可以使用mybatis-plus的配置日志，在application.properties配置sql输出日志

  ```properties
  #mybatis日志
  mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
  ```


## CRUD

```java
List<User> users = userMapper.selectList(null); // 查
int result = userMapper.updateById(user); // 改
int result = userMapper.insert(user); // 增
int result = userMapper.deleteById(8L); // 删
```

### insert

```
int result = userMapper.insert(user); // 增
```

* 数据库插入id值默认为：全局唯一id

* 主键策略：

* MyBatis-Plus默认的主键策略是：ID_WORKER  *全局唯一ID*

* ```java
  # 可以在配置文件配置全局设置主键生成策略
  mybatis-plus.global-config.db-config.id-type=auto
  
  @TableId(type = IdType.AUTO)   // 主键自增
  private Long id;
  ```
  

### update

```
int result = userMapper.updateById(user); // 改
```

* 自动填充（如自动更新创建时间，更新时间）

  ```java
  @TableField(fill = FieldFill.INSERT)  // 表示添加时会自动填充字段createTime
  private Date createTime;
  
  @TableField(fill = FieldFill.INSERT_UPDATE) // 添加和修改会自动填充字段updateTime
  private Date updateTime;
  ```

* **实现元对象处理器接口**

```java
@Component  
public class MyMetaObjectHandler implements MetaObjectHandler {  
	@Override   // 使用mabatis-plus实现添加操作，将执行这 个方法（selectFill、updateFill、deleteFill也同理）
    public void insertFill(MetaObject metaObject) {  // metaObject 相当于与属性名对应的数据库表字段 
        this.setFieldValByName("createTime", new Date(), metaObject); // “createTime”表示标注了@TableField的属性名
        this.setFieldValByName("updateTime", new Date(), metaObject);
    }
    @Override
    public void updateFill(MetaObject metaObject) {
        LOGGER.info("start update fill ....");
        this.setFieldValByName("updateTime", new Date(), metaObject);
    }
}
```

* #### 乐观锁

**主要适用场景：**当要更新一条记录的时候，希望这条记录没有被别人更新，也就是说实现线程安全的数据更新

* **乐观锁实现方式：**
  - 取出记录时，获取当前version
  - 更新时，带上这个version
  - 执行更新时， set version = newVersion where version = oldVersion
  - 如果version不对，就更新失败

* 使用方式：

  * （1）在数据库中添加version字段

  * （2）实体类添加version字段，并添加@Version字段

    ```java
    @Version  // 每次该字段修改都会加1，初始为null
    private Integer version;
    ```

  * （3）在 MybatisPlusConfig 中注册 Bean

    ```java
    @EnableTransactionManagement
    @Configuration
    @MapperScan("com.atguigu.mybatis_plus.mapper")
    public class MybatisPlusConfig {
        /**
         * 乐观锁插件
         */
        @Bean
        public OptimisticLockerInterceptor optimisticLockerInterceptor() {
            return new OptimisticLockerInterceptor();
        }
    }
    ```

  * 测试后分析打印的sql语句，每次先查询后更新数据，更新数据时将修改的对象与数据库version字段比较，相等则将version的数值进行了加1操作

  * 测试乐观锁修改失败：

    ```java
    @Test
    public void testOptimisticLockerFail() {
        //查询
        User user = userMapper.selectById(1L);
        //修改数据
        user.setName("Helen Yao1");
        user.setEmail("helen@qq.com1");
        //模拟取出数据后，数据库中version实际数据比取出的值大，即已被其它线程修改并更新了version
        user.setVersion(user.getVersion() - 1);
        //执行更新
        userMapper.updateById(user);
    }
    ```

### select

* 多个id批量查询

  ```
  User user = userMapper.selectById(1L);
  List<User> users = userMapper.selectBatchIds(Arrays.asList(1, 2, 3)); // 批量查询
  ```

* **特别说明:**

  支持的数据类型只有 int,Integer,long,Long,Date,Timestamp,LocalDateTime

  整数类型下 `newVersion = oldVersion + 1`

  `newVersion` 会回写到 `entity` 中

  仅支持 `updateById(id)` 与 `update(entity, wrapper)` 方法

  在 `update(entity, wrapper)` 方法下, `wrapper` 不能复用!!!

#### 分页

* 1）配置分页插件

  ```java
  @Bean
  public PaginationInterceptor paginationInterceptor() {
      return new PaginationInterceptor();
  }
  ```

* 2）测试分页

  ```java
  @Test
  public void testSelectPage() {
      Page<User> page = new Page<>(1,5);// sql语句：SELECT id,name,age,email,create_time,update_time FROM user LIMIT 0,5 
      userMapper.selectPage(page, null);  // 参数分别是page对象和分页条件。查询的分页结果会保存到page对象里
     // IPage<Map<String, Object>> mapIPage = userMapper.selectMapsPage(page, null); // 结果集为map
      page.getRecords().forEach(System.out::println);
      System.out.println(page.getCurrent());
      System.out.println(page.getPages());
      System.out.println(page.getSize());
      System.out.println(page.getTotal());
      System.out.println(page.hasNext());
      System.out.println(page.hasPrevious());
  }
  ```

  

### delete

```
int result = userMapper.deleteById(8L);
int result = userMapper.deleteBatchIds(Arrays.asList(8, 9, 10));  // 批量删除
int result = userMapper.deleteByMap(map); // 条件删除
```

* 逻辑删除

  ```java
  // （1）数据库中添加 deleted字段
  //（2）实体类添加deleted 字段   ALTER TABLE `user` ADD COLUMN `deleted`
  @TableLogic  // 开启逻辑删除
  private Integer deleted;
  ```

* application.properties 加入配置 (默认已经这样配置，可不写)

  ```properties
  mybatis-plus.global-config.db-config.logic-delete-value=1
  mybatis-plus.global-config.db-config.logic-not-delete-value=0
  ```

* （3）在 MybatisPlusConfig 中注册 Bean

  ```java
  @Bean
  public ISqlInjector sqlInjector() {
      return new LogicSqlInjector();
  }
  ```

* 测试逻辑删除

  ```java
  int result = userMapper.deleteById(1L);  // 底层实现 set deleted=1
  List<User> users = userMapper.selectList(null);   // 底层的查询封装了查询条件 where deleted=0
  ```

## 性能分析

* 性能分析拦截器

```java
/**
 * SQL 执行性能分析插件
 * 开发环境使用，线上不推荐。 maxTime 指的是 sql 最大执行时长
 */
@Bean
@Profile({"dev","test"})// 设置 dev test 环境开启，指当前插件对 开发和测试环境生效
public PerformanceInterceptor performanceInterceptor() {
    PerformanceInterceptor performanceInterceptor = new PerformanceInterceptor();
    performanceInterceptor.setMaxTime(100);// 单位ms，超过此处设置的ms则sql不执行
    performanceInterceptor.setFormat(true); // 开启检测
    return performanceInterceptor;
}
```

* 设置环境

  ```properties
  #环境设置：dev、test、prod
  spring.profiles.active=dev
  ```

* 常规测试

  ```java
  /**
   * 测试 性能分析插件
   */
  @Test
  public void testPerformance() {
      User user = new User();
      user.setName("我是Helen");
      user.setEmail("helen@sina.com");
      user.setAge(18);
      userMapper.insert(user);
  }
  ```

  

## MyBatisPlus条件构造器

![image-20200727233718933](../../../../Software/Typora/Picture/image-20200727233718933.png)

* Wrapper ： 条件构造抽象类，最顶端父类

    AbstractWrapper ： 用于查询条件封装，生成 sql 的 where 条件

  ​    **QueryWrapper ： Entity 对象封装操作类，不是用lambda语法**

  ​    UpdateWrapper ： Update 条件封装，用于Entity对象更新操作

    AbstractLambdaWrapper ： Lambda 语法使用 Wrapper统一处理解析 lambda 获取 column。

  ​    LambdaQueryWrapper ：看名称也能明白就是用于Lambda语法使用的查询Wrapper

  ​    LambdaUpdateWrapper ： Lambda 更新封装Wrapper

* **注意：**以下条件构造器的方法入参中的 `column `均表示数据库字段

  ## **1、ge、gt、le、lt、isNull、isNotNull**

  ```java
  @Test
  public void testDelete() {
      QueryWrapper<User> queryWrapper = new QueryWrapper<>();
      queryWrapper
          .isNull("name")
          .ge("age", 12)  // age >= 12
          .isNotNull("email");
      int result = userMapper.delete(queryWrapper);
      System.out.println("delete return count = " + result);
  }
  ```

  SQL：UPDATE user SET deleted=1 WHERE deleted=0 AND name IS NULL AND age >= ? AND email IS NOT NULL

  ## **2、eq、ne**

  **注意：**seletOne返回的是一条实体记录，当出现多条时会报错

   

  ```java
  @Test
  public void testSelectOne() {
      QueryWrapper<User> queryWrapper = new QueryWrapper<>();
      queryWrapper.nq("name", "Tom"); // ne表示不等于
      User user = userMapper.selectOne(queryWrapper);
      System.out.println(user);
  }
  ```

  SELECT id,name,age,email,create_time,update_time,deleted,version FROM user WHERE deleted=0 AND name = ? 

  ## **3、between、notBetween**

  包含大小边界

   

  ```java
  @Test
  public void testSelectCount() {
      QueryWrapper<User> queryWrapper = new QueryWrapper<>();
      queryWrapper.between("age", 20, 30);
      Integer count = userMapper.selectCount(queryWrapper);
      System.out.println(count);
  }
  ```

  SELECT COUNT(1) FROM user WHERE deleted=0 AND age BETWEEN ? AND ? 

  ## **4、allEq**

   

  ```java
  @Test
  public void testSelectList() {
      QueryWrapper<User> queryWrapper = new QueryWrapper<>();
      Map<String, Object> map = new HashMap<>();
      map.put("id", 2);
      map.put("name", "Jack");
      map.put("age", 20);
      queryWrapper.allEq(map);
      List<User> users = userMapper.selectList(queryWrapper);
      users.forEach(System.out::println);
  }
  ```

  SELECT id,name,age,email,create_time,update_time,deleted,version 

  FROM user WHERE deleted=0 AND name = ? AND id = ? AND age = ? 

  ## **5、like、notLike、likeLeft、likeRight**

  selectMaps返回Map集合列表

   

  ```java
  @Test
  public void testSelectMaps() {
      QueryWrapper<User> queryWrapper = new QueryWrapper<>();
      queryWrapper
          .notLike("name", "e")  // name字段中不包含‘e’的数据，即 %e%
          .likeRight("email", "t");// 即 %t
      List<Map<String, Object>> maps = userMapper.selectMaps(queryWrapper);//返回值是Map列表
      maps.forEach(System.out::println);
  }
  ```

  SELECT id,name,age,email,create_time,update_time,deleted,version 

  FROM user WHERE deleted=0 AND name NOT LIKE ? AND email LIKE ? 

  ## **6、in、notIn、inSql、notinSql、exists、notExists**

  in、notIn：

  ```java
  notIn("age",{1,2,3})--->age not in (1,2,3)notIn("age", 1, 2, 3)--->age not in (1,2,3)
  ```

  inSql、notinSql：可以实现子查询

  - 例: `inSql("age", "1,2,3,4,5,6")`--->`age in (1,2,3,4,5,6)`
  - 例: `inSql("id", "select id from table where id < 3")`--->`id in (select id from table where id < 3)`

   

  ```java
  @Test
  public void testSelectObjs() {
      QueryWrapper<User> queryWrapper = new QueryWrapper<>();
      //queryWrapper.in("id", 1, 2, 3);
      queryWrapper.inSql("id", "select id from user where id < 3");
      List<Object> objects = userMapper.selectObjs(queryWrapper);//返回值是Object列表
      objects.forEach(System.out::println);
  }
  ```

  SELECT id,name,age,email,create_time,update_time,deleted,version 

  FROM user WHERE deleted=0 AND id IN (select id from user where id < 3) 

  ## **7、or、and**

  **注意：**这里使用的是 UpdateWrapper 

  不调用`or`则默认为使用 `and `连

   

  ```java
  @Test
  public void testUpdate1() {
      //修改值
      User user = new User();
      user.setAge(99);
      user.setName("Andy");
      //修改条件
      UpdateWrapper<User> userUpdateWrapper = new UpdateWrapper<>();
      userUpdateWrapper
          .like("name", "h")
          .or()
          .between("age", 20, 30);
      int result = userMapper.update(user, userUpdateWrapper);
      System.out.println(result);
  }
  ```

  UPDATE user SET name=?, age=?, update_time=? WHERE deleted=0 AND name LIKE ? OR age BETWEEN ? AND ?

  ## **8、嵌套or、嵌套and**

  这里使用了lambda表达式，or中的表达式最后翻译成sql时会被加上圆括号

   

  ```java
  @Test
  public void testUpdate2() {
      //修改值
      User user = new User();
      user.setAge(99);
      user.setName("Andy");
      //修改条件
      UpdateWrapper<User> userUpdateWrapper = new UpdateWrapper<>();
      userUpdateWrapper
          .like("name", "h")
          .or(i -> i.eq("name", "李白").ne("age", 20));
      int result = userMapper.update(user, userUpdateWrapper);
      System.out.println(result);
  }
  ```

  UPDATE user SET name=?, age=?, update_time=? 

  WHERE deleted=0 AND name LIKE ? 

  OR ( name = ? AND age <> ? ) 

  ## **9、orderBy、orderByDesc、orderByAsc**

   

  ```java
  @Test
  public void testSelectListOrderBy() {
      QueryWrapper<User> queryWrapper = new QueryWrapper<>();
      queryWrapper.orderByDesc("id");
      List<User> users = userMapper.selectList(queryWrapper);
      users.forEach(System.out::println);
  }
  ```

  SELECT id,name,age,email,create_time,update_time,deleted,version 

  FROM user WHERE deleted=0 ORDER BY id DESC 

  ## **10、last**

  直接拼接到 sql 的最后

  **注意：**只能调用一次,多次调用以最后一次为准 有sql注入的风险,请谨慎使用

  ```java
@Test
  public void testSelectListLast() {
      QueryWrapper<User> queryWrapper = new QueryWrapper<>();
      queryWrapper.last("limit 1"); // 改字符串拼接在sql语句的最后
      List<User> users = userMapper.selectList(queryWrapper);
      users.forEach(System.out::println);
  }
  ```
  
  SELECT id,name,age,email,create_time,update_time,deleted,version  FROM user WHERE deleted=0 limit 1 

  ## **11、**指定要查询的列

  ```java
@Test
  public void testSelectListColumn() {
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
      queryWrapper.select("id", "name", "age"); // 即select id, name, age ...
      List<User> users = userMapper.selectList(queryWrapper);
      users.forEach(System.out::println);
  }
  ```
  
  SELECT id,name,age FROM user WHERE deleted=0 
  
  ## **12、set、setSql**

  最终的sql会合并 user.setAge()，以及 userUpdateWrapper.set()  和 setSql() 中 的字段

  ```java
@Test
  public void testUpdateSet() {
    //修改值
      User user = new User();
    user.setAge(99);
      //修改条件
      UpdateWrapper<User> userUpdateWrapper = new UpdateWrapper<>();
      userUpdateWrapper
          .like("name", "h")
          .set("name", "老李头")//除了可以查询还可以使用set设置修改的字段
          .setSql(" email = '123@qq.com'");//可以有子查询
      int result = userMapper.update(user, userUpdateWrapper);
  }
  ```
  
  UPDATE user SET age=?, update_time=?, name=?, email = '123@qq.com' WHERE deleted=0 AND name LIKE ? 

## 代码生成器

* 引入依赖

  ```
          <!-- velocity 模板引擎, Mybatis Plus 代码生成器需要 -->
          <dependency>
              <groupId>org.apache.velocity</groupId>
              <artifactId>velocity-engine-core</artifactId>
          </dependency>
  ```

* ```java
  public void run() {
  
          // 1、创建代码生成器
          AutoGenerator mpg = new AutoGenerator();
  
          // 2、全局配置
          GlobalConfig gc = new GlobalConfig();
          String projectPath = System.getProperty("user.dir"); // 项目目录,但可能会出错，最好是绝对路径
          gc.setOutputDir("D:\\JAVA\\Java_Learning\\IDEA_Project02\\guli_parent\\service\\service_edu" + "/src/main/java");
  
          gc.setAuthor("testjava");
          gc.setOpen(false); //生成后是否打开资源管理器
          gc.setFileOverride(false); //重新生成时文件是否覆盖
  
          gc.setServiceName("%sService");	//去掉Service接口的首字母I
          gc.setIdType(IdType.ID_WORKER_STR); //主键策略
          gc.setDateType(DateType.ONLY_DATE);//定义生成的实体类中日期类型
          gc.setSwagger2(true);//开启Swagger2模式
  
          mpg.setGlobalConfig(gc);
  
          // 3、数据源配置
          DataSourceConfig dsc = new DataSourceConfig();
          dsc.setUrl("jdbc:mysql://localhost:3306/guli?serverTimezone=GMT%2B8");
          dsc.setDriverName("com.mysql.cj.jdbc.Driver");
          dsc.setUsername("root");
          dsc.setPassword("root");
          dsc.setDbType(DbType.MYSQL);
          mpg.setDataSource(dsc);
  
          // 4、包配置
          PackageConfig pc = new PackageConfig();
          pc.setParent("com.atguigu");
          pc.setModuleName("eduservice"); //模块名
          pc.setController("controller");
          pc.setEntity("entity");
          pc.setService("service");
          pc.setMapper("mapper");
          mpg.setPackageInfo(pc);
  
          // 5、策略配置
          StrategyConfig strategy = new StrategyConfig();
          strategy.setInclude("edu_teacher");
          strategy.setNaming(NamingStrategy.underline_to_camel);//数据库表映射到实体的命名策略
          strategy.setTablePrefix(pc.getModuleName() + "_"); //生成实体时去掉表前缀
  
          strategy.setColumnNaming(NamingStrategy.underline_to_camel);//数据库表字段映射到实体的命名策略
          strategy.setEntityLombokModel(true); // lombok 模型 @Accessors(chain = true) setter链式操作
  
          strategy.setRestControllerStyle(true); //restful api风格控制器
          strategy.setControllerMappingHyphenStyle(true); //url中驼峰转连字符
  
          mpg.setStrategy(strategy);
  
  
          // 6、执行
          mpg.execute();
      }
  ```


## Wrapper

```java

/**
 * 通过单个ID主键进行查询
 */
@Test
public void selectById() {
        User user = userMapper.selectById(1094592041087729666L);
        System.out.println(user);
        }

/**
 * 通过多个ID主键查询
 */
@Test
public void selectByList() {
        List<Long> longs = Arrays.asList(1094592041087729666L, 1094590409767661570L);
        List<User> users = userMapper.selectBatchIds(longs);
        users.forEach(System.out::println);
        }

/**
 * 通过Map参数进行查询
 */
@Test
public void selectByMap() {
        Map<String, Object> params = new HashMap<>();
        params.put("name", "张雨琪");
        List<User> users = userMapper.selectByMap(params);
        users.forEach(System.out::println);
        }
        MyBatis-Plus还提供了Wrapper条件构造器，具体使用看如下代码：

/**
 * 名字包含雨并且年龄小于40
 * <p>
 * WHERE name LIKE '%雨%' AND age < 40
 */
@Test
public void selectByWrapperOne() {
        QueryWrapper<User> wrapper = new QueryWrapper();
        wrapper.like("name", "雨").lt("age", 40);
        List<User> users = userMapper.selectList(wrapper);
        users.forEach(System.out::println);
        }

/**
 * 名字包含雨
 * 年龄大于20小于40
 * 邮箱不能为空
 * <p>
 * WHERE name LIKE '%雨%' AND age BETWEEN 20 AND 40 AND email IS NOT NULL
 */
@Test
public void selectByWrapperTwo() {
        QueryWrapper<User> wrapper = Wrappers.query();
        wrapper.like("name", "雨").between("age", 20, 40).isNotNull("email");
        List<User> users = userMapper.selectList(wrapper);
        users.forEach(System.out::println);
        }

/**
 * 名字为王性
 * 或者年龄大于等于25
 * 按照年龄降序排序，年龄相同按照id升序排序
 * <p>
 * WHERE name LIKE '王%' OR age >= 25 ORDER BY age DESC , id ASC
 */
@Test
public void selectByWrapperThree() {
        QueryWrapper<User> wrapper = Wrappers.query();
        wrapper.likeRight("name", "王").or()
        .ge("age", 25).orderByDesc("age").orderByAsc("id");
        List<User> users = userMapper.selectList(wrapper);
        users.forEach(System.out::println);
        }

/**
 * 查询创建时间为2019年2月14
 * 并且上级领导姓王
 * <p>
 * WHERE date_format(create_time,'%Y-%m-%d') = '2019-02-14' AND manager_id IN (select id from user where name like '王%')
 */
@Test
public void selectByWrapperFour() {
        QueryWrapper<User> wrapper = Wrappers.query();
        wrapper.apply("date_format(create_time,'%Y-%m-%d') = {0}", "2019-02-14")
        .inSql("manager_id", "select id from user where name like '王%'");
        List<User> users = userMapper.selectList(wrapper);
        users.forEach(System.out::println);
        }

/**
 * 查询王姓
 * 并且年龄小于40或者邮箱不为空
 * <p>
 * WHERE name LIKE '王%' AND ( age < 40 OR email IS NOT NULL )
 */
@Test
public void selectByWrapperFive() {
        QueryWrapper<User> wrapper = Wrappers.query();
        wrapper.likeRight("name", "王").and(qw -> qw.lt("age", 40).or().isNotNull("email"));
        List<User> users = userMapper.selectList(wrapper);
        users.forEach(System.out::println);
        }

/**
 * 查询王姓
 * 并且年龄大于20 、年龄小于40、邮箱不能为空
 * <p>
 * WHERE name LIKE ? OR ( age BETWEEN ? AND ? AND email IS NOT NULL )
 */
@Test
public void selectByWrapperSix() {
        QueryWrapper<User> wrapper = Wrappers.query();
        wrapper.likeRight("name", "王").or(
        qw -> qw.between("age", 20, 40).isNotNull("email")
        );
        List<User> users = userMapper.selectList(wrapper);
        users.forEach(System.out::println);
        }

/**
 * (年龄小于40或者邮箱不为空) 并且名字姓王
 * WHERE ( age < 40 OR email IS NOT NULL ) AND name LIKE '王%'
 */
@Test
public void selectByWrapperSeven() {
        QueryWrapper<User> wrapper = Wrappers.query();
        wrapper.nested(qw -> qw.lt("age", 40).or().isNotNull("email"))
        .likeRight("name", "王");
        List<User> users = userMapper.selectList(wrapper);
        users.forEach(System.out::println);
        }

/**
 * 查询年龄为30、31、32
 * WHERE age IN (?,?,?)
 */
@Test
public void selectByWrapperEight() {
        QueryWrapper<User> wrapper = Wrappers.query();
        wrapper.in("age", Arrays.asList(30, 31, 32));
        List<User> users = userMapper.selectList(wrapper);
        users.forEach(System.out::println);
        }

/**
 * 查询一条数据
 * limit 1
 */
@Test
public void selectByWrapperNine() {
        QueryWrapper<User> wrapper = Wrappers.query();
        wrapper.in("age", Arrays.asList(30, 31, 32)).last("limit 1");
        List<User> users = userMapper.selectList(wrapper);
        users.forEach(System.out::println);
        }
 
```

### 获取spu的规格参数信息

```xml
    <resultMap id="spuItemAttrGroupVo" type="com.atguigu.gulimall.product.vo.SpuItemAttrGroup">
        <result column="attr_group_name" property="groupName"></result>
        <collection property="attrs" ofType="com.atguigu.gulimall.product.vo.Attr">
            <result column="attr_name" property="attrName"></result>
            <result column="attr_value" property="attrValue"></result>
        </collection>
    </resultMap>

    <select id="getAttrGroupWithAttrsBySpuId"
            resultType="spuItemAttrGroupVo">

            SELECT
                pav.spu_id,
                ag.attr_group_name,
                ag.attr_group_id,
                aar.attr_id,
                pav.attr_name,
                pav.attr_value
            FROM pms_attr_group ag
            LEFT JOIN pms_attr_attrgroup_relation aar on ag.attr_group_id=aar.attr_group_id
            LEFT JOIN pms_product_attr_value pav on pav.attr_id = aar.attr_id
            WHERE ag.catelog_id=#{catalogId} AND pav.spu_id=#{spuId}

    </select>
```

























