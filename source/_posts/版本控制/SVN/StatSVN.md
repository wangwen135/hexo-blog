---
title: StatSVN
date: 2017-05-05 15:35
tags: 
  - SVN
  - StatSVN
categories:
  - [版本控制, SVN]
---


## StatSVN介绍
StatSVN是一个Java写的开源代码统计程序，从statCVS移植而来，能够从Subversion版本库中取得信息，然后生成描述项目开发的各种表格和图表。比如：代码行数的时间线；针对每个开发者的代码行数；开发者的活跃程度；开发者最近所提交的；文件数量；平均文件大小；最大文件；哪个文件是修改最多次数的；目录大小；带有文件数量和代码行数的Repository tree。StatSVN当前版本能够生成一组包括表格与图表的静态HTML文档。

## StatSVN 使用条件
如前所述，StatSVN是一个Java写的开源代码统计程序，是从Subversion版本库中取得信息的，所以使用StatSVN有两个限制。
1. 需要安装Java的运行环境（Java Runtime Environment）
2. 需要使用svn客户端，必须保证本机的svn客户端命令可用

## StatSVN 使用方法
使用之前需要先下载StatSVN：http://www.statsvn.org/downloads.html

### checkout 工作目录
将需要统计的svn路径下的代码checkout到本地工作目录里，版本可以自由选择，如果你要统计某个版本下的代码量checkout出对应的版本即可，如果需要统计最近的版本时的代码量，checkout最新版本。

### 生成log文件
使用StatSVN统计代码量时需要使用log文件，生成log文件方法：

命令行下进入工作目录后：svn log -v –xml > logfile.log

### 使用StatSVN统计SVN中的代码量
将下载好的StatSVN解压，得到statsvn.jar文件,在命令行里执行命令
```
java -jar statsvn.jar C:\project\logfile.log C:\project
```
这里的C:\project\logfile.log是前一步生成的log文件，C:\project是工作目录。

执行完后，就在当前目录下生成了对应的html结果文档。

### 命令介绍
格式：
```
java -jar statsvn.jar [options] <logfile> <checked-out-module>

```
参数
```
<logfile>
```
为前一步中生成的svn log文件
```
<checked-out-module>
```
为checkout工作拷贝目录，注意两个参数都要列出正确的全路径，否则会提示错误如logfile.log找不到等等


## 实际上使用的SH脚本

```
#!/bin/sh

#
# 用于统计深暗项目的代码情况
#

#进入SVN检出目录
cd /data/statsvn/project/xxx/xxx-parent


#SVN更新
svn update

#生成log文件
svn log -v --xml > ../xxx-svn.log

#/data/statsvn/project/xxx/xxx-svn.log
#/data/statsvn/statsvn-0.7.0/statsvn.jar

#进入到输出目录
cd /data/statsvn/report/xxx

#删除全部内容
rm -rf *


#生成html文件


#全部后台相关
#java -jar /data/statsvn/statsvn-0.7.0/statsvn.jar /data/statsvn/project/xxx/xxx-svn.log /data/statsvn/project/xxx/xxx-parent -include **/*.java:**/*.xml:**/*.properties:**/*.conf


#包括前后台的
java -jar /data/statsvn/statsvn-0.7.0/statsvn.jar /data/statsvn/project/xxx/xxx-svn.log /data/statsvn/project/xxx/xxx-parent -exclude **/jquery*.js:**/bootstrap*.css:**/bootstrap*.js:**/d3.*:**/echarts.*:**
/webapp/fonts/*

```

配合Apache，将该脚本定时执行就能得到最新的统计结果
