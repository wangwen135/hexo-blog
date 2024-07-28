---
title: 一些实用的bat脚本
date: 2018-09-27 9:31
tags: 
  - bat
  - Windows
categories:
  - [脚本, bat]
---


在windows 和 linux的命令行来回切换，经常敲错命令，如：ls、pwd、grep 等

通过写bat的办法让windows支持这几个常用命令

找一个目录，将其添加到环境变量中，在目录中新建几个bat文件，如下：

### ll.bat
```
@echo off
REM 显示一个目录中的文件和子目录
dir
```

### pwd.bat
```
@echo off
REM 显示当前目录的名称或将其更改。
chdir
```

### ls.bat
```
@echo off
REM 显示一个目录中的文件和子目录 /W 用宽列表格式

dir /W
```

### grep.bat
```
@echo off
REM 模拟grep
REM 在文件中搜索字符串。
REM FIND [/V] [/C] [/N] [/I] [/OFF[LINE]] "string" [[drive:][path]filename[ ...]]
REM /I  搜索字符串时忽略大小写。


find /I "%1"

```

### clear.bat
```
@echo off
REM 清除屏幕

cls
```

### checksum.bat
```
@echo OFF
:LOOP
    set index=%1
    if %index%! == ! goto END
    echo.
    echo File   : %index%
    echo.
    set /p="MD5    : "<nul
    certutil -hashfile "%index%" MD5|findstr /V "MD5"|findstr /V "CertUtil"
    set /p="SHA1   : "<nul
    certutil -hashfile "%index%" SHA1|findstr /V "SHA1"|findstr /V "CertUtil"
    set /p="SHA256 : "<nul
    certutil -hashfile "%index%" SHA256|findstr /V "SHA256"|findstr /V "CertUtil"
    shift
    goto LOOP
:END
echo.
```

### exp.bat
```
@echo off
REM 资源管理器打开当前目录

explorer /select=%cd%
```

### sp.bat
```
@echo off
REM 打开windows截图工具

start snippingtool
```
> win10快捷键 win + shift + s

