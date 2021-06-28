## 01 对比Vector、ArrayList、LinkedList有何区别？

这三者都实现集合框架中的List，也就是所谓的有序集合，因此具体功能也比较接近，比如都是按照位置进行定位、添加和删除操作，都提供迭代器以遍历其内容等。但是具体的设计区别，在行为、性能、线程安全等方面，表现又大为不同。

Vector是Java早期提供的**线程安全的动态数组**，如果不需要线程安全，并不建议选择，毕竟同步是额外的开销。Vector内部是使用对象数组来报存数据，可以根据需要自动的增加容量，当数组已满时，会创建新的数组，并拷贝原有数组数据。

ArrayList是应用更加广泛的**动态数组**的实现，它本身不是线程安全的，但性能就好很多。与Vector类似，ArrayList 也是可以根据需要调整容量，不过两者的调整逻辑有所区别，Vector 在扩容时会提高 1 倍，而 ArrayList 则是增加 50%。

LinkedList 顾名思义是 Java 提供的**双向链表**，所以它不需要像上面两种那样调整容量，它也不是线程安全的。



## 02 对比Hashtable、HashMap、TreeMap有什么不同？

Hashtable、HashMap、TreeMap 都是最常见的一些 Map 实现，是以**键值对**的形式存储和操作数据的容器类型。

Hashtable 是早期 Java 类库提供的一个[哈希表](https://zh.wikipedia.org/wiki/哈希表)实现，本身是同步的，不支持 null 键和值，由于同步导致的性能开销，所以已经很少被推荐使用。

HashMap 是应用更加广泛的哈希表实现，行为上大致上与 Hashtable一致，主要区别在于 HashMap 不是同步的，支持 null 键和值等。通常情况下，HashMap 进行 put 或者 get 操作，可以达到常数时间的性能，所以**它是绝大部分利用键值对存取场景的首选**，比如，实现一个用户 ID 和用户信息对应的运行时存储结构。

TreeMap 则是基于**红黑树**的一种提供顺序访问的 Map，和 HashMap 不同，它的 get、put、remove 之类操作都是 O(log(n))的时间复杂度，具体顺序可以由指定的 Comparator 来决定，或者根据键的自然顺序来判断。



## 03 如何保证集合是线程安全的? ConcurrentHashMap如何实现高效地线程安全?

Java 提供了不同层面的线程安全支持。在传统集合框架内部，除了 Hashtable 等同步容器，还提供了所谓的同步包装器（Synchronized Wrapper），我们可以调用 Collections 工具类提供的包装方法，来获取一个同步的包装容器（如 Collections.synchronizedMap），但是它们都是利用非常粗粒度的同步方式，在高并发情况下，性能比较低下。

另外，更加普遍的选择是利用并发包提供的线程安全容器类，它提供了：

- 各种并发容器，比如 ConcurrentHashMap、CopyOnWriteArrayList。
- 各种线程安全队列（Queue/Deque），如 ArrayBlockingQueue、SynchronousQueue。
- 各种有序容器的线程安全版本等。

具体保证线程安全的方式，包括有从简单的 synchronize 方式，到基于更加精细化的，比如基于分离锁实现的 ConcurrentHashMap 等并发实现等。具体选择要看开发的场景需求，总体来说，并发包内提供的容器通用场景，远优于早期的简单同步实现。
