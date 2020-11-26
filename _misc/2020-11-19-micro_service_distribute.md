---
layout: cnblog_post
title:  "micro_service_distribute"
permalink: '/misc/micro_service_distribute'
date:   2020-11-19 07:34:39
show: false
categories: misc
---


有赞后端：https://tech.youzan.com/tag/back-end/
setnx锁 https://zhuanlan.zhihu.com/p/111354065?from_voters_page=true



服务通信：
同步： HTTP gRPC
异步通信模式：作业队列， 发布-订阅

服务定位：consul 服务发现注册中心

服务边界：
1.API网关：Kong
2.服务于前端的后端（BFF）
3.消费者驱动型网关： GraphQL


分布式应用的事务一致性
二阶段提交  2PC
CAP 理论：一致性 可用性 分区容错性

基于事件的通信：P97 编排
1.服务恢复处理积压
2.回滚时发送事件，订阅的服务有相应的处理

Saga

