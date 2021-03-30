



















## 写在最前

**Netty是一个异步事件驱动的网络应用程序框架，用于快速开发可维护的高性能协议服务器和客户端。**

| 分类     | Netty特性                                                    |
| -------- | ------------------------------------------------------------ |
| 设计     | 统一的 API，支持多种传输类型，阻塞的和非阻塞的<br/>简单而强大的线程模型<br/>真正的无连接数据报套接字支持<br/>链接逻辑组件以支持复用 |
| 易于使用 | 详实的Javadoc和大量的示例集<br/>不需要超过JDK 1.6+的依赖（一些可选的特性可能需要Java 1.7+和/或额外的依赖） |
| 性能     | 拥有比 Java 的核心 API 更高的吞吐量以及更低的延迟<br/>得益于池化和复用，拥有更低的资源消耗<br/>最少的内存复制 |
| 健壮性   | 不会因为慢速、快速或者超载的连接而导致 OutOfMemoryError <br/>消除在高速网络中 NIO 应用程序常见的不公平读/写比率 |
| 安全性   | 完整的 SSL/TLS 以及 StartTLS 支持<br/>可用于受限环境下，如 Applet 和 OSGI |
| 社区驱动 | 发布快速而且频繁                                             |

Netty场见应用场景：

* **作为 RPC 框架的网络通信工具** 
* **实现一个自己的 HTTP 服务器**
* **实现一个即时通讯系统** 
* **实现消息推送系统** 
* ... ...

> 使用Netty的开源项目：[Related projects](https://netty.io/wiki/related-projects.html)

# 一、Netty架构设计

## 1.1 功能特性

Netty 的分层很清晰：

- 核心层，主要定义一些基础设施，比如可扩展事件模型、通用通信 API、支持零拷贝的ByteBuffer缓冲区。
- 传输服务层，主要定义一些通信的底层能力，或者说是传输协议的支持，比如 TCP、UDP、HTTP 隧道、虚拟机管道等。
- 协议支持层，支持 HTTP、Protobuf、二进制、文本、WebSocket等一系列常见协议， 还支持通过实行编码解码逻辑来实现自定义协议。

 <div align="center"> <img src="..\..\..\images\nio\netty框架.png" width="500px"></div>

## 1.2 核心组件

### 1.2.1 Bootstrap&ServerBootstrap（启动引导类）

Bootstrap意思是引导，一个Netty应用通常由一个Bootstrap开始，主要作用是配置整个Netty程序，串联各个组件，Netty中Bootstrap类是客户端程序的启动引导类，ServerBootstrap是服务端启动引导类。

 <div align="center"> <img src="..\..\..\images\nio\Bootstrap.png" width="400px"></div>
**`Bootstrap` 是客户端的启动引导类/辅助类**，具体使用方法如下：

```java
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            //创建客户端启动引导/辅助类：Bootstrap
            Bootstrap b = new Bootstrap();
            //指定线程模型
            b.group(group).
                    ......
            // 尝试建立连接
            ChannelFuture f = b.connect(host, port).sync();
            f.channel().closeFuture().sync();
        } finally {
            // 优雅关闭相关线程组资源
            group.shutdownGracefully();
        }
```

**`ServerBootstrap` 客户端的启动引导类/辅助类**，具体使用方法如下：

```java
        // 1.bossGroup 用于接收连接，workerGroup 用于具体的处理
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            //2.创建服务端启动引导/辅助类：ServerBootstrap
            ServerBootstrap b = new ServerBootstrap();
            //3.给引导类配置两大线程组,确定了线程模型
            b.group(bossGroup, workerGroup).
                   ......
            // 6.绑定端口
            ChannelFuture f = b.bind(port).sync();
            // 等待连接关闭
            f.channel().closeFuture().sync();
        } finally {
            //7.优雅关闭相关线程组资源
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
```

从上面的示例中，我们可以看出：

1. `Bootstrap` 通常使用 `connet()` 方法连接到远程的主机和端口，作为一个 Netty TCP 协议通信中的客户端。另外，`Bootstrap` 也可以通过 `bind()` 方法绑定本地的一个端口，作为 UDP 协议通信中的一端。
2. `ServerBootstrap`通常使用 `bind()` 方法绑定本地的端口上，然后等待客户端的连接。
3. `Bootstrap` 只需要配置一个线程组— `EventLoopGroup` ,而 `ServerBootstrap`需要配置两个线程组— `EventLoopGroup` ，一个用于接收连接，一个用于具体的 IO 处理。

相对于 Bootstrap，ServerBootstrap 多了一个维度，用于处理 Accept 事件，所以它的很多方法都会多一份 `childXxx()`，比如，`childHandler()`、`childOption()` 等（没有 `childChannel()` 这个方法，因为子 Channel 是通过 ServerSocketChannel 创建出来的，跟踪源码会发现 ServerSocketChannel 读取到消息的时候会把这个消息转换成连接，即SocketChannel）。

### 1.2.2 EventLoop（事件循环）

#### （1）NioEventLoopGroup 

NioEventLoopGroup，主要管理eventLoop的生命周期，可以理解为一个线程池，内部维护了一组线程，每个线程(NioEventLoop)负责处理多个Channel上的事件，而一个Channel只对应于一个线程。

 <div align="center"> <img src="..\..\..\images\nio\NioEventLoopGroup.png" width="400px"></div>

上面四个接口很熟悉：

- Iterable，迭代器的接口，说穿 EventLoopGroup 是一个容器，可以通过迭代的方式查看里面的元素。
- Executor，线程池的顶级接口，包含一个 execute()方法，用于提交任务到线程池中。
- ExecutorService，扩展自 Executor 接口，提供了通过 submit()方法提交任务的方式，并增加了 shutdown()等其它方法。
- ScheduledExecutorService，扩展自 ExecutorService，增加了定时任务执行相关的方法。

其中，后面三个都是线程池中的接口，位于著名的 `java.util.concurrent` 包下面。

下面的几个接口或类都属于 Netty：

- EventExecutorGroup，扩展自 ScheduledExecutorService，并增加了两大功能，一是提供了 next()方法用于获取一个 EventExecutor，二是管理这些 EventExecutor 的生命周期。

- EventLoopGroup，扩展自 EventExecutorGroup，并增加或修改了两大功能，一是提供了 next()方法用于获取一个 EventLoop，二是提供了注册 Channel 到事件轮询器中。

- MultithreadEventLoopGroup，抽象类，EventLoopGroup 的所有实现类都继承自这个类，可以看作是一种模板，从名字也可以看出来它里面包含多个线程来处理任务。

- NioEventLoopGroup，具体实现类，使用 NIO 形式（多路复用中的 select）工作的 EventLoopGroup。更换前缀就可以得到不同的实现类，比如 EpollEventLoopGroup 专门用于 Linux 平台，KQueueEventLoopGroup 专门用于 MacOS/BSD 平台。

  > select/epoll/kqueue，它们是实现 IO 多路复用的不同形式，select 支持的平台比较广泛，epoll 和 kqueue 比 select 更高效，epoll 只支持 linux，kqueue 只支持 BSD 平台，其中 MacOS 衍生自 BSD，所以 kqueue 也支持 MacOS。Netty 专门为两个平台做了的不同实现，也是对性能的极致追求，而且，我们服务端通常都是运行在 Linux 系统上，所以在上线的时候完全可以使用 EpollEventLoopGroup 来代替 NioEventLoopGroup。

#### （2）NioEventLoop 

EventLoop 可以理解为是 EventLoopGroup 中的工作线程，类似于 ThreadPoolExecutor 中的 Worker。但是，实际上它并不是一个线程，NioEventLoop中维护了一个线程和任务队列，支持异步提交执行任务，线程启动时会调用NioEventLoop的run方法，执行I/O任务和非I/O任务：

- I/O任务 即selectionKey中ready的事件，如accept、connect、read、write等，由processSelectedKeys方法触发。
- 非IO任务 添加到taskQueue中的任务，如register0、bind0等任务，由runAllTasks方法触发。

两种任务的执行时间比由变量ioRatio控制，默认为50，则表示允许非IO任务执行的时间与IO任务的执行时间相等。

 <div align="center"> <img src="..\..\..\images\nio\NioEventLoop.png" width="700px"></div>

可以看到这个继承体系比 EventLoopGroup 还复杂，这里介绍几个关键的接口或类：

- EventExecutor，扩展自 EventLoopGroup，主要增加了判断一个线程是不是在 EventLoop 中的方法。
- OrderedEventExecutor，扩展自 EventExecutor，这是一个标记接口，标志着里面的任务都是按顺序执行的。
- EventLoop，扩展自 EventLoopGroup，它将为已注册进来的 Channel 处理所有的 IO 事件，另外，它还扩展自 OrderedEventExecutor 接口，说明里面的任务是按顺序执行的。
- SingleThreadEventLoop，抽象类，EventLoop 的所有实现类都继承自这个类，可以看作是一种模板，从名字也可以看出来它是使用单线程处理的。
- NioEventLoop，具体实现类，绑定到一个 Selector 上，同时可以注册多个 Channel 到 Selector 上，同时，它继承自 SingleThreadEventLoop，也就说明了一个 Selector 对应一个线程。同样地，更换前缀就可以得到不同的实现，比如 EpollEventLoop、KQueueEventLoop。

### 1.2.4 ByteBuf（字节容器）

**网络通信最终都是通过字节流进行传输的。 `ByteBuf` 就是 Netty 提供的一个字节容器，其内部是一个字节数组。** 当我们通过 Netty 传输数据的时候，就是通过 `ByteBuf` 进行的。

我们可以将 `ByteBuf` 看作是 Netty 对 Java NIO 提供了 `ByteBuffer` 字节容器的封装和抽象。

**为什么不直接使用 Java NIO 提供的 `ByteBuffer` 呢？**因为 `ByteBuffer` 这个类使用起来过于复杂和繁琐。

那么 Netty 是怎么实现的呢？ByteBuf 声明了两个指针：一个读指针 `readIndex` 用于读取数据，一个写指针 `writeIndex` 用于写数据。

 <div align="center"> <img src="..\..\..\images\nio\ByteBuf1.png" width="900px"></div>

使用读写指针分离带来的好处是明显的，彻底解决了读写模式切换来切换去、position 指针变来变去的问题。那么，新的 **ByteBuf 都有哪些特性呢？**

首先，让我们看看 ByteBuf 的分类，常见的分类方式有三种：

- Pooled 和 Unpooled，池化和非池化

  池化，即初始化时分配好一块内存作为内存池，每次创建 ByteBuf 时从这个内存池中分配一块连续的内存给这个 ByteBuf 使用，待这个 ByteBuf 使用完了之后再放回内存池中，供后续的 ByteBuf 使用。利用池化技术，可以减少虚拟机频繁的内存回收带来的性能开销及资源消耗。池化技术在很多场景中都有使用到，比如，数据库连接池、线程池等，它们都有一些共同的特点，就是创建对象比较耗费资源。

  池化技术在生活中也随处可见，比如饭店的大厅、快递公司的网点。我们以饭店的大厅为例，它就像一块内存，来一个客人就给他分配一个桌子，如果没有大厅怎么办呢？来一个客人，饭店老板先去买个空间，也可能买块地建个空间，然后分配给这个客人，这个客人用完了再把这个空间卖掉，等下一个客人来的时候再重复以上步骤，想像一下都很美 ^^，所以，老板直接买下一个大厅，来一个客人给他分配一个桌子，用完回收，完全自己管理这片空间，省时又省力。

  非池化，即完全利用 JVM 本身的内存管理能力来管理对象的生命周期，即我们平时开发使用的模式，对象的内存分配完全交给 JVM 来管理，我们不用管对象内存的管理和回收。

- Heap 和 Direct，堆内存和直接内存

  堆内存，比较好理解，即 JVM 本身的堆内存。

  直接内存，独立于 JVM 内存之外的内存空间，直接向操作系统申请一块内存。

- Safe 和 Unsafe，安全和非安全

  Unsafe，底层使用 Java 本身的 Unsafe 来操作底层的数据结构，即直接利用对象在内存中的指针来操作对象，所以，比较危险。

基于以上三个维度，而且是完全不相干的三个维度，就形成了 `2 * 2 * 2 = 8` 种完全不一样的 ByteBuf，即 PooledHeapByteBuf、PooledUnsafeHeapByteBuf、PooledDirectByteBuf、PooledUnsafeDirectByteBuf、UnpooledHeapByteBuf、UnpooledUnsafeHeapByteBuf、UnpooledDirectByteBuf、UnpooledUnsafeDirectByteBuf。

 <div align="center"> <img src="..\..\..\images\nio\ByteBuf.png" width="900px"></div>

好了，上面介绍了 ByteBuf 的分类，你一定会说，这么多 ByteBuf，到底用哪个呢？怎么使用呢？

其实，Netty 都已经为我们想好了，关于上面八种 ByteBuf，我们并不需要显式地去调用它们的构造方法，而是使用一种叫作 `ByteBufAllocator` 分配器的东西来为我们创建 ByteBuf 对象，而这种分配器又有四种不同的类型：

- PooledByteBufAllocator，使用池化技术，内部会根据平台特性自行决定使用哪种 ByteBuf
- UnpooledByteBufAllocator，不使用池化技术，内部会根据平台特性自行决定使用哪种 ByteBuf
- PreferHeapByteBufAllocator，更偏向于使用堆内存，即除了显式地指明了使用直接内存的方法都使用堆内存
- PreferDirectByteBufAllocator，更偏向于使用直接内存，即除了显式地指明了使用堆内存的方法都使用直接内存

 <div align="center"> <img src="..\..\..\images\nio\ByteBufAllocator.png" width="900px"></div>
看到这里，你可能已经想骂粗口了，别急，淡定，八种 ByteBuf，四种 Allocator，对于拥有选择恐惧症的我该怎么办？

Netty 真的为你想好了，只需要简单地调用如下几行代码，Netty 就可以帮你创建最适合当前平台的 ByteBuf：

```java
ByteBufAllocator allocator = ByteBufAllocator.DEFAULT;
ByteBuf buffer = allocator.buffer(length);
buffer.writeXxx(xxx);
```

是不是很贴心呢？

默认地，Netty 将`最大努力`地使用池化、Unsafe、直接内存的方式为你创建 ByteBuf，为什么说是`最大努力`呢？因为在有些平台下某种特性支持地不是很好，所以 Netty 默认不会开启，比如 Android 平台下不会使用 Unsafe。

```java
// io.netty.util.internal.PlatformDependent#unsafeUnavailabilityCause0
if (isAndroid()) {
    logger.debug("sun.misc.Unsafe: unavailable (Android)");
    return new UnsupportedOperationException("sun.misc.Unsafe: unavailable (Android)");
}
```

### 1.2.5 Channel（网络操作抽象类）

通道，Java NIO 中的基础概念，代表一个打开的连接，可执行读取/写入 IO 操作。 Netty 对 Channel 的所有 IO 操作都是非阻塞的。

Netty 的 Channel 是对 Java 原生 Channel 的进一步封装，不仅封装了原生 Channel 操作的复杂性，还提供了一些很酷且实用的功能，比如：

- 可以获取当前连接的状态及配置参数（例如接收缓冲区大小）
- 通过 ChannelPipeline 来处理 IO 事件
- 在 Netty 中的所有 IO 操作都是异步的
- 可继承的 Channel 体系

与原生 Channel 对应，Netty 的 Channel 都有相应的包装类，并且还扩展了其它协议的实现：

- NioSocketChannel，异步的客户端 TCP Socket 连接
- NioServerSocketChannel，异步的服务器端 TCP Socket 连接
- NioDatagramChannel，异步的 UDP 连接
- NioSctpChannel，异步的客户端 Sctp 连接
- NioSctpServerChannel，异步的 Sctp 服务器端连接 这些通道涵盖了 UDP 和 TCP网络 IO以及文件 IO

可以看到，对各种协议的支持在 Netty 中很容易实现，且它很擅长。

Netty 不仅支持这些协议的 NIO 通用平台实现，还支持特定平台的实现，而且只需要简单地更换前缀就可以达到对不同平台的支持，比如，ServerSocketChannel 的通用实现为 NioServerSocketChannel，在 Linux 下完全可以更换成 EpollServerSocketChannel，代码只需要做很小的修改，就可以达到平台级的性能提升。

 <div align="center"> <img src="..\..\..\images\nio\Channel.png" width="900px"></div>

`Channel` 接口是 Netty 对网络操作抽象类。通过 `Channel` 我们可以进行 I/O 操作。

一旦客户端成功连接服务端，就会新建一个 `Channel` 同该用户端进行绑定，示例代码如下：

```java
   //  通过 Bootstrap 的 connect 方法连接到服务端
   public Channel doConnect(InetSocketAddress inetSocketAddress) {
        CompletableFuture<Channel> completableFuture = new CompletableFuture<>();
        bootstrap.connect(inetSocketAddress).addListener((ChannelFutureListener) future -> {
            if (future.isSuccess()) {
                completableFuture.complete(future.channel());
            } else {
                throw new IllegalStateException();
            }
        });
        return completableFuture.get();
    }
```

比较常用的`Channel`接口实现类是 ：

- `NioServerSocketChannel`（服务端）
- `NioSocketChannel`（客户端）

这两个 `Channel` 可以和 BIO 编程模型中的`ServerSocket`以及`Socket`两个概念对应上。

### 1.2.6 ChannelHandler（消息处理器） 

ChannelHandler 是核心业务处理接口，处理I / O事件或拦截I / O操作，并将其转发到 ChannelPipeline 中的下一个 ChannelHandler，运用的是责任链设计模式。

- ChannelHandler本身并没有提供很多方法，因为这个接口有许多的方法需要实现，方便使用期间，可以继承它的子类：
  - ChannelInboundHandler用于处理入站I / O事件
  - ChannelOutboundHandler用于处理出站I / O操作
  - ChannelDuplexHandler用于处理双向I / O操作

或者使用以下适配器类：

- ChannelInboundHandlerAdapter用于处理入站I / O事件
- ChannelOutboundHandlerAdapter用于处理出站I / O操作
- ChannelDuplexHandler用于处理入站和出站事件

其中，SimpleChannelInboundHandler 相比于 ChannelInboundHandlerAdapter 优势更明显，它可以帮我们做资源的自动释放等操作。

### 1.2.7 ChannelHandlerContext

ChannelHandlerContext 保存着 Channel 的上下文，同时关联着一个 ChannelHandler对象，通过 ChannelHandlerContext，ChannelHandler 方能与 ChannelPipeline 或者其它 ChannelHandler 进行交互，ChannelHandlerContext 是它们之间的纽带。

### 1.2.8 ChannelFuture（操作执行结果）

在Netty中所有的IO操作都是异步的，不能立刻得知消息是否被正确处理，但是可以过一会等它执行完成或者直接注册一个监听，具体的实现就是通过Future和ChannelFutures，他们可以注册一个监听，当操作执行成功或失败时监听会自动触发注册的监听事件。

通过 ChannelFuture，可以查看 IO 操作是否已完成、是否成功、是否已取消等等。

### 1.2.9 ChannelPipeline（ChannelHandler 对象链表）

ChannelPipeline 是 ChannelHandler 的集合，它负责处理和拦截入站和出站的事件和操作，每个 Channel 都有一个 ChannelPipeline 与之对应，会自动创建。

更确切地说，ChannelPipeline 中存储的是 ChannelHandlerContext 链，通过这个链把 ChannelHandler 连接起来，让我们仔细研究一下几者之间的关系：

- 一个 Channel 对应一个 ChannelPipeline
- 一个 ChannelPipeline 包含一条双向的 ChannelHandlerContext 链
- 一个 ChannelHandlerContext 中包含一个 ChannelHandler
- 一个 Channel 会绑定到一个 EventLoop 上
- 一个 NioEventLoop 维护了一个 Selector（使用的是 Java 原生的 Selector）
- 一个 NioEventLoop 相当于一个线程

通过以上分析，可以得出，ChannelPipeline、ChannelHandlerContext 都是线程安全的，因为同一个 Channel 的事件都会在一个线程中处理完毕（假设用户不自己启动线程）。但是，ChannelHandler 却不一定，ChannelHandler 类似于 Spring MVC 中的 Service 层，专门处理业务逻辑的地方，一个 ChannelHandler 实例可以供多个 Channel 使用，所以，不建议把有状态的变量放在 ChannelHandler 中，而是放在消息本身或者 ChannelHandlerContext 中。

好了，上面的关系已经描述清楚，让我们画个图直观地感受一下：

 <div align="center"> <img src="..\..\..\images\nio\ChannelPipeline.png" width="800px"></div>

### 1.2.10 ChannelOption

ChannelOption 严格来说不算是一种组件，它保存了很多我们拿来即用的参数，使用这些参数能够让我们以类型安全地方式来配置 Channel，比如，我们前面使用过的 ChannelOption.SO_BACKLOG，Netty 还提供了很多这种类似的参数，使得我们能够以更精细地方式控制程序正确、正常、高性能地运行。

## 1.3 小结

这些组件看似散乱，其实内含逻辑，如果非要给它们归类的话，我认为可以分成以下四类：

* 引导相关：Bootstrap 和 ServerBootstrap

* 线程相关：NioEventLoopGroup 、NioEventLoop（EventExecutorGroup、EventExecutor）

* Buffer 相关：ByteBuf

* Channel 相关：Channel、ChannelHandler、ChannelHandlerContext、ChannelFuture、ChannelPipeline、ChannelOption

# 二、高性能设计

高性能要求无非三方面：高并发用户（Concurrent Users）、高吞吐量（Throughout）与低延迟（Latency)。

Netty作为异步事件驱动的网络，高性能之处主要来自于其I/O模型和线程处理模型，前者决定如何收发数据，后者决定如何处理数据。

## 2.1 IO模型

Netty的非阻塞I/O的实现关键是基于I/O多路复用模型：

就是在单个线程里同时监控多个套接字，通过 select 或 poll 轮询所负责的所有 socket，当某个 socket 有数据到达了，就通知用户进程。

 <div align="center"> <img src="..\..\..\images\nio\复用IO.png" width="1000px"></div>

Netty的IO线程NioEventLoop由于聚合了多路复用器Selector，可以同时并发处理成百上千个客户端连接。当线程从某客户端Socket通道进行读写数据时，若没有数据可用时，该线程可以进行其他任务。线程通常将非阻塞 IO 的空闲时间用于在其他通道上执行 IO 操作，所以单独的线程可以管理多个输入和输出通道。

由于读写操作都是非阻塞的，这就可以充分提升IO线程的运行效率，避免由于频繁I/O阻塞导致的线程挂起，一个I/O线程可以并发处理N个客户端连接和读写操作，这从根本上解决了传统同步阻塞I/O一连接一线程模型，架构的性能、弹性伸缩能力和可靠性都得到了极大的提升。

## 2.2 线程模型

### 2.2.1 事件驱动模型

通常，我们设计一个事件处理模型的程序有两种思路

- 轮询方式 线程不断轮询访问相关事件发生源有没有发生事件，有发生事件就调用事件处理逻辑。
- 事件驱动方式 发生事件，主线程把事件放入事件队列，在另外线程不断循环消费事件列表中的事件，调用事件对应的处理逻辑处理事件。事件驱动方式也被称为消息通知方式，其实是设计模式中**观察者模式**的思路。

以GUI的逻辑处理为例，说明两种逻辑的不同：

- 轮询方式 线程不断轮询是否发生按钮点击事件，如果发生，调用处理逻辑
- 事件驱动方式 发生点击事件把事件放入事件队列，在另外线程消费的事件列表中的事件，根据事件类型调用相关事件处理逻辑

 <div align="center"> <img src="..\..\..\images\nio\事件驱动模型.png" width="700px"></div>

主要包括4个基本组件：

- 事件队列（event queue）：接收事件的入口，存储待处理事件
- 分发器（event mediator）：将不同的事件分发到不同的业务逻辑单元
- 事件通道（event channel）：分发器与处理器之间的联系渠道
- 事件处理器（event processor）：实现业务逻辑，处理完成后会发出事件，触发下一步操作

可以看出，相对传统轮询模式，事件驱动有如下优点：

- 可扩展性好，分布式的异步架构，事件处理器之间高度解耦，可以方便扩展事件处理逻辑
- 高性能，基于队列暂存事件，能方便并行异步处理事件

### 2.2.2 Reactor线程模型

Reactor是反应堆的意思，Reactor模型，是指通过一个或多个输入同时传递给服务处理器的服务请求的**事件驱动处理模式**。 服务端程序处理传入多路请求，并将它们同步分派给请求对应的处理线程，Reactor模式也叫Dispatcher模式，即I/O多了复用统一监听事件，收到事件后分发(Dispatch给某进程)，是编写高性能网络服务器的必备技术之一。

Reactor模型中有2个关键组成：

- Reactor Reactor在一个单独的线程中运行，负责监听和分发事件，分发给适当的处理程序来对IO事件做出反应。 它就像公司的电话接线员，它接听来自客户的电话并将线路转移到适当的联系人
- Handlers 处理程序执行I/O事件要完成的实际事件，类似于客户想要与之交谈的公司中的实际官员。Reactor通过调度适当的处理程序来响应I/O事件，处理程序执行非阻塞操作

 <div align="center"> <img src="..\..\..\images\nio\Reactor模型.png" width="600px"></div>

取决于Reactor的数量和Hanndler线程数量的不同，Reactor模型有3个变种

- 单Reactor单线程
- 单Reactor多线程
- 主从Reactor多线程

可以这样理解，Reactor就是一个执行while (true) { selector.select(); ...}循环的线程，会源源不断的产生新的事件，称作反应堆很贴切。

篇幅关系，这里不再具体展开Reactor特性、优缺点比较，有兴趣的读者可以参考我之前另外一篇文章：[《理解高性能网络模型》](https://www.jianshu.com/p/2965fca6bb8f)

#### （1）Reacto单线程

所有的IO操作都由同一个NIO线程处理。

 <div align="center"> <img src="..\..\..\images\nio\单线程.png" width="600px"></div>

 <div align="center"> <img src="..\..\..\images\nio\单线程1.png" width="600px"></div>

**单线程 Reactor 的优点是对系统资源消耗特别小，但是，没办法支撑大量请求的应用场景并且处理请求的时间可能非常慢，毕竟只有一个线程在工作嘛！所以，一般实际项目中不会使用单线程Reactor 。**

为了解决这些问题，演进出了Reactor多线程模型。

#### （2）Reacto多线程

一个线程负责接受请求,一组NIO线程处理IO操作。

 <div align="center"> <img src="..\..\..\images\nio\多线程.png" width="600px"></div>

 <div align="center"> <img src="..\..\..\images\nio\多线程1.png" width="600px"></div>



**大部分场景下多线程Reactor模型是没有问题的，但是在一些并发连接数比较多（如百万并发）的场景下，一个线程负责接受客户端请求就存在性能问题了。**

为了解决这些问题，演进出了主从多线程Reactor模型。

#### （3）Reacto主从线程

一组NIO线程负责接受请求，一组NIO线程处理IO操作。

 <div align="center"> <img src="..\..\..\images\nio\主从模型.png" width="600px"></div>

 <div align="center"> <img src="..\..\..\images\nio\主从模型1.png" width="600px"></div>

### 2.2.4 Netty线程模型

大部分网络框架都是基于 Reactor 模式设计开发的。

> Reactor 模式基于事件驱动，采用多路复用将事件分发给相应的 Handler 处理，非常适合处理海量 IO 的场景。

在 Netty 主要靠 `NioEventLoopGroup` 线程池来实现具体的线程模型的 。

我们实现服务端的时候，一般会初始化两个线程组：

1. **`bossGroup`** :接收连接。
2. **`workerGroup`** ：负责具体的处理，交由对应的 Handler 处理。

下面我们来详细看一下 Netty 中的线程模型吧！

#### （1）单线程模型

一个线程需要执行处理所有的 `accept`、`read`、`decode`、`process`、`encode`、`send` 事件。对于高负载、高并发，并且对性能要求比较高的场景不适用。

对应到 Netty 代码是下面这样的

> 使用 `NioEventLoopGroup` 类的无参构造函数设置线程数量的默认值就是 **CPU 核心数 \*2** 。

```java
  //1.eventGroup既用于处理客户端连接，又负责具体的处理。
  EventLoopGroup eventGroup = new NioEventLoopGroup(1);
  //2.创建服务端启动引导/辅助类：ServerBootstrap
  ServerBootstrap b = new ServerBootstrap();
            boobtstrap.group(eventGroup, eventGroup)
            //......
```

#### （2）多线程模型

一个 Acceptor 线程只负责监听客户端的连接，一个 NIO 线程池负责具体处理： `accept`、`read`、`decode`、`process`、`encode`、`send` 事件。满足绝大部分应用场景，并发连接量不大的时候没啥问题，但是遇到并发连接大的时候就可能会出现问题，成为性能瓶颈。

对应到 Netty 代码是下面这样的：

```java
// 1.bossGroup 用于接收连接，workerGroup 用于具体的处理
EventLoopGroup bossGroup = new NioEventLoopGroup(1);
EventLoopGroup workerGroup = new NioEventLoopGroup();
try {
  //2.创建服务端启动引导/辅助类：ServerBootstrap
  ServerBootstrap b = new ServerBootstrap();
  //3.给引导类配置两大线程组,确定了线程模型
  b.group(bossGroup, workerGroup)
    //......
```

![img](https://images.xiaozhuanlan.com/photo/2020/c1331bc2a17c2e0bc6b832833ffda9bb.png)

#### （3）主从多线程模型

从一个 主线程 NIO 线程池中选择一个线程作为 Acceptor 线程，绑定监听端口，接收客户端连接的连接，其他线程负责后续的接入认证等工作。连接建立完成后，Sub NIO 线程池负责具体处理 I/O 读写。如果多线程模型无法满足你的需求的时候，可以考虑使用主从多线程模型 。

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

![img](https://images.xiaozhuanlan.com/photo/2020/fcf90a64c3f7ce49b47c1d07ba3b9edb.png)

## 

Netty主要**基于主从Reactors多线程模型**（如下图）做了一定的修改，其中主从Reactor多线程模型有多个Reactor：MainReactor和SubReactor：

- MainReactor负责客户端的连接请求，并将请求转交给SubReactor
- SubReactor负责相应通道的IO读写请求
- 非IO请求（具体逻辑处理）的任务则会直接写入队列，等待worker threads进行处理

特别说明的是： 虽然Netty的线程模型基于主从Reactor多线程，借用了MainReactor和SubReactor的结构，但是实际实现上，SubReactor和Worker线程在同一个线程池中：

```
EventLoopGroup bossGroup = new NioEventLoopGroup();
EventLoopGroup workerGroup = new NioEventLoopGroup();
ServerBootstrap server = new ServerBootstrap();
server.group(bossGroup, workerGroup)
 .channel(NioServerSocketChannel.class)
复制代码
```

上面代码中的bossGroup 和workerGroup是Bootstrap构造方法中传入的两个对象，这两个group均是线程池

- bossGroup线程池则只是在bind某个端口后，获得其中一个线程作为MainReactor，专门处理端口的accept事件，**每个端口对应一个boss线程**
- workerGroup线程池会被各个SubReactor和worker线程充分利用

### 2.2.5 异步处理

异步的概念和同步相对。当一个异步过程调用发出后，调用者不能立刻得到结果。实际处理这个调用的部件在完成后，通过状态、通知和回调来通知调用者。

Netty中的I/O操作是异步的，包括bind、write、connect等操作会简单的返回一个ChannelFuture，调用者并不能立刻获得结果，通过Future-Listener机制，用户可以方便的主动获取或者通过通知机制获得IO操作结果。

当future对象刚刚创建时，处于非完成状态，调用者可以通过返回的ChannelFuture来获取操作执行的状态，注册监听函数来执行完成后的操，常见有如下操作：

- 通过isDone方法来判断当前操作是否完成
- 通过isSuccess方法来判断已完成的当前操作是否成功
- 通过getCause方法来获取已完成的当前操作失败的原因
- 通过isCancelled方法来判断已完成的当前操作是否被取消
- 通过addListener方法来注册监听器，当操作已完成(isDone方法返回完成)，将会通知指定的监听器；如果future对象已完成，则理解通知指定的监听器

例如下面的的代码中绑定端口是异步操作，当绑定操作处理完，将会调用相应的监听器处理逻辑

```
    serverBootstrap.bind(port).addListener(future -> {
        if (future.isSuccess()) {
            System.out.println(new Date() + ": 端口[" + port + "]绑定成功!");
        } else {
            System.err.println("端口[" + port + "]绑定失败!");
        }
    });
复制代码
```

相比传统阻塞I/O，执行I/O操作后线程会被阻塞住, 直到操作完成；异步处理的好处是不会造成线程阻塞，线程在I/O操作期间可以执行别的程序，在高并发情形下会更稳定和更高的吞吐量。

# 三、启动流程

## Netty 服务端和客户端的启动过程了解么？

### 服务端

```java
        // 1.bossGroup 用于接收连接，workerGroup 用于具体的处理
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            //2.创建服务端启动引导/辅助类：ServerBootstrap
            ServerBootstrap b = new ServerBootstrap();
            //3.给引导类配置两大线程组,确定了线程模型
            b.group(bossGroup, workerGroup)
                    // (非必备)打印日志
                    .handler(new LoggingHandler(LogLevel.INFO))
                    // 4.指定 IO 模型
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        public void initChannel(SocketChannel ch) {
                            ChannelPipeline p = ch.pipeline();
                            //5.可以自定义客户端消息的业务处理逻辑
                            p.addLast(new HelloServerHandler());
                        }
                    });
            // 6.绑定端口,调用 sync 方法阻塞知道绑定完成
            ChannelFuture f = b.bind(port).sync();
            // 7.阻塞等待直到服务器Channel关闭(closeFuture()方法获取Channel 的CloseFuture对象,然后调用sync()方法)
            f.channel().closeFuture().sync();
        } finally {
            //8.优雅关闭相关线程组资源
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
```

简单解析一下服务端的创建过程具体是怎样的：

1.首先你创建了两个 `NioEventLoopGroup` 对象实例：`bossGroup` 和 `workerGroup`。

- `bossGroup` : 用于处理客户端的 TCP 连接请求。
- `workerGroup` ： 负责每一条连接的具体读写数据的处理逻辑，真正负责 I/O 读写操作，交由对应的 Handler 处理。

举个例子：我们把公司的老板当做 bossGroup，员工当做 workerGroup，bossGroup 在外面接完活之后，扔给 workerGroup 去处理。一般情况下我们会指定 bossGroup 的 线程数为 1（并发连接量不大的时候） ，workGroup 的线程数量为 **CPU 核心数 \*2\*** *。另外，根据源码来看，使用 `NioEventLoopGroup` 类的无参构造函数设置线程数量的默认值就是 **CPU 核心数 \*****2** 。

2.接下来 我们创建了一个服务端启动引导/辅助类： `ServerBootstrap`，这个类将引导我们进行服务端的启动工作。

3.通过 `.group()` 方法给引导类 `ServerBootstrap` 配置两大线程组，确定了线程模型。

通过下面的代码，我们实际配置的是多线程模型，这个在上面提到过。

```java
    EventLoopGroup bossGroup = new NioEventLoopGroup(1);
    EventLoopGroup workerGroup = new NioEventLoopGroup();
```

4.通过`channel()`方法给引导类 `ServerBootstrap`指定了 IO 模型为`NIO`

- `NioServerSocketChannel` ：指定服务端的 IO 模型为 NIO，与 BIO 编程模型中的`ServerSocket`对应

- `NioSocketChannel` : 指定客户端的 IO 模型为 NIO， 与 BIO 编程模型中的`Socket`对应

   

  5.通过 `.childHandler()`给引导类创建一个`ChannelInitializer` ，然后指定了服务端消息的业务处理逻辑 `HelloServerHandler` 对象

   

  6.调用 `ServerBootstrap` 类的 `bind()`方法绑定端口

### 客户端

```java
        //1.创建一个 NioEventLoopGroup 对象实例
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            //2.创建客户端启动引导/辅助类：Bootstrap
            Bootstrap b = new Bootstrap();
            //3.指定线程组
            b.group(group)
                    //4.指定 IO 模型
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        public void initChannel(SocketChannel ch) throws Exception {
                            ChannelPipeline p = ch.pipeline();
                            // 5.这里可以自定义消息的业务处理逻辑
                            p.addLast(new HelloClientHandler(message));
                        }
                    });
            // 6.尝试建立连接
            ChannelFuture f = b.connect(host, port).sync();
            // 7.等待连接关闭（阻塞，直到Channel关闭）
            f.channel().closeFuture().sync();
        } finally {
            group.shutdownGracefully();
        }
```

继续分析一下客户端的创建流程：

1.创建一个 `NioEventLoopGroup` 对象实例

2.创建客户端启动的引导类是 `Bootstrap`

3.通过 `.group()` 方法给引导类 `Bootstrap` 配置一个线程组

4.通过`channel()`方法给引导类 `Bootstrap`指定了 IO 模型为`NIO`

5.通过 `.childHandler()`给引导类创建一个`ChannelInitializer` ，然后指定了客户端消息的业务处理逻辑 `HelloClientHandler` 对象

6.调用 `Bootstrap` 类的 `connect()`方法进行连接，这个方法需要指定两个参数：

- `inetHost` : ip 地址
- `inetPort` : 端口号

```java
    public ChannelFuture connect(String inetHost, int inetPort) {
        return this.connect(InetSocketAddress.createUnresolved(inetHost, inetPort));
    }
    public ChannelFuture connect(SocketAddress remoteAddress) {
        ObjectUtil.checkNotNull(remoteAddress, "remoteAddress");
        this.validate();
        return this.doResolveAndConnect(remoteAddress, this.config.localAddress());
    }
```

`connect` 方法返回的是一个 `Future` 类型的对象

```java
public interface ChannelFuture extends Future<Void> {
  ......
}
```

也就是说这个方是异步的，我们通过 `addListener` 方法可以监听到连接是否成功，进而打印出连接信息。具体做法很简单，只需要对代码进行以下改动：

```java
ChannelFuture f = b.connect(host, port).addListener(future -> {
  if (future.isSuccess()) {
    System.out.println("连接成功!");
  } else {
    System.err.println("连接失败!");
  }
}).sync();
```

# 参考

* [一文理解Netty模型架构](https://juejin.cn/post/6844903712435994631)