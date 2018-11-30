---
layout: default
title: "学习 Beam"
permalink: /documentation/
redirect_from:
  - /learn/
  - /docs/learn/
---

# Apache Beam 文档

本章提供了深入的概念信息，并针对 Beam 模型, SDKs 与 Runners 也提供了参考资料:

## 概念

学习 Beam 编程模型与所有 Beam SDKs 和 Runners 的共同概念.

* 该 [编程指南]({{ site.baseurl }}/documentation/programming-guide/) 介绍了所有 Beam 中较为关键的概念.
* 学习 Beam 的 [执行模型]({{ site.baseurl }}/documentation/execution-model/) 以更好的理解 pipelines（管道）是如何执行的.
* 访问 [附加资源]({{ site.baseurl }}/documentation/resources/) 了解一些我们最喜欢的文章和关于 Beam 的话题.

## Pipeline 基础原理

* [设计 Pipeline]({{ site.baseurl }}/documentation/pipelines/design-your-pipeline/). 通过规划 pipeline 的结构, 选择 transforms（转换）以应用到数据上, 并确定 input（输入）和 output（输出）方法来设计.
* [创建 Pipeline]({{ site.baseurl }}/documentation/pipelines/create-your-pipeline/). 使用 Beam SDKs 中的类来创建.
* [测试 Pipeline]({{ site.baseurl }}/documentation/pipelines/test-your-pipeline/). 最小化调试 pipeline 的远程执行，然后查看反馈情况.

## SDKs

在所有的 Beam SDK 上查找有用的状态和参考信息.

* [Java SDK]({{ site.baseurl }}/documentation/sdks/java/)
* [Python SDK]({{ site.baseurl }}/documentation/sdks/python/)

## Runners

在一个特定的（通常是分布式的）数据处理系统上一个 Beam Runner 运行一个 Beam pipeline.

### 可用的 Runners

* [DirectRunner]({{ site.baseurl }}/documentation/runners/direct/): 在您的机器上本地运行 -- 非常适合开发, 测试与调试.
* [ApexRunner]({{ site.baseurl }}/documentation/runners/apex/): 运行在 [Apache Apex](http://apex.apache.org) 上.
* [FlinkRunner]({{ site.baseurl }}/documentation/runners/flink/): 运行在 [Apache Flink](http://flink.apache.org) 上.
* [SparkRunner]({{ site.baseurl }}/documentation/runners/spark/): 运行在 [Apache Spark](http://spark.apache.org) 上.
* [DataflowRunner]({{ site.baseurl }}/documentation/runners/dataflow/): 运行在 [Google Cloud Dataflow](https://cloud.google.com/dataflow) 上, 一个在 [Google Cloud Platform](https://cloud.google.com/) 中完全.
* [GearpumpRunner]({{ site.baseurl }}/documentation/runners/gearpump/): 运行在 [Apache Gearpump (incubating)](http://gearpump.apache.org) 上.

### 选择 Runner

Beam 设计的目的是让 pipeline 能够在不同的 runner 上都可以移动.
然而, 由于每个 runner 拥有不同的功能, 它们也有不同功能以实现 Beam 模型中的核心概念.
该 [兼容性矩阵]({{ site.baseurl }}/documentation/runners/capability-matrix/) 提供了 runner 功能上更详细的比较.

一旦您选择了要使用哪一个 runner, 请看 runner 的页面以获得有关任何初始化指定 runner 设置的更多信息, 以及任何需要或可选的 `PipelineOptions` 用于它的执行.
您也许也想要去参阅针对 [Java]({{ site.baseurl }}/get-started/quickstart-java) 或 [Python]({{ site.baseurl }}/get-started/quickstart-py) 的快速入门, 以了解有关执行 WordCount（示例）pipeline 的说明.