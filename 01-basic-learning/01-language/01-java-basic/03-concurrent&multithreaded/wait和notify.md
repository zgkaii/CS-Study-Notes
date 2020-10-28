## 5 wait和notify

建议先看看wait和notify方法的javadoc文档

### 5.1 同步模式之保护性暂停

即 Guarded Suspension，用在一个线程等待另一个线程的执行结果，要点：

1. 有一个结果需要从一个线程传递到另一个线程，让他们关联同一个 GuardedObject
2. 如果有结果不断从一个线程到另一个线程那么可以使用消息队列（见生产者/消费者）
3. JDK 中，join 的实现、Future 的实现，采用的就是此模式
4. 因为要等待另一方的结果，因此归类到同步模式

代码：Test22.java    Test23.java这是带超时时间的

![](https://img-blog.csdnimg.cn/20201026221941471.png)

关于超时的增强，在join(long millis) 的源码中得到了体现：

```java
    public final synchronized void join(long millis)
    throws InterruptedException {
        long base = System.currentTimeMillis();
        long now = 0;

        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }
		
        if (millis == 0) {
            while (isAlive()) {
                wait(0);
            }
        } else {
        // join一个指定的时间
            while (isAlive()) {
                long delay = millis - now;
                if (delay <= 0) {
                    break;
                }
                wait(delay);
                now = System.currentTimeMillis() - base;
            }
        }
    }
```



多任务版 GuardedObject图中 Futures 就好比居民楼一层的信箱（每个信箱有房间编号），左侧的 t0，t2，t4 就好比等待邮件的居民，右侧的 t1，t3，t5 就好比邮递员如果需要在多个类之间使用 GuardedObject 对象，作为参数传递不是很方便，因此设计一个用来解耦的中间类，这样不仅能够解耦【结果等待者】和【结果生产者】，还能够同时支持多个任务的管理。和生产者消费者模式的区别就是：这个生产者和消费者之间是一一对应的关系，但是生产者消费者模式并不是。rpc框架的调用中就使用到了这种模式。  Test24.java

![1594518049426](D:/workplace/IdeaProjects/CS-Notes-Kz/01-basic-learning/01-language/01-java-basic/03-concurrent&multithreaded/assets/1594518049426.png)

### 5.2 异步模式之生产者/消费者

要点

1. 与前面的保护性暂停中的 GuardObject 不同，不需要产生结果和消费结果的线程一一对应
2. 消费队列可以用来平衡生产和消费的线程资源
3. 生产者仅负责产生结果数据，不关心数据该如何处理，而消费者专心处理结果数据
4. 消息队列是有容量限制的，满时不会再加入数据，空时不会再消耗数据
5. JDK 中各种[阻塞队列](https://blog.csdn.net/yanpenglei/article/details/79556591)，采用的就是这种模式

“异步”的意思就是生产者产生消息之后消息没有被立刻消费，而“同步模式”中，消息在产生之后被立刻消费了。

![1594524622020](D:/workplace/IdeaProjects/CS-Notes-Kz/01-basic-learning/01-language/01-java-basic/03-concurrent&multithreaded/assets/1594524622020.png)

我们写一个线程间通信的消息队列，要注意区别，像rabbit mq等消息框架是进程间通信的。

### 5.3 同步模式之顺序控制

1. 固定运行顺序，比如，必须先 2 后 1 打印
   1.  wait notify 版  Test35.java
   2.  Park Unpark 版  Test36.java
2. 交替输出，线程 1 输出 a 5 次，线程 2 输出 b 5 次，线程 3 输出 c 5 次。现在要求输出 abcabcabcabcabc 怎么实现
   1. wait notify 版   Test37.java
   2. Lock 条件变量版 Test38.java
   3. Park Unpark 版 Test39.java