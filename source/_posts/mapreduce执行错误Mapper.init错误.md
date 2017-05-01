---
title: mapreduce执行错误Mapper.<init>错误
date: 2016/11/3 20:07:23 
tags: MapReduce
categories: 大数据

---

> 参考
> http://blog.itpub.net/30066956/viewspace-2107549/

错误详情：
```
16/11/02 21:37:26 INFO mapreduce.Job: Task Id : attempt_1476760655616_0574_m_000027_2, Status : FAILED
Error: java.lang.RuntimeException: java.lang.NoSuchMethodException: com.hdu.recommend.tools.CopyData$QLMapper.<init>()
	at org.apache.hadoop.util.ReflectionUtils.newInstance(ReflectionUtils.java:131)
	at org.apache.hadoop.mapred.MapTask.runNewMapper(MapTask.java:742)
	at org.apache.hadoop.mapred.MapTask.run(MapTask.java:341)
	at org.apache.hadoop.mapred.YarnChild$2.run(YarnChild.java:168)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:415)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1614)
	at org.apache.hadoop.mapred.YarnChild.main(YarnChild.java:163)
Caused by: java.lang.NoSuchMethodException: com.hdu.recommend.tools.CopyData$QLMapper.<init>()
	at java.lang.Class.getConstructor0(Class.java:2849)
	at java.lang.Class.getDeclaredConstructor(Class.java:2053)
	at org.apache.hadoop.util.ReflectionUtils.newInstance(ReflectionUtils.java:125)
	... 7 more

```
<!-- more -->
解决：
注意Mapper 与 Reducer 类写成内部类，一定要加static ！！！！