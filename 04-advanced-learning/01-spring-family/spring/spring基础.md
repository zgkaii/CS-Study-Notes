### Spring Bean到底是什么？
Bean是Spring中一个重要的概念。Spring官方文档对bean的解释是：  
```
In Spring, the objects that form the backbone of your application and that are managed by the Spring IoC container are called beans. 
A bean is an object that is instantiated, assembled, and managed by a Spring IoC container.Otherwise, a bean is simply one of many 
objects in your application. Beans, and the dependencies among them, are reflected in the configuration metadata used by a container.
```
也就是说：
```
在Spring中，构成应用程序主干并由Spring IoC容器管理的对象统称为beans。 bean是一个由Spring IoC容器实例化、组装和管理的对象。
此外,bean只是应用程序中许多对象中的一个。Beans以及他们之间的依赖关系是通过容器配置元数据反映出来。
```
所以说：
* bean是一个对象
* bean由IoC容器管理
* 实际应用程序中，有很多bean

### IoC(Inversion Of Control)容器又是什么？

IoC（控制反转）思想：  
在程序开发中，实例的创建不再由调用者管理，而是由Spring容器创建。Spring容器会负责对象的实例化、定位、配置及建立这些对象间的依赖，控制权由程序员转移到了Spring容器中。

##### Spring主要提供两种IoC容器：  

* Spring BeanFactory
	- 提供了能够管理任何类型对象的高级配置机制。由org.springframework.beans.factory.BeanFactory接口来定义。
	
* Spring ApplicationContext
	- ApplicationContext是BeanFactory的子接口，提供了BeanFactory的所有功能，更容易集成Spring的AOP功能、资源访问、事件传播和特定的上下文应用层。

##### ApplicationContext接口常用实现类：
* ClassPathXmlApplicationContext:  
	从类的根路径下加载配置文件。（推荐使用）
* FileSystemXmlApplicationContext:   
	从磁盘路径上加载配置文件，配置文件可以在磁盘的任意位置。
* AnnotationConfigApplicationContext:   
	当使用注解配置容器对象时，需要使用此类来创建spring容器。（用来读取注解）

![](https://img-blog.csdnimg.cn/20200910173222719.png)

>通常不建议使用BeanFactory。

### Bean与Spring容器之间到底什么关系呢？
下图完美说明了上述问题：  
![容器高层视图](https://img-blog.csdnimg.cn/20200910151141825.png)

Bean是用容器提供的配置元数据创建的,这些配置元数据定义了Bean的实现及依赖关系，Spring容器根据各种形式的Bean配置信息在容器内部建立Bean定义注册表，    
然后根据注册表加载、实例化Bean，并建立Bean和Bean的依赖关系，最后将这些准备就绪的Bean放到Bean缓存池中，以供外层的应用程序进行调用。

结合下面Spring Demo，可以更好理解:  
1）创建UserDao与AccountDao接口：
```java
package com.kai.demo.dao;

public interface UserDao {
    public void save();
}
```
```java
package com.kai.demo.dao;

public interface AccountDao {
    public void add();
}
```
2）创建接口实现类
```java
package com.kai.demo.dao.impl;

import com.kai.demo.dao.UserDao;

public class UserDaoImpl implements UserDao {
    @Override
    public void save() {
        System.out.println("执行了save()");
    }
}
```
```java
package com.kai.demo.dao.impl;

import com.kai.demo.dao.AccountDao;

public class AccountDaoImpl implements AccountDao {
    @Override
    public void add() {
        System.out.println("执行了add()");
    }
}
```
3）创建 Spring 的核心配置文件 applicationContext.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.2.xsd">
    <!-- 由 Spring容器创建该类的实例对象 -->
    <bean id="UserDao" class="com.kai.demo.dao.impl.UserDaoImpl" />
    <bean id="AccountDao" class="com.kai.demo.dao.impl.AccountDaoImpl" />
</beans>
```
4）编写测试类
```java
package com.kai.demo;

import com.kai.demo.dao.AccountDao;
import com.kai.demo.dao.UserDao;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

@SpringBootTest
class SpringDemoApplicationTests {

    @Test
    void contextLoads() {
        // 定义Spring配置文件的路径
        String xmlPath = "applicationContext.xml";
        // 初始化Spring容器，加载配置文件
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext(
                xmlPath);
        
		// 通过容器获取UserDao与AccountDao实例
        UserDao userDao = (UserDao) applicationContext
                .getBean("UserDao");
        AccountDao accountDao = (AccountDao) applicationContext
                .getBean("AccountDao");
        
		// 调用UserDao的save()方法
        userDao.save();
        // 调用AccountDao的add()方法
        accountDao.add();
    }
}
```
5）测试结果
```
执行了save()
执行了add()
```
### Spring配置文件怎么书写呢？

通常情况下，Spring会以XML文件格式作为Spring的配置文件，这种配置方式通过XML文件注册并管理 Bean 之间的依赖关系。  

XML格式配置文件的根元素是<beans>，该元素包含了多个<bean>子元素，每一个<bean>子元素定义了一个Bean，并描述了该Bean如何被装配到Spring容器中 。  
例如上诉代码：  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	   xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans-3.2.xsd">
    <!-- 由 Spring容器创建该类的实例对象 -->
    <bean id="UserDao" class="com.kai.demo.dao.impl.UserDaoImpl" />
    <bean id="AccountDao" class="com.kai.demo.dao.impl.AccountDaoImpl" />
</beans>
```
<bean>元素中包含很多属性，其常用属性：  

|属性名称 | 描述|
| --- | --- |
|	id 	    |	给对象在容器中提供一个唯一标识，Spring容器对Bean的配置和管理都通过该属性完成															    |
|	name 	|	Spring 容器同样可以通过此属性对容器中的 Bean 进行配置和管理，name 属性中可以为 Bean 指定多个名称，每个名称之间用逗号或分号隔开					|
|	class 	|	指定类的全限定类名。用于反射创建对象。默认情况下调用无参构造函数。															            |
|	scope  	|	用于设定 Bean 实例的作用域，其属性值有 singleton（单例）、prototype（原型）、request、session 和 global Session。其默认值是 singleton		|
|	constructor-arg 	|	<bean>元素的子元素，可以使用此元素传入构造参数进行实例化。该元素的 index 属性指定构造参数的序号（从 0 开始），type 属性指定构造参数的类型	|
|	property 	|	<bean>元素的子元素，用于调用 Bean 实例中的 Set 方法完成属性赋值，从而完成依赖注入。该元素的 name 属性指定 Bean 实例中的相应属性名			|
|	ref 	|	<property> 和 <constructor-arg> 等元素的子元索，该元素中的 bean 属性用于指定对 Bean 工厂中某个 Bean 实例的引用	                        |   
|	value 	|	<property> 和 <constractor-arg> 等元素的子元素，用于直接指定一个常量值						|
|	list 	|	用于封装 List 或数组类型的依赖注入														|
|	set 	|	用于封装 Set 类型属性的依赖注入															|
|	map 	|	用于封装 Map 类型属性的依赖注入															|
|	entry 	|	<map> 元素的子元素，用于设置一个键值对。其 key 属性指定字符串类型的键值，ref 或 value 子元素指定其值	|






