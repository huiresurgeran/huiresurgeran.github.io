---
title: 数据处理说明
date: 2021-11-11 22:26:01
tags: [data, trace]
categories:
	- 答辩
	- 模调
---

数据处理延迟：2min
理由：
- 1min，数据累计时间（99%）
- 1min，数据延迟buffer（30s重启）
- 业务可接受

保留10min内的数据，去做数据计算
	- 若延迟难以恢复，直接写入CTSDB，不经过ckv+
	- 控制数据丢失情况，2-10min
	- 考虑到需要重启