---
title: Redis安装
date: 2021-02-19 23:41
tags: 
  - Redis
categories:
  - [Redis]
---

# Redis 安装

#### 1. 安装 gcc-c++
```
yum install gcc-c++
```
一般都装好了

#### 2. 下载  
下载地址：http://download.redis.io/releases/  
这里下载最新的稳定版 (当前的稳定版本是：4.0.2)  

```
cd /var/tmp

wget http://download.redis.io/releases/redis-stable.tar.gz

#or

curl -O http://download.redis.io/releases/redis-stable.tar.gz
```

#### 3. 解压编译
```
tar -zxvf redis-stable.tar.gz

cd redis-stable

make
```

#### 4. 安装
```
make PREFIX=/usr/local/redis install
```

#### 5. 复制配置文件
```
cp redis.conf /usr/local/redis/

```

#### 6. 启动
```
cd /usr/local/redis/

ls

cd bin

./redis-server 


```

#### 7. 配置文件
修改配置文件
```
cd ..
vi redis.conf

```
后台启动
```
# 修改
# daemonize no
# 为 yes

daemonize yes
```
指定服务器绑定的IP地址
```
bind 0.0.0.0
```
监听端口号，默认为 6379
```
port 6379
```
定本地数据文件名，默认值为dump.rdb 
```
dbfilename dump.rdb
```
数据文件存放目录，可以改成绝对路径
```
#dir ./
dir /data/redis/data/
```

#### 8. 通过指定配置文件进行启动
实现后台运行等
```
bin/redis-server redis.conf 
```

#### 9. 关闭Redis
```
bin/redis-cli shutdown
```
*不要随便杀进程*


#### 9. 服务方式运行
进入到程序包的 **redis-4.0.2/utils** 目录中
执行./install_server.sh 文件，按照提示进行服务安装：
```
 ./install_server.sh 
Welcome to the redis service installer
This script will help you easily set up a running redis server

Please select the redis port for this instance: [6379] 
Selecting default: 6379
Please select the redis config file name [/etc/redis/6379.conf] /usr/local/redis/redis.conf
Please select the redis log file name [/var/log/redis_6379.log] 
Selected default - /var/log/redis_6379.log
Please select the data directory for this instance [/var/lib/redis/6379] /data/redis
Please select the redis executable path [] /usr/local/redis/bin/redis-server
Selected config:
Port           : 6379
Config file    : /usr/local/redis/redis.conf
Log file       : /var/log/redis_6379.log
Data dir       : /data/redis
Executable     : /usr/local/redis/bin/redis-server
Cli Executable : /usr/local/redis/bin/redis-cli
Is this ok? Then press ENTER to go on or Ctrl-C to abort.
Copied /tmp/6379.conf => /etc/init.d/redis_6379
Installing service...
Successfully added to chkconfig!
Successfully added to runlevels 345!
Starting Redis server...
Installation successful!

```
完成后将在/etc/rc.d/init.d目录生成一个redis_6379 文件，安装的服务也叫这个名字，服务默认开机启动。


启动redis服务:
```
service redis_6379 start
```

停止redis服务:
```
service redis_6379 stop
```

设为开机启动:
```
chkconfig redis_6379 on
```

关闭开机启动:
```
chkconfig redis_6379 off
```

