<!-- MarkdownTOC -->

- [1. Java内存模型](#1-java内存模型)
- [2. 程序计数器](#2-程序计数器)
- [3. 虚拟机栈](#3-虚拟机栈)
  - [3.1 概述](#31-概述)
  - [3.2 栈帧](#32-栈帧)
  - [3.3 栈帧内部结构](#33-栈帧内部结构)
    - [3.3.1 局部变量表](#331-局部变量表)
      - [(1) 变量槽slot](#1-变量槽slot)
      - [(2) 静态变量与局部变量的对比](#2-静态变量与局部变量的对比)
    - [3.3.2 操作数栈](#332-操作数栈)
      - [(1) 代码追踪](#1-代码追踪)
      - [(2) 栈顶缓存技术](#2-栈顶缓存技术)
    - [3.3.3 动态链接与方法返回地址](#333-动态链接与方法返回地址)
- [4. 本地方法栈](#4-本地方法栈)
- [5. 堆](#5-堆)
  - [5.1 堆内存划分](#51-堆内存划分)
    - [5.1.1 设置堆内存大小](#511-设置堆内存大小)
    - [5.1.2 新生代与老年代](#512-新生代与老年代)
  - [5.2 堆对象分配过程](#52-堆对象分配过程)
  - [5.3 Minor GC、Major GC、Full GC](#53-minor-gcmajor-gcfull-gc)
  - [5.4 堆空间分代](#54-堆空间分代)
    - [5.4.1 内存分配策略](#541-内存分配策略)
    - [5.4.2 对象分配内存：TLAB](#542-对象分配内存tlab)
    - [5.4.3 堆空间的参数设置](#543-堆空间的参数设置)
    - [5.4.4 逃逸分析](#544-逃逸分析)
      - [(1) 概述](#1-概述)
      - [(2) 栈上分配](#2-栈上分配)
      - [(3) 同步省略](#3-同步省略)
      - [(4) 分离对象和标量替换](#4-分离对象和标量替换)
      - [(5) 逃逸分析的不足](#5-逃逸分析的不足)
- [6. 方法区](#6-方法区)
  - [6.1 概述](#61-概述)
  - [6.2 方法区的内部结构](#62-方法区的内部结构)
    - [6.2.1 类信息](#621-类信息)
    - [6.2.2 方法信息](#622-方法信息)
    - [6.2.3 运行时常量池](#623-运行时常量池)
      - [(1) 常量池](#1-常量池)
      - [(2) 运行时常量池](#2-运行时常量池)
      - [(3) 图解实例](#3-图解实例)
  - [6.3 方法区的演进细节](#63-方法区的演进细节)
  - [6.4 方法区的垃圾回收](#64-方法区的垃圾回收)
- [7. 直接内存](#7-直接内存)
- [参考资料](#参考资料)

<!-- /MarkdownTOC -->

# 1. Java内存模型

内存是非常重要的系统资源，是硬盘和CPU的中间仓库及桥梁，承载着操作系统和应用程序的实时运行。

JVM是一个完整的计算机模型，所以自然就需要有对应的内存模型，这个模型被称为 “ Java内存模型 ”，对应的英文是“ Java Memory Model ”，简称 JMM 。Java内存模型规定了JVM应该如何使用计算机内存（RAM）。 广义来讲， Java内存模型分为两个部分：

*  JVM内存结构
* JMM与线程规范

其中，JVM内存结构是底层实现，其规定了Java在运行过程中内存申请、分配、管理的策略，保证了JVM的高效稳定运行，也是我们理解和认识JMM的基础。 大家熟知的堆内 存、栈内存等运行时数据区的划分就可以归为JVM内存结构。

**不同的JVM对于内存的划分方式和管理机制存在着部分差异**。结合JVM虚拟机规范，下面就来探讨一下经典的JVM内存布局。

JVM 内存主要分为**本地方法栈、程序计数器、虚拟机栈、堆、方法区**五个部分。这些区域有各自的用途和创建与销毁的时间。有的区域随着虚拟机进程的启动而一直存在，有些区域则是依赖用户线程的启动和结束而建立和销毁。

 <div align="center"> <img src=".\images\运行时数据区.png" width="900px"></div>

> 《Java 虚拟机规范》中指明：”尽管所有的方法区在逻辑上是属于堆的一部分，但一些简单的实现可能不会选择区进行垃圾收集或者进行压缩”。 但对于 HotSpotJVM 而言，方法区还有一个别名叫做**Non-Heap(非堆)**，目的就是要和堆分开。所以**方法区看作是一块独立于 Java 堆的内存空间**。
>
> 如上图所描述一样，在JDK 1.7 之前，方法区在**逻辑上**属于**非堆**，是一块独立的内存区域，但其实际物理内存实现其实是在堆区的(存放于永久代)；直到JDK 1.8后，方法区的实现才被移动至直接内存中的元空间(MetaSpace)中，逻辑内存与实际物理内存都与堆区分开。一般来讲，黄色部分为线程隔离的数据区域，其他部分为线程共享的区域。

一般来讲，黄色部分为线程隔离的数据区域，其他部分为线程共享的区域。

|    区域    | 是否线程共享 | 是否存在内存溢出 | 是否存在GC |
| :--------: | :----------: | :--------------: | :--------: |
| 本地方法栈 |      否      |        是        |     否     |
|  虚拟机栈  |      否      |        是        |     否     |
| 程序计数器 |      否      |        否        |     否     |
|    堆区    |      是      |        是        |     是     |
|   方法区   |      是      |        是        |     是     |

> **注意：程序计数器是唯一一个不会出现 `OutOfMemoryError` 的内存区域，它的生命周期随着线程的创建而创建，随着线程的结束而死亡。**

# 2. 程序计数器

**程序计数器**（Program Counter Register）是一块较小的内存空间，是**运行速度最快**的存储区域。它可以看作是当前线程所执行的字节码的行号指示器。Register的命名源于CPU的寄存器，存储指令相关的现场信息。这里，并非指广义上所指的物理寄存器，或许将其翻译为**PC寄存器**（或指令计数器）会更加贴切（也称为程序钩子）。**JVM中的程序计数器是对物理PC寄存器的一种抽象模拟**。

**程序计数器用来存放下一条指令的地址（将要执行的字节码指令地址）**。在JVM规范中，每个线程都有它自己的程序计数器，是**线程私有**的，生命周期与线程的生命周期保持一致。

任何时间一个线程都只有一个方法在执行，也就是所谓的当前方法。程序计数器会存储当前线程正在执行的Java方法的JVM指令地址；或者，如果是在执行native方法，则是未指定值（undefined）。

它是程序控制流的指示器，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令。

例如下面的字节码文件，发现在字节码的左边都有一个行号标识，它其实就是**指令地址**。

```bash
 0 bipush 10
 2 istore_1
 3 bipush 20
 5 istore_2
 6 iload_1
 7 iload_2
 8 iadd
 9 istore_3
10 return
```

**使用程序计数器存储字节码指令地址有什么用呢？**

因为CPU需要不停的切换各个线程，在线程切换回来以后，就得知道接着从哪开始继续执行。JVM的字节码解释器就需要通过改变程序计数器的值来明确下一条应该执行什么样的字节码指令。

**PC寄存器为什么被设定为线程私有的？**

多线程在一个特定的时间段内只会执行其中某一个线程的方法，CPU会不停地做任务切换，这样必然导致经常中断或恢复，如何保证分毫无差呢？**为了能够准确地记录各个线程正在执行的当前字节码指令地址，最好的办法自然是为每一个线程都分配一个PC寄存器**，这样一来各个线程之间便可以进行独立计算，从而不会出现相互干扰的情况。

由于**CPU时间片**轮限制，众多线程在并发执行过程中，任何一个确定的时刻，一个处理器或者多核处理器中的一个内核，只会执行某个线程中的一条指令。

这样必然导致经常中断或恢复，如何保证分毫无差呢？每个线程在创建后，都会产生自己的程序计数器和栈帧，程序计数器在各个线程之间互不影响。

**CPU时间片**

CPU时间片即CPU分配给各个程序的时间，每个线程被分配一个时间段，称作它的时间片。

在宏观上：可以同时打开多个应用程序，每个程序并行不悖，同时运行。

但在微观上：一个CPU一次只能处理程序要求的一部分。如何处理公平，一种方法就是引入时间片，每个程序轮流执行。

 <div align="center"> <img src=".\images\20201007083751805.png" width="400px"></div>

# 3. 虚拟机栈

## 3.1 概述

常常有人把Java内存区域笼统地划分为堆内存（Heap）和栈内存（Stack），这种划分方式直接继承自传统的C、C++程序的内存布局结构，在Java语言里就显得有些粗糙了，实际的内存区域划分要比这更复杂。“栈”通常就是指这里讲的虚拟机栈，可以明确的是——**栈是运行时的单位，而堆是存储的单位**。

- 栈解决程序的运行问题，即程序如何执行，或者说如何处理数据。
- 堆解决的是数据存储的问题，即数据怎么放，放哪里。

**那么Java虚拟机栈又是什么?**

每个Java虚拟机线程都有一个私有的**Java虚拟机栈(Java Virtual Machine Stack)**，与该线程同时创建，其内部保存一个个的**栈帧(Stack Frame)**，对应着每一次的Java方法调用。其生命周期和线程保持一致。虚拟机栈主管Java程序的运行，它保存方法的局部变量（8种基本数据类型、对象的引用地址）、部分结果，并参与方法的调用和返回。

栈是一种快速有效的分配存储方式，**访问速度仅次于程序计数器**。JVM直接对Java栈的操作只有两个：

- 每个方法执行，伴随着压栈（入栈、进栈）
- 执行结束后的出栈工作

**与Java虚拟机栈相关异常`StackOverFlowError` 和 `OutOfMemoryError`**

- **StackOverFlowError**： 若Java虚拟机栈不允许动态扩展，当线程请求栈的深度超过当前虚拟机栈的最大深度的时候，就抛出 StackOverFlowError 错误。
- **OutOfMemoryError**：若Java虚拟机栈可以动态扩展，当扩展时无法申请到足够的内存时会抛出OutOfMemoryError异常。

```java
public class StackErrorTest {
    private static int count = 1;
    public static void main(String[] args) {
        System.out.println(count++);
        main(args);// Exception in thread "main" java.lang.StackOverflowError错误
    }
}
```

**设置栈内存大小**

我们可以使用参数`-Xss`选项来设置线程的最大栈空间，栈的大小直接决定了函数调用的最大可达深度。

```bash
-Xss1m
-Xss1k
```

## 3.2 栈帧

栈的存储单位即栈帧。也就是说，每个线程都有自己的栈，栈中的数据都是以**栈帧（Stack Frame）**的格式存在。线程上正在执行的每个方法都各自对应一个栈帧（Stack Frame）。栈帧是一个内存区块，是一个数据集，维系着方法执行过程中的各种数据信息。

 <div align="center"> <img src=".\images\20201010192721945.png" width="800px"></div>

JVM直接对Java栈的操作只有两个，就是对栈帧的压栈和出栈，遵循“**先进后出/后进先出**”原则。

**在一条活动线程中，一个时间点上，只会有一个活动的栈帧。**即只有当前正在执行的方法的栈帧（栈顶栈帧）是有效的，这个栈帧被称为当前栈帧（Current Frame），与当前栈帧相对应的方法就是当前方法（Current Method），定义这个方法的类就是当前类（Current Class）。

执行引擎运行的所有字节码指令只针对当前栈帧进行操作。如果在该方法中调用了其他方法，对应的新的栈帧会被创建出来，放在栈的顶端，成为新的当前帧。

```java
public class StackFrameTest {
    public static void main(String[] args) {
        method01();
    }

    private static int method01() {
        System.out.println("方法1的开始");
        int i = method02();
        System.out.println("方法1的结束");
        return i;
    }

    private static int method02() {
        System.out.println("方法2的开始");
        int i = method03();
        System.out.println("方法2的结束");
        return i;
    }

    private static int method03() {
        System.out.println("方法3的开始");
        int i = 30;
        System.out.println("方法3的结束");
        return i;
    }
}
```

输出结果满足栈先进后出的概念，通过DEBUG，也能够看到栈相关信息。

```bash
方法1的开始
方法2的开始
方法3的开始
方法3的结束
方法2的结束
方法1的结束
```

**栈运行原理**

不同线程中所包含的栈帧是不允许存在相互引用的，即不可能在一个栈帧之中引用另外一个线程的栈帧。

如果当前方法调用了其他方法，方法返回之际，当前栈帧会传回此方法的执行结果给前一个栈帧，接着，虚拟机会丢弃当前栈帧，使得前一个栈帧重新成为当前栈帧。

Java方法有两种返回函数的方式，一种是正常的函数返回，使用**return指令**；另外一种是**抛出异常**。不管使用哪种方式，**都会导致栈帧被弹出**。

## 3.3 栈帧内部结构

每个栈帧中存储着：

- **局部变量表（Local Variables）**
- **操作数栈（Operand Stack）（或表达式栈）**
- 动态链接（Dynamic Linking）（或指向运行时常量池的方法引用）
- 方法返回地址（Return Address）（或方法正常退出或者异常退出的定义）
- 一些附加信息

并行每个线程下的栈都是私有的，因此每个线程都有自己各自的栈，并且每个栈里面都有很多栈帧，栈帧的大小主要由局部变量表和操作数栈决定的。

### 3.3.1 局部变量表

局部变量表（Local Variables），又称局部变量数组或本地变量表，是一个**数字数组**，主要用于存储**方法参数**和定义在方法体内的**局部变量**，这些数据类型包括基本数据类型（8种）、对象引用（reference），以及returnAddress（指向了一条字节码指令的地址）类型。

由于局部变量表是建立在线程的栈上，是线程的私有数据，因此**不存在数据安全问题**。

**局部变量表所需的容量大小是在编译期确定下来的**，并保存在方法的Code属性的`Maximum local variables`数据项中。在方法运行期间是不会改变局部变量表的大小的。

例如上面案例的方法1：

 <div align="center"> <img src=".\images\2020101019385339.png" width="800px"></div>

**方法嵌套调用的次数由栈的大小决定**。一般来说，**栈越大，方法嵌套调用次数越多**。对一个函数而言，它的参数和局部变量越多，使得局部变量表膨胀，它的栈帧就越大，以满足方法调用所需传递的信息增大的需求。进而函数调用就会占用更多的栈空间，导致其嵌套调用次数就会减少。

**局部变量表中的变量只在当前方法调用中有效**。在方法执行时，虚拟机通过使用局部变量表完成参数值到参数变量列表的传递过程。**当方法调用结束后，随着方法栈帧的销毁，局部变量表也会随之销毁**。

#### (1) 变量槽slot

参数值的存放总是在局部变量数组的index0开始，到数组长度-1的索引结束。

**局部变量表，最基本的存储单元是slot（变量槽）**。局部变量表中存放编译期可知的基本数据类型、引用类型、returnAddress类型的变量。

在局部变量表里，**32位以内的类型只占用一个slot**（包括returnAddress类型）**，64位的类型**（long和double）**占用两个slot**。

>byte、short、char 、boolean在存储前被转换为int。0表示false，非0表示true。

JVM会为局部变量表中的每一个slot都分配一个访问索引，通过这个索引即可成功访问到局部变量表中指定的局部变量值。

当一个实例方法被调用的时候，它的方法参数和方法体内部定义的局部变量将会按照顺序被复制到局部变量表中的每一个slot上。如果需要访问局部变量表中一个64位的局部变量值时，只需要使用前一个索引即可。（比如：访问long或doub1e类型变量）

如果当前帧是由构造方法或者实例方法创建的，那么该对象引用this将会存放在index为0的slot处，其余的参数按照参数表顺序继续排列。

**slot的重复利用**

**栈帧中的局部变量表中的槽位是可以重用的**，如果一个局部变量过了其作用域，那么在其作用域之后申明的新的局部变就很有可能会复用过期局部变量的槽位，从而达到节省资源的目的。

#### (2) 静态变量与局部变量的对比

变量的分类：

- 按数据类型分：基本数据类型、引用数据类型
- 按类中声明的位置分：成员变量（类变量，实例变量）、局部变量
  - 类变量：Linking的Prepare阶段，给类变量默认赋值，initial阶段给类变量显示赋值即静态代码块。
  - 实例变量：随着对象创建，会在堆空间中分配实例变量空间，并进行默认赋值。
  - 局部变量：在使用前必须进行显式赋值，不然编译不通过。

参数表分配完毕之后，再根据方法体内定义的变量的顺序和作用域分配。

我们知道类变量表有两次初始化的机会，第一次是在“**准备阶段**”，执行系统初始化，对类变量设置零值，另一次则是在“**初始化**”阶段，赋予程序员在代码中定义的初始值。

和类变量初始化不同的是，局部变量表不存在系统初始化的过程，这意味着一旦定义了局部变量则必须人为的初始化，否则无法使用。

```java
public void test(){
	int i;
	System.out.println(i);//报错，局部变量没有赋值不能使用。
}
```

在栈帧中，与性能调优关系最为密切的部分就是前面提到的局部变量表。在方法执行时，虚拟机使用局部变量表完成方法的传递。

**局部变量表中的变量也是重要的垃圾回收根节点，只要被局部变量表中直接或间接引用的对象都不会被回收**。

### 3.3.2 操作数栈

每一个独立的栈帧除了包含局部变量表以外，还包含一个后进先出（Last - In - First -Out）的 **操作数栈**，也可以称之为**表达式栈**（Expression Stack）。

操作数栈，在方法执行过程中，根据字节码指令，往栈中写入数据或提取数据，即**入栈（push）/出栈（pop）**。

- 某些字节码指令将值压入操作数栈，其余的字节码指令将操作数取出栈。使用它们后再把结果压入栈
- 比如：执行复制、交换、求和等操作

操作数栈，**主要用于保存计算过程的中间结果，同时作为计算过程中变量临时的存储空间**。

操作数栈就是JVM执行引擎的一个工作区，当一个方法刚开始执行的时候，一个新的栈帧也会随之被创建出来，这个方法的操作数栈是**空**的。

> 这个时候数组是有长度的，数组一旦创建，长度是不可变的。

每一个操作数栈都会拥有一个明确的栈深度用于存储数值，其所需的最大深度在编译期就定义好了，保存在方法的Code属性中，为maxstack的值。

栈中的任何一个元素都是可以任意的Java数据类型

- 32bit的类型占用一个栈单位深度
- 64bit的类型占用两个栈单位深度

操作数栈**并非采用访问索引的方式来进行数据访问**，而是只能通过标准的入栈和出栈操作来完成一次数据访问。

**如果被调用的方法带有返回值的话，其返回值将会被压入当前栈帧的操作数栈中**，并更新PC寄存器中下一条需要执行的字节码指令。

操作数栈中元素的数据类型必须与字节码指令的序列严格匹配，这由编译器在编译器期间进行验证，同时在类加载过程中的类检验阶段的数据流分析阶段要再次验证。

另外，我们说Java虚拟机的**解释引擎是基于栈的执行引擎**，其中的栈指的就是操作数栈。

#### (1) 代码追踪

以下面代码为例子：

```java
public void testAddOperation() {
    byte i = 15;
    int j = 8;
    int k = i + j;
}
```

使用javap 命令反编译class文件： `javap -v 类名.class`

```shell
 0 bipush 15
 2 istore_1
 3 bipush 8
 5 istore_2
 6 iload_1
 7 iload_2
 8 iadd
 9 istore_3
10 return
```

从上面的代码我们可以知道，我们都是通过`bipush`对操作数 15 和  8进行入栈操作，同时使用的是 iadd方法进行相加操作，`i `-> 代表的是`int`类型的加法操作。

> Tips：byte、short、char、boolean 内部都是使用int型来进行保存的。

执行流程如下所示：

首先执行第一条语句 `0 bipush 15`，PC寄存器指向的是0，也就是指令地址为0，然后使用bipush让操作数15入栈。

```shell
# ==> 0 bipush 15
PC register       |   0    |
———————————————————————————
Local Variables   | 1 |    |
                  | 2 |    |
                  | 3 |    |
———————————————————————————
Expression Stack  |        |
		          |   15   | < -- 栈顶
———————————————————————————	    

# ==> 2 istore_1
PC register       |   2    |
———————————————————————————
Local Variables   | 1 | 15 |
                  | 2 |    |
                  | 3 |    |
———————————————————————————
Expression Stack  |        |
		          |        | < -- 栈顶
———————————————————————————	                                  
```

执行完后，让PC + 1，指向下一行代码，下一行代码就是将操作数栈的元素存储到局部变量表**1的位置**，我们可以看到局部变量表的已经增加了一个元素。

> **为什么局部变量表不是从0开始的呢**？
>
> 局部变量表也是从0开始的，但是因为0号位置存储的是this指针，这里省略书写了。

然后PC+1，指向的是下一行。让操作数8也入栈，同时执行store操作，存入局部变量表中。

```shell
# ==> 3 bipush 8
PC register       |   3    |
———————————————————————————
Local Variables   | 1 | 15 |
                  | 2 |    |
                  | 3 |    |
———————————————————————————
Expression Stack  |        |
		          |   8    | < -- 栈顶
———————————————————————————	    

# ==> 5 istore_2
PC register       |   5    |
———————————————————————————
Local Variables   | 1 | 15 |
                  | 2 | 8  |
                  | 3 |    |
———————————————————————————
Expression Stack  |        |
		          |        | < -- 栈顶
———————————————————————————	                                  
```

然后从局部变量表中，依次将数据放在操作数栈中。

```shell
# ==> 6 iload_1
PC register       |   6    |
———————————————————————————
Local Variables   | 1 | 15 |
                  | 2 | 8  |
                  | 3 |    |
———————————————————————————
Expression Stack  |        |
		          |   15   | < -- 栈顶
———————————————————————————	    

# ==>  7 iload_2
PC register       |   7    |
———————————————————————————
Local Variables   | 1 | 15 |
                  | 2 | 8  |
                  | 3 |    |
———————————————————————————
Expression Stack  |   8    |
		          |   15   | < -- 栈顶
———————————————————————————	                                  
```

然后将操作数栈中的两个元素执行相加操作，并存储在局部变量表3的位置。

```shell
# ==>  8 iadd
PC register       |   8    |
———————————————————————————
Local Variables   | 1 | 15 |
                  | 2 | 8  |
                  | 3 |    |
———————————————————————————
Expression Stack  |        |
		          |   23   | < -- 栈顶
———————————————————————————	    

# ==>  9 istore_3
PC register       |   9    |
———————————————————————————
Local Variables   | 1 | 15 |
                  | 2 | 8  |
                  | 3 | 23 |
———————————————————————————
Expression Stack  |        |
		          |        | < -- 栈顶
———————————————————————————	       
```

最后PC寄存器的位置指向10，也就是return方法，则直接退出方法。

#### (2) 栈顶缓存技术

栈顶缓存技术：Top Of Stack Cashing

基于栈式架构的虚拟机所使用的零地址指令更加紧凑，但完成一项操作的时候必然需要使用更多的入栈和出栈指令，这同时也就意味着将需要更多的指令分派（instruction dispatch）次数和内存读/写次数。

由于操作数是存储在内存中的，因此频繁地执行内存读/写操作必然会影响执行速度。为了解决这个问题，HotSpot JVM的设计者们提出了栈顶缓存（Tos，Top-of-Stack Cashing）技术，**将栈顶元素全部缓存在物理CPU的寄存器中，以此降低对内存的读/写次数，提升执行引擎的执行效率**。

### 3.3.3 动态链接与方法返回地址

**动态链接**

每一个栈帧内部都包含一个指向**运行时常量池中该栈帧所属方法的引用**。包含这个引用的目的就是为了支持当前方法的代码能够实现**动态链接（Dynamic Linking）**。比如invokedynamic指令。

在Java源文件被编译到字节码文件中时，所有的变量和方法引用都作为符号引用（Symbolic Reference）保存在class文件的常量池里。比如：描述一个方法调用了另外的其他方法时，就是通过常量池中指向方法的符号引用来表示的，那么**动态链接的作用就是为了将这些符号引用转换为调用方法的直接引用**。

 <div align="center"> <img src=".\images\2020100720535831.png" width="700px"></div>

> 为什么需要运行时常量池？因为在不同的方法，都可能调用常量或者方法，所以只需要存储一份即可，节省了空间。
>
> 常量池的作用：就是为了提供一些符号和常量，便于指令的识别。

**方法返回地址**

当一个方法开始执行后，只有两种方式可以退出这个方法：

- 正常执行完成

- 出现未处理的异常

执行引擎遇到任意一个方法返回的字节码指令（return），会有返回值传递给上层的方法调用者，简称“**正常调用完成**”（Normal Method Invocation Completion）：

- 一个方法在正常调用完成之后，究竟需要使用哪一个返回指令，还需要根据方法返回值的实际数据类型而定。
- 在字节码指令中，返回指令包含ireturn（当返回值是boolean，byte，char，short和int类型时使用），lreturn（Long类型），freturn（Float类型），dreturn（Double类型），areturn。另外还有一个return指令声明为void的方法，实例初始化方法，类和接口的初始化方法使用。

在方法执行过程中遇到异常（Exception），并且这个异常没有在方法内进行处理，也就是只要在本方法的异常表中没有搜索到匹配的异常处理器，就会导致方法退出，这种退出方法的方式称为“**异常调用完成**“（Abrupt MethodInvocation Completion）。

无论通过哪种方式退出，在方法退出后都返回到该方法被调用的位置。方法正常退出时，**调用者的PC寄存器的值作为返回地址，即调用该方法的指令的下一条指令的地址**。而通过异常退出的，返回地址是要通过异常表来确定，栈帧中一般不会保存这部分信息。

本质上，方法的退出就是当前栈帧出栈的过程。此时，需要恢复上层方法的局部变量表、操作数栈、将返回值压入调用者栈帧的操作数栈、设置PC寄存器值等，让调用者方法继续执行下去。

正常完成出口和异常完成出口的区别在于：**通过异常完成出口退出的不会给他的上层调用者产生任何的返回值**。

# 4. 本地方法栈

Java虚拟机栈于管理Java方法的调用，而**本地方法栈（Native Method Stack）用于管理本地方法（Native Method）的调用**。与虚拟机栈一样，**本地方法栈会在栈深度溢出或者栈扩展失败时分别抛出StackOverflowError和OutOfMemoryError异常**。

当某个线程调用一个本地方法时，它就进入了一个全新的并且不再受虚拟机限制的世界。它和虚拟机拥有同样的权限。

- 本地方法可以通过本地方法接口来**访问虚拟机内部的运行时数据区**。
- 它甚至可以直接使用本地处理器中的寄存器
- 直接从本地内存的堆中分配任意数量的内存。

**并不是所有的JVM都支持本地方法**。因为Java虚拟机规范并没有明确要求本地方法栈的使用语言、具体实现方式、数据结构等。如果JVM产品不打算支持native方法，也可以无需实现本地方法栈。

>  在Hotspot JVM中，直接将本地方法栈和虚拟机栈合二为一。

什么是本地方法呢？

* **Native Method（本地方法）就是Java调用非Java代码的接囗**。该方法的实现由**非Java语言实现**，比如C。这个特征并非Java所特有，很多其它的编程语言都有这一机制，比如在C++中，你可以用extern "c" 告知c++编译器去调用一个C的函数。

* "A native method is a Java method whose implementation is provided by non-java code."（本地方法是一个非Java的方法，它的具体实现是非Java代码的实现）

* 例如`java.lang.Object`中的`public final native Class<?> getClass()`方法；又如`java.lang.Thread`中的`private native void start0()`方法... ...

> Tips：标识符native可以与其它java标识符连用，abstract除外。

# 5. 堆

## 5.1 堆内存划分

一个进程只有一个JVM，一个JVM实例只存在一个堆内存。但是进程可包含多个线程，他们是共享同一堆空间的。

Java堆区（Heap）在JVM启动的时候即被创建时就确定了空间大小，是JVM管理的最大一块内存空间。《Java虚拟机规范》中规定，堆可以处于**物理上不连续**的内存空间中，但在**逻辑上被视为连续**的。所有的对象实例以及数组都应当在运行时分配在堆上。更准确说法是——“几乎”所有的对象实例都在堆区分配内存。

- 因为还有一些对象是在栈上分配的。数组和对象可能永远不会存储在栈上，因为栈帧中保存引用，这个引用指向对象或者数组在堆中的位置。

Java 7及之前堆内存逻辑上分为三部分：新生区+养老区+**永久区**

| 新生区                                    | 养老区                  | 永久区          |
| ----------------------------------------- | ----------------------- | --------------- |
| Young Generation Space                    | Tenure generation space | Permanent Space |
| Young/New（又被划分为Eden区和Survivor区） | Old/Tenure              | Perm            |

Java 8及之后堆内存逻辑上分为三部分：新生区+养老区+**元空间**

| 新生区                                    | 养老区                  | 元空间     |
| ----------------------------------------- | ----------------------- | ---------- |
| Young Generation Space                    | Tenure generation space | Meta Space |
| Young/New（又被划分为Eden区和Survivor区） | Old/Tenure              | Meta       |

> 新生区=新生代=年轻代；养老区=老年区=老年代；永久区=永久代。

### 5.1.1 设置堆内存大小

Java堆区用于存储Java对象实例，那么堆的大小在JVM启动时就已经设定好了，大家可以通过选项"-Xmx"和"-Xms"来进行设置。例如：

> -Xms10m：最小堆内存   -Xmx10m：最大堆内存

- “**-Xms**"用于表示堆区的起始内存，等价于`-XX:InitialHeapSize`
- “-**Xmx**"则用于表示堆区的最大内存，等价于`-XX:MaxHeapSize`

一旦堆区中的内存大小超过“-Xmx"所指定的最大内存时，将会抛出`OutofMemoryError`异常（俗称OOM异常）。

通常会将-Xms和-Xmx两个参数配置相同的值，其目的是**为了能够在Java垃圾回收机制清理完堆区后不需要重新分隔计算堆区的大小，从而提高性能**。

默认情况：

- 初始内存大小：`物理电脑内存大小 / 64`

- 最大内存大小：`物理电脑内存大小 / 4`

```java
/**
 * -Xms 用来设置堆空间（年轻代+老年代）的初始内存大小
 *  -X：是jvm运行参数
 *  ms：memory start
 * -Xmx：用来设置堆空间（年轻代+老年代）的最大内存大小
 */
public class HeapSpaceInitial {
    public static void main(String[] args) {
        // 返回Java虚拟机中的堆内存总量
        long initialMemory = Runtime.getRuntime().totalMemory() / 1024 / 1024;
        // 返回Java虚拟机试图使用的最大堆内存
        long maxMemory = Runtime.getRuntime().maxMemory() / 1024 / 1024;
        System.out.println("-Xms:" + initialMemory + "M");
        System.out.println("-Xmx:" + maxMemory + "M");
    }
}
```

输出结果：

```shell
-Xms:243M
-Xmx:3591M
系统内存大小：15.1875G
系统内存大小：14.02734375G
```

**查看堆内存的内存分配**

* 方法一：CMD敲入命令`jps`——>`jstat -gc 进程id`

 <div align="center"> <img src=".\images\20201008211326122.png" width="900px"></div>

* 方法二：配置VM option时加上`-XX:+PrintGCDetails`

 <div align="center"> <img src=".\images\20201008211723111.png" width="900px"></div>

### 5.1.2 新生代与老年代

从生命周期角度可将存储在JVM中的Java对象可以被划分为两类：

- 一类是生命周期较短的瞬时对象，这类对象的创建和消亡都非常迅速（生命周期短的，及时回收即可）。
- 另外一类对象的生命周期却非常长，在某些极端的情况下还能够与JVM的生命周期保持一致。

根据存储对象的不同，Java堆区便划分为年轻代（YoungGen）和老年代（OldGen）。其中年轻代又可以划分为Eden空间、Survivor0空间和Survivor1空间（有时也叫做from区、to区）。

 <div align="center"> <img src=".\images\新生代与老年代.png" width="800px"></div>

配置新生代与老年代堆结构的占比：

- 默认`-XX:NewRatio=2`，表示新生代占占整个堆的1/3，老年代占2/3。

- 可以修改`-XX:NewRatio=4`，表示新生代占整个堆的1/5，老年代占4/5。

> Tips：生命周期长的对象偏多，就可以通过调整 老年代的大小，来进行调优。

在新生代中，Eden空间和另外两个survivor空间所占的比例默认是8：1：1。

`-XX:-UseAdaptiveSizePolicy`：关闭自适应的内存分配策略。可以通过选项`-XX:SurvivorRatio`调整这个空间比例。（实际比例不是8：1：1，如要确定是8：1：1需要指定`-XX:SurvivorRatio=8`）

几乎所有的Java对象都是在Eden区被new出来的。绝大部分的Java对象的销毁都在新生代进行了。（有些大的对象在Eden区无法存储时候，将直接进入老年代）

可以使用选项"-Xmn"设置新生代最大内存大小。

## 5.2 堆对象分配过程

为新对象分配内存是一件非常严谨和复杂的任务，JVM的设计者们不仅需要考虑内存如何分配、在哪里分配等问题，并且由于内存分配算法与内存回收算法密切相关，所以还需要考虑GC执行完内存回收后是否会在内存空间中产生内存碎片。

* new的对象先放Eden区。
* 当Eden区的空间填满时，程序还需创建对象，JVM的垃圾回收器将对Eden区进行垃圾回收（**MinorGC**，又称Young GC），将Eden区中的不再被其他对象所引用的对象进行销毁，再加载新的对象放到Eden区。
* 然后将Eden区中的幸存的对象移动到From区（Survivor From区）。

- 如果再次触发垃圾回收，此时Eden区和From区幸存下来的对象就会放到To区（Survivor To区）。
  - 此过程后From区对象都放到To区，故From区变To区，原To区变From区。

- 如果再次经历垃圾回收，此时Eden区对象会重新放回From区，接着再去To区。
- 啥时候能去养老区呢？当Survivor中的对象的年龄达到15的时候，将会触发一次 Promotion晋升的操作，对象晋升至养老区。可以设置次数：`-Xx:MaxTenuringThreshold= N`，**默认是15次**。
- 当养老区内存不足时，再次触发垃圾回收（**Major GC**），进行养老区的内存清理。
- 若养老区执行了Major GC之后，发现依然无法进行对象的保存，就会产生OOM异常。

特别注意，在Eden区满了的时候，才会触发MinorGC；而幸存者区满了后，不会触发MinorGC操作。如果Survivor区满了后，将会触发一些特殊的规则，也就是可能直接晋升老年代。

**对象分配的特殊情况**

 <div align="center"> <img src=".\images\堆对象分配过程.png" width="600px"></div>

**代码演示对象分配过程**

```java
public class HeapInstanceTest {
    byte[] buffer = new byte[new Random().nextInt(1024 * 200)];

    public static void main(String[] args) throws InterruptedException {
        ArrayList<HeapInstanceTest> list = new ArrayList<>();
        while (true) {
            list.add(new HeapInstanceTest());
            Thread.sleep(10);
        }
    }
}
```

然后设置JVM参数

```bash
-Xms600m -Xmx600m
```

执行上面代码，通过VisualGC进行动态化查看。最终，老年代和新生代都满了，出现OOM错误：

```shell
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at com.kai.jvm.HeapInstanceTest.<init>(HeapInstanceTest.java:12)
	at com.kai.jvm.HeapInstanceTest.main(HeapInstanceTest.java:17)
```

**小结**

- 针对幸存者S0，S1区：**复制之后有交换，谁空谁是To区**。
- 关于垃圾回收：**频繁在新生区收集，很少在老年代收集，几乎不再永久代和元空间进行收集**。
- 新生代采用复制算法的目的：是为了减少内碎片。

## 5.3 Minor GC、Major GC、Full GC

- Minor GC：新生代的GC
- Major GC：老年代的GC
- Full GC：整堆收集，收集整个Java堆和方法区的垃圾收集

>Major GC 和 Full GC出现STW的时间，是Minor GC的10倍以上

JVM在进行GC时，并非每次都对上面三个内存区域（新生代，老生代；方法区）一起回收的，大部分时候回收的都是指新生代。针对Hotspot VM的实现，它里面的GC按照回收区域又分为两大种类型：一种是**部分收集（Partial GC）**，一种是**整堆收集（Full GC）**。

**部分收集**：不是完整收集整个Java堆的垃圾收集。其中又分为：

- 新生代收集（Minor GC/Young GC）：只是新生代的垃圾收集
- 老年代收集（Major GC/Old GC）：只是老年代的圾收集。
  - 目前，只有CMS GC会有单独收集老年代的行为。
  - 注意，**很多时候Major GC会和Full GC混淆使用，需要具体分辨是老年代回收还是整堆回收**。
- 混合收集（Mixed GC）：收集整个新生代以及部分老年代的垃圾收集。
  - 目前，只有G1 GC会有这种行为

**整堆收集**：收集整个java堆和方法区的垃圾收集。

**（1）Minor GC**

当年轻代空间不足时，就会触发Minor GC，这里的年轻代指的是Eden满，Survivor满不会引发GC（每次Minor GC会清理年轻代的内存）。

因为Java对象大多都具备 **朝生夕灭** 的特性，所以Minor GC非常频繁，一般回收速度也比较快。这一定义既清晰又易于理解。

Minor GC会引发**STW**，暂停其它用户的线程，等垃圾回收结束，用户线程才恢复运行

> STW：stop the word

**（2）Major GC**

指发生在老年代的GC，对象从老年代消失时，我们说 “Major GC” 或 “Full GC” 发生了。

出现了MajorGC，经常会伴随至少一次的Minor GC（但非绝对的，在Parallel Scavenge收集器的收集策略里就有直接进行Major GC的策略选择过程）

- 也就是在老年代空间不足时，会先尝试触发Minor GC。如果之后空间还不足，则触发Major GC。

Major GC的速度一般会比Minor GC慢10倍以上，STW的时间更长，如果Major GC后，内存还不足，就报OOM了。

**（3）Full GC**

触发Full GC执行的情况有如下五种：

- 调用`System.gc()`时，系统建议执行Full GC，但是不必然执行。
- 老年代空间不足
- 方法区空间不足
- 通过Minor GC后进入老年代的平均大小大于老年代的可用内存
- 由Eden区、survivor space0（From Space）区向survivor space1（To Space）区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小。

说明：**Full GC 是开发或调优中尽量要避免的**。这样暂时时间会短一些。

**GC 举例**

不断的创建字符串是存放在堆区元空间中：

```java
public class GCTest {
    public static void main(String[] args) {
        int i = 0;
        try {
            List<String> list = new ArrayList<>();
            String a = "Hello World!";
            while(true) {
                list.add(a);
                a = a + a;
                i++;
            }
        }catch (Exception e) {
            e.getStackTrace();
        }
    }
}
```

设置JVM启动参数：

```bash
-Xms10m -Xmx10m -XX:+PrintGCDetails
```

打印出日志：

```java
[GC (Allocation Failure) [PSYoungGen: 1996K->480K(2560K)] 1996K->872K(9728K), 0.0010677 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 2460K->472K(2560K)] 2852K->2304K(9728K), 0.0007179 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 2061K->440K(2560K)] 3893K->3040K(9728K), 0.0007400 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 1322K->472K(2560K)] 6994K->6152K(9728K), 0.0014277 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Ergonomics) [PSYoungGen: 472K->0K(2560K)] [ParOldGen: 5680K->3698K(7168K)] 6152K->3698K(9728K), [Metaspace: 3209K->3209K(1056768K)], 0.0039549 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Ergonomics) [PSYoungGen: 1609K->0K(2560K)] [ParOldGen: 6770K->6750K(7168K)] 8380K->6750K(9728K), [Metaspace: 3262K->3262K(1056768K)], 0.0048638 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[GC (Allocation Failure) [PSYoungGen: 0K->0K(2560K)] 6750K->6750K(9728K), 0.0003074 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 0K->0K(2560K)] [ParOldGen: 6750K->6732K(7168K)] 6750K->6732K(9728K), [Metaspace: 3262K->3262K(1056768K)], 0.0050359 secs] [Times: user=0.11 sys=0.00, real=0.01 secs] 
Heap
 PSYoungGen      total 2560K, used 114K [0x00000000ffd00000, 0x0000000100000000, 0x0000000100000000)
  eden space 2048K, 5% used [0x00000000ffd00000,0x00000000ffd1cac8,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
  to   space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
 ParOldGen       total 7168K, used 6732K [0x00000000ff600000, 0x00000000ffd00000, 0x00000000ffd00000)
  object space 7168K, 93% used [0x00000000ff600000,0x00000000ffc93090,0x00000000ffd00000)
 Metaspace       used 3321K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 362K, capacity 388K, committed 512K, reserved 1048576K
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOfRange(Arrays.java:3664)
	at java.lang.String.<init>(String.java:207)
	at java.lang.StringBuilder.toString(StringBuilder.java:407)
	at com.kai.jvm.GCTest.main(GCTest.java:19)
```

> **触发OOM的时候，一定是进行了Full GC，因为只有在老年代空间不足时候，才会爆出OOM异常**。

## 5.4 堆空间分代

 为什么要把Java堆分代？不分代就不能正常工作了吗？经研究，不同对象的生命周期不同。`70%-99%`的对象是临时对象。

>新生代：有Eden、两块大小相同的survivor（又称为from/to，s0/s1）构成，**to总为空**。
>老年代：存放新生代中经历多次GC仍然存活的对象。

分代的唯一理由就是优化GC性能。分代会把新创建的对象放到某一地方，当GC的时候先把这块存储“朝生夕死”对象的区域进行回收，这样就会腾出很大的空间出来。

### 5.4.1 内存分配策略

如果对象在Eden出生并经过第一次Minor GC后仍然存活，并且能被survivor容纳的话，将被移动到survivor空间中，并将对象年龄设为1。对象在survivor区中每熬过一次MinorGC，年龄就增加1岁，当它的年龄增加到一定程度（默认为15岁，其实每个JVM、每个GC都有所不同）时，就会被晋升到老年代（对象晋升老年代的年龄阀值，可以通过选项`-XX:MaxTenuringThreshold`来设置）。

针对不同年龄段的对象分配原则如下所示：

- **优先分配到Eden**
  - 开发中比较长的字符串或者数组，会直接存在老年代，但是因为新创建的对象 都是 朝生夕死的，所以这个大对象可能也很快被回收，但是因为老年代触发Major GC的次数比 Minor GC要更少，因此可能回收起来就会比较慢
- **大对象直接分配到老年代**
  - 尽量避免程序中出现过多的大对象
- **长期存活的对象分配到老年代**
- **动态对象年龄判断**
  - 如果survivor区中相同年龄的所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象可以直接进入老年代，无须等到MaxTenuringThreshold 中要求的年龄。

### 5.4.2 对象分配内存：TLAB

**问题：堆空间都是共享的么？**

不一定，因为还有TLAB这个概念，在堆中划分出一块区域，为每个线程所独占。

**为什么有TLAB？**

`TLAB：Thread Local Allocation Buffer`，**线程本地分配缓存区**，也就是为每个线程在Eden区单独分配了一个缓冲区。

堆区是线程共享区域，任何线程都可以访问到堆区中的共享数据。由于对象实例的创建在JVM中非常频繁，因此在并发环境下从堆区中划分内存空间是线程不安全的。为避免多个线程操作同一地址，需要使用加锁等机制，进而影响分配速度。简而言之，**TLAB可以避免多线程内存分配时的线程安全问题，提升内存分配的吞吐量**。

**什么是TLAB？**

从内存模型而不是垃圾收集的角度，对Eden区域继续进行划分，JVM为**每个线程分配了一个私有缓存区域**，它包含在Eden空间内。

多线程同时分配内存时，使用TLAB可以避免一系列的非线程安全问题，同时还能够提升内存分配的吞吐量，因此我们可以将这种内存分配方式称为快速分配策略。所有OpenJDK衍生出来的JVM都提供了TLAB的设计。

尽管不是所有的对象实例都能够在TLAB中成功分配内存，但**JVM确实是将TLAB作为内存分配的首选**。

在程序中，开发人员可以通过选项`-XX:UseTLAB`设置是否开启TLAB空间。默认情况下，TLAB空间的内存非常小，仅占有**整个Eden空间的1%**，当然我们可以通过选项`-XX:TLABWasteTargetPercent`设置TLAB空间所占用Eden空间的百分比大小。

一旦对象在TLAB空间分配内存失败时，JVM就会尝试着通过**使用加锁机制**确保数据操作的原子性，从而直接在Eden空间中分配内存。

**TLAB分配过程**

对象首先是通过TLAB开辟空间，如果不能放入，那么需要通过Eden来进行分配。

 <div align="center"> <img src=".\images\20201008102018209.png" width="700px"></div>

### 5.4.3 堆空间的参数设置

- `-XX:+PrintFlagsInitial`：查看所有的参数的默认初始值
- `-XX:+PrintFlagsFinal`：查看所有的参数的最终值（可能会存在修改，不再是初始值）
- `-Xms`：初始堆空间内存（默认为物理内存的1/64）
- `-Xmx`：最大堆空间内存（默认为物理内存的1/4）
- `-Xmn`：设置新生代的大小。（初始值及最大值）
- `-XX:NewRatio`：配置新生代与老年代在堆结构的占比

- `-XX:SurvivorRatio`：设置新生代中Eden和S0/S1空间的比例
- `-XX:MaxTenuringThreshold`：设置新生代垃圾的最大年龄
- -`XX:+PrintGCDetails`：输出详细的GC处理日志
  - 打印GC简要信息：①`-XX:+PrintGC ` ② `- verbose:gc`
- `-XX:HandlePromotionFalilure`：是否设置空间分配担保

在发生Minor GC之前，虚拟机会**检查老年代最大可用的连续空间是否大于新生代所有对象的总空间**。

- 如果大于，则此次Minor GC是安全的。
- 如果小于，则虚拟机会查看`-XX:HandlePromotionFailure`设置值是否允担保失败。
  - 如果HandlePromotionFailure=true，那么会**继续检查老年代最大可用连续空间是否大于历次晋升到老年代的对象的平均大小**。
  - 如果大于，则尝试进行一次Minor GC，但这次Minor GC依然是有风险的；
  - 如果小于，则改为进行一次Full GC。
  - 如果HandlePromotionFailure=false，则改为进行一次Full GC。

在JDK6 Update24之后，HandlePromotionFailure参数不会再影响到虚拟机的空间分配担保策略，观察openJDK中的源码变化，虽然源码中还定义了HandlePromotionFailure参数，但是在代码中已经不会再使用它。JDK6 Update 24之后的规则变为只要老年代的连续空间大于新生代对象总大小或者历次晋升的平均大小就会进行Minor GC，否则将进行FullGC。

### 5.4.4 逃逸分析

#### (1) 概述

在《深入理解Java虚拟机》中关于Java堆内存有这样一段描述：

随着JIT编译期的发展与**逃逸分析技术**逐渐成熟，**栈上分配、标量替换优化技术**将会导致一些微妙的变化，所有的对象都分配到堆上也渐渐变得不那么“绝对”了。

在Java虚拟机中，对象是在Java堆中分配内存的，这是一个普遍的常识。但是，有一种特殊情况，那就是如果**经过逃逸分析（Escape Analysis）后发现，一个对象并没有逃逸出方法的话，那么就可能被优化成栈上分配**。这样就无需在堆上分配内存，也无须进行垃圾回收了。这也是最常见的堆外存储技术。

此外，前面提到的基于openJDk深度定制的TaoBaovm，其中创新的GCIH（GC invisible heap）技术实现off-heap，将生命周期较长的Java对象从heap中移至heap外，并且GC不能管理GCIH内部的Java对象，以此达到降低GC的回收频率和提升GC的回收效率的目的。

如何将堆上的对象分配到栈，需要使用逃逸分析手段。

**逃逸分析可以理解为一种减少同步负载与堆内存分配压力的数据流分析算法，主要分析对象使用的作用域范围**。。通过逃逸分析，Java Hotspot编译器能够分析出一个新的对象的引用的使用范围从而决定是否要将这个对象分配到堆上。逃逸分析的基本行为就是分析对象动态作用域：

- 当一个对象在方法中被定义后，对象只在方法内部使用，则认为没有发生逃逸。
- 当一个对象在方法中被定义后，它被外部方法所引用，则认为发生逃逸。例如作为调用参数传递到其他地方中。

**逃逸分析举例**

没有发生逃逸的对象，则可以分配到栈上，随着方法执行的结束，栈空间就被移除。

```java
public void my_method() {
    V v = new V();
    // use v
    // ....
    v = null;
}
```

针对下面的代码

```java
public static StringBuffer createStringBuffer(String s1, String s2) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    return sb;
}
```

如果想要StringBuffer sb不发生逃逸，可以这样写

```java
public static String createStringBuffer(String s1, String s2) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    return sb.toString();
}
```

完整的逃逸分析代码举例

```java
public class EscapeAnalysis {

    public EscapeAnalysis obj;

    /**
     * 方法返回EscapeAnalysis对象，发生逃逸
     * @return
     */
    public EscapeAnalysis getInstance() {
        return obj == null ? new EscapeAnalysis():obj;
    }

    /**
     * 为成员属性赋值，发生逃逸
     */
    public void setObj() {
        this.obj = new EscapeAnalysis();
    }

    /**
     * 对象的作用于仅在当前方法中有效，没有发生逃逸
     */
    public void useEscapeAnalysis() {
        EscapeAnalysis e = new EscapeAnalysis();
    }

    /**
     * 引用成员变量的值，发生逃逸
     */
    public void useEscapeAnalysis2() {
        EscapeAnalysis e = getInstance();
        // getInstance().XXX  发生逃逸
    }
}
```

> **开发中能使用局部变量的，就不要使用在方法外定义**。

在JDK 1.7 版本之后，HotSpot中默认就已经开启了逃逸分析。如果使用的是较早的版本，则可以通过：

- 选项`-XX:+DoEscapeAnalysis`显式开启逃逸分析。
- 通过选项`-xx:+PrintEscapeAnalysis`查看逃逸分析的筛选结果。

使用逃逸分析，编译器可以对代码做如下优化：

- **栈上分配**：将堆分配转化为栈分配。如果一个对象在子程序中被分配，要使指向该对象的指针永远不会发生逃逸，对象可能是栈上分配的候选，而不是堆上分配
- **同步省略**：如果一个对象被发现只有一个线程被访问到，那么对于这个对象的操作可以不考虑同步。
- **分离对象或标量替换**：有的对象可能不需要作为一个连续的内存结构存在也可以被访问到，那么对象的部分（或全部）可以不存储在内存，而是存储在CPU寄存器中。

#### (2) 栈上分配

JIT编译器在编译期间根据逃逸分析的结果，发现如果一个对象并没有逃逸出方法的话，就可能被优化成栈上分配。分配完成后，继续在调用栈内执行，最后线程结束，栈空间被回收，局部变量对象也被回收。这样就无须进行垃圾回收了。

常见的栈上分配的场景：**给成员变量赋值、方法返回值、实例引用传递**。

**举例**

我们通过举例来说明 开启逃逸分析 和 未开启逃逸分析时候的情况

```java
public class StackAllocation {
    public static void main(String[] args) throws InterruptedException {
        long start = System.currentTimeMillis();
        for (int i = 0; i < 100000000; i++) {
            alloc();
        }
        long end = System.currentTimeMillis();
        System.out.println("花费的时间为：" + (end - start) + " ms");

        // 为了方便查看堆内存中对象个数，线程sleep
        Thread.sleep(10000000);
    }

    private static void alloc() {
        User user = new User();
    }
}

class User {
    private String name;
    private String age;
    private String gender;
    private String phone;
}
```

设置JVM参数，未开启逃逸分析：

```shell
-Xmx1G -Xms1G -XX:-DoEscapeAnalysis -XX:+PrintGCDetails
```

运行结果，同时还触发了GC操作：

```
花费的时间为：366 ms
```

然后查看内存的情况，发现有大量的User存储在堆中。

 <div align="center"> <img src=".\images\20201009004554162.png" width="1000px"></div>

我们再开启逃逸分析：

```shell
-Xmx1G -Xms1G -XX:+DoEscapeAnalysis -XX:+PrintGCDetails
```

然后查看运行时间，我们能够发现花费的时间快速减少，同时不会发生GC操作。

```
花费的时间为：5 ms
```

然后再看内存情况，我们发现只有很少的User对象，说明User发生了逃逸，因为他们存储在栈中，随着栈的销毁而消失。

 <div align="center"> <img src=".\images\20201009004323454.png" width="1000px"></div>

#### (3) 同步省略

线程同步的代价是相当高的，同步的后果是降低并发性和性能。

在动态编译同步块的时候，JIT编译器可以借助逃逸分析来判断同步块所使用的锁对象是否只能够被一个线程访问而没有被发布到其他线程。如果没有，那么JIT编译器在编译这个同步块的时候就会取消对这部分代码的同步。这样就能大大提高并发性和性能。这个取消同步的过程就叫**同步省略**，也叫**锁消除**。

例如下面的代码：

```java
public void f() {
    Object hollis = new Object();
    synchronized(hollis) {
        System.out.println(hollis);
    }
}
```

代码中对hollis这个对象加锁，但是hollis对象的生命周期只在f()方法中，并不会被其他线程所访问到，所以在JIT编译阶段就会被优化成：

```java
public void f() {
    Object hollis = new Object();
	System.out.println(hollis);
}
```

我们将其转换成字节码：

 <div align="center"> <img src=".\images\20201009005824846.png" width="800px"></div>

#### (4) 分离对象和标量替换

**标量（scalar）**是指一个无法再分解成更小的数据的数据。Java中的原始数据类型就是标量。

相对的，那些还可以分解的数据叫做**聚合量（Aggregate）**，Java中的对象就是聚合量，因为他可以分解成其他聚合量和标量。

在JIT阶段，如果经过逃逸分析，发现一个对象不会被外界访问的话，那么经过JIT优化，就会把这个对象拆解成若干个其中包含的若干个成员变量来代替。这个过程就是**标量替换**。

参数`-XX:+EliminateAllocations`开启标量替换（默认打开），允许对象打散分配在栈上。

```java
public static void main(String args[]) {
    alloc();
}
class Point {
    private int x;
    private int y;
}
private static void alloc() {
    Point point = new Point(1,2);
    System.out.println("point.x" + point.x + ";point.y" + point.y);
}
```

以上代码，经过标量替换后，就会变成

```java
private static void alloc() {
    int x = 1;
    int y = 2;
    System.out.println("point.x = " + x + "; point.y=" + y);
}
```

可以看到，Point这个聚合量经过逃逸分析后，发现他并没有逃逸，就被替换成两个聚合量了。那么标量替换有什么好处呢？就是可以大大减少堆内存的占用。因为一旦不需要创建对象了，那么就不再需要分配堆内存了。
标量替换为栈上分配提供了很好的基础。

**代码优化之标量替换**

```java
public class ScalarReplaceTest {
    public static class User {
        private int age;
        private String name;
    }

    private static void alloc() {
        User user = new User(); //未发生逃逸
        user.age = 20;
        user.name = "张三";
    }

    public static void main(String args[]) {
        long start = System.currentTimeMillis();

        for (int i = 0; i < 100000000; i++) {
            alloc();
        }
        long end = System.currentTimeMillis();
        System.out.println("花费时间：" + (end - start) + "ms");
    }
}
```

上述代码在主函数中进行了1亿次alloc。调用进行对象创建，由于User对象实例需要占据约16字节的空间，因此累计分配空间达到将近1.5GB。如果堆空间小于这个值，就必然会发生GC。使用如下参数运行上述代码：

```bash
-server -Xmx100m -Xms100m -XX:+DoEscapeAnalysis -XX:+PrintGC -XX:+EliminateAllocations
```

这里设置参数如下：

- 参数`-server`：启动Server模式，因为在server模式下，才可以启用逃逸分析。
- 参数`-XX:+DoEscapeAnalysis`：启用逃逸分析
- 参数`-Xmx10m`：指定了堆空间最大为10MB
- 参数`-XX:+PrintGC`：将打印GC日志。
- 参数`-xx：+EliminateAllocations`：开启了标量替换（默认打开），允许将对象打散分配在栈上，比如对象拥有id和name两个字段，那么这两个字段将会被视为两个独立的局部变量进行分配

#### (5) 逃逸分析的不足

关于逃逸分析的论文在1999年就已经发表了，但直到JDK1.6才有实现，而且这项技术到如今也并不是十分成熟的。

其根本原因就是**无法保证逃逸分析的性能消耗一定能高于它的消耗。虽然经过逃逸分析可以做标量替换、栈上分配、和锁消除。但是逃逸分析自身也是需要进行一系列复杂的分析的，这其实也是一个相对耗时的过程**。

一个极端的例子，就是经过逃逸分析之后，发现没有一个对象是不逃逸的。那这个逃逸分析的过程就白白浪费掉了。

虽然这项技术并不十分成熟，但是它也是**即时编译器优化技术中一个十分重要的手段**。注意到有一些观点，认为通过逃逸分析，JVM会在栈上分配那些不会逃逸的对象，这在理论上是可行的，但是取决于JVM设计者的选择。Oracle Hotspot JVM中并未这么做，这一点在逃逸分析相关的文档里已经说明，所以可以明确所有的对象实例都是创建在堆上。

目前很多书籍还是基于JDK7以前的版本，JDK已经发生了很大变化，intern字符串的缓存和静态变量曾经都被分配在永久代上，而永久代已经被元数据区取代。但是，intern字符串缓存和静态变量并不是被转移到元数据区，而是直接在堆上分配，所以这一点同样符合前面的结论：**对象实例都是分配在堆上**。

# 6. 方法区

## 6.1 概述

《Java虚拟机规范》中明确说明：“尽管所有的方法区在逻辑上是属于堆的一部分，但一些简单的实现可能不会选择去进行垃圾收集或者进行压缩。”但对于HotSpot JVM而言，**方法区还有一个别名叫做Non-Heap（非堆**），目的就是要和堆分开。

所以，**方法区看作是一块独立于Java堆的内存空间**。

方法区主要存放的是 **Class**，而堆中主要存放的是 **实例化的对象**。

- 方法区（**Method Area**）与**Java堆**一样，是各个线程共享的内存区域，其在JVM启动的时候被创建，并且它的实际的物理内存空间中和Java堆区一样都可以是不连续的。
- 方法区的大小，跟堆空间一样，可以选择固定大小或者可扩展。方法区的大小决定了系统可以保存多少个类，如果系统定义了太多的类，导致方法区溢出，虚拟机同样会抛出内存溢出错误：`java.lang.OutofMemoryError：PermGen space` 或者`java.lang.OutOfMemoryError:Metaspace`。
- 关闭JVM就会释放这个方法区的内存。

 **设置方法区大小**

* jdk7及以前
  * 通过`-XX:Permsize`来设置永久代初始分配空间。默认值是20.75M。
  * `-XX:MaxPermsize`来设定永久代最大可分配空间。32位机器默认是64M，64位机器模式是82M。
  * 当JVM加载的类信息容量超过了这个值，会报异常`OutofMemoryError:PermGen space`。

* jdk8以后
  * 元数据区大小可以使用参数 `-XX:MetaspaceSize `和 `-XX:MaxMetaspaceSize`指定。
  * 默认值依赖于平台。Windows下，`-XX:MetaspaceSize`是21M，`-XX:MaxMetaspaceSize`的值是`-1`，即没有限制。
  * 与永久代不同，如果不指定大小，默认情况下，虚拟机会耗尽所有的可用系统内存。如果元数据区发生溢出，虚拟机一样会抛出异常`OutOfMemoryError:Metaspace`。
  * `-XX:MetaspaceSize`：设置初始的元空间大小。对于一个64位的服务器端JVM来说，其默认的`-XX:MetaspaceSize`值为21MB。这就是初始的高水位线，一旦触及这个水位线，Full GC将会被触发并卸载没用的类（即这些类对应的类加载器不再存活）然后这个高水位线将会重置。新的高水位线的值取决于GC后释放了多少元空间。如果释放的空间不足，那么在不超过`MaxMetaspaceSize`时，适当提高该值。如果释放空间过多，则适当降低该值。
  * 如果初始化的高水位线设置过低，上述高水位线调整情况会发生很多次。通过垃圾回收器的日志可以观察到FullGC多次调用。为了避免频繁地GC，建议将`-XX:MetaspaceSize`设置为一个相对较高的值。

**如何解决OOM**

- 要解决OOM异常或heap space的异常，一般的手段是首先通过内存映像分析工具（如Eclipse Memory Analyzer）对dump出来的堆转储快照进行分析，重点是确认内存中的对象是否是必要的，也就是要先分清楚到底是出现了**内存泄漏**（Memory Leak）还是**内存溢出**（Memory Overflow）

- 如果是内存泄漏，可进一步通过工具查看泄漏对象到GC Roots的引用链。于是就能找到泄漏对象是通过怎样的路径与GCRoots相关联并导致垃圾收集器无法自动回收它们的。掌握了泄漏对象的类型信息，以及GCRoots引用链的信息，就可以比较准确地定位出泄漏代码的位置。

- 如果不存在内存泄漏，换句话说就是内存中的对象确实都还必须存活着，那就应当检查虚拟机的堆参数（-Xmx与-Xms），与机器物理内存对比看是否还可以调大，从代码上检查是否存在某些对象生命周期过长、持有状态时间过长的情况，尝试减少程序运行期的内存消耗。

## 6.2 方法区的内部结构

《深入理解Java虚拟机》书中对方法区（Method Area）存储内容描述如下：它用于存储已被虚拟机加载的**类型信息**、**常量**、**静态变量**、**即时编译器编译后的代码缓存**等（jdk7）。说简单点，主要存放常量池与方法信息与类信息。

 <div align="center"> <img src=".\images\20201010101335691.png" width="600px"></div>

### 6.2.1 类信息

**类型信息**

对每个加载的类型（类class、接口interface、枚举enum、注解annotation），JVM必须在方法区中存储以下类型信息：

- 这个类型的完整有效名称（全名=包名.类名）
- 这个类型直接父类的完整有效名（对于interface或是`java.lang.object`，都没有父类）
- 这个类型的修饰符（public，abstract，final的某个子集）
- 这个类型直接接口的一个有序列表

**域信息**

JVM必须在方法区中保存类型的所有域的相关信息以及域的声明顺序。

域的相关信息包括：域名称、域类型、域修饰符（public，private，protected，static，final，volatile，transient的某个子集）

**non-final的类变量**

静态变量和类关联在一起，随着类的加载而加载，他们成为类数据在逻辑上的一部分，类变量被类的所有实例共享，即使没有类实例时，也可以访问它。

```java
public class MethodAreaTest {
    public static void main(String[] args) {
        Order order = new Order();
        order.hello();
        System.out.println(order.count);
    }
}

class Order {
    public static int count = 1;
    public static final int number = 2;
    public static void hello() {
        System.out.println("hello!");
    }
}
```

如上代码所示，即使我们把order设置为null，也不会出现空指针异常。

### 6.2.2 方法信息

JVM必须保存所有方法的以下信息，同域信息一样包括声明顺序：

- 方法名称
- 方法的返回类型（或void）
- 方法参数的数量和类型（按顺序）
- 方法的修饰符（public，private，protected，static，final，synchronized，native，abstract的一个子集）
- 方法的字节码（bytecodes）、操作数栈、局部变量表及大小（abstract和native方法除外）
- 异常表（abstract和native方法除外）

>每个异常处理的开始位置、结束位置、代码处理在程序计数器中的偏移地址、被捕获的异常类的常量池索引。

### 6.2.3 运行时常量池

#### (1) 常量池

Java中的常量池，实际上分为两种形态：**静态常量池**和**运行时常量池**。

所谓**静态常量池**，即`*.class`文件中的常量池，class文件中的常量池不仅仅包含字符串(数字)字面量，还包含类、方法的信息，占用class文件绝大部分空间。这种常量池主要用于存放两大类常量：**字面量**(Literal)和**符号引用量**(Symbolic References)，字面量相当于Java语言层面常量的概念，如文本字符串，声明为final的常量值等，符号引用则属于编译原理方面的概念，包括了如下三种类型的常量：

- 类和接口的全限定名
- 字段名称和描述符
- 方法名称和描述符

 <div align="center"> <img src=".\images\20201010211912485.png" width="700px"></div>

**为什么需要常量池**

一个java源文件中的类、接口，编译后产生一个字节码文件。而Java中的字节码需要数据支持，通常这种数据会很大以至于不能直接存到字节码里，换另一种方式，可以存到常量池，这个字节码包含了指向常量池的引用。

比如：

```java
public class SimpleClass {
    public void sayHello() {
        System.out.println("hello");
    }
}
```

虽然上述代码只有194字节，但是里面却使用了String、System、PrintStream及Object等结构。这里的代码量其实很少了，如果代码多的话，引用的结构将会更多，这里就需要用到常量池了。

运行时常量池是方法区的一部分。Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有常量池表（用于存放编译期生成的各种字面量和符号引用）。

既然运行时常量池是方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会抛出 OutOfMemoryError 错误。

例如下面这段代码：

```java
public class MethodAreaTest2 {
    public static void main(String args[]) {
        Object obj = new Object();
    }
}
```

将会被翻译成如下字节码：

```bash
new #2  
dup
invokespecial
```

**小结**

常量池、可以看做是一张表，虚拟机指令根据这张常量表找到要执行的类名、方法名、参数类型、字面量等类型。

#### (2) 运行时常量池

运行时常量池（Runtime Constant Pool）是方法区的一部分。

常量池表（Constant Pool Table）是Class文件的一部分，**用于存放编译期生成的各种字面量与符号引用，这部分内容将在类加载后存放到方法区的运行时常量池中**。

运行时常量池，在加载类和接口到虚拟机后，就会创建对应的运行时常量池。

JVM为每个已加载的类型（类或接口）都维护一个常量池。池中的数据项像数组项一样，是通过索引访问的。

运行时常量池中包含多种不同的常量，包括编译期就已经明确的数值字面量，也包括到运行期解析后才能够获得的方法或者字段引用。此时不再是常量池中的符号地址了，这里换为真实地址。

运行时常量池类似于传统编程语言中的符号表（symboltable），但是它所包含的数据却比符号表要更加丰富一些。当创建类或接口的运行时常量池时，如果构造运行时常量池所需的内存空间超过了方法区所能提供的最大值，则JVM会抛OutOfMemoryError异常。

#### (3) 图解实例

如下代码

```java
public class MethodAreaDemo {
    public static void main(String args[]) {
        int x = 500;
        int y = 100;
        int a = x / y;
        int b = 50;
        System.out.println(a+b);
    }
}
```

字节码执行过程展示，首先现将操作数500放入到操作数栈中：

```shell
   Method Area	      |        |  PC register       |   0      |                   
——————————————————————|        —————————————————————————————————	            
0: sipush        500  |  <==   |  Local Variables   | 0 | args |
3: istore_1			  |		   |                    | 1 |      | 	
4: bipush        100  |        |                    | 2 |      |
6: istore_2			  |		   |                    | 3 |      |
7: iload_1			  |		   |                    | 4 |      |
8: iload_2			  |        —————————————————————————————————	  
9: idiv				  |        |  Expression Stack  |          |
10: istore_3		  |        |                    |   500    | < -- 栈顶
11: bipush        50  |        —————————————————————————————————	 
13: istore        4   |
15: getstatic     #2  |                  
18: iload_3			  |
19: iload         4	  |
21: iadd			  |
22: invokevirtual #3  |               
25: return			  |
```

然后再存储到局部变量表中：

```shell
   Method Area	      |        |  PC register       |   3      |                   
——————————————————————|        —————————————————————————————————	            
0: sipush        500  |        |  Local Variables   | 0 | args |
3: istore_1			  |	 <==   |                    | 1 | 500  | 	
4: bipush        100  |        |                    | 2 |      |
6: istore_2			  |		   |                    | 3 |      |
7: iload_1			  |		   |                    | 4 |      |
8: iload_2			  |        —————————————————————————————————	  
9: idiv				  |        |  Expression Stack  |          |
10: istore_3		  |        |                    |          | < -- 栈顶
11: bipush        50  |        —————————————————————————————————	 
13: istore        4   |
15: getstatic     #2  |                  
18: iload_3			  |
19: iload         4	  |
21: iadd			  |
22: invokevirtual #3  |               
25: return			  |
```

然后重复一次，把100放入局部变量表中，最后再将变量表中的500 和 100 取出，进行操作：

```shell
   Method Area	      |        |  PC register       |    8     |                   
——————————————————————|        —————————————————————————————————	            
0: sipush        500  |        |  Local Variables   | 0 | args |
3: istore_1			  |	       |                    | 1 | 500  | 	
4: bipush        100  |        |                    | 2 | 100  |
6: istore_2			  |		   |                    | 3 |      |
7: iload_1			  |	       |                    | 4 |      |
8: iload_2			  |   <==  —————————————————————————————————	  
9: idiv				  |        |  Expression Stack  |    100   |
10: istore_3		  |        |                    |    500   | < -- 栈顶
11: bipush        50  |        —————————————————————————————————	 
13: istore        4   |
15: getstatic     #2  |                  
18: iload_3			  |
19: iload         4	  |
21: iadd			  |
22: invokevirtual #3  |               
25: return			  |
```

将500 和 100 进行一个除法运算，结果入栈：

```shell
   Method Area	      |        |  PC register       |    9     |                   
——————————————————————|        —————————————————————————————————	            
0: sipush        500  |        |  Local Variables   | 0 | args |
3: istore_1			  |	       |                    | 1 | 500  | 	
4: bipush        100  |        |                    | 2 | 100  |
6: istore_2			  |		   |                    | 3 |      |
7: iload_1			  |	       |                    | 4 |      |
8: iload_2			  |        —————————————————————————————————	  
9: idiv				  |   <==  |  Expression Stack  |    5     |
10: istore_3		  |        |                    |    500   | < -- 栈顶
11: bipush        50  |        —————————————————————————————————	 
13: istore        4   |
15: getstatic     #2  |                  
18: iload_3			  |
19: iload         4	  |
21: iadd			  |
22: invokevirtual #3  |               
25: return			  |
```

最后就是输出流，需要调用运行时常量池的常量：

```shell
   Method Area	      |        |  PC register       |    15    |                   
——————————————————————|        —————————————————————————————————	            
0: sipush        500  |        |  Local Variables   | 0 | args |
3: istore_1			  |	       |                    | 1 | 500  | 	
4: bipush        100  |        |                    | 2 | 100  |
6: istore_2			  |		   |                    | 3 | 5    |
7: iload_1			  |	       |                    | 4 | 50   |
8: iload_2			  |        —————————————————————————————————	  
9: idiv				  |        |  Expression Stack  |          |
10: istore_3		  |        |                    |    #2    | < -- 栈顶
11: bipush        50  |        —————————————————————————————————	 
13: istore        4   |
15: getstatic     #2  |  <==                
18: iload_3			  |
19: iload         4	  |
21: iadd			  |
22: invokevirtual #3  |               
25: return			  |
```

最后调用invokevirtual（虚方法调用），然后返回：

```shell
   Method Area	      |        |  PC register       |    22    |                   
——————————————————————|        —————————————————————————————————	            
0: sipush        500  |        |  Local Variables   | 0 | args |
3: istore_1			  |	       |                    | 1 | 500  | 	
4: bipush        100  |        |                    | 2 | 100  |
6: istore_2			  |		   |                    | 3 | 5    |
7: iload_1			  |	       |                    | 4 | 50   |
8: iload_2			  |        —————————————————————————————————	  
9: idiv				  |        |  Expression Stack  |    55    |
10: istore_3		  |        |                    |    #2    | < -- 栈顶
11: bipush        50  |        —————————————————————————————————	 
13: istore        4   |
15: getstatic     #2  |                  
18: iload_3			  |
19: iload         4	  |
21: iadd			  |
22: invokevirtual #3  |  <==               
25: return			  |
```

调用静态方法，JVM会根据这个方法的描述，创建新栈帧，方法的参数从操作数栈中弹出，压入虚拟机栈然后虚拟机会开始执行虚拟机栈最上面的栈帧。到22行，执行完毕后，再继续执行main方法对象的栈帧。

返回：

```shell
   Method Area	      |        |  PC register       |    25    |                   
——————————————————————|        —————————————————————————————————	            
0: sipush        500  |        |  Local Variables   | 0 | args |
3: istore_1			  |	       |                    | 1 | 500  | 	
4: bipush        100  |        |                    | 2 | 100  |
6: istore_2			  |		   |                    | 3 | 5    |
7: iload_1			  |	       |                    | 4 | 50   |
8: iload_2			  |        —————————————————————————————————	  
9: idiv				  |        |  Expression Stack  |          |
10: istore_3		  |        |                    |          | < -- 栈顶
11: bipush        50  |        —————————————————————————————————	 
13: istore        4   |
15: getstatic     #2  |                  
18: iload_3			  |
19: iload         4	  |
21: iadd			  |
22: invokevirtual #3  |               
25: return			  |  <==
```

程序计数器始终计算的都是当前代码运行的位置，目的是为了方便记录 方法调用后能够正常返回，或者是进行了CPU切换后，也能回来到原来的代码进行执行。

## 6.3 方法区的演进细节

在jdk7及以前，习惯上把方法区，称为永久代；jdk8开始，使用元空间取代了永久代，而元空间存放在堆外内存中。

《Java虚拟机规范》对如何实现方法区，不做统一要求。例如：BEA JRockit / IBM J9 中不存在永久代的概念。而到了JDK8，终于完全废弃了永久代的概念，改用与JRockit、J9一样在本地内存中实现的元空间（Metaspace）来代替。

**元空间的本质和永久代类似，都是对JVM规范中方法区的实现**。不过元空间与永久代最大的区别在于：**元空间不在虚拟机设置的内存中，而是使用本地内存**。

永久代、元空间二者并不只是名字变了，内部结构也调整了。根据《Java虚拟机规范》的规定，如果方法区无法满足新的内存分配需求时，将抛出OOM异常（永久代更容易导致Java程序OOM，即超过`-XX:MaxPermsize`上限）。

| 版本         | 主要变化                                                                                         |
| ------------ | ------------------------------------------------------------------------------------------------ |
| JDK1.6及以前 | 有永久代（Permanent Generation），静态变量存储在永久代上。                                       |
| JDK1.7       | 有永久代，但已经逐步 “去永久代”，字符串常量池，静态变量移除，保存在堆中。                        |
| JDK1.8       | 无永久代，类型信息，字段，方法，常量保存在本地内存的元空间；但字符串常量池、静态变量仍然在堆中。 |

JDK6的时候：

 <div align="center"> <img src=".\images\20201010215128423.png" width="600px"></div>

JDK7的时候：

 <div align="center"> <img src=".\images\20201010215208923.png" width="600px"></div>

JDK8的时候，元空间大小只受物理内存影响：

 <div align="center"> <img src=".\images\20201010215243374.png" width="600px"></div>

**为什么永久代要被元空间替代？**

由于类的元数据分配在本地内存中，元空间的最大可分配空间就是系统可用内存空间，这项改动是很有必要的，原因有：

- **为永久代设置空间大小是很难确定的，对永久代进行调优也是很困难的**。

在某些场景下，如果动态加载类过多，容易产生Perm区的OOM。比如某个实际Web工程中，因为功能点比较多，在运行过程中，要不断动态加载很多类，经常出现致命错误。（`“Exception in thread‘dubbo client x.x connector 'java.lang.OutOfMemoryError:PermGen space”`）。

而元空间和永久代之间最大的区别在于：**元空间并不在虚拟机中，而是使用本地内存**。因此，默认情况下，元空间的大小仅受本地内存限制。

**StringTable为什么要调整位置？**

jdk7中将StringTable放到了堆空间中。因为永久代的回收效率很低，在Full GC的时候才会触发。而Full GC是老年代的空间不足、永久代不足时才会触发。

这就导致StringTable回收效率不高。而我们开发中会有大量的字符串被创建，回收效率低，导致永久代内存不足。放到堆里，能及时回收内存。

**静态变量到底存放在那里？**

首先需要明白的一点是，所有的对象都存储在堆中。例如基本类型的常量是基础类型的话，就储存于方法区常量池；是引用类型的话，就存储在堆中。静态变量也是类似的，**静态引用对应的对象实体始终都存在堆空间，只要是对象实例必然会在Java堆中分配**。

从《Java虚拟机规范》所定义的概念模型来看，所有Class相关的信息都应该存放在方法区之中，但方法区该如何实现，《Java虚拟机规范》并未做出规定，这就成了一件允许不同虚拟机自己灵活把握的事情。JDK7及其以后版本的HotSpot虚拟机选择把静态变量与类型在Java语言一端的映射class对象存放在一起，存储于Java堆之中。

## 6.4 方法区的垃圾回收

有些人认为方法区（如HotSpot虚拟机中的元空间或者永久代）是没有垃圾收集行为的，其实不然。《Java虚拟机规范》对方法区的约束是非常宽松的，提到过可以不要求虚拟机在方法区中实现垃圾收集。事实上也确实有未实现或未能完整实现方法区类型卸载的收集器存在（如JDK11时期的zGC收集器就不支持类卸载）。

一般来说这个区域的回收效果比较难令人满意，尤其是类型的卸载，条件相当苛刻。但是这部分区域的回收**有时又确实是必要的**。以前Sun公司的Bug列表中，曾出现过的若干个严重的Bug就是由于低版本的HotSpot虚拟机对此区域未完全回收而导致内存泄漏。

方法区的垃圾收集主要回收两部分内容：**常量池中废弃的常量和不再使用的类型**。

先来说说方法区内常量池之中主要存放的两大类常量：字面量和符号引用。字面量比较接近Java语言层次的常量概念，如文本字符串、被声明为final的常量值等。而符号引用则属于编译原理方面的概念，包括下面三类常量：

- 类和接口的全限定名
- 字段的名称和描述符
- 方法的名称和描述符

HotSpot虚拟机对常量池的回收策略是很明确的，只要常量池中的常量没有被任何地方引用，就可以被回收。

回收废弃常量与回收Java堆中的对象非常类似。（关于常量的回收比较简单，重点是类的回收）

判定一个常量是否“废弃”还是相对简单，而要判定一个类型是否属于“不再被使用的类”的条件就比较苛刻了。需要同时满足下面三个条件：

- 该类所有的实例都已经被回收，也就是Java堆中不存在该类及其任何派生子类的实例。
  加载该类的类加载器已经被回收，这个条件除非是经过精心设计的可替换类加载器的场景，如osGi、JSP的重加载等，否则通常是很难达成的。
- 该类对应的java.lang.C1ass对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。I Java虚拟机被允许对满足上述三个条件的无用类进行回收，这里说的仅仅是“被允许”，而并不是和对象一样，没有引用了就必然会回收。关于是否要对类型进行回收，HotSpot虚拟机提供了-Xnoclassgc参数进行控制，还可以使用`-verbose:class` 以及 `-XX：+TraceClass-Loading`、`-XX:+TraceClassUnLoading`查看类加载和卸载信息
- 在大量使用反射、动态代理、CGLib等字节码框架，动态生成JSP以及oSGi这类频繁自定义类加载器的场景中，通常都需要Java虚拟机具备类型卸载的能力，以保证不会对方法区造成过大的内存压力。

# 7. 直接内存

**直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，也不是虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用。而且也可能导致 OutOfMemoryError 错误出现。**

JDK1.4 中新加入的 **NIO(New Input/Output) 类**，引入了一种基于**通道（Channel）** 与**缓存区（Buffer）** 的 I/O 方式，它可以直接使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆中的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样就能在一些场景中显著提高性能，因为**避免了在 Java 堆和 Native 堆之间来回复制数据**。

通常，访问直接内存的速度会优于Java堆，读写性能高。

- 因此出于性能考虑，读写频繁的场合可能会考虑使用直接内存。
- Java的NIO库允许Java程序使用直接内存，用于数据缓冲区。

本机直接内存的分配不会受到 Java 堆的限制，不会直接受限于`-xmx`指定的最大堆大小，但是系统内存是有限的，Java堆和直接内存的总和依然受限于操作系统能给出的最大内存。

- 因为不受JVM GC回收管理，分配回收成本较高。

直接内存大小可以通过`-XX:MaxDirectMemorySize`设置，如果不指定，默认与堆的最大值`-xmx`参数值一致。

# 参考资料

* 《深入理解Java虚拟机：JVM高级特性与最佳实践（第3版）》
* [Oracle.The Structure of the Java Virtual Machine](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.5)