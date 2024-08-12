---
title: CentOS 7 离线安装 MySQL 5.7
date: 2017-09-07
tags: 
  - MySQL
  - CentOS 7
categories:
  - [MySQL]
---



1. 确定系统版本
```
cat /etc/redhat-release

# CentOS Linux release 7.2.1511 (Core) 

```
2. 下载对应版本的包  
包含依赖的捆绑包，如：   
MySQL-5.6.35-1.el7.x86_64.rpm-bundle.tar    
mysql-5.7.17-1.el7.x86_64.rpm-bundle.tar  


3. 卸载MariaDB
MariaDB数据库管理系统是MySQL的一个分支，主要由开源社区在维护，采用GPL授权许可。开发这个分支的原因之一是：甲骨文公司收购了MySQL后，有将MySQL闭源的潜在风险，因此社区采用分支的方式来避开这个风险。MariaDB的目的是完全兼容MySQL，包括API和命令行，使之能轻松成为MySQL的代替品。  
**查看当前安装的mariadb包**  
```
rpm -qa | grep mariadb

mariadb-devel-5.5.50-1.el7_2.x86_64
mariadb-libs-5.5.50-1.el7_2.x86_64

#卸载 mariadb

rpm -e mariadb-devel-5.5.50-1.el7_2.x86_64
#强制卸载
rpm -e --nodeps mariadb-libs-5.5.50-1.el7_2.x86_64

```

4. 创建用户
安装之前先添加mysql用户组及mysql用户
```
groupadd mysql  
  
useradd -r -g mysql mysql  
```


4. 安装  
**这里安装的是 MySQL 5.7**

```
 rpm -ivh mysql-community-common-5.7.17-1.el7.x86_64.rpm
警告：mysql-community-common-5.7.17-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-common-5.7.17-1.e################################# [100%]
   
```

```
rpm -ivh mysql-community-libs-5.7.17-1.el7.x86_64.rpm
警告：mysql-community-libs-5.7.17-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-libs-5.7.17-1.el7################################# [100%]
```

```
rpm -ivh mysql-community-client-5.7.17-1.el7.x86_64.rpm
警告：mysql-community-client-5.7.17-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-client-5.7.17-1.e################################# [100%]
```

```
rpm -ivh mysql-community-server-5.7.17-1.el7.x86_64.rpm
警告：mysql-community-server-5.7.17-1.el7.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql-community-server-5.7.17-1.e################################# [100%]
```


5. 启动
```
systemctl start mysqld
```

6. 登录
找到临时密码
```
grep 'temporary password' /var/log/mysqld.log  

2017-02-10T15:48:22.225974Z 1 [Note] A temporary password is generated for root@localhost: H7k#D;u;&aK(

```

登录成功
```
 mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 6
Server version: 5.7.17

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

7. 修改密码
```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';  
```
一般来说这个密码是属于一个弱强度密码，mysql不会接收这个密码，并产生下面的报错信息： 
```
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```

降低Mysql的密码检验强度：  
```
set global validate_password_policy=0;  
```
这个设置下会只检查长度，默认长度为8，也就是是说密码长度至少为8.
要查看这个长度的值，可以这样做：
```
select @@validate_password_length;  
```
修改密码长度：
```
set global validate_password_length=1;
```

然后就可以修改密码了


8. 设置Mysql远程访问
```
grant all privileges on *.* to 'root'@'%' identified by 'root' with grant option;
```
使授权生效
```
flush privileges;
```


