Netty是一个异步事件驱动的网络应用程序框架，用于快速开发可维护的高性能协议服务器和客户端。

# 一、Netty架构设计

## 1.1 功能特性

Netty 的分层很清晰：

- 核心层，主要定义一些基础设施，比如可扩展事件模型、通用通信 API、支持零拷贝的ByteBuffer缓冲区。
- 传输服务层，主要定义一些通信的底层能力，或者说是传输协议的支持，比如 TCP、UDP、HTTP 隧道、虚拟机管道等。
- 协议支持层，支持 HTTP、Protobuf、二进制、文本、WebSocket等一系列常见协议， 还支持通过实行编码解码逻辑来实现自定义协议。

 <div align="center"> <img src="..\..\..\images\nio\netty框架.png" width="500px"></div>

## 1.2 模块组件



# 参考

* [一文理解Netty模型架构](https://juejin.cn/post/6844903712435994631)