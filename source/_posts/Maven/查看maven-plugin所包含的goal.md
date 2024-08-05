---
title: 查看maven-plugin所包含的goal
date: 2018-08-18 17:51
tags: 
  - Maven
categories:
  - [Maven]
---



要使用maven插件提供的goal，首先得知道这些插件提供了哪些goal以及他们的用法。

maven的核心插件之一 --- help插件（Maven Help Plugin）可以用于查看插件提供了哪些goal。

```
mvn help:describe -Dplugin=groupId:artifactId:version

```
>可以省略版本号

还可以使用插件目标前缀替换坐标:
```
$ mvn help:describe -Dplugin=compiler
```

#### 例子：
查看 maven-source-plugin 的信息
```
mvn help:describe -Dplugin=org.apache.maven.plugins:maven-source-plugin


......
Name: Apache Maven Source Plugin
Description: The Maven Source Plugin creates a JAR archive of the source
  files of the current project.
Group Id: org.apache.maven.plugins
Artifact Id: maven-source-plugin
Version: 3.0.1
Goal Prefix: source

This plugin has 7 goals:

source:aggregate
  Description: Aggregate sources for all modules in an aggregator project.

source:generated-test-jar
  Description: This plugin bundles all the test sources into a jar archive.

source:help
  Description: Display help information on maven-source-plugin.
    Call mvn source:help -Ddetail=true -Dgoal=<goal-name> to display parameter
    details.

source:jar
  Description: This plugin bundles all the sources into a jar archive.

source:jar-no-fork
  Description: This goal bundles all the sources into a jar archive. This
    goal functions the same as the jar goal but does not fork the build and is
    suitable for attaching to the build lifecycle.

source:test-jar
  Description: This plugin bundles all the test sources into a jar archive.

source:test-jar-no-fork
  Description: This goal bundles all the test sources into a jar archive.
    This goal functions the same as the test-jar goal but does not fork the
    build, and is suitable for attaching to the build lifecycle.

For more information, run 'mvn help:describe [...] -Ddetail'
......

```
