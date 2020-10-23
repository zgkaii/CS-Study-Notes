## 一、线程基础

### 1.1 线程状态

Java线程在运行的生命周期中可能处于如下6种不同的状态，在给定的一个时刻，线程只能处于其中的一个状态。

| 线程状态      | 说明                                                         |
| :------------ | ------------------------------------------------------------ |
| NEW           | 初始状态，线程刚被创建，但是并未启动（还未调用start方法）。  |
| RUNNABLE      | 运行状态，JAVA线程将操作系统中的就绪（READY）和运行（RUNNING）两种状态笼统地称为“运行中”。 |
| BLOCKED       | 阻塞状态，表示线程阻塞于锁。                                 |
| WAITING       | 等待状态，表示该线程无限期等待另一个线程执行一个特别的动作。 |
| TIMED_WAITING | 超时等待状态，不同于WAITING的是，它可以在指定时间自动返回。  |
| TERMINATED    | 终止状态，表示当前状态已经执行完毕。                         |

线程在自身的生命周期中，并不是固定地处于某个状态，而是随着代码的执行在不同的状态之间进行切换。

<img src="https://img-blog.csdnimg.cn/20200929220833538.png" style="zoom:80%;" />

### 1.2 Thread类常用方法

#### （1）start()与run()

void start()启动一个新线程，在新的线程运行run方法中的代码。

```java
@Slf4j
public class ThreadTest {
    public static void main(String[] args) {
        Thread thread = new Thread() {
            @Override
            public void run() {
                log.debug("running...");
            }
        };
        thread.setName("new thread");
        thread.start();
        log.debug("main thread");
    }
}
```

输出结果如下，可见`run()`方法里面内容的调用是异步的。

```java
14:06:43.754 [main] DEBUG com.kai.demo.basic.ThreadTest - main thread
14:06:43.754 [new thread] DEBUG com.kai.demo.basic.ThreadTest - running...
```

将上面代码的`thread.start()`改为 `thread.run()`，输出结果如下，可见 `run()`方法里面内容的调用是同步的。

```java
14:08:17.974 [main] DEBUG com.kai.demo.basic.ThreadTest - running...
14:08:17.979 [main] DEBUG com.kai.demo.basic.ThreadTest - main thread
```

#### （2）sleep()与yield()

调用 sleep()会让当前线程从 RUNNING进入TIMED_WAITING状态（超时等待）。

```java
public class TimeWaitingTest extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            if ((i) % 5 == 0) {
                System.out.println("开启线程数为:" + i);
            }
            System.out.print(i);
            try {
                Thread.sleep(1000);
                System.out.print(" 线程睡眠1秒！\n");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        new TimeWaitingTest().start();
    }
}
```

运行结果：

```java
开启线程数为:0
0 线程睡眠1秒！
1 线程睡眠1秒！
2 线程睡眠1秒！
3 线程睡眠1秒！
4 线程睡眠1秒！
开启线程数为:5
5 线程睡眠1秒！
6 线程睡眠1秒！
7 线程睡眠1秒！
8 线程睡眠1秒！
9 线程睡眠1秒！
```

可见，线程睡眠到时间就会自动苏醒，并返回到Runnable（可运行）中的就绪状态。

> 建议用 TimeUnit 的 `sleep()` 代替 Thread 的 `sleep()`来获得更好的可读性。



调用yield()静态方法，也能暂停当前线程，那其与`sleep()`有何区别呢？

- sleep(long)方法会**使线程转入超时等待状态**，时间到了之后才会转入就绪状态。而yield()方法不会将线程转入等待，而是强制线程进入就绪状态。
- 使用sleep(long)方法**需要处理异常**`InterruptedException` ，而yield()不用。

> Tips：sleep()中指定的时间是线程暂停运行的最短时间，不能保证该线程睡眠到时后就开始立刻执行。

#### （3）wait()与join()

线程中调用wait()、join()等方法时，当前线程就会进入等待状态，会释放CPU资源和释放同步锁（类锁和对象锁））。直到其他线程调用此对象的`notify()`方法或 `notifyAll()`方法。

* **wait()**

wait是Object类中的方法，方法内部很简单，使用native方法实现：

```java
	// 进入WAITING状态
	public final void wait() throws InterruptedException {
        wait(0);
    }
	// 进入TIMED_WAITING状态
	public final native void wait(long timeout) throws InterruptedException;

    // 进入TIMED_WAITING状态
    public final void wait(long timeout, int nanos) throws InterruptedException {
        if (timeout < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }
        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }
        if (nanos > 0) {
            timeout++;
        }
        wait(timeout);
    }    
```

wait()方法只能在**同步方法**中调用。如果当前线程不是锁的持有者，该方法抛出一个`IllegalMonitorStateException`异常。

如果当前线程在等待之前或在等待时被任何线程中断，则会抛出 `InterruptedException `异常。

案例：

```java
public class WaitingTest {
    public static Object obj = new Object();

    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    synchronized (obj) {
                        try {
                            System.out.println(Thread.currentThread().getName() + "=== 调用wait方法，进入WAITING状态，释放锁对象");
                            obj.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        System.out.println(Thread.currentThread().getName() + "=== 从WAITING状态醒来，获取到锁对象，继续执行");
                    }
                }
            }
        }, "线程A").start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) { //每隔3秒 唤醒一次
                    try {
                        System.out.println(Thread.currentThread().getName() + "‐‐‐ 等待3秒钟");
                        Thread.sleep(3000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized (obj) {
                        System.out.println(Thread.currentThread().getName() + "‐‐‐ 获取到锁对象, 调用notify方法，释放锁对象");
                        obj.notify();
                    }
                }
            }
        }, "线程B").start();
    }
}
```

执行结果：

```java
线程A=== 调用wait方法，进入WAITING状态，释放锁对象
线程B‐‐‐ 等待3秒钟
线程B‐‐‐ 获取到锁对象, 调用notify方法，释放锁对象
线程B‐‐‐ 等待3秒钟
线程A=== 从WAITING状态醒来，获取到锁对象，继续执行
... ... 
```

通过上述案例我们会发现，线程A在调用了某个对象的 `Object.wait `方法后，需要等待线程B调用此对象的
`Object.notify()`方法将其唤醒。

* **join()**

join()是Thread类中的方法，若没有指定时间，当前线程中调用另一个线程的join时，当前线程会挂起，等到另一个线程执行完毕之后才能继续向下执行；若指定了时间，到等待指定时间，即便调用join的线程没运行完run方法，当前线程也会继续往下运行。查看源码可知，join方法是通过调用wait方法实现的。

```java
    public final void join() throws InterruptedException {
        join(0);
    }
	public final synchronized void join(long millis) throws InterruptedException {
        long base = System.currentTimeMillis();
        long now = 0;

        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (millis == 0) {
            while (isAlive()) {// 判断子线程是否存活
                wait(0);// 调用wait(0)方法，进入WAITING状态
            }
        } else {
            while (isAlive()) {
                long delay = millis - now;
                if (delay <= 0) {
                    break;
                }
                wait(delay);// 调用wait(long timeout),进入TIMED_WAITING状态
                now = System.currentTimeMillis() - base;
            }
        }
    }
    public final synchronized void join(long millis, int nanos) throws InterruptedException {
		---Omit---
    }
```

查看`isAlive()`方法源码：

```java
/**
     * Tests if this thread is alive. A thread is alive if it has
     * been started and has not yet died.
     * 测试线程是否存活（线程已经开始，还没有死亡的状态即为存活）。
     * @return  <code>true</code> if this thread is alive;
     *          <code>false</code> otherwise.
     */****
    public final native boolean isAlive();
```

可见，当A线程调用B线程的join方法时，其实就是在A线程中调B的wait方法，然后使A线程处于等待状态。但是B线程并没有中调notify()，B线程执行完，A线程如何被唤醒呢？

通过debug，发现在线程执行完系统会调用Thread类中的`exit()`方法，查看源码：

```java
 /**
     * This method is called by the system to give a Thread
     * a chance to clean up before it actually exits.
     * 这个方法由系统调用，当该线程完全退出前给它一个机会去释放空间。
     */
private void exit() {
        if (group != null) {                //线程组在Thread初始化时创建，存有创建的子线程
            group.threadTerminated(this);   //调用threadTerminated()方法
            group = null;
        }
        /* Aggressively null out all reference fields: see bug 4006245 */
        target = null;
        /* Speed the release of some of these resources */
        threadLocals = null;
        inheritableThreadLocals = null;
        inheritedAccessControlContext = null;
        blocker = null;
        uncaughtExceptionHandler = null;
    }
```

此时线程组中存在当前子线程，因此会调用线程组的threadTerminated()方法。 查看ThreadGroup.threadTerminated()方法源码：

```java
/** Notifies the group that the thread {@code t} has terminated.
  * 通知线程组，t线程已经终止。
  *
void threadTerminated(Thread t) {
        synchronized (this) {
            remove(t);                          //从线程组中删除此线程

            if (nthreads == 0) {                //当线程组中线程数为0时
                notifyAll();                    //唤醒所有待定中的线程
            }
            if (daemon && (nthreads == 0) &&
                (nUnstartedThreads == 0) && (ngroups == 0))
            {
                destroy();
            }
        }
    }
```

通过此方法，将子线程从线程组中删除，并**唤醒其他等待的线程**。

案例：

```java
public class JoinTest {
    public static void main(String[] args) {
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("[" + Thread.currentThread().getName() + "] begins");
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("[" + Thread.currentThread().getName() + "] ends");
            }
        }, "child thread");
        thread.start();

        try {
            thread.join(1000);
            if (thread.isAlive()) {
                System.out.println("[" + Thread.currentThread().getName() + "] child thread has not finished");
            } else {
                System.out.println("[" + Thread.currentThread().getName() + "] child thread has finished");
            }
            System.out.println("Join Success");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

执行结果：

```java
[child thread] begins
[main] child thread has not finished
Join Success
[child thread] ends
```

#### （4）interrupt()

打断 sleep，wait，join 的线程

先了解一些interrupt()方法的相关知识：[博客地址](https://www.cnblogs.com/noteless/p/10372826.html#0)

 sleep，wait，join 的线程，这几个方法都会让线程进入阻塞状态，以 sleep 为例Test7.java

```java
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread() {
            @Override
            public void run() {
                log.debug("线程任务执行");
                try {
                    Thread.sleep(10000); // wait, join
                } catch (InterruptedException e) {
                    //e.printStackTrace();
                    log.debug("被打断");
                }
            }
        };
        t1.start();
        Thread.sleep(500);
        log.debug("111是否被打断？{}",t1.isInterrupted());
        t1.interrupt();
        log.debug("222是否被打断？{}",t1.isInterrupted());
        Thread.sleep(500);
        log.debug("222是否被打断？{}",t1.isInterrupted());
        log.debug("主线程");
    }
```

输出结果：（我下面将中断和打断两个词混用）可以看到，打断 sleep 的线程, 会清空中断状态，刚被中断完之后`t1.isInterrupted()`的值为`true`，后来变为`false`，即中断状态会被清除。那么线程是否被中断过可以通过异常来判断。【同时要注意如果打断被`join()`，`wait()` blocked的线程也是一样会被清除，被清除(interrupt status will be cleared)的意思即中断状态设置为`false`，被设置( interrupt status will be set)的意思就是中断状态设置为`true`】

```properties
17:06:11.890 [Thread-0] DEBUG com.concurrent.test.Test7 - 线程任务执行
17:06:12.387 [main] DEBUG com.concurrent.test.Test7 - 111是否被打断？false
17:06:12.390 [Thread-0] DEBUG com.concurrent.test.Test7 - 被打断
17:06:12.390 [main] DEBUG com.concurrent.test.Test7 - 222是否被打断？true
17:06:12.890 [main] DEBUG com.concurrent.test.Test7 - 222是否被打断？false
17:06:12.890 [main] DEBUG com.concurrent.test.Test7 - 主线程
```

打断正常运行的线程

打断正常运行的线程, 线程并不会暂停，只是调用方法`Thread.currentThread().isInterrupted();`的返回值为true，可以判断`Thread.currentThread().isInterrupted();`的值来手动停止线程

```java
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            while(true) {
                boolean interrupted = Thread.currentThread().isInterrupted();
                if(interrupted) {
                    log.debug("被打断了, 退出循环");
                    break;
                }
            }
        }, "t1");
        t1.start();
        Thread.sleep(1000);
        log.debug("interrupt");
        t1.interrupt();
    }
```

终止模式之两阶段终止模式

Two Phase Termination，就是考虑在一个线程T1中如何优雅地终止另一个线程T2？这里的优雅指的是给T2一个料理后事的机会（如释放锁）。

如下所示：那么线程的`isInterrupted()`方法可以取得线程的打断标记，如果线程在睡眠`sleep`期间被打断，打断标记是不会变的，为false，但是`sleep`期间被打断会抛出异常，我们据此手动设置打断标记为`true`；如果是在程序正常运行期间被打断的，那么打断标记就被自动设置为`true`。处理好这两种情况那我们就可以放心地来料理后事啦！

![1583496991915](https://gitee.com/gu_chun_bo/picture/raw/master/image/20200306201632-731186.png)

代码实现如下：

```java
@Slf4j
public class Test11 {
    public static void main(String[] args) throws InterruptedException {
        TwoParseTermination twoParseTermination = new TwoParseTermination();
        twoParseTermination.start();
        Thread.sleep(3000);  // 让监控线程执行一会儿
        twoParseTermination.stop(); // 停止监控线程
    }
}


@Slf4j
class TwoParseTermination{
    Thread thread ;
    public void start(){
        thread = new Thread(()->{
            while(true){
                if (Thread.currentThread().isInterrupted()){
                    log.debug("线程结束。。正在料理后事中");
                    break;
                }
                try {
                    Thread.sleep(500);
                    log.debug("正在执行监控的功能");
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    e.printStackTrace();
                }
            }
        });
        thread.start();
    }
    public void stop(){
        thread.interrupt();
    }
}
```

3.3.6 sleep，yiled，wait，join 对比

关于join的原理和这几个方法的对比：[看这里](https://blog.csdn.net/dataiyangu/article/details/104956755)

> 补充：
>
> 1. sleep，join，yield，interrupted是Thread类中的方法
> 2. wait/notify是object中的方法
>
> sleep 不释放锁、释放cpu
> join 释放锁、抢占cpu
> yiled 不释放锁、释放cpu
> wait 释放锁、释放cpu

### 1.3 线程优先级

现代操作系统基本采用时分的形式调度运行的线程，操作系统会分出一个个时间片，线程会分配到若干时间片，当线程的时间片用完了就会发生线程调度，并等待着下次分配。线程分配到的时间片多少也就决定了线程使用处理器资源的多少，而线程优先级就是决定线程需要多或者少分配一些处理器资源的线程属性。

在Java线程中，通过一个整型成员变量priority来控制优先级，优先级的范围从1～10，默认优先级是5，优先级高的线程分配时间片的数量要多于优先级低的线程。

查看源代码：

```java
	private int priority;
	
	// 设置优先级
	public final void setPriority(int newPriority) {
        ThreadGroup g;
        checkAccess();
        if (newPriority > MAX_PRIORITY || newPriority < MIN_PRIORITY) {
            throw new IllegalArgumentException();
        }
        if((g = getThreadGroup()) != null) {
            if (newPriority > g.getMaxPriority()) {
                newPriority = g.getMaxPriority();
            }
            setPriority0(priority = newPriority);
        }
    }
	// 获取优先级
    public final int getPriority() {
        return priority;
    }

    public String toString() {
        ThreadGroup group = getThreadGroup();
        if (group != null) {
            return "Thread[" + getName() + "," + getPriority() + "," +
                           group.getName() + "]";
        } else {
            return "Thread[" + getName() + "," + getPriority() + "," +
                            "" + "]";
        }
    }

	// 优先级常量
     public final static int MIN_PRIORITY = 1;
     public final static int NORM_PRIORITY = 5;
     public final static int MAX_PRIORITY = 10;
```

查看Thread 初始化 priority 的源代码：

```java
        Thread parent = currentThread();  
        // 获取父线程的线程优先级
        this.priority = parent.getPriority();
```

可见，**子线程默认优先级和父线程保持一致，Java主线程默认的优先级是 5。**

```java
public class PriorityTest {
    public static void main(String[] args) {
        Thread.currentThread().setPriority(3);// 主线程优先级修改为3
        Thread thread = new Thread();
        System.out.println(thread.getPriority());// 子线程优先级跟主线程优先级保持一致，并不是默认的5，而是3。
    }
}
```

下面通过一个例子来测试一下高优先级与低优先级差别：

```java
public class PriorityTest {
    private static volatile boolean notStart = true;
    private static volatile boolean notEnd = true;

    public static void main(String[] args) throws Exception {
        List<Task> tasks = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            int priority = i < 5 ? Thread.MIN_PRIORITY : Thread.MAX_PRIORITY;
            Task task = new Task(priority);
            tasks.add(task);
            Thread thread = new Thread(task, "Thread:" + i);
            thread.setPriority(priority);
            thread.start();
        }
        notStart = false;
        TimeUnit.SECONDS.sleep(10);
        notEnd = false;
        for (Task task : tasks) {
            System.out.println("Task Priority:" + task.priority + " Count:" + task.taskCount);
        }
    }

    static class Task implements Runnable {
        private int priority;
        private long taskCount;

        public Task(int priority) {
            this.priority = priority;
        }

        @Override
        public void run() {
            while (notStart) {
                Thread.yield();
            }
            while (notEnd) {
                Thread.yield();
                taskCount++;
            }
        }
    }
}
```

执行结果：

```java
Task Priority:1 Count:12774
Task Priority:1 Count:12775
Task Priority:1 Count:12762
Task Priority:1 Count:12747
Task Priority:1 Count:12758
Task Priority:10 Count:1984746
Task Priority:10 Count:1984810
Task Priority:10 Count:1982704
Task Priority:10 Count:1985309
Task Priority:10 Count:1980892
```

可见，**高优先级的线程大概率比低优先的线程优先获得 CPU 资源**。

### 1.4 守护线程

默认情况下，java进程需要等待所有的线程结束后才会停止，但是有一种特殊的线程，叫做守护线程，在其他线程全部结束的时候即使守护线程还未结束代码未执行完java进程也会停止。普通线程t1可以调用`t1.setDeamon(true);` 方法变成守护线程 

> 注意
> 垃圾回收器线程就是一种守护线程
> Tomcat 中的 Acceptor 和 Poller 线程都是守护线程，所以 Tomcat 接收到 shutdown 命令后，不会等
> 待它们处理完当前请求



## 参考资料

[多线程基础](https://blog.csdn.net/KAIZ_LEARN/article/details/108890366)

[Java线程优先级](https://juejin.im/post/6844903951339372557)

[JAVA并发编程的艺术](https://weread.qq.com/web/reader/247324e05a66a124750d9e9k8f132430178f14e45fce0f7)

[Thread-线程的常见方法](https://www.cnblogs.com/xiaokw/p/12678774.html)

[Java Thread的join() 之刨根问底](https://juejin.im/post/6844903624842149895)