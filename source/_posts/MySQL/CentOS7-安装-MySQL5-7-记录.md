---
title: CentOS7-安装-MySQL5-7-记录
date: 2022-02-12 23:41
tags: 
  - MySQL
categories:
  - [MySQL]
---

*2017年11月*

---

### 系统版本
CentOS Linux 7 (Core)

```
 uname -a
Linux localhost.localdomain 3.10.0-327.el7.x86_64 #1 SMP Thu Nov 19 22:10:57 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
```


### 安装MySql的源到系统中
MySql 5.7

下载对应版本的源rpm文件
```
wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm

```

安装
```
sudo rpm -Uvh mysql57-community-release-el7-11.noarch.rpm 
warning: mysql57-community-release-el7-11.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:mysql57-community-release-el7-11 ################################# [100%]

```

默认是启动5.7版本的

```
 yum repolist all | grep mysql
mysql-cluster-7.5-community/x86_64 MySQL Cluster 7.5 Community    disabled
mysql-cluster-7.5-community-source MySQL Cluster 7.5 Community -  disabled
mysql-cluster-7.6-community/x86_64 MySQL Cluster 7.6 Community    disabled
mysql-cluster-7.6-community-source MySQL Cluster 7.6 Community -  disabled
mysql-connectors-community/x86_64  MySQL Connectors Community     enabled:    42
mysql-connectors-community-source  MySQL Connectors Community - S disabled
mysql-tools-community/x86_64       MySQL Tools Community          enabled:    51
mysql-tools-community-source       MySQL Tools Community - Source disabled
mysql-tools-preview/x86_64         MySQL Tools Preview            disabled
mysql-tools-preview-source         MySQL Tools Preview - Source   disabled
mysql55-community/x86_64           MySQL 5.5 Community Server     disabled
mysql55-community-source           MySQL 5.5 Community Server - S disabled
mysql56-community/x86_64           MySQL 5.6 Community Server     disabled
mysql56-community-source           MySQL 5.6 Community Server - S disabled
mysql57-community/x86_64           MySQL 5.7 Community Server     enabled:   227
mysql57-community-source           MySQL 5.7 Community Server - S disabled
mysql80-community/x86_64           MySQL 8.0 Community Server     disabled
mysql80-community-source           MySQL 8.0 Community Server - S disabled
```


### 安装MySQL
```
 sudo yum install mysql-community-server
```


### 修改配置文件，调整数据文件存放目录
```
vi /etc/my.cnf

  datadir=/data/mysql/datadir
```

创建对应的目录，调整权限

mkdir /data/mysql/datadir

chown mysql:mysql /data/mysql




启动报错
```
systemctl start mysqld.service

Job for mysqld.service failed because the control process exited with error code. See "systemctl status mysqld.service" and "journalctl -xe" for details.

```

日志文件中说是文件权限问题，但没看到是那个文件

调整一下/var/lib/mysql 目录
```
 ls -al
total 8
drwxr-x--x.  2 mysql mysql   17 Nov  7 16:27 .
drwxr-xr-x. 27 root  root  4096 Nov  7 16:00 ..
-rw-------.  1 root  root  1024 Nov  7 16:27 .rnd
[root@localhost mysql]# chown mysql:mysql -R /var/lib/mysql
[root@localhost mysql]# ls -al
total 8
drwxr-x--x.  2 mysql mysql   17 Nov  7 16:27 .
drwxr-xr-x. 27 root  root  4096 Nov  7 16:00 ..
-rw-------.  1 mysql mysql 1024 Nov  7 16:27 .rnd

```

再启动还是报错


后来尝试了很多方法之后，把数据目录改回去才能启动


启动之后获取密码：

```
shell> sudo grep 'temporary password' /var/log/mysqld.log


2017-11-07T09:10:43.302284Z 1 [Note] A temporary password is generated for root@localhost: -u(qyyu>L2n3

```

通过临时密码登录并修改root密码

```
shell> mysql -uroot -p

ALTER USER 'root'@'localhost' IDENTIFIED BY 'Root@666';
```


修改了配置后再次测试，还是不行

```
datadir=/data/mysql/datadir

```


修改/etc/selinux/config文件中设置SELINUX=disabled ，然后重启服务器。

再次启动可以了


设置Mysql远程访问
```
grant all privileges on *.* to 'root'@'%' identified by 'Root@666' with grant option;
```
使授权生效
```
flush privileges;

```

```
#关闭防火墙
systemctl stop firewalld.service

#关闭开机启动
systemctl disable firewalld.service
```

### 数据库导入导出
先在目标库中创建表

在目标机器上导出
```
mysqldump -udap -p -h192.168.1.215 dap > dap.sql

```

进入mysql
```
mysql -udap -p

#切换数据库
use dap;

#导入数据： 
source /data/mysql/215/dap.sql
```

测试的时候发现部分表，比如user表，permission表等创建不了，换成root用户就好了


