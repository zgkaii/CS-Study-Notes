# 架构设计

Netty 的分层很清晰：

- 核心层，主要定义一些基础设施，比如事件模型、通信 API、缓冲区等。
- 传输服务层，主要定义一些通信的底层能力，或者说是传输协议的支持，比如 TCP、UDP、HTTP 隧道、虚拟机管道等。
- 协议支持层，这里的协议比较广泛，不仅仅指编解码协议，还可以是应用层协议的编解码，比如 HTTP、WebSocket、SSL、Protobuf、文本协议、二进制协议、压缩协议、大文件传输等，基本上主流的协议都支持。

 <div align="center"> <img src="..\..\..\images\nio\netty框架.png" width="500px"></div>

Netty 的核心在于其可扩展的事件模型、通用的通信 API、基于零拷贝的缓冲区等，它们就像厨房中的锅碗瓢盆和油盐酱醋，有了厨房的这些基础设施，才能做出美味的佳肴。然后，我们才能定义更上层的服务，比如传输协议、序列化协议、编解码协议等。传输协议就像是我们的点餐系统，是通过普通的菜单点餐，还是扫码点餐等。编解码协议更像是怎么盛放佳肴的问题，是通过好看的餐盘，还是通过纸质饭盒，当然，有了基础设施和传输服务，怎么编解码传输的内容就更简单了，甚至，我们可以定义自己的编解码方式，比如，JSON 序列化。

# Netty 编码十步曲

**1. 声明线程池（必须）**

声明两个 Group，一个是 bossGroup，用于处理 Accept 事件，一个是 workerGroup，用于处理消息的读写事件。其中，bossGroup 一般声明为一个线程。当然，如果声明一个 Group 也是可以的，只是不建议。

```java
        EventLoopGroup bossGroup = new NioEventLoopGroup(2);
        NioEventLoopGroup workerGroup = new NioEventLoopGroup(16);
```

这就像是大型餐厅一般有接待生和服务员两种职位一样，接待生一般形象良好，专门负责接待客人，服务员形象稍差，专门负责上菜收碟，你要说不区分两种职位，混用行不行呢，当然也可以，只是不建议，专人干专事。

**2. 创建服务端引导类（必须）**

引导类，是用来集成所有配置，引导程序加载的，分成两种，一种是客户端引导类 Bootstrap（个人觉得叫 ClientBootstrap 可能更贴切），另一种是服务端引导类 ServerBootstrap，我们这里编写的是服务端程序，创建的当然是服务端引导类。注意，Bootstrap 和 ServerBootstrap 之间并不是继承关系，他们是平等的，都继承了 AbstractBootstrap 抽象类。

```java
ServerBootstrap serverBootstrap = new ServerBootstrap();
```

Bootstrap/ServerBootstrap 就像是店长，它负责统筹整个 Netty 程序的正常运行。

**3.设置线程池（必须）**

把第一步声明的线程池设置到 ServerBootstrap 中，它说明了 Netty 应用程序以什么样的线程模型运行，正如前面所说 bossGroup 负责接受（Accept）连接，workerGroup 负责读写数据。

把第一步声明的线程池设置到 ServerBootstrap 中，它说明了 Netty 应用程序以什么样的线程模型运行，正如前面所说 bossGroup 负责接受（Accept）连接，workerGroup 负责读写数据。

```java
serverBootstrap.group(bossGroup, workerGroup);
```

**4. 设置 ServerSocketChannel 类型（必须）**

设置 Netty 程序以什么样的 IO 模型运行，我们这里介绍的是 NIO 编程，选择的当然是 NioServerSocketChannel。

```java
serverBootstrap.channel(NioServerSocketChannel.class);
```

如果您需要使用阻塞型 IO 模型，直接把 Nio 改成 Oio 就可以了，即 OioServerSocketChannel，不过它已经废弃了，所以不建议。

另外，如果您的程序运行在 Linux 系统上，还可以使用一种更高效的方式，即 EpollServerSocketChannel，它使用的是 Linux 系统上的 epoll 模型，比 select 模型更高效，可见 Netty 把性能优化做到了极致。

**5. 设置参数（必须）**

设置 Netty 中可以使用的任何参数，这些参数都在 ChannelOption 及其子类中，后面我们会详细介绍各个参数的含义，不过，很遗憾地告诉你，大多数情况下并不需要修改 Netty 的默认参数，这就是 Netty 比较牛的地方。

```java
serverBootstrap.option(ChannelOption.SO_BACKLOG, 100);
```

我们这里设置了一个 SO_BACKLOG 系统参数，它表示的是最大等待连接数量。

**6. 设置 Handler（可选）**

设置 ServerSocketChannel 对应的 Handler，注意只能设置一个，它会在 SocketChannel 建立起来之前执行，等我们看源码的时候会详细介绍它的执行时机。

```java
serverBootstrap.handler(new LoggingHandler(LogLevel.INFO))
```

我们这里简单地设置一个打印日志的 Handler。

**7. 编写并设置子 Handler（必须）**

Netty 中的 Handler 分成两种，一种叫做 Inbound，一种叫做 Outbound。我们这里简单地写一个 Inbound 类型的 Handler，它接收到数据后立即写回客户端。

```java
public class EchoServerHandler extends ChannelInboundHandlerAdapter {

	@Override
	public void channelRead(ChannelHandlerContext ctx, Object msg) {
		// 读取数据后写回客户端
		ctx.write(msg);
	}

	@Override
	public void channelReadComplete(ChannelHandlerContext ctx) {
		ctx.flush();
	}

	@Override
	public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
		cause.printStackTrace();
		ctx.close();
	}
}
```

设置子 Handler 设置的是 SocketChannel 对应的 Handler，注意也是只能设置一个，它用于处理 SocketChannel 的事件。

```java
serverBootstrap.childHandler(new ChannelInitializer<SocketChannel>() {
	@Override
	public void initChannel(SocketChannel ch) throws Exception {
		ChannelPipeline p = ch.pipeline();
		// 可以添加多个子Handler
		p.addLast(new LoggingHandler(LogLevel.INFO));
		p.addLast(new EchoServerHandler());
	}
});
```

虽然只能设置一个，但是 Netty 的提供了一种可以设置多个 Handler 的途径，即使用 ChannelInitializer 方式，当然，第六步的设置 Handler 也可以使用这种方式设置多个 Handler。

这里，我们设置了一个打印的 Handler 和一个自定义的 EchoServerHandler。

**8. 绑定端口（必须）**

绑定端口，并启动服务端程序，sync () 会阻塞直到启动完成才执行后面的代码。

```java
ChannelFuture f = serverBootstrap.bind(PORT).sync();
```

**9. 等待服务端端口关闭（必须）**

等待服务端监听端口关闭，sync () 会阻塞主线程，内部调用的是 Object 的 wait () 方法。

```java
f.channel().closeFuture().sync();
```

**10. 优雅地关闭线程池（建议）**

最后，是在 finally 中调用 shutdownGracefully () 方法优雅地关闭线程池，优雅停机。

```java
bossGroup.shutdownGracefully();
workerGroup.shutdownGracefully();
```

好了，Netty 编码十步曲介绍完毕了，不知道你有没有什么疑问呢？至少我是有的。

**为什么需要设置 ServerSocketChannel 的类型，而不需要设置 SocketChannel 的类型呢？**

那是因为 SocketChannel 是 ServerSocketChannel 在接受连接之后创建出来的，所以，并不需要单独再设置它的类型，比如，NioServerSocketChannel 创建出来的肯定是 NioSocketChannel，而 EpollServerSocketChannel 创建出来的肯定是 EpollSocketChannel。