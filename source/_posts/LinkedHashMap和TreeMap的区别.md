---
title: LinkedHashMap和TreeMap的区别
date: 2016/10/18 21:53:44    
tags: Java
categories: Java

---

### LinkedHashMap和TreeMap的区别

首先2个都是map，所以用key取值肯定是没区别的，区别在于用Iterator遍历的时候
LinkedHashMap保存了记录的插入顺序，先插入的先遍历到
TreeMap默认是按升序排，也可以指定排序的比较器。遍历的时候按升序遍历。
例如：a是LinkedHashMap，b是TreeMap。
a.put("2","ab");
a.put("1","bc");
b.put("2","ab");
b.put("1","bc");

那么遍历a的时候，先遍历到key是2的，因为2先放进去。
遍历b的时候，先遍历到“1”，因为按顺序是先1后2