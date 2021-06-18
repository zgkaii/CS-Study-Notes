
<!-- MarkdownTOC -->
- [JVM常用启动参数](#jvm常用启动参数)
  - [系统属性参数](#系统属性参数)
  - [运行模式参数](#运行模式参数)
  - [堆内存设置参数](#堆内存设置参数)
    - [显式指定堆内存`–Xms`和`-Xmx`](#显式指定堆内存xms和-xmx)
    - [显式新生代内存](#显式新生代内存)
    - [显示指定永久代/元空间的大小](#显示指定永久代元空间的大小)
    - [其他常用参数](#其他常用参数)
  - [GC设置参数](#gc设置参数)
    - [垃圾回收器](#垃圾回收器)
    - [GC记录](#gc记录)
  - [分析诊断参数](#分析诊断参数)
  - [JavaAgent参数](#javaagent参数)

<!-- /MarkdownTOC -->

# JVM常用启动参数

JVM启动参数的前缀主要有`-`、`-D`、`-X`、`-XX`、`+/-`、`:`

* 以`-`开头为标准参数，所以的JVM都要实现这些参数，并且向后兼容。例如`-server`。

* `-D`设置系统属性。例如`-Dfile.encoding=UTF-8`。

* 以`-X`开头为非标准参数，基本都传给JVM，默认 JVM 实现这些参数的功能，但是并不保证所 有 JVM 实现都满足，且不保证向后兼容。 可以使 用 java -X 命令来查看当前 JVM 支持的非标准参数。例如`-Xmx8g`。

* 以`-XX`开头的为非稳定参数，专门用于控制JVM的行为，跟具体的JVM实现有关，随时可能在下一个版本中取消。

* `-XX：+/-Flags` 形式，`+/- `是对布尔值进行开关。例如`-XX:+UseG1GC`。

* `-XX：key=value `形式， 指定某个选项的值。例如`-XX:MaxPermSize=256m`。

根据JVM启动参数的的特点与作用来分类，大致如下几种：

* 系统属性参数
* 运行模式参数
* 堆内存设置参数
* GC设置参数
* 分析诊断参数
* `JavaAgent`参数

## 系统属性参数

```java
-Dfile.encoding=UTF-8
-Duser.timezone=GMT+08
-Dmaven.test.skip=true
-Dio.netty.eventLoopThreads=8
... ...
```

> JVM里配置的环境变量只影响当前一个进程。

## 运行模式参数

* `-server`：设置 JVM 使用 server 模式，特点是启动速度比较慢，但运行时性能和内存管理效率 很高，适用于生产环境。在具有 64 位能力的 JDK 环境下将默认启用该模式，而忽略 -client 参 数。
* `-client` ：JDK1.7 之前在32位的 x86 机器上的默认值是 -client 选项。设置 JVM 使用 client 模 式，特点是启动速度比较快，但运行时性能和内存管理效率不高，通常用于客户端应用程序或 者 PC 应用开发和调试。此外，我们知道 JVM 加载字节码后，可以解释执行，也可以编译成本 地代码再执行，所以可以配置 JVM 对字节码的处理模式。
* `-Xint`：在解释模式（interpreted mode）下运行，`-Xint` 标记会强制 JVM 解释执行所有的字节 码，这当然会降低运行速度，通常低10倍或更多。 
*  `-Xcomp`：`-Xcomp` 参数与`-Xint` 正好相反，JVM 在第一次使用时会把所有的字节码编译成本地 代码，从而带来最大程度的优化。【注意预热】
*  `-Xmixed`：`-Xmixed` 是混合模式，将解释模式和编译模式进行混合使用，有 JVM 自己决定，这 是 JVM 的默认模式，也是推荐模式。 我们使用 java -version 可以看到 mixed mode 等信息。

## 堆内存设置参数

### 显式指定堆内存`–Xms`和`-Xmx`

与性能有关的最常见实践之一是根据应用程序要求初始化堆内存。如果我们需要指定最小和最大堆大小（推荐显示指定大小），以下参数可以帮助你实现：

* `-Xmx`，指定最大堆内存。 如 -Xmx4g。这只是限制了 Heap 部分的最大值为 4g。这个内存不包括栈内存，也不包括堆外使用的内存。 
* `-Xms`，指定堆内存空间的初始大小。 如 -Xms4g。 而且指定的内存大小，并 不是操作系统实际分配的初始值，而是GC先规划好，用到才分配。 专用服务 器上需要保持 `–Xms` 和 `–Xmx` 一致，否则应用刚启动可能就有好几个 `FullGC`。 当两者配置不一致时，堆内存扩容可能会导致性能抖动。 

> 通常会将`-Xms`和`-Xmx`两个参数配置相同的值，其目的是**为了能够在Java垃圾回收机制清理完堆区后不需要重新分隔计算堆区的大小，从而提高性能**。

### 显式新生代内存

根据[Oracle官方文档](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/sizing.html)，在堆总可用内存配置完成之后，第二大影响因素是为 `Young Generation` 在堆内存所占的比例。默认情况下，YG 的最小大小为 1310 MB，最大大小为无限制。

* 一般而言，通过`-Xmn`等价于`-XX:NewSize`来指定新生代内存大小，使用 G1 垃圾收集器**不应该**设置该选项，在其他的某些业务场景下可以设置。官方建议设置为 `-Xmx 的 1/2 ~ 1/4`。 

举个栗子，如果我们要为 新生代分配 最小256m 的内存，最大 1024m的内存我们的参数应该这样来写：

```
-XX:NewSize=256m
-XX:MaxNewSize=1024m
```

如果我们要为 新生代分配256m的内存（`NewSize`与`MaxNewSize`设为一致），我们的参数应该这样来写：

```
-Xmn256m 
```

GC 调优策略中很重要的一条经验总结是这样说的：

> 将新对象预留在新生代，由于 Full GC 的成本远高于 Minor GC，因此尽可能将对象分配在新生代是明智的做法，实际项目中根据 GC 日志分析新生代空间大小分配是否合理，适当通过“-Xmn”命令调节新生代大小，最大限度降低新对象直接进入老年代的情况。

另外，还可以通过**`-XX:NewRatio=<int>`**来设置新生代和老年代内存的比值。

比如下面的参数就是设置新生代（包括Eden和两个Survivor区）与老年代的比值为1。也就是说：新生代与老年代所占比值为1：1，新生代占整个堆栈的 1/2。

```
-XX:NewRatio=1
```

### 显示指定永久代/元空间的大小

**从Java 8开始，如果我们没有指定 Metaspace 的大小，随着更多类的创建，虚拟机会耗尽所有可用的系统内存（永久代并不会出现这种情况）。**

JDK1.7 及之前永久代还没被彻底移除的时候通常通过下面这些参数来调节方法区大小，JDK 1.8设置此类参数会无效。

```java
-XX:PermSize=size //方法区 (永久代) 初始大小 
-XX:MaxPermSize=size //方法区 (永久代) 最大大小,超过这个值将会抛出 OutOfMemoryError 异常:java.lang.OutOfMemoryError: PermGen
```

相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入方法区后就“永久存在”了。

**JDK 1.8 的时候，方法区（HotSpot 的永久代）被彻底移除了，取而代之是元空间，元空间使用的是直接内存。**

```java
-XX:MetaspaceSize=size //设置 Metaspace 的初始（和最小大小）
-XX:MaxMetaspaceSize=size //设置 Metaspace 的最大大小，Java8默认不限制Meta空间，一般不允许设置该选项。
```

### 其他常用参数

*  `-XX：MaxDirectMemorySize=size`，系统可以使用的最大堆外内存，这个参数跟 `-Dsun.nio.MaxDirectMemorySize` 效果相同。
*  `-Xss`, 设置每个线程栈的字节数，影响栈的深度。 例如 `-Xss1m` 指定线程栈为 1MB，与`-XX:ThreadStackSize=1m` 等价。

下面我画了一张图，这样有助力理清`Xmx、Xms、Xmn、Meta、 DirectMemory、Xss `这些内存参数的关系：

 <div align="center"> <img src="..\images\jvm\JVM启动参数.png" width="850px"></div>

## GC设置参数

### 垃圾回收器

截止JDK1.8，一共有7款不同的垃圾收集器：

| 垃圾收集器             | 分类 | 作用位置       | 算法                    | 特点         | 使用场景                               |
| ---------------------- | ---- | -------------- | ----------------------- | ------------ | -------------------------------------- |
| Serial                 | 串行 | 新生代         | 标记-复制               | 响应速度优先 | 单CPU环境下的client模式                |
| Serial Old             | 串行 | 老年代         | 标记-压缩               | 响应速度优先 | 单CPU环境下的client模式（CMS后备方案） |
| ParNew                 | 并行 | 新生代         | 标记-复制               | 响应速度优先 | 多CPU环境Server模式下与CMS配合使用     |
| Parallel <br/>Scavenge | 并行 | 新生代         | 标记-复制               | 吞吐量优先   | 后台运算而不需要太多交互的场景         |
| Parallel Old           | 并行 | 老年代         | 标记-压缩               | 吞吐量优先   | 后台运算而不需要太多交互的场景         |
| CMS                    | 并发 | 老年代         | 标记-清除               | 响应速度优先 | 互联网站或B/S业务                      |
| G1                     | 并发 | 新生代、老年代 | 标记-压缩<br/>标记-复制 | 响应速度优先 | 面向服务端应用，将来替换CMS            |

**垃圾收集器的组合关系**

<div align="center"> <img src="..\images\jvm\20201015092111694.png" width="600px"></div>

新生代收集器：Serial、ParNew、Parallel Scavenge；老年代收集器：Serial Old、Parallel Old、CMS；整堆收集器：G1。

> 两个收集器间有连线，表明它们可以搭配使用：Serial/Serial Old、Serial/CMS、ParNew/Serial Old、ParNew/CMS、Parallel Scavenge/Serial Old、Parallel Scavenge/Parallel Old、G1。

```java
-XX:+UseSerialGC：使用串行垃圾回收器（Serial/Serial Old 组合）
-XX:+UseParallelGC：使用并行垃圾回收器（ParNew/Serial Old 组合）
-XX:+UseParallelGC -XX:+UseParallelOldGC：使用并行垃圾回收器（Parallel Scavenge/Parallel Old 组合）
-XX:+UseConcMarkSweepGC：使用 CMS 垃圾回收器（只能与ParNew/Serial搭配）
-XX:+UseG1GC：使用 G1 垃圾回收器
// Java 11+
-XX:+UnlockExperimentalVMOptions -XX:+UseZGC： 使用ZGC垃圾回收器
// Java 12+
-XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC：使用Shenandoah垃圾回收器
```

### GC记录

为了严格监控应用程序的运行状况，我们应该始终检查JVM的*垃圾回收*性能。最简单的方法是以人类可读的格式记录*GC*活动。

很多人使用`UseGCLogFileRotation`参数记录*GC*活动：

```java
-XX:+PrintGCDetails 打印GC日志详情
-Xloggc:gc.demo.log 指定日志文件
-Xloggc:/log/gc.demo.log 指定GC日志文件存放的绝对路径
-XX:+UseGCLogFileRotation 
-XX:NumberOfGCLogFiles=5 
-XX:GCLogFileSize=20M log文件大小

-XX:+PrintGCDateStamps 打印GC事件发生的日期时间
-XX:+PrintGCApplicationStoppedTime 输出每次GC的持续时间和程序暂停时间
-XX:+PrintReferenceGC 输出GC清理了多少引用类型
```

上面这一段JVM日志文件达到了20M以后，就会写入另一个新的文件，最多会有5个日志文件，他们的名字分别是：`gc.demo.log.0、gc.demo.log.1、... 、gc.demo.log.4`。

这样有以下几个问题：

* 丢失旧日志：5个日志文件写满后会再重写这5哥日志文件；
* 日志变混乱：例如重启JVM，此时GC的日志会重新从log.0开始写入，但是`log.1~og.4`日志仍是旧的日志；
* 不方便日志集中管理对日志分析工具不友好：用日志分析工具（gceasy、GCViewer）分析日志需要上传多个日志文件而非一个；

推荐解决办法：

可以给GC日志的文件后缀加上时间戳，当JVM重启以后，会生成新的日志文件，新的日志也不会覆盖老的日志，只需要在日志文件名中添加%t的后缀即可：

```java
-XX:+PrintGCDetails 
-XX:+PrintGCDateStamps 
-Xloggc:/home/GCEASY/gc-%t.log
```

> 详细分析可参考[Try to avoid -XX:+UseGCLogFileRotation](https://blog.gceasy.io/2019/01/29/try-to-avoid-xxusegclogfilerotation/)一文。

## 分析诊断参数

* `-XX：+-HeapDumpOnOutOfMemoryError` 选项，当 `OutOfMemoryError` 产生，即内存溢出（堆内存或持久代) 时，自动 Dump 堆内存。
  * 示例用法： `java -XX:+HeapDumpOnOutOfMemoryError -Xmx256m ConsumeHeap `
* `-XX：HeapDumpPath` 选项，与 `HeapDumpOnOutOfMemoryError` 搭配使用，指定内存溢出时 Dump 文件的 目录。 如果没有指定则默认为启动 Java 程序的工作目录。 
  * 示例用法：` java -XX:+HeapDumpOnOutOfMemoryError `
* `-XX:HeapDumpPath=/usr/local/ ConsumeHeap` 自动 Dump 的 hprof 文件会存储到 `/usr/local/` 目录下。 
* `-XX：OnError` 选项，发生致命错误时（fatal error）执行的脚本。例如, 写一个脚本来记录出错时间, 执行一些命令，或者 curl 一下某个在线报警的 url。 
  * 示例用法：`java -XX:OnError="gdb - %p" MyApp` 可以发现有一个 %p 的格式化字符串，表示进程 PID。 
* `-XX：OnOutOfMemoryError` 选项，抛出 `OutOfMemoryError `错误时执行的脚本。 `-XX：ErrorFile=filename` 选项，致命错误的日志文件名,绝对路径或者相对路径。 
* `-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=1506`，远程调试。

## JavaAgent参数

Agent 是 JVM 中的一项黑科技，可以通过无侵入方式来做很多事情，比如注入 AOP 代码，执行统 计等等，权限非常大。这里简单介绍一下配置选项，详细功能需要专门来讲。 设置 agent 的语法如下： 

* `-agentlib:libname[=options] `启用 native 方式的 agent，参考 LD_LIBRARY_PATH 路径。 
* `-agentpath:pathname[=options]` 启用 native 方式的 agent。 
* `-javaagent:jarpath[=options] `启用外部的 agent 库，比如 pinpoint.jar 等等。
* `-Xnoagent` 则是禁用所有 agent。 以下示例开启 CPU 使用时间抽样分析： 
  * `JAVA_OPTS="-agentlib:hprof=cpu=samples,file=cpu.samples.log"`

