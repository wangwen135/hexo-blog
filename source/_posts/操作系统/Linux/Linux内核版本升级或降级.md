---
title: Linux内核版本升级或降级
date: 2024-02-20 23:41
tags: 
  - Linux
categories:
  - [操作系统, Linux]
---


> 测试系统PVE 7.0


## PVE内核升级或降级


### 当前版本信息
```
# uname -r
5.11.22-1-pve


# lsb_release -a
No LSB modules are available.
Distributor ID: Debian
Description:    Debian GNU/Linux 11 (bullseye)
Release:        11
Codename:       bullseye

```

### 查找内核
```
# apt-cache search linux | grep 'PVE Kernel Image'

pve-kernel-5.11.17-1-pve - The Proxmox PVE Kernel Image
pve-kernel-5.11.21-1-pve - The Proxmox PVE Kernel Image
pve-kernel-5.11.22-2-pve - The Proxmox PVE Kernel Image
pve-kernel-5.11.7-1-pve - The Proxmox PVE Kernel Image
pve-kernel-5.4.119-1-pve - The Proxmox PVE Kernel Image
pve-kernel-5.4.124-1-pve - The Proxmox PVE Kernel Image
pve-kernel-5.4.128-1-pve - The Proxmox PVE Kernel Image

```

### 安装内核

```
# apt-get install pve-kernel-5.11.21-1-pve

apt-get install pve-kernel-5.4.128-1-pve

# 安装不了配置镜像
# 还不行挂代理

apt -o Acquire::http::proxy="http://192.168.1.100:10809/" install pve-kernel-5.4.128-1-pve

```


### 查看当前系统内核启动顺序
```
grep menuentry /boot/grub/grub.cfg

# 或者

cat /boot/grub/grub.cfg | grep menuentry

```


### 修改内核启动顺序
如果你升级的版本比当前内核版本高的话，默认新安装的内核就是第一顺序启动的，只需重启系统就行了，否则，则需要修改配置文件


找到上一步中的名称（启动到时候可以看到）  
如：

> Advanced options for Proxmox VE GNU/Linux  
> Proxmox VE GNU/Linux, with Linux 5.4.128-1-pve  



#### 修改/etc/default/grub中 GRUB_DEFAULT

**可以使用顺序号（从0开始）或使用菜单名称**

```
vi /etc/default/grub

# 将 GRUB_DEFAULT=0 修改想要的菜单，如果有二级菜单的，用> 符合指定
# 如这里的改成第二个菜单的第三项

GRUB_DEFAULT="1>2"

# 或者
#GRUB_DEFAULT="Advanced options for Proxmox VE GNU/Linux>Proxmox VE GNU/Linux, with Linux 5.4.128-1-pve"

```
**注意有二级菜单时要有引号**

其他示例：
- GRUB_DEFAULT= “Previous Linux versions>Ubuntu, with Linux 3.2.0-18-generic-pae”
- GRUB_DEFAULT= “Previous Linux versions>0”
- GRUB_DEFAULT= “2>0”
- GRUB_DEFAULT= “2>Ubuntu, with Linux 3.2.0-18-generic-pae”


### 更新引导并重启
```
update-grub

reboot

```

重启后，使用命令uname -r查看
```
# uname -r
5.4.128-1-pve
```

-----
-----


## PS
因为在PVE下创建虚拟机 Realtek RTL8125 2.5GbE 的网卡 与1G的交换机连接 无法跑满速（只有大概20 ~ 40Mb/s），为降内核版本了安装 realtek-r8125-dkms_9.005.06-1_amd64.deb 驱动进行测试


实际上据说也是驱动bug，新的r8169驱动也支持这个网卡并且修复了这个bug，但是实际上测试并没有......

#### 解决办法：
Chipset -> South Cluster Configuration -> PIC Express Configuration -> PCI Express Root Port

将全部PCI Express Root Port 的  ASPM 的Auto改成Disable

