---
title: JVM内存配置
date: 2016/7/26 15:10:59 
tags: 
- JVM 
- eclipse
categories: Java

---

参数中-vmargs的意思是设置JVM参数，所以后面的其实都是JVM的参数了，我们首先了解一下JVM内存管理的机制，然后再解释每个参数代表的含义。

### 堆(Heap)和非堆(Non-heap)内存

按照官方的说法：“Java虚拟机具有一个堆，堆是运行时数据区域，所有类实例和数组的内存均从此处分配。堆是在Java虚拟机启动时创建的。”“在JVM中堆之外的内存称为非堆内存(Non-heapmemory)”。可以看出JVM主要管理两种类型的内存：堆和非堆。简单来说堆就是Java代码可及的内存，是留给开发人员使用的；非堆就是JVM留给自己用的，所以方法区、JVM内部处理或优化所需的内存(如JIT编译后的代码缓存)、每个类结构(如运行时常数池、字段和方法数据)以及方法和构造方法的代码都在非堆内存中。

### 堆内存分配

JVM初始分配的内存由-Xms指定，默认是物理内存的1/64；JVM最大分配的内存由-Xmx指定，默认是物理内存的1/4。默认空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制；空余堆内存大于70%时，JVM会减少堆直到-Xms的最小限制。因此服务器一般设置-Xms、-Xmx相等以避免在每次GC后调整堆的大小。

非堆内存分配

JVM使用-XX:PermSize设置非堆内存初始值，默认是物理内存的1/64；由XX:MaxPermSize设置最大非堆内存的大小，默认是物理内存的1/4。

### JVM内存限制(最大值)

首先JVM内存限制于实际的最大物理内存(废话！呵呵)，假设物理内存无限大的话，JVM内存的最大值跟操作系统有很大的关系。简单的说就32位处理器虽然可控内存空间有4GB,但是具体的操作系统会给一个限制，这个限制一般是2GB-3GB（一般来说Windows系统下为1.5G-2G，Linux系统下为2G-3G），而64bit以上的处理器就不会有限制了。

### 减少jvm内存回收引起的eclipse卡的问题 

这个主要是jvm在client模式，进行内存回收时，会停下所有的其它工作，带回收完毕才去执行其它任务，在这期间eclipse就卡住了。所以适当的增加jvm申请的内存大小来减少其回收的次数甚至不回收，就会是卡的现象有明显改善。 

主要通过以下的几个jvm参数来设置堆内存的： 
- -Xmx512m	最大总堆内存，一般设置为物理内存的1/4
- -Xms512m	初始总堆内存，一般将它设置的和最大堆内存一样大，这样就不需要根据当前堆使用情况而调整堆的大小了
- -Xmn192m	年轻带堆内存，sun官方推荐为整个堆的3/8
- 堆内存的组成	总堆内存 = 年轻带堆内存 + 年老带堆内存 + 持久带堆内存
- 年轻带堆内存	对象刚创建出来时放在这里
- 年老带堆内存	对象在被真正会回收之前会先放在这里
- 持久带堆内存	class文件，元数据等放在这里
- -XX:PermSize=128m	持久带堆的初始大小
- -XX:MaxPermSize=128m	持久带堆的最大大小，eclipse默认为256m。如果要编译jdk这种，一定要把这个设的很大，因为它的类太多了。

### eclipse运行配置

Eclipse -> Run -> Run Configurations -> Arguments -> VM arguments
或者 Run as -> Run Configurations -> Arguments -> VM arguments
 -Xms2048m -Xmx2048m 
![](http://i.imgur.com/O2JUIKO.png)

-Xms是设置内存初始化的大小(如上面的2048m)
-Xmx是设置最大能够使用内存的大小（如上面的2048m, 最好不要超过物理内存）
也可通过 eclipse.ini配置

参考文章：
[Eclipse中进行JVM内存设置](http://blog.csdn.net/gf771115/article/details/20220915)
[JVM监控与调优](http://blog.csdn.net/buptdavid/article/details/43270997)
[JVM调优总结(这个总结得比较全面)](http://blog.csdn.net/wuzhilon88/article/details/49201891)
[JVM性能调优](http://uule.iteye.com/blog/2114697)