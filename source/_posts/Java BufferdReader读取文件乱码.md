---
title: Java BufferdReader读取文件乱码
date: 2016/8/29 21:23:57 
tags: Java
categories: Java

---

### javaBufferdReader读取文件乱码

以下为读取文件方法

```
	private static void putIdGame(){
		
		URL.setURLStreamHandlerFactory(new FsUrlStreamHandlerFactory());
		int count = 0;
		try {

			
			URL url = new URL(HDFS + path);
			InputStream gameList = url.openStream();
			BufferedReader reader_url = new BufferedReader(new InputStreamReader(gameList,"UTF-8"));
			String inString_RL = reader_url.readLine();
			
			while (inString_RL != null && count < 50) {
				int userId;
				String[] str = inString_RL.split(","); 
				count ++;
				map.put(str[1], str[0]);
				System.out.println(str[0]);
				inString_RL = reader_url.readLine();
			}
			reader_url.close();
		} catch (FileNotFoundException e) {
			System.out.println("未找文件！");
		} catch (IOException e1) {
			System.out.println("文件读写错误！");
		}
	}
```

在`InputStreamReader`中加入"UTF-8"即可

### Java读取文件时第一行出现乱码“？”问号

在windows 环境下，使用java文件流读取文本文件时，会出现第一个字符为未知字符"?" ,其他字符完整。而且第一个字符显示为？但是用equals比对发现并非是"?"号,google之，了解到bom编码标记。使用 16进制打印输出结果：

只要出现该头的16进制编码为这种字符便可以断定该文本文件的编码方式了。



bom编码标记：

bom全称是：byte order mark，汉语意思是标记字节顺序码。只是出现在：unicode字符集中，只有unicode字符集，存储时候，要求指定编码，如果不指定，windows还会用默认的：ANSI读取。常见的bom头是：

  UTF-8 ║ EF BB BF 
  UTF-16LE ║ FF FE (小尾）
  UTF-16BE ║ FE FF （大尾）
  UTF-32LE ║ FF FE 00 00 
  UTF-32BE ║ 00 00 FE FF


#### 解决方法：
1. 工具将txt文件另存为UTF-8无BOM格式

2. 

```
public String readerFile(InputStream in) throws IOException {
		StringBuffer strBuff = new StringBuffer();
		String temp = null;
		BufferedReader reader = new BufferedReader(new InputStreamReader(in,Charset.forName("utf-8")));
		while ((temp = reader.readLine()) != null) {
			byte[] by = temp.getBytes();
			String header = Integer.toHexString(by[0]).toUpperCase();
			//判断是否拥有无法识别的字符
			if (header.equalsIgnoreCase("FFFFFFEF") || header.equalsIgnoreCase("3F")) {
				strBuff.append(temp.substring(1) + "\n");
				continue;
			}
			strBuff.append(temp + "\n");
		}
		reader.close();
		in.close();
		return strBuff.toString();
	}
```
