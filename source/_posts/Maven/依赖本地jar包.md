---
title: 依赖本地jar包
date: 2018-08-18 17:51
tags: 
  - Maven
categories:
  - [Maven]
---


## 依赖本地jar包
有的jar包，在maven中心库里面没有，我们一般是放到项目的lib目录下  
如：这里的xxxx.jar
```
├─lib
│  └─xxxx.jar
├─src
│  ├─main
│  │  ├─java
│  │  │  └─com
│  │  │      └─xxx
│  │  └─resources
│  │      └─xxx
│  └─test
│      ├─java
│      └─resources
└─target
``` 

此时在pom.xml里面增加依赖引入该jar  
例如：
```
<dependencies>
	<dependency>
		<groupId>org.xxx</groupId>
		<artifactId>xxx</artifactId>
		<version>1.0</version>
		<scope>system</scope>
		<systemPath>${basedir}/lib/xxxxx.jar</systemPath>
	</dependency>
</dependencies>
```
