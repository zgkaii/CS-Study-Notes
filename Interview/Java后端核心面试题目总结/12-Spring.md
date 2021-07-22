<!-- MarkdownTOC -->

- [Spring Framework](#spring-framework)
  - [什么是 Spring Framework？](#什么是-spring-framework)
  - [Spring Framework 有哪些重要模块？](#spring-framework-有哪些重要模块)
  - [使用 Spring 框架能带来哪些好处？](#使用-spring-框架能带来哪些好处)
  - [Spring 框架用到了哪些设计模式？](#spring-框架用到了哪些设计模式)
  - [Spring Bean 是什么？](#spring-bean-是什么)
- [Spring IoC](#spring-ioc)
  - [什么是 Spring IoC 容器？](#什么是-spring-ioc-容器)
  - [IoC 和 DI 有什么区别？](#ioc-和-di-有什么区别)
  - [依赖注入常见方式？](#依赖注入常见方式)
  - [Spring 中有多少种 IoC 容器？](#spring-中有多少种-ioc-容器)
  - [常用的 ApplicationContext 容器？](#常用的-applicationcontext-容器)
  - [列举一些 IoC 的一些好处？](#列举一些-ioc-的一些好处)
  - [IoC 容器的初始化过程？？](#ioc-容器的初始化过程)
  - [Spring 框架中有哪些不同类型的事件？](#spring-框架中有哪些不同类型的事件)
- [Spring AOP](#spring-aop)
  - [什么是 AOP ？](#什么是-aop-)
  - [JoinPoint 与 PointCut 的区别？](#joinpoint-与-pointcut-的区别)
  - [Advice有哪些类型？](#advice有哪些类型)
  - [AOP 有哪些实现方式？](#aop-有哪些实现方式)
  - [Spring AOP and AspectJ AOP 有什么区别？](#spring-aop-and-aspectj-aop-有什么区别)
- [Spring Beans](#spring-beans)
  - [Spring 有哪些配置方式?](#spring-有哪些配置方式)
  - [Spring 支持几种 Bean Scope ？](#spring-支持几种-bean-scope-)
  - [Spring Bean 的生命周期？](#spring-bean-的生命周期)
  - [什么是 Spring 的内部 bean？](#什么是-spring-的内部-bean)
  - [什么是 Spring 装配？](#什么是-spring-装配)
  - [解释什么叫延迟加载？](#解释什么叫延迟加载)
  - [Spring 框架中的单例 Bean 是线程安全的么？](#spring-框架中的单例-bean-是线程安全的么)
  - [Spring 如何解决循环依赖？](#spring-如何解决循环依赖)
- [Spring 注解](#spring-注解)
  - [什么是基于注解的容器配置？](#什么是基于注解的容器配置)
  - [如何在 Spring 中启动注解装配？](#如何在-spring-中启动注解装配)
  - [@Component, @Controller, @Repository, @Service 有何区别？](#component-controller-repository-service-有何区别)
  - [@Required 注解有什么用？](#required-注解有什么用)
  - [@Autowired 注解有什么用？原理？](#autowired-注解有什么用原理)
  - [@Resource与@Autowired区别？](#resource与autowired区别)
  - [@Qualifier 注解有什么用？原理？](#qualifier-注解有什么用原理)
  - [基于注解配置与基于XML配置的区别？](#基于注解配置与基于xml配置的区别)
- [Spring Transaction](#spring-transaction)
  - [什么是事务？有何特性？](#什么是事务有何特性)
  - [Spring 支持的事务管理类型？](#spring-支持的事务管理类型)
  - [Spring 中事务管理器是什么？](#spring-中事务管理器是什么)
  - [Spring 事务如何和不同的数据持久层框架做集成？](#spring-事务如何和不同的数据持久层框架做集成)
  - [为什么在 Spring 事务中不能切换数据源？](#为什么在-spring-事务中不能切换数据源)
  - [@Transactional 注解有哪些属性？如何使用？](#transactional-注解有哪些属性如何使用)
  - [@Transactional实现原理？](#transactional实现原理)
  - [什么是事务的隔离级别？](#什么是事务的隔离级别)
  - [什么是事务的传播级别？](#什么是事务的传播级别)
  - [什么是事务的超时属性与只读属性？](#什么是事务的超时属性与只读属性)
  - [什么是事务的回滚规则？](#什么是事务的回滚规则)
  - [@Transactional/事务失效场景有哪些？](#transactional事务失效场景有哪些)
- [Spring Data Access](#spring-data-access)
  - [Spring 支持哪些 ORM 框架？](#spring-支持哪些-orm-框架)
  - [在 Spring 框架中如何更有效地使用 JDBC ？](#在-spring-框架中如何更有效地使用-jdbc-)
  - [Spring 数据数据访问层有哪些异常？](#spring-数据数据访问层有哪些异常)
  - [使用 Spring 访问 Hibernate 的方法有哪些？](#使用-spring-访问-hibernate-的方法有哪些)

<!-- /MarkdownTOC -->

# Spring Framework

Spring是一个**生态体系**（也可以说是技术体系），它包含了许多应用在特定场景的具体框架，如`Spring Framework、Spring Security、Spring Boot、Spring Cloud`等等，其中`Spring Framework`框架是整个生态的核心基础，其他框架都需要依赖`Spring Framework`提供的基础功能。

> 我们常常提及的Spring/Spring框架，其实指的是Spring Framework。

## 什么是 Spring Framework？

Spring Framework是分层的 Java SE/EE 应用 full-stack **轻量级开源框架**，用于构建企业级应用的轻量级一站式解决方案，它以（Inverse Of Control： 反转控制）和 AOP（Aspect Oriented Programming：面向切面编程）为内核，提供了展现层 Spring MVC 和持久层 Spring JDBC 以及业务层事务管理等众多的企业级应用技术，还能整合开源世界众多著名的第三方框架和类库，逐渐成为使用最多的 Java EE 企业应用开源框架。

- 它是轻量级、松散耦合的。
- 它具有分层体系结构，允许用户选择组件，同时还为 J2EE 应用程序开发提供了一个有凝聚力的框架。
- 它可以集成其他框架，如 Spring MVC、Hibernate、MyBatis 等，所以又称为框架的框架( 粘合剂、脚手架 )。

## Spring Framework 有哪些重要模块？

如下是一张比较早期版本的 Spring Framework 的模块图：

<div align="center">  
<img src="http://static.iocoder.cn/images/Spring/2018-12-24/01.jpg" width="600px"/>
</div>

**Spring 核心容器——Core Container**是 Spring Framework 的核心。它包含以下模块：

- Spring Core：提供了框架的基本组成部分，包括控制反转（Inversion of Control，IOC）和依赖注入（Dependency Injection，DI）功能。

- Spring Beans：核心容器提供 Spring 框架的基本功能。核心容器的主要组件是 BeanFactory，它是工厂模式的实现。BeanFactory 使用控制反转 （IoC）模式将应用程序的配置和依赖性规范与实际的应用程序代码分开。

- Spring Context：Spring 上下文是一个配置文件，向 Spring 框架提供上下文信息。Spring 上下文包括企业服务，例如 JNDI、EJB、电子邮件、国际化、事件机制、校验和调度功能。

- SpEL (Spring Expression Language)：Spring 表达式语言全称为 “Spring Expression Language”，缩写为 “SpEL” ，类似于 Struts2 中使用的 OGNL 表达式语言，能在运行时构建复杂表达式、存取对象图属性、对象方法调用等等，并且能与 Spring 功能完美整合，如能用来配置 Bean 定义。


**数据访问——Data Access**提供与数据库交互的支持。它包含以下模块：

- JDBC (Java DataBase Connectivity)：Spring 对 JDBC 的封装模块，提供了对关系数据库的访问。

- ORM (Object Relational Mapping)：Spring ORM 模块，提供了对 hibernate5 和 JPA 的集成。

- OXM (Object XML Mappers)：Spring 提供了一套类似 ORM 的映射机制，用来将 Java 对象和 XML 文件进行映射。这就是 Spring 的对象 XML 映射功能，有时候也成为 XML 的序列化和反序列化。

- Transaction：Spring 简单而强大的事务管理功能，包括声明式事务和编程式事务。


**Web**层提供了创建 Web 应用程序的支持。它包含以下模块：

- WebMVC：MVC 框架是一个全功能的构建 Web 应用程序的 MVC 实现。通过策略接口，MVC 框架变成为高度可配置的，MVC 容纳了大量视图技术，其中包括 JSP、Velocity、Tiles、iText 和 POI。

- WebFlux：基于 Reactive 库的响应式的 Web 开发框架

- WebSocket：Websocket 提供了一个在 Web 应用中实现高效、双向通讯，需考虑客户端(浏览器)和服务端之间高频和低延时消息交换的机制。（一般的应用场景有：在线交易、网页聊天、游戏、协作、数据可视化等）。


**AOP**支持面向切面编程。它包含以下模块：

- AOP：通过配置管理特性，Spring AOP 模块直接将面向方面的编程功能集成到了 Spring 框架中。所以，可以很容易地使 Spring 框架管理的任何对象支持 AOP。Spring AOP 模块为基于 Spring 的应用程序中的对象提供了事务管理服务。通过使用 Spring AOP，不用依赖 EJB 组件，就可以将声明性事务管理集成到应用程序中。

- Aspects：该模块为与 AspectJ 的集成提供支持。

- Instrumentation该层为类检测和类加载器实现提供支持。


**其它**

- JMS (Java Messaging Service)：提供了Java消息服务，简化了 JMS API 的使用。

- Test：该模块为使用 JUnit 和 TestNG 进行测试提供支持。

- Messaging：该模块为 STOMP 提供支持。它还支持注解编程模型，该模型用于从 WebSocket 客户端路由和处理 STOMP 消息。


## 使用 Spring 框架能带来哪些好处？

下面列举了一些使用 Spring 框架带来的主要好处：

- **DI** ：**[Dependency Injection(DI)](http://howtodoinjava.com/2013/03/19/inversion-of-control-ioc-and-dependency-injection-di-patterns-in-spring-framework-and-related-interview-questions/)** 方法，使得构造器和 JavaBean、properties 文件中的依赖关系一目了然。
- **轻量级**：与 EJB 容器相比较，IoC 容器更加趋向于**轻量级**。这样一来 IoC 容器在有限的内存和 CPU 资源的情况下，进行应用程序的开发和发布就变得十分有利。
- **面向切面编程(AOP)**： Spring 支持面向**切面编程**，同时把应用的业务逻辑与系统的服务分离开来。
- **集成主流框架**：Spring 并没有闭门造车，Spring **集成**了已有的技术栈，比如 ORM 框架、Logging 日期框架、J2EE、Quartz 和 JDK Timer ，以及其他视图技术。
- 模块化：Spring 框架是按照**模块**的形式来组织的。由包和类的命名，就可以看出其所属的模块，开发者仅仅需要选用他们需要的模块即可。
- **便捷的测试**：要 [测试一项用Spring开发的应用程序](http://howtodoinjava.com/2013/04/19/how-to-unit-test-spring-security-authentication-with-junit/) 十分简单，因为**测试**相关的环境代码都已经囊括在框架中了。更加简单的是，利用 JavaBean 形式的 POJO 类，可以很方便的利用依赖注入来写入测试数据。
- **Web 框架**：Spring 的 **Web 框架**亦是一个精心设计的 Web MVC 框架，为开发者们在 Web 框架的选择上提供了一个除了主流框架比如 Struts 、过度设计的、不流行 Web 框架的以外的有力选项。
- **事务管理**：Spring 提供了一个便捷的**事务管理**接口，适用于小型的本地事物处理（比如在单 DB 的环境下）和复杂的共同事物处理（比如利用 JTA 的复杂 DB 环境）。
- **异常处理**：Spring 提供一个方便的 API ，将特定技术的异常(由JDBC，Hibernate，或 JDO 抛出)转化为一致的、Unchecked 异常。

当然，Spring 代码优点的同时，一定会带来相应的缺点：

- 每个框架都有的问题，调试阶段不直观，后期的 bug 对应阶段，不容易判断问题所在。要花一定的时间去理解它。

- 把很多 JavaEE 的东西封装了，在满足快速开发高质量程序的同时，隐藏了实现细节。


## Spring 框架用到了哪些设计模式？

Spring 框架中使用到了大量的设计模式，下面列举了比较有代表性的：

1. **工厂设计模式** ： Spring 使用工厂模式通过 BeanFactory、ApplicationContext 创建 Bean 对象；
2. **代理设计模式** ： Spring AOP 功能的实现； 
3. **单例设计模式** ： Spring 中的 Bean 默认都是单例的；
4. **模板方法模式** ： Spring 中 jdbcTemplate、hibernateTemplate 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式； 
5. **装饰器模式** ：我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源；
6. **观察者模式**：Spring 事件驱动模型就是观察者模式很经典的⼀个应用；
7. **适配器模式**：Spring AOP 的增强或通知(Advice)使用到了适配器模式、SpringMVC 中也是用到了适配器模式适配 Controller。
8. ... ...

## Spring Bean 是什么？

Bean是Spring中一个基础却重要的概念。Spring官方文档对bean的解释是：  

```java
In Spring, the objects that form the backbone of your application and that are managed by the Spring IoC container are called beans. A bean is an object that is instantiated, assembled, and managed by a Spring IoC container.
Otherwise, a bean is simply one of many objects in your application. Beans, and the dependencies among them, are reflected in the configuration metadata used by a container.
```

也就是说：

```java
在Spring中，构成应用程序的主体并由Spring IoC容器管理的对象统称为beans。 bean是一个由Spring IoC容器实例化、组装和管理的对象。
此外，bean只是应用程序中许多对象中的一个。Beans以及他们之间的依赖关系是通过容器配置元数据反映出来。
```

可以看出：bean是一个对象，bean由IoC容器管理，实际应用程序中有很多bean。

我们可以这样理解Spring Bean：

- **Bean 由 Spring IoC 容器实例化，装配和管理的对象，除了这点，它与应用程序中其他对象没有分别；**
- **Bean 的定义以及bean相互间依赖关系通过配置元数据（Bean Definition）来描述。**

# Spring IoC

## 什么是 Spring IoC 容器？

Spring 框架的核心是 Spring IoC (Inversion of Control)容器。容器创建 Bean 对象，将它们装配在一起，配置它们并管理它们的完整生命周期。

- Spring 容器使用**依赖注入**来管理组成应用程序的 Bean 对象。
- 容器通过读取提供的**配置元数据** Bean Definition 来接收对象进行实例化，配置和组装的指令。
- 该配置元数据 Bean Definition 可以通过 XML，Java 注解或 Java Config 代码**提供**。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210719104744359.png" width="600px"/>
</div>
## IoC 和 DI 有什么区别？

**什么是控制反转（IoC）？**

* 实际上，控制反转（Inversion Of Control）是一个比较笼统的设计思想，并不是一种具体的实现方法，一般用来指导框架层面的设计。这里所说的“控制”指的是对程序执行流程的控制，而“反转”指的是在没有使用框架之前，程序员自己控制整个程序的执行。在使用框架之后，整个程序的执行流程通过框架来控制。流程的控制权从程序员“反转”给了框架。

**什么是依赖注入（DI）？**

* 依赖注入（Dependency Injection）和控制反转恰恰相反，它是一种具体的编码技巧。我们不通过 new 的方式在类内部创建依赖类的对象，而是将依赖的类对象在外部创建好之后，通过构造函数、函数参数等方式传递（或注入）给类来使用。

**什么是依赖注入框架（DI Framework）？**

* 我们通过依赖注入框架提供的扩展点，简单配置一下所有需要的类及其类与类之间依赖关系，就可以实现由框架来自动创建对象、管理对象的生命周期、依赖注入等原本需要程序员来做的事情。
* 实际上，现成的依赖注入框架有很多，比如 Google Guice、Java Spring、Pico Container、Butterfly Container 等。Spring 框架自己声称是控制反转容器（Inversion Of Control Container）。

**IoC与DI的区别？**

* 依赖注入（DI）和控制反转（IoC）是从不同的角度描述的同一件事情，就是指通过引入IoC容器，利用依赖关系注入的方式，实现对象之间的解耦。其本质是用反射获取对象。

> **Dependency Injection**原来就叫 IoC 。但后来Martin Flower 发话，是个框架都有 IoC ，这不足以新生容器反转的“如何定位插件的具体实现”，于是，它有了个新名字，Dependency Injection 。
> 
> 其实，它就是一种将调用者与被调用者分离的思想，Uncle Bob 管它叫DIP（Dependency Inversion Principle），并把它归入OO设计原则。

## 依赖注入常见方式？

通常，依赖注入可以通过**三种**方式完成，即：

* 接口注入

- 构造函数注入
- setter 注入

目前，在 Spring Framework 中，仅使用构造函数和 setter 注入这**两种**方式。其中，setter注入更加常用：

| 构造函数注入               | setter 注入                |
| :------------------------- | :------------------------- |
| 没有部分注入               | 有部分注入                 |
| 不会覆盖 setter 属性       | 会覆盖 setter 属性         |
| 任意修改都会创建一个新实例 | 任意修改不会创建一个新实例 |
| 适用于设置很多属性         | 适用于设置少量属性         |

## Spring 中有多少种 IoC 容器？

Spring 提供了**两种**( 不是“个” ) IoC 容器，分别是 BeanFactory、ApplicationContext 。

**1. BeanFactory**

BeanFactory 由`Spring Beans`提供，它就像一个包含 Bean 集合的工厂类。它会在客户端要求时实例化 Bean 对象，也就是**懒加载**的方式，这意味着beans只有在我们通过`getBean()`方法直接调用它们时才进行实例化。实现BeanFactory最常用的API是`XMLBeanFactory`。

**2. ApplicationContext**

ApplicationContext 接口继承了 BeanFactory 接口，由`Spring Context` 提供，它在 BeanFactory 基础上提供了一些额外的功能，更容易集成Spring的AOP功能、资源访问、事件传播和特定的上下文应用层。

- MessageSource ：管理 message ，实现国际化等功能。
- ApplicationEventPublisher ：事件发布。
- ResourcePatternResolver ：多资源加载。
- EnvironmentCapable ：系统 Environment（profile + Properties）相关。
- Lifecycle ：管理生命周期。
- Closable ：关闭，释放资源
- InitializingBean：自定义初始化。
- BeanNameAware：设置 beanName 的 Aware 接口。

另外，ApplicationContext 加载方式是**预加载**，每一个bean都在Application启动之后实例化。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200910173222719.png" width="800px"/>
</div>

简单总结下 BeanFactory 与 ApplicationContext 两者的差异：

| BeanFactory                | ApplicationContext       |
| :------------------------- | :----------------------- |
| 它使用懒加载               | 它使用即时加载           |
| 它使用语法显式提供资源对象 | 它自己创建和管理资源对象 |
| 不支持国际化               | 支持国际化               |
| 不支持基于依赖的注解       | 支持基于依赖的注解       |

另外，BeanFactory 也被称为**低级**容器，而 ApplicationContext 被称为**高级**容器，推荐使用ApplicationContext 。

## 常用的 ApplicationContext 容器？

以下是三种较常见的 ApplicationContext 实现方式：

**1. ClassPathXmlApplicationContext** ：从类根目录ClassPath 的 XML配置文件中读取上下文（推荐使用）

```java
ApplicationContext context = new ClassPathXmlApplicationContext(“bean.xml”);
```

**2. FileSystemXmlApplicationContext** ：从磁盘路径上加载配置文件，配置文件可以在磁盘的任意位置。不推荐使用）

```java
ApplicationContext context = new FileSystemXmlApplicationContext(“bean.xml”);
```

**3. XmlWebApplicationContext** ：由 Web 应用的XML文件读取上下文。例如我们在 Spring MVC 使用的情况。

Spring Boot 使用的是第四种 ApplicationContext 容器**ConfigServletWebServerApplicationContext** 。

## 列举一些 IoC 的一些好处？

- 它将最小化应用程序中的代码量。
- 它以最小的影响和最少的侵入机制促进松耦合。
- 它支持即时的实例化和延迟加载 Bean 对象。
- 它将使您的应用程序易于测试，因为它不需要单元测试用例中的任何单例或 JNDI 查找机制。

## IoC 容器的初始化过程？？

简单来说，Spring 中的 IoC 的实现原理，就是**工厂模式**加**反射机制**。

> 在基友 [《面试问烂的 Spring IoC 过程》](http://www.iocoder.cn/Fight/Interview-poorly-asked-Spring-IOC-process-1/) 的文章中，把 Spring IoC 相关的内容，讲的非常不错。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200915131957564.png" width="600px"/>
</div>

Spring IoC容器在启动时：

- 第一步：从XML/注解中读取Bean配置信息Resource，将配置文件解析成BeanDefinition（承载Bean 的生命周期、销毁、初始化等操作），生产Bean定义注册表；
- 第二步：根据Bean注册表，使用Beanfactory（实例化、定位、配置应用程序中的对象及建立这些对象间的依赖关系）实例化Bean；
- 第三步：将Bean实例放到Spring容器Bean缓存池中，以供外层的应用程序进行调用。

## Spring 框架中有哪些不同类型的事件？

Spring 的 ApplicationContext 提供了支持事件和代码中监听器的功能。

我们可以创建 Bean 用来监听在 ApplicationContext 中发布的事件。如果一个 Bean 实现了 ApplicationListener 接口，当一个ApplicationEvent 被发布以后，Bean 会自动被通知。示例代码如下：

```java
public class AllApplicationEventListener implements ApplicationListener<ApplicationEvent> {  
    
    @Override  
    public void onApplicationEvent(ApplicationEvent applicationEvent) {  
        // process event  
    }
}
```

Spring 提供了以下五种标准的事件：

1. 上下文更新事件（ContextRefreshedEvent）：该事件会在ApplicationContext 被初始化或者更新时发布。也可以在调用ConfigurableApplicationContext 接口中的 `#refresh()` 方法时被触发。
2. 上下文开始事件（ContextStartedEvent）：当容器调用ConfigurableApplicationContext 的 `#start()` 方法开始/重新开始容器时触发该事件。
3. 上下文停止事件（ContextStoppedEvent）：当容器调用 ConfigurableApplicationContext 的 `#stop()` 方法停止容器时触发该事件。
4. 上下文关闭事件（ContextClosedEvent）：当ApplicationContext 被关闭时触发该事件。容器被关闭时，其管理的所有单例 Bean 都被销毁。
5. 请求处理事件（RequestHandledEvent）：在 We b应用中，当一个HTTP 请求（request）结束触发该事件。

------

除了上面介绍的事件以外，还可以通过扩展 ApplicationEvent 类来开发**自定义**的事件。

① 示例自定义的事件的类，代码如下：

```java
public class CustomApplicationEvent extends ApplicationEvent{  

    public CustomApplicationEvent(Object source, final String msg) {  
        super(source);
    }  

}
```

② 为了监听这个事件，还需要创建一个监听器。示例代码如下：

```java
public class CustomEventListener implements ApplicationListener<CustomApplicationEvent> {

    @Override  
    public void onApplicationEvent(CustomApplicationEvent applicationEvent) {  
        // handle event  
    }
    
}
```

③ 之后通过 ApplicationContext 接口的 `#publishEvent(Object event)` 方法，来发布自定义事件。示例代码如下：

```java
// 创建 CustomApplicationEvent 事件
CustomApplicationEvent customEvent = new CustomApplicationEvent(applicationContext
                                                                , "Test message");
// 发布事件
applicationContext.publishEvent(customEvent);
```

# Spring AOP

## 什么是 AOP ？

AOP(Aspect-Oriented Programming)，即**面向切面编程**，它与 OOP( Object-Oriented Programming, 面向对象编程) 相辅相成，提供了与 OOP 不同的抽象软件结构的视角。可以把AOP理解为OOP的一种横向扩展，其将与核心业务无关的代码，如**日志记录、性能监控、事务处理**等从业务逻辑代码中抽离出来，进行横向排列，从而实现低耦合，提高开发效率。

- 在 OOP 中，以类( Class )作为基本单元
- 在 AOP 中，以**切面( Aspect )**作为基本单元。

AOP主要的术语有：

| 名称                | 说明                                                                                                                                                                           |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Advice（通知）      | 例如日志记录、性能监控、事务处理等增强的功能。                                                                                                                                 |
| Joinpoint（连接点） | 具体执行增强的地方。例如方法调用前、调用后和方法捕获异常后。                                                                                                                   |
| Pointcut（切入点）  | 匹配连接点的正则表达式，是实现增强的具体连接点的集合。就是提供一组规则(使用 AspectJ PointCut expression language 来描述) 来匹配 JoinPoint ，给满足规则的 JoinPoint 添加 Advice |
| Aspect（切面）      | 切入点和通知的结合，使用 @Aspect 注解的类就是切面。                                                                                                                            |
| Introduction(引介)  | 一种特殊的通知，可以在运行期为类动态地添加一些方法或 Field。                                                                                                                   |
| Proxy（代理）       | 一个类被 AOP 织入增强后，就产生一个结果代理类。                                                                                                                                |
| Target（目标）      | 指代理的目标对象。注意Advised Object 指的不是原来的对象，而是织入 Advice 后所产生的代理对象。                                                                                  |
| Weaving（织入）     | 把切面应用到目标对象，并创建代理对象的过程。                                                                                                                                   |

## JoinPoint 与 PointCut 的区别？

JoinPoint 和 PointCut 本质上就是**两个不同纬度上**的东西。

- 在 Spring AOP 中，所有的方法执行都是 JoinPoint 。而 PointCut 是一个描述信息，它修饰的是 JoinPoint ，通过 PointCut ，我们就可以确定哪些 JoinPoint 可以被织入 Advice 。
- Advice 是在 JoinPoint 上执行的，而 PointCut 规定了哪些 JoinPoint 可以执行哪些 Advice 。

或者，我们在换一种说法：

1. 首先，Advice 通过 PointCut 查询需要被织入的 JoinPoint 。
2. 然后，Advice 在查询到 JoinPoint 上执行逻辑。

## Advice有哪些类型？

Advice ，**通知**。

- 特定 JoinPoint 处的 Aspect 所采取的动作称为 Advice 。
- Spring AOP 使用一个 Advice 作为拦截器，在 JoinPoint “周围”维护一系列的**拦截器**。

**有哪些类型的 Advice？**

| 名称                                | 说明                                                                                                                       |
| ----------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| MethodBeforeAdvice（前置通知）      | 在 JoinPoint 方法之前执行，并使用 `@Before` 注解标记进行配置。                                                             |
| AfterReturningAdvice（后置通知）    | 在JoinPoint方法正常执行后执行，并使用 `@AfterReturning` 注解标记进行配置，可以应用于关闭流、上传文件、删除临时文件等功能。 |
| MethodInterceptor（环绕通知）       | 在JoinPoint之前和之后执行，并使用 `@Around` 注解标记进行配置，可以应用于日志、事务管理等功能。                             |
| ThrowsAdvice（异常通知）            | 仅在 JoinPoint 方法通过抛出异常退出并使用 `@AfterThrowing` 注解标记配置时执行，可以应用于处理异常记录日志等功能。          |
| IntroductionInterceptor（引介通知） | 在目标类中添加一些新的方法和属性，可以应用于修改旧版本程序（增强类）。                                                     |

相关的注解：

| 名称            | 说明                                                            |
| --------------- | --------------------------------------------------------------- |
| @Aspect         | 用于定义一个切面。                                              |
| @Before         | 用于定义前置通知，相当于 BeforeAdvice。                         |
| @AfterReturning | 用于定义后置通知，相当于 AfterReturningAdvice。                 |
| @Around         | 用于定义环绕通知，相当于MethodInterceptor。                     |
| @AfterThrowing  | 用于定义抛出通知，相当于ThrowAdvice。                           |
| @After          | 用于定义最终final通知，不管是否异常，该通知都会执行。           |
| @DeclareParents | 用于定义引介通知，相当于IntroductionInterceptor（不要求掌握）。 |

看起来，是不是和拦截器的执行时间，有几分相似。实际上，用于拦截效果的各种实现，大体都是类似的。

## AOP 有哪些实现方式？

实现 AOP 的技术，主要分为两大类：

**1. 静态代理** - 指使用 AOP 框架提供的命令进行编译，从而在编译阶段就可生成 AOP 代理类，因此也称为编译时增强。

- 编译时编织（特殊编译器实现）

- 类加载时编织（特殊的类加载器实现）。

  > 例如，SkyWalking 基于 Java Agent 机制，配置上 ByteBuddy 库，实现类加载时编织时增强，从而实现链路追踪的透明埋点。

**2. 动态代理** - 在运行时在内存中“临时”生成 AOP 动态代理类，因此也被称为运行时增强。目前 Spring 中使用了两种动态代理方式：JDK 动态代理与CGLIB 动态代理。

Spring AOP**默认**为 JDK 动态代理，这使得任何接口（或者接口的集合）可以被代理；Spring AOP 也可以使用 CGLIB 代理，这对代理类而不是接口是必须的。

* JDK 动态代理通过反射来接收被代理的类，并且要求被代理的类必须实现一个接口。JDK动态代理的核心是 InvocationHandler 接口和 Proxy 类。
* 如果目标类没有实现接口，那么 Spring AOP 会选择使用 CGLIB 来动态代理目标类。当然，Spring 也支持配置，**强制**使用 CGLIB 动态代理。CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成某个类的子类，注意，CGLIB 是通过继承的方式做的动态代理，因此如果某个类被标记为 `final` ，那么它是无法使用 CGLIB 做动态代理的。

两种代理模式的优缺点：

- JDK代理-生成的代理类只有一个，而由于被代理的目标类是动态传入代理类中的，JDK代理的执行效率相对来说低一点，这也是JDK代理被称为动态代理的原因；

- CGLIB 代理需要为每个目标类生成相应的子类，因而在实际运行过程中，其可能会生成非常多的子类，过多的子类始终不是太好的，因为这影响了虚拟机编译类的效率；但由于在调用过程中，代理类的方法是已经静态编译生成了的，因而CGLIB 代理的执行效率相对来说高一些。

## Spring AOP and AspectJ AOP 有什么区别？

| 比较       | Spring AOP                                                                                               | AspectJ                                                                                                                      |
| ---------- | -------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| 能力和目标 | 旨在通过Spring IoC提供一个简单的AOP实现。<br>不是一个完整的AOP解决方案，只能用于被Spring容器管理的bean。 | 完整的AOP解决方案，比Spirng AOP复杂。<br>能够被应用于所有的领域对象。                                                        |
| 织入时机   | 只能在运行时织入                                                                                         | 可以在编译时，加载期，编译后（jar包和字节码文件）。                                                                          |
| 代理       | 使用**JDK动态代理或者CGLIB动态代理**。                                                                   | 静态代理，不依赖任何运行时环境，只需要引入编译器AspectJ compiler(ajc)完成织入。                                              |
| 连接点     | 使用代理模式，所以不能作用于final类、static方法、final方法。<br>**只支持方法执行连接点**。               | 没有限制。方法调用，方法执行，构造器调用，构造器执行，静态初始化执行，对象初始化，成员引用，成员赋值，处理器执行，通知执行。 |
| 复杂度     | 不需要额外的编译器，只能和Spirng管理的bean一起工作 AOP                                                   | 需要引入AJC编译器并重新打包                                                                                                  |
| 性能       | 基于代理的框架，运行时会有目标类的代理对象生成。                                                         | 在应用执行前织入切面到代码中，没有额外的运行时开销。                                                                         |

# Spring Beans

## Spring 有哪些配置方式?

单纯从 Spring Framework 提供的方式，一共有三种：

**1. XML 配置文件**

Bean 所需的依赖项和服务在 XML 格式的配置文件中指定。这些配置文件通常包含许多 bean 定义和特定于应用程序的配置选项。它们通常以 bean 标签开头。例如：

```java
<bean id="studentBean" class="org.edureka.firstSpring.StudentBean">
    <property name="name" value="Edureka"></property>
</bean>
```

**2. 注解配置**

您可以通过在相关的类，方法或字段声明上使用注解，将 Bean 配置为组件类本身，而不是使用 XML 来描述 Bean 装配。默认情况下，Spring 容器中未打开注解装配。因此，您需要在使用它之前在 Spring 配置文件中启用它。例如：

```java
<beans><context:annotation-config/><!-- bean definitions go here --></beans>
```

**3. Java Config 配置**

Spring 的 Java 配置是通过使用 @Bean 和 @Configuration 来实现。

- `@Bean` 注解扮演与 `<bean />` 元素相同的角色。

- `@Configuration` 类允许通过简单地调用同一个类中的其他 `@Bean` 方法来定义 Bean 间依赖关系。


 Spring Boot主要使用 **Java Config** 配置为主。当然，三种配置方式是可以混合使用的。

## Spring 支持几种 Bean Scope ？

Spring Bean 有五个作用域，其中最基础的有下面两种：

- Singleton，这是 Spring 的**默认**作用域，也就是为每个 IOC 容器创建唯一的一个 Bean 实例。
- Prototype，针对每个 getBean 请求，容器都会单独创建一个 Bean 实例。

从 Bean 的特点来看，Prototype 适合有状态的 Bean，而 Singleton 则更适合无状态的情况。另外，使用 Prototype 作用域需要经过仔细思考，毕竟频繁创建和销毁 Bean 是有明显开销的。

如果是 Web 容器，则支持另外三种作用域：

- Request，为每个 HTTP 请求创建单独的 Bean 实例。
- Session，很显然 Bean 实例的作用域是 Session 范围。
- GlobalSession，用于 Portlet 容器，因为每个 Portlet 有单独的 Session，GlobalSession 提供一个全局性的 HTTP Session。

仅当用户使用支持 Web 的 ApplicationContext 时，**最后三个才可用**。

## Spring Bean 的生命周期？

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210719114119572.png" width="600px"/>
    <div>Spring Bean生命周期</div>
</div>

Spring Bean 生命周期比较复杂，大致可以分为实例化、属性赋值、初始化 、销毁四个阶段：

**1. 实例化 Bean 对象**

- Spring 容器根据配置中的 Bean Definition(定义)，利用Java Reflection API**实例化** Bean 对象。

  > Bean Definition 可以通过 XML，Java 注解或 Java Config 代码提供。

- Spring 使用依赖注入（一般setter注入）**填充**所有属性，如 Bean 中所定义的配置。

**2. Aware 相关的属性，注入到 Bean 对象**

- 如果 Bean 实现 **BeanNameAware** 接口，则工厂通过传递 Bean 的 beanName 来调用 `#setBeanName(String name)` 方法。
- 如果Bean实现了**BeanClassLoaderAware**接口，调用`#setBeanClassLoader(ClassLoader classLoader)`方法，传入`ClassLoader`对象的实例。
- 如果 Bean 实现 **BeanFactoryAware** 接口，则工厂通过传递自身的实例来调用 `#setBeanFactory(BeanFactory beanFactory)` 方法。
- 与上面的类似，如果实现了其他`*Aware`接口，就调用相应的方法。

**3. 初始化 Bean 对象**

- 如果存在与 Bean 关联的任何 **BeanPostProcessor** 们，则调用 `#preProcessBeforeInitialization(Object bean, String beanName)` 方法。
- 如果 Bean 实现 **InitializingBean** 接口，则会调用 `#afterPropertiesSet()` 方法。
- 如果为 Bean 指定了 **init** 方法（例如 `<bean />` 的 `init-method` 属性），那么将调用该方法。
- 如果存在与 Bean 关联的任何 **BeanPostProcessor** 们，则将调用 `#postProcessAfterInitialization(Object bean, String beanName)` 方法。
- 创建过程完毕。

**4. 销毁 Spring Bean**

- 如果 Bean 实现 **DisposableBean** 接口，当 spring 容器关闭时，会调用 `#destroy()` 方法。
- 如果 Bean 指定了自定义的 **destroy** 方法（例如 `<bean />` 的 `destroy-method` 属性），那么将调用该方法。

## 什么是 Spring 的内部 bean？

只有将 Bean **仅**用作另一个 Bean 的属性时，才能将 Bean 声明为内部 Bean。

- 为了定义 Bean，Spring 提供基于 XML 的配置元数据在 `<property>`或 `<constructor-arg>` 中提供了 `<bean>`元素的使用。
- 内部 Bean 总是**匿名**的，并且它们总是作为**原型 Prototype** 。

例如，假设我们有一个 Student 类，其中引用了 Person 类。这里我们将只创建一个 Person 类实例并在 Student 中使用它。示例代码如下：

```java
// Student.java
public class Student {
    private Person person;
    // ... Setters and Getters
}

// Person.java
public class Person {
    private String name;
    private String address;
    // ... Setters and Getters
}
```

```java
<!-- bean.xml -->

<bean id=“StudentBean" class="com.edureka.Student">
    <property name="person">
        <!--This is inner bean -->
        <bean class="com.edureka.Person">
            <property name="name" value=“Scott"></property>
            <property name="address" value=“Bangalore"></property>
        </bean>
    </property>
</bean>
```

## 什么是 Spring 装配？

当 Bean 在 Spring 容器中组合在一起时，它被称为**装配**或 **Bean 装配**。Spring 容器需要知道需要什么 Bean 以及容器应该如何使用依赖注入来将 Bean 绑定在一起，同时装配 Bean 。

**自动装配有哪些方式？**

Spring 容器能够自动装配 Bean 。也就是说，可以通过检查 BeanFactory 的内容让 Spring 自动解析 Bean 的协作者。

自动装配的不同模式：

- no - 这是默认设置，表示没有自动装配。应使用显式 Bean 引用进行装配。
- byName - 它根据 Bean 的名称注入对象依赖项。它匹配并装配其属性与 XML 文件中由相同名称定义的 Bean 。
- 【最常用】**byType** - 它根据类型注入对象依赖项。如果属性的类型与 XML 文件中的一个 Bean 类型匹配，则匹配并装配属性。
- 构造函数 - 它通过调用类的构造函数来注入依赖项。它有大量的参数。
- autodetect - 首先容器尝试通过构造函数使用 autowire 装配，如果不能，则尝试通过 byType 自动装配。

**自动装配有什么局限？**

- 覆盖的可能性 - 您始终可以使用 `<constructor-arg>` 和 `<property>` 设置指定依赖项，这将覆盖自动装配。

- 基本元数据类型 - 简单属性（如原数据类型，字符串和类）无法自动装配。

- 令人困惑的性质 - 总是喜欢使用明确的装配，因为自动装配不太精确。

## 解释什么叫延迟加载？

默认情况下，容器启动之后会将所有作用域为**单例**的 Bean 都创建好，但是有的业务场景我们并不需要它提前都创建好。此时，我们可以在Bean 中设置 `lzay-init = "true"` 。

- 这样，当容器启动之后，作用域为单例的 Bean ，就不在创建。
- 而是在获得该 Bean 时，才真正在创建加载。

## Spring 框架中的单例 Bean 是线程安全的么？

通常，通常情况下 60% 的业务代码是三层架构，数据经过无状态的 Controller、Service、Repository 流转到数据库，默认情况下 Controller、Service、Repository 是单例的。Spring 框架并没有对这些单例 Bean 进行任何多线程的封装处理，关于单例 Bean 的线程安全和并发问题，需要开发者自行去搞定。

如果你的 Bean 有多种状态的话，就需要自行保证线程安全。**对于Web项目，最浅显的解决办法，就是将多态 Bean 的作用域( Scope )由 Singleton 变更为 Prototype 或者Request；非Web项目将 Singleton 变更为 Prototype** 。

当然也可以通过加锁的方法（例如synchronized）来解决线程安全，但这类方式一般性能消耗大，显然是不实际的。

这里需要注意的是，TreadLocal可以实现线程隔离，但还是无法保证并发操作下共享变量的线程安全（使用前三思）。

下面这个例子：

```java
@Controller
public class HomeController {
    private int i;
    @GetMapping("testsingleton1")
    @ResponseBody
    public int test1() {
        return ++i;
    }
}
```

最好的处理方式就是：**尽量避免使用成员变量**。

```javascript
@Controller
public class HomeController {
    @GetMapping("testsingleton1")
    @ResponseBody
    public int test1() {
         int i = 0;
         // TODO biz code
         return ++i;
    }
}
```

如果在特殊情况下，非要使用成员变量，那怎么保证单例bean的线程安全呢？

* **使用并发安全的类**：Java作为功能性超强的编程语言，API丰富，如果非要在单例bean中使用成员变量，可以考虑使用并发安全的容器，如ConcurrentHashMap、ConcurrentHashSet等等，将我们的成员变量（一般可以是当前运行中的任务列表等这类变量）包装到这些并发安全的容器中进行管理即可。
* **分布式或微服务的并发安全**：如果还要进一步考虑到微服务或分布式服务的影响，上面这个方法便不足以应付了，可以借助共享某些信息的分布式缓存中间件如Redis等，来保证同一种服务的不同服务实例都拥有同一份共享信息（如当前运行中的任务列表等这类变量）。

## Spring 如何解决循环依赖？

> 强烈推荐[Spring AOP中循环依赖解决 - 简书](https://www.jianshu.com/p/3bc6c6713b08)、[图解Spring解决循环依赖](https://juejin.cn/post/6844904122160775176)、[烂大街的 Spring 循环依赖问题，你觉得自己会了吗](https://juejin.cn/post/6870391753581690894)、[面试必杀技，讲一讲Spring中的循环依赖](https://developer.aliyun.com/article/766880)文章。

Spring中循环依赖大致有三种场景：

1. 构造器的循环依赖；
2. 单例模式下setter注入的循环依赖；
3. 非单例模式的循环依赖。

对于构造器的循环依赖与非单例模式下的循环依赖，Spring 是无法解决的，只能抛出 `BeanCurrentlyInCreationException` 异常；而对于单例模式下的setter循环依赖，**可以通过提前曝光机制，配合”三级缓存“来解决**。

**Spring是如何解决的循环依赖的？**

* Spring通过三级缓存解决了循环依赖，其中一级缓存为单例池（`singletonObjects`），二级缓存为早期曝光对象`earlySingletonObjects`，三级缓存为早期曝光对象工厂（`singletonFactories`）。
* 当A、B两个类发生循环引用时，在A完成实例化后，就使用实例化后的对象去创建一个对象工厂，并添加到三级缓存中，如果A被AOP代理，那么通过这个工厂获取到的就是A代理后的对象，如果A没有被AOP代理，那么这个工厂获取到的就是A实例化的对象。当A进行属性注入时，会去创建B，同时B又依赖了A，所以创建B的同时又会去调用getBean(a)来获取需要的依赖，此时的getBean(a)会从缓存中获取，第一步，先获取到三级缓存中的工厂；第二步，调用对象工工厂的getObject方法来获取到对应的对象，得到这个对象后将其注入到B中。紧接着B会走完它的生命周期流程，包括初始化、后置处理器等。当B创建完后，会将B再注入到A中，此时A再完成它的整个生命周期。至此，循环依赖结束！

**为什么要使用三级缓存呢？二级缓存能解决循环依赖吗？**

* 如果没有 AOP 代理，二级缓存可以解决问题，但是有 AOP 代理的情况下，只用二级缓存就意味着所有 Bean 在实例化后就要完成 AOP 代理，这样违背了 Spring 设计的原则，Spring 在设计之初就是通过 `AnnotationAwareAspectJAutoProxyCreator` 这个后置处理器来在 Bean 生命周期的最后一步来完成 AOP 代理，而不是在实例化后就立马进行 AOP 代理。

------

Spring中创建bean的过程大体可以分为`初始化bean`，`对bean的属性进行填充`，`对bean进行初始化`三个步骤：

- 实例化bean：即new一个bean实例，是通过反射调用构造器实现的；
- 对bean的属性进行填充：可以理解为对标签相应的属性进行赋值；
- 对bean进行初始化：即调用事先配置好的`init-method`方法，所以可以将一些初始化的行为写到这个方法中；

如果说，有这样的一个循环依赖的Demo：

```java
@Component
public class A {
    // A中注入了B
    @Autowired
    private B b;
}

@Component
public class B {
    // B中也注入了A
    @Autowired
    private A a;
}
```

**Spring在创建Bean的时候默认是按照自然排序来进行创建的，所以第一步Spring会去创建A**。创建A的过程实际上就是调用`getBean`方法，这个方法有两层含义:

* 创建一个新的bean实例；
* 从缓存中查询是否以有创建的实例。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210719210005591.png" width="600px"/>
</div>

其中，主要是通过 `getSingleton(beanName)` 方法从缓存中查找是不是有实例 sharedInstance（单例在 Spring 的同一容器只会被创建一次，后续再获取 bean，就直接从缓存获取即可）

这里就用到”三级缓存“了，大致作用是：

* **singletonObjects**：第一级缓存，存放成品的Bean（单例对象），即实例化、初始化后的Bean。
* **earlySingletonObjects**：第二级缓存，存放半成品的Bean（singletonObject），即已实例化但未注入属性和初始化的Bean。
* **singletonFactories**：第三级缓存，存放Bean工厂对象（singletonObject），用来生成半成品的Bean并放入到二级缓存中。

查看`getSingleton(beanName)` 源码，首先从三级缓存中试着获取 bean：

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    // 从 singletonObjects 获取实例，singletonObjects 中的实例都是准备好的 bean 实例，可以直接使用
    Object singletonObject = this.singletonObjects.get(beanName);
    //isSingletonCurrentlyInCreation() 判断当前单例bean是否正在创建中
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            // 一级缓存没有，就去二级缓存找
            singletonObject = this.earlySingletonObjects.get(beanName);
            if (singletonObject == null && allowEarlyReference) {
                // 二级缓存也没有，就去三级缓存找
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    // 三级缓存有的话，就把它移动到二级缓存
                    // getObject() 匿名内部类的实现真正调用createBean()
                    singletonObject = singletonFactory.getObject();
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return singletonObject;
}
```

以单例对象为例，再看下创建 bean 的逻辑（大括号表示内部类调用方法）：

1. 创建 bean 从以下代码开始，一个匿名内部类方法参数（总觉得 Lambda 的方式可读性不如内部类好理解）

```java
if (mbd.isSingleton()) {
    sharedInstance = getSingleton(beanName, () -> {
        try {
            return createBean(beanName, mbd, args);
        }
        catch (BeansException ex) {
            destroySingleton(beanName);
            throw ex;
        }
    });
    bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
}
```

`getSingleton()` 方法内部主要有两个方法：

```java
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
    // 创建 singletonObject
	singletonObject = singletonFactory.getObject();
    // 将 singletonObject 放入缓存
    addSingleton(beanName, singletonObject);
}
```

2. getObject()匿名内部类的实现真正调用的又是createBean(beanName, mbd, args)
3. 往里走，主要的实现逻辑在 `doCreateBean`方法，先通过 `createBeanInstance` 创建一个原始 bean 对象
4. 接着 `addSingletonFactory` 添加 bean 工厂对象到 singletonFactories 缓存（三级缓存）
5. 通过 `populateBean` 方法向原始 bean 对象中填充属性，并解析依赖，假设这时候创建 A 之后填充属性时发现依赖 B，然后创建依赖对象 B 的时候又发现依赖 A，还是同样的流程，又去 `getBean(A)`，这个时候三级缓存已经有了 beanA 的“半成品”，这时就可以把 A 对象的原始引用注入 B 对象（并将其移动到二级缓存）来解决循环依赖问题。这时候 `getObject()` 方法就算执行结束了，返回完全实例化的 bean
6. 最后调用 `addSingleton` 把完全实例化好的 bean 对象放入 singletonObjects 缓存（一级缓存）中。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210719222549284.png" width="1000px"/>
</div>

结合着上图我们再看下具体细节，用大白话再捋一捋：

1. Spring 创建 bean 主要分为两个步骤，创建原始 bean 对象，接着去填充对象属性和初始化；
2. 每次创建 bean 之前，我们都会从缓存中查下有没有该 bean，因为是单例，只能有一个；
3. 当我们创建 beanA 的原始对象后，并把它放到三级缓存中，接下来就该填充对象属性了，这时候发现依赖了 beanB，接着就又去创建 beanB，同样的流程，创建完 beanB 填充属性时又发现它依赖了 beanA，又是同样的流程，不同的是，这时候可以在三级缓存中查到刚放进去的原始对象 beanA，所以不需要继续创建，用它注入 beanB，完成 beanB 的创建
4. 既然 beanB 创建好了，所以 beanA 就可以完成填充属性的步骤了，接着执行剩下的逻辑，闭环完成

这就是单例模式下 Spring 解决循环依赖的流程了。

如果循环依赖的时候，所有类又都需要`Spring AOP自动代理`，那Spring如何提前曝光？曝光的是`原始bean`还是`代理后的bean`？

当 Spring 中存在该后置处理器，所有的单例 bean 在实例化后都会被进行提前曝光到三级缓存中，但是并不是所有的 bean 都存在循环依赖，也就是三级缓存到二级缓存的步骤不一定都会被执行，有可能曝光后直接创建完成，没被提前引用过，就直接被加入到一级缓存中。因此可以确保只有提前曝光且被引用的 bean 才会进行该后置处理。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210719222635140.png" width="1000px"/>
</div>

**AOP 代理对象提前放入了三级缓存，没有经过属性填充和初始化，这个代理又是如何保证依赖属性的注入的呢？**

这个又涉及到了 Spring 中动态代理的实现，不管是`cglib`代理还是`jdk`动态代理生成的代理类，代理时，会将目标对象 target 保存在最后生成的代理 `$proxy` 中，当调用 `$proxy` 方法时会回调 `h.invoke`，而 `h.invoke` 又会回调目标对象 target 的原始方法。所有，其实在 AOP 动态代理时，原始 bean 已经被保存在 **提前曝光代理**中了，之后 `原始 bean` 继续完成`属性填充`和`初始化`操作。因为 AOP 代理`$proxy `中保存着 `traget` 也就是是 `原始bean` 的引用，因此后续 `原始bean` 的完善，也就相当于Spring AOP中的 `target` 的完善，这样就保证了 AOP 的`属性填充`与`初始化`了！

# Spring 注解

## 什么是基于注解的容器配置？

不使用 XML 来描述 Bean 装配，开发人员通过在相关的类，方法或字段声明上使用**注解**将配置移动到组件类本身。它可以作为 XML 设置的替代方案。例如：

Spring 的 Java 配置是通过使用 `@Bean` 和 `@Configuration` 来实现。

- `@Bean` 注解，扮演与 `<bean />` 元素相同的角色。
- `@Configuration` 注解的类，允许通过简单地调用同一个类中的其他 `@Bean` 方法来定义 Bean 间依赖关系。

示例如下：

```java
@Configurationpublic class StudentConfig {    
    @Bean    
    public StudentBean myStudent() {        
        return new StudentBean();    
    }    
}
```

## 如何在 Spring 中启动注解装配？

默认情况下，Spring 容器中未打开注解装配。因此，要使用基于注解装配，我们必须通过配置 `<context：annotation-config />` 元素在 Spring 配置文件中启用它。

当然，如果是使用的Spring Boot ，默认情况下已经开启。

## @Component, @Controller, @Repository, @Service 有何区别？

我们一般使用 `@Autowired` 注解让 Spring 容器帮我们自动装配 bean。要想把类标识成可用于 `@Autowired` 注解自动装配的 bean 的类，可以采用以下注解实现：

- `@Component` ：它将 Java 类标记为 Bean 。它是任何 Spring 管理组件的**通用**构造型。
- `@Controller` ：它将一个类标记为 Spring Web MVC **控制器**。
- `@Service` ：此注解是组件注解的特化。它不会对 `@Component` 注解提供任何其他行为。您可以在**服务层**类中使用 @Service 而不是 `@Component` ，因为它以更好的方式指定了意图。
- `@Repository` ：这个注解是具有类似用途和功能的 `@Component` 注解的特化。它为 **DAO** 提供了额外的好处。它将 DAO 导入 IoC 容器，并使未经检查的异常有资格转换为 Spring DataAccessException 。

## @Required 注解有什么用？

`@Required` 注解，应用于 Bean 属性 setter 方法。

- 此注解仅指示必须在配置时使用 Bean 定义中的显式属性值或使用自动装配填充受影响的 Bean 属性。
- 如果尚未填充受影响的 Bean 属性，则容器将抛出 BeanInitializationException 异常。

示例代码如下：

```java
public class Employee {    
    private String name;        
    @Required    
    public void setName(String name){        
        this.name=name;    
    }        
    public string getName(){        
        return name;    
    }    
}
```

## @Autowired 注解有什么用？原理？

`@Autowired` 注解，可以更准确地控制应该在何处以及如何进行自动装配。

- 此注解用于在 setter 方法，构造函数，具有任意名称或多个参数的属性或方法上自动装配 Bean。
- 默认情况下，它是类型驱动的注入。

示例代码如下：

```java
public class EmpAccount {        
    @Autowired    
    private Employee emp;    
}
```

## @Resource与@Autowired区别？

@Resource与@Autowired的作用类似，两者的共同点：

* @Autowired 与 @Resource 注解都可以用来装配 Bean；
* 二者都可以写在字段上，也可以写在 setter 方法上。

两者的不同点有：

* @Autowired为Spring提供的注解，需要导入包`org.springframework.beans.factory.annotation.Autowired`，@Resource注解由J2EE提供，需要导入包`javax.annotation.Resource`。
* @Autowired 注解默认按照 Bean 类型(**byType**) 装配依赖的 Bean，默认的情况下，它要求依赖的对象必须存在，默认是不允许 null 值的，如果想要设置允许 null 值，可以设置它的 required 属性为 false；@Resource 注解**默认**按照  Bean 名字(**byName**)来装配依赖的 Bean。

## @Qualifier 注解有什么用？原理？

当你创建多个**相同类型**的 Bean ，并希望仅使用属性装配**其中一个** Bean 时，您可以使用 `@Qualifier` 注解和 `@Autowired` 通过指定 ID 应该装配哪个**确切的** Bean 来消除歧义。

例如，应用中有两个类型为 Employee 的 Bean ID 为 `"emp1"` 和 `"emp2"` ，此处，我们希望 EmployeeAccount Bean 注入 `"emp1"` 对应的 Bean 对象。代码如下：

```java
public class EmployeeAccount {    
    @Autowired    
    @Qualifier(emp1)    
    private Employee emp;
}
```

## 基于注解配置与基于XML配置的区别？

| 管理bean | 基于XML配置                                                   | 基于注解配置                                                                                                          |
| -------- | ------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| bean定义 | `<bean id="" class=""/>`                                      | @Component  配置一个bean<br>@Controller 表现层 <br>@Service  业务层（Service 层）<br>@Repository  数据访问层（DAO层） |
| bean注入 | `<property name="" ref=""> `<br>`<property name="" value="">` | @Autowired按照类型注入<br>@Qualifier按名称注入 <br>@Resource按名称注入<br>@Value 读取简单的配置信息                   |
| 生命周期 | `<bean id="" class="" init-method=""  destroy-method=""/>`    | @PostConstruct初始化<br>@PreDestroy销毁                                                                               |
| 作用范围 | `<bean id="" class="" scope="">`                              | @Scope设置作用范围                                                                                                    |
| 适合场景 | Bean来自第三方                                                | Bean的实现类由用户自己开发                                                                                            |

注解的优势：配置简单，维护方便（我们找到类，就相当于找到了对应的配置）。 
XML的优势：修改时，不用改源码。不涉及重新编译和部署。

# Spring Transaction

> 可参考[可能是最漂亮的 Spring 事务管理详解 (qq.com)](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247484702&idx=1&sn=c04261d63929db09ff6df7cadc7cca21&chksm=fa497aafcd3ef3b94082da7bca841b5b7b528eb2a52dbc4eb647b97be63a9a1cf38a9e71bf90&token=165108535&lang=zh_CN#rd)一文。

## 什么是事务？有何特性？

**事务**就是用户定义的一系列数据库操作，这些操作可以视为一个完成的逻辑处理工作单元（unit），**要么全部执行，要么全部不执行，是不可分割的工作单元**。

需要注意的是，**事务能否生效取决于数据库引擎是否支持事务**。常用的MySQL 数据库默认使用`innodb`引擎是支持事务的。但是，如果把数据库引擎变为 `myisam`，那么也就不再支持事务了。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20201117135453425.png" width="300px"/>
    <div>事务的关键属性(ACID)</div>
</div>

**原子性(atomicity)**：事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；

**一致性(consistent)**：执行事务前后，数据保持一致；

**隔离性(isolation)**：并发访问数据库时，一个用户的事物不被其他事物所干扰，各并发事务之间数据库是独立的；

**持久性(durability)**：一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。

## Spring 支持的事务管理类型？

目前 Spring 提供两种类型的事务管理：

- **声明式**事务：通过使用注解或基于 XML 的配置事务，从而事务管理与业务代码分离。
- **编程式**事务：通过编码的方式实现事务管理，需要在代码中显式的调用事务的获得、提交、回滚。它为您提供极大的灵活性，但维护起来非常困难。

实际场景下，我们一般使用 Spring Boot + 注解的**声明式**事务。

## Spring 中事务管理器是什么？

Spring从不同的事务管理API中抽象出了一整套事务管理机制，让事务管理代码从特定的事务技术中独立出来。开发人员通过配置的方式进行事务管理，而不必了解其底层是如何实现的。 

Spring的核心事务管理接口是**PlatformTransactionManager**。它为事务管理封装了一组独立于技术的方法。无论使用Spring的哪种事务管理策略(编程式或声明式)，事务管理器都是必须的。事务管理器可以以普通的bean的形式声明在Spring IOC容器中。

```java
// PlatformTransactionManager.java
public interface PlatformTransactionManager {
    
    // 根据事务定义 TransactionDefinition,获取 TransactionStatus 。 
    TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;

    // 根据情况，提交事务
    void commit(TransactionStatus status) throws TransactionException;
    
    // 根据情况，回滚事务
    void rollback(TransactionStatus status) throws TransactionException;
}
```

`PlatformTransactionManager` 是负责事务管理的接口，一共有三个接口方法，分别负责事务的获取、提交、回滚。

- `#getTransaction(TransactionDefinition definition)`方法，根据事务定义 TransactionDefinition ，获得 TransactionStatus 。
- `#commit(TransactionStatus status)`方法，根据 TransactionStatus 情况，提交事务。
- `#rollback(TransactionStatus status)`方法，根据 TransactionStatus 情况，回滚事务。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/2020091515255066.png" width="800px"/>
</div>

这里有两个重要的接口：TransactionDefinition与TransactionStatus

**TransactionDefinition** 接口是事务定义（描述）的对象，它提供了事务相关信息获取的方法，其中包括五个操作，具体如下。  

* `String getName()`：获取事务对象名称。  
* `int getIsolationLevel()`：获取事务的隔离级别。  
* `int getPropagationBehavior()`：获取事务的传播行为。  
* `int getTimeout()`：获取事务的超时时间。  
* `boolean isReadOnly()`：获取事务是否只读。  

**TransactionStatus** 接口是事务运行状态，不仅仅包含事务本身，还包含事务的其它信息。

| 名称                       | 说明               |
| -------------------------- | ------------------ |
| void flush()               | 刷新事务           |
| boolean hasSavepoint()     | 获取是否存在保存点 |
| boolean isCompleted()      | 获取事务是否完成   |
| boolean isNewTransaction() | 获取是否是新事务   |
| boolean isRollbackOnly()   | 获取是否回滚       |
| void setRollbackOnly()     | 设置事务回滚       |

- 为什么没有事务对象呢？在 TransactionStatus 的实现类 DefaultTransactionStatus 中，有个 `Object transaction` 属性，表示事务对象。
- `#isNewTransaction()` 方法，表示是否是新创建的事务，通过该方法，我们可以判断，当前事务是否当前方法所创建的，只有创建事务的方法，**才能且应该真正的提交事务**。

结合上面，可以把 **`PlatformTransactionManager`** 接口可以被看作是事务上层的管理者，而 **`TransactionDefinition`** 和 **`TransactionStatus`** 这两个接口可以看作是事物的描述。**`PlatformTransactionManager`** 会根据 **`TransactionDefinition`** 的定义比如事务超时时间、隔离级别、传播行为等来进行事务管理 ，而 **`TransactionStatus`** 接口则提供了一些方法来获取事务相应的状态比如是否新事务、是否可以回滚等等。

那么，通过事务管理器管理事务有什么优点：

1. 通过 PlatformTransactionManager ，为不同的数据层持久框架提供统一的 API ，无需关心到底是原生 JDBC、Spring JDBC、JPA、Hibernate 还是 MyBatis 。
2. 通过使用声明式事务，使业务代码和事务管理的逻辑分离，更加清晰。

## Spring 事务如何和不同的数据持久层框架做集成？

首先，我们先明确下，这里数据持久层框架，指的是 Spring JDBC、Hibernate、Spring JPA、MyBatis 等等。

最后，不同的数据持久层框架，会有其对应的 PlatformTransactionManager 实现类，如下图所示：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210719174327513.png" width="800px"/>
</div>

- 所有的实现类，都基于 AbstractPlatformTransactionManager 这个骨架类。
- HibernateTransactionManager ，和 Hibernate的事务管理做集成。
- DataSourceTransactionManager ，和 JDBC 的事务管理做集成。所以，它也适用于 MyBatis、Spring JDBC 等等。
- JpaTransactionManager ，和 JPA 的事务管理做集成。

如下，是一个比较常见的 XML 方式来配置的事务管理器，使用的是 DataSourceTransactionManager 。代码如下：

```java
<!-- 事务管理器 -->
<bean id="transactionManager"
class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <!-- 数据源 -->
    <property name="dataSource" ref="dataSource" />
</bean>
```

## 为什么在 Spring 事务中不能切换数据源？

做过 Spring 多数据源的胖友，都会有个惨痛的经历，为什么在开启事务的 Service 层的方法中，无法切换数据源呢？因为，在 Spring 的事务管理中，**所使用的数据库连接会和当前线程所绑定**，即使我们设置了另外一个数据源，使用的还是当前的数据源连接。

另外，多个数据源且需要事务的场景，本身会带来**多事务一致性**的问题，暂时没有特别好的解决方案。

所以一般一个应用，推荐除非了读写分离所带来的多数据源，其它情况下，建议只有一个数据源。并且，随着微服务日益身形，一个服务对应一个 DB 是比较常见的架构选择。

## @Transactional 注解有哪些属性？如何使用？

`@Transactional` 注解的**属性**如下：

| 属性                   | 类型                               | 描述                                   |
| :--------------------- | :--------------------------------- | :------------------------------------- |
| value                  | String                             | 可选的限定描述符，指定使用的事务管理器 |
| propagation            | enum: Propagation                  | 可选的事务传播行为设置                 |
| isolation              | enum: Isolation                    | 可选的事务隔离级别设置                 |
| readOnly               | boolean                            | 读写或只读事务，默认读写               |
| timeout                | int (in seconds granularity)       | 事务超时时间设置                       |
| rollbackFor            | Class对象数组，必须继承自Throwable | 导致事务回滚的异常类数组               |
| rollbackForClassName   | 类名数组，必须继承自Throwable      | 导致事务回滚的异常类名字数组           |
| noRollbackFor          | Class对象数组，必须继承自Throwable | 不会导致事务回滚的异常类数组           |
| noRollbackForClassName | 类名数组，必须继承自Throwable      | 不会导致事务回滚的异常类名字数组       |

- 一般情况下，我们直接使用 `@Transactional` 的所有属性默认值即可。

具体**用法**如下：

- `@Transactional` 可以作用于接口、接口方法、类以及类方法上。当作用于类上时，该类的所有 `public` 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。
- 虽然 `@Transactional` 注解可以作用于接口、接口方法、类以及类方法上，但是 Spring 建议不要在接口或者接口方法上使用该注解，因为这只有在使用基于接口的代理时它才会生效。另外， **`@Transactional` 注解应该只被应用到 `public` 方法上，这是由 Spring AOP 的本质决定的**。如果你在 `protected`、`private` 或者默认可见性的方法上使用 `@Transactional` 注解，这将被忽略，也不会抛出任何异常。**这一点，非常需要注意**。

------

下面，我们来简单说下**源码**相关的东西。

`@Transactional` 注解的属性，会解析成 `org.springframework.transaction.TransactionDefinition` 对象，即事务定义。TransactionDefinition 代码如下：

```java
public interface TransactionDefinition {

	int getPropagationBehavior(); // 事务的传播行为
	int getIsolationLevel(); // 事务的隔离级别
	int getTimeout(); // 事务的超时时间
	boolean isReadOnly(); // 事务是否只读
	@Nullable
	String getName(); // 事务的名字

}
```

- 可能会胖友有以后，`@Transactional` 注解的 `rollbackFor`、`rollbackForClassName`、`noRollbackFor`、`noRollbackForClassName` 属性貌似没体现出来？它们提现在 TransactionDefinition 的实现类 RuleBasedTransactionAttribute 中。
- `#getPropagationBehavior()` 方法，返回事务的**传播行为**，该值是个枚举。
- `#getIsolationLevel()` 方法，返回事务的**隔离级别**，该值是个枚举。

## @Transactional实现原理？

> 可参考[你必须懂也可以懂的@Transactional原理](https://juejin.cn/post/6968384376824561671)一文。

简而言之，**@Transactional的工作机制是基于AOP实现的，而AOP是使用动态代理实现的**。如果目标对象实现了接口使用JDK动态代理，目标类的方法是在接口中定义的，也都必须是public修饰才能被JDK动态代理；如果目标类没有实现接口，使用CGLIB动态代理，代理类是目标类的子类，理论上可以代理public和protected方法，但是Spring在进行事务增强是否能够应用到当前目标类判断的时候，遍历的是目标类的public方法，所以CGLIB方式也只对public方法有效。

这里是`createAopProxy()` 方法决定了是使用 JDK 还是 CGLIB来做动态代理，源码如下：

```java
public class DefaultAopProxyFactory implements AopProxyFactory, Serializable {

	@Override
	public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
		if (!NativeDetector.inNativeImage() &&
				(config.isOptimize() || config.isProxyTargetClass() ||
                 hasNoUserSuppliedProxyInterfaces(config))) {
			Class<?> targetClass = config.getTargetClass();
			if (targetClass == null) {
				throw new AopConfigException("TargetSource cannot determine target class: " +
						"Either an interface or a target is required for proxy creation.");
			}
			if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
				return new JdkDynamicAopProxy(config);
			}
            // 最终调用到CglibAopProxy(AdvisedSupport config) 
			return new ObjenesisCglibAopProxy(config);
		}
		else {
			return new JdkDynamicAopProxy(config);
		}
	}
  .......
}
```

在创建代理的过程中会获取当前目标方法对应的拦截器，此时会得到`TransactionInterceptor`实例，在它的`invoke`方法中实现事务的开启和回滚，在需要进行事务操作的时候，Spring会在调用目标类的目标方法之前进行开启事务、调用异常回滚事务、调用完成会提交事务。是否需要开启新事务，是根据`@Transactional`注解上配置的参数值来判断的。

如果需要开启新事务，获取`Connection`连接，然后将连接的自动提交事务改为false，改为手动提交。当对目标类的目标方法进行调用的时候，若发生异常将会进入`completeTransactionAfterThrowing`方法。

Spring并不会对所有类型异常都进行事务回滚操作，默认是只对Unchecked Exception(Error和RuntimeException)进行事务回滚操作。

## 什么是事务的隔离级别？

一个事务与其他事务隔离的程度称为隔离级别。在谈隔离级别之前，首先要知道，**隔离级别越高，数据一致性就越好，但效率越弱**。因此很多时候，我们都要在二者之间寻找一个平衡点。SQL标准的事务隔离级别包括：**读未提交（READ_UNCOMMITTED）、读提交（READ_COMMITTED）、可重复读（REPEATABLE_READ）和串行化（SERIALIZABLE）**。

在 TransactionDefinition 接口中，也定义了“**四种**”的隔离级别枚举：

| 隔离级别         | 含义                                                                                                               | 举例                                                                                                                                     |
| ---------------- | ------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------- |
| READ_UNCOMMITTED | 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读。                                     | 允许A读取B未提交的修改。                                                                                                                 |
| READ_COMMITTED   | 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生。                                   | 要求A只能读取B已提交的修改。                                                                                                             |
| REPEATABLE_READ  | 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。 | 确保A可以多次从一个字段中读取到相同的值，即A执行期间禁止其它事务对这个字段进行更新。                                                     |
| SERIALIZABLE     | 最高的隔离级别，完全服从ACID的隔离级别，确保阻止脏读、不可重复读以及幻读。                                         | 确保A可以多次从一个表中读取到相同的行，在A执行期间，禁止其它事务对这个表进行添加、更新、删除操作。可以避免任何并发问题，但性能十分低下。 |

各个隔离级别所能解决的并发问题（x——不能解决，√——能解决）：

| 隔离级别     | 脏读 | 不可重复读 | 幻读        |
| ------------ | ---- | ---------- | ----------- |
| RU           | ×    | ×          | ×           |
| RC           | √    | ×          | ×           |
| RR           | √    | √          | ×（可解决） |
| SERIALIZABLE | √    | √          | √           |

## 什么是事务的传播级别？

事务的**传播行为**，指的是当前带有事务配置的方法，需要怎么处理事务。

- 例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。

- 有一点需要注意，事务的传播级别，并不是数据库事务规范中的名词，**而是 Spring 自身所定义的**。通过事务的传播级别，Spring 才知道如何处理事务，是创建一个新事务呢，还是继续使用当前的事务。


在 TransactionDefinition 接口中，定义了**三类七种**传播级别。代码如下：

| 传播行为                      | 含义                                                                                                                                                                                         |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| PROPAGATION_**REQUIRED**      | **默认**的Spring事务传播级别，使用该级别的特点是，如果上下文中已经存在事务，那么就加入到事务中执行，如果当前上下文中不存在事务，则新建事务执行。所以这个级别通常能满足处理大多数的业务场景。 |
| PROPAGATION_**SUPPORTS**      | 当前方法不需要事务上下文就可执行，但是如果存在当前事务的话，那么该方法会在这个事务中运行。应用场景较少。                                                                                     |
| PROPAGATION_**MANDATORY**     | 要求**上下文中必须要存在事务，否则就会抛出异常**！配置该方式的传播级别是有效的控制上下文调用代码遗漏添加事务控制的保证手段。                                                                 |
| PROPAGATION_**REQUIRED_NEW**  | 当前方法必须运行在它自己的事务中。一个新的事务将被启动。如果上下文存在事务，在该方法执行期间，其他事务会被挂起。                                                                             |
| PROPAGATION_**NOT_SUPPORTED** | 该方法不应该运行在事务中。如果存在当前事务，在该方法运行期间，当前事务将被挂起。                                                                                                             |
| PROPAGATION_**NEVER**         | 该事务更严格，上面一个事务传播级别只是不支持而已，有事务就挂起，而PROPAGATION_NEVER传播级别要求上下文中不能存在事务，一旦有事务，就抛出Runtime异常，强制停止执行！                           |
| PROPAGATION_**NESTED**        | 嵌套级别事务。该传播级别特征是，如果上下文中存在事务，则在该事务中嵌套新事务执行；如果不存在事务，则与PROPAGATION_REQUIRED一样，新建事务。                                                   |

分类之后，其实还是比较好记的。当然，绝大数场景，我们只用 `PROPAGATION_REQUIRED` 传播级别。

这里需要指出的是，前面的六种事务传播行为是 Spring 从 EJB 中引入的，他们共享相同的概念。而`PROPAGATION_NESTED`是 Spring 所特有的。那么如何理解嵌套事务呢？

- 以 `PROPAGATION_NESTED` 启动的事务内嵌于外部事务中（如果存在外部事务的话），内嵌事务并不是一个独立的事务，它依赖于外部事务的存在，只有通过外部的事务提交，才能引起内部事务的提交，嵌套的子事务不能单独提交。如果熟悉 JDBC 中的保存点（SavePoint）的概念，那嵌套事务就很容易理解了，其实嵌套的子事务就是保存点的一个应用，一个事务中可以包括多个保存点，每一个嵌套子事务。另外，外部事务的回滚也会导致嵌套子事务的回滚。

## 什么是事务的超时属性与只读属性？

**超时属性**

* 事务超时属性，就是指一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。在 TransactionDefinition 中以 `int` 的值来表示超时时间，其单位是秒。当然，这个属性，貌似我们基本也没用过。

**只读属性**

事务的只读属性是指，对事务性资源进行只读操作或者是读写操作。在 TransactionDefinition 中以 `boolean` 类型来表示该事务是否只读。

- 所谓事务性资源就是指那些被事务管理的资源，比如数据源、JMS 资源，以及自定义的事务性资源等等。
- 如果确定只对事务性资源进行**只读**操作，那么我们可以将事务标志为只读的，以提高事务处理的性能。

很多人就会疑问了，为什么一个数据查询操作还要启用事务支持呢？

* 如果你一次执行单条查询语句，则没有必要启用事务支持，数据库默认支持 SQL 执行期间的读一致性；
* 如果你一次执行多条查询语句，例如统计查询，报表查询，在这种场景下，多条查询 SQL 必须保证整体的读一致性，否则，在前条 SQL 查询之后，后条 SQL 查询之前，数据被其他用户改变，则该次整体的统计查询将会出现读数据不一致的状态，此时，应该启用事务支持。

## 什么是事务的回滚规则？

回滚规则，定义了哪些异常会导致事务回滚而哪些不会。

- 默认情况下，事务只有遇到运行期异常时才会回滚，而在遇到检查型异常时不会回滚（这一行为与EJB的回滚行为是一致的）。
- 但是你可以声明事务在遇到特定的检查型异常时像遇到运行期异常那样回滚。同样，你还可以声明事务遇到特定的异常不回滚，即使这些异常是运行期异常。

注意，事务的回滚规则，并不是数据库事务规范中的名词，**而是 Spring 自身所定义的**。

## @Transactional/事务失效场景有哪些？

> 可参考[一口气说出 6种，@Transactional注解的失效场景](https://juejin.cn/post/6844904096747503629)一文。

**1. @Transactional 应用在`protected`、`private`修饰的方法上**

之所以会失效是因为在Spring AOP 代理时， `TransactionInterceptor` （事务拦截器）在目标方法执行前后进行拦截，`DynamicAdvisedInterceptor`（CglibAopProxy 的内部类）的 intercept 方法或 `JdkDynamicAopProxy` 的 invoke 方法会间接调用 `AbstractFallbackTransactionAttributeSource`的 `computeTransactionAttribute` 方法，获取Transactional 注解的事务配置信息。

```java
protected TransactionAttribute computeTransactionAttribute(Method method,
    Class<?> targetClass) {
        // Don't allow no-public methods as required.
        if (allowPublicMethodsOnly() && !Modifier.isPublic(method.getModifiers())) {
        return null;
}
```

此方法会检查目标方法的修饰符是否为 public，不是 public则不会获取@Transactional 的属性配置信息。

**2. @Transactional 注解属性 propagation 设置错误**

配置错误，若是错误的配置以下三种 propagation，事务将不会发生回滚。

`TransactionDefinition.PROPAGATION_SUPPORTS`：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。 `TransactionDefinition.PROPAGATION_NOT_SUPPORTED`：以非事务方式运行，如果当前存在事务，则把当前事务挂起。 `TransactionDefinition.PROPAGATION_NEVER`：以非事务方式运行，如果当前存在事务，则抛出异常。

**3. @Transactional  注解属性 rollbackFor 设置错误**

`rollbackFor` 可以指定能够触发事务回滚的异常类型。**Spring默认抛出了未检查`unchecked`异常（继承自 `RuntimeException` 的异常）或者 `Error`才回滚事务**；其他异常不会触发回滚事务。如果在事务中抛出其他类型的异常，但却期望 Spring 能够回滚事务，就需要指定 **rollbackFor**属性。

```java
// 希望自定义的异常可以进行回滚
@Transactional(propagation= Propagation.REQUIRED,rollbackFor= MyException.class
```

若在目标方法中抛出的异常是 `rollbackFor` 指定的异常的子类，事务同样会回滚。Spring源码如下：

```java
private int getDepth(Class<?> exceptionClass, int depth) {
        if (exceptionClass.getName().contains(this.exceptionName)) {
            // Found it!
            return depth;
}
        // If we've gone as far as we can go and haven't found it...
        if (exceptionClass == Throwable.class) {
            return -1;
}
return getDepth(exceptionClass.getSuperclass(), depth + 1);
}
```

**4. 同一个类中方法调用，导致@Transactional失效**

开发中避免不了会对同一个类里面的方法调用，比如有一个类Test，它的一个方法A，A再调用本类的方法B（不论方法B是用public还是private修饰），但方法A没有声明注解事务，而B方法有。则外部调用方法A之后，方法B的事务是不会起作用的。这也是经常犯错误的一个地方。

那为啥会出现这种情况？其实这还是由于使用`Spring AOP`代理造成的，因为只有当事务方法被当前类以外的代码调用时，才会由`Spring`生成的代理对象来管理。

**5. 异常被你的 catch“吃了”导致@Transactional失效**

这种情况是最常见的一种@Transactional注解失效场景，

```java
    @Transactional
    private Integer A() throws Exception {
        int insert = 0;
        try {
            CityInfoDict cityInfoDict = new CityInfoDict();
            cityInfoDict.setCityName("2");
            cityInfoDict.setParentCityId(2);
            /**
             * A 插入字段为 2的数据
             */
            insert = cityInfoDictMapper.insert(cityInfoDict);
            /**
             * B 插入字段为 3的数据
             */
            b.insertB();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

如果B方法内部抛了异常，而A方法此时try catch了B方法的异常，那这个事务还能正常回滚吗？

答案：不能！

会抛出异常：

```java
org.springframework.transaction.UnexpectedRollbackException: Transaction rolled back because it has been marked as rollback-only
```

因为当`ServiceB`中抛出了一个异常以后，`ServiceB`标识当前事务需要`rollback`。但是`ServiceA`中由于你手动的捕获这个异常并进行处理，`ServiceA`认为当前事务应该正常`commit`。此时就出现了前后不一致，也就是因为这样，抛出了前面的`UnexpectedRollbackException`异常。

`spring`的事务是在调用业务方法之前开始的，业务方法执行完毕之后才执行`commit` or `rollback`，事务是否执行取决于是否抛出`runtime异常`。如果抛出`runtime exception` 并在你的业务方法中没有catch到的话，事务会回滚。

在业务方法中一般不需要catch异常，如果非要catch一定要抛出`throw new RuntimeException()`，或者注解中指定抛异常类型`@Transactional(rollbackFor=Exception.class)`，否则会导致事务失效，数据commit造成数据不一致，所以有些时候try catch反倒会画蛇添足。

**6. 数据库引擎不支持事务**

这种情况出现的概率并不高，事务能否生效数据库引擎是否支持事务是关键。常用的MySQL数据库默认使用支持事务的`innodb`引擎。一旦数据库引擎切换成不支持事务的`myisam`，那事务就从根本上失效了。

# Spring Data Access

## Spring 支持哪些 ORM 框架？

Hibernate、JPA、MyBatis（JDBC不是ORM框架）

## 在 Spring 框架中如何更有效地使用 JDBC ？

Spring 提供了 Spring JDBC 框架，方便我们使用 JDBC 。

对于开发者，只需要使用 JdbcTemplate 类，它提供了很多便利的方法解决诸如把数据库数据转变成基本数据类型或对象，执行写好的或可调用的数据库操作语句，提供自定义的数据错误处理。

## Spring 数据数据访问层有哪些异常？

通过使用 Spring 数据数据访问层，它统一了各个数据持久层框架的不同异常，统一进行提供 `org.springframework.dao.DataAccessException` 异常及其子类。如下图所示：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210719181250646.png" width="600px"/>
</div>

## 使用 Spring 访问 Hibernate 的方法有哪些？

我们可以通过两种方式使用 Spring 访问 Hibernate：

- 使用 Hibernate 模板和回调进行控制反转。
- 扩展 HibernateDAOSupport 并应用 AOP 拦截器节点。
