---
title: 配置IPTV用wifi
date: 2019-08-11 23:41
tags: 
  - 路由器&光猫
categories:
  - [硬件与物联网, 路由器&光猫]
---

使用原因：弱电箱在门口，光猫在弱电箱里面，从弱电箱到电视机只有一根网线，路由器放到电视柜上面，这根网线给了路由器用了，IPTV没有线。

其他方案：
1. 8芯网线分成两根4芯的网线（速度只有100M）
2. 电力猫（干扰大）
3. 使用网管交换机，用vlan （要另外买设备）
4. 路由器和光猫上分别配置Vlan


### 用超级账号登陆光猫

### 去掉宽带的无线绑定
**网络 ->  宽带设置**
连接名称 选择 2_INTERNET_R_VID_
将下面  绑定端口： 无线(2.4G-x) 的勾 去掉，点击应用
![修改端口绑定](https://upload-images.jianshu.io/upload_images/2043910-8eb87a8c8bec7967.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 设置iptv的无线绑定
**网络 ->  宽带设置**
选择 3_Other_B_VID_45
这个就是IPTV，将无线勾上，点击应用
![IPTV无线绑定](https://upload-images.jianshu.io/upload_images/2043910-138ed15fa441365f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 修改WLAN配置启动无线
**网络 -> WLAN配置 **
SSID 使能的勾勾上
![SSID使能](https://upload-images.jianshu.io/upload_images/2043910-1c08b9799567706f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

============================

### 机顶盒设置wifi连接
打开电视 和 机顶盒，按遥控的设置，进入维护登陆界面
机顶盒维护初始密码： 6321

进入网络选择无线，填入密码就行了（不行就试试将连接方式换成PPPOE试试，忘了）
