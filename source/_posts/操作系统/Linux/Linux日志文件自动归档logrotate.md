---
title: Linux日志文件自动归档logrotate
date: 2024-09-08 01:02
tags: 
  - Linux
  - Log
  - logrotate
categories:
  - [操作系统, Linux]
---



## logrotate

logrotate 是 Linux 系统中用于管理日志文件的工具，它可以根据定义的规则对日志文件进行轮转、压缩、删除或归档，常用于防止日志文件占用过多的磁盘空间。


### 主要功能
1. **轮转日志文件**：当日志文件达到设定的大小或时间限制时，生成一个新的日志文件。
2. **压缩旧日志**：对轮转后的日志进行压缩（如 .gz 格式）。
3. **删除旧日志**：可以设置保留一定数量的日志文件，超过数量的旧日志会被删除。
4. **发送信号**：轮转后可以通知服务重新生成新的日志文件，通常通过 SIGHUP 信号完成。
5. **灵活的配置**：支持基于文件大小、日期等多种条件来轮转日志。
 

### 配置文件
logrotate 的配置文件位于 /etc/logrotate.conf，你可以在这里指定全局的日志轮转策略。此外，每个服务可以通过 /etc/logrotate.d/ 目录下的文件配置自己的日志轮转规则。

**配置文件示例**
```
/var/log/myapp/*.log {
    daily             # 每天轮转
    missingok         # 忽略不存在的日志文件
    rotate 7          # 保留7天的日志
    compress          # 轮转后压缩日志文件
    delaycompress     # 延迟到下一次轮转时才压缩
    notifempty        # 如果日志文件为空，不轮转
    create 0640 root root  # 轮转后创建新的日志文件，权限为0640，所有者为root
    postrotate        # 轮转后执行的命令
        systemctl reload myapp
    endscript
}
```
**常用选项说明**
- daily/weekly/monthly：指定轮转周期为天、周、月。
- rotate [数字]：指定保留日志文件的数量。
- compress：启用压缩（默认使用gzip）。
- create [权限] [所有者] [组]：轮转后创建一个新的日志文件，并指定权限和所有者。
- postrotate/endscript：日志轮转完成后执行的脚本，可以用于通知服务重启或重新加载日志。


### 手动执行日志轮转
可以使用以下命令手动执行轮转：
```
logrotate -f /etc/logrotate.conf
```

-f 表示强制轮转，即使日志没有达到轮转条件。


-----

### nginx 日志文件
nginx 日志默认会自动归档，配置文件位于：/etc/logrotate.d/nginx
```
/var/log/nginx/*.log {
    create 0640 nginx root
    daily
    rotate 10
    missingok
    notifempty
    compress
    delaycompress
    sharedscripts
    postrotate
        /bin/kill -USR1 `cat /run/nginx.pid 2>/dev/null` 2>/dev/null || true
    endscript
}
```
但是只会归档/var/log/nginx/*.log，如果你配置了其他目录则需要自己写配置文件  

`/bin/kill -USR1 `cat /run/nginx.pid 2>/dev/null` 2>/dev/null || true` 这一行是在日志轮转后通过向 Nginx 主进程发送USR1` 信号，通知 Nginx 重新打开日志文件。  
-USR1 是用户定义的信号（SIGUSR1），Nginx 支持使用 USR1 信号来指示它重新打开日志文件。

### java app out日志文件
在 Java 应用程序中，日志归档通常是通过日志框架如 Log4j、SLF4J（结合其他日志实现如 Logback 或 Log4j）来管理的。这些日志框架提供了非常灵活的日志输出和轮转功能，允许开发者对日志进行归档、压缩、删除等操作。

一般启动java应用还是会使用 nohup 命令生成一个 out 文件，确保即使日志配置不正确或者应用程序崩溃时，依然能保留日志信息。这是为了增加日志记录的安全性和完整性，尤其是在程序异常退出或日志框架配置出错的情况下。
```
nohup java -jar test.jar > test.out 2>&1 &
```

由于 test.out 文件包含了应用程序的所有输出（包括标准输出和错误输出），随着程序运行时间的增长，该日志文件会越来越大。为了避免 test.out 文件过大导致磁盘空间耗尽，可以使用 logrotate 管理 test.out 的日志轮转。

#### 配置示例：
在 /etc/logrotate.d/ 下创建一个文件，比如 test，内容如下：
```
/opt/java/test/out/*.out {
    # 每天轮转一次
    daily
    # 如果日志文件不存在则跳过
    missingok
    # 保留 30 天的日志文件
    rotate 30
    # 轮转后压缩旧的日志文件
    compress
    # 延迟到下一次轮转时才压缩
    delaycompress
    # 如果文件为空，则不轮转
    notifempty
    # 复制日志文件并清空原始文件（避免重启应用）
    copytruncate
}
```

其中：  
- copytruncate：复制当前日志内容并清空原文件，而不需要重启 Java 程序。这是因为直接删除或移动 test.out 文件后，正在运行的程序仍然会继续写入已被删除的文件描述符，因此需要使用 copytruncate 来保留文件描述符。

使用命令进行测试：
```
logrotate -f /etc/logrotate.d/test
```
