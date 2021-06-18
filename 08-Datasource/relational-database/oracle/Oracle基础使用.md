<!-- MarkdownTOC -->
- [一、简介](#一简介)
- [二、体系结构](#二体系结构)
- [三、用户](#三用户)
- [四、表的管理](#四表的管理)
  - [1.Oracle数据类型](#1oracle数据类型)
  - [2.创建表](#2创建表)
  - [3.删除表](#3删除表)
  - [4.修改表](#4修改表)
  - [5.数据库表数据的更新](#5数据库表数据的更新)
  - [6.序列](#6序列)
- [五、单行函数](#五单行函数)
  - [1.字符函数](#1字符函数)
  - [2.数值函数](#2数值函数)
  - [3.日期函数](#3日期函数)
  - [4.转换函数](#4转换函数)
  - [5.通用函数](#5通用函数)
- [六、多行函数](#六多行函数)
- [七、分组统计](#七分组统计)
- [八、多表查询](#八多表查询)
  - [1.基础查询](#1基础查询)
  - [2.外连接（左右连接）](#2外连接左右连接)
- [九、子查询](#九子查询)
- [十、ROWNUM与分页查询](#十rownum与分页查询)
- [十一、视图](#十一视图)
- [十二、索引](#十二索引)
- [十三、PL/SQL基本语法](#十三plsql基本语法)
- [十四、分支](#十四分支)

<!-- /MarkdownTOC -->

# 一、简介
Oracle数据库系统是美国Oracle公司（甲骨文）提供的以分布式数据库为核心的一组软件产品，是目前最流行的客户/服务器(CLIENT/SERVER)或B/S体系结构的数据库之一。

# 二、体系结构
**1.数据库**

Oracle数据库是数据的物理存储。这就包括（数据文件ORA或者DBF、控制文件、联机日志、参数文件）。其实Oracle数据库的概念和其它数据库不一样，这里的数据库是一个操作系统只有一个库。可以看作是Oracle就只有一个大数据库。

**2.实例**

一个Oracle实例（Oracle Instance）有一系列的后台进程（Backguound Processes)和内存结构（Memory Structures)组成。一个数据库可以有n个实例。

**3.用户**

用户是在实例下建立的。不同实例可以建相同名字的用户。

**4.表空间**

表空间是Oracle对物理数据库上相关数据文件（ORA或者DBF文件）的逻辑映射。一个数据库在逻辑上被划分成一到若干个表空间，每个表空间包含了在逻辑上相关联的一组结构。每个数据库至少有一个表空间(称之为system表空间)。  
每个表空间由同一磁盘上的一个或多个文件组成，这些文件叫数据文件(datafile)。一个数据文件只能属于一个表空间。

**5.数据文件（dbf、ora）**

数据文件是数据库的物理存储单位。数据库的数据是存储在表空间中的，真正是在某一个或者多个数据文件中。而一个表空间可以由一个或多个数据文件组成，一个数据文件只能属于一个表空间。一旦数据文件被加入到某个表空间后，就不能删除这个文件，如果要删除某个数据文件，只能删除其所属于的表空间才行。  
注：表的数据，是有用户放入某一个表空间的，而这个表空间会随机把这些表数据放到一个或者多个数据文件中。  

由于oracle的数据库不是普通的概念，oracle是有用户和表空间对数据进行管理和存放的。但是表不是有表空间去查询的，而是由用户去查的。因为不同用户可以在同一个表空间建立同一个名字的表！这里区分就是用户了。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200920124944833.png" width="600px"/>
</div>

下面我们创建一个表空间learn：
```oracle
CREATE TABLESPACE learn   --表空间名称
DATAFILE 'd:\learn.dbf'   --指定表空间对应的数据文件
SIZE 100m                 --表空间的初始大小
AUTOEXTEND ON             --表空间存储都占满时，自动增长
NEXT 10m;                 --自动增长的大小
```

# 三、用户
oracle的表和其它的数据库对象都是存储在用户下的。  

**1.创建用户**

```oracle
CREATE USER root
IDENTIFIED BY root         --用户的密码
DEFAULT TABLESPACE learn;  --表空间名称
```
**2.赋予权限**

初始化Oracle数据库只有system用户具有DBA权限，所有需要通过system用户给root用户赋予DBA权限，否则无法正常登陆root用户。
```oracle
GRANT DBA TO root;
```
切换到root用户下： `root@ORCL - PL/SQL Developer -- 创建表空间 create tablespace...`

==注意== 
Oracle主要存在三个重要的角色：CONNECT角色，RESOURCE角色，DBA角色。

| CONNECT                                                                                                                                                                                                            | RESOURCE                                                                                                                                                               | DBA          |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ |
| 授予最终用户的典型权利                                                                                                                                                                                             | 授予开发人员                                                                                                                                                           | 系统最高权限 |
| ALTER SESSION --修改会话<br>CREATE CLUSTER --建立聚簇<br>CREATE DATABASE LINK --建立数据库链接<br>CREATE SEQUENCE --建立序列<br>CREATE SESSION --建立会话<br>CREATE SYNONYM --建立同义词<br>CREATE VIEW --建立视图 | CREATE CLUSTER --建立聚簇<br>CREATE PROCEDURE --建立过程<br>CREATE SEQUENCE --建立序列<br>CREATE TABLE --建表<br>CREATE TRIGGER --建立触发器<br>CREATE TYPE --建立类型 | 任何操作     |

# 四、表的管理
## 1.Oracle数据类型

| 数据类型          | 描述                                                                                      |
| ----------------- | ----------------------------------------------------------------------------------------- |
| Varchar，varchar2 | 表示一个字符串                                                                            |
| NUMBER            | NUMBER(n)表示一个整数，长度是n<br>NUMBER(m,n):表示一个小数，总长度是m，小数是n，整数是m-n |
| DATA              | 表示日期类型                                                                              |
| CLOB              | 大对象，表示大文本数据类型，可存4G                                                        |
| BLOB              | 大对象，表示二进制数据，可存4G                                                            |

## 2.创建表
```oracle
CREATE TABLE person(
       pid NUMBER(20),
       pname VARCHAR2(10),
       gender NUMBER(1) DEFAULT 1,
       brithday DATE
);
```
## 3.删除表
```oracle
DELETE FROM person;
DROP TABLE person;
TRUNCATE TABLE person;
```
## 4.修改表
* 添加一列
```oracle
ALTER TABLE person ADD (gender NUMBER(1));
```
* 修改列类型
```oracle
ALTER TABLE person MODIFY father CHAR(10);
```
* 修改列名称
```oracle
ALTER TABLE person RENAME COLUMN gender TO sex;
```
* 删除一列
```oracle
ALTER TABLE person DROP COLUMN father;
```

## 5.数据库表数据的更新
* 查询数据
```oracle
SELECT * FROM person;
```
* 插入数据
```oracle
INSERT INTO person(pid, pname, gender, brithday)VALUES(1, '张三', 1, to_date('2000-01-01', 'yyyy-MM-dd'));
COMMIT;
```
<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200920132651564.png" width="400px"/>
</div>

* 修改数据
```oracle
UPDATE person SET pname = '李四' WHERE pid =1;
COMMIT;
```

## 6.序列
在很多数据库中都存在一个自动增长的列,如果现在要想在oracle 中完成自动增长的功能, 则只能依靠序列完成,所有的自动增长操作,需要用户手工完成处理。

序列不属于任何一张表，但是可以逻辑和表绑定。  
下面，创建一个s_person的序列,验证自动增长的操作：
```oracle
CREATE SEQUENCE s_person;
```
序列创建完成之后,所有的自动增长应该由用户自己处理,所以在序列中提供了以下的两种操作：  
```oracle
SELECT s_person.nextval FROM dual; --取得序列的下一个内容 
SELECT s_person.currval FROM dual; --取得序列的当前内容 
```
其中，dual是虚表，补全语法，没有任何意义  

在插入数据时需要自增的主键中可以这样使用：
```oracle
INSERT INTO person (pid,pname) VALUES (s_person.nextval,'小明');
COMMIT;
```

# 五、单行函数
初始化Oracle数据库中有一个Scott用户，密码默认为tiger，我们就用它来学习。
解锁并切换到Scott用户：
```oracle
ALTER USER scott ACCOUNT UNLOCK;

ALTER USER scott IDENTIFIED BY tiger;
```
## 1.字符函数
接收字符输入返回字符或者数值，dual是伪表。  
```oracle
SELECT UPPER('yes') FROM dual;  --字符转换为大写
SELECT LOWER('NO') FROM dual;   --字符转换为小写
```
其他更多操作可查看API。
## 2.数值函数
* 四舍五入函数：ROUND()
```oracle
SELECT ROUND(28.12,0) FROM dual; --28
SELECT ROUND(28.12,-1) FROM dual; --28.1
SELECT TRUNC(28.12,3) FROM dual; --28.12
```
* 取余
```oracle
SELECT MOD(10,3) FROM dual; --1
```
其他更多操作可查看API。
## 3.日期函数
Oracle中提供了很多和日期相关的函数，包括日期的加减，在日期加减时有一些规律：  
* 日期–数字= 日期  
* 日期+数字= 日期  
* 日期–日期= 数字  

范例：  
（1）查询雇员的进入公司的周数。
```oracle
SELECT ROUND((SYSDATE-e.hiredate)/7) FROM emp e;
```
（2）获得两个时间段中的月数：months_between()
```oracle
SELECT months_between(SYSDATE,e.hiredate) FROM emp e;
```
（3）获得两个时间段中的年数：
```oracle
SELECT months_between(SYSDATE,e.hiredate)/12 FROM emp e;
```
## 4.转换函数
* TO_CHAR:字符串转换函数 

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200920145655693.png" width="400px"/>
</div>

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200920150036970.png" width="600px"/>
</div>

* TO_DATE:日期转换函数 

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200920150243918.png" width="500px"/>
</div>

## 5.通用函数
（1）空值处理nvl
null和任何数值计算都是null。 

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200920150602587.png" width="300px"/>
</div>

使用nvl来处理：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200920150930355.png" width="500px"/>
</div>

（2）Decode函数 
该函数类似if....else if...else 
语法：`DECODE(col/expression, [search1,result1],[search2, result2]....[default])`

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200920151536684.png" width="400px"/>
</div>

（3）case when 

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200920151725774.png" width="500px"/>
</div>

# 六、多行函数
多行函数：作用于多行，返回一个值  

**1.统计记录数count()**  

```oracle
SELECT COUNT(1) FROM emp;  
SELECT COUNT(*) FROM emp;
SELECT COUNT(ename) FROM emp;
```
**2.最小值查询min()**

```oracle
SELECT MIN(sal) FROM emp;
```
**3.最大值查询max()**

```oracle
SELECT MAX(sal) FROM emp;
```
**4.查询平均值avg()**

```oracle
SELECT AVG(sal) FROM emp;
```
**5.求和函数sum()**

```oracle
SELECT SUM(sal) FROM emp t WHERE t.deptno = 20;
```
# 七、分组统计
分组统计需要使用GROUP BY来分组。 
例如，查询出每个部门的平均工资：  

```oracle
SELECT e.deptno,AVG(e.sal) FROM emp e GROUP BY e.deptno;
```
查询出来部门编号，和部门下的人数：

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200920153157824.png" width="500px"/>
</div>

可见：  
* 如果使用分组函数，SQL只可以把GOURP BY分组条件字段和分组函数查询出来，不能有其他字段。  
* 如果使用分组函数，不使用GROUP BY 只可以查询出来分组函数的值。

可以这样修改： 

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200920153527495.png" width="400px"/>
</div>

又例如，查询出部门人数大于5人的部门： 

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200920153735947.png" width="400px"/>
</div>

# 八、多表查询
## 1.基础查询
使用一张以上的表做查询就是多表查询。例如：
```oracle
SELECT * FROM emp e,dept d WHERE e.deptno=d.deptno;
```
例子1：查询出雇员的编号，姓名，部门的编号和名称，地址 

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200920154735509.png" width="400px"/>
</div>

例子2：查询出每个员工的上级领导（自连接） 

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200920154948798.png" width="400px"/>
</div>

## 2.外连接（左右连接）
当我们在做基本连接查询的时候，查询出所有的部门下的员工，我们发现编号为40的部门下没有员工，但是要求把该部门也展示出来，我们发现上面的基本查询是办不到的。

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200920155336590.png" width="400px"/>
</div>
使用(+)表示左连接或者右连接，当(+)在左边表的关联条件字段上时是左连接，如果是在右边表的关联条件字段上就是右连接。

# 九、子查询
在一个查询的内部还包括另一个查询，则此查询称为子查询。 Sql的任何位置都可以加入子查询。  

子查询在操作中有三类：  
* 单列子查询——返回的结果是一列的一个内容  
* 单行子查询_返回多个列，有可能是一个完整的记录  
* 多行子查询：返回多条记录  

范例：查询出比雇员7654的工资高，同时从事和7788的工作一样的员工

<div align="center">  
<img src="https://img-blog.csdnimg.cn/2020092016021219.png" width="700px"/>
</div>

范例：要求查询每个部门的最低工资和最低工资的雇员和部门名称 

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200920160633623.png" width="700px"/>
</div>

# 十、ROWNUM与分页查询
ROWNUM:表示行号，实际上是一个列,但是这个列是一个伪列,此列可以在每张表中出现。 

范例：查询emp表带有rownum列 

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200920161119652.png" width="700px"/>
</div>

==注意== ：ROWNUM不支持大于号，只支持小于号。

如果想实现我们的需求怎么办呢？答案是使用子查询，也正是oracle分页的做法。  

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200920161726946.png" width="800px"/>
</div>

另一种写法： 

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200920162032645.png" width="700px"/>
</div>

# 十一、视图
视图就是封装了一条复杂查询的语句。 

**语法1**：CREATE VIEW 视图名称AS 子查询  

范例：建立一个视图，此视图包括了20部门的全部员工信息  

```oracle
CREATE VIEW empvd20 AS SELECT * FROM emp t WHERE t.deptno = 20;
```
视图创建完毕就可以使用视图来查询，查询出来的都是20部门的员工 

<div align="center">  
<img src="https://img-blog.csdnimg.cn/20200920193649460.png" width="700px"/>
</div>

**语法2**：CREATE OR REPLACE VIEW 视图名称AS 子查询

```oracle
CREATE OR REPLACE VIEW empvd20 AS SELECT * FROM emp t WHERE t.deptno = 20;
```
**语法3**：CREATE OR REPLACE VIEW 视图名称AS 子查询WITH READ ONLY  

视图作用：  
* 屏蔽掉一些敏感字段。
* 保证总部和分部数据及时统一。

# 十二、索引
索引是用于加速数据存取的数据对象。合理的使用索引可以大大降低I/O 次数,从而提高数据访问性能。  

**1.单列索引**

单列索引是基于单个列所建立的索引，比如:
```oracle
CREATE index 索引名on 表名(列名)
```
**2.复合索引**

复合索引是基于两个列或多个列的索引。在同一张表上可以有多个索引，但是要求列的组合必须不同,比如：  

Create index emp_idx1 on emp(ename,job); 

Create index emp_idx1 on emp(job,ename);  

范例：给person表的name建立索引

```oracle
CREATE INDEX pname_index ON person(pname);
```
范例：给person表创建一个name和gender的索引
```oracle
CREATE INDEX pname_gender_index ON person(pname, sex);
```
**3.查看索引**

```oracle
select * from user_ind_columns where table_name = upper('表名');
```
# 十三、PL/SQL基本语法
**1.什么是PL/SQL**

PL/SQL（Procedure Language/SQL）是Oracle对sql语言的过程化扩展，指在SQL命令语言中增加了过程处理语句（如分支、循环等），使SQL语言具有过程处理能力。
把SQL语言的数据操纵能力与过程语言的数据处理能力结合起来，使得PLSQL面向过程但比过程语言简单、高效、灵活和实用。

例如：为职工涨工资，每人涨10％的工资
```oracle
update emp set sal=sal*1.1
```
**2.程序语法**

```oracle
DECLARE
       --说明部分（变量说明，游标申明，例外说明〕
BEGIN
       --语句序列（DML语句〕... 
EXCEPTION
       --例外处理语句
END;
```
**3.常量和变量定义**

* 变量的基本类型 

  变量的基本类型是oracle中的建表时字段的变量如char, varchar2, date, number, boolean, long。

定义语法：
```oracle
varl char(15);
Psal number(9,2);
```
说明变量名、数据类型和长度后用分号结束说明语句。
```oracle
married CONSTANT BOOLEAN:=TRUE
```
* 引用变量：
```oracle
myname emp.ename%TYPE;
```
引用型变量，即my_name的类型与emp表中ename列的类型一样在sql中使用into来赋值。
```oracle
DECLARE
	emprec emp.ename%TYPE;
begin
	SELECT  t.ename INTO emprec FROM emp t WHERE t.empno = 7369;
	dbms_output.put_line(emprec);
END;
```
输出：
```shell script
SMITH
```
* 记录型变量
```oracle
Emprec emp%ROWTYPE;
```
实例：
```oracle
DECLARE
	p emp%ROWTYPE;
BEGIN
	SELECT * INTO p FROM emp t WHERE t.empno = 7369;
	dbms_output.put_line(p.ename || ' ' || p.sal);
END;
```
输出：
```shell script
SMITH  800
```
# 十四、分支
**1.if分支**

* 语法一
```oracle
IF 条件 THEN 
	语句 1;
	语句 2;
END IF;
```
* 语法二
```oracle
IF 条件 THEN 
	语句序列 1;
ELSE 
	语句序列 2;
END IF;
```
* 语法三
```oracle
IF 条件 THEN 
	语句;
ELSIF 语句 THEN 
	语句;
ELSE 
	语句;
END IF;
```

范例：断人的不同年龄段18岁以下是未成年人，18岁以上40以下是成年人，40以上是老年人
```oracle
DECLARE 
  mynum NUMBER:= &NUM;
BEGIN
  IF mynum<18 THEN
    dbms_output.put_line('未成年');
  ELSIF mynum >= 18 AND mynum < 40 THEN
    dbms_output.put_line('中年');
 ELSIF mynum >= 40 THEN
    dbms_output.put_line('老年人');
  END IF;
END;
```
**2.LOOP循环语句**

* 语法一
```oracle
WHILE total <= 25000 LOOP
	.. .
	total : = total + salary;
END LOOP;
```
* 语法二
```oracle
Loop
	EXIT [when 条件];
	……
End loop
```
* 语法三
```oracle
FOR I IN 1 . . 3 LOOP
	语句序列 ;
END LOOP ;
```
范例：用三种方式分别输出1-10
```Oracle
---while 
DECLARE
  i NUMBER(2) :=1;
BEGIN
  WHILE i<11 LOOP
    dbms_output.put_line(i);
    i := i+1;
  END LOOP;
END;
--exit循环
DECLARE
  i NUMBER(2) :=1;
BEGIN
  LOOP
    EXIT WHEN i>10;
    dbms_output.put_line(i);
    i := i+1;
   END LOOP;
END;
--for循环
DECLARE
  i NUMBER(2) :=1;
BEGIN
  FOR i IN 1..10 LOOP
    dbms_output.put_line(i);
  END LOOP;
END;
```
**3.游标Cursor**

游标类似于Java中的集合。游标可以存储查询返回的多条数据。语法：  
```oracle
CURSOR 游标名 [ (参数名 数据类型,参数名 数据类型,...)] IS SELECT 语句;
```
例如：
```oracle
CURSOR c1 IS SELECT ename FROM emp;
```
游标的使用步骤：  
* 打开游标： open c1; (打开游标执行查询)
* 取一行游标的值： fetch c1 into pjob; (取一行到变量中)
* 关闭游标： close c1;(关闭游标释放资源)
* 游标的结束方式 exit when c1%notfound

**注意**： 上面的 pjob 必须与 emp 表中的 job 列类型一致：  
定义： pjob emp.empjob%type;

实例：
```oracle
--输出emp表中所有员工的姓名
DECLARE 
  CURSOR c1 IS SELECT * FROM emp;
  emprow emp%ROWTYPE;
BEGIN
  OPEN c1;
      LOOP
        FETCH c1 INTO emprow;
        EXIT WHEN c1%NOTFOUND;
        dbms_output.put_line(emprow.ename);
      END LOOP;
  CLOSE c1;
END;


--给指定部门员工涨工资
DECLARE 
  CURSOR c2(eno emp.deptno%TYPE) IS SELECT empno FROM emp WHERE deptno =  eno;
  en emp.empno%TYPE;
BEGIN
  OPEN c2(10);
       LOOP
         FETCH c2 INTO en;
         EXIT WHEN c2%NOTFOUND;
         UPDATE emp SET sal = sal + 100 WHERE empno=en;
         COMMIT; 
       END LOOP;
  CLOSE c2;
END;
```