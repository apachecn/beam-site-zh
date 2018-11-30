---
layout: default
title: "Apache Spark Runner"
permalink: /documentation/runners/spark/
redirect_from: /learn/runners/spark/
---
# Apache Spark Runner的使用

Apache Spark Runner可以使用[Apache Spark](http://spark.apache.org/). 来执行管道(Pipeline)。
Spark Runner可以像本机Spark应用程序那样执行Spark管道;为本地模式部署一个自包含的应用程序，在Spark的独立RM上运行，或者使用纱线或Mesos。

Spark为Spark Runner执行Beam pipelines提供支持如下：

* 补丁和Streaming pipelines 或者combined pipelines;
* 由RDDs和DStreams提供的相同的容错能力(fault-tolerance [guarantees](http://spark.apache.org/docs/1.6.3/streaming-programming-guide.html#fault-tolerance-semantics));
* 与Spark相同的安全特性([security](http://spark.apache.org/docs/1.6.3/security.html))；
* **Built-in metrics reporting using Spark's metrics system, which reports Beam Aggregators as well.使用Spark的度量系统的内置度量报告，它还报告Beam 聚合。**
* 通过spark的广播变量对Beam的侧输入原生支持。

[Beam Capability Matrix]({{ site.baseurl }}/documentation/runners/capability-matrix/)文档记录了Spark Runner当前支持的功能。

_**注:**_支持流媒体流中的Beam模型(Beam Model)目前处于实验性阶段，请在邮件列表中跟踪状态发展。[mailing list]({{ site.baseurl }}/get-started/support/)

## Spark Runner 先决条件和设置

The Spark runner currently supports Spark's 1.6 branch, and more specifically any version greater than 1.6.0.

You can add a dependency on the latest version of the Spark runner by adding to your pom.xml the following:
```java
<dependency>
  <groupId>org.apache.beam</groupId>
  <artifactId>beam-runners-spark</artifactId>
  <version>{{ site.release_latest }}</version>
</dependency>
```

### Deploying Spark with your application

In some cases, such as running in local mode/Standalone, your (self-contained) application would be required to pack Spark by explicitly adding the following dependencies in your pom.xml:
```java
<dependency>
  <groupId>org.apache.spark</groupId>
  <artifactId>spark-core_2.10</artifactId>
  <version>${spark.version}</version>
</dependency>

<dependency>
  <groupId>org.apache.spark</groupId>
  <artifactId>spark-streaming_2.10</artifactId>
  <version>${spark.version}</version>
</dependency>
```

And shading the application jar using the maven shade plugin:
```java
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-shade-plugin</artifactId>
  <configuration>
    <createDependencyReducedPom>false</createDependencyReducedPom>
    <filters>
      <filter>
        <artifact>*:*</artifact>
        <excludes>
          <exclude>META-INF/*.SF</exclude>
          <exclude>META-INF/*.DSA</exclude>
          <exclude>META-INF/*.RSA</exclude>
        </excludes>
      </filter>
    </filters>
  </configuration>
  <executions>
    <execution>
      <phase>package</phase>
      <goals>
        <goal>shade</goal>
      </goals>
      <configuration>
        <shadedArtifactAttached>true</shadedArtifactAttached>
        <shadedClassifierName>shaded</shadedClassifierName>
      </configuration>
    </execution>
  </executions>
</plugin>
```

After running <code>mvn package</code>, run <code>ls target</code> and you should see (assuming your artifactId is `beam-examples` and the version is `1.0.0`):
```
beam-examples-1.0.0-shaded.jar
```

To run against a Standalone cluster simply run:
```
spark-submit --class com.beam.examples.BeamPipeline --master spark://HOST:PORT target/beam-examples-1.0.0-shaded.jar --runner=SparkRunner
```

### Running on a pre-deployed Spark cluster

Deploying your Beam pipeline on a cluster that already has a Spark deployment (Spark classes are available in container classpath) does not require any additional dependencies.
For more details on the different deployment modes see: [Standalone](http://spark.apache.org/docs/latest/spark-standalone.html), [YARN](http://spark.apache.org/docs/latest/running-on-yarn.html), or [Mesos](http://spark.apache.org/docs/latest/running-on-mesos.html).

## Pipeline options for the Spark Runner

When executing your pipeline with the Spark Runner, you should consider the following pipeline options.

<table class="table table-bordered">
<tr>
  <th>Field</th>
  <th>Description</th>
  <th>Default Value</th>
</tr>
<tr>
  <td><code>runner</code></td>
  <td>The pipeline runner to use. This option allows you to determine the pipeline runner at runtime.</td>
  <td>Set to <code>SparkRunner</code> to run using Spark.</td>
</tr>
<tr>
  <td><code>sparkMaster</code></td>
  <td>The url of the Spark Master. This is the equivalent of setting <code>SparkConf#setMaster(String)</code> and can either be <code>local[x]</code> to run local with x cores, <code>spark://host:port</code> to connect to a Spark Standalone cluster, <code>mesos://host:port</code> to connect to a Mesos cluster, or <code>yarn</code> to connect to a yarn cluster.</td>
  <td><code>local[4]</code></td>
</tr>
<tr>
  <td><code>storageLevel</code></td>
  <td>The <code>StorageLevel</code> to use when caching RDDs in batch pipelines. The Spark Runner automatically caches RDDs that are evaluated repeatedly. This is a batch-only property as streaming pipelines in Beam are stateful, which requires Spark DStream's <code>StorageLevel</code> to be <code>MEMORY_ONLY</code>.</td>
  <td>MEMORY_ONLY</td>
</tr>
<tr>
  <td><code>batchIntervalMillis</code></td>
  <td>The <code>StreamingContext</code>'s <code>batchDuration</code> - setting Spark's batch interval.</td>
  <td><code>1000</code></td>
</tr>
<tr>
  <td><code>enableSparkMetricSinks</code></td>
  <td>Enable reporting metrics to Spark's metrics Sinks.</td>
  <td>true</td>
</tr>
</table>

## Additional notes

### Using spark-submit

When submitting a Spark application to cluster, it is common (and recommended) to use the <code>spark-submit</code> script that is provided with the spark installation.
The <code>PipelineOptions</code> described above are not to replace <code>spark-submit</code>, but to complement it.
Passing any of the above mentioned options could be done as one of the <code>application-arguments</code>, and setting <code>--master</code> takes precedence.
For more on how to generally use <code>spark-submit</code> checkout Spark [documentation](http://spark.apache.org/docs/1.6.3/submitting-applications.html#launching-applications-with-spark-submit).

### Monitoring your job

You can monitor a running Spark job using the Spark [Web Interfaces](http://spark.apache.org/docs/1.6.3/monitoring.html#web-interfaces). By default, this is available at port `4040` on the driver node. If you run Spark on your local machine that would be `http://localhost:4040`.
Spark also has a history server to [view after the fact](http://spark.apache.org/docs/1.6.3/monitoring.html#viewing-after-the-fact).
Metrics are also available via [REST API](http://spark.apache.org/docs/1.6.3/monitoring.html#rest-api).
Spark provides a [metrics system](http://spark.apache.org/docs/1.6.3/monitoring.html#metrics) that allows reporting Spark metrics to a variety of Sinks. The Spark runner reports user-defined Beam Aggregators using this same metrics system and currently supports <code>GraphiteSink</code> and <code>CSVSink</code>, and providing support for additional Sinks supported by Spark is easy and straight-forward.

### Streaming Execution

If your pipeline uses an <code>UnboundedSource</code> the Spark Runner will automatically set streaming mode. Forcing streaming mode is mostly used for testing and is not recommended.

### Using a provided SparkContext and StreamingListeners

If you would like to execute your Spark job with a provided <code>SparkContext</code>, such as when using the [spark-jobserver](https://github.com/spark-jobserver/spark-jobserver), or use <code>StreamingListeners</code>, you can't use <code>SparkPipelineOptions</code> (the context or a listener cannot be passed as a command-line argument anyway).
Instead, you should use <code>SparkContextOptions</code> which can only be used programmatically and is not a common <code>PipelineOptions</code> implementation.
