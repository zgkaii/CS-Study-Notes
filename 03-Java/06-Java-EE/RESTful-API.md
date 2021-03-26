- [1 相关概念](#1-相关概念)
- [2 REST 接口规范](#2-rest-接口规范)
  - [2.1 动作](#21-动作)
  - [2.2 路径（接口命名）](#22-路径接口命名)
  - [2.3 过滤信息（Filtering）](#23-过滤信息filtering)
  - [2.4 状态码（Status Codes）](#24-状态码status-codes)
- [3 HATEOAS](#3-hateoas)
- [文章推荐](#文章推荐)
# 1 相关概念

REST，即 **Representational State Transfer** 的缩写，意为"表现层状态转化"，它是由Roy Fielding于2000年在[论文](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)中首次提出。实际上REST的全称是 **Resource Representational State Transfe** ，也就是 **“资源”在网络传输中以某种“表现形式”进行“状态转移”** 。这样的解释理解起来仍然比较晦涩，这三个关键词的含义到底是指什么呢？

- **资源（Resource）** ：任何真实的对象数据都可称为资源，例如文档，图像等。一个资源既可以是一个集合，也可以是单个个体。比如班级 classes 是代表一个集合形式的资源，而特定的 class 代表单个个体资源。每一种资源都有特定的 URL（统一资源定位符）与之对应，如果我们需要获取这个资源，访问这个 URL就可以了，比如获取特定的班级：`/class/6`。另外，资源也可以包含子资源，比如 `/classes/classId/teachers`：列出某个指定班级的所有老师的信息。
- **表现形式（Representational）**："资源"是一种信息实体，它可以有多种外在表现形式。我们把"资源"具体呈现出来的形式比如 json，xml，image,txt 等等叫做它的"表现层/表现形式"。
- **状态转移（State Transfer）** ：REST 中的状态转移更多地描述的服务器端资源的状态，比如你通过增删改查（通过 HTTP 动词实现）引起资源状态的改变。ps:互联网通信协议 HTTP 协议，是一个无状态协议，所有的资源状态都保存在服务器端。

综上，总结一下什么是 RESTful 架构：

1. 每一个 URL 代表一种资源；
2. 客户端和服务器之间，传递这种资源的某种表现形式比如 json，xml，image,txt 等等；
3. 客户端通过四个HTTP动词（get、post、put、delete），对服务器端资源进行操作，实现”表现层状态转化”。

# 2 REST 接口规范

## 2.1 动作

- GET ：请求从服务器获取特定资源。举个例子：`GET /classes`（获取所有班级）
- POST ：在服务器上创建一个新的资源。举个例子：`POST /classes`（创建班级）
- PUT ：更新服务器上的资源（客户端提供更新后的整个资源）。举个例子：`PUT /classes/12`（更新编号为 12 的班级）
- DELETE ：从服务器删除特定的资源。举个例子：`DELETE /classes/12`（删除编号为 12 的班级）
- PATCH ：更新服务器上的资源（客户端提供更改的属性，可以看做作是部分更新），使用的比较少，这里就不举例子了。

## 2.2 路径（接口命名）

路径又称"终点"（endpoint），表示 API 的具体网址。实际开发中常见的规范如下：

1. **网址中不能有动词，只能有名词，API 中的名词也应该使用复数。** 因为 REST 中的资源往往和数据库中的表对应，而数据库中的表都是同种记录的"集合"（collection）。**如果 API 调用并不涉及资源（如计算，翻译等操作）的话，可以用动词。** 比如：`GET /calculate?param1=11&param2=33`
2. 不用大写字母，建议用中杠 - 不用下杠 \_ 比如邀请码写成 `invitation-code`而不是 ~~invitation_code~~

Talk is cheap！来举个实际的例子来说明一下吧！现在有这样一个 API 提供班级（class）的信息，还包括班级中的学生和教师的信息，则它的路径应该设计成下面这样。

**接口尽量使用名词，禁止使用动词。** 下面是一些例子：

```
GET    /classes：列出所有班级
POST   /classes：新建一个班级
GET    /classes/classId：获取某个指定班级的信息
PUT    /classes/classId：更新某个指定班级的信息（一般倾向整体更新）
PATCH  /classes/classId：更新某个指定班级的信息（一般倾向部分更新）
DELETE /classes/classId：删除某个班级
GET    /classes/classId/teachers：列出某个指定班级的所有老师的信息
GET    /classes/classId/students：列出某个指定班级的所有学生的信息
DELETE classes/classId/teachers/ID：删除某个指定班级下的指定的老师的信息
```

反例：

```
/getAllclasses
/createNewclass
/deleteAllActiveclasses
```

理清资源的层次结构，比如业务针对的范围是学校，那么学校会是一级资源:`/schools`，老师: `/schools/teachers`，学生: `/schools/students` 就是二级资源。

## 2.3 过滤信息（Filtering）

如果我们在查询的时候需要添加特定条件的话，建议使用 url 参数的形式。比如我们要查询 state 状态为 active 并且 name 为 guidegege 的班级：

```
GET    /classes?state=active&name=kai
```

比如我们要实现分页查询：

```
GET    /classes?page=1&size=10 //指定第1页，每页10个数据
```

## 2.4 状态码（Status Codes）

**状态码范围：**

| 2xx：成功 | 3xx：重定向    | 4xx：客户端错误  | 5xx：服务器错误 |
| --------- | -------------- | ---------------- | --------------- |
| 200 成功  | 301 永久重定向 | 400 错误请求     | 500 服务器错误  |
| 201 创建  | 304 资源未修改 | 401 未授权       | 502 网关错误    |
|           |                | 403 禁止访问     | 504 网关超时    |
|           |                | 404 未找到       |                 |
|           |                | 405 请求方法不对 |                 |


# 3 HATEOAS

> **RESTful 的极致是 hateoas ，但是这个基本不会在实际项目中用到。**

上面是 RESTful API 最基本的东西，也是我们平时开发过程中最容易实践到的。实际上，RESTful API 最好做到 Hypermedia，即返回结果中提供链接，连向其他 API 方法，使得用户不查文档，也知道下一步应该做什么。

比如，当用户向 api.example.com 的根目录发出请求，会得到这样一个文档。

```javascript
{"link": {
  "rel":   "collection https://www.example.com/classes",
  "href":  "https://api.example.com/classes",
  "title": "List of classes",
  "type":  "application/vnd.yourformat+json"
}}
```

上面代码表示，文档中有一个 link 属性，用户读取这个属性就知道下一步该调用什么 API 了。rel 表示这个 API 与当前网址的关系（collection 关系，并给出该 collection 的网址），href 表示 API 的路径，title 表示 API 的标题，type 表示返回类型 Hypermedia API 的设计被称为[HATEOAS](http://en.wikipedia.org/wiki/HATEOAS)。

在 Spring 中有一个叫做 HATEOAS 的 API 库，通过它我们可以更轻松的创建除符合 HATEOAS 设计的 API。

# 文章推荐

**RESTful API 介绍：**

- [RESTful API Tutorial](https://RESTfulapi.net/)
- [RESTful API 最佳指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)
- [[译] RESTful API 设计最佳实践](https://juejin.im/entry/59e460c951882542f578f2f0)
- [那些年，我们一起误解过的 REST](https://segmentfault.com/a/1190000016313947)
- [Testing RESTful Services in Java: Best Practices](https://phauer.com/2016/testing-RESTful-services-java-best-practices/)
- [一文搞懂 RESTful API](https://xie.infoq.cn/article/ea83211ad22d0efe7299c3f11)

**Spring 中使用 HATEOAS：**

- [在 Spring Boot 中使用 HATEOAS](a)
- [Building REST services with Spring](https://spring.io/guides/tutorials/classmarks/) 
- [An Intro to Spring HATEOAS](https://www.baeldung.com/spring-hateoas-tutorial) 
- [spring-hateoas-examples](https://github.com/spring-projects/spring-hateoas-examples/tree/master/hypermedia)
- [Spring HATEOAS](https://spring.io/projects/spring-hateoas#learn) 