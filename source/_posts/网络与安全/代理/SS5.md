---
title: SS5
date: 2021-12-17 23:41
tags: 
  - 代理
categories:
  - [网络与安全, 代理]
---

centos7安装socks5

项目地址：
http://ss5.sourceforge.net

下载地址：
https://sourceforge.net/projects/ss5/files/

## 一次安装配置的记录

下载解压
```
$ wget https://jaist.dl.sourceforge.net/project/ss5/ss5/3.8.9-8/ss5-3.8.9-8.tar.gz

$ tar -zxvf ss5-3.8.9-8.tar.gz

```

编译
```
$ cd ss5-3.8.9

$ ./configure --prefix=/usr/local/ss5

#提示缺省 pam 模块
......
......
checking for syslog.h... yes
checking for unistd.h... (cached) yes
checking security/pam_misc.h usability... no
checking security/pam_misc.h presence... no
checking for security/pam_misc.h... no
configure: error: *** Some of the headers weren't found ***
```

**安装pam**
```
$ yum -y install pam pam-devel
```

再编译
```
$ ./configure --prefix=/usr/local/ss5
......
......
checking for socket... yes
checking for strdup... yes
checking for strtol... yes
configure: creating ./config.status
config.status: creating Makefile
config.status: creating modules/Makefile
config.status: creating modules/mod_authen/Makefile
config.status: creating modules/mod_author/Makefile
config.status: creating modules/mod_balance/Makefile
config.status: creating modules/mod_bandwidth/Makefile
config.status: creating modules/mod_dump/Makefile
config.status: creating modules/mod_filter/Makefile
config.status: creating modules/mod_log/Makefile
config.status: creating modules/mod_proxy/Makefile
config.status: creating modules/mod_socks4/Makefile
config.status: creating modules/mod_socks5/Makefile
config.status: creating modules/mod_statistics/Makefile
config.status: creating common/Makefile
config.status: creating src/Makefile
config.status: creating include/config.h
```

```
$ make 
make[1]: 进入目录“/data/softwares/ss5-3.8.9/common”
gcc -g -O2 -DLINUX -D_FILE_OFFSET_BITS=64 -I . -I ../include   -fPIC   -c -o SS5OpenLdap.o SS5OpenLdap.c
SS5OpenLdap.c:29:18: 致命错误：ldap.h：没有那个文件或目录
 #include <ldap.h>
                  ^
编译中断。
make[1]: *** [SS5OpenLdap.o] 错误 1
make[1]: 离开目录“/data/softwares/ss5-3.8.9/common”
make: *** [common] 错误 2

```

**再装ldap**
```
yum -y install openldap-devel 

```

再来一次
```
$ make
make[1]: 进入目录“/data/softwares/ss5-3.8.9/common”
gcc -g -O2 -DLINUX -D_FILE_OFFSET_BITS=64 -I . -I ../include   -fPIC   -c -o SS5OpenLdap.o SS5OpenLdap.c
gcc -g -O2 -DLINUX -D_FILE_OFFSET_BITS=64 -I . -I ../include   -fPIC   -c -o SS5Radius.o SS5Radius.c
In file included from SS5Radius.c:22:0:
../include/SS5Radius.h:22:25: 致命错误：openssl/md5.h：没有那个文件或目录
 #include <openssl/md5.h>
                         ^
编译中断。
make[1]: *** [SS5Radius.o] 错误 1
make[1]: 离开目录“/data/softwares/ss5-3.8.9/common”
make: *** [common] 错误 2
```

**再安装openssl**
```
$ yum install openssl-devel

......
......

已安装:
  openssl-devel.x86_64 1:1.0.2k-8.el7                                                         
作为依赖被安装:
  keyutils-libs-devel.x86_64 0:1.5.8-3.el7     krb5-devel.x86_64 0:1.15.1-8.el7        libcom_err-devel.x86_64 0:1.42.9-10.el7     libkadm5.x86_64 0:1.15.1-8.el7       libselinux-devel.x86_64 0:2.5-11.el7    
  libsepol-devel.x86_64 0:2.5-6.el7            libverto-devel.x86_64 0:0.2.5-4.el7     pcre-devel.x86_64 0:8.32-17.el7             zlib-devel.x86_64 0:1.2.7-17.el7    

作为依赖被升级:
  e2fsprogs.x86_64 0:1.42.9-10.el7             e2fsprogs-libs.x86_64 0:1.42.9-10.el7        krb5-libs.x86_64 0:1.15.1-8.el7        libcom_err.x86_64 0:1.42.9-10.el7        libselinux.x86_64 0:2.5-11.el7       
  libselinux-python.x86_64 0:2.5-11.el7        libselinux-utils.x86_64 0:2.5-11.el7         libss.x86_64 0:1.42.9-10.el7           pcre.x86_64 0:8.32-17.el7               

完毕！
```

再来一次就正常了
```
make

make install
```
安装完成之后相关配置文件在：
/etc/opt/ss5/ 目录下
```
$ ll /etc/opt/ss5/
总用量 20
-rw-r--r-- 1 root root 11609 5月   2 15:07 ss5.conf
-rw-r--r-- 1 root root   118 5月   2 15:07 ss5.ha
-rw-r--r-- 1 root root     1 5月   2 15:07 ss5.passwd
```
启动一下试试看

**先给文件加上执行权限**
```
chmod +x /etc/rc.d/init.d/ss5
```
**启动服务**
```
$ service ss5 start
Reloading systemd:                                         [  确定  ]
Starting ss5 (via systemctl):                              [  确定  ]
```
检查一下端口（默认使用1080端口）
```
$ netstat -an | grep 1080
tcp        0      0 0.0.0.0:1080            0.0.0.0:*               LISTEN   
```
测试一下
```
$ curl --socks5 127.0.0.1:1080 http://www.baidu.com
curl: (7) No authentication method was acceptable. (It is quite likely that the SOCKS5 server wanted a username/password, since none was supplied to the server on this connection.)
```
并不行

修改ss5.conf文件试试看  
去掉 auth 和 permit 两项配置的注释
```
......
......
#       SHost           SPort           Authentication
#
auth    0.0.0.0/0               -               -

......
......
# /////////////////////////////////////////////////////////////////////////////////////////////////
#      Auth     SHost           SPort   DHost           DPort   Fixup   Group   Band    ExpDate
#
permit -        0.0.0.0/0       -       0.0.0.0/0       -       -       -       -       -

```
重启服务
```
[root@wwh215 init.d]# service ss5 stop 
Stopping ss5 (via systemctl):                              [  确定  ]
[root@wwh215 init.d]# service ss5 start
Starting ss5 (via systemctl):                              [  确定  ]
```
再用curl命令测试，能正常访问了
```
curl --socks5 127.0.0.1:1080 http://www.baidu.com

```
下一步测试需要授权的，使用用户名、密码授权

修复ss5.conf文件  
>后面改成 u
```
auth    0.0.0.0/0               -               u
```
修改ss5.passwd，增加一对用户名密码
```
test test
```

重启服务，再用上面的curl命令测试，发现又不能访问了  
加上用户名密码就可以访问了
```
curl --socks5 test:test@127.0.0.1:1080 --user test:test http://www.baidu.com   
```

测试完成

将ss5加入开机自启动
```
chkconfig --add ss5
```
------


## 精简版本

```
$ yum install pam-devel openldap-devel openssl-devel

#如果没有编译环境，也可以把gcc和make也安装上
$ yum -y install gcc automake make pam-devel openldap-devel cyrus-sasl-devel openssl-devel


$ wget https://jaist.dl.sourceforge.net/project/ss5/ss5/3.8.9-8/ss5-3.8.9-8.tar.gz

$ tar -zxvf ss5-3.8.9-8.tar.gz

$ cd ss5-3.8.9

$ ./configure --prefix=/usr/local/ss5

$ make

$ make install

$ chmod +x /etc/rc.d/init.d/ss5

# 修改 /etc/opt/ss5/ss5.conf 文件
# 去掉 auth 和 permit 前面的注释

$ service ss5 start

``` 
-----

### 修改ss5启动的参数，自定义代理端口

默认是1080

修改文件 **/etc/sysconfig/ss5**
将
```
#SS5_OPTS=” -u root”
```

取消注释，修改成下面这样
```
SS5_OPTS=" -u root -b 0.0.0.0:10808"
```

-----
## 防火墙配置

开放一个端口
```
firewall-cmd --zone=public --add-port=1080/tcp --permanent
```

重新加载防火墙规则并保存状态信息
```
firewall-cmd --reload
```

查看已经开放的端口
```
firewall-cmd --list-ports
```

-----

## 浏览器使用socks5 代理不能设置用户名密码的问题

firefox浏览器安装FoxyProxy Standard 扩展  

https://addons.mozilla.org/zh-CN/firefox/addon/foxyproxy-standard/

-----

## 服务启动失败的问题

服务器启动失败时，查看服务状态
```
# service ss5 status
● ss5.service - SYSV: This script takes care of starting and stopping ss5
   Loaded: loaded (/etc/rc.d/init.d/ss5; bad; vendor preset: disabled)
   Active: active (exited) since Mon 2018-05-28 16:51:10 CST; 1min 29s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 32456 ExecStart=/etc/rc.d/init.d/ss5 start (code=exited, status=0/SUCCESS)

May 28 16:51:10 MyCloudServer ss5[32456]: [29B blob data]
May 28 16:51:10 MyCloudServer ss5[32456]: Can't create pid file /var/run/ss5/ss5.pid
May 28 16:51:10 MyCloudServer ss5[32456]: Can't unlink pid file /var/run/ss5/ss5.pid
```
发现不能创建pid文件，原因是父级目录没有创建好

创建一下即可：
```
mkdir /var/run/ss5/
```
