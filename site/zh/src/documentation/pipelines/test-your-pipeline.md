---
layout: default
title: "Test Your Pipeline"
permalink: /documentation/pipelines/test-your-pipeline/
---
# 测试你的Pipeline

* TOC
{:toc}

测试你的pipeline是一种开发有效数据处理解决方案中特别重要的步骤。 Beam 模型的间接性质, 在你的用户代码构造了一个远程执行的pipeline图, 可以使调试或者失败运行一个重要或者不重要的任务。 通常在pipeline代码上执行本地单元测试比调试pipeline远程执行更快更简单。

在你选择的流道上运行pipeline之前, 在本地测试管道代码的单元通常是识别和修复管道代码中错误的最佳方式。 在本地测试你的pipeline的单元还允许你使用熟悉的或者最喜欢的本地调试工具。

你可以使用 [DirectRunner]({{ site.baseurl }}/documentation/runners/direct) ，本地运行器有助于测试和本地开发。

在使用 `DirectRunner` 测试pipeline之后，你可以使用你选择的流道小规模进行测试。 例如，使用 Flink runner 与本地或远程 Flink集群。 






Beam SDKs 提供了多种方式来对从最低到最高级别的 pipeline 代码进行单元测试。 从最低到最高，这些是：

*   你可以测试单个函数对象, 例如 [DoFns]({{ site.baseurl }}/documentation/programming-guide/#pardo), 在你的pipeline的核心变换中。
*   你可以测试整个 [Composite Transform]({{ site.baseurl }}/documentation/programming-guide/#composite-transforms) 作为一个单位.
*   您可以对整个 pipeline 执行端到端测试。

为了支持单元测试，`Beam SDK for Java` 在测试包中提供了许多测试类。(https://github.com/apache/beam/tree/master/sdks/java/core/src/test/java/org/apache/beam/sdk). 你可以使用这些测试作为参考和指南。

## 测试单个DoFn对象

pipeline 的 `DoFn` 函数中的代码经常运行, 并且经常跨多个计算引擎实例运行。 在使用运行程序服务运行之前对 `DoFn` 对象进行单元测试可以节省大量的调试时间和精力。

用于 Java 的 `Beam SDK` 提供了一种方便的方法来测试名为 `DoFnTester` 的单个 `DoFn` ，它包含在 `SDK Transforms` 包中。
Beam SDK 为 java 提供了一种简便的方法来测试名为 [DoFnTester] 的单个 `DoFn` ,(https://github.com/apache/beam/blob/master/sdks/java/core/src/test/java/org/apache/beam/sdk/transforms/DoFnTesterTest.java), 它包含在 SDK `Transforms` 包中。

`DoFnTester` 使用 [JUnit](http://junit.org) 框架. 要使用 `DoFnTester`, 你需要执行以下操作:

1. 创建一个 `DoFnTester`. 将你需要测试的 `DoFn` 实例传递给 `DoFnTester` 的静态工厂方法.
2. 为你的 `DoFn` 创建一个或者多个合适类型的主要测试输入. 如果你的 `DoFn` 采用侧面输入 and/or 产生 [多个输出]({{ site.baseurl }}/documentation/programming-guide#additional-outputs), 你还应该创建侧面输入和输出的标签。
3. 调用 `DoFnTester.processBundle` 来处理主要输入。
4. 使用JUnit的 `Assert.assertThat` 方法来确保从 `processBundle` 返回的测试输出与您的预期值相匹配。

### 创建一个DoFnTester

创建一个 `DoFnTester` , 首先创建要测试的 `DoFn` 的实例。 然后，当您使用 `.of()` 静态工厂方法创建 `DoFnTester` 时，可以使用该实例 ：

```java 
static class MyDoFn extends DoFn<String, Integer> { ... }
  MyDoFn myDoFn = ...;

  DoFnTester<String, Integer> fnTester = DoFnTester.of(myDoFn);
```

### 创建测试输入

你需要为 `DoFnTester` 创建一个或多个测试输入以发送到你的 `DoFn`。 要创建测试输入，只需创建一个或多个与 `DoFn` 接受的输入类型相同的输入变量。 在上述情况下：

```java
static class MyDoFn extends DoFn<String, Integer> { ... }
MyDoFn myDoFn = ...;
DoFnTester<String, Integer> fnTester = DoFnTester.of(myDoFn);

String testInput = "test1";
```

#### 侧面（？） 输入

如果你的 `DoFn` 接受侧输入,您可以使用 `DoFnTester.setSideInputs` 方法创建这些侧输入。

```java
static class MyDoFn extends DoFn<String, Integer> { ... }
MyDoFn myDoFn = ...;
DoFnTester<String, Integer> fnTester = DoFnTester.of(myDoFn);

PCollectionView<List<Integer>> sideInput = ...;
Iterable<Integer> value = ...;
fnTester.setSideInputInGlobalWindow(sideInput, value);
```

有关更多信息，请参阅[side inputs]({{ site.baseurl }}/documentation/programming-guide/#side-inputs)的 `ParDo` 文档。

#### 附加的输出

如果你的 `DoFn` 产生多个输出 `PCollections`，则需要设置你将用于访问每个输出的适当 `TupleTag` 对象。 具有多个输出的 `DoFn` 为每个输出产生一个 `PCollectionTuple`; 您需要提供一个与该元组中的每个输出相对应的 `TupleTagList`。

假设你的 `DoFn` 产生 `String` 和 `Integer` 的输出。 您为每个对象创建 `TupleTag` 对象，并将它们捆绑到 `TupleTagList` 中，然后将其设置为`DoFnTester`，如下所示：

```java
static class MyDoFn extends DoFn<String, Integer> { ... }
MyDoFn myDoFn = ...;
DoFnTester<String, Integer> fnTester = DoFnTester.of(myDoFn);

TupleTag<String> tag1 = ...;
TupleTag<Integer> tag2 = ...;
TupleTagList tags = TupleTagList.of(tag1).and(tag2);

fnTester.setOutputTags(tags);
```

有关更多信息，请参阅 `ParDo` 文档了解[additional outputs]({{ site.baseurl }}/documentation/programming-guide/#additional-outputs)。

### 处理测试输入和检查结果

要处理输入（并因此在 `DoFn`上运行测试），可以调用 `DoFnTester.processBundle` 方法。 当您调用 `processBundle` 时，您将传递 `DoFn`的一个或多个主要测试输入值。 如果设置侧面输入，则侧面输入可用于您提供的每批主要输入。

`DoFnTester.processBundle` 返回一个输出列表，即与 `DoFn` 指定输出类型相同类型的对象。 对于`DoFn<String, Integer>`，`processBundle` 返回 `List<Integer>` ：

```java  
static class MyDoFn extends DoFn<String, Integer> { ... }
MyDoFn myDoFn = ...;
DoFnTester<String, Integer> fnTester = DoFnTester.of(myDoFn);

String testInput = "test1";
List<Integer> testOutputs = fnTester.processBundle(testInput);
```

要检查 `processBundle` 的结果，您可以使用 JUnit 的 `Assert.assertThat` 方法来测试输出列表是否包含您期望的值：

```java  
String testInput = "test1";
List<Integer> testOutputs = fnTester.processBundle(testInput);

Assert.assertThat(testOutputs, Matchers.hasItems(...));

// Process a larger batch in a single step.
Assert.assertThat(fnTester.processBundle("input1", "input2", "input3"), Matchers.hasItems(...));
```

## 测试复合变换

要测试您创建的复合变换，可以使用以下模式：

*   创建 `TestPipeline` 。
*   创建一些静态的，已知的测试输入数据。
*   使用 `Create` 创建一个 `PCollection` 的输入数据。
*   `Apply` 将你的复合变换应用于输入`PCollection` 并保存生成的输出 `PCollection`。
*   使用 `PAssert` 及其子类验证输出 `PCollection` 包含你期望的元素。

### TestPipeline

[TestPipeline](https://github.com/apache/beam/blob/master/sdks/java/core/src/main/java/org/apache/beam/sdk/testing/TestPipeline.java) 是专门用于测试转换的 Beam Java SDK 中包含的类。对于测试, 当你在创建 `pipeline` 对象的时候，使用 `TestPipeline` 代替 `Pipeline` 。 与 `Pipeline.create` 不同 , `TestPipeline.create` 在内部处理设置 `PipelineOptions` 。

你创建一个 `TestPipeline` 如下:

```java
Pipeline p = TestPipeline.create();
```

> **Note:** Read about testing unbounded pipelines in Beam in [this blog post]({{ site.baseurl }}/blog/2016/10/20/test-stream.html).

### 使用 Create 转换（transform ？）

你可以使用 `Create` 在一个标准的内存集合类 ( 如 `Java List` ) 中创建 `PCollection` 。 有关详细信息，请参阅 [Creating a PCollection]({{ site.baseurl }}/documentation/programming-guide/#creating-a-pcollection)。

### PAssert
[PAssert]({{ site.baseurl }}/documentation/sdks/javadoc/{{ site.release_latest }}/index.html?org/apache/beam/sdk/testing/PAssert.html) 是一个包含在Beam Java SDK中的类，它是对` PCollection` 的内容的断言。 您可以使用 `PAssert` 来验证 `PCollection` 是否包含一组特定的预期元素。

对于给定的 `PCollection`，您可以使用 `PAssert` 验证内容如下：

```java  
PCollection<String> output = ...;

// Check whether a PCollection contains some elements in any order.
PAssert.that(output)
.containsInAnyOrder(
  "elem1",
  "elem3",
  "elem2");
```

任何使用 `PAssert `的代码都必须在 `JUnit `和 `Hamcrest `中链接。 如果您使用 `Maven `，可以通过向项目的 `pom.xml `文件添加以下依赖关系来连接 `Hamcrest `：

```java 
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>hamcrest-all</artifactId>
    <version>1.3</version>
    <scope>test</scope>
</dependency>
```

有关这些类如何工作的更多信息，请参阅[org.apache.beam.sdk.testing]({{ site.baseurl }}/documentation/sdks/javadoc/{{ site.release_latest }}/index.html?org/apache/beam/sdk/testing/package-summary.html) 包文档。

### 复合变换的示例测试

以下代码显示了一个复合变换的完整测试。 测试将 `Count` 转换应用于 `String` 元素的输入`PCollection` 。 测试使用 `Create `变换从 `Java List <String>` 创建输入 `PCollection` 。

```java  
public class CountTest {

// Our static input data, which will make up the initial PCollection.
static final String[] WORDS_ARRAY = new String[] {
"hi", "there", "hi", "hi", "sue", "bob",
"hi", "sue", "", "", "ZOW", "bob", ""};

static final List<String> WORDS = Arrays.asList(WORDS_ARRAY);

public void testCount() {
  // Create a test pipeline.
  Pipeline p = TestPipeline.create();

  // Create an input PCollection.
  PCollection<String> input = p.apply(Create.of(WORDS)).setCoder(StringUtf8Coder.of());

  // Apply the Count transform under test.
  PCollection<KV<String, Long>> output =
    input.apply(Count.<String>perElement());

  // Assert on the results.
  PAssert.that(output)
    .containsInAnyOrder(
        KV.of("hi", 4L),
        KV.of("there", 1L),
        KV.of("sue", 2L),
        KV.of("bob", 2L),
        KV.of("", 3L),
        KV.of("ZOW", 1L));

  // Run the pipeline.
  p.run();
}
```

## 测试一个 Pipeline 的端到端

您可以使用 Beam SDKs 中的测试类（例如，用于 `Java` 的 `Beam SDK` 中的 `TestPipeline` 和 `PAssert` ）来测试整个Pipeline端到端。 通常，要测试整个Pipeline，请执行以下操作：

*   对于您的 `Pipeline` 的每个输入数据源，创建一些已知的静态测试输入数据。
*   创建一些静态测试输出数据，匹配您在  `Pipeline` 的最终输出 `PCollection` 中的期望值。
*   创建一个 `TestPipeline` 代替标准的 `Pipeline.create` 。
*   使用 `Create` 转换代替您的Pipeline的 `Read`转换，从静态输入数据中创建一个或多个 `PCollection`。
*   应用你的 `Pipeline` 的转换。
*   代替 `Pipeline` 的 `写` 转换，使用 `PAssert` 来验证您的 `Pipeline` 产生的最终 `PCollection` 的内容是否与静态输出数据中的预期值相匹配。

### 测试 WordCount Pipeline

以下示例代码显示了如何测试 [WordCount example pipeline]({{ site.baseurl }}/get-started/wordcount-example/)。`WordCount` 通常从输入数据的文本文件中读取行; 相反，测试创建一个包含一些文本行的 Java `List<String>` ，并使用 `Create` 转换创建一个初始的 `PCollection`。

`WordCount` 的最终变换（来自复合变换 `CountWords`）产生适合于打印的格式化单词统计的 `PCollection <String>` 。 我们的测试流程不是将该 `PCollection` 写入输出文本文件，而是使用 `PAssert` 验证 `PCollection` 的元素是否与包含我们预期输出数据的静态 `String` 数组的元素相匹配。

```java
public class WordCountTest {

    // Our static input data, which will comprise the initial PCollection.
    static final String[] WORDS_ARRAY = new String[] {
      "hi there", "hi", "hi sue bob",
      "hi sue", "", "bob hi"};

    static final List<String> WORDS = Arrays.asList(WORDS_ARRAY);

    // Our static output data, which is the expected data that the final PCollection must match.
    static final String[] COUNTS_ARRAY = new String[] {
        "hi: 5", "there: 1", "sue: 2", "bob: 2"};

    // Example test that tests the pipeline's transforms.

    public void testCountWords() throws Exception {
      Pipeline p = TestPipeline.create();

      // Create a PCollection from the WORDS static input data.
      PCollection<String> input = p.apply(Create.of(WORDS)).setCoder(StringUtf8Coder.of());

      // Run ALL the pipeline's transforms (in this case, the CountWords composite transform).
      PCollection<String> output = input.apply(new CountWords());

      // Assert that the output PCollection matches the COUNTS_ARRAY known static output data.
      PAssert.that(output).containsInAnyOrder(COUNTS_ARRAY);

      // Run the pipeline.
      p.run();
    }
}
```
