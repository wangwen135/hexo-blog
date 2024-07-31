---
title: shell判断命令执行情况
date: 2018-03-08 14:54
tags: 
  - shell
  - linux
categories:
  - [脚本, shell]
---

shell判断命令执行情况


命令执行失败时退出shell

判断
```
if ! command; then echo "command failed"; exit 1; fi
```

判断上一个命令的返回值不为0
```
command
if [ "$?" -ne 0 ]; then echo "command failed"; exit 1; fi
```

逻辑与  
执行多个命令
```
command1  && command2
```
&&左边的命令（命令1）返回真(即返回0，成功被执行）后，&&右边的命令（命令2）才能够被执行；换句话说，“如果这个命令执行成功&&那么执行这个命令”。

逻辑或
```
command1 || command2
```
||则与&&相反。如果||左边的命令（command1）未执行成功，那么就执行||右边的命令（command2）；或者换句话说，“如果这个命令执行失败了||那么就执行这个命令。
```
command || { echo "command failed"; exit 1; }
```