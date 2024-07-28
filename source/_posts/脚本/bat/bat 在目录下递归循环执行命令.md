---
title: bat 在目录下递归循环执行命令
date: 2018-02-15 12:01
tags: 
  - bat
  - Windows
categories:
  - [脚本, bat]
---

```
@echo off

rem 想用bat 做一个递归清理文件
rem 目前就是递归执行一下 maven的清理命令
rem 语法真的是渣

rem echo 当前盘符：%~d0
rem echo 当前盘符和路径：%~dp0
rem echo 当前批处理全路径：%~f0
rem echo 当前盘符和路径的短文件名格式：%~sdp0
rem echo 当前CMD默认目录：%cd%

echo ************************************
echo ************ 清理数据 **************
echo ************************************

IF "%1"=="" (
  echo 清理目录:%cd%
  set prodir=%cd%
) ELSE (
  echo 当前参数参数是：[%1]
  echo 清理目录：%1
  set prodir=%1
  if exist %1/ (
	cd %1
  ) else (
  	echo 目录:[%1]不存在，即将退出！
	pause
	exit
  )
)


rem echo 路径：
rem chdir


for /D %%s in (*) do (
  
  echo 当前循环的值是:%%s
  

  REM 跳过git目录
 
  IF %%s==.git (
	echo 这是一个git目录，跳过
	
  ) ELSE (
	  echo 进入目录:%%s
	  cd %%s	  
	  REM echo 判断pom文件是否存在
	  REM dir
	  
	   IF EXIST pom.xml (
		 echo 存在pom文件，开始执行清理命令...
		 
		 mvn clean
		 
		 @echo off
		 
	   ) ELSE (
		 echo 不存在pom文件，进入下一层目录
		 REM goto BB
		 REM call %~f0 %cd%
		 call %~f0
		 
		 echo 子目录处理完成
		 
	   )
	  
	  echo 退出目录:%%s
	  
	  cd .. 
  )

  echo 本次循环结束，循环值是：%%s
  echo ============================================================

)

echo 执行完成

```


