---
title: Redis基本命令
date: 2017-08-17 23:41
tags: 
  - Redis
categories:
  - [Redis]
---

## 使用客户端工具

```
bin/redis-cli 
```

测试命令
```
127.0.0.1:6379> ping
PONG

```

## 数据库
redis 可以提供16个数据库

通过select 选择，默认是0数据库
```
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> keys *
(empty list or set)
127.0.0.1:6379[1]> select 0
OK
127.0.0.1:6379> keys *
 1) "num"
 2) "myb2"
 3) "mylist2"
 4) "myhash"
 5) "myb1"
 6) "myset"
 7) "name"
 8) "eclipse"
 9) "myb3"
10) "name2"
11) "mya1"
12) "mylist"
13) "mya2"
14) "mysort"
15) "myahs"
16) "aa1"
17) "mya3"
```

清空数据库
```
flushall
```

## 基本数据

存数据  
```
127.0.0.1:6379> set name zhangsan
OK
```

获取数据
```
127.0.0.1:6379> get name
"zhangsan"
```

删除数据
```
127.0.0.1:6379> del name
(integer) 1
```

查看所有的key
```
127.0.0.1:6379> keys *
1) "name2"
2) "name"
```
查看匹配可以
```
127.0.0.1:6379> keys a*
1) "abc"
2) "aa1"
```


数字递增
```
127.0.0.1:6379> incr num
(integer) 1
127.0.0.1:6379> get num
"1"
127.0.0.1:6379> incr num
(integer) 2
127.0.0.1:6379> get num
"2"
127.0.0.1:6379> 
```
如果不存在会设置初始值为0，然后+1

数字递减
```
127.0.0.1:6379> decr num
(integer) 1
127.0.0.1:6379> get num
"1"
127.0.0.1:6379> decr num
(integer) 0
127.0.0.1:6379> get num
"0"
127.0.0.1:6379> 
127.0.0.1:6379> decr num
(integer) -1
127.0.0.1:6379> get num
"-1"
```

数字增加指定的值
```
127.0.0.1:6379> incrby num 5
(integer) 4
127.0.0.1:6379> incrby num 5
(integer) 9
127.0.0.1:6379> 
```

数字减去指定的值
```
127.0.0.1:6379> decrby num 3
(integer) 6
127.0.0.1:6379> decrby num 3
(integer) 3
127.0.0.1:6379> decrby num 3
(integer) 0
127.0.0.1:6379> 
```

拼接字符串
```
127.0.0.1:6379> append name laoli
(integer) 13
127.0.0.1:6379> get name
"zhangsanlaoli"
```

重命名KEY
```
127.0.0.1:6379> set a1 a1
OK
127.0.0.1:6379> get a1
"a1"
127.0.0.1:6379> rename a1 aa1
OK
127.0.0.1:6379> get a1
(nil)
127.0.0.1:6379> get aa1
"a1"
```

设置过期时间  
单位秒
```
127.0.0.1:6379> get abc
"abc"
127.0.0.1:6379> expire abc 10
(integer) 1
127.0.0.1:6379> get abc
"abc"
127.0.0.1:6379> get abc
(nil)
```

查看超时时间
```
127.0.0.1:6379> set abc abc
OK
127.0.0.1:6379> get abc
"abc"
127.0.0.1:6379> ttl abc
(integer) -1
127.0.0.1:6379> expire abc 100
(integer) 1
127.0.0.1:6379> ttl abc
(integer) 96
127.0.0.1:6379> 
```

获取key存储的数据类型
```
127.0.0.1:6379> type abc
string
127.0.0.1:6379> type mylist
list
127.0.0.1:6379> type myset
set
```

## Hash 类型

设值
```
127.0.0.1:6379> hset myhash uname zhangsan
(integer) 1
127.0.0.1:6379> hset myhash age 18
(integer) 1
```

设置多个值
```
127.0.0.1:6379> hmset myhash2 uname zhangs age 11
OK
```

取值
```
127.0.0.1:6379> hget myhash uname
"zhangsan"
```

一次取多个值
```
127.0.0.1:6379> hmget myhash uname age
1) "zhangsan"
2) "18"
```

获取全部值
```
127.0.0.1:6379> hgetall myhash
1) "uname"
2) "zhangsan"
3) "age"
4) "18"
```

删除一个值
```
127.0.0.1:6379> hdel myhash2 uname age
(integer) 2
127.0.0.1:6379> hgetall myhash2
(empty list or set)
```
删除不存在的
```
127.0.0.1:6379> hdel myhash2 uname
(integer) 0
```

删除整个集合
```
127.0.0.1:6379> hmset myhash2 uname zhangs age 21
OK
127.0.0.1:6379> del myhash2
(integer) 1
127.0.0.1:6379> hget myhash2 uname
(nil)
```

增加数据
```
127.0.0.1:6379> hget myhash age
"18"
127.0.0.1:6379> hincrby myhash age 5
(integer) 23
127.0.0.1:6379> hget myhash age
"23"
```

判断hash中某个键值是否存在
```
127.0.0.1:6379> hexists myhash uname
(integer) 1
```
1表示存在，0表示不存在


获取HASH中的键值对数量
```
127.0.0.1:6379> hgetall myhash
1) "uname"
2) "zhangsan"
3) "age"
4) "23"
127.0.0.1:6379> hlen myhash
(integer) 2
```

获取Hash中所有的key
```
127.0.0.1:6379> hkeys myhash
1) "uname"
2) "age"
```

获取hash中所有的值
```
127.0.0.1:6379> hvals myhash
1) "zhangsan"
2) "23"
```

## 数据结构list

ArrayList使用数组方式， LinkedList使用双向链表

从左侧向列表中添加数据
```
127.0.0.1:6379> lpush mylist a b c
(integer) 3
127.0.0.1:6379> lpush mylist 1 2 3
(integer) 6
127.0.0.1:6379> 

```

右侧添加
```
127.0.0.1:6379> rpush mylist2 a b c
(integer) 3
127.0.0.1:6379> rpush mylist2 1 2 3
(integer) 6
```

查看列表
```
127.0.0.1:6379> lrange mylist 0 5
1) "3"
2) "2"
3) "1"
4) "c"
5) "b"
6) "a"
```
后面指定范围，可以是负数，负数从后面开始

弹出列表中的元素  
左侧弹出
```
127.0.0.1:6379> lpop mylist
"3"
```
右侧弹出
```
127.0.0.1:6379> rpop mylist2
"3"
```

获取列表中的元素数量
```
127.0.0.1:6379> llen mylist
(integer) 5
```

lrem 删除

lset 设置某个index 的值

插入 linsert 列表 before index value

rpoplpush 列表1 列表2   从一个队列中移除添加到另外一个队列中


## 数据结构set
Set不允许出现重复的元素

添加
```
127.0.0.1:6379> sadd myset a b c
(integer) 3
127.0.0.1:6379> sadd myset a 
(integer) 0
```


删除
```
127.0.0.1:6379> sadd myset 1 2 3
(integer) 3
127.0.0.1:6379> srem myset 1 2
(integer) 2
```

查看
```
127.0.0.1:6379> smembers myset
1) "a"
2) "b"
3) "c"
4) "3"
```

判断是否存在
```
127.0.0.1:6379> sismember myset a
(integer) 1
127.0.0.1:6379> sismember myset x
(integer) 0
```
1表示存在，0表示不存在

差集运算
```
127.0.0.1:6379> sadd mya1 a b c
(integer) 3
127.0.0.1:6379> sadd myb1 a c 1 2
(integer) 4
127.0.0.1:6379> sdiff mya1 myb1
1) "b"
```

交集运算
```
127.0.0.1:6379> sadd mya2 a b c
(integer) 3
127.0.0.1:6379> sadd myb2 a c 1 2
(integer) 4
127.0.0.1:6379> sinter mya2 myb2
1) "a"
2) "c"
```

并集运算
```
127.0.0.1:6379> sadd mya3 a b c
(integer) 3
127.0.0.1:6379> sadd myb3 a c 1 2
(integer) 4
127.0.0.1:6379> sunion mya3 myb3
1) "2"
2) "a"
3) "1"
4) "c"
5) "b"
```

获取set中的成员数量
```
127.0.0.1:6379> scard myset
(integer) 4
```

随机返回一个
```
127.0.0.1:6379> srandmember myset
"b"
127.0.0.1:6379> srandmember myset
"c"
127.0.0.1:6379> srandmember myset
"a"
```

存储交集、并集、差集到一个新的集合中
sdiffstore sinterstore sunionstore


## sorted-set
排序，有个分数

添加
```
127.0.0.1:6379> zadd mysort 70 zhangsan 80 lisi 90 wangwu
(integer) 3
127.0.0.1:6379> zadd mysort 100 zhangsan
(integer) 0
127.0.0.1:6379> zadd mysort 60 tom
(integer) 1
```

获取分数
```
127.0.0.1:6379> zscore mysort zhangsan
"100"
```

或成员数量
```
127.0.0.1:6379> zcard mysort
(integer) 4
```

删除
```
127.0.0.1:6379> zrem mysort tom wangwu
(integer) 2
127.0.0.1:6379> zcard mysort
(integer) 2
```

范围查找
```
127.0.0.1:6379> zadd mysort 85 jack 95 rose
(integer) 2
127.0.0.1:6379> zrange mysort 0 -1
1) "lisi"
2) "jack"
3) "rose"
4) "zhangsan"
```
显示分数
```
127.0.0.1:6379> zrange mysort 0 -1 withscores
1) "lisi"
2) "80"
3) "jack"
4) "85"
5) "rose"
6) "95"
7) "zhangsan"
8) "100"
```
从大到小
```
127.0.0.1:6379> zrevrange mysort 0 -1 withscores
1) "zhangsan"
2) "100"
3) "rose"
4) "95"
5) "jack"
6) "85"
7) "lisi"
8) "80"

```

范围删除
zremrangebyrank mysort 0 4

按照分数删除
zremrangebyscore mysort 80 100


## 事物

开启事物
multi

提交事物
exec 

回滚事物
discard

---
---
---
图形化客户端工具 Redis Desktop Manager 更好用 
