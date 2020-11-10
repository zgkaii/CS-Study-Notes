## 1 HashMap简介
HashMap 是一个散列表，它存储的内容是键值对(key-value)映射。它继承于AbstractMap，实现了Map、Cloneable、`java.io.Serializable`接口。

<img src="https://img-blog.csdnimg.cn/20201108151702116.png" style="zoom: 80%;" />

HashMap的实现是不同步的，这意味着**线程不安全**，在并发的环境中建议使用`ConcurrentHashMap`。

HashMap的key、value都可以为null（除了不同步和允许使用 null 之外，HashMap与 Hashtable大致相同）。此外，HashMap中的映射是**无序**的。

HashMap 的实例有两个参数影响其性能：“**初始容量**” 和 “**加载因子**”。容量是哈希表中桶的数量，初始容量只是哈希表在创建时的容量。

加载因子是哈希表在其容量自动增加之前可以达到多满的一种尺度。当哈希表中的条目数超出了加载因子与当前容量的乘积时，则要对

该哈希表进行 rehash 操作（即重建内部数据结构），从而哈希表将具有大约两倍的桶数。

通常，**默认加载因子是0.75**，这是在时间和空间成本上寻求一种折衷。加载因子过高虽然减少了空间开销，但同时也增加了查询成本

（大多数 HashMap 类的操作中，包括 get 和 put 操作，都反映了这一点）。在设置初始容量时应该考虑到映射中所需的条目数及其

加载因子，以便最大限度地减少 rehash 操作次数。如果初始容量大于最大条目数除以加载因子，则不会发生 rehash 操作。

## 2 底层数据结构

### 2.1 JDK1.7

JDK1.7中HashMap是**数组+链表**结合在一起使用的**链表散列**。HashMap 定义了扰动函数`hash()`来减少碰撞，当两个不同元素经过计算发现在数组中的位置一样时，就通过拉链法解决冲突。

所谓 **“拉链法”** 就是，将链表和数组相结合。也就是说创建一个链表数组，数组中每一格就是一个链表。若遇到哈希冲突，则将冲突的值加到链表中即可。

![](https://img-blog.csdnimg.cn/20201109212426593.png)

HashMap 在JDK1.7中详细实现可参考[深入理解HashMap（jdk7）](https://juejin.im/post/6844903903595593741)一文。

### 2.2 JDK1.8

如果位于链表中的结点过多，那么很显然通过key值依次查找效率就太低了。因此JDK1.8在解决哈希冲突时有了较大的变化，**当链表长度大于阈值（默认为8）时，将链表转化为红黑树**，以减少搜索时间。

数组的查询效率为O(1)，链表的查询效率是O(k)，红黑树的查询效率是O(log k)，k为桶中的元素个数，所以当元素数量非常多的时候，转化为红黑树能极大地提高效率。

也就是说，JDK1.8之后，HashMap底层数据结构是**数组+链表+红黑树**。

![](https://img-blog.csdnimg.cn/20201109213850718.png)

**Node节点类源码:**

```java
// 继承自 Map.Entry<K,V>
static class Node<K,V> implements Map.Entry<K,V> {
       final int hash;// 哈希值，存放元素到hashmap中时用来与其他元素hash值比较
       final K key;// 键
       V value;// 值
       // 指向下一个节点
       Node<K,V> next;// 链表的next指针
       Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }
        // 重写hashCode()方法
        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }
        // 重写 equals() 方法
        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
}
```
**TreeNode节点类源码:**

```java
    // 转变为红黑树后的结点类
	static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // 父节点
        TreeNode<K,V> left;    // 左子树
        TreeNode<K,V> right;   // 右子树
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;		   // 判断颜色
        TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
        }
		// 返回当前节点的根节点
        final TreeNode<K,V> root() {
            for (TreeNode<K,V> r = this, p;;) {
                if ((p = r.parent) == null)
                    return r;
                r = p;
            }
        }    
        --- omit ---
    }
```
## 3 HashMap源码分析（JDK 1.8）

### 3.1 静态属性

```java
    // 序列号
    private static final long serialVersionUID = 362498820763181265L;    
    // 默认的初始容量是2的4次方，即16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;   
    // 最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30; 
    // 默认的加载因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 树化阈值。当上某个桶(bucket)的结点数大于这个值时会转成红黑树
    static final int TREEIFY_THRESHOLD = 8; 
    // 链表还原阈值。当某个桶(bucket)上的结点数小于这个值时树转链表
    static final int UNTREEIFY_THRESHOLD = 6;
    // 桶中结构转化为红黑树对应的table的最小大小
    static final int MIN_TREEIFY_CAPACITY = 64;
```

### 3.2 成员属性

```java
    // 哈希桶数组
    transient Node<k,v>[] table; 
    // 保存缓存的entrySet
    transient Set<map.entry<k,v>> entrySet;
    // 桶的实际元素个数 != table.length。
    transient int size;
    // 每次扩容和更改map结构的计数器都会+1
    transient int modCount;   
    // 临界值，当实际大小(容量*加载因子)超过临界值时，会进行扩容
    int threshold;
    // 加载因子
    final float loadFactor;
```

- **loadFactor加载因子**

  loadFactor加载因子是控制数组存放数据的疏密程度，loadFactor越趋近于1，那么   数组中存放的数据(entry)也就越多，也就越密，也就是会让链表的长度增加，loadFactor越小，也就是趋近于0，数组中存放的数据(entry)也就越少，也就越稀疏。

  **loadFactor太大导致查找元素效率低，太小导致数组的利用率低，存放的数据会很分散。loadFactor的默认值为0.75f是官方给出的一个比较好的临界值**。 

  给定的默认容量为 16，加载因子为 0.75。Map 在使用过程中不断的往里面存放数据，当数量达到了 16 * 0.75 = 12 就需要将当前 16 的容量进行扩容，而扩容这个过程涉及到 rehash、复制数据等操作，所以非常消耗性能。

- **threshold**

  **threshold = capacity * loadFactor**，**当Size>=threshold**的时候，那么就要考虑对数组的扩增了，也就是说，这个的意思就是**衡量数组是否需要扩增的一个标准**。

### 3.3 构造方法

HashMap 中有四个构造方法，它们分别如下：

```java
    // 默认构造函数。
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
     }
     
     // 调用putMapEntries将Map中的值加入新的Map集合中
     public HashMap(Map<? extends K, ? extends V> m) {
         this.loadFactor = DEFAULT_LOAD_FACTOR;
         putMapEntries(m, false);// 在构造函数使用的时候是false其他时候是true
     }
     
     // 指定“容量大小”的构造函数
     public HashMap(int initialCapacity) {
         this(initialCapacity, DEFAULT_LOAD_FACTOR);
     }
     
     // 指定“容量大小”和“加载因子”的构造函数
     public HashMap(int initialCapacity, float loadFactor) {
         if (initialCapacity < 0)
             throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
         if (initialCapacity > MAXIMUM_CAPACITY)
             initialCapacity = MAXIMUM_CAPACITY;
         if (loadFactor <= 0 || Float.isNaN(loadFactor))
             throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
         this.loadFactor = loadFactor;
         this.threshold = tableSizeFor(initialCapacity);// 临界值为大于initialCapacity的最小二次幂。
     }
```

可以发现，不论是哪个构造构造方法，loadFactor一定会被初始化的。

第二个构造方法是将传入的Map集合元素添加到Map实例中，源码如下：

```java
// 传递的Map集合中的所有元素加入本Map实例中
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    // 传入Map元素个数
    int s = m.size();
    if (s > 0) {
        // 判断table是否已经初始化
        if (table == null) {
            // ft = m.size() / 0.75 + 1，因为会计算出小数因此+1.0F向上取整
            float ft = ((float)s / loadFactor) + 1.0F;
            // 判断ft是否小于最大容量
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                    (int)ft : MAXIMUM_CAPACITY);
            // 计算得到的t大于临界值，则重新计算阈值
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        // 已初始化，并且m中元素个数大于临界值，进行扩容处理
        else if (s > threshold)
            resize();
        // 遍历，将m中的所有元素添加至HashMap中
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```
### 3.4 tableSizeFor(int cap)方法

tableSizeFor(int cap) 静态方法是用来**计算合理的初始容量，返回大于等于参数 cap的最小 2 次幂**。

```java
// 返回大于cap的最小的二次幂数值
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    //  >>>，无符号右移，忽略符号位，空位都以0补齐。对于正数移位来说等同于：>>。
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    // n<0返回1
    // n>最大容量，返回最大容量MAXIMUM_CAPACITY
    // 否则返回n+1
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

**这里为什么` cap - 1`呢？**先来分析有关n位操作部分

先来假设n的二进制为01xxx...xxx。

对n右移1位：001xx...xxx，再位或：011xx...xxx（1 与 `0 或1 `位或结果都为 1）

对n右移2为：00011...xxx，再位或：01111...xxx

... ...

同理，如果有8个1，右移8位肯定会让后八位也为1。综上可得，该算法让最高位的1后面的位全变为1。

最后再让结果n+1，即得到了2的整数次幂的值了。

回过头再来看`int n = cap - 1;`，就明白用`cap-1`赋值给n是**为了找出大于或等于原值的目标值**。

例如，二进制数0100，即4。如果不进行减1操作，通过运算会等到1000，即8，直接翻倍了，不符合结果。进行减1操作后，二进制数为0011，再进行运算会得到0100，即4。

### 3.4 确定数组索引位置

不管增加、删除、查找键值对，定位到哈希桶数组的位置都是关键的第一步。

HashMap先是通过**扰动函数**`hash(Object key)`处理key的hashCode而得到其hash 值，然后通过 `(n - 1) & hash` 判断当前元素存放的位置（这里的 n 指的是数组的长度）。

#### 3.4.1 hash(Object key) 

该方法是 HashMap 的核心静态方法，用于**计算 key 的 hash 值**。我们在使用HashMap存放数据时，当然希望元素位置分布均匀一点。而使用hash算法计算位置的时，可以直接确定位置，不用遍历链表，大大优化了查询的效率。

**JDK1.7**的 HashMap 的 hash 方法源码：

```java
static int hash(int h) {

    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

JDK 1.8 的 hash方法 相比于 JDK 1.7 hash 方法更加简化，但是**原理不变**。JDK 1.7 的 hash 方法的性能会稍差一点点，毕竟扰动了 4 次。

**JDK1.8**中 HashMap 的 hash 方法源码：

  ```java
    // 通过位运算减少 Hash 冲突  
	static final int hash(Object key) {
        int h;
        // key.hashCode()  传入key的散列值
		// 按位异或^，如果相对应位值相同，则结果为0，否则为1        	
        // h ^ (h >>> 16)  高位参与运算
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
  ```

key.hashCode()函数调用的是key键值类型自带的哈希函数`public native int hashCode();`，返回的hashcode是int类型。如果直接拿散列值做数组下标，那么索引范围就是‑2147483648到2147483648，前后加起来大概40亿的映射空间，很难出现碰撞。问题是默认的初始容量最大才16，这就说明散列值是不能直接用来访问。

在**JDK1.7**中，散列值使用前需要**对数组长度取模运算**，到的余数才能用来访问数组下标。源码中模运算是在这个indexFor( )函数里完成。

```java
bucketIndex = indexFor(hash, table.length);

static int indexFor(int h, int length) {
    // 散列值和数组长度做一个“与”操作
    return h & (length-1);
}
```

位与运算符`&`运算规则是只有**二进制位同时为 1，那么计算结果才为 1，否则为 0**。所有说，“与”操作的结果就是将散列值的高位全部归0，只保留低位值用来做数组下标访问。

以初始长度8为例，8-1=7。2进制表示为00000000 00000000 00000111。与某散列值做“与”操作，结果就是截取了最低4位。

```java
		10100101 11000100 00100101
&		00000000 00000000 00000111
----------------------------------	
		00000000 00000000 00000101
```

在**JDK1.8**中，**putVal()** 中寻址部分如下，原理跟上面一样。

```java
tab[i = (n - 1) & hash]
```

问题来了，就算散列值再松散，总是截取最后几位，碰撞还是会很严重。比如初始长度为16，16-1=15。与某散列值做“与”操作结果跟之前是一样的。

```java
		11110101 10000101 00110100
&		00000000 00000000 00001111
----------------------------------	
		00000000 00000000 00000101
```

这时候，”扰动函数“的价值就体现出来了。

![](https://img-blog.csdnimg.cn/img_convert/e65b38bc4f21af766926790f1b34387b.png)

右位移16位，正好是32bit的一半，自己的高半区与低半区做异或，就是**为了混合原始哈希码的高位与低位，以此来加大低位随机性**。而混合后的低位参杂了高位部分特征，高位信息也参入了寻址计算（进行扰动）。

#### 3.4.2 桶下标计算公式

由之前的分析可知，桶下标计算公式即是`(capacity - 1) & hash` 。之所以不使用`hash % capacity`方式取模，是因为HashMap中规定了哈希表长度为2 的幂，在这种情况下，位与运算更快， resize() 扩容时可以更高效的重新计算桶下标。

### 3.5 put方法

put方法中调用了putVal方法，传递的参数是调用了hash()方法计算key的hash值，主要逻辑写在putVal中。

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 步骤1：桶数组未初始化或者长度为0，进行扩容
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 步骤2：指定桶为空，新生成结点直接放入
    // (n - 1) & hash 计算元素在哪个桶中
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 指定位置已经存在元素
    else {
        Node<K,V> e; K k;
        // 步骤3： 如果桶中第一个元素的key与待插入元素的key相同，保存到e中用于后续修改value值
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
                // 将第一个元素赋值给e，用e来记录
                e = p;
        // 步骤4：如果第一个元素是树节点，则调用树节点的putTreeVal插入元素
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 步骤5：为链表结点，遍历链表进行操作
        else {
            // inCount用于存储链表中元素的个数
            for (int binCount = 0; ; ++binCount) {
                // 到达链表的尾部还没有找到相同key的元素，则在尾部插入新结点
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // 结点数量达到临界值，转化为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) 
                        treeifyBin(tab, hash);
                    break;
                }
                // 判断链表中结点的key值与插入的元素的key值相等，则退出循环
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                // 用于遍历桶中的链表，与前面的e = p.next组合，可以遍历链表
                p = e;
            }
        }
        // 步骤6：表示在桶中找到key值、hash值与插入元素相等的结点
        if (e != null) { 
            // 记录e的旧value
            V oldValue = e.value;
            // 判断是否要置换旧值
            if (!onlyIfAbsent || oldValue == null)
                // 直接覆盖，并返回旧值
                e.value = value;
            // 访问后回调
            afterNodeAccess(e);
            // 返回旧值
            return oldValue;
        }
    }

    ++modCount;
    // 步骤7：元素加1后，判断是否扩容
    if (++size > threshold)
        resize();
    // 插入后回调
    afterNodeInsertion(evict);
    return null;
} 
```

**对putVal方法添加元素的分析如下：**

* 如果桶容量为0，则初始化桶。

- 如果定位到的桶位置没有元素就直接插入元素。
- 如果定位到的桶位置有元素就和要插入的元素key比较，如果key相同就直接覆盖。
- 如果定位到的桶位置有元素且与插入的key不相同，就判断是否是一个树节点，如果是就调用`e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value)`将元素添加进入。如果不是就遍历链表插入(插入的是链表尾部)。
- 插入了元素，则桶的容量加1，并判断是否需要扩容

### 3.6 resize方法

进行扩容，会伴随着一次重新hash分配，并且会遍历hash表中所有的元素，是非常耗时的。在编写程序中，要尽量避免resize。
```java
final Node<K,V>[] resize() {
    // 旧数组
    Node<K,V>[] oldTab = table;
    // 旧容量
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    // 旧临界值
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        // 如果旧容量达到了最大容量，则不再进行扩容
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 没超过最大值，就扩充为原来的2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        // 使用非默认构造方法创建的map，第一次插入元素会走到这里
        // 如果旧容量为0且旧扩容门槛大于0，则把新容量赋值为旧门槛
        newCap = oldThr;
    else { 
        // 调用默认构造方法创建的map，第一次插入元素会走到这里
        // 如果旧容量旧扩容门槛都是0，说明还未初始化过，则初始化容量为默认容量，扩容门槛为默认容量*默认加载因子
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 如果新扩容门槛为0，则计算为容量*装载因子，但不能超过最大容量
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // 把每个bucket都移动到新的buckets中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { 
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 原索引
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 原索引+oldCap
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 原索引放到bucket里
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 原索引+oldCap放到bucket里
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```
### 3.7 get方法

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 数组元素相等
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 桶中不止一个节点
        if ((e = first.next) != null) {
            // 在树中get
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 在链表中get
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

### 3.8 treeifyBin()

### 3.9 remove(Object key)

### 3.10 TreeNode.putTreeVal(...)

### 3.11 TreeNode.treeify()

### 3.12 TreeNode.getTreeNode(int h, Object k)

### 3.13 TreeNode.removeTreeNode(...)

## 4 遍历方式

遍历分为5种方式：

（1）**调用`map.entrySet()`的集合迭代器**

```java
	Map map = new HashMap();
	Iterator<Map.Entry<?, ?>> iter = map.entrySet().iterator();
	while (iter.hasNext()) {
        Map.Entry<?, ?>> entry = iter.next();
        Object key = entry.getKey();
        Object val = entry.getValue();
	}
```

（2） **调用`map.keySet()`的集合迭代器**

```java
　　Map map = new HashMap();
　　Iterator<?> iter = map.keySet().iterator();
　　while (iter.hasNext()) {
      Object key = iter.next();
      Object val = map.get(key);
　　}
```

（3）**`for each map.entrySet()`**

```java
    Map<?, ?> map = new HashMap<?, ?>();
    for (Entry<?, ?> entry : map.entrySet()) {
        entry.getKey();
        entry.getValue();
    }
```

（4） **`for each map.entrySet()`，临时变量保存`map.entrySet()`**

```java
	Map<?, ?> map = new HashMap<?, ?>();    
	Set<Entry<?, ?> entrySet = map.entrySet();
    for (Entry<?, ?> entry : entrySet) {
        entry.getKey();
        entry.getValue();
    }
```

（5）**`for each map.keySet()`**

```java
    Map<?, ?> map = new HashMap<?, ?>();
    for (Object key : map.keySet()) {
        map.get(key);
    }
```

**推荐使用第一种方式。**



## 参考

[HashMap源码分析](https://juejin.im/post/6844904048185851911)

[Java 集合系列10之 HashMap详细介绍(源码解析)和使用示例](https://www.cnblogs.com/skywang12345/p/3310835.html)

[HashMap(JDK1.8)源码+底层数据结构分析](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/collection/HashMap(JDK1.8)%E6%BA%90%E7%A0%81%2B%E5%BA%95%E5%B1%82%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%88%86%E6%9E%90.md)

[深入理解HashMap（jdk7）](https://juejin.im/post/6844903903595593741)

[全网把Map中的hash()分析的最透彻的文章，别无二家](https://www.hollischuang.com/archives/2091)

[JDK1.8中HashMap的hash算法和寻址算法](https://www.cnblogs.com/eycuii/p/12015283.html)

[HashMap 的 7 种遍历方式与性能分析！](https://juejin.im/post/6844904144331866119)

[Java中的移位运算符](https://zhuanlan.zhihu.com/p/30108890)

[JDK 源码中 HashMap 的 hash 方法原理是什么？](https://www.zhihu.com/question/20733617)

[死磕 java集合之HashMap源码分析](https://juejin.im/post/6844903817855631373)