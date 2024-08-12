---
title: CentOS 7 安装配置MySQL 5.6
date: 2017-11-07
tags: 
  - MySQL
categories:
  - [MySQL]
---



CentOS7的yum源中默认好像是没有mysql的。为了解决这个问题，我们要先下载mysql的repo源。

**MySQL Yum Repository**  
https://dev.mysql.com/downloads/repo/yum/

### 1. 下载mysql的repo源
```
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
```

### 2. 安装mysql-community-release-el7-5.noarch.rpm包
```
sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
```
安装这个包后，会获得两个mysql的yum repo源：  
>/etc/yum.repos.d/mysql-community.repo  
/etc/yum.repos.d/mysql-community-source.repo  

### 3. 安装mysql
```
sudo yum install mysql-server
```
根据步骤安装就可以了，不过安装完成后，没有密码，需要重置密码。

### 4. 启动
```
service mysqld start
```

### 5. 重置密码

重置密码前，首先要登录
```
mysql -u root
```
登录时有可能报这样的错：  
ERROR 2002 (HY000): Can‘t connect to local MySQL server through socket ‘/var/lib/mysql/mysql.sock‘   
原因是/var/lib/mysql的访问权限问题。下面的命令把/var/lib/mysql的拥有者改为当前用户：
```
sudo chown -R openscanner:openscanner /var/lib/mysql
```
然后，重启服务：
```
service mysqld restart
```
接下来登录重置密码：
```
$ mysql -u root
mysql > use mysql;
mysql > update user set password=password('root') where user='root';
mysql > exit;
```

设置Mysql远程访问
```
grant all privileges on *.* to 'root'@'%' identified by 'root' with grant option;
```
使授权生效
```
flush privileges;
```

### 6. 开放3306端口
**CentOS 7  默认已经不适用这个防火墙了！**
```
sudo vim /etc/sysconfig/iptables
```
添加以下内容：
>-A INPUT -p tcp -m state --state NEW -m tcp --dport 3306 -j ACCEPT

保存后重启防火墙：
```
sudo service iptables restart
```


### 7. 设置开机启动

```
chkconfig --list

systemctl status mysqld

systemctl is-enabled mysqld
```

