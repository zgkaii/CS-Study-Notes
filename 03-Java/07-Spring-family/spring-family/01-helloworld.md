##### 1.创建Spring工程
```
GroupId:org.spring
ArtifacttId:001_helloworld
package:org.spring.learn
dependencies:Spring Web、Spring Actuator
```
##### 2.创建测试方法hello()
```java
@SpringBootApplication
@RestController
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @RequestMapping("/hello")
    public String hello() {
        return "Hello World!";
    }
}
```
##### 3.测试

- 运行Application
- 打开本地Terminal
  * 可以根据查看 Actuator监控（[Spring Boot Actuator监控使用详解](https://juejin.cn/post/6844904000832159751)）

```shell
C:\WorkSpace\IdeaProjects\spring-family>curl http://localhost:8080/hello
Hello World!
C:\WorkSpace\IdeaProjects\spring-family>curl http://localhost:8080/actuator/health
{"status":"UP"}
```
- maven打包并测试
```shell script
C:\WorkSpace\IdeaProjects\spring-family>cd 001-helloworld  (切换到pom.xml所在目录）

C:\WorkSpace\IdeaProjects\spring-family\001-helloworld>mvn clean package -Dmaven.test.skip
[INFO] Scanning for projects...
[INFO]
[INFO] -----------------------< org.spring:001-helloworld >-----------------------
[INFO] Building 001-helloworld 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
       ---------------------------------snip-----------------------------------
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  11.092 s
[INFO] Finished at: 2020-09-03T16:57:46+08:00
[INFO] ------------------------------------------------------------------------

C:\WorkSpace\IdeaProjects\spring-family\001-helloworld>cd target

C:\WorkSpace\IdeaProjects\spring-family\001-helloworld\target>dir
 驱动器 D 中的卷是 Data
 卷的序列号是 B8A9-CEA2

 C:\WorkSpace\IdeaProjects\spring-family\001-helloworld\target 目录

2020/09/03  20:35    <DIR>          .
2020/09/03  20:35    <DIR>          ..
2020/09/03  20:35        18,420,374 001-helloworld-0.0.1-SNAPSHOT.jar
2020/09/03  20:35             2,859 001-helloworld-0.0.1-SNAPSHOT.jar.original
2020/09/03  20:35    <DIR>          classes
2020/09/03  20:35    <DIR>          generated-sources
2020/09/03  20:35    <DIR>          maven-archiver
2020/09/03  20:35    <DIR>          maven-status
               2 个文件     18,423,233 字节
               6 个目录 298,277,195,776 可用字节

C:\WorkSpace\IdeaProjects\spring-family\001-helloworld\target>java -jar 001-helloworld-0.0.1-SNAPSHOT.jar
```
打开Google，访问http://localhost:8080/hello

![helloworld](https://img-blog.csdnimg.cn/20200903171001190.png)