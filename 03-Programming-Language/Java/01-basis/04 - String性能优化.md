<!-- MarkdownTOC -->

- [String性能优化](#string性能优化)
  - [String底层实现](#string底层实现)
  - [String对象的不可变性](#string对象的不可变性)
  - [String对象的优化](#string对象的优化)
    - [如何构造超大字符串？](#如何构造超大字符串)
    - [如何使用 String.intern节省内存？](#如何使用-stringintern节省内存)
    - [如何使用字符串的分割方法？](#如何使用字符串的分割方法)
  - [思考](#思考)
  - [参考资料](#参考资料)

<!-- /MarkdownTOC -->


String对象作为Java语言中最重要的数据类型，是内存中占据空间最大的一个对象。高效地使用字符串，可以提升系统的整体性能。

# String性能优化

## String底层实现

在Java中，Sun公司工程师对String对象做了大量优化，来节省内存空间，提升 String 对象在系统中的性能。一起来看看优化过程，如下图所示：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210610094956647.png" width="600px"/>
</div>
1、**Java6及之前版本**，String对象是对char数组进行封装实现的对象，主要有四个成员变量：char 数组、偏移量 offset、字符数量 count、哈希值 hash。

* String对象是通过offset和count两个属性来定位char[]数组，获取字符串。这么做可以高效、快速地共享数组对象，同时节约内存空间，但是很有可能产生内存泄漏的问题。

2、**从Java7到Java8版本**，String 类中不再有 offset 和 count 两个变量了。这样的好处是 String 对象占用的内存稍微少了些，同时，`String.substring` 方法也不再共享 char[]，从而解决了使用该方法可能导致的内存泄漏问题。

3、**从 Java9 版本开始，**工程师将 char[] 字段改为了 byte[] 字段，又维护了一个新的属性 coder，它是一个编码格式的标识。

**为什么Java9中char[]字段改成了byte[]字段？**

- 一个char字符占16位，2个字节。JDK1.9 的String类为了节省空间，于是使用了占8位，1个字节的byte数组来存放字符串。
- 新属性 coder 的作用是，在计算字符串长度或者使用 `indexOf()`函数时，根据coder这个字段，判断如何计算字符串长度。coder 属性默认有 0 和 1 两个值，0 代表 Latin-1（单字节编码），1 代表 UTF-16。如果 String 判断字符串只包含了 Latin-1，则 coder 属性值为 0，反之则为 1。

## String对象的不可变性

在实现代码中 String 类被 final 关键字修饰了，而且变量 char 数组也被 final 修饰了。

类被 final 修饰代表该类不可继承，而 char[] 被 final+private 修饰，代表了 **String 对象不可被更改**。Java 实现的这个特性叫作 String 对象的不可变性，即 String 对象一旦创建成功，就不能再对它进行改变。

这样做的好处在哪里呢？

1. **保证String对象的安全性**。假设 String 对象是可变的，那么 String 对象将可能被恶意修改。

2. **保证hash值不会被频繁变更，确保了唯一性**，使得类似 HashMap 容器才能实现相应的 key-value 缓存功能。
3. **可以实现字符串常量池**。在 Java 中，通常有两种创建字符串对象的方式，一种是通过字符串常量的方式创建，如 String str=“abc”；另一种是字符串变量通过 new 形式的创建，如 `String str = new String(“abc”)`。

* 当代码中使用第一种方式创建字符串对象时，JVM 首先会检查该对象是否在字符串常量池中，如果在，就返回该对象引用，否则新的字符串将在常量池中被创建。这种方式可以减少同一个值的字符串对象的重复创建，节约内存。
* `String str = new String(“abc”) `这种方式，首先在编译类文件时，"abc"常量字符串将会放入到常量结构中，在类加载时，“abc"将会在常量池中创建；其次，在调用 new 时，JVM 命令将会调用 String 的构造函数，同时引用常量池中的"abc” 字符串，在堆内存中创建一个 String 对象；最后，str 将引用 String 对象。

## String对象的优化

### 如何构造超大字符串？

编程过程中，字符串的拼接很常见。前面我讲过 String 对象是不可变的，如果我们使用 String 对象相加，拼接我们想要的字符串，是不是就会产生多个对象呢？例如以下代码：

```java
    public static void test3() {
        String s1 = "a";
        String s2 = "b";
        String s3 = "ab";
        String s4 = s1 + s2;
        System.out.println(s3 == s4); // false
    }
```

查看字节码：

```java
 0 ldc #17 <a>
 2 astore_0
 3 ldc #18 <b>
 5 astore_1
 6 ldc #19 <ab>
 8 astore_2
 9 new #12 <java/lang/StringBuilder>					------（1）
12 dup
13 invokespecial #13 <java/lang/StringBuilder.<init>>
16 aload_0
17 invokevirtual #14 <java/lang/StringBuilder.append>	------（2）
20 aload_1
21 invokevirtual #14 <java/lang/StringBuilder.append>	------（3）
24 invokevirtual #15 <java/lang/StringBuilder.toString> ------（4）
27 astore_3
28 getstatic #6 <java/lang/System.out>
31 aload_2
32 aload_3
33 if_acmpne 40 (+7)
36 iconst_1
37 goto 41 (+4)
40 iconst_0
41 invokevirtual #7 <java/io/PrintStream.println>
44 return
```

可见`s1 + s2`的执行细节：

（1）`StringBuilder s = new StringBuilder()；`

（2）`s.append(s1)；`

（3）`s.append(s2)；`

（4）`s.toString();`  ——> 类似于`new String("ab")；`

**可知：即使使用 + 号作为字符串的拼接，也一样可以被编译器优化成 StringBuilder 的方式**。在编译器优化的代码中，每次循环都会生成一个新的 StringBuilder 实例，同样也会降低系统的性能。

所以平时做字符串拼接的时候，建议你还是要**显示**地使用 StringBuilder 来提升系统性能。

如果在多线程编程中，String 对象的拼接涉及到线程安全，可以使用 StringBuffer。但是要注意，由于 StringBuffer 是线程安全的，涉及到锁竞争，所以从性能上来说，要比 StringBuilder 差一些。

在JDK5之后，使用的是StringBuilder，在JDK5之前使用的是StringBuffer。

| String                                                                                                           | StringBuffer                                                                                                                                                                                                                 | StringBuilder    |
| ---------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------- |
| String的值是不可变的，这就导致每次对String的操作都会生成新的String对象，不仅效率低下，而且浪费大量优先的内存空间 | StringBuffer是可变类，和线程安全的字符串操作类，任何对它指向的字符串的操作都不会产生新的对象。每个StringBuffer对象都有一定的缓冲区容量，当字符串大小没有超过容量时，不会分配新的容量，当字符串大小超过容量时，会自动增加容量 | 可变类，速度更快 |
| 不可变                                                                                                           | 可变                                                                                                                                                                                                                         | 可变             |
|                                                                                                                  | 线程安全                                                                                                                                                                                                                     | 线程不安全       |
|                                                                                                                  | 多线程操作字符串                                                                                                                                                                                                             | 单线程操作字符串 |

注意，字符串拼接不一定使用的是StringBuilder。拼接符左右两边如果是变量的话，StringBuilder进行拼接更好；但是如果使用的是final修饰，则是从常量池中获取。所以说拼接符号左右两边都是**字符串常量或常量引用**，则仍然使用编译器优化。

- 在开发中，针对final修饰类、方法、基本数据类型时，能够使用就尽量使用。

```java
    final String s1 = "a";
    final String s2 = "b";
    String s3 = "ab";
    String s4 = s1 + s2;
    System.out.println(s3 == s4);// true
```

### 如何使用 String.intern节省内存？

这里先来看一个案例：Twitter 每次发布消息状态的时候，都会产生一个地址信息，以当时 Twitter 用户的规模预估，服务器需要 32G 的内存来存储地址信息。

```java
public class Location {
    private String city;
    private String region;
    private String countryCode;
    private double longitude;
    private double latitude;
} 
```

考虑到其中有很多用户在地址信息上是有重合的，比如，国家、省份、城市等，这时就可以将这部分信息单独列出一个类，以减少重复，代码如下：

```java
public class SharedLocation {
	private String city;
	private String region;
	private String countryCode;
}
 
public class Location {
	private SharedLocation sharedLocation;
	double longitude;
	double latitude;
}
```

通过优化，数据存储大小减到了 20G 左右。但对于内存存储这个数据来说，依然很大，怎么办呢？

这个案例来自一位 Twitter 工程师在 QCon 全球软件开发大会上的演讲，他们想到的解决方法，就是使用 String.intern 来节省内存空间，从而优化 String 对象的存储。

具体做法就是，在每次赋值的时候使用 String 的 intern 方法，如果常量池中有相同值，就会重复使用该对象，返回对象引用，这样一开始的对象就可以被回收掉。这种方式可以使重复性非常高的地址信息存储大小从 20G 降到几百兆。

```java
SharedLocation sharedLocation = new SharedLocation();
 
sharedLocation.setCity(messageInfo.getCity().intern());		sharedLocation.setCountryCode(messageInfo.getRegion().intern());
sharedLocation.setRegion(messageInfo.getCountryCode().intern());
 
Location location = new Location();
location.set(sharedLocation);
location.set(messageInfo.getLongitude());
location.set(messageInfo.getLatitude());
```

**为了更好地理解，我们再来通过一个简单的例子，回顾下其中的原理：**

```java
        String a = new String("abc").intern();
        String b = new String("abc").intern();
        if (a == b) {
            System.out.print("a==b"); // a==b
        }
```

在字符串常量中，默认会把字符串放入字符串常量池中；在字符串变量中，对象会被创建在堆内存中，同时也会在常量池中创建一个字符串对象，复制到堆对象内存中，并返回堆内存中的对象引用。

如果调用`intern()`方法，会去查看字符串常量池中是否有等于该字符串的对象，如果没有，就在常量池中新增该对象，并返回该对象引用；如果有，就返回常量池中的字符串引用。堆内存中原本的对象由于没用引用指向它，将会通过垃圾回收器回收。

了解了原理，我们再一起看看上边的例子。

在一开始创建 a 变量时，会在堆内存中创建一个对象，同时会在加载类时，在常量池中创建一个字符串对象，在调用 intern 方法之后，会去常量池中查找是否有等于该字符串的对象，有就返回引用。

在创建 b 字符串变量时，也会在堆中创建一个对象，此时常量池中有该字符串对象，就不再创建。调用 intern 方法则会去常量池中判断是否有等于该字符串的对象，发现有等于"abc"字符串的对象，就直接返回引用。而在堆内存中的对象，由于没有引用指向它，将会被垃圾回收。所以 a 和 b 引用的是同一个对象。

下面我用一张图来总结下 String 字符串的创建分配内存地址情况：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210610101200927.png" width="1000px"/>
</div>
使用 intern 方法需要注意的一点是，一定要结合实际场景。因为常量池的实现是类似于一个 HashTable 的实现方式，HashTable 存储的数据越大，遍历的时间复杂度就会增加。如果数据过大，会增加整个字符串常量池的负担。

### 如何使用字符串的分割方法？

Split() 方法使用了正则表达式实现了其强大的分割功能，而正则表达式的性能是非常不稳定的，使用不恰当会引起回溯问题，很可能导致 CPU 居高不下。

所以我们应该慎重使用 Split() 方法，我们可以用 `String.indexOf()` 方法代替 Split() 方法完成字符串的分割。如果实在无法满足需求，你就在使用 Split() 方法时，对回溯问题加以重视就可以了。

## 思考

**思考一**：String、StringBuffer、StringBuilder有什么区别？

**解答**：String是Java中非常基础而又重要的类，它提供了构造和处理字符串的各种逻辑。String类是典型的immutable类，被声明为final class，所以的属性都是不可变的。也正是也由于它的不可变性，类似拼接、裁剪字符串等动作，都会产生新的String对象。这些频繁的操作往往对应用性能有很大的影响。

StringBuffe是为了解决String拼接产生太多中间对象的问题而提供的一个类，我们可以用 append 或者 add 方法，把字符串添加到已有序列的末尾或者指定位置。StringBuffer 本质是一个线程安全的可修改字符序列，它保证了线程安全，也随之带来了额外的性能开销，所以除非有线程安全的需要，不然还是推荐使用它的后继者，也就是 StringBuilder。

StringBuilder是Java 1.5中新增的，在能力上和 StringBuffer 没有本质区别，但是它去掉了线程安全的部分，有效减小了开销，是绝大部分情况下进行字符串拼接的首选。

> 效率：StringBuilder > StringBuffer > String

**StringBuffer的初始容量？扩容机制呢？**

- StringBuffer无参构造时，初始容量默认为16；有参构造时，初始容量为传入字符串的length + 16。
- `StringBuffer`和`StringBuilder`都继承了`AbstractStringBuilder`，它们的底层char数组`value`默认的初始化容量是**16**，扩容只需要修改底层的char数组，两者的扩容最终都会调用到`AbstractStringBuilder`类相同的方法：

1. 新的字符串的长度超过了底层原char数组`value`的大小，才需要进行扩容；先尝试默认扩容，将新容量变成 (**value**.**length** << 1) + 2 ，也就是**两倍的原数组长度+2**；
2. 若默认扩充后的值还是小于至少容量的值，直接扩充到当前需要的至少容量大小；
3. 经过前两步骤确定的新数组大小，若大于Interger.MAX_VALUE，则报异常，若小于等于0，则新数组大小改为`Interger.MAX_VALUE -8`。
4. 确定了新数组的值后，通过`Arrays.copy(value,newCapactity)`进行复制。
5. 最终给value数组完成扩容。

------

**思考二**：通过三种不同的方式创建了三个对象，再依次两两匹配，每组被匹配的两个对象是否相等？代码如下：

```java
String str1= "abc";
String str2= new String("abc");
String str3= str2.intern();
assertSame(str1==str2);
assertSame(str2==str3);
assertSame(str1==str3)
```

**解答**：

`String str1 = "abc";`通过字面量的方式创建，abc存储于字符串常量池中；
`String str2 = new String("abc");`通过new对象的方式创建字符串对象，引用地址存放在堆内存中，abc则存放在字符串常量池中；所以str1 == str2显然是false。
`String str3 = str2.intern();`由于str2调用了intern()方法，会返回常量池中的数据，地址直接指向常量池，所以str1 == str3返回true；而str2和str3地址值不等，所以str2==str3返回也是false（str2指向堆空间，str3直接指向字符串常量池）。

------

**思考三**：new String("ab")会创建几个对象（JDK8）

```java
public class StringNewTest {
    public static void main(String[] args) {
        String str = new String("ab");
    }
}
```

**解答**：查看字节码文件

```java
 0 new #2 <java/lang/String>					------（1）
 3 dup
 4 ldc #3 <ab>									------（2）
 6 invokespecial #4 <java/lang/String.<init>>
 9 astore_1
10 return
```

可见会创建两个对象：

- 一个对象是：new关键字在堆空间中创建的对象。
- 另一个对象：字符串常量池（JDK 8 字符串常量池也在堆中）中的对象。

------

**思考四**：new String("a") + new String("b") 会创建几个对象（JDK8）

```java
public class StringNewTest {
    public static void main(String[] args) {
        String str = new String("a") + new String("b");
    }
}
```

**解答**：查看字节码文件

```java
 0 new #2 <java/lang/StringBuilder>						------（1）
 3 dup
 4 invokespecial #3 <java/lang/StringBuilder.<init>>
 7 new #4 <java/lang/String>							------（2）
10 dup
11 ldc #5 <a>											------（3）
13 invokespecial #6 <java/lang/String.<init>>
16 invokevirtual #7 <java/lang/StringBuilder.append>
19 new #4 <java/lang/String>							------（4）
22 dup
23 ldc #8 <b>											------（5）
25 invokespecial #6 <java/lang/String.<init>>
28 invokevirtual #7 <java/lang/StringBuilder.append>	
31 invokevirtual #9 <java/lang/StringBuilder.toString>	------（6）
34 astore_1
35 return
```

这里创建了5个对象：

- 对象1：new StringBuilder()
- 对象2：new String("a")
- 对象3：常量池的 a
- 对象4：new String("b")
- 对象5：常量池的 b

查看`java.lang.StringBuilder`的`toString`方法字节码文件：

```java
 0 new #80 <java/lang/String>
 3 dup
 4 aload_0
 5 getfield #234 <java/lang/StringBuilder.value>
 8 iconst_0
 9 aload_0
10 getfield #233 <java/lang/StringBuilder.count>
13 invokespecial #291 <java/lang/String.<init>>
16 areturn
```

可见toString中会创建一个 new String("ab")

- 但**不会在常量池中生成ab**，并没有`ldc <ab>`

------

**思考五**：下面一段代码在JDK6和JDK7/8运行有什么区别？

```java
String s = new String("a") + new String("b");
String s2 = s.intern();
System.out.println(s2 == "ab");
System.out.println(s == "ab");
```

**解答**：JDK6中，字符串常量池及静态变量都存放与方法区中，JDK 7/ JDK 8都存放于堆中。

在JDK6中，在字符串常量池中创建一个字符串 “ab”：

<img src="https://img-blog.csdnimg.cn/20201013232312678.png" style="zoom: 67%;" />

在JDK8中，在字符串常量池中没有创建 “ab”，而是将堆中的地址复制到串池中：

<img src="https://img-blog.csdnimg.cn/20201013232533265.png" style="zoom: 67%;" />

所以上述结果，在JDK6中是：

```
true
false
```

在JDK8中是：

```
true
true
```

题目改成这样，在JDK6和8中结果都是一样的：

```java
String x = "ab";
String s = new String("a") + new String("b");
String s2 = s.intern();
System.out.println(s2 == "ab");// true		
System.out.println(s == "ab");// false
```

<img src="https://img-blog.csdnimg.cn/20201013232834642.png" style="zoom:67%;" />

## 参考资料

* [字符串性能优化不容小觑，百M内存轻松存储几十G数据](https://time.geekbang.org/column/article/97215)

