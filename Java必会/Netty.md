

# Netty

IO=》网络通信、IO、socket =》BIO、NIO、多路复用器 =》netty

### 知识准备-计算机组成原理

1、内核：相当于一个程序，负责管理IO设备，如硬盘、网卡、外设等，统一资源控制

2、保护模式：内核能访问其他程序，其他程序无法直接访问内核，防止对内核及系统的侵入和破坏

3、中断：指时钟中断，由晶振实现，cpu指令为：INT x80（在中断向量表里255个地址的其中一个），实现了cpu系统调用，保证了保护现场、快速切换进程、恢复现场等功能。如果进程过多，会导致晶振数量过多，从而导致了cpu穿插在各进程之间的进程切换、保护现场、恢复现场所花费的时间变多，也就是浪费过多的时间在内核态和用户态的上下文切换，从而导致cpu性能下降。

4、Java线程：通过调用内核的系统调用，得到在操作系统里的一个轻量级的进程，并且生成一个线程id，同属于原来的生成该线程的一个进程组

### Reactor模式

IO复用结合线程池，就是Reactor模式基本设计思想。

1、通过一个或多个输入同时传递给服务处理器的模式，基于事件驱动

2、服务器端程序处理传入的多个请求，并将他们同步分派到相应的处理线程，因此也叫DIspatchar分发模式

3、使用IO复用监听事件，收到事件后分发给某个线程（进程），这点是网络服务器高并发处理的关键

![image-20201124165727377](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201124165727377.png)

#### 三大模式分类

根据Reactor数量和处理资源池线程的数量不同，有三种模式

单Reactor单线程，单Reactor多线程、主从Reactor多线程（演变为Netty模型）

1、单Reactor单线程：一个Reactor一个Handler

2、单Reactor多线程：一个Reactor多个Handler，一个Handler一个单独的线程，该并发下容易出现性能瓶颈

##### 3、主从Reactor多线程：多个Reactor多个Handler，一个Reactor子线程对应多个Handler

* 1、Reactor主线程MainReactor对象通过select监听连接事件，收到事件后，通过Acceptor处理连接请求
* 2、当Acceptor处理连接事件后，MainReactor将连接分发到Reactor子线程SubReactor
* 3、subReactor将连接加入到连接队列并进行监听，并创建Handler进行各种事件的处理
* 4、当有新事件发生时，subReactor就会调用对应的Handler进行处理
* 5、Handler通过read读取数据，分发给后面的worker线程处理
* 6、worker线程池分配独立的worker线程进行业务处理，并返回结果
* 7、handler收到响应结果后，再通过send将结果返回给client
* 8、Reactor主线程可以对应多个Reactor子线程，即MainReactor对应多个SubReactor
* 即：Reactor负责分发和建立连接，Handler负责读取和发送请求，Worker负责业务处理

![image-20201125085330958](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201125085330958.png)



#### Reactor核心组成

Reactor：在一个单独的线程中运行，负责监听和分发事件，分发给适当的处理程序来对IO事件做出反应

Handlers：eventHandler, 处理程序执行IO事件要完成的实际事件

### IO复用模型

Netty 的非阻塞 I/O 的实现关键是基于 I/O 复用模型，主要通过Selector实现

Netty 的 IO 线程 NioEventLoop 由于聚合了多路复用器 Selector，可以同时并发处理成百上千个客户端连接。

当线程从某客户端 Socket 通道进行读写数据时，若没有数据可用时，该线程可以进行其他任务。

线程通常将非阻塞 IO 的空闲时间用于在其他通道上执行 IO 操作，所以单独的线程可以管理多个输入和输出通道。

由于读写操作都是非阻塞的，这就可以充分提升 IO 线程的运行效率，避免由于频繁 I/O 阻塞导致的线程挂起。

一个 I/O 线程可以并发处理 N 个客户端连接和读写操作，这从根本上解决了传统同步阻塞 I/O 一连接一线程模型，架构的性能、弹性伸缩能力和可靠性都得到了极大的提升。

在 NIO 中，抛弃了传统的 I/O 流，而是引入了 Channel 和 Buffer 的概念。在 NIO 中，只能从 Channel 中读取数据到 Buffer 中或将数据从 Buffer 中写入到 Channel。基于 Buffer 操作不像传统 IO 的顺序操作，NIO 中可以随意地读取任意位置的数据。

### Netty线程模型

![image-20201125091048655](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201125091048655.png)

1、Netty对NIO的主从Reactor多线程模式进行改进，抽象出BossGroup专门负责接收客户端连接，WorkerGroup负责网络读写

2、BossGroup 和 WorkGroup 类型都是 NioEventGroup

3、NioEventGroup 相当于一个事件循环组，组中包含有多个事件循环，每一个事件循环是NioEventLoop

4、NioEventLoop 表示一个不断循环的执行处理任务的线程，每个 NioEventLoop 都有一个Selector，用于监听绑定在Selector上的网络通讯通道

5、NioEventGroup可以有多个线程，即可以包含多个NioEventLoop

6、每个 Boss Group 的 NioEventLoop 循环执行步骤有3步：

* 轮询Accept事件
* 处理Accept事件，与Client建立连接 ，生成 NioSocketChannel，并将其注册到某个Work Group的 NioEventLoop 上的Selector
* 处理任务队列中的任务，即runAllTasks

7、每个 Worker Group 的 NioEventLoop 循环执行步骤有3步：

* 轮询 read、write 事件
* 处理 IO 事件，即 read、write 事件，在对应的 NioSocketChannel 处理
* 处理任务队列的任务，即 runAllTasks

8、每个 Worker Group 的 NioEventLoop 处理业务时，会使用 Pipeline（管道），pipeline中包含channel，通过 pipeline 可以获取到对应通道 channel，pipeline 中维护了很多处理器

#### 注意点

* 每个NioEventGroup包含（cpu核数*2）个数的NioEventLoop（相当于NIO的Reactor线程），且每个NioEventLoop包含一个Selector和一个taskQueue
* ChannelPipeline管道的底层是一个双向链表，出栈（OutBoundEvent）入栈（InBoundEvent），循环监听事件连接
* ChannelHandlerContext是上下文对象，底层是双向链表，可以通过该对象获取 DefaultChannelPipeline、pipeline 里的 NioSocketChannel、channel里的eventLoop等
* DefaultChannelPipeline 可以获取对应的 NioSocketChannel，NioSocketChannel 也可以获取到 DefaultChannelPipeline
* 每个NioSocketChannel只会绑定唯一的NioEventLoop和自己的channelPipeline

#### Netty是一个异步事件驱动框架，怎么理解

1.异步怎么理解：外部线程执行write或者execute任务的时候会立马返回，为什么会立马返回，是因为他内部有一个牛逼哄哄的mpsc(多生产者单消费者)队列，所有外部线程的任务都给扔到这个队列里，同时把回调，也就是future绑定在这个任务上，reactor线程会在第三步挨个执行这些任务，执行完之后callback，典型的观察者模式
2.事件怎么理解：netty内部默认两倍cpu核数个reactor线程，每个reactor线程维护一个selector，每个selector维护一堆channel，每个channel都把读写等事件都注册到selector上
3.驱动怎么理解：netty里面默认所有的操作都在reactor线程中执行，reactor线程核心的就是run方法，所有的操作都由这个方法来驱动，驱动的过程分为三个过程，轮询事件，处理事件，处理异步任务

### Netty任务

1、用户程序自定义普通任务 taskQueue

```java
// 异步任务
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
ctx.channel().eventLoop().execute(new Runnable() {
    @Override
    public void run() {
    }
});
```

2、用户自定义定时任务 scheduleTaskQueue

```java
// 定时任务
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
ctx.channel().eventLoop().schedule(new Runnable() {
    @Override
    public void run() {
    }
})
```

3、非当前Reactor线程调用Channel各种方法

### Netty异步模型

建立在 future 和 callback 之上的，callback就是回调。

Future核心思想是：假设有一个方法 fun() ，计算过程很耗时，可以在调用fun方法的时候，立马返回一个 Future，后续可以通过 Future 去监控fun方法的处理过程，这就是 Future-Listener 机制

#### Future：

在Netty中的future是ChannelFuture，一个接口，可以添加监听器，当监听的时间发生时，就会通知到监听器

# Netty核心模块

## 1.Bootstrap、ServerBootstrap

### （1）简介

Bootstrap 意思是引导，一个 Netty 应用通常由一个 Bootstrap 开始，主要作用是配置整个 Netty 程序，串联各个组件，Netty 中 Bootstrap 类是客户端程序的启动引导类，ServerBootstrap 是服务端启动引导类

### （2）常用方法

| 方法                                                         | 描述                                       |
| :----------------------------------------------------------- | :----------------------------------------- |
| group(EventLoopGroup parentGroup, EventLoopGroup childGroup) | 该方法用于服务器端，用来设置两个 EventLoop |
| group(EventLoopGroup group)                                  | 该方法用于客户端，用来设置一个 EventLoop   |
| channel(Class<? extends C> channelClass)                     | 该方法用来设置一个服务器端的通道实现       |
| option(ChannelOption<T> option, T value)                     | 用来给 ServerChannel 添加配置              |
| childOption(ChannelOption<T> childOption, T value)           | 用来给接收到的通道添加配置                 |
| childHandler(ChannelHandler childHandler)                    | 该方法用来设置业务处理类                   |
| bind(int inetPort)                                           | 该方法用于服务器端，用来设置占用的端口号   |
| connect(String inetHost, int inetPort)                       | 该方法用于客户端，用来连接服务器端         |

## 2.Future、ChannelFuture

### （1）简介

Netty 中所有的 IO 操作都是异步的，不能立刻得知消息是否被正确处理。但是可以过一会等它执行完成或者直接注册一个监听，具体的实现就是通过 Future 和 ChannelFutures，他们可以注册一个监听，当操作执行成功或失败时监听会自动触发注册的监听事件

### （2）常用方法

| 方法                 | 描述                           |
| :------------------- | :----------------------------- |
| Channel channel()    | 返回当前正在进行 IO 操作的通道 |
| ChannelFuture sync() | 等待异步操作执行完毕           |

## 3.Channel

- Netty 网络通信的组件，能够用于执行网络 I/O 操作
- 通过Channel可获得当前网络连接的通道的状态
- 通过Channel可获得网络连接的配置参数(例如接收缓冲区大小)
- Channel 提供异步的网络 I/O 操作(如建立连接，读写，绑定端口)，异步调用意味着任何 I/O 调用都将立即返回，并且不保证在调用结束时所请求的 I/O 操作已完成
- 调用立即返回一个 ChannelFuture 实例，通过注册监听器到ChannelFuture 上，可以 I/O 操作成功、失败或取消时回调通知调用方
- 支持关联 I/O 操作与对应的处理程序
- 不同协议、不同的阻塞类型的连接都有不同的 Channel 类型与之对应，常用的 Channel 类型:

| 类                     | 介绍                                                         |
| :--------------------- | :----------------------------------------------------------- |
| NioSocketChannel       | 异步的客户端 TCP Socket 连接                                 |
| NioServerSocketChannel | 异步的服务器端 TCP Socket 连接                               |
| NioDatagramChannel     | 异步的 UDP 连接                                              |
| NioSctpChannel         | 异步的客户端 Sctp 连接                                       |
| NioSctpServerChannel   | 异步的 Sctp 服务器端连接，这些通道涵盖了 UDP 和 TCP 网络 IO 以及文件 IO |

## 4.Selector

- Netty 基于 Selector 对象实现 I/O 多路复用，通过 Selector 一个线程可以监听多个连接的 Channel 事件
- 当向一个 Selector 中注册 Channel 后，Selector 内部的机制就可以自动不断地查询(Select) 这些注册的 Channel 是否有已就绪的 I/O 事件(例如可读，可写，网络连接完成等)，这样程序就可以很简单地使用一个线程高效地管理多个 Channel

## 5.ChannelHandler及其实现类

- ChannelHandler 是一个接口，处理 I/O 事件或拦截 I/O 操作，并将其转发到其 ChannelPipeline(业务处理链) 中的下一个处理程序
- ChannelHandler 本身并没有提供很多方法，因为这个接口有许多的方法需要实现，方便使用期间，可以继承它的子类
- ChannelHandler 及其实现类的类图如下：

![img](https://img-blog.csdnimg.cn/20191216121653512.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Z5X2phdmExOTk1,size_16,color_FFFFFF,t_70)

 

- 我们经常需要自定义一个 Handler 类去继承ChannelInboundHandlerAdapter，然后通过重写相应方法实现业务逻辑

## 6.Pipeline和ChannelPipeline  

- ChannelPipeline 是一个 Handler 的集合，它负责处理和拦截 inbound 或者 outbound 的事件和操作，相当于 一个贯穿 Netty 的链。(也可以这样理解:ChannelPipeline 是 保存 ChannelHandler 的 List，用于处理或拦截 Channel 的入站事件和出站操作)
- ChannelPipeline 实现了一种高级形式的拦截过滤器模式，使用户可以完全控制事件的处理方式，以及 Channel 中各个的 ChannelHandler 如何相互交互
- 在 Netty 中每个 Channel 都有且仅有一个 ChannelPipeline 与之对应，它们的组成关系如下:
  - 一个Channel包含了一个ChannelPipeline，而ChannelPipeline中又维护了一个由ChannelHandlerContext组成的双向链表，并且每个ChannelHandlerContext中又关联着一个ChannelHandler
  - 入站时间和出站时间在一个双向链表中，入站事件会从链表head往后传递到最后一个入站的handler，出站事件会从链表tail往前传递到最前一个出站的handler，两种类型的handler互不干扰

![img](https://img-blog.csdnimg.cn/20191216121726997.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Z5X2phdmExOTk1,size_16,color_FFFFFF,t_70)

 

- 常用方法

| 方法                               | 描述                                              |
| :--------------------------------- | :------------------------------------------------ |
| addFirst(ChannelHandler… handlers) | 把一个业务处理类(handler)添加到链中的第一个位置   |
| addLast(ChannelHandler… handlers)  | 把一个业务处理类(handler)添加到链中的最后一个位置 |

## 7.ChannelHandlerContext

- 保存 Channel 相关的所有上下文信息，同时关联一个 ChannelHandler 对象
- ChannelHandlerContext中包含一个具体的事件处理器 ChannelHandler，同时 ChannelHandlerContext 中也绑定了对应的 pipeline 和 Channel 的信息，方便对 ChannelHandler 进行调用
- 常用方法

| 方法                      | 描述                                                         |
| :------------------------ | :----------------------------------------------------------- |
| close()                   | 关闭通道                                                     |
| flush()                   | 刷新                                                         |
| writeAndFlush(object msg) | 将数据写到ChannelPipeline中，当前ChannelHandler的下一个ChannelHandler开始处理 |

## 8.ChannelOption

- Netty 在创建 Channel 实例后,一般都需要设置 ChannelOption 参数
- ChannelOption 参数如下:
  - ChannelOption.SO_BACKLOG：对应TCP/IP协议listen函数中的backlog参数，用来初始化服务器可连接队列。服务端处理客户端连接请求是顺序处理的，所以同一时间只能处理一个客户端连接。多个客户端来的时候，服务端将不能处理的客户端连接请求放在队列中等待处理，backlog参数指定了队列的大小
  - ChannelOption.SO_KEEPALIVE：一直保持连接活动状态

## 9.EventLoopGroup

- EventLoopGroup 是一组 EventLoop 的抽象，Netty 为了更好的利用多核 CPU 资源，一般会有多个 EventLoop 同时工作，每个 EventLoop 维护着一个 Selector 实例
- EventLoopGroup 提供 next 接口，可以从组里面按照一定规则获取其中一个 EventLoop 来处理任务。在 Netty 服务器端编程中，我们一般都需要提供两个 EventLoopGroup，例如:BossEventLoopGroup 和 WorkerEventLoopGroup
- 通常一个服务端口即一个ServerSocketChannel对应一个Selector和一个EventLoop线程。BossEventLoop 负责接收客户端的连接并将 SocketChannel 交给 WorkerEventLoopGroup 来进行 IO 处理，如下图所示

![img](https://img-blog.csdnimg.cn/20191216121812169.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Z5X2phdmExOTk1,size_16,color_FFFFFF,t_70)

 

## 10.Unpooled 类

- Netty 提供一个专门用来操作缓冲区(即 Netty 的数据容器)的工具类
- 常用方法如下

```
//通过给定的数据和字符编码返回一个ByteBuf对象
ByteBuf byteBuf = Unpooled.copiedBuffer("hello,world!", Charset.forName("utf-8"));
```

* ByteBuf中不需要使用flip进行读写模式反转，底层维护了 readerIndex 和 writerIndex，分别指下一个读的位置和下一个写的位置
* 通过 readerIndex 和 writerIndex 和 capacity 将 buffer 分为三个区域：0-readerIndex为已经读取的区域，readerIndex-writerIndex为可读的区域，writerIndex-capacity为可写的区域

### Netty基本使用

Netty是一个异步事件驱动的网络应用框架，用于快速开发可维护的高性能服务器和客户端。

1. 使用JDK自带的NIO需要了解太多的概念，编程复杂
2. Netty底层IO模型随意切换，而这一切只需要做微小的改动，改改参数，Netty可以直接从NIO模型变身为IO模型
3. Netty自带的拆包解包，异常检测等机制让你从NIO的繁重细节中脱离出来，让你只需要关心业务逻辑
4. Netty解决了JDK的很多包括空轮询在内的bug
5. Netty底层对线程，selector做了很多细小的优化，精心设计的reactor线程模型做到非常高效的并发处理
6. 自带各种协议栈让你处理任何一种通用协议都几乎不用亲自动手
7. Netty社区活跃，遇到问题随时邮件列表或者issue
8. Netty已经历各大rpc框架，消息中间件，分布式通信中间件线上的广泛验证，健壮性无比强大

```
<!-- https://mvnrepository.com/artifact/io.netty/netty-all -->
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.42.Final</version>
</dependency>

```

NettyServer

```java
public class NettyServer {
    public static void main(String[] args) throws InterruptedException {
        // 负责处理连接请求，该线程组下的子线程（NioEventLoop）数量为 cpu核数*2
        NioEventLoopGroup bossGroup = new NioEventLoopGroup();
        // 负责客户端业务处理，该线程组下的子线程（NioEventLoop）数量为 cpu核数*2，都放在EventExecutor数组里进行管理
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            // 创建服务端启动对象，配置参数
            ServerBootstrap bootstrap = new ServerBootstrap();
            bootstrap.group(bossGroup, workerGroup) // 设置两个线程组
                    .channel(NioServerSocketChannel.class) // 作为服务器的通道实现
                    .option(ChannelOption.SO_BACKLOG, 128) // 设置线程队列等待连接的个数
                    .childOption(ChannelOption.SO_KEEPALIVE, true) // 设置保持活动连接状态
                    .childHandler(new ChannelInitializer<SocketChannel>() { // 创建通道初始化对象
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new NettyServerHandler());
                        }
                    }); // workerGroup的 EventLoop对应的管道设置处理器
            System.out.println("。。。服务器 is ready。。。");
            // 同步生成一个ChannelFuture对象
            // 启动服务器，绑定端口
            ChannelFuture cf = bootstrap.bind(6668).sync();
            // 对关闭通道进行监听
            cf.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

NettyServerHandler

```java
public class NettyServerHandler extends ChannelInboundHandlerAdapter {
    // 读取数据事件，这里读取客户端发送的信息，通过Pipeline传输
    /**
     * 读取数据
     * @param ctx 含有pipeline、channel、地址
     * @param msg 客户端发送来的数据
     * @throws Exception
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.out.println("server ctx = " + ctx);
        ByteBuf buf = (ByteBuf) msg;
        System.out.println("客户端发送消息是：" + buf.toString(CharsetUtil.UTF_8));
        System.out.println("客户端地址：" + ctx.channel().remoteAddress());
    }

    // 数据读取完毕
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.writeAndFlush(Unpooled.copiedBuffer("hello, 客户端~", CharsetUtil.UTF_8)); // 将数据写入到缓存并刷新，刷新就是缓存的数据写到pipeline管道上
    }
    
    // 处理异常
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

NettyClient

```java
public class NettyClient {
    public static void main(String[] args) throws InterruptedException {
        // 客户端需要一个时间循环组
        NioEventLoopGroup group = new NioEventLoopGroup();
        try {
            // 客户端启动对象
            Bootstrap bootstrap = new Bootstrap();
            // 设置相关参数
            bootstrap.group(group) // 设置线程组
                .channel(NioSocketChannel.class) // 设置客户端通道的实现类（发射）
                .handler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel ch) throws Exception {
                        ch.pipeline().addLast(new NettyClientHandler()); // 加入自己的处理器
                    }
                });
            System.out.println("客户端 ok。。。");
            // 启动客户端去连接服务器端，返回一个future通道
            ChannelFuture channelFuture = bootstrap.connect("127.0.0.1", 6668).sync();
            // 给channelFuture注册监听器，监控关心的事件
            channelFuture.addListener(new ChannelFutureListener() {
                @Override
                public void operationComplete(ChannelFuture future) throws Exception {
                    if(channelFuture.isSuccess()) {
                        System.out.println("监听端口6668成功");
                    } else {
                        System.out.println("监听端口6668成功");
                    }
                }
            });
            // 给关闭通道进行监听
            channelFuture.channel().closeFuture().sync();
        } finally {
            group.shutdownGracefully();
        }
    }
}
```

NettyClientHandler

```java
public class NettyClientHandler extends ChannelInboundHandlerAdapter {
    // 当通道就绪就会触发该方法
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("client " + ctx);
        ctx.writeAndFlush(Unpooled.copiedBuffer("hello, server", CharsetUtil.UTF_8));
    }

    // 当通道有读取事件时，会触发
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf buf = (ByteBuf) msg;
        System.out.println("服务器回复的消息是：" + buf.toString(CharsetUtil.UTF_8));
        System.out.println("服务器的地址：" + ctx.channel().remoteAddress());
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

### Netty实现HTTP服务

HttpServerHandler

```java
public class HttpServerHandler extends SimpleChannelInboundHandler<HttpObject> {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, HttpObject msg) throws Exception {
        if(msg instanceof HttpRequest) {
            ByteBuf content = Unpooled.copiedBuffer("Hello", CharsetUtil.UTF_8);
            FullHttpResponse response = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK, content);
            response.headers().set(HttpHeaderNames.CONTENT_TYPE, "text/plain");
            response.headers().set(HttpHeaderNames.CONTENT_LENGTH, content.readableBytes());
            ctx.writeAndFlush(response);
        }
    }
}
```

ServerInitializer

```java
public class ServerInitializer extends ChannelInitializer<SocketChannel> {
    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline pipeline = ch.pipeline();
        pipeline.addLast("MyHttpServerCodec", new HttpServerCodec());
        pipeline.addLast("myHttpServerHandler", new HttpServerHandler());
    }
}
```

HttpServer

```java
public static void main(String[] args) {
    // 负责处理连接请求
    NioEventLoopGroup bossGroup = new NioEventLoopGroup();
    // 负责客户端业务处理
    NioEventLoopGroup workerGroup = new NioEventLoopGroup();
    try {
        // 创建服务端启动对象，配置参数
        ServerBootstrap bootstrap = new ServerBootstrap();
        bootstrap.group(bossGroup, workerGroup) // 设置两个线程组
                .channel(NioServerSocketChannel.class) // bossGroup的NioSocketChannel，作为服务器的通道实现
                .childHandler(new ServerInitializer()); // workerGroup的 EventLoop对应的管道设置处理器
        // 同步生成一个ChannelFuture对象
        // 启动服务器，绑定端口
        ChannelFuture cf = bootstrap.bind(6668).sync();
        // 对关闭通道进行监听
        cf.channel().closeFuture().sync();
    } catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        bossGroup.shutdownGracefully();
        workerGroup.shutdownGracefully();
    }
}
```

#### Netty实现群聊系统

GroupServer

```java
public class GroupChatServer {
    private int port;
    public GroupChatServer(int port) {
        this.port = port;
    }
    public void run() throws InterruptedException {
        NioEventLoopGroup bossGroup = new NioEventLoopGroup(1);
        NioEventLoopGroup workerGroup = new NioEventLoopGroup(8);
        ServerBootstrap bootstrap = new ServerBootstrap();
        try {
            bootstrap.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .option(ChannelOption.SO_BACKLOG, 128)
                    .childOption(ChannelOption.SO_KEEPALIVE, true)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ChannelPipeline pipeline = ch.pipeline();
                            pipeline.addLast("decoder", new StringDecoder()); // 解码器
                            pipeline.addLast("encoder", new StringEncoder());
                            pipeline.addLast(new GroupChatServerHandler());
                        }
                    });
            System.out.println("netty 服务器启动");
            ChannelFuture channelFuture = bootstrap.bind(port).sync();// 同步阻塞绑定服务端端口
            channelFuture.channel().closeFuture().sync(); // 关闭通道
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new GroupChatServer(7000).run();
    }

}
```

GroupServerHandler

```java
public class GroupChatServerHandler extends SimpleChannelInboundHandler<String> {

    // channel组，管理客户端所有channel
    // GlobalEventExecutor: 全局事件执行器，是一个单例
    private static ChannelGroup channelGroup = new DefaultChannelGroup(GlobalEventExecutor.INSTANCE);

    SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    // 表示客户端连接建立，即上线，一旦连接就第一个被执行的方法
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        Channel channel = ctx.channel();
        // 将该客户端的上线信息转发到其他在线的客户端,
        // 将刚连接的客户端上线信息写到缓冲区后刷新到pipeline管道，才发送信息
        // 将ChannelGroup中的channel遍历并发送消息，从而其他在线的客户单就能收到消息
        channelGroup.writeAndFlush("[客户端]" + channel.remoteAddress() + "加入聊天" + simpleDateFormat.format(new Date()) + "\n");
        channelGroup.add(channel);
    }

    // 表示cahnnel处于活动状态，一般只在服务端提示 xxx上线
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println(ctx.channel().remoteAddress() + "上线了");
    }
    // 只在服务端提示xxx离线
    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        System.out.println(ctx.channel().remoteAddress() + "离线了");
    }

    // 断开连接
    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
        Channel channel = ctx.channel();
        channelGroup.writeAndFlush("[客户端]" + channel.remoteAddress() + "离开了\n");
        // 此channel会自动被channelGroup remove
        System.out.println("channelGroup size" + channelGroup.size());
    }

    // 读取客户端发送的数据，并转发给其他在线客户端
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
        Channel channel = ctx.channel();
        channelGroup.forEach(ch -> {
            // 不是当前channel
            if(channel != ch) {
                ch.writeAndFlush("[客户]" + channel.remoteAddress() + "发送了消息" + msg + "\n");
            } else {
                ch.writeAndFlush("[自己]发送了消息" + msg + "\n");
            }
        });
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}
```

GroupClient

```java
public class GroupChatClient {

    private final String host;
    private final int port;

    public GroupChatClient(String host, int port) {
        this.host = host;
        this.port = port;
    }

    public void run() throws InterruptedException {
        NioEventLoopGroup group = new NioEventLoopGroup();
        Bootstrap bootstrap = new Bootstrap();
        try {
            bootstrap.group(group)
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ChannelPipeline pipeline = ch.pipeline();
                            pipeline.addLast("decoder", new StringDecoder());
                            pipeline.addLast("encoder", new StringEncoder());
                            pipeline.addLast(new GroupChatClientHandler());
                        }
                    });
            ChannelFuture channelFuture = bootstrap.connect(host, port).sync();
            // 连接后生成的channel, 用户传输数据
            Channel channel = channelFuture.channel();

            System.out.println("------" + channel.localAddress() + "------");
            // 客户端输入信息，创建一个扫描器
            Scanner scanner = new Scanner(System.in);
            while(scanner.hasNextLine()) {
                String msg = scanner.nextLine();
                channel.writeAndFlush(msg + "\r\n");
            }
        } finally {
            group.shutdownGracefully();
        }

    }

    public static void main(String[] args) throws InterruptedException {
        new GroupChatClient("127.0.0.1", 7000).run();
    }

}
```

GroupClientHandler

```java
public class GroupChatClientHandler extends SimpleChannelInboundHandler<String> {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
        System.out.println(msg.trim());
    }
}
```





































