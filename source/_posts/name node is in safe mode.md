---
title: name node is in safe mode问题
date: 2016/10/18 21:56:09 
tags: Hadoop
categories: 大数据

---

### name node is in safe mode

问题： 向hdfs put数据的时候，导致了 name node is in safe mode，然后使用 Hadoop dfsadmin -safemode leave 后， 解除了安全模式。可是再次使用hdfs put或rm数据，仍旧导致name node 进入安全模式。

答案：分析了一下，问题是namenode所在机器的硬盘满了。因此即使使用了 hadoop dfsadmin -safemode leave 之后， 仍旧不能使用hdfs。

#### 解决办法：
1. 删除namenode所在机器的一些数据（本地数据）
2. 结束安全模式   hadoop dfsadmin -safemode leave 
3. 可以正常使用hdfs了