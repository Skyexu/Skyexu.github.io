---
title: oracle启动dbstart出错或无反应的解决办法及自启动
date: 2016/8/20 9:51:07 
tags: Oracle
categories: Linux

---

### 问题一
启动dbstart 报错 
```
ORACLE_HOME_LISTNER is not SET, unable to auto-start Oracle Net Listener Usage: /home/oracle/oracle11g/product/11.2.0/dbhome_1/bin/dbstart ORACLE_HOME
```
linux成功安装Oracle后切换到Oracle用户后，直接使用`dbstart`($ORACLE_HOME/bin中)启动oracle数据库报错如上。原因是dbstart调用的tnslsnr脚本位置有错。解决办法：
打开该脚本：`vim $ORACLE_HOME/bin/dbstart`
查找“ORACLE_HOME_LISTENER”变量的定义处，
修改
`ORACLE_HOME_LISTENER＝$1`
为`ORACLE_HOME_LISTENER＝$ORACLE_HOME`
### 问题二

启动`dbstart`没有反应，即不报错也不显示启动信息
原因是oracle的配置需要修改才能使用dbstart启动对应的数据实例。
解决办法：

`sudo vim /etc/oratab`
将`orcl:/home/oracle/oracle11g/product/11.2.0/dbhome_1:N`改为`orcl:/home/oracle/oracle11g/product/11.2.0/dbhome_1:Y`

### 问题三
```
>dbstart
Can't find init file for Database "orcl".

Database "orcl" NOT started.
```
原因就是没有找到init文件 我的数据库实例是orcl
这个文件`在$ORACLE_HOME/dbs/`目录下
`cd $ORACLE_HOME/dbs`
解决办法就是建立一个initorcl.ora的软连接就可以了
`ln -s spfileego.ora initorcl.ora`


### Oracle自启动

创建开机自动启动数据库的脚本
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
# chmod 755 /etc/init.d/Oracledb
# update-rc.d Oracledb defaults 99
```
最后：
`# vi /etc/oratab`
把文件中的N改成Y，即"orcl:/opt/oracle/product/db:N"修改为"orcl:/opt/oracle/product/db:Y"。