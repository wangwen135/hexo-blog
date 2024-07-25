---
title: Nginx-配置一个文件下载功能
date: 2024-05-01 23:41
tags: 
  - Nginx
categories:
  - [Nginx]
---


```
server {
    listen 443 ssl;
    server_name download.domain.com;
    
    # 证书
    ssl_certificate /etc/letsencrypt/live/domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/domain.com/privkey.pem;

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
    }

    # 禁止访问上级目录，防止目录遍历攻击
    location ~ /\.\./ {
        deny all;
    }

    # 禁止访问 .ht* 文件
    location ~ /\.ht {
        deny all;
    }
}
```


