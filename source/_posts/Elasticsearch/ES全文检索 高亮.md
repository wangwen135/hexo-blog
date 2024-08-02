---
title: ES全文检索 高亮
date: 2017-04-25 10:29
tags: 
  - Elasticsearch
categories:
  - [Elasticsearch]
---

# ES全文检索-高亮  

全文检索  
高亮  
统计各个索引命中情况  


http://192.168.1.1:9200/dwd-p1,dnd*/_search/

POST

```

{
  "query": {
    "match": {
      "_all": "pro  duct"
    }
  },
  "highlight": {
    "require_field_match": false,
    "fields": {
      "*": {}
    }
  },
  "aggs": {
    "indexCount": {
      "terms": {
        "field": "_index"
      }
    }
  }
}

```

java 代码  

```


        logger.debug("查询字符串：{}", queryStr);

        // 查询索引
        SearchRequestBuilder search = client.prepareSearch(p1IndexName, darknetIndexPrefix + "*");// 查询全部的类型

        QueryStringQueryBuilder qs = new QueryStringQueryBuilder(queryStr);
        // 最匹配的在前
        qs.useDisMax(true);
        search.setQuery(qs);

        // 高亮所有的字段
        search.addHighlightedField("*");

        // 所有的字段都进行高亮，而不单单只包含查询匹配的字段
        search.setHighlighterRequireFieldMatch(false);

        // 分页
        int start = (page.getPageNumber() - 1) * page.getPageSize();
        search.setFrom(start).setSize(page.getPageSize());

        // 统计各个索引的命中情况
        search.addAggregation(AggregationBuilders.terms("by_index").field("_index"));

        SearchResponse response = search.get();

        List<JSONObject> jsonsList = new ArrayList<>();
        int length = darknetIndexPrefix.length();
        // 将HIT转对象
        JSONObject json;
        for (SearchHit hit : response.getHits()) {
            json = new JSONObject();
            String index = hit.getIndex();
            String type = hit.getType();
            // 大分类
            // 小分类
            if (p1IndexName.equals(index)) {
                json.put("class", "xx数据");
                json.put("subclass", xx type);
            } else {
                json.put("class", "xx2数据");
                json.put("subclass", index.substring(length));
            }

            json.put("index", index);// 索引
            json.put("type", type);// 类型
            json.put("id", hit.getId());// ID
            json.put("score", hit.getScore()); // 得分

            // 高亮
            JSONArray highlightArray = new JSONArray();
            json.put("highlight", highlightArray);

            JSONObject fieldJsonObj;
            JSONArray _jsonArray;

            Map<String, HighlightField> highlightMap = hit.getHighlightFields();
            // 如果匹配到了非字符串字段，这个可能为空
            // TODO 应该需要进行另外的处理

            for (Map.Entry<String, HighlightField> entry : highlightMap.entrySet()) {
                fieldJsonObj = new JSONObject();
                _jsonArray = new JSONArray();

                fieldJsonObj.put(entry.getKey(), _jsonArray);

                HighlightField hlfield = entry.getValue();
                Text[] texts = hlfield.getFragments();
                for (Text text : texts) {
                    _jsonArray.add(text.toString());
                }

                highlightArray.add(fieldJsonObj);
            }

            String dss = hit.sourceAsString();// json格式的数据类容
            JSONObject ct = JSON.parseObject(dss, JSONObject.class);
            json.put("source", ct);

            jsonsList.add(json);
        }

        /**
         * 获取命中情况
         */
        // 获取聚合结果
        Terms tos = response.getAggregations().get("by_index");

        JSONArray jsonArray = new JSONArray();

        for (Terms.Bucket bucket : tos.getBuckets()) {

            JSONObject hitJson = new JSONObject();

            String index = bucket.getKey().toString();

            // 大分类
            // 小分类
            if (p1IndexName.equals(index)) {
                hitJson.put("class", "xx数据");
                hitJson.put("subclass", "p1");
            } else {
                hitJson.put("class", "xx2数据");
                hitJson.put("subclass", index.substring(length));
            }

            hitJson.put("hitCount", bucket.getDocCount());

            jsonArray.add(hitJson);
        }

        Page<JSONObject> jsonPage = new Page<>(start, page.getPageSize(), (int) response.getHits().getTotalHits(), jsonsList);
        jsonPage.addProperty("hit", jsonArray);// 附加命中结果

        return jsonPage;

    
```
