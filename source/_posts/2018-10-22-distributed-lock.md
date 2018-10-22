---
title: Distributed Lock
date: 2018-10-22 09:09:33
tags: [mysql, zookeeper, redis]
categories: tech
---
分布式锁，在微服务，多服务器水平扩展以及高并发的场景下，要保证共享资源正确处理，这个锁很重要。
我了解到了三种实现方式：
- 数据库方式，基于主键唯一性约束或行级锁实现
- Zookeeper实现，基于其临时有序节点实现
- Redis方式，基于setNX操作实现

三种方式各有优缺点，在恢复此博客之前我已经实现了基于Redis版本的分布式锁，现在把另外两种加以实现，并做相应的对比。

<!-- more -->

## [Mysql](https://github.com/gino2010/javatech/tree/master/distributed_lock_db)
数据库方式实现分布式锁，主要用三种形式：
1. 乐观锁，version，通过where条件中的version决定是否可以更新成功，从而获得锁
1. 主键唯一约束方式，是否可以正常写入数据，从而获得锁
1. 数据库自身锁 for update 来决定是否可以获得锁

这里我们不讨论乐观锁，为了和zookeeper、redis的锁做对比，这里采用使用主键唯一约束方式实现锁。

创建锁表，其中method_name需要索引并唯一，时间默认
```sql
CREATE TABLE method_lock
(
    id int PRIMARY KEY NOT NULL AUTO_INCREMENT,
    method_name varchar(64) NOT NULL,
    update_time timestamp not null default current_timestamp on update current_timestamp
);
CREATE UNIQUE INDEX method_lock_method_name_uindex ON method_lock (method_name);
```
关闭自动提交，开始手动提交，保证事务中的操作原子性
```java
connection.setAutoCommit(false);
```
获得锁，两个操作，删除超时的锁（超时自动释放机制），插入数据获得锁
```java
PreparedStatement preparedDelete = MySQLConfig.connection.prepareStatement(clearSQL);
preparedDelete.setString(1, name);
preparedDelete.setInt(2, 3);
preparedDelete.executeUpdate();

PreparedStatement preparedInsert = MySQLConfig.connection.prepareStatement(insertSQL);
preparedInsert.setString(1, name);

MySQLConfig.connection.commit();
```
如果提交失败，进行重试，设定时间间隔和重试次数
```java
public static boolean getLockTimes(String name, int times, int interval)
```
释放锁，直接删除锁占有的记录
```java
PreparedStatement preparedDelete = MySQLConfig.connection.prepareStatement(deleteSQL);
preparedDelete.setString(1, name);
preparedDelete.executeUpdate();
MySQLConfig.connection.commit();
```

## [Zookeeper](https://github.com/gino2010/javatech/tree/master/distributed_lock_zk)
不重复造轮子了，轮子原理简单说一下吧，两种实现方式：
1. 固定节点，利用文件名称唯一性，谁可以创建create指令该节点，谁就获得锁，类似redis中的获取锁操作，删除节点释放锁，其它watch该节点
1. 利用zookeeper中临时有序节点，尝试获得锁，如果当前线程申请到的节点序号为最小，则获得锁，否则监听前一个序号等待状态改变，然后再次判断。

建议大家，如果有轮子就不要自己造了，因为绝大多数情况下，我们造的不如已有的轮子。
所以，实验使用成熟的CuratorFramework，创建InterProcessMutex锁对象

通过acquire尝试获得锁，其中3s为可以接受的尝试获得锁的时间，如果太短，可能资源竞争激烈导致获得不到锁，而返回false，所以考虑了重试机制。
```java
mutex.acquire(3, TimeUnit.SECONDS);
```
释放锁，事务处理完毕后，需要释放锁，最好判断一下，是不是当前线程拥有这个锁，然后尝试释放。
```java
if (mutex.isOwnedByCurrentThread()) {
    mutex.release();
    log.info("release lock....");
}
```

如果按照网上原理介绍来看，其判断是自增序号的最小值为获得锁，那么其应该是是一个公平锁，这一点我需要再进一步验证一下。

## [Redis](https://github.com/gino2010/javatech/tree/master/distributed_lock_redis)
Redis分布式锁，redis官网有相应的[文章](https://redis.io/topics/distlock)阐述，这里相当于模仿着造了一个轮子

首先实现一个redis实例即单节点的例子，这个是基础。通过多线程方式模拟测试，实际应用不应该是多线程环境，否则使用线程同步相关技术就好了。
其次，初步实现了RedLock方式，用于多个Redis节点，增强其稳定性。但实际使用可能还是单节点比较多，需要考虑锁失效的弥补方式。

这里的set方法是关键，其保证了判断是否存在以及设置锁值和有效时间一系列操作的原子性，否则这个分布式锁的实现是不成立的。
```java
String res = resource.set(lockKey, uuid, "NX", "PX", keyExpire);
if ("OK".equals(res)) {
    resource.close();
    log.info("Get lock, uuid: {}", uuid);
    return uuid;
}
```
设置成功，则获得锁，否则失败。其中设置的值为uuid随机数，所以这个理论上是非公平锁

释放锁，即删除值，这是需要watch这个值，并在事务内将其删除
```java
resource.watch(lockKey);
if (uuid.equals(resource.get(lockKey))) {
    Transaction multi = resource.multi();
    Response<Long> del = multi.del(lockKey);
    multi.exec();
    if (del.get() == 1) {
        log.info("Release lock, uuid: {}", uuid);
        retFlag = true;
    } else {
        log.info("Release lock, failed!");
    }
    multi.close();
}
```

当redis有多个节点时，我们需要奇数个点来按照多数原则判断是否获得了锁。程序中通过一个redis中的不通DB来模拟的。
有效时间内，节点数大于一半，就认为获得成功，否则失败
```java
if ((passNode.get() >= nodes / 2 + 1) && (end - start) < keyExpire) {
    log.info("Get RedLock uuid {}, pass node:{}", uuid, passNode.get());
    return uuid;
} else {
    log.info("Get RedLock failed");
    releaseLock(lockName, uuid);
    return null;
}
```

## 总结
无论什么怎样实现分布式锁，都需要考虑一下内容：

- 判断锁是否存在和设置锁值，两步需要原子性
- 考虑申请锁时，申请的超时时间和尝试次数，具备非阻塞特性
- 需要考虑锁的正常销毁方式，主动删除锁
- 需要考虑锁的异常销毁方式，例如有效时间或者回话断开删除
- 适当考虑重入特性

使用总结：
- mysql 利用数据库特性还是很方便的，性能有限
- zookeeper实现应该是非常严谨的，但是性能一般
- Redis虽然实现上还存在争议，但是性能很好

性能: 缓存 > Zookeeper > 数据库
可靠性: Zookeeper > 缓存 > 数据库

实现复杂度，看你对谁熟悉了，我个人感觉得差不多。