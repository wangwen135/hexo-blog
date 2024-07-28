---
title: git新建项目并推送到远端
date: 2017-06-27 16:52
tags: 
  - git
categories:
  - [版本控制, git]
---


### 命令行指令

#### Git 全局设置
```
git config --global user.name "wwh"
git config --global user.email "wwh@xxxx.com"
```

#### 创建新版本库
```
git clone http://gitlab.xxxx.com:8100/tools/demo.git
cd demo
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master
```

#### 已存在的文件夹
```
cd existing_folder
git init
git remote add origin http://gitlab.xxxx.com:8100/tools/demo.git
git add .
git commit -m "Initial commit"
git push -u origin master
```

#### 已存在的 Git 版本库
```
cd existing_repo
git remote rename origin old-origin
git remote add origin http://gitlab.xxxx.com:8100/tools/demo.git
git push -u origin --all
git push -u origin --tags
```

### 关于认证

#### SSH 协议
需要配置SSH 密钥  
SSH 密钥用于在您的电脑和 GitLab 建立安全连接。

1、先检查本地目录中是否已经有了SSH 秘钥对
```
%userprofile%\.ssh\id_rsa.pub
```
2、如果没有就需要生成一个
```
ssh-keygen -t rsa -C "your.email@example.com" -b 4096

```
3、复制公钥，到 github、gitLab 中进行设置即可

##### 配置了sshkey 还是需要输入密码的问题
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

#### http 协议

需要记住用户名密码

```
git config --global credential.helper store
```
会在 ~/.gitconfig 文件下增加以下内容：  
```
[credential]  
        helper = store
```

查看
```
$ git config --global -l

user.name=wwh
user.email=wwh@xxxx.com
core.autocrlf=false
credential.helper=store
gui.recentrepo=E:/git/xxxx/doc
```

会在home文件下生成 .git-credentials 文件（如无法自动创建需要手动创建），第一次输入完之后就会记录下来

文件内容如下：
```
http://wwh:123456@220.231.228.87%3a8100

# 格式
# https://{username}:{password}@github.com
```
> 注意！是明文存储

