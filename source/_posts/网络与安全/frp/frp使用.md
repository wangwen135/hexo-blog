---
title: frp使用
date: 2024-07-13 14:29
tags: 
  - frp
  - 端口映射
categories:
  - [网络与安全, frp]
---


https://github.com/fatedier/frp


frp 采用 C/S 模式，将服务端部署在具有公网 IP 的机器上，客户端部署在内网或防火墙内的机器上，通过访问暴露在服务器上的端口，反向代理到处于内网的服务。 在此基础上，frp 支持 TCP, UDP, HTTP, HTTPS 等多种协议，提供了加密、压缩，身份认证，代理限速，负载均衡等众多能力。此外，还可以通过 xtcp 实现 P2P 通信。


https://gofrp.org/zh-cn/#td-block-1



### 安装与配置

> 在centos 7 上安装和使用


#### 下载最新版本到指定目录：

```
wget https://github.com/fatedier/frp/releases/download/v0.59.0/frp_0.59.0_linux_amd64.tar.gz
```


#### 创建 frps.service 文件

```
$ sudo vim /etc/systemd/system/frps.service
```
内容如下：
```
[Unit]
# 服务名称，可自定义
Description = frp server
After = network.target syslog.target
Wants = network.target

[Service]
Type = simple
# 启动frps的命令，需修改为您的frps的安装路径
ExecStart = /opt/frp/current/frps -c /opt/frp/current/frps.toml

[Install]
WantedBy = multi-user.target
```

#### 使用 systemd 命令管理 frps 服务

```
# 启动frp
sudo systemctl start frps
# 停止frp
sudo systemctl stop frps
# 重启frp
sudo systemctl restart frps
# 查看frp状态
sudo systemctl status frps
```

#### 设置 frps 开机自启动
```
sudo systemctl enable frps
```



#### 修改配置文件
```
vim /opt/frp/current/frps.toml 
```

内容如下：
```
bindPort = 6800

auth.token = "abc123"


######################
#日志配置
######################
log.to = "/opt/frp/logs/frps.log"
log.level = "info"
log.maxDays = 7
log.disablePrintColor = false


###############################
### 服务端 Dashboard
###############################
# 默认为 127.0.0.1，如果需要公网访问，需要修改为 0.0.0.0。
webServer.addr = "0.0.0.0"
webServer.port = 7500
# dashboard 用户名密码，可选，默认为空
webServer.user = "admin"
webServer.password = "admin"

```

> 修改完之后重启


#### 查看web 管理界面

http://192.168.1.8:7500/



-----

### 防火墙和端口映射

#### 在路由器上进行端口映射

16800端口映射：

```
192.168.1.8
6800
tcp+udp

frp主端口
```

16801端口映射：
```
192.168.1.8
16801
tcp+udp

frp 映射端口1
```
> 这里先只开放一个端口


#### 防火墙放开端口

放开端口6800
```
firewall-cmd --permanent --zone=public --add-port=6800/tcp

firewall-cmd --reload
firewall-cmd --zone=public --list-ports
```
放开端口16801
```
firewall-cmd --permanent --zone=public --add-port=16801/tcp
firewall-cmd --reload
firewall-cmd --zone=public --list-ports
```


### 客户端测试

#### 下载
https://github.com/fatedier/frp/releases

下载windows版本的程序

下载了一个v0.59.0版本的，被报检测到病毒

后面又下载了一个v0.56.0版本的


#### 配置

配置文件：

```
serverAddr = "[你的域名或公网ip]"
serverPort = 16800

auth.token = "abc123"


[[proxies]]
name = "test-tcp"
type = "tcp"
localIP = "127.0.0.1"
localPort = 80
remotePort = 16801

```

#### 启动

启动客户端程序

进入目录执行：
```
>frpc -c frpc.toml
```

这里代理了一个80端口，在本地随便启动一个80端口的程序，这里用HFS，然后访问：  
http://[你的域名或公网ip]:16801
