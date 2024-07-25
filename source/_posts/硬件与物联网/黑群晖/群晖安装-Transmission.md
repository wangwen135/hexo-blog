---
title: 群晖安装-Transmission
date: 2021-05-17 23:41
tags: 
  - 黑群晖
categories:
  - [硬件与物联网, 黑群晖]
---



Transmission是一种BitTorrent客户端，特点是一个跨平台的后端和其上的简洁的用户界面。Transmission以MIT许可证和GNU通用公共许可证双许可证授权，因此是一款自由软件。支持包括Linux、Mac OS X等多种操作系统（也有爱好者制作的windows安装包），以及Networked Media Tank、WD MyBook、ReadyNAS、D-Link DNS-323 & CH3SNAS、Synology等多种设备。支持GTK+、命令行、Web等多种界面。

其特点是开源、无广告，硬件资源消耗极少，界面极度精简，支持BT种子和磁力链接下载，支持web界面、远程控制等。



#### 安装

1. 设置默认下载文件夹
2. 添加第三方套件源
    之前已经加过了：http://packages.synocommunity.com/
3. 在“套件中心”-“社群”，就可以找到Transmission，点击安装


#### 配置：
1. 配置下载目录
2. 设置登录的用户名和密码  


默认的访问地址是：http://xxxxxx.xxx:9091/  
需要在路由器上做端口映射
> 想要通过域名访问需要自己配置DDNS


#### 配置Transmission Web Control
https://github.com/ronggang/transmission-web-control

> 汉化与加强Transmission Web的操作能力




ssh 到群晖上

切换到root
```
sudo -i
```

下载脚本并执行
```
wget https://github.com/ronggang/transmission-web-control/raw/master/release/install-tr-control.sh

# 中文版本
wget https://github.com/ronggang/transmission-web-control/raw/master/release/install-tr-control-cn.sh


sh install-tr-control.sh
```

选择1安装最小版本，直到安装完成


#### 权限设置：
控制面板 --> 用户群组 --> sc-download 
> 设置download目录权限为可读写

控制面板 --> 共享文件夹 --> 编辑download文件夹
> 设置权限 --> 系统内部账号 --> sc-transmission --> 可读写

**为了能直接下载到video目录**

控制面板 --> 用户群组 --> sc-download 
> 设置video目录权限为可读写

控制面板 --> 共享文件夹 --> 编辑video文件夹
> 设置权限 --> 系统内部账号 --> sc-transmission --> 可读写

**注意！**
*有些教程上是设置everyone可读写，这样是不安全的*
----
#### 外网访问
路由器做端口映射，默认端口：9091


----

### 使用https
进入群晖 --> 控制面板  --> 引用程序门户 --> 反向代理服务  
新增一个：  
**来源**
> 协议：https  
> 主机名：*  
> 端口：19091  

> 启用HSTS  
> 启用HTTP/2

**目的地**  
> 协议：http
> 主机名：192.168.1.66
> 端口：9091


修改路由器上的端口应用：外网端口改成 19091，内网指向群晖的19091端口

访问：https://xxxxx.xxx:19091/transmission/web/



-----
### 端口映射
进入 Transmission Web Control  
进入设置--> 网络传输 --> 默认使用固定端口：51413

点击下方的测试端口，显示不可连接

在路由器上配置端口映射，将外网的51413端口映射到群晖（transmission）的51413端口

再次点击测试，显示端口可连接




