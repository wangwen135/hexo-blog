---
title: Zookeeper
date: 2017-04-21 23:41
tags: 
  - Zookeeper
categories:
  - [Zookeeper]
---

ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby一个开源的实现，是Hadoop和Hbase的重要组件。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务、互斥锁等。

ZooKeeper是一个高性能，可扩展的服务，只要集群中超过一半的节点正常，那么整个集群就可以正常对外提供服务。Zookeeper有数据一致性的保证：顺序一致性，客户端的更新会按照它们发送的次序排序；原子性，更新要么成功，要么失败，不会出现部分成功的（更新操作）结果。单独系统镜像，不管客户端连哪个服务器，它看来都是同一个。可靠性，一旦更新生效，它就会一直保存到下一次客户端更新。

## 安装

#### 下载：
```
# 这个已经不是稳定了
wget http://archive.apache.org/dist/zookeeper/stable/zookeeper-3.4.6.tar.gz

# 
http://archive.apache.org/dist/zookeeper/zookeeper-3.3.6/zookeeper-3.3.6.tar.gz
http://archive.apache.org/dist/zookeeper/zookeeper-3.4.8/zookeeper-3.4.8.tar.gz


```

#### 解压与软链接：
```
tar -zxvf zookeeper-3.4.6.tar.gz -C /opt

#可以不要
ln -s /opt/zookeeper-3.4.6 /opt/zookeeper

chown -R zookeeper:zookeeper /opt/zookeeper*
```

#### 复制配置文件
```
cp conf/zoo_sample.cfg conf/zoo.cfg
```

## 配置

### zoo.cfg

```
tickTime=2000

#数据目录. 可以是任意目录
dataDir=/home/zookeeper/data

#log目录, 可以是任意目录
dataLogDir=/home/zookeeper/dataLog

#client连接的端口
clientPort=2181

#配置集群
server.1=192.168.1.213:2888:3888
server.2=192.168.1.214:2888:3888
server.3=192.168.1.215:2888:3888

```
>server.A=B：C：D：其中A 是一个数字，表示这个是第几号服务器；B 是这个服务器的 ip 地址；C 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口；D 表示的是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。

在 **dataDir** 目录中创建一个名为 **myid** 文件，在文件中写入节点ID，与zoo.cfg中的集群配置相对应
>在集群模式中各server的dataDir目录下的myid文件中的数字必须不同

    单机版直接解压缩启动就可以了，不用改什么配置

## 命令

启动  
```
bin/zkServer.sh start  
```

查看状态
```
bin/zkServer.sh status
```

停止  
```
bin/zkServer.sh stop
```

启动client连接server  
```
bin/zkCli.sh -server localhost:2181  
```


## 查看状态

```
echo stat | nc 192.168.1.213 2181
echo stat | nc 192.168.1.214 2181
echo stat | nc 192.168.1.215 2181
echo stat | nc 192.168.1.216 2181
echo stat | nc 192.168.1.217 2181
```

## 服务化
【基于 CentOS 7】

在/usr/lib/systemd/system/ 目录中创建zookeeper.service文件。  
文件内容如下：  

> 日志目录和安装目录自己按照实际情况改

```
[Unit]
Description=Zookeeper Service
After=network.target 

[Service]
Environment=ZOO_LOG_DIR=/data/zookeeper/zookeeper-3.4.8/
Type=forking
User=dap
ExecStart=/data/zookeeper/zookeeper-3.4.8/bin/zkServer.sh start
ExecStop=/data/zookeeper/zookeeper-3.4.8/bin/zkServer.sh stop
ExecReload=/data/zookeeper/zookeeper-3.4.8/bin/zkServer.sh restart
SuccessExitStatus=SIGKILL

[Install]
WantedBy=multi-user.target

```

再执行如下命令：
```
# 重新加载
systemctl daemon-reload

#启动服务
systemctl start zookeeper

#查看服务启动情况
systemctl status zookeeper

#设置为开机启动
systemctl enable zookeeper

```
