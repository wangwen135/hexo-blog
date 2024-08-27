---
title: sshuttle
date: 2024-08-27 23:58
tags: 
  - SSH
  - VPN
  - sshuttle
  - 隧道
categories:
  - [网络与安全, SSH]
---

### sshuttle 一个使用ssh协议的透明代理VPN

sshuttle 通过 SSH 隧道将本地网络流量转发到远程主机，类似于一个轻量级的 VPN。  
sshuttle 的工作原理是通过 SSH 将流量转发到远程主机，然后远程主机再将流量发出。因此，它非常适合在只有 SSH 访问权限的情况下进行简单的网络隧道代理。


### 安装 sshuttle

在大多数 Linux 发行版上，可以通过包管理器安装 sshuttle：

#### Debian/Ubuntu:
```
sudo apt-get update
sudo apt-get install sshuttle
```

#### CentOS/RHEL:
```
sudo yum install epel-release
sudo yum install sshuttle
```

### 使用 sshuttle

sshuttle 的基本使用方法如下：

```
sshuttle [options...] [-r [username@]sshserver[:port]] \<subnets...>

sshuttle -r <user>@<remote_host> <network>
```

- -r <user>@<remote_host>：指定 SSH 连接的远程主机及其用户名
- <network>：指定你想要通过 SSH 隧道转发的目标网络或 IP 地址

其他选项：
- --dns：通过 SSH 隧道转发 DNS 请求
- -v 或 --verbose：启用详细输出，用于调试


#### 示例用法

##### 将所有网络流量转发到远程主机：

假设你的远程主机 IP 是 192.168.1.100，用户名是 user，并且你希望将本地的所有网络流量通过这个远程主机进行转发：

```
sshuttle -r user@192.168.1.100 0.0.0.0/0
```
这条命令会将所有网络流量通过 SSH 隧道转发到 192.168.1.100。

##### 仅转发特定网络的流量：

如果你只想通过 SSH 隧道转发特定子网的流量，比如 192.168.10.0/24：

```
sshuttle -r user@192.168.1.100 192.168.10.0/24
```
这条命令会将所有指向 192.168.10.0/24 网络的流量通过 SSH 隧道转发到远程主机 192.168.1.100。

多个子网
```
sshuttle -r user@192.168.1.100 192.168.10.0/24 10.10.10.0/24
```


##### 后台运行：
如果你希望 sshuttle 在后台运行，可以加上 -D 参数：

```
sshuttle -r user@192.168.1.100 0.0.0.0/0 -D
```
使用 -D 选项后，sshuttle 会在后台运行并持续工作。

### 注意事项

- 权限：sshuttle 通常需要 root 权限或使用 sudo 执行，因为它需要修改本地的网络路由表和防火墙规则。
- Python：sshuttle 是用 Python 编写的，因此需要安装 Python 环境。
- SSH 访问：使用 sshuttle 需要目标远程主机上有 SSH 访问权限，并且 SSH 连接成功后才能转发流量。
- 端口转发：sshuttle 默认会转发 TCP 流量，但也支持 UDP 流量，可以使用 -x 参数排除特定的地址或使用 -e 选项指定自定义的 SSH 选项。



----


### 检测和验证

#### 查看当前的iptables NAT规则
```
sudo iptables -t nat -L -n -v
```
应该能看到：Chain sshuttle 如：
```
Chain sshuttle-12300 (2 references)
 pkts bytes target     prot opt in     out     source               destination         
   38  1976 REDIRECT   tcp  --  *      *       0.0.0.0/0            10.10.0.0/16         TTL match TTL != 42 redir ports 12300
    1    52 RETURN     tcp  --  *      *       0.0.0.0/0            127.0.0.0/8
```

> firewalld 是 CentOS 7 上的默认防火墙管理工具，它为用户提供了一个更高级的管理界面来配置防火墙规则。
iptables 是 Linux 内核中用于设置、维护和检查 IP 包过滤规则的工具。firewalld 在后台实际上还是使用 iptables 来应用防火墙规则的。

#### 增加调试日志
```
sshuttle -vvr username@sshserver 0.0.0.0/0
```
这里的 -vv 参数会增加日志的详细程度，有助于调试问题


