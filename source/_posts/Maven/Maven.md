---
title: Maven
date: 2023-12-27 23:41
tags: 
  - Maven
categories:
  - [Maven]
---

*王某某 2016年8月*

Maven项目对象模型(POM)，可以通过一小段描述信息来管理项目的构建，报告和文档的软件项目管理工具。  
Maven是一个用于项目构建的工具，通过它便捷的管理项目的生命周期。即项目的jar包依赖，开发，测试，发布打包。

## 安装

*以下是以Linux为例，Windows下配置也基本相同。*

#### 1. 下载
```
wget http://mirrors.cnnic.cn/apache/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
```

#### 2. 解压
```
tar -zxvf apache-maven-3.3.9-bin.tar.gz 
```

如需使用软连接  
```
ln -s apache-maven-3.3.9 maven
```

#### 3. 修改配置文件
本地仓库默认是**用户目录**下的`.m2\repository`，用户级的配置文件`settings.xml`也放在这里。

全局配置修改：
```
vi conf/settings.xml
```
```
#设置本地资源库位置
<localRepository>/home/maven/.m2/repository</localRepository>
```

#### 4. 配置环境变量
添加环境变量
```
vi /etc/profile
```
```
export M2_HOME=/home/maven/maven
export PATH=$PATH:$M2_HOME/bin
```

使环境变量生效  
```
source /etc/profile
```

#### 5. 验证是否安装成功  
在其他目录执行
```
mvn -version
```
----

## 常用命令
1. `mvn archetype:create`   ：创建 Maven 项目
2. `mvn compile`   ：编译源代码
3. `mvn test-compile`   ：编译测试代码
4. `mvn test`   ： 运行应用程序中的单元测试
5. `mvn site`   ： 生成项目相关信息的网站
6. `mvn clean`   ：清除目标目录中的生成结果
7. `mvn package`   ： 依据项目生成 jar 文件
8. `mvn install`   ：在本地 Repository 中安装 jar
9. `mvn eclipse:eclipse`   ：生成 Eclipse 项目文件
10. `mvn idea:idea `   : 生成idea项目
11. `mvn jar:jar`   : 只打jar包
12. `mvn eclipse:clean`   : 清除eclipse的一些系统设置
13. `mvn jetty:run`   : 运行项目于jetty上
14. `mvn -DskipTests=true`   : 忽略测试文档编译

----

## 关于仓库

#### 本地仓库
本地仓库默认是用户目录下的.m2\repository，用户可以通过settings.xml设置  

#### 远程仓库
maven的远程仓库有多种存在形式，中央仓库，其他远程仓库，镜像，私服

##### 1. 中央仓库
中央仓库是默认的远程仓库，如果不做任何特殊配置那么将会从中央仓库下载依赖，这在 
`$M2_HOME/lib/maven-model-builder-3.0.4.jar`里的`org/apache/maven/model/pom-4.0.0.xml`里做了指定，如下：

```
<repositories>
    <repository>
        <id>central</id>
        <name>Central Repository</name>
        <url>http://repo.maven.apache.org/maven2</url>
        <layout>default</layout>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
</repositories>
```

##### 2. 镜像
镜像是将的maven依赖请求转发至相应服务器，配置如下：
```
<mirror>
    <id>mirrorId</id>
    <mirrorOf>repositoryId</mirrorOf>
    <name>Human Readable Name for this Mirror.</name>
    <url>http://my.repository.com/repo/path</url>
</mirror>
```

这里即是将仓库id为`repositoryId`的所有请求转发至`http://my.repository.com/repo/path`镜像服务器  
镜像更为常用的作法是结合私服，如下配置：
```
<mirror>
    <id>mirrorId</id>
    <mirrorOf>*</mirrorOf>
    <name>Human Readable Name for this Mirror.</name>
    <url>http://my.repository.com/repo/path</url>
</mirror>
```
这里即是将对所有仓库的请求转发至私服`http://my.repository.com/repo/path`

##### 3. 私服
私服一般采用nexus部署

----

### 添加其他远程仓库
添加一个其他的远程库只需在项目的pom.xml文件中添加以下配置即可
```
<repositories>
    <repository>
        <id>jboss</id>
        <name>JBoss Repository</name>
        <url>http://repository.jboss.org/nexus/content/groups/public/</url>
        <snapshots>
        <enabled>false</enabled>
        </snapshots>
        <releases>
            <enabled>true</enabled>
        </releases>
        <layout>default</layout>
    </repository>
</repositories>
```

- snapshots false表示关闭jboss远程仓库的snapshots版本下载 
- releases true表示打开jboss远程仓库的release版本下载

### 远程服务器的验证
远程服务器的验证，只需在settings.xml添加如下代码即可
```
<server>
    <id>deploymentRepo</id>
    <username>repouser</username>
    <password>repopwd</password>
</server>
```
**注意**：id要与配置的远程服务器id对应比如 jboss

