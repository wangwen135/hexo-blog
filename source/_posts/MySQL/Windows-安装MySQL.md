---
title: Windows-安装MySQL
date: 2021-12-09 23:41
tags: 
  - MySQL
categories:
  - [MySQL]
---

*2017年11月*

---

> 确保以具有管理员权限的用户身份登录。

## 解压安装 Mysql 5.6
#### 1. 下载Windows版本的压缩包  
如：mysql-5.6.31-winx64.zip 

#### 2. 将文件解压缩到指定目录  
如：**C:\Program Files\MySQL\MySQL Server 5.6**  

#### 3. 设置环境变量
将.../bin 添加到系统环境变量 path 中  

#### 4. 修改配置文件  
默认的配置文件是../my-default.ini，重命名该文件或者自己建立一个my.ini文件  
在其中修改或添加配置： 
```
[mysqld] 
basedir=C:/Program Files/MySQL/MySQL Server 5.6 
datadir=C:/Program Files/MySQL/MySQL Server 5.6/data 
```

#### 5. 安装windows服务   
进入到bin目录中，输入 `mysqld -install` 安装服务  
>如果不进入bin目录，install能成功，服务能注册，但服务无法启动，提示找不到文件  

install指令默认会按照如下顺序读取指定配置文件:  
>C:\Windows\my.ini  
>C:\Windows\my.cnf  
>C:\my.ini  
>C:\my.cnf  
>C:\Program Files\MySQL\MySQL Server 5.6\my.ini  
>C:\Program Files\MySQL\MySQL Server 5.6\my.cnf  
```
C:\Users\Administrator>mysqld -install
Service successfully installed.

basedir = C:\Program Files\MySQL\mysql-5.6.31-winx64
datadir = C:\Program Files\MySQL\mysql-5.6.31-winx64\data
```

#### 6. 初始数据目录
进入到bin目录中，输入 `mysqld  --initialize` 初始化数据目录
> （好像是可以不需要）  
```
C:\Users\Administrator>mysqld --initialize
2016-11-16 16:54:03 0 [Warning] TIMESTAMP with implicit DEFAULT value is depreca
ted. Please use --explicit_defaults_for_timestamp server option (see documentati
on for more details).
2016-11-16 16:54:03 0 [Note] mysqld (mysqld 5.6.31) starting as process 2564 ...

```

#### 7. 启动服务  
```
net start mysql

# 停止服务用
net stop mysql
```

#### 8. 修改密码
```
# 指定root用户登录否则没有权限

mysql -u root     

# 切换到mysql数据库
use mysql; 

# 默认是没有密码的
mysql> select Host,User,Password from user;
+-----------+------+----------+
| Host      | User | Password |
+-----------+------+----------+
| localhost | root |          |
| 127.0.0.1 | root |          |
| ::1       | root |          |
| localhost |      |          |
+-----------+------+----------+
4 rows in set (0.00 sec)

# 修改密码
mysql> update mysql.user set password=password('root') where user='root';
Query OK, 3 rows affected (0.00 sec)
Rows matched: 3  Changed: 3  Warnings: 0

# 刷新权限
mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

# 再查看user表
mysql> select Host,User,Password from user;
+-----------+------+-------------------------------------------+
| Host      | User | Password                                  |
+-----------+------+-------------------------------------------+
| localhost | root | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B |
| 127.0.0.1 | root | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B |
| ::1       | root | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B |
| localhost |      |                                           |
+-----------+------+-------------------------------------------+
4 rows in set (0.00 sec)

## 可以用客户端工具进行测试

# 远程访问，Host为%
mysql> grant all privileges on *.* to 'root'@'%' identified by 'root' with grant
 option;
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

# 再看多了一条记录
mysql> select Host,User,Password from user;
+-----------+------+-------------------------------------------+
| Host      | User | Password                                  |
+-----------+------+-------------------------------------------+
| localhost | root | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B |
| 127.0.0.1 | root | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B |
| ::1       | root | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B |
| localhost |      |                                           |
| %         | root | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B |
+-----------+------+-------------------------------------------+
5 rows in set (0.00 sec)


```

----
----


## 解压安装 MySQL 5.7

#### 1. 下载Mysql 5.7的ZIP包

地址：  
https://dev.mysql.com/downloads/file/?id=478884  
https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.23-winx64.zip


#### 2. 选择安装位置，解压缩  
如解压到：D:\mysql\mysql-5.7.23-winx64

> 如果想在其他目录直接执行mysql命令，则需要将bin加入到环境变量

#### 2. 创建my.ini文件
在安装目录（D:\mysql\mysql-5.7.23-winx64）创建my.ini文件，填入如下内容
>纯文本文件，可以使用任何文本编辑器进行编辑

```
[mysqld]
# set basedir to your installation path
basedir=D:/mysql/mysql-5.7.23-winx64
# set datadir to the location of your data directory
datadir=D:/mysql/mysql-5.7.23-winx64/data
```

如果使用反斜杆需要两个
```
[mysqld]
# set basedir to your installation path
basedir=D:\\mysql\\mysql-5.7.23-winx64
# set datadir to the location of your data directory
datadir=D:\\mysql\\mysql-5.7.23-winx64\\data
```

#### 3. 初始化数据目录

```
cd D:\mysql\mysql-5.7.23-winx64

D:\mysql\mysql-5.7.23-winx64>bin\mysqld --initialize --console

# 会生成临时密码
# [Note] A temporary password is generated for root@localhost: r!i*sp)-E9hA

```

#### 4. 启动Mysql
```
D:\mysql\mysql-5.7.23-winx64\bin>mysqld.exe --console
......
......
......
2018-08-25T19:36:15.598206Z 0 [Note] InnoDB: Buffer pool(s) load completed at 180826  3:36:15
2018-08-25T19:36:15.599873Z 0 [Note]   - '::' resolves to '::';
2018-08-25T19:36:15.604039Z 0 [Note] Server socket created on IP: '::'.
2018-08-25T19:36:15.693851Z 0 [Note] Event Scheduler: Loaded 0 events
2018-08-25T19:36:15.703020Z 0 [Note] mysqld.exe: ready for connections.
Version: '5.7.23'  socket: ''  port: 3306  MySQL Community Server (GPL)

```
这种方式启动直接 CTRL + C 停止即可

不加 --console 则输出信息在日志文件夹中

#### 5. 进入mysql，并修改密码

```
cd bin
mysql -uroot -p
# 输入上面的临时密码

#进入mysql，修改密码

ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';

# 退出重新登录
```

#### 6. 停止mysql

```
>mysqladmin.exe -uroot -p shutdown

#输入root密码
```

----
#### 7. 作为Windows服务
需要想将bin目录加入到环境变量
> 没有进行测试，通过命令行方式启动更加可控
```
# 安装服务
bin\mysqld --install

# 删除服务
bin\mysqld --remove

```


----
----

## 关于登录和密码

### Mysql 5.6  
```
# 修改密码  
set password for 'root'@'localhost' =password('root');  

# 设置远程登录  
grant all privileges on *.* to 'root'@'%' identified by 'root' with grant option;
flush privileges;  

# 创建新用户  
create user 'hinge'@'%' identified by 'hinge';  
grant all privileges on *.* to hinge@'%' identified by 'hinge';  
flush privileges;  

# 不指定时默认是%*  
create user hinge identified by 'hinge';  
grant all privileges on *.* to hinge identified by 'hinge';  
flush privileges;  


grant all privileges on *.* to hinge@'localhost' identified by 'hinge';  
flush privileges;  

# 刷新权限   
flush privileges;  
```
---

### Mysql 5.7

在配置文件[mysqld]字段下增加skip-grant-tables 字段，用以忽略权限验证  

使用mysql -u root -p 登录数据， 密码直接回车  

修改[mysql]数据库，user表的authentication_string字段  

```
update mysql.user set authentication_string=password('root') where user='root'；  
flush privileges;  
```


---
---
---

## 2020-11-12
这个时候的mysql5.7  小版本已经到了32 了 

下载地址：  
https://dev.mysql.com/downloads/mysql/5.7.html


#### 缺少dll的问题
运行mysqld 时，系统提示“由于找不到MSVCR120.dll，无法继续执行代码。重新安装程序可能会解决此问题。”

官方说明：  
https://support.microsoft.com/en-us/help/3179560/update-for-visual-c-2013-and-visual-c-redistributable-package
下载这个程序并执行：  
http://download.microsoft.com/download/1/8/0/180FA2CE-506D-4032-AAD1-9D7636F85179/vcredist_x64.exe




