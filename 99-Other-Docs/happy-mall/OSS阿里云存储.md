### 添加上传
和传统的单体应用不同，这里我们选择将数据上传到分布式文件服务器上。
这里我们选择将图片放置到阿里云上，使用对象存储。
阿里云上使使用对象存储方式：
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341338010-fa69b29e-a712-4c16-980e-620ee9f25322.png#align=left&display=inline&height=495&margin=%5Bobject%20Object%5D&originHeight=495&originWidth=1276&status=done&style=none&width=1276)
创建Bucket
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341338058-a030ed06-1b0c-40cf-94a8-dced1c9c108d.png#align=left&display=inline&height=845&margin=%5Bobject%20Object%5D&originHeight=845&originWidth=639&status=done&style=none&width=639)
上传文件：
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341338113-f9ee394c-8169-4b15-9282-bc89e089a201.png#align=left&display=inline&height=270&margin=%5Bobject%20Object%5D&originHeight=270&originWidth=1169&status=done&style=none&width=1169)
上传成功后，取得图片的URL
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341338160-6b4dd44f-1771-4c1c-a078-9cda1a7f38fb.png#align=left&display=inline&height=437&margin=%5Bobject%20Object%5D&originHeight=437&originWidth=1340&status=done&style=none&width=1340)
这种方式是手动上传图片，实际上我们可以在程序中设置自动上传图片到阿里云对象存储。
上传模型：
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341338207-790b0991-c495-41ca-9827-043e06e64630.png#align=left&display=inline&height=626&margin=%5Bobject%20Object%5D&originHeight=626&originWidth=1063&status=done&style=none&width=1063)
查看阿里云关于文件上传的帮助： [https://help.aliyun.com/document_detail/32009.html?spm=a2c4g.11186623.6.768.549d59aaWuZMGJ](https://help.aliyun.com/document_detail/32009.html?spm=a2c4g.11186623.6.768.549d59aaWuZMGJ)
#### 1）添加依赖包
在Maven项目中加入依赖项（推荐方式）
在 Maven 工程中使用 OSS Java SDK，只需在 pom.xml 中加入相应依赖即可。以 3.8.0 版本为例，在  内加入如下内容：
```xml
<dependency>
    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-sdk-oss</artifactId>
    <version>3.8.0</version>
</dependency>
```
#### 2）上传文件流
以下代码用于上传文件流：
```java
// Endpoint以杭州为例，其它Region请按实际情况填写。
String endpoint = "http://oss-cn-hangzhou.aliyuncs.com";
// 云账号AccessKey有所有API访问权限，建议遵循阿里云安全最佳实践，创建并使用RAM子账号进行API访问或日常运维，请登录 https://ram.console.aliyun.com 创建。
String accessKeyId = "<yourAccessKeyId>";
String accessKeySecret = "<yourAccessKeySecret>";
// 创建OSSClient实例。
OSS ossClient = new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);
// 上传文件流。
InputStream inputStream = new FileInputStream("<yourlocalFile>");
ossClient.putObject("<yourBucketName>", "<yourObjectName>", inputStream);
// 关闭OSSClient。
ossClient.shutdown();
```
endpoint的取值：
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341338264-88c9b94b-f5df-49f7-9500-ec4761cd96b0.png#align=left&display=inline&height=241&margin=%5Bobject%20Object%5D&originHeight=241&originWidth=1715&status=done&style=none&width=1715)
accessKeyId和accessKeySecret需要创建一个RAM账号：
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341338302-b174253f-772e-4869-908a-bc5acf42dd49.png#align=left&display=inline&height=431&margin=%5Bobject%20Object%5D&originHeight=431&originWidth=1062&status=done&style=none&width=1062)
创建用户完毕后，会得到一个“AccessKey ID”和“AccessKeySecret”，然后复制这两个值到代码的“AccessKey ID”和“AccessKeySecret”。
另外还需要添加访问控制权限：
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341338350-fb2fd07b-f1bd-467a-b29a-116149d77c0a.png#align=left&display=inline&height=356&margin=%5Bobject%20Object%5D&originHeight=356&originWidth=907&status=done&style=none&width=907)
```java
@Test
    public void testUpload() throws FileNotFoundException {
        // Endpoint以杭州为例，其它Region请按实际情况填写。
        String endpoint = "oss-cn-shanghai.aliyuncs.com";
        // 云账号AccessKey有所有API访问权限，建议遵循阿里云安全最佳实践，创建并使用RAM子账号进行API访问或日常运维，请登录 https://ram.console.aliyun.com 创建。
        String accessKeyId = "LTAI4G4W1RA4JXz2QhoDwHhi";
        String accessKeySecret = "R99lmDOJumF2x43ZBKT259Qpe70Oxw";
        // 创建OSSClient实例。
        OSS ossClient = new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);
        // 上传文件流。
        InputStream inputStream = new FileInputStream("C:\\Users\\Administrator\\Pictures\\timg.jpg");
        ossClient.putObject("mall-images", "time.jpg", inputStream);
        // 关闭OSSClient。
        ossClient.shutdown();
        System.out.println("上传成功.");
    }
```
更为简单的使用方式，是使用SpringCloud Alibaba
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341338415-600fabd3-ea74-4dca-8d00-cd16053cd0e6.png#align=left&display=inline&height=664&margin=%5Bobject%20Object%5D&originHeight=664&originWidth=1303&status=done&style=none&width=1303)
详细使用方法，见： [https://help.aliyun.com/knowledge_detail/108650.html](https:_help.aliyun.com_knowledge_detail_108650)
（1）添加依赖
```xml
<dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alicloud-oss</artifactId>
            <version>2.2.0.RELEASE</version>
        </dependency>
```
（2）创建“AccessKey ID”和“AccessKeySecret”
（3）配置key，secret和endpoint相关信息
```yaml
access-key: LTAI4G4W1RA4JXz2QhoDwHhi
      secret-key: R99lmDOJumF2x43ZBKT259Qpe70Oxw
      oss:
        endpoint: oss-cn-shanghai.aliyuncs.com
```
（4）注入OSSClient并进行文件上传下载等操作
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341338521-bf752315-b0fc-4722-8681-84c7ac1336e0.png#align=left&display=inline&height=405&margin=%5Bobject%20Object%5D&originHeight=405&originWidth=1264&status=done&style=none&width=1264)
但是这样来做还是比较麻烦，如果以后的上传任务都交给mall-product来完成，显然耦合度高。最好单独新建一个Module来完成文件上传任务。
### 其他方式
#### 1）新建mall-third-party
#### 2）添加依赖，将原来mall-common中的“spring-cloud-starter-alicloud-oss”依赖移动到该项目中
```xml
<dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alicloud-oss</artifactId>
            <version>2.2.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>com.bigdata.mall</groupId>
            <artifactId>jdmall-common</artifactId>
            <version>1.0-SNAPSHOT</version>
            <exclusions>
                <exclusion>
                    <groupId>com.baomidou</groupId>
                    <artifactId>mybatis-plus-boot-starter</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```
另外也需要在“pom.xml”文件中，添加如下的依赖管理
```xml
<dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>2.2.1.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```
#### 3）在主启动类中开启服务的注册和发现
```java
@EnableDiscoveryClient
```
#### 4）在nacos中注册
（1）创建命名空间“ third-party ”
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341338572-9975d2d4-62cb-4856-8a9e-43d75da5f085.png#align=left&display=inline&height=375&margin=%5Bobject%20Object%5D&originHeight=375&originWidth=942&status=done&style=none&width=942)
（2）在“ third-party”命名空间中，创建“ oss.yml”文件
```yaml
spring:
  cloud:
    alicloud:
      access-key: LTAI4G4W1RA4JXz2QhoDwHhi
      secret-key: R99lmDOJumF2x43ZBKT259Qpe70Oxw
      oss:
        endpoint: oss-cn-shanghai.aliyuncs.com
```
#### 5）编写配置文件
application.yml
```yaml
server:
  port: 30000
spring:
  application:
    name: mall-third-party
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.137.14:8848
logging:
  level:
    com.bigdata.mall.product: debug
```
bootstrap.properties
```properties
spring.cloud.nacos.config.name=jdmall-third-party
spring.cloud.nacos.config.server-addr=192.168.137.14:8848
spring.cloud.nacos.config.namespace=f995d8ee-c53a-4d29-8316-a1ef54775e00
spring.cloud.nacos.config.extension-configs[0].data-id=jdmall-third-party.yml
spring.cloud.nacos.config.extension-configs[0].group=DEFAULT_GROUP
spring.cloud.nacos.config.extension-configs[0].refresh=true
```
#### 6） 编写测试类
```java
package com.bigdata.mall.thirdparty;
import com.aliyun.oss.OSS;
import com.aliyun.oss.OSSClient;
import com.aliyun.oss.OSSClientBuilder;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.InputStream;
@SpringBootTest
class jdmallThirdPartyApplicationTests {
    @Autowired
    OSSClient ossClient;
    @Test
    public void testUpload() throws FileNotFoundException {
        // Endpoint以杭州为例，其它Region请按实际情况填写。
        String endpoint = "oss-cn-shanghai.aliyuncs.com";
        // 云账号AccessKey有所有API访问权限，建议遵循阿里云安全最佳实践，创建并使用RAM子账号进行API访问或日常运维，请登录 https://ram.console.aliyun.com 创建。
        String accessKeyId = "LTAI4G4W1RA4JXz2QhoDwHhi";
        String accessKeySecret = "R99lmDOJumF2x43ZBKT259Qpe70Oxw";
        // 创建OSSClient实例。
        OSS ossClient = new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);
         //上传文件流。
        InputStream inputStream = new FileInputStream("C:\\Users\\Administrator\\Pictures\\timg.jpg");
        ossClient.putObject("mall-images", "time3.jpg", inputStream);
        // 关闭OSSClient。
        ossClient.shutdown();
        System.out.println("上传成功.");
    }
}
```
[https://help.aliyun.com/document_detail/31926.html?spm=a2c4g.11186623.6.1527.228d74b8V6IZuT](https://help.aliyun.com/document_detail/31926.html?spm=a2c4g.11186623.6.1527.228d74b8V6IZuT)
**背景**
采用JavaScript客户端直接签名（参见[JavaScript客户端签名直传](https://help.aliyun.com/document_detail/31925.html#concept-frd-4gy-5db)）时，AccessKeyID和AcessKeySecret会暴露在前端页面，因此存在严重的安全隐患。因此，OSS提供了服务端签名后直传的方案。
**原理介绍**
[![](https://cdn.nlark.com/yuque/0/2020/png/1033455/1592614256788-be8554d9-1db3-4916-ad64-879b65888125.png#align=left&display=inline&height=323&margin=%5Bobject%20Object%5D&originHeight=323&originWidth=692&size=0&status=done&style=none&width=692)](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/6875011751/p1472.png)
服务端签名后直传的原理如下：

1. 用户发送上传Policy请求到应用服务器。
1. 应用服务器返回上传Policy和签名给用户。
1. 用户直接上传数据到OSS。

编写“com.bigdata.mall.thirdparty.controller.OssController”类：
```java
package com.bigdata.mall.thirdparty.controller;
import com.aliyun.oss.OSS;
import com.aliyun.oss.common.utils.BinaryUtil;
import com.aliyun.oss.model.MatchMode;
import com.aliyun.oss.model.PolicyConditions;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.LinkedHashMap;
import java.util.Map;
@RestController
public class OssController {
    @Autowired
    OSS ossClient;
    @Value ("${spring.cloud.alicloud.oss.endpoint}")
    String endpoint ;
    @Value("${spring.cloud.alicloud.oss.bucket}")
    String bucket ;
    @Value("${spring.cloud.alicloud.access-key}")
    String accessId ;
    @Value("${spring.cloud.alicloud.secret-key}")
    String accessKey ;
    @RequestMapping("/oss/policy")
    public Map<String, String> policy(){
        String host = "https://" + bucket + "." + endpoint; // host的格式为 bucketname.endpoint
        String format = new SimpleDateFormat("yyyy-MM-dd").format(new Date());
        String dir = format; // 用户上传文件时指定的前缀。
        Map<String, String> respMap=null;
        try {
            long expireTime = 30;
            long expireEndTime = System.currentTimeMillis() + expireTime * 1000;
            Date expiration = new Date(expireEndTime);
            PolicyConditions policyConds = new PolicyConditions();
            policyConds.addConditionItem(PolicyConditions.COND_CONTENT_LENGTH_RANGE, 0, 1048576000);
            policyConds.addConditionItem(MatchMode.StartWith, PolicyConditions.COND_KEY, dir);
            String postPolicy = ossClient.generatePostPolicy(expiration, policyConds);
            byte[] binaryData = postPolicy.getBytes("utf-8");
            String encodedPolicy = BinaryUtil.toBase64String(binaryData);
            String postSignature = ossClient.calculatePostSignature(postPolicy);
            respMap= new LinkedHashMap<String, String>();
            respMap.put("accessid", accessId);
            respMap.put("policy", encodedPolicy);
            respMap.put("signature", postSignature);
            respMap.put("dir", dir);
            respMap.put("host", host);
            respMap.put("expire", String.valueOf(expireEndTime / 1000));
        } catch (Exception e) {
            // Assert.fail(e.getMessage());
            System.out.println(e.getMessage());
        } finally {
            ossClient.shutdown();
        }
        return respMap;
    }
}
```
测试： [http://localhost:30000/oss/policy](http://localhost:30000/oss/policy)
```
{"accessid":"LTAI4G4W1RA4JXz2QhoDwHhi","policy":"eyJleHBpcmF0aW9uIjoiMjAyMC0wNC0yOVQwMjo1ODowNy41NzhaIiwiY29uZGl0aW9ucyI6W1siY29udGVudC1sZW5ndGgtcmFuZ2UiLDAsMTA0ODU3NjAwMF0sWyJzdGFydHMtd2l0aCIsIiRrZXkiLCIyMDIwLTA0LTI5LyJdXX0=","signature":"s42iRxtxGFmHyG40StM3d9vOfFk=","dir":"2020-04-29/","host":"https://mall-images.oss-cn-shanghai.aliyuncs.com","expire":"1588129087"}
```
以后在上传文件时的访问路径为“ http://localhost:88/api/thirdparty/oss/policy”，
在“mall-gateway”中配置路由规则：
```yaml
- id: third_party_route
          uri: lb://mall-gateway
          predicates:
            - Path=/api/thirdparty/**
          filters:
            - RewritePath=/api/thirdparty/(?<segment>/?.*),/$\{segment}
```
测试是否能够正常跳转： [http://localhost:88/api/thirdparty/oss/policy](http://localhost:88/api/thirdparty/oss/policy)
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341338630-6b7278b4-25f1-4e0c-a7f1-5cec9e453c62.png#align=left&display=inline&height=324&margin=%5Bobject%20Object%5D&originHeight=324&originWidth=1112&status=done&style=none&width=1112)
### 上传组件
放置项目提供的upload文件夹到components目录下，一个是单文件上传，另外一个是多文件上传
```shell
PS D:\Project\jdmall\renren-fast-vue\src\components\upload> ls
    目录: D:\Project\jdmall\renren-fast-vue\src\components\upload
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----  2020/4/29 星期三     12:0           3122 multiUpload.vue
                                2
-a----  2019/11/11 星期一     21:            343 policy.js
                               20
-a----  2020/4/29 星期三     12:0           3053 singleUpload.vue
                                1
PS D:\Project\jdmall\renren-fast-vue\src\components\upload>
```
修改这两个文件的配置后
开始执行上传，但是在上传过程中，出现了如下的问题：
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341338677-4f3196df-ae7e-4b30-88c2-cbe46c4632e1.png#align=left&display=inline&height=72&margin=%5Bobject%20Object%5D&originHeight=72&originWidth=1729&status=done&style=none&width=1729)
```
Access to XMLHttpRequest at 'http://mall-images.oss-cn-shanghai.aliyuncs.com/' from origin 'http://localhost:8001' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```
这又是一个跨域的问题，解决方法就是在阿里云上开启跨域访问：
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341338714-c99d4d6f-413b-4c1e-9897-4ab65195f98e.png#align=left&display=inline&height=616&margin=%5Bobject%20Object%5D&originHeight=616&originWidth=1861&status=done&style=none&width=1861)
再次执行文件上传。