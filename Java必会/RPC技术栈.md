[TOC]

摘自：https://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247487507&idx=1&sn=7511f822bf95b25a2586dfdb0c06546f&chksm=eb539525dc241c33507a02d137bd48b9d9e2a33c8b76030f6cc372d0cfa478c8d83d230a9e96&scene=21#wechat_redirect

# RPC框架原理

RPC（Remote Procedure Call）–远程过程调用，通过网络通信调用不同的服务，共同支撑一个软件系统，微服务实现的基石技术。使用RPC可以解耦系统，方便维护，同时增加系统处理请求的能力。

目前开源RPC框架主要有：Dubbo（阿里）、Spring Cloud、Motan（微博）、Tars（腾讯）、gRPC（Google）、Thrift（Facebook）

![img](../../../../Software/Typora/Picture/640)

如上图所示，我将一个RPC调用流程概括为上图中5个流程，左边3个为客户端流程，右边两个为服务端流程。 

##### 即客户端调用 =》 动态代理 =》 网络请求 =》 服务端接收 =》 调用实现方法

![img](../../../../Software/Typora/Picture/20388163-fbed6070e6169548.jpg)

RPC框架要做到的最基本的三件事：

1、服务端如何确定客户端要调用的函数；

在远程调用中，客户端和服务端分别维护一个【ID->函数】的对应表， ID在所有进程中都是唯一确定的。客户端在做远程过程调用时，附上这个ID，服务端通过查表，来确定客户端需要调用的函数，然后执行相应函数的代码。

2、如何进行序列化和反序列化；

客户端和服务端交互时将参数或结果转化为字节流在网络中传输，那么数据转化为字节流的或者将字节流转换成能读取的固定格式时就需要进行序列化和反序列化，序列化和反序列化的速度也会影响远程调用的效率。

3、如何进行网络传输（选择何种网络协议）；

多数RPC框架选择TCP作为传输协议，也有部分选择HTTP。如gRPC使用HTTP2。不同的协议各有利弊。**TCP更加高效，而HTTP在实际应用中更加的灵活。**



### 客户端调用

服务调用方在调用服务时，一般进行相关初始化，通过配置文件/配置中心 获取服务端地址用户调用。

```java
// 用户服务接口
public interface UserService {    
　public User genericUser(Integer id,String name,Long phone);
}

//调用方
//服务初始化
KRPC.init("D:\\krpc\\service\\demo\\conf\\client.xml");
UserService service = ProxyFactory.create(UserService.class, "demo","demoService");
User user = service.genericUser(1, "yasin", 1888888888L);
```

## 动态代理

动态代理这东西意如其名，它代理你帮你做事情，动态代理看这篇文章《[详解 Java 中的三种代理模式](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247486759&idx=2&sn=6769d8ff9d163babe726b6213c6d15e4&chksm=eb538811dc240107bcf2a6e65b5381b2a68175af8ff12f4e2c1b0a06f7d16850db4acb64a18e&scene=21#wechat_redirect)》。 

上面我们不说道直接调用一个接口中的方法，并且没有用该接口的实现类调用，那么方法是怎么生效的呢？

可以看到这个用户服务这个service是由ProxyFactory代理工程创造的，在该ProxyFactory#create()方法中就跟一个代理处理器绑定在一起了。

```java
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

    //构造请求request
    Request request = new Request();
    ....

    return RequestHandler.request(serviceName, request,returnClass);
}
```

这个类实现了InvocationHandler接口（JDK提供的动态代理技术），每次去调用接口方法，最终都交由该handler进行处理。 

这个环节一般会获取方法的一些信息，例如方法名，方法参数类型，方法参数值，返回对象类型。

同时这个环节会提供序列化功能，一般的RPC网络传输使用TCP（哪怕使用HTTP）传输，这里也要将这些参数进行封装成我们定义的数据接口进行传输。

### 网络请求

使用tcp传输，利用socket通信进行通信

### 服务端接收

这一块使用netty，可以快速构建一个高性能、高可靠的一个服务端。

```java
public class ServerHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf buf = (ByteBuf)msg;  
        byte[] bytes = new byte[buf.readableBytes()];  
        buf.readBytes(bytes);  

        byte[] responseBytes = RequestHandler.handler(bytes);
        ByteBuf resbuf = ctx.alloc().buffer(responseBytes.length);
        resbuf.writeBytes(responseBytes);
        ctx.writeAndFlush(resbuf);

    }    

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

### 调用实现方法

服务端获取客户端请求的数据后， 调用请求中的方法，方法参数值，通过反射调用真实的方法，获取其返回值，将其序列化封装，通过netty进行数据返回，客户端再接受数据并解析，这就完成了一次rpc请求调用的全过程。

```
ethod = clazz.getMethod(request.getMethodName(),requestParamTypes);
method.setAccessible(true);
result = method.invoke(service, requestParmsValues)
```

### 服务端的动态加载

通过2.2到2.6的说明，一次RPC请求过程大致如此，但是一个RPC框架会有很多细节需要处理。

其实在一次请求调用前，服务端肯定要先启动。

服务端作为一个容器，跟我们熟知的tomcat一样，它可以动态的加载任何项目。所以在服务端启动的时候，必须要进行一个动态加载的过程。

在KRPC中，我使用了URLClassLoader动态加载一个指定路径的jar包，任何业务服务的实现所依赖的jar包都可以放入该路径中。



# 手写RPC框架

RPC远程调用过程：

* 建立远程通信（Socket）TCP/UDP
* 数据传递
* 序列化和反序列化

## server

* 代理

  ```java
  public class RpcServerProxy {
  
      // new一个缓存线程池，特性几乎不受核心线程数量控制，核心线程为Integer.MAX_VALUE
      ExecutorService executorService = Executors.newCachedThreadPool();
  
      public void publisher(Object service, int port) {
          ServerSocket serverSocket = null;  // 服务端socket
          try {
              serverSocket = new ServerSocket(port); // socket绑定端口
              while(true) {
                  Socket socket = serverSocket.accept();  // 接收一个请求 (bio)
                  executorService.execute(new ProcessorHandler(socket, service)); // 执行反射
              }
          }catch (Exception e) {
              e.printStackTrace();
          } finally {
              if(serverSocket != null) {
                  try {
                      serverSocket.close(); // 关闭服务端socket
                  } catch (IOException e) {
                      e.printStackTrace();
                  }
              }
          }
  
      }
  
  }
  ```

* 执行目标方法

  ```java
  public class ProcessorHandler implements Runnable {
  
      Socket socket;
      Object service;
  
      public ProcessorHandler(Socket socket, Object service) {
          this.socket = socket;
          this.service = service;  // 要调用的目标类
      }
  
      public ProcessorHandler() {
  
      }
  
      // 线程执行,将调用后的信息返回给client
      @Override
      public void run() {
          System.out.println("开始处理客户端请求");
          ObjectInputStream inputStream = null;
          ObjectOutputStream outputStream = null;
          try {
              inputStream = new ObjectInputStream(socket.getInputStream());
              // 通过输入流，从请求的client获取到请求相关的数据
              RPCRequest rpcRequest = (RPCRequest) inputStream.readObject();  // 反序列化
              Object result = invoke(rpcRequest);   // 反射
              // 将调用到的方法执行结果返回给客户端
              outputStream = new ObjectOutputStream(socket.getOutputStream());
              outputStream.writeObject(result);  // 开始写数据流到client
              outputStream.flush();
  
          } catch (IOException e) {
              e.printStackTrace();
          } catch (ClassNotFoundException e) {
              e.printStackTrace();
          } finally {
              try {
                  outputStream.close();
                  inputStream.close();
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
      }
  
      // 通过执行本服务端的方法并返回数据
      private Object invoke(RPCRequest request) {
           Object[] args = request.getParameters(); // 获取到请求参数集合
           Class<?>[] types = new Class[args.length]; // 参数类型
          for (int i = 0; i < args.length; i++) {
              types[i] = args[i].getClass();  // 通过参数获得对应的参数类型
          }
          try {
              // 通过反射获取类的方法
              Method method = service.getClass().getMethod(request.getMethodName(), types);
              Object result = method.invoke(service, args); // 执行目标方法HelloServiceImpl，并获取到返回值
              return result;
          } catch (NoSuchMethodException e) {
              e.printStackTrace();
          } catch (IllegalAccessException e) {
              e.printStackTrace();
          } catch (InvocationTargetException e) {
              e.printStackTrace();
          }
          return null;
      }
  
  }
  ```

* 监听远程调用请求

  ```java
  public class App {
      public static void main(String[] args) {
          HelloService helloService = new HelloServiceImpl();
          RpcServerProxy rpcServerProxy = new RpcServerProxy();
          rpcServerProxy.publisher(helloService, 8080);
      }
  }
  ```

  

## client

* 代理

  ```java
  public class RpcClientProxy {
      public <T> T clientProxy(Class<T> interfaceCls, String host, Integer port) {
          return (T) Proxy.newProxyInstance(interfaceCls.getClassLoader(),
                  new Class<?>[] {interfaceCls}, new RemoteInvocationHandler(host, port));
      }
  }
  ```

* InvocationHandler

  ```java
  public class RemoteInvocationHandler implements InvocationHandler {
  
      String host;
      Integer port;
  
      public String getHost() {
          return host;
      }
  
      public void setHost(String host) {
          this.host = host;
      }
  
      public Integer getPort() {
          return port;
      }
  
      public void setPort(Integer port) {
          this.port = port;
      }
  
      public RemoteInvocationHandler(String host, Integer port) {
          this.host = host;
          this.port = port;
      }
  
      @Override
      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
          RPCRequest request = new RPCRequest();
          request.setMethodName(method.getName());
          request.setParameters(args);
          RPCNetTransport rpcNetTransport = new RPCNetTransport(host, port);
          return rpcNetTransport.sendRequest(request);  // 远程socket连接服务端并发送请求调用对应方法
      }
  }
  ```

* 通信，建立连接，发送请求

  ```java
  public class RPCNetTransport {
      String host;
      Integer port;
  
      public RPCNetTransport(String host, Integer port) {
          this.host = host;
          this.port = port;
      }
  
      private Socket newSocket() {
          System.out.println("创建一个新的socket连接");
          Socket socket;
          try {
              socket = new Socket(host, port);
          } catch (IOException e) {
              throw new RuntimeException("连接建立失败");
          }
          return socket;
      }
  
      public Object sendRequest(RPCRequest request) {
          Socket socket = null;
          ObjectOutputStream outputStream = null;
          ObjectInputStream inputStream = null;
          try {
              socket = newSocket();
              outputStream = new ObjectOutputStream(socket.getOutputStream());
              outputStream.writeObject(request);
              outputStream.flush();
              inputStream = new ObjectInputStream(socket.getInputStream());
              Object result = inputStream.readObject();
              return result;
          } catch(Exception e) {
              throw new RuntimeException("发送数据异常：" + e);
          } finally {
              if(socket != null) {
                  try {
                      socket.close();
                  } catch (IOException e) {
                      e.printStackTrace();
                  }
              }
              try {
                  outputStream.close();
                  inputStream.close();
  
              } catch (IOException e) {
                  e.printStackTrace();
              }
  
          }
      }
  }
  ```

* 测试

  ```java
  public class App {
      public static void main(String[] args) {
          RpcClientProxy proxy = new RpcClientProxy();  // 创建代理对象
  
          // TODO：还可以进一步优化，比如类似直接通过注解注入HelloService
          // 通过代理对象和socket远程连接并调用方法，返回service
          HelloService helloService = proxy.clientProxy(HelloService.class, "localhost", 8080);
  
          System.out.println(helloService.sayHello("mike")); // 调用远程方法
  
          User user = new User();
          user.setName("tom");
          System.out.println(helloService.saveUser(user));
  
      }
  }
  ```





# 面试



### REST和RPC的区别

1. REST是一种风格或者说是约束条件或者原则，是一种架构风格；满足这些约束条件和原则的应用程序或设计就是 RESTful。REST规范把所有内容都视为资源，网络上一切皆资源。 可以完全通过HTTP协议实现，使用 HTTP 协议处理数据通信。REST架构对资源的操作包括获取、创建、修改和删除资源的操作正好对应HTTP协议提供的GET、POST、PUT和DELETE方法。

   而RPC是远程方法调用的框架，实现框架有Dubbo、SpringCloud等

2. REST是完全通过HTTP协议进行网络传输通信的，而RPC大部分是通过TCP进行网络传输的

3. 内对RPC，对外REST























