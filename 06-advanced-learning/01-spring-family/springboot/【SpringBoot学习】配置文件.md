## 一、配置文件

在使用`Spring Initializer`快速创建SpringBoot项目时，会在resources目录下给我们一个默认的**空的**全局配置文件 `application.properties`。配置文件的作用也正是用来修改SpringBoot自动配置的默认值。

配置文件名是固定的

> application.properties

但我们可以修改为

> application.yml

这两个文件本质是一样的，区别只是其中的语法略微不同。

application.yml 配置文件使用YMAL语言，YMAL不是如XML般的标记语言，以数据为中心，更适合作为配置文件。

YAML：

```yaml
server:
  port: 8081
```

XML：

```xml
<server>
	<port>8081</port>
</server>
```

## 二、YAML语法

### 2.1 基本语法

* 使用缩进表示层级关系 

* 缩进时不允许使用Tab键，只允许使用空格。 
* 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可 
* 大小写敏感

**k:(空格)v**——表示一对键值对（空格必须有）。以**空格**的缩进来控制层级关系，只要是左对齐的一列数据，都是同一个层级的。

```yaml
server:
    port: 8081
    path: /hello
```

### 2.2 值的写法

#### （1）字面量

字面量：普通的值（数字，字符串，布尔，日期 ）

`k: v`——字面直接书写

* 字符串默认不用加上单引号或者双引号

```yaml
name: zhangsan
```

`""`：双引号，不会转义字符串里面的特殊字符，特殊字符会作为本身想表示的意思

```yaml
name: "zhangsan \n lisi"
```

输出: 

```shell
zhangsan
lisi
```

`''`：单引号，会转义特殊字符，特殊字符最终只是一个普通的字符串数据

```yaml
name: 'zhangsan \n lisi'
```

输出：

```shell
zhangsan \n lisi
```

#### （2）对象、Map（属性和值）（键值对）

​	`k: v`——在下一行来写对象的属性和值的关系（注意缩进）

```yaml
friends:
	lastName: zhangsan
	age: 20
```

行内写法：

```yaml
friends: {lastName: zhangsan,age: 18}
```

#### （3）数组（List、Set）

用`- 值`表示数组中的一个元素

```yaml
pets:
 - cat
 - dog
 - pig
```

行内写法

```yaml
pets: [cat,dog,pig]
```

## 三、配置文件值注入

### 3.1 配置文件书写

#### （1）yaml配置文件

配置文件application.yml

```yaml
person:
  lastName: zhangsan
  age: 20
  boss: false
  birth: 2020/10/10
  maps: {k1: v1,k2: 15}
  lists:
    - lisi
    - wangwu
  dog:
    name: 狗狗
    age: 5
```

#### （2）properties配置文件

或者application.properties

```properties
person.last-name=张三 #松散绑定
person.age=20
person.birth=2020/10/10
person.boss=true
person.maps.k1=v1
person.maps.k2=15
person.lists=lisi,wangwu
person.dog.name=狗狗
person.dog.age=5
```

解决properties配置文件默认utf-8可能乱码问题：

![](https://img-blog.csdnimg.cn/2020101616292441.png#pic_center)

### 3.2 属性注入

#### （1）@ConfigurationProperties

JavaBean：

```java
/**
 * 将配置文件中配置的每一个属性的值，映射到这个组件中。
 * @ConfigurationProperties：告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定。
 *      prefix = "person"：与配置文件中person下面的所有属性一一映射
 *
 * 只有这个组件是容器中的组件，才能容器提供的@ConfigurationProperties功能。
 *
 */
@Component
@ConfigurationProperties(prefix = "person")
@Validated
@Data
public class Person {
    @NotNull //JSR303数据校验
    private String LastName;
    private String age;
    private boolean boss;
    private Date birthday;

    private Map<String,Object> maps;//复杂类型封装
    private List<Object> lists;
    private Dog dog;
}
```

导入配置文件处理器，配置文件进行绑定就会有提示：

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-configuration-processor</artifactId>
			<optional>true</optional>
		</dependency>
```

编写测试方法：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
class SpringbootConfigApplicationTests {
    @Autowired
    Person person;

    @Test
    void test01() {
        System.out.println(person);
    }
}
```

输出结果：

```java
Person(LastName=zhangsan, age=20, boss=false, birthday=Sat Oct 10 00:00:00 CST 2020, maps={k1=v1, k2=15}, lists=[lisi, wangwu], dog=Dog(name=狗狗, age=5))
```

#### （2）@Value

```java
@Component
@Data
public class Person {

   	/**
    * <bean class="Person">
    *      <property name="lastName" value="字面量/${key}从环境变量、配置文件中获取值/#{SpEL}"></property>
    * <bean/>
    */
    @Value("${person.last-name}")
    private String lastName;
    @Value("#{11*2}")//SpEL
    private Integer age;
    @Value("true")
    private Boolean boss;
	@Value("2020/10/10")
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

输出结果：

```java
Person(LastName=张三, age=22, boss=true, birthday=Fri Oct 10 00:00:00 CST 1997, maps=null, lists=null, dog=null)
```

**@Value获取值和@ConfigurationProperties获取值比较**:

| 比较                 | @ConfigurationProperties | @Value     |
| -------------------- | ------------------------ | ---------- |
| 功能                 | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定（松散语法） | 支持                     | 不支持     |
| SpEL                 | 不支持                   | 支持       |
| JSR303数据校验       | 支持                     | 不支持     |
| 复杂类型封装         | 支持                     | 不支持     |

不论配置文件yml还是properties他们都能获取到值。

在某个业务逻辑中需要获取配置文件中的某项值——使用@Value；专门编写了一个JavaBean来和配置文件进行映射——使用@ConfigurationProperties。

### 3.3 @PropertySource&@ImportResource

@**PropertySource**：加载指定的配置文件。

例如将person相关配置都写进person.properties

```java
/**
 *
 * 只有这个组件是容器中的组件，才能容器提供的@ConfigurationProperties功能。
 *  @ConfigurationProperties(prefix = "person")默认从全局配置文件中获取值。
 *
 */
@PropertySource(value = {"classpath:person.properties"})
@Component
@ConfigurationProperties(prefix = "person")
@Data
public class Person {

    private String lastName;
    private Integer age;
    private Boolean boss;
```



@**ImportResource**：导入Spring的配置文件，让配置文件里面的内容生效。

Spring Boot里面没有Spring的配置文件，我们自己编写的配置文件，不能自动识别，需要通过@**ImportResource**注解加载进来。

```java
@ImportResource(locations = {"classpath:beans.xml"})
```



编写Spring的配置文件bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">


    <bean id="helloService" class="com.kai.springboot.service.HelloService"></bean>
</beans>
```

这样编写bean配置文件再导入Spring的配置文件让其生效的方式很麻烦，在SpringBoot中不推荐使用。



SpringBoot推荐给容器中添加组件的方式，使用**全注解**的方式：

（1）配置类**@Configuration**------>Spring配置文件

（2）使用**@Bean**给容器中添加组件

```java
/**
 * @Configuration：指明当前类是一个配置类，就是来替代之前的Spring配置文件。
 *
 * 在配置文件中用<bean><bean/>标签添加组件
 *
 */
@Configuration
public class MyAppConfig {

    //将方法的返回值添加到容器，容器中这个组件默认的id就是方法名。
    @Bean
    public HelloService helloService02(){
        System.out.println("配置类@Bean给容器中添加组件了...");
        return new HelloService();
    }
}
```



## 四、配置文件占位符

### 4.1 随机数

配置文件中可以使用随机数 :

```java
${random.value}、${random.int}、${random.long}
${random.int(10)}、${random.int[1024,65536]}
```

### 4.2 属性配置占位符

可以在配置文件中引用前面配置过的属性（优先级前面配置过的这里都能用），如果没有可以是用 `${app.name:默认值}`来指定找不到属性时的默认值。

```properties
person.last-name=张三${random.uuid}
person.age=${random.int}
person.birth=2020/12/15
person.boss=false
person.maps.k1=v1
person.maps.k2=14
person.lists=a,b,c
person.dog.name=${person.hello:hello}_dog
person.dog.age=15
```

## 五、Profile

Profile是Spring对不同环境提供不同配置功能的支持，可以通过激活、 指定参数等方式快速切换环境。

### 5.1 profile多文件形式

我们在主配置文件编写的时候，文件名可以是   application-{profile}.properties/yml

* application-dev.properties、application-prod.properties

默认使用application.properties的配置。

### 5.2 yml多profile文档块模式

```yml
server:
  port: 8081
spring:
  profiles:
    active: default #表示未指定默认配置

---
server:
  port: 8083
spring:
  profiles: dev

---

server:
  port: 8084
spring:
  profiles: prod  #指定属于哪个环境
```



### 5.3 激活指定profile

​	1、在配置文件中指定

​		`spring.profiles.active=dev`

​	2、命令行

​		`java -jar spring-boot-02-config-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev`

​		可以直接在测试的时候，配置传入命令行参数`spring.profiles.active=dev`

​	3、虚拟机参数

​		`-Dspring.profiles.active=dev`



## 六、配置文件加载位置

springboot 启动会扫描以下位置的application.properties或者application.yml文件作为Spring boot的默认配置文件。

`– file:./config/`

`– file:./`

`– classpath:/config/`

`– classpath:/`

优先级由高到底，**高优先级配置内容**会覆盖**低优先级配置**的重复内容。

SpringBoot会从这四个位置**全部加载**主配置文件——**互补配置**。

![](https://img-blog.csdnimg.cn/20200923111052310.png)



==我们还可以通过spring.config.location来改变默认的配置文件位置。==

**项目打包好以后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置，指定配置文件和默认加载的这些配置文件共同起作用形成互补配置。**

```shell
java -jar spring-boot-config-0.0.1-SNAPSHOT.jar --spring.config.location=C:/application.properties
```



## 七、外部配置加载顺序

**==SpringBoot也可以从以下位置加载配置，优先级从高到低，高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置==**

**1.命令行参数**

所有的配置都可以在命令行上进行指定，多个配置用**空格**分开：`--配置项=值`

```shell
java -jar spring-boot-config-0.0.1-SNAPSHOT.jar --server.port=8087  --server.context-path=/abc
```

2.来自java:comp/env的JNDI属性

3.Java系统属性（System.getProperties()）

4.操作系统环境变量

5.RandomValuePropertySource配置的random.*属性值



==由jar包外向jar包内进行寻找，优先加载带profile配置文件==

**6.jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件**

**7.jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件**



==再来加载不带profile配置文件==

**8.jar包外部的application.properties或application.yml(不带spring.profile)配置文件**

**9.jar包内部的application.properties或application.yml(不带spring.profile)配置文件**



10.@Configuration注解类上的@PropertySource

11.通过SpringApplication.setDefaultProperties指定的默认属性

## 八、自动配置原理

### 8.1 自动配置原理

（1）SpringBoot启动的时候加载主配置类，开启了自动配置功能 @EnableAutoConfiguration

**（2）@EnableAutoConfiguration 作用：**

 - 利用EnableAutoConfigurationImportSelector给容器中导入一些组件，查看`selectImports()`方法的内容：

   `List<String> configurations = getCandidateConfigurations(annotationMetadata,attributes)`获取候选的配置

   - `SpringFactoriesLoader.loadFactoryNames()`

   扫描所有jar包类路径下  `META-INF/spring.factories`，把扫描到的这些文件的内容包装成properties对象，从properties中获取到EnableAutoConfiguration.class类（类名）对应的值，然后把他们添加在容器中。

查看spring.factories：

```properties
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.cloud.CloudAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.ldap.LdapDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.solr.SolrRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
org.springframework.boot.autoconfigure.elasticsearch.jest.JestAutoConfiguration,\
org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\
org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\
org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\
org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\
org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\
org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,\
org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
org.springframework.boot.autoconfigure.mobile.DeviceResolverAutoConfiguration,\
org.springframework.boot.autoconfigure.mobile.DeviceDelegatingViewResolverAutoConfiguration,\
org.springframework.boot.autoconfigure.mobile.SitePreferenceAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
org.springframework.boot.autoconfigure.reactor.ReactorAutoConfiguration,\
org.springframework.boot.autoconfigure.security.SecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.SecurityFilterAutoConfiguration,\
org.springframework.boot.autoconfigure.security.FallbackWebSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.OAuth2AutoConfiguration,\
org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\
org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\
org.springframework.boot.autoconfigure.social.SocialWebAutoConfiguration,\
org.springframework.boot.autoconfigure.social.FacebookAutoConfiguration,\
org.springframework.boot.autoconfigure.social.LinkedInAutoConfiguration,\
org.springframework.boot.autoconfigure.social.TwitterAutoConfiguration,\
org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,\
org.springframework.boot.autoconfigure.web.DispatcherServletAutoConfiguration,\
org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.ErrorMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.web.HttpEncodingAutoConfiguration,\
org.springframework.boot.autoconfigure.web.HttpMessageConvertersAutoConfiguration,\
org.springframework.boot.autoconfigure.web.MultipartAutoConfiguration,\
org.springframework.boot.autoconfigure.web.ServerPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.web.WebClientAutoConfiguration,\
org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.WebSocketAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.WebSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration
```

每一个这样的  xxxAutoConfiguration类都是容器中的一个组件，都加入到容器中，用他们来做自动配置。

（3）每一个自动配置类进行自动配置功能

* 以**HttpEncodingAutoConfiguration（Http编码自动配置）**为例解释自动配置原理：

```java
@Configuration //表示这是一个配置类，以前编写的配置文件一样，也可以给容器中添加组件
@EnableConfigurationProperties(HttpEncodingProperties.class)  //启动指定类的ConfigurationProperties功能；将配置文件中对应的值和HttpEncodingProperties绑定起来，并把HttpEncodingProperties加入到IoC容器中。

@ConditionalOnWebApplication //Spring底层@Conditional注解（Spring注解版），根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效，判断当前应用是否是web应用，如果是则当前配置类生效。

@ConditionalOnClass(CharacterEncodingFilter.class) //判断当前项目有没有这个类CharacterEncodingFilter，SpringMVC中进行乱码解决的过滤器。

@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)  //判断配置文件中是否存在某个配置  spring.http.encoding.enabled，如果不存在，判断也是成立的。
//即使我们配置文件中不配置pring.http.encoding.enabled=true，也是默认生效的。
public class HttpEncodingAutoConfiguration {
  
  	//它已经和SpringBoot的配置文件映射
  	private final HttpEncodingProperties properties;
  
   //只有一个有参构造器的情况下，参数的值就会从容器中获取
  	public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
		this.properties = properties;
	}
  
    @Bean //给容器中添加一个组件，这个组件的某些值需要从properties中获取。
	@ConditionalOnMissingBean(CharacterEncodingFilter.class) //判断容器没有这个组件？
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
        // 配置文件书写内容，都是来自这些properties
        // spring.http.encoding.enabled=true
		// spring.http.encoding.charset=utf-8
		// spring.http.encoding.force=true
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
		return filter;
	}
```

根据当前不同的条件判断，决定这个配置类是否生效。一旦这个配置类生效，这个配置类就会给容器中添加各种组件，这些组件的属性是从对应的properties类中获取的，这些类里面的每一个属性又是和配置文件绑定的。

![](https://img-blog.csdnimg.cn/20201016235003799.png)

（4）所有在配置文件中能配置的属性都是在`xxxxProperties类`中封装的，配置文件能配置什么就可以参照某个功能对应的这个属性类。

```java
@ConfigurationProperties(prefix = "spring.http.encoding")  //从配置文件中获取指定的值和bean的属性进行绑定。
public class HttpEncodingProperties {

   public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");
}
```

==SpringBoot精髓==

* SpringBoot启动会加载大量的自动配置类

* 我们看我们需要的功能有没有SpringBoot默认写好的自动配置类

* 我们再来看这个自动配置类中到底配置了哪些组件（只要我们要用的组件有，我们就不需要再来配置了）

* 给容器中自动配置类添加组件的时候，会从properties类中获取某些属性，我们就可以在配置文件中指定这些属性的值

>  xxxxAutoConfigurartion：自动配置类，给容器中添加组件。
>
> xxxxProperties：封装配置文件中相关属性。



### 8.2 @Conditional

 @**Conditional**派生注解（Spring注解版原生的@Conditional作用）

作用：必须是@Conditional指定的条件成立，才给容器中添加组件，配置配里面的所有内容才生效。

| @Conditional扩展注解            | 作用（判断是否满足当前指定条件）                 |
| ------------------------------- | ------------------------------------------------ |
| @ConditionalOnJava              | 系统的java版本是否符合要求                       |
| @ConditionalOnBean              | 容器中存在指定Bean                               |
| @ConditionalOnMissingBean       | 容器中不存在指定Bean                             |
| @ConditionalOnExpression        | 满足SpEL表达式指定                               |
| @ConditionalOnClass             | 系统中有指定的类                                 |
| @ConditionalOnMissingClass      | 系统中没有指定的类                               |
| @ConditionalOnSingleCandidate   | 容器中只有一个指定的Bean，或者这个Bean是首选Bean |
| @ConditionalOnProperty          | 系统中指定的属性是否有指定的值                   |
| @ConditionalOnResource          | 类路径下是否存在指定资源文件                     |
| @ConditionalOnWebApplication    | 当前是web环境                                    |
| @ConditionalOnNotWebApplication | 当前不是web环境                                  |
| @ConditionalOnJndi              | JNDI存在指定项                                   |

**自动配置类必须在一定的条件下才能生效**，我们怎么知道哪些自动配置类生效呢?

我们在配置文件中启用 **debug=true**属性，来让控制台打印自动配置报告，这样我们就可以很方便的知道哪些自动配置类生效。

```java
=========================
AUTO-CONFIGURATION REPORT
=========================


Positive matches:（自动配置类启用的）
-----------------

   DispatcherServletAutoConfiguration matched:
      - @ConditionalOnClass found required class 'org.springframework.web.servlet.DispatcherServlet'; @ConditionalOnMissingClass did not find unwanted class (OnClassCondition)
      - @ConditionalOnWebApplication (required) found StandardServletEnvironment (OnWebApplicationCondition)
        
    
Negative matches:（没有启动，没有匹配成功的自动配置类）
-----------------

   ActiveMQAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required classes 'javax.jms.ConnectionFactory', 'org.apache.activemq.ActiveMQConnectionFactory' (OnClassCondition)

   AopAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required classes 'org.aspectj.lang.annotation.Aspect', 'org.aspectj.lang.reflect.Advice' (OnClassCondition)
        
```

## 参考

[视频](https://www.bilibili.com/video/BV1gW411W76m)

[配置加载来源](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-external-config)

[配置文件能配置全部属性](https://docs.spring.io/spring-boot/docs/2.4.0-SNAPSHOT/reference/html/appendix-application-properties.html#logging.file.clean-history-on-start)