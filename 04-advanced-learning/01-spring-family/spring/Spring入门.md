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
当某个 Java 实例需要另一个 Java 实例时，传统的方法是由调用者创建被调用者的实例（例如，使用 new 关键字获得被调用者实例），而使用 Spring 框架后，被调用者的实例不再由调用者创建，而是由 Spring 容器创建，这称为控制反转。

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

![容器高层视图](https://img-blog.csdnimg.cn/20200911130501741.png)

要使应用程序中的Spring容器成功启动，需要以下三方面条件：  
* Spring框架所需的包都放在类路径下。
* 应用程序为Spring提供了完备的Bean配置信息。
* Bean的实现类都已放在应用程序的类路径下。

Spring 启动时读取应用程序提供的Bean配置信息，并在Spring容器内部建立Bean定义注册表，然后根据注册表加载、实例化Bean,并装配好Bean之间的依赖关系，最后将这些准备就绪的Bean放到Bean缓存池中，以供外层的应用程序进行调用。。

Bean的配置信息即是Bean的元数据信息，它包含：

* Bean 的实现类。
* Bean 的属性信息,如数据源的连接数、用户名、密码等。
* Bean 的依赖关系,Spring根据依赖关系配置完成Bean之间的装配。
* Bean 的行为配置,如生命周期范围及生命周期各过程的回调函数等。

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

| 属性名称 | 描述|
| --- | --- |
|	id 	 |	给对象在容器中提供一个唯一标识，Spring容器对Bean的配置和管理都通过该属性完成	  |
|	name 	|	Spring 容器同样可以通过此属性对容器中的 Bean 进行配置和管理，name 属性中可以为 Bean 指定多个名称，每个名称之间用逗号或分号隔开	|
|	class 	|	指定类的全限定类名。用于反射创建对象。默认情况下调用无参构造函数。 |
|	scope  	|	用于设定 Bean 实例的作用域，其属性值有 singleton（单例）、prototype（原型）、request、session 和 global Session。其默认值是 singleton		|
|	constructor-arg 	|	<bean>元素的子元素，可以使用此元素传入构造参数进行实例化。该元素的 index 属性指定构造参数的序号（从 0 开始），type 属性指定构造参数的类型	|
|	property 	|	<bean>元素的子元素，用于调用 Bean 实例中的 Set 方法完成属性赋值，从而完成依赖注入。该元素的 name 属性指定 Bean 实例中的相应属性名			|
|	ref 	|	<property> 和 <constructor-arg> 等元素的子元索，该元素中的 bean 属性用于指定对 Bean 工厂中某个 Bean 实例的引用	 |
|	value 	|	<property> 和 <constractor-arg> 等元素的子元素，用于直接指定一个常量值	|
|	list 	|	用于封装 List 或数组类型的依赖注入	|
|	set 	|	用于封装 Set 类型属性的依赖注入		|
|	map 	|	用于封装 Map 类型属性的依赖注入		|
|	entry 	|	<map> 元素的子元素，用于设置一个键值对。其 key 属性指定字符串类型的键值，ref 或 value 子元素指定其值	|

##### Bean的作用域

|作用域 | 描述|
| --- | --- |
| singleton | 单例模式，使用 singleton 定义的 Bean 在 Spring 容器中只有一个实例，这也是 Bean 默认的作用域。 |
| prototype | 原型模式，每次通过 Spring 容器获取 prototype 定义的 Bean 时，容器都将创建一个新的 Bean 实例。 |
| request | 每次HTTP请求都会创建一个新的Bean，该作用域仅在当前 HTTP Request 内有效。|
| session | 同一个HTTP Session共享一个Bean，不同Session使用不同的Bean，该作用域仅在当前 HTTP Session 内有效。 |
| global-session | 在一个全局的 HTTP Session 中，容器会返回该 Bean 的同一个实例。该作用域仅在使用 portlet context 时有效。 |

（1）单例模式下:
```xml
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl" scope="singleton"></bean>
```
输出结果：
```shell
对象创建了
com.itheima.service.impl.AccountServiceImpl@61832929
com.itheima.service.impl.AccountServiceImpl@61832929
```
（2）原型模式下:
```xml
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl" scope="prototype"></bean>
```
输出结果：
```shell
对象创建了
com.itheima.service.impl.AccountServiceImpl@26be92ad
对象创建了
com.itheima.service.impl.AccountServiceImpl@4c70fda8
```

##### Bean的生命周期
[详细参考](http://c.biancheng.net/view/4261.html)
```java
public class AccountServiceImpl implements IAccountService {

    public AccountServiceImpl(){
        System.out.println("对象创建了");
    }

    public void  saveAccount(){
        System.out.println("saveAccount()执行");
    }

    public void  init(){
        System.out.println("对象初始化了");
    }
    public void  destroy(){
        System.out.println("对象销毁了");
    }
}
```
（1）单例模式：  
出生：当容器创建时对象出生  
活着：只要容器还在，对象一直活着    
死亡：容器销毁，对象消亡    
总结：单例对象的生命周期和容器相同  
```xml
    <bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl"
          scope="singleton" init-method="init" destroy-method="destroy"></bean>
```
执行结果：
```
对象创建了
对象初始化了
saveAccount()执行
对象销毁了
```
（2）原型模式：   
出生：当我们使用对象时spring框架为我们创建    
活着：对象只要是在使用过程中就一直活着    
死亡：当对象长时间不用，且没有别的对象引用时，由Java的垃圾回收器回收  
```xml
    <bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl"
          scope="prototype" init-method="init" destroy-method="destroy"></bean>
```
执行结果：
```
对象创建了
对象初始化了
saveAccount()执行
```
### Bean实例化3种方式

###### 构造器实例化
	在Spring的配置文件中使用bean标签，配以id和class属性之后，且没有其他属性和标签时。采用的就是默认构造函数创建bean对象，此时如果类中没有默认构造函数，则对象无法创建。
```xml
<bean id="accountService"class="com.kai.service.impl.AccountServiceImpl"/>
```
	
##### 静态工厂方式实例化
使用StaticFactory类中的静态方法createAccountService创建对象，并存入spring容器  
id属性：指定bean的id，用于从容器中获取  
class属性：指定静态工厂的全限定类名  
factory-method属性：指定生产对象的静态方法
```java
/**
 * 模拟一个工厂类（该类可能是存在于jar包中的，我们无法通过修改源码的方式来提供默认构造函数）
 */
public class StaticFactory {

    public static IAccountService createAccountService(){

        return new AccountServiceImpl();
    }
}
```
```xml
<bean id="accountService" class="com.kai.factory.StaticFactory" factory-method="createAccountService"></bean>
```
##### 实例工厂方式实例化
先把工厂的创建交给spring来管理。然后在使用工厂的bean来调用里面的方法。  
factory-bean属性：用于指定实例工厂bean的id。  
factory-method属性：用于指定实例工厂中创建对象的方法。
```java
/**
 * 模拟一个实例工厂，创建业务层实现类
 * 此工厂创建对象，必须现有工厂实例对象，再调用方法
 */
public class InstanceFactory {

    public IAccountService getAccountService(){
        return new AccountServiceImpl();
    }
}
```

```xml
<bean id="instanceFactory" class="com.kai.factory.InstanceFactory"></bean>
<bean id="accountService" factory-bean="instanceFactory" factory-method="createAccountService"></bean>
```

### 什么是依赖注入？
依赖注入（Dependency Injection，DI）和控制反转含义相同，它们是从两个角度描述的同一个概念。  

之前我们已经知道，将被调用者的实例交由 Spring容器创建，即是控制反转；而Spring容器在创建被调用者的实例时，会自动将调用者需要的对象实例注入给调用者，这样，调用者通过 Spring容器获得被调用者实例，这称为依赖注入。

依赖注入主要有两种实现方式，分别是属性 set方法注入 和 构造方法注入。

##### set方法注入
```java
public class AccountServiceImpl implements IAccountService {

    //如果是经常变化的数据，并不适用于注入的方式
    private String name;
    private Integer age;
    private Date birthday;

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public void saveAccount() {
        System.out.println(name + "," + age + "," + birthday);
    }
}
```
涉及的标签：property  
出现的位置：bean标签的内部  
标签的属性  
* name：用于指定注入时所调用的set方法名称
* value：用于提供基本类型和String类型的数据
* ref：用于指定其他的bean类型数据。它指的就是在spring的Ioc核心容器中出现过的bean对象

优势：  
	创建对象时没有明确的限制，可以直接使用默认构造函数  
弊端：  
	如果有某个成员必须有值，则获取对象是有可能set方法没有执行。
```xml
    <bean id="accountService" class="com.kai.service.impl.AccountServiceImpl">
        <property name="name" value="zhangsan" ></property>
        <property name="age" value="22"></property>
        <property name="birthday" ref="now"></property>
    </bean>
	
	<!-- 配置一个日期对象 -->
    <bean id="now" class="java.util.Date"></bean>
```

##### 构造方法注入

```java
public class AccountServiceImpl implements IAccountService {

    //如果是经常变化的数据，并不适用于注入的方式
    private String name;
    private Integer age;
    private Date birthday;

    public AccountServiceImpl(String name,Integer age,Date birthday){
        this.name = name;
        this.age = age;
        this.birthday = birthday;
    }

    public void  saveAccount(){
        System.out.println(name+","+age+","+birthday);
    }
}
```
使用的标签:constructor-arg  
标签出现的位置：bean标签的内部  
标签中的属性:  
* type：用于指定要注入的数据的数据类型，该数据类型也是构造函数中某个或某些参数的类型
* index：用于指定要注入的数据给构造函数中指定索引位置的参数赋值。索引的位置是从0开始
* name：用于指定给构造函数中指定名称的参数赋值  
=============以上三个用于指定给构造函数中哪个参数赋值======================
* value：用于提供基本类型和String类型的数据  
* ref：用于指定其他的bean类型数据。它指的就是在spring的Ioc核心容器中出现过的bean对象  

优势：  
    在获取bean对象时，注入数据是必须的操作，否则对象无法创建成功。
弊端：  
    改变了bean对象的实例化方式，使我们在创建对象时，如果用不到这些数据，也必须提供。

```xml
    <bean id="accountService" class="com.kai.service.impl.AccountServiceImpl">
        <constructor-arg name="name" value="泰斯特"></constructor-arg>
        <constructor-arg name="age" value="18"></constructor-arg>
        <constructor-arg name="birthday" ref="now"></constructor-arg>
    </bean>
	
	<!-- 配置一个日期对象 -->
    <bean id="now" class="java.util.Date"></bean>
```

##### 复杂类型的注入/集合类型的注入
```java
@Data
public class AccountServiceImpl3 implements IAccountService {

    private String[] myStrs;
    private List<String> myList;
    private Set<String> mySet;
    private Map<String, String> myMap;
    private Properties myProps;

    public void saveAccount() {
        System.out.println(Arrays.toString(myStrs));
        System.out.println(myList);
        System.out.println(mySet);
        System.out.println(myMap);
        System.out.println(myProps);
    }
}
```
用于给List结构集合注入的标签：  
list array set
用于个Map结构集合注入的标签:  
map  props
```xml
    <bean id="accountService" class="com.kai.service.impl.AccountServiceImpl">
        <property name="myStrs">
            <set>
                <value>AAA</value>
                <value>BBB</value>
                <value>CCC</value>
            </set>
        </property>

        <property name="myList">
            <array>
                <value>AAA</value>
                <value>BBB</value>
                <value>CCC</value>
            </array>
        </property>

        <property name="mySet">
            <list>
                <value>AAA</value>
                <value>BBB</value>
                <value>CCC</value>
            </list>
        </property>

        <property name="myMap">
            <props>
                <prop key="testC">ccc</prop>
                <prop key="testD">ddd</prop>
            </props>
        </property>

        <property name="myProps">
            <map>
                <entry key="testA" value="aaa"></entry>
                <entry key="testB">
                    <value>BBB</value>
                </entry>
            </map>
        </property>
    </bean>
```
### 注解装配Bean

常用注解：

|基于注解配置 | 描述 |
| --- | --- | 
| @Component | 把资源给spring容器来管理。相当于在xml中配置一个bean。<br>属性：value（用于指定bean的id），默认值是首字母改小写的当前类名。|
| @Controller | 通常作用在表现层，用于将控制层的类标识为 Spring中的Bean。|
| @Service | 通常作用在业务层（Service 层），用于将业务层的类标识为Spring中的Bean。|
| @Repository | 用于将数据访问层（DAO层）的类标识为Spring中的Bean。  | 
| @Autowired | 自动按照类型注入。用于对Bean的属性变量、属性的Set方法及构造函数进行标注，配合对应的注解处理器完成Bean的自动配置工作。 | 
| @Resource | 其作用与Autowired一样。其区别在于@Autowired默认按照Bean类型装配，而@Resource默认按照Bean实例名称进行装配。 |
| @Qualifier | 与 @Autowired 注解配合使用，会将默认的按 Bean 类型装配修改为按 Bean 的实例名称装配，Bean的实例名称由 @Qualifier 注解的参数指定。 |


Spring管理Bean方式比较：

| | 基于XML配置| 基于注解配置 |
| --- | --- | --- |
| bean定义 | `<bean id="" class=""scope="">` | @Component<br>衍生类 @Controller、@Service和@Repository |
| bean名称 | 通过id或者name指定 | @Component("person")  |
| bean注入 | `<property>`或者p命名空间 | @Autowired按照类型注入<br>@Qualifier按名称注入 <br> @Resource  |
| 生命周期、作用范围 | init-method<br> destroy-method <br> socpe属性| @PostConstruct初始化<br>@PreDestroy销毁<br>@Scope设置作用范围 |
| 适合场景 | Bean来自第三方 | Bean的实现类由用户自己开发 |

注解的优势:  
配置简单，维护方便（我们找到类，就相当于找到了对应的配置）。  
XML的优势：   
修改时，不用改源码。不涉及重新编译和部署。



### 自动装配Bean



