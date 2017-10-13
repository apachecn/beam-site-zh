---
layout: default
title: "Apache Beam Python SDK"
permalink: /documentation/sdks/python/
---
# Apache Beam Python SDK

The Python SDK for Apache Beam provides a simple, powerful API for building batch data processing pipelines in Python.
Apache Beam 提供了一份简明扼要的构建批量数据处理管道的Python API.

## Get Started with the Python SDK
## 开始您的Python SDK之旅吧

Get started with the [Beam Programming Guide]({{ site.baseurl }}/documentation/programming-guide) to learn the basic concepts that apply to all SDKs in Beam. Then, follow the [Beam Python SDK Quickstart]({{ site.baseurl }}/get-started/quickstart-py) to set up your Python development environment, get the Beam SDK for Python, and run an example pipeline.
转到 [Beam 编程指南]({{ site.baseurl }}/documentation/programming-guide) 来学习Beam SDK中出现的基础概念. 接下来, 遵循[Beam Python SDK 快速开始]({{ site.baseurl }}/get-started/quickstart-py) 来建立你的Python开发环境,达成Python的Beam SDK,运行一个管道的例子吧.

See the [Python API Reference]({{ site.baseurl }}/documentation/sdks/pydoc/) for more information on individual APIs.
查看 [Python API 指导]({{ site.baseurl }}/documentation/sdks/pydoc/) 来获得个人相关的API的更多信息.

## Python Type Safety
## Python类型安全

Python is a dynamically-typed language with no static type checking. The Beam SDK for Python uses type hints during pipeline construction and runtime to try to emulate the correctness guarantees achieved by true static typing. [Ensuring Python Type Safety]({{ site.baseurl }}/documentation/sdks/python-type-safety) walks through how to use type hints, which help you to catch potential bugs up front with the [Direct Runner]({{ site.baseurl }}/documentation/runners/direct/).
Python是一门典型的非静态语言.他的Beam SDK使用代码编辑提示来保证程序管道设计和运行时尝试仿真正确性来完成静态类型. [Python类型安全的保证]({{ site.baseurl }}/documentation/sdks/python-type-safety)是通过使用键入提示，能帮助你在[直接运行]({{ site.baseurl }}/documentation/runners/direct/)之前捕获潜在的BUG.

## Managing Python Pipeline Dependencies
## 管理你的Python管道附件

When you run your pipeline locally, the packages that your pipeline depends on are available because they are installed on your local machine. However, when you want to run your pipeline remotely, you must make sure these dependencies are available on the remote machines. [Managing Python Pipeline Dependencies]({{ site.baseurl }}/documentation/sdks/python-pipeline-dependencies) shows you how to make your dependencies available to the remote workers.
当你运行你本地的管道时，管道附带的包可以使用是由于他们在你本地的机器上安装。然而，当你想远程运行你的管道时，你必须确保远程机器上面同样有附带的包。[管理Python管道附件]({{ site.baseurl }}/documentation/sdks/python-pipeline-dependencies)将展示您如何在远程节点上确保你的依赖包.

## Creating New Sources and Sinks
## 创建新的源码和池

The Beam SDK for Python provides an extensible API that you can use to create new data sources and sinks. [Creating New Sources and Sinks with the Python SDK]({{ site.baseurl }}/documentation/sdks/python-custom-io) shows how to create new sources and sinks using [Beam's Source and Sink API](https://github.com/apache/beam/blob/master/sdks/python/apache_beam/io/iobase.py).
Beam SDK for Pyhton提供了一个可扩展的API来用于创建新的源码和池.[由Python SDK创建新的源码和池]({{ site.baseurl }}/documentation/sdks/python-custom-io) 展示了如何使用 [Beam的源码和池API](https://github.com/apache/beam/blob/master/sdks/python/apache_beam/io/iobase.py)建立新的源码和池.

