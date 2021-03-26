- [1 LinkedList简介](#1-linkedlist简介)
- [2 成员属性](#2-成员属性)
- [3 构造方法](#3-构造方法)
- [4 常用方法](#4-常用方法)
  - [4.1 添加元素](#41-添加元素)
  - [4.2 查找元素](#42-查找元素)
  - [4.3 删除元素](#43-删除元素)
  - [4.4 修改元素](#44-修改元素)
- [5 ArrayList与LinkedList异同](#5-arraylist与linkedlist异同)
- [参考](#参考)
## 1 LinkedList简介

`LinkedList`是一个实现了<font color="red">List接口</font>和<font color="red">Deque接口</font>的<font color="red">双端链表</font>。

<img src="../../../images/collection/20201108105348736.png" style="zoom:80%;" />

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

![](../../../images/collection/20201108111123539.png)

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

因此LinkedList的结构如下：

![](../../../images/collection/20201108111340847.png)

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
        // 找到当前尾节点
        final Node<E> l = last;
        // 构建新节点，这个节点的前驱节点是上一个尾节点
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
            linkBefore(element, node(index));// 插入到index所指位置的节点前面
    }
	
	/**
	 * 将新节点插入到指定节点前面
	 */
    void linkBefore(E e, Node<E> succ) {
        // 取出指定位置的前一个节点
        final Node<E> pred = succ.prev;
        // 构建新节点，pred指定位置的前驱节点，succ指定位置，e是添加的元素
        final Node<E> newNode = new Node<>(pred, e, succ);
        // 指定节点的前驱节点指向要插入的节点。
        succ.prev = newNode;
        // 如果指定位置的前驱节点为空，即指定位置succ是头节点，即newNode设为头节点
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode; // 不为空，原位置的前驱节点的后置节点设置为新节点。
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
**addAll(Collection  c )**：将集合元素插入到链表中

```java
public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c);
    }

public boolean addAll(int index, Collection<? extends E> c) {
        // 1、检查index范围是否在size之内
        checkPositionIndex(index);

        // 2、把集合的数据存到对象数组中
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
                // 把前一个节点向后指向newNode
                first = newNode;
            else
                pred.next = newNode;
            // 在if-else之外，如果没有这语句，pred节点就固定是那一个节点，所以需要在每一次循环内更换节点
            pred = newNode;
        }

        // 如果插入位置在尾部，重置last节点
        if (succ == null) {
            last = pred;
        }
        // 插入的链表与先前链表连接起来
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
    	// 找到当前头节点
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);// 新建节点，以之前头节点为后继节点
        // 新节点为头节点
    	first = newNode;
        // 如果链表为空，last节点也指向新节点
        if (f == null)
            last = newNode;
        // 否则，将之前头节点的前驱指针指向新节点，也就是指向前一个元素
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
        // 检查index是否在合法位置
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
**getFirst()** 和**element()** 方法将会在链表为空时，抛出异常；

**element()**方法的内部就是使用**getFirst()**实现的。它们会在链表为空时，抛出NoSuchElementException 。

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
**getLast()** 方法在链表为空时，会抛出**NoSuchElementException**；而**peekLast()** 只是会返回 **null**。

### 4.3 删除元素
**remove()** 、**removeFirst()、pop():** 删除头节点

```java
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
**removeLast()、pollLast():** 删除尾节点

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
**区别：** removeLast()在链表为空时将抛出NoSuchElementException；pollLast()方法返回null。

**remove(Object o):** 删除指定元素

```java
public boolean remove(Object o) {
        // 如果要删除对象为null
        if (o == null) {
            // 从头开始遍历
            for (Node<E> x = first; x != null; x = x.next) {
                // 找到元素
                if (x.item == null) {
                   // 从链表中移除找到的元素
                    unlink(x);
                    return true;
                }
            }
        } else {
            // 从头开始遍历
            for (Node<E> x = first; x != null; x = x.next) {
                // 找到元素
                if (o.equals(x.item)) {
                    // 从链表中移除找到的元素
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
```
当删除指定对象时，只需调用remove(Object o)即可，不过该方法一次只会删除一个匹配的对象，如果删除了匹配对象，返回true，否则false。

**unlink(Node<E> x)** 方法：删除指定节点

```java
E unlink(Node<E> x) {
        final E element = x.item;
    	// 得到x的后继节点
        final Node<E> next = x.next;
    	// 得到x的前驱节点
        final Node<E> prev = x.prev;

    	// x是头节点
        if (prev == null) {
            // 如果删除的节点是头节点,令头节点指向该节点的后继节点
            first = next;
        } else {
            // 将前驱节点的后继节点指向x的后继节点
            prev.next = next;
            // 删除x的前指向
            x.prev = null;
        }

        // x是尾节点
        if (next == null) {
            last = prev;// 如果删除的节点是尾节点,令尾节点指向该节点的前驱节点
        } else {
            next.prev = prev;
            x.next = null;
        }
		
    	// 删除指定节点内容
        x.item = null;
        size--;
        modCount++;
        return element;
    }
```
**remove(int index)**：删除指定位置的元素
```java
public E remove(int index) {
        // 检查index范围
        checkElementIndex(index);
        // 将节点删除
        return unlink(node(index));
    }
```
### 4.4 修改元素

set(int index, E element)方法：修改指定节点元素

```java
    public E set(int index, E element) {
        // 检查是否越界
        checkElementIndex(index);
        // 根据索引查找节点
        Node<E> x = node(index);
        // oldVal存放需要改变的内容
        E oldVal = x.item;
        x.item = element;
        // 返回改变前的值
        return oldVal;
    }
```

## 5 ArrayList与LinkedList异同

| 异同                           | ArrayList                                       | LinkedList                                           |
| ------------------------------ | ----------------------------------------------- | ---------------------------------------------------- |
| 是否保证线程安全               | 否                                              | 否                                                   |
| 插入和删除是否受元素位置的影响 | 是                                              | 否                                                   |
| 是否支持快速随机访问           | 是                                              | 否                                                   |
| 是否实现了RandomAccess接口     | 是                                              | 否                                                   |
| 底层数据结构                   | Object数组                                      | 双向链表（JDK1.6 之前为循环链表，JDK1.7 取消了循环） |
| 内存空间占用                   | 扩容机制导致list 列表的结尾会预留一定的容量空间 | 每一个元素都需额外消耗存放后继/前驱的空间            |

## 参考

[LinkedList源码分析（一）](https://juejin.im/post/6844903615811813384)

[LinkedList源码分析（二）](https://juejin.im/post/6844903617904771085)

