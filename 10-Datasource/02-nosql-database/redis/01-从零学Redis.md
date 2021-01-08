# 前言

> 只有光头才能变强

最近在学Redis，我相信只要是接触过Java开发的都会听过Redis这么一个技术。面试也是非常高频的一个知识点，之前一直都是处于了解阶段。秋招过后这段时间是没有什么压力的，所以打算系统学学Redis，这也算是我从零学习Redis的笔记吧。

本文**力求讲清每个知识点**，希望大家看完能有所收获。

# 一、介绍一下Redis

首先，肯定是去官网看看官方是怎么介绍Redis的啦。https://redis.io/topics/introduction

> 如果像我一样，英语可能不太好的，可能看不太懂。没事，咱们Chrome浏览器可以切换成中文的，中文是我们的母语，肯定没啥压力了。Eumm…

读完之后，发现中文也就那样了。

> 一大堆没见过的技术：lua(Lua脚本)、replication(复制)、Redis Sentinel(哨兵)、Redis Cluster(Redis 集群)，当然我们也会有看得懂的技术：transactions(事务)、different levels of on-disk persistence(数据持久化)、LRU eviction(LRU淘汰机制)..

至少官方介绍Redis的第一句应该是可以很容易看懂："Redis is an open source (BSD licensed),**in-memory data structure store**, used as a database,**cache** and message broker."

Redis是一个开源的，**基于内存的数据结构存储**，可用作于数据库、**缓存**、消息中间件。

- 从官方的解释上，我们可以知道：Redis是基于内存，支持多种数据结构。
- 从经验的角度上，我们可以知道：Redis常用作于缓存。

就我个人认为：学习一种新技术，先把握该技术整体的知识(思想)，再扣细节，这样学习起来会比较轻松一些。所以我们先以“内存”、“数据结构”、“缓存”来对Redis入门。

## 1.1为什么要用Redis？

从上面可知：Redis是**基于内存**，常用作于**缓存**的一种技术，并且Redis存储的方式是以`key-value`的形式。

我们可以发现这不就是Java的Map容器所拥有的特性吗，那为什么还需要Redis呢？

- Java实现的Map是**本地缓存**，如果有多台实例(机器)的话，每个实例都需要**各自**保存一份缓存，缓存**不具有一致性**
- Redis实现的是**分布式缓存**，如果有多台实例(机器)的话，每个实例都**共享**一份缓存，缓存**具有一致性**。
- Java实现的Map**不是专业**做缓存的，JVM内存太大容易挂掉的。一般用做于容器来存储临时数据，缓存的数据随着JVM销毁而结束。Map所存储的数据结构，缓存过期机制等等是需要程序员自己手写的。
- Redis是**专业**做缓存的，可以用几十个G内存来做缓存。Redis一般用作于缓存，可以将缓存数据保存在硬盘中，Redis重启了后可以将其恢复。原生提供丰富的数据结构、缓存过期机制等等简单好用的功能。

参考资料：

- 为什么要用redis而不用map做缓存?https://segmentfault.com/q/1010000009106416

## 1.2为什么要用缓存？

如果我们的网站出现了性能问题(访问时间慢)，按经验来说，一般是由于**数据库撑不住了**。因为一般数据库的读写都是要经过**磁盘**的，而磁盘的速度可以说是相当慢的(相对内存来说)

- 科普文：让 CPU 告诉你硬盘和网络到底有多慢https://zhuanlan.zhihu.com/p/24726196

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3601DAl7JNGKSFZFgcxM67J5zFKh0gT0M8M2voDibxpWWATlGQNR3ALtmputlHxOelMnticO7yqmyw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)数据库撑不住了

如果学过Mybaits、Hibernate的同学就可以知道，它们有一级缓存、二级缓存这样的功能(终究来说还是本地缓存)。目的就是为了：**不用每次读取的时候，都要查一次数据库**。

有了缓存之后，我们的访问就变成这样了：

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3601DAl7JNGKSFZFgcxM675pYxIMxdW1O19iaJSCp0YaJyXp1MDsjA0JNnfbdKPV124QynA8UT8QQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)有了缓存提高了并发和性能

## 二、Redis的数据结构

> 本文不会讲述命令的使用方式，具体的如何使用可查询API。

- Redis 命令参考：http://doc.redisfans.com/
- try Redis(不用安装Redis即可体验Redis命令)：http://try.redis.io/

Redis支持丰富的数据结构，**常用**的有string、list、hash、set、sortset这几种。学习这些数据结构是使用Redis的基础！

"Redis is written in ANSI C"-->Redis由C语言编写

首先还是得声明一下，Redis的存储是以`key-value`的形式的。Redis中的key一定是字符串，value可以是string、list、hash、set、sortset这几种常用的。

![redis数据结构](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)redis数据结构

但要值得注意的是：Redis并**没有直接使用**这些数据结构来实现`key-value`数据库，而是**基于**这些数据结构创建了一个**对象系统**。

- 简单来说：Redis使用对象来表示数据库中的键和值。每次我们在Redis数据库中新创建一个键值对时，**至少会创建出两个对象**。一个是键对象，一个是值对象。

Redis中的每个对象都由一个redisObject结构来表示：

```
typedef struct redisObject{

    // 对象的类型
    unsigned type 4:;

    // 对象的编码格式
    unsigned encoding:4;

    // 指向底层实现数据结构的指针
    void * ptr;

    //.....


}robj;
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3601DAl7JNGKSFZFgcxM67Jj8PAvoh38hsibJxVx3NibI03PI6WicTzUicPzzRbkoBGicUoc24K8d4Yng/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)数据结构对应的类型与编码

简单来说就是Redis对`key-value`封装成对象，key是一个对象，value也是一个对象。每个对象都有type(类型)、encoding(编码)、ptr(指向底层数据结构的指针)来表示。

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3601DAl7JNGKSFZFgcxM67nZxDIeVpOQ9sniae3iaenMvXMGaDTLDCUibqFicib3lxv9ZZevXAX3fQ17g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)以值为1006的字符串对象为例

下面我就来说一下我们Redis常见的数据类型：string、list、hash、set、sortset。它们的底层数据结构究竟是怎么样的！

## 2.1SDS简单动态字符串

> 简单动态字符串(Simple dynamic string,SDS)

Redis中的字符串跟C语言中的字符串，是**有点差距的**。

Redis使用sdshdr结构来表示一个SDS值：

```
struct sdshdr{

    // 字节数组，用于保存字符串
    char buf[];

    // 记录buf数组中已使用的字节数量，也是字符串的长度
    int len;

    // 记录buf数组未使用的字节数量
    int free;
}
```

例子：

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3601DAl7JNGKSFZFgcxM67x7w7oZCLtZt6DkC4TUNMvO00JiauQvw4a15Ey1CYNiauPSYB5UqrZP1Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)SDS例子

### 2.1.1使用SDS的好处

> SDS与C的字符串表示比较

1. sdshdr数据结构中用len属性记录了字符串的长度。那么**获取字符串的长度时，时间复杂度只需要O(1)**。
2. SDS不会发生溢出的问题，如果修改SDS时，空间不足。先会扩展空间，再进行修改！(**内部实现了动态扩展机制**)。
3. SDS可以**减少内存分配的次数**(空间预分配机制)。在扩展空间时，除了分配修改时所必要的空间，还会分配额外的空闲空间(free 属性)。
4. SDS是**二进制安全的**，所有SDS API都会以处理二进制的方式来处理SDS存放在buf数组里的数据。

## 2.2链表

对于链表而言，我们不会陌生的了。在大学期间肯定开过数据结构与算法课程，链表肯定是讲过的了。在Java中Linkedxxx容器底层数据结构也是链表+[xxx]的。我们来看看Redis中的链表是怎么实现的：

使用listNode结构来表示每个节点：

```
typedef strcut listNode{

    //前置节点
    strcut listNode  *pre;

    //后置节点
    strcut listNode  *pre;

    //节点的值
    void  *value;

}listNode
```

使用listNode是可以组成链表了，Redis中**使用list结构来持有链表**：

```
typedef struct list{

    //表头结点
    listNode  *head;

    //表尾节点
    listNode  *tail;

    //链表长度
    unsigned long len;

    //节点值复制函数
    void *(*dup) (viod *ptr);

    //节点值释放函数
    void  (*free) (viod *ptr);

    //节点值对比函数
    int (*match) (void *ptr,void *key);

}list
```

具体的结构如图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3601DAl7JNGKSFZFgcxM673eKPWeNClyBfxZILlYYf4icDjX5r1j48wJMJZpic2c1ZDwaDZJcqkM3A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 2.2.1Redis链表的特性

Redis的链表有以下特性：

- 无环双向链表
- 获取表头指针，表尾指针，链表节点长度的时间复杂度均为O(1)
- 链表使用`void *`指针来保存节点值，可以保存各种不同类型的值

## 2.3哈希表

> 声明：《Redis设计与实现》里边有“字典”这么一个概念，我个人认为还是直接叫哈希表比较通俗易懂。从代码上看：“字典”也是在哈希表基础上再抽象了一层而已。

在Redis中，`key-value`的数据结构底层就是哈希表来实现的。对于哈希表来说，我们也并不陌生。在Java中，哈希表实际上就是数组+链表的形式来构建的。下面我们来看看Redis的哈希表是怎么构建的吧。

在Redis里边，哈希表使用dictht结构来定义：

```
    typedef struct dictht{

        //哈希表数组
        dictEntry **table;  

        //哈希表大小
        unsigned long size;    

        //哈希表大小掩码，用于计算索引值
        //总是等于size-1
        unsigned long sizemark;     

        //哈希表已有节点数量
        unsigned long used;

    }dictht
```

哈希表的数据结构

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

哈希表的数据结构

我们下面继续写看看哈希表的节点是怎么实现的吧：

```
    typedef struct dictEntry {

        //键
        void *key;

        //值
        union {
            void *value;
            uint64_tu64;
            int64_ts64;
        }v;    

        //指向下个哈希节点，组成链表
        struct dictEntry *next;

    }dictEntry;
```

从结构上看，我们可以发现：Redis实现的哈希表和Java中实现的是**类似**的。只不过Redis多了几个属性来记录常用的值：sizemark(掩码)、used(已有的节点数量)、size(大小)。

同样地，Redis为了更好的操作，对哈希表往上再封装了一层(参考上面的Redis实现链表)，使用dict结构来表示：

```
typedef struct dict {

    //类型特定函数
    dictType *type;

    //私有数据
    void *privdata;

    //哈希表
    dictht ht[2];

    //rehash索引
    //当rehash不进行时，值为-1
    int rehashidx;  

}dict;


//-----------------------------------

typedef struct dictType{

    //计算哈希值的函数
    unsigned int (*hashFunction)(const void * key);

    //复制键的函数
    void *(*keyDup)(void *private, const void *key);

    //复制值得函数
    void *(*valDup)(void *private, const void *obj);  

    //对比键的函数
    int (*keyCompare)(void *privdata , const void *key1, const void *key2)

    //销毁键的函数
    void (*keyDestructor)(void *private, void *key);

    //销毁值的函数
    void (*valDestructor)(void *private, void *obj);  

}dictType
```

所以，最后我们可以发现，Redis所实现的哈希表最后的数据结构是这样子的：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

从代码实现和示例图上我们可以发现，**Redis中有两个哈希表**：

- ht[0]：用于存放**真实**的`key-vlaue`数据
- ht[1]：用于**扩容(rehash)**

Redis中哈希算法和哈希冲突跟Java实现的差不多，它俩**差异**就是：

- Redis哈希冲突时：是将新节点添加在链表的**表头**。
- JDK1.8后，Java在哈希冲突时：是将新的节点添加到链表的**表尾**。

## 2.3.1rehash的过程

下面来具体讲讲Redis是怎么rehash的，因为我们从上面可以明显地看到，**Redis是专门使用一个哈希表来做rehash的**。这跟Java一次性直接rehash是有区别的。

> 在对哈希表进行扩展或者收缩操作时，reash过程并不是一次性地完成的，而是**渐进式**地完成的。

Redis在rehash时采取渐进式的原因：**数据量如果过大的话，一次性rehash会有庞大的计算量，这很可能导致服务器一段时间内停止服务**。

Redis具体是rehash时这么干的：

- (1:在字典中维持一个索引计数器变量rehashidx，并将设置为0，表示rehash开始。
- (2:在rehash期间每次对字典进行增加、查询、删除和更新操作时，**除了执行指定命令外**；还会将ht[0]中rehashidx索引上的值**rehash到ht[1]**，操作完成后rehashidx+1。
- (3:字典操作不断执行，最终在某个时间点，所有的键值对完成rehash，这时**将rehashidx设置为-1，表示rehash完成**
- (4:在渐进式rehash过程中，字典会同时使用两个哈希表ht[0]和ht[1]，所有的更新、删除、查找操作也会在两个哈希表进行。例如要查找一个键的话，**服务器会优先查找ht[0]，如果不存在，再查找ht[1]**，诸如此类。此外当执行**新增操作**时，新的键值对**一律保存到ht[1]**，不再对ht[0]进行任何操作，以保证ht[0]的键值对数量只减不增，直至变为空表。

## 2.4跳跃表(shiplist)

跳跃表(shiplist)是实现sortset(**有序**集合)的底层数据结构之一！

跳跃表可能对于大部分人来说不太常见，之前我在学习的时候发现了一篇不错的文章讲跳跃表的，建议大家先去看完下文再继续回来阅读：

- 漫画算法：什么是跳跃表？http://blog.jobbole.com/111731/

Redis的跳跃表实现由zskiplist和zskiplistNode两个结构组成。其中**zskiplist保存跳跃表的信息**(表头，表尾节点，长度)，**zskiplistNode则表示跳跃表的节点**。

按照惯例，我们来看看zskiplistNode跳跃表节点的结构是怎么样的：

```
typeof struct zskiplistNode {
        // 后退指针
        struct zskiplistNode *backward;
        // 分值
        double score;
        // 成员对象
        robj *obj;
        // 层
        struct zskiplistLevel {
                // 前进指针
                struct zskiplistNode *forward;
                // 跨度
                unsigned int span;
        } level[];
} zskiplistNode;
```

zskiplistNode的对象示例图(带有不同层高的节点)：

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3601DAl7JNGKSFZFgcxM67NsfFuxCSXrKZOROAVRZWcuMe6DeD5iamzbcNwQr4pMEWZibUPceiaOWJA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)带有不同层高的节点

示例图如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3601DAl7JNGKSFZFgcxM67F45KWmwiaB0ibWexpAqn0kA8nRVrfRkic5ib4UwkMQgX3C06GQelA3Ut1g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)跳跃表节点的示例图

zskiplist的结构如下：

```
typeof struct zskiplist {
        // 表头节点，表尾节点
        struct skiplistNode *header,*tail;
        // 表中节点数量
        unsigned long length;
        // 表中最大层数
        int level;
} zskiplist;
```

最后我们整个跳跃表的示例图如下：

![跳跃表示例图](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3601DAl7JNGKSFZFgcxM67TbVKcmAP3gnianDtiauhAOniaDMcicf5xvVnR3At991oqYosuEO5kiaZUNQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 2.5整数集合(intset)

整数集合是set(集合)的底层数据结构之一。当一个set(集合)**只包含整数值元素**，并且**元素的数量不多**时，Redis就会采用整数集合(intset)作为set(集合)的底层实现。

整数集合(intset)保证了元素是**不会出现重复**的，并且是**有序**的(从小到大排序)，intset的结构是这样子的：

```
typeof struct intset {
        // 编码方式
        unit32_t encoding;
        // 集合包含的元素数量
        unit32_t lenght;
        // 保存元素的数组
        int8_t contents[];
} intset;
```

intset示例图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3601DAl7JNGKSFZFgcxM67ENSVmJO5ZKf9Gm7okPobiab96DQBqzKMbfPJt4as1egtZBEvUbnjia1w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

说明：虽然intset结构将contents属性声明为int8_t类型的数组，但实际上contents数组并不保存任何int8_t类型的值，**contents数组的真正类型取决于encoding属性的值**：

- INTSET_ENC_INT16
- INTSET_ENC_INT32
- INTSET_ENC_INT64

从编码格式的名字我们就可以知道，16,32,64编码对应能存放的数字范围是不一样的。16明显最少，64明显最大。

如果本来是INTSET_ENC_INT16的编码，想要存放大于INTSET_ENC_INT16编码能存放的整数值，此时就得编码**升级**(从16升级成32或者64)。步骤如下：

- 1）根据新元素类型拓展整数集合底层数组的空间并为新元素分配空间。
- 2）将底层数组现有的所以元素都转换成与新元素相同的类型，并将类型转换后的元素放到正确的位上，需要维持底层数组的有序性质不变。
- 3）将新元素添加到底层数组。

另外一提：**只支持升级操作，并不支持降级操作**。

## 2.6压缩列表(ziplist)

压缩列表(ziplist)是list和hash的底层实现之一。如果list的每个都是小整数值，或者是比较短的字符串，压缩列表(ziplist)作为list的底层实现。

压缩列表(ziplist)是Redis为了节约内存而开发的，是由一系列的**特殊编码的连续内存块**组成的**顺序性**数据结构。

压缩列表结构图例如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3601DAl7JNGKSFZFgcxM67dPV9J3ANXSZ3OZiaodMUlQYbL6L06KzFP3ia8niaxia2Ljul9PX7ya3mpg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

下面我们看看节点的结构图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3601DAl7JNGKSFZFgcxM67Fd7lkj3IWjkicM0SibERgVm2KMpu1gNEpYaAvUtd6ibOm0SicHJdBnPib8Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> 压缩列表从表尾节点**倒序遍历**，首先指针通过zltail偏移量指向表尾节点，然后通过指向**节点记录的前一个节点的长度依次向前遍历访问整个压缩列表**。

# 三、Redis中数据结构的对象

再次看回这张图，觉不觉得就很好理解了？

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3601DAl7JNGKSFZFgcxM67Jj8PAvoh38hsibJxVx3NibI03PI6WicTzUicPzzRbkoBGicUoc24K8d4Yng/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)数据结构对应的类型与编码

## 3.1字符串(stirng)对象

在上面的图我们知道string类型有三种**编码格式**：

- int：整数值，这个整数值可以使用long类型来表示

- - 如果是浮点数，那就用embstr或者raw编码。具体用哪个就看这个数的长度了

- embstr：字符串值，这个字符串值的长度小于32字节

- raw：字符串值，这个字符串值的长度大于32字节

embstr和raw的**区别**：

- raw分配内存和释放内存的次数是两次，embstr是一次
- embstr编码的数据保存在一块**连续**的内存里面

编码之间的**转换**：

- int类型如果存的**不再是一个整数值**，则会从int转成raw
- embstr是只读的，在修改的时候回从embstr转成raw

## 3.2列表(list)对象

在上面的图我们知道list类型有两种**编码格式**：

- ziplist：字符串元素的长度都小于64个字节`&&`总数量少于512个
- linkedlist：字符串元素的长度大于64个字节`||`总数量大于512个

ziplist编码的列表结构：

```
    redis > RPUSH numbers 1 "three" 5
    (integer) 3 
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3601DAl7JNGKSFZFgcxM67SG3rgPaicjApEUQZKe40uAUia5MJaBumdUncDq6rjPFZoHAoPBD3CFqA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

linkedlist编码的列表结构：

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3601DAl7JNGKSFZFgcxM67ocib84ibcuIbrrnibUoa3gdFLXmRPMnBdJN5uy7OpE7DTutpOBMDBicia5w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

编码之间的**转换：**

- 原本是ziplist编码的，如果保存的数据长度太大或者元素数量过多，会转换成linkedlist编码的。

## 3.3哈希(hash)对象

在上面的图我们知道hash类型有两种**编码格式**：

- ziplist：key和value的字符串长度都小于64字节`&&`键值对总数量小于512
- hashtable：key和value的字符串长度大于64字节`||`键值对总数量大于512

ziplist编码的哈希结构：

![ziplist编码的哈希结构1](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3601DAl7JNGKSFZFgcxM67XRtUC6zfo0D4SdFj47zAufeYGvS3y5AibLKqlXQjUBhNmDuu01F8Rhg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3601DAl7JNGKSFZFgcxM67PDiakS3wWzXicial21yI3yFKDIoth6EasgsWk16YjQpibmzqdMaJ7VNtYg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

hashtable编码的哈希结构：

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3601DAl7JNGKSFZFgcxM67uMtd6D6Rke4LiaftvzuoA3QFlpcIylnurdshibbTS9dXC3ialtqZw1Csg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)hashtable编码的哈希结构

编码之间的**转换：**

- 原本是ziplist编码的，如果保存的数据长度太大或者元素数量过多，会转换成hashtable编码的。

## 3.4集合(set)对象

在上面的图我们知道set类型有两种**编码格式**：

- intset：保存的元素全都是整数`&&`总数量小于512
- hashtable：保存的元素不是整数`||`总数量大于512

intset编码的集合结构：

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3601DAl7JNGKSFZFgcxM67FMQXLgOoFFokjHRllXf4fI0XOLQ5ibqsLY3IlyiaBOFBjasNiagr3oeIA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)intset编码的集合结构

hashtable编码的集合结构：

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3601DAl7JNGKSFZFgcxM677XymKVSpc0e2gCPSVESN7CCGTZdBpIrFbVOA8jWvATjkyZ2wLDibwpw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

编码之间的**转换：**

- 原本是intset编码的，如果保存的数据不是整数值或者元素数量大于512，会转换成hashtable编码的。

## 3.5有序集合(sortset)对象

在上面的图我们知道set类型有两种**编码格式**：

- ziplist：元素长度小于64`&&`总数量小于128
- skiplist：元素长度大于64`||`总数量大于128

ziplist编码的有序集合结构：

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3601DAl7JNGKSFZFgcxM67v42qD77qib4mgac75ibs6KwUMBIyIDcO6CcfEicmzJOiagoo8toSq5937A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3601DAl7JNGKSFZFgcxM67E9LCvJNGfe5nicB2M57S5icfLibMkEGaiakPoDeznwzA1n0kJ3YbmN6I6w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

skiplist编码的有序集合结构：

![图片](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3601DAl7JNGKSFZFgcxM67GbNDScP1ViccbOJ5AISw07R1puEibYmbtI7pBXSjibfTbBWgObsWUYmBQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)skiplist编码的有序集合结构

有序集合(sortset)对象**同时采用skiplist和哈希表来实现**：

- skiplist能够达到插入的时间复杂度为O(logn)，根据成员查分值的时间复杂度为O(1)

编码之间的**转换：**

- 原本是ziplist编码的，如果保存的数据长度大于64或者元素数量大于128，会转换成skiplist编码的。

## 3.6Redis对象一些细节

- (1：服务器在执行某些命令的时候，会**先检查给定的键的类型**能否执行指定的命令。

- - 比如我们的数据结构是sortset，但你使用了list的命令。这是不对的，服务器会检查一下我们的数据结构是什么才会进一步执行命令

- (2：Redis的对象系统带有**引用计数**实现的**内存回收机制**。

- - 对象不再被使用的时候，对象所占用的内存会释放掉

- (3：Redis会共享值为0到9999的字符串对象

- (4：对象**会记录自己的最后一次被访问时间**，这个时间可以用于计算对象的空转时间。

# 最后

本文主要讲了一下Redis常用的数据结构，以及这些数据结构的底层设计是怎么样的。整体来说不会太难，因为这些数据结构我们在学习的过程中多多少少都接触过了，《Redis设计与实现》这本书写得也足够通俗易懂。

至于我们在使用的时候挑选哪些数据结构作为存储，可以简单看看：

- string-->简单的`key-value`
- list-->有序列表(底层是双向链表)-->可做简单队列
- set-->无序列表(去重)-->提供一系列的交集、并集、差集的命令
- hash-->哈希表-->存储结构化数据
- sortset-->有序集合映射(member-score)-->排行榜

如果大家有更好的理解方式或者文章有错误的地方还请大家不吝在评论区留言，大家互相学习交流~~~

参考博客：

- Redis简明教程http://bridgeforyou.cn/2018/05/19/Redis-Tutorial/
- 五旬大爷教你一窥redis之谜https://zhuanlan.zhihu.com/p/34762100

参考资料：

- 《Redis设计与实现》
- 《Redis实战》

> 一个**坚持原创的Java技术公众号：Java3y**，欢迎大家关注