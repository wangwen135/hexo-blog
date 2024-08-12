---
title: Mysql数据文件存放位置调整
date: 2017-07-26 23:41
tags: 
  - MySQL
categories:
  - [MySQL]
---


## 查看数据文件存放位置
1. 通过命令查看
```
show variables like '%dir%'; 

show global variables like '%datadir%';
```
datadir 指数据文件存放位置

2. 查看配置文件
```
cat /etc/my.cnf
```
```
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.6/en/server-configuration-defaults.html

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

# Recommended in standard MySQL setup
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
~                                                                  
```

## 修改数据文件位置  
1. 停止mysql服务
```
service mysqld stop  
```
2. 创建新的数据库存放目录
```
mkdir /data/mysql  
```
3. 移动/复制之前存放数据库目录文件，到新的数据库存放目录位置 
```
cp -R /var/lib/mysql/* /data/mysql/ 
```
4. 修改mysql数据库目录权限
```
chown mysql:mysql -R /data/mysql/
```
5. 修改my.cnf配置文件
```
vi /etc/my.cnf
    datadir=/data/mysql （指定为新的数据存放目录）
```
6. 启动数据库服务
```
service mysqld start
```
