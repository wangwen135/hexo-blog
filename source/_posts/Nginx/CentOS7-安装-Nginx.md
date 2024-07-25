---
title: CentOS7-安装-Nginx
date: 2023-09-21 23:41
tags: 
  - Nginx
categories:
  - [Nginx]
---

# CentOS7 安装 Nginx

Nginx是一个高性能的Web服务器软件。它是一个比Apache HTTP服务器更灵活和轻量级的程序。

## 安装
1. 安装epel源
    EPEL (Extra Packages for Enterprise Linux，企业版Linux的额外软件包) 是Fedora小组维护的一个软件仓库项目，为RHEL/CentOS提供他们默认不提供的软件包。这个源兼容RHEL及像CentOS和Scientific Linux这样的衍生版本。  
    我们可以很容易地通过yum命令从EPEL源上获取上万个在CentOS自带源上没有的软件。EPEL提供的软件包大多基于其对应的Fedora软件包，不会与企业版Linux发行版本的软件发生冲突或替换其文件。
```
yum install epel-release
#安装完成之后可以通过下面的命令查看
yum repolist
#可以看到多了一个
#Extra Packages for Enterprise Linux 7 - x86_64
```
2. 安装Nginx
```
yum install nginx
```
3. 启动Nginx
```
systemctl start nginx
```
如果运行防火墙，请运行以下命令以允许HTTP和HTTPS流量：
```
sudo firewall-cmd --permanent --zone=public --add-service=http 
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```
4. 访问  
在浏览器中访问
```
http://server_domain_name_or_IP/
http://192.168.1.213/
```
能看Nginx的默认页
5. 系统启动时启动Nginx
```
systemctl enable nginx
```
6. 配置  
**默认服务器根**  
默认服务器根目录是 /usr/share/nginx/html 放置在其中的文件将在您的Web服务器上提供访问。  
此位置在Nginx附带的默认服务器块配置文件中指定，该文件位于 /etc/nginx/conf.d/default.conf  
**服务器块配置**  
可以通过在/etc/nginx/conf.d中创建新的配置文件来添加任何附加的服务器块（在Apache中称为虚拟主机）。当Nginx启动时，将加载该目录中以.conf结尾的文件。  
**Nginx全局配置**  
主要的Nginx配置文件位于/etc/nginx/nginx.conf。在这里您可以更改设置，如运行Nginx守护进程的用户，以及当Nginx运行时生成的工作进程数等。

-----

-----

*忘了之前是怎么安装的了  ~~*


#### 服务名是：
nginx16-nginx.service

#### 配置文件位于：  

/opt/rh/nginx16/root/etc/nginx/nginx.conf

内容：

```
error_log  /var/log/nginx16/error.log;

......

access_log  /var/log/nginx16/access.log  main;
   
......

 server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  /opt/rh/nginx16/root/var/log/nginx/host.access.log  main;

        location / {
            #root   /opt/rh/nginx16/root/usr/share/nginx/html;
            #index  index.html index.htm;

            root   /data/bdmi/html;
            index  login.html index.html index.htm;

        }


        location /baseUrl {
            rewrite  ^.+baseUrl/?(.*)$ /bdmi-biz/$1 break;
            include  uwsgi_params;
            proxy_pass   http://192.168.1.92:8080;

            proxy_set_header Cookie $http_cookie;
            proxy_cookie_path /bdmi-biz /;

            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        
......

```

#### 关于SELINUX问题

首先查看本机SELinux的开启状态，如果SELinux status参数为enabled即为开启状态

```
/usr/sbin/ sestatus -v
```

或者使用getenforce命令检查

SELinux can be run in **enforcing**, **permissive**, or **disabled** mode

To add httpd_t to the list of permissive domains, run this command:
```
# semanage permissive -a httpd_t
```

To delete httpd_t from the list of permissive domains, run:
```
# semanage permissive -d httpd_t

```

To set the mode globally to enforcing, run:
```
# setenforce 1

```
