---
title: eclipse中操作hive错误org.apache.hadoop.security.AccessControlException
date: 2016/11/3 19:27:08  
tags: Hive
categories: 大数据

---

错误：
```
java.sql.SQLException: Error while processing statement: FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.mr.MapRedTask

Job Submission failed with exception 'org.apache.hadoop.security.AccessControlException(Permission denied: user=anonymous, access=WRITE, inode="/user":hdfs:supergroup:drwxr-xr-x
```
解决：
权限问题
```
hadoop fs -chmod -R 777  /
```

<!-- more -->