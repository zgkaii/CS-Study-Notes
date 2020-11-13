## 【MyBatis学习笔记】入门案例
### 一、概述
MyBatis 是一个优秀的基于 java 的持久层框架，它内部封装了 jdbc，使开发者只需要关注 sql 语句本身，
而不需要花费精力去处理加载驱动、创建连接、创建 statement 等繁杂的过程。  
MyBatis 通过 xml 或注解的方式将要执行的各种 statement 配置起来，并通过 java 对象和 statement 中
sql 的动态参数进行映射生成最终执行的 sql 语句，最后由 MyBatis 框架执行 sql 并将结果映射为 java 对象并
返回。  
采用 ORM 思想解决了实体和数据库映射的问题，对 jdbc 进行了封装，屏蔽了 jdbc api 底层访问细节，使我
们不用与 jdbc api 打交道，就可以完成对数据库的持久化操作。  
为了我们能够更好掌握框架运行的内部过程，并且有更好的体验，下面我们将从自定义 MyBatis 框架开始来
学习框架。此时我们将会体验框架从无到有的过程体验，也能够很好的综合前面阶段所学的基础。  

### 二、基于xml配置的MyBatis
##### 1.创建maven工程
```
GroupId:com.kai
ArtifacttId:mybatis_basic
packing:jar
```
##### 2.导入依赖
```xml
<dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.5</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.17</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.10</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.12</version>
        </dependency>
```
##### 3.编写 User 实体类 
```java
@Data
public class User {

    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;

}
```
##### 4.编写持久层接口 UserDao
```java
public interface UserDao {
    /**
     * 查询所有操作
     * @return
     */
    List<User> findAll();
}
```
##### 5.编写持久层接口的映射文件 UserDao.xml 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.kai.dao.UserDao">
    <!--配置查询所有-->
    <select id="findAll" resultType="com.kai.domain.User">
        select * from user
    </select>
</mapper>
```
##### 6.编写MyBatis主配置文件 SqlMapConfig.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- mybatis的主配置文件 -->
<configuration>
    <!-- 配置环境 -->
    <environments default="mysql">
        <!-- 配置mysql的环境-->
        <environment id="mysql">
            <!-- 配置事务的类型-->
            <transactionManager type="JDBC"></transactionManager>
            <!-- 配置数据源（连接池） -->
            <dataSource type="POOLED">
                <!-- 配置连接数据库的4个基本信息 -->
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?serverTimezone=UTC"/>
                <property name="username" value="root"/>
                <property name="password" value="513701"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件 -->
    <mappers>
        <mapper resource="com/kai/dao/UserDao.xml"/>
    </mappers>
</configuration>
```
##### 7.配置log4j.properties
```properties
# Set root category priority to INFO and its only appender to CONSOLE.
#log4j.rootCategory=INFO, CONSOLE            debug   info   warn error fatal
log4j.rootCategory=debug, CONSOLE, LOGFILE

# Set the enterprise logger category to FATAL and its only appender to CONSOLE.
log4j.logger.org.apache.axis.enterprise=FATAL, CONSOLE

# CONSOLE is set to be a ConsoleAppender using a PatternLayout.
log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
log4j.appender.CONSOLE.layout.ConversionPattern=%d{ISO8601} %-6r [%15.15t] %-5p %30.30c %x - %m\n

# LOGFILE is set to be a File appender using a PatternLayout.
log4j.appender.LOGFILE=org.apache.log4j.FileAppender
log4j.appender.LOGFILE.File=d:\axis.log
log4j.appender.LOGFILE.Append=true
log4j.appender.LOGFILE.layout=org.apache.log4j.PatternLayout
log4j.appender.LOGFILE.layout.ConversionPattern=%d{ISO8601} %-6r [%15.15t] %-5p %30.30c %x - %m\n
```
##### 8.编写测试类
```java
public class MybatisTest {
    /**
     * 入门案例
     *
     * @param args
     */
    public static void main(String[] args) throws Exception {
        //1.读取配置文件
        InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.创建SqlSessionFactory工厂
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(in);
        //3.使用工厂生产SqlSession对象
        SqlSession session = factory.openSession();
        //4.使用SqlSession创建Dao接口的代理对象
        UserDao userDao = session.getMapper(UserDao.class);
        //5.使用代理对象执行方法
        List<User> users = userDao.findAll();
        for (User user : users) {
            System.out.println(user);
        }
        //6.释放资源
        session.close();
        in.close();
    }
}
```
### 三、基于注解配置的MyBatis
##### 1.移除持久层接口的映射文件
删除 UserDao.xml 
##### 2.在持久层接口中添加注解 
```java
public interface UserDao {
    /**
     * 查询所有操作
     * @return
     */
    @Select("select * from user")
    List<User> findAll();
}
```
##### 1. 修改MyBatis主配置文件SqlMapConfig.xml 
```xml
    <!--用注解来配置的话，此处应该使用class属性指定被注解的dao全限定类名-->
    <mappers>
        <mapper class="com.kai.dao.UserDao"/>
    </mappers>
```

![mybatis](https://img-blog.csdnimg.cn/20200902071539603.png)







### 附录
##### 1、Config约束与Mapper约束
* Config:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration  
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"  
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
```

* Mapper:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper  
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"  
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
```

##### 2.MyBatis配置文件详解
```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration><!-- 配置 -->
    <properties /><!-- 属性 -->
    <settings /><!-- 全局配置参数 -->
    <typeAliases /><!-- 类型别名 -->
    <typeHandlers /><!-- 类型处理器 -->
    <objectFactory /><!-- 对象工厂 -->
    <plugins /><!-- 插件 -->
    <environments><!-- 环境集合属性对象 -->
        <environment><!-- 环境子属性对象 -->
            <transactionManager /><!-- 事务管理器 -->
            <dataSource /><!-- 数据源 -->
        </environment>
    </environments>
    <databaseIdProvider /><!-- 数据库厂商标识 -->
    <mappers /><!-- 映射器 -->
</configuration>
```

### 参考资料
[Mybatis](https://blog.csdn.net/isea533/category_2092001.html)  
[MyBatis入门](http://c.biancheng.net/mybatis/)