---
title: Java读取properties配置文件
date: 2017/2/20 12:24:33  
tags: Java
categories: Java

---


读取.properties配置文件在实际的开发中使用的很多，总结了一下，有以下几种方法（仅仅是我知道的）：

### 通过jdk提供的java.util.Properties类

此类继承自java.util.HashTable，即实现了Map接口，所以，可使用相应的方法来操作属性文件，但不建议使用像put、putAll这两个方法，因为put方法不仅允许存入String类型的value，还可以存入Object类型的。因此java.util.Properties类提供了getProperty()和setProperty()方法来操作属性文件，同时使用store或save(已过时)来保存属性值（把属性值写入.properties配置文件）。在使用之前，还需要加载属性文件，它提供了两个方法：load和loadFromXML。load有两个方法的重载：load(InputStream inStream)、load(Reader reader)，所以，可根据不同的方式来加载属性文件。可根据不同的方式来获取InputStream，如：

1. 通过当前类加载器的getResourceAsStream方法获取
```
InputStream inStream = TestProperties.class.getClassLoader().getResourceAsStream("test.properties");  
```
2. 从文件获取
```
InputStream inStream = new FileInputStream(new File("filePath"));  
```

<!-- more -->

3. 也是通过类加载器来获取，和第一种一样
```
InputStream in = ClassLoader.getSystemResourceAsStream("filePath");  
```

4. 在servlet中，还可以通过context来获取InputStream
```
InputStream in = context.getResourceAsStream("filePath");  
```

5. 通过URL来获取
```
URL url = new URL("path");  
InputStream inStream = url.openStream();  



```
读取方法如下：
```
Properties prop = new Properties();  
prop.load(inStream);  
String key = prop.getProperty("username");  
//String key = (String) prop.get("username");
```

### 通过java.util.ResourceBundle类来读取

这种方式比使用Properties要方便一些。

1. 通过ResourceBundle.getBundle()静态方法来获取（ResourceBundle是一个抽象类），这种方式来获取properties属性文件不需要加.properties后缀名，只需要文件名即可。
```
ResourceBundle resource = ResourceBundle.getBundle("com/mmq/test");
// test为属性文件名，放在包com.mmq下，如果是放在src下，直接用test即可
```
String key = resource.getString("username");  

2. 从InputStream中读取，获取InputStream的方法和上面一样，不再赘述。
```
ResourceBundle resource = new PropertyResourceBundle(inStream);
```

注意：在使用中遇到的最大的问题可能是配置文件的路径问题，如果配置文件入在当前类所在的包下，那么需要使用包名限定，如：test.properties入在com.mmq包下，则要使用com/mmq/test.properties（通过Properties来获取）或com/mmq/test（通过ResourceBundle来获取）；属性文件在src根目录下，则直接使用test.properties或test即可。


> 本文转载自 http://blog.csdn.net/mhmyqn/article/details/7683909