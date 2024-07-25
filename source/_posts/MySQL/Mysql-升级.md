---
title: Mysql-升级
date: 2020-06-25 23:41
tags: 
  - MySQL
categories:
  - [MySQL]
---

*2017年8月*

---
# 升级之前请备份数据
---

https://dev.mysql.com/downloads/repo/yum/

https://dev.mysql.com/doc/refman/5.7/en/upgrading.html

https://dev.mysql.com/doc/refman/5.7/en/mysql-upgrade.html

## 更新程序

### 使用MySQL Yum Repository升级MySQL
https://dev.mysql.com/doc/refman/5.7/en/updating-yum-repo.html


## 通过yum升级
1. 更换新的yum 资源库，或者禁用旧版本系列 启用新版本系列。
2. 通过以下命令升级MySQL及其组件
```
shell> sudo yum update mysql-server

```
或者，通过Yum更新系统上的所有内容来更新MySQL
```
shell> sudo yum update

```

3. 查看已经安装了的
```
shell> sudo yum list installed | grep "^mysql"

```

## 关闭旧的服务器
1. 配置MySQL通过设置innodb_fast_shutdown来 执行缓慢的关闭 0。例如：
```
mysql -u root -p --execute="SET GLOBAL innodb_fast_shutdown=0"
```
缓慢关闭时，InnoDB执行完全清除并在关闭之前更改缓冲区合并，从而确保数据文件在发布之间的文件格式不同的情况下完全准备好。

2. 关闭旧的MySQL服务器。例如：
```
mysqladmin -u root -p shutdown
```


## 升级数据

## mysql_upgrade - 检查和升级MySQL表
mysql_upgrade检查所有数据库中的所有表与当前版本的MySQL Server的不兼容性。mysql_upgrade还升级系统表，以便您可以利用可能已添加的新特权或功能。

如果mysql_upgrade发现表有可能的不兼容性，它会执行表检查，如果发现问题，则尝试进行表修复。如果表不能修复，请参见第2.11.3节“重建或修复表或索引”以获取手动表修复策略。

每次升级MySQL时，都 应该执行mysql_upgrade。

从MySQL 5.7.5开始，mysql_upgrade与MySQL服务器直接通信，发送执行升级所需的SQL语句。5.7.5之前， mysql_upgrade调用 mysql和mysqlcheck 客户端程序来执行所需的操作。对于较旧的实现，如果在Linux上从RPM软件包安装MySQL，则必须安装服务器和客户机RPM。 mysql_upgrade包含在服务器RPM中，但需要客户端RPM，因为后者包括 mysqlcheck。（请参见 第2.5.5节“使用Oracle的RPM软件包在Linux上安装MySQL”。）

要使用mysql_upgrade，请确保服务器正在运行。然后像这样调用它来检查和修复表并升级系统表：
```
shell> mysql_upgrade [options]

# 例如：
shell> mysql_upgrade -u root -p
```
运行mysql_upgrade后，停止服务器并重新启动，以便对系统表进行的任何更改生效。
```
mysqladmin -u root -p shutdown

```

