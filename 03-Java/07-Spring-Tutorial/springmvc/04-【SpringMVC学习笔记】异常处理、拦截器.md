- [一、SpringMVC的异常处理](#一springmvc的异常处理)
    - [1. 自定义异常类](#1-自定义异常类)
    - [2. 自定义异常处理器](#2-自定义异常处理器)
    - [3. 配置异常处理器](#3-配置异常处理器)
    - [4.测试](#4测试)
- [二、SpringMVC 中的拦截器](#二springmvc-中的拦截器)
    - [1. 拦截器的概述](#1-拦截器的概述)
    - [2.自定义拦截器](#2自定义拦截器)
    - [3.HandlerInterceptor接口中的方法](#3handlerinterceptor接口中的方法)
    - [4. 配置多个拦截器](#4-配置多个拦截器)
- [参考资料](#参考资料)
### 一、SpringMVC的异常处理
 Controller调用service，service调用dao，异常都是向上抛出的，最终有DispatcherServlet找异常处理器进
行异常的处理。
<img src="https://img-blog.csdnimg.cn/20200917222310199.png" width="60%" alt=""/>

##### 1. 自定义异常类
```java
@Data
@AllArgsConstructor
public class SysException extends Exception{

    /**
     *  异常提示信息
     */
    private String message;

}
```
##### 2. 自定义异常处理器
```java
public class SysExceptionResolver implements HandlerExceptionResolver {

    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        // 获取到异常对象
        SysException e = null;
        // 获取到异常对象
        if (ex instanceof SysException) {
            e = (SysException) ex;
        } else {
            e = new SysException("请联系管理员...");
        }
        // 创建ModelAndView对象
        ModelAndView mv = new ModelAndView();
        // 存入错误的提示信息
        mv.addObject("errorMsg", e.getMessage());
        // 跳转的Jsp页面
        mv.setViewName("error");
        return mv;
    }
}
```
##### 3. 配置异常处理器
```xml
    <!--配置异常处理器-->
    <bean id="sysExceptionResolver" class="cn.mvc.exception.SysExceptionResolver"/>
```
##### 4.测试
```java
    @RequestMapping("/testException")
    public String testException() throws SysException{
        System.out.println("testException执行了...");

        try {
            // 模拟异常
            int a = 10/0;
        } catch (Exception e) {
            // 打印异常信息
            e.printStackTrace();
            // 抛出自定义异常信息
            throw new SysException("查询所有用户出现错误了...");
        }
        return "success";
    }
```

###  二、SpringMVC 中的拦截器
##### 1. 拦截器的概述

（1）SpringMVC框架中的拦截器用于对处理器进行预处理和后处理的技术。  
（2）可以定义拦截器链，连接器链就是将拦截器按着一定的顺序结成一条链，在访问被拦截的方法时，拦截器链中的拦截器会按着定义的顺序执行。   
（3）拦截器和过滤器的功能比较类似，有区别：
* 过滤器是Servlet规范的一部分，任何框架都可以使用过滤器技术。
* 拦截器是SpringMVC框架独有的。
* 过滤器配置了/*，可以拦截任何资源。
* 拦截器只会对控制器中的方法进行拦截。

（4）拦截器也是AOP思想的一种实现方式。  
（5） 想要自定义拦截器，需要实现HandlerInterceptor接口。
##### 2.自定义拦截器
（1）编写一个普通类实现 HandlerInterceptor 接口
```java
public class MyInterceptor1 implements HandlerInterceptor{

    /**
     * 前置处理：controller请求方法执行前拦截
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("拦截器执行前置处理...111");
        // request.getRequestDispatcher("/WEB-INF/pages/error.jsp").forward(request,response);
        return true;
    }

    /**
     * 后置处理：controller方法执行后，success.jsp执行之前处理
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("拦截器执行后置处理...111");
        // request.getRequestDispatcher("/WEB-INF/pages/error.jsp").forward(request,response);
    }

    /**
     * 最终处理：success.jsp页面执行后执行
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("拦截器执最终处理...111");
    }
}
```
（2）配置拦截器
```xml
    <!--配置拦截器-->
    <mvc:interceptors>
        <!--配置拦截器-->
        <mvc:interceptor>
            <!--要拦截的具体的方法-->
            <mvc:mapping path="/user/*"/>  <!-- 用于指定对拦截的 url -->
            <!--不要拦截的方法
            <mvc:exclude-mapping path=""/> <!-- 用于指定排除的 url-->
            
            <!--配置拦截器对象-->
            <bean class="cn.mvc.controller.cn.itcast.interceptor.MyInterceptor1" />
        </mvc:interceptor>
    </mvc:interceptors>
```
（3）运行结果
```shell script
拦截器执行前置处理...111
Controller中请求方法执行了...
拦截器执行后置处理...111
success.jsp执行了...
拦截器执行最终处理...111
```
##### 3.HandlerInterceptor接口中的方法
（1）preHandle方法是controller方法执行前拦截的方法  
* 可以使用request或者response跳转到指定的页面。  
* return true放行，执行下一个拦截器，如果没有拦截器，执行controller中的方法。  
* return false不放行，不会执行controller中的方法。  
（2） postHandle是controller方法执行后执行的方法，在JSP视图执行前  
* 可以使用request或者response跳转到指定的页面。  
* 如果指定了跳转的页面，那么controller方法跳转的页面将不会显示。  
（3） postHandle方法是在JSP执行后执行  
* request或者response不能再跳转页面了。

##### 4. 配置多个拦截器
（1）再编写一个拦截器的类
```java
public class MyInterceptor2 implements HandlerInterceptor{

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("拦截器执行前置处理...222");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("拦截器执行后置处理...222");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("拦截器执行最终处理...222");
    }
}
```
（2）配置2个拦截器
```xml
    <!--配置拦截器-->
    <mvc:interceptors>
        <!--配置拦截器-->
        <mvc:interceptor>
            <!--要拦截的具体的方法-->
            <mvc:mapping path="/user/*"/>
            <!--不要拦截的方法
            <mvc:exclude-mapping path=""/>
            -->
            <!--配置拦截器对象-->
            <bean class="cn.mvc.controller.cn.itcast.interceptor.MyInterceptor1" />
        </mvc:interceptor>

        <!--配置第二个拦截器-->
        <mvc:interceptor>
            <!--要拦截的具体的方法-->
            <mvc:mapping path="/**"/>
            <!--不要拦截的方法
            <mvc:exclude-mapping path=""/>
            -->
            <!--配置拦截器对象-->
            <bean class="cn.mvc.controller.cn.itcast.interceptor.MyInterceptor2" />
        </mvc:interceptor>
    </mvc:interceptors>
```
（3）运行结果
```shell script
拦截器执行前置处理...111
拦截器执行前置处理...222
Controller中请求方法执行了...
拦截器执行后置处理...222
拦截器执行后置处理...111
success.jsp执行了...
拦截器执行最终处理...222
拦截器执行最终处理...111
```

### 参考资料
[详述 Spring MVC 框架中拦截器 Interceptor 的使用方法](https://blog.csdn.net/qq_35246620/article/details/68487904)