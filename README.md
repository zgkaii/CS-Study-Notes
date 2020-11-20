## 写在最前 

[John Washam](https://startupnextdoor.com/author/john/) 记录了从Web开发者（自学、非计算机科学学位）蜕变至 Google软件工程师所制定的学习历程[coding-interview-university](https://github.com/jwasham/coding-interview-university)。

![白板上编程 ———— 来自 HBO 频道的剧集，“硅谷”](https://d3j2pkmjtin6ou.cloudfront.net/coding-at-the-whiteboard-silicon-valley.png)

受其启发，我根据自身情况及诉求，也制定了学习计划。希望通过接下来一段时间的学习，打通任督二脉，对计算机科学有体系的理解，然后逐渐蜕变为一位优秀的软件工程师。

**正式开始之前**

**1. 不可能把所有的东西都记住**

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

## 项目结构

我的学习笔记都以Markdown文档格式放在到不同的文件夹下，结构如下：

```java
学而时习之
├── 基础学习
|    ├── 计算机网络
|    ├── 操作系统
|    ├── 信息安全
|    ├── 计算机组成原理
|    ├── 数据结构与算法
|    |	  ├── 数据结构
|    |	  ├── 算法
|    |	  └── LeetCode
|    └── 数据库
|     	  ├── 数据库原理
|     	  ├── Mysql
|     	  └── Oracle
├── 开发工具
├── JavaSE
|    ├── Java基础
|    ├── JVM
|    ├── 并发编程
|    ├── 设计模式
|    ├── Java新特性
|    └── 源码学习
├── JavaEE
|	 └── JDBC
├── 前端
|    ├── 前端基础
|    |    ├── HTML
|    |    ├── CSS
|    |    └── JAVAScript
|    ├── 模板引擎
|    |    ├── Thymeleaf
|    |    └── FreeMarker
|    └── 组合框架
|    	  ├── Vue
|    	  └── React
├── Spring全家桶
|    ├── Spring
| 	 ├── SpringMVC
| 	 └── SprinBoot
├── 进阶学习
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
|    ├── 服务注册/发现
|    ├── 网关
|    ├── 服务调用（负载均衡）
|    ├── 熔断/降级
|    ├── 配置中心
|    ├── 认证和鉴权
|    ├── 分布式事务
|    ├── 任务调度
|    ├── 链路追踪与监控    
|    ├── 日志分析与监控
|    └── 虚拟化/容器化
|    	  ├── 容器技术
|    	  └── 容器编排技术
├── 运维
|    ├── CND加速
|    ├── 代码质量检验
|    └── 日志分析/收集
├── 大数据
|    ├── Hadoop
|    └── Spark
├── Python学习
├── 面试
└── 其他
     └── 错误记录
```

## Let‘s go!!!

------

## 计算机基础

* [计算机网络](https://www.bilibili.com/video/BV19E411D78Q)
  - [x] 物理层
  - [x] 数据链路层
  - [x] 网络层
  - [x] 传输层
  - [x] 应用层
  
* [TCP/IP详解 卷1：协议](http://www.52im.net/topic-tcpipvol1.html)
* [计算机组成原理](https://www.bilibili.com/video/BV1BE411D7ii)
* [操作系统](https://www.bilibili.com/video/BV1YE411D7nH)
* [信息安全——复旦大学（MOOC 优质课程）](https://www.bilibili.com/video/BV1jJ411X7nN)

相关书籍推荐：

* [计算机网络（谢希仁）](https://book.douban.com/subject/2970300/)
* [TCP/IP详解 卷1：协议](https://book.douban.com/subject/1088054/)
* [编码的奥秘（Charles Petzold）](https://book.douban.com/subject/1024570/)
* [深入理解计算机系统（Randal E.Bryant ）](https://book.douban.com/subject/5333562/)
* [信息安全原理与实践](https://book.douban.com/subject/24733262/)

## 数据结构与算法

[数据结构脑图](https://www.processon.com/view/5d108d6ce4b0955b9368c1df#map)

[算法脑图](http://naotu.baidu.com/file/0a53d3a5343bd86375f348b2831d3610?token=5ab1de1c90d5f3ec)

```java
--- 待深入学习---
```



### LeetCode

**五遍刷题法**

* 第1遍
  * 5分种：读题+思考
  * 无思路直接看解法，多解法比较优劣。
* 第2遍
  * 自己写，解法优化
* 第3遍
  * 24小时后再重复做题
* 第4遍
  * 一周后再重复做题
* 第5遍
  * 面试前恢复性训练

## 数据库

```java
--- 待深入学习---
```

## JavaSE

- [x] 基本特性、常用API
- [x] 面向对象、抽象类与接口
- [x] 集合框架
- [x] 异常
- [x] 多线程
- [x] 字节流、字符流
- [x] 缓冲流、转换流、序列流、打印流
- [x] 网络编程
- [x] Lambda表达式
- [x] Stream流

## JVM

- [x] 类加载机制
- [x] JVM内存分布
  - [x] 程序计数器
  - [x] 虚拟机栈
  - [x] 本地方法栈
  - [x] 堆
  - [x] 方法区

- [x] 执行引擎
- [x] 垃圾回收器
- [ ] 字节码文件
- [ ] GC性能调优

## 并发编程
- [x] 线程常用方法
- [x] 线程状态
- [x] synchronized原理（monitor、偏向锁、轻量级锁、重量级锁）
- [x] park与unpark、notify与notifyAll
- [x] volatile
- [x] 重入锁ReentrantLock与ReadWriteLock
- [x] AQS及并发工具类(Semaphore、CountDownLatch 、CyclicBarrier)
- [x] Atomic原子类
- [x] 死锁
- [ ] ConcurrentHashMap
- [ ] ConcurrentLinkedQueue
- [ ] 阻塞队列
- [ ] Fork&Join框架
- [ ] 线程池
- [ ] Executor框架
- [ ] Callable-Future-FutureTask
- [ ] ThreadLocal
- [ ] Condition接口

## 设计模式

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

> 待学习

## JDK新特性

* JDK8新特性
  - [x] Lamdba表达式

> 待深究

## 源码浅析

- [x] ArrayList源码
- [ ] Vector
- [x] LinkedList源码
- [x] HashMap

- [ ] Hashtable
- [ ] HashSet
- [ ] TreeMap

> 待更新

## JavaWeb

> 待更新

## Spring全家桶

* Spring
  - [x] IoC、DI
  - [x] AOP
  - [x] 事务管理
* SpringBoot
  - [x] 日志框架
  - [x] 配置文件
  - [x] Web开发
* SpringMVC
  - [x] 动态绑定
  - [x] 响应数据与结果视图、文件上传
  - [x] 异常处理、拦截器

> 待深究

## 前端

* 前端基础
  - [x] [HTML](https://www.runoob.com/html/html-tutorial.html)
  - [x] [CSS布局学习指南](https://juejin.im/post/6844903634849759239)
  - [ ] [JavaScript 教程](https://wangdoc.com/javascript/)
  - [ ] [ES6 教程](https://wangdoc.com/es6/)
* 模板引擎
  - [ ] Thymeleaf
* 前端框架
  - [x] React
    * React基础使用
  - [x] Vue
    * Vue基础使用



> 待更新... ...

---

## 任务清单

- [ ] React学习
- [ ] Response/Request...更新 
- [ ] [Swagger UI]([http://docs.swagger.io/swagger-core/current/apidocs/index.html?io/swagger/annotations](http://docs.swagger.io/swagger-core/current/apidocs/index.html?io/swagger/annotations) )
- [ ] PostgreSQL 
- [ ] [REST](https://zhuanlan.zhihu.com/p/90367875)

- [ ] DTO——[java的几种对象(PO,VO,DAO,BO,POJO，DTO)解释](https://www.cnblogs.com/firstdream/archive/2012/04/13/2445582.html)





















