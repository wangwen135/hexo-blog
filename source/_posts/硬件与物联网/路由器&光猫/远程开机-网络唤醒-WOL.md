---
title: 远程开机-网络唤醒-WOL
date: 2021-11-29 23:41
tags: 
  - 路由器&光猫
categories:
  - [硬件与物联网, 路由器&光猫]
---


## 什么是网络唤醒WOL
> 见维基百科：https://zh.wikipedia.org/wiki/%E7%B6%B2%E8%B7%AF%E5%96%9A%E9%86%92


**Wake-on-LAN**简称**WOL**或**WoL**，中文多译为“**网络唤醒**”、“**远程唤醒**”技术。WOL是一种技术，同时也是该技术的规范标准，它的功效在于让==休眠==状态或==关机==状态的电脑，透过局域网的另一台电脑对其发令，使其唤醒、恢复成运作状态，或从关机状态转成引导状态。

### 要求
要想实现WOL，首先必须要有具备WOL功能的主板、网卡。

现在的主板通常只需在BIOS中开启PCI/PCIe唤醒功能或网卡唤醒功能，主板内置的网卡可支持WOL。除了开启BIOS中的PCIe唤醒功能外，可能还要在操作系统中设置网卡的唤醒功能

> 除了古董机之外现在的电脑一般都支持，多在BIOS中找找相关选项

### 工作原理
电脑处在关机（或休眠）状态时，机内的网卡及主板部分仍保有微弱的供电，此微弱供电能让网卡保有最低的运作能力，使网卡能聆听来自电脑外部的网络广播信息，并对信息内容进行侦测与解读，一旦发现网络广播的内容中有特定的“魔法数据包”（Magic Packet），就会对该数据包的内容进行研判。

魔法数据包是以广播方式发送的，广播的方式与范畴可以是整个局域网（LAN），也可以是特定的子网（Subnet），同时魔法数据包内会有某部（或一群）电脑的网络地址资料，网卡一旦解读研判出所指的地址是自身所处的电脑时，网卡就会通知机内的主板、电源供应器，开始进行引导（或唤醒）的程序。


### 魔法数据包
**魔法数据包**（Magic Packet）是一个广播性的帧（frame），透过端口7或端口9进行发送，且可以用无连接（Connectionless protocol）的通信协议（如UDP、IPX）来传递，不过一般而言多是用UDP

在魔法数据包内，每次都会先有连续6个"FF"（十六进制，换算成二进制即：11111111）的资料，即：FF FF FF FF FF FF，在连续6个"FF"后则开始带出MAC地址信息，有时还会带出4字节或6字节的密码，一旦经由网卡侦测、解读、研判（广播）魔法数据包的内容，内容中的MAC地址、密码若与电脑自身的地址、密码吻合，就会引导唤醒、引导的程序。

如：
```
00000000 : FF FF FF FF FF FF 6C BC 4A B3 F4 BC 6C BC 4A B3 
00000010 : F4 BC 6C BC 4A B3 F4 BC 6C BC 4A B3 F4 BC 6C BC 
00000020 : 4A B3 F4 BC 6C BC 4A B3 F4 BC 6C BC 4A B3 F4 BC 
00000030 : 6C BC 4A B3 F4 BC 6C BC 4A B3 F4 BC 6C BC 4A B3 
00000040 : F4 BC 6C BC 4A B3 F4 BC 6C BC 4A B3 F4 BC 6C BC 
00000050 : 4A B3 F4 BC 6C BC 4A B3 F4 BC 6C BC 4A B3 F4 BC 
00000060 : 6C BC 4A B3 F4 BC 

```

## 设置

### BIOS设置
BIOS设置各不相同，如：

1. 高级 -> 唤醒事件设置 -> 将 PCIE设备唤醒 和 网络唤醒 设置为 允许 (Enable)

2. BIOS选单，选择Power Management Setup  把PME Event Wake Up 改成 Enabled即可设置成功

3. 进入BIOS设置，Power->Automatic Power On里面，设置Wake on LAN = Enable/Automatic

 
> 不同机器的BIOS设置位置不同，找到对应的Wake on LAN选项设置就OK。一般2010年后的网卡都支持网卡唤醒功能，如果在BIOS设置里面找不到相应的设置项，很可能默认就是开启的。



### 电脑端设置

进入网络适配器 ，选择对应网卡 右键 -> 属性 -> 配置 -> 高级
- 魔术封包唤醒 = 开启

电源管理 -> 勾选 允许此设备唤醒计算机


## 软件
这个网站有各个平台的工具
https://www.depicus.com/wake-on-lan/


### WEB网页工具
http://www.dslreports.com/wakeup

### 桌面软件

发包软件：  
https://www.depicus.com/downloads/wakeonlangui.zip

收包测试软件：  
https://www.depicus.com/downloads/wakeonlanmonitor.zip
> 不用真正的不停的开关机，这个软件有监听到数据包就可以了  
> 建议还是用Wireshark，规则栏里面写wol就可以，这个工具只能收指定端口的UDP包    

### 手机端控制软件

下载：

https://www.depicus.com/wake-on-lan/wake-on-lan-andriod

安装 -> 打开应用，提示不兼容 没有关系忽略

或者：https://apkpure.com/search?q=wake+on+lan  
里面有很多app


## 测试
这里用手机测试
> 手机连入和电脑相同的局域网中

输入mac地址，通过 getmac 或 ipconfig /all 获取

IP地址输入广播地址，如：192.168.31.255

子网掩码输入当前网络的子网掩码，如：255.255.255.0

端口随意，点击 Wake Up

即可启动电脑


## 公网远程开机
前提是：**路由器取得公网IP**
> 没有公网IP就打电话找网络运营商，如中国电信，让他们改  
> 光猫弄成桥接模式，用路由器拨号上网，要做端口映射  


直接百度IP
```
123.0.99.18（本机）  
地理地址: 中国 XXX XXX XXX  
运营商: 中国电信
```

#### 路由器设置端口映射

由于路由的端口转发不支持广播地址

> "子网定向广播”，默认情况下，大多数路由器和防火墙都禁用此选项

这里做一下 DHCP静态IP分配

设备名称 |	IP地址 |	MAC地址
----|----|----
TEST-PC | 192.168.31.111 | AA:BB:CC:11:11:11


端口转发规则列表：  

名称 |协议 | 外部端口 | 内部IP地址 | 内部端口
----|----|----|----|----
远程唤醒 | TCP和UDP | 999 | 192.168.31.111 | 9	



用唤醒工具，填入公网的ip
> 设置了DDNS后就可以用域名唤醒了，不用管IP的变化


注意子网掩码填：255.255.255.255  

> 这里唤醒的是指定的IP，不再是广播了，掩码要全是1


端口填 前面映射的999




-------------------------



## 问题
配置到这里，已经可以通过因特网发送唤醒数据包到我的电脑，电脑关机后两分钟内可以再次唤醒，但是超过2分钟，唤醒操作失败，原因就是路由器arp映射表动态更新后把关机电脑的arp项删除了，导致路由器接收到魔术包后不能正确的转发。

参考：  
https://github.com/melonbo/wolTool  
https://www.office26.com/luyouqi/miwifi-wol-waiwang.html  

## 解决办法
路由器开启ssh，获取root权限，配置静态arp

### 配置arp
通过ssh登录路由器

设置arp静态绑定mac
```
arp -s ip地址 ma地址

```
> 用 man arp 查看说明

完了可以用arp命令查看结果


#### 路由器重启自动配置

但是有个问题，重启之后会丢失
> window下就是永久的....

所以我们把这条指令写到启动脚本rc.local里面
```
$ vi /etc/rc.local 

# 增加arp映射
arp -s 192.168.31.111 XX:XX:XX:XX:XX:XX
```

##### 如果需要配置多个可以考虑配置在文件中

查看说明可以看到有个 arp -f 命令，可以从文件中读取，默认文件是：/etc/ethers


我们在这个文件中写入配置：
```
vi /etc/ethers

192.168.31.aa XX:XX:XX:XX:XX:XX
192.168.31.aa XX:XX:XX:XX:XX:XX
```

在开机启动下加入
```
vi /etc/rc.local 

arp -f
```


### 远程桌面
在电脑上设置一下开启远程桌面，设置一个强密码  
在路由器上再配置一个端口映射，远程桌面的端口是3389

名称    | 	协议	|   外部端口    |	内部IP地址      |	内部端口
---     |---        |---            |---                |---
远程桌面|	TCP和UDP|	43389       |	192.168.31.111  |	3389


#### 远程桌面软件：
电脑端：mstsc

手机端：RD client


---------------------

### Centos wol 工具
```
[root@wol ~]# yum -y install net-tools
# ether-wake [MAC address of the computer you'd like to turn on]
[root@wol ~]# ether-wake 00:22:68:5E:34:06   # send magick packets
```

指定接口
```
ether-wake -i eth0 11:22:33:44:55
```

### java 工具
https://github.com/wangwen135/wol4j  

直接java -jar 运行

参数说明：Mac地址 [广播地址] [端口]

如：
```
# java -jar wol4j-1.0.0.jar 22-00-DD-11-44-7A 192.168.1.255
```
> 默认端口是：9

这个工具也是发的UDP包


