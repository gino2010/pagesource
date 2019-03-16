---
title: All You Need To Know For Migrating To Java 11
date: 2019-01-06 16:39:37
tags: Java
categories: tech
---

## 前言
首先声明，此文内容大多为译文，并非原创
原文地址：[All You Need To Know For Migrating To Java 11](https://blog.codefx.org/java/java-11-migration-guide/)
并非逐句对应翻译，只对关键内容进行翻译，如有问题请指正，感谢！

翻译此文章的初衷源于我刚刚第一次搭建了基于Java 11的spring boot工程，遇到了一些包的缺失导致的问题，当然这些问题在Java 8时是不会出现的
另外，Java 8的生命周期已经进入尾声，开发者必然面临切换到Java 11的局面，所以在搜索相关信息看到了这个文章。我将尝试翻译此文章，也让自己有更为深入的理解

## 正文
Java 11 早在2018年9月25日发布（原文在当日发表）它标志着Java生态系统重大转变的结束（Java 9 & 10都是这个过程的中间态）Java 11是LTS（Long Term Support）版本，伴随着从Java 8迁移到模块化灵活的JDK和6个月release迭代周期
在作者看来，我们进入了新纪元，我们需要遵循新的Java规则，很多项目需要考虑从Java 8 迁移到Java 11。此文章将告诉你所有你需要知道的关于从Java 8迁移到Java 11
<!-- more -->

此外列出Java 11的新特性链接：
- [New HTTP2 Client](https://blog.codefx.org/java/http-2-api-tutorial/)
- [Reactive](https://blog.codefx.org/java/reactive-http-2-requests-responses/)
- [Scripting](https://blog.codefx.org/java/scripting-java-shebang/)
- [Eleven Hidden Gems ](https://blog.codefx.org/java/java-11-gems/)

请注意，我们将在这里讨论迁移，并不是模块化（modularization），Java 11并不要求一定要模块化，那是一个独立步骤，所以这里没有创建模块

我们先从发布周期，授权方式和支持说起，最后讲解如何克服常见的4个阻碍，做过Java 9 迁移的人可能已经熟悉其中大部分内容。

## 发布和授权
发布周期貌似没有什么，但是这是Oracle JDK商业内容的组成部分，并且相比技术迁移的挑战，公开问题（Open Question）的长期支持可能将对你的项目带来更多的影响。

### 发布周期频率
- 每六个月，发布一个新的主版本，即每年的三月和九月
- 每个主版本将有两个小版本更新，在每个主版本发布后的一或四个月后

（相比之前Java语言迭代发展缓慢，现在可以看出Java进入了快速发展的时代，展示其顽强的生命力，可以感觉它将继续保持语言霸主的地位）

### OpenJDK是新的默认版本
（这个我之前的一个文章有所简单涉及，这次说更清楚些）2018年9月之前，Oracle JDK是大家使用的版本，我们一般会认为其功能更为丰富，更加稳定和高效，虽然事实并不一定（OpenJDK其实也表现的不错）。所以大多数人都是选在Oracle JDK，OpenJDK很少使用，但这一情况将在Java 11将改变。
Oracle努力将Oracle JDK 11 和 OpenJDK 11 在技术上一样（OpenJDK和Oracle JDK在历史上有过不少的差异，某些时候，其实两个社区是在各自发展，虽然这段历史我可能说不清楚，但之前遇到过版本差异带来的问题），所以现在两个版本的差异只有授权文件了。Oracle期望通过将其JDK商业化方式，推动开发者进入OpenJDK。你可以使用Oracle JDK进行开发和测试，但如果没有授权文件是不可以用于产品中的。
结果就是，OpenJDK将成为默认值，同时拥有全部的特性集合、主要性能以及免费的license（GPL+CE）以后可能会出现，Oracle和OpenJDK自定义厂商，都在销售他们长期支持。

### 长期支持 Long-Term Support
首先说 [Oracle JDK](https://jdk.java.net) 和 [OpenJDK](https://openjdk.java.net)，6个月内会更新两个小版本，六个月后呢？如果你还想使用Java 11呢？（下一个长期支持版本是Java 17，Oracle计划每三年释放一个新的LTS版本）怎么获得安全或修复补丁呢？两个选择：
- 向某人（Oracle或其他供应商）购买商业支持
- 期待OpenJDK有免费的支持

关于商业支持，目前可以想到的提供LTS服务的供应商有：
- Oracle
- IBM
- RedHat
- Azul

（这样看，oracle不是为了自己收费哦，而是重新定义了Java世界的商业规则，你可能觉得不爽，但是这些vendor都乐在其中呢，这样来看不是一件坏事，良好的商业模式是支持Java发展的动力）

那么OpenJDK呢？
根据社区讨论的邮件来看，建议对应相同的版本，将有4年左右的长期公开升级支持(Oracle Java 11 主支持要到2023年，扩展支持可到2026年，这样看，OpenJDK社区支持可以到2022年，也还不错，现在是否定下来这样执行还需进一步考证)
最有可能，每一个LTS版本将有一个专属的升级负责人，例如Java 11可能由Red Hat负责。这些都是针对源码，那么我们哪里可以获得二进制文件（Binaries）呢？[AdoptOpenJDK](https://adoptopenjdk.net/)将持续生成不同平台的OpenJDK版本
总结，我们可以获得免费的OpenJDK，由Java社区知名企业共同维护，并通过AdoptOpenJDK持续更新，这非常棒！

### 关于Amazon Corretto
财大气粗的Amazon宣布进入OpenJDK供应商行列，并发布了[Corretto](https://amazonaws-china.com/corretto/)，并提供的长期免费的服务（目前，它只放出了Java 8，Java 11预计4月放出），关键信息如下：
- 基于OpenJDK，添加亚马逊实现的安全、性能、稳定、bug等修复
- 支持Linux Mac OS Windows
- 免费使用，GPL+CE授权（这就是个"搅局者"说好的供应商可以自己定制，然后买服务的你怎么不按套路出牌呢？答：有钱任性）
- 长期支持Java 8 到2023
- 长期支持Java 11 到2024
- 季度更新和不定期的紧急更新

参考其对OpenJDK的贡献，Amazon从2017年开始为OpenJDK贡献，所以其很可能将其Corretto的更新到OpenJDK中，这样大家就可以都使用了（好人做到底啦，贡献出来吧）。如果没有，自己看[Corretto的源码](https://github.com/corretto)，自己搞合并吧。
简单说，Amazon会合并Oracle的补丁到其Corretto分支上，即使他们没有这么做，我们自己做也不难。这样使得免费的长期支持，社区驱动OpenJDK11更可行，非常酷～

## 准备你的迁移
（简化翻译以下内容）
好了，我们准备迁移

### 长痛还是短痛？
建议短痛：
- 不要建立长期合并分支，直接在开发分支尝试Java 11，避免不要的合并冲突
- 配置的你的持续集成服务器，普通编译一次，Java 11编译一次
- 学习配置编译工具特殊配置，添加新Java版本支持
- 版本特殊化配置尽可能的小，更新依赖好于在命令行中添加参数

### 更新所有
开发工具建议更新：
- IntelliJ IDEA：2018.2以上
- Eclipse：Photon 4.9RC2 with Java 11 plugin以上
- Maven：3.5.0以上
- compiler plugin: 3.8.0
- surefire and failsafe: 2.22.0
- gradle 4.10.2以上

所有操作字节码的包需要更行，因为Java版本变动，字节码的Level会每6个月更新一次
- ASM 
- Byte Buddy 
- cglib 
- Javassist
- Spring
- Hibernate
- Mockito

注意此方面的错误，例如升级Hibernate：
```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <!-- LOOK OUT: YOU SHOULD USE A NEWER VERSION! -->
    <version>5.2.12.Final</version>
</dependency>
<dependency>
    <!-- update Hibernate dependency on
            Javassist from 3.20.0 to 3.23.1
            for Java 11 compatibility -->
    <groupId>org.javassist</groupId>
    <artifactId>javassist</artifactId>
    <version>3.23.1-GA</version>
</dependency>
```
如果不行，那就要深入问题，查找其他原因

### 关于模块系统
就一个原则，Java 9以后，你并不需要模块化，所以该怎么组织package就怎么组织

## 开始迁移
准备就绪，看看Java 11都会遇到什么，然后怎么解决。

未完待续......