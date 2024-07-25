---
title: Elasticsearch-JDBC-导入器
date: 2019-08-08 23:41
tags: 
  - Elasticsearch
categories:
  - [Elasticsearch]
---

# Elasticsearch JDBC 导入器

通过Java数据库连接（JDBC）从JDBC源获取数据导入到Elasticsearch中。  

项目地址：  
https://github.com/jprante/elasticsearch-jdbc

## 问题
使用 1.7.0_80 版本的JDK报错：Unsupported major.minor version 52.0  

换成 1.8.0_101 版本的JDK之后就可以了。  
据说是JDK本身的一个问题。

## 操作过程
简单的记录操作过程，详情见github。

1. 下载
```
    wget http://xbib.org/repository/org/xbib/elasticsearch/importer/elasticsearch-jdbc/2.3.4.0/elasticsearch-jdbc-2.3.4.0-dist.zip
```
2. 解压缩  
```
    unzip elasticsearch-jdbc-2.3.4.0-dist.zip 
```
3. 确定JDBC驱动jar  
    检查lib目录是否有你需要的jdbc驱动jar，如果没有需要将相关jar放到该目录中。

4. 编写一个导入脚本  
```
#!/bin/sh
DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
bin=${DIR}/../bin
lib=${DIR}/../lib
echo '{
    "type" : "jdbc",
    "jdbc" : {
        "url" : "jdbc:mysql://192.168.1.212:3306/hxx",
        "user" : "root",
        "password" : "root",
        "sql" : "SELECT *, id as _id FROM xxtable",
        "index" : "test",
        "type" : "rt1",
        "metrics": {
            "enabled" : true
        },
        "elasticsearch" : {
             "cluster" : "wwh_es_cluster",
             "host" : "192.168.1.213",
             "port" : 9300
        }
    }
}' | java \
       -cp "${lib}/*" \
       -Dlog4j.configurationFile=${bin}/log4j2.xml \
       org.xbib.tools.Runner \
       org.xbib.tools.JDBCImporter
~                                      
```
5. 给脚本添加执行权限然后执行
