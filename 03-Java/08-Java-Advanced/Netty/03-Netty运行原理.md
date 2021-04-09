# 一、创建Netty程序

## 1.1 服务端

```java
public class EchoServer {
    static final int PORT = Integer.parseInt(System.getProperty("port", "8808"));

    public static void main(String[] args) throws Exception {
        final EchoServerHandler serverHandler = new EchoServerHandler();
        // 1.创建线程池
        EventLoopGroup bossGroup = new NioEventLoopGroup(2);
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            // 2.服务端引导类
            ServerBootstrap b = new ServerBootstrap();
            // 3. 设置参数(可选)
            b.option(ChannelOption.SO_BACKLOG, 128)
                    .childOption(ChannelOption.TCP_NODELAY, true);
            // 4.设置线程池
            b.group(bossGroup, workerGroup)
                    // 5.指定所使用的NIO传输Channel类型
                    .channel(NioServerSocketChannel.class)
                    // 6.设置ServerSocketChannel对应的Handler，只能设置一个（可选）
                    .handler(new LoggingHandler(LogLevel.INFO))
                    // 7. 设添加一个EchoServerHandler到子Channel的ChannelPipeline中
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            // EchoServerHandler被标注为@Shareable，所以我们可以总是使用同样的实例
                            ch.pipeline().addLast(serverHandler);
                        }
                    });
            // 8.异步地绑定服务器，调用 sync()方法阻塞等待直到绑定完成
            Channel ch = b.bind(PORT).sync().channel();
            System.out.println("开启netty http服务器，监听地址和端口为 http://127.0.0.1:" + PORT + '/');
            // 9.阻塞，直到Channel 关闭
            ch.closeFuture().sync();
        } finally {
            // 10.关闭线程池并且释放所有的资源
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

简单解析一下服务端的创建过程具体是怎样的：

**1. 声明线程池（必须）**

这里首先创建了两个 `NioEventLoopGroup` 对象实例：`bossGroup` 和 `workerGroup`。
`bossGroup` 用于处理客户端的 TCP 连接请求；`workerGroup` 负责每一条连接的具体读写数据的处理逻辑，真正负责 I/O 读写操作，交由对应的 Handler 处理。

> 公司的老板就如bossGroup，员工就如workerGroup，bossGroup接活扔给 workerGroup 去处理。一般情况下我们会指定 bossGroup 的 线程数为 1（并发连接量不大的时候），workGroup 的线程数量为 **CPU核心数\*2**（使用 `NioEventLoopGroup` 类的无参构造函数设置线程数量的默认值就是 **CPU核心数\*2**）。

**2. 创建服务端引导类（必须）**

引导类，是用来集成所有配置，引导程序加载的。分成两种，一种是客户端引导类 Bootstrap，另一种是服务端引导类 ServerBootstrap，这里编写的是服务端程序。

> Bootstrap 和 ServerBootstrap 之间并不是继承关系，他们是“平级”的，都继承于AbstractBootstrap 抽象类。

**3. 设置参数（可选）**

设置 Netty 中可以使用的任何参数，这些参数都在 ChannelOption 及其子类中，例如我们上面设置了SO_BACKLOG 系统参数，它表示的是最大等待连接数量为128。这里暂时多介绍各个参数的含义，大多数情况下我们也并不需要修改Netty 的默认参数。

**4. 设置线程池（必须）**

通过 `.group()` 方法给引导类 `ServerBootstrap` 配置两大线程组，确定了线程模型。它说明了 Netty 应用程序以什么样的线程模型运行，正如前面所说 bossGroup 负责接受（Accept）连接，workerGroup 负责读写数据。

**5 设置 ServerSocketChannel 类型（必须）**

通过`channel()`方法给引导类 `ServerBootstrap`指定了服务端 IO 模型为`NIO`(`NioSocketChannel` )，如果需要使用阻塞型 IO 模型，直接把 Nio 改成 Oio 就可以了，即 OioServerSocketChannel，不过它已经废弃了，不建议使用。

另外，如果程序运行在 Linux 系统上，还可以使用一种更高效的方式，即 EpollServerSocketChannel，它使用的是 Linux 系统上的 epoll 模型，比 select 模型更高效。

> **为什么需要设置 ServerSocketChannel 的类型，而不需要设置 SocketChannel 的类型呢？**
>
> 因为 SocketChannel 是 ServerSocketChannel 在连接之后创建出来的，并不需要单独再设置它的类型。NioServerSocketChannel 创建出来的肯定是 NioSocketChannel，而 EpollServerSocketChannel 创建出来的肯定是 EpollSocketChannel。

**6. 设置ServerSocketChannel的Handler（可选）**

设置 ServerSocketChannel 对应的 Handler，注意只能设置一个，它会在 SocketChannel 建立起来之前执行，等我们看源码的时候会详细介绍它的执行时机。上面简单地设置一个打印日志的 Handler。

**7. 编写并设置子 Handler（必须）**

当一个新的连接被接受时，一个新的子 Channel 将会被创建，而 ChannelInitializer 将会把一个你的 EchoServerHandler 的实例添加到该 Channel 的 ChannelPipeline 中。正如我们之前所解释的，这个 ChannelHandler 将会收到有关入站消息的通知。

```java
@ChannelHandler.Sharable// 标示一个ChannelHandler 可以被多个 Channel 安全地共享
public class EchoServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ByteBuf in = (ByteBuf) msg;
        // 将消息记录到控制台
        System.out.println("Server received: " + in.toString(CharsetUtil.UTF_8));
        // 将接收到的消息写给发送者，而不冲刷出站消息
        ctx.write(in);
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) {
        // 将未决消息冲刷到远程节点，并且关闭该Channel
        ctx.writeAndFlush(Unpooled.EMPTY_BUFFER)
                .addListener(ChannelFutureListener.CLOSE);
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();// 关闭该Channel
    }
}
```

**8. 绑定端口（必须）**

调用 `ServerBootstrap` 类的 `bind()`方法绑定端口，并启动服务端程序，sync()会阻塞直到启动完成才执行后面的代码。

**9. 等待服务端端口关闭（必须）**

等待服务端监听端口关闭，sync()会阻塞主线程，内部调用的是 Object 的 wait () 方法。

**10. 优雅地关闭线程池（建议）**

最后，是在 finally 中调用 shutdownGracefully () 方法优雅地关闭线程池，优雅停机。

## 1.2 客户端

```java
public class EchoClient {
    static final String HOST = System.getProperty("host", "127.0.0.1");
    static final int PORT = Integer.parseInt(System.getProperty("port", "8808"));

    public static void main(String[] args) throws Exception {
        // 1.创建一个 NioEventLoopGroup 对象实例
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            // 2.创建客户端启动引导类
            Bootstrap b = new Bootstrap();
            // 3.指定线程组
            b.group(group)
                    // 4.指定 IO 模型
                    .channel(NioSocketChannel.class)
                    // 5.在创建Channel时，向ChannelPipeline中添加一个 EchoClientHandler 实例
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        public void initChannel(SocketChannel ch) throws Exception {
                            ChannelPipeline p = ch.pipeline();
                            p.addLast(new EchoClientHandler());
                        }
                    });
            // 6.连接到远程节点，阻塞等待直到连接完成
            ChannelFuture f = b.connect(HOST, PORT).sync();
            // 7.阻塞，直到Channel 关闭
            f.channel().closeFuture().sync();
        } finally {
            // 8.关闭线程池并且释放所有的资源
            group.shutdownGracefully();
        }
    }
}
```

客户端的创建流程与服务端类似：

1.创建一个 `NioEventLoopGroup` 对象实例

2.创建客户端启动的引导类 `Bootstrap`

3.通过 `.group()` 方法给引导类 `Bootstrap` 配置一个线程组

4.通过`channel()`方法给引导类 `Bootstrap`指定了 IO 模型为`NIO`（NioSocketChannel）

5.通过 `.childHandler()`给引导类创建一个`ChannelInitializer` ，然后指定了客户端消息的业务处理逻辑 `EchoClientHandler` 对象

```java
@ChannelHandler.Sharable // 标记该类的实例可以被多个Channel共享
public class EchoClientHandler extends SimpleChannelInboundHandler<ByteBuf> {
    /**
     * 当被通知Channel是活跃的时候，发送一条消息
     * @param ctx
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) {
        ctx.writeAndFlush(Unpooled.copiedBuffer("Netty rocks!", CharsetUtil.UTF_8));
    }

    /**
     * 记录已接收消息的转储
     */
    @Override
    public void channelRead0(ChannelHandlerContext ctx, ByteBuf in) {
        System.out.println("Client received: " + in.toString(CharsetUtil.UTF_8));
    }

    /**
     * 在发生异常时，记录错误并关闭Channel
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```

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

7.等待客户端端口关闭

等待客户端监听端口关闭，sync()会阻塞主线程，内部调用的是 Object 的 wait () 方法。

8.优雅地关闭线程池

最后，是在 finally 中调用 shutdownGracefully () 方法优雅地关闭线程池，优雅停机。

## 1.3 启动测试

启动服务端与客户端，在服务器的控制台中，可以看到：

```html
Server received: Netty rocks!
```

每次运行客户端时，在服务器的控制台中都能看到这条日志语句。 下面是发生的事： 

（1）一旦客户端建立连接，它就发送它的消息——Netty rocks!； 

（2）服务器报告接收到的消息，并将其回送给客户端； 

（3）客户端报告返回的消息并退出。

# 二、Netty运行原理

## 2.1 Netty启动与运行流程

结合Netty模型架构和上面的简易应用案例，下面分析一下Netty的服务端启动运行流程和运行原理。

 <div align="center"> <img src="..\..\..\images\nio\Netty启动和处理流程.png" width="700px"></div>

大致可以总结为5个步骤：

* 创建线程池：例如在服务端中创建了两个 `NioEventLoopGroup` 对象实例`bossGroup` 和 `workerGroup`，`bossGroup` 用于处理客户端的 TCP 连接请求，`workerGroup` 负责每一条连接的具体读写数据的处理逻辑（这里是多线程Reactor模型）；而客户端只创建了一个 `NioEventLoopGroup`线程组 。
* 绑定监听端口：客户端与服务端都是通过各自的启动引导类异步绑定服务器，调用 sync()方法阻塞等待直到绑定完成。
* 客户端连接：客户端通过host与port访问服务端，服务端通过bossGroup接受连接请求事件并返回响应。
* 注册Channel：服务端的通过NioEventLoop 得到SockectChannel，然后创建对应的NioSocketChannel对象；在创建Channel时，向ChannelPipeline中添加一个 EchoClientHandler 实例。
* 事件处理：服务端的workerGroup再注册多个NioEventGroup，每个NioEventGroup都绑定有具体的Handler，对这次的请求进行相应的业务逻辑处理。

## 2.2 Netty运行原理

从上面可以看出，NioEventGroup尤为关键。`EventLoop` 定义（事件循环）Netty 的核心抽象，用于处理连接的生命周期中所发生的事件，其主要作用实际就是责监听网络事件并调用事件处理器进行相关 I/O 操作（读写）的处理。

**Channel 和 EventLoop 的关系**

* `Channel` 为 Netty 网络操作（读写等操作）抽象类，`EventLoop` 负责处理注册到其上的`Channel` 的 I/O 操作，两者配合进行 I/O 操作。

**EventloopGroup 和 EventLoop 的关系**

* `EventLoopGroup` 包含多个 `EventLoop`（每一个 `EventLoop` 通常内部包含一个线程），它管理着所有的 `EventLoop` 的生命周期。

* 并且，`EventLoop` 处理的 I/O 事件都将在它专有的 `Thread` 上被处理，即 `Thread` 和 `EventLoop` 属于 1 : 1 的关系，从而保证线程安全。

> NIO模型中即对应NioEventLoopGroup与NioEventLoop。 

 <div align="center"> <img src="..\..\..\images\nio\线程模式.png" width="600px"></div>

当前实现中， 使用**顺序循环（round-robin）**的方式进行分配以获取一个均衡的分布，并且相同的EventLoop可能会被分配给多个 Channel。 一旦一个 Channel 被分配给一个EventLoop，它将在它的整个生命周期中都使用这个 EventLoop（以及相关联的 Thread）。

 <div align="center"> <img src="..\..\..\images\nio\eventloop.png" width="600px"></div>

可以这里理解`EventLoop`：如果把Netty应用比作虚拟机的话，那么`EventLoop`就是CPU；如果把Netty比作一个餐厅的话，那么`EventLoop`就是上菜员。

 <div align="center"> <img src="..\..\..\images\nio\核心对象.png" width="600px"></div>

一个 EventLoopGroup 包含一个或者多个 EventLoop，而一个 EventLoop 在它的生命周期内只和一个 Thread 绑定，所有由 EventLoop 处理的 I/O 事件都将在它专有的 Thread 上被处理； 一个 Channel 在它的生命周期内只注册于一个 EventLoop，而一个 Channel 在它的生命周期内只注册于一个 EventLoop。

 <div align="center"> <img src="..\..\..\images\nio\Netty运行原理.png" width="600px"></div>

