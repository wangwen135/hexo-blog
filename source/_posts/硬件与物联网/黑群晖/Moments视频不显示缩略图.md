---
title: Moments视频不显示缩略图
date: 2020-02-12 23:41
tags: 
  - 黑群晖
categories:
  - [硬件与物联网, 黑群晖]
---



洗白的方式比较麻烦

安装第三方FFmpeg是最快的解决方法



### 添加第三方源

#### 设置第三方源
套件中心 —> 设置 —> 常规 —> 选择任何发行者

套件中心 —> 设置 —> 套件来源 —> 新增
- 名称：synocommunity
- 位置：https://packages.synocommunity.com/
 

如果添加时提示：无效的位置  
查了下是系统证书过期导致的，更新证书
```
sudo mv /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt.bak && sudo curl -Lko /etc/ssl/certs/ca-certificates.crt https://curl.se/ca/cacert.pem

```
几个常用的第三方套件源：  
- https://spk.520810.xyz:666
- http://packages.synocommunity.com
- https://spk.imnks.com/
- https://spk7.imnks.com/?arch=apollolake    (7.0版本的)




#### 安装第三方ffmpeg
套件中心 —> 社群 —> 安装ffmpeg


#### 备份原始ffmpeg、创建第三方ffmpeg软连接
用ssh工具连接到群晖，切换到root账号
```
sudo -i
```

备份
```
mv /usr/bin/ffmpeg /usr/bin/ffmpeg_back

```
创建第三方ffmpeg软连接
```
ln -s /volume1/@appstore/ffmpeg/bin/ffmpeg /usr/bin/ffmpeg
```

#### 重建Moments索引
进入Moments套件，设置——常规——重建索引


