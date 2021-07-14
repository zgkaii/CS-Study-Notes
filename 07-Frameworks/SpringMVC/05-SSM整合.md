- [整合SSM框架](#整合ssm框架)
  - [一. 搭建整合环境](#一-搭建整合环境)
      - [1.整合的思路](#1整合的思路)
      - [2.创建数据库和表结构](#2创建数据库和表结构)
      - [3.创建maven的工程](#3创建maven的工程)
      - [4.pom.xml引入坐标依赖](#4pomxml引入坐标依赖)
      - [5. 编写实体类](#5-编写实体类)
      - [6. 编写dao接口](#6-编写dao接口)
      - [7. 编写service接口和实现类](#7-编写service接口和实现类)
  - [二、Spring框架代码的编写](#二spring框架代码的编写)
      - [1. applicationContext.xml配置文件](#1-applicationcontextxml配置文件)
      - [2. 编写测试方法SpringTest](#2-编写测试方法springtest)
  - [三、SpringMVC框架编写](#三springmvc框架编写)
      - [1. 配置DispatcherServlet前端控制器](#1-配置dispatcherservlet前端控制器)
      - [2. 配置DispatcherServlet过滤器解决中文乱码](#2-配置dispatcherservlet过滤器解决中文乱码)
      - [3. 创建springmvc.xml的配置文件](#3-创建springmvcxml的配置文件)
      - [4.测试SpringMVC的框架搭建是否成功](#4测试springmvc的框架搭建是否成功)
  - [四、Spring整合SpringMVC框架](#四spring整合springmvc框架)
      - [1.配置ContextLoaderListener监听器](#1配置contextloaderlistener监听器)
      - [2.在controller中注入service对象](#2在controller中注入service对象)
  - [五、MyBatis框架代码的编写](#五mybatis框架代码的编写)
      - [1.编写SqlMapConfig.xml配置文件](#1编写sqlmapconfigxml配置文件)
      - [2.添加log4j.properties](#2添加log4jproperties)
      - [3. AccountDao接口上注解编写SQL语句](#3-accountdao接口上注解编写sql语句)
      - [4. 编写测试的方法MyBatisTest](#4-编写测试的方法mybatistest)
  - [六、Spring整合MyBatis框架](#六spring整合mybatis框架)
      - [1.修改applicationContext.xml](#1修改applicationcontextxml)
      - [2.AccountDao接口中添加@Repository注解](#2accountdao接口中添加repository注解)
      - [3.在service中注入dao对象](#3在service中注入dao对象)
      - [4.修改Controller测试方法](#4修改controller测试方法)
      - [5.修改list.jsp](#5修改listjsp)
      - [6.修改index.jsp](#6修改indexjsp)
      - [7. 配置Spring的声明式事务管理](#7-配置spring的声明式事务管理)
      - [8. 最终](#8-最终)
## 整合SSM框架
### 一. 搭建整合环境

##### 1.整合的思路
SSM整合可以使用多种方式，咱们会选择XML + 注解的方式  
* 先搭建整合的环境  
* 先把Spring的配置搭建完成  
* 再使用Spring整合SpringMVC框架  
* 最后使用Spring整合MyBatis框架  

##### 2.创建数据库和表结构
```mysql
create database ssm;
use ssm;
create table account(
	id int primary key auto_increment,
	name varchar(20),
	money double
);
```
##### 3.创建maven的工程
* 创建ssm_parent父工程（打包方式选择pom）  
* 创建ssm_web子模块（打包方式是war包）  
* 创建ssm_service子模块（打包方式是jar包）  
* 创建ssm_dao子模块（打包方式是jar包）  
* 创建ssm_domain子模块（打包方式是jar包）  
* web依赖于service，service依赖于dao，dao依赖于domain  
##### 4.pom.xml引入坐标依赖
```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <spring.version>5.0.2.RELEASE</spring.version>
    <slf4j.version>1.6.6</slf4j.version>
    <log4j.version>1.2.12</log4j.version>
    <mysql.version>5.1.6</mysql.version>
    <mybatis.version>3.4.5</mybatis.version>
  </properties>

  <dependencies>
    <!-- spring -->
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.6.8</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aop</artifactId>
      <version>${spring.version}</version>
    </dependency>
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
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>${mysql.version}</version>
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
    <dependency>
      <groupId>jstl</groupId>
      <artifactId>jstl</artifactId>
      <version>1.0.2</version>
    </dependency>
    <!-- log start -->
    <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
      <version>${log4j.version}</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>${slf4j.version}</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-log4j12</artifactId>
      <version>${slf4j.version}</version>
    </dependency>
    <!-- log end -->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>${mybatis.version}</version>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>1.3.2</version>
    </dependency>
    <dependency>
      <groupId>c3p0</groupId>
      <artifactId>c3p0</artifactId>
      <version>0.9.1.2</version>
      <type>jar</type>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <version>1.18.10</version>
    </dependency>
    <dependency>
      <groupId>taglibs</groupId>
      <artifactId>standard</artifactId>
      <version>1.1.0</version>
    </dependency>
  </dependencies>
```
* 部署ssm_web的项目，只要把ssm_web项目加入到tomcat服务器中即可

##### 5. 编写实体类
```java
@Data
public class Account implements Serializable {
    private Integer id;
    private String name;
    private Double money;
}
```

##### 6. 编写dao接口
```java
public interface AccountDao {

    // 查询所有账户
    public List<Account> findAll();

    // 保存账户所有信息
    public void saveAccount(Account account);
}
```
##### 7. 编写service接口和实现类
```java
public interface AccountService {
    // 查询所有账户
    public List<Account> findAll();

    // 保存账户所有信息
    public void saveAccount(Account account);
}
```
```java
@Service("accountService")
public class AccountServiceImpl implements AccountService {
    @Override
    public List<Account> findAll() {
        System.out.println("业务层：查询所有账户...");
        return null;
    }

    @Override
    public void saveAccount(Account account) {
        System.out.println("业务层：保存所有账户...");
    }
}
```

### 二、Spring框架代码的编写
##### 1. applicationContext.xml配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/aop
            http://www.springframework.org/schema/aop/spring-aop.xsd
            http://www.springframework.org/schema/tx
            http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!--开启注解扫描，只处理service与dao，controller用springMVC处理-->
    <context:component-scan base-package="com.mvc">
        <!--排除controller包的扫描-->
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
</beans>
```
##### 2. 编写测试方法SpringTest
```java
public class SpringTest {
    @Test
    public void test() {
        // 加载配置文件
        ApplicationContext ac = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
        // 获取对象
        AccountService as = (AccountService) ac.getBean("accountService");
        // 调用方法
        as.findAll();
    }
}
```
### 三、SpringMVC框架编写
##### 1. 配置DispatcherServlet前端控制器
```xml
<!--配置前端控制器-->
  <servlet>
    <servlet-name>springMVC</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
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
```
##### 2. 配置DispatcherServlet过滤器解决中文乱码
```xml
  <!-- 配置解决中文乱码的过滤器 -->
  <filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```
##### 3. 创建springmvc.xml的配置文件
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
    <!-- 1.配置spring创建容器时要扫描的包 只扫描controller-->
    <context:component-scan base-package="com.mvc">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
    <!-- 2.配置视图解析器 -->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--文件所在目录-->
        <property name="prefix" value="/WEB-INF/pages/"></property>
        <!--文件后缀名-->
        <property name="suffix" value=".jsp"></property>
    </bean>
    <!-- 3.过滤静态资源 -->
    <mvc:resources location="/css/" mapping="/css/**"/>
    <mvc:resources location="/images/" mapping="/images/**"/>
    <mvc:resources location="/js/" mapping="/js/**"/>
    <!-- 4.配置spring开启注解mvc的支持-->
    <!-- 自动加载 RequestMappingHandlerMapping（处理映射器）和RequestMappingHandlerAdapter（处理适配器）-->
    <mvc:annotation-driven/>
</beans>
```
##### 4.测试SpringMVC的框架搭建是否成功
* 编写index.jsp
```jsp
    <h3>整合SpringMVC</h3>
    <a href="account/findAll">整合</a>
```
* 编写list.jsp
```jsp
    <h3>查询了所有账户信息！</h3>
```
* 创建AccountController类，编写方法
```java
@Controller
@RequestMapping("/account")
public class AccountController {

    @RequestMapping("/findAll")
    public String findAll() {
        System.out.println("表现层：查询所有账户...");
        return "list";
    }
}
```
### 四、Spring整合SpringMVC框架
##### 1.配置ContextLoaderListener监听器
```xml
  <!--配置Spring的监听器,默认只加载WEB-INF目录下的applicationContext.xml配置文件-->-->
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  <!--设置配置文件的路径 applicationContext.xml-->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
  </context-param>
```
##### 2.在controller中注入service对象
```java
@Controller
@RequestMapping("/account")
public class AccountController {
    @Autowired
    private AccountService accoutService;
    /**
     * 查询所有的数据
     * @return
     */
    @RequestMapping("/findAll")
    public String findAll() {
        System.out.println("表现层：查询所有账户...");
        accoutService.findAll();
        return "list";
    }
}
```
### 五、MyBatis框架代码的编写
##### 1.编写SqlMapConfig.xml配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="mysql">
        <environment id="mysql">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql:///ssm"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>
    <!-- 使用的是注解 -->
    <mappers>
        <!-- <mapper class="cn.mvc.dao.AccountDao"/> -->
        <!-- 该包下所有的dao接口都可以使用 -->
        <package name="cn.mvc.dao"/>
    </mappers>
</configuration>
```
##### 2.添加log4j.properties
```powershell
# Set root category priority to INFO and its only appender to CONSOLE.
#log4j.rootCategory=INFO, CONSOLE            debug   info   warn error fatal
log4j.rootCategory=info, CONSOLE, LOGFILE

# Set the enterprise logger category to FATAL and its only appender to CONSOLE.
log4j.logger.org.apache.axis.enterprise=FATAL, CONSOLE

# CONSOLE is set to be a ConsoleAppender using a PatternLayout.
log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
log4j.appender.CONSOLE.layout.ConversionPattern=%d{ISO8601} %-6r [%15.15t] %-5p %30.30c %x - %m\n

# LOGFILE is set to be a File appender using a PatternLayout.
log4j.appender.LOGFILE=org.apache.log4j.FildeAppender
log4j.appender.LOGFILE.File=d:\axis.log
log4j.appender.LOGFILE.Append=true
log4j.appender.LOGFILE.layout=org.apache.log4j.PatternLayout
log4j.appender.LOGFILE.layout.ConversionPattern=%d{ISO8601} %-6r [%15.15t] %-5p %30.30c %x - %m\n
```

##### 3. AccountDao接口上注解编写SQL语句
```java
@Repository
public interface AccountDao {

    // 查询所有账户
    @Select("select * from account")
    public List<Account> findAll();

    // 保存账户所有信息
    @Insert("insert into account (name,money) values (#{name},#{money})")
    public void saveAccount(Account account);
}
```
##### 4. 编写测试的方法MyBatisTest
```java
    /**
     * 测试查询
     * @throws Exception
     */
    @Test
    public void test01() throws Exception {
        // 加载配置文件
        InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
        // 创建SqlSessionFactory对象
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
        // 创建SqlSession对象
        SqlSession session = factory.openSession();
        // 获取到代理对象
        AccountDao dao = session.getMapper(AccountDao.class);
        // 查询所有数据
        List<Account> list = dao.findAll();
        for(Account account : list){
            System.out.println(account);
        }
        // 关闭资源
        session.close();
        in.close();
    }

    /**
     * 测试保存
     * @throws Exception
     */
    @Test
    public void test02() throws Exception {
        Account account = new Account();
        account.setName("wangwu");
        account.setMoney(400d);

        // 加载配置文件
        InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
        // 创建SqlSessionFactory对象
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
        // 创建SqlSession对象
        SqlSession session = factory.openSession();
        // 获取到代理对象
        AccountDao dao = session.getMapper(AccountDao.class);
        // 保存
        dao.saveAccount(account);
        // 提交事务
        session.commit();
        // 关闭资源
        session.close();
        in.close();
    }
```
### 六、Spring整合MyBatis框架
##### 1.修改applicationContext.xml
SqlMapConfig.xml配置文件中的内容配置到applicationContext.xml配置文件中
```xml
    <!--Spring整合MyBatis框架-->
    <!--配置连接池-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/ssm"/>
        <property name="user" value="root"/>
        <property name="password" value="root"/>
    </bean>
    <!--配置SqlSessionFactory工厂-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
    </bean>
    <!--配置AccountDao接口所在包-->
    <bean id="mapperScanner" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.mvc.dao"/>
    </bean>
```
##### 2.AccountDao接口中添加@Repository注解
##### 3.在service中注入dao对象
```java
    @Autowired
    private AccountDao accountDao;

    @Override
    public List<Account> findAll() {
        System.out.println("业务层：查询所有账户...");
        return accountDao.findAll();
    }

    @Override
    public void saveAccount(Account account) {
        System.out.println("业务层：保存所有账户...");
        accountDao.saveAccount(account);
    }
}
```
##### 4.修改Controller测试方法
```java
    @Autowired
    private AccountService accountService;

    @RequestMapping("/findAll")
    public String findAll(Model model){
        System.out.println("表现层：查询所有账户...");
        // 调用service的方法
        List<Account> list = accountService.findAll();
        model.addAttribute("list", list);
        return "list";
    }

    /**
     * 保存
     * @return
     */
    @RequestMapping("/save")
    public void save(Account account, HttpServletRequest request, HttpServletResponse response) throws IOException {
        accountService.saveAccount(account);
        response.sendRedirect(request.getContextPath()+"/account/findAll");
        return;
    }
```
##### 5.修改list.jsp
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h3>查询了所有账户信息！</h3>

    <c:forEach items="${list}" var="account">
        ${account.name}
    </c:forEach>
</body>
</html>
```

##### 6.修改index.jsp
```jsp
    <h3>整合SpringMVC</h3>
    <a href="account/findAll">整合</a>

    <h3>测试包</h3>

    <form action="account/save" method="post">
        name：<input type="text" name="name" /><br/>
        money：<input type="text" name="money" /><br/>
        <input type="submit" value="保存"/><br/>
    </form>
```
##### 7. 配置Spring的声明式事务管理
```xml
    <!--配置Spring框架声明式事务管理-->
    <!--配置事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>
    <!--配置事务通知-->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="find*" read-only="true"/>
            <tx:method name="*" isolation="DEFAULT"/>
        </tx:attributes>
    </tx:advice>
    <!--配置AOP增强-->
    <aop:config>
        <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.mvc.service.impl.*ServiceImpl.*(..))"/>
    </aop:config>
```

##### 8. 最终
* 项目结构
![](https://img-blog.csdnimg.cn/20200918233637584.png)
* 测试结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918234321474.png)
* 数据库
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918234407183.png)





