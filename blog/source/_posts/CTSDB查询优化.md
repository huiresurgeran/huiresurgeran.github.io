---
title: CTSDB查询优化
date: 2021-11-09 21:24:09
tags: [ctsdb, search]
categories:
	- 答辩
	- 大数据
---
## 1. from/size
避免分页深度

## 2. 实时聚合 =》 rollup
实时聚合影响：内存占用，分桶多的数据

## 3. 排序优化
- index sorting，docID和indexSorting的顺序一致
- 遍历 =》 提前中断
- 降低了写入性能
- 提高了查询性能
	- 预排序 + after key + 提前中断
	- 数据压缩率

## 4. 查询剪枝
segment，最大值最小值
遍历 =》 跳过

## 5. routing
一次聚合 + 分片级并发

## 6. rollup
流式聚合（并发分页）
indexSorting（提前中断）
查询剪枝
routing（分片级并发）

## 7. composite
