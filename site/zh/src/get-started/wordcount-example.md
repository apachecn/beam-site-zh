---
layout: default
title: "Beam WordCount Examples"
permalink: get-started/wordcount-example/
redirect_from: /use/wordcount-example/
---

# Apache Beam WordCount Examples

* TOC
{:toc}

<nav class="language-switcher">
  <strong>Adapt for:</strong>
  <ul>
    <li data-type="language-java">Java SDK</li>
    <li data-type="language-py">Python SDK</li>
  </ul>
</nav>

WordCount 例子展示了如何构建一个可处理的pipeline（管道）来读取文本，将一行行文本分割为逐个的单词，
并且对每个单词进行频率统计。 Beam SDK 源码里面包含四个更加详细的 WordCount 例子，彼此之间相互构建。
所有例子的输入文本都是 Shakespeare（莎士比亚）的文本。

每个 WordCount 例子介绍了 Beam 处理模型中不同的概念。首先理解 Minimal WordCount 例子，这也是
最简单的例子。一旦你感觉使用基础的语法构建一个 pipeline（管道）没有问题了，可以通过其他例子学习
更多的概念。

* **Minimal WordCount** 例子展示了构建一个 pipeline（管道）所涉及的基础概念。
* **WordCount** 例子介绍了在创建可重用和可维护的 pipeline（管道）的一些更基础的练习。
* **Debugging WordCount** 介绍如何打印日志和debug。
* **Windowed WordCount** 展示了如何使用 Beam 程序模型来处理无界和有界的数据集。


## MinimalWordCount 例子

Minimal WordCount 例子展示了一个简单的读取文件文件的 pipeline（管道），并使用 transform（转换）
操作将文本进行分割，对单词进行计数，然后将数据写入到一个输出文本文件。这个例子将输出和输入的地址写死在代码里面，
并且没有做错误检查；这样目的是为了让你了解创建一个 Beam pipeline（管道）的基本步骤。相比于标准化的 Beam
pipelines（管道）的构建，这种缺乏参数化的特定 pipeline（管道）在不同的 runners 之间就很不方便使用。
在后面的例子，我们会参数化 pipeline（管道）的输入和输出源，并且展示一些更好的 pipeline（管道）的练习。

**用 Java 运行这个例子:**

{:.runner-direct}
```
$ mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.MinimalWordCount
```

{:.runner-apex}
```
$ mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.MinimalWordCount \
     -Dexec.args="--inputFile=pom.xml --output=counts --runner=ApexRunner" -Papex-runner
```

{:.runner-flink-local}
```
$ mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.MinimalWordCount \
     -Dexec.args="--runner=FlinkRunner --inputFile=pom.xml --output=counts" -Pflink-runner
```

{:.runner-flink-cluster}
```
$ mvn package exec:java -Dexec.mainClass=org.apache.beam.examples.MinimalWordCount \
     -Dexec.args="--runner=FlinkRunner --flinkMaster=<flink master> --filesToStage=target/word-count-beam-bundled-0.1.jar \
                  --inputFile=/path/to/quickstart/pom.xml --output=/tmp/counts" -Pflink-runner

You can monitor the running job by visiting the Flink dashboard at http://<flink master>:8081
```

{:.runner-spark}
```
$ mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.MinimalWordCount \
     -Dexec.args="--runner=SparkRunner --inputFile=pom.xml --output=counts" -Pspark-runner
```

{:.runner-dataflow}
```
$ mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.MinimalWordCount \
   -Dexec.args="--runner=DataflowRunner --gcpTempLocation=gs://<your-gcs-bucket>/tmp \
                --inputFile=gs://apache-beam-samples/shakespeare/* --output=gs://<your-gcs-bucket>/counts" \
     -Pdataflow-runner
```

想要看 Java 版本的完整代码的话, 可以阅读
**[MinimalWordCount](https://github.com/apache/beam/blob/master/examples/java/src/main/java/org/apache/beam/examples/MinimalWordCount.java).**

**用 Python 运行这个例子:**

{:.runner-direct}
```
python -m apache_beam.examples.wordcount_minimal --input README.md --output counts
```

{:.runner-apex}
```
This runner is not yet available for the Python SDK.
```

{:.runner-flink-local}
```
This runner is not yet available for the Python SDK.
```

{:.runner-flink-cluster}
```
This runner is not yet available for the Python SDK.
```

{:.runner-spark}
```
This runner is not yet available for the Python SDK.
```

{:.runner-dataflow}
```
# As part of the initial setup, install Google Cloud Platform specific extra components.
pip install apache-beam[gcp]
python -m apache_beam.examples.wordcount_minimal --input gs://dataflow-samples/shakespeare/kinglear.txt \
                                                 --output gs://<your-gcs-bucket>/counts \
                                                 --runner DataflowRunner \
                                                 --project your-gcp-project \
                                                 --temp_location gs://<your-gcs-bucket>/tmp/
```

想要看 Python 版本的完整代码的话, 可以阅读
**[wordcount_minimal.py](https://github.com/apache/beam/blob/master/sdks/python/apache_beam/examples/wordcount_minimal.py).**

**核心概念:**

* 创建 Pipeline
* 对 Pipeline 进行 transform（转换）
* Reading input (在这个例子里面就是: 读取文本文件)
* 运用 ParDo transforms
* 运用 SDK-provided transforms (在这个例子里面就是: Count)
* Writing output (在这个例子里面就是: 写入到一个文本文件)
* 执行这个 Pipeline（管道）


下面部分会更详细介绍这些概念，相关的代码摘自于 Minimal WordCount 的 pipeline 中。


### 创建 Pipeline

在这个例子中，代码首先创建一个 `PipelineOptions` 对象。这个对象可以让我们为 pipeline 设置不同的选项，
例如一个 pipeline runner 会执行我们的pipeline，这个 runner 需要许多指定的配置。在这个例子中，我们
在代码里面设置这些选项，但是一般情况下，都会通过命令行参数设置 `PipelineOptions`。

你可以指定一个 runner 执行你的 pipeline,例如`DataflowRunner` 或者 `SparkRunner`。
如果你忽略了指定一个 runner，在这个例子中，pipeline 会默认使用 `DirectRunner` 执行 pipeline。
下一节，我们会指定 pipeline 的 runner。


```java
 PipelineOptions options = PipelineOptionsFactory.create();

    // In order to run your pipeline, you need to make following runner specific changes:
    //
    // CHANGE 1/3: Select a Beam runner, such as DataflowRunner or FlinkRunner.
    // CHANGE 2/3: Specify runner-required options.
    // For DataflowRunner, set project and temp location as follows:
    //   DataflowPipelineOptions dataflowOptions = options.as(DataflowPipelineOptions.class);
    //   dataflowOptions.setRunner(DataflowRunner.class);
    //   dataflowOptions.setProject("SET_YOUR_PROJECT_ID_HERE");
    //   dataflowOptions.setTempLocation("gs://SET_YOUR_BUCKET_NAME_HERE/AND_TEMP_DIRECTORY");
    // For FlinkRunner, set the runner as follows. See {@code FlinkPipelineOptions}
    // for more details.
    //   options.setRunner(FlinkRunner.class);
```

```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:examples_wordcount_minimal_options
%}```

下一步是通过我们构造的选项创建一个 `Pipeline`。Pipeline 对象其实就是构造了数据转换操作的流程，并执行这个流程，当然这和 pipeline（管道）是密切相关的。

```java
Pipeline p = Pipeline.create(options);
```

```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:examples_wordcount_minimal_create
%}```

### 运用 pipeline transforms
Minimal WordCount pipeline（管道）的实现中包含一些 pipeline 的 transforms（转换）操作，将数据读入pipeline，
操作或以其他方式转换数据，并写出结果。Transforms（转换）可以由单个操作组成，也可以包含多个复杂的 transforms（转换）操作(就是一个 [复合 transform]({{ site.baseurl }}/documentation/programming-guide#composite-transforms))

每个 transform （转换）都会接受一些输入数据，并产生一些输出数据。输入数据和输出数据通常用 SDK 中 `PCollection` 类来声明。`PCollection` 是一个比较特殊的类，由 Beam SDK提供，你可以用来标识任何大小的数据集，包括无界数据集。


<img src="{{ "/images/wordcount-pipeline.png" | prepend: site.baseurl }}" alt="Word Count pipeline diagram">
图 1: pipeline 的流水线.

The Minimal WordCount pipeline 包含五个 transforms（转换操作）:

1.  `Pipeline` 对象执行 `Read` transform（转换），并产生一个  `PCollection` 数据集作为输出。
    输出的 `PCollection` 的每个元素都代表输入文件的一行文本。这个例子使用的输入数据存储在可公开访问的Google云端存储
    bucket ("gs://")。

    ```java
    p.apply(TextIO.read().from("gs://apache-beam-samples/shakespeare/*"))
    ```

    ```py
    {% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:examples_wordcount_minimal_read
    %}```

2.  一个 [ParDo]({{ site.baseurl }}/documentation/programming-guide/#pardo)
    transform（转换），每个元素都会调用这个转换，将文本行分割成逐个的单词。这个 transform（转换的）的输入是上一步
    `TextIO.Read` transform（转换）生成的 `PCollection`。这个 `ParDo` 转换输出一个新的 `PCollection`，
    这个 `PCollection`中的每个元素都是文本中一个个独立的单词。

    ```java
    .apply("ExtractWords", ParDo.of(new DoFn<String, String>() {
        @ProcessElement
        public void processElement(ProcessContext c) {
            // \p{L} denotes the category of Unicode letters,
            // so this pattern will match on everything that is not a letter.
            for (String word : c.element().split("[^\\p{L}]+")) {
                if (!word.isEmpty()) {
                    c.output(word);
                }
            }
        }
    }))
    ```

    ```py
    # The Flatmap transform is a simplified version of ParDo.
    {% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:examples_wordcount_minimal_pardo
    %}```

3.  SDK中提供的 `Count` transform（转换） 是一个通用的转换，接受一个任何类型的 `PCollection`，并返回一个键值对
    `PCollection`。每个 key 都代表输入集合中唯一的元素，每个值都代表 key在输入集合中出现的次数。

    在这个pipeline中， `Count`操作的输入是前一步 `ParDo` 转换生成的 `PCollection`，输出就是一个键值对的`PCollection`。key代表文本中出现的单词，value就是单词出现的次数。
   

    ```java
    .apply(Count.<String>perElement())
    ```

    ```py
    {% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:examples_wordcount_minimal_count
    %}```

4.  下一步的转换就是将单词出现次数的键值对格式化，转换成可以打印的字符串，写入到输出文件中。

    map transform（转换）是一个更高级别的复合 transform（转换），对于 输入`PCollection`中的每个元素，map
    transform（转换） 都会执行一个函数，然后一对一的输出这个元素。


    ```java
    .apply("FormatResults", MapElements.via(new SimpleFunction<KV<String, Long>, String>() {
        @Override
        public String apply(KV<String, Long> input) {
            return input.getKey() + ": " + input.getValue();
        }
    }))
    ```

    ```py
    {% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:examples_wordcount_minimal_map
    %}```

5.  一个文本文件 transform（转换）。这个 transform接受一个不可变的`PCollection`作为输入，这个 `PCollection`
    就是上一步输出的格式化字符串，并将每个元素写入到输出文本文件中。输入 `PCollection` 的每个元素都表示输出文件中
    的一行文本。

    ```java
    .apply(TextIO.write().to("wordcounts"));
    ```

    ```py
    {% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:examples_wordcount_minimal_write
    %}```

请注意，`Write` transform（转换） 产生了一个不重要的 `PDone` 类型的结果，在这个例子中被忽略了。


### 运行 pipeline（运行）

调用  `run` 方法运行 pipeline，将你的pipeline 发送给你在`PipelineOptions`中指定的 pipeline runner。


```java
p.run().waitUntilFinish();
```

```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:examples_wordcount_minimal_run
%}```

由于  `run` 方法是异步的。如果要同步执行，就调用 run 方法返回结果的
Note that the `run` method is asynchronous. For a blocking execution, call the
<span class="language-java">`waitUntilFinish`</span>
<span class="language-py">`wait_until_finish`</span> 方法 。

## WordCount example

这个 WordCount 例子介绍了一些推荐程序的实践，使得你的 pipeline 更容易的读取，写入和维护。虽然没有明确要求，他们有助于
使你的 pipeline 执行更加灵活，有助于测试你的 pipeline（管道），并帮助你让 pipelines 的代码可重用。

这一节假设你对构造一个 pipeline 的基础概念有一个很好的理解。如果你觉得你还没有到这个阶段，请阅读上一节，[Minimal WordCount](#minimalwordcount-example).


**用 Java运行这个例子:**

{:.runner-direct}
```
$ mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.WordCount \
     -Dexec.args="--inputFile=pom.xml --output=counts" -Pdirect-runner
```

{:.runner-apex}
```
$ mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.WordCount \
     -Dexec.args="--inputFile=pom.xml --output=counts --runner=ApexRunner" -Papex-runner
```

{:.runner-flink-local}
```
$ mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.WordCount \
     -Dexec.args="--runner=FlinkRunner --inputFile=pom.xml --output=counts" -Pflink-runner
```

{:.runner-flink-cluster}
```
$ mvn package exec:java -Dexec.mainClass=org.apache.beam.examples.WordCount \
     -Dexec.args="--runner=FlinkRunner --flinkMaster=<flink master> --filesToStage=target/word-count-beam-bundled-0.1.jar \
                  --inputFile=/path/to/quickstart/pom.xml --output=/tmp/counts" -Pflink-runner

你可以通过 Flink dashboard http://<flink master>:8081 监控这个运行的 job。
```

{:.runner-spark}
```
$ mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.WordCount \
     -Dexec.args="--runner=SparkRunner --inputFile=pom.xml --output=counts" -Pspark-runner
```

{:.runner-dataflow}
```
$ mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.WordCount \
     -Dexec.args="--runner=DataflowRunner --gcpTempLocation=gs://<your-gcs-bucket>/tmp \
                  --inputFile=gs://apache-beam-samples/shakespeare/* --output=gs://<your-gcs-bucket>/counts" \
     -Pdataflow-runner
```

完整的Java代码，可以参考
**[WordCount](https://github.com/apache/beam/blob/master/examples/java/src/main/java/org/apache/beam/examples/WordCount.java).**

**用 Python 运行这个例子:**

{:.runner-direct}
```
python -m apache_beam.examples.wordcount --input README.md --output counts
```

{:.runner-apex}
```
This runner is not yet available for the Python SDK.
```

{:.runner-flink-local}
```
This runner is not yet available for the Python SDK.
```

{:.runner-flink-cluster}
```
This runner is not yet available for the Python SDK.
```

{:.runner-spark}
```
This runner is not yet available for the Python SDK.
```

{:.runner-dataflow}
```
# As part of the initial setup, install Google Cloud Platform specific extra components.
pip install apache-beam[gcp]
python -m apache_beam.examples.wordcount --input gs://dataflow-samples/shakespeare/kinglear.txt \
                                         --output gs://<your-gcs-bucket>/counts \
                                         --runner DataflowRunner \
                                         --project your-gcp-project \
                                         --temp_location gs://<your-gcs-bucket>/tmp/
```

完整的Python代码, 可以参考
**[wordcount.py](https://github.com/apache/beam/blob/master/sdks/python/apache_beam/examples/wordcount.py).**

**新的概念:**

* Applying `ParDo` with an explicit `DoFn`
* Creating Composite Transforms （穿件复合的 Transforms（转换）操作）
* Using Parameterizable `PipelineOptions` （使用参数化的 `PipelineOptions`）

下面几节会详细描述这些概念，并将 pipeline 的代码拆分成更小的部分。


### 指定 显式的 DoFns

当使用 `ParDo` transforms（转换），你需要为输入的 `PCollection` 中的每个元素指定做什么样的处理操作。这个处理操作就是实现 SDK `DoFn` class的子类。你可以在每个 `ParDo` 转换内部创建一个 `DoFn` 子类，可以作为一个匿名的内部类，在前面的例子(Minimal WordCount)中完成。但是，将 `DoFn`定义成全局的，更容易吃，代码的可读性也更高。


```java
// In this example, ExtractWordsFn is a DoFn that is defined as a static class:

static class ExtractWordsFn extends DoFn<String, String> {
    ...

    @ProcessElement
    public void processElement(ProcessContext c) {
        ...
    }
}
```

```py
# In this example, the DoFns are defined as classes:

{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:examples_wordcount_wordcount_dofn
%}```

### 创建复合 transforms（转换）

如果你有一个处理操作包含多个 transform（转换）或者 `ParDo`步骤，你可以创建一个  `PTransform`的子类。
创建一个 `PTransform` 子类允许你封装复杂的 transforms（转换）操作，可以使得你的 pipeline（管道）结构更清晰和模块化，并且使得整体代码更容易测试。

在这个例子中，两个 transforms（转换）被封装成 `PTransform` 的子类 `CountWords`。`CountWords`包含用来运行 `ExtractWordsFn` 的 `ParDo` 转换和 SDK 提供的 `Count`转换。

定义 `CountWords` 时，我们指定其最终的输入和输出；输入就是 extraction 操作的`PCollection<String>`，
输出就是 count 操作的 `PCollection<KV<String, Long>>`。


```java
public static class CountWords extends PTransform<PCollection<String>,
    PCollection<KV<String, Long>>> {
  @Override
  public PCollection<KV<String, Long>> expand(PCollection<String> lines) {

    // Convert lines of text into individual words.
    PCollection<String> words = lines.apply(
        ParDo.of(new ExtractWordsFn()));

    // Count the number of times each word occurs.
    PCollection<KV<String, Long>> wordCounts =
        words.apply(Count.<String>perElement());

    return wordCounts;
  }
}

public static void main(String[] args) throws IOException {
  Pipeline p = ...

  p.apply(...)
   .apply(new CountWords())
   ...
}
```

```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:examples_wordcount_wordcount_composite
%}```

### 使用参数化的 PipelineOptions 选项

你可以在运行 pipeline 的时候将执行的选项写死在代码中。但是，更常见的方式是通过命令行参数解析来定义自己的配置选项。
通过命令行定义配置选项，可以使得代码更轻松地跨越不同的 runner。

添加要由命令行解析器处理的参数，并指定默认值，然后你可以给 pipeline 代码塞入选项值。


```java
public static interface WordCountOptions extends PipelineOptions {
  @Description("Path of the file to read from")
  @Default.String("gs://dataflow-samples/shakespeare/kinglear.txt")
  String getInputFile();
  void setInputFile(String value);
  ...
}

public static void main(String[] args) {
  WordCountOptions options = PipelineOptionsFactory.fromArgs(args).withValidation()
      .as(WordCountOptions.class);
  Pipeline p = Pipeline.create(options);
  ...
}
```

```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:examples_wordcount_wordcount_options
%}```

## Debugging WordCount example

Debugging WordCount 例子展示了检测你的 pipeline（管道）代码的一些好的实践。


**用Java运行这个例子:**

{:.runner-direct}
```
$ mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.DebuggingWordCount \
     -Dexec.args="--inputFile=pom.xml --output=counts" -Pdirect-runner
```

{:.runner-apex}
```
$ mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.DebuggingWordCount \
     -Dexec.args="--inputFile=pom.xml --output=counts --runner=ApexRunner" -Papex-runner
```

{:.runner-flink-local}
```
$ mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.DebuggingWordCount \
     -Dexec.args="--runner=FlinkRunner --inputFile=pom.xml --output=counts" -Pflink-runner
```

{:.runner-flink-cluster}
```
$ mvn package exec:java -Dexec.mainClass=org.apache.beam.examples.DebuggingWordCount \
     -Dexec.args="--runner=FlinkRunner --flinkMaster=<flink master> --filesToStage=target/word-count-beam-bundled-0.1.jar \
                  --inputFile=/path/to/quickstart/pom.xml --output=/tmp/counts" -Pflink-runner

你可以通过 Flink dashboard at http://<flink master>:8081 监控这个运行的job。
```

{:.runner-spark}
```
$ mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.DebuggingWordCount \
     -Dexec.args="--runner=SparkRunner --inputFile=pom.xml --output=counts" -Pspark-runner
```

{:.runner-dataflow}
```
$ mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.DebuggingWordCount \
   -Dexec.args="--runner=DataflowRunner --gcpTempLocation=gs://<your-gcs-bucket>/tmp \
                --inputFile=gs://apache-beam-samples/shakespeare/* --output=gs://<your-gcs-bucket>/counts" \
     -Pdataflow-runner
```

完整的Java代码，可以参照
[DebuggingWordCount](https://github.com/apache/beam/blob/master/examples/java/src/main/java/org/apache/beam/examples/DebuggingWordCount.java).

**用 Pyhton 运行这个例子:**

{:.runner-direct}
```
python -m apache_beam.examples.wordcount_debugging --input README.md --output counts
```

{:.runner-apex}
```
This runner is not yet available for the Python SDK.
```

{:.runner-flink-local}
```
This runner is not yet available for the Python SDK.
```

{:.runner-flink-cluster}
```
This runner is not yet available for the Python SDK.
```

{:.runner-spark}
```
This runner is not yet available for the Python SDK.
```

{:.runner-dataflow}
```
# As part of the initial setup, install Google Cloud Platform specific extra components.
pip install apache-beam[gcp]
python -m apache_beam.examples.wordcount_debugging --input gs://dataflow-samples/shakespeare/kinglear.txt \
                                         --output gs://<your-gcs-bucket>/counts \
                                         --runner DataflowRunner \
                                         --project your-gcp-project \
                                         --temp_location gs://<your-gcs-bucket>/tmp/
```

完整的Python代码，可以参照
**[wordcount_debugging.py](https://github.com/apache/beam/blob/master/sdks/python/apache_beam/examples/wordcount_debugging.py).**

**新的概念:**

* 打印日志
* 通过 `PAssert`测试你的 Pipeline。

以下部分将详细解释这些关键概念，并将 pipeline 代码分成更小的部分。


### 打印日志

每个 runner 都可以通过自己的方式处理日志。

```java
// This example uses .trace and .debug:

public class DebuggingWordCount {

  public static class FilterTextFn extends DoFn<KV<String, Long>, KV<String, Long>> {
    ...

    @ProcessElement
    public void processElement(ProcessContext c) {
      if (...) {
        ...
        LOG.debug("Matched: " + c.element().getKey());
      } else {
        ...
        LOG.trace("Did not match: " + c.element().getKey());
      }
    }
  }
}
```

```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/snippets/snippets.py tag:example_wordcount_debugging_logging
%}```


#### Direct Runner

当使用 `DirectRunner` 指定你的 pipeline，你可以直接将日志打印到本地 console。<span class="language-java">如果你使用Java版本的 Beam SDK，你必须添加 `Slf4j` 到你的class path。</span>


#### Cloud Dataflow Runner
当使用 `DataflowRunner` 执行你的 pipeline，你可以使用 Stackdriver Logging 打印日志。Stackdriver Logging 聚合所有 Cloud Dataflow 工作人员的日志到 Google Cloud Platform Console 一个单独的地方。你可以通过 Stackdriver Logging 搜索和获取所有工作人员在 Cloud Dataflow 已经完成的任务的日志。 当你的 pipeline 运行时，pipeline 中 `DoFn` 实例中的日志声明会在  Stackdriver Logging 中出现。

你可以控制工作台日志级别。 Cloud Dataflow 工作台制定使用者的代码，打印日志到 Stackdriver Logging 默认配置为 "INFO" 或者更高。你可以通过指定`--workerLogLevelOverrides={"Name1":"Level1","Name2":"Level2",...}`来覆盖 日志级别，定制化日志命名空间。 例如，指定 `--workerLogLevelOverrides={"org.apache.beam.examples":"DEBUG"}`，
当使用 Cloud Datasflow 服务执行一个 pipeline，在Stackdriver Logging 会只包含 "DEBUG" 或者更高的日志级别，默认是"INFO"或者更高的级别。

默认情况下，Cloud Dataflow 工作台日志配置可以通过指定 `--defaultWorkerLogLevel=<one of TRACE, DEBUG, INFO, WARN, ERROR>`来覆盖。例如，当通过 Cloud Dataflow 服务指定一个 pipeline 指定`--defaultWorkerLogLevel=DEBUG`，Cloud Logging会包含"DEBUG" 或者更高的级别。请注意，将默认的工作台日志级别更改为 TRACE 或者 DEBUG 会更加日志输出的量。


#### Apache Spark Runner

> **Note:** This section is yet to be added. There is an open issue for this
> ([BEAM-792](https://issues.apache.org/jira/browse/BEAM-792)).

#### Apache Flink Runner

> **Note:** This section is yet to be added. There is an open issue for this
> ([BEAM-791](https://issues.apache.org/jira/browse/BEAM-791)).

#### Apache Apex Runner

> **Note:** This section is yet to be added. There is an open issue for this
> ([BEAM-2285](https://issues.apache.org/jira/browse/BEAM-2285)).

###  通过 PAssert 测试你的pipeline。
`PAssert` 是一种方便的 PTransforms 集合，就是 Hamcrest 风格的集合匹配器，可以用来验证 PCollections 的内容。
`PAssert` 最好用于单元测试小数据集，但在这里被证明是一种教学工具。

下面，我们验证过滤的单词集合符合我们的预期计数。请注意， `PAssert`操作不会产生很多输出，并且只有当所有的期望都满足，pipeline才会成功。可以参阅
[DebuggingWordCountTest](https://github.com/apache/beam/blob/master/examples/java/src/test/java/org/apache/beam/examples/DebuggingWordCountTest.java)
作为单元测试的用例.

```java
public static void main(String[] args) {
  ...
  List<KV<String, Long>> expectedResults = Arrays.asList(
        KV.of("Flourish", 3L),
        KV.of("stomach", 1L));
  PAssert.that(filteredWords).containsInAnyOrder(expectedResults);
  ...
}
```

```py
# This feature is not yet available in the Beam SDK for Python.
```

## WindowedWordCount example

这个例子，`WindowedWordCount`，和之前的例子一样对文本的单词进行计数，但是介绍几个高级的概念。          

**New Concepts:**

* 无界和有界的 pipeline 输入模型。
* 添加 时间戳到模型中。
* 窗口
* Reusing PTransforms over windowed PCollections

以下部分将详细解释这些关键概念，并将 pipeline 代码分成更小的部分。

**用Java运行这个例子:**

{:.runner-direct}
```
$ mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.WindowedWordCount \
     -Dexec.args="--inputFile=pom.xml --output=counts" -Pdirect-runner
```

{:.runner-apex}
```
$ mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.WindowedWordCount \
     -Dexec.args="--inputFile=pom.xml --output=counts --runner=ApexRunner" -Papex-runner
```

{:.runner-flink-local}
```
$ mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.WindowedWordCount \
     -Dexec.args="--runner=FlinkRunner --inputFile=pom.xml --output=counts" -Pflink-runner
```

{:.runner-flink-cluster}
```
$ mvn package exec:java -Dexec.mainClass=org.apache.beam.examples.WindowedWordCount \
     -Dexec.args="--runner=FlinkRunner --flinkMaster=<flink master> --filesToStage=target/word-count-beam-bundled-0.1.jar \
                  --inputFile=/path/to/quickstart/pom.xml --output=/tmp/counts" -Pflink-runner

You can monitor the running job by visiting the Flink dashboard at http://<flink master>:8081
```

{:.runner-spark}
```
$ mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.WindowedWordCount \
     -Dexec.args="--runner=SparkRunner --inputFile=pom.xml --output=counts" -Pspark-runner
```

{:.runner-dataflow}
```
$ mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.WindowedWordCount \
   -Dexec.args="--runner=DataflowRunner --gcpTempLocation=gs://<your-gcs-bucket>/tmp \
                --inputFile=gs://apache-beam-samples/shakespeare/* --output=gs://<your-gcs-bucket>/counts" \
     -Pdataflow-runner
```

完整的Java代码，可以参阅
**[WindowedWordCount](https://github.com/apache/beam/blob/master/examples/java/src/main/java/org/apache/beam/examples/WindowedWordCount.java).**

> **Note:** WindowedWordCount 在Python SDK还无法使用。

### 无界和有界的 pipeline 输入模型。

Beam 允许你创建一个单一的 pipeline 同时处理有界和无界类型的输入。如果你的输入有固定数量的元素，就认为是一个 'bounded'数据集。如果你的输入是持续更新，就被认为是 'unbounded' 的，并且你必须使用一个支持流处理的 runner。


如果你的 pipeline 输入是有界的，那么所有下游的 PCollections 也会是有界的。同样的，如果输入是无界的，那么 pipeline 所有下游的PCollections也会是无界的，尽管部分分支可能是有界的。

回想一下，这个例子的输入是 Shakespeare 的文本集，是一组有限的数据集。因此，这个例子从文本文件中读取有界的数据:

```java
public static void main(String[] args) throws IOException {
    Options options = ...
    Pipeline pipeline = Pipeline.create(options);

    PCollection<String> input = pipeline
      .apply(TextIO.read().from(options.getInputFile()))

```

```py
# This feature is not yet available in the Beam SDK for Python.
```

### 添加时间戳到数据中。

`PCollection` 中的每个元素都有一个相关联的 [timestamp]({{ site.baseurl }}/documentation/programming-guide#element-timestamps).
每个元素的时间戳在 source 创建 `PCollection`的时候初始化创建。一些 sources 创建无界的 PCollections，每个新的元素都可以分配一个时间戳，当元素被read或者添加的时候。你可以使用 `DoFn`手动分配和调整时间戳；但是，你只能实时的 move 时间戳。

在这个例子中，输入是有界的。为了示例的目的， `DoFn`方法被命名为 `AddTimestampsFn`（被 `ParDo`调用），`AddTimestampsFn` 会为`PCollection`每个元素设置一个时间戳。


```java
.apply(ParDo.of(new AddTimestampFn(minTimestamp, maxTimestamp)));
```

```py
# This feature is not yet available in the Beam SDK for Python.
```
下面是 `AddTimestampFn` 的代码，`ParDo`调用 `DoFn`，为输入的元素设置 timestamp（时间戳）元素。例如，如果元素是日志行，`ParDo` 可以解析日志字符串的时间，并将这个时间设置为元素的时间戳。Shakespeare 工作空间内没有时间戳，所以在这个例子中我们会随机一些时间戳，为了说明这个概念。每个输入文本行，都会给予一个两小时间隔内的随机时间戳。


```java
static class AddTimestampFn extends DoFn<String, String> {
  private final Instant minTimestamp;
  private final Instant maxTimestamp;

  AddTimestampFn(Instant minTimestamp, Instant maxTimestamp) {
    this.minTimestamp = minTimestamp;
    this.maxTimestamp = maxTimestamp;
  }

  @ProcessElement
  public void processElement(ProcessContext c) {
    Instant randomTimestamp =
      new Instant(
          ThreadLocalRandom.current()
          .nextLong(minTimestamp.getMillis(), maxTimestamp.getMillis()));

    /**
     * Concept #2: Set the data element with that timestamp.
     */
    c.outputWithTimestamp(c.element(), new Instant(randomTimestamp));
  }
}
```

```py
# This feature is not yet available in the Beam SDK for Python.
```

### 窗口

Beam 使用叫做 **Windowing**  的概念将 `PCollection` 细分为有界的数据集。PTransforms聚合多个用来处理  `PCollection`的元素，有限的窗口，尽管整体集合看上去可能是是无界的。

`WindowedWordCount` 例子使用 fixed-time 窗口，每个窗口都表示一个固定时间间隔。这个例子的固定窗口大小是一分钟(你可以通过命令行选项来改变)。


```java
PCollection<String> windowedWords = input
  .apply(Window.<String>into(
    FixedWindows.of(Duration.standardMinutes(options.getWindowSize()))));
```

```py
# This feature is not yet available in the Beam SDK for Python.
```

### Reusing PTransforms over windowed PCollections

你可以重用已经存在的 PTransforms，这是为了在窗口 PCollections 操作简单的 PCollections。


```java
PCollection<KV<String, Long>> wordCounts = windowedWords.apply(new WordCount.CountWords());
```

```py
# This feature is not yet available in the Beam SDK for Python.
```

### 将结果写入到无界的 sink

当你的输入是无界的，同样输出的`PCollection`也是无界的。我们需要确定一个合适的，无界的 sink。一些输出 sinks只支持有界的输出，其他的sink同时支持有界和无界的输出。 通过使用一个 `FilenamePolicy`，我们可以使用  `TextIO` 通过窗口对文件进行分区。我们使用一个复合的 `PTransform` ，在内部使用这样一个策略，一个窗口写一个分片文件。

在这个例子中，我们将数据流式的传输到 Google BigQuery。下面代码格式化结果，并使用 `BigQueryIO.Write` 将结果写入到 BigQuery 表中去。


```java
  wordCounts
      .apply(MapElements.via(new WordCount.FormatAsTextFn()))
      .apply(new WriteOneFilePerWindow(output, options.getNumShards()));
```

```py
# This feature is not yet available in the Beam SDK for Python.
```

