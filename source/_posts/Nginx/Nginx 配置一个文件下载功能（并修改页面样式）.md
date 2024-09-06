---
title: Nginx 配置一个文件下载功能（并修改页面样式）
date: 2024-05-01 23:41
tags: 
  - Nginx
  - CSS
categories:
  - [Nginx]
---
> 2024-9-6 修改，新增CSS样式

nginx 指定一个目录提供文件列表和下载功能

### 原始的
#### nginx 配置文件
```
server {
    listen 8443 ssl;
    server_name download.yousite.com;
    
    access_log /var/log/nginx/downloads_access.log;
    
    # 证书
    ssl_certificate /etc/letsencrypt/live/yousite.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yousit.com/privkey.pem;
    
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout 10m;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # 根目录设置为 /www/mnt/downloads
    root /www/mnt/downloads;

    # 启用目录列表
    location / {
        autoindex on; # 启用目录列表
        autoindex_exact_size off; # 以更人性化的格式显示文件大小
        autoindex_localtime on; # 以服务器本地时间显示文件时间
        charset utf-8; # 设置字符集为 UTF-8
        try_files $uri $uri/ =404; # 确保请求的文件或目录存在，否则返回 404

        #阻止其他网站直接链接你的文件
        valid_referers none blocked yousite.com *.yousite.com;
        if ($invalid_referer) {
                return 403;
        }
    }

    # 禁止访问上级目录，防止目录遍历攻击
    location ~ /\.\./ {
        deny all;
    }

}
```

#### 展示效果
![1725635088030.png](https://img.wangwen135.top:23456/note/2024/09/66db1a136e0fa.png)

但是这个页面比较难看，特别是对于移动端浏览器很难用，文件都选不中，于是就想着修改一下

### 修改后的

不想使用其他的文件服务程序，就用 **sub_filter** 替换内容来实现引入一个CSS样式文件  

#### 修改后的nginx配置
```
server {
    listen 8443 ssl;
    server_name download.yousite.com;

    access_log /var/log/nginx/downloads_access.log;

    # 证书
    ssl_certificate /etc/letsencrypt/live/yousite.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yousite.com/privkey.pem;

    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout 10m;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # 根目录设置为 /www/mnt/downloads
    root /www/mnt/downloads;

    # 启用目录列表
    location / {
        autoindex on; # 启用目录列表
        autoindex_exact_size off; # 以更人性化的格式显示文件大小
        autoindex_localtime on; # 以服务器本地时间显示文件时间
        charset utf-8; # 设置字符集为 UTF-8
        try_files $uri $uri/ =404; # 确保请求的文件或目录存在，否则返回 404

        #内容替换
        sub_filter '</head>' '<link rel="stylesheet" href="/css/download.css"></head>';
        sub_filter '<title>Index of' '<title>目录';
        sub_filter '<h1>Index of' '<h1>目录';
        sub_filter '</body>' '<p>王某某的公开下载服务</p></body>';
        sub_filter '<hr>' '';
        sub_filter_once off;

        #阻止其他网站直接链接你的文件
        valid_referers none blocked yousite.com *.yousite.com;
        if ($invalid_referer) {
                return 403;
        }
    }

    # 禁止访问上级目录，防止目录遍历攻击
    location ~ /\.\./ {
        deny all;
    }

    #配置自定义的CSS
    location /css/download.css {
        alias /www/mnt/service/download/download.css;
    }

    #favicon.ico
    location /favicon.ico {
        alias /www/mnt/service/download/favicon.ico;
    }

}
```
> 主要是使用 sub_filter 替换内容，从而实现引入一个CSS 文件  

#### download.css文件内容
```
body {
    margin: 0;
    box-sizing: border-box;
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
}

body pre {
    font-family: Microsoft YaHei;
    font-size: 1rem;
    width: 960px;
    text-align: right;
    padding-right: 20px;
    margin-top: 0px;
}

a {
    text-decoration: none;
    border: 1px solid #ccc;
    border-radius: 5px;
    padding: 5px 10px;

    display: flex;
    position: relative;
    top: 26px;
    left: 10px;
}

a:hover {
    border-color: #000;
}

body p {
    margin-top: auto;
    margin-bottom: 10px;
}

/* 针对移动设备的 */
@media (max-width: 980px) {
    body pre {
        font-size: 1.4rem;
    }

    a {
        top: 35px;
    }
}
```
>将CSS文件保存到上面指定的位置，如：/www/mnt/service/download/download.css;

#### 展示效果
![1725634593930.png](https://img.wangwen135.top:23456/note/2024/09/66db1825604da.png)

##### 手机端浏览器
![1725634719193.png](https://img.wangwen135.top:23456/note/2024/09/66db18a29eb41.png)

