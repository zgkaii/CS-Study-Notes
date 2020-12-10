## 写在最前 

[John Washam](https://startupnextdoor.com/author/john/) 记录了从Web开发者（自学、非计算机科学学位）蜕变至 Google软件工程师所制定的学习历程[coding-interview-university](https://github.com/jwasham/coding-interview-university)。

受其启发，我根据自身情况及诉求，也制定了学习计划。希望通过接下来一段时间的学习，打通任督二脉，对计算机科学有体系的理解，然后逐渐蜕变为一位优秀的软件工程师。

## 正式开始之前

**不可能把所有的东西都记住**

与John Washam文章[记住计算机科学知识](https://startupnextdoor.com/retaining-computer-science-knowledge/)描述的一样，我经常是看了数小时的视频，并记录大量的笔记。一段时间过后，基本上忘却大部分的内容。

推荐学习课程[Learning How to Learn: Powerful mental tools to help you master tough subjects](https://www.coursera.org/learn/learning-how-to-learn)。

**使用抽认卡**

为了解决善忘的问题，可以制作不同的抽认卡帮助学习。

John Washam的抽认卡：

- [抽认卡页面的代码仓库](https://github.com/jwasham/computer-science-flash-cards)
- [抽认卡数据库 ── 旧 1200 张](https://github.com/jwasham/computer-science-flash-cards/blob/master/cards-jwasham.db)
- [抽认卡数据库 ── 新 1800 张](https://github.com/jwasham/computer-science-flash-cards/blob/master/cards-jwasham-extreme.db)

另外推荐支持多平台的软件——[Anki](http://ankisrs.net/)，中国国内相似功能APP推荐——[记乎](http://www.geefoo.cn/)。

**复习、复习、再复习**

制作出各种抽认卡后，需要我们在空余的时候去复习。编程累了就休息半个小时，并去复习抽认卡，如此反复。

**专注**

在学习的过程中，往往会有许多令人分心的事占据着我们宝贵的时间。因此，专注和集中注意力是非常困难的。放点纯音乐能帮上一些忙。

## 项目结构

```java
Cs-Notes-Kz
├── 计算机理论知识
|    ├── 计算机网络
|    ├── 操作系统
|    ├── 信息安全
|    ├── 计算机组成原理
|    ├── 数据结构与算法
|    |	  ├── 数据结构
|    |	  ├── 算法
|    |	  └── LeetCode
|    └── 数据库原理
├── 开发工具
├── Java
|    ├── JavaSE
|    ├── JVM
|    ├── 并发编程
|    ├── 设计模式
|    ├── Java新特性
|    ├── JavaEE
|    └── Spring全家桶
|         ├── Spring
|         ├── SpringMVC
|         └── SprinBoot
├── Python
|    ├── 基础
|    └── Django
├── Golang
├── C
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
├── 软件设计
|    ├── 编程范式
|    └── 设计原则
├── Linux
├── 数据库
|    ├── 关系型数据库
|    |    ├── Mysql
|    |    └── Oracle
|    └── NoSQL数据库
├── 分布式系统
|    ├── 分布式理论
|    ├── 分布式事务
|    ├── 分布式存储和数据库
|    ├── 分布式消息系统
|    ├── 分布式监控和跟踪
|    ├── 设计模式
|    ├── 弹性伸缩
|    ├── 一致性哈希
|    ├── 数据库分布式    
|    ├── 缓存
|    └── 消息队列    
├── 微服务
|    ├── 服务注册/发现
|    ├── 网关
|    ├── 服务调用（负载均衡）
|    ├── 熔断/降级
|    ├── 配置中心
|    ├── 认证和鉴权
|    ├── 分布式事务
|    ├── 任务调度
|    ├── 链路追踪与监控    
|    └── 日志分析与监控
├── 容器化/自动化运维
|    ├── 容器技术
|    ├── CND加速
|    ├── 代码质量检验
|    └── 日志分析/收集
├── 机器学习/人工智能
├── 大数据
|    ├── Hadoop
|    └── Spark
├── 源码分析
├── 面试
└── 其他
     ├── 错误记录
     └── 感悟
```

## Let‘s go!!!

## 程序员修养

### 英语能力

可以说，现代计算机技术几乎都来自于西方，英文世界是有价值的信息集散地。如果我们要成为一个高手，就必须到信息的源头去。学好英文是非常有必要的，不只是读写，还有听和说。

* 坚持 Google 英文关键词，而不是在 Google 里搜中文。

* 在GitHub上只用英文。代码注释、Commit信息、Issue和Pull Request......尽量都用英文书写。

* 坚持到 YouTube 上每天看 5 分钟的视频。

* 坚持使用英译词典，例如安装Chrome 插件 [Google Dictionary](https://chrome.google.com/webstore/detail/google-dictionary-by-goog/mgijmajocgfcbeboacabfgobmjgjcoja)。

* 坚持用英文的教材而不是中文的。

### 问问题的能力

提问的智慧（[How To Ask Questions The Smart Way](http://www.catb.org/~esr/faqs/smart-questions.html)）一文最早是由 Eric Steven Raymond 所撰写的，详细描述了发问者事前应该做好什么，而什么又是不该做的。作者认为这样能让问题容易令人理解，而且发问者自己也能学到较多东西。

另外，还有一个经典的问题叫 [X-Y Problem](http://xyproblem.info/)，我们需要小心避免（可以参考Coolshell上文章《[X-Y 问题](https://coolshell.cn/articles/10804.html)》）。

StackOverflow 上也有如何问问题的回答 -- “[FAQ for StackExchange Site](https://meta.stackexchange.com/questions/7931/faq-for-stack-exchange-sites)”。

### 写代码的修养

除了《代码大全》外，程序员还需要补充一些如何写好代码的知识，有以下几本书推荐。

- 《[重构：改善既有代码的设计](https://book.douban.com/subject/4262627/)》，Martin Fowler 的经典之作。这本书的意义不仅仅在于 " 改善既有代码的设计 "，也指导了我们如何从零开始构建代码的时候避免不良的代码风格。这是一本程序员**必读**的书。
- 《[修改代码的艺术](https://book.douban.com/subject/2248759/)》，这本书是继《重构》之后探讨修改代码技术的又一里程碑式的著作，而且从涵盖面和深度上都超过了前两部经典（《代码大全》和《重构》）。
- 《[代码整洁之道](https://book.douban.com/subject/4199741/)》，此书提出一种观念：代码质量与其整洁度成正比。干净的代码，既在质量上较为可靠，也为后期维护和升级奠定了良好基础。
- 《[程序员的职业素养](https://book.douban.com/subject/11614538/)》，这本书是编程大师 Bob 大叔 40 余年编程生涯的心得体会，讲解成为真正专业的程序员需要什么样的态度、原则，需要采取什么样的行动。作者以自己以及身边的同事走过的弯路、犯过的错误为例，意在为后来人引路，助其职业生涯迈上更高台阶。

另外，作为一个程序员，Code Review 是非常重要的程序员修养。 下面有几篇挺不错的Code Review 的文章，可供参考。

- [Code Review Best Practices](https://medium.com/@palantir/code-review-best-practices-19e02780015f)
- [How Google Does Code Review](https://dzone.com/articles/how-google-does-code-review)
- [LinkedIn’s Tips for Highly Effective Code Review](https://thenewstack.io/linkedin-code-review/)

Unit Test 也是程序员的一个很重要的修养。写 Unit Test 的框架一般来说都是从 JUnit 衍生出来的，比如 CppUnit 之类的。学习 JUnit 使用的最好方式就是到其官网上看 [JUnit User Guide](https://junit.org/junit5/docs/current/user-guide/)（[中文版](http://sjyuan.cc/junit5/user-guide-cn/)）。以下文章可供参考（也可以自行 Google）：

- [You Still Don’t Know How to Do Unit Testing](https://stackify.com/unit-testing-basics-best-practices/)
- [Unit Testing Best Practices: JUnit Reference Guide](https://dzone.com/articles/unit-testing-best-practices)
- [JUnit Best Practices](http://www.kyleblaney.com/junit-best-practices/)

### 安全防范

代码中没有最基本的安全漏洞问题，也是我们程序员必需要保证的重要大事，尤其是对外暴露 Web 服务的软件，其安全性就更为重要了。对于在 Web 上经常出现的安全问题，有必要介绍一下 [OWASP - Open Web Application Security Project](https://www.owasp.org/index.php/Main_Page)。

OWASP 是一个开源的、非盈利的全球性安全组织，致力于应用软件的安全研究，其被视为 Web 应用安全领域的权威参考。美国联邦贸易委员会（FTC）强烈建议所有企业需遵循 OWASP 十大 Web 弱点防护守则。对于[OWASP Top Ten](https://owasp.org/www-project-top-ten)项目是程序员非常需要关注的安全问题，现在其已经成了一种标准，这里是其中文版《[OWASP Top 10 2017 PDF 中文版](https://www.owasp.org/images/d/dc/OWASP_Top_10_2017_中文版v1.3.pdf)》。

下面是安全编程方面的一些 Guideline。

- [伯克立大学的 Secure Coding Practice Guidelines](https://security.berkeley.edu/secure-coding-practice-guidelines)。
- [卡内基梅隆大学的 SEI CERT Coding Standards](https://wiki.sei.cmu.edu/confluence/display/seccode/SEI+CERT+Coding+Standards)。

此外，有一篇和 HTTP 相关的安全文章也是每个程序员必需要读的——《[Hardening Your HTTP Security Headers](https://www.keycdn.com/blog/http-security-headers/)》。

最后想说的是 " 防御性编程 "，英文叫[Defensive Programming](https://en.wikipedia.org/wiki/Defensive_programming)，它是为了保证对程序的不可预见的使用，不会造成程序功能上的损坏。它可以被看作是为了减少或消除墨菲定律效力的想法。防御式编程主要用于可能被滥用，恶作剧或无意地造成灾难性影响的程序上。下面是一些文章。

- [The Art of Defensive Programming](https://medium.com/web-engineering-vox/the-art-of-defensive-programming-6789a9743ed4)。
- [Overly defensive programming](https://medium.com/@cvitullo/overly-defensive-programming-e7a1b3d234c2)。

### 软件工程和上线

系统上线是一件比较严肃的事，我们写的软件不是跑在自己的机器上的玩具，或是实验室里的实验品，而是交互给用户使用的，甚至是用户付费的软件。对于这样的软件或系统，我们需要遵守一些上线规范。比如，需要认真测试，并做上线前检查，以及上线后监控。下面是几个简单的规范，可供参考。

- 关于测试，推荐两本书。
  - 《[完美软件：对软件测试的各种幻想](https://book.douban.com/subject/4187479/)》，这本书重点讨论了与软件测试有关的各种心理问题及其表现与应对方法。
  - 《[Google 软件测试之道](https://book.douban.com/subject/25742200/)》，描述了测试解决方案，揭示了测试架构是如何设计、实现和运行的，介绍了软件测试工程师的角色；讲解了技术测试人员应该具有的技术技能；阐述了测试工程师在产品生命周期中的职责；讲述了测试管理，并对在 Google 的测试历史上或者主要产品上发挥了重要作用的工程师的访谈，这令那些试图建立类似 Google 的测试流程或团队的人受益很大。
- 当你的系统要上线时，这里有两个 Checklist 可为上线前检查做一些参考。
  - [Server Side checklist](https://github.com/mtdvio/going-to-production/blob/master/serverside-checklist.md)
  - [Single Page App Checklist](https://github.com/mtdvio/going-to-production/blob/master/spa-checklist.md)
- 《[Monitoring 101](https://www.datadoghq.com/blog/monitoring-101-collecting-data/)》这是一篇运维方面的入门文章，告诉最基本的监控线上运行软件的方法和实践。
