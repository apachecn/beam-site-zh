---
layout: default
title: "Design Your Pipeline"
permalink: /documentation/pipelines/design-your-pipeline/
---
# 设计你的Pipeline

* TOC
{:toc}

这一页内容有助于你设计你的  Apache Beam pipeline（管道）。其中包含如何确定你的 pipeline 结构，选择什么样的 transforms（转换）应用到数据中，如何确定你的输入和输出方法。

在阅读这一节之前，推荐你先对 [Beam programming guide]({{ site.baseurl }}/documentation/programming-guide) 的内容进行熟悉。


## 当设计你的pipeline 需要考虑什么

当设计你的 Beam pipeline，考虑一些基础问题。

*   **你的输入数据存储在哪里?** 你的输入数据有多少？这会决定你的pipelien 在开始的时候做什么样的 `Read` Transform（转换）操作。
*   **你的数据是什么样的?** 可能是明文，格式化的日志文件或者数据库表中的行.一些 Beam transforms 仅仅对 键值对的  `PCollection` 有效；你需要确定你的数据如何转换成键值对数据，如何在 pipeline 的 `PCollection`(s) 中更好的表示。

*   **你想要对你的数据干什么?** Beam SDKs 的核心 transforms 是通用的。知道如何更改和操纵你的数据将决定如何构建核心的 transforms（转换），就像[ParDo]({{ site.baseurl }}/documentation/programming-guide/#pardo)，或者当你使用 Beam SDKs 在写入之前的预先转换的时候。
*   **你的输出数据要是什么样的，你想存储到哪里?** 这一步会确定在 pipelien 结束的时候使用什么样的  `Write` transforms（转换）。

## 一个基础 pipeline。

最简单的 pipeline 代表一个线性的操作流程，如下图1:


<figure id="fig1">
    <img src="{{ site.baseurl }}/images/design-your-pipeline-linear.png"
         alt="A linear pipeline.">
</figure>
图 1: 一个线性的 pipeline.

但是，你的 pipeline可能会更加复杂。一个 pipeline 表示 [Directed Acyclic Graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph) 步骤。这个pipeline 包含多个输入源，多个输出 sinks，并且它的操作(`PTransform`s) 都会读取和输出多个 `PCollection`s。下面的例子展示了管道可以采取一些不同的形状。


## Branching PCollections
比较重要的是理解 transforms 不会消费 `PCollection`；相反，他们考虑`PCollection` 的每个单独的元素，并创建一个新的 `PCollection`作为输出。在这种方式下，你可以为同一个 `PCollection` 中的不同元素做不同的事情。 


### Multiple transforms process the same PCollection

你可以将相同的 `PCollection` 作为多个 transforms的输入，不会消费和转换输入。

图2所示的 pipeline 读取它的输入，得到第一个名称数据，从一个单一的源，数据库表，并创建一个基于table row的 `PCollection`。然后，pipeline 对相同的 `PCollection` 执行多个转换。 Transform A 将`PCollection`中以字母 A开头的名称导出，Transform B 将 `PCollection`中以字母B开头的名称导出。Transform A和B有相同的输入 `PCollection`。


<figure id="fig2">
    <img src="{{ site.baseurl }}/images/design-your-pipeline-multiple-pcollections.png"
         alt="A pipeline with multiple transforms. Note that the PCollection of table rows is processed by two transforms.">
</figure>
图2: 有多个 transforms 的pipeline。请注意，这个数据库表 rows的 PCollection被两个 transforms处理。例子代码如下:
```java
PCollection<String> dbRowCollection = ...;

PCollection<String> aCollection = dbRowCollection.apply("aTrans", ParDo.of(new DoFn<String, String>(){
  @ProcessElement
  public void processElement(ProcessContext c) {
    if(c.element().startsWith("A")){
      c.output(c.element());
    }
  }
}));

PCollection<String> bCollection = dbRowCollection.apply("bTrans", ParDo.of(new DoFn<String, String>(){
  @ProcessElement
  public void processElement(ProcessContext c) {
    if(c.element().startsWith("B")){
      c.output(c.element());
    }
  }
}));
```

### 一个单一的 transform 生产出多个输出。

另一种对 pipeline 做分支的方式是通过 [tagged outputs]({{ site.baseurl }}/documentation/programming-guide/#additional-outputs) 将一个单一的transform输出到多个 `PCollection`。Transforms（转换）产生多个输出过程，每个输入元素一次，并输出到零个或多个 `PCollection`。

下面图3 描述了相同的例子，但是一个transform（转换）产生多个输出。以字母 'A' 开头的名称被添加到主要的输出 `PCollection`，以字母 'B'开头的名称被添加到另一个输出 `PCollection`。


<figure id="fig3">
    <img src="{{ site.baseurl }}/images/design-your-pipeline-additional-outputs.png"
         alt="A pipeline with a transform that outputs multiple PCollections.">
</figure>
Figure 3: A pipeline with a transform that outputs multiple PCollections.

图2的 pipeline 包含两个transforms，两个transforms 处理相同的输入 `PCollection` 元素，一个 transform 使用下面的逻辑:


<pre>if (starts with 'A') { outputToPCollectionA }</pre>

另一个 transform 使用 :

<pre>if (starts with 'B') { outputToPCollectionB }</pre>

因为每个 transform 读取完整的输入 `PCollection`，所以输入`PCollection` 的每个元素被处理两次。

图 3 的pipeline以另外一种方式执行相同的操作 - 只使用一个transform，使用下面的逻辑:


<pre>if (starts with 'A') { outputToPCollectionA } else if (starts with 'B') { outputToPCollectionB }</pre>

输入 `PCollection` 的每个元素被处理一次，示例代码如下:

```java
// Define two TupleTags, one for each output.
final TupleTag<String> startsWithATag = new TupleTag<String>(){};
final TupleTag<String> startsWithBTag = new TupleTag<String>(){};

PCollectionTuple mixedCollection =
    dbRowCollection.apply(ParDo
        .of(new DoFn<String, String>() {
          @ProcessElement
          public void processElement(ProcessContext c) {
            if (c.element().startsWith("A")) {
              // Emit to main output, which is the output with tag startsWithATag.
              c.output(c.element());
            } else if(c.element().startsWith("B")) {
              // Emit to output with tag startsWithBTag.
              c.output(startsWithBTag, c.element());
            }
          }
        })
        // Specify main output. In this example, it is the output
        // with tag startsWithATag.
        .withOutputTags(startsWithATag,
        // Specify the output with tag startsWithBTag, as a TupleTagList.
                        TupleTagList.of(startsWithBTag)));

// Get subset of the output with tag startsWithATag.
mixedCollection.get(startsWithATag).apply(...);

// Get subset of the output with tag startsWithBTag.
mixedCollection.get(startsWithBTag).apply(...);
```

你可以使用这两种机制产生多个输出 `PCollection`。但是，如果 transform 的计算中每个元素都是耗时的，那么使用额外的输出更有效果。


## 合并 PCollections

通常，通过多个 transforms 将`PCollection`分成多个 `PCollection`，你可能想要合并一些或者所有的`PCollection`s。你可以使用下面其中一个操作来做:


*   **Flatten** - 你可以使用 Beam SDKs中的 `Flatten`转换来合并多个相同类型的`PCollection`。 
*   **Join** - 你可以使用Beam SDK中的 `CoGroupByKey` transform 将两个 `PCollection` 做一个关联Join。 这个`PCollection` 必须是键值对类型的数据，并且这两个 `PCollection`必须有相同的 key 类型。 


图4示例是[上面](#multiple-transforms-process-the-same-pcollection)图2所示示例的延续。在分解成两个 `PCollection` 后，一个是以 'A' 开头名称的 `PCollection`，一个是以 'B' 开头名称的`PCollection`，pipeline 合并两个`PCollection`到一个`PCollection`，这个`PCollection`包含所有以 'A' 或者'B'的名称， 在这里，使用 `Flatten`是有意义的，因为`PCollection`被合并，都包含相同的key类型。


<figure id="fig4">
    <img src="{{ site.baseurl }}/images/design-your-pipeline-flatten.png"
         alt="Part of a pipeline that merges multiple PCollections.">
</figure>
Figure 4: Part of a pipeline that merges multiple PCollections. See the example code below:
```java
//merge the two PCollections with Flatten
PCollectionList<String> collectionList = PCollectionList.of(aCollection).and(bCollection);
PCollection<String> mergedCollectionWithFlatten = collectionList
    .apply(Flatten.<String>pCollections());

// continue with the new merged PCollection		
mergedCollectionWithFlatten.apply(...);
```

## 多个 sources

你的 pipeline 从一个或者多个 sources读取数据。如果你的 pipeline 从多个source读取数据，并且这些sources的数据是相关联的，这样就可以对输入做 join。我们来看下面图5的例子，这个 pipeline 从数据库表读取names和address，还有从Kafka topic读取的names和order numbers。这个 pipeline可以使用 `CoGroupByKey` 做join，key 就是nme; 结果 `PCollection`包含所有 names, addresses, and orders的合并。


<figure id="fig5">
    <img src="{{ site.baseurl }}/images/design-your-pipeline-join.png"
         alt="A pipeline with multiple input sources.">
</figure>
Figure 5: A pipeline with multiple input sources. See the example code below:
```java
PCollection<KV<String, String>> userAddress = pipeline.apply(JdbcIO.<KV<String, String>>read()...);

PCollection<KV<String, String>> userOrder = pipeline.apply(KafkaIO.<String, String>read()...);

final TupleTag<String> addressTag = new TupleTag<String>();
final TupleTag<String> orderTag = new TupleTag<String>();

// Merge collection values into a CoGbkResult collection.
PCollection<KV<String, CoGbkResult>> joinedCollection =
  KeyedPCollectionTuple.of(addressTag, userAddress)
                       .and(orderTag, userOrder)
                       .apply(CoGroupByKey.<String>create());

coGbkResultCollection.apply(...);
```

##  下一步

*   [创建你自己的 pipeline]({{ site.baseurl }}/documentation/pipelines/create-your-pipeline).
*   [测试你的 pipeline]({{ site.baseurl }}/documentation/pipelines/test-your-pipeline).
