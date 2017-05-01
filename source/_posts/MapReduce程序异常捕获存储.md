---
title: MapReduce程序异常捕获存储
date: 2016/12/16 15:24:29 
tags: MapReduce
categories: 大数据

---

在Map或reduce中使用multipleOutput来进行异常存储：
```
try{
	...
	...
	context.write(newKey,newValue);

}catch(Exception e){
	multipleOutput.write(new Text( null == e.getMessage()? ("error:"): e.getMessage),new Text(value.toString()),"_error/part");
	e.printStackTrace();
}

```