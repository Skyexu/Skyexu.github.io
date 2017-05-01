---
title: Git SSH公钥配置
date: 2016/10/28 16:28:47 
tags: Git
categories: Git

---
> Coding.net配置

### 账户 SSH 公钥
账户 SSH 公钥是跟用户账户关联的公钥，一旦设置，SSH 就拥有账户下所有项目仓库的读写权限。 设置“账户 SSH 公钥”是开发者使用 SSH 方式访问/修改代码仓库的“前置工作”，分为“获取 SSH 协议地址”、“生成公钥”、“在 Coding.net 添加公钥”三个步骤。
<!-- more -->
### 获取 SSH 协议地址
在项目的代码页面点击 SSH 切换到 SSH 协议， 获得 clone 地址，形如`git@git.coding.net:wzw/leave-a-message.git`。 请使用这个地址来访问您的代码仓库。

### 生成公钥
Mac/Linux 打开命令行终端, Windows 打开 Git Bash 。 输入`ssh-keygen -t rsa -C "username@example.com"`,( 注册的邮箱)，接下来点击enter键即可（也可以输入密码）。
```
$ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
# Creates a new ssh key, using the provided email as a label
# Generating public/private rsa key pair.
Enter file in which to save the key (/Users/you/.ssh/id_rsa): [Press enter]  // 推荐使用默认地址,如果使用非默认地址可能需要配置 .ssh/config
成功之后

Your identification has been saved in /Users/you/.ssh/id_rsa.
# Your public key has been saved in /Users/you/.ssh/id_rsa.pub.
# The key fingerprint is:
# 01:0f:f4:3b:ca:85:d6:17:a1:7d:f0:68:9d:f0:a2:db your_email@example.com
```
### 在 Coding.net 添加公钥
本地打开 id_rsa.pub 文件（或执行 $cat id_rsa.pub ），复制其中全部内容，添加到账户“SSH 公钥”页面 中，公钥名称可以随意起名字。
完成后点击“添加”，然后输入密码或动态码即可添加完成。

### 完成后在命令行测试，首次建立链接会要求信任主机
```
$ ssh -T git@git.coding.net // 注意 git.coding.net 接入到 CDN 上所以会解析多个不同的 host ip The authenticity of host ‘git.coding.net (61.146.73.68)’ can not be established. RSA key fingerprint is 98:ab:2b:30:60:00:82:86:bb:85:db:87:22:c4:4f:b1. Are you sure you want to continue connecting (yes/no)? yes Warning: Permanently added ‘git.coding.net,61.146.73.68’ (RSA) to the list of kn own hosts.

Enter passphrase for key ‘/c/Users/Yuankai/.ssh/id_rsa’: Coding.net Tips : [ Hello Kyle_lyk! You have connected to Coding.net by SSH successfully! ] 
```

接下来就可以用git操作仓库了，是使用SourceTree非常方便