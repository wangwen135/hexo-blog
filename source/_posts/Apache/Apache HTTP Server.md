---
title: Apache HTTP Server
date: 2017-02-05 15:14
tags: 
  - Apache HTTP Server 
categories:
  - [Apache]
---

## 关于
Apache HTTP Server是一个为包括UNIX和Windows在内的现代操作系统努力开发和维护的开源HTTP服务器项目。这个项目的目标是提供一个安全，高效和可扩展的服务器，提供与当前HTTP标准同步的HTTP服务。   
Apache HTTP服务器（“httpd”）于1995年推出，它自1996年4月以来一直是互联网上最受欢迎的Web服务器。它于2015年2月庆祝了其20岁生日。  
Apache HTTP Server是Apache Software Foundation的一个项目。

## 安装
> 这里是在CentOS7 下的安装

```
yum -install httpd

# 会自动的安装一些依赖

```

## 配置

简单的说明一下，主要是修改：  
**/etc/httpd/conf/httpd.conf**
```
ServerRoot "/etc/httpd"  #apache软件安装的位置。其它指定的目录如果没有指定绝对路径，则目录是相对于该目录

Listen 80 #服务器监听的端口号

DocumentRoot "/var/www/html" #主站点的网页存储位置

<Directory "/data/statsvn/report"> #权限，用的时候再去查

```

## 启动
```
systemctl start httpd

#检查
ps -ef | grep httpd  

netstat -an | grep 80
```

配置开机启动
```
systemctl enable httpd

Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
```
