- [1 AQS 概述](#1-aqs-概述)
- [2 AQS 原理](#2-aqs-原理)
  - [2.1 同步队列](#21-同步队列)
  - [2.2 同步状态](#22-同步状态)
    - [2.2.1 独占式(EXCLUSIVE)](#221-独占式exclusive)
    - [2.2.2 共享式(SHARED)](#222-共享式shared)
    - [2.2.3 超时获取方式](#223-超时获取方式)
  - [2.3 模板方法](#23-模板方法)
- [3 Semaphore(信号量)](#3-semaphore信号量)
- [4 CountDownLatch (倒计时器)](#4-countdownlatch-倒计时器)
  - [4.1 概述](#41-概述)
  - [4.2 应用场景](#42-应用场景)
- [5 CyclicBarrier(循环栅栏)](#5-cyclicbarrier循环栅栏)
  - [5.1 概述](#51-概述)
  - [5.2 源码分析](#52-源码分析)
  - [5.3 应用场景](#53-应用场景)
  - [5.4 CyclicBarrier和CountDownLatch的区别](#54-cyclicbarrier和countdownlatch的区别)
- [参考](#参考)
## 1 AQS 概述

**AQS** 的全称为（AbstractQueuedSynchronizer），中文即“**队列同步器**”，这个类放在 java.util.concurrent.locks 包下面。

![](https://img-blog.csdnimg.cn/20201103161410728.png)

AQS是用来构建锁或者其他同步组件的基础框架，它使用了一个int成员变量表示同步状态，通过内置的FIFO队列来完成资源获取线程的排队工作。使用 AQS 能简单且高效地构造出应用广泛的大量的同步器，比如上篇文章写的ReentrantLock与ReentrantReadWriteLock。除此之外，AQS还能构造出Semaphore，FutureTask(jdk1.7) 等同步器。

## 2 AQS 原理

### 2.1 同步队列

**AQS 是依赖 CLH 队列锁来完成同步状态的管理**。当前线程获取同步状态失败时，同步器会将当前线程以及等待状态等信息构建为一个**节点(Node)**并将其加入同步队列，同步会阻塞当前线程，当同步状态释放时，会将首节点中的线程唤醒，使其再次尝试获取同步状态。

> CLH(Craig,Landin,and Hagersten)队列是一个虚拟的双向队列（**FIFO双向队列**）（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。**AQS 是将每条请求共享资源的线程封装成一个 CLH 锁队列的一个结点（Node）来实现锁的分配**。

同步队列中的节点（Node）用来保存获取同步状态失败的线程引用、等待状态以及前驱和后继节点信息。

| 属性类型与名称 | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| int waitStatus | 等待状态(如CANCELLED=1、SIGNAL=-1、CONDITION=-2、PROPAGATE=-3、INITIAL=0) |
| Node prev      | 前驱节点(当节点加入同步队列时被设置，在尾部添加)             |
| Node next      | 后继节点                                                     |
| Thread thread  | 当前获取同步状态的线程                                       |

节点源码如下：

```java
static final class Node {
    // 表示该节点等待模式为共享式，通常记录于nextWaiter，
    // 通过判断nextWaiter的值可以判断当前结点是否处于共享模式
    static final Node SHARED = new Node();
    // 表示节点处于独占式模式，与SHARED相对
    static final Node EXCLUSIVE = null;
    // waitStatus的不同状态
    // 当前结点是因为超时或者中断取消的，进入该状态后将无法恢复
    static final int CANCELLED =  1;
    // 当前结点的后继结点是(或者将要)由park导致阻塞的，当结点被释放或者取消时，需要通过unpark唤醒后继结点
    static final int SIGNAL    = -1;
    // 表明结点在等待队列中，结点线程等待在Condition上
    // 当其他线程对Condition调用了signal()方法时，会将其加入到同步队列中   
    static final int CONDITION = -2;
    // 下一次共享式同步状态的获取将会无条件地向后继结点传播
    static final int PROPAGATE = -3;
    volatile int waitStatus;
    // 记录前驱结点
    volatile Node prev;
    // 记录后继结点
    volatile Node next;
    // 记录当前的线程
    volatile Thread thread;
    // 用于记录共享模式(SHARED), 也可以用来记录CONDITION队列
    Node nextWaiter;
    // 通过nextWaiter的记录值判断当前结点的模式是否为共享模式
    final boolean isShared() {	return nextWaiter == SHARED;}
    // 获取当前结点的前置结点
    final Node predecessor() throws NullPointerException { ... }
    // 用于初始化时创建head结点或者创建SHARED结点
    Node() {}
    // 在addWaiter方法中使用，用于创建一个新的结点
    Node(Thread thread, Node mode) {     
        this.nextWaiter = mode;
        this.thread = thread;
    }
	// 在CONDITION队列中使用该构造函数新建结点
    Node(Thread thread, int waitStatus) { 
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
// 记录头结点
private transient volatile Node head;
// 记录尾结点
private transient volatile Node tail;
```

节点是构成同步队列的基础，同步器拥有**首节点（Head）和尾节点（Tail）**，没有成功**获取**同步状态的线程将会成为节点加入该队列的尾部。同步器提供了一个基于CAS的设置尾节点的方法：`compareAndSetTail(Node expect, Node update)`，它需要传递当前线程“认为”的尾节点和当前节点，只有设置成功后，当前节点才正式与之前的尾节点建立关联。

![](https://img-blog.csdnimg.cn/20201104225437655.png)

**首节点是获取同步状态成功的节点**，首节点的线程在**释放**同步状态时，将会唤醒后继节点，而后继节点将会在获取同步状态成功时将自己设置为首节点。设置首节点是通过获取同步状态成功的线程来完成的，不需要使用CAS来保证，只需将首节点设置成为原首节点的后继节点并断开原首节点的next引用即可。

![](https://img-blog.csdnimg.cn/20201104225709623.png)

### 2.2 同步状态

#### 2.2.1 独占式(EXCLUSIVE)

独占式(EXCLUSIVE)获取需重写`tryAcquire`、`tryRelease`方法，并访问`acquire`、`release`方法实现相应的功能。

```java
    public final void acquire(int arg) {
        // 如果线程直接获取成功，或者再尝试获取成功后都是直接工作，
        // 如果是从阻塞状态中唤醒开始工作的线程，将当前的线程中断        
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
	// 封装线程，新建结点并加入到同步队列中
    private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);
        Node pred = tail;
        // 尝试入队， 成功返回
        if (pred != null) {
            node.prev = pred;
            // CAS操作设置队尾
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        // 通过CAS操作自旋完成node入队操作
        enq(node);
        return node;
    }
	// 在同步队列中等待获取同步状态
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            // 自旋
            for (;;) {
                final Node p = node.predecessor();
                // 前驱节点是否为头节点&&tryAcquire获取同步状态
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null;
                    failed = false;
                    return interrupted;
                }
                // 获取不到同步状态，将前置结点标为SIGNAL状态并且通过park操作将Node封装的线程阻塞
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                // 如果获取失败，将node标记为CANCELLED
                cancelAcquire(node);
        }
    }
```

独占式获取同步状态流程：

![](https://img-blog.csdnimg.cn/20201104234304168.png)

通过调用同步器的`release(int arg)`方法可以释放同步状态，该方法在释放了同步状态之后，会唤醒其**后继节点**（进而使后继节点重新尝试获取同步状态）。

```java
public final boolean release(int arg) {
    // 首先尝试释放并更新同步状态
    if (tryRelease(arg)) {
        Node h = head;
        // 检查是否需要唤醒后置结点
        if (h != null && h.waitStatus != 0)
            // 唤醒后置结点
            unparkSuccessor(h);
        return true;
    }
    return false;
}
// 唤醒后继结点
private void unparkSuccessor(Node node) {
    int ws = node.waitStatus;
    // 通过CAS操作将waitStatus更新为0
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);
    Node s = node.next;
    // 检查后置结点，若为空或者状态为CANCELLED，找到后置非CANCELLED结点
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    // 唤醒后继结点
    if (s != null)
        LockSupport.unpark(s.thread);
}
```

#### 2.2.2 共享式(SHARED)

**共享式获取与独占式获取最主要的区别在于同一时刻能否有多个线程同时获取到同步状态**。

共享式(SHARED)获取需重写`tryAcquireShared`、`tryReleaseShared`方法，并访问`acquireShared`、`releaseShared`方法实现相应的功能。与独占式相对，共享式支持多个线程同时获取到同步状态并进行工作，如 Semaphore、CountDownLatch、 CyclicBarrier等。ReentrantReadWriteLock 可以看成是组合式，因为 ReentrantReadWriteLock 也就是读写锁允许多个线程同时对某一资源进行读。

```java
public final void acquireShared(int arg) {
    // 尝试共享式获取同步状态，如果成功获取则可以继续执行，否则执行doAcquireShared
    if (tryAcquireShared(arg) < 0)
        // 以共享式不停得尝试获取同步状态
        doAcquireShared(arg);
}

private void doAcquireShared(int arg) {
    // 向同步队列中新增一个共享式的结点
    final Node node = addWaiter(Node.SHARED);
    // 标记获取失败状态
    boolean failed = true;
    try {
        // 标记中断状态(若在该过程中被中断是不会响应的，需要手动中断)
        boolean interrupted = false;
        // 自旋
        for (;;) {
            // 获取前置结点
            final Node p = node.predecessor();
            // 若前置结点为头结点
            if (p == head) {
                // 尝试获取同步状态
                int r = tryAcquireShared(arg);
                // 若获取到同步状态。
                if (r >= 0) {
                    // 此时，当前结点存储的线程恢复执行，需要将当前结点设置为头结点并且向后传播，
                    // 通知符合唤醒条件的结点一起恢复执行
                    setHeadAndPropagate(node, r);
                    p.next = null;
                    // 需要中断，中断当前线程
                    if (interrupted)
                        selfInterrupt();
                    // 获取成功
                    failed = false;
                    return;
                }
            }
            // 获取同步状态失败，需要进入阻塞状态
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        // 获取失败，CANCELL node
        if (failed)
            cancelAcquire(node);
    }
}
// 将node设置为同步队列的头结点，并且向后通知当前结点的后置结点，完成传播
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head; 
    setHead(node);
	// 向后传播
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {
        Node s = node.next;
        if(s == null || s.isShared())
            doReleaseShared();
    }
}
```

与独占式一样，共享式获取也需要释放同步状态，通过调用releaseShared(intarg)方法可以释放同步状态，释放同步状态成功后，会唤醒后置结点，并且保证传播性。

```java
public final boolean releaseShared(int arg) {
    // 尝试释放同步状态
    if (tryReleaseShared(arg)) {
        // 成功后唤醒后置结点
        doReleaseShared();
        return true;
    }
    return false;
}
// 唤醒后置结点
private void doReleaseShared() {
    // 循环的目的是为了防止新结点在该过程中进入同步队列产生的影响，同时要保证CAS操作的完成
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;            
                    unparkSuccessor(h);
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                
        }
        if (h == head)                   
            break;
    }
}
```

#### 2.2.3 超时获取方式

通过调用同步器的`doAcquireNanos(int arg, long nanosTimeout)`方法可以**超时获取同步状态**，即在指定的时间段内获取同步状态，如果获取到同步状态则返回true，否则，返回false。该方法提供了传统Java同步操作（比如synchronized关键字）所不具备的特性。

```java
private boolean doAcquireNanos(int arg, long nanosTimeout)
        throws InterruptedException {
    if (nanosTimeout <= 0L)
        return false;
    // 计算超时的时间=当前虚拟机的时间+设置的超时时间
    final long deadline = System.nanoTime() + nanosTimeout;
    // 调用addWaiter将当前线程封装成独占模式的节点，并且加入到同步队列尾部。
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        // 自旋
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                // 如果当前节点的前驱节点为头结点，则让当前节点去尝试获取锁。
                setHead(node);
                p.next = null; 
                failed = false;
                return true;
            }
            // 如果当前节点的前驱节点不是头结点，或当前节点获取锁失败，
            // 则再次判断当前线程是否已经超时。
            nanosTimeout = deadline - System.nanoTime();
            if (nanosTimeout <= 0L)
                return false;
            // 调用shouldParkAfterFailedAcquire方法，告诉当前节点的前驱节点，马上进入
            // 等待状态了，即做好进入等待状态前的准备。
            if (shouldParkAfterFailedAcquire(p, node) &&
                nanosTimeout > spinForTimeoutThreshold)
                // 调用LockSupport.parkNanos方法，将当前线程设置成超时等待的状态。
                LockSupport.parkNanos(this, nanosTimeout);
            if (Thread.interrupted())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

由上面代码可知，超时获取也是调用addWaiter将当前线程封装成**独占模式**的节点，并且加入到同步队列尾部。

超时获取与独占式获取同步状态区别**在于获取同步状态失败后的处理**。如果当前线程获取同步状态失败，则判断是否超时（nanosTimeout小于等于0表示已经超时）；如果没有超时，重新计算超时间隔nanosTimeout，然后使当前线程等待nanosTimeout纳秒（当已到设置的超时时间，该线程会从`LockSupport.parkNanos(Object blocker, long nanos)`方法返回）。

独占式超时获取同步状态流程：

![](https://img-blog.csdnimg.cn/20201105002205786.png)

### 2.3 模板方法

AQS 使用一个 int 成员变量来表示同步状态，通过内置的 FIFO 队列来完成获取资源线程的排队工作。AQS 使用 CAS 对该同步状态进行原子操作实现对其值的修改。

```java
private volatile int state;// 共享变量，使用volatile修饰保证线程可见性
```

同步状态`state`通过 protected 类型的`getState`，`setState`，`compareAndSetState`方法进行操作

```java
// 返回同步状态的当前值
protected final int getState() {
        return state;
}
// 设置同步状态的值
protected final void setState(int newState) {
        state = newState;
}
// CAS更新同步状态，该方法能够保证状态设置的原子性
protected final boolean compareAndSetState(int expect, int update) {
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

同步器的设计是基于模板方法模式的，也就是说，使用者需要继承同步器并重写指定的方法，随后将同步器组合在自定义同步组件的实现中，并调用同步器提供的模板方法，而这些模板方法将会调用使用者重写的方法。

**自定义同步器时需要重写下面几个 AQS 提供的模板方法：**

```java
isHeldExclusively()// 该线程是否正在独占资源。只有用到condition才需要去实现它。
tryAcquire(int)// 独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)// 独占方式。尝试释放资源，成功则返回true，失败则返回false。
tryAcquireShared(int)// 共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)// 共享方式。尝试释放资源，成功则返回true，失败则返回false。
```

同步器提供的模板方法基本上分为3类：**独占式**获取与释放同步状态、**共享式**获取与释放同步状态和查询同步队列中的等待线程情况。

一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现`tryAcquire-tryRelease`、`tryAcquireShared-tryReleaseShared`中的一种即可。

以 ReentrantLock 为例，state 初始化为 0，表示未锁定状态。A 线程 lock()时，会调用 tryAcquire()独占该锁并将 state+1。此后，其他线程再 tryAcquire()时就会失败，直到 A 线程 unlock()到 state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A 线程自己是可以重复获取此锁的（state 会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证 state 是能回到零态的。

再以 CountDownLatch 以例，任务分为 N 个子线程去执行，state 也初始化为 N（注意 N 要与线程个数一致）。这 N 个子线程是并行执行的，每个子线程执行完后 countDown()一次，state 会 CAS(Compare and Swap)减 1。等到所有子线程都执行完后(即 state=0)，会 unpark()主调用线程，然后主调用线程就会从 await()函数返回，继续后续动作。

但 AQS 也支持自定义同步器同时实现独占和共享两种方式，如`ReentrantReadWriteLock`。

下面就来学习几个常用的并发同步工具。

## 3 Semaphore(信号量)

Semaphore（信号量）用来控制 **同时访问特定资源的线程数量**，它通过协调各个线程，以保证合理的使用公共资源。synchronized 和 ReentrantLock 都是一次只允许一个线程访问某个资源，而**Semaphore(信号量)可以指定多个线程同时访问某个资源**。

以停车场为例。假设一个停车场只有10个车位，这时如果同时来了15辆车，则只允许其中10辆不受阻碍的进入。剩下的5辆车则必须在入口等待，此后来的车也都不得不在入口处等待。这时，如果有5辆车离开停车场，放入5辆；如果又离开2辆，则又可以放入2辆，如此往复。

在这个停车场系统中，车位即是共享资源，每辆车就好比一个线程，信号量就是空车位的数目。

Semaphore中包含了一个实现了AQS的同步器Sync，以及它的两个子类FairSync和NonFairSync。查看Semaphore类结构：

![](https://img-blog.csdnimg.cn/20201105094613463.png)

可见Semaphore也是区分公平模式和非公平模式的。

- **公平模式：** 调用 acquire 的顺序就是获取许可证的顺序，遵循 FIFO。
- **非公平模式：** 抢占式的。

Semaphore 对应的两个构造方法如下：

```java
   public Semaphore(int permits) {
        sync = new NonfairSync(permits);
    }

    public Semaphore(int permits, boolean fair) {
        sync = fair ? new FairSync(permits) : new NonfairSync(permits);
    }
```

**这两个构造方法，都必须提供许可的数量，第二个构造方法可以指定是公平模式还是非公平模式，默认非公平模式。**

Semaphore实现原理这里就不分析了，可以参考[死磕 java同步系列之Semaphore源码解析](https://juejin.im/post/6844903866329202701)这篇文章。

需要明白的是，**Semaphore也是共享锁的一种实现**。它默认构造AQS的state为permits。当执行任务的线程数量超出permits，那么多余的线程将会被放入阻塞队列Park，并自旋判断state是否大于0。只有当state大于0的时候，阻塞的线程才能继续执行，此时先前执行任务的线程继续执行release方法，release方法使得state的变量会加1，那么自旋的线程便会判断成功。如此，每次只有最多不超过permits数量的线程能自旋成功，便限制了执行任务线程的数量。

Semaphore常用于做**流量控制**，特别是公用资源有限的应用场景。

| 常用方法                                                     | 描述                                                  |
| ------------------------------------------------------------ | ----------------------------------------------------- |
| acquire()/acquire(int permits)                               | 获取许可证。获取许可失败，会进入AQS的队列中排队。     |
| tryAcquire()/tryAcquire(int permits)                         | 获取许可证。获取许可失败，直接返回false。             |
| tryAcquire(long timeout, TimeUnit unit)/<br>tryAcquire(int permits, long timeout, TimeUnit unit) | 超时等待获取许可证。                                  |
| release()                                                    | 归还许可证。                                          |
| intavailablePermits()                                        | 返回此信号量中当前可用的许可证数。                    |
| intgetQueueLength()                                          | 返回正在等待获取许可证的线程数。                      |
| booleanhasQueuedThreads()                                    | 是否有线程正在等待获取许可证。                        |
| void reducePermits（int reduction）                          | 减少reduction个许可证，是个protected方法。            |
| Collection getQueuedThreads()                                | 返回所有等待获取许可证的线程集合，是个protected方法。 |

使用示例：

```java
public class SemaphoreTest {
    private static final int THREAD_COUNT = 50;

    public static void main(String[] args) throws InterruptedException {
        // 创建一个具有固定线程数量的线程池对象
        ExecutorService threadPool = Executors.newFixedThreadPool(THREAD_COUNT);
        // 一次只能允许执行的线程数量
        final Semaphore semaphore = new Semaphore(10);

        for (int i = 0; i < THREAD_COUNT; i++) {
            final int threadNum = i;
            threadPool.execute(() -> {
                try {
                    semaphore.acquire();// 获取1个许可，所以可运行线程数量为10/1=10
                    test(threadNum);
                    semaphore.release();// 释放1个许可，所以可运行线程数量为10/1=10
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
        threadPool.shutdown();
    }

    public static void test(int threadNum) throws InterruptedException {
        Thread.sleep(1000);// 模拟请求的耗时操作
        System.out.println("threadNum:" + threadNum);
        Thread.sleep(1000);// 模拟请求的耗时操作
    }
}
```

在代码中，虽然有50个线程在执行，但是只允许10个并发执行。Semaphore的构造方法Semaphore（int permits）接受一个整型的数字，表示可用的许可证数量。Semaphore（10）表示允许10个线程获取许可证，也就是最大并发数是10。Semaphore的用法也很简单，首先线程使用Semaphore的acquire()方法获取一个许可证，使用完之后调用release()方法归还许可证。

除了 `acquire`方法之外，另一个比较常用的与之对应的方法是`tryAcquire`方法，该方法如果获取不到许可就立即返回 false。

## 4 CountDownLatch (倒计时器)

### 4.1 概述

在日常开发中经常会遇到需要在主线程中开启多个线程去并行执行任务，并且主线程需要等待所有子线程执行完毕后再进行汇总的场景。jdk 1.5之前一般都使用线程的join()方法来实现这一点，但是join方法不够灵活，难以满足不同场景的需要，所以jdk 1.5之后concurrent包提供了CountDownLatch这个类。

**CountDownLatch**是一种同步辅助工具，它**允许一个或多个线程等待其他线程完成操作**。

CountDownLatch是通过一个计数器来实现的，计数器的初始化值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就相应得减1。当计数器到达0时，表示所有的线程都已完成任务，然后在闭锁上等待的线程就可以恢复执行任务。

![](https://img-blog.csdnimg.cn/20201105140506223.png)

CountDownLatch的方法：

| 方法                               | 描述                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| await()                            | 调用该方法的线程等到构造方法传入的 N 减到 0 的时候，才能继续往下执行。 |
| await(long timeout, TimeUnit unit) | 调用该方法的线程等到指定的 timeout 时间后，不管 N 是否减至为 0，都会继续往下执行。 |
| countDown()                        | 使 CountDownLatch 初始值 N 减 1。                            |
| getCount()                         | 获取当前 CountDownLatch 维护的值，也就是AQS的state的值。     |

CountDownLatch的实现原理，可以查看 [【JUC】JDK1.8源码分析之CountDownLatch（五）](https://www.cnblogs.com/leesf456/p/5406191.html)一文。

根据源码分析可知，**CountDownLatch是AQS中共享锁的一种实现**。AbstractQueuedSynchronizer中维护了一个volatile类型的整数state，volatile可以保证多线程环境下该变量的修改对每个线程都可见，并且由于该属性为整型，因而对该变量的修改也是原子的。

CountDownLatch默认构造 AQS 的 state 值为 count。创建一个CountDownLatch对象时，所传入的整数N就会赋值给state属性。

当调用countDown()方法时，其实是调用了`tryReleaseShared`方法以CAS的操作来对state减1；而调用await()方法时，当前线程就会判断state属性是否为0。如果为0，阻塞线程被唤醒继续往下执行；如果不为0，则使当前线程放入阻塞队列Park，直至最后一个线程调用了countDown()方法使得state == 0，再唤醒在await()方法中等待的线程。

**特别注意的是**：

**CountDownLatch 是一次性的**，计数器的值只能在构造方法中初始化一次，之后没有任何机制再次对其设置值，当 CountDownLatch 使用完毕后，它不能再次被使用。如果**需要能重置计数，可以使用CyclicBarrier**。

### 4.2 应用场景

CountDownLatch主要应用场景：

1. **实现最大的并行性**：同时启动多个线程，实现最大程度的并行性。例如110跨栏比赛中，所有运动员准备好起跑姿势，进入到预备状态，等待裁判一声枪响。裁判开了枪，所有运动员才可以开跑。
2. **开始执行前等待N个线程完成各自任务**：例如一群学生在教室考试，学生们都完成了作答，老师才可以进行收卷操作。

案例：

```java
public class CountDownLatchTest {
    private static final int THREAD_COUNT = 30;

    public static void main(String[] args) throws InterruptedException {
        ExecutorService threadPool = Executors.newFixedThreadPool(10);

        final CountDownLatch countDownLatch = new CountDownLatch(THREAD_COUNT);
        for (int i = 0; i < THREAD_COUNT; i++) {
            final int threadNum = i;
            threadPool.execute(() -> {
                try {
                    Thread.sleep(1000);// 模拟请求的耗时操作
                    System.out.println("子线程:" + threadNum);
                    Thread.sleep(1000);// 模拟请求的耗时操作
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    countDownLatch.countDown();// 表示一个请求已经被完成
                }
            });
        }
        System.out.println("主线程启动...");
        countDownLatch.await();
        threadPool.shutdown();
        System.out.println("子线程执行完毕...");
        System.out.println("主线程执行完毕...");
    }
}
```

上面的代码中，我们定义了请求的数量为30，当这 30 个请求被处理完成之后，才会打印`子线程执行完毕`。

主线程在启动其他线程后调用 `CountDownLatch.await()` 方法进入阻塞状态，直到其他线程完成各自的任务才被唤醒。

开启的30个线程必须引用闭锁对象，因为他们需要通知 `CountDownLatch` 对象，他们已经完成了各自的任务。这种通知机制是通过 `CountDownLatch.countDown()`方法来完成的；每调用一次这个方法，在构造函数中初始化的 count 值就减 1。所以当30个线程都调用了这个方法后，count 的值才等于0，然后主线程就能通过 `await()`方法，继续执行自己的任务。

## 5 CyclicBarrier(循环栅栏)

### 5.1 概述

**CyclicBarrier的字面意思是可循环使用（Cyclic）的屏障（Barrier）**。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续运行。CyclicBarrier 的功能和应用场景与CountDownLatch都非常类似。

![](https://img-blog.csdnimg.cn/20201105150659614.png)

CyclicBarrier常用方法：

| 常用方法                           | 描述                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| await()                            | 在所有线程都已经在此 barrier上并调用 await 方法之前，将一直等待。 |
| await(long timeout, TimeUnit unit) | 所有线程都已经在此屏障上调用 await 方法之前将一直等待，或者超出了指定的等待时间。 |
| getNumberWaiting()                 | 返回当前在屏障处等待的线程数目。                             |
| getParties()                       | 返回要求启动此 barrier 的线程数目。                          |
| isBroken()                         | 查询此屏障是否处于损坏状态。                                 |
| reset()                            | 将屏障重置为其初始状态。                                     |

### 5.2 源码分析

构造函数：

```java
// 创建一个新的 CyclicBarrier，它将在给定数量的参与者（线程）处于等待状态时启动，但它不会在启动 barrier 时执行预定义的操作。
public CyclicBarrier(int parties) {
    this(parties, null);
}

// 创建一个新的 CyclicBarrier，它将在给定数量的参与者（线程）处于等待状态时启动，并在启动 barrier 时执行给定的屏障操作。
// 该操作由最后一个进入 barrier 的线程执行。
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}
```

其中，**parties 就表示屏障拦截的线程数量**。

 CyclicBarrier 的最重要的方法就是 await 方法，`await()` 方法就像树立起一个栅栏的行为一样，将线程挡住了，**当拦住的线程数量达到 parties 的值时，栅栏才会打开，线程才得以通过执行**。

```java
    public int await() throws InterruptedException, BrokenBarrierException {
        try {
            return dowait(false, 0L);
        } catch (TimeoutException toe) {
            throw new Error(toe);
        }
    }
```

当调用 `await()` 方法时，实际上调用的是`dowait(false, 0L)`方法。查看`dowait(boolean timed, long nanos)`：

```java
    // 当线程数量或者请求数量达到 count 时 await 之后的方法才会被执行。
    private int count;

    private int dowait(boolean timed, long nanos)
        throws InterruptedException, BrokenBarrierException,
               TimeoutException {
        final ReentrantLock lock = this.lock;
        // 获取”独占锁“
        lock.lock();
        try {
            // 保存“当前的generation”
            final Generation g = generation;
			// 如果当前代损坏，抛出异常
            if (g.broken)
                throw new BrokenBarrierException();
            // 如果线程中断，抛出异常
            if (Thread.interrupted()) {
                // 将损坏状态设置为 true，并唤醒所有阻塞在此栅栏上的线程
                breakBarrier();
                throw new InterruptedException();
            }
            // 将“count计数器”-1
            int index = --count;
            // 当 count== 0，说明最后一个线程已经到达栅栏
            if (index == 0) {
                boolean ranAction = false;
                try {
                    final Runnable command = barrierCommand;
                    // 执行栅栏任务
                    if (command != null)
                        command.run();
                    ranAction = true;
                    // 将 count 重置为 parties 属性的初始化值
                    // 唤醒之前等待的线程，并更新generation。
                    nextGeneration();
                    // 结束，等价于return index
                    return 0;
                } finally {
                    if (!ranAction)
                        breakBarrier();
                }
            }

            // 当前线程一直阻塞，直到“有parties个线程到达barrier” 或 “当前线程被中断” 或 “超时”这3者条件之一发生
            // 当前线程才继续执行。
            for (;;) {
                try {
                    // 如果没有时间限制，则直接等待，直到被唤醒。
                    if (!timed)
                        trip.await();
                    // 如果有时间限制，则等待指定时间再唤醒(超时等待)。
                    else if (nanos > 0L)
                        nanos = trip.awaitNanos(nanos);
                } catch (InterruptedException ie) {
                    // 当前代没有损坏
                    if (g == generation && ! g.broken) {
                        // 让栅栏失效
                        breakBarrier();
                        throw ie;
                    } else {
                        // 上面条件不满足,说明这个线程不是这代的。
                        // 就不会影响当前这代栅栏执行逻辑。中断。
                        Thread.currentThread().interrupt();
                    }
                }
				// 如果“当前generation已经损坏”，则抛出异常。
                if (g.broken)
                    throw new BrokenBarrierException();  
				// 如果“generation已经换代”，则返回index。
                if (g != generation)
                    return index; 
                // 如果是“超时等待”，并且时间已到，则通过breakBarrier()终止CyclicBarrier
                // 唤醒CyclicBarrier中所有等待线程。
                if (timed && nanos <= 0L) {
                    breakBarrier();
                    throw new TimeoutException();
                }
            }
        } finally {
            // 释放“独占锁(lock)”
            lock.unlock();
        }
    }
```

generation是CyclicBarrier的一个成员变量：

```java
   /**
     * Generation一代的意思。
     * CyclicBarrier是可以循环使用的，用它来标志本代和下一代。
     * broken：当前代是否损坏的标志。标志有线程发生了中断，或者异常，就是任务没有完成。
     */
    private static class Generation {
        boolean broken = false;
    }

    // 实现独占锁
    private final ReentrantLock lock = new ReentrantLock();
    // 实现多个线程之间相互等待通知，就是满足某些条件之后，线程才能执行，否则就等待
    private final Condition trip = lock.newCondition();
    // 初始化时屏障数量 
    private final int parties;
    // 当条件满足(即屏障数量为0)之后，会回调这个Runnable
    private final Runnable barrierCommand;
    //当前代
    private Generation generation = new Generation();

    // 剩余的屏障数量count
    private int count;
```

在CyclicBarrier中，**同一批的线程属于同一代**，即同一个generation；CyclicBarrier中通过generation对象，记录属于哪一代。
当有parties个线程到达barrier，generation就会被更新换代。

**总结**：

1. `CyclicBarrier` 内部通过一个 count 变量作为计数器，cout 的初始值为 parties 属性的初始化值，每当一个线程到了栅栏，那么就将计数器减1。如果 count 值为 0 了，表示这是这一代最后一个线程到达栅栏，就会将代更新并重置计数器，并唤醒所有之前等待在栅栏上的线程。
2. 如果在等待的过程中，线程中断都也会抛出BrokenBarrierException异常，并且这个异常会传播到其他所有的线程，CyclicBarrier会被损坏。

3. 如果超出指定的等待时间，当前线程会抛出 TimeoutException 异常，其他线程会抛出BrokenBarrierException异常，CyclicBarrier会被损坏。

### 5.3 应用场景

CyclicBarrier 可以用于多线程计算数据，最后合并计算结果的应用场景。比如我们用一个 Excel 保存了用户所有银行流水，每个 Sheet 保存一个帐户近一年的每笔银行流水，现在需要统计用户的日均银行流水，先用多线程处理每个 sheet 里的银行流水，都执行完之后，得到每个 sheet 的日均银行流水，最后，再用 barrierAction 用这些线程的计算结果，计算出整个 Excel 的日均银行流水。

示例 1：

```java
public class CyclicBarrierTest1 {
    private static final int THREAD_COUNT = 30;
    // 需要同步的线程数量
    private static final CyclicBarrier cyclicBarrier = new CyclicBarrier(5);

    public static void main(String[] args) throws InterruptedException {
        // 创建线程池
        ExecutorService threadPool = Executors.newFixedThreadPool(10);

        for (int i = 0; i < THREAD_COUNT; i++) {
            final int threadNum = i;
            Thread.sleep(1000);
            threadPool.execute(() -> {
                System.out.println("childThread:" + threadNum + " is ready");
                try {
                    // 等待60秒，保证子线程完全执行结束
                    cyclicBarrier.await(60, TimeUnit.SECONDS);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                } catch (TimeoutException e) {
                    e.printStackTrace();
                }
                System.out.println("childThread:" + threadNum + " is finish");
            });
        }
        threadPool.shutdown();
    }
}
```

运行结果如下：

```java
childThread:0 is ready
childThread:1 is ready
childThread:2 is ready
childThread:3 is ready
childThread:4 is ready
childThread:4 is finish
childThread:0 is finish
childThread:1 is finish
childThread:3 is finish
childThread:2 is finish
childThread:5 is ready
childThread:6 is ready
childThread:7 is ready
childThread:8 is ready
childThread:9 is ready
childThread:9 is finish
childThread:8 is finish
... ...
```

可以看到当线程数量也就是请求数量达定义的 5 个的时候， `await`方法之后的方法才被执行。

另外，CyclicBarrier 还提供一个更高级的构造函数`CyclicBarrier(int parties, Runnable barrierAction)`，用于在线程到达屏障时，优先执行`barrierAction`，方便处理更复杂的业务场景。示例代码如下：

```java
public class CyclicBarrierTest2 {
    private static final int THREAD_COUNT = 30;
    // 需要同步的线程数量
    private static final CyclicBarrier cyclicBarrier = new CyclicBarrier(5, () -> {
        System.out.println("------优先执行------");
    });

    public static void main(String[] args) throws InterruptedException {
        // 创建线程池
        ExecutorService threadPool = Executors.newFixedThreadPool(10);

        for (int i = 0; i < THREAD_COUNT; i++) {
            final int threadNum = i;
            Thread.sleep(1000);
            threadPool.execute(() -> {
                System.out.println("childThread:" + threadNum + " is ready");
                try {
                    // 等待60秒，保证子线程完全执行结束
                    cyclicBarrier.await(60, TimeUnit.SECONDS);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                } catch (TimeoutException e) {
                    e.printStackTrace();
                }
                System.out.println("childThread:" + threadNum + " is finish");
            });
        }
        threadPool.shutdown();
    }
}
```

运行结果如下：

```java
childThread:0 is ready
childThread:1 is ready
childThread:2 is ready
childThread:3 is ready
childThread:4 is ready
------优先执行------
childThread:4 is finish
childThread:0 is finish
childThread:1 is finish
childThread:3 is finish
childThread:2 is finish
childThread:5 is ready
childThread:6 is ready
childThread:7 is ready
childThread:8 is ready
childThread:9 is ready
------优先执行------
childThread:9 is finish
childThread:6 is finish
... ...
```

### 5.4 CyclicBarrier和CountDownLatch的区别

1. CountDownLatch的计数器只能使用一次。而CyclicBarrier的计数器可以使用reset()方法重置，可多次使用。

2. 侧重点不同。CountDownLatch多用于某一个线程等待若干个其他线程执行完任务之后，它才执行；而CyclicBarrier一般用于多个线程互相等待至一个同步点，然后这些线程再继续一起执行。
3. CyclicBarrier还提供其他有用的方法，比如getNumberWaiting方法可以获得Cyclic-Barrier阻塞的线程数量；isBroken()方法用来了解阻塞的线程是否被中断。

## 参考

[JAVA并发编程的艺术](https://weread.qq.com/web/reader/247324e05a66a124750d9e9)

[AQS原理学习笔记](https://juejin.im/post/6844903782636060679#heading-6)

[【JUC】JDK1.8源码分析之AbstractQueuedSynchronizer（二）](https://www.cnblogs.com/leesf456/p/5350186.html)

[死磕 java同步系列之Semaphore源码解析](https://juejin.im/post/6844903866329202701)

 [【JUC】JDK1.8源码分析之CountDownLatch（五）](https://www.cnblogs.com/leesf456/p/5406191.html)

[并发编程之 CyclicBarrier 源码分析](https://juejin.im/post/5ae755256fb9a07ac3634067)

[Java并发之CyclicBarrier](https://juejin.im/entry/6844903487482904584)