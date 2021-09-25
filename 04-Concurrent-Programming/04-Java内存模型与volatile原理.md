<!-- MarkdownTOC -->

- [1 Java内存模型](#1-java内存模型)
  - [1.1 可见性、原子性、有序性](#11-可见性原子性有序性)
  - [1.2 可见性](#12-可见性)
  - [1.3 有序性](#13-有序性)
    - [1.3.1 重排序](#131-重排序)
    - [1.3.2 案例分析](#132-案例分析)
- [2 volatile原理](#2-volatile原理)
  - [2.1 volatile基础语义](#21-volatile基础语义)
  - [2.2 happens-before](#22-happens-before)
  - [2.3 volatile实现原理](#23-volatile实现原理)
    - [2.3.1 数据一致性](#231-数据一致性)
    - [2.3.2 MESI协议](#232-mesi协议)
    - [2.3.3 内存屏障](#233-内存屏障)
  - [2.4 double-checked locking](#24-double-checked-locking)
  - [2.5 volatile的缺点](#25-volatile的缺点)
- [参考资料](#参考资料)

<!-- /MarkdownTOC -->

# 1 Java内存模型

## 1.1 可见性、原子性、有序性

**JMM(Java Memory Model)**，是一种基于`计算机内存模型`（定义了共享内存系统中多线程程序读写操作行为的规范）并屏蔽了各种硬件和操作系统的访问差异，保证了Java程序在各种平台下对内存的访问都能保证效果一致的机制及规范。

Java内存模型描述了Java程序中各种变量（线程共享变量）的访问规则，以及在JVM中将变量存储到内存和从内存中读取出变量这样的底层细节。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200928202635957.png" width="700px"/>
</div>
在Java中，所有实例域、静态域和数组元素都存储在堆内存中，堆内存在线程之间共享。由于线程的工作内存是线程私有内存，线程间无法互相访问对方的工作内存。所以线程 0 、线程 1 和线程 2需要读写主内存的`共享变量`时，就都先将该共享变量拷贝（load）到自己的工作内存，然后在自己的工作内存中对该变量进行所有操作，线程工作内存对**变量副本完成操作之后再将结果同步（save）至主内存**。

JMM存在三大特性：**原子性**、**可见性**、**有序性**。

**原子性**：保证指令不会受到线程**上下文切换**的影响。对共享内存的操作必须是要么全部执行成功，中间过程不能被任何外部因素打断，要么就不执行。

**可见性**：保证指令不会受**CPU缓存**的影响。多线程操作共享内存时，执行结果能够及时的同步到共享内存，确保其他线程对此结果可见。

**有序性**：保证指令不会受C**PU指令重排优化**的影响。程序的执行顺序按照代码顺序执行，在单线程环境下，程序的执行都是有序的；但是在多线程环境下，JMM 为了性能优化，编译器和处理器会对指令进行重排，程序的执行会变成无序。

## 1.2 可见性

在下面案例中，main 线程中`run`变量的修改对于子线程不可见，导致子线程无法停止：

```java
public class Test {
    static boolean run = true;

    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(() -> {
            while (run) {
            }
        });
        System.out.println(t.isAlive());
        t.start();
        System.out.println(t.isAlive());
        Thread.sleep(1000);
        run = false; // 线程t不会如预想的停下来
    }
}
```

运行结果：

```java
false
true
```

不难分析：

1. 初始状态， t 线程刚开始从主内存读取了`run`的值到工作内存。
2. 因为t线程频繁地从主存中读取run的值，JIT即时编译器会将run的值缓存至自己工作内存中的高速缓存中，减少对主存中run的访问以提高效率。
3. 1 秒之后，main 线程修改了run的值，并同步至主内存，而 t线程仍是从自己工作内存中的高速缓存中读取这个变量。
   的值，结果永远是旧值。

关于这个问题，我们可以使用sychronized关键字解决。

```java
public class Test2 {
    static boolean run = true;
    final static Object lock = new Object();

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            while (true) {
                synchronized (lock) {
                    if (!run) {
                        break;
                    }
                }
            }
        });
        thread.start();
        Thread.sleep(1);
        synchronized (lock) {
            run = false;
        }
    }
}
```

synchronized 语句块既可以保证代码块的原子性，也同时保证代码块内变量的可见性。但缺点是synchronized 是重量级锁，性能相对更低。

在上一篇文章中，我们了解到导致可见性的原因是缓存，导致有序性的原因是编译优化，那解决可见性、有序性最直接的办法就是**禁用缓存和编译优化**，但是这样问题虽然解决了，我们程序的性能可就堪忧了。

合理的方案应该是**按需禁用缓存以及编译优化**。那么，如何做到“按需禁用”呢？对于并发程序，何时禁用缓存以及编译优化只有程序员知道，那所谓“按需禁用”其实就是指按照程序员的要求来禁用。所以，为了解决可见性和有序性问题，只需要提供给程序员按需禁用缓存和编译优化的方法即可。

Java 内存模型是个很复杂的规范，可以从不同的视角来解读，站在我们这些程序员的视角，本质上可以理解为，Java 内存模型规范了 JVM 如何提供按需禁用缓存和编译优化的方法。具体来说，这些方法包括 **volatile**、**synchronized** 和 **final** 三个关键字，以及六项 **Happens-Before 规则**，文章后面主要来讲解volatile与Happens-Before 规则。

上面问题也可以用volatile关键字解决。

```java
public class Test {
    volatile static boolean run = true;

    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(() -> {
            while (run) {
            }
        });
        t.start();
        Thread.sleep(1000);
        run = false; // 线程t会停下来
    }
}
```

volatile体现的就是JMM的可见性，volatile保证的是在多个线程之间，一个线程对 volatile 变量的修改对另一个线程可
见， 但不能保证多线程的原子性，仅用在一个写线程，多个读线程的情况。 上例从字节码理解是这样的：

```java
getstatic run // 线程 t 获取 run true
getstatic run // 线程 t 获取 run true
getstatic run // 线程 t 获取 run true
getstatic run // 线程 t 获取 run true
putstatic run // 线程 main 修改 run 为 false， 仅此一次
getstatic run // 线程 t 获取 run false 。
```

## 1.3 有序性

### 1.3.1 重排序

JVM 会在不影响正确性的前提下，可以调整语句的执行顺序，例如下面代码：

```java
static int i;
static int j;
// 在某个线程内执行如下赋值操作
i = ...;
j = ...; 
```

可以看到，至于是先执行 i 还是 先执行 j ，对最终的结果不会产生影响。所以，上面代码真正执行时，既可以是

```java
i = ...;
j = ...;
// 或者
j = ...;
i = ...; 
```

 这种特性称之为**重排序**，重排序主要分3种类型。

（1）编译器优化的重排序。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。

（2）指令级并行的重排序。现代处理器采用了指令级并行技术（ILP）来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。

（3）内存系统的重排序。由于处理器使用缓存和读/写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210618021511985.png" width="600px"/>
</div>
上述的1属于编译器重排序，2和3属于处理器重排序。这些**重排序可能会导致多线程程序出现内存可见性问题**。

对于编译器，JMM的编译器重排序规则会禁止特定类型的编译器重排序（不是所有的编译器重排序都要禁止）。

对于处理器重排序，JMM的处理器重排序规则会要求Java编译器在生成指令序列时，插入特定类型的**内存屏障**（Memory Barriers，Intel称之为Memory Fence）指令，通过内存屏障指令来禁止特定类型的处理器重排序。

**指令重排序优化**

事实上，现代处理器会设计为一个时钟周期完成一条执行时间最长的 CPU 指令。为什么这么做呢？可以想到指令还可以再划分成一个个更小的阶段，例如，每条指令都可以分为： `取指令 - 指令译码 - 执行指令 - 内存访问 - 数据写回`这 5 个阶段。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20201101223818595.png" width="600px"/>
</div>

> 术语参考： 
>
> instruction fetch (IF)     instruction decode (ID)     execute (EX)     memory access (MEM)     register write back (WB)

在不改变程序结果的前提下，这些指令的各个阶段可以通过重排序和组合来实现指令级并行，分阶段、分工正是提升效率的关键！

**支持流水线的处理器**

现代 CPU 支持多级指令流水线，例如支持同时执行 `取指令 - 指令译码 - 执行指令 - 内存访问 - 数据写回` 的处理器，就可以称之为五级指令流水线。这时 CPU 可以在一个时钟周期内，同时运行五条指令的不同阶段（相当于一 条执行时间最长的复杂指令），IPC = 1，本质上，流水线技术并不能缩短单条指令的执行时间，但它变相地提高了 指令地吞吐率。 

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20201101224137958.png" width="600px"/>
</div>
### 1.3.2 案例分析

```java
int num = 0;

// volatile 修饰的变量，可以禁用指令重排 volatile boolean ready = false; 可以防止变量之前的代码被重排序
boolean ready = false; 
// 线程1 执行此方法
public void actor1(I_Result r) {
	if(ready) {
		r.r1 = num + num;
     } 
 	else {
        r.r1 = 1;
	}
}
// 线程2 执行此方法
public void actor2(I_Result r) {
	num = 2;
	ready = true;
}
```

I_Result 是一个对象，有一个属性 r1 用来保存结果，可能的结果有几种？

情况1：线程1 先执行，这时 ready = false，所以进入 else 分支结果为 1。

情况2：线程2 先执行 num = 2，但没来得及执行 ready = true，线程1 执行，还是进入 else 分支，结果为1。

情况3：线程2 执行到 ready = true，线程1 执行，这回进入 if 分支，结果为 4。

情况4：线程2 执行 ready = true，切换到线程1，进入 if 分支，相加为 0，再切回线程2 执行 num = 2。

情况4出现的在于出现了指令重排，指令重排是 JIT 编译器在运行时的一些优化，这个现象需要通过大量测试才能复现，可以使用jcstress工具进行测试。上面仅是从代码层面体现出了有序性问题，下面在讲到 double-checked locking 问题时还会从java字节码的层面了解有序性的问题。

重排序也需要遵守一定规则：

1. 重排序操作不会对存在数据依赖关系的操作进行重排序。比如：`a=1;b=a;` 这个指令序列，由于第二个操作依赖于第一个操作，所以在编译时和处理器运行时这两个操作不会被重排序。
2. 重排序是为了优化性能，但是不管怎么重排序，单线程下程序的执行结果不能被改变。比如：`a=1;b=2;c=a+b`这三个操作，第一步（a=1)和第二步(b=2)由于不存在数据依赖关系，所以可能会发生重排序，但是c=a+b这个操作是不会被重排序的，因为需要保证最终的结果一定是c=a+b=3。

**重排序在单线程模式下是一定会保证最终结果的正确性**，但是在多线程环境下，问题就出来了。解决方法：**volatile 修饰的变量，可以禁用指令重排**。

> Tips：使用synchronized并不能解决所有有序性问题，但是变量完全在synchronized代码块的保护范围内，那么变量就不会被多个线程同时操作，也不用考虑有序性问题！

   #     2 volatile原理

## 2.1 volatile基础语义

从上文可知，一旦一个共享变量（类的成员变量、类的静态成员变量）被volatile修饰之后，那么就具备了两层语义：

(1) **保证可见性**：**当一个共享变量被 volatile 修饰时，它会把修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取最新值**。

(2) **保证有序性**：**禁止进行指令重排序**。对一个volatile域的写，happens-before于任意后续对这个volatile域的读，禁止了局部指令重排序。

> Tips：**volatile不能保证原子性**，对任意单个volatile变量的读/写具有原子性，但类似于 volatile++这种复合操作不具有原子性。为什么呢？
>
> 假设有线程A、B同时执行volatile变量 i++操作。根据缓存一致性，一个处理器的缓存回写到主存会导致其他处理器的缓存失效。当线程A，B同时执行i的自增的时候，先执行完的线程A回写数据到主存，导致B的缓存变量无效（因此执行+1操作已经没有意义），直接从主存读取最新的值，这样就少做了一次+1。
>
> 那AtomicInteger为什么能保证原子性呢？虽然AtomicInteger用的到volatile变量，它的自增方法incrementAndGet()最终调用的Unsafe类的compareAndSetInt()方法，此方法是采用的无锁算法，是一个原子操作。

我们先来看JVM如何保证有序性的？由此引出Java中的happen-before规则。

## 2.2 happens-before

JMM可以通过happens-before关系**JMM可以通过happens-before（先行发生）关系向程序员提供跨线程的内存可见性保证**。

《JSR-133:Java Memory Model and Thread Specif ication》对happens-before关系的定义如下。

（1）如果一个操作happens-before另一个操作，那么第一个操作的所有**执行结果**将对第二个操作可见，而且第一个操作的执行顺序一般排在第二个操作之前。

（2）两个操作之间存在happens-before关系，**并不**意味着Java平台的具体实现必须要按照happens-before关系指定的顺序来执行。如果重排序之后的执行结果，与按happens-before关系来执行的结果一致，JMM允许这种重排序。

**happens-before具体规则**：

（1）**程序次序规则**：一个线程内，按照代码顺序，书写在前面的操作，happens-before 于书写在后面的操作。

（2）**监视器锁规则**：对一个锁的解锁，happens-before于随后对这个锁的加锁。

```java
    // 线程解锁lock之前对变量的写，对于接下来对lock加锁的其它线程对该变量的读可见。
	static int x;
	static Object lock = new Object();
	new Thread(()->{
		synchronized(lock) {
			x = 10;
		}
	},"t1").start();
	new Thread(()->{
		synchronized(lock) { 
			System.out.println(x);
		}
	},"t2").start();
```

（3）**volatile变量规则**：对一个volatile域的写，happens-before于任意**后续**对这个volatile域的读。

```java
    volatile static int x;
    public static void main(String[] args) {
        new Thread(()->{
            x = 10;
        },"t1").start();
        new Thread(()->{
            System.out.println(x);
        },"t2").start();
    }
```

（4）**传递规则**：如果A happens-before B，且B happens-before C，那么A happens-before C。

```java
	// 具有传递性，如果 x hb-> y 并且 y hb-> z 那么有 x hb-> z ，配合 volatile 的防指令重排
	volatile static int x;
    static int y;
    
    public static void main(String[] args) {
        new Thread(() -> {
            y = 10;
            x = 20;
        }, "t1").start();
        new Thread(() -> {
            // x=20 对 t2 可见, 同时 y=10 也对 t2 可见
            System.out.println(x);
        }, "t2").start();
    }
```

（5）**线程启动规则**：如果线程A执行操作ThreadB.start()（启动线程B），那么A线程的ThreadB.start()操作happens-before于线程B中的任意操作。

 ```java
	// 线程 start 前对变量的写，对该线程开始后对该变量的读可见      
	static int x;
    x = 10;
    new Thread(()->{
       System.out.println(x);
    },"t2").start();
 ```

（6）**线程中断规则**：对线程 interrupt 方法的调用，happens-before 被中断线程的代码检测到中断事件的发生。

```java
    // 线程 t1 打断 t2（interrupt）前对变量的写，对于其他线程得知t2被打断后对变量的读可见（通过
    // t2.interrupted 或 t2.isInterrupted）
	static int x;
    public static void main(String[] args) {
        Thread t2 = new Thread(()->{
            while(true) {
                if(Thread.currentThread().isInterrupted()) {
                    System.out.println(x);
                    break;
                }
            }
        },"t2");
        t2.start();
        new Thread(()->{
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            x = 10;
            t2.interrupt();
        },"t1").start();
        while(!t2.isInterrupted()) {
            Thread.yield();
        }
        System.out.println(x);
    }
```

（7）**线程终结规则**：如果线程A执行操作ThreadB. join()并成功返回，那么线程B中的任意操作happens-before于线程A从ThreadB. join()操作成功返回。

```java
	// 线程结束前对变量的写，对其它线程得知它结束后的读可见（比如其它线程调用 t1.isAlive() 或 t1.join()等待它结束）    
	static int x;
    Thread t1 = new Thread(()->{
     x = 10;
    },"t1");
    t1.start();
    t1.join();
    System.out.println(x);
```

（8）**对象终结规则**：一个对象的初始化完成，happens-before 它的 finalize() 方法的开始。

我们着重看第三点 Volatile规则：对 volatile变量的写操作，happen-before 后续的读操作。

为了实现 volatile 内存语义，JMM会重排序，其规则如下：

| 是否能重排序   | 第二个操作 | 第二个操作 | 第二个操作 |
| -------------- | ---------- | ---------- | ---------- |
| **第一个操作** | 普通读/写  | Volatile读 | Volatile写 |
| 普通读/写      |            |            | No         |
| Volatile读     | No         | No         | No         |
| Volatile写     |            | No         | No         |

**当第二个操作是 volatile 写操作时，不管第一个操作是什么，都不能重排序**。

## 2.3 volatile实现原理

**volatile 的底层实现原理是内存屏障**（Memory Barrier/Memory Fence），下面这段话摘自《深入理解Java虚拟机》：

“观察加入volatile关键字和没有加入volatile关键字时所生成的汇编代码发现，**加入volatile关键字时，会多出一个lock前缀指令**”。

lock前缀指令实际上相当于一个**内存屏障**（也成内存栅栏），内存屏障会提供3个功能：

（1）它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成。

（2）它会强制将对缓存的修改操作立即写入主存。

（3）如果是写操作，它会导致其他CPU中对应的缓存行无效。

简单地说，Java在实现volatile时，就是写入了一条Lock前缀的汇编指令（内存屏障），这个指令会强制CPU缓存数据回写到内存中，然后让其他CPU的缓存无效化，保证数据的一致性。

至于如何让CPU缓存数据回写到主内存中，又让其他CPU的缓存无效化，就牵扯到**MESI协议**了。

### 2.3.1 数据一致性

这里先说一说数据的一致性。其实在系统的很多地方都存在数据一致性的相关问题。除了在并发编程中保证共享变量数据的一致性之外，还有数据库的 ACID 中的 C（Consistency 一致性）、分布式系统的 CAP 理论中的 C（Consistency 一致性）。在并发编程中，Java 是通过共享内存来实现共享变量操作的，所以在多线程编程中就会涉及到数据一致性的问题。

我们知道，共享变量存储在JVM堆或方法区中，而堆内存和方法区的数据是线程共享的。而堆内存中的共享变量在被不同线程操作时，会被加载到自己的工作内存中，也就是 CPU 中的高速缓存。正是如此，在多线程对是共享变量操作时，容易出现数据不一致的问题。

> CPU 缓存可以分为一级缓存（L1）、二级缓存（L2）和三级缓存（L3），每一级缓存中所储存的全部数据都是下一级缓存的一部分。当 CPU 要读取一个缓存数据时，首先会从一级缓存中查找；如果没有找到，再从二级缓存中查找；如果还是没有找到，就从三级缓存或内存中查找。

JVM为了性能优化，可能对执行代码进行重排序，也可能会给并发编程带来数据一致性的问题。

为了解决这些问题，Java 提出了 happens-before 规则来规范线程的执行顺序。结合这些规则，我们可以将一致性分为以下几个级别：

* **强一致性（严格一致性）**：所有的读写操作都按照全局时钟下的顺序执行，且任何时刻线程读取到的缓存数据都是一样的，Hashtable 就是严格一致性；

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210618222208132.png" width="800px"/>
</div>


* **顺序一致性**：多个线程的整体执行可能是无序的，但对于单个线程而言执行是有序的，要保证任何一次读都能读到最近一次写入的数据，volatile 可以阻止指令重排序，所以修饰的变量的程序属于顺序一致性；

<div align="center">  
<img src="https://img-blog.csdnimg.cn/2021061822225340.png" width="800px"/>
</div>


* **弱一致性**：不能保证任何一次读都能读到最近一次写入的数据，但能保证最终可以读到写入的数据，单个写锁 + 无锁读，就是弱一致性的一种实现。

### 2.3.2 MESI协议

为了解决缓存不一致的问题，我们就需要有一种机制，来同步两个不同核心里面的缓存数据。那这样的机制需要满足什么条件呢？

* 第一点叫**写传播**（Write Propagation）。写传播是说，在一个 CPU 核心里，我们的 Cache 数据更新，必须能够传播到其他的对应节点的 Cache Line 里。
* 第二点叫**事务的串行化**（Transaction Serialization），事务串行化是说，我们在一个 CPU 核心里面的读取和写入，在其他的节点看起来，顺序是一样的。要实现事务串行化，需要实现以下两点：
  * CPU的数据操作需要通知其他CPU核心。
  * 如果两个CPU核心里有同一个数据Cache操作，需要加锁。只有拿到Cache Block的锁后，才能进行对应的数据更新。

要解决缓存一致性问题，首先要解决的是多个 CPU 核心之间的数据传播问题。最常见的一种解决方案呢，叫作**总线嗅探（Bus Snooping）**。这个策略，本质上就是把所有的读写请求都通过总线（Bus）广播给所有的 CPU 核心，然后让各个核心去“嗅探”这些请求，再根据本地的情况进行响应。基于总线嗅探机制，其实还可以分成很多种不同的缓存一致性协议。其中最常用的，就是MESI 协议。

**MESI 协议，是一种叫作写失效（Write Invalidate）的协议**。在写失效协议里，只有一个 CPU 核心负责写入数据，其他的核心，只是同步读取到这个写入。在这个 CPU 核心写入 Cache 之后，它会去广播一个“失效”请求告诉所有其他的 CPU 核心。其他的 CPU 核心通过嗅探在总线上传播的数据来检查自己缓存是不是过期，如果发现自己缓存行对应的内存地址被修改，就会将当前的缓存行设置成“失效”状态，当需要对数据进行修改操作的时候，会重新从系统内存中把数据读到CPU缓存里。

MESI 协议的由来呢，来自于我们对 Cache Line 的四个不同的标记，分别是：

* M：代表已修改（Modified）
* E：代表独占（Exclusive）
* S：代表共享（Shared）
* I：代表已失效（Invalidated）

所谓的“已修改”，就是的“脏”的 Cache Block，Cache Block 里面的内容我们已经更新过了，但是还没有写回到主内存里面；而所谓的“已失效“，自然是这个 Cache Block 里面的数据已经失效了，我们不可以相信这个 Cache Block 里面的数据。

独占和共享状态，就好像我们在多线程应用开发里面的读写锁机制，确保了我们的缓存一致性。独占状态是可以变为读共享的。之所以区分为两个状态，是因为独占状态下写无须通知其他CPU失效它对应的缓存行，减少不必要的操作。 共享状态写会先失效其他CPU的缓存。写并不一定是第一个独占的CPU可以写，可以由后来共享的CPU写，只要写入后，把同一个变量的其他CPU缓存行失效即可，类比volitle理解。

而整个 MESI 的状态变更，则是根据来自自己 CPU 核心的请求，以及来自其他 CPU 核心通过总线传输过来的操作信号和地址信息，可以用一个有限状态机来表示它的状态流转。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210618223902110.png" width="600px"/>
</div>

### 2.3.3 内存屏障

前面也讲了，Lock前缀指令实际上相当于一个内存屏障（也成内存栅栏），它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成。

基于保守策略的JMM内存屏障插入策略。每个volatile写操作的前插入StoreStore屏障、每个volatile写操作的后插入StoreLoad屏障、每个volatile读操作后插入LoadLoad屏障、 每个volatile读操作后插入LoadStore屏障。

下图是完成happens-before规则所需要的内存屏障：

| 是否能重排序   | 第二个操作 | 第二个操作 | 第二个操作 | 第二个操作 |
| -------------- | ---------- | ---------- | ---------- | ---------- |
| **第一个操作** | 普通读     | 普通写     | Volatile读 | Volatile写 |
| 普通读         |            |            |            | LoadStore  |
| 普通写         |            |            |            | StoreStore |
| Volatile读     | LoadLoad   | LoadStore  | LoadLoad   | LoadStore  |
| Volatile写     |            |            | StoreLoad  | StoreStore |

(1) LoadLoad 屏障
执行顺序：Load1—>Loadload—>Load2
确保Load2及后续Load指令加载数据之前能访问到Load1加载的数据。

(2) StoreStore 屏障
执行顺序：Store1—>StoreStore—>Store2
确保Store2以及后续Store指令执行前，Store1操作的数据对其它处理器可见。

(3) LoadStore 屏障
执行顺序： Load1—>LoadStore—>Store2
确保Store2和后续Store指令执行前，可以访问到Load1加载的数据。

(4) S toreLoad 屏障
执行顺序: Store1—> StoreLoad—>Load2

**案例分析**

还是以之前代码为例：

```java
    int num = 0;						// 共享变量num
										// 根据程序顺序规则，num happens-before ready
    volatile boolean ready = false;		// volatile变量ready
    // 线程1 执行此方法
    public void actor1(I_Result r) {
 										// LoadLoad屏障
        if(ready) {						// ready是被volatile修饰的，读取值带LoadLoad屏障
            r.r1 = num + num;
        }
        else {
            r.r1 = 1;
        }
    }
    // 线程2 执行此方法
    public void actor2(I_Result r) {
        num = 2;
        ready = true;					// ready是被volatile修饰的 ，赋值带LoadStore屏障
										// LoadStore屏障
    }
```

##  2.4 double-checked locking

下面以著名的 double-checked locking 单例模式为例，这是volatile最常使用的地方。

实现单例模式时，如果未考虑多线程的情况，就容易写出下面的代码：

```java
public final class Singleton {
    private Singleton() {
    }

    private static Singleton INSTANCE = null;

    public static Singleton getInstance() {
        // 首次访问会同步，而之后的使用不用进入synchronized
        synchronized (Singleton.class) {
            if (INSTANCE == null) {
                INSTANCE = new Singleton();
            }
        }
        return INSTANCE;
    }
}
```

上面代码的问题在于，即使已经产生了单实例之后，之后调用了getInstance()方法之后还是会加锁，这会严重影响性能！

**双重检查锁**（double checked locking）是对上述问题的一种优化。

```java
public final class Singleton {
    private Singleton() {
    }
    
    private static Singleton INSTANCE = null;
    
    public static Singleton getInstance() {
        if(INSTANCE == null) {
            // 首次访问会同步，而之后的使用没有 synchronized
            synchronized(Singleton.class) {
                if (INSTANCE == null) {
                    INSTANCE = new Singleton();// error
                }
            }
        }
        return INSTANCE;
    }
}
```

如果这样写，运行顺序就成了：

（1）检查变量是否被初始化(不去获得锁)，如果已被初始化则立即返回。

（2）获取锁。

（3）再次检查变量是否已经被初始化，如果还没被初始化就初始化一个对象。

这样，除了初始化的时候会出现加锁的情况，后续的所有调用都会避免加锁而直接返回，解决了性能消耗的问题。

上述写法看似解决了问题，但在多线程环境下，是有很大的**隐患**的。`if(INSTANCE == null`代码没有在同步代码块synchronized中，不能享有synchronized保证的原子性、可见性。

查看getInstance 方法对应的字节码为：

```java
public static com.kai.demo.memory.Singleton getInstance();
    descriptor: ()Lcom/kai/demo/memory/Singleton;
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=0
         0: getstatic     #2                  // 获取到INSTANCE静态变量
         3: ifnonnull     37
         6: ldc           #3                  // 获得Singleton.class类对象
         8: dup								  // 将类对象的引用地址复制了一份->临时类对象引用
         9: astore_0						  // 临时类对象引用 -> 存入局部变量表slot 1中
        10: monitorenter					  // 将类对象的Mark Word置为指向Monitor指针
        11: getstatic     #2                  // 再次获取到INSTANCE静态变量
        14: ifnonnull     27			      
        17: new           #3                  // 新建一个Singleton实例，实例对象引用入栈
        20: dup								  // 复制Singleton实例的引用->临时引用
        21: invokespecial #4                  // 临时实例引用调用构造方法<init>
        24: putstatic     #2                  // 实例的赋值操作
        27: aload_0							  // 获取到临时类对象引用
        28: monitorexit						  // 将lock对象的Mark Word重置，唤醒EntryList
        29: goto          37
        32: astore_1
        33: aload_0
        34: monitorexit
        35: aload_1
        36: athrow
        37: getstatic     #2                  // return INSTANCE;
        40: areturn
        --- omit ---
```

主要来看17-24步：

* 17：创建Singleton实例，将实例对象引用入栈

* 20：复制一份对象引用（临时引用）
* 21：利用临时引用，调用构造方法
* 24：利用对象引用，赋值给静态INSTANCE

编译器为了性能优化，可能会将21和24进行**重排序**。如果两个线程 t1、t2 按时间序列执行：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20201102150244872.png" width="600px"/>
</div>

由于 `0: getstatic` 这行代码（` if(INSTANCE == null)`）在 monitor 控制之外，t2可以越过 monitor 读取INSTANCE 变量的值。这时 t1 还未完全将构造方法执行完毕， t2 拿到的是将是一个未初始化完毕的单例。

**double-checked locking 解决方法**

对 INSTANCE 使用 volatile 修饰即可。

```java
public final class Singleton {
    private Singleton() {
    }
    
    private static volatile Singleton INSTANCE = null;

    public static Singleton getInstance() {
        // 实例没创建，才会进入内部的 synchronized代码块
        if (INSTANCE == null) {
            synchronized (Singleton.class) { // t2
                // 也许有其它线程已经创建实例，所以再判断一次
                if (INSTANCE == null) { // t1
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }
}
```

字节码上看不出来 volatile 指令的效果：

```java
// -------------------------------------> 加入对 INSTANCE 变量的LoadLoad屏障
 0 getstatic #2 <com/kai/demo/memory/Singleton.INSTANCE>
 3 ifnonnull 37 (+34)
 6 ldc #3 <com/kai/demo/memory/Singleton>
 8 dup
 9 astore_0
10 monitorenter-----------------------> 保证原子性、可见性
11 getstatic #2 <com/kai/demo/memory/Singleton.INSTANCE>
14 ifnonnull 27 (+13)
17 new #3 <com/kai/demo/memory/Singleton>
20 dup
21 invokespecial #4 <com/kai/demo/memory/Singleton.<init>>
24 putstatic #2 <com/kai/demo/memory/Singleton.INSTANCE>
// -------------------------------------> 加入对 INSTANCE 变量的LoadStore屏障
27 aload_0
28 monitorexit-----------------------> 保证原子性、可见性
29 goto 37 (+8)
32 astore_1
33 aload_0
34 monitorexit
35 aload_1
36 athrow
37 getstatic #2 <com/kai/demo/memory/Singleton.INSTANCE>
40 areturn
```

如上面的注释内容所示，读写 volatile 变量操作（即getstatic操作和putstatic操作）时会加入内存屏障（Memory Barrier（Memory Fence）），保证下面两点：

1. 可见性

   （1）写屏障（sfence）保证在该屏障之前的 t1 对共享变量的改动，都同步到主存当中

   （1）而读屏障（lfence）保证在该屏障之后 t2 对共享变量的读取，加载的是主存中最新数据
   
2. 有序性

   （1）写屏障会确保指令重排序时，不会将写屏障之前的代码排在写屏障之后

   （2）读屏障会确保指令重排序时，不会将读屏障之后的代码排在读屏障之前
   
3. 更底层是读写变量时使用 lock 指令来多核 CPU 之间的可见性与有序性

<div align="center">  
<img src="https://img-blog.csdnimg.cn/2020110215235336.png" width="600px"/>
</div>
## 2.5 volatile的缺点

简单来说，使用volatile还是有两个大的隐患：

其一，volatile禁止了指令重排，还是会带来一定的性能的问题；

其二，根据MESI缓存一致性协议实现，用到了 CPU 的嗅探机制，需要不断的对内存总线进行内存嗅探，大量交互会导致带宽达到峰值，滥用volatile可能会导致总线风暴。

# 参考资料

* 《JAVA并发编程的艺术》
* [volatile关键字的作用、原理](https://juejin.im/post/6844903502418804743)
* [MESI协议：如何让多核CPU的高速缓存保持一致？](https://time.geekbang.org/column/article/109874)
* [什么是数据的强、弱一致性？](https://time.geekbang.org/column/article/105756)