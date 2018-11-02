---
title: Java 11 License
date: 2018-10-31 14:24:48
tags: Java
categories: tech
---

晴天我的霹雳～～～～

今天在MacOS下玩Kubernets，无意间发现Java 11的资源包变成的OpenJDK，去找Oracle版本时看到这个说明：

[Oracle's Java 11 trap - Use OpenJDK instead!](https://blog.joda.org/2018/09/do-not-fall-into-oracles-java-11-trap.html)

文章重点就是，Java 11的License是商业授权，用于商业活动时，你是要付费的，不是免费的。

Oracle License原文可以参见这里：[Oracle Technology Network License Agreement for Oracle Java SE](https://www.oracle.com/technetwork/java/javase/terms/license/javase-license.html)

Further, You may not:
- use the Programs for any data processing or any commercial, production, or internal business purposes other than developing, testing, prototyping, and demonstrating your Application;

就是这句话。 根据上文博主的说法，之前的license肯定都是免费的（我之前没有特别留意过），现在商业用途竟然需要收费了。

<!-- more -->

现在Google搜索引擎中搜索Java 11，可以看到Oracle 维护的open jdk 版本了。是的，它在维护openjdk版本
地址在这里：https://jdk.java.net/11/

当然文中博主还给出了其它的编译版本，https://adoptopenjdk.net

使用Java的小伙伴们，升级到Java11那是必然的，毕竟Java8支持终止时间已经明确了，只有Java11是最近的一个LTS版本
然而，如果你还想免费用的话，请使用openjdk，别到头来，你开发了商业软件，最后要支付一大笔钱给Oracle
钱多的土豪和拿Java写着玩（非商业软件）的人士可以忽略。

顺便说一句：作为国内的程序员，希望大家重视商业授权方式，开源 不等于 免费，一个是代码维护方式（开源协议也很多，请自行查找），一个是商业授权方式。
因为，如果你想走的更远，你需要在规则体系之内，良好的意识也需要慢慢培养。共勉～