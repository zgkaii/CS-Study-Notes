在Java 8中，得益于Lambda所带来的函数式编程，引入了一个全新的Stream概念，用于解决已有集合类库既有的弊端。Stream流尽管被称作为“流”，但它和文件流、字符流、字节流**完全没有任何关系**。

### 1.1 引言

**传统集合的多步遍历代码**

几乎所有的集合（如Collection接口或Map接口等）都支持直接或间接的遍历操作。而当我们需要对集合中的元素进行操作的时候，除了必需的添加、删除、获取外，最典型的就是集合遍历。例如：

```java
public class ForEachTest {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("张三");
        list.add("李四");
        list.add("王五");
        list.add("王二");
        list.add("王二狗");
        
        for (String name : list) {
            System.out.println(name);
        }
    }
}
```
这是一段非常简单的集合遍历操作：对集合中的每一个字符串都进行打印输出操作。

**循环遍历的弊端**

试想一下，如果希望对集合中的元素进行筛选过滤：

```shell
1. 将集合A根据条件一过滤为子集B；
2. 然后再根据条件二过滤为子集C。
```
那怎么办？在Java 8之前的做法可能为：

```java
public class ForEachTest {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("张三");
        list.add("李四");
        list.add("王五");
        list.add("王二");
        list.add("王二狗");

        List<String> List2 = new ArrayList<>();
        for (String name : list) {
            if (name.startsWith("王")) {
                List2.add(name);
            }
        }
        
        List<String> shortList = new ArrayList<>();
        for (String name : List2) {
            if (name.length() == 2) {
                shortList.add(name);
            }
        }

        for (String name : shortList) {
            System.out.println(name);
        }
    }
}
```
这段代码中含有三个循环，每一个作用不同：

1. 首先筛选所有姓王的人；
2.  然后筛选名字有两个字的人； 
3.  最后进行对结果进行打印输出。

每当我们需要对集合中的元素进行操作的时候，总是需要进行循环、循环、再循环，很不优雅。 那，Lambda的衍生物Stream能给我们带来怎样更加优雅的写法呢？

**Stream的更优写法**

```java
public class ForEachTest {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("张三");
        list.add("李四");
        list.add("王五");
        list.add("王二");
        list.add("王二狗");

        list.stream()
                .filter(s -> s.startsWith("王"))
                .filter(s -> s.length() == 2)
                .forEach(System.out::println);
    }
}
```

直接阅读代码的字面意思即可完美展示无关逻辑方式的语义：获取流、过滤王姓、过滤长度为2、逐一打印。代码中并没有体现使用线性循环或是其他任何算法进行遍历，我们真正要做的事情内容被更好地体现在代码中。

### 1.2 Stream流概述

整体来看，Stream流类似于工厂车间的“生产流水线**”，只需要把集合、命令还有一些参数灌输到**流水线中去，就可以加工成得出想要的结果。

```shell
原集合 —> 获取流 —> 中间操作 (过滤、分组、统计) —> 终端操作  
```

当**使用流**的时候，通常有以下几个步骤：集合→**获取流**→**中间操作**（比如过滤、筛选、分组、计算）获取想要的结果→**终端操作**（就是转化成我们想要的数据，这个数据的形式一般还是集合，有时也会按照需求输出 count 计数。）

> Tips：“Stream流”其实是一个集合元素的函数模型，它并不是集合，也不是数据结构，其本身并不存储任何元素或其地址。

可以这把Stream流理解为一个来自数据源的元素队列并支持聚合操作。

Stream（流）是一个来自数据源的元素队列：

* **元素**：是特定类型的对象，形成一个队列。 Java中的Stream并不会存储元素，而是按需计算。
* **数据源**：流的来源。 可以是集合，数组，I/O channel，产生器 generator 等。
* **聚合操作**：类似 SQL 语句一样的操作，比如 filter, map, reduce, find, match, sorted 等。

和Collection操作不同， Stream操作还有两个基础的特征：

* **Pipelining**：中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluentstyle）。 这样做可以对操作进行优化， 比如延迟执行（laziness）和短路（short-circuiting）。
* **内部迭代**： 以前对集合遍历都是通过Iterator或者foreach的方式，在集合外部进行外部迭代。 Stream提供了内部迭代的方式，流可以直接调用遍历方法。

### 1.3 获取流

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

#### （1）Collection获取流

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

#### （2）Map获取流

`java.util.Map`接口不是Collection的子接口，且其`K-V`数据结构不符合流元素的单一特征，所以获取对应的流
需要分key、value或entry等情况。

```java
        Map<String, String> map = new HashMap<>();
        // ...
        Stream<String> keyStream = map.keySet().stream();
        Stream<String> valueStream = map.values().stream();
        Stream<Map.Entry<String, String>> entryStream = map.entrySet().stream();
```

#### （3）数组获取流

如果使用的是数组，由于数组对象不可能添加默认方法，所以Stream接口中提供了静态方法`of`，使用很简单：

```java
        String[] array = { "张三", "李四", "王五"};
        Stream<String> stream = Stream.of(array);
```


>Tips：of方法的参数其实是一个可变参数，所以支持数组。

### 1.3 中间操作

常用方法：

|   操作   |        类型         | 返回类型  |    操作参数     |  函数描述符   |
| :------: | :-----------------: | :-------: | :-------------: | :-----------: |
|  filter  |        中间         | Stream<T> |  Predicate<T>   | T -> boolean  |
|   map    |        中间         | Stream<R> | Funcation<T，R> |    T -> R     |
|  limit   | 中间（有状态-有界） | Stream<T> |                 |               |
|  sorted  |        中间         | Stream<T> |  Comparator<T>  | (T，T) -> int |
| distinct | 中间（有状态-无界） | Stream<T> |                 |               |
|   skip   | 中间（有状态-有界） | Stream<T> |      long       |               |

#### （1）filter 过滤

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

#### （2）map 映射

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

#### （3）limit 截取

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

#### （4）sorted 排序

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

#### （5）distinct 去重

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

#### （6）skip 跳过

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

### 1.4 终端操作

|  操作   |        类型         |  返回类型   |      操作参数      |  函数描述符   |
| :-----: | :-----------------: | :---------: | :----------------: | :-----------: |
| forEach |        终端         |    void     |    Consumer<T>     |   T -> void   |
| collect |        终端         |      R      | Collector<T，A，R> |               |
| reduce  | 终端（有状态-有界） | Optional<T> |   BinaryOperator   | （T，T） -> T |
|  count  |        终端         |    long     |                    |               |

#### （1）forEach

虽然方法名字叫 forEach ，但是与for循环中的“for-each”昵称不同。方法签名：

```java
void forEach(Consumer<? super T> action);
```

该方法接收一个 Consumer 接口函数，会将每一个流元素交给该函数进行处理。基本使用：

```java
        Stream<String> stream = Stream.of("张三", "李四", "王五");
		stream.forEach(name-> System.out.println(name));
```

#### （2）collect

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

#### （3）count

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

#### （4）reduce

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

### 1.5 总结

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

## 参考资料

* [简洁方便的集合处理 Java 8 stream流](https://www.infoq.cn/article/vKZp6uczINXh3u9zOtDl)
* [Java8 Stream流学习笔记](https://www.jianshu.com/p/dda6ae5cf34e)
* [Java 函数式编程（三）流（Stream）](https://juejin.im/post/6844903662133706760)
* [Java 8 新特性 Stream类的collect方法](https://www.cnblogs.com/Deters/p/11137532.html)
* [Stream流](https://www.cnblogs.com/yulinfeng/p/12561664.html)