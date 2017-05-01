---
title: mapreduce多目录输出笔记
date: 2016/11/26 22:23:21 
tags: MapReduce
categories: 大数据

---

### 使用MultipleOutputs实现多目录/文件输出
`org.apache.hadoop.mapreduce.lib.output.MultipleOutputs`
<!-- more -->
**在map或者reduce类中加入如下方法**
```
	private MultipleOutputs<Text, NullWritable> mos;

		@Override
		protected void setup(Reducer<Text, NullWritable, Text, NullWritable>.Context context)
				throws IOException, InterruptedException {
			// TODO Auto-generated method stub
			super.setup(context);
			mos = new MultipleOutputs<Text, NullWritable>(context);// 初始化mos
		}

		@Override
		protected void cleanup(Reducer<Text, NullWritable, Text, NullWritable>.Context context)
				throws IOException, InterruptedException {
			// TODO Auto-generated method stub
			super.cleanup(context);
			mos.close();
		}
```

**在需要输出数据的地方，可以使用定义好的 mos 进行输出**
```
mos.write("outputName", key, value);
mos.write("outputName", key, value, "filePrefix"); 
mos.write("outputName", key, value, "path/filePrefix");//到文件夹
```
**在Job Driver 时定义一些 Named Output**
```
MultipleOutputs.addNamedOutput(job, "outputXName",
    XXXOutputFormat.class, OutputXKey.class, OutputXValue.class);
MultipleOutputs.addNamedOutput(job, "outputYName",
    YYYOutputFormat.class, OutputYKey.class, OutputYValue.class);
```
**取消类似part-r-00000的空文件**
`LazyOutputFormat.setOutputFormatClass(job, TextOutputFormat.class)`
**例子**
```
package com.hdu.recommend.mr;

import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.LazyOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.MultipleOutputs;
import org.apache.hadoop.mapreduce.lib.output.NullOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;
import org.apache.hadoop.yarn.conf.YarnConfiguration;


 * @author Skye
 *
 */
public class DataCleanIconAndWeb {
	public static class QLMapper extends
			Mapper<LongWritable, Text, Text, NullWritable> {


		private String webGame = "网页游戏";

		Text outputValue = new Text();
		// 设置多文件输出
		private MultipleOutputs<Text,NullWritable> mos;
		@Override
		protected void setup(Mapper<LongWritable, Text, Text, NullWritable>.Context context)
				throws IOException, InterruptedException {
			// TODO Auto-generated method stub
			super.setup(context);
			mos = new MultipleOutputs<Text, NullWritable>(context);// 初始化mos
		}
		@Override
		protected void cleanup(Mapper<LongWritable, Text, Text, NullWritable>.Context context)
				throws IOException, InterruptedException {
			// TODO Auto-generated method stub
			super.cleanup(context);
			mos.close();
		}
		@Override
		protected void map(LongWritable key, Text value, Context context)
				throws IOException, InterruptedException {
			// 接收数据v1
			String line = value.toString();
			// 切分数据
			String[] words = line.split("");
			// String[] words = line.split("\t");
			boolean isWeb = false;
			boolean flag = true;
			
			//一系列处理代码
			//***
			//***
			//***
			String action = words[1] + "\t" + words[0] + "\t" + words[2]
						+ "\t" + words[3] + "\t" + words[5];

			outputValue.set(action);
			mos.write("iconRecord", outputValue, NullWritable.get(),"iconRecord/icon");
			
			
	
			String action = words[1] + "\t" + words[0] + "\t"
							+ words[2] + "\t" + words[3] + "\t" + words[4]
							+ "\t" + words[5];

			outputValue.set(action);
			mos.write( "webRecord",outputValue, NullWritable.get(),"webRecord/web");
				

			
		}

	}

	

	public static void run(String originalDataPath, String dataCleanOutputFile)
			throws Exception {

		// 构建Job对象
		Configuration conf = new Configuration();

		Job job = Job.getInstance(conf);

		// 注意：main方法所在的类
		job.setJarByClass(DataCleanIconAndWeb.class);
		job.getConfiguration().setBoolean("mapreduce.output.fileoutputformat.compress", false);
		job.getConfiguration().setStrings(
				"mapreduce.reduce.shuffle.input.buffer.percent", "0.1");
		job.getConfiguration().setBoolean("mapreduce.output.fileoutputformat.compress", false);
		job.setNumReduceTasks(3);

		// 设置Mapper相关属性
		job.setMapperClass(QLMapper.class);
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(NullWritable.class);

		
		FileInputFormat.setInputPaths(job, new Path(originalDataPath));

		

		// 设置Reducer相关属性
		
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(NullWritable.class);

		FileOutputFormat.setOutputPath(job, new Path(dataCleanOutputFile));
		
		MultipleOutputs.addNamedOutput(job, "iconRecord",
				TextOutputFormat.class, Text.class, NullWritable.class);
		MultipleOutputs.addNamedOutput(job, "webRecord",
				TextOutputFormat.class, Text.class, NullWritable.class);
		
		// 文件格式
		job.setInputFormatClass(TextInputFormat.class);
		//取消part-r-00000新式文件输出
		LazyOutputFormat.setOutputFormatClass(job, TextOutputFormat.class);
		
		
		//job.setOutputFormatClass(TextOutputFormat.class);
		// 提交任务
		job.waitForCompletion(true);

		long endTime = System.currentTimeMillis();

	}
 
}

```

> 参考
> http://gnailuy.com/dataplatform/2015/11/22/common-techniques-for-mapreduce/
> http://blog.csdn.net/zgc625238677/article/details/51524786
> https://www.iteblog.com/archives/848