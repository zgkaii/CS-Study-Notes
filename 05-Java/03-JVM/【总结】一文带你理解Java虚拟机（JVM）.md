-- MarkdownTOC -->
- [一、JVM基础知识](#一jvm基础知识)
  - [1.1 JDK、JRE与JVM](#11-jdkjre与jvm)
  - [1.2 字节码](#12-字节码)
  - [1.3 .class文件执行流程](#13-class文件执行流程)
- [二、类加载器](#二类加载器)
  - [2.1 类加载生命周期](#21-类加载生命周期)
  - [2.2 类加载器](#22-类加载器)
  - [2.3 双亲委派机制](#23-双亲委派机制)
- [三、运行时数据区](#三运行时数据区)
  - [3.1 程序计数器](#31-程序计数器)
  - [3.2 虚拟机栈](#32-虚拟机栈)
  - [3.3 本地方法栈](#33-本地方法栈)
  - [3.4 堆](#34-堆)
  - [3.5 方法区](#35-方法区)
- [四、垃圾回收器](#四垃圾回收器)
  - [4.1 垃圾回收算法](#41-垃圾回收算法)
    - [4.1.1 标记阶段](#411-标记阶段)
    - [4.1.2 清除阶段](#412-清除阶段)
  - [4.2 常见GC](#42-常见gc)
    - [4.2.1 Serial GC：串行回收](#421-serial-gc串行回收)
    - [4.2.2 ParNew GC：并行回收](#422-parnew-gc并行回收)
    - [4.2.3 Parallel GC：吞吐量优先](#423-parallel-gc吞吐量优先)
    - [4.2.4 CMS：低延迟](#424-cms低延迟)
    - [4.2.5 G1：垃圾优先](#425-g1垃圾优先)
  - [4.3 GC总结](#43-gc总结)
- [五、执行引擎](#五执行引擎)
  - [5.1 Java代码编译和执行过程](#51-java代码编译和执行过程)
  - [5.2 编译器分类](#52-编译器分类)
    - [5.2.1 前端编译器](#521-前端编译器)
    - [5.2.2 JIT编译器](#522-jit编译器)
    - [5.2.3 AOT编译器](#523-aot编译器)
- [六、JVM启动参数](#六jvm启动参数)
  - [6.1 系统属性参数](#61-系统属性参数)
  - [6.2 运行模式参数](#62-运行模式参数)
  - [6.3 堆内存设置参数](#63-堆内存设置参数)
  - [6.4 GC设置参数](#64-gc设置参数)
    - [6.4.1 垃圾回收器](#641-垃圾回收器)
    - [6.4.2 GC记录](#642-gc记录)
  - [6.5 分析诊断参数](#65-分析诊断参数)
  - [6.6 JavaAgent参数](#66-javaagent参数)
- [七、命令行及图形工具](#七命令行及图形工具)
  - [7.1 命令行工具](#71-命令行工具)
  - [7.2 JDK 可视化分析工具](#72-jdk-可视化分析工具)

<!-- /MarkdownTOC -->

# 一、JVM基础知识

## 1.1 JDK、JRE与JVM

**JDK(Java Development Kit)**是用于开发Java应用程序的软件工具集合，主要包括了Java运行时环境（JRE）和一些Java开发工具（例如解释器`java.exe`，编译器`javac.exe`，打包工具`jar.exe`，文档生成器`javadoc.exe`等等）。

**JRE(Java RunTime Environment)**提供Java应用程序执行时所需的环境，由Java虚拟机（JVM）、核心类（libraries）及支持文件组成。

**JVM(Java Virtual Environment)**即所谓的Java虚拟机，它用于解释由`.java`文件编译生成的`.class`字节码文件，并将其映射到本地的CPU的指令集或OS系统调用中。虽然不同的操作系统使用不同的JVM映射规则，但JVM解释/编译字节码文件的规则一致，这使得Java语言具有了跨平台的特性。除此之外，JVM平台的不同语言还共享JVM所带来的垃圾回收器（GC）与即时编译器（JIT）。

<div align="center"> <img src="..\images\jvm\runanywhere.png" width="600px"></div>

三者之间关系：

```html
JDK > JRE > JVM
JDK = JRE + tools => JRE = JVM + libraries
```

## 1.2 字节码

我们平时说的Java字节码（Java bytecode），指的是用Java语言编译成的字节码。不同的编译器，可以编译出相同的字节码文件，字节码文件也可以在不同的JVM上运行。

Java虚拟机与Java语言并没有必然的联系，它只与特定的**二进制文件格式**——Class文件格式所关联，Class文件中包含了Java虚拟机指令集（或者称为字节码、Bytecodes）和符号表，还有一些其他辅助信息。

Java虚拟机根本不关心运行在其内部的程序到底是使用何种编程语言编写的，它只关心**“字节码”文件**。也就是说Java虚拟机拥有语言无关性，并不会单纯地与Java语言“终身绑定”，**只要其他编程语言的编译结果满足并包含Java虚拟机的内部指令集、符号表以及其他的辅助信息，它就是一个有效的字节码文件，就能够被虚拟机所识别并装载运行**。

Java 源程序经过编译器编译后变成字节码，字节码由虚拟机解释执行，虚拟机将每一条要执行的字节码送给解释器，解释器将其翻译成特定机器上的机器码，然后在特定的机器上运行。**这也就是解释了 Java 的编译与解释并存的特点**。

## 1.3 .class文件执行流程

Java虚拟机与Java语言并没有必然的联系，它只与特定的**二进制文件格式**——`.class文件`格式所关联，class文件中包含了Java虚拟机指令集（或者称为字节码、Bytecodes）和符号表，还有一些其他辅助信息。

<div align="center"> <img src="..\images\jvm\class执行.png" width="600px"></div>

# 二、类加载器

## 2.1 类加载生命周期

一个类在JVM里的生命周期有7个阶段，分别是加载（Loading）、校验（Verification）、准备（Preparation）、解析（Resolution）、初始化 （Initialization）、使用（Using）、卸载（Unloading）。 其中前五部分（加载，验证，准备，解析，初始化）统称为类加载。

<div align="center"> <img src="..\images\jvm\类的生命周期.png" width="600px"></div>

**加载阶段**：是整个“类加载”（Class Loading）过程中的第一个阶段，JVM需要完成三件事：

* 通过一个类的**全限定名获取定义此类的二进制字节流**，简单点说就是 **找到文件系统中/jar包中/或存在于任何地方的“ class文件 ”**。 如果找不到则会抛出 `NoClassDefFound `错误。
* 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
* **在内存中生成一个代表这个类的`java.lang.Class`对象**，作为方法区这个类的各种数据的访问入口。

**校验阶段**：确保Class文件的**字节流**中包含信息符合当前虚拟机要求，保证被加载类的正确性，不会危害虚拟机自身安全。

* 文件格式验证：验证字节流是否符合 Class 文件格式的规范。例如：是否以**魔数0xCAFEBABE**开头、主次版本号是否在当前虚拟机的处理范围之内、常量池中的常量是否有不被支持的类型。
* 元数据验证：对字节码描述的信息进行语义分析，以保证其描述的信息符合 Java 语言规范的要求。例如：这个类是否有父类（除了`java.lang.Object`之外，所有的类都应当有父类）；这个类的父类是否继承了不允许被继承的类（被final修饰的类）... ...
* 字节码验证：通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的。
* 符号引用验证：对类自身以外（常量池中的各种符号引用）的各类信息进行匹配性校验，通俗来说就是，该类是否缺少或者被禁止访问它依赖的某些外部类、方法、字段等资源。

**准备阶段**：为类的静态变量分配内存，并将其初始化为默认值。

这个阶段将会创建静态字段, 并将其初始化为标准默认值（**比如 null 或者 0值**，Boolean类型数据的零值为False ），并分配方法表，即在方法区中分配这些变量所使用的内存空间。

* **这里不包含用final修饰的static，因为final在编译的时候就会分配了，准备阶段会显式初始化**；

* 这里不会为实例变量分配初始化，类变量会分配在方法区中，而实例变量是会随着对象一起分配到Java堆中。

**解析阶段**：解析符号引用阶段主要将**常量池**内的**符号引用转换为直接引用的过程**。

* 解析动作主要针对**类或接口、字段、类方法、接口方法、方法类型**等。对应常量池中的`CONSTANT Class info、CONSTANT Fieldref info、CONSTANT_Methodref_info`等。

**初始化阶段**：JVM规范明确规定, 必须在类的首次“主动使用”时才能执行类初始化。 初始化的过程包括执行： **类构造器方法，static静态变量赋值语句 和static静态代码块**。

* 编译器收集`<clinit>()`动作按照源文件顺序执行。所以说，静态语句块中只能访问到定义在静态语句块之外的变量，静态语句块中访问静态语句块中的变量会提示“非法向前引用”。
* 父类的`<cinit>()`方法优先执行，也就意味着父类中定义的静态语句块要优先于子类的变量赋值操作

## 2.2 类加载器

类加载过程可以描述为“通过一个类的全限定类名来获取此类的class对象”，即所谓的全限定名来获取描述该类的二进制字节流），这个过程由**类加载器**（Class Loader）来完成。

在程序中我们最常见的类加载器主要有3类：**启动类加载器**（BootstrapClassLoader）、**扩展类加载器**（ExtClassLoader） 和**应用类加载器**（AppClassLoader）。

<div align="center"> <img src="..\images\jvm\JVM 类加载器.png" width="800px"></div>

其中，启动类加载器又名引导类加载器，用C/C++语言实现，主要用于加载Java核心类库，如`rt.jar`。

扩展类加载器派生于`ClassLoader`类，主要加载`/lib/ext`扩展目录的jar包。

应用程序类加载器又名系统类加载器，派生于`ClassLoader`类，负责加载用户环境变量`CLASSPATH`或系统属性`java.class.path`指定路径下的类库，该类加载是程序中默认的类加载器。

用户自定义类加载器常用于实现隔离加载类、修改类加载的方式、扩展加载源与防止源码泄漏。实现自定义加载器步骤如下：

* 继承抽象类`java.lang.ClassLoader`类。
* JDK 1.2之前，重写`loadClass()`方法；JDK 1.2 之后，建议将类加载逻辑重写在`findclass()`方法中。
* 没有过于复杂的需求，可直接继承`URIClassLoader`类，这样可避免去编写`findclass()`方法及其获取字节码流的方式，

## 2.3 双亲委派机制

Java中类加载机制为**双亲委派机制**，它指的是，当某个特定的类加载器在接到加载类的请求时，首先将加载任务委托给父类加载器，**依次递归**，如果父类加载器可以完成类加载任务，就成功返回；只有父类加载器无法完成此加载任务时，才自己去加载。

<div align="center"> <img src="..\images\jvm\双亲委托.png" width="600px"></div>

**双亲委派机制的优势**：（1）避免类的重复加载（2）保护程序安全，防止核心API被随意篡改。

例如，自定义一个`java.lang.String`类。由于双亲委派，JVM在加载自定义String类的时候会率先使用启动类加载器加载，而启动类加载器在加载的过程中会先加载Java核心类库`rt.jar`中的`java\lang\String.class`，那么就根本无法加载到main方法。

```java
public class String {
    static {
        System.out.println("这是自定义的String类的静态代码块！");
    }
	
    // 无法加载，错误
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```

# 三、运行时数据区

JVM在执行Java程序的过程中会把它所管理的内存主要划分5个不同的数据区域：**本地方法栈、程序计数器、虚拟机栈、堆、方法区**。这些区域有各自的用途和创建与销毁的时间。有的区域随着虚拟机进程的启动而一直存在，有些区域则是依赖用户线程的启动和结束而建立和销毁。

 <div align="center"> <img src="..\images\jvm\运行时数据区.png" width="800px"></div>

==注意==，《Java 虚拟机规范》中指明：“尽管所有的方法区在逻辑上是属于堆的一部分，但一些简单的实现可能不会选择区进行垃圾收集或者进行压缩。” 但对于 HotSpotJVM 而言，方法区还有一个别名叫做**Non-Heap(非堆）**，目的就是要和堆分开。所以**方法区看作是一块独立于 Java 堆的内存空间**。

如上图所描述一样，在JDK 1.7 之前，方法区在**逻辑上**属于**非堆**，是一块独立的内存区域，但其实际物理内存其实是在堆区的（存放于永久代）；直到JDK 1.8后，方法区的实现才被移动至直接内存中的元空间（MetaSpace）中，逻辑内存与实际物理内存都与堆区分开。

一般来讲，黄色部分为线程隔离的数据区域，其他部分为线程共享的区域。

|    区域    | 是否线程共享 | 是否存在内存溢出 | 是否存在GC |
| :--------: | :----------: | :--------------: | :--------: |
| 本地方法栈 |      否      |        是        |     否     |
| 程序计数器 |      否      |        否        |     否     |
|  虚拟机栈  |      否      |        是        |     否     |
|    堆区    |      是      |        是        |     是     |
|   方法区   |      是      |        是        |     是     |

> 内存溢出与内存泄漏：
>
> 内存泄漏（Memory Leak）是指本来无用的对象却继续占用内存，没有在恰当的时机释放占用的内存的情况。
>
> 内存溢出（Out Of Memory，简称OOM）是指可使用的内存不足的情况。
>
> 一般来讲，内存泄漏是资源管理问题或程序BUG，而内存溢出则是内存空间不足和内存泄漏的最终结果。

## 3.1 程序计数器

程序计数器（Program Counter Register）是对物理PC寄存器的一种模拟，主要用于存储当前线程需要执行的字节码指令地址。在JVM规范中，每个线程都有自己的程序计数器，是线程私有的，生命周期与线程生命周期保持一致。

例如下面的字节码文件，字节码左边的行号标识，它其实就是**指令地址**。

```java
 0 bipush 10
 2 istore_1
 3 bipush 20
 5 istore_2
```

> 注意：程序计算器是运行速度最快的存储区域，也是唯一一块不会出现OOM的内存区域。

## 3.2 虚拟机栈

常常有人把Java内存区域笼统地划分为堆内存（Heap）和栈内存（Stack），这种划分方式直接继承自传统的C、C++程序的内存布局结构，在Java语言里就显得有些粗糙了，实际的内存区域划分要比这更复杂。在Java中，“栈”通常就是指这里讲的虚拟机栈。

Java虚拟机栈（Java Virtual Machine Stack）也是线程私有的，它的生命周期与线程相同。虚拟机栈描述的是Java方法执行的线程内存模型：每个方法被执行的时候，Java虚拟机都会同步创建一个栈帧（Stack Frame）用于存储局部变量表、操作数栈、动态连接、方法出口等信息。每一个方法被调用直至执行完毕的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。

**Java 虚拟机栈会出现两种错误：`StackOverFlowError` 和 `OutOfMemoryError`。**

- **`StackOverFlowError`：** 若Java虚拟机栈的内存大小不允许动态扩展，那么当线程请求栈的深度超过当前虚拟机栈的最大深度的时候，就抛出 StackOverFlowError 错误。
- **`OutOfMemoryError`：** 若Java虚拟机堆中没有空闲内存，并且垃圾回收器也无法提供更多内存的话。就会抛出OutOfMemoryError 错误。

**扩展：方法/函数如何调用？**

栈是一种快速有效的分配存储方式，**访问速度仅次于程序计数器**。JVM直接对Java栈的操作只有两个：

- 每一次方法执行，都伴随着对应的栈帧压栈（进栈/入栈）。
- 每一个方法执行结束，都会有一个对应栈帧被弹出。

## 3.3 本地方法栈

本地方法栈（Native Method Stack）用于管理本地方法的调用。其中，本地方法（NatIve Method）就是调用非Java代码的接口，例如`java.lang.Thread`中的`private native void start0()`方法。

Java虚拟机规范并没有明确要求本地方法栈的使用语言、具体实现方式、数据结构等。因此虚拟机可以根据需要自由实现它，甚至有的Java虚拟机（譬如Hot-Spot虚拟机）直接就把本地方法栈和虚拟机栈合二为一。与虚拟机栈一样，本地方法栈会在栈深度溢出或者栈扩展失败时分别抛出StackOverflowError和OutOfMemoryError异常。

## 3.4 堆

对Java应用程序而言，Java堆（Java Heap）是虚拟机管理的内存中最大的一块。Java堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，Java 世界里“几乎”所有的对象实例都在这里分配内存。

> 注意：由于逃逸分析，栈上分配和标量替换等技术的发展，导致并不是所有对象实例一定要在堆中分配内存。

Java堆是垃圾收集器管理的主要区域，又称GC 堆（Garbage Collected Heap）。从垃圾回收的角度，由于现在收集器基本都采用分代垃圾收集算法，所以 Java 堆还可以细分为：新生代和老年代。新生代分细致一点有：Eden 空间、From Survivor、To Survivor 空间等。这样对堆进行分代划分是为了更好地回收内存，或者更快地分配内存。

如果从分配内存的角度看，所有线程共享的Java堆中可以划分出多个线程私有的分配缓冲区 （Thread Local Allocation Buffer，TLAB），以提升对象分配时的效率。

JDK 1.7及之前堆内存逻辑上分为三部分：新生代+老年代+**永久代**

| 新生代                                    | 老年代                  | 永久代          |
| ----------------------------------------- | ----------------------- | --------------- |
| Young Generation Space                    | Tenure generation space | Permanent Space |
| Young/New（又被划分为Eden区和Survivor区） | Old/Tenure              | Perm            |

Java 8及之后堆内存逻辑上分为三部分：新生代+养老区+**元空间**

| 新生代                                    | 老年代                  | 元空间     |
| ----------------------------------------- | ----------------------- | ---------- |
| Young Generation Space                    | Tenure generation space | Meta Space |
| Young/New（又被划分为Eden区和Survivor区） | Old/Tenure              | Meta       |

## 3.5 方法区

方法区（Method Area）与Java堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存等数据。上面我们已经提到过，方法区还有一个别名叫做**Non-Heap(非堆）**，目的就是要和堆分开。所以**方法区看作是一块独立于 Java 堆的内存空间**。

但实际上，JDK 1.7 之前，方法区正是在堆内存的永久代中实现的，但这样导致Java更容易遇到内存溢出问题。JDK 1.7时，就已经把原本（JDK 1.6）放在永久代的字符串常量池、静态变量等移出到堆；而到了 JDK 1.8，终于完全废弃了永久代的概念，改用在本地内存中实现的元空间（Metaspace）来代替，把JDK 1.7中永久代还剩余的内容（主要是类型信息）移到元空间中。

根据Java虚拟机规范，如果方法区无法满足新的内存分配需求时，也将抛出OutOfMemoryError异常。

>  方法区和永久代的关系
>
> > 《Java 虚拟机规范》只是规定了有方法区这么个概念和它的作用，并没有规定如何去实现它。那么，不同的 JVM 上方法区的实现是可以不同的。  **方法区和永久代的关系很像 Java 中接口和类的关系，类实现了接口，而永久代就是 HotSpot 虚拟机对虚拟机规范中方法区的一种实现方式。** 也就是说，永久代是 HotSpot 的概念，方法区是 Java 虚拟机规范中的定义，是一种规范，而永久代是一种实现，一个是标准一个是实现，其他的虚拟机实现并没有永久代这一说法。

# 四、垃圾回收器

## 4.1 垃圾回收算法

GC是英文词汇`Garbage Collection` 的缩写， 中文一般直译为 “ 垃圾收集 ”。垃圾回收器回收垃圾一般分为两阶段：（1）**Marking (标记)**，标记“存活”的对象；**Sweeping (清除)**， 清除"死亡"的垃圾对象，释放内存。

### 4.1.1 标记阶段

**引用计数算法**：对每个对象保存一个整型的引用计数器属性，用于记录对象被引用的情况。

* 优点：实现简单，垃圾对象便于标识；判定效率高，无延迟。
* 缺点：计数器增加存储开销；更新计数增加时间开销；**无法处理循环引用**问题。

**可达性分析算法**：又称根搜索算法、追踪性垃圾收集（Tracing Garbage Collection），其以根对象集合（GC Roots）为起始点，按照从上至下的方式搜索被根对象集合所连接的目标对象是否可达。如果目标对象没有任何引用链（Reference Chain，即搜索所走过的路径）相连，则是不可达的，意味着该对象己经死亡，可以标记为垃圾对象。

 <div align="center"> <img src="..\images\jvm\可达性.png" width="600px"></div>

* 优点：实现简单和执行高效；能有效处理循环引用的问题。

### 4.1.2 清除阶段

**(1) 标记-清除算法**

标记-清除算法(Mark-Sweep)思想很简单，就是直接清除掉垃圾对象（注意，标记的是可达的对象，不是垃圾对象）。

这种算法需要使用空闲表(**free-list**)，来记录所有的空闲区域，以及每个区域的大小。维护空闲表增加了对象分配时的开销。

* 如果内存规整，采用指针碰撞的方式进行内存分配；内存不规整，虚拟机需要维护一个列表，空闲列表分配。
* 清理出来的空闲内存大多是不连续的，产生内存碎片，仍然无法存放大对象。
* GC时需要暂停整个应用程序线程（STW）。

 <div align="center"> <img src="..\images\jvm\标记清除算法.png" width="800px"></div>

**(2) 标记复制算法**

**标记-复制算法(Mark and Copy)**核心思想是，将还存活着的对象复制到另外一块内存上面，然后再把已使用过的内存空间一次清理掉。

* 优点：实现简单，运行高效；保证了空间的连续性。
* 缺点：是将可用内存缩小为了原来的一半，空间浪费未免太多；对象存活率较高时就要进行较多的复制操作，效率会大大降低。

 <div align="center"> <img src="..\images\jvm\标记复制算法.png" width="700px"></div>

**(3) 标记-整理算法**

标记-整理算法又称标记-压缩或标记-清除-压缩（Mark-Sweep-Compact）算法，其核心思想是让所有存活的对象都向内存空间一端移动（压缩到一端），然后直接清理掉边界以外的内存。

* 优点：消除了标记-清除算法当中，内存区域分散的缺点；消除了复制算法当中，内存减半的高额代价。
* 缺点：GC时需要暂停整个应用程序线程（STW）；如果被移动对象被其他对象引用，则还需要调整引用的地址。

 <div align="center"> <img src="..\images\jvm\标记整理算法.png" width="700px"></div>

**三者之间比较**

| 比较         | 标记-清除          | 标记-整理        | 标记-复制                             |
| ------------ | ------------------ | ---------------- | ------------------------------------- |
| **速率**     | 中等               | 最慢             | 最快                                  |
| **空间开销** | 少（但会堆积碎片） | 少（不堆积碎片） | 通常需要活对象的2倍空间（不堆积碎片） |
| **移动对象** | 否                 | 是               | 是                                    |

> 没有最好的算法，只有最合适的算法。

**扩展——其他算法**：

分代收集算法（Generational Collecting）：对不同生命周期的对象可以采取不同的收集方式，以便提高回收效率。目前几乎所有的GC都采用分代收集(Generational Collecting)算法执行垃圾回收的。

增量收集算法（Incremental Collecting）：让垃圾收集线程和应用程序线程交替执行。每次，垃圾收集线程只收集一小片区域的内存空间，接着切换到应用程序线程。依次反复，直到垃圾收集完成。

* 优点：对线程间冲突的妥善处理。
* 缺点：造成系统吞吐量的下降。

## 4.2 常见GC

JDK 1.8之前，主要有7款不同的垃圾回收器。

* 新生代收集器：Serial、ParNew、Parallel Scavenge

* 老年代收集器：Serial Old、Parallel Old、CMS

* 整堆收集器：G1

 <div align="center"> <img src="..\images\jvm\20201015092111694.png" width="600px"></div>

目前来说，常用的GC组合有：

（1）`Serial+Serial Old` 实现单线程的低延迟垃圾回收机制； 

（2）`ParNew+CMS`，实现多线程的低延迟垃圾回收机制； 

（3）`Parallel Scavenge+Parallel Old`，实现多线程的高吞吐量垃圾回收机制。

### 4.2.1 Serial GC：串行回收

Serial GC（`Serial/Serial Old`组合）是最基本、历史最悠久的垃圾收集器，其对年轻代使用标记-复制算法，对老年代使用标记-整理算法，年轻代和老年代的垃圾回收都会触发STW。适用于可用内存一般不大（几十MB至一两百MB），可以在较短时间内完成垃圾收集（几十ms至一百多ms）的场景。

 <div align="center"> <img src="..\images\jvm\Serial.png" width="650px"></div>

启动参数：`-XX:+UseSerialGC`

### 4.2.2 ParNew GC：并行回收

ParNew GC（`ParNew/Serial Old` 组合）对年轻代使用标记-复制算法 ，在老年代使用标记-整理算法 ，是Serial GC的多线程版本，年轻代和老年代的垃圾回收都会触发STW。对于新生代，回收次数频繁，使用并行方式高效；对于老年代，回收次数少，使用串行方式节省资源。

 <div align="center"> <img src="..\images\jvm\ParNew.png" width="650px"></div>

启动参数：`-XX:+UseParNewGC`，还可通过命令行参数`-XX:ParallelGCThreads=N`来指定 GC 线程数，其默认值为CPU核心数。

### 4.2.3 Parallel GC：吞吐量优先

Parallel GC（`Parallel Scavenge/Parallel Old`组合）对年轻代使用标记-复制算法 ，在老年代使用标记-整理算法，同样具备并行回收功能，年轻代和老年代的垃圾回收都会触发STW。

* 和ParNew GC不同，Parallel GC采用自适应调节策略，目的则是**达到一个可控制的吞吐量（Throughput）**，它也被称为吞吐量优先的垃圾收集器。
* **在Java8中，默认是此垃圾收集器**。

 <div align="center"> <img src="..\images\jvm\Parallel.png" width="650px"></div>

启动参数：`-XX:+UseParallelGC -XX:+UseParallelOldGC`。

### 4.2.4 CMS：低延迟

CMS GC的官方名称为 **“Mostly Concurrent Mark and Sweep Garbage Collector”（最大并发-标记-清除-垃圾收集器）**。其对年轻代采用并行STW方式的标记-复制算法 ，对老年代主要使用并发**标记-清除**算法 ，**它第一次实现了让垃圾收集线程与用户线程同时工作**。

* 优点：并发收集、低延迟。
* 缺点：老年代内存碎片。

 <div align="center"> <img src="..\images\jvm\CMS.png" width="750px"></div>

启动参数：`-XX:+UseConcMarkSweepGC`。

CMS整个过程比之前的收集器要复杂，整个过程分为6个主要阶段：

- **初始标记**（Initial-Mark）阶段：程序中所有的工作线程都将会因为**STW**机制而出现短暂的暂停，这个阶段的主要任务仅仅只是**标记出GC Roots能直接关联到的对象**。一旦标记完成之后就会恢复之前被暂停的所有应用线程。由于直接关联对象比较小，所以这里的速度非常快。

- **并发标记**（Concurrent-Mark）阶段：在此阶段，CMS GC 遍历老年代，标记所有的存活对象，从前一阶段 “Initial Mark” 找到的根对象开始算起。 “并发标记”阶段，就是与应用程序同时运行，不用暂停的阶段。

- **并发预处理**（Concurrent-Preclean）阶段：此阶段同样是与应用线程并发执行的，不需要停止应用线程。 因为前一阶段【并发标记】与程序并发运行，可能有一些引用关系已经发生了改变。如果在并发标记过程中 引用关系发生了变化，JVM 会通过“Card（卡片）”的方式将发生了改变的区域标记为“脏”区，这就是所谓的卡片标记（Card Marking）。

- **最终标记**（Final-Remark）阶段：最终标记阶段是此次 GC 事件中的**第二次（也是最后一次）STW 停顿**。本阶段的目标是完成老年代中所有存活对象的标记。因为之前的预清理阶段是并发执行的，有可能 GC 线程跟不上应用程 序的修改速度。所以需要一次 STW 暂停来处理各种复杂的情况。 通常 CMS 会尝试在年轻代尽可能空的情况下执行 Final Remark 阶段，以免连续触发多次 STW 事件。

- **并发清除**（Concurrent-Sweep）阶段：此阶段与应用程序并发执行，不需要 STW 停顿。JVM 在此阶段删除不再使用的对象，并回收他们占用的内存空间。

- **并发重置**（Concurrent Reset）阶段：此阶段与应用程序并发执行，重置 CMS 算法相关的内部 数据，为下一次 GC 循环做准备。

### 4.2.5 G1：垃圾优先

G1 的全称是 Garbage-First，意为垃圾优先，哪一块的垃圾最多就优先清理它。G1 GC 最主要的设计目标是：将 STW 停顿的时间和分布，变成可预期且可配置的。

使用G1收集器时，它将整个Java堆划分成**约2048**个大小相同的独立堆区域(smaller heap regions)，每个Region块，可能一会被定义成 Eden区，一会被指定为 Survivor区或者Old 区。在逻辑上，所有的 Eden 区和 Survivor 区合起来 就是年轻代，所有的 Old 区拼在一起那就是老年代。

G1垃圾收集器还增加了一种新的内存区域，叫做Humongous内存区域，如图中的H块。主要用于存储大对象，如果超过1.5个region，就放到H。

 <div align="center"> <img src="..\images\jvm\20201015092110555.png" width="600px"></div>

启动参数`-XX:+UseG1GC`。

G1 GC的垃圾回收过程主要包括如下三个环节：

**年轻代模式转移暂停（Evacuation Pause）**：当年轻代空间用满后，应用线程会被暂停（STW），年轻代内存块中的存活对象被拷贝到存活区（标记-复制算法）。如果还没有存活区，则任意选择一部分空闲的内存块作为存活区。 拷贝的过程称为转移(Evacuation)，这和前面介绍的其他年轻代收集器是一样的工作原理。

**并发标记（Concurrent Marking）**： G1并发标记的过程与CMS基本上是一样的。

* **阶段 1: Initial Mark(初始标记)** ：标记所有从GC根对象直接可达的对象。在CMS中需要一次STW暂停，但G1里面通常是在转移暂停的同时处理这些事情，所以它的开销是很小的。 

  **阶段 2: Root Region Scan(Root区扫描)**：此阶段标记所有从 "根区域" 可达的存活对象。根区域包括：非空的区域，以及在标记过程中不得不收集的区域。因为在并发标记的过程中迁移对象会造成很多麻烦，所以此阶段必须在下一次转移暂停之前完成。如果必须 启动转移暂停，则会先要求根区域扫描中止，等它完成才能继续扫描。在当前版本的实现中，根区域是存活 的小堆块：包括下一次转移暂停中肯定会被清理的那部分年轻代小堆块。 

  **阶段 3: Concurrent Mark(并发标记)**：此阶段和CMS的并发标记阶段非常类似：只遍历对象图，并在一个特殊的位图中标记能访问到的对象。 为了确保标记开始时的快照准确性，所有应用线程并发对对象图执行引用更新，G1 要求放弃前面阶段为了 标记目的而引用的过时引用。 

  **阶段 4: Remark(再次标记)**：和CMS类似，这是一次STW停顿(因为不是并发的阶段)，以完成标记过程。G1收集器会短暂地停止应用线程，停止并发更新信息的写入，处理其中的少量信息，并标记所有在并发标 记开始时未被标记的存活对象。 

  **阶段 5: Cleanup(清理)**：这一阶段也执行某些额外的清理，如引用处理或者类卸载(class unloading)。 最后这个清理阶段为即将到来的转移阶段做准备，统计小堆块中所有存活的对象，并将小堆块进行排序，以提升GC的效率（标记-整理算法）。此阶段也为下一次标记执行必需的所有整理工作(house-keeping activities)：维护并发标记 的内部状态。 

  * 所有不包含存活对象的小堆块在此阶段都被回收了。有一部分任务是并发的：例如空堆区的回收，还有大部分的存活率计算。此阶段也需要一个短暂的STW暂停，才能不受应用线程的影响并完成作业。

**转移暂停: 混合模式（Evacuation Pause (mixed)）**：并发标记完成之后，G1将执行一次**混合收集（mixed collection）**，就是不只清理年轻代，还将一部分老年代区域也加入到回收集 中。混合模式的转移暂停不一定紧跟并发标记阶段。有很多规则和历史数据会影响混合模式的启动时机。比如，假若在老年代中可以并发地腾出很多的小堆块，就没有必要启动混合模式。因此，在并发标记与混合转移暂停之间，很可能会存在多次`young`模式的转移暂停。 具体添加到回收集的老年代小堆块的大小及顺序，也是基于许多规则来判定的。 其中包括指定的软实时性 能指标，存活性，以及在并发标记期间收集的GC效率等数据，外加一些可配置的JVM选项。混合收集的过 程，很大程度上和前面的`fully-young gc`是一样的。

## 4.3 GC总结

| 垃圾收集器             | 分类 | 作用位置       | 算法                    | 特点         | 使用场景                               |
| ---------------------- | ---- | -------------- | ----------------------- | ------------ | -------------------------------------- |
| Serial                 | 串行 | 新生代         | 标记-复制               | 响应速度优先 | 单CPU环境下的client模式                |
| Serial Old             | 串行 | 老年代         | 标记-整理               | 响应速度优先 | 单CPU环境下的client模式（CMS后备方案） |
| ParNew                 | 并行 | 新生代         | 标记-复制               | 响应速度优先 | 多CPU环境Server模式下与CMS配合使用     |
| Parallel <br/>Scavenge | 并行 | 新生代         | 标记-复制               | 吞吐量优先   | 后台运算而不需要太多交互的场景         |
| Parallel Old           | 并行 | 老年代         | 标记-整理               | 吞吐量优先   | 后台运算而不需要太多交互的场景         |
| CMS                    | 并发 | 老年代         | 标记-清除               | 响应速度优先 | 互联网站或B/S业务                      |
| G1                     | 并发 | 新生代、老年代 | 标记-复制<br/>标记-压缩 | 响应速度优先 | 面向服务端应用，将来替换CMS            |

JDK11与JDK12起，分别开始支持ZGC与Shennandoah GC：

* ZGC（Z Garbage Collector）: 通过着色指针和读屏障，实现几乎全部的并发执行，几毫秒级别的延 迟，线性可扩展；
* Shenandoah: G1的改进版本，跟ZGC类似。

可以看出GC算法和实现的演进路线：

* `串行 -> 并行`: 重复利用多核CPU的优势，大幅降低GC暂停时间，提升吞吐量。
* `并行 -> 并发`: 不只开多个GC线程并行回收，还将GC操作拆分为多个步骤，让很多繁重的任务和应用线 程一起并发执行，减少了单次GC暂停持续的时间，这能有效降低业务系统的延迟。
* `CMS -> G1`: G1可以说是在CMS基础上进行迭代和优化开发出来的。修正了CMS一些存在的问题，而 且在GC思想上有了重大进步，也就是划分为多个小堆块进行增量回收，这样就更进一步地降低了单次 GC暂停的时间。 可以发现，随着硬件性能的提升，业界对延迟的需求也越来越迫切。
* `G1 -> ZGC`: ZGC号称无停顿垃圾收集器，这又是一次极大的改进。ZGC和G1有一些相似的地方，但是 底层的算法和思想又有了全新的突破。 ZGC把一部分GC工作，通过读屏障触发陷阱处理程序的方式， 让业务线程也可以帮忙进行GC。这样业务线程会有一点点工作量，但是不用等，延迟也被极大地降下 来了。

**GC选择**

选择正确的 GC，唯一可行的方式就是去尝试，一般性的指导原则： 

* 如果系统考虑吞吐优先，CPU 资源都用来最大程度处理业务，用 Parallel GC；
* 如果系统考虑低延迟，每次 GC 时间尽量短，用 CMS GC；
* 如果系统内存堆较大，同时希望整体来看平均 GC 时间可控，使用 G1 GC。 

对于内存大小的考量： 

* 一般 4G 以上，算是比较大，用 G1 的性价比较高。 
* 一般超过 8G，比如 16G-64G 内存，非常推荐使用 G1 GC。

最后需要明确一个观点：

- 没有最好的收集器，更没有万能的收集器。
- 调优永远是针对特定场景、特定需求，不存在一劳永逸的收集器。

**JDK8 的默认 GC 是什么？ JDK9，JDK10，JDK11…等等默认的 GC 是什么？**

虽然推出了这么多不同的垃圾回收器，但是每个JDK版本默认GC变化不多：

- 在JDK 7，默认是`Parallel Scavenge + Serial Old`。
- 在JDK 8 及JDK 7u4之后的版本，默认是`Parallel Scavenge + Parallel Old`。
- 在JDK 9 到JDK 16，默认是`G1`。

# 五、执行引擎

## 5.1 Java代码编译和执行过程

**执行引擎（Execution Engine）**是Java虚拟机核心的组成部分之一，它的任务就是**将字节码指令解释/编译为对应平台上的本地机器指令**。简单来说，JVM中的执行引擎充当了将高级语言翻译为机器语言的译者。

 <div align="center"> <img src="..\images\jvm\执行引擎.png" width="500px"></div>

至于如和把高级语言翻译成机器语言，Java提供了两种选择：

* 第一种是将源代码编译成字节码文件，然后在运行时通过解释器将字节码文件转为机器码执行。
* 第二种是编译执行（直接编译成机器码）。现代虚拟机为了提高执行效率，会使用**即时编译技术（JIT，Just In Time）将**方法编译成机器码后再执行.

HotSpot VM是目前市面上高性能虚拟机的代表作之一。它**采用解释器与即时编译器并**存的架构。在Java虚拟机运行时，解释器和即时编译器能够相互协作，各自取长补短，尽力去选择最合适的方式来权衡编译本地代码的时间和直接解释执行代码的时间。

 <div align="center"> <img src="..\images\jvm\202010120947139.png" width="750px"></div>

**什么是解释器**

解释器（Interpreter）：当Java虚拟机启动时会根据预定义的规范对字节码采用逐行解释的方式执行，将每条字节码文件中的内容“翻译”为对应平台的本地机器指令执行。

**什么是JIT编译器**

JIT（Just In Time Compiler）编译器：就是虚拟机将源代码直接编译成和本地机器平台相关的机器语言。

**为什么Java是半编译半解释型语言**

现在JVM在执行Java代码的时候，通常都会将解释执行与编译执行二者结合起来进行。我们编写的`.java`源文件先是通过`javac`编译器编译成`bytecode`字节码文件，然后通过JVM内嵌的解释器将字节码文件转换成机器码。但是，我们常用的JVM例如`Hotspot JVM`中，提供了 JIT（Just-In-Time）编译器，也就是所谓的动态编译器。**JIT编译器**能够在运行时将热点代码编译成机器码，这种情况下部分热点代码就是属于编译执行了。

**为什么还需要再使用解释器来“拖累”程序的执行性能呢？**比如JRockit VM内部就不包含解释器，字节码全部都依靠JIT编译器编译后执行。

- JRockit虚拟机只采JIT编译器。那是因为呢JRockit只部署在服务器上，一般有时间让它进行指令编译的过程了，对于响应来说要求不高，等JIT编译器的编译完成后，就会提供更好的性能。

我们需要明确的是，当程序启动后，解释器可以马上发挥作用，省去编译的时间，立即执行。编译器要想发挥作用，把代码编译成本地代码，需要一定的编译时间；但编译为本地代码后，执行效率高。

所以，尽管JRockit VM中程序的执行性能会非常高效，但程序在启动时必然需要花费更长的时间来进行编译。对于服务端应用来说，启动时间并非是关注重点，但对于那些看中启动时间的应用场景而言，或许就需要采用解释器与即时编译器并存的架构来换取一个平衡点。

**HotSpot JVM执行方式**

当虚拟机启动的时候，**解释器可以首先发挥作用**，而不必等待JIT编译器全部编译完成再执行，这样可以**省去许多不必要的编译时间**。并且随着程序运行时间的推移，即时编译器逐渐发挥作用，根据**热点探测**功能，将**有价值的字节码编译为本地机器指令**，以换取更高的程序执行效率。

## 5.2 编译器分类

当我们谈论编译器时，我们具体在讨论什么呢？

### 5.2.1 前端编译器

Java 语言的“编译期”其实是一段“不确定”的操作过程，因为它可能是指一个**前端编译器**（其实叫“编译器的前端”更准确一些）把`.java`文件转变成`.class`文件的过程。常见前端编译器：Sun的**javac**、Eclipse JDT中的增量式编译器（ECJ）。

### 5.2.2 JIT编译器

也可能是指虚拟机的**后端运行期编译器**（JIT编译器，Just In Time Compiler），把字节码转变成机器码的过程。常见JIT编译器：HotSpot VM的C1、C2编译器。

- `-client`：指定Java虚拟机运行在Client模式下，并使用C1编译器；
  - C1编译器会对字节码进行**简单和可靠的优化**，耗时短。以达到更快的编译速度。

- `-server`：指定Java虚拟机运行在server模式下，并使用C2编译器。
  - C2进行**耗时较长的优化，以及激进优化**。但优化的代码执行效率更高。（使用C++）

### 5.2.3 AOT编译器

还可能是指使用**静态提前编译器**（AOT编译器，Ahead of Time Compiler）。所谓AOT编译，是与即时编译相对立的一个概念。我们知道，即时编译指的是在程序的运行过程中，将字节码转换为可在硬件上直接运行的机器码，并部署至托管环境中的过程。而AOT编译指的则是，**在程序运行之前，便将字节码转换为机器码**的过程。常见AOT 编译器有：GNU Compiler for the Java（GCJ）、Excelsior JET。

```java
.java -> .class -> (使用jaotc) -> .so
```

JDK9引入了AOT编译器和实验性AOT编译工具**aotc**。它借助了Graal编译器，将所输入的Java类文件转换为机器码，并存放至生成的动态共享库之中。AOT编译器的好处：Java虚拟机加载已经预编译成二进制库，可以直接执行；不必等待及时编译器的预热，减少Java应用给人带来“第一次运行慢” 的不良体验。

缺点：

- 破坏了 java  “ 一次编译，到处运行”，必须为每个不同的硬件，OS编译对应的发行包。
- **降低了Java链接过程的动态性**，加载的代码在编译器就必须全部已知。
- 还需要继续优化中，最初只支持Linux X64 java base。

# 六、JVM启动参数

JVM启动参数的前缀主要有`-`、`-D`、`-X`、`-XX`、`+/-`、`:`

* 以`-`开头为标准参数，所以的JVM都要实现这些参数，并且向后兼容。例如`-server`。

* `-D`设置系统属性。例如`-Dfile.encoding=UTF-8`。

* 以`-X`开头为非标准参数，基本都传给JVM，默认 JVM 实现这些参数的功能，但是并不保证所 有 JVM 实现都满足，且不保证向后兼容。 可以使 用 java -X 命令来查看当前 JVM 支持的非标准参数。例如`-Xmx8g`。

* 以`-XX`开头的为非稳定参数，专门用于控制JVM的行为，跟具体的JVM实现有关，随时可能在下一个版本中取消。

* `-XX：+/-Flags` 形式，`+/- `是对布尔值进行开关。例如`-XX:+UseG1GC`。

* `-XX：key=value `形式， 指定某个选项的值。例如`-XX:MaxPermSize=256m`。

## 6.1 系统属性参数

```java
-Dfile.encoding=UTF-8
-Duser.timezone=GMT+08
-Dmaven.test.skip=true
-Dio.netty.eventLoopThreads=8
```

## 6.2 运行模式参数

* `-server`：设置 JVM 使用 server 模式，特点是启动速度比较慢，但运行时性能和内存管理效率 很高，适用于生产环境。在具有 64 位能力的 JDK 环境下将默认启用该模式，而忽略 -client 参 数。
* `-client` ：JDK1.7 之前在32位的 x86 机器上的默认值是 -client 选项。设置 JVM 使用 client 模 式，特点是启动速度比较快，但运行时性能和内存管理效率不高，通常用于客户端应用程序或 者 PC 应用开发和调试。此外，我们知道 JVM 加载字节码后，可以解释执行，也可以编译成本 地代码再执行，所以可以配置 JVM 对字节码的处理模式。
* `-Xint`：在解释模式（interpreted mode）下运行，`-Xint` 标记会强制 JVM 解释执行所有的字节 码，这当然会降低运行速度，通常低10倍或更多。 
* `-Xcomp`：`-Xcomp` 参数与`-Xint` 正好相反，JVM 在第一次使用时会把所有的字节码编译成本地 代码，从而带来最大程度的优化。【注意预热】
* `-Xmixed`：`-Xmixed` 是混合模式，将解释模式和编译模式进行混合使用，有 JVM 自己决定，这 是 JVM 的默认模式，也是推荐模式。 我们使用 java -version 可以看到 mixed mode 等信息。

## 6.3 堆内存设置参数

**（1）显式指定堆内存`–Xms`和`-Xmx`**

* `-Xmx`，指定最大堆内存。 如 -Xmx4g。这只是限制了 Heap 部分的最大值为 4g。这个内存不包括栈内存，也不包括堆外使用的内存。 
* `-Xms`，指定堆内存空间的初始大小。 如 -Xms4g。 

> 通常会将`-Xms`和`-Xmx`两个参数配置相同的值，其目的是**为了能够在Java垃圾回收机制清理完堆区后不需要重新分隔计算堆区的大小，从而提高性能**。

**（2）显式新生代内存**

默认情况下，YG 的最小大小为 1310 MB，最大大小为无限制。

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

**（3）显示指定永久代/元空间的大小**

* **从Java 8开始，如果我们没有指定 Metaspace 的大小，随着更多类的创建，虚拟机会耗尽所有可用的系统内存（永久代并不会出现这种情况）。**

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

**（4）其他常用参数**

*  `-XX：MaxDirectMemorySize=size`，系统可以使用的最大堆外内存，这个参数跟 `-Dsun.nio.MaxDirectMemorySize` 效果相同。
*  `-Xss`, 设置每个线程栈的字节数，影响栈的深度。 例如 `-Xss1m` 指定线程栈为 1MB，与`-XX:ThreadStackSize=1m` 等价。

下面我画了一张图，这样有助力理清`Xmx、Xms、Xmn、Meta、 DirectMemory、Xss `这些内存参数的关系：

 <div align="center"> <img src="..\images\jvm\JVM启动参数.png" width="850px"></div>

## 6.4 GC设置参数

### 6.4.1 垃圾回收器

**垃圾收集器的组合关系**

<div align="center"> <img src="..\images\jvm\20201015092111694.png" width="600px"></div>

新生代收集器：Serial、ParNew、Parallel Scavenge；老年代收集器：Serial Old、Parallel Old、CMS；整堆收集器：G1。

> 两个收集器间有连线，表明它们可以搭配使用：Serial/Serial Old、Serial/CMS、ParNew/Serial Old、ParNew/CMS、Parallel Scavenge/Serial Old、Parallel Scavenge/Parallel Old、G1。

```java
-XX:+UseSerialGC——使用串行垃圾回收器（Serial/Serial Old 组合）
-XX:+UseParallelGC——使用并行垃圾回收器（ParNew/Serial Old 组合）
-XX:+UseParallelGC -XX:+UseParallelOldGC——使用并行垃圾回收器（Parallel Scavenge/Parallel Old组合）
-XX:+UseConcMarkSweepGC——使用 CMS 垃圾回收器（只能与ParNew/Serial搭配）
-XX:+UseG1GC——使用 G1 垃圾回收器
// Java 11+
-XX:+UnlockExperimentalVMOptions -XX:+UseZGC——使用ZGC垃圾回收器
// Java 12+
-XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC——使用Shenandoah垃圾回收器
```

### 6.4.2 GC记录

为了严格监控应用程序的运行状况，我们应该始终检查JVM的垃圾回收性能。最简单的方法是以人类可读的格式记录*GC*活动。

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

## 6.5 分析诊断参数

* `-XX：+-HeapDumpOnOutOfMemoryError` 选项，当 `OutOfMemoryError` 产生，即内存溢出（堆内存或持久代) 时，自动 Dump 堆内存。
  * 示例用法： `java -XX:+HeapDumpOnOutOfMemoryError -Xmx256m ConsumeHeap `
* `-XX：HeapDumpPath` 选项，与 `HeapDumpOnOutOfMemoryError` 搭配使用，指定内存溢出时 Dump 文件的 目录。 如果没有指定则默认为启动 Java 程序的工作目录。 
  * 示例用法：` java -XX:+HeapDumpOnOutOfMemoryError `
* `-XX:HeapDumpPath=/usr/local/ ConsumeHeap` 自动 Dump 的 hprof 文件会存储到 `/usr/local/` 目录下。 
* `-XX：OnError` 选项，发生致命错误时（fatal error）执行的脚本。例如, 写一个脚本来记录出错时间, 执行一些命令，或者 curl 一下某个在线报警的 url。 
  * 示例用法：`java -XX:OnError="gdb - %p" MyApp` 可以发现有一个 %p 的格式化字符串，表示进程 PID。 
* `-XX：OnOutOfMemoryError` 选项，抛出 `OutOfMemoryError `错误时执行的脚本。 `-XX：ErrorFile=filename` 选项，致命错误的日志文件名,绝对路径或者相对路径。 
* `-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=1506`，远程调试。

## 6.6 JavaAgent参数

Agent 是 JVM 中的一项黑科技，可以通过无侵入方式来做很多事情，比如注入 AOP 代码，执行统 计等等，权限非常大。这里简单介绍一下配置选项，详细功能需要专门来讲。 设置 agent 的语法如下： 

* `-agentlib:libname[=options] `启用 native 方式的 agent，参考 LD_LIBRARY_PATH 路径。 
* `-agentpath:pathname[=options]` 启用 native 方式的 agent。 
* `-javaagent:jarpath[=options] `启用外部的 agent 库，比如 pinpoint.jar 等等。
* `-Xnoagent` 则是禁用所有 agent。 以下示例开启 CPU 使用时间抽样分析： 
  * `JAVA_OPTS="-agentlib:hprof=cpu=samples,file=cpu.samples.log"`

# 七、命令行及图形工具

## 7.1 命令行工具

**（1）jps：查看所有 Java 进程**

**`jps`** (JVM Process Status）命令类似 UNIX 的 `ps` 命令。用于查看所有 Java 进程的启动类、传入参数和 Java 虚拟机参数等信息；

`jps`：显示虚拟机执行主类名称以及这些进程的本地虚拟机唯一 ID（Local Virtual Machine Identifier, LVMID）。

`jps -q` ：只输出进程的本地虚拟机唯一 ID。

`jps -l`：输出主类的全名，如果进程执行的是 Jar 包，输出 Jar 路径。

`jps -v`：输出虚拟机进程启动时 JVM 参数。

`jps -m`：输出传递给 Java 进程 main() 函数的参数。
`jps -mlv`： 打印Java 各个进程的详尽参数。

**（2）jinfo：实时地查看和调整虚拟机各项参数**

`jinfo vmid` ：输出当前 jvm 进程的全部参数和系统属性 (第一部分是系统的属性，第二部分是 JVM 的参数)。

`jinfo -flag name vmid` ：输出对应名称的参数的具体值。比如输出 MaxHeapSize、查看当前 jvm 进程是否开启打印 GC 日志 ( `-XX:PrintGCDetails` ：详细 GC 日志模式，这两个都是默认关闭的)。

使用 jinfo 可以在不重启虚拟机的情况下，可以动态的修改 jvm 的参数：`jinfo -flag [+|-]name vmid` 开启或者关闭对应名称的参数。

**（3）jstat：监视虚拟机各种运行状态信息**

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

**（4）jmap：生成堆转储快照**

`jmap`（Memory Map for Java）命令用于生成堆转储快照。 如果不使用 `jmap` 命令，要想获取 Java 堆转储，可以使用 `“-XX:+HeapDumpOnOutOfMemoryError”` 参数，可以让虚拟机在 OOM 异常出现之后自动生成 dump 文件，Linux 命令下可以通过 `kill -3` 发送进程退出信号也能拿到 dump 文件。

`jmap` 的作用并不仅仅是为了获取 dump 文件，它还可以查询 finalizer 执行队列、Java 堆和永久代的详细信息，如空间使用率、当前使用的是哪种收集器等。和`jinfo`一样，`jmap`有不少功能在 Windows 平台下也是受限制的。

`jmap`常用选项为3个：

* `-heap pid`  ：打印堆内存（/内存池）的配置和 使用信息。 
* `-histo pid` ：看哪些类占用的空间最多, 直方图。 
* `-dump:format=b,file=xxxx.hprof pid`：生成堆转储快照。

**（5）jhat：分析 heapdump 文件**

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

**（6）jstack ：生成虚拟机当前时刻的线程快照**

`jstack`（Stack Trace for Java）命令用于生成虚拟机当前时刻的线程快照。线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合.

生成线程快照的目的主要是定位线程长时间出现停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等都是导致线程长时间停顿的原因。线程出现停顿的时候通过`jstack`来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做些什么事情，或者在等待些什么资源。

常见使用选项有：

* `-F `强制执行 thread dump，可在 Java 进程卡死 （hung 住）时使用，此选项可能需要系统权限。 
* `-m` 混合模式（mixed mode)，将 Java 帧和 native 帧一起输出，此选项可能需要系统权限。
* `-l` 长列表模式，将线程相关的 locks 信息一起输 出，比如持有的锁，等待的锁。

**我们可以通过 `jstack` 命令进行死锁检查，输出死锁信息，找到发生死锁的线程。**

**（7）jcmd：执行 JVM 相关分析命令（整合命令）**

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

## 7.2 JDK 可视化分析工具

**（1）jsonsole：Java 监视与管理控制台**

JConsole 是基于 JMX 的可视化监视、管理工具。可以很方便的监视本地及远程服务器的 java 进程的内存使用情况。你可以在控制台输出`console`命令启动或者在 JDK 目录下的 bin 目录找到`jconsole.exe`然后双击启动。

JConsole具体使用教程可参考：

* https://docs.oracle.com/javase/7/docs/technotes/guides/management/jconsole.html

**（2）jvisualvm**

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

 VisualVM 的具体使用可参考:

- <https://visualvm.github.io/documentation.html>
- <https://www.ibm.com/developerworks/cn/java/j-lo-visualvm/index.html>

**（3）VisualGC**

IDEA中有插件VisualGC，安装后即在右下角：

<div align="center"> <img src="https://img-blog.csdnimg.cn/20210529152219910.png" width="700px"></div>

**（4）jmc**

jmc是目前Oracle提供最强大的JVM可视化工具，它支持远程连接JVM（通过JMX连接如果想要用jmc监控远程的JVM进程，配置方式和jvisualvm方式一一样即可），除此之外它还支持Java飞行记录器。

<div align="center"> <img src="https://img-blog.csdnimg.cn/20210529152824420.png" width="700px"></div>

详细使用可查看[https://cs.xieyonghui.com/java/java-flight-recorder_72.html](https://cs.xieyonghui.com/java/java-flight-recorder_72.html)
