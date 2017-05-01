---
title: mapreduce运行内存及JVM配置错误running beyond physical memory limits
date: 2016/11/3 20:09:27
tags: MapReduce
categories: 大数据

---

### 错误描述：
```
16/11/02 21:52:58 INFO mapreduce.Job: Task Id : attempt_1476760655616_0575_r_000000_2, Status : FAILED
Container [pid=24537,containerID=container_1476760655616_0575_01_000052] is running beyond physical memory limits. Current usage: 4.0 GB of 3 GB physical memory used; 6.9 GB of 6.3 GB virtual memory used. Killing container.
Dump of the process-tree for container_1476760655616_0575_01_000052 :
	|- PID PPID PGRPID SESSID CMD_NAME USER_MODE_TIME(MILLIS) SYSTEM_TIME(MILLIS) VMEM_USAGE(BYTES) RSSMEM_USAGE(PAGES) FULL_CMD_LINE
```
<!-- more -->

### 解决：
查看hadoop配置参数
```
<property>
  <name>mapreduce.map.memory.mb</name>
    <value>3072</value>
    </property>
    <property>
      <name>mapreduce.reduce.memory.mb</name>
        <value>3072</value>
        </property>
        <property>
          <name>mapreduce.map.java.opts</name>
            <value>-Xmx3072m</value>
            </property>
            <property>
              <name>mapreduce.reduce.java.opts</name>
                <value>-Xmx6144m</value>
                </property>
<property>
```
`mapreduce.reduce.java.opts`参数大于`mapreduce.reduce.memory.mb`,需小于才行

在mapreduce执行函数中设置参数
```
conf.setInt("mapreduce.reduce.memory.mb", 6144);
```
1.

参考 
> http://stackoverflow.com/questions/21005643/container-is-running-beyond-memory-limits

我们集群中的每台机器都有48 GB的RAM。此RAM的一些应保留为操作系统使用。在每个节点上，我们将为YARN分配40 GB RAM以使用操作系统并保留8 GB

对于我们的示例集群，我们有一个容器的最小RAM（yarn.scheduler.minimum-allocation-mb）= 2 GB。因此，我们将为Map任务容器分配4 GB，为Reduce任务容器分配8 GB。

在mapred-site.xml中：
```
mapreduce.map.memory.mb：4096

mapreduce.reduce.memory.mb：8192
```

每个容器将运行Map和Reduce任务的JVM。 JVM堆大小应设置为低于上面定义的Map和Reduce内存，以使它们在YARN分配的Container内存的边界内。

在mapred-site.xml中：
```
mapreduce.map.java.opts：-Xmx3072m

mapreduce.reduce.java.opts：-Xmx6144m
```

以上设置配置Map和Reduce任务将使用的物理RAM的上限。
。

2.

> http://blog.chinaunix.net/uid-25691489-id-5587957.html

大概是job运行超过了map和reduce设置的内存大小，导致任务失败，调整增加了map和reduce的内容，问题排除，一些参数介绍如下：


RM的内存资源配置，主要是通过下面的两个参数进行的（这两个值是Yarn平台特性，应在yarn-site.xml中配置好）：
yarn.scheduler.minimum-allocation-mb
yarn.scheduler.maximum-allocation-mb
说明：单个容器可申请的最小与最大内存，应用在运行申请内存时不能超过最大值，小于最小值则分配最小值，从这个角度看，最小值有点想操作系统中的页。最小值还有另外一种用途，计算一个节点的最大container数目注：这两个值一经设定不能动态改变(此处所说的动态改变是指应用运行时)。

NM的内存资源配置，主要是通过下面两个参数进行的（这两个值是Yarn平台特性，应在yarn-sit.xml中配置） ：
```
yarn.nodemanager.resource.memory-mb
yarn.nodemanager.vmem-pmem-ratio
```
说明：每个节点可用的最大内存，RM中的两个值不应该超过此值。此数值可以用于计算container最大数目，即：用此值除以RM中的最小容器内存。虚拟内存率，是占task所用内存的百分比，默认值为2.1倍;注意：第一个参数是不可修改的，一旦设置，整个运行过程中不可动态修改，且该值的默认大小是8G，即使计算机内存不足8G也会按着8G内存来使用。

AM内存配置相关参数，此处以MapReduce为例进行说明（这两个值是AM特性，应在mapred-site.xml中配置），如下：
```
mapreduce.map.memory.mb
mapreduce.reduce.memory.mb
```
说明：这两个参数指定用于MapReduce的两个任务（Map and Reduce task）的内存大小，其值应该在RM中的最大最小container之间。如果没有配置则通过如下简单公式获得：
max(MIN_CONTAINER_SIZE, (Total Available RAM) / containers))
一般的reduce应该是map的2倍。注：这两个值可以在应用启动时通过参数改变；

AM中其它与内存相关的参数，还有JVM相关的参数，这些参数可以通过，如下选项配置：
mapreduce.map.java.opts
mapreduce.reduce.java.opts
说明：这两个参主要是为需要运行JVM程序（java、scala等）准备的，通过这两个设置可以向JVM中传递参数的，与内存有关的是，-Xmx，-Xms等选项。此数值大小，应该在AM中的map.mb和reduce.mb之间。

我们对上面的内容进行下总结，当配置Yarn内存的时候主要是配置如下三个方面：每个Map和Reduce可用物理内存限制；对于每个任务的JVM对大小的限制；虚拟内存的限制；

下面通过一个具体错误实例，进行内存相关说明，错误如下：
```
Container[pid=41884,containerID=container_1405950053048_0016_01_000284] is running beyond virtual memory limits. Current usage: 314.6 MB of 2.9 GB physical memory used; 8.7 GB of 6.2 GB virtual memory used. Killing container.
```
配置如下：
```
<property>
            <name>yarn.nodemanager.resource.memory-mb</name>
            <value>100000</value>
        </property>
        <property>
            <name>yarn.scheduler.maximum-allocation-mb</name>
            <value>10000</value>
        </property>
        <property>
            <name>yarn.scheduler.minimum-allocation-mb</name>
            <value>3000</value>
        </property>
       <property>
            <name>mapreduce.reduce.memory.mb</name>
            <value>2000</value>
        </property>
```

通过配置我们看到，容器的最小内存和最大内存分别为：3000m和10000m，而reduce设置的默认值小于2000m，map没有设置，所以两个值均为3000m，也就是log中的“2.9 GB physical
memory used”。而由于使用了默认虚拟内存率(也就是2.1倍)，所以对于Map Task和Reduce Task总的虚拟内存为都为3000*2.1=6.2G。而应用的虚拟内存超过了这个数值，故报错 。解决办
法：在启动Yarn是调节虚拟内存率或者应用运行时调节内存大小。