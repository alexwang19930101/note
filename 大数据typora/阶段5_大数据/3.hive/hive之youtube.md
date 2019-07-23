## Youtube练习

### 1.hive复习

#### 行转列

**文件**：[./file/person_info.tsv](./file/person_info.tsv)

`concat`			拼接字符串

`concat_ws`		拼接集合中的元素

`collect_set`		将group by后的字段转换成集合

1.1建表--导入数据

```
create table person_info(
	name string,
	constellaction string,
	blood_type string
)
rows format delimited 
fields terminated by "\t";
```

1.2查询

同血型的人

```sql
select 
	blood_type,concat_ws("|",collect_set(name)) 
from 
	person_info 
group by 
	blood_type;
```

同血型同星座的人

```sql
select
    t1.base,
    concat_ws('|', collect_set(t1.name)) name
from
    (select
        name,
        concat(constellation, ",", blood_type) base
from
     person_info) t1
group by
	t1.base;
```



#### 列转行

**文件**：[./file/moive.tsv](./file/moive.tsv)

**`array<T>`**		hive中的数据类型，集合

**`lateral`** 侧写	**`explode`** 将集合分离

**table** lateral view explode(category) **table_tmp** as **tabel_col_name**		语法固定改动两个表名并且命名列名

1.1建表--导入数据

```sql
create table movie_info(
    movie string,
    category array<string>
)
row format delimited fields terminated by "\t"
collection items terminated by ",";

load data local inpath "movie_info.tsv" into table movie_info;
```

1.2查询

```sql
select
    movie,
    category_name
from
	movie_info lateral view explode(category) table_tmp as category_name;
```



**fields terminated by**：字段与字段之间的分隔符。
**collection items terminated by**：一个字段中各个子元素 item的分隔符。



## YouTube

**视频表**

字段  备注  详细描述
video id  视频唯一 id  11 位字符串
uploader  视频上传者  上传视频的用户名 String
age  视频年龄  视频上传日期和 2007 年 2 月
15 日之间的整数天（Youtube
的独特设定）
category  视频类别  上传视频指定的视频分类
length  视频长度  整形数字标识的视频长度
views  观看次数  视频被浏览的次数
rate  视频评分  满分 5 分
ratings  流量  视频的流量，整型数字
conments  评论数  一个视频的整数评论数
related ids  相关视频 id  相关视频的 id，最多 20 个



**用户表**

字段  备注  字段类型
uploader  上传者用户名  string
videos  上传视频数  int
friends  朋友数量  int



#### MR进行ETL

创建表

```sql
create table youtube_ori(
videoId string,
uploader string,
age int,
category array<string>,
length int,
views int,
rate float,
ratings int,
comments int,
relatedId array<string>)
row format delimited
fields terminated by "\t"
collection items terminated by "&"
stored as textfile;

create table youtube_user_ori(
uploader string,
videos int,
friends int)
clustered by (uploader) into 24 buckets
row format delimited
fields terminated by "\t"
stored as textfile;
```

然后把原始数据插入到 orc 表中

```sql
create table youtube_orc(
videoId string,
uploader string,
age int,
category array<string>,
length int,
views int,
rate float,
ratings int,
comments int,
relatedId array<string>)
clustered by (uploader) into 8 buckets
row format delimited fields terminated by "\t"
collection items terminated by "&"
stored as orc;

create table youtube_user_orc(
uploader string,
videos int,
friends int)
clustered by (uploader) into 24 buckets
row format delimited
fields terminated by "\t"
stored as orc;
```

加载数据

```sql
load data inpath "/youtube/output/video/2008/0222" into table youtube_ori;
load data inpath "/youtube/user/2008/0903" into table youtube_user_ori;
```

```sql
insert into table youtube_orc select * from youtube_ori;
insert into table youtube_user_orc select * from youtube_user_ori;
```



#### 1.统计视频观看数 Top10

```sql
select
    videoId,
    uploader,
    age,
    category,
    length,
    views,
    rate,
    ratings,
    comments
from
	youtube_orc
order by
	views
desc limit 10;
```

#### 2.统计视频类别热度 Top10

```sql
select
	category_name as category,
	count(t1.videoId) as hot
from(
	select
		videoId,category_name
    from
		youtube_orc lateral view explode(category) t_category as category_name) t1
group by
	t1.category_name
order by
	hot
desc limit 10;
```

#### 3.统计出视频观看数最高的 20  个视频的所属类别以及类别包含这 Top20  视频的个数

1) 先找到观看数最高的 20 个视频所属条目的所有信息，降序排列
2) 把这 20 条信息中的 category 分裂出来(列转行)
3) 最后查询视频分类名称和该分类下有多少个 Top20 的视频

```sql
select * from youtube_orc order by views desc limit 20;
```

```sql
select * from (
select * from youtube_orc order by views desc limit 20
) t1 lateral view explode(category) t_category as category_name;
```

```sql
select category_name as category,count(t2.videoId) as hot_with_views
from
(select * from (
select * from youtube_orc order by views desc limit 20
) t1 lateral view explode(category) t_category as category_name) t2
group by category_name
order by hot_with_views desc;
```



#### 4.统计视频观看数 Top50 所关联视频的所属类别的热度排名

1) 查询出观看数最多的前 50 个视频的所有信息(当然包含了每个视频对应的关联视频)，记为临时表 t1

```sql
select relatedId from youtube_orc order by views desc limit 50; 
```

2) 将找到的 50 条视频信息的相关视频 relatedId 列转行，记为临时表 t2

```sql
select videoId from t1 lateral view explode(relatedId) explode_relatedid as videoId;
```

3) 将相关视频的 id 和 youtube_orc 表进行 inner join 操作

```sql
(
select distinct(t2.videoId),t3.category from t2 inner join youtube_orc t3 on t2.videoId=t3.videoId
) t4 lateral view explode(category) t_category as category_name;
```

4) 按照视频类别进行分组，统计每组视频个数，然后排行

```sql
select count(videoId) as hot,category_name from t5 group by category_name order by hot desc;
```

```sql
select category_name as category, count(t5.videoId) as hot from (
    select videoId, category_name from (
        select distinct(t2.videoId),t3.category from (
            select explode(relatedId) as videoId from (
                select relatedId from youtube_orc order by views desc limit 50) t1
        	) t2
        inner join youtube_orc t3 on t2.videoId = t3.videoId
    ) t4 lateral view explode(category) t_catetory as category_name) t5
group by category_name
order by hot desc;
```

#### 5.统计每个类别中的视频热度 Top10 ，以 Music 

```sql
create table youtube_category(
    videoId string,
    uploader string,
    age int,
    categoryId string,
    length int,
    views int,
    rate float,
    ratings int,
    comments int,
    relatedId array<string>)
row format delimited
fields terminated by "\t"
collection items terminated by "&"
stored as orc;


insert into table youtube_category
    select
        videoId,
        uploader,
        age,
        categoryId,
        length,
        views,
        rate,
        ratings,
        comments,
        relatedId
from
	youtube_orc lateral view explode(category) catetory as categoryId;
```

```sql
select 
	videoId,views 
from 
	youtube_category t1 
inner join 
	(select distinct(categoryId) as categoryId from youtube_category) t2 on 			t1.categoryId = t2.categoryId 
order by views 
desc limit 10;


select videoId,views,t2.categoryId from youtube_category t1 inner join (select distinct(categoryId) as categoryId from youtube_category) t2 on t1.categoryId = t2.categoryId distribute by t2.categoryId sort by views desc limit 10;
```

#### 6.统计上传视频最多的用户Top10以及他们上传的观看次数在前20的视频

```sql
select t2.videoId, t2.views, t2.ratings, t1.videos, t1.friends
from (
select * from youtube_user_orc order by videos desc limit 10) t1
join youtube_orc t2
on t1.uploader = t2.uploader
order by views desc;


select t2.videoId, t2.views, t2.ratings, t2.uploader, t1.videos, t1.friends
from (
select * from youtube_user_orc order by videos desc limit 10) t1
join youtube_orc t2
on t1.uploader = t2.uploader
distribute by t2.uploader
sort by views desc;
```

#### 7.统计每数 个类别视频观看数 Top10

1) 先得到 categoryId 展开的表数据

```
youtube_category
```

2) 子查询按照 categoryId 进行分区，然后分区内排序，并生成递增数字，该递增数字这一列起名为 rank 列

```sql
select
videoId,
categoryId,
views,
row_number() over(partition by categoryId order by views desc) rank from
youtube_category
```

3) 通过子查询产生的临时表，查询 rank 值小于等于 10 的数据行即可

```sql
select t1.* from (
select videoId, categoryId, views, row_number() over(partition by categoryId order by views desc) rank from youtube_category) t1
where rank <= 10;


select videoId, categoryId, views, row_number() over(partition by categoryId order by views desc) rank from youtube_category where rank <= 10;
```

