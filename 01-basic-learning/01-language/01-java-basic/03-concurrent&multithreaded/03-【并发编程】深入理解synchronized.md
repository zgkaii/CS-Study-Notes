## 1 线程安全问题

在并发编程中，需要处理两个关键问题：线程之间如何**通信**及线程之间如何**同步**（这里的线程是指并发执行的活动实体）。

**通信**是指线程之间以何种机制来交换信息。**Java中并发采用的是共享内存模型**，在共享内存的并发模型里，线程之间共享程序的公共状态，通过写-读内存中的公共状态进行**隐式通信**。

而**同步**是指程序中用于控制不同线程间操作发生相对顺序的机制。在共享内存并发模型里，同步是显式进行的。程序员必须显式指定某个方法或某段代码需要在线程之间**互斥**执行。

Java内存模型（Java Memory Model，JMM）描述了Java程序中各种变量（线程共享变量）的访问规则，以及在JVM中将变量存储到内存和从内存中读取出变量这样的底层细节。

![](https://img-blog.csdnimg.cn/20200928202635957.png)****

在Java中，所有实例域、静态域和数组元素都存储在堆内存中，堆内存在线程之间共享。由于线程的工作内存是线程私有内存，线程间无法互相访问对方的工作内存。所以线程 0 、线程 1 和线程 2需要读写主内存的`共享变量`时，就都先将该共享变量拷贝（load）到自己的工作内存，然后在自己的工作内存中对该变量进行所有操作，线程工作内存对**变量副本完成操作之后再将结果同步（save）至主内存**。

因此，在[线程上下文切换](https://blog.csdn.net/KAIZ_LEARN/article/details/109225490)期间，**多线程读写共享内存中的全局变量及静态变量容易引发竞态条件**。

下面就用案例1来说明：

```java
@Slf4j
public class SafeTest {
    static int count = 0;

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            for (int i = 1; i < 5000; i++) {
                count++;
            }
        });
        Thread t2 = new Thread(() -> {
            for (int i = 1; i < 5000; i++) {
                count--;
            }
        });
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        log.debug("count的值是: {}", count);
    }
}
```

按照常理而言，count最终结果应该为0。多次运行发现，最终count值还可能是正数，也可能为负数。这是为什么呢？

从字节码的层面进行分析：

```java
 0 iconst_1
 1 istore_0
 2 iload_0
 3 sipush 5000
 6 if_icmpge 23 (+17)
 9 getstatic #10 <com/kai/demo/basic/SafeTest.count> // 获取静态变量i的值
12 iconst_1 // 准备常量1
13 iadd // 自增
14 putstatic #10 <com/kai/demo/basic/SafeTest.count> // 将修改后的值存入静态变量i
17 iinc 0 by 1
20 goto 2 (-18)
23 return
```

```java
 0 iconst_1
 1 istore_0
 2 iload_0
 3 sipush 5000
 6 if_icmpge 23 (+17)
 9 getstatic #10 <com/kai/demo/basic/SafeTest.count> // 获取静态变量i的值
12 iconst_1 // 准备常量1
13 isub // 自减
14 putstatic #10 <com/kai/demo/basic/SafeTest.count> // 将修改后的值存入静态变量i
17 iinc 0 by 1
20 goto 2 (-18)
23 return
```

可见`count++` 和 `count--` 操作实际都是需要这个4个指令完成的，那么这里问题就来了！Java 的内存模型如下，完成静态变量的自增、自减需要在主存和工作内存中进行数据交换。

正常顺序执行：

![](https://img-blog.csdnimg.cn/20201025112256131.png)

实际出现负数的情况：

![](https://img-blog.csdnimg.cn/20201025112454585.png)

实际出现正数的情况：

![](https://img-blog.csdnimg.cn/20201025112642320.png)

像上面`count++` 和 `count--` 的代码所在区域又称**临界区**（**一段代码内如果存在对共享资源的多线程读写操作，那么称这段代码为临界区**）。

跟上面案例一样。如果多个线程临界区代码执行竞争同一资源时，对资源的访问顺序敏感，执行时序的不同导致会出现某种不正常的行为，就称存在**竞态条件（Race Condition）**。

为避免竞态条件的出现，保证java共享内存的原子性、可见性、有序性，很有必要保持线程同步。Java中提供了synchronized、volatile关键字与`Lock`类。

## 2 初识synchronized

synchronized采用**互斥同步**（Mutual Exclusion & Synchnronization）的方式，让多个线程并发访问共享数据时，保证共享数据在同一时刻只被一个（或一些，使用信号量的时候）线程使用。互斥是实现同步的一种手段，临界区、互斥量和信号量都是主要的互斥实现方式。因此在互斥同步四个字中，互斥是因，同步是果；互斥是方法，同步是目的。

### 2.1 使用场景

在Java代码中使用synchronized可使用在代码块和方法中，根据Synchronized用的位置可以有这些使用场景：

![](https://img-blog.csdnimg.cn/20201025123258136.png)

可见，synchronized的使用场景主要有3种：

* 修饰静态方法，给当前类对象加锁，进入同步方法时需要获得类对象的锁；
* 修饰实例方法，给当前实例变量加锁，进入同步方法时需要获得当前实例的锁；
* 修饰同步方法块，指定加锁对象（实例对象/是类变量），进入同步方法块时需要获得加锁对象的锁。

### 2.2 案例分析

下面，将synchronized应用到上面的案例1中：

```java
@Slf4j
public class SafeTest {
    static int count = 0;
    static final Object lock = new Object();

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            for (int i = 1; i < 5000; i++) {
                synchronized (lock){
                    count++;
                }
            }
        });
        Thread t2 = new Thread(() -> {
            for (int i = 1; i < 5000; i++) {
                synchronized (lock){
                    count--;
                }
            }
        });
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        log.debug("count的值是: {}", count);
    }
}
```

多次测试结果都为0。这是因为synchronized利用对象锁保证了临界区代码的原子性，临界区内的代码在外界看来是不可分割的，不会被线程切换所打断。

![](https://img-blog.csdnimg.cn/20201025213813109.png)

synchronized为什么这么神奇，它到底做了什么呢？下面就进一步探讨一下synchronized的实现原理。

## 3 Monitor机制

先来了解两个重要的概念：“Java对象头”、"锁记录"。

### 3.1 Java对象头

[对象实例化内存布局与访问定位](https://blog.csdn.net/KAIZ_LEARN/article/details/109281030)中描述了，JVM中对象内存布局主要分为三块区域：对象头区、实例数据区和填充区。

<img src="https://img-blog.csdnimg.cn/202010111456447.jpg" style="zoom: 67%;" />

**Synchronized用到的锁就是存在Java对象头里的**。对象头区又主要分为两部分，分别是 运行时元数据（Mark Word）和 类型指针。

如果对象是数组类型，则虚拟机用3个字宽（Word）存储对象头，如果对象是非数组类型，则用2字宽存储对象头。在32位虚拟机中，1字宽等于4字节，即32bit。 Java对象头具体结构描述如下：

![](https://img-blog.csdnimg.cn/20201026155604418.png#pic_center)

**Mark Word用于存储对象自身的运行时数据，如：哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程 ID、偏向时间戳等**。32位JVM的Mark Word的默认存储结构如下：

![](https://img-blog.csdnimg.cn/2020102615580277.png#pic_center)

在运行期间，Mark Word里存储的数据会随着锁标志位的变化而变化。Mark Word可能变化为存储以下4种数据：

![](https://img-blog.csdnimg.cn/20201026155859957.png)

在64位虚拟机下，Mark Word是64bit大小的，其存储结构如下：

![](https://img-blog.csdnimg.cn/20201026155937989.png#pic_center)

### 3.2 锁记录

在线程进入同步代码块的时候，如果此**同步对象没有被锁定，它的锁标志位是01**，则虚拟机首先在当前线程的栈中创建我们称之为**“锁记录（Lock Record）”**的空间，用于存储锁对象的Mark Word的拷贝，官方把这个拷贝称为Displaced Mark Word。整个Mark Word及其拷贝至关重要。

**Lock Record是线程私有的数据结构**，每一个线程都有一个可用Lock Record列表，同时还有一个全局的可用列表。**每一个被锁住的对象Mark Word都会和一个Lock Record关联**（对象头的Mark Word中的Lock Word指向Lock Record的起始地址），同时**Lock Record中有一个Owner字段存放拥有该锁的线程的唯一标识**（或者`object mark word`），表示该锁被这个线程占用。

| Lock Record | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| Owner       | 初始时为NULL表示当前没有任何线程拥有该monitor record，当线程成功拥有该锁后保存线程唯一标识，当锁被释放时又设置为NULL。 |
| EntryQ      | 关联一个系统互斥锁（semaphore），阻塞所有试图锁住monitor record失败的线程。 |
| RcThis      | 表示blocked或waiting在该monitor record上的所有线程的个数。   |
| Nest        | 用来实现重入锁的计数。                                       |
| HashCode    | 保存从对象头拷贝过来的HashCode值（可能还包含GC age）。       |
| Candidate   | 用来避免不必要的阻塞或等待线程唤醒，因为每一次只有一个线程能够成功拥有锁，如果每次前一个释放锁的线程唤醒所有正在阻塞或等待的线程，会引起不必要的上下文切换（从阻塞到就绪然后因为竞争锁失败又被阻塞）从而导致性能严重下降。Candidate只有两种可能的值0表示没有需要唤醒的线程1表示要唤醒一个继任线程来竞争锁 |

### 3.3 Monitor原理

**Monitor，常被翻译为“监视器”或者“管程”**。

操作系统在面对进程/线程间同步时，所支持的最重要的同步原语即是semaphore **信号量** 和 mutex **互斥量**。在使用基本的 mutex 进行并发控制时，需要程序员非常小心地控制 mutex 的 down 和 up 操作，否则很容易引起死锁等问题。为了更容易地编写出正确的并发程序，在 mutex 和 semaphore 的基础上，提出了更高层次的同步原语 Monitor。

不过需要注意的是，操作系统本身并不支持 Monitor机制，Monitor是属于编程语言的范畴。例如C语言它就不支持 monitor，Java 语言支持 Monitor。Java对象则是天生的Monitor，每一个Java对象都有成为Monitor的“潜质”。这是为什么呢？

因为在Java的设计中，每一个对象自打娘胎里出来，就带了一把看不见的锁，通常我们叫**“内部锁”，或者“Monitor锁”**，或者“Intrinsic lock”。有了这个锁的帮助，**只要把类的对象方法或者代码块用synchronized关键字修饰，就会先获取到与 synchronized 关键字绑定在一起的 Object 的对象锁**，这个锁会限定其它线程进入与这个锁相关的synchronized 代码区域。而这个对象锁，也就是一个货真价实的Monitor。

因此，可以把Monitor理解为一种同步工具，也可以理解是一种同步机制，它通常被描述为一个对象。其主要特点有：

- 对象的所有方法都被“互斥”的执行。也就是说，同一个时刻，只有一个 进程/线程 能进入 Monitor 中定义的临界区。
- 通常提供singal机制。即允许正持有“许可”的线程暂时放弃“许可”，等待某个谓词成真（条件变量），而条件成立后，当前进程可以“通知”正在等待这个条件变量的线程，让他可以重新去获得运行许可。

使用`synchronized`给对象上锁时，该对象头的Mark Word中就被设置为指向Monitor对象的指针。**Mark Word锁标识位为10，其中指针指向的是Monitor对象的起始地址**。在Java虚拟机（HotSpot）中，**Monitor是由ObjectMonitor实现的**，其主要数据结构如下（位于HotSpot虚拟机源码objectMonitor.hpp文件，C++实现的）：

```java
  // initialize the monitor, exception the semaphore, all other fields
  // are simple integers or pointers
  ObjectMonitor() {
    _header       = NULL;
    _count        = 0;			// 记录个数
    _waiters      = 0,
    _recursions   = 0;
    _object       = NULL;
    _owner        = NULL;
    _WaitSet      = NULL;		// 处于wait状态的线程，会被加入到_WaitSet
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ;		// 处于等待锁block状态的线程，会被加入到该列表
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
    _previous_owner_tid = 0;
  }
```

ObjectMonitor中有两个队列，**_WaitSet 和 _EntryList**，用来保存ObjectWaiter对象列表（ **每个等待锁的线程都会被封装成ObjectWaiter对象** ），**_owner指向持有ObjectMonitor对象的线程**，当多个线程同时访问一段同步代码：

![](https://img-blog.csdnimg.cn/20201026000053726.png)

* 线程需要获取 Object 的锁时，会被放入 EntrySet（入口区） 中进行等待（enter）。
* 如果该线程获取到了锁（acquire），成为当前锁的 Owner。
* 如果根据程序逻辑，一个已经获得了锁的线程缺少某些外部条件，而无法继续进行下去（例如生产者发现队列已满或者消费者发现队列为空），那么该线程可以通过调用 wait 方法将锁释放（release），进入 Wait Set （等待区）中阻塞（BLOCKED）进行等待。
* 其它线程在这个时候有机会获得锁，从而使得之前不成立的外部条件成立，这样先前被阻塞的线程就可以重新进入 EntrySet 去竞争锁（acquire）。这个外部条件在 Monitor 机制中称为**条件变量**。

> Tips：由于进入等待区只有号入口，由此可以推断，一个线程只有在持有监视器时才能执行wait操作，处于等待的线程只有再次获得监视器才能退出等待状态。

下面再从字节码角度理解一下Monitor原理：

```java
public class Test {
    static int counter = 0;
    static final Object lock = new Object();

    public static void main(String[] args) {
        synchronized (lock) {
            counter++;
        }
    }
}
```

使用javap 命令反编译class文件： `javap -v Test.class`

```java
public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=3, args_size=1
         0 getstatic #2 <com/kai/demo/basic/Test.lock>		// <- 取得lock的引用（synchronized开始）
         3 dup												// 复制了一份lock临时引用
         4 astore_1											// lock临时引用 -> 存入局部变量表slot 1中
         5 monitorenter										// 将lock对象的Mark Word置为指向Monitor指针
         6 getstatic #3 <com/kai/demo/basic/Test.counter>	// <- i
         9 iconst_1											// 准备常数1
        10 iadd												// +1
        11 putstatic #3 <com/kai/demo/basic/Test.counter>	// -> i
        14 aload_1											// <- 取得lock临时引用，放入操作数栈栈顶
        15 monitorexit										// 将lock对象的Mark Word重置，唤醒EntryList
        16 goto 24 (+8)										// 执行到24行，代码结束
        //下面是异常处理指令。可见，如果出现异常，也能自动地释放锁。
        19 astore_2											// exception -> slot 2
        20 aload_1											// <- 取得lock的引用
        21 monitorexit										// 将lock对象的Mark Word重置，唤醒EntryList
        22 aload_2											// <- slot 2(exception)
        23 athrow											// throw(exception)
        24 return
      Exception table:
         from    to  target type
             6    16    19   any							// 异常检测6-16行代码（即临时区）
            19    22    19   any							
      LineNumberTable:
        line 22: 0
        line 23: 6
        line 24: 14
        line 25: 24
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      25     0  args   [Ljava/lang/String;
	  --- omit ---
```

执行同步代码块后首先要先执行**monitorenter**指令，退出的时候**monitorexit**指令。

通过分析之后可以看出，使用Synchronized进行同步，其关键就是必须要对对象的监视器monitor进行获取，当线程获取monitor后才能继续往下执行，否则就只能等待。而这个获取的过程是**互斥**的，即同一时刻只有一个线程能够获取到monitor。过程大致如下：

> 1. 如果monitor的进入数为0，则该线程进入monitor，然后将进入数设置为1，该线程即为monitor的所有者；
>
> 2. 如果线程已经占有该monitor，如果重新进入，则进入monitor的进入数加1（锁的重入性）；
>
> 3. 如果其他线程已经占用了monitor，则该线程进入阻塞状态，直到monitor的进入数为0，再重新尝试获取monitor的所有权。

monitorexit指令执行时，monitor的进入数减1，如果减1后进入数为0，那线程退出monitor，不再是这个monitor的所有者。其他被这个monitor阻塞的线程可以尝试去获取这个 monitor 的所有权。

> 上面案例中，monitorexit指令出现了两次，第1次为同步正常退出释放锁；第2次为发生异步退出释放锁。

通过上面两段描述，我们应该能很清楚的看出Synchronized的实现原理，**Synchronized的语义底层是通过一个monitor的对象来完成**。

需要注意的是，**synchronized用在同步方法上时，字节码指令中不会出现monitorenter和monitorexit指令**。

例如：

```java
public class Test1 {
    public synchronized void method() {
        System.out.println("Hello World!");
    }
}
```

使用javap 命令反编译class文件：`javap -v Test1.class`

```java
public synchronized void method();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String Hello World!
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 11: 0
        line 12: 8
	--- omit ---
```

方法的同步并没有通过指令 **`monitorenter`** 和 **`monitorexit`** 来完成，不过相对于普通方法，其常量池中多了 **`ACC_SYNCHRONIZED`** 标示符。**JVM就是根据该标示符来实现方法的同步的**：

> 当方法调用时，**调用指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置**，如果设置了，**执行线程将先获取monitor**，获取成功之后才能执行方法体，**方法执行完后再释放monitor**。在方法执行期间，其他任何线程都无法再获得同一个monitor对象。

两种同步方式本质上没有区别，只是方法的同步是一种隐式的方式来实现，无需通过字节码来完成**。两个指令的执行是JVM通过调用操作系统的互斥原语mutex来实现**，被阻塞的线程会被挂起、等待重新调度，会导致“用户态和内核态”两个态之间来回切换。

## 4 synchronized进阶

在JDK 6之前，synchronized通过Monitor来实现线程同步，Monitor可以认为直接对应底层操作系统中的互斥量（mutex）。这种同步方式的成本非常高，包括系统调用引起的内核态与用户态切换、线程阻塞造成的线程切换等。因此，后来称这种锁为“重量级锁”。

JDK 6为了减少获得锁和释放锁带来的性能消耗，引入了“偏向锁”和“轻量级锁”。所以，目前锁一共有4种状态，级别从低到高依次是：**无锁状态、偏向锁状态、轻量级锁状态和重量级锁状态**，这几个状态会随着竞争情况逐渐升级。**锁可以升级但不能降级**，意味着偏向锁升级成轻量级锁后不能降级成偏向锁。这种锁升级却不能降级的策略，目的是为了提高获得锁和释放锁的效率。

![](https://img-blog.csdnimg.cn/20201027235703700.png)

在了解各种锁状态之前，先来看一个重要的概念，“CAS”。

### 4.1 CAS

CAS全称 **Compare And Swap（比较与交换）**，是一种无锁算法。在不使用锁（没有线程被阻塞）的情况下实现多线程之间的变量同步。

CAS算法涉及到三个操作数：

- 需要读写的内存值 V。
- 进行比较的值 A。
- 要写入的新值 B。

当且仅当 V 的值等于 A 时，CAS通过原子方式用新值B来更新V的值（“比较+更新”整体是一个原子操作），否则不会执行任何操作。一般情况下，“更新”是一个不断重试的操作。

进入原子类AtomicInteger的源码，看一下AtomicInteger的定义：

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    // 获取并操作内存的数据。
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    // 存储value在AtomicInteger中的偏移量。
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }
	
    // 存储AtomicInteger的int值，该属性需要借助volatile关键字保证其在线程间是可见的。
    private volatile int value;
```

接下来，我们查看AtomicInteger的自增函数incrementAndGet()的源码时，发现自增函数底层调用的是unsafe.getAndAddInt()。

```java
    // AtomicInteger 自增方法
    public final int incrementAndGet() {
      return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
    }    

	// Unsafe.class
	public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

        return var5;
    }
```

`compareAndSwapInt`这个函数，它也是`CAS`缩写的由来。通过OpenJDK 8 来查看Unsafe.cpp的源码：

```java
// Unsafe.java
public final int getAndAddInt(Object o, long offset, int delta) {
   int v;
   do {
       v = getIntVolatile(o, offset);
   } while (!compareAndSwapInt(o, offset, v, v + delta));
   return v;
}
```

根据OpenJDK 8的源码我们可以看出，getAndAddInt()循环获取给定对象o中的偏移量处的值v，然后判断内存值是否等于v。如果相等则将内存值设置为 v + delta，否则返回false，继续循环进行重试，直到设置成功才能退出循环，并且将旧值返回。整个“比较+更新”操作封装在compareAndSwapInt()中，在JNI里是借助于一个CPU指令完成的，属于原子操作，可以保证多个线程都能够看到同一个变量的修改值。

后续JDK通过CPU的cmpxchg指令，去比较寄存器中的 A 和 内存中的值 V。如果相等，就把要写入的新值 B 存入内存中。如果不相等，就将内存值 V 赋值给寄存器中的值 A。然后通过Java代码中的while循环再次调用cmpxchg指令进行重试，直到设置成功为止。

### 4.2 自旋

#### 4.2.1 自旋锁

**线程的阻塞和唤醒需要CPU从用户态切转为核心态**，而且这种切换不易优化。如果锁的粒度很小，即锁持有的时间很短的时候。由锁竞争造成频繁地阻塞和唤醒线程就显得非常不值得，因此引入了自旋锁。

**自旋锁可以减少线程阻塞造成的线程切换**。其执行步骤如下：

* 当前线程尝试去竞争锁。
* 竞争失败，准备阻塞自己。
* 但是并没有阻塞自己，进入自旋状态（空等待，比如一个空的有限for循环）。
* 自旋状态下，继续竞争锁。
* 如果自旋期间成功获取锁，那么结束自旋状态，否则进入阻塞状态。

> **如果在自旋期间成功获取锁，那么就减少一次线程的切换。**

可见，如果持有锁的线程很快就释放了锁，那么自旋的效率就非常好，反之，自旋的线程就会白白消耗掉处理的资源。**所以自旋锁适合在持有锁时间短，并且竞争不激烈的场景下使用。**

在JDK1.6中自旋锁默认开启。可以使用`-XX:+UseSpinning`开启，`-XX:-UseSpinning`关闭自旋锁优化。

自旋的默认次数为10次，可以使用`-XX:preBlockSpin`参数修改默认的自旋次数。

#### 4.2.2 适应性自旋锁

适应性自旋，是赋予了自旋一种学习能力，它并不固定自旋10次一下。他可以根据它前面线程的自旋情况，从而调整它的自旋。

例如，线程总是自旋成功，那么虚拟机就会允许自旋等待持续的次数更多。反之，如果对于某个锁，很少有自旋能够成功，那么在以后要或者这个锁的时候自旋的次数会减少甚至省略掉自旋过程，以免浪费处理器资源。

### 4.3  锁消除与锁粗化

#### 4.3.1 锁消除

JVM对不会存在线程安全的锁进行锁消除。例如使用JDK的内置API，如StringBuffer、Vector、HashTable等时，会存在隐形的加锁操作。

```java
public void vectorTest(){
    Vector<String> vector = new Vector<String>();
    for(int i = 0 ; i < 10 ; i++){
        vector.add(i + "");
    }

    System.out.println(vector);
}
```

运行这段代码时，JVM可以明显检测到变量vector没有逃逸出方法vectorTest()之外，所以JVM可以大胆地将vector内部的加锁操作消除。

> 锁消除的依据是逃逸分析的数据支持。

#### 4.3.2 锁粗化

在遇到一连串地对同一锁不断进行请求和释放的操作时，把所有的锁操作整合成锁的一次请求，从而减少对锁的请求同步次数，这个操作叫做锁的粗化。

例如：

```java
    for(int i = 0 ; i < 10 ; i++){
		synchronized(lock){
            // 同步块 
    	}
    }
```

锁粗化后：

```java
    synchronized(lock){
        for(int i = 0 ; i < 10 ; i++){
           // 同步块 
        }
    }
```

### 4.4 偏向锁

在大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得，为了减少此类情况下线程获得锁的性能消耗，JDK6中引进了偏向锁。

当一个线程访问同步代码块并获取锁时，会在Mark Word里存储锁偏向的线程ID。在线程进入和退出同步块时不再通过CAS操作来加锁和解锁，而是检测Mark Word里是否存储着指向当前线程的偏向锁。引入偏向锁是为了在无多线程竞争的情况下尽量减少不必要的轻量级锁执行路径，因为轻量级锁的获取及释放依赖多次CAS原子指令，而偏向锁只需要在置换ThreadID的时候依赖一次CAS原子指令即可。

偏向锁只有遇到其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁，线程不会主动释放偏向锁。偏向锁的撤销，需要等待全局安全点（在这个时间点上没有字节码正在执行），它会首先暂停拥有偏向锁的线程，判断锁对象是否处于被锁定状态。撤销偏向锁后恢复到无锁（标志位为“01”）或轻量级锁（标志位为“00”）的状态。

偏向锁在JDK 6及以后的JVM里是默认启用的。可以通过JVM参数关闭偏向锁：-XX:-UseBiasedLocking=false，关闭之后程序默认会进入轻量级锁状态。

在轻量级的锁中，我们可以发现，如果同一个线程对同一个对象进行重入锁时，也需要执行CAS操作，这是有点耗时滴，那么java6开始引入了偏向锁的东东，只有第一次使用CAS时将对象的Mark Word头设置为入锁线程ID，**之后这个入锁线程再进行重入锁时，发现线程ID是自己的，那么就不用再进行CAS了**

![1583760728806](https://gitee.com/gu_chun_bo/picture/raw/master/image/20200309213209-28609.png)

#### 4.4.1 偏向状态

![1583762169169](https://gitee.com/gu_chun_bo/picture/raw/master/image/20200309215610-51761.png)

一个对象的创建过程

1. 如果开启了偏向锁（默认是开启的），那么对象刚创建之后，Mark Word 最后三位的值101，并且这是它的Thread，epoch，age都是0，在加锁的时候进行设置这些的值.

2. 偏向锁默认是延迟的，不会在程序启动的时候立刻生效，如果想避免延迟，可以添加虚拟机参数来禁用延迟：-`XX:BiasedLockingStartupDelay=0`来禁用延迟

3. 注意：处于偏向锁的对象解锁后，线程 id 仍存储于对象头中

4. 实验Test18.java，加上虚拟机参数-XX:BiasedLockingStartupDelay=0进行测试

   1. ```java
      public static void main(String[] args) throws InterruptedException {
              Test1 t = new Test1();
              test.parseObjectHeader(getObjectHeader(t))；
              synchronized (t){
                  test.parseObjectHeader(getObjectHeader(t));
              }
              test.parseObjectHeader(getObjectHeader(t));
          }
      ```

   2. 输出结果如下，三次输出的状态码都为101

      ```properties
      biasedLockFlag (1bit): 1
      	LockFlag (2bit): 01
      biasedLockFlag (1bit): 1
      	LockFlag (2bit): 01
      biasedLockFlag (1bit): 1
      	LockFlag (2bit): 01
      ```

      

测试禁用：如果没有开启偏向锁，那么对象创建后最后三位的值为001，这时候它的hashcode，age都为0，hashcode是第一次用到`hashcode`时才赋值的。在上面测试代码运行时在添加 VM 参数`-XX:-UseBiasedLocking`禁用偏向锁（禁用偏向锁则优先使用轻量级锁），退出`synchronized`状态变回001

1. 测试代码Test18.java 虚拟机参数`-XX:-UseBiasedLocking`

2. 输出结果如下，最开始状态为001，然后加轻量级锁变成00，最后恢复成001

   ```properties
   biasedLockFlag (1bit): 0
   	LockFlag (2bit): 01
   LockFlag (2bit): 00
   biasedLockFlag (1bit): 0
   	LockFlag (2bit): 01
   ```

#### 4.4.2 撤销偏向锁-hashcode方法

测试 `hashCode`：当调用对象的hashcode方法的时候就会撤销这个对象的偏向锁，因为使用偏向锁时没有位置存`hashcode`的值了

1. 测试代码如下，使用虚拟机参数`-XX:BiasedLockingStartupDelay=0`  ，确保我们的程序最开始使用了偏向锁！但是结果显示程序还是使用了轻量级锁。  Test20.java

   ```java
       public static void main(String[] args) throws InterruptedException {
           Test1 t = new Test1();
           t.hashCode();
           test.parseObjectHeader(getObjectHeader(t));
   
           synchronized (t){
               test.parseObjectHeader(getObjectHeader(t));
           }
           test.parseObjectHeader(getObjectHeader(t));
       }
   ```

2. 输出结果

   ```properties
   biasedLockFlag (1bit): 0
   	LockFlag (2bit): 01
   LockFlag (2bit): 00
   biasedLockFlag (1bit): 0
   	LockFlag (2bit): 01
   ```


#### 4.4.3 撤销偏向锁-其它线程使用对象

这里我们演示的是偏向锁撤销变成轻量级锁的过程，那么就得满足轻量级锁的使用条件，就是没有线程对同一个对象进行锁竞争，我们使用`wait` 和 `notify` 来辅助实现

1. 代码 Test19.java，虚拟机参数`-XX:BiasedLockingStartupDelay=0`确保我们的程序最开始使用了偏向锁！

2. 输出结果，最开始使用的是偏向锁，但是第二个线程尝试获取对象锁时，发现本来对象偏向的是线程一，那么偏向锁就会失效，加的就是轻量级锁

   ```properties
   biasedLockFlag (1bit): 1
   	LockFlag (2bit): 01
   biasedLockFlag (1bit): 1
   	LockFlag (2bit): 01
   biasedLockFlag (1bit): 1
   	LockFlag (2bit): 01
   biasedLockFlag (1bit): 1
   	LockFlag (2bit): 01
   LockFlag (2bit): 00
   biasedLockFlag (1bit): 0
   	LockFlag (2bit): 01
   ```

   

#### 4.4.4 撤销 - 调用 wait/notify

会使对象的锁变成重量级锁，因为wait/notify方法之后重量级锁才支持。

#### 4.4.5 批量重偏向

如果对象被多个线程访问，但是没有竞争，这时候偏向了线程一的对象又有机会重新偏向线程二，即可以不用升级为轻量级锁，可这和我们之前做的实验矛盾了呀，其实要实现重新偏向是要有条件的：就是超过20对象对同一个线程如线程一撤销偏向时，那么第20个及以后的对象才可以将撤销对线程一的偏向这个动作变为将第20个及以后的对象偏向线程二。Test21.java

### 4.5 轻量级锁

轻量级锁的使用场景：多个线程使用锁的时间是错开的，相当于没有竞争。轻量级锁对使用者是透明的，即语法仍然是`synchronized`。

例如有两个方法同步块，利用同一个对象加锁：

```java
static final Object obj = new Object();
public static void method1() {
     synchronized( obj ) {
         // 同步块 A
         method2();
     }
}
public static void method2() {
     synchronized( obj ) {
         // 同步块 B
     }
}
```

1. 每次指向到synchronized代码块时，都会创建锁记录（Lock Record）对象，每个线程都会包括一个锁记录的结构，锁记录内部可以储存对象的Mark Word和对象引用reference。![1583755737580](https://gitee.com/gu_chun_bo/picture/raw/master/image/20200309200902-382362.png)
2. 让锁记录中的Object reference指向对象，并且尝试用cas(compare and sweep)替换Object对象的Mark Word ，将Mark Word 的值存入锁记录中
   1. ![1583755888236](https://gitee.com/gu_chun_bo/picture/raw/master/image/20200309201132-961387.png)
3. 如果cas替换成功，那么对象的对象头储存的就是锁记录的地址和状态01，如下所示
   1. ![1583755964276](https://gitee.com/gu_chun_bo/picture/raw/master/image/20200309201247-989088.png)
4. 如果cas失败，有两种情况
   1. 如果是其它线程已经持有了该Object的轻量级锁，那么表示有竞争，将进入锁膨胀阶段
   2. 如果是自己的线程已经执行了synchronized进行加锁，那么那么再添加一条 Lock Record 作为重入的计数
      1. ![1583756190177](https://gitee.com/gu_chun_bo/picture/raw/master/image/20200309201634-451646.png)
5. 当线程退出synchronized代码块的时候，**如果获取的是取值为 null 的锁记录 **，表示有重入，这时重置锁记录，表示重入计数减一
   1. ![1583756357835](https://gitee.com/gu_chun_bo/picture/raw/master/image/20200309201919-357425.png)
6. 当线程退出synchronized代码块的时候，如果获取的锁记录取值不为 null，那么使用cas将Mark Word的值恢复给对象
   1. 成功则解锁成功
   2. 失败，则说明轻量级锁进行了锁膨胀或已经升级为重量级锁，进入重量级锁解锁流程

### 4.6 重量级锁

### 4.7 锁膨胀/锁升级

![](https://img-blog.csdnimg.cn/20201028011119984.png)

## 5 wait和notify

建议先看看wait和notify方法的javadoc文档

### 5.1 同步模式之保护性暂停

即 Guarded Suspension，用在一个线程等待另一个线程的执行结果，要点：

1. 有一个结果需要从一个线程传递到另一个线程，让他们关联同一个 GuardedObject
2. 如果有结果不断从一个线程到另一个线程那么可以使用消息队列（见生产者/消费者）
3. JDK 中，join 的实现、Future 的实现，采用的就是此模式
4. 因为要等待另一方的结果，因此归类到同步模式

代码：Test22.java    Test23.java这是带超时时间的

![](https://img-blog.csdnimg.cn/20201026221941471.png)

关于超时的增强，在join(long millis) 的源码中得到了体现：

```java
    public final synchronized void join(long millis)
    throws InterruptedException {
        long base = System.currentTimeMillis();
        long now = 0;

        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }
		
        if (millis == 0) {
            while (isAlive()) {
                wait(0);
            }
        } else {
        // join一个指定的时间
            while (isAlive()) {
                long delay = millis - now;
                if (delay <= 0) {
                    break;
                }
                wait(delay);
                now = System.currentTimeMillis() - base;
            }
        }
    }
```



多任务版 GuardedObject图中 Futures 就好比居民楼一层的信箱（每个信箱有房间编号），左侧的 t0，t2，t4 就好比等待邮件的居民，右侧的 t1，t3，t5 就好比邮递员如果需要在多个类之间使用 GuardedObject 对象，作为参数传递不是很方便，因此设计一个用来解耦的中间类，这样不仅能够解耦【结果等待者】和【结果生产者】，还能够同时支持多个任务的管理。和生产者消费者模式的区别就是：这个生产者和消费者之间是一一对应的关系，但是生产者消费者模式并不是。rpc框架的调用中就使用到了这种模式。  Test24.java

![1594518049426](D:/workplace/IdeaProjects/CS-Notes-Kz/01-basic-learning/01-language/01-java-basic/03-concurrent&multithreaded/assets/1594518049426.png)

### 5.2 异步模式之生产者/消费者

要点

1. 与前面的保护性暂停中的 GuardObject 不同，不需要产生结果和消费结果的线程一一对应
2. 消费队列可以用来平衡生产和消费的线程资源
3. 生产者仅负责产生结果数据，不关心数据该如何处理，而消费者专心处理结果数据
4. 消息队列是有容量限制的，满时不会再加入数据，空时不会再消耗数据
5. JDK 中各种[阻塞队列](https://blog.csdn.net/yanpenglei/article/details/79556591)，采用的就是这种模式

“异步”的意思就是生产者产生消息之后消息没有被立刻消费，而“同步模式”中，消息在产生之后被立刻消费了。

![1594524622020](D:/workplace/IdeaProjects/CS-Notes-Kz/01-basic-learning/01-language/01-java-basic/03-concurrent&multithreaded/assets/1594524622020.png)

我们写一个线程间通信的消息队列，要注意区别，像rabbit mq等消息框架是进程间通信的。

### 5.3 同步模式之顺序控制

1. 固定运行顺序，比如，必须先 2 后 1 打印
   1.  wait notify 版  Test35.java
   2.  Park Unpark 版  Test36.java
2. 交替输出，线程 1 输出 a 5 次，线程 2 输出 b 5 次，线程 3 输出 c 5 次。现在要求输出 abcabcabcabcabc 怎么实现
   1. wait notify 版   Test37.java
   2. Lock 条件变量版 Test38.java
   3. Park Unpark 版 Test39.java

## 6 活跃性

活跃性相关的一系列问题都可以用ReentrantLock进行解决。

### 6.1 死锁

有这样的情况：一个线程需要同时获取多把锁，这时就容易发生死锁t1 线程获得A对象锁，接下来想获取B对象的锁；t2 线程获得B对象锁，接下来想获取A对象的锁例。Test28.java

### 6.2 检测死锁

检测死锁可以使用 jconsole工具；或者使用 jps 定位进程 id，再用 jstack 定位死锁：Test28.java

下面使用jstack工具进行演示

```java
D:\我的项目\JavaLearing\java并发编程\jdk8>jps
1156 RemoteMavenServer36
20452 Test25
9156 Launcher
23544 Jps
23848
22748 Test28

D:\我的项目\JavaLearing\java并发编程\jdk8>jstack 22748
2020-07-12 18:54:44
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.211-b12 mixed mode):

"DestroyJavaVM" #14 prio=5 os_prio=0 tid=0x0000000002a03800 nid=0x5944 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

//................省略了大部分内容.............//
Found one Java-level deadlock:
=============================
"线程二":
  waiting to lock monitor 0x0000000002afc0e8 (object 0x00000000db9f76d0, a java.lang.Object),
  which is held by "线程1"
"线程1":
  waiting to lock monitor 0x0000000002afe1e8 (object 0x00000000db9f76e0, a java.lang.Object),
  which is held by "线程二"

Java stack information for the threads listed above:
===================================================
"线程二":
        at com.concurrent.test.Test28.lambda$main$1(Test28.java:39)
        - waiting to lock <0x00000000db9f76d0> (a java.lang.Object)
        - locked <0x00000000db9f76e0> (a java.lang.Object)
        at com.concurrent.test.Test28$$Lambda$2/326549596.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)
"线程1":
        at com.concurrent.test.Test28.lambda$main$0(Test28.java:23)
        - waiting to lock <0x00000000db9f76e0> (a java.lang.Object)
        - locked <0x00000000db9f76d0> (a java.lang.Object)
        at com.concurrent.test.Test28$$Lambda$1/1343441044.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)
```



### 6.3 哲学家就餐问题

![1594553609905](D:/workplace/IdeaProjects/CS-Notes-Kz/01-basic-learning/01-language/01-java-basic/03-concurrent&multithreaded/assets/1594553609905.png)

有五位哲学家，围坐在圆桌旁。
他们只做两件事，思考和吃饭，思考一会吃口饭，吃完饭后接着思考。
吃饭时要用两根筷子吃，桌上共有 5 根筷子，每位哲学家左右手边各有一根筷子。
如果筷子被身边的人拿着，自己就得等待  Test29.java

当每个哲学家即线程持有一根筷子时，他们都在等待另一个线程释放锁，因此造成了死锁。这种线程没有按预期结束，执行不下去的情况，归类为【活跃性】问题，除了死锁以外，还有活锁和饥饿者两种情
况

### 6.4 饥饿

很多教程中把饥饿定义为，一个线程由于优先级太低，始终得不到 CPU 调度执行，也不能够结束，饥饿的情况不易演示，讲读写锁时会涉及饥饿问题下面我讲一下一个线程饥饿的例子，先来看看使用顺序加锁的方式解决之前的死锁问题，就是两个线程对两个不同的对象加锁的时候都使用相同的顺序进行加锁。 但是会产生饥饿问题Test29

![1594558469826](https://gitee.com/gu_chun_bo/picture/raw/master/image/20200712205431-675389.png)

顺序加锁的解决方案

![1594558499871](D:/workplace/IdeaProjects/CS-Notes-Kz/01-basic-learning/01-language/01-java-basic/03-concurrent&multithreaded/assets/1594558499871.png)

**本章小结**

本章我们需要重点掌握的是

1. 分析多线程访问共享资源时，哪些代码片段属于临界区
2. 使用 synchronized 互斥解决临界区的线程安全问题
   1. 掌握 synchronized 锁对象语法
   2. 掌握 synchronzied 加载成员方法和静态方法语法
   3. 掌握 wait/notify 同步方法
3. 使用 lock 互斥解决临界区的线程安全问题
   掌握 lock 的使用细节：可打断、锁超时、公平锁、条件变量
4. 学会分析变量的线程安全性、掌握常见线程安全类的使用
5. 了解线程活跃性问题：死锁、活锁、饥饿
6. 应用方面
   1. **互斥：使用 synchronized 或 Lock 达到共享资源互斥效果，实现原子性效果，保证线程安全。**
   2. **同步：使用 wait/notify 或 Lock 的条件变量来达到线程间通信效果。**
7. 原理方面
   1. monitor、synchronized 、wait/notify 原理
   2. synchronized 进阶原理
   3. park & unpark 原理
8. 模式方面
   1. 同步模式之保护性暂停
   2. 异步模式之生产者消费者
   3. 同步模式之顺序控制

## 参考资料

[多线程基础](https://blog.csdn.net/KAIZ_LEARN/article/details/108890366)

[JAVA并发编程的艺术](https://weread.qq.com/web/reader/247324e05a66a124750d9e9k8f132430178f14e45fce0f7)

[让你彻底理解Synchronized](https://www.jianshu.com/p/d53bf830fa09)

[Synchronized如何实现同步？锁优化](https://juejin.im/post/6844904065462173709)

[Java 中的 Monitor 机制](https://segmentfault.com/a/1190000016417017)

[java的monitor对象](https://www.cnblogs.com/bravecode/p/12620719.html)

[java object header](https://stackoverflow.com/questions/26357186/what-is-in-java-object-header)

[浅谈偏向锁、轻量级锁、重量级锁](https://juejin.im/post/6844903550586191885)

[深入分析Synchronized原理](https://juejin.im/post/6844903640197513230)

[Java CAS 原理剖析](https://juejin.im/post/6844903558937051144)

[不可不说的Java“锁”事](https://tech.meituan.com/2018/11/15/java-lock.html)



以32 位虚拟机为例，普通对象的对象头：

![](https://img-blog.csdnimg.cn/20201025233948575.png#pic_center)

如果是数组对象，还需记录数组长度：

![](https://img-blog.csdnimg.cn/20201026011336872.png#pic_center)

其中Mark Word 结构为：

![](https://img-blog.csdnimg.cn/20201026011726240.png#t_70#pic_center)

64位虚拟机对象Mark Word结构：

![](https://img-blog.csdnimg.cn/20201026011946370.png#pic_center)





