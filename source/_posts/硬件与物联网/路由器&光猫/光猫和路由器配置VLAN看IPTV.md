---
title: 光猫和路由器配置VLAN看IPTV
date: 2020-05-25 23:41
tags: 
  - 路由器&光猫
categories:
  - [硬件与物联网, 路由器&光猫]
---

**路由器型号 小米R2D**

### 路由器端口
wan 口 编号：4
lan 口 编号：0 2 3
CPU端口： 5 

------------------------

### 参考资料
https://openwrt.org/zh-cn/doc/uci/network
https://openwrt.org/docs/guide-user/network/vlan/switch
https://openwrt.org/zh/docs/guide-user/network/vlan/switch_configuration


### 相关资料
一般来说，0、1、2、3是路由器LAN口，4是路由器WAN口，5表示CPU，而5*表示这个接口是trunk

使用“ *”和“ u”分别表示PVID和未标记的端口（因为它们具有隐式标记的CPU端口，因此需要使用“ u”来取消标记） ）。

在端口上收到的未标记的数据包将被定向到默认端口VLAN（通常称为PVID）。需要一个单独的config switch_port部分来设置默认端口VLAN
虚拟局域网。

小米是定制的openwrt系统，采用的是博通闭源驱动，因此vlan设置不能采用openwrt的设定方式，必须采用类似于dd-wr闭源驱动nvram set方式才能使vlan生效。具体是修改/etc/config/misc，将相应的vlanXports参数修改成/etc/config/network里面的port端口号，甚至需要修改/etc/init.d/boot里面的nvram vlan配置参数，然后reboot，重启，新的vlan端口充当wan才能生效


-----------------------

#### 原来的

端口编号 | 5 | 0 | 2 | 3 | 4
--|--|--|--|--|--
物理接口 | CPU (eth0) |	LAN 1 |	LAN 2 |	LAN 3 |	WAN
VLAN ID 1 (eth0_1) |	已标记 |	未标记 |	未标记 |	未标记 |	禁用
VLAN ID 2 (eth0_2) |	已标记 |	禁用   |	禁用   |	禁用   |	未标记


#### 修改后
端口编号 | 5 | 0 | 2 | 3 | 4
--|--|--|--|--|--
物理接口 | CPU (eth0) |	LAN 1 |	LAN 2 |	LAN 3 |	WAN
VLAN ID 1 (eth0_1) |	已标记 |	未标记 |	未标记 |	禁用   |	禁用
VLAN ID 2 (eth0_2) |	已标记 |	禁用   |	禁用   |	禁用   |	已标记（Internet）
VLAN ID 3 (eth0_3) |	已标记 |	禁用   |	禁用   |	未标记 |	已标记 （IPTV）

**LAN3口直接连接机顶盒**

> 这个没有测试，因为下面这个更简单

-----------------
-----------------
-----------------
-----------------
-----------------
-----------------

## 使用robocfg 配置VLAN

> 上面配置比较麻烦，还是下载一个 robocfg工具，通过工具来进行配置

下载地址：
https://www.right.com.cn/forum/forum.php?mod=viewthread&tid=334441&page=1
> 就是按照这个弄的

### 工具说明
Broadcom BCM5325/535x/536x/5311x switch configuration utility

其实这个CPU是 CPU BCM4709C


### 复制文件

先弄到路由器的硬盘上

为了之后使用方便，再复制文件到/usr/bin目录
```
cp /userdisk/data/ftp/robocfg /usr/bin/
cp: can't create '/usr/bin/robocfg': Read-only file system
```
报错，提示是只读的

以读写方式重新挂载根目录
```
mount -o remount rw /
```

然后再复制就可以了

加上执行权限
```
chmod +x robocfg
```


### 查看现有VLAN配置
robocfg show
```
# ./robocfg show
Switch: enabled 
Port 0:   DOWN enabled stp: none vlan: 1 jumbo: off mac: 00:00:00:00:00:00
Port 1:   DOWN enabled stp: none vlan: 1 jumbo: off mac: 00:00:00:00:00:00
Port 2: 1000FD enabled stp: none vlan: 1 jumbo: off mac: ec:00:00:d4:00:xx
Port 3:   DOWN enabled stp: none vlan: 1 jumbo: off mac: d4:00:00:c1:00:xx
Port 4: 1000FD enabled stp: none vlan: 2 jumbo: off mac: 00:00:01:00:00:xx
Port 8:   DOWN enabled stp: none vlan: 1 jumbo: off mac: 00:00:00:00:00:00
VLANs: BCM5301x enabled mac_check mac_hash
   1: vlan1: 0 2 3 5t
   2: vlan2: 4 5t
```

### 重新配置VLAN

多插拔两次就可以确定 物理网卡 与 port 0 1 2 3 4 的对应关系了



Port | 物理端口
---|---
Port 0 | LAN 口 1
Port 2 | LAN 口 2
Port 3 | LAN 口 3
Port 4 | WAN 口
Port 5 | CPU端口


```
robocfg vlan 3 ports "3 4t"
```

配置之后
```
root@XiaoQiang:~# robocfg vlan 3 ports "3 4t"
root@XiaoQiang:~# robocfg show
Switch: enabled 
Port 0: 1000FD enabled stp: none vlan: 1 jumbo: off mac: xx:xx:xx:xx:40:75
Port 1:   DOWN enabled stp: none vlan: 1 jumbo: off mac: 00:00:00:00:00:00
Port 2:   DOWN enabled stp: none vlan: 1 jumbo: off mac: xx:xx:xx:xx:40:75
Port 3:   DOWN enabled stp: none vlan: 3 jumbo: off mac: xx:xx:xx:xx:d0:4f
Port 4: 1000FD enabled stp: none vlan: 2 jumbo: off mac: 00:xx:xx:xx:xx:58
Port 8:   DOWN enabled stp: none vlan: 1 jumbo: off mac: 00:00:00:00:00:00
VLANs: BCM5301x enabled mac_check mac_hash
   1: vlan1: 0 2 3 5t
   2: vlan2: 4 5t
   3: vlan3: 3 4t
```


### 光猫配置
取消 Internet 和 IPTV 连接的端口绑定，使用VLAN绑定


默认的 Internet 是没有VLAN的，IPTV默认有两个VLAN：45 和 47 



#### 配置VLAN绑定
用户侧Vlan ID 为上面定义的 3，Wan口Vlan ID 就填写IPTV的Vlan ID

> 选千兆口


类型 | 用户侧Vlan | WAN侧Vlan
---|---|---
IPTV 单播 | 3 | 45
IPTV 组播 | 3 | 47



### 开机自动执行
将上面的命令写入 /etc/rc.local 文件中
```
......
robocfg vlan 3 ports "3 4t"
exit 0
```


### 路由器连接光猫
将路由器的3号lan口与机顶盒用网线连接即可


### 总结
看4K 高清不卡顿，比wifi稳定，wifi卡是因为干扰太多，弱电箱的位置不好，弱电箱有金属屏蔽了信号，导致看4K高清时偶尔会卡顿，wifi的带宽其实是足够了，机顶盒的网口也是百兆的，wifi还有300兆





