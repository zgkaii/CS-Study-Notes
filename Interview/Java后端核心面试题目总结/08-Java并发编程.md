<!-- MarkdownTOC -->
- [并发编程基础](#并发编程基础)
  - [为什么有多线程](#为什么有多线程)
  - [守护线程是什么](#守护线程是什么)
  - [线程的生命周期](#线程的生命周期)
  - [Runnable和Callable区别](#runnable和callable区别)
  - [run()与start()方法区别](#run与start方法区别)
  - [Thread.interrupt() 方法的工作原理](#threadinterrupt-方法的工作原理)
  - [什么是线程上下文切换](#什么是线程上下文切换)
  - [可见性、原子性、有序性问题产生原因](#可见性原子性有序性问题产生原因)
- [线程安全相关](#线程安全相关)
  - [volatile原理](#volatile原理)
  - [happens-before原则是什么](#happens-before原则是什么)
  - [MESI协议是什么](#mesi协议是什么)
  - [synchronized原理](#synchronized原理)
  - [synchronized与Lock区别](#synchronized与lock区别)
  - [Java中如何进行锁优化](#java中如何进行锁优化)
  - [Java 锁升级过程](#java-锁升级过程)
  - [死锁、活锁、饥饿区别](#死锁活锁饥饿区别)
  - [乐观锁与悲观锁](#乐观锁与悲观锁)
  - [公平锁与非公平锁](#公平锁与非公平锁)
  - [可重入锁与非可重入锁](#可重入锁与非可重入锁)
  - [独享锁与共享锁](#独享锁与共享锁)
- [原子类](#原子类)
  - [累加器](#累加器)
  - [CAS原理](#cas原理)
  - [CAS实现原子操作的三大问题](#cas实现原子操作的三大问题)
- [并发容器类](#并发容器类)
  - [ConcurrentHashMap](#concurrenthashmap)
  - [ConcurrentSkipListMap](#concurrentskiplistmap)
  - [CopyOnWriteArrayList](#copyonwritearraylist)
  - [非阻塞队列](#非阻塞队列)
    - [ConcurrentLinkedQueue](#concurrentlinkedqueue)
  - [阻塞队列](#阻塞队列)
    - [ArrayBlockingQueue](#arrayblockingqueue)
    - [LinkedBlockingQueue](#linkedblockingqueue)
    - [PriorityBlockingQueue](#priorityblockingqueue)
    - [DelayQueue](#delayqueue)
    - [SynchronousQueue](#synchronousqueue)
- [常见并发工具类](#常见并发工具类)
    - [Semaphore 信号量](#semaphore-信号量)
    - [CountDownLatch 倒计时器](#countdownlatch-倒计时器)
    - [CyclicBarrier 循环栅栏](#cyclicbarrier-循环栅栏)
    - [CyclicBarrier和CountDownLatch的区别](#cyclicbarrier和countdownlatch的区别)
- [AQS原理](#aqs原理)
- [ThreadLocal应用及原理](#threadlocal应用及原理)
- [线程池](#线程池)
  - [使用线程池使用好处](#使用线程池使用好处)
  - [线程池的处理流程](#线程池的处理流程)
    - [submit()与 execute()方法区别](#submit与-execute方法区别)
  - [线程池状态转换](#线程池状态转换)
  - [线程池参数](#线程池参数)
    - [线程池拒绝策略](#线程池拒绝策略)
    - [缓冲队列](#缓冲队列)
  - [线程池为什么不直接Executors创建线程池](#线程池为什么不直接executors创建线程池)
  - [线程池线程回收策略](#线程池线程回收策略)
  - [线程池数量计算: CPU密集型 VS I/O密集型](#线程池数量计算-cpu密集型-vs-io密集型)

<!-- /MarkdownTOC -->

# 并发编程基础

## 为什么有多线程

随着摩尔定律的失效与多核+分布式时代的来临，操作系统可以更多的并行资源，这时需要新的方式来提升系统性能。

从度量的角度，主要是**降低延迟，提高吞吐量**。因此，我们主要有两个方向，一是**优化算法，另一个是将硬件的性能发挥到极致**。那计算机主要有哪些硬件呢？主要是两类：一个是 **I/O**，一个是 **CPU**。简言之，**在并发编程领域，提升性能本质上就是提升硬件的利用率，再具体点来说，就是提升 I/O 的利用率和 CPU 的利用率**。

操作系统虽然没有办法完美解决，但是却给我们提供了方案，那就是：多线程。

## 守护线程是什么

Java 中的线程分为两种：守护线程（Daemon）和用户线程（User）。任何线程都可以设置为守护线程和用户线程，通过方法`Thread.setDaemon(true)`则把该线程设置为守护线程，反之则为用户线程。`Thread.setDaemon()`必须在 `Thread.start()`之前调用，否则运行时会抛出异常。

两者唯一的区别是判断虚拟机(JVM)何时离开，Daemon 是为其他线程提供服务，如果全部的 User Thread已经关闭，Daemon 没有可服务的线程，JVM会直接关闭。

可见，我们在日常编码中要注意，守护线程不能持有任何需要关闭的资源，例如打开文件等。因为虚拟机退出时，守护线程没有任何机会来关闭文件，这容易导致数据丢失。

## 线程的生命周期

当线程被创建并启动以后，它既不是一启动就进入了执行状态，也不是一直处于执行状态。在线程的生命周期中，有几种状态呢？在API中`java.lang.Thread.State`枚举出了六种线程状态：

| 线程状态      | 说明                                                                                       |
| :------------ | ------------------------------------------------------------------------------------------ |
| NEW           | 初始状态，线程刚被创建，但是并未启动（还未调用start方法）。                                |
| RUNNABLE      | 运行状态，JAVA线程将操作系统中的就绪（READY）和运行（RUNNING）两种状态笼统地称为“运行中”。 |
| BLOCKED       | 阻塞状态，表示线程阻塞于锁。                                                               |
| WAITING       | 等待状态，表示该线程无限期等待另一个线程执行一个特别的动作。                               |
| TIMED_WAITING | 超时等待状态，不同于WAITING的是，它可以在指定时间自动返回。                                |
| TERMINATED    | 终止状态，表示当前状态已经执行完                                                           |

图解如下：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200929220833538.png" width="800px"/>
</div>
其中涉及主要方法：

1. **Thread.sleep(long millis)**，一定是当前线程调用此方法，当前线程进入 TIMED_WAITING 状态，但**不释放对象锁**，millis 后线程自动苏醒进入就绪状态。作用：给其它线程执行机会的最佳方式。
2. **Thread.yield()**，一定是当前线程调用此方法，当前线程放弃获取的 CPU 时间片，但不释放锁资源，由运行状态变为就绪状态，让 OS 再次选择线程。作用：让相同优先级的线程轮流执行，但并不保证一定会轮流执行。实际中无法保证yield() 达到让步目的，因为让步的线程还有可能被线程调度程序再次选中。Thread.yield() 不会导致阻塞。该方法与sleep() 类似，只是不能由用户指定暂停多长时间。
3. **t.join()/t.join(long millis)**，当前线程里调用其它线程 t 的 join 方法，当前线程进入WAITING/TIMED_WAITING 状态，**当前线程不会释放已经持有的对象锁**，因为**内部调用了 t.wait，所以会释放t这个对象上的同步锁**。线程 t 执行完毕或者 millis 时间到，当前线程进入就绪状态。其中，wait 操作对应的 notify 是由 jvm 底层的线程执行结束前触发的。
4. **obj.wait()**，当前线程调用对象的 wait() 方法，**当前线程释放 obj 对象锁**，进入等待队列。依靠 notify()/notifyAll()唤醒或者 wait(long timeout) timeout 时间到自动唤醒。唤醒会，线程恢复到 wait 时的状态。
5. **obj.notify()**唤醒在此对象监视器上等待的单个线程，选择是任意性的。notifyAll() 唤醒在此对象监视器上等待的所有线程。

## Runnable和Callable区别

创建线程有三种方式：

1. 继承Thread类创建线程；
2. 通过Runnable接口创建线程；
3. 通过Callable和Future接口创建线程；

Runnable和Callable区别在于：

* Runnable是自从Java 1.1就有了，而Callable是Java 1.5之后才加上去的；

* Runnable 接口中的 run() 方法的返回值是 void，它做的事情只是纯粹地去执行 run() 方法中的代码，不能抛出异常； 

* Callable 接口中的 call() 方法是有返回值的，可以抛出异常，是⼀个泛型，和 Future、FutureTask 配合可以用来获取异步执行的结果；
* 加入线程池运行，Runnable使用`ExecutorService`的execute方法，Callable使用submit方法。

## run()与start()方法区别

start()用来启动一个线程，当调用start()方法时，系统才会开启一个线程（`start()`调用`native start0()`方法）来启动的线程处于就绪状态（可运行状态），此时并没有运行，一旦得到CPU时间片，就自动开始执行run()方法。此时不需要等待run()方法执行完也可以继续执行下面的代码，所以也由此看出run()方法并没有实现多线程。 

run()方法是在本线程里的，只是线程里的一个函数，而不是多线程的。如果直接调用run()，其实就相当于是调用了一个普通函数而已，直接待用run()方法必须等待run()方法执行完毕才能执行下面的代码，所以执行路径还是只有一条，根本就没有多线程的特征，所以在多线程执行时要使用start()方法而不是run()方法。

**一个线程两次调用start()方法会出现什么情况？**

**解答**：Java线程是不允许一个线程两次调用`start()`方法的，第二次调用必然会抛出`IllegalThreadStateException`，这是一种运行时异常，多次调用start会被认为是编译错误。

在第二次调用 start() 方法的时候，线程可能处于终止或者其他（非 NEW）状态，但是不论如何，都是不可以再次启动的。

## wait方法和sleep方法的区别

主要有以下几个不同：

* 所属的类不同：sleep()属于Thread类，wait()属于Object类；
* 时间不同：sleep()必须指定时间，wait()可以指定时间也可以不指定时间；
* 释放锁不同：sleep()释放CPU执行权不释放同步锁；wait()即释放CPU执行权也释放同步锁；
* 使用的地方不同：sleep()可以在任意地方使用；wait()只能在同步代码方法或者同步代码块中使用；
* 捕获异常不同：sleep()必须捕获异常；wait()是Object方法，调用也许捕获/抛出异常。

## Thread.interrupt() 方法的工作原理

线程的`Thread.interrupt()`方法用于中断线程，他会设置该线程的中断状态位为true。中断后线程是死亡、还是等待新的任务或是继续运行至下一步，就取决于这个程序本身。线程会不时地检测这个中断标示位，以判断线程是否应该被中断（中断标示值是否为true）。它并不像stop方法那样会中断一个正在运行的线程。

==注意==，**Java的中断是一种协作机制**。调用线程对象的interrupt方法并不一定就中断了正在运行的线程，它只是要求线程自己在合适的时机中断自己。每个线程都有一个boolean的中断状态（这个状态不在Thread的属性上），interrupt方法仅仅只是将该状态置为true。比如对正常运行的线程调用interrupt()并不能终止它，**只是改变了interrupt标示符**。

一般说来，如果一个方法声明抛出InterruptedException，表示该方法是可中断的，比如wait/sleep/join。也就是说可中断方法会对interrupt调用做出响应（例如sleep响应interrupt的操作包括清除中断状态，抛出InterruptedException），异常都是由可中断方法自己抛出来的，并不是直接由interrupt方法直接引起的。

**正是如此，Object.wait()/Thread.sleep()/Thread.join()方法，才会不断的轮询监听 interrupted 标志位，发现其为true后，会停止阻塞并抛出 InterruptedException异常**。

也就是说，`Thread.interrupt()`方法不会真正地中断一个正在运行的线程。它主要用于设置线程的中断标示位，在线程受到阻塞的地方（如Thread.sleep()、Thread.join()、Object.wait()检查到线程为“中断状态”后）抛出一个InterruptedException异常，并且“中断状态”也将被清除，这样线程就得以退出阻塞的状态。如果线程没有被阻塞，这时调用 interrupt() 将不起作用，直到执行到 wait/sleep/join 时，才马上会抛出InterruptedException。

## 什么是线程上下文切换

线程上下文切换的过程，**就是一个线程被暂停剥夺使用权，另一个线程被选中开始或者继续运行的过程**。

由于计算机大多是**抢占式操作系统，**线程上下文切换的原因大概有以下几种：

- 当前执行任务的时间片用完之后，系统CPU正常调度下一个任务。
- 当前执行任务碰到IO阻塞，调度器将此任务挂起，继续下一任务。
- 多个任务抢占锁资源，当前任务没有抢到锁资源，被调度器挂起，继续下一任务。
- 用户代码挂起当前任务，让出CPU时间。
- 硬件中断。

Java程序中，线程上下文切换的主要原因可分为：

- 程序本身触发的**自发性上下文切换**：sleep、wait、yield、join、park、synchronized、lock等方法
- 系统或虚拟机触发的**非自发性上下文切换**：线程被分配的时间片用完、JVM垃圾回收（STW、线程暂停）、执行优先级高的线程

减少上下文切换：

减少上下文切换的方法有**无锁并发编程、CAS算法、使用最少线程和使用协程**。

- 无锁并发编程。多线程竞争时，会引起上下文切换，所以多线程处理数据时，可以用一些办法来避免使用锁，如将数据的ID按照Hash取模分段，不同的线程处理不同段的数据。
- CAS算法。Java的Atomic包使用CAS算法来更新数据，而不需要加锁。
- 使用最少线程。避免创建不需要的线程，比如任务很少，但是创建了很多线程来处理，这样会造成大量线程都处于等待状态。
- 协程。在单线程里实现多任务的调度，并在单线程里维持多个任务间的切换。

## 可见性、原子性、有序性问题产生原因

线程上下文切换是影响Java并发编程性能的重要原因，却不是根本原因。并发编程的主要瓶颈还是体现在**CPU、内存、I/O** 设备三者速度差异的核心矛盾上。CPU 和内存的速度差异可以形象地描述为：CPU 是天上一天，内存是地上一年，那么I/O设备就是十年了。根据木桶理论，程序整体的性能取决于最慢的操作——读写 I/O 设备，也就是说单方面提高 CPU 性能是无效的。

为了合理利用 CPU 的高性能，平衡这三者的速度差异，计算机体系机构、操作系统、编译程序都做出了贡献，主要体现为：

- CPU增加缓存，以均衡与内存的差异，但是会引发可见性问题；
- 编译程序优化指令执行次序，使得缓存能够得到更加合理地利用，但是会引发有序性问题。
- 操作系统增加了进程、线程，以分时复用CPU，进而均衡CPU和I/O设备的速度差异，但是会引发原子性问题；

其实缓存、线程、编译优化的目的和我们写并发程序的目的是相同的，都是提高程序性能。但是技术在解决一个问题的同时，必然会带来另外一个问题，所以在采用一项技术的同时，一定要清楚它带来的问题是什么，以及如何规避。

# 线程安全相关

## volatile原理

这要从**Java内存模型**（JMM）说起， JMM定义了线程和主内存的抽象关系：线程之间的共享遍历存储在主内存（Main Memory）中，每个线程都有一个私有的本地内存（Local Memory），本地内存中存储了该线程以读/写共享变量的副本。在并发编程场景中，**多线程读写共享内存中的全局变量及静态变量容易引发竞态条件**，这会导致可见性问题。

除此之外，在执行程序时，为了提高性能，**编译器和处理器常常会对指令做重排序**，这会导致有序性问题。

上述的可见性与有序性的问题，都可以使用volatile关键字解决。

**当一个共享变量被 volatile 修饰时，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取新值，**这保证了可见性。

此外，**happens-before volatile变量规则**描述到：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。也就是说，**volatile禁止进行指令重排序**，这就保证了有序性。

如何实现的呢？其实，volatile 的底层实现原理是应用到**内存屏障**，内存屏障会提供3个功能：

- 它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成。
- 它会强制将对缓存的修改操作立即写入主存。
- 如果是写操作，它会导致其他CPU中对应的缓存行无效。

下面是完成happens-before规则所需要的内存屏障：

| 是否能重排序   | 第二个操作 | 第二个操作 | 第二个操作 | 第二个操作 |
| -------------- | ---------- | ---------- | ---------- | ---------- |
| **第一个操作** | 普通读     | 普通写     | Volatile读 | Volatile写 |
| 普通读         |            |            |            | LoadStore  |
| 普通写         |            |            |            | StoreStore |
| Volatile读     | LoadLoad   | LoadStore  | LoadLoad   | LoadStore  |
| Volatile写     |            |            | StoreLoad  | StoreStore |

> volatile常见应用——双重检查锁

## happens-before原则是什么

JMM可以通过happens-before（先行发生）关系向程序员提供跨线程的内存可见性保证，同时也是对编译器和处理器重排序的约束原则。

1. 程序次序规则：一个线程内，按照代码顺序，书写在前面的操作，happens-before 于书写在后面的操作。
2. 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁。
3. volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。
4. 传递规则：如果A happens-before B，且B happens-before C，那么A happens-before C。
5. 线程启动规则：Thread 对象的 start() 方法先行发生于此线程的每个一个动作
6. .线程中断规则：对线程 interrupt() 方法的调用先行发生于被中断线程的代码检测到中断事件的发生
7. 线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过 Thread.join() 方法结束、Thread.isAlive() 的返回值手段检测到线程已经终止执行
8. 对象终结规则：一个对象的初始化完成，happens-before 它的 finalize() 方法的开始。

## MESI协议是什么

MESI 协议，是一种叫作**写失效（Write Invalidate）**的协议。在写失效协议里，只有一个 CPU 核心负责写入数据，其他的核心，只是同步读取到这个写入。在这个 CPU 核心写入 Cache 之后，它会去广播一个“失效”请求告诉所有其他的 CPU 核心。其他的 CPU 核心，只是去判断自己是否也有一个“失效”版本的 Cache Block，然后把这个也标记成失效的就好了。

MESI 协议的由来呢，来自于我们对 Cache Line 的四个不同的标记，分别是：

- M：代表已修改（Modified）
- E：代表独占（Exclusive）
- S：代表共享（Shared）
- I：代表已失效（Invalidated）

所谓的“已修改”，就是的“脏”的 Cache Block，Cache Block 里面的内容我们已经更新过了，但是还没有写回到主内存里面；而所谓的“已失效“，自然是这个 Cache Block 里面的数据已经失效了，我们不可以相信这个 Cache Block 里面的数据。

独占和共享状态，就好像我们在多线程应用开发里面的读写锁机制，确保了我们的缓存一致性。独占状态是可以变为读共享的。之所以区分为两个状态，是因为独占状态下写无须通知其他CPU失效它对应的缓存行，减少不必要的操作。 共享状态写会先失效其他CPU的缓存。写并不一定是第一个独占的CPU可以写，可以由后来共享的CPU写，只要写入后，把同一个变量的其他CPU缓存行失效即可，类比volitle理解。

## synchronized原理

synchronized又称为重量级锁，是实现同步的基础方式之一。它具体使用场景如下：

1. 修饰普通同步方法，锁是当前实例对象；
2. 修饰静态同步方法，锁是当前类的Class对象；
3. 修饰同步方法块，锁是synchonized括号中配置的对象。

可以看到，synchronized都是对对象的加锁。

HotSpot虚拟机中，对象在内存中存储的布局可以分为三块区域：对象头、实例数据和对齐填充。对象头区又主要分为两部分，分别是运行时元数据（Mark Word）和 类型指针。**Mark Word用于存储对象自身的运行时数据， 如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等等，它是实现轻量级锁和偏向锁的关键。**

**synchronized锁对象就存储在MarkWord中**，下面是MarkWord的布局：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/2021060615134026.png" width="800px"/>
</div>

如果你看过通过synchronized加锁代码的字节码文件，你会发现：

- 当加锁在同步代码块上时，JVM是通过*`monitorenter`* 和 *`monitorexit`* 两个指令进行同步控制的。
- 当加锁在同步方法时，JVM是通过将会检查方法的 *`ACC_SYNCHRONIZED`* 访问标志来判断是否获取monitor。

也就是说，**synchronized就是通过monitor机制来实现同步**。

那什么是monitor呢？

- monitor，常被翻译为“监视器”或者“管程”。它是一种同步原语，相较于操作系统同步原语（semaphore 信号量 和 mutex 互斥量）更加的高层次。

Java是如何实现monitor呢？

- 在Java虚拟机（HotSpot）中，**Monitor是由ObjectMonitor实现的。**每个 Java 对象在 JVM 的对等对象的头中Mark Word区域保存锁状态，指向*ObjectMonitor*。

*ObjectMonitor*中有两个队列，_WaitSet 和 _EntryList，用来保存ObjectWaiter对象列表（ 每个等待锁的线程都会被封装成ObjectWaiter对象 ），_owner指向持有ObjectMonitor对象的线程，处理详细流程如下：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20201026000053726.png" width="450px"/>
</div>

1. 加锁时，即遇到synchronized关键字时，线程会先进入monitor的_EntryList队列阻塞等待。
2. 如果monitor的_owner为空，则从队列中移出并赋值与_owner。
3. 如果在程序里调用了wait()方法，则该线程进入_WaitSet队列。我们都知道wait方法会释放monitor锁，即将_owner赋值为null并进入_WaitSet队列阻塞等待。这时其他在_EntryList中的线程就可以获取锁了。
4. 当程序里其他线程调用了notify/notifyAll方法时，就会唤醒_WaitSet中的某个线程，这个线程就会再次尝试获取monitor锁。如果成功，则就会成为monitor的owner。
5. 当程序里遇到synchronized关键字的作用范围结束时，就会将monitor的owner设为null，退出。

以上就是synchronized原理。

## synchronized与Lock区别

- synchronized属于JVM层面，底层通过 `monitorenter` 和 `monitorexit` 两个指令实现，Lock是API层面的东西，JUC提供的具体类。
- synchronized不需要用户手动释放锁，当synchronized代码执行完毕之后会自动让线程释放持有的锁，Lock需要一般使用try-finally模式去手动释放锁。
- synchronized是不可中断的，除非抛出异常或者程序正常退出，Lock可中断，使用`lockInterruptibly`，调用interrupt方法可中断。
- synchronized是非公平锁，Lock默认是非公平锁，但是可以通过构造函数传入`boolean`类型值更改是否为公平锁。
- 锁是否能绑定多个条件，synchronized没有condition的说法，要么唤醒所有线程，要么随机唤醒一个线程，Lock可以使用condition实现分组唤醒需要唤醒的线程，实现精准唤醒。
- synchronized 锁只能同时被一个线程拥有，但是 Lock 锁没有这个限制，例如在读写锁中的读锁，是可以同时被多个线程持有的，可是 synchronized做不到。
- 性能区别：在 Java 5 以及之前synchronized的性能比较低，但是到了 Java 6 以后 JDK 对 synchronized 进行了很多优化，比如自适应自旋、锁消除、锁粗化、轻量级锁、偏向锁等。

## Java中如何进行锁优化

在 JDK 1.6 中 `HotSopt` 虚拟机对 synchronized 内置锁的性能进行了很多优化，包括自旋锁、自适应的自旋、锁消除、锁粗化、偏向锁、轻量级锁等。

**自旋锁**

自旋锁可以减少线程阻塞造成的线程切换。其执行步骤如下：

1. 当前线程尝试去竞争锁。
2. 竞争失败，准备阻塞自己。
3. 但是并没有阻塞自己，进入自旋状态（空等待，比如一个空的有限for循环）。
4. 自旋状态下，继续竞争锁。
5. 如果自旋期间成功获取锁，那么结束自旋状态，否则进入阻塞状态。

自旋锁适合在持有锁时间短，并且竞争激烈的场景下使用。在JDK1.6中自旋锁默认开启。可以使用*-XX:+UseSpinning*开启，*-XX:-UseSpinning*关闭自旋锁优化。自旋的默认次数为**10次**，可以使用*-XX:preBlockSpin*参数修改默认的自旋次数。

**自适应的自旋锁**

适应性自旋，是赋予了自旋一种学习能力，它并不固定自旋10次。他可以根据它前面线程的自旋情况，从而调整它的自旋。

**锁消除**

经过逃逸分析之后，如果发现某些对象不可能被其他线程访问到，那么就可以把它们当成栈上数据，栈上数据由于只有本线程可以访问，自然是线程安全的，也就无需加锁，所以会把这样的锁给自动去除掉。

**锁粗化**

同步块的作用范围应该尽可能小，仅在共享数据的实际作用域中才进行同步，这样做的目的是为了使需要同步的操作数量尽可能缩小，缩短阻塞时间，如果存在锁竞争，那么等待锁的线程也能尽快拿到锁。

但是加锁解锁也需要消耗资源，如果存在一系列的连续加锁解锁操作，可能会导致不必要的性能损耗。

锁粗化就是将多个连续的加锁、解锁操作连接在一起，扩展成一个范围更大的锁，避免频繁的加锁解锁操作。

**偏向锁/轻量级锁/重量级锁 锁升级**

下面细讲... ...

## Java 锁升级过程

synchronized通过monitor机制来实现线程同步，而monitor机制有依赖于操作系统的mutex lock实现。当每次获取锁/释放锁都会涉及内核态与用户态的转换，成本高，所以synchronized又被称为重量级锁。

为了减少获得锁和释放锁带来的性能消耗，从JDK 6开始，引入了“偏向锁”和“轻量级锁”。所以，目前锁一共有4种状态，级别从低到高依次是：**无锁状态、偏向锁状态、轻量级锁状态和重量级锁状态，这几个状态会随着竞争情况逐渐升级**。**锁可以升级但不能降级**，为的是提高获得锁与释放锁的效率。

四种锁状态对应的的Mark Word内容描述如下：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20201027235703700.png" width="600px"/>
</div>

在64位虚拟机下，Mark Word在不同锁状态存储结构如下：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20201026011946370.png" width="600px"/>
</div>

**无锁**：通过CAS操作来加锁（CAS原理需要知道）

**偏向锁**：锁总是由同一线程获得，锁升级为偏向锁。**在线程进入和退出同步块时不再通过CAS操作来加锁和解锁，而是检测Mark Word里是否存储着指向当前线程的偏向锁**。引入偏向锁是为了在没有多线程竞争的情况下尽量减少不必要的轻量级锁执行路径，**偏向锁只需要在置换ThreadID的时候依赖一次CAS原子指令即可**。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20201029093518296.png" width="650px"/>
</div>

**轻量级锁**：偏向锁多应用只有一个线程访问同步块场景中，一旦偏向锁被其他线程访问，就会升级为轻量级锁。线程会通过**自旋**的形式尝试获取锁，不会阻塞，从而提高性能。

**使用轻量级锁的多线程之间不存在锁竞争，线程是交替执行同步块的**。引入轻量级锁的目的正是**在没有多线程竞争的前提下，减少传统的重量级锁使用操作系统互斥量产生的性能消耗**。

轻量级锁加锁过程如下：

1. 在代码块进入同步块时，如果同步对象锁状态为无锁状态（锁标志位01，是否偏向锁0），虚拟机首先将在当前线程的栈帧中建立一个名为锁记录（Lock Record）的空间，用于存储锁对象目前的Mark Word的拷贝，官方称之为Displaced Mark Word。
2. 拷贝对象头中的Mark Word复制到锁记录中。
3. 拷贝成功后，虚拟机将使用**CAS操作**尝试将对象的Mark Word更新为指向Lock Record的指针，并将Lock Record里的owner指针指向对象的Mark Word。
4. 如果更新动作成功了，那么这个线程就拥有了该对象的锁，此时对象Mark Word锁标志位设置为00，表示此对象处于轻量级锁定状态。
5. 如果更新操作失败了，虚拟机首先会检查对象的Mark Word是否指向当前线程的栈帧，如果是，说明当前线程已经拥有了这个对象的锁，那么可用直接进入同步块继续执行。否则说明有多个线程竞争锁，若当前只有一个等待线程，则线程会通过自旋进行等待；但当自旋超过一定次数或者一个线程持有锁，一个在自旋，又来了第三个线程竞争锁，那么轻量级锁会膨胀升级为重量级锁，锁标志位设置为10。

**重量级锁**：当多线程竞争激烈，轻量级锁升级为重量级锁。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20201029210053486.png" width="1000px"/>
</div>

Java锁升级过程即：`无锁-->偏向锁-->轻量级锁-->重量级锁`。它们4者并不是相互替代的关系，它们只是在不同场景下的不同选择：

| 锁       | 优点                                                                     | 缺点                                                | 适用场景                                                         |
| -------- | ------------------------------------------------------------------------ | --------------------------------------------------- | ---------------------------------------------------------------- |
| 偏向锁   | 加锁和解锁不需要额外消耗，<br>和执行非同步方法相比仅仅存在纳秒级的差距。 | 线程间存在锁竞争，<br>会带来额外的锁撤销的消耗。    | 适用于只有一个线程访问同步块场景。                               |
| 轻量级锁 | 竞争的线程不会阻塞，<br>提高了线程的响应速度。                           | 如果始终得不到锁竞争的线程，<br>使用自旋会消耗CPU。 | 适用于少量线程竞争锁，且线程持有锁时间不长，追求响应时间的场景。 |
| 重量级锁 | 线程竞争不会使用自旋，<br>不会消耗CPU。                                  | 线程阻塞，响应时间缓慢。                            | 追求吞吐量，<br>锁竞争激烈，持有时间较长的场景。                 |

## 死锁、活锁、饥饿区别

**所谓死锁，就是一组互相竞争资源的线程因互相等待，导致“永久”阻塞的现象**。

产生死锁的四个必要条件：

（1） **互斥**：一个资源每次只能被一个线程使用。

（2） **请求与保持**：一个线程因请求资源而阻塞时，不释放已获得的资源。

（3） **不剥夺**：进程已获得的资源，在末使用完之前，不能强行剥夺。

（4） **循环等待**：若干线程之间形成一种头尾相接的循环等待资源关系。

只有这四个条件都发生时才会出现死锁，那么反过来，也就是说只要我们破坏其中一个，就可以成功预防死锁的发生。

定位死锁最常见的方式就是利用 **jstack** 等工具获取线程栈，然后定位互相之间的依赖关系，进而找到死锁。如果是比较明显的死锁，往往 `jstack` 等就能直接定位，类似 **JConsole** 甚至可以在图形界面进行有限的死锁检测。

如果程序运行时发生了死锁，绝大多数情况下都是无法在线解决的，只能重启、修正程序本身问题。编写程序时避免死锁的常见方法：

（1）避免一个线程同时获取多个锁。

（2）避免一个线程在锁内同时占用多个资源，尽量保证每个锁只占用一个资源。

（3）尝试使用定时锁，使用lock.tryLock（timeout）来替代使用内部锁机制。

（4）对于数据库锁，加锁和解锁必须在一个数据库连接里，否则会出现解锁失败的情况。

死锁、活锁、饥饿都是常见的活跃性问题（指的是某个操作无法执行下去）。

**活锁，指的是线程虽然没有发生阻塞，但仍然会存在执行不下去的情况**（类比行人让路）。

- 解决“活锁”的方案很简单，谦让时，尝试等待一个**随机的等待时间**就可以了。

**饥饿，指线程因无法访问所需资源而无法执行下去的情况**。在 CPU 繁忙的情况下，优先级低的线程得到执行的机会很小，就可能发生线程“饥饿”。

- 在并发编程里，解决“饥饿”问题主要是使用**公平锁**。所谓公平锁，是一种先来后到的方案，线程的等待是有顺序的，排在等待队列前面的线程会优先获得资源。

## 乐观锁与悲观锁

对于同一个数据的并发操作，**悲观锁认为自己在使用数据的时候一定会别的线程来修改数据，因此在获取数据的时候会先加锁，确保数据不会被别的线程修改**。Java中，synchronized关键字和Lock的实现类都是悲观锁。

而**乐观锁认为自己使用数据时不会有别的线程来修改数据，所以不会添加锁，只是在更新数据的时候会去判断之前有没有别的线程更新这个数据**。如果这个数据没有被更新，当前线程将子哦系修改的数据成功写入。如果数据已经被其他线程更新，则根据不同的实现方式执行不同的操作（例如报错或者自动重试）。Java中，常用CAS算法来实现乐观锁。

它们应用场景区别：

- 悲观锁适合写操作多的场景，先加锁可以保证写操作时数据正确。
- 乐观锁适合读操作多的场景，不加锁的特点能够使其读操作的性能大幅提升。

## 公平锁与非公平锁

公平锁是指多**个线程按照申请锁的顺序来获取锁**，线程直接进入队列中排队，队列中的第一个线程才能获得锁。公平锁的优点是等待锁的线程不会饥饿，缺点是整体吞吐率相对较低。

非公平锁是多个线程加锁时直接尝试获取锁，获取不到才会到等待队列的队尾等待。非公平锁优点是可以减少唤起线程的开销，整体的吞吐效率高，缺点是可能产生饥饿。

查看ReentrantLock源码，ReentrantLock里面有一个内部类Sync，Sync继承AQS（AbstractQueuedSynchronizer），添加锁和释放锁的大部分操作实际上都是在Sync中实现的。它有公平锁FairSync和非公平锁NonfairSync两个子类。**ReentrantLock**默认使用非公平锁，也可以通过构造器来显示的指定使用公平锁。

可以明显的看出公平锁与非公平锁的lock()方法唯一的区别就在于公平锁在获取同步状态时多了一个限制条件：hasQueuedPredecessors()。

<div align="center">  
<img src="https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018b/bc6fe583.png" width="900px"/>
</div>

再进入hasQueuedPredecessors()，可以看到该方法主要做一件事情：主要是判断当前线程是否位于同步队列中的第一个。如果是则返回true，否则返回false。

<div align="center">  
<img src="https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018b/bd0036bb.png" width="800px"/>
</div>

综上，公平锁就是**通过同步队列来实现多个线程按照申请锁的顺序来获取锁**，从而实现公平的特性。非公平锁加锁时不考虑排队等待问题，直接尝试获取锁，所以存在后申请却先获得锁的情况。

## 可重入锁与非可重入锁

可重入锁又名递归锁，是指在同一个线程在外层方法获取锁的时候，再进入该线程的内层方法会自动获取锁（前提锁对象得是同一个对象或者class），不会因为之前已经获取过还没释放而阻塞。**Java中ReentrantLock和synchronized都是可重入锁**，可重入锁的一个优点是可**一定程度避免死锁**。

首先ReentrantLock和NonReentrantLock都继承父类AQS，其父类AQS中维护了一个同步状态status来计数重入次数，status初始值为0。

当线程尝试获取锁时，可重入锁先尝试获取并更新status值，如果status == 0表示没有其他线程在执行同步代码，则把status置为1，当前线程开始执行。如果status != 0，则判断当前线程是否是获取到这个锁的线程，如果是的话执行status+1，且当前线程可以再次获取锁。而非可重入锁是直接去获取并尝试更新当前status的值，如果status != 0的话会导致其获取锁失败，当前线程阻塞。

释放锁时，可重入锁同样先获取当前status的值，在当前线程是持有锁的线程的前提下。如果status-1 == 0，则表示当前线程所有重复获取锁的操作都已经执行完毕，然后该线程才会真正释放锁。而非可重入锁则是在确定当前线程是持有锁的线程之后，直接将status置为0，将锁释放。

<div align="center">  
<img src="https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018b/32536e7a.png" width="1000px"/>
</div>

## 独享锁与共享锁

**独享锁也叫排他锁，是指该锁一次只能被一个线程所持有。**如果线程T对数据A加上排它锁后，则其他线程不能再对A加任何类型的锁。**获得排它锁的线程即能读数据又能修改数据**。JDK中的synchronized和JUC中Lock的实现类都是独享锁。

**共享锁是指该锁可被多个线程所持有。**如果线程T对数据A加上共享锁后，则其他线程只能对A再加共享锁，不能加排它锁。**获得共享锁的线程只能读数据，不能修改数据。**

独享锁与共享锁也是通过AQS来实现的，通过实现不同的方法，来实现独享或者共享。

我们看到ReentrantReadWriteLock有两把锁：ReadLock和WriteLock，由词知意，一个读锁一个写锁，合称“读写锁”。再进一步观察可以发现ReadLock和WriteLock是靠内部类Sync实现的锁。

<div align="center">  
<img src="https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018b/762a042b.png" width="1000px"/>
</div>

在ReentrantReadWriteLock里面，读锁和写锁的锁主体都是Sync，但读锁和写锁的加锁方式不一样。**读锁是共享锁，写锁是独享锁。读锁的共享锁可保证并发读非常高效，而读写、写读、写写的过程互斥，因为读锁和写锁是分离的**。所以ReentrantReadWriteLock的并发性相比一般的互斥锁有了很大提升。

ReadWriteLock读写锁执行流程：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20201104111903784.png" width="800px"/>
</div>
**读写锁不支持锁升级，支持锁降级**。锁降级指的是线程获取到了写锁，在没有释放写锁的情况下，又获取读锁。

# 原子类

原子（atomic）本意是“不能被进一步分割的最小粒子”，而原子操作（atomic operation）意为“不可被中断的一个或一系列操作”。

即使是在多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程干扰。简而言之，**原子类就是具有原子/原子操作特征的类，它们能无锁地避免原子性问题**。

并发包 `java.util.concurrent` 的原子类都存放在`java.util.concurrent.atomic`下。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210606194422177.png" width="900px"/>
</div>

## 累加器

**原子类型累加器**是**JDK1.8**引进的并发新技术，它可以看做基本类型原子类的部分加强。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/2020110522343065.png" width="600px"/>
</div>

那么以`LongAdder`为例，**有了AtomictLong为什么还要引入LongAdder？**

* 引入LongAdder累加器的初衷------解决高并发环境下AtomictLong的自旋瓶颈问题。

**那LongAdder为什么高并发性场景能会更好？**

* 原子累加器通过**分段思想**改进原子类，其改进思路：
  * AtomicInteger 和 AtomicLong 里的 value 是所有线程竞争读写的热点数据；
  * 将单个 value 拆分成跟线程一样多的数组 `Cell[]`；
  * 每个线程写自己的 `Cell[i]++`，最后对数组求和。

>  AtomicLong 里内部变量value保存着实际值，在高并发场景下，value变量也就成了多线程竞争的一个热点。而LongAdder的基本思路就是“分散热点”，将value值分散到一个数组中，不同线程会命中到数组的不同槽中，各个线程只对自己槽中的那个值进行CAS操作，这样分散热点，减小冲突的概率。如果要获取真正的值，只要将各个槽中的变量值累加返回即可。这种做法其实跟JDK1.8之前的ConcurrentHashMap“分段锁”的实现思路是一样的。

**LongAdder能否替代AtomicLong吗？**

* AtomicLong提供的功能更丰富，尤其是`addAndGet、decrementAndGet、compareAndSet`这些方法。addAndGet、decrementAndGet除了单纯的做自增自减外，还可以立即获取增减后的值，而LongAdder则需要做同步控制才能精确获取增减后的值。如果业务需求需要精确的控制计数，则使用AtomicLong比较合适；
* 一般而言，**在低竞争的并发环境下 `AtomicInteger` 的性能是要比 `LongAdder` 的性能好，而高竞争环境下和者写多读少环境下， `LongAdder` 的性能比 `AtomicInteger` 好**。

## CAS原理

CAS全称 **Compare And Swap（比较与交换）**，是一种无锁算法。在不使用锁（没有线程被阻塞）的情况下实现多线程之间的**变量同步**。

CAS是**原子性**的操作(读和写两者同时具有原子性)，其实现方式是通过借助`C/C++`调用CPU指令完成的，效率很高。

CAS算法涉及到三个操作数：

- V 内存地址存放的实际值
- A 比较的旧值 
- B 更新的新值

**当且仅当V的值等于A时（旧值和内存中实际的值相同），表明旧值A已经是目前最新版本的值，自然而然可以将新值 B 赋值给 V**。反之则表明V和A变量不同步，直接返回V即可。当多个线程使用CAS操作一个变量时，只有一个线程会更新成功，其余失败的线程会重新尝试。也就是说，“更新”是一个不断重试的操作。

下面以基本原子类`AtomicInteger`为例，来理解原子操作的实现原理。

查看`AtomicInteger`源码：

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

接下来，查看`AtomicInteger`的自增函数`incrementAndGet()`的源码时，发现自增函数底层调用的是`unsafe.getAndAddInt()`。

```java
    // AtomicInteger 自增方法
    public final int incrementAndGet() {
      return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
    }    
------------------------------------------------------------------------
	// Unsafe.class
	public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

        return var5;
    }
```

`Unsafe`还有很多个`CAS`操作的相关方法，比如：

```java
public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);

public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);

--- omit ---
```

这些函数是是`CAS`缩写的由来。

还是以`compareAndSwapInt`为例，查看OpenJDK 8 中Unsafe.cpp的源码：

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

根据OpenJDK 8的源码我们可以看出，`getAndAddInt()`循环获取给定对象o中的偏移量处的值v，然后判断内存值是否等于v。如果相等则将内存值设置为 v + delta，否则返回false，继续循环进行重试，直到设置成功才能退出循环，并且将旧值返回。整个“比较+更新”操作封装在`compareAndSwapInt()`中，在JNI里是借助于一个CPU指令完成的，属于原子操作，可以保证多个线程都能够看到同一个变量的修改值。

后续JDK通过CPU的**cmpxchg指令**，去比较寄存器中的 A 和 内存中的值 V。如果相等，就把要写入的新值 B 存入内存中。如果不相等，就将内存值 V 赋值给寄存器中的值 A。然后通过Java代码中的**while循环调用cmpxchg指令进行重试，直到设置成功为止**。

## CAS实现原子操作的三大问题

CAS虽然很高效地解决了原子操作，但是CAS仍然存在三大问题。ABA问题，循环时间长开销大，以及只能保证一个共享变量的原子操作。

* ABA问题：CAS需要在操作值的时候去检查内存中的值是否发生变化，没有发生变化才会更新内存值。但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有发生变化，但是实际上却变化了。
  * ABA问题的解决思路就是在变量前面**添加版本号**，每次变量更新的时候都把版本号加1，这样变化过程就从“A－B－A”变成了“1A－2B－3A”。JDK从1.5开始提供了**带版本号的引用类型AtomicStampedReference类**来解决ABA问题，具体操作封装在compareAndSet()中。compareAndSet()首先检查当前引用和当前标志与预期引用和预期标志是否相等，如果都相等，则以原子方式将引用值和标志的值设置为给定的更新值。
* 循环时间长开销大：CAS操作如果长时间不成功，会导致其一直自旋，给CPU带来非常大的开销。
  * 如果JVM能支持处理器提供的**pause指令**，那么效率会有一定的提升。pause指令有两个作用：第一，它可以延迟流水线执行指令（de-pipeline），使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零；第二，它可以避免在退出循环的时候因内存顺序冲突（MemoryOrder Violation）而引起CPU流水线被清空（CPU Pipeline Flush），从而提高CPU的执行效率。
*  只能保证一个共享变量的原子操作：对一个共享变量执行操作时，CAS能够保证原子操作，但是对多个共享变量操作时，CAS是无法保证操作的原子性的。
  * 多个共享变量操作时，一般都用锁解决。但Java从1.5开始JDK提供了**引用类型AtomicReference类**来保证引用对象之间的原子性，可以把多个变量放在一个对象里来进行CAS操作。

# 并发容器类

Java 在 1.5 版本之前所谓的线程安全的容器，主要指的就是**同步容器**。不过同步容器有个最大的问题，那就是性能差，所有方法都用 synchronized 来保证互斥，串行度太高了（例如Hashtable，内部的方法基本都经过`synchronized` 修饰，存储数据具有强一致特点）。因此 Java 在 1.5 及之后版本提供了性能更高的容器，我们一般称为**并发容器**。

并发编程大师Doug Lea开发了非常多的并发容器和框架，这些容器大部分在 `java.util.concurrent` 包中。并发容器数量非常多，主要分为四大类：List、Map、Set 和 Queue。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210606192835508.png" width="900px"/>
</div>

## ConcurrentHashMap

HashMap不是线程安全的，在并发情况下可能会造成`Race Condition`，形成环形链表从而导致死循环。在并发场景下要保证线程安全可以使用 `Collections.synchronizedMap()` 方法来包装HashMap；Hashtable使用synchronized来保证线程安全，所有访问Hashtable的线程都必须竞争同一把锁，在线程竞争激烈的情况下效率非常低下。

基于此，ConcurrentHashMap应运而生。在 ConcurrentHashMap 中，无论是读操作还是写操作都能保证很高的性能：在进行读操作时(几乎)不需要加锁，在进行写操作时，**在 JDK1.7 的时候，`ConcurrentHashMap`（分段锁）** 对整个桶数组进行了分割分段(Segment)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率；到了 JDK1.8 的时候已经摒弃了 Segment 的概念，而是直接用 Node 数组+链表+红黑树的数据结构来实现，并发控制使用 **synchronized 和 CAS** 来操作。

## ConcurrentSkipListMap

ConcurrentHashMap 是基于 HashMap 实现的，该容器在数据量比较大的时候，链表会转换为红黑树。红黑树在并发情况下，删除和插入过程中有个平衡的过程，会牵涉到大量节点，因此竞争锁资源的代价相对比较高。因此，ConcurrentHashMap 对于小数据量的存取比较有优势。

ConcurrentSkipListMap 是基于基于**跳表**实现，ConcurrentSkipListMap 的特点是跳表**插入、删除、查询操作平均的时间复杂度是 O(log n)**，适用于大数据量存取的场景，最常见的是基于跳跃表实现的数据量比较大的缓存。

## CopyOnWriteArrayList

CopyOnWriteArrayList 是 `java.util.concurrent` 包提供的方法，它实现了**读操作无锁，写操作则通过操作底层数组的新副本来实现**，是一种读写分离（读读共享、写写互斥、读写互斥、写读互斥）的并发策略。

CopyOnWriteArrayList 内部维护了一个数组，成员变量 array 就指向这个内部数组，所有的读操作都是基于 array 进行的。


如果在遍历 array 的同时，还有一个写操作，例如增加元素，CopyOnWriteArrayList 是如何处理的呢?

CopyOnWriteArrayList 会将 array 复制一份，然后在新复制处理的数组上执行增加元素的操作，执行完之后再将 array 指向这个新的数组。通过下图你可以看到，读写是可以并行的，遍历操作一直都是基于原 array 执行，而写操作则是基于新 array 进行。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210619003132775.png" width="800px"/>
</div>

从 `CopyOnWriteArrayList` 的名字就能看出`CopyOnWriteArrayList` 是满足`CopyOnWrite` 的 ArrayList，所谓`CopyOnWrite` 也就是说：在计算机，如果你想要对一块内存进行修改时，我们不在原有内存块中进行写操作，而是将内存拷贝一份，在新的内存中进行写操作，写完之后呢，就将指向原来内存指针指向新的内存，原来的内存就可以被回收掉了。

使用 CopyOnWriteArrayList 需要注意的两个方面。一个是应用场景，CopyOnWriteArrayList 仅适用于写操作非常少的场景，而且能够容忍读写的短暂不一致。另一个需要注意的是，**CopyOnWriteArrayList 迭代器是只读的，不支持增删改。因为迭代器遍历的仅仅是一个快照，而对快照进行增删改是没有意义的**。

## 非阻塞队列

### ConcurrentLinkedQueue

ConcurrentLinkedQueue是一个基于链接节点的无界线程安全队 列，它采用先进先出的规则对节点进行排序，当我们添加一个元素的时候，它会添加到队列的尾部；当我们获取一个元素时，它会返回队 列头部的元素。它采用了“wait-free”算法（即CAS算法）来实现，该算 法在Michael&Scott算法上进行了一些修改。

ConcurrentLinkedQueue 源码这里就不分析了，我们只需要记住 ConcurrentLinkedQueue 主要使用 **CAS** 非阻塞算法来实现线程安全就好了。ConcurrentLinkedQueue 适合在对性能要求相对较高，同时对队列的读写存在多个线程同时进行的场景。

## 阻塞队列

### ArrayBlockingQueue

**ArrayBlockingQueue** 是 BlockingQueue 接口的**有界**队列实现类，底层采用**数组**来实现。ArrayBlockingQueue 一旦创建，容量不能改变。其并发控制采用可重入锁来控制，不管是插入操作还是读取操作，都需要获取到锁才能进行操作。当队列容量满时，尝试将元素放入队列将导致操作阻塞；尝试从一个空队列中取一个元素也会同样阻塞。

ArrayBlockingQueue 默认情况下不能保证线程访问队列的公平性，所谓公平性是指严格按照线程等待的绝对时间顺序，即最先等待的线程能够最先访问到 ArrayBlockingQueue。而非公平性则是指访问 ArrayBlockingQueue 的顺序不是遵守严格的时间顺序。有可能存在，当 ArrayBlockingQueue 可以被访问时，长时间阻塞的线程依然无法访问到 ArrayBlockingQueue。如果保证公平性，通常会降低吞吐量。如果需要获得公平性的 ArrayBlockingQueue，可采用如下代码：

```java
private static ArrayBlockingQueue<Integer> blockingQueue = new ArrayBlockingQueue<Integer>(10,true);
```

### LinkedBlockingQueue

**LinkedBlockingQueue** 底层基于**单向链表**实现的有界阻塞队列，**可以当做无界队列也可以当做有界队列来使用**，同样满足 FIFO 的特性，与 ArrayBlockingQueue 相比起来具有更高的吞吐量，为了防止 LinkedBlockingQueue 容量迅速增，损耗大量内存。通常在创建 LinkedBlockingQueue 对象时，会指定其大小，如果未指定，容量等于 `Integer.MAX_VALUE`。

**相关构造方法:**

```java
    /**
     *某种意义上的无界队列
     * Creates a {@code LinkedBlockingQueue} with a capacity of
     * {@link Integer#MAX_VALUE}.
     */
    public LinkedBlockingQueue() {
        this(Integer.MAX_VALUE);
    }

    /**
     *有界队列
     * Creates a {@code LinkedBlockingQueue} with the given (fixed) capacity.
     *
     * @param capacity the capacity of this queue
     * @throws IllegalArgumentException if {@code capacity} is not greater
     *         than zero
     */
    public LinkedBlockingQueue(int capacity) {
        if (capacity <= 0) throw new IllegalArgumentException();
        this.capacity = capacity;
        last = head = new Node<E>(null);
    }
```

### PriorityBlockingQueue

**PriorityBlockingQueue** 是一个**支持优先级的无界阻塞队列**。默认情况下元素采用自然顺序进行排序，也可以通过自定义类实现 `compareTo()` 方法来指定元素排序规则，或者初始化时通过构造器参数 `Comparator` 来指定排序规则。

PriorityBlockingQueue 并发控制采用的是 **ReentrantLock**，队列为无界队列（ArrayBlockingQueue 是有界队列，LinkedBlockingQueue 也可以通过在构造函数中传入 capacity 指定队列最大的容量，但是 PriorityBlockingQueue 只能指定初始的队列大小，后面插入元素的时候，**如果空间不够的话会自动扩容**）。

简单地说，它就是 PriorityQueue 的线程安全版本。不可以插入 null 值，同时，插入队列的对象必须是可比较大小的（comparable），否则报 ClassCastException 异常。它的插入操作 put 方法不会 block，因为它是无界队列（take 方法在队列为空的时候会阻塞）。

### DelayQueue

DelayQueue是一个**支持延时获取元素的无界阻塞队列**。队列使用 PriorityQueue来实现。队列中的元素必须实现Delayed接口，在创建元 素时可以指定多久才能从队列中获取当前元素。只有在延迟期满时才 能从队列中提取元素。

DelayQueue非常有用，可以将DelayQueue运用在以下应用场景。 

* 缓存系统的设计：可以用DelayQueue保存缓存元素的有效期，使 用一个线程循环查询DelayQueue，一旦能从DelayQueue中获取元素 时，表示缓存有效期到了。
* 定时任务调度：使用DelayQueue保存当天将会执行的任务和执行 时间，一旦从DelayQueue中获取到任务就开始执行，比如TimerQueue 就是使用DelayQueue实现的。

### SynchronousQueue

SynchronousQueue是一个**不存储元素的阻塞队列**。每一个put操作必须等待一个take操作，否则不能继续添加元素。 它支持公平访问队列。默认情况下线程采用非公平性策略访问队列。使用以下构造方法可以创建公平性访问的SynchronousQueue，如 果设置为true，则等待的线程会采用先进先出的顺序访问队列。

```java
public SynchronousQueue(boolean fair) {
    transferer = fair new TransferQueue() : new TransferStack();
}
```

SynchronousQueue可以看成是一个传球手，负责把生产者线程处理的数据直接传递给消费者线程。队列本身并不存储任何元素，非常适合传递性场景。SynchronousQueue的吞吐量高于 LinkedBlockingQueue和ArrayBlockingQueue。

# 常见并发工具类

### Semaphore 信号量

Semaphore（信号量）用来控制同时访问特定资源的线程数量，它可以指定多个线程同时访问某个资源。

Semaphore可以选择公平|非公平的构造方式，但都是共享锁的实现方式。它默认构造AQS的state为permits。当执行任务的线程数量超出permits，那么多余的线程将会被放入阻塞队列park，并自旋判断state是否大于0。只有当state大于0的时候，阻塞的线程才能继续执行，此时先前执行任务的线程继续执行release方法，release方法使得state的变量会加1，那么自旋的线程便会判断成功。如此，每次只有最多不超过permits数量的线程能自旋成功，便限制了执行任务线程的数量。

Semaphore常用于做流量控制，特别是公用资源有限的应用场景。

### CountDownLatch 倒计时器

CountDownLatch是一种同步辅助工具，它允许一个或多个线程等待其他线程完成操作，例如阻塞主线程，N 个子线程满足条件时主线程继续。

CountDownLatch是通过一个计数器来实现的，计数器的初始化值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就相应得减1。当计数器到达0时，表示所有的线程都已完成任务，然后在闭锁上等待的线程就可以恢复执行任务。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210609160234329.png" width="400px"/>
</div>

CountDownLatch是AQS中共享锁的一种实现。CountDownLatch 是一次性的，计数器的值只能在构造方法中初始化一次，之后没有任何机制再次对其设置值，当 CountDownLatch 使用完毕后，它不能再次被使用。如果需要能重置计数，可以使用CyclicBarrier。

CountDownLatch主要应用场景：

1. **实现最大的并行性**：同时启动多个线程，实现最大程度的并行性。例如110跨栏比赛中，所有运动员准备好起跑姿势，进入到预备状态，等待裁判一声枪响。裁判开了枪，所有运动员才可以开跑。
2. **开始执行前等待N个线程完成各自任务**：例如Master 线程等待 Worker 线程把任务执行完。形象理解为等所有人干完手上的活，一起去吃饭。

### CyclicBarrier 循环栅栏

CyclicBarrier的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续运行。CyclicBarrier 可以用于多线程计算数据，最后合并计算结果的应用场景。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20201105150659614.png" width="400px"/>
</div>

- CyclicBarrier 内部通过一个 count 变量作为计数器，cout 的初始值为 parties 属性的初始化值，每当一个线程到了栅栏，那么就将计数器减1。如果 count 值为 0 了，表示这是这一代最后一个线程到达栅栏，就会将代更新并重置计数器，并唤醒所有之前等待在栅栏上的线程。
- 如果在等待的过程中，线程中断都也会抛出BrokenBarrierException异常，并且这个异常会传播到其他所有的线程，CyclicBarrier会被损坏。
- 如果超出指定的等待时间，当前线程会抛出 TimeoutException 异常，其他线程会抛出BrokenBarrierException异常，CyclicBarrier会被损坏。

### CyclicBarrier和CountDownLatch的区别

| CountDownLatch                         | CyclicBarrier                              |
| -------------------------------------- | ------------------------------------------ |
| 在主线程里 await 阻塞并做聚合          | 直接在各个子线程里 await 阻塞，回调聚合    |
| N 个线程执行了countdown，主线程继续    | N个线程执行了 await 时，N 个线程继续       |
| 主线程里拿到同步点                     | 回调被最后到达同步点的线程执行             |
| 基于 AQS 实现，state 为 count，递减到0 | 基于可重入锁 condition.await/signalAll实现 |
| 不可以复用                             | 计数为0时重置为 N，可以复用                |

形象地比较：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210609160907584.png" width="800px"/>
</div>

- CountDownLatch的计数器只能使用一次。而CyclicBarrier的计数器可以使用reset()方法重置，可多次使用。
- 侧重点不同。CountDownLatch多用于某一个线程等待若干个其他线程执行完任务之后，它才执行；而CyclicBarrier一般用于多个线程互相等待至一个同步点，然后这些线程再继续一起执行。
- CyclicBarrier还提供其他有用的方法，比如getNumberWaiting方法可以获得Cyclic-Barrier阻塞的线程数量；isBroken()方法用来了解阻塞的线程是否被中断。

# AQS原理

AbstractQueuedSynchronizer，即队列同步器，一般简称AQS。**它是构建锁或者其他同步组件的基础**（如 Semaphore、CountDownLatch、ReentrantLock、ReentrantReadWriteLock），**是 JUC 并发包中的核心基础组件。**

它**抽象了竞争的资源和线程队列**，使用了一个int成员变量state表示同步状态， CLH 队列（FIFO双向队列）来完成同步状态的管理。如果当前线程获取同步状态失败（锁）时，AQS 则会将当前线程以及等待状态等信息构造成一个节点（Node）并将其加入同步队列队尾，同时会阻塞当前线程；首节点是获取同步状态成功的节点，当同步状态释放时，则会把节点中的线程唤醒，使其再次尝试获取同步状态。

**独占式**同步状态获取和释放锁的过程大致为：在获取同步状态时，同步器维护一个FIFO同步队列，获取状态失败的线程都会加入到队列中并在队列中进行**自旋**；移出队列（或停止）自旋的条件时前驱节点为头节点且成功获取了同步状态。在释放同步状态时，同步器用*tryReleasw(int arg)*方法释放同步状态，然后唤醒头节点的后继节点。

**独占式超时**获取同步状态doAcquireNanos(int arg,long nanosTimeout)*和独占式获取同步状态*acquire(int args)*在流程上 非常相似，其主要区别在于未获取到同步状态时的处理逻辑。 *acquire(int args)*在未获取到同步状态时，将会使当前线程一直处于等待 状态，而*doAcquireNanos(int arg,long nanosTimeout)*会使当前线程等待 nanosTimeout纳秒，如果当前线程在nanosTimeout纳秒内没有获取到同 步状态，将会从等待逻辑中自动返回。

**共享式**获取与独占式获取最主要的区别在于同一时刻能否有多个线程同时获取到同步状态。

共享式获取需重写*tryAcquireShared、tryReleaseShared*方法，并访问*acquireShared、releaseShared*方法实现相应的功能。与独占式相对，共享式支持多个线程同时获取到同步状态并进行工作，如 *Semaphore、CountDownLatch、 CyclicBarrier*等。

当我们需要阻塞或者唤醒一个线程的时候，AQS 都是通过*LockSupport* 这个工具类来完成的。

# ThreadLocal应用及原理

`ThreadLocal`可以理解为线程的变量副本，该副本只能由当前 Thread 使用，每个线程隔离，`ThreadLocal`无法解决共享对象的更新问题，使用时建议用static修饰！（**Synchronized用于线程间的数据共享，而`ThreadLocal`则用于线程间的数据隔离**）`ThreadLocal` 适用于每个线程需要自己独立的实例且该实例需要在多个方法中被使用，也即变量在线程间隔离而在方法或类间共享的场景。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210708233447977.png" width="800px"/>
</div>

`ThreadLocal`数据结构是什么？

* `Thread`类有一个类型为`ThreadLocal.ThreadLocalMap`的实例变量`threadLocals`，也就是说每个线程有一个自己的`ThreadLocalMap`。`ThreadLocalMap`有自己的独立实现，可以简单地将它的`key`视作`ThreadLocal`，`value`为代码中放入的值（实际上`key`并不是`ThreadLocal`本身，而是它的一个**弱引用**）。
* 每个线程在往`ThreadLocal`里放值的时候，都会往自己的`ThreadLocalMap`里存，读也是以`ThreadLocal`作为引用，在自己的`map`里找对应的`key`，从而实现了**线程隔离**。`ThreadLocalMap`有点类似`HashMap`的结构，只是`HashMap`是由**数组+链表**实现的，而`ThreadLocalMap`中并没有**链表**结构。

GC时，key一定会被回收吗？

* 在垃圾回收时，如果存在强引用，则不会被回收；如果强引用不存在的话，那么 `key` 就会被回收，也就是会出现我们 `value` 没被回收，`key` 被回收，导致 `value` 永远存在，出现**内存泄漏**。我们只要在使用完该 key 值之后，将 value 值通过 remove 方法 remove 掉，或者触发`expungeStaleEntry()`方法探测式清理数据，就可以防止内存泄漏了。

`ThreaLocal`的hash算法？

* hash算法采用**斐波那契数**实现，带来好处是处理后的key分布十分均匀，相对而言减少了哈希冲突。

`ThreadLocalMap.set()`原理？往`ThreadLocalMap`中`set`数据（**新增**或者**更新**数据）分为好几种情况：

* **第一种情况：** 通过`hash`计算后的槽位对应的`Entry`数据为空，直接将数据放到该槽位即可。
* **第二种情况：** 槽位数据不为空，`key`值与当前`ThreadLocal`通过`hash`计算获取的`key`值一致，直接更新该槽位的数据。
* **第三种情况：** 槽位数据不为空，往后遍历过程中，在找到`Entry`为`null`的槽位之前，没有遇到`key`过期的`Entry`，线性往后查找，如果找到`Entry`为`null`的槽位，则将数据放入该槽位中，或者往后遍历过程中，遇到了**key值相等**的数据，直接更新即可。
* **第四种情况：** 槽位数据不为空，往后遍历过程中，在找到`Entry`为`null`的槽位之前，遇到`key`过期的`Entry`，**进行探测式数据清理工作**，然后插入槽中。

`ThreadLocalMap`数据清理方式——探测式清理：

* 探测式清理，也就是`expungeStaleEntry`方法，遍历散列数组，从开始位置向后探测清理过期数据，将过期数据的`Entry`设置为`null`，沿途中碰到未过期的数据则将此数据`rehash`后重新在`table`数组中定位，如果定位的位置已经有了数据，则会将未过期的数据放到最靠近此位置的`Entry=null`的桶中，使`rehash`后的`Entry`数据距离正确的桶的位置更近一些。

`ThreadLocalMap`扩容机制

* 在`ThreadLocalMap.set()`方法的最后，如果执行完启发式清理工作后，未清理到任何数据，且当前散列数组中`Entry`的数量已经达到了列表的扩容阈值`(len*2/3)`，就开始执行rehash()扩容。

* 首先是会进行探测式清理工作，从`table`的起始位置往后清理。清理完成之后，`table`中可能有一些`key`为`null`的`Entry`数据被清理掉，所以此时通过判断`size >= threshold - threshold / 4` 也就是`size >= threshold* 3/4` 来决定是否扩容。

* 扩容后的`tab`的大小为`oldLen * 2`，然后遍历老的散列表，重新计算`hash`位置，然后放到新的`tab`数组中，如果出现`hash`冲突则往后寻找最近的`entry`为`null`的槽位，遍历完成之后，`oldTab`中所有的`entry`数据都已经放入到新的`tab`中了。通过`resize()重新`计算`tab`下次扩容的**阈值**。

* `ThreadLocal`常见应用场景有哪些呢？

  * 比如使用SimpleDataFormat的parse()方法，内部有一个Calendar对象，调用SimpleDataFormat的parse()方法会先调用Calendar.clear()，然后调用Calendar.add()，如果一个线程先调用了add()然后另一个线程又调用了clear()，这时候parse()方法解析的时间就不对了，我们就可以用`ThreadLocal<SimpleDataFormat>`，再调用initialValue让每个线程有一个SimpleDataFormat的副本，从而解决了线程安全的问题，也提高了性能。
  * 另一种场景是Spring事务，事务是和线程绑定起来的，Spring框架在事务开始时会给当前线程绑定一个JDBC Connection,在整个事务过程都是使用该线程绑定的connection来执行数据库操作，实现了事务的隔离性。Spring框架里面就是用的ThreadLocal来实现这种隔离，主要是在TransactionSynchronizationManager这个类里面，代码如下所示：

  ```java
  private static final Log logger = LogFactory.getLog(TransactionSynchronizationManager.class);
  
   private static final ThreadLocal<Map<Object, Object>> resources =
     new NamedThreadLocal<>("Transactional resources");
  
   private static final ThreadLocal<Set<TransactionSynchronization>> synchronizations =
     new NamedThreadLocal<>("Transaction synchronizations");
  
   private static final ThreadLocal<String> currentTransactionName =
     new NamedThreadLocal<>("Current transaction name");
  
  ```

《阿里编程规范》对ThreadLocal使用的建议：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210708233208908.png" width="800px"/>
</div>

# 线程池

## 使用线程池使用好处

这里借用《Java 并发编程的艺术》提到的来说一下**使用线程池的好处**：

- **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
- **提高响应速度**。当任务到达时，任务可以不需要的等到线程创建就能立即执行。
- **提高线程的可管理性**。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

## 线程池的处理流程

**为了搞懂线程池的原理，我们需要首先分析一下 `execute`方法**。下面来看看它的源码：

```java
	// 存放线程池的运行状态 (runState) 和线程池内有效线程的数量 (workerCount)
	private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));

    // 任务队列
	private final BlockingQueue<Runnable> workQueue;

	public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();

        int c = ctl.get();							 	// ------ 1
        if (workerCountOf(c) < corePoolSize) {          // ------ 2
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) {	 // ------ 3
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command)) // ------ 4
                reject(command);
            else if (workerCountOf(recheck) == 0)		 // ------ 5
                addWorker(null, false);
        }
        else if (!addWorker(command, false))			 // ------ 6
            reject(command);
    }
```

细化一下，我们可以把`execute()`执行过程分为6步：

1. 获取线程池状态；
2. 如果当前线程池中执行任务数量 < `corePoolSize`核心线程数，通过`addWorker(command, true)`新建一个线程用于执行任务。
3. 如果当前线程池中任务处于运行状态，并且能够成功阻塞队列写入，那么进入第4步。
4. **双重检查**，再次获取线程状态；如果线程状态变了（非运行状态）就需要从阻塞队列移除任务，并尝试判断线程是否全部执行完毕。同时执行拒绝策略。
5. 如果线程池为空，通过`addWorker(command, false)`新建一个线程用于执行任务。
6. 如果在判断第3步就判断没有处于运行状态的线程，或者任务无法成功入队，通过`addWorker(command, false)`新建一个线程用于执行任务。如果队列已 满，就执行拒绝策略。

如官方代码注释所描述，整体上线程池的处理流程分为三步：

1. **线程池判断核心线程池里的线程是否都在执行任务**。如果不是，则创建一个新的工作线程来执行任务。如果核心线程池里的线程都在执行任务，则进入下个流程。
2. **线程池判断工作队列是否已经满**。如果工作队列没有满，则将新提交的任务存储在这个工作队列里。如果工作队列满了，则进入下个流程。
3. **线程池判断线程池的线程是否都处于工作状态**。如果没有，则创建一个新的工作线程来执行任务。如果已经满了，则交给**饱和策略（拒绝策略）**来处理这个任务。

通过下图可以更好的对上面这 3 步做一个展示：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210619144744868.png" width="800px"/>
</div>

### submit()与 execute()方法区别

| 比较         | submit()                                                              | execute()                                     |
| ------------ | --------------------------------------------------------------------- | --------------------------------------------- |
| 有无返回值   | 有返回值，用 Future 封装<br/>根据返回值能判断任务是否被线程池成功执行 | 无返回值<br/>无法判断任务是否被成功线程池执行 |
| 能否捕获异常 | 可在主线程中 get 捕获到                                               | 不能捕获                                      |

## 线程池状态转换

**线程池实现类 `ThreadPoolExecutor` 是 `Executor` 框架最核心的类**。在了解线程池参数之前，我们先来了解一下线程池状态转换。查看`ThreadPoolExecutor`源码，发现定义有如下几类状态：

```java
    // runState is stored in the high-order bits
    private static final int RUNNING    = -1 << COUNT_BITS;
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    private static final int STOP       =  1 << COUNT_BITS;
    private static final int TIDYING    =  2 << COUNT_BITS;
    private static final int TERMINATED =  3 << COUNT_BITS;
```

也就是说，线程池有一下5种状态：

* **RUNNING**：运行状态，指可以接受任务执行队列里的任务，以及对已添加的任务进行处理。
  * 线程池被一旦被创建，就处于RUNNING状态，并且线程池中的任务数为0！
* **SHUTDOWN**：指调用了 `shutdown()` 方法，**不再接受新任务了，但是队列里的任务得执行完毕**。
* **STOP**：指调用了 `shutdownNow()` 方法，**不再接受新任务，同时抛弃阻塞队列里的所有任务并中断所有正在执行任务**。
  * 调用线程池的`shutdownNow()`时，线程池由(RUNNING or SHUTDOWN ) -> STOP。
* **TIDYING**：当所有的任务已执行完毕或终止，`ctl`记录的”任务数量”为0，线程池会变为TIDYING状态。在调用 `shudown()/shutdownNow()` 中都会尝试更新为这个状态。
  * 当线程池在SHUTDOWN状态下，阻塞队列为空并且线程池中执行的任务也为空时，就会由 SHUTDOWN -> TIDYING。当线程池在STOP状态下，线程池中执行的任务为空时，就会由STOP -> TIDYING。。
* **TERMINATED**：终止状态，当执行 `terminated()` 后会更新为这个状态。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210619134641513.png" width="800px"/>
</div>

## 线程池参数

查看`ThreadPoolExecutor`的构造函数：

```java
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        // --- omit ---
    }
```

**`ThreadPoolExecutor` 3 个最重要的参数：**

- **`corePoolSize` :** 线程池的核心线程数量。
- **`maximumPoolSize` :** 线程池的最大线程数（核心线程数 + 非核心线程数 = 最大线程数量）。
- **`workQueue`:** 任务队列，用来储存等待执行任务的队列。

`ThreadPoolExecutor`其他参数：

* **`keepAliveTime`**：当线程数大于核心线程数`corePoolSize`时，多余出来的空闲线程存活的最长时间（多余出来的空闲线程不会立即销毁，而是会等待，直到等待的时间超过了 `keepAliveTime`会被回收销毁）。
* **`unit`** : `keepAliveTime` 参数的时间单位（时、分、秒、毫秒等）。
* **`threadFactory`** ：线程工厂，用来创建线程，一般默认即可。
* **`handler`** ：拒绝策略，当提交的任务过多而不能及时处理时，我们可以定制策略来处理任务。

线程池有两个线程数的设置，一个为核心线程数，一个为最大线程数。如果当前的线程总个数 < `corePoolSize`，那么新建的线程为核心线程，如果当前线程总个数 >= `corePoolSize`，那么新建的线程为非核心线程。 核心线程默认会一直存活下去，即便是空闲状态，但是如果设置了`allowCoreThreadTimeOut(true)`的话，那么核心线程空闲时间达到`keepAliveTime`也将关闭。

在创建完线程池之后，默认情况下，线程池中并没有任何线程，**等到有任务**来才创建线程去执行任务。但有一种情况排除在外，就是调用 `prestartAllCoreThreads()` 或者 `prestartCoreThread()` 方法的话，可以提前创建等于核心线程数的线程数量，这种方式被称为**预热**，在抢购系统中就经常被用到。

当创建的线程数等于 `corePoolSize` 时，提交的任务会被加入到设置的阻塞队列中。当队列满了，会创建线程执行任务，直到线程池中的数量等于 `maximumPoolSize`。

当线程数量已经等于 `maximumPoolSize` 时， 新提交的任务无法加入到等待队列，也无法创建非核心线程直接执行，我们又没有为线程池设置拒绝策略，这时线程池就会抛出 `RejectedExecutionException` 异常，即线程池拒绝接受这个任务。

当线程池中创建的线程数量超过设置的 `corePoolSize`，在某些线程处理完任务后，如果等待 `keepAliveTime` 时间后仍然没有新的任务分配给它，那么这个线程将会被回收。线程池回收线程时，会对所谓的“核心线程”和“非核心线程”一视同仁，直到线程池中线程的数量等于设置的 `corePoolSize` 参数，回收过程才会停止。

### 线程池拒绝策略

**`ThreadPoolExecutor` 拒绝策略定义：**如果线程池中所有的线程都在忙碌，并且工作队列也满了（前提是工作队列是有界队列），那么此时提交任务，线程池就会拒绝接收。至于拒绝的策略，你可以通过 handler 这个参数来指定。

`ThreadPoolExecutor` 已经提供了以下 4 种策略。

* **`CallerRunsPolicy`**：提交任务的线程自己去执行该任务。也就是说，直接在调用`execute`方法的线程中运行(`run`)被拒绝的任务，如果执行程序已关闭，则会丢弃该任务。这种策略会降低对于新任务提交速度，影响程序的整体性能。如果您的应用程序可以承受此延迟并且你要求任何一个任务请求都要被执行的话，你可以选择这个策略。
* **`AbortPolicy`**：**默认的拒绝策略**，会 `throws RejectedExecutionException`拒绝新任务的处理。
* **`DiscardPolicy`**：直接丢弃任务，没有任何异常抛出。
* **`DiscardOldestPolicy`**：丢弃最老的任务，其实就是把最早进入工作队列的任务丢弃，然后把新任务加入到工作队列。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210619150145807.png" width="600px"/>
</div>

### 缓冲队列

`BlockingQueue` 是双缓冲队列。`BlockingQueue` 允许两个线程同时向队列一个存储，一个取出操作。在保证并发安全的同时，提高了队列的存取效率。常用的队列如下

* **`ArrayBlockingQueue`**：规定大小的 `BlockingQueue`，其构造必须指定大小。其所含的对象是 FIFO 顺序排序的。

* **`LinkedBlockingQueue`**：大小不固定的 `BlockingQueue`，若其构造时指定大小，生成的`BlockingQueue` 有大小限制，不指定大小，其大小有 `Integer.MAX_VALUE` 来决定。其所含的对象是 FIFO 顺序排序的。

* **`PriorityBlockingQueue`**：类似于 `LinkedBlockingQueue`，但是其所含对象的排序不是 FIFO，而是依据对象的自然顺序或者构造函数的 Comparator 决定。
* **`SynchronousQueue`**：特殊的 `BlockingQueue`，对其的操作必须是放和取交替完成，它支持公平访问队列。

## 线程池为什么不直接Executors创建线程池

在`java.util.concurrent.Executors`线程工厂类里面提供了一些静态工厂，实现了以下4种类型的 `ThreadPoolExecutor`：

| 类型                      | 特性                                                                                                                                                                                                           |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `newSingleThreadExecutor` | 创建一个单线程的线程池。这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。 |
| `newFixedThreadPool`      | 创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。                       |
| `newCachedThreadPool`     | 创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。                                   |
| `newScheduledThreadPool`  | 创建一个大小无限的线程池，此线程池支持定时以及周期性执行任务的需求。                                                                                                                                           |

在生产环境下的实际场景中，一般不太推荐使用它们。因为选择使用 Executors 提供的工厂类实现的五种线程池，将会忽略很多线程池的参数设置，工厂类一旦选择设置默认参数，就很容易导致无法调优参数设置，从而产生性能问题或者资源浪费。

比如，`SingleThreadExecutor`与`FixedThreadPool`都使用无界队列 `LinkedBlockingQueue`（队列的容量为 `Intger.MAX_VALUE`）作为线程池的工作队列，可能会导致 OOM。`CachedThreadPool`使用了`SynchronousQueue`，但是允许创建的线程数量为 `Integer.MAX_VALUE` ，可能会创建大量线程，而导致 OOM。

## 线程池线程回收策略

线程池中线程的销毁依赖JVM自动的回收，线程池做的工作是根据当前线程池的状态维护一定数量的线程引用，防止这部分线程被JVM回收，当线程池决定哪些线程需要回收时，只需要将其引用消除即可。Worker被创建出来后，就会不断地进行轮询，然后获取任务去执行，核心线程可以无限等待获取任务，非核心线程要限时获取任务。当Worker无法获取到任务，也就是获取的任务为空时，循环会结束，Worker会主动消除自身在线程池内的引用。

```Java
try {
  while (task != null || (task = getTask()) != null) {
    //执行任务
  }
} finally {
  processWorkerExit(w, completedAbruptly);//获取不到任务时，主动回收自己
}
```

线程回收的工作是在`processWorkerExit`方法完成的。

<div align="center">  
<img src="https://p0.meituan.net/travelcube/90ea093549782945f2c968403fdc39d415386.png" width="800px"/>
</div>


事实上，在这个方法中，将线程引用移出线程池就已经结束了线程销毁的部分。但由于引起线程销毁的可能性有很多，线程池还要判断是什么引发了这次销毁，是否要改变线程池的现阶段状态，是否要根据新状态，重新分配线程。

## 线程池数量计算: CPU密集型 VS I/O密集型

**CPU 密集型任务**：这种任务消耗的主要是 CPU 资源，可以将线程数设置为 **N（CPU 核心数）+1**，比 CPU 核心数多出来的一个线程是为了防止线程偶发的缺页中断，或者其它原因导致的任务暂停而带来的影响。一旦任务暂停，CPU 就会处于空闲状态，而在这种情况下多出来的一个线程就可以充分利用 CPU 的空闲时间。

**I/O 密集型任务**：这种任务应用起来，系统会用大部分的时间来处理 I/O 交互，而线程在处理 I/O 的时间段内不会占用 CPU 来处理，这时就可以将 CPU 交出给其它线程使用。因此在 I/O 密集型任务的应用中，我们可以多配置一些线程，具体的计算方法是 **2N**。

**如何判断是 CPU 密集任务还是 IO 密集任务？**

CPU 密集型简单理解就是利用 CPU 计算能力的任务比如你在内存中对大量数据进行排序。单凡涉及到网络读取，文件读取这类都是 IO 密集型，这类任务的特点是 CPU 计算耗费时间相比于等待 IO 操作完成的时间来说很少，大部分时间都花在了等待 IO 操作完成上。