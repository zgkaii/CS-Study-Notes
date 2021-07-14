# 1. Dubbo 框架介绍

## 1.1 Dubbo发展历史

Dubbo 产生于阿里巴巴 B2B 的实际业务需要。 淘系的 HSF，随着 B2B 退市，团队分流，导致 Dubbo 停滞。 2013年以后，国内 Dubbo 存量用户一直最多，增量 Spring Cloud 后来居上（类似 IE 浏览器与 Chrome 浏览器）。 

 可以将Dubbo发展历史分为三个时期：

* 开源期（2011-2013，横空出世）：Dubbo 是阿里巴巴 B2B 开发的，2011年开源。 
* 沉寂期（2013-2017，潜龙在渊）：2013年到2017年，Dubbo 的维护程度很低。
* 复兴期（2017-2019，朝花夕拾）：2017年8月份重启维护，2018年2月加入 Apache 孵化器， 2019年5月顺利毕业。

如今，Dubbo在众多公司中使用，例如当当、京东等公司的服务化都是基于 Dubbo 实现的，四大行也都有使用。

## 1.2 Dubbo 的主要功能

Apache Dubbo 是一款高性能、轻量级的开源 Java 服务框架。它有六大核心功能：面向接口代理的**高性能 RPC 调用**，**智能负载均衡**，**服务自动注册和发现**，**高度可扩展能力**，运行期**流量调度**，可视化的**服务治理与运维**。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210621132853628.png" width="800px"/>
</div>

**基础功能：RPC调用**

* 多协议（序列化，传输，RPC）
* 服务注册发现
* 配置、元数据管理

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210621133327670.png" width="600px"/>
</div>

框架分层设计，可任意组装和扩展。图例说明：

- 图中小方块 Protocol, Cluster, Proxy, Service, Container, Registry, Monitor 代表层或模块，蓝色的表示与业务有交互，绿色的表示只对 Dubbo 内部交互。
- 图中背景方块 Consumer, Provider, Registry, Monitor 代表部署逻辑拓扑节点。
- 图中蓝色虚线为初始化时调用，红色虚线为运行时异步调用，红色实线为运行时同步调用。
- 图中只包含 RPC 的层，不包含 Remoting 的层，Remoting 整体都隐含在 Protocol 中。

**扩展功能：集群、高可用、管控**

* 集群，负载均衡
* 治理，路由
* 控制台，管理与监控

灵活扩展+简单易用，是 Dubbo 成功的秘诀。

# 2. Dubbo 技术原理

## 2.1 整体框架

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210621133544259.png" width="1000px"/>
</div>

> 绿色方框——接口；蓝色方框——类；红色箭头——调用流程。

**各层说明**

* **config 配置层**：对外配置接口，以 ServiceConfig, ReferenceConfig 为中心，可以直接初始化配置类， 也可以通过 spring 解析配置生成配置类。
* **proxy 服务代理层**：服务接口透明代理，生成服务的客户端 Stub 和服务器端 Skeleton, 以 ServiceProxy 为中心，扩展接口为 ProxyFactory。
* **registry 注册中心层**：封装服务地址的注册与发现，以服务 URL 为中心，扩展接口为 RegistryFactory, Registry, RegistryService。
* **cluster 路由层**：封装多个提供者的路由及负载均衡，并桥接注册中心，以 Invoker 为中心，扩展接口为 Cluster，Directory，Router，LoadBalance。
* **monitor 监控层**：RPC 调用次数和调用时间监控，以 Statistics 为中心，扩展接口为 MonitorFactory, Monitor, MonitorService。
* **protocol 远程调用层**：封装 RPC 调用，以 Invocation，Result 为中心，扩展接口为 Protocol， Invoker，Exporter。
* **exchange 信息交换层**：封装请求响应模式，同步转异步，以 Request，Response 为中心，扩展接口为 Exchanger，ExchangeChannel，ExchangeClient，ExchangeServe。
* **transport 网络传输层**：抽象 mina 和 netty 为统一接口，以 Message 为中心，扩展接口为 Channel， Transporter，Client，Server，Codec 。
* **serialize 数据序列化层**：可复用的一些工具，扩展接口为 Serialization，ObjectInput， ObjectOutput， ThreadPool。

**关系说明**

- 在 RPC 中，Protocol 是核心层，也就是只要有 Protocol + Invoker + Exporter 就可以完成非透明的 RPC 调用，然后在 Invoker 的主过程上 Filter 拦截点。
- 图中的 Consumer 和 Provider 是抽象概念，只是想让看图者更直观的了解哪些类分属于客户端与服务器端，不用 Client 和 Server 的原因是 Dubbo 在很多场景下都使用 Provider, Consumer, Registry, Monitor 划分逻辑拓普节点，保持统一概念。
- 而 Cluster 是外围概念，所以 Cluster 的目的是将多个 Invoker 伪装成一个 Invoker，这样其它人只要关注 Protocol 层 Invoker 即可，加上 Cluster 或者去掉 Cluster 对其它层都不会造成影响，因为只有一个提供者时，是不需要 Cluster 的。
- Proxy 层封装了所有接口的透明化代理，而在其它层都以 Invoker 为中心，只有到了暴露给用户使用时，才用 Proxy 将 Invoker 转成接口，或将接口实现转成 Invoker，也就是去掉 Proxy 层 RPC 是可以 Run 的，只是不那么透明，不那么看起来像调本地服务一样调远程服务。
- 而 Remoting 实现（Exchange+Exchange+Serialize ）是 Dubbo 协议的实现，如果你选择 RMI 协议，整个 Remoting 都不会用上，Remoting 内部再划为 Transport 传输层和 Exchange 信息交换层，Transport 层只负责单向消息传输，是对 Mina, Netty, Grizzly 的抽象，它也可以扩展 UDP 传输，而 Exchange 层是在传输层之上封装了 Request-Response 语义。
- Registry 和 Monitor 实际上不算一层，而是一个独立的节点，只是为了全局概览，用层的方式画在一起。

**调用链**

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210621134645571.png" width="800px"/>
</div>

## 2.2 SPI扩展实现

SPI (Service Provider Interface) 被称为服务提供接口，SPI 扩展接口仅用于系统集成，或 Contributor 扩展功能插件。

而API被称为应用程序接口，两者主要区别：

- The API is the description of classes/interfaces/methods/... that you *call and use* to achieve a goal.
- the SPI is the description of classes/interfaces/methods/... that you *extend and implement* to achieve a goal.

Dubbo 的 SPI 扩展，最关键的 SPI：`Protocol xxx=com.alibaba.xxx.XxxProtocol`。

### 2.2.1 服务如何暴露

以 InjvmProtocol 为例， 主要包含：InjvmProtocol、InjvmExporter、XX Invoker。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210621135658273.png" width="800px"/>
</div>

### 2.2.2 服务如何引用

ServiceReference为例，ReferenceConfig，createProxy 中创建 Invoker。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210621135912846.png" width="800px"/>
</div>

### 2.2.3 集群与路由

Cluster 

-- Directory : return List 

-- Router ： 选取此次调用可以提供服务的 invoker 集合 Condition，Script，Tag 

-- LoadBalance ：从上述集合选取一个作为最终调用者 Random，RoundRobin，ConsistentHash

### 2.2.4 泛化引用

GenericService 当我们知道接口、方法和参数，不用存根方式，而是用反射方式调用任何服务。 

方法1： 

```java
<dubbo:reference id="barService" interface="com.foo.BarService" generic="true" />

GenericService barService = (GenericService) applicationContext.getBean("barService"); 

Object result = barService.$invoke("sayHello", new String[] { "java.lang.String" }, new Object[] { "World" }); 
```

方法2： 

```java
ReferenceConfig<GenericService> reference = new ReferenceConfig<GenericService>();
reference.setInterface("com.xxx.XxxService");
reference.setVersion("1.0.0");
reference.setGeneric(true);
GenericService genericService = reference.get();
genericService.$invoke.......
```

### 2.2.5 隐式传参

Context 模式`RpcContext.getContext().setAttachment("index", "1");` 

此参数可以传播到 RPC 调用的整个过程。

### 2.2.6 mock

```java
<dubbo.reference id="helloService" interface="io.kmmking.HelloService" mock="true"
timeout="1000" check="false">
```

需要实现一个 io.kimmking.HelloServiceMock 类。可以方便用来做测试。

# 3. Dubbo 应用场景

**(1) 分布式服务化改造**

业务系统规模复杂，垂直拆分改造  

* 数据相关改造 -
* 服务设计 
* 不同团队的配合 
* 开发、测试运维

**(2) 开放平台**

平台发展的两个模式：开放模式、容器模式。 

将公司的业务能力开发出来，形成开发平台，对外输出业务或技术能力。 

API 与 SPI，分布式服务化与集中式 ESB

**(3) 直接作为前端使用的后端（BFF）** 

基于 Dubbo 实现 BFF 作为 BFF（Backend For Frontend）给前端（Web 或 Mobile）提供服务。 

一般不太建议这种用法。 

灵活性，更好的支持前台业务，向中台发展。

**(4) 通过服务化建设中台**

基于 Dubbo 实现业务中台，将公司的所有业务服务能力，包装成 API，形成所谓的业务中台。

前端业务服务，各个业务线，通过调用中台的业务服务，灵活组织自己的业务。

从而实现服务的服用能力，以及对于业务变化的快速响应。

# 4. Dubbo 最佳实践

**(1) 开发分包**

建议将**服务接口、服务模型、服务异常等均放在 API 包**中，因为服务模型和异常也是 API 的一部 分，这样做也符合分包原则：重用发布等价原则(REP)，共同重用原则(CRP)。 

**服务接口尽可能大粒度**，每个服务方法应代表一个功能，而不是某功能的一个步骤，否则将面临分 布式事务问题，Dubbo 暂未提供分布式事务支持。 

**服务接口建议以业务场景为单位划分**，并对相近业务做抽象，防止接口数量爆炸。 

**不建议使用过于抽象的通用接口**，如：Map query(Map)，这样的接口没有明确语义，会给后期维 护带来不便。

**(2) 环境隔离与分组** 

怎么做多环境的隔离？ 

1、部署多套？ 

2、多注册中心机制 

3、group 机制 

4、版本机制 

服务接口增加方法，或服务模型增加字段，可向后兼容，删除方法或删除字段，将不兼容，枚举类 型新增字段也不兼容，需通过变更版本号升级。

**(3) 参数配置** 

通用参数以 consumer 端为准，如果 consumer 端没有设置，使用 provider 数值。

建议在 Provider 端配置的 Consumer 端属性有： 

* timeout：方法调用的超时时间 retries：失败重试次数，缺省是 2 。
* loadbalance：负载均衡算法，缺省是随机 random。 
* actives：消费者端的最大并发调用限制，即当 Consumer 对一个服务的并发调用到上限后，新调用会阻塞直 到超时，可以配置在方法或服务上。 

建议在 Provider 端配置的 Provider 端属性有： 

* threads：服务线程池大小 
* executes：一个服务提供者并行执行请求上限，即当 Provider 对一个服务的并发调用达到上限后，新调用会 阻塞，此时 Consumer 可能会超时。可以配置在方法或服务上。

**(4) 容器化部署** 

注册的IP问题，容器内提供者使用的 IP，如果注册到 zk，消费者无法访问。 

两个解决办法： 

1、docker **使用宿主机网络** ：`docker xxx -net xxxxx` 

2、docker **参数指定注册的IP和端口**，`-e` 

* `DUBBO_IP_TO_REGISTRY` — 注册到注册中心的 IP 地址 
* `DUBBO_PORT_TO_REGISTRY` — 注册到注册中心的端口 
* `DUBBO_IP_TO_BIND` — 监听 IP 地址 
* `DUBBO_PORT_TO_BIND` — 监听端口

**(5) 运维与监控**

Admin 功能较简单，大规模使用需要定制开发，整合自己公司的运维监控系统。

可观测性：tracing、metrics、logging(ELK)

- APM(skywalking,pinpoint,cat,zipkin,,,) 
- Promethus+Grafana

**(6) 分布式事务** 

柔性事务，SAGA、TCC、AT

* Seata 
* hmily + dubbo

**(7) 重试与幂等** 

服务调用失败默认重试2次，如果接口不是幂等的，会造成业务重复处理。 

如何设计幂等接口？

1、去重-->(bitmap --> 16M)，100w 

2、类似乐观锁机制