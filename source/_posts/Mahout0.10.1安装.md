---
title: Mahout0.10.1安装
date: 2016-08-15 20:00:16
tags: Mahout
categories: 大数据

---

### 解压安装包

### 编辑环境变量
`sudo vim /etc/profile`
```
#MAHOUT
export MAHOUT_HOME=/home/ubuntu/cloud/mahout-0.10.1
export MAHOUT_CONF_DIR=$MAHOUT_HOME/conf
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop

export PATH=$PATH:${JAVA_HOME}/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$MAHOUT_HOME/conf:$MAHOUT_HOME/bin:
```
### 更新配置
`source /etc/profile`

### 输入mahout测试 出现许多算法

### 进行kmeans算法简单运行
- 下载测试数据集synthetic_control.data
   http://archive.ics.uci.edu/ml/databases/synthetic_control/
- 在hdfs上创建目录  `/user/ubuntu/testdata`
- 上传测试数据
` hadoop fs -put synthetic_control.data /user/ubuntu/testdata`  
- 运行
`mahout org.apache.mahout.clustering.syntheticcontrol.kmeans.Job`

参考
> http://www.cnblogs.com/zhangduo/p/4679907.html