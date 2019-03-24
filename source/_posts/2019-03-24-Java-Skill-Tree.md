---
title: Java Skill Tree
date: 2019-03-24 20:48:57
tags: Java
categories: tech
---

让我梳理一下，作为Java工程师我们应该掌握的知识点和技能。作为整理总结也好提醒自己你还差什么，如有补充欢迎提醒

## Java 基础

Java基础当然是围绕JDK展开，JDK的不断也在提醒我们需要与时俱进不断更新相关技能，Java基本语法规则除外，应该掌握熟知以下内容

<!-- more -->

1. 常用类型相关： String StringBuffer StringBuilder，基础类型和包裹类型、Vector、不同的List、Map，以及ConcurrentMap
1. 异常处理：Exception和Error，final、finally、finalize
1. 动态代理和反射，动态生成Class
1. 输入输出流、IO处理方式、NIO
1. 强引用、软引用、弱引用、幻引用
1. 常见设计模式：Adapter、Proxy、Factory、Singleton、Observer等等
1. 多线程，线程安全、synchronized、ReentrantLock，并发包中的常用工具类，例如：CountDownLatch、BlockedQueue等、线程池和CAS操作
1. JVM基础，Java类加载过程、JVM内存区域划分，堆内、堆外使用诊断、常见的GC机制及优化
1. Java新特性Lambda表达式、Stream等
1. 常见算法Java实现，LeetCood刷刷题

## 通讯协议

1. JSON XML，虽然xml已经被淘汰，但是还是需要了解的，毕竟还有很多老系统在使用
1. HTTP Restful，已经成为最常见的通讯方式
1. RPC 例如：Dubbo、gRPC、Thrift，非常常见及普遍使用
1. WebService SOAP，已经很少使用
1. RMI，较少使用
1. MQ Queue Topic，不同MQ厂商细节与写差别，需要分别对待
1. TCP/UDP Socket，最传统的方式但是还是需要掌握的

## 常用框架

1. Spring 全家桶，Spring mvc、Spring boot、Spring cloud，至于其他类似框架，我就不学了，遇到再说了。
1. Netty，经典网络请求框架，广泛使用，以及老牌apache mina
1. Guava，Google开发的Java工具包、核心库里面有丰富的功能
1. Disruptor，高性能吞吐处理框架

这里会设计大量并发设计模式可以深入研究

## 数据库

1. JPA、Hibernate、MyBatis，数据类型、事务等
1. MySQL 广泛使用的关系型数据库，内容很多
1. Oracle 大型企业商用场景
1. DB2 金融行业为主
1. Redis、mongodb（不推荐使用）
1. ElasticSearch 全文搜索，Log日志处理ELK
1. 其他：PostgreSQL、TiDB

数据库，停留在实际项目中使用过，并不深入，懂得基本常用功能，可进行表设计，不熟悉数据库优化和其他复杂维护操作。

## 其他技术

1. Linux、Docker、K8S、DevOps
1. Tomcat、Nginx
1. Https及证书、域名
1. DNS、CSDN、Load Balance
1. 常用云服务 ECS、OSS等
1. Git SVN

其实以上内容，不少也适用于其他语言的工程师，例如Python和Go等

先整理这些，内容还将不断丰富，已督促自己全面掌握和使用。