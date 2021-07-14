<!-- MarkdownTOC -->
- [==、equals()与hashcode()的区别和联系](#equals与hashcode的区别和联系)
  - [已经有`==`运算符了，为什么还需要equals()呢？](#已经有运算符了为什么还需要equals呢)
  - [hashCode()方法又有什么用？](#hashcode方法又有什么用)
  - [hashCode()与equals()有什么联系呢？](#hashcode与equals有什么联系呢)
  - [如何重写equals()与hashCode()方法呢？](#如何重写equals与hashcode方法呢)
    - [重写equals()](#重写equals)
    - [重写hashCode()](#重写hashcode)
  - [总结](#总结)
  - [参考资料](#参考资料)

<!-- /MarkdownTOC -->

# ==、equals()与hashcode()的区别和联系

## 已经有`==`运算符了，为什么还需要equals()呢？

在理解`==`运算符之前，我们先来明确一下基本数据类型与引用数据类型的区别：

**基本数据类型**包括 boolean（布尔型）、float（单精度浮点型）、char（字符型）、byte（字节型）、short（短整型）、int（整型）、long（长整型）和 double （双精度浮点型）共 8 种（注意，并没有String）。

**引用数据类型**建立在基本数据类型的基础上，主要包括数组、类和接口。可以这么理解，引用数据类型就是对一个对象的引用，而对象包括实例和数组两种。

也就是说，**基本数据类型变量存的是数据本身，而引用类型变量存的是保存数据的空间地址**（基本数据类型变量里存储的是直接放在抽屉里的东西；而引用数据类型变量里存储的是这个抽屉的钥匙，钥匙和抽屉一一对应）。

举个例子：

```java
    Integer n1 = new Integer(47);
    Integer n2 = 47;// 自动装箱，等价于Integer n2 = new Integer(47);
    int n3 = 47;
```

上面的n1、n2都是引用对象类型，对象的引用存储在**栈空间**中，对象的数据存储在**堆空间**中；n3是基本数据类型，数据存储在**栈空间**中（注意，Java中的基本数据类型并不一定存储在栈中，在类中声明的变量是成员变量，也叫全局变量，是放在堆中的）。

当应用`==`运算符时：

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

Java当中所有的类都是继承于Object这个超类的，在Object类中定义了一个equals()的方法：

```java
    public boolean equals(Object obj) {
        return (this == obj);
    }
```

可见，在equals()未被重写之前，equals()等价于`==`运算符，初始默认行为就是比较对象的内存地址值（obj是引用数据类型），意义并不大。然而，在一些类库当中这个方法被重写了，如String、Integer、Date等。在这些类当中equals()有其自身的实现（一般都是用来比较对象的成员变量值是否相同），而不再是比较类在堆内存中的存放地址了。

还是上面的例子：

```java
    System.out.println(n1 == n2);		// false
    System.out.println(n1.equals(n2));	// true
```

查看此处equals()源码：

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

可见，`java.lang.Integer`中重写了Object超类中的equals()方法，比较的不再是堆内存的存放地址，而是对象所包含的`int`值。

可以再看看`java.lang.String`中的equals()方法：

```java
    public boolean equals(Object anObject) {
        // 使用==操作符检查“参数是否为这个对象的引用”
        if (this == anObject) {
            return true;
        }
        // 使用instanceof操作符检查“参数是否是正确的类型”
        if (anObject instanceof String) {
            // 把参数转化为正确的类型
            String anotherString = (String)anObject;
            int n = value.length;
            // 对于该类中的“关键域”，检查参数中的域是否与对象中的对应域相等
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

## hashCode()方法又有什么用？

hashCode() 的作用是**获取哈希码**（hashcode），也称为散列码；它实际上是返回一个int整数。这个**哈希码的作用**是确定该对象在哈希表中的索引位置。在使用Object的equals()方法进行对象比较时，Java会首先计算出两个对象的hashcode并进行对比（这个过程非常简单而且节省JVM的时间），如果两个两个对象的hashcode相同，则直接判定为两对象相等。

## hashCode()与equals()有什么联系呢？

首先，我们只重写equals() 方法，将其运用在非散列表的数据结构中：

```java
public class NormalHashCodeTest {
    public static void main(String[] args) {
        Person p1 = new Person("AAA"), p2 = new Person("AAA"), p3 = new Person("BBB");
        System.out.printf("p1.equals(p2) : %s; p1(%d) p2(%d)\n"
                          , p1.equals(p2), p1.hashCode(), p2.hashCode());
        System.out.printf("p1.equals(p3) : %s; p1(%d) p3(%d)\n"
                          , p1.equals(p3), p1.hashCode(), p3.hashCode());
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

可见，**p1和p2相等的情况下，hashCode()也不一定相等。**似乎可以得出equals()与equals()没有关联的结论，但是这样设计本身就是错误的。

倘若还是只重写equals() 方法，将其运用在使HashSet, Hashtable, HashMap等本质是散列表的数据结构中，下面以HashSet为例：

```java
public class ConflictHashCodeTest {
    public static void main(String[] args) {
        Person p1 = new Person("AAA"), p2 = new Person("AAA"), p3 = new Person("BBB");
        HashSet set = new HashSet();
        set.add(p1);
        set.add(p2);
        set.add(p3);
        // 比较p1 和 p2， 并打印它们的hashCode()
        System.out.printf("p1.equals(p2) : %s; p1(%d) p2(%d)\n"
                          , p1.equals(p2), p1.hashCode(), p2.hashCode());
        // 打印set
        System.out.printf("set:%s\n", set);
    }
    class Person {
    	// 同上
    }
}
```

运行结果：

```shell
    p1.equals(p2) : true; p1(1020371697) p2(789451787)
    set:[Person{name='AAA'}, Person{name='BBB'}, Person{name='AAA'}]
```

结果发现，**HashSet中竟然有重复元素：p1 和 p2**，这是因为虽然p1 和 p2的内容相等，但是它们的hashCode()不等；所以，HashSet在添加p1和p2的时候，认为它们不相等。这样，Set集合的特性不就是失去了意义了吗？

正应《Effective Java》所说，**如果两个对象根据equals(Object)方法是相等的，那么调用这两个对象中任一个对象的hashCode方法必须产生同样的整数结果**。也就是说，当**equals()方法被重写时，hashCode()也要被重写**。

当我们同时重写equals() 和 hashCode()方法，再应用在上面HashSet的例子中：

```java
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
    
    /**
     * 重写hashCode方法
     */
    @Override
    public int hashCode() {
        return Objects.hash(name);
    }
}
```

运行结果：

```shell
    p1.equals(p2) : true; p1(64576) p2(64576)
    set:[Person{name='AAA'}, Person{name='BBB'}]
```

查看String、Integer、Date等重写了equals()方法的类，发现它们也都重写了hashCode方法，例如`java.lang.Integer`中：

```java
    @Override
    public int hashCode() {
        return Integer.hashCode(value);
    }
```

## 如何重写equals()与hashCode()方法呢？

### 重写equals() 

对象内容的比较才是设计equals()的真正目的，Java语言对equals()的要求如下，这些要求是重写该方法时必须遵循的：

- **对称性**： 如果x.equals(y)返回是true，那么y.equals(x)也应该返回是true ；
- **自反性**： x.equals(x)必须返回是true ；
- **传递性**： 如果x.equals(y)返回是true，而且y.equals(z)返回是true，那么z.equals(x)也应该返回是true ；
- **一致性**： 如果x.equals(y)返回是true，只要x和y内容一直不变，不管重复执行x.equals(y)多少次，返回都是true ；
- 任何情况下，**x.equals(null)，永远返回是“false”；x.equals(和x不同类型的对象)永远返回false**。

### 重写hashCode()

一个好的hashCode的方法的目标：**为不相等的对象产生不相等的散列码**，同样的，相等的对象必须拥有相等的散列码。下面书写一种简单的解决方法：

* A、初始化一个整形变量，为此变量赋予一个非零的常数值，比如`int result = 17`;

* B、对于对象中每个关键域`f`(**指`equals`方法中涉及的每个域**)，完成以下步骤，为该域计算`int`类型的散列码c：
  * 如果该域是**boolean**值，则计算  `f ? 1:0`；
  * 如果该域是**byte\char\short\int**，则计算 `(int)f`；
  * 如果该域是**long**值，则计算 `(int)(f ^ (f >>> 32))`；
  * 如果该域是**float**值，则计算 `Float.floatToIntBits(f)`；
  * 如果该域是**double**值，则计算 `Double.doubleToLongBits(f)`，然后返回的结果是long，再用规则(3)去处理long，得到int；
  * 如果该域是**对象引用**，如果equals方法中采取递归调用的比较方式，那么hashCode中同样采取递归调用hashCode的方式。如果需要更复杂的比较，则为这个域计算一个范式`(canonical representation)`，然后针对这个范式调用`hashCode`。如果这个域的值为`null`，则返回`0`(其他常数也行)；
  * 如果该域是**数组**，那么需要为每个元素当做单独的域来处理。`java.util.Arrays.hashCode` 方法包含了8种基本类型数组和引用数组的hashCode计算，算法同上。

* 按照下面的公式，把步骤B中计算得到的散列码`c`合并到`result`中：`result = 31 * result + c`; //此处`31`是个奇素数，并且有个很好的特性，即用移位和减法来代替乘法，可以得到更好的性能：31*i == (i<<5) - i， 现代JVM能自动完成此优化。
* 返回`result`。
* 检验并测试该`hashCode`实现是否符合通用约定。

例如：

```java
    @Override
    public int hashCode() {
        int result = 17;
        result = 31 * result + mInt;
        result = 31 * result + (mBoolean ? 1 : 0);
        result = 31 * result + Float.floatToIntBits(mFloat);
        result = 31 * result + (int) (mLong ^ (mLong >>> 32));
        long mDoubleTemp = Double.doubleToLongBits(mDouble);
        result = 31 * result + (int) (mDoubleTemp ^ (mDoubleTemp >>> 32));
        result = 31 * result + (mString == null ? 0 : mMsgContain.hashCode());
        result = 31 * result + (mObj == null ? 0 : mObj.hashCode());
        return result;
    }
```

## 总结

**==运算符和equals()方法的区别与联系**：

* ==运算符比较基本数据类型时，比较的是实际内容值是否相同；比较引用数据类型时，比较的是堆内存地址是否相同。

* equals() 方法主要用于判断两个对象的值是否相等（当然这是在equals()被重写的情况下）；未重写的equals() 方法与`==`运算符类似，比较的是对象的内存地址。
* 需要注意的是当equals()方法被重写时，hashCode()也要被重写；

**引申问题：equals()方法被重写时，hashCode()为什么也要被重写呢？**

* hashCode() 的作用是获取哈希码（hashcode），也称为散列码；它实际上是返回一个int整数。这个哈希码的作用是确定该对象在哈希表中的索引位置。在使用Object的equals()方法进行对象比较时，Java会首先计算出两个对象的hashcode并进行对比（这个过程非常简单而且节省JVM的时间），如果两个两个对象的hashcode相同，则直接判定为两对象相等。

 * 按照一般hashCode()方法的实现来说，equals()相等的两个对象，hashcode()必须保持相等；equals()不相等的两个对象，hashcode()未必不相等；
 * 因此，在使用`HashSet, Hashtable, HashMap`等等这些本质是**散列表**数据结构的集合类时，重写hashCode()是很有必要的。

## 参考资料

* 《Java编程思想》
* 《Effective Java》
* [Java hashCode() 和 equals()的若干问题解答](https://www.cnblogs.com/skywang12345/p/3324958.html)

