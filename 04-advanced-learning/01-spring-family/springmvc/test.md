
# 响应数据与结果视图

### 一、返回值分类
SpringMVC响应数据类型可以是String返回值、void返回值、ModelAndView返回值。
##### 1.字符串
Controller方法返回字符串可以指定逻辑视图的名称，根据视图解析器为物理视图的地址。  
示例代码：
```java
@Controller
@RequestMapping("/user")
public class UserController {

    @RequestMapping(value="/initUpdate")
    public String initUpdate(Model model){
        System.out.println("The method is executed ...");
        // 模拟从数据库中查询数据
        User user = new User();
        user.setUsername("Harry");
        user.setPassword("12345");
        user.setAge(20);
        // model对象
        model.addAttribute("user",user);
        return "update";
    }
}
```
update.jsp:
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false"%>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h3>修改用户</h3>

    <form action="user/update" method="post">
        姓名：<input type="text" name="username" value="${ user.username }"><br>
        密码：<input type="text" name="password" value="${ user.password }"><br>
        年龄：<input type="text" name="age" value="${ user.age }"><br>
        <input type="submit" value="提交">
    </form>
</body>
</html>
```
执行结果：  
![](https://img-blog.csdnimg.cn/20200917151440952.png)  

##### 2.void
Servlet 原始 API 可以作为控制器中方法的参数。
在 controller 方法形参上可以定义 request 和 response，使用 request 或 response 指定响应结果：  
（1）使用 request 转向页面  
（2）通过 response 页面重定向  
（3）也可以通过 response 直接指定响应结果  
```java
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/initAdd")
    public void initAdd(HttpServletRequest request, HttpServletResponse response) throws Exception {
        System.out.println("The testVoid method is executed ...");
        // 情况一:请求转发
        // request.getRequestDispatcher("/WEB-INF/pages/update.jsp").forward(request,response);

        // 情况二:重定向
        // response.sendRedirect(request.getContextPath()+"/index.jsp");

        // 设置中文乱码
        response.setCharacterEncoding("UTF-8");
        response.setContentType("text/html;charset=UTF-8");

        // 情况三:直接会进行响应
        response.getWriter().print("你好");
        return;
    }
}
```
##### 3.ModelAndView
 ModelAndView对象是Spring提供的一个对象，可以用来调整具体的JSP视图。
 示例代码：
 ```java
@Controller
@RequestMapping("/user")
public class UserController {
    @RequestMapping(value = "/findAll")
    public ModelAndView findAll() throws Exception{
        // 创建ModelAndView对象
        ModelAndView mv = new ModelAndView();
        System.out.println("The testVoid method is executed ...");
        // 模拟从数据库中查询出User对象
        User user = new User();
        user.setUsername("wanger");
        user.setPassword("456");
        user.setAge(20);

        // 把user对象存储到mv对象中，也会把user对象存入到request对象
        mv.addObject("user",user);

        // 跳转到哪个页面
        mv.setViewName("list");

        return mv;
    }
}
```
list.jsp:
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h3>success</h3>

    ${user.username}
    ${user.password}
    ${user.age}
</body>
</html> 
```
结果：  
![](https://img-blog.csdnimg.cn/20200917160737113.png#pic_center)
### 二、转发和重定向
##### 1.请求转发转发
controller 方法在提供了 String 类型的返回值之后，默认就是请求转发。  
代码实例：
```java
    /**
     * 使用关键字的方式进行转发或者重定向
     * @return
     */
    @RequestMapping("/testForwardOrRedirect")
    public String testForwardOrRedirect(){
        System.out.println("The testForwardOrRedirect method is executed ...");

        // 请求转发
        return "forward:/WEB-INF/pages/success.jsp";
    }
```
==注意：==
如果用了 formward：则路径必须写成实际视图 url，不能写逻辑视图。它相当于“request.getRequestDispatcher("url").forward(request,response)”。

##### 2.重定向
redirect相当于“response.sendRedirect(url)”。如果是重定向到 jsp 页面，则 jsp 页面不能写在 WEB-INF 目录中，否则无法找到。

代码实例：
```java
    /**
     * 使用关键字的方式进行转发或者重定向
     * @return
     */
    @RequestMapping("/testForwardOrRedirect")
    public String testForwardOrRedirect(){
        System.out.println("The testForwardOrRedirect method is executed ...");

        // 重定向
        return "redirect:/index.jsp";
    }
```
### 三、json数据交互

#### 1.JSON概述
JSON（JavaScript Object Notation, JS 对象标记）是一种轻量级的数据交换格式。与 XML 一样，JSON 也是基于纯文本的数据格式。它有对象结构和数组结构两种数据结构。
* 对象结构
例如：
```json
{
    "pname":"zhangsan",
    "password":"123",
    "page":10
}
```
* 数组结构
例如：
```json
{
    "name":"lisi",
    "sex":"male",
    "hobby":["football","CS"]，
    "college":{
        "cname":"Tsinghua University",
        "city":"BeiJing"
    }
}
```
在使用注解开发时需要用到两个重要的 JSON 格式转换注解，分别是 @RequestBody 和 @ResponseBody。  
@RequestBody：用于将请求体中的数据绑定到方法的形参中，该注解应用在方法的形参上。  
@ResponseBody：用于直接返回 return 对象，该注解应用在方法上。  

#### 2.@ResponseBody响应json数据

##### 1.mvc:resources标签配置不过滤
* location元素表示webapp目录下的包下的所有文件
* mapping元素表示以/static开头的所有请求路径，如/static/xx 或者/static/a/xx
```xml
    <!--前端控制器，哪些静态资源不拦截-->
    <mvc:resources location="/css/" mapping="/css/**"/>
    <mvc:resources location="/images/" mapping="/images/**"/>
    <mvc:resources location="/js/" mapping="/js/**"/>
```
##### 2.使用@RequestBody获取请求体数据
```jsp
<script src="js/jquery.min.js"></script>

    <script>
        // 页面加载，绑定单击事件
        $(function(){
            $("#btn").click(function(){
                // alert("hello btn");
                // 发送ajax请求
                $.ajax({
                    // 编写json格式，设置属性和值
                    url:"user/testJson",
                    contentType:"application/json;charset=UTF-8",
                    data: '{"userName":"logan","password":123,"age":20}',
                    dataType:"json",
                    type:"post",
                    success:function(data){
                        // data服务器端响应的json的数据，进行解析
                        alert(data);
                        alert(data.username);
                        alert(data.password);
                        alert(data.age);
                    }
                });
            });
        });
    </script>
```
```java
    @RequestMapping("/testJson")
    public void testJson(@RequestBody String body) {
        System.out.println(body);
    }
```
结果：`{"userName":"logan","password":123,"age":20}`
#####  使用@RequestBody注解把json的字符串转换成JavaBean的对象
```jsp
        $(function(){
            // 绑定点击事件
            $("#btn").click(function(){
                $.ajax({
                    url:"user/testJson",
                    contentType:"application/json;charset=UTF-8",
                    data:'{"addressName":"aa","addressNum":100}',
                    dataType:"json",
                    type:"post",
                    success:function(data){
                        alert(data);
                        alert(data.username);
                        alert(data.password);
                        alert(data.age);                        
                    }
                });
            });
        });
```
```java
    @RequestMapping("/testJson")
    public void testJson(@RequestBody User user) {
        System.out.println(user);
    }
```
结果：`cn.mvc.dao.User@56614914 `

#####  使用@ResponseBody注解把JavaBean对象转换成json字符串
```jsp
        $(function () {
            // 绑定点击事件
            $("#btn").click(function () {
                $.ajax({
                    url: "user/testJson",
                    contentType: "application/json;charset=UTF-8",
                    data: '{"userName":"logan","password":123,"age":20}',
                    dataType: "json",
                    type: "post",
                    success: function (data) {
                        alert(data);
                        alert(data.username);
                        alert(data.password);
                        alert(data.age);  
                    }
                });
            });
        });
```
```java
    @RequestMapping("/testJson")
    public @ResponseBody User testJson(@RequestBody User user) {
        System.out.println(user);
        user.setUsername("Harry");
        
        return user;
    }
```
结果：  
![](https://img-blog.csdnimg.cn/2020091717200944.png)


### 参考资料
[HTTP中的重定向和请求转发的区别](https://blog.csdn.net/meiyalei/article/details/2129120)
[重定向与转发](https://www.liaoxuefeng.com/wiki/1252599548343744/1328761739935778)