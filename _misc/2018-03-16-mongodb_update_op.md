---
layout: cnblog_post
title:  "mongodb_update_op"
permalink: '/misc/mongodb_update_op'
date:   2018-03-16 08:34:39
categories: misc
---

#### 常用的修改器 $inc $set $unset $push $pop $upsert

##### $inc
```
# 文档示例
{"uid":"201203","type":"1",size:10}

# 插入一条
db.b.insert({"uid":"201203","type":"1",size:10})

# 增1
db.b.update({"uid" : "201203"},{"$inc":{"size" : 1}})
# db.b.find()
{ "_id" : ObjectId("5003b6135af21ff428dafbe6"), "uid" : "201203", "type" : "1",
"size" : 11 }
# 增2
db.b.update({"uid" : "201203"},{"$inc":{"size" : 2}})
# db.b.find()
{ "_id" : ObjectId("5003b6135af21ff428dafbe6"), "uid" : "201203", "type" : "1",
"size" : 13 }
# 减1
db.b.update({"uid" : "201203"},{"$inc":{"size" : -1}})
# db.b.find()
{ "_id" : ObjectId("5003b6135af21ff428dafbe6"), "uid" : "201203", "type" : "1",
"size" : 12 }
```

##### $set

用来指定一个键并更新键值，若键不存在并创建。来看看下面的效果：<br>
(为了方便查看不再显示文档id了)

```
# db.a.findOne({"uid" : "20120002","type" : "3"})
{ "desc" : "hello world2!", "sname" : "jk", "type" : "3", "uid" : "20120002" }

### 如果字段不存在，使用$set直接插入
# $set size
db.a.update({"uid" : "20120002","type" : "3"},{"$set":{"size":10}})
# db.a.findOne({"uid" : "20120002","type" : "3"})
{"desc" : "hello world2!", "size" : 10, "sname" : "jk", "type" : "3", "uid" : "20120002" }

## 如果字段存在，会修改
# $set sname
db.a.update({"uid" : "20120002","type" : "3"},{"$set":{"sname":"ssk"}})
# find
{ "desc" : "hello world2!", "size" : 10, "sname" : "ssk", "type" : "3", "uid" : "20120002" }


## 可以通过$set直接将字段值设置为另一种类型的值
db.a.update({"uid" : "20120002","type" : "3"},{"$set":{"sname":["java",".net","c++"]}})
# find
"sname" : ["java",".net","c++"],

## 设置下一级的字段
# 准备数据：db.c.findOne({"name":"toyota"})
{
        "_id" : ObjectId("5003be465af21ff428dafbe7"),
        "name" : "toyota",
        "type" : "suv",
        "size" : {
                "height" : 10,
                "width" : 5,
                "length" : 15
        }
}
# 设置size.height
db.c.update({"name":"toyota"},{"$set":{"size.height":8}})
# 结果
{
        "_id" : ObjectId("5003be465af21ff428dafbe7"),
        "name" : "toyota",
        "type" : "suv",
        "size" : {
                "height" : 8,
                "width" : 5,
                "length" : 15
        }
}
```


##### $unset
主要是用来删除键

```
db.a.update({"uid" : "20120002","type" : "3"},{"$unset":{"sname":1}})

sname: 接的值可以是 1 -1 0 或其他任意字符，都是讲sname这个字段删除
```

##### $push 向数组添加元素


```
## 准备数据：
{"name" : "toyota" }

## 先push一个当前文档中不存在的键title
db.t.update({"name" : "toyota"},{$push:{"title":"t1"}})
## 结果是：title是一个数组，并且有了元素
{"name" : "toyota", "title" : [ "t1" ]}

## 再向title中push一个值
db.t.update({"name" : "toyota"},{$push:{"title":"t2"}})
# 结果增加了一个新push的元素
{"name" : "toyota", "title" : [ "t1" , "t2"]}

## 不能向非数组类型push否则报错
Cannot apply $push/$pushAll modifier to non-array
```


##### $ne $addToSet 不向数组添加重复的元素


```
# 准备数据
{"name" : "toyota", "title" : [ "t1" , "t2"]}

# $ne
db.t.update({title: {$ne: "t2"}}, {$push: {"title": "t2"}})
# 结果没有改变，数组保持原来的值

# addToSet
db.t.update({title: "toyota"}, {$addToSet: {title: "t2"}})
# 结果也没有什么改变
```

##### 数组修改器：$pop $pull 都是删除

###### $pop 按照索引删除数组元素

```
# 准备数据
{"name" : "toyota", "title" : [ "t1", "t2", "t3", "t4" ] }

# 0 或大于0的数: 在尾部删除一个
db.t.update({name: "toyota"}, {$pop: {title: 2}})
# 结果
{"name" : "toyota", "title" : [ "t1", "t2", "t3" ] }

# 小于0的数： 在头部删除一个
db.t.update({name: "toyota"}, {$pop: {title: -1}})
# 结果
{"name" : "toyota", "title" : [ "t2", "t3" ] }
```

###### $pull从数组中删除满足条件的元素

```
# 准备数据
{"name" : "toyota", "title" : [ "t1", "t2", "t2", "t3" ] }
# 删除元素之为t2的元素
db.t.update({name: "toyota"}, {$pull: {title: "t2"}})
# 结果
{"name" : "toyota", "title" : [ "t1", "t3" ]     }
```

##### 数据跨级修改
比如想修改xx字段的第一个元素的yy字段的值<br>

```
# 准备数据
{"uid":"001",comments:[{"name":"t1","size":10},{"name":"t2","size":12}]}

# 将comment的第一个元素的size属性增一
db.t.update({"uid": "001"}, {$inc: {"comments.0.size": 1}})
# 结果
{"uid" : "001", "comments" : [ { "name" : "t1", "size" : 11 }, { "name" : "t2", "size" : 12 } ] }

# 将name为t1的comments元素的size更改为1
db.t.update({"comments.name": "t1"}, {$set: {"comments.$.size": 1}})
# 结果为
{ "uid" : "001", "comments" : [ { "name" : "t1", "size" : 1 }, { "name" : "t2", "size" : 12 } ] }
# 若为多个文档满足条件，则只更新第一个文档。
```

##### upsert

upsert是一种特殊的更新。当没有符合条件的文档，就以这个条件和更新文档为基础创建一个新的文档，如果找到匹配的文档就正常的更新。<br>
使用upsert，既可以避免竞态问题，也可以减少代码量（update的第三个参数就表示这个upsert，参数为true时）

```
# 假设表中现在没有数据
db.t.update({"size": 11}, {$inc: {"size": 3}})
# WriteResult({ "nMatched" : 0, "nUpserted" : 0, "nModified" : 0 })

# upsert 默认值是false
db.t.update({"size": 11}, {$inc: {"size": 3}}, false)
# 和上面一样依然没有任何影响

db.t.update({"size": 11}, {$inc: {"size": 3}}, true)
# 结果插入了一条数据
{ "_id" : ObjectId("5aacb8d4cfd7aea328745c5d"), "size" : 14 }
```

##### save()
1.可以在文档不存在的时候插入，存在的时候更新，只有一个参数文档。<br>
2.要是文档含有"_id"，会调用upsert。否则，会调用插入。

```
# 准备数据
{ "_id" : ObjectId("5aacb8d4cfd7aea328745c5d"), "size" : 14 }

# save
# 必须通过findOne调用 这样返回值文档类型
# 如果使用的find,则返回的是一个DBQuery object
var obj = db.t.findOne() 
obj.size = 10
db.t.save(obj) ## 更新

## obj不含ObjectId的情况
var obj2 = {"size": 10}
db.t.save(obj2) ## 新增一条
```