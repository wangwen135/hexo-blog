---
title: Kafka 2.11-0.10.1.0 故障分析与处理
date: 2020-04-24 10:43:36
tags: 
  - Kafka
  - 死锁
categories:
  - [大数据, Kafka]
---


## 问题描述
线上环境每过一段时间（一个月左右） kafka就会故障无法访问，三个服务器都正常，kafka进程正常，但是会出现一个节点和其他两个节点断开的情况，表现形式上类似脑裂，日志中显示正常节点无法连接到故障的节点

出现问题 就不会再自动恢复了

重启故障节点即可解决这个问题


## 环境信息
### 集群
三台机器组成的kafka集群  
- 192.168.1.217 RD-MQ-01
- 192.168.1.218 RD-MQ-02
- 192.168.1.219 RD-MQ-03

### kafka 版本
kafka_2.11-0.10.1.0

### zookeeper 版本
3.4.10-4181
>ZK独立部署，不与其他程序共用

### java 版本
```
[root@RD-MQ-01 ~]# java -version
java version "1.8.0_162"
Java(TM) SE Runtime Environment (build 1.8.0_162-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.162-b12, mixed mode)
```

### 操作系统版本
CentOS 6.8
```
[root@RD-MQ-02 bin]# uname -a
Linux RD-MQ-02 2.6.32-642.el6.x86_64 #1 SMP Tue May 10 17:27:01 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux

[root@RD-MQ-02 bin]# cat /etc/redhat-release 
CentOS release 6.8 (Final)
```

## 相关配置
### kafka
```
############################# Server Basics #############################
broker.id=218
delete.topic.enable=true

############################# Socket Server Settings #############################
host.name=192.168.1.218
listeners=PLAINTEXT://192.168.1.218:9092
num.network.threads=4
num.io.threads=4
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600

############################# Log Basics #############################
log.dirs=/opt/data/kafka
num.partitions=32

############################# Internal Topic Settings  #############################
offsets.topic.replication.factor=3
transaction.state.log.replication.factor=3
transaction.state.log.min.isr=3

############################# Log Flush Policy #############################
# 每当producer写入10000条消息时，刷数据到磁盘
log.flush.interval.messages=2000
# 每间隔1秒钟时间，刷数据到磁盘
log.flush.interval.ms=1000

############################# Log Retention Policy #############################
# 日志保留的时长与大小，二者满足其一即生效，此处大小为200GB
log.retention.hours=72
log.retention.bytes=214748364800
# 消息体大小
message.max.byte=1
# 副本数,Replica过少会影响数据的可用性，太多则会白白浪费存储资源，一般建议在2~3为宜。
default.replication.factor=3
# Producer用于压缩数据的压缩类型
compression.type=gzip
# replicas每次获取数据的最大字节数
replica.fetch.max.bytes=5242880

# 在Replica上会启动若干Fetch线程把对应的数据同步到本地，而num.replica.fetchers这个参数是用来控制Fetch线程的数量。
# 每个Partition启动的多个Fetcher，通过共享offset既保证了同一时间内Consumer和Partition之间的一对一关系，又允许我们通过增多Fetch线程来提高效率。
num.replica.fetchers=4

# 段文件配置1GB，有利于快速回收磁盘空间，重启kafka加载也会加快(如果文件过小，则文件数量比较多，kafka启动时是单线程扫描目录(log.dir)下所有数据文件),此处为1G
log.segment.bytes=1073741824

log.retention.check.interval.ms=300000

############################# Zookeeper #############################
zookeeper.connect=192.168.1.217:5181,192.168.1.218:5181,192.168.1.219:5181

# Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=6000
rebalance.backoff.ms=2000
rebalance.max.retries=10
```

## 错误日志
开发环境下出现了该问题时相关日志信息
### 217上的kafka日志
```
[2020-04-22 10:43:36,369] WARN [ReplicaFetcherThread-1-219], Error in fetch kafka.server.ReplicaFetcherThread$FetchRequest@5641fe72 (kafka.server.ReplicaFetcherThread)
java.io.IOException: Connection to 219 was disconnected before the response was read
        at kafka.utils.NetworkClientBlockingOps$$anonfun$blockingSendAndReceive$extension$1$$anonfun$apply$1.apply(NetworkClientBlockingOps.scala:115)
        at kafka.utils.NetworkClientBlockingOps$$anonfun$blockingSendAndReceive$extension$1$$anonfun$apply$1.apply(NetworkClientBlockingOps.scala:112)
        at scala.Option.foreach(Option.scala:257)
        at kafka.utils.NetworkClientBlockingOps$$anonfun$blockingSendAndReceive$extension$1.apply(NetworkClientBlockingOps.scala:112)
        at kafka.utils.NetworkClientBlockingOps$$anonfun$blockingSendAndReceive$extension$1.apply(NetworkClientBlockingOps.scala:108)
        at kafka.utils.NetworkClientBlockingOps$.recursivePoll$1(NetworkClientBlockingOps.scala:137)
        at kafka.utils.NetworkClientBlockingOps$.kafka$utils$NetworkClientBlockingOps$$pollContinuously$extension(NetworkClientBlockingOps.scala:143)
        at kafka.utils.NetworkClientBlockingOps$.blockingSendAndReceive$extension(NetworkClientBlockingOps.scala:108)
        at kafka.server.ReplicaFetcherThread.sendRequest(ReplicaFetcherThread.scala:253)
        at kafka.server.ReplicaFetcherThread.fetch(ReplicaFetcherThread.scala:238)
        at kafka.server.ReplicaFetcherThread.fetch(ReplicaFetcherThread.scala:42)
        at kafka.server.AbstractFetcherThread.processFetchRequest(AbstractFetcherThread.scala:118)
        at kafka.server.AbstractFetcherThread.doWork(AbstractFetcherThread.scala:103)
        at kafka.utils.ShutdownableThread.run(ShutdownableThread.scala:63)
```

### 218上的kafka日志
```
[2020-04-22 10:59:42,687] WARN [ReplicaFetcherThread-1-219], Error in fetch kafka.server.ReplicaFetcherThread$FetchRequest@1722c608 (kafka.server.ReplicaFetcherThread)
java.io.IOException: Connection to 219 was disconnected before the response was read
        at kafka.utils.NetworkClientBlockingOps$$anonfun$blockingSendAndReceive$extension$1$$anonfun$apply$1.apply(NetworkClientBlockingOps.scala:115)
        at kafka.utils.NetworkClientBlockingOps$$anonfun$blockingSendAndReceive$extension$1$$anonfun$apply$1.apply(NetworkClientBlockingOps.scala:112)
        at scala.Option.foreach(Option.scala:257)
        at kafka.utils.NetworkClientBlockingOps$$anonfun$blockingSendAndReceive$extension$1.apply(NetworkClientBlockingOps.scala:112)
        at kafka.utils.NetworkClientBlockingOps$$anonfun$blockingSendAndReceive$extension$1.apply(NetworkClientBlockingOps.scala:108)
        at kafka.utils.NetworkClientBlockingOps$.recursivePoll$1(NetworkClientBlockingOps.scala:137)
        at kafka.utils.NetworkClientBlockingOps$.kafka$utils$NetworkClientBlockingOps$$pollContinuously$extension(NetworkClientBlockingOps.scala:143)
        at kafka.utils.NetworkClientBlockingOps$.blockingSendAndReceive$extension(NetworkClientBlockingOps.scala:108)
        at kafka.server.ReplicaFetcherThread.sendRequest(ReplicaFetcherThread.scala:253)
        at kafka.server.ReplicaFetcherThread.fetch(ReplicaFetcherThread.scala:238)
        at kafka.server.ReplicaFetcherThread.fetch(ReplicaFetcherThread.scala:42)
        at kafka.server.AbstractFetcherThread.processFetchRequest(AbstractFetcherThread.scala:118)
        at kafka.server.AbstractFetcherThread.doWork(AbstractFetcherThread.scala:103)
        at kafka.utils.ShutdownableThread.run(ShutdownableThread.scala:63)
```

### 219上的kafka日志
```
[root@RD-MQ-03 logs]# tail -f server.log
[2020-04-22 10:12:50,317] INFO Rolled new log segment for '__consumer_offsets-3' in 26 ms. (kafka.log.Log)
[2020-04-22 10:14:00,179] INFO Deleting segment 485381953 from log __consumer_offsets-3. (kafka.log.Log)
[2020-04-22 10:14:00,179] INFO Deleting segment 0 from log __consumer_offsets-3. (kafka.log.Log)
[2020-04-22 10:14:00,181] INFO Deleting index /opt/data/kafka/__consumer_offsets-3/00000000000485381953.index.deleted (kafka.log.OffsetIndex)
[2020-04-22 10:14:00,181] INFO Deleting index /opt/data/kafka/__consumer_offsets-3/00000000000000000000.index.deleted (kafka.log.OffsetIndex)
[2020-04-22 10:14:00,181] INFO Deleting index /opt/data/kafka/__consumer_offsets-3/00000000000485381953.timeindex.deleted (kafka.log.TimeIndex)
[2020-04-22 10:14:00,181] INFO Deleting index /opt/data/kafka/__consumer_offsets-3/00000000000000000000.timeindex.deleted (kafka.log.TimeIndex)
[2020-04-22 10:14:01,478] INFO Deleting segment 486188481 from log __consumer_offsets-3. (kafka.log.Log)
[2020-04-22 10:14:01,517] INFO Deleting index /opt/data/kafka/__consumer_offsets-3/00000000000486188481.index.deleted (kafka.log.OffsetIndex)
[2020-04-22 10:14:01,517] INFO Deleting index /opt/data/kafka/__consumer_offsets-3/00000000000486188481.timeindex.deleted (kafka.log.TimeIndex)

```

### ZK 日志
zk日志正常，zk数据文件正常（大小正常，有定期清理）


## 问题排查记录

### 检查网络
实际上这三个机器都是互通的

#### ping
```
# 217

[root@RD-MQ-01 ~]# ping 192.168.1.219
PING 192.168.1.219 (192.168.1.219) 56(84) bytes of data.
64 bytes from 192.168.1.219: icmp_seq=1 ttl=64 time=1.23 ms
64 bytes from 192.168.1.219: icmp_seq=2 ttl=64 time=1.79 ms

# 218

[root@RD-MQ-02 ~]# ping 192.168.1.219
PING 192.168.1.219 (192.168.1.219) 56(84) bytes of data.
64 bytes from 192.168.1.219: icmp_seq=1 ttl=64 time=1.76 ms
64 bytes from 192.168.1.219: icmp_seq=2 ttl=64 time=1.28 ms

# 219

[root@RD-MQ-03 ~]# ping 192.168.1.218
PING 192.168.1.218 (192.168.1.218) 56(84) bytes of data.
64 bytes from 192.168.1.218: icmp_seq=1 ttl=64 time=0.987 ms
64 bytes from 192.168.1.218: icmp_seq=2 ttl=64 time=1.18 ms
```

#### telnet
telnet 也是通的
```
[root@RD-MQ-02 ~]# telnet 192.168.1.219 9092
Trying 192.168.1.219...
Connected to 192.168.1.219.
Escape character is '^]'.

aa
^]
telnet> q
Connection closed.
```

#### netstat
```
[root@RD-MQ-02 ~]# netstat -an | grep 219:9092
tcp        0      0 ::ffff:192.168.1.218:36841  ::ffff:192.168.1.219:9092   FIN_WAIT2   
tcp        0      0 ::ffff:192.168.1.218:36840  ::ffff:192.168.1.219:9092   FIN_WAIT2   
tcp        0      0 ::ffff:192.168.1.218:36842  ::ffff:192.168.1.219:9092   ESTABLISHED 
tcp        0      0 ::ffff:192.168.1.218:36839  ::ffff:192.168.1.219:9092   FIN_WAIT2   
tcp        0      0 ::ffff:192.168.1.218:36837  ::ffff:192.168.1.219:9092   FIN_WAIT2   
tcp        0      0 ::ffff:192.168.1.218:36844  ::ffff:192.168.1.219:9092   ESTABLISHED 
tcp        0      0 ::ffff:192.168.1.218:36838  ::ffff:192.168.1.219:9092   FIN_WAIT2   
tcp        0      0 ::ffff:192.168.1.218:36843  ::ffff:192.168.1.219:9092   ESTABLISHED 
```
连接一直在建立，然后关闭

### 检查Zookeeper
ZK 能正常操作读写数据

在ZK中能看到3个kafka的brokers
```
[zk: 192.168.1.218:5181(CONNECTED) 4] ls /brokers/ids
[217, 218, 219]
```

### 检查kafka manager
只能查询到 Brokers 信息

无法查看topic 和 Consumers

### 使用kafka命令查看

#### topic list
分别在三个机器上查看topic信息，都能列出
```
./kafka-topics.sh --list --zookeeper 192.168.1.217:5181
```
#### topic describe
挑选一个测试队列【st.topic.yery.test】 查看详情：
```
[root@RD-MQ-01 bin]#  ./kafka-topics.sh --zookeeper 192.168.1.217:5181 --topic st.topic.yery.test --describe
Topic:st.topic.yery.test        PartitionCount:32       ReplicationFactor:3     Configs:
        Topic: st.topic.yery.test       Partition: 0    Leader: 218     Replicas: 218,217,219   Isr: 217,218,219
        Topic: st.topic.yery.test       Partition: 1    Leader: 219     Replicas: 219,218,217   Isr: 219
        Topic: st.topic.yery.test       Partition: 2    Leader: 217     Replicas: 217,219,218   Isr: 217,218,219
        Topic: st.topic.yery.test       Partition: 3    Leader: 218     Replicas: 218,219,217   Isr: 217,218,219
        Topic: st.topic.yery.test       Partition: 4    Leader: 219     Replicas: 219,217,218   Isr: 219,217,218
        Topic: st.topic.yery.test       Partition: 5    Leader: 217     Replicas: 217,218,219   Isr: 217,218,219
        Topic: st.topic.yery.test       Partition: 6    Leader: 218     Replicas: 218,217,219   Isr: 217,218,219
        Topic: st.topic.yery.test       Partition: 7    Leader: 219     Replicas: 219,218,217   Isr: 219
        Topic: st.topic.yery.test       Partition: 8    Leader: 217     Replicas: 217,219,218   Isr: 217,218,219
        Topic: st.topic.yery.test       Partition: 9    Leader: 218     Replicas: 218,219,217   Isr: 218,217,219
        Topic: st.topic.yery.test       Partition: 10   Leader: 219     Replicas: 219,217,218   Isr: 219
        Topic: st.topic.yery.test       Partition: 11   Leader: 217     Replicas: 217,218,219   Isr: 217,218,219
        Topic: st.topic.yery.test       Partition: 12   Leader: 218     Replicas: 218,217,219   Isr: 218,217,219
        Topic: st.topic.yery.test       Partition: 13   Leader: 219     Replicas: 219,218,217   Isr: 219,217,218
        Topic: st.topic.yery.test       Partition: 14   Leader: 217     Replicas: 217,219,218   Isr: 217,218,219
        Topic: st.topic.yery.test       Partition: 15   Leader: 218     Replicas: 218,219,217   Isr: 218,217,219
        Topic: st.topic.yery.test       Partition: 16   Leader: 219     Replicas: 219,217,218   Isr: 219,217,218
        Topic: st.topic.yery.test       Partition: 17   Leader: 217     Replicas: 217,218,219   Isr: 217,218,219
        Topic: st.topic.yery.test       Partition: 18   Leader: 218     Replicas: 218,217,219   Isr: 217,218,219
        Topic: st.topic.yery.test       Partition: 19   Leader: 219     Replicas: 219,218,217   Isr: 219,217,218
        Topic: st.topic.yery.test       Partition: 20   Leader: 217     Replicas: 217,219,218   Isr: 217,218,219
        Topic: st.topic.yery.test       Partition: 21   Leader: 218     Replicas: 218,219,217   Isr: 218,217,219
        Topic: st.topic.yery.test       Partition: 22   Leader: 219     Replicas: 219,217,218   Isr: 219,217,218
        Topic: st.topic.yery.test       Partition: 23   Leader: 217     Replicas: 217,218,219   Isr: 217,218,219
        Topic: st.topic.yery.test       Partition: 24   Leader: 218     Replicas: 218,217,219   Isr: 218,217,219
        Topic: st.topic.yery.test       Partition: 25   Leader: 219     Replicas: 219,218,217   Isr: 219
        Topic: st.topic.yery.test       Partition: 26   Leader: 217     Replicas: 217,219,218   Isr: 217,218,219
        Topic: st.topic.yery.test       Partition: 27   Leader: 218     Replicas: 218,219,217   Isr: 218,217,219
        Topic: st.topic.yery.test       Partition: 28   Leader: 219     Replicas: 219,217,218   Isr: 219,217,218
        Topic: st.topic.yery.test       Partition: 29   Leader: 217     Replicas: 217,218,219   Isr: 217,218,219
        Topic: st.topic.yery.test       Partition: 30   Leader: 218     Replicas: 218,217,219   Isr: 218,217,219
        Topic: st.topic.yery.test       Partition: 31   Leader: 219     Replicas: 219,218,217   Isr: 219,217,218
```

#### consumer-groups list
查看消费者
```
[root@RD-MQ-02 bin]# ./kafka-consumer-groups.sh --bootstrap-server 192.168.1.217:9092 --list
by.consumer.online.storage
dev.consumer.binlog.analysis
by.consumer.alarm.platform
dp.command.consumer
qa.consumer.binlog.install
qa.consumer.binlog.zone
dp.device.consumer
by.consumer.alarm.zone.normal
qa.consumer.binlog.business
```

#### consumer-groups describe
僵死，无法列出消费者的相关信息
```
[root@RD-MQ-02 bin]# ./kafka-consumer-groups.sh --bootstrap-server 192.168.1.217:9092 --group dp.device.consumer --describe
GROUP                          TOPIC                          PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             OWNER

```
换一个消费者组  
能看到两个分区的消费信息，实际上应该是32个分区
```
[root@RD-MQ-01 bin]# ./kafka-consumer-groups.sh --bootstrap-server 192.168.1.217:9092,192.168.1.218:9092,192.168.1.219:9092 --group by.consumer.online.storage --describe
GROUP                          TOPIC                          PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             OWNER
by.consumer.online.storage     by.topic.dp.online.device.message.info 0          515             515             0               consumer-1_/192.168.1.216
by.consumer.online.storage     by.topic.dp.online.device.message.info 1          11              11              0               consumer-1_/192.168.1.216
```

### 使用Kafka命令行工具测试
用命令行生产者 和 消费者 进行测试

#### 消费者
无法消费到数据任何，没有错误
```
./kafka-console-consumer.sh --bootstrap-server 192.168.1.217:9092,192.168.1.218:9092,192.168.1.219:9092 --topic by.topic.install.car.device --from-beginning
```
#### 生产者
发送数据报错
```
[root@RD-MQ-02 bin]# ./kafka-console-producer.sh --broker-list 192.168.1.217:9092,192.168.1.218:9092 --topic by.topic.install.car.device
111111
2222
3333
[2020-04-22 17:09:45,589] WARN Got error produce response with correlation id 2 on topic-partition by.topic.install.car.device-19, retrying (2 attempts left). Error: NETWORK_EXCEPTION (org.apache.kafka.clients.producer.internals.Sender)
[2020-04-22 17:09:45,592] WARN Got error produce response with correlation id 2 on topic-partition by.topic.install.car.device-1, retrying (2 attempts left). Error: NETWORK_EXCEPTION (org.apache.kafka.clients.producer.internals.Sender)
[2020-04-22 17:09:45,592] WARN Got error produce response with correlation id 1 on topic-partition by.topic.install.car.device-10, retrying (2 attempts left). Error: NETWORK_EXCEPTION (org.apache.kafka.clients.producer.internals.Sender)
5
[2020-04-22 17:09:48,368] WARN Got error produce response with correlation id 6 on topic-partition by.topic.install.car.device-28, retrying (2 attempts left). Error: NETWORK_EXCEPTION (org.apache.kafka.clients.producer.internals.Sender)
[2020-04-22 17:09:48,369] WARN Got error produce response with correlation id 5 on topic-partition by.topic.install.car.device-10, retrying (1 attempts left). Error: NETWORK_EXCEPTION (org.apache.kafka.clients.producer.internals.Sender)
[2020-04-22 17:09:48,369] WARN Got error produce response with correlation id 4 on topic-partition by.topic.install.car.device-19, retrying (1 attempts left). Error: NETWORK_EXCEPTION (org.apache.kafka.clients.producer.internals.Sender)
[2020-04-22 17:09:48,369] WARN Got error produce response with correlation id 4 on topic-partition by.topic.install.car.device-1, retrying (1 attempts left). Error: NETWORK_EXCEPTION (org.apache.kafka.clients.producer.internals.Sender)
[2020-04-22 17:09:49,978] WARN Got error produce response with correlation id 8 on topic-partition by.topic.install.car.device-1, retrying (0 attempts left). Error: NETWORK_EXCEPTION (org.apache.kafka.clients.producer.internals.Sender)
[2020-04-22 17:09:49,978] WARN Got error produce response with correlation id 8 on topic-partition by.topic.install.car.device-10, retrying (0 attempts left). Error: NETWORK_EXCEPTION (org.apache.kafka.clients.producer.internals.Sender)
[2020-04-22 17:09:49,979] WARN Got error produce response with correlation id 8 on topic-partition by.topic.install.car.device-19, retrying (0 attempts left). Error: NETWORK_EXCEPTION (org.apache.kafka.clients.producer.internals.Sender)
[2020-04-22 17:09:49,979] WARN Got error produce response with correlation id 8 on topic-partition by.topic.install.car.device-28, retrying (1 attempts left). Error: NETWORK_EXCEPTION (org.apache.kafka.clients.producer.internals.Sender)
[2020-04-22 17:09:51,587] ERROR Error when sending message to topic by.topic.install.car.device with key: null, value: 4 bytes with error: (org.apache.kafka.clients.producer.internals.ErrorLoggingCallback)
org.apache.kafka.common.errors.NetworkException: The server disconnected before a response was received.
[2020-04-22 17:09:51,593] ERROR Error when sending message to topic by.topic.install.car.device with key: null, value: 6 bytes with error: (org.apache.kafka.clients.producer.internals.ErrorLoggingCallback)
org.apache.kafka.common.errors.NetworkException: The server disconnected before a response was received.
[2020-04-22 17:09:51,593] ERROR Error when sending message to topic by.topic.install.car.device with key: null, value: 4 bytes with error: (org.apache.kafka.clients.producer.internals.ErrorLoggingCallback)
org.apache.kafka.common.errors.NetworkException: The server disconnected before a response was received.
[2020-04-22 17:09:51,594] WARN Got error produce response with correlation id 10 on topic-partition by.topic.install.car.device-28, retrying (0 attempts left). Error: NETWORK_EXCEPTION (org.apache.kafka.clients.producer.internals.Sender)
[2020-04-22 17:09:53,192] ERROR Error when sending message to topic by.topic.install.car.device with key: null, value: 1 bytes with error: (org.apache.kafka.clients.producer.internals.ErrorLoggingCallback)
org.apache.kafka.common.errors.NetworkException: The server disconnected before a response was received.
```

### 查进程信息
#### 进程占用的文件句柄数
##### 217 
```
[root@RD-MQ-01 opt]# lsof -p 13517 | wc -l
12824
```
##### 218
```
[root@RD-MQ-02 ~]# lsof -p 13557 | wc -l
12819
```
##### 219 
```
[root@RD-MQ-03 ~]# lsof -p 81068 | wc -l
326639
```
**可以看到219占用了非常多的文件句柄数**

#### 网络端口9092查看
##### 217
存在很多 time_wait 状态的连接
```
tcp        0      0 ::ffff:192.168.1.217:9092   ::ffff:192.168.1.215:47810  ESTABLISHED 13517/java          
tcp        0      0 ::ffff:192.168.1.217:54504  ::ffff:192.168.1.218:9092   TIME_WAIT   -                   
tcp        0  53321 ::ffff:192.168.1.217:33408  ::ffff:192.168.1.219:9092   FIN_WAIT1   -                   
tcp        0      0 ::ffff:192.168.1.217:9092   ::ffff:192.168.1.218:55812  ESTABLISHED 13517/java          
tcp        0  64905 ::ffff:192.168.1.217:39404  ::ffff:192.168.1.219:9092   FIN_WAIT1   -                   
tcp        0  54769 ::ffff:192.168.1.217:33158  ::ffff:192.168.1.219:9092   FIN_WAIT1   -                   
tcp        0      0 ::ffff:192.168.1.217:9092   ::ffff:192.168.1.216:35251  ESTABLISHED 13517/java          
tcp        0      0 ::ffff:192.168.1.217:39720  ::ffff:192.168.1.217:9092   TIME_WAIT   -                   
tcp        0      0 ::ffff:192.168.1.217:54466  ::ffff:192.168.1.218:9092   TIME_WAIT   -                   
tcp        0  48081 ::ffff:192.168.1.217:33639  ::ffff:192.168.1.219:9092   FIN_WAIT1   -                   
tcp        0  46505 ::ffff:192.168.1.217:39272  ::ffff:192.168.1.219:9092   FIN_WAIT1   -                   
tcp        0      0 ::ffff:192.168.1.217:39646  ::ffff:192.168.1.217:9092   TIME_WAIT   -                   
tcp        0  47953 ::ffff:192.168.1.217:33365  ::ffff:192.168.1.219:9092   FIN_WAIT1   -                   
tcp        0      0 ::ffff:192.168.1.217:54494  ::ffff:192.168.1.218:9092   TIME_WAIT   -                   
tcp        0  53449 ::ffff:192.168.1.217:33037  ::ffff:192.168.1.219:9092   FIN_WAIT1   -                   
tcp        0      0 ::ffff:192.168.1.217:40217  ::ffff:192.168.1.219:9092   FIN_WAIT2   -  
```
```
[root@RD-MQ-01 ~]# netstat -antp | grep 9092 | wc -l
324
```

##### 218
基本正常
```
tcp        0      0 ::ffff:192.168.1.218:9092   ::ffff:192.168.1.216:48369  ESTABLISHED 13557/java          
tcp        0      0 ::ffff:192.168.1.218:9092   ::ffff:192.168.1.214:42207  ESTABLISHED 13557/java          
tcp        0      0 ::ffff:192.168.1.218:9092   ::ffff:192.168.1.216:47984  ESTABLISHED 13557/java          
tcp        0      0 ::ffff:192.168.1.218:9092   ::ffff:192.168.1.216:44771  ESTABLISHED 13557/java          
tcp        0      0 ::ffff:192.168.1.218:9092   ::ffff:192.168.1.215:42045  ESTABLISHED 13557/java          
tcp        0      0 ::ffff:192.168.1.218:9092   ::ffff:192.168.1.215:51056  ESTABLISHED 13557/java          
tcp        0      0 ::ffff:192.168.1.218:9092   ::ffff:192.168.1.214:37368  ESTABLISHED 13557/java          
tcp        0      0 ::ffff:192.168.1.218:9092   ::ffff:192.168.1.217:59185  ESTABLISHED 13557/java          
tcp        0      0 ::ffff:192.168.1.218:9092   ::ffff:192.168.1.216:58880  ESTABLISHED 13557/java          
tcp        0      0 ::ffff:192.168.1.218:9092   ::ffff:192.168.1.216:44758  ESTABLISHED 13557/java          
tcp        0      0 ::ffff:192.168.1.218:9092   ::ffff:192.168.1.219:55019  ESTABLISHED 13557/java          
tcp        0      0 ::ffff:192.168.1.218:9092   ::ffff:192.168.1.215:35721  ESTABLISHED 13557/java          
tcp        0      0 ::ffff:192.168.1.218:9092   ::ffff:192.168.1.219:55014  ESTABLISHED 13557/java  
```
```
[root@RD-MQ-02 ~]# netstat -antp | grep 9092 | wc -l
77
```

##### 219
存在大量 CLOSE_WAIT 状态的连接
```
tcp      247      0 ::ffff:192.168.1.219:9092   ::ffff:192.168.1.217:58837  CLOSE_WAIT  81068/java          
tcp   178232      0 ::ffff:192.168.1.219:9092   ::ffff:192.168.1.217:34750  ESTABLISHED 81068/java          
tcp       55      0 ::ffff:192.168.1.219:9092   ::ffff:192.168.1.213:55954  CLOSE_WAIT  81068/java          
tcp      238      0 ::ffff:192.168.1.219:9092   ::ffff:192.168.1.217:40862  CLOSE_WAIT  81068/java          
tcp      226      0 ::ffff:192.168.1.219:9092   ::ffff:192.168.1.217:58967  CLOSE_WAIT  81068/java          
tcp    14535      0 ::ffff:192.168.1.219:9092   ::ffff:192.168.1.218:49425  CLOSE_WAIT  81068/java          
tcp    14986      0 ::ffff:192.168.1.219:9092   ::ffff:192.168.1.218:49330  CLOSE_WAIT  81068/java          
tcp      221      0 ::ffff:192.168.1.219:9092   ::ffff:192.168.1.217:37406  CLOSE_WAIT  81068/java          
tcp      223      0 ::ffff:192.168.1.219:9092   ::ffff:192.168.1.217:36455  CLOSE_WAIT  81068/java          
tcp    14620      0 ::ffff:192.168.1.219:9092   ::ffff:192.168.1.217:59460  CLOSE_WAIT  81068/java          
tcp      247      0 ::ffff:192.168.1.219:9092   ::ffff:192.168.1.217:59087  CLOSE_WAIT  81068/java  
```
数量远多于其他两台
```
[root@RD-MQ-03 opt]# netstat -antp | grep 9092 | wc -l
5589
```


### 查线程堆栈内存信息
#### 使用内存
基本正常
##### 217
```
ps aux|grep java|grep "kafka"|awk '{print $6}'     
2625240
```
##### 218
```
ps aux|grep java|grep "kafka"|awk '{print $6}'  
2071136
```
##### 219
```
ps aux|grep java|grep "kafka"|awk '{print $6}'  
1901924
```

#### jstack 
在219上找到死锁
```
Found one Java-level deadlock:
=============================
"executor-Heartbeat":
  waiting to lock monitor 0x00007f972c03db78 (object 0x000000070dbce4c8, a kafka.coordinator.GroupMetadata),
  which is held by "group-metadata-manager-0"
"group-metadata-manager-0":
  waiting to lock monitor 0x00007f979420e438 (object 0x0000000715a00000, a java.util.LinkedList),
  which is held by "kafka-request-handler-3"
"kafka-request-handler-3":
  waiting to lock monitor 0x00007f972c03db78 (object 0x000000070dbce4c8, a kafka.coordinator.GroupMetadata),
  which is held by "group-metadata-manager-0"

Java stack information for the threads listed above:
===================================================
"executor-Heartbeat":
	at kafka.coordinator.GroupCoordinator.onExpireHeartbeat(GroupCoordinator.scala:739)
	- waiting to lock <0x000000070dbce4c8> (a kafka.coordinator.GroupMetadata)
	at kafka.coordinator.DelayedHeartbeat.onExpiration(DelayedHeartbeat.scala:33)
	at kafka.server.DelayedOperation.run(DelayedOperation.scala:107)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
"group-metadata-manager-0":
	at kafka.server.DelayedOperationPurgatory$Watchers.tryCompleteWatched(DelayedOperation.scala:308)
	- waiting to lock <0x0000000715a00000> (a java.util.LinkedList)
	at kafka.server.DelayedOperationPurgatory.checkAndComplete(DelayedOperation.scala:234)
	at kafka.server.ReplicaManager.tryCompleteDelayedProduce(ReplicaManager.scala:199)
	at kafka.cluster.Partition.tryCompleteDelayedRequests(Partition.scala:374)
	at kafka.cluster.Partition.appendMessagesToLeader(Partition.scala:457)
	at kafka.coordinator.GroupMetadataManager$$anonfun$cleanupGroupMetadata$1$$anonfun$apply$10.apply(GroupMetadataManager.scala:600)
	at kafka.coordinator.GroupMetadataManager$$anonfun$cleanupGroupMetadata$1$$anonfun$apply$10.apply(GroupMetadataManager.scala:593)
	at scala.Option.foreach(Option.scala:257)
	at kafka.coordinator.GroupMetadataManager$$anonfun$cleanupGroupMetadata$1.apply(GroupMetadataManager.scala:593)
	- locked <0x000000070dbce4c8> (a kafka.coordinator.GroupMetadata)
	at kafka.coordinator.GroupMetadataManager$$anonfun$cleanupGroupMetadata$1.apply(GroupMetadataManager.scala:579)
	at scala.collection.Iterator$class.foreach(Iterator.scala:893)
	at kafka.utils.Pool$$anon$1.foreach(Pool.scala:89)
	at scala.collection.IterableLike$class.foreach(IterableLike.scala:72)
	at kafka.utils.Pool.foreach(Pool.scala:26)
	at kafka.coordinator.GroupMetadataManager.cleanupGroupMetadata(GroupMetadataManager.scala:579)
	at kafka.coordinator.GroupMetadataManager$$anonfun$1.apply$mcV$sp(GroupMetadataManager.scala:101)
	at kafka.utils.KafkaScheduler$$anonfun$1.apply$mcV$sp(KafkaScheduler.scala:110)
	at kafka.utils.CoreUtils$$anon$1.run(CoreUtils.scala:58)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.runAndReset(FutureTask.java:308)
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$301(ScheduledThreadPoolExecutor.java:180)
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:294)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
"kafka-request-handler-3":
	at kafka.coordinator.GroupMetadataManager.kafka$coordinator$GroupMetadataManager$$putCacheCallback$2(GroupMetadataManager.scala:301)
	- waiting to lock <0x000000070dbce4c8> (a kafka.coordinator.GroupMetadata)
	at kafka.coordinator.GroupMetadataManager$$anonfun$prepareStoreOffsets$1.apply(GroupMetadataManager.scala:357)
	at kafka.coordinator.GroupMetadataManager$$anonfun$prepareStoreOffsets$1.apply(GroupMetadataManager.scala:357)
	at kafka.server.DelayedProduce.onComplete(DelayedProduce.scala:123)
	at kafka.server.DelayedOperation.forceComplete(DelayedOperation.scala:70)
	at kafka.server.DelayedProduce.tryComplete(DelayedProduce.scala:105)
	at kafka.server.DelayedOperationPurgatory$Watchers.tryCompleteWatched(DelayedOperation.scala:315)
	- locked <0x0000000715a4d358> (a kafka.server.DelayedProduce)
	- locked <0x0000000715a00000> (a java.util.LinkedList)
	at kafka.server.DelayedOperationPurgatory.checkAndComplete(DelayedOperation.scala:234)
	at kafka.server.ReplicaManager.tryCompleteDelayedProduce(ReplicaManager.scala:199)
	at kafka.server.ReplicaManager$$anonfun$updateFollowerLogReadResults$2.apply(ReplicaManager.scala:909)
	at kafka.server.ReplicaManager$$anonfun$updateFollowerLogReadResults$2.apply(ReplicaManager.scala:902)
	at scala.collection.mutable.ResizableArray$class.foreach(ResizableArray.scala:59)
	at scala.collection.mutable.ArrayBuffer.foreach(ArrayBuffer.scala:48)
	at kafka.server.ReplicaManager.updateFollowerLogReadResults(ReplicaManager.scala:902)
	at kafka.server.ReplicaManager.fetchMessages(ReplicaManager.scala:475)
	at kafka.server.KafkaApis.handleFetchRequest(KafkaApis.scala:523)
	at kafka.server.KafkaApis.handle(KafkaApis.scala:79)
	at kafka.server.KafkaRequestHandler.run(KafkaRequestHandler.scala:60)
	at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.
```

#### jmap
dump 219 上的堆内存信息
```
jmap -dump:format=b,file=kafka-219-jmap.dump 81068
```
先保留现场，必要时再进行分析，文件大小：665M


## 相关资料
### 匹配度高的
#### KAFKA-3994
https://issues.apache.org/jira/browse/KAFKA-3994  
executor-Heartbeat线程 与 kafka-request-handler线程 互锁  
死锁信息与我们发现的有一点差异，没有group-metadata-manager线程   

版本比我们用低一个版本

> 影响版本：0.10.0.0      
> 修复版本：0.10.1.1, 0.10.2.0

#### KAFKA-4399
https://issues.apache.org/jira/browse/KAFKA-4399  
死锁信息与我们发现的有一个差异，没有executor-Heartbeat线程  

> 影响版本：0.10.1.0     
> 修复版本：0.10.1.1，0.10.2.0

#### KAFKA-4478
https://issues.apache.org/jira/browse/KAFKA-4478  
心跳执行器、组元数据管理器、请求处理器 相互锁死了  
这个的死锁堆栈信息与我们碰到的一致  

> 影响版本：0.10.1.0    
> 修复版本：无 （在bug 3994中被解决了）

#### KAFKA-4562
https://issues.apache.org/jira/browse/KAFKA-4562

死锁情况与我们的一样

> 影响版本：0.10.1.0    
> 修复版本：无 （在bug 3994中被解决了）

--------

### 相关性高的
#### 邮件列表一
https://users.kafka.apache.narkive.com/7ekMKZSX/deadlock-using-latest-0-10-1-kafka-release  

3个线程的死锁堆栈信息，和我们的一样，文件句柄耗尽  

作者已经很肯定 KAFKA-3994中的补丁可以很好地解决该问题。

> 关联：KAFKA-3994


#### 邮件列表二
https://users.kafka.apache.narkive.com/mP9QlK2j/connectivity-problem-with-controller-breaks-cluster

从描述的问题来看，我们的一致

> 关联：KAFKA-4477


#### KAFKA-4477
https://issues.apache.org/jira/browse/KAFKA-4477

我们的故障节点 ISR 也减少了，也有文件句柄数和死锁相关的描述

> 影响版本：0.10.1.0      
> 修复版本：0.10.1.1


## 结论
Kafka程序内部死锁导致了集群的故障问题  
根据官方文档这个问题在issues [KAFKA-4399] 中被解决了  
可以通过将现有的kafka升级至 kafka 0.10.1.1 来解决该问题

**kafka 0.10.1.1 发版说明**  
https://archive.apache.org/dist/kafka/0.10.1.1/RELEASE_NOTES.html.

官方发版说明中已经记录解决了bug [KAFKA-4399]  
- [KAFKA-3994] - Deadlock between consumer heartbeat expiration and offset commit.
- [KAFKA-4399] - Deadlock between cleanupGroupMetadata and offset commit

----

## 升级

### kafka服务端
服务器端升级一个小版本到 0.10.1.1  
https://archive.apache.org/dist/kafka/0.10.1.1/RELEASE_NOTES.html.  
这个版本主要是修复bug，没有什么新的改变，这样我们的数据应该是完全兼容的，只需要更换服务端的包，复制配置文件，之后逐个重启kafka服务即可。

#### 官方升级说明文档
http://kafka.apache.org/0101/documentation.html#upgrade


### 程序的客户端
全部的 **生产者** 和 **消费者** 都升级一个小版本到 0.10.1.1

```
<!-- https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients -->
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>0.10.1.1</version>
</dependency>
```
这样相关的代码应该都无需修改，Api不会有什么改动，只需要升级依赖包即可

修改父项目的pom文件依赖，子项目梳理一下，重新打包部署即可

### 其他问题
可能存在一些项目使用的是更老版本的API 和 连接方式等，如不是直连kafka而是老的连接zk的，这部分的项目需要修改代码


## 升级步骤记录
### 下载文件
```
wget https://archive.apache.org/dist/kafka/0.10.1.1/kafka_2.11-0.10.1.1.tgz
```
>这里选择的scala 版本是 2.11，之前装的也是2.11的

解压到新目录  
使用原来的配置位置  
注意：数据文件（log）的位置  

### 准备生产者和消费者
先准备一个生产者，产生数据
```
./kafka-console-producer.sh --broker-list 192.168.1.217:9092,192.168.1.218:9092,192.168.1.219:9092 --topic st.topic.yery.test
```
在准备一个消费者，消费数据
```
./kafka-console-consumer.sh --bootstrap-server 192.168.1.217:9092,192.168.1.218:9092,192.168.1.219:9092 --topic st.topic.yery.test --from-beginning
```
升级过程保持不变

### 停止节点219
先关闭节点219，观察集群状态：
217 218 报错了，但是集群整体运行正常  

### 测试验证
生产者能正常产生数据

消费者能正常消费数据

消费者会打印一个警告

进入kafka manager 可以查看到只有两个 Broker（217、218）  
查看topic st.topic.yery.test 的信息  
所有Partition 的leader都分到217 和 218 上  
In Sync Replicas 只有(217,218)  

### 修改配置文件
复制原配置文件到kafka_2.11-0.10.1.1 
```
cd kafka_2.11-0.10.1.0

# 复制启动脚本（自己写的）
cp start.sh ../kafka_2.11-0.10.1.1/

# 复制配置文件
cd config

cp server.properties ../../kafka_2.11-0.10.1.1/config/
cp log4j.properties ../../kafka_2.11-0.10.1.1/config/
```

### 在219启动新版本的kafka
直接启动219 （0.10.1.1版本的）

查看219日志，可以看到在恢复数据

数据恢复完成之后 会加入集群

加入集群后，217 和 218 会打印日志，将分区ISR从[217,218] 扩大到 [217,218,219]

在kafka manager 上可以看到Broker 219 加入了集群  
查看topic [st.topic.yery.test] 的信息  
Partition 的leader分到217，218 和 219 三台上了  
In Sync Replicas 有(217,218,219)  

### 再次验证测试
此时命令行 生产数据 消费数据都是正常的

### 应用程序测试
**直接验证binlog程序**  
程序在不升级kafka客户端包版本的情况下测试

在数据库中修改 by_device 表的记录x，将记录x的remark修改为‘kafka update’
可以在缓存管理中看到记录x的remark改成了‘kafka update’

对应的队列 qa.topic.binlog.base-by_device  
Sum of partition offsets	也加了1，表明消息发送消费正常

程序在不升级kafka-clients 包的情况下，也没有问题  

### 再升级217 和 218
用同样的步骤更换 217 和 218 上的kafka程序，配置文件就用原来的对应的配置文件

逐个关闭kafka程序再启动，等待数据库恢复，集群恢复到正常状态

### 验证老数据
升级完成之后再从头开始消费队列[st.topic.yery.test]的数据
```
./kafka-console-consumer.sh --bootstrap-server 192.168.1.217:9092,192.168.1.218:9092,192.168.1.219:9092 --topic st.topic.yery.test --from-beginning
```

**数据正常，升级完成**


----

## 补充

### 升级说明

#### 从0.8.x，0.9.x，0.10.0.x，0.10.1.x或0.10.2.x升级到0.11.0.0

http://kafka.apache.org/0102/documentation.html#upgrade

Kafka 0.11.0.0引入了新的消息格式版本以及有线协议更改。通过遵循以下建议的滚动升级计划，可以保证升级期间不会停机。但是，请在升级之前查看0.11.0.0中的显着更改。

从版本0.10.2开始，Java客户端（生产者和消费者）已经具有与较早的代理进行通信的能力。版本0.11.0的客户可以与版本0.10.0或更高版本的代理通信。但是，如果您的代理早于0.10.0，则必须先升级Kafka群集中的所有代理，然后再升级客户端。版本0.11.0代理支持0.8.x和更高版本的客户端。

---

###################################

---

## 升级之后的问题

**这是一个坑，升级之后还是有死锁问题，一个新的死锁**

由于 DelayedProduce  和 GroupMetadata 导致的死锁

```
Found one Java-level deadlock:
=============================
"executor-Heartbeat":
  waiting to lock monitor 0x00007f113c00b678 (object 0x00000000c8ab9028, a kafka.server.DelayedProduce),
  which is held by "kafka-scheduler-5"
"kafka-scheduler-5":
  waiting to lock monitor 0x00007f1918013848 (object 0x00000000ea18c4d8, a kafka.coordinator.GroupMetadata),
  which is held by "executor-Heartbeat"

Java stack information for the threads listed above:
```

bug:  
https://issues.apache.org/jira/browse/KAFKA-5970


修复版本：  
0.11.0.2， 1.0.0

在 0.10.X 版本中没有看到有修复该问题，且在0.11.x版本中又看到有内存泄漏问题，~~~


**好在这个问题出现的概率非常低，只在测试环境出现过1次**



