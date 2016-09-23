---
title: Hive1.2.1安装笔记
date: 2016/8/22 0:47:36 
tags: Hive
categories: 大数据

---

### 环境
```
ubuntu 16.04
4台机器的Hadoop2.7.2集群
Mysql安装在slave2中
hive安装在master上
```

### 下载Hive
```
$ wget http://mirrors.cnnic.cn/apache/hive/hive-1.2.1/apache-hive-1.2.1-bin.tar.gz
# 解压
$ tar -zxvf apache-hive-1.2.1-bin.tar.gz /home/ubuntu/cloud
```

### 配置Hive环境变量
```
$ sudo vim /etc/profile

#添加
export HIVE_HOME=/home/ubuntu/cloud/apache-hive-1.2.1-bin
export PATH=$PATH:$HIVE_HOME/bin

$source /etc/profile
```
### 在Mysql中创建Hive用户

```
mysql>CREATE USER 'hive' IDENTIFIED BY 'hive';
mysql>GRANT ALL PRIVILEGES ON *.* TO 'hive'@'%' IDENTIFIED BY '123' WITH GRANT OPTION;
mysql>flush privileges;
```

### 创建Hive数据库
```
$ mysql -uhive -phive
mysql>create database hive;
```

### 配置Hive
进入Hive的cong目录,找到`hive-default.xml.template`，cp份为`hive-default.xml`,另创建`hive-site.xml`并添加参数
```
$ vim hive-site.xml
# 删除configuration标签里的所有内容 添加如下内容

	<property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://slave2:3306/hive?createDatabaseIfNotExist=true</value>
        <description>JDBC connect string for a JDBC metastore</description>    
	</property>   
	<property> 
        <name>javax.jdo.option.ConnectionDriverName</name> 
        <value>com.mysql.jdbc.Driver</value> 
        <description>Driver class name for a JDBC metastore</description>     
	</property>               
 
	<property> 
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>hive</value>
        <description>username to use against metastore database</description>
	</property>
	<property>  
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>hive</value>
        <description>password to use against metastore database</description>  
	</property>          

```

### 下载mysql-connector-java-5.1.32-bin.jar 
这里用5.1.32版本测试不报错，5.1.38会报warn

```
#将连接jar包拷贝到Hive的lib目录
$ cp mysql-connector-java-5.1.32-bin.jar /home/ubuntu/cloud/apache-hive-1.2.1-bin/lib/
```
### Hive启动

要启动metastore服务
```
$ hive --service metastore &
$ jps
10288 RunJar  #多了一个进程
9365 NameNode
9670 SecondaryNameNode
11096 Jps
9944 NodeManager
9838 ResourceManager
9471 DataNode

```
启动hive命令行
```
ubuntu@master:~$ hive

Logging initialized using configuration in jar:file:/home/ubuntu/cloud/apache-hive-1.2.1-bin/lib/hive-common-1.2.1.jar!/hive-log4j.properties
hive> show tables;
OK
Time taken: 0.705 seconds
```
