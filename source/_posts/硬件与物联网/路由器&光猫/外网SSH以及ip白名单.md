---
title: 外网SSH以及ip白名单
date: 2022-07-06 23:41
tags: 
  - 路由器
  - firewall
categories:
  - [硬件与物联网, 路由器&光猫]
---

### 外网访问SSH
开22端口不安全，这里在防火墙上做一个转发，端口：61622

vi /etc/config/firewall
```
config redirect             
    option name 'Allow-Wan-SSH-61622'
    option src 'wan'
    option src_dport '61622'
    option dest 'lan'
    option dest_port '22'
    option proto 'tcp'           
    option target 'DNAT'
```


### IP白名单设置
> 如公司的出口IP为：123.240.202.234

#### 临时的
```
iptables -I INPUT -s 123.240.202.234 -j ACCEPT
```
重启之后就没有了

#### 永久的  
编辑 vi /etc/config/firewall  
加上如：
```
config rule
        option name 'Company Ip White List'
        option src 'wan'
        option src_ip '123.240.202.234'
        option family 'ipv4'
        option target 'ACCEPT'
```

重启：/etc/init.d/firewall restart

### 直接开放wan上的22口
修改防火墙文件
```
vi /etc/config/firewall 
```
在末尾加上
```
config rule
    option name 'Allow-Wan-SSH-22'
    option src  'wan'
    option dest_port  '22'
    option target  'ACCEPT'
    option proto  'tcp'
```
重启防火墙，并测试
```
service firewall restart
# 或
/etc/init.d/firewall restart
```
**密码要搞复杂点**



----
----


## 其他防火墙配置的例子
https://openwrt.org/zh-cn/doc/uci/firewall

### 开放端口
```
config rule
        option src              wan
        option dest_port        22
        option target           ACCEPT
        option proto            tcp

```

使Internet上的计算机可以使用SSH访问路由器


### 端口转发(NAT/DNAT)
```
config redirect
        option src       wan
        option src_dport 80
        option proto     tcp
        option dest_ip   192.168.1.10
```

### 源NAT (SNAT)
```
config redirect
        option src              lan
        option dest             wan
        option src_ip           10.55.34.85
        option src_dip          63.240.161.99
        option dest_port        123
        option target           SNAT
```
源NAT更改发往系统的传出数据包，以使系统看起来像是该数据包的源。  
为定向到端口123的，从IP地址为10.55.34.85的主机发起的UDP和TCP流量定义源NAT。源地址被重写为63.240.161.99。  

当单独使用时，源NAT用来限制一台计算机访问互联网，但允许它访问一些服务，我手动转发一些似乎是本地服务，如NTP到互联网。

### 实际端口转发
```
config redirect
        option src              wan
        option src_dport        80
        option dest             lan
        option dest_port        80
        option proto            tcp
```

### 限制指定机器
```
config rule
        option src              lan
        option dest             wan
        option dest_ip          123.45.67.89
        option target           REJECT
```
阻止所有到指定主机地址的连接尝试。


### 通过MAC限制访问互联网
```
config rule
        option src              lan
        option dest             wan
        option src_mac          00:00:00:00:00
        option target           REJECT
```
则阻止从客户端到Internet的所有连接尝试。


### 转发规则限制
```
config rule
        option src              lan
        option dest             wan
        option dest_port        1000-1100
        option proto            'tcp udp'
        option target           REJECT
```
创建一个转发规则，以拒绝端口1000-1100上从lan到wan的流量。


### 透明代理规则（同一主机）
```
config redirect
	option src              lan
	option proto            tcp
	option src_dport        80
	option dest_port        3128
```
通过侦听路由器本身端口3128上的代理服务器将所有来自lan的HTTP通信重定向到lan。


### 透明代理规则（外部）
```
config redirect
        option src              lan
        option proto            tcp
        option src_ip           !192.168.1.100
        option src_dport        80
        option dest_ip          192.168.1.100
        option dest_port        3128
```
侦听端口3128的192.168.1.100处的外部代理将来自lan的所有传出HTTP流量重定向。


### 简单DMZ规则
```
config redirect
	option src              wan
	option proto            all
	option dest_ip          192.168.1.2
```

所有协议的所有WAN端口重定向到内部主机192.168.1.2。
