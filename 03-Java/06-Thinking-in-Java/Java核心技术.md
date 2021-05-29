## 01 谈谈你对Java平台的理解？“Java是解释执行”，这句话正确吗？

​	Java本身是一种面向对象的语言，最显著的特性有两个方面，一是所谓的“书写一次，到处运行”（Write once, run anywhere），能够非常容易地获得跨平台能力；另外就是垃圾收集（GC, Garbage Collection），Java通过垃圾收集器（Garbage Collector）回收分配内存，大部分情况下，程序员不需要自己操心内存的分配和回收。
​	我们日常会接触到JRE（Java Runtime Environment）或者JDK（Java Development Kit）。 JRE，也就是Java运行环境，包含了JVM和Java类库，以及一些模块等。而JDK可以看作是JRE的一个超集，提供了更多工具，比如编译器、各种诊断工具等。
​	对于“Java是解释执行”这句话，这个说法不太准确。我们开发的Java的源代码，首先通过Javac编译成为字节码（bytecode），然后，在运行时，通过 Java虚拟机（JVM）内嵌的解释器将字节码转换成为最终的机器码。但是常见的JVM，比如我们大多数情况使用的Oracle JDK提供的Hotspot JVM，都提供了JIT（Just-In-Time）编译器，也就是通常所说的动态编译器，JIT能够在运行时将热点代码编译成机器码，这种情况下部分热点代码就属于编译执行，而不是解释执行了。

## 02 Exception和Error有什么区别？

​	Exception和Error都是继承了Throwable类，在Java中只有Throwable类型的实例才可以被抛出（throw）或者捕获（catch），它是异常处理机制的基本组成类型。
​	Exception和Error体现了Java平台设计者对不同异常情况的分类。Exception是程序正常运行中，可以预料的意外情况，可能并且应该被捕获，进行相应处理。
​	Error是指在正常情况下，不大可能出现的情况，绝大部分的Error都会导致程序（比如JVM自身）处于非正常的、不可恢复状态。既然是非正常情况，所以不便于也不需要捕获，常见的比如OutOfMemoryError之类，都是Error的子类。
​	Exception又分为可检查（checked）异常和不检查（unchecked）异常，可检查异常在源代码里必须显式地进行捕获处理，这是编译期检查的一部分。前面我介绍的不可查的Error，是Throwable不是Exception。不检查异常就是所谓的运行时异常，类似 NullPointerException、ArrayIndexOutOfBoundsException之类，通常是可以编码避免的逻辑错误，具体根据需要来判断是否需要捕
获，并不会在编译期强制要求。

## 03 final、finally、 finalize有什么不同？

​	final可以用来修饰类、方法、变量，分别有不同的意义，final修饰的class代表不可以继承扩展，final的变量是不可以修改的，而final的方法也是不可以重写的（override）。
​	finally则是Java保证重点代码一定要被执行的一种机制。我们可以使用try-finally或者try-catch-finally来进行类似关闭JDBC连接、保证unlock锁等动作。
​	finalize是基础类java.lang.Object的一个方法，它的设计目的是保证对象在被垃圾收集前完成特定资源的回收。finalize机制现在已经不推荐使用，并且在JDK 9开始被标记为deprecated。

## 04 强引用、软引用、弱引用、幻象引用有什么区别？

**1 强引用**
	特点：我们平常典型编码`Object obj = new Object()`中的obj就是强引用。通过关键字new创建的对象所关联的引用就是强引用。 当JVM内存空间不足，JVM宁愿抛出OutOfMemoryError运行时错误（OOM），使程序异常终止，也不会靠随意回收具有强引用的“存活”对象来解决内存不足的问题。对于一个普通的对象，如果没有其他的引用关系，只要超过了引用的作用域或者显式地将相应（强）引用赋值为 null，就是可以被垃圾收集的了，具体回收时机还是要看垃圾收集策略。
**2 软引用**
	特点：软引用通过SoftReference类实现。 软引用的生命周期比强引用短一些。只有当 JVM 认为内存不足时，才会去试图回收软引用指向的对象：即JVM 会确保在抛出 OutOfMemoryError之前，清理软引用指向的对象。软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收器回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中。后续，我们可以调用ReferenceQueue的poll()方法来检查是否有它所关心的对象被回收。如果队列为空，将返回一个null,否则该方法返回队列中前面的一个Reference对象。
	应用场景：软引用通常用来实现内存敏感的缓存。如果还有空闲内存，就可以暂时保留缓存，当内存不足时清理掉，这样就保证了使用缓存的同时，不会耗尽内存。
**3 弱引用**
	特点：弱引用通过WeakReference类实现。弱引用的生命周期比软引用短。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。由于垃圾回收器是一个优先级很低的线程，因此不一定会很快回收弱引用的对象。弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中。
	应用场景：弱应用同样可用于内存敏感的缓存。
**4 虚引用**
	特点：虚引用也叫**幻象引用**，通过PhantomReference类来实现。无法通过虚引用访问对象的任何属性或函数。幻象引用仅仅是提供了一种确保对象被 fnalize 以后，做某些事情的机制。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。虚引用必须和引用队列 （ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。

```java
ReferenceQueue queue = new ReferenceQueue ();
PhantomReference pr = new PhantomReference (object, queue);
```

​	程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。如果程序发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取一些程序行动。
​	应用场景：可用来跟踪对象被垃圾回收器回收的活动，当一个虚引用关联的对象被垃圾收集器回收之前会收到一条系统通知。

## 05 String、StringBuffer、StringBuilder有什么区别？

​	String是Java语言非常基础和重要的类，提供了构造和管理字符串的各种基本逻辑。它是典型的Immutable类，被声明成为final class，所有属性也都是final的。也由于它的不可变性，类似拼接、裁剪字符串等动作，都会产生新的String对象。由于字符串操作的普遍性，所以相关操作的效率往往对应用性能有明显影响。
​	StringBuffer是为解决上面提到拼接产生太多中间对象的问题而提供的一个类，我们可以用append或者add方法，把字符串添加到已有序列的末尾或者指定位置。StringBuffer本质是一个**线程安全**的可修改字符序列，它保证了线程安全，也随之带来了额外的性能开销，所以除非有线程安全的需要，不然还是推荐使用它的后继者，也就是StringBuilder。
​	StringBuilder是Java 1.5中新增的，在能力上和StringBufer没有本质区别，但是它去掉了线程安全的部分，有效减小了开销，是绝大部分情况下进行字符串拼接的首选。

应用场景：

* 在字符串内容不经常发生变化的业务场景优先使用String类。例如：常量声明、少量的字符串拼接操作等。如果有大量的字符串内容拼接，避免使用String与String之间的“+”操作，因为这样会产生大量无用的中间对象，耗费空间且执行效率低下（新建对象、回收对象花费大量时间）。
* 在频繁进行字符串的运算（如拼接、替换、删除等），并且运行在多线程环境下，建议使用StringBuffer，例如XML解析、HTTP参数解析与封装。
* 在频繁进行字符串的运算（如拼接、替换、删除等），并且运行在单线程环境下，建议使用StringBuilder，例如SQL语句拼装、JSON封装等。

## 06 动态代理是基于什么原理？

​	反射机制是Java语言提供的一种基础功能，赋予程序在运行时自省（introspect，官方用语）的能力。通过反射我们可以直接操作类或者对象，比如获取某个对象的类定义，获取类声明的属性和方法，调用方法或者构造对象，甚至可以运行时修改类定义。
​	动态代理是一种方便运行时动态构建代理、动态处理代理方法调用的机制，很多场景都是利用类似机制做到的，比如用来包装RPC调用、面向切面的编程（AOP）。实现动态代理的方式很多，比如JDK自身提供的动态代理，就是主要利用了上面提到的反射机制。还有其他的实现方式，比如利用传说中更高性能的字节码操作机制，类似ASM、cglib（基于ASM）、Javassist等。

## 07 int和Integer有什么区别？

​	int是我们常说的整形数字，是Java的8个原始数据类型（Primitive Types，boolean、byte 、short、char、int、foat、double、long）之一。Java语言虽然号称一切都是对象，但原始数据类型是例外。
​	Integer是int对应的包装类，它有一个int类型的字段存储数据，并且提供了基本操作，比如数学运算、int和字符串之间转换等。在Java 5中，引入了自动装箱和自动拆箱功能（boxing/unboxing），Java可以根据上下文，自动进行转换，极大地简化了相关编程。
​	关于Integer的值缓存，这涉及Java 5中另一个改进。构建Integer对象的传统方式是直接调用构造器，直接new一个对象。但是根据实践，我们发现大部分数据操作都是集中在有限的、较小的数值范围，因而，在Java 5中新增了静态工厂方法valueOf，在调用它的时候会利用一个缓存机制，带来了明显的性能改进。按照Javadoc，这个值默认缓存是-128到127之间。

## 08 对比Vector、ArrayList、LinkedList有何区别？

​	这三者都是实现集合框架中的List，也就是所谓的有序集合，因此具体功能也比较近似，比如都提供按照位置进行定位、添加或者删除的操作，都提供迭代器以遍历其内容等。但因为具体的设计区别，在行为、性能、线程安全等方面，表现又有很大不同。
​	Vector是Java早期提供的线程安全的动态数组，如果不需要线程安全，并不建议选择，毕竟同步是有额外开销的。Vector内部是使用对象数组来保存数据，可以根据需要自动的增加容量，当数组已满时，会创建新的数组，并拷贝原有数组数据。
​	ArrayList是应用更加广泛的动态数组实现，它本身不是线程安全的，所以性能要好很多。与Vector近似，ArrayList也是可以根据需要调整容量，不过两者的调整逻辑有所区别，Vector在扩容时会提高1倍，而ArrayList则是增加50%。
​	LinkedList顾名思义是Java提供的双向链表，所以它不需要像上面两种那样调整容量，它也不是线程安全的。

## 09 对比Hashtable、HashMap、TreeMap有什么不同？

* 元素特性
  	Hashtable中的key、value都不能为null；HashMap中的key、value可以为null，很显然只能有一个key为null的键值对，但是允许有多个值为null的键值对；TreeMap中当未实现Comparator 接口时，key 不可以为null；当实现 Comparator 接口时，若未对null情况进行判断，则key不可以为null，反之亦然。
* 顺序特性
  	Hashtable、HashMap具有无序特性。TreeMap是利用红黑树来实现的（树中的每个节点的值，都会大于或等于它的左子树种的所有节点的值，并且小于或等于它的右子树中的所有节点的值），实现了SortMap接口，能够对保存的记录根据键进行排序。所以一般需要排序的情况下是选择TreeMap来进行，默认为升序排序方式（深度优先搜索），可自定义实现Comparator接口
  实现排序方式。
* 初始化与增长方式
  	初始化时：Hashtable在不指定容量的情况下的默认容量为11，且不要求底层数组的容量一定要为2的整数次幂；HashMap默认容量为16，且要求容量一定为2的整数次幂。扩容时：Hashtable将容量变为原来的2倍加1；HashMap扩容将容量变为原来的2倍。
* 线程安全性
  	Hashtable其方法函数都是同步的（采用synchronized修饰），不会出现两个线程同时对数据进行操作的情况，因此保证了线程安全性。也正因为如此，在多线程运行环境下效率表现非常低下。因为当一个线程访问Hashtable的同步方法时，其他线程也访问同步方法就会进入阻塞状态。比如当一个线程在添加数据时候，另外一个线程即使执行获取其他数据的操作也必须被阻塞，大大降低了程序的运行效率，在新版本中已被废弃，不推荐使用。
  	HashMap不支持线程的同步，即任一时刻可以有多个线程同时写HashMap;可能会导致数据的不一致。如果需要同步（1）可以用 Collections的synchronizedMap方法；（2）使用ConcurrentHashMap类，相较于Hashtable锁住的是对象整体， ConcurrentHashMap基于lock实现锁分段技术。首先将Map存放的数据分成一段一段的存储方式，然后给每一段数据分配一把锁，当一个线程占用锁访问其中一个段的数据时，其他段的数据也能被其他线程访问。ConcurrentHashMap不仅保证了多线程运行环境下的数据访问安全性，而且性能上有长足的提升。
* 一段话总结HashMap
  	HashMap基于哈希思想，实现对数据的读写。当我们将键值对传递给put()方法时，它调用键对象的hashCode()方法来计hashcode，让后找到bucket位置来储存值对象。当获取对象时，通过键对象的equals()方法找到正确的键值对，然后返回值对象。HashMap使用链表来解决碰撞问题，当发生碰撞了，对象将会储存在链表的下一个节点中。 HashMap在每个链表节点中储存键值对对象。当两个不同的键对象的hashcode相同时，它们会储存在同一个bucket位置的链表中，可通过键对象的equals()方法用来找到键值对。如果链表大小超过阈值（TREEIFY_THRESHOLD, 8），链表就会被改造为树形结构。

## 10 如何保证集合是线程安全的? ConcurrentHashMap如何实现高效地线程安全?

​	Java提供了不同层面的线程安全支持。在传统集合框架内部，除了Hashtable等同步容器，还提供了所谓的同步包装器（Synchronized Wrapper），我们可以调用Collections工具类提供的包装方法，来获取一个同步的包装容器（如Collections.synchronizedMap），但是它们都是利用非常粗粒度的同步方式，在高并发情况下，性能比较低下。
​	另外，更加普遍的选择是利用并发包提供的线程安全容器类，它提供了：

* 各种并发容器，比如ConcurrentHashMap、CopyOnWriteArrayList。
* 各种线程安全队列（Queue/Deque），如ArrayBlockingQueue、SynchronousQueue。
* 各种有序容器的线程安全版本等。
* 具体保证线程安全的方式，包括有从简单的synchronize方式，到基于更加精细化的，比如基于分离锁实现的ConcurrentHashMap等并发实现等。具体选择要看开发的场景需求，总体来说，并发包内提供的容器通用场景，远优于早期的简单同步实现。

## 11 Java提供了哪些IO方式？ NIO如何实现多路复用？

​	Java IO方式有很多种，基于不同的IO抽象模型和交互方式，可以进行简单区分。
首先，传统的java.io包，它基于流模型实现，提供了我们最熟知的一些IO功能，比如File抽象、输入输出流等。交互方式是同步、阻塞的方式，也就是说，在读取输入流或者写入输出流时，在读、写动作完成之前，线程会一直阻塞在那里，它们之间的调用是可靠的线性顺序。
​	java.io包的好处是代码比较简单、直观，缺点则是IO效率和扩展性存在局限性，容易成为应用性能的瓶颈。
​	很多时候，人们也把java.net下面提供的部分网络API，比如Socket、ServerSocket、HttpURLConnection也归类到同步阻塞IO类库，因为网络通信同样是IO行为。
​	第二，在Java 1.4中引入了NIO框架（java.nio包），提供了Channel、Selector、Bufer等新的抽象，可以构建多路复用的、同步非阻塞IO程序，同时提供了更接近操作系统底层的高性能数据操作方式。
​	第三，在Java 7中，NIO有了进一步的改进，也就是NIO 2，引入了异步非阻塞IO方式，也有很多人叫它AIO（Asynchronous IO）。异步IO操作基于事件和回调机制，可以简单理解为，应用操作直接返回，而不会阻塞在那里，当后台处理完成，操作系统会通知相应线程进行后续工作。

## 12 Java有几种文件拷贝方式？哪一种最高效？

​	Java有多种比较典型的文件拷贝实现方式，比如：利用java.io类库，直接为源文件构建一个FileInputStream读取，然后再为目标文件构建一个FileOutputStream，完成写入工作。

```java
  public static void copyFileByStream(File source, File des) throws IOException {
    try (
      InputStream is = new FileInputStream(source);
      OutputStream os = new FileOutputStream(des);
    ) {
      byte[] bufer = new byte[1024];
      int length;
      while ((length = is.read(bufer)) > 0) {
        os.write(bufer, 0, length);
      }
    }
  }
```

或者，利用java.nio类库提供的transferTo或transferFrom方法实现。

```java
  public static void copyFileByChannel(File source, File des) throws IOException {
    try (
      FileChannel sourceChannel = new FileInputStream(source).getChannel();
      FileChannel targetChannel = new FileOutputStream(des).getChannel();
    ) {
      for (long count = sourceChannel.size(); count > 0;) {
        long transferred = sourceChannel.transferTo(
          sourceChannel.position(),
          count,
          targetChannel
        );
        sourceChannel.position(sourceChannel.position() + transferred);
        count -= transferred;
      }
    }
  }
```

​	当然，Java标准类库本身已经提供了几种Files.copy的实现。
​	对于Copy的效率，这个其实与操作系统和配置等情况相关，总体上来说，NIO transferTo/From的方式可能更快，因为它更能利用现代操作系统底层机制，避免不必要拷贝和上下文切换。

## 13 谈谈接口和抽象类有什么区别？

​	接口和抽象类是Java面向对象设计的两个基础机制。
​	接口是对行为的抽象，它是抽象方法的集合，利用接口可以达到API定义和实现分离的目的。接口，不能实例化；不能包含任何非常量成员，任何feld都是隐含着public static final的意义；同时，没有非静态方法实现，也就是说要么是抽象方法，要么是静态方法。Java标准类库中，定义了非常多的接口，比如java.util.List。
​	抽象类是不能实例化的类，用abstract关键字修饰class，其目的主要是代码重用。除了不能实例化，形式上和一般的Java类并没有太大区别，可以有一个或者多个抽象方法，也可以没有抽象方法。抽象类大多用于抽取相关Java类的共用方法实现或者是共同成员变量，然后通过继承的方式达到代码复用的目的。Java标准库中，比如collection框架，很多通用部分就被抽取成为抽象类，例如java.util.AbstractList。
​	Java类实现interface使用implements关键词，继承abstract class则是使用extends关键词，我们可以参考Java标准库中的ArrayList。

```java
  public class ArrayList<E> extends AbstractList<E> 
  		implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    //...
  }
```

## 14 谈谈你知道的设计模式？

大致按照模式的应用目标分类，设计模式可以分为创建型模式、结构型模式和行为型模式。

* 创建型模式，是对对象创建过程的各种问题和解决方案的总结，包括各种工厂模式（Factory、Abstract Factory）、单例模式（Singleton）、构建器模式（Builder）、原型模式（ProtoType）。
* 结构型模式，是针对软件设计结构的总结，关注于类、对象继承、组合方式的实践经验。常见的结构型模式，包括桥接模式（Bridge）、适配器模式（Adapter）、装饰者模式（Decorator）、代理模式（Proxy）、组合模式（Composite）、外观模式（Facade）、享元模式（Flyweight）等。
* 行为型模式，是从类或对象之间交互、职责划分等角度总结的模式。比较常见的行为型模式有策略模式（Strategy）、解释器模式（Interpreter）、命令模式（Command）、观察者模式（Observer）、迭代器模式（Iterator）、模板方法模式（Template Method）、访问者模式（Visitor）。

## 15 synchronized和ReentrantLock有什么区别呢？

​	synchronized是Java内建的同步机制，所以也有人称其为Intrinsic Locking，它提供了互斥的语义和可见性，当一个线程已经获取当前锁时，其他试图获取的线程只能等待或者阻塞在那里。
​	在Java 5以前，synchronized是仅有的同步手段，在代码中， synchronized可以用来修饰方法，也可以使用在特定的代码块儿上，本质上synchronized方法等同于把方法全部语句用synchronized块包起来。
​	ReentrantLock，通常翻译为再入锁，是Java 5提供的锁实现，它的语义和synchronized基本相同。再入锁通过代码直接调用lock()方法获取，代码书写也更加灵活。与此同时，ReentrantLock提供了很多实用的方法，能够实现很多synchronized无法做到的细节控制，比如可以控制fairness，也就是公平性，或者利用定义条件等。但是，编码中也需
​	要注意，必须要明确调用unlock()方法释放，不然就会一直持有该锁。
​	synchronized和ReentrantLock的性能不能一概而论，早期版本synchronized在很多场景下性能相差较大，在后续版本进行了较多改进，在低竞争场景中表现可能优于ReentrantLock。

## 16 synchronized底层如何实现？什么是锁的升级、降级？

​	synchronized代码块是由一对儿monitorenter/monitorexit指令实现的，Monitor对象是同步的基本实现单元。
​	在Java 6之前，Monitor的实现完全是依靠操作系统内部的互斥锁，因为需要进行用户态到内核态的切换，所以同步操作是一个无差别的重量级操作。
​	现代的（Oracle）JDK中，JVM对此进行了大刀阔斧地改进，提供了三种不同的Monitor实现，也就是常说的三种不同的锁：偏斜锁（Biased Locking）、轻量级锁和重量级锁，大大改进了其性能。
​	所谓锁的升级、降级，就是JVM优化synchronized运行的机制，当JVM检测到不同的竞争状况时，会自动切换到适合的锁实现，这种切换就是锁的升级、降级。
​	当没有竞争出现时，默认会使用偏斜锁。JVM会利用CAS操作（compare and swap），在对象头上的Mark Word部分设置线程ID，以表示这个对象偏向于当前线程，所以并不涉及真正的互斥锁。这样做的假设是基于在很多应用场景中，大部分对象生命周期中最多会被一个线程锁定，使用偏斜锁可以降低无竞争开销。
​	如果有另外的线程试图锁定某个已经被偏斜过的对象，JVM就需要撤销（revoke）偏斜锁，并切换到轻量级锁实现。轻量级锁依赖CAS操作Mark Word来试图获取锁，如果重试成功，就使用普通的轻量级锁；否则，进一步升级为重量级锁。
​	我注意到有的观点认为Java不会进行锁降级。实际上据我所知，锁降级确实是会发生的，当JVM进入安全点（SafePoint）的时候，会检查是否有闲置的Monitor，然后试图进行降级。

## 17 一个线程两次调用start()方法会出现什么情况？

​	Java的线程是不允许启动两次的，第二次调用必然会抛出IllegalThreadStateException，这是一种运行时异常，多次调用start被认为是编程错误。
​	关于线程生命周期的不同状态，在Java 5以后，线程状态被明确定义在其公共内部枚举类型java.lang.Thread.State中，分别是：

* 新建（NEW），表示线程被创建出来还没真正启动的状态，可以认为它是个Java内部状态。
* 就绪（RUNNABLE），表示该线程已经在JVM中执行，当然由于执行需要计算资源，它可能是正在运行，也可能还在等待系统分配给它CPU片段，在就绪队列里面排队。
* 在其他一些分析中，会额外区分一种状态RUNNING，但是从Java API的角度，并不能表示出来。
  阻塞（BLOCKED），这个状态和我们前面两讲介绍的同步非常相关，阻塞表示线程在等待Monitor lock。比如，线程试图通过synchronized去获取某个锁，但是其他线程已经独占了，那么当前线程就会处于阻塞状态。
* 等待（WAITING），表示正在等待其他线程采取某些操作。一个常见的场景是类似生产者消费者模式，发现任务条件尚未满足，就让当前消费者线程等待（wait），另外的生产者线程去准备任务数据，然后通过类似notify等动作，通知消费线程可以继续工作了。Thread.join()也会令线程进入等待状态。
* 计时等待（TIMED_WAIT），其进入条件和等待状态类似，但是调用的是存在超时条件的方法，比如wait或join等方法的指定超时版本，如下面示例：

```java
public final native void wait(long timeout) throws InterruptedException;
```

* 终止（TERMINATED），不管是意外退出还是正常执行结束，线程已经完成使命，终止运行，也有人把这个状态叫作死亡。
  在第二次调用start()方法的时候，线程可能处于终止或者其他（非NEW）状态，但是不论如何，都是不可以再次启动的。

## 18 什么情况下Java程序会产生死锁？如何定位、修复？

死锁是一种特定的程序状态，在实体之间，由于循环依赖导致彼此一直处于等待之中，没有任何个体可以继续前进。死锁不仅仅是在线程之间会发生，存在资源独占的进程之间同样也可能出现死锁。通常来说，我们大多是聚焦在多线程场景中的死锁，指两个或多个线程之间，由于互相持有对方需要的锁，而永久处于阻塞的状态。

![](https://img-blog.csdnimg.cn/20210118163203909.png)

​	定位死锁最常见的方式就是利用jstack等工具获取线程栈，然后定位互相之间的依赖关系，进而找到死锁。如果是比较明显的死锁，往往jstack等就能直接定位，类似JConsole甚至可以在图形界面进行有限的死锁检测。
​	如果程序运行时发生了死锁，绝大多数情况下都是无法在线解决的，只能重启、修正程序本身问题。所以，代码开发阶段互相审查，或者利用工具进行预防性排查，往往也是很重要
的。

## 19 Java并发包提供了哪些并发工具类？

我们通常所说的并发包也就是java.util.concurrent及其子包，集中了Java并发的各种基础工具类，具体主要包括几个方面：

* 提供了比synchronized更加高级的各种同步结构，包括CountDownLatch、CyclicBarrier、Semaphore等，可以实现更加丰富的多线程操作，比如利用Semaphore作为资源控制器，限制同时进行工作的线程数量。
* 各种线程安全的容器，比如最常见的ConcurrentHashMap、有序的ConcunrrentSkipListMap，或者通过类似快照机制，实现线程安全的动态数组CopyOnWriteArrayList等。
* 各种并发队列实现，如各种BlockedQueue实现，比较典型的ArrayBlockingQueue、 SynchorousQueue或针对特定场景的PriorityBlockingQueue等。
* 强大的Executor框架，可以创建各种不同类型的线程池，调度任务运行等，绝大部分情况下，不再需要自己从头实现线程池和任务调度器。

## 20 并发包中的ConcurrentLinkedQueue和LinkedBlockingQueue有什么区别？

有时候我们把并发包下面的所有容器都习惯叫作并发容器，但是严格来讲，类似ConcurrentLinkedQueue这种“`Concurrent*`”容器，才是真正代表并发。
关于问题中它们的区别：

* Concurrent类型基于lock-free，在常见的多线程访问场景，一般可以提供较高吞吐量。
* 而LinkedBlockingQueue内部则是基于锁，并提供了BlockingQueue的等待性方法。

不知道你有没有注意到，java.util.concurrent包提供的容器（Queue、List、Set）、Map，从命名上可以大概区分为Concurrent、CopyOnWrite和`Blocking*`等三类，同样是线程安全容器，可以简单认为：

* Concurrent类型没有类似CopyOnWrite之类容器相对较重的修改开销。
* 但是，凡事都是有代价的，Concurrent往往提供了较低的遍历一致性。你可以这样理解所谓的弱一致性，例如，当利用迭代器遍历时，如果容器发生修改，迭代器仍然可以继续进行遍历。
* 与弱一致性对应的，就是我介绍过的同步容器常见的行为“fast-fail”，也就是检测到容器在遍历过程中发生了修改，则抛出ConcurrentModifcationException，不再继续遍历。
* 弱一致性的另外一个体现是，size等操作准确性是有限的，未必是100%准确。
  与此同时，读取的性能具有一定的不确定性。

## 21 Java并发类库提供的线程池有哪几种？ 分别有什么特点？

通常开发者都是利用Executors提供的通用线程池创建方法，去创建不同配置的线程池，主要区别在于不同的ExecutorService类型或者不同的初始参数。
Executors目前提供了5种不同的线程池创建配置：

* newCachedThreadPool()，它是一种用来处理大量短时间工作任务的线程池，具有几个鲜明特点：它会试图缓存线程并重用，当无缓存线程可用时，就会创建新的工作线程；如果线程闲置的时间超过60秒，则被终止并移出缓存；长时间闲置时，这种线程池，不会消耗什么资源。其内部使用SynchronousQueue作为工作队列。
* newFixedThreadPool(int nThreads)，重用指定数目（nThreads）的线程，其背后使用的是无界的工作队列，任何时候最多有nThreads个工作线程是活动的。这意味着，如果任务数量超过了活动队列数目，将在工作队列中等待空闲线程出现；如果有工作线程退出，将会有新的工作线程被创建，以补足指定的数目nThreads。
* newSingleThreadExecutor()，它的特点在于工作线程数目被限制为1，操作一个无界的工作队列，所以它保证了所有任务的都是被顺序执行，最多会有一个任务处于活动状态，并且不允许使用者改动线程池实例，因此可以避免其改变线程数目。
* newSingleThreadScheduledExecutor()和newScheduledThreadPool(int corePoolSize)，创建的是个ScheduledExecutorService，可以进行定时或周期性的工作调度，区别在于单一工作线程还是多个工作线程。
* newWorkStealingPool(int parallelism)，这是一个经常被人忽略的线程池，Java 8才加入这个创建方法，其内部会构建ForkJoinPool，利用Work-Stealing算法，并行地处
  理任务，不保证处理顺序。

## 22 AtomicInteger底层实现原理是什么？如何在自己的产品代码中应用CAS操作？

​	AtomicIntger是对int类型的一个封装，提供原子性的访问和更新操作，其原子性操作的实现是基于CAS（compare-and-swap）技术。
​	所谓CAS，表征的是一些列操作的集合，获取当前数值，进行一些运算，利用CAS指令试图进行更新。如果当前数值未变，代表没有其他线程进行并发修改，则成功更新。否则，可能出现不同的选择，要么进行重试，要么就返回一个成功或者失败的结果。
​	从AtomicInteger的内部属性可以看出，它依赖于Unsafe提供的一些底层能力，进行底层操作；以volatile的value字段，记录数值，以保证可见性。

```java
  private static final jdk.internal.misc.Unsafe U = jdk.internal.misc.Unsafe.getUnsafe();
  private static final long VALUE = U.objectFieldOffset(
    AtomicInteger.class,
    "value"
  );
  private volatile int value;
```

具体的原子操作细节，可以参考任意一个原子更新方法，比如下面的getAndIncrement。Unsafe会利用value字段的内存地址偏移，直接完成操作。

```java
    public final int getAndIncrement() {
    	return U.getAndAddInt(this, VALUE, 1);
    }
```

因为getAndIncrement需要返归数值，所以需要添加失败重试逻辑。

```java
public final int getAndAddInt(Object o, long offset, int delta) {
    int v;
    do {
    	v = getIntVolatile(o, offset);
    } while (!weakCompareAndSetInt(o, ofset, v, v + delta));
    return v;
}
```

而类似compareAndSet这种返回boolean类型的函数，因为其返回值表现的就是成功与否，所以不需要重试。

```java
	public final boolean compareAndSet(int expectedValue, int newValue);
```

CAS是Java并发中所谓lock-free机制的基础。

## 23 请介绍类加载过程，什么是双亲委派模型？

​	一般来说，我们把Java的类加载过程分为三个主要步骤：加载、链接、初始化，具体行为在Java虚拟机规范里有非常详细的定义。
​	首先是加载阶段（Loading），它是Java将字节码数据从不同的数据源读取到JVM中，并映射为JVM认可的数据结构（Class对象），这里的数据源可能是各种各样的形态，如jar文件、class文件，甚至是网络数据源等；如果输入数据不是ClassFile的结构，则会抛出ClassFormatError。加载阶段是用户参与的阶段，我们可以自定义类加载器，去实现自己的类加载过程。
​	第二阶段是链接（Linking），这是核心的步骤，简单说是把原始的类定义信息平滑地转化入JVM运行的过程中。这里可进一步细分为三个步骤：

* 验证（Verifcation），这是虚拟机安全的重要保障，JVM需要核验字节信息是符合Java虚拟机规范的，否则就被认为是VerifyError，这样就防止了恶意信息或者不合规的信息危害JVM的运行，验证阶段有可能触发更多class的加载。
* 准备（Preparation），创建类或接口中的静态变量，并初始化静态变量的初始值。但这里的“初始化”和下面的显式初始化阶段是有区别的，侧重点在于分配所需要的内存空间，不会去执行更进一步的JVM指令。
* 解析（Resolution），在这一步会将常量池中的符号引用（symbolic reference）替换为直接引用。在Java虚拟机规范中，详细介绍了类、接口、方法和字段等各个方面的解析。

​	最后是初始化阶段（initialization），这一步真正去执行类初始化的代码逻辑，包括静态字段赋值的动作，以及执行类定义中的静态初始化块内的逻辑，编译器在编译阶段就会把这部分逻辑整理好，父类型的初始化逻辑优先于当前类型的逻辑。

​	再来谈谈双亲委派模型，简单说就是当类加载器（Class-Loader）试图加载某个类型的时候，除非父加载器找不到相应类型，否则尽量将这个任务代理给当前加载器的父加载器去 做。使用委派模型的目的是避免重复加载Java类型。

## 24 有哪些方法可以在运行时动态生成一个Java类？

​	我们可以从常见的Java类来源分析，通常的开发过程是，开发者编写Java代码，调用javac编译成class文件，然后通过类加载机制载入JVM，就成为应用运行时可以使用的Java类了。
​	从上面过程得到启发，其中一个直接的方式是从源码入手，可以利用Java程序生成一段源码，然后保存到文件等，下面就只需要解决编译问题了。
​	有一种笨办法，直接用ProcessBuilder之类启动javac进程，并指定上面生成的文件作为输入，进行编译。最后，再利用类加载器，在运行时加载即可。
​	前面的方法，本质上还是在当前程序进程之外编译的，那么还有没有不这么low的办法呢？
​	你可以考虑使用Java Compiler API，这是JDK提供的标准API，里面提供了与javac对等的编译器功能，具体请参考java.compiler相关文档。
​	进一步思考，我们一直围绕Java源码编译成为JVM可以理解的字节码，换句话说，只要是符合JVM规范的字节码，不管它是如何生成的，是不是都可以被JVM加载呢？我们能不能直接生成相应的字节码，然后交给类加载器去加载呢？
​	当然也可以，不过直接去写字节码难度太大，通常我们可以利用Java字节码操纵工具和类库来实现，比如在专栏第6讲中提到的ASM、Javassist、cglib等。

## 25 谈谈JVM内存区域的划分，哪些区域可能发生OutOfMemoryError?

通常可以把JVM内存区域分为下面几个方面，其中，有的区域是以线程为单位，而有的区域则是整个JVM进程唯一的。
首先，程序计数器（PC，Program Counter Register）。在JVM规范中，每个线程都有它自己的程序计数器，并且任何时间一个线程都只有一个方法在执行，也就是所谓的当前方法。程序计数器会存储当前线程正在执行的Java方法的JVM指令地址；或者，如果是在执行本地方法，则是未指定值（undefned）。
第二，Java虚拟机栈（Java Virtual Machine Stack），早期也叫Java栈。每个线程在创建时都会创建一个虚拟机栈，其内部保存一个个的栈帧（Stack Frame），对应着一次次的Java方法调用。
前面谈程序计数器时，提到了当前方法；同理，在一个时间点，对应的只会有一个活动的栈帧，通常叫作当前帧，方法所在的类叫作当前类。如果在该方法中调用了其他方法，对应的新的栈帧会被创建出来，成为新的当前帧，一直到它返回结果或者执行结束。JVM直接对Java栈的操作只有两个，就是对栈帧的压栈和出栈。
栈帧中存储着局部变量表、操作数（operand）栈、动态链接、方法正常退出或者异常退出的定义等。
第三，堆（Heap），它是Java内存管理的核心区域，用来放置Java对象实例，几乎所有创建的Java对象实例都是被直接分配在堆上。堆被所有的线程共享，在虚拟机启动时，我们指定的“Xmx”之类参数就是用来指定最大堆空间等指标。
理所当然，堆也是垃圾收集器重点照顾的区域，所以堆内空间还会被不同的垃圾收集器进行进一步的细分，最有名的就是新生代、老年代的划分。
第四，方法区（Method Area）。这也是所有线程共享的一块内存区域，用于存储所谓的元（Meta）数据，例如类结构信息，以及对应的运行时常量池、字段、方法代码等。
由于早期的Hotspot JVM实现，很多人习惯于将方法区称为永久代（Permanent Generation）。Oracle JDK 8中将永久代移除，同时增加了元数据区（Metaspace）。
第五，运行时常量池（Run-Time Constant Pool），这是方法区的一部分。如果仔细分析过反编译的类文件结构，你能看到版本号、字段、方法、超类、接口等各种信息，还有一项信息就是常量池。Java的常量池可以存放各种常量信息，不管是编译期生成的各种字面量，还是需要在运行时决定的符号引用，所以它比一般语言的符号表存储的信息更加宽泛。
第六，本地方法栈（Native Method Stack）。它和Java虚拟机栈是非常相似的，支持对本地方法的调用，也是每个线程都会创建一个。在Oracle Hotspot JVM中，本地方法栈
和Java虚拟机栈是在同一块儿区域，这完全取决于技术实现的决定，并未在规范中强制。

## 26 如何监控和诊断JVM堆内和堆外内存使用？

了解JVM内存的方法有很多，具体能力范围也有区别，简单总结如下：

* 可以使用综合性的图形化工具，如JConsole、VisualVM（注意，从Oracle JDK 9开始，VisualVM已经不再包含在JDK安装包中）等。这些工具具体使用起来相对比较直观，直接连接到Java进程，然后就可以在图形化界面里掌握内存使用情况。

以JConsole为例，其内存页面可以显示常见的堆内存和各种堆外部分使用状态。

* 也可以使用命令行工具进行运行时查询，如jstat和jmap等工具都提供了一些选项，可以查看堆、方法区等使用数据。
* 或者，也可以使用jmap等提供的命令，生成堆转储（Heap Dump）文件，然后利用jhat或Eclipse MAT等堆转储分析工具进行详细分析。
* 如果你使用的是Tomcat、Weblogic等Java EE服务器，这些服务器同样提供了内存管理相关的功能。
* 另外，从某种程度上来说，GC日志等输出，同样包含着丰富的信息。

这里有一个相对特殊的部分，就是是堆外内存中的直接内存，前面的工具基本不适用，可以使用JDK自带的Native Memory Tracking（NMT）特性，它会从JVM本地内存分配的角度进行解读。

## 27 Java常见的垃圾收集器有哪些？

实际上，垃圾收集器（GC，Garbage Collector）是和具体JVM实现紧密相关的，不同厂商（IBM、Oracle），不同版本的JVM，提供的选择也不同。接下来，我来谈谈最主流的Oracle JDK。

* Serial GC，它是最古老的垃圾收集器，“Serial”体现在其收集工作是单线程的，并且在进行垃圾收集过程中，会进入臭名昭著的“Stop-The-World”状态。当然，其单线程设计也意味着精简的GC实现，无需维护复杂的数据结构，初始化也简单，所以一直是Client模式下JVM的默认选项。
  从年代的角度，通常将其老年代实现单独称作Serial Old，它采用了标记-整理（Mark-Compact）算法，区别于新生代的复制算法。
  Serial GC的对应JVM参数是：
  
  ```
    -XX:+UseSerialGC
  ```

* ParNew GC，很明显是个新生代GC实现，它实际是Serial GC的多线程版本，最常见的应用场景是配合老年代的CMS GC工作，下面是对应参数

  ```
  -XX:+UseConcMarkSweepGC -XX:+UseParNewGC
  ```


* CMS（Concurrent Mark Sweep） GC，基于标记-清除（Mark-Sweep）算法，设计目标是尽量减少停顿时间，这一点对于Web等反应时间敏感的应用非常重要，一直到今天，仍然有很多系统使用CMS GC。但是，CMS采用的标记-清除算法，存在着内存碎片化问题，所以难以避免在长时间运行等情况下发生full GC，导致恶劣的停顿。另外，既然强调了并发（Concurrent），CMS会占用更多CPU资源，并和用户线程争抢。

* Parrallel GC，在早期JDK 8等版本中，它是server模式JVM的默认GC选择，也被称作是吞吐量优先的GC。它的算法和Serial GC比较相似，尽管实现要复杂的多，其特点是新生代和老年代GC都是并行进行的，在常见的服务器环境中更加高效。
  开启选项是：
  
  ```
  -XX:+UseParallelGC
  ```
  
  另外，Parallel GC引入了开发者友好的配置项，我们可以直接设置暂停时间或吞吐量等目标，JVM会自动进行适应性调整，例如下面参数：
  
  ```
  -XX:MaxGCPauseMillis=value
  -XX:GCTimeRatio=N // GC时间和用户时间比例 = 1 / (N+1)
  ```
  
* G1 GC这是一种兼顾吞吐量和停顿时间的GC实现，是Oracle JDK 9以后的默认GC选项。G1可以直观的设定停顿时间的目标，相比于CMS GC，G1未必能做到CMS在最好情况下的延时停顿，但是最差情况要好很多。
  G1 GC仍然存在着年代的概念，但是其内存结构并不是简单的条带式划分，而是类似棋盘的一个个region。Region之间是复制算法，但整体上实际可看作是标记-整理（Mark-Compact）算法，可以有效地避免内存碎片，尤其是当Java堆非常大的时候，G1的优势更加明显。
  G1吞吐量和停顿表现都非常不错，并且仍然在不断地完善，与此同时CMS已经在JDK 9中被标记为废弃（deprecated），所以G1 GC值得你深入掌握。

## 28 谈谈你的GC调优思路。

谈到调优，这一定是针对特定场景、特定目的的事情， 对于GC调优来说，首先就需要清楚调优的目标是什么？从性能的角度看，通常关注三个方面，内存占用（footprint）、延时（latency）和吞吐量（throughput），大多数情况下调优会侧重于其中一个或者两个方面的目标，很少有情况可以兼顾三个不同的角度。当然，除了上面通常的三个方面，也可能需要考虑其他GC相关的场景，例如，OOM也可能与不合理的GC相关参数有关；或者，应用启动速度方面的需求，GC也会是个考虑的方面。
基本的调优思路可以总结为：

* 理解应用需求和问题，确定调优目标。假设，我们开发了一个应用服务，但发现偶尔会出现性能抖动，出现较长的服务停顿。评估用户可接受的响应时间和业务量，将目标简化为，希望GC暂停尽量控制在200ms以内，并且保证一定标准的吞吐量。
* 掌握JVM和GC的状态，定位具体的问题，确定真的有GC调优的必要。具体有很多方法，比如，通过jstat等工具查看GC等相关状态，可以开启GC日志，或者是利用操作系统提供的诊断工具等。例如，通过追踪GC日志，就可以查找是不是GC在特定时间发生了长时间的暂停，进而导致了应用响应不及时。
* 这里需要思考，选择的GC类型是否符合我们的应用特征，如果是，具体问题表现在哪里，是Minor GC过长，还是Mixed GC等出现异常停顿情况；如果不是，考虑切换到什么类型，如CMS和G1都是更侧重于低延迟的GC选项。
* 通过分析确定具体调整的参数或者软硬件配置。
* 验证是否达到调优目标，如果达到目标，即可以考虑结束调优；否则，重复完成分析、调整、验证这个过程。

## 29 Java内存模型中的happen-before是什么？

Happen-before关系，是Java内存模型中保证多线程操作可见性的机制，也是对早期语言规范中含糊的可见性概念的一个精确定义。
它的具体表现形式，包括但远不止是我们直觉中的synchronized、volatile、lock操作顺序等方面，例如：

* 线程内执行的每个操作，都保证happen-before后面的操作，这就保证了基本的程序顺序规则，这是开发者在书写程序时的基本约定。
* 对于volatile变量，对它的写操作，保证happen-before在随后对该变量的读取操作。
* 对于一个锁的解锁操作，保证happen-before加锁操作。
* 对象构建完成，保证happen-before于fnalizer的开始动作。
* 甚至是类似线程内部操作的完成，保证happen-before其他Thread.join()的线程等。

这些happen-before关系是存在着传递性的，如果满足a happen-before b和b happen-before c，那么a happen-before c也成立。
前面我一直用happen-before，而不是简单说前后，是因为它不仅仅是对执行时间的保证，也包括对内存读、写操作顺序的保证。仅仅是时钟顺序上的先后，并不能保证线程交互的可见性。

## 30 Java程序运行在Docker等容器环境有哪些新问题？

对于Java来说，Docker毕竟是一个较新的环境，例如，其内存、CPU等资源限制是通过CGroup（Control Group）实现的，早期的JDK版本（8u131之前）并不能识别这些限制，进而会导致一些基础问题：

* 如果未配置合适的JVM堆和元数据区、直接内存等参数，Java就有可能试图使用超过容器限制的内存，最终被容器OOM kill，或者自身发生OOM。
* 错误判断了可获取的CPU资源，例如，Docker限制了CPU的核数，JVM就可能设置不合适的GC并行线程数等。

从应用打包、发布等角度出发，JDK自身就比较大，生成的镜像就更为臃肿，当我们的镜像非常多的时候，镜像的存储等开销就比较明显了。
如果考虑到微服务、Serverless等新的架构和场景，Java自身的大小、内存占用、启动速度，都存在一定局限性，因为Java早期的优化大多是针对长时间运行的大型服务器端应
用。

## 31 你了解Java应用开发中的注入攻击吗？

注入式（Inject）攻击是一类非常常见的攻击方式，其基本特征是程序允许攻击者将不可信的动态内容注入到程序中，并将其执行，这就可能完全改变最初预计的执行过程，产生恶意效果。
下面是几种主要的注入式攻击途径，原则上提供动态执行能力的语言特性，都需要提防发生注入攻击的可能。
首先，就是最常见的SQL注入攻击。一个典型的场景就是Web系统的用户登录功能，根据用户输入的用户名和密码，我们需要去后端数据库核实信息。
假设应用逻辑是，后端程序利用界面输入动态生成类似下面的SQL，然后让JDBC执行。

```mysql
Select * from use_info where username = “input_usr_name” and password = “input_pwd”
```

但是，如果我输入的input_pwd是类似这样的文本`“ or “”=”`。
那么，拼接出的SQL字符串就变成了下面的条件，OR的存在导致输入什么名字都是复合条件的。

```mysql
Select * from use_info where username = “input_usr_name” and password = “” or “” = “”
```

这里只是举个简单的例子，它是利用了期望输入和可能输入之间的偏差。上面例子中，期望用户输入一个数值，但实际输入的则是SQL语句片段。类似场景可以利用注入的不同SQL语句，进行各种不同目的的攻击，甚至还可以加上“;delete xxx”之类语句，如果数据库权限控制不合理，攻击效果就可能是灾难性的。
第二，操作系统命令注入。Java语言提供了类似Runtime.exec(…)的API，可以用来执行特定命令，假设我们构建了一个应用，以输入文本作为参数，执行下面的命令：

```
ls –la input_fle_name
```

但是如果用户输入是 “input_fle_name;rm –rf /*”，这就有可能出现问题了。当然，这只是个举例，Java标准类库本身进行了非常多的改进，所以类似这种编程错误，未必可以真的完成攻击，但其反映的一类场景是真实存在的。
第三，XML注入攻击。Java核心类库提供了全面的XML处理、转换等各种API，而XML自身是可以包含动态内容的，例如XPATH，如果使用不当，可能导致访问恶意内容。
还有类似LDAP等允许动态内容的协议，都是可能利用特定命令，构造注入式攻击的，包括XSS（Cross-site Scripting）攻击，虽然并不和Java直接相关，但也可能在JSP等动态页面中发生。

## 32 如何写出安全的Java代码？

这个问题可能有点宽泛，我们可以用特定类型的安全风险为例，如拒绝服务（DoS）攻击，分析Java开发者需要重点考虑的点。
DoS是一种常见的网络攻击，有人也称其为“洪水攻击”。最常见的表现是，利用大量机器发送请求，将目标网站的带宽或者其他资源耗尽，导致其无法响应正常用户的请求。
我认为，从Java语言的角度，更加需要重视的是程序级别的攻击，也就是利用Java、JVM或应用程序的瑕疵，进行低成本的DoS攻击，这也是想要写出安全的Java代码所必须考虑的。例如：

* 如果使用的是早期的JDK和Applet等技术，攻击者构建合法但恶劣的程序就相对容易，例如，将其线程优先级设置为最高，做一些看起来无害但空耗资源的事情。幸运的是类似技术已经逐步退出历史舞台，在JDK 9以后，相关模块就已经被移除。
* 上一讲中提到的哈希碰撞攻击，就是个典型的例子，对方可以轻易消耗系统有限的CPU和线程资源。从这个角度思考，类似加密、解密、图形处理等计算密集型任务，都要防范被恶意滥用，以免攻击者通过直接调用或者间接触发方式，消耗系统资源。
* 利用Java构建类似上传文件或者其他接受输入的服务，需要对消耗系统内存或存储的上限有所控制，因为我们不能将系统安全依赖于用户的合理使用。其中特别注意的是涉及解压缩功能时，就需要防范Zip bomb等特定攻击。
* 另外，Java程序中需要明确释放的资源有很多种，比如文件描述符、数据库连接，甚至是再入锁，任何情况下都应该保证资源释放成功，否则即使平时能够正常运行，也可能被攻击者利用而耗尽某类资源，这也算是可能的DoS攻击来源。

所以可以看出，实现安全的Java代码，需要从功能设计到实现细节，都充分考虑可能的安全影响。

## 33 后台服务出现明显“变慢”，谈谈你的诊断思路？

首先，需要对这个问题进行更加清晰的定义:

* 服务是突然变慢还是长时间运行后观察到变慢？类似问题是否重复出现？
* “慢”的定义是什么，我能够理解是系统对其他方面的请求的反应延时变长吗?

第二，理清问题的症状，这更便于定位具体的原因，有以下一些思路：

* 问题可能来自于Java服务自身，也可能仅仅是受系统里其他服务的影响。初始判断可以先确认是否出现了意外的程序错误，例如检查应用本身的错误日志。
* 对于分布式系统，很多公司都会实现更加系统的日志、性能等监控系统。一些Java诊断工具也可以用于这个诊断，例如通过JFR（Java Flight Recorder），监控应用是否大量出现了某种类型的异常。如果有，那么异常可能就是个突破点。如果没有，可以先检查系统级别的资源等情况，监控CPU、内存等资源是否被其他进程大量占用，并且这种占用是否不符合系统正常运行状况。
* 监控Java服务自身，例如GC日志里面是否观察到Full GC等恶劣情况出现，或者是否Minor GC在变长等；利用jstat等工具，获取内存使用的统计信息也是个常用手段；利用jstack等工具检查是否出现死锁等。
* 如果还不能确定具体问题，对应用进行Profling也是个办法，但因为它会对系统产生侵入性，如果不是非常必要，大多数情况下并不建议在生产系统进行。
* 定位了程序错误或者JVM配置的问题后，就可以采取相应的补救措施，然后验证是否解决，否则还需要重复上面部分过程。

## 34 有人说“Lambda能让Java程序慢30倍”，你怎么看？

我认为，“Lambda能让Java程序慢30倍”这个争论实际反映了几个方面：
第一，基准测试是一个非常有效的通用手段，让我们以直观、量化的方式，判断程序在特定条件下的性能表现。
第二，基准测试必须明确定义自身的范围和目标，否则很有可能产生误导的结果。前面代码片段本身的逻辑就有瑕疵，更多的开销是源于自动装箱、拆箱（auto-boxing/unboxing），而不是源自Lambda和Stream，所以得出的初始结论是没有说服力的。
第三，虽然Lambda/Stream为Java提供了强大的函数式编程能力，但是也需要正视其局限性：

* 一般来说，我们可以认为Lambda/Stream提供了与传统方式接近对等的性能，但是如果对于性能非常敏感，就不能完全忽视它在特定场景的性能差异了，例如初始化的开销。Lambda并不算是语法糖，而是一种新的工作机制，在首次调用时，JVM需要为其构建CallSite实例。这意味着，如果Java应用启动过程引入了很多Lambda语句，会导致启动过程变慢。其实现特点决定了JVM对它的优化可能与传统方式存在差异。
* 增加了程序诊断等方面的复杂性，程序栈要复杂很多，Fluent风格本身也不算是对于调试非常友好的结构，并且在可检查异常的处理方面也存在着局限性等。

## 35 JVM优化Java代码时都做了什么？

​	JVM在对代码执行的优化可分为运行时（runtime）优化和即时编译器（JIT）优化。运行时优化主要是解释执行和动态编译通用的一些机制，比如说锁机制（如偏斜锁）、内存分配机制（如TLAB）等。除此之外，还有一些专门用于优化解释执行效率的，比如说模版解释器、内联缓存（inline cache，用于优化虚方法调用的动态绑定）。
​	JVM的即时编译器优化是指将热点代码以方法为单位转换成机器码，直接运行在底层硬件之上。它采用了多种优化方式，包括静态编译器可以使用的如方法内联、逃逸分析，也包括基于程序运行profle的投机性优化（speculative/optimistic optimization）。这个怎么理解呢？比如我有一条instanceof指令，在编译之前的执行过程中，测试对象的类一直是同一个，那么即时编译器可以假设编译之后的执行过程中还会是这一个类，并且根据这个类直接返回instanceof的结果。如果出现了其他类，那么就抛弃这段编译后的机器码，并且切换回解释执行。
​	当然，JVM的优化方式仅仅作用在运行应用代码的时候。如果应用代码本身阻塞了，比如说并发时等待另一线程的结果，这就不在JVM的优化范畴啦。

## 36 谈谈MySQL支持的事务隔离级别，以及悲观锁和乐观锁的原理和应用场景？

​	所谓隔离级别（Isolation Level），就是在数据库事务中，为保证并发数据读写的正确性而提出的定义，它并不是MySQL专有的概念，而是源于ANSI/ISO制定的SQL-92标准。
​	每种关系型数据库都提供了各自特色的隔离级别实现，虽然在通常的定义中是以锁为实现单元，但实际的实现千差万别。以最常见的MySQL InnoDB引擎为例，它是基于MVCC（Multi-Versioning Concurrency Control）和锁的复合实现，按照隔离程度从低到高，MySQL事务隔离级别分为四个不同层次：

* 读未提交（Read uncommitted），就是一个事务能够看到其他事务尚未提交的修改，这是最低的隔离水平，允许脏读出现。
* 读已提交（Read committed），事务能够看到的数据都是其他事务已经提交的修改，也就是保证不会看到任何中间性状态，当然脏读也不会出现。读已提交仍然是比较低级别的隔离，并不保证再次读取时能够获取同样的数据，也就是允许其他事务并发修改数据，允许不可重复读和幻象读（Phantom Read）出现。
* 可重复读（Repeatable reads），保证同一个事务中多次读取的数据是一致的，这是MySQL InnoDB引擎的默认隔离级别，但是和一些其他数据库实现不同的是，可以简单认为MySQL在可重复读级别不会出现幻象读。
* 串行化（Serializable），并发事务之间是串行化的，通常意味着读取需要获取共享读锁，更新需要获取排他写锁，如果SQL使用WHERE语句，还会获取区间锁（MySQL以GAP锁形式实现，可重复读级别中默认也会使用），这是最高的隔离级别。

至于悲观锁和乐观锁，也并不是MySQL或者数据库中独有的概念，而是并发编程的基本概念。主要区别在于，操作共享数据时，“悲观锁”即认为数据出现冲突的可能性更大，而“乐观锁”则是认为大部分情况不会出现冲突，进而决定是否采取排他性措施。
反映到MySQL数据库应用开发中，悲观锁一般就是利用类似SELECT … FOR UPDATE这样的语句，对数据加锁，避免其他事务意外修改数据。乐观锁则与Java并发包中的AtomicFieldUpdater类似，也是利用CAS机制，并不会对数据加锁，而是通过对比数据的时间戳或者版本号，来实现乐观锁需要的版本判断。
我认为前面提到的MVCC，其本质就可以看作是种乐观锁机制，而排他性的读写锁、双阶段锁等则是悲观锁的实现。
有关它们的应用场景，你可以构建一下简化的火车余票查询和购票系统。同时查询的人可能很多，虽然具体座位票只能是卖给一个人，但余票可能很多，而且也并不能预测哪个查询者会购票，这个时候就更适合用乐观锁。

## 37 谈谈Spring Bean的生命周期和作用域？

Spring Bean生命周期比较复杂，可以分为创建和销毁两个过程。
首先，创建Bean会经过一系列的步骤，主要包括：

* 实例化Bean对象。
* 设置Bean属性。
* 如果我们通过各种Aware接口声明了依赖关系，则会注入Bean对容器基础设施层面的依赖。具体包括BeanNameAware、BeanFactoryAware和ApplicationContextAware，
* 分别会注入Bean ID、Bean Factory或者ApplicationContext。
* 调用BeanPostProcessor的前置初始化方法postProcessBeforeInitialization。
* 如果实现了InitializingBean接口，则会调用afterPropertiesSet方法。
* 调用Bean自身定义的init方法。
* 调用BeanPostProcessor的后置初始化方法postProcessAfterInitialization。
* 创建过程完毕。

第二，Spring Bean的销毁过程会依次调用DisposableBean的destroy方法和Bean自身定制的destroy方法。
Spring Bean有五个作用域，其中最基础的有下面两种：

* Singleton，这是Spring的默认作用域，也就是为每个IOC容器创建唯一的一个Bean实例。
* Prototype，针对每个getBean请求，容器都会单独创建一个Bean实例。

从Bean的特点来看，Prototype适合有状态的Bean，而Singleton则更适合无状态的情况。另外，使用Prototype作用域需要经过仔细思考，毕竟频繁创建和销毁Bean是有明显开销的。
如果是Web容器，则支持另外三种作用域：

* Request，为每个HTTP请求创建单独的Bean实例。
* Session，很显然Bean实例的作用域是Session范围。
* GlobalSession，用于Portlet容器，因为每个Portlet有单独的Session，GlobalSession提供一个全局性的HTTP Session。

## 38 对比Java标准NIO类库，你知道Netty是如何实现更高性能的吗？

单独从性能角度，Netty在基础的NIO等类库之上进行了很多改进，例如：

* 更加优雅的Reactor模式实现、灵活的线程模型、利用EventLoop等创新性的机制，可以非常高效地管理成百上千的Channel。
* 充分利用了Java的Zero-Copy机制，并且从多种角度，“斤斤计较”般的降低内存分配和回收的开销。例如，使用池化的Direct Bufer等技术，在提高IO性能的同时，减少了对象的创建和销毁；利用反射等技术直接操纵SelectionKey，使用数组而不是Java容器等。
* 使用更多本地代码。例如，直接利用JNI调用Open SSL等方式，获得比Java内建SSL引擎更好的性能。
* 在通信协议、序列化等其他角度的优化。

总的来说，Netty并没有Java核心类库那些强烈的通用性、跨平台等各种负担，针对性能等特定目标以及Linux等特定环境，采取了一些极致的优化手段

## 39 谈谈常用的分布式ID的设计方案？Snowflake是否受冬令时切换影响？

首先，我们需要明确通常的分布式ID定义，基本的要求包括：

* 全局唯一，区别于单点系统的唯一，全局是要求分布式系统内唯一。
* 有序性，通常都需要保证生成的ID是有序递增的。例如，在数据库存储等场景中，有序ID便于确定数据位置，往往更加高效。

目前业界的方案很多，典型方案包括：

* 基于数据库自增序列的实现。这种方式优缺点都非常明显，好处是简单易用，但是在扩展性和可靠性等方面存在局限性。
* 基于Twitter早期开源的Snowfake的实现，以及相关改动方案。这是目前应用相对比较广泛的一种方式，其结构定义你可以参考下面的示意图。

![](https://img-blog.csdnimg.cn/20210118171511867.png)

整体长度通常是64 （1 + 41 + 10+ 12 = 64）位，适合使用Java语言中的long类型来存储。
头部是1位的正负标识位。
紧跟着的高位部分包含41位时间戳，通常使用System.currentTimeMillis()。
后面是10位的WorkerID，标准定义是5位数据中心 + 5位机器ID，组成了机器编号，以区分不同的集群节点。
最后的12位就是单位毫秒内可生成的序列号数目的理论极限。
Snowfake的官方版本是基于Scala语言，Java等其他语言的参考实现有很多，是一种非常简单实用的方式，具体位数的定义是可以根据分布式系统的真实场景进行修改的，并不一定要严格按照示意图中的设计。

* Redis、Zookeeper、MangoDB等中间件，也都有各种唯一ID解决方案。其中一些设计也可以算作是Snowfake方案的变种。例如，MongoDB的ObjectId提供了一个12byte（96位）的ID定义，其中32位用于记录以秒为单位的时间，机器ID则为24位，16位用作进程ID，24位随机起始的计数序列。
* 国内的一些大厂开源了其自身的部分分布式ID实现，InfoQ就曾经介绍过微信的seqsvr，它采取了相对复杂的两层架构，并根据社交应用的数据特点进行了针对性设计，具体请参考相关代码实现。另外，百度、美团等也都有开源或者分享了不同的分布式ID实现，都可以进行参考。

关于第二个问题，Snowfake是否受冬令时切换影响？
我认为没有影响，你可以从Snowfake的具体算法实现寻找答案。我们知道Snowfake算法的Java实现，大都是依赖于System.currentTimeMillis()，这个数值代表什么呢？
从Javadoc可以看出，它是返回当前时间和1970年1月1号UTC时间相差的毫秒数，这个数值与夏/冬令时并没有关系，所以并不受其影响。