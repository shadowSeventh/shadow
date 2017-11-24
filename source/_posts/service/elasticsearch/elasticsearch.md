---
title: elasticsearch
date: 2017-11-20 10:34:22
categories: 服务器
tags: [elasticsearch]
description: "elasticsearch docker安装以及常用操作"
---

### 安装
```bash
docker pull elasticsearch

docker run -itd \
    --name my-es \
    -p 9200:9200 \
    -p 9300:9300 \
    -v /Users/lit/docker/elasticsearch/es-conf/:/usr/share/elasticsearch/config/ \
    -v /Users/lit/docker/elasticsearch/es-data/:/usr/share/elasticsearch/data/:rw \
    elasticsearch

docker start my-es

docker exec -it my-es bash

vi /Users/lit/docker/elasticsearch/es-conf/elasticsearch.yml
cluster.name: "my-es"   # 节点名如果不设置，会自动随机生成
node.name: "local"
```


    
测试 Elasticsearch 是否启动成功

```bash
curl 'http://localhost:9200/?pretty'
```
### RESTful API
```bash
#计算集群中文件的数量
    curl -XGET 'http://localhost:9200/_count?pretty' -d '
    {  
        "query": {
            "match_all": {}
        }
    }'
```
### 索引
在 Elasticsearch 中，文档属于一种 类型(type)，各种各样的类型存在于一个 索引 中。你也可以通过类比传统的关系数据库得到一些大致的相似之处：

```bash
关系数据库     ⇒ 数据库 ⇒ 表    ⇒ 行    ⇒ 列(Columns)
Elasticsearch  ⇒ 索引   ⇒ 类型  ⇒ 文档  ⇒ 字段(Fields)
一个 Elasticsearch 集群可以包含多个
```
索引（数据库），也就是说其中包含了很多 类型（表）。这些类型中包含了很多的 文档（行），然后每个文档中又包含了很多的 字段（列）。
```bash
#插入示例：
    curl -XPUT 'http://localhost:9200/megacorp/employee/1?pretty' -d '
    {
        "first_name" : "John",
        "last_name" :  "Smith",
        "age" :        25,
        "about" :      "I love to go rock climbing",
        "interests": [ "sports", "music" ]
    }'	
# 会得到类似返回值
    {
      "_index" : "megacorp",
      "_type" : "employee",
      "_id" : "1",
      "_version" : 1,
      "result" : "created",
      "_shards" : {
        "total" : 2,
        "successful" : 1,
        "failed" : 0
      },
      "created" : true
    }
#检索示例：
    curl -XGET 'http://localhost:9200/megacorp/employee/1?pretty'
# 会得到类似返回值
    {
      "_index" : "megacorp",
      "_type" : "employee",
      "_id" : "1",
      "_version" : 2,
      "found" : true,
      "_source" : {
        "first_name" : "Douglas",
        "last_name" : "Fir",
        "age" : 35,
        "about" : "I like to build cabinets",
        "interests" : [
          "forestry"
        ]
      }
    }
#删除示例
    curl -XDELETE 'http://localhost:9200/megacorp/employee/3?pretty'
# 全部搜索
    curl -XGET 'http://localhost:9200/megacorp/employee/_search?pretty'
# 关键字搜索
    curl -XGET 'http://localhost:9200/megacorp/employee/_search?q=last_name:Fir&pretty'
```
### Query DSL搜索
```bash
curl -XGET 'http://localhost:9200/megacorp/employee/_search?pretty' -d '
    {
        "query" : {
            "match" : {
                "last_name" : "Smith"
            }
        }
    }'
```
    
    
    