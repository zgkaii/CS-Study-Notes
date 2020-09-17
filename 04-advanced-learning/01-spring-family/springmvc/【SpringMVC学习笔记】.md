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

##### 2.常用注解
1）@RequestParam
* 作用：把请求中的指定名称的参数传递给控制器中的形参赋值  
* 属性：  
value：请求参数中的名称  
required：请求参数中是否必须提供此参数。默认值：true。表示必须提供，如果不提供将报错。
```jsp
<a href="anno/testRequestParam?name=lisi">RequestParam</a>
```
```java
    @RequestMapping("/testRequestParam")
    public String testRequestParam(@RequestParam(name = "name") String username) {
        System.out.println("Finished...");
        System.out.println(username);
        return "success";
    }
```
2）@RequestBody
* 作用：用于获取请求体的内容（注意：get方法不可以）
* 属性：
required：是否必须有请求体，默认值是true。当取值为 true 时,get 请求方式会报错。如果取值为 false，get 请求得到是 null。

```jsp
    <form action="anno/testRequestBody" method="post">
        用户姓名：<input type="text" name="username"/><br/>
        用户年龄：<input type="text" name="age"/><br/>
        用户年龄：<input type="text" name="sex"/><br/>
        <input type="submit" value="提交"/>
```
```java
    @RequestMapping("/testRequestBody")
    public String testRequestBody(@RequestBody String body) {
        System.out.println("Finished...");
        System.out.println(body);
        return "success";
    }
```
4）@PathVaribale
作用：  
用于绑定 url 中的占位符。例如：请求 url 中 /delete/{id}，这个{id}就是 url 占位符。  
url 支持占位符是 spring3.0 之后加入的。是 springmvc 支持 rest 风格 URL 的一个重要标志。  
属性：  
value：用于指定 url 中占位符名称。  
required：是否必须提供占位符。  
```jsp
    <a href="anno/testPathVariable/10">testPathVariable</a>
```
```java
    @RequestMapping(value = "/testPathVariable/{sid}")
    public String testPathVariable(@PathVariable(name = "sid") String id) {
        System.out.println("Finished...");
        System.out.println(id);
        return "success";
    }
```
5）@RequestHeader
作用：  
用于获取请求消息头。
属性：  
value：提供消息头名称  
required：是否必须有此消息头  
注：  
在实际开发中一般不怎么用  
```jsp
    <a href="anno/testRequestHeader">RequestHeader</a>
```
```java
    @RequestMapping(value = "/testRequestHeader")
    public String testRequestHeader(@RequestHeader(value = "Accept") String header, HttpServletRequest request, HttpServletResponse response) throws IOException {
        System.out.println("Finished...");
        System.out.println(header);
        // return "success";
        // response.sendRedirect(request.getContextPath()+"/anno/testCookieValue");
        return "redirect:/param.jsp";
    }
```
6）@CookieValue
作用：  
用于把指定 cookie 名称的值传入控制器方法参数。  
属性：  
value：指定 cookie 的名称。  
required：是否必须有此 cookie。  
```jsp
    <a href="anno/testCookieValue">CookieValue</a>
```
```java
    @RequestMapping(value = "/testCookieValue")
    public String testCookieValue(@CookieValue(value = "JSESSIONID") String cookieValue) {
        System.out.println("Finished...");
        System.out.println(cookieValue);
        return "success";
    }
```
7）@ModelAttribute
作用：  
该注解是 SpringMVC4.3 版本以后新加入的。它可以用于修饰方法和参数。  
出现在方法上，表示当前方法会在控制器的方法执行之前，先执行。它可以修饰没有返回值的方法，也可以修饰有具体返回值的方法。  
出现在参数上，获取指定的数据给参数赋值。  
属性：  
value：用于获取数据的 key。key 可以是 POJO 的属性名称，也可以是 map 结构的 key。  
应用场景：  
当表单提交数据不是完整的实体类数据时，保证没有提交数据的字段使用数据库对象原来的数据。
例如：  
我们在编辑一个用户时，用户有一个创建信息字段，该字段的值是不允许被修改的。在提交表单数据是肯定没有此字段的内容，一旦更新会把该字段内容置为 null，此时就可以使用此注解解决问题。
```jsp
    ${ msg }

    ${sessionScope}
```
```jsp
    <form action="anno/testModelAttribute" method="post">
        用户姓名：<input type="text" name="uname"/><br/>
        用户年龄：<input type="text" name="age"/><br/>
        <input type="submit" value="提交"/>
    </form>
```
```java
    @RequestMapping(value = "/testModelAttribute")
    public String testModelAttribute(@ModelAttribute("abc") User user) {
        System.out.println("testModelAttribute Finished...");
        System.out.println(user);
        return "success";
    }

    @ModelAttribute
    public void showUser(String name, Map<String, User> map) {
        System.out.println("showUser Finished...");
        // 通过用户查询数据库（模拟）
        User user = new User();
        user.setName(name);
        user.setAge(20);
        user.setDate(new Date());
        map.put("abc", user);
    }
```
8）@SessionAttribute
作用：  
用于多次执行控制器方法间的参数共享。  
属性：  
value：用于指定存入的属性名称；  
type：用于指定存入的数据类型。  
```jsp
    <a href="anno/testSessionAttributes">testSessionAttributes</a>
    <a href="anno/getSessionAttributes">getSessionAttributes</a>
    <a href="anno/delSessionAttributes">delSessionAttributes</a>
```
```java
    /**
     * SessionAttributes的注解
     *
     * @return
     */
    @RequestMapping(value = "/testSessionAttributes")
    public String testSessionAttributes(Model model) {
        System.out.println("testSessionAttributes...");
        // 底层会存储到request域对象中
        model.addAttribute("msg", "美美");
        return "success";
    }

    /**
     * 获取值
     *
     * @param modelMap
     * @return
     */
    @RequestMapping(value = "/getSessionAttributes")
    public String getSessionAttributes(ModelMap modelMap) {
        System.out.println("getSessionAttributes...");
        String msg = (String) modelMap.get("msg");
        System.out.println(msg);
        return "success";
    }

    /**
     * 清除
     *
     * @param status
     * @return
     */
    @RequestMapping(value = "/delSessionAttributes")
    public String delSessionAttributes(SessionStatus status) {
        System.out.println("getSessionAttributes...");
        status.setComplete();
        return "success";
    }
```

##### 3.REST 风格 URL
1）什么是 rest：
REST（英文：Representational State Transfer，简称 REST）描述了一个架构样式的网络系统，
比如 web 应用程序。它首次出现在 2000 年 Roy Fielding 的博士论文中，他是 HTTP 规范的主要编写者之
一。在目前主流的三种 Web 服务交互方案中，REST 相比于 SOAP（Simple Object Access protocol，简单
对象访问协议）以及 XML-RPC 更加简单明了，无论是对 URL 的处理还是对 Payload 的编码，REST 都倾向于用更
加简单轻量的方法设计和实现。值得注意的是 REST 并没有一个明确的标准，而更像是一种设计的风格。
它本身并没有什么实用性，其核心价值在于如何设计出符合 REST 风格的网络接口。

2）restful 的优点  
它结构清晰、符合标准、易于理解、扩展方便，所以正得到越来越多网站的采用。
3）restful 的特性：  
资源（Resources）：  
网络上的一个实体，或者说是网络上的一个具体信息。  
它可以是一段文本、一张图片、一首歌曲、一种服务，总之就是一个具体的存在。可以用一个 URI（统一
资源定位符）指向它，每种资源对应一个特定的 URI 。要获取这个资源，访问它的 URI 就可以，因此 URI 即为每一个资源的独一无二的识别符。  

表现层（Representation）：  
把资源具体呈现出来的形式，叫做它的表现层 （Representation）。  
比如，文本可以用 txt 格式表现，也可以用 HTML 格式、XML 格式、JSON 格式表现，甚至可以采用二进制格式。  

状态转化（State Transfer）：  
每发出一个请求，就代表了客户端和服务器的一次交互过程。  
HTTP 协议，是一个无状态协议，即所有的状态都保存在服务器端。因此，如果客户端想要操作服务器，
必须通过某种手段，让服务器端发生“状态转化”（State Transfer）。而这种转化是建立在表现层之上的，所以
就是 “表现层状态转化”。具体说，就是 HTTP 协议里面，四个表示操作方式的动词：GET 、POST 、PUT、
DELETE。它们分别对应四种基本操作：GET 用来获取资源，POST 用来新建资源，PUT 用来更新资源，DELETE 用来
删除资源。  

restful 的示例：    
/account/1 HTTP GET ： 得到 id = 1 的 account  
/account/1 HTTP DELETE： 删除 id = 1 的 account  
/account/1 HTTP PUT： 更新 id = 1 的 account   
### 参考资料
[三层架构与MVC模式](https://www.cnblogs.com/zdxster/p/5305187.html)
[SpringMVC执行原理](https://blog.csdn.net/GavinLi2588/article/details/78696867)
[SpringMVC中的参数绑定总结](https://blog.csdn.net/eson_15/article/details/51718633)