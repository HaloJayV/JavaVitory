[TOC]

# 一、Spring Security介绍

**1、框架介绍**

Spring 是一个非常流行和成功的 Java 应用开发框架。Spring Security 基于 Spring 框架，提供了一套 Web 应用安全性的完整解决方案。一般来说，Web 应用的安全性包括**用户认证****（****Authentication****）和用户授权（****Authorization****）**两个部分。

（1）用户认证指的是：验证某个用户是否为系统中的合法主体，也就是说用户能否访问该系统。用户认证一般要求用户提供用户名和密码。系统通过校验用户名和密码来完成认证过程。

（2）用户授权指的是验证某个用户是否有权限执行某个操作。在一个系统中，不同用户所具有的权限是不同的。比如对一个文件来说，有的用户只能进行读取，而有的用户可以进行修改。一般来说，系统会为不同的用户分配不同的角色，而每个角色则对应一系列的权限。

**Spring Security其实就是用filter，多请求的路径进行过滤。**

（1）如果是基于Session，那么Spring-security会对cookie里的sessionid进行解析，找到服务器存储的sesion信息，然后判断当前用户是否符合请求的要求。

（2）如果是token，则是解析出token，然后将当前请求加入到Spring-security管理的权限信息中去

**2、认证与授权实现思路**

如果系统的模块众多，每个模块都需要授权与认证，所以我们选择基于token的形式进行授权与认证，用户根据用户名密码认证成功，然后获取当前用户角色的一系列权限值，并以用户名为key，权限列表为value的形式存入redis缓存中，根据用户名相关信息生成token返回，浏览器将token记录到cookie中，每次调用api接口都默认将cookie中的token携带到header请求头中，Spring-security解析header头从而获取token信息，再解析token获取当前用户名，根据用户名就可以从redis中获取权限列表，这样Spring-security就能够判断当前请求是否有权限访问

Spring Security 支持两种不同的认证方式：

- 可以通过 form 表单来认证
- 可以通过 HttpBasic 来认证

## 二、整合Spring Security

**1、在common下创建spring_security模块**

![img](../../../../Software/Typora/Picture/423034c7-c21a-4555-8380-393ac7ebca4d.png)

**2、在spring_security引入相关依赖**

```
<dependencies>
    <!-- Spring Security依赖 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt</artifactId>
    </dependency>
</dependencies>
```

***\3、在service_acl引入\*\*spring_security\*\*依赖\****

```
<dependency>
    <groupId>com.atguigu</groupId>
    <artifactId>spring_security</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```

**0、代码结构说明：**

![img](../../../../Software/Typora/Picture/6aafe337-8a5d-440c-b93d-ff60d4f9cef6.png)
4、创建spring security核心配置类TokenWebSecurityConfig

Spring Security的核心配置就是继承WebSecurityConfigurerAdapter并注解@EnableWebSecurity的配置。

这个配置指明了用户名密码的处理方式、请求路径的开合、登录登出控制等和安全相关的配置

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class TokenWebSecurityConfig extends WebSecurityConfigurerAdapter {
    private UserDetailsService userDetailsService;
    private TokenManager tokenManager;
    private DefaultPasswordEncoder defaultPasswordEncoder;
    private RedisTemplate redisTemplate;
    @Autowired
    public TokenWebSecurityConfig(UserDetailsService userDetailsService, DefaultPasswordEncoder defaultPasswordEncoder,
                                  TokenManager tokenManager, RedisTemplate redisTemplate) {
        this.userDetailsService = userDetailsService;
        this.defaultPasswordEncoder = defaultPasswordEncoder;
        this.tokenManager = tokenManager;
        this.redisTemplate = redisTemplate;
    }
    /**
     * 配置设置
     * @param http
     * @throws Exception
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.exceptionHandling()
                .authenticationEntryPoint(new UnauthorizedEntryPoint())
                .and().csrf().disable()
                .authorizeRequests()
                .anyRequest().authenticated()
                .and().logout().logoutUrl("/admin/acl/index/logout")
                .addLogoutHandler(new TokenLogoutHandler(tokenManager,redisTemplate)).and()
                .addFilter(new TokenLoginFilter(authenticationManager(), tokenManager, redisTemplate))
                .addFilter(new TokenAuthenticationFilter(authenticationManager(), tokenManager, redisTemplate)).httpBasic();
    }
    /**
     * 密码处理
     * @param auth
     * @throws Exception
     */
    @Override
    public void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService).passwordEncoder(defaultPasswordEncoder);
    }
    /**
     * 配置哪些请求不拦截
     * @param web
     * @throws Exception
     */
    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers("/api/**",
                "/swagger-resources/**", "/webjars/**", "/v2/**", "/swagger-ui.html/**"
               );
    }
}
```

**5、创建认证授权相关的工具类**

![img](../../../../Software/Typora/Picture/75ccbeef-10c1-45fd-8f68-b598019832b1.png)

（1）DefaultPasswordEncoder：密码处理的方法

```java
package com.atguigu.serurity.security;
import com.atguigu.commonutils.utils.MD5;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Component;
/**
 * <p>
 * 密码的处理方法类型
 * </p>
 */
@Component
public class DefaultPasswordEncoder implements PasswordEncoder {
    public DefaultPasswordEncoder() {
        this(-1);
    }
    /**
     * @param strength
     *            the log rounds to use, between 4 and 31
     */
    public DefaultPasswordEncoder(int strength) {
    }
    public String encode(CharSequence rawPassword) {
        return MD5.encrypt(rawPassword.toString()); // 密码加密
    }
    public boolean matches(CharSequence rawPassword, String encodedPassword) {
        return encodedPassword.equals(MD5.encrypt(rawPassword.toString())); // 密码认证
    }
}
```

（2）`TokenManager：token操作的工具类`

```java
/**
 * <p>
 * token管理
 * </p>
 */
@Component
public class TokenManager {
    private long tokenExpiration = 24*60*60*1000;
    private String tokenSignKey = "123456";
    // 根据用户名创建token值
    public String createToken(String username) {
        String token = Jwts.builder().setSubject(username)
                .setExpiration(new Date(System.currentTimeMillis() + tokenExpiration))
                .signWith(SignatureAlgorithm.HS512, tokenSignKey).compressWith(CompressionCodecs.GZIP).compact();
        return token;
    }
    // 获取到请求头的token，并解析获取到对应用户
    public String getUserFromToken(String token) {
        String user = Jwts.parser().setSigningKey(tokenSignKey).parseClaimsJws(token).getBody().getSubject();
        return user;
    }
    public void removeToken(String token) {
        //jwttoken无需删除，客户端扔掉即可。
    }
}
```

（3）`TokenLogoutHandler：退出实现`

```java
/**
 * <p>
 * 登出业务逻辑类
 * </p>
 */
public class TokenLogoutHandler implements LogoutHandler {
    private TokenManager tokenManager;
    private RedisTemplate redisTemplate;
    
    public TokenLogoutHandler(TokenManager tokenManager, RedisTemplate redisTemplate) {
        this.tokenManager = tokenManager;
        this.redisTemplate = redisTemplate;
    }
    // 退出登录处理
    @Override
    public void logout(HttpServletRequest request, HttpServletResponse response, Authentication authentication) {
        String token = request.getHeader("token");
        if (token != null) {
            tokenManager.removeToken(token);
            //清空当前用户缓存中的权限数据
            String userName = tokenManager.getUserFromToken(token);
            redisTemplate.delete(userName);
        }
        ResponseUtil.out(response, R.ok());
    }
}
```

（4）`UnauthorizedEntryPoint：未授权统一处理`

```java
import com.atguigu.commonutils.R;
import com.atguigu.commonutils.utils.ResponseUtil;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.AuthenticationEntryPoint;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
/**
 * <p>
 * 未授权的统一处理方式
 * </p>
 */
public class UnauthorizedEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response,
                         AuthenticationException authException) throws IOException, ServletException {
        ResponseUtil.out(response, R.error());
    }
}
```

#### 6、创建认证授权实体类

![img](../../../../Software/Typora/Picture/76236594-2238-4d28-a15e-7cf743bb53e2.png)

**（1）SecutityUser**

```java
/**
 * <p>
 * 安全认证用户详情信息
 * </p>
 */
@Data
@Slf4j
public class SecurityUser implements UserDetails {
    //当前登录用户
    private transient User currentUserInfo;  // 禁止序列化user
    //当前权限
    private List<String> permissionValueList;
    
    public SecurityUser() {
    }
    public SecurityUser(User user) {
        if (user != null) {
            this.currentUserInfo = user;
        }
    }
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        Collection<GrantedAuthority> authorities = new ArrayList<>();
        for(String permissionValue : permissionValueList) {
            if(StringUtils.isEmpty(permissionValue)) continue;
            SimpleGrantedAuthority authority = new SimpleGrantedAuthority(permissionValue);
            authorities.add(authority);
        }
        return authorities; // 返回当前用户权限信息
    }
    @Override
    public String getPassword() {
        return currentUserInfo.getPassword();
    }
    @Override
    public String getUsername() {
        return currentUserInfo.getUsername();
    }
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }
    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

**（2）User**

```java
/**
 * <p>
 * 用户实体类
 * </p>
 */
@Data
@ApiModel(description = "用户实体类")
public class User implements Serializable {
    private static final long serialVersionUID = 1L;
    @ApiModelProperty(value = "微信openid")
    private String username;
    @ApiModelProperty(value = "密码")
    private String password;
    @ApiModelProperty(value = "昵称")
    private String nickName;
    @ApiModelProperty(value = "用户头像")
    private String salt;
    @ApiModelProperty(value = "用户签名")
    private String token;
}
```

**7、创建认证和授权的filter
![img](../../../../Software/Typora/Picture/bee4d171-befe-4b08-a792-5f1d50a8642b.png)
（1）TokenLoginFilter：认证的filter**

```java
/**
 * <p>
 * 登录过滤器，继承UsernamePasswordAuthenticationFilter，对用户名密码进行登录校验
 * </p>
 */
public class TokenLoginFilter extends UsernamePasswordAuthenticationFilter {
    private AuthenticationManager authenticationManager;
    private TokenManager tokenManager;
    private RedisTemplate redisTemplate;
    public TokenLoginFilter(AuthenticationManager authenticationManager, TokenManager tokenManager, RedisTemplate redisTemplate) {
        this.authenticationManager = authenticationManager;
        this.tokenManager = tokenManager;
        this.redisTemplate = redisTemplate;
        this.setPostOnly(false);
        this.setRequiresAuthenticationRequestMatcher(new AntPathRequestMatcher("/admin/acl/login","POST"));
    }
    
    @Override
    public Authentication attemptAuthentication(HttpServletRequest req, HttpServletResponse res)
            throws AuthenticationException {
        try {
            User user = new ObjectMapper().readValue(req.getInputStream(), User.class);
            return authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(user.getUsername(), user.getPassword(), new ArrayList<>()));
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
    /**
     * 登录成功
     * @param req
     * @param res
     * @param chain
     * @param auth
     * @throws IOException
     * @throws ServletException
     */
    @Override
    protected void successfulAuthentication(HttpServletRequest req, HttpServletResponse res, FilterChain chain,
                                            Authentication auth) throws IOException, ServletException {
        SecurityUser user = (SecurityUser) auth.getPrincipal();
        String token = tokenManager.createToken(user.getCurrentUserInfo().getUsername());
        redisTemplate.opsForValue().set(user.getCurrentUserInfo().getUsername(), user.getPermissionValueList());
        ResponseUtil.out(res, R.ok().data("token", token));
    }
    /**
     * 登录失败
     * @param request
     * @param response
     * @param e
     * @throws IOException
     * @throws ServletException
     */
    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response,
                                              AuthenticationException e) throws IOException, ServletException {
        ResponseUtil.out(response, R.error());
    }
}
```

**`（2）TokenAuthenticationFilter：`**

```java
/**
 * <p>
 * 访问过滤器
 * </p>
 */
public class TokenAuthenticationFilter extends BasicAuthenticationFilter {
    private TokenManager tokenManager;
    private RedisTemplate redisTemplate;
    public TokenAuthenticationFilter(AuthenticationManager authManager, TokenManager tokenManager,RedisTemplate redisTemplate) {
        super(authManager);
        this.tokenManager = tokenManager;
        this.redisTemplate = redisTemplate;
    }
    @Override
    protected void doFilterInternal(HttpServletRequest req, HttpServletResponse res, FilterChain chain)
            throws IOException, ServletException {
        logger.info("================="+req.getRequestURI());
        if(req.getRequestURI().indexOf("admin") == -1) {
            chain.doFilter(req, res);
            return;
        }
        UsernamePasswordAuthenticationToken authentication = null;
        try {
            authentication = getAuthentication(req);
        } catch (Exception e) {
            ResponseUtil.out(res, R.error());
        }
        if (authentication != null) {
            SecurityContextHolder.getContext().setAuthentication(authentication);
        } else {
            ResponseUtil.out(res, R.error());
        }
        chain.doFilter(req, res);
    }
    private UsernamePasswordAuthenticationToken getAuthentication(HttpServletRequest request) {
        // token置于header里
        String token = request.getHeader("token");
        if (token != null && !"".equals(token.trim())) {
            String userName = tokenManager.getUserFromToken(token);
            List<String> permissionValueList = (List<String>) redisTemplate.opsForValue().get(userName);
            Collection<GrantedAuthority> authorities = new ArrayList<>();
            for(String permissionValue : permissionValueList) {
                if(StringUtils.isEmpty(permissionValue)) continue;
                SimpleGrantedAuthority authority = new SimpleGrantedAuthority(permissionValue);
                authorities.add(authority);
            }
            if (!StringUtils.isEmpty(userName)) {
                return new UsernamePasswordAuthenticationToken(userName, token, authorities);
            }
            return null;
        }
        return null;
    }
}
```



# 底层原理

### 核心组件

1.    SecurityContextHolder：提供对SecurityContext的访问
2.    SecurityContext,：持有Authentication对象和其他可能需要的信息
3.    AuthenticationManager 其中可以包含多个AuthenticationProvider
4.    ProviderManager对象为AuthenticationManager接口的实现类
5.    AuthenticationProvider 主要用来进行认证操作的类 调用其中的authenticate()方法去进行认证操作
6.    Authentication：Spring Security方式的认证主体
7.    GrantedAuthority：对认证主题的应用层面的授权，含当前用户的权限信息，通常使用角色表示
8.    UserDetails：构建Authentication对象必须的信息，可以自定义，可能需要访问DB得到
9.    UserDetailsService：通过username构建UserDetails对象，通过loadUserByUsername根据userName获取UserDetail对象 （可以在这里基于自身业务进行自定义的实现 如通过数据库，xml,缓存获取等）

### 主要过滤器

想要对对Web资源进行保护，最好的办法莫过于Filter，要想对方法调用进行保护，最好的办法莫过于AOP。所以springSecurity在我们进行用户认证以及授予权限的时候，通过各种各样的拦截器来控制权限的访问，从而实现安全。

1. ​    WebAsyncManagerIntegrationFilter 
2. ​     SecurityContextPersistenceFilter 
3. ​     HeaderWriterFilter 
4. ​     CorsFilter 
5. ​     LogoutFilter
6. ​     RequestCacheAwareFilter
7. ​     SecurityContextHolderAwareRequestFilter
8. ​     AnonymousAuthenticationFilter
9. ​     SessionManagementFilter
10. ​     ExceptionTranslationFilter
11. ​     FilterSecurityInterceptor
12. ​     UsernamePasswordAuthenticationFilter
13. ​     BasicAuthenticationFilter

## 自定义安全机制的加载机制

自定义了一个springSecurity安全框架的配置类 继承WebSecurityConfigurerAdapter，重写其中的方法configure

实现该类后，在web容器启动的过程中该类实例对象会被WebSecurityConfiguration类处理。

* 继承WebSecurityConfigurerAdapter

```java
@Configuration
public class SpringSecurityConfig extends WebSecurityConfigurerAdapter{

    //重写了其中的configure（）方法设置了不同url的不同访问权限
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .authorizeRequests()
                .antMatchers("/home", "/about","/img/*").permitAll()
                .antMatchers("/admin/**","/upload/**").hasAnyRole("ADMIN")
                .antMatchers("/order/**").hasAnyRole("USER","ADMIN")
                .antMatchers("/room/**").hasAnyRole("USER","ADMIN")
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .loginPage("/login")
                .permitAll()
                .and()
                .logout()
                .permitAll()
                .and()
                .exceptionHandling().accessDeniedHandler(accessDeniedHandler);
    }
}
```

* ##### WebSecurityConfiguration

  ```java
  @Configuration
  public class WebSecurityConfiguration implements ImportAware, BeanClassLoaderAware {
      private WebSecurity webSecurity;
      private Boolean debugEnabled;
      private List<SecurityConfigurer<Filter, WebSecurity>> webSecurityConfigurers;
      private ClassLoader beanClassLoader;
     
     ...省略部分代码
  
      @Bean(
          name = {"springSecurityFilterChain"}
      )
      public Filter springSecurityFilterChain() throws Exception {
          boolean hasConfigurers = this.webSecurityConfigurers != null
           && !this.webSecurityConfigurers.isEmpty();
          if(!hasConfigurers) {
              WebSecurityConfigurerAdapter adapter = (WebSecurityConfigurerAdapter)
              this.objectObjectPostProcessor
                .postProcess(new WebSecurityConfigurerAdapter() {
              });
              this.webSecurity.apply(adapter);
          }
  
          return (Filter)this.webSecurity.build();
      }
      /*1、先执行该方法将我们自定义springSecurity配置实例
         （可能还有系统默认的有关安全的配置实例 ） 配置实例中含有我们自定义业务的权限控制配置信息
         放入到该对象的list数组中webSecurityConfigurers中
         使用@Value注解来将实例对象作为形参注入
       */   
   @Autowired(
          required = false
      )
      public void setFilterChainProxySecurityConfigurer(ObjectPostProcessor<Object> 
      objectPostProcessor,
     @Value("#{@autowiredWebSecurityConfigurersIgnoreParents.getWebSecurityConfigurers()}") 
    List<SecurityConfigurer<Filter, WebSecurity>> webSecurityConfigurers) 
  throws Exception {
      
      //创建一个webSecurity对象    
      this.webSecurity = (WebSecurity)objectPostProcessor.postProcess(new WebSecurity(objectPostProcessor));
          if(this.debugEnabled != null) {
              this.webSecurity.debug(this.debugEnabled.booleanValue());
          }
  
          //对所有配置类的实例进行排序
          Collections.sort(webSecurityConfigurers, WebSecurityConfiguration.AnnotationAwareOrderComparator.INSTANCE);
          Integer previousOrder = null;
          Object previousConfig = null;
  
  
          //迭代所有配置类的实例 判断其order必须唯一
          Iterator var5;
          SecurityConfigurer config;
          for(var5 = webSecurityConfigurers.iterator(); var5.hasNext(); previousConfig = config) {
              config = (SecurityConfigurer)var5.next();
              Integer order = Integer.valueOf(WebSecurityConfiguration.AnnotationAwareOrderComparator.lookupOrder(config));
              if(previousOrder != null && previousOrder.equals(order)) {
                  throw new IllegalStateException("@Order on WebSecurityConfigurers must be unique. Order of " + order + " was already used on " + previousConfig + ", so it cannot be used on " + config + " too.");
              }
  
              previousOrder = order;
          }
  
          //将所有的配置实例添加到创建的webSecutity对象中
          var5 = webSecurityConfigurers.iterator();
          while(var5.hasNext()) {
              config = (SecurityConfigurer)var5.next();
              this.webSecurity.apply(config);
          }
          //将webSercurityConfigures 实例放入该对象的webSecurityConfigurers属性中
          this.webSecurityConfigurers = webSecurityConfigurers;
      }
  }
  ```

## SecurityContextHolder

这是一个工具类，只提供一些静态方法。这个工具类的目的是用来保存应用程序中当前使用人的安全上下文。

**作用：**保留系统当前的安全上下文细节，其中就包括当前使用系统的用户的信息。

（1）单机系统，即应用从开启到关闭的整个生命周期只有一个用户在使用。由于整个应用只需要保存一个SecurityContext（安全上下文即可）

（2）多用户系统，比如典型的Web系统，整个生命周期可能同时有多个用户在使用。这时候应用需要保存多个SecurityContext（安全上下文），需要利用ThreadLocal进行保存，每个线程都可以利用ThreadLocal获取其自己的SecurityContext，及安全上下文。



### 缺省工作模式 `MODE_THREADLOCAL`

一个应用同时可能有多个使用者，每个使用者对应不同的安全上下文。缺省情况下，`SecurityContextHolder`使用了`ThreadLocal`机制来保存每个使用者的安全上下文。这意味着，只要针对某个使用者的逻辑执行都是在同一个线程中进行，即使不在各个方法之间以参数的形式传递其安全上下文，各个方法也能通过`SecurityContextHolder`工具获取到该安全上下文。只要在处理完当前使用者的请求之后注意清除`ThreadLocal`中的安全上下文，这种使用`ThreadLocal`的方式是很安全的。当然在`Spring Security`中，这些工作已经被`Spring Security`自动处理，开发人员不用担心这一点。

`SecurityContextHolder`基于`ThreadLocal`的工作方式天然很适合`Servlet Web`应用，因为缺省情况下根据`Servlet`规范，一个`Servlet request`的处理不管经历了多少个`Filter`，自始至终都由同一个线程来完成。

注意 : 这里讲的是一个`Servlet request`的处理不管经历了多少个`Filter`，自始至终都由同一个线程来完成;而对于同一个使用者的不同`Servlet request`,它们在服务端被处理时，使用的可不一定是同一个线程(存在由同一个线程处理的可能性但不确保)。

### 其他工作模式

有一些应用并不适合使用`ThreadLocal`模式，那么还能不能使用`SecurityContextHolder`了呢？答案是可以的。`SecurityContextHolder`还提供了其他工作模式。

比如有些应用，像`Java Swing`客户端应用，它就可能希望`JVM`中所有的线程使用同一个安全上下文。此时我们可以在启动阶段将`SecurityContextHolder`配置成全局策略`MODE_GLOBAL`。

还有其他的一些应用会有自己的线程创建，并且希望这些新建线程也能使用创建者的安全上下文。这种效果，可以通过将`SecurityContextHolder`配置成`MODE_INHERITABLETHREADLOCAL`策略达到。

### 获取当前用户信息

在`SecurityContextHolder`中保存的是当前访问者的信息。`Spring Security`使用一个`Authentication`对象来表示这个信息。一般情况下，我们都不需要创建这个对象，在登录过程中，`Spring Security`已经创建了该对象并帮我们放到了`SecurityContextHolder`中。从`SecurityContextHolder`中获取这个对象也是很简单的。比如，获取当前登录用户的用户名，可以这样 :

```java
// 获取安全上下文对象，就是那个保存在 ThreadLocal 里面的安全上下文对象
// 总是不为null(如果不存在，则创建一个authentication属性为null的empty安全上下文对象)
SecurityContext securityContext = SecurityContextHolder.getContext();

// 获取当前认证了的 principal(当事人),或者 request token (令牌)
// 如果没有认证，会是 null,该例子是认证之后的情况
Authentication authentication = securityContext.getAuthentication()

// 获取当事人信息对象，返回结果是 Object 类型，但实际上可以是应用程序自定义的带有更多应用相关信息的某个类型。
// 很多情况下，该对象是 Spring Security 核心接口 UserDetails 的一个实现类，你可以把 UserDetails 想像
// 成我们数据库中保存的一个用户信息到 SecurityContextHolder 中 Spring Security 需要的用户信息格式的
// 一个适配器。
Object principal = authentication.getPrincipal();
if (principal instanceof UserDetails) {
	String username = ((UserDetails)principal).getUsername();
} else {
	String username = principal.toString();
}
```

### 修改`SecurityContextHolder`的工作模式

综上所述，`SecurityContextHolder`可以工作在以下三种模式之一:

- `MODE_THREADLOCAL` (缺省工作模式)

- `MODE_GLOBAL`

- `MODE_INHERITABLETHREADLOCAL`

  修改SecurityContextHolder的工作模式有两种方法 :

  - `SecurityContextHolder`会自动从该系统属性中尝试获取被设定的工作模式：设置一个系统属性(`system.properties`) : `spring.security.strategy`;

  - 程序化方式主动设置工作模式的方法：调用`SecurityContextHolder`静态方法`setStrategyName()`

### 源码

```java
/**
 * 将一个给定的SecurityContext绑定到当前执行线程。
 */
public class SecurityContextHolder {
	// ~ Static fields/initializers
	// =====================================================================================
	// 三种工作模式的定义，每种工作模式对应一种策略
	public static final String MODE_THREADLOCAL = "MODE_THREADLOCAL";
	public static final String MODE_INHERITABLETHREADLOCAL = "MODE_INHERITABLETHREADLOCAL";
	public static final String MODE_GLOBAL = "MODE_GLOBAL";

	// 类加载时首先尝试从环境属性中获取所指定的工作模式
	public static final String SYSTEM_PROPERTY = "spring.security.strategy";	
	private static String strategyName = System.getProperty(SYSTEM_PROPERTY);
	private static SecurityContextHolderStrategy strategy;

	// 初始化计数器,初始为0,
	// 1. 类加载过程中会被初始化一次，此值变为1
	// 2. 此后每次调用 setStrategyName 会对新的策略对象执行一次初始化，相应的该值会增1
	private static int initializeCount = 0;

	static {
		initialize();
	}
	// ~ Methods
	// =====================================================================================

	/**
	 * Explicitly clears the context value from the current thread.
	 */
	public static void clearContext() {
		strategy.clearContext();
	}

	/**
	 * Obtain the current SecurityContext.
	 *
	 * @return the security context (never null)
	 */
	public static SecurityContext getContext() {
		return strategy.getContext();
	}

	/**
	 * Primarily for troubleshooting purposes, this method shows how many times the class
	 * has re-initialized its SecurityContextHolderStrategy.
	 *
	 * @return the count (should be one unless you've called
	 * #setStrategyName(String) to switch to an alternate strategy.
	 */
	public static int getInitializeCount() {
		return initializeCount;
	}

	private static void initialize() {
		if (!StringUtils.hasText(strategyName)) {
			// Set default, 设置缺省工作模式/策略 MODE_THREADLOCAL
			strategyName = MODE_THREADLOCAL;
		}

		if (strategyName.equals(MODE_THREADLOCAL)) {
			strategy = new ThreadLocalSecurityContextHolderStrategy();
		}
		else if (strategyName.equals(MODE_INHERITABLETHREADLOCAL)) {
			strategy = new InheritableThreadLocalSecurityContextHolderStrategy();
		}
		else if (strategyName.equals(MODE_GLOBAL)) {
			strategy = new GlobalSecurityContextHolderStrategy();
		}
		else {
			// Try to load a custom strategy
			try {
				Class<?> clazz = Class.forName(strategyName);
				Constructor<?> customStrategy = clazz.getConstructor();
				strategy = (SecurityContextHolderStrategy) customStrategy.newInstance();
			}
			catch (Exception ex) {
				ReflectionUtils.handleReflectionException(ex);
			}
		}

		initializeCount++;
	}

	/**
	 * Associates a new SecurityContext with the current thread of execution.
	 *
	 * @param context the new SecurityContext (may not be null)
	 */
	public static void setContext(SecurityContext context) {
		strategy.setContext(context);
	}

	/**
	 * Changes the preferred strategy. Do NOT call this method more than once for
	 * a given JVM, as it will re-initialize the strategy and adversely affect any
	 * existing threads using the old strategy.
	 *
	 * @param strategyName the fully qualified class name of the strategy that should be
	 * used.
	 */
	public static void setStrategyName(String strategyName) {
		SecurityContextHolder.strategyName = strategyName;
		initialize();
	}

	/**
	 * Allows retrieval of the context strategy. See SEC-1188.
	 *
	 * @return the configured strategy for storing the security context.
	 */
	public static SecurityContextHolderStrategy getContextHolderStrategy() {
		return strategy;
	}

	/**
	 * Delegates the creation of a new, empty context to the configured strategy.
	 */
	public static SecurityContext createEmptyContext() {
		return strategy.createEmptyContext();
	}

	public String toString() {
		return "SecurityContextHolder[strategy='" + strategyName + "'; initializeCount="
				+ initializeCount + "]";
	}
}


```

由源码可知，SecurityContextHolder利用了一个SecurityContextHolderStrategy（存储策略）进行上下文的存储。

SecurityContestHolderStrategy只是一个接口，这个接口提供创建、清空、获取、设置上下文的操作。

1. GlobalSecurityContextHolderStrategy:  全局的上下文存取策略，只存储一个上下文，对应前面说的单机系统。

2. ThreadLocalSecurityContextHolderStrategy:  ThreadLocal内部会用数组来存储多个对象的。原理是，ThreadLocal会为每个线程开辟一个存储区域，来存储相应的对象。

**Authentication——用户信息的表示：**

   在SecurityContextHolder中存储了当前与系统交互的用户的信息。Spring Security使用一个Authentication 对象来表示这些信息。一般不需要自己创建这个对象，但是查找这个对象的操作对用户来说却非常常见。

![image-20201001152800526](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201001152800526.png)

  Principal（准则）=> 允许通过的规则，即允许访问的规则，基本等价于UserDetails（用户信息）

SecurityContext（安全上下文）只是保存了Authentication（认证信息）主要包含了以下内容

- 用户权限集合 => 可用于访问受保护资源时的权限验证
- 用户证书（密码） => 初次认证的时候，进行填充，认证成功后将被清空
- 细节 => 暂不清楚，猜测应该是记录哪些保护资源已经验证授权，下次不用再验证，等等。
- Pirncipal => 大概就是账号吧
- 是否已认证成功



### SpringSecurity底层认证流程

![img](https://img-blog.csdnimg.cn/2020040715130283.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zNzE4OTcyNg==,size_16,color_FFFFFF,t_70)





















