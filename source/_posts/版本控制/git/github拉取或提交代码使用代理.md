---
title: github拉取或提交代码使用代理
date: 2021-07-02 14:32
tags: 
  - git
categories:
  - [版本控制, git]
---


### http协议代理
> windows下测试的，Git for Windows 原生只支持http 和 https代理

使用**HTTPS**的方式克隆代码
```
git clone https://github.com/xxx/xxx.git
```

#### 使用socks5代理
```
git config --local http.proxy 'socks5://127.0.0.1:1080'

git config --local https.proxy 'socks5://127.0.0.1:1080'

```

#### 使用http 代理
```
git config --local https.proxy http://127.0.0.1:8080

git config --local https.proxy https://127.0.0.1:8080

```

#### 全局或单个项目配置
全局配置用 **git config --global http.proxy......**  
单个项目用 **git config --local http.proxy......** 

> 一般我们设置单个项目就行了，内部项目一般不需要使用代理

#### 查看
##### 命令方式
查看当前项目配置，前提是处于项目目录下
```
git config --list
```
> 不再任何项目下时，查看的就是全局配置

查看全局配置
```
git config --global --list

```

##### 文件方式配置
查看全局配置文件
```
cat ~/.gitconfig
```

查看单个项目的配置文件  
先进入项目的.git目录下
```
pwd
/d/code/xxx/xxx/.git

MINGW64 /d/code/xxx/xxx/.git (GIT_DIR!)
```
在查看config文件
```
$ cat config

```


#### 取消代理
```
git config --local --unset http.proxy

git config --local --unset https.proxy
```
或者修改文件，删掉对应记录
```
vi ~/.gitconfig
```

> 全局的参考上面，用：--global


#### 测试
不启动代理时：
```
git clone https://github.com/xxx/xxx.git
Cloning into 'xxxx'...
fatal: unable to access 'https://github.com/xxx/xxx.git/': Failed to connect to 127.0.0.1 port 1080: Connection refused

```


### git协议转http协议

之前是通过**git**协议克隆的项目
```
git clone git@github.com:xxx/xxx.git
```
查看
```
$ git remote -v
origin  git@github.com:xxx/xxx.git (fetch)
origin  git@github.com:xxx/xxx.git (push)

```

删除origin，原来是git协议
```
git remote remove origin
```

重新设置origin，这次用https协议
```
git remote add origin https://github.com/xxx/xxx.git
```

为分支设置追踪信息
```
git branch --set-upstream-to=origin/<branch> master
```

--------

### 设置用户和邮箱
```
git config --local user.name "wwh"
git config --local user.email "wangwen135@gmail.com"
```
> 为什么要单独设置用户名和密码  
> 全局设置的用户名和邮箱是另外一个，用于提交公司的代码



