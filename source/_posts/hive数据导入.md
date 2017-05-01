---
title: hive数据查询导出
date: 2016/11/26 22:17:43 
tags: Hive
categories: 大数据

---

### hive数据查询导出

```
insert overwrite directory '/user/hdu/recommend/gameRecommendNew4/test11.26/gameprestep1'
row format delimited
fields terminated by '\t'
SELECT userid , gamename , COUNT(*) AS count , MAX(gamestarttime) AS lasttime
FROM userdetailtwo
GROUP BY userid , gamename
```
<!-- more -->
**出错**
`FAILED: ParseException line 2:0 cannot recognize input near 'row' 'format' 'delimited' in statement`

**原因**
This is because the hive query will by default use the ^ as the delimiter. You can try the same by exporting to local file system.That should be supported.

**解决**
create an external table to the location where you want your output file.Use create table as command and insert the required data into the external table.By that you will get the data in the HDFS location
```
create external table user_game_count_lasttime2(userid STRING ,gamename STRING,count INT,lasttime STRING)
ROW FORMAT DELIMITED
FIElDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION '/user/hdu/recommend/gameRecommendNew4/test11.26/gameprestep1_2';

insert overwrite table user_game_count_lasttime2 
SELECT userid , gamename , COUNT(*) AS count , MAX(gamestarttime) AS lasttime
FROM userdetailtwo
GROUP BY userid , gamename;
```
