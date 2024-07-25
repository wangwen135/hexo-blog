---
title: centos7 安装最新版本的git
date: 2020-05-21 16:10
tags: 
  - git
  - centos7
categories:
  - [版本控制, git]
---

### 1 最新git源码下载地址：
https://github.com/git/git/releases  
https://www.kernel.org/pub/software/scm/git/  
可以手动下载下来在上传到服务器上面

### 2 移除旧版本git
centos自带Git，7.x版本自带git 1.8.3.1（应该是，也可能不是），
安装新版本之前需要使用yun remove git卸载（安装后卸载也可以）。
```
[root@Git ~]# git --version    ## 查看自带的版本
git version 1.8.3.1
[root@Git ~]# yum remove git   ## 移除原来的版本
```

### 3 安装所需软件包
```
[root@Git ~]# yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel 
[root@Git ~]# yum install gcc-c++ perl-ExtUtils-MakeMaker
```
### 4 下载&安装
```
[root@Git ~]# cd /usr/src
[root@Git ~]# wget https://www.kernel.org/pub/software/scm/git/git-2.7.3.tar.gz
```

### 5 解压
```
[root@Git ~]# tar xf git-2.7.3.tar.gz
```

### 6 配置编译安装
```
[root@Git ~]# cd git-2.7.3
[root@Git ~]# make configure
[root@Git ~]# ./configure --prefix=/usr/local/git ##配置目录
[root@Git ~]# make profix=/usr/local/git
[root@Git ~]# make install
```

### 7 加入环境变量
```
[root@Git ~]# echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/profile
[root@Git ~]# source /etc/profile
```

### 8 检查版本
```
[root@Git git-2.7.3]# git --version 
git version 2.7.3
```
