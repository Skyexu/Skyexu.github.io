---
title: MapReduce应用中CombineFileInputFormat原理与用法
date: 2016/12/12 21:26:41 
tags: MapReduce
categories: 大数据

---

> 转载自：http://ju.outofmemory.cn/entry/72024

HDFS本身被设计来存储大文件，但是有时难免会有小文件出现，有时很可能时大量的小文件。通过MapReduce处理大量小文件时会遇到些问题。

MapReduce程序会将输入的文件进行分片(Split)，每个分片对应一个map任务，而默认一个文件至少有一个分片，一个分片也只属于一个文件。这样大量的小文件会导致大量的map任务，导致资源过度消耗，且效率低下。Hadoop自身包含了CombineFileInputFormat，其功能是将多个小文件合并如一个分片，由一个map任务处理，这样就减少了不必要的map数量。

在了解CombineFileInputFormat之前，我们应了解其父类FileInputFormat的基本处理逻辑。注意这里的FileInputFormat的路径是**org.apache.hadoop.mapreduce.lib.input.FileInputFormat**，是新的MapReduce API。 mapred包下的FileInputFormat对应老的API，不再推荐使用。

<!-- more -->

[![CombineFileInputFormat_type](http://www.sqlparty.com/wp-content/uploads/2013/12/CombineFileInputFormat_type.png)](http://www.sqlparty.com/wp-content/uploads/2013/12/CombineFileInputFormat_type.png)

## FileInputFormat的基本处理逻辑

FileInputFormat是基于文件的InputFormat的抽象基类，如上图所示，基于文件的衍生类有很多，如文本文件TextInputFormat，序列文件SequenceFileInputFormat等。

FileInputFormat提供了分片的基本实现getSplits(JobContext)，其子类可以重写isSplitable(JobContext, Path)方法，来使得输入的一个或多个文件是否不做分片，完整由被一个Map任务进行处理。

FileInputFormat有如下特点：

1.  将以下划线&#8221;_&#8221;或点&#8221;.&#8221;开头的文件作为隐藏文件，不做为输入文件。
2.  所有文件isSplitable(JobContext, Path)都是true，但是针对如果输入文件时压缩的、流式的，那么子类中应重新该函数，判断是否真的可以分片。
3.  由于每个文件块大小可能不一样，所以每个文件分别计算分片大小，计算规则如下：

    1.  取值A：

        1.  FormatMinSplitSize，本Format设置的最小Split大小，通过GetFormatMinSplitSize()获取，此类中定义为1个字节。
        2.  MinSplitSize，配置文件(配置键mapreduce.input.fileinputformat.split.minsize)或者直接设置，通过GetMinSplitSize()获取)，未设置则为0。
        3.  取两者较大值。

    2.  取值B:

        1.  MaxSplitSize，通过配置文件设置或者SetMaxSplitSize()设置，通过GetMaxSplitSize()获取，无设置则取LONG.MAXVALUE。
        2.  文件块大小BLOCKSIZE
        3.  取两者较小值

    3.  再取A、B的较大值。
    4.  例：常用的TextInputFormat类中，没有对分片算法进行重写，那么我们可以认为，使用TextInputFormat时，在未做其他设置的情况下，默认分片大小等于BLOCKSIZE，如果设置了mapreduce.input.fileinputformat.split.minsize，则取其与BLOCK的较大值。

4.  分片大小确定后，就将文件进行依次划分。

## 文本类型TextInputFormat的使用

如上我们知道TextInputFormat是FileInputFormat的子类，其自定义内容很少，通过它可以大致知道扩展FileInputFormat大致需要做些什么。整个类如下：

    public class TextInputFormat extends FileInputFormat&lt;LongWritable, Text&gt; {

      @Override
      public RecordReader&lt;LongWritable, Text&gt;
        createRecordReader(InputSplit split,
                           TaskAttemptContext context) {
        String delimiter = context.getConfiguration().get(
            &quot;textinputformat.record.delimiter&quot; );
        byte[] recordDelimiterBytes = null;
        if ( null != delimiter)
          recordDelimiterBytes = delimiter.getBytes(Charsets. UTF_8);
        return new LineRecordReader(recordDelimiterBytes);
      }

      @Override
      protected boolean isSplitable(JobContext context, Path file) {
        final CompressionCodec codec =
          new CompressionCodecFactory(context.getConfiguration()).getCodec(file);
        if ( null == codec) {
          return true;
        }
        return codec instanceof SplittableCompressionCodec;
      }
    }

分片的方式采用的是父类FileInputFormat的逻辑，上文中已经说明。重写了isSplitable()，根据文本文件的压缩属性来判断是否可以进行分片。

而createRecorderReader()是定义文本文件的读取方式，实际文件读取是通过它返回的RecordReader&lt;LongWritable, Text&gt;类实现的。

这样，在整个输入文件读取过程，大致会涉及如下几个步骤：

1.  指定输入文件路径，如FileInputFormat.addInputPaths(job, args[0])
2.  指定文件的处理类型，如job.setInputFormatClass(MyInputFormat. class)
3.  在这个InputFormatClass内部，考虑:

    1.  是否可以进行分片 isSplitable(JobContext context, Path file)
    2.  如何分片 List&lt;InputSplit&gt; getSplits(JobContext job)
    3.  如何读取分片中的记录 RecordReader&lt;LongWritable, Text&gt; createRecordReader(InputSplit split,TaskAttemptContext context)
    4.  以上确定后，用户开发的Map任务中就可以直接处理每一条记录KeyValue。

这些步骤下，我们基本就可以理解TextInputFormat如何被使用了。其他类型的InputFormat子类，其流程步骤也基本与上相同，可以重写相关的类方法来实现不同处理方式。

从TextInputFormat中的分片逻辑(FileInputFormat的getSplits)中可以确定，分片都是针对单个文件而言的，如果文件本身较小，没有达到一个分片大小，那么每个小文件都是一个分片。而一个分片就对应一个Map任务。如果有大量的小文件作为Map的输入，那么其会导致生成大量Map任务，造成处理的缓慢、资源的浪费，如何减少map任务的数量提高处理效率呢？CombineInputFormat就是为解决这样的问题。

## 3.CombineInputFormat原理与用法

CombineInputFormat的功能，是将一个**目录**（可能包括多个小文件，不包括子目录）作为一个map的输入，而不是通常使用一个**文件**作为输入。

CombineInputFormat本身是个抽象类，要使用它，涉及：

** 1)CombineFileSplit **

我们的目标是使得一个split不是属于一个文件，而是可能包含多个文件，所以这里不再使用常用的FileSplit，而是CombineFileSplit，包括了各个文件的路径、长度、读的起始位置等信息。CombineFileSplit是CombineInputFormat中getSplits()的对象类型。

**2)CombineInputFormat 核心处理类**

2.1)其基本思想：

分片从指定路径下的多个文件构建，不同文件可以放入不同的pool，一个分片只能包含一个pool中的文件，可以包括多个文件的Block。pool其实是针对文件进行了逻辑划分，不同的pool中的文件分别进行分片。分片的逻辑如下文所示。

2.2)分片的逻辑：

1.  如果指定了maxSplitSize(&#8220;mapreduce.input.fileinputformat.split.maxsize&#8221;)，那么在同一个节点上的Blocks合并，一个超过maxSplitSize就生成新分片。如果没有指定，则只汇总本节点BLock，暂不分片。
2.  如果指定了minSizeNode(&#8220;mapreduce.input.fileinputformat.split.minsize.per.node&#8221;),那么会把1.中处理剩余的Block，进行合并，如果超过minSizeNode，那么全部作为一个分片。否则这些Block与同一机架Rack上的块进行合并。
3.  每个节点上如上同样的方式处理，然后针对整个Rack的所有Block，按照1.方式处理。剩余部分，如果指定了minSizeRack(&#8220;mapreduce.input.fileinputformat.split.minsize.per.rack&#8221;)，并且超过minSizeRack，则全部作为一个分片，否则这些Block保留，等待与所有机架上的剩余Block进行汇总处理。
4.  每个机架上都按照1，2，3方式处理，汇总所有处理剩下的部分，再按照1的逻辑处理。再剩余的，作为一个分片。

以上逻辑我们可以知道：

如果只设置maxSplitSize(如job.getConfiguration().set( &#8220;mapreduce.input.fileinputformat.split.maxsize&#8221; , &#8220;33554432&#8243;))，那么基本每个分片大小都需凑满maxSplitSize。

**如果maxSplitSize，minSizeNode，minSizeRack三个都没有设置，那是所有输入整合成一个分片！**

**3)CombineFileRecordReader **

针对一个CombineFileSplit分片的通用RecordReader。CombineFileSplit中包含多个文件的块信息，CombineFileRecordReader是文件层面的处理，例如何时切换到分片中的下一个文件，而单个文件的处理，则需要自定义RecordReader的子类，读取文件的记录。

hadoop自带的示例应用org.apache.hadoop.examples.MultiFileWordCount，用到了CombineInputFormat，其处理流程：

[![MultiFileWordCount](http://www.sqlparty.com/wp-content/uploads/2013/12/MultiFileWordCount.png)](http://www.sqlparty.com/wp-content/uploads/2013/12/MultiFileWordCount.png)

要使用CombineInputFormat进行应用开发，可以参考org.apache.hadoop.examples.MultiFileWordCount中使用方式，需要自行实现CombineFileInputFormat的子类与实际读取逐条记录的RecordReader子类。

而MultiFileWordCount的使用如下：
```
shell&gt; hadoop fs -ls -h /tmp/carl/2013-07-12/

 Found 4 items

 -rw-r&#8211;r&#8211;   3 hdfs hadoop    246.6 M 2013-12-03 13:07 /tmp/carl/2013-07-12/000059_0

 -rw-r&#8211;r&#8211;   3 hdfs hadoop    244.9 M 2013-12-03 13:11 /tmp/carl/2013-07-12/000124_0

 -rw-r&#8211;r&#8211;   3 hdfs hadoop    244.9 M 2013-12-03 13:15 /tmp/carl/2013-07-12/000126_0

 -rw-r&#8211;r&#8211;   3 hdfs hadoop     85.2 M 2013-12-03 13:17 /tmp/carl/2013-07-12/000218_0

shell&gt; hadoop jar hadoop-mapreduce-examples-2.0.0-cdh4.4.0.jar multifilewc /tmp/carl/2013-07-12/ /tmp/carl/c8/

 &#8230;

 13/12/24 19:31:23 INFO input.FileInputFormat: Total input paths to process : 4

 13/12/24 19:31:23 INFO mapreduce.JobSubmitter: number of splits:1

 &#8230;

 Job Counters

 Killed reduce tasks=1

 Launched map tasks=1

 Launched reduce tasks=17

 Other local map tasks=1

 Total time spent by all maps in occupied slots (ms)=37923

 Total time spent by all reduces in occupied slots (ms)=1392681

 ...
```
因为MultiFileWordCount没有设置maxSplitSize，所以这里只有一个分片。

## 4.CombineInputFormat应用

### 4.1.使用场景

CombineInputFormat处理少量，较大的文件没有优势，相反，如果没有合理的设置maxSplitSize，minSizeNode，minSizeRack，则可能会导致一个map任务需要大量访问非本地的Block造成网络开销，反而比正常的非合并方式更慢。

而针对大量远小于块大小的小文件处理，CombineInputFormat的使用还是很有优势。

### 4.2.测试

我们以hadoop的示例程序WordCount和MulitFileWordCount来处理1000个小文件为例进行对比。

生成小文件：
```
shell&gt; cat file_gen.sh

 #!/bin/sh

 for i in $(seq 1000)

 do

 echo &#8220;abc asdf as df asd f sadf  werweiro &#8220;$i &gt; file_${i}

 done

 shell&gt; ./file_gen.sh

 shell&gt; ls -lh

 &#8230;

 -rw-r&#8211;r&#8211; 1 root root 40 Dec 25 10:14 file_962

 -rw-r&#8211;r&#8211; 1 root root 40 Dec 25 10:14 file_963

 -rw-r&#8211;r&#8211; 1 root root 40 Dec 25 10:14 file_964

 -rw-r&#8211;r&#8211; 1 root root 40 Dec 25 10:14 file_965

 -rw-r&#8211;r&#8211; 1 root root 40 Dec 25 10:14 file_966

 -rw-r&#8211;r&#8211; 1 root root 40 Dec 25 10:14 file_967

 &#8230;

上面生成大量小文件，上传这些小文件：

shell&gt; hadoop fs -put file* /tmp/carl2/

**wordcount**使用TextInputFormat方式，小文件一个个处理：

shell&gt; hadoop jar hadoop-mapreduce-examples-2.0.0-cdh4.4.0.jar wordcount /tmp/carl2/ /tmp/carl_result3/

 &#8230;

 13/12/25 10:34:25 INFO input.FileInputFormat: Total input paths to process : 1000

 13/12/25 10:34:27 INFO mapreduce.JobSubmitter: number of splits:1000

 &#8230;

 13/12/25 10:34:33 INFO mapreduce.Job:  map 0% reduce 0%

 13/12/25 10:34:40 INFO mapreduce.Job:  map 1% reduce 0%

 13/12/25 10:34:41 INFO mapreduce.Job:  map 2% reduce 0%

 &#8230;

 13/12/25 10:40:56 INFO mapreduce.Job:  map 100% reduce 94%

 13/12/25 10:40:57 INFO mapreduce.Job:  map 100% reduce 100%

 &#8230;

可以看到，生成了1000个map任务，总耗时超过6分钟!

**multifilewc**使用CombineInputFormat方式，没有设置maxSplitSize的情况下，所有小文件会汇总成一个Split。

shell&gt; hadoop jar hadoop-mapreduce-examples-2.0.0-cdh4.4.0.jar multifilewc /tmp/carl2/ /tmp/carl_result4/

 &#8230;

 13/12/25 10:42:04 INFO input.FileInputFormat: Total input paths to process : 1000

 13/12/25 10:42:07 INFO mapreduce.JobSubmitter: number of splits:1

 &#8230;

 13/12/25 10:42:12 INFO mapreduce.Job:  map 0% reduce 0%

 13/12/25 10:42:19 INFO mapreduce.Job:  map 100% reduce 0%

 13/12/25 10:42:24 INFO mapreduce.Job:  map 100% reduce 25%

 13/12/25 10:42:25 INFO mapreduce.Job:  map 100% reduce 31%

 13/12/25 10:42:27 INFO mapreduce.Job:  map 100% reduce 38%

 13/12/25 10:42:28 INFO mapreduce.Job:  map 100% reduce 56%

 13/12/25 10:42:29 INFO mapreduce.Job:  map 100% reduce 63%

 13/12/25 10:42:30 INFO mapreduce.Job:  map 100% reduce 69%

 13/12/25 10:42:31 INFO mapreduce.Job:  map 100% reduce 81%

 13/12/25 10:42:32 INFO mapreduce.Job:  map 100% reduce 88%

 13/12/25 10:42:33 INFO mapreduce.Job:  map 100% reduce 100%

```

可以看到，只用一个map任务进行处理，大量小文件可以使用网络迅速的汇总，总耗时不到30秒！

测试的结果可以大致看出，针对大量小文件，使用CombineInputFormat具有较大优势。

参考：

[http://stackoverflow.com/questions/14541759/how-can-i-work-with-large-number-of-small-files-in-hadoop](http://stackoverflow.com/questions/14541759/how-can-i-work-with-large-number-of-small-files-in-hadoop)

