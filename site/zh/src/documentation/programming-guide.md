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
%}
```

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
%}
```

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
%}
```

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
%}
```

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
%}
```

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
%}
```

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
%}
```

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
%}
```

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
%}
```

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
%}
```

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
%}
```

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

窗口根据单个元素的时间戳来细分一个` pcollection `。转化即是聚合多样元素，隐式的作用于每一个基础窗口，例如方法` groupbykey `和`Combine`，－它们将每个PCollection处理成多个、有限的窗口，尽管整个集合本身可能是无界的。

一个相关的概念，称为触发器，决定何时计算聚合的结果，而这些数据是无界数据。您可以使用触发器来完善您的PCollection的窗口策略。触发器允许您处理延延迟达的数据或提供早期结果。有关更多信息，请参见触发器部分。

### 7.1. Windowing 基础

一些Beam转化，例如GroupByKey和Combine，通过一个通用的键来分组多个元素。通常，分组操作是将所有在整个数据集中具有相同键的元素分组。对于无界数据集，不可能收集所有的元素，因为新元素不断被添加，并且可能是无限的(例如，流数据)。‘如果您使用的是无界的PCollection，则窗口尤其有用。

在Beam模型中，任何PCollection(包括无界的PCollection)都可以被划分为逻辑窗口。PCollection中的每个元素都根据PCollection的窗口函数分配给一个或多个窗口，每个单独的窗口都包含一个有限数量的元素。分组转化然后在每个窗口的基础上考虑每个PCollection的元素。例如，GroupByKey通过键和窗口来隐式地将PCollection的元素分组。

**注意:** Beam的默认窗口行为是将一个PCollection的所有元素分配到一个全局窗口中，并丢弃最近的数据，即使是无界的PCollection。在对无界的PCollection使用如GroupByKey等分组转化之前，，您必须至少做以下的一个:
* 设置一个非全局的窗口功能。查看设置您的PCollection的窗口函数(设置-您的pcollections-windowing-function)。
* 设置一个非默认触发器(触发器)。这允许全局窗口在其他条件下发出结果，因为默认的窗口行为(等待所有数据到达)将永远不会发生。

如果您没有为您的无界PCollection设置非全局窗口函数或非默认触发器，并随后使用诸如GroupByKey或组合之类的分组转化，那么您的管道将在构建过程中产生错误，您的job将失败。

#### 7.1.1. Windowing 约束

在为PCollection设置了窗口函数之后，在下一次将分组转化应用到PCollection时，该窗口才起作用。
窗口分组发生在需要的基础上。如果您使用窗口转化设置一个窗口函数，那么每个元素都被分配到一个窗口，但是窗口不会被考虑到GroupByKey，或者在窗口和键之间合并聚合。这可能对您的管道有不同的影响。  
考虑下面图中的示例管道:
[![管道应用窗口图]({{ "/images/windowing-pipeline-unbounded.png" | prepend: site.baseurl }} "Pipeline applying windowing")

**Figure:** Pipeline 应用窗口

在上面的管道中，我们通过使用`KafkaIO`来读取一组键/值对来创建一个无界的PCollection，然后使用窗口转化向该集合应用一个窗口函数。
之后，我们将一个ParDo应用到集合中，并使用GroupByKey对ParDo的结果进行分组。
窗口函数对ParDo转化没有影响，因为在GroupByKey需要它们之前，窗口实际上不会被使用。
然而，随后应用了GroupByKey分组的转化结果——数据按键和窗口分组。

#### 7.1.2. 有界窗口应用

您可以在有界的PCollection中使用固定大小的数据集。
但是，请注意，窗口只考虑与PCollection的每个元素相关联的内隐时间戳，而创建固定数据集(例如TextIO)的数据源将相同的时间戳分配给每个元素。这意味着所有的元素都默认是一个全局窗口的默认部分。

为了使用固定数据集的窗口，您可以将自己的时间戳分配给每个元素。使用带`DoFn`的`ParDo`转化,将为每个元素输出一个新的时间戳(例如,在Beam SDK for java中[时间戳]({{ site.baseurl }}/documentation/sdks/javadoc/{{ site.release_latest}}/index.html?org/apache/beam/sdk/transforms/WithTimestamps.html)转化)。

演示如何使用有界的PCollection来影响您的操作
管道处理数据，考虑以下管道:
![无窗口的`GroupByKey`和`ParDo`应用到有界的集合上]({{ "/images/unwindowed-pipeline-bounded.png" | prepend: site.baseurl }} "GroupByKey and ParDo without windowing, on a bounded collection")

**Figure:** 有界集上应用无窗口的`GroupByKey` 与 `ParDo`

在上面的管道中，我们通过使用TextIO读取一组键/值对来创建一个有界的PCollection。然后，我们使用GroupByKey对集合进行分组，并将ParDo转化应用到分组的PCollection。在本例中，GroupByKey创建了一个惟一键的集合，然后ParDo对每一个键应用一次。

请注意，即使您没有设置一个窗口函数，仍然有一个窗口—您的PCollection中的所有元素都被分配给一个全局窗口。
现在，考虑相同的管道，但是使用一个窗口函数:

![带窗口的`GroupByKey`和`ParDo`应用到有界的集合上]({{ "/images/windowing-pipeline-bounded.png" | prepend: site.baseurl }} "带窗口的`GroupByKey`和`ParDo`应用到有界的集合上")

**Figure:** 带窗口的`GroupByKey`和`ParDo`应用到有界的集合上

与前面一样，管道创建了一个有界的键/值对集合。然后，我们为该PCollection设置了一个窗口函数(设置您的Pcollections-windowing-function)。根据窗口的功能对PCollection的元素进行GroupByKey分组转化。随后对每个窗口的每个键多次应用ParDo转化。

### 7.2. 预设 windowing functions

您可以定义不同类型的窗口来划分您的PCollection元素。Beam提供了几个窗口功能，包括:

*  固定时间窗口
*  滑动时间窗口
*  会话窗口
*  全局窗口
*  基于日历的窗户(暂不支持Python)

如果您有更复杂的需求，您也可以定义自己的`windowsFn`。   
注意，每个元素逻辑上可以属于多个窗口，这取决于你使用的窗口函数。例如，滑动时间窗口会创建重叠的窗口，其中一个元素可以分配给多个窗口。

#### 7.2.1. 固定时间窗口（Fixed time windows）

最简单的窗口形式是使用**固定时间窗口**:给定一个时间戳的`PCollection`，它可能会不断地更新，每个窗口可能会捕获到(例如)所有带有时间戳在时间间隔为5分钟时间窗口内的所有元素。

固定时间窗口表示数据流中不重叠的时间间隔。考虑以5分钟时间间隔的windows:在您的无界`PCollection`中所有的元素中，从0:00:00到(但不包括)0:05:00属于第一个窗口，从0:05:00到(但不包括)0:10的时间间隔内的元素属于第二个窗口，以此类推。

![图：时间间隔为30s的固定时间窗口]({{ "/images/fixed-time-windows.png" | prepend: site.baseurl }} "Fixed time windows, 30s in duration")

**图:** 固定时间窗口, 时间间隔30s.

#### 7.2.2. 滑动时间窗口（Sliding time windows）

一个**滑动时间窗**口也表示数据流中的时间间隔;然而，滑动时间窗口可以重叠。例如，每个窗口可能捕获5分钟的数据，但是每隔10秒就会启动一个新窗口。
滑动窗口开始的频率称为周期。
因此，我们的示例是一个窗口持续时间为5分钟和周期为10秒的滑动窗口。

由于多个窗口重叠，数据集中的大多数元素将属于多个窗口。这种类型的窗口对于获取运行数据的平均值是很有用的;在我们的示例中使用滑动时间窗口，您可以计算过去5分钟数据的运行平均值，每10秒更新一次。

![图：华东时间窗口, 时间间隔1min,周期30s]({{ "/images/sliding-time-windows.png" | prepend: site.baseurl }} "Sliding time windows, with 1 minute window duration and 30s window period")

**图:** 华东时间窗口, 时间间隔1min,周期30s.

#### 7.2.3. 会话窗口（Session windows）

一个**会话窗口**定义了包含在另一个元素的某个间隙时间内的元素的窗口。会话窗口应用于每一个基础键上，对于不定期分发的数据非常有用。例如，代表用户鼠标活动的数据流可能有很长一段时间的空闲时间，其中穿插了大量的点击。
如果数据在最小指定的间隔时间之后到达，这将启动一个新窗口的开始。

![时间间隔一分钟的会话窗口]({{ "/images/session-windows.png" | prepend: site.baseurl }} "Session windows, with a minimum gap duration")

**图:** 会话窗口，时间间隔1min. 注意，根据其数据分布，每个数据键都有不同的窗口。

#### 7.2.4. 全局窗口（The single global window）

默认情况下，`PCollection`中的所有数据都被分配给单个全局窗口，而延迟数据将被丢弃。    
如果数据集是固定大小的，那么您可以使用全局窗口缺省值来进行`PCollection`。
如果您使用的是一个无界的数据集(例如来自流数据源)，那么您可以使用单全局窗口，但是在应用`GroupByKey`和`Combine`等聚合转化时要特别谨慎。     
带有默认触发器的单一全局窗口通常要求在处理前全部数据集不可能持续更新数据的可用数据集。     
在无界的`PCollection`上使用全局窗口执行聚合转化时，您应该为该`PCollection`指定一个非默认触发器。

### 7.3. 设置 PCollection's windowing function

您可以通过应用窗口转化为`PCollection`设置窗口函数。当您应用窗口转化时，您必须提供一个窗口`fn`。
`windows fn`决定了您的`PCollection`将用于后续分组转化的窗口函数，例如固定的或滑动的时间窗口。   
当您设置一个窗口函数时，您可能还想为您的`PCollection`设置一个触发器。触发器决定了每个单独的窗口被聚合和释放的时间，并帮助改进窗口在计算较晚的数据和计算早期结果中的执行性能。有关更多信息，请参阅触发器(触发器)部分。

#### 7.3.1. 固定时间窗口（Fixed-time windows）

下面的示例代码展示了如何应用窗口来划分PCollection
到时间间隔为1min的固定窗户上，:

```java
    PCollection<String> items = ...;
    PCollection<String> fixed_windowed_items = items.apply(
        Window.<String>into(FixedWindows.of(Duration.standardMinutes(1))));
```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:setting_fixed_windows
%}
```

#### 7.3.2. 滑动时间窗口（Sliding time windows）

下面的示例代码展示了如何应用窗口将`PCollection`分割为**##滑动时间窗口**。每个窗口长度为30分钟，每5秒启动一个新窗口:

```java
    PCollection<String> items = ...;
    PCollection<String> sliding_windowed_items = items.apply(
        Window.<String>into(SlidingWindows.of(Duration.standardMinutes(30)).every(Duration.standardSeconds(5))));
```
```py
{% github_sample   /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:setting_sliding_windows
%}
```

#### 7.3.3. 会话窗口（Session windows）

下面的示例代码展示了如何应用窗口将PCollection划分为会话窗口，其中每个会话必须被至少10分钟的时间间隔分隔开:
```java
    PCollection<String> items = ...;
    PCollection<String> session_windowed_items = items.apply(
        Window.<String>into(Sessions.withGapDuration(Duration.standardMinutes(10))));
```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:setting_session_windows
%}
```
注意，会话是每个键-集合中的每个键将根据数据分布有自己的会话分组。

#### 7.3.4. 全局窗口（Single global window）

如果您的`PCollection`是有界的(大小是固定的)，您可以将所有的元素分配到一个全局窗口中。下面的示例代码展示了如何为PCollection设置单一的全局窗口:

```java
    PCollection<String> items = ...;
    PCollection<String> batch_items = items.apply(
        Window.<String>into(new GlobalWindows()));
```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:setting_global_window
%}
```

### 7.4. 水印与延迟数据（Watermarks and late data）

在任何数据处理系统中,有一定的时间差数据事件(**“事件时间”**,由时间戳数据元素本身)和实际数据元素的时间可能在`Pipeline`的任何阶段发生或被处理(**处理时间**,由系统上的时钟处理的元素)。
此外，也不能保证数据事件将在您的`Pipeline`中以与它们生成相同的顺序出现。    
例如，假设我们有一个使用固定时间窗口的`PCollection`，有5分钟的窗口。
对于每个窗口，`Beam`必须在给定的窗口范围内收集所有的事件时间时间戳(例如，在第一个窗口的0:00到4:59之间)。
在该范围之外的时间戳(从5点或以后的数据)属于不同的窗口。

然而，数据并不总是保证按时间顺序到达管道，或者总是以可预测的间隔到达。
Beam追踪的是一个水印，这是系统的概念，即当某个窗口中的所有数据都以期望的时间到达管道时。
从而把时间戳在水印后的数据被认为是`延迟数据`。

在我们的示例中，假设我们有一个简单的水印，它假定数据时间戳(事件时间)和数据出现在管道中的时间(处理时间)之间大约有30秒的延迟时间，那么Beam将在5:30关闭第一个窗口。
如果数据记录在5:34到达，但是有一个时间戳将它放在0:00-4:59窗口(例如，3:38)，那么该记录就是较晚的数据。

> **注意:** 为了简单起见，我们假设我们使用的是一个非常简单的水印，用来估计延迟时间。  

在实践中，您的PCollection的数据源决定了水印，而水印可以更精确或更复杂。    
`Beam` 的默认窗口配置尝试确定所有数据何时到达(基于数据源的类型)，然后在窗口的末端向前推进水印。
这种默认配置不允许延迟数据。
触发器(触发器)允许您修改和细化`PCollection`的窗口策略。
您可以使用触发器来决定每个单独的窗口何时聚合并报告其结果，包括窗口如何释放延迟的元素。

#### 7.4.1. 延迟数据管理

> **Note:** 管理延迟数据在Python的Beam SDK中暂不支持。

当你设置你的窗口处策略时，你可以通过调用`.withAllowedLateness`来允许处理延迟数据。下面的代码示例演示了一个窗口策略，该策略允许在窗口结束后的两天内进行后期数据。

```java
    PCollection<String> items = ...;
    PCollection<String> fixed_windowed_items = items.apply(
        Window.<String>into(FixedWindows.of(Duration.standardMinutes(1)))
              .withAllowedLateness(Duration.standardDays(2)));
```

当您在“PCollection”上设置`.withAllowedLateness`时，允许延迟从第一个“PCollection”中派生出来的任何一个子“PCollection”上前向传播。如果您希望在以后的管道中能更改允许的延迟，那么您必须使用`Window.configure().withAllowedLateness()`来明确地实现。

### 7.5. 将时间戳添加到PCollection的元素中

无界源为每个元素提供时间戳。根据您的无界源，您可能需要配置如何从原始数据流中提取时间戳。

然而，有界源(例如来自`TextIO`的文件)不提供时间戳。
如果需要时间戳，必须将它们添加到`PCollection`的元素中。

您可以通过应用[ParDo](#ParDo)转化来为`PCollection`的元素分配新的时间戳，该转化将使用您设置的时间戳来输出新元素。

一个可能的示例，如果您的管道从输入文件读取日志记录，并且每个日志记录都包含一个时间戳字段;
由于您的管道从一个文件中读取记录，则文件源不会自动地分配时间戳。
您可以从每个记录中解析时间戳字段，并使用`DoFn`的`ParDo`转化将时间戳附加到您的`PCollection`中的每个元素。

```java
      PCollection<LogEntry> unstampedLogs = ...;
      PCollection<LogEntry> stampedLogs =
          unstampedLogs.apply(ParDo.of(new DoFn<LogEntry, LogEntry>() {
            public void processElement(ProcessContext c) {
              // 从当前正在处理的日志条目中提取时间戳。
              Instant logTimeStamp = extractTimeStampFromLogEntry(c.element());
              // 使用ProcessContext.outputWithTimestamp(而不是ProcessContext.output)
              //发送带有时间戳的实体。
              c.outputWithTimestamp(c.element(), logTimeStamp);
            }
          }));
```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets_test.py tag:setting_timestamp
%}
```

## 8. Triggers

> **注意:** Triggers只能应用于Java的the Beam SDK。
> python的the Beam SDK不支持Triggers.

当收集和分组数据到窗口中时，Beam使用**tirggers**来确定什么时候发出每个窗口的汇总结果(称为a
*面板*)。如果你使用Beam的默认窗口配置和[默认触发器 default
trigger](#the-default-trigger)，当`beam`判断所有的[数据都已经到达（estimates all data has arrived）](#watermarks-and-late-data)，它会输出汇总结果
并丢弃该窗口的所有后续数据。

你可以为你的`PCollection`设置其他触发器来更改这个默认行为。
Beam提供了许多预置的触发器:

*   **基于事件时间的触发器**：正如每个数据元素的时间戳所暗示的那样，这些触发器运行在事件时间上。而且Beam的默认触发就是基于事件时间。

*   **基于处理时间的触发器**：这些触发器运行在处理时间——数据元素可以在传输过程中在任何给定阶段被处理
*   **数据驱动的触发器**：基于这些触发器通过检查到达每个窗口的数据来运行，一旦数据遇到某个特定属性就触发。目前，数据驱动的触发器仅支持遇到特定数字后触发。
*   **复合触发器**：这些触发器将多个触发器用不同的方式组合起来。


在较高的层次上，触发器提供了两个额外的功能，而不是简单地在窗口的末尾输出:

*   在给定窗口的所有数据都到达之前，触发器允许Beam发送早期结果。
例如，在一定时间流逝之后或者在特定数量的元素到达之后发出结果。
*   触发器允许在事件时间水印通过窗口结束后触发处理延迟数据。

这些功能允许你根据不同的用例来控制你数据流，并且平衡不同的影响因素:

*   **完整性:** 在你计算出结果之前，拥有所有数据有多么重要？
*   **延迟:** 你希望等待数据的时间有多长?例如，你是否会一直等到你认为你拿到所有的数据的时候?当它到达时，你处理数据吗?
*   **成本：** 你愿意花多少钱来降低延迟?

例如，一个要求时间敏感的系统更新可能会使用一个严格的基于时间的触发器——每N秒就会发出一个窗口，比起数据完整性更重视数据及时性。
一个重视数据完整性而不是结果的精确时间的系统可能会选择使用Beam的默认触发器，在窗口的最后面发出结果。

你还可以为一个[采用全局窗口的 global window for its windowing function](#windowing)无界`PCollection`设置触发器。在你希望你的数据管道能够为一个无界数据集提供定期更新的时候是非常有用的
——例如，提供当前时间或者更新的每N秒的所有数据的运行平均值，或者每N个元素。

### 8.1. 事件时间触发(Event time triggers)

`AfterWatermark`触发器在事件时间上运行。基于与数据元素相连的时间戳，在水印经过窗口后，`AfterWatermark`触发器会发出窗口的内容。[水印](#Watermark)是一种全局进度度量，在任何给定的点上，都是Beam在管道内输入完整性的概念。`AfterWatermark.pastEndOfWindow()`只有当水印经过窗口时才会起作用。
此外，如果您的管道在窗口结束之前或之后接收到数据，那么可以使用`.withEarlyFirings(trigger)` 与`.withLateFirings(trigger)`配置一个触发器去处理。
下面的示例展示了一个账单场景，并使用了早期和后期的[**Firings**]:

```java
  // 在月末创建一个账单
  AfterWatermark.pastEndOfWindow()
      // 在这个月，要接近实时的估计。
      .withEarlyFirings(
          AfterProcessingTime
              .pastFirstElementInPane()
              .plusDuration(Duration.standardMinutes(1))
      // Fire on any late data so the bill can be corrected.
      //对任何迟来的数据进行Fire，这样就可以纠正该账单。
      .withLateFirings(AfterPane.elementCountAtLeast(1))
```
```py
  # Beam SDK for Python 不支持Striggers
```

#### 8.1.1. 默认触发器

`PCollection`的默认触发器是基于事件时间的，当Beam的watermark经过窗口的末端时，会发出窗口的结果，然后每次延迟的数据到达时都会触发。

但是，如果您同时使用默认的窗口配置和默认触发器，默认触发器只会发送一次，而延迟的数据则会被丢弃。这是因为默认的窗口配置有一个允许的延迟值为0。有关修改此行为的信息，请参见处理延迟数据部分。

### 8.2. 处理时间触发器（**time trigger**）

`AfterProcessingTime`触发器在处理时间上运行。
例如，在接收到数据之后，`AfterProcessingTime.pastFirstElementInPane() `会释放一个窗口。处理时间由系统时钟决定，而不是数据元素的时间戳。
`AfterProcessingTime`对于来自窗口的早期结果非常有用，尤其是具有大型时间框架的窗口，例如一个全局窗口。

### 8.3. 数据驱动触发器(Data-driven triggers)

Beam提供了一种数据驱动触发器`AfterPane.elementCountAtLeast()`。这个触发器在元素计数上起作用;在当前panes至少收集了N个元素之后，它就会触发。这允许一个窗口可以释放早期的结果(在所有的数据积累之前)，如果您使用的是一个全局窗口，那么这个窗口就特别有用。

需要特别注意，例如，如果您使用`.elementCountAtLeast(50)`计数但是只有32个元素到达，那么这32个元素永远存在。如果32个元素对您来说很重要，那么考虑使用复合触发器(组合-触发器)来结合多个条件。这允许您指定多个触发条件，例如“当我接收到50个元素时，或者每1秒触发一次”。

### 8.4. 触发器设置


当您使用窗口转换为PCollection设置一个窗口函数时，您还可以指定一个触发器。
You set the trigger(s) for a `PCollection` by invoking the method
`.triggering()` on the result of your `Window.into()` transform, as follows:
您可以在`Window.into()`转化基础上调用`.triggering()`方法来为PCollection设置触发器(s)，如下:
```java
  PCollection<String> pc = ...;
  pc.apply(Window.<String>into(FixedWindows.of(1, TimeUnit.MINUTES))
                               .triggering(AfterProcessingTime.pastFirstElementInPane()
                                                              .plusDelayOf(Duration.standardMinutes(1)))
                               .discardingFiredPanes());
```
```py
  # Beam SDK for Python 不支持Striggers.
```
这个代码样例为`PCollection`设置了一个基于时间的触发器，在该窗口的第一个元素被处理后一分钟就会发出结果。
代码样例中的最后一行`.discardingFiredPanes()`是窗口的积累模式**accumulation mode**。

#### 8.4.1. 窗口积累模式[Window accumulation modes]

当您指定一个触发器时，您还必须设置窗口的累积模式。当触发器触发时，它会将窗口的当前内容作为一个panes发出。由于触发器可以多次触发，所以积累模式决定系统是否会在触发器触发时积累窗口panes，或者丢弃它们。
要设置一个窗口来积累触发器触发时产生的panes，请调用`.accumulatingFiredPanes()`当你设置触发器时。要设置一个窗口来丢弃被触发的panes，调用`.discardingFiredPanes()`。

让我们来看一个使用固定时间窗口和基于数据的触发器的`PCollection`的例子。这是您可能会做的事情，例如，每个窗口代表一个10分钟的运行平均值，但是您想要在UI中显示平均值的当前值，而不是每十分钟。我们将假设以下条件:

*   The `PCollection` 采用10-minute 固定时间窗口.
*   The `PCollection` 每次三个元素到达，可被仿佛触发的触发器.

下图显示了键X的数据事件，它们到达了PCollection并被分配给了windows。为了保持图表的简单，我们假定所有事件都按顺序到达了管道。
![图数据事件积累模式示例]({{ "/images/trigger-accumulation.png" | prepend: site.baseurl }} "Data events for accumulating mode example")

##### 8.4.1.1. 积累模式（Accumulating mode）

如果我们的触发器被设置为`.accumulatingFiredPanes`。每次触发时，触发器都会释放出如下的值。请记住，每当有三个元素到达时，触发器就会触发:
```
  First trigger firing:  [5, 8, 3]
  Second trigger firing: [5, 8, 3, 15, 19, 23]
  Third trigger firing:  [5, 8, 3, 15, 19, 23, 9, 13, 10]
```


##### 8.4.1.2. 丢弃模式

如果你的触发器设置成`.discardingFiredPanes`, 则在每一次触发时会释放如下值:

```
  First trigger firing:  [5, 8, 3]
  Second trigger firing:           [15, 19, 23]
  Third trigger firing:                         [9, 13, 10]
```

#### 8.4.2. 延迟数据处理

如果您希望您的管道处理watermark在窗口结束后到达的数据，那么您可以在设置窗口配置时应用一个允许的延迟。这使您的触发器有机会对最近的数据作出反应。如果设置了延迟置，默认的触发器将在任何延迟的数据到达时立即释放新的结果。

当你设置窗口功能时，使用`.withAllowedLateness()`设置允许延迟:

```java
  PCollection<String> pc = ...;
  pc.apply(Window.<String>into(FixedWindows.of(1, TimeUnit.MINUTES))
                              .triggering(AfterProcessingTime.pastFirstElementInPane()
                                                             .plusDelayOf(Duration.standardMinutes(1)))
                              .withAllowedLateness(Duration.standardMinutes(30));
```
```py
  # Beam SDK for Python 不支持Striggers
```

这允许延迟传播到所有的`PCollection`，这是由于将转换应用到原始的PCollection而产生的。如果您想要在您的管道中更改允许的延迟，您可以再次使用`Window.configure().withAllowedLateness()`。

### 8.5. 复合触发器Composite triggers

您可以组合多个触发器来形成复合触发器，并且可以指定触发器，在大多数情况下，或者在其他定制条件下，多次释放结果。

#### 8.5.1. Composite trigger 类型

Beam 包括如下类型 composite triggers:

*   你可以额外通过`.withEarlyFirings` 和`.withLateFirings`添加早期 firings 或者晚期 firings的`AfterWatermark.pastEndOfWindow`.
*   `Repeatedly.forever`指定一个永远执行的触发器。只要触发了触发器的条件，它就会产生一个窗口来释放结果，然后重新设置并重新启动。将`Repeatedly.forever`与`.orFinally`结合在一起，指定条件，使某个重复触发的触发器停止触发。
*   `AfterEach.inOrder`将多个触发器组合在一个特定的序列中。每当序列中的触发器发出一个窗口时，序列就会向下一个触发器前进。
*   `AfterFirst`获取多个触发器，并第一次发出它的任何一个参数触发器都是满意的。这相当于多个触发器的逻辑或操作。当所有的参数触发器都被满足时，
*   `AfterAll`就需要多个触发器并发出,这相当于多个触发器的逻辑和操作。
*   `orFinally` ，它可以作为最后的条件，使任何触发器在最后时刻触发，而不会再次触发。

#### 8.5.2. AfterWatermark.pastEndOfWindow的复合触发器

当Beam估计所有的数据都已经到达(例如，当水印通过窗口的末端)时，一些最有用的组合触发了一段时间，或者两者的结合，或者两者的结合:
*   推测触发(Speculative firings)能够在水印通过窗口的末端时，允许对部分结果进行更快的处理。
*   延迟触发（Late firings）在水印经过窗口后的延迟触发，以便处理延迟到达的数据。

你可以使用`AfterWatermark.pastEndOfWindow`表示这种模式。例如，下面的例子触发了以下条件下的代码:
*   根据Beam的估计，所有的数据都已经到达(水印通过窗口的末端)
*   任何时间延迟的数据到达，经过10分钟的延迟
*   两天之后，我们假设没有更多的数据将到达，触发器停止执行

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
  # Beam SDK for Python 不支持Striggers
```

#### 8.5.3. 其他 composite triggers

您还可以构建其他类型的复合触发器。下面的示例代码显示了一个简单的复合触发器，只要该pane至少有100个元素，或者一分钟后就会触发。

```java
  Repeatedly.forever(AfterFirst.of(
      AfterPane.elementCountAtLeast(100),
      AfterProcessingTime.pastFirstElementInPane().plusDelayOf(Duration.standardMinutes(1))))
```
```py
  # Beam SDK for Python 不支持Striggers
```
