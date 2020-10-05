## 一、基础

### 1.1 关键字

Java 语言目前定义了 51 个关键字，这些**关键字不能作为变量名、类名和方法名**来使用。

- 数据类型：boolean、int、long、short、byte、float、double、char、class、interface
- 流程控制：if、else、do、while、for、switch、case、default、break、continue、return、try、catch、finally
- 修饰符：public、protected、private、final、void、static、strict、abstract、transient、synchronized、volatile、native
- 动作：package、import、throw、throws、extends、implements、this、supper、instanceof、new
- 保留字：true、false、null、goto、const

### 1.2 标识符

- 标识符由**数字**（0-9）和**字母**（A-Z和 a-z）、**美元符号**（$）、**下划线**（_）以及 Unicode 字符集中符号大于 0xC0 的所有符号组合构成（各符号之间没有空格）。
- 标识符的第一个符号为字母、下划线和美元符号，后面可以是任何字母、数字、美元符号或下划线。
- 标识符区分大小写。

### 1.3 注释

```java
/**
 * @author: Kai
 * @version jdk1.8.0
 * 文档注释
 */
public class HelloWorld {

    public static void main(String[] args) {
        /**
         * 多行注释
         * 可以注释多行类容
         */
        System.out.println("HelloWorld");// 单行注释
    }
}
```

文档注释与多行注释的区别：

- 文档注释的内容可以生成一个API 帮助文档。

```shell
D:\workplace\IdeaProjects\java-learn\basic\com\kai>javadoc -encoding UTF-8 -charset UTF-8  HelloWorld.java
Picked up JAVA_TOOL_OPTIONS: -Dfile.encoding=gb2312
正在加载源文件HelloWorld.java...
正在构造 Javadoc 信息...
标准 Doclet 版本 1.8.0_261
正在构建所有程序包和类的树...
正在生成.\com\kai\HelloWorld.html...
正在生成.\com\kai\package-frame.html...
正在生成.\com\kai\package-summary.html...
正在生成.\com\kai\package-tree.html...
正在生成.\constant-values.html...
正在构建所有程序包和类的索引...
正在生成.\overview-tree.html...
正在生成.\index-all.html...
正在生成.\deprecated-list.html...
正在构建所有类的索引...
正在生成.\allclasses-frame.html...
正在生成.\allclasses-noframe.html...
正在生成.\index.html...
正在生成.\help-doc.html...
```

打开 HelloWorld.java 文件存储的位置，会发现多出了一个 HelloWorld.html 文档。

![](https://img-blog.csdnimg.cn/20200924223305625.png)

### 1.4 变量

1）成员变量：

- 成员变量定义在类中，在整个类中都可以被访问。
- 成员变量随着对象的建立而建立，随着对象的消失而消失，存在于对象所在的堆内存中。
- 成员变量有默认初始化值。

2）局部变量：

- 局部变量只定义在局部范围内，如：函数内，语句内等，只在所属的区域有效。
- 局部变量存在于栈内存中，作用的范围结束，变量空间会自动释放。
- 局部变量没有默认初始化值

 在使用变量时需要遵循的原则为：**就近原则**（ 首先在局部范围找，有就使用；接着在成员位置找）

3）静态变量

- 由**static**修饰的变量，实质上即是一个全局变量。
- 其生命周期取决于类的生命周期。类被垃圾回收机制彻底回收时才会被销毁。

### 1.5 数据类型

| 类型名称     | 关键字  | 占用内存 | 取值范围                                   |
| ------------ | ------- | -------- | ------------------------------------------ |
| 字节型       | byte    | 1 字节   | -128~127                                   |
| 短整型       | short   | 2 字节   | -32768~32767                               |
| 整型         | int     | 4字节    | -2147483648~2147483647                     |
| 长整型       | long    | 8字节    | -9223372036854775808L~9223372036854775807L |
| 单精度浮点型 | float   | 4字节    | +/-3.4E+38F（6~7 个有效位）                |
| 双精度浮点型 | double  | 8 字节   | +/-1.8E+308 (15 个有效位）                 |
| 字符型       | char    | 2 字节   | ISO 单一字符集                             |
| 单元格       | boolean | 1 字节   | true 或 false                              |

### 1.6 运算符

1、自增与自减
前缀自增自减法(`++i`，`--i`)：先进行自增或者自减运算，再进行表达式运算。
后缀自增自减法(`i++`，`i--`)：先进行表达式运算，再进行自增或者自减运算。

2、逻辑运算符

| 运算符 | 用法   | 含义   | 说明                                                 |
| ------ | ------ | ------ | ---------------------------------------------------- |
| &&     | a&&b   | 短路与 | ab 全为 true 时，计算结果为 true，否则为 false。     |
| `||`   | a`||`b | 短路或 | ab 全为 false 时，计算结果为 false，否则为 true。    |
| !      | !a     | 逻辑非 | a 为 true 时，值为 false，a 为 false 时，值为 true。 |
| `|`    | a`|`b  | 逻辑或 | ab 全为 false 时，计算结果为 false，否则为 true 。   |
| &      | a&b    | 逻辑非 | ab 全为 true 时，计算结果为 true，否则为 false。     |

&& 与 & 区别：如果 a 为 false，则不计算 b（因为不论 b 为何值，结果都为 false）
|| 与 | 区别：如果 a 为 true，则不计算 b（因为不论 b 为何值，结果都为 true）

因此，**短路与**（&&）和**短路或**（||）的计算方式更优化，在实际编程时，应该优先考虑使用短路与和短路或。

3、三目运算符（条件运算符）

```java
(x>y) ? (x-=y): (x+=y);
```

4、`instanceof `运算符
该运算符用于操作对象实例，检查该对象是否是一个特定类型（类类型或接口类型）。

```java
class Animal{} 
public class Dog extends Animal{
   public static void main(String[] args){
      Animal a = new Dog();
      boolean result =  a instanceof Dog;
      System.out.println(result); // true
   }
}
```

... ...

### 1.7 流程控制语句

1、`foreach`语句
语法格式如下：

```java
for(类型 变量名:集合) {
    语句块;
}
```

2、`continue` 关键字
在 for 循环中，continue 语句使程序立即跳转到更新语句。
在 while 或者 do…while 循环中，程序立即跳转到布尔表达式的判断语句。

... ...

### 1.8 数组

1、声明数组：

```java
type[] arrayName;    // 数据类型[] 数组名;
type[][] arrayName;    // 数据类型 数组名[][];
```

2、分配空间：
声明了数组，只是得到了一个存放数组的变量，并没有为数组元素分配内存空间，不能使用。
在 Java 中可以使用 **new 关键字**来给数组分配空间。

```java
arrayName = new type[size];    // 数组名 = new 数据类型[数组长度];
```

<font color="red" >注意：一旦声明了数组的大小，就不能再修改。</font>

3、数组初始化：

- 使用 new 指定数组大小后进行初始化：

```java
type[] arrayName = new int[size];
arrayName[0] = 值1;
--- snip ---
arrayName[n] = 值n;
```

- 使用 new 指定数组元素的值

```java
type[] arrayName = new type[]{值 1,值 2,值 3,值 4,• • •,值 n};
```

- 直接指定数组元素的值

```java
type[] arrayName = {值 1,值 2,值 3,...,值 n};
```

## 二、Object类

`java.lang.Object`类是Java语言中的根类，即所有类的父类。它中描述的所有方法子类都可以使用。在对象实例化的时候，最终找的父类就是Object。

如果一个类没有特别指定父类，那么默认则继承自Object类。例如：

```java
public class MyClass /*extends Object*/ {
  	// ...
}
```

根据JDK源代码及Object类的API文档，Object类当中包含的方法有11个。今天学习其中的3个：

* `public String toString()`：返回该对象的字符串表示。
* `public boolean equals(Object obj)`：指示其他某个对象是否与此对象“相等”。
* `public static int hashCode(Object o)`：用于返回字符串的哈希码。

### 2.1 toString()

`toString`返回该对象的字符串表示，其实该字符串内容就是对象的类型+@+**内存地址值**。

由于`toString`方法返回的结果是内存地址，而在开发中，经常需要按照对象的属性得到相应的字符串表现形式，因此也需要重写它。

例如：

```java
public class Person {  
    private String name;
    private int age;

    @Override
    public String toString() {
        return "Person{" + "name='" + name + '\'' + ", age=" + age + '}';
    }

    // Getter/Setter方法
}
```

在IntelliJ IDEA中，可以点击`Code`菜单中的`Generate...`，也可以使用快捷键`alt+insert`，点击`toString()`选项，选择需要包含的成员变量并确定。

> Tips：在我们直接使用输出语句输出对象名的时候，其实通过该对象调用了其toString()方法。

### 2.2 equals()

Object类中的实例方法，调用成员方法equals()并指定参数为另一个对象，则可以判断这两个对象是否是相同的。这里的“相同”有默认和重写两种方式。

**1）默认**

如果没有覆盖重写equals方法，那么Object类中默认进行`==`运算符的对象地址比较，只要不是同一个对象，结果必然为false。

**2）重写**

如果对象的内容比较，即所有或指定的部分成员变量相同就判定两个对象相同，则可以覆盖重写equals方法。例如：

```java
import java.util.Objects;

public class Person {	
	private String name;
	private int age;
	
    @Override
    public boolean equals(Object o) {
        // 如果对象地址一样，则认为相同
        if (this == o)
            return true;
        // 如果参数为空，或者类型信息不一样，则认为不同
        if (o == null || getClass() != o.getClass())
            return false;
        // 转换为当前类型
        Person person = (Person) o;
        // 要求基本类型相等，并且将引用类型交给java.util.Objects类的equals静态方法取用结果
        return age == person.age && Objects.equals(name, person.name);
    }
}
```

### 3.3 hashCode()

Object类中的native方法 , 获取对象的哈希值，用于确定该对象在哈希表中的索引位置，它实际上是一个int型整数。

例如：

```java
public class Test {
    public static void main(String args[]) {
        String Str = new String("HelloWorld!");
        System.out.println("字符串的哈希码为 :" + Str.hashCode() );
    }
}
```

执行结果为：

```java
字符串的哈希码为 :734305825
```

## 三、日期时间类

### 3.1 Date类

` java.util.Date`类表示特定的瞬间，精确到毫秒。

- `public Date()`：分配Date对象并初始化此对象，以表示分配它的时间（精确到毫秒）。
- `public Date(long date)`：分配Date对象并初始化此对象，以表示自从标准基准时间（称为“历元（epoch）”，即1970年1月1日00:00:00 GMT）以来的指定毫秒数。

> Tips： 中国处于东八区，基准时间为1970年1月1日8时0分0秒。

简单来说：使用无参构造，可以自动设置当前系统时间的毫秒时刻；指定long类型的构造参数，可以自定义毫秒时刻。例如：

```java
public class DateTest {
    public static void main(String[] args) {
        // 创建日期对象，把当前的时间
        System.out.println(new Date()); // Thu Sep 24 23:12:27 CST 2020
        // 创建日期对象，把当前的毫秒值转成日期对象
        System.out.println(new Date(0L)); // Thu Jan 01 08:00:00 CST 1970
    }
}
```

> Tips：在使用println方法时，会自动调用Date类中的toString方法。Date类对Object类中的toString方法进行了覆盖重写，所以结果为指定格式的字符串。

### 3.2 DateFormat类

`java.text.DateFormat` 是日期/时间格式化子类的抽象类，可以帮我们完成日期和文本之间的转换，也就是可以在Date对象与String对象之间进行来回转换。

* **格式化**：按照指定的格式，从Date对象转换为String对象。
* **解析**：按照指定的格式，从String对象转换为Date对象。



1）构造方法

DateFormat为抽象类，不能直接使用，所以需要常用的子类`java.text.SimpleDateFormat`。这个类需要一个模式（格式）来指定格式化或解析的标准。构造方法为：

* `public SimpleDateFormat(String pattern)`：用给定的模式和默认语言环境的日期格式符号构造`SimpleDateFormat`。

参数pattern是一个字符串，代表日期时间的自定义格式。

2）格式规则

| 标识字母（区分大小写） | 含义 |
| ---------------------- | ---- |
| y                      | 年   |
| M                      | 月   |
| d                      | 日   |
| H                      | 时   |
| m                      | 分   |
| s                      | 秒   |

> Tips：更详细的格式规则，可以参考SimpleDateFormat类的API文档。

`DateFormat`类的常用方法有：

- `public String format(Date date)`：将Date对象格式化为字符串。
- `public Date parse(String source)`：将字符串解析为Date对象。

使用format()实例：

```java
/*
 把Date转换成String对象
*/
public class SimpleDateFormatTest {
    public static void main(String[] args) {
        // 对应的日期格式如：2020-09-24 23:23:43
        Date date = new Date();
        DateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println(sdf.format(date));
    }
}
```

使用parse()实例：

```java
/*
 把String转换成Date对象
*/
public class SimpleDateFormatTest {

    public static void main(String[] args) throws ParseException {
        DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        Date date = df.parse("2018-12-11 00:00:00");
        // Tue Dec 11 00:00:00 CST 2018
        System.out.println(date);
    }
}
```

### 3.3 Calendar类

`java.util.Calendar`是日历类，将所有可能用到的时间信息封装为静态成员变量，更方便获取各个时间属性的。

Calendar为抽象类，Calendar类是通过静态方法创建对象，返回子类对象：

* `public static Calendar getInstance()`：使用默认时区和语言环境获得一个日历

例如：

```java
public class CalendarTest {
    public static void main(String[] args) {
        Calendar cal = Calendar.getInstance();
    }
}
```



**1）常用方法：**

根据Calendar类的API文档，常用方法有：

- `public int get(int field)`：返回给定日历字段的值。
- `public void set(int field, int value)`：将给定的日历字段设置为给定值。
- `public abstract void add(int field, int amount)`：根据日历的规则，为给定的日历字段添加或减去指定的时间量。
- `public Date getTime()`：返回一个表示此Calendar时间值（从历元到现在的毫秒偏移量）的Date对象。

Calendar类中提供很多成员常量，代表给定的日历字段：

| 字段值       | 含义                                  |
| ------------ | ------------------------------------- |
| YEAR         | 年                                    |
| MONTH        | 月（从0开始，可以+1使用）             |
| DAY_OF_MONTH | 月中的天（几号）                      |
| HOUR         | 时（12小时制）                        |
| HOUR_OF_DAY  | 时（24小时制）                        |
| MINUTE       | 分                                    |
| SECOND       | 秒                                    |
| DAY_OF_WEEK  | 周中的天（周几，周日为1，可以-1使用） |

**2）get/set方法**

get方法用来获取指定字段的值，set方法用来设置指定字段的值，代码使用演示：

```java
public class CalendarTest {
    public static void main(String[] args) {
        Calendar cal = Calendar.getInstance();
        // 获得年份
        int year = cal.get(Calendar.YEAR);
        // 获得月份
        int month = cal.get(Calendar.MONTH) + 1;
        // 获得天数
        int dayOfMonth = cal.get(Calendar.DAY_OF_MONTH);
        System.out.print(year + "-" + month + "-" + dayOfMonth);
    }
}
```

```java
CalendarTestimport java.util.Calendar;

public class CalendarTest {
    public static void main(String[] args) {
        Calendar cal = Calendar.getInstance();
        cal.set(Calendar.YEAR, 2020);
    }
}
```

3）add方法

add方法可以对指定日历字段的值进行加减操作，如果第二个参数为正数则加上偏移量，如果为负数则减去偏移量。代码如：

```java
cal.add(Calendar.DATE, 10);
cal.add(Calendar.DATE, -10);
```

4）getTime方法

Calendar中的getTime方法并不是获取毫秒时刻，而是拿到对应的Date对象。

```java
public class CalendarTest {
    public static void main(String[] args) {
        Calendar cal = Calendar.getInstance();
        Date date = cal.getTime();
        System.out.println(date); // Fri Sep 25 00:00:55 CST 2020
    }
}
```

> Tips：
>
> ​     西方星期的开始为周日，中国为周一。
>
> ​     在Calendar类中，月份的表示是以0-11代表1-12月。
>
> ​     日期是有大小关系的，时间靠后，时间越大。

## 四、System类

`java.lang.System`类中提供了大量的静态方法，可以获取与系统相关的信息或系统级操作，在System类的API文档中，常用的方法有：

- `public static long currentTimeMillis()`：返回以毫秒为单位的当前时间。
- `public static void arraycopy(Object src, int srcPos, Object dest, int destPos, int length)`：将数组中指定的数据拷贝到另一个数组中。

### 4.1 currentTimeMillis方法

实际上，currentTimeMillis方法就是 获取当前系统时间与1970年01月01日00:00点之间的毫秒差值

```java
public class CurrentTimeMillisTest {
    public static void main(String[] args) {
        //获取当前时间毫秒值
        System.out.println(System.currentTimeMillis()); // 1600963403083
    }
}
```

**应用：**验证for循环打印数字1-9999所需要使用的时间（毫秒）

~~~java
public class Test {
    public static void main(String[] args) {
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10000; i++) {
            System.out.println(i);
        }
        long end = System.currentTimeMillis();
        System.out.println("共耗时毫秒：" + (end - start));
    }
}
~~~

### 4.2 arraycopy方法

数组的拷贝动作是系统级的，性能很高。System.arraycopy方法具有5个参数，含义分别为：

| 参数序号 | 参数名称 | 参数类型 | 参数含义             |
| -------- | -------- | -------- | -------------------- |
| 1        | src      | Object   | 源数组               |
| 2        | srcPos   | int      | 源数组索引起始位置   |
| 3        | dest     | Object   | 目标数组             |
| 4        | destPos  | int      | 目标数组索引起始位置 |
| 5        | length   | int      | 复制元素个数         |

实例：

将src数组中前3个元素，复制到dest数组的前3个位置上复制元素前。其中src数组元素[1,2,3,4,5]，dest数组元素[6,7,8,9,10]。那么复制元素后：src数组元素[1,2,3,4,5]，dest数组元素[1,2,3,9,10]

```java
public class ArrayCopyTest {
    public static void main(String[] args) {
        int[] src = new int[]{1, 2, 3, 4, 5};
        int[] dest = new int[]{6, 7, 8, 9, 10};
        System.arraycopy(src, 0, dest, 0, 3);
    }
}
```

## 五、StringBuilder类

### 5.1 字符串拼接问题

由于String类的对象内容不可改变，所以每当进行字符串拼接时，总是会在内存中创建一个新的对象。例如：

~~~java
public class StringDemo {
    public static void main(String[] args) {
        String s = "Hello";
        s += "World";
        System.out.println(s);
    }
}
~~~

在API中对String类有这样的描述：字符串是常量，它们的值在创建后不能被更改。

根据这句话分析我们的代码，其实总共产生了三个字符串，即`"Hello"`、`"World"`和`"HelloWorld"`。引用变量s首先指向`Hello`对象，最终指向拼接出来的新字符串对象，即`HelloWord` 。

由此可知，如果对字符串进行拼接操作，每次拼接，都会构建一个新的String对象，既耗时，又浪费空间。为了解决这一问题，可以使用`java.lang.StringBuilder`类。

### 5.2 StringBuilder概述

查阅`java.lang.StringBuilder`的API，StringBuilder又称为可变字符序列，它是一个类似于 String 的字符串缓冲区，通过某些方法调用可以改变该序列的长度和内容。

原来StringBuilder是个字符串的缓冲区，即它是一个容器，容器中可以装很多字符串。并且能够对其中的字符串进行各种操作。

它的内部拥有一个数组用来存放字符串内容，进行字符串拼接时，直接在数组中加入新内容。StringBuilder会自动维护数组的扩容。原理如下图所示：(默认16字符空间，超过自动扩充)

### 5.3 构造方法

根据StringBuilder的API文档，常用构造方法有2个：

- `public StringBuilder()`：构造一个空的StringBuilder容器。
- `public StringBuilder(String str)`：构造一个StringBuilder容器，并将字符串添加进去。

```java
public class StringBuilderTest {
    public static void main(String[] args) {
        StringBuilder sb1 = new StringBuilder();
        System.out.println(sb1); // (空白)
        // 使用带参构造
        StringBuilder sb2 = new StringBuilder("kai");
        System.out.println(sb2); // kai
    }
}
```

### 5.4 常用方法

StringBuilder常用的方法有2个：

- `public StringBuilder append(...)`：添加任意类型数据的字符串形式，并返回当前对象自身。
- `public String toString()`：将当前StringBuilder对象转换为String对象。

1）append方法

append方法具有多种重载形式，可以接收任意类型的参数。任何数据作为参数都会将对应的字符串内容添加到StringBuilder中。例如：

```java
public class AppendTest {
    public static void main(String[] args) {
        //创建对象
        StringBuilder builder = new StringBuilder();
        //public StringBuilder append(任意类型)
        StringBuilder builder2 = builder.append("hello");
        //对比
        System.out.println("builder:"+builder);
        System.out.println("builder2:"+builder2);
        System.out.println(builder == builder2); //true
        // 可添加任何类型
        builder.append("!@#$%^&*");
        builder.append("````");
        builder.append(true);
        builder.append(2345);
        //链式编程
        builder.append("hello").append("world").append(true).append(100);
        System.out.println("builder:"+builder);
    }
}
```

> Tips：StringBuilder已经覆盖重写了Object当中的toString方法。

2）toString方法

通过toString方法，StringBuilder对象将会转换为不可变的String对象。如：

```java
public class ToStringTest {
    public static void main(String[] args) {
        // 链式创建
        StringBuilder sb = new StringBuilder("Hello").append("World").append("Java");
        // 调用方法
        String str = sb.toString();
        System.out.println(str); // HelloWorldJava
    }
}
```

## 六、字符串处理

<font color="red">字符串变量必须经过初始化才能使用。</font>

### 6.1 String字符串和整型int的相互转换

1）String转换为int

- Integer.parseInt(str)
- Integer.valueOf(str).intValue()

2）int转换为String

- String s = String.valueOf(i);
- String s = Integer.toString(i);
- String s = "" + i;

3）valueOf() 、parse()和toString()

- valueOf() 它是一种静态方法，它将任何对象转换成字符串类型。

```java
static String valueOf(double num)
static String valueOf(long num)
static String valueOf(Object ob)
static String valueOf(char chars[])
static String valueOf(char chars[ ], int startIndex, int numChars)
```

- parse()
  parseXxx(String) 这种形式，是指把字符串转换为数值型，其中 Xxx 对应不同的数据类型，然后转换为 Xxx 指定的类型，如 int 型和 float 型。

- toString()
  toString() 可以把一个引用类型转换为 String 字符串类型。

### 6.2 常用方法

1）字符串大小写转换

```java
字符串名.toLowerCase()    // 将字符串中的字母全部转换为小写，非字母不受影响
字符串名.toUpperCase()    // 将字符串中的字母全部转换为大写，非字母不受影响
```

2）去除字符串中的首尾空格

```java
字符串名.trim()
```

3）截取（提取）子字符串

```java
substring(int beginIndex)
substring(int beginIndex，int endIndex)
```

4）分割字符串

```java
//str 待分割的目标字符串
//sign 分割符，可为任意字符串
//limit 分割后生成的字符串的限制个数；如果不指定，则表示不限制，直到将整个目标字符串完全分割为止
str.split(String sign)
str.split(String sign,int limit)
```

<font color="red">注意：</font>

- “.”和“|”都是转义字符，分隔符前必须得加“\\”;
- 多个分隔符，可以用“|”作为连字符，比如：“acount=? and uu =? or n=?”，把三个都分隔出来，可以用String.split("and|or")。
  ... ...

### 6.3 String、StringBuffer和StringBuilder

![](https://img-blog.csdnimg.cn/20200806103642453.png)
StringBuilder与StringBuffer是可变字符串类。一般情况下，速度从快到慢为 StringBuilder > StringBuffer > String。
操作少量的数据使用 String；单线程操作大量数据使用 StringBuilder；多线程操作大量数据使用 StringBuffer。

## 七、内置包装类

| 序号 | 基本数据类 | 包装类        |
| ---- | ---------- | ------------- |
| 1    | byte       | Byte          |
| 2    | short      | Short         |
| 3    | int        | **Integer**   |
| 4    | long       | Long          |
| 5    | char       | **Character** |
| 6    | float      | Float         |
| 7    | double     | Double        |
| 8    | boolean    | Boolean       |

### 7.1 装箱与拆箱

基本类型与对应的包装类对象之间，来回转换的过程称为”装箱“与”拆箱“：

* **装箱**：从基本类型转换为对应的包装类对象。

* **拆箱**：从包装类对象转换为对应的基本类型。

  

用Integer与 int为例：

基本数值---->包装对象

~~~shell script
Integer i = new Integer(4);//使用构造函数函数
Integer iii = Integer.valueOf(4);//使用包装类中的valueOf方法
~~~

包装对象---->基本数值

~~~shell script
int num = i.intValue();
~~~

### 7.2 自动装箱与自动拆箱

从Java 5（JDK 1.5）开始，基本类型与包装类的装箱、拆箱动作可以自动完成。例如：

```shell script
Integer i = 4;//1.自动装箱------>Integer i = Integer.valueOf(4);
i = i + 5;    //2.自动拆箱:等号右边将i对象转成基本数值------> i.intValue() + 5;
		      //3.自动装箱:加法运算完成后，再次装箱，把基本数值转成对象。
```

### 7.3 基本类型与字符串之间的转换

**基本类型转换String**总共有三种方式，查看课后资料可以得知，这里只讲最简单的一种方式： 

~~~
基本类型直接与""相连接即可；如：34+""
~~~

String转换成对应的基本类型 

除了Character类之外，其他所有包装类都具有parseXxx静态方法可以将字符串参数转换为对应的基本类型：

- `public static byte parseByte(String s)`：将字符串参数转换为对应的byte基本类型。
- `public static short parseShort(String s)`：将字符串参数转换为对应的short基本类型。
- `public static int parseInt(String s)`：将字符串参数转换为对应的int基本类型。
- `public static long parseLong(String s)`：将字符串参数转换为对应的long基本类型。
- `public static float parseFloat(String s)`：将字符串参数转换为对应的float基本类型。
- `public static double parseDouble(String s)`：将字符串参数转换为对应的double基本类型。
- `public static boolean parseBoolean(String s)`：将字符串参数转换为对应的boolean基本类型。

以Integer类的静态方法parseXxx为例：

```java
public class Demo18WrapperParse {
    public static void main(String[] args) {
        int num = Integer.parseInt("100");
    }
}
```

> Tips：如果字符串参数的内容无法正确转换为对应的基本类型，则会抛出`java.lang.NumberFormatException`异常。

### 7.4 疑问解答

==1）为什么存在基本类型与包装类型这两种类型呢？==
在Java语言中，new一个对象存储在堆里，我们通过栈中的引用来使用这些对象；但是对于经常用到的一系列类型如int，如果我们用new将其存储在堆里就不是很有效——特别是简单的小的变量。所以就出现了基本类型，同C++一样，Java采用了相似的做法，对于这些类型不是用new关键字来创建，而是直接将变量的值存储在栈中，因此更加高效。

==2）有了基本类型为什么还要有包装类型呢？==
Java是一个面相对象的编程语言，基本类型并不具有对象的性质，为了让基本类型也具有对象的特征，就出现了包装类型（如我们在使用集合类型Collection时就一定要使用包装类型而非基本类型），它相当于将基本类型“包装起来”，使得它具有了对象的性质，并且为其添加了属性和方法，丰富了基本类型的操作。

另外，使用ArrayList，HashMap中时，像int，double等基本类型是放不进去的，因为容器都是装object的，这时就需要这些基本类型的包装类了。

==3）二者的主要区别在哪里呢？==

1. 声明方式不同：
   基本类型不使用new关键字，而包装类型需要使用new关键字来在堆中分配存储空间

2. 存储方式及位置不同： 
   基本类型是直接将变量值存储在栈中，而包装类型是将对象放在堆中，然后通过引用来使用；

3. 初始值不同：
   基本类型的初始值如int为0，boolean为false，而包装类型的初始值为null

4. 使用方式不同：

   基本类型直接赋值直接使用就好，而包装类型在集合如Collection、Map时会使用到。

## 参考资料

[浅析Java程序的执行过程](https://juejin.im/post/6844903775056953352)

[equals与hashCode详解](https://www.cnblogs.com/qian123/p/5703507.html)

[equals()、compareTo() 与==](https://blog.csdn.net/u012369153/article/details/52982022)

[内置包装类](http://c.biancheng.net/java/60/)