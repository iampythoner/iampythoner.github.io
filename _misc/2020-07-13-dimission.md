---
layout: cnblog_post
title: dimission
permalink: '/misc/dimission'
date: 2020-07-13 08:37:39
show: false
categories: misc
---

技术清单：

```
docker k8s
ci cd jenkins
tracer 全链路监控 kafka埋点 灰度：灰度上线(流量灰度)，功能灰度，设备灰度
profile
rpc
fastapi asyncio
postgreSQL upsert any returning gin索引 分词 插件
django
pytest tox apidoc mkserver
各种流程
流计算 Flink 维表 cKafka TKE 

binglog -> EFK loki  vector->sink   OLAP 普罗米修斯
vector-source syslog kafka(update->row 更新操作单独一条新数据)
clickhouse

airflow

grafana 监控面板
istio
```


以k8s搭建EFK环境
MySQL优化 分页 慢查询 索引相关
Redis 缓存穿透，缓存击穿，分布式锁
epoll select 的区别
单机redis和集群redis redis数据过期策略是什么
SQL事务里有网络请求操作，会发生什么


总融资超过10亿
目前D1轮 5000万美元
-----
-----
go 大驼峰代码规范
基本规范要注意，举个例子：Go变量都是驼峰式命名，看到过有人把它写成了unix命名，这样就会有一个从代码风格上就不便于探究这个语言，比如当遇到大写字幕开头的公有方法，小写字幕开头是私有方法的情况，这个时候因为一味地使用unix命名，而不再强调大小驼峰，以至于在写出大写字母开头接下划线这样的第三种风格代码，不仅形式上不伦不类，非常影响代码阅读，而且无法契合Go语言规范的整体性。


云风：一个编程的自由人（图灵访谈） https://www.ituring.com.cn/article/58692/


Python 为什么list不能作为字典的key？ https://www.kawabangga.com/posts/1821



git submodule add https://github.com/mikezone/mikezone.github.io.git  public
无法使用 https://stackoverflow.com/questions/20929336/git-submodule-add-a-git-directory-is-found-locally-issue
三次握手与四次挥手： https://baijiahao.baidu.com/s?id=1654225744653405133&wfr=spider&for=pc