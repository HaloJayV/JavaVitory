[TOC]

#### 基础

* web.xml

```
<!-- SpringMVC思想是有一个前端控制器能拦截所有请求，并智能派发;
  	这个前端控制器是一个servlet；应该在web.xml中配置这个servlet来拦截所有请求-->
   <!-- The front controller of this Spring Web application, 
   responsible for handling all application requests -->
	<servlet>
		<servlet-name>springDispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<!-- contextConfigLocation:指定SpringMVC配置文件位置 -->
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:springmvc.xml</param-value>
		</init-param>
		<!-- servlet启动加载，servlet原本是第一次访问创建对象；load-on-startup:服务器启动的时候创建对象；值越小优先级越高，越先创建对象； -->
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>springDispatcherServlet</servlet-name>
		<!--  /*和/都是拦截所有请求； /：会拦截所有请求，但是不会拦截*.jsp；能保证jsp访问正常；
			/*的范围更大；还会拦截到*.jsp这些请求；一但拦截jsp页面就不能显示了； -->
		<url-pattern>/</url-pattern>
	</servlet-mapping>
```

* SpringMVC.xml

  ```
  <context:component-scan base-package="二级包名"></context:component-scan>
  <!-- 视图解析器可以简化方法的返回值，返回值就算作为目标页面的地址，解析器可以进行返回页面地址的拼串 -->
  <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
  	<property name="prefix" value="/WEB-INF/pages/"></property>
  	<property name="suffix" value=".jsp"></property>
  </bean>
  ```

 

#### 细节

* @RequestMapping可以标在方法和类上

* @RequestMapping("/anTest0?")   // ant风格的url模糊匹配： ?匹配一个字符，*匹配多个字符

* Rest风格url：GET查询，PUT更新修改、DELETE删除、POST添加

* 高版本的Tomcat8以上，在JSP不支持Rest风格的DELETE和PUT，只需要在JSP上加上：
  
* ![image-20200712124453387](../../../../Software/Typora/Picture/image-20200712124453387.png)
  
* @RequestHeader("User-Agent", required=false) : 获取浏览器请求头信息
* @CookieValue("JSESSIONID" , required=false) : 获取浏览器Cookie信息

* `public String handle01(Map<String, Object> map){map.put("msg","123")}`  // 在前台可以接收到request域中的map数据：${requestScope.msg}，Map的类型是BindingAwareModelMap
* `public String handle01(Model model){model.addAttribute("msg","123")}`  // 在前台可以接收到request域中的model数据：${requestScope.msg}, Model是一个Spring里的接口，Model 类型是BindingAwareModelMap

* `public String handle01(ModelMap modelMap){modelMap.addAttribute("msg","123")}`  // 在前台可以接收到request域中的modelMap数据：${requestScope.msg}，ModelMap 类型是BindingAwareModelMap， 继承LinkedHashMap

* ```
  ModelAndView mv = new ModelAndView("success");  // success为该Controller的返回页面
  mv.addObject("msg", "OK");  // 数据放在请求域中
  return mv;
  ```




#### @ModelAttribute

* @ModelAttribute放在类上，会提前查出数据库的数据，提前于目标方法先运行

* ModelAttribute放在参数里：@ModelAttribute("user") User user ：该参数user可以在不同方法中数据互联 

* 凡是@ModelAttribute修饰过的，在不同方法之间，Model、Map、ModelMap 保存的数据都相等。因为都是底层都是 BindingAwareModelMap(隐含模型)

#### 转发和重定向

* 转发： 

  ```
  return "forward:/success.jsp";
  ```

* 重定向：

  ```
  return "redirect:/success.jsp";
  ```

  

#### 数据绑定

* ModelAttributeMethodProcessor的resolveArgument方法。将ServletRequest请求带过来的参数与JavaBean对象绑定在一起

```
	public final Object resolveArgument(
            MethodParameter parameter, ModelAndViewContainer mavContainer,
            NativeWebRequest request, WebDataBinderFactory binderFactory)
            throws Exception {
        String name = ModelFactory.getNameForParameter(parameter);
        Object attribute = (mavContainer.containsAttribute(name)) ?
                mavContainer.getModel().get(name) : createAttribute(name, parameter, binderFactory, request);
                // 数据绑定器
        WebDataBinder binder = binderFactory.createBinder(request, attribute, name);

        if (binder.getTarget() != null) {
               将页面提交过来的数据封装到javaBean的属性中
            bindRequestParameters(binder, request);
            validateIfApplicable(binder, parameter);
            if (binder.getBindingResult().hasErrors()) {
                if (isBindExceptionRequired(binder, parameter)) {
                    throw new BindException(binder.getBindingResult());
                }
            }

        }
```

* WebDataBinder：数据绑定器负责数据绑定工作；数据绑定期间产生的类型转换、格式化、数据校验等问题；![img](../../../../Software/Typora/Picture/Image-1594959294378.png)
* ConversionService组件：负责数据类型的转换以及格式化功能；    ConversionService中有非常多的**converter；**不同类型的转换和格式化用它自己的**converter**
* validators组件：数据校验器，负责数据校验
* 若经过validators数据校验出错误后，由bindingResult组件解决错误

#### Ajax&Json格式化

```
@DateTimeFormat(pattern="yyyy-MM-dd")  // 日期格式
@Past // 必须是一个过去的时间   @Future则是未来的时间
@JsonFormat(pattern="yyyy-MM-dd") // JSON格式
private Date birth = new Date();

@JsonIgnore  // 忽略这个json字段
@NumberFormat(pattern="#,###.##") // 数字格式化
@RequestBody  // 标注在参数前，将请求的JSON串封装为Bean对象
@ResponseBody // 标注在方法上，将返回的Bean对象封装为JSON串，放在响应体中
return new ResponseEntity<String>(body,headers,HttpStatus.OK); // 搭配@ResponseBody一起使用，全部一起响应

@HttpEntity<String> str  // 标注的参数str为获取的所有请求头
@RequestHeader("") // 只能获取指定的请求头
```

* Ajax

```
$("a:first").click(function(){
	$ajax({
		url: "${ctp/test}",
		type: "POST",
		data: Param, // 请求参数
		contentType:"application/json", // 参数Param以JSON格式传入
		success:function(data){ // data为请求后台的返回值
			console(data); 
		}
	})
})
```



#### 拦截器（相当于Filter）

* SpringMVC提供了拦截器机制；允许运行目标方法之前进行一些拦截工作，或者目标方法运行之后进行一些其他处理；

* preHandle：在目标方法运行之前调用；返回boolean；return true；(chain.doFilter())放行； return false；不放行

* postHandle：在目标方法运行之后调用：目标方法调用之后

* afterCompletion：在请求整个完成之后；来到目标页面之后；

* chain.doFilter()放行；资源响应之后；

* 正常运行流程； 拦截器类实现HandlerInterceptor········拦截器的preHandle------目标方法-----拦截器postHandle-----页面-------拦截器的afterCompletion;

* 基于XML配置：

  * ```
    	// 对DispatcherServlet的请求进行处理，如果该请求已经作了映射，那么会接着交给后台对应的处理程序，如果没有作映射，就交给 WEB 应用服务器默认的 Servlet 处理，从而找到对应的静态资源
      	<mvc:default-servlet-handler/>  
      	// 提供Controller请求转发，json自动转换等功能
      	<mvc:annotation-driven></mvc:annotation-driven>
      	<!-- 测试拦截器 -->
      	<mvc:interceptors>
      		<!--用法1：配置某个拦截器；默认是拦截所有请求的；  -->
      		<bean class="com.atguigu.controller.MyFirstInterceptor"></bean>
      		
      		<!-- 用法2：配置某个拦截器更详细的信息 -->
      		<mvc:interceptor>
      			<!-- 只来拦截test01请求 -->
      			<mvc:mapping path="/test01"/>
      			<bean class="com.atguigu.controller.MySecondInterceptor"></bean>
      		</mvc:interceptor>
      	</mvc:interceptors>
    ```

* 1、只要preHandle不放行就没有以后的流程；2、只要preHandle放行了，afterCompletion都会执行；

* 异常流程：preHandle不放行该拦截器其他都不执行； Interceptor2不放行；但是他前面的Interceptor1已经放行了的拦截器的afterCompletion总会执行；

#### 异常处理

* 异常解析器：
  * 如果异常解析器都不能处理就直接抛出去。默认就是这个几个HandlerExceptionResolver异常解析器：
  * ExceptionHandlerExceptionResolver：处理标有 @ExceptionHandler(value={能处理的异常类})
    * 在方法参数位置加上Exception e便可获得异常信息。可以放在ModelAndView里。
    * 在类上标注@ControllerAdvice：表示这个类专门用来处理异常
  * ResponseStatusExceptionResolver：
    * 处理类上TestException标有 @ResponseStatus(reason="设置要展示的错误信息", value=HttpStatus.xxx) 的注解. 抛出异常信息和状态码
    * 其他地方直接throw new TestException
  * DefaultHandlerExceptionResolver：判断是否SpringMVC自带的异常。是则直接抛出异常

####  整合Spring

* 1）方法1：合并Spring配置文件：`<import resource="Spring.xml" />`。容器是合并的

* 2）方法2：

  * ioc容器分开，Spring管理业务逻辑组件（service）；

  ```
      <context:component-scan base-package="com.atguigu">  // 除了controller，其他都扫描进ioc容器
          <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
          <context:exclude-filter type="annotation" expression="org.springframework.web.bind.annotation.ControllerAdvice"/>
      </context:component-scan>
  ```

  * SpringMVC管理控制器组件（controller）；只扫描controller类和异常控制组件ControllerAdvice

  ```
      <context:component-scan base-package="com.atguigu" use-default-filters="false"> // 禁用默认过滤规则，只扫描下面配置的2个类 
          <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>// controller类
          // ControllerAdvice：异常控制
          <context:include-filter type="annotation" expression="org.springframework.web.bind.annotation.ControllerAdvice"/>
      </context:component-scan>
  ```

* 当有2个容器时，默认Spring作为父容器，SpringMVC是一个子容器；子容器还可以引用父容器的组件；父容器不能引用子容器的组件；





## SpringMVC源码

#### DispatcherServlet分析

* 继承关系：![image-20200713113116312](../../../../Software/Typora/Picture/image-20200713113116312.png)

* 其中，doGet等方法都在FrameworkServlet中，且每个do方法都调用了processRequest方法

![image-20200713114507070](../../../../Software/Typora/Picture/image-20200713114507070.png)

* FrameworkServlet 的 ProcessRequest方法

```
protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {

		long startTime = System.currentTimeMillis();
		Throwable failureCause = null;
		// 跟国际化有关
		LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
		LocaleContext localeContext = buildLocaleContext(request);

		RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
		ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
		asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());

		initContextHolders(request, localeContext, requestAttributes);

		try {
			doService(request, response);  // 断点1：核心方法，进入该方法
		}
		catch (ServletException ex) {
			failureCause = ex;
			throw ex;
		}
		catch (IOException ex) {
			failureCause = ex;
			throw ex;
		}
		catch (Throwable ex) {
			failureCause = ex;
			throw new NestedServletException("Request processing failed", ex);
		}

		finally {
			resetContextHolders(request, previousLocaleContext, previousAttributes);
			if (requestAttributes != null) {
				requestAttributes.requestCompleted();
			}

			if (logger.isDebugEnabled()) {
				if (failureCause != null) {
					this.logger.debug("Could not complete request", failureCause);
				}
				else {
					if (asyncManager.isConcurrentHandlingStarted()) {
						logger.debug("Leaving response open for concurrent processing");
					}
					else {
						this.logger.debug("Successfully completed request");
					}
				}
			}

			publishRequestHandledEvent(request, startTime, failureCause);
		}
	}
```

* 断点1：doService(request, response); 是一个抽象方法，接下来找到子类DispatcherServlet的doService方法

```
protected abstract void doService(HttpServletRequest request, HttpServletResponse response)
			throws Exception;
```

* 断点2：DispatcherServlet的doService方法

```
@Override
	protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
		if (logger.isDebugEnabled()) {
			String requestUri = urlPathHelper.getRequestUri(request);
			String resumed = WebAsyncUtils.getAsyncManager(request).hasConcurrentResult() ? " resumed" : "";
			logger.debug("DispatcherServlet with name '" + getServletName() + "'" + resumed +
					" processing " + request.getMethod() + " request for [" + requestUri + "]");
		}

		// Keep a snapshot of the request attributes in case of an include,
		// to be able to restore the original attributes after the include.
		Map<String, Object> attributesSnapshot = null;
		if (WebUtils.isIncludeRequest(request)) {
			logger.debug("Taking snapshot of request attributes before include");
			attributesSnapshot = new HashMap<String, Object>();
			Enumeration<?> attrNames = request.getAttributeNames();  // 获取请求域中的数据并保存到attrNames
			while (attrNames.hasMoreElements()) {
				String attrName = (String) attrNames.nextElement();
				if (this.cleanupAfterInclude || attrName.startsWith("org.springframework.web.servlet")) {
					attributesSnapshot.put(attrName, request.getAttribute(attrName)); // 请求域中的数据并保存到attributesSnapshot
				}
			}
		}

		// Make framework objects available to handlers and view objects.
		request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
		request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
		request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
		request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

		FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
		if (inputFlashMap != null) {
			request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
		}
		request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
		request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);

		try {
			doDispatch(request, response);  // 断点3：进入该方法
		}
		finally {
			if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
				return;
			}
			// Restore the original attribute snapshot, in case of an include.
			if (attributesSnapshot != null) {
				restoreAttributesAfterInclude(request, attributesSnapshot);
			}
		}
	}
```

* **进入断点3：DispatcherServlet的doDispatch方法（关键）**

```
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				processedRequest = checkMultipart(request); // 检查当前请求是否是文件上传下载请求
				multipartRequestParsed = processedRequest != request;

				// Determine handler for the current request. 决定当前请求用哪个handler
				 // (1)根据当前请求地址找到哪个类(控制器controller)来处理
				mappedHandler = getHandler(processedRequest);
				if (mappedHandler == null || mappedHandler.getHandler() == null) {
					noHandlerFound(processedRequest, response); // 如果找不到哪个处理器（控制器controller）能处理这个请求就请求404或抛异常
					return;
				}

				// Determine handler adapter for the current request. 根据当前请求决定处理器的适配器
				 // (2)获取处理器适配器（反射工具）：AnnotationMethodHandlerAdaper
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
				
				// Process last-modified header, if supported by the handler.
				String method = request.getMethod(); // 获取当前请求方式
				boolean isGet = "GET".equals(method);
				if (isGet || "HEAD".equals(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (logger.isDebugEnabled()) {
						String requestUri = urlPathHelper.getRequestUri(request);
						logger.debug("Last-Modified value for [" + requestUri + "] is: " + lastModified);
					}
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}
				
				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				try {
					// Actually invoke the handler.  真正执行处理器
(3)断点4：通过处理器的适配器ha执行目标方法，执行完后的返回值作为视图名，保存到ModelAndView中。无论目标方法怎么写，执行后的信息都会封装为ModelAndView
					mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
				}
				finally {
					if (asyncManager.isConcurrentHandlingStarted()) {
						return;
					}
				}
	
				applyDefaultViewName(request, mv); // 如果没有视图名，则将返回值设置为默认视图名  
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
// (4)断点5：页面渲染，根据方法最终执行完成后将封装的ModelAndView转发到目标页面，并把ModelAndView可以从请求域中获取
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
		catch (Exception ex) {
			triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
		}
		catch (Error err) {
			triggerAfterCompletionWithError(processedRequest, response, mappedHandler, err);
		}
		finally {
			if (asyncManager.isConcurrentHandlingStarted()) {
				// Instead of postHandle and afterCompletion
				mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				return;
			}
			// Clean up any resources used by a multipart request.
			if (multipartRequestParsed) {
				cleanupMultipart(processedRequest);
			}
		}
	}
```

#### doDispatch()底层细节

#### (1) DispatcherServlet的getHandler方法：根据当前请求找到处理该请求的处理器

```
// processedRequest：Tomcat的连接对象，保存着url请求地址
mappedHandler = getHandler(processedRequest);  // 断点：返回目标处理器类的执行链：mappedHandler = HandlerExcutionChain
```

* 断点： 进入DispatcherServlet的getHandler方法

```
	protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
	// handlerMappings处理器映射：保存了每个处理器(controller类)能处理哪些请求的映射信息
	// 包含了基于xml配置的处理BeanNameUrlHandlerMapping 和 基于注解RequestMapping的处理DefaultAnnotationHandlerMapping
	// 其中DefaultAnnotationHandlerMapping中的handlerMap保存了这些映射信息，每个处理器能处理哪些方法
		for (HandlerMapping hm : this.handlerMappings) {
			if (logger.isTraceEnabled()) {
				logger.trace(
						"Testing handler map [" + hm + "] in DispatcherServlet with name '" + getServletName() + "'");
			}
			HandlerExecutionChain handler = hm.getHandler(request);
			if (handler != null) {
				return handler;
			}
		}
		return null;
	}
```



#### (2) DispatcherServlet的getHandlerAdapter方法，根据处理器类获取对应的适配器

```
// 获得处理器适配器，拿到适配器才能去执行目标方法
HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
```

```
	protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
		for (HandlerAdapter ha : this.handlerAdapters) { // 遍历适配器
			if (logger.isTraceEnabled()) {
				logger.trace("Testing handler adapter [" + ha + "]");
			}
			if (ha.supports(handler)) { // 断点1：判断当前适配器是否支持处理器
				return ha; // 得到该适配器
			}
		}
		throw new ServletException("No adapter for handler [" + handler +
				"]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
	}
```

*  handlerAdapters 有3个，第三个是基于注解的适配器（AnnotationMethodHandlerAdapter）

![image-20200716105107206](../../../../Software/Typora/Picture/image-20200716105107206.png)

* 断点1：进入AnnotationMethodHandlerAdapter的supports方法，解析注解方法的适配器，只要标注了注解的方法就能用

```
	@Override
	public boolean supports(Object handler) {
		return getMethodResolver(handler).hasHandlerMethods(); // 返回该处理器(controller)的方法
	}
```

#### (3) DispatcherServlet的ha.handle方法，使用适配器执行目标方法

* 进入AnnotationMethodHandlerAdapter的handle方法

```
	@Override
	public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {

		Class<?> clazz = ClassUtils.getUserClass(handler); // 处理器类型
		Boolean annotatedWithSessionAttributes = this.sessionAnnotatedClassesCache.get(clazz);
		if (annotatedWithSessionAttributes == null) {
			annotatedWithSessionAttributes = (AnnotationUtils.findAnnotation(clazz, SessionAttributes.class) != null);
			this.sessionAnnotatedClassesCache.put(clazz, annotatedWithSessionAttributes);
		}
		
		if (annotatedWithSessionAttributes) {
			// Always prevent caching in case of session attribute management.
			checkAndPrepare(request, response, this.cacheSecondsForSessionAttributeHandlers, true);
			// Prepare cached set of session attributes names.
		}
		else {
			// Uses configured default cacheSeconds setting.
			checkAndPrepare(request, response, true);
		}

		// Execute invokeHandlerMethod in synchronized block if required.
		if (this.synchronizeOnSession) {
			HttpSession session = request.getSession(false);
			if (session != null) {
				Object mutex = WebUtils.getSessionMutex(session);
				synchronized (mutex) {
					return invokeHandlerMethod(request, response, handler);
				}
			}
		}
		// 断点1：执行处理器（controller类）中的方法
		return invokeHandlerMethod(request, response, handler);
	}
```

* 断点1：进入AnnotationMethodHandlerAdapter的invokeHandlerMethod方法

```
	protected ModelAndView invokeHandlerMethod(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		// 获取这个处理器中的方法获得方法解析器
		ServletHandlerMethodResolver methodResolver = getMethodResolver(handler);
		// 通过方法解析器解析当前请求，找到真正的目标方法
		Method handlerMethod = methodResolver.resolveHandlerMethod(request);
		// 通过方法解析器获得该方法执行器
		ServletHandlerMethodInvoker methodInvoker = new ServletHandlerMethodInvoker(methodResolver);
		ServletWebRequest webRequest = new ServletWebRequest(request, response);
		// （重点）创建隐含模型BindingAwareModelMap，包含Model、Map、ModelAndView等
		ExtendedModelMap implicitModel = new BindingAwareModelMap();
		// 断点2：真正执行目标方法。方法执行器来执行方法处理器，返回结果
		Object result = methodInvoker.invokeHandlerMethod(handlerMethod, handler, webRequest, implicitModel);
		ModelAndView mav =
				methodInvoker.getModelAndView(handlerMethod, handler.getClass(), result, implicitModel, webRequest);
		methodInvoker.updateModelAttributes(handler, (mav != null ? mav.getModel() : null), implicitModel, webRequest);
		return mav;
	}
```

* 断点2：进入HandlerMethodInvoker的invokeHandlerMethod，方法执行的细节

```
	public final Object invokeHandlerMethod(Method handlerMethod, Object handler,
			NativeWebRequest webRequest, ExtendedModelMap implicitModel) throws Exception {

		Method handlerMethodToInvoke = BridgeMethodResolver.findBridgedMethod(handlerMethod);
		try {
			boolean debug = logger.isDebugEnabled();
			for (String attrName : this.methodResolver.getActualSessionAttributeNames()) {
				// 从session域中查询数据
				Object attrValue = this.sessionAttributeStore.retrieveAttribute(webRequest, attrName);
				if (attrValue != null) {
					implicitModel.addAttribute(attrName, attrValue); // 将session中的数据放到隐含模型中
				}
			}
			// 获取所有标注了 ModelAttribute 注解的方法
			for (Method attributeMethod : this.methodResolver.getModelAttributeMethods()) {
				Method attributeMethodToInvoke = BridgeMethodResolver.findBridgedMethod(attributeMethod);
				// 断点3：先确定ModelAttribute方法执行时要使用的每一个参数的值
				Object[] args = resolveHandlerArguments(attributeMethodToInvoke, handler, webRequest, implicitModel);
				if (debug) {
					logger.debug("Invoking model attribute method: " + attributeMethodToInvoke);
				}
				// 获取类上面 @ModelAttribute 注解中的value属性值
				String attrName = AnnotationUtils.findAnnotation(attributeMethod, ModelAttribute.class).value();
				if (!"".equals(attrName) && implicitModel.containsAttribute(attrName)) {
					continue;
				}
				// 变为可访问的方法执行器
				ReflectionUtils.makeAccessible(attributeMethodToInvoke);
				// ModelAttribute标注的方法总是在其他方法之前执行，执行后会获得ModelAttribute数据，attrValue为该目标方法的返回值
				Object attrValue = attributeMethodToInvoke.invoke(handler, args);
				if ("".equals(attrName)) { // attrName为注解中的value值	
					// 解析目标方法的返回值类型
					Class<?> resolvedType = GenericTypeResolver.resolveReturnType(attributeMethodToInvoke, handler.getClass());
					// 为返回值起变量名，若@ModelAttribute(“”)中没有设置attrName，则以方法返回值首字母小写作为变量名    
					attrName = Conventions.getVariableNameForReturnType(attributeMethodToInvoke, resolvedType, attrValue);
				}
				// 把提前运行的ModelAttribute方法的返回值也放入隐含模型中
				if (!implicitModel.containsAttribute(attrName)) {
				// 给隐含模型放入：方法运行后的返回值按照方法上的@ModelAttribute("abc")指定的key，如果没有key则以返回值类型首字母小写作为变量
					implicitModel.addAttribute(attrName, attrValue);
				}
			}
			// 断点4：再次解析目标方法的参数是哪些值，进入resolveHandlerArguments方法
			Object[] args = resolveHandlerArguments(handlerMethodToInvoke, handler, webRequest, implicitModel);
			if (debug) {
				logger.debug("Invoking request handler method: " + handlerMethodToInvoke);
			}
			ReflectionUtils.makeAccessible(handlerMethodToInvoke);
			// 断点5：执行目标方法。所有ModelAttribute标注的方法执行后，才执行其他方法
			return handlerMethodToInvoke.invoke(handler, args);
		}
		catch (IllegalStateException ex) {
			// Internal assertion failed (e.g. invalid signature):
			// throw exception with full handler method context...
			throw new HandlerMethodInvocationException(handlerMethodToInvoke, ex);
		}
		catch (InvocationTargetException ex) {
			// User-defined @ModelAttribute/@InitBinder/@RequestMapping method threw an exception...
			ReflectionUtils.rethrowException(ex.getTargetException());
			return null;
		}
	}

```

*  断点3：进入resolveHandlerArguments方法

```
	private Object[] resolveHandlerArguments(Method handlerMethod, Object handler,
			NativeWebRequest webRequest, ExtendedModelMap implicitModel) throws Exception {

		Class<?>[] paramTypes = handlerMethod.getParameterTypes();  // 参数类型
		Object[] args = new Object[paramTypes.length]; // 所有参数放在一个数组

		for (int i = 0; i < args.length; i++) {
			MethodParameter methodParam = new MethodParameter(handlerMethod, i);
			methodParam.initParameterNameDiscovery(this.parameterNameDiscoverer);
			GenericTypeResolver.resolveParameterType(methodParam, handler.getClass());
			String paramName = null;
			String headerName = null;
			boolean requestBodyFound = false;
			String cookieName = null;
			String pathVarName = null;
			String attrName = null;
			boolean required = false;
			String defaultValue = null;
			boolean validate = false;
			Object[] validationHints = null;
			int annotationsFound = 0;
			// 得到该参数的所有注解
			Annotation[] paramAnns = methodParam.getParameterAnnotations();
			// 解析各种参数注解
			for (Annotation paramAnn : paramAnns) {
				if (RequestParam.class.isInstance(paramAnn)) {
					RequestParam requestParam = (RequestParam) paramAnn;
					paramName = requestParam.value();
					required = requestParam.required();
					defaultValue = parseDefaultValueAttribute(requestParam.defaultValue());
					annotationsFound++;
				}
				else if (RequestHeader.class.isInstance(paramAnn)) {
					RequestHeader requestHeader = (RequestHeader) paramAnn;
					headerName = requestHeader.value();
					required = requestHeader.required();
					defaultValue = parseDefaultValueAttribute(requestHeader.defaultValue());
					annotationsFound++;
				}
				else if (RequestBody.class.isInstance(paramAnn)) {
					requestBodyFound = true;
					annotationsFound++;
				}
				else if (CookieValue.class.isInstance(paramAnn)) {
					CookieValue cookieValue = (CookieValue) paramAnn;
					cookieName = cookieValue.value();
					required = cookieValue.required();
					defaultValue = parseDefaultValueAttribute(cookieValue.defaultValue());
					annotationsFound++;
				}
				else if (PathVariable.class.isInstance(paramAnn)) {
					PathVariable pathVar = (PathVariable) paramAnn;
					pathVarName = pathVar.value();
					annotationsFound++;
				}
				// 参数有注解ModelAttribute自定义类型参数，就拿到注解的值让attrName保存
				else if (ModelAttribute.class.isInstance(paramAnn)) {
					ModelAttribute attr = (ModelAttribute) paramAnn;
					attrName = attr.value(); // 拿到注解的值让attrName保存
					annotationsFound++;
				}
				else if (Value.class.isInstance(paramAnn)) {
					defaultValue = ((Value) paramAnn).value();
				}
				else if (paramAnn.annotationType().getSimpleName().startsWith("Valid")) {
					validate = true;
					Object value = AnnotationUtils.getValue(paramAnn);
					validationHints = (value instanceof Object[] ? (Object[]) value : new Object[] {value});
				}
			}
			// 每个参数只能有一个注解，否则报错
			if (annotationsFound > 1) {
				throw new IllegalStateException("Handler parameter annotations are exclusive choices - " +
						"do not specify more than one such annotation on the same parameter: " + handlerMethod);
			}
			// 没有找到注解的情况下，先看看是否是普通参数（是否原生API）
			// 再看看是否是Model或Map，如果是就传入隐含模型
			// 自定义类型的参数没有ModelAttribute注解：
			//	先看是否原生API、再看是否Model或Map、再看是否是其他类型
			// 再看是否 是简单类型属性，即基本数据类型属性
			// 如果是自定义类型对象，会有两种情况：
				1）如果参数标注了ModelAttribute注解就给attrName赋值为这个注解的value值
				2）如果参数没有标注ModelAttribute注解就给attrName赋值为“”
			if (annotationsFound == 0) {
				// 解析没有注解的普通参数和标准参数
				Object argValue = resolveCommonArgument(methodParam, webRequest);
				if (argValue != WebArgumentResolver.UNRESOLVED) {
					args[i] = argValue; // 解析参数成功，放进数组args中
				}
				else if (defaultValue != null) {
					args[i] = resolveDefaultValue(defaultValue);
				}
				else {
					Class<?> paramType = methodParam.getParameterType();
					// 该参数是否是Model或Map下的类型，是就将之前创建的隐含模型直接赋值给这个参数
					if (Model.class.isAssignableFrom(paramType) || Map.class.isAssignableFrom(paramType)) {
						if (!paramType.isAssignableFrom(implicitModel.getClass())) {
							throw new IllegalStateException("Argument [" + paramType.getSimpleName() + "] is of type " +
									"Model or Map but is not assignable from the actual model. You may need to switch " +
									"newer MVC infrastructure classes to use this argument.");
						}
						args[i] = implicitModel;
					}
					else if (SessionStatus.class.isAssignableFrom(paramType)) {
						args[i] = this.sessionStatus;
					}
					else if (HttpEntity.class.isAssignableFrom(paramType)) {
						args[i] = resolveHttpEntityRequest(methodParam, webRequest);
					}
					else if (Errors.class.isAssignableFrom(paramType)) {
						throw new IllegalStateException("Errors/BindingResult argument declared " +
								"without preceding model attribute. Check your handler method signature!");
					}
					else if (BeanUtils.isSimpleProperty(paramType)) {
						paramName = "";
					}
					else {
						attrName = "";
					}
				}
			}
			
			// 确定参数值的环节
			if (paramName != null) {
				args[i] = resolveRequestParam(paramName, required, defaultValue, methodParam, webRequest, handler);
			}
			else if (headerName != null) {
				args[i] = resolveRequestHeader(headerName, required, defaultValue, methodParam, webRequest, handler);
			}
			else if (requestBodyFound) {
				args[i] = resolveRequestBody(methodParam, webRequest, handler);
			}
			else if (cookieName != null) {
				args[i] = resolveCookieValue(cookieName, required, defaultValue, methodParam, webRequest, handler);
			}
			else if (pathVarName != null) {
				args[i] = resolvePathVariable(pathVarName, methodParam, webRequest, handler);
			}
			// 确定自定义类型参数的值；还要将请求中的每个参数赋值给这个对象
			else if (attrName != null) {
			// 断点4：绑定器，resolveModelAttribute：确定自定义类型参数的值
				WebDataBinder binder =
						resolveModelAttribute(attrName, methodParam, implicitModel, webRequest, handler);
				boolean assignBindingResult = (args.length > i + 1 && Errors.class.isAssignableFrom(paramTypes[i + 1]));
				if (binder.getTarget() != null) {
				// 把刚拿到的对象和请求里的数据进行一一绑定
					doBind(binder, webRequest, validate, validationHints, !assignBindingResult);
				}
				args[i] = binder.getTarget();
				if (assignBindingResult) {
					args[i + 1] = binder.getBindingResult();
					i++;
				}
				implicitModel.putAll(binder.getBindingResult().getModel());
			}
		}

		return args; // 返回参数，回到断点3
	}
```

*  断点4：进入HandlerMethodInvoker的resolveModelAttribute方法：确定自定义类型参数的值

```
	private WebDataBinder resolveModelAttribute(String attrName, MethodParameter methodParam,
			ExtendedModelMap implicitModel, NativeWebRequest webRequest, Object handler) throws Exception {

		// Bind request parameter onto object...
		String name = attrName;
		if ("".equals(name)) {
			// 如果attrName是“”，即没有标注@ModelAttribute，就将参数类型的首字母小写作为值
			name = Conventions.getVariableNameForParameter(methodParam);
		}
		Class<?> paramType = methodParam.getParameterType();
		Object bindObject;
		// 确定目标对象的值POJO的三步：
		 // 1）隐含模型有这个key(即自定义类型参数名)指定的值
		if (implicitModel.containsKey(name)) {
			bindObject = implicitModel.get(name);
		}
		// 2）session域中拿属性，拿不到就抛异常
		else if (this.methodResolver.isSessionAttribute(name, paramType)) {
			bindObject = this.sessionAttributeStore.retrieveAttribute(webRequest, name);
			if (bindObject == null) {
				raiseSessionRequiredException("Session attribute '" + name + "' required - not found in session");
			}
		}
		// 3）如果都不是就利用反射创建对象
		else {
			bindObject = BeanUtils.instantiateClass(paramType);
		}
		WebDataBinder binder = createBinder(webRequest, bindObject, name);
		initBinder(handler, name, binder, webRequest);
		return binder;
	}
```

![image-20200716210654918](../../../../Software/Typora/Picture/image-20200716210654918.png)

#### (4) DispatcherServlet的processDispatchResult方法，视图解析，页面渲染

* 进入DispatcherServlet的processDispatchResult方法

```
	private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
			HandlerExecutionChain mappedHandler, ModelAndView mv, Exception exception) throws Exception {

		boolean errorView = false;

		if (exception != null) {
			if (exception instanceof ModelAndViewDefiningException) {
				logger.debug("ModelAndViewDefiningException encountered", exception);
				mv = ((ModelAndViewDefiningException) exception).getModelAndView();
			}
			else {
				Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
				mv = processHandlerException(request, response, handler, exception);
				errorView = (mv != null);
			}
		}

		// Did the handler return a view to render?
		if (mv != null && !mv.wasCleared()) {
		// 断点1：渲染页面
			render(mv, request, response);
			if (errorView) {
				WebUtils.clearErrorRequestAttributes(request);
			}
		}
		else {
			if (logger.isDebugEnabled()) {
				logger.debug("Null ModelAndView returned to DispatcherServlet with name '" + getServletName() +
						"': assuming HandlerAdapter completed request handling");
			}
		}

		if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
			// Concurrent handling started during a forward
			return;
		}

		if (mappedHandler != null) {
			mappedHandler.triggerAfterCompletion(request, response, null);
		}
	}
```

* 断点1：渲染页面，进入render方法

```
protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
		// Determine locale for request and apply it to the response.
		Locale locale = this.localeResolver.resolveLocale(request);
		response.setLocale(locale);
		// ViewResolver的作用是根据视图名（方法返回值）得到View对象
		View view;
		if (mv.isReference()) {
			// We need to resolve the view name.
			// 断点2：通过ViewResolver得到View对象
			view = resolveViewName(mv.getViewName(), mv.getModelInternal(), locale, request);
			if (view == null) {
				throw new ServletException(
						"Could not resolve view with name '" + mv.getViewName() + "' in servlet with name '" +
								getServletName() + "'");
			}
		}
		else {
			// No need to lookup: the ModelAndView object contains the actual View object.
			view = mv.getView();
			if (view == null) {
				throw new ServletException("ModelAndView [" + mv + "] neither contains a view name nor a " +
						"View object in servlet with name '" + getServletName() + "'");
			}
		}

		// Delegate to the View object for rendering.
		if (logger.isDebugEnabled()) {
			logger.debug("Rendering view [" + view + "] in DispatcherServlet with name '" + getServletName() + "'");
		}
		try {
			// 断点5：执行页面渲染
			view.render(mv.getModelInternal(), request, response);
		}
		catch (Exception ex) {
			if (logger.isDebugEnabled()) {
				logger.debug("Error rendering view [" + view + "] in DispatcherServlet with name '"
						+ getServletName() + "'", ex);
			}
			throw ex;
		}
	}
```

* 断点2：进入resolveViewName方法，根据视图名获得View对象

```
	protected View resolveViewName(String viewName, Map<String, Object> model, Locale locale,
			HttpServletRequest request) throws Exception {
		// 遍历所有的viewResolver，所有配置的视图解析器都来尝试根据视图名（返回值）得到View对象。得到就返回，得不到就换一个视图解析器
		for (ViewResolver viewResolver : this.viewResolvers) {
			// 断点3：视图解析器viewResolver根据方法返回值获得view对象
			View view = viewResolver.resolveViewName(viewName, locale);
			if (view != null) {
				return view;
			}
		}
		// 返回断点1
		return null;
	}
```

* 断点3：进入AbstractCachingViewResolver的resolveViewName方法

```
	@Override
	public View resolveViewName(String viewName, Locale locale) throws Exception {
		if (!isCache()) {
			return createView(viewName, locale);
		}
		else {
			Object cacheKey = getCacheKey(viewName, locale);
			View view = this.viewAccessCache.get(cacheKey);
			if (view == null) {
				synchronized (this.viewCreationCache) {
					view = this.viewCreationCache.get(cacheKey);
					if (view == null) {
						// Ask the subclass to create the View object.
						// 断点4：根据方法返回值创建View视图对象
						view = createView(viewName, locale);
						if (view == null && this.cacheUnresolved) {
							view = UNRESOLVED_VIEW;
						}
						if (view != null) {
							this.viewAccessCache.put(cacheKey, view);
							this.viewCreationCache.put(cacheKey, view);
							if (logger.isTraceEnabled()) {
								logger.trace("Cached view [" + cacheKey + "]");
							}
						}
					}
				}
			}
			// 返回断点2
			return (view != UNRESOLVED_VIEW ? view : null);
		}
	}
```

* 断点4：进入UrlBasedViewResolver的createView方法

```
	@Override
	protected View createView(String viewName, Locale locale) throws Exception {
		// If this resolver is not supposed to handle the given view,
		// return null to pass on to the next resolver in the chain.
		if (!canHandle(viewName, locale)) {
			return null;
		}
		// Check for special "redirect:" prefix.
		// new一个Redirect视图
		if (viewName.startsWith(REDIRECT_URL_PREFIX)) {
			String redirectUrl = viewName.substring(REDIRECT_URL_PREFIX.length());
			RedirectView view = new RedirectView(redirectUrl, isRedirectContextRelative(), isRedirectHttp10Compatible());
			return applyLifecycleMethods(viewName, view);
		}
		// Check for special "forward:" prefix
		// new一个Forward视图
		if (viewName.startsWith(FORWARD_URL_PREFIX)) {
			String forwardUrl = viewName.substring(FORWARD_URL_PREFIX.length());
			// 如果是以forward为前缀，就创建InternalResourceView
			return new InternalResourceView(forwardUrl);
		}
		// Else fall back to superclass implementation: calling loadView.
		// 没有以上两个前缀，就使用父类创建一个View。返回到断点3
		return super.createView(viewName, locale);
	}
```

*  断点5：进入AbstractView的render方法，执行页面渲染

```
	@Override
	public void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
		if (logger.isTraceEnabled()) {
			logger.trace("Rendering view with name '" + this.beanName + "' with model " + model +
				" and static attributes " + this.staticAttributes);
		}

		Map<String, Object> mergedModel = createMergedOutputModel(model, request, response);
		// 准备响应
		prepareResponse(request, response);
		// 断点6：渲染要给页面输出的所有数据
		renderMergedOutputModel(mergedModel, request, response);
	}
```

* 断点6：进入InternalResourceView的renderMergedOutputModel方法

```
	@Override
	protected void renderMergedOutputModel(
			Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws Exception {

		// Determine which request handle to expose to the RequestDispatcher.
		HttpServletRequest requestToExpose = getRequestToExpose(request);

		// Expose the model object as request attributes.
		// 断点7：将模型中的数据暴露在请求域中
		exposeModelAsRequestAttributes(model, requestToExpose);

		// Expose helpers as request attributes, if any.
		exposeHelpers(requestToExpose);

		// Determine the path for the request dispatcher.
		// 获得要转发的路径
		String dispatcherPath = prepareForRendering(requestToExpose, response);

		// Obtain a RequestDispatcher for the target resource (typically a JSP).
		// 转发器
		RequestDispatcher rd = getRequestDispatcher(requestToExpose, dispatcherPath);
		if (rd == null) {
			throw new ServletException("Could not get RequestDispatcher for [" + getUrl() +
					"]: Check that the corresponding file exists within your web application archive!");
		}

		// If already included or response already committed, perform include, else forward.
		if (useInclude(requestToExpose, response)) {
			response.setContentType(getContentType());
			if (logger.isDebugEnabled()) {
				logger.debug("Including resource [" + getUrl() + "] in InternalResourceView '" + getBeanName() + "'");
			}
			rd.include(requestToExpose, response);
		}

		else {
			// Note: The forwarded resource is supposed to determine the content type itself.
			if (logger.isDebugEnabled()) {
				logger.debug("Forwarding to resource [" + getUrl() + "] in InternalResourceView '" + getBeanName() + "'");
			}
			// 转发到页面，结束页面渲染
			rd.forward(requestToExpose, response);
		}
	}
```

* 断点7：进入到AbstractView的exposeModelAsRequestAttributes方法。将模型中的数据暴露在请求域中

```
protected void exposeModelAsRequestAttributes(Map<String, Object> model, HttpServletRequest request) throws Exception {
		for (Map.Entry<String, Object> entry : model.entrySet()) {
			String modelName = entry.getKey();
			Object modelValue = entry.getValue();
			if (modelValue != null) {
				// 将key和value放在map中
				request.setAttribute(modelName, modelValue);
				if (logger.isDebugEnabled()) {
					logger.debug("Added model object '" + modelName + "' of type [" + modelValue.getClass().getName() +
							"] to request in view with name '" + getBeanName() + "'");
				}
			}
			else {
				request.removeAttribute(modelName);
				if (logger.isDebugEnabled()) {
					logger.debug("Removed model object '" + modelName +
							"' from request in view with name '" + getBeanName() + "'");
				}
			}
		}
	}
```

* 视图解析器只是为了得到视图对象。视图对象才能真正转发（将模型数据全部放在请求域中）或者重定向到页面

* 视图对象由视图解析器负责实例化。由于视图是无状态的（两次实例化无关联无影响）。所以不会由线程安全问题



#### SpringMVC九大组件

* 九大组件都是接口，接口是规范，提供扩展性

```
	/** MultipartResolver used by this servlet */
	private MultipartResolver multipartResolver;	// 文件上传下载解析器

	/** LocaleResolver used by this servlet */
	private LocaleResolver localeResolver;	// 国际化和区域信息解析器

	/** ThemeResolver used by this servlet */
	private ThemeResolver themeResolver;	// 主题解析器，主题效果更换

	/** List of HandlerMappings used by this servlet */
	private List<HandlerMapping> handlerMappings;	// Handler映射信息
	
	/** List of HandlerAdapters used by this servlet */
	private List<HandlerAdapter> handlerAdapters; // Handler的适配器

	/** List of HandlerExceptionResolvers used by this servlet */
	private List<HandlerExceptionResolver> handlerExceptionResolvers;  // 异常解析器

	/** RequestToViewNameTranslator used by this servlet */
	private RequestToViewNameTranslator viewNameTranslator;    // 转换器：若方法没有返回值，则将请求地址作为视图名
 
	/** FlashMapManager used by this servlet */
	private FlashMapManager flashMapManager;  // FlashMap + Manager：SpringMVC运行重定向携带数据（在session中一取就消失）的功能

	/** List of ViewResolvers used by this servlet */
	private List<ViewResolver> viewResolvers;  // 视图解析器
```

* 初始化细节

```
	@Override
	protected void onRefresh(ApplicationContext context) {
		initStrategies(context); // 初始化九大组件策略
	}
	
	protected void initStrategies(ApplicationContext context) {
		initMultipartResolver(context);
		initLocaleResolver(context);
		initThemeResolver(context);
		initHandlerMappings(context);
		initHandlerAdapters(context);
		initHandlerExceptionResolvers(context);
		initRequestToViewNameTranslator(context);
		initViewResolvers(context);
		initFlashMapManager(context);
	}
```

* 以HandlerMappings为例

```
	private void initHandlerMappings(ApplicationContext context) {
		this.handlerMappings = null;
		// detectAllHandlerMappings探查所有HandlerMappings，默认为true，可以在web.xml修改init-param标签属性
		if (this.detectAllHandlerMappings) {
			// Find all HandlerMappings in the ApplicationContext, including ancestor contexts.
			// 找出ioc容器中所有的HandlerMapping
			Map<String, HandlerMapping> matchingBeans =
					BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
			if (!matchingBeans.isEmpty()) {
				this.handlerMappings = new ArrayList<HandlerMapping>(matchingBeans.values());
				// We keep HandlerMappings in sorted order.
				OrderComparator.sort(this.handlerMappings);
			}
		}
		else {
			try {
				HandlerMapping hm = context.getBean(HANDLER_MAPPING_BEAN_NAME, HandlerMapping.class);
				this.handlerMappings = Collections.singletonList(hm);
			}
			catch (NoSuchBeanDefinitionException ex) {
				// Ignore, we'll add a default HandlerMapping later.
			}
		}

		// Ensure we have at least one HandlerMapping, by registering
		// a default HandlerMapping if no other mappings are found.
		if (this.handlerMappings == null) {
			// 如果找不到这个组件，就用默认的配置，可以在DispatcherServlet.properties中修改
			// 九大组件只有multipartResolver找不到就为null，而不是默认配置
			this.handlerMappings = getDefaultStrategies(context, HandlerMapping.class);
			if (logger.isDebugEnabled()) {
				logger.debug("No HandlerMappings found in servlet '" + getServletName() + "': using default");
			}
		}
	}

```



#### 拦截器底层源码

* 顺序

* DispatcherServlet 的 doDispatch 方法

```
	protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
            ModelAndView mv = null;
            Exception dispatchException = null;

            try {
                processedRequest = checkMultipart(request);
                multipartRequestParsed = processedRequest != request;

                // Determine handler for the current request.拿到方法的执行链，包含拦截器
                mappedHandler = getHandler(processedRequest);
                if (mappedHandler == null || mappedHandler.getHandler() == null) {
                    noHandlerFound(processedRequest, response);
                    return;
                }

                // Determine handler adapter for the current request.
                HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

                // Process last-modified header, if supported by the handler.
                String method = request.getMethod();
                boolean isGet = "GET".equals(method);
                if (isGet || "HEAD".equals(method)) {
                    long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                    if (logger.isDebugEnabled()) {
                        String requestUri = urlPathHelper.getRequestUri(request);
                        logger.debug("Last-Modified value for [" + requestUri + "] is: " + lastModified);
                    }
                    if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
                        return;
                    }
                }

               // （1）拦截器preHandle，执行位置;有一个拦截器返回false目标方法以后都不会执行；直接跳到afterCompletion
                if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                    return;
                }

                try {
                    //（2）适配器执行目标方法.Actually invoke the handler.
                    mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
                }
                finally {
                    if (asyncManager.isConcurrentHandlingStarted()) {
                        return;
                    }
                }

                applyDefaultViewName(request, mv);
                 // （3）目标方法只要正常就会走到postHandle;如果有异常直接跳到afterCompletion；
                mappedHandler.applyPostHandle(processedRequest, response, mv);
            }
            catch (Exception ex) {
                dispatchException = ex;
            }
			
            //（4）页面渲染；如果完蛋也是直接跳到afterCompletion；
            processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
        }
        catch (Exception ex) {
        	// （5）afterCompletion
            triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
        }
        catch (Error err) {
            triggerAfterCompletionWithError(processedRequest, response, mappedHandler, err);
        }
        finally {
            if (asyncManager.isConcurrentHandlingStarted()) {
                // Instead of postHandle and afterCompletion
                mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
                return;
            }
            // Clean up any resources used by a multipart request.
            if (multipartRequestParsed) {
                cleanupMultipart(processedRequest);
            }
        }
    }
```

* PreHandle

```
boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
        if (getInterceptors() != null) {
            for (int i = 0; i < getInterceptors().length; i++) {
                HandlerInterceptor interceptor = getInterceptors()[i];

                //preHandle-true-false，不放行返回true，就会进入该if
                if (!interceptor.preHandle(request, response, this.handler)) {
                    //执行完afterCompletion（）;
                    triggerAfterCompletion(request, response, null);
                    //返回一个false
                    return false;
                }
               //记录一下索引
               //this.interceptorIndex = i;
            }
        }
        return true;
    }
```

* PostHandle

```
void applyPostHandle(HttpServletRequest request, HttpServletResponse response, ModelAndView mv) throws Exception {
        if (getInterceptors() == null) {
            return;
        }
        //逆向执行每个拦截器的postHandle
        for (int i = getInterceptors().length - 1; i >= 0; i--) {
            HandlerInterceptor interceptor = getInterceptors()[i];
            interceptor.postHandle(request, response, this.handler, mv);
        }
    }
```

* 页面渲染

```
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
            HandlerExecutionChain mappedHandler, ModelAndView mv, Exception exception) throws Exception {

        boolean errorView = false;

        if (exception != null) {
            if (exception instanceof ModelAndViewDefiningException) {
                logger.debug("ModelAndViewDefiningException encountered", exception);
                mv = ((ModelAndViewDefiningException) exception).getModelAndView();
            }
            else {
                Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
                mv = processHandlerException(request, response, handler, exception);
                errorView = (mv != null);
            }
        }

        // Did the handler return a view to render?
        if (mv != null && !mv.wasCleared()) {
             页面渲染
            render(mv, request, response);
            if (errorView) {
                WebUtils.clearErrorRequestAttributes(request);
            }
        }
        else {
            if (logger.isDebugEnabled()) {
                logger.debug("Null ModelAndView returned to DispatcherServlet with name '" + getServletName() +
                        "': assuming HandlerAdapter completed request handling");
            }
        }

        if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
            // Concurrent handling started during a forward
            return;
        }

        if (mappedHandler != null) {
               //页面正常执行afterCompletion；即使没走到这，afterCompletion总会执行；
            mappedHandler.triggerAfterCompletion(request, response, null);
        }
    }
```

* AfterCompletion，只要preHandler放行，必定执行AfterCompletion

```
void triggerAfterCompletion(HttpServletRequest request, HttpServletResponse response, Exception ex)
            throws Exception {

        if (getInterceptors() == null) {
            return;
        }
         
          //倒序遍历，有记录最后一个放行拦截器的索引，从他开始把之前所有放行的拦截器的afterCompletion都执行
        for (int i = this.interceptorIndex; i >= 0; i--) {
            HandlerInterceptor interceptor = getInterceptors()[i];
            try {
                interceptor.afterCompletion(request, response, this.handler, ex);
            }
            catch (Throwable ex2) {
                logger.error("HandlerInterceptor.afterCompletion threw exception", ex2);
            }
        }
    }
```

* 多拦截器执行顺序：

![image-20200717174846566](../../../../Software/Typora/Picture/image-20200717174846566.png)



#### 异常处理底层源码

* 来到页面渲染

```
	private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
            HandlerExecutionChain mappedHandler, ModelAndView mv, Exception exception) throws Exception {

        boolean errorView = false;

        if (exception != null) {//如果有异常
            if (exception instanceof ModelAndViewDefiningException) {
                logger.debug("ModelAndViewDefiningException encountered", exception);
                mv = ((ModelAndViewDefiningException) exception).getModelAndView();
            }
            else {//处理异常
                Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
                mv = processHandlerException(request, response, handler, exception);
                errorView = (mv != null);
            }
        }

        // Did the handler return a view to render?
        if (mv != null && !mv.wasCleared()) {
               //来到页面
            render(mv, request, response);
            if (errorView) {
                WebUtils.clearErrorRequestAttributes(request);
            }
        }
        else {
            if (logger.isDebugEnabled()) {
                logger.debug("Null ModelAndView returned to DispatcherServlet with name '" + getServletName() +
                        "': assuming HandlerAdapter completed request handling");
            }
        }

        if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
            // Concurrent handling started during a forward
            return;
        }

        if (mappedHandler != null) {
            mappedHandler.triggerAfterCompletion(request, response, null);
        }
    }
```

* 异常解析器尝试解析，解析完成进行后续，解析失败下一个解析器

```
	protected ModelAndView processHandlerException(HttpServletRequest request, HttpServletResponse response,
            Object handler, Exception ex) throws Exception {

        // Check registered HandlerExceptionResolvers...
        ModelAndView exMv = null;
        // 三个解析器轮流解析异常，都不能处理则抛出异常
        for (HandlerExceptionResolver handlerExceptionResolver : this.handlerExceptionResolvers) {
            exMv = handlerExceptionResolver.resolveException(request, response, handler, ex);
            if (exMv != null) {
                break;
            }
        }
        if (exMv != null) {
            if (exMv.isEmpty()) {          
                return null;
            }
            // We might still need view name translation for a plain error model...
            if (!exMv.hasView()) {
                exMv.setViewName(getDefaultViewName(request));
            }
            if (logger.isDebugEnabled()) {
                logger.debug("Handler execution resulted in exception - forwarding to resolved error view: " + exMv, ex);
            }
            WebUtils.exposeErrorRequestAttributes(request, ex, getServletName());
            return exMv;
        }

        throw ex;
    }
```













