---
title: maven-打包-cannot-access-问题
date: 2023-04-16 23:41
tags: 
  - Maven
categories:
  - [Maven]
---


### 问题
一个项目在开发环境，本地都能打包通过

在测试环境打包过不去

提示 error: cannot access xxxxxxx

### 原因

开发和测试环境用了两套私服

测试环境有人手动上传了jar包，该jar包对应的pom文件，由nexus生成【>POM was created by Sonatype Nexus】，没有dependencies 节点


### 处理
在nexus的 3rd party 中删除该jar，重新上传即可

#### 注意：
如果通过手动上传的方式 GAV Definition 不能选择 GAV Parameters  
应该选择：  
**GAV Definition： from pom**



### 排查问题
检查环境、检查配置，对比jar包等都没有问题

使用 mvn dependency:tree 命令，发现测试环境比开发环境少依赖了几个jar包

然后找的mavne 的 【localRepository】 目录中，按照路径找到对应jar包【根据提示缺少的类和依赖关系】的路径

如：
> /data/repo/com/xxx/xxx/xxxxxxx/0.1.9.RELEASE

这个目录下有xxxx-version.jar和一个对应的xxxx-version.pom文件，问题就是这个pom文件没有dependencies信息

有问题pom文件如：
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.appleframework.jms</groupId>
    <artifactId>apple-jms-kafka</artifactId>
    <version>0.1.9.RELEASE</version>
    <description>POM was created by Sonatype Nexus</description>
</project>
```

正常的应该是：
```
<?xml version="1.0"?>
<project
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"
	xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>com.appleframework.jms</groupId>
		<artifactId>apple-jms</artifactId>
		<version>0.1.9.RELEASE</version>
	</parent>
	<artifactId>apple-jms-kafka</artifactId>
	<name>apple-jms-kafka</name>
	<url>http://mvnrepo.appleframework.com</url>
		
	<dependencies>
	
		<dependency>
			<groupId>com.appleframework.jms</groupId>
			<artifactId>apple-jms-core</artifactId>
			<version>${project.version}</version>
		</dependency>

		<dependency>
			<groupId>org.apache.kafka</groupId>
			<artifactId>kafka_2.11</artifactId>
			<version>0.11.0.3</version>
			<scope>provided</scope>
		</dependency>
		
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
			<scope>provided</scope>
		</dependency>
		
		<dependency>
  			<groupId>com.google.code.gson</groupId>
  			<artifactId>gson</artifactId>
  			<version>2.8.5</version>
  			<scope>provided</scope>
		</dependency>
		
	</dependencies>
</project>

```

