---
title: CentOS-7-安装-Tor
date: 2022-10-16 23:41
tags: 
  - 代理
categories:
  - [网络与安全, 代理]
---


```
yum install tor
```

如果yum中没有tor，请安装epel源：

```
yum install epel-release

#安装完成之后可以通过下面的命令查看
yum repolist


#可以看到多了一个
#Extra Packages for Enterprise Linux 7 - x86_64
```

修改配置文件：/etc/tor/torrc 指定服务绑定IP和端口：
```
SOCKSPort 0.0.0.0:9150
```

启动服务：
```
Systemctl start tor
```

检查上方配置文件中指定的端口是否正在监听：
```
netstat -an | grep 9150
```

检查防火墙设置，确保服务器能正常连接到VPS上的Tor服务端口。


