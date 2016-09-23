---
title: MapReduce 多文件输入
date: 2016-06-16 16:26:16
tags: MapReduce
categories: 大数据

---

## 多路径输入
1.  FileInputFormat.addInputPath 多次调用加载不同路径
```
FileInputFormat.addInputPath(job, new Path(args[0]));
FileInputFormat.addInputPath(job, new Path(args[1]));
```
2.  FileInputFormat.addInputPaths一次调用加载 多路径字符串用逗号隔开
```
FileInputFormat.addInputPaths(job, "hdfs://master:9000/cs/path1,hdfs://RS5-112:9000/cs/path2");
```
3. 多种输入**MultipleInputs可以加载不同路径的输入文件，并且每个路径可用不同的
```
maperMultipleInputs.addInputPath(job, new Path("hdfs://master:9000/cs/path1"), TextInputFormat.class,MultiTypeFileInput1Mapper.class);
MultipleInputs.addInputPath(job, new Path("hdfs://master:9000/cs/path3"), TextInputFormat.class,MultiTypeFileInput3Mapper.class);
```

## 网上例子：

	package example;
	
	import java.io.IOException;
	
	import org.apache.hadoop.conf.Configuration;
	import org.apache.hadoop.fs.Path;
	import org.apache.hadoop.io.LongWritable;
	import org.apache.hadoop.io.Text;
	import org.apache.hadoop.mapreduce.Job;
	import org.apache.hadoop.mapreduce.Mapper;
	import org.apache.hadoop.mapreduce.Reducer;
	import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
	import org.apache.hadoop.mapreduce.lib.input.MultipleInputs;
	import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
	import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
	/**
	* 多类型文件输入
	* @author lijl
	*
	*/
	
	public class MultiTypeFileInputMR {
	static class MultiTypeFileInput1Mapper extends Mapper<LongWritable, Text, Text, Text>{
	public void map(LongWritable key,Text value,Context context){
	try {
	String[] str = value.toString().split("\\|");
	context.write(new Text(str[0]), new Text(str[1]));
	} catch (IOException e) {
	e.printStackTrace();
	} catch (InterruptedException e) {
	e.printStackTrace();
	}
	}
	}
	static class MultiTypeFileInput3Mapper extends Mapper<LongWritable, Text, Text, Text>{
	public void map(LongWritable key,Text value,Context context){
	try {
	String[] str = value.toString().split("");
	context.write(new Text(str[0]), new Text(str[1]));
	} catch (IOException e) {
	e.printStackTrace();
	} catch (InterruptedException e) {
	e.printStackTrace();
	}
	}
	}
	static class MultiTypeFileInputReducer extends Reducer<Text, Text, Text, Text>{
	public void reduce(Text key,Iterable<Text> values,Context context){
	try {
	for(Text value:values){
	context.write(key,value);
	}
	
	} catch (IOException e) {
	e.printStackTrace();
	} catch (InterruptedException e) {
	e.printStackTrace();
	}
	}
	}
	
	public static void main(String[] args) throws IOException, InterruptedException, ClassNotFoundException {
	Configuration conf = new Configuration();
	conf.set("mapred.textoutputformat.separator", ",");
	Job job = new Job(conf,"MultiPathFileInput");
	job.setJarByClass(MultiTypeFileInputMR.class);
	FileOutputFormat.setOutputPath(job, new Path("hdfs://RS5-112:9000/cs/path6"));
	
	job.setMapOutputKeyClass(Text.class);
	job.setMapOutputValueClass(Text.class);
	job.setOutputKeyClass(Text.class);
	job.setOutputValueClass(Text.class);
	
	job.setReducerClass(MultiTypeFileInputReducer.class);
	job.setNumReduceTasks(1);
	MultipleInputs.addInputPath(job, new Path("hdfs://RS5-112:9000/cs/path1"), TextInputFormat.class,MultiTypeFileInput1Mapper.class);
	MultipleInputs.addInputPath(job, new Path("hdfs://RS5-112:9000/cs/path3"), TextInputFormat.class,MultiTypeFileInput3Mapper.class);
	System.exit(job.waitForCompletion(true)?0:1);
	}
	
	}

## 自己例子
QLMapper.java
```
package com.hdu.mr;

import java.io.IOException;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class QLMapper extends Mapper<LongWritable, Text, Text, LongWritable> {
	String[] mbUnlike = { "盒子", "助手", "输入法", "平台" };
	String mbdylxLike = "游戏";
	String mbdylxUnlike = "网页游戏";
	String delWeb = "访问网站";
	Text outputValue = new Text();

	@Override
	protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, LongWritable>.Context context)
			throws IOException, InterruptedException {
		// 接收数据v1
		String line = value.toString();
		// 切分数据
		String[] words = line.split("");
		// String[] words = line.split("\t");

		boolean flag = true;

		for (int i = 0; i < 4; i++) {
			if (words.length < 5) { // 过滤 长度小于4的信息 即访问网站等
				flag = false;
				break;
			}
			if (words[3].indexOf(mbUnlike[i]) != -1) { // 有其中一个则为false
				flag = false;
				break;
			}
		}

		if (flag == true) {
			if (words[4].indexOf(mbdylxUnlike) != -1) { // 有网页游戏则为false
				flag = false;
			} else if (words[4].indexOf(mbdylxLike) == -1) { // 没有游戏则为false
				flag = false;
			}
		}
		if (flag == true) {
			outputValue.set(line);
			context.write(outputValue, new LongWritable(1L));
		}
	}

}

```

QLReducer.java
```
package com.hdu.mr;

import java.io.IOException;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class QLReducer extends Reducer<Text, LongWritable, Text, NullWritable> {

	@Override
	protected void reduce(Text key, Iterable<LongWritable> values,
			Reducer<Text, LongWritable, Text, NullWritable>.Context context) throws IOException, InterruptedException {
		// 接收数据

		// 输出
		context.write(key, NullWritable.get());
	}
}

```

DataClean.java
```
package com.hdu.mr;

import java.io.IOException;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class QLReducer extends Reducer<Text, LongWritable, Text, NullWritable> {

	@Override
	protected void reduce(Text key, Iterable<LongWritable> values,
			Reducer<Text, LongWritable, Text, NullWritable>.Context context) throws IOException, InterruptedException {
		// 接收数据

		// 输出
		context.write(key, NullWritable.get());
	}
}
```