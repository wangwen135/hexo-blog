---
title: 群晖使用iperf3测试网速
date: 2021-09-24
tags: 
  - 黑群晖
  - iperf3
categories:
  - [硬件与物联网, 黑群晖]
---



黑群晖安装后复制文件速度没有达到预期值

> 千兆网卡：113MB/s   
> 2.5G网卡：280MB/s



### 一、服务端配置
群晖下安装Docker套件

在docker中安装iperf3
> 注册表中搜索，安装星星最多的就行了

镜像下载好了之后点启动

然后选择高级设置：
- 端口设置：将本地端口配置为5201
- 环境 -> 执行命令，填写命令-s（设置为服务端）




### 二、客户端配置
在iperf3官网下载宽带测速软件

https://iperf.fr/iperf-download.php


在命令行中：
```
.\iperf3.exe -c 192.168.1.66

.\iperf3.exe -c 192.168.1.66 -t 300

.\iperf3.exe -c 192.168.1.66 -R -t 300

# -R 以反向模式运行(服务器发送，客户端接收)  
```



