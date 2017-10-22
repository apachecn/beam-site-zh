---
layout: default
title: "Beam Releases"
permalink: get-started/downloads/
redirect_from:
  - /get-started/releases/
  - /use/releases/
  - /releases/
---

# Apache Beam&#8482; 下载

下载 Apache Beam 最简单的方式就是通过Apache Beam 中心仓库获取一份稳定地发行版本。并可以利用[Maven Central Repository](https://search.maven.org/#search%7Cga%7C1%7Cg%3A%22org.apache.beam%22) 或[PyPI](https://pypi.python.org/pypi/apache-beam)获取Java SDK 或 Python SDK 开发工具。 

例如，如果你正在利用 Maven 开发并想在 Java SDK 运行`DirectRunner`,则可以将依赖按照如下添加到配置文件`pom.xml`中：

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

类似地，如果利用PyPI, 使用python 执行`DirectRunner`，则需要添加如下需求到`setup.py`文件中：

    apache-beam=={{ site.release_latest }}

Additionally, you may want to depend on additional SDK modules, such as IO
connectors or other extensions, and additional runners to execute your pipeline
at scale.

## API Stability

Apache Beam uses [semantic versioning](http://semver.org/). Version numbers use the form `major.minor.incremental` and are incremented as follows:

* major version for incompatible API changes
* minor version for new functionality added in a backward-compatible manner
* incremental version for forward-compatible bug fixes

Please note that APIs marked [`@Experimental`]({{ site.baseurl }}/documentation/sdks/javadoc/{{ site.release_latest }}/org/apache/beam/sdk/annotations/Experimental.html)
may change at any point and are not guaranteed to remain compatible across versions.

Additionally, any API may change before the first stable release, i.e., between versions denoted `0.x.y`.

## Releases

### 2.1.0 (2017-08-23)
Official [source code download](https://www.apache.org/dyn/closer.cgi?filename=beam/2.1.0/apache-beam-2.1.0-source-release.zip&action=download).

[Release notes](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12319527&version=12340528).

### 2.0.0 (2017-05-17)
Official [source code download](https://www.apache.org/dyn/closer.cgi?filename=beam/2.0.0/apache-beam-2.0.0-source-release.zip&action=download).

[Release notes](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12319527&version=12339746).

### 0.6.0 (2017-03-11)
Official [source code download](https://www.apache.org/dyn/closer.cgi?filename=beam/0.6.0/apache-beam-0.6.0-source-release.zip&action=download).

[Release notes](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12319527&version=12339256).

### 0.5.0 (2017-02-02)
Official [source code download](https://www.apache.org/dyn/closer.cgi?filename=beam/0.5.0/apache-beam-0.5.0-source-release.zip&action=download).

[Release notes](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12319527&version=12338859).

### 0.4.0 (2016-12-29)
Official [source code download](https://www.apache.org/dyn/closer.cgi?filename=beam/0.4.0/apache-beam-0.4.0-source-release.zip&action=download).

[Release notes](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12319527&version=12338590).

### 0.3.0-incubating (2016-10-31)
Official [source code download](https://www.apache.org/dyn/closer.cgi?filename=beam/0.3.0-incubating/apache-beam-0.3.0-incubating-source-release.zip&action=download).

[Release notes](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12319527&version=12338051).

### 0.2.0-incubating (2016-08-08)
Official [source code download](https://www.apache.org/dyn/closer.cgi?filename=beam/0.2.0-incubating/apache-beam-0.2.0-incubating-source-release.zip&action=download).

[Release notes](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12319527&version=12335766).

### 0.1.0-incubating (2016-06-15)
Official [source code download](https://www.apache.org/dyn/closer.cgi?filename=beam/0.1.0-incubating/apache-beam-0.1.0-incubating-source-release.zip&action=download).

The first incubating release of Apache Beam.
