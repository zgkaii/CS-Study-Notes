## Spring

Spring是一个**生态体系**（也可以说是技术体系），它包含了许多应用在特定场景的具体框架，如`Spring Framework、Spring Security、Spring Boot、Spring Cloud`等等，其中`Spring Framework`框架是整个生态的核心基础，其他框架都需要依赖`Spring Framework`提供的基础功能。

> Spring 官网：<https://spring.io/>。

## Spring Framework

Spring 是分层的 Java SE/EE 应用 full-stack **轻量级开源框架**，⽤于构建企业级应⽤的轻量级⼀站式解决⽅案，它以（Inverse Of Control： 反转控制）和 AOP（Aspect Oriented Programming：面向切面编程）为内核，提供了展现层 Spring MVC 和持久层 Spring JDBC 以及业务层事务管理等众多的企业级应用技术，还能整合开源世界众多著名的第三方框架和类库，逐渐成为使用最多的 Java EE 企业应用开源框架。

Spring 官网列出的Spring Framework的主要特征：

| Features                                                     |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [Core](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#spring-core) | IoC Container, Events, Resources, i18n, Validation, Data Binding, Type Conversion, SpEL, AOP. |
| [Testing](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testing) | Mock Objects, TestContext Framework, Spring MVC Test, WebTestClient. |
| [Data Access](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#spring-data-tier) | Transactions, DAO Support, JDBC, R2DBC, O/R Mapping, XML Marshalling. |
| [Web Servlet](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#spring-web) | Spring MVC, WebSocket, SockJS, STOMP Messaging.              |
| [Web Reactive](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#spring-webflux) | Spring WebFlux, WebClient, WebSocket.                        |
| [Integration](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#spring-integration) | Remoting, JMS, JCA, JMX, Email, Tasks, Scheduling, Caching.  |
| [Languages](https://docs.spring.io/spring-framework/docs/current/reference/html/languages.html#languages) | Kotlin, Groovy, Dynamic Languages.                           |

>我们一般说Spring框架实际指的都是 Spring Framework，这种说法很容易将两者混淆。

## Spring MVC

SpringMVC 是一种基于 Java 的实现 MVC 设计模型的请求驱动类型的轻量级 Web 框架，属于 Spring Framework的后续产品，已经融合在 Spring Web Flow 里面。Spring 框架提供了构建 Web 应用程序的全功能 MVC 模块。使用 Spring 可插入的 MVC 架构，从而在使用 Spring 进行 WEB 开发时，可以选择使用 Spring的 Spring MVC 框架或集成其他 MVC 开发框架，如 Struts1(现在一般不用)，Struts2 等。

SpringMVC 已经成为目前最主流的 MVC 框架之一，并且随着 Spring3.0 的发布，全面超越 Struts2，成为最优秀的 MVC 框架。

## Spring Boot

SpringBoot简化了Spring应用开发，整合了Spring技术栈， 去繁从简，`just run`就能创建一个**独立的**、生产级别的应用，为J2EE开发提供了一站式解决方案。

Spring 官网列出的Spring Boot的主要特征：

- 快速创建独立运行的Spring项目以及与主流框架集成
- 使用嵌入式的Tomcat，Jetty或Undertow容器，应用无需打成WAR包
- starters自动依赖与版本控制
- 大量的自动配置，简化开发，也可修改默认值
- 无需配置XML，无代码生成，开箱即用
- 准生产环境的运行时应用监控

## Spring Cloud

Spring Cloud事实上是一整套基于Spring Boot的微服务解决方案。它为开发者提供了很多工具，用于快速构建分布式系统的一些通用模式，例如：配置管理、注册中心、服务发现、限流、网关、链路追踪等。

<img src="https://img-blog.csdnimg.cn/2020122115550081.png" style="zoom:80%;" />

如下图所示，很好的说明了Spring Boot和Spring Cloud的关系，Spring Boot是build anything，而Spring Cloud是coordinate anything，Spring Cloud的每一个微服务解决方案都是基于Spring Boot构建的。

![](https://img-blog.csdnimg.cn/20201221140615747.png)



