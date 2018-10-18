---
title: Zookeeper
date: 2018-10-18 09:05:27
tags: zookeeper
categories: tech
---

近期在做MQ的实验，所以又接触了一下[Zookeeper](https://zookeeper.apache.org/)，之所说又是因为之前碰到过，但是只是用而已没有了解过一些细节。

## Zookeeper 做什么？

引用官方原话：
ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services. 

他可以提供高可靠的分布式协调服务，包括中心化配置服务等。
之前我用它做了什么，注册发现服务器，你是不是也是用了这个功能，但是具体怎么实现的，Spring Cloud都已经实现了，没有深入关心过。

这个文章我们补一些必要的知识，更加深入的了解Zookeeper。

<!-- more -->

## Zookeeper 怎么玩？

首先，简单说他好似一个文件目录，是一个可以存储数据的服务器，这里我们先忽略其集群中的一些特性（选举制度等），我们看它对外可以提供什么。
- 它内部有类似文件目录的结构，节点称之为znode
- 文件结构可以存储数据，可以想象为目录结构的Redis
- 节点数据大小有限，不可以超过1M
- 可以通过接口获得数据，这也是我们用其分布式协调服务的关键

补充，数据过大怎么办，目前了解到的有两个思路，<font color=red>没有实验过</font>：
- 使用专业的分布式存储HDFS等
- 使用Redis，然后zookeeper记录索引就好了

### 命令行
两种形式，分别对应查看管理 服务器状态和内部存储。

查看管理服务器状态，可以通过telnet 和 nc发出指令，一般使用nc比较方便（具体命令参考网上信息）
```
echo stat| nc 127.0.0.1 2181 # 来查看哪个节点被选择作为follower或者leader
echo ruok| nc 127.0.0.1 2181 # 测试是否启动了该Server，若回复imok表示已经启动。
echo dump| nc 127.0.0.1 2181 # 列出未经处理的会话和临时节点。
echo kill | nc 127.0.0.1 2181 # 关掉server
echo conf | nc 127.0.0.1 2181 # 输出相关服务配置的详细信息。
echo cons | nc 127.0.0.1 2181 # 列出所有连接到服务器的客户端的完全的连接 / 会话的详细信息。
echo envi | nc 127.0.0.1 2181 # 输出关于服务环境的详细信息（区别于 conf 命令）。
echo reqs | nc 127.0.0.1 2181 # 列出未经处理的请求。
echo wchs | nc 127.0.0.1 2181 # 列出服务器 watch 的详细信息。
echo wchc | nc 127.0.0.1 2181 # 通过 session 列出服务器 watch 的详细信息，它的输出是一个与 watch 相关的会话的列表。
echo wchp | nc 127.0.0.1 2181 # 通过路径列出服务器 watch 的详细信息。它输出一个与 session 相关的路径。
```

如果提示：xxx is not executed because it is not in the whitelist.
请在conf/zoo.cfg中添加：
```
4lw.commands.whitelist=* #或者具体指令stat, ruok
```

查看管理服务中的节点信息，需要zkCli命令（客户端），这个在zookeeper所对应的bin目录下有，zkCli进入后，随便敲点啥，你可以看到如下：
```
[zk: localhost:2181(CONNECTED) 0] ?
ZooKeeper -server host:port cmd args
	addauth scheme auth
	close
	config [-c] [-w] [-s]
	connect host:port
	create [-s] [-e] [-c] [-t ttl] path [data] [acl]
	delete [-v version] path
	deleteall path
	delquota [-n|-b] path
	get [-s] [-w] path
	getAcl [-s] path
	history
	listquota path
	ls [-s] [-w] [-R] path
	ls2 path [watch]
	printwatches on|off
	quit
	reconfig [-s] [-v version] [[-file path] | [-members serverID=host:port1:port2;port3[,...]*]] | [-add serverId=host:port1:port2;port3[,...]]* [-remove serverId[,...]*]
	redo cmdno
	removewatches path [-c|-d|-a] [-l]
	rmr path
	set [-s] [-v version] path data
	setAcl [-s] [-v version] path acl
	setquota -n|-b val path
	stat [-w] path
	sync path
Command not found: Command not found ?
[zk: localhost:2181(CONNECTED) 1]
```
我主要使用了：
1. ls 显示znode
1. crate 创建znode，并设置初始内容
1. get 获取znode内容
1. set 修改znode内容
1. delete 删除znode
1. 退出客户端： quit

## 实验工程
[测试工程地址](https://github.com/gino2010/javatech/tree/master/zookeeper) ，开发环境：

- mac os x & idea & gradle
- spring cloud & zookeeper config & discovery
- zookeeper 3.5 in docker

请注意，其中使用的最新版的curator包只支持zookeeper3.5

### 关键代码
通过CuratorFramework 设置或读取zookeeper中的节点
```java
curatorFramework.blockUntilConnected();
Stat stat = curatorFramework.checkExists().forPath("/test");
if(stat==null) {
    curatorFramework.create().creatingParentsIfNeeded()
            .withACL(ZooDefs.Ids.OPEN_ACL_UNSAFE)
            .forPath("/test", "data".getBytes());
}
byte[] bytes = curatorFramework.getData().forPath("/test");
log.info("path test:{}", new String(bytes));
```

将zookeeper作为配置中心，通过value注解获取zookeeper中的配置信息，这样可以做到在不重新启动服务的情况下，动态加载配置信息
代码需要：

```java 
@Value("${zc}")
private String zc;
```


配置文件需要：

```javascript
application:
  name: zookeeperDemo
  
config:
  enabled: true
  root: /demo
  defaultContext: context
  profileSeparator: ','
```


因为spring cloud支持active profile，所以对应的zookeeper的znode路径可以是：
- /demo/zookeeperDemo,default
- /demo/zookeeperDemo
- /demo/context,default
- /demo/context

具体到zc这个配置，对应在default下的全路径是：/demo/zookeeperDemo::default/zc，你可以在zookeeper中创建一个：

```
create /demo/zookeeperDemo,default/zc data
```

以上为工程实验的关键点，具体工程参考源码，工程后续可能还会更新。

## 吐槽
这些问题不是针对zookeeper，只是借此表达一下，其它国内实例代码也有类似的普遍现象

- Curator的使用，国内几乎没有，说明大家几乎没有尝试自己去使用zookeeper的存储功能，都是人云亦云般的介绍功能
- 查看中心化配置，路径的写法几乎都没有验证，实例代码几乎都不起作用，你们贴别人代码时不实验一下吗？

请大家（同行）做到，我写的，我贴的，我抄的代码，我至少都实验过，不要传播错误的代码。
共勉！