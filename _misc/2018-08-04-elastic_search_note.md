---
layout: cnblog_post
title:  "elastic search note"
permalink: '/misc/elastic_search_note'
date:   2018-08-04 07:34:39
categories: misc
---

### 5.Elasticsearch集群管理

#### 5.1.索引管理

```
创建索引 PUT 索引名（只能小写）
如: PUT blog


创建索引的的时候指定设置：3分分片 0个副本
curl -H 'Content-Type: application/json' -XPUT localhost:9200/blog -d '
{
    "settings": {
        "number_of_shards": 3,
        "number_of_replicas": 0
    }
}
'

创建索引之后更改设置: 将blog的副本数设置为2
curl -H 'Content-Type: application/json' -XPUT localhost:9200/blog/_settings -d '
{
    "number_of_replicas": 2
}
'

读写权限设置:
blocks.read_only:true # 设置只读，不能写入
blocks.read:true # 禁止读
blocks.write:true # 禁止写

curl -H 'Content-Type: application/json' -XPUT localhost:9200/blog/_settings -d '
{
    "blocks.write": true
}   
'
此时再写入就会报错
curl -H 'Content-Type: application/json' -XPUT localhost:9200/blog/article/1 -d '
 {"title": "Java 虚拟机"}
 '

# 错误
{
    "error": {
        "root_cause": [
            {
                "type": "cluster_block_exception",
                "reason": "blocked by: [FORBIDDEN/8/index write (api)];"
            }
        ],
        "type": "cluster_block_exception",
        "reason": "blocked by: [FORBIDDEN/8/index write (api)];"
    },
    "status": 403
}

恢复索引的写入权限

curl -H 'Content-Type: application/json' -XPUT localhost:9200/blog/_settings -d '
{
    "blocks.write": false
}
'

查看索引的所有配置信息：
curl localhost:9200/blog/_settings\?pretty=true

# 结果是
{
  "blog" : {
    "settings" : {
      "index" : {
        "number_of_shards" : "3",
        "blocks" : {
          "write" : "false"
        },
        "provided_name" : "blog",
        "creation_date" : "1533393875236",
        "number_of_replicas" : "0",
        "uuid" : "r-jahOyFQbCeWRJ_AYVjtQ",
        "version" : {
          "created" : "6020399"
        }
      }
    }
  }
}

同时查看多个索引的设置信息：
GET blog,twitter/_settings

查看所有索引的设置信息：
GET _all/_settings



关闭和打开索引
POST blog/_close
POST blog/_open

关闭以test开头的索引
test*/_close

复制索引
_reindex 可以把文档从一个索引复制到另一个索引，但不会复制索引的配置信息,
_reindex 之前需要设置目标索引的分片数、副本数等信息。

POST _reindex
{
    "source": {"index": "blog"},
    "dest": {"index": "blog_news"}
}

例如： 把blog索引article类型下的title字段含有git关键字的文档复制到blog_new索引中：
POST _reindex
{
    "source": {
        "index": "blog",
        "type": "article",
        "query": {
            "term": {"title": "git"}
        }
    },
    "dest": {
        "index": "blog_news"
    }
}


收缩索引？？？？
索引别名？？？？
```

#### 5.2.文档管理

```
##### 5.2.1新建文档
PUT 必加 ID
POST 不加ID

PUT blog/article/1
POST blog/article

##### 获取文档
GET blog/article/1

检查文档是否存在
HEAD blog/article/1
存在放回200 不存在则返回404

HEAD还可以检查index是否存在，但是不能检查index的type是否存在
http HEAD localhost:9200/blog
HTTP/1.1 200 OK

http HEAD localhost:9200/blog/article
HTTP/1.1 405 Method Not Allowed

###### _mget 根据索引名、type名、id一次获取多个文档
GET _mget
{
    "docs": [
        {
            "_index": "blog",
            "_type": "article",
            "_id": "1"
        },
        {
            "_index": "twitter",
            "_type": "tweet",
            "_id": "2"
        }
    ]
}

如果是同一个index下的不同type，可以简写为：
GET blog/_mget
{
    "docs": [
        {
            "_type": "typea",
            "_id": "1"
        },
        {
            "_type": "typeb",
            "_id": "2"
        }
    ]
}
如果index和type都相同，可以简写为：
GET blog/article/_mget
{
    "docs": [
        {
            "_id": "1"
        },
        {
            "_id": "2"
        }
    ]
}
进一步简化
GET blog/article/_mget
{
    "ids": [
        "1", "2"
    ]
}

##### 更新文档
先创建一个文档：
http PUT :9200/test/type1/1 <<< '{
    "counter": 1,
    "tags": ["red"]
}'

对这个文档进行更新，把counter字段的值加上4，更新命令如下：
http POST :9200/test/type1/1/_update <<< '
{
    "script": {
        "inline": "ctx._source.counter += params.count",
        "lang": "painless",
        "params": {
            "count": 4
        }  
    }
}
'


对tags字段增加一个元素，
http POST :9200/test/type1/1/_update <<< '
{
    "script" : {
        "inline": "ctx._source.tags.add(params.tag)",
        "lang": "painless",
        "params": {
            "tag": "blue"
        }
    }
}
'

如果想给文档增加一个字段，可以执行下面的命令：
http POST :9200/test/type1/1/_update <<< '
{
    "script": "ctx._source.new_field = \"value_of_new_field\""
}
'

如：
http POST :9200/test/type1/1/_update <<< '
{
    "script": "ctx._source.age = 10"
}
'

可以移除一个字段:
http POST :9200/test/type1/1/_update <<< '
{
    "script": "ctx._source.remove(\"field_name\")"
}
'

删除tags数组中含有red的文档
http POST :9200/test/type1/1/_update <<< '
{
    "script": {
        "inline": "if (ctx._source.tags.contains(params.tag)) { ctx.op = \"delete\" } else { ctx.op = \"none\" }",
        "lang": "painless",
        "params": {
            "tag": "red"
        }
    }
}
'

upsert 操作
例子：存在文档和counter字段，则增加4，不存在则设置为1
http POST :9200/test/type1/1/_update <<< '
{
    "script": {
        "inline": "ctx._source.counter += params.count",
        "lang": "painless",
        "params": {"count": 4}
    },
    "upsert": {
        "counter": 1
    }
}
'

如果id为1的文档不存在，或者counter字段不存在，但是执行的时候没有使用upsert，会报错document_missing_exception，返回状态404 Not Found


##### 5.2.4 查询更新
对title中包含git关键字的文档增加一个category字段：
http POST :9200/blog/_update_by_query <<< '
{
    "script": {
        "inline": "ctx._source.category = params.category",
        "lang": "painless",
        "params": {
            "category": "git"
        }
    },
    "query": {
        "term": {"title": "git"}
    }
}
'

##### 5.2.5 删除文档
curl -XDELETE localhost:9200/blog/article/1
http DELETE :9200/blog/article/1

##### 5.2.6 查询删除
删除title字段包含Java的文档
http POST :9200/blog/_delete_by_query <<< '
{
    "query": {
        "match": {
            "title": "Java"
        }
    }
}
'

删除一个type下的所有文档，命令如下：
http POST :9200/blog/csdn/_delete_by_query <<< '
{
    "query": {
        "match_all": {}
    }
}
'

##### 5.2.7 批量操作
如同mget允许一次查询多个文档一样，Bulk API允许单一请求来实现多个文档的create、index、update或delete，Bulk API的使用方法如下：
```

#### 5.3.？？？？

地理坐标如何存入 [https://www.elastic.co/guide/en/elasticsearch/reference/1.4/mapping-geo-point-type.html](https://www.elastic.co/guide/en/elasticsearch/reference/1.4/mapping-geo-point-type.html)

```
1.设置字段类型为geo_point
http PUT :9200/test <<< '
{
    "mappings": {
        "type1": {
            "properties": {
                "lo": {"type": "geo_point"}
            }
        }
    }
}
'
2.四种插入数据的方法
①经纬度坐标键值对
http PUT :9200/test/type1/1 <<< '
{
    "lo": {
        "lat":41.12,
        "lon":-71.34
    }
}
'
② lat,lon
{
    "lo": "41.12,-71.34"
}
③geohash
{
    "lo": "drm3btev3e86"
}
④[lon, lat]
{
    "lo" : [-71.34, 41.12]
}
```


```
##### 5.3.6元字段
5.3.7映射参数？？？？
5.3.8映射模板？？？？
```



token_count [https://www.elastic.co/guide/en/elasticsearch/reference/1.4/mapping-core-types.html#token_count](https://www.elastic.co/guide/en/elasticsearch/reference/1.4/mapping-core-types.html#token_count)

```
http PUT :9200/test_token_count <<< '
{
    "mappings": {
        "type1": {
            "properties": {
                "name": {
                    "type": "text",
                    "fields": {
                        "length": {
                            "type": "token_count",
                            "analyzer": "standard"
                        }
                    }
                }
            }
        }
    }
}
'

http PUT :9200/test_token_count/type1/1 <<< '
{
    "name": "John Smith"
}
'

http PUT :9200/test_token_count/type1/2 <<< '
{
    "name": "Rachel Alice Williams"
}
'

http :9200/test_token_count/_search <<< '
{
    "query": {
        "term": {
            "name.length": 2
        }
    }
}'
```


### 查询

创建index

```
ik 问题
https://blog.csdn.net/wang_shen_tao/article/details/52834106


命令如下:
http PUT :9200/books <<< '
{
    "settings": {
        "number_of_replicas": 1,
        "number_of_shards": 3
    },
    "mappings": {
        "IT": {
            "properties": {
                "id": {
                    "type": "long"
                },
                "title": {
                    "type": "text",
                    "analyzer": "ik_max_word"
                },
                "language": {
                    "type": "keyword"
                },
                "author": {
                    "type": "keyword"
                },
                "price": {
                    "type": "double"
                },
                "year": {
                    "type": "date",
                    "format": "yyyy-MM-dd"
                },
                "description": {
                    "type": "text",
                    "analyzer": "ik_max_word"
                }
            }
        }
    }
}
'
```

导入数据

```
# curl
curl -XPOST 'localhost:9200/_bulk?pretty' --data-binary @books.json
# http
http POST 'localhost:9200/_bulk?pretty' @books.json
```

查询全部

```
http :9200/books/_search <<< '
{
    "query": {
        "match_all": {}
    }
}
'
```

可以简化为:

```
http :9200/books/_search
```

term 查询用来查找指定字段中包含给定单词的文档，term查询不被解析，只有查询词和文档中的词精确匹配才会被搜索到，应用场景为查询人名、地名等需要精确匹配的需求。

比如：查询title字段中含有关键词'思想'的书籍,查询的命令为：

```
http :9200/books/_search <<< '
{
    "query": {
        "term": {"title": "思想"}
    }
}
'
```

返回:

```
{
    "_shards": {
        "failed": 0,
        "skipped": 0,
        "successful": 3,
        "total": 3
    },
    "hits": {
        "hits": [
            {
                "_id": "1",
                "_index": "books",
                "_score": 0.6931472,
                "_source": {
                    "author": "Bruce Eckel",
                    "description": "Java学习必读经典,殿堂级著作！赢得了全球程序员的广泛赞誉。",
                    "id": "1",
                    "language": "java",
                    "price": 70.2,
                    "publish_time": "2007-10-01",
                    "title": "Java编程思想"
                },
                "_type": "IT"
            }
        ],
        "max_score": 0.6931472,
        "total": 1
    },
    "timed_out": false,
    "took": 8
}
```

如果匹配项太多，需要将结果分页，用到两个属性：from和size

```
http :9200/books/_search <<< '
{
    "query": {
        "term": {"title": "思想"}
    },
    "from": 0,
    "size": 100
}
'
```

只返回需要的字段，而不显示全部字段<br>
如查询标题包含java的书籍，只返回title和price字段

```
http :9200/books/_search <<< '
{
    "query": {
        "term": {"title": "java"}
    },
    "_source": ["title", "author"]
}
'
```

默认返回结果中不包含文档版本，如果需要，可以在查询体重设置version为true:

```
http :9200/books/_search <<< '
{
    "query": {
        "term": {"title": "java"}
    },
    "version": true
}
'
```

过滤掉评分低的, 查询评分不低于0.6的文档

```
http :9200/books/_search <<< '
{
    "query": {
        "term": {"title": "java"}
    },
    "min_score": 0.6
}
'
```

高亮查询关键字：

```
http :9200/books/_search <<< '
{
    "query": {
        "term": {"title": "java"}
    },
    "highlight": {
        "fields": {
            "title": {}
        }
    }
}
'

## 结果中的一个文档
{
    "_id": "1",
    "_index": "books",
    "_score": 0.6931472,
    "_source": {
        "author": "Bruce Eckel",
        "description": "Java学习必读经典,殿堂级著作！赢得了全球程序员的广泛赞誉。",
        "id": "1",
        "language": "java",
        "price": 70.2,
        "publish_time": "2007-10-01",
        "title": "Java编程思想"
    },
    "_type": "IT",
    "highlight": {
        "title": [
            "<em>Java</em>编程思想"
        ]
    }
}
```

#### 对比term-query和 match-query

```
GET books/_search
{
    "query": {
        "term": {
            "title": "java编程"
        }
    }
}

GET books/_search
{
    "query": {
        "match": {
            "title": "java编程"
        }
    }
}
```

假设原始数据为"Java编程思想"，数据经过分词后有“Java” “编程” “思想”，这里的每一个term都无法与term查询的条件“java编程”匹配。 而match查询首先会将查询条件分词，然后查询条件中的每一个词项与文档匹配就会搜索到，因此相当于多个term查询。<br>
因此同样的查询条件term查询能查到的文档，match查询一定能查到。<br>
term查询是条件与数据词项的碰撞，match查询时条件分词后的词项与数据词项的碰撞。

因为任意的match条件词项匹配都会返回， 如果想查询匹配所有的关键词，可以用and

```
http :9200/books/_search <<< '
{
    "query": {
        "match": {
            "title": {
                "query": "java编程思想",
                "operator": "and"
            }
        }
    }
}
'
```
这其实相当于查询match-query “java”、match-query “编程”、match-query “思想”都同时满足的文档，仍然与term查询不同，因为term查询始终查询文档的一个词项值为“java编程思想”的文档。

另一方面：以下三种语法结果是一致的:

```
"match": {
    "title": "java编程思想"
}

"match": {
    "title": {
        "query": "java编程思想",
        "operator": "or"
    }
}

"match": {
    "title": {
        "query": "java编程思想"
    }
}

```





#### 词项查询

##### terms
terms是term查询的升级，可以查询文档中包含多个词的文档。
如：想查询title中包含关键字java或python的文档

```
http :9200/books/_search <<< '
{
    "query": {
        "terms": {
            "title": ["java", "python"]
        }
    }
}
'
```

其实也可以使用match实现这个功能：

```
http :9200/books/_search <<< '
{
    "query": {
        "match": {
            "title": "java python"
        }
    }
}
'
```

##### range query

适用于数值型、日期类型、字符串类型字段的文档，range查询支持参数有： gt、gte、lt、lte

查询价格大于50小于等于70的书籍：

```
http :9200/books/_search <<< '
{
    "query": {
        "range": {
            "price": {
                "gt": 50,
                "lte": 70
            }
        }
    }
}
'
```

查询出版日期在2016年1月1日和2016年12月31日之间的书籍:

```
http :9200/books/_search <<< '
{
    "query": {
        "range": {
            "publish_time": {
                "gte": "2016-01-01",
                "lte": "2016-12-31",
                "format": "yyyy-MM-dd"
            }
        }
    }
}
'


# 可以使用时间戳代替字符串，必须以毫秒为单位
http :9200/books/_search <<< '
{
    "query": {
        "range": {
            "publish_time": {
                "gte": 1451577600000,
                "lte": 1483113600000
            }
        }
    }
}
'
```

##### exists query

exists 查询会返回字段中至少有一个非空值的文档：

```
http :9200/books/_search <<< '
{
    "query": {
        "exists": {
            "field": "author"
        }
    }
}
'
```

##### prefix query

查询description字段分词以java为前缀的文档：

```
http :9200/books/_search <<< '
{
    "query": {
        "prefix": {
            "description": "java"
        }
    }
}
'
```

##### wildcard query

查询author分词以'张'开头的文档

```
http :9200/books/_search <<< '
{
    "query": {
        "wildcard": {
            "author": "张*"
        }
    }
}
'
```

##### regexp query

title词项中含有python或java的书籍：

```
http :9200/books/_search <<< '
{
    "query": {
        "regexp": {
            "title": "(python|java)"
        }
    }
}
'
```

##### fuzzy query

消耗资源较大， 即使拼写错了也能查到

```
http :9200/books/_search <<< '
{
    "query": {
        "fuzzy": {
            "title": "javascritp"
        }
    }
}
'
```


##### type query

```
http :9200/books/_search <<< '
{
    "query": {
        "type": {
            "value": "IT"
        }
    }
}
'

# http :9200/books/IT/_search
```

##### ids query

指定多个id

```
http :9200/books/_search <<< '
{
    "query": {
        "ids": {
            "type": "IT",
            "values": ["1", "3", "5"]
        }
    }
}
'

http :9200/books/IT/_search <<< '
{
    "query": {
        "ids": {
            "values": ["1", "3", "5"]
        }
    }
}
'
```

#### 复合查询

##### constant_score_query

可以包装一个其他类型的查询，返回匹配过滤器中的查询条件且具有相同评分的文档。<br>
如：查询title中含有java的文档，所有文档的评分都是1.2

```
http :9200/books/IT/_search <<< '
{
    "query": {
        "constant_score": {
            "filter": {
                "term": {"title": "java"}
            },
            "boost": 1.2
        }
    }
}
'
```

##### bool query

组合多个查询项：<br>
must 相当于逻辑AND<br>
should 相当于逻辑OR<br>
must_not 相当于逻辑NOT<br>
filter和must一样, 但是filter不评分,只起到过滤的作用

```
http :9200/books/IT/_search <<< '
{
    "query": {
        "bool": {
            "minimum_should_match": 1,
            "must": {
                "match": {"title": "java"}
            },
            "should": [
                {"match": {"description": " 虚拟机"}}
            ],
            "must_not": {
                "range": {"price": {"gte": 70}}
            }
        }
    }
}
'
```



















