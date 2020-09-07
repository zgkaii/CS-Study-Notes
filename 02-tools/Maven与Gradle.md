##### 1、下载解压
点击[下载地址](https://maven.apache.org/download.cgi)， 下载 apache-maven-3.6.3-bin.zip 并解压。

##### 2、配置环境变量
环境变量添加：MAVEN_HOME
变量值设为：C:\apache\apache-maven-3.6.3

PATH中添加：%MAVEN_HOME%\bin
##### 3、验证
CMD中输入：mvn -version
```shell
C:\Users\XXX>mvn -version
Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
Maven home: C:\apache\apache-maven-3.6.3\bin\..
Java version: 1.8.0_151, vendor: Oracle Corporation, runtime: C:\Program Files\Java\jdk1.8.0_151\jre
Default locale: ja_JP, platform encoding: MS932
OS name: "windows 10", version: "10.0", arch: "amd64", family: "windows"
```
##### 4、配置settings文件找到conf文件夹中的settings.xml文件
（1）添加本地仓库
```shell
<localRepository>C:\My_workspace\my_maven_local_repository</localRepository>
```
（2）添加镜像仓库
```shell
 <mirrors>
 <mirror>  
  <id>alimaven</id>  
        <name>aliyun maven</name>  
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>  
        <mirrorOf>central</mirrorOf>  
 </mirror>
 </mirrors>
```
（3）配置JDK
```shell
 <profile>
        <id>jdk-1.8</id>
        <activation>
            <activeByDefault>true</activeByDefault>
            <jdk>1.8</jdk>
        </activation>
        <properties>
            <maven.compiler.source>1.8</maven.compiler.source>
            <maven.compiler.target>1.8</maven.compiler.target>
            <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
        </properties>
  </profile>
```
##### 5、IDEA中Maven的设置
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200728164250370.png)

配置完成。
