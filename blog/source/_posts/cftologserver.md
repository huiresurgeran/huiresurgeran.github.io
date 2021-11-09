---
title: cftologserver
date: 2021-11-09 20:56:11
tags: [log]
categories:
	- 答辩
	- 日志服务
---
## 1. 模块化

#### （1）五个模块

#### （2）资源配置
kafka集群：深圳 + 上海
消费者：分片16
cacher：ThreadPoolExecutor，100个线程，2000个无界队列
processor：每个尾号一个线程（尾号hash % 线程数）
writer：1/分片 + 1/processor

#### （3）机器说明
B70
内存：125G，16G * 8 = 128G
硬盘：300G
物理CPU：2
核数：12
逻辑CPU：48（2 * 12 *2）
CPU Mhz：2301.00
RAID：RAID1
网络：10GE * 2，10G以太网网络

## 2. 线程池

#### （1）作用
降低资源消耗
提高系统运行速度
提高可管理性

#### （2）使用
ThreadPoolExecutor
核心线程池：100
阻塞队列：ArrayBlockingQueue，2000
线程池：200
饱和策略：DiscardPolicy

## 3. 缓存
同步锁，synchronized，多线程环境，线程安全同步锁