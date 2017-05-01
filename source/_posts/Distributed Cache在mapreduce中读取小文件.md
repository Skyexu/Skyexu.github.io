---
title: Distributed Cache在mapreduce中读取小文件
date: 2016/11/26 22:48:13  
tags: MapReduce
categories: 大数据

---

Distributed Cache 在 MapReduce 任务中应用很广， 它可以大大提高一些被频繁读取文件的访问速度。被添加到 Distributed Cache 的文件会被拷贝到 Mapper 和 Reducer 的运行目录中。

**在job添加如下方法 **
```
remoteReGamePath为hdfs文件路径字符串
job.addCacheFile(new Path(remoteReGamePath).toUri());
```
<!-- more -->
**以下例子为在map中读取此文件并存入集合**
```
private Set<String> recommendGame = new HashSet<String>();
/**
		 * 读取推荐游戏文件
		 * 
		 * @param uri
		 */
		private void readReGame(URI uri) {
			try {
				Path patternsPath = new Path(uri.getPath());
				String patternsFileName = patternsPath.getName().toString();
				BufferedReader reader = new BufferedReader(new FileReader(
						patternsFileName));
				String line;
				while ((line = reader.readLine()) != null) {
					// TODO: your code here
					//
					recommendGame.add(line.split(",")[0]);
				}
				reader.close();

			} catch (FileNotFoundException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}

		}

		@Override
		protected void setup(
				Mapper<LongWritable, Text, Text, Text>.Context context)
				throws IOException, InterruptedException {
			// TODO Auto-generated method stub
			super.setup(context);
			//获取cache  uri
			URI[] uri = context.getCacheFiles();

			readReGame(uri[0]);

		}


```
