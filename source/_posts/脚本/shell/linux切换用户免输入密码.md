---
title: linux切换用户免输入密码
date: 2021-09-27
tags: 
  - shell
  - linux
  - expect
categories:
  - [脚本, shell]
---


### expect - 自动交互脚本


### 选项
- -c:执行脚本前先执行的命令，可多次使用。
- -d:debug模式，可以在运行时输出一些诊断信息，与在脚本开始处使用exp_internal 1相似。
- -D:启用交换调式器,可设一整数参数。
- -f:从文件读取命令，仅用于使用#!时。如果文件名为"-"，则从stdin读取(使用"./-"从文件名为-的文件读取)。
- -i:交互式输入命令，使用"exit"或"EOF"退出输入状态。
- --:标示选项结束(如果你需要传递与expect选项相似的参数给脚本时)，可放到#!行:#!/usr/bin/expect --。
- -v:显示expect版本信息。
 
### expect常用命令
```
spawn               交互程序开始后面跟命令或者指定程序
expect              获取匹配信息匹配成功则执行expect后面的程序动作
send exp_send       用于发送指定的字符串信息
exp_continue        在expect中多次匹配就需要用到
send_user           用来打印输出 相当于shell中的echo
exit                退出expect脚本
eof                 expect执行结束 退出
set                 定义变量
puts                输出变量
set timeout         设置超时时间
```


### 实践步骤

#### 先安装 expect
```
yum install expect
 
#查看一下版本
expect -v
 ```
 
 
 
#### 编写shell脚本
 
 
 
 ```
#!/usr/bin/expect
# 获取第1个参数，从0开始
set username [lindex $argv 0]

# 设置超时
set timeout 5

# spawn是expect内部命令
spawn su - $username

# 判断上次输出结果里是否包含“password:”的字符串，如果有则立即返回，否则就等待一段时间(timeout)后返回
# expect "Password:"
# 有时候会提示中文的密码，这里改成通配符，? 表示1个字符 * 表示零个或多个字符

expect "??*"

#此处输入密码，\r回车
send "123456\r" 

# 执行完成后保持交互状态，控制权交给控制台(手工操作)。否则会完成后会退出。
interact

 ```
 
 