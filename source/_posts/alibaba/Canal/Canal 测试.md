---
title: Canal 测试
date: 2021-06-19 23:41
tags:   
  - Canal
  - Mysql
categories:
  - [alibaba,Canal]
---


## Canal
canal [kə'næl]，译意为水道/管道/沟渠，主要用途是基于 MySQL 数据库增量日志解析，提供增量数据订阅和消费

https://github.com/alibaba/canal

https://github.com/alibaba/canal/releases/tag/canal-1.1.4

### 工作原理
#### MySQL主备复制原理
- MySQL master 将数据变更写入二进制日志( binary log, 其中记录叫做二进制日志事件binary log events，可以通过 show binlog events 进行查看)
- MySQL slave 将 master 的 binary log events 拷贝到它的中继日志(relay log)
- MySQL slave 重放 relay log 中事件，将数据变更反映它自己的数据

#### canal 工作原理
- canal 模拟 MySQL slave 的交互协议，伪装自己为 MySQL slave ，向 MySQL master 发送dump 协议
- MySQL master 收到 dump 请求，开始推送 binary log 给 slave (即 canal )
- canal 解析 binary log 对象(原始为 byte 流)


### Canal Admin
canal 1.1.4版本，迎来最重要的WebUI能力，引入canal-admin工程，支持面向WebUI的canal动态管理能力，支持配置、任务、日志等在线白屏运维能力，具体文档：Canal Admin Guide

https://github.com/alibaba/canal/wiki/Canal-Admin-Guide

https://github.com/alibaba/canal/wiki/Canal-Admin-QuickStart


### Canal java 客户端
https://github.com/alibaba/canal/wiki/ClientExample


## 使用

### Mysql 开启binlog日志
先开启 Binlog 写入功能，配置 binlog-format 为 ROW 模式，my.cnf 中配置如下

```
# 唯一标识 不要和 canal 的 slaveId 重复
server-id=1

# binlog文件名
log-bin=mysql-binlog

# 选择 ROW 模式
binlog-format=ROW 

# 要忽略的数据库
binlog-ignore-db=mysql
```

启动mysql后可以在data目录中看到新增的文件
- mysql-binlog.000001
- mysql-binlog.index

查看一下状态：
```
SHOW MASTER STATUS;

"File"	"Position"	"Binlog_Do_DB"	"Binlog_Ignore_DB"	"Executed_Gtid_Set"
"mysql-binlog.000001"	"154"	""	"mysql"	""
```

### 新建账号
授权 canal 链接 MySQL 账号具有作为 MySQL slave 的权限
```
CREATE USER canal IDENTIFIED BY 'canal';

GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';

FLUSH PRIVILEGES;
```
查看一下：  
SELECT USER,HOST FROM mysql.user;


### Canal 配置
下载地址：https://github.com/alibaba/canal/releases/download/canal-1.1.4/canal.deployer-1.1.4.tar.gz

解压缩下载的canal-deployer压缩包

#### conf/canal.properties
默认配置，注意：
```
# 服务端口配置
canal.port = 11111

# 客户端连接账号配置(目前未设置)
# canal.user = canal
# canal.passwd = E3619321C1A937C46A0D8BD1DAC39F93B27D4458
# 客户端连接端点名
canal.destinations = example
```

#### conf/example/instance.properties
默认配置，注意：
```
## canal伪装的slaveId(只要和已有的节点ID不冲突即可)
canal.instance.mysql.slaveId=99

# 数据库连接配置
canal.instance.master.address=127.0.0.1:3306
canal.instance.dbUsername=canal
canal.instance.dbPassword=canal
canal.instance.connectionCharset = UTF-8

# 监听对象(数据库/数据表) 逗号分隔
#canal.instance.filter.regex=test.table1,test.table2
canal.instance.filter.regex=.*\\..*

# 监听黑名单(当需要监听很多表, 只有少量表不需要监听时配置)
canal.instance.filter.black.regex=

```

#### 运行
- 启动 - ${canal}/bin/start.sh
- 停止 - ${canal}/bin/stop.sh
- 重启 - ${canal}/bin/restart.sh

如果启动报错，如：
```
ch.qos.logback.core.LogbackException: Unexpected filename extension of file [file:/D:/dev/canal/canal.deployer-1.1.4/conf/]. Should be either .groovy or .xml
```
修改启动脚本：
```
@rem set logback_configurationFile=%conf_dir%\logback.xml

去掉 @rem

或者去掉

-Dlogback.configurationFile="%logback_configurationFile%" 

```

#### 查看日志
server日志 - ${canal}/logs/canal/canal.log
```
......
start the canal server[10.10.20.112(10.10.20.112):11111]
Start prometheus HTTPServer on port 11112.
 ## the canal server is running now ......
```

instance日志 - ${canal}/logs/example/example.log
```
RegisterSlaveCommandPacket[reportHost=127.0.0.1,reportPort=59599,reportUser=canal,reportPasswd=canal,serverId=99,command=21]
position:BinlogDumpCommandPacket[binlogPosition=973,slaveServerId=99,binlogFileName=mysql-binlog.000001,command=18]
```

**完成后就可以用下面的java代码进行测试了**


### Canal Admin 配置
下载地址：https://github.com/alibaba/canal/releases/download/canal-1.1.4/canal.admin-1.1.4.tar.gz

解压canal.admin-1.1.4.tar.gz

#### conf/application.yml
默认配置文件
```
server:
  port: 8089
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8

spring.datasource:
  address: 127.0.0.1:3306
  database: canal_manager
  username: canal
  password: canal
  driver-class-name: com.mysql.jdbc.Driver
  url: jdbc:mysql://${spring.datasource.address}/${spring.datasource.database}?useUnicode=true&characterEncoding=UTF-8&useSSL=false
  hikari:
    maximum-pool-size: 30
    minimum-idle: 1

canal:
  adminUser: admin
  adminPasswd: admin
```
注意：这里使用的是上面新建canal 账号，此时需要赋权
```
GRANT SELECT, INSERT, UPDATE, DELETE ON `canal_manager`.* TO 'canal'@'%';

FLUSH PRIVILEGES;
```


#### 导入初始化SQL
```
mysql -h127.0.0.1 -uroot -p

# 导入初始化SQL
> source conf/canal_manager.sql
```
初始化SQL脚本里会默认创建canal_manager的数据库，建议使用root等有超级权限的账号进行初始化


#### 启动
```
sh bin/startup.sh
```

查看 admin 日志
```
vi logs/admin.log
```

#### 访问
 http://127.0.0.1:8089/
 
>默认密码：admin/123456


---


### Canal 与 Canal admin 集成
canal-admin设计上是为canal提供整体配置管理、节点运维等面向运维的功能，提供相对友好的WebUI操作界面，方便更多用户快速和安全的操作

#### canal-admin的核心模型主要有：
1. instance，对应canal-server里的instance，一个最小的订阅mysql的队列
2. server，对应canal-server，一个server里可以包含多个instance
3. 集群，对应一组canal-server，组合在一起面向高可用HA的运维



#### 修改canal配置文件
修改canal配置文件让其注册到 canal-admin

>使用canal_local.properties的配置覆盖canal.properties
```
# register ip
canal.register.ip =

# canal admin config
canal.admin.manager = 127.0.0.1:8089
canal.admin.port = 11110
canal.admin.user = admin
canal.admin.passwd = 4ACFE3202A5FF5CF467898FC58AAB1D615029441
# admin auto register
canal.admin.register.auto = true
canal.admin.register.cluster =
```


原本canal-server运行所需要的canal.properties/instance.properties配置文件就需要在web ui上进行统一运维，每个server只需要以最基本的启动配置 (比如知道一下canal-admin的manager地址，以及访问配置的账号、密码即可)


### canal-admin 使用

#### canal-amdin 》 server管理 》 新建Server
- 所属集群：单机
- server名称：测试
- Server IP：192.168.31.111
- admin端口：11110

> 注意IP，如果canal启动不了，就需要去看看日志，是不是ip配置错误了
> managerAddress:127.0.0.1:8089 can't not found config for [10.10.20.112:11110]

操作 》  配置  可以修改canal server的配置
```
# tcp bind ip
canal.ip =
# register ip to zookeeper
canal.register.ip =
canal.port = 11111
canal.metrics.pull.port = 11112
# canal instance user/passwd
# canal.user = canal
# canal.passwd = E3619321C1A937C46A0D8BD1DAC39F93B27D4458
```
> 注释掉canal的用户名密码，不然会报错，不知道为啥
> 这里默认的用户名密码是 canal/canal

#### canal-amdin 》 Instance管理 》 新建Instance
- Instance名称：demo
- 所属集群/主机：测试

**载入模板**

修改配置
```
canal.instance.mysql.slaveId=99

canal.instance.dbUsername=canal
canal.instance.dbPassword=canal
canal.instance.connectionCharset = UTF-8
canal.instance.filter.regex=.*\\..*
canal.instance.filter.black.regex=

```
> 这里只改slaveId=99
> 要注意数据库连接信息


#### 重新启动canal
查看日志

此时可以看到 server 和 instance 都是启动状态

## 使用java客户端进行测试

> 单机模式 和 用canal-admin 配置两种方式都用下面的代码测试


新建一个maven项目

导入依赖
```
<dependency>
    <groupId>com.alibaba.otter</groupId>
    <artifactId>canal.client</artifactId>
    <version>1.1.0</version>
</dependency>
```

> canal server 的用户名密码配置在 canal.properties 文件中，使用canal-amdin时配置在网页中  
> 默认的用户名密码为：canal/canal


```
package com.wwh.canal.test;

import java.net.InetSocketAddress;
import java.util.List;

import com.alibaba.otter.canal.client.CanalConnectors;
import com.alibaba.otter.canal.client.CanalConnector;
import com.alibaba.otter.canal.common.utils.AddressUtils;
import com.alibaba.otter.canal.protocol.Message;
import com.alibaba.otter.canal.protocol.CanalEntry.Column;
import com.alibaba.otter.canal.protocol.CanalEntry.Entry;
import com.alibaba.otter.canal.protocol.CanalEntry.EntryType;
import com.alibaba.otter.canal.protocol.CanalEntry.EventType;
import com.alibaba.otter.canal.protocol.CanalEntry.RowChange;
import com.alibaba.otter.canal.protocol.CanalEntry.RowData;

public class CanalTest {

    public static void main(String args[]) {
        System.out.println(AddressUtils.getHostIp());

        // 创建链接
        CanalConnector connector = CanalConnectors
                .newSingleConnector(new InetSocketAddress(AddressUtils.getHostIp(), 11111), "demo", "", "");
        // destination 配置为 example 或者 demo
        
        int batchSize = 1000;
        int emptyCount = 0;
        try {
            connector.connect();
            connector.subscribe(".*\\..*");
            connector.rollback();
            int totalEmptyCount = 120;
            while (emptyCount < totalEmptyCount) {
                Message message = connector.getWithoutAck(batchSize); // 获取指定数量的数据
                long batchId = message.getId();
                int size = message.getEntries().size();
                if (batchId == -1 || size == 0) {
                    emptyCount++;
                    System.out.println("empty count : " + emptyCount);
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                    }
                } else {
                    emptyCount = 0;
                    // System.out.printf("message[batchId=%s,size=%s] \n", batchId, size);
                    printEntry(message.getEntries());
                }

                connector.ack(batchId); // 提交确认
                // connector.rollback(batchId); // 处理失败, 回滚数据
            }

            System.out.println("empty too many times, exit");
        } finally {
            connector.disconnect();
        }
    }

    private static void printEntry(List<Entry> entrys) {
        for (Entry entry : entrys) {
            if (entry.getEntryType() == EntryType.TRANSACTIONBEGIN
                    || entry.getEntryType() == EntryType.TRANSACTIONEND) {
                continue;
            }

            RowChange rowChage = null;
            try {
                rowChage = RowChange.parseFrom(entry.getStoreValue());
            } catch (Exception e) {
                throw new RuntimeException("ERROR ## parser of eromanga-event has an error , data:" + entry.toString(),
                        e);
            }

            EventType eventType = rowChage.getEventType();
            System.out.println(String.format("================&gt; binlog[%s:%s] , name[%s,%s] , eventType : %s",
                    entry.getHeader().getLogfileName(), entry.getHeader().getLogfileOffset(),
                    entry.getHeader().getSchemaName(), entry.getHeader().getTableName(), eventType));

            for (RowData rowData : rowChage.getRowDatasList()) {
                if (eventType == EventType.DELETE) {
                    printColumn(rowData.getBeforeColumnsList());
                } else if (eventType == EventType.INSERT) {
                    printColumn(rowData.getAfterColumnsList());
                } else {
                    System.out.println("-------&gt; before");
                    printColumn(rowData.getBeforeColumnsList());
                    System.out.println("-------&gt; after");
                    printColumn(rowData.getAfterColumnsList());
                }
            }
        }
    }

    private static void printColumn(List<Column> columns) {
        for (Column column : columns) {
            System.out.println(column.getName() + " : " + column.getValue() + "    update=" + column.getUpdated());
        }
    }

}

```
