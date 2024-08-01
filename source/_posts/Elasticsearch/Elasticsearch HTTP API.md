---
title: Elasticsearch HTTP API
date: 2017-05-15
tags: 
  - Elasticsearch
categories:
  - [Elasticsearch]
---

# Elasticsearch HTTP API
https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html

## 文档

### Index API
添加或更新JSON文档到指定的索引
```
curl -XPUT 'http://192.168.1.213:9200/twitter/tweet/1' -d '{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}'
```

### Get API
从索引中获取JSON文档根据ID
```
curl -XGET 'http://192.168.1.213:9200/twitter/tweet/1'

```

### Delete API 
从索引中删除JSON文档根据ID
```
curl -XDELETE 'http://192.168.1.213:9200/twitter/tweet/1'

```

### 查询

请求体查询  
```
curl -XGET 'http://192.168.1.213:9200/twitter/tweet/_search' -d '{
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}'

```

URI查询
```
curl -XGET 'http://192.168.1.213:9200/twitter/tweet/_search?q=user:kimchy'
```

#### Query DSL 
https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html#query-dsl

Elasticsearch提供了基于JSON来定义查询完整的查询DSL。查询DSL看作查询的AST，由两种类型的子句：
- 叶查询子句
    >叶查询子句寻找一个特定的值在某一特定领域，如匹配(match)，期限(term)或范围(range)查询。这些查询可以自行使用。
- 复合查询子句
    >复合查询包含其他叶查询或复合的查询，并用于多个查询以逻辑方式（如 bool 或 dis_max 查询）结合，或者改变它们的行为（如 not 或 constant_score 查询）


#### 聚合
取出现次数最多的前20个

```
POST http://192.168.1.91:9200/_search?search_type=count 

{
  "aggs": {
    "topLocation": {
      "terms": {
        "field": "location.raw",
        "size": 20
      }
    }
  }
}
```


##### 查询结果高亮
只有针对字段查询的时候才会高亮，对 **_all** 进行查询时是不会高亮的。  

如：
```
{
  "query": {
    "match": {
      "commandReply": "USER ftpuser"
    }
  },
  "highlight": {
    "fields": {
      "commandReply": {}
    }
  }
}
```
结果：  
```
...
...
    "frontNodeId": 0,
    "routeIp": "192.168.1.1"
    }
},
"highlight": {
    "commandReply": [
        "220 (vsFTPd 3.0.2) <em>USER</em> <em>ftpuser</em> 331 Please specify the password. PASS <em>ftpuser</em> 230 Login"
        ,
        " successful. ### ### Login successful UserName: <em>ftpuser</em> Password: <em>ftpuser</em> ### SYST 215 UNIX Type: L8"
        ]
    }
}
```


## 集群
### Cluster Health 
获取简单的集群状态
```
curl -XGET 'http://192.168.1.213:9200/_cluster/health?pretty=true'
```

该API还可以针对一个或多个索引执行以只获得指定索引的健康状态
```
curl -XGET 'http://192.168.1.213:9200/_cluster/health/wwh_test?pretty'
```

### Cluster State 
群集状态API允许获得整个集群的综合状态信息
```
curl -XGET 'http://192.168.1.213:9200/_cluster/state'
curl -XGET 'http://192.168.1.213:9200/_cluster/stats?human&pretty'

#过滤
curl -XGET 'http://localhost:9200/_cluster/state/{metrics}/{indices}'

```

---

## Cat API

- 查看主节点

http://192.168.1.213:9200/_cat/master?v

curl '192.168.1.213:9200/_cat/master?v'

- 查看集群是否健康

http://192.168.1.213:9200/_cat/health?v

curl '192.168.1.213:9200/_cat/health?v'

- 查看节点列表

http://192.168.1.213:9200/_cat/nodes?v

curl '192.168.1.213:9200/_cat/nodes?v'

- 列出所有索引

http://192.168.1.213:9200/_cat/indices?v

curl '192.168.1.213:9200/_cat/indices?v'


