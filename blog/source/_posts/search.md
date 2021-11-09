---
title: 慢查询 & 限流 & 限频
date: 2021-11-09 21:09:20
tags: [server, search]
categories:
	- 答辩
	- 其他
---
## 1. 慢查询

#### （1）ctsdb：30s超时
影响：CPU，内存

优化：
- 索引，不用通配符
- agg改成composite
- 分片数据
- 聚合拆分成多个，避免分桶太多
- 独立协调节点
- routing
- 增加刷新间隔
- 配置熔断：内存限制

#### （2）fmha
影响：session长期占用，cpu、内存升高，可用性降低

优化：
- 主键
- 索引减少like
- 拆表/中间表
- 查询分解
- limit优化

## 2. 限流

#### （1）接口限流
- 计数算法：简单
- 滑动窗口：存储空间大
- 漏桶算法
- 令牌桶：允许一定程度的并发

#### （2）数据上报限流

writer：
- RateLimiter，令牌桶
- 2w/s/线程

processor：
- writer queue满了，熔断，蓄水池
- 恢复，writer queue
- 蓄水池，writer queue

Loader：
- LoadQuata，数值判断大小
- 6w/s/线程

