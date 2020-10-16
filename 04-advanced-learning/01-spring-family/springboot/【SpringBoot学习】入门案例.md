## 一、Spring Boot 简介

J2EE的开发略显笨重，配置繁多、 部署流程复杂、第三方技术集成难度大，都很大程度上降低了开发效率。SpringBoot应运而生，SpringBoot简化了Spring应用开发，整合了Spring技术栈， 去繁从简，just run就能创建一个独立的、产品级别的应用，为J2EE开发提供了一站式解决方案。

* 快速创建独立运行的Spring项目以及与主流框架集成 
* 使用嵌入式的Servlet容器，应用无需打成WAR包 
*  starters自动依赖与版本控制 
* 大量的自动配置，简化开发，也可修改默认值 
*  无需配置XML，无代码生成，开箱即用 
*  准生产环境的运行时应用监控 
*  与云计算的天然集成

## 二、入门案例：HelloWorld

一个功能：浏览器发送hello请求，服务器接受请求并处理，响应Hello World字符串。

### 2.1 创建maven工程

### 2.2 引入starter 

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.3.RELEASE</version>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```

### 2.3 编写主程序

```java
/**
 *  @SpringBootApplication 标注主程序类，说明这是一个Spring Boot应用
 */
@SpringBootApplication
public class HelloWorldApplication {
    public static void main(String[] args) {

        // 启动Spring应用
        SpringApplication.run(HelloWorldApplication.class,args);
    }
}
```

### 2.4 编写Controller

```java
@Controller
public class HelloController {
    
    @ResponseBody
    @RequestMapping("/hello")
    public String hello(){
        return "Hello World!";
    }
}
```

### 2.5 运行主程序

打开本地Terminal：

```shell script
C:\WorkSpace\IdeaProjects\springboot-learn>curl http://localhost:8080/hello
Hello World!
```

打开Google，访问http://localhost:8080/hello

![](https://img-blog.csdnimg.cn/20201016103240519.png)

### 2.6 简化部署

```xml
 <!-- 这个插件，可以将应用打包成一个可执行的jar包-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

将这个应用打成jar包，直接使用`java -jar`的命令进行执行。

```shell script
C:\WorkSpace\IdeaProjects\springboot-learn>cd springboot-helloworld  (切换到pom.xml所在目录)

C:\WorkSpace\IdeaProjects\springboot-learn\springboot-helloworld>mvn clean package -Dmaven.test.skip
[INFO] Scanning for projects...
[INFO]
[INFO] -----------------------< com.kai:springboot-helloworld >-----------------------
[INFO] Building springboot-helloworld 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
       ---------------------------------snip-----------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  11.092 s
[INFO] Finished at: 2020-10-16T10:37:55+08:00
[INFO] ------------------------------------------------------------------------

C:\WorkSpace\IdeaProjects\springboot-learn\springboot-helloworld>cd target

C:\WorkSpace\IdeaProjects\springboot-learn\springboot-helloworld\target>dir
 驱动器 D 中的卷是 Data
 卷的序列号是 B8A9-CEA2

 C:\WorkSpace\IdeaProjects\springboot-learn\springboot-helloworld\target 目录

2020/10/16  10:37    <DIR>          .
2020/10/16  10:37    <DIR>          ..
2020/10/16  10:37    <DIR>          classes
2020/10/16  10:37    <DIR>          generated-sources
2020/10/16  10:37    <DIR>          maven-archiver
2020/10/16  10:37    <DIR>          maven-status
2020/10/16  10:37        16,519,536 springboot-helloworld-1.0-SNAPSHOT.jar
2020/10/16  10:37             3,081 springboot-helloworld-1.0-SNAPSHOT.jar.original
               2 个文件     16,522,617 字节
               6 个目录 34,072,743,936 可用字节

C:\WorkSpace\IdeaProjects\springboot-learn\springboot-helloworld\target>java -jar springboot-helloworld-0.0.1-SNAPSHOT.jar
```

打开Google，访问http://localhost:8080/hello

![](https://img-blog.csdnimg.cn/20201016103240519.png)

## 三、Hello World案例分析

### 3.1 POM文件

#### （1）父项目

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.3.RELEASE</version>
</parent>

它的父项目是:真正管理Spring Boot应用里面的所有依赖版本
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-dependencies</artifactId>
  <version>2.3.3.RELEASE</version>
  <relativePath>../../spring-boot-dependencies</relativePath>
</parent>
```

Spring Boot的版本仲裁中心，所以再在dependencies里面管理的依赖不需要写版本。

#### （2）场景启动器

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

**spring-boot-starter**-==web==

​	spring-boot-starter：spring-boot场景启动器，帮我们导入了web模块正常运行所依赖的组件。

Spring Boot将所有的功能场景都抽取出来，做成一个个的starter（启动器），只需要在项目里面引入这些starter相关场景的所有依赖都会导入进来，要用什么功能就导入什么场景的启动器。

### 3.2 主程序类

```java
/**
 *  @SpringBootApplication 标注主程序类，说明这是一个Spring Boot应用
 */
@SpringBootApplication
public class HelloWorldApplication {
    public static void main(String[] args) {

        // 启动Spring应用
        SpringApplication.run(HelloWorldApplication.class,args);
    }
}
```

@SpringBootApplication 标注在某个类上说明这个类是SpringBoot的主配置类，SpringBoot就应该运行这个类的main方法来启动SpringBoot应用。

```java
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
```

#### (1) @SpringBootConfiguration

@**SpringBootConfiguration**标注在某个类上，表示这是一个Spring Boot的主配置类，SpringBoot启动这个类的main方法启动应用。

```java
@Configuration
public @interface SpringBootConfiguration {
```

* @**Configuration**：用于指定当前类是一个 Spring 配置类，当创建容器时会从该类上加载注解。

```java
@Component
public @interface Configuration {
```

* @**Component** 标注一个类为Spring容器的Bean，（把普通pojo实例化到Spring容器中，相当于配置文件中的`<bean id="" class=""/>`）

#### (2) @EnableAutoConfiguration

开启自动配置功能

```java
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
```

@**AutoConfigurationPackage**：自动配置包

```java
@Import({Registrar.class})
public @interface AutoConfigurationPackage {
```

* @**Import**({Registrar.class})  
* Spring的底层注解@Import，给容器中导入一个组件，导入的组件由Registrar.class指定，==将主配置类（@SpringBootApplication标注的类）的所在包及下面所在子包里面的所有组件扫描到Spring容器。==

@**Import**({AutoConfigurationImportSelector.class})

AutoConfigurationImportSelector：导入组件的选择器，将所有需要导入的组件以全类名的方式返回，这些组件就会被添加到容器中。会给容器中导入非常多的自动配置类（xxxAutoConfiguration），就是给容器中导入这个场景需要的所有组件，并配置好这些组件。

查看源码：

![](https://img-blog.csdnimg.cn/20201016140609878.png)

查看spring.factories：

```properties
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
\--- snip ---\
\--- snip ---\
\--- snip ---\
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.client.WebServiceTemplateAutoConfiguration
```

可见，==Spring Boot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值，将这些值作为自动配置类导入到容器中，自动配置类就生效，帮我们进行自动配置工作==。

J2EE的整体整合解决方案和自动配置都在`spring-boot-autoconfigure-2.3.3.RELEASE.jar`中。