---
layout: default
title: "Beam Java SDK"
permalink: /documentation/sdks/java/
redirect_from: /learn/sdks/java/
---
# Apache Beam Java SDK

Apache Beam 的 Java SDK 提供了一个简单而强大的 API，用于在 Java 中构建批处理和流式的并行数据处理管道。

## Java SDK 入门指南

入门只能可以阅读 [Beam 编程模型]({{ site.baseurl }}/documentation/programming-guide/) 以学习 Beam 中应用到所有 SDK 上的基础概念。

更多单个 API 的信息请参阅 [Java API 参考文档]({{ site.baseurl }}/documentation/sdks/javadoc/)。

## 支持的功能

Java SDK 支持 Beam 模型当前所支持的所有功能。

## Pipeline I/O
请参阅 [Beam-provided I/O Transforms]({{site.baseurl }}/documentation/io/built-in/) 页面以获取当前可用的 I/O transforms 列表.

## Extensions

Java SDK 有以下扩展:

- [join-library]({{site.baseurl}}/documentation/sdks/java-extensions/#join-library) 提供了 inner join, outer left join 和 outer right join 功能.
- [sorter]({{site.baseurl}}/documentation/sdks/java-extensions/#sorter) 是一个高效的，可伸缩的大型迭代器.
- [Nexmark]({{site.baseurl}}/documentation/sdks/nexmark) 是一个以批处理和流式模式运行的 benchmark 套件.
