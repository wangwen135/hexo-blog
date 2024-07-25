---
title: ES-全文搜索
date: 2024-07-07 23:41
tags: 
  - Elasticsearch
categories:
  - [Elasticsearch]
---

# ES 全文搜索

1. 全文搜索
```
GET /my_index/my_type/_search
{
    "query": {
        "match": {
            "title": "BROWN DOG!"
        }
    }
}
```
使用了match查询的多词查询只是简单地将生成的term查询包含在了一个bool查询中。通过默认的or操作符，每个term查询都以一个语句被添加，所以至少一个should语句需要被匹配。以下两个查询是等价的：
```
{
    "match": { "title": "brown fox"}
}
{
  "bool": {
    "should": [
      { "term": { "title": "brown" }},
      { "term": { "title": "fox"   }}
    ]
  }
}
```

2. 提高查询精度  
match查询接受一个operator参数，该参数的默认值是"or"。你可以将它改变为"and"来要求所有的词条都需要被匹配：
```
GET /my_index/my_type/_search
{
    "query": {
        "match": {
            "title": {      
                "query":    "BROWN DOG!",
                "operator": "and"
            }
        }
    }
}
```
使用and操作符时，所有的term查询都以must语句被添加，因此所有的查询都需要匹配。以下两个查询是等价的：
```
{
    "match": {
        "title": {
            "query":    "brown fox",
            "operator": "and"
        }
    }
}
{
  "bool": {
    "must": [
      { "term": { "title": "brown" }},
      { "term": { "title": "fox"   }}
    ]
  }
}
```

3. 控制查询精度  
match查询支持minimum_should_match参数，它能够让你指定有多少词条必须被匹配才会让该文档被当做一个相关的文档。尽管你能够指定一个词条的绝对数量，但是通常指定一个百分比会更有意义，因为你无法控制用户会输入多少个词条：
```
GET /my_index/my_type/_search
{
  "query": {
    "match": {
      "title": {
        "query":                "quick brown dog",
        "minimum_should_match": "75%"
      }
    }
  }
}
```
如果指定了minimum_should_match参数，它会直接被传入到bool查询中，因此下面两个查询是等价的：
```
{
    "match": {
        "title": {
            "query":                "quick brown fox",
            "minimum_should_match": "75%"
        }
    }
}
{
  "bool": {
    "should": [
      { "term": { "title": "brown" }},
      { "term": { "title": "fox"   }},
      { "term": { "title": "quick" }}
    ],
    "minimum_should_match": 2 
  }
}
```

4. 合并查询  
bool查询通过must，must_not以及should参数来接受多个查询。比如：
```
GET /my_index/my_type/_search
{
  "query": {
    "bool": {
      "must":     { "match": { "title": "quick" }},
      "must_not": { "match": { "title": "lazy"  }},
      "should": [
                  { "match": { "title": "brown" }},
                  { "match": { "title": "dog"   }}
      ]
    }
  }
}
```

5. 控制精度  
正如可以控制match查询的精度，也能够通过minimum_should_match参数来控制should语句需要匹配的数量，该参数可以是一个绝对数值或者一个百分比：
```
GET /my_index/my_type/_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title": "brown" }},
        { "match": { "title": "fox"   }},
        { "match": { "title": "dog"   }}
      ],
      "minimum_should_match": 2 
    }
  }
}
```

6. 提升查询子句(Boosting Query Clause)  
假设我们需要搜索和"full-text search"相关的文档，但是我们想要给予那些提到了"Elasticsearch"或者"Lucene"的文档更多权重。更多权重的意思是，对于提到了"Elasticsearch"或者"Lucene"的文档，它们的相关度_score会更高，即它们会出现在结果列表的前面。  
一个简单的bool查询能够让我们表达较为复杂的逻辑：
```
GET /_search
{
    "query": {
        "bool": {
            "must": {
                "match": {
                    "content": { 
                        "query":    "full text search",
                        "operator": "and"
                    }
                }
            },
            "should": [ 
                { "match": { "content": "Elasticsearch" }},
                { "match": { "content": "Lucene"        }}
            ]
        }
    }
}
```

7. 多个查询字符串(Multiple Query Strings)  
```
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title":  "War and Peace" }},
        { "match": { "author": "Leo Tolstoy"   }}
      ]
    }
  }
}
```
```
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title":  "War and Peace" }},
        { "match": { "author": "Leo Tolstoy"   }},
        { "bool":  {
          "should": [
            { "match": { "translator": "Constance Garnett" }},
            { "match": { "translator": "Louise Maude"      }}
          ]
        }}
      ]
    }
  }
}
```

8. 设置子句优先级  
通过boost参数，增加字段的权重
```
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { 
            "title":  {
              "query": "War and Peace",
              "boost": 2
        }}},
        { "match": { 
            "author":  {
              "query": "Leo Tolstoy",
              "boost": 2
        }}},
        { "bool":  { 
            "should": [
              { "match": { "translator": "Constance Garnett" }},
              { "match": { "translator": "Louise Maude"      }}
            ]
        }}
      ]
    }
  }
}
```

9. 多字段查询
```
{
    "multi_match": {
        "query":                "Quick brown fox",
        "type":                 "best_fields", 
        "fields":               [ "title", "body" ],
        "tie_breaker":          0.3,
        "minimum_should_match": "30%" 
    }
}
```
注意到以上的type属性为best_fields、minimum_should_match和operator参数会被传入到生成的match查询中。  
在字段名中使用通配符
```
{
    "multi_match": {
        "query":  "Quick brown fox",
        "fields": "*_title"
    }
}
```
提升个别字段
个别字段可以通过caret语法(^)进行提升：仅需要在字段名后添加^boost，其中的boost是一个浮点数：
```
{
    "multi_match": {
        "query":  "Quick brown fox",
        "fields": [ "*_title", "chapter_title^2" ] 
    }
}
```
同一个字段使用了不同的解析器的情况
```
GET /my_index/_search
{
   "query": {
        "multi_match": {
            "query":  "jumping rabbits",
            "type":   "most_fields", 
            "fields": [ "title", "title.std" ]
        }
    }
}
```
