---
layout: default
title: "Direct Runner"
permalink: /documentation/runners/direct/
redirect_from: /learn/runners/direct/
---
# 使用 Direct Runner

<nav class="language-switcher">
  <strong>适用于:</strong>
  <ul>
    <li data-type="language-java" class="active">Java SDK</li>
    <li data-type="language-py">Python SDK</li>
  </ul>
</nav>

Direct Runner在您的机器上执行pipelines，旨在尽可能接近地验证pipelines是否遵守Apache Beam模型。 Direct Runner不是专注于高效的pipeline执行，而是执行额外的检查，以确保用户不依赖模型不能保证的语义。其中一些检查包括:

* 强化元素的不变性
* 强制元素的编码
* 元素在所有点以任意顺序进行处理
* 用户功能的序列化 (`DoFn`, `CombineFn`等.)

使用Direct Runner进行测试和开发有助于确保pipelines在不同Beam runners之间是稳健的。另外，当在远程集群上执行pipeline时，调试失败运行可能是一项非常简单的任务。相反，在pipeline代码上执行本地单元测试通常会更快更简单。在本地单元测试您的pipeline还允许您使用您首选的本地调试工具。

以下是有关如何测试pipelines的信息的一些资源。
<ul>
  <!-- Java specific links -->
  <li class="language-java"><a href="{{ site.baseurl }}/blog/2016/10/20/test-stream.html">Testing Unbounded Pipelines in Apache Beam</a> 谈论了使用Java类 <a href="{{ site.baseurl }}/documentation/sdks/javadoc/{{ site.release_latest }}/index.html?org/apache/beam/sdk/testing/PAssert.html">PAssert</a> 和 <a href="{{ site.baseurl }}/documentation/sdks/javadoc/{{ site.release_latest }}/index.html?org/apache/beam/sdk/testing/TestStream.html">TestStream</a> 来测试您的pipelines。</li>
  <li class="language-java">The <a href="{{ site.baseurl }}/get-started/wordcount-example/#testing-your-pipeline-via-passert">Apache Beam WordCount Example</a> 包含了使用 <a href="{{ site.baseurl }}/documentation/sdks/javadoc/{{ site.release_latest }}/index.html?org/apache/beam/sdk/testing/PAssert.html"><code>PAssert</code></a>记录和测试pipeline的示例。</li>

  <!-- Python specific links -->
  <li class="language-py">您可以使用 <a href="https://github.com/apache/beam/blob/master/sdks/python/apache_beam/testing/util.py#L76">assert_that</a> 来测试您的pipeline. Python <a href="https://github.com/apache/beam/blob/master/sdks/python/apache_beam/examples/wordcount_debugging.py">WordCount Debugging Example</a> 包含了使用<code>assert_that</code>进行日志记录和测试的示例。</li>
</ul>

## Direct Runner 先决条件和设置

### 指定你的依赖

<span class="language-java">使用Java时，必须指定您的`pom.xml`中的Direct Runner的依赖关系。 </span>
```java
<dependency>
   <groupId>org.apache.beam</groupId>
   <artifactId>beam-runners-direct-java</artifactId>
   <version>{{ site.release_latest }}</version>
   <scope>runtime</scope>
</dependency>
```

<span class="language-py">本节不适用于Python的Beam SDK。</span>

## Direct Runner的 Pipeline 选项

从命令行执行Pipeline时, 将 `runner` 设置为 `direct` 或者是 `DirectRunner`。 其他Pipeline选项的默认值通常就足够了。

请参阅参考文档
<span class="language-java">[`DirectOptions`]({{ site.baseurl }}/documentation/sdks/javadoc/{{ site.release_latest }}/index.html?org/apache/beam/runners/direct/DirectOptions.html)</span>
<span class="language-py">[`DirectOptions`]({{ site.baseurl }}/documentation/sdks/pydoc/{{ site.release_latest }}/apache_beam.options.html#apache_beam.options.pipeline_options.DirectOptions)</span>
用于默认值和其他管道配置选项的接口.

## 附加信息和注意事项

本地执行受本地环境中可用的内存限制。 强烈建议您使用足够小的数据集来运行管道，以适应本地内存。  您可以创建一个小内存数据集使用<span class="language-java">[`Create`]({{ site.baseurl }}/documentation/sdks/javadoc/{{ site.release_latest }}/index.html?org/apache/beam/sdk/transforms/Create.html)</span><span class="language-py">[`Create`](https://github.com/apache/beam/blob/master/sdks/python/apache_beam/transforms/core.py)</span> transform, 或者您可以使用 <span class="language-java">[`Read`]({{ site.baseurl }}/documentation/sdks/javadoc/{{ site.release_latest }}/index.html?org/apache/beam/sdk/io/Read.html)</span><span class="language-py">[`Read`](https://github.com/apache/beam/blob/master/sdks/python/apache_beam/io/iobase.py)</span> transform 来处理小型本地或远程文件。

