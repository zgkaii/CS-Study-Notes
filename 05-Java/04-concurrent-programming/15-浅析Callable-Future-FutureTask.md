<!-- MarkdownTOC -->
- [浅析Callable/Future/FutureTask](#浅析callablefuturefuturetask)
  - [1. Runnable](#1-runnable)
  - [2. Callable](#2-callable)
  - [3. Future](#3-future)
  - [4. FutureTask](#4-futuretask)
    - [4.1 概述](#41-概述)
    - [4.2 源码分析](#42-源码分析)
  - [参考资料](#参考资料)

<!-- /MarkdownTOC -->

## 浅析Callable/Future/FutureTask

[多线程基础](https://blog.csdn.net/KAIZ_LEARN/article/details/108890366)一文中，描述了创建线程主要有两种方式，一种是直接继承Thread类，另外一种就是实现Runnable接口。

这两种方式都有一个缺陷就是：**在执行完任务之后无法获取执行结果**。如果需要获取执行结果，就必须通过共享变量或者使用线程通信的方式来达到效果。

Java 1.5 开始，就提供了Callable、Future接口和FutureTask方法，通过它们可以在任务执行完毕之后得到任务执行结果。

### 1. Runnable

Runnable 是一个接口，只有一个抽象方法：`run()`，run()方法使用void修饰的，方法不支持返回值。

```java
public interface Runnable {
    public abstract void run();
}
```

Runnable对象可以传入到Thread类的构造方法中，通过Thread来运行Runnable任务。

```java
public class RunnableTest {
    public static void main(String[] args) {
        new Thread(new Runnable() {
            // 重写抽象方法
            @Override
            public void run() {
                System.out.println("Hello World!");
            }
        }).start();// 启动线程
    }
}
```

### 2. Callable

Callable与Runnable类似，它也是一个接口，也只有一个方法：`call()`，不同的是Callable的call()方法是有返回值的，返回值的类型是一个泛型，泛型在创建Callable对象时指定。某种程度上，Callable可以看作是Runnable接口的补充。

```java
public interface Callable<V> {
    V call() throws Exception;// 说明能受检异常
}
```

Callable接口不能直接传入到Thread中来运行，它通常结合线程池来使用。一般情况下是配合ExecutorService来使用的。可以看到，在ExecutorService接口中声明了若干个submit方法的重载版本。

<img src="https://img-blog.csdnimg.cn/20201022234728698.png" style="zoom:80%;" />

可见，使用ExecutorService 的 `execute()` 方法不会return返回值，而 `submit()` 方法都return `Future` 类型的返回值。

简要归纳一下，Runnable与Callable主要有以下区别：

| 对比                  | Runnable | Callable |
| --------------------- | -------- | -------- |
| 方法返回值            | 没有     | 有       |
| 受检异常              | 无法处理 | 可以处理 |
| Thread类中使用        | 可以     | 不可以   |
| ExecutorService中使用 | 可以     | 可以     |

上面提到，调用submit()方法都会产生一个`Future` 类型的返回值。那什么是Future呢？怎么通过它获取返回值呢？

### 3. Future

查看源码，发现Future是一个接口，接口中声明了5个方法：

```java
public interface Future<V> {
    // 取消任务，取消任务成功则返回true。
    boolean cancel(boolean mayInterruptIfRunning);
    // 判断任务是否已经取消，任务正常完成前被取消成功，则返回true。
    boolean isCancelled();
    // 判断任务是否已经完成，任务已经完成，则返回true。
    boolean isDone();
    // 获取任务执行结果，会产生阻塞，会一直等到任务执行完毕才返回结果。
    V get() throws InterruptedException, ExecutionException;
    // 获取任务执行结果，带有超时时间限制（在指定时间内，还没获取到结果，就直接返回null）。
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

也就是说，Future主要提供了三种功能：　　

* 判断任务是否完成
* 中断任务
* 获取任务执行结果

可以把Future理解为Callable任务返回给调用方这么一个句柄，通过这个句柄我们可以跟这个异步任务联系起来，对任务进行查询、取消、执行结果的获取的操作。也就是说，Future是调用方与异步执行方之间沟通的桥梁。

使用案例：

```java
@Slf4j
public class CallableTest {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newSingleThreadExecutor();

        log.info("提交Callable到线程池");
        Future<Integer> future = executorService.submit(new CallableTask());

        try {
            long startTime = System.currentTimeMillis();
            log.info("主线程等待获取 Future 结果");
            while (!future.isDone()) {
                log.info("子线程还未完成");
                Thread.sleep(2000);
            }
            if (!future.isCancelled()) {
                log.info("子线程任务已完成");
                log.info("主线程获取到 Future 结果: {}", future.get());
            } else {
                log.warn("子线程任务被取消");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
        executorService.shutdown();
    }

    static class CallableTask implements Callable<Integer> {
        @Override
        public Integer call() throws Exception {
            log.info("子线程任务开始");
            Thread.sleep(3000);
            return new Random().nextInt();
        }
    }
}
```

### 4. FutureTask 

了解FutureTask 之前，我们先来了解上面提到ExecutorService中对应有三种submit方法：

```java
	Future<?> submit(Runnable task);
	<T> Future<T> submit(Callable<T> task);
    <T> Future<T> submit(Runnable task, T result);// 一般不使用
```

AbstractExecutorService中对应的实现方法：

```java
    public Future<?> submit(Runnable task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<Void> ftask = newTaskFor(task, null);
        execute(ftask);
        return ftask;
    }

    public <T> Future<T> submit(Callable<T> task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<T> ftask = newTaskFor(task);
        execute(ftask);
        return ftask;
    }
```

可见，submit(Runnable)和submit(Callable)方法的实现步骤都是：

1. 判空
2. 生成RunnableFuture对象
3. 调用execute()方法

查看RunnableFuture，发现其继承了 `Runnable` 和 `Future` 接口。

```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    /**
     * Sets this Future to the result of its computation
     * unless it has been cancelled.
     */
    void run();
}
```

再查看newTaskFor()方法：

```java
    protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
        return new FutureTask<T>(callable);
    }
```

也就是说，RunnableFuture接口同时继承了 `Runnable` 和 `Future` 接口，也就同时拥有了这两个接口的特性，而这些特性的具体实现是由FutureTask完成的。

#### 4.1 概述

`FutureTask` 类实现了 `RunnableFuture` 接口，而  `RunnableFuture` 接口又继承了 `Runnable` 和 `Future` 接口。可以推断出 `FutureTask` 同时具有这两种接口的特性：

- 有 `Runnable` 特性，所以可以用在 `ExecutorService` 中配合线程池使用。
- 有 `Future` 特性，所以可以从中获取到执行结果。

<img src="https://img-blog.csdnimg.cn/20201023002648638.png" style="zoom:80%;" />

#### 4.2 源码分析

```java
    public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();
        this.callable = callable;
        this.state = NEW;       // ensure visibility of callable
    }

    public FutureTask(Runnable runnable, V result) {
        this.callable = Executors.callable(runnable, result);
        this.state = NEW;       // ensure visibility of callable
    }
```

可见，即便在 FutureTask 构造方法中传入的是 Runnable 形式的线程，该构造方法也会通过 `Executors.callable` 工厂方法将其转换为 Callable 类型。

FutureTask 实现的是 Runnable 接口，也就是只能重写 run() 方法。

```java
public void run() {
    // 如果状态不是 NEW，说明任务已经执行过或者已经被取消，直接返回
    // 如果状态是 NEW，则尝试把执行线程保存在 runnerOffset（runner字段），如果赋值失败，则直接返回
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                     null, Thread.currentThread()))
        return;
    try {
        // 获取构造函数传入的 Callable 值
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                // 正常调用 Callable 的 call 方法就可以获取到返回值
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                 // 保存 call 方法抛出的异常
                setException(ex);
            }
            if (ran)
                // 保存 call 方法的执行结果
                set(result);
        }
    } finally {
        runner = null;
		// 如果任务被中断，则执行中断处理
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
```

`run()` 方法没有返回值，至于 `run()` 方法是如何将 `call()` 方法的返回结果和异常都保存起来的呢？其实非常简单, 就是通过 set(result) 保存正常程序运行结果，或通过 setException(ex) 保存程序异常信息

```java
/** The result to return or exception to throw from get() */
private Object outcome; // non-volatile, protected by state reads/writes

// 保存异常结果
protected void setException(Throwable t) {
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        outcome = t;
        UNSAFE.putOrderedInt(this, stateOffset, EXCEPTIONAL); // final state
        finishCompletion();
    }
}

// 保存正常结果
protected void set(V v) {
  if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
    outcome = v;
    UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state
    finishCompletion();
  }
}
```

`setException` 和 `set` 方法非常相似，都是将异常或者结果保存在 `Object` 类型的 `outcome` 变量中，`outcome` 是成员变量，就要考虑线程安全，所以他们要通过 CAS方式设置 outcome 变量的值，既然是在 CAS 成功后 更改 outcome 的值，这也就是 outcome 没有被 `volatile` 修饰的原因所在。

保存正常结果值（set方法）与保存异常结果值（setException方法）两个方法代码逻辑，唯一的不同就是 CAS 传入的 state 不同。我们上面提到，state 多数用于控制代码逻辑，FutureTask 也是这样，所以要搞清代码逻辑，我们需要先对 state 的状态变化有所了解

```java
 /*
 *
 * Possible state transitions:
 * NEW -> COMPLETING -> NORMAL  //执行过程顺利完成
 * NEW -> COMPLETING -> EXCEPTIONAL //执行过程出现异常
 * NEW -> CANCELLED // 执行过程中被取消
 * NEW -> INTERRUPTING -> INTERRUPTED //执行过程中，线程被中断
 */
private volatile int state;
private static final int NEW          = 0;
private static final int COMPLETING   = 1;
private static final int NORMAL       = 2;
private static final int EXCEPTIONAL  = 3;
private static final int CANCELLED    = 4;
private static final int INTERRUPTING = 5;
private static final int INTERRUPTED  = 6;
```

FutureTask 对象被创建出来，state 的状态就是 NEW 状态，从上面的构造函数中你应该已经发现了，四个最终状态 NORMAL ，EXCEPTIONAL ， CANCELLED ， INTERRUPTED 也都很好理解，两个中间状态稍稍有点让人困惑:

- COMPLETING: outcome 正在被set 值的时候
- INTERRUPTING：通过 cancel(true) 方法正在中断线程的时候

总的来说，这两个中间状态都表示一种瞬时状态，我们将几种状态图形化展示一下：

![](https://img-blog.csdnimg.cn/20201028235637379.png)

我们知道了 run() 方法是如何保存结果的，以及知道了将正常结果/异常结果保存到了 outcome 变量里，那就需要看一下 FutureTask 是如何通过 get() 方法获取结果的：

```java
public V get() throws InterruptedException, ExecutionException {
    int s = state;
   // 如果 state 还没到 set outcome 结果的时候，则调用 awaitDone() 方法阻塞自己
    if (s <= COMPLETING)
        s = awaitDone(false, 0L);
   // 返回结果
    return report(s);
}
```

awaitDone 方法是 FutureTask 最核心的一个方法

```java
// get 方法支持超时限制，如果没有传入超时时间，则接受的参数是 false 和 0L
// 有等待就会有队列排队或者可响应中断，从方法签名上看有 InterruptedException，说明该方法这是可以被中断的
private int awaitDone(boolean timed, long nanos)
    throws InterruptedException {
   // 计算等待截止时间
    final long deadline = timed ? System.nanoTime() + nanos : 0L;
    WaitNode q = null;
    boolean queued = false;
    for (;;) {
       // 如果当前线程被中断，如果是，则在等待对立中删除该节点，并抛出 InterruptedException
        if (Thread.interrupted()) {
            removeWaiter(q);
            throw new InterruptedException();
        }

        int s = state;
       // 状态大于 COMPLETING 说明已经达到某个最终状态（正常结束/异常结束/取消）
       // 把 thread 只为空，并返回结果
        if (s > COMPLETING) {
            if (q != null)
                q.thread = null;
            return s;
        }
       // 如果是COMPLETING 状态（中间状态），表示任务已结束，但 outcome 赋值还没结束，这时主动让出执行权，让其他线程优先执行（只是发出这个信号，至于是否别的线程执行一定会执行可是不一定的）
        else if (s == COMPLETING) // cannot time out yet
            Thread.yield();
       // 等待节点为空
        else if (q == null)
           // 将当前线程构造节点
            q = new WaitNode();
       // 如果还没有入队列，则把当前节点加入waiters首节点并替换原来waiters
        else if (!queued)
            queued = UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                 q.next = waiters, q);
       // 如果设置超时时间
        else if (timed) {
            nanos = deadline - System.nanoTime();
           // 时间到，则不再等待结果
            if (nanos <= 0L) {
                removeWaiter(q);
                return state;
            }
           // 阻塞等待特定时间
            LockSupport.parkNanos(this, nanos);
        }
        else
           // 挂起当前线程，知道被其他线程唤醒
            LockSupport.park(this);
    }
}
```

总的来说，进入这个方法，通常会经历三轮循环

1. 第一轮for循环，执行的逻辑是 `q == null`, 这时候会新建一个节点 q, 第一轮循环结束。
2. 第二轮for循环，执行的逻辑是 `!queue`，这个时候会把第一轮循环中生成的节点的 next 指针指向waiters，然后CAS的把节点q 替换waiters, 也就是把新生成的节点添加到waiters 中的首节点。如果替换成功，queued=true。第二轮循环结束。
3. 第三轮for循环，进行阻塞等待。要么阻塞特定时间，要么一直阻塞知道被其他线程唤醒。

对于第二轮循环，大家可能稍稍有点迷糊，我们前面说过，有阻塞，就会排队，有排队自然就有队列，FutureTask 内部同样维护了一个队列

```java
/** Treiber stack of waiting threads */
private volatile WaitNode waiters;
```

说是等待队列，其实就是一个 Treiber 类型 stack，既然是 stack， 那就像手枪的弹夹一样（脑补一下子弹放入弹夹的情形），后进先出，所以刚刚说的第二轮循环，会把新生成的节点添加到 waiters stack 的首节点

如果程序运行正常，通常调用 get() 方法，会将当前线程挂起，那谁来唤醒呢？自然是 run() 方法运行完会唤醒，设置返回结果（set方法）/异常的方法(setException方法) 两个方法中都会调用 finishCompletion 方法，该方法就会唤醒等待队列中的线程

```java
private void finishCompletion() {
    // assert state > COMPLETING;
    for (WaitNode q; (q = waiters) != null;) {
        if (UNSAFE.compareAndSwapObject(this, waitersOffset, q, null)) {
            for (;;) {
                Thread t = q.thread;
                if (t != null) {
                    q.thread = null;
                   // 唤醒等待队列中的线程
                    LockSupport.unpark(t);
                }
                WaitNode next = q.next;
                if (next == null)
                    break;
                q.next = null; // unlink to help gc
                q = next;
            }
            break;
        }
    }

    done();

    callable = null;        // to reduce footprint
}
```

将一个任务的状态设置成终止态只有三种方法：

- set
- setException
- cancel

前两种方法已经分析完，接下来我们就看一下 `cancel` 方法

查看 Future cancel()，该方法注释上明确说明三种 cancel 操作一定失败的情形

1. 任务已经执行完成了
2. 任务已经被取消过了
3. 任务因为某种原因不能被取消

其它情况下，cancel操作将返回true。值得注意的是，cancel操作返回 true 并不代表任务真的就是被取消, **这取决于发动cancel状态时，任务所处的状态**

- 如果发起cancel时任务还没有开始运行，则随后任务就不会被执行；

- 如果发起cancel时任务已经在运行了，则这时就需要看 `mayInterruptIfRunning` 参数了：

- - 如果mayInterruptIfRunning 为true, 则当前在执行的任务会被中断
  - 如果mayInterruptIfRunning 为false, 则可以允许正在执行的任务继续运行，直到它执行完

有了这些铺垫，看一下 cancel 代码的逻辑就秒懂了

```java
public boolean cancel(boolean mayInterruptIfRunning) {
  
    if (!(state == NEW &&
          UNSAFE.compareAndSwapInt(this, stateOffset, NEW,
              mayInterruptIfRunning ? INTERRUPTING : CANCELLED)))
        return false;
    try {    // in case call to interrupt throws exception
       // 需要中断任务执行线程
        if (mayInterruptIfRunning) {
            try {
                Thread t = runner;
               // 中断线程
                if (t != null)
                    t.interrupt();
            } finally { // final state
               // 修改为最终状态 INTERRUPTED
                UNSAFE.putOrderedInt(this, stateOffset, INTERRUPTED);
            }
        }
    } finally {
       // 唤醒等待中的线程
        finishCompletion();
    }
    return true;
}
```

### 参考资料

* [《JAVA并发编程的艺术》](https://weread.qq.com/web/reader/247324e05a66a124750d9e9k8f132430178f14e45fce0f7)
* [Java Future详解](https://mp.weixin.qq.com/s/RX5rVuGr6Ab0SmKigmZEag)
* [Java Future](https://mp.weixin.qq.com/s/RX5rVuGr6Ab0SmKigmZEag)
* [Runnable,Callable,Future关系浅析](https://juejin.im/post/6844903577882722312)