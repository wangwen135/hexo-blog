---
title: Curator-例子
date: 2022-10-06 23:41
tags: 
  - Zookeeper
categories:
  - [Zookeeper]
---


先在applicationContext.xml 文件中集成Curator

```
<!-- 重连策略 -->
<bean id="retryPolicy" class="org.apache.curator.retry.ExponentialBackoffRetry">
    <constructor-arg index="0" value="${curator.retry.baseSleepTimeMs}" /> <!-- 间隔时间基数 -->
    <constructor-arg index="1" value="${curator.retry.maxRetries}" /><!-- 最多重试几次 -->
</bean>

<bean id="curatorClient" class="org.apache.curator.framework.CuratorFrameworkFactory" factory-method="newClient" init-method="start" destroy-method="close">
    <constructor-arg index="0" value="${curator.server.list}" />
    <constructor-arg index="1" value="${curator.session.timeout}" /><!-- sessionTimeoutMs会话超时时间，单位为毫秒。默认是60000ms -->
    <constructor-arg index="2" value="${curator.connection.timeout}" /><!-- connectionTimeoutMs连接创建超时时间，单位毫秒，默认15000ms -->
    <constructor-arg index="3" ref="retryPolicy" />
</bean>
    
```
config.properties
```
## curator 的相关配置
#间隔时间基数
curator.retry.baseSleepTimeMs=1000
#最多重试几次
curator.retry.maxRetries=3

curator.server.list=192.168.1.213:2181
curator.session.timeout=15000
curator.connection.timeout=10000
```

java
```

import java.util.List;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.recipes.locks.InterProcessMutex;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

/**
 * <pre>
 * 任务线程
 * </pre>
 * 
 * @author wwh
 */
@Component
public class TaskThread2 implements Runnable {

    private final static Logger logger = LoggerFactory.getLogger(TaskThread2.class);
    /**
     * 获取锁的超时时间
     */
    private static final int LOCK_ACQUIRE_TIMEOUT = 1000;
    /**
     * 锁路径<br>
     * 全路径为：zkBasePath + LOCK_PATH + task.getId();
     */
    private static final String LOCK_PATH = "/lock/";
    @Autowired
    private CuratorFramework curatorClient;
    /**
     * 基础路径
     */
    @Value("${zookeeper.basePath}")
    private String zkBasePath;

    private Thread thread;

    /**
     * 运行标记
     */
    private boolean runFlag = true;

    private final ReentrantLock lock = new ReentrantLock();
    private final Condition sysClose = lock.newCondition();

    private String task;

    @Override
    public void run() {
        InterProcessMutex lock = new InterProcessMutex(curatorClient, getLockPath());

        while (runFlag) {
            try {
                if (lock.acquire(LOCK_ACQUIRE_TIMEOUT, TimeUnit.MILLISECONDS)) {
                    try {
                        logger.debug("当前线程获取到锁，开始处理数据");
                        doWithLock();
                    } catch (Exception e) {
                        logger.error("执行任务：{} 处理逻辑时异常，", task, e);
                    } finally {
                        try {
                            lock.release();
                        } catch (Exception e) {
                            logger.error("任务：{} , 释放锁时：ZK错误或链接中断", task, e);
                        }
                    }
                }
            } catch (Exception e) {
                logger.error("任务：{} 获取锁时：ZK错误或链接中断", task, e);
            }
        }
        logger.info("任务：{} , 处理线程退出", task);
        // 移除zookeeper上锁节点
        destroyZKLockPath();
    }

    /**
     * 具体执行代码<br>
     * 获取到锁的时候才会运行
     */
    private void doWithLock() {
    }

    /**
     * 获取锁的路径，以任务ID为锁
     * 
     * @return
     */
    private String getLockPath() {
        // 以任务id作为锁的路径
        return zkBasePath + LOCK_PATH + task;
    }

    private void destroyZKLockPath() {
        // 删除锁的条件是没有其他的节点了
        try {
            String path = getLockPath();
            List<String> list = curatorClient.getChildren().forPath(path);
            if (list == null || list.isEmpty()) {
                // 如果没有其他节点还在获取锁就删除
                curatorClient.delete().forPath(path);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 启动任务
     * 
     * @param task
     */
    public void start(String task) {
        // 只能启动一次
        if (thread != null) {
            throw new IllegalStateException("任务线程只能启动一次");
        }
        logger.info("开始处理任务：{}", task);
        this.task = task;
        // 启动线程
        thread = new Thread(this, "T-" + task);
        thread.start();
    }

    /**
     * 停止任务
     */
    public void stop() {
        runFlag = false;
        // 如果是获取锁时的等待无法唤醒
        lock.lock();
        try {
            sysClose.signalAll();// 唤醒空数据时的等待
        } finally {
            lock.unlock();
        }
    }
}

```
