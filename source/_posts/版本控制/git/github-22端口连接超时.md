---
title: github-22端口连接超时
date: 2023-04-05 23:41
tags: 
  - git
categories:
  - [版本控制, git]
---


现象
```
>git pull
ssh: connect to host github.com port 22: Connection timed out
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

但是443端口可以
```
>ssh -T -p 443 git@ssh.github.com
The authenticity of host '[ssh.github.com]:443 ([20.205.243.160]:443)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[ssh.github.com]:443' (ED25519) to the list of known hosts.
Hi wangwen135! You've successfully authenticated, but GitHub does not provide shell access.
```

在用户目录新增config文件
```
$ vim ~/.ssh/config
```
内容如下：
```
# Add section below to it
Host github.com
  Hostname ssh.github.com
  Port 443
```

再测试一下：
```
$ ssh -T git@github.com
Hi xxxxx! You've successfully authenticated, but GitHub does not
provide shell access.

```



