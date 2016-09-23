---
title: 亚马逊EC2搭建Shadowsocks
date: 2016-07-19 11:26:26
tags: 科学上网
categories: 计算机技巧

---

在网上寻找优秀的科学上网方式中发现，自己搭建Shadowscoks是非常不错的方式，并且AWS有一年的免费使用期限，到期后再买别的VPS就行了。
[https://segmentfault.com/a/1190000003101075](https://segmentfault.com/a/1190000003101075)
上面连接的教程从注册申请AWS到搭建服务已经非常详细，自己再做一些补充。

## 服务器端配置

在EC2上创建好Linux服务器后（本人为Ubuntu）需要对其安装环境
参考官方文档[使用 PuTTY 从 Windows 连接到 Linux 实例](http://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/putty.html)连接服务器

### 安装shadowsocks依赖
1. `sudo -s // 获取超级管理员权限`
2. `apt-get update // 更新apt-get`
3. `apt-get install python-pip // 安装python包管理工具pip`
4. `pip install shadowsocks // 安装shadowsocks`

### 配置shadowsocks

`vim /etc/shadowsocks.json`

单一端口配置
```
{
"server":"server_ip", #EC2实例的IP，注意这里我们不能填写公有IP，需要填写私有IP或者0.0.0.0  填0.0.0.0即可
"server_port":8388, #server端监听的端口，需要在EC2实例中开放此端口
"local_address": "127.0.0.1",
"local_port":1080,
"password":"password", #密码
"timeout":300,
"method":"aes-256-cfb", #加密方式
"fast_open": false #是否开启fast open
}
```
如果想要把VPN分享给其它人而不泄露自己的密码，也可以在配置文件中设置多端口+多密码的模式，如：
```
{
"server":"server_ip", #EC2实例的IP，注意这里我们不能填写公有IP，需要填写私有IP或者0.0.0.0
"local_address": "127.0.0.1",
"local_port":1080,
"port_password":
{
"8088”: “password8088”,
"8089”: "password8089”
}
"timeout":300,
"method":"aes-256-cfb", #加密方式
"fast_open": false #是否开启fast open
}
```

### 配置完成后启动Shawdowsocks

```
启动：ssserver -c /etc/shadowsocks.json -d start
停止：ssserver -c /etc/shadowsocks.json -d stop
重启：ssserver -c /etc/shadowsocks.json -d restart
查看状态：ssserver -c /etc/shadowsocks.json -d status
```

### 关闭服务器防火墙

`sudo ufw disable`

### 开启AWS入站端口
配置好shaodowsocks后，还需要将配置中的端口打开,这样客户端的服务才能链接得上EC2中的shadowsocks服务。
在EC2网页中编辑入站规则将配置文件中的端口号（如8388）加入入站规则。
服务器端配置完毕。

## 客户端下载
[shadowsocks下载地址1](https://www.gaotizi.com/knowledgebase/2/shadowsocks.html)
[shadowsocks下载地址2](https://shadowsocks.com/client.html)
windows10尽量使用2.3版本，否则可能出现500或者502错误
http://pan.baidu.com/s/1hqIk4mS


### 500或者502错误
- 使用2.3版本
- 或者尝试以下命令
```
netsh interface ipv4  reset
netsh interface ipv6  reset
netsh winsock reset
```