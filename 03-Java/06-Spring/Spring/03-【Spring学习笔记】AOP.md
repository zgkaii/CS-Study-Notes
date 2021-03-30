- [一、何为AOP？](#一何为aop)
- [二、AOP相关术语](#二aop相关术语)
- [三、AOP流行框架比较](#三aop流行框架比较)
- [四、动态代理](#四动态代理)
    - [1.创建接口UserDao](#1创建接口userdao)
    - [2.创建实现类 UserDaoImpl](#2创建实现类-userdaoimpl)
    - [3.创建切面类](#3创建切面类)
    - [4.创建代理类 JdkBeanFactory](#4创建代理类-jdkbeanfactory)
    - [5.创建代理类 CglibBeanFactory](#5创建代理类-cglibbeanfactory)
- [五、使用AspectJ开发AOP](#五使用aspectj开发aop)
    - [1.创建接口UserDao](#1创建接口userdao-1)
    - [2.创建实现类 UserDaoImpl](#2创建实现类-userdaoimpl-1)
    - [3. 创建切面类 XmlAspect](#3-创建切面类-xmlaspect)
    - [4.创建基于Xml的配置文件 xmlBean.xml](#4创建基于xml的配置文件-xmlbeanxml)
    - [5. 创建测试方法 testXml()](#5-创建测试方法-testxml)
    - [6.创建切面类 XmlAspect](#6创建切面类-xmlaspect)
    - [7.创建基于注解的配置文件 AnnotationBean.xml](#7创建基于注解的配置文件-annotationbeanxml)
    - [8.创建测试方法 testAnnotation()](#8创建测试方法-testannotation)
- [六、AOP相关细节](#六aop相关细节)
    - [1.Spring通知类型](#1spring通知类型)
    - [2.切入点表达式](#2切入点表达式)
    - [3. Annotation 注解](#3-annotation-注解)
- [参考资料：](#参考资料)
### 一、何为AOP？

AOP(Aspect-Oriented Programming，面向切面编程)是对传统传统 OOP(Object-Oriented Programming，面向对象编程）的补充，属于一种横向扩展。其将与核心业务无关的代码，如**日志记录、性能监控、事务处理**等从业务逻辑代码中抽离出来，进行横向排列，从而实现低耦合，提高开发效率。

<img src="https://img-blog.csdnimg.cn/20200914213530314.png" width="60%" alt="3"/>

### 二、AOP相关术语

|名称|说明|
|---|---|
|Advice（通知）|例如日志记录、性能监控、事务处理等增强的功能。|
|Joinpoint（连接点）|具体执行增强的地方。例如方法调用前、调用后和方法捕获异常后。|
|Pointcut（切入点）|匹配连接点的正则表达式，是实现增强的具体连接点的集合。|
|Aspect（切面）|切入点和通知的结合。|
|Introduction(引介)|一种特殊的通知，可以在运行期为类动态地添加一些方法或 Field。|
|Proxy（代理）|	一个类被 AOP 织入增强后，就产生一个结果代理类。|
|Target（目标）|指代理的目标对象。|
|Weaving（织入）|把切面应用到目标对象，并创建代理对象的过程。|

### 三、AOP流行框架比较
>当下最流行的AOP框架为Spring AOP 及 AspectJ

|比较|Spring AOP|AspectJ|
|---|---|---|
|能力和目标|旨在通过Spring IoC提供一个简单的AOP实现。<br>不是一个完整的AOP解决方案，只能用于被Spring容器管理的bean。|完整的AOP解决方案，比Spirng AOP复杂。<br>能够被应用于所有的领域对象。|
|织入时机|只能在运行时织入|可以在编译时，加载期，编译后（jar包和字节码文件）。|
|依赖|使用JDK动态代理或者VGLIB动态代理。|不依赖任何运行时环境，只需要引入编译器AspectJ compiler(ajc)完成织入。|
|连接点|使用代理模式，所以不能作用于final类、static方法、final方法。<br>只支持方法执行连接点。|没有限制。方法调用，方法执行，构造器调用，构造器执行，静态初始化执行，对象初始化，成员引用，成员赋值，处理器执行，通知执行。|
|复杂度|不需要额外的编译器，只能和Spirng管理的bean一起工作 AOP|需要引入AJC编译器并重新打包|
|性能|基于代理的框架，运行时会有目标类的代理对象生成。|在应用执行前织入切面到代码中，没有额外的运行时开销。|

### 四、动态代理
**基于接口的动态代理**  
提供者：JDK官方的ava.lang.reflect.Proxy 类。  
要求： 被代理类至少实现一个接口。  

**基于子类的动态代理**  
提供者：第三方的CGLib， CGLIB 要依赖于 ASM 的包。  
要求：被代理类不能用final修饰的类（最终类）。  

下面以一个例子来了解两者的差别：   

##### 1.创建接口UserDao
```java
public interface UserDao {

    public void add();// 添加

    public void delete();// 删除
}
```
##### 2.创建实现类 UserDaoImpl
```java
public class UserDaoImpl implements UserDao {
    @Override
    public void add() {
        System.out.println("添加用户");
    }

    @Override
    public void delete() {
        System.out.println("删除用户");
    }
}
```
##### 3.创建切面类
```java
public class MyAspect {
    public void myBefore() {
        System.out.println("方法执行之前...");
    }
    public void myAfter() {
        System.out.println("方法执行之后...");
    }
}
```
##### 4.创建代理类 JdkBeanFactory
[JDK动态代理实现原理](https://blog.csdn.net/yhl_jxy/article/details/80586785)
```java
public class JdkBeanFactory {
    public static UserDao getBean() {
        // 准备目标类
        final UserDaoImpl userDao = new UserDaoImpl();
        // 创建切面类实例
        final MyAspect myAspect = new MyAspect();

        // 使用代理类，进行增强
        return (UserDao) Proxy.newProxyInstance(JdkBeanFactory.class.getClassLoader(),
                new Class[]{UserDao.class}, new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        myAspect.myBefore(); // 前增强
                        Object obj = method.invoke(userDao, args);
                        myAspect.myAfter(); // 后增强
                        return obj;
                    }
                });
    }
}
```
创建测试类 JDKProxyTests
```java
class JdkProxyTests {

    @Test
    public void test() {
        // 从工厂获得指定的内容（相当于spring获得，但此内容时代理对象）
        UserDao userDao = JdkBeanFactory.getBean();
        // 执行方法
        userDao.add();
        userDao.delete();
    }
}
```

执行结果：
```shell script
方法执行之前...
添加用户
方法执行之后...
方法执行之前...
删除用户
方法执行之后...
```
##### 5.创建代理类 CglibBeanFactory
[CGLIB动态代理实现原理](https://blog.csdn.net/yhl_jxy/article/details/80633194)
```java
public class CglibBeanFactory {
    public static UserDao getBean() {
        // 准备目标类
        final UserDaoImpl userDao = new UserDaoImpl();
        // 创建切面类实例
        final MyAspect myAspect = new MyAspect();
        // 生成代理类，CGLIB在运行时，生成指定对象的子类，增强
        Enhancer enhancer = new Enhancer();
        // 确定需要增强的类
        enhancer.setSuperclass(userDao.getClass());
        // 添加回调函数
        enhancer.setCallback(new MethodInterceptor() {
            // intercept 相当于 jdk invoke，前三个参数与 jdk invoke—致
            @Override
            public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                myAspect.myBefore();// 前增强
                Object obj = method.invoke(userDao, objects);// 目标方法执行
                myAspect.myAfter();// 后增强
                return obj;
            }
        });
        // 创建代理类
        UserDao userDaoProxy = (UserDao) enhancer.create();
        return userDaoProxy;
    }
}
```
创建测试类 CglibProxyTests
```java
public class CglibProxyTests {

    @Test
    public void test() {
        // 从工厂获得指定的内容（相当于spring获得，但此内容时代理对象）
        UserDao userDao = CglibBeanFactory.getBean();
        // 执行方法
        userDao.add();
        userDao.delete();
    }
}
```
测试结果：
```shell script
方法执行之前...
添加用户
方法执行之后...
方法执行之前...
删除用户
方法执行之后...
```
### 五、使用AspectJ开发AOP
##### 1.创建接口UserDao
```java
public interface UserDao {

    public void add();// 添加

    public void delete();// 删除
}
```
##### 2.创建实现类 UserDaoImpl
```java
public class UserDaoImpl implements UserDao {
    @Override
    public void add() {
        System.out.println("添加用户");
    }

    @Override
    public void delete() {
        System.out.println("删除用户");
    }
}
```
##### 3. 创建切面类 XmlAspect
```java
public class XmlAspect {
    // 前置通知
    public void myBefore(JoinPoint joinPoint) {
        System.out.print("前置通知，目标："+joinPoint.getTarget()+" ");
        System.out.print("方法名称:"+joinPoint.getSignature().getName()+"\n");
    }

    // 后置通知
    public void myAfterReturning(JoinPoint joinPoint) {
        System.out.print("后置通知，目标："+joinPoint.getTarget()+" ");
        System.out.print("方法名称:"+joinPoint.getSignature().getName());
    }

    // 环绕通知
    public Object myAround(ProceedingJoinPoint proceedingJoinPoint)
            throws Throwable {
        System.out.println("环绕开始"); // 开始
        Object obj = proceedingJoinPoint.proceed(); // 执行当前目标方法
        System.out.println("环绕结束"); // 结束
        return obj;
    }

    // 异常通知
    public void myAfterThrowing(JoinPoint joinPoint, Throwable e) {
        System.out.println("异常通知" + "出错了" + e.getMessage());
    }

    // 最终通知
    public void myAfter() {
        System.out.println("最终通知");
    }
}
```
##### 4.创建基于Xml的配置文件 xmlBean.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--1.配置通知类-->
    <bean id="myAspect" class="com.aop.aspectj.xml.XmlAspect"/>
    <!--2.配置目标类 -->
    <bean id="userDao" class="com.aop.aspectj.dao.impl.UserDaoImpl"/>

    <!--3.声明aop配置 -->
    <aop:config>
        <!--4.配置切面 id：给切面提供一个唯一标识;ref：引用配置好的通知类 bean 的 id。-->
        <aop:aspect id="xmlAspect" ref="myAspect">
            <!-- 5.配置切入点，通知最后增强哪些方法
                expression：用于定义切入点表达式。
                id：用于给切入点表达式提供一个唯一标识-->
            <aop:pointcut id="myPointCut" expression="execution ( * com.aop.aspectj.dao.*.* (..))"/>
            <!-- 6.使用 aop:xxx 配置对应的通知类型
                method：指定通知中方法的名称。
                pointct：定义切入点表达式
                pointcut-ref：指定切入点表达式的引用-->
            <!--6.1 前置通知，关联通知 Advice和切入点PointCut -->
            <aop:before method="myBefore" pointcut-ref="myPointCut"/>
            <!--6.2 后置通知，在方法返回之后执行，就可以获得返回值returning 属性 -->
            <aop:after-returning method="myAfterReturning"
                                 pointcut-ref="myPointCut" returning="returnVal"/>
            <!--6.3 环绕通知 -->
            <aop:around method="myAround" pointcut-ref="myPointCut"/>
            <!--6.4 异常通知：用于处理程序发生异常，可以接收当前方法产生的异常
                *注意：如果程序没有异常，则不会执行增强
                *throwing属性：用于设置通知第二个参数的名称，类型Throwable -->
            <aop:after-throwing method="myAfterThrowing"
                                pointcut-ref="myPointCut" throwing="e"/>
            <!--6.5 最终通知：无论程序发生任何事情，都将执行 -->
            <aop:after method="myAfter" pointcut-ref="myPointCut"/>
        </aop:aspect>
    </aop:config>
</beans>
```
##### 5. 创建测试方法 testXml()
```java
    @Test
    public void testXml() {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext(
                "XmlBean.xml");
        // 从spring容器获取实例
        UserDao userDao = (UserDao) applicationContext
                .getBean("userDao");
        // 执行方法
        userDao.add();
    }
```
测试结果：
```shell script
前置通知，目标：com.aop.aspectj.dao.impl.UserDaoImpl@5b78fdb1 方法名称:add
环绕开始
添加用户
最终通知
环绕结束
后置通知，目标：com.aop.aspectj.dao.impl.UserDaoImpl@5b78fdb1 方法名称:add
```
测试异常通知：  
在UserDaoImpl中add()添加 `int i= 1/0 ;`
```shell script
前置通知，目标：com.aop.aspectj.dao.impl.UserDaoImpl@5b78fdb1 方法名称:add
环绕开始
最终通知
异常通知出错了/ by zero
```
##### 6.创建切面类 XmlAspect
```java
@Aspect
@Component
public class AnnotationAspect {
    // 用于取代：<aop:pointcut expression="execution(*com.aop.aspectj.dao..*.*(..))" id="myPointCut"/>
    // 要求：方法必须是private，且没有值，名称可自定义，没有参数
    @Pointcut("execution(* com.aop.aspectj.dao.*.*(..))")
    private void myPointCut() {
    }
    // 前置通知
    @Before("myPointCut()")
    public void myBefore(JoinPoint joinPoint) {
        System.out.print("前置通知，目标："+joinPoint.getTarget()+" ");
        System.out.print("方法名称:"+joinPoint.getSignature().getName()+"\n");
    }
    // 后置通知
    @AfterReturning(value = "myPointCut()")
    public void myAfterReturning(JoinPoint joinPoint) {
        System.out.print("后置通知，目标："+joinPoint.getTarget()+" ");
        System.out.print("方法名称:"+joinPoint.getSignature().getName());
    }
    // 环绕通知
    @Around("myPointCut()")
    public Object myAround(ProceedingJoinPoint proceedingJoinPoint)
            throws Throwable {
        System.out.println("环绕开始"); // 开始
        Object obj = proceedingJoinPoint.proceed(); // 执行当前目标方法
        System.out.println("环绕结束"); // 结束
        return obj;
    }
    // 异常通知
    @AfterThrowing(value = "myPointCut()", throwing = "e")
    public void myAfterThrowing(JoinPoint joinPoint, Throwable e) {
        System.out.println("异常通知" + "出错了" + e.getMessage());
    }
    // 最终通知
    @After("myPointCut()")
    public void myAfter() {
        System.out.println("最终通知");
    }
}
```
##### 7.创建基于注解的配置文件 AnnotationBean.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
    <!--扫描含com.aop-->
    <context:component-scan base-package="com.aop"/>
    <!-- 使切面开启自动代理 -->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```
##### 8.创建测试方法 testAnnotation()
```java
    @Test
    public void testAnnotation() {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext(
                "AnnotationBean.xml");
        // 从spring容器获取实例
        UserDao userDao = (UserDao) applicationContext
                .getBean("userDao");
        // 执行方法
        userDao.delete();
    }
```
测试结果：
```shell script
环绕开始
前置通知，目标：com.aop.aspectj.dao.impl.UserDaoImpl@245a060f 方法名称:delete
删除用户
后置通知，目标：com.aop.aspectj.dao.impl.UserDaoImpl@245a060f 方法名称:delete
最终通知
环绕结束
```
测试异常通知：  
在UserDaoImpl中add()添加 `int i= 1/0 ;`
```shell script
环绕开始
前置通知，目标：com.aop.aspectj.dao.impl.UserDaoImpl@1ded7b14 方法名称:delete
异常通知出错了/ by zero
最终通知
```
### 六、AOP相关细节
##### 1.Spring通知类型

|名称|说明|
|---|---|
|org.springframework.aop.MethodBeforeAdvice（前置通知）|在方法之前自动执行的通知称为前置通知，可以应用于权限管理等功能。|
|org.springframework.aop.AfterReturningAdvice（后置通知）|在方法之后自动执行的通知称为后置通知，可以应用于关闭流、上传文件、删除临时文件等功能。|
|org.aopalliance.intercept.MethodInterceptor（环绕通知）|在方法前后自动执行的通知称为环绕通知，可以应用于日志、事务管理等功能。|
|org.springframework.aop.ThrowsAdvice（异常通知）|在方法抛出异常时自动执行的通知称为异常通知，可以应用于处理异常记录日志等功能。|
|org.springframework.aop.IntroductionInterceptor（引介通知）|在目标类中添加一些新的方法和属性，可以应用于修改旧版本程序（增强类）。|

##### 2.切入点表达式
2.1 语法格式
```shell
execution([权限修饰符] [返回值类型] [简单类名/全类名] [方法名]([参数列表]))
```
2.2	实例分析

|表达式|含义|
|---|---|
|`execution(* com.spring.iop.dao.*(..))`|dao接口中声明的所有方法。<br>第一个`*`代表任意修饰符及任意返回值。<br>第二个`*`代表任意方法。<br>`..`匹配任意数量、任意类型的参数。|
|`execution(public * dao.*(..))`|dao接口的所有公有方法|
|`execution(public double dao.*(..))`|dao接口中返回double类型数值的方法|
|`execution(public double dao.*(double, ..))`|第一个参数为double类型,<br>`..` 匹配任意数量、任意类型的参数。|
|`execution(public double dao.*(double, double))`|参数类型为double，double类型的方法|

>特别地，在AspectJ中，切入点表达式可以通过 “&&”、“||”、“!”等操作符结合起来。

|表达式|含义|
|---|---|
|`execution (* *.add(int,..))` | 任意类中第一个参数为int类型的add方法或sub方法。|

##### 3. Annotation 注解

|名称|说明|
|---|---|
|@Aspect 	|用于定义一个切面。|
|@Before |	用于定义前置通知，相当于 BeforeAdvice。|
|@AfterReturning |	用于定义后置通知，相当于 AfterReturningAdvice。|
|@Around 	|用于定义环绕通知，相当于MethodInterceptor。|
|@AfterThrowing 	|用于定义抛出通知，相当于ThrowAdvice。|
|@After |	用于定义最终final通知，不管是否异常，该通知都会执行。|
|@DeclareParents |	用于定义引介通知，相当于IntroductionInterceptor（不要求掌握）。|


### 参考资料：
[Spring AOP概念理解 ](https://blog.csdn.net/qukaiwei/article/details/50367761)    
[比较Spring AOP与AspectJ](https://juejin.im/post/6844903555531276296)  
[AOP（面向切面编程）](http://c.biancheng.net/view/4268.html)  
[Spring-aop 全面解析（从应用到原理）](https://juejin.im/post/6844903478582575111)  
[JDK动态代理实现原理](https://blog.csdn.net/yhl_jxy/article/details/80586785)  
[CGLIB动态代理实现原理](https://blog.csdn.net/yhl_jxy/article/details/80633194)  