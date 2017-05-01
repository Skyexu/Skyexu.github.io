---
title: Java获取与当天相差几天的日期两种方式
date: 2016/12/9 21:32:27 
tags: Java
categories: Java

---


```
Date date=new Date();//取时间  
Calendar calendar = new GregorianCalendar();  
calendar.setTime(date);  
calendar.add(calendar.DATE,1);//把日期往后增加一天.整数往后推,负数往前移动  
date=calendar.getTime(); //这个时间就是日期往后推一天的结果   
SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd");  
String dateString = formatter.format(date);  
System.out.println(dateString);  
```


```
/** 
     *获取两日期之间天数 
     */  
    public String getDate(Date d,long i){  
         SimpleDateFormat df=new SimpleDateFormat("yyyy-MM-dd");     
         /*System.out.println("今天的日期："+df.format(d));    
         System.out.println("两天前的日期：" + df.format(new Date(d.getTime() - 2 * 24 * 60 * 60 * 1000)));   
         System.out.println("三天后的日期：" + df.format(new Date(d.getTime() + 3 * 24 * 60 * 60 * 1000)));*/  
         return df.format(new Date(d.getTime() + i * 24 * 60 * 60 * 1000));  
    }  
```