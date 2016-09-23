title: mapreduce任务运行时shuffle Error
date: 2016-07-11 12:36:36
tags: MapReduce
categories: 大数据

---

*本文引用参考*：[MapReduce任务Shuffle Error错误](http://blog.csdn.net/dslztx/article/details/46445725)
*相关参考连接*：[ yarn & mapreduce 配置参数总结](http://blog.csdn.net/stark_summer/article/details/48494391)

## 错误描述

在运行MapReduce任务的时候，出现如下错误：
```
Error: org.apache.hadoop.mapreduce.task.reduce.Shuffle$ShuffleError: error in shuffle in fetcher#1
        at org.apache.hadoop.mapreduce.task.reduce.Shuffle.run(Shuffle.java:134)
        at org.apache.hadoop.mapred.ReduceTask.run(ReduceTask.java:376)
        at org.apache.hadoop.mapred.YarnChild$2.run(YarnChild.java:167)
        at java.security.AccessController.doPrivileged(Native Method)
        at javax.security.auth.Subject.doAs(Subject.java:396)
        at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1556)
        at org.apache.hadoop.mapred.YarnChild.main(YarnChild.java:162)
Caused by: java.lang.OutOfMemoryError: Java heap space
        at org.apache.hadoop.io.BoundedByteArrayOutputStream.<init>(BoundedByteArrayOutputStream.java:56)
        at org.apache.hadoop.io.BoundedByteArrayOutputStream.<init>(BoundedByteArrayOutputStream.java:46)
        at org.apache.hadoop.mapreduce.task.reduce.InMemoryMapOutput.<init>(InMemoryMapOutput.java:63)
        at org.apache.hadoop.mapreduce.task.reduce.MergeManagerImpl.unconditionalReserve(MergeManagerImpl.java:297)
        at org.apache.hadoop.mapreduce.task.reduce.MergeManagerImpl.reserve(MergeManagerImpl.java:287)
        at org.apache.hadoop.mapreduce.task.reduce.Fetcher.copyMapOutput(Fetcher.java:411)
        at org.apache.hadoop.mapreduce.task.reduce.Fetcher.copyFromHost(Fetcher.java:341)
        at org.apache.hadoop.mapreduce.task.reduce.Fetcher.run(Fetcher.java:165)
```

## 解决方案
  根据《Hadoop:The Definitive Guide 4th　Edition》所述(P203-219)，map任务和reduce任务之间要经过一个shuffle过程，该过程复制map任务的输出作为reduce任务的输入。
  具体的来说，shuffle过程的输入是：map任务的输出文件，它的输出接收者是：运行reduce任务的机子上的内存buffer，并且shuffle过程以并行方式运行。
  参数mapreduce.reduce.shuffle.input.buffer.percent控制运行reduce任务的机子上多少比例的内存用作上述buffer(默认值为0.70)，参数mapreduce.reduce.shuffle.parallelcopies控制shuffle过程的并行度(默认值为5)。那么"mapreduce.reduce.shuffle.input.buffer.percent" * "mapreduce.reduce.shuffle.parallelcopies" 必须小于等于1，否则就会出现如上错误
因此，我将mapreduce.reduce.shuffle.input.buffer.percent设置成值为0.1，就可以正常运行了（设置成0.2，还是会抛同样的错）

`job.getConfiguration().setStrings("mapreduce.reduce.shuffle.input.buffer.percent", "0.1");`
或者在`maperd-site.xml`中修改
```
<property>
   <name>mapreduce.reduce.input.buffer.percent</name>
   <value>0.0</value>
</property>
```

  另外，可以发现如果使用两个参数的默认值，那么两者乘积为3.5，大大大于1了，为什么没有经常抛出以上的错误呢？
1)首先，把默认值设为比较大，主要是基于性能考虑，将它们设为比较大，可以大大加快从map复制数据的速度
2)其次，要抛出如上异常，还需满足另外一个条件，就是map任务的数据一下子准备好了等待shuffle去复制，在这种情况下，就会导致shuffle过程的“线程数量”和“内存buffer使用量”都是满负荷的值，自然就造成了内存不足的错误；而如果map任务的数据是断断续续完成的，那么没有一个时刻shuffle过程的“线程数量”和“内存buffer使用量”是满负荷值的，自然也就不会抛出如上错误

另外，如果在设置以上参数后，还是出现错误，那么有可能是运行Reduce任务的进程的内存总量不足，可以通过mapred.child.java.opts参数来调节，比如设置mapred.child.java.opts=-Xmx2024m