## 写在最前 

[John Washam](https://startupnextdoor.com/author/john/) 记录了从Web开发者（自学、非计算机科学学位）蜕变至 Google软件工程师所制定的学习历程[coding-interview-university](https://github.com/jwasham/coding-interview-university)。

![白板上编程 ———— 来自 HBO 频道的剧集，“硅谷”](https://d3j2pkmjtin6ou.cloudfront.net/coding-at-the-whiteboard-silicon-valley.png)

受其启发，我根据自身情况及诉求，也制定了学习计划。希望通过接下来一段时间的学习，打通任督二脉，对计算机科学有体系的理解，能够获取BAT独角兽公司的Offer。

## 项目结构

学习笔记都以Markdown文档格式整理到不同的文件夹下，目前结构如下：

```java
学而时习之
├── 基础学习
|    ├── 计算机网络
|    ├── 操作系统
|    ├── 信息安全
|    ├── 计算机组成原理
|    ├── 数据结构与算法
|    |	  ├── 数据结构
|    |	  └── 算法
|    └── 数据库
|     	  ├── 数据库原理
|     	  ├── Mysql
|     	  └── Oracle
├── 开发工具
├── Java基础
|	  ├── JavaSE
|	  ├── JVM
|     ├── 并发编程
|     ├── 设计模式
|	  └── Java新特性
├── JavaWeb
|	 ├── 前端
|    |    ├── 前端基础
|    |    |	  ├── HTML
|    |    |	  ├── CSS
|    |    |	  └── JAVAScript
|    |    ├── 模板引擎
|    |    |	  ├── Thymeleaf
|    |    |	  └── FreeMarker
|    |    └── 组合框架
|    |    	  ├── Vue
|    |    	  └── React
|	 └── JDBC
├── 进阶学习
|    ├── Spring全家桶
|    |    ├── Spring
|    | 	  ├── SpringMVC
|    | 	  └── SprinBoot
|    ├── 服务器
|    |	  ├── Web服务器
|    |	  └── 应用服务器
|    ├── 中间件
|    |	  ├── 缓存
|    |	  ├── 消息队列
|    |	  └── RPC框架
|    ├── 数据库
|    |	  ├── ORM框架
|    |    |	  ├── MyBatis
|    |    |	  ├── HIbernate
|    |    |	  └── JPA
|    |	  ├── 连接池
|    |	  └── 分库列表
|    ├── 搜索引擎
|    |	  ├── ElasticSearch
|    |	  ├── Solr
|    |	  └── Lucene
├── 分布式/微服务
|	 ├── 服务注册/发现
|	 ├── 网关
|	 ├── 服务调用（负载均衡）
|	 ├── 熔断/降级
|	 ├── 配置中心
|	 ├── 认证和鉴权
|	 ├── 分布式事务
|	 ├── 任务调度
|	 ├── 链路追踪与监控    
|	 └── 日志分析与监控
|    └── 虚拟化/容器化
|    	  ├── 容器技术
|    	  └── 容器编排技术
├── 大数据
|    ├── Hadoop
|    └── Spark
├── 运维
|    ├── CND加速
|    ├── 代码质量检验
|    └── 日志分析/收集
├── 面试
├── Python学习
├── 错误记录
└── 其他  
```

## 正式开始之前

**1. 你不可能把所有的东西都记住**

与John Washam文章[记住计算机科学知识](https://startupnextdoor.com/retaining-computer-science-knowledge/)描述的一样，我经常是看了数小时的视频，并记录大量的笔记。一段时间过后，基本上忘却大部分的内容。

推荐学习课程[Learning How to Learn: Powerful mental tools to help you master tough subjects](https://www.coursera.org/learn/learning-how-to-learn)。

**2. 使用抽认卡**

为了解决善忘的问题，可以制作不同的抽认卡帮助学习。

John Washam的抽认卡网站：

- [抽认卡页面的代码仓库](https://github.com/jwasham/computer-science-flash-cards)
- [抽认卡数据库 ── 旧 1200 张](https://github.com/jwasham/computer-science-flash-cards/blob/master/cards-jwasham.db)
- [抽认卡数据库 ── 新 1800 张](https://github.com/jwasham/computer-science-flash-cards/blob/master/cards-jwasham-extreme.db)

另外推荐支持多平台的软件[Anki](http://ankisrs.net/)与国内的[记乎](http://www.geefoo.cn/)。

**3. 复习、复习、再复习**

制作出各种抽认卡后，需要我们在空余的时候去复习。编程累了就休息半个小时，并去复习抽认卡，如此反复。

**4. 专注**

在学习的过程中，往往会有许多令人分心的事占据着我们宝贵的时间。因此，专注和集中注意力是非常困难的。放点纯音乐能帮上一些忙。

## Let‘s go!!!

------

## 计算机基础

视频推荐：

- [ ] [王道考研 计算机网络](https://www.bilibili.com/video/BV19E411D78Q)

- [ ] [王道考研 计算机组成原理](https://www.bilibili.com/video/BV1BE411D7ii)

- [ ] [王道考研 操作系统](https://www.bilibili.com/video/BV1YE411D7nH)
- [ ] [信息安全——复旦大学（MOOC 优质课程）](https://www.bilibili.com/video/BV1jJ411X7nN)

相关书籍推荐：

* [计算机网络（谢希仁）](https://book.douban.com/subject/2970300/)
* [编码的奥秘（Charles Petzold）](https://book.douban.com/subject/1024570/)
* [深入理解计算机系统（Randal E.Bryant ）](https://book.douban.com/subject/5333562/)
* [信息安全原理与实践](https://book.douban.com/subject/24733262/)

## 数据结构

视频推荐：

- [ ] [Java 数据结构和算法](https://www.bilibili.com/video/BV1E4411H73v)

#### 数组（Arrays）
- 实现一个可自动调整大小的动态数组。
- [ ] 介绍：
    - [数组（视频）](https://www.coursera.org/learn/data-structures/lecture/OsBSF/arrays)
    - [UC Berkeley CS61B - 线性数组和多维数组（视频）](https://archive.org/details/ucberkeley_webcast_Wp8oiO_CZZE)（从15分32秒开始）
    - [数组的基础知识（视频）](https://archive.org/details/0102WhatYouShouldKnow/02_04-basicArrays.mp4)
    - [多维数组（视频）](https://archive.org/details/0102WhatYouShouldKnow/02_05-multidimensionalArrays.mp4)
    - [动态数组（视频）](https://www.coursera.org/learn/data-structures/lecture/EwbnV/dynamic-arrays)
    - [不规则数组（视频）](https://www.youtube.com/watch?v=1jtrQqYpt7g)
    - [不规则数组（视频）](https://archive.org/details/0102WhatYouShouldKnow/02_06-jaggedArrays.mp4)
    - [调整数组的大小（视频）](https://archive.org/details/0102WhatYouShouldKnow/03_01-resizableArrays.mp4)
- [ ] 实现一个动态数组（可自动调整大小的可变数组）：
    - [ ] 练习使用数组和指针去编码，并且指针是通过计算去跳转而不是使用索引
    - [ ] 通过分配内存来新建一个原生数据型数组
        - 可以使用 int 类型的数组，但不能使用其语法特性
        - 从大小为16或更大的数（使用2的倍数 —— 16、32、64、128）开始编写
    - [ ] size() —— 数组元素的个数
    - [ ] capacity() —— 可容纳元素的个数
    - [ ] is_empty()
    - [ ] at(index) —— 返回对应索引的元素，且若索引越界则愤然报错
    - [ ] push(item)
    - [ ] insert(index, item) —— 在指定索引中插入元素，并把后面的元素依次后移
    - [ ] prepend(item) —— 可以使用上面的 insert 函数，传参 index 为 0
    - [ ] pop() —— 删除在数组末端的元素，并返回其值
    - [ ] delete(index) —— 删除指定索引的元素，并把后面的元素依次前移
    - [ ] remove(item) —— 删除指定值的元素，并返回其索引（即使有多个元素）
    - [ ] find(item) —— 寻找指定值的元素并返回其中第一个出现的元素其索引，若未找到则返回 -1
    - [ ] resize(new_capacity) // 私有函数
        - 若数组的大小到达其容积，则变大一倍
        - 获取元素后，若数组大小为其容积的1/4，则缩小一半
- [ ] 时间复杂度
    - 在数组末端增加/删除、定位、更新元素，只允许占 O(1) 的时间复杂度（平摊（amortized）去分配内存以获取更多空间）
    - 在数组任何地方插入/移除元素，只允许 O(n) 的时间复杂度
- [ ] 空间复杂度
    - 因为在内存中分配的空间邻近，所以有助于提高性能
    - 空间需求 = （大于或等于 n 的数组容积）* 元素的大小。即便空间需求为 2n，其空间复杂度仍然是 O(n)

#### 链表（Linked Lists）
- [ ] 介绍：
    - [ ] [单向链表（视频）](https://www.coursera.org/learn/data-structures/lecture/kHhgK/singly-linked-lists)
    - [ ] [CS 61B —— 链表（一）（视频）](https://archive.org/details/ucberkeley_webcast_htzJdKoEmO0)
    - [ ] [CS 61B —— 链表（二）（视频）](https://archive.org/details/ucberkeley_webcast_-c4I3gFYe3w)
- [ ] [C 代码（视频）](https://www.youtube.com/watch?v=QN6FPiD0Gzo) ── 并非看完整个视频，只需要看关于节点结构和内存分配那一部分即可
- [ ] 链表 vs 数组：
    - [基本链表 Vs 数组（视频）](https://www.coursera.org/lecture/data-structures-optimizing-performance/core-linked-lists-vs-arrays-rjBs9)
    - [在现实中，链表 Vs 数组（视频）](https://www.coursera.org/lecture/data-structures-optimizing-performance/in-the-real-world-lists-vs-arrays-QUaUd)
- [ ] [为什么你需要避免使用链表（视频）](https://www.youtube.com/watch?v=YQs6IC-vgmo)
- [ ] 的确：你需要关于“指向指针的指针”的相关知识：（因为当你传递一个指针到一个函数时，该函数可能会改变指针所指向的地址）该页只是为了让你了解“指向指针的指针”这一概念。但我并不推荐这种链式遍历的风格。因为，这种风格的代码，其可读性和可维护性太低。
    - [指向指针的指针](https://www.eskimo.com/~scs/cclass/int/sx8.html)
- [ ] 实现（我实现了使用尾指针以及没有使用尾指针这两种情况）：
    - [ ] size() —— 返回链表中数据元素的个数
    - [ ] empty() —— 若链表为空则返回一个布尔值 true
    - [ ] value_at(index) —— 返回第 n 个元素的值（从0开始计算）
    - [ ] push_front(value) —— 添加元素到链表的首部
    - [ ] pop_front() —— 删除首部元素并返回其值
    - [ ] push_back(value) —— 添加元素到链表的尾部
    - [ ] pop_back() —— 删除尾部元素并返回其值
    - [ ] front() —— 返回首部元素的值
    - [ ] back() —— 返回尾部元素的值
    - [ ] insert(index, value) —— 插入值到指定的索引，并把当前索引的元素指向到新的元素
    - [ ] erase(index) —— 删除指定索引的节点
    - [ ] value_n_from_end(n) —— 返回倒数第 n 个节点的值
    - [ ] reverse() —— 逆序链表
    - [ ] remove_value(value) —— 删除链表中指定值的第一个元素
- [ ] 双向链表
    - [介绍（视频）](https://www.coursera.org/learn/data-structures/lecture/jpGKD/doubly-linked-lists)
    - 并不需要实现

#### 堆栈（Stack）
- [ ] [堆栈（视频）](https://www.coursera.org/learn/data-structures/lecture/UdKzQ/stacks)
- [ ] [使用堆栈 —— 后进先出（视频）](https://archive.org/details/0102WhatYouShouldKnow/05_01-usingStacksForLast-inFirst-out.mp4)
- [ ] 可以不实现，因为使用数组来实现并不重要

#### 队列（Queue）
- [ ] [使用队列 —— 先进先出（视频）](https://archive.org/details/0102WhatYouShouldKnow/05_03-usingQueuesForFirst-inFirst-out.mp4)
- [ ] [队列（视频）](https://www.coursera.org/learn/data-structures/lecture/EShpq/queue)
- [ ] [原型队列/先进先出（FIFO）](https://en.wikipedia.org/wiki/Circular_buffer)
- [ ] [优先级队列（视频）](https://archive.org/details/0102WhatYouShouldKnow/05_04-priorityQueuesAndDeques.mp4)
- [ ] 使用含有尾部指针的链表来实现:
    - enqueue(value) —— 在尾部添加值
    - dequeue() —— 删除最早添加的元素并返回其值（首部元素）
    - empty()
- [ ] 使用固定大小的数组实现：
    - enqueue(value) —— 在可容的情况下添加元素到尾部
    - dequeue() —— 删除最早添加的元素并返回其值
    - empty()
    - full()
- [ ] 花销：
    - 在糟糕的实现情况下，使用链表所实现的队列，其入列和出列的时间复杂度将会是 O(n)。因为，你需要找到下一个元素，以致循环整个队列
    - enqueue：O(1)（平摊（amortized）、链表和数组 [探测（probing）]）
    - dequeue：O(1)（链表和数组）
    - empty：O(1)（链表和数组）

#### 哈希表（Hash table）
- [ ] 视频：
    - [ ] [链式哈希表（视频）](https://www.youtube.com/watch?v=0M_kIqhwbFo&list=PLUl4u3cNGP61Oq3tWYp6V_F-5jb5L2iHb&index=8)
    - [ ] [Table Doubling 和 Karp-Rabin（视频）](https://www.youtube.com/watch?v=BRO7mVIFt08&index=9&list=PLUl4u3cNGP61Oq3tWYp6V_F-5jb5L2iHb)
    - [ ] [Open Addressing 和密码型哈希（Cryptographic Hashing）（视频）](https://www.youtube.com/watch?v=rvdJDijO2Ro&index=10&list=PLUl4u3cNGP61Oq3tWYp6V_F-5jb5L2iHb)
    - [ ] [PyCon 2010：The Mighty Dictionary（视频）](https://www.youtube.com/watch?v=C4Kc8xzcA68)
    - [ ] [（进阶）随机取样（Randomization）：全域哈希（Universal Hashing）& 完美哈希（Perfect Hashing）（视频）](https://www.youtube.com/watch?v=z0lJ2k0sl1g&list=PLUl4u3cNGP6317WaSNfmCvGym2ucw3oGp&index=11)
    - [ ] [（进阶）完美哈希（Perfect hashing）（视频）](https://www.youtube.com/watch?v=N0COwN14gt0&list=PL2B4EEwhKD-NbwZ4ezj7gyc_3yNrojKM9&index=4)

- [ ] 在线课程：
    - [ ] [理解哈希函数（视频）](https://archive.org/details/0102WhatYouShouldKnow/06_02-understandingHashFunctions.mp4)
    - [ ] [使用哈希表（视频）](https://archive.org/details/0102WhatYouShouldKnow/06_03-usingHashTables.mp4)
    - [ ] [支持哈希（视频）](https://archive.org/details/0102WhatYouShouldKnow/06_04-supportingHashing.mp4)
    - [ ] [哈希表的语言支持（视频）](https://archive.org/details/0102WhatYouShouldKnow/06_05-languageSupportForHashTables.mp4)
    - [ ] [基本哈希表（视频）](https://www.coursera.org/learn/data-structures-optimizing-performance/lecture/m7UuP/core-hash-tables)
    - [ ] [数据结构（视频）](https://www.coursera.org/learn/data-structures/home/week/3)
    - [ ] [电话薄问题（Phone Book Problem）（视频）](https://www.coursera.org/learn/data-structures/lecture/NYZZP/phone-book-problem)
    - [ ] 分布式哈希表：
        - [Dropbox 中的瞬时上传及存储优化（视频）](https://www.coursera.org/learn/data-structures/lecture/DvaIb/instant-uploads-and-storage-optimization-in-dropbox)
        - [分布式哈希表（视频）](https://www.coursera.org/learn/data-structures/lecture/tvH8H/distributed-hash-tables)

- [ ] 使用线性探测的数组去实现
    - hash(k, m) —— m 是哈希表的大小
    - add(key, value) —— 如果 key 已存在则更新值
    - exists(key)
    - get(key)
    - remove(key)

- - [ ] - 

#### 树（Trees）

- ### 树 —— 笔记 & 背景
    - [ ] [系列：树（视频）](https://www.coursera.org/learn/data-structures/lecture/95qda/trees)
    - 基本的树形结构
    - 遍历
    - 操作算法
    - [ ] [BFS（广度优先检索，breadth-first search）和 DFS（深度优先检索，depth-first search）](https://www.youtube.com/watch?v=uWL6FJhq5fM)
        - BFS 笔记
            - 层序遍历（使用队列的 BFS 算法）
            - 时间复杂度： O(n)
            - 空间复杂度：
                - 最好情况：O(1)
                - 最坏情况：O(n/2)=O(n)
        - DFS 笔记：
            - 时间复杂度：O(n)
            - 空间复杂度：
                - 最好情况：O(log n) - 树的平均高度
                - 最坏情况：O(n)
            - 中序遍历（DFS：左、节点本身、右）
            - 后序遍历（DFS：左、右、节点本身）
            - 先序遍历（DFS：节点本身、左、右）

- ### 二叉查找树（Binary search trees）：BSTs
    - [ ] [二叉查找树概览（视频）](https://www.youtube.com/watch?v=x6At0nzX92o&index=1&list=PLA5Lqm4uh9Bbq-E0ZnqTIa8LRaL77ica6)
    - [ ] [系列（视频）](https://www.coursera.org/learn/data-structures-optimizing-performance/lecture/p82sw/core-introduction-to-binary-search-trees)
        - 从符号表开始到 BST 程序
    - [ ] [介绍（视频）](https://www.coursera.org/learn/data-structures/lecture/E7cXP/introduction)
    - [ ] [MIT（视频）](https://www.youtube.com/watch?v=9Jry5-82I68)
    - C/C++:
        - [ ] [二叉查找树 —— 在 C/C++ 中实现（视频）](https://www.youtube.com/watch?v=COZK7NATh4k&list=PL2_aWCzGMAwI3W_JlcBbtYTwiQSsOTa6P&index=28)
        - [ ] [BST 的实现 —— 在堆栈和堆中的内存分配（视频）](https://www.youtube.com/watch?v=hWokyBoo0aI&list=PL2_aWCzGMAwI3W_JlcBbtYTwiQSsOTa6P&index=29)
        - [ ] [在二叉查找树中找到最小和最大的元素（视频）](https://www.youtube.com/watch?v=Ut90klNN264&index=30&list=PL2_aWCzGMAwI3W_JlcBbtYTwiQSsOTa6P)
        - [ ] [寻找二叉树的高度（视频）](https://www.youtube.com/watch?v=_pnqMz5nrRs&list=PL2_aWCzGMAwI3W_JlcBbtYTwiQSsOTa6P&index=31)
        - [ ] [二叉树的遍历 —— 广度优先和深度优先策略（视频）](https://www.youtube.com/watch?v=9RHO6jU--GU&list=PL2_aWCzGMAwI3W_JlcBbtYTwiQSsOTa6P&index=32)
        - [ ] [二叉树：层序遍历（视频）](https://www.youtube.com/watch?v=86g8jAQug04&index=33&list=PL2_aWCzGMAwI3W_JlcBbtYTwiQSsOTa6P)
        - [ ] [二叉树的遍历：先序、中序、后序（视频）](https://www.youtube.com/watch?v=gm8DUJJhmY4&index=34&list=PL2_aWCzGMAwI3W_JlcBbtYTwiQSsOTa6P)
        - [ ] [判断一棵二叉树是否为二叉查找树（视频）](https://www.youtube.com/watch?v=yEwSGhSsT0U&index=35&list=PL2_aWCzGMAwI3W_JlcBbtYTwiQSsOTa6P)
        - [ ] [从二叉查找树中删除一个节点（视频）](https://www.youtube.com/watch?v=gcULXE7ViZw&list=PL2_aWCzGMAwI3W_JlcBbtYTwiQSsOTa6P&index=36)
        - [ ] [二叉查找树中序遍历的后继者（视频）](https://www.youtube.com/watch?v=5cPbNCrdotA&index=37&list=PL2_aWCzGMAwI3W_JlcBbtYTwiQSsOTa6P)
    - [ ] 实现：
        - [ ] insert    // 往树上插值
        - [ ] get_node_count // 查找树上的节点数
        - [ ] print_values // 从小到大打印树中节点的值
        - [ ] delete_tree
        - [ ] is_in_tree // 如果值存在于树中则返回 true
        - [ ] get_height // 返回节点所在的高度（如果只有一个节点，那么高度则为1）
        - [ ] get_min   // 返回树上的最小值
        - [ ] get_max   // 返回树上的最大值
        - [ ] is_binary_search_tree
        - [ ] delete_value
        - [ ] get_successor // 返回给定值的后继者，若没有则返回-1

- ### 堆（Heap） / 优先级队列（Priority Queue） / 二叉堆（Binary Heap）
    - 可视化是一棵树，但通常是以线性的形式存储（数组、链表）
    - [ ] [堆](https://en.wikipedia.org/wiki/Heap_(data_structure))
    - [ ] [介绍（视频）](https://www.coursera.org/learn/data-structures/lecture/2OpTs/introduction)
    - [ ] [简单的实现（视频）](https://www.coursera.org/learn/data-structures/lecture/z3l9N/naive-implementations)
    - [ ] [二叉树（视频）](https://www.coursera.org/learn/data-structures/lecture/GRV2q/binary-trees)
    - [ ] [关于树高的讨论（视频）](https://www.coursera.org/learn/data-structures/supplement/S5xxz/tree-height-remark)
    - [ ] [基本操作（视频）](https://www.coursera.org/learn/data-structures/lecture/0g1dl/basic-operations)
    - [ ] [完全二叉树（视频）](https://www.coursera.org/learn/data-structures/lecture/gl5Ni/complete-binary-trees)
    - [ ] [伪代码（视频）](https://www.coursera.org/learn/data-structures/lecture/HxQo9/pseudocode)
    - [ ] [堆排序 —— 跳到起点（视频）](https://youtu.be/odNJmw5TOEE?list=PLFDnELG9dpVxQCxuD-9BSy2E7BWY3t5Sm&t=3291)
    - [ ] [堆排序（视频）](https://www.coursera.org/learn/data-structures/lecture/hSzMO/heap-sort)
    - [ ] [构建一个堆（视频）](https://www.coursera.org/learn/data-structures/lecture/dwrOS/building-a-heap)
    - [ ] [MIT：堆与堆排序（视频）](https://www.youtube.com/watch?v=B7hVxCmfPtM&index=4&list=PLUl4u3cNGP61Oq3tWYp6V_F-5jb5L2iHb)
    - [ ] [CS 61B Lecture 24：优先级队列（视频）](https://www.youtube.com/watch?v=yIUFT6AKBGE&index=24&list=PL4BBB74C7D2A1049C)
    - [ ] [构建线性时间复杂度的堆（大顶堆）](https://www.youtube.com/watch?v=MiyLo8adrWw)
    - [ ] 实现一个大顶堆：
        - [ ] insert
        - [ ] sift_up —— 用于插入元素
        - [ ] get_max —— 返回最大值但不移除元素
        - [ ] get_size() —— 返回存储的元素数量
        - [ ] is_empty() —— 若堆为空则返回 true
        - [ ] extract_max —— 返回最大值并移除
        - [ ] sift_down —— 用于获取最大值元素
        - [ ] remove(i) —— 删除指定索引的元素
        - [ ] heapify —— 构建堆，用于堆排序
        - [ ] heap_sort() —— 拿到一个未排序的数组，然后使用大顶堆或者小顶堆进行就地排序

#### 排序（Sorting）

- [ ] 笔记:
    - 实现各种排序，知道每种排序的最坏、最好和平均的复杂度分别是什么场景:
        - 不要用冒泡排序 - 效率太差 - 时间复杂度 O(n^2), 除非 n <= 16
    - [ ] 排序算法的稳定性 ("快排是稳定的么?")
        - [排序算法的稳定性](https://en.wikipedia.org/wiki/Sorting_algorithm#Stability)
        - [排序算法的稳定性](http://stackoverflow.com/questions/1517793/stability-in-sorting-algorithms)
        - [排序算法的稳定性](http://www.geeksforgeeks.org/stability-in-sorting-algorithms/)
        - [排序算法 - 稳定性](http://homepages.math.uic.edu/~leon/cs-mcs401-s08/handouts/stability.pdf)
    - [ ] 哪种排序算法可以用链表？哪种用数组？哪种两者都可？
        - 并不推荐对一个链表排序，但归并排序是可行的.
        - [链表的归并排序](http://www.geeksforgeeks.org/merge-sort-for-linked-list/)

- 关于堆排序，请查看前文堆的数据结构部分。堆排序很强大，不过是非稳定排序。
- [ ] [Sedgewick ── 归并排序（5个视频）](https://www.coursera.org/learn/algorithms-part1/home/week/3)
    - [ ] 1. 归并排序
    - [ ] 2. 自下而上的归并排序
    - [ ] 3. 排序复杂度
    - [ ] 4. 比较器
    - [ ] 5. 稳定性
- [ ] [Sedgewick ── 快速排序（4个视频）](https://www.coursera.org/learn/algorithms-part1/home/week/3)
    - [ ] 1. 快速排序
    - [ ] 2. 选择
    - [ ] 3. 重复键值
    - [ ] 4. 系统排序
- [ ] [冒泡排序（视频）](https://www.youtube.com/watch?v=P00xJgWzz2c&index=1&list=PL89B61F78B552C1AB)
- [ ] [冒泡排序分析（视频）](https://www.youtube.com/watch?v=ni_zk257Nqo&index=7&list=PL89B61F78B552C1AB)
- [ ] [插入排序 & 归并排序（视频）](https://www.youtube.com/watch?v=Kg4bqzAqRBM&index=3&list=PLUl4u3cNGP61Oq3tWYp6V_F-5jb5L2iHb)
- [ ] [插入排序（视频）](https://www.youtube.com/watch?v=c4BRHC7kTaQ&index=2&list=PL89B61F78B552C1AB)
- [ ] [归并排序（视频）](https://www.youtube.com/watch?v=GCae1WNvnZM&index=3&list=PL89B61F78B552C1AB)
- [ ] [快排（视频）](https://www.youtube.com/watch?v=y_G9BkAm6B8&index=4&list=PL89B61F78B552C1AB)
- [ ] [选择排序（视频）](https://www.youtube.com/watch?v=6nDMgr0-Yyo&index=8&list=PL89B61F78B552C1AB)

- [ ] 归并排序代码：
    - [ ] [使用外部数组（C语言）](http://www.cs.yale.edu/homes/aspnes/classes/223/examples/sorting/mergesort.c)
    - [ ] [使用外部数组（Python语言）](https://github.com/jwasham/practice-python/blob/master/merge_sort/merge_sort.py)
    - [ ] [对原数组直接排序（C++）](https://github.com/jwasham/practice-cpp/blob/master/merge_sort/merge_sort.cc)
- [ ] 快速排序代码：
    - [ ] [实现（C语言）](http://www.cs.yale.edu/homes/aspnes/classes/223/examples/randomization/quick.c)
    - [ ] [实现（C语言）](https://github.com/jwasham/practice-c/blob/master/quick_sort/quick_sort.c)
    - [ ] [实现（Python语言）](https://github.com/jwasham/practice-python/blob/master/quick_sort/quick_sort.py)

- [ ] 实现:
    - [ ] 归并：平均和最差情况的时间复杂度为 O(n log n)。
    - [ ] 快排：平均时间复杂度为 O(n log n)。
    - 选择排序和插入排序的最坏、平均时间复杂度都是 O(n^2)。
    - 关于堆排序，请查看前文堆的数据结构部分。

- [ ] 有兴趣的话，还有一些补充，但并不是必须的:
    - [Sedgewick──基数排序 (6个视频)](https://www.coursera.org/learn/algorithms-part2/home/week/3)
        - [ ] [1. Java 中的字符串](https://www.coursera.org/learn/algorithms-part2/lecture/vGHvb/strings-in-java)
        - [ ] [2. 键值索引计数（Key Indexed Counting）](https://www.coursera.org/learn/algorithms-part2/lecture/2pi1Z/key-indexed-counting)
        - [ ] [3. Least Significant Digit First String Radix Sort](https://www.coursera.org/learn/algorithms-part2/lecture/c1U7L/lsd-radix-sort)
        - [ ] [4. Most Significant Digit First String Radix Sort](https://www.coursera.org/learn/algorithms-part2/lecture/gFxwG/msd-radix-sort)
        - [ ] [5. 3中基数快速排序](https://www.coursera.org/learn/algorithms-part2/lecture/crkd5/3-way-radix-quicksort)
        - [ ] [6. 后继数组（Suffix Arrays）](https://www.coursera.org/learn/algorithms-part2/lecture/TH18W/suffix-arrays)
    - [ ] [基数排序](http://www.cs.yale.edu/homes/aspnes/classes/223/notes.html#radixSort)
    - [ ] [基数排序（视频）](https://www.youtube.com/watch?v=xhr26ia4k38)
    - [ ] [基数排序, 计数排序 (线性时间内)（视频）](https://www.youtube.com/watch?v=Nz1KZXbghj8&index=7&list=PLUl4u3cNGP61Oq3tWYp6V_F-5jb5L2iHb)
    - [ ] [随机算法: 矩阵相乘, 快排, Freivalds' 算法（视频）](https://www.youtube.com/watch?v=cNB2lADK3_s&index=8&list=PLUl4u3cNGP6317WaSNfmCvGym2ucw3oGp)
    - [ ] [线性时间内的排序（视频）](https://www.youtube.com/watch?v=pOKy3RZbSws&list=PLUl4u3cNGP61hsJNdULdudlRL493b-XZf&index=14)

总结一下，这是[15种排序算法](https://www.youtube.com/watch?v=kPRA0W1kECg)的可视化表示。如果你需要有关此主题的更多详细信息，请参阅“[一些主题的额外内容](#一些主题的额外内容)”中的“排序”部分。

#### 图（Graphs）

图论能解决计算机科学里的很多问题，所以这一节会比较长，像树和排序的部分一样。

- 笔记:
    - 有4种基本方式在内存里表示一个图:
        - 对象和指针
        - 邻接矩阵
        - 邻接表
        - 邻接图
    - 熟悉以上每一种图的表示法，并了解各自的优缺点
    - 广度优先搜索和深度优先搜索：知道它们的计算复杂度和设计上的权衡以及如何用代码实现它们
    - 遇到一个问题时，首先尝试基于图的解决方案，如果没有再去尝试其他的。

- MIT（视频）：
    - [广度优先搜索](https://www.youtube.com/watch?v=s-CYnVz-uh4&list=PLUl4u3cNGP61Oq3tWYp6V_F-5jb5L2iHb&index=13)
    - [深度优先搜索](https://www.youtube.com/watch?v=AfSk24UTFS8&list=PLUl4u3cNGP61Oq3tWYp6V_F-5jb5L2iHb&index=14)

- [ ] Skiena 教授的课程 - 很不错的介绍:
    - [ ] [CSE373 2012 - 课程 11 - 图的数据结构（视频）](https://www.youtube.com/watch?v=OiXxhDrFruw&list=PLOtl7M3yp-DV69F32zdK7YJcNXpTunF2b&index=11)
    - [ ] [CSE373 2012 - 课程 12 - 广度优先搜索（视频）](https://www.youtube.com/watch?v=g5vF8jscteo&list=PLOtl7M3yp-DV69F32zdK7YJcNXpTunF2b&index=12)
    - [ ] [CSE373 2012 - 课程 13 - 图的算法（视频）](https://www.youtube.com/watch?v=S23W6eTcqdY&list=PLOtl7M3yp-DV69F32zdK7YJcNXpTunF2b&index=13)
    - [ ] [CSE373 2012 - 课程 14 - 图的算法 (1)（视频）](https://www.youtube.com/watch?v=WitPBKGV0HY&index=14&list=PLOtl7M3yp-DV69F32zdK7YJcNXpTunF2b)
    - [ ] [CSE373 2012 - 课程 15 - 图的算法 (2)（视频）](https://www.youtube.com/watch?v=ia1L30l7OIg&index=15&list=PLOtl7M3yp-DV69F32zdK7YJcNXpTunF2b)
    - [ ] [CSE373 2012 - 课程 16 - 图的算法 (3)（视频）](https://www.youtube.com/watch?v=jgDOQq6iWy8&index=16&list=PLOtl7M3yp-DV69F32zdK7YJcNXpTunF2b)

- [ ] 图 (复习和其他):

    - [ ] [6.006 单源最短路径问题（视频）](https://www.youtube.com/watch?v=Aa2sqUhIn-E&index=15&list=PLUl4u3cNGP61Oq3tWYp6V_F-5jb5L2iHb)
    - [ ] [6.006 Dijkstra 算法（视频）](https://www.youtube.com/watch?v=2E7MmKv0Y24&index=16&list=PLUl4u3cNGP61Oq3tWYp6V_F-5jb5L2iHb)
    - [ ] [6.006 Bellman-Ford 算法（视频）](https://www.youtube.com/watch?v=ozsuci5pIso&list=PLUl4u3cNGP61Oq3tWYp6V_F-5jb5L2iHb&index=17)
    - [ ] [6.006 Dijkstra 效率优化（视频）](https://www.youtube.com/watch?v=CHvQ3q_gJ7E&list=PLUl4u3cNGP61Oq3tWYp6V_F-5jb5L2iHb&index=18)
    - [ ] [Aduni: 图的算法 I - 拓扑排序，最小生成树，Prim 算法 - 第六课（视频）](https://www.youtube.com/watch?v=i_AQT_XfvD8&index=6&list=PLFDnELG9dpVxQCxuD-9BSy2E7BWY3t5Sm)
    - [ ] [Aduni: 图的算法 II - 深度优先搜索, 广度优先搜索, Kruskal 算法, 并查集数据结构 - 第七课（视频）](https://www.youtube.com/watch?v=ufj5_bppBsA&list=PLFDnELG9dpVxQCxuD-9BSy2E7BWY3t5Sm&index=7)
    - [ ] [Aduni: 图的算法 III: 最短路径 - 第八课（视频）](https://www.youtube.com/watch?v=DiedsPsMKXc&list=PLFDnELG9dpVxQCxuD-9BSy2E7BWY3t5Sm&index=8)
    - [ ] [Aduni: 图的算法. IV: 几何算法介绍 - 第九课（视频）](https://www.youtube.com/watch?v=XIAQRlNkJAw&list=PLFDnELG9dpVxQCxuD-9BSy2E7BWY3t5Sm&index=9)
    - [ ] <del>[CS 61B 2014 (从 58:09 开始)（视频）](https://youtu.be/dgjX4HdMI-Q?list=PL-XXv-cvA_iAlnI-BQr9hjqADPBtujFJd&t=3489)</del>
    - [ ] [CS 61B 2014: 加权图（视频）](https://www.youtube.com/watch?v=aJjlQCFwylA&list=PL-XXv-cvA_iAlnI-BQr9hjqADPBtujFJd&index=19)
    - [ ] [贪心算法: 最小生成树（视频）](https://www.youtube.com/watch?v=tKwnms5iRBU&index=16&list=PLUl4u3cNGP6317WaSNfmCvGym2ucw3oGp)
    - [ ] [图的算法之强连通分量 Kosaraju 算法（视频）](https://www.youtube.com/watch?v=RpgcYiky7uw)

- 完整的 Coursera 课程:
    - [ ] [图的算法（视频）](https://www.coursera.org/learn/algorithms-on-graphs/home/welcome)

- 我会实现:
    - [ ] DFS 邻接表 (递归)
    - [ ] DFS 邻接表 (栈迭代)
    - [ ] DFS 邻接矩阵 (递归)
    - [ ] DFS 邻接矩阵 (栈迭代)
    - [ ] BFS 邻接表
    - [ ] BFS 邻接矩阵
    - [ ] 单源最短路径问题 (Dijkstra)
    - [ ] 最小生成树
    - 基于 DFS 的算法 (根据上文 Aduni 的视频):
        - [ ] 检查环 (我们会先检查是否有环存在以便做拓扑排序)
        - [ ] 拓扑排序
        - [ ] 计算图中的连通分支
        - [ ] 列出强连通分量
        - [ ] 检查双向图

可以从 Skiena 的书（参考下面的书推荐小节）和面试书籍中学习更多关于图的实践。

#### 更多知识

- ### 面向对象编程
    - 可选：[UML 2.0系列（视频）](https://www.youtube.com/watch?v=OkC7HKtiZC0&list=PLGLfVvz_LVvQ5G-LdJ8RLqe-ndo7QITYc)
    - SOLID 面向对象编程原则：[SOLID 原则（视频）](https://www.youtube.com/playlist?list=PL4CE9F710017EA77A)

- ### 设计模式
    - [ ] [UML 统一建模语言概览 (视频)](https://www.youtube.com/watch?v=3cmzqZzwNDM&list=PLGLfVvz_LVvQ5G-LdJ8RLqe-ndo7QITYc&index=3)
    - [ ] 主要有如下的设计模式:
        - [ ] 策略模式（strategy）
        - [ ] 单例模式（singleton）
        - [ ] 适配器模式（adapter）
        - [ ] 原型模式（prototype）
        - [ ] 装饰器模式（decorator）
        - [ ] 访问者模式（visitor）
        - [ ] 工厂模式，抽象工厂模式（factory, abstract factory）
        - [ ] 外观模式（facade）
        - [ ] 观察者模式（observer）
        - [ ] 代理模式（proxy）
        - [ ] 委托模式（delegate）
        - [ ] 命令模式（command）
        - [ ] 状态模式（state）
        - [ ] 备忘录模式（memento）
        - [ ] 迭代器模式（iterator）
        - [ ] 组合模式（composite）
        - [ ] 享元模式（flyweight）
    - [ ] [第六章 (第 1 部分 ) - 设计模式 (视频)](https://youtu.be/LAP2A80Ajrg?list=PLJ9pm_Rc9HesnkwKlal_buSIHA-jTZMpO&t=3344)
    - [ ] [第六章 (第 2 部分 ) - Abstraction-Occurrence, General Hierarchy, Player-Role, Singleton, Observer, Delegation (视频)](https://www.youtube.com/watch?v=U8-PGsjvZc4&index=12&list=PLJ9pm_Rc9HesnkwKlal_buSIHA-jTZMpO)
    - [ ] [第六章 (第 3 部分 ) - Adapter, Facade, Immutable, Read-Only Interface, Proxy（视频）](https://www.youtube.com/watch?v=7sduBHuex4c&index=13&list=PLJ9pm_Rc9HesnkwKlal_buSIHA-jTZMpO)
    - [ ] [系列视频（27个）](https://www.youtube.com/playlist?list=PLF206E906175C7E07)
    - [ ] [Head First 设计模型](https://www.amazon.com/Head-First-Design-Patterns-Freeman/dp/0596007124)
        - 尽管《设计模式：可复用面向对象软件的基础》才是这方面的经典，但是我还是认为Head First对于新手更加友好。
    - [ ] [实际操作：设计模式和对入门开发者的建议](https://sourcemaking.com/design-patterns-and-tips)
    - [ ] [Design patterns for humans](https://github.com/kamranahmedse/design-patterns-for-humans#structural-design-patterns)

- ### 组合（Combinatorics） (n 中选 k 个) & 概率（Probability）
    - [ ] [数据技巧: 如何找出阶乘、排列和组合(选择)（视频）](https://www.youtube.com/watch?v=8RRo6Ti9d0U)
    - [ ] [来点学校的东西: 概率（视频）](https://www.youtube.com/watch?v=sZkAAk9Wwa4)
    - [ ] [来点学校的东西: 概率和马尔可夫链（视频）](https://www.youtube.com/watch?v=dNaJg-mLobQ)
    - [ ] 可汗学院:
        - 课程设置:
            - [ ] [概率理论基础](https://www.khanacademy.org/math/probability/probability-and-combinatorics-topic)
        - 只有视频 - 41 (每一个都短小精悍):
            - [ ] [概率解释（视频）](https://www.youtube.com/watch?v=uzkc-qNVoOk&list=PLC58778F28211FA19)

- ### NP, NP-完全和近似算法
    - 知道最经典的一些 NP 完全问题，比如旅行商问题和背包问题，而且能在面试官试图忽悠你的时候识别出他们。
    - 知道 NP 完全是什么意思.
    - [ ] [计算复杂度（视频）](https://www.youtube.com/watch?v=moPtwq_cVH8&list=PLUl4u3cNGP61Oq3tWYp6V_F-5jb5L2iHb&index=23)
    - [ ] Simonson:
        - [ ] [贪心算法. II & 介绍 NP-完全性（视频）](https://youtu.be/qcGnJ47Smlo?list=PLFDnELG9dpVxQCxuD-9BSy2E7BWY3t5Sm&t=2939)
        - [ ] [NP-完全性 II & 归约（视频）](https://www.youtube.com/watch?v=e0tGC6ZQdQE&index=16&list=PLFDnELG9dpVxQCxuD-9BSy2E7BWY3t5Sm)
        - [ ] [NP-完全性 III（视频）](https://www.youtube.com/watch?v=fCX1BGT3wjE&index=17&list=PLFDnELG9dpVxQCxuD-9BSy2E7BWY3t5Sm)
        - [ ] [NP-完全性 IV（视频）](https://www.youtube.com/watch?v=NKLDp3Rch3M&list=PLFDnELG9dpVxQCxuD-9BSy2E7BWY3t5Sm&index=18)
    - [ ] Skiena:
        - [ ] [CSE373 2012 - 课程 23 - 介绍 NP-完全性 IV（视频）](https://youtu.be/KiK5TVgXbFg?list=PLOtl7M3yp-DV69F32zdK7YJcNXpTunF2b&t=1508)
        - [ ] [CSE373 2012 - 课程 24 - NP-完全性证明（视频）](https://www.youtube.com/watch?v=27Al52X3hd4&index=24&list=PLOtl7M3yp-DV69F32zdK7YJcNXpTunF2b)
        - [ ] [CSE373 2012 - 课程 25 - NP-完全性挑战（视频）](https://www.youtube.com/watch?v=xCPH4gwIIXM&index=25&list=PLOtl7M3yp-DV69F32zdK7YJcNXpTunF2b)
    - [ ] [复杂度: P, NP, NP-完全性, 规约（视频）](https://www.youtube.com/watch?v=eHZifpgyH_4&list=PLUl4u3cNGP6317WaSNfmCvGym2ucw3oGp&index=22)
    - [ ] [复杂度: 近视算法 Algorithms（视频）](https://www.youtube.com/watch?v=MEz1J9wY2iM&list=PLUl4u3cNGP6317WaSNfmCvGym2ucw3oGp&index=24)
    - [ ] [复杂度: 固定参数算法（视频）](https://www.youtube.com/watch?v=4q-jmGrmxKs&index=25&list=PLUl4u3cNGP6317WaSNfmCvGym2ucw3oGp)
    - Peter Norvik 讨论旅行商问题的近似最优解:
        - [Jupyter 笔记本](http://nbviewer.jupyter.org/url/norvig.com/ipython/TSP.ipynb)
    - 《算法导论》（CLRS）的第 1048 - 1140 页。

- ### 缓存（Cache）
    - [ ] LRU 缓存:
        - [ ] [LRU 的魔力 (100 Days of Google Dev)（视频）](https://www.youtube.com/watch?v=R5ON3iwx78M)
        - [ ] [实现 LRU（视频）](https://www.youtube.com/watch?v=bq6N7Ym81iI)
        - [ ] [LeetCode - 146 LRU Cache (C++)（视频）](https://www.youtube.com/watch?v=8-FZRAjR7qU)
    - [ ] CPU 缓存:
        - [ ] [MIT 6.004 L15: 存储体系（视频）](https://www.youtube.com/watch?v=vjYF_fAZI5E&list=PLrRW1w6CGAcXbMtDFj205vALOGmiRc82-&index=24)
        - [ ] [MIT 6.004 L16: 缓存的问题（视频）](https://www.youtube.com/watch?v=ajgC3-pyGlk&index=25&list=PLrRW1w6CGAcXbMtDFj205vALOGmiRc82-)

- ### 进程（Processe）和线程（Thread）
    - [ ] 计算机科学 162 - 操作系统 (25 个视频):
        - 视频 1-11 是关于进程和线程
        - [操作系统和系统编程（视频）](https://archive.org/details/ucberkeley-webcast-PL-XXv-cvA_iBDyz-ba4yDskqMDY6A1w_c)
    - [进程和线程的区别是什么?](https://www.quora.com/What-is-the-difference-between-a-process-and-a-thread)
    - 涵盖了:
        - 进程、线程、协程
            - 进程和线程的区别
            - 进程
            - 线程
            - 锁
            - 互斥
            - 信号量
            - 监控
            - 他们是如何工作的
            - 死锁
            - 活锁
        - CPU 活动, 中断, 上下文切换
        - 现代多核处理器的并发式结构
        - [分页（paging），分段（segmentation）和虚拟内存（视频）](https://www.youtube.com/watch?v=LKe7xK0bF7o&list=PLCiOXwirraUCBE9i_ukL8_Kfg6XNv7Se8&index=2)
        - [中断（视频）](https://www.youtube.com/watch?v=uFKi2-J-6II&list=PLCiOXwirraUCBE9i_ukL8_Kfg6XNv7Se8&index=3)
        - 进程资源需要（内存：代码、静态存储器、栈、堆、文件描述符、I/O）
        - 线程资源需要（在同一个进程内和其他线程共享以上（除了栈）的资源，但是每个线程都有独立的程序计数器、栈计数器、寄存器和栈）
        - Fork 操作是真正的写时复制（只读），直到新的进程写到内存中，才会生成一份新的拷贝。
        - 上下文切换
            - 操作系统和底层硬件是如何初始化上下文切换的？
    - [ ] [C++ 的线程 (系列 - 10 个视频)](https://www.youtube.com/playlist?list=PL5jc9xFGsL8E12so1wlMS0r0hTQoJL74M)
    - [ ] Python 的并发 (视频):
        - [ ] [线程系列](https://www.youtube.com/playlist?list=PL1H1sBF1VAKVMONJWJkmUh6_p8g4F2oy1)
        - [ ] [Python 线程](https://www.youtube.com/watch?v=Bs7vPNbB9JM)
        - [ ] [理解 Python 的 GIL (2010)](https://www.youtube.com/watch?v=Obt-vMVdM8s)
            - [参考](http://www.dabeaz.com/GIL)
        - [ ] [David Beazley - Python 协程 - PyCon 2015](https://www.youtube.com/watch?v=MCs5OvhV9S4)
        - [ ] [Keynote David Beazley - 兴趣主题 (Python 异步 I/O)](https://www.youtube.com/watch?v=ZzfHjytDceU)
        - [ ] [Python 中的互斥](https://www.youtube.com/watch?v=0zaPs8OtyKY)

- ### 测试
    - 涵盖了:
        - 单元测试是如何工作的
        - 什么是模拟对象
        - 什么是集成测试
        - 什么是依赖注入
    - [ ] [James Bach 讲敏捷软件测试（视频）](https://www.youtube.com/watch?v=SAhJf36_u5U)
    - [ ] [James Bach 软件测试公开课（视频）](https://www.youtube.com/watch?v=ILkT_HV9DVU)
    - [ ] [Steve Freeman - 测试驱动的开发（视频）](https://vimeo.com/83960706)
        - [slides](http://gotocon.com/dl/goto-berlin-2013/slides/SteveFreeman_TestDrivenDevelopmentThatsNotWhatWeMeant.pdf)
    - [ ] [Python：测试驱动的 Web 开发](http://www.obeythetestinggoat.com/pages/book.html#toc)
    - [ ] 依赖注入:
        - [ ] [视频](https://www.youtube.com/watch?v=IKD2-MAkXyQ)
        - [ ] [测试之道](http://jasonpolites.github.io/tao-of-testing/ch3-1.1.html)
    - [ ] [如何编写测试](http://jasonpolites.github.io/tao-of-testing/ch4-1.1.html)

- ### 调度
    - 在操作系统中是如何运作的
    - 在操作系统部分的视频里有很多资料

- ### 字符串搜索和操作
    - [ ] [Sedgewick──后缀数组（Suffix Arrays）（视频）](https://www.coursera.org/learn/algorithms-part2/lecture/TH18W/suffix-arrays)
    - [ ] [Sedgewick──子字符串搜寻（视频）](https://www.coursera.org/learn/algorithms-part2/home/week/4)
        - [ ] [1. 子字符串搜寻导论](https://www.coursera.org/learn/algorithms-part2/lecture/n3ZpG/introduction-to-substring-search)
        - [ ] [2. 子字符串搜寻──暴力法](https://www.coursera.org/learn/algorithms-part2/lecture/2Kn5i/brute-force-substring-search)
        - [ ] [3. KMP算法](https://www.coursera.org/learn/algorithms-part2/lecture/TAtDr/knuth-morris-pratt)
        - [ ] [4. Boyer-Moore算法](https://www.coursera.org/learn/algorithms-part2/lecture/CYxOT/boyer-moore)
        - [ ] [5. Rabin-Karp算法](https://www.coursera.org/learn/algorithms-part2/lecture/3KiqT/rabin-karp)
    - [ ] [文本的搜索模式（视频）](https://www.coursera.org/learn/data-structures/lecture/tAfHI/search-pattern-in-text)

如果你需要有关此主题的更多详细信息，请参阅“[一些主题的额外内容](#一些主题的额外内容)”中的“字符串匹配”部分。

- ### 字典树（Tries）

    - 需要注意的是，字典树各式各样。有些有前缀，而有些则没有。有些使用字符串而不使用比特位来追踪路径。
    - 阅读代码，但不实现。
    - [Sedgewick──字典树（3个视频）](https://www.coursera.org/learn/algorithms-part2/home/week/4)
        - [ ] [1. R Way字典树](https://www.coursera.org/learn/algorithms-part2/lecture/CPVdr/r-way-tries)
        - [ ] [2. 三元搜索树](https://www.coursera.org/learn/algorithms-part2/lecture/yQM8K/ternary-search-tries)
        - [ ] [3. 基于字符串的操作](https://www.coursera.org/learn/algorithms-part2/lecture/jwNmV/character-based-operations)
    - [ ] [数据结构笔记及编程技术](http://www.cs.yale.edu/homes/aspnes/classes/223/notes.html#Tries)
    - [ ] 短课程视频：
        - [ ] [对字典树的介绍（视频）](https://www.coursera.org/learn/data-structures-optimizing-performance/lecture/08Xyf/core-introduction-to-tries)
        - [ ] [字典树的性能（视频）](https://www.coursera.org/learn/data-structures-optimizing-performance/lecture/PvlZW/core-performance-of-tries)
        - [ ] [实现一棵字典树（视频）](https://www.coursera.org/learn/data-structures-optimizing-performance/lecture/DFvd3/core-implementing-a-trie)
    - [ ] [字典树：一个被忽略的数据结构](https://www.toptal.com/java/the-trie-a-neglected-data-structure)
    - [ ] [TopCoder —— 使用字典树](https://www.topcoder.com/community/data-science/data-science-tutorials/using-tries/)
    - [ ] [标准教程（现实中的用例）（视频）](https://www.youtube.com/watch?v=TJ8SkcUSdbU)
    - [ ] [MIT，高阶数据结构，字符串（视频中间有点困难）（视频）](https://www.youtube.com/watch?v=NinWEPPrkDQ&index=16&list=PLUl4u3cNGP61hsJNdULdudlRL493b-XZf)

- ### 浮点数

    - [ ] 简单8位：[浮点数的表示形式-1（视频──计算中存在错误，请参见视频说明）](https://www.youtube.com/watch?v=ji3SfClm8TU)
    - [ ] 32位：[IEEE754 32位浮点二进制（视频）](https://www.youtube.com/watch?v=50ZYcZebIec)

- ### Unicode
    - [ ] [每一个软件开发者的绝对最低限度，必须要知道的关于 Unicode 和字符集知识](http://www.joelonsoftware.com/articles/Unicode.html)
    - [ ] [关于处理文本需要的编码和字符集，每个程序员绝对需要知道的知识](http://kunststube.net/encoding/)

- ### 字节序（Endianness）
    - [大/小端序](https://web.archive.org/web/20180107141940/http://www.cs.umd.edu:80/class/sum2003/cmsc311/Notes/Data/endian.html)
    - [大端序 Vs 小端序（视频）](https://www.youtube.com/watch?v=JrNF0KRAlyo)
    - [由里入内的大端序与小端序（视频）](https://www.youtube.com/watch?v=oBSuXP-1Tc0)
        - 对于内核开发非常具有技术性，如果大多数的内容听不懂也没关系。
        - 前半部就已经足够了。

- ### 网络（视频）
    - **如果你具有网络经验或想成为可靠性工程师或运维工程师，期待你的问题**
    - 知道这些有益无害，多多益善!
    - [ ] [可汗学院](https://www.khanacademy.org/computing/computer-science/computers-and-internet-code-org)
    - [ ] [UDP 和 TCP：网络传输协议中的数据压缩（视频）](https://www.youtube.com/watch?v=Vdc8TCESIg8)
    - [ ] [TCP/IP 和 OSI 模型解释！（视频）](https://www.youtube.com/watch?v=e5DEVa9eSN0)
    - [ ] [互联网上的数据包传输。网络和 TCP/IP 教程。（视频）](https://www.youtube.com/watch?v=nomyRJehhnM)
    - [ ] [HTTP（视频）](https://www.youtube.com/watch?v=WGJrLqtX7As)
    - [ ] [SSL 和 HTTPS（视频）](https://www.youtube.com/watch?v=S2iBR2ZlZf0)
    - [ ] [SSL/TLS（视频）](https://www.youtube.com/watch?v=Rp3iZUvXWlM)
    - [ ] [HTTP 2.0（视频）](https://www.youtube.com/watch?v=E9FxNzv1Tr8)
    - [ ] [视频系列（21个视频）](https://www.youtube.com/playlist?list=PLEbnTDJUr_IegfoqO4iPnPYQui46QqT0j)
    - [ ] [子网络解密 - 第五部分 经典内部域名指向 CIDR 标记（视频）](https://www.youtube.com/watch?v=t5xYI0jzOf4)
    - [ ] 套接字（Sockets）：
        - [Java──套接字──介绍（视频）](https://www.youtube.com/watch?v=6G_W54zuadg&t=6s)
        - [套接字编程（视频）](https://www.youtube.com/watch?v=G75vN2mnJeQ)

## 算法

[算法（Sedgewick 和 Wayne）](https://www.amazon.com/Algorithms-4th-Robert-Sedgewick/dp/032157351X/)

- 包含课程内容（和Sedgewick！）的视频
  - [第一部分](https://www.coursera.org/learn/algorithms-part1)
  - [第二部分](https://www.coursera.org/learn/algorithms-part2)

### 算法复杂度 / Big-O / 渐进分析法

- 并不需要实现
- 这里有很多视频，看到你真正了解它为止。你随时可以回来复习。
- 如果这些课程太过数学的话，你可以去看看最下面离散数学的视频，它能让你更了解这些数学背后的来源以及原理。
- [ ] [Harvard CS50 —— 渐进表示（视频）](https://www.youtube.com/watch?v=iOq5kSKqeR4)
- [ ] [Big O 记号（通用快速教程）（视频）](https://www.youtube.com/watch?v=V6mKVRU1evU)
- [ ] [Big O 记号（以及 Omega 和 Theta）——  最佳数学解释（视频）](https://www.youtube.com/watch?v=ei-A_wy5Yxw&index=2&list=PL1BaGV1cIH4UhkL8a9bJGG356covJ76qN)
- [ ] Skiena 算法：
  - [视频](https://www.youtube.com/watch?v=gSyDMtdPNpU&index=2&list=PLOtl7M3yp-DV69F32zdK7YJcNXpTunF2b)
  - [幻灯片](http://www3.cs.stonybrook.edu/~algorith/video-lectures/2007/lecture2.pdf)
- [ ] [对于算法复杂度分析的一次详细介绍](http://discrete.gr/complexity/)
- [ ] [增长阶数（Orders of Growth）（视频）](https://class.coursera.org/algorithmicthink1-004/lecture/59)
- [ ] [渐进性（Asymptotics）（视频）](https://class.coursera.org/algorithmicthink1-004/lecture/61)
- [ ] [UC Berkeley Big O（视频）](https://youtu.be/VIS4YDpuP98)
- [ ] [UC Berkeley Big Omega（视频）](https://youtu.be/ca3e7UVmeUc)
- [ ] [平摊分析法（Amortized Analysis）（视频）](https://www.youtube.com/watch?v=B3SpQZaAZP4&index=10&list=PL1BaGV1cIH4UhkL8a9bJGG356covJ76qN)
- [ ] [举证“Big O”（视频）](https://class.coursera.org/algorithmicthink1-004/lecture/63)
- [ ] TopCoder（包括递归关系和主定理）：
  - [计算性复杂度：第一部](https://www.topcoder.com/community/data-science/data-science-tutorials/computational-complexity-section-1/)
  - [计算性复杂度：第二部](https://www.topcoder.com/community/data-science/data-science-tutorials/computational-complexity-section-2/)
- [ ] [速查表（Cheat sheet）](http://bigocheatsheet.com/)

### 更多的知识

- ### 二分查找（Binary search）

  - [ ] [二分查找（视频）](https://www.youtube.com/watch?v=D5SrAga1pno)
  - [ ] [二分查找（视频）](https://www.khanacademy.org/computing/computer-science/algorithms/binary-search/a/binary-search)
  - [ ] [详情](https://www.topcoder.com/community/data-science/data-science-tutorials/binary-search/)
  - [ ] 实现：
    - 二分查找（在一个已排序好的整型数组中查找）
    - 迭代式二分查找

- ### 按位运算（Bitwise operations）

  - [ ] [Bits 速查表](https://github.com/jwasham/coding-interview-university/blob/master/extras/cheat%20sheets/bits-cheat-sheet.pdf) ── 你需要知道大量2的幂数值（从2^1 到 2^16 及 2^32）
  - [ ] 好好理解位操作符的含义：&、|、^、~、>>、<<
    - [ ] [字码（words）](https://en.wikipedia.org/wiki/Word_(computer_architecture))
    - [ ] 好的介绍：
      [位操作（视频）](https://www.youtube.com/watch?v=7jkIUgLC29I)
    - [ ] [C 语言编程教程 2-10：按位运算（视频）](https://www.youtube.com/watch?v=d0AwjSpNXR0)
    - [ ] [位操作](https://en.wikipedia.org/wiki/Bit_manipulation)
    - [ ] [按位运算](https://en.wikipedia.org/wiki/Bitwise_operation)
    - [ ] [Bithacks](https://graphics.stanford.edu/~seander/bithacks.html)
    - [ ] [位元抚弄者（The Bit Twiddler）](http://bits.stephan-brumme.com/)
    - [ ] [交互式位元抚弄者（The Bit Twiddler Interactive）](http://bits.stephan-brumme.com/interactive.html)
    - [ ] [位操作技巧（Bit Hacks）（视频）](https://www.youtube.com/watch?v=ZusiKXcz_ac)
    - [ ] [练习位操作](https://pconrad.github.io/old_pconrad_cs16/topics/bitOps/)
  - [ ] 一补数和补码
    - [二进制：利 & 弊（为什么我们要使用补码）（视频）](https://www.youtube.com/watch?v=lKTsv6iVxV4)
    - [一补数（1s Complement）](https://en.wikipedia.org/wiki/Ones%27_complement)
    - [补码（2s Complement）](https://en.wikipedia.org/wiki/Two%27s_complement)
  - [ ] 计算置位（Set Bits）
    - [计算一个字节中置位（Set Bits）的四种方式（视频）](https://youtu.be/Hzuzo9NJrlc)
    - [计算比特位](https://graphics.stanford.edu/~seander/bithacks.html#CountBitsSetKernighan)
    - [如何在一个 32 位的整型中计算置位（Set Bits）的数量](http://stackoverflow.com/questions/109023/how-to-count-the-number-of-set-bits-in-a-32-bit-integer)
  - [ ] 交换值：
    - [交换（Swap）](http://bits.stephan-brumme.com/swap.html)
  - [ ] 绝对值：
    - [绝对整型（Absolute Integer）](http://bits.stephan-brumme.com/absInteger.html)

- ### 递归（Recursion）

  - [ ] Stanford 大学关于递归 & 回溯的课程:
    - [ ] [课程 8 | 抽象编程（视频）](https://www.youtube.com/watch?v=gl3emqCuueQ&list=PLFE6E58F856038C69&index=8)
    - [ ] [课程 9 | 抽象编程（视频）](https://www.youtube.com/watch?v=uFJhEPrbycQ&list=PLFE6E58F856038C69&index=9)
    - [ ] [课程 10 | 抽象编程（视频）](https://www.youtube.com/watch?v=NdF1QDTRkck&index=10&list=PLFE6E58F856038C69)
    - [ ] [课程 11 | 抽象编程（视频）](https://www.youtube.com/watch?v=p-gpaIGRCQI&list=PLFE6E58F856038C69&index=11)
  - 什么时候适合使用
  - 尾递归会更好么?
    - [ ] [什么是尾递归以及为什么它如此糟糕?](https://www.quora.com/What-is-tail-recursion-Why-is-it-so-bad)
    - [ ] [尾递归（视频）](https://www.youtube.com/watch?v=L1jjXGfxozc)

- ### 动态规划（Dynamic Programming）

  - 在你的面试中或许没有任何动态规划的问题，但能够知道一个题目可以使用动态规划来解决是很重要的。
  - 这一部分会有点困难，每个可以用动态规划解决的问题都必须先定义出递推关系，要推导出来可能会有点棘手。
  - 我建议先阅读和学习足够多的动态规划的例子，以便对解决 DP 问题的一般模式有个扎实的理解。
  - [ ] 视频:
    - Skiena 的视频可能会有点难跟上，有时候他用白板写的字会比较小，难看清楚。
    - [ ] [Skiena: CSE373 2012 - 课程 19 - 动态规划介绍（视频）](https://youtu.be/Qc2ieXRgR0k?list=PLOtl7M3yp-DV69F32zdK7YJcNXpTunF2b&t=1718)
    - [ ] [Skiena: CSE373 2012 - 课程 20 - 编辑距离（视频）](https://youtu.be/IsmMhMdyeGY?list=PLOtl7M3yp-DV69F32zdK7YJcNXpTunF2b&t=2749)
    - [ ] [Skiena: CSE373 2012 - 课程 21 - 动态规划举例（视频）](https://youtu.be/o0V9eYF4UI8?list=PLOtl7M3yp-DV69F32zdK7YJcNXpTunF2b&t=406)
    - [ ] [Skiena: CSE373 2012 - 课程 22 - 动态规划应用（视频）](https://www.youtube.com/watch?v=dRbMC1Ltl3A&list=PLOtl7M3yp-DV69F32zdK7YJcNXpTunF2b&index=22)
    - [ ] [Simonson: 动态规划 0 (starts at 59:18)（视频）](https://youtu.be/J5aJEcOr6Eo?list=PLFDnELG9dpVxQCxuD-9BSy2E7BWY3t5Sm&t=3558)
    - [ ] [Simonson: 动态规划 I - 课程 11（视频）](https://www.youtube.com/watch?v=0EzHjQ_SOeU&index=11&list=PLFDnELG9dpVxQCxuD-9BSy2E7BWY3t5Sm)
    - [ ] [Simonson: 动态规划 II - 课程 12（视频）](https://www.youtube.com/watch?v=v1qiRwuJU7g&list=PLFDnELG9dpVxQCxuD-9BSy2E7BWY3t5Sm&index=12)
    - [ ] 单独的 DP 问题 (每一个视频都很短)：[动态规划（视频）](https://www.youtube.com/playlist?list=PLrmLmBdmIlpsHaNTPP_jHHDx_os9ItYXr)
  - [ ] 耶鲁课程笔记:
    - [ ] [动态规划](http://www.cs.yale.edu/homes/aspnes/classes/223/notes.html#dynamicProgramming)
  - [ ] Coursera 课程:
    - [ ] [RNA 二级结构问题（视频）](https://www.coursera.org/learn/algorithmic-thinking-2/lecture/80RrW/the-rna-secondary-structure-problem)
    - [ ] [动态规划算法（视频）](https://www.coursera.org/learn/algorithmic-thinking-2/lecture/PSonq/a-dynamic-programming-algorithm)
    - [ ] [DP 算法描述（视频）](https://www.coursera.org/learn/algorithmic-thinking-2/lecture/oUEK2/illustrating-the-dp-algorithm)
    - [ ] [DP 算法的运行时间（视频）](https://www.coursera.org/learn/algorithmic-thinking-2/lecture/nfK2r/running-time-of-the-dp-algorithm)
    - [ ] [DP vs 递归实现（视频）](https://www.coursera.org/learn/algorithmic-thinking-2/lecture/M999a/dp-vs-recursive-implementation)
    - [ ] [全局成对序列排列（视频）](https://www.coursera.org/learn/algorithmic-thinking-2/lecture/UZ7o6/global-pairwise-sequence-alignment)
    - [ ] [本地成对序列排列（视频）](https://www.coursera.org/learn/algorithmic-thinking-2/lecture/WnNau/local-pairwise-sequence-alignment)

- ### 增强数据结构
    - [CS 61B 第 39 课: 增强数据结构](https://youtu.be/zksIj9O8_jc?list=PL4BBB74C7D2A1049C&t=950)

- ### 平衡查找树（Balanced search trees）
    - 掌握至少一种平衡查找树（并懂得如何实现）：
    - “在各种平衡查找树当中，AVL 树和2-3树已经成为了过去，而红黑树（red-black trees）看似变得越来越受人青睐。这种令人特别感兴趣的数据结构，亦称伸展树（splay tree）。它可以自我管理，且会使用轮换来移除任何访问过根节点的键。” —— Skiena
    - 因此，在各种各样的平衡查找树当中，我选择了伸展树来实现。虽然，通过我的阅读，我发现在面试中并不会被要求实现一棵平衡查找树。但是，为了胜人一筹，我们还是应该看看如何去实现。在阅读了大量关于红黑树的代码后，我才发现伸展树的实现确实会使得各方面更为高效。
        - 伸展树：插入、查找、删除函数的实现，而如果你最终实现了红黑树，那么请尝试一下：
        - 跳过删除函数，直接实现搜索和插入功能
    - 我希望能阅读到更多关于 B 树的资料，因为它也被广泛地应用到大型的数据集当中。
    - [自平衡二叉查找树](https://en.wikipedia.org/wiki/Self-balancing_binary_search_tree)

    - **AVL 树**
        - 实际中：我能告诉你的是，该种树并无太多的用途，但我能看到有用的地方在哪里：AVL 树是另一种平衡查找树结构。其可支持时间复杂度为 O(log n) 的查询、插入及删除。它比红黑树严格意义上更为平衡，从而导致插入和删除更慢，但遍历却更快。正因如此，才彰显其结构的魅力。只需要构建一次，就可以在不重新构造的情况下读取，适合于实现诸如语言字典（或程序字典，如一个汇编程序或解释程序的操作码）。
        - [MIT AVL 树 / AVL 树的排序（视频）](https://www.youtube.com/watch?v=FNeL18KsWPc&list=PLUl4u3cNGP61Oq3tWYp6V_F-5jb5L2iHb&index=6)
        - [AVL 树（视频）](https://www.coursera.org/learn/data-structures/lecture/Qq5E0/avl-trees)
        - [AVL 树的实现（视频）](https://www.coursera.org/learn/data-structures/lecture/PKEBC/avl-tree-implementation)
        - [分离与合并](https://www.coursera.org/learn/data-structures/lecture/22BgE/split-and-merge)

    - **伸展树**
        - 实际中：伸展树一般用于缓存、内存分配者、路由器、垃圾回收者、数据压缩、ropes（字符串的一种替代品，用于存储长串的文本字符）、Windows NT（虚拟内存、网络及文件系统）等的实现。
        - [CS 61B：伸展树（Splay trees）（视频）](https://www.youtube.com/watch?v=Najzh1rYQTo&index=23&list=PL-XXv-cvA_iAlnI-BQr9hjqADPBtujFJd)
        - MIT 教程：伸展树（Splay trees）：
            - 该教程会过于学术，但请观看到最后的10分钟以确保掌握。
            - [视频](https://www.youtube.com/watch?v=QnPl_Y6EqMo)

    - **红黑树**
        - 这些是2-3棵树的翻译（请参见下文）。
        - 实际中：红黑树提供了在最坏情况下插入操作、删除操作和查找操作的时间保证。这些时间值的保障不仅对时间敏感型应用有用，例如实时应用，还对在其他数据结构中块的构建非常有用，而这些数据结构都提供了最坏情况下的保障；例如，许多用于计算几何学的数据结构都可以基于红黑树，而目前 Linux 内核所采用的完全公平调度器（the Completely Fair Scheduler）也使用到了该种树。在 Java 8中，Collection HashMap也从原本用Linked List实现，储存特定元素的哈希码，改为用红黑树实现。
        - [Aduni —— 算法 —— 课程4（该链接直接跳到开始部分）（视频）](https://youtu.be/1W3x0f_RmUo?list=PLFDnELG9dpVxQCxuD-9BSy2E7BWY3t5Sm&t=3871)
        - [Aduni —— 算法 —— 课程5（视频）](https://www.youtube.com/watch?v=hm2GHwyKF1o&list=PLFDnELG9dpVxQCxuD-9BSy2E7BWY3t5Sm&index=5)
        - [黑树（Black Tree）](https://en.wikipedia.org/wiki/Red%E2%80%93black_tree)
        - [二分查找及红黑树的介绍](https://www.topcoder.com/community/data-science/data-science-tutorials/an-introduction-to-binary-search-and-red-black-trees/)

    - **2-3查找树**
        - 实际中：2-3树的元素插入非常快速，但却有着查询慢的代价（因为相比较 AVL 树来说，其高度更高）。
        - 你会很少用到2-3树。这是因为，其实现过程中涉及到不同类型的节点。因此，人们更多地会选择红黑树。
        - [2-3树的直感与定义（视频）](https://www.youtube.com/watch?v=C3SsdUqasD4&list=PLA5Lqm4uh9Bbq-E0ZnqTIa8LRaL77ica6&index=2)
        - [2-3树的二元观点](https://www.youtube.com/watch?v=iYvBtGKsqSg&index=3&list=PLA5Lqm4uh9Bbq-E0ZnqTIa8LRaL77ica6)
        - [2-3树（学生叙述）（视频）](https://www.youtube.com/watch?v=TOb1tuEZ2X4&index=5&list=PLUl4u3cNGP6317WaSNfmCvGym2ucw3oGp)

    - **2-3-4树 (亦称2-4树)**
        - 实际中：对于每一棵2-4树，都有着对应的红黑树来存储同样顺序的数据元素。在2-4树上进行插入及删除操作等同于在红黑树上进行颜色翻转及轮换。这使得2-4树成为一种用于掌握红黑树背后逻辑的重要工具。这就是为什么许多算法引导文章都会在介绍红黑树之前，先介绍2-4树，尽管**2-4树在实际中并不经常使用**。
        - [CS 61B Lecture 26：平衡查找树（视频）](https://www.youtube.com/watch?v=zqrqYXkth6Q&index=26&list=PL4BBB74C7D2A1049C)
        - [自底向上的2-4树（视频）](https://www.youtube.com/watch?v=DQdMYevEyE4&index=4&list=PLA5Lqm4uh9Bbq-E0ZnqTIa8LRaL77ica6)
        - [自顶向下的2-4树（视频）](https://www.youtube.com/watch?v=2679VQ26Fp4&list=PLA5Lqm4uh9Bbq-E0ZnqTIa8LRaL77ica6&index=5)

    - **N 叉树（K 叉树、M 叉树）**
        - 注意：N 或 K 指的是分支系数（即树的最大分支数）：
        - 二叉树是一种分支系数为2的树
        - 2-3树是一种分支系数为3的树
        - [K 叉树](https://en.wikipedia.org/wiki/K-ary_tree)

    - **B 树**
        - 有趣的是：为啥叫 B 仍然是一个神秘。因为 B 可代表波音（Boeing）、平衡（Balanced）或 Bayer（联合创造者）
        - 实际中：B 树会被广泛适用于数据库中，而现代大多数的文件系统都会使用到这种树（或变种)。除了运用在数据库中，B 树也会被用于文件系统以快速访问一个文件的任意块。但存在着一个基本的问题，那就是如何将文件块 i 转换成一个硬盘块（或一个柱面-磁头-扇区）上的地址。
        - [B 树](https://en.wikipedia.org/wiki/B-tree)
        - [B 树数据结构](http://btechsmartclass.com/data_structures/b-trees.html)
        - [B 树的介绍（视频）](https://www.youtube.com/watch?v=I22wEC1tTGo&list=PLA5Lqm4uh9Bbq-E0ZnqTIa8LRaL77ica6&index=6)
        - [B 树的定义及其插入操作（视频）](https://www.youtube.com/watch?v=s3bCdZGrgpA&index=7&list=PLA5Lqm4uh9Bbq-E0ZnqTIa8LRaL77ica6)
        - [B 树的删除操作（视频）](https://www.youtube.com/watch?v=svfnVhJOfMc&index=8&list=PLA5Lqm4uh9Bbq-E0ZnqTIa8LRaL77ica6)
        - [MIT 6.851 —— 内存层次模块（Memory Hierarchy Models）（视频）](https://www.youtube.com/watch?v=V3omVLzI0WE&index=7&list=PLUl4u3cNGP61hsJNdULdudlRL493b-XZf)
            - 覆盖有高速缓存参数无关型（cache-oblivious）B 树和非常有趣的数据结构
            - 头37分钟讲述的很专业，或许可以跳过（B 指块的大小、即缓存行的大小）

- ### k-D树
    - 非常适合在矩形或更高维度的对象中查找点数
    - 最适合k近邻
    - [Kd树（视频）](https://www.youtube.com/watch?v=W94M9D_yXKk)
    - [kNN K-d树算法（视频）](https://www.youtube.com/watch?v=Y4ZgLlDfKDg)

- ### 跳表
    - "有一种非常迷幻的数据类型" - Skiena
    - [随机化: 跳表 (视频)](https://www.youtube.com/watch?v=2g9OSRKJuzM&index=10&list=PLUl4u3cNGP6317WaSNfmCvGym2ucw3oGp)
    - [更生动详细的解释](https://en.wikipedia.org/wiki/Skip_list)

- ### 网络流
    - [5分钟简析 Ford-Fulkerson──一步步示例 (视频)](https://www.youtube.com/watch?v=v1VgJmkEJW0)
    - [Ford-Fulkerson 算法 (视频)](https://www.youtube.com/watch?v=v1VgJmkEJW0)
    - [网络流 (视频)](https://www.youtube.com/watch?v=2vhN4Ice5jI)

- ### 不相交集 & 联合查找
    - [UCB 61B - 不相交集；排序 & 选择(视频)](https://www.youtube.com/watch?v=MAEGXTwmUsI&list=PL-XXv-cvA_iAlnI-BQr9hjqADPBtujFJd&index=21)
    - [Sedgewick算法──Union-Find（6视频）](https://www.coursera.org/learn/algorithms-part1/home/week/1)

- ### 快速处理的数学
    - [整数运算, Karatsuba 乘法 (视频)](https://www.youtube.com/watch?v=eCaXlAaN2uE&index=11&list=PLUl4u3cNGP61Oq3tWYp6V_F-5jb5L2iHb)
    - [中国剩余定理 (在密码学中的使用) (视频)](https://www.youtube.com/watch?v=ru7mWZJlRQg)

- ### 树堆 (Treap)
    - 一个二叉搜索树和一个堆的组合
    - [树堆](https://en.wikipedia.org/wiki/Treap)
    - [数据结构：树堆的讲解（视频）](https://www.youtube.com/watch?v=6podLUYinH8)
    - [集合操作的应用(Applications in set operations)](https://www.cs.cmu.edu/~scandal/papers/treaps-spaa98.pdf)

- ### 线性规划（Linear Programming）（视频）
    - [线性规划](https://www.youtube.com/watch?v=M4K6HYLHREQ)
    - [寻找最小成本](https://www.youtube.com/watch?v=2ACJ9ewUC6U)
    - [寻找最大值](https://www.youtube.com/watch?v=8AA_81xI3ik)
    - [用 Python 解决线性方程式──单纯形算法](https://www.youtube.com/watch?v=44pAWI7v5Zk)

- ### 几何：凸包（Geometry, Convex hull）（视频）
    - [Graph Alg. IV: 几何算法介绍 - 第 9 课](https://youtu.be/XIAQRlNkJAw?list=PLFDnELG9dpVxQCxuD-9BSy2E7BWY3t5Sm&t=3164)
    - [Graham & Jarvis: 几何算法 - 第 10 课](https://www.youtube.com/watch?v=J5aJEcOr6Eo&index=10&list=PLFDnELG9dpVxQCxuD-9BSy2E7BWY3t5Sm)
    - [分而治之: 凸包, 中值查找](https://www.youtube.com/watch?v=EzeYI7p9MjU&list=PLUl4u3cNGP6317WaSNfmCvGym2ucw3oGp&index=2)

- ### 离散数学
    - 查看下面的视频

- ### 机器学习（Machine Learning）
    - 为什么学习机器学习？
        - [谷歌如何将自己改造成一家「机器学习优先」公司？](https://backchannel.com/how-google-is-remaking-itself-as-a-machine-learning-first-company-ada63defcb70)
        - [智能计算机系统的大规模深度学习 (视频)](https://www.youtube.com/watch?v=QSaZGT4-6EY)
        - [Peter Norvig：深度学习和理解与软件工程和验证的对比](https://www.youtube.com/watch?v=X769cyzBNVw)
    - [谷歌云机器学习工具（视频）](https://www.youtube.com/watch?v=Ja2hxBAwG_0)
    - [谷歌开发者机器学习清单 (Scikit Learn 和 Tensorflow) (视频)](https://www.youtube.com/playlist?list=PLOU2XLYxmsIIuiBfYad6rFYQU_jL2ryal)
    - [Tensorflow (视频)](https://www.youtube.com/watch?v=oZikw5k_2FM)
    - [Tensorflow 教程](https://www.tensorflow.org/versions/r0.11/tutorials/index.html)
    - [Python 实现神经网络实例教程（使用 Theano）](http://www.analyticsvidhya.com/blog/2016/04/neural-networks-python-theano/)
    - 课程:
        - [很棒的初级课程：机器学习](https://www.coursera.org/learn/machine-learning)
            - [视频教程](https://www.youtube.com/playlist?list=PLZ9qNFMHZ-A4rycgrgOYma6zxF4BZGGPW)
            - 看第 12-18 集复习线性代数（第 14 集和第 15 集是重复的）
        - [机器学习中的神经网络](https://www.coursera.org/learn/neural-networks)
        - [Google 深度学习微学位](https://www.udacity.com/course/deep-learning--ud730)
        - [Google/Kaggle 机器学习工程师微学位](https://www.udacity.com/course/machine-learning-engineer-nanodegree-by-google--nd009)
        - [无人驾驶工程师微学位](https://www.udacity.com/drive)
        - [Metis 在线课程 (两个月 99 美元)](http://www.thisismetis.com/explore-data-science)
    - 资源:
        - 书籍：
            - [Python 机器学习](https://www.amazon.com/Python-Machine-Learning-Sebastian-Raschka/dp/1783555130/)
            - [Data Science from Scratch: First Principles with Python](https://www.amazon.com/Data-Science-Scratch-Principles-Python/dp/149190142X)
            - [Python 机器学习简介](https://www.amazon.com/Introduction-Machine-Learning-Python-Scientists/dp/1449369413/)
        - [软件工程师的机器学习](https://github.com/ZuzooVn/machine-learning-for-software-engineers)
        - Data School：http://www.dataschool.io/

## 面试过程 & 通用的面试准备

- [ ] [ABC：不要停止编程（Always Be Coding）](https://medium.com/always-be-coding/abc-always-be-coding-d5f8051afce2#.4heg8zvm4)
- [ ] [白板编程（Whiteboarding）](https://medium.com/@dpup/whiteboarding-4df873dbba2e#.hf6jn45g1)
- [ ] [揭秘技术招聘](https://www.youtube.com/watch?v=N233T0epWTs)
- [ ] 如何在科技四强企业中获得一份工作：
  - [ ] [“如何在科技四强企业中获得一份工作 —— Amazon、Facebook、Google 和 Microsoft”（视频）](https://www.youtube.com/watch?v=YJZCUhxNCv8)
- [ ] 解密开发类面试第一集：
  - [ ] [Gayle L McDowell —— 解密开发类面试（视频）](https://www.youtube.com/watch?v=rEJzOhC5ZtQ)
  - [ ] [解密开发类面试 —— 作者 Gayle Laakmann McDowell（视频）](https://www.youtube.com/watch?v=aClxtDcdpsQ)
- [ ] 解密 Facebook 编码面试：
  - [方法](https://www.youtube.com/watch?v=wCl9kvQGHPI)
  - [问题演练](https://www.youtube.com/watch?v=4UWDyJq8jZg)
- [ ] 准备课程：
  - [ ] [软件工程师面试发布（收费课程）](https://www.udemy.com/software-engineer-interview-unleashed)：
    - 从前 Google 面试官身上学习如何准备自己，让自己能够应付软件工程师的面试。
  - [ ] [Python 数据结构，算法和面试（收费课程）](https://www.udemy.com/python-for-data-structures-algorithms-and-interviews/)：
    - Python 面试准备课程，内容涉及数据结构，算法，模拟面试等。
  - [ ] [Python 的数据结构和算法简介（Udacity 免费课程）](https://www.udacity.com/course/data-structures-and-algorithms-in-python--ud513)：
    - 免费的 Python 数据结构和算法课程。
  - [ ] [数据结构和算法纳米学位！（Udacity 收费纳米学位）](https://www.udacity.com/course/data-structures-and-algorithms-nanodegree--nd256)：
    - 获得超过100种数据结构和算法练习以及指导的动手练习，专门导师帮助你在面试和职场中做好准备。
  - [ ] [探究行为面试（Educative 免费课程）](https://www.educative.io/courses/grokking-the-behavioral-interview)：
    - 很多时候，不是你的技术能力会阻碍你获得理想的工作，而是你在行为面试中的表现。