---
title: ssh 端口转发
date: 2024-08-05 22:15
updated: 2024-8-21 23:11
tags: 
  - SSH
  - 端口转发
  - 隧道
categories:
  - [网络与安全, SSH]
---


SSH 端口转发可以灵活的解决网络访问受限的问题

通常分为两种方式：
- **本地端口转发** 
- **远程（反向）端口转发**   


-----

## 本地端口转发

SSH本地端口转发（Local Port Forwarding）是一种通过SSH隧道将本地计算机的某个端口转发到远程服务器的某个端口的技术。这样，访问本地端口的请求会通过SSH隧道传输到远程服务器，从而实现安全的通信。

### 使用场景
- **访问内网服务**：通过SSH隧道访问公司内网或远程服务器上的服务，避免直接暴露服务端口。
- **安全传输数据**：将本地端口转发到远程服务器上的敏感服务，实现加密传输。
- **绕过防火墙**：通过SSH隧道绕过防火墙限制访问某些服务。


### 基本命令格式
```
ssh -L [本地端口]:[目标主机]:[目标端口] [用户名]@[SSH服务器]
```

### 配置示例

#### 示例 1
假设你有一台远程服务器remote_server_ip，上面运行了一个Web服务，监听在端口80。你希望通过SSH隧道，将本地计算机的端口8080转发到远程服务器的端口80，以便在本地浏览器中访问localhost:8080时能够访问远程服务器的Web服务。

```
ssh -L 8080:localhost:80 user@remote_server_ip
```
- -L 8080:localhost:80：指定本地端口8080转发到远程服务器的localhost:80。
- user@remote_server_ip：SSH登录的用户名和远程服务器的IP地址。

##### 访问方法
在执行上述命令后，在本地浏览器中访问http://localhost:8080，这将通过SSH隧道转发到远程服务器的端口80。


#### 示例 2
又比如假设你想访问远程服务器上的MySQL服务，该服务只监听**localhost**。
```
ssh -L 3306:localhost:3306 user@remote-ssh-server
```
这会将本地计算机的3306端口上的所有连接通过remote-ssh-server，转发到其localhost的3306端口。

##### 访问方法
使用Mysql客户端工具连127.0.0.1:3306


#### 转发到不同的远程主机
如果需要转发到远程服务器能访问到的另一台主机（ssh服务器作为跳板），可以使用远程主机的IP或域名。
```
ssh -L 8080:target_host:80 user@remote_server_ip
```

- target_host：远程服务器可以访问的目标主机的IP或域名。


----


## 远程端口转发

远程端口转发（Remote Port Forwarding）是指在远程计算机上创建一个端口，并将其流量通过 SSH 隧道转发到本地计算机的指定目标。通常用于让外部网络访问本地服务。

### 使用场景
- **让外部访问内部服务**：适合在远程服务器不能直接访问本地服务时，通过中间服务器进行访问。
- **绕过防火墙限制**：利用远程端口转发，通过中间服务器绕过防火墙访问内部服务。
- **安全访问**：让远程用户安全地访问公司内部网络中的资源。
- **反向 SSH 隧道**：将本地机器的端口暴露到远程服务器，方便远程访问本地服务。


### 基本命令格式
```
ssh -R [远程端口]:[目标主机]:[目标端口] [用户名]@[SSH服务器]

```

### 配置示例
假设你想让远程服务器（remote.example.com）通过其 8080 端口访问你本地机器上的一个运行在 localhost:3000 的服务，可以使用以下命令：
```
ssh -R 8080:localhost:3000 user@remote.example.com
```
- -R：用于指定反向隧道的选项。
- 8080：远程服务器上的端口号。
- localhost:3000：本地机器上运行服务的地址和端口号。
- user@remote.example.com：远程服务器的 SSH 用户和地址。

#### 访问方法
在执行上述命令后，你可以在远程服务器上访问 http://localhost:8080 来访问本地机器上 localhost:3000 端口运行的服务。



----


### 更多
#### 后台运行SSH隧道：
使用-f选项将隧道命令放入后台运行。
```
ssh -L 8080:localhost:80 -f user@remote_server_ip -N

```

- -N：表示不执行远程命令，仅进行端口转发。
- -f：表示后台运行SSH隧道。



#### 保持隧道活跃：
如果需要保持隧道长时间活跃，可以使用 autossh 来自动重连，或者设置 SSH 的 ServerAliveInterval 和 ServerAliveCountMax 参数：
```
ssh -R 8080:localhost:3000 user@remote.example.com -o ServerAliveInterval=60 -o ServerAliveCountMax=3
```
