---
title: windows 杀占用端口的进程
date: 2019-08-27 9:52
tags: 
  - bat
  - Windows
categories:
  - [脚本, bat]
---


#### 先找到占用端口的进程号
```
C:\Users\Administrator>netstat -ano  | grep 8080
  TCP    0.0.0.0:8080           0.0.0.0:0              LISTENING       10288
  TCP    [::]:8080              [::]:0                 LISTENING       10288
```
grep 命令原生是没有的，通过写批处理命令的方式支持的，可以用findstr命令
```
>netstat -ano  | findstr 8080
```

#### 杀进程
```
C:\Users\Administrator>taskkill -PID 10288
错误: 无法终止 PID 为 10288 的进程。
原因: 只能强行终止这个进程(带 /F 选项)。
```

#### 强制杀进程
```
C:\Users\Administrator>taskkill /F -PID 10288
成功: 已终止 PID 为 10288 的进程。
```

