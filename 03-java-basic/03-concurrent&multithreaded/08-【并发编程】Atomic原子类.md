## 1 Atomic原子类简介

原子（atomic）本意是“不能被进一步分割的最小粒子”，而原子操作（atomic operation）意为“不可被中断的一个或一系列操作”。

即使是在多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程干扰。简而言之，**原子类就是具有原子/原子操作特征的类，它们能无锁地避免原子性问题**。

并发包 `java.util.concurrent` 的原子类都存放在`java.util.concurrent.atomic`下，如下图所示。

![](https://img-blog.csdnimg.cn/20201105205300727.png)

根据操作的数据类型，可以将JUC包中的原子类分为主要5类：

**（1）基本类型**

- AtomicInteger：整型原子类
- AtomicLong：长整型原子类
-  AtomicBoolean ：布尔型原子类

**（2）数组类型**


- AtomicIntegerArray：整型数组原子类
- AtomicLongArray：长整型数组原子类
- AtomicReferenceArray ：引用类型数组原子类

**（3）引用类型**

- AtomicReference：引用类型原子类
- AtomicMarkableReference：原子更新带有标记的引用类型。
- AtomicStampedReference ：原子更新带有版本号的引用类型。

**（4）对象的属性修改类型**

- AtomicIntegerFieldUpdater：原子更新整型字段的更新器
- AtomicLongFieldUpdater：原子更新长整型字段的更新器
- AtomicReferenceFieldUpdater：原子更新引用类型里的字段

**（5）原子累加器**

- DoubleAccumulator
- DoubleAdder
- LongAccumulator
- LongAdder

### 1.1 基本类型

基本类型原子类有三个：`AtomicInteger`、`AtomicLong`与`AtomicBoolean `。介于三个类提供的方法几乎相同，这里以整型原子类` AtomicInteger`为例来学习。

 AtomicInteger 类常用方法有：

| 方法                                            | 描述                                                         |
| ----------------------------------------------- | ------------------------------------------------------------ |
| `get()`                                         | 获取当前的值                                                 |
| `getAndSet(int newValue)`                       | 获取当前的值，并设置新的值                                   |
| `getAndIncrement()`                             | 获取当前的值，并自增                                         |
| `getAndDecrement()`                             | 获取当前的值，并自减                                         |
| `getAndAdd(int delta)`                          | 获取当前的值，并加上预期的值                                 |
| `boolean compareAndSet(int expect, int update)` | 如果输入的数值等于预期值，则以原子方式将该值设置为输入值（update） |
| `lazySet(int newValue)`                         | 最终设置为newValue,使用 lazySet 设置之后可能导致其他线程在之后的一小段时间内还是可以读到旧的值。 |

代码示例：

```java
public class AtomicIntegerTest1 {
    public static void main(String[] args) {
        int temValue = 0;
        AtomicInteger i = new AtomicInteger(0);
        temValue = i.getAndSet(12);
        System.out.println("temValue:" + temValue + ";  i:" + i);
        temValue = i.getAndIncrement();
        System.out.println("temValue:" + temValue + ";  i:" + i);
        temValue = i.getAndAdd(-10);
        System.out.println("temValue:" + temValue + ";  i:" + i);
    }
}
```

执行结果：

```java
temValue:0;  i:12
temValue:12;  i:13
temValue:13;  i:3
```

### 1.2 数组类型

 数组类型原子类有三个：`AtomicIntegerArray`、`AtomicLongArray`与`AtomicReferenceArray `。介于三个类提供的方法几乎相同，这里以 `AtomicIntegerArray`为例来学习。

**AtomicIntegerArray常用方法**

| 方法                                           | 描述                                                         |
| ---------------------------------------------- | ------------------------------------------------------------ |
| `get(int i)`                                   | 获取 index=i 位置元素的值                                    |
| `getAndSet(int i, int newValue)`               | 返回 index=i 位置的当前的值，并将其设置为新值：newValue      |
| `getAndIncrement(int i)`                       | 获取 index=i 位置元素的值，并让该位置的元素自增              |
| `getAndDecrement(int i)`                       | 获取 index=i 位置元素的值，并让该位置的元素自减              |
| `getAndAdd(int i, int delta)`                  | 获取 index=i 位置元素的值，并加上预期的值                    |
| `compareAndSet(int i, int expect, int update)` | 如果输入的数值等于预期值，则以原子方式将 index=i 位置的元素值设置为输入值（update） |
| `lazySet(int i, int newValue)`                 | 最终将index=i 位置的元素设置为newValue,使用 lazySet 设置之后可能导致其他线程在之后的一小段时间内还是可以读到旧的值。 |

代码示例：

```java
public class AtomicIntegerArrayTest {
    public static void main(String[] args) {
        int temValue = 0;
        int[] nums = {-2, -1, 1, 2, 3, 4};
        AtomicIntegerArray i = new AtomicIntegerArray(nums);
        for (int j = 0; j < nums.length; j++) {
            System.out.println(i.get(j));
        }
        temValue = i.getAndSet(0, 2);
        System.out.println("temValue:" + temValue + ";  i:" + i);
        temValue = i.getAndIncrement(0);
        System.out.println("temValue:" + temValue + ";  i:" + i);
        temValue = i.getAndAdd(0, 5);
        System.out.println("temValue:" + temValue + ";  i:" + i);
    }
}
```

执行结果：

```java
-2
-1
1
2
3
4
temValue:-2;  i:[2, -1, 1, 2, 3, 4]
temValue:2;  i:[3, -1, 1, 2, 3, 4]
temValue:3;  i:[8, -1, 1, 2, 3, 4]

Process finished with exit code 0
```

### 1.3 对象的属性修改类型

对象的属性修改类型原子类有`AtomicIntegerFieldUpdater`、`AtomicLongFieldUpdater`、`AtomicReferenceFieldUpdater `三个。

介于三个类提供的方法几乎相同，这里以 `AtomicIntegerFieldUpdater `为例来学习。

子地更新对象的属性需要两步：

* 因为对象的属性修改类型原子类都是抽象类，所以每次使用都必须使用静态方法**newUpdater()**创建一个更新器，并且需要设置想要更新的类和属性。

* 更新的对象属性必须使用**public volatile修饰符**。

`AtomicIntegerFieldUpdater`类使用示例

```java
public class AtomicIntegerFieldUpdaterTest {
	public class AtomicIntegerFieldUpdaterTest {
    public static void main(String[] args) {
        AtomicIntegerFieldUpdater<User> a = AtomicIntegerFieldUpdater.newUpdater(User.class, "age");

        User user = new User("ZhangSan", 20);
        System.out.println(a.getAndIncrement(user));
        System.out.println(a.get(user));
    }
}

class User {
    private String name;
    public volatile int age;

    public User(String name, int age) {
        super();
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
}
```

输出结果：

```java
20
21
```

### 1.4  引用类型

基本类型原子类只能更新一个变量，如果需要原子**更新多个变量**，需要使用引用类型原子类。引用类型原子类分为`AtomicReference`、`AtomicStampedReference`、`AtomicMarkableReference `三类。

* AtomicReference：引用类型原子类

- AtomicStampedReference：原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于解决原子的更新数据和数据的版本号，可以解决使用 CAS 进行原子更新时可能出现的 ABA 问题。
- AtomicMarkableReference ：原子更新带有标记的引用类型。该类将 boolean 标记与引用关联起来，~~也可以解决使用 CAS 进行原子更新时可能出现的 ABA 问题。~~

（1）`AtomicReference类`使用示例：

```java
public class AtomicReferenceTest {
    public static void main(String[] args) {
        AtomicReference<Person> personAtomicReference = new AtomicReference<>();
        Person person = new Person("Zhangsan", 20);
        personAtomicReference.set(person);
        Person updatePerson = new Person("LiSi", 30);
        personAtomicReference.compareAndSet(person, updatePerson);

        System.out.println(personAtomicReference.get().getName());
        System.out.println(personAtomicReference.get().getAge());
    }
}

class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        super();
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
}
```

上述代码首先创建了一个 Person 对象，然后把 Person 对象设置进 AtomicReference 对象中，然后调用 compareAndSet 方法，该方法就是通过 CAS 操作设置 personAtomicReference。如果 personAtomicReference的值为 person 的话，则将其设置为 updatePerson。实现原理与 AtomicInteger 类中的 compareAndSet 方法相同。

输出结果：

```java
LiSi
30
```

（2） `AtomicStampedReference`类使用示例

```java
public class AtomicStampedReferenceDemo {
    public static void main(String[] args) {
        // 实例化、取当前值和 stamp 值
        final Integer initialRef = 0, initialStamp = 0;
        final AtomicStampedReference<Integer> asr = new AtomicStampedReference<>(initialRef, initialStamp);
        System.out.println("currentValue=" + asr.getReference() + ", currentStamp=" + asr.getStamp());

        // compare and set
        final Integer newReference = 666, newStamp = 999;
        final boolean casResult = asr.compareAndSet(initialRef, newReference, initialStamp, newStamp);
        System.out.println("currentValue=" + asr.getReference()
                + ", currentStamp=" + asr.getStamp()
                + ", casResult=" + casResult);

        // 获取当前的值和当前的 stamp 值
        int[] arr = new int[1];
        final Integer currentValue = asr.get(arr);
        final int currentStamp = arr[0];
        System.out.println("currentValue=" + currentValue + ", currentStamp=" + currentStamp);

        // 单独设置 stamp 值
        final boolean attemptStampResult = asr.attemptStamp(newReference, 88);
        System.out.println("currentValue=" + asr.getReference()
                + ", currentStamp=" + asr.getStamp()
                + ", attemptStampResult=" + attemptStampResult);

        // 重新设置当前值和 stamp 值
        asr.set(initialRef, initialStamp);
        System.out.println("currentValue=" + asr.getReference() + ", currentStamp=" + asr.getStamp());

        // [不推荐使用，除非搞清楚注释的意思了] weak compare and set
        // 困惑！weakCompareAndSet 这个方法最终还是调用 compareAndSet 方法。[版本: jdk-8u191]
        // 但是注释上写着 "May fail spuriously and does not provide ordering guarantees,
        // so is only rarely an appropriate alternative to compareAndSet."
        // todo 感觉有可能是 jvm 通过方法名在 native 方法里面做了转发
        final boolean wCasResult = asr.weakCompareAndSet(initialRef, newReference, initialStamp, newStamp);
        System.out.println("currentValue=" + asr.getReference()
                + ", currentStamp=" + asr.getStamp()
                + ", wCasResult=" + wCasResult);
    }
}
```

输出结果：

```java
currentValue=0, currentStamp=0
currentValue=666, currentStamp=999, casResult=true
currentValue=666, currentStamp=999
currentValue=666, currentStamp=88, attemptStampResult=true
currentValue=0, currentStamp=0
currentValue=666, currentStamp=999, wCasResult=true
```

（3）` AtomicMarkableReference`类使用示例

``` java
public class AtomicMarkableReferenceDemo {
    public static void main(String[] args) {
        // 实例化、取当前值和 mark 值
        final Boolean initialRef = null, initialMark = false;
        final AtomicMarkableReference<Boolean> amr = new AtomicMarkableReference<>(initialRef, initialMark);
        System.out.println("currentValue=" + amr.getReference() + ", currentMark=" + amr.isMarked());

        // compare and set
        final Boolean newReference1 = true, newMark1 = true;
        final boolean casResult = amr.compareAndSet(initialRef, newReference1, initialMark, newMark1);
        System.out.println("currentValue=" + amr.getReference()
                + ", currentMark=" + amr.isMarked()
                + ", casResult=" + casResult);

        // 获取当前的值和当前的 mark 值
        boolean[] arr = new boolean[1];
        final Boolean currentValue = amr.get(arr);
        final boolean currentMark = arr[0];
        System.out.println("currentValue=" + currentValue + ", currentMark=" + currentMark);

        // 单独设置 mark 值
        final boolean attemptMarkResult = amr.attemptMark(newReference1, false);
        System.out.println("currentValue=" + amr.getReference()
                + ", currentMark=" + amr.isMarked()
                + ", attemptMarkResult=" + attemptMarkResult);

        // 重新设置当前值和 mark 值
        amr.set(initialRef, initialMark);
        System.out.println("currentValue=" + amr.getReference() + ", currentMark=" + amr.isMarked());

        // [不推荐使用，除非搞清楚注释的意思了] weak compare and set
        // 困惑！weakCompareAndSet 这个方法最终还是调用 compareAndSet 方法。[版本: jdk-8u191]
        // 但是注释上写着 "May fail spuriously and does not provide ordering guarantees,
        // so is only rarely an appropriate alternative to compareAndSet."
        // todo 感觉有可能是 jvm 通过方法名在 native 方法里面做了转发
        final boolean wCasResult = amr.weakCompareAndSet(initialRef, newReference1, initialMark, newMark1);
        System.out.println("currentValue=" + amr.getReference()
                + ", currentMark=" + amr.isMarked()
                + ", wCasResult=" + wCasResult);
    }
}
```

输出结果：

```java
currentValue=null, currentMark=false
currentValue=true, currentMark=true, casResult=true
currentValue=true, currentMark=true
currentValue=true, currentMark=false, attemptMarkResult=true
currentValue=null, currentMark=false
currentValue=true, currentMark=true, wCasResult=true
```

### 1.5 原子累加器

**原子类型累加器**是**JDK1.8**引进的并发新技术，它可以看做**AtomicLong**和**AtomicDouble**的部分加强类型。

![](https://img-blog.csdnimg.cn/2020110522343065.png)

原子类型累加器的用法及原理可以参考[Java多线程进阶（十七）—— J.U.C之atomic框架：LongAdder](https://segmentfault.com/a/1190000015865714)一文。

## 2 原子类实现原理

### 1 简析CAS

CAS全称 **Compare And Swap（比较与交换）**，是一种无锁算法。在不使用锁（没有线程被阻塞）的情况下实现多线程之间的**变量同步**。

CAS算法涉及到三个操作数：

- V 内存地址存放的实际值
- A 比较的旧值 
- B 更新的新值

当且仅当V的值等于A时（旧值和内存中实际的值相同），表明旧值A已经是目前最新版本的值，自然而然可以将新值 N 赋值给 V。反之则表明V和A变量不同步，直接返回V即可。当多个线程使用CAS操作一个变量时，只有一个线程会更新成功，其余失败的线程会重新尝试。也就是说，“更新”是一个不断重试的操作。

进入原子类AtomicInteger的源码，看一下AtomicInteger的定义：

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    // 获取并操作内存的数据。
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    // 存储value在AtomicInteger中的偏移量。
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }
	
    // 存储AtomicInteger的int值，该属性需要借助volatile关键字保证其在线程间是可见的。
    private volatile int value;
```

接下来，我们查看AtomicInteger的自增函数incrementAndGet()的源码时，发现自增函数底层调用的是unsafe.getAndAddInt()。

```java
    // AtomicInteger 自增方法
    public final int incrementAndGet() {
      return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
    }    

	// Unsafe.class
	public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

        return var5;
    }
```

`compareAndSwapInt`这个函数，它也是`CAS`缩写的由来。通过OpenJDK 8 来查看Unsafe.cpp的源码：

```java
// Unsafe.java
public final int getAndAddInt(Object o, long offset, int delta) {
   int v;
   do {
       v = getIntVolatile(o, offset);
   } while (!compareAndSwapInt(o, offset, v, v + delta));
   return v;
}
```

根据OpenJDK 8的源码我们可以看出，getAndAddInt()循环获取给定对象o中的偏移量处的值v，然后判断内存值是否等于v。如果相等则将内存值设置为 v + delta，否则返回false，继续循环进行重试，直到设置成功才能退出循环，并且将旧值返回。整个“比较+更新”操作封装在compareAndSwapInt()中，在JNI里是借助于一个CPU指令完成的，属于原子操作，可以保证多个线程都能够看到同一个变量的修改值。

后续JDK通过CPU的**cmpxchg指令**，去比较寄存器中的 A 和 内存中的值 V。如果相等，就把要写入的新值 B 存入内存中。如果不相等，就将内存值 V 赋值给寄存器中的值 A。然后通过Java代码中的while循环再次调用cmpxchg指令进行重试，直到设置成功为止。

CAS虽然很高效，但是它也存在三大问题：

（1）ABA问题

CAS需要在操作值的时候检查内存值是否发生变化，没有发生变化才会更新内存值。但是如果内存值原来是A，后来变成了B，然后又变成了A，那么CAS进行检查时会发现值没有发生变化，但是实际上是有变化的。ABA问题的解决思路就是在变量前面添加版本号，每次变量更新的时候都把版本号加一，这样变化过程就从“A－B－A”变成了“1A－2B－3A”。

JDK从1.5开始提供了AtomicStampedReference类来解决ABA问题，具体操作封装在compareAndSet()中。compareAndSet()首先检查当前引用和当前标志与预期引用和预期标志是否相等，如果都相等，则以原子方式将引用值和标志的值设置为给定的更新值。

（2）**循环时间长开销大**。CAS操作如果长时间不成功，会导致其一直自旋，给CPU带来非常大的开销。

（3）只能保证一个共享变量的原子操作

对一个共享变量执行操作时，CAS能够保证原子操作，但是对多个共享变量操作时，CAS是无法保证操作的原子性的。

Java从1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，可以把多个变量放在一个对象里来进行CAS操作。







在多线程环境不使用原子类保证线程安全（基本数据类型）

```java
class Test1 {
    private volatile int count = 0;

    // 若要线程安全执行count++，需要加锁
    public synchronized void increment() {
        count++;
    }

    public int getCount() {
        return count;
    }
}
```

多线程环境使用**原子类**保证线程安全（基本数据类型）

```java
class Test2 {
    private AtomicInteger count = new AtomicInteger();

    public void increment() {
        count.incrementAndGet();
    }

    // 使用AtomicInteger之后，不需要加锁，也可以实现线程安全。
    public int getCount() {
        return count.get();
    }
}
```







## 参考

- 《Java并发编程的艺术》
- [Java中的无锁编程](https://my.oschina.net/cqqcqqok/blog/1925073)
- [Java高并发：CAS无锁原理及广泛应用](https://developer.aliyun.com/article/694255)

* [Java多线程进阶（十七）—— J.U.C之atomic框架：LongAdder](https://segmentfault.com/a/1190000015865714)



> 修正: **AtomicMarkableReference 不能解决ABA问题**   **[issue#626](https://github.com/Snailclimb/JavaGuide/issues/626)**

```java
    /**

AtomicMarkableReference是将一个boolean值作是否有更改的标记，本质就是它的版本号只有两个，true和false，

修改的时候在这两个版本号之间来回切换，这样做并不能解决ABA的问题，只是会降低ABA问题发生的几率而已

@author : mazh

@Date : 2020/1/17 14:41
*/

public class SolveABAByAtomicMarkableReference {
       
       private static AtomicMarkableReference atomicMarkableReference = new AtomicMarkableReference(100, false);

        public static void main(String[] args) {

            Thread refT1 = new Thread(() -> {
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                atomicMarkableReference.compareAndSet(100, 101, atomicMarkableReference.isMarked(), !atomicMarkableReference.isMarked());
                atomicMarkableReference.compareAndSet(101, 100, atomicMarkableReference.isMarked(), !atomicMarkableReference.isMarked());
            });

            Thread refT2 = new Thread(() -> {
                boolean marked = atomicMarkableReference.isMarked();
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                boolean c3 = atomicMarkableReference.compareAndSet(100, 101, marked, !marked);
                System.out.println(c3); // 返回true,实际应该返回false
            });

            refT1.start();
            refT2.start();
        }
    }
```

**CAS ABA 问题**

- 描述: 第一个线程取到了变量 x 的值 A，然后巴拉巴拉干别的事，总之就是只拿到了变量 x 的值 A。这段时间内第二个线程也取到了变量 x 的值 A，然后把变量 x 的值改为 B，然后巴拉巴拉干别的事，最后又把变量 x 的值变为 A （相当于还原了）。在这之后第一个线程终于进行了变量 x 的操作，但是此时变量 x 的值还是 A，所以 compareAndSet 操作是成功。
- 例子描述(可能不太合适，但好理解): 年初，现金为零，然后通过正常劳动赚了三百万，之后正常消费了（比如买房子）三百万。年末，虽然现金零收入（可能变成其他形式了），但是赚了钱是事实，还是得交税的！
- 代码例子（以``` AtomicInteger ```为例）

```java
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicIntegerDefectDemo {
    public static void main(String[] args) {
        defectOfABA();
    }

    static void defectOfABA() {
        final AtomicInteger atomicInteger = new AtomicInteger(1);

        Thread coreThread = new Thread(
                () -> {
                    final int currentValue = atomicInteger.get();
                    System.out.println(Thread.currentThread().getName() + " ------ currentValue=" + currentValue);

                    // 这段目的：模拟处理其他业务花费的时间
                    try {
                        Thread.sleep(300);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    boolean casResult = atomicInteger.compareAndSet(1, 2);
                    System.out.println(Thread.currentThread().getName()
                            + " ------ currentValue=" + currentValue
                            + ", finalValue=" + atomicInteger.get()
                            + ", compareAndSet Result=" + casResult);
                }
        );
        coreThread.start();

        // 这段目的：为了让 coreThread 线程先跑起来
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        Thread amateurThread = new Thread(
                () -> {
                    int currentValue = atomicInteger.get();
                    boolean casResult = atomicInteger.compareAndSet(1, 2);
                    System.out.println(Thread.currentThread().getName()
                            + " ------ currentValue=" + currentValue
                            + ", finalValue=" + atomicInteger.get()
                            + ", compareAndSet Result=" + casResult);

                    currentValue = atomicInteger.get();
                    casResult = atomicInteger.compareAndSet(2, 1);
                    System.out.println(Thread.currentThread().getName()
                            + " ------ currentValue=" + currentValue
                            + ", finalValue=" + atomicInteger.get()
                            + ", compareAndSet Result=" + casResult);
                }
        );
        amateurThread.start();
    }
}
```

输出内容如下：

```
Thread-0 ------ currentValue=1
Thread-1 ------ currentValue=1, finalValue=2, compareAndSet Result=true
Thread-1 ------ currentValue=2, finalValue=1, compareAndSet Result=true
Thread-0 ------ currentValue=1, finalValue=2, compareAndSet Result=true
```

