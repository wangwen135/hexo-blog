---
title: Burp-Suite-修改服务端返回数据
date: 2021-06-30 23:41
tags: 
  - 渗透
categories:
  - [网络与安全, 渗透]
---

### 1、Burp Suite设置代理
![设置代理](https://upload-images.jianshu.io/upload_images/2043910-d7730f374c9ff264.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![拦截服务端响应](https://upload-images.jianshu.io/upload_images/2043910-a8d1112794319487.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 也可在这里配置拦截某些匹配条件的服务端响应，如果条件能控制后就不用下面那样一个个的调试了

启动拦截
![启用拦截](https://upload-images.jianshu.io/upload_images/2043910-e5da46a7f6d2acd5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 2、其他端设置代理
如手机端，浏览器等等，配置为上面的Burp Suite 代理

### 3、
