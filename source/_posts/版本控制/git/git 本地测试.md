---
title: git 本地测试
date: 2020-11-03 23:21
tags: 
  - git
categories:
  - [版本控制, git]
---






在window平台安装了git之后，想模拟从远程仓库拉取文件推送文件等，测试merge、rebase等


### 创建本地git仓库
随便找一个目录作为 repository

在gitBash 中执行 初始化git仓库的命令，顺便也可以加一个文件

```
git init

vim aaa.txt

git add .

git commit -m "add a file"
```

### 将仓库中克隆到本地其他目录

在其他的目录中执行 git clone file:xxxx

```
git clone file file:///d/test/repository/git1/.git/
```


### 修改文件之后，push到仓库中

```
git pull

git checkout -b dev

vim bbb.txt

git add .

git commit

git push

# 需要指定远程的分支
git push --set-upstream origin dev

```



### 当不能push时，

切换到master分支，推一个文件

```
$ git push
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 12 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 297 bytes | 297.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
remote: error: refusing to update checked out branch: refs/heads/dev
remote: error: By default, updating the current branch in a non-bare repository
remote: is denied, because it will make the index and work tree inconsistent
remote: with what you pushed, and will require 'git reset --hard' to match
remote: the work tree to HEAD.
remote:
remote: You can set the 'receive.denyCurrentBranch' configuration variable
remote: to 'ignore' or 'warn' in the remote repository to allow pushing into
remote: its current branch; however, this is not recommended unless you
remote: arranged to update its work tree to match what you pushed in some
remote: other way.
remote:
remote: To squelch this message and still keep the default behaviour, set
remote: 'receive.denyCurrentBranch' configuration variable to 'refuse'.
To file:///d/test/repository/git1/.git/
 ! [remote rejected] dev -> dev (branch is currently checked out)
error: failed to push some refs to 'file:///d/test/repository/git1/.git/'

```

这是由于git默认拒绝了push操作，需要进行设置，可以在服务器（就是本机）执行
```
$ git config receive.denyCurrentBranch ignore
```
