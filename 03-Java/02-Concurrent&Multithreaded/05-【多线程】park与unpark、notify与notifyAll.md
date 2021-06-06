- [1 线程状态简述](#1-线程状态简述)
- [2 wait和notify/notifyAll](#2-wait和notifynotifyall)
  - [2.1 源码简析](#21-源码简析)
  - [2.2 等待/通知机制](#22-等待通知机制)
  - [2.3  生产者/消费者模式](#23--生产者消费者模式)
- [3 park与unpark](#3-park与unpark)
- [参考资料](#参考资料)
## 1 线程状态简述

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

在[【并发编程基础】线程基础（常用方法、状态）](https://blog.csdn.net/KAIZ_LEARN/article/details/109302495)一文中，主要学习了wait()、join()和sleep()等方法，在[【并发编程】深入理解synchronized原理](https://blog.csdn.net/KAIZ_LEARN/article/details/109372692)一文中，主要探讨了synchronized原理。下面就进行park()/unpark()、wait()/notify()/notifyAll()的学习。

## 2 wait和notify/notifyAll

### 2.1 源码简析

wait( )，notify( )，notifyAll( )都是Object基础类中的方法，所以在任何 Java 对象上都可以使用。

```java
public class Object {
    // 导致当前线程等待，直到另一个线程调用此对象的notify()方法或notifyAll()方法。
    public final void wait() throws InterruptedException {
        wait(0);
    }

    // 导致当前线程等待，直到另一个线程调用此对象的notify()方法或notifyAll()方法，或者已经过了指定的时间。
    public final native void wait(long timeout) throws InterruptedException;

    //  唤醒正在此对象监视器上等待的单个线程。
    public final native void notify();

    //  唤醒等待此对象监视器的所有线程。
    public final native void notifyAll();
}
```

[打开objectMonitor.cpp](http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/9deea71d83dd/src/share/vm/runtime/objectMonitor.cpp)，查看wait方法：

```c++
 	...
   // create a node to be put into the queue
   // Critically, after we reset() the event but prior to park(), we must check
   // for a pending interrupt.
   ObjectWaiter node(Self);		  		 // 将当前线程封装成ObjectWatier
   node.TState = ObjectWaiter::TS_WAIT ; // 状态改为等待状态
   Self->_ParkEvent->reset() ;
   OrderAccess::fence();          // ST into Event; membar ; LD interrupted-flag

   // Enter the waiting queue, which is a circular doubly linked list in this case
   // but it could be a priority queue or any data structure.
   // _WaitSetLock protects the wait queue.  Normally the wait queue is accessed only
   // by the the owner of the monitor *except* in the case where park()
   // returns because of a timeout of interrupt.  Contention is exceptionally rare
   // so we use a simple spin-lock instead of a heavier-weight blocking lock.

   Thread::SpinAcquire (&_WaitSetLock, "WaitSet - add") ;// 自旋操作
   AddWaiter (&node) ;
   Thread::SpinRelease (&_WaitSetLock) ;				 // 添加到_WaitSet节点中
	...
```

查看AddWaiter()方法：

```c++
inline void ObjectMonitor::AddWaiter(ObjectWaiter* node) {
  assert(node != NULL, "should not dequeue NULL node");
  assert(node->_prev == NULL, "node already in list");
  assert(node->_next == NULL, "node already in list");
  // put node at end of queue (circular doubly linked list)
  if (_WaitSet == NULL) {
    _WaitSet = node;
    node->_prev = node;
    node->_next = node;
  } else {
    ObjectWaiter* head = _WaitSet ; // 通过双向链表的方式，将ObjectWaiter对象添加到_WaitSet列表中
    ObjectWaiter* tail = head->_prev;
    assert(tail->_next == head, "invariant check");
    tail->_next = node;
    head->_prev = node;
    node->_next = head;
    node->_prev = tail;
  }
}
```

查看notify方法源码：

```c++
void ObjectMonitor::notify(TRAPS) {
  CHECK_OWNER();
  if (_WaitSet == NULL) {
     TEVENT (Empty-Notify) ;// _WaitSet=NULL表明没有等待状态的线程，直接返回。
     return ;
  }
  DTRACE_MONITOR_PROBE(notify, this, object(), THREAD);

  int Policy = Knob_MoveNotifyee ;

  Thread::SpinAcquire (&_WaitSetLock, "WaitSet - notify") ;
  ObjectWaiter * iterator = DequeueWaiter() ;// 获取一个ObjectWaiter对象
  if (iterator != NULL) {
      ...
     ObjectWaiter * List = _EntryList ;
     if (List != NULL) {
        assert (List->_prev == NULL, "invariant") ;
        assert (List->TState == ObjectWaiter::TS_ENTER, "invariant") ;
        assert (List != iterator, "invariant") ;
     }
	 // 根据不同状态采取不同策略，将从_WaitSet列表中移出来的ObjectWaiter对象加入到_EntryList列表中。
     if (Policy == 0) {       // prepend to EntryList
         if (List == NULL) {
             iterator->_next = iterator->_prev = NULL ;
             _EntryList = iterator ;
         } else {
             List->_prev = iterator ;
             iterator->_next = List ;
             iterator->_prev = NULL ;
             _EntryList = iterator ;
        }
     } else
     if (Policy == 1) {...} else     // append to EntryList
	 if (Policy == 2) {...} else     // prepend to cxq
	 if (Policy == 3) {				 // append to cxq
         ...     
     } else {
        ParkEvent * ev = iterator->_event ;
        iterator->TState = ObjectWaiter::TS_RUN ;
        OrderAccess::fence() ;		 // 被唤醒的线程又变成run状态。
        ev->unpark() ;
     }
}
```

查看notifyAll方法源码：

```c++
void ObjectMonitor::notifyAll(TRAPS) {
  CHECK_OWNER();
  ObjectWaiter* iterator;
  if (_WaitSet == NULL) {
      TEVENT (Empty-NotifyAll) ;// _WaitSet=NULL表明没有等待状态的线程，直接返回。
      return ;
  }
  DTRACE_MONITOR_PROBE(notifyAll, this, object(), THREAD);

  int Policy = Knob_MoveNotifyee ;
  int Tally = 0 ;
  Thread::SpinAcquire (&_WaitSetLock, "WaitSet - notifyall") ;

  for (;;) {
     iterator = DequeueWaiter () ;// 循环获取所以ObjectWaiter对象
	   ...
     ObjectWaiter * List = _EntryList ;
     if (List != NULL) {
        assert (List->_prev == NULL, "invariant") ;
        assert (List->TState == ObjectWaiter::TS_ENTER, "invariant") ;
        assert (List != iterator, "invariant") ;
     }
	  // 根据不同状态采取不同策略，将从_WaitSet列表中移出来的ObjectWaiter对象加入到_EntryList列表中。	
     if (Policy == 0) {       // prepend to EntryList
         if (List == NULL) {
             iterator->_next = iterator->_prev = NULL ;
             _EntryList = iterator ;
         } else {
             List->_prev = iterator ;
             iterator->_next = List ;
             iterator->_prev = NULL ;
             _EntryList = iterator ;
        }
     } else
     if (Policy == 1) {      // append to EntryList
		...
     } else
     if (Policy == 2) {      // prepend to cxq
		...
     } else
     if (Policy == 3) {      // append to cxq
		...
     } else {
        ParkEvent * ev = iterator->_event ;
        iterator->TState = ObjectWaiter::TS_RUN ;// 被唤醒的线程又变成run状态。
        OrderAccess::fence() ;
        ev->unpark() ;
     }

      ...
```

可见，wait()与notify()/notifyAll()的实现都跟Monitor有很大关联。

![](https://img-blog.csdnimg.cn/20201026000053726.png)

* 当多线程访问一段同步代码块时，这些都线程会被被封装成一个个ObjectWatier对象，并被放入 _EntryList列表中，也就是被放到 Entry Set（入口区） 中等待获取锁。
* 如果该线程获取到了锁（acquire），线程就会成为当前锁的 Owner。
* 获取到锁的线程可也以通过调用 wait 方法将锁释放（release），然后该线程对象会被放入_WaitSet列表中，进入Wait Set （等待区）进行等待（阻塞BLOCKED）。
* 当获取到锁的对象调用notify/notifyAll方法唤醒等待区被阻塞的线程时，线程重新竞争锁。如果竞争锁成功，那么线程就进入RUNNABLE状态；如果竞争锁失败，这些线程会重新进入到Entry Set区再重新去竞争锁。

wait方法的使用对应上图的第3步，也就是说，**调用`wait()`、`notify()`/`notifyAll()`方法的对象必须已经获取到锁**。

如何确保调用对象获取到锁呢？使用sychronized关键字呗！**所以说这些方法调用也必须发生在sychronized修饰的同步代码块内**。

### 2.2 等待/通知机制

**（1）什么是等待/通知机制**

等待/通知机制是多个线程间的一种**协作**机制。谈到线程我们经常想到的是线程间的**竞争（race）**，比如去竞争锁。但这并不是故事的全部，线程间也有协作机制。就好比我们在公司中与同事关系，可能存在在晋升时的竞争，但更多时候是一起合作以完成某些任务。

**wait/notify** 就是线程间的一种协作机制。

当一个线程调用wait()/wait(long)方法后，进入WAITING状态或者TIMED_WAITING状态（阻塞），并释放锁与CPU资源。只有其他获取到锁的线程执行完他们的指定代码过后，再通过notify()方法将其唤醒。 如果需要，也可以使用` notifyAll()`来唤醒所有的阻塞线程。

**（2）等待/通知使用方法**

等待/通知机制就是用于解决线程间通信的问题的，使用到的3个方法的含义如下：

1. **wait**：线程不再活动，不再参与调度，**释放**它对锁的拥有权。它还要等着别的线程执行一个**特别的动作**，也即是“**通知（notify）**”在这个对象上等待的线程从WAITING状态中释放出来，重新进入到调度队列（ready queue）中。
2. **notify**：唤醒一个等待当前对象的锁的线程。唤醒在此对象监视器上等待的单个线程。
3. **notifyAll**：唤醒在此对象监视器上等待的所有线程。

>注意：
>
>哪怕只通知了一个等待的线程，被通知线程也不能立即恢复执行，因为它当初中断的地方是在同步块内，而此刻它已经不持有锁，所以它需要**再次尝试去获取锁**（很可能面临其它线程的竞争），成功后才能在当初调用 wait 方法之后的地方恢复执行。
>
>总结如下：
>
>- 如果能获取锁，线程就从 WAITING/TIMED_WAITING 状态转换为RUNNABLE 状态；
>- 否则，从 Wait Set 区出来，又进入 Entry Set区，线程就从 WAITING 状态又变成 BLOCKED 状态。

**（3）调用wait和notify方法需要注意的细节**

* wait方法与notify方法**必须要由同一个锁对象调用**。因为对应的锁对象可以通过notify唤醒使用同一个锁对象调用的wait方法后的线程。
* wait方法与notify方法是属于Object类的方法的。因为锁对象可以是任意对象，而任意对象的所属类都是继承了Object类的。
* wait方法与notify方法**必须要在同步代码块或者是同步函数中使用**。因为必须要通过锁对象调用这2个方法。

下面就通过一个案例来进一步了解等待/通知机制：

```java
public class WaitAndNotify {

    static boolean flag = true;
    static Object lock = new Object();

    public static void main(String[] args) throws Exception {
        Thread waitThread = new Thread(new Wait(), "WaitThread");
        waitThread.start();
        TimeUnit.SECONDS.sleep(1);
        Thread notifyThread = new Thread(new Notify(), "NotifyThread");
        notifyThread.start();
    }

    static class Wait implements Runnable {
        @Override
        public void run() {
            // 加锁，拥有lock的Monitor
            synchronized (lock) {
                // 当条件不满足时，继续wait，同时释放了lock的锁
                while (flag) {
                    try {
                        System.out.println(Thread.currentThread().getName() + " flag is true. wait @ " +
                                new SimpleDateFormat("HH:mm:ss").format(new Date()));
                        lock.wait();
                    } catch (InterruptedException e) {

                    }
                }
                // 当条件满足时，完成工作
                System.out.println(Thread.currentThread().getName() + " flag is false. running @ " +
                        new SimpleDateFormat("HH:mm:ss").format(new Date()));
            }
        }
    }

    static class Notify implements Runnable {
        @Override
        public void run() {
            // 加锁，拥有lock的Monitor
            synchronized (lock) {
                // 获得lock的锁，然后进行通知
                System.out.println(Thread.currentThread().getName() + " hold lock. notify @ " +
                        new SimpleDateFormat("HH:mm:ss").format(new Date()));
                lock.notifyAll();
                flag = false;
                SleepUtils.second(5);
            }
            // 再次加锁
            synchronized (lock) {
                System.out.println(Thread.currentThread().getName() + " hold lock. again. sleep @ " +
                        new SimpleDateFormat("HH:mm:ss").format(new Date()));
                SleepUtils.second(5);
            }
        }
    }
}

class SleepUtils {
    public static final void second(long seconds) {
        try {
            TimeUnit.SECONDS.sleep(seconds);
        } catch (InterruptedException e) {

        }
    }
}
```

输出结果可能如下：

```java
WaitThread flag is true. wait @ 00:38:53
NotifyThread hold lock. notify @ 00:38:54
NotifyThread hold lock. again. sleep @ 00:38:59
WaitThread flag is false. running @ 00:39:04
```

也可能如下：

```java
WaitThread flag is true. wait @ 00:38:53
NotifyThread hold lock. notify @ 00:38:54
WaitThread flag is false. running @ 00:39:04
NotifyThread hold lock. again. sleep @ 00:38:59
```

之所以出现这类情况，在于调用notify()或notifyAll()方法调用后，waitThread是否成功获取到锁。竞争成功，则进入RUNNABLE状态；如果竞争失败，waitThread会重新进入到Entry Set区再重新去竞争锁。也就是说，**从wait()方法返回的前提是获得了调用对象的锁**。

从上述细节中可以看到，等待/通知机制依托于同步机制，其目的就是确保等待线程从wait()方法返回时能够感知到通知线程对变量做出的修改。

下图描述了上述示例的过程：

![](https://img-blog.csdnimg.cn/20201031005323535.png)

### 2.3  生产者/消费者模式

从上面案例中，可以提炼出等待/通知的经典范式——生产者/消费者模式。该范式主要分为两部分，分别针对生产者（通知方）和消费者（等待方）。

**消费者**遵循如下原则：

（1）获取对象的锁。

（2）如果条件不满足，那么调用对象的wait()方法，被通知后仍要检查条件。

（3）条件满足则执行对应的逻辑。

对应的伪代码如下。

```java
 synchronized (对象) {
     while (条件) {
         对象.wait();
     }
     对应的处理逻辑
 }
```

**生产者**遵循如下原则：

（1）获得对象的锁。

（2）改变条件。

（3）通知所有等待在对象上的线程。对应的伪代码如下。

对应的伪代码如下。

```java
 synchronized (对象) {
	改变的条件
    对象.notifyAll();//或者 对象.notify()
 }
```

**代码实例**：

首先建了一个简单的 `Product` 类，用来表示生产和消费的产品。

```java
public class Product {
    private String name;

    public Product(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

创建生产者类：

```java
public class Producer implements Runnable {
    private Queue<Product> queue;
    private int maxCapacity;

    public Producer(Queue<Product> queue, int maxCapacity) {
        this.queue = queue;
        this.maxCapacity = maxCapacity;
    }

    @Override
    public void run() {
        synchronized (queue) {
            while (queue.size() == maxCapacity) {
                try {
                    System.out.println("生产者" 
                                       + Thread.currentThread().getName() + "Queue 已满，WAITING");
                    wait();
                    System.out.println("生产者" + Thread.currentThread().getName() + "退出等待");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            if (queue.size() == 0) { //队列里的产品从无到有，需要通知在等待的消费者
                queue.notifyAll();
            }
            Integer i = new Random().nextInt(50);
            queue.offer(new Product("产品" + i.toString()));
            System.out.println("生产者" + Thread.currentThread().getName() + "生产了产品" + i.toString());
        }
    }
}
```

创建消费者类：

```java
public class Consumer implements Runnable {

    private Queue<Product> queue;
    private int maxCapacity;

    public Consumer(Queue queue, int maxCapacity) {
        this.queue = queue;
        this.maxCapacity = maxCapacity;
    }

    @Override
    public void run() {
        synchronized (queue) {
            while (queue.isEmpty()) {
                try {
                    System.out.println("消费者" 
                                       + Thread.currentThread().getName() + "Queue已空，WAITING");
                    wait();
                    System.out.println("消费者" + Thread.currentThread().getName() + "退出等待");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            if (queue.size() == maxCapacity) {
                queue.notifyAll();
            }

            Product product = queue.poll();
            System.out.println("消费者" + Thread.currentThread().getName() + "消费了" + product.getName());
        }
    }
}
```

开启多线程：

```java
public class TreadTest {
    public static void main(String[] args) {
        Queue<Product> queue = new ArrayDeque<>();
        for (int i = 0; i < 10; i++) {
            new Thread(new Producer((Queue<Product>) queue, 10)).start();
            new Thread(new Consumer((Queue) queue, 10)).start();
        }
    }
}
```

测试结果：

```shell
生产者Thread-0生产了产品35
消费者Thread-1消费了产品35
生产者Thread-2生产了产品43
消费者Thread-3消费了产品43
消费者Thread-5Queue已空，WAITING
生产者Thread-6生产了产品17
生产者Thread-8生产了产品39
消费者Thread-7消费了产品17
生产者Thread-10生产了产品17
生产者Thread-12生产了产品3
消费者Thread-13消费了产品39
生产者Thread-14生产了产品10
消费者Thread-17消费了产品17
生产者Thread-16生产了产品8
消费者Thread-19消费了产品3
生产者Thread-4生产了产品29
消费者Thread-9消费了产品10
消费者Thread-11消费了产品8
消费者Thread-15消费了产品29
生产者Thread-18生产了产品33
```

## 3 park与unpark

LockSupport类是Java6(JSR166-JUC)引入的一个类，用来**创建锁和其他同步工具类的基本线程阻塞原语**。使用LockSupport类中的park()和unpark()方法也可以实现线程的阻塞与唤醒。Park有停车的意思，假设线程为车辆，那么park方法代表着停车，而unpark方法则是指车辆启动离开。

```java
    public static void park() {
        UNSAFE.park(false, 0L);
    }

    public static void park(Object blocker) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        UNSAFE.park(false, 0L);
        setBlocker(t, null);
    }

    public static void unpark(Thread thread) {
        if (thread != null)
            UNSAFE.unpark(thread);
    }
```

归根到底还是调用了UNSAFE类中的函数：

```java
    public native void unpark(Object var1);

    public native void park(boolean var1, long var2);
```

与 wait/notify 相比，park/unpark 方法更贴近操作系统层面的阻塞与唤醒线程，**并不需要获取对象的监视器**。

park/unpark 原理可参考[LockSupport中的park与unpark原理](https://blog.csdn.net/weixin_39687783/article/details/85058686)一文。

需要明白的是，每个java线程都有一个Parker对象，主要三部分组成 _counter、 _cond和 _mutex。Parker类是这样定义的：

```C++
class Parker : public os::PlatformParker {
private:
  //表示许可
  volatile int _counter ;
  ...
public:
  Parker() : PlatformParker() {
    //初始化_counter
    _counter       = 0 ; 
...
public:
  void park(bool isAbsolute, jlong time);
  void unpark();
  ...
}
class PlatformParker : public CHeapObj<mtInternal> {
  protected:
    pthread_mutex_t _mutex [1] ;
    pthread_cond_t  _cond  [1] ;
    ...
}
```

Parker类里的_counter字段，就是用来记录所谓的“**许可**”的。**当调用park时，这个变量置为了0；当调用unpark时，这个变量置为1**。

park和unpark的灵活之处在于，**unpark函数可以先于park调用**。比如线程B调用unpark函数，给线程A发了一个“许可”，那么当线程A调用park时，它发现已经有“许可”了，那么它会马上再继续运行。

**先调用park再调用upark时**：

1.先调用park

（1）当前线程调用 Unsafe.park() 方法，检查_counter情况（为0），获得 _mutex 互斥锁。

（2）mutex对象有个等待队列 _cond，线程进入等待队列中阻塞。

（4）设置 _counter = 0。

2.再调用upark

（1）调用 Unsafe.unpark方法，设置 _counter 为 1

（2）唤醒 _cond 条件变量中的 阻塞线程，线程恢复运行。

（3）设置 _counter 为 0

**先调用upark再调用park时**：

（1）调用 Unsafe.unpark方法，设置 _counter 为 1。

（2）当前线程调用 Unsafe.park() 方法。

（3）检查 _counter ，本情况为 1，这时线程无需阻塞，继续运行。

（4）设置 _counter 为 0。

特别注意的是，**LockSupport是不可重入**的，如果一个线程连续2次调用LockSupport.park()，那么该线程一定会一直阻塞下去。

```java
    public static void main(String[] args) throws Exception {
        Thread thread = Thread.currentThread();

        LockSupport.unpark(thread);
        System.out.println("线程处于运行状态");
        LockSupport.park();
        System.out.println("线程处于阻塞状态");
        LockSupport.park();
        System.out.println("线程处于阻塞状态");
        LockSupport.unpark(thread);
        System.out.println("线程处于运行状态？？？");
    }
```

运行结果如下：

```java
线程处于运行状态
线程处于阻塞状态
```

可见，在第二次调用park后，线程无法再获取许可出现了死锁。

## 参考资料

* [《JAVA并发编程的艺术》](https://weread.qq.com/web/reader/247324e05a66a124750d9e9k8f132430178f14e45fce0f7)
* [Java精通并发-透过openjdk源码分析wait与notify方法的本地实现](https://www.cnblogs.com/webor2006/p/11443392.html)
* [LockSupport中的park与unpark原理](https://blog.csdn.net/weixin_39687783/article/details/85058686)
* [Java的LockSupport.park()实现分析](https://hengyunabc.blog.csdn.net/article/details/28126139?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)