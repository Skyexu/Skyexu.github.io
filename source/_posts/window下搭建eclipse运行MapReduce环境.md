---
title: window下搭建eclipse运行MapReduce环境
date: 2016-07-13 09:47:21
tags: MapReduce
categories: 大数据

---

## 系统环境及所需文件

- eclipse-jee-mars-2
- hadoop2.7.2
- [hadoop-eclipse-plugin](https://github.com/winghc/hadoop2x-eclipse-plugin)
- [hadoop.dll & winutils.exe](http://download.csdn.net/download/chenxf10/9564165)

## 修改Master节点的hdfs-site.xml

    <property>      
		<name>dfs.permissions</name>      
		<value>false</value>  
	</property> 
旨在取消权限检查

```
<property> 
	<name>dfs.web.ugi</name> 
	<value>Skye,supergroup</value> 
</property>
```

## 配置Hadoop插件

1. windows下载hadoop-2.7.2解压到某目录下，如：E:\hadoop\hadoop-2.7.2
2. 下载hadoop-eclipse-plugin插件hadoop-eclipse-plugin，将release目录下的hadoop-eclipse-plugin-2.6.0.jar拷贝到eclipse/plugins，重启eclipse。
3. 插件配置windows->show view->other 显示mapreduce视图
4. window->preferences->hadoop map/reduce 指定windows上的hadoop根目录（即：E:\hadoop\hadoop-2.7.2）
5. 在Map/Reduce Locations 面板中，点击小象图标定义hadoop 
![](http://i.imgur.com/61yzGm9.png)
解释： 
MapReduce Master 
Host：虚拟机hadoop master对应ip 
Port：hdfs-site.xml中dfs.datanode.ipc.address指定的的端口号。此处填9001 
DFS Master中Port：core-site.xml中fs.defaultFS指定的端口。应填9000 
User name：linux中运行hadoop的用户。 

## 配置完毕查看结果
![](http://i.imgur.com/zVOXsbw.png)

## windows下运行环境配置
1. 在系统环境变量中增加HADOOP_HOME，并在Path中加入%HADOOP_HOME%\bin
2. 将下载下来的hadoop.dll,winutils.exe拷贝到HADOOP_HOME/bin目录下

## 创建 MapReduce工程并运行
需要拷贝 服务器hadoop中的log4j.properties文件到工程的src目录

`run on hadoop`

运行时报如下错误，弄了好长一段时间，发现原因是服务器通过内网ip访问，外网无法解析。用虚拟机连接成功.
```
16/07/13 10:42:38 INFO util.ProcfsBasedProcessTree: ProcfsBasedProcessTree currently is supported only on Linux.
16/07/13 10:42:39 INFO mapreduce.Job: Job job_local510776960_0001 running in uber mode : false
16/07/13 10:42:39 INFO mapreduce.Job:  map 0% reduce 0%
16/07/13 10:42:39 INFO mapred.Task:  Using ResourceCalculatorProcessTree : org.apache.hadoop.yarn.util.WindowsBasedProcessTree@3bfe5dd7
16/07/13 10:42:39 INFO mapred.MapTask: Processing split: hdfs://Master:9000/test/test3.txt:0+259
16/07/13 10:42:39 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
16/07/13 10:42:39 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
16/07/13 10:42:39 INFO mapred.MapTask: soft limit at 83886080
16/07/13 10:42:39 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
16/07/13 10:42:39 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
16/07/13 10:42:39 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
16/07/13 10:43:00 WARN hdfs.BlockReaderFactory: I/O error constructing remote block reader.
java.net.ConnectException: Connection timed out: no further information
	at sun.nio.ch.SocketChannelImpl.checkConnect(Native Method)
	at sun.nio.ch.SocketChannelImpl.finishConnect(Unknown Source)
	at org.apache.hadoop.net.SocketIOWithTimeout.connect(SocketIOWithTimeout.java:206)
	at org.apache.hadoop.net.NetUtils.connect(NetUtils.java:531)
	at org.apache.hadoop.hdfs.DFSClient.newConnectedPeer(DFSClient.java:3436)
	at org.apache.hadoop.hdfs.BlockReaderFactory.nextTcpPeer(BlockReaderFactory.java:777)
	at org.apache.hadoop.hdfs.BlockReaderFactory.getRemoteBlockReaderFromTcp(BlockReaderFactory.java:694)
	at org.apache.hadoop.hdfs.BlockReaderFactory.build(BlockReaderFactory.java:355)
	at org.apache.hadoop.hdfs.DFSInputStream.blockSeekTo(DFSInputStream.java:656)
	at org.apache.hadoop.hdfs.DFSInputStream.readWithStrategy(DFSInputStream.java:882)
	at org.apache.hadoop.hdfs.DFSInputStream.read(DFSInputStream.java:934)
	at java.io.DataInputStream.read(Unknown Source)
	at org.apache.hadoop.mapreduce.lib.input.UncompressedSplitLineReader.fillBuffer(UncompressedSplitLineReader.java:59)
	at org.apache.hadoop.util.LineReader.readDefaultLine(LineReader.java:216)
	at org.apache.hadoop.util.LineReader.readLine(LineReader.java:174)
	at org.apache.hadoop.mapreduce.lib.input.UncompressedSplitLineReader.readLine(UncompressedSplitLineReader.java:91)
	at org.apache.hadoop.mapreduce.lib.input.LineRecordReader.skipUtfByteOrderMark(LineRecordReader.java:144)
	at org.apache.hadoop.mapreduce.lib.input.LineRecordReader.nextKeyValue(LineRecordReader.java:184)
	at org.apache.hadoop.mapred.MapTask$NewTrackingRecordReader.nextKeyValue(MapTask.java:556)
	at org.apache.hadoop.mapreduce.task.MapContextImpl.nextKeyValue(MapContextImpl.java:80)
	at org.apache.hadoop.mapreduce.lib.map.WrappedMapper$Context.nextKeyValue(WrappedMapper.java:91)
	at org.apache.hadoop.mapreduce.Mapper.run(Mapper.java:145)
	at org.apache.hadoop.mapred.MapTask.runNewMapper(MapTask.java:787)
	at org.apache.hadoop.mapred.MapTask.run(MapTask.java:341)
	at org.apache.hadoop.mapred.LocalJobRunner$Job$MapTaskRunnable.run(LocalJobRunner.java:243)
	at java.util.concurrent.Executors$RunnableAdapter.call(Unknown Source)
	at java.util.concurrent.FutureTask.run(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
	at java.lang.Thread.run(Unknown Source)
16/07/13 10:43:00 WARN hdfs.DFSClient: Failed to connect to /10.0.0.14:50010 for block, add to deadNodes and continue. java.net.ConnectException: Connection timed out: no further information
java.net.ConnectException: Connection timed out: no further information
	at sun.nio.ch.SocketChannelImpl.checkConnect(Native Method)
	at sun.nio.ch.SocketChannelImpl.finishConnect(Unknown Source)
	at org.apache.hadoop.net.SocketIOWithTimeout.connect(SocketIOWithTimeout.java:206)
	at org.apache.hadoop.net.NetUtils.connect(NetUtils.java:531)
	at org.apache.hadoop.hdfs.DFSClient.newConnectedPeer(DFSClient.java:3436)
	at org.apache.hadoop.hdfs.BlockReaderFactory.nextTcpPeer(BlockReaderFactory.java:777)
	at org.apache.hadoop.hdfs.BlockReaderFactory.getRemoteBlockReaderFromTcp(BlockReaderFactory.java:694)
	at org.apache.hadoop.hdfs.BlockReaderFactory.build(BlockReaderFactory.java:355)
	at org.apache.hadoop.hdfs.DFSInputStream.blockSeekTo(DFSInputStream.java:656)
	at org.apache.hadoop.hdfs.DFSInputStream.readWithStrategy(DFSInputStream.java:882)
	at org.apache.hadoop.hdfs.DFSInputStream.read(DFSInputStream.java:934)
	at java.io.DataInputStream.read(Unknown Source)
	at org.apache.hadoop.mapreduce.lib.input.UncompressedSplitLineReader.fillBuffer(UncompressedSplitLineReader.java:59)
	at org.apache.hadoop.util.LineReader.readDefaultLine(LineReader.java:216)
	at org.apache.hadoop.util.LineReader.readLine(LineReader.java:174)
	at org.apache.hadoop.mapreduce.lib.input.UncompressedSplitLineReader.readLine(UncompressedSplitLineReader.java:91)
	at org.apache.hadoop.mapreduce.lib.input.LineRecordReader.skipUtfByteOrderMark(LineRecordReader.java:144)
	at org.apache.hadoop.mapreduce.lib.input.LineRecordReader.nextKeyValue(LineRecordReader.java:184)
	at org.apache.hadoop.mapred.MapTask$NewTrackingRecordReader.nextKeyValue(MapTask.java:556)
	at org.apache.hadoop.mapreduce.task.MapContextImpl.nextKeyValue(MapContextImpl.java:80)
	at org.apache.hadoop.mapreduce.lib.map.WrappedMapper$Context.nextKeyValue(WrappedMapper.java:91)
	at org.apache.hadoop.mapreduce.Mapper.run(Mapper.java:145)
	at org.apache.hadoop.mapred.MapTask.runNewMapper(MapTask.java:787)
	at org.apache.hadoop.mapred.MapTask.run(MapTask.java:341)
	at org.apache.hadoop.mapred.LocalJobRunner$Job$MapTaskRunnable.run(LocalJobRunner.java:243)
	at java.util.concurrent.Executors$RunnableAdapter.call(Unknown Source)
	at java.util.concurrent.FutureTask.run(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
	at java.lang.Thread.run(Unknown Source)

```