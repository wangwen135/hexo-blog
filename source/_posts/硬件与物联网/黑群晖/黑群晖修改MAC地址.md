---
title: 黑群晖修改MAC地址
date: 2021-09-22
tags: 
  - 黑群晖
categories:
  - [硬件与物联网, 黑群晖]
---


黑群晖多个虚拟网卡，MAC地址一样

挂载引导分区
```
sudo -i
# 输入密码

echo 1 > /proc/sys/kernel/syno_install_flag
mkdir -p /tmp/synoboot1
mount /dev/synoboot1 /tmp/synoboot1
cd /tmp/synoboot1
ls -l

```


修改引导文件
```
cd grub/

vi grub.cfg

```

修改内容，在set mac1=xxxxx 后面增加set mac2=xxx set mac3=xxx  
xxx 就是mac地址，按照格式填写就行了

```
set sn=1780PDN123458
set mac1=008132123456
set mac2=118132123457
set mac3=aa8132123458
set mac4=bb8132123459
```

> 之前已经将黑群晖 DS918+ 改成最多支持4个网卡了

