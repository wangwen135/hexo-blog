---
title: SVN安装配置
date: 2016-06-13 14:28
tags: 
  - SVN
  - subversion
categories:
  - [版本控制, SVN]
---

### 查看：
```
rpm -qa subversion
```

### 列出：
```
yum list | grep subversion
```

### 安装：
```
yum install subversion
```

### 查看安装版本 
```
svnserve --version
```

### 创建SVN版本库目录 
```
mkdir -p /var/svn/svnrepos
```

### 创建版本库
```
svnadmin create /var/svn/svnrepos
```

执行了这个命令之后会在/var/svn/svnrepos目录下生成一些文件

进入conf目录（该svn版本库配置文件）
```
cd conf
```
    authz  文件是权限控制文件  
    passwd  是帐号密码文件  
    svnserve.conf SVN服务配置文件  

**以上文件， 左侧不能留空格, 否则会出错！**


### 设置帐号密码
```
vi passwd
```
在[users]块中添加用户和密码，格式：帐号=密码，如：hello=123

```
[users]
# harry = harryssecret
# sally = sallyssecret
hello=123
用户名=密码
```

### 设置权限
```
vi authz
```
指定单个用户：
```
[/]  
hello=rw  
```
*意思是hello用户对所有的目录有读写权限*  

指定用户组：
```    
[groups]
admin=admin,admin2,admin3
user=user1,user2,user3
[/]
@admin=rw   #admin组内的用户有obj的读写权限
@user=rw    #user组内的用户有obj的读写权限
```


### 修改svnserve.conf文件
```
vi svnserve.conf
```
打开下面的几个注释：

    anon-access = none  #匿名用无法访问  
    auth-access = write  #授权用户可写  
    password-db = passwd  #使用哪个文件作为账号文件  
    authz-db = authz  #使用哪个文件作为权限文件  
    realm = /var/svn/svnrepos  #认证空间名，版本库所在目录  



### 启动svn版本库
```
svnserve -d -r /var/svn/svnrepos

#改变端口
svnserve -d -r /opt/svn/repos --listen-port 3391
```

**-d [--daemon]**  后台运行  
**-r [--root]**  服务的根目录。可以具体指定到一个Repository的所在目录，仅提供一个Repository的访问；而如果要同时访问多个Repository，则-r选项需要指向多个Repository所在的父目录。

地址为: svn://your server address （如果指定端口需要添加端口  :端口号）


#### 停止SVN
```
ps -aux | grep svnserve

kill -9 ID号


killall svnserve
```
