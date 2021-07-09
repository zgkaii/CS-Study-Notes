<!-- MarkdownTOC -->

- [1 使用线程池的好处](#1-使用线程池的好处)
- [2 Executor 框架](#2-executor-框架)
  - [2.1 Executor 框架结构](#21-executor-框架结构)
  - [2.2 Executor 框架使用示意图](#22-executor-框架使用示意图)
  - [2.3 Executor 框架成员](#23-executor-框架成员)
    - [2.3.1 Executor 与 ExecutorService](#231-executor-与-executorservice)
    - [2.3.2 Future接口](#232-future接口)
    - [2.3.3 Runnable接口和Callable接口](#233-runnable接口和callable接口)
    - [2.3.4 Executors创建线程池](#234-executors创建线程池)
- [3 ThreadPoolExecutor 详解](#3-threadpoolexecutor-详解)
  - [3.1 线程池状态转换](#31-线程池状态转换)
  - [3.2 线程池处理流程](#32-线程池处理流程)
    - [3.2.1 execute()方法](#321-execute方法)
    - [3.2.2 Worker线程管理](#322-worker线程管理)
      - [3.2.2.1 Worker线程](#3221-worker线程)
      - [3.2.2.2 Worker线程增加](#3222-worker线程增加)
      - [3.2.2.3 Worker线程回收](#3223-worker线程回收)
      - [3.3.3.4 Worker线程执行任务](#3334-worker线程执行任务)
  - [3.2 线程池参数](#32-线程池参数)
    - [3.2.1 线程池参数简述](#321-线程池参数简述)
    - [3.3.1 拒绝策略](#331-拒绝策略)
    - [3.3.2 缓冲队列](#332-缓冲队列)
- [4 ScheduledThreadPoolExecutor 详解](#4-scheduledthreadpoolexecutor-详解)
  - [4.2 ScheduledThreadPoolExecutor 运行机制](#42-scheduledthreadpoolexecutor-运行机制)
  - [4.3 ScheduledThreadPoolExecutor 执行周期任务的步骤](#43-scheduledthreadpoolexecutor-执行周期任务的步骤)
- [5 常见线程池](#5-常见线程池)
  - [5.1 FixedThreadPool](#51-fixedthreadpool)
    - [5.1.1 FixedThreadPool 处理流程](#511-fixedthreadpool-处理流程)
    - [5.1.2 为什么不推荐使用`FixedThreadPool`？](#512-为什么不推荐使用fixedthreadpool)
  - [5.2 SingleThreadExecutor 详解](#52-singlethreadexecutor-详解)
    - [5.2.1 SingleThreadExecutor 处理流程](#521-singlethreadexecutor-处理流程)
    - [5.2.2 为什么不推荐使用`SingleThreadExecutor`？](#522-为什么不推荐使用singlethreadexecutor)
  - [5.3 CachedThreadPool 详解](#53-cachedthreadpool-详解)
    - [5.3.1 SingleThreadExecutor 处理流程](#531-singlethreadexecutor-处理流程)
    - [5.3.2 为什么不推荐使用`CachedThreadPool`？](#532-为什么不推荐使用cachedthreadpool)
- [6 线程池实践](#6-线程池实践)
  - [6.1 线程池数量计算: CPU密集型 VS I/O密集型](#61-线程池数量计算-cpu密集型-vs-io密集型)
  - [6.2 线程池监控](#62-线程池监控)
    - [6.2.1 负载监控和告警](#621-负载监控和告警)
    - [6.2.2 任务级精细化监控](#622-任务级精细化监控)
    - [6.2.3 运行时状态实时查看](#623-运行时状态实时查看)
  - [6.3 线程池命名](#63-线程池命名)
  - [6.4 动态化线程池](#64-动态化线程池)
    - [6.4.1 整体设计](#641-整体设计)
    - [6.4.2 功能架构](#642-功能架构)
- [总结](#总结)
- [参考资料](#参考资料)

<!-- /MarkdownTOC -->

# 1 使用线程池的好处

在 `HotSpot VM` 的线程模型中，Java 线程被一对一映射为内核线程。Java 在使用线程执行程序时，需要创建一个内核线程；当该 Java 线程被终止时，这个内核线程也会被回收。因此 Java 线程的创建与销毁将会消耗一定的计算机资源，从而增加系统的性能开销。

除此之外，大量创建线程同样也会给系统带来性能问题，因为内存和 CPU 资源都将被线程抢占，如果处理不当，就会发生内存溢出、CPU 使用率超负荷等问题。

为了解决上述两类问题，Java 提供了线程池概念，对于频繁创建线程的业务场景，线程池可以创建固定的线程数量，并且在操作系统底层，轻量级进程将会把这些线程映射到内核。

这里借用《Java 并发编程的艺术》提到的来说一下**使用线程池的好处**：

- **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
- **提高响应速度**。当任务到达时，任务可以不需要的等到线程创建就能立即执行。
- **提高线程的可管理性**。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

# 2 Executor 框架

Java并发编程中，操作系统会调度所有线程并将它们分配给可用的CPU。 在上层，Java多线程程序通常把应用分解为若干个任务，然后使用用户级的调度器（Executor框架）将这些任务映射为固定数量的线程；在底层，操作系统内核将这些线程映射到硬件处理器上。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210619025149950.png" width="550px"/>
</div>

从图中可以看出，应用程序通过Executor框架控制上层的调度； 而下层的调度由操作系统内核控制，下层的调度不受应用程序的控 制。

## 2.1 Executor 框架结构

Executor框架主要由3大部分组成如下：

* **任务**。包括被执行任务需要实现的接口：`Runnable`接口或`Callable`接口。
  * `Runnable`接口和`Callable`接口的实现类，都可以被`ThreadPoolExecutor`或`ScheduledThreadPoolExecutor`执行.
* **任务的执行**。包括任务执行机制的核心接口`Executor`，以及继承自Executor的 `ExecutorService`接口。`Executor`框架有两个关键类实现了`ExecutorService`接口 （`ThreadPoolExecutor`和`ScheduledThreadPoolExecutor`）。
  * Executor是一个接口，它是Executor框架的基础，它将任务的提交与任务的执行分离开来。
  * `ScheduledThreadPoolExecutor` 用来定时执行任务； `ThreadPoolExecutor` 用来执行被提交的任务。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210619023146515.png" width="500px"/>
</div>

* **异步计算的结果**。包括接口`Future`和 实现`Future`接口的`FutureTask`类。
  * 把 `Runnable`接口 或 `Callable` 接口 的实现类提交给 `ThreadPoolExecutor` 或 `ScheduledThreadPoolExecutor` 执行。（调用 `submit()` 方法时会返回一个 `FutureTask` 对象）

## 2.2 Executor 框架使用示意图

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210619023859364.png" width="600px"/>
</div>

（1）主线程首先要创建实现Runnable或者Callable接口的任务对象。工具类`Executors`可以把 一个Runnable对象封装为一个Callable对象（`Executors.callable(Runnable task)`或 `Executors.callable(Runnable task，Object resule)`）。

（2）然后可以把Runnable对象直接交给`ExecutorService`执行 （`ExecutorService.execute(Runnable command)`）；或者也可以把Runnable对象或Callable 对象提交给`ExecutorService`执行（`Executor-Service.submit(Runnable task)`或 `ExecutorService.submit(Callabletask)`）。

（3）如果执行 `ExecutorService.submit(...)`，`ExecutorService` 将返回一个实现`Future`接口的对象。由于 `FutureTask` 实现了Runnable，我们也可以创建 `FutureTask`，然后直接交给 `ExecutorService` 执行。

（4）最后，主线程可以执行 `FutureTask.get()`方法来等待任务执行完成。主线程也可以执行 `FutureTask.cancel(boolean mayInterruptIfRunning)`来取消此任务的执行。

## 2.3 Executor 框架成员

这里先了解一下Executor框架的主要成员：`Executor`、`ExecutorService`、`Future`接口、`Runnable`接口、`Callable`接口和工具类`Executors`。至于核心实现类`ThreadPoolExecutor`、 `ScheduledThreadPoolExecutor`稍后讲解。

### 2.3.1 Executor 与 ExecutorService

线程池从功能上看，Executor 就是一个任务执行器。它只有一个方法：`void execute(Runnable command);`，用来执行可运行的任务。

`ExecutorService`接口重要方法：

| **重要方法**                                     | 说明                                              |
| ------------------------------------------------ | ------------------------------------------------- |
| `void execute(Runnable command);`                | 执行可运行的任务                                  |
| `void shutdown();`                               | 关闭线程池<br/>停止接收新任务，原来的任务继续执行 |
| `List<Runnable> shutdownNow();`                  | 立即关闭<br/>停止接收新任务，原来的任务停止执行   |
| `Future<?> submit(Runnable task);`               | 提交任务; 允许获取执行结果                        |
| `<T> Future<T> submit(Runnable task, T result);` | 提交任务（指定结果）; 控制\|获取执行结果          |
| `<T> Future<T> submit(Callable<T> task);`        | 提交任务; 允许控制任务和获取执行结果              |
| `boolean awaitTermination(timeOut, unit);`       | 阻塞当前线程，返回是否线程都执行完                |

**需要注意submit()与 execute()方法区别**：

| 比较         | submit()                                                              | execute()                                     |
| ------------ | --------------------------------------------------------------------- | --------------------------------------------- |
| 有无返回值   | 有返回值，用 Future 封装<br/>根据返回值能判断任务是否被线程池成功执行 | 无返回值<br/>无法判断任务是否被成功线程池执行 |
| 能否捕获异常 | 可在主线程中 get 捕获到                                               | 不能捕获                                      |

### 2.3.2 Future接口

Future接口和实现Future接口的`FutureTask`类用来表示异步计算的结果。当我们把 Runnable接口或Callable接口的实现类提交（submit）给`ThreadPoolExecutor`或 `ScheduledThreadPoolExecutor`时，`ThreadPoolExecutor`或`ScheduledThreadPoolExecutor`会向我们返回一个`FutureTask`对象。下面是对应的API。

```java
<T> Future<T> submit(Callable<T> task)
<T> Future<T> submit(Runnable task, T result)
Future<> submit(Runnable task)
```

到目前最新的JDK 8为止，Java通过上述API返回的是一个 `FutureTask`对象。但从API可以看到，Java仅仅保证返回的是一个实现了`Future`接口的对象。

### 2.3.3 Runnable接口和Callable接口 

Runnable接口和Callable接口的实现类，都可以被`ThreadPoolExecutor`或`ScheduledThreadPoolExecutor`执行。它们之间的区别是Runnable不会返回结果，而Callable可以返回结果。

除了可以自己创建实现Callable接口的对象外，还可以使用工厂类Executors来把一个 Runnable包装成一个Callable。 下面是Executors提供的，把一个Runnable包装成一个Callable的API。

```java
public static Callable<Object> callable(Runnable task) // 假设返回对象Callable1
```

下面是Executors提供的，把一个Runnable和一个待返回的结果包装成一个Callable的 API。

```java
public static <T> Callable<T> callable(Runnable task, T result) // 假设返回对象Callable2
```

当我们把一个Callable对象（比如上面的Callable1或Callable2）提交给 `ThreadPoolExecutor`或`ScheduledThreadPoolExecutor`执行时，submit()会向我们返回一 个`FutureTask`对象。我们可以执行`FutureTask.get()`方法来等待任务执行完成。当任务成功完 成后`FutureTask.get()`将返回该任务的结果。例如，如果提交的是对象Callable1， `FutureTask.get()`方法将返回null；如果提交的是对象Callable2，`FutureTask.get()`方法将返回 result对象。

### 2.3.4 Executors创建线程池

在`java.util.concurrent.Executors`线程工厂类里面提供了一些静态工厂，实现了以下五种类型的 `ThreadPoolExecutor`：

| 类型                      | 特性                                                         |
| ------------------------- | ------------------------------------------------------------ |
| `newSingleThreadExecutor` | 创建一个单线程的线程池。这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。 |
| `newFixedThreadPool`      | 创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。 |
| `newCachedThreadPool`     | 创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。 |
| `newScheduledThreadPool`  | 创建一个大小无限的线程池，此线程池支持定时以及周期性执行任务的需求。 |
| `newWorkStealingPool()`   | Java 8 才加入这个线程池，其内部会构建`ForkJoinPool`，利用`Work-Stealing`算法，并行地处理任务，不保证处理顺序。 |

在生产环境下的实际场景中，一般不太推荐使用它们。因为选择使用 Executors 提供的工厂类实现的五种线程池，将会忽略很多线程池的参数设置，工厂类一旦选择设置默认参数，就很容易导致无法调优参数设置，从而产生性能问题或者资源浪费。这里建议使用 `ThreadPoolExecutor` 自我定制一套线程池。

# 3 ThreadPoolExecutor 详解

## 3.1 线程池状态转换

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

## 3.2 线程池处理流程

### 3.2.1 execute()方法

**为了搞懂线程池的原理，我们需要首先分析一下 `execute`方法**。下面来看看它的源码：

```java
	// 存放线程池的运行状态 (runState) 和线程池内有效线程的数量 (workerCount)
	private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));

    // 任务队列
	private final BlockingQueue<Runnable> workQueue;

	public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        /*
         * Proceed in 3 steps:
         *
         * 1. If fewer than corePoolSize threads are running, try to
         * start a new thread with the given command as its first
         * task.  The call to addWorker atomically checks runState and
         * workerCount, and so prevents false alarms that would add
         * threads when it shouldn't, by returning false.
         *
         * 2. If a task can be successfully queued, then we still need
         * to double-check whether we should have added a thread
         * (because existing ones died since last checking) or that
         * the pool shut down since entry into this method. So we
         * recheck state and if necessary roll back the enqueuing if
         * stopped, or start a new thread if there are none.
         *
         * 3. If we cannot queue task, then we try to add a new
         * thread.  If it fails, we know we are shut down or saturated
         * and so reject the task.
         */
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

### 3.2.2 Worker线程管理

#### 3.2.2.1 Worker线程

线程池为了掌握线程的状态并维护线程的生命周期，设计了线程池内的工作线程Worker。我们来看一下它的部分代码：

```Java
private final class Worker extends AbstractQueuedSynchronizer implements Runnable{
    final Thread thread;//Worker持有的线程
    Runnable firstTask;//初始化的任务，可以为null
}
```

Worker这个工作线程，实现了Runnable接口，并持有一个线程thread，一个初始化的任务`firstTask`。thread是在调用构造方法时通过`ThreadFactory`来创建的线程，可以用来执行任务；`firstTask`用它来保存传入的第一个任务，这个任务可以有也可以为null。如果这个值是非空的，那么线程就会在启动初期立即执行这个任务，也就对应核心线程创建时的情况；如果这个值是null，那么就需要创建一个线程去执行任务列表（`workQueue`）中的任务，也就是非核心线程的创建。

Worker执行任务的模型如下图所示：

<div align="center">  
<img src="https://p0.meituan.net/travelcube/03268b9dc49bd30bb63064421bb036bf90315.png" width="600px"/>
</div>

线程池需要管理线程的生命周期，需要在线程长时间不运行的时候进行回收。线程池使用一张Hash表去持有线程的引用，这样可以通过添加引用、移除引用这样的操作来控制线程的生命周期。这个时候重要的就是如何判断线程是否在运行。

Worker是通过继承AQS，使用AQS来实现独占锁这个功能。没有使用可重入锁`ReentrantLock`，而是使用AQS，为的就是实现不可重入的特性去反应线程现在的执行状态。

（1）lock方法一旦获取了独占锁，表示当前线程正在执行任务中。 （2）如果正在执行任务，则不应该中断线程。 （3）如果该线程现在不是独占锁的状态，也就是空闲的状态，说明它没有在处理任务，这时可以对该线程进行中断。（4）线程池在执行shutdown方法或`tryTerminate`方法时会调用`interruptIdleWorkers`方法来中断空闲的线程，`interruptIdleWorkers`方法会使用`tryLock`方法来判断线程池中的线程是否是空闲状态；如果线程是空闲状态则可以安全回收。

在线程回收过程中就使用到了这种特性，回收过程如下图所示：

<div align="center">  
<img src="https://p1.meituan.net/travelcube/9d8dc9cebe59122127460f81a98894bb34085.png" width="600px"/>
</div>

#### 3.2.2.2 Worker线程增加

增加线程是通过线程池中的`addWorker`方法，该方法的功能就是增加一个线程，该方法不考虑线程池是在哪个阶段增加的该线程，这个分配线程的策略是在上个步骤完成的，该步骤仅仅完成增加线程，并使它运行，最后返回是否成功这个结果。`addWorker`方法有两个参数：`firstTask`、core。`firstTask`参数用于指定新增的线程执行的第一个任务，该参数可以为空；core参数为true表示在新增线程时会判断当前活动线程数是否少于`corePoolSize`，false表示新增线程前需要判断当前活动线程数是否少于`maximumPoolSize`，其执行流程如下图所示：

<div align="center">  
<img src="https://p0.meituan.net/travelcube/49527b1bb385f0f43529e57b614f59ae145454.png" width="700px"/>
</div>

#### 3.2.2.3 Worker线程回收

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

#### 3.3.3.4 Worker线程执行任务

在Worker类中的run方法调用了`runWorker`方法来执行任务，`runWorker`方法的执行过程如下：

1. while循环不断地通过`getTask()`方法获取任务。
2. `getTask()`方法从阻塞队列中取任务。
3. 如果线程池正在停止，那么要保证当前线程是中断状态，否则要保证当前线程不是中断状态。 
4. 执行任务。 
5. 如果`getTask`结果为null则跳出循环，执行`processWorkerExit()`方法，销毁线程。

执行流程如下图所示：

<div align="center">  
<img src="https://p0.meituan.net/travelcube/879edb4f06043d76cea27a3ff358cb1d45243.png" width="600px"/>
</div>

## 3.2 线程池参数

### 3.2.1 线程池参数简述

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

我们还可以通过下面这张图来了解下线程池中各个参数的相互关系：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210619040444375.png" width="700px"/>
</div>

线程池有两个线程数的设置，一个为核心线程数，一个为最大线程数。如果当前的线程总个数 < `corePoolSize`，那么新建的线程为核心线程，如果当前线程总个数 >= `corePoolSize`，那么新建的线程为非核心线程。 核心线程默认会一直存活下去，即便是空闲状态，但是如果设置了`allowCoreThreadTimeOut(true)`的话，那么核心线程空闲时间达到`keepAliveTime`也将关闭。

在创建完线程池之后，默认情况下，线程池中并没有任何线程，**等到有任务**来才创建线程去执行任务。但有一种情况排除在外，就是调用 `prestartAllCoreThreads()` 或者 `prestartCoreThread()` 方法的话，可以提前创建等于核心线程数的线程数量，这种方式被称为**预热**，在抢购系统中就经常被用到。

当创建的线程数等于 `corePoolSize` 时，提交的任务会被加入到设置的阻塞队列中。当队列满了，会创建线程执行任务，直到线程池中的数量等于 `maximumPoolSize`。

当线程数量已经等于 `maximumPoolSize` 时， 新提交的任务无法加入到等待队列，也无法创建非核心线程直接执行，我们又没有为线程池设置拒绝策略，这时线程池就会抛出 `RejectedExecutionException` 异常，即线程池拒绝接受这个任务。

当线程池中创建的线程数量超过设置的 `corePoolSize`，在某些线程处理完任务后，如果等待 `keepAliveTime` 时间后仍然没有新的任务分配给它，那么这个线程将会被回收。线程池回收线程时，会对所谓的“核心线程”和“非核心线程”一视同仁，直到线程池中线程的数量等于设置的 `corePoolSize` 参数，回收过程才会停止。

### 3.3.1 拒绝策略

**`ThreadPoolExecutor` 拒绝策略定义：**

如果线程池中所有的线程都在忙碌，并且工作队列也满了（前提是工作队列是有界队列），那么此时提交任务，线程池就会拒绝接收。至于拒绝的策略，你可以通过 handler 这个参数来指定。

`ThreadPoolExecutor` 已经提供了以下 4 种策略。

* **`CallerRunsPolicy`**：提交任务的线程自己去执行该任务。也就是说，直接在调用`execute`方法的线程中运行(`run`)被拒绝的任务，如果执行程序已关闭，则会丢弃该任务。这种策略会降低对于新任务提交速度，影响程序的整体性能。如果您的应用程序可以承受此延迟并且你要求任何一个任务请求都要被执行的话，你可以选择这个策略。
* **`AbortPolicy`**：**默认的拒绝策略**，会 `throws RejectedExecutionException`拒绝新任务的处理。
* **`DiscardPolicy`**：直接丢弃任务，没有任何异常抛出。
* **`DiscardOldestPolicy`**：丢弃最老的任务，其实就是把最早进入工作队列的任务丢弃，然后把新任务加入到工作队列。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210619150145807.png" width="600px"/>
</div>

### 3.3.2 缓冲队列

`BlockingQueue` 是双缓冲队列。`BlockingQueue` 允许两个线程同时向队列一个存储，一个取出操作。在保证并发安全的同时，提高了队列的存取效率。常用的队列如下

* **`ArrayBlockingQueue`**：规定大小的 `BlockingQueue`，其构造必须指定大小。其所含的对象是 FIFO 顺序排序的。

* **`LinkedBlockingQueue`**：大小不固定的 `BlockingQueue`，若其构造时指定大小，生成的`BlockingQueue` 有大小限制，不指定大小，其大小有 `Integer.MAX_VALUE` 来决定。其所含的对象是 FIFO 顺序排序的。

* **`PriorityBlockingQueue`**：类似于 `LinkedBlockingQueue`，但是其所含对象的排序不是 FIFO，而是依据对象的自然顺序或者构造函数的 Comparator 决定。
* **`SynchronousQueue`**：特殊的 `BlockingQueue`，对其的操作必须是放和取交替完成，它支持公平访问队列。

# 4 ScheduledThreadPoolExecutor 详解

`ScheduledThreadPoolExecutor` 主要用来在给定的**延迟后运行任务，或者定期执行任务**。`ScheduledThreadPoolExecutor` 使用的任务队列 `DelayQueue` 封装了一个 `PriorityQueue`，`PriorityQueue` 会对队列中的任务进行排序，执行所需时间短的放在前面先被执行(`ScheduledFutureTask` 的 `time` 变量小的先执行)，如果执行所需时间相同则先提交的任务将被先执行(`ScheduledFutureTask` 的 `squenceNumber` 变量小的先执行)。

**`ScheduledThreadPoolExecutor` 和 `Timer` 的比较：**

- `Timer` 对系统时钟的变化敏感，`ScheduledThreadPoolExecutor`不是；
- `Timer` 只有一个执行线程，因此长时间运行的任务可以延迟其他任务。 `ScheduledThreadPoolExecutor` 可以配置任意数量的线程。 此外，可以通过提供 `ThreadFactory`完全控制创建的线程;
- 在`TimerTask` 中抛出的运行时异常会杀死一个线程，从而导致 `Timer` 死机:-( ...即计划任务将不再运行。`ScheduledThreadExecutor` 不仅捕获运行时异常，还允许您在需要时处理它们（通过重写 `afterExecute` 方法`ThreadPoolExecutor`）。抛出异常的任务将被取消，但其他任务将继续运行。

在 JDK1.5 之后，你没有理由再使用 Timer 进行任务调度了。

## 4.2 ScheduledThreadPoolExecutor 运行机制

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210619151335333.png" width="600px"/>
</div>

**`ScheduledThreadPoolExecutor` 的执行主要分为两大部分：**

1. 当调用 `ScheduledThreadPoolExecutor` 的 **`scheduleAtFixedRate()`** 方法或者**`scheduleWirhFixedDelay()`** 方法时，会向 `ScheduledThreadPoolExecutor` 的 **`DelayQueue`** 添加一个实现了 **`RunnableScheduledFuture`** 接口的 **`ScheduledFutureTask`** 。
2. 线程池中的线程从 `DelayQueue` 中获取 `ScheduledFutureTask`，然后执行任务。

**`ScheduledThreadPoolExecutor` 为了实现周期性的执行任务，对 `ThreadPoolExecutor`做了如下修改：**

- 使用 **`DelayQueue`** 作为任务队列；
- 获取任务的方不同；
- 执行周期任务后，增加了额外的处理。

## 4.3 ScheduledThreadPoolExecutor 执行周期任务的步骤

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210619151437709.png" width="600px"/>
</div>

1. 线程 1 从 `DelayQueue` 中获取已到期的 `ScheduledFutureTask（DelayQueue.take()）`。到期任务是指 `ScheduledFutureTask`的 time 大于等于当前系统的时间；
2. 线程 1 执行这个 `ScheduledFutureTask`；
3. 线程 1 修改 `ScheduledFutureTask` 的 time 变量为下次将要被执行的时间；
4. 线程 1 把这个修改 time 之后的 `ScheduledFutureTask` 放回 `DelayQueue` 中（`DelayQueue.add()`)。

# 5 常见线程池

## 5.1 FixedThreadPool

### 5.1.1 FixedThreadPool 处理流程

`FixedThreadPool` 被称为**可重用固定线程数的线程池**。通过 Executors 类中的相关源代码来看一下相关实现：

```java
    public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>(),
                                      threadFactory);
    }

    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

从上面源代码可以看出新创建的 `FixedThreadPool` 的 `corePoolSize` 和 `maximumPoolSize` 都被设置为 `nThreads`，这个 `nThreads` 参数是我们使用的时候自己传递的。

`FixedThreadPool` 的 `execute()` 方法运行示意图：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/202106191518384.png" width="600px"/>
</div>

**上图说明：**

1.  如果当前运行的线程数小于 `corePoolSize`， 如果再来新任务的话，就创建新的线程来执行任务；
2.  当前运行的线程数等于 `corePoolSize` 后， 如果再来新任务的话，会将任务加入 `LinkedBlockingQueue`；
3.  线程池中的线程执行完手头的任务后，会在循环中反复从 `LinkedBlockingQueue` 中获取任务来执行；

### 5.1.2 为什么不推荐使用`FixedThreadPool`？

**`FixedThreadPool` 使用无界队列 `LinkedBlockingQueue`（队列的容量为 `Intger.MAX_VALUE`）作为线程池的工作队列会对线程池带来如下影响 ：**

1. 当线程池中的线程数达到 `corePoolSize` 后，新任务将在无界队列中等待，因此线程池中的线程数不会超过 `corePoolSize`；
2. 由于使用无界队列时 `maximumPoolSize` 将是一个无效参数，因为不可能存在任务队列满的情况。所以，通过创建 `FixedThreadPool`的源码可以看出创建的 `FixedThreadPool` 的 `corePoolSize` 和 `maximumPoolSize` 被设置为同一个值。
3. 由于 1 和 2，使用无界队列时 `keepAliveTime` 将是一个无效参数；
4. 运行中的 `FixedThreadPool`（未执行 `shutdown()`或 `shutdownNow()`）不会拒绝任务，在任务比较多的时候会导致 OOM（内存溢出）。

## 5.2 SingleThreadExecutor 详解

### 5.2.1 SingleThreadExecutor 处理流程

`SingleThreadExecutor` 是只有一个线程的线程池。下面看看**SingleThreadExecutor 的实现：**

```java
    public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>(),
                                    threadFactory));
    }

	public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```

从上面源代码可以看出新创建的 `SingleThreadExecutor` 的 `corePoolSize` 和 `maximumPoolSize` 都被设置为 1。其他参数和 `FixedThreadPool` 相同。

`SingleThreadExecutor` 的运行示意图：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210619152213331.png" width="600px"/>
</div>

**上图说明：**

1. 如果当前运行的线程数少于 `corePoolSize`，则创建一个新的线程执行任务；
2. 当前线程池中有一个运行的线程后，将任务加入 `LinkedBlockingQueue`
3. 线程执行完当前的任务后，会在循环中反复从`LinkedBlockingQueue` 中获取任务来执行；

### 5.2.2 为什么不推荐使用`SingleThreadExecutor`？

`SingleThreadExecutor` 使用无界队列 `LinkedBlockingQueue` 作为线程池的工作队列（队列的容量为 `Intger.MAX_VALUE`）。`SingleThreadExecutor` 使用无界队列作为线程池的工作队列会对线程池带来的影响与 `FixedThreadPool` 相同。说简单点就是可能会导致 OOM。	

## 5.3 CachedThreadPool 详解

### 5.3.1 CachedThreadPool处理流程

`CachedThreadPool` 是一个会根据需要创建新线程的线程池。下面通过源码来看看 `CachedThreadPool` 的实现：

```java
    public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>(),
                                      threadFactory);
    }

    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

`CachedThreadPool` 的`corePoolSize` 被设置为空（0），`maximumPoolSize`被设置为 `Integer.MAX.VALUE`，即它是无界的，这也就意味着如果主线程提交任务的速度高于 `maximumPool` 中线程处理任务的速度时，`CachedThreadPool` 会不断创建新的线程。极端情况下，这样会导致耗尽CPU和内存资源。

`CachedThreadPool` 的 execute()方法的执行示意图：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210619152425774.png" width="600px"/>
</div>

**上图说明：**

1. 首先执行 `SynchronousQueue.offer(Runnable task)` 提交任务到任务队列。如果当前 `maximumPool` 中有闲线程正在执行 `SynchronousQueue.poll(keepAliveTime,TimeUnit.NANOSECONDS)`，那么主线程执行 offer 操作与空闲线程执行的 `poll` 操作配对成功，主线程把任务交给空闲线程执行，`execute()`方法执行完成，否则执行下面的步骤 2；
2. 当初始 `maximumPool` 为空，或者 `maximumPool` 中没有空闲线程时，将没有线程执行 `SynchronousQueue.poll(keepAliveTime,TimeUnit.NANOSECONDS)`。这种情况下，步骤 1 将失败，此时 `CachedThreadPool` 会创建新线程执行任务，execute 方法执行完成；

### 5.3.2 为什么不推荐使用`CachedThreadPool`？

`CachedThreadPool`允许创建的线程数量为 `Integer.MAX_VALUE` ，可能会创建大量线程，从而导致 OOM。

# 6 线程池实践

## 6.1 线程池数量计算: CPU密集型 VS I/O密集型

线程池数量计算一直是困扰程序员的一个难题，创建线程池大小时，并不说越大越好，当然太小肯定也不好。

举个例子，假如要盖一套房子，1个人工作需要花100天。为了缩短工期，肯定要增加合作的人数。那直接增加到100人，就可以1天就盖好房子？肯定不是的，人越多沟通成本越高，消耗资源越多。就跟多线程并发编程场景一样，很多时候，过多的线程只会徒增资源的开销，增加了上下文切换成本。

当然，环境也具有多变性，设置一个绝对精准的线程数其实是不大可能的，但计算出一个合理的线程数，避免由于线程池设置不合理而导致的性能问题还是有迹可循的。下面我们就来看看具体的计算方法。

一般多线程执行的任务类型可以分为 CPU 密集型和 I/O 密集型，根据不同的任务类型，我们计算线程数的方法也不一样。

**CPU 密集型任务**：这种任务消耗的主要是 CPU 资源，可以将线程数设置为 **N（CPU 核心数）+1**，比 CPU 核心数多出来的一个线程是为了防止线程偶发的缺页中断，或者其它原因导致的任务暂停而带来的影响。一旦任务暂停，CPU 就会处于空闲状态，而在这种情况下多出来的一个线程就可以充分利用 CPU 的空闲时间。

**I/O 密集型任务**：这种任务应用起来，系统会用大部分的时间来处理 I/O 交互，而线程在处理 I/O 的时间段内不会占用 CPU 来处理，这时就可以将 CPU 交出给其它线程使用。因此在 I/O 密集型任务的应用中，我们可以多配置一些线程，具体的计算方法是 **2N**。

**如何判断是 CPU 密集任务还是 IO 密集任务？**

CPU 密集型简单理解就是利用 CPU 计算能力的任务比如你在内存中对大量数据进行排序。单凡涉及到网络读取，文件读取这类都是 IO 密集型，这类任务的特点是 CPU 计算耗费时间相比于等待 IO 操作完成的时间来说很少，大部分时间都花在了等待 IO 操作完成上。

在4 核 intel i5 CPU 机器上对[Github](https://github.com/nickliuchao/threadpollsizetest/tree/master/src/main/java/com/demo/threadpoolsize)代码进行了测试：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210619161631641.png" width="600px"/>
</div>

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210619161657831.png" width="600px"/>
</div>

可以看到，在CPU密集型任务中，当线程数量太小，同一时间大量请求将被阻塞在线程队列中排队等待执行线程，此时 CPU 没有得到充分利用；当线程数量太大，被创建的执行线程同时在争取 CPU 资源，又会导致大量的上下文切换，从而增加线程的执行时间，影响了整体执行效率。通过测试可知，4~6 个线程数是最合适的。

在IO密集型任务中，看到每个线程所花费的时间。当线程数量在 8 时，线程平均执行时间是最佳的，这个线程数量和我们的计算公式所得的结果就差不多。

在平常的应用场景中，我们常常遇不到这两种极端情况，**那么碰上一些常规的业务操作，比如，通过一个线程池实现向用户定时推送消息的业务，我们又该如何设置线程池的数量呢？**

此时我们可以参考以下公式来计算线程数：

```java
线程数=N（CPU核数）*（1+WT（线程等待时间）/ST（线程时间运行时间））
```

我们可以通过 JDK 自带的工具 `VisualVM` 来查看 `WT/ST` 比例，以下例子是基于运行纯 `CPU` 运算的例子，我们可以看到：

```java
WT（线程等待时间）= 36788ms [线程运行总时间] - 36788ms[ST（线程时间运行时间）]= 0
线程数=N（CPU核数）*（1+ 0 [WT（线程等待时间）]/36788ms[ST（线程时间运行时间）]）= N（CPU核数）
```

这跟之前通过 CPU 密集型的计算公式 N+1 所得出的结果差不多。

综合来看，我们可以根据自己的业务场景，从“N+1”和“2N”两个公式中选出一个适合的，计算出一个大概的线程数量，之后通过实际压测，逐渐往“增大线程数量”和“减小线程数量”这两个方向调整，然后观察整体的处理时间变化，最终确定一个具体的线程数量。

## 6.2 线程池监控

除了参数动态化之外，为了更好地使用线程池，我们需要对线程池的运行状况有感知，比如当前线程池的负载是怎么样的？分配的资源够不够用？任务的执行情况是怎么样的？是长任务还是短任务？基于对这些问题的思考，动态化线程池提供了多个维度的监控和告警能力，包括：线程池活跃度、任务的执行Transaction（频率、耗时）、Reject异常、线程池内部统计信息等等，既能帮助用户从多个维度分析线程池的使用情况，又能在出现问题第一时间通知到用户，从而避免故障或加速故障恢复。

### 6.2.1 负载监控和告警

线程池负载关注的核心问题是：基于当前线程池参数分配的资源够不够。对于这个问题，我们可以从事前和事中两个角度来看。事前，线程池定义了“活跃度”这个概念，来让用户在发生Reject异常之前能够感知线程池负载问题，线程池活跃度计算公式为：`线程池活跃度 = activeCount/maximumPoolSize`。这个公式代表当活跃线程数趋向于`maximumPoolSize`的时候，代表线程负载趋高。事中，也可以从两方面来看线程池的过载判定条件，一个是发生了Reject异常，一个是队列中有等待任务（支持定制阈值）。以上两种情况发生了都会触发告警，告警信息会通过大象推送给服务所关联的负责人。

<div align="center">  
<img src="https://p0.meituan.net/travelcube/04e73f7186a91d99181e1b5615ce9e4a318600.png" width="600px"/>
</div>

### 6.2.2 任务级精细化监控

在传统的线程池应用场景中，线程池中的任务执行情况对于用户来说是透明的。比如在一个具体的业务场景中，业务开发申请了一个线程池同时用于执行两种任务，一个是发消息任务、一个是发短信任务，这两类任务实际执行的频率和时长对于用户来说没有一个直观的感受，很可能这两类任务不适合共享一个线程池，但是由于用户无法感知，因此也无从优化。动态化线程池内部实现了任务级别的埋点，且允许为不同的业务任务指定具有业务含义的名称，线程池内部基于这个名称做Transaction打点，基于这个功能，用户可以看到线程池内部任务级别的执行情况，且区分业务，任务监控示意图如下图所示：

<div align="center">  
<img src="https://p1.meituan.net/travelcube/cd0b9445c3c93a866201b7cfb24d2ce7214776.png" width="800px"/>
</div>

### 6.2.3 运行时状态实时查看

用户基于JDK原生线程池`ThreadPoolExecutor`提供的几个public的getter方法，可以读取到当前线程池的运行状态以及参数，如下图所示：

<div align="center">  
<img src="https://p0.meituan.net/travelcube/aba8d9c09e6f054c7061ddd720a04a26147951.png" width="800px"/>
</div>

动态化线程池基于这几个接口封装了运行时状态实时查看的功能，用户基于这个功能可以了解线程池的实时状态，比如当前有多少个工作线程，执行了多少个任务，队列中等待的任务数等等。效果如下图所示：

<div align="center">  
<img src="https://p1.meituan.net/travelcube/38d5fbeaebd4998f3a30d44bd20b996f113233.png" width="600px"/>
</div>

## 6.3 线程池命名

初始化线程池的时候需要显示命名（设置线程池名称前缀），**有利于定位问题**。默认情况下创建的线程名字类似 `pool-1-thread-n` 这样的，没有业务含义，不利于我们定位问题。

给线程池里的线程命名通常有下面两种方式：

**1. 利用 guava 的 `ThreadFactoryBuilder` **

```java
    ThreadFactory threadFactory = new ThreadFactoryBuilder()
                            .setNameFormat(threadNamePrefix + "-%d")
                            .setDaemon(true).build();
    ExecutorService threadPool = new ThreadPoolExecutor(corePoolSize, maximumPoolSize
                                                        , keepAliveTime, TimeUnit.MINUTES
                                                        , workQueue, threadFactory)
```

**2. 自己实现 `ThreadFactor`。**

```java
import java.util.concurrent.Executors;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 线程工厂，它设置线程名称，有利于我们定位问题。
 */
public final class NamingThreadFactory implements ThreadFactory {

    private final AtomicInteger threadNum = new AtomicInteger();
    private final ThreadFactory delegate;
    private final String name;

    /**
     * 创建一个带名字的线程池生产工厂
     */
    public NamingThreadFactory(ThreadFactory delegate, String name) {
        this.delegate = delegate;
        this.name = name; // TODO consider uniquifying this
    }

    @Override
    public Thread newThread(Runnable r) {
        Thread t = delegate.newThread(r);
        t.setName(name + " [#" + threadNum.incrementAndGet() + "]");
        return t;
    }
}
```

## 6.4 动态化线程池

### 6.4.1 整体设计

动态化线程池的核心设计包括以下三个方面：

1. 简化线程池配置：线程池构造参数有8个，但是最核心的是3个：`corePoolSize`、`maximumPoolSize`，`workQueue`，它们最大程度地决定了线程池的任务分配和线程分配策略。考虑到在实际应用中我们获取并发性的场景主要是两种：（1）并行执行子任务，提高响应速度。这种情况下，应该使用同步队列，没有什么任务应该被缓存下来，而是应该立即执行。（2）并行执行大批次任务，提升吞吐量。这种情况下，应该使用有界队列，使用队列去缓冲大批量的任务，队列容量必须声明，防止任务无限制堆积。所以线程池只需要提供这三个关键参数的配置，并且提供两种队列的选择，就可以满足绝大多数的业务需求，Less is More。
2. 参数可动态修改：为了解决参数不好配，修改参数成本高等问题。在Java线程池留有高扩展性的基础上，封装线程池，允许线程池监听同步外部的消息，根据消息进行修改配置。将线程池的配置放置在平台侧，允许开发同学简单的查看、修改线程池配置。
3. 增加线程池监控：对某事物缺乏状态的观测，就对其改进无从下手。在线程池执行任务的生命周期添加监控能力，帮助开发同学了解线程池状态。

<div align="center">  
<img src="https://p1.meituan.net/travelcube/4d5c410ad23782350cc9f980787151fd54144.png" width="800px"/>
</dv>

### 6.4.2 功能架构

动态化线程池提供如下功能：

**动态调参**：支持线程池参数动态调整、界面化操作；包括修改线程池核心大小、最大核心大小、队列长度等；参数修改后及时生效。 **任务监控**：支持应用粒度、线程池粒度、任务粒度的Transaction监控；可以看到线程池的任务执行情况、最大任务执行时间、平均任务执行时间、95/99线等。 **负载告警**：线程池队列任务积压到一定值的时候会通过大象（美团内部通讯工具）告知应用开发负责人；当线程池负载数达到一定阈值的时候会通过大象告知应用开发负责人。 **操作监控**：创建/修改和删除线程池都会通知到应用的开发负责人。 **操作日志**：可以查看线程池参数的修改记录，谁在什么时候修改了线程池参数、修改前的参数值是什么。 **权限校验**：只有应用开发负责人才能够修改应用的线程池参数。

<div align="center">  
<img src="https://p0.meituan.net/travelcube/6c0091e92e90f50f89fd83f3b9eb5472135718.png" width="800px"/>
</dv>

**参数动态化**

JDK原生线程池`ThreadPoolExecutor`提供了如下几个public的setter方法，如下图所示：

<div align="center">  
<img src="https://p1.meituan.net/travelcube/efd32f1211e9cf0a3ca9d35b0dc5de8588353.png" width="800px"/>
</dv>

JDK允许线程池使用方通过`ThreadPoolExecutor`的实例来动态设置线程池的核心策略，以`setCorePoolSize`为方法例，在运行期线程池使用方调用此方法设置`corePoolSize`之后，线程池会直接覆盖原来的`corePoolSize`值，并且基于当前值和原始值的比较结果采取不同的处理策略。对于当前值小于当前工作线程数的情况，说明有多余的worker线程，此时会向当前idle的worker线程发起中断请求以实现回收，多余的worker在下次`idel`的时候也会被回收；对于当前值大于原始值且当前队列中有待执行任务，则线程池会创建新的worker线程来执行队列任务，`setCorePoolSize`具体流程如下：

<div align="center">  
<img src="https://p0.meituan.net/travelcube/9379fe1666818237f842138812bf63bd85645.png" width="600px"/>
</dv>

线程池内部会处理好当前状态做到平滑修改，其他几个方法限于篇幅，这里不一一介绍。重点是基于这几个public方法，我们只需要维护`ThreadPoolExecutor`的实例，并且在需要修改的时候拿到实例修改其参数即可。基于以上的思路，我们实现了线程池参数的动态化、线程池参数在管理平台可配置可修改，其效果图如下图所示：

<div align="center">  
<img src="https://p0.meituan.net/travelcube/414ba7f3abd11e5f805c58635ae10988166121.png" width="600px"/>
</dv>

用户可以在管理平台上通过线程池的名字找到指定的线程池，然后对其参数进行修改，保存后会实时生效。目前支持的动态参数包括核心数、最大值、队列长度等。除此之外，在界面中，我们还能看到用户可以配置是否开启告警、队列等待任务告警阈值、活跃度告警等等。关于监控和告警，我们下面一节会对齐进行介绍。

# 总结

在《阿里巴巴 Java 开发手册》“并发处理”这一章节，明确指出线程资源必须通过线程池提供，不允许在应用中自行显示创建线程。使用线程池会带来的好处有：（1）降低资源消耗；（2）提高响应速度；（3）提高线程的可管理性。

Executor框架中`ScheduledThreadPoolExecutor` 用来定时执行任务， `ThreadPoolExecutor` 用来执行被提交的任务。其中线程池实现类 `ThreadPoolExecutor` 是 `Executor` 框架最核心的类，在日常开发中应用较多。

另外，《阿里巴巴 Java 开发手册》中强制线程池不允许使用 Executors 去创建，而是通过 `ThreadPoolExecutor` 构造函数的方式。这时因为Executors创建线程池会有资源耗尽的风险。

> - **`FixedThreadPool` 和 `SingleThreadExecutor`** ： 允许请求的队列长度为 `Integer.MAX_VALUE`，可能堆积大量的请求，从而导致 OOM。
> - **`CachedThreadPool` 和 `ScheduledThreadPool`** ： 允许创建的线程数量为 `Integer.MAX_VALUE`，可能会创建大量线程，从而导致 OOM。

因此，使用线程池时，我们一定要根据场景和需求**配置合理的线程数、任务队列、拒绝策略、线程回收策略**，并对线程进行明确的命名方便排查问题。

# 参考资料

- 《Java 并发编程的艺术》
- [如何设置线程池大小？](https://time.geekbang.org/column/article/104094)
- [Executor与线程池：如何创建正确的线程池？](https://time.geekbang.org/column/article/90771)
- [线程池：业务代码最常用也最容易犯错的组件](https://time.geekbang.org/column/article/210337)
- [图解Java线程池原理](https://juejin.cn/post/6844903920444112904)
- [Java线程池实现原理及其在美团业务中的实践](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)
- [线程池最佳实践！安排！](https://juejin.cn/post/6844904186400899086)

