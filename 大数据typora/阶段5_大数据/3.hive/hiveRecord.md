## DDL

### 创建外部表

创建部门表

```mysql
create external table if not exists default.dept(deptno int,dname string,loc int)
row format delimited fields terminated by '\t';
```

创建员工表

```mysql
create external table if not exists default.emp(empno int,
ename string,
job string,
mgr int,
hiredate string, 
sal double, 
comm double,
deptno int)
row format delimited fields terminated by '\t';
```

导入数据

```mysql
load data local inpath '/opt/module/datas/dept.txt' into table default.dept;
load data local inpath '/opt/module/datas/emp.txt' into table default.emp;
```

### 分区表

创建

```mysql
create table dept_partition(
    deptno int, dname string, loc string
)
partitioned by (month string)
row format delimited fields terminated by '\t';
```

导入数据

```mysql
load data local inpath '/opt/module/datas/dept.txt' into table default.dept_partition partition(month='201711');
load data local inpath '/opt/module/datas/dept.txt' into table default.dept_partition partition(month='201711');
```



多分区连接

```sql
> select * from dept_partition where month=201711
> union
> select * from dept_partition where month=201712;
```

添加分区

```sql
alter table dept_partition add partition(month='201705') partition(month='201704');
```

删除分区

```sql
 alter table dept_partition drop partition(month='201704');
```

二级分区

```sql
 create table dept_partition2(
     deptno int, dname string, loc string
 )
 partitioned by (month string, day string)
row format delimited fields terminated by '\t';
```

加载数据

```sql
load data local inpath '/opt/module/datas/dept.txt' into table default.dept_partition2 partition(month='201709', day='13');
```

```
> dfs -ls /user/hive/warehouse/dept_partition2;
Found 1 items
drwxrwxrwx   - wxy supergroup          0 2018-11-25 09:35 /user/hive/warehouse/dept_partition2/month=201709
> dfs -ls /user/hive/warehouse/dept_partition2/month=201709;
Found 1 items
drwxrwxrwx   - wxy supergroup          0 2018-11-25 09:35 /user/hive/warehouse/dept_partition2/month=201709/day=13
```

查询

```sql
select * from dept_partition2 where month='201709' and day='13';
```



#### 二级分区数据关联

1.先创建分区 上传文件后修复

```sql
--上传数据
dfs -mkdir -p /user/hive/warehouse/dept_partition2/month=201709/day=12;
dfs -put /opt/module/datas/dept.txt /user/hive/warehouse/dept_partition2/month=201709/day=12;
--查询
select * from dept_partition2 where month='201709' and day='12';
OK
dept_partition2.deptno	dept_partition2.dname	dept_partition2.loc	dept_partition2.month	dept_partition2.day
Time taken: 2.824 seconds

--修复
msck repair table dept_partition2;
OK
Partitions not in metastore:	dept_partition2:month=201709/day=12
Repair: Added partition to metastore dept_partition2:month=201709/day=12
Time taken: 0.453 seconds, Fetched: 2 row(s)

--查询
select * from dept_partition2 where month='201709' and day='12';
OK
dept_partition2.deptno	dept_partition2.dname	dept_partition2.loc	dept_partition2.month	dept_partition2.day
10	ACCOUNTING	1700	201709	12
20	RESEARCH	1800	201709	12
30	SALES	1900	201709	12
40	OPERATIONS	1700	201709	12
Time taken: 0.216 seconds, Fetched: 4 row(s)
```

2.上传文件再分区

```sql
dfs -mkdir -p /user/hive/warehouse/dept_partition2/month=201709/day=11;
dfs -put /opt/module/datas/dept.txt  /user/hive/warehouse/dept_partition2/month=201709/day=11;

alter table dept_partition2 add partition(month='201709', day='11');
OK
Time taken: 0.272 seconds

select * from dept_partition2 where month='201709' and day='11';
OK
dept_partition2.deptno	dept_partition2.dname	dept_partition2.loc	dept_partition2.month	dept_partition2.day
10	ACCOUNTING	1700	201709	11
20	RESEARCH	1800	201709	11
30	SALES	1900	201709	11
40	OPERATIONS	1700	201709	11
Time taken: 0.157 seconds, Fetched: 4 row(s)

```

3.创建分区 load数据

```
dfs -mkdir -p /user/hive/warehouse/dept_partition2/month=201709/day=10;
load data local inpath '/opt/module/datas/dept.txt' into table dept_partition2 partition(month='201709',day='10');
```



### 修改表

重命名

```sql
alter table dept_partition2 rename to dept_partition3;
```

增加修改删除表分区



增加修改替换列信息

（1）查询表结构

```sql
desc dept_partition;
```

（2）添加列

```sql
alter table dept_partition add columns(desc string);
```

（3）更新列

```sql
alter table dept_partition change column deptdesc desc int;
```

（4）查询表结构

```sql
alter table dept_partition change column desc deptdesc int;
```

（5）替换列

```sql
alter table dept_partition replace columns(deptno string, dname string, loc string);
```



## DML
```
load data [local] inpath '/opt/module/datas/student.txt' [overwrite] into table student [partition (partcol1=val1,…)];
（1）load data:表示加载数据
（2）local:表示从本地加载数据到hive表；否则从HDFS加载数据到hive表
（3）inpath:表示加载数据的路径
（4）into table:表示加载到哪张表
（5）student:表示具体的表
（6）overwrite:表示覆盖表中已有数据，否则表示追加
（7）partition:表示上传到指定分区
```
### 1.数据导入

#### 通过查询语句向表中插入数据（Insert）

1）创建一张分区表

```
create table student(id string, name string) partitioned by (month string) row format delimited fields terminated by '\t';
```

2）基本插入数据

```
insert into table student partition(month='201709') values('100','lisi');
```

3）基本模式插入（根据单张表查询结果）

```
insert overwrite table student3 partition(month='201708')
select id, name from student;
```

4）多插入模式（根据多张表查询结果）

```
from student3
insert overwrite table student3 partition(month='201707')
select id, name where month='201708'
insert overwrite table student3 partition(month='201706')
select id, name where month='201708';
```



#### 查询语句中创建表并加载数据（As Select）

```
create tabel student4(id string,num string) as select * from student;
```



#### 创建表时通过Location指定加载数据路径

1）创建表，并指定在hdfs上的位置

```sql
create table if not exists student5(
    id int, name string
)
row format delimited fields terminated by '\t'
location '/user/hive/warehouse/student5';
```

2）上传数据到hdfs上

```sql
dfs -put /opt/module/datas/student.txt  /user/hive/warehouse/student5;
```

3）查询数据

```
select * from student5;
```



#### Import数据到指定Hive表中

先用export导出后，再将数据导入。

```
import table student2 partition(month='201709') from '/user/hive/warehouse/export/student';
```



### 2.数据导出

#### 2.1 Insert导出

​	1）将查询的结果导出到本地

```
insert overwrite local directory '/opt/module/datas/export/student'
select * from student;

[wxy@hadoop102 ~]$ cat /opt/module/datas/export/student/000000_0 
1001zhangshan
1002lishi
1003zhaoliu
1001zhangshan
1002lishi
1003zhaoliu
```

​	2）将查询的结果格式化导出到本地

```
insert overwrite local directory '/opt/module/datas/export/student1'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' COLLECTION ITEMS TERMINATED BY '\n'
select * from student;
```

​	3）将查询的结果导出到HDFS上(没有local)

```
insert overwrite directory '/user/atguigu/hive/warehouse/student2'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' COLLECTION ITEMS TERMINATED BY '\n'
select * from student;
```

#### 2.2 Hadoop命令导出到本地

```
dfs -get /user/hive/warehouse/student3/month=201709/000000_0  /opt/module/datas/export/student3.txt;
```

#### 2.3 Hive Shell 命令导出

```
bin/hive -e 'select * from default.student;' > /opt/module/datas/export/student4.txt;
```

#### 2.4 Export导出到HDFS上

```
export table default.student3 to '/user/hive/warehouse/export/student3';
```

#### 2.5 Sqoop导出

后续专门讲。



### 3.数据清理

truncate table



## 查询

### 1 基本查询（Select…From） 

#### 	1.1 全表和特定字段查询

​		1）全表查询
​		2）选择特定列查询
​		注意：
​			（1）SQL 语言大小写不敏感。 
​			（2）SQL 可以写在一行或者多行
​			（3）关键字不能被缩写也不能分行
​			（4）各子句一般要分行写。
​			（5）使用缩进提高语句的可读性。

#### 	1.2 列别名

​		1）重命名一个列。
​		2）便于计算。
​		3）紧跟列名，也可以在列名和别名之间加入关键字‘AS’ 
​		4）案例实操
​			（1）查询名称和部门

```sql
select ename name, deptno as dno from emp;
```

#### 	1.3 算术运算符

​		算术运算符.png
​		案例实操

```sql
select sal +1 from emp;
```
#### 	1.4 常用函数

​		1）求总行数（count）
​		2）求工资的最大值（max）
​		3）求工资的最小值（min）
​		4）求工资的总和（sum）
​		5）求工资的平均值（avg）

```sql
select count(*) from emp;
select max(sal) max_sal from emp;
select min(sal) min_sal from emp;
select sum(sal) sum_sal from emp;
select avg(sal) avg_sal from emp;
```

#### 	1.5 Limit语句

​		典型的查询会返回多行数据。LIMIT子句用于限制返回的行数。
​		hive (default)> select * from emp limit 5;



### 2 Where语句

介绍
1）使用WHERE 子句，将不满足条件的行过滤掉。
2）WHERE 子句紧随 FROM 子句。
3）案例实操
​	查询出薪水大于1000的所有员工

```
select * from emp where sal > 1000;
```



#### 2.1 比较运算符（Between/In/ Is Null）

1）下面表中描述了谓词操作符，这些操作符同样可以用于JOIN…ON和HAVING语句中。

2）案例实操
（1）查询出薪水等于5000的所有员工
（2）查询工资在500到1000的员工信息
（3）查询comm为空的所有员工信息
（4）查询工资是1500和5000的员工信息

```sql
select * from emp where sal =5000;

select * from emp where sal between 500 and 1000;
select * from emp where sal > 500 and sal < 1000;

select * from emp where comm is null;

select * from emp where sal IN (1500, 5000);
select * from emp where sal = 1500 or sal = 5000;
```



#### 2.2 Like和RLike

1）使用LIKE运算选择类似的值
2）选择条件可以包含字符或数字:
​	% 代表零个或多个字符(任意个字符)。
​	_ 代表一个字符。
3）RLIKE子句是Hive中这个功能的一个扩展，其可以通过Java的正则表达式这个更强大的语言来指定匹配条件。
4）案例实操
（1）查找以2开头薪水的员工信息
（2）查找第二个数值为2的薪水的员工信息
（3）查找薪水中含有2的员工信息

```sql
select * from emp where sal LIKE '2%';

select * from emp where sal LIKE '_2%';

select * from emp where sal LIKE '%2%';
```



#### 2.3 逻辑运算符（And/Or/Not）

逻辑运算符（AND_OR_NOT）.png
案例实操
（1）查询薪水大于1000，部门是30
（2）查询薪水大于1000，或者部门是30
（3）查询除了20部门和30部门以外的员工信息

```sql
select * from emp where sal>1000 and deptno=30;

select * from emp where sal>1000 or deptno=30;

select * from emp where deptno not IN(30, 20);
```



### 3 分组

#### 3.1 Group By语句

GROUP BY语句通常会和聚合函数一起使用，按照一个或者多个列队结果进行分组，然后对每个组执行聚合操作。
案例实操：
（1）计算emp表每个部门的平均工资
（2）计算emp每个部门中每个岗位的最高薪水

```
select avg(sal) avg_sal,deptno from emp group by deptno;

select max(sal) max_sal,deptno from emp group by deptno;
```

#### 3.2 Having语句

1）having与where不同点
（1）where针对表中的列发挥作用，查询数据；having针对查询结果中的列发挥作用，筛选数据。
（2）where后面不能写分组函数，而having后面可以使用分组函数。
（3）having只用于group by分组统计语句。
2）案例实操：
（1）求每个部门的平均薪水大于2000的部门

```sql
select avg(sal) avg_sal,deptno from emp group by deptno having avg_sal>2000;
```



### 4 Join语句

#### 4.1 等值Join

Hive支持通常的SQL JOIN语句，但是只支持等值连接，不支持非等值连接。
案例操作
​	（1）根据员工表和部门表中的部门编号相等，查询员工编号、员工名称和部门编号；

```sql
select e.empno,e.ename,d.deptno from emp e join dept d on e.deptno=d.deptno;
```

#### 4.2 表的别名

1）好处
​	（1）使用别名可以简化查询。
​	（2）使用表名前缀可以提高执行效率。
2）案例实操
​	合并员工表和部门表

#### 4.3 内连接

```sql
--只有进行连接的两个表中都存在与连接条件相匹配的数据才会被保留下来。
select e.empno, e.ename, d.deptno from emp e join dept d on e.deptno = d.deptno;
```

#### 4.4 左外连接

```
JOIN操作符左边表中符合WHERE子句的所有记录将会被返回。
select e.empno, e.ename, d.deptno from emp e left join dept d on e.deptno = d.deptno;
```

#### 4.5 右外连接

```
JOIN操作符右边表中符合WHERE子句的所有记录将会被返回。
hive (default)> select e.empno, e.ename, d.deptno from emp e right join dept d on e.deptno = d.deptno;
```

#### 4.6 满外链接

```
将会返回所有表中符合WHERE语句条件的所有记录。如果任一表的指定字段没有符合条件的值的话，那么就使用NULL值替代。
select e.empno, e.ename, d.deptno from emp e full join dept d on e.deptno = d.deptno;
```

#### 4.7 多表连接

​	注意：连接 n个表，至少需要n-1个连接条件。例如：连接三个表，至少需要两个连接条件。
​	0）数据准备
​		location.txt
​	1）创建位置表
​	2）导入数据
​	3）多表连接查询

```sql
create table if not exists default.location(loc int, loc_name string)
row format delimited fields terminated by '\t';

load data local inpath '/opt/module/datas/location.txt' into table default.location;
 
SELECT e.ename, d.deptno, l. loc_name
FROM   emp e 
JOIN   dept d
ON     d.deptno = e.deptno 
JOIN   location l
ON     d.loc = l.loc;
```



#### 4.8 笛卡尔积 JOIN

​	1）笛卡尔集会在下面条件下产生:
​		（1）省略连接条件
​		（2）连接条件无效
​		（3）所有表中的所有行互相连接
​	2）案例实操

```sql
set hive.mapred.mdoe=strict;

select empno, deptno from emp, dept;

FAILED: SemanticException [Error 10052]: In strict mode, cartesian product is not allowed. If you really want to perform the operation, set hive.mapred.mode=nonstrict
```



### 5 排序

#### 5.1 全局排序（Order By）

​	Order By：全局排序，一个MapReduce
​	1）使用 ORDER BY 子句排序
​		ASC（ascend）: 升序（默认）
​		DESC（descend）: 降序
​	2）ORDER BY 子句在SELECT语句的结尾。
​	3）案例实操
​		（1）查询员工信息按工资升序排列
​		（2）查询员工信息按工资降序排列

```
hive (default)> select * from emp order by sal;
FAILED: SemanticException 1:27 In strict mode, if ORDER BY is specified, LIMIT must also be specified. Error encountered near token 'sal'

select * from emp order by sal limit 10;


select * from emp order by sal desc;
```



#### 5.2 按照别名排序

​	按照员工薪水的2倍排序

```
select ename, deptno, sal*2 doublesal from emp order by doublesal ;
```

#### 5.3 多个列排序

​	按照部门和工资升序排序

```
select ename, deptno, sal from emp order by deptno, sal ;
```

#### 5.4 每个MapReduce内部排序（Sort By）

​	Sort By：每个MapReduce内部进行排序，对全局结果集来说不是排序
​	1）设置reduce个数
​	2）查看设置reduce个数
​	3）根据部门降序查看员工信息
​	4）将查询结果导入到文件中（按照部门编号降序排序）

```
set mapreduce.job.rduces=3;
select * from emp sort by deptno desc;

insert [overwrite] [local] directory '/opt/module/datas/sortby-result' ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' COLLECTION ITEMS TERMINATED BY '\n' select * from emp distribute by empno desc;
```

#### 5.5 分区排序（Distribute By）

​	Distribute By：类似MR中partition，进行分区，结合sort by使用。
​	注意，Hive要求DISTRIBUTE BY语句要写在SORT BY语句之前。
​	案例实操：
​		（1）先按照部门编号分区，再按照员工编号降序排序。

```sql
insert overwrite local directory '/opt/module/datas/distributeby-result' ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' COLLECTION ITEMS TERMINATED BY '\n' select * from emp distribute by deptno sort by empno desc;
```

#### 5.6 Cluster By

​	当distribute by和sorts by字段相同时，可以使用cluster by方式。
​	cluster by除了具有distribute by的功能外还兼具sort by的功能。但是排序只能是倒序排序，不能指定排序规则为ASC或者DESC。
​	1）以下两种写法等价
​		select * from emp cluster by deptno;
​		select * from emp distribute by deptno sort by deptno;
​	注意：按照部门编号分区，不一定就是固定死的数值，可以是20号和30号部门分到一个分区里面去。



### 6 分桶及抽样查询

#### 6.1 分桶表数据存储

​	分区针对的是数据的存储路径；分桶针对的是数据文件。
​	分区提供一个隔离数据和优化查询的便利方式。不过，并非所有的数据集都可形成合理的分区，特别是之前所提到过的要确定合适的划分大小这个疑虑。
​	分桶是将数据集分解成更容易管理的若干部分的另一个技术。
​	1）先创建分桶表，通过直接导入数据文件的方式
​		（0）数据准备
​			students.txt
​		（1）创建分桶表
​		（2）查看表结构
​		（3）导入数据到分桶表中
​		（4）查看创建的分桶表中是否分成4个桶
​			发现并没有分成4个桶。是什么原因呢？

```
create table stu_buck(id int, name string)
clustered by(id) 
into 4 buckets
row format delimited fields terminated by '\t';

desc formatted stu_buck;

load data local inpath '/opt/module/datas/students.txt' into table stu_buck;
```



​	2）创建分桶表时，数据通过子查询的方式导入
​		（1）先建一个普通的stu表
​		（2）向普通的stu表中导入数据
​		（3）清空stu_buck表中数据
​		（4）导入数据到分桶表，通过子查询的方式
​		（5）发现还是只有一个分桶
​		（6）需要设置一个属性
​		（7）查询分桶的数据

```sql
create table stu(id int, name string)
row format delimited fields terminated by '\t';

load data local inpath '/opt/module/datas/students.txt' into table stu;

truncate table stu_buck;

insert into table stu_buck
select id, name from stu cluster by(id);

```



#### 6.2 分桶抽样查询

```
select * from stu_buck TABLESAMPLE(bucket 1 out of 4 on id);
```

#### 6.3 数据块抽样

​		Hive提供了另外一种按照百分比进行抽样的方式，这种事基于行数的，按照输入路径下的数据块百分比进行的抽样。
​		hive (default)> select * from stu tablesample(0.1 percent);
​		提示：这种抽样方式不一定适用于所有的文件格式。另外，这种抽样的最小抽样单元是一个HDFS数据块。因此，如果表的数据大小小于普通的块大小128M的话，那么将会返回所有行。

```
select * from stu TABLESAMPLE(0.1 percent);
```





## 函数



## 压缩和存储

```
create table log_text (
    track_time string,
    url string,
    session_id string,
    referer string,
    ip string,
    end_user_id string,
    city_id string
)
row format delimited fields terminated by '\t'
stored as textfile ;

load data local inpath '/opt/module/datas/log.data' into table log_text ;
 
dfs -du -h /user/hive/warehouse/log_text;
```

