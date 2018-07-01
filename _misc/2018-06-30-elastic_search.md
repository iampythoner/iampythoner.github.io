---
layout: cnblog_post
title:  "elastic search"
permalink: '/misc/elastic_search'
date:   2018-06-30 07:34:39
categories: misc
---

###### 重要技术点
基本操作
集群搭建

有关分片shard（primary shard、replica shard）的介绍[https://www.jianshu.com/p/123bbfb2ad7f](https://www.jianshu.com/p/123bbfb2ad7f)


es版本6.2.3

#### Index相关操作

查看所有的索引`curl -X GET 'http://localhost:9200/_cat/indices?v'`

```
health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   weather  zaIxSytBQuOQrMQyhCQUtw   5   1          0            0      1.1kb          1.1kb
green  open   .kibana  rh_Bbb40TRWFnTXi09fVEg   1   0          2            1     10.8kb         10.8kb
yellow open   accounts vIH6BOKpQeCN8ZHZcz4iaQ   5   1          2            0     10.2kb         10.2kb
```
去掉querystring里的`v`则不会显示 `health status...`这些header

查看每个index所有的type `curl 'localhost:9200/_mapping?pretty=true'`

新建Index `curl -X PUT 'localhost:9200/weather'`
返回

```json
{
    "acknowledged":true,
    "shards_acknowledged":true,
    "index":"weather"
}
```

删除Index`curl -X DELETE 'localhost:9200/weather'`
结果`{"acknowledged":true}`


#### Document相关操作

##### 新增记录

向index: accounts _type:person 增加一条新的记录：

```
curl -H "Content-Type: application/json" -X PUT 'localhost:9200/accounts/person/1' -d '
{
    "user": "张三",
    "title": "工程师",
    "desc": "数据库管理"
}
'
```

返回

```json
{
    "_index": "accounts",
    "_type": "person",
    "_id": "1",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 0,
    "_primary_term": 1
}
```

请求路径`/accounts/person/1` 中的 `1`是该记录的id，也可以不指定，如果不指定要改成POST请求(ES开发者对RESTful理解之深！！！[stackoverflow](https://stackoverflow.com/questions/630453/put-vs-post-in-rest?page=1&tab=votes#tab-top)、[w3](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9.5))

使用POST创建一条Document: 

```
curl -H "Content-Type: application/json" -X POST 'localhost:9200/accounts/person' -d '
{
    "user": "李四",
    "title": "工程师",
    "desc": "系统管理"
}
'
```

返回:

```json
{
    "_index": "accounts",
    "_type": "person",
    "_id": "RTN6T2QB7yPpCHMAyqJy",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 0,
    "_primary_term": 1
}
```

这里的id是一个字符串,注意，如果没有先创建 Index（这个例子是accounts），直接执行上面的命令，Elastic 也不会报错，而是直接生成指定的 Index。所以，打字的时候要小心，不要写错 Index 的名称。

##### 查看记录

`curl 'localhost:9200/accounts/person/1?pretty=true'`

```json
{
  "_index" : "accounts",
  "_type" : "person",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "user" : "张三",
    "title" : "工程师",
    "desc" : "数据库管理"
  }
}
```

返回的数据中，found字段表示查询成功，_source字段返回原始记录,
如果Id 不正确，就查不到数据，found字段就是false。

`curl 'localhost:9200/accounts/person/abc?pretty=true'`

```json
{
  "_index" : "accounts",
  "_type" : "person",
  "_id" : "abc",
  "found" : false
}
```

这是另一种错误： index名字输入错误: `curl 'localhost:9200/account/person/abc?pretty=true'`

```json
{
  "error" : {
    "root_cause" : [
      {
        "type" : "index_not_found_exception",
        "reason" : "no such index",
        "resource.type" : "index_expression",
        "resource.id" : "account",
        "index_uuid" : "_na_",
        "index" : "account"
      }
    ],
    "type" : "index_not_found_exception",
    "reason" : "no such index",
    "resource.type" : "index_expression",
    "resource.id" : "account",
    "index_uuid" : "_na_",
    "index" : "account"
  },
  "status" : 404
}
```

可以看到与找不到结果不到，这里直接是给出404的状态码和错误信息。

##### 删除记录

`curl -X DELETE 'localhost:9200/accounts/person/1'`

```json
{
    "_index": "accounts",
    "_type": "person",
    "_id": "1",
    "_version": 2,
    "result": "deleted",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 1,
    "_primary_term": 1
}
```

如果没有找到指定的id：

```json
{
  "_index" : "accounts",
  "_type" : "person",
  "_id" : "1",
  "_version" : 3,
  "result" : "not_found",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}
```

差别在于`result`字段对结果的描述: 成功:`deleted`, 未找到:`not_found`


##### 更新记录

对accounts索引的person _type中id为1的document进行更新：

```
curl -H "Content-Type: application/json" -X PUT 'localhost:9200/accounts/person/1' -d '
{
    "user": "张三他哥",
    "title": "工程师",
    "desc": "DBA"
}
'
```


```json
{
    "_index": "accounts",
    "_type": "person",
    "_id": "1",
    "_version": 2,
    "result": "updated",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 4,
    "_primary_term": 1
}
```

虽然和新增的操作相同，但是这里的`result`值为`updated`。

注意：<br/>
①使用POST方法更新也是可以的，请求路径和参数完全一致。<br/>
②这种更新方式(POST、PUT都这样)会使用请求体替换原文档，如上面若使用`'{"user":"张三他哥","title":"工程师"}'`更新，会删除原有的`desc`字段。


#### 数据查询

##### 返回所有记录

请求的格式是 `/Index/Type/_search`

如`curl 'localhost:9200/accounts/person/_search?pretty=true'`

返回:

```json
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 2,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "accounts",
        "_type" : "person",
        "_id" : "RTN6T2QB7yPpCHMAyqJy",
        "_score" : 1.0,
        "_source" : {
          "user" : "李四",
          "title" : "工程师",
          "desc" : "系统管理"
        }
      },
      {
        "_index" : "accounts",
        "_type" : "person",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "user" : "张三",
          "title" : "工程师",
          "desc" : "数据库管理,软件开发"
        }
      }
    ]
  }
}
```

结果中，各个增加的字段释义如下：

| -- | -- |
| 字段 | 含义 |
| -- | -- |
| took | 该操作的耗时（单位为毫秒） |
| timed_out | 是否超时 |
| hits | 命中的记录 |
| hits.total | 返回记录数 |
| hits.max_score | 最高的匹配程度 (0~1.0)|
| hits.hits | 命中的记录数组 |
| hits.hits.$._source | 记录元数据 |

##### 按条件查询

###### 全文搜索

请求：匹配条件是desc字段里面包含"软件"这个词

```
curl -XPOST -H 'Content-Type:application/json'  'localhost:9200/accounts/person/_search?pretty=true' -d '
{
    "query": {
        "match": {
            "desc": "软件"
        }
    }
}
'
```

返回:

```json
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 0.5753642,
    "hits" : [
      {
        "_index" : "accounts",
        "_type" : "person",
        "_id" : "1",
        "_score" : 0.5753642,
        "_source" : {
          "user" : "张三",
          "title" : "工程师",
          "desc" : "数据库管理,软件开发"
        }
      }
    ]
  }
}
```

Elastic 默认一次返回10条结果，可以通过size字段改变这个设置,如:指定每次只返回一条结果

```
curl -XPOST -H 'Content-Type:application/json'  'localhost:9200/accounts/person/_search?pretty=true' -d '
{
    "query": {
        "match": {
            "desc": "软件"
        }
    },
    "size": 1
}
'
```

还可以通过from字段，指定位移

```
curl -XPOST -H 'Content-Type:application/json'  'localhost:9200/accounts/person/_search?pretty=true' -d '
{
    "query": {
        "match": {
            "desc": "管理"
        }
    },
    "from": 1,
    "size": 1
}
'
```

返回:

```json
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 2,
    "max_score" : 0.5753642,
    "hits" : [
      {
        "_index" : "accounts",
        "_type" : "person",
        "_id" : "1",
        "_score" : 0.5753642,
        "_source" : {
          "user" : "张三",
          "title" : "工程师",
          "desc" : "数据库管理,软件开发"
        }
      }
    ]
  }
}
```

可以看到`hits.total`返回的是匹配的总数目，而`hits.hits`是本次查询返回的数据,个数还要用`hits.hits.length`。

查看其它查询 [https://www.elastic.co/guide/en/elasticsearch/reference/5.5/query-dsl.html](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/query-dsl.html)

match 语法 [https://www.elastic.co/guide/en/elasticsearch/reference/5.5/query-dsl-match-query.html](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/query-dsl-match-query.html)  


###### 逻辑运算

如果有多个搜索关键字， Elastic 认为它们是`or` 关系， 如：

```
curl -XPOST -H 'Content-Type:application/json'  'localhost:9200/accounts/person/_search?pretty=true' -d '
{
    "query": {
        "match": {
            "desc": "软件 系统"
        }
    }
}
'
```

结果：

```
{
  "took" : 7,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 2,
    "max_score" : 0.5753642,
    "hits" : [
      {
        "_index" : "accounts",
        "_type" : "person",
        "_id" : "RTN6T2QB7yPpCHMAyqJy",
        "_score" : 0.5753642,
        "_source" : {
          "user" : "李四",
          "title" : "工程师",
          "desc" : "系统管理"
        }
      },
      {
        "_index" : "accounts",
        "_type" : "person",
        "_id" : "1",
        "_score" : 0.5753642,
        "_source" : {
          "user" : "张三",
          "title" : "工程师",
          "desc" : "数据库管理,软件开发"
        }
      }
    ]
  }
}
```

如果想使用`and`查询，必须使用bool查询([文档](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/query-dsl-bool-query.html))，如：


```
curl -XPOST -H 'Content-Type:application/json'  'localhost:9200/accounts/person/_search?pretty=true' -d '
{
    "query": {
        "bool": {
            "must": [
                {"match": {"desc": "数据库"}},
                {"match": {"desc": "开发"}}
            ]
        }
    }
}
'
```

返回:

```
{
  "took" : 14,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 1.4384105,
    "hits" : [
      {
        "_index" : "accounts",
        "_type" : "person",
        "_id" : "1",
        "_score" : 1.4384105,
        "_source" : {
          "user" : "张三",
          "title" : "工程师",
          "desc" : "数据库管理,软件开发"
        }
      }
    ]
  }
}
```




