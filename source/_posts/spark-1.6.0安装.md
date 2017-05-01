---
title: spark-1.6.0安装
date: 2016/10/27 12:52:59 
tags: Spark
categories: 大数据

---


> 已安装Hadoop2.7.2的三节点集群


### 安装Scala

1. 下载scala-2.11.7.tgz
2. 解压Scala
```
$ tar -zxvf scala-2.11.7.tgz -C ~/cloud/
```
3. 配置环境变量
```
export  SCALA_HOME=/home/ubuntu/cloud/scala-2.11.7
export PATH = $PATH:$SCALA_HOME/bin
```
4. 测试
```
scala -version
Scala code runner version 2.11.7 -- Copyright 2002-2013, LAMP/EPFL
```
5. 配置到每台节点
<!-- more -->
### 安装Spark

#Spark
export SPARK_HOME=/home/ubuntu/cloud/spark-1.6.0
export PATH=$PATH:$SPARK_HOME/bin

export JAVA_HOME=/usr/java/jdk1.8.0_91
export SPARK_MASTER_IP=10.0.0.7
#export SPARK_WORKER_MEMORY=2g
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop

slave1
slave2