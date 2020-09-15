### 一、什么是事务（Transaction）
##### 1.概念
**事务**就是用户定义的一系列数据库操作，这些操作可以视为一个完成的逻辑处理工作单元（unit），要么全部执行，要么全部不执行，是不可分割的工作单元。

##### 2.事务的关键属性(ACID)
* 原子性(atomicity)：  
“原子”的本意是“不可再分”，事务的原子性表现为一个事务中涉及到的多个操作在逻辑上缺一不可。事务的原子性要求事务中的所有操作要么都执行，要么都不执行。
* 一致性(Consistent)：  
“一致”指的是数据的一致，具体是指：所有数据都处于满足业务规则的一致性状态。一致性原则要求：一个事务中不管涉及到多少个操作，都必须保证事务执行之前数据是正确的，事务执行之后数据仍然是正确的。如果一个事务在执行的过程中，其中某一个或某几个操作失败了，则必须将其他所有操作撤销，将数据恢复到事务执行之前的状态，这就是回滚。
* 隔离性(isolation)：  
在应用程序实际运行过程中，事务往往是并发执行的，所以很有可能有许多事务同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏。隔离性原则要求多个事务在并发执行过程中不会互相干扰。
* 持久性(durability)：  
持久性原则要求事务执行完成后，对数据的修改永久的保存下来，不会因各种系统错误或其他意外情况而受到影响。通常情况下，事务对数据的修改应该被写入到持久化存储器中。

##### 3.举例
在执行SQL语句的时候，某些业务要求，一系列操作必须全部执行，而不能仅执行一部分。例如，一个转账操作：
```shell script
-- 从id=1的账户给id=2的账户转账100元
-- 第一步：将id=1的A账户余额(500)减去100
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- 第二步：将id=2的B账户余额(500)加上100
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
```
那么从A账户到B账户需要6个操作：
1）从A账户中把余额读出来（500）。
2）对A账户做减法操作（500-100）。
3）把结果写回A账户中（400）。
4）从B账户中把余额读出来（500）。
5）对B账户做加法操作（500+100）。
6）把结果写回B账户中（600）。

原子性：  
保证1-6所有过程要么都执行，要么都不执行。一旦在执行某一步骤的过程中发生问题，就需要执行回滚操作。假如执行到第五步的时候，B账户突然不可用（比如被注销），那么之前的所有操作都应该回滚到执行事务之前的状态。

一致性：  
在转账之前，A和B的账户中共有500+500=1000元钱。在转账之后，A和B的账户中共有400+600=1000元。也就是说，数据的状态在执行该事务操作之后从一个状态改变到了另外一个状态。同时一致性还能保证账户余额不会变成负数等。

隔离性：  
在A向B转账的整个过程中，只要事务还没有提交（commit），查询A账户和B账户的时候，两个账户里面的钱的数量都不会有变化。
如果在A给B转账的同时，有另外一个事务执行了C给B转账的操作，那么当两个事务都结束的时候，B账户里面的钱应该是A转给B的钱加上C转给B的钱再加上自己原有的钱。

持久性：  
一旦转账成功（事务提交），两个账户的里面的钱就会真的发生变化（会把数据写入数据库做持久化保存）。

### 二、Spring事务管理
Spring事务管理主要分为编程式事务管理。
##### 1.编程式事务管理
使用原生的JDBC API进行事务管理 ：
1）获取数据库连接Connection对象  
2）取消事务的自动提交  
3）执行操作  
4）正常完成操作时手动提交事务  
5）执行失败时回滚事务  
6）关闭相关资源  
评价：   
	使用原生的JDBC API实现事务管理是所有事务管理方式的基石，同时也是最典型的编程式事务管理。编程式事务管理需要将事务管理代码嵌入到业务方法中来控制事务的提交和回滚。在使用编程的方式管理事务时，必须在每个事务操作中包含额外的事务管理代码。  
	相对于核心业务而言，事务管理的代码显然属于非核心业务，如果多个模块都使用同样模式的代码进行事务管理，显然会造成较大程度的代码冗余。
##### 2.声明式事务管理
大多数情况下声明式事务比编程式事务管理更好：它将事务管理代码从业务方法中分离出来，以声明的方式来实现事务管理。  
==事务管理代码的固定模式作为一种横切关注点，可以通过AOP方法模块化，进而借助Spring AOP框架实现声明式事务管理。==

Spring在不同的事务管理API之上定义了一个抽象层，通过配置的方式使其生效，从而让应用程序开发人员不必了解事务管理API的底层实现细节，就可以使用Spring的事务管理机制。  
Spring既支持编程式事务管理，也支持声明式的事务管理。

##### 3.Spring提供的事务管理器
Spring从不同的事务管理API中抽象出了一整套事务管理机制，让事务管理代码从特定的事务技术中独立出来。开发人员通过配置的方式进行事务管理，而不必了解其底层是如何实现的。  
Spring的核心事务管理接口是**PlatformTransactionManager**。它为事务管理封装了一组独立于技术的方法。无论使用Spring的哪种事务管理策略(编程式或声明式)，事务管理器都是必须的。
事务管理器可以以普通的bean的形式声明在Spring IOC容器中。

![](https://img-blog.csdnimg.cn/2020091515255066.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0tBSVpfTEVBUk4=,size_16,color_FFFFFF,t_70#pic_center)

另外的核心接口分别是：
**TransactionDefinition**  
TransactionDefinition 接口是事务定义（描述）的对象，它提供了事务相关信息获取的方法，其中包括五个操作，具体如下。  
* String getName()：获取事务对象名称。  
* int getIsolationLevel()：获取事务的隔离级别。  
* int getPropagationBehavior()：获取事务的传播行为。  
* int getTimeout()：获取事务的超时时间。  
* boolean isReadOnly()：获取事务是否只读。  

**TransactionStatus**  
TransactionStatus 接口是事务的状态，它描述了某一时间点上事务的状态信息。

|名称| 	说明|
|---|---|
|void flush() |	刷新事务|
|boolean hasSavepoint() |获取是否存在保存点|
|boolean isCompleted()| 获取事务是否完成|
|boolean isNewTransaction() |获取是否是新事务|
|boolean isRollbackOnly() |获取是否回滚|
|void setRollbackOnly() |设置事务回滚|

##### 4.事务管理器的主要实现
①DataSourceTransactionManager：在应用程序中只需要处理一个数据源，而且通过JDBC存取。     
②JtaTransactionManager：在JavaEE应用服务器上用JTA(Java Transaction API)进行事务管理。   
③HibernateTransactionManager：用Hibernate框架存取数据库。  

### 三、事务的传播行为
当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。

|传播行为	|含义|
|---|---|
|PROPAGATION_REQUIRED |	有事务运行，当前方法就在这个事务内运行；否则，就启动一个新的事务，并在自己的事务内运行。一般的默认选择。|
|PROPAGATION_SUPPORTS |	当前方法不需要事务上下文就可执行，但是如果存在当前事务的话，那么该方法会在这个事务中运行|
|PROPAGATION_MANDATORY |方法必须在事务中运行，如果当前事务不存在，则会抛出一个异常|
|PROPAGATION_REQUIRED_NEW |当前方法必须运行在它自己的事务中。一个新的事务将被启动。如果存在当前事务，在该方法执行期间，当前事务会被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager。|
|PROPAGATION_NOT_SUPPORTED|该方法不应该运行在事务中。如果存在当前事务，在该方法运行期间，当前事务将被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager。|
|PROPAGATION_NEVER |表示当前方法不应该运行在事务上下文中。如果当前正有一个事务在运行，则会抛出异常。|
|PROPAGATION_NESTED |表示如果当前已经存在一个事务，那么该方法将会在嵌套事务中运行。嵌套的事务可以独立于当前事务进行单独地提交或回滚。如果当前事务不存在，那么其行为与PROPAGATION_REQUIRED一样。|

### 四、事务的隔离级别
##### 1.数据库事务并发问题
假设现在有两个事务：A和B并发执行。  
* **脏读**（Dirty reads)  
1）A将某条记录的AGE值从20修改为30。  
2）B读取了A更新后的值：30。  
3）A回滚，AGE值恢复到了20。  
4）B读取到的30就是一个无效的值。  
* **不可重复读**（Nonrepeatable read）
1）A读取了AGE值为20。
2）B将AGE值修改为30。
3）A再次读取AGE值为30，和第一次读取不一致。
* **幻读**（Phantom read）
1）A读取了STUDENT表中的一部分数据。
2）B向STUDENT表中插入了新的行。
3）A读取了STUDENT表时，多出了一些行或少了一些行。

##### 2.隔离级别
数据库系统必须具有隔离并发运行各个事务的能力，使它们不会相互影响，避免各种并发问题。一个事务与其他事务隔离的程度称为隔离级别。SQL标准中规定了多种事务隔离级别，不同隔离级别对应不同的干扰程度，隔离级别越高，数据一致性就越好，但并发性越弱。

|隔离级别|含义|举例|
|---|---|---|
|DEFAULT|使用后端数据库默认的隔离级别|
|READ_UNCOMMITTED |最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读|允许A读取B未提交的修改。|
|READ_COMMITTED |允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生|要求A只能读取B已提交的修改。|
|REPEATABLE_READ |对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。|确保A可以多次从一个字段中读取到相同的值，即A执行期间禁止其它事务对这个字段进行更新。|
|SERIALIZABLE |最高的隔离级别，完全服从ACID的隔离级别，确保阻止脏读、不可重复读以及幻读。|确保A可以多次从一个表中读取到相同的行，在A执行期间，禁止其它事务对这个表进行添加、更新、删除操作。可以避免任何并发问题，但性能十分低下。|

各个隔离级别解决并发问题的能力见下表: 

|隔离级别|脏读|不可重复读|幻读|
|---|---|---|---|
|READ_UNCOMMITTED|√|√|√|
|READ_COMMITTED|×|√|√|
|REPEATABLE_READ|×|×|√|
|SERIALIZABLE|×|×|×|

各种数据库产品对事务隔离级别的支持程度：

|隔离级别	|Oracle	|MySQL|
|---|---|---|
|READ_UNCOMMITTED|×|√|
|READ_COMMITTED|√|√|
|REPEATABLE_READ|×|√(默认)|
|SERIALIZABLE|√|√|

### 五、事务其他属性
##### 1.只读
事务的第三个特性是它是否为只读事务。如果事务只对后端的数据库进行该操作，数据库可以利用事务的只读特性来进行一些特定的优化。通过将事务设置为只读，你就可以给数据库一个机会，让它应用它认为合适的优化措施。

##### 2.事务超时 
为了使应用程序很好地运行，事务不能运行太长的时间。因为事务可能涉及对后端数据库的锁定，所以长时间的事务会不必要的占用数据库资源。事务超时就是事务的一个定时器，在特定时间内事务如果没有执行完毕，那么就会自动回滚，而不是一直等待其结束。

##### 3.回滚规则
事务五边形的最后一个方面是一组规则，这些规则定义了哪些异常会导致事务回滚而哪些不会。默认情况下，事务只有遇到运行期异常时才会回滚，而在遇到检查型异常时不会回滚（这一行为与EJB的回滚行为是一致的） 
但是你可以声明事务在遇到特定的检查型异常时像遇到运行期异常那样回滚。同样，你还可以声明事务遇到特定的异常不回滚，即使这些异常是运行期异常。
### 六、基于XML实现事务管理
##### 1.创建数据库、插入数据表
```shell script
CREATE DATABASE spring;
USE spring;
CREATE TABLE account (
    id INT (11) PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(20) NOT NULL,
    money INT DEFAULT NULL
);
INSERT INTO account VALUES (1,'张三',1000);
INSERT INTO account VALUES (2,'李四',1000);
```
##### 2.引入数据源
创建c3p0-db.properties 的配置文件
```properties
jdbc.driverClass = com.mysql.jdbc.Driver
jdbc.jdbcUrl = jdbc:mysql://localhost:3306/spring
jdbc.user = root
jdbc.password = root
```
##### 3.创建DAO层接口实现类
```java
public interface AccountDao {
    // 汇款
    public void out(String outUser, int money);
    // 收款
    public void in(String inUser, int money);
}
```
```java
public class AccountDaoImpl implements AccountDao {

    private JdbcTemplate jdbcTemplate;

    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
    // 汇款的实现方法
    @Override
    public void out(String outUser, int money) {
        this.jdbcTemplate.update("update account set money =money - ? "
                + "where username =?", money, outUser);
    }
    // 收款的实现方法
    @Override
    public void in(String inUser, int money) {
        this.jdbcTemplate.update("update account set money =money + ? "
                + "where username =?", money, inUser);
    }
}
```
##### 4.创建Service层接口实现类
```java
public interface AccountService {
    //转账
    public void transfer(String outUser, String inUser, int money);
}
```
```java
public class AccountServiceImpl implements AccountService {

    private AccountDao accountDao;

    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }

    @Override
    public void transfer(String outUser, String inUser, int money) {
        this.accountDao.out(outUser, money);
        this.accountDao.in(inUser, money);
    }
}
```

##### 5.创建 Spring 配置文件XmlBean.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/tx
            http://www.springframework.org/schema/tx/spring-tx.xsd
            http://www.springframework.org/schema/aop
            http://www.springframework.org/schema/aop/spring-aop.xsd">
    <!-- 加载properties文件 -->
    <context:property-placeholder location="classpath:c3p0-db.properties"/>
    <!-- 配置数据源，读取properties文件信息 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driverClass}"/>
        <property name="jdbcUrl" value="${jdbc.jdbcUrl}"/>
        <property name="user" value="${jdbc.user}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
    <!-- 配置jdbc模板 -->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!-- 配置dao -->
    <bean id="accountDao" class="com.aop.tran.dao.impl.AccountDaoImpl">
        <property name="jdbcTemplate" ref="jdbcTemplate"/>
    </bean>
    <!-- 配置service -->
    <bean id="accountService" class="com.aop.tran.service.impl.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"/>
    </bean>

    <!--配置步骤-->
    <!-- 1.事务管理器，依赖于数据源 -->
    <bean id="txManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!-- 2.配置事务的通知引用事务管理器 编写通知：对事务进行增强（通知），需要编写切入点和具体执行事务的细节 -->
    <tx:advice id="txAdvice" transaction-manager="txManager">
        <!-- 3.在 tx:advice 标签内部配置事务的属性 -->
        <!--指定方法名称：是业务核心方法
        read-only：是否是只读事务。默认false，不只读。
        isolation：指定事务的隔离级别。默认值是使用数据库的默认隔离级别。
        propagation：指定事务的传播行为。
        timeout：指定超时时间。默认值为：-1。永不超时。
        rollback-for：用于指定一个异常，当执行产生该异常时，事务回滚。产生其他异常，事务不回滚。
        没有默认值，任何异常都回滚。
        no-rollback-for：用于指定一个异常，当产生该异常时，事务不回滚，产生其他异常时，事务回
        滚。没有默认值，任何异常都回滚。
        -->
        <tx:attributes>
            <!-- 给切入点方法添加事务详情，name表示方法名称，*表示任意方法名称-->
            <tx:method name="find*" propagation="SUPPORTS"
                       rollback-for="Exception"/>
            <tx:method name="*" propagation="REQUIRED" isolation="DEFAULT"
                       read-only="false"/>
        </tx:attributes>
    </tx:advice>
    <!-- 4.配置AOP切入点表达式，让Spring自动对目标生成代理 -->
    <aop:config>
        <!-- 切入点 -->
        <aop:pointcut expression="execution(* com.aop.tran..service.*.*(..))"
                      id="txPointCut"/>
        <!-- 5.整合切入点表达式和事务通知-->
        <aop:advisor pointcut-ref="txPointCut" advice-ref="txAdvice"/>
    </aop:config>
</beans>
```
##### 6.创建测试方法
```java
    @Test
    void testXml() {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("XmlBean.xml");
        AccountService accountService = (AccountService) applicationContext
                .getBean("accountService");
        accountService.transfer("张三", "李四", 100);
    }
```
查看测试结果：
### 七、基于Annotation实现事务管理
##### 1.创建配置文件AnnotationBean.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/tx
            http://www.springframework.org/schema/tx/spring-tx.xsd
            http://www.springframework.org/schema/aop
            http://www.springframework.org/schema/aop/spring-aop.xsd">
    <!-- 加载properties文件 -->
    <context:property-placeholder location="classpath:c3p0-db.properties" />
    <!-- 配置数据源，读取properties文件信息 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driverClass}" />
        <property name="jdbcUrl" value="${jdbc.jdbcUrl}" />
        <property name="user" value="${jdbc.user}" />
        <property name="password" value="${jdbc.password}" />
    </bean>
    <!-- 配置jdbc模板 -->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource" />
    </bean>
    <!-- 配置dao -->
    <bean id="userDao" class="com.aop.tran.dao.impl.UserDaoImpl">
        <property name="jdbcTemplate" ref="jdbcTemplate" />
    </bean>
    <!-- 配置service -->
    <bean id="userService" class="com.aop.tran.service.impl.UserServiceImpl">
        <property name="userDao" ref="userDao" />
    </bean>
    <!-- 事务管理器，依赖于数据源 -->
    <bean id="txManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>
    <!-- 注册事务管理驱动 -->
    <tx:annotation-driven transaction-manager="txManager"/>
</beans>
```
##### 2.添加 @Transactional 注解
AccountServiceImpl，在文件中添加 @Transactional 注解及参数.
```java
@Transactional(propagation = Propagation.REQUIRED, isolation = Isolation.DEFAULT, readOnly = false)
public class UserServiceImpl implements UserService {
    private UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    public void transfer(String outUser, String inUser, int money) {
        this.userDao.out(outUser, money);
        // 模拟断电
        int i = 1 / 0;
        this.userDao.in(inUser, money);
    }
}
```
##### 3.编写测试方法
```java
    @Test
    void testAnnotation() {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("AnnotationBean.xml");
        UserService userService = (UserService) applicationContext
                .getBean("userService");
        userService.transfer("张三", "李四", 100);
    }
```
测试结果：
### 八、配置数据源
##### 1. 配置 C3P0 数据源
```xml
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driverClass}" />
        <property name="jdbcUrl" value="${jdbc.jdbcUrl}" />
        <property name="user" value="${jdbc.user}" />
        <property name="password" value="${jdbc.password}" />
    </bean>
```
##### 2. 配置 DBCP 数据源
````xml
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql://localhost:3306/spring"></property>
        <property name="username" value="root"></property>
        <property name="password" value="root"></property>
    </bean>
````
##### 3. 配置 spring 内置数据源
```xml
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql://localhost:3306/spring"></property>
        <property name="username" value="root"></property>
        <property name="password" value="root"></property>
    </bean>
```

### 参考资料
[事务](https://www.liaoxuefeng.com/wiki/1177760294764384/1179611198786848)  
[彻底理解数据库事务](https://mp.weixin.qq.com/s?__biz=MzI3NzE0NjcwMg==&mid=402303897&idx=2&sn=5c04638c1a1e555ff50012561fa70ea9&scene=21#wechat_redirect)  
[Spring 事务管理](https://www.w3cschool.cn/wkspring/by1r1ha2.html)  
[java开发事务篇](https://juejin.im/post/6844904186870497287)  
[Spring事务管理 与 SpringAOP](https://www.cnblogs.com/xdyixia/p/9376077.html)