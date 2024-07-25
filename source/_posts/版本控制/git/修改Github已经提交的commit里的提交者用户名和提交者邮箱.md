---
title: 修改Github已经提交的commit里的提交者用户名和提交者邮箱
date: 2021-07-06 16:56
tags: 
  - git
  - github
categories:
  - [版本控制, git]
---


由于某些原因需要修改已经提交的代码里面的提交者名称和提交者邮箱

### git filter-branch
https://git-scm.com/docs/git-filter-branch

git-filter-branch - 重写分支



#### 描述（翻译的） 
允许您通过重写 <rev-list options> 中提到的分支来重写 Git 修订历史记录，对每个修订应用自定义过滤器。这些过滤器可以修改每个树（例如删除一个文件或在所有文件上运行 perl 重写）或有关每个提交的信息。否则，将保留所有信息（包括原始提交时间或合并信息）。

该命令只会重写命令行中提到的正引用（例如，如果您传递a..b，则只会重写b）。如果您未指定过滤器，则将重新提交提交而不进行任何更改，这通常不会产生任何影响。尽管如此，这在将来可能对补偿某些 Git 错误等有用，因此允许这种用法。

**注意：** 此命令尊重命名空间.git/info/grafts中的文件和引用refs/replace/。如果您定义了任何移植或替换引用，运行此命令将使它们永久化。

**警告！** 重写后的历史将对所有对象具有不同的对象名称，并且不会与原始分支收敛。您将无法轻松地在原始分支之上推送和分发重写的分支。如果您不知道全部含义，请不要使用此命令，并且如果简单的单次提交就足以解决您的问题，请避免使用它。（有关重写已发布历史记录的更多信息，请参阅git-rebase[1] 中的“从上游重新数据库恢复”部分。）

### 操作

#### 1、找个地方下载代码
```
cd /xxx/tmp
git clone git@github.com:xxxxx/xxxxxxx.git
```

#### 2、修改提交信息

在项目目录下，随便创建一个xxx.sh，内容如下

**修改下面的邮箱和用户名**
```
git filter-branch --env-filter '
WRONG_EMAIL="wrong@example.com"
NEW_NAME="New Name Value"
NEW_EMAIL="correct@example.com"

if [ "$GIT_COMMITTER_EMAIL" = "$WRONG_EMAIL" ]
then
    export GIT_COMMITTER_NAME="$NEW_NAME"
    export GIT_COMMITTER_EMAIL="$NEW_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$WRONG_EMAIL" ]
then
    export GIT_AUTHOR_NAME="$NEW_NAME"
    export GIT_AUTHOR_EMAIL="$NEW_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags
```
加执行权限
```
chmod +x xxx.sh
```
然后执行
```
./xxx.sh
```

#### 3、检查
```
git log
```
看看提交人和提交邮箱是不是改了


#### 4、强制提交
```
git push --force
```

----

#### 多个分支时
切换到不同的分支执行xxx.sh  
然后强行提交

如果提示：A previous backup already exists in refs/original/

就改改脚本，增加一个 -f 参数，强制执行
```
git filter-branch -f --env-filter '
...
...
```

---
---


### 之前已经有的本地代码

#### 1、 删掉重新下载
整个项目目录删掉，重新clone 代码

#### 2、强制更新为远端的版本
```
git fetch --all
git reset --hard origin/master
```

#### 3、 删掉分支再重新检出分支
```
git branch -d xxxxx

git fetch --all
git branch -a

git checkout xxx
```


