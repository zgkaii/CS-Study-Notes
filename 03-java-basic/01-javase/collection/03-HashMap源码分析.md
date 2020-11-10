## 1 HashMap简介
HashMap 是一个散列表，它存储的内容是键值对(key-value)映射。它继承了AbstractMap，实现了Map、Cloneable、`java.io.Serializable`接口。

<img src="https://img-blog.csdnimg.cn/20201108151702116.png" style="zoom: 80%;" />

HashMap的实现是不同步的，这意味着**线程不安全**。并发的环境下建议使用`ConcurrentHashMap`。

HashMap的key、value都可以为null（除了不同步和允许使用 null 之外，HashMap与 Hashtable大致相同）。此外，HashMap中的映射是**无序**的。

HashMap 的实例有两个参数影响其性能：“**初始容量**” 和 “**加载因子**”。容量是哈希表中桶的数量，初始容量是哈希表在创建时的容量。

loadFactor加载因子是控制数组存放数据的疏密程度，loadFactor越趋近于1，那么   数组中存放的数据(entry)也就越多，也就越密，会导致查找元素效率低；loadFactor越小，也就是趋近于0，数组中存放的数据(entry)也就越少，也就越稀疏，但会导致数组的利用率低。

通常，**默认加载因子是0.75**，这是在时间和空间成本上寻求一种折衷。在设置初始容量时应该考虑到映射中所需的条目数及其

加载因子，以便最大限度地减少 rehash 操作次数。如果初始容量大于（最大容量数/加载因子），则不会发生 rehash 操作。

## 2 底层数据结构

### 2.1 JDK1.7

JDK1.7中HashMap是**数组+链表**结合在一起使用的**链表散列**。HashMap 定义了扰动函数`hash()`来减少碰撞，当两个不同元素经过计算发现在数组中的位置一样时，就通过拉链法解决冲突。

所谓 **“拉链法”** 就是，将链表和数组相结合。也就是说创建一个链表数组，数组中每一格就是一个链表。若遇到哈希冲突，则将冲突的值加到链表中即可。

![](https://img-blog.csdnimg.cn/20201109212426593.png)

HashMap 在JDK1.7中详细实现可参考[深入理解HashMap（jdk7）](https://juejin.im/post/6844903903595593741)一文。

### 2.2 JDK1.8

众所周知，数组的查询效率为O(1)，链表的查询效率是O(n)，红黑树的查询效率是O(log n)，n为桶中的元素个数。

所有当位于链表中的结点过多，显然通过key值依次查找效率就太低了。因此JDK1.8在解决哈希冲突时有了较大的变化，**当链表长度大于阈值（默认为8）时，将链表转化为红黑树**，以提高查询效率。

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
    // 桶默认的初始容量，2的4次方，即16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;   
    // 最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30; 
    // 默认的加载因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 树化阈值。当上某个桶(bucket)的结点数大于这个值时会转成红黑树
    static final int TREEIFY_THRESHOLD = 8; 
    // 链表还原阈值。当某个桶(bucket)上的结点数小于这个值时树转链表
    static final int UNTREEIFY_THRESHOLD = 6;
    // 桶中链表结构转化为红黑树对应的table的最小容量
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
    // 每次扩容和更改map结构计数器都会+1
    transient int modCount;   
    // 临界值，当实际大小(容量*加载因子)超过临界值时，会进行扩容。是衡量数组是否需扩容的一个标准。
    int threshold;
    // 加载因子
    final float loadFactor;
```

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

JDK 1.8 的 hash方法 相比于 JDK 1.7 hash 方法更加简化，但是**原理不变**。（JDK 1.7 的 hash 方法的性能会稍差一点点，毕竟扰动了 4 次。）

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

由之前的分析可知，桶下标计算公式即是`(capacity - 1) & hash` 。之所以不使用`hash % capacity`方式取模，是因为HashMap中规定了哈希表长度为2 的幂，在这种情况下，位与运算更快， resize() 扩容时也可以更高效地重新计算桶下标。

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
        // 就容量小于最大容量大于默认初始容量，就扩容为原来的2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // 新临界值等于旧临界值两倍
    }
    else if (oldThr > 0) 
        // 使用 非默认构造方法 创建的map，第一次插入元素会走到这里
        // 如果旧容量为0且旧扩容门槛大于0，则把新容量等于旧临界值
        newCap = oldThr;
    else { 
        // 调用 默认构造方法 创建的map，第一次插入元素会走到这里
        // 如果旧容量旧临界值都是0，说明还未初始化过，则初始化容量为默认容量，临界值=默认容量*默认加载因子
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 如果新临界值为0，则计算为容量*装载因子，但不能超过最大容量
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
    }
    // 将旧临界值设为扩容门槛
    threshold = newThr;
    // 新建一个新容量的数组
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    // 把桶赋值为新数组
    table = newTab;
    if (oldTab != null) {
        // 把每个bucket都移动到新的buckets中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                // 清空旧桶，便于GC回收
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { 
                    // 如果这个链表不止一个元素且不是一颗树
                    // 则分化成两个链表插入到新的桶中去
                    // 比如，假如原来容量为4，3、7、11、15这四个元素都在三号桶中
                    // 现在扩容到8，则3和11还是在三号桶，7和15要搬移到七号桶中去
                    // 也就是分化成了两个链表                  
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

### 3.8 treeifyBin方法

```java
final void treeifyBin(Node<K, V>[] tab, int hash) {
    int n, index;
    Node<K, V> e;
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        // 如果桶数量小于64，直接扩容而不用树化
        // 因为扩容之后，链表会分化成两个链表，达到减少元素的作用
        // 当然也不一定，比如容量为4，里面存的全是除以4余数等于3的元素
        // 这样即使扩容也无法减少链表的长度
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        TreeNode<K, V> hd = null, tl = null;
        // 把所有节点换成树节点
        do {
            TreeNode<K, V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        // 如果进入过上面的循环，则从头节点开始树化
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```

### 3.9 remove(Object key)

```java
public V remove(Object key) {
    Node<K, V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
            null : e.value;
}

final Node<K, V> removeNode(int hash, Object key, Object value,
                            boolean matchValue, boolean movable) {
    Node<K, V>[] tab;
    Node<K, V> p;
    int n, index;
    // 如果桶的数量大于0且待删除的元素所在的桶的第一个元素不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
            (p = tab[index = (n - 1) & hash]) != null) {
        Node<K, V> node = null, e;
        K k;
        V v;
        if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
            // 如果第一个元素正好就是要找的元素，赋值给node变量后续删除使用
            node = p;
        else if ((e = p.next) != null) {
            if (p instanceof TreeNode)
                // 如果第一个元素是树节点，则以树的方式查找节点
                node = ((TreeNode<K, V>) p).getTreeNode(hash, key);
            else {
                // 否则遍历整个链表查找元素
                do {
                    if (e.hash == hash &&
                            ((k = e.key) == key ||
                                    (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        // 如果找到了元素，则看参数是否需要匹配value值，如果不需要匹配直接删除，如果需要匹配则看value值是否与传入的value相等
        if (node != null && (!matchValue || (v = node.value) == value ||
                (value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
                // 如果是树节点，调用树的删除方法（以node调用的，是删除自己）
                ((TreeNode<K, V>) node).removeTreeNode(this, tab, movable);
            else if (node == p)
                // 如果待删除的元素是第一个元素，则把第二个元素移到第一的位置
                tab[index] = node.next;
            else
                // 否则删除node节点
                p.next = node.next;
            ++modCount;
            --size;
            // 删除节点后置处理
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}

```

### 3.10 TreeNode.putTreeVal(...)

```java
final TreeNode<K, V> putTreeVal(HashMap<K, V> map, Node<K, V>[] tab,
                                int h, K k, V v) {
    Class<?> kc = null;
    // 标记是否找到这个key的节点
    boolean searched = false;
    // 找到树的根节点
    TreeNode<K, V> root = (parent != null) ? root() : this;
    // 从树的根节点开始遍历
    for (TreeNode<K, V> p = root; ; ) {
        // dir=direction，标记是在左边还是右边
        // ph=p.hash，当前节点的hash值
        int dir, ph;
        // pk=p.key，当前节点的key值
        K pk;
        if ((ph = p.hash) > h) {
            // 当前hash比目标hash大，说明在左边
            dir = -1;
        }
        else if (ph < h)
            // 当前hash比目标hash小，说明在右边
            dir = 1;
        else if ((pk = p.key) == k || (k != null && k.equals(pk)))
            // 两者hash相同且key相等，说明找到了节点，直接返回该节点
            // 回到putVal()中判断是否需要修改其value值
            return p;
        else if ((kc == null &&
                // 如果k是Comparable的子类则返回其真实的类，否则返回null
                (kc = comparableClassFor(k)) == null) ||
                // 如果k和pk不是同样的类型则返回0，否则返回两者比较的结果
                (dir = compareComparables(kc, k, pk)) == 0) {
            // 这个条件表示两者hash相同但是其中一个不是Comparable类型或者两者类型不同
            // 比如key是Object类型，这时可以传String也可以传Integer，两者hash值可能相同
            // 在红黑树中把同样hash值的元素存储在同一颗子树，这里相当于找到了这颗子树的顶点
            // 从这个顶点分别遍历其左右子树去寻找有没有跟待插入的key相同的元素
            if (!searched) {
                TreeNode<K, V> q, ch;
                searched = true;
                // 遍历左右子树找到了直接返回
                if (((ch = p.left) != null &&
                        (q = ch.find(h, k, kc)) != null) ||
                        ((ch = p.right) != null &&
                                (q = ch.find(h, k, kc)) != null))
                    return q;
            }
            // 如果两者类型相同，再根据它们的内存地址计算hash值进行比较
            dir = tieBreakOrder(k, pk);
        }

        TreeNode<K, V> xp = p;
        if ((p = (dir <= 0) ? p.left : p.right) == null) {
            // 如果最后确实没找到对应key的元素，则新建一个节点
            Node<K, V> xpn = xp.next;
            TreeNode<K, V> x = map.newTreeNode(h, k, v, xpn);
            if (dir <= 0)
                xp.left = x;
            else
                xp.right = x;
            xp.next = x;
            x.parent = x.prev = xp;
            if (xpn != null)
                ((TreeNode<K, V>) xpn).prev = x;
            // 插入树节点后平衡
            // 把root节点移动到链表的第一个节点
            moveRootToFront(tab, balanceInsertion(root, x));
            return null;
        }
    }
}
```

### 3.11 TreeNode.treeify()

```java
final void treeify(Node<K, V>[] tab) {
    TreeNode<K, V> root = null;
    for (TreeNode<K, V> x = this, next; x != null; x = next) {
        next = (TreeNode<K, V>) x.next;
        x.left = x.right = null;
        // 第一个元素作为根节点且为黑节点，其它元素依次插入到树中再做平衡
        if (root == null) {
            x.parent = null;
            x.red = false;
            root = x;
        } else {
            K k = x.key;
            int h = x.hash;
            Class<?> kc = null;
            // 从根节点查找元素插入的位置
            for (TreeNode<K, V> p = root; ; ) {
                int dir, ph;
                K pk = p.key;
                if ((ph = p.hash) > h)
                    dir = -1;
                else if (ph < h)
                    dir = 1;
                else if ((kc == null &&
                        (kc = comparableClassFor(k)) == null) ||
                        (dir = compareComparables(kc, k, pk)) == 0)
                    dir = tieBreakOrder(k, pk);

                // 如果最后没找到元素，则插入
                TreeNode<K, V> xp = p;
                if ((p = (dir <= 0) ? p.left : p.right) == null) {
                    x.parent = xp;
                    if (dir <= 0)
                        xp.left = x;
                    else
                        xp.right = x;
                    // 插入后平衡，默认插入的是红节点，在balanceInsertion()方法里
                    root = balanceInsertion(root, x);
                    break;
                }
            }
        }
    }
    // 把根节点移动到链表的头节点，因为经过平衡之后原来的第一个元素不一定是根节点了
    moveRootToFront(tab, root);
}
```

### 3.12 TreeNode.getTreeNode(int h, Object k)

```java
final TreeNode<K, V> getTreeNode(int h, Object k) {
    // 从树的根节点开始查找
    return ((parent != null) ? root() : this).find(h, k, null);
}

final TreeNode<K, V> find(int h, Object k, Class<?> kc) {
    TreeNode<K, V> p = this;
    do {
        int ph, dir;
        K pk;
        TreeNode<K, V> pl = p.left, pr = p.right, q;
        if ((ph = p.hash) > h)
            // 左子树
            p = pl;
        else if (ph < h)
            // 右子树
            p = pr;
        else if ((pk = p.key) == k || (k != null && k.equals(pk)))
            // 找到了直接返回
            return p;
        else if (pl == null)
            // hash相同但key不同，左子树为空查右子树
            p = pr;
        else if (pr == null)
            // 右子树为空查左子树
            p = pl;
        else if ((kc != null ||
                (kc = comparableClassFor(k)) != null) &&
                (dir = compareComparables(kc, k, pk)) != 0)
            // 通过compare方法比较key值的大小决定使用左子树还是右子树
            p = (dir < 0) ? pl : pr;
        else if ((q = pr.find(h, k, kc)) != null)
            // 如果以上条件都不通过，则尝试在右子树查找
            return q;
        else
            // 都没找到就在左子树查找
            p = pl;
    } while (p != null);
    return null;
}
```

### 3.13 TreeNode.removeTreeNode(...)

```java
final void removeTreeNode(HashMap<K, V> map, Node<K, V>[] tab,
                          boolean movable) {
    int n;
    // 如果桶的数量为0直接返回
    if (tab == null || (n = tab.length) == 0)
        return;
    // 节点在桶中的索引
    int index = (n - 1) & hash;
    // 第一个节点，根节点，根左子节点
    TreeNode<K, V> first = (TreeNode<K, V>) tab[index], root = first, rl;
    // 后继节点，前置节点
    TreeNode<K, V> succ = (TreeNode<K, V>) next, pred = prev;

    if (pred == null)
        // 如果前置节点为空，说明当前节点是根节点，则把后继节点赋值到第一个节点的位置，相当于删除了当前节点
        tab[index] = first = succ;
    else
        // 否则把前置节点的下个节点设置为当前节点的后继节点，相当于删除了当前节点
        pred.next = succ;

    // 如果后继节点不为空，则让后继节点的前置节点指向当前节点的前置节点，相当于删除了当前节点
    if (succ != null)
        succ.prev = pred;

    // 如果第一个节点为空，说明没有后继节点了，直接返回
    if (first == null)
        return;

    // 如果根节点的父节点不为空，则重新查找父节点
    if (root.parent != null)
        root = root.root();

    // 如果根节点为空，则需要反树化（将树转化为链表）
    // 如果需要移动节点且树的高度比较小，则需要反树化
    if (root == null
            || (movable
            && (root.right == null
            || (rl = root.left) == null
            || rl.left == null))) {
        tab[index] = first.untreeify(map);  // too small
        return;
    }

    // 分割线，以上都是删除链表中的节点，下面才是直接删除红黑树的节点（因为TreeNode本身即是链表节点又是树节点）

    // 删除红黑树节点的大致过程是寻找右子树中最小的节点放到删除节点的位置，然后做平衡，此处不过多注释
    TreeNode<K, V> p = this, pl = left, pr = right, replacement;
    if (pl != null && pr != null) {
        TreeNode<K, V> s = pr, sl;
        while ((sl = s.left) != null) // find successor
            s = sl;
        boolean c = s.red;
        s.red = p.red;
        p.red = c; // swap colors
        TreeNode<K, V> sr = s.right;
        TreeNode<K, V> pp = p.parent;
        if (s == pr) { // p was s's direct parent
            p.parent = s;
            s.right = p;
        } else {
            TreeNode<K, V> sp = s.parent;
            if ((p.parent = sp) != null) {
                if (s == sp.left)
                    sp.left = p;
                else
                    sp.right = p;
            }
            if ((s.right = pr) != null)
                pr.parent = s;
        }
        p.left = null;
        if ((p.right = sr) != null)
            sr.parent = p;
        if ((s.left = pl) != null)
            pl.parent = s;
        if ((s.parent = pp) == null)
            root = s;
        else if (p == pp.left)
            pp.left = s;
        else
            pp.right = s;
        if (sr != null)
            replacement = sr;
        else
            replacement = p;
    } else if (pl != null)
        replacement = pl;
    else if (pr != null)
        replacement = pr;
    else
        replacement = p;
    if (replacement != p) {
        TreeNode<K, V> pp = replacement.parent = p.parent;
        if (pp == null)
            root = replacement;
        else if (p == pp.left)
            pp.left = replacement;
        else
            pp.right = replacement;
        p.left = p.right = p.parent = null;
    }

    TreeNode<K, V> r = p.red ? root : balanceDeletion(root, replacement);

    if (replacement == p) {  // detach
        TreeNode<K, V> pp = p.parent;
        p.parent = null;
        if (pp != null) {
            if (p == pp.left)
                pp.left = null;
            else if (p == pp.right)
                pp.right = null;
        }
    }
    if (movable)
        moveRootToFront(tab, r);
}

```

## 4 遍历方式

遍历分为7种方式：

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