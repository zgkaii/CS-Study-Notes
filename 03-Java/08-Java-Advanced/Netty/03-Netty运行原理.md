# 一、创建Netty程序

## 1.1 服务端

```java
        // 1. 声明线程池
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            // 2. 服务端引导器
            ServerBootstrap b = new ServerBootstrap();
            // 3. 设置参数(非必须)
            b.option(ChannelOption.SO_BACKLOG, 128)
                    .childOption(ChannelOption.TCP_NODELAY, true)
                    .childOption(ChannelOption.SO_KEEPALIVE, true)
                    .childOption(ChannelOption.SO_REUSEADDR, true)
                    .childOption(ChannelOption.SO_RCVBUF, 32 * 1024)
                    .childOption(ChannelOption.SO_SNDBUF, 32 * 1024)
                    .childOption(EpollChannelOption.SO_REUSEPORT, true)
                    .childOption(ChannelOption.SO_KEEPALIVE, true)
                    .childOption(ChannelOption.ALLOCATOR, PooledByteBufAllocator.DEFAULT);
            // 4. 设置线程池
            b.group(bossGroup, workerGroup)
                    // 5. 设置ServerSocketChannel的类型
                    .channel(NioServerSocketChannel.class)
                    // 6. 设置ServerSocketChannel对应的Handler，只能设置一个
                    .handler(new LoggingHandler(LogLevel.INFO))
                    // 7. 设置SocketChannel对应的Handler
                   .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        public void initChannel(SocketChannel ch) {
                            ChannelPipeline p = ch.pipeline();
                            // 可以添加多个子Handler
							p.addLast(new LoggingHandler(LogLevel.INFO));
                            // 可以自定义客户端消息的业务处理逻辑
                            p.addLast(new MyServerHandler());
                        }
                    });

            // 8. 绑定端口
            Channel ch = b.bind(port).sync().channel();
            System.out.println("开启netty http服务器，监听地址和端口为 http://127.0.0.1:" + port + '/');
            // 9. 等待服务端监听端口关闭，这里会阻塞主线程
            ch.closeFuture().sync();
        } finally {
            // 10. 优雅地关闭两个线程池
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
```

简单解析一下服务端的创建过程具体是怎样的：

**1. 声明线程池（必须）**

首先你创建了两个 `NioEventLoopGroup` 对象实例：`bossGroup` 和 `workerGroup`。
`bossGroup` 用于处理客户端的 TCP 连接请求；`workerGroup` 负责每一条连接的具体读写数据的处理逻辑，真正负责 I/O 读写操作，交由对应的 Handler 处理。

> 公司的老板就如bossGroup，员工就如workerGroup，bossGroup接活扔给 workerGroup 去处理。一般情况下我们会指定 bossGroup 的 线程数为 1（并发连接量不大的时候），workGroup 的线程数量为 **CPU核心数\*2**（使用 `NioEventLoopGroup` 类的无参构造函数设置线程数量的默认值就是 **CPU核心数\*2**）。

**2. 创建服务端引导类（必须）**

引导类，是用来集成所有配置，引导程序加载的。分成两种，一种是客户端引导类 Bootstrap（个人觉得叫 ClientBootstrap 可能更贴切），另一种是服务端引导类 ServerBootstrap，我们这里编写的是服务端程序，创建的当然是服务端引导类。
注意，Bootstrap 和 ServerBootstrap 之间并不是继承关系，他们是平等的，都继承了 AbstractBootstrap 抽象类。

**3. 设置参数（可选）**

设置 Netty 中可以使用的任何参数，这些参数都在 ChannelOption 及其子类中，这里暂时不介绍各个参数的含义。不过大多数情况下我们也并不需要修改Netty 的默认参数。

**4. 设置线程池（必须）**

通过 `.group()` 方法给引导类 `ServerBootstrap` 配置两大线程组，确定了线程模型。它说明了 Netty 应用程序以什么样的线程模型运行，正如前面所说 bossGroup 负责接受（Accept）连接，workerGroup 负责读写数据。

**5 设置 ServerSocketChannel 类型（必须）**

通过`channel()`方法给引导类 `ServerBootstrap`指定了服务端 IO 模型为`NIO`(指定客户端的 IO 模型为 NIO则是`NioSocketChannel` )。

如果您需要使用阻塞型 IO 模型，直接把 Nio 改成 Oio 就可以了，即 OioServerSocketChannel，不过它已经废弃了，所以不建议。另外，如果程序运行在 Linux 系统上，还可以使用一种更高效的方式，即 EpollServerSocketChannel，它使用的是 Linux 系统上的 epoll 模型，比 select 模型更高效。

> **为什么需要设置 ServerSocketChannel 的类型，而不需要设置 SocketChannel 的类型呢？**
>
> 因为 SocketChannel 是 ServerSocketChannel 在连接之后创建出来的，并不需要单独再设置它的类型。NioServerSocketChannel 创建出来的肯定是 NioSocketChannel，而 EpollServerSocketChannel 创建出来的肯定是 EpollSocketChannel。

**6. 设置 Handler（可选）**

设置 ServerSocketChannel 对应的 Handler，注意只能设置一个，它会在 SocketChannel 建立起来之前执行，等我们看源码的时候会详细介绍它的执行时机。

```java
serverBootstrap.handler(new LoggingHandler(LogLevel.INFO))
```

我们这里简单地设置一个打印日志的 Handler。

**7. 编写并设置子 Handler（必须）**

通过 `.childHandler()`给引导类创建一个`ChannelInitializer` ，然后指定了服务端消息的业务处理逻辑 `MyServerHandler` 对象。

**8. 绑定端口（必须）**

调用 `ServerBootstrap` 类的 `bind()`方法绑定端口，并启动服务端程序，sync()会阻塞直到启动完成才执行后面的代码。

**9. 等待服务端端口关闭（必须）**

等待服务端监听端口关闭，sync () 会阻塞主线程，内部调用的是 Object 的 wait () 方法。

**10. 优雅地关闭线程池（建议）**

最后，是在 finally 中调用 shutdownGracefully () 方法优雅地关闭线程池，优雅停机。

## 1.2 客户端

```java
        // 1.创建一个 NioEventLoopGroup 对象实例
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            // 2.创建客户端启动引导/辅助类：Bootstrap
            Bootstrap b = new Bootstrap();
            // 3.指定线程组
            b.group(group)
                    // 4.指定 IO 模型
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

# 二、Netty启动与处理流程

 <div align="center"> <img src="..\..\..\images\nio\Netty启动和处理流程.png" width="700px"></div>

# 三、Netty运行原理

# 参考