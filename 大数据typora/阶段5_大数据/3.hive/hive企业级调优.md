## hive企业级调优

### 1 Fetch抓取

```xml
<property>
    <name>hive.fetch.task.conversion</name>
    <value>more</value>
    <description>
        Expects one of [none, minimal, more].
        Some select queries can be converted to single FETCH task minimizing latency.
        Currently the query should be single sourced not having any subquery and should not have
        any aggregations or distincts (which incurs RS), lateral views and joins.
        0. none : disable hive.fetch.task.conversion
        1. minimal : SELECT STAR, FILTER on partition columns, LIMIT only
        2. more  : SELECT, FILTER, LIMIT only (support TABLESAMPLE and virtual columns)
    </description>
</property>
```

​	Fetch抓取是指，Hive中对某些情况的查询可以不必使用MapReduce计算。例如：SELECT * FROM employees;在这种情况下，Hive可以简单地读取employee对应的存储目录下的文件，然后输出查询结果到控制台。在hive-default.xml.template文件中hive.fetch.task.conversion默认是more，老版本hive默认是minimal，该属性修改为more以后，在全局查找、字段查找、limit查找等都不走mapreduce。



### 2 本地模式

​	大多数的Hadoop Job是需要Hadoop提供的完整的可扩展性来处理大数据集的。不过，有时Hive的输入数据量是非常小的。在这种情况下，为查询触发执行任务时消耗可能会比实际job的执行时间要多的多。对于大多数这种情况，Hive可以通过本地模式在单台机器上处理所有的任务。对于小数据集，执行时间可以明显被缩短。

用户可以通过设置`hive.exec.mode.local.auto`的值为`true`，来让Hive在适当的时候自动启动这个优化。

```
set hive.exec.mode.local.auto=true;  //开启本地mr
//设置local mr的最大输入数据量，当输入数据量小于这个值时采用local  mr的方式，默认为134217728，即128M
set hive.exec.mode.local.auto.inputbytes.max=50000000;
//设置local mr的最大输入文件个数，当输入文件个数小于这个值时采用local mr的方式，默认为4
set hive.exec.mode.local.auto.input.files.max=10;
```



```
hive (default)> set hive.fetch.task.conversion;
hive.fetch.task.conversion=more
hive (default)> set hive.exec.mode.local.auto;
hive.exec.mode.local.auto=false
hive (default)> select * from emp cluster by deptno;
...
Time taken: 41.8 seconds, Fetched: 14 row(s)

hive (default)> set hive.exec.mode.local.auto=true;
hive (default)> select * from emp cluster by deptno;
...
Time taken: 3.251 seconds, Fetched: 14 row(s)
```



------



### 3 表的优化(重点)

#### 3.1 小表、大表Join

​	小表jion大表

#### 3.2 大表Join大表

1）空KEY过滤

​	有时join超时是因为某些key对应的数据太多，而相同key对应的数据都会发送到相同的reducer上，从而导致内存不够。此时我们应该仔细分析这些异常的key，很多情况下，这些key对应的数据是异常数据，我们需要在SQL语句中进行过滤。

```
测试不过滤空id
hive (default)> insert overwrite table jointable 
				select n.* from nullidtable n left join ori o on n.id = o.id;

测试过滤空id
hive (default)> insert overwrite table jointable 
				select n.* from (select * from nullidtable where id is not null ) n  left join ori o on n.id = o.id;

```



2）空key转换

有时虽然某个key为空对应的数据很多，但是相应的数据不是异常数据，必须要包含在join的结果中，此时我们可以表a中key为空的字段赋一个随机的值，使得数据随机均匀地分不到不同的reducer上。

```sql
set mapreduce.job.reduces = 5;

insert overwrite table jointable
select n.* from nullidtable n full join ori o on 
case when n.id is null then concat('hive', rand()) else n.id end = o.id;
```





#### 3.3 MapJoin

​	如果不指定MapJoin或者不符合MapJoin的条件，那么Hive解析器会将Join操作转换成Common Join，即：在Reduce阶段完成join。容易发生数据倾斜。可以用MapJoin把小表全部加载到内存在map端进行join，避免reducer处理。

1）开启MapJoin参数设置：

（1）设置自动选择Mapjoin

set hive.auto.convert.join = true; 默认为true

（2）大表小表的阀值设置（默认25M一下认为是小表）：

set hive.mapjoin.smalltable.filesize=25000000;

2）MapJoin工作机制

如果不指定MapJoin或者不符合MapJoin的条件，那么Hive解析器会将Join操作转换成Common Join，即：在Reduce阶段完成join。容易发生数据倾斜。可以用MapJoin把小表全部加载到内存在map端进行join，避免reducer处理。

1）开启MapJoin参数设置：

（1）设置自动选择Mapjoin

set hive.auto.convert.join = true; 默认为true

（2）大表小表的阀值设置（默认25M一下认为是小表）：

set hive.mapjoin.smalltable.filesize=25000000;

2）MapJoin工作机制

![Hive MapJoin](..\图片\clip_image002.jpg)

首先是Task A，它是一个Local Task（在客户端本地执行的Task），负责扫描小表b的数据，将其转换成一个HashTable的数据结构，并写入本地的文件中，之后将该文件加载到DistributeCache中。

接下来是Task B，该任务是一个没有Reduce的MR，启动MapTasks扫描大表a,在Map阶段，根据a的每一条记录去和DistributeCache中b表对应的HashTable关联，并直接输出结果。

由于MapJoin没有Reduce，所以由Map直接输出结果文件，有多少个Map Task，就有多少个结果文件。



#### 3.4 Group By

​	默认情况下，Map阶段同一Key数据分发给一个reduce，当一个key数据过大时就倾斜了。

​    并不是所有的聚合操作都需要在Reduce端完成，很多聚合操作都可以先在Map端进行部分聚合，最后在Reduce端得出最终结果。

1）开启Map端聚合参数设置

（1）是否在Map端进行聚合，默认为True

hive.map.aggr = true

（2）在Map端进行聚合操作的条目数目

​    hive.groupby.mapaggr.checkinterval = 100000

（3）有数据倾斜的时候进行负载均衡（默认是false）

​    hive.groupby.skewindata = true

​    当选项设定为 true，生成的查询计划会有两个MR Job。第一个MR Job中，Map的输出结果会随机分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样处理的结果是相同的Group By Key有可能被分发到不同的Reduce中，从而达到负载均衡的目的；第二个MR Job再根据预处理的数据结果按照Group By Key分布到Reduce中（这个过程可以保证相同的Group By Key被分布到同一个Reduce中），最后完成最终的聚合操作。

#### 3.5 Count(Distinct) 去重统计

数据量小的时候无所谓，数据量大的情况下，由于COUNT DISTINCT操作需要用一个Reduce Task来完成，这一个Reduce需要处理的数据量太大，就会导致整个Job很难完成，一般COUNT DISTINCT使用先GROUP BY再COUNT的方式替换：

```
select count(id) from (select id from bigtable group by id) a;
```

#### 3.6 笛卡尔积

避免

#### 3.7 行列过滤

列处理：在SELECT中，只拿需要的列，如果有，尽量使用分区过滤，少用SELECT *。

行处理：在分区剪裁中，当使用外关联时，如果将副表的过滤条件写在Where后面，那么就会先全表关联，之后再过滤

```shell
#（1）测试先关联两张表，再用where条件过滤
hive (default)> select o.id from bigtable b
join ori o on o.id = b.id
where o.id <= 10;
#Time taken: 34.406 seconds, Fetched: 100 row(s)
#Time taken: 26.043 seconds, Fetched: 100 row(s)

#（2）通过子查询后，再关联表
hive (default)> select b.id from bigtable b
join (select id from ori where id <= 10 ) o on b.id = o.id;
#Time taken: 30.058 seconds, Fetched: 100 row(s)
#Time taken: 29.106 seconds, Fetched: 100 row(s)

```



#### 3.8 动态分区调整

​	尚硅谷大数据技术之动态分区调整.doc

#### 3.9 分桶

#### 3.10 分区





------



### 4 数据倾斜

​		4.1 Map数
​		4.2 小文件进行合并
​		4.3 复杂文件增加Map数
​		4.4 Reduce数
5 并行执行
​	尚硅谷大数据技术之并行执行.doc
6 严格模式
​	尚硅谷大数据技术之严格模式.doc
7 JVM重用
​	尚硅谷大数据技术之JVM重用.doc
8 推测执行
​	尚硅谷大数据技术之推测执行.doc
9 压缩
10 执行计划（Explain）

