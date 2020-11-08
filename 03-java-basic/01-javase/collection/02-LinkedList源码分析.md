## 1 LinkedList简介

`LinkedList`是一个实现了<font color="red">List接口</font>和<font color="red">Deque接口</font>的<font color="red">双端链表</font>。

<img src="https://img-blog.csdnimg.cn/20201108105348736.png"  />

`LinkedList`继承于 **`AbstractSequentialList`**，实现了 **`List`**、 **`Deque`**、 **`Cloneable`**、**`java.io.Serializable`** 接口。

`LinkedList `实现了` List `接口，表明其能进行链表的高效插入和删除操作。
`LinkedList `实现了` Deque` 接口，即能将`LinkedList`当作双端队列使用。
`LinkedList `实现了了`Cloneable`接口，即覆盖了函数clone()，能够克隆。
`LinkedList `实现了`java.io.Serializable`接口，这意味着`LinkedList `支持序列化，能通过序列化传输数据。

> LinkedList不是线程安全的，如果想使LinkedList变成线程安全的，可以调用静态类Collections类中的synchronizedList方法
>
> `List list=Collections.synchronizedList(new LinkedList(...));`

## 2 成员属性

```java
// 总节点数
transient int size = 0;
// 头节点
transient Node<E> first;
// 尾节点
transient Node<E> last;
```

可以发现LinkedList是基于双向链表实现的，使用 Node 存储链表节点信息。

![](https://img-blog.csdnimg.cn/20201108111123539.png)

```java
    private static class Node<E> {
        E item;//节点值
        Node<E> next;// 后继节点
        Node<E> prev;// 前驱节点

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

LinkedList的结构图：

![](https://img-blog.csdnimg.cn/20201108111340847.png)

## 3 构造方法

```java
// 空参的构造方法，只生成了对象
public LinkedList() {
}

public LinkedList(Collection<? extends E> c) {
    // 先构造一个空链表
    this();
    // 添加Collection中的元素
    addAll(c);
}
```

## 4 常用方法

### 4.1 添加元素

**add(E e)** ：将元素添加到链表尾部

```java
    public boolean add(E e) {
            linkLast(e);
            return true;
        }   

	/**
     * 链接使e作为最后一个元素。
     */
    void linkLast(E e) {
        // 取出尾节点
        final Node<E> l = last;
        // 根据传入的元素构建新节点，这个节点前置节点是上一个尾节点
        final Node<E> newNode = new Node<>(l, e, null);
        // 新创建的节点作为当前链表的尾节点
        last = newNode;
        if (l == null)
            first = newNode;// 如果尾节点为空，那么说明链表是空的，然后把新构建的节点作为头节点
        else
            l.next = newNode;// 如果不为空，那么把添加前的尾节点的后置节点设置为我们新的尾节点
        size++;
        modCount++;
    }
```
**add(int index,E e)**：在指定位置添加元素
```java
public void add(int index, E element) {
        checkPositionIndex(index);// 检查索引index是否处于[0-size]之间

        if (index == size)
            linkLast(element);// 添加在链表尾部
        else 
            linkBefore(element, node(index));// 插入到index所在位置的节点的前面
    }
	
	/**
	 * 将新节点插入到指定节点前面
	 */
    void linkBefore(E e, Node<E> succ) {
        // 取出查找到指定位置的节点
        final Node<E> pred = succ.prev;
        // 构建新节点，前置节点找到节点的原前置节点，e 是元素值，后置节点是根据位置找到的 succ
        final Node<E> newNode = new Node<>(pred, e, succ);
        // // 原位置的前置节点设置为要插入的节点。
        succ.prev = newNode;
        // 如果原位置的前置节点为空，即原位置 succ 是头节点，即 add(0 ,E )然后把新建节点赋值为头节点
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode; // 不为空，原位置的前置节点的后置节点设置为新节点。
        size++;
        modCount++;
    }

    private void checkPositionIndex(int index) {
        if (!isPositionIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private boolean isPositionIndex(int index) {
        return index >= 0 && index <= size;// 检查索引index是否处于[0-size]之间
    }
```
**addAll(Collection  c )：将集合插入到链表尾部**

```java
public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c);
    }

public boolean addAll(int index, Collection<? extends E> c) {
        // 1、检查index范围是否在size之内
        checkPositionIndex(index);

        // 2、toArray()方法把集合的数据存到对象数组中
        Object[] a = c.toArray();
        int numNew = a.length;
        if (numNew == 0)
            return false;

        // 3、得到插入位置的前驱节点和后继节点
        Node<E> pred, succ;
        // 如果插入位置为尾部，前驱节点为last，后继节点为null
        if (index == size) {
            succ = null;
            pred = last;
        }
        // 否则，调用node()方法得到后继节点，再得到前驱节点
        else {
            succ = node(index);
            pred = succ.prev;
        }

        // 4、遍历数据将数据插入
        for (Object o : a) {
            @SuppressWarnings("unchecked") E e = (E) o;
            // 创建新节点
            Node<E> newNode = new Node<>(pred, e, null);
            // 如果插入位置在链表头部
            if (pred == null)
                first = newNode;
            else
                pred.next = newNode;
            pred = newNode;
        }

        // 如果插入位置在尾部，重置last节点
        if (succ == null) {
            last = pred;
        }
        // 否则，将插入的链表与先前链表连接起来
        else {
            pred.next = succ;
            succ.prev = pred;
        }

        size += numNew;
        modCount++;
        return true;
    }    
```
**addFirst(E e)：** 将元素添加到链表头部

```java
 public void addFirst(E e) {
        linkFirst(e);
    }

private void linkFirst(E e) {
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);//新建节点，以头节点为后继节点
        first = newNode;
        //如果链表为空，last节点也指向该节点
        if (f == null)
            last = newNode;
        //否则，将头节点的前驱指针指向新节点，也就是指向前一个元素
        else
            f.prev = newNode;
        size++;
        modCount++;
    }
```
**addLast(E e)：** 将元素添加到链表尾部，与 **add(E e)** 方法一样
```java
public void addLast(E e) {
        linkLast(e);
    }
```
### 4.2 查找元素
**get(int index)：** 根据指定索引返回数据
```java
public E get(int index) {
        // 检查index 是否在合法位置
        checkElementIndex(index);
        // 调用Node(index)去找到index对应的node然后返回它的值item
        return node(index).item;
    }
```
**获取头节点（index=0）数据方法:**

```java
public E getFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return f.item;
    }
public E element() {
        return getFirst();
    }
public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }

public E peekFirst() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
     }
```
**区别：**
getFirst(),element(),peek(),peekFirst()
这四个获取头结点方法的区别在于对链表为空时的处理，是抛出异常还是返回null，其中**getFirst()** 和**element()** 方法将会在链表为空时，抛出异常

element()方法的内部就是使用getFirst()实现的。它们会在链表为空时，抛出NoSuchElementException  
**获取尾节点（index=-1）数据方法:**
```java
 public E getLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return l.item;
    }
 public E peekLast() {
        final Node<E> l = last;
        return (l == null) ? null : l.item;
    }
```
**两者区别：**
**getLast()** 方法在链表为空时，会抛出**NoSuchElementException**，而**peekLast()** 则不会，只是会返回 **null**。

**根据对象得到索引的方法**

**int indexOf(Object o)：** 从头遍历找
```java
public int indexOf(Object o) {
        int index = 0;
        if (o == null) {
            //从头遍历
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            //从头遍历
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }
```
**int lastIndexOf(Object o)：** 从尾遍历找
```java
public int lastIndexOf(Object o) {
        int index = size;
        if (o == null) {
            //从尾遍历
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (x.item == null)
                    return index;
            }
        } else {
            //从尾遍历
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (o.equals(x.item))
                    return index;
            }
        }
        return -1;
    }
```
### 4.3 删除元素
**remove()** ,**removeFirst(),pop():** 删除头节点
```
public E pop() {
        return removeFirst();
    }
public E remove() {
        return removeFirst();
    }
public E removeFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f);
    }
```
**removeLast(),pollLast():** 删除尾节点
```java
public E removeLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return unlinkLast(l);
    }
public E pollLast() {
        final Node<E> l = last;
        return (l == null) ? null : unlinkLast(l);
    }
```
**区别：** removeLast()在链表为空时将抛出NoSuchElementException，而pollLast()方法返回null。

**remove(Object o):** 删除指定元素
```java
public boolean remove(Object o) {
        //如果删除对象为null
        if (o == null) {
            //从头开始遍历
            for (Node<E> x = first; x != null; x = x.next) {
                //找到元素
                if (x.item == null) {
                   //从链表中移除找到的元素
                    unlink(x);
                    return true;
                }
            }
        } else {
            //从头开始遍历
            for (Node<E> x = first; x != null; x = x.next) {
                //找到元素
                if (o.equals(x.item)) {
                    //从链表中移除找到的元素
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
```
当删除指定对象时，只需调用remove(Object o)即可，不过该方法一次只会删除一个匹配的对象，如果删除了匹配对象，返回true，否则false。

unlink(Node<E> x) 方法：
```java
E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;//得到后继节点
        final Node<E> prev = x.prev;//得到前驱节点

        //删除前驱指针
        if (prev == null) {
            first = next;//如果删除的节点是头节点,令头节点指向该节点的后继节点
        } else {
            prev.next = next;//将前驱节点的后继节点指向后继节点
            x.prev = null;
        }

        //删除后继指针
        if (next == null) {
            last = prev;//如果删除的节点是尾节点,令尾节点指向该节点的前驱节点
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }
```
**remove(int index)**：删除指定位置的元素
```java
public E remove(int index) {
        //检查index范围
        checkElementIndex(index);
        //将节点删除
        return unlink(node(index));
    }
```
## 5 Arraylist与LinkedList异同

1. **是否保证线程安全：** `ArrayList` 和 `LinkedList` 都是不同步的，都不保证线程安全；
2. **底层数据结构：** `Arraylist` 底层使用的是 **`Object` 数组**；`LinkedList` 底层使用的是 **双向链表** 数据结构（JDK1.6 之前为循环链表，JDK1.7 取消了循环。）
3. **插入和删除是否受元素位置的影响：** ① **`ArrayList` 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。** 比如：执行`add(E e)`方法的时候， `ArrayList` 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是 O(1)。但是如果要在指定位置 i 插入和删除元素的话（`add(int index, E element)`）时间复杂度就为 O(n-i)。因为在进行上述操作的时候集合中第 i 和第 i 个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作。 ② **`LinkedList` 采用链表存储，所以对于`add(E e)`方法的插入，删除元素时间复杂度不受元素位置的影响，近似 O(1)，如果是要在指定位置`i`插入和删除元素的话（`(add(int index, E element)`） 时间复杂度近似为`o(n))`因为需要先移动到指定位置再插入。**
4. **是否支持快速随机访问：** `LinkedList` 不支持高效的随机元素访问，而 `ArrayList` 支持。快速随机访问就是通过元素的序号快速获取元素对象(对应于`get(int index)`方法)。
5. **内存空间占用：** ArrayList 的空 间浪费主要体现在在 list 列表的结尾会预留一定的容量空间，而 LinkedList 的空间花费则体现在它的每一个元素都需要消耗比 ArrayList 更多的空间（因为要存放直接后继和直接前驱以及数据）。

### 5.1 双向链表与双向循环链表

**双向链表：** 包含两个指针，一个 prev 指向前一个节点，一个 next 指向后一个节点。

![](https://img-blog.csdnimg.cn/20201107160432215.png)

**双向循环链表：** 最后一个节点的 next 指向 head，而 head 的 prev 指向最后一个节点，构成一个环。

![](https://img-blog.csdnimg.cn/20201107160457158.png)

### 5.2 RandomAccess 接口

```java
public interface RandomAccess {
}
```

查看源码我们发现实际上 `RandomAccess` 接口中没有任何方法与定义。可以把`RandomAccess` 接口理解为一个标识。

标识什么？ **标识实现这个接口的类具有随机访问功能**。

在 `binarySearch（)` 方法中，它要判断传入的 list 是否 `RamdomAccess` 的实例，如果是，调用`indexedBinarySearch()`方法，如果不是，那么调用`iteratorBinarySearch()`方法

```java
    public static <T>
    int binarySearch(List<? extends Comparable<? super T>> list, T key) {
        if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
            return Collections.indexedBinarySearch(list, key);
        else
            return Collections.iteratorBinarySearch(list, key);
    }
```

`ArrayList` 实现了 `RandomAccess` 接口， 而 `LinkedList` 没有实现。为什么呢？我觉得还是和底层数据结构有关！`ArrayList` 底层是数组，而 `LinkedList` 底层是链表。数组天然支持随机访问，时间复杂度为 O(1)，所以称为快速随机访问。链表需要遍历到特定位置才能访问特定位置的元素，时间复杂度为 O(n)，所以不支持快速随机访问。，`ArrayList` 实现了 `RandomAccess` 接口，就表明了他具有快速随机访问功能。 `RandomAccess` 接口只是标识，并不是说 `ArrayList` 实现 `RandomAccess` 接口才具有快速随机访问功能的！

## 参考

[LinkedList源码分析（一）](https://juejin.im/post/6844903615811813384)

[LinkedList源码分析（二）](https://juejin.im/post/6844903617904771085)

