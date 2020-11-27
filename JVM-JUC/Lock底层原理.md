[TOC]

# Java锁底层原理

当多个线程需要访问某个公共资源的时候，我们知道需要通过加锁来保证资源的访问不会出问题。java提供了**两种方式来加锁**，

一种是关键字：synchronized，一种是concurrent包下的lock锁。

![image-20200916154451400](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916154451400.png)

# synchronized

1. synchronized的作用：保证了**原子性、可见性、有序性**。

为什么synchronized无法禁止指令重排，却能保证有序性？
为了进一步提升计算机各方面能力，在硬件层面做了很多优化，如处理器优化和指令重排等，但是这些技术的引入就会导致有序性问题。
我们也知道，最好的解决有序性问题的办法，就是禁止处理器优化和指令重排，就像volatile中使用内存屏障一样。
虽然很多硬件都会为了优化做一些重排，但是在Java中，不管怎么排序，都不能影响单线程程序的执行结果。这就是as-if-serial语义，所有硬件优化的前提都是必须遵守as-if-serial语义。
synchronized，他是Java提供的锁，可以通过他对Java中的对象加锁，并且他是一种排他的、可重入的锁。
所以，当某个线程执行到一段被synchronized修饰的代码之前，会先进行加锁，执行完之后再进行解锁。在加锁之后，解锁之前，其他线程是无法再次获得锁的，只有这条加锁线程可以重复获得该锁。
synchronized通过**排他锁**的方式就保证了同一时间内，被synchronized修饰的代码是单线程执行的。所以呢，这就满足了as-if-serial语义的一个关键前提，那就是单线程，因为有as-if-serial语义保证，单线程的有序性就天然存在了。

1. Synchronized可以把修饰的任何一**个非null对象**作为"锁"，synchronized的用法有以下三种:

- 修饰实例方法：锁住的是对象实例this，属于对象锁
- 修饰静态方法：锁住的是对象class实例，属于类锁
- 修饰代码块：锁住的是括号里面的对象实例，属于对象锁

注意，synchronized内置锁是一种对象锁（锁的是对象而非引用变量），作用粒度是对象，可以用来实现对临界资源的同步互斥访问，是可重入的。其可重入最大的作用是避免死锁，如：**子类同步方法调用了父类同步方法，如没有可重入的特性，则会发生死锁；**

## synchronized的底层同步原理

synchronized是在软件层面依赖于JVM，而j.u.c下的lock是依赖于硬件层面。

Synchronized底层原理分为两种：对象锁和方法锁，即修饰对象和修饰方法

### 1、Synchronized修饰对象

如果synchronized修饰的是对象，那么它是依赖于**monitor对象—监视器锁**来实现锁的机制的。

```java
public class SynchronizeTest {
    public void method(){
        synchronized (this) {  // 锁的是调用该method方法的实例对象this
            for (int i = 0; i < 5; i++) {
                System.out.println(Thread.currentThread().getName() + "+++++++" + i);
            }
        }
    }
}
```

先编译上述文件为class文件：javac -encoding utf-8 类名.java

反编译class类文件:  javap -c 类名.class

#### 编译结果解析：

- **monitorenter**：每个对象都是一个监视器锁（monitor）。当monitor被其他线程占用时就会处于锁定状态，线程执行**monitorenter**指令时尝试获取monitor的所有权，过程如下：

1. 如果monitor的进入数为0，则该线程将进入monitor，然后将进入monitor的进入数设置为1，该线程即为monitor的所有者；
2. 如果线程已经占有该monitor，只是重新进入，则进入monitor的进入数加1；
3. 如果其他线程已经占用了monitor，则该线程进入阻塞状态，直到monitor的进入数为0，再重新尝试获取monitor的所有权；

- **monitorexit**：执行monitorexit的线程必须是objecter所对应的monitor的所有者。指令执行时，monitor的进入数减1，如果减1后进入数为0，那线程退出monitor，不再是这个monitor的所有者。其他被这个monitor阻塞的线程可以尝试去获取这个 monitor 的所有权。

  monitorexit指令如果出现了两次，第1次为同步正常退出释放锁；第2次为发生异步退出释放锁；

##### 通过上面两段描述，我们应该能很清楚的看出Synchronized的实现原理：

##### Synchronized的语义底层是通过一个monitor的对象来完成，其实wait/notify等方法也依赖于monitor对象，
这就是为什么只有在同步的块或者方法中才能调用wait/notify等方法，否则会抛出 **java.lang.IllegalMonitorStateException** 的异常的原因。

### 2、Synchronized修饰方法

 如果synchronized修饰的是方法，那么则是通过ACC_SYNCHRONIZED 标示符来进行加锁的。

```
public class SynchronizeTest {
   public synchronized void method() {
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName() + "+++++++" + i);
        }
    }
}
```

反编译如下：

![image-20200916154526070](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916154526070.png)

当方法调用时，调用指令将会检查方法的 **ACC_SYNCHRONIZED** 标志符是否被设置，如果设置了，线程将会先获取monitor，获取成功之后才能执行方法体，方法执行完之后再释放monitor。在方法执行期间，其他任何线程都无法再获得同一个monitor对象。

两种同步方式本质上没有区别，只是方法的同步是一种隐式的方式来实现，无需通过字节码来完成。两个指令的执行是JVM通过调用操作系统的互斥原语mutex来实现，被阻塞的线程会被挂起、等待重新调度，但会导致 “用户态和内核态” 两个态之间来回切换，对性能有较大影响。

### Synchronized的锁对象

无论是实例对象（包括实例this和方法）还是类对象。在JVM中，每个对象都是由三部分组成的：**对象头、实例数据、数据填充**。synchronized的锁的信息都是存储在对象头里。对象组成结构如下：

![image-20200916154512514](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916154512514.png)

- 实例数据：存放类的属性数据信息，包括父类的属性信息；
- 对齐填充：由于虚拟机要求，对象起始地址必须是8字节的整数倍。填充数据不是必须存在的，仅仅是为了字节对齐；
- **对象头**：Java对象头一般占有2个机器码（在32位虚拟机中，1个机器码等于4字节，也就是32bit，在64位虚拟机中，1个机器码是8个字节，也就是64bit），但是，如果对象是数组类型，则需要3个机器码，因为JVM虚拟机可以通过Java对象的元数据信息确定Java对象的大小，但是无法从数组的元数据来确认数组的大小，所以用一块来记录数组长度。

Synchronized用的锁就是存在Java对象头里的，那么什么是Java对象头呢？Hotspot虚拟机的对象头主要包括两部分数据：Mark Word（标记字段）、Class Pointer（类型指针）。其中 Class Pointer是对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例，Mark Word用于存储对象自身的运行时数据，它是实现轻量级锁和偏向锁的关键。 Java对象头具体结构描述如下：

![image-20200916154559001](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916154559001.png)

其中Mark Word在默认情况下存储着对象的哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程 ID、偏向时间戳等，以下是32位JVM的Mark Word默认存储结构：

![image-20200916154630009](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916154630009.png)

对象头信息是与对象自身定义的数据无关的额外存储成本，但是考虑到虚拟机的空间效率，Mark Word被设计成一个非固定的数据结构以便在极小的空间内存存储尽量多的数据，**它会根据对象的状态复用自己的存储空间**，也就是说，Mark Word会随着程序的运行发生变化，可能变化为存储以下4种数据：

![image-20200916154635454](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916154635454.png)

synchronized属于对象锁，而任何一个对象都有一个Monitor与之关联，当且一个Monitor被持有后，它将处于锁定状态。在Java虚拟机（HotSpot）中，Monitor是由ObjectMonitor实现的，其主要数据结构如下（位于HotSpot虚拟机源码ObjectMonitor.hpp文件，C++实现的）：

```
ObjectMonitor() {
    _header       = NULL;
    _count        = 0; //记录个数
    _waiters      = 0,
    _recursions   = 0;
    _object       = NULL;
    _owner        = NULL;
    _WaitSet      = NULL; //处于wait状态的线程，会被加入到_WaitSet
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ; //处于等待锁block状态的线程，会被加入到该列表
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
```

结构中有几个重要的字段，_count、_owner、_EntryList、_WaitSet。

- count用来记录线程进入加锁代码的次数。
- owner记录当前持有锁的线程,即持有ObjectMonitor对象的线程。
- EntryList是想要持有锁的线程的集合。
- WaitSet 是加锁对象调用wait（）方法后，等待被唤醒的线程的集合

#### 当多个线程访问同步代码块时：

**1>** 首先线程会进入EntryList集合，然后当线程拿到Monitor对象时，进入owner区域，并把Monitor的owner设置为当前线程，_owner指向持有ObjectMonitor对象的线程，并把计数器count加1.

**2>** 若线程调用wait方法，将释放当前持有的monitor对象，同时owner变量恢复为null，count自减1，同时该线程进入WaitSet集合中等待被唤醒；

**3>** 当前线程执行完毕，也将释放monitor（锁）并复位count的值，以便其他线程进入获取monitor(锁)；
过程如下图所示：![image-20200916154643603](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916154643603.png)

**Synchronized与等待唤醒：**

- 等待唤醒是指调用对象的wait、notify、notifyAll方法。调用这三个方法时，对象必须被synchronized修饰，因为这三个方法在执行时，必须获得当前对象的监视器monitor对象。
- 另外，与sleep方法不同的是wait方法调用完成后，线程将被暂停，但wait方法将会释放当前持有的监视器锁(monitor)，直到有线程调用notify/notifyAll方法后方能继续执行。而sleep方法只让线程休眠并不释放锁。notify/notifyAll方法调用后，并不会马上释放监视器锁，而是在相应的synchronized代码块或synchronized方法执行结束后才自动释放锁。

**Synchronized的可重入与中断：**

- 可重入：当多个线程请求同一个临界资源，执行到同一个临界区时会产生互斥，未获得资源的线程会阻塞。而当一个已获得临界资源的线程再次请求此资源时并不会发生阻塞，仍能获取到资源、进入临界区，这就是重入。Synchronized是可重入的。

- 中断：与中断相关的有三个方法：

  ```java
  /**
   * Interrupt设置一个线程为中断状态
   * Interrupt操作的线程处于sleep,wait,join 阻塞等状态的时候，清除“中断”状态，抛出一个InterruptedException
   * Interrupt操作的线程在可中断通道上因调用某个阻塞的 I/O 操作(serverSocketChannel. accept()、socketChannel.connect、socketChannel.open、 
   * socketChannel.read、socketChannel.write、fileChannel.read、fileChannel.write)，会抛出一个ClosedByInterruptException
   **/
  public void interrupt();
  /**
   * 判断线程是否处于“中断”状态，然后将“中断”状态清除
   **/
  public static boolean interrupted();
  /**
   * 判断线程是否处于“中断”状态
   **/
  public boolean isInterrupted();
  ```

  在实际使用中，当线程正处于调用sleep、wait、join方法后，调用interrupt会清除线程中断状态，并抛出异常。而当线程已进入临界区、正在执行，则需要isInterrupted()或interrupted()与interrupt()配合使用中断执行中的线程。

  **Sychronized修饰的方法、代码块被多个线程请求时，调用中断，正在执行的线程响应中断，正在阻塞的线程、执行中的线程都会标记中断状态，但阻塞的线程不会立刻处理中断，而是在进入临界区后再响应。**

### 3、synchronized锁的1.6升级优化

偏向锁、轻量级锁、重量级锁、锁消除、锁粗化。

1>锁的升级只能从低到高，不能从高到低。
锁的升级过程如下：
![image-20200916154656494](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916154656494.png)

2>锁标志位变化:

![image-20200916154659727](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916154659727.png)

- 无锁状态：锁的对象头是无锁状态，有1bit专门记录是否为偏向锁，0代表无锁，1代表偏向锁。有2bit位记录锁标志位，

- 偏向锁：线程开始占有锁对象，偏向锁的标志位变为1，23bit位的hashcode存放线程A的线程ID，2bit位存放epoch（共25bit位），如果在多线程并发的环境下（即线程A尚未执行完同步代码块，线程B发起了申请锁的申请），如果线程B成功拿到锁，那么此时还是属于偏向锁状态。

- 轻量级锁：如果此时线程获取锁失败,则转化为轻量级锁。首先会在线程A和线程B都开辟一块LockRecord空间，然后把锁对象复制一份到自己的LockRecord空间下，并且开辟一块owner空间留作执行锁使用，并且锁对象的前30bit位合并，等待线程A和线程B来修改指向自己的线程，假如线程A修改成功，则锁对象头的前30bit位会存线程A的LockRecord的内存地址，并且线程A的owner也会存一份锁对象的内存地址，形成一个双向指向的形式。而线程B修改失败，则进入一个自旋状态，就是持续来修改锁对象。

- 重量级锁：如果线程B自旋一定次数后，还没有拿到锁，这个时候锁就会升级为重量级锁，这时我们的线程B会由用户态切换到内核态，申请一个互斥量matux，并且将锁对象的前30bit指向我们的互斥量地址，并且进入睡眠状态，然后我们的线程A继续运行直到完成时，当线程A想要释放锁资源时，发现原来锁的前30bit位并不是指向自己了，这时线程A释放锁，并且去唤醒那些处于睡眠状态的线程，锁升级到重量级锁。


3>锁消除：因为Synchronized锁的是对象，如果每一个线程都锁一个新的对象，那么这个时候就不需要进行上锁了，就是所谓的锁消除，锁消除的底层实现原理是JVM的逃逸分析原理。如以下代码所示：

```
 	synchronized (new Object()){
        System.out.println("开始处理逻辑");
    }
```

4>锁粗化：
把很多次锁的请求合并成一个请求，以降低短时间内大量锁请求、同步、释放带来的性能损耗。

```
StringBuffer sb = new StringBuffer();

public void lockCoarseningMethod() {
    synchronized (Test.class) {
        sb.append("1");
    }

    synchronized (Test.class) {
        sb.append("2");
    }
    synchronized (Test.class) {
        sb.append("3");
    }
    synchronized (Test.class) {
        sb.append("4");
    }
}
1234567891011121314151617
```

锁粗化后：

```
StringBuffer sb = new StringBuffer();

public void lockCoarseningMethod() {
    synchronized (Test.class) {
        sb.append("1");
        sb.append("2");
        sb.append("3");
        sb.append("4");
    }
}
```

## ReentrantLock

 可以实现公平锁和非公平锁（ 当有线程竞争锁时，当前线程会首先尝试获得锁而不是在队列中进行排队等候，这对于那些已经在队列中排队的线程来说显得不公平，这也是非公平锁的由来），ReentrantLock默认情况下为非公平锁。

* ReentrantLock构造方法

```java
 /**
     * Creates an instance of {@code ReentrantLock}.
     * This is equivalent to using {@code ReentrantLock(false)}.
     */
    public ReentrantLock() {
        sync = new NonfairSync();
    }
```

* ReentrantLock的非公平锁类NonfairSync ，里面有 lock方法

  ```java
  /**
       * Sync object for non-fair locks
       */
      static final class NonfairSync extends Sync {
          private static final long serialVersionUID = 7316153563782823691L;
          /**
           * Performs lock.  Try immediate barge, backing up to normal
           * acquire on failure.
           */
          final void lock() {
              if (compareAndSetState(0, 1))
                  setExclusiveOwnerThread(Thread.currentThread());
              else
                  acquire(1);
          }
  
          protected final boolean tryAcquire(int acquires) {
              return nonfairTryAcquire(acquires);
          }
      }
  ```
  
* 而非公平锁类NonfairSync 继承了抽象类Sync，Sync又继承抽象类AbstractQueuedSynchronizer（简称AQS）

  ```
  abstract static class Sync extends AbstractQueuedSynchronizer {...}
  ```

* AQS又继承了AbstractOwnableSynchronizer（简称AOS），AOS主要是保存获取当前锁的线程对象，继承关系：

* ![image-20200916154709695](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916154709695.png)

* FairSync 与 NonfairSync的区别在于，是不是保证获取锁的公平性，因为默认是NonfairSync（**非公平性**）

## AQS

AbstractQueuedSynchronizer（简称AQS）是除了java自带的synchronized关键字之外的锁机制。

#### **AQS的核心思想**

如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并将共享资源设置为锁定状态，如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用CLH队列锁实现的，即：将暂时获取不到锁的线程加入到等待（阻塞）队列中。
CLH（Craig，Landin，and Hagersten）队列是一个虚拟的双向队列，虚拟的双向队列即不存在队列实例，仅存在节点之间的关联关系。

AQS是将每一条请求共享资源的被阻塞的等待线程封装成一个CLH锁队列的一个节点，来实现锁的分配

简单来说，AQS就是基于CLH队列，用volatile修饰共享变量state状态符，线程通过CAS去改变状态符，成功则获取锁成功，失败进入CLH队列，等待被唤醒

注意：AQS是通过自旋锁实现，即：在等待唤醒过程中，经常会使用自旋（类似while(CAS)）的方式不停尝试获取锁，直到被其他线程获取成功。

实现了AQS的锁有：自旋锁、互斥锁、读写可重入锁ReentrantReadWriteLock、条件产量、信号量、栅栏都是AQS的衍生物

AQS实现的具体方法：

![image-20200916154714681](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916154714681.png)

如图，AQS维护了一个volatile int state和一个FIFO线程等待队列，多线程争用资源被阻塞的时候就会进入这个队列。**state**就是共享资源，其访问方式有如下三种：
getState();

setState();

compareAndSetState();

AQS 定义了两种资源共享方式：
1.**Exclusive**：独占，只有一个线程能执行，如ReentrantLock
2.**Share**：共享，多个线程可以同时执行，如Semaphore、CountDownLatch、ReadWriteLock，CyclicBarrier

不同的自定义的同步器争用共享资源的方式也不同。

### AQS底层使用了模板方法模式

同步器的设计是基于模板方法模式的，如果需要自定义同步器一般的方式是这样（模板方法模式很经典的一个应用）：

1. 使用者继承AbstractQueuedSynchronizer并重写指定的方法。（这些重写方法很简单，无非是对于共享资源state的获取和释放）
2. 将AQS组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。
   这和我们以往通过实现接口的方式有很大区别，这是模板方法模式很经典的一个运用。

自定义同步器在实现的时候只需要实现共享资源state的获取和释放方式即可，至于具体线程等待队列的维护，AQS已经在顶层实现好了。自定义同步器实现的时候主要实现下面几种方法：

* isHeldExclusively()：该线程是否正在独占资源。只有用到condition才需要去实现它。
  tryAcquire(int)：独占方式。尝试获取资源，成功则返回true，失败则返回false。
  tryRelease(int)：独占方式。尝试释放资源，成功则返回true，失败则返回false。
  tryAcquireShared(int)：共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
  tryReleaseShared(int)：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false。

ReentrantLock为例，（可重入独占式锁）：state初始化为0，表示未锁定状态，A线程lock()时，会调用tryAcquire()独占锁并将state+1.之后其他线程再想tryAcquire的时候就会失败，直到A线程unlock（）到state=0为止，其他线程才有机会获取该锁。A释放锁之前，自己也是可以重复获取此锁（state累加），这就是可重入的概念。
注意：获取多少次锁就要释放多少次锁，保证state是能回到零态的。

以CountDownLatch为例，任务分N个子线程去执行，state就初始化 为N，N个线程并行执行，每个线程执行完之后countDown（）一次，state就会CAS减一。当N子线程全部执行完毕，state=0，会unpark()主调用线程，主调用线程就会从await()函数返回，继续之后的动作。

一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现tryAcquire-tryRelease、tryAcquireShared-tryReleaseShared中的一种即可。但AQS也支持自定义同步器同时**实现独占和共享两种方式，如ReentrantReadWriteLock。**
　在acquire() acquireShared()两种方式下，线程在等待队列中都是忽略中断的，**acquireInterruptibly()/acquireSharedInterruptibly()是支持响应中断**的。



#### AQS底层数据结构是双向链表，锁的存储结构就两个东西 : 双向链表 + "int类型状态"

简单来说，ReentrantLock的实现是一种**自旋锁**，通过循环调用CAS操作来实现加锁。它的性能比较好也是因为**避免了使线程从用户态进入内核态的阻塞状态。**想尽办法避免线程进入内核的阻塞状态是我们去分析和理解锁设计的关键。

需要注意的是，他们的变量都被**transient**和**volatile**修饰。

![image-20200916154719323](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916154719323.png)

## J.U.C 同步队列（CLH）

一种FIFO双向队列，队列中每个节点等待前驱结点释放共享状态（锁）被唤醒就可以了

AQS依赖它来完成同步状态的管理，当前线程如果获取同步状态失败时，AQS则会将当前线程已经等待状态等信息构造成一个节点（Node）并将其加入到CLH同步队列，同时会阻塞当前线程，当同步状态释放时，会把首节点唤醒（公平锁），使其再次尝试获取同步状态。

#### Node节点

这里是基于CAS（保证线程的安全）来设置尾节点的。

![image-20200916154722663](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916154722663.png)

```java
static final class Node {
        // 节点分为两种模式： 共享式和独占式
        /** 共享式 */
        static final Node SHARED = new Node();
        /** 独占式 */
        static final Node EXCLUSIVE = null;

        /** 等待线程超时或者被中断、需要从同步队列中取消等待（也就是放弃资源的竞争），此状态不会在改变 */
        static final int CANCELLED =  1;
        /** 后继节点会处于等待状态，当前节点线程如果释放同步状态或者被取消则会通知后继节点线程，使后继节点线程的得以运行 */
        static final int SIGNAL    = -1;
        /** 节点在等待队列中，线程在等待在Condition 上，其他线程对Condition调用singnal()方法后，该节点加入到同步队列中。 */
        static final int CONDITION = -2;
        /**
         * 表示下一次共享式获取同步状态的时会被无条件的传播下去。
         */
        static final int PROPAGATE = -3;

        /**等待状态*/
        volatile int waitStatus;

        /**前驱节点 */
        volatile Node prev;

        /**后继节点*/
        volatile Node next;

        /**获取同步状态的线程 */
        volatile Thread thread;

        /**链接下一个等待状态 */
        Node nextWaiter;
        
        // 下面一些方法就不贴了
    }
```

### 入列

如上图了解了同步队列的结构， 我们在分析其入列操作在简单不过。无非就是将tail（使用CAS保证原子操作）指向新节点，新节点的prev指向队列中最后一节点（旧的tail节点），原队列中最后一节点的next节点指向新节点以此来建立联系，来张图帮助大家理解。

![image-20200916154727235](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916154727235.png)

* addWaiter源码：先通过addWaiter(Node node)方法尝试快速将该节点设置尾成尾节点，设置失败走enq(final Node node)方法

  ```java
  private Node addWaiter(Node mode) {
  // 以给定的模式来构建节点， mode有两种模式 
  //  共享式SHARED， 独占式EXCLUSIVE;
    Node node = new Node(Thread.currentThread(), mode);
      // 尝试快速将该节点加入到队列的尾部
      Node pred = tail;
       if (pred != null) {
          node.prev = pred;
              if (compareAndSetTail(pred, node)) {
                  pred.next = node;
                  return node;
              }
          }
          // 如果快速加入失败，则通过 anq方式入列
          enq(node);
          return node;
      }
  ```

* enq：通过“自旋”也就是死循环的方式来保证该节点能顺利的加入到队列尾部，只有加入成功才会退出循环，否则会一直循序直到成功。

  ```
  private Node enq(final Node node) {
  // CAS自旋，直到加入队尾成功        
  for (;;) {
      Node t = tail;
          if (t == null) { // 如果队列为空，则必须先初始化CLH队列，新建一个空节点标识作为Hader节点,并将tail 指向它
              if (compareAndSetHead(new Node()))
                  tail = head;
              } else {// 正常流程，加入队列尾部
                  node.prev = t;
                      if (compareAndSetTail(t, node)) {
                          t.next = node;
                          return t;
                  }
              }
          }
      }
  
  ```

* 上述两个方法都是通过compareAndSetHead(new Node())方法来设置尾节点，以保证节点的添加的原子性（保证节点的添加的线程安全。）

### 出列

同步队列（CLH）遵循FIFO，首节点是获取同步状态的节点，首节点的线程释放同步状态后，将会唤醒它的后继节点（next），而后继节点将会在获取同步状态成功时将自己设置为首节点，这个过程非常简单。如下图

![](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916154732450.png)

同步队列-出列.jpg

设置首节点是通过获取同步状态成功的线程来完成的（获取同步状态是通过CAS来完成），只能有一个线程能够获取到同步状态，因此设置头节点的操作并不需要CAS来保证，只需要将首节点设置为其原首节点的后继节点并断开原首节点的next（等待GC回收）应用即可

### 总结

同步队列就是一个FIFO双向对队列，其每个节点包含获取同步状态失败的线程应用、等待状态、前驱节点、后继节点、节点的属性类型以及名称描述。

其入列操作也就是利用CAS(保证线程安全)来设置尾节点，出列就很简单了直接将head指向新头节点并断开老头节点联系就可以了。

参考：https://www.jianshu.com/p/6fc0601ffe34



## Lock.lock()

* lock是Lock接口的方法，它的抽象方法在ReentrantLock类中的Sync类里，实现方法在 NonfairSync.lock()

  ```
   abstract void lock(); 
  ```

* 公平锁的上锁 FairSync.lock()

  ```
      public void lock() {
          sync.lock(); 
      }
  ```

* 可以看到公平锁的lock() 是通过调用 **NonfairSync.lock()** 实现的

* 这里就是通过**CAS**（乐观锁）去修改state的值(**锁状态值**)。lock的基本操作还是通过**乐观锁**来实现的。

  ```java
          final void lock() {
              if (compareAndSetState(0, 1))  // 比较锁状态值status是否为0，是则修改为1
                  setExclusiveOwnerThread(Thread.currentThread()); // 通过CAS获取到锁了，当前线程设置为专有线程
              else  
                  acquire(1);
          }
  ```

* **获取锁通过CAS**，那么没有获取到锁，**等待获取锁**是如何实现的？我们可以看一下else分支的逻辑，acquire方法：

  - tryAcquire：会尝试再次通过CAS获取一次锁
- addWaiter：将当前线程加入上面锁的双向链表（等待队列）中
  - acquireQueued：通过自旋，判断当前队列节点是否可以获取锁。

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&   
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt(); // 中断线程，但对于正在运行的线程没有作用
}
```

* addWaiter: 将当前线程加入上面锁的双向链表（等待队列）中, 通过CAS确保能够在线程安全的情况下，通过尾插法将当前线程加入到链表的**尾部。**enq是个自旋+上述逻辑

  ```java
      private Node addWaiter(Node mode) {
          Node node = new Node(Thread.currentThread(), mode);
          // Try the fast path of enq; backup to full enq on failure
          Node pred = tail;
          if (pred != null) {
              node.prev = pred;
              if (compareAndSetTail(pred, node)) {
                  pred.next = node;
                  return node;
              }
          }
          enq(node);
          return node;
      }
  ```

* **acquireQueued()  自旋+CAS**尝试获取锁

当前**线程到头部的时候，尝试CAS更新锁状态，如果更新成功表示该等待线程获取成功。从头部移除。**因为等待队列是从尾进从头出

```java
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) { // 自旋
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) { // 还是通过 tryAcquire 的CAS操作尝试获取锁
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

#### **每一个线程都在 自旋+CAS**

![image-20200916154740385](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916154740385.png)

### 获得锁的过程

![image-20200916154743401](C:\Users\qq285\AppData\Roaming\Typora\typora-user-images\image-20200916154743401.png)



## Lock.unlock()

```
public void unlock() {
    sync.release(1);
}
```

#### **NonfairSync.release()** 该方法在AOS中

```java
    public final boolean release(int arg) {
        if (tryRelease(arg)) {  	// 尝试释放锁
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
```

#### **NonfairSync.tryRelease()**

**释放锁就是对AQS中的状态值State进行修改。同时更新下一个链表中的线程等待节点**。

```java
        protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
```



#### **总结**

- lock的存储结构：一个int类型状态值（用于锁的状态变更）+  一个双向链表（用于存储等待中的线程）
- lock获取锁的过程：本质上是通过CAS来获取状态值修改，如果当场没获取到，会将该线程放在线程等待链表中。
- lock释放锁的过程：修改状态值，调整等待链表。

可以看到在整个实现过程中，lock大量使用CAS+自旋。因此根据CAS特性，lock建议使用在低锁冲突的情况下。目前java1.6以后，官方对synchronized做了大量的锁优化（偏向锁、自旋、轻量级锁）。因此在非必要的情况下，建议使用synchronized做同步操作。

____________________________________________________________________________

# **锁实现**

   简单说来，AQS会把所有的请求线程构成一个CLH队列，当一个线程执行完毕（lock.unlock()）时会激活自己的后继节点，但正在执行的线程并不在队列中，而那些等待执行的线程全 部处于**阻塞状态**，经过调查线程的显式阻塞是通过调用LockSupport.park()完成，而LockSupport.park()则调用 sun.misc.Unsafe.park()本地方法，再进一步，HotSpot在Linux中通过调用pthread_mutex_lock函数把 线程交给系统内核进行阻塞。

   与synchronized相同的是，这也是一个虚拟队列，不存在队列实例，仅存在节点之间的前后关系。令人疑惑的是为什么采用CLH队列呢？原生的CLH队列是用于自旋锁，但Doug Lea把其改造为阻塞锁。

   当有线程竞争锁时，该线程会首先尝试获得锁，这对于那些已经在队列中排队的线程来说显得不公平，这也是非公平锁的由来，与synchronized实现类似，这样会极大提高吞吐量。 如果已经存在Running线程，则新的竞争线程会被追加到队尾，具体是采用基于CAS的Lock-Free算法，因为线程并发对Tail调用CAS可能会 导致其他线程CAS失败，解决办法是循环CAS直至成功。AQS的实现非常精巧，令人叹为观止，不入细节难以完全领会其精髓，下面详细说明实现过程：



   AbstractQueuedSynchronizer通过构造一个基于阻塞的CLH队列容纳所有的阻塞线程，而对该队列的操作均通过Lock-Free（CAS）操作，但对已经获得锁的线程而言，ReentrantLock实现了偏向锁的功能。

synchronized 的底层也是一个基于CAS操作的等待队列，但JVM实现的更精细，把等待队列分为ContentionList和EntryList，目的是为了降低线程的出列速度；当然也实现了偏向锁，从数据结构来说二者设计没有本质区别。但synchronized还实现了自旋锁，并针对不同的系统和硬件体系进行了优 化，而Lock则完全依靠系统阻塞挂起等待线程。

当然Lock比synchronized更适合在应用层扩展，可以继承 AbstractQueuedSynchronizer定义各种实现，比如实现读写锁（ReadWriteLock），公平或不公平锁；同时，Lock对 应的Condition也比wait/notify要方便的多、灵活的多。

 

state值，若为0，意味着此时没有线程获取到资源

**简述总结：**

  总体来讲线程获取锁要经历以下过程(非公平)：

  1、调用lock方法，会先进行cas操作看下可否设置同步状态1成功，如果成功执行临界区代码

  2、如果不成功获取同步状态，如果状态是0那么cas设置为1.

  3、如果同步状态既不是0也不是自身线程持有会把当前线程构造成一个节点。

  4、把当前线程节点CAS的方式放入队列中，行为上线程阻塞，内部自旋获取状态。

   （acquireQueued的主要作用是把已经追加到队列的线程节点进行**阻塞**，但阻塞前又通过tryAccquire重试是否能获得锁，如果重试成功能则无需阻塞，直接返回。）

  5、线程释放锁，唤醒队列第一个节点，参与竞争。重复上述。





# 面试

#### synchronized和lock的底层区别

**synchronized的底层也是一个基于CAS操作的等待队列**，但JVM实现的更精细，把等待队列分为ContentionList和EntryList，目的是为了降低线程的出列速度；当然也实现了偏向锁，从数据结构来说二者设计没有本质区别。但synchronized还实现了**自旋锁**，并针对不同的系统和硬件体系进行了优化，而Lock则完全依靠系统阻塞挂起等待线程。

 

当然Lock比synchronized更适合在应用层扩展，可以继承AbstractQueuedSynchronizer定义各种实现，比如实现读写锁（ReadWriteLock），公平或不公平锁；同时，Lock对应的Condition也比wait/notify要方便的多、灵活的多。

ReentrantLock是一个可重入的互斥锁，ReentrantLock由最近成功获取锁，还没有释放的线程所拥有

ReentrantLock与synchronized的区别

--ReentrantLock的lock机制有2种，忽略中断锁和响应中断锁

 

--synchronized实现的锁机制是可重入的，主要区别是中断控制和竞争锁公平策略

------

两者区别：

1.首先synchronized是java内置关键字，在jvm层面，Lock是个java类；

2.synchronized无法判断是否获取锁的状态，Lock可以判断是否获取到锁；

3.synchronized会自动释放锁(a 线程执行完同步代码会释放锁 ；b 线程执行过程中发生异常会释放锁)，Lock需在finally中手工释放锁（unlock()方法释放锁），否则容易造成线程死锁；

4.用synchronized关键字的两个线程1和线程2，如果当前线程1获得锁，线程2线程等待。如果线程1阻塞，线程2则会一直等待下去，而Lock锁就不一定会等待下去，如果尝试获取不到锁，线程可以不用一直等待就结束了；

5.synchronized的锁可重入、不可中断、非公平，而Lock锁可重入、可判断、可公平（两者皆可）

6.Lock锁适合大量同步的代码的同步问题，synchronized锁适合代码少量的同步问题。

 

#### **synchronized底层实现**

synchronized 属于重量级锁，效率低下，因为监视器锁（monitor）是依赖于底层操作系统的 Mutex Lock 来实现的，而操作系统实现线程之间的切换时需要从用户态转换到核心态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高，这也是为什么早期的 synchronized 效率低的原因。在 Java 6 之后 Java 官方从 JVM 层面对 synchronized 进行了较大优化，所以现在的 synchronized 锁效率也优化得很不错了。Java 6 之后，**为了减少获得锁和释放锁所带来的性能消耗**，引入了轻量级锁和偏向锁，

 

#### **Lock底层实现**

Lock底层实现基于AQS实现，采用线程独占的方式，在硬件层面依赖特殊的CPU指令（CAS）。

简单来说，ReenTrantLock的实现是一种**自旋锁**，通过循环调用CAS操作来实现加锁。它的性能比较好也是因为**避免了使线程进入内核态的阻塞状态。**想尽办法避免线程进入内核的阻塞状态是我们去分析和理解锁设计的关键钥匙。

 

#### **volatile底层实现**

在JVM底层volatile是采用“**内存屏障**”来实现的。

 

##### lock和Monitor的区别

一、lock的底层本身是Monitor来实现的，所以Monitor可以实现lock的所有功能。

二、Monitor有TryEnter的功能，可以防止出现死锁的问题，lock没有。















