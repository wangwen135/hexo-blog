---
title: HBase-Shell
date: 2023-03-04 23:41
tags: 
  - HBase
categories:
  - [大数据, HBase]
---


```
$ ./bin/hbase shell
hbase(main):001:0>
```
输入help 显示帮助信息

#### 1. 创建表
使用 **create** 命令创建表，指定表名和列族名称
```
hbase(main):001:0> create 'test', 'cf'
0 row(s) in 0.4170 seconds

=> Hbase::Table - test
```

#### 2. 查看表
使用 **list** 查看表
```
hbase(main):002:0> list 'test'
TABLE
test
1 row(s) in 0.0180 seconds

=> ["test"]
```

#### 3. 放数据到表中
使用 **put** 命令
```
hbase(main):003:0> put 'test', 'row1', 'cf:a', 'value1'
0 row(s) in 0.0850 seconds

hbase(main):004:0> put 'test', 'row2', 'cf:b', 'value2'
0 row(s) in 0.0110 seconds

hbase(main):005:0> put 'test', 'row3', 'cf:c', 'value3'
0 row(s) in 0.0100 seconds
```

#### 4. 扫描表中的数据
使用 **scan** 命令
```
hbase(main):006:0> scan 'test'
ROW                                      COLUMN+CELL
 row1                                    column=cf:a, timestamp=1421762485768, value=value1
 row2                                    column=cf:b, timestamp=1421762491785, value=value2
 row3                                    column=cf:c, timestamp=1421762496210, value=value3
3 row(s) in 0.0230 seconds
```
使用 limit 限制返回结果
```
scan 'test' ,{'LIMIT' => 5}
```

#### 5. 获取单行数据
使用 **get** 命令
```
hbase(main):007:0> get 'test', 'row1'
COLUMN                                   CELL
 cf:a                                    timestamp=1421762485768, value=value1
1 row(s) in 0.0350 seconds
```

#### 6. 禁用表
如果要删除表或更改其设置，以及在某些其他情况下，需要先使用 **disable** 命令禁用表。你可以使用 **enable** 命令重新启用它。
```
hbase(main):008:0> disable 'test'
0 row(s) in 1.1820 seconds

hbase(main):009:0> enable 'test'
0 row(s) in 0.1770 seconds
```

#### 7. 删除表
使用 **drop** 命令
```
hbase(main):011:0> drop 'test'
0 row(s) in 0.1370 seconds
```

-------------

## 高级操作

####  扫描

指定起始行 和 终止行，并指定返回的列
```



scan 'downloads', {STARTROW => 'nare7pqnmojs2pg.onion' , ENDROW=> 'nare7pqnmojs2pg.onionz',COLUMN=>['cf1:url']}


scan 'downloads', {STARTROW => 'nare7' , ENDROW=> 'nare7z',COLUMN=>['cf1:url']}


```
