---
title: SVN迁移
date: 2016-08-13 11:13
tags: 
  - SVN
categories:
  - [版本控制, SVN]
---


从window平台迁移到linux平台

### 导出
    windows 平台导出到 linux 的时候不能使用powershell，需要用CMD导出dump文件  
    否则会出现：`svnadmin: E140001: XXX`错误

```
#svnadmin dump 原先的repos的目录路径（/repository/directory） > dumpfile

svnadmin dump /opt/svn/xxx/ > /var/tmp/dump-xxx.svn
```


### 创建

    在目标机器上创建同名的资源库
    
```
svnadmin create xx1

svnadmin create software

svnadmin create xx2
```


### 导入

```
svnadmin load xx1 < /data/svn-backup/repository-xx1-0826.svn

svnadmin load software < /data/svn-backup/repository-software-0826.svn

svnadmin load xx2 < /data/svn-backup/repository-xx2-0826.svn
```


### SVN重定向
```
svn help relocate
relocate: 重新定位工作副本，指向不同的版本库根 URL。
用法:  1. relocate FROM-PREFIX TO-PREFIX [PATH...]
       2. relocate TO-URL [PATH]

  改写工作副本 URL 元数据，以反映仅版本库根 URL 的改变。这用于仅版本库根
  URL 改变(例如方案或主机名称)，但是工作副本内容仍旧与版本库对应的情况。

  1. FROM-PREFIX 和 TO-PREFIX 分别对应工作副本的旧 URL 与新 URL 开始子串
     (如果你喜欢，可以指定完整的 URL)。请使用 'svn info' 来确定当前工作
     副本的 URL。

  2. TO-URL 是用于 PATH 的(完整的)新版本库 URL。

  例如:
    svn relocate http:// svn:// project1 project2
    svn relocate http://www.example.com/repo/project \
                 svn://svn.example.com/repo/project

有效选项: 
  --ignore-externals       : 忽略外部项目

全局选项: 
  --username ARG           : 指定用户名称 ARG
  --password ARG           : 指定密码 ARG
  --no-auth-cache          : 不要缓存用户认证令牌
  --non-interactive        : 不要交互提示
  --trust-server-cert      : 不提示的接受未知的证书颁发机构发行的 SSL 服务器证书(只用于选项 “--non-interactive”)
  --config-dir ARG         : 从目录 ARG 读取用户配置文件
  --config-option ARG      : 以下属格式设置用户配置选项：
                                 FILE:SECTION:OPTION=[VALUE]
                             例如：
                                 servers:global:http-library=serf
```


查看信息：
````
svn info
````
```
路径: .
工作副本根目录: /home/svn/xx1-parent
URL: https://192.168.1.99/svn/xx1/trunk/xx1-parent
版本库根: https://192.168.1.99/svn/xx1
版本库 UUID: 6b1cbf4a-421d-7440-86c9-1259917cfc4a
版本: 966
节点种类: 目录
调度: 正常
最后修改的作者: wwh
最后修改的版本: 966
最后修改的时间: 2016-08-25 18:10:15 +0800 (四, 2016-08-25)
```

命令
```
#svn relocate 【之前svn路径】【新的svn路径】
xx1-parent]# svn relocate https://192.168.1.99/svn/xx1/trunk/xx1-parent svn://192.168.1.210/xx1/trunk/xx1-parent
```

```
svn relocate https://192.168.1.99/svn/xx1/trunk/xx1-parent svn://192.168.1.210/xx1/trunk/xx1-parent

认证领域: <svn://192.168.1.210:3690> 6b1cbf4a-421d-7440-86c9-1259917cfc4a
“root”的密码: 
认证领域: <svn://192.168.1.210:3690> 6b1cbf4a-421d-7440-86c9-1259917cfc4a
用户名: wang.wenhua
“wang.wenhua”的密码: 

-----------------------------------------------------------------------
注意!  你的密码，对于认证域:

   <svn://192.168.1.210:3690> 6b1cbf4a-421d-7440-86c9-1259917cfc4a

只能明文保存在磁盘上!  如果可能的话，请考虑配置你的系统，让 Subversion
可以保存加密后的密码。请参阅文档以获得详细信息。

你可以通过在“/root/.subversion/servers”中设置选项“store-plaintext-passwords”为“yes”或“no”，
来避免再次出现此警告。
-----------------------------------------------------------------------
保存未加密的密码(yes/no)?yes
```

eclipse中    
>切换到SVN 视图，右键资源库，选择重定位 --> 输入新的URL  

>如果不能重定位，可在资源管理器的目录上使用SVN客户端重定位
>在项目上右键 --> team --> 断开连接 （不删除SVN元信息）
>在SVN视图中创建新的资源库，指向新的地址
>然后再在项目上右键 --> team --> share project（eclipse会识别目录中SVN信息）


TortoiseSVN客户端:  
>在工作复本的根目录上右键 --> TortoiseSVN --> 重新定位(Relocate)，然后修改URL
