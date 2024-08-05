---
title: Maven下载构建测试
date: 2020-05-11 10:45
tags: 
  - Maven
categories:
  - [Maven]
---


随意写一个Maven项目，依赖一个**不存在**的构建时的测试记录

使用 mvn compile 命令测试

-----

## 没有配置镜像的测试
Maven 的 settings.xml 中没有配置<mirror>

### 一、没有配置repository
使用默认的中央仓库

#### 依赖snapshot版本的构建
直接拒绝，因为中央仓库不支持snapshot，不会连到中央仓库
```
[ERROR] Failed to execute goal on project csv2excel: Could not resolve dependencies for project com.wwh.csv2excel:csv2excel:jar:0.1-SNAPSHOT: Could not find artifact com.wwh.csv2excel:csv2excel:jar:0.7-SNAPSHOT -> [Help 1]
```
#### 依赖非snapshot版本的构建
尝试去中央服务器下载，然后提示不存在
```
[ERROR] Failed to execute goal on project csv2excel: Could not resolve dependencies for project com.wwh.csv2excel:csv2excel:jar:0.1-SNAPSHOT: Could not find artifact com.wwh.csv2excel:csv2excel:jar:0.8 in central (https://repo.maven.apache.org/maven2) -> [Help 1]
```

### 二、 配置了一个repository，且支持snapshot
```
<repositories>
	<repository>
		<id>nexusRep</id>
		<url>http://nexus.vvsmart.com:8081/nexus/content/groups/public</url>
		<releases>
			<enabled>true</enabled>
		</releases>
		<snapshots>
			<enabled>true</enabled>
		</snapshots>
	</repository>
</repositories>
```

#### 依赖snapshot版本的构建
尝试去配置的仓库下载，然后提示不存在
```
[ERROR] Failed to execute goal on project csv2excel: Could not resolve dependencies for project com.wwh.csv2excel:csv2excel:jar:0.1-SNAPSHOT: Could not find artifact com.wwh.csv2excel:csv2excel:jar:1.2-SNAPSHOT in nexusRep (http://nexus.vvsmart.com:8081/nexus/content/groups/public) -> [Help 1]
```

#### 依赖非snapshot版本的构建
先从配置的仓库下载，再尝试去中央仓库下载，然后提示无法解析
```
Downloading from nexusRep: http://nexus.vvsmart.com:8081/nexus/content/groups/public/com/wwh/csv2excel/csv2excel/1.4/csv2excel-1.4.pom
Downloading from central: https://repo.maven.apache.org/maven2/com/wwh/csv2excel/csv2excel/1.4/csv2excel-1.4.pom
[WARNING] The POM for com.wwh.csv2excel:csv2excel:jar:1.4 is missing, no dependency information available
Downloading from nexusRep: http://nexus.vvsmart.com:8081/nexus/content/groups/public/com/wwh/csv2excel/csv2excel/1.4/csv2excel-1.4.jar
Downloading from central: https://repo.maven.apache.org/maven2/com/wwh/csv2excel/csv2excel/1.4/csv2excel-1.4.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 11.304 s
[INFO] Finished at: 2020-05-09T21:02:10+08:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal on project csv2excel: Could not resolve dependencies for project com.wwh.csv2excel:csv2excel:jar:0.1-SNAPSHOT: Could not find artifact com.wwh.csv2excel:csv2excel:jar:1.4 in nexusRep (http://nexus.vvsmart.com:8081/nexus/content/groups/public) -> [Help 1]

```

### 三、 配置了多个repository，且支持snapshot
配置三个repository，地址实际上是不存在的，这里只验证下载构建的顺序
```
[INFO] --------------------------------[ jar ]---------------------------------
Downloading from nexusRep: http://nexus.vvsmart.com:8081/nexus/content/groups/public/com/wwh/csv2excel/csv2excel/1.5/csv2excel-1.5.pom
Downloading from aaa: http://192.168.0.11:8081/nexus/content/groups/public/com/wwh/csv2excel/csv2excel/1.5/csv2excel-1.5.pom
Downloading from bbb: http://192.168.0.12:8081/nexus/content/groups/public/com/wwh/csv2excel/csv2excel/1.5/csv2excel-1.5.pom
Downloading from central: https://repo.maven.apache.org/maven2/com/wwh/csv2excel/csv2excel/1.5/csv2excel-1.5.pom

```


### 结论
如果repository不支持snapshot的构建，则不会去这个仓库下载  
优先使用配置的repository，按照配置的顺序进行下载，最后再用mavne的中央仓库  


------

## 配置了镜像的测试
```
<mirror>
    <id>nexus</id>
    <mirrorOf>*</mirrorOf>
    <url>http://nexus.vvsmart.com:8081/nexus/content/groups/public</url>
</mirror>
```
一个地址镜像全部仓库

### 一、没有配置repository

#### 依赖snapshot版本的构建
默认走到了中央仓库，这个直接被拒绝了，因为没有开启snapshot，提示无法解析，不会连接到镜像服务器
```
[ERROR] Failed to execute goal on project csv2excel: Could not resolve dependencies for project com.wwh.csv2excel:csv2excel:jar:0.1-SNAPSHOT: Could not find artifact com.wwh.csv2excel:csv2excel:jar:0.4-SNAPSHOT -> [Help 1]
```
#### 依赖非snapshot版本的构建
会尝试去镜像地址下载，然后提示不存在
```
[ERROR] Failed to execute goal on project csv2excel: Could not resolve dependencies for project com.wwh.csv2excel:csv2excel:jar:0.1-SNAPSHOT: Could not find artifact com.wwh.csv2excel:csv2excel:jar:0.3 in nexus (http://nexus.vvsmart.com:8081/nexus/content/groups/public) -> [Help 1]
```

### 二、 配置了一个repository，且支持snapshot
都会连接到镜像地址，然后提示不存在

#### repository的id的影响

##### repository的id为central时
依赖一个不存在的snapshot版本的构建时，尝试去镜像地址下载
```
[ERROR] Failed to execute goal on project csv2excel: Could not resolve dependencies for project com.wwh.csv2excel:csv2excel:jar:0.1-SNAPSHOT: Could not find artifact com.wwh.csv2excel:csv2excel:jar:0.5-SNAPSHOT in nexus (http://nexus.vvsmart.com:8081/nexus/content/groups/public) -> [Help 1]
```

##### repository的id为test时
依赖一个不存在的snapshot版本的构建时，尝试去镜像地址下载
```
[ERROR] Failed to execute goal on project csv2excel: Could not resolve dependencies for project com.wwh.csv2excel:csv2excel:jar:0.1-SNAPSHOT: Could not find artifact com.wwh.csv2excel:csv2excel:jar:0.6-SNAPSHOT in nexus (http://nexus.vvsmart.com:8081/nexus/content/groups/public) -> [Help 1]
```

### 结论
Maven也会先判断本地的repository是否支持snapshot，再决定是否需要去该仓库下载，再连到镜像服务器  

是否需要将repository的id设置为central（ 为了覆盖超级POM中央仓库的配置）并不重要，反正都会走的镜像服务器中，只要有任意一个仓库支持snapshot就可以了

且配置了镜像之后，repository的Url并不重要，反正不会用到


