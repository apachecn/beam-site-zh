---
layout: default
title: "Beam 概述"
permalink: /get-started/beam-overview/
redirect_from:
  - /use/beam-overview/
  - /docs/use/beam-overview/
---

# Apache Beam 概述

Apache Beam 是一个用来定义批处理和流数据并行处理管道的开源统一模型。使用开源的Beam SDK，你就可以构建一个定义管道的程序，这个管道由Beam支持的分布式处理后端执行，其中包括[Apache Apex](http://apex.apache.org), [Apache Flink](http://flink.apache.org), [Apache Spark](http://spark.apache.org), and [Google Cloud Dataflow](https://cloud.google.com/dataflow)。

Beam对高度并行(http://en.wikipedia.org/wiki/Embarassingly_parallel)数据处理任务特别有用，做法是将这些数据分解成许多可以独立且并行处理的小数据包。还可以使用Beam做ETL任务和纯数据集成。这些任务对于在不同的存储介质和数据源之间移动数据，将数据转换为更理想的格式，或者将数据加载到新的系统上是有用的。

## Apache Beam SDKs
 
Beam提供统一的编程模型，它可以表示和转换任何大小的数据集，无论输入是来自批量数据的有限数据集，还是来自流式数据源的无限数据集。Beam SDKs使用相同的类来表示有界数据和无界数据，以及对该数据进行相同的转换。您可以使用您选择的Beam SDK构建一个定义数据处理流程的程序。

Beam 目前支持以下指定语言的SDKs:

* Java <img src="{{ site.baseurl }}/images/logos/sdks/java.png"
         alt="Java SDK">
* Python <img src="{{ site.baseurl }}/images/logos/sdks/python.png"
         alt="Python SDK ">

## Apache Beam Pipeline Runners

The Beam Pipeline Runners把你用Beam程序定义的数据处理管道翻译成你选择的分部署处理后端兼容的API。您运行您的Beam程序时，您需要为执行管道的后端指定一个[合适的Runner]({{ site.baseurl }}/documentation/runners/capability-matrix)。

Beam 目前支持Runners可以在下面这些分布式处理后端下工作：

* Apache Apex <img src="{{ site.baseurl }}/images/logos/runners/apex.png"
         alt="Apache Apex">
* Apache Flink <img src="{{ site.baseurl }}/images/logos/runners/flink.png"
         alt="Apache Flink">
* Apache Gearpump (incubating) <img src="{{ site.baseurl }}/images/logos/runners/gearpump.png"
         alt="Apache Gearpump">
* Apache Spark <img src="{{ site.baseurl }}/images/logos/runners/spark.png"
         alt="Apache Spark">
* Google Cloud Dataflow <img src="{{ site.baseurl }}/images/logos/runners/dataflow.png"
         alt="Google Cloud Dataflow">
    
**注意:** 您始终可以在本地执行您的pipeline用来测试和调试。

## 开始

开始使用Beam做你的数据处理任务.

1. 参照快速入门 [Java SDK]({{ site.baseurl }}/get-started/quickstart-java) 或者 [Python SDK]({{ site.baseurl }}/get-started/quickstart-py).

2. 理解例子[WordCount Examples Walkthrough]({{ site.baseurl }}/get-started/wordcount-example) 中介绍的SDKs的各种功能。

3. 深入[Documentation]({{ site.baseurl }}/documentation/) 中快速翻找深入的概念和Beam模型，SDKs，Runners的参考资料。

## 贡献

Beam 是一个基于[Apache Software Foundation](http://www.apache.org) 的项目，获得了 Apache v2 license。 Beam 是一个开源社区，贡献非常值得赞赏！如果您想贡献，请参阅[贡献]({{ site.baseurl }}/contribute/)部分。
