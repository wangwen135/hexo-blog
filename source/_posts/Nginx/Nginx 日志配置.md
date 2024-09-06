---
title: Nginx 日志配置
date: 2021-06-10 23:41
tags: 
  - Nginx
  - Log
categories:
  - [Nginx]
---

## 错误日志配置
如果Nginx发生异常，它将在错误日志中记录所有事件

启用错误日志，使用error_log指令  
语法：
```
error_log log_file log_level ;
```
- 第一个参数表示日志文件路径，必填
- 第二个参数表示日志事件的安全级别， debug | info | notice | warn | error | crit
 
全局日志配置：
```
error_log /var/log/nginx/error.log;
```

为单个虚拟主机设置单独的错误日志
```
http {
       ...
       ...
       error_log  /var/log/nginx/error_log;
	   ...
       server {
                listen 80;
                server_name example1.com;
                error_log  /var/log/nginx/example1.error_log  warn;
                ...
       }
       server {
                listen 80;
                server_name example2.com;
                error_log  /var/log/nginx/example2.error_log  debug;
                ...
   }
}

```


## 访问日志配置

在HTTP中使用access_log指令启用访问日志
```
access_log log_file log_format ;
```
- log_file 指定日志文件路径和名称，必须要填 
- log_format 是可选的用于指定日志格式，不设置则将以默认的组合格式写入日志

默认情况下，访问日志是在HTTP块下定义的，因此，所有虚拟主机的访问日志将存储在同一配置文件中。

如：  
指定格式的配置
```
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

...
...
    server {
        ...
        ...
    }
```
或使用默认格式
```
http {
      ...
      access_log  /var/log/nginx/access.log;
      ...
}
```

如果需要将每个虚拟主机的日志单独记录，则可以为每个server块单独进行设置

```
http {
      ...
      access_log  /var/log/nginx/access.log;
      ...
         server {
                  listen 80;
                  Server_name example.com
                  access_log  /var/log/nginx/example.access.log;
                  ...
                  ...
                }
}

```

### 自定义日志格式
Nginx内置的默认格式
```
log_format combined '$remote_addr - $remote_user [$time_local] '
                '"$request" $status $body_bytes_sent '
                '"$http_referer" "$http_user_agent"'; 
```

**格式参数说明**

参数                        |  说明
----|------                 
$remote_addr 				|客户端ip地址（若使用了高匿代理服务器，显示代理服务ip），例如 123.57.78.100
$remote_port  				|客户端的端口，例如 54673
$http_x_forwarded_for  		|可以记录客户端IP，通过代理服务器来记录客户端的ip地址
$remote_user 				|用户记录远程客户端的用户名称，一般为 '-'
$http_host（$host） 		|浏览器中输入的地址（IP或域名），例如： proxy.mimvp.com 或 123.57.78.100
$time_local   				|用具记录访问时间和时区，日期格式：18/Feb/2017:14:10:17 +0800
$time_iso8601   			|用具记录访问时间和时区，日期格式：2017-02-18T14:10:17+08:00
$status 					|响应状态码 ‘404’页面找不到 ‘200’成功等
$request_time 				|整个请求的总时间，从接收用户请求的第一个字节到发送完响应数据的时间，即包括接收请求数据时间，程序响应时间，输出响应数据时间
$bytes_sent  				|传输给客户端的全部字节数，包含响应头等信息
$body_bytes_sent 			|给客户端发送的文件主题内容字节数，响应头不计算在内
$request_length 			|请求的长度（包括请求的地址，http请求头和请求主体）
$http_referer  				|url跳转来源，用来记录从哪个页面链接访问过来的，例如：https://proxy.mimvp.com
$upstream_addr  			|后台提供服务的地址（即转发处理的目标地址）
$upstream_response_time  	|从nginx向后端建立连接开始到接受完数据然后关闭连接为止的时间
$upstream_status 			| upstream状态，例如 200
$http_user_agent 			|用户所使用的代理（一般为浏览器）
$request   					|用于记录请求的url以及请求方法，格式："GET /free.php?proxy=in_tp HTTP/1.1"
$request_method   			|用于记录请求的url以及请求方法，格式：GET、POST
$request_body 				|客户端的请求主体，此变量可以在location中使用，将请求主体通过proxy_pass,fastcgi_pass,uwsgi_pass和scgi_pass传递给下一级的代理
$args 						|请求中的参数值，格式："proxy=in_tp"
$uri 						|请求中的当前URI（不带请求参数，参数位于$args），可以不同于浏览器传递的$request_uri的值，它可以通过内部重定向，或者使用index指令修改，格式："/free.php"
$request_uri 				|这个变量等于包含一些客户端请求参数的原始URI，它无法修改
$ssl_protocol   			|SSL协议版本，例如 TLSv1、TLSv2
$ssl_cipher   				|SSL协议的交换数据中的算法，例如 RC4-SHA，ECDHE-RSA-AES128-GCM-SHA256，ECDHE-RSA-AES256-GCM-SHA384
$geoip_country_code  		|用户地理位置代码（国家代码）
$http_accept_language  		|用户浏览器语言。如：上例中的 "es-ES,es;q=0.8"
$http_true_client_ip 		|客户端的真实ip地址



```
http {
            log_format custom '$remote_addr - $remote_user [$time_local] '
                           '"$request" $status $body_bytes_sent '
                           '"$http_referer" "$http_user_agent" "$gzip_ratio"';

            access_log /var/log/nginx/example.access.log custom;
......
}
```