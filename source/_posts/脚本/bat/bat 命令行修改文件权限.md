---
title: bat 命令行修改文件权限
date: 2019-03-14 23:41
tags: 
  - bat
  - Windows
categories:
  - [脚本, bat]
---



```

SET PATH=G:\360WIFI
Takeown /F %PATH% /r /d y
cacls %PATH% /t /e /g Administrators:F
rd /s /q %PATH%
@pause
```


实际测试情况：
```
C:\Windows\SysWOW64\Macromed\Flash>del /F activex.vch
C:\Windows\SysWOW64\Macromed\Flash\activex.vch
拒绝访问。

C:\Windows\SysWOW64\Macromed\Flash>takeown /F activex.vch

成功: 此文件(或文件夹): "C:\Windows\SysWOW64\Macromed\Flash\activex.vch" 现在由用户 "WWH\wangw" 所有。

C:\Windows\SysWOW64\Macromed\Flash>cacls activex.vch /e /g "WWH\wangw":F
处理的文件: C:\Windows\SysWOW64\Macromed\Flash\activex.vch

C:\Windows\SysWOW64\Macromed\Flash>del activex.vch

C:\Windows\SysWOW64\Macromed\Flash>

```

删除目录测试：
```

C:\Windows\SysWOW64\Macromed>rd /s Flash
Flash, 是否确认(Y/N)? y
Flash\Flash.ocx - 拒绝访问。
Flash\FlashUtil_ActiveX.dll - 拒绝访问。
Flash\FlashUtil_ActiveX.exe - 拒绝访问。

C:\Windows\SysWOW64\Macromed>takeown /F Flash /r

成功: 此文件(或文件夹): "C:\Windows\SysWOW64\Macromed\Flash" 现在由用户 "WWH\wangw" 所有。

成功: 此文件(或文件夹): "C:\Windows\SysWOW64\Macromed\Flash\Flash.ocx" 现在由用户 "WWH\wangw" 所有。

成功: 此文件(或文件夹): "C:\Windows\SysWOW64\Macromed\Flash\FlashUtil_ActiveX.dll" 现在由用户 "WWH\wangw" 所有。

成功: 此文件(或文件夹): "C:\Windows\SysWOW64\Macromed\Flash\FlashUtil_ActiveX.exe" 现在由用户 "WWH\wangw" 所有。

C:\Windows\SysWOW64\Macromed>cacls Flash /t /e /g "WWH\wangw":F
处理的目录: C:\Windows\SysWOW64\Macromed\Flash
处理的文件: C:\Windows\SysWOW64\Macromed\Flash\Flash.ocx
处理的文件: C:\Windows\SysWOW64\Macromed\Flash\FlashUtil_ActiveX.dll
处理的文件: C:\Windows\SysWOW64\Macromed\Flash\FlashUtil_ActiveX.exe

C:\Windows\SysWOW64\Macromed>rd /s Flash
Flash, 是否确认(Y/N)? y

C:\Windows\SysWOW64\Macromed>
```
