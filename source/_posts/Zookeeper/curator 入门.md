---
title: curator 入门
date: 2018-06-13 23:41
tags: 
  - Zookeeper
  - Curator
categories:
  - [Zookeeper]
---


http://curator.apache.org/getting-started.html

>学习ZooKeeper  
http: //zookeeper.apache.org/doc/trunk/zookeeperStarted.html

## 使用curator
curator的JAR包可从Maven Central获得。Maven，Gradle，Ant等的用户可以轻松地将curator包含在他们的构建脚本中。

Maven
```
<dependency>
 <groupId>org.apache.curator</groupId>
 <artifactId>curator-recipes</artifactId>
 <version>4.0.0</version>
</dependency>
```


## 获得连接
curator使用 Fluent Style 风格的API。

curator连接实例（CuratorFramework）由CuratorFrameworkFactory分配。要连接到的ZooKeeper群集只需要 **一个** CuratorFramework对象：
```
CuratorFrameworkFactory.newClient(zookeeperConnectionString, retryPolicy)
 ）
```
使用默认值创建与ZooKeeper群集的连接，唯一需要指定的是重试策略。在大多数情况下，应该使用：
```
RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3)
CuratorFramework client = CuratorFrameworkFactory.newClient(zookeeperConnectionString, retryPolicy);
client.start();
 ```
必须启动客户端（不再需要时关闭）。

## 直接使用ZooKeeper
一旦有一个CuratorFramework实例，就可以直接调用ZooKeeper，方法与使用ZooKeeper发行版中提供的原始ZooKeeper对象类似。例如：
```
client.create().forPath("/my/path", myData)

```
这里的好处是curator管理ZooKeeper连接，如果有连接问题，将重试操作。

## 食谱 Recipes

### 分布式锁
```
InterProcessMutex lock = new InterProcessMutex(client, lockPath);
if ( lock.acquire(maxWait, waitUnit) ) 
{
    try 
    {
        // do some work inside of the critical section here
    }
    finally
    {
        lock.release();
    }
}
```

### 领导选举

```
LeaderSelectorListener listener = new LeaderSelectorListenerAdapter()
{
    public void takeLeadership(CuratorFramework client) throws Exception
    {
        // this callback will get called when you are the leader
        // do whatever leader work you need to and only exit
        // this method when you want to relinquish leadership
    }
}

LeaderSelector selector = new LeaderSelector(client, path, listener);
selector.autoRequeue();  // not required, but this is behavior that you will probably expect
selector.start();
```
