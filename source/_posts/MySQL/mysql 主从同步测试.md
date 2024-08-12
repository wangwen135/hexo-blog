---
title: mysql 主从同步测试
date: 2018-10-10 23:41
tags: 
  - MySQL
  - binlog
categories:
  - [MySQL]
---


测试环境
- 系统：WIN 10
- MySQL版本：mysql-5.7.23

### 安装配置


#### 解压文件
使用mysql-5.7.23-winx64.zip包解压缩安装  
路径分别为：
- D:\mysql\mysql-5.7.23-winx64
- D:\mysql\mysql-5.7.23-winx64-3307

>解压安装路径随意指定，注意需要与my.ini文件中的路径一致



#### my.ini配置文件
分别在两个目录中创建my.ini 文件

>在同一台机器上测试，mysql实例使用不同端口  
>主数据库：3306 (默认)  
>从数据库：3307

内容如下：

##### 主库my.ini
```
[mysqld]
# set basedir to your installation path
basedir=D:/mysql/mysql-5.7.23-winx64
# set datadir to the location of your data directory
datadir=D:/mysql/mysql-5.7.23-winx64/data

# 唯一标识
server-id=1

# binlog文件名
log-bin=mysql-binlog

# 要写binlog的数据库，要同步多个数据库，就多加几个binlog-do-db=数据库名
binlog-do-db=mstest
binlog-do-db=test

# 要忽略的数据库
binlog-ignore-db=mysql

```
##### 从库my.ini
```
[mysqld]
# 指定端口
port=3307
# set basedir to your installation path
basedir=D:/mysql/mysql-5.7.23-winx64-3307
# set datadir to the location of your data directory
datadir=D:/mysql/mysql-5.7.23-winx64-3307/data

# 唯一标识
server-id=2

# 要复制多个数据库，就多加几个replicate-do-db=数据库名
replicate-do-db=mstest
replicate-do-db=test

# 要忽略的数据库
replicate-ignore-db=mysql
```

#### 初始化并启动数据库

分别初始化、启动两个数据库，并修改root密码  
```
mysqld --initialize --console

mysqld.exe --console

ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';
```
>具体见安装文档


#### 数据库配置
##### 主库配置 
登录主库
```
>mysql -uroot -p
```

主数据库创建用于同步的用户
```
GRANT REPLICATION SLAVE ON *.* TO 'mstest'@'%' IDENTIFIED BY '123456';
```

##### 从数据库配置
登录从库，指定端口3307
```
>mysql -uroot -p -P3307
```

配置从库连接到主库
```
change master to master_host='127.0.0.1',master_port=3306,
master_user='mstest',master_password='123456';

start slave;

```

#### 查看状态

##### 主库
```
mysql> show master status;
+---------------------+----------+--------------+------------------+-------------------+
| File                | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------------+----------+--------------+------------------+-------------------+
| mysql-binlog.000002 |      313 | mstest,test  | mysql            |                   |
+---------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```


##### 从库
```

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 127.0.0.1
                  Master_User: mstest
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-binlog.000002
          Read_Master_Log_Pos: 1306
               Relay_Log_File: wwh-relay-bin.000005
                Relay_Log_Pos: 323
        Relay_Master_Log_File: mysql-binlog.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: mstest,test
          Replicate_Ignore_DB: mysql
 ......
 ......
 ......
             Master_Server_Id: 1
                  Master_UUID: ea65d565-a8a0-11e8-807e-0a002700000a
             Master_Info_File: D:\mysql\mysql-5.7.23-winx64-3307\data\master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
......
......
1 row in set (0.00 sec)
```


### 同步测试

#### 创建数据库

#### 主库创建表，从库查看


#### 主库插入记录，从库查看




```
mysql> create database test;
Query OK, 1 row affected (0.01 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test               |
| wwh                |
+--------------------+
6 rows in set (0.00 sec)



mysql> status;
--------------
mysql  Ver 14.14 Distrib 5.7.23, for Win64 (x86_64)

Connection id:          2
Current database:
Current user:           root@localhost
SSL:                    Not in use
Using delimiter:        ;
Server version:         5.7.23 MySQL Community Server (GPL)
Protocol version:       10
Connection:             localhost via TCP/IP
Server characterset:    latin1
Db     characterset:    latin1
Client characterset:    gbk
Conn.  characterset:    gbk
TCP port:               3307
Uptime:                 7 min 37 sec




show binlog events\G


show master status\G




```

## 查询命令
### 在master上查看当前有多少个从节点
```
 select * from information_schema.processlist as p where p.command = 'Binlog Dump'; 
 
 ```
 
