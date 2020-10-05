## 一、Java程序运行过程

### 1.1 Java运行过程

Java 程序的运行必须经过**编写**、**编译**和**运行** 3 个步骤。

1）编写：是指在 Java 开发环境中进行程序代码的输入，最终形成后缀名为` .java `的 Java 源文件。

2）编译：是指使用 Java 编译器（`javac.exe`）对源文件进行错误排査的过程，编译后将生成后缀名为` .class `的字节码文件。

3）运行：是指使用 Java 解释器（`java.exe`）将字节码文件翻译成机器代码，执行并显示结果。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200805155428223.jpg)

### 1.2 JDK、JRE和JVM区别和联系

**（1）JDK**（Java Development Kid，Java 开发开源工具包），是整个 **Java 的核心**，包括了 Java 运行环境 JRE、Java 工具（javac/java/jdb等）和 Java 基础类库（即Java API 包括rt.jar）。
JDK的主要作用的5个文件：bin、include、lib、 jre、db。

* bin：一堆exe文件，可执行的开发工具， 例如：**javac**、java、javadoc、native2ascii。
* include：Java 和 JVM交互用的头文件。
* lib：包含dt.jar + tools.jar 的常用类库，开发依赖包。
* jre：java运行环境，包括JVM+Java基础&核心类库
* db：jdk从1.6之后内置的Derby数据库，是一个纯用Java实现的内存数据库，属于Apache的一个开源项目，体积小，免安装，只需要几个小jar包就可以跨平台运行。



**（2）JRE**（Java Runtime Environment，Java 运行环境）是运行JAVA 程序所必须的环境的集合，包含 JVM 标准实现及 Java 核心类库。JRE 1.8目录包含：

- bin：有**java.exe（解释器）**但没有javac.exe（编译器），无法编译Java程序，但可以运行Java程序，可以把这个bin目录理解成JVM。

- lib：Java基础&核心类库，如rt.jar，也包含JVM运行时需要的类库。

  

（3）**JVM**（Java Virtual Machine，Java 虚拟机）是整个 Java 实现跨平台的**最核心**的部分，能够运行以 Java 语言写作的软件程序。

JVM主要负责运行Java编译器编译后的字节码文件（*.class文件）。JVM在执行字节码时，把字节码解释成具体平台上的机器码执行。JVM自己无法执行，必须要联合JRE中的Java基础&核心类库(特别是rt.jar)才能使用。

总的来看，**<font color="red"  size="4">JDK=JRE+多种Java开发工具；JRE=JVM+各种类库。</font>**

## 二、