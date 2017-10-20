---
layout: default
title: "Beam Programming Guide"
permalink: /documentation/programming-guide/
redirect_from:
  - /learn/programming-guide/
  - /docs/learn/programming-guide/
---

# Apache Beam 编程指南

 **Beam 编程指南** 适用于想要使用的Beam用户Beam SDK创建数据处理流水线。 它提供使用的指导Beam SDK类来构建和测试您的管道。 它不是作为一个详尽的参考，但作为语言无关的高级指导以编程方式构建您的梁管道。 随着编程指南的填补
文本将包含多种语言的代码示例，以帮助说明如何在您的管道中实施Beam概念。

<nav class="language-switcher">
  <strong>适用:</strong>
  <ul>
    <li data-type="language-java" class="active">Java SDK</li>
    <li data-type="language-py">Python SDK</li>
  </ul>
</nav>

**目录:**



## 1.概述

要使用Beam，您需要首先使用一个类中的类创建一个驱动程序的Beam SDK。 你的驱动程序*定义了你的管道，包括所有的输入，变换和输出; 它还为您设置执行选项管道（通常使用命令行选项传递）。 这些包括管道运行器，这又决定了您的管道将运行什么后端。

Beam SDK提供了许多简化机制的抽象大规模分布式数据处理。 同样的光束抽象与批量和流数据源。 当你创建你的梁管道，你可以根据这些抽象思考你的数据处理任务。 他们
包括：

* `Pipeline`: 一个 `Pipeline` 封装您的整个数据处理任务开始完成 这包括读取输入数据，转换数据，以及编写输出数据。 所有Beam驱动程序都必须创建一“Pipeline”。 什么时候您创建“Pipeline”，您还必须指定执行选项告诉“Pipeline”在哪里和如何运行。

* `PCollection`:“PCollection”表示一个分布式数据集梁管道运行。 数据集可以是*有界的，这意味着它来了来自固定的源文件，或*无界*，意思来自于通过订阅或其他机制不断更新源。 你的管道通常通过从中读取数据来创建一个初始“PCollection”外部数据源，但也可以从内存中创建一个“PCollection”。您的驱动程序中的数据。 从那里，“PCollection”是输入和 输出您的管道中的每个步骤。

* `Transform`: “Transform”表示数据处理操作，或步骤，在你的管道 每个“Transform”都会使用一个或多个“PCollection”对象输入，执行您提供的元素的处理功能`PCollection`，并产生一个或多个输出`PCollection`对象。

* I/O `Source` and `Sink`:Beam提供“Source”和“Sink”API来表示分别读写数据。 `Source`封装了代码从一些外部来源读取数据到您的光束管道是必要的，如作为云文件存储或订阅流数据源。`Sink`同样封装了写入的元素所需的代码“PCollection”到外部数据接收器。

典型的Beam驱动程序的工作原理如下:

* 创建一个“Pipeline”对象并设置管道执行选项，包括管道运行器。
* 为管道数据创建一个初始的“PCollection”，使用“Source” API从外部源读取数据，或使用“Create”转换从内存数据构建一个“PCollection”。
* 应用**将**转换为每个“PCollection”。 变换可以改变，过滤，组合，分析或以其他方式处理“PCollection”中的元素。 一个
transform创建一个新的输出`PCollection` *，而不消耗输入
采集*。 典型的管道将后续转换应用于每个新的输出“PCollection”，直到处理完成。
* 输出最终的转换后的“PCollection”，通常使用“Sink”API将数据写入外部源。
* **使用指定的流水线运行**管道。

当您运行您的Beam驱动程序时，您指定的管道运行器
根据“PCollection”构建您的管道的**工作流图**
您已创建并转换已应用的对象。 那个图是那个
使用适当的分布式处理后端执行，成为
异步“作业”（或等效的）在后端。

## 2. 创建管道

“Pipeline”抽象封装了数据中的所有数据和步骤处理任务。 您的光束驱动程序通常从构建开始
<span class="language-java">[Pipeline]({{ site.baseurl }}/documentation/sdks/javadoc/{{ site.release_latest }}/index.html?org/apache/beam/sdk/Pipeline.html)</span>
<span class="language-py">[Pipeline](https://github.com/apache/beam/blob/master/sdks/python/apache_beam/pipeline.py)</span>
对象，然后使用该对象作为创建管道数据的基础
设置为“PCollection”，其操作为“变换”。

要使用Beam，您的驱动程序必须首先创建一个Beam SDK的实例类'Pipeline`（通常在`main（）`函数中）。 当你创建你的
`Pipeline`，你还需要设置一些**配置选项**。 你可以设置您的管道的配置选项以编程方式，但通常更容易提前设置选项（或从命令行读取它们）并传递它们到创建对象时的“Pipeline”对象。

```java
// Start by defining the options for the pipeline.
PipelineOptions options = PipelineOptionsFactory.create();

// Then create the pipeline.
Pipeline p = Pipeline.create(options);
```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:pipelines_constructing_creating
%}
```

### 2.1.配置管道选项

使用管道选项来配置管道的不同方面，例如作为管道运行者，将执行您的管道和任何运动员的具体所选择的跑步者所需的配置。 你的管道选项将会可能包括诸如您的项目ID或位置的信息存储文件。

当您选择的跑步者上运行管道时，该副本
PipelineOptions将可用于您的代码。 例如，您可以阅读
来自DoFn上下文的流水线选项。

#### 2.1.1.从命令行参数设置PipelineOptions

虽然您可以通过创建一个“PipelineOptions”对象来配置您的管道直接设置字段，Beam SDK包括一个命令行解析器您可以使用命令行参数在“PipelineOptions”中设置字段。

要从命令行读取选项，构造您的“PipelineOptions”对象
如以下示例代码所示：

```java
MyOptions options = PipelineOptionsFactory.fromArgs(args).withValidation().create();
```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:pipelines_constructing_creating
%}
```

这将解释遵循以下格式的命令行参数：

```
--<option>=<value>
```

> **Note:** 附加方法`.withValidation`将检查所需的命令行参数并验证参数值。

以这种方式构建您的“PipelineOptions”，可以将任何选项指定为
一个命令行参数。

> **Note:** The [WordCount示例管道]({{ site.baseurl }}/get-started/wordcount-example)
> 演示如何使用命令行选项在运行时设置管道选项。

#### 2.1.2. 创建自定义选项

除了标准之外，您还可以添加自己的自定义选`PipelineOptions`。要添加您自己的选项，请使用getter和setter方法为每个选项，如下例所示：

```java
public interface MyOptions extends PipelineOptions {
    String getMyCustomOption();
    void setMyCustomOption(String myCustomOption);
  }
```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:pipeline_options_define_custom
%}
```

您还可以指定用户通过`--help`时显示的描述
命令行参数和默认值。

您可以使用注释设置描述和默认值，如下所示：

```java
public interface MyOptions extends PipelineOptions {
    @Description("My custom command line argument.")
    @Default.String("DEFAULT")
    String getMyCustomOption();
    void setMyCustomOption(String myCustomOption);
  }
```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:pipeline_options_define_custom_with_help_and_default
%}
```

{language-java}
建议您使用“PipelineOptionsFactory”注册您的界面
然后在创建“PipelineOptions”对象时传递界面。 当你
用“PipelineOptionsFactory”注册你的界面，`--help`可以找到
您的自定义选项界面，并将其添加到`--help`命令的输出。
`PipelineOptionsFactory`也会验证你的自定义选项是
与所有其他注册选项兼容。

{language-java}
以下示例代码显示如何注册您的自定义选项界面
与“PipelineOptionsFactory”：

```java
PipelineOptionsFactory.register(MyOptions.class);
MyOptions options = PipelineOptionsFactory.fromArgs(args)
                                                .withValidation()
                                                .as(MyOptions.class);
```

现在您的管道可以接受`--myCustomOption = value`作为命令行参数。

## 3. PCollections

The <span class="language-java">[PCollection]({{ site.baseurl }}/documentation/sdks/javadoc/{{ site.release_latest }}/index.html?org/apache/beam/sdk/values/PCollection.html)</span>
<span class="language-py">`PCollection`</span> 抽象代表一个潜在的分布式多元素数据集。 你可以想到一个
“PCollection”为“管道”数据; 梁变换使用“PCollection”对象作为输入和输出。 因此，如果您想要处理管道中的数据必须以“PCollection”的形式。


创建“Pipeline”之后，您需要先创建至少一个“PCollection”在某种形式。 您创建的“PCollection”作为输入为您的管道中的第一个操作。

### 3.1.创建PCollection

您可以通过使用外部源读取数据来创建“PCollection”
Beam的[Source API]（＃pipeline-io），或者你可以创建一个“PCollection”的数据
存储在驱动程序中的内存中集合类中。 前者是
通常生产线将如何摄取数据; 梁的源API
包含适配器，以帮助您从大型基于云的外部来源阅读
文件，数据库或订阅服务。 后者主要用于
测试和调试目的。

#### 3.1.1. 从外部来源读取
要从外部来源读取，请使用[Beam提供的I / O]
适配器（＃管道-IO）。 适配器的确切用途有所不同，但都是这些
从一些外部数据源并返回一个“PCollection”的元素
代表该来源中的数据记录。

每个数据源适配器都有一个`Read`变换; 阅读，你必须申请转换为“Pipeline”对象本身。
<span class="language-java">`TextIO.Read`</span>
<span class="language-py">`io.TextFileSource`</span>, 或者从一个
外部文本文件并返回一个“PCollection”，其元素是类型
`String`，每个`String`代表文本文件中的一行。 这是你的方式
将适用<span class="language-java">`TextIO.Read`</span>
<span class="language-py">`io.TextFileSource`</span> 到你的“管道”来创建一个“PCollection”：

```java
public static void main(String[] args) {
    // Create the pipeline.
    PipelineOptions options =
        PipelineOptionsFactory.fromArgs(args).create();
    Pipeline p = Pipeline.create(options);

    // Create the PCollection 'lines' by applying a 'Read' transform.
    PCollection<String> lines = p.apply(
      "ReadMyFile", TextIO.read().from("protocol://path/to/some/inputData.txt"));
}
```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:pipelines_constructing_reading
%}
```

请参阅[I / O]部分（＃pipeline-io）以了解有关如何从中读取的更多信息
Beam SDK支持的各种数据源。

#### 3.1.2.从内存数据创建PCollection

{language-java}
要从内存中的Java集合中创建一个“PCollection”，你可以使用
Beam提供的`Create`变换。 很像数据适配器的“读”，你应用
`直接创建'到你的`Pipeline`对象本身。

{language-java}
作为参数，“Create”接受Java“Collection”和“Coder”对象。该
`Coder`指定'Collection`中的元素应该如何
[encoded](#element-type).

{language-py}
要从内存“列表”中创建一个“PCollection”，您可以使用Beam提供的`Create`变换。 将此转换直接应用于您的“Pipeline”对象本身。

以下示例代码显示了如何从内存中创建“PCollection”
<span class="language-java">`List`</span><span class="language-py">`列表`</span>:

```java
public static void main(String[] args) {
    // Create a Java Collection, in this case a List of Strings.
    static final List<String> LINES = Arrays.asList(
      "To be, or not to be: that is the question: ",
      "Whether 'tis nobler in the mind to suffer ",
      "The slings and arrows of outrageous fortune, ",
      "Or to take arms against a sea of troubles, ");

    // Create the pipeline.
    PipelineOptions options =
        PipelineOptionsFactory.fromArgs(args).create();
    Pipeline p = Pipeline.create(options);

    // Apply Create, passing the list and the coder, to create the PCollection.
    p.apply(Create.of(LINES)).setCoder(StringUtf8Coder.of())
}
```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:model_pcollection
%}
```

### 3.2.PCollection特性

一个`PCollection`由它所属的具体`Pipeline`对象拥有
创建; 多个管道不能共享“PCollection”。 在某些方面，a
“PCollection”的功能就像一个集合类。 但是，“PCollection”可以
在几个关键方面有所不同：

#### 3.2.1. 元素类型

“PCollection”的元素可能是任何类型的，但都必须相同
类型。 然而，为了支持分布式处理，Beam需要能够
将每个单独的元素编码为字节串（因此可以传递元素
周围分配工作人员）。 光束SDK提供了一种数据编码机制
其中包括常用类型的内置编码以及支持
根据需要指定自定义编码。

#### 3.2.2. 不变性

“PCollection”是不可变的。 创建后，您无法添加，删除或更改
个人元素 光束变换可以处理一个的每个元素
“PCollection”并生成新的流水线数据（作为一个新的“PCollection”），但它
不消耗或修改原始输入集合*。

#### 3.2.3. 随机访问

“PCollection”不支持对单个元素的随机访问。 代替，
Beam变换单独考虑“PCollection”中的每个元素。

#### 3.2.4. 尺寸和边界

“PCollection”是一个大的，不可变的“包”元素。 没有上限“PCollection”可以包含多少个元素; 任何给定的“PCollection”可能
在单个机器上适合内存，或者它可能代表非常大的
由持久数据存储支持的分布式数据集。

“PCollection”可以是**有界**或**无界**的大小。 一个
**有界**“PCollection”表示已知的固定大小的数据集，而
**无限制**“PCollection”表示无限大小的数据集。 无论是
“PCollection”是有界或无界取决于数据集的来源
它代表。 从批量数据源（例如文件或数据库）读取数据，
创建一个有界的“PCollection”。 从流媒体或
不断更新的数据源，如Pub / Sub或Kafka，创建一个无界的
“PCollection”（除非你明确告诉它不要）。

您的“PCollection”的有界（或无界）性质影响了梁
处理您的数据。 有限的“PCollection”可以使用批处理作业进行处理，
这可能会读取整个数据集一次，并在其中执行处理
有限长。 必须使用流式处理无限制的“PCollection”
连续运行的作业，因为整个集合永远不可用
随时处理。

当执行组合无限“PCollection”中的元素的操作时，
光束需要一个称为**窗口**的概念来分割不断更新
数据集合成有限大小的逻辑窗口。 梁将每个窗口处理为
捆绑，并且随着数据集的生成，处理继续进行。 这些逻辑
窗口由与数据元素相关联的一些特征确定，
如**时间戳**。

#### 3.2.5. 元素时间戳

“PCollection”中的每个元素都具有相关联的内在**时间戳**。该
每个元素的时间戳最初由[Source]（＃pipeline-io）
它创建了“PCollection”。 来源创造一个无限的“PCollection”
通常为每个新元素分配与元素何时相对应的时间戳
被阅读或添加。

> **Note**: 来源为固定数据集创建一个有界的“PCollection”
也自动分配时间戳，但最常见的行为是
  分配每个元素相同的时间戳（`Long.MIN_VALUE`）。

时间戳对于包含元素的“PCollection”很有用
固有的时间观念。 如果您的管道正在读取一系列事件，如
推文或其他社交媒体消息，每个元素可能会使用事件的时间
被发布为元素时间戳。

您可以手动将时间戳分配给“PCollection”的元素
源不为你做。 如果元素有一个，你会想要这样做
固有的时间戳，但时间戳是在结构的某处
元素本身（如服务器日志条目中的“时间”字段）。 梁有
[变换]（＃变换）将“PCollection”作为输入并输出
相同的“PCollection”，附有时间戳; 请参阅[分配
时间戳]
(#adding-timestamps-to-a-pcollections-elements)了解更多信息
关于怎么做

## 4. Transforms

转换是您的管道中的操作，并提供通用的
处理框架。 您以函数的形式提供处理逻辑
对象（俗称“用户代码”），并且您的用户代码被应用
输入“PCollection”（或多个“PCollection”）的每个元素。
根据您选择的管道运行器和后端，有很多不同
群集中的员工可以并行执行用户代码的实例。
在每个工作人员上运行的用户代码生成输出元素
最终添加到转换产生的最终输出“PCollection”中。

Beam SDK包含许多可以应用的不同转换
你的管道的“PCollection”。 这些包括通用核心转换，
例如[ParDo]（＃pardo）或[Combine]（＃combine）。 还有预先写的
[复合变换]（＃复合转换）包含在SDK中
在有用的处理模式中组合一个或多个核心变换，如
作为计数或组合集合中的元素。 你也可以定义自己的
更复杂的复合转换适合您的管道的确切用例。

### 4.1. 应用transforms

要调用变换，您必须**将其应用于输入“PCollection”。 每
在Beam SDK中的转换有一个通用的`apply`方法<span class="language-py">(or pipe operator `|`)</span>.
调用多个波束变换类似于*方法链接*，但使用一个
轻微差异：将变换应用于输入“PCollection”，传递
转换本身作为参数，并且操作返回输出
`PCollection`。 这取一般形式：

```java
[Output PCollection] = [Input PCollection].apply([Transform])
```
```py
[Output PCollection] = [Input PCollection] | [Transform]
```

因为Beam对于“PCollection”使用通用的“apply”方法，所以你都可以链接
顺序转换，并且还应用包含其他变换的变换
在Beam中嵌套（称为[复合变换]（＃复合变换））
软件开发工具包）。

如何应用管道的变换决定您的结构
管道。 想想你的管道的最好方法就是一个有针对性的无环图，
其中节点是“PCollection”，边是变换。 例如，
你可以链式变换创建一个顺序管道，像这样：

```java
[Final Output PCollection] = [Initial Input PCollection].apply([First Transform])
.apply([Second Transform])
.apply([Third Transform])
```
```py
[Final Output PCollection] = ([Initial Input PCollection] | [First Transform]
              | [Second Transform]
              | [Third Transform])
```

上述管道的生成工作流程图如下所示：

[Sequential Graph Graphic]

但是，请注意，变换*不消耗或以其他方式改变输入
集合 - 记住，“PCollection”是不可变的定义。 意即
您可以将多个变换应用于同一个输入“PCollection”来创建
分支管道，如：

```java
[Output PCollection 1] = [Input PCollection].apply([Transform 1])
[Output PCollection 2] = [Input PCollection].apply([Transform 2])
```
```py
[Output PCollection 1] = [Input PCollection] | [Transform 1]
[Output PCollection 2] = [Input PCollection] | [Transform 2]
```

从上面的分支管道生成的工作流图如下所示：

[Branching Graph Graphic]

你也可以建立自己的[复合变换]（＃复合转换）
将多个子步骤嵌入到单个较大的变换中。 复合变换
对于构建可重复使用的简单步骤序列特别有用
被用在很多不同的地方。

### 4.2.核心Beam变换
Beam提供以下核心转换，每个变换代表不同的变换
加工范式：

* `ParDo`
* `GroupByKey`
* `CoGroupByKey`
* `Combine`
* `Flatten`
* `Partition`

#### 4.2.1. ParDo

`ParDo`是用于通用并行处理的波束变换。 “ParDo”
处理范例类似于Map / Shuffle / Reduce风格的“Map”阶段
算法：“ParDo”转换考虑了输入中的每个元素
`PCollection`，执行一些处理功能（你的用户代码）
元素，并向输出“PCollection”发出零个，一个或多个元素。

`ParDo`可用于各种常见的数据处理操作，包括：

* **过滤数据集.** 你可以使用`ParDo`来考虑一个元素“PCollection”，并将该元素输出到新集合，或丢弃它。
* **格式化或类型转换数据集中的每个元素.** 如果你的输入
   “PCollection”包含与不同类型或格式的元素
   你想要的，你可以使用`ParDo`对每个元素执行一个转换
   将结果输出到一个新的“PCollection”。
* **提取数据集中每个元素的部分.** 如果你有
 “PCollection”的记录有多个字段，例如可以使用a
 “ParDo”可以将您想要考虑的字段解析为新的
`PCollection`。
* **对数据集中的每个元素执行计算.** 你可以使用`ParDo` 对每个元素执行简单或复杂的计算，或确定元素，“PCollection”，并将结果输出为新“PCollection”。

在这样的角色中，“ParDo”是管道中的常见中间步骤。 你可能
使用它从一组原始输入记录中提取某些字段，或转换原始
输入到不同的格式; 您也可以使用“ParDo”转换处理
数据转换为适合输出的格式，如数据库表行或可打印
字符串。

当您应用“ParDo”变换时，您需要在表单中提供用户代码
的“DoFn”对象。 `DoFn`是一个定义一个分布式的Beam SDK类
处理功能。

> 当您创建“DoFn”的子类时，请注意您的子类应遵循
  [Beam 变换用户代码的编写要求](#requirements-for-writing-user-code-for-beam-transforms).

##### 4.2.1.1.应用 ParDo

像所有的Beam变换一样，你通过调用`apply`方法来应用`ParDo`
输入“PCollection”并传递“ParDo”作为参数，如图所示
以下示例代码：

```java
// The input PCollection of Strings.
PCollection<String> words = ...;

// The DoFn to perform on each element in the input PCollection.
static class ComputeWordLengthFn extends DoFn<String, Integer> { ... }

// Apply a ParDo to the PCollection "words" to compute lengths for each word.
PCollection<Integer> wordLengths = words.apply(
    ParDo
    .of(new ComputeWordLengthFn()));        // The DoFn to perform on each element, which
                                            // we define above.
```
```py
# The input PCollection of Strings.
words = ...

# The DoFn to perform on each element in the input PCollection.
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:model_pardo_pardo
%}
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:model_pardo_apply
%}```

在这个例子中，我们的输入`PCollection`包含`String`值。 我们申请
`ParDo`变换指定一个函数（`ComputeWordLengthFn`）进行计算
每个字符串的长度，并将结果输出到一个新的“PCollection”
存储每个单词长度的“整数”值。

 ####4.2.1.2.创建一个 DoFn

传递给`ParDo`的`DoFn`对象包含处理逻辑应用于输入集合中的元素。 当你使用Beam时，经常你写的最重要的代码是这些“DoFn” - 他们是什么定义管道的确切数据处理任务。

> **Note:** 当您创建“DoFn”时，请注意[要求]
> [用于编写Beam变换的用户代码](#requirements-for-writing-user-code-for-beam-transforms)
>并确保您的代码遵循它们。

{language-java}
一个`DoFn`一次从输入`PCollection`处理一个元素。 当你创建一个`DoFn`的子类，你需要提供匹配的类型参数
输入和输出元素的类型。 如果你的“DoFn”进程进来`String`元素，并为输出集合生成“Integer”元素
（像我们前面的例子'ComputeWordLengthFn`），你的类声明会
看起来像这样：

```java
static class ComputeWordLengthFn extends DoFn<String, Integer> { ... }
```

{language-java}
在你的“DoFn”子类中，你将编写一个注释的方法
`@ ProcessElement`提供实际的处理逻辑。 你不需要
从输入集合中手动提取元素; 光束SDK处理
那为你 你的`@ ProcessElement`方法应该接受类型的对象
`ProcessContext`。 “ProcessContext”对象允许您访问输入
元素和发射输出元件的方法：

{language-py}
在你的“DoFn”子类中，你会写一个方法`process'
实际处理逻辑。 您不需要手动提取元素
从输入集合; Beam SDK可以为您处理。 你的进程
方法应该接受一个类型为“element”的对象。 这是输入元素
在`process`中使用`yield`或`return`语句来发出输出
方法。

```java
static class ComputeWordLengthFn extends DoFn<String, Integer> {
  @ProcessElement
  public void processElement(ProcessContext c) {
    // Get the input element from ProcessContext.
    String word = c.element();
    // Use ProcessContext.output to emit the output element.
    c.output(word.length());
  }
}
```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:model_pardo_pardo
%}
```

{language-java}
> **Note:** 如果您的输入“PCollection”中的元素是键/值对，则可以使用
可以通过使用访问密钥或值 `ProcessContext.element().getKey()` or
> `ProcessContext.element().getValue()`

给定的“DoFn”实例一般被调用一次或多次来处理一些
任意捆绑的元素。 但是，梁不能保证确切的数量
调用; 可以在给定的工作节点上多次调用它来进行帐户管理
失败和重试。 因此，您可以跨多个缓存信息
调用您的处理方法，但如果这样做，请确保实现
**不依赖于调用次数**.

在处理方法中，您还需要满足一些不变性
要求确保梁和加工后端可以安全
序列化并缓存管道中的值。 你的方法应该符合
以下要求：

{language-java}
* 你不应该以任何方式修改返回的元素
  `ProcessContext.element()` or `ProcessContext.sideInput()` (传入元素从输入集合).
* 一旦你输出一个值 `ProcessContext.output()` or
  `ProcessContext.sideOutput()`,你不应该以任何方式修改该值。

##### 4.2.1.3. 轻量级DoFns等抽象

如果您的功能相对简单，您可以简化您的使用
`ParDo`通过提供一个轻量级的“DoFn”在线，
<span class="language-java">一个匿名的内部类实例</span>
<span class="language-py">一个lambda函数</span>.

这是以前的例子，`ParDo`与`ComputeLengthWordsFn`，
`DoFn`指定为
<span class="language-java">an anonymous inner class instance</span>
<span class="language-py">a lambda function</span>:

```java
// 输入PCollection
PCollection<String> words = ...;

// 将具有匿名DoFn的ParDo应用于PCollection字
// 将结果保存为PCollection wordLengths.
PCollection<Integer> wordLengths = words.apply(
  "ComputeWordLengths",                     // the transform name
  ParDo.of(new DoFn<String, Integer>() {    // a DoFn as an anonymous inner class instance
      @ProcessElement
      public void processElement(ProcessContext c) {
        c.output(c.element().length());
      }
    }));
```

```py
# 输入PCollection的字符串。
words = ...

# 将一个lambda函数应用于PCollection.
# 将结果保存为PCollection word_lengths.
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:model_pardo_using_flatmap
%}```

如果您的ParDo对输出元素执行一对一的映射uage-元素 - 即对于每个输入元素，它应用产生的函数
*一个*输出元素，可以使用更高级别
<span class="langjava">`MapElements`</span><span class="language-py">`Map`</span>
transform. <span class="language-java">`MapElements`可以接受匿名
Java 8 lambda功能，用于额外的简洁.</span>

这是以前使用的例子 <span class="language-java">`MapElements`</span>
<span class="language-py">`Map`</span>:

```java
// The input PCollection.
PCollection<String> words = ...;

// App一个MapElements与匿名lambda功能的PCollection
// 将结果保存为PCollection wordLengths.
PCollection<Integer> wordLengths = words.apply(
  MapElements.into(TypeDescriptors.integers())
             .via((String word) -> word.length()));
```

```py
# 输入PCollection的字符串.
words = ...

# 使用lambda函数应用到PCollection字的地图.
# 将结果保存为PCollection word_lengths.
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:model_pardo_using_map
%}```

{language-java}
> **Note:** 您可以使用Java 8 lambda函数与其他几个Beam
  转换，包括`Filter`，`FlatMapElements`和`Partition`。

#### 4.2.2. GroupByKey

`GroupByKey`是一个用于处理键/值对集合的Beam变换。这是一个并行还原操作，类似于a的Shuffle阶段
Map / Shuffle / Reduce-style算法。 “GroupByKey”的输入是一个集合
键/值对代表一个* multimap *，其中集合包含具有相同键，但不同值的多对。 给了这样一个
收集，你使用`GroupByKey`来收集所有与之关联的值每个唯一的键。


`GroupByKey`是汇总具有共同点的数据的好方法。 对于例如，如果您有一个存储客户订单记录的集合
可能希望将来自同一邮政编码的所有订单组合在一起（其中键/值对的“键”是邮政编码字段，“值”是
剩余的记录）。


我们来看一下简单例子的“GroupByKey”的机制我们的数据集由文本文件中的字和其上的行号组成
他们出现 我们想将所有共享的行号（值）组合在一起同一个字（关键），让我们看到文本中的所有地方
特定的词出现。


我们的输入是键/值对的“PCollection”，每个单词都是一个键，而且该值是该文本出现的文件中的行号。 这是一个列表
输入集合中的键/值对：

```
cat, 1
dog, 5
and, 1
jump, 3
tree, 2
cat, 5
dog, 2
and, 2
cat, 9
and, 6
...
```

`GroupByKey`用相同的键收集所有的值并输出一个新的对包括唯一的密钥和所有值的集合与输入集合中的该键相关联。 如果我们应用`GroupByKey`到
我们上面的输入集合，输出集合将如下所示：

```
cat, [1,5,9]
dog, [5,2]
and, [1,2,6]
jump, [3]
tree, [2]
...
```

因此，“GroupByKey”表示从多重映射（多个键到单个值）映射到单一映射（值的集合的唯一键）。

#### 4.2.3. CoGroupByKey

`CoGroupByKey`连接两个或更多的键/值`PCollection`具有相同的键
键入，然后发出“KV <K，CoGbkResult”对的集合。 [设计你的
Pipeline]（{{site.baseurl}} / documentation / pipelines / design-your-pipeline /＃多个来源）
显示使用连接的示例管道。

给出以下输入集合：
```
// collection 1
user1, address1
user2, address2
user3, address3

// collection 2
user1, order1
user1, order2
user2, order3
guest, order4
...
```

`CoGroupByKey`从所有的“PCollection”中收集相同的键值，并输出一个由唯一键和一个对象“CoGbkResult”组成的新对
包含与该关键字相关联的所有值。 如果你申请“CoGroupByKey”到上面的输入集合，输出集合将会显示
像这个：
```
user1, [[address1], [order1, order2]]
user2, [[address2], [order3]]
user3, [[address3], []]
guest, [[], [order4]]
...
````

> **关于 Key/Value Pairs:** Beam represents表示键/值对根据您使用的语言和SDK而有所不同。 在Beam SDK中对于Java，您可以使用类型为“KV <K，V>”的对象表示键/值对。 在
  Python，您使用2-tuples表示键/值对。

#### 4.2.4. Combine

<span class="language-java">[`Combine`]({{ site.baseurl }}/documentation/sdks/javadoc/{{ site.release_latest }}/index.html?org/apache/beam/sdk/transforms/Combine.html)</span>
<span class="language-py">[`Combine`](https://github.com/apache/beam/blob/master/sdks/python/apache_beam/transforms/core.py)</span>
是一个用于组合元素或值集合的Beam变换数据。 “Combine”具有可在整个“PCollection”上运行的变体，还有一些
将“PCollection”中的每个键的值组合key/value pairs.

当您应用“Combine”转换时，您必须提供该功能包含组合元素或值的逻辑。 组合功能
应该是交换和关联，因为功能不一定在给定的键的所有值上调用一次。 因为输入数据
（包括价值收集）可以分布在多个工作人员之间可以多次调用组合功能来执行部分组合
在集合的子集上。 Beam SDK还提供了一些预制的组合函数用于常用的数字组合操作，如sum，min，
和max


简单的组合操作（如和）通常可以简单实现功能。 更复杂的组合操作可能需要您创建一个
具有与...不同的累积类型的“CombineFn”的子类输入/输出类型。

##### 4.2.4.1. 简单组合使用简单的功能

以下示例代码显示了一个简单的组合函数

```java
// Sum a collection of Integer values. The function SumInts implements the interface SerializableFunction.
public static class SumInts implements SerializableFunction<Iterable<Integer>, Integer> {
  @Override
  public Integer apply(Iterable<Integer> input) {
    int sum = 0;
    for (int item : input) {
      sum += item;
    }
    return sum;
  }
}
```

```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:combine_bounded_sum
%}```

##### 4.2.4.2. 使用CombineFn的高级组合

对于更复杂的组合函数，可以定义“CombineFn”的子类。如果组合功能需要更复杂的话，应该使用“CombineFn”
累加器，必须执行额外的预处理或后处理，可能会更改输出类型，或将密钥考虑在内。

一般组合操作由四个操作组成。 当你创建一个“CombineFn”的子类，你必须提供四个操作来覆盖
相应的方法：

1. **创建 Accumulator**创建一个新的“本地”累加器。 在例子中
    取平均值，本地累加器跟踪运行总和值（我们最终平均分数的分子值）和数量值到目前为止（分母值）。 它可以称为任何数量
    时代以分布式的方式。


2. **添加 Input** 向累加器添加一个输入元素，返回累加器值。 在我们的例子中，它将更新和并增加
    计数。 也可以并行调用它。


3. **合并 Accumulators** 将多个累加器合并成单个累加器;这是多个累加器中的数据如何在最终之前组合计算。 在平均计算的情况下，累加器
    代表部门的每一部分合并在一起。 可能是再次在其输出上再次呼叫任何次数.

4. **额外 Output** 执行最终计算。 在计算平均值的情况下，这意味着将所有值的总和除以
    数值总和。 在最后的合并累加器上调用一次。

以下示例代码显示了如何定义一个计算的“CombineFn”
平均值：

```java
public class AverageFn extends CombineFn<Integer, AverageFn.Accum, Double> {
  public static class Accum {
    int sum = 0;
    int count = 0;
  }

  @Override
  public Accum createAccumulator() { return new Accum(); }

  @Override
  public Accum addInput(Accum accum, Integer input) {
      accum.sum += input;
      accum.count++;
      return accum;
  }

  @Override
  public Accum mergeAccumulators(Iterable<Accum> accums) {
    Accum merged = createAccumulator();
    for (Accum accum : accums) {
      merged.sum += accum.sum;
      merged.count += accum.count;
    }
    return merged;
  }

  @Override
  public Double extractOutput(Accum accum) {
    return ((double) accum.sum) / accum.count;
  }
}
```
```py
pc = ...
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:combine_custom_average_define
%}```

如果你正在组合一个“PCollection” key-value pairs, [per-key
combining](#combining-values-in-a-keyed-pcollection)经常就够了 如果
您需要基于键的组合策略进行更改（例如，MIN for一些用户和其他用户的MAX），您可以定义一个“KeyedCombineFn”进行访问
组合策略的关键。

##### 4.2.4.3. 将PCollection组合成单个值

使用全局组合来转换给定的“PCollection”中的所有元素成为一个单一的值，在您的管道中表示为一个新的“PCollection”
包含一个元素。 以下示例代码显示如何应用beam提供总和组合函数以产生“PCollection”的单个和值
的整数。

```java
// Sum.SumIntegerFn() 组合元素 在输入PCollection。 所得到的PCollection，称为sum，
//包含一个值：输入PCollection中所有元素的总和.
PCollection<Integer> pc = ...;
PCollection<Integer> sum = pc.apply(
   Combine.globally(new Sum.SumIntegerFn()));
```
```py
# sum组合输入PCollection中的元素。
#结果PCollection（称为result）包含一个值：all的总和
# 输入PCollection中的元素.
pc = ...
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:combine_custom_average_execute
%}```

##### 4.2.4.4.组合和全局窗口

如果您的输入“PCollection”使用默认的全局窗口，则为默认值。行为是返回一个包含一个项目的“PCollection”。 那个物品的价值来自您指定的组合功能中的累加器
应用“Combine”。 例如，Beam提供sum组合函数返回零值（空输入的和），而最大组合函数返回最大或无限值。

如果输入为空，要使“Combine”返回一个空的“PCollection”当你应用'Combine`变换时，指定`.withoutDefaults`，如同
以下代码示例：

```java
PCollection<Integer> pc = ...;
PCollection<Integer> sum = pc.apply(
  Combine.globally(new Sum.SumIntegerFn()).withoutDefaults());
```
```py
pc = ...
sum = pc | beam.CombineGlobally(sum).without_defaults()
```

##### 4.2.4.5. 组合和非全局窗口

组合和非全局窗口如果您的“PCollection”使用任何非全局窗口函数，则Beam不会
提供默认行为。 何时必须指定以下选项之一应用“Combine”：

* 指定`.withoutDefaults`，其中输入中为空的窗口输出集合中的“PCollection”也将为空。
* 指定`.asSingletonView`，其中输出立即转换为 `PCollectionView`，它将为每个空窗口提供默认值
   用作侧输入。 通常只需要使用这个选项。您的管道的“Combine”的结果将被用作后面的一个侧面输入管道。

##### 4.2.4.6. 组合 keyed PCollection中的值

创建一个键控的PCollection（例如，通过使用`GroupByKey`转换），一个常见的模式是组合相关值的集合
将每个密钥合并成一个单独的合并值。 借鉴上一个例子`GroupByKey`，一个名为`groupingWords`的按键组合的'PCollection'看起来像这样：
```
  cat, [1,5,9]
  dog, [5,2]
  and, [1,2,6]
  jump, [3]
  tree, [2]
  ...
```

在上述“PCollection”中，每个元素都有一个字符串键（例如“cat”）和一个可迭代的整数的值（在第一个元素中，包含[1，
5,9]）。 如果我们的管道的下一个处理步骤组合了值（而不是单独考虑它们），您可以将整数的迭代组合到
创建一个单个合并的值以与每个键配对。 这种模式`GroupByKey`后跟合并的值的集合相当于
梁的组合PerKey变换。 组合功能您提供给Combine PerKey必须是关联缩减函数或“CombineFn”的子类。


```java
// PCollection按键分组，与每个键相关联的Double值组合为Double。
PCollection<KV<String, Double>> salesRecords = ...;
PCollection<KV<String, Double>> totalSalesPerPerson =
  salesRecords.apply(Combine.<String, Double, Double>perKey(
    new Sum.SumDoubleFn()));

// 组合值与每个键的原始集合值不同。 PCollection有
//String类型的键和Integer类型的值，组合值是Double
PCollection<KV<String, Integer>> playerAccuracy = ...;
PCollection<KV<String, Double>> avgAccuracyPerPlayer =
  playerAccuracy.apply(Combine.<String, Integer, Double>perKey(
    new MeanInts())));
```
```py
# PCollection按键和与每个键相关联的数值分组
# 被平均成一个浮点数。
player_accuracies = ...
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:combine_per_key
%}
```

#### 4.2.5. Flatten

<span class="language-java">[`Flatten`]({{ site.baseurl }}/documentation/sdks/javadoc/{{ site.release_latest }}/index.html?org/apache/beam/sdk/transforms/Flatten.html)</span>
<span class="language-py">[`Flatten`](https://github.com/apache/beam/blob/master/sdks/python/apache_beam/transforms/core.py)</span> 和
是用于存储相同数据类型的“PCollection”对象的Beam变换。“Flatten”将多个“PCollection”对象合并成一个逻辑
`PCollection`。

以下示例显示如何应用“Flatten”变换来合并多个`PCollection`对象。

```java
// Flatten接收给定类型的PCollection对象的PCollectionList。
// 返回单个PCollection，它包含该列表中PCollection对象中的所有元素。
PCollection<String> pc1 = ...;
PCollection<String> pc2 = ...;
PCollection<String> pc3 = ...;
PCollectionList<String> collections = PCollectionList.of(pc1).and(pc2).and(pc3);

PCollection<String> merged = collections.apply(Flatten.<String>pCollections());
```

```py
# Flatten需要一个PCollection对象的元组.
# 返回一个包含所有元素的PCollection
{%
github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:model_multiple_pcollections_flatten
%}
```

##### 4.2.5.1. 合并集合中的数据编码

默认情况下，输出“PCollection”的编码器与编码器相同第一个“PCollection”在输入“PCollectionList”中。 但是，输入
“PCollection”对象可以分别使用不同的编码器，只要它们都包含在您选择的语言中相同的数据类型e.

##### 4.2.5.2.合并窗口集合

当使用“Flatten”来合并具有窗口的“PCollection”对象应用策略，您要合并的所有“PCollection”对象必须使用兼容的窗口策略和窗口大小。 例如，所有的
您合并的集合必须全部使用（假设）相同的5分钟固定的窗户或每30秒钟开始的4分钟滑动窗口。

如果您的管道尝试使用“Flatten”将“PCollection”对象合并不兼容的窗口，Beam会生成一个“IllegalStateException”错误
管道建成。

#### 4.2.6. Partition

<span class="language-java">[`Partition`]({{ site.baseurl }}/documentation/sdks/javadoc/{{ site.release_latest }}/index.html?org/apache/beam/sdk/transforms/Partition.html)</span>
<span class="language-py">[`Partition`](https://github.com/apache/beam/blob/master/sdks/python/apache_beam/transforms/core.py)</span>
是用于存储相同数据的“PCollection”对象的beam变换类型。 “分区”将一个“PCollection”分成一个固定数量较小的
集合。

`Partition`根据分区划分“PCollection”的元素你提供的功能 分区函数包含逻辑
确定如何将输入“PCollection”的元素分割成每个元素导致分区`PCollection`。 必须确定分区数
在图形构建时间。 例如，您可以传递分区数作为运行时的命令行选项（将用于构建您的命令行选项）
管道图），但是您无法确定分区的数量中间流水线（根据您的流水线图构建后计算的数据，
例如）。

以下示例将“PCollection”分成百分位组。

```java
// 提供具有所需数量的结果分区的int值，以及代表该分区的PartitionFn
// 分区功能。 在这个例子中，我们定义了PartitionFn。 返回一个PCollectionList
//将每个生成的分区包含为单个PCollection对象。
PCollection<Student> students = ...;
// 将学生分成10个分区，百分位数：
PCollectionList<Student> studentsByPercentile =
    students.apply(Partition.of(10, new PartitionFn<Student>() {
        public int partitionFor(Student student, int numPartitions) {
            return student.getPercentile()  // 0..99
                 * numPartitions / 100;
        }}));

//您可以使用get方法从PCollectionList中提取每个分区，如下所示：
PCollection<Student> fortiethPercentile = studentsByPercentile.get(4);
```
```py
# 提供具有所需数量的结果分区的int值，以及分区函数（在本示例中为partition_fn）。
#将包含每个生成的分区的PCollection对象的元组返回为单独的PCollection对象。
students = ...
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:model_multiple_pcollections_partition
%}

# 您可以从PCollection对象的元组中提取每个分区，如下所示：
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:model_multiple_pcollections_partition_40th
%}
```

### 4.3.为Beam转换编写用户代码的要求

当您建立一个Beam变换的用户代码时，您应该记住执行的分布性质。 例如，可能有很多副本
功能运行在许多不同的机器上并行，这些副本功能独立，不与任何通信或共享状态
其他副本。 根据管道运行和处理后端您选择您的管道，您的用户代码功能的每个副本可能会重试或
运行多次。 因此，你应该谨慎的包括诸如此类的东西状态依赖于您的用户代码。

一般来说，您的用户代码至少必须满足以下要求：

* 您的函数对象必须是**可序列化**。
* 你的函数对象必须是**线程兼容的**，并注意* beam SDK不是线程安全的*。

此外，建议您使您的函数对象**幂等**。

> **Note:** 这些要求适用于“DoFn”（一个函数对象）的子类与[ParDo]（＃pardo）变换一起使用），CombineFn（与[Combine]（＃combine）变换一起使用的函数对象）和“WindowFn”（函数对象
与[Window]（＃windowing）变换一起使用）。

#### 4.3.1. 序列化

您提供给转换的任何函数对象必须是**完全可序列化的**。这是因为函数的副本需要序列化并传输到
处理群集中的远程工作人员。 用户代码的基类，如“DoFn”，“CombineFn”和“WindowFn”，已经实现了“Serializable”;
但是，您的子类不能添加任何不可序列化的成员。

你应该记住的其他序列化性因素有：

*函数对象中的瞬态字段是*不*传输给工作者实例，因为它们不是自动序列化的。
*序列化前避免加载大量数据的字段。
*您的函数对象的个别实例不能共享数据。
*函数对象在应用后会变得无效。
* 通过使用匿名方式内联函数对象时要小心内部类实例。 在非静态上下文中，您的内部类实例将会
   隐含地包含一个指向封闭类和该类的状态的指针。封闭类也将被序列化，因此也是相同的考虑因素
   适用于函数对象本身也适用于这个外部类。

#### 4.3.2.线程兼容

你的函数对象应该是线程兼容的。 你的功能的每个实例对象由worker实例上的单个线程访问，除非你
明确创建自己的线程。 但是请注意，** Beam SDK不是线程安全的**。 如果您在用户代码中创建自己的线程，则必须
提供自己的同步。 注意你的函数中的静态成员对象不会传递给worker实例，并且您的多个实例
函数可以从不同的线程访问。

#### 4.3.3. 幂等

建议你使你的函数对象的幂等 - 就是这样可以根据需要经常重复或重试，而不会导致意外的一面
效果。 光束模型不保证您的次数用户代码可能被调用或重试; 因此，保持你的功能对象
权力保持你的管道的输出确定性，你的转换'行为更可预测，更容易调试。

### 4.4. Side inputs

除了主输入“PCollection”之外，还可以提供额外的输入以侧面输入的形式转换为“ParDo”。 侧面输入是额外的
输入您的“DoFn”可以在每次处理输入中的元素时访问`PCollection`。 当您指定侧面输入时，您将创建一个其他视图
在“ParDo”变换的“DoFn”中可以读取数据，同时处理每个元素

如果您的ParDo需要在其中注入额外的数据，则侧面输入是有用的处理输入“PCollection”中的每个元素，但附加数据需要在运行时确定（而不是硬编码）。 这样的价值可能是
由输入数据确定，或取决于您的管道的不同分支。


#### 4.4.1.将side inputs传递给ParDo

```java
  // 通过调用.withSideInputs将边输入传递给ParDo转换。
  // 在DoFn中，使用DoFn.ProcessContext.sideInput方法访问边输入。

  //输入PCollection到ParDo
  PCollection<String> words = ...;

  // 我们将组合成单个值的单词长度的PCollection
  PCollection<Integer> wordLengths = ...; // Singleton PCollection

  // 使用Combine.globally和View.asSingleton从wordLengths创建一个单例PCollectionView。
  final PCollectionView<Integer> maxWordLengthCutOffView =
     wordLengths.apply(Combine.globally(new Max.MaxIntFn()).asSingletonView());


  //应用将MaxWordLengthCutOffView作为边输入的ParDo
  PCollection<String> wordsBelowCutOff =
  words.apply(ParDo
      .of(new DoFn<String, String>() {
          public void processElement(ProcessContext c) {
            String word = c.element();
            // In our DoFn, access the side input.
            int lengthCutOff = c.sideInput(maxWordLengthCutOffView);
            if (word.length() <= lengthCutOff) {
              c.output(word);
            }
          }
      }).withSideInputs(maxWordLengthCutOffView)
  );
```
```py
# 侧面输入可作为DoFn进程方法中的额外参数或Map / FlatMap的可调用
# 可选，位置和关键字参数都受支持。 延期的参数被展开
# 实际值。 例如，在管道构建时使用pvalue.AsIteor（pcoll）导致可迭代
# 将pcoll的实际元素传递到每个进程调用中。 在这个例子中，inside input是
# 传递给FlatMap变换作为额外的参数，并由filter_using_length消耗。
words = ...
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:model_pardo_side_input
%}

# 我们还可以将边输入传递给ParDo转换，这将被传递给它的进程方法
# 进程方法的前两个参数将是self和eement

{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:model_pardo_side_input_dofn
%}
...
```

#### 4.4.2. Side inputs 和 窗口
一个窗口的“PCollection”可能是无限的，因此不能被压缩成一个单值（或单集合类）。 当你创建一个“PCollectionView”
一个窗口的“PCollection”，“PCollectionView”表示单个实体每个窗口（每个窗口一个单例，每个窗口一个列表等）。

Beam 使用窗口作为主输入元素来查找适当的窗口为侧面输入元素。 光束投影主输入元素的窗口
进入侧面输入的窗口设置，然后使用侧面输入结果窗口。 如果主输入和侧输入具有相同的窗口，则投影提供了确切的对应窗口。 但是，如果输入有不同的窗口，光束使用投影来选择最合适的一面输入窗口。


例如，如果使用固定时间窗口打开主输入分钟，侧面输入使用一小时的固定时间窗口，
Beam将主输入窗口投影到侧面输入窗口集合上从适当的时长侧输入窗口中选择边输入值。

如果主输入元素存在于多个窗口中，那么`processElement`被调用多次，每次窗口一次。 每个调用`processElement`
为主输入元素投影“当前”窗口，从而可能提供每次侧面输入的不同视图。

如果侧面输入有多个触发发生，则Beam将使用该值最新的触发器。 如果您使用侧面输入，这是特别有用的
单个全局窗口并指定触发器。

### 4.5.附加输出

而“ParDo”总是产生一个主输出“PCollection”（作为返回值）从`apply`），你也可以让你的`ParDo`产生任何数量的附加值输出“PCollection”。 如果你选择有多个输出，你的“ParDo”
返回捆绑的所有输出“PCollection”（包括主输出）

#### 4.5.1.多个输出的标签

```java
// 要将元素发送到多个输出PCollections，请创建一个TupleTag对象来标识每个集合
//您的ParDo生产。 例如，如果您的ParDo产生三个输出PCollections（主输出和两个附加输出），则必须创建三个TupleTag。 以下示例代码显示了如何使用三个输出PCollections为ParDo创建TupleTag。

  // 输入PCollection到我们的ParDo.
  PCollection<String> words = ...;

  // ParDo将过滤长度低于截止值的字，并将其添加到
  主要输出PCollection <String>.
  //如果一个单词高于截止值，ParDo将会将单词长度添加到输出PCollection <Integer>
  // 如果一个单词高于截止值，ParDo将会将单词长度添加到输出PCollection <Integer>
  final int wordLengthCutOff = 10;

  // 创建三个TupleTags，每个输出PCollection一个
  // 包含长度截止字下的单词的输出
  final TupleTag<String> wordsBelowCutOffTag =
      new TupleTag<String>(){};
  // 包含字长的输出.
  final TupleTag<Integer> wordLengthsAboveCutOffTag =
      new TupleTag<Integer>(){};
  //包含“MARKER”字的输出
  final TupleTag<String> markedWordsTag =
      new TupleTag<String>(){};

//将输出标签传递给ParDo：
//为每个ParDo输出指定TupleTags后，通过调用将标签传递给ParDo
// .withOutputTags 首先传递主输出的标签，然后传递任何其他输出的标签
// 在TupleTagList中。 基于我们前面的例子，我们通过了三个TupleTags为我们的三个输出
// PCollections到我们的ParDo  请注意，所有输出（包括主输出PCollection）都捆绑在返回的PCollectionTuple中。

  PCollectionTuple results =
      words.apply(ParDo
          .of(new DoFn<String, String>() {
            // DoFn continues here.
            ...
          })
          // Specify the tag for the main output.
          .withOutputTags(wordsBelowCutOffTag,
          // Specify the tags for the two additional outputs as a TupleTagList.
                          TupleTagList.of(wordLengthsAboveCutOffTag)
                                      .and(markedWordsTag)));
```

```py
# 要将元素发送到多个输出PCollections，请在ParDo上调用with_outputs（），并指定
输出的预期标签。 with_outputs（）返回一个DoOutputsTuple对象。 标签中指定
#with_outputs是返回的DoOutputsTuple对象的属性。 标签允许访问
＃对应输出PCollections。

{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:model_pardo_with_tagged_outputs
%}

＃结果也是可迭代的，按照与标签传递给with_outputs（）的顺序相同的顺序排列，主标签（如果指定的话）

{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:model_pardo_with_tagged_outputs_iter
%}```

#### 4.5.2.发送到DoFn中的多个输出

```java
// 在ParDo的DoFn中，您可以通过传入一个元素到特定的输出PCollection 调用ProcessContext.output时适当的TupleTag
//在您的ParDo之后，从返回的PCollectionTuple中提取产生的输出PCollections
// 基于前面的例子，这显示了DoFn发送到主输出和两个附加输出

  .of(new DoFn<String, String>() {
     public void processElement(ProcessContext c) {
       String word = c.element();
       if (word.length() <= wordLengthCutOff) {
         // Emit short word to the main output.
         // In this example, it is the output with tag wordsBelowCutOffTag.
         c.output(word);
       } else {
         // Emit long word length to the output with tag wordLengthsAboveCutOffTag.
         c.output(wordLengthsAboveCutOffTag, word.length());
       }
       if (word.startsWith("MARKER")) {
         // Emit word to the output with tag markedWordsTag.
         c.output(markedWordsTag, word);
       }
     }}));
```

```py
#在ParDo的DoFn中，您可以通过包装值和输出标签（str）将元素发送到特定的输出。
＃使用pvalue.OutputValue包装器类。
＃根据前面的例子，这显示了DoFn发送到主输出和两个附加输出。

{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:model_pardo_emitting_values_on_tagged_outputs
%}

#在Map和FlatMap中也可以生成多个输出
# 以下是使用FlatMap的示例，并显示标签不需要提前指定

{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:model_pardo_with_undeclared_outputs
%}```

### 4.6.复合变换

变换可以具有嵌套结构，复杂变换执行多个简单的变换（如多个“ParDo”，“Combine”，“
`GroupByKey`，甚至其他复合变换）。 调用这些转换复合变换。 将多个变换嵌入到单个复合中
转换可以使您的代码更加模块化，更易于理解。

Beam SDK包含许多有用的复合转换。 请参阅API
转换列表的参考页面：
  * [Pre-written Beam transforms for Java]({{ site.baseurl }}/documentation/sdks/javadoc/{{ site.release_latest }}/index.html?org/apache/beam/sdk/transforms/package-summary.html)
  * [Pre-written Beam transforms for Python]({{ site.baseurl }}/documentation/sdks/pydoc/{{ site.release_latest }}/apache_beam.transforms.html)

#### 4.6.1.一个复合变换示例

The `CountWords` transform in the [WordCount example program]({{ site.baseurl }}/get-started/wordcount-example/)
是复合变换的例子。 “CountWords”是一个“PTransform”子类它由多个嵌套变换组成。

在“扩展”方法中，“CountWords”转换应用以下变换操作：

  1. 它在文本行的输入“PCollection”上应用了一个“ParDo”，产生单个单词的输出“PCollection”.
  2. 它在“PCollection”上应用了Beam SDK库转换“Count”单词，生成键/值对的“PCollection”。 每个键代表文字中的单词，每个值表示该单词的次数
      出现在原始数据中。

请注意，这也是嵌套复合转换的示例，如“Count”本身就是复合变换。

您的复合变换的参数和返回值必须与初始值相匹配输入类型和最终返回类型为整个变换，即使是
变换的中间数据更改多次。

```java
  public static class CountWords extends PTransform<PCollection<String>,
      PCollection<KV<String, Long>>> {
    @Override
    public PCollection<KV<String, Long>> expand(PCollection<String> lines) {

      //将文字行转换成单词.
      PCollection<String> words = lines.apply(
          ParDo.of(new ExtractWordsFn()));

      // 计算每个单词发生的次数
      PCollection<KV<String, Long>> wordCounts =
          words.apply(Count.<String>perElement());

      return wordCounts;
    }
  }
```

```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:pipeline_monitoring_composite
%}```

#### 4.6.2.创建一个复合变换

要创建自己的复合变换，创建一个“PTransform”的子类并覆盖`expand`方法来指定实际的处理逻辑。
然后，您可以像使用内置的转换一样使用此转换Beam SDK。

{language-java}
对于“PTransform”类类型参数，可以传递“PCollection”类型您的变换将作为输入，并作为输出生成。 要多了
“PCollection”作为输入，或产生多个“PCollection”作为输出，使用一个
的相关类型参数的多种收集类型。

以下代码示例显示了如何声明接受的“PTransform”
`PCollection``````````为输入，输出`PCollection````teger`s：

```java
  static class ComputeWordLengths
    extends PTransform<PCollection<String>, PCollection<Integer>> {
    ...
  }
```

```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:model_composite_transform
%}```

在“PTransform”子类中，您需要覆盖“扩展”方法。`expand`方法是添加“PTransform”的处理逻辑的地方。
您的“expand”的覆盖必须接受适当的输入类型“PCollection”作为参数，并指定输出“PCollection”作为返回值
值。

以下代码示例显示如何覆盖`expand` 在前面的例子中声明的`ComputeWordLengths`类：

```java
  static class ComputeWordLengths
      extends PTransform<PCollection<String>, PCollection<Integer>> {
    @Override
    public PCollection<Integer> expand(PCollection<String>) {
      ...
      // transform logic goes here
      ...
    }
```

```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:model_composite_transform
%}```

只要你在“PTransform”子类中覆盖`expand`方法即可接受适当的输入“PCollection”并返回相应的输出“PCollection”，您可以根据需要添加多个转换。 这些
变换可以包括核心变换，复合变换或变换包含在Beam SDK库中。

**Note:**“PTransform”的`expand`方法不是要调用的直接由用户进行转换。 而应该调用`apply`方法在“PCollection”本身，以变换为参数。 这允许
转换为嵌套在您的管道的结构中。

#### 4.6.3.PTransform风格指南

The [PTransform Style Guide]({{ site.baseurl }}/contribute/ptransform-style-guide/)
包含此处未包含的其他信息，如风格指南，记录和测试指导，以及语言特定的考虑。 导游
是一个有用的起点，当你想写新的复合PTransforms。

## 5. Pipeline I/O

When you create a pipeline, you often need to read data from some external
source, such as a file in external data sink or a database. Likewise, you may
want your pipeline to output its result data to a similar external data sink.
Beam provides read and write transforms for a [number of common data storage
types]({{ site.baseurl }}/documentation/io/built-in/). If you want your pipeline
to read from or write to a data storage format that isn't supported by the
built-in transforms, you can [implement your own read and write
transforms]({{site.baseurl }}/documentation/io/io-toc/).

### 5.1. Reading input data

Read transforms read data from an external source and return a `PCollection`
representation of the data for use by your pipeline. You can use a read
transform at any point while constructing your pipeline to create a new
`PCollection`, though it will be most common at the start of your pipeline.

```java
PCollection<String> lines = p.apply(TextIO.read().from("gs://some/inputData.txt"));
```

```py
lines = pipeline | beam.io.ReadFromText('gs://some/inputData.txt')
```

### 5.2. Writing output data

Write transforms write the data in a `PCollection` to an external data source.
You will most often use write transforms at the end of your pipeline to output
your pipeline's final results. However, you can use a write transform to output
a `PCollection`'s data at any point in your pipeline.

```java
output.apply(TextIO.write().to("gs://some/outputData"));
```

```py
output | beam.io.WriteToText('gs://some/outputData')
```

### 5.3. File-based input and output data

#### 5.3.1. Reading from multiple locations

Many read transforms support reading from multiple input files matching a glob
operator you provide. Note that glob operators are filesystem-specific and obey
filesystem-specific consistency models. The following TextIO example uses a glob
operator (\*) to read all matching input files that have prefix "input-" and the
suffix ".csv" in the given location:

```java
p.apply(“ReadFromText”,
    TextIO.read().from("protocol://my_bucket/path/to/input-*.csv");
```

```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:model_pipelineio_read
%}
```

To read data from disparate sources into a single `PCollection`, read each one
independently and then use the [Flatten](#flatten) transform to create a single
`PCollection`.

#### 5.3.2. Writing to multiple output files

For file-based output data, write transforms write to multiple output files by
default. When you pass an output file name to a write transform, the file name
is used as the prefix for all output files that the write transform produces.
You can append a suffix to each output file by specifying a suffix.

The following write transform example writes multiple output files to a
location. Each file has the prefix "numbers", a numeric tag, and the suffix
".csv".

```java
records.apply("WriteToText",
    TextIO.write().to("protocol://my_bucket/path/to/numbers")
                .withSuffix(".csv"));
```

```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:model_pipelineio_write
%}
```

### 5.4. Beam-provided I/O transforms

See the [Beam-provided I/O Transforms]({{site.baseurl }}/documentation/io/built-in/)
page for a list of the currently available I/O transforms.

## 6. Data encoding and type safety

When Beam runners execute your pipeline, they often need to materialize the
intermediate data in your `PCollection`s, which requires converting elements to
and from byte strings. The Beam SDKs use objects called `Coder`s to describe how
the elements of a given `PCollection` may be encoded and decoded.

> Note that coders are unrelated to parsing or formatting data when interacting
> with external data sources or sinks. Such parsing or formatting should
> typically be done explicitly, using transforms such as `ParDo` or
> `MapElements`.

{:.language-java}
In the Beam SDK for Java, the type `Coder` provides the methods required for
encoding and decoding data. The SDK for Java provides a number of Coder
subclasses that work with a variety of standard Java types, such as Integer,
Long, Double, StringUtf8 and more. You can find all of the available Coder
subclasses in the [Coder package](https://github.com/apache/beam/tree/master/sdks/java/core/src/main/java/org/apache/beam/sdk/coders).

{:.language-py}
In the Beam SDK for Python, the type `Coder` provides the methods required for
encoding and decoding data. The SDK for Python provides a number of Coder
subclasses that work with a variety of standard Python types, such as primitive
types, Tuple, Iterable, StringUtf8 and more. You can find all of the available
Coder subclasses in the
[apache_beam.coders](https://github.com/apache/beam/tree/master/sdks/python/apache_beam/coders)
package.

> Note that coders do not necessarily have a 1:1 relationship with types. For
> example, the Integer type can have multiple valid coders, and input and output
> data can use different Integer coders. A transform might have Integer-typed
> input data that uses BigEndianIntegerCoder, and Integer-typed output data that
> uses VarIntCoder.

### 6.1. Specifying coders

The Beam SDKs require a coder for every `PCollection` in your pipeline. In most
cases, the Beam SDK is able to automatically infer a `Coder` for a `PCollection`
based on its element type or the transform that produces it, however, in some
cases the pipeline author will need to specify a `Coder` explicitly, or develop
a `Coder` for their custom type.

{:.language-java}
You can explicitly set the coder for an existing `PCollection` by using the
method `PCollection.setCoder`. Note that you cannot call `setCoder` on a
`PCollection` that has been finalized (e.g. by calling `.apply` on it).

{:.language-java}
You can get the coder for an existing `PCollection` by using the method
`getCoder`. This method will fail with an `IllegalStateException` if a coder has
not been set and cannot be inferred for the given `PCollection`.

Beam SDKs use a variety of mechanisms when attempting to automatically infer the
`Coder` for a `PCollection`.

{:.language-java}
Each pipeline object has a `CoderRegistry`. The `CoderRegistry` represents a
mapping of Java types to the default coders that the pipeline should use for
`PCollection`s of each type.

{:.language-py}
The Beam SDK for Python has a `CoderRegistry` that represents a mapping of
Python types to the default coder that should be used for `PCollection`s of each
type.

{:.language-java}
By default, the Beam SDK for Java automatically infers the `Coder` for the
elements of a `PCollection` produced by a `PTransform` using the type parameter
from the transform's function object, such as `DoFn`. In the case of `ParDo`,
for example, a `DoFn<Integer, String>` function object accepts an input element
of type `Integer` and produces an output element of type `String`. In such a
case, the SDK for Java will automatically infer the default `Coder` for the
output `PCollection<String>` (in the default pipeline `CoderRegistry`, this is
`StringUtf8Coder`).

{:.language-py}
By default, the Beam SDK for Python automatically infers the `Coder` for the
elements of an output `PCollection` using the typehints from the transform's
function object, such as `DoFn`. In the case of `ParDo`, for example a `DoFn`
with the typehints `@beam.typehints.with_input_types(int)` and
`@beam.typehints.with_output_types(str)` accepts an input element of type int
and produces an output element of type str. In such a case, the Beam SDK for
Python will automatically infer the default `Coder` for the output `PCollection`
(in the default pipeline `CoderRegistry`, this is `BytesCoder`).

> NOTE: If you create your `PCollection` from in-memory data by using the
> `Create` transform, you cannot rely on coder inference and default coders.
> `Create` does not have access to any typing information for its arguments, and
> may not be able to infer a coder if the argument list contains a value whose
> exact run-time class doesn't have a default coder registered.

{:.language-java}
When using `Create`, the simplest way to ensure that you have the correct coder
is by invoking `withCoder` when you apply the `Create` transform.

### 6.2. Default coders and the CoderRegistry

Each Pipeline object has a `CoderRegistry` object, which maps language types to
the default coder the pipeline should use for those types. You can use the
`CoderRegistry` yourself to look up the default coder for a given type, or to
register a new default coder for a given type.

`CoderRegistry` contains a default mapping of coders to standard
<span class="language-java">Java</span><span class="language-py">Python</span>
types for any pipeline you create using the Beam SDK for
<span class="language-java">Java</span><span class="language-py">Python</span>.
The following table shows the standard mapping:

{:.language-java}
<table>
  <thead>
    <tr class="header">
      <th>Java Type</th>
      <th>Default Coder</th>
    </tr>
  </thead>
  <tbody>
    <tr class="odd">
      <td>Double</td>
      <td>DoubleCoder</td>
    </tr>
    <tr class="even">
      <td>Instant</td>
      <td>InstantCoder</td>
    </tr>
    <tr class="odd">
      <td>Integer</td>
      <td>VarIntCoder</td>
    </tr>
    <tr class="even">
      <td>Iterable</td>
      <td>IterableCoder</td>
    </tr>
    <tr class="odd">
      <td>KV</td>
      <td>KvCoder</td>
    </tr>
    <tr class="even">
      <td>List</td>
      <td>ListCoder</td>
    </tr>
    <tr class="odd">
      <td>Map</td>
      <td>MapCoder</td>
    </tr>
    <tr class="even">
      <td>Long</td>
      <td>VarLongCoder</td>
    </tr>
    <tr class="odd">
      <td>String</td>
      <td>StringUtf8Coder</td>
    </tr>
    <tr class="even">
      <td>TableRow</td>
      <td>TableRowJsonCoder</td>
    </tr>
    <tr class="odd">
      <td>Void</td>
      <td>VoidCoder</td>
    </tr>
    <tr class="even">
      <td>byte[ ]</td>
      <td>ByteArrayCoder</td>
    </tr>
    <tr class="odd">
      <td>TimestampedValue</td>
      <td>TimestampedValueCoder</td>
    </tr>
  </tbody>
</table>

{:.language-py}
<table>
  <thead>
    <tr class="header">
      <th>Python Type</th>
      <th>Default Coder</th>
    </tr>
  </thead>
  <tbody>
    <tr class="odd">
      <td>int</td>
      <td>VarIntCoder</td>
    </tr>
    <tr class="even">
      <td>float</td>
      <td>FloatCoder</td>
    </tr>
    <tr class="odd">
      <td>str</td>
      <td>BytesCoder</td>
    </tr>
    <tr class="even">
      <td>bytes</td>
      <td>StrUtf8Coder</td>
    </tr>
    <tr class="odd">
      <td>Tuple</td>
      <td>TupleCoder</td>
    </tr>
  </tbody>
</table>

#### 6.2.1. Looking up a default coder

{:.language-java}
You can use the method `CoderRegistry.getDefaultCoder` to determine the default
Coder for a Java type. You can access the `CoderRegistry` for a given pipeline
by using the method `Pipeline.getCoderRegistry`. This allows you to determine
(or set) the default Coder for a Java type on a per-pipeline basis: i.e. "for
this pipeline, verify that Integer values are encoded using
`BigEndianIntegerCoder`."

{:.language-py}
You can use the method `CoderRegistry.get_coder` to determine the default Coder
for a Python type. You can use `coders.registry` to access the `CoderRegistry`.
This allows you to determine (or set) the default Coder for a Python type.

#### 6.2.2. Setting the default coder for a type

To set the default Coder for a
<span class="language-java">Java</span><span class="language-py">Python</span>
type for a particular pipeline, you obtain and modify the pipeline's
`CoderRegistry`. You use the method
<span class="language-java">`Pipeline.getCoderRegistry`</span>
<span class="language-py">`coders.registry`</span>
to get the `CoderRegistry` object, and then use the method
<span class="language-java">`CoderRegistry.registerCoder`</span>
<span class="language-py">`CoderRegistry.register_coder`</span>
to register a new `Coder` for the target type.

The following example code demonstrates how to set a default Coder, in this case
`BigEndianIntegerCoder`, for
<span class="language-java">Integer</span><span class="language-py">int</span>
values for a pipeline.

```java
PipelineOptions options = PipelineOptionsFactory.create();
Pipeline p = Pipeline.create(options);

CoderRegistry cr = p.getCoderRegistry();
cr.registerCoder(Integer.class, BigEndianIntegerCoder.class);
```

```py
apache_beam.coders.registry.register_coder(int, BigEndianIntegerCoder)
```

#### 6.2.3. Annotating a custom data type with a default coder

{:.language-java}
If your pipeline program defines a custom data type, you can use the
`@DefaultCoder` annotation to specify the coder to use with that type. For
example, let's say you have a custom data type for which you want to use
`SerializableCoder`. You can use the `@DefaultCoder` annotation as follows:

```java
@DefaultCoder(AvroCoder.class)
public class MyCustomDataType {
  ...
}
```

{:.language-java}
If you've created a custom coder to match your data type, and you want to use
the `@DefaultCoder` annotation, your coder class must implement a static
`Coder.of(Class<T>)` factory method.

```java
public class MyCustomCoder implements Coder {
  public static Coder<T> of(Class<T> clazz) {...}
  ...
}

@DefaultCoder(MyCustomCoder.class)
public class MyCustomDataType {
  ...
}
```

{:.language-py}
The Beam SDK for Python does not support annotating data types with a default
coder. If you would like to set a default coder, use the method described in the
previous section, *Setting the default coder for a type*.

## 7. Windowing

Windowing subdivides a `PCollection` according to the timestamps of its
individual elements. Transforms that aggregate multiple elements, such as
`GroupByKey` and `Combine`, work implicitly on a per-window basis — they process
each `PCollection` as a succession of multiple, finite windows, though the
entire collection itself may be of unbounded size.

A related concept, called **triggers**, determines when to emit the results of
aggregation as unbounded data arrives. You can use triggers to refine the
windowing strategy for your `PCollection`. Triggers allow you to deal with
late-arriving data or to provide early results. See the [triggers](#triggers)
section for more information.

### 7.1. Windowing basics

Some Beam transforms, such as `GroupByKey` and `Combine`, group multiple
elements by a common key. Ordinarily, that grouping operation groups all of the
elements that have the same key within the entire data set. With an unbounded
data set, it is impossible to collect all of the elements, since new elements
are constantly being added and may be infinitely many (e.g. streaming data). If
you are working with unbounded `PCollection`s, windowing is especially useful.

In the Beam model, any `PCollection` (including unbounded `PCollection`s) can be
subdivided into logical windows. Each element in a `PCollection` is assigned to
one or more windows according to the `PCollection`'s windowing function, and
each individual window contains a finite number of elements. Grouping transforms
then consider each `PCollection`'s elements on a per-window basis. `GroupByKey`,
for example, implicitly groups the elements of a `PCollection` by _key and
window_.

**Caution:** Beam's default windowing behavior is to assign all elements of a
`PCollection` to a single, global window and discard late data, _even for
unbounded `PCollection`s_. Before you use a grouping transform such as
`GroupByKey` on an unbounded `PCollection`, you must do at least one of the
following:
 * Set a non-global windowing function. See [Setting your PCollection's
   windowing function](#setting-your-pcollections-windowing-function).
 * Set a non-default [trigger](#triggers). This allows the global window to emit
   results under other conditions, since the default windowing behavior (waiting
   for all data to arrive) will never occur.

If you don't set a non-global windowing function or a non-default trigger for
your unbounded `PCollection` and subsequently use a grouping transform such as
`GroupByKey` or `Combine`, your pipeline will generate an error upon
construction and your job will fail.

#### 7.1.1. Windowing constraints

After you set the windowing function for a `PCollection`, the elements' windows
are used the next time you apply a grouping transform to that `PCollection`.
Window grouping occurs on an as-needed basis. If you set a windowing function
using the `Window` transform, each element is assigned to a window, but the
windows are not considered until `GroupByKey` or `Combine` aggregates across a
window and key. This can have different effects on your pipeline.  Consider the
example pipeline in the figure below:

![Diagram of pipeline applying windowing]({{ "/images/windowing-pipeline-unbounded.png" | prepend: site.baseurl }} "Pipeline applying windowing")

**Figure:** Pipeline applying windowing

In the above pipeline, we create an unbounded `PCollection` by reading a set of
key/value pairs using `KafkaIO`, and then apply a windowing function to that
collection using the `Window` transform. We then apply a `ParDo` to the the
collection, and then later group the result of that `ParDo` using `GroupByKey`.
The windowing function has no effect on the `ParDo` transform, because the
windows are not actually used until they're needed for the `GroupByKey`.
Subsequent transforms, however, are applied to the result of the `GroupByKey` --
data is grouped by both key and window.

#### 7.1.2. Using windowing with bounded PCollections

You can use windowing with fixed-size data sets in **bounded** `PCollection`s.
However, note that windowing considers only the implicit timestamps attached to
each element of a `PCollection`, and data sources that create fixed data sets
(such as `TextIO`) assign the same timestamp to every element. This means that
all the elements are by default part of a single, global window.

To use windowing with fixed data sets, you can assign your own timestamps to
each element. To assign timestamps to elements, use a `ParDo` transform with a
`DoFn` that outputs each element with a new timestamp (for example, the
[WithTimestamps]({{ site.baseurl }}/documentation/sdks/javadoc/{{ site.release_latest }}/index.html?org/apache/beam/sdk/transforms/WithTimestamps.html)
transform in the Beam SDK for Java).

To illustrate how windowing with a bounded `PCollection` can affect how your
pipeline processes data, consider the following pipeline:

![Diagram of GroupByKey and ParDo without windowing, on a bounded collection]({{ "/images/unwindowed-pipeline-bounded.png" | prepend: site.baseurl }} "GroupByKey and ParDo without windowing, on a bounded collection")

**Figure:** `GroupByKey` and `ParDo` without windowing, on a bounded collection.

In the above pipeline, we create a bounded `PCollection` by reading a set of
key/value pairs using `TextIO`. We then group the collection using `GroupByKey`,
and apply a `ParDo` transform to the grouped `PCollection`. In this example, the
`GroupByKey` creates a collection of unique keys, and then `ParDo` gets applied
exactly once per key.

Note that even if you don’t set a windowing function, there is still a window --
all elements in your `PCollection` are assigned to a single global window.

Now, consider the same pipeline, but using a windowing function:

![Diagram of GroupByKey and ParDo with windowing, on a bounded collection]({{ "/images/windowing-pipeline-bounded.png" | prepend: site.baseurl }} "GroupByKey and ParDo with windowing, on a bounded collection")

**Figure:** `GroupByKey` and `ParDo` with windowing, on a bounded collection.

As before, the pipeline creates a bounded `PCollection` of key/value pairs. We
then set a [windowing function](#setting-your-pcollections-windowing-function)
for that `PCollection`.  The `GroupByKey` transform groups the elements of the
`PCollection` by both key and window, based on the windowing function. The
subsequent `ParDo` transform gets applied multiple times per key, once for each
window.

### 7.2. Provided windowing functions

You can define different kinds of windows to divide the elements of your
`PCollection`. Beam provides several windowing functions, including:

*  Fixed Time Windows
*  Sliding Time Windows
*  Per-Session Windows
*  Single Global Window
*  Calendar-based Windows (not supported by the Beam SDK for Python)

You can also define your own `WindowFn` if you have a more complex need.

Note that each element can logically belong to more than one window, depending
on the windowing function you use. Sliding time windowing, for example, creates
overlapping windows wherein a single element can be assigned to multiple
windows.


#### 7.2.1. Fixed time windows

The simplest form of windowing is using **fixed time windows**: given a
timestamped `PCollection` which might be continuously updating, each window
might capture (for example) all elements with timestamps that fall into a five
minute interval.

A fixed time window represents a consistent duration, non overlapping time
interval in the data stream. Consider windows with a five-minute duration: all
of the elements in your unbounded `PCollection` with timestamp values from
0:00:00 up to (but not including) 0:05:00 belong to the first window, elements
with timestamp values from 0:05:00 up to (but not including) 0:10:00 belong to
the second window, and so on.

![Diagram of fixed time windows, 30s in duration]({{ "/images/fixed-time-windows.png" | prepend: site.baseurl }} "Fixed time windows, 30s in duration")

**Figure:** Fixed time windows, 30s in duration.

#### 7.2.2. Sliding time windows

A **sliding time window** also represents time intervals in the data stream;
however, sliding time windows can overlap. For example, each window might
capture five minutes worth of data, but a new window starts every ten seconds.
The frequency with which sliding windows begin is called the _period_.
Therefore, our example would have a window _duration_ of five minutes and a
_period_ of ten seconds.

Because multiple windows overlap, most elements in a data set will belong to
more than one window. This kind of windowing is useful for taking running
averages of data; using sliding time windows, you can compute a running average
of the past five minutes' worth of data, updated every ten seconds, in our
example.

![Diagram of sliding time windows, with 1 minute window duration and 30s window period]({{ "/images/sliding-time-windows.png" | prepend: site.baseurl }} "Sliding time windows, with 1 minute window duration and 30s window period")

**Figure:** Sliding time windows, with 1 minute window duration and 30s window
period.

#### 7.2.3. Session windows

A **session window** function defines windows that contain elements that are
within a certain gap duration of another element. Session windowing applies on a
per-key basis and is useful for data that is irregularly distributed with
respect to time. For example, a data stream representing user mouse activity may
have long periods of idle time interspersed with high concentrations of clicks.
If data arrives after the minimum specified gap duration time, this initiates
the start of a new window.

![Diagram of session windows with a minimum gap duration]({{ "/images/session-windows.png" | prepend: site.baseurl }} "Session windows, with a minimum gap duration")

**Figure:** Session windows, with a minimum gap duration. Note how each data key
has different windows, according to its data distribution.

#### 7.2.4. The single global window

By default, all data in a `PCollection` is assigned to the single global window,
and late data is discarded. If your data set is of a fixed size, you can use the
global window default for your `PCollection`.

You can use the single global window if you are working with an unbounded data set
(e.g. from a streaming data source) but use caution when applying aggregating
transforms such as `GroupByKey` and `Combine`. The single global window with a
default trigger generally requires the entire data set to be available before
processing, which is not possible with continuously updating data. To perform
aggregations on an unbounded `PCollection` that uses global windowing, you
should specify a non-default trigger for that `PCollection`.

### 7.3. Setting your PCollection's windowing function

You can set the windowing function for a `PCollection` by applying the `Window`
transform. When you apply the `Window` transform, you must provide a `WindowFn`.
The `WindowFn` determines the windowing function your `PCollection` will use for
subsequent grouping transforms, such as a fixed or sliding time window.

When you set a windowing function, you may also want to set a trigger for your
`PCollection`. The trigger determines when each individual window is aggregated
and emitted, and helps refine how the windowing function performs with respect
to late data and computing early results. See the [triggers](#triggers) section
for more information.

#### 7.3.1. Fixed-time windows

The following example code shows how to apply `Window` to divide a `PCollection`
into fixed windows, each one minute in length:

```java
    PCollection<String> items = ...;
    PCollection<String> fixed_windowed_items = items.apply(
        Window.<String>into(FixedWindows.of(Duration.standardMinutes(1))));
```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:setting_fixed_windows
%}
```

#### 7.3.2. Sliding time windows

The following example code shows how to apply `Window` to divide a `PCollection`
into sliding time windows. Each window is 30 minutes in length, and a new window
begins every five seconds:

```java
    PCollection<String> items = ...;
    PCollection<String> sliding_windowed_items = items.apply(
        Window.<String>into(SlidingWindows.of(Duration.standardMinutes(30)).every(Duration.standardSeconds(5))));
```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:setting_sliding_windows
%}
```

#### 7.3.3. Session windows

The following example code shows how to apply `Window` to divide a `PCollection`
into session windows, where each session must be separated by a time gap of at
least 10 minutes:

```java
    PCollection<String> items = ...;
    PCollection<String> session_windowed_items = items.apply(
        Window.<String>into(Sessions.withGapDuration(Duration.standardMinutes(10))));
```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:setting_session_windows
%}
```

Note that the sessions are per-key — each key in the collection will have its
own session groupings depending on the data distribution.

#### 7.3.4. Single global window

If your `PCollection` is bounded (the size is fixed), you can assign all the
elements to a single global window. The following example code shows how to set
a single global window for a `PCollection`:

```java
    PCollection<String> items = ...;
    PCollection<String> batch_items = items.apply(
        Window.<String>into(new GlobalWindows()));
```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:setting_global_window
%}
```

### 7.4. Watermarks and late data

In any data processing system, there is a certain amount of lag between the time
a data event occurs (the "event time", determined by the timestamp on the data
element itself) and the time the actual data element gets processed at any stage
in your pipeline (the "processing time", determined by the clock on the system
processing the element). In addition, there are no guarantees that data events
will appear in your pipeline in the same order that they were generated.

For example, let's say we have a `PCollection` that's using fixed-time
windowing, with windows that are five minutes long. For each window, Beam must
collect all the data with an _event time_ timestamp in the given window range
(between 0:00 and 4:59 in the first window, for instance). Data with timestamps
outside that range (data from 5:00 or later) belong to a different window.

However, data isn't always guaranteed to arrive in a pipeline in time order, or
to always arrive at predictable intervals. Beam tracks a _watermark_, which is
the system's notion of when all data in a certain window can be expected to have
arrived in the pipeline. Data that arrives with a timestamp after the watermark
is considered **late data**.

From our example, suppose we have a simple watermark that assumes approximately
30s of lag time between the data timestamps (the event time) and the time the
data appears in the pipeline (the processing time), then Beam would close the
first window at 5:30. If a data record arrives at 5:34, but with a timestamp
that would put it in the 0:00-4:59 window (say, 3:38), then that record is late
data.

Note: For simplicity, we've assumed that we're using a very straightforward
watermark that estimates the lag time. In practice, your `PCollection`'s data
source determines the watermark, and watermarks can be more precise or complex.

Beam's default windowing configuration tries to determines when all data has
arrived (based on the type of data source) and then advances the watermark past
the end of the window. This default configuration does _not_ allow late data.
[Triggers](#triggers) allow you to modify and refine the windowing strategy for
a `PCollection`. You can use triggers to decide when each individual window
aggregates and reports its results, including how the window emits late
elements.

#### 7.4.1. Managing late data

> **Note:** Managing late data is not supported in the Beam SDK for Python.

You can allow late data by invoking the `.withAllowedLateness` operation when
you set your `PCollection`'s windowing strategy. The following code example
demonstrates a windowing strategy that will allow late data up to two days after
the end of a window.

```java
    PCollection<String> items = ...;
    PCollection<String> fixed_windowed_items = items.apply(
        Window.<String>into(FixedWindows.of(Duration.standardMinutes(1)))
              .withAllowedLateness(Duration.standardDays(2)));
```

When you set `.withAllowedLateness` on a `PCollection`, that allowed lateness
propagates forward to any subsequent `PCollection` derived from the first
`PCollection` you applied allowed lateness to. If you want to change the allowed
lateness later in your pipeline, you must do so explictly by applying
`Window.configure().withAllowedLateness()`.

### 7.5. Adding timestamps to a PCollection's elements

An unbounded source provides a timestamp for each element. Depending on your
unbounded source, you may need to configure how the timestamp is extracted from
the raw data stream.

However, bounded sources (such as a file from `TextIO`) do not provide
timestamps. If you need timestamps, you must add them to your `PCollection`’s
elements.

You can assign new timestamps to the elements of a `PCollection` by applying a
[ParDo](#pardo) transform that outputs new elements with timestamps that you
set.

An example might be if your pipeline reads log records from an input file, and
each log record includes a timestamp field; since your pipeline reads the
records in from a file, the file source doesn't assign timestamps automatically.
You can parse the timestamp field from each record and use a `ParDo` transform
with a `DoFn` to attach the timestamps to each element in your `PCollection`.

```java
      PCollection<LogEntry> unstampedLogs = ...;
      PCollection<LogEntry> stampedLogs =
          unstampedLogs.apply(ParDo.of(new DoFn<LogEntry, LogEntry>() {
            public void processElement(ProcessContext c) {
              // Extract the timestamp from log entry we're currently processing.
              Instant logTimeStamp = extractTimeStampFromLogEntry(c.element());
              // Use ProcessContext.outputWithTimestamp (rather than
              // ProcessContext.output) to emit the entry with timestamp attached.
              c.outputWithTimestamp(c.element(), logTimeStamp);
            }
          }));
```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:setting_timestamp
%}
```

## 8. Triggers

> **NOTE:** This content applies only to the Beam SDK for Java. The Beam SDK for
> Python does not support triggers.

When collecting and grouping data into windows, Beam uses **triggers** to
determine when to emit the aggregated results of each window (referred to as a
*pane*). If you use Beam's default windowing configuration and [default
trigger](#the-default-trigger), Beam outputs the aggregated result when it
[estimates all data has arrived](#watermarks-and-late-data), and discards all
subsequent data for that window.

You can set triggers for your `PCollection`s to change this default behavior.
Beam provides a number of pre-built triggers that you can set:

*   **Event time triggers**. These triggers operate on the event time, as
    indicated by the timestamp on each data element. Beam's default trigger is
    event time-based.
*   **Processing time triggers**. These triggers operate on the processing time
    -- the time when the data element is processed at any given stage in the
    pipeline.
*   **Data-driven triggers**. These triggers operate by examining the data as it
    arrives in each window, and firing when that data meets a certain property.
    Currently, data-driven triggers only support firing after a certain number
    of data elements.
*   **Composite triggers**. These triggers combine multiple triggers in various
    ways.

At a high level, triggers provide two additional capabilities compared to simply
outputting at the end of a window:

*   Triggers allow Beam to emit early results, before all the data in a given
    window has arrived. For example, emitting after a certain amouint of time
    elapses, or after a certain number of elements arrives.
*   Triggers allow processing of late data by triggering after the event time
    watermark passes the end of the window.

These capabilities allow you to control the flow of your data and balance
between different factors depending on your use case:

*   **Completeness:** How important is it to have all of your data before you
    compute your result?
*   **Latency:** How long do you want to wait for data? For example, do you wait
    until you think you have all data? Do you process data as it arrives?
*   **Cost:** How much compute power/money are you willing to spend to lower the
    latency?

For example, a system that requires time-sensitive updates might use a strict
time-based trigger that emits a window every *N* seconds, valuing promptness
over data completeness. A system that values data completeness more than the
exact timing of results might choose to use Beam's default trigger, which fires
at the end of the window.

You can also set a trigger for an unbounded `PCollection` that uses a [single
global window for its windowing function](#windowing). This can be useful when
you want your pipeline to provide periodic updates on an unbounded data set —
for example, a running average of all data provided to the present time, updated
every N seconds or every N elements.

### 8.1. Event time triggers

The `AfterWatermark` trigger operates on *event time*. The `AfterWatermark`
trigger emits the contents of a window after the
[watermark](#watermarks-and-late-data) passes the end of the window, based on the
timestamps attached to the data elements. The watermark is a global progress
metric, and is Beam's notion of input completeness within your pipeline at any
given point. `AfterWatermark.pastEndOfWindow()` *only* fires when the watermark
passes the end of the window.

In addition, you can use `.withEarlyFirings(trigger)` and
`.withLateFirings(trigger)` to configure triggers that fire if your pipeline
receives data before or after the end of the window.

The following example shows a billing scenario, and uses both early and late
firings:

```java
  // Create a bill at the end of the month.
  AfterWatermark.pastEndOfWindow()
      // During the month, get near real-time estimates.
      .withEarlyFirings(
          AfterProcessingTime
              .pastFirstElementInPane()
              .plusDuration(Duration.standardMinutes(1))
      // Fire on any late data so the bill can be corrected.
      .withLateFirings(AfterPane.elementCountAtLeast(1))
```
```py
  # The Beam SDK for Python does not support triggers.
```

#### 8.1.1. The default trigger

The default trigger for a `PCollection` is based on event time, and emits the
results of the window when the Beam's watermark passes the end of the window,
and then fires each time late data arrives.

However, if you are using both the default windowing configuration and the
default trigger, the default trigger emits exactly once, and late data is
discarded. This is because the default windowing configuration has an allowed
lateness value of 0. See the Handling Late Data section for information about
modifying this behavior.

### 8.2. Processing time triggers

The `AfterProcessingTime` trigger operates on *processing time*. For example,
the `AfterProcessingTime.pastFirstElementInPane() ` trigger emits a window after
a certain amount of processing time has passed since data was received. The
processing time is determined by the system clock, rather than the data
element's timestamp.

The `AfterProcessingTime` trigger is useful for triggering early results from a
window, particularly a window with a large time frame such as a single global
window.

### 8.3. Data-driven triggers

Beam provides one data-driven trigger, `AfterPane.elementCountAtLeast()`. This
trigger works on an element count; it fires after the current pane has collected
at least *N* elements. This allows a window to emit early results (before all
the data has accumulated), which can be particularly useful if you are using a
single global window.

It is important to note that if, for example, you use `.elementCountAtLeast(50)`
and only 32 elements arrive, those 32 elements sit around forever. If the 32
elements are important to you, consider using [composite
triggers](#composite-triggers) to combine multiple conditions. This allows you
to specify multiple firing conditions such as “fire either when I receive 50
elements, or every 1 second”.

### 8.4. Setting a trigger

When you set a windowing function for a `PCollection` by using the `Window`
transform, you can also specify a trigger.

You set the trigger(s) for a `PCollection` by invoking the method
`.triggering()` on the result of your `Window.into()` transform, as follows:

```java
  PCollection<String> pc = ...;
  pc.apply(Window.<String>into(FixedWindows.of(1, TimeUnit.MINUTES))
                               .triggering(AfterProcessingTime.pastFirstElementInPane()
                                                              .plusDelayOf(Duration.standardMinutes(1)))
                               .discardingFiredPanes());
```
```py
  # The Beam SDK for Python does not support triggers.
```

This code sample sets a time-based trigger for a `PCollection`, which emits
results one minute after the first element in that window has been processed.
The last line in the code sample, `.discardingFiredPanes()`, is the window's
**accumulation mode**.

#### 8.4.1. Window accumulation modes

When you specify a trigger, you must also set the the window's **accumulation
mode**. When a trigger fires, it emits the current contents of the window as a
pane. Since a trigger can fire multiple times, the accumulation mode determines
whether the system *accumulates* the window panes as the trigger fires, or
*discards* them.

To set a window to accumulate the panes that are produced when the trigger
fires, invoke`.accumulatingFiredPanes()` when you set the trigger. To set a
window to discard fired panes, invoke `.discardingFiredPanes()`.

Let's look an example that uses a `PCollection` with fixed-time windowing and a
data-based trigger. This is something you might do if, for example, each window
represented a ten-minute running average, but you wanted to display the current
value of the average in a UI more frequently than every ten minutes. We'll
assume the following conditions:

*   The `PCollection` uses 10-minute fixed-time windows.
*   The `PCollection` has a repeating trigger that fires every time 3 elements
    arrive.

The following diagram shows data events for key X as they arrive in the
PCollection and are assigned to windows. To keep the diagram a bit simpler,
we'll assume that the events all arrive in the pipeline in order.

![Diagram of data events for acculumating mode example]({{ "/images/trigger-accumulation.png" | prepend: site.baseurl }} "Data events for accumulating mode example")

##### 8.4.1.1. Accumulating mode

If our trigger is set to `.accumulatingFiredPanes`, the trigger emits the
following values each time it fires. Keep in mind that the trigger fires every
time three elements arrive:

```
  First trigger firing:  [5, 8, 3]
  Second trigger firing: [5, 8, 3, 15, 19, 23]
  Third trigger firing:  [5, 8, 3, 15, 19, 23, 9, 13, 10]
```


##### 8.4.1.2. Discarding mode

If our trigger is set to `.discardingFiredPanes`, the trigger emits the
following values on each firing:

```
  First trigger firing:  [5, 8, 3]
  Second trigger firing:           [15, 19, 23]
  Third trigger firing:                         [9, 13, 10]
```

#### 8.4.2. Handling late data

If you want your pipeline to process data that arrives after the watermark
passes the end of the window, you can apply an *allowed lateness* when you set
your windowing configuration. This gives your trigger the opportunity to react
to the late data. If allowed lateness is set, the default trigger will emit new
results immediately whenever late data arrives.

You set the allowed lateness by using `.withAllowedLateness()` when you set your
windowing function:

```java
  PCollection<String> pc = ...;
  pc.apply(Window.<String>into(FixedWindows.of(1, TimeUnit.MINUTES))
                              .triggering(AfterProcessingTime.pastFirstElementInPane()
                                                             .plusDelayOf(Duration.standardMinutes(1)))
                              .withAllowedLateness(Duration.standardMinutes(30));
```
```py
  # The Beam SDK for Python does not support triggers.
```

This allowed lateness propagates to all `PCollection`s derived as a result of
applying transforms to the original `PCollection`. If you want to change the
allowed lateness later in your pipeline, you can apply
`Window.configure().withAllowedLateness()` again, explicitly.


### 8.5. Composite triggers

You can combine multiple triggers to form **composite triggers**, and can
specify a trigger to emit results repeatedly, at most once, or under other
custom conditions.

#### 8.5.1. Composite trigger types

Beam includes the following composite triggers:

*   You can add additional early firings or late firings to
    `AfterWatermark.pastEndOfWindow` via `.withEarlyFirings` and
    `.withLateFirings`.
*   `Repeatedly.forever` specifies a trigger that executes forever. Any time the
    trigger's conditions are met, it causes a window to emit results and then
    resets and starts over. It can be useful to combine `Repeatedly.forever`
    with `.orFinally` to specify a condition that causes the repeating trigger
    to stop.
*   `AfterEach.inOrder` combines multiple triggers to fire in a specific
    sequence. Each time a trigger in the sequence emits a window, the sequence
    advances to the next trigger.
*   `AfterFirst` takes multiple triggers and emits the first time *any* of its
    argument triggers is satisfied. This is equivalent to a logical OR operation
    for multiple triggers.
*   `AfterAll` takes multiple triggers and emits when *all* of its argument
    triggers are satisfied. This is equivalent to a logical AND operation for
    multiple triggers.
*   `orFinally` can serve as a final condition to cause any trigger to fire one
    final time and never fire again.

#### 8.5.2. Composition with AfterWatermark.pastEndOfWindow

Some of the most useful composite triggers fire a single time when Beam
estimates that all the data has arrived (i.e. when the watermark passes the end
of the window) combined with either, or both, of the following:

*   Speculative firings that precede the watermark passing the end of the window
    to allow faster processing of partial results.
*   Late firings that happen after the watermark passes the end of the window,
    to allow for handling late-arriving data

You can express this pattern using `AfterWatermark.pastEndOfWindow`. For
example, the following example trigger code fires on the following conditions:

*   On Beam's estimate that all the data has arrived (the watermark passes the
    end of the window)
*   Any time late data arrives, after a ten-minute delay
*   After two days, we assume no more data of interest will arrive, and the
    trigger stops executing

```java
  .apply(Window
      .configure()
      .triggering(AfterWatermark
           .pastEndOfWindow()
           .withLateFirings(AfterProcessingTime
                .pastFirstElementInPane()
                .plusDelayOf(Duration.standardMinutes(10))))
      .withAllowedLateness(Duration.standardDays(2)));
```
```py
  # The Beam SDK for Python does not support triggers.
```

#### 8.5.3. Other composite triggers

You can also build other sorts of composite triggers. The following example code
shows a simple composite trigger that fires whenever the pane has at least 100
elements, or after a minute.

```java
  Repeatedly.forever(AfterFirst.of(
      AfterPane.elementCountAtLeast(100),
      AfterProcessingTime.pastFirstElementInPane().plusDelayOf(Duration.standardMinutes(1))))
```
```py
  # The Beam SDK for Python does not support triggers.
```
