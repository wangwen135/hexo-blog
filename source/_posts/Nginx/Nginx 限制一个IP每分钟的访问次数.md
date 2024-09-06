---
title: Nginx 限制一个IP每分钟的访问次数
date: 2023-04-10 23:41
tags: 
  - Nginx
  - Limit
categories:
  - [Nginx]
---


在 Nginx 中，你可以使用 **ngx_http_limit_req_module** 模块来限制一个 IP 每分钟的访问次数。

首先，确保 Nginx 已经启用 ngx_http_limit_req_module 模块，默认情况下，这个模块是启用的。如果不确定，可以通过 nginx -V 检查。


### 配置示例：
```
http {
    # 定义限制规则，rate 表示每秒允许的请求数
    limit_req_zone $binary_remote_addr zone=one:10m rate=10r/m;

    server {
        listen 80;
        server_name example.com;

        location / {
            # 应用限制
            limit_req zone=one burst=5 nodelay;

            # 其他配置...
            proxy_pass http://backend;
        }
    }
}
```

### 说明：

#### limit_req_zone $binary_remote_addr zone=one:10m rate=10r/m;

该行定义了一个限流区域 one，使用 $binary_remote_addr 作为键（即基于客户端 IP 地址），总共使用 10MB 内存（大约可存储 16,000 个唯一 IP 地址）。rate=10r/m 表示每个 IP 地址每分钟最多允许 10 个请求。


#### limit_req zone=one burst=5 nodelay;
应用限流规则，burst=5 允许最多积累 5 个请求的突发请求，nodelay 指定突发请求也立即处理，而不是按速率排队。


##### 配置：
limit_req zone 不能直接写在 server 段中，必须写在具体的 location 段内。limit_req 指令是作用于某个 location，用于限制该 location 的访问速率。

如果你的 server 段下配置的是一个 root，你仍然可以在 location 段中使用 limit_req 进行速率限制，而不必具体区分路径。实际上，即使你不明确定义 location，Nginx 也会有一个默认的隐式 location /，你只需在该隐式 location 中添加 limit_req 规则即可。

```
# 对所有请求设置访问限制
location / {
    limit_req zone=one burst=5 nodelay;
}
```

#### 速度说明
在 Nginx 中，**rate=10r/m** 实际上是按照秒来计算的，即**每 6 秒钟允许 1 个请求**，而不是严格按照 1 分钟的时间间隔。虽然配置中使用的是 "r/m"（每分钟请求数），但 Nginx 是将这个速率平滑到每秒来控制请求的频率。

如果请求频率超过了这个速率（即短时间内请求超过这个阈值），Nginx 将根据 burst 参数来决定是否允许额外请求排队，或直接拒绝返回 503 错误。

**burst** 不涉及时间单位，它只是指在短时间内允许的突发请求数。当请求超过速率限制时，Nginx 会将多余的请求放入队列，队列的长度最多为 **burst**，之后的请求将被拒绝（返回 503 错误）。

### 测试

打开测试页面，强制刷新10次（快速刷新7次），就能看到
```
503 Service Temporarily Unavailable
```







