---
title: git相关配置
date: 2017-16-26 23:41
tags: 
  - git
categories:
  - [版本控制, git]
---



### .gitconfig 文件
Git的设置文件为.gitconfig，它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）。

```
# 显示当前的Git配置
$ git config --list

# 编辑Git配置文件
$ git config -e [--global]

```

### 设置提交代码时的用户信息
```
$ git config [--global] user.name "[name]"
$ git config [--global] user.email "[email address]"

```
```
git config --global user.name "wwh"
git config --global user.email "wangwen135@163.com"

```


### 自动转换回车换行的问题  
```
git config --global core.autocrlf false
```
通过全局设置将其关闭，默认是关闭的


### 建议使用UTF-8编码，和Unix 风格的界定符

Eclipse 上的配置方法：

>windows --> preferences --> general --> Workspace
```
Text file encoding = UTF-8

New text file line delimiter = Unix
```


### git bash 下log 中文乱码的问题
在窗口上右键 > Option > Text > zh_CN > UTF-8
```
git config --global gui.encoding utf-8
git config --global i18n.commitencoding utf-8
git config --global i18n.logoutputencoding utf-8
```

### 配置了sshkey 还是需要输入密码的问题
检查项目的 .git/config 文件

```
[remote "origin"]
        url = 192.168.1.91:/data/git/dap.git
```
修改为使用git协议：
```
[remote "origin"]
        url = git@192.168.1.91:/data/git/dap.git
```


### 使用HTTP协议时自动记住用户名密码
```
git config --global credential.helper store

# 
# 会在 .gitconfig 文件下增加以下内容：
[credential]
        helper = store
```
会在home文件下生成 .git-credentials 文件（如无法自动创建需要手动创建），第一次输入完之后就会记录下来

如：
```
http://wwh:123456@220.231.228.87%3a8100

# 格式
# https://{username}:{password}@github.com
```

### 生成ssh key

```
ssh-keygen -t rsa -C "wangwen135@163.com"  

```

---

---

### 差异比较工具（用开发工具吧，这个不好使）
在合并代码的是经常会出现冲突，此时就需要一个解决冲突的图形画工具

二进制文件一般没法合并，直接选择使用哪边的
```
$ git checkout - -ours a.file
或者
$ git checkout - -theirs a.file
```

#### git mergetool

```
git mergetool

This message is displayed because 'merge.tool' is not configured.
See 'git mergetool --tool-help' or 'git help config' for more details.
'git mergetool' will now attempt to use one of the following tools:
opendiff kdiff3 tkdiff xxdiff meld tortoisemerge gvimdiff diffuse diffmerge ecmerge p4merge araxis bc codecompare emerge vimdiff

```
这里安装 kdiff3  
https://sourceforge.net/projects/kdiff3/files/

配置 **mergetool.kdiff3.path** 来设置kdif3的绝对路径
>如果将 **kdiff3** 添加到Path中，则不需要配置
```
git config --global mergetool.kdiff3.path "D:/Program Files/KDiff3/kdiff3.exe"

```

配置全局默认的merge tool 为 kdiff3
```
git config --global merge.tool kdiff3


#退出后询问
git config --global mergetool.kdiff3.trustExitCode false


#让git mergetool不再生成备份文件(*.orig)  
git config --global mergetool.keepBackup false


```

在有冲突时候使用 git mergetool
```
Administrator@wwh MINGW64 /d/git/tmp/sample (wwh|MERGING)
$ git mergetool
Merging:
merger.txt

Normal merge conflict for 'merger.txt':
  {local}: modified file
  {remote}: modified file

```


##### git difftool

```
git config --global diff.guitool kdiff3
git config --global difftool.kdiff3.path "D:/Program Files/KDiff3/kdiff3.exe"
git config --global difftool.kdiff3.trustExitCode false

```
