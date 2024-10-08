---
title: Netty 4.0.30 连接池bug分析记录
date: 2019-06-14 23:41
tags: 
  - Netty
  - FastDFS
categories:
  - [Netty]
---

Fastdfs 问题分析记录

## 问题
生产上存在上传文件到fastdfs中时一直卡死的问题

大概半个月出现一次，重启就能恢复
日志中没有任何错误

## 分析
线程堆栈：
```
"taskExecutor-5" #638 prio=5 os_prio=0 tid=0x00007f970800b000 nid=0x1be3 waiting on condition [0x00007f9632008000]
   java.lang.Thread.State: WAITING (parking)
	at sun.misc.Unsafe.park(Native Method)
	- parking to wait for  <0x00000000e134e670> (a java.util.concurrent.CompletableFuture$Signaller)
	at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
	at java.util.concurrent.CompletableFuture$Signaller.block(CompletableFuture.java:1693)
	at java.util.concurrent.ForkJoinPool.managedBlock(ForkJoinPool.java:3323)
	at java.util.concurrent.CompletableFuture.waitingGet(CompletableFuture.java:1729)
	at java.util.concurrent.CompletableFuture.get(CompletableFuture.java:1895)
	at com.appleframework.file.provider.fdfs.FdfsProvider.upload(FdfsProvider.java:61)
	at com.appleframework.file.spring.AbstractFSProviderSpringFacade.upload(AbstractFSProviderSpringFacade.java:70)
	at com.chedaia.biz.renew.provider.service.impl.CustomerShoppingServiceImpl$1.run(CustomerShoppingServiceImpl.java:956)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
```
只知道是在等一个CompletableFuture异步计算的返回结果，可能是再等待获取存储服务器地址，也可能是再等待上传结果

检查代码并进行测试，没有发现问题
猜测可能是获取不到连接导致的，连接没有归还到连接池中等，也可能是fastdfs一直没有响应，或者网络出现了什么问题等。

直到生产上再次出现该问题
将生产上的程序堆栈dump出来查看对比之前看的代码，发现是获取的连接数达到了最大值
```
jmap -dump:format=b,file=文件名 [pid]
```
![图片1.png](https://img.wangwen135.top:23456/image/2024/08/66ba2dc9af4f4.png)

而连接池中并没连接

![图片2.png](https://img.wangwen135.top:23456/image/2024/08/66ba2e2f65550.png)

OQL
```
select x.deque from io.netty.channel.pool.FixedChannelPool x
```


### 总结问题：
`FixedChannelPool` 类的 `acquiredChannelCount` 字段
一直增加到了最大的限制（100）导致的
获取到的连接一直没有释放

### 检查原因：
查代码，并且重点测试这个变量，一直无法重现该问题（怀疑人生~_~）

导出测试程序的dump文件和线上的进行比较
对比了一下测试版本和线上版本的dump文件，发现类不一致

![图片3.png](https://img.wangwen135.top:23456/image/2024/08/66ba2ea3cc665.png)

线上版本没有这个变量

![图片4.png](https://img.wangwen135.top:23456/image/2024/08/66ba2ec28d5b5.png)

检查发现 调试版本 和 线上的版本 不一致

- 调试版本：**netty-all-4.0.34.Final.jar**
 
- 线上版本：**netty-all-4.0.30.Final.jar**


再次查看netty-all-4.0.30的代码，发现netty的连接池存在问题

## 问题重现

检查代码发现连接归还到连接池中再次获取时存在问题
当连接用完之后，归还到连接池中，过了一段时间之后连接变成了 TIME_WAIT 状态（或断开），再去获取连接（此时将获取到一个异常的连接）

![图片5.png](https://img.wangwen135.top:23456/image/2024/08/66ba2f20ad8fc.png)

获取到连接之后会对连接的有效性进行检查

![图片6.png](https://img.wangwen135.top:23456/image/2024/08/66ba2f3c04e77.png)


检查到连接异常之后会关掉这个连接并再次获取连接

![图片7.png](https://img.wangwen135.top:23456/image/2024/08/66ba2f6623b14.png)

此时会再调用**FixedChannelPool**类的`acquire0`方法获取连接，这将导致**acquiredChannelCount** 变量再次 +1，这次获取连接将会+2，而在释放时只会-1，这将导致这个值一直增大，直到达到maxConnections 的限制，之后就再也获取不到连接了

![图片8.png](https://img.wangwen135.top:23456/image/2024/08/66ba2fbd6e512.png)

释放连接时只会-1

![图片9.png](https://img.wangwen135.top:23456/image/2024/08/66ba2fdaa5906.png)


## 修复方案
升级netty的包到更高的版本

**`netty-all-4.0.30.Final.jar 包存在这个问题`**

**netty-all-4.0.34.Final.jar 已经修复了这个问题**

>PS：
这个4.0.34版本还是可能会有问题
private int acquiredChannelCount;
这个变量是int类型，存在并发问题，可能多个线程同时修改这个变量，应该用AtomicInteger，但是以目前系统的并发量来看，完全可以忽略。
