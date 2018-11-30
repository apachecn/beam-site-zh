---
layout: default
title: "Apache Apex Runner"
permalink: /documentation/runners/apex/
---

# 使用 Apache Apex Runner

Apex Runner 使用 [Apache Apex](http://apex.apache.org/) 作为底层引擎执行 Apache Beam Pipeline.
该 runner 针对 [Beam 模型, 支持流与批处理 Pipelines]({{ site.baseurl }}/documentation/runners/capability-matrix/) 有着较为广泛的支持.

[Apache Apex](http://apex.apache.org/) 是一个流处理平台和框架, 它用于 Apache Hadoop 上的低延迟, 高吞吐量与容错分析的应用程序.
Apex 具有统一的流架构，可用于实时和批处理。

## Apex Runner 的先决条件

您可以部署您自己的 Hadoop 集群.
Beam 不需要任何额外的东西在 YARN 上启动 Pipeline.
可选的 Apex 安装可能对监视和故障排除是有用的.
Apex CLI 可以被 [built](http://apex.apache.org/docs/apex/apex_development_setup/) 或者作为 [binary build](http://www.atrato.io/blog/2017/04/08/apache-apex-cli/) 来获得。
获得更多的下载选项请参阅 [Apache Apex 网站上的发布信息](http://apex.apache.org/downloads.html).

## 使用 Apex Runner 运行 wordcount 

将待处理的数据放到 HDFS:

```
hdfs dfs -mkdir -p /tmp/input/
hdfs dfs -put pom.xml /tmp/input/
```

在 HDFS 上不应该存在输出目录:

```
hdfs dfs -rm -r -f /tmp/output/
```

运行 wordcount 示例（*示例项目需要修改，以包括 HDFS 文件 provider*）

```
mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.WordCount -Dexec.args="--inputFile=/tmp/input/pom.xml --output=/tmp/output/ --runner=ApexRunner --embeddedExecution=false --configFile=beam-runners-apex.properties" -Papex-runner
```

应用程序将异步运行. 
使用 `yarn application -list -appStates ALL` 来检查状态.

配置文件是可选的，它可以用来影响如何将 Apex operators 部署到 YARN 容器中.
下面的示例将通过将 operators 配置到同一个容器中，并降低每个 operator 的堆内存，从而减少所需容器的数量 — 这适用于单个节点 Hadoop sandbox 中的执行.

```
apex.application.*.operator.*.attr.MEMORY_MB=64
apex.stream.*.prop.locality=CONTAINER_LOCAL
apex.application.*.operator.*.attr.TIMEOUT_WINDOW_COUNT=1200
```


## 检查输出

检查 pipeline 在 HDFS 响应位置中的输出.

```
hdfs dfs -ls /tmp/output/
```

## 监控 job 的进度

根据您的安装方式，您也许可以监控 Hadoop集群上的 job 的进度.
或者，您有以下选项:

* YARN : 使用 YARN web UI, 通常在运行 Resource Manager 节点的 8088 上运行.
* Apex 命令行接口: [使用 Apex 命令行来获取正在运行的 application（应用）信息](http://apex.apache.org/docs/apex/apex_cli/#apex-cli-commands).

