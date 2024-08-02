---
title: ES集群 分片UNASSIGNED
date: 2018-03-30 16:06
tags: 
  - Elasticsearch
categories:
  - [Elasticsearch]
---

## 问题以及解决步骤记录

由于启动的问题导致了集群状态为 yellow

出现分片UNASSIGNED

(应该是所谓的出现了脑裂)

关闭主节点后，重新选择了之前的主节点之后，数据没这么乱，稍微好点

部分分片一直处于INITIALIZING，并且分片不均衡，节点1上之前有分片1 、 3 、4，现在变成了分片 2
```
curl -s "http://node3:9205/_cat/shards" 

...
...
downloads          1 p STARTED      641603  24.2gb 192.168.1.92 node-2 
downloads          1 r INITIALIZING                192.168.1.93 node-3 
downloads          2 r INITIALIZING                192.168.1.92 node-2 
downloads          2 p STARTED      641049  26.5gb 192.168.1.93 node-3 
downloads          3 r INITIALIZING                192.168.1.91 node-1 
downloads          3 p STARTED      639813  26.3gb 192.168.1.93 node-3 
downloads          4 r INITIALIZING                192.168.1.91 node-1 
downloads          4 p STARTED      642036  26.1gb 192.168.1.93 node-3 
downloads          0 r INITIALIZING                192.168.1.92 node-2 
downloads          0 p STARTED      640368  25.6gb 192.168.1.93 node-3 
...
...
```

使用下面的命令关闭分片复制
```
curl -XPUT 'node3:9205/downloads/_settings' -d '{
    "index": {
       "number_of_replicas": "0"
    }
}'
```
返回
```
{"acknowledged":true}
```

此时节点状态变成绿色，之前节点上的备份分片被删除，查看数据文件  
(目录：/data/dap/es/data/dap_es/nodes/0/indices/downloads)  
之前：
```
[root@node1 downloads]# du -h --max-dept=1
27G     ./1
27G     ./3
27G     ./4
8.0K    ./_state
16K     ./2
80G     .
```
之后：
可以看到删除了之前的分片数据，并重新迁移分片数据，数据文件慢慢变大
```
[root@node1 downloads]# du -h --max-dept=1
8.0K    ./_state
13G     ./0
13G     .
```

此时状态为：RELOCATING 
```
curl -s "http://node3:9205/_cat/shards" 

...
...
downloads          1 p STARTED    641603  24.2gb 192.168.1.92 node-2                                               
downloads          2 p RELOCATING 641049  26.5gb 192.168.1.93 node-3 -> 192.168.1.92 MNskHyFsQfC0PixAx-3hBQ node-2 
downloads          3 p STARTED    639813  26.3gb 192.168.1.93 node-3                                               
downloads          4 p STARTED    642036  26.1gb 192.168.1.93 node-3                                               
downloads          0 p RELOCATING 640368  25.6gb 192.168.1.93 node-3 -> 192.168.1.91 pkjX2YIbTnm3A49re8COPg node-1 
...

```

ES将逐个分片进行迁移
```
downloads          1 p STARTED    641603  24.2gb 192.168.1.92 node-2                                               
downloads          2 p RELOCATING 641049  26.5gb 192.168.1.93 node-3 -> 192.168.1.92 MNskHyFsQfC0PixAx-3hBQ node-2 
downloads          3 p STARTED    639813  26.3gb 192.168.1.93 node-3                                               
downloads          4 p STARTED    642036  26.1gb 192.168.1.93 node-3                                               
downloads          0 p STARTED    640368  25.6gb 192.168.1.91 node-1
```
可以看到节点1已经迁移完成

等待迁移完成

此时在ES的head插件上可以看到全部变成绿色了，之前迁移的分片的紫色的

再将复制分片数改为1

```
curl -XPUT 'node3:9205/downloads/_settings' -d '{
    "index": {
       "number_of_replicas": "1"
    }
}'
```
返回
```
{"acknowledged":true}
```
查看状态
```
curl -s "http://node3:9205/_cat/shards"

...
...
downloads          1 p STARTED      641603  24.2gb 192.168.1.92 node-2 
downloads          1 r INITIALIZING                192.168.1.91 node-1 
downloads          2 p STARTED      641049  26.5gb 192.168.1.92 node-2 
downloads          2 r INITIALIZING                192.168.1.93 node-3 
downloads          3 r INITIALIZING                192.168.1.92 node-2 
downloads          3 p STARTED      639813  26.3gb 192.168.1.93 node-3 
downloads          4 r INITIALIZING                192.168.1.91 node-1 
downloads          4 p STARTED      642036  26.1gb 192.168.1.93 node-3 
downloads          0 p STARTED      640368  25.6gb 192.168.1.91 node-1 
downloads          0 r INITIALIZING                192.168.1.93 node-3 
...
...

```
系统将再次变成yellow状态

再查看数据文件，可以看到备份的数据文件再增长

完了之后就变成绿色状态了

-----
-----

## 处理方法2

另外一种处理方式，当出现UNASSIGNED，强行指定

```
curl -s "http://node3:9205/_cat/shards" | grep UNASSIGNED
```


```
curl -XPOST 'node3:9205/_cluster/reroute' -d '{
    "commands" : [ {
          "allocate" : {
              "index" : "downloads",
              "shard" : 4,
              "node" : "node-1",
              "allow_primary" : true
          }
        }
    ]
}'
```

5.0 之后的ES 改成了 **allocate_replica** 
```
{
  "commands": [
    {
      "allocate_replica": {
        "index": "mail_store",
        "shard": 1,
        "node": "slave2",
      }
    }
  ]
}
```
>将未分配的副本分片分配给节点

**注意如果主分片也未分配，则需要先分配主分片**

将主分片分配给包含陈旧副本的节点
```
{
  "commands": [
    {
      "allocate_stale_primary": {
        "index": "mail_store",
        "shard": 1,
        "node": "slave2",
        "accept_data_loss": true
      }
    }
  ]
}
```
>使用此命令可能会导致所提供的分片ID发生数据丢失。如果稍后具有良好数据副本的节点重新加入群集，则该数据将被使用此命令强制分配的旧副本数据覆盖。为确保这些影响得到充分理解，该命令需要accept_data_loss明确设置专用字段才能true使其工作。


还有一个命令是：**allocate_empty_primary**
>将空主分片分配给节点。  
>使用此命令会导致索引到此分片中的所有数据（如果它先前已启动）完全丢失。如果稍后具有数据副本的节点重新加入群集，则该数据将被删除！为确保这些影响得到充分理解，该命令需要accept_data_loss明确设置专用字段才能true使其工作。

------
------


## 其他资料

动态设置es索引副本数量  
```
curl -XPUT 'http://node3:9200/xxindex/_settings' -d '{  
   "number_of_replicas" : 2  
}' 
```
  
设置es不自动分配分片  
```
curl -XPUT 'http://node3:9200/xxindex/_settings' -d '{  
   "cluster.routing.allocation.disable_allocation" : true  
}' 
```
> 需要先关闭索引
  
手动移动分片  
```
curl -XPOST "http://node3:9200/_cluster/reroute' -d  '{  
   "commands" : [{  
        "move" : {  
            "index" : "xlog",  
            "shard" : 0,  
            "from_node" : "es-0",  
            "to_node" : "es-3"  
        }  
    }]  
}' 
```
  
手动分配分片
```
curl -XPOST "http://node3:9200/_cluster/reroute' -d  '{  
   "commands" : [{  
        "allocate" : {  
            "index" : ".kibana",  
            "shard" : 0,  
            "node" : "es-2",  
        }  
    }]  
}'
```

取消分配
```
curl -XPOST "http://ESnode:9200/_cluster/reroute" -d '{
  "commands" : [ {
      "cancel" : {
          "index" : "ops",
          "shard" : 0,
          "node" : "es_node_one"
      }
  } ]
  }'
```
主分片和启动状态下的分片不能取消

