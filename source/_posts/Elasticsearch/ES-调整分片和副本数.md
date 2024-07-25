---
title: ES-调整分片和副本数
date: 2022-07-08 23:41
tags: 
  - Elasticsearch
categories:
  - [Elasticsearch]
---



#### 一、调整副本数


如调整副本数为0

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

#### 二、调整索引分片

索引分片数在索引创建好了之后就不能调整了，只能重建索引

>(ES 5.X 版本中有一个缩小分片的api，需要先设置为只读，然后缩减过程需要大量的IO)

先创建索引
```
curl -XPUT 'http://localhost:9200/wwh_test2/' -d '{
    "settings" : {
        "index" : {
            "number_of_shards" : 2, 
            "number_of_replicas" : 2 
        }
    }
}'
```
或者同时指定mappings
```
curl -XPOST localhost:9200/test -d '{
    "settings" : {
        "number_of_shards" : 1
    },
    "mappings" : {
        "type1" : {
            "properties" : {
                "field1" : { "type" : "string", "index" : "not_analyzed" }
            }
        }
    }
}'
```

之后再进行重新索引
```
curl -XPOST 'http://localhost:9200/_reindex' -d '{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter"
  }
}'
```



#### 开关索引

关闭
```
curl -XPOST 'localhost:9200/lookupindex/_close'

```

打开
```
curl -XPOST 'localhost:9200/lookupindex/_open'

```
