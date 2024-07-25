---
title: ElasticSearch-5-2-X-安装ElasticSearch-head
date: 2022-10-13 23:41
tags: 
  - Elasticsearch
categories:
  - [Elasticsearch]
---

1. 安装nodejs环境
```
wget https://npm.taobao.org/mirrors/node/latest-v4.x/node-v4.4.7-linux-x64.tar.gz

tar -zxvf node-v4.4.7-linux-x64.tar.gz

export PATH=$PATH:/soft/node-v4.4.7-linux-x64/bin


```
nodejs包中已经包含了nmp  
测试
```
node --version
v4.4.7

npm --version
2.15.8
```

2. 安装grunt
安装grunt命令行工具grunt-cli
```
npm install -g grunt-cli
```
安装grunt及其插件
```
npm install -g grunt --save-dev
```
测试
```
grunt -version
grunt-cli v1.2.0
```


3. 安装elasticsearch-head
如果没有Git先安装
```
yum install git

```
先克隆项目
```
git clone git://github.com/mobz/elasticsearch-head.git

```
安装
```
cd elasticsearch-head

npm install

npm install grunt --save

```
elasticsearch-head的配置文件是Gruntfile.js，默认监听在127.0.0.1下9100端口


然后cd /usr/local/elasticsearch-head  执行
```
grunt server
```

浏览器访问 http://172.16.31.220:9100/


-----
-----
问题：

启动完之后，head主控页面是可以显示的，但是显示连接失败
```
“集群健康值: 未连接”
```
查资料：解决方案，修改elasticsearch.yml文件
```
vim $ES_HOME$/config/elasticsearch.yml
# 增加如下字段
http.cors.enabled: true
http.cors.allow-origin: "*"
```
重启es和head即可  

---

说明：

key | 说明
--| --
http.cors.enabled   |	是否支持跨域，默认为false
http.cors.allow-origin  |	当设置允许跨域，默认为*,表示支持所有域名，如果我们只是允许某些网站能访问，那么可以使用正则表达式。比如只允许本地地址。 /https?:\/\/localhost(:[0-9]+)?/
http.cors.max-age   |	浏览器发送一个“预检”OPTIONS请求，以确定CORS设置。最大年龄定义多久的结果应该缓存。默认为1728000（20天）
http.cors.allow-methods |	允许跨域的请求方式，默认OPTIONS,HEAD,GET,POST,PUT,DELETE
http.cors.allow-headers |	跨域允许设置的头信息，默认为X-Requested-With,Content-Type,Content-Length
http.cors.allow-credentials |	是否返回设置的跨域Access-Control-Allow-Credentials头，如果设置为true,那么会返回给客户端。
