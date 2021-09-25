<!-- MarkdownTOC -->

- [1 Semaphore(信号量)](#1-semaphore信号量)
- [2 CountDownLatch (倒计时器)](#2-countdownlatch-倒计时器)
  - [2.1 概述](#21-概述)
  - [2.2 应用场景](#22-应用场景)
- [3 CyclicBarrier(循环栅栏)](#3-cyclicbarrier循环栅栏)
  - [3.1 概述](#31-概述)
  - [3.2 源码分析](#32-源码分析)
  - [3.3 应用场景](#33-应用场景)
  - [3.4 CyclicBarrier和CountDownLatch的区别](#34-cyclicbarrier和countdownlatch的区别)
- [4 Exchanger(数据交换)](#4-exchanger数据交换)
- [参考资料](#参考资料)

<!-- /MarkdownTOC -->

# 1 Semaphore(信号量)

Semaphore（信号量）用来控制 **同时访问特定资源的线程数量**，它通过协调各个线程，以保证合理的使用公共资源。synchronized 和 ReentrantLock 都是一次只允许一个线程访问某个资源，而**Semaphore(信号量)可以指定多个线程同时访问某个资源**。（准入数量 N, N =1 则等价于独占锁，相当于 synchronized 的进化版）

以停车场为例。假设一个停车场只有10个车位，这时如果同时来了15辆车，则只允许其中10辆不受阻碍的进入。剩下的5辆车则必须在入口等待，此后来的车也都不得不在入口处等待。这时，如果有5辆车离开停车场，放入5辆；如果又离开2辆，则又可以放入2辆，如此往复。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210408152244233.png" width="400px"/>
</div>
在这个停车场系统中，车位即是共享资源，每辆车就好比一个线程，信号量就是空车位的数目。

Semaphore中包含了一个实现了AQS的同步器Sync，以及它的两个子类FairSync和NonFairSync。查看Semaphore类结构：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20201105094613463.png" width="500px"/>
</div>
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

| 常用方法                                                                                         | 描述                                                  |
| ------------------------------------------------------------------------------------------------ | ----------------------------------------------------- |
| acquire()/acquire(int permits)                                                                   | 获取许可证。获取许可失败，会进入AQS的队列中排队。     |
| tryAcquire()/tryAcquire(int permits)                                                             | 获取许可证。获取许可失败，直接返回false。             |
| tryAcquire(long timeout, TimeUnit unit)/<br>tryAcquire(int permits, long timeout, TimeUnit unit) | 超时等待获取许可证。                                  |
| release()                                                                                        | 归还许可证。                                          |
| intavailablePermits()                                                                            | 返回此信号量中当前可用的许可证数。                    |
| intgetQueueLength()                                                                              | 返回正在等待获取许可证的线程数。                      |
| booleanhasQueuedThreads()                                                                        | 是否有线程正在等待获取许可证。                        |
| void reducePermits（int reduction）                                                              | 减少reduction个许可证，是个protected方法。            |
| Collection getQueuedThreads()                                                                    | 返回所有等待获取许可证的线程集合，是个protected方法。 |

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

# 2 CountDownLatch (倒计时器)

## 2.1 概述

在日常开发中经常会遇到需要在主线程中开启多个线程去并行执行任务，并且主线程需要等待所有子线程执行完毕后再进行汇总的场景。jdk 1.5之前一般都使用线程的join()方法来实现这一点，但是join方法不够灵活，难以满足不同场景的需要，所以jdk 1.5之后concurrent包提供了CountDownLatch这个类。

**CountDownLatch**是一种同步辅助工具，它**允许一个或多个线程等待其他线程完成操作**，例如阻塞主线程，N 个子线程满足条件时主线程继续。

CountDownLatch是通过一个计数器来实现的，计数器的初始化值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就相应得减1。当计数器到达0时，表示所有的线程都已完成任务，然后在闭锁上等待的线程就可以恢复执行任务。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210609160234329.png" width="400px"/>
</div>

CountDownLatch的方法：

| 方法                               | 描述                                                                               |
| ---------------------------------- | ---------------------------------------------------------------------------------- |
| await()                            | 调用该方法的线程等到构造方法传入的 N 减到 0 的时候，才能继续往下执行。             |
| await(long timeout, TimeUnit unit) | 调用该方法的线程等到指定的 timeout 时间后，不管 N 是否减至为 0，都会继续往下执行。 |
| countDown()                        | 使 CountDownLatch 初始值 N 减 1。                                                  |
| getCount()                         | 获取当前 CountDownLatch 维护的值，也就是AQS的state的值。                           |

CountDownLatch的实现原理，可以查看 [【JUC】JDK1.8源码分析之CountDownLatch（五）](https://www.cnblogs.com/leesf456/p/5406191.html)一文。

根据源码分析可知，**CountDownLatch是AQS中共享锁的一种实现**。AbstractQueuedSynchronizer中维护了一个volatile类型的整数state，volatile可以保证多线程环境下该变量的修改对每个线程都可见，并且由于该属性为整型，因而对该变量的修改也是原子的。

CountDownLatch默认构造 AQS 的 state 值为 count。创建一个CountDownLatch对象时，所传入的整数N就会赋值给state属性。

当调用countDown()方法时，其实是调用了`tryReleaseShared`方法以CAS的操作来对state减1；而调用await()方法时，当前线程就会判断state属性是否为0。如果为0，阻塞线程被唤醒继续往下执行；如果不为0，则使当前线程放入阻塞队列Park，直至最后一个线程调用了countDown()方法使得state == 0，再唤醒在await()方法中等待的线程。

**特别注意的是**：

**CountDownLatch 是一次性的**，计数器的值只能在构造方法中初始化一次，之后没有任何机制再次对其设置值，当 CountDownLatch 使用完毕后，它不能再次被使用。如果**需要能重置计数，可以使用CyclicBarrier**。

## 2.2 应用场景

CountDownLatch主要应用场景：

1. **实现最大的并行性**：同时启动多个线程，实现最大程度的并行性。例如110跨栏比赛中，所有运动员准备好起跑姿势，进入到预备状态，等待裁判一声枪响。裁判开了枪，所有运动员才可以开跑。
2. **开始执行前等待N个线程完成各自任务**：例如Master 线程等待 Worker 线程把任务执行完。形象理解为等所有人干完手上的活，一起去吃饭。

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

开启的30个线程必须引用闭锁对象，因为它们需要通知 `CountDownLatch` 对象，它们已经完成了各自的任务。这种通知机制是通过 `CountDownLatch.countDown()`方法来完成的；每调用一次这个方法，在构造函数中初始化的 count 值就减 1。所以30个线程都调用了这个方法后，count 的值才等于0，然后主线程通过 `await()`方法，继续执行自己的任务。

# 3 CyclicBarrier(循环栅栏)

## 3.1 概述

**CyclicBarrier的字面意思是可循环使用（Cyclic）的屏障（Barrier）**。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续运行。CyclicBarrier 的功能和应用场景与CountDownLatch都非常类似。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20201105150659614.png" width="400px"/>
</div>

CyclicBarrier常用方法：

| 常用方法                           | 描述                                                                              |
| ---------------------------------- | --------------------------------------------------------------------------------- |
| await()                            | 在所有线程都已经在此 barrier上并调用 await 方法之前，将一直等待。                 |
| await(long timeout, TimeUnit unit) | 所有线程都已经在此屏障上调用 await 方法之前将一直等待，或者超出了指定的等待时间。 |
| getNumberWaiting()                 | 返回当前在屏障处等待的线程数目。                                                  |
| getParties()                       | 返回要求启动此 barrier 的线程数目。                                               |
| isBroken()                         | 查询此屏障是否处于损坏状态。                                                      |
| reset()                            | 将屏障重置为其初始状态。                                                          |

## 3.2 源码分析

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

            // 当前线程一直阻塞，直到“有parties个线程到达barrier” 
            // 或 “当前线程被中断” 或 “超时”这3者条件之一发生，当前线程才继续执行。
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

## 3.3 应用场景

CyclicBarrier意味着**任务执行到一定阶段, 等待其他任务对齐，阻塞 N 个线程时所有线程被唤醒继续**。形象地理解为：等待所有人都到达，再一起开吃。

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

## 3.4 CyclicBarrier和CountDownLatch的区别

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

1. CountDownLatch的计数器只能使用一次。而CyclicBarrier的计数器可以使用reset()方法重置，可多次使用。
2. 侧重点不同。**CountDownLatch多用于某一个线程等待若干个其他线程执行完任务之后，它才执行；而CyclicBarrier一般用于多个线程互相等待至一个同步点，然后这些线程再继续一起执行**。
3. CyclicBarrier还提供其他有用的方法，比如getNumberWaiting方法可以获得Cyclic-Barrier阻塞的线程数量；isBroken()方法用来了解阻塞的线程是否被中断。

# 4 Exchanger(数据交换)

Exchanger（交换者）是一个用于线程间协作的工具类。 Exchanger用于进行线程间的数据交换。它提供一个同步点，在这个同 步点，两个线程可以交换彼此的数据。这两个线程通过exchange方法 交换数据，如果第一个线程先执行exchange()方法，它会一直等待第二 个线程也执行exchange方法，当两个线程都到达同步点时，这两个线 程就可以交换数据，将本线程生产出来的数据传递给对方。

Exchanger常见应用场景：

* 用于遗传算法：遗传算法里需要选出两个人作为交 配对象，这时候会交换两人的数据，并使用交叉规则得出2个交配结 果。
* 校对工作：如我们需要将纸制银行流水通 过人工的方式录入成电子银行流水，为了避免错误，采用AB岗两人进 行录入，录入到Excel之后，系统需要加载这两个Excel，并对两个 Excel数据进行校对，看看是否录入一致。

案例：来一个非常经典的并发问题：你有相同的数据buffer，一个或多个数据生产者，和一个或多个数据消费者。只是Exchange类只能同步2个线程，所以你只能在你的生产者和消费者问题中只有一个生产者和一个消费者时使用这个类。

```java
public class ExchangerTest {
    static class Producer extends Thread {
        private Exchanger<Integer> exchanger;
        private static int data = 0;

        Producer(String name, Exchanger<Integer> exchanger) {
            super("Producer-" + name);
            this.exchanger = exchanger;
        }

        @Override
        public void run() {
            for (int i = 1; i < 5; i++) {
                try {
                    TimeUnit.SECONDS.sleep(1);
                    data = i;
                    System.out.println(getName() + " 交换前:" + data);
                    data = exchanger.exchange(data);
                    System.out.println(getName() + " 交换后:" + data);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    static class Consumer extends Thread {
        private Exchanger<Integer> exchanger;
        private static int data = 0;

        Consumer(String name, Exchanger<Integer> exchanger) {
            super("Consumer-" + name);
            this.exchanger = exchanger;
        }

        @Override
        public void run() {
            while (true) {
                data = 0;
                System.out.println(getName() + " 交换前:" + data);
                try {
                    TimeUnit.SECONDS.sleep(1);
                    data = exchanger.exchange(data);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(getName() + " 交换后:" + data);
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Exchanger<Integer> exchanger = new Exchanger<Integer>();
        new Producer("", exchanger).start();
        new Consumer("", exchanger).start();
        TimeUnit.SECONDS.sleep(7);
        System.exit(-1);
    }
}
```

可以看到，其结果可能如下：

```java
Consumer- 交换前:0
Producer- 交换前:1
Consumer- 交换后:1
Consumer- 交换前:0
Producer- 交换后:0
Producer- 交换前:2
Producer- 交换后:0
Consumer- 交换后:2
Consumer- 交换前:0
Producer- 交换前:3
Producer- 交换后:0
Consumer- 交换后:3
Consumer- 交换前:0
Producer- 交换前:4
Producer- 交换后:0
Consumer- 交换后:4
Consumer- 交换前:0
```

# 参考资料

* 《JAVA并发编程的艺术》
* [死磕 java同步系列之Semaphore源码解析](https://juejin.im/post/6844903866329202701)
* [【JUC】JDK1.8源码分析之CountDownLatch（五）](https://www.cnblogs.com/leesf456/p/5406191.html)
* [Java并发之CyclicBarrier](https://juejin.im/entry/6844903487482904584)
* [JUC工具类: Exchanger详解](https://www.pdai.tech/md/java/thread/java-thread-x-juc-tool-exchanger.html)



