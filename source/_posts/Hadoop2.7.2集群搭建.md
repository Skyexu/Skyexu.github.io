---
title: Hadoop2.7.2集群搭建
date: 2016-08-15 17:26:16
tags: Hadoop
categories: 大数据

---

### 四台电脑集群
```
192.168.1.111   master
192.168.1.112   slave1
192.168.1.113   slave2
192.168.1.114   slave3
```
### 修改hosts
`vim /etc/hosts`

### 配置master到其它三台slave的免密码登陆
各服务器上使用   ssh-keygen -t rsa    一路按回车就行了。
刚才都作甚了呢？主要是设置ssh的密钥和密钥的存放路径。 路径为~/.ssh下。
打开~/.ssh 下面有三个文件
authorized_keys，已认证的keys
id_rsa，私钥
id_rsa.pub，公钥   三个文件。
下面就是关键的地方了，（我们要做ssh认证。进行下面操作前，可以先搜关于认证和加密区别以及各自的过程。）
①在master上将公钥放到authorized_keys里。命令：
`sudo cat id_rsa.pub >> authorized_keys`
②将master上的authorized_keys放到其他linux的~/.ssh目录下。命令：
`sudo scp authorized_keys ubuntu@192.168.1.112:~/.ssh       `
sudo scp authorized_keys 远程主机用户名@远程主机名或ip:存放路径。
③修改authorized_keys权限，命令：chmod 644 authorized_keys
④测试是否成功
ssh slave1 输入用户名密码，然后退出，再次ssh slave1不用密码，直接进入系统。这就表示成功了。

### 安装jdk1.7

1. `mkdir /usr/java`
2. `sudo tar -zxvf jdk-7u79-linux-x64.tar.gz -C /usr/java`
3. `vim /etc/profile`
```
export JAVA_HOME=/usr/java/jdk1.7.0_79
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```
4. 刷新配置
`source /etc/profile`
5. 测试 
`java -version`

### 关闭每台机器的防火墙
`sudo ufw disable` (重启生效)

### 创建/home/cloud目录
`mkdir ~/cloud`

### 解压hadoop
`tar -zxvf hadoop-2.7.2.tar.gz -C ./cloud`

### 配置hadoop-env.sh yarn-env.sh
修改JAVA_HOME值`export JAVA_HOME=/usr/java/jdk1.7.0_79`

### 创建文件夹
```
mkdir /home/ubuntu/cloud/hadoop-2.7.2/tmp
mkdir /home/ubuntu/cloud/hadoop-2.7.2/dfs/data
mkdir /home/ubuntu/cloud/hadoop-2.7.2/dfs/name
```
### 配置slaves
```
slave1
slave2
slave3
```
### 配置core-site.xml
```
<configuration>
       <property>
                <name>fs.defaultFS</name>
                <value>hdfs://master:9000</value>
       </property>
       <property>
                <name>io.file.buffer.size</name>
                <value>131072</value>
        </property>
       <property>
               <name>hadoop.tmp.dir</name>
               <value>file:/home/ubuntu/cloud/hadoop-2.7.2/tmp</value>
               <description>Abase for other temporary   directories.</description>
       </property>
        <property>
               <name>hadoop.proxyuser.ubuntu.hosts</name>
               <value>*</value>
       </property>
       <property>
               <name>hadoop.proxyuser.ubuntu.groups</name>
               <value>*</value>
       </property>
</configuration>

```

### 配置hdfs-site.xml
```
<configuration>  
    <property>  
        <name>dfs.namenode.secondary.http-address</name>  
        <value>master:9001</value>  
    </property>  
    <property>  
        <name>dfs.replication</name>  
        <value>4</value>  
    </property>  
    <property>  
        <name>dfs.permissions.enabled</name>  
        <value>false</value>  
    </property>  
    <property>  
        <name>dfs.namenode.name.dir</name>  
        <value>file:/home/ubuntu/cloud/hadoop-2.7.2/dfs/name</value>  
    </property>  
    <property>  
        <name>dfs.datanode.data.dir</name>  
        <value>file:/home/ubuntu/cloud/hadoop-2.7.2/dfs/data</value>  
    </property>  
    <property>  
        <name>dfs.webhdfs.enabled</name>  
        <value>true</value>  
    </property>  
</configuration>  
```

### 配置mapred-site.xml

```
<configuration>
          <property>                                                                  <name>mapreduce.framework.name</name>
                <value>yarn</value>
           </property>
          <property>
                  <name>mapreduce.jobhistory.address</name>
                  <value>master:10020</value>
          </property>
          <property>
                <name>mapreduce.jobhistory.webapp.address</name>
                <value>master:19888</value>
       </property>
</configuration>

```

### 配置yarn-site.xml
```
<configuration>
        <property>
               <name>yarn.nodemanager.aux-services</name>
               <value>mapreduce_shuffle</value>
        </property>
        <property>                                                                
            <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
               <value>org.apache.hadoop.mapred.ShuffleHandler</value>
        </property>
        <property>
               <name>yarn.resourcemanager.address</name>
               <value>master:8032</value>
       </property>
       <property>
               <name>yarn.resourcemanager.scheduler.address</name>
               <value>master:8030</value>
       </property>
       <property>
            <name>yarn.resourcemanager.resource-tracker.address</name>
             <value>master:8031</value>
      </property>
      <property>
              <name>yarn.resourcemanager.admin.address</name>
               <value>master:8033</value>
       </property>
       <property>
               <name>yarn.resourcemanager.webapp.address</name>
               <value>master:8088</value>
       </property>
</configuration>

```

### 复制到其它节点
```
sudo scp -r /home/ubuntu/cloud ubuntu@slave1:~/
sudo scp -r /home/ubuntu/cloud ubuntu@slave2:~/
sudo scp -r /home/ubuntu/cloud ubuntu@slave3:~/
```

### 配置环境变量
`vim /etc/profile`
```
export HADOOP_HOME=/home/ubuntu/cloud/hadoop-2.7.2
export PATH=$PATH:${JAVA_HOME}/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:
```

### 启动hadoop
- 格式化namenode `hadoop namenode –format`
- 启动hdfs `start-dfs.sh`
- 启动yarn `start-yarn.sh`
- web端查看 `http://master:8088 http://master:50070`