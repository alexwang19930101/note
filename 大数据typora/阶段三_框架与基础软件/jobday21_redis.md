## redis

![1537684876286](../图片/1537684876286.png)

### 1.redis引入

为什么要redis？

![1537685668710](../图片/1537685668710.png)

​	随着数据量海量增加，所有互联网平台都面临挑战。

##### 解决服务器的CPU和内存的压力,负载均衡

解决方案：

1.存在cookie中

- 不安全
- 网络负担加重

2.文件服务器或者数据库

- 大量IO操作性能不好

3.session复制(tomcat等的官方解决方案，广播到整个集群)

- 节点数据冗余
- 节点越多浪费越大

4.缓存数据库

- 完全存在内存中速度快，
- 数据结构简单

##### 解决IO压力

基本读写比为：8:2

利用缓存解决读压力，可以对数据库本身进行优化

![1537686106716](../图片/1537686106716.png)

### 2.NoSQL介绍

not only sql（不仅仅是sql）泛指非关系型数据库

NoSQL不依赖业务逻辑方式存储，而是以简单的**key-value**模式存储，大大增加了数据库的扩展能力(用key-json存，随便加字段)

不支持ACID，不遵循SQL标准

远超SQL的性能 

#### 2.1NoSQL使用场景

- 对数据库并发的读写
- 海量数据的读写
- 对数据的高扩展性

#### 2.2NoSQL不适合场景

- 需要事务支持
- 基于sql的结构化查询存储，处理复杂关系，需要即席查询



------

#### 3 各种NoSQL

**Memcached**

- 数据都在内存中，不支持持久化
- 支持简单的key-value
- 一般作为缓存数据库辅助持久化的数据库

**Redis**

- 覆盖Memcached的绝大多数功能
- 数据存在内存中，支持持久化，主要是备份恢复
- 支持key-value，还是有其他多种数据结构list、set、hash、zset等
- 一般是作为缓存数据库辅助持久化的数据库

**mongoDB**

- 高性能、开源、shema free的文档型数据库
- 数据都在内存中，内存不足时，不常用的保存到硬盘中
- 为key-value(尤其是json)提供了丰富的查询功能
- 支持二进制数据及大型数据库
- 可以根据数据特点替代RDBMS，成为独立的数据库。或者配合RDBMS存储特定数据

**列式数据库**

![1537687741207](../图片/1537687741207.png)

列式数据库 --> OLAP 分析型处理

行式数据库 --> OLTP 事务性处理

#### HBASE

Hadoop中的数据库，对大量数据进行随机、实时的书写场景。可以处理庞大的表，使普通计算机具备处理超过billion级的行数据，而已处理数百万的列元素的数据表。

##### Cassandra

apache开源NoSQL使用庞大集群处理海量数据（PB）



------

### 4.Redis应用场景

高频次，热门访问数据，降低数据库IO

分布式架构做session共享

**多种功能场景**

![1537689542115](../图片/1537689542115.png)

#### 4.1redis安装

```shell
yum install redis
vim /etc/redis.conf
systemctl start redis
redis-cli
```

**二进制安装**

redis依赖gcc

1.make disclean再make 

2.make install

安装在了/usr/local/redis

3.复制redis.conf到上面的目录

##### redis常用命令

```shell
#redis默认有16个库0-15，其中默认连接0，切换使用select
select 1

#所有库使用同一密码管理

#查询匹配的key  key [pattern] 
key *
#判断key是否存在
exists k1
#删除key
del key
#时效性key  expire
expire k1 10
#ttl <key>判断key过期时间,-1代表永不过期，-2代表已经过期
ttl k1
#查看当前数据库的key的数量
dbsize
#清空当前和清空所有
flushdb
flushall
```

##### redis单线程+多路IO复用

单线程  --> 多线程+锁(memcached) --> 单线程+多路IO复用(redis)

多路io复用见引用

> https://www.cnblogs.com/aspirant/p/6877350.html?utm_source=itdadao&utm_medium=referral



------

### **5.Redis五大数据类型**

#### 5.1 String

介绍：

- String是二进制安全的，可以操作任何数据，比如jpg图片或者序列化对象
- String是Redis最基本的类型，一个Redis中字符串value最多512M

```shell
get <key>	#查询
set <key> <value> #设置或是覆盖
append <key> <value> #给定的<value>追加都原值的末尾
strlen <key> #获得值的长度
setnx <key> <value> #只有key不存在的时候才设置key值
incr <key>	+1，#只能操作数字或数值字符串，如果不存在设置为1
decr <key>	-1

mset <key1> <value1> <key2> <value2>...	#多设置
mget <key1> <key2> <key3>	#多获取
msetnx <key1> <value1> <key2> <value2>...	#（原子性）
getrange <key> <start> <end>	#包括头也包括尾
setrange <key> <start> <value>	#覆盖一段
setex <key> <过期时间>	#设置一个带过期时间的key（原子性）
getset <key> <value>	#以新换久，返回新
```



##### Redis的原子性

原子操作是指线程不会被调度机制打断，操作一旦开始就一直运行到结束。中间不会有context switch。

（1）在单线程中单条指令操作可以认为是原子性的

（2）多线程中，不能被其他进程（线程）打断的操作就叫原子操作

Redis的原子性得益于Redis是单线程+多路IO复用



#### 5.2 List

单键多列

![1538015256276](../图片/1538015256276.png)

List是一个双端队列，一个key代表的队列中放入多个value

```shell
lpush/rpush <key> <value1> <value2> <value3>
lpop/rpop <key>
rpoplpush <key1> <key2>	#一个列表右出到另一个列表的左（原子性）

lrange <key> <start> <end>	#end如果是-1表明取全集
lindex <key> <indext>
llen <key>
linsert l2 <before|after> <pivot> <value>
lrem <key> <n> <value>
```



#### 5.3 Hash

hash是一个键值对集合，适合存放值对象

![1538018326922](../图片/1538018326922.png)

存放值对象的问题：

1. json，每次都要序列化，读取要反序列化，改部分却要操作整体
2. custemer:01:name 对于string进行分割，不凡便维护
3. hash

```shell
hset <key> <field> <value>							#增加/赋值
hget <key> <field>									#查询field
hmset <key1> <field> <value> <key1> <field> <value>	#多增加/多设置
hsetnx <key> <field> <value>						#增加field-value
hexists <key> <field>								#field存在？
hkeys <key>											#获取key的field
hvals <key>											#获取key的val
hincrby <key> <field> <增加量>						  #field增加一个数量
```



#### 5.4 Set

无序不重复

```shell
sadd <key> <value1> <value2>	#增加
smembers <key>					#查询
sismenber <key> <value>			#查询是否有该元素

scard <key>						#返回集合元素个数
srem <key> <value1> <value2>	#删除某些元素
spop <key>						#随机取一个(随机是指不按存入顺序)		
srandmember						#随机取n个值

#集合操作
sinter <key1> <key2>			#返回两个的交集
sunion <key1> <key2>			#返回两个的并集
sdiff  <key1> <key2>			#此差集是key1 - key1&key2
```

#### 5.5 ZSet/SortedSet

 一个按score排序的Set

![1538019337112](../图片/1538019337112.png)

```shell
zadd <key> <score1> <value1> <score2> <value2>		#增加
zrange <key> <start> <stop> [withscores]		#获取一段,可以带score
zrangebyscore <key> <min> <max> [withscores]	#【min，max】由小->大
zrerangebycore <key> <min> <max> [withscores]	#【min，max】由大->小
zincrby <key> <increment> <value>				#为元素score增加
zrem <key> <value>								#移除
zcount <key> <min> <max>						#【min，max】元素个数
zrank <key> <value>								#一个元素的排名(0开始)
```

### 6.Redis配置文件

```shell
#1.ip绑定
bind 
#2.保护模式
protected-mod
#3.端口
port
#4.连接超时时间
timeout
#5.tcp存活时长，发心跳
tcp-keepalive
#6.后台运行
daemonize
#7.进程文件
pidfile
#8.password
requirepass
#9.最大连接数
maxclient
#10.最大memory
maxmemory
#11.淘汰策略
# volatile-lru -> remove the key with an expire set using an LRU algorithm
# allkeys-lru -> remove any key according to the LRU algorithm
# volatile-random -> remove a random key with an expire set
# allkeys-random -> remove a random key, any key
# volatile-ttl -> remove the key with the nearest expire time (minor TTL)
# noeviction -> don't expire at all, just return an error on write operations
maxmemory-policy
#12.丢弃选取样本数
maxmemory-samples
```



------

### 7.Redis事务

Redis事务是一个单独的隔离操作：事务的所有命令都会序列化按顺序来执行，事务在执行过程中，不会被其他客户端发的命令请求打断。

Redis事务是串联多个命令防止别的命令插队

**步骤**

1. Redis通过Multi命令开始将接下来的命令放入命令队列，但是并不会执行，
   - 命令会返回queue

2. 到输入Exec后Redis才会将之前的命令队列的命令依次执行。

组队过程可以由discard来放弃

![1538035268125](../图片/1538035268125.png)

事务的两种错误：

1. **组队阶段报错**

   事务命令入队时发现错误，整个事务执行时被取消

![1538035603518](../图片/1538035603518.png)

2. 执行阶段命令报错，除了本条以外其他正常执行，不会回滚。
  ![1538035876773](../图片/1538035876773.png)

#### 锁机制(悲观锁和乐观锁)

![1538036111008](../图片/1538036111008.png)

**悲观锁**（pessimistic lock）获取资源时认为会被并发修改，直接对数据上锁，其他线程向获取该数据资源就会block直到上线程释放资源。传统关系型数据库都是这种锁机制，表锁、行锁、读锁、写锁，都是在操作之前上锁

**乐观锁**CAS又称无锁，实际上它是在更新时判断其他线程有没有更新这个数据，可以使用版本号等机制。乐观锁适合多读的场景，可以提高并发性能。Redis就是通过CAS来实现事务的。

#### Redis锁机制

Redis就是通过CAS来实现事务的。

在multi开始事务组队之前先`watch <key1> [<key2> <key3>]`若执行前这些key被其他命令修改，那么事务将会被打断。

可以通过unwatch取消对所有key的监视。exec、discard不用显示uncatch



Redis事务的几个特性：

1. 单独的隔离操作
   - 事务所有命令会序列化按顺序执行，事务在执行过程中不会被打断
2. 没有隔离级别
   - 队列中的命令没有exec不会执行，就不存在事务中查询和事务外查询的问题
3. 不保证原子性
   - Redis如果执行过程中有命令失败，其他命令正常执行，没有回滚



------

### 8.Redis持久化

#### 8.1RBD

​	Redis DataBase会在指定的时间间隔内将数据集快照写入磁盘，也就是做Snapshot快照。恢复时是将快照文件读到内存中。

​	Redis单独创建(fork)一个子进程，将数据写入临时文件，在持久化结束后再用临时文件替换上次持久化好的文件。这种方式是一个全盘备份。主进程不进行IO操作确保极高的性能，在大规模备份且对数据完整性不敏感的情况下，RDB比AOF要更加高效。但是最后一次备份到下一次备份之间的数据可能会丢失。

> fork()会为父进程产生一个相同的子进程，子进程通过写时复制与父进程共享一段物理内存，只有在进程空间的内容要发生改变时才会将父进程的内容复制一份给子进程。

关于RBD的配置在SNAPSHOTING中

```shell
################################ SNAPSHOTTING  ################################
#
# Save the DB on disk:
#
#   save <seconds> <changes>
#
#   Will save the DB if both the given number of seconds and the given
#   number of write operations against the DB occurred.
#
#   In the example below the behaviour will be to save:
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed
#
#   Note: you can disable saving completely by commenting out all "save" lines.
#
#   It is also possible to remove all the previously configured save
#   points by adding a save directive with a single empty string argument
#   like in the following example:
#	
#   save ""

save 900 1
save 300 10
save 60 10000

# By default Redis will stop accepting writes if RDB snapshots are enabled
# (at least one save point) and the latest background save failed.
# This will make the user aware (in a hard way) that data is not persisting
# on disk properly, otherwise chances are that no one will notice and some
# disaster will happen.
#
# If the background saving process will start working again Redis will
# automatically allow writes again.
#
# However if you have setup your proper monitoring of the Redis server
# and persistence, you may want to disable this feature so that Redis will
# continue to work as usual even if there are problems with disk,
# permissions, and so forth.
stop-writes-on-bgsave-error yes

# Compress string objects using LZF when dump .rdb databases?
# For default that's set to 'yes' as it's almost always a win.
# If you want to save some CPU in the saving child set it to 'no' but
# the dataset will likely be bigger if you have compressible values or keys.
rdbcompression yes

# Since version 5 of RDB a CRC64 checksum is placed at the end of the file.
# This makes the format more resistant to corruption but there is a performance
# hit to pay (around 10%) when saving and loading RDB files, so you can disable it
# for maximum performances.
#
# RDB files created with checksum disabled have a checksum of zero that will
# tell the loading code to skip the check.
rdbchecksum yes

# rdb快照文件的名称
dbfilename dump.rdb

# rdb文件存放的路径
dir /var/lib/redis
```

手动存盘save,会在主进程执行，阻塞其他进程

| 优点         | 缺点                                     |
| ------------ | ---------------------------------------- |
| 节省磁盘空间 | fork虽然用了写时复制数据大时还是消耗性能 |
| 恢复快       | 备份间隔中的数据可能丢失                 |



#### 8.2AOF

aof和rbd不一致以aof为准。

以日志的形式来记录每个写操作。只允许追加文件不但可以改写文件，redis启动之初读文件重构数据。

AOF默认不开启，需要手动在配置文件中配置

```shell

############################## APPEND ONLY MODE ###############################
#是否开启aof
appendonly no
#aof文件名
appendfilename "appendonly.aof"
#aof同步方式
# appendfsync always
appendfsync everysec
# appendfsync no
```

AOF采用文件追加的方式来写文件当文件过大超过设定阈值，会启动AOF文件内容压缩，只保留恢复数据的最小指令集，可以使用命令bgrewriteaof。

首先fork一个新进程，将文件重写如一个临时文件，遍历新进程的内存中数据每一条一个set语句。

| 优点                                                      | 缺点                                                |
| --------------------------------------------------------- | --------------------------------------------------- |
| 备注稳定，丢失数据更低(appendfsync everysec,丢失缓存中1s) | 比RDB占用更多的磁盘空间                             |
| 可读日志文本，可以处理误操作                              | 恢复速度慢                                          |
|                                                           | 每次读写都同步，性能压力比较大                      |
|                                                           | 官方生成在高频写入的情况下存在一点的bug造成不能恢复 |

- 在数据不敏感的情况下，可以单独用RDB

- 不建议单独用AOF，存在bug

- 如果纯内存缓存可以都不用