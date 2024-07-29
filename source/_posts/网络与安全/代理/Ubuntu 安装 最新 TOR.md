---
title: Ubuntu-安装-最新-TOR
date: 2017-11-23
tags: 
  - 代理
  - TOR
categories:
  - [网络与安全, 代理]
---



https://www.torproject.org/download/download-unix.html.en

按照这个站点上的进行操作

---
系统：Ubuntu

---

1、先设置软件包存储库，然后才能获取Tor

2、弄清楚你的发行版的名字
```
$ lsb_release -c
Codename:       precise

cat /etc/issue

```
 
3、先卸载之前安装的老版本  
sudo apt-get remove package --purge 删除包，包括配置文件等
```
sudo apt-get remove tor --purge
```

4、系统比较老，先升级
```
apt-get dist-upgrade

do-release-upgrade
```

在/etc/apt/sources.list.d/下添加一个新文件
```
vi torproject.list
```
内容如下：
```
deb http://deb.torproject.org/torproject.org xenial main
deb-src http://deb.torproject.org/torproject.org xenial main
```
或者
```
deb http://deb.torproject.org/torproject.org trusty main
deb-src http://deb.torproject.org/torproject.org trusty main
```


然后通过在命令提示符处运行以下命令来添加用于签署软件包的gpg密钥：

```
gpg --keyserver keys.gnupg.net --recv A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89
gpg --export A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89 | sudo apt-key add -
```

更新并安装

```
$ apt update
$ apt install tor
```


有可能会
```
W: 无法下载 http://deb.torproject.org/torproject.org/dists/xenial/InRelease  无法连接上 deb.torproject.org:80 (31.13.84.8)，连接超时
W: 部分索引文件下载失败。如果忽略它们，那将转而使用旧的索引文件。

```


我们提供了一个Debian软件包来帮助您保持我们的签名密钥。建议您使用它。安装它使用：

```
apt-get install deb.torproject.org-keyring

```



修改配置：/etc/tor/torrc

```
SOCKSPort 0.0.0.0:9150
```

重启服务：
```
service tor restart

```
