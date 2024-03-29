# Open JDK 与 Oracle JDK区别

OpenJDK 是 在 2007 年由 Sun Corporation（现在的Oracle Corporation） 发布的。是 Oracle JDK 的开源实现版本，以 GPL 协议发布。在 JDK 7 的时候，Sub JDK 就是在 Open JDK 7 的基础上发布的，只替换了少量的源码。在 Sun 公司被 Oracle 收购以后，Sun SDK 就被称为 Oracle JDK。Oracle JDK 是基于 Oracle Binary COde License Agreement 协议。 两者的区别如下：

1. Oracle JDK 将三年发布一次稳定版本，OpenJDK 每三个月发布一次。
2. Oracle JDK 支持 LTS，OpenJDK 只支持当前版本至下一个版本发布。
3. Oracle JDK 采用 Oracle Binary Code License 协议，OpenJDK 采用 GPL v2 协议。
4. Oracle JDK 基于 OpenJDK 构建，技术上基本没有差异。

# Java 8

## 接口的默认方法

Java 8使我们能够通过使用 `default` 关键字向接口添加非抽象方法实现。 此功能也称为[虚拟扩展方法](http://stackoverflow.com/a/24102730)。例如下面这个例子：

```java
public interface Formula {
    double calculate(int a);

    // 接口默认方法
    default double sqrt(int a) {
        return Math.sqrt(a);
    }
}
```

Formula 接口中除了抽象方法计算接口公式还定义了默认方法 `sqrt`。 实现该接口的类只需要实现抽象方法 `calculate`。 默认方法`sqrt` 可以直接使用。不能通过抽象类或者接口直接创建对象，但是不管是抽象类还是接口，都可以通过匿名内部类的方式访问。

```java
public class FormulaTest {
    public static void main(String[] args) {
        Formula formula = new Formula() {
            @Override
            public double calculate(int a) {
                return sqrt(a * 100);
            }
        };

        System.out.println(formula.calculate(100)); // 100.0
        System.out.println(formula.sqrt(25)); // 5.0
    }
}
```

对于上面通过匿名内部类方式访问接口，我们可以这样理解：一个内部类实现了接口里的抽象方法并且返回一个内部类对象，之后我们让接口的引用来指向这个对象。 `formula`正是是作为匿名对象实现的。

## Lambda表达式和函数式接口

### Lambda表达式

Lambda 表达式，也可称为闭包，它是推动Java 8 发布的最重要新特性。 Lambda 允许把函数作为一个方法的参数（函数作为参数传递进方法中）。 使用Lambda 表达式可以使代码变的更加简洁紧凑。

老版本的Java中是如何排列字符串的：

```java
List<String> names = Arrays.asList("peter", "anna", "mike", "xenia");
Collections.sort(names, new Comparator<String>() {    
    @Override    
    public int compare(String a, String b) {        
        return b.compareTo(a);    
    }
});
```

只需要给静态方法 `Collections.sort` 传入一个 List 对象以及一个比较器来按指定顺序排列。通常做法都是创建一个匿名的比较器对象然后将其传递给 `sort` 方法。

在Java 8 中你就没必要使用这种传统的匿名对象的方式了，Java 8提供了更简洁的语法，lambda表达式：

```java
Collections.sort(names, (String a, String b) -> {    
    return b.compareTo(a);
});
```

可以看出，代码变得更段且更具有可读性，但是实际上还可以写得更短：

```java
Collections.sort(names, (String a, String b) -> b.compareTo(a));
```

对于函数体只有一行代码的，你可以去掉大括号{}以及return关键字，但是你还可以写得更短点：

```java
names.sort((a, b) -> b.compareTo(a));
```

List 类本身就有一个 `sort` 方法。并且Java编译器可以自动推导出参数类型，所以你可以不用再写一次类型。

### 函数式接口

**函数式接口**就是：**有且仅有一个抽象方法的接口（但是可以有多个非抽象方法的接口)**。

- `FunctionalInterface`注解标注一个函数式接口，不能标注`类`，`方法`，`枚举`，`属性`。
- 如果接口被标注了`@FunctionalInterface`，这个类就必须符合函数式接口的规范。
- 即使一个接口没有标注`@FunctionalInterface`，如果这个接口满足函数式接口规则，依旧被当作函数式接口。

Java中的函数式编程体现就是Lambda，所以函数式接口就是可以适用于Lambda使用的接口。只有确保接口中有且仅有一个抽象方法，Java中的Lambda才能顺利地进行推导。


>Tips：“**语法糖**”是指使用更加方便，但是原理不变的代码语法。例如在遍历集合时使用的for-each语法，其实
>底层的实现原理仍然是迭代器，这便是“语法糖”。从应用层面来讲，Java中的Lambda可以被当做是匿名内部
>类的“语法糖”，但是二者在原理上是不同的。


只要确保接口中有且仅有一个抽象方法即可：

```java
修饰符 interface 接口名称 {
	public abstract 返回值类型 方法名称(可选参数信息);
	// 其他非抽象方法内容
}
```

由于接口当中抽象方法的public abstract是可以省略的，所以定义一个函数式接口很简单：

```java
public interface MyFunctionalInterface {
	void myMethod();
}
```

与@Override注解的作用类似，Java 8中专门为函数式接口引入了一个新的注解：`@FunctionalInterface`。该注
解可用于一个接口的定义上：

```java
@FunctionalInterface
public interface MyFunctionalInterface {
	void myMethod();
}
```

一旦使用该注解来定义接口，编译器将会强制检查该接口是否确实有且仅有一个抽象方法，否则将会报错。需要注 意的是，即使不使用该注解，只要满足函数式接口的定义，这仍然是一个函数式接口，使用起来都一样。

> 详细可参考——[Lambda表达式与函数式接口](https://blog.csdn.net/KAIZ_LEARN/article/details/108912425)一文。

## 方法引用

Java 8允许您通过`::`关键字传递方法或构造函数的引用。 但我们也可以引用对象方法：

```java
class Something {
    String startsWith(String s) {
        return String.valueOf(s.charAt(0));
    }
}
```

```java
Something something = new Something();
Converter<String, String> converter = something::startsWith;
String converted = converter.convert("Java");
System.out.println(converted);    // "J"
```

接下来看看构造函数是如何使用`::`关键字来引用的，首先我们定义一个包含多个构造函数的简单类：

```java
class Person {
    String firstName;
    String lastName;

    Person() {}

    Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
}
```

接下来我们指定一个用来创建Person对象的对象工厂接口：

```java
interface PersonFactory<P extends Person> {
    P create(String firstName, String lastName);
}
```

这里我们使用构造函数引用来将他们关联起来，而不是手动实现一个完整的工厂：

```java
PersonFactory<Person> personFactory = Person::new;
Person person = personFactory.create("Peter", "Parker");
```

我们只需要使用 `Person::new` 来获取Person类构造函数的引用，Java编译器会自动根据`PersonFactory.create`方法的参数类型来选择合适的构造函数。

## Optionals

Optionals不是函数式接口，而是用于防止 NullPointerException 的漂亮工具。

Optional 是一个简单的容器，其值可能是null或者不是null。在Java 8之前一般某个函数应该返回非空对象但是有时却什么也没有返回，而在Java 8中，你应该返回 Optional 而不是 null。

```java
// of()：为非null的值创建一个Optional
Optional<String> optional = Optional.of("bam");
// isPresent（）： 如果值存在返回true，否则返回false
optional.isPresent();           // true
// get()：如果Optional有值则将其返回，否则抛出NoSuchElementException
optional.get();                 // "bam"
// orElse（）：如果有值则将其返回，否则返回指定的其它值
optional.orElse("fallback");    // "bam"
// ifPresent（）：如果Optional实例有值则为其调用consumer，否则不做处理
optional.ifPresent((s) -> System.out.println(s.charAt(0)));     // "b"
```

推荐阅读：[Java8如何正确使用Optional](https://blog.kaaass.net/archives/764)。

## Stream

`java.util.Stream` 表示能应用在一组元素上一次执行的操作序列。Stream 操作分为中间操作或者最终操作两种，最终操作返回一特定类型的计算结果，而中间操作返回Stream本身，这样你就可以将多个操作依次串起来。Stream 的创建需要指定一个数据源，比如` java.util.Collection` 的子类，List 或者 Set， Map 不支持。Stream 的操作可以串行执行或者并行执行。

首先看看Stream是怎么用，首先创建实例代码的用到的数据List：

```java
List<String> stringList = new ArrayList<>();
stringList.add("ddd2");
stringList.add("aaa2");
stringList.add("bbb1");
stringList.add("aaa1");
stringList.add("bbb3");
stringList.add("ccc");
stringList.add("bbb2");
stringList.add("ddd1");
```

Java 8扩展了集合类，可以通过 Collection.stream() 或者 Collection.parallelStream() 来创建一个Stream。

>  Stream流基础使用可查看——[Stream流基础使用指南](https://blog.csdn.net/KAIZ_LEARN/article/details/108923964)一文。

## Parallel Stream(并行流)

前面提到过Stream有串行和并行两种，串行Stream上的操作是在一个线程中依次完成，而并行Stream则是在多个线程上同时执行。

下面的例子展示了是如何通过并行Stream来提升性能：

首先我们创建一个没有重复元素的大表：

```java
int max = 1000000;
List<String> values = new ArrayList<>(max);
for (int i = 0; i < max; i++) {
    UUID uuid = UUID.randomUUID();
    values.add(uuid.toString());
}
```

我们分别用串行和并行两种方式对其进行排序，最后看看所用时间的对比。

**Sequential Sort(串行排序)**

```java
//串行排序
long t0 = System.nanoTime();
long count = values.stream().sorted().count();
System.out.println(count);

long t1 = System.nanoTime();

long millis = TimeUnit.NANOSECONDS.toMillis(t1 - t0);
System.out.println(String.format("sequential sort took: %d ms", millis));
```

```java
1000000
sequential sort took: 563 ms // 串行排序所用的时间
```

**Parallel Sort(并行排序)**

```java
//并行排序
long t0 = System.nanoTime();

long count = values.parallelStream().sorted().count();
System.out.println(count);

long t1 = System.nanoTime();

long millis = TimeUnit.NANOSECONDS.toMillis(t1 - t0);
System.out.println(String.format("parallel sort took: %d ms", millis));
```

```java
1000000
parallel sort took: 221 ms // 并行排序所用的时间
```

上面两个代码几乎是一样的，但是并行版的快了 50% 左右，唯一需要做的改动就是将 `stream()` 改为`parallelStream()`。

## 日期相关API

Java 8在 `java.time` 包下包含一个全新的日期和时间API。新的Date API与Joda-Time库相似，但它们不一样。以下示例涵盖了此新 API 的最重要部分。

- Clock 类提供了访问当前日期和时间的方法，Clock 是时区敏感的，可以用来取代 `System.currentTimeMillis()` 来获取当前的微秒数。某一个特定的时间点也可以使用 `Instant` 类来表示，`Instant` 类也可以用来创建旧版本的`java.util.Date` 对象。

- 在新API中时区使用 ZoneId 来表示。时区可以很方便的使用静态方法of来获取到。 抽象类`ZoneId`（在`java.time`包中）表示一个区域标识符。 它有一个名为`getAvailableZoneIds`的静态方法，它返回所有区域标识符。

- jdk1.8中新增了 LocalDate 与 LocalDateTime等类来解决日期处理方法，同时引入了一个新的类DateTimeFormatter 来解决日期格式化问题。可以使用Instant代替 Date，LocalDateTime代替 Calendar，DateTimeFormatter 代替 SimpleDateFormat。


**Clock**

Clock 类提供了访问当前日期和时间的方法，Clock 是时区敏感的，可以用来取代 `System.currentTimeMillis()` 来获取当前的微秒数。某一个特定的时间点也可以使用 `Instant` 类来表示，`Instant` 类也可以用来创建旧版本的`java.util.Date` 对象。

```java
        // Clock类提供了访问当前日期和时间的方法
        Clock clock = Clock.systemDefaultZone();
        long millis = clock.millis();
        System.out.println(millis);	// 1622180409157
        // 某一个特定的时间点也可以使用Instant类来表示
        Instant instant = clock.instant();
        System.out.println(instant); // 2021-05-28T05:40:09.157Z
        // Instant类也可以用来创建旧版本的java.util.Date对象
        Date legacyDate = Date.from(instant);
        System.out.println(legacyDate); // Fri May 28 13:40:09 CST 2021
```

**Timezones(时区)**

在新API中时区使用 ZoneId 来表示。时区可以很方便的使用静态方法of来获取到。 抽象类`ZoneId`（在`java.time`包中）表示一个区域标识符。 它有一个名为`getAvailableZoneIds`的静态方法，它返回所有区域标识符。

```java
// 输出所有区域标识符
System.out.println(ZoneId.getAvailableZoneIds());

ZoneId zone1 = ZoneId.of("Europe/Berlin");
ZoneId zone2 = ZoneId.of("Brazil/East");
System.out.println(zone1.getRules()); // ZoneRules[currentStandardOffset=+01:00]
System.out.println(zone2.getRules()); // ZoneRules[currentStandardOffset=-03:00]
```

**LocalTime(本地时间)**

jdk1.8中新增了 LocalDate 与 LocalDateTime等类来解决日期处理方法，同时引入了一个新的类DateTimeFormatter 来解决日期格式化问题。可以使用Instant代替 Date，LocalDateTime代替 Calendar，DateTimeFormatter 代替 SimpleDateFormat。

```java
LocalTime now1 = LocalTime.now(zone1);
LocalTime now2 = LocalTime.now(zone2);
System.out.println(now1.isBefore(now2));  // false

long hoursBetween = ChronoUnit.HOURS.between(now1, now2);
long minutesBetween = ChronoUnit.MINUTES.between(now1, now2);

System.out.println(hoursBetween);       // -4
System.out.println(minutesBetween);     // -299
```

LocalTime 提供了多种工厂方法来简化对象的创建，包括解析时间字符串.

```java
LocalTime late = LocalTime.of(23, 59, 59);
System.out.println(late);       // 23:59:59
DateTimeFormatter germanFormatter =
    DateTimeFormatter
        .ofLocalizedTime(FormatStyle.SHORT)
        .withLocale(Locale.GERMAN);

LocalTime leetTime = LocalTime.parse("13:37", germanFormatter);
System.out.println(leetTime);   // 13:37
```

**LocalDate(本地日期)**

LocalDate 表示了一个确切的日期，比如 2014-03-11。该对象值是不可变的，用起来和LocalTime基本一致。下面的例子展示了如何给Date对象加减天/月/年。另外要注意的是这些对象是不可变的，操作返回的总是一个新实例。

```java
LocalDate today = LocalDate.now();// 获取现在的日期
System.out.println("今天的日期: "+today);//2019-03-12
LocalDate tomorrow = today.plus(1, ChronoUnit.DAYS);
System.out.println("明天的日期: "+tomorrow);//2019-03-13
LocalDate yesterday = tomorrow.minusDays(2);
System.out.println("昨天的日期: "+yesterday);//2019-03-11
LocalDate independenceDay = LocalDate.of(2019, Month.MARCH, 12);
DayOfWeek dayOfWeek = independenceDay.getDayOfWeek();
System.out.println("今天是周几:"+dayOfWeek);//TUESDAY
```

从字符串解析一个 LocalDate 类型和解析 LocalTime 一样简单,下面是使用  `DateTimeFormatter` 解析字符串的例子：

```java
    String str1 = "2014==04==12 01时06分09秒";
        // 根据需要解析的日期、时间字符串定义解析所用的格式器
        DateTimeFormatter fomatter1 = DateTimeFormatter
                .ofPattern("yyyy==MM==dd HH时mm分ss秒");

        LocalDateTime dt1 = LocalDateTime.parse(str1, fomatter1);
        System.out.println(dt1); // 输出 2014-04-12T01:06:09

        String str2 = "2014$$$四月$$$13 20小时";
        DateTimeFormatter fomatter2 = DateTimeFormatter
                .ofPattern("yyy$$$MMM$$$dd HH小时");
        LocalDateTime dt2 = LocalDateTime.parse(str2, fomatter2);
        System.out.println(dt2); // 输出 2014-04-13T20:00

```

再来看一个使用 `DateTimeFormatter` 格式化日期的示例

```java
LocalDateTime rightNow=LocalDateTime.now();
String date=DateTimeFormatter.ISO_LOCAL_DATE_TIME.format(rightNow);
System.out.println(date);//2019-03-12T16:26:48.29
DateTimeFormatter formatter=DateTimeFormatter.ofPattern("YYYY-MM-dd HH:mm:ss");
System.out.println(formatter.format(rightNow));//2019-03-12 16:26:48
```

**LocalDateTime(本地日期时间)**

LocalDateTime 同时表示了时间和日期，相当于前两节内容合并到一个对象上了。LocalDateTime 和 LocalTime还有 LocalDate 一样，都是不可变的。LocalDateTime 提供了一些能访问具体字段的方法。

```java
LocalDateTime sylvester = LocalDateTime.of(2014, Month.DECEMBER, 31, 23, 59, 59);

DayOfWeek dayOfWeek = sylvester.getDayOfWeek();
System.out.println(dayOfWeek);      // WEDNESDAY

Month month = sylvester.getMonth();
System.out.println(month);          // DECEMBER

long minuteOfDay = sylvester.getLong(ChronoField.MINUTE_OF_DAY);
System.out.println(minuteOfDay);    // 1439
```

只要附加上时区信息，就可以将其转换为一个时间点Instant对象，Instant时间点对象可以很容易的转换为老式的`java.util.Date`。

```java
Instant instant = sylvester
        .atZone(ZoneId.systemDefault())
        .toInstant();

Date legacyDate = Date.from(instant);
System.out.println(legacyDate);     // Wed Dec 31 23:59:59 CET 2014
```

格式化LocalDateTime和格式化时间和日期一样的，除了使用预定义好的格式外，我们也可以自己定义格式：

```java
DateTimeFormatter formatter =
    DateTimeFormatter
        .ofPattern("MMM dd, yyyy - HH:mm");
LocalDateTime parsed = LocalDateTime.parse("Nov 03, 2014 - 07:13", formatter);
String string = formatter.format(parsed);
System.out.println(string);     // Nov 03, 2014 - 07:13
```

和java.text.NumberFormat不一样的是新版的DateTimeFormatter是不可变的，所以它是线程安全的，关于时间日期格式的详细信息在[这里](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html)。

## 多重注解

在Java 8中支持重复注解，如果要把同一个类型的注解使用多次，只需要给该注解标注一下`@Repeatable`即可。

先定义一个包装类Hints注解用来放置一组具体的Hint注解：

```java
@interface Hints {
    Hint[] value();
}

@Repeatable(Hints.class)
@interface Hint {
    String value();
}
```

例 1: 使用包装类当容器来存多个注解（老方法）

```java
@Hints({@Hint("hint1"), @Hint("hint2")})
class Person {}
```

例 2：使用多重注解（新方法）

```java
@Hint("hint1")
@Hint("hint2")
class Person {}
```

第二个例子里java编译器会隐性的帮你定义好@Hints注解，了解这一点有助于你用反射来获取这些信息：

```java
Hint hint = Person.class.getAnnotation(Hint.class);
System.out.println(hint);                   // null
Hints hints1 = Person.class.getAnnotation(Hints.class);
System.out.println(hints1.value().length);  // 2

Hint[] hints2 = Person.class.getAnnotationsByType(Hint.class);
System.out.println(hints2.length);          // 2
```

即便我们没有在 `Person`类上定义 `@Hints`注解，我们还是可以通过 `getAnnotation(Hints.class) `来获取 `@Hints`注解，更加方便的方法是使用 `getAnnotationsByType` 可以直接获取到所有的`@Hint`注解。
另外Java 8的注解还增加到两种新的target上了：

```java
@Target({ElementType.TYPE_PARAMETER, ElementType.TYPE_USE})
@interface MyAnnotation {}
```

## Base64 支持

Java 8 标准库中提供了对 Base 64 编码的支持。具体 API 见可参照[文档](https://docs.oracle.com/javase/8/docs/api/java/util/Base64.html)。下面是示例代码。

```java
String base64 = Base64.getEncoder().encodeToString("aaa".getBytes());
System.out.println(base64);
byte[] bytes = Base64.getDecoder().decode(base64);
System.out.println(new String(bytes));
```

## 其他新特性

- 对并发的增强 在`java.util.concurrent.atomic`包中还增加了下面这些类： `DoubleAccumulator/DoubleAdder LongAccumulator/LongAdder`；
- 提供了新的 `Nashorn javascript `引擎；
- 提供了 jjs，是一个给予 Nashorn 的命令行工具，可以用来执行 JavaScript 源码；
- 提供了新的类依赖分析工具 jdeps；
- JVM 的新特性 JVM内存永久区已经被metaspace替换（JEP 122）。JVM参数 `-XX:PermSize` 和 `–XX:MaxPermSize`被`-XX:MetaSpaceSize` 和 `-XX:MaxMetaspaceSize`代替。

> 其他一些改进可参照 [www.oracle.com/technetwork…](https://www.oracle.com/technetwork/java/javase/8-whats-new-2157071.html)

# Java9

发布于 2017 年 9 月 21 日 。作为 Java8 之后 3 年半才发布的新版本，Java 9 带 来了很多重大的变化其中最重要的改动是 Java 平台模块系统的引入,其他还有诸如集合、Stream 流

## Java 平台模块系统

Java 平台模块系统，也就是 Project Jigsaw，把模块化开发实践引入到了 Java 平台中。在引入了模块系统之后，JDK 被重新组织成 94 个模块。Java 应用可以通过新增的 jlink 工具，创建出只包含所依赖的 JDK 模块的自定义运行时镜像。这样可以极大的减少 Java 运行时环境的大小。

Java 9 模块的重要特征是在其工件（artifact）的根目录中包含了一个描述模块的 module-info.class 文 件。 工件的格式可以是传统的 JAR 文件或是 Java 9 新增的 JMOD 文件。

## Jshell

jshell 是 Java 9 新增的一个实用工具。为 Java 提供了类似于 Python 的实时命令行交互工具。

在 Jshell 中可以直接输入表达式并查看其执行结果

## 集合、Stream 和 Optional

- 增加 了 `List.of()`、`Set.of()`、`Map.of()` 和 `Map.ofEntries()`等工厂方法来创建不可变集合，比如`List.of("Java", "C++");`、`Map.of("Java", 1, "C++", 2)`;（这部分内容有点参考 Guava 的味道）
- `Stream` 中增加了新的方法 `ofNullable`、`dropWhile`、`takeWhile` 和 `iterate` 方法。`Collectors` 中增加了新的方法 `filtering` 和 `flatMapping`
- `Optional` 类中新增了 `ifPresentOrElse`、`or` 和 `stream` 等方法

## 进程 API

Java 9 增加了 `ProcessHandle` 接口，可以对原生进程进行管理，尤其适合于管理长时间运行的进程

## 平台日志 API 和服务

Java 9 允许为 JDK 和应用配置同样的日志实现。新增了 `System.LoggerFinder` 用来管理 JDK 使 用的日志记录器实现。JVM 在运行时只有一个系统范围的 `LoggerFinder` 实例。

我们可以通过添加自己的 `System.LoggerFinder` 实现来让 JDK 和应用使用 SLF4J 等其他日志记录框架。

## 反应式流 （ Reactive Streams ）

- 在 Java9 中的 `java.util.concurrent.Flow` 类中新增了反应式流规范的核心接口
- Flow 中包含了 `Flow.Publisher`、`Flow.Subscriber`、`Flow.Subscription` 和 `Flow.Processor` 等 4 个核心接口。Java 9 还提供了`SubmissionPublisher` 作为`Flow.Publisher` 的一个实现。

## 变量句柄

- 变量句柄是一个变量或一组变量的引用，包括静态域，非静态域，数组元素和堆外数据结构中的组成部分等
- 变量句柄的含义类似于已有的方法句柄`MethodHandle`
- 由 Java 类`java.lang.invoke.VarHandle` 来表示，可以使用类 `java.lang.invoke.MethodHandles.Lookup` 中的静态工厂方法来创建 `VarHandle` 对 象

## 改进方法句柄（Method Handle）

- 方法句柄从 Java7 开始引入，Java9 在类`java.lang.invoke.MethodHandles` 中新增了更多的静态方法来创建不同类型的方法句柄

## 其它新特性

- **接口私有方法** ：Java 9 允许在接口中使用私有方法
- **try-with-resources 增强** ：在 try-with-resources 语句中可以使用 effectively-final 变量（什么是 effectively-final 变量，见这篇文章 [http://ilkinulas.github.io/programming/java/2016/03/27/effectively-final-java.html](http://ilkinulas.github.io/programming/java/2016/03/27/effectively-final-java.html)）
- **类 `CompletableFuture` 中增加了几个新的方法（`completeAsync` ，`orTimeout` 等）**
- **Nashorn 引擎的增强** ：Nashorn 从 Java8 开始引入的 JavaScript 引擎，Java9 对 Nashorn 做了些增强，实现了一些 ES6 的新特性
- **I/O 流的新特性** ：增加了新的方法来读取和复制 InputStream 中包含的数据
- **改进应用的安全性能** ：Java 9 新增了 4 个 SHA- 3 哈希算法，SHA3-224、SHA3-256、SHA3-384 和 S HA3-512
- ......

> 其他一些改进可参照 [docs.oracle.com/javase/9/wh…](https://docs.oracle.com/javase/9/whatsnew/toc.htm#JSNEW-GUID-C23AFD78-C777-460B-8ACE-58BE5EA681F6)

# Java10

发布于 2018 年 3 月 20 日，最知名的特性应该是 var 关键字（局部变量类型推断）的引入了，其他还有垃圾收集器改善、GC 改进、性能提升、线程管控等一批新特性

## var 关键字

- **介绍** :提供了 var 关键字声明局部变量：`var list = new ArrayList<String>(); // ArrayList<String>`
- **局限性** ：只能用于带有构造器的**局部变量**和 for 循环中

_Guide 哥：实际上 Lombok 早就体用了一个类似的关键字，使用它可以简化代码，但是可能会降低程序的易读性、可维护性。一般情况下，我个人都不太推荐使用。_

## 不可变集合

**list，set，map 提供了静态方法`copyOf()`返回入参集合的一个不可变拷贝（以下为 JDK 的源码）**

```java
static <E> List<E> copyOf(Collection<? extends E> coll) {
    return ImmutableCollections.listCopy(coll);
}
```

**`java.util.stream.Collectors`中新增了静态方法，用于将流中的元素收集为不可变的集合**

## Optional

- 新增了`orElseThrow()`方法来在没有值时抛出异常

## 并行全垃圾回收器 G1

从 Java9 开始 G1 就了默认的垃圾回收器，G1 是以一种低延时的垃圾回收器来设计的，旨在避免进行 Full GC,但是 Java9 的 G1 的 FullGC 依然是使用单线程去完成标记清除算法,这可能会导致垃圾回收期在无法回收内存的时候触发 Full GC。

为了最大限度地减少 Full GC 造成的应用停顿的影响，从 Java10 开始，G1 的 FullGC 改为并行的标记清除算法，同时会使用与年轻代回收和混合回收相同的并行工作线程数量，从而减少了 Full GC 的发生，以带来更好的性能提升、更大的吞吐量。

## 应用程序类数据共享

在 Java 5 中就已经引入了类数据共享机制 (Class Data Sharing，简称 CDS)，允许将一组类预处理为共享归档文件，以便在运行时能够进行内存映射以减少 Java 程序的启动时间，当多个 Java 虚拟机（JVM）共享相同的归档文件时，还可以减少动态内存的占用量，同时减少多个虚拟机在同一个物理或虚拟的机器上运行时的资源占用

Java 10 在现有的 CDS 功能基础上再次拓展，以允许应用类放置在共享存档中。CDS 特性在原来的 bootstrap 类基础之上，扩展加入了应用类的 CDS (Application Class-Data Sharing) 支持。其原理为：在启动时记录加载类的过程，写入到文本文件中，再次启动时直接读取此启动文本并加载。设想如果应用环境没有大的变化，启动速度就会得到提升

## 其他特性

- **线程-局部管控**：Java 10 中线程管控引入 JVM 安全点的概念，将允许在不运行全局 JVM 安全点的情况下实现线程回调，由线程本身或者 JVM 线程来执行，同时保持线程处于阻塞状态，这种方式使得停止单个线程变成可能，而不是只能启用或停止所有线程

- **备用存储装置上的堆分配**：Java 10 中将使得 JVM 能够使用适用于不同类型的存储机制的堆，在可选内存设备上进行堆内存分配
- **统一的垃圾回收接口**：Java 10 中，hotspot/gc 代码实现方面，引入一个干净的 GC 接口，改进不同 GC 源代码的隔离性，多个 GC 之间共享的实现细节代码应该存在于辅助类中。统一垃圾回收接口的主要原因是：让垃圾回收器（GC）这部分代码更加整洁，便于新人上手开发，便于后续排查相关问题。

> 其他一些改进可以参照 [www.oracle.com/technetwork…](https://www.oracle.com/technetwork/java/javase/10-relnote-issues-4108729.html#NewFeature)

# Java11

Java11 于 2018 年 9 月 25 日正式发布，这是很重要的一个版本！Java 11 和 2017 年 9 月份发布的 Java 9 以及 2018 年 3 月份发布的 Java 10 相比，其最大的区别就是：在长期支持(Long-Term-Support)方面，**Oracle 表示会对 Java 11 提供大力支持，这一支持将会持续至 2026 年 9 月。这是据 Java 8 以后支持的首个长期版本。**

## 字符串加强

Java 11 增加了一系列的字符串处理方法，说白点就是多了层封装，JDK 开发组的人没少看市面上常见的工具类框架啊!

```java
//判断字符串是否为空
" ".isBlank();//true
//去除字符串首尾空格
" Java ".strip();// "Java" 
//去除字符串首部空格
" Java ".stripLeading();   // "Java "  
//去除字符串尾部空格
" Java ".stripTrailing();  // " Java"  
//重复字符串多少次
"Java".repeat(3);             // "JavaJavaJava"  

//返回由行终止符分隔的字符串集合。
"A\nB\nC".lines().count();    // 3 
"A\nB\nC".lines().collect(Collectors.toList()); 
```

## ZGC：可伸缩低延迟垃圾收集器

**ZGC 即 Z Garbage Collector**，是一个可伸缩的、低延迟的垃圾收集器。

ZGC 主要为了满足如下目标进行设计：

- GC 停顿时间不超过 10ms
- 即能处理几百 MB 的小堆，也能处理几个 TB 的大堆
- 应用吞吐能力不会下降超过 15%（与 G1 回收算法相比）
- 方便在此基础上引入新的 GC 特性和利用 colord
- 针以及 Load barriers 优化奠定基础
- 当前只支持 Linux/x64 位平台

ZGC 目前 **处在实验阶段**，只支持 Linux/x64 平台

## 标准 HTTP Client 升级

Java 11 对 Java 9 中引入并在 Java 10 中进行了更新的 Http Client API 进行了标准化，在前两个版本中进行孵化的同时，Http Client 几乎被完全重写，并且现在完全支持异步非阻塞。

并且，Java11 中，Http Client 的包名由 `jdk.incubator.http` 改为`java.net.http`，该 API 通过 `CompleteableFuture` 提供非阻塞请求和响应语义。

使用起来也很简单，如下：

```java
var request = HttpRequest.newBuilder()  
    .uri(URI.create("https://javastack.cn"))  
    .GET()  
    .build();  

var client = HttpClient.newHttpClient();  

// 同步  
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());  
System.out.println(response.body());  

// 异步  
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())  
    .thenApply(HttpResponse::body)  
    .thenAccept(System.out::println); 
```

## 简化启动单个源代码文件的方法

- 增强了 Java 启动器，使其能够运行单一文件的 Java 源代码。此功能允许使用 Java 解释器直接执行 Java 源代码。源代码在内存中编译，然后由解释器执行。唯一的约束在于所有相关的类必须定义在同一个 Java 文件中
- 对于 Java 初学者并希望尝试简单程序的人特别有用，并且能和 jshell 一起使用
- 一定能程度上增强了使用 Java 来写脚本程序的能力

## 用于 Lambda 参数的局部变量语法

- 从 Java 10 开始，便引入了局部变量类型推断这一关键特性。类型推断允许使用关键字 var 作为局部变量的类型而不是实际类型，编译器根据分配给变量的值推断出类型
- Java 10 中对 var 关键字存在几个限制
  - 只能用于局部变量上
  - 声明时必须初始化
  - 不能用作方法参数
  - 不能在 Lambda 表达式中使用
- Java11 开始允许开发者在 Lambda 表达式中使用 var 进行参数声明

## 其他特性

- 新的垃圾回收器 Epsilon，一个完全消极的 GC 实现，分配有限的内存资源，最大限度的降低内存占用和内存吞吐延迟时间
- 低开销的 Heap Profiling：Java 11 中提供一种低开销的 Java 堆分配采样方法，能够得到堆分配的 Java 对象信息，并且能够通过 JVMTI 访问堆信息
- TLS1.3 协议：Java 11 中包含了传输层安全性（TLS）1.3 规范（RFC 8446）的实现，替换了之前版本中包含的 TLS，包括 TLS 1.2，同时还改进了其他 TLS 功能，例如 OCSP 装订扩展（RFC 6066，RFC 6961），以及会话散列和扩展主密钥扩展（RFC 7627），在安全性和性能方面也做了很多提升
- 飞行记录器：飞行记录器之前是商业版 JDK 的一项分析工具，但在 Java 11 中，其代码被包含到公开代码库中，这样所有人都能使用该功能了

> 其他一些改进可以参照 [www.oracle.com/technetwork…](https://www.oracle.com/technetwork/java/javase/11-relnote-issues-5012449.html#NewFeature)

# Java12

## 增强 Switch

- 传统的 switch 语法存在容易漏写 break 的问题，而且从代码整洁性层面来看，多个 break 本质也是一种重复

- Java12 提供了 swtich 表达式，使用类似 lambda 语法条件匹配成功后的执行块，不需要多写 break

- 作为预览特性加入，需要在`javac`编译和`java`运行时增加参数`--enable-preview`

  ```java
  switch (day) {
      case MONDAY, FRIDAY, SUNDAY -> System.out.println(6);
      case TUESDAY                -> System.out.println(7);
      case THURSDAY, SATURDAY     -> System.out.println(8);
      case WEDNESDAY              -> System.out.println(9);
  }
  ```

## 数字格式化工具类

- `NumberFormat` 新增了对复杂的数字进行格式化的支持

  ```java
  NumberFormat fmt = NumberFormat.getCompactNumberInstance(Locale.US, NumberFormat.Style.SHORT);
  String result = fmt.format(1000);
   System.out.println(result); // 输出为 1K，计算工资是多少K更方便了。。。
  ```

## Shenandoah GC

- Redhat 主导开发的 Pauseless GC 实现，主要目标是 99.9% 的暂停小于 10ms，暂停与堆大小无关等
- 和 Java11 开源的 ZGC 相比（需要升级到 JDK11 才能使用），Shenandoah GC 有稳定的 JDK8u 版本，在 Java8 占据主要市场份额的今天有更大的可落地性

## G1 收集器提升

- **Java12 为默认的垃圾收集器 G1 带来了两项更新:**
  - 可中止的混合收集集合：JEP344 的实现，为了达到用户提供的停顿时间目标，JEP 344 通过把要被回收的区域集（混合收集集合）拆分为强制和可选部分，使 G1 垃圾回收器能中止垃圾回收过程。 G1 可以中止可选部分的回收以达到停顿时间目标
  - 及时返回未使用的已分配内存：JEP346 的实现，增强 G1 GC，以便在空闲时自动将 Java 堆内存返回给操作系统。

> 其他一些改进可以参照 [www.oracle.com/technetwork…](https://www.oracle.com/technetwork/java/javase/12-relnote-issues-5211422.html#NewFeature)

# Java13

## 引入 yield 关键字到 Switch 中

- `Switch` 表达式中就多了一个关键字用于跳出 `Switch` 块的关键字 `yield`，主要用于返回一个值

- `yield`和 `return` 的区别在于：`return` 会直接跳出当前循环或者方法，而 `yield` 只会跳出当前 `Switch` 块，同时在使用 `yield` 时，需要有 `default` 条件

  ```java
   private static String descLanguage(String name) {
          return switch (name) {
              case "Java": yield "object-oriented, platform independent and secured";
              case "Ruby": yield "a programmer's best friend";
              default: yield name +" is a good language";
          };
   }
  ```

## 文本块

- 解决 Java 定义多行字符串时只能通过换行转义或者换行连接符来变通支持的问题，引入**三重双引号**来定义多行文本

- 两个`"""`中间的任何内容都会被解释为字符串的一部分，包括换行符

  ```java
  String json ="{\n" +
                "   \"name\":\"mkyong\",\n" +
                "   \"age\":38\n" +
                "}\n";   // 未支持文本块之前
  ```

  ```java
   String json = """
                  {
                      "name":"mkyong",
                      "age":38
                  }
                  """;
  ```

## 增强 ZGC 释放未使用内存

- 在 Java 11 中是实验性的引入的 ZGC 在实际的使用中存在未能主动将未使用的内存释放给操作系统的问题
- ZGC 堆由一组称为 ZPages 的堆区域组成。在 GC 周期中清空 ZPages 区域时，它们将被释放并返回到页面缓存 **ZPageCache** 中，此缓存中的 ZPages 按最近最少使用（LRU）的顺序，并按照大小进行组织
- 在 Java 13 中，ZGC 将向操作系统返回被标识为长时间未使用的页面，这样它们将可以被其他进程重用

## SocketAPI 重构

- Java 13 为 Socket API 带来了新的底层实现方法，并且在 Java 13 中是默认使用新的 Socket 实现，使其易于发现并在排除问题同时增加可维护性

## 动态应用程序类-数据共享

- Java 13 中对 Java 10 中引入的 应用程序类数据共享进行了进一步的简化、改进和扩展，即：**允许在 Java 应用程序执行结束时动态进行类归档**，具体能够被归档的类包括：所有已被加载，但不属于默认基层 CDS 的应用程序类和引用类库中的类。

> 其他一些改进可以参照 [www.oracle.com/technetwork…](https://www.oracle.com/java/technologies/javase/13-relnote-issues.html#NewFeature)

# Java14

## record 关键字

- 简化数据类的定义方式，使用 record 代替 class 定义的类，只需要声明属性，就可以在获得属性的访问方法，以及 toString，hashCode,equals 方法

- 类似于使用 Class 定义类，同时使用了 lomobok 插件，并打上了`@Getter,@ToString,@EqualsAndHashCode`注解

- 作为预览特性引入

  ```java
  /**
   * 这个类具有两个特征
   * 1. 所有成员属性都是final
   * 2. 全部方法由构造方法，和两个成员属性访问器组成（共三个）
   * 那么这种类就很适合使用record来声明
   */
  final class Rectangle implements Shape {
      final double length;
      final double width;
  
      public Rectangle(double length, double width) {
          this.length = length;
          this.width = width;
      }
  
      double length() { return length; }
      double width() { return width; }
  }
  /**
   * 1. 使用record声明的类会自动拥有上面类中的三个方法
   * 2. 在这基础上还附赠了equals()，hashCode()方法以及toString()方法
   * 3. toString方法中包括所有成员属性的字符串表示形式及其名称
   */
  record Rectangle(float length, float width) { }
  ```

## 空指针异常精准提示

- 通过 JVM 参数中添加`-XX:+ShowCodeDetailsInExceptionMessages`，可以在空指针异常中获取更为详细的调用信息，更快的定位和解决问题

  ```java
  a.b.c.i = 99; // 假设这段代码会发生空指针
  ```

  ```java
  Exception in thread "main" java.lang.NullPointerException:
          Cannot read field 'c' because 'a.b' is null.
      at Prog.main(Prog.java:5) // 增加参数后提示的异常中很明确的告知了哪里为空导致
  ```

## switch 的增强终于转正

- JDK12 引入的 switch（预览特性）在 JDK14 变为正式版本，不需要增加参数来启用，直接在 JDK14 中就能使用
- 主要是用`->`来替代以前的`:`+`break`；另外就是提供了 yield 来在 block 中返回值

_Before Java 14_

```java
switch (day) {
    case MONDAY:
    case FRIDAY:
    case SUNDAY:
        System.out.println(6);
        break;
    case TUESDAY:
        System.out.println(7);
        break;
    case THURSDAY:
    case SATURDAY:
        System.out.println(8);
        break;
    case WEDNESDAY:
        System.out.println(9);
        break;
}
```

_Java 14 enhancements_

```java
switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> System.out.println(6);
    case TUESDAY                -> System.out.println(7);
    case THURSDAY, SATURDAY     -> System.out.println(8);
    case WEDNESDAY              -> System.out.println(9);
}
```

## instanceof 增强

- instanceof 主要在**类型强转前探测对象的具体类型**，然后执行具体的强转

- 新版的 instanceof 可以在判断的是否属于具体的类型同时完成转换

```java
Object obj = "我是字符串";
if(obj instanceof String str){
	System.out.println(str);
}
```

## 其他特性

- 从 Java11 引入的 ZGC 作为继 G1 过后的下一代 GC 算法，从支持 Linux 平台到 Java14 开始支持 MacOS 和 Window（个人感觉是终于可以在日常开发工具中先体验下 ZGC 的效果了，虽然其实 G1 也够用）
- 移除了 CMS 垃圾收集器（功成而退）
- 新增了 jpackage 工具，标配将应用打成 jar 包外，还支持不同平台的特性包，比如 linux 下的`deb`和`rpm`，window 平台下的`msi`和`exe`

> 其他一些改进可以参照 [www.oracle.com/technetwork…](https://www.oracle.com/java/technologies/javase/14-relnote-issues.html#NewFeature)

# 参考

* [聊聊 Java8 以后各个版本的新特性](https://juejin.cn/post/6844903918586052616)
* [Java 8 新特性最佳指南](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247484744&idx=1&sn=9db31dca13d327678845054af75efb74&chksm=cea24a83f9d5c3956f4feb9956b068624ab2fdd6c4a75fe52d5df5dca356a016577301399548&token=1082669959&lang=zh_CN&scene=21#wechat_redirect)

