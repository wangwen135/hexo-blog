---
title: ssh 本地端口转发
date: 2024-08-05 22:15
tags: 
  - SSH
  - 端口
  - 隧道
categories:
  - [网络与安全, SSH]
---

SSH本地端口转发（Local Port Forwarding）是一种通过SSH隧道将本地计算机的某个端口转发到远程服务器的某个端口的技术。这样，访问本地端口的请求会通过SSH隧道传输到远程服务器，从而实现安全的通信。

### 使用场景
#### 访问内网服务：
通过SSH隧道访问公司内网或远程服务器上的服务，避免直接暴露服务端口。

#### 安全传输数据：
将本地端口转发到远程服务器上的敏感服务，实现加密传输。

#### 绕过防火墙：
通过SSH隧道绕过防火墙限制访问某些服务。


### 配置示例

假设你有一台远程服务器remote_server_ip，上面运行了一个Web服务，监听在端口80。你希望通过SSH隧道，将本地计算机的端口8080转发到远程服务器的端口80，以便在本地浏览器中访问localhost:8080时能够访问远程服务器的Web服务。

```
ssh -L 8080:localhost:80 user@remote_server_ip

```

- -L 8080:localhost:80：指定本地端口8080转发到远程服务器的localhost:80。
- user@remote_server_ip：SSH登录的用户名和远程服务器的IP地址。


又比如假设你想访问远程服务器上的MySQL服务，该服务只监听**localhost**。
```
ssh -L 3306:localhost:3306 user@remote-ssh-server
```
这会将本地计算机的3306端口上的所有连接通过remote-ssh-server，转发到其localhost的3306端口。

#### 步骤如：

1. 打开终端：在本地计算机上打开一个终端。
2. 运行SSH命令：执行上述SSH命令创建隧道。
3. 访问本地端口：在本地浏览器中访问http://localhost:8080，这将通过SSH隧道转发到远程服务器的端口80。


----

### 更多
#### 后台运行隧道：
使用-f选项将隧道命令放入后台运行。
```
ssh -L 8080:localhost:80 -f user@remote_server_ip -N

```

- -N：表示不执行远程命令，仅进行端口转发。
- -f：表示后台运行SSH隧道。


#### 转发到不同的远程主机：
如果需要转发到远程服务器能访问到的另一台主机（ssh服务器作为跳板），可以使用远程主机的IP或域名。
```
ssh -L 8080:target_host:80 user@remote_server_ip
```

- target_host：远程服务器可以访问的目标主机的IP或域名。

