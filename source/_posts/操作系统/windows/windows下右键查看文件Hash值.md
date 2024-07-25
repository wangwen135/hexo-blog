---
title: windows下右键查看文件Hash值
date: 2020-08-31 23:41
tags: 
  - windows
categories:
  - [操作系统, windows]
---

*荣艺给的脚本*

### CMD命令
**certutil -hashfile filename [MD5|SHA1|SHA256]**

示例：
```
C:\Users\Administrator>certutil -hashfile "D:\test\123.txt" MD5
MD5 的 D:\test\123.txt 哈希:
3aed2464da46a2719ff1bf0766d2754f
CertUtil: -hashfile 命令成功完成。
```

### 添加“右键>发送到”快速计算校验值
1. 在文件夹地址栏输入 **shell:sendto** 进入 “发送到” 引用目录  
2. 新建批处理文件（如：checksum.bat），将下面的代码拷贝至文件
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
pause
```

3. 在资源管理器中，右键需要校验的文件，发送至该文件即可  
ps：也可将批处理文件放在别处，创建文件的快捷方式，然后将快捷方式移动到引用目录

效果：
```
File   : D:\tools\bin\clear.bat

MD5    : 98bbdca979f85ee3a491769b89021306
SHA1   : ead014790c1adfdfaf10c5ee701d3e59080a662f
SHA256 : 58969ebe81e0669a2d4a995a9a0c62dcae5a711ea61126ddf8eab868a924112b

请按任意键继续. . .

```
