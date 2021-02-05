# Java里面已有`==`运算符了，为什么还需要equals()呢？

在理解`==`之前，我们先来明确什么是基本数据类型与引用数据类型：

**基本数据类型**包括 boolean（布尔型）、float（单精度浮点型）、char（字符型）、byte（字节型）、short（短整型）、int（整型）、long（长整型）和 double （双精度浮点型）共 8 种（注意并没有string）。

**引用数据类型**建立在基本数据类型的基础上，主要包括数组、类和接口。可以这么理解，引用数据类型就是对一个对象的引用，而对象包括实例和数组两种。

也就是说，**基本数据类型变量存的是数据本身，而引用类型变量存的是保存数据的空间地址**（基本数据类型变量里存储的是直接放在抽屉里的东西；而引用数据类型变量里存储的是这个抽屉的钥匙，钥匙和抽屉一一对应）。

来举个例子：

```java
        Integer n1 = new Integer(47);
        Integer n2 = 47;// 自动装箱，等价于Integer n2 = new Integer(47);
        int n3 = 47;
```

上面的n1、n2都是引用对象类型，对象的引用存储在**栈空间**中，对象的数据存储在**堆空间**中；n3是基本数据类型，数据存储在**栈空间**中。（注意，Java中的基本数据类型并不一定存储在栈中，在类中声明的变量是成员变量，也叫全局变量，是放在堆中的。）

再来理解`==`的含义，当应用`==`时：

* 比较**基本数据类型**时，比较的是**实际内容值**是否相同。
* 比较**引用数据类型**时，比较的是**堆内存地址**是否相同。

还是上面那个例子，我们来比较一下：

```java
        System.out.println(n1 == n2);
        System.out.println(n1 == n3);
        System.out.println(n2 == n3);
```

结果为：

```shell
false // n1和n2数据存储的堆空间地址根本就不一样
true  // 自动拆箱，实际上就变为两个int变量的比较
true  // 自动拆箱，实际上就变为两个int变量的比较
```

下面我们再来看`equals()`方法：

Java当中所有的类都是继承于Object这个超类的，在Object类中定义了一个equals的方法：

```java
    public boolean equals(Object obj) {
        return (this == obj);
    }
```

可见，在equals未被重写之前，equals等价于`==`运算符，初始默认行为就是比较对象的内存地址值（obj是引用数据类型），意义并不大。然而，在一些类库当中这个方法被重写了，如String、Integer、Date。在这些类当中equals有其自身的实现（一般都是用来比较对象的成员变量值是否相同），而不再是比较类在堆内存中的存放地址了。

还是上面的例子：

```java
        System.out.println(n1 == n2);		// false
        System.out.println(n1.equals(n2));	// true
```

点开equals方法：

```java
public final class Integer extends Number implements Comparable<Integer> {    
	public boolean equals(Object obj) {
        if (obj instanceof Integer) {
            return value == ((Integer)obj).intValue();
        }
        return false;
    }
}
```

可见，`java.lang.Integer`重写了Object超类中的equals，比较的不再是堆内存的存放地址，而是对象所包含的`int`值。

可以再看看`java.lang.String`的equals方法，发现功能类似：

```java
    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```

那么hashCode()方法又有什么用？

hashCode() 的作用是**获取哈希码**，也称为散列码；它实际上是返回一个int整数。这个**哈希码的作用**是确定该对象在哈希表中的索引位置。**hashCode() 在散列表中才有用，在其它情况下没用。**在散列表中hashCode() 的作用是获取对象的散列码，进而确定该对象在散列表中的位置。

 那么hashCode()与equals() 有什么联系呢？

1 没有使用在HashSet, Hashtable, HashMap等本质是散列表的数据结构中，hashCode() 和 equals()没有任何关系。

例如：

```java
public class NormalHashCodeTest {
    public static void main(String[] args) {
        Person p1 = new Person("AAA"), p2 = new Person("AAA"), p3 = new Person("BBB");
        System.out.printf("p1.equals(p2) : %s; p1(%d) p2(%d)\n", p1.equals(p2), p1.hashCode(), p2.hashCode());
        System.out.printf("p1.equals(p3) : %s; p1(%d) p3(%d)\n", p1.equals(p3), p1.hashCode(), p3.hashCode());
    }
}

class Person {
    String name;

    public Person(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                '}';
    }

    /**
     * 重写equals方法
     */
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return Objects.equals(name, person.name);
    }
}
```

运行结果：

```shell
p1.equals(p2) : true; p1(1020371697) p2(789451787)
p1.equals(p3) : false; p1(1020371697) p3(1554547125)
```

可见，**p1和p2相等的情况下，hashCode()也不一定相等。**

2 使用在HashSet, Hashtable, HashMap等本质是散列表的数据结构中。

以HashSet为例：

```java
public class ConflictHashCodeTest {
    public static void main(String[] args) {
        Person p1 = new Person("AAA"), p2 = new Person("AAA"), p3 = new Person("BBB");
        HashSet set = new HashSet();
        set.add(p1);
        set.add(p2);
        set.add(p3);
        // 比较p1 和 p2， 并打印它们的hashCode()
        System.out.printf("p1.equals(p2) : %s; p1(%d) p2(%d)\n", p1.equals(p2), p1.hashCode(), p2.hashCode());
        // 打印set
        System.out.printf("set:%s\n", set);
    }
}
```

运行结果：

```shell
p1.equals(p2) : true; p1(1020371697) p2(789451787)
set:[Person{name='AAA'}, Person{name='BBB'}, Person{name='AAA'}]
```

结果发现，HashSet中竟然有重复元素：p1 和 p2，这是因为虽然p1 和 p2的内容相等，但是它们的hashCode()不等；所以，HashSet在添加p1和p2的时候，认为它们不相等。

当我们同时覆盖equals() 和 hashCode()方法。

```java
public class ConflictHashCodeTest {
    public static void main(String[] args) {
        Person p1 = new Person("AAA"), p2 = new Person("AAA"), p3 = new Person("BBB"), p4 = new Person("aaa");
        HashSet set = new HashSet();
        set.add(p1);
        set.add(p2);
        set.add(p3);
        set.add(p4);
        // 比较p1 和 p2， 并打印它们的hashCode()
        System.out.printf("p1.equals(p2) : %s; p1(%d) p2(%d)\n", p1.equals(p2), p1.hashCode(), p2.hashCode());
        // 比较p1 和 p4， 并打印它们的hashCode()
        System.out.printf("p1.equals(p4) : %s; p1(%d) p4(%d)\n", p1.equals(p4), p1.hashCode(), p4.hashCode());
        // 打印set
        System.out.printf("set:%s\n", set);
    }
}

class Person {
    String name;

    public Person(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                '}';
    }

    /**
     * 重写equals方法
     */
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return Objects.equals(name, person.name);
    }
}
```

运行结果：

```shell
p1.equals(p2) : true; p1(64576) p2(64576)
p1.equals(p4) : false; p1(64576) p4(96352)
set:[Person{name='AAA'}, Person{name='BBB'}, Person{name='aaa'}]
```

总结

* `==`运算符 : 它的作用是**判断两个对象的地址是不是相等。**即，判断两个对象是不试同一个对象。

* equals() : 它的作用也是**判断两个对象是否相等。**但它一般有两种使用情况(前面第1部分已详细介绍过)：
  * 情况1，类没有覆盖equals()方法。则通过equals()比较该类的两个对象时，等价于通过“==”比较这两个对象。
  * 情况2，类覆盖了equals()方法。一般，我们都覆盖equals()方法来两个对象的内容相等；若它们的内容相等，则返回true(即，认为这两个对象相等)。

 * Object的 hashCode()返回该对象的内存地址编号，而equals()比较的是内存地址是否相等；
 * 需要注意的是当equals()方法被重写时，hashCode()也要被重写；
 * 按照一般hashCode()方法的实现来说，equals()相等的两个对象，hashcode()必须保持相等；equals()不相等的两个对象，hashcode()未必不相等；
 * 一个类如果要作为 HashMap 的 key，必须重写equals()和hashCode()方法。

# 参考

* https://juejin.cn/post/6844903542440853518#heading-9
* https://www.cnblogs.com/skywang12345/p/3324958.html