[TOC]





## 1.Gateway的拦截器

我们要在项目中实现一个拦截器，需要继承两个类：GlobalFilter, Ordered

GlobalFilter：全局过滤拦截器，在gateway中已经有部分实现，具体参照：https://www.cnblogs.com/liukaifeng/p/10055862.html

Ordered：拦截器的顺序，不多说

于是一个简单的拦截器就有了

```
@Slf4j
@Component
public class AuthFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return -10;
    }
}
```

Gateway的核心接口：GatewayFilter，GlobalFilter，GatewayFilterChain。具体介绍：https://www.cnblogs.com/bjlhx/p/9786478.html

Gateway的路由转发规则介绍：https://segmentfault.com/a/1190000019101829

## 2.简介

我们在使用Spring Cloud Gateway的时候，注意到过滤器（包括GatewayFilter、GlobalFilter和过滤器链GatewayFilterChain），都依赖到ServerWebExchange。

这里的设计和Servlet中的Filter是相似的，当前过滤器可以决定是否执行下一个过滤器的逻辑，由GatewayFilterChain#filter()是否被调用来决定。而ServerWebExchange就相当于当前请求和响应的上下文。

ServerWebExchange实例不单存储了Request和Response对象，还提供了一些扩展方法，如果想实现改造请求参数或者响应参数，就必须深入了解ServerWebExchange。

## 3.ServerWebExchange

ServerWebExchange的注释： ServerWebExchange是一个HTTP请求-响应交互的契约。提供对HTTP请求和响应的访问，并公开额外的服务器端处理相关属性和特性，如请求属性。

其实，ServerWebExchange命名为服务网络交换器，存放着重要的请求-响应属性、请求实例和响应实例等等，有点像Context的角色。

### 3.1.所有接口

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public interface ServerWebExchange {

    // 日志前缀属性的KEY，值为org.springframework.web.server.ServerWebExchange.LOG_ID
    // 可以理解为 attributes.set("org.springframework.web.server.ServerWebExchange.LOG_ID","日志前缀的具体值");
    // 作用是打印日志的时候会拼接这个KEY对饮的前缀值，默认值为""
    String LOG_ID_ATTRIBUTE = ServerWebExchange.class.getName() + ".LOG_ID";
    String getLogPrefix();

    // 获取ServerHttpRequest对象
    ServerHttpRequest getRequest();

    // 获取ServerHttpResponse对象
    ServerHttpResponse getResponse();
    
    // 返回当前exchange的请求属性，返回结果是一个可变的Map
    Map<String, Object> getAttributes();
    
    // 根据KEY获取请求属性
    @Nullable
    default <T> T getAttribute(String name) {
        return (T) getAttributes().get(name);
    }
    
    // 根据KEY获取请求属性，做了非空判断
    @SuppressWarnings("unchecked")
    default <T> T getRequiredAttribute(String name) {
        T value = getAttribute(name);
        Assert.notNull(value, () -> "Required attribute '" + name + "' is missing");
        return value;
    }

     // 根据KEY获取请求属性，需要提供默认值
    @SuppressWarnings("unchecked")
    default <T> T getAttributeOrDefault(String name, T defaultValue) {
        return (T) getAttributes().getOrDefault(name, defaultValue);
    } 

    // 返回当前请求的网络会话
    Mono<WebSession> getSession();

    // 返回当前请求的认证用户，如果存在的话
    <T extends Principal> Mono<T> getPrincipal();  
    
    // 返回请求的表单数据或者一个空的Map，只有Content-Type为application/x-www-form-urlencoded的时候这个方法才会返回一个非空的Map -- 这个一般是表单数据提交用到
    Mono<MultiValueMap<String, String>> getFormData();   
    
    // 返回multipart请求的part数据或者一个空的Map，只有Content-Type为multipart/form-data的时候这个方法才会返回一个非空的Map  -- 这个一般是文件上传用到
    Mono<MultiValueMap<String, Part>> getMultipartData();
    
    // 返回Spring的上下文
    @Nullable
    ApplicationContext getApplicationContext();   

    // 这几个方法和lastModified属性相关
    boolean isNotModified();
    boolean checkNotModified(Instant lastModified);
    boolean checkNotModified(String etag);
    boolean checkNotModified(@Nullable String etag, Instant lastModified);
    
    // URL转换
    String transformUrl(String url);    
   
    // URL转换映射
    void addUrlTransformer(Function<String, String> transformer); 

    // 注意这个方法，方法名是：改变，这个是修改ServerWebExchange属性的方法，返回的是一个Builder实例，Builder是ServerWebExchange的内部类
    default Builder mutate() {
         return new DefaultServerWebExchangeBuilder(this);
    }

    interface Builder {      
         
        // 覆盖ServerHttpRequest
        Builder request(Consumer<ServerHttpRequest.Builder> requestBuilderConsumer);
        Builder request(ServerHttpRequest request);
        
        // 覆盖ServerHttpResponse
        Builder response(ServerHttpResponse response);
        
        // 覆盖当前请求的认证用户
        Builder principal(Mono<Principal> principalMono);
    
        // 构建新的ServerWebExchange实例
        ServerWebExchange build();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

注意到ServerWebExchange#mutate()方法，ServerWebExchange实例可以理解为不可变实例。

如果我们想要修改它，需要通过mutate()方法生成一个新的实例，后面会修改请求以及响应时会用到，暂时不做介绍

## 4.ServerHttpRequest

ServerHttpRequest实例是用于承载请求相关的属性和请求体，Spring Cloud Gateway中底层使用Netty处理网络请求。

通过追溯源码，可以从ReactorHttpHandlerAdapter中得知ServerWebExchange实例中持有的ServerHttpRequest实例的具体实现是ReactorServerHttpRequest。

之所以列出这些实例之间的关系，是因为这样比较容易理清一些隐含的问题，例如：ReactorServerHttpRequest的父类AbstractServerHttpRequest中初始化内部属性headers的时候把请求的HTTP头部封装为只读的实例：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
// HttpHeaders类中的readOnlyHttpHeaders方法，其中ReadOnlyHttpHeaders屏蔽了所有修改请求头的方法，直接抛出UnsupportedOperationException
public static HttpHeaders readOnlyHttpHeaders(HttpHeaders headers) {
    Assert.notNull(headers, "HttpHeaders must not be null");
    if (headers instanceof ReadOnlyHttpHeaders) {
        return headers;
    }
    else {
        return new ReadOnlyHttpHeaders(headers);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### 4.1.所有接口

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
ublic interface HttpMessage {
    
    // 获取请求头，目前的实现中返回的是ReadOnlyHttpHeaders实例，只读
    HttpHeaders getHeaders();
}    

public interface ReactiveHttpInputMessage extends HttpMessage {
    
    // 返回请求体的Flux封装
    Flux<DataBuffer> getBody();
}

public interface HttpRequest extends HttpMessage {

    // 返回HTTP请求方法，解析为HttpMethod实例
    @Nullable
    default HttpMethod getMethod() {
        return HttpMethod.resolve(getMethodValue());
    }
    
    // 返回HTTP请求方法，字符串
    String getMethodValue();    
    
    // 请求的URI
    URI getURI();
}    

public interface ServerHttpRequest extends HttpRequest, ReactiveHttpInputMessage {
    
    // 连接的唯一标识或者用于日志处理标识
    String getId();   
    
    // 获取请求路径，封装为RequestPath对象
    RequestPath getPath();
    
    // 返回查询参数，是只读的MultiValueMap实例
    MultiValueMap<String, String> getQueryParams();

    // 返回Cookie集合，是只读的MultiValueMap实例
    MultiValueMap<String, HttpCookie> getCookies();  
    
    // 远程服务器地址信息
    @Nullable
    default InetSocketAddress getRemoteAddress() {
       return null;
    }

    // SSL会话实现的相关信息
    @Nullable
    default SslInfo getSslInfo() {
       return null;
    }  
    
    // 修改请求的方法，返回一个建造器实例Builder，Builder是内部类
    default ServerHttpRequest.Builder mutate() {
        return new DefaultServerHttpRequestBuilder(this);
    } 

    interface Builder {

        // 覆盖请求方法
        Builder method(HttpMethod httpMethod);
         
        // 覆盖请求的URI、请求路径或者上下文，这三者相互有制约关系，具体可以参考API注释
        Builder uri(URI uri);
        Builder path(String path);
        Builder contextPath(String contextPath);

        // 覆盖请求头
        Builder header(String key, String value);
        Builder headers(Consumer<HttpHeaders> headersConsumer);
        
        // 覆盖SslInfo
        Builder sslInfo(SslInfo sslInfo);
        
        // 构建一个新的ServerHttpRequest实例
        ServerHttpRequest build();
    }         
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

## 5.ServerHttpResponse

ServerHttpResponse实例是用于承载响应相关的属性和响应体，Spring Cloud Gateway中底层使用Netty处理网络请求。

通过追溯源码，可以从ReactorHttpHandlerAdapter中得知ServerWebExchange实例中持有的ServerHttpResponse实例的具体实现是ReactorServerHttpResponse。

之所以列出这些实例之间的关系，是因为这样比较容易理清一些隐含的问题，例如：ReactorServerHttpResponse构造函数初始化实例的时候，存放响应Header的是HttpHeaders实例，也就是响应Header是可以直接修改的。

```
public ReactorServerHttpResponse(HttpServerResponse response, DataBufferFactory bufferFactory) {
    super(bufferFactory, new HttpHeaders(new NettyHeadersAdapter(response.responseHeaders())));
    Assert.notNull(response, "HttpServerResponse must not be null");
    this.response = response;
}
```

### 5.1.所有接口

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public interface HttpMessage {
    
    // 获取响应Header，目前的实现中返回的是HttpHeaders实例，可以直接修改
    HttpHeaders getHeaders();
}  

public interface ReactiveHttpOutputMessage extends HttpMessage {
    
    // 获取DataBufferFactory实例，用于包装或者生成数据缓冲区DataBuffer实例(创建响应体)
    DataBufferFactory bufferFactory();

    // 注册一个动作，在HttpOutputMessage提交之前此动作会进行回调
    void beforeCommit(Supplier<? extends Mono<Void>> action);

    // 判断HttpOutputMessage是否已经提交
    boolean isCommitted();
    
    // 写入消息体到HTTP协议层
    Mono<Void> writeWith(Publisher<? extends DataBuffer> body);

    // 写入消息体到HTTP协议层并且刷新缓冲区
    Mono<Void> writeAndFlushWith(Publisher<? extends Publisher<? extends DataBuffer>> body);
    
    // 指明消息处理已经结束，一般在消息处理结束自动调用此方法，多次调用不会产生副作用
    Mono<Void> setComplete();
}

public interface ServerHttpResponse extends ReactiveHttpOutputMessage {
    
    // 设置响应状态码
    boolean setStatusCode(@Nullable HttpStatus status);
    
    // 获取响应状态码
    @Nullable
    HttpStatus getStatusCode();
    
    // 获取响应Cookie，封装为MultiValueMap实例，可以修改
    MultiValueMap<String, ResponseCookie> getCookies();  
    
    // 添加响应Cookie
    void addCookie(ResponseCookie cookie);  
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

## 6.ServerWebExchangeUtils和上下文属性

ServerWebExchangeUtils里面存放了很多静态公有的字符串KEY值(这些字符串KEY的实际值是org.springframework.cloud.gateway.support.ServerWebExchangeUtils. + 下面任意的静态公有KEY)

这些字符串KEY值一般是用于ServerWebExchange的属性(Attribute，见上文的ServerWebExchange#getAttributes()方法)的KEY

这些属性值都是有特殊的含义，在使用过滤器的时候如果时机适当可以直接取出来使用，下面逐个分析。

> 1.PRESERVE_HOST_HEADER_ATTRIBUTE：
>
> 　　是否保存Host属性，值是布尔值类型，写入位置是PreserveHostHeaderGatewayFilterFactory，使用的位置是NettyRoutingFilter，作用是如果设置为true，HTTP请求头中的Host属性会写到底层Reactor-Netty的请求Header属性中。
>
> 2.CLIENT_RESPONSE_ATTR：
>
> 　　保存底层Reactor-Netty的响应对象，类型是reactor.netty.http.client.HttpClientResponse
>
> 3.CLIENT_RESPONSE_CONN_ATTR：
>
> 　　保存底层Reactor-Netty的连接对象，类型是reactor.netty.Connection。
>
> 4.URI_TEMPLATE_VARIABLES_ATTRIBUTE：
>
> 　　PathRoutePredicateFactory解析路径参数完成之后，把解析完成后的占位符KEY-路径Path映射存放在ServerWebExchange的属性中，KEY就是URI_TEMPLATE_VARIABLES_ATTRIBUTE。
>
> 5.CLIENT_RESPONSE_HEADER_NAMES：
>
> 　　保存底层Reactor-Netty的响应Header的名称集合。
>
> 6.GATEWAY_ROUTE_ATTR：
>
> 　　用于存放RoutePredicateHandlerMapping中匹配出来的具体的路由(org.springframework.cloud.gateway.route.Route)实例，通过这个路由实例可以得知当前请求会路由到下游哪个服务。
>
> 7.GATEWAY_REQUEST_URL_ATTR：
>
> 　　java.net.URI类型的实例，这个实例代表直接请求或者负载均衡处理之后需要请求到下游服务的真实URI。
>
> 8.GATEWAY_ORIGINAL_REQUEST_URL_ATTR：
>
> 　　java.net.URI类型的实例，需要重写请求URI的时候，保存原始的请求URI。
>
> 9.GATEWAY_HANDLER_MAPPER_ATTR：
>
> 　　保存当前使用的HandlerMapping具体实例的类型简称(一般是字符串”RoutePredicateHandlerMapping”)。
>
> 10.GATEWAY_SCHEME_PREFIX_ATTR：
>
> 　　确定目标路由URI中如果存在schemeSpecificPart属性，则保存该URI的scheme在此属性中，路由URI会被重新构造，见RouteToRequestUrlFilter。
>
> 11.GATEWAY_PREDICATE_ROUTE_ATTR：
>
> 　　用于存放RoutePredicateHandlerMapping中匹配出来的具体的路由(org.springframework.cloud.gateway.route.Route)实例的ID。
>
> 12.WEIGHT_ATTR：
>
> 　　实验性功能(此版本还不建议在正式版本使用)存放分组权重相关属性，见WeightCalculatorWebFilter。
>
> 13.ORIGINAL_RESPONSE_CONTENT_TYPE_ATTR：
>
> 　　存放响应Header中的ContentType的值。
>
> 14.HYSTRIX_EXECUTION_EXCEPTION_ATTR：
>
> 　　Throwable的实例，存放的是Hystrix执行异常时候的异常实例，见HystrixGatewayFilterFactory。
>
> 15.GATEWAY_ALREADY_ROUTED_ATTR：
>
> 　　布尔值，用于判断是否已经进行了路由，见NettyRoutingFilter。
>
> 16.GATEWAY_ALREADY_PREFIXED_ATTR：
>
> 　　布尔值，用于判断请求路径是否被添加了前置部分，见PrefixPathGatewayFilterFactory。

ServerWebExchangeUtils提供的上下文属性用于Spring Cloud Gateway的ServerWebExchange组件处理请求和响应的时候，内部一些重要实例或者标识属性的安全传输和使用

使用它们可能存在一定的风险，因为没有人可以确定在版本升级之后，原有的属性KEY或者VALUE是否会发生改变，如果评估过风险或者规避了风险之后，可以安心使用。

例如我们在做请求和响应日志(类似Nginx的Access Log)的时候，可以依赖到GATEWAY_ROUTE_ATTR，因为我们要打印路由的目标信息。

## 7.修改数据

### 7.1.修改请求路径

可以将原来的请求路径进行修改，也可以修改其他属性，具体可以看： interface Builder下的方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Slf4j
@Component
public class ModifyRequestFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();//修改请求路径
        String newServletPath = "/test"
        ServerHttpRequest newRequest = request.mutate().path(newServletPath).build();return chain.filter(exchange.mutate().request(decorator).build());
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### 7.2.修改请求数据

ServerHttpRequest的getBody方法获取的是Flux<DataBuffer>，遍历这个DataBuffer就可以取出数据

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Slf4j
@Component
public class ModifyRequestBodyGlobalFilter implements GlobalFilter {

    private final DataBufferFactory dataBufferFactory = new NettyDataBufferFactory(ByteBufAllocator.DEFAULT);

    @Autowired
    private ObjectMapper objectMapper;

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        // 新建一个ServerHttpRequest装饰器,覆盖需要装饰的方法
        ServerHttpRequestDecorator decorator = new ServerHttpRequestDecorator(request) {

            @Override
            public Flux<DataBuffer> getBody() {
                Flux<DataBuffer> body = super.getBody();
                InputStreamHolder holder = new InputStreamHolder();
                body.subscribe(buffer -> holder.inputStream = buffer.asInputStream());
                if (null != holder.inputStream) {
                    try {
                        // 解析JSON的节点
                        JsonNode jsonNode = objectMapper.readTree(holder.inputStream);
                        Assert.isTrue(jsonNode instanceof ObjectNode, "JSON格式异常");
                        ObjectNode objectNode = (ObjectNode) jsonNode;
                        // JSON节点最外层写入新的属性
                        objectNode.put("userId", accessToken);
                        DataBuffer dataBuffer = dataBufferFactory.allocateBuffer();
                        String json = objectNode.toString();
                        log.info("最终的JSON数据为:{}", json);
                        dataBuffer.write(json.getBytes(StandardCharsets.UTF_8));
                        return Flux.just(dataBuffer);
                    } catch (Exception e) {
                        throw new IllegalStateException(e);
                    }
                } else {
                    return super.getBody();
                }
            }
        };
        // 使用修改后的ServerHttpRequestDecorator重新生成一个新的ServerWebExchange
        return chain.filter(exchange.mutate().request(decorator).build());
    }

    private class InputStreamHolder {
        InputStream inputStream;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### 7.3.修改路由

路由信息等存储在ServerWebExchange的属性中的，修改后重新存进去接口覆盖

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Slf4j
@Component
public class ModifyRequestFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

        String serviceName = exchange.getAttribute(GatewayConstant.SERVICE_NAME);

        //修改路由
        Route route = exchange.getAttribute(ServerWebExchangeUtils.GATEWAY_ROUTE_ATTR);
        Route newRoute = Route.async()
                .asyncPredicate(route.getPredicate())
                .filters(route.getFilters())
                .id(route.getId())
                .order(route.getOrder())
                .uri(GatewayConstant.URI.LOAD_BALANCE+serviceName).build();

        exchange.getAttributes().put(ServerWebExchangeUtils.GATEWAY_ROUTE_ATTR,newRoute);return chain.filter(exchange);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### 7.4.修改响应体

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Slf4j
@Component
public class ModifyResponseFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String requestId = exchange.getAttribute(GatewayConstant.REQUEST_TRACE_ID);
        ServerHttpResponse originalResponse = exchange.getResponse();
        DataBufferFactory bufferFactory = originalResponse.bufferFactory();
        ServerHttpResponseDecorator decoratedResponse = new ServerHttpResponseDecorator(originalResponse) {
            @Override
            public Mono<Void> writeWith(Publisher<? extends DataBuffer> body) {
                if (body instanceof Flux) {
                    Flux<? extends DataBuffer> fluxBody = (Flux<? extends DataBuffer>) body;
                    return super.writeWith(fluxBody.map(dataBuffer -> {
                        byte[] content = new byte[dataBuffer.readableByteCount()];
                        dataBuffer.read(content);
                        //释放掉内存
                        DataBufferUtils.release(dataBuffer); 　　　　　　　　　　　　　　//原始响应
                        String originalbody = new String(content, Charset.forName("UTF-8"));
                        //修改后的响应体
                        finalBody = JSON.toJSONString("{test:'test'}");

                        return bufferFactory.wrap(finalBody.getBytes());
                    }));
                }

                return super.writeWith(body);
            }
        };
        return chain.filter(exchange.mutate().response(decoratedResponse).build());
    }}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

##  8.ServerWebExchange对比Servlet

这里总结部分我在写代码中遇到的一些不同与相应代替办法

### 8.1.request.setAttribute

在HttpServletRequest中：request.setAttribute(“key”, "value");

而在ServerHttpRequest中并无Attribute相关操作，可以存数据的HttpHeader也是read-only的

替代：ServerWebExchange --> exchange.getAttributes().put(“key”, "value");

###  8.2.request.getHeader()

在HttpServletRequest中：request.getHeader("test");

替代：ServerHttpRequest -->request.getHeaders().getFirst(“test”)

getFirst的原因是获取的到Headers里面的List数组，遍历Header如下：

```
HttpHeaders headers = request.getHeaders();
for (Map.Entry<String,List<String>> header:headers.entrySet()) {
　　String key = header.getKey();　　List<String> values = header.getValue()；
}
```

 

内容参考：http://throwable.coding.me/2019/05/18/spring-cloud-gateway-modify-request-response