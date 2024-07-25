---
title: 查看binlog日志
date: 2022-11-07 23:41
tags: 
  - MySQL
categories:
  - [MySQL]
---

 > *2018年9月*

### mysqlbinlog
使用mysqlbinlog 命令查看


常用选项：  
 * --start-position=953                   起始pos点
 * --stop-position=1437                   结束pos点
 * --start-datetime="2013-11-29 13:18:54" 起始时间点
 * --stop-datetime="2013-11-29 13:21:53"  结束时间点

如：
>进入到binlog目录执行，指定具体的binlog文件名
```
# 从指定的时间开始查
$ mysqlbinlog mysql-bin.001575 --start-datetime="2018-09-25 11:25:54"

# 从指定位置开始查
$ mysqlbinlog mysql-bin.001575 --start-position=16839697

```

> 默认只能查看到base-64编码的信息
如：
```
# at 22452867
#180925 11:44:02 server id 8  end_log_pos 22452944 CRC32 0xc23d3641     Table_map: `base`.`by_renew_history` mapped to number 182
# at 22452944
#180925 11:44:02 server id 8  end_log_pos 22453106 CRC32 0x7a809e4e     Update_rows: table id 182 flags: STMT_END_F

BINLOG '
Aq+pWxMIAAAATQAAANCaVgEAALYAAAAAAAEABGJhc2UAEGJ5X3JlbmV3X2hpc3RvcnkACwMPAw8S
ARIDEgMDB5YAWgAAAAD+B0E2PcI=
Aq+pWx8IAAAAogAAAHKbVgEAALYAAAAAAAEAAgAL/////wD+igAAAA4yMDE4MDcyNTE2MzcyNugD
AAAQMTExMTExMTExMDExMTExNJmgcwlbAZmgcwlbAwAAAJmgcwlaAP6KAAAADjIwMTgwNzI1MTYz
NzI26AMAABAxMTExMTExMTEwMTExMTEzmaBzCVsBmaBzCVsDAAAAmaBzCVpOnoB6
'/*!*/;
# at 22453106
#180925 11:44:02 server id 8  end_log_pos 22453137 CRC32 0x3787881e     Xid = 8121164
COMMIT/*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
```

此时需要 **--base64-output=value** 选项

此选项确定何时应使用BINLOG语句将事件编码为base-64字符串。该选项具有以下允许值（不区分大小写）:

- AUTO （“自动”）或UNSPEC（“未指定”）在必要时自动显示BINLOG语句（即，用于格式描述事件和行事件）。如果没有给出--base64-output选项，则效果与--base64-output = AUTO相同。

- NEVER 不显示BINLOG语句

- DECODE-ROWS 通过配合--verbose选项将行事件解码并显示为注释的SQL语句。

-----

使用参数--verbose(或-v)，将生成带注释的语句，如果使用两次这个参数(如-v -v)，会生成字段的类型、长度、是否为null等属性信息。


如：
```
$ mysqlbinlog -v --base64-output=DECODE-ROWS mysql-bin.001576 --start-datetime="2018-09-25 13:41:30"
```
输出：
```
# at 1664232
#180925 13:42:02 server id 8  end_log_pos 1664309 CRC32 0x28bbcc7e      Table_map: `base`.`by_renew_history` mapped to number 182
# at 1664309
#180925 13:42:02 server id 8  end_log_pos 1664471 CRC32 0xca13320e      Update_rows: table id 182 flags: STMT_END_F
### UPDATE `base`.`by_renew_history`
### WHERE
###   @1=138
###   @2='20180725163726'
###   @3=1000
###   @4='1111111110111113'
###   @5='2018-07-25 16:37:27'
###   @6=1
###   @7='2018-07-25 16:37:27'
###   @8=3
###   @9='2018-07-25 16:37:26'
###   @10=NULL
###   @11=NULL
### SET
###   @1=138
###   @2='20180725163726'
###   @3=1000
###   @4='1111111110111114'
###   @5='2018-07-25 16:37:27'
###   @6=1
###   @7='2018-07-25 16:37:27'
###   @8=3
###   @9='2018-07-25 16:37:26'
###   @10=NULL
###   @11=NULL
# at 1664471
#180925 13:42:02 server id 8  end_log_pos 1664502 CRC32 0xb64f6d4f      Xid = 8266920
COMMIT/*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
```

```
mysqlbinlog -v -v --base64-output=DECODE-ROWS mysql-bin.001576 --start-datetime="2018-09-25 13:41:30"
```
输出：
```
# at 1899587
#180925 13:44:02 server id 8  end_log_pos 1899664 CRC32 0x7fff0312      Table_map: `base`.`by_renew_history` mapped to number 182
# at 1899664
#180925 13:44:02 server id 8  end_log_pos 1899826 CRC32 0x5cdb92c1      Update_rows: table id 182 flags: STMT_END_F
### UPDATE `base`.`by_renew_history`
### WHERE
###   @1=138 /* INT meta=0 nullable=0 is_null=0 */
###   @2='20180725163726' /* VARSTRING(150) meta=150 nullable=1 is_null=0 */
###   @3=1000 /* INT meta=0 nullable=1 is_null=0 */
###   @4='1111111110111114' /* VARSTRING(90) meta=90 nullable=1 is_null=0 */
###   @5='2018-07-25 16:37:27' /* DATETIME(0) meta=0 nullable=1 is_null=0 */
###   @6=1 /* TINYINT meta=0 nullable=1 is_null=0 */
###   @7='2018-07-25 16:37:27' /* DATETIME(0) meta=0 nullable=1 is_null=0 */
###   @8=3 /* INT meta=0 nullable=1 is_null=0 */
###   @9='2018-07-25 16:37:26' /* DATETIME(0) meta=0 nullable=1 is_null=0 */
###   @10=NULL /* INT meta=0 nullable=1 is_null=1 */
###   @11=NULL /* INT meta=0 nullable=1 is_null=1 */
### SET
###   @1=138 /* INT meta=0 nullable=0 is_null=0 */
###   @2='20180725163726' /* VARSTRING(150) meta=150 nullable=1 is_null=0 */
###   @3=1000 /* INT meta=0 nullable=1 is_null=0 */
###   @4='1111111110111113' /* VARSTRING(90) meta=90 nullable=1 is_null=0 */
###   @5='2018-07-25 16:37:27' /* DATETIME(0) meta=0 nullable=1 is_null=0 */
###   @6=1 /* TINYINT meta=0 nullable=1 is_null=0 */
###   @7='2018-07-25 16:37:27' /* DATETIME(0) meta=0 nullable=1 is_null=0 */
###   @8=3 /* INT meta=0 nullable=1 is_null=0 */
###   @9='2018-07-25 16:37:26' /* DATETIME(0) meta=0 nullable=1 is_null=0 */
###   @10=NULL /* INT meta=0 nullable=1 is_null=1 */
###   @11=NULL /* INT meta=0 nullable=1 is_null=1 */
# at 1899826
#180925 13:44:02 server id 8  end_log_pos 1899857 CRC32 0xc1b4c0c7      Xid = 8297691
COMMIT/*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
```
