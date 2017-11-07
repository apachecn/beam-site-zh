---
layout: default
title: "Create Your Pipeline"
permalink: /documentation/pipelines/create-your-pipeline/
---
# 创建你的 Pipeline

* TOC
{:toc}

你的 Beam程序表达的是一种数据处理的 pipeline，从开始到完成。这一节介绍使用 Beam SDKs 中的类构造一个 pipeline机制。为了使用 Beam SDKs中的类构造 pipeline ，你的程序需要执行下面通用的步骤:


*   创建一个 `Pipeline` 对象.
*   使用 一个 **Read** 或者 **Create** transform 对pipeline 数据创建一个或者多个`PCollection`。 
*   对每个PCollection` 应用 **transforms** . Transforms 可以改变，过滤，分组，分组，或者其他操作来处理 `PCollection`中的元素。每个 transform（转换）创建一个新的输出 `PCollection`，你还可以当处理完成后做额外的 transform（转换）。
*   **Write** 或以其他方式输出最终转换的 `PCollection`s。
*   **运行** the pipeline.

## 创建你的 `Pipeline` 对象

一个 Beam 程序经常从创建一个 `Pipeline` 对象开始。每个 `Pipeline`对象都是一个独立的实体，封装了 pipeline 运行的数据和应用于该数据的 transform（转换）。


创建一个 pipeline, 声明一个 `Pipeline` 对象, 并传递一些 [配置选项]({{ site.baseurl }}/documentation/programming-guide#configuring-pipeline-options).

```java
// Start by defining the options for the pipeline.
PipelineOptions options = PipelineOptionsFactory.create();

// Then create the pipeline.
Pipeline p = Pipeline.create(options);
```

## 从你的 Pipeline 读取数据

为了创建 pipeline 的初始`PCollection`，你对你的 pipeline 对象应用一个 root transform。 root transform 从外部数据源或指定的某些本地数据创建 `PCollection`。  

在 Beam SDKs 中有两类transforms: `Read` 和 `Create`。`Read` 转换从外部源读取数据，例如文件文件或者数据库表。`Create` transforms 使用存储在内存里的 `java.util.Collection` 创建一个 `PCollection`。

下面例子的代码展示了应用一个 `TextIO.Read` root transform，从文本文件中读取数据。transform 应用到 `Pipeline`对象`p`，并将 pipeline 数据集以 `PCollection<String>`格式返回:


```java
PCollection<String> lines = p.apply(
  "ReadLines", TextIO.read().from("gs://some/inputData.txt"));
```

## 运用 Transforms 处理 Pipeline 数据

你可以使用Beam SDKs提供的许多[transforms]({{ site.baseurl }}/documentation/programming-guide/#transforms) 封装你的数据.你可以对每个`PCollection` 调用 `apply` 方法，对 pipeline's 的  `PCollection` 做 transforms ， `PCollection`就是你想要处理和 transform 所需要的参数。

下面的代码展示了如何对一个 `PCollection` 字符串  `apply` 一个transform。transform 是一个用户定义的自定义变换，可以反转每个字符串的内容，并输出一个包含反转字符串的新的 `PCollection`。

输入是一个叫做 `words` 的`PCollection<String>`;这个代码将一个名为`ReverseWords`的 `PTransform` 实例传递给  `apply`方法，并保存返回的 `PCollection<String>`类型的 值为  `reversedWords`。


```java
PCollection<String> words = ...;

PCollection<String> reversedWords = words.apply(new ReverseWords());
```

## Writing or Outputting Your Final Pipeline Data

一旦你的 pipeline 已经运用过了所有的 transforms，你经常需要输出结果。为了输出你的 pipeline 最后的`PCollection`s,你会对 `PCollection` 做  `Write` transform。 `Write` transform 可以输出`PCollection` 的元素到一个外部数据 sink，例如数据库表。你可以在你的 pipeline任何时候 使用 `Write`操作输出 `PCollection`，尽管你一般是在 pipeline 结束的时候写出你的数据。

下面的示例代码展示了如何 `apply` 一个 `TextIO.Write` transform，将一个`String` 类型的  `PCollection`写入到一个文本文件。


```java
PCollection<String> filteredWords = ...;

filteredWords.apply("WriteMyFile", TextIO.write().to("gs://some/outputData.txt"));
```

## 运行你的 Pipeline

一旦你构造好你的 pipeline,使用  `run` 方法运行 pipeline（管道）。Pipelines是异步执行的: 你创建的程序会将你的 pipeline 发送给 **pipeline runner**，然后构建并运行一系列的管道操作。


```java
p.run();
```

The `run` method is asynchronous. If you'd like a blocking execution instead, run your pipeline appending the `waitUntilFinish` method:

```java
p.run().waitUntilFinish();
```

## 下一步

*   [编程指南]({{ site.baseurl }}/documentation/programming-guide) - Learn the details of creating your pipeline, configuring pipeline options, and applying transforms.
*   [测试你的 pipeline]({{ site.baseurl }}/documentation/pipelines/test-your-pipeline).
