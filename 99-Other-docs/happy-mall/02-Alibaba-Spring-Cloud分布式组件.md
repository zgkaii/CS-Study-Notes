## 一、Nacos注册中心（Nacos Discovery）
1、首先，修改 pom.xml 文件，引入 Nacos Discovery Starter。

``` yml
<dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
```
2、在应用的 /src/main/resources/application.yml 配置文件中配置 Nacos Server 地址与服务模块名

```properties
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848 #Nacos Server 地址
  application:
    name: mall-coupon #注册中心 模块服务名
```
3、使用 @EnableDiscoveryClient 注解开启服务注册与发现功能

```java
@EnableDiscoveryClient
@SpringBootApplication //开启服务发现客户端
public class MallCouponApplication {

    public static void main(String[] args) {
        SpringApplication.run(MallCouponApplication.class, args);
    }
}
```
同上，在mall-member中也注解开启服务注册与发现功能。

启动[nacos](https://github.com/alibaba/nacos)，访问[http://127.0.0.1:8848/nacos/](http://127.0.0.1:8848/nacos/)，在**服务管理-服务列表**可以查看所有运行的实例。

![](https://img-blog.csdnimg.cn/20201220232048109.png)

 ## 二、OpenFeign远程调用
 以member会员服务远程调用coupon仓储服务为例：  
 1、member服务模块中引入open-feign
 ```yml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
 ```
2、编写一个接口，告诉SpringCLoud这个接口需要调用远程服务
mall-coupon中修改“com\happy\mall\coupon\controller\CouponController.java”，添加以下controller方法：

```yml
@RequestMapping("/member/list")
public R memberCoupons(){
    CouponEntity couponEntity = new CouponEntity();
    couponEntity.setCouponName("discount 20%");
    return R.ok().put("coupons",Arrays.asList(couponEntity));
}
```
mall-member中新建“com\happy\mall\member\feign\CouponFeignService.java”接口
```java
@FeignClient("mall-coupon")//告诉SpringCloud这个接口需要调用远程服务
public interface CouponFeignService {
    @RequestMapping("/coupon/coupon/member/list")//声明接口的方法 调用哪个远程服务的哪个请求
    public R memberCoupons();
}
```
3、开启远程调用功能
mall-member中修改“com\happy\mall\member\MallMemberApplication.java”类，添加上"@EnableFeignClients"：

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients(basePackages = "com.happy.mall.member.feign")
public class MallMemberApplication {

    public static void main(String[] args) {
        SpringApplication.run(MallMemberApplication.class, args);
    }
}
```
mall-member中于com\happy\mall\member\controller\MemberController.java编写测试方法
```yml
    @Autowired
    CouponFeignService couponFeignService;

    @RequestMapping("/coupons")
    public R test(){
        MemberEntity memberEntity=new MemberEntity();
        memberEntity.setNickname("张三");
        R memberCoupons = couponFeignService.memberCoupons();
        return memberCoupons.put("member",memberEntity).put("coupons",memberCoupons.get("coupons"));
    }
```
4、查看  
访问[http://localhost:8000/member/member/coupons](http://localhost:8000/member/member/coupons)
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341334279-1282e3ff-1786-4648-8dff-8dd7672bb341.png#align=left&display=inline&height=200&margin=%5Bobject%20Object%5D&originHeight=200&originWidth=1212&status=done&style=none&width=1212)
停止“mall-coupon”服务，能够看到注册中心显示该服务的健康值为0：
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341334391-bcd0cc89-fcfc-4640-923f-d97ecaf454ba.png#align=left&display=inline&height=415&margin=%5Bobject%20Object%5D&originHeight=415&originWidth=1409&status=done&style=none&width=1409)
再次访问：[http://localhost:8000/member/member/coupons](http://localhost:8000/member/member/coupons)
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341334471-334ddb93-08aa-4d86-ab4e-22619d03f367.png#align=left&display=inline&height=266&margin=%5Bobject%20Object%5D&originHeight=266&originWidth=1098&status=done&style=none&width=1098)
启动“mall-coupon”服务，再次访问，又恢复了正常。

## 三、配置中心（Nacos Config）
[Nacos三种配置加载方方案](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-docs/src/main/asciidoc-zh/nacos-config.adoc)

### 1、读取本地配置文件
（1）首先，修改 pom.xml 文件，引入 Nacos Config Starter。
```yaml
 <dependency>
     <groupId>com.alibaba.cloud</groupId>
     <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
 </dependency>
```
（2）在应用的 /src/main/resources/bootstrap.properties 配置文件中配置 Nacos Config 元数据；该配置文件会优先于“application.yml”加载。
```yaml
 spring.application.name=mall-coupon
 spring.cloud.nacos.config.server-addr=127.0.0.1:8848
```
（3）创建“application.properties”配置文件，添加如下配置内容：
```properties
coupon.user.name="张三"
coupon.user.age=20
```
（4）从 Nacos Config 中获取相应的配置，并添加在 Spring Environment 的 PropertySources 中。这里我们使用 @Value 注解来将对应的配置注入到 SampleController 的 userName 和 age 字段，并添加 @RefreshScope 打开动态刷新功能
 ```yml
    @Value("${coupon.user.name}")
    private String name;
    @Value("${coupon.user.age}")
    private Integer age;

    @RequestMapping("/test")
    public R test(){
        return R.ok().put("name",name).put("age",age);
    }
 ```
启动“mall-coupon”服务：
访问：[http://localhost:7000/coupon/coupon/test](http://localhost:7000/coupon/coupon/test)
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341334525-d99419de-8f9e-4fc4-ba71-f976e1b6cff7.png#align=left&display=inline&height=107&margin=%5Bobject%20Object%5D&originHeight=107&originWidth=1075&status=done&style=none&width=1075)
这样做存在的一个问题，如果频繁的修改application.properties，就需要频繁重新打包部署。下面我们将采用Nacos的配置中心来解决这个问题。
### 2、读取nacos配置文件
 （1）在nacos配置管理 | 配置列表 创建mall-coupon.properties
 ![](https://img-blog.csdnimg.cn/20200729210220485.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0tBSVpfTEVBUk4=,size_16,color_FFFFFF,t_70)
 （2）添加刷新注解
 ```java
 @RefreshScope
 public class CouponController {
    
 }
 
 ```
 （3）启动微服务mall-coupon，访问http://localhost:7000/coupon/coupon/test
 ```yaml
 {
     "msg":"success",
     "code":0,
     "name":"zhangsan2",
     "age":20
 }
 
 ```
 能够看到读取到了nacos 中的最新的配置信息，并且在指明了相同的配置信息时，配置中心中设置的值优先于本地配置。
 ### 3、Namespace方案
（1）命名空间：配置隔离
 * 开发dev，测试test，生产prod：利用命名空间来做环境隔离。
    注意：在bootstrap.properties配置上，需要配置使用命名空间下的ID。
 * 每一个微服务之间互相隔离配置，每一个微服务都创建自己的命名空间，只加载自己命名空间下的所有配置。

（2）配置集：所有的配置的集合
 例如一个application.yml文件就是一个配置集。

（3）配置集ID：类似文件名，如application.yml，在Nacos中，即是Data ID。  
 ![](https://img-blog.csdnimg.cn/20200729212315870.png)

（4）配置分组：默认所有的配置集都属于：DEFAULT_GROUP。
 项目中的使用：每个微服务创建自己的命名空间，使用配置分组区分环境，dev，test，prod。

 5、同时加载多个配置集
 在日常开发中，不会把所有配置写在一个配置集中（如以下application.yml）。
 ```yaml
 spring:
   datasource:
     username: root
     password: root
     url: jdbc:mysql://192.168.56.10:3306/jdmall_sms
     driver-class-name: com.mysql.jdbc.Driver
   cloud:
     nacos:
       discovery:
         server-addr: 127.0.0.1:8848
   application:
     name: mall-coupon
 
 mybatis-plus:
   mapper-locations: classpath:/mapper/**/*.xml
   global-config:
     db-config:
       id-type: auto
 server:
   port: 7000
 ```
 会根据需求拆分成
 * datasource.yml（数据源），分组设置为dev
 ```yaml
 spring:
   datasource:
     username: root
     password: root
     url: jdbc:mysql://192.168.56.10:3306/jdmall_sms
     driver-class-name: com.mysql.jdbc.Driver
 ```
 * mybatis.yml，分组设置为dev
 ```yaml
 mybatis-plus:
   mapper-locations: classpath:/mapper/**/*.xml
   global-config:
     db-config:
       id-type: auto
 ```
 * other.yml，分组设置为dev
 ```yaml
 spring:
   cloud:
     nacos:
       discovery:
         server-addr: 127.0.0.1:8848
   application:
     name: mall-coupon
 server:
   port: 7000
 ```
 6、修改bootstrap.properties
 ```yaml
 spring.application.name=mall-coupon
 
 spring.cloud.nacos.config.server-addr=127.0.0.1:8848
 spring.cloud.nacos.config.namespace=a04ac132-4a74-45bf-bc60-a749d4c140c6
 #spring.cloud.nacos.config.group=dev
 
 spring.cloud.nacos.config.ext-config[0].data-id=datasource.yml
 spring.cloud.nacos.config.ext-config[0].group=dev
 #动态刷新
 spring.cloud.nacos.config.ext-config[0].refresh=true
 
 spring.cloud.nacos.config.ext-config[1].data-id=mybatis.yml
 spring.cloud.nacos.config.ext-config[1].group=dev
 spring.cloud.nacos.config.ext-config[1].refresh=true
 
 spring.cloud.nacos.config.ext-config[2].data-id=other.yml
 spring.cloud.nacos.config.ext-config[2].group=dev
 spring.cloud.nacos.config.ext-config[2].refresh=true
 ```
![](https://img-blog.csdnimg.cn/20200729214714288.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0tBSVpfTEVBUk4=,size_16,color_FFFFFF,t_70)

## 四、GatewayAPI网关
1、使用Spring Intitializr创建模块
* Project Metadata 
  * Group: com.happy.mall
  * Artifact: mall-gateway
  * Package: com.happy.mall.gateway
* Dependencies添加依赖 
  * Spring Cloud Routing-Gateway

2、添加依赖
```yaml
<dependency>
    <groupId>com.happy.mall</groupId>
    <artifactId>mall-common</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>

```
排除数据源相关配置：
```java
@EnableDiscoveryClient
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class MallGatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(MallGatewayApplication.class, args);
    }
}
```
3、开启服务注册发现  
（1）配置nacos的注册中心地址(application.properties)
```yaml
spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
spring.application.name=mall-gateway
server.port=88
```
(2)nacos配置中心
创建gateway命名空间，创建bootstrap.properties文件
```yaml
spring.application.name=mall-coupon

spring.cloud.nacos.config.server-addr=127.0.0.1:8848
spring.cloud.nacos.config.namespace=c7cb33d9-2a43-47b2-9c8a-eb32955f605a

```
![](https://img-blog.csdnimg.cn/20200729224116766.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0tBSVpfTEVBUk4=,size_16,color_FFFFFF,t_70)

4、编写网关配置文件（application.yml）
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: test_route
          uri: https://www.baidu.com
          predicates:
            - Query=url,baidu

        - id: qq_route
          uri: https://www.qq.com
          predicates:
            - Query=url,qq
```
5、启动mall-gateway
```java
@EnableDiscoveryClient
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class MallGatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(MallGatewayApplication.class, args);
    }
}
```

* 访问http://localhost:88/hello?url=baidu，  跳转至百度
* 问http://localhost:88/hello?url=qq，  跳转至QQ


