---
title: Privoxy-安装配置
date: 2022-10-15 23:41
tags: 
  - 代理
categories:
  - [网络与安全, 代理]
---

使用Privoxy 将socks5代理转成http代理

操作系统 CentOS 7

#### 1、先安装epel源
```
yum install epel-release

#安装完成之后可以通过下面的命令查看
yum repolist


#可以看到多了一个
#Extra Packages for Enterprise Linux 7 - x86_64
```


#### 2、使用yum命令安装

用yum命令看一下，是最新版本

```
 yum info privoxy
 
名称    ：privoxy
架构    ：x86_64
版本    ：3.0.26
发布    ：1.el7
大小    ：936 k
源    ：epel/x86_64
简介    ： Privacy enhancing proxy
网址    ：http://www.privoxy.org/
协议    ： GPLv2+
描述    ： Privoxy is a web proxy with advanced filtering capabilities for
         : protecting privacy, filtering web page content, managing cookies,
         : controlling access, and removing ads, banners, pop-ups and other
         : obnoxious Internet junk. Privoxy has a very flexible configuration and
         : can be customized to suit individual needs and tastes. Privoxy has application
         : for both stand-alone systems and multi-user networks.
         : 
         : Privoxy is based on the Internet Junkbuster.
```

直接安装 privoxy
```

yum install privoxy

```

#### 3、 配置
配置文件位于目录：**/etc/privoxy**

##### 3.1、修改config文件

>说明：
https://www.privoxy.org/user-manual/config.html

修改绑定地址，搜索 ==listen-address== ，修改需要绑定的IP

```
listen-address  0.0.0.0:8118
```

设置socks5 转发，搜索 ==forward-socks5t== ，去掉注释，修改对应IP

```

forward-socks5t   /               118.193.225.166:9150 .
```
>注意后面的点不要删掉

配置不走代理，直接本地转发的
```
forward         192.168.*.*/     .
forward           127.*.*.*/     .
```

------

由于网络不稳定，经常出现503，增加转发重试  
默认值是：0
```
forwarded-connect-retries  1
```

配置最大客户端的连接  
默认值是：128
```
max-client-connections 256
```

-----
这个用于开启和关闭广告过滤和内容过滤，1表示开启，0表示关闭
默认值是：1
```
toggle  0
```

----

共享连接

是否保持活动的传出连接应该在不同的传入连接之间共享

>这个还没怎么测试，按照字面意思理解
```
connection-sharing 1
```



##### 3.1、修改user.action 配置文件

拦截服务端禁止在iframe中加载的响应头，在user.action 末尾添加

**只能处理http的连接**
```
{ +crunch-server-header{X-Frame-Options} }
/
```

----

修改服务端的响应头，去掉设置cookie时的 HttpOnly ，让客户端可以通过js获取cookie  
Privoxy 使用类似Perl的 s/// 操作来实现对内容的替换修改

>注意，它使用的是|作为分隔符，而不是/，因为模式包含一个正斜杠，否则必须以反斜杠(\\)来转义。如果表达式中有|线则用@符号。（文档中没有看到明确的说明，但是例子中是这么写的）

**只能处理http的连接**

在user.filter文件中新增
```
SERVER-HEADER-FILTER: delete-http-only delete server response head setCookie http only tag
s@^(Set-Cookie.+)(;[ ]*httponly)@$1@i
```

在user.action文件中新增
```
{+server-header-filter{delete-http-only}}
/

```

----

#### 4、启动服务

```
$ systemctl start privoxy
$ systemctl status privoxy

● privoxy.service - Privoxy Web Proxy With Advanced Filtering Capabilities
   Loaded: loaded (/usr/lib/systemd/system/privoxy.service; disabled; vendor preset: disabled)
   Active: active (running) since 三 2017-11-15 16:38:12 CST; 9s ago
  Process: 22643 ExecStart=/usr/sbin/privoxy --pidfile /run/privoxy.pid --user privoxy /etc/privoxy/config (code=exited, status=0/SUCCESS)
 Main PID: 22644 (privoxy)
   CGroup: /system.slice/privoxy.service
           └─22644 /usr/sbin/privoxy --pidfile /run/privoxy.pid --user privoxy /etc/privoxy/config

```


#### 5、检查测试

```
netstat -an | grep 8118

tcp        0      0 127.0.0.1:8118          0.0.0.0:*               LISTEN     
```


#### 6、浏览器访问

浏览器配置代理指向privoxy

访问地址：http://p.p/ 可以进入到privoxy的一个管理页面


访问地址：https://check.torproject.org/ 可以进入到一个tor检查页面

