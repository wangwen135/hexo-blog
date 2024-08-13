---
title: 清理Zookeeper的历史数据
date: 2018-04-21 23:41
tags: 
  - Zookeeper
categories:
  - [Zookeeper]
---

## zkCleanup
zookeeper使用一段时间后占用了非常多的磁盘空间

```
[root@node2 zookeeper]# du -h --max-dept=1
294G    ./version-2
294G    .

```

使用自带的zkCleanup进行清理
```
./zkCleanup.sh 
Usage:
PurgeTxnLog dataLogDir [snapDir] -n count
        dataLogDir -- path to the txn log directory
        snapDir -- path to the snapshot directory
        count -- the number of old snaps/logs you want to keep, value should be greater than or equal to 3
```

```     
[root@node2 bin]# ./zkCleanup.sh /data/zookeeper -n 5

......
Removing file: Mar 10, 2018 7:57:04 PM  /data/zookeeper/version-2/log.3150c663239
Removing file: Feb 4, 2018 5:39:46 AM   /data/zookeeper/version-2/log.313054d1843
Removing file: Apr 20, 2018 7:40:19 PM  /data/zookeeper/version-2/log.31607ac00d7
Removing file: Apr 6, 2018 3:33:47 PM   /data/zookeeper/version-2/log.3153be932d2
Removing file: Mar 31, 2018 3:46:12 PM  /data/zookeeper/version-2/log.3152f255a35
Removing file: Mar 19, 2018 1:28:12 AM  /data/zookeeper/version-2/log.31516d3c5d7
Removing file: Jan 16, 2018 10:16:43 AM /data/zookeeper/version-2/log.31205b9f349
Removing file: Mar 26, 2018 7:05:37 AM  /data/zookeeper/version-2/log.315245d23e8
Removing file: Apr 12, 2018 6:31:10 AM  /data/zookeeper/version-2/log.31548340db4
......
```

## 设置自动清理
修改zoo.cfg配置文件中的 autopurge.snapRetainCount 和 autopurge.purgeInterval 两个参数实现定时清理  

去掉注释即可
```
# The number of snapshots to retain in dataDir
autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
autopurge.purgeInterval=1

```
>autopurge.purgeInterval  这个参数指定了清理频率，单位是小时，需要填写一个1或更大的整数，默认是0，表示不开启自动清理功能。

>autopurge.snapRetainCount 这个参数和上面的参数搭配使用，这个参数指定了需要保留的快照文件数目，默认是保留3个。

