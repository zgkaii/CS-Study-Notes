<!-- MarkdownTOC -->

- [什么是RPC](#什么是rpc)
- [RPC原理浅析](#rpc原理浅析)
- [RPC技术核心](#rpc技术核心)
  - [RPC 协议](#rpc-协议)
  - [RPC 序列化](#rpc-序列化)
    - [(1) JSON](#1-json)
    - [(2) Hessian](#2-hessian)
    - [(3) Protobuf](#3-protobuf)
  - [RPC 网络通信模型](#rpc-网络通信模型)
  - [RPC 动态代理](#rpc-动态代理)
- [常见的 RPC 框架](#常见的-rpc-框架)
  - [RMI 的原理及应用](#rmi-的原理及应用)
- [思考](#思考)
- [总结](#总结)
- [参考资料](#参考资料)

<!-- /MarkdownTOC -->

# 什么是RPC

RPC（Remote Procedure Call）,中文意为“**远程过程调用**”，它通过网络请求远程计算机服务的通信技术。RPC框架封装了底层网络通信、序列化等技术，我们只需要在项目中引入各个服务的接口包，**就可以像调用本地方法一样调用RPC服务**。

无论是微服务、SOA、还是 RPC 架构，它们都是分布式服务架构，都需要实现服务之间的互相通信，我们通常把这种通信统称为 RPC 通信。正因为远程调用的方便、透明，RPC 被广泛应用于当下企业级以及互联网项目中，**是实现分布式系统的核心**。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210615113128698.png" width="1000px"/>
</div>

# RPC原理浅析

RPC调用的核心原理就是**代理机制**，简化后RPC 执行流程：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210615114239570.png" width="500px"/>
</div>

可以看出，RPC具体可以分为如下几个步骤（同步调用）：

1. 本地代理存根: Stub
2. 本地序列化反序列化
3. 网络通信
4. 远程序列化反序列化
5. 远程服务存根: Skeleton
6. 调用实际业务服务
7. 原路返回服务结果
8. 返回给本地调用方

**案例分析**

以电商购物平台为例，每一笔交易都涉及订单系统、支付系统和库存系统，假设三个系统分别部署在三台机器 A、B、C 中独立运行，订单交易流程如下所示：

1. 用户下单时，调用本地（机器 A）的订单系统进行下单；
2. 下单完成后，会远程调用机器 B 上的支付系统进行支付，待支付完成后返回结果，之后在本地更新订单状态；
3. 在本地远程调用机器 C 上的仓库系统出货，出货完成后返回出货结果。

在整个过程中，“下单”和“订单状态更新”两个操作属于本地调用，而“支付”和“出货”这两个操作是通过本地的订单系统调用其他两个机器上的函数（方法）实现的，属于远程调用。

整个订单交易流程如下图所示。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210615124410590.png" width="550px"/>
</div>

那么以上面的“支付”操作为例，来详细看看一次 RPC 调用的完整流程：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210615124555449.png" width="550px"/>
</div>

（1）本地服务器也就是机器A的订单系统，调用本地服务器上的支付系统的支付操作Pay(Order)，该方法会直接调用 Client Stub（其中，Stub 是用于转换 RPC 过程中在订单系统和支付系统所在机器之间传递的参数），这是一次正常的本地调用。

（2）Client Stub 将方法 Pay、参数 Order 等打包成一个适合网络传输的消息，通过执行一次系统调用（也就是调用操作系统中的函数）来发送消息。

（3）订单系统所在机器 A 的本地操作系统通过底层网络通信，将打包好的消息根据支付系统所在机器 B 的地址发送出去。

（4）机器 B 上的操作系统接收到消息后，将消息传递给 Server Stub。

（5）机器 B 上的 Server Stub 将接收到的消息进行解包，获得里面的参数，然后调用本地的支付订单的操作 Pay(Order)。

（6）机器 B 上的支付操作 Par(Order) 完成后，将结果发送给 Server Stub，其中结果可使用 XDR（External Data Representation，一种可以在不同计算机系统间传输的数据格式）语言表示。

（7）机器 B 上的 Server Stub 将结果数据打包成适合网络传输的消息，然后进行一次系统调用发送消息。

（8）机器 B 的本地操作系统将打包好的消息通过网络发送回机器 A。

（9）机器 A 的操作系统接收到来自机器 B 的消息，并将消息发送给本地的 Client Stub。

（10）本地的 Client Stub 对消息进行解包，然后将解包得到的结果返回给本地的订单系统。

到此，整个 RPC 过程结束。RPC 的目的，其实就是要将第 2 到第 8 步的几个过程封装起来，让用户看不到这些细节。从用户的角度看，订单系统的进程只是做了一次普通的本地调用，然后就得到了结果。

也就是说，订单系统进程并不需要知道底层是如何传输的，在用户眼里，远程过程调用和调用一次本地服务没什么不同。这，就是 RPC 的核心。

# RPC技术核心

## RPC 协议

RPC传输协议跟HTTP协议一样，都是应用层协议，是建立在TCP协议实现之上的。RPC请求在发送到网络中之前，需要把方法调用的请求参数转化为二进制；转化为二进制后，写入本地Socket中，然后被网卡发送到网络设备中。在这个过程中，容易发生拆包/粘包的问题。所以我们就需要在发送请求前就确定一个边界进行数据分隔，而这个边界语义的表达，就是所谓的协议。

> 思考：既然有HTTP协议，为什么还要用RPC 进行服务调用？

**如何设计设计一个可扩展且向后兼容的协议呢？**

我们大体可以把协议分为协议头与协议体两部分，协议头除了存放协议长度、序列化方式，还会放一些协议标示，消息ID、消息类型这样的参数，而协议体一般只放请求接口方法、请求业务参数值和一些拓展属性。协议头是由一堆固定的长度参数组成，而协议体是根据接口和参数构造的，长度可变。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210615135115892.png" width="800px"/>
</div>

这种长协议头不能往协议里面加新参数，容易导致线上兼容问题；而协议体中加新参数使用时需要反序列化，代价高。所以设计一种支持可扩展的协议，其关键在于让协议头支持可扩展，扩展后协议头的长度就不能定长了。

那要实现读取不定长的协议头里面的内容，在这之前肯定需要一个固定的地方读取长度，所以我们需要一个固定的写 入协议头的长度。整体协议就变成了三部分内容：固定部分、协议头内容、协议体内容，前两部分我们还是可以统称为“协议头”，具体协议如下：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210615135719223.png" width="700px"/>
</div>

也就是说，要做到协议可扩展且向后兼容，协议的结构不仅要支持协议的扩展，还要做到协议头也能扩展。

## RPC 序列化

简单地将，序列化就是将对象转化成二进制数据的过程，而反序列化就是将二进制数据转换为对象的过程。因为网络传输的数据必须是二进制数据，所以在RPC调用中，对入参对象和返回值对象进行序列化和反序列化是一个必然的过程。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210615140757508.png" width="900px"/>
</div>
**有哪些常见的序列化呢？**

### (1) JSON

JSON 可能是我们最熟悉的一种序列化格式了，JSON 是典型的 `Key-Value` 方式，没有数据类型，是一种文本型序列化框架，JSON 的具体格式和特性，这里就不再介绍了。 他在应用上还是很广泛的，无论是前台 Web 用 Ajax 调用、用磁盘存储文本类型的数据， 还是基于 HTTP 协议的 RPC 框架通信，都会选择 JSON 格式。

 但用 JSON 进行序列化有这样两个问题，需要格外注意：

* JSON 进行序列化的额外空间开销比较大，对于大数据量服务这意味着需要巨大的内存 和磁盘开销； 
* JSON 没有类型，但像 Java 这种强类型语言，需要通过反射统一解决，所以性能不会太 好。

### (2) Hessian

Hessian 是动态类型、二进制、紧凑的，并且可跨语言移植的一种序列化框架。Hessian协议要比原始JDK序列化、JSON 更加紧凑，性能上要比JDK、JSON 序列化高效很多，而且生成的字节数也更小。

使用代码示例如下： 

```java
Student student = new Student();
student.setNo(101);
student.setName("HESSIAN");
//把student对象转化为byte数组
ByteArrayOutputStream bos = new ByteArrayOutputStream();
Hessian2Output output = new Hessian2Output(bos);
output.writeObject(student);
output.flushBuffer();
byte[] data = bos.toByteArray();
bos.close();
//把刚才序列化出来的byte数组转化为student对象
ByteArrayInputStream bis = new ByteArrayInputStream(data);
Hessian2Input input = new Hessian2Input(bis);
Student deStudent = (Student) input.readObject();
input.close();
System.out.println(deStudent);
```

相对于 JDK、JSON，由于 Hessian 更加高效，生成的字节数更小，有非常好的兼容性和稳 定性，所以 Hessian 更加适合作为 RPC 框架远程通信的序列化协议。

但 Hessian 本身也有问题，官方版本对 Java 里面一些常见对象的类型不支持，比如： 

* Linked 系列，`LinkedHashMap`、`LinkedHashSet` 等，但是可以通过扩展 `CollectionDeserializer` 类修复； 
* Locale 类，可以通过扩展 `ContextSerializerFactory` 类修复； 
* Byte/Short 反序列化的时候变成 Integer。

### (3) Protobuf

`Protobuf` 是 Google 公司内部的混合语言数据标准，是一种轻便、高效的结构化数据存储 格式，可以用于结构化数据序列化，支持 Java、Python、C++、Go 等语言。`Protobuf` 使用的时候需要定义 IDL（Interface description language），然后使用不同语言的 IDL 编译器，生成序列化工具类，它的优点是：

* 序列化后体积相比 JSON、Hessian 小很多； 
* IDL（[接口描述语言](https://zh.wikipedia.org/wiki/%E6%8E%A5%E5%8F%A3%E6%8F%8F%E8%BF%B0%E8%AF%AD%E8%A8%80)） 能清晰地描述语义，足以帮助并保证应用程序之间的类型不会丢失，无需类似 XML 解析器； 
* 序列化反序列化速度很快，不需要通过反射获取类型； 消息格式升级和兼容性不错，可以做到向后兼容。

 使用代码示例如下： 

```java
/**
*
* // IDL 文件格式
* synax = "proto3";
* option java_package = "com.test";
* option java_outer_classname = "StudentProtobuf";
*
* message StudentMsg {
* //序号
* int32 no = 1;
* //姓名
* string name = 2;
* }
*
*/
StudentProtobuf.StudentMsg.Builder builder = 
    StudentProtobuf.StudentMsg.newBuilbuilder.setNo(103);
builder.setName("protobuf");
//把student对象转化为byte数组
StudentProtobuf.StudentMsg msg = builder.build();
byte[] data = msg.toByteArray();
//把刚才序列化出来的byte数组转化为student对象
StudentProtobuf.StudentMsg deStudent = 
    StudentProtobuf.StudentMsg.parseFrom(datSystem.out.println(deStudent);
```

`Protobuf` 非常高效，但是对于具有反射和动态能力的语言来说，这样用起来很费劲，这一 点就不如 Hessian，比如用 Java 的话，这个预编译过程不是必须的，可以考虑使用 `Protostuff`。

 `Protostuff` 不需要依赖 IDL 文件，可以直接对 Java 领域对象进行反 / 序列化操作，在效率上跟 `Protobuf` 差不多，生成的二进制格式和 `Protobuf` 是完全相同的，可以说是一个 Java 版本的 `Protobuf` 序列化框架。但在使用过程中，可能会遇到一些不支持的情况，例如：不支持NULL；`ProtoStuff` 不支持单纯的 Map、List 集合对象，需要包在对象里面。

## RPC 网络通信模型

RPC解决的是进程间通信的一种方式。一次RPC调用，本质就是服务消费者和服务提供者间的一次网络信息交换的过程。服务调用者通过网络IO发送一条请求消息，服务提供者接受并解析，处理完相关的业务逻辑后，再发送一条响应消息给服务调用者，服务调用者接收并解析响应消息，处理完相关的响应逻辑，一次RPC调用便结束了。可以说，网络通信时整个RPC调用流程的基础。

RPC调用在大多数情况下，是一个高并发调用的场景，考虑到系统内核的支持，编程语言已经支持IO模型本身的特点，在RPC框架实现网络通信的处理上，一般会选择IO多路复用的方式。开发语言的网络通信框架的选型上，我们最优的选择是基于 Reactor 模式实现的框架，如 Java 语言，首选的框架便是 `Netty` 框架（Java 还有很多其 他 NIO 框架，但目前 `Netty` 应用得最为广泛），并且在 Linux 环境下，也要开启 `epoll` 来 提升系统性能。

在网络通信应用中，有一项特别重要的技术——**零拷贝（Zero-copy）技术**。

所谓的零拷贝，就是取消用户空间与内核空间之间的数据拷贝操作，应用进程每一次的读写操作，可以通过一种方式，直接将数据写入内核或从内核中读取数据，再通过 DMA 将内 核中的数据拷贝到网卡，或将网卡中的数据 copy 到内核。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/2021061514511858.png" width="800px"/>
</div>

零拷贝有两种解决方式，分别是 `mmap+write` 方式和 `sendfile` 方式，其核心原理都是 通过虚拟内存来解决的

在 OS 层面上的 `Zero-copy` 通常指避免在 `用户态(User-space)` 与 `内核态(Kernel-space)` 之间来回拷贝数据，可以提升 CPU 的利用率。而在 `Netty` 层面 ，零拷贝主要体现在对于数据操作的优化。

上面提到，在传输过程中，RPC 并不会把请求参数的所有二进制数据整体一下子发送到对端机器上，中间可能会拆分成好几个数据包，也可能会合并其他请求的数据包，所以消息都需要有边界。那 么一端的机器收到消息之后，就需要对数据包进行处理，根据边界对数据包进行分割和合 并，最终获得一条完整的消息。

那收到消息后，对数据包的分割和合并，是在用户空间完成，还是在内核空间完成的呢？当然是在用户空间，因为对数据包的处理工作都是由应用程序来处理的，那么这里有没有可 能存在数据的拷贝操作？可能会存在，当然不是在用户空间与内核空间之间的拷贝，是用户 空间内部内存中的拷贝处理操作。`Netty` 的零拷贝就是为了解决这个问题，在用户空间对数据操作进行优化。

那么 `Netty` 是怎么对数据操作进行优化的呢？

1. 使用 `Netty` 提供的 `CompositeByteBuf` 类，可以将多个`ByteBuf` 合并为一个逻辑上的 `ByteBuf`，避免了各个 `ByteBuf` 之间的拷贝。
2. `ByteBuf` 支持 `slice` 操作，因此可以将 `ByteBuf` 分解为多个共享同一个存储区域的 `ByteBuf`，避免了内存的拷贝。
3. 通过 wrap 操作，我们可以将 byte[] 数组、`ByteBuf`、`ByteBuffer` 等包装成一个 `Netty ByteBuf` 对象, 进而避免拷贝操作。

`Netty` 框架中很多内部的 `ChannelHandler` 实现类，都是通过 `CompositeByteBuf`、 slice、wrap 操作来处理 TCP 传输中的拆包与粘包问题的。 

那么 `Netty` 有没有解决用户空间与内核空间之间的数据拷贝问题的方法呢？

`Netty` 的 `ByteBuffer` 可以采用 Direct Buffers，使用堆外直接内存进行 `Socketd` 的读写 操作，最终的效果与我刚才讲解的虚拟内存所实现的效果是一样的。 `Netty` 还提供 `FileRegion` 中包装 NIO 的 `FileChannel.transferTo()` 方法实现了零拷 贝，这与 Linux 中的 `sendfile` 方式在原理上也是一样的。

## RPC 动态代理

在项目中，我们使用RPC的时候，往往只需要通过依赖注入的方式把接口注入到项目中就行，然后在代码中直接调用接口中方法。而接口中并不包含真实的业务逻辑，业务逻辑都在服务提供方应用里，我们通过调用接口方法就能拿到想要结果。其实这里应用到的核心技术就是[动态代理](https://github.com/zgkaii/CS-Study-Notes/blob/master/03-Java/01-Java-Basic/14-%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86.md)。

RPC 会自动给接口生成一个代理类，当我们在项目中注入接口的时候，运行过程中实际绑定的是这个接口生成的代理类。这样在接口方法被调用的时候，它实际上是被生成代理类拦截到了，这样我们就可以在生成的代理类里面，加入远程调用逻辑。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210615151113325.png" width="500px"/>
</div>

这里就不谈论动态代理的实现原理了。其实在 Java 领域，除了 JDK 默认的 `InvocationHandler` 能完成代理功能，我们还有很多其他的第三方框架也可以，比如像 `Javassist`、`Byte Buddy` 这样的框架。

单纯从代理功能上来讲，JDK默认的代理功能具有一定的局限性，它要求被代理的类只能是接口。原因是因为生成的代理类会继承 Proxy 类，但 Java 是不支持多重继承的。

相对 JDK 自带的代理功能，`Javassist` 的定位是能够操纵底层字节码，所以使用起来并不简 单，要生成动态代理类恐怕是有点复杂了。但好的方面是，通过 `Javassist` 生成字节码，不需要通过反射完成方法调用，所以性能肯定是更胜一筹的。在使用中，我们要注意一个问 题，通过 `Javassist` 生成一个代理类后，此 `CtClass` 对象会被冻结起来，不允许再修改；否则，再次生成时会报错。 

Byte Buddy 则属于后起之秀，在很多优秀的项目中，像 Spring、Jackson 都用到了 Byte Buddy 来完成底层代理。相比 `Javassist`，Byte Buddy 提供了更容易操作的 API，编写的代码可读性更高。更重要的是，生成的代理类执行速度比 `Javassist` 更快。

# 常见的 RPC 框架

- **RMI(JDK自带)：** JDK自带的RPC，能够让本地 Java 虚拟机上运行的对象调用远程方法如同调用本地方法，隐藏通信细节。但有很多局限性，不推荐直接使用。
- **Dubbo**：`Dubbo`是 阿里巴巴公司开源的一个高性能优秀的服务框架，使得应用可通过高性能的 RPC 实现服务的输出和输入功能，可以和 Spring框架无缝集成。目前 Dubbo 已经成为 Spring Cloud Alibaba 中的官方组件。
- **gRPC** ：`gRPC`是可以在任何环境中运行的现代开源高性能RPC框架。它可以通过可插拔的支持来有效地连接数据中心内和跨数据中心的服务，以实现负载平衡，跟踪，运行状况检查和身份验证。它也适用于分布式计算的最后一英里，以将设备，移动应用程序和浏览器连接到后端服务。
- **Hessian：** Hessian是一个轻量级的`remotingonhttp`工具，使用简单的方法提供了RMI的功能。 相比`WebService`，Hessian更简单、快捷。采用的是二进制RPC协议，因为采用的是二进制协议，所以它很适合于发送二进制数据。
- **Thrift：**  Apache Thrift是Facebook开源的跨语言的RPC通信框架，目前已经捐献给Apache基金会管理，由于其跨语言特性和出色的性能，在很多互联网公司得到应用，有能力的公司甚至会基于thrift研发一套分布式服务框架，增加诸如服务注册、服务发现等功能。
- ... ...

## RMI 的原理及应用

RMI 是一个用于实现 RPC 的 Java API，能够让本地 Java 虚拟机上运行的对象调用远程方法如同调用本地方法，隐藏通信细节。RMI 可以说是 RPC 的一种具体形式，其原理与 RPC 基本一致，唯一不同的是 RMI 是基于对象的，充分利用了面向对象的思想去实现整个过程，其本质就是一种基于对象的 RPC 实现。RMI 的具体原理如下图所示：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210615132714565.png" width="500px"/>
</div>

RMI 的实现中，客户端的订单系统中的 Stub 是客户端的一个辅助对象，用于与服务端实现远程调用；服务端的支付系统中 Skeleton 是服务端的一个辅助对象，用于与客户端实现远程调用。

也就是说，客户端订单系统的 Pay(Order) 调用本地 Stub 对象上的方法，Stub 对调用信息（比如变量、方法名等）进行打包，然后通过网络发送给服务端的 Skeleton 对象，Skeleton 对象将收到的包进行解析，并调用服务端 Pay(Order) 系统中的相应对象和方法进行计算，计算结果又会以类似的方式返回给客户端。

为此，我们可以看出，RMI 与 PRC 最大的不同在于调用方式和返回结果的形式，RMI 通过对象作为远程接口来进行远程方法的调用，返回的结果也是对象形式，比如 Java 对象类型，或者是基本数据类型等。

RMI 的典型实现框架有 EJB（Enterprise JavaBean，企业级 JavaBean）。

接下来我通过一个表格来对比下RPC与RMI异同：

| 对比         | RPC                                            | RMI                                                                                          |
| ------------ | ---------------------------------------------- | -------------------------------------------------------------------------------------------- |
| 定位         | 一种网络服务应用框架                           | 基于对象的RPC的具体对象实现，应用编程接口（API）                                             |
| 适用范围     | 与操作系统和语言无关，不支持传输对象           | 用于Java环境，支持传输对象                                                                   |
| 调用方式     | 不同服务器上的进程通过网络协议向对方发送请求。 | 不同服务器上的进程之间，通过对象作为远程接口来进行远程方法的调用，每个远程方法都有唯一的ID。 |
| 结果返回形式 | 由XDR表示                                      | 返回Java对象类型或者基本数据类型                                                             |
| 典型实现     | Dubbo                                          | EJB                                                                                          |

# 思考

**思考1：既然存在HTTP协议，为什么还要用RPC进行服务调用？**

**解答**：

* 相对于 HTTP 的用处，RPC 更多的是负责应用间的通信，所以性能要求相对更高；而HTTP 协议的数据包大小相对请求数据本身要大很多，又需要加入 很多无用的内容，比如换行符号、回车符等。例如下面一个POST协议的格式大致如下：

```html
HTTP/1.0 200 OK 
Content-Type: text/plain
Content-Length: 137582
Expires: Thu, 05 Dec 1997 16:00:00 GMT
Last-Modified: Wed, 5 August 1996 15:55:28 GMT
Server: Apache 0.84

<html>
  <body>Hello World</body>
</html>
```

* HTTP 协议属于无状态协议，客户端无法对请求和响应进行关联，每次请求都需要重新建立连接，响应完成后再关闭连接。因此，对于要求高性能的 RPC 来说，HTTP 协议基本很难满足需求，所以 RPC 会选择设计更紧凑的私有协议。

* 简单来说成熟的RPC库相对HTTP容器，更多的是封装了“服务注册与发现”，"负载均衡"，“熔断降级”，“可视化的服务治理和运维”、“运行期流量调度”一类面向服务的高级特性。可以这么理解，RPC框架是面向服务的更高级的封装。


# 总结

RPC（Remote Procedure Call）即所谓的远程过程调用，是分布式系统实现的核心。它通过封装了一底层网络通信、序列化等技术，我们只需要在项目中引入各个服务的接口包，**就可以像调用本地方法一样调用RPC服务**。

在RPC中，传输协议的作用就类似于文字的符号，作为应用拆解请求的边界，保证二进制数据经过网络传输后，还能被正确的还原语义，避免出现调用方于被调用方之间“鸡同鸭讲”。在设计RPC协议的时候，我们需要保证其能很好的扩展，支持新增功能。

RPC序列化方式很很多种，类如JDK原生序列化、JSON/XML、Hessian、`Protobuf`等。当我们技术选型时，参考因素有安全性、通用性、兼容性、性能、效率、空间开销（优先度由高到低）。一般首选时Hessian或`Probobuf`，Hessian在适用上更方便，对象兼容性更好；`Protobuf`则更加高效，通用性更由优势。

RPC 框架可以让我们发起全程调用就像调用本地，但在 RPC 框架的传输过 程中，入参与返回值的根本作用就是用来传递信息的，为了提高 RPC 调用整体的性能和稳定性，我们构造入参、返回值对象，主要记住以下几点：

1. 对象要尽量简单，没有太多的依赖关系，属性不要太多，尽量高内聚；
2. 入参对象与返回值对象体积不要太大，更不要传太大的集合；
3.  尽量使用简单的、常用的、开发语言原生的对象，尤其是集合类；
4. 对象不要有复杂的继承关系，最好不要有父子类的情况。

RPC 框架在网络通信的 处理上，我们更倾向选择 IO 多路复用 技术和零拷贝技术。在 OS 层面上的 `Zero-copy` 通常指避免在 `用户态(User-space)` 与 `内核态(Kernel-space)` 之间来回拷贝数据，可以提升 CPU 的利用率。而在 `Netty` 层面 ，零拷贝偏向于用户空间中对数据操作的优化，这对处理 TCP 传输中的拆包粘包问题有着重要的意义，对应用程序处理请求数据与返回数据也有重要的意义。

RPC实现远程调用的重要一项技术即动态代理，它屏蔽了具体的操作细节，在编码时，代理逻辑与业务逻辑互相独立，各不影响，没有侵入。在 Java 领域，除了 JDK 默认的 `InvocationHandler` 能完成代理功能，我们还有很多其他的第三方框架也可以，比如像 `Javassist`、`Byte Buddy` 这样的框架。

# 参考资料

* [网络通信优化之通信协议：如何优化RPC网络通信？](https://time.geekbang.org/column/article/100355) 
* [分布式通信之远程调用：我是你的千里眼](https://time.geekbang.org/column/article/160408)
* [RPC协议综述：远在天边，近在眼前](https://time.geekbang.org/column/article/12230)
* [核心原理：能否画张图解释下RPC的通信流程？](https://time.geekbang.org/column/article/199650)
* [既然有 HTTP 请求，为什么还要用 RPC 调用？](https://www.zhihu.com/question/41609070/answer/191965937)
