---
title: Burp Suite 修改服务端返回数据
date: 2021-06-30 23:41
tags: 
  - 渗透
  - Burp Suite
categories:
  - [网络与安全, 渗透]
---

### 1、Burp Suite设置代理

![设置代理.png](https://img.wangwen135.top:23456/image/2024/07/66a7b45776492.png)


![拦截服务端响应.png](https://img.wangwen135.top:23456/image/2024/07/66a7b4719626a.png)


> 也可在这里配置拦截某些匹配条件的服务端响应，如果条件能控制后就不用下面那样一个个的调试了

启动拦截

![启用拦截.png](https://img.wangwen135.top:23456/image/2024/07/66a7b49baa969.png)




### 2、其他端设置代理
如手机端，浏览器等等，配置为上面的Burp Suite 代理



### 3、访问目标网页并进行拦截
访问目标网页，在代理的拦截选项中右键选择  Do Intercept -->  Respense to this requests

![执行拦截.png](https://img.wangwen135.top:23456/image/2024/07/66a7b59c2bec9.png)


### 4、然后forward发送，可以看到服务器端返回的数据

![转发.png](https://img.wangwen135.top:23456/image/2024/07/66a7b5f1c0f56.png)

这里是Response

### 5、修改响应数据，此时数据还未传递到客户端（手机、浏览器等）

![修改响应数据.png](https://img.wangwen135.top:23456/image/2024/07/66a7b62ef2bdd.png)

这里是文本内容，随意修改

### 6、点击上面的forward，将结果返回给客户端

客户端收到的是修改过后的响应



----


## 其他

### 设置拦截匹配规则

拦截的请求太多，可以在Options中设置拦截规则，如只配某个站点的请求

![拦截规则.png](https://img.wangwen135.top:23456/image/2024/07/66a7b6840cfb8.png)
