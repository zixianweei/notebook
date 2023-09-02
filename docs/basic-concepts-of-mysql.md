---
comments: true
---

# MySQL基础知识

本篇包含了MySQL主要知识点总结。

<!-- more -->

## 数据库，数据库管理系统，SQL

这三个名词在数据库相关的内容中十分常见，但是表示不同的意思且容易混淆。DB，数据库；DBMS，数据库管理系统(database management system)；SQL，结构化查询语言(structured query language)，一种高级语言。

## 数据表的结构

数据表是数据库的基本组成单元。所有的数据在数据库中都以表格的形式组织，目的是可读性强。
数据表的行，数据；数据表的列，字段。

## SQL语句的分类

SQL语句在分类上可以被分为五个类别：

- DQL，数据查询语言：`select`
- DML，数据操作语言：`insert`，`delete`，`update`
- DDL，数据定义语言：`create`，`drop`，`alter`
- TCL，事务控制语言：`commit`，`rollback`
- DCL，数据控制语言：`grant`，`revoke`

## MySQL导入数据的步骤

- 登录MySQL数据管理系统：`mysql -uroot -p`
- 查看有哪些数据库：`show databases;`，(不是SQL语句，是MySQL命令)
- 创建数据库：`create database <dbname>;`，(MySQL命令)
- 使用`<dbname>`数据库：`use <dbname>;`，(MySQL命令)
- 查看当前使用的数据库中有哪些数据表：`show tables;`，(MySQL命令)
- 初始化数据表：`source xxx.sql`，(MySQL命令)

## 其他常用的命令

- 查看数据表的结构：`desc <tablename>;`
- 查看当前使用的数据库：`select database();`
- 查看MySQL的版本信息：`select version();`
- 结束一条语句：加上`\c`
- 查看创建数据表的语句：`show create table <tablename>;`

## 简单的查询语句(DQL)

一般的格式：`select <字段1>,<字段2>,... from <tablename>;`

查询语句注意事项：

- 任何一条SQL语句都应该以`;`结尾
- SQL语句不区分大小写
- 字段可以参与数学运算
- 标准的SQL语句中要求字符串使用单引号包裹
- `as`关键字可以被省略：`select ename, sal*12 year_sal from emp;`
- 语句`select * from <tablename>;`可以查询整个表中的所有内容，但是效率比较低

## 条件查询

### `where`语句

语句格式：`select <字段1>,<字段2>,... from <tablename> where <condition>;`

执行顺序：先`from`，再`where`，最后`select`。

### `between ... and ...`语句

`between ... and ...`表示闭区间，并且左小右大；该语句还可以用在字符串中，但是此时表示左闭右开的区间。

### `is null`语句

在数据库中，`null != 0`，`null`表示的含义是什么都没有，为空，而`0`为一个具体的值。因此，不能够将`null`和`0`之间划等号。表示为空应使用`is null`；表示不为空应使用`is not null`。

### `in`和`or`语句

`in`和`or`可以视为相同，但是二者后面的参数不同。同时，注意`in`后面的参数是具体的值，而不是区间。`not in`是`in`的反义。

`select ename,job from emp where job='SALESMAN' or job='MANAGER';`

`select ename,job from emp where job in ('SALESMAN','MANAGER');`

### 模糊查询

模糊查询使用`like`关键字。而使用时需要使用`%`和`_`两个字符来匹配。其中，`%`表示任意多个字符，`_`表示任意一个字符。

`select ename from emp where ename like '%o%';`

`select ename from emp where ename like '_a%';`

## 排序

语句格式：`select <字段1>,<字段2>,... from <tablename> where <condition> order by <字段i> (asc/desc);`

排序涉及的关键字有`asc`和`desc`两个。其中，`asc`表示升序，`desc`表示降序。连接语句时，使用`order by`，默认情况下为升序。`order by`是最后执行的，且字段可以用`select`后的字段名表示，因此不具备健壮性。

## 分组函数

分组函数是指对一组数据进行操作的函数。分组函数共有5个，分别是`count`，`sum`，`avg`，`max`和`min`。分组函数还有另一个名字：多行处理函数。分组函数的特点：输入是多行，但是输出只有一行。

分组函数在遇到`null`时情况较为特殊。分组函数会自动忽略`null`，只要有`null`参与运算，那么结果一定是`null`。而为了得到正确的结果，需要使用`ifnull(<字段>, value)`来处理，即，如果某个字段被检索到后发现是`null`，那就使用`value`来代替`null`进行后续的数据处理。注意，`ifnull`是一个单行处理函数。

`select ename,(sal + ifnull(comm,0))*12 yealsal from emp;`

分组函数不可以直接出现在`where`语句中。特殊的，`count(*)`不是统计某个字段中数据的个数，而是总的记录数，和字段无关；`count(<字段>)`统计`<字段>`中不为`null`的数据总数。最后，分组函数也可组合起来使用。

## `group by`与`having`

`group by`：按照某个字段或者某些字段进行分组；`having`：对分组之后的数据进行再次过滤。

分组函数一般都会和`group by`联合使用，并且任何一个分组函数都是在`group by`语句执行完成之后才会执行。当一条SQL语句中没有`group by`时，整张表的数据会自动被视作一组；当一条SQL语句中有`group by`，`select`后面只能跟分组函数和参与分组的字段。

`select deptno,max(sal) from emp group by deptno having max(sal)>2900;`

## 去重

去重使用的关键字是`distinct`，该关键字只能出现在所有字段的最前面；所有字段联合去重。

## 连接查询

在实际开发过程中，大部分的情况下都不是在单张表中进行数据查询，一般都是在多张表中进行李联合查询，取出最终结果。在表的连接查询方面有一种现象被称为：笛卡尔积现象，当两张表进行连接查询时，没有任何条件限制的情况下，最终查询的结果条数是两张表记录的乘积。

### 分类

根据语法出现的年代：SQL92和SQL99

根据表的连接方式：内连接(等值连接，非等值连接，自连接)；外连接(左连接，右连接)；全连接。

### 内连接(等值连接)

最大的特点：进行连接的条件是等量关系。语句格式：`... A join B <condition> where ...`

SQL92：`select e.ename,d.dname from emp e, dept d where e.deptno=d.deptno;`

SQL99：`select e.ename,d.dname from emp e (inner) join dept d on e.deptno=d.deptno;`

SQL99语法结构更加清晰，表的连接条件和后来的where条件分离。

### 内连接(非等值连接)

最大的特点：连接中的关系是非等量关系。

*找出每个员工的工资等级，要求显示员工名，工资，和工资等级*

`select e.ename,e.sal,s.grade from emp e join salgrade s on e.sal between s.losal and s.hisal;`

### 内连接(自连接)

最大的特点：一张表看作两张表，自己连接自己。

*找出员工的上级领导，要求显示员工名和对应的领导名*

`select a.ename,b.ename from emp a join emp b on a.mgr=b.empno;`

### 外连接

内连接：假设A表和B表进行连接，使用内连接的话，凡是A表和B表能够匹配上的记录都查询出来。A表和B表没有主副之分，是平等的关系。

外连接：假设A表和B表进行连接，使用外连接的话，AB两张表中有一张表是主表，另一个是副表。主要查询主表中的数据，捎带查询副表，当副表中的数据没有与主表中的数据匹配上，副表会自动模拟出`null`与之匹配。

外连接最重要的特点是：主表的数据会无条件的全部查询出来。

### 三张表的连接查询

*找出每个员工的部门名称以及工资等级*

`select e.ename,d.dname,s.grade from emp e join dept d on e.deptno = d.deptno join salgrade s on e.sal between s.losal and s.hisal;`

### 子查询

`select`语句中嵌套`select`语句，被嵌套的`select`语句是子查询。子查询的位置：`select`后，`from`后，`where`后。

### `union`

可以将两次查询的结果相叠加。

### `limit`

分页查询，`limit`是MySQL中特有的，其他的数据库中没有，不通用。Oracle中有一个相同的机制，叫做`rownum`。`limit`的作用是取结果集中的部分数据。`limit`是SQL语句中最后执行的环节。

语句格式：`... limit <startindex>,<length>`，其中，`<startindex>`表示起始位置，从0开始；`<length>`表示取出几个数据。

## 创建表

语句格式：
```mysql
create table <tablename> (
    <字段名>,<数据类型>,<约束>
    ...
);
```

常见的数据类型：

- `INT`：整数型
- `BIGINT`：长整型
- `FLOAT`：浮点型
- `CHAR`：定长字符型
- `VARCHAR`：可变长字符型
- `DATE`：日期类型
- `BLOB`：二进制大对象(存储图片，视频等流媒体信息)
- `CLOB`：字符大对象(存储较大文本)

### `CHAR`与`VARCHAR`

在实际开发中，当某个字段中的数据长度不发生改变的时候，是定长的，采用`CHAR`；当一个字段的数据长度不确定，采用`VARCHAR`。


### 插入数据

语句格式：
```mysql
insert into <tablename> (<字段1>,<字段2>,...) values (
    <value1>,<value2>,...
);
```

字段的数量和值的数量应该相同，并且数据类型要对应相同。当一条`insert`语句执行成功后，表格中会多一条记录，即便多的这行记录中某些字段是`null`，后期也无法再执行`insert`语句进行数据插入而修改，只能通过`update`进行更新。`insert`可以一次插入多条记录，values使用"`,`"隔开。

### 表的复制

将查询的结果当作表创建出来：`create table <tablename> as <DQL语句>;`

### 将查询结果插入到表中

`insert into <tablename> <DQL语句>`

### 修改数据

`update <tablename> set <字段1>=<value1>,<字段2>=<value2>,... where <condition>;`

没有条件时表示整张表的数据都进行更新。

### 删除数据

`delete from <tablename> where <condition>;`

没有条件时表示整张表的数据全部删除。

大表删除：`truncate table <tablename>;`，表会被截断并且不可回滚，永久丢失。

### 约束

在创建表的时候，可以给表的字段添加相应的约束。添加约束的目的是为了表中数据的合法性，有效性，和完整性。

常见的约束：

| 名称 | 关键字 | 作用 |
| :---: | :---: | :---: |
| 非空约束 | `not null` | 约束的字段不可为`null` |
| 唯一约束 | `unique` | 约束的字段不能重复|
| 主键约束 | `primary key` | 约束的字段即不能为`null`也不能重复 |
| 外键约束 | `foreign key` |  |
| *检查约束 | `check` | Oracle数据库中有check约束，MySQL中没有 |

### 非空约束

没有列级约束和表级约束之分。

### 唯一约束

唯一约束修饰的字段具有唯一性，不可重复，但是可以为`null`，具有列级约束和表级约束。

### 主键约束

添加了主键约束的字段，数据不能为`null`，也不能重复。主键约束，主键字段，主键值。

作用：表的设计三范式中有要求，第一范式就要求任何一张表都应该有主键，主键值是这行记录在这张表中的唯一标识。

分类：根据主键字段的数量来划分：单一主键，复合主键(违反三范式)；根据主键的性质划分：自然主键(主键值最好就是一个和业务无关的自然数)，业务主键(主键值和业务挂钩)。

最好不要拿着和业务挂钩的字段作为主键值。因此以后的业务一旦发生改变的时候，主键值可以需要随之发生变化，但是有的时候没有办法改变，因此变化可能会导致主键重复。

一张表的主键约束只能有一个，包含列级约束和表级约束。MySQL提供主键值自增的功能。
`... id,INT,primary key, auto_increment ...`

### 外键约束

外键约束，外键字段，外键值。

删除数据的时候，先删除子表，再删除父表；添加数据的时候，先添加父表，再添加子表；创建表的时候，先创建父表，再创建子表；删除表的时候，先删除子表，再删除父表。

外键值可以为`null`，外键字段引用的字段不一定是主键，但必须有唯一性(有unique约束)。

## 存储引擎

存储引擎描述的是表的存储方式。一个完整的建表语句应该是：

```mysql
create table <tablename> (
    <字段>,<type>,<constraint>
    ...
) ENGINE=InnoDB DEFAULT CHARSET=uft8;
```

建表的时候可以指定存储引擎，也可以指定字符集。MySQL默认使用的存储引擎是InnoDB，默认使用的字符集是UTF8。“存储引擎”这个名词只在MySQL中使用，Oracle中有一个对应的机制叫做“表的存储方式”。MySQL支持很多的存储引擎，每一种存储引擎都对应了一种不同的存储方式。

查看MySQL支持的引擎：`show engines \G;`

### 常见的存储引擎(MyISAM)

MyISAM不支持事务，是MySQL中最常用的存储引擎，但是不是默认的。MyISAM使用三个文件表示每张数据表：

- 格式文件：存储表结构的定义，`.frm`，`format`
- 数据文件：存储表行的内容，`.MYD`，`data`
- 索引文件：存储表中的索引，`.MYI`，`index`

MyISAM可以被转换为压缩的，只读的文件，来节省空间。

### 常见的存储引擎(InnoDB)

InnoDB支持事务，支持行级锁，和外键。因此，这种存储引擎的数据安全能够得到保障。该引擎中，表的结构存储在`.frm`文件中，数据存储在`tablespace`的表空间中，无法被压缩，无法被转换为只读。`tablespace`是一个逻辑概念，不对应实际文件。

InnoDB引擎在MySQL服务器崩溃后提供自动恢复功能，并且支持级联删除和级联更新(外键)。

### 常见的存储引擎(MEMORY/HEAP)

MEMOEY存储引擎不支持事务，数据容易丢失。因此所有的数据和索引信息都是存储在内存中的。但是该存储引擎的查询速度最快。

## 事务(transaction)

一个事务是一个完整的业务逻辑单元，不可再分。要想保证两条以上的DML语句同时成功或同时失败，就需要借助数据库中的“事务机制”。和事务相同的语句只有DML语句(`insert`，`delete`，`update`)。因为这三个语句都是和数据库表中的数据相关的。事务的存在是为了保障数据的完整性和安全性。假设所有的业务都能使用一条DML语句完成，则不需要事务；但是在实际业务中，通常一项任务就需要多条DML语句配合完成，因此需要事务存在。

事务的使用语句：`commit`，`rollback`，`savepoint`(TCL语句)

### 事务的特性

ACID：A，原子性，事务是最小的工作单元，不可再分；C，一致性，事务必须保证多条DML语句同时完成或同时失败；I，隔离性，事务A与事务B之间相互隔离；D，持久性，最终数据必须持久化到硬盘文件中，事务才算成功的结束。

### 事务之间的隔离性

事务的隔离性存在隔离级别，理论上隔离级别包括4个。

- 第一级别：读未提交(read uncommited)，对方事务还没有提交，当前事务可以读取到对方未提交的数据；读未提交存在脏读现象(dirty read)，表示读到了脏的数据。
- 第二级别：读已提交(read commited)，对方事务提交之后的数据被读取到；读已提交存在不可重复读的问题，但是解决了脏读现象。
- 第三级别：可重复读(repeatable read)，解决了不可重复读的问题，但是存在读取的数据是幻象的问题。
- 第四级别：序列化读(serializable)，解决了所有问题，但是效率低，各个事务需要排队处理。

Oracle数据库默认的隔离级别是：读已提交；MySQL数据库默认的隔离级别是：可重复读。MySQL数据库中，事务在默认情况下是自动提交的，即，只要执行了任意一条DML语句，则进行一次提交。

关闭自动提交：`start transaction;`，设置全局事务的隔离级别：`set global transaction ioslation level read uncommited`，查看事务的全局隔离级别：`select @@global.tx_isolation;`

## 索引

索引就相当于一本书的目录，通过目录可以快速的查找到对应的资源。在数据库中，查询一张表的时候有两种检索方式：全表扫描和根据索引扫描，显然，后者效率高。

索引缩小了扫描的范围，但是索引虽然可以提高检索的效率，但是不能随意的添加。因为，索引也是数据库中的对象，也需要数据库不断的维护，是有维护成本的。**表中经常被修改的部分就不适合添加索引，因为数据一旦修改，索引就需要重新排列，进行维护**。添加索引是为某一个或者某一些字段添加。

何时考虑添加索引？1.数据量庞大；2.该字段很少进行DML操作；3.该字段经常出现在`where`子句中。主键和具有唯一约束的字段会自动添加索引，因此根据主键查询的效率较高，应尽量根据主键进行检索。

### 添加索引

语句格式：`create index <indexname> on <tablename>(<字段>);`。

查看SQL语句的执行方式：`explain <SQL语句>;`

### 删除索引

语句格式：`drop index <indexname> on <tablename>;`

### 索引的底层结构与实现原理

B+树。通过B+树缩小扫描范围，底层索引进行了排序，分区。索引会携带数据在表中的“物理地址”，最终通过索引检索数据的时候，会获取到关联的物理地址，通过物理地址定位表中的数据，效率是最高的。

### 索引的分类

- 单一索引：给单个字段添加索引
- 复合索引：多个字段联合起来，添加一个索引
- 主键索引：主键上会自动添加索引
- 唯一索引：有唯一约束的字段会自动添加索引

### 索引失败

索引失败的情况：进行模糊查询时，查询的名称中第一个字符是通配符`%`。

## 视图

站在不同的角度去看待数据。同一张表，通过不同的角度去看。

创建视图：`create view <viewname> as <select statement>;`

删除视图：`drop view <viewname>;`

对视图进行增删查改会影响到原表中的数据，通过视图影响原表的数据而不是直接操作原表的数据，因而视图可以隐藏原表的实现细节。

## DBA命令

将数据库中的数据导出：

`mysqldump <dbname> > <filename>.sql -uroot -p;`

`mysqldump <dbname> <tablename> > <filename>.sql -uroot -p;`

## 数据库设计三范式

设计数据表的依据，按照三范式设计的数据表不会出现数据冗余。

- 第一范式：任何一张表都应该有主键，并且每个字段都应该是原子性的，不可再分。
- 第二范式：建立在第一范式的基础上，所有非主键字段完全依赖主键，不能产生部分依赖。(多对多，三张表，关系表两个外键)
- 第三范式：建立在第二范式的基础上，所有非主键字段直接依赖主键，不能产生传递依赖。(一对多，两张表，多的表加外键)

(一对一，主键共享，主键也是外键，外键唯一，外键加上唯一约束)