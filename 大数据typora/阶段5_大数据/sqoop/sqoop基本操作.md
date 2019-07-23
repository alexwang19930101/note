### 1.安装与修改配置

```shell
#export HADOOP_COMMON_HOME=
export HADOOP_COMMON_HOME=/opt/module/hadoop-2.7.2
#Set path to where hadoop-*-core.jar is available
#export HADOOP_MAPRED_HOME=
export HADOOP_MAPRED_HOME=/opt/module/hadoop-2.7.2
#set the path to where bin/hbase is available
#export HBASE_HOME=

#Set the path to where bin/hive is available
#export HIVE_HOME=
export HIVE_HOME=/opt/module/hive
#Set the path for where zookeper config dir is
#export ZOOCFGDIR=
export ZOOCFGDIR=/opt/module/zookeeper-3.4.10/conf
export ZOOKEEPER_HOME=/opt/module/zookeeper-3.4.10
```

拷贝mysql连接驱动

验证Sqoop

```
bin/sqoop help
```

验证连接

```sh
bin/sqoop list-databases --connect jdbc:mysql://hadoop102:3306/ --username root --password 000000
```



### 2.数据导入导出

**RDBMS-->HDFS**

#### 2.1数据全部导入

```shell
bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--table staff \
--target-dir /user/company \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t";
```

import:导入到hdfs

connect：连接协议、ip、端口、库名

tabel：表名

target-dir：hdfs目标目录

delete-target-dir：存在目标目录时删除

#### 2.2查询导入

比上面多了`--query 'select name,sex from staff where id <=1 and $CONDITIONS;'`

```
bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--target-dir /user/company \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t" \
--query 'select name,sex from staff where id <=1 and $CONDITIONS;'
```

`$CONDITIONS`：在查询时必须带上，不可以省略。它是用来标记map任务读取数据的位置

如果 query 后使用的是双引号，则$CONDITIONS 前必须加转移符`\$CONDITIONS` ，防止 shell识别为自己的变量。

--query 选项，不能同时与--table 选项使用

#### 2.3导入指定列

```
bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--target-dir /user/company \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t" \
--columns id,sex \
--table staff
```

columns 中如果涉及到多列，用**逗号分隔**，分隔时**不要添加空格**

```
 bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--target-dir /user/company \
--delete-target-dir \
--num-mappers 1 \
--fields-terminated-by "\t" \
--table staff \
--where "id=1"
```

sqoop配置文件中注释掉的东西就是没有这个参数了

在 Sqoop 中可以使用 sqoop import -D property.name=property.value 这样的方式加
入执行任务的参数，多个参数用空格隔开。

**RDBMS-->HIVE**

```shell
 bin/sqoop import \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--table staff \
--num-mappers 1 \
--hive-import \
--fields-terminated-by "\t" \
--hive-overwrite \
--hive-table staff_hive
```

--hive-overwrite 覆盖表数据

--hive-table 指定表

该过程分为两步，第一步将数据导入到 HDFS，第二步将导入到 HDFS 的数据迁移到 Hive 仓库，表不存在会自动创建

#### 2.4数据导出

```shell
 bin/sqoop export \
--connect jdbc:mysql://hadoop102:3306/company \
--username root \
--password 000000 \
--table staff \
--num-mappers 1 \
--export-dir /user/hive/warehouse/staff_hive \
--input-fields-terminated-by "\t"
```

Mysql 中如果表不存在，不会自动创建



#### 2.6脚本打包

```
$ vi opt/job_HDFS2RDBMS.opt

export
--connect
jdbc:mysql://linux01:3306/company
--username
root
--password
123456
--table
staff
--num-mappers
1
--export-dir
/user/hive/warehouse/staff_hive
--input-fields-terminated-by
"\t"
```

注意：必须一个参数一行

```
bin/sqoop --options-file opt/job_HDFS2RDBMS.opt
```





3.公用参数：数据库连接
序号  参数  说明
1  --connect  连接关系型数据库的 URL
2  --connection-manager  指定要使用的连接管理类
3  --driver  JDBC 的 driver class
4  --help  打印帮助信息
5  --password  连接数据库的密码
6  --username  连接数据库的用户名
7  --verbose  在控制台打印出详细信息



import
序号  参数  说明
1  --enclosed-by <char>  给字段值前后加上指定的字
符
2  --escaped-by <char>  对字段中的双引号加转义符
3  --fields-terminated-by <char>  设定每个字段是以什么符号作为结束，默认为逗号

4  --lines-terminated-by <char>  设定每行记录之间的分隔符，默认是\n
5  --mysql-delimiters  Mysql 默认的分隔符设置，字段之间以逗号分隔，行之间以\n 分隔，默认转义符是\，字段值以单引号包裹。
6  --optionally-enclosed-by <char> 给带有双引号或单引号的字段值前后加上指定字符。