## 一、 Map集合

### 1.1 概述

现实生活中，我们常会看到这样的一种集合：IP地址与主机名，身份证号与个人，系统用户名与系统用户对象等，这种一一对应的关系，就叫做**映射**。Java提供了专门的集合类用来存放这种对象关系的对象，即`java.util.Map`接口。

`Map`接口下的集合与`Collection`接口下的集合，它们存储数据的形式不同，是有很大区别的：

* `Collection`中的集合，元素是孤立存在的（理解为单身），向集合中存储元素采用一个个元素的方式存储。
* `Map`中的集合，元素是成对存在的**键值对**(key-value)。每个元素由键与值两部分组成，通过键可以找对所对应的值。
* `Collection`中的集合称为单列集合，`Map`中的集合称为双列集合。
* 需要注意的是，**`Map`中的集合不能包含重复的键，值可以重复；每个键只能对应一个值**。

### 1.2  Map架构

<img src="https://img-blog.csdnimg.cn/202009262213133.png" width="60%" alt=""/>

* AbstractMap 是**继承于Map的抽象类，它实现了Map中的大部分API**。其它Map的实现类可以通过继承AbstractMap来减少重复编码。
* SortedMap 是继承于Map的接口。SortedMap中的内容是**排序的键值对**，排序的方法是通过比较器(Comparator)。
* NavigableMap 是继承于SortedMap的接口。相比于SortedMap，NavigableMap有一系列的导航方法；如"获取大于/等于某对象的键值对"、“获取小于/等于某对象的键值对”等等。
* **TreeMap** 继承于AbstractMap，且实现了NavigableMap接口；TreeMap中的内容是“**有序的键值对**”！
* **HashMap<K,V>**：存储数据采用的哈希表结构，因为没实现NavigableMap接口，所以**元素的存取顺序不能保证**。由于要保证键的唯一、不重复，需要重写键的hashCode()方法、equals()方法。
* **LinkedHashMap<K,V>**：HashMap的子类，存储数据采用的**哈希表结构+链表**结构。通过链表结构可以保证元素的存取顺序一致；通过哈希表结构可以保证的键的唯一、不重复，需要重写键的hashCode()方法、equals()方法。
* **Hashtable** 虽然不是继承于AbstractMap，但它继承于Dictionary(Dictionary也是键值对的接口)，而且也实现Map接口；因此，Hashtable的内容也是“**键值对，也不保证次序**”。但和HashMap相比，Hashtable是线程安全的，而且它支持通过Enumeration去遍历。
* WeakHashMap 继承于AbstractMap。它和HashMap的键类型不同，**WeakHashMap的键是“弱键”**。
* EnumMap几乎和HashMap是一样的，区别在于EnumMap的key是一个Enum。

> Tips：Map接口中的集合都有两个泛型变量<K,V>，在使用时，要为两个泛型变量赋予数据类型。两个泛型变量<K,V>的数据类型可以相同，也可以不同。
>

### 1.3  常用方法

Map接口中定义了很多方法，常用的如下：

* `public V put(K key, V value)`：把指定的键与指定的值添加到Map集合中。
* `public V remove(Object key)`：把指定的键 所对应的键值对元素 在Map集合中删除，返回被删除元素的值。
* `public V get(Object key)` ：根据指定的键，在Map集合中获取对应的值。
* `boolean containsKey(Object key)  ` ：判断集合中是否包含指定的键。
* `public Set<K> keySet()`：获取Map集合中所有的键，存储到Set集合中。
* `public Set<Map.Entry<K,V>> entrySet()`：获取到Map集合中所有的键值对对象的集合(Set集合)。

Map接口的方法演示：

~~~java
public class MapTest {
    public static void main(String[] args) {
        //创建 map对象
        HashMap<String, String> map = new HashMap<String, String>();

        //添加元素到集合
        map.put("黄晓明", "杨颖");
        map.put("邓超", "孙俪");
        map.put("我", "刘亦菲");

        System.out.println(map);

        //String remove(String key)
        System.out.println(map.remove("邓超"));
        System.out.println(map);

        // 想要查看 我的媳妇 是谁
        System.out.println(map.get("我"));
        System.out.println(map.get("邓超"));
    }
}
~~~

执行结果：

```shell
{我=刘亦菲, 黄晓明=杨颖}
null
null
```

> Tips：
>
> 使用put方法时，若指定的键(key)在集合中没有，则这个键对应的值返回null，并把指定的键值添加到集合中； 
>
> 若指定的键(key)在集合中存在，则返回值为集合中键对应的值（该值为替换前的值），并把指定键所对应的值，替换成指定的新值。 

### 1.4   遍历键取值

键找值方式：即通过元素中的键，获取键所对应的值。

分析步骤：

1. 获取Map中所有的键，由于键是唯一的，所以返回一个Set集合存储所有的键。方法提示:`keyset()`
2. 遍历键的Set集合，得到每一个键。
3. 根据键，获取键所对应的值。方法提示:`get(K key)`

代码演示：

~~~java
public class MapTest {
    public static void main(String[] args) {
        HashMap<String, String> map = new HashMap<>();

        //添加元素到集合
        map.put("黄晓明", "杨颖");
        map.put("邓超", "孙俪");
        map.put("我", "刘亦菲");

        //获取所有的键  获取键集
        Set<String> keys = map.keySet();
        // 遍历键集 得到 每一个键
        for (String key : keys) {
            //key  就是键
            //获取对应值
            String value = map.get(key);
            System.out.println(key + "的老婆是：" + value);
        }
    }
}
~~~

执行结果：

```java
邓超的老婆是：孙俪
我的老婆是：刘亦菲
黄晓明的老婆是：杨颖
```

### 1.5  Entry键值对

我们已经知道，`Map`中存放的是两种对象，一种称为**key**(键)，一种称为**value**(值)，它们在在`Map`中是一一对应关系，这一对对象又称做`Map`中的一个`Entry(项)`。`Entry`将键值对的对应关系封装成了对象，即**键值对对象**，这样我们在遍历`Map`集合时，就可以从每一个键值对（`Entry`）对象中获取对应的键与对应的值。

 既然Entry表示了一对键和值，那么也同样提供了获取对应键和对应值得方法：

* `public K getKey()`：获取Entry对象中的键。
* `public V getValue()`：获取Entry对象中的值。

在Map集合中也提供了获取所有Entry对象的方法：

* `public Set<Map.Entry<K,V>> entrySet()`: 获取到Map集合中所有的键值对对象的集合(Set集合)。

### 1.6 遍历Entry取值

遍历键值对方式：即通过集合中每个键值对(Entry)对象，获取键值对(Entry)对象中的键与值。

操作步骤：

1.  获取Map集合中，所有的键值对(Entry)对象，以Set集合形式返回。方法提示:`entrySet()`。

2.  遍历包含键值对(Entry)对象的Set集合，得到每一个键值对(Entry)对象。
3.  通过键值对(Entry)对象，获取Entry对象中的键与值。  方法提示:`getkey() getValue()`     

~~~java
public class MapTest {
    public static void main(String[] args) {
        HashMap<String, String> map = new HashMap<>();
        //添加元素到集合
        map.put("黄晓明", "杨颖");
        map.put("邓超", "孙俪");
        map.put("我", "刘亦菲");

        // 获取所有的entry对象
        Set<Map.Entry<String,String>> entrySet = map.entrySet();
        // 遍历得到entry对象
        for (Map.Entry<String,String> entry : entrySet) {
            // 解析
            String key = entry.getKey();
            String value = entry.getValue();
            System.out.println(key + "的老婆是：" + value);
        }
    }
}
~~~

> Tips：**Map集合不能直接使用迭代器或者foreach进行遍历**，但是转成Set之后就可以使用了。
>

## 二、HashMap

### 2.1 概述

HashMap 是一个散列表，它存储的内容是键值对(key-value)映射。它继承于AbstractMap，实现了Map、Cloneable、java.io.Serializable接口。

![](https://img-blog.csdnimg.cn/20200927084947814.png)

HashMap的实现是不同步的，这意味着**线程不安全**，所以不要在并发的环境中同时操作HashMap，建议使用ConcurrentHashMap。

HashMap的key、value都可以为null（**最多只允许**一条记录的键为null，可允许多条记录的值为null）。

此外，HashMap中的映射是**无序**的。

HashMap 的实例有两个参数影响其性能：“**初始容量**” 和 “**加载因子**”。容量 是哈希表中桶的数量，初始容量 只是哈希表在创建时的容量。

加载因子 是哈希表在其容量自动增加之前可以达到多满的一种尺度。当哈希表中的条目数超出了加载因子与当前容量的乘积时，则要对

该哈希表进行 rehash 操作（即重建内部数据结构），从而哈希表将具有大约两倍的桶数。

通常，**默认加载因子是 0.75**, 这是在时间和空间成本上寻求一种折衷。加载因子过高虽然减少了空间开销，但同时也增加了查询成本

（大多数 HashMap 类的操作中，包括 get 和 put 操作，都反映了这一点）。在设置初始容量时应该考虑到映射中所需的条目数及其

加载因子，以便最大限度地减少 rehash 操作次数。如果初始容量大于最大条目数除以加载因子，则不会发生 rehash 操作。

### 2.2 数据结构

HashMap是**数组+链表+红黑树**（JDK1.8增加了红黑树部分）实现的，如下所示。

<img src="https://img-blog.csdnimg.cn/img_convert/6061bbb279b3263b041149dbf82fa244.png" width="60%" alt=""/>

查看源码可知，HashMap类中有一个非常重要的字段，就是 Node[] table，即哈希桶数组。

```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;    //用来定位数组索引位置
        final K key;
        V value;
        Node<K,V> next;   //链表的下一个node

        Node(int hash, K key, V value, Node<K,V> next) { ... }
        public final K getKey(){ ... }
        public final V getValue() { ... }
        public final String toString() { ... }
        public final int hashCode() { ... }
        public final V setValue(V newValue) { ... }
        public final boolean equals(Object o) { ... }
}
```

Node是HashMap的一个内部类，实现了Map.Entry接口，本质是就是一个映射(键值对)。上图中的每个黑色圆点就是一个Node对象，它持有一个指向下一个元素的引用，这就构成了链表。

### 2.3 常用方法

HashMap中常用的方法有：

* **void clear（）**：从指定的Map中删除所有键和值对。
* **Object clone（）**：它返回一个映射的所有映射的副本，并用于将它们克隆到另一个映射中。
* **boolean containsKey（Object key）**：这是一个布尔函数，根据在Map中是否找到指定的键来返回true或false。
* **boolean containsValue（Object Value）**：类似于containsKey（）方法，但是它将查找指定的值而不是键。
* **Value get(Object key)**）：它返回指定键的值。
* **boolean isEmpty（）**：检查映射是否为空。如果映射中不存在键值映射，则此函数返回true，否则返回false。
* **Set keySet（）**：它返回从Map获取的键的Set。
* **value put（Key k，Value v）**：将键值映射插入到映射中。
* **int size（）**：返回映射的大小–键值映射的数量。
* **Collection values（）**：它返回Map的值的集合。
* **Value remove(Object key)**：删除指定键的键值对。
* **void putAll（Map m）**：将Map的所有元素复制到另一个指定的Map。

### 2.4 遍历方式

遍历分为四种方式：

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

### 2.5  存储自定义类型键值

编写学生类：

~~~java
public class Student {
    private String name;
    private int age;

    public Student() {
    }

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o)
            return true;
        if (o == null || getClass() != o.getClass())
            return false;
        Student student = (Student) o;
        return age == student.age && Objects.equals(name, student.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
~~~

编写测试类：

~~~java 
public class HashMapTest {
    public static void main(String[] args) {
        //1、创建Hashmap集合对象。
        Map<Student, String> map = new HashMap<Student, String>();
        //2、添加元素。
        map.put(new Student("张三", 10), "上海");
        map.put(new Student("李四", 20), "北京");
        map.put(new Student("王五", 24), "深圳");

        //3、取出元素
        Iterator<Map.Entry<Student,String>> iter = map.entrySet().iterator();
        while (iter.hasNext()) {
            Map.Entry<Student,String> entry = iter.next();
            Student key = entry.getKey();
            String val = entry.getValue();
            System.out.println(key.toString() + "..." + val);
        }
    }
}
~~~

输出结果：

```shell
Student{name='李四', age=20}...北京
Student{name='王五', age=24}...深圳
Student{name='张三', age=10}...上海
```

* 当给HashMap中存放自定义对象时，如果自定义对象作为key存在，这时要保证对象唯一，必须复写对象的hashCode和equals方法
* 如果要保证map中存放的key和取出的顺序一致，可以使用`java.util.LinkedHashMap`集合来存放。

## 三、LinkedHashMap

对于 LinkedHashMap 而言，它继承与 HashMap(`public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>`)、底层使用**哈希表与双向链表**来保存所有元素。其基本操作与父类 HashMap 相似，它通过重写父类相关的方法，来实现自己的链接列表特性。

**LinkedHashMap 能够做到按照插入顺序或者访问顺序进行迭代。**

~~~java
public class LinkedHashMapTest {
    public static void main(String[] args) {
        LinkedHashMap<String, String> map = new LinkedHashMap<>();
        //添加元素到集合
        map.put("黄晓明", "杨颖");
        map.put("邓超", "孙俪");
        map.put("我", "刘亦菲");
        Set<Map.Entry<String, String>> entrySet = map.entrySet();
        for (Map.Entry<String, String> entry : entrySet) {
            System.out.println(entry.getKey() + "的老婆是" + entry.getValue());
        }
    }
}
~~~

结果:

~~~
黄晓明的老婆是杨颖
邓超的老婆是孙俪
我的老婆是刘亦菲
~~~

## 四、Hashtable

### 4.1 概述

和Hashtable 一样，Hashtable 也是一个**散列表**，它存储的内容是**键值对(key-value)映射**。

Hashtable **继承于Dictionary**，实现了Map、Cloneable、java.io.Serializable接口。

![](https://img-blog.csdnimg.cn/20200927150723352.png)

Hashtable 的函数都是**同步的**，这意味着它是线程安全的。它的key、value都不可以为null。此外，Hashtable中的映射不是有序的。Hashtable 的实例有两个参数影响其性能：**初始容量** 和 **加载因子**。容量 是哈希表中桶 的数量，初始容量 就是哈希表创建时的容量。注意，哈希表的状态为 open：在发生“哈希冲突”的情况下，单个桶会存储多个条目，这些条目必须按顺序搜索。加载因子 是对哈希表在其容量自动增加之前可以达到多满的一个尺度。初始容量和加载因子这两个参数只是对该实现的提示。关于何时以及是否调用 rehash 方法的具体细节则依赖于该实现。
通常，**默认加载因子是 0.75**, 这是在时间和空间成本上寻求一种折衷。加载因子过高虽然减少了空间开销，但同时也增加了查找某个条目的时间（在大多数 Hashtable 操作中，包括 get 和 put 操作，都反映了这一点）。

### 4.2 构造函数

```java
// 默认构造函数。
public Hashtable() 

// 指定“容量大小”的构造函数
public Hashtable(int initialCapacity) 

// 指定“容量大小”和“加载因子”的构造函数
public Hashtable(int initialCapacity, float loadFactor) 

// 包含“子Map”的构造函数
public Hashtable(Map<? extends K, ? extends V> t)
```

### 4.3 成员变量

Hashtable是通过"拉链法"实现的哈希表。它包括几个重要的成员变量：table， count，threshold，loadFactor， modCount。

- table是一个 Entry[] 数组类型，而 Entry实际上就是一个单向链表。哈希表的"key-value键值对"都是存储在Entry数组中的。
- count 是 Hashtable 的大小，它是 Hashtable 保存的键值对的数量。
- threshold 是 Hashtable 的阈值，用于判断是否需要调整 Hashtable 的容量。threshold 的值="容量*加载因子"。
- loadFactor 就是加载因子。
- modCount 是用来实现 fail-fast 机制的。

### 4.4 常用方法

```java
synchronized void                clear()
synchronized Object              clone()
             boolean             contains(Object value)
synchronized boolean             containsKey(Object key)
synchronized boolean             containsValue(Object value)
synchronized Enumeration<V>      elements()
synchronized Set<Entry<K, V>>    entrySet()
synchronized boolean             equals(Object object)
synchronized V                   get(Object key)
synchronized int                 hashCode()
synchronized boolean             isEmpty()
synchronized Set<K>              keySet()
synchronized Enumeration<K>      keys()
synchronized V                   put(K key, V value)
synchronized void                putAll(Map<? extends K, ? extends V> map)
synchronized V                   remove(Object key)
synchronized int                 size()
synchronized String              toString()
synchronized Collection<V>       values()
```

## 五、 模拟斗地主洗牌发牌

### 5.1 案例介绍

按照斗地主的规则，完成洗牌发牌的动作。

具体规则：

1. 组装54张扑克牌将
2. 54张牌顺序打乱
3. 三个玩家参与游戏，三人交替摸牌，每人17张牌，最后三张留作底牌。
4. 查看三人各自手中的牌（按照牌的大小排序）、底牌

> 规则：手中扑克牌从大到小的摆放顺序：大王,小王,2,A,K,Q,J,10,9,8,7,6,5,4,3
>

### 5.2 案例需求分析

1.  准备牌：


完成数字与纸牌的映射关系：

使用双列Map(HashMap)集合，完成一个数字与字符串纸牌的对应关系(相当于一个字典)。

2.  洗牌：

通过数字完成洗牌发牌

3.  发牌：

将每个人以及底牌设计为ArrayList<String>,将最后3张牌直接存放于底牌，剩余牌通过对3取模依次发牌。

存放的过程中要求数字大小与斗地主规则的大小对应。

将代表不同纸牌的数字分配给不同的玩家与底牌。

4.  看牌：

通过Map集合找到对应字符展示。

通过查询纸牌与数字的对应关系，由数字转成纸牌字符串再进行展示。

### 5.3  实现代码步骤

~~~java
public class Poker {
    public static void main(String[] args) {
        /*
         * 1组装54张扑克牌
         */
        // 1.1 创建Map集合存储
        HashMap<Integer, String> pokerMap = new HashMap<Integer, String>();
        // 1.2 创建 花色集合 与 数字集合
        ArrayList<String> colors = new ArrayList<String>();
        ArrayList<String> numbers = new ArrayList<String>();

        // 1.3 存储 花色 与数字
        Collections.addAll(colors, "♦", "♣", "♥", "♠");
        Collections.addAll(numbers, "2", "A", "K", "Q", "J", "10", "9", "8", "7", "6", "5", "4", "3");
        // 设置 存储编号变量
        int count = 1;
        pokerMap.put(count++, "大王");
        pokerMap.put(count++, "小王");
        // 1.4 创建牌 存储到map集合中
        for (String number : numbers) {
            for (String color : colors) {
                String card = color + number;
                pokerMap.put(count++, card);
            }
        }
        /*
         * 2 将54张牌顺序打乱
         */
        // 取出编号 集合
        Set<Integer> numberSet = pokerMap.keySet();
        // 因为要将编号打乱顺序 所以 应该先进行转换到 list集合中
        ArrayList<Integer> numberList = new ArrayList<Integer>();
        numberList.addAll(numberSet);

        // 打乱顺序
        Collections.shuffle(numberList);

        // 3 完成三个玩家交替摸牌，每人17张牌，最后三张留作底牌
        // 3.1 发牌的编号
        // 创建三个玩家编号集合 和一个 底牌编号集合
        ArrayList<Integer> noP1 = new ArrayList<Integer>();
        ArrayList<Integer> noP2 = new ArrayList<Integer>();
        ArrayList<Integer> noP3 = new ArrayList<Integer>();
        ArrayList<Integer> dipaiNo = new ArrayList<Integer>();

        // 3.2发牌的编号
        for (int i = 0; i < numberList.size(); i++) {
            // 获取该编号
            Integer no = numberList.get(i);
            // 发牌
            // 留出底牌
            if (i >= 51) {
                dipaiNo.add(no);
            } else {
                if (i % 3 == 0) {
                    noP1.add(no);
                } else if (i % 3 == 1) {
                    noP2.add(no);
                } else {
                    noP3.add(no);
                }
            }
        }

        // 4 查看三人各自手中的牌（按照牌的大小排序）、底牌
        // 4.1 对手中编号进行排序
        Collections.sort(noP1);
        Collections.sort(noP2);
        Collections.sort(noP3);
        Collections.sort(dipaiNo);

        // 4.2 进行牌面的转换
        // 创建三个玩家牌面集合 以及底牌牌面集合
        ArrayList<String> player1 = new ArrayList<String>();
        ArrayList<String> player2 = new ArrayList<String>();
        ArrayList<String> player3 = new ArrayList<String>();
        ArrayList<String> dipai = new ArrayList<String>();

        // 4.3转换
        for (Integer i : noP1) {
            // 4.4 根据编号找到 牌面 pokerMap
            String card = pokerMap.get(i);
            // 添加到对应的 牌面集合中
            player1.add(card);
        }

        for (Integer i : noP2) {
            String card = pokerMap.get(i);
            player2.add(card);
        }
        for (Integer i : noP3) {
            String card = pokerMap.get(i);
            player3.add(card);
        }
        for (Integer i : dipaiNo) {
            String card = pokerMap.get(i);
            dipai.add(card);
        }

        //4.5 查看
        System.out.println("张三："+player1);
        System.out.println("李四："+player2);
        System.out.println("王五："+player3);
        System.out.println("底牌："+dipai);
    }
}
~~~

## 参考资料

[Java 集合系列之 Map架构](https://www.cnblogs.com/skywang12345/p/3308931.html)

[Java 8系列之重新认识HashMap](https://tech.meituan.com/2016/06/24/java-hashmap.html)

[Java HashMap工作原理及实现](https://yikun.github.io/2015/04/01/Java-HashMap%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/)

 [图解LinkedHashMap原理](https://www.jianshu.com/p/8f4f58b4b8ab)

 

 