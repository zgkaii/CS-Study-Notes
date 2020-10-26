## 5 park & unpack

### 5.1 基本使用

它们是 LockSupport 类中的方法

```java
// 暂停当前线程
LockSupport.park();
// 恢复某个线程的运行
LockSupport.unpark;
```

### 5.2 park unpark 原理

每个线程都有自己的一个 Parker 对象，由三部分组成 _counter， _cond和 _mutex

1. 打个比喻线程就像一个旅人，Parker 就像他随身携带的背包，条件变量 _ cond就好比背包中的帐篷。_counter 就好比背包中的备用干粮（0 为耗尽，1 为充足）
2. 调用 park 就是要看需不需要停下来歇息
   1. 如果备用干粮耗尽，那么钻进帐篷歇息
   2. 如果备用干粮充足，那么不需停留，继续前进
3. 调用 unpark，就好比令干粮充足
   1. 如果这时线程还在帐篷，就唤醒让他继续前进
   2. 如果这时线程还在运行，那么下次他调用 park 时，仅是消耗掉备用干粮，不需停留继续前进
      1. 因为背包空间有限，多次调用 unpark 仅会补充一份备用干粮

可以不看例子，直接看实现过程

#### 5.2.1 先调用park再调用upark的过程

1.先调用park

1. 当前线程调用 Unsafe.park() 方法
2. 检查 _counter ，本情况为 0，这时，获得 _mutex 互斥锁(mutex对象有个等待队列 _cond)
3. 线程进入 _cond 条件变量阻塞
4. 设置 _counter = 0

![1594531894163](https://gitee.com/gu_chun_bo/picture/raw/master/image/20200712133136-910801.png)

2.调用upark

1. 调用 Unsafe.unpark(Thread_0) 方法，设置 _counter 为 1
2. 唤醒 _cond 条件变量中的 Thread_0
3. Thread_0 恢复运行
4. 设置 _counter 为 0

![1594532057205](D:/workplace/IdeaProjects/CS-Notes-Kz/01-basic-learning/01-language/01-java-basic/03-concurrent&multithreaded/assets/1594532057205.png)

#### 5.2.2 先调用upark再调用park的过程

1. 调用 Unsafe.unpark(Thread_0) 方法，设置 _counter 为 1
2. 当前线程调用 Unsafe.park() 方法
3. 检查 _counter ，本情况为 1，这时线程无需阻塞，继续运行
4. 设置 _counter 为 0

![1594532135616](https://gitee.com/gu_chun_bo/picture/raw/master/image/20200712133539-357066.png)

