---
title: ubuntu相关设置
date: 2016-08-15 00:39:26
tags: Linux技巧
categories: Linux

---

### 修改时区
`sudo dpkg-reconfigure tzdata`

出现时区列表，按照提示选择“Asia/Shanghai”

结果如下
```
Package configuration

     ┌───────────────────────┤ Configuring tzdata ├───────────────────────┐
     │ Please select the city or region corresponding to your time zone.  │
     │                                                                    │
     │ Time zone:                                                         │
     │                                                                    │
     │                          Qyzylorda        ↑                        │
     │                          Rangoon          ▒                        │
     │                          Riyadh           ▒                        │
     │                          Sakhalin         ▒                        │
     │                          Samarkand        ▒                        │
     │                          Seoul            ▒                        │
     │                          Shanghai         ▮                        │
     │                          Singapore        ▒                        │
     │                          Srednekolymsk    ▒                        │
     │                          Taipei           ↓                        │
     │                                                                    │
     │                                                                    │
     │                 <Ok>                     <Cancel>                  │
     │                                                                    │
     └────────────────────────────────────────────────────────────────────┘

                                                                               
Current default time zone: 'Asia/Shanghai'
Local time is now:      Mon Aug 15 00:11:59 CST 2016.
Universal Time is now:  Sun Aug 14 16:11:59 UTC 2016.

ubuntu@master:~$ date
Mon Aug 15 00:12:13 CST 2016
```