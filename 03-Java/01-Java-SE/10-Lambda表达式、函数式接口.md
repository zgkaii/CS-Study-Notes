## 一、Lambda表达式

### 1.1 函数式编程思想

在数学中，**函数**是输入元素的集合到可能的输出元素的集合之间的映射关系，并且每个输入元素只能映射到一个输出元素。比如典型的函数 `f(x)=x*x` 把所有实数的集合映射到其平方值的集合，如 `f(2)=4` 和 `f(-2)=4`。

而**函数式编程**则是一种编程范式。它把计算当成是数学函数的求值，从而避免改变状态和使用可变数据。它是一种声明式的编程范式，通过表达式和声明而不是语句来编程。

函数式编程有个重要的概念：**纯函数**。

纯函数需要具备两个特征：

- 对于相同的输入参数，总是返回相同的值（**引用透明**）。
- 求值过程中不产生**副作用**，也就是不会对运行环境产生影响。

对于第一个特征，如果是从数学概念上抽象出来的函数，则很容易理解。比如 `f(x)=x+1` 和 `g(x)=x*x` 这样的函数，都是典型的纯函数。可见，纯函数中不能使用静态局部变量、非局部变量，可变对象的引用或 I/O 流。这是因为这些变量的值可能在不同的函数执行中发生变化，导致产生不一样的输出。第二个特征，要求函数体中不能对静态局部变量、非局部变量，可变对象的引用或 I/O 流进行修改。这就保证了函数的执行过程中不会对外部环境造成影响。纯函数的这两个特征缺一不可。

```java
// 纯函数
int f1(int x) {
  return x + 1;
}

// 不是纯函数，因为引用了外部变量 y
int f2(int x) {
  return x + y;
}

// 不是纯函数，因为使用了调用了产生副作用的 Counter 对象的 inc 方法
int f3(Counter c) {
  c.inc();
  return 0;
}

// 不是纯函数，因为调用 writeFile 方法会写入文件，从而对外部环境造成影响
int f4(int x) {
  writeFile();
  return 1;
}
```

除此之外，函数式编程还强调**函数为“第一公民”**。所谓"第一等公民"（first class），指的是函数与其他数据类型一样，处于平等地位，它不仅拥有一切传统函数的使用方式（声明和调用），可以赋值给其他变量（赋值），也可以作为参数，传入另一个函数（传参），或者作为别的函数的返回值（返回）。函数可以作为参数进行传递，意味我们可以把行为"参数化"，处理逻辑可以从外部传入，这样程序就可以设计得更灵活。

与传统的命令式编程范式相比，函数式编程范式由于其函数天然的无状态特性，在并发编程中有着独特的优势。像多线程编程的问题根源在于对共享变量的并发访问。如果这样的访问并不需要存在，那么自然就不存在多线程相关的问题。在函数式编程范式中，函数中并不存在可变的状态，也就不需要对它们的访问进行控制。这就从根本上避免了多线程的问题。

当提到 Java 8 的时候，Lambda 表达式总是第一个提到的新特性。Lambda 表达式把函数式编程风格引入到了 Java 平台上，可以极大的提高 Java 开发人员的效率。

在兼顾面向对象特性的基础上，Java语言通过Lambda表达式与方法引用等，为开发者打开了函数式编程的大门。

### 1.2 体验Lambda表达式

#### （1）匿名内部类方式启动线程

当需要启动一个线程去完成任务时，通常会通过`java.lang.Runnable`接口来定义任务内容，并使用`java.lang.Thread`类来启动该线程。代码如下：

```java
public class RunnableTest {
    public static void main(String[] args) {
        new Thread(new Runnable() {
            // 重写抽象方法
            @Override
            public void run() {
                System.out.println("Hello World!");
            }
        }).start();// 启动线程
    }
}
```

**代码分析**

对于`Runnable`的匿名内部类用法，可以分析出几点内容：

- `Thread`类需要`Runnable`接口作为参数，其中的抽象`run`方法是用来指定线程任务内容的核心；
- 为了指定`run`的方法体，**不得不**需要`Runnable`接口的实现类；
- 为了省去定义一个`RunnableImpl`实现类的麻烦，**不得不**使用匿名内部类；
- 必须覆盖重写抽象`run`方法，所以方法名称、方法参数、方法返回值**不得不**再写一遍，且不能写错；
- 而实际上，**只有方法体才是关键所在**。

#### （2）Lambda表达式启动线程

借助Java 8的全新语法，上述`Runnable`接口的匿名内部类写法可以这样书写：

```java
public class LambdaTest {
    public static void main(String[] args) {
        new Thread(() -> System.out.println("Hello World!")).start();
    }
}
```

这段代码和刚才的执行效果是完全一样的，可以在1.8或更高的编译级别下通过。从代码的语义中可以看出：我们启动了一个线程，而线程任务的内容以一种更加简洁的形式被指定。

不再有“不得不创建接口对象”的束缚，不再有“抽象方法覆盖重写”的负担，就是这么简单！

### 1.3 格式及使用

#### （1）Lambda表达式格式

Lambda表达式省去面向对象中的条条框框，由**3个部分**组成：

* 一些参数
* 一个箭头
* 一段代码

Lambda表达式的**标准格式**为：

```
(参数类型 参数名称) -> { 代码语句 }
```

格式说明：

* 小括号内的语法与传统方法参数列表一致：无参数则留空；多个参数则用逗号分隔。
* `->`是新引入的语法格式，代表指向动作。
* 大括号内的语法与传统方法体要求基本一致。

以下是 Lambda 表达式的一些示例：

```java
(int a, int b) -> {  return a + b; }

() -> System.out.println("Hello World");

(String s) -> { System.out.println(s); }

() -> 42

() -> { return 3.1415 };
```

#### （2）Lambda的使用前提

虽然使用 Lambda 表达式可以对某些接口进行简单的实现，语法非常简洁，完全没有面向对象编程复杂的束缚。但是使用时有几个问题需要特别注意：

1. 使用Lambda必须具有接口，且要求**接口中有且仅有一个方法需要被实现**。

   无论是JDK内置的`Runnable`、`Comparator`接口还是自定义的接口，只有当接口中的抽象方法存在且唯一时，才可以使用Lambda。

2. 使用Lambda必须具有**上下文推断**。
   也就是方法的参数或局部变量类型必须为Lambda对应的接口类型，才能使用Lambda作为该接口的实例。

> Tips：有且仅有一个抽象方法的接口，称为“**函数式接口**”。

可以使用`@FunctionalInterface`来修饰函数式接口，要求接口中的抽象方法只有一个。 

#### （3）函数式接口

**函数式接口**就是：有且仅有一个抽象方法的接口。

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

#### （4）Lambda基本使用

我们这里给出六个接口。

```java
/**多参数无返回*/
@FunctionalInterface
public interface NoReturnMultiParam {
    void method(int a, int b);
}

/**无参无返回值*/
@FunctionalInterface
public interface NoReturnNoParam {
    void method();
}

/**一个参数无返回*/
@FunctionalInterface
public interface NoReturnOneParam {
    void method(int a);
}

/**多个参数有返回值*/
@FunctionalInterface
public interface ReturnMultiParam {
    int method(int a, int b);
}

/**无参有返回*/
@FunctionalInterface
public interface ReturnNoParam {
    int method();
}

/**一个参数有返回值*/
@FunctionalInterface
public interface ReturnOneParam {
    int method(int a);
}
```

使用Lambda表达式：

```java
public class LambdaTest1 {
    public static void main(String[] args) {
        //无参无返回
        NoReturnNoParam noReturnNoParam = () -> {
            System.out.println("无参无返回");
        };
        noReturnNoParam.method();

        //一个参数无返回
        NoReturnOneParam noReturnOneParam = (int a) -> {
            System.out.println("一个参数无返回:" + a);
        };
        noReturnOneParam.method(6);

        //多个参数无返回
        NoReturnMultiParam noReturnMultiParam = (int a, int b) -> {
            System.out.println("多个参数无返回:" + "{" + a + "," + +b + "}");
        };
        noReturnMultiParam.method(6, 8);

        //无参有返回值
        ReturnNoParam returnNoParam = () -> {
            System.out.print("无参有返回值");
            return 1;
        };
        int res = returnNoParam.method();
        System.out.println("return:" + res);

        //一个参数有返回值
        ReturnOneParam returnOneParam = (int a) -> {
            System.out.println("一个参数有返回值:" + a);
            return 1;
        };
        int res2 = returnOneParam.method(6);
        System.out.println("return:" + res2);

        //多个参数有返回值
        ReturnMultiParam returnMultiParam = (int a, int b) -> {
            System.out.println("多个参数有返回值:" + "{" + a + "," + b + "}");
            return 1;
        };
        int res3 = returnMultiParam.method(6, 8);
        System.out.println("return:" + res3);
    }
}
```

**Lambda表达式简写规则**为：

1. 小括号内参数的类型可以省略；
2. 如果小括号内**有且仅有一个参**，则小括号可以省略；
3. 如果大括号内**有且仅有一个语句**，则无论是否有返回值，都可以省略大括号、return关键字及语句分号。

```java
public class LambdaTest2 {
    public static void main(String[] args) {

        //1.简化参数类型，可以不写参数类型，但是必须所有参数都不写
        NoReturnMultiParam lamdba1 = (a, b) -> {
            System.out.println("简化参数类型");
        };
        lamdba1.method(1, 2);

        //2.简化参数小括号，如果只有一个参数则可以省略参数小括号
        NoReturnOneParam lambda2 = a -> {
            System.out.println("简化参数小括号");
        };
        lambda2.method(1);

        //3.简化方法体大括号，如果方法条只有一条语句，则可以胜率方法体大括号
        NoReturnNoParam lambda3 = () -> System.out.println("简化方法体大括号");
        lambda3.method();

        //4.如果方法体只有一条语句，并且是 return 语句，则可以省略方法体大括号
        ReturnOneParam lambda4 = a -> a + 3;
        System.out.println(lambda4.method(5));

        ReturnMultiParam lambda5 = (a, b) -> a + b;
        System.out.println(lambda5.method(1, 1));
    }
}
```

#### （5）方法引用

双冒号（`::`）操作符是 Java 中的**方法引用**。 当们使用一个方法的引用时，目标引用放在 `::` 之前，目标引用提供的方法名称放在 `::` 之后，即 `目标引用::方法`。

```java
public class LambdaTest3 {
    public static void main(String[] args) {
        ReturnMultiParam lambda1 = (a,b)-> addTwo(a,b);
        System.out.println(lambda1.method(10,20));

        //lambda2 引用了doubleNum方法
        ReturnOneParam lambda2 = LambdaTest3::doubleNum;
        System.out.println(lambda2.method(3));
    }

    /**
     * 要求
     * 1.参数数量和类型要与接口中定义的一致
     * 2.返回值类型要与接口中定义的一致
     */
    public static int doubleNum(int a) {
        return a * 3;
    }

    public static int addTwo(int a, int b) {
        return a + b;
    }
}
```

- **构造方法的引用**

一般我们需要声明接口，该接口作为对象的生成器，通过 类名::new 的方式来实例化对象，然后调用方法返回对象。

```java
public interface ItemCreatorBlankConstruct {
    Item getItem();
}
```

```java
public interface ItemCreatorParamContruct {
    Item getItem(int id, String name, double price);
}
```

```java
public class Item {
    private int id;
    private String name;
    private double price;

    public Item(int id, String name, double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }

    public Item() {

    }

    @Override
    public String toString() {
        return "Item{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", price=" + price +
                '}';
    }
}
```

```java
public class LambdaTest4 {
    public static void main(String[] args) {

         new ItemCreatorBlankConstruct() {
            @Override
            public Item getItem() {
                return null;
            }
        };

        ItemCreatorBlankConstruct creator1 = () -> new Item();
        System.out.println(creator1.getItem());

        ItemCreatorBlankConstruct creator2 = Item::new;
        System.out.println(creator2.getItem());

        ItemCreatorParamContruct creator3 = Item::new;
        System.out.println(creator3.getItem(100, "手机", 4000));
    }
}
```

### 1.4 常见使用场景

#### （1）线程初始化

```java
new Thread(
    () -> System.out.println("Hello world")
).start();
```

#### （2）遍例集合

```java
// old way
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7);
for (Integer n : list) {
    System.out.println(n);
}

// 使用 -> 的 Lambda 表达式
list.forEach(n -> System.out.println(n));

// 使用 :: 的 Lambda 表达式（方法引用）
list.forEach(System.out::println);
```

#### （3）元素排序

```java
 ArrayList<Item> list = new ArrayList<>();
        list.add(new Item(13, "背心", 7.80));
        list.add(new Item(11, "半袖", 37.80));
        list.add(new Item(14, "风衣", 139.80));
        list.add(new Item(12, "秋裤", 55.33));

		// old way
        list.sort(new Comparator<Item>() {
            @Override
            public int compare(Item o1, Item o2) {
                return o1.getId()  - o2.getId();
            }
        });
		// 使用 -> 的 Lambda 表达式
        list.sort((o1, o2) -> o1.getId() - o2.getId());

        System.out.println(list);
```

#### （4）事件处理

```java
// old way
JButton show =  new JButton("Show");
show.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
    System.out.println("old way");
    }
}); 

// 使用 -> 的 Lambda 表达式
show.addActionListener((e) -> {
    System.out.println("使用 -> 的 Lambda 表达式");
});
```

## 二、常用函数式接口

JDK在`ava.util.function`包提供了大量常用的函数式接口以丰富Lambda的典型使用场景，主要嘘唏是Consumer（消费型）、supplier（供给型）、predicate（谓词型）、function（功能性）接口。

### 2.1 Supplier接口

`java.util.function.Supplier<T>`接口仅包含一个无参的方法：`T get()`，用来获取一个泛型参数指定类型的对
象数据。

为实现 `Supplier` 接口，需要提供一个不传入参数且返回泛型类型（generic type）的方法。根据 Javadoc 的描述，调用 `Supplier` 时**，不要求每次都返回一个新的或不同的结果**。

`Supplier` 的一个主要用例是**延迟执行**（deferred execution）。`java.util.logging.Logger` 类定义的 `info` 方法传入 `Supplier`，仅当日志级别（log level）控制日志消息可见时，才调用其 get 方法。目前，`Logger` 类中的日志方法（如 `info`、`warning`、`severe` 等）包括两种重载形式，一种传入单个 `String` 作为参数，另一种传入 `Supplier<String>` 作为参数。。

`Logger` 类传入 `Supplier`的重载形式比较：

```java
	//传入单个String    
	void finer(String msg)
    //传入 Supplier<String> 
    void finer(Supplier<String> msgSupplier)
```

其实现细节：

```java
    //传入单个String        
    public void info(Supplier<String> msgSupplier) {
            log(Level.INFO, msgSupplier);
        }
	//传入 Supplier<String>
    public void log(Level level, Supplier<String> msgSupplier) {
        if (!isLoggable(level)) {// 如果不显示消息则返回
            return;
        }
        LogRecord lr = new LogRecord(level, msgSupplier.get());//调用 get 方法以便在 Supplier 中检索消息
        doLog(lr);
    }
```

 `info` 方法中使用 `Supplier`：

```java
public class SupplierTest {
    public static void main(String[] args) {
        Logger logger = getLogger(SupplierTest.class.getName());
        List<String> data = new ArrayList<>();
        data.add("张三");
        data.add("李四");
        
        logger.info("The data is " + data.toString());//无论是否显示 Info 消息，都会构建参数
        logger.info(() -> "The data is " + data.toString());//仅当日志级别显示 Info 消息时，才会构建参数
    }
}
```

### 2.2 Consumer接口

`java.util.function.Consumer<T>`接口则正好与Supplier接口相反，它不是生产一个数据，而是消费一个数据，
其数据类型由泛型决定。

**抽象方法：accept**

Consumer 接口中包含抽象方法 `void accept(T t)` ，意为消费一个指定泛型的数据。基本使用如：

```java
public class ConsumerTest {
    private static void consumeString(Consumer<String> function) {
        function.accept("Hello World!");
    }
    public static void main(String[] args) {
        consumeString(s -> System.out.println(s));
    }
}
```
当然，更好的写法是使用方法引用。

**默认方法：andThen**

如果一个方法的参数和返回值全都是Consumer类型，那么就可以实现效果：消费数据的时候，首先做一个操作，
然后再做一个操作，实现组合。而这个方法就是Consumer接口中的default方法andThen。下面是JDK的源代码：

```java
    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
```

>Tips：java.util.Objects的requireNonNull静态方法将会在参数为null时主动抛出NullPointerException异常。这省去了重复编写if语句和抛出空指针异常的麻烦。

要想实现组合，需要两个或多个Lambda表达式即可，而andThen的语义正是“一步接一步”操作。例如两个步骤组合的情况：

```java
public class ConsumerTest {
    private static void consumeString(Consumer<String> one, Consumer<String> two) {
        one.andThen(two).accept("Hello World!");
    }

    public static void main(String[] args) {
        consumeString(
                s -> System.out.println(s.toUpperCase()),
                s -> System.out.println(s.toLowerCase()));
    }
}
```
运行结果将会首先打印完全大写的HELLO，然后打印完全小写的hello。当然，通过链式写法可以实现更多步骤的组合。

**forEach**：

在所有传入 `Consumer` 作为参数的方法中，最常见的是 `java.lang.Iterable` 接口的默认 `forEach` 方法：

```java
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
```

打印集合中的元素：

```java
public class ConsumerTest {
    public static void main(String[] args) {
        List<String> strings = Arrays.asList("this", "is", "a", "list", "of", "strings");

        strings.forEach(new Consumer<String>() {//匿名内部类实现
            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        });

        strings.forEach(s -> System.out.println(s));//lambda 表达式
        strings.forEach(System.out::println);//方法引用
    }
}
```

### 2.3 Predicate接口

有时候我们需要对某种类型的数据进行判断，从而得到一个boolean值结果。这时可以使用`java.util.function.Predicate<T>`接口。

**抽象方法：test**

Predicate接口中包含一个抽象方法：`boolean test(T t)`。用于条件判断的场景：

```java
public class PredicateTest {
    public static void method(Predicate<String> predicate){
        boolean helloWorld = predicate.test("HelloWorld");
        System.out.println("字符串长吗：" + helloWorld);
    }

    public static void main(String[] args) {
        method(s -> s.length() > 5);
    }
}
```

条件判断的标准是传入的Lambda表达式逻辑，只要字符串长度大于 5 则认为很长。

**默认方法：and**

既然是条件判断，就会存在与、或、非三种常见的逻辑关系。其中将两个Predicate条件使用“与”逻辑连接起来实
现“**与**”的效果时，可以使用default方法and。其JDK源码为：

```java
    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }
```

如果要判断一个字符串既要包含大写“H”，又要包含小写“r”，那么：

```java
public class PredicateTest {java
    private static void method(Predicate<String> one, Predicate<String> two) {
        boolean isValid = one.and(two).test("HelloWorld");
        System.out.println("字符串符合要求吗：" + isValid);
    }

    public static void main(String[] args) {
        method(s -> s.contains("H"), s -> s.contains("r"));
    }
}
```

**默认方法：or**

与and的“与”类似，默认方法or实现逻辑关系中的“**或**”。JDK源码为：

```java
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }
```
如果希望实现逻辑“字符串包含大写H或者包含大写W”，那么代码只需要将“and”修改为“or”名称即可，其他都不变：

```java
boolean isValid = one.or(two).test("HelloWorld");
```

**默认方法：negate**

“与”、“或”已经了解了，剩下的“**非**”（取反）也会简单。默认方法negate的JDK源代码为：

```java
default Predicate<T> negate() {
        return (t) -> !test(t);
    }
```

从实现中很容易看出，它是执行了test方法之后，对结果boolean值进行“!”取反而已。**一定要在test方法调用之前调用negate方法**，正如and和or方法一样：

```java
public class PredicateTest {
    public static void method(Predicate<String> predicate) {
        boolean helloWorld = predicate.negate().test("HelloWorld");
        System.out.println("字符串长吗：" + helloWorld);
    }

    public static void main(String[] args) {
        method(s -> s.length() < 5);
    }
}
```

### 2.4 Function接口

`java.util.function.Function<T,R>`接口用来根据一个类型的数据得到另一个类型的数据，前者称为前置条件，后者称为后置条件。

**抽象方法：apply**

Function接口中最主要的抽象方法为：`R apply(T t)`，根据类型T的参数获取类型R的结果。

使用的场景例如：将String类型转换为Integer类型。

```java
public class FunctionTest {
    public static void method(Function<String, Integer> function) {
        int num = function.apply("10");
        System.out.println(num + 20);

    }

    public static void main(String[] args) {
        method(s -> Integer.parseInt(s));
    }
}
```

**默认方法：andThen**

Function接口中有一个默认的`andThen`方法，用来进行组合操作。JDK源代码如：

```java
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }
```
该方法同样用于“先做什么，再做什么”的场景，和Consumer中的`andThen`方法差不多：

```java
public class FunctionTest {
   private static void method(Function<String, Integer> one, Function<Integer, Integer> two) {
        int num = one.andThen(two).apply("10");
        System.out.println(num + 20);
    }

    public static void main(String[] args) {
        method(str -> Integer.parseInt(str) + 10, i -> i *= 10);
    }
}
```

第一个操作是将字符串解析成为`int`数字，第二个操作是乘以 10 。两个操作通过`andThen`按照前后顺序组合到了一起。

>Tips：Function的前置条件泛型和后置条件泛型可以相同。

## 参考资料

[理解函数式编程](https://wudaijun.com/2018/05/understand-functional-programing/)

[Lambda表达式详解](https://www.cnblogs.com/haixiang/p/11029639.html)

[Java8-方法引用详解](https://segmentfault.com/a/1190000012269548)

[函数式接口](https://geek-docs.com/java/java-examples/java-consumer-interface.html)