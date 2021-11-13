---
title: kafka
date: 2021-11-13 21:20:33
tags: [big-data, kafka]
categories:
	- 答辩
	- 大数据
---

## 1. 作用
- 削峰填谷
- 防止数据丢失

## 2. 集群
- 天然支持集群，一个节点也是集群
- 注册到同一个zk上，就是同一个集群
- id区分节点

## 3. 分片
- partition（文件类）下有多个segment片段（文件）
- 每个segment对应一个.log文件和一个.index文件

### （1） 分片的功能
- 负载均衡：均匀地分布在多个broker
- 提高读写并发度和吞吐量：消费者
- 系统伸缩性： 根据业务量配置集群
- 提高集群故障处理能力：分片拆分，只影响一部分数据

### （2） 分片策略
- 消息发送分区分配：指定分区 + key（murmur hash） + 轮询
- 消费者分区分配：consumer leader负责，也是rebalance过程，分区按照数字顺序排序，消费者按名称的字典序排序
	- RoundRobin:依次顺序分配
	- Range:分区总数/消费者数
- 分区放在哪个节点：master负责，随机挑一个broker放分区0，依次放置其他分区；副本分片基于主分片，按顺序以此方式

### (3) 分片缺点
- 分片太多，导致集群不可用
- 副本数据同步，数据延迟
- 需要大量内存，不适合资源紧张的环境

## 4. 消费者
- 消费能力横向伸缩性
- 容错性/故障容灾
- Rebalance，减少消费抖动（consumer停止消费）
- 高性能，一个消费者 =》 多个消费者

## 5. 副本

### （1） 作用
- 数据备份，数据恢复，数据高可靠和完整性
- 防止单点故障，支持容错
- 故障自动转移

### （2） 数据一致性
- 对外一致性：领导者副本机制，follower不对外提供服务，follower被动、定期、异步拉取leader数据
- 对内一致性：ISR副本集合
	- Follower落后时间，10s
	- 动态调整
	- 平衡数据丢失率和吞吐率
- 复制一致性：leader和follower都commit后，更新到HW，才能对外读取

### （3） 容错性
broker异常或者退出，分片会把leader切换到ISR中对应的follower节点上，继续对外提供写入服务

### （4） 数据恢复
follower从leader进行数据同步

### （5） 全部宕机
- 等待ISR中的一个恢复，选他为leader：降低可用性
- 选择第一个恢复的副本作为leader，无论是否在ISR：数据丢失

### （6） 副本分配
副本分片基于主分片的选择，按顺序防止

## 6. 选举

### （1） controller
- zk中创建临时节点，第一个注册成功的，成为controller
- 功能
	- 主题管理
	- 分区重分配
	- preferred leader选举
	- 集群成员管理
	- 集群元数据

### （2） leader
ISR集合，分区选择算法，ISR列表中的第一个

### （3） 消费者选主
- 第一个加入消费者组的
- 重新分配：随机
- 消费者协调器负责选主，主负责分配分片

## 7. 故障处理

### （1） controller故障
选举新的controller，zk注册，第一个注册成功的是controller

### （2） leader故障
- controller从ISR列表中选出新的leader
- ISR列表中的第一个
- 数据一致性：其余follower，log文件中高于HW的截掉，从新的leader同步数据
- 副本数据：controller选一个节点，分配follower

### （3） follower故障
- 重新从leader同步数据，追上leader（LEO > HW）
- 重新加入ISR中

### （4） 写入，主挂了
重试

### （5） 写入，follower挂了
从ISR中剔除，等待恢复

### （6） 全部挂了
- 非ISR列表中的节点选为leader：数据丢失，对外可用
- ISR列表中第一个恢复的选为leader：数据不丢失，对外不可用

### （7） 异常切换耗时
leader挂了，zk监听，6s内发现

## 8. 水平扩展
- broker启动/停止：partition再分配
- partition增加/减少：提高/降低并发处理能力

## 9. kafka api

### （1） consumer api升级

| | 旧API | 新API |
| offset | zk管理 | kafka中的topic管理 |
| 机器连接 | zk | kafka broker |
| consumer group | 多线程，一个线程一个分区 | 单线程，非线程安全，轮询的方式；支持单线程和多线程 |
| 心跳 | 无 | 双线程，用户主线程 + 心跳线程；定期发送心跳请求，确保存活性 | 

- 好处
	- 旧的API：offset依赖zk，数据量大且消费频繁，zk是瓶颈
	- 新的API：分片分配，依赖broker coordinate和consumer leader，减少zk依赖
	- 新的API：增加心跳，减少rebalance
	- 新的API：可以批量拉取数据

### （2） offset提交
kafka中的一个topic，保存每个分区的偏移量。

更新方式
- 自动：at most once，频率5s
- 手动，at least once
	- 同步提交：失败重试，影响吞吐量
	- 异步提交：不重试，可回调，注意顺序

### （3） 新版本使用方案
- 多线程 + 多kafka consumer实例
	 - 优点：方便实现；速度快，无线程切换；维护分区消费顺序
	 - 缺点：占用更多系统资源；受限于分区数，扩展性差；处理超时
 - 单线程 + 单线程kafka consumer + worker线程池
 	- 优点：可独立扩展线程数；伸缩性好
 	- 缺点： 实现难度高；难以维护分区消息顺序；处理拉长，不易于位移提交管理

## 10. 数据丢失
- leader存储，follower未同步，leader故障，切换，数据丢失
- leader存储，follower同步，未写入磁盘，两个都挂掉，数据丢失


## 11. 为什么kafka很快
- 顺序读写：追加到本地磁盘文件末尾
- Page Cache：操作系统的内存而不是JVM空间内存
	- 避免Object消耗
	- 避免GC问题
- 零拷贝：sendfile方法直接将数据从内核空间的度缓冲区拷贝到内核空间的socket缓冲区，不经过用户空间缓存区
- 分区分段 + 索引：segment文件有.index
- 批量写入 + 批量压缩，减少网络IO


## 12. 为什么选择kafka
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

## 13. zk

### （1） zk性能
tps：1W+ / s，弱

### （2） zk存储信息
集群元信息

- controller数据
- brokers数据
	- 每个broker有哪些topic + partition
	- leader，ISR等的state状态
	- broker的ids，临时节点的ids
- 消费者不再向zk注册，由消费者协调器统一管理

