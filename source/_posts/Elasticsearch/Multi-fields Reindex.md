---
title: Multi-fields Reindex
date: 2017-04-25 10:35
tags: 
  - Elasticsearch
categories:
  - [Elasticsearch]
---

# Multi-fields Reindex

#### 1. 先获取索引
```
get http://192.168.1.51:9200/xxd/_mapping/p1  
```
得到一个JSON，修改这个JSON  

官方文档地址：  
https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-get-mapping.html

#### 2. 修改索引  
官网的例子：
```
PUT my_index
{
  "mappings": {
    "my_type": {
      "properties": {
        "city": {
          "type": "text",
          "fields": {
            "raw": { 
              "type":  "keyword"
            }
          }
        }
      }
    }
  }
}
```
如果直接照上面的改会报错：no handler for type [keyword] declared on field [raw]  
keyword是ES 5中的一个新类型，ES 2.X不支持  
这里实际上修改为：  

```
 "currentCompany": {
    "type": "string",
    "fields": {
      "raw": {
        "type": "string",
        "index": "not_analyzed"
      }
    }
  },
  
"headline": {
    "type": "string",
    "fields": {
      "raw": {
        "type": "string",
        "index": "not_analyzed"
      }
    }
  },
  
"industry": {
    "type": "string",
    "fields": {
      "raw": {
        "type": "string",
        "index": "not_analyzed"
      }
    }
  },
  
  "location": {
    "type": "string",
    "fields": {
      "raw": {
        "type": "string",
        "index": "not_analyzed"
      }
    }
  },
```

删掉第一级别的==xxd== ，保留如下：
```
{
  "mappings": {
    "p1": {
    ......
```

#### 3. 添加新的类型到索引中  
根据上面修改的JSON创建一个新的类型  
```
put http://192.168.1.51:9200/xxd2
{
  "mappings": {
    "p1": {
      "properties": {
      ......
```

官方文档地址：  
https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-put-mapping.html

#### 4. 通过get验证新创建的索引
```
get http://192.168.1.51:9200/xxd2/_mapping/p1  
```

#### 5. Reindex API  
官网地址：  
https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html#docs-reindex  
reindex API是新的，应该仍然被认为是实验性的。 API可能以不向后兼容的方式更改。  
重建索引不会尝试设置目标索引。它不复制源索引的设置。您应该在运行_reindex操作之前设置目标索引，包括设置映射，分片计数，副本等。

```
POST http://192.168.1.51:9200/_reindex
{
  "source": {
    "index": "xxd"
  },
  "dest": {
    "index": "xxd2"
  }
}
```

#### 6. 测试

```
POST http://192.168.1.51:9200/xxd2/p1/_search?search_type=count 
{
  "aggs": {
    "colors": {
      "terms": {
        "field": "location.raw"
      }
    }
  }
}
```
