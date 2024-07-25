---
title: git rebase 测试
date: 2020-11-03 23:28
tags: 
  - git
categories:
  - [版本控制, git]
---

凑个数  

----


当前在xxx分支上，然后执行rebase命令 master分支
```
git rebase master
```

可以理解为将xxx分支上的改动抽出来弄成一坨，然后将master分支的代码弄到xxx分支这边来，然后再将刚刚弄出来的代码堆上去，就像这一坨代码是接在master分支的末尾上写的一样，这样你再回到master将这一堆代码merge过来，提交日志就不会分叉，看提交日志就特别清晰明了。


如果这一坨代码中有多个提交，不想要这么多的提交记录，就要加一个-i参数
```
git rebase -i B 
```
就会让你挑选你想要的提交



-----

......
不想写了
......

----


从master 检出 dev,然后分别修改 master 和 dev，然后 合并
```
git init

vi aaa.txt

git checkout -b dev

vi bbb.txt

git add .

git commit

git checkout master

vi aaa.txt

git add .

git commit

git checkout dev

git merge master
```

会导致提交日志里面出现分叉的merge信息，比较难看

```
$ git log --graph --pretty=oneline
*   baba7341616a7fd6e6932bf76a90fca9782a8e54 (HEAD -> dev) Merge branch 'master' into dev
|\
| * 3c408871ffcc5cfc37a295ecf936a23aff808c68 (master) master modified aaa.txt
* | a9dc6221ce4b152b292cbcd4e242026edbb7d379 dev branch add bbb.txt file
|/
* 171e52a8cd9c96706cb0112013d556b62932ee39 first commit

```

使用 rebase 后
```
$ git rebase master
```
```
$ git log --oneline --graph
* 6879874 (HEAD -> dev) dev branch add bbb.txt file
* 3c40887 (master) master modified aaa.txt
* 171e52a first commit

```

在 rebase 的过程中，也许会出现冲突 conflict 。在这种情况， git 会停止 rebase 并会让你去解决冲突。在解决完冲突后，用 git add 命令去更新这些内容。

注意，你无需执行 git-commit，只要执行 continue
```
git rebase --continue
```
这样 git 会继续应用余下的 patch 补丁文件。

在任何时候，我们都可以用 --abort 参数来终止 rebase 的行动，并且分支会回到 rebase 开始前的状态。
```
git rebase —abort
```


切回master分支，并进行合并
```
$ git checkout master

$ git merge dev

$ git log --oneline --graph

```



---
---

### rebase有冲突时

处理完冲突后，添加文件到暂存区
```
git add/rm <conflicted_files>
```

然后再继续
```
git rebase --continue
```

也可以跳过这次提交
```
git rebase --skip
```

要中止并回到“git rebase”之前的状态，请运行
```
git rebase --abort

```
