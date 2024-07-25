---
title: Yum-安装-MySQL-5-7
date: 2020-12-15 23:41
tags: 
  - MySQL
categories:
  - [MySQL]
---

*2017年10月*

---

https://dev.mysql.com/doc/mysql-yum-repo-quick-guide/en/

## 1. 将MySQL Yum存储库添加到系统的存储库列表中
A. 转到MySQL Yum存储库的下载页面，网址为 http://dev.mysql.com/downloads/repo/yum

B. 选择并下载您的平台的发行包。

C. 使用以下命令安装下载的发行包，并platform-and-version-specific-package-name 使用下载的软件包的名称进行替换 ：
```
shell> sudo rpm -Uvh platform-and-version-specific-package-name.rpm
```
例如，对于n基于EL6的系统的版本，命令是：
```
shell> sudo rpm -Uvh mysql57-community-release-el6-n.noarch.rpm
```
>**注意**  
>一旦您的系统上安装了发行包，yum update 命令（或dnf启用的系统的dnf升级）的任何系统级更新将自动升级系统上的MySQL软件包，并替换任何本机第三方软件包，如果Yum在MySQL Yum存储库中找到替换它们。有关详细信息，请参阅使用MySQL Yum存储库升级MySQL并 替换MySQL 的本机第三方分发。

## 2. 选择版本系列

使用MySQL Yum存储库时，默认情况下选择最新的MySQL版本的MySQL进行安装。如果这是你想要的，你可以跳到下一步， 使用Yum安装MySQL。

在MySQL Yum存储库（http://repo.mysql.com/yum/）中，MySQL社区服务器的不同版本系列托管在不同的子链接库中。默认情况下，最新的GA系列（目前为MySQL 5.7）的子功能启用，默认情况下禁用所有其他系列（例如，MySQL 5.6系列）的子修复。使用此命令查看MySQL Yum存储库中的所有子修补程序，并查看其中哪些启用或禁用（对于启用dnf的系统，请使用dnf在命令中 替换 yum）：
```
shell> yum repolist all | grep mysql
```
要安装最新的GA系列的最新版本，不需要配置。要安装最新的GA系列以外的特定系列的最新版本，请禁用最新GA系列的子功能，并在运行安装命令之前启用特定系列的子功能。如果您的平台支持 yum-config-manager或dnf config-manager命令，您可以通过发出以下命令来执行此操作，这些命令禁用5.7系列的子链接，并启用5.6系列的子目录; 对于不启用dnf的平台：
```
shell> sudo yum-config-manager --disable mysql57-community
shell> sudo yum-config-manager --enable mysql56-community
```
对于启用dnf的平台：
```
shell> sudo dnf config-manager --disable mysql57-community
shell> sudo dnf config-manager --enable mysql56-community
```
除了使用yum-config-manager或 dnf config-manager命令外，还可以通过手动编辑/etc/yum.repos.d/mysql-community.repo 文件来选择一个系列 。这是一个典型的条目：文件中的subrepository发行系列：
```
[mysql57-community]
name=MySQL 5.7 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/6/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```
找到要配置的子链接的条目，然后编辑该enabled选项。指定 enabled=0禁用子广告素材，或 enabled=1启用子广告素材。例如，要安装MySQL 5.6，请确保您具有 enabled=0以上用于MySQL 5.7的子功能表项，并具有enabled=15.6系列的条目：
```
# Enable to use MySQL 5.6
[mysql56-community]
name=MySQL 5.6 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.6-community/el/6/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```
您只能在任何时间启用一个版本系列的子链接。当启用了多个版本系时，Yum将使用最新的系列。

通过运行以下命令并检查其输出（对于启用dnf的系统，使用dnf在命令中替换yum）， 验证是否已启用和禁用正确的子修补剂 ：
```
shell> yum repolist enabled | grep mysql
```

## 3. 安装MySQL

通过以下命令安装MySQL（对于启用dnf的系统，用命令 dnf替换yum）：
```
shell> sudo yum install mysql-community-server
```
这将安装MySQL服务器的包以及其他所需的包。

## 4. 启动MySQL服务器

使用以下命令启动MySQL服务器：
```
shell> sudo service mysqld start
```
对于基于EL7的平台，这是首选命令：
```
shell> sudo systemctl start mysqld.service
```
您可以使用以下命令检查MySQL服务器的状态：
```
shell> sudo service mysqld status
```
对于基于EL7的平台，这是首选命令：
```
shell> sudo systemctl status mysqld.service
```

MySQL服务器初始化（仅适用于MySQL 5.7）：在服务器初始启动时，如果服务器的数据目录为空，则会发生以下情况：
- 服务器已初始化。

- SSL证书和密钥文件在数据目录中生成。

- 该 validate_password插件安装并启用。

- 'root'@'localhost' 创建 超级用户帐户。超级用户的密码被设置并存储在错误日志文件中。要显示它，请使用以下命令：
```
shell> sudo grep 'temporary password' /var/log/mysqld.log
```
通过使用生成的临时密码登录，尽快更改root密码，并为超级用户帐户设置自定义密码：
```
shell> mysql -uroot -p
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';
```
>**注意**  
>MySQL的 validate_password 插件默认安装。这将要求密码至少包含一个大写字母，一个小写字母，一位数字和一个特殊字符，并且总密码长度至少为8个字符。

---

## 上面都是翻译的
## 下面才是操作的记录

因为想使用mysql的全文索引，需要5.7才支持，使用原来的数据文件进行升级，原来的是5.6.31

---

```
shell> uname -r
3.10.0-514.26.2.el7.x86_64


shell> wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm


shell>  rpm -Uvh mysql57-community-release-el7-11.noarch.rpm 
警告：mysql57-community-release-el7-11.noarch.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:mysql57-community-release-el7-11 ################################# [100%]


shell> yum list | grep mysql


shell> yum install mysql-community-server


shell> systemctl status mysqld
# 安装完成并没有启动


# 修改配置文件
shell> vi /etc/my.cnf

  datadir=/data/mysql/datadir
  

#将之前的数据文件复制到新安装的机器上
shell> scp -r xxxx/xxx/ root@1.1.1.1:/data/xxx

#修改权限
shell> chown -R mysql:mysql mysql

#启动服务
shell> systemctl start mysqld

#查看服务
shell> systemctl status mysqld


#升级
shell> mysql_upgrade -u root -p
Enter password: 
Checking if update is needed.
Checking server version.
Running queries to upgrade MySQL server.
Checking system database.
mysql.columns_priv                                 OK
mysql.db                                           OK
......
hinge.user                                         OK
hinge.user_profile                                 OK
sys.sys_config                                     OK
Upgrade process completed successfully.
Checking if update is needed.

#重启
shell> systemctl restart mysqld

```

