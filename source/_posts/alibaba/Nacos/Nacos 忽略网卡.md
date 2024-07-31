---
title: Nacos 忽略网卡
date: 2021-12-20 16:49
tags: 
  - Nacos
categories:
  - [alibaba,Nacos]
---


https://github.com/spring-cloud/spring-cloud-commons/blob/main/docs/src/main/asciidoc/spring-cloud-commons.adoc


注册是注册了一个不能访问的地址

默认是 eth0，比如我们要忽略这个网卡

直接再nacos上增加配置
```
spring:
  cloud:
    inetutils:
      ignored-interfaces: eth0
```

优先获取ip
```
spring.cloud.inetutils.preferred-networks=10.34.12
```

使服务优先获取前缀为10.34.12的IP


如果选择固定网卡配置项
```
spring.cloud.nacos.discovery.networkInterface = eth0

```

固定IP地址
```
spring:
  cloud:
    nacos:
      discovery:
        ip: 10.13.19.91
        
```