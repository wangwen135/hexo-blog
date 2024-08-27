---
title: Mysql事务和锁测试
date: 2020-10-15 23:41
tags: 
  - MySQL
  - 事务
  - 锁
categories:
  - [MySQL]
---

### 数据库版本

#### 使用 SELECT 语句
```
mysql> SELECT VERSION();
+------------+
| VERSION()  |
+------------+
| 5.6.41-log |
+------------+
1 row in set (0.09 sec)
```
#### 使用 SHOW VARIABLES
```
mysql> SHOW VARIABLES LIKE 'version';
+---------------+------------+
| Variable_name | Value      |
+---------------+------------+
| version       | 5.6.41-log |
+---------------+------------+
1 row in set (0.14 sec)
```




### 查看事务隔离级别

#### 查看当前会话的事务隔离级别
```
mysql> SELECT @@SESSION.tx_isolation;
+------------------------+
| @@SESSION.tx_isolation |
+------------------------+
| READ-COMMITTED         |
+------------------------+
1 row in set (0.07 sec)
```
> 读已提交 READ-COMMITTED


#### 查看全局事务隔离级别
```
mysql> SELECT @@GLOBAL.tx_isolation;
+-----------------------+
| @@GLOBAL.tx_isolation |
+-----------------------+
| READ-COMMITTED        |
+-----------------------+
1 row in set (0.06 sec)
```

#### 使用 SHOW VARIABLES
```
mysql> SHOW VARIABLES LIKE 'tx_isolation';
+---------------+----------------+
| Variable_name | Value          |
+---------------+----------------+
| tx_isolation  | READ-COMMITTED |
+---------------+----------------+
1 row in set (0.08 sec)
```

### 使用事务

```
# 开启事务
START TRANSACTION;

# 执行操作A...
# 执行操作B...

# 提交或回滚事务
COMMIT;
ROLLBACK;

```


### 查看事务

```
SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX;
```
这条查询语句将返回一个表，显示所有当前活动的事务的信息，例如：
- trx_id：事务 ID。
- trx_state：事务状态（如 RUNNING、LOCK WAIT 等）。
- trx_started：事务开始的时间。
- trx_requested_lock_id：如果事务在等待锁定，则显示请求的锁定 ID。
- trx_wait_started：如果事务在等待锁定，则显示等待开始的时间。
- trx_mysql_thread_id：执行事务的 MySQL 线程 ID。

查看一个事务运行了多长时间：
```
SELECT TIMESTAMPDIFF(SECOND, t.trx_started, NOW()) AS trx_runtime_seconds,t.* FROM information_schema.INNODB_TRX t;
```

#### 查看 INNODB_TRX 表的锁等待状态
INNODB_TRX 表包含当前正在执行的事务信息，通过它可以查看是否有事务处于等待锁的状态。

```
SELECT 
    trx_id, 
    trx_state, 
    trx_started, 
    trx_wait_started, 
    TIMESTAMPDIFF(SECOND, trx_wait_started, NOW()) AS wait_time_seconds,
    trx_requested_lock_id
FROM 
    INFORMATION_SCHEMA.INNODB_TRX
WHERE 
    trx_state = 'LOCK WAIT';
```
- trx_state = 'LOCK WAIT' 表示正在等待锁的事务。
- wait_time_seconds 显示事务等待锁的时间。

如果有长时间等待锁的事务，可以怀疑存在潜在的死锁。


---


### 手动制造一个死锁
现在有一个表 test，有Id 和 v 两个字段，里面有两条数据 1，1；2，2

> 确保wait_timeout 时间足够长

#### 步骤：
1. 在第一个会话中，开始一个事务并锁定记录 id=1
```
START TRANSACTION;
UPDATE test SET v = 11 WHERE id = 1;
```
2. 在第二个会话中，开始另一个事务并锁定记录 id=2：
```
START TRANSACTION;
UPDATE test SET v = 22 WHERE id = 2;
```
3. 回到第一个会话，尝试更新记录 id=2（此时这个记录已被第二个事务锁定，导致第一个事务无法继续）：
```
UPDATE test SET value = 12 WHERE id = 2;
```
4. 回到第二个会话，尝试更新记录 id=1（此时这个记录已被第一个事务锁定，导致第二个事务无法继续）：
```
UPDATE test SET v = 21 WHERE id = 1;

```
此时会报错，Mysql会检测到一个死锁错误：

```
1213 - Deadlock found when trying to get lock; try restarting transaction
```
MySQL 会自动检测到这个死锁，并回滚其中一个事务（通常是后发起的事务），这是 MySQL 中的一个重要特性，它能有效地避免数据库的无限期阻塞。




----- 

### 查当前正在运行的 MySQL 线程
####  使用 SHOW PROCESSLIST

SHOW PROCESSLIST 命令用于显示当前 MySQL 服务器中所有连接的信息。默认情况下，它会显示所有会话的状态，包括线程 ID、用户、主机、数据库、命令、时间、状态和查询等信息。

```
SHOW PROCESSLIST;
SHOW FULL PROCESSLIST;
```
> 在 MySQL 5.6 及更高版本中，你可以使用 SHOW FULL PROCESSLIST 获取更详细的信息


#### 使用 INFORMATION_SCHEMA.PROCESSLIST 视图

通过查询 INFORMATION_SCHEMA.PROCESSLIST 视图来实现更灵活的过滤
```
SELECT * FROM INFORMATION_SCHEMA.PROCESSLIST;
SELECT * FROM INFORMATION_SCHEMA.PROCESSLIST WHERE DB = 'test';
```

**示例**
显示所有运行时间超过 60 秒的查询：
```
SELECT * FROM INFORMATION_SCHEMA.PROCESSLIST WHERE TIME > 60;
```
显示所有正在等待锁的查询：
```
SELECT * FROM INFORMATION_SCHEMA.PROCESSLIST WHERE STATE LIKE '%lock%';
```
只显示特定用户的会话：
```
SELECT * FROM INFORMATION_SCHEMA.PROCESSLIST WHERE USER = 'your_user';
```
显示所有非空闲会话：
```
SELECT * FROM INFORMATION_SCHEMA.PROCESSLIST WHERE COMMAND != 'Sleep';
```


### Kill 命令
KILL 命令用于终止指定会话（连接）或线程。它可以强制关闭一个正在运行的会话或终止一个特定的查询。KILL 命令对于管理数据库的并发访问和处理意外情况（如死锁、长时间运行的查询或网络中断导致的挂起事务）非常有用。

#### KILL 命令的作用
1. **终止指定的会话**：当你执行 KILL <thread_id> 时，MySQL 会终止与该线程 ID 关联的会话。这意味着 MySQL 会关闭这个会话连接并释放它所持有的所有资源（例如锁、内存等）。
2. **中断当前执行的查询**：KILL 命令还可以用于中断一个正在执行的查询。如果一个查询运行时间过长或者出现了性能问题，使用 KILL 可以立即停止该查询。

#### KILL 命令的使用语法
```
KILL [CONNECTION | QUERY] thread_id;
```
- KILL thread_id：默认操作是 KILL CONNECTION，它终止与给定线程 ID 关联的会话。会话中所有正在进行的操作都会被中止，该会话也会被关闭。
- KILL CONNECTION thread_id：显式地终止一个会话，与 KILL thread_id 的效果相同。
- KILL QUERY thread_id：仅中断给定线程 ID 的当前执行的查询，而不会关闭会话。如果你想停止某个特定查询但保持会话连接有效，可以使用这个命令。

**示例：**
假设你想要终止一个 ID 为 1234 的线程（会话）
```
KILL 1234;
```
如果你只是想中断 ID 为 1234 的线程正在执行的查询，但不关闭会话：
```
KILL QUERY 1234;
```





### 空闲超时（Idle Timeout）
MySQL 服务器有一个 wait_timeout 参数，定义了一个连接在空闲状态下的最大等待时间。如果一个事务在没有进行任何操作的情况下达到这个超时时间，MySQL 会关闭连接，未提交的事务将被自动回滚。


#### 查看和修改 wait_timeout 设置:
```
-- 查看当前会话超时设置
SHOW VARIABLES LIKE 'wait_timeout';

-- 查看全局超时设置
SHOW VARIABLES LIKE 'interactive_timeout';

-- 修改会话超时（单位：秒）
SET @@SESSION.wait_timeout = 28800;  -- 8小时

-- 修改全局超时（单位：秒）
SET @@GLOBAL.wait_timeout = 28800;   -- 8小时

```

### Lock wait timeout exceeded

锁等待超时（Lock Wait Timeout Exceeded）：发生在一个事务在等待另一个事务持有的锁时，超过了设置的最大等待时间。默认情况下，MySQL 的 innodb_lock_wait_timeout 参数定义了事务等待锁的最长时间（默认是 50 秒）。如果超过了这个时间，MySQL 会中止等待，并返回 1205 错误。

查看和修改锁等待超时时间
```
# 查看，默认是50秒
show variables like 'innodb_lock_wait_timeout';

# 设置锁等待超时时间为 360 秒
set @@SESSION.innodb_lock_wait_timeout=360;

# 设置全局锁等待超时时间
SET GLOBAL innodb_lock_wait_timeout = 120;

```














