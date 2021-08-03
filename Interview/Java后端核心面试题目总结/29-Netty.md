<!-- MarkdownTOC -->

- [IO模型](#io模型)
  - [I/O 模型有哪几类呢？](#io-模型有哪几类呢)
  - [BIO、NIO、AIO 是什么？](#bionioaio-是什么)
  - [BIO、NIO 有什么区别？](#bionio-有什么区别)
  - [BIO、NIO、AIO 适用场景?](#bionioaio-适用场景)
- [Netty基础](#netty基础)
  - [什么是 Netty ？](#什么是-netty-)
  - [为什么选择 Netty ？](#为什么选择-netty-)
  - [为什么说 Netty 使用简单？](#为什么说-netty-使用简单)
  - [说说业务中 Netty 的使用场景？](#说说业务中-netty-的使用场景)
  - [说说 Netty 如何实现高性能？](#说说-netty-如何实现高性能)
  - [Netty 的高性能如何体现？](#netty-的高性能如何体现)
  - [Netty 的可用性如何体现？](#netty-的可用性如何体现)
  - [Netty 的可扩展如何体现？](#netty-的可扩展如何体现)
- [Netty模型架构](#netty模型架构)
  - [简单介绍 Netty 的核心组件？](#简单介绍-netty-的核心组件)
  - [说说 Netty 的逻辑架构？](#说说-netty-的逻辑架构)
  - [什么是 Reactor 模型？](#什么是-reactor-模型)
  - [请介绍 Netty 的线程模型？](#请介绍-netty-的线程模型)
    - [Reacto单线程](#reacto单线程)
    - [Reacto多线程](#reacto多线程)
    - [Reacto主从多线程](#reacto主从多线程)
    - [Netty线程模型实现](#netty线程模型实现)
  - [什么是业务线程池？](#什么是业务线程池)
- [Netty网络程序优化](#netty网络程序优化)
  - [TCP 粘包 / 拆包的原因？](#tcp-粘包--拆包的原因)
  - [应该这么解决粘包 / 拆包？](#应该这么解决粘包--拆包)
  - [了解哪几种序列化协议？](#了解哪几种序列化协议)
  - [Netty 的零拷贝实现？](#netty-的零拷贝实现)
  - [原生的 NIO 存在 Epoll Bug 是什么？Netty 是怎么解决的？](#原生的-nio-存在-epoll-bug-是什么netty-是怎么解决的)
  - [Netty 长连接、心跳机制了解么？](#netty-长连接心跳机制了解么)
  - [什么是 Netty 空闲检测？](#什么是-netty-空闲检测)
  - [Netty 如何实现重连？](#netty-如何实现重连)
  - [Netty 自己实现的 ByteBuf 有什么优点？](#netty-自己实现的-bytebuf-有什么优点)
  - [Netty 为什么要实现内存管理？](#netty-为什么要实现内存管理)
  - [Netty 如何实现内存管理？](#netty-如何实现内存管理)
  - [什么是 Netty 的内存泄露检测？是如何进行实现的？](#什么是-netty-的内存泄露检测是如何进行实现的)

<!-- /MarkdownTOC -->

# IO模型

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

## I/O 模型有哪几类呢？

**阻塞型 IO**

阻塞型 IO，即当用户进程发起请求时，一直阻塞直到数据拷贝到用户空间为止才返回（两个阶段都阻塞）。

在阻塞的过程中，其它应用进程还可以执行，因此阻塞不意味着整个操作系统都被阻塞。因为其它应用进程还可以执行，所以不消耗 CPU 时间，这种模型的 CPU 利用率会比较高。

 <div align="center"> <img src="..\..\06-Network-Programming\images\nio\阻塞式IO.png" width="1000px"></div>

**非阻塞型 IO**

和阻塞 IO 类比，无论内核空间数据是否准备好，非阻塞型 IO内核都会立即返回，返回后获得足够的 CPU 时间继续做其它的事情。 用户进程第一个阶段不是阻塞的，需要不断的主动询问 kernel 数据好了没有（轮询`polling`）；第二个阶段依然总是阻塞的。

由于 CPU 要处理更多的系统调用，因此这种模型的 CPU 利用率比较低。

 <div align="center"> <img src="..\..\06-Network-Programming\images\nio\非阻塞式IO.png" width="1000px"></div>

**I/O 多路复用**

IO 多路复用(IO multiplexing)，也称事件驱动 IO(event-driven IO)，就是在单个线程里同时监控多个套接字，通过 select 或 poll 轮询所负责的所有 socket，当某个 socket 有数据到达了，就通知用户进程。

 <div align="center"> <img src="..\..\06-Network-Programming\images\nio\复用IO.png" width="1000px"></div>

IO 复用同非阻塞 IO 本质一样，不过利用 了新的 select 系统调用，由内核来负责本来是请求进程该做的轮询操作。看似比非阻塞 IO 还多了一个系统调用开销，不过因为可以支持多路 IO，才算提高了效率。

进程先是阻塞在 select/poll 上，再是阻塞在读操作的第二个阶段上。

 <div align="center"> <img src="..\..\06-Network-Programming\images\nio\复用IO2.png" width="600px"></div>

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

**信号驱动 I/O**

信号驱动 IO 与 BIO 和 NIO 最大的区别就在 于，在 IO 执行的数据准备阶段，不需要轮询。 

当用户进程需要等待数据的时候 ，会向内核发送一个信号，告诉内核我要什么数据，然后用户进程就继续做别的事情去 了，而当内核中的数据准备好之后，内核立马发给用户进程一个信号，说”数据准备好 了，快来查收“，用户进程收到信号之后， 立马调用 `recvfrom`，去查收数据。

相比于非阻塞式I/O的轮询方式，信号驱动 I/O 的CPU利用率更高。

 <div align="center"> <img src="..\..\06-Network-Programming\images\nio\信号驱动IO.png" width="600px"></div>

**异步 I/O**

异步 IO 真正实现了 IO 全流程的非阻塞。 用户进程发出系统调用后立即返回，内核等待数据准备完成，然后将数据拷贝到用户进程缓冲区，然后发送信号告诉用户进程 IO 操作执行完毕（与 SIGIO 相比，一个是发送信号告诉用户进程数据准备完毕， 一个是 IO执行完毕）。

 <div align="center"> <img src="..\..\06-Network-Programming\images\nio\异步IO.png" width="600px"></div>

**五大 I/O 模型比较**

 <div align="center"> <img src="..\..\06-Network-Programming\images\nio\IO模型.png" width="400px"></div>

- 同步 I/O：将数据从内核缓冲区复制到应用进程缓冲区的阶段（第二阶段），应用进程会阻塞。
- 异步 I/O：第二阶段应用进程不会阻塞。

同步 I/O 包括阻塞式 I/O、非阻塞式 I/O、I/O 复用和信号驱动 I/O ，它们的主要区别在第一个阶段。

非阻塞式 I/O 、信号驱动 I/O 和异步 I/O 在第一阶段不会阻塞。

## BIO、NIO、AIO 是什么？

**1.BIO (Blocking I/O)**

BIO (Blocking I/O)，阻塞型 IO，也称为 OIO，Old IO。是一种**阻塞** + **同步**的通信模式。是一个比较传统的通信方式，模式简单，使用方便。但并发处理能力低，通信耗时，依赖网速。

服务器通过一个 Acceptor 线程，负责监听客户端请求和为每个客户端创建一个新的线程进行链路处理。典型的**一请求一应答模式**。

 <div align="center"> <img src="..\..\06-Network-Programming\images\nio\BIO.png" width="600px"></div>

若客户端数量增多，频繁地创建和销毁线程会给服务器打开很大的压力。后改良为用线程池的方式代替新增线程，被称为伪异步 IO 。

 <div align="center"> <img src="..\..\06-Network-Programming\images\nio\BIO2.png" width="600px"></div>

BIO 模型中，通过 Socket 和 `ServerSocket` 实现套接字通道的通信。阻塞，同步，建立连接耗时。

**2. NIO**

NIO（New IO也可以理解为No-Locking IO），Java 中使用 IO 多路复用技术实现，放在 `java.nio` 包下，JDK1.4 引入。

NIO是一种**同步非阻塞**的多路复用 IO模型，NIO 通过一个线程轮询，实现千万个客户端的请求，这就是非阻塞 NIO 的特点。

它支持面向缓冲的，基于通道的I/O操作方法。 NIO提供了与传统BIO模型中的 `Socket` 和 `ServerSocket` 相对应的 `SocketChannel` 和 `ServerSocketChannel` 两种不同的套接字通道实现，两种通道都支持阻塞和非阻塞两种模式。阻塞模式使用就像传统中的支持一样，比较简单，但是性能和可靠性都不好；非阻塞模式正好与之相反。对于低负载、低并发的应用程序，可以使用同步阻塞I/O来提升开发速率和更好的维护性；对于高负载、高并发的（网络）应用，应使用 NIO 的非阻塞模式来开发。

 <div align="center"> <img src="..\..\06-Network-Programming\images\nio\NIO.png" width="600px"></div>

NIO**主要有 Channel（通道） , Selector（选择器），Buffer（缓冲区）三大核心组件**（整个NIO体系包含的类远远不止这三个，只能说这三个是NIO体系的“核心API”）。

- 缓冲区 Buffer ：NIO 的数据操作都是在 Buffer 中进行的。Buffer 实际上是一个数组。Buffer 最常见的类型是`ByteBuffer`，另外还有`CharBuffer/ShortBuffer/IntBuffer/LongBuffer/FloatBuffer/DoubleBuffer`等。
- 通道 Channel ：Channel 是一种 IO 操作的连接（nexus，连接的意思），它代表的是到实体的开放连接，这个实体可以是硬件设备、文件、网络套接字或者可执行 IO 操作（比如读、写）的程序组件。
  - **通道是双向的也是有前提的**，那就是通道基于随机访问文件`RandomAccessFile`的可读可写文件指针。NIO可以通过 Channel 进行数据的读、写和同时读写操作。
  - 基本的通道类型有：`FileChannel/DatagramChannel/SocketChannel/ServerSocketChannel`等。FileChannel 是基于文件的通道，`SocketChannel` 和 `ServerSocketChannel` 用于网络 TCP 套接字数据报读写，`DatagramChannel` 是用于网络 UDP 套接字数据报读写。
  - 通道不能单独存在，它**必须绑定一个缓存区**，所有的数据只会存在于缓存区中，无论你是写或是读，必然是缓存区通过通道到达磁盘文件，或是磁盘文件通过通道到达缓存区。
- 多路复用器 Selector ：NIO 编程的基础，是一个多路复用器。 Selector 会不断地轮询注册在其上的通道（Channel），如果某个通道处于就绪状态，会被 Selector 轮询出来，然后通过 `SelectionKey` 可以取得就绪的Channel集合，从而进行后续的 IO 操作。服务器端只要提供一个线程负责 `Selector` 的轮询，就可以接入成千上万个客户端，这就是 JDK NIO 库的巨大进步。

**3. AIO（Asynchronous I/O）**

AIO（Asynchronous I/O），异步 IO，又称为 NIO2，也是放在 `java.nio` 包下，JDK1.7 引入。它是一种**非阻塞** + **异步**的通信模式。在 NIO 的基础上，引入了新的异步通道的概念，并提供了异步文件通道和异步套接字通道的实现。

AIO 并没有采用 NIO 的多路复用器，而是使用异步通道的概念。其 read，write 方法的返回类型，都是 Future 对象。而 Future 模型是异步的，其核心思想是：**去主函数等待时间**。

AIO 模型中通过 `AsynchronousSocketChannel` 和 `AsynchronousServerSocketChannel` 实现套接字通道的通信。非阻塞，异步。

## BIO、NIO 有什么区别？

**阻塞 VS 非阻塞**：BIO流是阻塞的，NIO流是不阻塞的。

**流 VS 缓冲区**：BIO 面向流(Stream oriented)，而 NIO 面向缓冲区(Buffer oriented)。

**单向 VS 双向**：BIO流的读写只能单向的，NIO 可通过Channel（通道） 进行双向读写。

**阻塞 VS 多路复用**：NIO有选择器，而BIO没有。

- BIO：一个连接一个线程，客户端有连接请求时服务器端就需要启动一个线程进行处理。所以，线程开销大。可改良为用线程池的方式代替新创建线程，被称为伪异步 IO 。
- NIO：一个请求一个线程，但客户端发送的连接请求都会注册到多路复用器（Selector ）上，多路复用器轮询到连接有新的 I/O 请求时，才启动一个线程进行处理。可改良为一个线程处理多个请求，基于多 Reactor 模型。

## BIO、NIO、AIO 适用场景?

| 对比     | **BIO**  | **NIO**                | **AIO**    |
| -------- | -------- | ---------------------- | ---------- |
| IO 模型  | 同步阻塞 | 同步非阻塞（多路复用） | 异步非阻塞 |
| 编程难度 | 简单     | 复杂                   | 复杂       |
| 可靠性   | 差       | 好                     | 好         |
| 吞吐量   | 低       | 高                     | 高         |

* BIO方式适用于**连接数目比较小且固定**的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK 1.4以前的唯一选择，但程序简单易理解。
* NIO方式适用于**连接数目多且连接比较短**（轻操作）的架构，比如聊天服务器，弹幕系统，服务器间通讯等。编程比较复杂，JDK 1.4开始支持。
* AIO方式使用于**连接数目多且连接比较长**（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，JDK 1.7开始支持。

# Netty基础

## 什么是 Netty ？

Netty 是由 JBOSS 提供的一个 Java 开源框架， 现为 Github 上的独立项目，它**是一个异步的、 基于事件驱动的网络应用框架， 用以快速开发高性能、 高可靠性的网络 IO 程序**。Netty 主要针对在 TCP 协议下， 面向 Clients 端的高并发应用， 或者 Peer-to-Peer 场景下的大量数据持续传输的应用。Netty 本质是一个 NIO 框架， 适用于服务器通讯相关的多种应用场景。

| 分类     | Netty特性                                                                                                                           |
| -------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| 设计     | 统一的 API，支持多种传输类型，阻塞的和非阻塞的<br/>简单而强大的线程模型<br/>真正的无连接数据报套接字支持<br/>链接逻辑组件以支持复用 |
| 易于使用 | 详实的Javadoc和大量的示例集<br/>不需要超过JDK 1.6+的依赖（一些可选的特性可能需要Java 1.7+和/或额外的依赖）                          |
| 性能     | 拥有比 Java 的核心 API 更高的吞吐量以及更低的延迟<br/>得益于池化和复用，拥有更低的资源消耗<br/>最少的内存复制                       |
| 健壮性   | 不会因为慢速、快速或者超载的连接而导致 OutOfMemoryError <br/>消除在高速网络中 NIO 应用程序常见的不公平读/写比率                     |
| 安全性   | 完整的 SSL/TLS 以及 StartTLS 支持<br/>可用于受限环境下，如 Applet 和 OSGI                                                           |
| 社区驱动 | 发布快速而且频繁                                                                                                                    |

Netty 的分层很清晰：

- 核心层，主要定义一些基础设施，比如可扩展事件模型、通用通信 API、支持零拷贝的ByteBuffer缓冲区。
- 传输服务层，主要定义一些通信的底层能力，或者说是传输协议的支持，比如 TCP、UDP、HTTP 隧道、虚拟机管道等。
- 协议支持层，支持 HTTP、Protobuf、二进制、文本、WebSocket等一系列常见协议， 还支持通过实行编码解码逻辑来实现自定义协议。

 <div align="center"> <img src="..\..\06-Network-Programming\images\netty\netty框架.png" width="500px"></div>

## 为什么选择 Netty ？

- **使用简单**：API 使用简单，开发门槛低。
- **功能强大**：预置了多种编解码功能，支持多种主流协议。
- **定制能力强**：可以通过 ChannelHandler 对通信框架进行灵活的扩展。
- **性能高**：通过与其它业界主流的 NIO 框架对比，Netty 的综合性能最优。
- **成熟稳定**：Netty 修复了已经发现的所有 JDK NIO BUG，业务开发人员不需要再为 NIO 的 BUG 而烦恼。
- **社区活跃**：版本迭代周期短，发现的BUG可以被及时修复，同时，更多的新功能会被加入。
- **案例丰富**：经历了大规模的商业应用考验，质量已经得到验证。在互联网、大数据、网络游戏、企业应用、电信软件等众多行业得到成功商用，证明了它可以完全满足不同行业的商业应用。

## 为什么说 Netty 使用简单？

我们假设要搭建一个 Server 服务器，使用 **Java NIO 的步骤**如下：

1. 创建 ServerSocketChannel ，绑定监听端口，并配置为非阻塞模式。

2. 创建 Selector，将之前创建的 ServerSocketChannel 注册到 Selector 上，监听 SelectionKey.OP_ACCEPT 。循环执行 Selector#select() 方法，轮询就绪的 Channel。	

3. 轮询就绪的 Channel 时，如果是处于 OP_ACCEPT 状态，说明是新的客户端接入，调用`ServerSocketChannel#accept()` 方法，接收新的客户端。设置新接入的 SocketChannel 为非阻塞模式，并注册到 Selector 上，监听 OP_READ 。

4. 如果轮询的 Channel 状态是 OP_READ ，说明有新的就绪数据包需要读取，则构造 ByteBuffer 对象，读取数据。这里，解码数据包的过程，需要我们自己编写。

使用 **Netty 的步骤**如下：

1. 创建 NIO 线程组 EventLoopGroup 和 ServerBootstrap。
   - 设置 ServerBootstrap 的属性：线程组、SO_BACKLOG 选项，设置 NioServerSocketChannel 为 Channel
   - 设置业务处理 Handler 和 编解码器 Codec 。
   - 绑定端口，启动服务器程序。
2. 在业务处理 Handler 中，处理客户端发送的数据，并给出响应。

那么相比 Java NIO，使用 Netty 开发程序，都**简化了哪些步骤**呢？

1. 无需关心 `OP_ACCEPT`、`OP_READ`、`OP_WRITE` 等等 **IO 操作**，Netty 已经封装，对我们在使用是透明无感的。
2. 使用 boss 和 worker EventLoopGroup ，Netty 直接提供**多 Reactor 多线程模型**。
3. 在 Netty 中，我们看到有使用一个解码器 FixedLengthFrameDecoder，可以用于处理定长消息的问题，能够解决 **TCP 粘包拆包**问题，十分方便。如果使用 Java NIO ，需要我们自行实现解码器。

## 说说业务中 Netty 的使用场景？

- 构建高性能、低时延的各种 Java 中间件，Netty 主要作为基础通信框架提供高性能、低时延的通信服务。例如：
  - RocketMQ ，分布式消息队列。
  - Dubbo ，服务调用框架。
  - Spring WebFlux ，基于响应式的 Web 框架。
  - HDFS ，分布式文件系统。
- 公有或者私有协议栈的基础通信框架，例如可以基于 Netty 构建异步、高性能的 WebSocket、Protobuf 等协议的支持。
- 各领域应用，例如大数据、游戏等，Netty 作为高性能的通信框架用于内部各模块的数据分发、传输和汇总等，实现模块之间高性能通信。

## 说说 Netty 如何实现高性能？

1. **线程模型** ：更加优雅的 Reactor 模式实现、灵活的线程模型、利用 EventLoop 等创新性的机制，可以非常高效地管理成百上千的 Channel 。

2. **内存池设计** ：使用池化的 Direct Buffer 等技术，在提高 IO 性能的同时，减少了对象的创建和销毁。并且，内吃吃的内部实现是用一颗二叉查找树，更好的管理内存分配情况。

3. **内存零拷贝** ：使用 Direct Buffer ，可以使用 Zero-Copy 机制。

   > Zero-Copy ，在操作数据时，不需要将数据 Buffer 从一个内存区域拷贝到另一个内存区域。因为少了一次内存的拷贝，因此 CPU 的效率就得到的提升。

4. **协议支持** ：提供对 Protobuf 等高性能序列化协议支持。

5. 使用更多本地代码

   - 直接利用 JNI 调用 Open SSL 等方式，获得比 Java 内建 SSL 引擎更好的性能。
   - 利用 JNI 提供了 Native Socket Transport ，在使用 Epoll edge-triggered 的情况下，可以有一定的性能提升。
   
6. 其它：

   - 利用反射等技术直接操纵 SelectionKey ，使用数组而不是 Java 容器等。
   - 实现 [FastThreadLocal](https://segmentfault.com/a/1190000012926809) 类，当请求频繁时，带来更好的性能。
   - ….

> 下面三连问！
>
> Netty 是一个高性能的、高可靠的、可扩展的异步通信框架，那么高性能、高可靠、可扩展设计体现在哪里呢？

## Netty 的高性能如何体现？

性能是设计出来的，而不是测试出来的。那么，Netty 的架构设计是如何实现高性能的呢？

1. **线程模型** ：采用异步非阻塞的 I/O 类库，基于 Reactor 模式实现，解决了传统同步阻塞 I/O 模式下服务端无法平滑处理客户端线性增长的问题。

2. **堆外内存** ：TCP 接收和发送缓冲区采用直接内存代替堆内存，避免了内存复制，提升了 I/O 读取和写入性能。

3. **内存池设计** ：支持通过内存池的方式循环利用 ByteBuf，避免了频繁创建和销毁 ByteBuf 带来的性能消耗。

4. **参数配置** ：可配置的 I/O 线程数目和 TCP 参数等，为不同用户提供定制化的调优参数，满足不同的性能场景。

5. **队列优化** ：采用环形数组缓冲区，实现无锁化并发编程，代替传统的线程安全容器或锁。

6. **并发能力** ：合理使用线程安全容器、原子类等，提升系统的并发能力。

7. **降低锁竞争** ：关键资源的使用采用单线程串行化的方式，避免多线程并发访问带来的锁竞争和额外的 CPU 资源消耗问题。

8. 内存泄露检测：通过引用计数器及时地释放不再被引用的对象，细粒度的内存管理降低了 GC 的频率，减少频繁 GC 带来的时延增大和 CPU 损耗。


## Netty 的可用性如何体现？

1. 链路有效性检测：由于长连接不需要每次发送消息都创建链路，也不需要在消息完成交互时关闭链路，因此相对于短连接性能更高。为了保证长连接的链路有效性，往往需要通过心跳机制周期性地进行链路检测。使用心跳机制的原因是，避免在系统空闲时因网络闪断而断开连接，之后又遇到海量业务冲击导致消息积压无法处理。为了解决这个问题，需要周期性地对链路进行有效性检测，一旦发现问题，可以及时关闭链路，重建 TCP 连接。为了支持心跳，Netty 提供了两种链路空闲检测机制：

   - 读空闲超时机制：连续 T 周期没有消息可读时，发送心跳消息，进行链路检测。如果连续 N 个周期没有读取到心跳消息，可以主动关闭链路，重建连接。
   - 写空闲超时机制：连续 T 周期没有消息需要发送时，发送心跳消息，进行链路检测。如果连续 N 个周期没有读取对方发回的心跳消息，可以主动关闭链路，重建连接。
   
2. 内存保护机制：Netty 提供多种机制对内存进行保护，包括以下几个方面：

   - 通过对象引用计数器对 ByteBuf 进行细粒度的内存申请和释放，对非法的对象引用进行检测和保护。
   - 可设置的内存容量上限，包括 ByteBuf、线程池线程数等，避免异常请求耗光内存。
   
3. 优雅停机：优雅停机功能指的是当系统推出时，JVM 通过注册的 Shutdown Hook 拦截到退出信号量，然后执行推出操作，释放相关模块的资源占用，将缓冲区的消息处理完成或清空，将待刷新的数据持久化到磁盘和数据库中，等到资源回收和缓冲区消息处理完成之后，再退出。


## Netty 的可扩展如何体现？

可定制、易扩展。

- **责任链模式** ：ChannelPipeline 基于责任链模式开发，便于业务逻辑的拦截、定制和扩展。
- **基于接口的开发** ：关键的类库都提供了接口或抽象类，便于用户自定义实现。
- **提供大量的工厂类** ：通过重载这些工厂类，可以按需创建出用户需要的对象。
- **提供大量系统参数** ：供用户按需设置，增强系统的场景定制性。

# Netty模型架构

## 简单介绍 Netty 的核心组件？

Netty 有如下核心组件：

- Bootstrap&ServerBootstrap（启动引导类）
- EventLoop（事件循环）
- ByteBuf（字节容器）
- Channel（网络操作抽象类）
- ChannelHandler（消息处理器）
- ChannelFuture（操作执行结果）
- ChannelPipeline（对象链表）

详细直接阅读 [《Netty模型架构》](https://github.com/zgkaii/CS-Study-Notes/blob/master/06-Network-Programming/Netty/01-Netty%E6%A8%A1%E5%9E%8B%E6%9E%B6%E6%9E%84.md#12-%E6%A0%B8%E5%BF%83%E7%BB%84%E4%BB%B6) 一文。

## 说说 Netty 的逻辑架构？

Netty 采用了典型的**三层网络架构**进行设计和开发，其逻辑架构如下图所示：

![Netty 逻辑架构图](http://static.iocoder.cn/85d98e3cb0e6e39f80d02234e039a4dd)

1. **Reactor 通信调度层**：由一系列辅助类组成，包括 Reactor 线程 NioEventLoop 及其父类，NioSocketChannel 和 NioServerSocketChannel 等等。该层的职责就是监听网络的读写和连接操作，负责将网络层的数据读到内存缓冲区，然后触发各自网络事件，例如连接创建、连接激活、读事件、写事件等。将这些事件触发到 pipeline 中，由 pipeline 管理的职责链来进行后续的处理。
2. **职责链 ChannelPipeline**：负责事件在职责链中的有序传播，以及负责动态地编排职责链。职责链可以选择监听和处理自己关心的事件，拦截处理和向后传播事件。
3. **业务逻辑编排层**：业务逻辑编排层通常有两类，一类是纯粹的业务逻辑编排，一类是应用层协议插件，用于特定协议相关的会话和链路管理。由于应用层协议栈往往是开发一次到处运行，并且变动较小，故而将应用协议到 POJO 的转变和上层业务放到不同的 ChannelHandler 中，就可以实现协议层和业务逻辑层的隔离，实现架构层面的分层隔离。

## 什么是 Reactor 模型？

通常，我们设计一个事件处理模型的程序有两种思路。

- 轮询方式线程不断轮询访问相关事件发生源有没有发生事件，有发生事件就调用事件处理逻辑。
- 事件驱动方式发生事件，主线程把事件放入事件队列，在另外线程不断循环消费事件列表中的事件，调用事件对应的处理逻辑处理事件。事件驱动方式也被称为消息通知方式，其实是设计模式中**观察者模式**的思路。

以GUI的逻辑处理为例，说明两种逻辑的不同：

- 轮询方式 线程不断轮询是否发生按钮点击事件，如果发生，调用处理逻辑。
- 事件驱动方式 发生点击事件把事件放入事件队列，在另外线程消费的事件列表中的事件，根据事件类型调用相关事件处理逻辑。

 <div align="center"> <img src="..\..\06-Network-Programming\images\netty\事件驱动模型.png" width="700px"></div>

主要包括4个基本组件：

- 事件队列（event queue）：接收事件的入口，存储待处理事件
- 分发器（event mediator）：将不同的事件分发到不同的业务逻辑单元
- 事件通道（event channel）：分发器与处理器之间的联系渠道
- 事件处理器（event processor）：实现业务逻辑，处理完成后会发出事件，触发下一步操作

可以看出，相对传统轮询模式，事件驱动有如下优点：

- 可扩展性好，分布式的异步架构，事件处理器之间高度解耦，可以方便扩展事件处理逻辑
- 高性能，基于队列暂存事件，能方便并行异步处理事件

## 请介绍 Netty 的线程模型？

Reactor是反应堆的意思，Reactor模型，是指通过一个或多个输入同时传递给服务处理器的服务请求的**事件驱动处理模式**。 服务端程序处理传入多路请求，并将它们同步分派给请求对应的处理线程，Reactor模式也叫Dispatcher模式，即I/O多了复用统一监听事件，收到事件后分发(Dispatch给某进程)，是编写高性能网络服务器的必备技术之一。

Reactor模型中有2个关键组成：

- Reactor Reactor在一个单独的线程中运行，负责监听和分发事件，分发给适当的处理程序来对IO事件做出反应。 它就像公司的电话接线员，它接听来自客户的电话并将线路转移到适当的联系人
- Handlers 处理程序执行I/O事件要完成的实际事件，类似于客户想要与之交谈的公司中的实际官员。Reactor通过调度适当的处理程序来响应I/O事件，处理程序执行非阻塞操作

 <div align="center"> <img src="..\..\06-Network-Programming\images\netty\Reactor模型.png" width="600px"></div>

取决于Reactor的数量和Hanndler线程数量的不同，Reactor模型有3个变种

- 单Reactor单线程
- 单Reactor多线程
- 主从Reactor多线程

可以这样理解，Reactor就是一个执行`while (true) { selector.select(); ...}`循环的线程，会源源不断的产生新的事件，称作反应堆很贴切。

### Reacto单线程

所有的IO操作都由同一个NIO线程处理。

 <div align="center"> <img src="..\..\06-Network-Programming\images\netty\单线程.png" width="600px"></div>

 <div align="center"> <img src="..\..\06-Network-Programming\images\netty\单线程1.png" width="600px"></div>

**单线程 Reactor 的优点是对系统资源消耗特别小，但是，没办法支撑大量请求的应用场景并且处理请求的时间可能非常慢，毕竟只有一个线程在工作嘛！所以，一般实际项目中不会使用单线程Reactor 。**

为了解决这些问题，演进出了Reactor多线程模型。

### Reacto多线程

一个线程负责接受请求,一组NIO线程处理IO操作。

 <div align="center"> <img src="..\..\06-Network-Programming\images\netty\多线程.png" width="600px"></div>

 <div align="center"> <img src="..\..\06-Network-Programming\images\netty\多线程1.png" width="600px"></div>

**大部分场景下多线程Reactor模型是没有问题的，但是在一些并发连接数比较多（如百万并发）的场景下，一个线程负责接受客户端请求就存在性能问题了。**

为了解决这些问题，演进出了主从多线程Reactor模型。

### Reacto主从多线程

一组NIO线程负责接受请求，一组NIO线程处理IO操作。

 <div align="center"> <img src="..\..\06-Network-Programming\images\netty\主从模型.png" width="600px"></div>

 <div align="center"> <img src="..\..\06-Network-Programming\images\netty\主从模型1.png" width="600px"></div>

### Netty线程模型实现

大部分网络框架都是基于 Reactor 模式设计开发的。

> Reactor 模式基于事件驱动，采用多路复用将事件分发给相应的 Handler 处理，非常适合处理海量 IO 的场景。

在 Netty 主要靠 `NioEventLoopGroup` 线程池来实现具体的线程模型的 。

| 线程模式                  | 实现                                                                                                                                                                                                                                  |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Reacto单线程              | EventLoopGroup eventGroup = new NioEventLoopGroup(**1**); <br/><br/>ServerBootstrap serverBootstrap = new ServerBootstrap(); <br/>serverBootstrap.group(eventGroup);                                                                  |
| 非主从 Reactor 多线程模式 | EventLoopGroup eventGroup = new NioEventLoopGroup(); <br/><br/>ServerBootstrap serverBootstrap = new ServerBootstrap(); <br/>serverBootstrap.group(eventGroup);                                                                       |
| 主从 Reactor 多线程模式   | EventLoopGroup bossGroup = new NioEventLoopGroup(); <br/>EventLoopGroup workerGroup = new NioEventLoopGroup(); <br/><br/>ServerBootstrap serverBootstrap = new ServerBootstrap(); <br/>serverBootstrap.group(bossGroup, workerGroup); |

我们实现服务端的时候，一般会初始化两个线程组：

1. **`bossGroup`** ：接收连接。
2. **`workerGroup`** ：负责具体的处理，交由对应的 Handler 处理。

**主从多线程模型**

从一个 主线程 NIO 线程池中选择一个线程作为 Acceptor 线程，绑定监听端口，接收客户端连接的连接，其他线程负责后续的接入认证等工作。连接建立完成后，Sub NIO 线程池负责具体处理 I/O 读写。如果多线程模型无法满足你的需求的时候，可以考虑使用主从多线程模型 。

虽然Netty的线程模型基于主从Reactor多线程，借用了MainReactor和SubReactor的结构，但是实际实现上，SubReactor和Worker线程在同一个线程池中：

```java
// 1.bossGroup 用于接收连接，workerGroup 用于具体的处理
EventLoopGroup bossGroup = new NioEventLoopGroup();
EventLoopGroup workerGroup = new NioEventLoopGroup();
try {
  //2.创建服务端启动引导/辅助类：ServerBootstrap
  ServerBootstrap b = new ServerBootstrap();
  //3.给引导类配置两大线程组,确定了线程模型
  b.group(bossGroup, workerGroup)
    //......
```

上面代码中的bossGroup 和workerGroup是Bootstrap构造方法中传入的两个对象，这两个group均是线程池

- bossGroup线程池则只是在bind某个端口后，获得其中一个线程作为MainReactor，专门处理端口的accept事件，**每个端口对应一个boss线程**。
- workerGroup线程池会被各个SubReactor和worker线程充分利用。

## 什么是业务线程池？

无论是那种类型的 Reactor 模型，都需要在 Reactor 所在的线程中，进行读写操作。那么此时就会有一个问题，如果我们读取到数据，需要进行业务逻辑处理，并且这个业务逻辑需要对数据库、缓存等等进行操作，会有什么问题呢？假设这个数据库操作需要 5 ms ，那就意味着这个 Reactor 线程在这 5 ms 无法进行注册在这个 Reactor 的 Channel 进行读写操作。也就是说，多个 Channel 的所有读写操作都变成了串行。势必，这样的效率会非常非常非常的低。

那么怎么解决呢？创建业务线程池，将读取到的数据，提交到业务线程池中进行处理。这样，Reactor 的 Channel 就不会被阻塞，而 Channel 的所有读写操作都变成了并行了。

# Netty网络程序优化

## TCP 粘包 / 拆包的原因？

**什么是粘包/拆包？**

TCP 粘包/拆包 就是基于 TCP 发送数据的时候，出现了多个字符串“粘”在了一起或者一个字符串被“拆”开的问题。我们发送两条消息：ABC 和 DEF，可能一次就把两条消息接收完了，即 ABCDEF；也可能分成了好多次，比如 AB、CD 和 EF。

对方一次性接收了多条消息这种现象，我们就称之为**粘包现象**。

对方多次接收了不完整消息这种现象，我们就称之为**拆包（半包）现象**。

**为什么会有粘包/拆包？**

TCP 发送消息的时候是有缓冲区的，当**消息的内容远小于缓冲区**的时候，这条消息不会立马发送出去，而是跟其它的消息合并之后再发送出去，这样合并发送是明显能够提高效率的。如果发送的消息太大，已经**超过了缓冲区的大小**，这时候就**必须拆分发送**。

出现粘包的原因无非两种：

- 发送方每次写入数据 < 套接字缓冲区大小
- 接收方读取套接字缓冲区数据不够及时

出现拆包的原因有两种：

- 发送方写入数据 > 套接字缓冲区大小
- 发送的数据大于协议的 MTU（Maximum Transmission Unit，最大传输单元），必须拆包

**本质原因**

* **TCP 是流式协议，消息无边界**。（UDP 协议不会出现粘包 / 拆包现象，它的消息是有明确边界的。UDP 像邮寄的包裹，虽然一次运输多个，但每个包裹都有“界限”，一个一个签收， 所以无粘包、半包问题）。

## 应该这么解决粘包 / 拆包？

解决问题的根本手段就是：找出消息的边界。可以通过封装成帧（Framing）的三种方法：

| 方法          | 如何确定消息边界                   | 优点               | 缺点                                     | 推荐度     |
| :------------ | :--------------------------------- | :----------------- | :--------------------------------------- | :--------- |
| 定长法        | 使用固定长度分割消息               | 简单               | 空间浪费                                 | 不推荐     |
| 分割符法      | 使用固定分割符分割消息             | 简单               | 分割符本身需要转义，且需要扫描消息的内容 | 不特别推荐 |
| 长度 + 内容法 | 先获取消息的长度，再按长度读取内容 | 精确获取消息的内容 | 需要预先知道消息的最大长度               | 推荐+      |

> 其他方法：
>
> 1. TCP 连接改成短连接，一个请求一个短连接，那么建立连接到释放连接之 间的信息即为传输信息。效率低下，不推荐。
> 2. 例如 JSON 可以看{}是否应已经成对，所以说衡量实际场景，很多是对现有协议的支持。

**Netty 是如何支持的？**

Netty 是通过三组类来处理粘包 / 半包问题的，主要对应于上面提到的三种方式。

| 方法          | 编码                   | 解码                           |
| :------------ | :--------------------- | :----------------------------- |
| 定长法        | 简单                   | `FixedLengthFrameDecoder`      |
| 分割符法      | 简单                   | `DelimiterBasedFrameDecoder`   |
| 长度 + 内容法 | `LengthFieldPrepender` | `LengthFieldBasedFrameDecoder` |

* **`FixedLengthFrameDecoder`**: 定长协议解码器，我们可以指定固定的字节数算一个完整的 报文

- **`DelimiterBasedFrameDecoder`** : 分隔符解码器，分隔符可以自己指定。
- **`LineBasedFrameDecoder`** : 行分隔符解码器， 遇到`\n` 或者`\r\n`，则认为是一个完整的报文。
- **`LengthFieldBasedFrameDecoder`**：长度编码解码器，将报文划分为报文头/报文体。
- `JsonObjectDecoder`：json 格式解码器，当检 测到匹配数量的“{” 、”}”或”[””]”时，则认为是 一个完整的 json 对象或者 json 数组

> 之所以以`*FrameDecoder`结尾，那是因为被解码之后的消息又叫作一帧一帧的消息，所以称为 “帧解码器”。 

## 了解哪几种序列化协议？

**序列化域反序列化**

- 序列化（编码），是将对象序列化为二进制形式（字节数组），主要用于网络传输、数据持久化等。
- 反序列化（解码），则是将从网络、磁盘等读取的字节数组还原成原始对象，主要用于网络传输对象的解码，以便完成远程调用。

在选择序列化协议的选择，主要考虑以下三个因素：

- 序列化后的**字节大小**。更少的字节数，可以减少网络带宽、磁盘的占用。
- 序列化的**性能**。对 CPU、内存资源占用情况。
- 是否支持**跨语言**。例如，异构系统的对接和开发语言切换。

**方案选择**

* **Java 默认提供的序列化**：无法跨语言；序列化后的字节大小太大；序列化的性能差。
* **XML** 
  * 优点：人机可读性好，可指定元素或特性的名称。
  * 缺点：序列化数据只包含数据本身以及类的结构，不包括类型标识和程序集信息；只能序列化公共属性和字段；不能序列化方法；文件庞大，文件格式复杂，传输占带宽。
  * 适用场景：当做配置文件存储数据，实时数据转换。
* **JSON** ，是一种轻量级的数据交换格式。
  * 优点：兼容性高、数据格式比较简单，易于读写、序列化后数据较小，可扩展性好，兼容性好。与 XML 相比，其协议比较简单，解析速度比较快。
  * 缺点：数据的描述性比 XML 差、不适合性能要求为 ms 级别的情况、额外空间开销比较大。
  * 适用场景（可替代 XML ）：跨防火墙访问、可调式性要求高、基于Restful API 请求、传输数据量相对小，实时性要求相对低（例如秒级别）的服务。
* Thrift ，不仅是序列化协议，还是一个 RPC 框架。
  * 优点：序列化后的体积小, 速度快、支持多种语言和丰富的数据类型、对于数据字段的增删具有较强的兼容性、支持二进制压缩编码。
  * 缺点：使用者较少、跨防火墙访问时，不安全、不具有可读性，调试代码时相对困难、不能与其他传输层协议共同使用（例如 HTTP）、无法支持向持久层直接读写数据，即不适合做数据持久化序列化协议。
  * 适用场景：分布式系统的 RPC 解决方案。

* Avro ，Hadoop 的一个子项目，解决了JSON的冗长和没有IDL的问题。
  * 优点：支持丰富的数据类型、简单的动态语言结合功能、具有自我描述属性、提高了数据解析速度、快速可压缩的二进制数据形式、可以实现远程过程调用 RPC、支持跨编程语言实现。
  * 缺点：对于习惯于静态类型语言的用户不直观。
  * 适用场景：在 Hadoop 中做 Hive、Pig 和 MapReduce 的持久化数据格式。

* **Protobuf** ，将数据结构以 `.proto` 文件进行描述，通过代码生成工具可以生成对应数据结构的 POJO 对象和 Protobuf 相关的方法和属性。
  * 优点：序列化后码流小，性能高、结构化数据存储格式（XML JSON等）、通过标识字段的顺序，可以实现协议的前向兼容、结构化的文档更容易管理和维护。
  * 缺点：需要依赖于工具生成代码、支持的语言相对较少，官方只支持Java 、C++、python。
  * 适用场景：对性能要求高的 RPC 调用、具有良好的跨防火墙的访问属性、适合应用层对象的持久化。

* [**Hessian**](https://www.oschina.net/p/hessian) ，采用二进制协议的轻量级 remoting on http 服务。
  * 目前，阿里 RPC 框架 Dubbo 的**默认**序列化协议。

## Netty 的零拷贝实现？

在 OS 层面上的 `Zero-copy` 通常指避免在 `用户态(User-space)` 与 `内核态(Kernel-space)` 之间来回拷贝数据，可以提升 CPU 的利用率。

而在 Netty 层面 ，零拷贝主要体现在对于数据操作的优化。

1. Netty 的接收和发送 `ByteBuffer` 采用堆外直接内存Direct Buffer。
2. 使用 Netty 提供的 `CompositeByteBuf` 类，可以将多个`ByteBuf` 合并为一个逻辑上的 `ByteBuf`，避免了各个 `ByteBuf` 之间的拷贝。
3. `ByteBuf` 支持 `slice` 操作，因此可以将 `ByteBuf` 分解为多个共享同一个存储区域的 `ByteBuf`，避免了内存的拷贝。
4. 通过 wrap 操作，我们可以将 byte[] 数组、`ByteBuf`、`ByteBuffer` 等包装成一个 `Netty ByteBuf` 对象, 进而避免拷贝操作。
5. 通过 `FileRegion` 包装的`FileChannel.tranferTo()` 实现文件传输，可以直接将文件缓冲区的数据发送到目标 `Channel`，避免了传统通过循环 write 方式导致的内存拷贝问题。

## 原生的 NIO 存在 Epoll Bug 是什么？Netty 是怎么解决的？

 **Java NIO Epoll BUG**

Java NIO Epoll 会导致 Selector 空轮询，最终导致 CPU 100% 。

官方声称在 JDK 1.6 版本的 update18 修复了该问题，但是直到 JDK 1.7 版本该问题仍旧存在，只不过该 BUG 发生概率降低了一些而已，它并没有得到根本性解决。

**Netty 解决方案**

对 Selector 的 select 操作周期进行**统计**，每完成一次**空**的 select 操作进行一次计数，若在某个周期内连续发生 N 次空轮询，则判断触发了 Epoll 死循环 Bug 。

> 艿艿：此处**空**的 select 操作的定义是，select 操作执行了 0 毫秒。

此时，Netty **重建** Selector 来解决。判断是否是其他线程发起的重建请求，若不是则将原 SocketChannel 从旧的 Selector 上取消注册，然后重新注册到新的 Selector 上，最后将原来的 Selector 关闭。

## Netty 长连接、心跳机制了解么？

**TCP 长连接和短连接了解么？**

我们知道 TCP 在进行读写之前，server 与 client 之间必须提前建立一个连接。建立连接的过程，需要我们常说的三次握手，释放/关闭连接的话需要四次挥手。这个过程是比较消耗网络资源并且有时间延迟的。

所谓，短连接说的就是 server 端 与 client 端建立连接之后，读写完成之后就关闭掉连接，如果下一次再要互相发送消息，就要重新连接。短连接的有点很明显，就是管理和实现都比较简单，缺点也很明显，每一次的读写都要建立连接必然会带来大量网络资源的消耗，并且连接的建立也需要耗费时间。

长连接说的就是 client 向 server 双方建立连接之后，即使 client 与 server 完成一次读写，它们之间的连接并不会主动关闭，后续的读写操作会继续使用这个连接。长连接的可以省去较多的 TCP 建立和关闭的操作，降低对网络资源的依赖，节约时间。对于频繁请求资源的客户来说，非常适用长连接。

**为什么需要心跳机制？Netty 中心跳机制了解么？**

在 TCP 保持长连接的过程中，可能会出现断网等网络异常出现，异常发生的时候， client 与 server 之间如果没有交互的话，它们是无法发现对方已经掉线的。为了解决这个问题, 我们就需要引入 **心跳机制** 。

心跳机制的工作原理是: 在 client 与 server 之间在一定时间内没有数据交互时, 即处于 idle 状态时, 客户端或服务器就会发送一个特殊的数据包给对方, 当接收方收到这个数据报文后, 也立即发送一个特殊的数据报文, 回应发送方, 此即一个 PING-PONG 交互。所以, 当某一端收到心跳消息后, 就知道了对方仍然在线, 这就确保 TCP 连接的有效性.

TCP 实际上自带的就有长连接选项，本身是也有心跳包机制，也就是 TCP 的选项：`SO_KEEPALIVE`。 但是，TCP 协议层面的长连接灵活性不够。所以，一般情况下我们都是在应用层协议上实现自定义心跳机制的，也就是在 Netty 层面通过编码实现。通过 Netty 实现心跳机制的话，核心类是 `IdleStateHandler` 。

## 什么是 Netty 空闲检测？

在 Netty 中，提供了 IdleStateHandler 类，正如其名，空闲状态处理器，用于检测连接的读写是否处于空闲状态。如果是，则会触发 IdleStateEvent 。

IdleStateHandler 目前提供三种类型的心跳检测，通过构造方法来设置。代码如下：

```java
// IdleStateHandler.java

public IdleStateHandler(
        int readerIdleTimeSeconds,
        int writerIdleTimeSeconds,
        int allIdleTimeSeconds) {
    this(readerIdleTimeSeconds, writerIdleTimeSeconds, allIdleTimeSeconds,
         TimeUnit.SECONDS);
}
```

- `readerIdleTimeSeconds` 参数：为读超时时间，即测试端一定时间内未接受到被测试端消息。
- `writerIdleTimeSeconds` 参数：为写超时时间，即测试端一定时间内向被测试端发送消息。
- `allIdleTimeSeconds` 参数：为读或写超时时间。

------

另外，我们会在网络上看到类似《IdleStateHandler 心跳机制》这样标题的文章，实际上空闲检测和心跳机制是**两件事**。

- 只是说，因为我们使用 IdleStateHandler 的目的，就是检测到连接处于空闲，通过心跳判断其是否还是**有效的连接**。
- 虽然说，TCP 协议层提供了 Keeplive 机制，但是该机制默认的心跳时间是 2 小时，依赖操作系统实现不够灵活。因而，我们才在应用层上，自己实现心跳机制。

## Netty 如何实现重连？

- 客户端，通过 IdleStateHandler 实现定时检测是否空闲，例如说 15 秒。
  - 如果空闲，则向服务端发起心跳。
  - 如果多次心跳失败，则关闭和服务端的连接，然后重新发起连接。
- 服务端，通过 IdleStateHandler 实现定时检测客户端是否空闲，例如说 90 秒。
  - 如果检测到空闲，则关闭客户端。
  - 注意，如果接收到客户端的心跳请求，要反馈一个心跳响应给客户端。通过这样的方式，使客户端知道自己心跳成功。

如下艿艿在自己的 [TaroRPC](https://github.com/YunaiV/TaroRPC) 中提供的一个示例：

- [NettyClient.java](https://github.com/YunaiV/TaroRPC/blob/master/transport/transport-netty4/src/main/java/cn/iocoder/taro/transport/netty4/NettyClient.java) 中，设置 IdleStateHandler 和 ClientHeartbeatHandler。核心代码如下：

  ```java
  // NettyHandler.java
  
  .addLast("idleState", new IdleStateHandler(TaroConstants.TRANSPORT_CLIENT_IDLE,
                                             TaroConstants.TRANSPORT_CLIENT_IDLE, 0,
                                             TimeUnit.MILLISECONDS))
      .addLast("heartbeat", new ClientHeartbeatHandler())
  ```

- [NettyServer.java](https://github.com/YunaiV/TaroRPC/blob/master/transport/transport-netty4/src/main/java/cn/iocoder/taro/transport/netty4/NettyServer.java) 中，设置 IdleStateHandler 和 ServerHeartbeatHandler。核心代码如下：

  ```java
  // NettyServer.java
  
  .addLast("idleState", new IdleStateHandler(0, 0, TaroConstants.TRANSPORT_SERVER_IDLE,
                                             TimeUnit.MILLISECONDS))
      .addLast("heartbeat", new ServerHeartbeatHandler())
  ```

- [ClientHeartbeatHandler.java](https://github.com/YunaiV/TaroRPC/blob/master/transport/transport-netty4/src/main/java/cn/iocoder/taro/transport/netty4/heartbeat/ClientHeartbeatHandler.java) 中，碰到空闲，则发起心跳。不过，如何重连，暂时没有实现。需要考虑，重新发起连接可能会失败的情况。具体的，可以看看 [《一起学Netty（十四）之 Netty生产级的心跳和重连机制》](https://blog.csdn.net/linuu/article/details/51509847) 文章中的，ConnectionWatchdog 的代码。

- [ServerHeartbeatHandler.java](https://github.com/YunaiV/TaroRPC/blob/6ce2af911ccec9ed5dc75c7f3ebda9c758272f3b/transport/transport-netty4/src/main/java/cn/iocoder/taro/transport/netty4/heartbeat/ServerHeartbeatHandler.java) 中，检测到客户端空闲，则直接关闭连接。

## Netty 自己实现的 ByteBuf 有什么优点？

如下是 [《Netty 实战》](http://svip.iocoder.cn/Netty/ByteBuf-1-1-ByteBuf-intro/#) 对它的**优点总**结：

 - 它可以被用户自定义的**缓冲区类型**扩展
 - 通过内置的符合缓冲区类型实现了透明的**零拷贝**
 - 容量可以**按需增长**
 - 在读和写这两种模式之间切换不需要调用 `#flip()` 方法
 - 读和写使用了**不同的索引**
 - 支持方法的**链式**调用
 - 支持引用计数
 - 支持**池化**

## Netty 为什么要实现内存管理？

在 Netty 中，IO 读写必定是非常频繁的操作，而考虑到更高效的网络传输性能，Direct ByteBuffer 必然是最合适的选择。但是 Direct ByteBuffer 的申请和释放是高成本的操作，那么进行**池化**管理，多次重用是比较有效的方式。但是，不同于一般于我们常见的对象池、连接池等**池化**的案例，ByteBuffer 是有**大小**一说。又但是，申请多大的 Direct ByteBuffer 进行池化又会是一个大问题，太大会浪费内存，太小又会出现频繁的扩容和内存复制！！！所以呢，就需要有一个合适的内存管理算法，解决**高效分配内存**的同时又解决**内存碎片化**的问题。

 **官方的说法**

> FROM [《Netty 学习笔记 —— Pooled buffer》](https://skyao.gitbooks.io/learning-netty/content/buffer/pooled_buffer.html)
>
> Netty 4.x 增加了 Pooled Buffer，实现了高性能的 buffer 池，分配策略则是结合了 buddy allocation 和 slab allocation 的 **jemalloc** 变种，代码在`io.netty.buffer.PoolArena` 中。
>
> 官方说提供了以下优势：
>
> - 频繁分配、释放 buffer 时减少了 GC 压力。
> - 在初始化新 buffer 时减少内存带宽消耗( 初始化时不可避免的要给buffer数组赋初始值 )。
> - 及时的释放 direct buffer 。

**hushi55 大佬的理解**

> > C/C++ 和 java 中有个围城，城里的想出来，城外的想进去！**
>
> 这个围城就是自动内存管理！
>
> **Netty 4 buffer 介绍**
>
> Netty4 带来一个与众不同的特点是其 ByteBuf 的实现，相比之下，通过维护两个独立的读写指针， 要比 `io.netty.buffer.ByteBuf` 简单不少，也会更高效一些。不过，Netty 的 ByteBuf 带给我们的最大不同，就是他不再基于传统 JVM 的 GC 模式，相反，它采用了类似于 C++ 中的 malloc/free 的机制，需要开发人员来手动管理回收与释放。从手动内存管理上升到GC，是一个历史的巨大进步， 不过，在20年后，居然有曲线的回归到了手动内存管理模式，正印证了马克思哲学观： **社会总是在螺旋式前进的，没有永远的最好。**
>
> **① GC 内存管理分析**
>
> 的确，就内存管理而言，GC带给我们的价值是不言而喻的，不仅大大的降低了程序员的心智包袱， 而且，也极大的减少了内存管理带来的 Crash 困扰，为函数式编程（大量的临时对象）、脚本语言编程带来了春天。 并且，高效的GC算法也让大部分情况下程序可以有更高的执行效率。 不过，也有很多的情况，可能是手工内存管理更为合适的。譬如：
>
> - 对于类似于业务逻辑相对简单，譬如网络路由转发型应用（很多erlang应用其实是这种类型）， 但是 QPS 非常高，比如1M级，在这种情况下，在每次处理中即便产生1K的垃圾，都会导致频繁的GC产生。 在这种模式下，erlang 的按进程回收模式，或者是 C/C++ 的手工回收机制，效率更高。
> - Cache 型应用，由于对象的存在周期太长，GC 基本上就变得没有价值。
>
> 所以，理论上，尴尬的GC实际上比较适合于处理介于这 2 者之间的情况： 对象分配的频繁程度相比数据处理的时间要少得多的，但又是相对短暂的， 典型的，对于OLTP型的服务，处理能力在 1K QPS 量级，每个请求的对象分配在 10K-50K 量级， 能够在 5-10s 的时间内进行一 次younger GC ，每次GC的时间可以控制在 10ms 水平上， 这类的应用，实在是太适合 GC 行的模式了，而且结合 Java 高效的分代 GC ，简直就是一个理想搭配。
>
> **② 影响**
>
> Netty 4 引入了手工内存的模式，我觉得这是一大创新，这种模式甚至于会延展， 应用到 Cache 应用中。实际上，结合 JVM 的诸多优秀特性，如果用 Java 来实现一个 Redis 型 Cache、 或者 In-memory SQL Engine，或者是一个 Mongo DB，我觉得相比 C/C++ 而言，都要更简单很多。 实际上，JVM 也已经提供了打通这种技术的机制，就是 Direct Memory 和 Unsafe 对象。 基于这个基础，我们可以像 C 语言一样直接操作内存。实际上，Netty4 的 ByteBuf 也是基于这个基础的。

## Netty 如何实现内存管理？

Netty 内存管理机制，基于 [Jemalloc](http://www.cnhalo.net/2016/06/13/memory-optimize/) 算法。

- 首先会预申请一大块内存 **Arena** ，Arena 由许多 Chunk 组成，而每个 Chunk 默认由2048个page组成。
- **Chunk** 通过 **AVL** 树的形式组织 **Page** ，每个叶子节点表示一个 Page ，而中间节点表示内存区域，节点自己记录它在整个 Arena 中的偏移地址。当区域被分配出去后，中间节点上的标记位会被标记，这样就表示这个中间节点以下的所有节点都已被分配了。大于 8k 的内存分配在 **PoolChunkList** 中，而 **PoolSubpage** 用于分配小于 8k 的内存，它会把一个 page 分割成多段，进行内存分配。

## 什么是 Netty 的内存泄露检测？是如何进行实现的？

> ToDo
