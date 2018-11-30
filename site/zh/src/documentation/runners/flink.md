---
layout: default
title: "Apache Flink Runner"
permalink: /documentation/runners/flink/
redirect_from: /learn/runners/flink/
---
使用 Apache Flink Runner

<nav class="language-switcher">
  <strong>适用于:</strong>
  <ul>
    <li data-type="language-java">Java SDK</li>
    <li data-type="language-py">Python SDK</li>
  </ul>
</nav>

The Apache Flink Runner  可用于使用 Apache Flink 执行 Beam 管道。使用 Flink Runner 时，您将创建一个包含可以在常规 Flink 群集上执行的作业的 jar 文件。也可以使用Flink的本地执行模式执行 Beam pipeline。这有助于您的 pipeline 的开发和调试.

Flink Runner 和 Flink 适用于大规模，连续工作，并提供：

* 支持批量处理和流数据处理的流式优先运行时
* 同时支持极高吞吐量和低事件延迟的运行时
* 容错和恰好一次的处理保证
* 在流处理程序中天生支持"背压"
* 自定义内存管理，用于在内存和外核之间数据处理算法中进行高效，可靠的切换
* 与 YARN 和 Apache Hadoop 生态系统的其他组件无缝集成

The Beam Capability Matrix 记录了 Flink Runne r的支持容量。
## Flink Runner 先决条件和设置

如果要使用 Flink runner 的本地执行模式不必完成任何设置。

要使用 Flink Runner 在群集上执行，您必须按照 Flink 安装快速入门设置Flink群集。

要了解您需要哪个版本的 Flink，您可以运行此命令来检查项目使用的 Flink 依赖关系的版本：
```
$ mvn dependency:tree -Pflink-runner |grep flink
...
[INFO] |  +- org.apache.flink:flink-streaming-java_2.10:jar:1.2.1:runtime
...
```
在这里，我们需要 Flink 1.2.1。请注意依赖关系名称中的 Scala 版本。在这种情况下，我们需要确保在 Scala 版本2.10中使用 Flink 集群。

有关更多信息，Flink 文档可能会有所帮助。

### 指定你的依赖

<span class="language-java">当使用 java 时, 在你的 pom.xml 文件中必须指定 Flink Runner 的依赖关系.</span>
```java
<dependency>
  <groupId>org.apache.beam</groupId>
  <artifactId>beam-runners-flink_2.10</artifactId>
  <version>{{ site.release_latest }}</version>
  <scope>runtime</scope>
</dependency>
```

<span class="language-py">本节不适用 于Python 的 Beam SDK.</span>

## 在 Flink 集群上执行管道

为了在Flink集群上执行一个管道，您需要将程序打包在一个所谓的大而全的 jar 包中。这样做取决于您的构建系统，但如果沿着“ 快速入门 ” （Beam Quickstart） 运行，则必须运行以下命令：

```
$ mvn package -Pflink-runner
```
Beam Quickstart Maven 项目设置为使用 Maven Shade 插件来创建一个大而全的 jar，该 -Pflink-runner 参数确保包含对 Flink Runner 的依赖。

对于实际运行管道，您将使用此命令
```
$ mvn exec:java -Dexec.mainClass=org.apache.beam.examples.WordCount \
    -Pflink-runner \
    -Dexec.args="--runner=FlinkRunner \
      --inputFile=/path/to/pom.xml \
      --output=/path/to/counts \
      --flinkMaster=<flink master url> \
      --filesToStage=target/word-count-beam--bundled-0.1.jar"
```
如果你有一个 Flink JobManager 在本地机器上运行，对于 flinkMaster 你可以设置为 localhost:6123。

## Flink Runner 管道可选项

使用 Flink Runner 执行管道时，可以设置这些管道选项。

<table class="table table-bordered">
<tr>
  <th>字段</th>
  <th>描述</th>
  <th>默认值</th>
</tr>
<tr>
  <td><code>runner</code></td>
  <td>使用The pipeline runner 时. 这个选项可以决定使用哪一种 runner 来运行.</td>
  <td>设置 <code>FlinkRunner</code>使用 Flink 运行.</td>
</tr>
<tr>
  <td><code>streaming</code></td>
  <td>流模式是启用还是关闭; <code>true</code> 如果启用. 设为 <code>true</code> 如果运行无限制的管道 <code>PCollection</code>s.</td>
  <td><code>false</code></td>
</tr>
<tr>
  <td><code>flinkMaster</code></td>
  <td>要执行管道的 Flink JobManager 的URL. 这可以是集群 JobManager 的地址，也可以是 <code>"host:port"</code> 特殊字符串 <code>"[local]"</code> 或者<code>"[auto]"</code>. <code>"[local]"</code>将启动 JVM 中的本地 Flink 群集，同时 <code>"[auto]"</code>让系统根据环境决定执行管道的位置。</td>
  <td><code>[auto]</code></td>
</tr>
<tr>
  <td><code>filesToStage</code></td>
  <td>Jar 文件发送到所有的 workers 节点并且放到 classpath 中. 这里必须把应用包含的所有依赖放到大而全的jar文件中.</td>
  <td>empty</td>
</tr>

<tr>
  <td><code>parallelism</code></td>
  <td>将操作分配给 workers 时使用的并行程度.</td>
  <td><code>1</code></td>
</tr>
<tr>
  <td><code>checkpointingInterval</code></td>
  <td>连续检查点之间的间隔（即用于容错的当前管道状态的快照）.</td>
  <td><code>-1L</code>, 等等关闭</td>
</tr>
<tr>
  <td><code>numberOfExecutionRetries</code></td>
  <td>设置重新执行任务失败的次数. <code>0</code> 是有效的禁用容错的价值. <code>-1</code> 表示应该使用系统默认值（在配置中定义）</td>
  <td><code>-1</code></td>
</tr>
<tr>
  <td><code>executionRetryDelay</code></td>
  <td>设置执行之间的延迟。<code>-1</code表示应该使用默认值</td>
  <td><code>-1</code></td>
</tr>
<tr>
  <td><code>stateBackend</code></td>
  <td>	将状态设置为在流式传输模式下使用。默认是从Flink配置中读取此设置。</td>
  <td><code>empty</code>, 即从 Flink 配置中读取</td>
</tr>
</table>

有关管道配置选项的完整列表，请参阅 FlinkPipelineOptions PipelineOptions 接口（及其子接口）的参考文档 。

附加信息和注意事项

### 监控job的状态

您可以使用 Flink JobManager 仪表板监视正在运行的 Flink 作业。默认情况下,  `8081` 在 JobManager 节点的端口可用。如果在你的本地机器安装了Flink ，则可以访问`http://localhost:8081`.

流式执行

如果您的管道使用无界数据源或接收器，Flink Runner 将自动切换到流模式。   你可以使用上述 `streaming` 设置强制执行流式传输模式。

