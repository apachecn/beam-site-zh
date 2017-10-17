---
layout: default
title: "Beam 发行版本"
permalink: get-started/downloads/
redirect_from:
  - /get-started/releases/
  - /use/releases/
  - /releases/
---

# Apache Beam&#8482; 下载

使用Apache Beam 最简单的方法就是通过中央仓库获取一个发布版本。可以使用Java SDK 或 Python SDK 从 [Maven Central Repository](https://search.maven.org/#search%7Cga%7C1%7Cg%3A%22org.apache.beam%22) 和 [PyPI](https://pypi.python.org/pypi/apache-beam) 获取.

例如，如果正在使用Maven项目管理工具，并想使用Java SDK 执行 `DirectRunner`，则添加如下依赖关系到 `pom.xml` 文件：

    <dependency>
      <groupId>org.apache.beam</groupId>
      <artifactId>beam-sdks-java-core</artifactId>
      <version>{{ site.release_latest }}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.beam</groupId>
      <artifactId>beam-runners-direct-java</artifactId>
      <version>{{ site.release_latest }}</version>
      <scope>runtime</scope>
    </dependency>

类似地，如果使用PypI并想使用Python SDK执行 `DirectRunner`, 请添加一下字段到 `setup.py` 文件:

    apache-beam=={{ site.release_latest }}

另外，您可能需要依赖其他SDK模块（如IO连接器或其他扩展）以及额外runners来批量执行pipeline。


## API 稳定性
Apache Beam 采用[semantic versioning](http://semver.org/) 语义化版本。版本号编写标准如下：
`major.minor.incremental`，即 **主版本.次版本.补丁版本** ，版本号递增按照如下情形:


* 主版本号：发生不兼容API增加或修改时递增；
* 次版本号：发生向后兼容的新功能的增加时递增；
* 补丁版本号：发生向前兼容的Bug修改是递增；

请注意标记[`@Experimental`]的APIs，他们可能在随时被修改，所以无法保证不同版本之的兼容性。

特别地，在稳定版本发布前，任何API都可以随时发生改变，则处于该阶段的不同版本号记作：`0.x.y`。
>*即主版本号为零（0.yz）的软件处于开发初始阶段，一切都可能随时被改变。这样的公共API 不应该被视为稳定版。*

## 版本发布

### 2.1.0 (2017-08-23)
官方源代码下载
Official [source code download](https://www.apache.org/dyn/closer.cgi?filename=beam/2.1.0/apache-beam-2.1.0-source-release.zip&action=download).


[发行说明 Release notes](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12319527&version=12340528).

### 2.0.0 (2017-05-17)
官方源代码下载
Official [source code download](https://www.apache.org/dyn/closer.cgi?filename=beam/2.0.0/apache-beam-2.0.0-source-release.zip&action=download).

[发行说明 Release notes](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12319527&version=12339746).

### 0.6.0 (2017-03-11)
官方源代码下载
Official [source code download](https://www.apache.org/dyn/closer.cgi?filename=beam/0.6.0/apache-beam-0.6.0-source-release.zip&action=download).

[发行说明 Release notes](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12319527&version=12339256).

### 0.5.0 (2017-02-02)
官方源代码下载
Official [source code download](https://www.apache.org/dyn/closer.cgi?filename=beam/0.5.0/apache-beam-0.5.0-source-release.zip&action=download).

[发行说明 Release notes](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12319527&version=12338859).

### 0.4.0 (2016-12-29)
官方源代码下载
Official [source code download](https://www.apache.org/dyn/closer.cgi?filename=beam/0.4.0/apache-beam-0.4.0-source-release.zip&action=download).

[发行说明 Release notes](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12319527&version=12338590).

### 0.3.0-incubating (2016-10-31)
官方源代码下载
Official [source code download](https://www.apache.org/dyn/closer.cgi?filename=beam/0.3.0-incubating/apache-beam-0.3.0-incubating-source-release.zip&action=download).

[发行说明 Release notes](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12319527&version=12338051).

### 0.2.0-incubating (2016-08-08)
官方源代码下载
Official [source code download](https://www.apache.org/dyn/closer.cgi?filename=beam/0.2.0-incubating/apache-beam-0.2.0-incubating-source-release.zip&action=download).

[发行说明 Release notes](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12319527&version=12335766).

### 0.1.0-incubating (2016-06-15)
官方源代码下载
Official [source code download](https://www.apache.org/dyn/closer.cgi?filename=beam/0.1.0-incubating/apache-beam-0.1.0-incubating-source-release.zip&action=download).

这是Apache Beam的第一个孵化版本。
