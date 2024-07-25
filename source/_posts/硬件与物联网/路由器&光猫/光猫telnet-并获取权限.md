---
title: 光猫telnet-并获取权限
date: 2024-04-25 23:41
tags: 
  - 路由器&光猫
categories:
  - [硬件与物联网, 路由器&光猫]
---


### 临时启用telnet和ftp
http://192.168.1.1:8080/bd/modify_hide.asp 


### telnet访问
> telnet 端口号是23

#### telent用户名密码
```
telnet 192.168.1.1

#用户名
admin

#密码
TeleCom_1234


# 再切换到超级用户

su telecomadmin

#密码
TeleCom_xxxxxx

```

> 密码：（后面6位是设备标识的后六位，光猫背面有）  
> 不能直接用telecomadmin


### ftp访问
> ftp控制端口一般为21，而数据端口不一定是20，这和FTP的应用模式有关，如果是主动模式，应该为20，如果为被动模式，由服务器端和客户端协商而定

#### 防火墙策略
开放/关闭 FTP
```
#添加规则
iptables -I ftp_account 1 -i br0 -p tcp --dport 21 -j ACCEPT 

#查看
iptables -L ftp_account -n -v

#删除规则
iptables -D ftp_account 1 
```


#### ftp用户名密码
忘记了。。。。



### SSH

开放关闭SSH
```
//开
iptables -I inacc 1 -i br0 -p tcp --dport 22 -j ACCEPT
//查看
iptables -L inacc -n -v --line-numbers
//关
iptables -D inacc 1 


# SSH用户名密码就是电信的超级密码
# SSH 算法协议较老 需要增加额外的弱密码算法 ssh

telecomadmin@192.168.1.1 -oKexAlgorithms=+diffie-hellman-group1-sha1 -oCiphers=+3des-cbc

```
