> 参考：《SpringBoot实战》

## 安装 Spring Boot CLI

- 下载 [spring-boot-cli-2.4.0](https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.4.0/)
- 解压到任意目录
- 将 bin 目录添加到环境变量中
- 查看安装结果

```
#输入
spring --version
#输出
Spring CLI v2.4.0
```

## 使用Spring Initializr 初始化 Spring Boot 项目

**Spring Initializr 的几种用法**

- 通过Web 界面使用
- 通过Spring Tool Suite 使用
- 通过 Intellij IDEA 使用
- 通过 Spring Boot CLI 使用

### 使用 Spring Boot CLI 初始化 Spring Boot

```
spring init
```

- init 命令会生成一个 demo.zip 文件。
- 解压后会看到一个典型的项目结构，包含一个Maven 的pom.xml构建描述文件。

**构建一个Web应用，使用JPA实现数据持久化，使用Spring Security 进行安全加固。**

- 使用--dependencies 或 -d来指定初始依赖

```
# 使用Maven 在pom.xml里增加了Spring Boot的Web、jpa和security起步依赖
spring init -dweb,jpa,security
# 使用Gradle
spring init -dweb,jpa,security --build gradle

# 官网手册 构建web应用，并使用data-jpa，生成文件夹myapp
  spring init --dependencies=web,data-jpa myapp
& spring init --dweb,data-jpa myapp
```

> 注意：-d和依赖之间没有空格，否则就变成 了下载一个ZIP文件，文件名是web,data-jpa。

### spring init 相关命令

- -d 或 --dependencies，指定依赖

```
spring init -dweb,jpa,security
```

- -p 或--packaging, 指定构建的包

```
spring init -dweb,jpa,security --build gradle -p war
```

- -x 或 --extract ,指定用于生成项目的解压目录

```
spring init -dweb,jpa,security --build gradle -p jar -x
```

- help 查看有哪些可选参数

```
spring help init
```

- -l 或 --list ，查看支持Initializr 支持的参数

```
spring init --list
```