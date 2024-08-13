---
title: Tomcat 安装
date: 2017-05-10 23:41
tags: 
  - Tomcat
categories:
  - [Tomcat]
---

# Tomcat 安装


### 部署
如在CentOS 7上部署Tomcat 8：

```
# 下载
wget http://apache.fayea.com/tomcat/tomcat-8/v8.5.4/bin/apache-tomcat-8.5.4.tar.gz

# 解压
tar -zxvf apache-tomcat-8.5.4.tar.gz 

# 创建一个软连接
ln -s apache-tomcat-8.5.4 tomcat
```


### 配置
##### 修改默认端口
```
vi tomcat/conf/server.xml 
```
    <Connector port="8080" protocol="HTTP/1.1"  connectionTimeout="20000"
        redirectPort="8443" />


#### 配置默认项目
```
vi tomcat/conf/server.xml
```
在Host 标签内加入如下代码段，docBase属性指定项目名称  

    <Context path="" docBase="XXXX"  reloadable="true" crossContext="true">  
    </Context>  


重新启动Tomcat，在浏览器下输入http://localhost:8080，即可看到 **XXXX** 的首页
