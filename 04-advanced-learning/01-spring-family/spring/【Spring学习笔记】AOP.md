### 一、何为AOP？

AOP(Aspect-Oriented Programming，面向切面编程)是对传统传统 OOP(Object-Oriented Programming，面向对象编程）的补充，属于一种横向扩展。其将与核心业务无关的代码，如**日志记录、性能监控、事务处理**等从业务逻辑代码中抽离出来，进行横向排列，从而实现低耦合，提高开发效率。

<img src="https://img-blog.csdnimg.cn/20200914213530314.png" width="60%" alt="3"/>

### 二、AOP相关术语：

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

### 基于接口的动态代理
提供者：JDK官方的Proxy类。
要求：被代理类最少实现一个接口。

### 基于子类的动态代理
提供者：第三方的CGLib，如果报asmxxxx异常，需要导入asm.jar。
要求：被代理类不能用final修饰的类（最终类）。

### 基于XML的AOP配置

### 基于注解的AOP配置

### AOP细节
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






##### 参考资料：
[https://blog.csdn.net/qukaiwei/article/details/50367761](https://blog.csdn.net/qukaiwei/article/details/50367761)  
[https://juejin.im/post/6844903555531276296](https://juejin.im/post/6844903555531276296)
[http://c.biancheng.net/view/4268.html](http://c.biancheng.net/view/4268.html)
[https://juejin.im/post/6844903478582575111](https://juejin.im/post/6844903478582575111)