---
title: Redis集群配置与测试
date: 2018-06-04 23:41
tags: 
  - Redis
categories:
  - [Redis]
---

>环境：  
系统：CentOS 7  
Redis： 4.0.1

测试记录，按照操作流程随手记

---

创建几个测试目录，在里面放redis.conf文件
```
$ ll
总用量 1708
drwxr-xr-x 2 root root      23 5月  31 10:30 d6379
drwxr-xr-x 2 root root      23 5月  31 10:31 d6380
drwxr-xr-x 2 root root      23 5月  31 10:32 d6381
drwxr-xr-x 2 root root      23 5月  31 10:32 d6382
drwxr-xr-x 2 root root      23 5月  31 10:32 d6383
drwxr-xr-x 2 root root      23 5月  31 10:32 d6384
drwxr-xr-x 2 root root      23 5月  31 10:33 d6385
```
文件内容如下，每个文件的端口不同，最好与目录对应，方便记忆
```
port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

启动6个redis实例  
注意要分别进入到每个目录中启动，启动后会自动在目录下创建nodes.conf文件
```
cd d6379
/usr/local/redis/bin/redis-server redis.conf 

cd ..
cd d6380
/usr/local/redis/bin/redis-server redis.conf 
...
...
```

查看一下进程
```
# ps -ef | grep redis
root      9608     1  0 11:10 ?        00:00:00 /usr/local/redis/bin/redis-server 0.0.0.0:6379 [cluster]
root      9615     1  0 11:11 ?        00:00:00 /usr/local/redis/bin/redis-server 0.0.0.0:6380 [cluster]
root      9620     1  0 11:11 ?        00:00:00 /usr/local/redis/bin/redis-server 0.0.0.0:6381 [cluster]
root      9625     1  0 11:12 ?        00:00:00 /usr/local/redis/bin/redis-server 0.0.0.0:6382 [cluster]
root      9630     1  0 11:12 ?        00:00:00 /usr/local/redis/bin/redis-server 0.0.0.0:6383 [cluster]
root      9635     1  0 11:12 ?        00:00:00 /usr/local/redis/bin/redis-server 0.0.0.0:6384 [cluster]
root      9640     1  0 11:12 ?        00:00:00 /usr/local/redis/bin/redis-server 0.0.0.0:6385 [cluster]
```

使用这些实例来创建集群， 并为每个节点编写配置文件。

通过使用 Redis 集群命令行工具 redis-trib， 编写节点配置文件的工作可以非常容易地完成：redis-trib 位于 Redis 源码的 src 文件夹中，它是一个 Ruby 程序，这个程序通过向实例发送特殊命令来完成创建新集群，检查集群，或者对集群进行重新分片（reshared）等工作。

```
./redis-trib.rb create --replicas 1 127.0.0.1:6379 127.0.0.1:6381 127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384 127.0.0.1:6385
```

报错：
```
/usr/bin/env: ruby: 没有那个文件或目录
```
安装ruby
```
yum install ruby
```

报错：
```
/usr/share/rubygems/rubygems/core_ext/kernel_require.rb:55:in `require': cannot load such file -- redis (LoadError)
        from /usr/share/rubygems/rubygems/core_ext/kernel_require.rb:55:in `require'
        from ./redis-trib.rb:25:in `<main>'
```
使用RubyGems安装ruby脚本所需要的redis包
```
gem install redis
```
报错:
```
Fetching: redis-4.0.1.gem (100%)
ERROR:  Error installing redis:
        redis requires Ruby version >= 2.2.2.
```
看了下当前安装的ruby版本
```
# ruby --version
ruby 2.0.0p648 (2015-12-16) [x86_64-linux]
```
CentOS7 yum库中ruby的版本只支持到2.0.0，使用RVM（Ruby Version Manager）程序升级Ruby的版本  
先安装RVM程序
```
curl -L get.rvm.io | bash -s stable 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   194  100   194    0     0    308      0 --:--:-- --:--:-- --:--:--   307
100 24361  100 24361    0     0  12772      0  0:00:01  0:00:01 --:--:-- 89410
Downloading https://github.com/rvm/rvm/archive/1.29.3.tar.gz
Downloading https://github.com/rvm/rvm/releases/download/1.29.3/1.29.3.tar.gz.asc
gpg: 已创建目录‘/root/.gnupg’
gpg: 新的配置文件‘/root/.gnupg/gpg.conf’已建立
gpg: 警告：在‘/root/.gnupg/gpg.conf’里的选项于此次运行期间未被使用
gpg: 钥匙环‘/root/.gnupg/pubring.gpg’已建立
gpg: 于 2017年09月11日 星期一 04时59分21秒 CST 创建的签名，使用 RSA，钥匙号 BF04FF17
gpg: 无法检查签名：没有公钥
Warning, RVM 1.26.0 introduces signed releases and automated check of signatures when GPG software found. Assuming you trust Michal Papis import the mpapis public key (downloading the signatures).

GPG signature verification failed for '/usr/local/rvm/archives/rvm-1.29.3.tgz' - 'https://github.com/rvm/rvm/releases/download/1.29.3/1.29.3.tar.gz.asc'! Try to install GPG v2 and then fetch the public key:

    gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3

or if it fails:

    command curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -

the key can be compared with:

    https://rvm.io/mpapis.asc
    https://keybase.io/mpapis

NOTE: GPG version 2.1.17 have a bug which cause failures during fetching keys from remote server. Please downgrade or upgrade to newer version (if available) or use the second method described above.
```
无法验证签名，按照提示下载公钥
>说明：  
>**gpg2 --recv-keys** : 该命令从密钥服务器下载一个或多个公钥。每个key-id都是一个密钥ID。该命令需要使用keyserver选项来指定gpg应该从哪个keyserver下载密钥。
```
# gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
gpg: 下载密钥‘D39DC0E3’，从 hkp 服务器 keys.gnupg.net
gpg: /root/.gnupg/trustdb.gpg：建立了信任度数据库
gpg: 密钥 D39DC0E3：公钥“Michal Papis (RVM signing) <mpapis@gmail.com>”已导入
gpg: 没有找到任何绝对信任的密钥
gpg: 合计被处理的数量：1
gpg:           已导入：1  (RSA: 1)
```
再次安装RVM
```
]# curl -L get.rvm.io | bash -s stable 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   194  100   194    0     0    303      0 --:--:-- --:--:-- --:--:--   302
100 24361  100 24361    0     0  11395      0  0:00:02  0:00:02 --:--:-- 41430
Downloading https://github.com/rvm/rvm/archive/1.29.3.tar.gz
Downloading https://github.com/rvm/rvm/releases/download/1.29.3/1.29.3.tar.gz.asc
gpg: 于 2017年09月11日 星期一 04时59分21秒 CST 创建的签名，使用 RSA，钥匙号 BF04FF17
gpg: 完好的签名，来自于“Michal Papis (RVM signing) <mpapis@gmail.com>”
gpg:               亦即“Michal Papis <michal.papis@toptal.com>”
gpg:               亦即“[jpeg image of size 5015]”
gpg: 警告：这把密钥未经受信任的签名认证！
gpg:       没有证据表明这个签名属于它所声称的持有者。
主钥指纹： 409B 6B17 96C2 7546 2A17  0311 3804 BB82 D39D C0E3
子钥指纹： 62C9 E5F4 DA30 0D94 AC36  166B E206 C29F BF04 FF17
GPG verified '/usr/local/rvm/archives/rvm-1.29.3.tgz'
Creating group 'rvm'

Installing RVM to /usr/local/rvm/
Installation of RVM in /usr/local/rvm/ is almost complete:

  * First you need to add all users that will be using rvm to 'rvm' group,
    and logout - login again, anyone using rvm will be operating with `umask u=rwx,g=rwx,o=rx`.

  * To start using RVM you need to run `source /etc/profile.d/rvm.sh`
    in all your open shell windows, in rare cases you need to reopen all shell windows.
```
设置下环境变量，在查看下rvm版本
```
[root@wwh214 redis]# source /etc/profile.d/rvm.sh
[root@wwh214 redis]# rvm --version
rvm 1.29.3 (latest) by Michal Papis, Piotr Kuczynski, Wayne E. Seguin [https://rvm.io]
```

使用RVM升级Ruby版本  
查看rvm库中已知的ruby版本
```
$ rvm list known
# MRI Rubies
[ruby-]1.8.6[-p420]
[ruby-]1.8.7[-head] # security released on head
[ruby-]1.9.1[-p431]
[ruby-]1.9.2[-p330]
[ruby-]1.9.3[-p551]
[ruby-]2.0.0[-p648]
[ruby-]2.1[.10]
[ruby-]2.2[.7]
[ruby-]2.3[.4]
[ruby-]2.4[.1]
ruby-head
```
安装2.4版本的ruby
```
$ rvm install 2.4.1
Searching for binary rubies, this might take some time.
Found remote file https://rvm_io.global.ssl.fastly.net/binaries/centos/7/x86_64/ruby-2.4.1.tar.bz2
Checking requirements for centos.
Installing requirements for centos.
Installing required packages: libffi-devel, readline-devel, sqlite-devel, libyaml-devel............
Requirements installation successful.
ruby-2.4.1 - #configure
ruby-2.4.1 - #download
...
...
```
在查看一下
```
[root@wwh214 redis]# rvm list

rvm rubies

=* ruby-2.4.1 [ x86_64 ]

# => - current
# =* - current && default
#  * - default

[root@wwh214 redis]# ruby --version
ruby 2.4.1p111 (2017-03-22 revision 58053) [x86_64-linux]
```
之前通过yum安装的ruby并没有在rvm管理中，将其卸载掉
```
yum remove ruby
```

再次使用RubyGems安装ruby脚本所需要的redis包
```
# gem install redis
Fetching: redis-4.0.1.gem (100%)
Successfully installed redis-4.0.1
Parsing documentation for redis-4.0.1
Installing ri documentation for redis-4.0.1
Done installing documentation for redis after 0 seconds
1 gem installed
```

再次使用命令创建集群
```
./redis-trib.rb create --replicas 1 127.0.0.1:6379 127.0.0.1:6381 127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384 127.0.0.1:6385
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
127.0.0.1:6379
127.0.0.1:6381
127.0.0.1:6382
Adding replica 127.0.0.1:6383 to 127.0.0.1:6379
Adding replica 127.0.0.1:6384 to 127.0.0.1:6381
Adding replica 127.0.0.1:6385 to 127.0.0.1:6382
M: b2ba9914a49563dabdf022c1f8031577e4b30e44 127.0.0.1:6379
   slots:0-5460 (5461 slots) master
M: d38eaf2cf3a5ed2a4d4b0bafcb174748fac009dc 127.0.0.1:6381
   slots:5461-10922 (5462 slots) master
M: b54907e579a7daf1fb7155a4f25f5ae485374636 127.0.0.1:6382
   slots:10923-16383 (5461 slots) master
S: 602191b3b062238e10c1284343b4677b51fd31a3 127.0.0.1:6383
   replicates b2ba9914a49563dabdf022c1f8031577e4b30e44
S: aa4feef4e0c426ead799d4b34288cb3c5b9baa58 127.0.0.1:6384
   replicates d38eaf2cf3a5ed2a4d4b0bafcb174748fac009dc
S: a642a61c228ea354d94c695ca9dcaa048750590e 127.0.0.1:6385
   replicates b54907e579a7daf1fb7155a4f25f5ae485374636
Can I set the above configuration? (type 'yes' to accept): 
```
这个命令在这里用于创建一个新的集群, 选项–replicas 1 表示我们希望为集群中的每个主节点创建一个从节点。之后跟着的其他参数则是这个集群实例的地址列表,3个master3个slave，redis-trib 会打印出一份预想中的配置给你看， 如果你觉得没问题的话， 就可以输入 yes ， redis-trib 就会将这份配置应用到集群当中,让各个节点开始互相通讯,最后可以得到如下信息：
```
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join...
>>> Performing Cluster Check (using node 127.0.0.1:6379)
M: b2ba9914a49563dabdf022c1f8031577e4b30e44 127.0.0.1:6379
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
S: a642a61c228ea354d94c695ca9dcaa048750590e 127.0.0.1:6385
   slots: (0 slots) slave
   replicates b54907e579a7daf1fb7155a4f25f5ae485374636
S: aa4feef4e0c426ead799d4b34288cb3c5b9baa58 127.0.0.1:6384
   slots: (0 slots) slave
   replicates d38eaf2cf3a5ed2a4d4b0bafcb174748fac009dc
M: d38eaf2cf3a5ed2a4d4b0bafcb174748fac009dc 127.0.0.1:6381
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
M: b54907e579a7daf1fb7155a4f25f5ae485374636 127.0.0.1:6382
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
S: 602191b3b062238e10c1284343b4677b51fd31a3 127.0.0.1:6383
   slots: (0 slots) slave
   replicates b2ba9914a49563dabdf022c1f8031577e4b30e44
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```


再对比一下node.conf配置文件：  
以d6381目录为例子，redis启动时创建的缺省文件是：
```
d38eaf2cf3a5ed2a4d4b0bafcb174748fac009dc :0@0 myself,master - 0 0 0 connected
vars currentEpoch 0 lastVoteEpoch 0
```
运行脚本之后，文件变成了：
```
602191b3b062238e10c1284343b4677b51fd31a3 127.0.0.1:6383@16383 slave b2ba9914a49563dabdf022c1f8031577e4b30e44 0 1527749631645 4 connected
d38eaf2cf3a5ed2a4d4b0bafcb174748fac009dc 127.0.0.1:6381@16381 myself,master - 0 1527749631000 2 connected 5461-10922
b54907e579a7daf1fb7155a4f25f5ae485374636 127.0.0.1:6382@16382 master - 0 1527749631144 3 connected 10923-16383
aa4feef4e0c426ead799d4b34288cb3c5b9baa58 127.0.0.1:6384@16384 slave d38eaf2cf3a5ed2a4d4b0bafcb174748fac009dc 0 1527749631244 5 connected
b2ba9914a49563dabdf022c1f8031577e4b30e44 127.0.0.1:6379@16379 master - 0 1527749630140 1 connected 0-5460
a642a61c228ea354d94c695ca9dcaa048750590e 127.0.0.1:6385@16385 slave b54907e579a7daf1fb7155a4f25f5ae485374636 0 1527749631000 6 connected
vars currentEpoch 6 lastVoteEpoch 0
```

使用redis-cli登录测试一下：
```
# /usr/local/redis/bin/redis-cli
127.0.0.1:6379> info
......
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6383,state=online,offset=742,lag=0
......

```
在往集群中某个redis中放入值的时候会计算key的hash所 属的槽，然后再计算槽是不是属于当前实例，如果不是会返回error，并且返回正确的节点地址和端口，如下：

```
127.0.0.1:6379> put aaa aaa
(error) ERR unknown command 'put'
127.0.0.1:6379> set aaa aaa
(error) MOVED 10439 127.0.0.1:6381

```
而在6381上可以设值成功
```
# /usr/local/redis/bin/redis-cli -p 6381
127.0.0.1:6381> set aaa aaa
OK
127.0.0.1:6381> 
```
不同的键值对存在不同的节点上，keys * 只会返回当前节点的所有key


