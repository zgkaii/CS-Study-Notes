- [1 Lock接口](#1-lock接口)
  - [1.1 Lock与synchronized](#11-lock与synchronized)
  - [1.2 Lock接口方法](#12-lock接口方法)
- [2 ReentrantLock](#2-reentrantlock)
  - [2.1 可重入](#21-可重入)
  - [2.2 公平/非公平](#22-公平非公平)
  - [2.3 小结](#23-小结)
  - [2.4 中断与超时等待](#24-中断与超时等待)
- [3 ReadWriteLock](#3-readwritelock)
  - [3.1 读锁](#31-读锁)
    - [3.1.1 读锁加锁](#311-读锁加锁)
    - [3.1.2 读锁释放](#312-读锁释放)
  - [3.2 写锁](#32-写锁)
    - [3.2.1 写锁加锁](#321-写锁加锁)
    - [3.2.2 写锁释放](#322-写锁释放)
  - [3.3 小结](#33-小结)
- [参考](#参考)
# 1 Lock接口

## 1.1 Lock与synchronized

在Lock接口出现之前，Java程序是靠**synchronized**关键字用来实现锁功能，使用时**隐式地获取和释放锁**，但是它将锁的获取和释放固化了。

所以，如果占有锁的线程由于要等待I/O或者其他原因（比如调用sleep方法）被阻塞了，其他线程就会只能一直等待，直到占有锁的线程释放掉锁，释放锁有以下几种情况：

（1）获取锁的线程执行完了该代码块，然后会自动释放锁。

（2）执行线程发生了异常，JVM会自动释放掉线程的锁。

（3）占有锁的线程进入 WAITING 状态从而释放锁，例如调用了wait()方法等。

这会极大影响程序执行效率。因此，需要有一种机制保证等待的线程不是一直处于无期限地等待的状态（解决方案：tryLock(long time, TimeUnit unit))/lockInterruptibly()）。

使用**synchronized的局限性**还有：

* 当多个线程读写文件时，读操作与读操作之间不会发生冲突。但采用synchronized关键字实现同步时，还是只能一个线程进行读操作，其他读线程只能等待锁的释放而无法进行读操作。因此，需要一种机制来保证多线程都只是进行读操作时，线程之间不会发生冲突(解决方案：ReentrantReadWriteLock) 。
* synchronized同步块无法异步处理锁（即不能立即知道是否可以拿到锁） (解决方案：ReentrantLock)。
* 同步块无法根据条件灵活的加锁解锁（即只能跟同步块范围一致）。
* 同步块的阻塞无法中断（不能 Interruptibly）。
* 同步块的阻塞无法控制超时（无法自动解锁）。

为弥补synchronized使用的局限性，Java SE 5之后，并发包中新增了Lock、ReadWriteLock等接口（以及相关实现）。虽然它们缺少了隐式获取释放锁的便捷性，但是却拥有了**锁获取与释放的可操作性、可中断的获取锁以及超时获取锁**等多种synchronized关键字所不具备的同步特性。

## 1.2 Lock接口方法

Lock 锁使用方式方式灵活可控，性能开销小，其锁工具包位于`java.util.concurrent.locks`下。

Lock接口有6个方法：

```java
// 获取锁; 类比 synchronized (lock)
void lock()   

// 如果当前线程未被中断，则获取锁，可以响应中断  
void lockInterruptibly() throws InterruptedException;  

// 返回绑定到此 Lock 实例的新 Condition 实例  
Condition newCondition()   

// 支持非阻塞获取锁的API，可以响应中断，成功则返回 true 
boolean tryLock()   

// 如果锁在给定的等待时间内空闲，并且当前线程未被中断，则获取锁  
boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

// 释放锁  
void unlock()
```

`lock()、tryLock()、tryLock(long time, TimeUnit unit)和lockInterruptibly()`方法是用来获取锁的。`unLock()`方法是用来释放锁的。

（1）`lock() & unlock()`　

`lock()`用来获取锁。如果锁已被其他线程获取，则进行等待。使用Lock，**必须主动去释放锁**，并且在发生异常时，不会自动释放锁。因此，一般来说，使用Lock必须在try()/catch()块中进行，并且将释放锁的操作放在finally块中进行，以保证锁一定被被释放，防止死锁的发生。通常使用Lock来进行同步的话，是以下面这种形式去使用的：

```java
Lock lock = ...;
lock.lock();// 获取锁
try{
    //处理任务
}catch(Exception ex){

}finally{
    lock.unlock();   //释放锁
}
```

（2）`tryLock() & tryLock(long time, TimeUnit unit)`

`tryLock()`方法有返回值。它表示尝试获取锁，如果获取成功，则返回true；如果获取失败（即锁已被其他线程获取），则返回false。值得注意的是，这个方法**无论如何都会立即返回**（在拿不到锁时不会一直在那等待）。

`tryLock(long time, TimeUnit  unit)`方法和`tryLock()`方法类似，区别在于`tryLock(long time, TimeUnit  unit)`在拿不到锁时会等待一定的时间，在时间期限之内如果还拿不到锁，就返回false，同时可以响应中断。如果一开始拿到锁或者在等待期间内拿到了锁，则返回true。

一般情况下，通过tryLock()是这样使用的：

```java
Lock lock = ...;
if(lock.tryLock()) {
     try{
         //处理任务
     }catch(Exception ex){

     }finally{
         lock.unlock();   //释放锁
     } 
}else {
    //如果不能获取锁，则直接做其他事情
}
```

（3）`lockInterruptibly()`

使用`lockInterruptibly()`方法能够响应中断，即中断线程的等待状态。例如，当两个线程A、B同时通过`lock.lockInterruptibly()`获取锁时，假若此时线程A获取到了锁，而线程B进入等待状态，那么线程B就可调用`interrupt()`方法中断线程B的等待过程。（`interrupt()`方法只能中断阻塞过程中的线程）

`lockInterruptibly()`一般的使用形式如下：

```java
public void method() throws InterruptedException {
    lock.lockInterruptibly();
    try {  
     //.....
    }
    finally {
        lock.unlock();
    }
}
```

Lock接口的实现类有：

![](https://img-blog.csdnimg.cn/20201103124416911.png)

# 2 ReentrantLock

ReentrantLock是Lock接口的主要实现类，ReentrantLock是**可重入锁**，顾名思义，就是支持重进入的锁，它表示该锁**能够支持一个线程对资源的重复加锁**。

![](https://img-blog.csdnimg.cn/20201103222250189.png)

它实现了 Lock 接口，内部类 Sync 是 AQS 的子类；Sync的两个子类 NonfairSync 和 FairSync 分别对应**公平锁和非公平锁两种策略**。（如果在绝对时间上，先对锁进行获取的请求一定先被满足，那么这个锁是公平的；反之，是不公平的。）公平的获取锁，也就是等待时间最长的线程最优先获取锁，也可以说锁获取是顺序的。ReentrantLock提供了一个构造函数，能够控制锁是否是公平的。

```java
    public ReentrantLock() {
        sync = new NonfairSync();
    }

    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```

也就是说，调用ReentrantLock时，不传参数或者传入参数`true`，即是公平锁；传入参数`false`，就是非公平锁。（**ReentrantLock默认采用非公平的策略**）

## 2.1 可重入

**可重入锁**又名递归锁。可重入指的是**任意线程在获取到锁之后能够再次获取该锁而不会被锁所阻塞**（前提锁对象得是同一个对象或者class）。Java中**ReentrantLock和synchronized都是可重入锁**，可重入锁的一个优点是可一定程度避免死锁。

例如：

```java
    public synchronized void fun1() {
        System.out.println("方法1执行...");
        fun2();
    }

    public synchronized void fun2() {
        System.out.println("方法2执行...");
    }
```

类中的两个方法都是被内置锁synchronized修饰的，fun1()方法中调用fun2()方法。因为内置锁是可重入的，所以同一个线程在调用fun2()时可以直接获得当前对象的锁，进入fun2()进行操作。

如果是一个不可重入锁，那么当前线程在调用fun2()之前需要将执行fun1()时获取当前对象的锁释放掉，实际上该对象锁已被当前线程所持有，且无法释放，这种情况下会出现死锁。

那么ReentrantLock是如何实现可重入的呢？下面以非公平锁为例，分析可重入实现原理。

首先查看NonfairSync方法：

```java
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        final void lock() {
            // 利用CAS尝试设置AQS的state为1。设置成功，表示获取锁成功；如果设置失败，表示state已经>=1。
            if (compareAndSetState(0, 1))
                // 线程获取AQS锁成功，需要设置AQS中的变量exclusiveOwnerThread为当前持有锁的线程，做保存记录
                setExclusiveOwnerThread(Thread.currentThread());
            else
                // 调用acquire()，再次尝试或者线程进入等待队列。
                acquire(1);
        }

        // 子类重写的tryAcquire方法
        protected final boolean tryAcquire(int acquires) {
            // 调用nonfairTryAcquire方法
            return nonfairTryAcquire(acquires);
        }
    }
```

查看acquire方法：

```java
public final void acquire(int arg) {
    // 调用子类重写的tryAcquire方法，如果tryAcquire方法返回false，那么线程就会进入同步队列。
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

查看nonfairTryAcquire方法：

```java
	final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();// 获取锁状态（0未加锁；1已加锁）
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {// 直接CAS尝试获取锁，直接返回true，当前线程不会进入同步队列。
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) { // 如果当前线程已占用锁，再次获取锁
                int nextc = c + acquires;// status+1（可重入性）
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);// 重新设置锁的状态
                return true;
            }
            return false;
        }
```

ReentrantLock继承父类AQS，重写了父类tryAcquire方法。其父类AQS中维护了一个同步状态status来计数重入次数，status初始值为0。

当线程尝试获取锁时，可重入锁先尝试获取并更新status值，如果status == 0表示没有其他线程在执行同步代码，则把status置为1，当前线程开始执行。如果status != 0，则判断当前线程是否是获取到这个锁的线程，如果是的话执行status+1，且当前线程可以再次获取锁。

释放锁时，调用tryRelease()方法：

```java
    protected final boolean tryRelease(int releases) {
        int c = getState() - releases;// 释放锁时status-1
        if (Thread.currentThread() != getExclusiveOwnerThread())
            throw new IllegalMonitorStateException();// 先判断当前线程是否已是占用锁的线程
        boolean free = false;
        if (c == 0) {// 只有status=0时才释放锁。
            free = true;
            setExclusiveOwnerThread(null);
        }
        setState(c);
        return free;
    }
```
释放锁时，可重入锁同样先获取当前status的值，在当前线程是持有锁的线程的前提下。如果status-1 == 0，则表示当前线程所有重复获取锁的操作都已经执行完毕，然后该线程才会真正释放锁。如果该锁被获取了n次，那么前(n-1)次tryRelease(int releases)方法必然返回false，而只有同步状态完全释放了，才能返回true。

## 2.2 公平/非公平

**公平锁**是指多个线程**按照申请锁的顺序来获取锁**，线程直接进入队列中排队，队列中的第一个线程才能获得锁（不可插队，等待时间越长，请求锁时会被优先满足）。**公平锁的优点是等待锁的线程不会饥饿**。缺点是整体吞吐效率相对非公平锁要低，等待队列中除第一个线程以外的所有线程都会阻塞，CPU唤醒阻塞线程的开销比非公平锁大。

**非公平锁**是多个线程**加锁时直接尝试获取锁**，获取不到才会到等待队列的队尾等待（可插队的）。但如果此时锁刚好可用，那么这个线程可以无需阻塞直接获取到锁，所以非公平锁有可能出现后申请锁的线程先获取锁的场景。非公平锁的优点是**可以减少唤起线程的开销，整体的吞吐效率高**，因为线程有几率不阻塞直接获得锁，CPU不必唤醒所有线程。缺点是处于等待队列中的线程可能会饥饿，或者等很久才会获得锁。

> Tips：如果一个进程被多次回滚，迟迟不能占用必需的系统资源，可能会导致进程饥饿
>
> 导致线程饥饿常见原因：
>
> 1. 高优先级线程吞噬所有的低优先级线程的CPU时间。
> 2. 线程被永久堵塞在一个等待进入同步快的状态。
> 3. 等待的线程永远不被唤醒。

查看公平加锁方法的源码：

```java
        protected final boolean tryAcquire(int acquires) {
            // 获取当前的线程
            final Thread current = Thread.currentThread();
             // 获取锁的状态
            int c = getState();
            if (c == 0) {
                // hasQueuedPredecessors 判断队列还有没有其它node
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    // 设置获取锁的线程
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            // 设置获取锁的线程
            else if (current == getExclusiveOwnerThread()) {
                // 获取过了就累加，因为可以重入
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                // 重新设置锁的状态
                setState(nextc);
                return true;
            }
            return false;
        }
```

非公平加锁方法的源码：

```java
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```

`unlock()`释放锁，其实就是把state从n（可能发生了锁的重入，需要多次释放）变成0，此方法**不区分公平与否**。

公平锁与非公平锁解锁方法的源码：

```java
    public void unlock() {
        sync.release(1);
    }

    public final boolean release(int arg) {
        // 子类重写的tryRelease方法，需要等锁的state=0，即tryRelease返回true的时候，才会去唤醒其它线程进行尝试获取锁。
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }

	protected final boolean tryRelease(int releases) {
        // 获取锁的状态
        int c = getState() - releases;
        // 判断锁的所有者是不是该线程
        if (Thread.currentThread() != getExclusiveOwnerThread())
            // 如果所的所有者不是该线程 则抛出异常 也就是锁释放的前提是线程拥有这个锁
            throw new IllegalMonitorStateException();
        boolean free = false;
        // 直到锁的状态是0，说明锁释放成功。即锁没有重入，那么直接将将锁的所有者设置成null
        // 我们在一个线程里面调用几次lock，就要调用几次unlock，才能最终释放锁
        if (c == 0) {
            free = true;
            // 释放线程的拥有者
            setExclusiveOwnerThread(null);
        }
        setState(c);
        return free;
    }
```

可见，平锁的释放和非公平锁的释放一样的。公平锁与非公平锁的lock()方法唯一的区别就在于**公平锁在获取同步状态时多了一个限制条件：hasQueuedPredecessors()**。

```java
    public final boolean hasQueuedPredecessors() {
        // 判断当前线程是否位于同步队列中的第一个。如果是则返回true，否则返回false。
        Node t = tail; 
        Node h = head;
        Node s;
        return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());
    }
```

由此可见，**公平锁就是通过同步队列来实现多个线程按照申请锁的顺序来获取锁，从而实现公平的特性。非公平锁加锁时不考虑排队等待问题，直接尝试获取锁，所以才会存在线程后申请却先获得锁的情况**。

## 2.3 小结

ReentrantLock重入锁执行流程：

![](https://img-blog.csdnimg.cn/2020110323501584.png)

## 2.4 中断与超时等待

**（1）lockInterruptibly可中断方式获取锁**

`lockInterruptibly()`支持中断的获取锁，其实是调用了AQS的lockInterruptibly方法。

```java
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }
```

最终调用AQS的doAcquireInterruptibly(int arg)方法：

```java
    public final void acquireInterruptibly(int arg)
        // 当前线程已经中断了，抛出异常。
        if (Thread.interrupted())
            throw new InterruptedException();
        // 当前线程仍然未成功获取锁，则调用doAcquireInterruptibly方法，这个方法和
        // acquireQueued方法没什么区别，就是线程在等待状态的过程中，如果线程被中断，线程会抛出异常。
        if (!tryAcquire(arg))
            doAcquireInterruptibly(arg);
    }
```

**（2）tryLock超时等待方式获取锁**

`tryLock(long timeout, TimeUnit unit)`也支持中断，并且在这个基础上增加了超时设置，其实也是调用了AQS的tryAcquireNanos方法。

```java
 	public boolean tryLock(long timeout, TimeUnit unit)
            throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(timeout));
    }
```

最终调用AQS的doAcquireNanos(int arg, long nanosTimeout)方法：

```java
    public final boolean tryAcquireNanos(int arg, long nanosTimeout)
            throws InterruptedException {
        // 如果当前线程已经中断，则抛出异常
        if (Thread.interrupted())
            throw new InterruptedException();
        //再尝试获取一次，如果不成功则调用doAcquireNanos方法进行超时等待获取锁。
        return tryAcquire(arg) ||
            doAcquireNanos(arg, nanosTimeout);
    }
```

查看tryAcquireNanos方法：

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
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                // 如果当前节点的前驱节点为头结点，则让当前节点去尝试获取锁。
                setHead(node);
                p.next = null; // help GC
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

# 3 ReadWriteLock

之前提到锁（如Mutex和ReentrantLock）基本都是**排他锁**，这些锁在同一时刻只允许一个线程进行访问。而**读写锁（ReadWriteLock）**在**同一时刻可以允许多个读线程访问，但是在写线程访问时，所有的读线程和其他写线程均被阻塞**。

ReadWriteLock维护了一组锁，一个是只读的锁，一个是写锁。读锁可以在没有写锁的时候被多个线程同时持有，写锁是独占的。

如何用一个共享变量来区分锁是写锁还是读锁呢？答案就是`按位拆分`。

由于state是int类型的变量，在内存中`占用4个字节，也就是32位`。将其拆分为两部分：高16位和低16位，其中`高16位用来表示读锁状态，低16位用来表示写锁状态`。当设置读锁成功时，就将高16位加1，释放读锁时，将高16位减1；当设置写锁成功时，就将低16位加1，释放写锁时，将第16位减1。

![](https://img-blog.csdnimg.cn/20201104144106893.png)

ReadWriteLock 接口只有两个方法：

```java
//返回读锁  
Lock readLock()   
//返回写锁  
Lock writeLock() 
```

Java并发库中ReetrantReadWriteLock实现了ReadWriteLock接口并添加了可重入的特性。

![](https://img-blog.csdnimg.cn/20201104131456221.png)

ReentrantReadWriteLock有两个构造方法：

```java
public ReentrantReadWriteLock() {
    this(false);// 默认为false，采用非公平模式
}

public ReentrantReadWriteLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
    readerLock = new ReadLock(this);
    writerLock = new WriteLock(this);
}
```

可以看到，默认的构造方法使用的是非公平模式，创建的Sync是NonfairSync对象，然后初始化读锁和写锁。一旦初始化后，ReadWriteLock接口中的两个方法就有返回值了，如下：

```java
    public ReentrantReadWriteLock.WriteLock writeLock() { return writerLock; }
    public ReentrantReadWriteLock.ReadLock  readLock()  { return readerLock; }
```

从上面可以看到，构造方法决定了Sync是FairSync还是NonfairSync。Sync继承了AQS，而Sync是一个抽象类，NonfairSync和FairSync继承了Sync，并重写了其中的抽象方法。

Sync中提供了很多方法，但是有两个方法是抽象的，子类必须实现。

```java
abstract boolean readerShouldBlock();
abstract boolean writerShouldBlock();
```

FairSync/NonfairSync实现方法如下：

```java
    /**
     * 非公平
     */
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = -8159625535654395037L;
        final boolean writerShouldBlock() {
            return false; // 直接返回false，说明不需要排队
        }
        final boolean readerShouldBlock() {
            /* 当前线程是写锁占用的线程时，返回true；否则返回false。
             * 如果当前有一个写线程正在写，那么该读线程应该阻塞。
             */
            return apparentlyFirstQueuedIsExclusive();
        }
    }

    /**
     * 公平
     */
    static final class FairSync extends Sync {
        private static final long serialVersionUID = -2274990926593161451L;
        final boolean writerShouldBlock() {
            return hasQueuedPredecessors();// 判断同步队列中是否有人在排队
        }
        final boolean readerShouldBlock() {
            return hasQueuedPredecessors();
        }
    }
```

writerShouldBlock()方法的作用是判断当前线程是否应该阻塞，对于公平的写锁和非公平写锁的具体实现不一样。    

* 对于非公平写锁而言，直接返回false，因为非公平锁获取锁之前不需要去判断是否排队。
* 对于公平锁写锁而言，它会判断同步队列中是否有人在排队，有人排队，就返回true，表示当前线程需要阻塞。无人排队就返回false。

## 3.1 读锁

### 3.1.1 读锁加锁

获取读锁时，首先调用ReadLock类中的lock方法：

```java
public void lock() {
    sync.acquireShared(1);
}
```

读锁使用的也是AQS的共享模式，AQS的acquireShared方法如下：

```java
    public final void acquireShared(int arg) {
        if (tryAcquireShared(arg) < 0)
            doAcquireShared(arg);
    }
```

当tryAcquireShared()方法小于0时，那么会执行doAcquireShared方法将该线程加入到等待队列中。

Sync实现了tryAcquireShared方法，如下：

```java
        protected final int tryAcquireShared(int unused) {
            Thread current = Thread.currentThread();
            int c = getState();
            // exclusiveCount(c)返回的是写锁的数量，如果它不为0，说明写锁被占用
            // 如果此时占用写锁的线程不是当前线程，就返回-1，表示获取锁失败
            if (exclusiveCount(c) != 0 &&
                getExclusiveOwnerThread() != current)
                return -1;
            // 得到读锁的数量
            int r = sharedCount(c);
            
            /**
             * 在下面的代码中进行了三个判断：
             * 1、读锁是否应该排队。没有排队，就进行if后面的判断。排队，就直接调用fullTryAcquireShared()方法。
             * 2、读锁数量是否超过最大值。（最大数量为2的16次方-1=65535）
             * 3、是否获取到同步变量的最新状态值
             */          
            if (!readerShouldBlock() &&
                r < MAX_COUNT &&
                compareAndSetState(c, c + SHARED_UNIT)) {
                // 如果读锁数量为0时，当前线程设置为firstReader，即第一个读线程就是当前线程
                if (r == 0) {
                    firstReader = current;
                    firstReaderHoldCount = 1;
                } else if (firstReader == current) {
                    // 如果当前读线程重入了，firstReaderHoldCount累加
                    firstReaderHoldCount++; 
                } else {
                    // 读锁数量不为0，且第一个获取到读锁的线程不是当前线程
                    // 下面这一段逻辑就是保存当前线程获取读锁的次数，如何保存的呢？
                    // 通过ThreadLocal来实现的，readHolds就是一个ThreadLocal的实例
                    HoldCounter rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current))
                        cachedHoldCounter = rh = readHolds.get();
                    else if (rh.count == 0)
                        readHolds.set(rh);
                    rh.count++;
                }
                // 返回1表示获取读锁成功
                return 1;
            }
            // 否则，循环尝试
            return fullTryAcquireShared(current);
        }
```

查看fullTryAcquiredShared方法：

```java
  final int fullTryAcquireShared(Thread current) {   
            HoldCounter rh = null;
     		// for死循环，直到满足相应的条件才会return退出，否则一直循环
            for (;;) {
                int c = getState();
                // 锁的状态为写锁时，持有锁的线程不等于当前线程，说明当前线程获取锁失败，返回-1
                if (exclusiveCount(c) != 0) {
                    if (getExclusiveOwnerThread() != current)
                        return -1;
                } 
                // 如果读锁需要排队
                else if (readerShouldBlock()) {
                    // Make sure we're not acquiring read lock reentrantly
                    if (firstReader == current) {
                        // assert firstReaderHoldCount > 0;
                    }
                    // 说明有别的读线程占有了锁
                    else {
                        if (rh == null) {
                            rh = cachedHoldCounter;
                            if (rh == null || rh.tid != getThreadId(current)) {
                                rh = readHolds.get();
                                if (rh.count == 0)
                                    readHolds.remove();
                            }
                        }
                        if (rh.count == 0)
                            return -1;
                    }
                }
                // 如果读锁达到了最大值，抛出异常
                if (sharedCount(c) == MAX_COUNT)
                    throw new Error("Maximum lock count exceeded");
        		// 尝试设置同步变量的值，只要设置成功了，就表示当前线程获取到了锁，然后就设置锁的获取次数等相关信息
                if (compareAndSetState(c, c + SHARED_UNIT)) {
                    if (sharedCount(c) == 0) {
                        firstReader = current;
                        firstReaderHoldCount = 1;
                    } else if (firstReader == current) {
                        firstReaderHoldCount++;
                    } else {
                        if (rh == null)
                            rh = cachedHoldCounter;
                        if (rh == null || rh.tid != getThreadId(current))
                            rh = readHolds.get();
                        else if (rh.count == 0)
                            readHolds.set(rh);
                        rh.count++;
                        cachedHoldCounter = rh; // cache for release
                    }
                    return 1;
                }
            }
        }
```

从上面可以看到多次调用了readerShouldBlock方法，对于公平锁，只要队列中有线程在等待，那么将会返回true，也就意味着读线程需要阻塞；对于非公平锁，如果当前有线程获取了写锁，则返回true。一旦不阻塞，那么读线程将会有机会获得读锁。

### 3.1.2 读锁释放

当调用readLock.unlock()方法时，会先调用到AQS的releaseShared()方法，在releaseShared()方法中会先调用子类的tryReleaseShared()方法。在这里会调用的是ReentrantReadWriteLock的内部类Sync的`tryReleaseShared()`方法。该方法的源码如下。

```java
protected final boolean tryReleaseShared(int unused) {
    Thread current = Thread.currentThread();
    if (firstReader == current) {
        // assert firstReaderHoldCount > 0;
        if (firstReaderHoldCount == 1)
            firstReader = null;
        else
            firstReaderHoldCount--;
    } else {
        HoldCounter rh = cachedHoldCounter;
        if (rh == null || rh.tid != getThreadId(current))
            rh = readHolds.get();
        int count = rh.count;
        if (count <= 1) {
            readHolds.remove();
            if (count <= 0)
                throw unmatchedUnlockException();
        }
        --rh.count;
    }
    for (;;) {
        int c = getState();
        // 将修改同步变量的值（读锁状态减去1<<16）
        int nextc = c - SHARED_UNIT;
        if (compareAndSetState(c, nextc))
            // Releasing the read lock has no effect on readers,
            // but it may allow waiting writers to proceed if
            // both read and write locks are now free.
            return nextc == 0;
    }
}
```

## 3.2 写锁

### 3.2.1 写锁加锁

首先调用WriteLock类中的lock方法：

```java
public void lock() {
    sync.acquire(1);
}
```

AQS的acquire方法如下：

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        // 写锁使用的是AQS的独占模式。首先尝试获取锁，如果获取失败，那么将会把该线程加入到等待队列中。
        selfInterrupt();
}
```

Sync实现了tryAcquire方法用于尝试获取一把锁，如下：

```java
protected final boolean tryAcquire(int acquires) {
            // 得到调用lock方法的当前线程
            Thread current = Thread.currentThread();
            int c = getState();
            // exclusiveCount()方法的作用是将同步变量与0xFFFF做&运算，计算结果就是写锁的数量。
    		// w即是写锁的数量。
            int w = exclusiveCount(c);
            // 如果线程占有了写锁或者读锁
            if (c != 0) {
                // 如果写锁数量为0，线程占有的必是读锁，而且这个线程并不是当前线程（完全不符合重入），返回false
                if (w == 0 || current != getExclusiveOwnerThread())
                    return false;
                // 如果写锁的数量超过了最大值，抛出异常
                if (w + exclusiveCount(acquires) > MAX_COUNT)
                    throw new Error("Maximum lock count exceeded");
                // 写锁重入，返回true
                setState(c + acquires);
                return true;
            }
    
            /**
             * 1. 当writerShouldBlock()返回true时，表示当前线程还不能直接获取锁，因此tryAcquire()方法直接返回false。
             * 2. 当writerShouldBlock()返回false时，表示当前线程可以尝试去获取锁，因此会执行if判断中后面的逻辑
             * 	  即通过CAS方法尝试去修改同步变量的值。
             * 3. 如果修改同步变量成功，则表示当前线程获取到了锁，最终tryAcquire()方法会返回true。
             *	  如果修改失败，那么tryAcquire()会返回false，表示获取锁失败。
             */
            // 如果当前没有写锁或者读锁，如果写线程应该阻塞或者CAS失败，返回false
            if (writerShouldBlock() ||
                !compareAndSetState(c, c + acquires))
                return false;
            // 否则将当前线程置为获得写锁的线程，返回true
            setExclusiveOwnerThread(current);
            return true;
        }
```

从上面可以看到调用了writerShouldBlock方法，FairSync的实现是如果等待队列中有等待线程，则返回false，说明公平模式下，只要队列中有线程在等待，那么后来的这个线程也是需要记入队列等待的；NonfairSync中的直接返回的直接是false，说明不需要阻塞。从上面的代码可以得出，当没有锁时，如果使用的非公平模式下的写锁的话，那么返回false，直接通过CAS就可以获得写锁。

### 3.2.2 写锁释放

写锁的释放与排他锁的释放逻辑也几乎一样。当调用writeLock.unlock()时，先调用到AQS的release()方法，在release()方法中会先调用子类的tryRelease()方法。在这里调用的是ReentrantReadWriteLock的内部类Sync的tryRelease()方法。写锁的释放逻辑比较简单，可以参考下面源码中的注释。方法的源码和注释如下。

```java
protected final boolean tryRelease(int releases) {
    // 判断是否是当前线程持有锁
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    // 将state的值减去releases
    int nextc = getState() - releases;
    // 调用exclusiveCount()方法，计算写锁的数量。如果写锁的数量为0，表示写锁被完全释放，此时将AQS的exclusiveOwnerThread属性置为null
    // 并返回free标识，表示写锁是否被完全释放
    boolean free = exclusiveCount(nextc) == 0;
    if (free)
        setExclusiveOwnerThread(null);
    setState(nextc);
    return free;
}
```

## 3.3 小结

ReadWriteLock读写锁执行流程：![](https://img-blog.csdnimg.cn/20201104111903784.png)

**读写锁不支持锁升级，支持锁降级**。锁降级指的是线程获取到了写锁，在没有释放写锁的情况下，又获取读锁。以下面代码为例：

```java
public class WriteReadLockTest {
    Object data;
    // 是否有效，如果失效，需要重新计算 data
    volatile boolean cacheValid;
    final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();

    void processCachedData() {
        rwl.readLock().lock();
        if (!cacheValid) {
            // 获取写锁前必须释放读锁
            rwl.readLock().unlock();
            rwl.writeLock().lock();
            try {
                // 判断是否有其它线程已经获取了写锁、更新了缓存, 避免重复更新
                if (!cacheValid) {
                    data = ...
                    cacheValid = true;
                }
                // 降级为读锁, 释放写锁, 这样能够让其它线程读取缓存
                rwl.readLock().lock();
            } finally {

                rwl.writeLock().unlock();
            }
        }
        // 自己用完数据, 释放读锁
        try {
            use(data);
        } finally {
            rwl.readLock().unlock();
        }
    }
}
```

# 参考

[JAVA并发编程的艺术](https://weread.qq.com/web/reader/247324e05a66a124750d9e9kc9f326d018c9f0f895fb5e4)

[不可不说的Java“锁”事](https://tech.meituan.com/2018/11/15/java-lock.html)

[深入理解ReentrantLock的实现原理](https://juejin.im/post/6844903805683761165)

[读写锁ReadWriteLock的实现原理](https://juejin.im/post/6844903988169555975)

[深入理解读写锁—ReadWriteLock源码分析](https://blog.csdn.net/qq_19431333/article/details/70568478)