---
title: Maven属性（properties）
date: 2021-05-20 23:41
tags: 
  - Maven
categories:
  - [Maven]
---

*王某某 2017年2月*

通过<properties>元素用户可以自定义一个或多个Maven属性，然后在POM的其他地方使用${属性名}的方式引用该属性，这种做法的最大意义在于消除重复和统一管理。

　　Maven总共有6类属性，内置属性、POM属性、自定义属性、Settings属性、java系统属性和环境变量属性；

1. 内置属性  
两个常用内置属性 ${basedir} 表示项目跟目录，即包含pom.xml文件的目录；${version} 表示项目版本

2. POM属性  
用户可以使用该类属性引用POM文件中对应元素的值。如${project.artifactId}就对应了<project> <artifactId>元素的值，常用的POM属性包括：

- ${project.build.sourceDirectory}:项目的主源码目录，默认为src/main/java/
- ${project.build.testSourceDirectory}:项目的测试源码目录，默认为src/test/java/
- ${project.build.directory} ： 项目构建输出目录，默认为target/
- ${project.outputDirectory} : 项目主代码编译输出目录，默认为target/classes/
- ${project.testOutputDirectory}：项目测试主代码输出目录，默认为target/testclasses/
- ${project.groupId}：项目的groupId
- ${project.artifactId}：项目的artifactId
- ${project.version}：项目的version,与${version} 等价
- ${project.build.finalName}：项目打包输出文件的名称，默认为${project.artifactId}-${project.version}

3. 自定义属性  
随便写

4. Settings属性  
与POM属性同理，用户使用以settings. 开头的属性引用settings.xml文件中的XML元素的值。

5. Java系统属性  
所有java系统属性都可以用Maven属性引用，如${user.home}指向了用户目录。

6. 环境变量属性  
所有环境变量属性都可以使用以env. 开头的Maven属性引用，如${env.JAVA_HOME}指代了JAVA_HOME环境变量的的值。
 
