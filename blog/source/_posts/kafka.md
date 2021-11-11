---
title: kafka
date: 2021-11-11 22:20:33
tags: [big-data, kafka]
categories:
	- 答辩
	- 大数据
---

为什么选择kafka
和RabbitMQ，RocketMQ，FMQ等等相比

### （1） kafka吞吐高
比RabbitMQ高出1-2个数量级
- kafka QPS：百万级
- RabbitMQ QPS：万级
- RocketMQ TPS：7万条/S，阿里

### （2） 数据可靠性
kafka：多副本，高
RabbitMQ：多副本，高
RocketMQ：异步实时刷盘/同步刷盘

### （3） 服务可用性
均有

### （4） 功能
kafka：适合海量数据收集，适合高吞吐
RabbitMQ：支持更多消息队列功能，适合数据一致性，稳定性，高可靠
RocketMQ：阿里，交易场景

### （5） FMQ（2018）
适合消息异步订阅，消息队列，一致性，可靠性
而不是日志海量存储，高吞吐

zk性能
tps：1W+ / s，弱