---
title: Ubuntu16.04安装Oracle11g
date: 2016-08-17  23:36:50 
tags: Oracle
categories: Linux

---

### Oracle用户创建
```
sudo groupadd oinstall
sudo groupadd dba
sudo adduser -g oinstall -G dba -s  oracle
sudo passwd oracle
```

### 修改/etc/sysctl.conf增加以下内容
```
kernel.sem = 250 32000 100 128
kernel.shmall = 2097152
kernel.shmmni = 4096
kernel.shmmax=1073741824
net.ipv4.ip_local_port_range = 9000  65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
fs.aio-max-nr = 1048576
fs.file-max = 6815744
vm.hugetlb_shm_group = 1002

```

### 运行一下命令更新内核参数
`sudo sysctl -p`

### 修改/etc/security/limits.conf增加以下内容
```
oracle soft nproc  2047
oracle hard nproc  16384
oracle soft nofile 1024
oracle hard nofile 65536
oracle soft stack  10240
```

### 修改/etc/pam.d/login增加以下内容

```
session required /lib/security/pam_limits.so
session required pam_limits.so
```

### 欺骗oracle的安装程序

oracle本身并不支持ubuntu来安装，所以要进行欺骗oracle的安装程序（sudo执行）：
```
mkdir /usr/lib64
ln -s /etc /etc/rc.d
ln -s /lib/x86_64-linux-gnu/libgcc_s.so.1 /lib64/
ln -s /usr/bin/awk /bin/awk
ln -s /usr/bin/basename /bin/basename
ln -s /usr/bin/rpm /bin/rpm
ln -s /usr/lib/x86_64-linux-gnu/libc_nonshared.a /usr/lib64/
ln -s /usr/lib/x86_64-linux-gnu/libpthread_nonshared.a /usr/lib64/
ln -s /usr/lib/x86_64-linux-gnu/libstdc++.so.6 /lib64/
ln -s /usr/lib/x86_64-linux-gnu/libstdc++.so.6 /usr/lib64/
vim /etc/redhat-release
echo 'Red Hat Linux release 5' > /etc/redhat-release
```

### 为Oracle配置环境变量
```
#oracle安装目录，第6步创建的文件夹
export ORACLE_BASE=/home/oracle/oracle11g
#网上说可以随便写
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/dbhome_1
#数据库的sid
export ORACLE_SID=orcl
export ORACLE_UNQNAME=orcl
#默认字符集
export NLS_LANG=.AL32UTF8
#环境变量
export PATH=${PATH}:${ORACLE_HOME}/bin/:$ORACLE_HOME/lib64;
```

### 安装oracle
上面的系统配置完成之后，最好重启一下服务器，**使用oracle用户登陆系统**。
1. 上传下载好的oracle压缩文件到/home/oracle目录下。
2. 进入/home/oracle目录，执行# unzip linux.x64_11gR2_database_1of2.zip和# unzip linux.x64_11gR2_database_2of2.zip，解压的文件在/home/oracle/database目录中。
3. 设置/home/oracle/database目录的权限：
```
# chown oracle:oinstall /home/oracle/database -R
# chmod 775 /home/oracle/database -R
```
4. 进入/home/oracle/database目录，执行$ ./runInstaller，当检查均通过，会出现oracle安装界面,一路next，有一步可以选择字符，选utf8

### 安装过程可能遇到的问题

- Oracle安装界面乱码解决方法 
执行:
```
exportNLS_LANG=AMERICAN_AMERICA.UTF8
export LC_ALL=C
```
- Error in invoking target ‘install’ of makefile ‘/home/dong/tools/oracle11g/product/11.2.0/dbhome_1/ctx/lib/ins_ctx.mk’. See ‘/home/dong/tools/oraInventory/logs/installActions2015-01-22_09-39-03AM.log’ for details.

 解决方法：

 从http://download.csdn.net/detail/adnerly/9467935下载，使用rpm安装这个glibc-static-2.17-55.el7.x86_64.rpm资源，安装即可， 然后点击retry ，接着往下执行 
 注:这是网上提供的解决方案，我的系统安装失败，我直接跳过了 


- Error in invoking target ‘agent nmhs’ of makefile ‘/home/dong/tools/oracle11g/product/11.2.0/dbhome_1/sysman/lib/ins_emagent.mk’

 解决方法：

 打开新的终端窗口 
 使用vi命令，打开`/home/oracle/oracle11g/product/11.2.0/dbhome_1/sysman/lib/ins_emagent.mk`文件，将$(MK_EMAGENT_NMECTL)修改成$(MK_EMAGENT_NMECTL)-lnnz11 即可，然后点击retry ，接着往下执行



- Error in invoking target ‘all_no_orcl’ of makefile ‘/home/oracle/oracle11g/product/11.2.0/dbhome_1/rdbms/lib/ins_rdbms.mk’. See ‘/home/dong/tools/Inventory/logs/installActions2016-03-19_02-37-44PM.log’ for details.

 解决办法：

 打开一个新的终端，输入如下四个命令：

```
sed -i 's/^\(TNSLSNR_LINKLINE.*\$(TNSLSNR_OFILES)\) \(\$(LINKTTLIBS)\)/\1 -Wl,--no-as-needed \2/g' $ORACLE_HOME/network/lib/env_network.mk

sed -i 's/^\(ORACLE_LINKLINE.*\$(ORACLE_LINKER)\) \(\$(PL_FLAGS)\)/\1 -Wl,--no-as-needed \2/g' $ORACLE_HOME/rdbms/lib/env_rdbms.mk

sed -i 's/^\(\$LD \$LD_RUNTIME\) \(\$LD_OPT\)/\1 -Wl,--no-as-needed \2/g' $ORACLE_HOME/bin/genorasdksh

sed -i 's/^\(\s*\)\(\$(OCRLIBS_DEFAULT)\)/\1 -Wl,--no-as-needed \2/g' $ORACLE_HOME/srvm/lib/ins_srvm.mk
```
然后在图形界面点击‘Retry’就能继续安装了。

### 然后按照安装程序提示最后执行两个脚本
```
sudo  /home/oracle/Inventory/orainstRoot.sh 
sudo /home/oracle/oracle11g/product/11.2.0/dbhome_1/root.sh
```

### 创建监听，执行`$ netca`启动配置界面
参考
> http://www.jianshu.com/p/9b2f601c275d

完成之后，执行命令$ lsnrctl start启动监听服务。

### 创建数据库实例，执行$ dbca启动配置界面

###  最后验证是否安装成功，浏览器访问
https://192.168.1.114:1158/em

### 创建开机自动启动数据库的脚本
开一个普通的字符终端连接到UbuntuServer，运行如下命令：
```
# vi /etc/init.d/Oracledb
文件内容如下：
#!/bin/bash
#
# /etc/init.d/oracledb
#
# Run-level Startup script for the Oracle Instance, Listener, and
# Web Interface

export ORACLE_HOME=/home/oracle/oracle11g/product/11.2.0/dbhome_1
export ORACLE_SID=orcl
export PATH=$ORACLE_HOME/bin:$PATH

ORA_OWNR="oracle"
# if the executables do not exist -- display error
if [ ! -f $ORACLE_HOME/bin/dbstart -o ! -d $ORACLE_HOME ]
then
echo "Oracle startup: cannot start"
exit 1
fi
# depending on parameter -- startup, shutdown, restart
# of the instance and listener or usage display
case "$1" in
start)
# Oracle listener and instance startup
echo -n "Starting Oracle: "
su $ORA_OWNR -c "$ORACLE_HOME/bin/lsnrctl start"
su $ORA_OWNR -c "$ORACLE_HOME/bin/dbstart"
touch /var/lock/oracle
su $ORA_OWNR -c "$ORACLE_HOME/bin/emctl start dbconsole"
echo "OK"
;;
stop)
# Oracle listener and instance shutdown
echo -n "Shutdown Oracle: "
su $ORA_OWNR -c "$ORACLE_HOME/bin/lsnrctl stop"
su $ORA_OWNR -c "$ORACLE_HOME/bin/dbshut"
rm -f /var/lock/oracle
su $ORA_OWNR -c "$ORACLE_HOME/bin/emctl stop dbconsole"
echo "OK"
;;
reload|restart)
$0 stop
$0 start
;;
*)
echo "Usage: `basename $0` start|stop|restart|reload"
exit 1
esac
exit 0
```
再运行如下命令设置权限，并放到启动脚本中去：
```
# chmod 755 /etc/init.d/oracledb
# update-rc.d oracledb defaults 99
```
最后：
`# vi /etc/oratab`
把文件中的N改成Y，即"orcl:/opt/oracle/product/db:N"修改为"orcl:/opt/oracle/product/db:Y"。
### 常用命令
```
$ ps -ef|grep ora_|grep -v grep  -->查看oracle进程
$ ps -ef|grep tnslsnr|grep -v grep  -->查看oracle的监听进程
$ lsnrctl start -->启动监听
$ dbstart -->启动数据库
$ dbstop -->停止数据库
$ emctl start dbconsole -->启动em控制台
$ isqlplusctl start -->启动pl/sql
$ sqlplus '/as sysdba' -->登录sqlplus

$ env  -->输出当前用户的环境变量

$ netca -->启用监听配置程序
```
参考文章
> [CentOS6.7安装Oracle 11g2R傻瓜图文教程](CentOS6.7安装Oracle 11g2R傻瓜图文教程)
> [Ubuntu 14.04安装Oracle11g 64位](http://www.jianshu.com/p/4d7ccf6135af)
> [ubuntu16.04安装oracle11g](http://blog.csdn.net/u010286751/article/details/51975741)
> [Ubuntu Server 11.04 安装 Oracle 11g r2 图解教程](http://www.linuxidc.com/Linux/2011-12/48931.htm)