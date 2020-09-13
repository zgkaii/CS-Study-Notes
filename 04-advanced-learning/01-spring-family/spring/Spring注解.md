### Spring新注解：
|注解|说明|
|---|---|
|@Configuration|用于指定当前类是一个 Spring 配置类，当创建容器时会从该类上加载注解|
|@ComponentScan|用于指定 Spring 在初始化容器时要扫描的包。 作用和在 Spring 的 xml 配置文件中的 `<context:component-scan base-package="com.kai"/>`一样|
|@Bean|解只能写在方法上，表明使用此方法创建一个对象，并且放入 spring 容器|
|@PropertySource|用于加载.properties 文件中的配置|
|@Import|用于导入其他配置类|

```java
@Configuration
@ComponentScan("com.kai")
@Import({DataSourceConfiguration.class})
public class SpringConfiguration {
}
```

```java
@PropertySource("classpath:jdbc.properties")
public class DataSourceConfiguration {
    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String username;
    @Value("${jdbc.password}")
    private String password;
```

```java
@Bean(name="dataSource")
public DataSource getDataSource() throws PropertyVetoException { 
    ComboPooledDataSource dataSource = new ComboPooledDataSource(); 
    dataSource.setDriverClass(driver);
    dataSource.setJdbcUrl(url);
    dataSource.setUser(username);
    dataSource.setPassword(password);
    return dataSource;
}
```

#####  Spring 整合 Junit  
1、导入spring整合junit的jar(坐标)  
2、使用Junit提供的一个注解把原有的main方法替换了，替换成spring提供的 @Runwith  
3、告知spring的运行器，spring和ioc创建是基于xml还是注解的，并且说明位置  
 @ContextConfiguration  
* locations：指定xml文件的位置，加上classpath关键字，表示在类路径下
* classes：指定注解类所在地位置

当我们使用spring 5.x版本的时候，要求junit的jar必须是4.12及以上

