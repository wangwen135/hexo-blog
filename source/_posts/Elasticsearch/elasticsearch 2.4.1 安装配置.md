
---
title: elasticsearch 2.4.1 安装配置
date: 2017-03-05
tags: 
  - Elasticsearch
categories:
  - [Elasticsearch]
---



<https://www.elastic.co/products/elasticsearch>

验证环境：

**Elasticsearch 2.4.1**\
**CentOS Linux release 7.2.1511 (Core)**

## 安装

### YUM安装

下载并安装公钥\
rpm --import <https://packages.elastic.co/GPG-KEY-elasticsearch>

在目录 /etc/yum.repos.d/ 下添加一个文件 elasticsearch.repo   内容如下：

    [elasticsearch-2.x]
    name=Elasticsearch repository for 2.x packages
    baseurl=https://packages.elastic.co/elasticsearch/2.x/centos
    gpgcheck=1
    gpgkey=https://packages.elastic.co/GPG-KEY-elasticsearch
    enabled=1

安装：

    yum install elasticsearch

开机启动：

    sudo /bin/systemctl daemon-reload
    sudo /bin/systemctl enable elasticsearch.service

### 解压安装

去官网下载合适的tar包\
<https://www.elastic.co/downloads/elasticsearch>

    # crul 下载
    curl -O https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.4.1/elasticsearch-2.4.1.tar.gz
    # ---------------------------------------------------
    # wget 下载
    # 如果没有wget需要先安装
    yum install wget

    ## 下载
    wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.4.1/elasticsearch-2.4.1.tar.gz

    ## 解压
    tar -xvf elasticsearch-2.4.1.tar.gz 

## 配置

修改配置文件config/elasticsearch.yml

    # ---------------------------------- Cluster -----------------------------------
    #配置集群名称
    cluster.name: wwh_es_cluster

    # ------------------------------------ Node ------------------------------------
    #配置节点名称，不同节点名字需要不同
    node.name: node-213

    # ----------------------------------- Paths ------------------------------------
    # Path to directory where to store the data (separate multiple locations by comma):
    #
    # path.data: /path/to/data
    #
    # Path to log files:
    #
    # path.logs: /path/to/logs

    # ---------------------------------- Network -----------------------------------
    # Set the bind address to a specific IP (IPv4 or IPv6):

    network.host: 192.168.1.213

    # Set a custom port for HTTP:
    #
    # http.port: 9200
    #

    # --------------------------------- Discovery ----------------------------------
    #集群中可以作为master节点的初始列表，通过这些节点来自动发现新加入集群的节点
    discovery.zen.ping.unicast.hosts: ["wwh213", "wwh214"]

    #其他的根据需要来配置

***

复制到其他机器（可以安装完其他的再一起复制）

```
scp -r elasticsearch-2.4.1 wwh214:/data/es/
scp -r elasticsearch-2.4.1 wwh215:/data/es/

```

**然后修改为不同的节点名称**

## 插件

### elasticsearch-head

ElasticSearch-Head 是一个与Elastic集群（Cluster）相交互的Web前台。\
ES-Head的主要作用

*   它展现ES集群的拓扑结构，并且可以通过它来进行索引（Index）和节点（Node）级别的操作
*   它提供一组针对集群的查询API，并将结果以json和表格形式返回
*   它提供一些快捷菜单，用以展现集群的各种状态

官网地址：\
<https://github.com/mobz/elasticsearch-head>\
<http://mobz.github.io/elasticsearch-head/>

安装：

    bin/plugin -install mobz/elasticsearch-head

    or

    bin/plugin install mobz/elasticsearch-head

访问地址：\
<http://192.168.1.213:9200/_plugin/head/>

### Kibana 4.6.1

Kibana是一个开源的数据可视化平台，使您可以通过惊艳、强大的图形结合定制的仪表板，帮助您对数据进行分析与数据进行交互。\
Kibana 4.6.1 与Elasticsearch 2.4.x.兼容\
Kibana也可以使用apt或yum安装\
<https://www.elastic.co/guide/en/kibana/4.6/setup.html#setup-repositories>

安装：

```
# 下载
wget https://download.elastic.co/kibana/kibana/kibana-4.6.1-linux-x86_64.tar.gz

# 解压
tar -xvf kibana-4.6.1-linux-x86_64.tar.gz 

```

编辑 config/kibana.yml 文件

    #设置elasticsearch.url 指向 Elasticsearch 实例  

    elasticsearch.url: "http://192.168.1.213:9200"

运行 ./bin/kibana 启动\
在浏览器中打开：\
<http://192.168.1.213:5601>

### Marvel 2.0+

Marvel是Elasticsearch的图形化监控客户端，可以用来查看当前的各项状态。优化您的Elasticsearch性能和快速诊断问题的工具。

*现在的elasticsearch 更改了插件安装方式，marvel是在kibana里安装的，而不是在elasticsearch里安装的，elasticsearch里安装的只是marvel-agent.*

*Marvel 2.0+ 兼容最新版本的Elasticsearch 和 Kibana*\
*Marvel 1.3 兼容 Elasticsearch 1.0 - 1.7*

官网地址：\
<https://www.elastic.co/products/marvel>\
<https://www.elastic.co/downloads/marvel>

安装：

```
#安装 Marvel 到 Elasticsearch 中
# 这个license只有30天的有效期，之后需要升级

./plugin install license
./plugin install marvel-agent

#安装 Marvel 到 Kibana 中

bin/kibana plugin --install elasticsearch/marvel/latest

```

启动 Elasticsearch 和 Kibana\
访问： <http://192.168.1.213:5601/app/marvel>

***

需要在每台机器上都安装

## 启动

**不能直接以root用户进行启动**\
**修改用户之后需要注意权限问题**\
添加用户并修改权限

    #添加用户
    adduser es  

    #修改权限
    chown -R es:es elasticsearch-2.4.1

    su es

或者指定 -Des.insecure.allow\.root=true

```
# -d参数表示在后台启动
bin/elasticsearch -d

```

查看端口是否启动

    netstat -an | grep 9200
    tcp6       0      0 ::1:9200                :::*                    LISTEN     
    tcp6       0      0 127.0.0.1:9200          :::*                    LISTEN    

    netstat -an | grep 9300
    tcp6       0      0 ::1:9300                :::*                    LISTEN     
    tcp6       0      0 127.0.0.1:9300          :::*                    LISTEN  

## 测试

通过命令

    curl -X GET http://192.168.1.213:9200/

如果安装了 elasticsearch-head 直接在浏览器中打开：\
<http://192.168.1.213:9200/_plugin/head/>

***

## 配置ElasticSearch使用内存

编辑文件：
\$ES\_HOME/bin/elasticsearch.in.sh

    if [ "x$ES_MIN_MEM" = "x" ]; then
        ES_MIN_MEM=10g  //最小内存，根据机器内存来定
    fi
    if [ "x$ES_MAX_MEM" = "x" ]; then
        ES_MAX_MEM=36g  //最大内存，根据机器内存来定，最好不要超过机器物理内存的50%
    fi

