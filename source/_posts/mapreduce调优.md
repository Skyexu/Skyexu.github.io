---
title: mapreduce调优
date: 2016/12/2 9:16:39  
tags: MapReduce
categories: 大数据

---



## 对应用程序进行调优

1. 避免输入大量小文件。大量的小文件(不足一个block大小)作为输入数据会产生很多的Map任务(默认一个分片对应一个Map任务)，而每个Map任务实际工作量又非常小，系统要花更多的时间来将这些Map任务的输出进行整合。如果将大量的小文件进行预处理合并成一个或几个大文件，任务执行的效率可能会提升几十倍。可手动将小文件合并成大文件，或通过Hadoop的SequenceFile、CombineFileInputFormat将多个文件打包到一个输入单元中，使得每个Map处理更多的数据，从而提高性能。

2. 输入文件size巨大，但不是小文件。这种情况可以通过增大每个mapper的input size，即增大minSize或者增大blockSize来减少所需的mapper的数量。增大blockSize通常不可行，因为当HDFS被hadoop namenode -format之后，blockSize就已经确定了（由格式化时dfs.block.size决定），如果要更改blockSize，需要重新格式化HDFS，这样当然会丢失已有的数据。所以通常情况下只能通过增大minSize，即增大mapred.min.split.size的值。
<!-- more -->
3. 预判并过滤无用数据。可以使用一些过滤工具，在作业执行之前将数据中无用的数据进行过滤，可极大提高MapReduce执行效率。Bloom Filter是一种功能强大的过滤器，执行效率高，时间复杂度为O(1)，缺点是存在一定的误判可能，详细参考《Bloom Filter概念和原理》。当将一个非常大的表和一个非常小的表进行表连接操作时，可以使用Bloom Filter将小表数据作为Bloom Filter的输入数据，将大表的原始数据进行过滤(过滤不通过的数据一定是不可用的，过滤通过的数据可能有用可能无用)，可提高程序执行的效率。

4. 合理使用分布式缓存DistributedCache。DistributedCache可以将一些字典、jar包、配置文件等缓存到需要执行map任务的节点中，避免map任务多次重复读取这些资源，尤其在join操作时，使用DistributedCache缓存小表数据在map端进行join操作，可避免shuffle、reduce等操作，提高程序运行效率。

5. 重用Writable类型。避免大量多次new这些Writable对象，这会花费java垃圾收集器大量的清理工作，建议在map函数外定义这些Writable对象，如下所示：
```
class MyMapper … {
    Text wordText = new Text();
    IntWritable one = new IntWritable(1);
    public void map(...) {
        for (String word: words) {
            wordText.set(word);
            context.write(wordText, one);
        }
    }
}
```

6. 合理设置Combiner。Combine阶段处于Map端操作的最后一步，设置Combine操作可大大提高MapReduce的执行效率，前提是增加Combine不能改变最终的结果值，换句话说，不是所有的MapReduce程序都能添加Combine，如求平均数的MapReduce程序就不适合设置Combine操作。通常Combine函数与Reduce函数一致

## 对参数进行调优（基于hadoop2.6.0）

**HDFS参数调优(hdfs-site.xml)**

- dfs.namenode.handler.count：namenode用于处理RPC的线程数，默认值10，可根据NameNode所在节点机器配置适当调大，如32、64；

- dfs.datanode.handler.count：datanode上用于处理RPC的线程数，2.6版本默认值10，早期1.x版本默认值为3，可根据datanode节点的配置适当调整；

**MapReduce参数调优(mapred-site.xml)**

- mapreduce.tasktracker.map.tasks.maximum：每个nodemanager节点上可运行的最大map任务数，默认值2，可根据实际值调整为10~100；

-  mapreduce.tasktracker.reduce.tasks.maximum：每个nodemanager节点上可运行的最大reduce任务数，默认值2，可根据实际值调整为10~100；

-  mapreduce.output.fileoutputformat.compress：是否对任务输出产生的结果进行压缩，默认值false。对传输数据进行压缩，既可以减少文件的存储空间，又可以加快数据在网络不同节点之间的传输速度。

- mapreduce.output.fileoutputformat.compress.type：输出产生任务数据的压缩方式，默认值RECORD，可配置值有：NONE、RECORD、BLOCK

- mapreduce.map.output.compress：map端压缩
 
- mapreduce.map.output.compress.codec：map压缩格式
 
- mapreduce.task.io.sort.mb：map任务输出结果的内存环形缓冲区大小，默认值100M，可根据map节点的机器进行配置，貌似不能超过值mapred.child.java.opts；

- mapreduce.map.sort.spill.percent：map任务输出环形缓冲区大小溢写触发最大比例，默认值80%，这个值一般不建议修改；

- mapreduce.reduce.shuffle.parallelcopies：reduce节点通过http拷贝map输出结果数据到本地的最大工作线程数，默认值5，可根据节点机器配置适当修改；

- mapreduce.reduce.shuffle.input.buffer.percent：reduce节点在shuffle阶段拷贝map输出结果数据到本地时，内存缓冲区大小所占JVM内存的比例，默认值0.7，一般不建议修改；

- mapreduce.reduce.shuffle.merge.percent：reduce节点shuffle内存缓冲区溢写触发最大比例，默认值0.66，一般不建议修改；

- mapred.child.java.opts：配置每个map或reduce使用的内存数量，默认值-Xmx200m，即200M。如果nodemanager所在节点


## Map和Reduce个数设置
1. map的数量
map的数量通常是由hadoop集群的DFS块大小确定的，也就是输入文件的总块数，正常的map数量的并行规模大致是每一个Node是10~100个，对于CPU消耗较小的作业可以设置Map数量为300个左右，但是由于hadoop的没一个任务在初始化时需要一定的时间，因此比较合理的情况是每个map执行的时间至少超过1分钟。具体的数据分片是这样的，InputFormat在默认情况下会根据hadoop集群的DFS块大小进行分片，每一个分片会由一个map任务来进行处理，当然用户还是可以通过参数mapred.min.split.size参数在作业提交客户端进行自定义设置。还有一个重要参数就是mapred.map.tasks，这个参数设置的map数量仅仅是一个提示，只有当InputFormat 决定了map任务的个数比mapred.map.tasks值小时才起作用。同样，Map任务的个数也能通过使用JobConf 的conf.setNumMapTasks(int num)方法来手动地设置。这个方法能够用来增加map任务的个数，但是不能设定任务的个数小于Hadoop系统通过分割输入数据得到的值。因此，如果你有一个大小是10TB的输入数据，并设置DFS块大小为 128M，你必须设置至少82K个map任务，除非你设置的mapred.map.tasks参数比这个数还要大。当然为了提高集群的并发效率，可以设置一个默认的map数量，当用户的map数量较小或者比本身自动分割的值还小时可以使用一个相对交大的默认值，从而提高整体hadoop集群的效率。

2. reduece的数量
reduce在运行时往往需要从相关map端复制数据到reduce节点来处理，因此相比于map任务。reduce节点资源是相对比较缺少的，同时相对运行较慢，正确的reduce任务的个数应该是0.95或者1.75 *（节点数 ×mapred.tasktracker.tasks.maximum参数值）。mapred.tasktracker.tasks.reduce.maximum的数量一般设置为各节点cpu core数量,或者数量减1，即能同时计算的slot数量。如果任务数是节点个数的0.95倍，那么所有的reduce任务能够在 map任务的输出传输结束后同时开始运行。如果任务数是节点个数的1.75倍，那么高速的节点会在完成他们第一批reduce任务计算之后开始计算第二批 reduce任务，这样的情况更有利于负载均衡。同时需要注意增加reduce的数量虽然会增加系统的资源开销，但是可以改善负载匀衡，降低任务失败带来的负面影响。同样，Reduce任务也能够与 map任务一样，通过设定JobConf 的conf.setNumReduceTasks(int num)方法来增加任务个数。
cpu数量 = 服务器CPU总核数 / 每个CPU的核数 
服务器CPU总核数 = more /proc/cpuinfo | grep 'processor' | wc -l 
每个CPU的核数 = more /proc/cpuinfo | grep 'cpu cores'

3. reduce数量为0
有些作业不需要进行归约进行处理，那么就可以设置reduce的数量为0来进行处理，这种情况下用户的作业运行速度相对较高，map的输出会直接写入到 SetOutputPath(path)设置的输出目录，而不是作为中间结果写到本地。同时Hadoop框架在写入文件系统前并不对之进行排序。

参考转载
> http://www.cnblogs.com/hanganglin/p/4563716.html
> https://my.oschina.net/Chanthon/blog/150500