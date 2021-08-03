<!-- MarkdownTOC -->

- [基础篇](#基础篇)
  - [Spring Boot 是什么？](#spring-boot-是什么)
  - [Spring Boot 提供了哪些核心功能？](#spring-boot-提供了哪些核心功能)
  - [Spring Boot 有什么优缺点？](#spring-boot-有什么优缺点)
  - [Spring Boot、Spring MVC 和 Spring 有什么区别？](#spring-bootspring-mvc-和-spring-有什么区别)
  - [Spring Boot 中的 Starter 是什么？](#spring-boot-中的-starter-是什么)
  - [pring-boot-starter-parent 有什么用?](#pring-boot-starter-parent-有什么用)
  - [创建一个 Spring Boot Project 的最简单的方法是什么？](#创建一个-spring-boot-project-的最简单的方法是什么)
  - [如何统一引入 Spring Boot 版本？](#如何统一引入-spring-boot-版本)
  - [运行 Spring Boot 有哪几种方式？](#运行-spring-boot-有哪几种方式)
  - [如何打包 Spring Boot 项目？](#如何打包-spring-boot-项目)
  - [如果更改内嵌 Tomcat 的端口？](#如果更改内嵌-tomcat-的端口)
  - [如何重新加载 Spring Boot 上的更改，而无需重新启动服务器？](#如何重新加载-spring-boot-上的更改而无需重新启动服务器)
  - [Spring Boot 2.X 有什么新特性？](#spring-boot-2x-有什么新特性)
  - [如何在 Spring Boot 启动的时候运行一些特殊的代码？](#如何在-spring-boot-启动的时候运行一些特殊的代码)
- [配置篇](#配置篇)
  - [Spring Boot 的配置文件有哪几种格式？](#spring-boot-的配置文件有哪几种格式)
  - [Spring Boot 默认配置文件是什么？](#spring-boot-默认配置文件是什么)
  - [Spring Boot 如何定义多套不同环境配置？](#spring-boot-如何定义多套不同环境配置)
  - [Spring Boot 配置文件不同加载位置？](#spring-boot-配置文件不同加载位置)
  - [Spring Boot 配置加载顺序？](#spring-boot-配置加载顺序)
  - [Spring Boot 有哪些配置方式？](#spring-boot-有哪些配置方式)
  - [什么是 JavaConfig？](#什么是-javaconfig)
  - [Spring Boot 读取配置的方式？](#spring-boot-读取配置的方式)
  - [Spring Boot 的核心注解？](#spring-boot-的核心注解)
  - [Spring Boot 自动配置原理？](#spring-boot-自动配置原理)
  - [@Conditional 扩展注解有哪些？](#conditional-扩展注解有哪些)
- [安全篇](#安全篇)
  - [如何集成 Spring Boot 和 Spring Security ？](#如何集成-spring-boot-和-spring-security-)
  - [如何集成 Spring Boot 和 Spring Security OAuth2 ？](#如何集成-spring-boot-和-spring-security-oauth2-)
  - [比较一下Spring Security 和Shiro各自的优缺点 ?](#比较一下spring-security-和shiro各自的优缺点-)
  - [Spring Boot 中如何解决跨域问题 ?](#spring-boot-中如何解决跨域问题-)
  - [什么是 CSRF 攻击？](#什么是-csrf-攻击)
  - [Spring Boot 中的监视器是什么](#spring-boot-中的监视器是什么)
  - [如何在 Spring Boot 中禁用Actuator端点安全性？](#如何在-spring-boot-中禁用actuator端点安全性)
  - [如何监视所有 Spring Boot 微服务？](#如何监视所有-spring-boot-微服务)
- [进阶篇](#进阶篇)
  - [Swagger 用过吗？它用来做什么？](#swagger-用过吗它用来做什么)
  - [Spring Boot 项目如何热部署？](#spring-boot-项目如何热部署)
  - [微服务中如何实现 session 共享?](#微服务中如何实现-session-共享)
  - [Spring Boot 中如何实现定时任务?](#spring-boot-中如何实现定时任务)
  - [如何将内嵌服务器换成 Jetty ？](#如何将内嵌服务器换成-jetty-)
  - [如何集成 Spring Boot 和 Spring MVC ？](#如何集成-spring-boot-和-spring-mvc-)
  - [Spring Boot 支持哪些日志框架？](#spring-boot-支持哪些日志框架)
- [整合篇](#整合篇)

<!-- /MarkdownTOC -->

# 基础篇

## Spring Boot 是什么？

[Spring Boot](https://github.com/spring-projects/spring-boot) 是 Spring 的**子项目**，正如其名字，提供 Spring 的引导( **Boot** )的功能。

通过 Spring Boot ，我们开发者可以快速配置 Spring 项目，引入各种 Spring MVC、Spring Transaction、Spring AOP、MyBatis 等等框架，而无需不断重复编写繁重的 Spring 配置，降低了 Spring 的使用成本。

Spring Boot 提供了各种 Starter 启动器，提供标准化的默认配置。例如：

- [`spring-boot-starter-web`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web/2.1.1.RELEASE) 启动器，可以快速配置 Spring MVC 。
- [`mybatis-spring-boot-starter`](https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter/1.3.2) 启动器，可以快速配置 MyBatis 。

并且，Spring Boot 基本已经一统 Java 项目的开发，大量的开源项目都实现了其的 Starter 启动器。例如：

- [`incubator-dubbo-spring-boot-project`](https://github.com/apache/incubator-dubbo-spring-boot-project) 启动器，可以快速配置 Dubbo 。
- [`rocketmq-spring-boot-starter`](https://github.com/maihaoche/rocketmq-spring-boot-starter) 启动器，可以快速配置 RocketMQ 。

## Spring Boot 提供了哪些核心功能？

**1、独立运行 Spring 项目**

Spring Boot 可以以 jar 包形式独立运行，运行一个 Spring Boot 项目只需要通过 `java -jar xx.jar` 来运行。

**2、内嵌 Servlet 容器**

Spring Boot 可以选择内嵌 Tomcat、Jetty 或者 Undertow，这样我们无须以 war 包形式部署项目。

> 第 2 点是对第 1 点的补充，在 Spring Boot 未出来的时候，大多数 Web 项目，是打包成 war 包，部署到 Tomcat、Jetty 等容器。

**3、提供 Starter 简化 Maven 配置**

Spring 提供了一系列的 starter pom 来简化 Maven 的依赖加载。例如，当你使用了 `spring-boot-starter-web` ，会自动加入如下依赖：

<div align="center">  
<img src="http://static.iocoder.cn/images/Spring/2018-12-26/01.png" width="600px"/>
</div>

**4、自动配置 Spring Bean**

Spring Boot 检测到特定类的存在，就会针对这个应用做一定的配置，进行自动配置 Bean ，这样会极大地减少我们要使用的配置。

当然，Spring Boot 只考虑大多数的开发场景，并不是所有的场景，若在实际开发中我们需要配置Bean ，而 Spring Boot 没有提供支持，则可以自定义自动配置进行解决。

**5、准生产的应用监控**

Spring Boot 提供基于 HTTP、JMX、SSH 对运行时的项目进行监控。

**6、无代码生成和 XML 配置**

Spring Boot 没有引入任何形式的代码生成，它是使用的 Spring 4.0 的条件 `@Condition` 注解以实现根据条件进行配置。同时使用了 Maven /Gradle 的**依赖传递解析机制**来实现 Spring 应用里面的自动配置。

## Spring Boot 有什么优缺点？

**Spring Boot 的优点**

> 优点和 「Spring Boot 提供了哪些核心功能？」 问题的答案，是比较重叠的。

- 快速构建项目。
- 对主流开发框架的无配置集成。
- 项目可独立运行，无须外部依赖Servlet容器。
- 提供运行时的应用监控。
- 极大地提高了开发、部署效率。
- 与云计算的天然集成。

**Spring Boot 的缺点**

- 版本迭代速度很快，一些模块改动很大。

- 由于不用自己做配置，报错时很难定位。
- 网上现成的解决方案比较少。

## Spring Boot、Spring MVC 和 Spring 有什么区别？

Spring的完整名字，应该是 Spring Framework ，它是一个一站式的轻量级的Java开发框架，核心是控制反转（IoC）和面向切面编程（AOP），针对于开发的WEB层(Spring MVC)、业务层(IoC/AOP)、持久层(JDBC Template)等都提供了多种配置解决方案；

Spring MVC是 Spring Framework 众多模块中的一个，主要用来处理WEB开发的路径映射和视图渲染；

Spring Boot 是构造在 Spring Framework 之上的 Boot 启动器，它遵循“约定大于配置”的原则，极大地简化了开发配置流程，集成了快速开发Spring的多个插件，简化了项目的开发配置流程，能够做到“自动配置，开箱即用”。

所以，用最简练的语言概括就是:

* Spring 是一个“引擎”;
* Spring MVC 是基于Spring的一个 MVC 框架;
* Spring Boot 是基于Spring4的条件注册的一套快速开发整合包。

## Spring Boot 中的 Starter 是什么？

谈到Spring Boot的特点， 每个人都能说出“**开箱即用、约定大于配置**”等词句，而这些都与Spring Boot Starter有关。

SpringBoot 提供的依赖模块都约定以 `spring-boot-starter-` 作为命名的前缀，并且皆位于 `org.springframework.boot` 包或者命名空间下。当我们需要使用的时候，只要将 `spring-boot-starter-`作为依赖加入到当前应用的 classpath，则“开箱即用”，不需要做任何多余的配置。

所有的 `spring-boot-starte`r 都有约定俗成的默认配置，但允许我们调整这些配置以改变默认的配置行为，即“约定优先于配置”。

**Spring Boot 常用的 Starter 有：**

- `spring-boot-starter-web` ：提供 Spring MVC + 内嵌的 Tomcat 。
- `spring-boot-starter-data-jpa` ：提供 Spring JPA + Hibernate 。
- `spring-boot-starter-data-redis` ：提供 Redis 。
- `mybatis-spring-boot-starter` ：提供 MyBatis 。

## pring-boot-starter-parent 有什么用?

新创建一个 SpringBoot 项目，默认都是有 parent 的，这个 parent 就是 s`pring-boot-starter-parent` ，spring-boot-starter-parent 主要有如下作用：

1. 定义了 Java 编译版本为 1.8 。
2. 使用 UTF-8 格式编码。
3. 继承自 `spring-boot-dependencies`，这个里边定义了依赖的版本，也正是因为继承了这个依赖，所以我们在写依赖时才不需要写版本号。
4. 执行打包操作的配置。
5. 自动化的资源过滤。
6. 自动化的插件配置。
7. 针对 `application.properties` 和 `application.yml` 的资源过滤，包括通过 profile 定义的不同环境的配置文件，例如 `application-dev.properties` 和 `application-dev.yml`。

## 创建一个 Spring Boot Project 的最简单的方法是什么？

Spring Initializr 是创建 Spring Boot Projects 的一个很好的工具。打开 `"https://start.spring.io/"` 网站，我们可以看到 Spring Initializr 工具，如下图所示：

<div align="center">  
<img src="http://static.iocoder.cn/images/Spring/2018-12-26/04.png" width="600px"/>
</div>

- 图中的每一个**红线**，都可以填写相应的配置。
- 点击生 GenerateProject ，生成 Spring Boot Project 。
- 将项目导入 IDEA ，记得选择现有的 Maven 项目。

------

当然，我们以前使用 IDEA 创建 Spring 项目的方式，也一样能创建 Spring Boot Project 。Spring Initializr 更多的是，提供一个便捷的工具。

## 如何统一引入 Spring Boot 版本？

**目前有两种方式**。

① 方式一：继承 `spring-boot-starter-parent` 项目。配置代码如下：

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.1.RELEASE</version>
</parent>
```

② 方式二：导入 `spring-boot-dependencies` 项目依赖。配置代码如下：

```java
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>1.5.1.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

**如何选择？**

因为一般我们的项目中，都有项目自己的 Maven parent 项目，所以【方式一】显然会存在冲突。所以实际场景下，推荐使用【方式二】。

## 运行 Spring Boot 有哪几种方式？

- 1、打包成 Fat Jar ，直接使用 `java -jar` 运行。目前主流的做法，推荐。
- 2、在 IDEA 或 Eclipse 中，直接运行应用的 Spring Boot 启动类的 `#main(String[] args)` 启动。适用于开发调试场景。
- 3、如果是 Web 项目，可以打包成 War 包，使用外部 Tomcat 或 Jetty 等容器。

## 如何打包 Spring Boot 项目？

通过引入 `spring-boot-maven-plugin` 插件，执行 `mvn clean package` 命令，将 Spring Boot 项目打成一个 Fat Jar 。后续，我们就可以直接使用 `java -jar` 运行。

## 如果更改内嵌 Tomcat 的端口？

- 方式一，修改 `application.properties` 配置文件的 `server.port` 属性。

  ```
  server.port=9090
  ```

- 方式二，通过启动命令增加 `server.port` 参数进行修改。

  ```
  java -jar xxx.jar --server.port=9090
  ```

当然，以上的方式，不仅仅适用于 Tomcat ，也适用于 Jetty、Undertow 等服务器。

## 如何重新加载 Spring Boot 上的更改，而无需重新启动服务器？

一共有三种方式，可以实现效果：

- 【推荐】`spring-boot-devtools` 插件。注意，这个工具需要配置 IDEA 的自动编译。

- Spring Loaded 插件。

  > Spring Boot 2.X 后，官方宣布不再支持 Spring Loaded 插件 的更新，所以基本可以无视它了。

- [JRebel](https://www.jianshu.com/p/bab43eaa4e14) 插件，需要付费。

关于如何使用 `spring-boot-devtools` 和 Spring Loaded 插件，胖友可以看看 [《Spring Boot 学习笔记：Spring Boot Developer Tools 与热部署》](https://segmentfault.com/a/1190000014488100) 。

## Spring Boot 2.X 有什么新特性？

1. 起步 JDK 8 和支持 JDK 9
2. 第三方库的升级
3. Reactive Spring
4. HTTP/2 支持
5. 配置属性的绑定
6. Gradle 插件
7. Actuator 改进
8. 数据支持的改进
9. Web 的改进
10. 支持 Quartz 自动配置
11. 测试的改进
12. ... …

## 如何在 Spring Boot 启动的时候运行一些特殊的代码？

如果需要在 SpringApplication 启动后执行一些特殊的代码，你可以实现 ApplicationRunner 或 CommandLineRunner 接口，这两个接口工作方式相同，都只提供单一的 run 方法，该方法仅在 `SpringApplication#run(...)` 方法**完成之前调用**。

# 配置篇

## Spring Boot 的配置文件有哪几种格式？

Spring Boot 目前支持两种格式的配置文件：

- `.properties` 格式。示例如下：

  ```java
  server.port = 9090
  ```

- `.yaml` 格式。示例如下：

  ```java
  server:
      port: 9090
  ```

------

可能有胖友不了解 **YAML 格式**？

YAML 是一种人类可读的数据序列化语言，它通常用于配置文件。

- 与 Properties 文件相比，如果我们想要在配置文件中添加复杂的属性 YAML 文件就更加**结构化**。从上面的示例，我们可以看出 YAML 具有**分层**配置数据。
- 当然 YAML 在 Spring 会存在一个缺陷，`@PropertySource`注解不支持读取 YAML 配置文件，仅支持 Properties 配置文件。不过这个问题也不大，可以麻烦一点使用 `@Value`注解，来读取 YAML 配置项。

## Spring Boot 默认配置文件是什么？

对于 Spring Boot 应用，默认的配置文件根目录下的 **application** 配置文件，当然可以是 Properties 格式，也可以是 YAML 格式。

可能有胖友说，我在网上看到面试题中，说还有一个根目录下的 **bootstrap** 配置文件。这个是 Spring Cloud 新增的启动配置文件，[需要引入 `spring-cloud-context` 依赖后，才会进行加载](https://my.oschina.net/freeskyjs/blog/1843048)。它的特点和用途主要是：

- 【特点】因为 bootstrap 由父 ApplicationContext 加载，比 application 优先加载。
- 【特点】因为 bootstrap 优先于 application 加载，所以不会被它覆盖。
- 【用途】使用配置中心 Spring Cloud Config 时，需要在 bootstrap 中配置配置中心的地址，从而实现父 ApplicationContext 加载时，从配置中心拉取相应的配置到应用中。

另外，[《Appendix A. Common application properties》](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html) 中，有 application 配置文件的通用属性列表。

## Spring Boot 如何定义多套不同环境配置？

生产环境的配置文件的安全性，显然我们不能且不应该把生产的配置放到项目的 Git 仓库中进行管理。那么应该怎么办呢？

- 方案一，生产环境的配置文件放在生产环境的服务器中，以 `java -jar myproject.jar --spring.config.location=/xxx/yyy/application-prod.properties` 命令，设置参数 `spring.config.location` 指向配置文件（`application-dev.properties`、`application-prod.properties`）。
- 方案二，使用 Jenkins 在执行打包，配置上 Maven Profile 功能，使用服务器上的配置文件。 整体来说，和【方案一】的差异是，将配置文件打包进了 Jar 包中。
- 方案三，使用配置中心。

> **什么是Spring Profiles？**
>
> * 主要用来**区分环境**； Spring Profiles 允许用户根据配置文件（dev，test，prod 等）来注册 bean。因此，当应用程序在开发中运行时，只有某些 bean 可以加载，而在 PRODUCTION中，某些其他 bean 可以加载。假设我们的要求是 Swagger 文档仅适用于 QA 环境，并且禁用所有其他文档。这可以使用配置文件来完成。Spring Boot 使得使用配置文件非常简单。

## Spring Boot 配置文件不同加载位置？

Spring Boot 启动会扫描以下位置的`application.properties`或者`application.yml`文件作为Spring boot的默认配置文件。加载的优先度为：

1. `– file:./config/`

2. `– file:./`

3. `– classpath:/config/`

4. `– classpath:/`

优先级由高到底，**高优先级配置内容**会覆盖**低优先级配置**的重复内容。

Spring Boot会从这四个位置**全部加载**主配置文件——**互补配置**。

![](https://img-blog.csdnimg.cn/20200923111052310.png)



==我们还可以通过spring.config.location来改变默认的配置文件位置。==项目打包好以后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置，指定配置文件和默认加载的这些配置文件共同起作用形成互补配置。

```shell
java -jar spring-boot-config-0.0.1-SNAPSHOT.jar --spring.config.location=C:/application.properties
```

## Spring Boot 配置加载顺序？

> 官方文档：[Externalized Configuration](https://docs.spring.io/spring-boot/docs/2.5.0-SNAPSHOT/reference/html/features.html#features.external-config)

在 Spring Boot 中，除了我们常用的 application 配置文件之外，还有：系统环境变量，命令行参数... ...

Spring Boot也可以从以下位置加载配置，**优先级从高到低，高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置**。

1. `spring-boot-devtools`依赖的`spring-boot-devtools.properties`配置文件。
2. 单元测试上的`@TestPropertySource`和`@SpringBootTest`注解指定的参数。（前者优先度高于后者）
3. **命令行参数**：所有的配置都可以在命令行上进行指定，多个配置用空格分开：`--配置项=值`

```shell
java -jar spring-boot-config-0.0.1-SNAPSHOT.jar --server.port=8087  --server.context-path=/abc
```

4. 命令行中的 `spring.application.json` 指定参数。例如 `java -Dspring.application.json='{"name":"Java"}' -jar springboot.jar` 。
5. ServletConfig 初始化参数。
6. ServletContext 初始化参数。
7. JNDI 参数。例如 `java:comp/env` 。
8. Java 系统变量，即 `System#getProperties()` 方法对应的。
9. 操作系统环境变量。
10. RandomValuePropertySource配置的`random.*`属性值
11. Jar **外部**的带指定 profile 的 application 配置文件。例如 `application-{profile}.yaml` 。
12. Jar **内部**的带指定 profile 的 application 配置文件。例如 `application-{profile}.yaml` 。
13. Jar **外部** application 配置文件。例如 `application.yaml` 。
14. Jar **内部** application 配置文件。例如 `application.yaml` 。
15. 在自定义的 `@Configuration` 类中定于的 `@PropertySource` 。
16. 启动的 main 方法中，定义的默认配置。即通过 `SpringApplication#setDefaultProperties(Map<String, Object> defaultProperties)` 方法进行设置。

## Spring Boot 有哪些配置方式？

和 Spring 一样，一共提供了三种方式。

1、XML 配置文件。

Bean 所需的依赖项和服务在 XML 格式的配置文件中指定。这些配置文件通常包含许多 bean 定义和特定于应用程序的配置选项。它们通常以 bean 标签开头。例如：

```java
<bean id="studentBean" class="org.edureka.firstSpring.StudentBean">
    <property name="name" value="Edureka"></property>
</bean>
```

2、注解配置。

您可以通过在相关的类，方法或字段声明上使用注解，将 Bean 配置为组件类本身，而不是使用 XML 来描述 Bean 装配。默认情况下，Spring 容器中未打开注解装配。因此，您需要在使用它之前在 Spring 配置文件中启用它。例如：

```java
<beans>
<context:annotation-config/>
<!-- bean definitions go here -->
</beans>
```

3、**Java Config** 配置。

Spring 的 Java 配置是通过使用 @Bean 和 @Configuration 来实现。

- `@Bean` 注解扮演与 `<bean />` 元素相同的角色。

- `@Configuration` 类允许通过简单地调用同一个类中的其他 `@Bean` 方法来定义 Bean 间依赖关系。

- 例如：

  ```java
  @Configuration
  public class StudentConfig {
      
      @Bean
      public StudentBean myStudent() {
          return new StudentBean();
      }
      
  }
  ```


目前主要使用 **Java Config** 配置为主。当然，三种配置方式是可以混合使用的。例如说：

- Dubbo 服务的配置，喜欢使用 XML 。
- Spring MVC 请求的配置，喜欢使用 `@RequestMapping` 注解。
- Spring MVC 拦截器的配置，喜欢 Java Config 配置。

## 什么是 JavaConfig？

Spring JavaConfig 是 Spring 社区的产品，它提供了配置 Spring IoC 容器的纯Java 方法。因此它有助于避免使用 XML 配置。使用 JavaConfig 的优点在于：

- **面向对象的配置**。 由于配置被定义为 JavaConfig 中的类，因此用户可以充分利用 Java 中的面向对象功能。一个配置类可以继承另一个，重写它的@Bean 方法等。
- **减少或消除 XML 配置**。 基于依赖注入原则的外化配置的好处已被证明。但是，许多开发人员不希望在 XML 和 Java 之间来回切换。JavaConfig 为开发人员提供了一种纯 Java 方法来配置与 XML 配置概念相似的 Spring 容器。从技术角度来讲，只使用 JavaConfig 配置类来配置容器是可行的，但实际上很多人认为将JavaConfig 与 XML 混合匹配是理想的。
- **类型安全和重构友好**。 JavaConfig 提供了一种类型安全的方法来配置 Spring容器。由于 Java 5.0 对泛型的支持，现在可以按类型而不是按名称检索 bean，不需要任何强制转换或基于字符串的查找。

## Spring Boot 读取配置的方式？

Spring Boot 目前支持 **2** 种读取配置：

* `@Value` 注解，读取配置到属性，最常用。

* `@ConfigurationProperties` 注解，读取配置到类上。

两者都可以和 `@PropertySource` 注解一起使用，指定使用的配置文件。

| 比较                 | @ConfigurationProperties | @Value     |
| -------------------- | ------------------------ | ---------- |
| 功能                 | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定（松散语法） | 支持                     | 不支持     |
| SpEL                 | 不支持                   | 支持       |
| JSR303数据校验       | 支持                     | 不支持     |
| 复杂类型封装         | 支持                     | 不支持     |

> 松散绑定（Realxed binding）是指person.firstName/first-name/first_name不同写法都能识别。

不论配置文件yml还是properties他们都能获取到值。在某个业务逻辑中需要获取配置文件中的某项值——使用@Value；专门编写了一个JavaBean来和配置文件进行映射——使用@ConfigurationProperties。

## Spring Boot 的核心注解？

```java
/**
 *  @Spring BootApplication 标注主程序类，说明这是一个Spring Boot应用
 */
@SpringBootApplication
public class HelloWorldApplication {
    public static void main(String[] args) {

        // 启动Spring应用
        SpringApplication.run(HelloWorldApplication.class,args);
    }
}
```

`@SpringBootApplication` 注解，就是 Spring Boot 的核心注解。**@Spring BootApplication 标注在某个类上说明这个类是Spring Boot的主配置类**，Spring Boot就应该运行这个类的main方法来启动Spring Boot应用。

`org.springframework.boot.autoconfigure.@SpringBootApplication` 注解的代码如下：

```java
// SpringBootApplication.java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
	// omit   
}

// SpringBootConfiguration.java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration //实际上它也是一个配置类
public @interface SpringBootConfiguration {
}
```

大概可以把 `@SpringBootApplication`看作是 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 注解的集合。这三个注解的作用分别是：

- `@EnableAutoConfiguration`：启用 SpringBoot 的自动配置机制（ Spring Boot带来的）；
- `@Configuration`：允许在上下文中注册额外的 bean 或导入其他配置类；
- `@ComponentScan`： 扫描被`@Component` (`@Service`，`@Controller`)注解的 bean，注解默认会扫描启动类所在的包下所有的类 ，可以自定义不扫描某些 bean。

## Spring Boot 自动配置原理？

上面提到`@EnableAutoConfiguration` 注解，打开了Spring Boot 自动配置的功能。如何实现的呢？先说总结：

* Spring Boot 通过`@EnableAutoConfiguration`开启自动装配，通过 `SpringFactoriesLoader` 最终加载`META-INF/spring.factories`中的自动配置类实现自动装配，这些自动配置类都是以`AutoConfiguration`结尾命名的，而自动配置类的相关属性都是封装在对应的`Properties`结尾命名的类中，通过`@ConfigurationProperties`注解与全局配置文件中对应的属性进行绑定的。
* 自动配置类其实就是通过`@Conditional`按需加载的配置类，想要其生效必须引入`spring-boot-starter-xxx`包实现起步依赖。

------

@EnableAutoConfiguration用于开启自动配置功能：

```java
// EnableAutoConfiguration.java
// ... ...
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
	// omit
}

// AutoConfigurationPackage.java
// ... ...
@Import({Registrar.class})
public @interface AutoConfigurationPackage {
    // omit
}
```

@**AutoConfigurationPackage**，它用于自动配置包，点进去可以发现**@Import({Registrar.class})** 

* @Import是Spring中的底层注解，给容器中导入一个组件，导入的组件由Registrar.class指定，==将主配置类（@Spring BootApplication标注的类）的所在包及下面所在子包里面的所有组件扫描到Spring容器。==

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210508094441448.png" width="800px"/>
</div>

**@Import({AutoConfigurationImportSelector.class})**：导入组件的选择器，将所有需要导入的组件以全类名的方式返回，这些组件就会被添加到容器中。它会给容器中导入非常多的自动配置类（xxxAutoConfiguration），也就是给容器中导入这个场景需要的所有组件，并配置好这些组件。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20201016140609878.png" width="1000px"/>
</div>

查看`spring-boot-autoconfigure-2.3.3.RELEASE.jar`下的`META-INF/spring.factories`：

```properties
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.cloud.CloudAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
// omit
```

可见，Spring Boot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值，将这些值作为自动配置类导入到容器中，自动配置类就生效，帮我们进行自动配置工作。

J2EE的整体整合解决方案和自动配置都在`spring-boot-autoconfigure-2.3.3.RELEASE.jar`中。

下面以HttpEncodingAutoConfiguration（Http编码自动配置）为例：

```java
@Configuration //表示这是一个配置类，以前编写的配置文件一样，也可以给容器中添加组件
// 启动指定类的ConfigurationProperties功能，将配置文件中对应的和HttpEncodingProperties绑定起来
// 并把HttpEncodingProperties加入到IoC容器中。
@EnableConfigurationProperties(HttpEncodingProperties.class)  
// 判断当前应用是否是web应用，如果是则当前配置类生效。
@ConditionalOnWebApplication
// 判断当前项目有没有这个类CharacterEncodingFilter，SpringMVC中进行乱码解决的过滤器。
@ConditionalOnClass(CharacterEncodingFilter.class) 
// 判断配置文件中是否存在某个配置  spring.http.encoding.enabled，如果不存在，判断也是成立的。
// 即使我们配置文件中不配置pring.http.encoding.enabled=true，也是默认生效的。
@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)  
public class HttpEncodingAutoConfiguration {
  	// 它已经和Spring Boot的配置文件映射
  	private final HttpEncodingProperties properties;
  
   // 只有一个有参构造器的情况下，参数的值就会从容器中获取
  	public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
		this.properties = properties;
	}
  
    @Bean //给容器中添加一个组件，这个组件的某些值需要从properties中获取。
	@ConditionalOnMissingBean(CharacterEncodingFilter.class) //判断容器没有这个组件？
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
        // 配置文件书写内容，都是来自这些properties
        // spring.http.encoding.enabled=true
		// spring.http.encoding.charset=utf-8
		// spring.http.encoding.force=true
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
		return filter;
	}
```

根据当前不同的条件判断，决定这个配置类是否生效。一旦这个配置类生效，这个配置类就会给容器中添加各种组件，这些组件的属性是从对应的properties类中获取的，这些类里面的每一个属性又是和配置文件绑定的。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20201016235003799.png" width="800px"/>
</div>

而所有在配置文件中能配置的属性都是在`xxxProperties`类中封装的，配置文件能配置什么就可以参照某个功能对应的这个属性类。

```java
// 从配置文件中获取指定的值和bean的属性进行绑定。
@ConfigurationProperties(prefix = "spring.http.encoding")  
public class HttpEncodingProperties {

   public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");
}
```

## @Conditional 扩展注解有哪些？

 @**Conditional**派生注解：@Conditional指定的条件成立，才给容器中添加组件，配置配里面的所有内容才生效。

| @Conditional扩展注解            | 作用（判断是否满足当前指定条件）                 |
| ------------------------------- | ------------------------------------------------ |
| @ConditionalOnJava              | 系统的java版本是否符合要求                       |
| @ConditionalOnBean              | 容器中存在指定Bean                               |
| @ConditionalOnMissingBean       | 容器中不存在指定Bean                             |
| @ConditionalOnExpression        | 满足SpEL表达式指定                               |
| @ConditionalOnClass             | 系统中有指定的类                                 |
| @ConditionalOnMissingClass      | 系统中没有指定的类                               |
| @ConditionalOnSingleCandidate   | 容器中只有一个指定的Bean，或者这个Bean是首选Bean |
| @ConditionalOnProperty          | 系统中指定的属性是否有指定的值                   |
| @ConditionalOnResource          | 类路径下是否存在指定资源文件                     |
| @ConditionalOnWebApplication    | 当前是web环境                                    |
| @ConditionalOnNotWebApplication | 当前不是web环境                                  |
| @ConditionalOnJndi              | JNDI存在指定项                                   |

**自动配置类必须在一定的条件下才能生效**，我们怎么知道哪些自动配置类生效呢?

在配置文件中启用 **debug=true**属性，让控制台打印自动配置报告，这样我们就可以很方便的知道哪些自动配置类生效。

```java
=========================
AUTO-CONFIGURATION REPORT
=========================


Positive matches:（自动配置类启用的）
-----------------

   DispatcherServletAutoConfiguration matched:
      - @ConditionalOnClass found required class 'org.springframework.web.servlet.DispatcherServlet'; @ConditionalOnMissingClass did not find unwanted class (OnClassCondition)
      - @ConditionalOnWebApplication (required) found StandardServletEnvironment (OnWebApplicationCondition)
        
    
Negative matches:（没有启动，没有匹配成功的自动配置类）
-----------------

   ActiveMQAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required classes 'javax.jms.ConnectionFactory', 'org.apache.activemq.ActiveMQConnectionFactory' (OnClassCondition)

   AopAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required classes 'org.aspectj.lang.annotation.Aspect', 'org.aspectj.lang.reflect.Advice' (OnClassCondition)
```

# 安全篇

## 如何集成 Spring Boot 和 Spring Security ？

目前比较主流的安全框架有两个：

1. Spring Security
2. Apache Shiro

对于任何项目来说，安全认证总是少不了，同样适用于使用 Spring Boot 的项目。相对来说，Spring Security 现在会比 Apache Shiro 更流行。

Spring Boot 和 Spring Security 的配置方式比较简单：

1. 引入 `spring-boot-starter-security` 的依赖。
2. 继承 WebSecurityConfigurerAdapter ，添加**自定义**的安全配置。

当然，每个项目的安全配置是不同的，需要胖友自己选择。更多详细的使用，建议认真阅读如下文章：

- [《Spring Boot中 使用 Spring Security 进行安全控制》](http://blog.didispace.com/springbootsecurity/) ，快速上手。
- [《Spring Security 实现原理与源码解析系统 —— 精品合集》](http://www.iocoder.cn/Spring-Security/good-collection/) ，深入源码。
- 安全相关可看看 [《Spring Boot 十种安全措施》](https://www.jdon.com/49653) 一文。

## 如何集成 Spring Boot 和 Spring Security OAuth2 ？

参见 [《Spring Security OAuth2 入门》](http://www.iocoder.cn/Spring-Security/OAuth2-learning/) 文章，内容有点多。

## 比较一下Spring Security 和Shiro各自的优缺点 ?

由于SpringBoot官方提供了大量的非常方便的开箱即用的Starter，包括Spring Security的Starter ，使得在 SpringBoot中使用Spring Security变得更加容易，甚至只需要添加一个依赖就可以保护所有的接口，所以，如果是SpringBoot 项目，一般选择 Spring Security 。当然这只是一个建议的组合，单纯从技术上来说，无论怎么组合，都是没有问题的。Shiro和Spring Security相比，主要有如下一些特点：

1. Spring Security 是一个重量级的安全管理框架；Shiro 则是一个轻量级的安全管理框架；
2. Spring Security 概念复杂，配置繁琐；Shiro 概念简单、配置简单；
3. Spring Security 功能强大；Shiro 功能简单。

## Spring Boot 中如何解决跨域问题 ?

跨域可以在前端通过 JSONP 来解决，但是 JSONP 只可以发送 GET 请求，无法发送其他类型的请求，在 RESTful 风格的应用中，就显得非常鸡肋，因此我们推荐在后端通过 **CORS**，（Cross-origin resource sharing） 来解决跨域问题。这种解决方案并非 Spring Boot 特有的，在传统的 SSM 框架中，就可以通过 CORS 来解决跨域问题，只不过之前我们是在 XML 文件中配置CORS ，现在可以通过实现`WebMvcConfigurer`接口然后重写`addCorsMappings`方法解决跨域问题。

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowCredentials(true)
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .maxAge(3600);
    }
}
```

项目中前后端分离部署，所以需要解决跨域的问题。 我们使用cookie存放用户登录的信息，在Spring拦截器进行权限控制，当权限不符合时，直接返回给用户固定的json结果。 当用户登录以后，正常使用；当用户退出登录状态时或者token过期时，由于拦截器和跨域的顺序有问题，出现了跨域的现象。 我们知道一个http请求，先走filter，到达servlet后才进行拦截器的处理，如果我们把cors放在filter里，就可以优先于权限拦截器执行。

```java
@Configuration
public class CorsConfig {
    @Bean
    public CorsFilter corsFilter() {
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedOrigin("*");
        corsConfiguration.addAllowedHeader("*");
        corsConfiguration.addAllowedMethod("*");
        corsConfiguration.setAllowCredentials(true);
        UrlBasedCorsConfigurationSource urlBasedCorsConfigurationSource = 
            new UrlBasedCorsConfigurationSource();
        urlBasedCorsConfigurationSource.registerCorsConfiguration("/**", corsConfiguration);
        return new CorsFilter(urlBasedCorsConfigurationSource);
    }
}
```

## 什么是 CSRF 攻击？

CSRF 代表跨站请求伪造。这是一种攻击，迫使最终用户在当前通过身份验证的Web 应用程序上执行不需要的操作。CSRF 攻击专门针对状态改变请求，而不是数据窃取，因为攻击者无法查看对伪造请求的响应。

## Spring Boot 中的监视器是什么

`spring-boot-actuator` 提供 Spring Boot 的监视器功能，可帮助我们访问生产环境中正在运行的应用程序的**当前状态**。有几个指标必须在生产环境中进行检查和监控。即使一些外部应用程序可能正在使用这些服务来向相关人员触发警报消息。监视器模块公开了一组可直接作为HTTPURL访问的REST端点来检查状态。

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
    <version>2.0.4.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

```java
server.tomcat.uri-encoding=UTF-8
# 程序运行端口
server.port=8888
# 监视程序运行端口
management.server.port=8090
# 激活所有的内置Endpoints
management.endpoints.web.exposure.include=*
# 开启shutdown这个endpoint
management.endpoint.shutdown.enabled=true
```

Spring Boot 2.X 默认情况下，`spring-boot-actuator` 产生的 Endpoint 是没有安全保护的，但是 Actuator 可能暴露敏感信息。所以一般的做法是，引入 `spring-boot-start-security` 依赖，使用 Spring Security 对它们进行安全保护。

## 如何在 Spring Boot 中禁用Actuator端点安全性？

默认情况下，所有敏感的HTTP端点都是安全的，只有具有ACTUATOR角色的用户才能访问它们。安全性是使用标准的 `HttpServletRequest.isUserlnRole` 方法实施的。我们可以使用`management.security.enabled=false`来禁用安全性。只有在执行机构端点在防火墙后访问时，才建议禁用安全性。

## 如何监视所有 Spring Boot 微服务？

SpringBoot提供监视器端点以监控各个微服务的度量。这些端点对于获取有关应用程序的信息（如它们是否已启动）以及它们的组件（如数据库等）是否正常运行很有帮助。但是，使用监视器的一个主要缺点或困难是，我们必须单独打开应用程序的知识点以了解其状态或健康状况。想象一下涉及 50 个应用程序的微服务，管理员将不得不击中所有 50 个应用程序的执行终端。为了帮助我们处理这种情况，我们将使用位于的开源项目。 它建立在 `Spring Boot Actuato`r 之上，它提供了一个 Web UI，使我们能够可视化多个应用程序的度量。

# 进阶篇

## Swagger 用过吗？它用来做什么？

Swagger广泛用于可视化API，使用SwaggerUl**为前端开发人员提供在线沙箱**。Swagger 是用于生成RESTful Web服务的可视化表示的工具，规范和完整框架实现。它**使文档能够以与服务器相同的速度更新**。当通过Swagger 正确定义时，消费者可以使用最少量的实现逻辑来理解远程服务并与其进行交互。因此，Swagger 消除了调用服务时的猜测。

```java
<!--https://mvnrepository.com/artifact/io.springfox/springfox-swagger2-->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<!--https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui-->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

> 前后端分离，如何维护接口文档 ?
>
> 使用 Swagger 我们可以快速生成一个接口文档网站，接口一旦发生变化，文档就会自动更新。

## Spring Boot 项目如何热部署？

这可以使用 DEV 工具来实现。通过这种依赖关系，您可以节省任何更改，嵌入式tomcat 将重新启动。Spring Boot 有一个开发工具（DevTools）模块，它有助于提高开发人员的生产力。Java 开发人员面临的一个主要挑战是将文件更改自动部署到服务器并自动重启服务器。开发人员可以重新加载 Spring Boot 上的更改，而无需重新启动服务器。这将消除每次手动部署更改的需要。Spring Boot 在发布它的第一个版本时没有这个功能。这是开发人员最需要的功能。DevTools 模块完全满足开发人员的需求。该模块将在生产环境中被禁用。它还提供 H2 数据库控制台以更好地测试应用程序。

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

## 微服务中如何实现 session 共享?

在微服务中，一个完整的项目被拆分成多个不相同的独立的服务，各个服务独立部署在不同的服务器上，各自的 session 被从物理空间上隔离开了，但是经常，我们需要在不同微服务之间共享 session ，常见的方案就是 `Spring session + Redis` 来实现 session 共享。将所有微服务的 session 统一保存在 Redis 上，当各个微服务对 session 有相关的读写操作时，都去操作 Redis 上的 session 。这样就实现了 session 共享，Spring Session 基于 Spring 中的代理过滤器实现，使得 session 的同步操作对开发人员而言是透明的，非常简便。

##  Spring Boot 中如何实现定时任务?

定时任务也是一个常见的需求，SpringBoot 中对于定时任务的支持主要还是来自 Spring 框架。

在 SpringBoot 中使用定时任务主要有两种不同的方式，一个就是使用 Spring 中的 @Scheduled 注解，另一个则是使用第三方框架 Quartz。

- 使用Spring中的 @Scheduled的方式主要通过@Scheduled注解来实现。
- 使用Quartz，则按照Quartz的方式，定义Job和Trigger即可。

## 如何将内嵌服务器换成 Jetty ？

Spring Boot主要能够集成Tomcat 、Jetty 、Undertow三种嵌入式 web 容器。默认情况下，`spring-boot-starter-web` 模块使用 Tomcat 作为内嵌的服务器。

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   <!-- 引入web模块默认就是使用嵌入式的Tomcat作为Servlet容器 -->
</dependency>
```

所以需要去除对 `spring-boot-starter-tomcat` 模块的引用，添加 `spring-boot-starter-jetty` 模块的引用。代码如下：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion> <!-- 去除 Tomcat -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency> <!-- 引入 Jetty -->
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

添加 `spring-boot-starter-undertow` 模块的引用：

```xml
<!-- 引入web模块 -->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   <exclusions>
      <exclusion>
         <artifactId>spring-boot-starter-tomcat</artifactId>
         <groupId>org.springframework.boot</groupId>
      </exclusion>
   </exclusions>
</dependency>
<!--引入其他的Servlet容器-->
<dependency>
   <artifactId>spring-boot-starter-undertow</artifactId>
   <groupId>org.springframework.boot</groupId>
</dependency>
```

## 如何集成 Spring Boot 和 Spring MVC ？

1. 引入 `spring-boot-starter-web` 的依赖。

2. 实现 WebMvcConfigurer 接口，可添加自定义的 Spring MVC 配置。

   > 因为 Spring Boot 2 基于 JDK 8 的版本，而 JDK 8 提供 `default` 方法，所以 Spring Boot 2 废弃了 WebMvcConfigurerAdapter 适配类，直接使用 WebMvcConfigurer 即可。

   ```java
   // WebMvcConfigurer.java
   public interface WebMvcConfigurer {
   
       /** 配置路径匹配器 **/
       default void configurePathMatch(PathMatchConfigurer configurer) {}
       
       /** 配置内容裁决的一些选项 **/
       default void configureContentNegotiation(ContentNegotiationConfigurer configurer) { }
   
       /** 异步相关的配置 **/
       default void configureAsyncSupport(AsyncSupportConfigurer configurer) { }
   
       default void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) { }
   
       default void addFormatters(FormatterRegistry registry) {
       }
   
       /** 添加拦截器 **/
       default void addInterceptors(InterceptorRegistry registry) { }
   
       /** 静态资源处理 **/
       default void addResourceHandlers(ResourceHandlerRegistry registry) { }
   
       /** 解决跨域问题 **/
       default void addCorsMappings(CorsRegistry registry) { }
   
       default void addViewControllers(ViewControllerRegistry registry) { }
   
       /** 配置视图解析器 **/
       default void configureViewResolvers(ViewResolverRegistry registry) { }
   
       /** 添加参数解析器 **/
       default void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
       }
   
       /** 添加返回值处理器 **/
       default void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> handlers) { }
   
       /** 这里配置视图解析器 **/
       default void configureMessageConverters(List<HttpMessageConverter<?>> converters) { }
   
       /** 配置消息转换器 **/
       default void extendMessageConverters(List<HttpMessageConverter<?>> converters) { }
   
      /** 配置异常处理器 **/
       default void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) { }
   
       default void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) { }
   
       @Nullable
       default Validator getValidator() { return null; }
   
       @Nullable
       default MessageCodesResolver getMessageCodesResolver() {  return null; }
   
   }
   ```

------

在使用 Spring MVC 时，我们一般会做如下几件事情：

1. 实现自己项目需要的拦截器，并在 WebMvcConfigurer 实现类中配置。可参见 [MVCConfiguration](https://github.com/YunaiV/oceans/blob/2a2d3746905f1349e260e88049e7e28346c7648f/bff/webapp-bff/src/main/java/cn/iocoder/oceans/webapp/bff/config/MVCConfiguration.java) 类。
2. 配置 `@ControllerAdvice` + `@ExceptionHandler` 注解，实现全局异常处理。可参见 [GlobalExceptionHandler](https://github.com/YunaiV/oceans/blob/2a2d3746905f1349e260e88049e7e28346c7648f/bff/webapp-bff/src/main/java/cn/iocoder/oceans/webapp/bff/config/GlobalExceptionHandler.java) 类。
3. 配置 `@ControllerAdvice` ，实现 ResponseBodyAdvice 接口，实现全局统一返回。可参见 [GlobalResponseBodyAdvice](https://github.com/YunaiV/oceans/blob/2a2d3746905f1349e260e88049e7e28346c7648f/bff/webapp-bff/src/main/java/cn/iocoder/oceans/webapp/bff/config/GlobalResponseBodyAdvice.java) 。

当然，有一点需要注意，WebMvcConfigurer、ResponseBodyAdvice、`@ControllerAdvice`、`@ExceptionHandler` 接口，都是 Spring MVC 框架自身已经有的东西。

- `spring-boot-starter-web` 的依赖，帮我们解决的是 Spring MVC 的依赖以及相关的 Tomcat 等组件。

## Spring Boot 支持哪些日志框架？

市场上存在非常多的日志框架：

JUL、JCL、Jboss-logging、logback、log4j、log4j2、slf4j... ...

| 日志门面  （日志的抽象层）                                                                                  | 日志实现                                                       |
| ----------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| ~~JCL（Jakarta  Commons Logging）~~  <br>SLF4j（Simple  Logging Facade for Java） <br>**~~jboss-logging~~** | Log4j <br>JUL（java.util.logging）<br/>Log4j2 <br/>**Logback** |

Spring框架默认是用JCL，Spring Boot的底层是Spring框架。Spring Boot在框架内容部使用JCL，spring-boot-starter-logging采用了 **SLF4j+Logback**的形式，Spring Boot也能自动适配（JUL、log4j2、logback） 并简化配置。

**Spring Boot能自动适配所有的日志，而且底层使用slf4j+logback的方式记录日志，引入其他框架的时候，只需要把这个框架依赖的日志框架排除掉即可。**

# 整合篇

* 如何集成 Spring Boot 和 JPA ？
* 如何集成 Spring Boot 和 MyBatis ？
* 如何集成 Spring Boot 和 RabbitMQ ？
* 如何集成 Spring Boot 和 Kafka ？
* 如何集成 Spring Boot 和 RocketMQ ？
* 如何集成 Spring Boot 和 Docker?
* ... ...