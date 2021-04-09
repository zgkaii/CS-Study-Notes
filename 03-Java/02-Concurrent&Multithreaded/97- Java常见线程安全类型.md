# JDK 基础数据类型与集合类

线性数据结构都源于 Collection 接口，并且拥有迭代器，常用的集合对象有：

* List：ArrayList、LinkedList、Vector、Stack
* Set：LinkedSet、HashSet、TreeSet
* Queue->Deque->LinkedList
* Map：HashMap、LinkedHashMap、TreeMap
* Dictionary->Hashtable->Properties

其中，线程安全的集合对象有：

* Vector、Hashtable

非线程安全的集合对象有：

* ArrayList 、LinkedList、HashMap、HashSet、TreeMap、TreeSet

## ArrayList与LinkedList

**ArrayList**
基本特点：基于数组，便于按 index 访问，超过数组需要扩容，扩容成本较高。

用途：大部分情况下操作一组数据都可以用 ArrayList。

原理：使用数组模拟列表，默认大小10，扩容 x1.5，newCapacity = oldCapacity + (oldCapacity >> 1)。

安全问题：

* 写冲突：两个写，相互操作冲突。
* 读写冲突：读，特别是 iterator 的时候，数据个数变了，拿到了非预期数据或者报错，产生 ConcurrentModificationException。

**LinkedList**

基本特点：使用链表实现，无需扩容。

用途：不知道容量，插入变动多的情况。

原理：使用双向指针将所有节点连起来，还提供了List接口中没有定义的方法，专门用于操作表头和表尾元素，可以当作堆栈、队列和双向队列使用。

安全问题：

* 写冲突：两个写，相互操作冲突。
* 读写冲突：读，特别是 iterator 的时候，数据个数变了，拿到了非预期数据或者报错，产生 ConcurrentModificationException。

### List线程安全的简单办法

既然线程安全是写冲突和读写冲突导致的，最简单办法就是，读写都加锁：

- 1.ArrayList 的方法都加上 synchronized -> Vector 
- 2.Collections.synchronizedList，强制将 List 的操作加上同步 
- 3.Arrays.asList，不允许添加删除，但是可以 set 替换元素 
- 4.Collections.unmodifiableList，不允许修改内容，包括添加删除和 set

## Hashtable、HashMap与HashSet

**HashMap**

基本特点：空间换时间，哈希冲突不大的情况下查找数据性能很高。

用途：存放指定 key 的对象，缓存对象 。

原理：使用 hash 原理，存 k-v 数据，初始容量16，扩容x2，负载因子0.75 JDK8 以后，在链表长度到8 & 数组长度到64时，使用红黑树 安全问题： 写冲突；读写问题，可能会死循环 ；keys()无序问题。

* **LinkedHashMap**

  基本特点：继承自 HashMap，对 Entry 集合添加了一个双向链表 

  用途：保证有序，特别是 Java8 stream 操作的 toMap 时使用 

  原理：同 LinkedList，包括插入顺序和访问顺序 

  安全问题： 同 HashMap

**Hashtable**
是线程安全；无论是key还是value都不允许有null值的存在；在HashTable中调用Put方法时，如果key为null，直接抛出NullPointerException异常；遍历使用的是Enumeration列举。

**HashSet**
基于HashMap实现，无容量限制；是非线程安全的；不保证数据的有序；

- **TreeSet、TreeMap：**
  TreeSet和TreeMap都是完全基于Map来实现的，并且都不支持get(index)来获取指定位置的元素，需要遍历来获取。另外，TreeSet还提供了一些排序方面的支持，例如传入Comparator实现、descendingSet以及descendingIterator等。
  * TreeSet：基于TreeMap实现的，支持排序；是非线程安全的。
  * TreeMap：典型的基于红黑树的Map实现，因此它要求一定要有key比较的方法，要么传入Comparator比较器实现，要么key对象实现Comparator接口；是非线程安全的。

# CopyOnWriteArrayList

# ConcurrentHashMap

# StringBuffer与StringBulider

StringBuilder与StringBuffer都继承自AbstractStringBuilder类，在AbstractStringBuilder中也是使用字符数组保存字符串。

1、在执行速度方面的比较：StringBuilder > StringBuffer ；
2、他们都是字符串变量，是可改变的对象，每当我们用它们对字符串做操作时，实际上是在一个对象上操作的，不像String一样创建一些对象进行操作，所以速度快；
3、 StringBuilder：线程非安全的；
4、StringBuffer：线程安全的；

**对于String、StringBuffer和StringBulider三者使用的总结：**
1.如果要操作少量的数据用 = String
2.单线程操作字符串缓冲区 下操作大量数据 = StringBuilder
3.多线程操作字符串缓冲区 下操作大量数据 = StringBuffer

# 总结

1. 通过`synchronized` 关键字给方法加上内置锁来实现线程安全
   **`Timer`，`TimerTask`，`Vector`，`Stack`，`HashTable`，`StringBuffer`**
2. 原子类`Atomicxxx`—包装类的线程安全类
   如 **`AtomicLong`，`AtomicInteger`等等**
   **`Atomicxxx` 是通过`Unsafe` 类的native方法实现线程安全的**
3. **`BlockingQueue`** 和`BlockingDeque`
   `BlockingDeque`接口继承了`BlockingQueue`接口，
   `BlockingQueue` 接口的实现类有 **`ArrayBlockingQueue`** ， **`LinkedBlockingQueue`** ， **`PriorityBlockingQueue`** 而`BlockingDeque`接口的实现类有`LinkedBlockingDeque`
   **`BlockingQueue`和`BlockingDeque` 都是通过使用定义为final的`ReentrantLock`作为类属性显式加锁实现同步的**
4. **`CopyOnWriteArrayList`** 和 **`CopyOnWriteArraySet`**
   `CopyOnWriteArraySet`的内部实现是在其类内部声明一个final的`CopyOnWriteArrayList`属性，并在调用其构造函数时实例化该`CopyOnWriteArrayList`，`CopyOnWriteArrayList`采用的是显式地加上`ReentrantLock`实现同步，而`CopyOnWriteArrayList`容器的线程安全性在于在每次修改时都会创建并重新发布一个新的容器副本，从而实现可变性。
5. `Concurrentxxx`
   最常用的就是 **`ConcurrentHashMap`** ，当然还有`ConcurrentSkipListSet`和`ConcurrentSkipListMap`等等。
   `ConcurrentHashMap`使用了一种完全不同的加锁策略来提供更高的并发性和伸缩性。`ConcurrentHashMap`并不是将每个方法都在同一个锁上同步并使得每次只能有一个线程访问容器，而是使用一种粒度更细的加锁机制—— **`分段锁`** 来实现更大程度的共享

在这种机制中，任意数量的读取线程可以并发访问Map，执行读取操作的线程和执行写入操作的线程可以并发地访问Map，并且一定数量的写入线程可以并发地修改Map，这使得在并发环境下吞吐量更高，而在单线程环境中只损失非常小的性能

1. **`ThreadPoolExecutor`**
   `ThreadPoolExecutor`也是使用了`ReentrantLock`显式加锁同步
2. `Collections`中的`synchronizedCollection(Collection c)`方法可将一个集合变为线程安全，其内部通过`synchronized`关键字加锁同步

# 参考

* [Java常见的线程安全类](https://blog.csdn.net/tiandao321/article/details/81300489)