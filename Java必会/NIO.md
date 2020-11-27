# NIO

NIO主要有**三个核心部分组成**：

- **buffer缓冲区**
- **Channel管道**
- **Selector选择器**

在NIO中并不是以流的方式来处理数据的，而是以buffer缓冲区和Channel管道**配合使用**来处理数据。

NIO就是**通过Channel管道运输着存储数据的Buffer缓冲区的来实现数据的处理**！

Channel不与数据打交道，它只负责运输数据。与数据打交道的是Buffer缓冲区

![image-20201123125747513](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201123125747513.png)

### Selector

```java
abstract class Selector implements Closeable
Selector open()  // 创建当前线程的唯一一个选择器
int select() // 返回当前选择器的带有事件的连接数量，阻塞操作
int select(1000) //返回当前选择器的带有事件的连接数量，阻塞1000毫秒
Selector wakeup(); // 唤醒selector
int selectNow() // 不阻塞，立刻返还
Set<SelectionKey> keys(); // 以set格式，返回当前selector所有连接
Set<SelectionKey> selectedKeys() // 以set格式，返回当前selector的所有 selectionKey
void close() // 释放当前选择器资源
```

#### ServerSocketChannel

在服务器端监听新的客户端socket连接

```java
sabstract class ServerSocketChannel
    extends AbstractSelectableChannel
    implements NetworkChannel
ServerSocketChannel open() // 新建监听连接通道
ServerSocketChannel bind(SocketAddress local, int backlog) // 绑定ServerSocketChannel的地址
ServerSocket socket() // 获取socket
SocketChannel accept() // 监听并接收带有请求连接事件的socketChannel通道连接，并返回该传输数据的通道socketChannel
SocketAddress getLocalAddress() // 
SelectionKey register(Selector sel, int ops)// 绑定事件，告诉Selector当前ServerSocketChannel通道或SocketChannel通道关注的事件
boolean configureBlocking(boolean blok); // false表示采用非阻塞
```

#### SocketChannel

一个Selector可以注册多个SocketChannel，网络IO通道，具体负责读写操作，把缓冲区数据写入通道或把通道里的数据读到缓冲区，读和取是相对缓冲区而言

```java
abstract class SocketChannel
    extends AbstractSelectableChannel
    implements ByteChannel, ScatteringByteChannel, GatheringByteChannel, NetworkChannel // 负责读写，因此多实现了三个字节通道接口
SocketChannel open() // 新建传输数据的通道
SocketChannel bind(SocketAddress local) // 当前通道SocketChannel绑定ip
SelectionKey register() // 将socketChannel注册到Selector上，并设置该连接的事件，相当于客户端的socket，用于传输数据，返回SelectionKey
Socket socket(); // 获取socket
int read(ByteBuffer dst) // 读取缓冲区中的数据
int write(ByteBuffer src) // 将数据写到缓冲区
// 绑定事件，告诉Selector当前ServerSocketChannel通道或SocketChannel通道关注的事件
SelectionKey register(Selector sel, int ops, Object att) // SocketChannel注册到Selector，返回事件连接，att参数设置共享数据 
boolean configureBlocking(boolean blok); // false表示采用非阻塞
abstract <T> SocketChannel setOption(SocketOption<T> name, T value) // 设置事件
abstract boolean connect(SocketAddress remote); // 连接服务器
abstract boolean finishConnect()  // 如果上面方法连接失败，该方法完成连接操作
abstract int read(ByteBuffer dst)	// 从通道里读数据到缓冲区
abstract int write(ByteBuffer dst)    // 将缓冲区的数据写到通道里
void close()  // 关闭通道
    
```

#### SelectionKey

连接事件，告诉Selector当前ServerSocketChannel通道或SocketChannel通道关注的事件

```java
public abstract class SelectionKey
Selector selector(); // 反向获取该事件连接对应的selector选择器
SelectableChannel channel(); // 反向获取该事件连接对应的channel通道，可以通过得到的channel完成业务处理
Object attachment(); // 得到与之关联的共享数据
SelectionKey interestOps(int ops); // 设置或改变监听事件

int OP_READ = 1 << 0; // 1 一般用于客户端通道SocketChannel，用于设置通道为可读事件
int OP_WRITE = 1 << 2; // 4  一般用于服务端通道ServerSocketChannel, 用于设置通道为可写事件
int OP_CONNECT = 1 << 3; // 8 一般用于客户端通道SocketChannel, 用于设置通道为可请求连接事件
int OP_ACCEPT = 1 << 4; // 16 一般用于服务端通道ServerSocketChannel，设置为可接收连接事件
boolean isReadable() // 
boolean isWritable() // 
boolean isConnectable() // 
boolean isAcceptable() // 该SelectionKey是OP_ACCEPT事件，表示该Selector选择器包含可接收连接事件
```

#### NIO模型

![image-20201123194744091](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201123194744091.png)















### Buffer

Buffer类维护了4个核心变量属性来提供**关于其所包含的数组的信息**。它们是：

- 容量Capacity

- - **缓冲区能够容纳的数据元素的最大数量**。容量在缓冲区创建时被设定，并且永远不能被改变。(底层是数组)

- 上界Limit

- - **缓冲区里的数据的总数**，代表了当前缓冲区中一共有多少数据。

- 位置Position

- - **下一个要被读或写的元素的位置**。Position会自动由相应的 `get( )`和 `put( )`函数更新。

- 标记Mark

- - 一个备忘位置。**用于记录上一次读写的位置**。

get() 和 put() 方法分别用于从缓冲区读取数据和写数据到缓冲区，

`flip()`方法可以**改动position和limit的位置**，**从缓存区拿数据**。当调用完`filp()`：**limit是限制读到哪里，而position是从哪里读**，`filp()`为**“切换成读模式”**

**读完我们还想写数据到缓冲区**，那就使用`clear()`函数，这个函数会“清空”缓冲区。数据没有真正被清空，只是被**遗忘**掉了

#### 直接与非直接缓冲区

- 非直接缓冲区是**需要**经过一个：copy的阶段的(从内核空间copy到用户空间)
- 直接缓冲区**不需要**经过copy阶段，也可以理解成--->**内存映射文件**，(上面的图片也有过例子)。

使用直接缓冲区有两种方式：

- 缓冲区创建的时候分配的是直接缓冲区
- 在FileChannel上调用`map()`方法，将文件直接映射到内存中创建

### Channel

![image-20201123132433152](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201123132433152.png)

使用**FileChannel配合缓冲区**实现文件复制的功能

使用**内存映射文件**的方式实现**文件复制**的功能(直接操作缓冲区)

```
MappedByteBuffer inMappedBuf = inChannel.map(MAPMODE.READ_ONLY, 0, inChannel.size());
```

通道之间通过`transfer()`实现数据的传输(直接操作缓冲区)

```
(FileChannel)inChannel.transferTo(0, inChannel.size, (FileChannel) outChannel)
```

### scatter、gather

分散读取(scatter)：将一个通道中的数据分散读取到多个缓冲区中

聚集写入(gather)：将多个缓冲区中的数据集中写入到一个通道中

## NIO模型

**文件描述符fd：**Linux 的内核将所有外部设备**都看做一个文件来操作**，对一个文件的读写操作会**调用内核提供的系统命令(api)**，返回一个`file descriptor`（fd，文件描述符）。而对一个socket的读写也会有响应的描述符，称为`socket fd`（socket文件描述符），描述符就是一个数字，**指向内核中的一个结构体**（文件路径，数据区等一些属性）。在Linux下对文件的操作是**利用文件描述符fd来实现的**。

##### 阻塞I/O模型：在进程(用户)空间中调用`recvfrom`，其系统调用直到数据包到达且**被复制到应用进程的缓冲区中或者发生错误时才返回**，在此期间**一直等待**。

##### 阻塞I/O模型：`recvfrom`从应用层到内核的时候，如果没有数据就**直接返回**一个EWOULDBLOCK错误，一般都对非阻塞I/O模型**进行轮询检查这个状态**，看内核是不是有数据到来。

#### **I/O复用模型**：

**通常**使用NIO是在网络IO中使用的，NIO都是在**网络通信的基础之上**，NIO是非阻塞的NIO也是**网络中体现**的，网络中使用NIO往往是I/O模型的**多路复用模型**！

Selector选择器就好比提醒取餐的**广播**。**一个线程能够管理多个Channel的状态**

调用`select/poll/epoll/pselect`其中一个函数，**传入多个文件描述符**，如果有一个文件描述符**就绪，则返回**，否则阻塞直到超时。

```
// poll
int poll(struct pollfd *fds,nfds_t nfds, int timeout);
struct pollfd {
    int fd;         /* 文件描述符 */
    short events;         /* 等待的事件 */
    short revents;       /* 实际发生了的事件 */
};
```

- （1）当用户进程调用了select，那么整个进程会被block；
- （2）而同时，kernel会“监视”所有select负责的socket；
- （3）当任何一个socket中的批量数据准备好了，select就会返回；
- （4）这个时候用户进程再调用read操作，将数据从kernel批量拷贝到用户进程(空间)。

所以，I/O 多路复用的特点是**通过一种机制一个进程能同时等待多个文件描述符**，而这些文件描述符**其中的任意一个进入读就绪状态**，select()函数**就可以返回**。

select/epoll的优势并不是对于单个连接能处理得更快，而是**在于能处理更多的连接**。

#### 非阻塞NIO用法

##### 服务端

```java
public class NoBlockServer {
    public static void main(String[] args) throws IOException {
        // 1.获取通道
        ServerSocketChannel server = ServerSocketChannel.open();
        // 2.切换成非阻塞模式
        server.configureBlocking(false);
        // 3. 绑定连接
        server.bind(new InetSocketAddress(6666));
        // 4. 获取选择器
        Selector selector = Selector.open();
        // 4.1将通道注册到选择器上，指定接收“监听通道”事件
        server.register(selector, SelectionKey.OP_ACCEPT);
        // 5. 轮训地获取选择器上已“就绪”的事件--->只要select()>0，说明已就绪
        while (selector.select() > 0) {
            // 6. 获取当前选择器所有注册的“选择键”(已就绪的监听事件)
            Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
            // 7. 获取已“就绪”的事件，(不同的事件做不同的事)
            while (iterator.hasNext()) {
                SelectionKey selectionKey = iterator.next();
                // 接收事件就绪
                if (selectionKey.isAcceptable()) {
                    // 8. 获取客户端的链接
                    SocketChannel client = server.accept();
                    // 8.1 切换成非阻塞状态
                    client.configureBlocking(false);
                    // 8.2 注册到选择器上-->拿到客户端的连接为了读取通道的数据(监听读就绪事件)
                    client.register(selector, SelectionKey.OP_READ);
                } else if (selectionKey.isReadable()) { // 读事件就绪
                    // 9. 获取当前选择器读就绪状态的通道
                    SocketChannel client = (SocketChannel) selectionKey.channel();
                    // 9.1读取数据
                    ByteBuffer buffer = ByteBuffer.allocate(1024);
                    // 9.2得到文件通道，将客户端传递过来的图片写到本地项目下(写模式、没有则创建)
                    FileChannel outChannel = FileChannel.open(Paths.get("2.png"), StandardOpenOption.WRITE, StandardOpenOption.CREATE);
                    while (client.read(buffer) > 0) {
                        // 在读之前都要切换成读模式
                        buffer.flip();
                        outChannel.write(buffer);
                        // 读完切换成写模式，能让管道继续读取文件的数据
                        buffer.clear();
                    }
                }
                // 10. 取消选择键(已经处理过的事件，就应该取消掉了)
                iterator.remove();
            }
        }
    }
}
```

##### 客户端

```java
public class NoBlockClient2 {
    public static void main(String[] args) throws IOException {
        // 1. 获取通道
        SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 6666));
        // 1.1切换成非阻塞模式
        socketChannel.configureBlocking(false);
        // 1.2获取选择器
        Selector selector = Selector.open();
        // 1.3将通道注册到选择器中，获取服务端返回的数据.在客户端上要想获取得到服务端的数据，也需要注册在register上(监听读事件)！
        socketChannel.register(selector, SelectionKey.OP_READ);
        // 2. 发送一张图片给服务端吧
        FileChannel fileChannel = FileChannel.open(Paths.get("X:\\Users\\ozc\\Desktop\\新建文件夹\\1.png"), StandardOpenOption.READ);
        // 3.要使用NIO，有了Channel，就必然要有Buffer，Buffer是与数据打交道的呢
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        // 4.读取本地文件(图片)，发送到服务器
        while (fileChannel.read(buffer) != -1) {
            // 在读之前都要切换成读模式
            buffer.flip();
            socketChannel.write(buffer);
            // 读完切换成写模式，能让管道继续读取文件的数据
            buffer.clear();
        }

        // 5. 轮训地获取选择器上已“就绪”的事件--->只要select()>0，说明已就绪
        while (selector.select() > 0) {
            // 6. 获取当前选择器所有注册的“选择键”(已就绪的监听事件)
            Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
            // 7. 获取已“就绪”的事件，(不同的事件做不同的事)
            while (iterator.hasNext()) {
                SelectionKey selectionKey = iterator.next();
                // 8. 读事件就绪
                if (selectionKey.isReadable()) {
                    // 8.1得到对应的通道
                    SocketChannel channel = (SocketChannel) selectionKey.channel();
                    ByteBuffer responseBuffer = ByteBuffer.allocate(1024);
                    // 9. 知道服务端要返回响应的数据给客户端，客户端在这里接收
                    int readBytes = channel.read(responseBuffer);
                    if (readBytes > 0) {
                        // 切换读模式
                        responseBuffer.flip();
                        System.out.println(new String(responseBuffer.array(), 0, readBytes));
                    }
                }
                // 10. 取消选择键(已经处理过的事件，就应该取消掉了)
                iterator.remove();
            }
        }
    }
}
```

- 将Socket通道注册到Selector中，监听感兴趣的事件
- 当感兴趣的时间就绪时，则会进去我们处理的方法进行处理
- 每处理完一次就绪事件，删除该选择键（迭代器）(因为我们已经处理完了)

### NIO实现UDP

![image-20201123151914264](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201123151914264.png)

### pipe管道

###### NIO的管道Pipe是2个线程之间的单项数据连接，pipe有source通道和sink通道，数据会被写到sink通过，客户端从source通道接收数据

![image-20201123152019893](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201123152019893.png)



### IO主要问题

1. 线程资源受限：线程是操作系统中非常宝贵的资源，同一时刻有大量的线程处于阻塞状态是非常严重的资源浪费，操作系统耗不起
2. 线程切换效率低下：单机cpu核数固定，线程爆炸之后操作系统频繁进行线程切换，应用性能急剧下降。
3. 除了以上两个问题，IO编程中，我们看到数据读写是以字节流为单位，效率不高。

 为了解决这三个问题，JDK在1.4之后提出了NIO。

### 线程资源受限

NIO编程模型中，新来一个连接不再创建一个新的线程，而是可以把这条连接直接绑定到某个固定的线程，然后这条连接所有的读写都由这个线程来负责

如上图所示，IO模型中，一个连接来了，会创建一个线程，对应一个while死循环，死循环的目的就是不断监测这条连接上是否有数据可以读，大多数情况下，1w个连接里面同一时刻只有少量的连接有数据可读，因此，很多个while死循环都白白浪费掉了，因为读不出啥数据。

而在NIO模型中，他把这么多while死循环变成一个死循环，这个死循环由一个线程控制，那么他又是如何做到一个线程，一个while死循环就能监测1w个连接是否有数据可读的呢？
这就是NIO模型中selector的作用，一条连接来了之后，现在不创建一个while死循环去监听是否有数据可读了，而是直接把这条连接注册到**selector**上，然后，通过检查这个selector，就可以批量监测出有数据可读的连接，进而读取数据

实际开发过程中，我们会开多个线程，每个线程都管理着一批连接，相对于IO模型中一个线程管理一条连接，消耗的线程资源大幅减少

#### 线程切换效率低下

由于NIO模型中线程数量大大降低，线程切换效率因此也大幅度提高

#### IO读写以字节为单位

NIO解决这个问题的方式是数据读写不再以字节为单位，而是以**字节块**为单位。IO模型中，每次都是从操作系统底层一个字节一个字节地读取数据，而NIO维护一个缓冲区，每次可以从这个缓冲区里面读取一块的数据

## 原生JDK的NIO写法

1. NIO模型中通常会有两个线程，每个线程绑定一个轮询器selector，在我们这个例子中`serverSelector`负责轮询是否有新的连接，`clientSelector`负责轮询连接是否有数据可读
2. 服务端监测到新的连接之后，不再创建一个新的线程，而是直接将新连接绑定到`clientSelector`上，这样就不用IO模型中1w个while循环在死等，参见(1)
3. `clientSelector`被一个while死循环包裹着，如果在某一时刻有多条连接有数据可读，那么通过 `clientSelector.select(1)`方法可以轮询出来，进而批量处理，参见(2)
4. 数据的读写以内存块为单位，参见(3)

```java
/**
 * @author 闪电侠
 */
public class NIOServer {
    public static void main(String[] args) throws IOException {
        Selector serverSelector = Selector.open();
        Selector clientSelector = Selector.open();
 
        new Thread(() -> {
            try {
                // 对应IO编程中服务端启动
                ServerSocketChannel listenerChannel = ServerSocketChannel.open();
                listenerChannel.socket().bind(new InetSocketAddress(8000));
                listenerChannel.configureBlocking(false); // 监听器设置为非阻塞
                listenerChannel.register(serverSelector, SelectionKey.OP_ACCEPT);
 
                while (true) {
                    // 监测是否有新的连接，这里的1指的是阻塞的时间为1ms
                    if (serverSelector.select(1) > 0) {
                        Set<SelectionKey> set = serverSelector.selectedKeys();
                        Iterator<SelectionKey> keyIterator = set.iterator();
                        while (keyIterator.hasNext()) {
                            SelectionKey key = keyIterator.next();
                            if (key.isAcceptable()) { // 接收到新连接
                                try {
                                    // (1) 每来一个新连接，不需要创建一个线程，而是直接注册到clientSelector
                                    SocketChannel clientChannel = ((ServerSocketChannel) key.channel()).accept();
                                    clientChannel.configureBlocking(false);
                                    // clientChannel 注册到 clientSelector
                                    clientChannel.register(clientSelector, SelectionKey.OP_READ);
                                } finally {
                                    keyIterator.remove(); // 释放连接迭代器
                                }
                            }
                        }
                    }
                }
            } catch (IOException ignored) {
            }
        }).start();
 
        new Thread(() -> {
            try {
                while (true) {
                    // (2) 批量轮询是否有哪些连接有数据可读，这里的1指的是阻塞的时间为1ms
                    if (clientSelector.select(1) > 0) {
                        // 获取set结构的连接集合
                        Set<SelectionKey> set = clientSelector.selectedKeys();
                        Iterator<SelectionKey> keyIterator = set.iterator();
 						// 遍历所有连接，查询有可读数据的连接
                        while (keyIterator.hasNext()) {
                            SelectionKey key = keyIterator.next();
                            if (key.isReadable()) { // 该连接有可读数据
                                try {
                                    SocketChannel clientChannel = (SocketChannel) key.channel();
                                    // 给缓存块分配内存地址，用于存放批量连接
                                    ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
                                    // (3) 读取数据，以块为单位从clientChannel管道中批量读取数据当到Bytebuffer
                                    clientChannel.read(byteBuffer);
                                    byteBuffer.flip(); // 转换为读模式
                                    System.out.println(Charset.defaultCharset().newDecoder().decode(byteBuffer)
                                            .toString());
                                } finally {
                                    keyIterator.remove();
                                    key.interestOps(SelectionKey.OP_READ);
                                }
                            }
                        }
                    }
                }
            } catch (IOException ignored) {
            }
        }).start();
    }
}
```

#### NIO实现群聊系统

##### server端

```java
public class GroupChatClient {

    private final String HOST = "127.0.0.1";
    private final int PORT = 6668;
    private Selector selector;
    private SocketChannel socketChannel;
    private String username;

    public GroupChatClient() throws IOException {
        selector = Selector.open();
        socketChannel = socketChannel.open(new InetSocketAddress("127.0.0.1", PORT));
        socketChannel.configureBlocking(false);
        socketChannel.register(selector, SelectionKey.OP_READ);
        username = socketChannel.getLocalAddress().toString().substring(1);
        System.out.println(username + "is ok...");
    }

    // 客户端发送信息
    public void sendInfo(String info) {
        info = username + "说：" + info;
        try {
            // 将缓冲区中的数据写到socketChannel通道中
            socketChannel.write(ByteBuffer.wrap(info.getBytes()));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    // 读取数据
    public void readInfo() {
        try {
            // 可用通道数量，即与selector绑定的通道
            int readChannels = selector.select();
            if(readChannels > 0) { // 有可以用的通道
                Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
                while (iterator.hasNext()) {
                    SelectionKey key = iterator.next();
                    if (key.isReadable()) {
                        SocketChannel sc = (SocketChannel) key.channel();
                        ByteBuffer buffer = ByteBuffer.allocate(1024);
                        // 从通道读取数据到buffer
                        sc.read(buffer);
                        String msg = new String(buffer.array());
                        System.out.println(msg.trim()); // 去掉msg头尾空格
                    }
                }
                iterator.remove(); // 防止重复读取同一个通道的数据
            } else {
//                System.out.println("没有可以用的通道。。");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws IOException {
        GroupChatClient chatClient = new GroupChatClient();
        new Thread() {
            @Override
            public void run() {
                while(true) {
                    chatClient.readInfo();
                    try {
                        Thread.currentThread().sleep(3000);
                    } catch (Exception e){
                        e.printStackTrace();
                    }
                }
            }
        }.start();
        // 发送数据给服务端
        Scanner scanner = new Scanner(System.in);
        while(scanner.hasNextLine()) {
            String s = scanner.nextLine();
            chatClient.sendInfo(s);
        }
    }
}
```

##### client端

```java
public class GroupChatServer {
    private static Selector selector;
    private static ServerSocketChannel serverSocketChannel;
    private static final int PORT = 6668;

    // 构造器初始化
    public GroupChatServer() {
        try {
            selector = Selector.open();
            serverSocketChannel = ServerSocketChannel.open();
            serverSocketChannel.bind(new InetSocketAddress(PORT));
            serverSocketChannel.configureBlocking(false);
            // 设置该服务端socket通道为可接收事件
            serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // 服务器启动并监听6667端口
    public void listen() {
        try {
            while(true) {
                // 监听连接通道，返回有事件触发的通道的个数
                // 阻塞操作，直到至少获取到一个带有事件的连接
                int count = selector.select(2000);
                // 有可读数据的连接请求
                if(count > 0) {
                    // 获取带事件的连接的迭代器
                    Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
                    // 遍历所有带有可接受时间的连接
                    while(iterator.hasNext()) {
                        // 接收包含可读数据的连接
                        SelectionKey selectionKey = iterator.next();
                        // 监听到有可连接事件的通道
                        if(selectionKey.isAcceptable()) { // 上面构造函数时已经设置了OP_ACCEPT
                            // 根据连接通道获取到传输数据的通道，相当于BIO的socket
                            SocketChannel socketChannel = serverSocketChannel.accept();
                            socketChannel.configureBlocking(false);
                            // 注册时间
                            // 将传输数据的通道SocketChannel注册到selecgtor选择器中
                            // 通道注册，使得连接通道与传输数据通道关联，即ServerSocketChannel和SocketChannel关联
                            // 选择OP_READ事件，SocketChannel相当于转换为读模式
                            socketChannel.register(selector, SelectionKey.OP_READ); // 告诉selector绑定的读事件
                            System.out.println(socketChannel.getRemoteAddress() + "上线了");
                        }
                        // 该通道触发可读事件
                        if(selectionKey.isReadable()) {
                            readData(selectionKey);
                        }
                    }
                    iterator.remove();
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {

        }
    }

    // 负责读取客户端消息，并转发到其他客户端
    private void readData(SelectionKey key) {
        SocketChannel channel = null;
        try {
            channel = (SocketChannel) key.channel();
            // 创建缓冲
            ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
            // 读取该socketChannel通道里的数据到缓冲区里
            int count = channel.read(byteBuffer);
            if(count > 0) {
                String msg = new String(byteBuffer.array());
                System.out.println("from 客户端：" + msg);

                // 向其他客户端转发消息
                sendInfoToOtherClients(msg, channel);
            }
        } catch (Exception e) {
            // 可能在读取该客户端数据时，该客户端关闭或宕机或网络抖动或离线
            try {
                System.out.println(channel.getRemoteAddress() + "离线了。。。");
                // 取消该客户端的注册
                key.cancel();
                channel.close(); // 该客户端的SocketChannel关闭

            } catch (IOException e1) {
                e1.printStackTrace();
            }
        } finally {

        }
    }

    // 转发消息给其他客户
    private void sendInfoToOtherClients(String msg, SocketChannel selfChannel) throws IOException {
        System.out.println("服务器转发消息中");
        for(SelectionKey key : selector.keys()) {
            Channel targetChannel = key.channel();
            // 排除自己，不需要转发自己的消息给自己
            if(targetChannel instanceof SocketChannel && targetChannel != selfChannel) {
                SocketChannel dest = (SocketChannel) targetChannel;
                ByteBuffer buffer = ByteBuffer.wrap(msg.getBytes());
                dest.write(buffer); // 将缓冲区数据写入到dest通道
            }
        }
    }

    public static void main(String[] args) {
        GroupChatServer groupChatServer = new GroupChatServer();
        groupChatServer.listen();
    }
}
```



### socket相关Linux命令

执行并追踪执行中的Java线程：**stace -ff -o out java [要追踪的 java 文件]**

查看当前进程状态以及pid：**netstat -natp**

打印某个文件尾10行： **tail -f xxx**

实现tcp三次握手连接：**nc localhost 8090 **

vim显示行数：**:set nu**

vim跳到指定第5行：**ngg/5G**  或 **:n**

查看有关系统调用的指令：**man 2 [要查看的系统调用命令, 如socket]**



### NIO零拷贝

##### 1、传统IO需要进行4次拷贝，2次DMA（直接内存拷贝，不使用cpu）和2次cpu拷贝（消耗性能多），3次上下文切换，性能较差

![image-20201124154030368](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201124154030368.png)

##### 2、而NIO出现了mmap，对传统IO进行优化。使得IO只需要2次DMA拷贝，1次CPU拷贝，三次上下文切换

###### mmap：通过内存映射，将文件映射到内核缓冲区，同时用户空间可以共享内核空间的数据。进行网络传输时，就可以减少内核空间到用户空间的拷贝次数

![image-20201124154001819](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201124154001819.png)

##### 3、进一步优化：sendFile函数

Linux2.1提供的sendFile函数，其原理：数据不经过用户态，而是直接从内核缓冲区进入到Socket Buffer，减少了一次上下文切换

![image-20201124154358575](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201124154358575.png)

##### 4、Linux2.4优化，即实现了零拷贝（不进行CPU拷贝就是零拷贝）：

避免了从内核缓冲区拷贝到Socketbuffer，而是直接拷贝到协议栈protocal engine，从而减少了数据拷贝（CPU拷贝），因此为：2次DMA copy，2次上下文切换。

（注意：零拷贝中间还是有一次cpu拷贝的，即copy desc，但是拷贝的信息很少，如kernel buffer的length、offset等描述信息，消耗低，可忽略）

零拷贝是从操作系统角度来说的，即内核缓冲区kernel buffer之间没有重复的数据



![image-20201124154805494](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20201124154805494.png)

##### 零拷贝的使用

```
FileChannel fileChannel = FileInputStream("fileName").getChannel;
long transferCount = fileChannel.transferTO(0, fileChannel.size(), socketChannel);
```



# NIO的BUG

### epoll空轮询bug

epoll空轮询bug体现在Selector空轮询,  若Selector的轮询结果为空，也没有wakeup或新消息处理，则发生空轮询，CPU使用率100%

原因：

- 正常情况下，`selector.select()`操作是阻塞的，只有被监听的fd有读写操作时，才被唤醒
- 但是，在这个bug中，没有任何fd有读写请求，但是`select()`操作依旧被唤醒
- 很显然，这种情况下，`selectedKeys()`返回的是个空数组
- 然后按照逻辑执行到`while(true)`处，循环执行，导致死循环。

##### **Netty的解决办法**

- 1) 根据该BUG的特征，首先侦测该BUG是否发生

    侦测方法：对Selector的select操作周期进行统计，每完成一次空的select操作进行一次计数；

  ​         若在某个周期内连续发生N次空轮询，则触发了epoll死循环bug， netty默认是512次

- 2) 将问题Selector上注册的Channel转移到新建的Selector上；老的问题Selector关闭，使用新建的Selector替换。

在netty中使用 rebuildSelector() 方法重建Selector，判断是否是其他线程发起的重建请求，若不是则将原SocketChannel从旧的Selector上去除注册，重新注册到新的Selector上，并将原来的Selector关闭。

```java
    public void rebuildSelector() {
        if (!inEventLoop()) {
            execute(new Runnable() {
                @Override
                public void run() {
                    rebuildSelector0();
                }
            });
            return;
        }
        rebuildSelector0();
    }
  private void rebuildSelector0() {
        final Selector oldSelector = selector;
        final SelectorTuple newSelectorTuple;
        if (oldSelector == null) {
            return;
        }
        try {
            newSelectorTuple = openSelector();
        } catch (Exception e) {
            logger.warn("Failed to create a new Selector.", e);
            return;
        }
        // Register all channels to the new Selector.
        int nChannels = 0;
        for (SelectionKey key: oldSelector.keys()) {
            Object a = key.attachment();
            try {
                if (!key.isValid() || key.channel().keyFor(newSelectorTuple.unwrappedSelector) != null) {
                    continue;
                }
                int interestOps = key.interestOps();
                key.cancel();
                SelectionKey newKey = key.channel().register(newSelectorTuple.unwrappedSelector, interestOps, a);
                if (a instanceof AbstractNioChannel) {
                    // Update SelectionKey
                    ((AbstractNioChannel) a).selectionKey = newKey;
                }
                nChannels ++;
            } catch (Exception e) {
                logger.warn("Failed to re-register a Channel to the new Selector.", e);
                if (a instanceof AbstractNioChannel) {
                    AbstractNioChannel ch = (AbstractNioChannel) a;
                    ch.unsafe().close(ch.unsafe().voidPromise());
                } else {
                    @SuppressWarnings("unchecked")
                    NioTask<SelectableChannel> task = (NioTask<SelectableChannel>) a;
                    invokeChannelUnregistered(task, key, e);
                }
            }
        }
        selector = newSelectorTuple.selector;
        unwrappedSelector = newSelectorTuple.unwrappedSelector;
        try {
            // time to close the old selector as everything else is registered to the new one
            oldSelector.close();
        } catch (Throwable t) {
            if (logger.isWarnEnabled()) {
                logger.warn("Failed to close the old Selector.", t);
            }
        }
        if (logger.isInfoEnabled()) {
            logger.info("Migrated " + nChannels + " channel(s) to the new Selector.");
        }
    }
```

netty 会在每次进行 selector.select(timeoutMillis) 之前记录一下开始时间currentTimeNanos，在select之后记录一下结束时间，判断select操作是否至少持续了timeoutMillis秒（这里将time - TimeUnit.MILLISECONDS.toNanos(timeoutMillis) >= currentTimeNanos改成time - currentTimeNanos >= TimeUnit.MILLISECONDS.toNanos(timeoutMillis)或许更好理解一些）,
如果持续的时间大于等于timeoutMillis，说明就是一次有效的轮询，重置selectCnt标志，否则，表明该阻塞方法并没有阻塞这么长时间，可能触发了jdk的空轮询bug，当空轮询的次数超过一个阀值的时候，默认是512，就开始重建selector

















