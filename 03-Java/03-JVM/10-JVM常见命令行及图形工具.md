- [JDK 命令行工具](#jdk-命令行工具)
  - [jps：查看所有 Java 进程](#jps查看所有-java-进程)
  - [jinfo：实时地查看和调整虚拟机各项参数](#jinfo实时地查看和调整虚拟机各项参数)
  - [jstat：监视虚拟机各种运行状态信息](#jstat监视虚拟机各种运行状态信息)
  - [jmap：生成堆转储快照](#jmap生成堆转储快照)
  - [jhat：分析 heapdump 文件](#jhat分析-heapdump-文件)
  - [jstack ：生成虚拟机当前时刻的线程快照](#jstack-生成虚拟机当前时刻的线程快照)
  - [jcmd：执行 JVM 相关分析命令（整合命令）](#jcmd执行-jvm-相关分析命令整合命令)
- [JDK 可视化分析工具](#jdk-可视化分析工具)
  - [jsonsole：Java 监视与管理控制台](#jsonsolejava-监视与管理控制台)
    - [连接 Jconsole](#连接-jconsole)
    - [查看 Java 程序概况](#查看-java-程序概况)
    - [内存监控](#内存监控)
    - [线程监控](#线程监控)
    - [VM概要](#vm概要)
  - [jvisualvm](#jvisualvm)
  - [VisualGC](#visualgc)
  - [jmc](#jmc)

# JDK 命令行工具

这些命令在 JDK 安装目录下的 bin 目录下：

| 工具           | 简介                                                                                                                          |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| **java**       | Java应用启动程序                                                                                                              |
| **javac**      | JDK 内置的编译工具                                                                                                            |
| **javap**      | 反编译 class 文件的工具                                                                                                       |
| javadoc        | 根据 Java 代码和标准注释，自动生成相关的 API 说明文档                                                                         |
| javah          | JNI 开发时, 根据 java 代码生成需要的 .h文件                                                                                   |
| extcheck       | 检查某个 jar 文件和运行时扩展 jar 有没有版本冲突，很少使用                                                                    |
| jdb            | Java Debugger ; 可以调试本地和远端程序，属于 JPDA 中的一个 demo 实现，供其 他调试器参考。开发时很少使用                       |
| jdeps          | 探测 class 或 jar 包需要的依赖                                                                                                |
| jar            | 打包工具，可以将文件和目录打包成为 .jar 文件；.jar 文件本质上就是 zip 文件，只 是后缀不同。使用时按顺序对应好选项和参数即可。 |
| keytool        | 安全证书和密钥的管理工具; （支持生成、导入、导出等操作）                                                                      |
| jarsigner      | JAR 文件签名和验证工具                                                                                                        |
| policytool     | 实际上这是一款图形界面工具, 管理本机的 Java 安全策略                                                                          |
| **jps/jinfo**  | 查看 java 进程                                                                                                                |
| **jstat**      | 查看 JVM 内部 gc 相关信息                                                                                                     |
| **jmap**       | 查看 heap 或类占用空间统计                                                                                                    |
| **jstack**     | 查看线程信息                                                                                                                  |
| **jhat**       | 用于分析 heapdump 文件，它会建立一个 HTTP/HTML 服务器，让用户可以在浏览器上查看分析结果;                                      |
| jcmd           | 执行 JVM 相关分析命令（整合命令）                                                                                             |
| jrunscript/jjs | 执行 js 命令                                                                                                                  |

- **`jps`** (JVM Process Status）: 类似 UNIX 的 `ps` 命令。
- **`jstat`**（ JVM Statistics Monitoring Tool）:  用于收集 HotSpot 虚拟机各方面的运行数据;
- **`jinfo`** (Configuration Info for Java) ：Configuration Info forJava,显示虚拟机配置信息;
- **`jmap`** (Memory Map for Java)  ：生成堆转储快照;
- **`jhat`** (JVM Heap Dump Browser )  ：用于分析 heapdump 文件，它会建立一个 HTTP/HTML 服务器，让用户可以在浏览器上查看分析结果;
- **`jstack`** (Stack Trace for Java) ：生成虚拟机当前时刻的线程快照，线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合。

## jps：查看所有 Java 进程

**`jps`** (JVM Process Status）命令类似 UNIX 的 `ps` 命令。用于查看所有 Java 进程的启动类、传入参数和 Java 虚拟机参数等信息；

`jps`：显示虚拟机执行主类名称以及这些进程的本地虚拟机唯一 ID（Local Virtual Machine Identifier, LVMID）。

`jps -q` ：只输出进程的本地虚拟机唯一 ID。

```powershell
C:\Users\kaiz1>jps
12724 Jps
10296 Launcher
10792
11816 RemoteMavenServer36
2848 ThreadTest
```

`jps -l`：输出主类的全名，如果进程执行的是 Jar 包，输出 Jar 路径。

```powershell
C:\Users\kaiz1>jps -l
18832 org.jetbrains.jps.cmdline.Launcher
2512 sun.tools.jps.Jps
5520 org.zgkaii.java8.ThreadTest
10792
11816 org.jetbrains.idea.maven.server.RemoteMavenServer36
```

`jps -v`：输出虚拟机进程启动时 JVM 参数。

`jps -m`：输出传递给 Java 进程 main() 函数的参数。
`jps -mlv`： 打印Java 各个进程的详尽参数。

## jinfo：实时地查看和调整虚拟机各项参数

`jinfo vmid` ：输出当前 jvm 进程的全部参数和系统属性 (第一部分是系统的属性，第二部分是 JVM 的参数)。

`jinfo -flag name vmid` ：输出对应名称的参数的具体值。比如输出 MaxHeapSize、查看当前 jvm 进程是否开启打印 GC 日志 ( `-XX:PrintGCDetails` ：详细 GC 日志模式，这两个都是默认关闭的)。

```powershell
C:\Users\kaiz1>jinfo -flag MaxHeapSize 2848
-XX:MaxHeapSize=4236247040

C:\Users\kaiz1>jinfo -flag PrintGC 2848
-XX:-PrintGC
```

使用 jinfo 可以在不重启虚拟机的情况下，可以动态的修改 jvm 的参数。尤其在线上的环境特别有用,请看下面的例子：

`jinfo -flag [+|-]name vmid` 开启或者关闭对应名称的参数。

```powershell
C:\Users\kaiz1>jinfo -flag PrintGC 2848
-XX:-PrintGC

C:\Users\kaiz1>jinfo -flag +PrintGC 2848

C:\Users\kaiz1>jinfo -flag PrintGC 2848
```

## jstat：监视虚拟机各种运行状态信息

jstat（JVM Statistics Monitoring Tool） 使用于监视虚拟机各种运行状态信息的命令行工具。 它可以显示本地或者远程（需要远程主机提供 RMI 支持）虚拟机进程中的类信息、内存、垃圾收集、JIT 编译等运行数据，在没有 GUI，只提供了纯文本控制台环境的服务器上，它将是运行期间定位虚拟机性能问题的首选工具。

**`jstat` 命令使用格式：**

```powershell
jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
```

比如 `jstat -gc -h3 2848 1000 10`表示分析进程 id 为 2848 的 gc 情况，每隔 1000ms 打印一次记录，打印 10 次停止，每 3 行后打印指标头部。

**常见的 option 如下：**

- `jstat -class vmid` ：类加载(Class loader)信息统计；
- `jstat -compiler vmid` ：JIT 即时编译器相关的统计信息；
- `jstat -gc vmid` ：GC 相关的堆内存信息；
- `jstat -gccapacity vmid` ： 各个内存池分代空间的容量；
- `jstat -gccause vmid`：看上次 GC，本次 GC（如果正在 GC 中）的原因， 其他 输出和 -gcutil 选项一致
- `jstat -gcnew vmid` ：年轻代的统计信息（New = Young = Eden + S0 + S1）；
- `jstat -gcnewcapcacity vmid` ：年轻代空间大小统计；
- `jstat -gcold vmid` ：老年代和元数据区的行为统计；
- `jstat -gcoldcapacity vmid` ：显示老年代的大小；
- `jstat -gcpermcapacity vmid` ：显示永久代大小；
- `jstat -gcmetacapacity vmid`： meta 区大小统计;
- `jstat -gcutil vmid` ：GC 相关区域的使用率（utilization）统计；

另外，加上 `-t`参数可以在输出信息上加一个 Timestamp 列，显示程序的运行时间。

**使用演示**

`jsat -gcutil -t 2848 1000 1000`：表示每隔1000ms打印一次2848进程的GC相关区域使用率，打印1000次结束。

```powershell
C:\Users\kaiz1>jstat -gcutil -t 2848 1000 1000
Timestamp     S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
    435.4   0.00   0.00  12.00   0.63  78.98  82.02    188    0.096     0    0.000    0.096
    436.4   0.00   0.00  64.00   0.63  78.98  82.02    188    0.096     0    0.000    0.096
    437.5   0.00   0.00  16.00   0.63  78.98  82.02    189    0.096     0    0.000    0.096
    438.4   0.00   0.00  64.00   0.63  78.98  82.02    189    0.096     0    0.000    0.096
    --- omit ---
```

其中，Timestamp列为JVM启动了438秒了，`S0`为0号存活区的百分比使用率为0，这很正常，因为`S0`与`S1`随时都是空的；`S1`为1号存活区的百分比使用率；E为Eden区，新生代的百分比使用率；O为Old区，老年代百分比使用率；M为Meta区，元数据百分比使用率；`CSS`即压缩class空间`Compressed class space`的百分比使用率；YGC（Young GC）年轻代GC的次数；YGCT，年轻代GC消耗的总时间；FGC为Full GC发生的次数；FGCT为Full GC消耗的总时间；GCT为所有GC总消耗时间，即FGCT+YGCT。

可以看到`-gcutil`打印的信息为百分比，不太直观，下面使用`-gc`：

```powershell
C:\Users\kaiz1>jstat -gc -t 2848 1000 1000
Timestamp   S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   
    1032.9 512.0  512.0   0.0    0.0   31232.0   1249.3   173568.0    1096.1   4864.0 3841.6 512.0  
CCSU   YGC  YGCT   FGC    FGCT   GCT         
419.9  482  0.245  0      0.000  0.245
```

可以看出，打印出的都是使用的容量(KB)。其中C后缀为容量，U后缀表示使用量。例如`CCSC`即压缩class空间容量为512.0 KB，CCSU表示已使用419.9KB。

## jmap：生成堆转储快照

`jmap`（Memory Map for Java）命令用于生成堆转储快照。 如果不使用 `jmap` 命令，要想获取 Java 堆转储，可以使用 `“-XX:+HeapDumpOnOutOfMemoryError”` 参数，可以让虚拟机在 OOM 异常出现之后自动生成 dump 文件，Linux 命令下可以通过 `kill -3` 发送进程退出信号也能拿到 dump 文件。

`jmap` 的作用并不仅仅是为了获取 dump 文件，它还可以查询 finalizer 执行队列、Java 堆和永久代的详细信息，如空间使用率、当前使用的是哪种收集器等。和`jinfo`一样，`jmap`有不少功能在 Windows 平台下也是受限制的。

`jmap`常用选项为3个：

* `-heap pid`  ：打印堆内存（/内存池）的配置和 使用信息。 
* `-histo pid` ：看哪些类占用的空间最多, 直方图。 
* `-dump:format=b,file=xxxx.hprof pid`：生成堆转储快照。

示例：将指定应用程序的堆快照输出到桌面。后面，可以通过 jhat、Visual VM 等工具分析该堆文件。

```powershell
C:\Users\kaiz1>jmap -dump:format=b,file=C:\Users\kaiz1\Desktop\heap.hprof 2848
Dumping heap to C:\Users\kaiz1\Desktop\heap.hprof ...
Heap dump file created
```

## jhat：分析 heapdump 文件

 **`jhat`** 用于分析 heapdump 文件，它会建立一个 HTTP/HTML 服务器，让用户可以在浏览器上查看分析结果。

```powershell
C:\Users\kaiz1>jhat C:\Users\kaiz1\Desktop\heap.hprof
Reading from C:\Users\kaiz1\Desktop\heap.hprof...
Dump file created Sat May 29 11:38:04 CST 2021
Snapshot read, resolving...
Resolving 251898 objects...
Chasing references, expect 50 dots..................................................
Eliminating duplicate references..................................................
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.
```

访问 <http://localhost:7000/>。

## jstack ：生成虚拟机当前时刻的线程快照

`jstack`（Stack Trace for Java）命令用于生成虚拟机当前时刻的线程快照。线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合.

生成线程快照的目的主要是定位线程长时间出现停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等都是导致线程长时间停顿的原因。线程出现停顿的时候通过`jstack`来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做些什么事情，或者在等待些什么资源。

常见使用选项有：

* `-F `强制执行 thread dump，可在 Java 进程卡死 （hung 住）时使用，此选项可能需要系统权限。 
* `-m` 混合模式（mixed mode)，将 Java 帧和 native 帧一起输出，此选项可能需要系统权限。
* `-l` 长列表模式，将线程相关的 locks 信息一起输 出，比如持有的锁，等待的锁。

**下面是一个线程死锁的代码。我们下面会通过 `jstack` 命令进行死锁检查，输出死锁信息，找到发生死锁的线程。**

```java
public class DeadLockTest {
    private static Object resource1 = new Object();
    private static Object resource2 = new Object();

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (resource1) {
                System.out.println(Thread.currentThread() + "get resource1");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource2");
                synchronized (resource2) {
                    System.out.println(Thread.currentThread() + "get resource2");
                }
            }
        }, "线程 1").start();

        new Thread(() -> {
            synchronized (resource2) {
                System.out.println(Thread.currentThread() + "get resource2");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource1");
                synchronized (resource1) {
                    System.out.println(Thread.currentThread() + "get resource1");
                }
            }
        }, "线程 2").start();
    }
}
```

控制台输出：

```java
Thread[线程 1,5,main]get resource1
Thread[线程 2,5,main]get resource2
Thread[线程 1,5,main]waiting get resource2
Thread[线程 2,5,main]waiting get resource1
```

线程 A 通过 synchronized (resource1) 获得 resource1 的监视器锁，然后通过` Thread.sleep(1000);`让线程 A 休眠 1s 为的是让线程 B 得到执行然后获取到 resource2 的监视器锁。线程 A 和线程 B 休眠结束了都开始企图请求获取对方的资源，然后这两个线程就会陷入互相等待的状态，这也就产生了死锁。

**通过 `jstack` 命令分析：**可以使用 jps 定位进程 pid，再用 jstack 定位死锁。

```powershell
C:\Users\kaiz1>jps
1792 DeadLockTest
7840 Launcher
17556 Jps
10792
11816 RemoteMavenServer36

C:\Users\kaiz1>jstack 1792
2021-05-29 11:54:08
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.261-b12 mixed mode):
--- omit ---
Found one Java-level deadlock:
=============================
"线程 2":
  waiting to lock monitor 0x00000237fc9425d8 (object 0x000000076c026748, a java.lang.Object),
  which is held by "线程 1"
"线程 1":
  waiting to lock monitor 0x00000237fc944e68 (object 0x000000076c026758, a java.lang.Object),
  which is held by "线程 2"

Java stack information for the threads listed above:
===================================================
"线程 2":
        at org.zgkaii.java8.DeadLockTest.lambda$main$1(DeadLockTest.java:38)
        - waiting to lock <0x000000076c026748> (a java.lang.Object)
        - locked <0x000000076c026758> (a java.lang.Object)
        at org.zgkaii.java8.DeadLockTest$$Lambda$2/558638686.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)
"线程 1":
        at org.zgkaii.java8.DeadLockTest.lambda$main$0(DeadLockTest.java:23)
        - waiting to lock <0x000000076c026758> (a java.lang.Object)
        - locked <0x000000076c026748> (a java.lang.Object)
        at org.zgkaii.java8.DeadLockTest$$Lambda$1/1831932724.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.
```

可以看到 `jstack` 命令已经帮我们找到发生死锁的线程的具体信息。

## jcmd：执行 JVM 相关分析命令（整合命令）

Jcmd 综合了前面的几个命令，例如：

```powershell
jcmd pid VM.version 
jcmd pid VM.flags 
jcmd pid VM.command_line 
jcmd pid VM.system_properties
jcmd pid Thread.print 
jcmd pid GC.class_histogram 
jcmd pid GC.heap_info
```

# JDK 可视化分析工具

## jsonsole：Java 监视与管理控制台

JConsole 是基于 JMX 的可视化监视、管理工具。可以很方便的监视本地及远程服务器的 java 进程的内存使用情况。你可以在控制台输出`console`命令启动或者在 JDK 目录下的 bin 目录找到`jconsole.exe`然后双击启动。

### 连接 Jconsole

 <div align="center"> <img src="https://img-blog.csdnimg.cn/20210529150144396.png" width="600px"></div>

如果需要使用 JConsole 连接远程进程，可以在远程 Java 程序启动时加上下面这些参数（打开JMX）:

```properties
-Djava.rmi.server.hostname=外网访问 ip 地址 
-Dcom.sun.management.jmxremote.port=60001   //监控的端口号
-Dcom.sun.management.jmxremote.authenticate=false   //关闭认证
-Dcom.sun.management.jmxremote.ssl=false
```

在使用 JConsole 连接时，远程进程地址如下：

```powershell
外网访问 ip 地址:60001 
```

### 查看 Java 程序概况

 <div align="center"> <img src="https://img-blog.csdnimg.cn/20210529150417276.png" width="800px"></div>

### 内存监控

JConsole 可以显示当前内存的详细信息。不仅包括堆内存/非堆内存的整体信息，还可以细化到 eden 区、survivor 区等的使用情况，如下图所示。

内存图表包括： 

* 堆内存使用量，主要包括老年代（内存池 “PS Old Gen”）、新生代（“PS Eden Space”）、存活区 （“PS Survivor Space”）；
* 非堆内存使用量，主要包括内存池“Metaspace”、 “Code Cache”、“Compressed Class Space”等。 可以分别选择对应的 6 个内存池。 

通过内存面板，我们可以看到各个区域的内存使用和变化情 况，并且可以： 

1. 手动执行 gc，见图上的标号1，点击按钮即可执行 JDK 中 的 System.gc() 
2. 通过图中右下角标号 2 的界面，可以看到各个内存池的百 分比使用率，以及堆/非堆空间的汇总使用情况。
3. 从左下角标号 3 的界面，可以看到 JVM 使用的垃圾收集器， 以及执行垃圾收集的次数，以及相应的时间消耗。

<div align="center"> <img src="https://img-blog.csdnimg.cn/20210529151106419.png" width="800px"></div>

### 线程监控

类似我们前面讲的 `jstack` 命令，不过这个是可视化的。线程面板展示了线程数变化信息，以及监测到 的线程列表。 我们可以常根据名称直接查看线程的状态（运 行还是等待中）和调用栈（正在执行什么操 作）。 特别地，我们还可以直接点击“检测死锁”按 钮来检测死锁，如果没有死锁则会提示“未检 测到死锁”。

<div align="center"> <img src="https://img-blog.csdnimg.cn/20210529150651610.png" width="800px"></div>

### VM概要

<div align="center"> <img src="https://img-blog.csdnimg.cn/20210529151203565.png" width="700px"></div>

VM 概要的数据有五个部分： 

* 第一部分是虚拟机的信息； 
* 第二部分是线程数量，以及类加载的汇 总信息；
* 第三部分是堆内存和 GC 统计； 
* 第四部分是操作系统和宿主机的设备信 息，比如 CPU 数量、物理内存、虚拟 内存等等； 
* 第五部分是 JVM 启动参数和几个关键 路径，这些信息其实跟 jinfo 命令看到 的差不多。

## jvisualvm

VisualVM 提供在 Java 虚拟机 (Java Virutal Machine, JVM) 上运行的 Java 应用程序的详细信息。在 VisualVM 的图形用户界面中，您可以方便、快捷地查看多个 Java 应用程序的相关信息。Visual VM 官网：<https://visualvm.github.io/> 。Visual VM 中文文档：<https://visualvm.github.io/documentation.html>。

<div align="center"> <img src="https://img-blog.csdnimg.cn/20210529151713143.png" width="700px"></div>

<div align="center"> <img src="https://img-blog.csdnimg.cn/20210529151913902.png" width="700px"></div>

下面这段话摘自《深入理解 Java 虚拟机》。

> VisualVM（All-in-One Java Troubleshooting Tool）是到目前为止随 JDK 发布的功能最强大的运行监视和故障处理程序，官方在 VisualVM 的软件说明中写上了“All-in-One”的描述字样，预示着他除了运行监视、故障处理外，还提供了很多其他方面的功能，如性能分析（Profiling）。VisualVM 的性能分析功能甚至比起 JProfiler、YourKit 等专业且收费的 Profiling 工具都不会逊色多少，而且 VisualVM 还有一个很大的优点：不需要被监视的程序基于特殊 Agent 运行，因此他对应用程序的实际性能的影响很小，使得他可以直接应用在生产环境中。这个优点是 JProfiler、YourKit 等工具无法与之媲美的。

 VisualVM 基于 NetBeans 平台开发，因此他一开始就具备了插件扩展功能的特性，通过插件扩展支持，VisualVM 可以做到：

- **显示虚拟机进程以及进程的配置、环境信息（jps、jinfo）。**
- **监视应用程序的 CPU、GC、堆、方法区以及线程的信息（jstat、jstack）。**
- **dump 以及分析堆转储快照（jmap、jhat）。**
- **方法级的程序运行性能分析，找到被调用最多、运行时间最长的方法。**
- **离线程序快照：收集程序的运行时配置、线程 dump、内存 dump 等信息建立一个快照，可以将快照发送开发者处进行 Bug 反馈。**
- **其他 plugins 的无限的可能性......**

这里就不具体介绍 VisualVM 的使用，如果想了解的话可以看:

- <https://visualvm.github.io/documentation.html>
- <https://www.ibm.com/developerworks/cn/java/j-lo-visualvm/index.html>

## VisualGC

IDEA中有插件VisualGC，安装后即在右下角：

<div align="center"> <img src="https://img-blog.csdnimg.cn/20210529152219910.png" width="700px"></div>

## jmc

jmc是目前Oracle提供最强大的JVM可视化工具，它支持远程连接JVM（通过JMX连接如果想要用jmc监控远程的JVM进程，配置方式和jvisualvm方式一一样即可），除此之外它还支持Java飞行记录器。

<div align="center"> <img src="https://img-blog.csdnimg.cn/20210529152824420.png" width="700px"></div>

详细使用可查看[https://cs.xieyonghui.com/java/java-flight-recorder_72.html](https://cs.xieyonghui.com/java/java-flight-recorder_72.html)