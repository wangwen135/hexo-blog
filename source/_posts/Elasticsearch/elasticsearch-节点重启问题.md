---
title: elasticsearch-节点重启问题
date: 2023-10-15 23:41
tags: 
  - Elasticsearch
categories:
  - [Elasticsearch]
---

ElasticSearch集群的高可用和自平衡方案会在节点挂掉（重启）后自动在别的结点上复制该结点的分片，这将导致了大量的IO和网络开销。

如果离开的节点重新加入集群，elasticsearch为了对数据分片(shard)进行再平衡，会为重新加入的节点再次分配数据分片(Shard), 当一台es因为压力过大而挂掉以后，其他的es服务会备份本应那台es保存的数据，造成更大压力，于是整个集群会发生雪崩。

生产环境下建议关闭自动平衡。

## 数据分片与自平衡

一：关闭自动分片，即使新建index也无法分配数据分片
```
curl -XPUT http://192.168.1.213:9200/_cluster/settings -d '{
  "transient" : {
    "cluster.routing.allocation.enable" : "none"
  }
}'
```

二：关闭自动平衡，只在增减ES节点时不自动平衡数据分片
```
curl -XPUT http://192.168.1.213:9200/_cluster/settings?pretty -d '{
  "transient" : {
    "cluster.routing.rebalance.enable" : "none"
  }
}'
```

设置完以后查看设置是否添加成功：
```
curl http://192.168.1.213:9200/_cluster/settings?pretty
```

重新启用自动分片

```
curl -XPUT http://192.168.1.213:9200/_cluster/settings -d '{
  "transient" : {
    "cluster.routing.allocation.enable" : "all"
  }
}
```


## 延迟副本的重新分配
```
PUT /_all/_settings
{
  "settings": {
    "index.unassigned.node_left.delayed_timeout": "5m"
  }
}
```
未分配节点重新分配延迟到5分钟之后


下面是修改 elasticsearch.yml 文件

```
gateway.recover_after_nodes: 8
```
这将防止Elasticsearch立即开始数据恢复，直到集群中至少有八个（数据节点或主节点）节点存在。

```
gateway.expected_nodes: 10 

gateway.recover_after_time: 5m
```
集群开始数据恢复等到5分钟后或者10个节点加入，以先到者为准。

参考：  
https://www.elastic.co/guide/en/elasticsearch/reference/2.4/modules-gateway.html

## 脑裂问题
对某一个实例进行重启后，很有可能会导致该实例无法找到master而将自己推举为master的情况出现，为防止这种情况，需要调整 elasticsearch.yml 中的内容
```
discovery.zen.minimum_master_nodes: 2
```
这个配置就是告诉Elasticsearch除非有足够可用的master候选节点，否则就不选举master，只有有足够可用的master候选节点才进行选举。
该设置应该始终被配置为有主节点资格的点数/2 + 1，例如:  
有10个符合规则的节点数，则配置为6.
有3个则配置为2.

---

## 关于设置的有效性

**persistent** 重启后设置也会存在   
**transient** 整个集群重启后会消失的设置

```
PUT /_cluster/settings
{
    "persistent" : {
        "discovery.zen.minimum_master_nodes" : 2
    }
}
```

----

----

----
## 一般设置下面两个就可以了

```
# 通过配置大多数节点(节点总数/ 2 + 1)来防止脑裂
#
discovery.zen.minimum_master_nodes: 2

# 在一个完整的集群重新启动到N个节点开始之前，阻止初始恢复
#
gateway.recover_after_nodes: 3
```



