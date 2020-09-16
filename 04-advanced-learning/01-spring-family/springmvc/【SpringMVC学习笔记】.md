### 一、三层架构与MVC
##### 1.三层架构
开发服务器端程序，架构一般基于两种形式：一种是 C/S 架构，也就是客户端/服务器，另一种是 B/S 架构，也就
是浏览器/服务器。
在 JavaEE 开发中，几乎全都是基于 B/S 架构的开发，B/S架构又分成了三层架构：

* 表现层（UI）：  
通俗讲就是展现给用户的界面，即用户在使用一个系统的时候他的所见所得。
* 业务逻辑层（BLL）：  
针对具体问题的操作，也可以说是对数据层的操作，对数据业务逻辑处理。 
* 数据访问层（DAL）：  
该层所做事务直接操作数据库，针对数据的增添、删除、修改、更新、查找等。 

##### 2.MVC模型

MVC全称是Model View Controller，意为模型视图控制器，主要作用与表现层（UI）。
* 视图（View）：  
负责界面的显示，以及与用户的交互功能，例如表单、网页等。

* 控制器（Controller）：  
可以理解为一个分发器，用来决定对于视图发来的请求，需要用哪一个模型来处理，以及处理完后需要跳回到哪一个视图。即用来连接视图和模型。

实际开发中，通常用控制器对客户端的请求数据进行封装（如将form表单发来的若干个表单字段值，封装到一个实体对象中），然后调用某一个模型来处理此请求，最后再转发请求（或重定向）到视图（或另一个控制器）。

* 模型（Model）：  
模型持有所有的数据、状态和程序逻辑。模型接受视图数据的请求，并返回最终的处理结果。

##### 3.SpringMVC的概述
SpringMVC 是一种基于 Java 的实现 MVC 设计模型的请求驱动类型的轻量级 Web 框架，属于 SpringFrameWork 的后续产品，已经融合在 Spring Web Flow 里面。Spring 框架提供了构建 Web 应用程序的全功
能 MVC 模块。  
使用 Spring 可插入的 MVC 架构，从而在使用 Spring 进行 WEB 开发时，可以选择使用 Spring的 Spring MVC 框架或集成其他 MVC 开发框架，如 Struts1(现在一般不用)，Struts2 等。  
SpringMVC 已经成为目前最主流的 MVC 框架之一，并且随着 Spring3.0 的发布，全面超越 Struts2，成为最优秀的 MVC 框架。

### 入门案例及分析
##### 1.创建WEB工程，引入依赖
```xml
    <properties>
        <spring.version>5.0.2.RELEASE</spring.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.0</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
```
##### 2.配置前端控制器（DispatcherServlet）
```xml
<web-app>
  <display-name>Archetype Created Web Application</display-name>
  <!-- 配置 spring mvc 的前端控制器 -->
  <servlet>
    <servlet-name>springMVC</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--全局化初始参数-->
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springmvc.xml</param-value>
    </init-param>
    <!-- 配置servlet启动时加载对象 -->
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>springMVC</servlet-name>
    <!-- 处理所有URL -->
    <url-pattern>/</url-pattern>
  </servlet-mapping>
  
</web-app>
```
##### 3.编写springmvc.xml的配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/mvc
            http://www.springframework.org/schema/mvc/spring-mvc.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd">
    <!-- 1.配置spring创建容器时要扫描的包 -->
    <context:component-scan base-package="com.mvc"></context:component-scan>
    <!-- 2.配置视图解析器 -->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--文件所在目录-->
        <property name="prefix" value="/WEB-INF/pages/"></property>
        <!--文件后缀名-->
        <property name="suffix" value=".jsp"></property>
    </bean>
    <!-- 3.配置spring开启注解mvc的支持-->
    <!-- 自动加载 RequestMappingHandlerMapping（处理映射器）和RequestMappingHandlerAdapter（处理适配器）-->
    <mvc:annotation-driven></mvc:annotation-driven>
</beans>
```
##### 4.编写index.jsp
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h3>学习SpringMVC</h3>
    <a href="hello">入门</a>
</body>
</html>
```
##### 5.编写HelloController
```java
@Controller
public class HelloController {

    @RequestParam(path = "/hello")
    public String sayHello(){
        System.out.println("Hello SpringMVC!");
        return "success";
    }
}
```
##### 6.编写success.jsp的页面
在WEB-INF目录下创建pages文件夹，编写success.jsp的成功页面：
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h3>入门成功！！</h3>
</body>
</html>
```
##### 7.启动Tomcat
![](https://img-blog.csdnimg.cn/20200916142357371.png#pic_center)

==入门案例简易分析：==
每次启动Tomcat服务器的时候，SpringMVC执行流程如下：
第一步：  
因为配置了`<load-on-startup>1</load-on-startup>`标签，应用启动时，即创建DispatcherServlet控制器，同时根据init-param加载springmvc.xml配置文件。

第二步：  
`context:component-scan`标签开启了注解扫描功能，HelloController对象就会被创建。

第三步：  
从index.jsp发送请求，请求`hello`会先到达DispatcherServlet核心控制器，根据@GetMapping注解找到执行的具体方法`sayHello()`。

第四步：  
在收到执行方法的返回值，springmvc配置文件视图解析器（InternalResourceViewResolver），去指定的目录下查找读取指定名称的JSP文件。

第五步：  
Tomcat服务器渲染页面，做出响应。

### SpringMVC请求参数的绑定

##### 1.请求参数格式
表单中请求参数都是基于`key=value`格式，比如`username=?&password=?`。
##### 2.请求参数值的数据类型
都是字符串类型的各种值。    
##### 3.请求参数值要绑定的目标类型#
Controller类中的方法参数，比如基本类型、POJO类型、集合类型等。  
* 基本类型参数：  
包括基本类型和 String 类型  
* POJO 类型参数：  
包括实体类，以及关联的实体类   
* 数组和集合类型参数：  
包括 List 结构和 Map 结构的集合（包括数组）  

















### 附录
##### 1.解决maven项目创建webapp过慢的问题
尽管maven配有阿里镜像源，但是创建还是花了不少时间：
![](https://img-blog.csdnimg.cn/20200916103953245.png#pic_center)

创建项目时，properties中添加一组Maven Property:  
Name: archetypeCatalog  
Value: internal  
![](https://img-blog.csdnimg.cn/20200916104432764.png#pic_center)

### 参考资料
[三层架构与MVC模式](https://www.cnblogs.com/zdxster/p/5305187.html)
[SpringMVC执行原理](https://blog.csdn.net/GavinLi2588/article/details/78696867)
[SpringMVC中的参数绑定总结](https://blog.csdn.net/eson_15/article/details/51718633)