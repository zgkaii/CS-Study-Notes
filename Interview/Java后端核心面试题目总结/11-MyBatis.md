## 什么是MyBatis?

MyBatis 是一款优秀的持久层框架，一个半 ORM（对象关系映射）框架，它支持定制化 sql、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生类型、接口和 Java 的 POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

**ORM又是什么？**

ORM（Object Relational Mapping），对象关系映射，是一种为了解决关系型数据库数据与简单Java对象（POJO）的映射关系的技术。简单的说，ORM是通过使用描述对象和数据库之间映射的元数据，将程序中的对象自动持久化到关系型数据库中。

**为什么说MyBatis是半自动ORM映射工具？它与全自动的区别在哪里？**

Hibernate属于全自动ORM映射工具，使用Hibernate查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取，所以它是全自动的。

而MyBatis在查询关联对象或关联集合对象时，需要手动编写sql来完成，所以，称之为半自动ORM映射工具。

## **JDBC编程有哪些不足之处，MyBatis是如何解决这些问题的？**

1、数据库链接创建、释放频繁造成系统资源浪费从而影响系统性能，可使用数据库连接池可解决此问题。

* 解决：在`MyBatis-config.xml`中配置数据连接池，使用连接池管理数据库连接。

2、sql语句写在代码中造成代码不易维护，实际应用sql变化的可能较大，sql变动需要改变Java后台代码。

* 解决：将sql语句配置在XXXXmapper.xml文件中与java代码分离。

3、向sql语句传参数麻烦，因为sql语句的where条件不一定，可能多也可能少，占位符需要和参数一一对应。

* 解决：MyBatis 提供 `<where />`、`<if />` 等等动态语句所需要的标签，并支持 OGNL 表达式，简化了动态 sql 拼接的代码，提升了开发效率。

4、对结果集解析麻烦，sql变化导致解析代码变化，且解析前需要遍历，如果能将数据库记录封装成pojo对象解析比较方便。

* 解决：MyBatis自动将sql执行结果映射至Java对象。

## MyBatis使用的优缺点？

**MyBatis框架的优点：**

　　1. 与JDBC相比，减少了50%以上的代码量。

　　2. MyBatis是最简单的持久化框架，小巧并且简单易学。

　　3. MyBatis灵活，不会对应用程序或者数据库的现有设计强加任何影响，SQL写在XML里，从程序代码中彻底分离，降低耦合度，便于统一管理和优化，可重用。

　　4. 提供XML标签，支持编写动态SQL语句（XML中使用if, else）。

　　5. 提供映射标签，支持对象与数据库的ORM字段关系映射（在XML中配置映射关系，也可以使用注解）。

**MyBatis框架的缺点：**

　　1. SQL语句的编写工作量较大，尤其是字段多、关联表多时，对开发人员编写SQL语句的功底有一定要求。

　　2. SQL语句依赖于数据库，导致数据库移植性差，不能随意更换数据库。

**MyBatis框架适用场合：**

* MyBatis专注于SQL本身，是一个足够灵活的DAO层解决方案。
* 对性能的要求很高，或者需求变化较多的项目，如互联网项目，MyBatis将是不错的选择。

## MyBatis与Hibernate有哪些不同？

MyBatis和hibernate不同，它不完全是一个ORM框架，因为MyBatis需要程序员自己编写Sql语句。

MyBatis直接编写原生态sql，可以严格控制sql执行性能，灵活度高，非常适合对关系数据模型要求不高的软件开发，因为这类软件需求变化频繁，一但需求变化要求迅速输出成果。但是灵活的前提是MyBatis无法做到数据库无关性，如果需要实现支持多种数据库的软件，则需要自定义多套sql映射文件，工作量大。

Hibernate对象/关系映射能力强，数据库无关性好。如果用 Hibernate 开发可以节省很多代码，提高效率。但是 Hibernate 的缺点是学习门槛高，要精通门槛更高，而且怎么设计 O/R 映射，在性能和对象模型之间如何权衡，以及怎样用好 Hibernate 需要具有很强的经验和能力才行。

总之，按照用户的需求在有限的资源环境下只要能做出维护性、扩展性良好的软件架构都是好架构，所以框架只有适合才是最好。简单总结如下：

- Hibernate 属于全自动 ORM 映射工具，使用 Hibernate 查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取。
- MyBatis 属于半自动 ORM 映射工具，在查询关联对象或关联集合对象时，需要手动编写 sql 来完成。

另外，在 [《浅析 MyBatis 与 Hibernate 的区别与用途》](https://www.jianshu.com/p/96171e647885) 文章，也是写的非常不错的。

当然，实际上，MyBatis 也可以搭配自动生成代码的工具，提升开发效率，还可以使用 [MyBatis-Plus](http://mp.baomidou.com/) 框架，已经内置常用的 SQL 操作，也是非常不错的。

## MyBatis 编程步骤

1. 创建 sqlSessionFactory 对象。
2. 通过 sqlSessionFactory 获取 sqlSession 对象。
3. 通过 sqlSession 获得 Mapper 代理对象。
4. 通过 Mapper 代理对象，执行数据库操作。
5. 执行成功，则使用 sqlSession 提交事务。
6. 执行失败，则使用 sqlSession 回滚事务。
7. 最终，关闭会话。

## `#{}` 和 `${}` 的区别是什么？

`${}` 是 Properties 文件中的变量占位符，它可以用于 XML 标签属性值和 sql 内部，属于**字符串替换**。例如将 `${driver}` 会被静态替换为 `com.mysql.jdbc.Driver` ：

```java
<dataSource type="UNPOOLED">
    <property name="driver" value="${driver}"/>
    <property name="url" value="${url}"/>
    <property name="username" value="${username}"/>
</dataSource>
```

`${}` 也可以对传递进来的参数**原样拼接**在 sql 中。代码如下：

```java
<select id="getSubject3" parameterType="Integer" resultType="Subject">
    SELECT * FROM subject
    WHERE id = ${id}
</select>
```

- 实际场景下，不推荐这么做。因为，可能有 sql 注入的风险。

------

`#{}` 是 sql 的参数占位符，MyBatis 会将 sql 中的 `#{}` 替换为 `?` 号，在 sql 执行前会使用 PreparedStatement 的参数设置方法，按序给 sql 的 `?` 号占位符设置参数值，比如 `ps.setInt(0, parameterValue)` 。 所以，`#{}` 是**预编译处理**，可以有效防止 sql 注入，提高系统安全性。

------

另外，`#{}` 和 `${}` 的取值方式非常方便。例如：`#{item.name}` 的取值方式，为使用反射从参数对象中，获取 `item` 对象的 `name` 属性值，相当于 `param.getItem().getName()` 。

## 当实体类中的属性名和表中的字段名不一样 ，怎么办？

第一种， 通过在查询的 sql 语句中定义字段名的别名，让字段名的别名和实体类的属性名一致。代码如下：

```java
<select id="selectOrder" parameterType="Integer" resultType="Order"> 
    SELECT order_id AS id, order_no AS orderno, order_price AS price 
    FROM orders 
    WHERE order_id = #{id}
</select>
```

- 这里，还有几点建议：
  - 1、数据库的关键字，统一使用大写，例如：`SELECT`、`AS`、`FROM`、`WHERE` 。
  - 2、每 5 个查询字段换一行，保持整齐。
  - 3、`,` 的后面，和 `=` 的前后，需要有空格，更加清晰。
  - 4、`SELECT`、`FROM`、`WHERE` 等，单独一行，高端大气。

------

第二种，是第一种的特殊情况。大多数场景下，数据库字段名和实体类中的属性名差，主要是前者为**下划线风格**，后者为**驼峰风格**。在这种情况下，可以直接配置如下，实现自动的下划线转驼峰的功能。

```java
<setting name="logImpl" value="LOG4J"/>
    <setting name="mapUnderscoreToCamelCase" value="true" />
</settings>
```

也就说，约定大于配置。非常推荐！

------

第三种，通过 `<resultMap>` 来映射字段名和实体类属性名的一一对应的关系。代码如下：

```java
<resultMap type="me.gacl.domain.Order" id=”OrderResultMap”> 
    <!–- 用 id 属性来映射主键字段 -–> 
    <id property="id" column="order_id"> 
    <!–- 用 result 属性来映射非主键字段，property 为实体类属性名，column 为数据表中的属性 -–> 
    <result property="orderNo" column ="order_no" /> 
    <result property="price" column="order_price" /> 
</resultMap>

<select id="getOrder" parameterType="Integer" resultMap="OrderResultMap">
    SELECT * 
    FROM orders 
    WHERE order_id = #{id}
</select>
```

- 此处 `SELECT *` 仅仅作为示例只用，实际场景下，千万千万千万不要这么干。用多少字段，查询多少字段。
- 相比第一种，第三种的**重用性**会一些。

## XML 映射文件中，除了常见的 select | insert | update | delete标 签之外，还有哪些标签？

如下部分，可见 [《MyBatis 文档 —— Mapper XML 文件》](http://www.MyBatis.org/MyBatis-3/zh/sqlmap-xml.html) ：

- `<cache />`标签，给定命名空间的缓存配置。
- `<cache-ref />` 标签，其他命名空间缓存配置的引用。
- `<resultMap />` 标签，是最复杂也是最强大的元素，用来描述如何从数据库结果集中来加载对象。
- `<sql />`标签，可被其他语句引用的可重用语句块。
- `<include />` 标签，引用 `<sql />` 标签的语句。
- `<selectKey />` 标签，不支持自增的主键生成策略标签。

如下部分，可见 [《MyBatis 文档 —— 动态 sql》](http://www.MyBatis.org/MyBatis-3/zh/dynamic-sql.html) ：

- `<if />`
- `<choose />`、`<when />`、`<otherwise />`
- `<trim />`、`<where />`、`<set />`
- `<foreach />`
- `<bind />`

## MyBatis 动态 sql 是做什么的？都有哪些动态 sql ？能简述一下动态 sql 的执行原理吗？

- MyBatis 动态 sql ，可以让我们在 XML 映射文件内，以 XML 标签的形式编写动态 sql ，完成逻辑判断和动态拼接 sql 的功能。
- MyBatis 提供了 9 种动态 sql 标签：`<if />`、`<choose />`、`<when />`、`<otherwise />`、`<trim />`、`<where />`、`<set />`、`<foreach />`、`<bind />` 。
- 其执行原理为，使用 **OGNL** 的表达式，从 sql 参数对象中计算表达式的值，根据表达式的值动态拼接 sql ，以此来完成动态 sql 的功能。

如上的内容，更加详细的话，请看 [《MyBatis 文档 —— 动态 sql》](http://www.MyBatis.org/MyBatis-3/zh/dynamic-sql.html) 文档。

## Mapper 接口的工作原理是什么？Mapper 接口里的方法，参数不同时，方法能重载吗？

Mapper 接口，对应的关系如下：

- 接口的全限名，就是映射文件中的 `"namespace"` 的值。
- 接口的方法名，就是映射文件中 MappedStatement 的 `"id"` 值。
- 接口方法内的参数，就是传递给 sql 的参数。

Mapper 接口是没有实现类的，当调用接口方法时，接口全限名 + 方法名拼接字符串作为 key 值，可唯一定位一个对应的 MappedStatement 。举例：`com.MyBatis3.mappers.StudentDao.findStudentById` ，可以唯一找到 `"namespace"` 为 `com.MyBatis3.mappers.StudentDao` 下面 `"id"` 为 `findStudentById` 的 MappedStatement 。

总结来说，在 MyBatis 中，每一个 `<select />`、`<insert />`、`<update />`、`<delete />` 标签，都会被解析为一个 MappedStatement 对象。

另外，Mapper 接口的实现类，通过 MyBatis 使用 **JDK Proxy** 自动生成其代理对象 Proxy ，而代理对象 Proxy 会拦截接口方法，从而“调用”对应的 MappedStatement 方法，最终执行 sql ，返回执行结果。整体流程如下图：

<div align="center">  
<img src="http://static.iocoder.cn/images/MyBatis/2020_03_15/02.png" width="600px"/>
</div>

- 其中，sqlSession 在调用 Executor 之前，会获得对应的 MappedStatement 方法。例如：`DefaultsqlSession#select(String statement, Object parameter, RowBounds rowBounds, ResultHandler handler)` 方法，代码如下：

  ```java
  // DefaultsqlSession.java
  
  @Override
  public void select(String statement, Object parameter, RowBounds rowBounds, ResultHandler handler) {
      try {
          // 获得 MappedStatement 对象
          MappedStatement ms = configuration.getMappedStatement(statement);
          // 执行查询
          executor.query(ms, wrapCollection(parameter), rowBounds, handler);
      } catch (Exception e) {
          throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);
      } finally {
          ErrorContext.instance().reset();
      }
  }
  ```

  - 完整的流程，胖友可以慢慢撸下 MyBatis 的源码。

------

Mapper 接口里的方法，是不能重载的，因为是**全限名 + 方法名**的保存和寻找策略。😈 所以有时，想个 Mapper 接口里的方法名，还是蛮闹心的，嘿嘿。

## Mapper 接口绑定有几种实现方式，分别是怎么实现的?

接口绑定有三种实现方式：

第一种，通过 **XML Mapper** 里面写 sql 来绑定。在这种情况下，要指定 XML 映射文件里面的 `"namespace"` 必须为接口的全路径名。

第二种，通过**注解**绑定，就是在接口的方法上面加上 `@Select`、`@Update`、`@Insert`、`@Delete` 注解，里面包含 sql 语句来绑定。

第三种，是第二种的特例，也是通过**注解**绑定，在接口的方法上面加上 `@SelectProvider`、`@UpdateProvider`、`@InsertProvider`、`@DeleteProvider` 注解，通过 Java 代码，生成对应的动态 sql 。

------

实际场景下，最最最推荐的是**第一种**方式。因为，sql 通过注解写在 Java 代码中，会非常杂乱。而写在 XML 中，更加有整体性，并且可以更加方便的使用 OGNL 表达式。

## MyBatis 的 XML Mapper文件中，不同的 XML 映射文件，id 是否可以重复？

不同的 XML Mapper 文件，如果配置了 `"namespace"` ，那么 id 可以重复；如果没有配置 `"namespace"` ，那么 id 不能重复。毕竟`"namespace"` 不是必须的，只是最佳实践而已。

原因就是，`namespace + id` 是作为 `Map<String, MappedStatement>` 的 key 使用的。如果没有 `"namespace"`，就剩下 id ，那么 id 重复会导致数据互相覆盖。如果有了 `"namespace"`，自然 id 就可以重复，`"namespace"`不同，`namespace + id` 自然也就不同。

## 如何获取自动生成的(主)键值?

不同的数据库，获取自动生成的(主)键值的方式是不同的。

Mysql 有两种方式，但是**自增主键**，代码如下：

```java
// 方式一，使用 useGeneratedKeys + keyProperty 属性
<insert id="insert" parameterType="Person" useGeneratedKeys="true" keyProperty="id">
    INSERT INTO person(name, pswd)
    VALUE (#{name}, #{pswd})
</insert>
    
// 方式二，使用 `<selectKey />` 标签
<insert id="insert" parameterType="Person">
    <selectKey keyProperty="id" resultType="long" order="AFTER">
        SELECT LAST_INSERT_ID()
    </selectKey>
        
    INSERT INTO person(name, pswd)
    VALUE (#{name}, #{pswd})
</insert>
```

- 其中，**方式一**较为常用。

------

Oracle 有两种方式，**序列**和**触发器**。因为艿艿自己不了解 Oracle ，所以问了银行的朋友，他们是使用**序列**。而基于**序列**，根据 `<selectKey />` 执行的时机，也有两种方式，代码如下：

```java
// 这个是创建表的自增序列
CREATE SEQUENCE student_sequence
INCREMENT BY 1
NOMAXVALUE
NOCYCLE
CACHE 10;

// 方式一，使用 `<selectKey />` 标签 + BEFORE
<insert id="add" parameterType="Student">
　　<selectKey keyProperty="student_id" resultType="int" order="BEFORE">
      select student_sequence.nextval FROM dual
    </selectKey>
    
     INSERT INTO student(student_id, student_name, student_age)
     VALUES (#{student_id},#{student_name},#{student_age})
</insert>

// 方式二，使用 `<selectKey />` 标签 + AFTER
<insert id="save" parameterType="com.threeti.to.ZoneTO" >
    <selectKey resultType="java.lang.Long" keyProperty="id" order="AFTER" >
      SELECT SEQ_ZONE.CURRVAL AS id FROM dual
    </selectKey>
    
    INSERT INTO TBL_ZONE (ID, NAME ) 
    VALUES (SEQ_ZONE.NEXTVAL, #{name,jdbcType=VARCHAR})
</insert>
```

- 他们使用第一种方式，没有具体原因，可能就没什么讲究吧。嘿嘿。

------

当然，数据库还有 sqlServer、Postgresql、DB2、H2 等等，具体的方式，胖友自己 Google 下噢。

## 在 Mapper 中如何传递多个参数?

第一种，使用 Map 集合，装载多个参数进行传递。代码如下：

```java
// 调用方法
Map<String, Object> map = new HashMap();
map.put("start", start);
map.put("end", end);
return studentMapper.selectStudents(map);

// Mapper 接口
List<Student> selectStudents(Map<String, Object> map);

// Mapper XML 代码
<select id="selectStudents" parameterType="Map" resultType="Student">
    SELECT * 
    FROM students 
    LIMIT #{start}, #{end}
</select>
```

- 显然，这不是一种优雅的方式。

------

第二种，保持传递多个参数，使用 `@Param` 注解。代码如下：

```java
// 调用方法
return studentMapper.selectStudents(0, 10);

// Mapper 接口
List<Student> selectStudents(@Param("start") Integer start, @Param("end") Integer end);

// Mapper XML 代码
<select id="selectStudents" resultType="Student">
    SELECT * 
    FROM students 
    LIMIT #{start}, #{end}
</select>
```

- 推荐使用这种方式。

------

第三种，保持传递多个参数，不使用 `@Param` 注解。代码如下：

```java
// 调用方法
return studentMapper.selectStudents(0, 10);

// Mapper 接口
List<Student> selectStudents(Integer start, Integer end);

// Mapper XML 代码
<select id="selectStudents" resultType="Student">
    SELECT * 
    FROM students 
    LIMIT #{param1}, #{param2}
</select>
```

- 其中，按照参数在方法方法中的位置，从 1 开始，逐个为 `#{param1}`、`#{param2}`、`#{param3}` 不断向下。

## MyBatis 是否可以映射 Enum 枚举类？

MyBatis 可以映射枚举类，对应的实现类为 EnumTypeHandler 或 EnumOrdinalTypeHandler 。

- EnumTypeHandler ，基于 `Enum.name` 属性( String )。**默认**。
- EnumOrdinalTypeHandler ，基于 `Enum.ordinal` 属性( `int` )。可通过 `<setting name="defaultEnumTypeHandler" value="EnumOrdinalTypeHandler" />` 来设置。

当然，实际开发场景，我们很少使用 Enum 类型，更加的方式是，代码如下：

```java
public class Dog {

    public static final int STATUS_GOOD = 1;
    public static final int STATUS_BETTER = 2;
    public static final int STATUS_BEST = 3；
    
    private int status;
    
}
```

------

并且，不单可以映射枚举类，MyBatis 可以映射任何对象到表的一列上。映射方式为自定义一个 TypeHandler 类，实现 TypeHandler 的`#setParameter(...)` 和 `#getResult(...)` 接口方法。

TypeHandler 有两个作用：

- 一是，完成从 javaType 至 jdbcType 的转换。
- 二是，完成 jdbcType 至 javaType 的转换。

具体体现为 `#setParameter(...)` 和 `#getResult(..)` 两个方法，分别代表设置 sql 问号占位符参数和获取列查询结果。

## MyBatis 都有哪些 Executor 执行器？它们之间的区别是什么？

MyBatis 有四种 Executor 执行器，分别是 SimpleExecutor、ReuseExecutor、BatchExecutor、CachingExecutor 。

- SimpleExecutor ：每执行一次 update 或 select 操作，就创建一个 Statement 对象，用完立刻关闭 Statement 对象。
- ReuseExecutor ：执行 update 或 select 操作，以 sql 作为key 查找**缓存**的 Statement 对象，存在就使用，不存在就创建；用完后，不关闭 Statement 对象，而是放置于缓存 `Map<String, Statement>` 内，供下一次使用。简言之，就是重复使用 Statement 对象。
- BatchExecutor ：执行 update 操作（没有 select 操作，因为 JDBC 批处理不支持 select 操作），将所有 sql 都添加到批处理中（通过 addBatch 方法），等待统一执行（使用 executeBatch 方法）。它缓存了多个 Statement 对象，每个 Statement 对象都是调用 addBatch 方法完毕后，等待一次执行 executeBatch 批处理。**实际上，整个过程与 JDBC 批处理是相同**。
- CachingExecutor ：在上述的三个执行器之上，增加**二级缓存**的功能。

------

通过设置 `<setting name="defaultExecutorType" value="">` 的 `"value"` 属性，可传入 SIMPLE、REUSE、BATCH 三个值，分别使用 SimpleExecutor、ReuseExecutor、BatchExecutor 执行器。

通过设置 `<setting name="cacheEnabled" value=""` 的 `"value"` 属性为 `true` 时，创建 CachingExecutor 执行器。

## MyBatis 如何执行批量插入?

首先，在 Mapper XML 编写一个简单的 Insert 语句。代码如下：

```java
<insert id="insertUser" parameterType="String"> 
    INSERT INTO users(name) 
    VALUES (#{value}) 
</insert>
```

然后，然后在对应的 Mapper 接口中，编写映射的方法。代码如下：

```java
public interface UserMapper {
    
    void insertUser(@Param("name") String name);

}
```

最后，调用该 Mapper 接口方法。代码如下：

```java
private static SqlSessionFactory sqlSessionFactory;

@Test
public void testBatch() {
    // 创建要插入的用户的名字的数组
    List<String> names = new ArrayList<>();
    names.add("占小狼");
    names.add("朱小厮");
    names.add("徐妈");
    names.add("飞哥");

    // 获得执行器类型为 Batch 的 SqlSession 对象，并且 autoCommit = false ，禁止事务自动提交
    try (SqlSession session = sqlSessionFactory.openSession(ExecutorType.BATCH, false)) {
        // 获得 Mapper 对象
        UserMapper mapper = session.getMapper(UserMapper.class);
        // 循环插入
        for (String name : names) {
            mapper.insertUser(name);
        }
        // 提交批量操作
        session.commit();
    }
}
```

代码比较简单，胖友仔细看看。当然，还有另一种方式，代码如下：

```java
INSERT INTO [表名]([列名],[列名]) VALUES([列值],[列值])),([列值],[列值])),([列值],[列值]));
```

- 对于这种方式，需要保证单条 sql 不超过语句的最大限制 `max_allowed_packet` 大小，默认为 1 M 。

这两种方式的性能对比，可以看看 [《[实验\]MyBatis批量插入方式的比较》](https://www.jianshu.com/p/cce617be9f9e) 。

## 介绍 MyBatis 的一级缓存和二级缓存的概念和实现原理？

> 详细可参考 [《聊聊 MyBatis 缓存机制》](https://tech.meituan.com/2018/01/19/mybatis-cache.html) 一文。

一级缓存：基于 PerpetualCache 的 HashMap 本地缓存，其存储作用域为 Session，当 Session flush 或 close 之 后，该 Session 中的所有 Cache 就将清空，默认打开一级缓存；

二级缓存：与一级缓存其机制相同，默认也是采用 PerpetualCache，HashMap 存储，不同在于其存储作用域为 Mapper(Namespace)，并且可自定义存储源，如 Ehcache。默认不打开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现 Serializable 序列化接口(可用来保存对象的状态)，可在它的映射文件中配置 ；

对于缓存数据更新机制，当某一个作用域(一级缓存 Session / 二级缓存 Namespaces)的进行了 C/U/D 操作后，默认该作用域下所有 select 中的缓存将被 clear。

## MyBatis 是否支持延迟加载？如果支持，它的实现原理是什么？

MyBatis 仅支持 association 关联对象和 collection 关联集合对象的延迟加载。其中，association 指的就是**一对一**，collection 指的就是**一对多查询**。

在 MyBatis 配置文件中，可以配置 `<setting name="lazyLoadingEnabled" value="true" />` 来启用延迟加载的功能。默认情况下，延迟加载的功能是**关闭**的。

------

它的原理是，使用 CGLIB 或 Javassist( 默认 ) 创建目标对象的代理对象。当调用代理对象的延迟加载属性的 getting 方法时，进入拦截器方法。比如调用 `a.getB().getName()` 方法，进入拦截器的 `invoke(...)` 方法，发现 `a.getB()` 需要延迟加载时，那么就会单独发送事先保存好的查询关联 B 对象的 sql ，把 B 查询上来，然后调用`a.setB(b)` 方法，于是 `a` 对象 `b` 属性就有值了，接着完成`a.getB().getName()` 方法的调用。这就是延迟加载的基本原理。

当然了，不光是 MyBatis，几乎所有的包括 Hibernate 在内，支持延迟加载的原理都是一样的。

## MyBatis 能否执行一对一、一对多的关联查询吗？都有哪些实现方式，以及它们之间的区别。

> 艿艿：这道题有点难度。理解倒是好理解，主要那块源码的实现，艿艿看的有点懵逼。大体的意思是懂的，但是一些细节没扣完。

能，MyBatis 不仅可以执行一对一、一对多的关联查询，还可以执行多对一，多对多的关联查询。

> 艿艿：不过貌似，我自己实际开发中，还是比较喜欢自己去查询和拼接映射的数据。😈

- 多对一查询，其实就是一对一查询，只需要把 `selectOne(...)` 修改为 `selectList(...)` 即可。案例可见 [《MyBatis：多对一表关系详解》](https://blog.csdn.net/xzm_rainbow/article/details/15336959) 。
- 多对多查询，其实就是一对多查询，只需要把 `#selectOne(...)` 修改为 `selectList(...)` 即可。案例可见 [《【MyBatis学习10】高级映射之多对多查询》](https://blog.csdn.net/eson_15/article/details/51655188) 。

------

关联对象查询，有两种实现方式：

- 一种是单独发送一个 sql 去查询关联对象，赋给主对象，然后返回主对象。好处是多条 sql 分开，相对简单，坏处是发起的 sql 可能会比较多。
- 另一种是使用嵌套查询，嵌套查询的含义为使用 `join` 查询，一部分列是 A 对象的属性值，另外一部分列是关联对象 B 的属性值。好处是只发一个 sql 查询，就可以把主对象和其关联对象查出来，坏处是 sql 可能比较复杂。

那么问题来了，`join` 查询出来 100 条记录，如何确定主对象是 5 个，而不是 100 个呢？其去重复的原理是 `<resultMap>` 标签内的`<id>` 子标签，指定了唯一确定一条记录的 `id` 列。MyBatis 会根据`<id>` 列值来完成 100 条记录的去重复功能，`<id>` 可以有多个，代表了联合主键的语意。

同样主对象的关联对象，也是根据这个原理去重复的。尽管一般情况下，只有主对象会有重复记录，关联对象一般不会重复。例如：下面 `join` 查询出来6条记录，一、二列是 Teacher 对象列，第三列为 Student 对象列。MyBatis 去重复处理后，结果为 1 个老师和 6 个学生，而不是 6 个老师和 6 个学生。

| t_id | t_name  | s_id |
| :--- | :------ | :--- |
| 1    | teacher | 38   |
| 1    | teacher | 39   |
| 1    | teacher | 40   |
| 1    | teacher | 41   |
| 1    | teacher | 42   |
| 1    | teacher | 43   |

## 简述 MyBatis 的插件运行原理？以及如何编写一个插件？

MyBatis 仅可以编写针对 ParameterHandler、ResultSetHandler、StatementHandler、Executor 这 4 种接口的插件。

MyBatis 使用 JDK 的动态代理，为需要拦截的接口生成代理对象以实现接口方法拦截功能，每当执行这 4 种接口对象的方法时，就会进入拦截方法，具体就是 InvocationHandler 的 `#invoke(...)`方法。当然，只会拦截那些你指定需要拦截的方法。

------

编写一个 MyBatis 插件的步骤如下：

1. 首先，实现 MyBatis 的 Interceptor 接口，并实现 `#intercept(...)` 方法。
2. 然后，在给插件编写注解，指定要拦截哪一个接口的哪些方法即可
3. 最后，在配置文件中配置你编写的插件。

具体的，可以参考 [《MyBatis 官方文档 —— 插件》](http://www.MyBatis.org/MyBatis-3/zh/configuration.html#plugins) 。

## MyBatis 是如何进行分页的？分页插件的原理是什么？

MyBatis 使用 RowBounds 对象进行分页，它是针对 ResultSet 结果集执行的**内存分页**，而非**数据库分页**。

所以，实际场景下，不适合直接使用 MyBatis 原有的 RowBounds 对象进行分页。而是使用如下两种方案：

- 在 sql 内直接书写带有数据库分页的参数来完成数据库分页功能
- 也可以使用分页插件来完成数据库分页。

这两者都是基于数据库分页，差别在于前者是工程师**手动**编写分页条件，后者是插件**自动**添加分页条件。

------

分页插件的基本原理是使用 MyBatis 提供的插件接口，实现自定义分页插件。在插件的拦截方法内，拦截待执行的 sql ，然后重写 sql ，根据dialect 方言，添加对应的物理分页语句和物理分页参数。

举例：`SELECT * FROM student` ，拦截 sql 后重写为：`select * FROM student LIMI 0，10` 。

目前市面上目前使用比较广泛的 MyBatis 分页插件有：

- [MyBatis-PageHelper](https://github.com/pagehelper/MyBatis-PageHelper)
- [MyBatis-Plus](https://github.com/baomidou/MyBatis-plus)

从现在看来，[MyBatis-Plus](https://github.com/baomidou/MyBatis-plus) 逐步使用的更加广泛。

## MyBatis 比 IBatis 比较大的几个改进是什么？

> 这是一个选择性了解的问题，因为可能现在很多面试官，都没用过 IBatis 框架。

1. 有接口绑定，包括注解绑定 sql 和 XML 绑定 sql 。
2. 动态 sql 由原来的节点配置变成 OGNL 表达式。
3. 在一对一或一对多的时候，引进了 `association` ，在一对多的时候，引入了 `collection`节点，不过都是在 `<resultMap />` 里面配置。

## MyBatis 映射文件中，如果 A 标签通过 include 引用了B标签的内容，请问，B 标签能否定义在 A 标签的后面，还是说必须定义在A标签的前面？

> 老艿艿：这道题目，已经和源码实现，有点关系了。

虽然 MyBatis 解析 XML 映射文件是**按照顺序**解析的。但是，被引用的 B 标签依然可以定义在任何地方，MyBatis 都可以正确识别。**也就是说，无需按照顺序，进行定义**。

原理是，MyBatis 解析 A 标签，发现 A 标签引用了 B 标签，但是 B 标签尚未解析到，尚不存在，此时，MyBatis 会将 A 标签标记为**未解析状态**。然后，继续解析余下的标签，包含 B 标签，待所有标签解析完毕，MyBatis 会重新解析那些被标记为未解析的标签，此时再解析A标签时，B 标签已经存在，A 标签也就可以正常解析完成了。

可能有一些绕，胖友可以看看 [《精尽 MyBatis 源码解析 —— MyBatis 初始化（一）之加载 MyBatis-config》](http://svip.iocoder.cn/MyBatis/builder-package-1) 。

此处，我们在引申一个问题，Spring IOC 中，存在互相依赖的 Bean 对象，该如何解决呢？答案见 [《【死磕 Spring】—— IoC 之加载 Bean：创建 Bean（五）之循环依赖处理》](http://svip.iocoder.cn/Spring/IoC-get-Bean-createBean-5/) 。

## 简述 MyBatis 的 XML 映射文件和 MyBatis 内部数据结构之间的映射关系？

MyBatis 将所有 XML 配置信息都封装到 All-In-One 重量级对象Configuration内部。

在 XML Mapper 文件中：

- `<parameterMap>` 标签，会被解析为 ParameterMap 对象，其每个子元素会被解析为 ParameterMapping 对象。
- `<resultMap>` 标签，会被解析为 ResultMap 对象，其每个子元素会被解析为 ResultMapping 对象。
- 每一个 `<select>`、`<insert>`、`<update>`、`<delete>` 标签，均会被解析为一个 MappedStatement 对象，标签内的 sql 会被解析为一个 Boundsql 对象。
