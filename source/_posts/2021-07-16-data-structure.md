---
title: Data Structure
date: 2021-07-16 21:06:09
tags: Java
categories: tech
---

书接上回，今天说Java中的数据结构，基于Java 11，相关内容点到为止，可能不会太深入。
知道以及够用目的来总结数据结构

将会涉及 Array List Vector Stack Dictionary Set Map Queue Deque

<!-- more -->

## Array

数组最简单的数据结构, 特点可以多维，可object也可基础类型，大小不可变，当然读取和写入速度快。
个人感觉应用场景偏底层，一般不会用于复杂业务数据的处理。与Python中的Array相比，我更喜欢Python（后面开始总结Python，顺手挖坑）
使用很简单，不多说
```java
int[][] array4 = {{1,2,3}, {2,3,4}};
```

## List
相比Array，List大家用的应该更多，最多莫过于ArrayList了吧
表，里面的每一个item就像是一行，整个数据就是一个表
List这个Interface的实现有很多，我们最常用的就是 ArrayList，别问我别的，我几乎没有用过其他List实现

出于好奇查了一下ArrayList和LinkedList，网上有很多比较文章，例如：[stackoverflow](https://stackoverflow.com/questions/322715/when-to-use-linkedlist-over-arraylist-in-java)
如果谁在实际工作中使用了LinkedList请告诉我应用场景和理由，感谢先。
多数情况下，简单一句话，用ArrayList 就对了。

CopyOnWriteArrayList 是Arraylist 线程安全的变种，没有用过，作为了解看看，如果以后遇到多线程，应该会优先考虑使用它。
当然，我们还可以用Collections.synchronizedList方法包裹整个对象，实现线程安全，此方式弊端也显而易见，他将锁定整个对象，性能不高。

## Vector & Stack
List的近亲，Vector继承了List接口，Stack继承了Vector。元老级对象结构，个人不建议使用。
两个都是线程安全，并通过synchronized关键字进行同步，效率不高

Vector和ArrayList使用上区别不大，Stack主要是实现LIFO
值得注意的是，都是List的实现，所以我们可以这样
```java
List  vector = new Vector();  
List  stack = new Stack();
```

记得当年第一份Java工作还在使用Vector，后来就不在用了（暴露了年龄，呵呵）

## Dictionary
作为抽象类，它的应用场景应该已经少之又少了吧，感觉可以忽略它了。
与之相关的提一句HashTable吧，线程安全切使用synchronized关键词，方法同步。
HashTable也继承了Map接口，有了ConcurrentHashMap，谁还用HashTable？

## Map
这个是个重要的接口，常用实现类超多，键值对的数据结构到处都是。
先说一个最常用的HashMap，无人不知无人不晓，看看他的源码实现应该会收益匪浅
HashMap不是线程安全的，如果再多线程中进行操作，可能会丢失数据哦，为什么，看看HashMap的实现原理你会找到答案的 [stackoverflow](https://stackoverflow.com/questions/18542037/how-to-prove-that-hashmap-in-java-is-not-thread-safe)

如何线程安全，两个方法
ConcurrentHashMap, 大多数人应该首先想到的方法，也是最好的方法
Collections.synchronizedMap(), 还要有这个隐藏方式，不常用，当然是因为性能不佳

还有LinkedHashMap和TreeMap，虽然我基本不用，尤其是TreeMap好像没有用过
三者之间的比较可以看看这个，一目了然。[stackoverflow](https://stackoverflow.com/questions/2889777/difference-between-hashmap-linkedhashmap-and-treemap)

Map还有什么值得注意，*ConcurrentSkipListMap*，这个建议要深入研究一下。

## Set
这个数据结构的最大特点就是数据不可以重复，实现类也超多
```java
Set hashSet = new HashSet();  
Set linkedHashSet = new LinkedHashSet();  
Set treeSet = new TreeSet();
```
根据Map的前车之鉴，这三个类型的对比也就大同小异了。[](https://stackoverflow.com/questions/20116660/hashset-vs-treeset-vs-linkedhashset-on-basis-of-adding-duplicate-value)

为了线程安全，和Map相似，有两个方式
CopyOnWriteArraySet，线程安全
Collections.synchronizedSet(); 或者此方法，不建议使用

当然他还有一个重要的类*ConcurrentSkipListSet*值得研究一下

## Queue & Deque
这两个放在一起说，其实有了他们，我们也就彻底废弃了Vector和Stack
* Queue 是FIFO的数据结构
* Deque 是同时支持 FIFO LIFO的数据结构，直接代替带Stack

如何你的数据有顺序要求,并且顺序消费，应该优先考虑Queue。其性能应该更高一些。
我应该认真对比一下Queue与List，看看他们的具体使用区别

说一说常见的实现类
PriorityQueue，可以比较对象权重来进行数据消费
```java
PriorityQueue<Integer> pQueue = new PriorityQueue<>();

// Adding items to the pQueue using add()
pQueue.add(10);
pQueue.add(20);
pQueue.add(15);

// Printing the top element of PriorityQueue
System.out.println(pQueue.peek());

// Printing the top element and removing it
// from the PriorityQueue container
System.out.println(pQueue.poll());

// Printing the top element again
System.out.println(pQueue.peek());
```

ArrayDeque，感觉可以代替到LinkedList
LinkedBlockingQueue / LinkedBlockingDeque
以及线程安全的数据结构
ConcurrentLinkedQueue / ConcurrentLinkedDeque

Queue的实现也很多，这个需要点时间好好再挖掘一下，后续有时间我再补充代码

## 结束语
很多数据类型我们平时并不会用到，但是在某些场合下，只有它最合适，所以还是需要了解一下以备不时之需。

下一个目标是什么呢？ Java中的时间好啦，应该比较简单

P.S. Python的总结也该开始了（挖坑，督促自己填）