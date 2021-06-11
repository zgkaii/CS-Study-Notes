**引言：什么是Stream?**

现在很多大数据量系统中都存在分表分库的情况。

例如，电商系统中的订单表，常常使用用户 ID 的 Hash 值来实现分表分库，这样是为了减少单个表的数据量，优化用户查询订单的速度。

但在后台管理员审核订单时，他们需要将各个数据源的数据查询到应用层之后进行合并操作。

例如，当我们需要查询出过滤条件下的所有订单，并按照订单的某个条件进行排序，单个数据源查询出来的数据是可以按照某个条件进行排序的，但多个数据源查询出来已经排序好的数据，并不代表合并后是正确的排序，所以我们需要在应用层对合并数据集合重新进行排序。

在 Java8 之前，我们通常是通过 for 循环或者 Iterator 迭代来重新排序合并数据，又或者通过重新定义 `Collections.sorts` 的 Comparator 方法来实现，这两种方式对于大数据量系统来说，效率并不是很理想。

Java8 中添加了一个新的接口类 Stream，他和我们之前接触的字节流概念不太一样，Java8 集合中的 Stream 相当于高级版的 Iterator，他可以通过 Lambda 表达式对集合进行各种非常便利、高效的聚合操作（Aggregate Operation），或者大批量数据操作 (Bulk Data Operation)。

Stream 的聚合操作与数据库 SQL 的聚合操作 sorted、filter、map 等类似。我们在应用层就可以高效地实现类似数据库 SQL 的聚合操作了，而在数据操作方面，Stream 不仅可以通过串行的方式实现数据操作，还可以通过并行的方式处理大批量数据，提高数据的处理效率。

这里用一个案例来说明Stream流的强大，下面这个 Demo 的需求是过滤分组一所中学里身高在 160cm 以上的男女同学，我们先用传统的迭代方式来实现，代码如下：

```java
	Map<String, List<Student>> stuMap = new HashMap<String, List<Student>>();
        for (Student stu: studentsList) {
            if (stu.getHeight() > 160) { // 如果身高大于 160
                if (stuMap.get(stu.getSex()) == null) { // 该性别还没分类
                    List<Student> list = new ArrayList<Student>(); // 新建该性别学生的列表
                    list.add(stu);// 将学生放进去列表
                    stuMap.put(stu.getSex(), list);// 将列表放到 map 中
                } else { // 该性别分类已存在
                    stuMap.get(stu.getSex()).add(stu);// 该性别分类已存在，则直接放进去即可
                }
            }
        }
 
```

我们再使用 Java8 中的 Stream API 进行实现：

1. 串行实现

```java
Map<String, List<Student>> stuMap = stuList.stream()
    .filter((Student s) -> s.getHeight() > 160)
    .collect(Collectors.groupingBy(Student ::getSex)); 
```

2. 并行实现

```java
Map<String, List<Student>> stuMap = stuList.parallelStream()
    .filter((Student s) -> s.getHeight() > 160)
    .collect(Collectors.groupingBy(Student ::getSex)); 
```

通过上面两个简单的例子，我们可以发现，Stream 结合 Lambda 表达式实现遍历筛选功能非常得简洁和便捷。

# 1 Stream流基础

在Java 8中，得益于Lambda所带来的函数式编程，引入了一个全新的Stream概念，用于解决已有集合类库既有的弊端。Stream流尽管被称作为“流”，但它和文件流、字符流、字节流**完全没有任何关系**。

## 1.1 概述

整体来看，Stream流类似于工厂车间的“生产流水线**”，只需要把集合、命令还有一些参数灌输到**流水线中去，就可以加工成得出想要的结果。

```shell
原集合 —> 获取流 —> 中间操作 (过滤、分组、统计) —> 终端操作  
```

当**使用流**的时候，通常有以下几个步骤：集合→获取流→**中间操作**（比如过滤、筛选、分组、计算）获取想要的结果→**终端操作**（就是转化成我们想要的数据，这个数据的形式一般还是集合，有时也会按照需求输出 count 计数。）

> Tips：“Stream流”其实是一个集合元素的函数模型，它并不是集合，也不是数据结构，其本身并不存储任何元素或其地址。

中间操作又可以分为无状态（Stateless）与有状态（Stateful）操作，前者是指元素的处理不受之前元素的影响，后者是指该操作只有拿到所有元素之后才能继续下去。

终结操作又可以分为短路（Short-circuiting）与非短路（Unshort-circuiting）操作，前者是指遇到某些符合条件的元素就可以得到最终结果，后者是指必须处理完所有元素才能得到最终结果。操作分类详情如下图所示：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210610111725492.png" width="800px"/>
</div>

可以这把Stream流理解为一个来自数据源的元素队列并支持聚合操作。

Stream（流）是一个来自数据源的元素队列：

* **元素**：是特定类型的对象，形成一个队列。 Java中的Stream并不会存储元素，而是按需计算。
* **数据源**：流的来源。 可以是集合，数组，I/O channel，产生器 generator 等。
* **聚合操作**：类似 SQL 语句一样的操作，比如 filter, map, reduce, find, match, sorted 等。

和Collection操作不同， Stream操作还有两个基础的特征：

* **Pipelining**：中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluentstyle）。 这样做可以对操作进行优化， 比如延迟执行（laziness）和短路（short-circuiting）。
* **内部迭代**： 以前对集合遍历都是通过Iterator或者foreach的方式，在集合外部进行外部迭代。 Stream提供了内部迭代的方式，流可以直接调用遍历方法。

## 1.2 获取流

`java.util.stream.Stream<T>`是Java 8中最常用的流接口。（并不是一个函数式接口。）

获取一个流非常简单，常用的方式：

* 所有的Collection集合、Map集合都可以通过stream默认方法获取流（串行流）

```java
    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }
```

* Stream接口的静态方法`of`可以获取数组对应的流

```java
    public static<T> Stream<T> of(T... values) {
        return Arrays.stream(values);
    }
```

**Collection获取流**

首先，`java.util.Collection`接口中加入了default方法stream用来获取流，所以其所有实现类均可获取流。

```java
public class StreamTest {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        // ...
        Stream<String> stream1 = list.stream();
        Set<String> set = new HashSet<>();
        // ...
        Stream<String> stream2 = set.stream();
        Vector<String> vector = new Vector<>();
        // ...
        Stream<String> stream3 = vector.stream();
    }
}
```

**Map获取流**

`java.util.Map`接口不是Collection的子接口，且其`K-V`数据结构不符合流元素的单一特征，所以获取对应的流
需要分key、value或entry等情况。

```java
        Map<String, String> map = new HashMap<>();
        // ...
        Stream<String> keyStream = map.keySet().stream();
        Stream<String> valueStream = map.values().stream();
        Stream<Map.Entry<String, String>> entryStream = map.entrySet().stream();
```

**数组获取流**

如果使用的是数组，由于数组对象不可能添加默认方法，所以Stream接口中提供了静态方法`of`，使用很简单：

```java
        String[] array = { "张三", "李四", "王五"};
        Stream<String> stream = Stream.of(array);
```


>Tips：of方法的参数其实是一个可变参数，所以支持数组。

## 1.3 中间操作

常用方法：

|   操作   |        类型         | 返回类型  |    操作参数     |  函数描述符   |
| :------: | :-----------------: | :-------: | :-------------: | :-----------: |
|  filter  |        中间         | Stream<T> |  Predicate<T>   | T -> boolean  |
|   map    |        中间         | Stream<R> | Funcation<T，R> |    T -> R     |
|  limit   | 中间（有状态-有界） | Stream<T> |                 |               |
|  sorted  |        中间         | Stream<T> |  Comparator<T>  | (T，T) -> int |
| distinct | 中间（有状态-无界） | Stream<T> |                 |               |
|   skip   | 中间（有状态-有界） | Stream<T> |      long       |               |

### (1) filter 过滤

可以通过 filter 方法将一个流转换成另一个子集流。方法签名：

```java
Stream<T> filter(Predicate<? super T> predicate);
```

filter 里面，`-> `箭头后面跟着的是一个**boolean 值**，可以写任何的过滤条件，就相当于 sql 中 where 后面的东西，换句话说，**能用 sql 实现的功能这里都可以实现**。

```java
public class StreamTest {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("aaa");
        list.add("bbb");
        list.add("ccc");
        list.add("aaa");

        List filterList = list.stream()
                .filter(value -> value.contains("a"))
                .collect(Collectors.toList());
        System.out.println(filterList);
    }
}
```

### (2) map 映射

如果需要将流中的元素映射到另一个流中，可以使用 map 方法。方法签名：

```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
```

该接口需要一个 Function 函数式接口参数，可以将当前流中的T类型数据转换为另一种R类型的流。

```java
        List mapList = list.stream()
                .map(String -> String.toUpperCase())
                .collect(Collectors.toList());
```

### (3) limit 截取

limit 方法可以对流进行截取，**只取用集合中前n个数据**。方法签名：

```java
Stream<T> limit(long maxSize);
```

参数是一个long型，如果集合当前长度大于参数则进行截取；否则不进行操作。基本使用：

```java
        List limitList = list.stream()
                .limit(2)
                .collect(Collectors.toList());
```

### (4) sorted 排序

 **sorted() 方法**对元素进行排序。方法签名：

```java
Stream<T> sorted(Comparator<? super T> comparator);
```

基本使用：

```java
        List sortedlist = list.stream()
                .sorted(Comparator.reverseOrder())
                .collect(Collectors.toList());
```

### (5) distinct 去重

和 sql 中的 distinct 关键字很相似，可以对重复元素去重。方法签名：

```java
Stream<T> distinct();
```

基本使用：

```
        List distinctList = list.stream()
                .distinct()
                .collect(Collectors.toList());
```

### (6) skip 跳过

limit 方法可以对流进行截取，**跳过集合中前n个数据其之后的数据**。方法签名：

```java
Stream<T> skip(long n);
```

如果流的当前长度大于n，则跳过前n个；否则将会得到一个长度为0的空流。基本使用：

```java
        List skipList = list.stream()
                .skip(2)
                .collect(Collectors.toList());
```

## 1.4 终端操作

|  操作   |        类型         |  返回类型   |      操作参数      |  函数描述符   |
| :-----: | :-----------------: | :---------: | :----------------: | :-----------: |
| forEach |        终端         |    void     |    Consumer<T>     |   T -> void   |
| collect |        终端         |      R      | Collector<T，A，R> |               |
| reduce  | 终端（有状态-有界） | Optional<T> |   BinaryOperator   | （T，T） -> T |
|  count  |        终端         |    long     |                    |               |

### (1) forEach

虽然方法名字叫 forEach ，但是与for循环中的“for-each”昵称不同。方法签名：

```java
void forEach(Consumer<? super T> action);
```

该方法接收一个 Consumer 接口函数，会将每一个流元素交给该函数进行处理。基本使用：

```java
        Stream<String> stream = Stream.of("张三", "李四", "王五");
		stream.forEach(name-> System.out.println(name));
```

### (2) collect

* `Collectors.toList()`：转换成List集合。/ `Collectors.toSet()`：转换成set集合。

```java
System.out.println(Stream.of("a", "b", "c","a").collect(Collectors.toSet()));
```

* `Collectors.toCollection(TreeSet::new)`：转换成特定的set集合。

```java
TreeSet<String> treeSet = Stream.of("a", "c", "b", "a").collect(Collectors.toCollection(TreeSet::new));
System.out.println(treeSet);
```

* `Collectors.toMap(keyMapper, valueMapper, mergeFunction)`：转换成map。

```java
Map<String, String> collect = Stream.of("a", "b", "c", "a").collect(Collectors.toMap(x -> x, x -> x + x,(oldVal, newVal) -> newVal)));
collect.forEach((k,v) -> System.out.println(k + ":" + v));
```

### (3) count

正如旧集合 Collection 当中的 `size` 方法一样，流提供 count 方法来数一数其中的元素个数。方法签名：

```java
long count();
```

基本使用：

```java
        Stream<String> original = Stream.of("张三", "李四", "王五");
        Stream<String> result = original.filter(s -> s.startsWith("张"));
        System.out.println(result.count());
```

### (4) reduce

reduce就是一个归一化的迭代操作。接受一个stream，通过重复应用操作将他们组合成一个简单结果。

三个重载方法：

```java
Optional<T> reduce(BinaryOperator<T> accumulator);

T reduce(T identity, BinaryOperator<T> accumulator);

<U> U reduce(U identity,
                 BiFunction<U, ? super T, U> accumulator,
                 BinaryOperator<U> combiner);
```

基本使用：

```java
public class StreamTest {
    public static void main(String[] args) {

        Stream<Integer> stream = Arrays.stream(new Integer[]{1, 2, 3, 4, 5});
        //求集合元素之和
        System.out.println(stream.reduce(0, Integer::sum));

        stream = Arrays.stream(new Integer[]{1, 2, 3, 4, 5});
        //求和
        stream.reduce((i, j) -> i + j).ifPresent(System.out::println);

        stream = Arrays.stream(new Integer[]{1, 2, 3, 4, 5});
        //求最大值
        stream.reduce(Integer::max).ifPresent(System.out::println);

        stream = Arrays.stream(new Integer[]{1, 2, 3, 4, 5});
        //求最小值
        stream.reduce(Integer::min).ifPresent(System.out::println);

        stream = Arrays.stream(new Integer[]{1, 2, 3, 4, 5});
        //做逻辑
        stream.reduce((i, j) -> i > j ? j : i).ifPresent(System.out::println);

        stream = Arrays.stream(new Integer[]{1, 2, 3, 4, 5});
        //求逻辑求乘机
        int result2 = stream.filter(i -> i % 2 == 0).reduce(1, (i, j) -> i * j);
        Optional.of(result2).ifPresent(System.out::println);
    }
}
```

## 1.5 总结

**中间操作和终端操作**

| **操作**    | **类型**          | **返回类型**  | **使用的类型/函数式接口** | **函数描述符**   |
| ----------- | ----------------- | ------------- | ------------------------- | ---------------- |
| *filter*    | 中间              | *Stream<T>*   | *Predicate<T>*            | *T -> boolean*   |
| *distinct*  | 中间(有状态-无界) | *Stream<T>*   |                           |                  |
| *skip*      | 中间(有状态-有界) | *Stream<T>*   | *long*                    |                  |
| *limit*     | *中间*            | *Strem<T>*    | *long*                    |                  |
| *map*       | 中间              | *Stream<R>*   | *Function<T,R>*           | *T -> R*         |
| *flatMap*   | 中间(有状态-无界) | *Stream<R>*   | *Function<T,Stream<R>>*   | *T -> Stream<R>* |
| *sorted*    | 终端              | *Stream<T>*   | *Comparator<T>*           | *(T, T) -> int*  |
| *anyMatch*  | 终端              | *boolean*     | *Predicate<T>*            | *T -> boolean*   |
| *noneMatch* | 终端              | *boolean*     | *Predicate<T>*            | *T -> boolean*   |
| *allMatch*  | 终端              | *boolean*     | *Predicate<T>*            | *T -> boolean*   |
| *findAny*   | 终端              | *Optional<T>* |                           |                  |
| *findFirst* | 终端              | *Optional<T>* |                           |                  |
| *forEach*   | 终端              | *void*        | *Consumer<T>*             | *T -> void*      |
| *collect*   | 终端              | *R*           | *Collector<T,A,R>*        |                  |
| *reduce*    | 终端(有状态-有界) | *Optional<T>* | *BinaryOperator<T>*       | *(T, T) -> T*    |
| *count*     | 终端              | *long*        |                           |                  |

# 2 Stream流原理剖析

 Stream 包是由下面的主要结构类组合而成的：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20210610112444122.png" width="500px"/>
</div>

`BaseStream` 和 Stream 为最顶端的接口类。`BaseStream` 主要定义了流的基本接口方法，例如，`spliterator、isParallel `等；Stream 则定义了一些流的常用操作方法，例如，map、filter 等。

`ReferencePipeline` 是一个结构类，他通过定义内部类组装了各种操作流。他定义了 Head、`StatelessOp`、`StatefulOp` 三个内部类，实现了 `BaseStream `与 Stream 的接口方法。

Sink 接口是定义每个 Stream 操作之间关系的协议，他包含 `begin()、end()、cancellationRequested()、accpt() `四个方法。`ReferencePipeline` 最终会将整个 Stream 流操作组装成一个调用链，而这条调用链上的各个 Stream 操作的上下关系就是通过 Sink 接口协议来定义实现的。

## 2.1 Stream 串行处理

一个 Stream 的各个操作是由处理管道组装，并统一完成数据处理的。在 JDK 中每次的中断操作会以使用阶段（Stage）命名。

管道结构通常是由 `ReferencePipeline` 类实现的，前面讲解 Stream 包结构时，我提到过 `ReferencePipeline` 包含了 `Head、StatelessOp、StatefulOp` 三种内部类。

Head 类主要用来定义数据源操作，在我们初次调用 `names.stream()` 方法时，会初次加载 Head 对象，此时为加载数据源操作；接着加载的是中间操作，分别为无状态中间操作 `StatelessOp` 对象和有状态操作 `StatefulOp` 对象，此时的 Stage 并没有执行，而是通过 `AbstractPipeline` 生成了一个中间操作 Stage 链表；当我们调用终结操作时，会生成一个最终的 Stage，通过这个 Stage 触发之前的中间操作，从最后一个 Stage 开始，递归产生一个 Sink 链。

```
	Sink1 ----> Sink2 ----> Sink3 ----> ReducingSink
```

这里举个例子：查找出一个长度最长，并且以张为姓氏的名字。从代码角度来看，你可能会认为是这样的操作流程：首先遍历一次集合，得到以“张”开头的所有名字；然后遍历一次 filter 得到的集合，将名字转换成数字长度；最后再从长度集合中找到最长的那个名字并且返回。

```java
List<String> names = Arrays.asList(" 张三 ", " 李四 ", " 王老五 ", 
                                   " 李三 ", " 刘老四 ", " 王小二 ", 
                                   " 张四 ", " 张五六七 ");
 
String maxLenStartWithZ = names.stream()
    	            .filter(name -> name.startsWith(" 张 "))
    	            .mapToInt(String::length)
    	            .max()
    	            .toString();
```

这里明确可以告知，==事实并非上述推测==。

首先 ，因为 names 是 `ArrayList` 集合，所以 `names.stream()` 方法将会调用集合类基础接口 Collection 的 Stream 方法：

```java
    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }
```

然后，Stream 方法就会调用 `StreamSupport` 类的 Stream 方法，方法中初始化了一个 `ReferencePipeline` 的 Head 内部类对象：

```java
    public static <T> Stream<T> stream(Spliterator<T> spliterator, boolean parallel) {
        Objects.requireNonNull(spliterator);
        return new ReferencePipeline.Head<>(spliterator,
                                            StreamOpFlag.fromCharacteristics(spliterator),
                                            parallel);
    }
```

再调用 filter 和 map 方法，这两个方法都是无状态的中间操作，所以执行 filter 和 map 操作时，并没有进行任何的操作，而是分别创建了一个 Stage 来标识用户的每一次操作。

而通常情况下 Stream 的操作又需要一个回调函数，所以一个完整的 Stage 是由数据来源、操作、回调函数组成的三元组来表示。如下所示，分别是 `ReferencePipeline` 的 filter 方法和 map 方法：

```java
    @Override
    public final Stream<P_OUT> filter(Predicate<? super P_OUT> predicate) {
        Objects.requireNonNull(predicate);
        return new StatelessOp<P_OUT, P_OUT>(this, StreamShape.REFERENCE,
                                     StreamOpFlag.NOT_SIZED) {
            @Override
            Sink<P_OUT> opWrapSink(int flags, Sink<P_OUT> sink) {
                return new Sink.ChainedReference<P_OUT, P_OUT>(sink) {
                    @Override
                    public void begin(long size) {
                        downstream.begin(-1);
                    }

                    @Override
                    public void accept(P_OUT u) {
                        if (predicate.test(u))
                            downstream.accept(u);
                    }
                };
            }
        };
    }

    @Override
    @SuppressWarnings("unchecked")
    public final <R> Stream<R> map(Function<? super P_OUT, ? extends R> mapper) {
        Objects.requireNonNull(mapper);
        return new StatelessOp<P_OUT, R>(this, StreamShape.REFERENCE,
                                     StreamOpFlag.NOT_SORTED | StreamOpFlag.NOT_DISTINCT) {
            @Override
            Sink<P_OUT> opWrapSink(int flags, Sink<R> sink) {
                return new Sink.ChainedReference<P_OUT, R>(sink) {
                    @Override
                    public void accept(P_OUT u) {
                        downstream.accept(mapper.apply(u));
                    }
                };
            }
        };
    }
```

`new StatelessOp` 将会调用父类 `AbstractPipeline` 的构造函数，这个构造函数将前后的 Stage 联系起来，生成一个 Stage 链表：

```java
    AbstractPipeline(AbstractPipeline<?, E_IN, ?> previousStage, int opFlags) {
        if (previousStage.linkedOrConsumed)
            throw new IllegalStateException(MSG_STREAM_LINKED);
        previousStage.linkedOrConsumed = true;
        previousStage.nextStage = this; // 将当前的 stage 的 next 指针指向之前的 stage

        this.previousStage = previousStage; // 赋值当前 stage 当全局变量 previousStage 
        this.sourceOrOpFlags = opFlags & StreamOpFlag.OP_MASK;
        this.combinedFlags = StreamOpFlag.combineOpFlags(opFlags,
                                                         previousStage.combinedFlags);
        this.sourceStage = previousStage.sourceStage;
        if (opIsStateful())
            sourceStage.sourceAnyStateful = true;
        this.depth = previousStage.depth + 1;
    }
```

因为在创建每一个 Stage 时，都会包含一个 `opWrapSink()` 方法，该方法会把一个操作的具体实现封装在 Sink 类中，Sink 采用（处理 -> 转发）的模式来叠加操作。

当执行 max 方法时，会调用 `ReferencePipeline` 的 max 方法，此时由于 max 方法是终结操作，所以会创建一个 `TerminalOp` 操作，同时创建一个 `ReducingSink`，并且将操作封装在 Sink 类中。

```java
    @Override
    public final Optional<P_OUT> max(Comparator<? super P_OUT> comparator) {
        return reduce(BinaryOperator.maxBy(comparator));
    }
```

最后，调用 `AbstractPipeline` 的 `wrapSink` 方法，该方法会调用 `opWrapSink` 生成一个 Sink 链表，Sink 链表中的每一个 Sink 都封装了一个操作的具体实现。

```java
@Override
@SuppressWarnings("unchecked")
final <P_IN> Sink<P_IN> wrapSink(Sink<E_OUT> sink) {
    Objects.requireNonNull(sink);

    for ( @SuppressWarnings("rawtypes") AbstractPipeline p=AbstractPipeline.this; 
         p.depth > 0; p=p.previousStage) {
        sink = p.opWrapSink(p.previousStage.combinedFlags, sink);
    }
    return (Sink<P_IN>) sink;
}
```

当 Sink 链表生成完成后，Stream 开始执行，通过 `spliterator` 迭代集合，执行 Sink 链表中的具体操作。

```java
    @Override
    final <P_IN, S extends Sink<E_OUT>> S wrapAndCopyInto(S sink, 
                                                          Spliterator<P_IN> spliterator) {
        copyInto(wrapSink(Objects.requireNonNull(sink)), spliterator);
        return sink;
    }

    @Override
    final <P_IN> void copyInto(Sink<P_IN> wrappedSink, Spliterator<P_IN> spliterator) {
        Objects.requireNonNull(wrappedSink);

        if (!StreamOpFlag.SHORT_CIRCUIT.isKnown(getStreamAndOpFlags())) {
            wrappedSink.begin(spliterator.getExactSizeIfKnown());
            spliterator.forEachRemaining(wrappedSink);
            wrappedSink.end();
        }
        else {
            copyIntoWithCancel(wrappedSink, spliterator);
        }
    }
```

Java8 中的 `Spliterator` 的 `forEachRemaining` 会迭代集合，每迭代一次，都会执行一次 filter 操作，如果 filter 操作通过，就会触发 map 操作，然后将结果放入到临时数组 object 中，再进行下一次的迭代。完成中间操作后，就会触发终结操作 max。

这就是串行处理方式了，那么 Stream 的另一种处理数据的方式又是怎么操作的呢？

## 2.2 Stream 并行处理

Stream 处理数据的方式有两种，串行处理和并行处理。要实现并行处理，我们只需要在例子的代码中新增一个 Parallel() 方法，代码如下所示：

```java
List<String> names = Arrays.asList(" 张三 ", " 李四 ", " 王老五 ", 
                                   " 李三 ", " 刘老四 ", " 王小二 ", 
                                   " 张四 ", " 张五六七 ");
 
String maxLenStartWithZ = names.stream()
                    .parallel()
    	            .filter(name -> name.startsWith(" 张 "))
    	            .mapToInt(String::length)
    	            .max()
    	            .toString();
```

Stream 的并行处理在执行终结操作之前，跟串行处理的实现是一样的。而在调用终结方法之后，实现的方式就有点不太一样，会调用 `TerminalOp` 的 `evaluateParallel` 方法进行并行处理。

```java
final <R> R evaluate(TerminalOp<E_OUT, R> terminalOp) {
    assert getOutputShape() == terminalOp.inputShape();
    if (linkedOrConsumed)
        throw new IllegalStateException(MSG_STREAM_LINKED);
    linkedOrConsumed = true;

    return isParallel()
           ? terminalOp.evaluateParallel(this, sourceSpliterator(terminalOp.getOpFlags()))
           : terminalOp.evaluateSequential(this, sourceSpliterator(terminalOp.getOpFlags()));
}
```

这里的并行处理指的是，Stream 结合了 `ForkJoin` 框架，对 Stream 处理进行了分片，`Splititerator` 中的 `estimateSize` 方法会估算出分片的数据量。

通过预估的数据量获取最小处理单元的阀值，如果当前分片大小大于最小处理单元的阀值，就继续切分集合。每个分片将会生成一个 Sink 链表，当所有的分片操作完成后，`ForkJoin` 框架将会合并分片任何结果集。

## 2.3 合理使用 Stream

我们将对常规的迭代、Stream 串行迭代以及 Stream 并行迭代进行性能测试对比，迭代循环中，我们将对数据进行过滤、分组等操作。分别进行以下几组测试：

- 多核 CPU 服务器配置环境下，对比长度 100 的 int 数组的性能；
- 多核 CPU 服务器配置环境下，对比长度 1.00E+8 的 int 数组的性能；
- 多核 CPU 服务器配置环境下，对比长度 1.00E+8 对象数组过滤分组的性能；
- 单核 CPU 服务器配置环境下，对比长度 1.00E+8 对象数组过滤分组的性能。

由于篇幅有限，我这里直接给出统计结果，你也可以自己去验证一下，具体的测试代码可以在[Github](https://github.com/nickliuchao/stream)上查看。通过以上测试，我统计出的测试结果如下（迭代使用时间）：

- 常规的迭代 <Stream 并行迭代 <Stream 串行迭代
- Stream 并行迭代 < 常规的迭代 <Stream 串行迭代
- Stream 并行迭代 < 常规的迭代 <Stream 串行迭代
- 常规的迭代 <Stream 串行迭代 <Stream 并行迭代

通过以上测试结果，我们可以看到：在循环迭代次数较少的情况下，常规的迭代方式性能反而更好；在单核 CPU 服务器配置环境中，也是常规迭代方式更有优势；而在大数据循环迭代中，如果服务器是多核 CPU 的情况下，Stream 的并行迭代优势明显。所以我们在平时处理大数据的集合时，应该尽量考虑将应用部署在多核 CPU 环境下，并且使用 Stream 的并行迭代方式进行处理。

用事实说话，我们看到其实使用 Stream 未必可以使系统性能更佳，还是要结合应用场景进行选择，也就是合理地使用 Stream。

## 2.4 小结

从大的设计方向上来说，Stream 将整个操作分解为了链式结构，不仅简化了遍历操作，还为实现了并行计算打下了基础。

从小的分类方向上来说，Stream 将遍历元素的操作和对元素的计算分为中间操作和终结操作，而中间操作又根据元素之间状态有无干扰分为有状态和无状态操作，实现了链结构中的不同阶段。

**在串行处理操作中，**Stream 在执行每一步中间操作时，并不会做实际的数据操作处理，而是将这些中间操作串联起来，最终由终结操作触发，生成一个数据处理链表，通过 Java8 中的 `Spliterator` 迭代器进行数据处理；此时，每执行一次迭代，就对所有的无状态的中间操作进行数据处理，而对有状态的中间操作，就需要迭代处理完所有的数据，再进行处理操作；最后就是进行终结操作的数据处理。

**在并行处理操作中，**Stream 对中间操作基本跟串行处理方式是一样的，但在终结操作中，Stream 将结合 `ForkJoin` 框架对集合进行切片处理，`ForkJoin` 框架将每个切片的处理结果 Join 合并起来。最后就是要注意 Stream 的使用场景。

# 参考资料

* [简洁方便的集合处理 Java 8 stream流](https://www.infoq.cn/article/vKZp6uczINXh3u9zOtDl)
* [Java8 Stream流学习笔记](https://www.jianshu.com/p/dda6ae5cf34e)
* [Java 函数式编程（三）流（Stream）](https://juejin.im/post/6844903662133706760)
* [Java 8 新特性 Stream类的collect方法](https://www.cnblogs.com/Deters/p/11137532.html)
* [Stream流](https://www.cnblogs.com/yulinfeng/p/12561664.html)