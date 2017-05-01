---
title: MapReduce 递归子目录和合并小文件
date: 2016/12/12 21:26:41 
tags: MapReduce
categories: 大数据

---

## 递归子目录

设置`mapreduce.input.fileinputformat.input.dir.recursive=true`，这个参数是客户端参数，可以在MapReduce中设置，也可以在mapred-site.xml中设置.在mapreduce程序中如
```
// 递归子目录
job.getConfiguration().setBoolean("mapreduce.input.fileinputformat.input.dir.recursive",true);
```

## CombineTextInputFormat 合并小文件
```
//设置split大小
job.getConfiguration().setLong("mapreduce.input.fileinputformat.split.maxsize", 128 * 1024 * 1024);

//job.setInputFormatClass(TextInputFormat.class);
		
// 合并小文件
job.setInputFormatClass(CombineTextInputFormat.class);
```