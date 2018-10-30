---
title: Registration and Discovery - Spring
date: 2018-10-29 09:52:26
tags: [spring cloud, CAP]
categories: tech
---

关于Spring Cloud支持的注册发现服务，一直想了解一下对比情况，今天无意间发现这个对比文章，简单直接，可以给你一个直观的初步认识。
转载原地址：[服务发现比较:Consul vs Zookeeper vs Etcd vs Eureka](https://luyiisme.github.io/2017/04/22/spring-cloud-service-discovery-products/)
这里仅贴出总结表格，稍作调整。其它内容请参见原文

| Feature              | consul                 | zookeeper             | etcd              | euerka                       |
|----------------------|------------------------|-----------------------|-------------------|------------------------------|
| 服务健康检查         | 服务状态，内存，硬盘等 | (弱)长连接，keepalive | 连接心跳          | 可配支持                     |
| 多数据中心           | 支持                   | —                     | —                 | —                            |
| KV存储服务           | 支持                   | 支持                  | 支持              | —                            |
| 一致性               | raft                   | zab                   | raft              | —                            |
| CAP                  | ca                     | cp                    | cp                | ap                           |
| 使用接口(多语言能力) | 支持http和dns          | 客户端                | http/grpc         | http（sidecar）              |
| Watch支持            | 全量/支持long polling  | 支持                  | 支持 long polling | 支持 long polling/大部分增量 |
| 自身监控             | metrics                | —                     | metrics           | metrics                      |
| 安全                 | acl/https              | acl                   | https支持（弱）   | —                            |
| Spring Cloud集成     | 已支持                 | 已支持                | 已支持            | 已支持，2.0闭源              |

<!-- more -->

感谢原作者的分享，并贴出与其文中一处不一样的观点，zookeeper的一致性协议不是Paxos，应该是ZAB。
此观点依据：[The core consensus algorithm of ZooKeeper is not Paxos](https://www.elastic.co/blog/found-zookeeper-king-of-coordination#consistency-algorithm)

另外，euerka宣布2.0闭源后，目前来看Consul是替代euerka一个不错的方案。

## CAP 简单解释
- 一致性(Consistency) (所有节点在同一时间具有相同的数据，客户端请求到同样的数据结果)
- 可用性(Availability) (保证每个请求不管成功或者失败都有响应)
- 分隔容忍(Partition tolerance) (系统中任意信息的丢失或失败不会影响系统的继续运作)

CAP 不可能全都同时满足，这个已经被证明过了，这里先记住结论吧，具体解释网上文章很多。

## Raft 演示
相比Paxos，raft更容易被人理解和接受，看一下这个[演示动画](http://thesecretlivesofdata.com/raft/)吧，简单直观。