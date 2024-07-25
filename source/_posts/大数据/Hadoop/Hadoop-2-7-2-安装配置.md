---
title: Hadoop-2-7-2-安装配置
date: 2024-07-13 23:41
tags: 
  - Hadoop
categories:
  - [大数据, Hadoop]
---

*王某某 2016年9月*

官网地址：
http://hadoop.apache.org/

各个版本文档地址：
http://hadoop.apache.org/docs/

## 1、环境

**版本信息**  
CentOS Linux release 7.2.1511 (Core)   
Hadoop 2.7.2  
java version "1.7.0_80"  

**/etc/hosts**
```
192.168.1.213    wwh213
192.168.1.214    wwh214
192.168.1.215    wwh215
```
确保各个主机都能正确解析主机名和IP
>***注意：***/etc/hosts 中配置节点与IP时需要注意，不要配置如：“127.0.0.1 master” 这样的，会导致无法绑定正确的IP  

**集群结构**
```
Master    192.168.1.213    wwh213  
Slave1    192.168.1.214    wwh214  
Slave2    192.168.1.215    wwh215
```

## 2、创建hadoop用户

```
#创建hadoop用户
adduser hadoop

#修改密码
passwd hadoop
```
以下操作都在hadoop用户下进行，注意目录权限。


## 3、SSH免密码登录
>无密码登陆是指通过证书认证的方式进行登陆，使用**公私钥**认证的方式来进行ssh登录。  
>**公私钥**认证方式简单的解释：首先在客户端上创建一对公私钥  
>公钥文件：~/.ssh/id_rsa.pub  
>私钥文件：~/.ssh/id_rsa  
>然后把公钥添加到服务器上的（~/.ssh/authorized_keys）文件里面，自己保留好私钥。
>登录时客户端会向服务器请求使用密匙进行安全验证并发送公钥到服务端，服务端会对比公钥如果匹配成功服务端会生成一个随机数然后用该公钥加密再发给客户端，客户端通过私钥解密后发给服务端进行确认。

受信任的远程主机公钥会保存到 ~/.ssh/known_hosts文件中

Hadoop需要通过SSH登录到各个节点进行操作，每台服务器都生成公钥，再合并到authorized_keys  

***==注意==*：authorized_keys的权限要是600**

1. CentOS默认没有启动ssh无密登录，去掉/etc/ssh/sshd_config其中两行的注释，每台服务器都要设置
```
RSAAuthentication yes
PubkeyAuthentication yes
```
2. 输入命令，**ssh-keygen -t rsa**，生成key，都不输入密码，一直回车，就会在~/.ssh文件夹生成
>**id_rsa**    私钥  
>**id_rsa.pub**    公钥  

在每台服务器都生成一次

3. 合并全部的公钥到一个authorized_keys文件，在Master服务器，进入~/.ssh目录，通过SSH命令合并
```
#cat id_rsa.pub>> authorized_keys
#使公钥添加到known_hosts中
ssh hadoop@wwh213 cat ~/.ssh/id_rsa.pub>> authorized_keys

ssh hadoop@wwh214 cat ~/.ssh/id_rsa.pub>> authorized_keys

ssh hadoop@wwh215 cat ~/.ssh/id_rsa.pub>> authorized_keys
```

4. 把Master服务器的**authorized_keys**、**known_hosts**复制到Slave服务器的~/.ssh目录  
```
scp authorized_keys known_hosts hadoop@wwh214:~/.ssh/

scp authorized_keys known_hosts hadoop@wwh215:~/.ssh/
```

5. 完成之后 ssh wwh214、ssh wwh215就不需要输入密码了  


## 4、安装JDK
Hadoop 2.7已经不再支持JDK6了，需要JDK7+

***之前已经安装过JDK了，下面是一个例子***

1. 下载“jdk-7u80-linux-x64.gz”，放到/data/java目录下  
2. 解压缩
```
tar -zxvf jdk-7u79-linux-x64.gz  
```
3. 编辑/etc/profile，加上：
```
export JAVA_HOME=/data/java/jdk1.7.0_80
export PATH=$PATH:$JAVA_HOME/bin 
```
4. 使配置生效，输入命令：
```
source /etc/profile
```
5. 进行测试
```
java -version
```

***或者直接下载 jdk-7u<version>-linux-x64.rpm 以root用户进行安装，将安装到/usr/java 目录中***

## 5、安装Hadoop 2.7.2
只需要在Master服务器解压配置好，再复制到Slave服务器  

配置hadoop，主要是配置**core-site.xml**，**hdfs-site.xml**，**mapred-site.xml**，**yarn-site.xml** 几个配置文件，默认这些配置文件都是空的，由于版本差异很难知道这些配置文件有哪些配置可以生效，需要去Apache的官网中找对应版本的默认配置文件。  

**官网文档地址：**  
>http://hadoop.apache.org/docs/r2.7.2/


---

1. 下载 hadoop-2.7.2.tar.gz ，放到/data/hadoop目录下  
```
wget http://archive.apache.org/dist/hadoop/common/hadoop-2.7.2/hadoop-2.7.2.tar.gz

wget http://apache.fayea.com/hadoop/common/hadoop-2.7.2/hadoop-2.7.2.tar.gz
```
2. 解压  
```   
tar -xzvf hadoop-2.7.2.tar.gz
```
3. 在/data/hadoop目录下创建数据存放的文件夹  
```   
mkdir tmp
mkdir hdfs
mkdir hdfs/data
mkdir hdfs/name
```
4. 配置JAVA_HOME  
修改/data/hadoop/hadoop-2.7.2/etc/hadoop目录下hadoop-env.sh、yarn-env.sh的JAVA_HOME  
```
export JAVA_HOME=/data/java/jdk1.7.0_80
```
> 如果环境变量配置好了也可以不用配置，默认是：export JAVA_HOME=${JAVA_HOME}
> 看启动时的提示

5. 配置/data/hadoop/hadoop-2.7.2/etc/hadoop目录下的core-site.xml  
   **官方默认的配置文件**  
   http://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-common/core-default.xml
```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://192.168.1.213:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:/data/hadoop/tmp</value>
    </property>
    <property>
        <name>io.file.buffer.size</name>
        <!-- 缺省值是4096 -->
        <value>131072</value>
    </property>
</configuration>
```
6. 配置/data/hadoop/hadoop-2.7.2/etc/hadoop目录下的hdfs-site.xml  
   **官方默认的配置文件**  
   http://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml
```
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/data/hadoop/hdfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/data/hadoop/hdfs/data</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <!-- 缺省值是3 -->
        <value>2</value>
    </property>

    <!-- 关闭权限检查 -->    
    <property>
        <name>dfs.permissions.enabled</name>
        <!-- 缺省值是true -->
        <value>false</value>
    </property>
    
</configuration>
```

7. 配置/data/hadoop/hadoop-2.7.2/etc/hadoop目录下的mapred-site.xml  
   **官方默认的配置文件**  
   http://hadoop.apache.org/docs/r2.7.2/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml
```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    
    <property>
        <name>mapreduce.jobhistory.address</name>
        <!-- 缺省值是：0.0.0.0:10020 -->
        <value>192.168.1.213:10020</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <!-- 缺省值是：0.0.0.0:19888 -->
        <value>192.168.1.213:19888</value>
    </property>
</configuration>
```

8. 配置/data/hadoop/hadoop-2.7.2/etc/hadoop目录下的yarn-site.xml  
   **官方默认的配置文件**  
   http://hadoop.apache.org/docs/r2.7.2/hadoop-yarn/hadoop-yarn-common/yarn-default.xml  
   >YARN：新一代的MapReduce。  MapReduce 2.0 (MRv2)
```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <!-- 缺省值是：0.0.0.0 -->
        <value>192.168.1.213</value>
    </property>
</configuration>
```

9. 配置/data/hadoop/hadoop-2.7.2/etc/hadoop目录下的slaves，删除默认的localhost，增加2个从节点  
   *如果配置主节点的IP的话就是既当 NameNode 又当 DataNode*  
```
192.168.1.213
192.168.1.214
192.168.1.215
```

10. 将配置好的Hadoop复制到各个节点对应位置上，通过scp传送
```
scp -r /data/hadoop wwh214:/data/
scp -r /data/hadoop wwh215:/data/
```

## 6、启动 Hadoop
在Master服务器启动hadoop，从节点会自动启动，进入/data/hadoop/hadoop-2.7.2目录  

(1)初始化，输入命令  
```
bin/hdfs namenode -format
```

(2)全部启动
```
sbin/start-all.sh
```

也可以分开启动
```
sbin/start-dfs.sh
sbin/start-yarn.sh
```

(3)停止的话，输入命令  
```
sbin/stop-all.sh
```

也可以分开停止
```
sbin/stop-dfs.sh
sbin/stop-yarn.sh
```


(4)输入命令，jps，可以看到相关信息


#### Web访问
要先开放端口或者直接关闭防火墙  
```
systemctl stop firewalld.service  
```

##### All Applications  
http://192.168.1.213:8088/  

##### HDFS  
http://192.168.1.213:50070/  

##### Map/Reduce  
http://192.168.1.213:50030/  

