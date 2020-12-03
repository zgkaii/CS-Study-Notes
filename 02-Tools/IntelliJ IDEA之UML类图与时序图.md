# 1 UML概述

## 1.1 简述

UML——Unified modeling language UML (**统一建模语言**)，是一种用于软件系统分析和设计的语言工具，它用于帮助软件开发人员进行思考和记录思路的结果。

UML 本身是一套符号的规定，就像数学符号和化学符号一样，这些符号用于描述软件模型中的各个元素和他们之间的关系，比如类、接口、实现、泛化、依赖、组合、聚合等。在IDEA中，选中Class文件，右键单击`Diagrams`，之后再点击`Show Diagrams`，或者快捷键`Ctrl+Alt+Shift+U`，则会生产类图：

![](https://img-blog.csdnimg.cn/2020120322233277.png)

工具栏（Toolbar）：

![](https://img-blog.csdnimg.cn/20201203223034966.png)

IDEA中自己绘制UML则需安装插件`PlantUML`：

![](https://img-blog.csdnimg.cn/20201203224135306.png)

安装`PlantUML`插件后还需要[安装Graphviz](https://segmentfault.com/a/1190000022789556)才能正常显示，其详细使用可参考——[PlantUML语言参考指引](http://plantuml.com/zh/guide)。`PlantUML`功能强大，但如果嫌弃在IDEA上使用 `PlantUML`麻烦的话，也可以用在线作图工具，如：`ProcessOn`，`WebChart`。

## 1.2 UML 图分类

画 UML 图与写文章差不多，都是把自己的思想描述给别人看，关键在于思路和条理，UML 图大致分类：

1. 用例图（use case）

2. 静态结构图：**类图**、对象图、包图、组件图、部署图

3. 动态行为图：交互图（**时序图**与协作图）、状态图、活动图

其中，类图（Class Diagram）是描述类与类之间的关系的，是UML图中最核心的。时序图（Sequence Diagram）是显示对象之间交互的图，设计模式解析中也经常会用到时序图。

# 2 UML 类图

类图用于描述系统中的类本身的组成和类之间的各种静态关系。PlantUML用下面的符号来表示类之间的关系：

* 依赖，`Dependency`：`<..`

- 泛化，`Generalization`：`<|--`
- 实现，`Realization`：`<|..`
- 关联，`Association`：`<--`
- 聚合，`Aggregation`：`o--`
- 组合，`Composition`：`*--`

以上是常见的六种关系，`--`可以替换成`..`就可以得到虚线。另外，其中的符号是可以改变方向的，例如：`<|--`表示右边的类泛化左边的类；`--|>`表示左边的类泛化右边的类。

例如：

```uml
@startuml
Class01 <.. Class02:依赖
class02 <|-- Class03:泛化
Class04 <|.. Class05:实现
Class06 <-- Class07:关联
Class08 o-- Class09:聚合
Class10 *-- Class11:组合
@enduml
```

生成的类图如下：

![](https://img-blog.csdnimg.cn/20201203234323439.png)

## 2.1 依赖（Dependence）

依赖关系是一种使用关系，表示某个类依赖于另外一个类，通常表现为，某个类的方法的参数使用了另外一个类的对象。

在UML类图中，依赖关系用带箭头的虚线表示，箭头从使用类指向被依赖的类。

## 2.2 泛化（Generalization）

泛化关系其实就是父子类之间的继承关系，表示一般与特殊的关系，指定子类如何特殊化父类的特征和行为。

## 2.3 实现（Realization）

实现关系就是接口和实现类之间的关系。类实现了接口中的抽象方法。

## 2.4 关联（Association）

关联关系是对象之间的一种引用关系，表示一个类和另外一个类之间的联系，如老师和学生，丈夫和妻子等。

关联具有导航性，即有双向关系或单向关系。

## 2.5 聚合（Aggregation）

聚合关系是关联关系的一种，表示整体和部分之间的关系，如学校和老师，车子和轮胎。

聚合关系在类中是通过成员对象来体现的，成员是整体的一部分，成员也可以脱离整体而存在。如老师是学校的一部分，同时老师也是独立的个体，可以单独存在。

## 2.6 组合（Composition）

组合关系是整体和部分之间的关系，也是关联关系的一种，是一种比聚合关系还要强的关系。部分对象不能脱离整体对象而单独存在，如人的身体和大脑之间的关系，大脑不能脱离身体而单独存在。





六种关系中，从弱到强依次是：
依赖关系 < 关联关系 < 聚合关系 < 组合关系 < 实现关系 = 泛化关系

# 3 UML 时序图

# 参考

* [设计模式中类的关系](https://blog.csdn.net/zhengzhb/article/details/7187278)

* [PlantUML语言参考指引](http://plantuml.com/zh/guide)

* [终于明白六大类UML类图关系了](https://segmentfault.com/a/1190000021317534)

* [IntelliJ IDEA之UML类图](https://my.oschina.net/u/4303145/blog/4289830)