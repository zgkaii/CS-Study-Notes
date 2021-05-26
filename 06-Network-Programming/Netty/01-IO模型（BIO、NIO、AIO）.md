# 一、IO模型分类

首先明确两个概念：**用户空间和内核空间**。

操作系统的核心是内核(kernel)，它独立于普通的应用程序，可以访问受保护的内存空间，也有访问底层硬件设备的所有权限。为了保证内核的安全，现在的操作系统一般都强制用户进程不能直接操作内核。具体的实现方式基本都是由操作系统将虚拟地址空间划分为两部分，一部分为内核空间，另一部分为用户空间。

其实所有的系统资源管理都是在内核空间中完成的。比如读写磁盘文件，分配回收内存，从网络接口读写数据等等。我们的应用程序是无法直接进行这样的操作的。但是我们可以通过内核提供的接口来完成这样的任务。

所以，当我们使用 TCP 发送数据的时候，需要先将数据从用户空间拷贝到内核空间，再由内核操作将数据从内核空间发送出去；当我们使用 TCP 读取数据的时候，数据先在内核空间准备好，再从内核空间拷贝到用户空间供用户进程使用。

```html
	客户端 --1.发送请求-> 网关 --2.拷贝-> 内核空间 --3.拷贝-> 用户空间
															 |
	客户端	<--7.响应-- 网关 <--6.拷贝-- 内核空间 <--5.拷贝-- 用户空间（4.用户进程处理数据）				 
```

所以，一次 IO 的读取操作分为两个阶段（写入操作类似）：

- 等待内核空间准备数据
- 数据从内核空间拷贝到用户空间

在这基础之上，Unix 把 IO 分成了以下五种 IO 模型：

- 阻塞型 IO
- 非阻塞型 IO
- IO 多路复用
- 信号驱动 IO
- 异步 IO

## 1.1 阻塞型 IO

阻塞型 IO，即当用户进程发起请求时，一直阻塞直到数据拷贝到用户空间为止才返回（两个阶段都阻塞）。

在阻塞的过程中，其它应用进程还可以执行，因此阻塞不意味着整个操作系统都被阻塞。因为其它应用进程还可以执行，所以不消耗 CPU 时间，这种模型的 CPU 利用率会比较高。

 <div align="center"> <img src="..\..\..\images\nio\阻塞式IO.png" width="1000px"></div>

## 1.2 非阻塞型 IO

和阻塞 IO 类比，无论内核空间数据是否准备好，非阻塞型 IO内核都会立即返回，返回后获得足够的 CPU 时间继续做其它的事情。 用户进程第一个阶段不是阻塞的，需要不断的主动询问 kernel 数据好了没有（轮询`polling`）；第二个阶段依然总是阻塞的。

由于 CPU 要处理更多的系统调用，因此这种模型的 CPU 利用率比较低。

 <div align="center"> <img src="..\..\..\images\nio\非阻塞式IO.png" width="1000px"></div>

## 1.3 I/O 多路复用

IO 多路复用(IO multiplexing)，也称事件驱动 IO(event-driven IO)，就是在单个线程里同时监控多个套接字，通过 select 或 poll 轮询所负责的所有 socket，当某个 socket 有数据到达了，就通知用户进程。

 <div align="center"> <img src="..\..\..\images\nio\复用IO.png" width="1000px"></div>

IO 复用同非阻塞 IO 本质一样，不过利用 了新的 select 系统调用，由内核来负责本 来是请求进程该做的轮询操作。看似比非阻塞 IO 还多了一个系统调用开销，不过因为可以支持多路 IO，才算提高了效率。

进程先是阻塞在 select/poll 上，再是阻塞在读操作的第二个阶段上。

 <div align="center"> <img src="..\..\..\images\nio\复用IO2.png" width="600px"></div>

select/poll 的几大缺点：

（1）每次调用 select，都需要把 `fd` 集合从用户态拷贝到 内核态，这个开销在 `fd` 很多时会很大 。

（2）同时每次调用 select 都需要在内核遍历传递进来的 所有 `fd`，这个开销在 `fd` 很多时也很大 。

（3）select 支持的文件描述符数量太小了，默认是1024 。

> epoll（Linux 2.5.44内核中引入，2.6内核正式引入，可被用 于代替 POSIX select 和 poll 系统调用）： 
>
> （1）内核与用户空间共享一块内存。 
>
> （2）通过回调解决遍历问题 。
>
> （3）`fd` 没有限制，可以支撑10万连接。

## 1.4 信号驱动 I/O

信号驱动 IO 与 BIO 和 NIO 最大的区别就在 于，在 IO 执行的数据准备阶段，不需要轮询。 

当用户进程需要等待数据的时候 ，会向内核发送一个信号，告诉内核我要什么数据，然后用户进程就继续做别的事情去 了，而当内核中的数据准备好之后，内核立马发给用户进程一个信号，说”数据准备好 了，快来查收“，用户进程收到信号之后， 立马调用 recvfrom，去查收数据。

相比于非阻塞式I/O的轮询方式，信号驱动 I/O 的CPU利用率更高。

 <div align="center"> <img src="..\..\..\images\nio\信号驱动IO.png" width="600px"></div>

## 1.5 异步 I/O

异步 IO 真正实现了 IO 全流程的非阻塞。 用户进程发出系统调用后立即返回，内核等待数据准备完成，然后将数据拷贝到用 户进程缓冲区，然后发送信号告诉用户进 程 IO 操作执行完毕（与 SIGIO 相比，一 个是发送信号告诉用户进程数据准备完毕， 一个是 IO执行完毕）。

 <div align="center"> <img src="..\..\..\images\nio\异步IO.png" width="600px"></div>

## 1.6 五大 I/O 模型比较

 <div align="center"> <img src="..\..\..\images\nio\IO模型.png" width="400px"></div>

- 同步 I/O：将数据从内核缓冲区复制到应用进程缓冲区的阶段（第二阶段），应用进程会阻塞。
- 异步 I/O：第二阶段应用进程不会阻塞。

同步 I/O 包括阻塞式 I/O、非阻塞式 I/O、I/O 复用和信号驱动 I/O ，它们的主要区别在第一个阶段。

非阻塞式 I/O 、信号驱动 I/O 和异步 I/O 在第一阶段不会阻塞。

# 二、BIO、NIO与AIO

先来回顾一下这样几个概念：同步与异步，阻塞与非阻塞。参考 [Stackoverflow](https://stackoverflow.com/questions/748175/asynchronous-vs-synchronous-execution-what-does-it-really-mean)相关问题的回答：

> When you execute something synchronously, you wait for it to finish before moving on to another task. When you execute something asynchronously, you can move on to another task before it finishes.
>
> 当你同步执行某项任务时，你需要等待其完成才能继续执行其他任务。当你异步执行某些操作时，你可以在完成另一个任务之前继续进行。

**同步与异步**

- **同步** ：两个同步任务相互依赖，并且一个任务必须以依赖于另一任务的某种方式执行。 比如在`A->B`事件模型中，你需要先完成 A 才能执行B。 再换句话说，同步调用中被调用者未处理完请求之前，调用不返回，调用者会一直等待结果的返回。
- **异步**： 两个异步的任务完全独立的，一方的执行不需要等待另外一方的执行。再换句话说，异步调用种一调用就返回结果不需要等待结果返回，当结果返回的时候通过回调函数或者其他方式拿着结果再做相关事情，

**阻塞和非阻塞**

- **阻塞：** 阻塞就是发起一个请求，调用者一直等待请求结果返回，也就是当前线程会被挂起，无法从事其他任务，只有当条件就绪才能继续。
- **非阻塞：** 非阻塞就是发起一个请求，调用者不用一直等着结果返回，可以先去干其他事情。

**如何区分 “同步/异步 ”和 “阻塞/非阻塞” 呢？**

同步/异步是从**行为**角度描述事物的，而阻塞和非阻塞描述的当前事物的**状态**（等待调用结果时的状态）。

**BIO、NIO与AIO：**

（1）BIO (Blocking I/O)，阻塞型 IO，也称为 OIO，Old IO。

（2）NIO（New IO也可以理解为No-Locking IO），Java 中使用 IO 多路复用技术实现，放在 `java.nio` 包下，JDK1.4 引入。

（3）AIO（Asynchronous I/O），异步 IO，又称为 NIO2，也是放在 `java.nio` 包下，JDK1.7 引入。

## 2.1 BIO

BIO采用同步阻塞模型，当用户进程发起请求时，一直阻塞到数据拷贝到用户空间为止。（**Java BIO 就是传统I/O编程**，其相关的类和接口都在 java.io包下）

### 2.1.1 传统 BIO

BIO通信（一请求一应答）模型图如下：

![传统BIO通信模型图](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2.png)

采用 **BIO 通信模型** 的服务端，通常由一个独立的 Acceptor 线程负责监听客户端的连接。我们一般通过在`while(true)` 循环中服务端会调用 `accept()` 方法等待接收客户端的连接的方式监听请求，请求一旦接收到一个连接请求，就可以建立通信套接字在这个通信套接字上进行读写操作，此时不能再接收其他客户端连接请求，只能等待同当前连接的客户端的操作执行完成， 不过可以通过多线程来支持多个客户端的连接，如上图所示。

如果要让 **BIO 通信模型** 能够同时处理多个客户端请求，就必须使用多线程（主要原因是`socket.accept()`、`socket.read()`、`socket.write()` 涉及的三个主要函数都是同步阻塞的），也就是说它在接收到客户端连接请求之后为每个客户端创建一个新的线程进行链路处理，处理完成之后，通过输出流返回应答给客户端，线程销毁。这就是典型的 **一请求一应答通信模型** 。我们可以设想一下如果这个连接不做任何事情的话就会造成不必要的线程开销，不过可以通过 **线程池机制** 改善，线程池还可以让线程的创建和回收成本相对较低。使用`FixedThreadPool` 可以有效的控制了线程的最大数量，保证了系统有限的资源的控制，实现了N(客户端请求数量):M(处理客户端请求的线程数量)的伪异步I/O模型（N 可以远远大于 M），下面一节"伪异步 BIO"中会详细介绍到。

**我们再设想一下当客户端并发访问量增加后这种模型会出现什么问题？**

在 Java 虚拟机中，线程是宝贵的资源，线程的创建和销毁成本很高，除此之外，线程的切换成本也是很高的。尤其在 Linux 这样的操作系统中，线程本质上就是一个进程，创建和销毁线程都是重量级的系统函数。如果并发访问量增加会导致线程数急剧膨胀可能会导致线程堆栈溢出、创建新线程失败等问题，最终导致进程宕机或者僵死，不能对外提供服务。

### 2.1.2 伪异步 IO

为了解决同步阻塞I/O面临的一个链路需要一个线程处理的问题，后来有人对它的线程模型进行了优化一一一后端通过一个**线程池**来处理多个客户端的请求接入，形成客户端个数M：线程池最大线程数N的比例关系，其中M可以远远大于N.通过线程池可以灵活地调配线程资源，设置线程的最大值，防止由于海量并发接入导致线程耗尽。

伪异步IO模型图：

<img src="https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/3.png" alt="伪异步IO模型图" style="zoom:80%;" />

采用线程池和任务队列可以实现一种叫做伪异步的 I/O 通信框架，它的模型图如上图所示。当有新的客户端接入时，将客户端的 Socket 封装成一个Task（该任务实现`java.lang.Runnable`接口）投递到后端的线程池中进行处理，JDK 的线程池维护一个消息队列和 N 个活跃线程，对消息队列中的任务进行处理。由于线程池可以设置消息队列的大小和最大线程数，因此，它的资源占用是可控的，无论多少个客户端并发访问，都不会导致资源的耗尽和宕机。

伪异步I/O通信框架采用了线程池实现，因此避免了为每个请求都创建一个独立线程造成的线程资源耗尽问题。不过因为它的底层仍然是同步阻塞的BIO模型，因此无法从根本上解决问题。

### 1.3 代码示例

下面代码中演示了BIO通信（一请求一应答）模型。我们会在客户端创建多个线程依次连接服务端并向其发送"当前时间+:hello world"，服务端会为每个客户端线程创建一个线程来处理。

**客户端**

```java
public class IOClient {
    public static void main(String[] args) {
        new Thread(() -> {
            try {
                Socket socket = new Socket("127.0.0.1", 6666);
                while (true) {
                    try {
                        socket.getOutputStream().write((new Date() + ": hello world").getBytes());
                        Thread.sleep(2000);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```

**服务端**

```java
public class IOServer {
    public static void main(String[] args) throws IOException {
        // TODO 服务端处理客户端连接请求
        ServerSocket serverSocket = new ServerSocket(6666);

        // 接收到客户端连接请求之后为每个客户端创建一个新的线程进行链路处理
        new Thread(() -> {
            while (true) {
                try {
                    // 阻塞方法获取新的连接
                    Socket socket = serverSocket.accept();

                    // 每一个新的连接都创建一个线程，负责读取数据
                    new Thread(() -> {
                        try {
                            int len;
                            byte[] data = new byte[1024];
                            InputStream inputStream = socket.getInputStream();
                            // 按字节流方式读取数据
                            while ((len = inputStream.read(data)) != -1) {
                                System.out.println(new String(data, 0, len));
                            }
                        } catch (IOException e) {
                        }
                    }).start();

                } catch (IOException e) {
                }
            }
        }).start();
    }
}
```

**小结**

在活动连接数不是特别高（小于单机1000）的情况下，这种模型是比较不错的，可以让每一个连接专注于自己的 I/O 并且编程模型简单，也不用过多考虑系统的过载、限流等问题。线程池本身就是一个天然的漏斗，可以缓冲一些系统处理不了的连接或请求。但是，当面对十万甚至百万级连接的时候，传统的 BIO 模型是无能为力的。因此，我们需要一种更高效的 I/O 处理模型来应对更高的并发量。

## 2.2 NIO (New I/O)

### 2.2.1 NIO 简介

NIO是一种**同步非阻塞**的多路复用 IO模型，在Java 1.4 中引入了 NIO 框架，对应 `java.nio` 包，**主要有 Channel（通道） , Selector（选择器），Buffer（缓冲区）三大核心组件**（整个NIO体系包含的类远远不止这三个，只能说这三个是NIO体系的“核心API”）。

NIO中的N可以理解为**Non-blocking**，不单纯是New。它支持面向缓冲的，基于通道的I/O操作方法。 NIO提供了与传统BIO模型中的 `Socket` 和 `ServerSocket` 相对应的 `SocketChannel` 和 `ServerSocketChannel` 两种不同的套接字通道实现，两种通道都支持阻塞和非阻塞两种模式。阻塞模式使用就像传统中的支持一样，比较简单，但是性能和可靠性都不好；非阻塞模式正好与之相反。对于低负载、低并发的应用程序，可以使用同步阻塞I/O来提升开发速率和更好的维护性；对于高负载、高并发的（网络）应用，应使用 NIO 的非阻塞模式来开发。

<img src="https://img-blog.csdnimg.cn/20210109215128592.png" style="zoom:80%;" />

通常来说，NIO中的所有读数据或写数据都是从 Channel（通道） 开始的。

- 从通道进行数据读取 ：创建一个缓冲区，然后请求通道读取数据。
- 从通道进行数据写入 ：创建一个缓冲区，填充数据，并要求通道写入数据。

数据读取和写入操作图示：

<img src="https://img-blog.csdnimg.cn/2021010921561582.png" style="zoom:80%;" />

### 2.2.2 NIO与BIO的区别

如果是在面试中回答这个问题，我觉得首先肯定要从 NIO 流是非阻塞 IO 而 BIO是阻塞 IO 说起。然后，可以从 NIO 的3个核心组件/特性为 NIO 带来的一些改进来分析。

#### （1）Non-blocking（非阻塞）

**BIO流是阻塞的，NIO流是不阻塞的。**

Java IO的各种流是阻塞的。这意味着，当一个线程调用 `read()` 或  `write()` 时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了。

Java NIO使我们可以进行非阻塞IO操作。比如说，单线程中从通道读取数据到buffer，同时可以继续做别的事情，当数据读取到buffer中后，线程再继续处理数据。写数据也是一样的。另外，非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。

#### （2）Buffer（缓冲区）

**BIO 面向流(Stream oriented)，而 NIO 面向缓冲区(Buffer oriented)。**

Buffer 是一个容器，它包含一些要写入或者要读出的数据。最常用的Buffer是 ByteBuffer，一个 ByteBuffer 提供了一组功能用于操作 byte 数组。除了ByteBuffer，还有其他的一些缓冲区，事实上，每一种Java基本类型（除了Boolean类型）都对应有一种缓冲区，例如`CharBuffer/ShortBuffer/IntBuffer/LongBuffer/FloatBuffer/DoubleBuffer/MappedByteBuffer`等。

Buffer 中有几个重要的成员属性：

| **属性**        | **描述**                                                     |
| --------------- | ------------------------------------------------------------ |
| （int）capacity | 容量，即可以容纳的最大数据量；在缓冲区创建时被设定并且不能改变。 |
| （int）limit    | 表示缓冲区的当前终点，不能对缓冲区超过极限的位置进行读写操作，但极限是可以修改的。 |
| （int）position | 位置，下一个要被读或写的元素的索引，每次读写缓冲区数据时都会改变改值，为下一次读写作准备。 |
| （int）mark     | 相当一个暂存属性，暂时保存position的值，方便后面的重复使用position值 ，在`flip()`方法被调用后作废。 |
| (long)address   | address 用于操作直接内存，区别于 jvm 内存。                  |

> 属性值大小比较：mark <= position <= limit <= capacity。

值得注意的是，在不同模式下，limit和position属性的值是不同的：

<img src="https://img-blog.csdnimg.cn/20210109223454134.png" style="zoom: 67%;" />

写模式下，**所谓写模式就是将缓存区中的内容写入通道**。position 代表下一个字节应该被写出去的字节在缓存区中的位置，limit 表示最后一个待写字节在缓存区的位置。

读模式下，**所谓读模式就是从通道读取数据到缓存区**。position 代表下一个读出来的字节应当存储在缓存区的位置，limit 等于 capacity。

可见，在NIO类库中加入Buffer对象，体现出了NIO与BIO的重要区别：

* BIO中，可以将数据直接写入或者将数据直接读到 Stream 对象中。虽然 Stream 中也有 Buffer 开头的扩展类，但只是流的包装类，还是从流读到缓冲区。

* 而在NIO中，所有数据都是用缓冲区处理的。在读取数据时，它是直接读到缓冲区中的; 在写入数据时，写入到缓冲区中。任何时候访问NIO中的数据，都是通过缓冲区进行操作。

#### （3）Channel (通道)

**BIO流的读写只能单向的，NIO 可通过Channel（通道） 进行双向读写。**

**Channel 是一种 IO 操作的连接**（nexus，连接的意思），它代表的是到实体的开放连接，这个实体可以是硬件设备、文件、网络套接字或者可执行 IO 操作（比如读、写）的程序组件。

`通道是双向的`也是有前提的，那就是通道基于随机访问文件`RandomAccessFile`的可读可写文件指针。`RandomAccessFile`是既可读又可写的，所以基于它的通道是双向的，所以，`通道是双向的`这句话是有前提的，不能断章取义。

基本的通道类型有：`FileChannel/DatagramChannel/SocketChannel/ServerSocketChannel`等。**FileChannel 是基于文件的通道，SocketChannel 和 ServerSocketChannel 用于网络 TCP 套接字数据报读写，DatagramChannel 是用于网络 UDP 套接字数据报读写**。

通道不能单独存在，它**必须绑定一个缓存区**，所有的数据只会存在于缓存区中，无论你是写或是读，必然是缓存区通过通道到达磁盘文件，或是磁盘文件通过通道到达缓存区。

#### （4）Selector (选择器)

**NIO有选择器，而BIO没有。**

Selector 是 Java NIO 的一个组件，Selector 是一个多路复用器。它用于监听多个 Channel 的各种状态，用于管理多个 Channel，用于完成IO**多路复用**。选择器和通道的关系是**监控与被监控的关系**。

通道和选择器的关系是通过`register`(注册)的方式完成的。通过调用通道的`Channel.register（Selector sel，int ops）`可以将**指定通道的一个或多个IO事件类型**注册到**选择器**中。可供选择器监控的通道IO事件类型，包括以下四种：

- 可读：`SelectionKey.OP_READ`
- 可写：`SelectionKey.OP_WRITE`
- 连接：`SelectionKey.OP_CONNECT`
- 接收：`SelectionKey.OP_A`

由于 FileChannel 不支持注册选择器，所以 Selector 一般被认为是服务于网络套接字通道的。而大家口中的`NIO 是非阻塞的`，准确来说，指的是**网络编程中客户端与服务端连接交换数据的过程是非阻塞的**。普通的文件读写依然是阻塞的，和 IO 是一样的。

<img src="https://img-blog.csdnimg.cn/20210109231301646.png" style="zoom: 50%;" />

### 2.2.3 代码示例

客户端 `IOClient.java`的代码不变，我们对服务端使用 NIO 进行改造。

```java
public class NIOEchoServer {
    public static void main(String[] args) throws IOException {
        // 创建一个Selector
        Selector selector = Selector.open();
        // 创建ServerSocketChannel
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        // 绑定6666端口
        serverSocketChannel.bind(new InetSocketAddress(6666));
        // 设置为非阻塞模式
        serverSocketChannel.configureBlocking(false);
        // 将Channel注册到selector上，并注册Accept事件
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        System.out.println("server start");

        while (true) {
            // 阻塞在select上（第一阶段阻塞）
            selector.select();
            // 如果使用的是select(timeout)或selectNow()需要判断返回值是否大于0

            // 有就绪的Channel
            Set<SelectionKey> selectionKeys = selector.selectedKeys();
            // 遍历selectKeys
            Iterator<SelectionKey> iterator = selectionKeys.iterator();
            while (iterator.hasNext()) {
                SelectionKey selectionKey = iterator.next();
                // 如果是accept事件
                if (selectionKey.isAcceptable()) {
                    // 强制转换为ServerSocketChannel
                    ServerSocketChannel ssc = (ServerSocketChannel) selectionKey.channel();
                    SocketChannel socketChannel = ssc.accept();
                    System.out.println("accept new conn: " + socketChannel.getRemoteAddress());
                    socketChannel.configureBlocking(false);
                    // 将SocketChannel注册到Selector上，并注册读事件
                    socketChannel.register(selector, SelectionKey.OP_READ);
                } else if (selectionKey.isReadable()) {
                    // 如果是读取事件
                    // 强制转换为SocketChannel
                    SocketChannel socketChannel = (SocketChannel) selectionKey.channel();
                    // 创建Buffer用于读取数据
                    ByteBuffer buffer = ByteBuffer.allocate(1024);
                    // 将数据读入到buffer中（第二阶段阻塞）
                    int length = socketChannel.read(buffer);
                    if (length > 0) {
                        buffer.flip();
                        byte[] bytes = new byte[buffer.remaining()];
                        // 将数据读入到byte数组中
                        buffer.get(bytes);

                        // 换行符会跟着消息一起传过来
                        String content = new String(bytes, "UTF-8").replace("\r\n", "");
                        System.out.println("receive msg: " + content);
                    }
                }
                iterator.remove();
            }
        }
    }
}
```

首先，我们创建了一个 Selector，充当 IO 多路复用中的选择器；其次，我们启动了一个 ServerSocketChannel，并设置其为非阻塞模式，与 BIO 中的 ServerSocket 类似，是服务端进程。再次，我们把 ServerSocketChannel 注册到 Selector 上；然后通过`while(true)`让Selector 轮询下去，每次轮询结束完会拿到一系列 Key，这些 Key 叫作 SelectionKey，每个 SelectionKey 里面都绑定了一个数据准备好了的 Channel，通过这个 Channel 我们就可以去取数据了。最后，遍历这些 SelectionKey，取出其中的 Channel，再根据不同的事件类型用 Channel 去读取数据并打印出来。

为什么大家都不愿意用 JDK 原生 NIO 进行开发呢？从上面的代码中大家都可以看出来，除了编程复杂、编程模型难之外，它还有以下让人诟病的问题：

- JDK 的 NIO 底层由 epoll 实现，该实现饱受诟病的空轮询 bug 会导致 cpu 飙升 100%。
- 项目庞大之后，自行实现的 NIO 很容易出现各类 bug，维护成本较高。

**Netty** 的出现很大程度上改善了 JDK 原生 NIO 所存在的一些让人难以忍受的问题。

## 2.3 AIO (Asynchronous I/O)

AIO 也就是 NIO 2。在 Java 7 中引入了 NIO 的改进版 NIO 2,它是异步非阻塞的IO模型。异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。

AIO 是异步IO的缩写，虽然 NIO 在网络操作中，提供了非阻塞的方法，但是 NIO 的 IO 行为还是同步的。对于 NIO 来说，我们的业务线程是在 IO 操作准备好时，得到通知，接着就由这个线程自行进行 IO 操作，IO操作本身是同步的。（除了 AIO 其他的 IO 类型都是同步的，这一点可以从底层IO线程模型解释，推荐一篇文章：[《漫话：如何给女朋友解释什么是Linux的五种IO模型？》](https://mp.weixin.qq.com/s?__biz=Mzg3MjA4MTExMw==&mid=2247484746&amp;idx=1&amp;sn=c0a7f9129d780786cabfcac0a8aa6bb7&source=41#wechat_redirect) ）

还是以上面代码为例，客户端 `IOClient.java`的代码不变，我们对服务端使用 AIO 进行改造。

```java
public class AIOEchoServer {
    public static void main(String[] args) throws IOException {
        // 启动服务端
        AsynchronousServerSocketChannel serverSocketChannel = AsynchronousServerSocketChannel.open();
        serverSocketChannel.bind(new InetSocketAddress(6666));

        System.out.println("server start");

        // 监听accept事件，完全异步，不会阻塞
        serverSocketChannel.accept(null, new CompletionHandler<AsynchronousSocketChannel, Object>() {
            @Override
            public void completed(AsynchronousSocketChannel socketChannel, Object attachment) {
                try {
                    System.out.println("accept new conn: " + socketChannel.getRemoteAddress());
                    // 再次监听accept事件
                    serverSocketChannel.accept(null, this);

                    // 消息的处理
                    while (true) {
                        ByteBuffer buffer = ByteBuffer.allocate(1024);
                        // 将数据读入到buffer中
                        Future<Integer> future = socketChannel.read(buffer);
                        if (future.get() > 0) {
                            buffer.flip();
                            byte[] bytes = new byte[buffer.remaining()];
                            // 将数据读入到byte数组中
                            buffer.get(bytes);

                            String content = new String(bytes, "UTF-8");
                            // 换行符会当成另一条消息传过来
                            if (content.equals("\r\n")) {
                                continue;
                            }
                            System.out.println("receive msg: " + content);
                        }
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }

            @Override
            public void failed(Throwable exc, Object attachment) {
                System.out.println("failed");
            }
        });
        // 阻塞住主线程
        System.in.read();
    }
}
```

首先，我们启动了一个 AsynchronousServerSocketChannel，它与 BIO 中的 ServerSocket 和 NIO 中的 ServerSocketChannel 类似，是一个服务端进程；然后，我们通过 accept () 方法监听客户端连接，用法跟 BIO 和 NIO 都一样，但是，这个 accept () 执行方式完全不一样了，BIO 中的 accept () 是完全阻塞当前线程的，NIO 中的 accept () 是通过 Accept 事件来实现的，而 AIO 中的 accept () 是完全异步的，执行了这个方法之后会立即执行后续的代码，不会停留在 accept () 这一行，所以，在 main () 方法的最后需要加一行阻塞代码，否则 main () 方法执行完毕，进程就结束了。

最后，在 accept () 方法的回调方法 complete () 中处理数据，这里的数据已经经历过数据准备和从内核空间拷贝到用户空间两个阶段了，到达用户空间是真正可用的数据，而不像 BIO 和 NIO 那样还要自己去阻塞着把数据从内核空间拷贝到用户空间再使用。

从效率上来看，AIO 无疑是最高的，然而，遗憾地是，目前作为广大服务器使用的系统 linux 对 AIO 的支持还不完善，导致 AIO 的应用还不是很广泛，Netty 之前也尝试使用过 AIO，不过又放弃了。

## 2.4. BIO、NIO、AIO适用场景

| 对比     | **BIO**  | **NIO**                | **AIO**    |
| -------- | -------- | ---------------------- | ---------- |
| IO 模型  | 同步阻塞 | 同步非阻塞（多路复用） | 异步非阻塞 |
| 编程难度 | 简单     | 复杂                   | 复杂       |
| 可靠性   | 差       | 好                     | 好         |
| 吞吐量   | 低       | 高                     | 高         |

* BIO方式适用于**连接数目比较小且固定**的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK 1.4以前的唯一选择，但程序简单易理解。
* NIO方式适用于**连接数目多且连接比较短**（轻操作）的架构，比如聊天服务器，弹幕系统，服务器间通讯等。编程比较复杂，JDK 1.4开始支持。
* AIO方式使用于**连接数目多且连接比较长**（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，JDK 1.7开始支持。

# 参考

- 《Netty 权威指南》第二版
- [Java NIO浅析](https://zhuanlan.zhihu.com/p/23488863)
- [详解 Java NIO](https://juejin.cn/post/6844903620454924301)
- [Buffer的4个属性及重要方法](https://zhuanlan.zhihu.com/p/23488863)
- [零拷贝介绍及对比](https://houbb.github.io/2018/09/22/java-nio-09-16-zero-copy-compare) / [Java NIO零拷贝实现](https://houbb.github.io/2018/09/22/java-nio-09-17-zero-copy-java-impl)