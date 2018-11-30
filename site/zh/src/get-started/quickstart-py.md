---
layout: default
title: "针对 Python 的 Beam 快速入门"
permalink: /get-started/quickstart-py/
---

# Apache Beam Python SDK 快速入门

该指南告诉你如何去安装你的 Python 开发环境, 获取 Python 的 Apache Beam SDK, 以及运行一个 Pipeline 例子.

* TOC
{:toc}

## 安装环境

### 检查 Python 版本

Beam SDK 的需要的 Python 版本是 2.7.x.
通过运行以下命令即可检查版本信息:

```
python --version
```

### 安装 pip

安装 [pip](https://pip.pypa.io/en/stable/installing/), Python 的软件包管理器.
通过运行以下命令来检查你已安装的版本是不是 7.0.0 或者更新的:

```
pip --version
```

### 安装 Python 虚拟环境

强烈推荐你安装一个 [Python 虚拟环境](http://docs.python-guide.org/en/latest/dev/virtualenvs/) 以初始化实验.
如果你 `virtualenv` 的版本不是 13.1.0 或者更新的话，通过运行以下命令来安装它: 

```
pip install --upgrade virtualenv
```

如果你不想去使用 Python 虚拟环境（不推荐这样做）, 确保在你的机器上已安装了 `setuptools`.
如果你的 `setuptools` 版本不是 17.1 或者更新的话，通过运行以下命令来安装它:

```
pip install --upgrade setuptools
```

## 获取 Apache Beam

### 创建并且激活虚拟环境

一个虚拟环境是一个包含它自己 Python 软件的目录树.
要创建一个虚拟环境，创建一个目录并运行:

```
virtualenv /path/to/directory
```

需要为每个使用它的 shell 激活一个虚拟环境.
激活它后设置一些指定虚拟环境目录的环境变量.

要激活 Bash 中的虚拟环境，可以运行:

```
. /path/to/directory/bin/activate
```

也就是说，在你创建的虚拟环境目录下输入脚本 `bin/activate`.

针对使用其它 shell 来安装的教程，请参阅 [virtualenv documentation](https://virtualenv.pypa.io/en/stable/userguide/#activate-script).

### 下载和安装

从 PyPI 中安装最新的 Python SDK:

```
pip install apache-beam
```

#### 其它要求

上述安装将不会安装所有使用 Google Cloud Dataflow runner 等功能的额外依赖关系.
有关不同功能需要什么额外的包的信息将在下面突出显示.
可以使用一些想 `pip install apache-beam[feature1, feature2]` 这样的操作来安装多个额外所需的依赖.

- **Google Cloud Platform**
  - 安装命令: `pip install apache-beam[gcp]`
  - 需求:
    - Google Cloud Dataflow Runner
    - GCS IO
    - Datastore IO
    - BigQuery IO
- **Tests**
  - 安装命令: `pip install apache-beam[test]`
  - Required for developing on beam and running unittests
- **Docs**
  - 安装命令: `pip install apache-beam[docs]`
  - Generating API documentation using Sphinx

## 执行一个本地的 pipeline（管道）

Apache Beam [examples](https://github.com/apache/beam/tree/master/sdks/python/apache_beam/examples) 目录有很多例子.
所有的例子都可以通过在示例脚本中传递所需的参数来以 locally（本地的）方式运行.

例如, 要运行 `wordcount.py` 示例, 请运行:

{:.runner-direct}
```
python -m apache_beam.examples.wordcount --input <PATH_TO_INPUT_FILE> --output counts
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

## 下一步

* 学习更多关于 [针对 Python 的 Beam SDK ]({{ site.baseurl }}/documentation/sdks/python/)
  以及浏览 [Python SDK API 参考文档]({{ site.baseurl }}/documentation/sdks/pydoc).
* 在 [WordCount 例子介绍]({{ site.baseurl }}/get-started/wordcount-example) 中查看这些 WordCount 示例.
* 深入到我们最喜欢的 [文章和演讲]({{ site.baseurl }}/documentation/resources) 部分.
* 加入 Beam [users@]({{ site.baseurl }}/get-started/support#mailing-lists) 邮件列表.

如果遇到任何问题，请随时 [联系]({{ site.baseurl }}/get-started/support)！

