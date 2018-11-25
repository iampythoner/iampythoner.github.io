---
layout: cnblog_post
title:  "SQL-statements"
permalink: '/misc/SQL-statements'
date:   2018-03-03 07:34:39
categories: misc
---

# 1

```
------------------ MySQL 服务
-- sudo service mysql start/stop/restart/status

------------------ 数据库相关的
-- 查看数据库
show databases;
-- 使用数据库
use 数据库名;
-- 查看当前使用的数据库
select database();
-- 创建数据库
create database 数据库名 charset=utf8;
-- 删除数据库
drop database 数据库名;

------------------ 备份和恢复
-- 备份 命令行中执行, 使用的是mysqldump
mysqldump -uroot -p 数据库名 > xxx.sql
-- 只备份指定的表
mysqldump -uroot -p 数据库名 --tables 表名1 表名2 > xxx.sql

-- 恢复
mysql -uroot -p 数据库名 < xxx.sql

-- 恢复，已经在mysql 交互环境中
source xxx.sql

------------------表操作
--- 查看当前库中所有的表
show tables;
--- 查看表结构
desc 表名;
--- 查看创建表的语句
show create table 表名;
--- 创建表
CREATE TABLE `students` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(20) DEFAULT NULL,
  `age` tinyint(3) unsigned DEFAULT '0',
  `height` decimal(5,2) DEFAULT NULL,
  `gender` enum('男','女','保密') DEFAULT NULL,
  `cls_id` int(10) unsigned DEFAULT '0',
  PRIMARY KEY (`id`)
)

--- 删除表
drop table students;

--- 修改表
-- 添加字段
-- alter table 表名 add 列名 类型;
alter table students add birthday datetime;
-- 修改字段，但是不重命名
alter table students modify birthday int;
-- 修改字段，并且重命名
alter table students change birthday birth bigint not null;
-- 删除字段
alter table students drop birthday;

------------------表的数据操作,基本的增删改查
-- 查询(retrieve)
select * from students;
select name,age from students;
-- 插入
insert into students values(id, name, age, height, gender, cls_id); -- 得填上id
insert into students(name, age) values('Mike', 10);
insert into students(name, age) values('Mike', 10), ('John', 11);
-- 修改
update students set name='Mike' where id=1; -- 没有TM的into from什么的
-- 删除
delete from students where id=1;
-- 逻辑删除
update students set is_delete=1 where id=1;



-------------------
-- DQL：数据查询语言，用于对数据进行查询，如select
-- DML：数据操作语言，对数据进行增加、修改、删除，如insert、udpate、delete
-- TPL：事务处理语言，对事务进行处理，包括begin transaction、commit、rollback
-- DCL：数据控制语言，进行授权与权限回收，如grant、revoke
-- DDL：数据定义语言，进行数据库、表的管理等，如create、drop
-- CCL：指针控制语言，通过控制指针完成表的操作，如declare cursor
-- 对于web程序员来讲，重点是数据的crud（增删改查），必须熟练编写DQL、DML，能够编写DDL完成数据库、表的操作，其它语言如TPL、DCL、CCL了解即可
-- SQL 是一门特殊的语言,专门用来操作关系数据库
-- 不区分大小写


-- TINYINT	  1	-128 ~ 127	        0 ~ 255
-- SMALLINT	2	-32768 ~ 32767	    0 ~ 65535
-- MEDIUMINT	3	-8388608 ~ 8388607	0 ~ 16777215
-- INT/INTEGER	4	-2147483648 ~2147483647	0 ~ 4294967295
-- BIGINT	8	-9223372036854775808 ~ 9223372036854775807	0 ~ 18446744073709551615
--
-- CHAR	0-255	类型:char(3) 输入 'ab', 实际存储为'ab ', 输入'abcd' 实际存储为 'abc'
-- VARCHAR	0-255	类型:varchar(3) 输 'ab',实际存储为'ab', 输入'abcd',实际存储为'abc'
-- TEXT	0-65535	大文本
--
-- DATE	4	'2020-01-01'
-- TIME	3	'12:29:59'
-- DATETIME	8	'2020-01-01 12:29:59'
-- YEAR	1	'2017'
-- TIMESTAMP	4	'1970-01-01 00:00:01' UTC ~ '2038-01-01 00:00:01' UTC
```

# 2 ad

```
------------------------------------ 高级查询
-- as 起别名
select name as 名字 from studnets;
-- 消除重复的行 -- 查看有哪几种xxx
select distinct gender from students; -- 查看有哪几种性别
select distinct name,gender from students; -- 查看有哪几种(性别，姓名)组合

------ where 运算符
---- 比较
-- 等于: =
-- 大于: >
-- 大于等于: >=
-- 小于: <
-- 小于等于: <=
-- 不等于: != 或 <>
select * from students where id <= 4;
---- 逻辑
-- and or not
select * from students where id < 4 or is_delete=0;
-------- 模糊查询
-- like
-- %表示任意多个任意字符
-- _表示一个任意字符
select * from students where name like 'M%';
select * from students where name like 'Mik_';
-------- 范围查询
-- in表示在一个非连续的范围内
select * from students where id in (1, 3);
-- between ... and ...表示在一个连续的范围内
select * from students wehre between 1 and 3; -- 1，2，3
--------- 空判断
select * from students where cls_id is not null;


--------------------------------------------------------排序
select * from students order by name asc默认/desc;
select * from students [where age in (10, 11)] order by name; -- 先拿到数据集再排序

--------------------------------------------------------聚合函数
select count(*) from students;
-- select max min avg
select sum(age) from students;

-------------------------------------------------------- 分组
select gender from students group by gender;
-- +--------+
-- | gender |
-- +--------+
-- | 男     |
-- | 女     |
-- | 中性   |
-- | 保密   |
-- +--------+
-------------------- group by + group_concat()
select gender, group_concat(name) from students group by gender;
-- +--------+-----------------------------------------------------------+
-- | gender | group_concat(name)                                        |
-- +--------+-----------------------------------------------------------+
-- | 男     | 彭于晏,刘德华,周杰伦,程坤,郭靖                            |
-- | 女     | 小明,小月月,黄蓉,王祖贤,刘亦菲,静香,周杰                  |
-- | 中性   | 金星                                                      |
-- | 保密   | 凤姐                                                      |
-- +--------+-----------------------------------------------------------+


--------------------  group by + 集合函数
-- 分别统计性别为男/女的人年龄平均值
select gender,avg(age) from students group by gender;


-------------------- group by + having 过滤
-- 平均年龄大于10的性别
select gender, avg(age) from students group by gender having avg(age) > 10;
-- 人数大于1的性别和人数
select gender, count(*) from students group by gender having count(*) > 1;

-------------------- group by + with rollup 汇总
---- with rollup的作用是：在最后新增一行，来记录当前列里所有记录的总和
select gender,count(*) from students group by gender with rollup;

----------------------------------------分页
select * from students limit 3; -- 取前3条，相当于0,3
---- select * from 表名 limit start,count
select * from students limit 2, 4; -- 跳过2条 从第3条开始取4条

---------------------------------------- 链接查询
select * from classes as c [inner] join students as s on c.id = s.cls_id;
-- +----+-----------+----+------+------+--------+--------+--------+
-- | id | name      | id | name | age  | height | gender | cls_id |
-- +----+-----------+----+------+------+--------+--------+--------+
-- |  1 | 精英班    |  1 | Mike |   10 | 170.00 | 男     |      1 |
-- |  1 | 精英班    |  2 | John |   11 | 180.00 | 男     |      1 |
-- +----+-----------+----+------+------+--------+--------+--------+
select * from classes as c left join students as s on c.id = s.cls_id;
-- +----+-----------+------+------+------+--------+--------+--------+
-- | id | name      | id   | name | age  | height | gender | cls_id |
-- +----+-----------+------+------+------+--------+--------+--------+
-- |  1 | 精英班    |    1 | Mike |   10 | 170.00 | 男     |      1 |
-- |  1 | 精英班    |    2 | John |   11 | 180.00 | 男     |      1 |
-- +----+-----------+------+------+------+--------+--------+--------+
select * from classes as c right join students as s on c.id=s.cls_id;
-- +------+-----------+----+------+------+--------+--------+--------+
-- | id   | name      | id | name | age  | height | gender | cls_id |
-- +------+-----------+----+------+------+--------+--------+--------+
-- |    1 | 精英班    |  1 | Mike |   10 | 170.00 | 男     |      1 |
-- |    1 | 精英班    |  2 | John |   11 | 180.00 | 男     |      1 |
-- +------+-----------+----+------+------+--------+--------+--------+

---------------------------------------自关联
-- 通常大分类有小分类这种形式的数据放到一个表中，并且pid指向表的id
CREATE TABLE `areainfo` (
  `id` int(10) unsigned NOT NULL,
  `name` varchar(32) DEFAULT NULL,
  `pid` int(10) unsigned DEFAULT NULL,
  PRIMARY KEY (`id`)
)
--- 查询广东省下的所有地级市
select c.name from areainfo as p inner join areainfo as c
on c.pid=p.id where p.name='广东省';

--- 查询山东省下的所有地级市和县区
-- 注意： [a join b on 条件] 结果是一个`表`********************

select city.name, county.name from areainfo as county join
(areainfo as city join areainfo as province
on city.pid=province.id and province.pid is null)
on city.id=county.pid
where province.name='山东省';


---------------------------------- 子查询
--- 每个SQL包含两部分 主查询 和 子查询
-- 子查询有三种类型
-- 标量子查询: 子查询返回的结果是一个数据(一行一列)
-- 列子查询: 返回的结果是一列(一列多行)
-- 行子查询: 返回的结果是一行(一行多列)


-- 标量子查询： 将子查询的结果当成一个值
-- 查询大于平均年龄的学生
select * from students where age > (select avg(age) from students);

-- 列子查询： 将子查询的结果当成同一属性(列)多个值的的集合
-- 查询班级还存在的学生的名字
select name from students where cls_id in (select id from classes);

-- 行子查询： 将多列数据看成一条数据
-- 查找班级年龄最大,身高最高的学生
select * from students where (height, age)=
(select max(height), max(age) from students);
-- 只有在最大身高、最大年龄刚好是一个人的时候才能查找到数据。
```

# 3NF

```
--------------------  三范式
-- 第一范式（1NF）：强调的是列的原子性，即列不能够再分成其他几列。

-- 第二范式（2NF）：首先是 1NF，另外包含两部分内容，一是表必须有一个主键；
-- 二是没有包含在主键中的列必须完全依赖于主键，而不能只依赖于主键的一部分,即字段必须完全依赖一个主关键字。

-- 第三范式（3NF）：首先是 2NF，另外非主键列必须直接依赖于主键，不能存在传递依赖。
-- 即不能存在：非主键列 A 依赖于非主键列 B，非主键列 B 依赖于主键的情况

-- 1NF: 列原子性
-- 2NF：同一表，如果主键是2个列，那么其他列的字段必须都依赖于这两个列，
-- 如果有的列只依赖于其中一个主键的列，那么就要拆分表，
-- 3NF：非主键一定要依赖主键，并且要直接依赖；不能传递依赖，如果有，也要拆分表

create table goods (
  id int unsigned primary key auto_increment,
  name varchar(150) not null,
  cate_name varchar(40) not null,
  brand_name varchar(40) not null,
  price decimal(10, 3) not null default 0,
  is_show bit not null default 1,
  is_saleoff bit not null default 0
);

----------------- SQL 强化
-- 求所有电脑产品的平均价格,并且保留两位小数
select round(avg(price),2) as avg_price from goods;
-- 查询所有价格大于平均价格的商品，并且按价格降序排序
select * from goods where price >
(select avg(price) as avg_price from goods)
order by price desc;
-- 查询类型cate_name为 '超极本' 的商品名称、价格
select name, price from goods where cate_name='超级本';
-- 显示商品的种类
select distinct cate_name from goods; -- 会按出现的顺序显示
select cate_name from goods group by cate_name; -- 会按照cate_name排序
-- 显示每种商品的平均价格
select cate_name, avg(price) from goods group by cate_name;
-- 查询每种类型的商品中 最贵、最便宜、平均价、数量
select cate_name, max(price), min(price), avg(price), count(*) from goods group by cate_name;

-- 查询每种类型中最贵的电脑信息*********************
select * from goods where (cate_name, price) in
(select cate_name, max(price) from goods group by cate_name);

select * from goods inner join
(select cate_name,max(price) as max_price from goods group by cate_name) as g
on goods.cate_name=g.cate_name and goods.price=g.max_price;

----------------------------------------------------拆表
-------------------------- "商品分类"
---- cate_name 是非主键，但是不依赖于主键goods.id，不符合3NF
-------------- 1.创建 "商品分类"" 表
create table if not exists goods_cates(
    id int unsigned primary key auto_increment,
    name varchar(40) not null
);
-- 查询goods表中商品的种类
select distinct cate_name from goods;
select cate_name from goods group by cate_name;
-- 将分组结果写入到goods_cates数据表*********************
-- 错误 ：insert into goods_cates(name) values(select cate_name from goods group by cate_name);
insert into goods_cates(name) select cate_name from goods group by cate_name;
-------------- 2.同步表数据 *******************
update goods as g inner join goods_cates as c on g.cate_name=c.name set g.cate_name=c.id;
-------------- 3.修改表goods表结构 *****************
alter table goods change cate_name cate_id int unsigned not null;
alter table goods add [constraint `约束名`] foreign key (cate_id) references goods_cates(id);

-------------------------- "商品品牌"
---- brand_name 是非主键，但是不依赖于主键goods.id，不符合3NF
-------------- 1.创建 "商品品牌"" 表，并直接从goods表中导入数据*********
create table if not exists goods_brands (
  id int unsigned primary key auto_increment,
  name varchar(50) not null
) select brand_name as name from goods group by brand_name;
-------------- 2.同步数据
update goods as g inner join goods_brands as b on g.brand_name=b.name set g.brand_name=b.id;
-------------- 3.修改表goods的表结构
alter table goods change brand_name brand_id int unsigned not null;
alter table goods add foreign key (brand_id) references goods_brands(id);


-------------------------------------
外键和主键的区别:
外键受唯一性约束，也就是说外键不一定就是其他表的主键，可能是其他表中一个普通的列，但是这个列必须是unique的。
但通常情况下外键就是其他表中的主键。
主键本来就是unique的，而且是not null的，它受唯一性约束和非空约束。


-------------------------------------删除外键约束
alter table goods drop foreign key 约束名称;
```

# 4 other

```
在 MySQL 中，有三种主要的类型：文本、数字和日期/时间类型。

Text 类型：
CHAR(size)
VARCHAR(size)
TINYTEXT
TEXT	存放最大长度为 65,535 个字符的字符串。
BLOB	用于 BLOBs (Binary Large OBjects)。存放最多 65,535 字节的数据。
MEDIUMTEXT	存放最大长度为 16,777,215 个字符的字符串。
MEDIUMBLOB	用于 BLOBs (Binary Large OBjects)。存放最多 16,777,215 字节的数据。
LONGTEXT	存放最大长度为 4,294,967,295 个字符的字符串。
LONGBLOB	用于 BLOBs (Binary Large OBjects)。存放最多 4,294,967,295 字节的数据。
ENUM(x,y,z,etc.)
SET	与 ENUM 类似，SET 最多只能包含 64 个列表项，不过 SET 可存储一个以上的值。

Number 类型：
TINYINT(size)	-128 到 127 常规。0 到 255 无符号*。在括号中规定最大位数。
SMALLINT(size)	-32768 到 32767 常规。0 到 65535 无符号*。在括号中规定最大位数。
MEDIUMINT(size)	-8388608 到 8388607 普通。0 to 16777215 无符号*。在括号中规定最大位数。
INT(size)	-2147483648 到 2147483647 常规。0 到 4294967295 无符号*。在括号中规定最大位数。
BIGINT(size)	-9223372036854775808 到 9223372036854775807 常规。0 到 18446744073709551615 无符号*。在括号中规定最大位数。
FLOAT(size,d)	4bytes 带有浮动小数点的小数字。在括号中规定最大位数。在 d 参数中规定小数点右侧的最大位数。
DOUBLE(size,d) 8bytes	带有浮动小数点的大数字。在括号中规定最大位数。在 d 参数中规定小数点右侧的最大位数。
DECIMAL(size,D) size字节(D+2 , 如果size<D) 	作为字符串存储的 DOUBLE 类型，允许固定的小数点。
REAL 8 个字节
NUMERIC(M,D) M字节(D+2 , 如果M <D)

Date 类型：

DATE() 3bytes 日期。格式：YYYY-MM-DD 注释：支持的范围是从 '1000-01-01' 到 '9999-12-31'
DATETIME() 8bytes 日期和时间的组合。格式：YYYY-MM-DD HH:MM:SS ，范围'1000-01-01 00:00:00' 到 '9999-12-31 23:59:59'
TIMESTAMP() 4bytes 时间戳。TIMESTAMP 值使用 Unix 纪元('1970-01-01 00:00:00' UTC) 至今的描述来存储。
格式：YYYY-MM-DD HH:MM:SS 支持的范围是从 '1970-01-01 00:00:01' UTC 到 '2038-01-09 03:14:07' UTC
TIME() 3bytes	时间。格式：HH:MM:SS 注释：支持的范围是从 '-838:59:59' 到 '838:59:59'
YEAR() 1bytes 2 位或 4 位格式的年。注释：4 位格式所允许的值：1901 到 2155。2 位格式所允许的值：70 到 69，表示从 1970 到 2069。



-----------------------------------------------python 链接数据库
# mysqlconnector
from mysql.connector import connect

config = {
    'host': '192.168.199.139',
    'port': 3306,
    'user': 'root',
    'password': '123456',
    'database': 'jd',
    'charset': 'utf8'
}

conn = connect(**config)
cs = conn.cursor()
rows_count = cs.execute('select * from goods;')
print(rows_count)
result = cs.fetchall()
print(result) # [(1, 'r510vc 15.6英寸笔记本', 5, 2, Decimal('3399.000'), 1, 0),...]
cs.close()
conn.close()

----------------------------------------------- 视图、索引
视图是从一个或几个基本表(或视图)导出的表。它与基本表不同，是一个虚表。数据库中只存放视图的定义，
而不存放视图对应的数据，这些数据仍存放在原来的基本表中。所以基本表中的数据发生的变化，
从视图中查询出的数据也就随之改变了。从这个意义上讲，视图就像一个窗口，透过它可以看到数据库中自己感兴趣的数据及其变化。
视图一经定义，就可以和基本表一样被查看、被删除。也可以在一个视图之上再定义新的视图，但对视图的更新(增、删、改)操作则有一定的限制。

定义视图
create view 视图名(列名，) as 子查询 [with check option]
create view v_student as select name, age from student where gender='男';
这时通过 show tables; 可以看到多了v_student 但实际上它不是一张表。

查询
select * from v_student;
+------+------+
| name | age  |
+------+------+
| Mike |   10 |
| John |   11 |
+------+------+

更新视图
update v_student set age=12 where name='Mike';
insert into v_student values('Jack', 13); -- 影响了student表，但是视图查不到这个数据，因为没有指定gender='男'
delete from v_student where name='John';

视图的作用：
1.视图能够简化用户的操作
2.视图使用户能以多种角度看待同一数据，增加了数据查看和操作的灵活性
3.视图对重构数据库提供了一定程度的逻辑独立性
4.视图能够对机密数据提供安全保护，可以将机密数据字段不放到视图上，而只对权限低的管理员提供操作视图的权限。
5.适当的利用视图可以更清晰地表达查询



删除视图
drop view v_student;

----------------索引
索引用于快速找出在某个列中有一特定值的行，不使用索引，MySQL必须从第一条记录开始读完整个表，
直到找出相关的行，表越大，查询数据所花费的时间就越多，如果表中查询的列有一个索引，
MySQL能够快速到达一个位置去搜索数据文件，而不必查看所有数据，那么将会节省很大一部分时间。

例如：有一张person表，其中有2W条记录，记录着2W个人的信息。
有一个Phone的字段记录每个人的电话号码，现在想要查询出电话号码为xxxx的人的信息。
如果没有索引，那么将从表中第一条记录一条条往下遍历，直到找到该条信息为止。
如果有了索引，那么会将该Phone字段，通过一定的方法进行存储，好让查询该字段上的信息时，
能够快速找到对应的数据，而不必在遍历2W条数据了。
其中MySQL中的索引的存储类型有两种：BTREE、HASH。

但过多的使用索引将会造成滥用。因此索引也会有它的缺点：
1.虽然索引大大提高了查询速度，同时却会降低更新表的速度，
如对表进行INSERT、UPDATE和DELETE。因为更新表时，MySQL不仅要保存数据，还要保存一下索引文件。
2.建立索引会占用磁盘空间的索引文件。

索引分：单列索引(普通索引，唯一索引，主键索引)、组合索引、全文索引、空间索引。
①单列索引，即一个索引只包含单个列，一个表可以有多个单列索引，但这不是组合索引。
②组合索引，即一个索引包含多个列。
③全文索引，只有在MyISAM引擎上才能使用，只能在CHAR,VARCHAR,TEXT类型字段上使用全文索引，
 介绍了要求，说说什么是全文索引，就是在一堆文字中，通过其中的某个关键字等，
 就能找到该字段所属的记录行，比如有"你是个大煞笔，二货 ..." 通过大煞笔，可能就可以找到该条记录。
 这里说的是可能，因为全文索引的使用涉及了很多细节。
④空间索引，空间索引是对空间数据类型的字段建立的索引，MySQL中的空间数据类型有四种，
  GEOMETRY、POINT、LINESTRING、POLYGON。
  在创建空间索引时，使用SPATIAL关键字。
  要求，引擎为MyISAM，创建空间索引的列，必须将其声明为NOT NULL。


创建索引
--------- 语法
CREATE TABLE table_name[col_name data type]
[unique|fulltext][index|key][index_name](col_name[length])[asc|desc]
1.unique|fulltext为可选参数，分别表示唯一索引、全文索引
2.index和key为同义词，两者作用相同，用来指定创建索引
3.col_name为需要创建索引的字段列，该列必须从数据表中该定义的多个列中选择
4.index_name指定索引的名称，为可选参数，如果不指定，默认col_name为索引值
5.length为可选参数，表示索引的长度，只有字符串类型的字段才能指定索引长度
6.asc或desc指定升序或降序的索引值存储

-- create index 索引名 on 表名(字段)
create index i_name on students(name);

还可以在创建表的时候直接指定：
CREATE TABLE students(
  id INT NOT NULL,
  name VARCHAR(16) NOT NULL,
  index [indexName] (name)
);

--- 唯一索引，要求索引列的值必须唯一
create unique index indexName ON students(id)

-- 删除索引
drop index indexName on tablename;

--- 查看表的索引
show index from students \G

--------------------- 一些例子 表明just,现在有这些字段


CREATE TABLE `just` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `msg_content` text NOT NULL,
  `msg_desc` varchar(255) NOT NULL,
  `receiver` int(11) NOT NULL,
  `using_id` varchar(11) NOT NULL,
  PRIMARY KEY (`id`),
) ENGINE=MyISAM AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4



----------------- 给receiver添加普通索引
alter table just add index index_receiver(receiver);
或
alter table just add key index_receiver(receiver);
或
create index index_receiver on just(receiver);

----------------- 给using_id添加唯一索引
alter table just add unique [index] index_using_id(using_id);

------------------ 给msg_content 添加全文索引
alter table just add fulltext index_msg_content(msg_content);
----使用
select msg_content from just where match(msg_content) against('tomorrow');

----------------- 添加一列geometry类型，为这个列添加空间索引
alter table just  add loc geometry not null;  # 必须是not null的 否则创建索引不成功
----补充数据
update just set loc=geomfromtext('point(108.9498710632 34.2588125935)') where id=2;
update just set loc=geomfromtext('point(108.9465236664 34.2598766768)') where id=1;
----- 添加空间索引
alter table just add SPATIAL INDEX index_loc(loc);


------------------ 查看数据表
CREATE TABLE `just` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `msg_content` text NOT NULL,
  `msg_desc` varchar(255) DEFAULT '',
  `receiver` int(11) NOT NULL,
  `using_id` varchar(11) NOT NULL,
  `loc` geometry NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `index_using_id` (`using_id`),
  KEY `index_receiver` (`receiver`),
  SPATIAL KEY `index_loc` (`loc`),
  FULLTEXT KEY `index_msg_content` (`msg_content`)
) ENGINE=MyISAM AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4


```
