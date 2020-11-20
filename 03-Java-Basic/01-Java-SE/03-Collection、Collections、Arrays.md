## 一、Collection架构

![](https://img-blog.csdnimg.cn/2020092615194548.png)

## 二、List集合

### 1.1 List接口

`java.util.List`接口继承自`Collection`接口，是单列集合的一个重要分支，习惯性地将实现了`List`接口的对象称为List集合。

List接口特点：

1. 它是一个元素存取**有序**的集合。
2. 它是一个**带有索引**的集合，通过索引就可以精确的操作集合中的元素（与数组的索引是一个道理）。
3. 集合中**元素可重复**，通过元素的equals方法，来比较是否为重复的元素。

### 1.2 List接口中常用方法

List作为Collection集合的子接口，不但继承了Collection接口中的全部方法，而且还增加了一些根据元素索引来操作集合的特有方法，如下：

- `public void add(int index, E element)`: 将指定的元素，添加到该集合中的指定位置上。
- `public E get(int index)`:返回集合中指定位置的元素。
- `public E remove(int index)`: 移除列表中指定位置的元素, 返回的是被移除的元素。
- `public E set(int index, E element)`:用指定元素替换集合中指定位置的元素,返回值的更新前的元素。

List集合特有的方法都是跟索引相关，我们在基础班都学习过，那么我们再来复习一遍吧：

```java
public class ListTest {
    public static void main(String[] args) {
        // 创建List集合对象
        List<String> list = new ArrayList<String>();

        // 往 尾部添加 指定元素
        list.add("张三");
        list.add("李四");
        list.add("王五");

        System.out.println(list);
        // 指定位置添加
        list.add(1, "帅气");

        System.out.println(list);
        // 删除指定位置元素  返回被删除元素
        System.out.println("删除索引位置为2的元素");
        System.out.println(list.remove(2));

        System.out.println(list);

        // 在指定位置 进行 元素替代（改）
        list.set(0, "三毛");
        System.out.println(list);

        // 跟size() 方法一起用来 遍历的
        for (int i = 0; i < list.size(); i++) {
            System.out.println(list.get(i));
        }
        // foreach
        for (String string : list) {
            System.out.println(string);
        }
    }
}
```

## 三、List的子类

### 3.1 `ArrayList`集合

`java.util.ArrayList` 是一个**数组队列**，相当于 **动态数组**。与Java中的数组相比，它的容量能动态增长，**元素增删慢，查找快**，由于日常开发中使用最多的功能为查询数据、遍历数据，`ArrayList`是最常用的集合。

![](https://img-blog.csdnimg.cn/20200926153745631.png)

* `ArrayList` 继承了`AbstractList`抽象方法，实现了List接口。它是一个数组队列，提供了相关的添加、删除、修改、遍历等功能。
* `ArrayList` 实现了`RandmoAccess`接口，即提供了随机访问功能。`RandmoAccess`是java中用来被List实现，为List提供快速访问功能的。在`ArrayList`中，我们即可以通过元素的序号快速获取元素对象；这就是快速随机访问。稍后，我们会比较List的“快速随机访问”和“通过Iterator迭代器访问”的效率。
* `ArrayList` 实现了Cloneable接口，即覆盖了函数clone()，能被克隆。

* `ArrayList` 实现`java.io.Serializable`接口，这意味着`ArrayList`支持序列化，能通过序列化去传输。

  >Tips：**`ArrayList`中的操作不是线程安全的**！所以，建议在单线程中才使用`ArrayList`，而在多线程中可以选择Vector或者`CopyOnWriteArrayList`。

### 3.2 `LinkedList`集合



![](https://img-blog.csdnimg.cn/20200926154414229.png)

`java.util.LinkedList`是一个继承于`AbstractSequentialList`的**双向链表**。它也可以被当作堆栈、队列或双端队列进行操作。
`LinkedList `实现来` List `接口，能对它进行队列操作。
`LinkedList `实现了` Deque` 接口，即能将`LinkedList`当作双端队列使用。
`LinkedList `实现了了`Cloneable`接口，即覆盖了函数clone()，能够克隆。
`LinkedList `实现了`java.io.Serializable`接口，这意味着`LinkedList `支持序列化，能通过序列化传输数据。



实际开发中对一个集合元素的添加与删除经常涉及到首尾操作，而`LinkedList`提供了大量首尾操作的方法：

* `public void addFirst(E e)`:将指定元素插入此列表的开头。
* `public void addLast(E e)`:将指定元素添加到此列表的结尾。
* `public E getFirst()`:返回此列表的第一个元素。
* `public E getLast()`:返回此列表的最后一个元素。
* `public E removeFirst()`:移除并返回此列表的第一个元素。
* `public E removeLast()`:移除并返回此列表的最后一个元素。
* `public E pop()`:从此列表所表示的堆栈处弹出一个元素。
* `public void push(E e)`:将元素推入此列表所表示的堆栈。
* `public boolean isEmpty()`：如果列表不包含元素，则返回true。

方法演示：

~~~java
public class LinkedListTest {
    public static void main(String[] args) {
        LinkedList<String> link = new LinkedList<String>();
        //添加元素
        link.addFirst("张三");
        link.addFirst("李四");
        link.addFirst("王二");
        System.out.println(link);
        // 获取元素
        System.out.println(link.getFirst());
        System.out.println(link.getLast());
        // 删除元素
        System.out.println(link.removeFirst());
        System.out.println(link.removeLast());

        while (!link.isEmpty()) { //判断集合是否为空
            System.out.println(link.pop()); //弹出集合中的栈顶元素
        }
        System.out.println(link);
    }
}
~~~

## 四、Set接口

`java.util.Set`接口和`java.util.List`接口一样，同样继承自`Collection`接口，它与`Collection`接口中的方法基本一致，并没有对`Collection`接口进行功能上的扩充，只是比`Collection`接口更加严格了。与`List`接口不同的是，`Set`接口中**元素无序**，并且都会以某种规则保证存入的**元素不重复**。

<img src="https://img-blog.csdnimg.cn/20200926201509451.png" width="40%" alt=""/>

`Set`集合有多个子类，这里我们介绍其中的`java.util.HashSet`、`java.util.LinkedHashSet`这两个集合。

> Tips：Set集合取出元素的方式可以采用：迭代器、foreach循环。

### 4.1 HashSet集合介绍

`java.util.HashSet`是`Set`接口的一个实现类，它所存储的元素是不可重复的，并且元素都是无序的(即存取顺序不一致)。`java.util.HashSet`底层的实现其实是一个`java.util.HashMap`支持。

`HashSet`是根据对象的哈希值来确定元素在集合中的存储位置，因此具有良好的存取和查找性能。保证元素唯一性的方式依赖于：

重写`hashCode`与`equals`方法。

我们先来使用一下Set集合存储，看下现象，再进行原理的讲解:

~~~java
public class HashSetTest {
    public static void main(String[] args) {
        //创建 Set集合
        HashSet<String>  set = new HashSet<String>();
        //添加元素
        set.add(new String("张三"));
        set.add("李四");
        set.add("王二"); 
        set.add("王二"); 
        //遍历
        for (String name : set) {
            System.out.println(name);
        }
    }
}
~~~

输出结果如下：

~~~
张三
李四
王二
~~~

> Tips：字符串"王二"只存储了一个，也就是说重复的元素set集合不存储。

### 4.2  HashSet集合存储数据的结构（哈希表）

什么是哈希表呢？

在**JDK1.8**之前，哈希表底层采用数组+链表实现，即使用链表处理冲突，同一hash值的链表都存储在一个链表里。但是当位于一个桶中的元素较多，即hash值相等的元素较多时，通过key值依次查找的效率较低。而JDK1.8中，哈希表存储采用**数组+链表+红黑树**实现，当链表长度超过阈值（8）时，将链表转换为红黑树，这样大大减少了查找时间。

简单的来说，哈希表是由数组+链表+红黑树（JDK1.8增加了红黑树部分）实现的，如下图所示。

<img src="https://img-blog.csdnimg.cn/20200926201855988.png" width="60%" alt=""/>

为了理解结合一个存储流程图来说明一下：

<img src="https://img-blog.csdnimg.cn/20200901131713701.png" width="80%" alt=""/>

总而言之，**JDK1.8**引入红黑树大程度优化了HashMap的性能，那么对于我们来讲保证HashSet集合元素的唯一，其实就是根据对象的hashCode和equals方法来决定的。如果我们往集合中存放自定义的对象，那么保证其唯一，就必须复写hashCode和equals方法建立属于当前对象的比较方式。

### 4.3 HashSet存储自定义类型元素

给HashSet中存放自定义类型元素时，需要重写对象中的`hashCode`和`equals`方法，建立自己的比较方式，才能保证HashSet集合中的对象唯一。

创建自定义Student类：

~~~java
public class Student {
    private String name;
    private int age;

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
        if (this == o) {
            return true;
        }
        if (o == null || getClass() != o.getClass()) {
            return false;
        }
        Student student = (Student) o;
        return age == student.age &&
                Objects.equals(name, student.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
~~~

~~~java
public class HashSetTest {
    public static void main(String[] args) {
        //创建集合对象   该集合中存储 Student类型对象
        HashSet<Student> student = new HashSet<Student>();
        //存储
        student.add(new Student("张三", 20));
        student.add(new Student("李四", 30));
        student.add(new Student("王五", 40));

        for (Student person : student) {
            System.out.println(person);
        }
    }
}
~~~

执行结果：

```shell
Student{name='张三', age=20}
Student{name='王五', age=40}
Student{name='李四', age=30}
```

### 4.4 LinkedHashSet

对于 LinkedHashSet 而言，它继承与 HashSet、又基于 LinkedHashMap 来实现的。

`java.util.LinkedHashSet`底层是 **数组 + 单链表 + 红黑树 + 双向链表**的数据结构。存储元素是无序的，但是由于双向链表的存在，迭**代时获取元素的顺序等于元素的添加顺序**，注意这里不是访问顺序。

查看LinkedHashSet源码并没有看到出现过 LinkedHashMap，那是因为LinkedHashSet 的构造方法中，其调用了父类的构造方法。

```java
/**
     * 以指定的initialCapacity和loadFactor构造一个新的空链接哈希集合。
     * 此构造函数为包访问权限，不对外公开，实际只是是对LinkedHashSet的支持。
     *
     * 实际底层会以指定的参数构造一个空LinkedHashMap实例来实现。
     * @param initialCapacity 初始容量。
     * @param loadFactor 加载因子。
     * @param dummy 标记。
     */
HashSet(int initialCapacity, float loadFactor, boolean dummy) {
    map = new LinkedHashMap<E,Object>(initialCapacity, loadFactor);
}
```

演示代码:

~~~java
public class LinkedHashSetTest {
	public static void main(String[] args) {
		Set<String> set = new LinkedHashSet<String>();
		set.add("123");
		set.add("456");
		set.add("789");
		set.add("abc");
        Iterator<String> it = set.iterator();
		while (it.hasNext()) {
			System.out.println(it.next());
		}
	}
}
~~~

结果：

```shell
123
456
789
abc
```

## 五、  Collections工具类

### 5.1 常用功能

`java.utils.Collections`是集合工具类，用来对集合进行操作。部分方法如下：

- `public static <T> boolean addAll(Collection<T> c, T... elements)  `：往集合中添加一些元素。
- `public static void shuffle(List<?> list) `：打乱集合顺序。
- `public static <T> void sort(List<T> list)`：将集合中元素按照默认规则排序。
- `public static <T> void sort(List<T> list，Comparator<? super T> )`：将集合中元素按照指定规则排序。

代码演示：

```java
public class CollectionsTest {

    public static void main(String[] args) {
        ArrayList<Integer> list = new ArrayList<Integer>();
        //原来写法
        //list.add(1);
        //list.add(2);
        //list.add(3);
        //list.add(4);
        //采用工具类 完成 往集合中添加元,
        Collections.addAll(list, 1, 2, 3, 4);
        System.out.println(list);
        //排序方法
        Collections.sort(list);
        System.out.println(list);
    }
}
```

测试结果：

```shell
[4, 2, 3, 1]
[1, 2, 3, 4]
```

实例中，集合按照默认顺序进行了排列，如果想要指定顺序那该怎么办呢？

我们发现还有个方法没有讲，`public static <T> void sort(List<T> list，Comparator<? super T> )`：将集合中元素按照指定规则排序。接下来讲解一下指定规则的排列。

### 5.2 Comparator比较器

我们还是先研究照默认规则排序：`public static <T> void sort(List<T> list)`

不过这次存储的是字符串类型。

```java
public class CollectionsTest2 {
    public static void main(String[] args) {
        ArrayList<String>  list = new ArrayList<String>();
        list.add("cba");
        list.add("aba");
        list.add("sba");
        list.add("nba");
        //排序方法
        Collections.sort(list);
        System.out.println(list);
    }
}
```

结果：

```
[aba, cba, nba, sba]
```

我们使用的是默认的规则完成字符串的排序，那么默认规则是怎么定义出来的呢？

说到排序了，简单的说就是两个对象之间比较大小，那么在JAVA中提供了两种比较实现的方式，一种是比较死板的采用`java.lang.Comparable`接口去实现，一种是灵活的当我需要做排序的时候在去选择的`java.util.Comparator`接口完成。

那么我们采用的`public static <T> void sort(List<T> list)`这个方法完成的排序，实际上要求了被排序的类型需要实现Comparable接口完成比较的功能，在String类型上如下：

```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
```

String类实现了这个接口，并完成了比较规则的定义，但是这样就把这种规则写死了，那比如我想要字符串按照第一个字符降序排列，那么这样就要修改String的源代码，这是不可能的了，那么这个时候我们可以使用

`public static <T> void sort(List<T> list，Comparator<? super T> )`方法灵活的完成，这个里面就涉及到了Comparator这个接口，它位于java.util包下，排序是comparator能实现的功能之一。需要比较两个对象谁排在前谁排在后，那么比较的方法就是：

* ` public int compare(String o1, String o2)`：比较其两个参数的顺序。

  > 两个对象比较的结果有三种：大于，等于，小于。
  >
  > 如果要按照升序排序，
  > 则o1 小于o2，返回（负数），相等返回0，01大于02返回（正数）
  > 如果要按照降序排序
  > 则o1 小于o2，返回（正数），相等返回0，01大于02返回（负数）

实例：

```java
public class CollectionsTest3 {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<String>();
        list.add("cba");
        list.add("aba");
        list.add("sba");
        list.add("nba");
        //排序方法  按照第二个单词的降序
        Collections.sort(list, new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                return o2.charAt(0) - o1.charAt(0);
            }
        });
        System.out.println(list);
    }
}
```

结果如下：

```
[sba, nba, cba, aba]
```

### 5.3 Comparable和Comparator异同

**Comparable**：Comparable位于java.lang包下。强行对实现它的每个类的对象进行整体排序。这种排序被称为类的自然排序，类的`compareTo`方法被称为它的**自然比较方法。**只能在类中实现compareTo()一次，不能经常修改类的代码实现自己想要的排序。实现此接口的对象列表（和数组）可以通过Collections.sort（和Arrays.sort）进行自动排序，对象可以用作有序映射中的键或有序集合中的元素，无需指定比较器。

**Comparator**：Comparator位于java.util包下。强行对某个对象进行整体排序。可以将Comparator 传递给sort方法（如Collections.sort或 Arrays.sort），从而允许在排序顺序上实现精确控制。还可以使用Comparator来控制某些数据结构（如有序set或有序映射）的顺序，或者为那些没有自然顺序的对象collection提供排序。

## 六、Arrays工具类

Arrays类是**数组**的操作类，定义在java.util包中，主要功能是实现数组元素的查找、数组内容的充填和排序等功能。 Arrays 类里均为 **static 修饰的方法**（static 修饰的方法可以直接通过类名调用），可以直接通过 **Arrays.xxx(xxx)** 的形式调用方法。下面了看几个常用方法。

**1．使用sort()方法排序**

```java
public class ArraysTest {
    public static void main(String[] args) {
        int[] num = {5, 1, 5, 7, 3, 8, -1, 0};
        Arrays.sort(num);
        for (int i = 0; i < num.length; i++) {
            System.out.print(num[i] + "\t");
        }
    }
}
```

**2．使用binarySearch(Object[] a, Object key)方法查找元素**

使用**二分搜索法**来搜索指定类型数组，以获得指定的值。

```java
public class ArraysTest {
    public static void main(String[] args) {
        int[] num = {5, 1, 5, 7, 3, 8, -1, 0};
        Arrays.sort(num);
        int index = Arrays.binarySearch(num, 3);
        System.out.println("元素3的索引是:" + index);
    }
}
```

**3．使用copyOfRange(int[] original, int from, int to)方法拷贝元素**

该方法中参数original表示被复制的数组，from表示被复制元素的初始索引（包括），to表示被复制元素的最后索引（不包括）。

```java
public class ArraysTest {
    public static void main(String[] args) {
        int[] num = {5, 1, 5, 7, 3, 8, -1, 0};
        // 复制一个指定范围的数组
        int[] copied = Arrays.copyOfRange(num, 1, 4);
        for (int i = 0; i < copied.length; i++) {
            System.out.print(copied[i] + " ");
        }
    }
}
```

运行结果：

```shel
1 5 7 
```

**4．使用fill(Object[] a, Object val)方法替换元素**

用指定的值来填充数组，可以指定需要填充的索引范围。

```java
public class ArraysTest {
    public static void main(String[] args) {
        int[] num = {5, 1, 5, 7, 3, 8, -1, 0};
        // 填充的开始位
        Integer startIndex = 2;
        // 填充的结束位
        Integer endIndex = 5;
        // 用8替换数组中的元素
        Arrays.fill(num, startIndex, endIndex, 9);

        System.out.println("当前数组容器："+Arrays.toString(num));
    }
}
```

运行结果：

```shell
当前数组容器：[5, 1, 9, 9, 9, 8, -1, 0]
```

## 参考资料

[Java 集合系列之 Collection架构](https://www.cnblogs.com/skywang12345/p/3308513.html)

[LinkedHashSet 的实现原理](https://wiki.jikexueyuan.com/project/java-collection/linkedhashset.html)

[JavaSE基础:Arrays工具类](https://juejin.im/post/6844903556131061767)