---
layout: default
title: "Apache Apex Runner"
permalink: /documentation/runners/apex/
---
# 使用Apache Apex Runner

 Apex Runner 使用 [Apache Apex](http://apex.apache.org/) 作为底层引擎执行 Apache Beam Pipeline。The runner 对[Beam 模型和支持流和批处理Pipeline]有广泛的支持({{ site.baseurl }}/documentation/runners/capability-matrix/).

[Apache Apex](http://apex.apache.org/) 是用于 Apache Hadoop 上的低延迟、高吞吐量和容错分析应用的流处理平台和框架。Apex 具有统一的流架构，可用于实时和批处理。

## Apex Runner的先决条件

您可以建立自己的 Hadoop 集群。Beam 不需要任何额外的东西来启动管道的 YARN.
可选的 Apex 安装可能对监视和故障排除有用。
The Apex CLI可以被 [built](http://apex.apache.org/docs/apex/apex_development_setup/) 或者作为 [binary build](http://www.atrato.io/blog/2017/04/08/apache-apex-cli/) 来获得。
获得更多的下载选项请参阅 [distribution information on the Apache Apex website](http://apex.apache.org/downloads.html).

## 使用 Apex Runner 运行 wordcount 

将数据处理为 HDFS:
```
hdfs dfs -mkdir -p /tmp/input/
hdfs dfs -put pom.xml /tmp/input/
```

在HDFS上不应该存在输出目录:
```
hdfs dfs -rm -r -f /tmp/output/
```

运行 wordcount 示例(*示例项目需要修改，以包括HDFS文件提供程序*)
```
mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.WordCount -Dexec.args="--inputFile=/tmp/input/pom.xml --output=/tmp/output/ --runner=ApexRunner --embeddedExecution=false --configFile=beam-runners-apex.properties" -Papex-runner
```

应用程序将异步运行。 用 `yarn application -list -appStates ALL` 来检查状态。

配置文件是可选的，它可以用来影响如何将Apex运算符部署到 YARN 容器中。
下面的示例将通过将操作符配置到同一个容器中，并降低每个操作符的堆内存，从而减少所需容器的数量——这适用于单个节点 Hadoop sandbox 中的执行。

```
apex.application.*.operator.*.attr.MEMORY_MB=64
apex.stream.*.prop.locality=CONTAINER_LOCAL
apex.application.*.operator.*.attr.TIMEOUT_WINDOW_COUNT=1200
```


## 检查输出

检查管道在HDFS位置的输出
```
hdfs dfs -ls /tmp/output/
```

## 监视你工作的进程

根据您的安装，您可能能够监视您在Hadoop集群上的工作进度。或者，您有以下选项:

* YARN : 在运行资源管理器的节点上，通常使用 YARN web UI 在8088上运行。
* Apex 命令行界面: [Using the Apex CLI to get running application information](http://apex.apache.org/docs/apex/apex_cli/#apex-cli-commands).

