#### 01 谈谈你对Java平台的理解？“Java是解释执行”，这句话正确吗？

Java发展至今，已经不止是一门计算机符号语言，更是一个平台、一种文化、一个社区。

* 作为一个平台
  * JVM可以支持Groovy、Koltin、JRuby、Scala等非Java语言（Java虚拟机根本不关心运行在其内部的程序到底是使用何种编程语言编写的，它只关心**“字节码”文件**。）

* 作为一种文化
  * 可以说是“开源”的代名词，如第三方开源软件或框架Tomcat、MyBatis、Spring等；JDK与JVM也有不少开源的实现，如openJDK、Harmony等。
* 作为一个社区
  * Java拥有全世界最多的技术拥护者和开源社区支持，有数不清的论坛和资料。同时应用广泛，从桌面应用软件、嵌入式开发到企业级应用、后台服务器、中间件，都可以看到Java的身影。

**JDK——Java Development ToolKit（Java开发工具包）**是整个JAVA的核心，包括了Java运行环境（Java Runtime Envirnment），一堆Java工具和Java基础的类库。

<img src="https://img-blog.csdnimg.cn/20201005202622207.png" style="zoom:80%;" />

**JRE——Java Runtime Enviromental（Java运行时环境）**，也就是我们说的Java平台，所有的Java程序都要在JRE下才能运行。包括JVM和Java核心类库和支持文件。与JDK相比，它**不包含**开发工具：编译器、调试器和其他工具。

**JVM——Java Virtual Mechinal（JAVA虚拟机）**。JVM是JRE的一部分，它是一个虚构出来的计算机，是通过在实际的计算机上仿真模拟各种计算机功能来实现的。JVM有自己完善的硬件架构，如处理器、堆栈、寄存器等，还具有相应的指令系统。JVM的主要工作是解释自己的质量集（即字节码）并映射到本地的CPU的指令集或OS的系统调用。Java语言是跨平台运行的，其实就是不同的操作系统，使用不同的JVM映射规则，让其与操作系统无关，完成了跨平台性。JVM对上层的Java源文件是不关心的，它关注的只是由源文件生成的类文件。类文件的组成包括JVM指令集，符合表以及一些补助信息。

Java文件执行过程：

![Java程序运行流程](https://img-blog.csdnimg.cn/img_convert/03efc6df62fa56a1cbb692e9f7ea8537.png)





Java语言最为显著的特性有两方面，一个是**跨平台的特性**，即“一次书写，到处执行”(Write Once,Run Anywhere)；另一是**垃圾回收**（GC，Garage Collection）机制，通过使用垃圾收集器（Garbage Collector），大部分情况下，程序员不需要自己操心内存的分配和回收。





02 Exception和Error有什么区别？

03 final、finally、 finalize有什么不同？

04 强引用、软引用、弱引用、幻象引用有什么区别？

05 String、StringBuffer、StringBuilder有什么区别？

06 动态代理是基于什么原理？

07 int和Integer有什么区别？

08 对比Vector、ArrayList、LinkedList有何区别？

09 对比Hashtable、HashMap、TreeMap有什么不同？

10 如何保证集合是线程安全的 ConcurrentHashMap如何实现高效地线程安全

11 Java提供了哪些IO方式？ NIO如何实现多路复用？

12 Java有几种文件拷贝方式？哪一种最高效？

13 谈谈接口和抽象类有什么区别？

14 谈谈你知道的设计模式？

15 synchronized和ReentrantLock有什么区别呢？

16 synchronized底层如何实现？什么是锁的升级、降级？

17 一个线程两次调用start()方法会出现什么情况？

18 什么情况下Java程序会产生死锁？如何定位、修复？

19 Java并发包提供了哪些并发工具类？

20 并发包中的ConcurrentLinkedQueue和LinkedBlockingQueue有什么区别？

21 Java并发类库提供的线程池有哪几种？ 分别有什么特点？

22 AtomicInteger底层实现原理是什么？如何在自己的产品代码中应用CAS操作？

23 请介绍类加载过程，什么是双亲委派模型？

24 有哪些方法可以在运行时动态生成一个Java类？

25 谈谈JVM内存区域的划分，哪些区域可能发生OutOfMemoryError

26 如何监控和诊断JVM堆内和堆外内存使用？

27 Java常见的垃圾收集器有哪些？

28 谈谈你的GC调优思路

29 Java内存模型中的happen-before是什么？

30 Java程序运行在Docker等容器环境有哪些新问题？

31 你了解Java应用开发中的注入攻击吗？

32 如何写出安全的Java代码？

33 后台服务出现明显“变慢”，谈谈你的诊断思路？

34 有人说“Lambda能让Java程序慢30倍”，你怎么看？

35 JVM优化Java代码时都做了什么？

36 谈谈MySQL支持的事务隔离级别，以及悲观锁和乐观锁的原理和应用场景？

37 谈谈Spring Bean的生命周期和作用域？

38 对比Java标准NIO类库，你知道Netty是如何实现更高性能的吗？

39 谈谈常用的分布式ID的设计方案？Snowflake是否受冬令时切换影响？