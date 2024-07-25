---
title: mysql-时区问题
date: 2021-04-13 23:41
tags: 
  - MySQL
categories:
  - [MySQL]
---


## 问题
插入时间到数据库中，会少13个小时（有时候是11个小时）

读取出来，则会自动加上少了的时间，读到的Date对象是正常的

## 环境

#### Linux

```
# uname -a
Linux had1 2.6.32-754.3.5.el6.x86_64 #1 SMP Tue Aug 14 20:46:41 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```

```
# cat /etc/centos-release 
CentOS release 6.10 (Final)
```

#### Mysql 版本
5.7.20

```
show variables like "%time_zone%";
```
Variable name | Value
---|---
system_time_zone | CST
time_zone | SYSTEM




## 原因
CTS 时区问题

mysql的驱动版本换成了：
mysql-connector-java 6.0.2

> 应该是6.X的版本更新了时区的判断方式，导致的

#### 时间类型
##### GMT
格林威治标准时间
##### UTC
世界协调时间
##### DST
夏日节约时间
##### CST时间
CST却同时可以代表如下 4 个不同的时区：

Central Standard Time (USA) UT-6:00

Central Standard Time (Australia) UT+9:30

China Standard Time UT+8:00

Cuba Standard Time UT-4:00

可见，CST可以同时表示美国，澳大利亚，中国，古巴四个国家的标准时间。

## 解决版本
### 一 更换驱动版本
换成之前一直用的的 5.1.45

### 二 连接串指定时区
**serverTimezone**
```
jdbc.url=jdbc:mysql://192.168.1.211:3306/analysis?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=Asia/Shanghai
```

### 三 mysql中指定

```
# vim /etc/my.cnf 

##在[mysqld]区域中加上
default-time_zone = ‘+8:00‘

##重启mysql使新时区生效
# /etc/init.d/mysqld restart 
```




