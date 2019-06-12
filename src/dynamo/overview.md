# Dynamo概述

## Dynamo论文的价值

[Dynamo论文](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf) 发表于2007的SOSP（ACM操作系统原理会议），并在2017年获得SOSP的名人堂奖。DynamoDB起初设计给Amazon内部系统使用，于2012年在AWS作为服务正式售卖。DynamoDB的在设计上融入了一致性哈希、草率仲裁、暗示移交、去中心化（Gossip协议）等技术实现了一个最终一致性的KV存储系统，并且随着AWS的发展证明了最终一致性方案在生产环境的可行性。

相对Google的GFS、Bigable、Spanner的中心化、强一致（Chubby）等技术方案DynamoDB涉及的上述技术给社区贡献了一个新的方案设计思路，尤其随着AWS在云计算霸主地位的巩固，Dynamo论文中分布式架构的设计也得到了社区的认可。


## Dynamo约束与假设

#### KV存储系统
Dynamo作者之一的Werner Vogels在2017年回顾Dynamo十年的文章中描述，在设计Dynamo之前对Amazon内部使用数据库的方式进行了分析，结论是大约70%的操作都是键-值类操作（仅使用一个主键，返回一个单行数据），大约20%的操作会返回一组行数据，但也仍然位于单个表上（NoSQL时至今日扔如火如荼的发展，相信绝大部分互联网公司使用模型亦然），所以放弃了SQL接口选用简单的KV接口。

#### ACID取舍
1. 原子性（A） 和 隔离性（I）：只允许单一的Key更新没有跨行跨表的多Key更新操作。
2. 一致性（C）：和一致性相比Dynamo更看重高可用性（CAP理论），所以选择了弱一致性。
3. 持久性（D）：多副本 + 缓存（可选）+ 磁盘，在不影响可用性的前提，尽最保证数据的可靠性。

#### 运维效率
Dynamo需要运行在异质硬件环境上，能够在各种故障（硬件的差异、机器故障、网络问题、机房故障、区域故障等场景）和运维操中（节点上下架、快速扩容、裁撤等场景）保证服务的SLA，这些需要在成本、可用性、持久性之间做权衡。


## Dynamo特点

1. 高度可扩展
	* 可扩展一直是Amazon系统的第一追求，从单集群规模、到跨地域（全球）部署，从单引擎到多引擎适配等多维度Dynamo都具备高度可扩展的能力。
2. 高可靠
	* 数据的可靠性对于AWS服务至关重要，如果因为硬件故障等原因导致用户数据丢失，对于AWS用户来说是不可接收的。
3. 高性能
	* DynamoDB SLA协议保证99.9%的请求在300ms内完成，AWS官网保证Dynamo内存引擎的耗时在几毫秒内。
4. 跨地域部署
	* DynamoDB利用去中心化方案保证了跨地域部署，毕竟中心化解决方案在跨地域部署中有跨地域网络频繁抖动等问题。
5. 多主
	* 多主指的是允许用户想同一Sharding内的多个副本同时写（不区分master/slave），Dynamo提供了写冲突解决方案。多主 + 跨地域部署实现了“异地多活”。
6. 不停写
	* Dynamo通过NWR方案权衡读写的性能，如果Sharding内可用副本数少于配置的W个数写会收到影响，Dynamo可以保证只有有节点存活就不会停止写，保证了写的高可用。

[后续文章](https://github.com/joeylichang/joeylichang.github.io/blob/master/src/dynamo/desgin.md)会详细介绍Dynamo的分布式设计方案，在最后还会重新审视上述特点（或者说设计目标）是如何实现的。