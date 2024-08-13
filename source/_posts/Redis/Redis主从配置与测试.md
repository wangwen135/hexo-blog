---
title: Redis主从配置与测试
date: 2019-02-19 23:41
tags: 
  - Redis
categories:
  - [Redis]
---

## 配置
先安装redis
```
make
make PREFIX=/usr/local/redis install
```

找个地方创建两个文件，如：
```
mkdir d6379
mkdir d6380
```
把redis.conf文件复制到这两个目录

进入d6379目录中，配置文件就改了
```
daemonize yes
```
启动第一个redis
```
/usr/local/redis/bin/redis-server redis.conf 
```
进入d6380目录中，配置文件修改为
```
port 6380
daemonize yes

# slaveof <masterip> <masterport>
slaveof 127.0.0.1 6379
```
启动从节点
```
/usr/local/redis/bin/redis-server redis.conf 
```

查看一下进程情况
```
ps -ef | grep redis
root      8428     1  0 17:06 ?        00:00:00 /usr/local/redis/bin/redis-server 127.0.0.1:6379
root      8453     1  0 17:16 ?        00:00:00 /usr/local/redis/bin/redis-server 127.0.0.1:6380
```

## 测试

再开一个shell窗口，分别登入两个redis  

登录主的
```
$ /usr/local/redis/bin/redis-cli 
127.0.0.1:6379> 
```

登录从的
```
$ /usr/local/redis/bin/redis-cli -p 6380
127.0.0.1:6380> 
```

分别查看他们的状态:  
主的
```
127.0.0.1:6379> info Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6380,state=online,offset=98,lag=0
```
从的
```
127.0.0.1:6380> info Replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
```

在主的上设置值
```
127.0.0.1:6379> set key1 vvv1
OK
127.0.0.1:6379> keys *
1) "key1"
```
在从的上查看
```
127.0.0.1:6380> keys *
1) "key1"
127.0.0.1:6380> get key1
"vvv1"
```
可以看到从的已经自动从主的上复制了数据

再试一下在从的上写数据
```
127.0.0.1:6380> set key2 vvv2
(error) READONLY You can't write against a read only slave.
```

## 主从切换

**SLAVEOF host port**  
SLAVEOF 命令用于在 Redis 运行时动态的修改复制(replication)功能的行为。

通过执行 SLAVEOF host port 命令，可以将当前服务器转变为指定服务器的从属服务器(slave server)。

如果当前服务器已经是某个主服务器(master server)的从属服务器，那么执行 SLAVEOF host port 将使当前服务器停止对旧主服务器的同步，丢弃旧数据集，转而开始对新主服务器进行同步。

另外，对一个从属服务器执行命令 SLAVEOF NO ONE 将使得这个从属服务器关闭复制功能，并从从属服务器转变回主服务器，原来同步所得的数据集不会被丢弃。

利用『 SLAVEOF NO ONE 不会丢弃同步所得数据集』这个特性，可以在主服务器失败的时候，将从属服务器用作新的主服务器，从而实现无间断运行。

---

将从服务器升级为主服务器，并且关闭复制功能
```
127.0.0.1:6380> SLAVEOF NO ONE
OK
127.0.0.1:6380> info Replication
# Replication
role:master
connected_slaves:0

```
此时可以写入数据
```
127.0.0.1:6380> set k5 'upgrade to master'
OK
```

将6379转成6380的从服务
```
127.0.0.1:6379> slaveof 127.0.0.1 6380
OK
127.0.0.1:6379> info Replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6380
master_link_status:up
master_last_io_seconds_ago:5
master_sync_in_progress:0
slave_repl_offset:2844
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:9c8b0e2e24636212f659b3f8bc17cbae51785db2
master_replid2:6e0c53cc1c7438c88273db17fa654f285026e1ce
master_repl_offset:2844
second_repl_offset:2777
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:2844
```
变成从的后会删除自己已有的数据，并将主服务器的数据复制过来  
可以看到之前添加的数据也复制过来了
```
127.0.0.1:6379> keys *
1) "kk2"
2) "k5"
3) "key1"
127.0.0.1:6379> get k5
"upgrade to master"
```


-----

主节点下的从节点的信息都可以通过info命令查看
```
127.0.0.1:6380> info Replication
# Replication
role:master
connected_slaves:3
slave0:ip=192.168.1.215,port=6379,state=online,offset=4768,lag=1
slave1:ip=192.168.1.215,port=6380,state=online,offset=4768,lag=0
slave2:ip=127.0.0.1,port=6379,state=online,offset=4768,lag=0
master_replid:d3ea04b9553a6c20b2bbe836dd35e2311ee1ed06
master_replid2:d7ded5c06b7d723590dd4aa8bee2b7020f3cbc96
master_repl_offset:4768
second_repl_offset:4617
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:4533
repl_backlog_histlen:236
```
