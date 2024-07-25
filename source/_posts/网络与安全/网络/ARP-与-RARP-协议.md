---
title: ARP-与-RARP-协议
date: 2022-08-03 23:41
tags: 
  - 网络
categories:
  - [网络与安全, 网络]
---



## ARP协议
地址解析协议，即ARP（Address Resolution Protocol），是根据IP地址获取物理地址的一个TCP/IP协议。  
主机发送信息时将包含目标IP地址的ARP请求广播到局域网络上的所有主机，并接收返回消息，以此确定目标的物理地址；收到返回消息后将该IP地址和物理地址存入本机ARP缓存中并保留一定时间，下次请求时直接查询ARP缓存以节约资源。


## RARP协议
反向地址转换协议（RARP）是局域网的物理机器从网关服务器的ARP表或者缓存上根据MAC地址请求IP地址的协议，其功能与地址解析协议相反。与ARP相比，RARP的工作流程也相反。首先是查询主机向网路送出一个RARP Request广播封包，向别的主机查询自己的IP地址。这时候网络上的RARP服务器就会将发送端的IP地址用RARP Reply封包回应给查询者，这样查询主机就获得自己的IP地址了。

## 为什么需要这个
数据包在物理链路上传输 以太网帧 需要目的地的物理地址（MAC）  
通过ARP协议来获取同一个网络内的机器的IP地址和Mac的对应关系，为上层协议提供支持，因为上层协议使用IP地址进行通信。

- 局域网内的IP地址（通过子网掩码计算）之间的通信，就可以理解为设备（MAC）对设备（MAC）之间的通信
- 跨局域网的通信就是设备(MAC)对 网关（MAC）之间的通信



## ARP 数据包
**Wireshark 抓包** 

ARP 广播请求 （由192.168.1.88 发起的广播请求，询问谁的IP是192.168.1.1）
```
4595	296.111634	Giga-Byt_18:04:4a	Broadcast	ARP	42	Who has 192.168.1.1? Tell 192.168.1.88
```
> 内容如下：
```
Frame 4595: 42 bytes on wire (336 bits), 42 bytes captured (336 bits) on interface 0
Ethernet II, Src: Giga-Byt_18:04:4a (e0:d5:5e:18:04:4a), Dst: Broadcast (ff:ff:ff:ff:ff:ff)
    Destination: Broadcast (ff:ff:ff:ff:ff:ff)
    Source: Giga-Byt_18:04:4a (e0:d5:5e:18:04:4a)
    Type: ARP (0x0806)
Address Resolution Protocol (request)
    Hardware type: Ethernet (1)
    Protocol type: IPv4 (0x0800)
    Hardware size: 6
    Protocol size: 4
    Opcode: request (1)
    Sender MAC address: Giga-Byt_18:04:4a (e0:d5:5e:18:04:4a)
    Sender IP address: 192.168.1.88
    Target MAC address: 00:00:00_00:00:00 (00:00:00:00:00:00)
    Target IP address: 192.168.1.1

```

ARP 应答 （192.168.1.1 直接应答 192.168.1.88，告知其Mac地址）
```
4596	296.111872	NewH3CTe_95:65:a9	Giga-Byt_18:04:4a	ARP	60	192.168.1.1 is at 04:40:a9:95:65:a9
```
> 内容如下：
```
Frame 4596: 60 bytes on wire (480 bits), 60 bytes captured (480 bits) on interface 0
Ethernet II, Src: NewH3CTe_95:65:a9 (04:40:a9:95:65:a9), Dst: Giga-Byt_18:04:4a (e0:d5:5e:18:04:4a)
    Destination: Giga-Byt_18:04:4a (e0:d5:5e:18:04:4a)
    Source: NewH3CTe_95:65:a9 (04:40:a9:95:65:a9)
    Type: ARP (0x0806)
    Padding: 000000000000000000000000000000000000
Address Resolution Protocol (reply)
    Hardware type: Ethernet (1)
    Protocol type: IPv4 (0x0800)
    Hardware size: 6
    Protocol size: 4
    Opcode: reply (2)
    Sender MAC address: NewH3CTe_95:65:a9 (04:40:a9:95:65:a9)
    Sender IP address: 192.168.1.1
    Target MAC address: Giga-Byt_18:04:4a (e0:d5:5e:18:04:4a)
    Target IP address: 192.168.1.88

```
其他收到该广播的机器判断【192.168.1.1】不是自己的IP，直接丢弃




## ARP 命令说明
#### windows
```
显示和修改地址解析协议(ARP)使用的“IP 到物理”地址转换表。

ARP -s inet_addr eth_addr [if_addr]
ARP -d inet_addr [if_addr]
ARP -a [inet_addr] [-N if_addr] [-v]

  -a            通过询问当前协议数据，显示当前 ARP 项。
                如果指定 inet_addr，则只显示指定计算机
                的 IP 地址和物理地址。如果不止一个网络
                接口使用 ARP，则显示每个 ARP 表的项。
  -g            与 -a 相同。
  -v            在详细模式下显示当前 ARP 项。所有无效项
                和环回接口上的项都将显示。
  inet_addr     指定 Internet 地址。
  -N if_addr    显示 if_addr 指定的网络接口的 ARP 项。
  -d            删除 inet_addr 指定的主机。inet_addr 可
                以是通配符 *，以删除所有主机。
  -s            添加主机并且将 Internet 地址 inet_addr
                与物理地址 eth_addr 相关联。物理地址是用
                连字符分隔的 6 个十六进制字节。该项是永久的。
  eth_addr      指定物理地址。
  if_addr       如果存在，此项指定地址转换表应修改的接口
                的 Internet 地址。如果不存在，则使用第一
                个适用的接口。
示例:
  > arp -s 157.55.85.212   00-aa-00-62-c6-09.... 添加静态项。
  > arp -a                                  .... 显示 ARP 表。
```

#### linux
```
# man arp
ARP(8)                     Linux Programmer’s Manual                    ARP(8)
NAME
       arp - manipulate the system ARP cache
SYNOPSIS
       arp [-evn] [-H type] [-i if] -a [hostname]
       arp [-v] [-i if] -d hostname [pub]
       arp [-v] [-H type] [-i if] -s hostname hw_addr [temp]
       arp [-v] [-H type] [-i if] -s hostname hw_addr [netmask nm] pub
       arp [-v] [-H type] [-i if] -Ds hostname ifa [netmask nm] pub
       arp [-vnD] [-H type] [-i if] -f [filename]
NOTE
       This program is obsolete. For replacement check ip neighbor.
DESCRIPTION
       Arp  manipulates the kernel’s ARP cache in various ways.  The primary options are clearing an address mapping entry and manually setting up one.  For debugging purposes, the arp pro-
       gram also allows a complete dump of the ARP cache.
......
......
```




## 测试
### ping不存在的局域网机器
```
# cmd
C:\Users\Administrator>ping 192.168.1.111

正在 Ping 192.168.1.111 具有 32 字节的数据:
来自 192.168.1.88 的回复: 无法访问目标主机。
来自 192.168.1.88 的回复: 无法访问目标主机。
来自 192.168.1.88 的回复: 无法访问目标主机。
来自 192.168.1.88 的回复: 无法访问目标主机。

192.168.1.111 的 Ping 统计信息:
    数据包: 已发送 = 4，已接收 = 4，丢失 = 0 (0% 丢失)，
```
#### wireshark 抓包
> 过滤条件：arp.dst.proto_ipv4==192.168.1.111 or icmp

```
2910	206.611909	Giga-Byt_18:04:4a	Broadcast	ARP	42	Who has 192.168.1.111? Tell 192.168.1.88
2917	207.570302	Giga-Byt_18:04:4a	Broadcast	ARP	42	Who has 192.168.1.111? Tell 192.168.1.88
2924	208.570334	Giga-Byt_18:04:4a	Broadcast	ARP	42	Who has 192.168.1.111? Tell 192.168.1.88
......
```
只有arp请求

#### 结论
可以看到只会发送ARP请求，且ARP请求没有获得响应，此时无法获取ping的目的地，故不会发送ping（ICMP）请求  
且错误显示为：**“无法访问目标主机”**



### 添加静态arp映射后再ping
先添加IP对MAC的映射关系
```
arp -s 192.168.1.111 00-2b-3c-f0-ff-ff
```
用 arp -a 查看结果
```
C:\Users\Administrator>arp -a

接口: 192.168.1.88 --- 0xb
  Internet 地址         物理地址              类型
  192.168.1.1           04-40-a9-95-65-a9     动态
  192.168.1.111          00-2b-3c-f0-ff-ff     静态
......
```

ping 测试 
```
C:\Users\Administrator>ping 192.168.1.111

正在 Ping 192.168.1.111 具有 32 字节的数据:
请求超时。
请求超时。
请求超时。
请求超时。

192.168.1.111 的 Ping 统计信息:
    数据包: 已发送 = 4，已接收 = 0，丢失 = 4 (100% 丢失)，
```

#### wireshark 抓包  
> 过滤条件：arp.dst.proto_ipv4==192.168.1.111 or icmp

```
275	19.179418	192.168.1.88	192.168.1.111	ICMP	74	Echo (ping) request  id=0x0001, seq=26/6656, ttl=64 (no response found!)
315	23.681918	192.168.1.88	192.168.1.111	ICMP	74	Echo (ping) request  id=0x0001, seq=27/6912, ttl=64 (no response found!)
......
```
只有ping

**数据报详情：**
```
Frame 315: 74 bytes on wire (592 bits), 74 bytes captured (592 bits) on interface 0
Ethernet II, Src: Giga-Byt_18:04:4a (e0:d5:5e:18:04:4a), Dst: 00:2b:3c:f0:ff:ff (00:2b:3c:f0:ff:ff)
    Destination: 00:2b:3c:f0:ff:ff (00:2b:3c:f0:ff:ff)
    Source: Giga-Byt_18:04:4a (e0:d5:5e:18:04:4a)
    Type: IPv4 (0x0800)
Internet Protocol Version 4, Src: 192.168.1.88, Dst: 192.168.1.111
Internet Control Message Protocol

```
#### 结论：
当本地的Arp缓存中有IP地址和Mac地址的对应关系时，会直接往目标地址发送数据报，从以太网帧中可以看到目的地Mac就是我们设置的Mac。
此时不会再发送ARP请求，直接发送了Ping请求，且错误显示为：**“请求超时”**



### 修改网关Mac，让本机无法上网
#### 找网关地址
```
>ipconfig

以太网适配器 本地连接:

   连接特定的 DNS 后缀 . . . . . . . :
   IPv4 地址 . . . . . . . . . . . . : 192.168.1.88
   子网掩码  . . . . . . . . . . . . : 255.255.255.0
   默认网关. . . . . . . . . . . . . : 192.168.1.1
```

#### 查网关真实的Mac地址
```
C:\Users\Administrator>arp -a | findstr 192.168.1.1
  192.168.1.1           04-40-a9-95-66-a9     动态
  192.168.1.111         00-2b-3c-f0-ff-ff     静态
```

#### 修改本地arp缓存中的网关Mac地址

这里修改 192.168.1.1 的Mac地址  
**arp 命令修改失败：拒绝访问**
```
arp -d 192.168.1.1
arp -s 192.168.1.1 00-2b-3c-f0-ff-f1 192.168.1.88
ARP 项添加失败: 拒绝访问。

```

改用netsh命令修改 
1. 先找本地网卡的 Idx，这里是：11
```
netsh i i show in

Idx     Met         MTU          状态                名称
---  ----------  ----------  ------------  ---------------------------
  1          50  4294967295  connected     Loopback Pseudo-Interface 1
 11          10        1500  connected     本地连接
 16          30        1500  connected     Npcap Loopback Adapter
```
2. 使用如下命令修改本地Mac地址缓存
```
netsh -c "i i" add neighbors 11 "网关IP" "Mac地址"  -- 这里11是idx号。

netsh -c "i i" add neighbors 11 "192.168.1.1" "00-2b-3c-f0-ff-f1"

```
3. 验证修改结果
```
arp -a

C:\Users\Administrator>arp -a

接口: 192.168.1.88 --- 0xb
  Internet 地址         物理地址              类型
  192.168.1.1           00-2b-3c-f0-ff-f1     静态
  192.168.1.171         d4-5d-df-01-32-f0     动态
......
......
```


#### 导致的结果是本机不能再上网了，但局域网访问正常

比如在浏览器中访问百度

使用wireshar 抓包
```
# 这里先进行DNS解析

6	0.783394	192.168.1.88	114.114.114.114	DNS	73	Standard query 0x341b A www.baidu.com

# 可以看到直接将数据包发送到了我们上面配置的假Mac地址： 00-2b-3c-f0-ff-f1 

Frame 6: 73 bytes on wire (584 bits), 73 bytes captured (584 bits) on interface 0
Ethernet II, Src: Giga-Byt_18:04:4a (e0:d5:5e:18:04:4a), Dst: 00:2b:3c:f0:ff:f1 (00:2b:3c:f0:ff:f1)
    Destination: 00:2b:3c:f0:ff:f1 (00:2b:3c:f0:ff:f1)
    Source: Giga-Byt_18:04:4a (e0:d5:5e:18:04:4a)
    Type: IPv4 (0x0800)
Internet Protocol Version 4, Src: 192.168.1.88, Dst: 114.114.114.114
User Datagram Protocol, Src Port: 64752, Dst Port: 53
Domain Name System (query)

```


#### 还原网关对应的mac地址

使用netsh 命令设置之后，再用 ```arp -d 192.168.1.1```命令删除，在重启之前是可以上网的，下次重启网关的mac地址还将是错误的

##### 查看：
```
netsh -c "i i" show neighbors 11

C:\Users\Administrator>netsh -c "i i" show neighbors 11

接口 11: 本地连接


Internet 地址                                 物理地址           类型
--------------------------------------------  -----------------  -----------
192.168.1.1                                   00-2b-3c-f0-ff-f1  永久
192.168.1.11                                  00-0c-29-47-fd-8d  停滞
192.168.1.63                                  00-00-00-00-00-00  无法访问
192.168.1.92                                  00-00-00-00-00-00  无法访问
......
```
##### 删除：
```
netsh -c "i i" delete neighbors 11

```
需要使用上面的命令删除，完了之后就会变成 “可以访问”


---


## ARP欺骗
ARP欺骗可以分成两种情况：
### 一、欺骗路由器
发送一系列错误的内网MAC地址给路由器，并按照一定的频率不断进行，使真实的地址信息无法通过更新保存在路由器中，结果路由器的所有数据只能发送给错误的MAC地址，造成正常PC无法收到信息。
### 二、欺骗局域网内的机器
不停发送错误的网关Mac到局域网的机器中，让局域网内的机器建立错误的绑定关系，让局域网内的机器不能正确的将数据包发送到网关设备，导致PC不能上网。

## ARP欺骗测试

### 使用发包工具
......
有时间再试试
......




## 其他
### 获取局域网某IP对应的Mac
```
> arping 192.168.1.88
ARPING 192.168.1.88 from 192.168.1.216 eth0
Unicast reply from 192.168.1.88 [E0:D5:5E:18:04:4A]  1.767ms
Unicast reply from 192.168.1.88 [E0:D5:5E:18:04:4A]  1.312ms
Unicast reply from 192.168.1.88 [E0:D5:5E:18:04:4A]  1.635ms
```

### 测试局域网个中某个IP是否被占用
返回值为1表示已被使用，0表示没有被使用
```
[root@RD-WEBSERVER-03 ~]# arping -D 192.168.1.163 -w 5  
ARPING 192.168.1.163 from 0.0.0.0 eth0
Sent 6 probes (6 broadcast(s))
Received 0 response(s)

[root@RD-WEBSERVER-03 ~]# arping -D 192.168.1.88 -w 5   
ARPING 192.168.1.88 from 0.0.0.0 eth0
Unicast reply from 192.168.1.88 [E0:D5:5E:18:04:4A]  1.848ms
Sent 1 probes (1 broadcast(s))
Received 1 response(s)
```


### wireshark
过滤目的地址
arp.dst.proto_ipv4==192.168.1.214


