---
layout: default
title: "Beam Execution Model"
permalink: /documentation/execution-model/
---

# Apache Beam 执行模型

Beam模型允许runners选择不同的方式执行你的管道(pipeline). 由于runner的这种选择，你可能会观察到不同的影响. 本页描述了这些影响，以便于你可以理解Beam管道(Pipeline)是如何执行的.

* toc
{:toc}

## 处理元素

在分布执行管道时候，序列化和机器之间的通信传输是最耗费资源的操作之一. 假如想避免序列化，则需要在失败之后重新处理或者限制分布输出到其他机器.

### 序列化和通信传输

为了在节点之间传输数据或者其他原因如持久化，runner需要序列化元素.

一个runner在transform之间传输元素的方式有很多种, 例如:

1.  在分组操作中将元素路由到某个节点.这可能牵扯到序列化元素和根据key分组或排序.

1.  在节点之间重新分配元素以便调整并行度. 这可能涉及到序列化元素和节点之间的数据传输.

1.  在`ParDo`中使用side input的元素. 这可能需要序列化和广播它们到所有执行`ParDo`的节点.    
   
1.  在同一个节点的不同transform之间传递元素. 这可能允许runner不对元素进行序列化，而直接在内存中传递元素.
    
在下面情况runner可能需要序列化和持久化元素:

1. 当使用有状态的`DoFn`时候, runner可能使用某些机制去持久化元素. 
1. 当提交处理结果时候，runner可能持久化输出为一个checkpoint.


### Bundling 和持久化

Beam管道经常关注"[embarassingly parallel](https://en.wikipedia.org/wiki/embarrassingly_parallel)"问题，
因为APIs强调并行的处理元素，会导致表达一些行为很困难，例如"给一个PCollection里面的每一个元素分配一个序号". 
这样做是有意义的，因为很多算法正在遭受扩展性问题.

并行化处理所有元素也有一些缺点. 特别的，很难对一些操作进行打包，例如在处理过程中往一个sink或者checkpointing progress写数据.

Beam并不是持续不断的处理所有数据，在
`PCollection`中的数据是以 _bundles_ 的形式被处理的. 将数据分割成bundles 是由runner完成的. 
这允许runner在失败发生时候，在提交每一个元素和全部重试之间选择一个合适的立场. 
例如，一个流式runner可能更喜欢处理和提交小的bundles, 而批处理runner更喜欢大的bundles.

## transforms之内和之间的失败和并行度 {#parallelism}

在这一章,我们将讨论输入输入集合中的元素是如何并行的被处理的，和当失败发生的时候transform是如何重试的.

### 在同一个transform中的数据并行 {#data-parallelism}

在执行单个`ParDo` ，runner可能会将示例输入集合中的九个元素分成两个包，如图1所示.

![bundling]({{ site.baseurl }}/images/execution_model_bundling.svg)

**Figure 1:** runner将包含九个元素的输入集合分成两个包.

`ParDo` 执行时，节点可以并行处理这两个包，如图2所示。.

![bundling_gantt]({{ site.baseurl }}/images/execution_model_bundling_gantt.svg)

**Figure 2:** 两个节点并行处理两个包。 每个包中的元素按顺序处理.

由于不能拆分元素，因此变换的最大并行度取决于集合中元素的数量。 在我们的例子中，输入集合有九个元素，所以最大并行度是九.

![bundling_gantt_max]({{ site.baseurl }}/images/execution_model_bundling_gantt_max.svg)

**Figure 3:** 最大并行度是9，因为输入集合中有9个元素.

Note: Splittable ParDo allows splitting the processing of a single input across
multiple bundles. This feature is a work in progress.

注意：可拆分的ParDo允许将多个包中的单个输入的处理分开,此功能正在开发中.

### 变换之间的依赖并行性 {#dependent-parallellism}

如果runner选择在产生输出元素的变换上执行消费变换而不改变包的大小，那么依次并行的 `ParDo`变换可以是 _dependently parallel_ 的。 在图4中，如果给定元素的`ParDo1`的输出必须在同一个worker上处理，则`ParDo1`和`ParDo2`是独立并行的。



![bundling_multi]({{ site.baseurl }}/images/execution_model_bundling_multi.svg)

**Figure 4:** 按顺序进行两次转换以及相应的输入集合.

图5显示了这些依赖并行转换如何执行。 第一个worker对bundle A中的元素（导致bundle C）执行`ParDo2` ，然后对bundle C中的元素执行`ParDo2` 。同样，第二个worker对bundle B中的元素执行`ParDo1` （导致bundle D） ，然后对包D中的元素执行`ParDo2` 。


![bundling_multi_gantt.svg]({{ site.baseurl }}/images/execution_model_bundling_multi_gantt.svg)

**Figure 5:** 两个节点执行独立并行的ParDo变换


以这种方式执行转换允许runner避免在节点之间重新分配元素，这节省了通信成本。 但是，最大平行度现在取决于第一个转换的最大平行度。


### 同一个转换中的失败处理

如果处理一个包中的一个元素失败，那么整个包就会失败。 包中的元素必须重试（否则整个管道将失败），尽管不需要使用相同的捆绑重试。

在这个例子中，我们将使用图1中的`ParDo` ，它具有九个元素的输入集合，并被分成两个包。

在图6中，第一个节点成功处理bundle A中的所有五个元素。第二个节点处理bundle B中的四个元素：前两个元素成功处理，第三个元素处理失败，还有一个元素仍在等待处理。

我们看到runner重试了bundle B中的所有元素，并且第二次成功完成了处理。 请注意，重试不一定发生在与原始处理尝试相同的节点上，如图所示。


![failure_retry]({{ site.baseurl }}/images/execution_model_failure_retry.svg)

**Figure 6:** bundle B中元素的处理失败，另一个worker重试整个bundle.

由于在处理输入包中的元素时失败，因此我们不得不重新处理输入包中的所有元素。 这意味着runner必须扔掉整个输出包，因为它包含的所有结果都将被重新计算。
请注意，如果失败的转换是`ParDo` ，则`DoFn`实例将被拆除并放弃。



### 耦合失败：转换之间的失败 {#coupled-failure}

如果在`ParDo2`无法处理元素会导致`ParDo1`重新执行，则说这两个步骤是 _co-failing_ 。

对于这个例子，我们将使用图4中的两个`ParDo` 。

在图7中，worker 2成功地对bundle B中的所有元素执行了`ParDo1` 。但是，worker没有处理bundle D中的元素，所以`ParDo2`失败（显示为红色的X）。 因此，runner必须丢弃并重新计算`ParDo2`的输出。 因为runner一起执行`ParDo1`和`ParDo2` ，所以`ParDo2`的输出包也必须被扔掉，并且必须重试输入包中的所有元素。 这两个ParDo是co-failing。



![bundling_coupled failure]({{ site.baseurl }}/images/execution_model_bundling_coupled_failure.svg)

**Figure 7:** 处理bundle D中的元素失败，所以输入bundle中的所有元素都被重试。.

请注意，重试不一定具有与原始尝试相同的处理时间，如图所示。

所有遇到耦合故障的`DoFns`都将被终止，并且必须被抛弃，因为它们没有遵循正常的`DoFn`生命周期。

以这种方式执行转换允许runner避免在转换之间保持元素，节省持久化成本。

