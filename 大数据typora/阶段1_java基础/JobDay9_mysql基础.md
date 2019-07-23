---
typora-copy-images-to: ./图片/
---

## sql部分

### 1 **SQL的分类**

#### 1.1 **DDL：数据定义语言**

create，drop，alter..

#### 1.2 **DCL：数据控制语言**

grant，if…

#### 1.3 **DML：数据操纵语言**

insert，update，delete…

#### 1.4 **DQL：数据查询语言**

select

------

####  **SQL对数据库进行操作**

##### 数据库CURD

- create database 数据库名称 [character set 字符集 collate 字符集校对规则];

  ##### 

- show databases;

- show create databases 数据库名称;

  ##### 

- alter database 数据库名称 character set 字符集 collate 校对规则;

  ##### 

- drop database 数据库名称；

- use 数据库/select database()

------

#### sql对于表的CRUD

##### 创建

语法：create table 表名称(字段名称 字段类型(长度) 约束,字段名称 字段类型(长度) 约束…);

字段类型，一个实体对应一个表，一个实体属性对应表的一个字段。

- Java中的类型						MySQL中的类型
	 byte/short/int/long 					tinyint/smallint/int/bigint
	 float								float
	 double								double
	 boolean							bit
	 char/String							char和varchar类型

char和varchar的区别：

char代表是固定长度的字符或字符串。定义类型char(8),向这个字段存入字符串hello，那么数据库使用三个空格将其补全。

varchar代表的是可变长度的字符串。定义类型varchar(8), 向这个字段存入字符串hello,那么存入到数据库的就是hello。

- Date								date/time/datetime/timestamp

datetime和timestamp区别

datetime就是既有日期又有时间的日期类型，如果没有向这个字段中存值，数据库使用null存入到数据库中； timestamp也是既有日期又有时间的日期类型，如果没有向这个字段中存值，数据库使用当前的系统时间存入到数据库中。

- File									BLOB/TEXT



约束

- 约束作用：保证数据的完整性
- 单表约束分类：
- 主键约束：primary key 主键约束默认就是唯一 非空的
- 唯一约束：unique
- 非空约束：not null

```sql
create database web_test1;
use web_test1;
create table user(
	id int primary key auto_increment,
	username varchar(20) unique,
	password varchar(20) not null,
	age int,
	birthday date
);
```

##### 查看

​	查看数据库下的所有表  show tables；

​	某个表的结构信息  desc 表名；

##### 删除

​	drop table 表名；

##### 修改

**添加列**

​	alter table 表名 add 列名 类型(长度) 约束；

**修改列类型，长度，约束**

​	alter table 表名 modify 列名 类型(长度) 约束

**修改列名，长度，类型，约束**

​	alter table 表名 change 旧列名 新列名 类型(长度) 约束

**修改表名**

​	rename table 表名 to 新的表名;

**修改表的字符集**

​	alter table 表名 character set 字符集;

```
1添加表字段

alter table table1 add transactor varchar(10) not Null;

alter table   table1 add id int unsigned not Null auto_increment primary key

2.修改某个表的字段类型及指定为空或非空
alter table 表名称 change 字段名称 字段名称 字段类型 [是否允许非空];
alter table 表名称 modify 字段名称 字段类型 [是否允许非空];

alter table 表名称 modify 字段名称 字段类型 [是否允许非空];

3.修改某个表的字段名称及指定为空或非空
alter table 表名称 change 字段原名称 字段新名称 字段类型 [是否允许非空

4如果要删除某一字段，可用命令：ALTER TABLE mytable DROP 字段名;
```



------

#### **SQL操作表的记录**

##### 1.添加记录

语法：

​	向表中插入某些列：`insert into 表名 (列名1,列名2,列名3…) values (值1,值2,值3…)`

​	向表中插入所有列：`insert into 表名 values (值1,值2,值3…);`

**注意事项**

1. 值的类型与数据库中表列的类型一致。
2. 值的顺序与数据库中表列的顺序一致。
3. 值的最大长度不能超过列设置最大长度。
4. 值的类型是字符串或者是日期类型，使用单引号引起来。

**添加某几列**

`insert into user (id,username,password) values (null,'aaa','123');`

**添加所有列**

`insert into user values (null,'bbb','123',23,'1993-09-01');`

##### 2.修改表的记录

语法:

`update 表名 set 列名=值,列名=值 [where 条件];`

**注意事项**

1. 值的类型与列的类型一致。
2. 值的最大长度不能超过列设置的最大长度。
3. 字符串类型和日期类型添加单引号。

**修改某一列的所有值**

`update user set password = 'abc';`

**按条件修改数据**

`update user set password = 'xyz' where username = 'bbb';`

**按条件修改多个列**

`update user set password='123',age=34 where username='aaa';`

##### 3.删除表的记录

语法：

`delete from 表名 [where 条件];`

**注意事项**

1. 删除表的记录，指的是删除表中的一行记录。
2. 删除如果没有条件，默认是删除表中的所有记录。

**删除某一条记录**

`delete from user where id = 2;`

**删除表中的记录有两种做法:**

delete from user;

​	删除所有记录，属于DML语句，一条记录一条记录删除。事务可以作用在DML语句上的

truncate table user;

​	删除所有记录，属于DDL语句，将表删除，然后重新创建一个结构一样的表。事务不能控制DDL的

- `有个问题是如果用delete来删除一个超大的表，由于可以回滚，会导致日志把磁盘撑爆的情况。所以truncate删除更快`

#### 4.单表查询

**基本查询**

​	语法:select [distinct] \*|列名 from 表 [条件];

##### 4.1查询所有

​	select * from 表名；

##### 4.2 查看某几列

​	select 列名1，列名2 from 表名；

##### 4.3 不显示重复的值

​	select distinct 列名1，列名2 from 表名；

##### 4.4 别名查询

​	select name as mingzi,english+chinese+math as sum from exam;

##### 4.5  **条件查询**

**使用where子句**

-  **> , < , >= , <= , <> ,=**
- **like:模糊查询**
- **in:范围查询**
- **条件关联:and , or ,not**

like可以进行模糊查询,在like子句中可以使用_或者%作为占位符。_只能代表一个字符，而%可以代表任意个字符。

- like ‘李_’		：名字中必须是两个字，而且是姓李的。
	 like ‘李%’		：名字中姓李的学生，李子后可以是1个或任意个字符。
	 like ‘%四’		：名字中以四结尾的。
	 like ‘%王%’	：只要名称中包含这个字就可以。

```sql
-- 6、找出姓名以A、B、S开始的员工信息。
select * from emp where ename like 'a%' or ename like 'b%' or ename like 's%';
-- 7、找到名字长度为7个字符的员工信息。
select * from emp where ename like '_______';
-- 8、名字中不包含R字符的员工信息。
select * from emp where ename not like '%r%';
```

**排序查询**

**使用order by 字段名称 asc/desc;**

```sql
-- 10、返回员工的信息并按姓名降序,工资升序排列。
select * from emp order by ename desc,sal asc;
```

##### 4.6**分组统计查询**

**聚合函数使用**

- sum()
- count()
- max()
- min()
- avg()

查询所有学生各科的总成绩：

```sql
select sum(english)+sum(chinese)+sum(math) from exam;
```

```sql
--忽略带null的那条数据
select sum(english+chinese+math) from exam;
```

与上面的语句有什么不同？

\* 上面的语句是按照列的方式统计，英语成绩总和+语文成绩总和+数学成绩总和。

\* 下面的语句先计算英语+数学+语文然后再求和。

​	* 使用ifnull的函数

##### 4.7**分组查询**

**语法：使用group by 字段名称;**

按商品名称统计，统计每类商品花费的总金额在5000元以上的商品

**where的子句后面不能跟着聚合函数。如果现在使用带有聚合函数的条件过滤（分组后条件过滤）需要使用一个关键字having**

```sql
select product,sum(price) from orderitem  group by product having sum(price) > 5000; 
```

`S(select)…F(from)…W(where)…G(group by)…H(having)…O(order by);`