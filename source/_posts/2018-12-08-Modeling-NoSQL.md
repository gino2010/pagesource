---
title: Modeling & NoSQL
date: 2018-12-08 21:54:50
tags: Design
categories: tech
---
建模与NoSQL，这一切源于最近对商品建模的思考。
场景，当你有很多产品，他们有不同的属性，并且差异较大，今后新增产品的属性又不确定，那么你的数据应该如何存储，如何建模呢？
<!-- more -->

## Modeling
商品数据的建模方式，其实我们平时经常使用，并不少见，但大多时可能只是模仿使用，并没有对其进行深入思考或整理归纳。
直观的数据库建模方式就是通过一张商品表对其进行记录，也是我们最常见的行建模方式（Row Modeling）。
稍微优点开发经验的朋友理解会意识到，这样的实体对象entity对应一张数据表的方式，不能很好满足我们需要面对的场景。其不适应将表现为：
1. 属性差异过大，造成表字段数量跟随产品类别变化，并且巨大
1. 属性差异过大，造成数据稀疏，表存储空间浪费
1. 今后新增常品，表结构需要调整非常不灵活
等等，就不一一列举了，这个思路估计连当今大学毕业设计的同学都不会犯的错误。
那么应该怎么做，开发者一般会想到字段拆分，表关联方式处理。那么有什么规范可以遵循吗？
答案是：有的，[EAV（Entity Attribute Value）](https://en.wikipedia.org/wiki/Entity%E2%80%93attribute%E2%80%93value_model)方式。
他把一个实体拆成三部分，进行建模：
- Entity 实体，即事物本身，也就是商品
- Attribute 属性，将事物所拥有的属性
- Value 值，商品的具体属性对应的值
简单说，这样形成了，多张关联的数据表，维护商品信息，并实现灵活调整。

这样做的坏处是什么？建模上满足了我们的场景需求，但在其他方面有不如意的地方，主要表现在：
1. 数据表多，数据结构复杂，维护繁琐
1. 查询语句复杂，严重影响查询性能
1. 实际数据使用时，需要大量行列转换操作，可能需要响应试图结构支撑，是CPU密集型操作，对数据库压力大

怎么解决上面的问题，有些朋友可能第一时间想到的是NoSQL，让我们看看他们适合不？

## NoSQL
说NoSQL是否合适，需要先了解NoSQL的分类
参考网上常见分类有四种，如下：

### 键值存储数据库
- 相关产品：Redis、Riak、SimpleDB、Chordless、Scalaris、Memcached
- 应用：内容缓存
- 优点：扩展性好、灵活性好、大量写操作时性能高
- 缺点：无法存储结构化信息、条件查询效率较低

### 列存储数据库
- 相关产品：BigTable、HBase、Cassandra、HadoopDB、GreenPlum、PNUTS
- 应用：分布式数据存储与管理
- 优点：查找速度快、可扩展性强、容易进行分布式扩展、复杂性低
- 缺点：功能相对局限

### 文档数据库
- 相关产品：MongoDB、CouchDB、ThruDB、CloudKit、Perservere、Jackrabbit
- 应用：存储、索引并管理面向文档的数据或者类似的半结构化数据
- 优点：性能好、灵活性高、复杂性低、数据结构灵活
- 缺点：缺乏统一的查询语言

### 图形数据库
- 相关产品：Neo4J、OrientDB、InfoGrid、GraphDB
- 应用：大量复杂、互连接、低结构化的图结构场合，如社交网络、推荐系统等
- 优点：灵活性高、支持复杂的图形算法、可用于构建复杂的关系图谱
- 缺点：复杂性高、只能支持一定的数据规模

所以，如果使用NoSQL存储多变的字段数据内容，第一优选就是文档数据库，甚至可以考虑使用ElasticSearch来完成更为复杂的搜索需求。
这样的方法是否合理呢？

## NoSQL 方案问题
增加系统开发复杂度，需要同时操纵两种不同类型的数据库。毕竟原有的业务数据还需要在关系型数据库中保存，同时无法合理关系化的字段数据将存入NoSQL数据库中。

1. 需要考虑两个数据库保证数据一致性问题
1. 数据使用时需要考虑合理的查询方式，拼接数据

简单说数据使用和维护的相关系统实现架构都需要有较好的设计和开发支撑，系统复杂度大，运营维护成本高。
同时不否认，这种方式也有其实际应用价值和优势，也是一种常见的处理方式。

那有没有其他方案？

## JSON 类型
数据引入JSON类型也逐步成为主流，也逐渐成熟。
记得最早是PostgreSQL 9.3的一次创新型实验，之后就是MySQL 5.7也添加了相应的支持，Oracle 12cr1

依MySQL为例，简单看一下其功能

创建表
```sql
CREATE TABLE t_json(id INT PRIMARY KEY, sname VARCHAR(20) , info  JSON);
```

插入数据
```sql
-- 插入含有json数组的记录
INSERT INTO t_json(id,sname,info) VALUES( 1, 'name1', JSON_ARRAY(1, "abc", NULL, TRUE, CURTIME()));

-- 插入含有json对象的记录
INSERT INTO t_json(id,sname,info) VALUES( 2, 'name2', JSON_OBJECT("age", 20, "time", now()));
INSERT INTO t_json(id,sname,info) VALUES( 3, 'name3', '{"age":20, "time":"2018-07-14 10:52:00"}');
```

```sql
-- 查询记录
SELECT sname,JSON_EXTRACT(info,'$.age') FROM t_json;
SELECT sname,info->'$.age' FROM t_json;
-- 查询key
SELECT id,json_keys(info) FROM t_json;
```

所以，关系库中的JSON类型是可以比较理想的满足我们使用场景，既兼顾了原有业务数据也保证了数据的灵活性，同时维护和使用技术环境统一，降低系统复杂度。
缺点，性能略有损失，代码处理略有复杂。针对JSON数据内容也是可以建立索引的，提升查询速度，这个的使用也许合理设计。

## 总结
文章从第一天建立到今天（1-4）补完，经历快一个月。这一个月本人也经历了很多，同时，做了一个从业以来最任性的决定，至于结果如何，那就看自己的努力了。

新的一年开始，感谢我接触过的所有人，感谢你们的帮助。希望个人今后可以走的更好。
祝大家安康！

## 参考
[EAV 模型](https://www.cnblogs.com/sthinker/p/5882520.html)
[NoSQL 分类](https://www.cnblogs.com/quinnsun/p/NoSQL_2.html)
[数据库 JSON 类型](https://www.cnblogs.com/ooo0/p/9309277.html)



