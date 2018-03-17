---
layout: cnblog_post
title:  "mongodb_op"
permalink: '/misc/mongodb_op'
date:   2018-03-16 07:34:39
categories: misc
---

#### 创建库、集合 删除库、集合
```
### 隐式创建库和集合
use dbname
dbname.collection_name.insert()
# 这两句执行完成之后会创建库dbname 创建集合collection_name

#### 删除库
db.dropDatabase() # 如 mike.dropDatabase()

###### 创建集合
# 创建带options的集合
db.createCollection(name, options)

options 可以是以下选项：
capped	布尔	（可选）如果为 true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。当该值为 true 时，必须指定 size 参数。
autoIndexId	布尔	（可选）如为 true，自动在 _id 字段创建索引。默认为 false。
size	数值	（可选）为固定集合指定一个最大值（以字节计）。如果 capped 为 true，也需要指定该字段。
max	数值	（可选）指定固定集合中包含文档的最大数量。

db.createCollection('mycol', {capped: true, autoIndexId: true, size:6142800, max: 10000})
###### 删除集合
db.collection.drop() # 成功返回true 否则返回false
```

#### insert

```
########## insert
#### insert只能插入一条，尽管传入了多个
db.test.insert({name: "mike"}, {name: "john"})
# 结果
WriteResult({ "nInserted" : 1 })
#### insertOne 只能插入一条， 尽管传入了多个
db.test.insertOne({name: "mike"}, {name: "john"})
# 返回结果
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5aab6e45b7df19c6910ab198")
}
### insertMany 能插入多条
db.test.insertMany([{name: 'alice'}, {name: 'jack'}])
# 返回结果
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("5aab6e91b7df19c6910ab199"),
		ObjectId("5aab6e91b7df19c6910ab19a")
	]
}
```

#### update save

```
###### 更新 update 和 save
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
)

query : update的查询条件，类似sql update查询内where后面的。
update : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
upsert : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
multi : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
writeConcern :可选，抛出异常的级别

db.col.insert({
    title: 'MongoDB 教程',
    description: 'MongoDB 是一个 Nosql 数据库',
    by: '菜鸟教程',
    url: 'http://www.runoob.com',
    tags: ['mongodb', 'database', 'NoSQL'],
    likes: 100
})
## 将title 更改为MongoDB
db.col.update({title: 'MongoDB 教程'}, {$set: {title: 'MongoDB'}})
# 结果 WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

## 默认只会修改第一个发现的文档，如果要修改所有符合query条件的数据，就要使用multi参数
db.col.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}},{multi:true})
# 结果：

######## save() 方法
#### 替换已经有的文档，通常要修改_id字段，该字段的数据存在则会直接替换
db.collection.save(
   <document>,
   {
     writeConcern: <document>
   }
)

db.col.save({"_id": ObjectId("5aab6fffb7df19c6910ab19b"), "title": "MongoDB", description: "MongoDB 是一个NoSQL 数据库", by: "Runoob", url: 'http://www.runoob.com', tags: ["mongodb", "NoSQL"], likes: 110})
#结果： WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

## 查找 如果找不到则插入 true，是否对全部查找的结果操作 false
db.col.update( { "count" : { $gt : 4 } } , { $set : { "test5" : "OK"} },true,false );
# 结果插入了一条数据
{ "_id" : ObjectId("5aab73b9011fd37acf5d4e2a"), "test5" : "OK" }

```


#### remove

```
################  删除文档
db.collection.remove(
   <query>,
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)

query :（可选）删除的文档的条件。
justOne : （可选）如果设为 true 或 1，则只删除一个文档。
writeConcern :（可选）抛出异常的级别。

db.col.insert()

## 只删除第一个符合条件的
db.col.remove({'title': 'MongoDB 教程'})

### 删除所有符合条件的 justOne传入false
db.col.remove({title: {$regex: "MongoDB"}}, false)

### 删除集合中所有的文档
db.col.remove({})
```

#### find

```
############### find 格式
db.collection.find(query, projection)

query ：可选，使用查询操作符指定查询条件
projection ：可选，使用投影操作符指定返回的键。查询时返回文档中所有键值， 只需省略该参数即可（默认省略）。

### pretty 美化输出
db.collection.find().pretty()

### 其他输出方法
# 只查询符合条件的第一个文档
findOne()

#### where 条件
等于 {<key>:<value>}
小于	{<key>:{$lt:<value>}}
小于或等于 $lte
大于 $gt
大于或等于 $gte
不等于 $ne
and ,分隔条件即可
or    $or: [{key1: value1}, {key2:value2}]
in $in  db.stu.find({age:{$in:[18,28]}})
where $where + 函数


db.test.insert({name: "Mike", age: 12})
db.test.insert({name: "john", age: 15})
# 查找姓名包含o的 或者年龄小于13的
db.test.find({$or: [{name : {$regex: /o/}}, {age: {$lt : 13}}]})
# 结果是：
{ "_id" : ObjectId("5aab783cb7df19c6910ab19e"), "name" : "Mike", "age" : 12 }
{ "_id" : ObjectId("5aab784cb7df19c6910ab19f"), "name" : "john", "age" : 15 }

#### where 查询
使⽤$where后⾯写⼀个函数， 返回满⾜条件的数据
查询年龄⼤于30的学⽣
db.stu.find({
    $where:function() {
        return this.age>30;
    }
})


######## projection的使用 （find投影）
## 指定哪个参数可以显示 inclusion
db.collection.find(query, {title: 1, by: 1})
## 指定哪个参数可以不显示 exclusion模式
db.collection.find(query, {title: 0, by: 0})
两种情况不可以混用，要么全部是0，要么全部是1，
但是_id是个例外，可以在inclusion模式下设置为0,但是不可以在exlusion模式下设置为1


#########  $type 条件过滤
客用户筛选指定类型的字段
Double	    1
String	    2
Object	    3
Array	    4
Binary data	5
Undefined	6	已废弃。
Object id	7
Boolean	    8
Date	    9
Null	    10
Regular Expression	11
JavaScript	13
Symbol	    14
JavaScript (with scope)	15
32-bit integer	16
Timestamp	17
64-bit integer	18
Min key	    255	Query with -1.
Max key     127

## 例如 查询name类型为String的文档
db.test.find({"name": {$type: 2}})
## 分页 limit + skip
# 假设page从0开始
db.test.find().limit(pageSize).skip(page*pageSize)
# page 从1开始
db.test.find().limit(pageSize).skip((page-1)*pageSize)

###### 排序 sort 1: asc  -1: desc
db.test.find().sort({name:1})

##### count
db.stu.find({gender:true}).count()
# count 直接加条件
db.stu.count({age:{$gt:20},gender:true})

#### distinct 消除重复

# db.集合名称.distinct('去重字段',{条件})
db.stu.distinct('hometown',{age:{$gt:18}})

```

#### 聚合 aggregate

```
# MongoDB 中的聚合经常配合管道使用

# 基本格式
db.COLLECTION_NAME.aggregate(AGGREGATE_OPERATION)

# 查询_id
db.test.aggregate([{$group: {_id: '$name', total: {$sum: 1}}}])

select name, count(*) from test group by name;

常用的聚合表达式：
$sum、$avg、$min、$max

$push 在结果文档中插入值到一个数组中。
# 如： 按照性别统计所有人的姓名
db.test.aggregate([{$group: {_id: '$gender', names: {$push: "$name"}}}])
# 得到的结果是 (两条数据)
{ "_id" : "男", "names" : [ "Mike" ] }
{ "_id" : "女", "names" : [ "alice" ] }

$addToSet 在结果文档中插入值到一个数组中，但不创建副本。 和push的结果一致

$first 根据资源文档的排序获取第一个文档数据
# 如 将所有人按年龄分组后，查看每个年龄的第一个人的名字
db.test.aggregate([{$group: {_id: '$age', first: {$first: "$name"}}}])
{ "_id" : 15, "first" : "john" }
{ "_id" : 12, "first" : "Mike" }

$last 根据资源文档的排序获取最后一个文档数据, 和first功能类似
```

#### 管道
<img src="http://7vim0m.com1.z0.glb.clouddn.com/mongo/pipe.png" width="400" alt="管道"/>

```
# MongoDB 管道可以配合aggregate方法使用，可以实现过条件分组过滤数据的功能
常用的管道操作有：
$project：修改输入文档的结构。可以用来重命名、增加或删除域，也可以用于创建计算结果以及嵌套文档。
$match：用于过滤数据，只输出符合条件的文档。$match使用MongoDB的标准查询操作。
$limit：用来限制MongoDB聚合管道返回的文档数。
$skip：在聚合管道中跳过指定数量的文档，并返回余下的文档。
$unwind：将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值。
$group：将集合中的文档分组，可用于统计结果。
$sort：将输入文档排序后输出。
$geoNear：输出接近某一地理位置的有序文档。


##### project 管道，；决定输出那个字段

### 只显示年龄和性别，这个返回的是符合的数据
db.test.aggregate({$project: {_id: 0, name: 1, age: 1}})
## 结果是：
{ "name" : "Mike", "age" : 12 }
{ "name" : "john", "age" : 15 }

##### match 管道； 返回的是满足指定条件的文档
# 获取年龄大于12的人
db.test.aggregate({$match: { age: {$gt: 12}}})
# 获取年龄大于12的人，并且只显示名字
# 这里使用了管道，其实是将两个管道过滤放到一个[]中即可
db.test.aggregate([{$match: { age: {$gt: 12}}}, {$project: {_id: 0, name:1}}])
# 结果是
{ "name" : "john" }
{ "name" : "zhangyu" }

##### $skip 管道;
# 取年龄大于12的人的名字，跳过1条，显示一条
db.test.aggregate([{$match: { age: {$gt: 12}}}, {$project: {_id: 0, name:1}}, {$skip: 1}, {$limit: 2}])

#### $unwind
db.t2.insert({_id:1,item:'t-shirt',size:['S','M','L']})
db.t2.aggregate({$unwind:'$size'})

# 结果如下：
{ "_id" : 1, "item" : "t-shirt", "size" : "S" }
{ "_id" : 1, "item" : "t-shirt", "size" : "M" }
{ "_id" : 1, "item" : "t-shirt", "size" : "L" }

# preserveNullAndEmptyArrays
# 值为false表示丢弃属性值为空的⽂档
# 值为true表示保留属性值为空的⽂档
db.inventory.aggregate({
    $unwind: {
        path: '$字段名称',
        preserveNullAndEmptyArrays: <boolean> # 防止数据丢失
    }
})

##### $$ROOT：符合条件的文档内容
db.stu.aggregate({
    $gruop: {
        _id: '$gender',
        name: {$push: '$$ROOT'}
    }
})
```

#### 索引

```
### ensureIndex
创建的时候指定字段和索引顺序
db.col.ensureIndex({"title":1})

多个字段创建索引，联合索引
db.col.ensureIndex({"title": 1, "description": -1})

其他可选参数:
http://www.runoob.com/mongodb/mongodb-indexing.html
```

例子：

```
索引：以提升查询速度

测试：插入10万条数据到数据库中
for(i=0;i<100000;i++){db.t12.insert({name:'test'+i,age:i})}

db.t1.find({name:'test10000'})
db.t1.find({name:'test10000'}).explain('executionStats')

建立索引之后对比：
语法：db.集合.ensureIndex({属性:1})，1表示升序， -1表示降序
具体操作：db.t1.ensureIndex({name:1})
db.t1.find({name:'test10000'}).explain('executionStats')
```

<img src="http://7vim0m.com1.z0.glb.clouddn.com/mongo/index1.png" width="400" alt="no index"/>

<img src="http://7vim0m.com1.z0.glb.clouddn.com/mongo/index2.png?fdf=df" width="400" alt="index"/>


```
在默认情况下创建的索引均不是唯一索引。
创建唯一索引:
     db.t1.ensureIndex({"name":1},{"unique":true})
创建唯一索引并消除重复：
     db.t1.ensureIndex({"name":1},{"unique":true,"dropDups":true})  
建立联合索引(什么时候需要联合索引)：
     db.t1.ensureIndex({name:1,age:1})
查看当前集合的所有索引：
     db.t1.getIndexes()
删除索引：
     db.t1.dropIndex('索引名称')
```


#### 备份与恢复

```

################################## 备份
-h： MongDB所在服务器地址，例如：127.0.0.1，当然也可以指定端口号：127.0.0.1:27017
-d： 需要备份的数据库实例，例如：test
-o：备份的数据存放位置，例如：c:\data\dump，当然该目录需要提前建立，在备份完成后，系统自动在dump目录下建立一个test目录，这个目录里面存放该数据库实例的备份数据。
mongodump -h dbhost -d dbname -o dbdirectory

################################## 恢复
恢复语法：
     mongorestore -h dbhost -d dbname --dir dbdirectory
-h： 服务器地址
-d： 需要恢复的数据库实例
--dir： 备份数据所在位置

mongorestore -h 192.168.196.128:27017 -d test2 --dir ~/Desktop/test1bak/test1
```


#### 登录验证、权限设置

```
####### 外网访问
vim /etc/mongod.conf
    bindip: 0.0.0.0
sudo mongod --config /etc/mongod.conf

################################## 登录与验证
连接时指定库和用户名
mongo user:pwd@host:port/db
mongo host:port/db -u user -p
mongo db -u -p

链接之后验证
mongo db
db.auth(user, pwd)

################################## 账户权限设置
创建账户：
谨记：先在不开启认证的情况下，创建用户，之后关闭服务，然后再开启认证，才生效
db.createUser({
	user:'root',
	pwd:'root',
	customData:{description:"管理员root"},
	roles:[{
		'role':'root',
		'db':'admin'
	}]
})
db.createUser({
	user:'user2',
	pwd:'user2',
	customData:{description:"数据库账户描述"},
	roles:[{
		'role':'readWrite',
		'db':'demo2'
	}]
})

#### 登录认证：
db.auth("root","123456")

#### 查询已添加的用户：
db.system.users.find()

###### 如果在mongo.conf 配置文件中设置了auth为true 才需要验证 默认是以admin登录的
```