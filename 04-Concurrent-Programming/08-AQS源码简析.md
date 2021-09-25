<!-- MarkdownTOC -->

- [1 AQS概述](#1-aqs概述)
- [2 AQS常用方法](#2-aqs常用方法)
- [2 AQS实现分析](#2-aqs实现分析)
  - [2.1 同步队列](#21-同步队列)
    - [2.1.1 入队](#211-入队)
    - [2.1.2 出列](#212-出列)
  - [2.2 同步状态](#22-同步状态)
    - [2.2.1 独占式(EXCLUSIVE)](#221-独占式exclusive)
    - [2.2.2 共享式(SHARED)](#222-共享式shared)
    - [2.2.3 超时获取方式](#223-超时获取方式)
  - [2.3 阻塞和唤醒线程](#23-阻塞和唤醒线程)
    - [2.3.1 parkAndCheckInterrupt](#231-parkandcheckinterrupt)
    - [2.3.2 unparkSuccessor](#232-unparksuccessor)
    - [2.3.3 LockSupport](#233-locksupport)
- [3 总结](#3-总结)
- [参考资料](#参考资料)
<!-- /MarkdownTOC -->

# 1 AQS概述

AbstractQueuedSynchronizer，即**队列同步器**，一般简称AQS。它是**构建锁或者其他同步组件的基础**（如 Semaphore、CountDownLatch、ReentrantLock、ReentrantReadWriteLock），**是 JUC 并发包中的核心基础组件**。

它**抽象了竞争的资源和线程队列**，使用了一个int成员变量表示同步状态，通过内置的FIFO队列来完成资源获取线程的排队工作。使用 AQS 能简单且高效地构造出应用广泛的大量的同步器，比如上篇文章写的ReentrantLock与ReentrantReadWriteLock。除此之外，AQS还能构造出Semaphore，FutureTask(jdk1.7) 等同步器。

AQS 的主要使用方式是**继承**，子类通过继承同步器，并实现它的**抽象方法**来管理同步状态。

AQS 使用一个 `int` 类型的成员变量 `state` 来**表示同步状态**：

- 当 `state > 0` 时，表示已经获取了锁。
- 当 `state = 0` 时，表示释放了锁。

它提供了三个方法，来对同步状态 `state` 进行操作，并且 AQS 可以确保对 `state` 的操作是**安全**的：

- `#getState()`
- `#setState(int newState)`
- `#compareAndSetState(int expect, int update)`

可以把AQS理解为抽象队列式的同步器，它有两种资源共享方式：独占|共享，子类负责实现公平|非公平，下面我们会详细讲到。

------

如何理解AQS与锁的关系呢？

其实很好理解，锁是面向使用者的，它定义了使用者与锁交互的接口（比如可以允许两个线程并行访问），隐藏了实现细节；同步器是锁的实现者，它简化了锁的实现方式，屏蔽了同步状态管理、线程排队、等待与唤醒等底层操作。锁和同步器很好地隔离了使用者和实现者所需关注的领域。

# 2 AQS常用方法

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
tryAcquire(int arg)		// 独占式获取同步状态，获取同步状态成功后，其他线程需要等待该线程释放同步状态才能获						   // 取同步状态。成功则返回true，失败则返回false。
tryRelease(int arg)		// 独占式释放同步状态。成功则返回true，失败则返回false。
tryAcquireShared(int arg)// 共享式获取同步状态，返回值大于等于0，则表示获取成功；否则，获取失败。
tryReleaseShared(int arg)// 共享式释放同步状态。成功则返回true，失败则返回false。
isHeldExclusively()		 // 当前同步器是否在独占式模式下被线程占用，一般该方法表示是否被当前线程所独占。
    					 // 只有用到condition才需要去实现它。
```

AQS 还主要提供了如下**方法**：

- `acquire(int arg)`：独占式获取同步状态。如果当前线程获取同步状态成功，则由该方法返回；否则，将会进入同步队列等待。该方法将会调用**可重写**的 `#tryAcquire(int arg)` 方法；
- `#acquireInterruptibly(int arg)`：与 `#acquire(int arg)` 相同，但是该方法响应中断。当前线程为获取到同步状态而进入到同步队列中，如果当前线程被中断，则该方法会抛出InterruptedException 异常并返回。
- `#tryAcquireNanos(int arg, long nanos)`：超时获取同步状态。如果当前线程在 nanos 时间内没有获取到同步状态，那么将会返回 false ，已经获取则返回 true 。
- `#acquireShared(int arg)`：共享式获取同步状态，如果当前线程未获取到同步状态，将会进入同步队列等待，与独占式的主要区别是在同一时刻可以有多个线程获取到同步状态；
- `#acquireSharedInterruptibly(int arg)`：共享式获取同步状态，响应中断。
- `#tryAcquireSharedNanos(int arg, long nanosTimeout)`：共享式获取同步状态，增加超时限制。
- `#release(int arg)`：独占式释放同步状态，该方法会在释放同步状态之后，将同步队列中第一个节点包含的线程唤醒。
- `#releaseShared(int arg)`：共享式释放同步状态。

从上面的方法看下来，同步器提供的模板方法基本上分为3类：

* **独占式**获取与释放同步状态
* **共享式**获取与释放同步状态
* 查询同步队列中的等待线程情况。

一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现`tryAcquire-tryRelease`、`tryAcquireShared-tryReleaseShared`中的一种即可。

以 ReentrantLock 为例，state 初始化为 0，表示未锁定状态。A 线程 lock()时，会调用 tryAcquire()独占该锁并将 state+1。此后，其他线程再 tryAcquire()时就会失败，直到 A 线程 unlock()到 state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A 线程自己是可以重复获取此锁的（state 会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证 state 是能回到零态的。

再以 CountDownLatch 以例，任务分为 N 个子线程去执行，state 也初始化为 N（注意 N 要与线程个数一致）。这 N 个子线程是并行执行的，每个子线程执行完后 countDown()一次，state 会 CAS(Compare and Swap)减 1。等到所有子线程都执行完后(即 state=0)，会 unpark()主调用线程，然后主调用线程就会从 await()函数返回，继续后续动作。

但 AQS 也支持自定义同步器同时实现独占和共享两种方式，如`ReentrantReadWriteLock`。

# 2 AQS实现分析

## 2.1 同步队列

**AQS 是依赖 CLH 队列锁来完成同步状态的管理**。如果当前线程获取同步状态失败（锁）时，AQS 则会将当前线程以及等待状态等信息构造成一个**节点（Node）**并将其加入同步队列，同时会阻塞当前线程；当同步状态**释放**时，则会把节点中的线程唤醒，使其再次尝试获取同步状态。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210605211037575.png" width="600px"/>
</div>
> CLH（Craig,Landin,and Hagersten）队列是一个虚拟的双向队列（**FIFO双向队列**）（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。**AQS 是将每条请求共享资源的线程封装成一个 CLH 锁队列的一个结点（Node）来实现锁的分配**。

同步队列中的节点（Node）用来保存获取同步状态失败的线程引用、等待状态以及前驱和后继节点信息。

| 属性类型与名称  | 描述                                                                                                                                             |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| int waitStatus  | 等待状态（如CANCELLED=1、SIGNAL=-1、CONDITION=-2、PROPAGATE=-3、INITIAL=0）<br>INITIAL值为0，为初始状态                                          |
| Node prev       | 前驱节点(当节点加入同步队列时被设置，在尾部添加)                                                                                                 |
| Node next       | 后继节点                                                                                                                                         |
| Node nextWaiter | 等待队列的后继节点。如果当前节点是共享的，那么这个节点一定是一个SHARED常量，也就是说节点类型（独占或者共享）和等待队列中的后继节点共用同一个实现 |
| Thread thread   | 当前获取同步状态的线程                                                                                                                           |

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

### 2.1.1 入队

节点是构成同步队列的基础，同步器拥有**首节点（Head）和尾节点（Tail）**，没有成功**获取**同步状态的线程将会成为节点加入该队列的尾部。同步器提供了一个基于CAS的设置尾节点的方法：`compareAndSetTail(Node expect, Node update)`，它需要传递当前线程“认为”的尾节点和当前节点，只有设置成功后，当前节点才正式与之前的尾节点建立关联。关联过程：（1）`tail` 指向新节点。（2）新节点的 `prev` 指向当前最后的节点。（3）当前最后一个节点的 `next` 指向当前节点。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20201104225437655.png" width="600px"/>
</div>

入队逻辑是实现的 `#addWaiter(Node)` 方法，需要考虑**并发**的情况。它通过 **CAS** 的方式，来保证正确的添加 Node 。

```java
    private Node addWaiter(Node mode) {
        // 新建节点。在创建的构造方法，`mode` 方法参数，传递获取同步状态的模式。
        Node node = new Node(Thread.currentThread(), mode);
        // 记录原节点
        Node pred = tail;
        // 尝试添加新尾节点
        if (pred != null) {
            node.prev = pred; // 新节点的prev指向当前最后的节点。
            if (compareAndSetTail(pred, node)) { // CAS 设置新的尾节点
                pred.next = node; // 前最后一个节点的 next 指向当前节点。
                return node;
            }
        }
        // 多次尝试直到成功
        enq(node);
        return node;
    }
```

我们来查看一下`enq()`方法：

```java
private Node enq(final Node node) {
    for (;;) { // 死循环，尝试查入到成功位置
        Node t = tail; // 记录原尾节点
        if (t == null) { // 原尾节点不存在，创建首尾节点都为 new Node()
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {  // 原尾节点存在，添加新节点为尾节点
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

第4 - 6行，原尾节点不存在，创建首尾节点都为new Node()。**注意**，此时修改的**首尾**节点是重新创建( `new Node()` )的，而不是**新节点**！

* 通过这样的方式，初始化好同步队列的**首尾**。另外，在 AbstractQueuedSynchronizer 的设计中，`head` 字段，是一个“占位节点”(暂时没想到特别好的比喻)，代表**最后一个**获得到同步状态的节点(线程)，实际它已经**出列**，所以它的 `Node.next` 才是**真正**的队首。当然，同步队列的初始时，`new Node()` 也是满足这个条件，因为有**新的** Node 进队列，**目前就已经有线程获得到同步状态**。
* `#compareAndSetHead(Node update)` 方法，使用 Unsafe 来 CAS 设置尾节点 `head` 为新节点。代码如下：

```java
private static final Unsafe unsafe = Unsafe.getUnsafe();

// 这块代码，实际在 static 代码块，此处为了方便理解，做了简化。
private static final long headOffset = unsafe.objectFieldOffset
                (AbstractQueuedSynchronizer.class.getDeclaredField("head"));  

/**
 * CAS head field. Used only by enq.
 */
private final boolean compareAndSetHead(Node update) {
    return unsafe.compareAndSwapObject(this, headOffset, null, update);
}
```

### 2.1.2 出列

**首节点是获取同步状态成功的节点**，首节点的线程在**释放**同步状态时，将会唤醒后继节点，而后继节点将会在获取同步状态成功时将自己设置为首节点。这个过程非常简单，`head` 执行该节点并断开原首节点的 `next` 和当前节点的 `prev` 即可。

注意，设置首节点是通过获取同步状态成功的线程来完成的，在这个过程是**不需要使用CAS来保证**的，因为**只有一个**线程，能够成功获取到同步状态。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20201104225709623.png" width="600px"/>
</div>

`#setHead(Node node)` 方法，实现了上述的**出列**逻辑。代码如下：

```java
    private void setHead(Node node) {
        head = node;
        node.thread = null;
        node.prev = null;
    }
```

## 2.2 同步状态

### 2.2.1 独占式(EXCLUSIVE)

独占式，即**同一时刻，仅有一个线程持有同步状态**。独占式(EXCLUSIVE)获取需重写`tryAcquire`、`tryRelease`方法，并访问`acquire`、`release`方法实现相应的功能。

`#acquire(int arg)` 方法，为 AQS 提供的**模板方法**。该方法为独占式获取同步状态，但是该方法对**中断不敏感**。也就是说，由于线程获取同步状态失败而加入到 CLH 同步队列中，后续对该线程进行中断操作时，线程**不会**从 CLH 同步队列中**移除**。代码如下：

```java
    public final void acquire(int arg) {
        // 如果线程直接获取成功，或者再尝试获取成功后都是直接工作，
        // 如果是从阻塞状态中唤醒开始工作的线程，将当前的线程中断        
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```

第4行掉用`tryAcquire(int arg)`方法，去尝试获取同步状态，获取成功则设置锁状态并返回true，否则择失败，返回false。若获取成功`#acquire(int arg)` 方法直接返回，**不用线程阻塞**，自旋直到获得同步状态成功。

`#tryAcquire(int arg)` 方法，**需要**自定义同步组件**自己实现**，该方法必须要保证**线程安全**的获取同步状态。代码如下：

```java
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}
```

如果第4行 `#tryAcquire(int arg)` 方法返回 false ，即获取同步状态失败，则调用 `#addWaiter(Node mode)` 方法，将当前线程加入到 CLH 同步队列尾部。并且， `mode` 方法参数为 `Node.EXCLUSIVE` ，表示**独占**模式。

其中`boolean #acquireQueued(Node node, int arg)` 方法，为一个**自旋**的过程，也就是说，当前线程（Node）进入同步队列后，就会进入一个自旋的过程，每个节点都会**自省**地观察，当条件满足，获取到同步状态后，就可以从这个自旋过程中退出，否则会一直执行下去。

```java
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
        boolean failed = true;// 记录是否获取同步成功
        try {
            // 记录过程中，是否发生线程中断
            boolean interrupted = false;
            // 自旋
            for (;;) {
                // 当前线程的前驱节点
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

这里为什么只有头驱节点才能获取到同步状态呢？

* 头节点是成功获取到同步状态的节点，而头节点的线程释放了同步状态后，将会唤醒其后的后继节点，后继节点的线程被唤醒后需要检查自己的前驱节点是否为头节点。
* 维护同步队列的FIFO原则。

独占式获取同步状态流程如下：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20201104234304168.png" width="600px"/>
</div>

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

### 2.2.2 共享式(SHARED)

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

### 2.2.3 超时获取方式

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

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20201105002205786.png" width="700px"/>
</div>

## 2.3 阻塞和唤醒线程

### 2.3.1 parkAndCheckInterrupt

在线程获取同步状态时，如果获取失败，则加入 CLH 同步队列，通过通过自旋的方式不断获取同步状态，但是在自旋的过程中，则需要判断当前线程是否需要阻塞，其主要方法在`acquireQueued(int arg)` ，代码如下：

```java
// ... 省略前面无关代码

if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;

// ... 省略前面无关代码
```

通过这段代码我们可以看到，在获取同步状态失败后，线程并不是立马进行阻塞，需要检查该线程的状态，检查状态的方法为 `#shouldParkAfterFailedAcquire(Node pred, Node node)`方法，该方法主要靠前驱节点判断当前线程**是否应该被阻塞**。

```java
// pred 和 node 方法参数，传入时，要求前者必须是后者的前一个节点。
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;// 获得前一个节点的等待状态
    if (ws == Node.SIGNAL)//  Node.SIGNAL
		// 等待状态为 Node.SIGNAL 时，表示 pred 的下一个节点 node 的线程需要阻塞等待。
        // 在 pred 的线程释放同步状态时，会对 node 的线程进行唤醒通知。
        // 所以返回true，表明当前线程可以被 park，安全的阻塞等待。
        return true;
	// 等待状态为 NODE.CANCELLED 时，则表明该线程的前一个节点已经等待超时或者被中断了
    // 则需要从 CLH 队列中将该前一个节点删除掉，循环回溯，直到前一个节点状态 <= 0 。
    if (ws > 0) {// Node.CANCEL
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {// 0 或者 Node.PROPAGATE
        // 等待状态为 0 或者 Node.PROPAGATE 时，通过 CAS 设置，将状态修改为 Node.SIGNAL
        // 即下一次重新执行 shouldParkAfterFailedAcquire()方法时，满足第4至8行的条件。
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

如果 `#shouldParkAfterFailedAcquire(Node pred, Node node)` 方法返回 **true** ，则调用`parkAndCheckInterrupt()` 方法，阻塞当前线程。代码如下：

```java
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
```

- 开始，调用 `LockSupport#park(Object blocker)` 方法，将当前线程挂起，此时就进入**阻塞等待**唤醒的状态。
- 然后，在线程被唤醒时，调用`Thread#interrupted()`方法，返回当前线程是否被打断，并清理打断状态。所以，实际上，线程被唤醒有两种情况：
  - 第一种，当前节点(线程)的**前序节点**释放同步状态时，唤醒了该线程。详细解析，见下面unparkSuccessor()解析。
  - 第二种，当前线程被打断导致唤醒。

### 2.3.2 unparkSuccessor

当线程释放同步状态后，则需要唤醒该线程的后继节点。代码如下：

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h); // 唤醒后继节点
        return true;
    }
    return false;
}
```

- 调用 `unparkSuccessor(Node node)` 方法，唤醒**后继**节点：

  ```java
  private void unparkSuccessor(Node node) {
      //当前节点状态
      int ws = node.waitStatus;
      //当前状态 < 0 则设置为 0
      if (ws < 0)
          compareAndSetWaitStatus(node, ws, 0);
  
      //当前节点的后继节点
      Node s = node.next;
      //后继节点为null或者其状态 > 0 (超时或者被中断了)
      if (s == null || s.waitStatus > 0) {
          s = null;
          //从tail节点来找可用节点
          for (Node t = tail; t != null && t != node; t = t.prev)
              if (t.waitStatus <= 0)
                  s = t;
      }
      //唤醒后继节点
      if (s != null)
          LockSupport.unpark(s.thread);
  }
  ```

  可能会存在当前线程的**后继**节点为 `null`，例如：超时、被中断的情况。如果遇到这种情况了，则需要跳过该节点。

  - 但是，为何是从 `tail` 尾节点开始，而不是从 `node.next` 开始呢？原因在于，取消的 `node.next.next` 指向的是 `node.next` 自己。如果顺序遍历下去，会导致**死循环**。所以此时，**只能**采用 `tail` 回溯的办法，找到第一个( 不是**最新**找到的，而是**最前序的** )可用的线程。
  - 再但是，为什么取消的 `node.next.next` 指向的是 `node.next` 自己呢？在 `#cancelAcquire(Node node)` 的末尾，`node.next = node;` 代码块，取消的 `node` 节点，将其 `next` 指向了自己。

  - 最后，调用 `LockSupport的unpark(Thread thread)` 方法，唤醒该线程。详细的实现，在LockSupport中。

### 2.3.3 LockSupport

从上面我可以看到，当需要阻塞或者唤醒一个线程的时候，AQS 都是使用 LockSupport 这个工具类来完成的。

> LockSupport 是用来创建锁和其他同步类的基本线程阻塞原语。

每个使用 LockSupport 的线程都会与一个许可与之关联：

- 如果该许可可用，并且可在进程中使用，则调用 `#park(...)` 将会立即返回，否则可能阻塞。
- 如果许可尚不可用，则可以调用 `#unpark(...)` 使其可用。
- 但是，注意许可**不可重入**，也就是说只能调用一次 `park(...)` 方法，否则会一直阻塞。

LockSupport 定义了一系列以 `park` 开头的方法来阻塞当前线程，`unpark(Thread thread)` 方法来唤醒一个被阻塞的线程。如下图所示：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210609100241585.png" width="700px"/>
</div>

- `park(Object blocker)` 方法的blocker参数，主要是用来标识当前线程在等待的对象，该对象主要用于**问题排查和系统监控**。
- park 方法和 `unpark(Thread thread)` 方法，都是**成对出现**的。同时 `unpark(Thread thread)` 方法，必须要在 park 方法执行之后执行。当然，并不是说没有调用 `unpark(Thread thread)` 方法的线程就会一直阻塞，park 有一个方法，它是带了时间戳的 `#parkNanos(long nanos)` 方法：为了线程调度禁用当前线程，最多等待指定的等待时间，除非许可可用。

**park与unpark方法**

```java
public static void park() {
    UNSAFE.park(false, 0L);
}

public static void unpark(Thread thread) {
    if (thread != null)
        UNSAFE.unpark(thread);
}
```

**实现原理**

从上面可以看出，其内部的实现都是通过 `sun.misc.Unsafe` 来实现的，其定义如下：

```java
// UNSAFE.java
public native void park(boolean var1, long var2);
public native void unpark(Object var1);
```

两个都是 `native` 本地方法。Unsafe 是一个比较危险的类，主要是用于执行低级别、不安全的方法集合。尽管这个类和所有的方法都是公开的（使用 `public` 进行修饰），但是这个类的使用仍然受限，你无法在自己的 Java 程序中直接使用该类，因为只有授信的代码才能获得该类的实例。

# 3 总结

AQS（AbstractQueuedSynchronizer，即**队列同步器**）是**构建锁或者其他同步组件的基础，是 JUC 并发包中的核心基础组件，它抽象了竞争的资源和线程队列**。

**AQS 是依赖 CLH 队列（FIFO双向队列）来完成同步状态的管理**。如果当前线程获取同步状态失败（锁）时，AQS 则会将当前线程以及等待状态等信息构造成一个**节点（Node）**并将其加入同步队列队尾，同时会阻塞当前线程；**首节点是获取同步状态成功的节点**，当同步状态**释放**时，则会把节点中的线程唤醒，使其再次尝试获取同步状态。

**独占式同步状态**获取和释放锁的过程大致为：在获取同步状态时，同步器维护一个FIFO同步队列，获取状态失败的线程都会假如到队列中并在队列中进行自旋；移出队列（或停止）自旋的条件时前驱节点为头节点且成功获取了同步状态。在释放同步状态时，同步器用`tryReleasw(int arg)`方法释放同步状态，然后唤醒头节点的后继节点。

**独占式超时获取同步状态**`doAcquireNanos(int arg,long nanosTimeout)`和独占式获取同步状态`acquire(int args)`在流程上 非常相似，其主要区别在于未获取到同步状态时的处理逻辑。 `acquire(int args)`在未获取到同步状态时，将会使当前线程一直处于等待 状态，而`doAcquireNanos(int arg,long nanosTimeout)`会使当前线程等待 `nanosTimeout`纳秒，如果当前线程在`nanosTimeout`纳秒内没有获取到同 步状态，将会从等待逻辑中自动返回。

**共享式获取与独占式获取最主要的区别在于同一时刻能否有多个线程同时获取到同步状态**。

共享式(SHARED)获取需重写`tryAcquireShared`、`tryReleaseShared`方法，并访问`acquireShared`、`releaseShared`方法实现相应的功能。与独占式相对，共享式支持多个线程同时获取到同步状态并进行工作，如 Semaphore、CountDownLatch、 CyclicBarrier等。

当我们需要阻塞或者唤醒一个线程的时候，AQS 都是通过 **LockSupport** 这个工具类来完成的。

# 参考资料

* 《Java并发编程的艺术》
* [Java并发之AQS全面详解](https://juejin.cn/post/6844903997438951437)
* [从ReentrantLock的实现看AQS的原理及应用](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html)

