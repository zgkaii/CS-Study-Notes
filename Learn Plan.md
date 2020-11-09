## 写在最前 

[John Washam](https://startupnextdoor.com/author/john/) 记录了从Web开发者（自学、非计算机科学学位）蜕变至 Google软件工程师所制定的学习历程[coding-interview-university](https://github.com/jwasham/coding-interview-university)。

![白板上编程 ———— 来自 HBO 频道的剧集，“硅谷”](https://d3j2pkmjtin6ou.cloudfront.net/coding-at-the-whiteboard-silicon-valley.png)

受其启发，我根据自身情况及诉求，也制定了学习计划。希望通过接下来一段时间的学习，打通任督二脉，对计算机科学有体系的理解，能够获取BAT独角兽公司的Offer，然后逐渐蜕变为一位优秀的软件工程师。

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

## 项目结构

整理的学习笔记都以Markdown文档格式放在到不同的文件夹下，结构如下：

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
