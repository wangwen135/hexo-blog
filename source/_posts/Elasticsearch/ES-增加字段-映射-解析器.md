---
title: ES-增加字段-映射-解析器
date: 2024-01-09 23:41
tags: 
  - Elasticsearch
categories:
  - [Elasticsearch]
---

# ES 增加字段 映射 解析器

## 字段与映射
获取索引
```
get http://192.168.1.91:9200/mmx2/_mapping/p1
```

为已经存在的索引添加一个新的字段
```
put http://192.168.1.91:9200/dnd-类型1/_mapping/dndata

{
  "properties": {
    "user_name": {
      "type": "String"
    }
  }
}
```

对一个字段提供多种索引模式，同一个字段的值，一个分词，一个不分词 
```
{
  "properties": {
    "state": {
      "type": "string",
      "fields": {
        "raw": {
          "type": "string",
          "index": "not_analyzed"
        }
      }
    }
  }
}
```

字段索引两次使用不的解析器
```
PUT /my_index
{
    "settings": { "number_of_shards": 1 }, 
    "mappings": {
        "my_type": {
            "properties": {
                "title": { 
                    "type":     "string",
                    "analyzer": "english",
                    "fields": {
                        "std":   { 
                            "type":     "string",
                            "analyzer": "standard"
                        }
                    }
                }
            }
        }
    }
}
```

为字段配置一个特定的解析器
```
PUT /my_index/_mapping/my_type

{
    "my_type": {
        "properties": {
            "english_title": {
                "type":     "string",
                "analyzer": "english"
            }
        }
    }
}
```

自定义_all字段  
```
PUT /my_index
{
    "mappings": {
        "person": {
            "properties": {
                "first_name": {
                    "type":     "string",
                    "copy_to":  "full_name" 
                },
                "last_name": {
                    "type":     "string",
                    "copy_to":  "full_name" 
                },
                "full_name": {
                    "type":     "string"
                }
            }
        }
    }
}
```
ES通过字段映射中的copy_to参数将值复制到其他字段，在first_name和last_name字段中的值会被拷贝到full_name字段中。first_name和last_name字段的映射和full_name字段的索引方式的无关。full_name字段会从其它两个字段中拷贝字符串的值，然后仅根据full_name字段自身的映射进行索引。

```

```


## 解析器

解析器可以在几个级别被指定。ES会依次检查每个级别直到它找到了一个可用的解析器。在索引期间，检查的顺序是这样的：

- 定义在字段映射中的analyzer
- 文档的_analyzer字段中定义的解析器
- type默认的analyzer，它的默认值是
- 在索引设置(Index Settings)中名为default的解析器，它的默认值是
- 节点上名为default的解析器，它的默认值是
- standard解析器

在搜索期间，顺序稍微有所不同：

- 直接定义在查询中的analyzer
- 定义在字段映射中的analyzer
- type默认的analyzer，它的默认值是
- 在索引设置(Index Settings)中名为default的解析器，它的默认值是
- 节点上名为default的解析器，它的默认值是
- standard解析器
 
>以上两个点表示的项目突出显示了索引期间和搜索期间顺序的不同之处。_analyzer字段允许你能够为每份文档指定一个默认的解析器(比如，english，french，spanish)，而查询中的analyzer参数则让你能够指定查询字符串使用的解析器。然而，这并不是在一个索引中处理多语言的最佳方案，因为在处理自然语言中提到的一些陷阱。

