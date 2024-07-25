---
title: Hadoop默认配置
date: 2024-03-14 23:41
tags: 
  - Hadoop
categories:
  - [大数据, Hadoop]
---

*王某某 2016年9月*

----

配置hadoop，主要是配置core-site.xml，hdfs-site.xml，mapred-site.xml，yarn-site.xml三个配置文件，默认下来，这些配置文件都是空的，所以很难知道这些配置文件有哪些配置可以生效，需要去Apache的官网中找对应版本的默认配置文件。YARN是新一代的MapReduce，又叫：MapReduce 2.0 (MRv2)。

### 官网地址
https://hadoop.apache.org/docs/  
里面有各个版本的文档

### 2.72版本默认配置
#### core
http://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-common/core-default.xml
#### hdfs
http://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml
#### mapreduce
http://hadoop.apache.org/docs/r2.7.2/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml
#### yarn
http://hadoop.apache.org/docs/r2.7.2/hadoop-yarn/hadoop-yarn-common/yarn-default.xml


### 常用的端口配置
网上复制的，没有经过验证

#### HDFS端口

参数 | 描述 | 默认 | 配置文件 | 例子值
---|---|---|---|---
fs.default.name | namenode RPC交互端口 | 8020 | core-site.xml | hdfs://master:8020/
dfs.http.address |  NameNode web管理端口 | 50070 | hdfs-site.xml | 0.0.0.0:50070
dfs.datanode.address | datanode　控制端口 | 50010 | hdfs-site.xml | 0.0.0.0:50010
dfs.datanode.ipc.address | datanode的RPC服务器地址和端口 | 50020 | hdfs-site.xml | 0.0.0.0:50020
dfs.datanode.http.address | datanode的HTTP服务器和端口 | 50075 | hdfs-site.xml | 0.0.0.0:50075

#### MR端口

参数 | 描述 | 默认 | 配置文件 | 例子值
---|---|---|---|---
mapred.job.tracker | job tracker交互端口 | 8021 | mapred-site.xml | hdfs://master:8021/
mapred.job.tracker.http.address | job tracker的web管理端口 | 50030 | mapred-site.xml | 0.0.0.0:50030
mapred.task.tracker.http.address | task tracker的HTTP端口 | 50060 | mapred-site.xml | 0.0.0.0:50060

#### 其他端口

参数 | 描述 | 默认 | 配置文件 | 例子值
---|---|---|---|---
dfs.secondary.http.address | secondary | NameNode web管理端口 | 50090 | hdfs-site.xml | 0.0.0.0:28680


#### 集群目录配置

参数 | 描述 | 默认 | 配置文件 | 例子值
---|---|---|---|---
dfs.name.dir | name node的元数据,以,号隔开,hdfs会把元数据冗余复制到这些目录，一般这些目录是不同的块设备，不存在的目录会被忽略掉 | {hadoop.tmp.dir}/dfs/name | hdfs-site.xm | /hadoop/hdfs/name
dfs.name.edits.dir | node node的事务文件存储的目录,以,号隔开,hdfs会把事务文件冗余复制到这些目录，一般这些目录是不同的块设备，不存在的目录会被忽略掉 | ${dfs.name.dir} | hdfs-site.xm	| ${dfs.name.dir}
fs.checkpoint.dir | secondary NameNode的元数据以,号隔开,hdfs会把元数据冗余复制到这些目录，一般这些目录是不同的块设备，不存在的目录会被忽略掉 | ${hadoop.tmp.dir}/dfs/namesecondary | core-site.xml | /hadoop/hdfs/namesecondary
fs.checkpoint.edits.dir | secondary NameNode的事务文件存储的目录,以,号隔开,hdfs会把事务文件冗余复制到这些目录 | ${fs.checkpoint.dir} | core-site.xml | ${fs.checkpoint.dir}
hadoop.tmp.dir | 临时目录,其他临时目录的父目录 | /tmp/hadoop-${user.name} | core-site.xml | /hadoop/tmp/hadoop-${user.name}
dfs.data.dir | data node的数据目录,以,号隔开,hdfs会把数据存在这些目录下，一般这些目录是不同的块设备，不存在的目录会被忽略掉 | ${hadoop.tmp.dir}/dfs/data | hdfs-site.xm | /hadoop/hdfs/data1/data,<br>/hadoop/hdfs/data2/data
mapred.local.dir | MapReduce产生的中间数据存放目录,以,号隔开,hdfs会把数据存在这些目录下，一般这些目录是不同的块设备，不存在的目录会被忽略掉 | ${hadoop.tmp.dir}/mapred/local | mapred-site.xml | /hadoop/hdfs/data1/mapred/local,<br>/hadoop/hdfs/data2/mapred/local
mapred.system.dir | MapReduce的控制文件 | ${hadoop.tmp.dir}/mapred/system | mapred-site.xml | /hadoop/hdfs/data1/system

#### 其他配置

参数 | 描述 | 默认 | 配置文件 | 例子值
---|---|---|---|---
dfs.support.append | 支持文件append，主要是支持hbase | false | hdfs-site.xml | true
dfs.replication | 文件复制的副本数，如果创建时不指定这个参数，就使用这个默认值作为复制的副本数 | 3 | hdfs-site.xml | 2
