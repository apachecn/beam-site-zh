---
layout: default
title: "Apache Beam Python SDK"
permalink: /documentation/sdks/python/
---
# Apache Beam Python SDK

Apache Beam 的 Python SDK 提供了一个简单而强大的 API, 用于在 Python 中构建批数据处理的 pipelines.

## Python SDK 入门指南

转到 [Beam 编程指南]({{ site.baseurl }}/documentation/programming-guide) 来学习 Beam SDK 中出现的基础概念.
接下来, 遵循[Beam Python SDK 快速入门]({{ site.baseurl }}/get-started/quickstart-py) 来设置你的 Python 开发环境, 获取 Python 的 Beam SDK, 并且运行一个 pipeline 的例子.

请参阅 [Python API 参考]({{ site.baseurl }}/documentation/sdks/pydoc/) 来获得相关 API 的更多信息.

## Python 类型安全

Python 是一种动态类型的语言, 没有静态类型检查.
他的 Beam SDK 使用代码编辑提示来保证程序的 pipeline 设计和运行时尝试仿真正确性来完成静态类型.
[Python 类型安全的保证]({{ site.baseurl }}/documentation/sdks/python-type-safety) 是通过使用键入提示, 能帮助你在 [直接运行]({{ site.baseurl }}/documentation/runners/direct/) 之前捕获潜在的 BUG.

## 管理 Python Pipeline 依赖

当你运行你本地的 pipeline 时, pipeline 附带的包可以使用是由于他们在你本地的机器上安装.
然而，当你想远程运行你的 pipeline 时，你必须确保远程机器上面同样有附带的包.
[管理 Python Pipeline 附件]({{ site.baseurl }}/documentation/sdks/python-pipeline-dependencies) 将展示您如何在远程节点上确保你的依赖包.

## 创建新的 Sources 和 Sinks
Python 的 Beam SDK 提供了一个可扩展的 API 来用于创建新的 data sources（数据源）和 sinks.
[由 Python SDK 创建新的 data sources 和 sinks]({{ site.baseurl }}/documentation/sdks/python-custom-io) 展示了如何使用 [Beam 的 Source 和 Sink API](https://github.com/apache/beam/blob/master/sdks/python/apache_beam/io/iobase.py) 来创建新的 sources 和 sinks.

