---
title: hive动态分区
date: 2017/1/3 16:24:26 
tags: Hive
categories: 大数据

---
实际使用中需要对处理后的数据按照时间分开，后续有按照时间的插入和删除操作，所以用hive的分区表是一个很好的解决方案。
比较蛋疼的是，由于 hive 不支持使用 load 语句进行动态分区插入数据，所以要新建一个表，再用 insert 语句把表中数据导入到新建的分区表中。

<!-- more -->

## 测试数据内容
```
103646	铁血皇城	3	2016-04-19	2016-04-19
104046	铁骑三国	39	2016-04-19	2016-04-19
1041	九阴绝学	2	2016-03-26	2016-03-26
104238	传奇盛世	3	2016-04-28	2016-04-28
104928	天问	1	2016-02-27	2016-02-27
106417	神曲	2	2016-04-15	2016-04-15
10883	灵域	1	2016-04-15	2016-04-15
```

*由于后续还需要使用分区后的数据进行MapReduce操作，所以在后面复制了一段时间字段（分区后，分区字段会变成hive中的文件夹名）*

## 创建表并导入数据
```
create table IGCT(id STRING,game STRING,count INT,time STRING,addtime  STRING)
row format delimited
fields terminated by '\t'
stored as textfile;

LOAD DATA INPATH '/recommend/hive/partitionTest' INTO TABLE IGCT_static;
```
## 建立分区表并导入数据
```
set hive.exec.dynamic.partition.mode=nonstrict; 
set hive.exec.dynamic.partition=true;

CREATE  TABLE IGCT_dynamic(id STRING,game STRING,count INT,time STRING)                                                                            
partitioned by(addtime  STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE


insert overwrite table IGCT_dynamic partition(addtime)      
select * from   IGCT;     
````

## 查看数据
```
hive> select * from IGCT_dynamic;
OK
104928	天问	1	2016-02-27	2016-02-27
1041	九阴绝学	2	2016-03-26	2016-03-26
106417	神曲	2	2016-04-15	2016-04-15
10883	灵域	1	2016-04-15	2016-04-15
103646	铁血皇城	3	2016-04-19	2016-04-19
104046	铁骑三国	39	2016-04-19	2016-04-19
104238	传奇盛世	3	2016-04-28	2016-04-28
Time taken: 0.054 seconds, Fetched: 7 row(s)

```


## 删除分区
```
ALTER TABLE IGCT_dynamic DROP IF EXISTS PARTITION (addtime='2016-02-27');
```

## 查看数据
```
hive> select * from IGCT_dynamic2;
OK
1041	九阴绝学	2	2016-03-26	2016-03-26
106417	神曲	2	2016-04-15	2016-04-15
10883	灵域	1	2016-04-15	2016-04-15
103646	铁血皇城	3	2016-04-19	2016-04-19
104046	铁骑三国	39	2016-04-19	2016-04-19
104238	传奇盛世	3	2016-04-28	2016-04-28
Time taken: 0.062 seconds, Fetched: 6 row(s)

```
