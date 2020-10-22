## 浅析Callable/Future/FutureTask

[【Java学习笔记】多线程](https://blog.csdn.net/KAIZ_LEARN/article/details/108890366)一文中，描述了创建线程主要有两种方式，一种是直接继承Thread类，另外一种就是实现Runnable接口。

这两种方式都有一个缺陷就是：**在执行完任务之后无法获取执行结果**。如果需要获取执行结果，就必须通过共享变量或者使用线程通信的方式来达到效果。

自从Java 1.5 开始，就提供了Callable、Future接口和FutureTask方法，通过它们可以在任务执行完毕之后得到任务执行结果。

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

#### 2.1 概述

Callable与Runnable类似，它也是一个接口，也只有一个方法：`call()`，不同的是Callable的call()方法是有返回值的，返回值的类型是一个泛型，泛型在创建Callable对象时指定。

```java
public interface Callable<V> {
    V call() throws Exception;
}
```

Callable接口则不能直接传入到Thread中来运行，Callable接口通常结合线程池来使用。一般情况下是配合ExecutorService来使用的，在ExecutorService接口中声明了若干个submit方法的重载版本。

<img src="https://img-blog.csdnimg.cn/20201022234728698.png" style="zoom:80%;" />

ExecutorService中对应的submit方法：

```java
	Future<?> submit(Runnable task);
    <T> Future<T> submit(Runnable task, T result);// 一般不使用
	<T> Future<T> submit(Callable<T> task);
```

AbstractExecutorService中对应的实现方法：

```java
    public Future<?> submit(Runnable task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<Void> ftask = newTaskFor(task, null);
        execute(ftask);
        return ftask;
    }

    public <T> Future<T> submit(Runnable task, T result) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<T> ftask = newTaskFor(task, result);
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

可见，使用ExecutorService 的 `execute()` 方法依旧得不到返回值，而 `submit()` 方法都return `Future` 类型的返回值。

#### 2.2 Runnable/Callable异同

| 对比                  | Runnable | Callable |
| --------------------- | -------- | -------- |
| 方法返回值            | 没有     | 有       |
| 受检异常              | 无法处理 | 可以处理 |
| Thread类中使用        | 可以     | 不可以   |
| ExecutorService中使用 | 可以     | 可以     |

### 3. Future

#### 3.1 概述

Future是一个接口，主要提供了三种功能：　　

* 判断任务是否完成

* 能够中断任务

* 能够获取任务执行结果

接口中声明了5个方法：

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

因此，可以把Future理解为一个异步计算的占位对象，用于获取一个将被计算的结果。

#### 3.2 案例分析

### 4. FutureTask 

#### 4.1 概述

FutureTask除了实现Future接口外，还实现了Runnable接口。

<img src="https://img-blog.csdnimg.cn/20201023002648638.png" style="zoom:80%;" />

`FutureTask` 实现了 `RunnableFuture` 接口，而  `RunnableFuture` 接口又分别实现了 `Runnable` 和 `Future` 接口，所以可以推断出 `FutureTask` 具有这两种接口的特性：

- 有 `Runnable` 特性，所以可以用在 `ExecutorService` 中配合线程池使用。
- 有 `Future` 特性，所以可以从中获取到执行结果。

#### 4.2 源码分析



#### 4.3 案例

FutureTask 能够接收 Callable 类型的参数，用来处理有返回结果的情况：

```java
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // 实现多线程的第三种方法可以返回数据
        FutureTask futureTask = new FutureTask<>(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                log.debug("多线程任务");
                Thread.sleep(100);
                return 100;
            }
        });
        // 主线程阻塞，同步等待 task 执行完毕的结果
        new Thread(futureTask,"我的名字").start();
        log.debug("主线程");
        log.debug("{}",futureTask.get());
    }
```

## 参考资料

[Java Future](https://mp.weixin.qq.com/s/RX5rVuGr6Ab0SmKigmZEag)

[JAVA并发编程的艺术](https://weread.qq.com/web/reader/247324e05a66a124750d9e9k8f132430178f14e45fce0f7)

[Runnable,Callable,Future关系浅析](https://juejin.im/post/6844903577882722312)