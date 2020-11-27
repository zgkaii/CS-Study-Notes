[java的几种对象(PO,VO,DAO,BO,POJO，DTO)解释](https://www.cnblogs.com/firstdream/archive/2012/04/13/2445582.html)

1、PO(persistant object)持久对象  
PO 就是对应数据库中某个表中的一条记录，多个记录可以用PO的集合。
PO 中不应该包含任何对数据库中的操作。

2、DO(Domain Object)领域对象  
就是从现实世界中抽象出来的有形或无形的业务实体。

3、TO(Transfer Object)数据传输对象  
不同的应用程序之间传输的对象

4、DTO(Data Transfer Object)数据传输对象  
这个概念来源于 J2EE 的设计模式，原来的目的是为了EJB的分布式应用提供粗粒度的数据实体，

以减少分布式调用的次数，从而提高分布式调用的性能和降低网络负载，但在这里，泛指用于展示层与服务层之间的数据传输对象。

5、VO(Value Object)值对象  
通常用于业务层之间的数据传递，和 PO 一样也是仅仅包含数据而已。

但应是抽象出的业务对象，可以和表对应，也可以不对应，这根据业务的需要。用new关键字创建，由 GC 回收。

6、BO(Business Object)业务对象  
从业务模型的角度看，UML 元件领域模型中的领域对象。封装业务逻辑的 Java 对象，通过调用 DAO 方法，结合 PO,VO 进行业务操作。

Business Object：业务对象   
主要作用是把业务逻辑封装为一个对象。这个对象可以包含一个或多个其它对象。

7、POJO(Plain Ordinary Java Object)简单无规则Java对象  
传统意义的 Java 对象。  

在一些 Object/Relation Mapping 工具中，能够做到维护数据库表记录的 Persistent Object 完全是一个符合 Java Bean 规范的纯Java对象，没有增加别的属性和方法。

8、DAO(Data Access Object)数据访问对象  
是一个 SUN 的标准J2EE设计模式，这个模式中有一个接口就是DAO，它负持久层的操作。  

为业务层提供接口，此对象用于访问数据库。通常和 PO 结合使用，DAO中包含了各种数据库的操作方法，通过它的方法，结合 PO 对数据库进行相关的操作。 
在业务逻辑与数据库资源中间，配合 VO，提供数据库的 CRUD 操作。   

[什么是JavaBean、bean? 什么是POJO、PO、DTO、VO、BO ? 什么是EJB、EntityBean？](https://blog.csdn.net/chenchunlin526/article/details/69939337)