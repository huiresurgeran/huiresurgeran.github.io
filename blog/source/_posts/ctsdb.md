---
title: ctsdb
date: 2021-11-13 21:20:37
tags: [big-data, ctsdb, es]
categories:
	- 答辩
	- 大数据
---

## 1. 集群
相同的集群名称 + 同网关下的机器，组成同一个集群
集群规模限制：集群状态信息，越大更新越慢，所以不能无限制的增大集群

## 2. 节点
- master节点
	- 功能：
		- 选举
		- 维护集群状态，分片分配等
- data node
- coordinating node

## 3. 索引

### （1） 拆分优点
- 减少索引之间的相互影响：配置变更，不影响其他
- 分片数设置，与数据量达到平衡
	- 分片太多，拆成字写入/查询，拒绝率上升
	- 分片太多，分段小，合并开销增加，小segment降低查询效率
	- 分片太少，单个分片太大，数据倾斜，机器故障
	- 分片太少，单个分片太大，需要过滤的数据增加
- 自定义配置，适应不同的业务需求

### （2） 拆分配置
- 数据量较小的：3-5个分片，每个分片<100G，2副本
- 数据量较大的：20-50G/分片，不超过50个分片

## 4. 分片

### （1） 作用
- 负载均衡
- 备份恢复迅速
- 分布式存储扩展，可伸缩性
- 提高查询并行度，提升读写性能

### （2） 分配分片
allocator（权重） + decider（配置规则）
- 新建的索引：先主分片再副本分片
	- 找到权重最低的节点列表（分片数量最少的机器），递增排序（allocator）
	- 一次遍历节点列表，判断是否需要分配（decider）
- rebalance超过阈值
	- 最不均衡索引，最先处理
	- 比较不同节点上分片的权重，确定重新分片是否能改善
- 权重计算
	- 分片weight：节点总分片 - 节点平均分片
	- 索引weight：索引在节点上的分片 - 索引在节点上的平均分片
	- weight = 分片权重 * 分片weight + 索引权重 * 索引weight
- 配置规则
	- 节点分片数限制 --- 负载均衡
	- 相同分片不在同一个节点 --- 负载均衡
	- rebalance并发数 --- 并发数量规范
	- recoverage并发数 --- 并发数量规范
	- include/exclude配置 --- 条件限制规则
	- 磁盘空间阈值（最低，最高） --- 条件限制规则
- rebalance触发
	- weight >= balance.threshold = 1.0

### (3) 文档写入哪个分片
routing（默认为id）
shard = hash（rounting） % number_of_primary_shards

### (4) 查询分片
routing 或者是 过滤所有分片

## 5. 副本

### （1） 作用
- 提高查询吞吐量，并发查询
- 高可用：保证数据完整性；节点异常恢复，副本分片提升为主分片
- 防止单点故障：主副本分片不能再同一个节点
- 容错：副本分片提升为主分片

### （2） IN-SYNC列表
- ES主节点维护，可供的分片副本列表
- primary挂掉了，master从in-sync中选一个replica

### （3） 数据一致性
- 主/副写入成功：最终一致性，refresh前，短暂不一致
- 主写入失败：失败重试
- 备写入失败
	- 强一致：重试
	- 非强一致：
		- 副本标记为不可用，不对外提供查询，后续恢复写入
		- 更新节点信息前，短暂不一致
- 主副本写入成功，还未写入磁盘flush（1min）：translog + check point

### （4） 数据恢复
translog + 数据备份
- recovery：translog
	- 主分片：translog
	- 副分片：从主分片中拉取lucene分段（可跳过）和translog
- 数据备份：HDFS

### （5） 容错
- master node宕机：自动master选举
- replica容错
	- master将replica提升为primary
	- master复制缺失的relica到节点上

## 6， 选举

### （1） master
- activeMaster列表：ping，过滤不通的，排除本地ip
- Bully算法：参选人数过半 + 投票人数过半 + 优先级
	- 优先级：版本号 + id
		- 版本号越高，优先级越高
		- id越小，优先级越高
- masterCandidate：配置
	- 集群状态版本编号 + 节点id
- 探测到节点离开，若当前节点数不过半，则放弃master身份，重新加入集群，防止脑裂

### （2） 集群元信息
- 各节点上报元信息，根据版本号确定最新的元信息
- 参与选举的元信息要过关，master发布也要过半

### （3） 主分片primary
master操作，选择集群元信息中“最新主分片列表”的最新分片（根据UUID）

### （4） 副本repllica

## 7. Recovery

### （1） index recovery
- 分片分配成功后进入recovery
- 用于恢复数据
	- 主分片，数据未刷盘
	- 副分片，数据未刷盘/副本未同步主

### （2） 主分片Recovery
重放translog

### （3） 副分片Recovery
- 等主分片恢复之后再操作
- 恢复期间允许索引操作
- 数据一致性：版本号

## 8. 集群数据

### （1） 正排索引
- 优点：易维护
- 缺点：搜索代价大，耗时大
- 结构：id - 数据
- 作用：排序/聚合

### （2） 倒排索引
- 优点：搜索快
- 缺点：难维护
- 结构：term index - term dictionary - id list
- 作用：查询

### （3） doc value
- 特点：序列化到磁盘，不支持analyzed
- 作用：排序，聚合

### （4） field data
- 特点：放在内存，JVM内存堆，支持analyzed
- 作用：排序，聚合

## 9. segment
每个分片有多个segment，每个segment占用内存，文件句柄，文件

### （1） refresh
index buffer -> file system cache
1s/1次 -> 10s/次

### （2） flush
file system -> disk
30min/次，translog限制

### （3） 影响
- 内存：每个segment的cache内存，随着gc不会释放掉，过大过多的segment会导致系统运行没有内存，查询超时
- 数据查询：查询遍历每个segment，过多的segment导致查询速度变慢

### （4） 优化措施：释放cache
- 删除索引
- force merge
- 关闭索引：文件在磁盘，释放内存

## 10. 写入

doc -> translog
			-> disk (flush, 大小 > 512M)
    -> index buffer
    		-> file system cache (refresh, 1s/次)
    					-> disk (flush, 30min/次)

- refresh
	 - open，可以被搜索
	 - 记录commit point
 - flush
 	- 持久化，删除translog
 	- 记录commit point到磁盘

## 11. 查询/检索
query then fetch
拆分到每一个分片：主/副本分片，随机轮询

## 12. 故障处理
副本 + primary选举

### （1） master
重新选举，Bully，投票

### （2） primary
master将In-sync列表中的Replica提升为primary
主分片选举，集群元信息记录的“最新主分片列表“（UUID）
数据恢复：translog

### （3） replica
master将这个node从In-sync中移除，在其他node启动shard
数据恢复：同步primary，primary segment + translog

### （4） 全部异常
translog + 数据备份

### （5） 切换过程
无感知，影响查询写入性能（重新分配分片，分片切换）

## 13. 查询分页方式

### （1） from/size
mysql类型，默认分页深度10000，超过报错

### （2） search after：实时请求下一页，不指定页数
利用上一次查询返回的游标，进行下一页查询
需要增加排序字段
开销小很多

### （3） scroll api：查询大量文档，实时性不高
第一次查询时，生成快照，之后的每次翻页，都是基于这个快照的结果，不会查询到新数据
第一次查询，返回scroll id
每个查询都可以设置一个过期时间

## 14. 防止拖库

### （1） CTSDB
- 用户名密码 + VPV
- 其他人操作，需要授权

### （2）接口层
openapi
- 限流
- 使用from + size，限制10000
- 鉴权，证书下发

### （3） 逻辑层
- 加密字段
- 密码加密 =》 字典攻击 =》 加盐

### （4） es
- 身份认证：登录
- 传输加密：防抓包，启动TLS，TLS加密通信（证书）
	- 集群内部加密，TLS协议 + 证书
	- 集群外部加密，https/ssl + 证书
- 用户鉴权：角色，基于角色的访问控制
- 日志审计


## 15. 其他
add快，buld request慢 =》 可以增大并发，add操作不会被阻塞 =》 改成同步
