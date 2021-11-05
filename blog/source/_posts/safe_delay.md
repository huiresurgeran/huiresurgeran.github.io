---
title: 神盾对账延迟故障复盘
date: 2021-02-25 20:49:22
tags: safe
categories: safe
---

2.19 开始有业务反馈神盾出现延迟(shuangchen)
![](shuangchen_1.jpg)

初步来看，agent上报到svr延迟较大，应是大量数据上报超时堆积导致，每次agent上报超时，都会等待20s

2.20 延迟进一步加剧，大约有10+对账，处于延迟边缘，引发大量延迟事件

2.22 下午，发布新版本（增加耗时日志打印），延迟进一步加大，到5点逐渐扩展到2h
管理员收到大量告警，agent等待svr处理超过20s

2.22 晚上，发布多个版本，修改数据库线程池大小，增加超时时间设置，延迟降低，出现漏对事件

2.23 早上，延迟再次增加，且出现大量漏对事件，数据库备机压力大，dba增加索引，svr修改线程池，调整超时时间，延迟逐渐减少，漏对逐渐减少

BasicDataSource 改为 HiKariDataSource
HikariCP是一个轻量级的十分快速的JDBC连接池框架。
使用了以下这些技术
  * 字节级别的工程技术
  * 微观层面的优化
  * 集合框架的智能使用
demo
```
  HikariConfig config = new HikariConfig();
  HikariDataSource ds;

  config.setJdbcUrl("jdbc_url");
  config.setUsername("xxx");
  config.setPassword("xxx");
  config.addDataSourceProperty("cachePrepStms", "true");
  config.addDataSourceProperty("prepStmtCacheSize", "250");
  config.addDataSourceProperty("prepStmtCacheSqlLimit", "2048");
  ds = new HikariDataSource(config);


```

HikariConfig 配置类，初始化数据源
HikariCP可以检测到连接泄漏


索引增加字段（索引机制还是挺复杂的，有时间需要好好研究）
跟数据量，数据过滤比例，排序都有关系
1. 返回数据比例>30%，不使用索引
2. where 和 order条件，先where后order
3. 没有过滤效果的条件，可以不建立索引
4. 时间段查询，>= && <= 索引效果没有=效果好
5. like索引效果，没有in效果好

2.24 下午，上报处理逻辑优化，尽量减少查询次数，减少排序，悲观锁改为乐观锁

后续，出现数次safe status更新失败告警，部分数据量大的表耗时偶尔较长，部分对账同时间上报多条数据，待优化
![](update_safe_status_fail.png)
