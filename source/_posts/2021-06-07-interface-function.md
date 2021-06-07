---
title: Functional Interface 
date: 2021-06-07 20:45:55
tags: Java
categories: tech
---

最近在重新学习Java Core，看到Functional Interface. 做个小总结。

何为Functional Interface？ @FunctionalInterface 可以看到很多这样的注解，其实注解只是标识而已。
本质是Any interface with a SAM(Single Abstract Method) is a functional interface, 有一个抽象方法定义的接口就是Functional Interface
它常常在Lambda表达式中使用，无需匿名类，直接实现方法定义。

让我们看看常见的Functional Interface，都式如何使用的
<!-- more -->

## 常见接口比较

| 接口  | 方法 |  功能 |
|------------|--------|---------------------|
| Consumer   |accept | 一个入参，无法返回结果 |
| Function   |apply | 一个入参，有返回结果 |
| Predicate  |test | 一个入参，true or false 返回结果 |
| Supplier   |get | 无入参，有返回结果 |
| Comparator | compare | 一个入参，正数，0，负数返回结果 |

## 极简演示

```java
        Consumer<String> consumer = System.out::println;
        consumer.accept("consumer");

        Function<String, String> function = x -> x + " gino";
        System.out.println(function.apply("Hello"));

        Predicate<String> predicate = x -> x.equals("Gino");
        System.out.println(predicate.test("Gino"));

        Supplier<String> supplier = ()->"Supplier";
        System.out.println(supplier.get());

        Comparator<String> comparator = String::compareTo;
        System.out.println(comparator.compare("1", "2"));
```
## 思考题
查看Comparator 源码，为什么 Comparator 有两个抽象方法定义，他却是functional interface呢？
想一想，你会找到答案的。