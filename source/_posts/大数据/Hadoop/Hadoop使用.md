---
title: Hadoop使用
date: 2022-07-06 23:41
tags: 
  - Hadoop
categories:
  - [大数据, Hadoop]
---

*王某某 2016年9月*

----


## 官方文档
#### Apache Hadoop 2.7.2 文档
http://hadoop.apache.org/docs/stable/index.html

#### 中文文档
比较老的，应该对照最新的文档看
##### 命令手册
https://hadoop.apache.org/docs/r1.0.4/cn/commands_manual.html
##### Hadoop Shell命令
https://hadoop.apache.org/docs/r1.0.4/cn/hdfs_shell.html

## Hadoop命令

---

**这些是从比较老的文档中复制的！**

---

所有的hadoop命令均由bin/hadoop脚本引发。不指定参数运行hadoop脚本会打印所有命令的描述。

用法：hadoop [--config confdir] [COMMAND] [GENERIC_OPTIONS] [COMMAND_OPTIONS]

Hadoop有一个选项解析框架用于解析一般的选项和运行类。

命令选项 | 描述
---|---
--config confdir | 覆盖缺省配置目录。缺省是${HADOOP_HOME}/conf。
GENERIC_OPTIONS | 多个命令都支持的通用选项。
COMMAND 命令选项S | 各种各样的命令和它们的选项会在下面提到。这些命令被分为 用户命令 管理命令两组。

**常规选项**  
下面的选项被 dfsadmin, fs, fsck和 job支持。 应用程序要实现 Tool来支持 常规选项。

常规选项 | 描述
---|---
-conf <configuration file> | 指定应用程序的配置文件。
-D <property=value> | 为指定property指定值value。
-fs <local \| namenode:port> | 指定namenode。
-jt <local \| jobtracker:port> | 指定job tracker。只适用于job。
-files <逗号分隔的文件列表> | 指定要拷贝到map reduce集群的文件的逗号分隔的列表。 只适用于job。
-libjars <逗号分隔的jar列表> | 指定要包含到classpath中的jar文件的逗号分隔的列表。 只适用于job。
-archives <逗号分隔的archive列表> | 指定要被解压到计算节点上的档案文件的逗号分割的列表。 只适用于job。


**命令主要分为两类：**  

---

用户命令 | 说明
---|---
archive | 创建一个hadoop档案文件
distcp | 递归地拷贝文件或目录
fs | 运行一个常规的文件系统客户端
fsck | 运行HDFS文件系统检查工具
jar | 运行jar文件。用户可以把他们的Map Reduce代码捆绑到jar文件中，使用这个命令执行
job | 用于和Map Reduce作业交互和命令
pipes | 运行pipes作业
version | 打印版本信息
CLASSNAME | hadoop脚本可用于调调用任何类

---

管理命令 | 说明
---|---
balancer | 运行集群平衡工具。管理员可以简单的按Ctrl-C来停止平衡过程
daemonlog | 获取或设置每个守护进程的日志级别
datanode | 运行一个HDFS的datanode
dfsadmin | 运行一个HDFS的dfsadmin客户端
jobtracker | 运行MapReduce job Tracker节点
namenode | 运行namenode。有关升级，回滚，升级
secondarynamenode | 运行HDFS的secondary namenode
tasktracker | 运行MapReduce的task Tracker节点

---

## Hadoop Shell命令

调用文件系统(FS)Shell命令应使用 bin/hadoop fs <args>的形式。 所有的的FS shell命令使用URI路径作为参数。URI格式是scheme://authority/path。对HDFS文件系统，scheme是hdfs，对本地文件系统，scheme是file。其中scheme和authority参数都是可选的，如果未加指定，就会使用配置中指定的默认scheme。一个HDFS文件或目录比如/parent/child可以表示成hdfs://namenode:namenodeport/parent/child，或者更简单的/parent/child（假设你配置文件中的默认值是namenode:namenodeport）。大多数FS Shell命令的行为和对应的Unix Shell命令类似，不同之处会在下面介绍各命令使用详情时指出。出错信息会输出到stderr，其他信息输出到stdout。

---

**下面这些是结合新旧文档写的！并且经过测试**

Hadoop 2.7.2 中推荐使用**hdfs dfs** 
 
查看命令
```
bin/hdfs dfs -help
```
查看命令详情
```
bin/hdfs dfs -help command-name 
```

---


### mkdir
使用方法：
>hadoop fs -mkdir <paths>   

接收路径的URI作为参数，并创建目录。

参数：
>-p 创建沿路径的各级父目录  

示例：
```
hadoop fs -mkdir /user/hadoop/dir1 /user/hadoop/dir2
hadoop fs -mkdir hdfs://nn1.example.com/user/hadoop/dir hdfs://nn2.example.com/user/hadoop/dir
```
```
./hadoop fs -mkdir /wwh
./hadoop fs -mkdir -p /user/hadoop/dir1
```

---

### ls
使用方法：
>hadoop fs -ls [-d] [-h] [-R] <args>

列出文件和目录。

参数：
>-d: 只列目录
>-h: 格式化大小为人类可读的形式
>-R: 递归子目录

示例：
```
hadoop fs -ls /user/hadoop/file1
```
```
[root@wwh213 bin]# ./hadoop fs -ls /
Found 3 items
drwxr-xr-x   - root supergroup          0 2016-09-19 20:42 /user
drwxr-xr-x   - root supergroup          0 2016-09-19 20:42 /wwh
-rw-r--r--   2 root supergroup        818 2016-09-19 20:43 /测试文件.txt
```

---

### put
使用方法：
>hadoop fs -put <localsrc> ... <dst>  

从本地文件系统中复制单个或多个源路径到目标文件系统。也支持从标准输入中读取输入写入目标文件系统。  

示例：
```
hadoop fs -put localfile /user/hadoop/hadoopfile
hadoop fs -put localfile1 localfile2 /user/hadoop/hadoopdir
hadoop fs -put localfile hdfs://host:port/hadoop/hadoopfile
hadoop fs -put - hdfs://host:port/hadoop/hadoopfile 
从标准输入中读取输入。
```
```
./hadoop fs -put /tmp/测试文件.txt /

./hadoop fs -put - /input.txt
# CTRL+D 结束输入
```

### cat
使用方法：
>hadoop fs -cat URI [URI …]

将路径指定文件的内容输出到stdout。

示例：
```
hadoop fs -cat hdfs://nn1.example.com/file1 hdfs://nn2.example.com/file2
hadoop fs -cat file:///file3 /user/hadoop/file4
```
```
./hadoop fs -cat /input.txt
```

---

### get
使用方法：
>hadoop fs -get [-ignorecrc] [-crc] <src> <localdst>

复制文件到本地文件系统

参数：
>-ignorecrc 可以复制CRC校验失败的文件  
>-crc 选择可以复制文件和 CRC信息 

示例：
```
hadoop fs -get /user/hadoop/file localfile
hadoop fs -get hdfs://nn.example.com/user/hadoop/file localfile
```
```
[root@wwh213 bin]# ./hadoop fs -get /测试文件.txt /home
[root@wwh213 bin]# 
```

### df
使用方法：
>hadoop fs -df [-h] URI [URI ...]

显示空闲空间  

参数：
> -h 将大小格式化成人类可读的形式

示例：
```
hadoop dfs -df /user/hadoop/dir1
```
```
#不再推荐的
./hadoop dfs -df /
DEPRECATED: Use of this script to execute hdfs command is deprecated.
Instead use the hdfs command for it.

Filesystem                        Size   Used   Available  Use%
hdfs://192.168.1.213:9900  14298382336  32768  7656443904    0%

#推荐的形式
./hdfs dfs -df
Filesystem                        Size   Used   Available  Use%
hdfs://192.168.1.213:9900  14298382336  32768  7658508288    0%

#-h选项
 ./hdfs dfs -df -h
Filesystem                   Size  Used  Available  Use%
hdfs://192.168.1.213:9900  13.3 G  32 K      7.1 G    0%
```

### du
使用方法：
>hadoop fs -du [-s] [-h] URI [URI ...]

显示目录中所有文件的大小，或者当只指定一个文件时，显示此文件的大小。

参数：
>-s 显示总大小  
>-h 格式化大小为人类可读的形式

示例：
```
hadoop fs -du /user/hadoop/dir1 /user/hadoop/file1 hdfs://nn.example.com/user/hadoop/dir1
```
```
./hadoop fs -du /
192  /input.txt
0    /user
275  /wwh
818  /测试文件.txt
```

---

## Hadoop WebHDFS

### WebHDFS REST API

https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/WebHDFS.html

直接用这个地址可以通过web浏览文件

http://192.168.1.213:50070/explorer.html#/

---

## Hadoop API

### java API

通过java API 对文件进行操作

Apache Hadoop Main 2.7.2 API  
https://hadoop.apache.org/docs/r2.7.2/api/index.html

1. 创建一个简单的Maven工程
2. 将JDK版本改到1.7
```
<plugin>
	<artifactId>maven-compiler-plugin</artifactId>
	<version>3.1</version>
	<configuration>
		<source>1.7</source>
		<target>1.7</target>
	</configuration>
</plugin>
```
3. 添加Hadoop-client依赖
```
<dependency>
	<groupId>org.apache.hadoop</groupId>
	<artifactId>hadoop-client</artifactId>
	<version>2.7.2</version>
</dependency>
```
4. 查看文件
```
String uri = "hdfs://192.168.1.213:9900/"; // hdfs 地址
Configuration conf = new Configuration();
FileSystem fs = FileSystem.get(URI.create(uri), conf);
Path path = new Path("/");
FileStatus[] fileStatus = fs.listStatus(path);
for (FileStatus file : fileStatus) {
...
}
```
5. 读写文件  
>权限问题解决：
>> - 修改 hdfs-core.xml 中的 dfs.permissions 配置项为false
>> - 通过hadoop shell 命令修改目录权限为 777
>> - 在客户端配置环境变量 HADOOP_USER_NAME=root
```
String uri = "hdfs://192.168.1.213:9900/"; // hdfs 地址
String writeFile = "/hello.txt";
// 设置一个环境变量
System.setProperty("HADOOP_USER_NAME", "root");

Configuration conf = new Configuration();
FileSystem fs = FileSystem.get(URI.create(uri), conf);
Path path = new Path(writeFile);

FSDataOutputStream fsdos = fs.create(path);
...写内容...
fsdos.close();

FSDataInputStream fsdis = fs.open(path);
...读内容...
fsdis.close();
```



