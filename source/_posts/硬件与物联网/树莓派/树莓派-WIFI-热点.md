---
title: 树莓派-WIFI-热点
date: 2022-12-02 23:41
tags: 
  - 树莓派
categories:
  - [硬件与物联网, 树莓派]
---


### 1. 安装配置DNSmasq  
>DNSmasq是一个小巧且方便地用于配置DNS和DHCP的工具，适用于小型网络，它提供了DNS功能和可选择的DHCP功能。它服务那些只在本地适用的域名，这些域名是不会在全球的DNS服务器中出现的。DHCP服务器和DNS服务器结合，并且允许DHCP分配的地址能在DNS中正常解析，而这些DHCP分配的地址和相关命令可以配置到每台主机中，也可以配置到一台核心设备中（比如路由器），DNSmasq支持静态和动态两种DHCP配置方式。

**安装**
```
sudo apt-get install dnsmasq
```

**配置**  
修改/etc/dnsmasq.conf文件
```
sudo vi /etc/dnsmasq.conf
```
这个文件配置比较大，但是都注释了  
这里指定一下接口，配置一下dhcp分配的ip范围，在末尾加上：

```
interface=wlan0
dhcp-range=192.168.0.100,192.168.0.120,255.255.255.0,12h

```
如需要指定DNS解析，再加上：
```
addn-hosts=/etc/dnsmasq.hosts  
```
然后创建/etc/dnsmasq.hosts 文件，这里复制hosts文件
```
cp /etc/hosts /etc/dnsmasq.hosts

```
然后按照需要改内容，如：
```
192.168.0.1 raspberrypi
```

默认开机启动  
可以查看一下：
```
systemctl status dnsmasq
```

### 2. 安装hostapd  
>hostapd 是一个用户态用于AP和认证服务器的守护进程。  
它实现了IEEE 802.11相关的接入管理，IEEE 802.1X/WPA/WPA2/EAP 认证, RADIUS客户端，EAP服务器和RADIUS 认证服务器。Linux下支持的驱动有：Host AP，madwifi，基于mac80211的驱动。

**安装**
```
sudo apt-get install hostapd 
```

**配置**  
创建 /etc/hostapd/hostapd.conf 
```
#sudo vi /etc/hostapd/hostapd.conf
```
填入以下内容：  
```
interface=wlan0
hw_mode=g
channel=10
auth_algs=1
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_pairwise=CCMP
rsn_pairwise=CCMP

ssid=rp3
wpa_passphrase=123456789

```


### 3. 配置接口
将无线接口wlan0的IP配置成静态地址  

1. dhcpcd不再管理wlan0,避免设置冲突  
```
# sudo vi /etc/dhcpcd.conf
```
禁用wlan0接口
```
denyinterfaces wlan0
```

2. 给wlan0配置固定IP 
修改 /etc/network/interfaces 文件成如下：
```
allow-hotplug wlan0
#iface wlan0 inet dhcp
#    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
iface wlan0 inet static
    address 192.168.0.1
    netmask 255.255.255.0
    network 192.168.0.0
    broadcast 192.168.0.255

```
3. 重启相关服务
```
sudo service dhcpcd restart

sudo ifdown wlan0

sudo ifup wlan0

```


### 4 启动hostapd
```
sudo hostapd -B /etc/hostapd/hostapd.conf & > /dev/null 2>&1
```
如果没有问题，并且手机等设备可以连接

修改配置文件
```
sudo vi /etc/default/hostapd
```
```
##将##
#DAEMON_CONF=""  

##修改为##
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```
设置为开机启动
```
sudo systemctl enable hostapd

#查看状态
sudo systemctl status hostapd
```


----
----

## 重新使用wifi连接网络

停掉两个服务的自启动
```
sudo systemctl disable dnsmasq

sudo systemctl disable hostapd

```
关闭两个服务
```
sudo systemctl stop dnsmasq

sudo systemctl stop hostapd
```


启动wlan0接口的dhcpcd服务
```
sudo vi /etc/dhcpcd.conf

#注释掉如下：
#denyinterfaces wlan0

```

修改为获取动态IP
修改 /etc/network/interfaces 文件成如下：
```
allow-hotplug wlan0
iface wlan0 inet dhcp
    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

#iface wlan0 inet static
#    address 192.168.0.1
#    netmask 255.255.255.0
#    network 192.168.0.0
#    broadcast 192.168.0.255

```

重启相关服务和网卡
```
sudo service dhcpcd restart

sudo ifdown wlan0

sudo ifup wlan0

```


----
----
**（下面的没有测试）**
----

配置NAT  
-- 这先不弄，觉得没有必要转发  

修改/etc/sysctl.conf，更改（如果有这一行，把#号去掉就行）
```
net.ipv4.ip_forward=1


#替换命令
sudo sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/g' /etc/sysctl.conf

```
通过iptables做NAT转发：

```
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
```




