---
title: HBase 1.2.4 安装配置
date: 2016-09-22 23:21
tags: 
  - HBase
  - Hadoop
  - Zookeeper
categories:
  - [大数据, HBase]
---

官方网站：  
http://hbase.apache.org/

文档：  
http://hbase.apache.org/book.html

## 版本信息
CentOS Linux release 7.2.1511 (Core)  
Hadoop 2.7.2  
java version "1.7.0_80"  
hbase 1.2.4

## 注意事项
1. 检查防火墙  
2. 检查SSH免密码登录  
3. 检查/etc/hosts  
4. 检查各个服务器的JDK  
5. 还应该注意各个服务器时间同步

## 部署结构

Node Name | Master | ZooKeeper | RegionServer
---|---|---|---
wwh213 | yes | yes | no
wwh214 | backup | yes | yes
wwh215 | no | yes | yes

----

#### /etc/hosts 配置
```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.1.213    wwh213
192.168.1.214    wwh214
192.168.1.215    wwh215
```

#### 修改limit限制
HBase 会在同一时间打开大量的文件句柄和进程，超过 Linux 的默认限制，导致可能会出现如下错误：
```
2010-04-06 03:04:37,542 INFO org.apache.hadoop.hdfs.DFSClient: Exception increateBlockOutputStream java.io.EOFException
2010-04-06 03:04:37,542 INFO org.apache.hadoop.hdfs.DFSClient: Abandoning block blk_-6935524980745310745_1391901
```

修改之前先查看一下：
```
$ ulimit -n -u
open files                      (-n) 1024
max user processes              (-u) 7274
```

以root用户编辑文件：/etc/security/limits.conf

在末尾加上:
```
hadoop  -       nofile  32768
hadoop  -       nproc   32000
```
退出shell重新进入后生效

切换到hadoop用户

再次查看：
```
$ ulimit -n -u
open files                      (-n) 32768
max user processes              (-u) 32000
```

## 安装

### 下载解压
选择一个合适的目录
```
wget http://apache.fayea.com/hbase/1.2.4/hbase-1.2.4-bin.tar.gz

tar -zxvf hbase-1.2.4-bin.tar.gz 
```

### 配置

#### conf/hbase-env.sh
    
    对于HBase 0.98.5和更高版本，需要在启动HBase之前设置JAVA_HOME环境变量  
    
    export JAVA_HOME=/usr/java/jdk1.7.0_80
    
    假定集群的每个节点使用相同的配置。如果不是这样，您需要为每个节点单独设置JAVA_HOME。
*这个环境变量设置不是必须的*

告诉HBase是否应该管理自己的Zookeeper实例
```
export HBASE_MANAGES_ZK=false
```
> 自带的 Zookeeper 如不用，可删除


设置PID文件存储路径，缺省是在/tmp 下
```
export HBASE_PID_DIR=/data/hbase/tmp/pids
```

使用JDK8 ，需要在HBase的配置文件中hbase-env.sh，注释掉两行
```
# Configure PermSize. Only needed in JDK7. You can safely remove it for JDK8+
export HBASE_MASTER_OPTS="$HBASE_MASTER_OPTS -XX:PermSize=128m -XX:MaxPermSize=128m"
export HBASE_REGIONSERVER_OPTS="$HBASE_REGIONSERVER_OPTS -XX:PermSize=128m -XX:MaxPermSize=128m"
```

#### conf/hbase-site.xml

这是HBase的主配置文件  
缺省配置：http://hbase.apache.org/book.html#hbase_default_configurations

配置本地临时目录，缺省值是'/tmp'
```
<property>
  <name>hbase.tmp.dir</name>
  <value>/data/hbase/tmp</value>
</property>
```

指定分布式运行
```
<property>
  <name>hbase.cluster.distributed</name>
  <value>true</value>
</property>
```

配置hbase目录指到HDFS
```
<property>
  <name>hbase.rootdir</name>
  <value>hdfs://192.168.1.213:9900/hbase</value>
</property>
```
>注意：  
$HBASE_HOME/conf/hbase-site.xml 的 hbase.rootdir的主机和端口号与$HADOOP_HOME/conf/core-site.xml 的 fs.default.name的主机和端口号一致

配置Zookeeper  
```
<property>
  <name>hbase.zookeeper.quorum</name>
  <value>wwh213,wwh214,wwh215</value>
</property>

```


##### conf/regionservers

配置RegionServer，删除之前的 ==localhost== ，增加主机名或者IP。
```
wwh214
wwh215
```

##### 配置 backup master 
创建一个新的文件**conf/backup-masters**，并添加一个主机名。
```
echo 'wwh214' > backup-masters

wwh214
```

##### 复制hbase到其他节点
集群中的节点都需要相同的配置信息
```
scp -r hbase-1.2.4 wwh214:/data/hbase/

scp -r hbase-1.2.4 wwh215:/data/hbase/
```

## 启动
确保Zookeeper是启动状态的  
随便找一台有hbase的节点，启动即可
```
bin/start-hbase.sh
```
输出信息：
```
starting master, logging to /data/hbase/hbase-1.2.4/bin/../logs/hbase-hadoop-master-wwh213.out
wwh215: starting regionserver, logging to /data/hbase/hbase-1.2.4/bin/../logs/hbase-hadoop-regionserver-wwh215.out
wwh214: starting regionserver, logging to /data/hbase/hbase-1.2.4/bin/../logs/hbase-hadoop-regionserver-wwh214.out
wwh214: starting master, logging to /data/hbase/hbase-1.2.4/bin/../logs/hbase-hadoop-master-wwh214.out
```

## 检查

1. 检查Java进程
```
$ JPS  

xxxxx HMaster
xxxxx HRegionServer

```

2. 检查HDFS中的HBase目录
```
$ bin/hadoop fs -ls /hbase
Found 7 items
drwxr-xr-x   - hadoop supergroup          0 2016-11-09 16:00 /hbase/.tmp
drwxr-xr-x   - hadoop supergroup          0 2016-11-09 16:00 /hbase/MasterProcWALs
drwxr-xr-x   - hadoop supergroup          0 2016-11-09 16:00 /hbase/WALs
drwxr-xr-x   - hadoop supergroup          0 2016-11-09 16:00 /hbase/data
-rw-r--r--   3 hadoop supergroup         42 2016-11-09 15:59 /hbase/hbase.id
-rw-r--r--   3 hadoop supergroup          7 2016-11-09 15:59 /hbase/hbase.version
drwxr-xr-x   - hadoop supergroup          0 2016-11-09 15:59 /hbase/oldWALs
```

3. 使用 HBase Shell
```
$ bin/hbase shell
.....
.....
hbase(main):001:0> 
```


## Web UI
HBase的Web界面，现在的端口为16010，原来是60010.  
http://192.168.1.213:16010

