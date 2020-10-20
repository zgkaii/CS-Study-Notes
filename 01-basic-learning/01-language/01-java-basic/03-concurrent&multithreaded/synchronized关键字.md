### synchronized原理

#### synchronized修饰代码块

​	当我们使用synchronized关键字来修饰代码块是，字节码层面上是通过monitorenter与monitorxeit指令来实现的锁的获取与释放动作，当线程进入到monitorenter指令后，线程将会持有monitor对象，退出monitorenter指令后，线程将会释放monitor对象。MyTest4.java

对如下代码进行反编译：

```java
public class MyTest4 {
    private Object object = new Object();
    public void method(){
        synchronized (object){
            System.out.println("--test--");
        }
    }
}
```

查看method()的字节码指令，可以证明我们说的结论

```properties
 		0: aload_0
         1: getfield      #3                  // Field object:Ljava/lang/Object;
         4: dup
         5: astore_1
         6: monitorenter
         7: getstatic     #4                  // Field java/lang/System.out:Ljava/io/PrintStream;
        10: ldc           #5                  // String --test--
        12: invokevirtual #6                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        15: aload_1
        16: monitorexit
        17: goto          25
        20: astore_2
        21: aload_1
        22: monitorexit
        23: aload_2
        24: athrow
        25: return
```

#### synchronized修饰方法

对于synchronized关键字修饰方法来说，在方法的字节码指令中没有出现monitorenter与monitorexit指令，而是出现了一个ACC_SYNCHRONIZED标志。JVM使用了ACC_SYNCHRONIZED访问标志来区分一个方法是否为同步方法；当方法被调用时，调用指令会检查该方法是否拥有ACC_SYNCHRONIZED标志，如果有，那么执行线程将会先持有方法所在对象的monitor对象，然后再去执行方法体；在该方法执行起劲啊，其它任何线程均无法再获取到这个monitor对象，当线程执行完该方法后，他会释放掉这个monitor对象。

对如下代码进行反编译：MyTest5.java

```java
public class MyTest5 {
    public static synchronized void method(){
        System.out.println("----test---");
    }
}
```



```properties
public static synchronized void method();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_STATIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=0, args_size=0
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String ----test---
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 6: 0
        line 7: 8
```



JVM中的同步时基于进入与退出监视器对象（管程对象）(monitor)来是实现的，每个对象实例都会有一个monitor对象，monitor对象会和java对象一同创建并销毁，monitor对象是由c++来实现的

当多个线程同时访问一段同步代码时，这些线程会被放到一个EntryList集合中，处于阻塞状态的线程都会被放到该列表当中，接下来，当线程获取到对象的monitor时，monitor是依赖于底层操作系统的mutex lock 来实现互斥的，线程获取mutex成功，则会持有mutex，这时其它线程就无法再获取到mutex。

如果线程调用了wait方法，那么该线程会释放掉所持有的mutex，并且该线程会进入到WaitSet集合中，等待下一次被其它线程调用notify/notifyAll唤醒。如果当前线程顺利执行完方法，那么它也会释放掉所持有的mutex。

总结一下：同步锁在这种实现方式中，因为monitor是依赖底层操作系统来实现的，这样就存在用户态和内核态之间的切换，所以会增加性能开销。

通过对象互斥锁的概念来保证共享数据操作的完整性，每个对象都对象于一个可称为【互斥锁】的标记，这个标记用于保证在任何时刻，只能有一个线程访问对象。

那些处于EntryList与WaitSet中的线程均处于阻塞状态，阻塞状态是由操作系统来完成的，在linux下是通过pthred_mutx_lock函数实现的，线程被阻塞后便会进入到内核调度状态，这会导致系统在用户态和内核态之间按的切换，严重影响锁的性能。

解决上述问题的办法便是自旋(Spin)，其原理是：当发生对monitor的争用时，若owner能够在很短的时间内释放掉锁，则那些正在争用的线程就可以稍微等待一下(即所谓的自旋)，在owner线程释放锁之后，争用线程可能会立刻获取到锁，从而避免了系统阻塞。不过，当owner运行的时间超过了临界值后，争用线程自旋一段时间后仍无法获取到锁，这时争用线程则会停止自旋而进入到阻塞状态，所以总体的思想是：先自旋，不成功在进行阻塞，尽量降低阻塞的可能性，这对那些执行时间很短的代码块来说所有极大的性能提升，显然，自旋在多核处理器上才有意义。



互斥锁的属性：

1. PTHREAD_MUTEX_TIMED_NP:这是缺省值，也就是普通锁，当一个线程加锁后，其余请求锁的线程将会形成一个等待队列，并且在解锁后按照优先级获取到锁。这种策略可以保证资源分配的公平性
2. PHTREAD_MUTEX_RECURSIVE_NP：嵌套锁，允许同一个线程对同一个锁成功获取多次，并通过unlock解锁，如果是不同线程请求，则在加锁时重新竞争
3. PHTREAD_MUTEX_ERRORCHECK_NP：检错锁，如果一个线程请求同一个锁，则返回EDEADLK，否则与PTHREAD_MUTEX_TIMED_NP类型动作相同，这样就保证了当不允许多次加锁时不会出现最简单情况下的死锁
4. PTHREAD_MUTEX_ADAPTIVE_NP：适应锁，动作最简单的类型，仅仅等待解锁后重新竞争





### 查看monitor的源码实现

[点击](http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/9deea71d83dd/src/share/vm/runtime/objectMonitor.hpp)打开在线查看openjdk的源码

有一个ObjectWaiter类，表示的是一个线程或者说一个代理线程，里面含有指向上一个ObjectWaiter和下一个ObjectWaiter的指针

```c
class ObjectWaiter : public StackObj {
 public:
  enum TStates { TS_UNDEF, TS_READY, TS_RUN, TS_WAIT, TS_ENTER, TS_CXQ } ;
  enum Sorted  { PREPEND, APPEND, SORTED } ;
  ObjectWaiter * volatile _next;
  ObjectWaiter * volatile _prev;
  Thread*       _thread;
  jlong         _notifier_tid;
  ParkEvent *   _event;
  volatile int  _notified ;
  volatile TStates TState ;
  Sorted        _Sorted ;           // List placement disposition
  bool          _active ;           // Contention monitoring is enabled
 public:
  ObjectWaiter(Thread* thread);
  void wait_reenter_begin(ObjectMonitor *mon);
  void wait_reenter_end(ObjectMonitor *mon);
};
```

ObjectMonitor表示的就是moniter对象，里面有 _ WaitSet 集合和 _ EntryList集合，这两个集合都是ObjectWaiter类型，都是表示正在阻塞的线程的集合。

```c
ObjectMonitor() {
    _header       = NULL;
    _count        = 0;
    _waiters      = 0,
    _recursions   = 0;
    _object       = NULL;
    _owner        = NULL;
    _WaitSet      = NULL;
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ;
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
    _previous_owner_tid = 0;
  }
```

### 查看wait的源码实现

[打开网址](http://hg.openjdk.java.net/jdk8u/jdk8u/hotspot/file/9deea71d83dd/src/share/vm/runtime/objectMonitor.cpp)，查看wait方法中的代码：

```c
ObjectWaiter node(Self);    //将当前线程封装成ObjectWatier
...
Thread::SpinAcquire (&_WaitSetLock, "WaitSet - add") ;  // 自旋操作
AddWaiter (&node) ;   // 添加到WaitSet节点中
...
```

查看AddWaiter()方法，就是将ObjectWaiter对象添加到WaitSet节点中，使用双向链表的数据结构

```c
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
    ObjectWaiter* head = _WaitSet ;
    ObjectWaiter* tail = head->_prev;
    assert(tail->_next == head, "invariant check");
    tail->_next = node;
    head->_prev = node;
    node->_next = head;
    node->_prev = tail;
  }
}
```

### 查看notify方法源码实现

根据不同的策略，将从WaitSet集合中移出来的ObjectWaiter对象加入到EntryList集合中

```c
void ObjectMonitor::notify(TRAPS) {
    ...
     ObjectWaiter * iterator = DequeueWaiter() ;
	...
	 if (Policy == 0) {   // prepend to EntryList
         ...
         if (List == NULL) {
             iterator->_next = iterator->_prev = NULL ;
             _EntryList = iterator ;
         } else {
             List->_prev = iterator ;
             iterator->_next = List ;
             iterator->_prev = NULL ;
             _EntryList = iterator ;}
         ... }
	 if (Policy == 1) {...}   // append to EntryList
	 if (Policy == 2) {...}      // prepend to cxq
	 if (Policy == 3) {...}      // append to cxq
	...
}
```

