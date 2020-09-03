##### 1.创建Spring工程
```
GroupId:com.kai
ArtifacttId:001_helloworld
package:com.kai.demo
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
```shell
C:\WorkSpace\IdeaProjects\spring-learn>curl http://localhost:8080/hello
Hello World!
C:\WorkSpace\IdeaProjects\spring-learn>curl http://localhost:8080/actuator/health
{"status":"UP"}
```
- maven打包并测试
```shell script
C:\WorkSpace\IdeaProjects\spring-learn>cd 001-helloworld  (切换到pom.xml所在目录）

C:\WorkSpace\IdeaProjects\spring-learn\001-helloworld>mvn clean package -Dmaven.test.skip
[INFO] Scanning for projects...
[INFO]
[INFO] -----------------------< com.kai:001-helloworld >-----------------------
[INFO] Building 001-helloworld 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
       ---------------------------------snip-----------------------------------
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  11.092 s
[INFO] Finished at: 2020-09-03T16:57:46+08:00
[INFO] ------------------------------------------------------------------------

C:\WorkSpace\IdeaProjects\spring-learn\001-helloworld>cd target

C:\WorkSpace\IdeaProjects\spring-learn\001-helloworld\target>dir
 ドライブ C のボリューム ラベルは Windows です
 ボリューム シリアル番号は F64A-4518 です

 C:\WorkSpace\IdeaProjects\spring-learn\001-helloworld\target のディレクトリ

2020/09/03  17:00    <DIR>          .
2020/09/03  17:00    <DIR>          ..
2020/09/03  17:00        18,420,367 001-helloworld-0.0.1-SNAPSHOT.jar
2020/09/03  17:00             2,852 001-helloworld-0.0.1-SNAPSHOT.jar.original
2020/09/03  17:00    <DIR>          classes
2020/09/03  17:00    <DIR>          generated-sources
2020/09/03  17:00    <DIR>          maven-archiver
2020/09/03  17:00    <DIR>          maven-status
               2 個のファイル          18,423,219 バイト
               6 個のディレクトリ  29,920,378,880 バイトの空き領域

C:\WorkSpace\IdeaProjects\spring-learn\001-helloworld\target>java -jar 001-helloworld-0.0.1-SNAPSHOT.jar
```
打开Google，访问http://localhost:8080/hello

![helloworld](https://img-blog.csdnimg.cn/20200903171001190.png#pic_center)