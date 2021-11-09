---
title: hermes & es & ces & ctsdb
date: 2021-11-09 20:22:30
tags: [big-data]
categories:
	- 答辩
	- 大数据
---
## 1. hermes
写入搜索性能：HDFS

## 2. es
存储规模限制：集群管理成本，信息同步成本

写入搜索性能：本地磁盘

## 3. ces
存储规模：集群管理成本，信息同步成本

写入搜索性能：同CTSDB

特点：适合检索

## 4. CTSDB
#### （1）存储规模
集群大小，管理成本，集群元数据大同步慢

#### （2）低成本存储
编码压缩算法 + rollup + 过期时间

#### （3）写入搜索性能
- 写入
	- 写入内存，之后再写入磁盘
	- 文件裁剪优化
- 查询
	- 缓存
	- 倒排索引
	- 存储模型，segment合并，按时间序分层合并
	- 执行引擎优化，index sorting + after key + 提前中断
	- 文件裁剪优化

#### （4）时序特性
时序场景管理能力
- metric封装
- 按时间滚动生成子表
- 自动产生自动销毁
- 过期自动清理

#### （5）聚合
rollup
- 流式聚合（分页并发）
- 查询剪枝（segment）
- index sorting（提前中断）
- routing（分片级并发）

#### (6) CTSDB manager
每个地域一个
功能：
- 任务下发
- index自动创建/删除
- 变配
- 监控
- 发货

#### （7）CTSDB-GATEWAY
VPC网络：安全性
VPC的VIP：负载均衡

