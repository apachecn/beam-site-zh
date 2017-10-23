---
layout: default
title: "Cloud Dataflow Runner"
permalink: /documentation/runners/dataflow/
redirect_from: /learn/runners/dataflow/
---

# 使用谷歌云数据流执行器
<nav class="language-switcher">
  <strong>适用于:</strong>
  <ul>
    <li data-type="language-java" class="active">Java SDK</li>
    <li data-type="language-py">Python SDK</li>
  </ul>
</nav>

谷歌云数据流执行器使用[Cloud Dataflow managed service](https://cloud.google.com/dataflow/service/dataflow-service-desc).当你使用谷歌数据流服务执行管道程序时，执行器会上传你的可执行代码和相应依赖到谷歌云存储器并且创建一个云数据流作业，它会使用在谷歌云平台上管理的资源去执行你的管道任务。


该云数据流执行器和服务适用于大量的、持续性的任务并且提供：

* 一个完全的托管服务
* [自动伸缩](https://cloud.google.com/dataflow/service/dataflow-service-desc#autoscaling) 在作业执行期间使用的工作节点。
* [动态负载均衡](https://cloud.google.com/blog/big-data/2016/05/
在谷歌云数据流中动态负载均衡使得每个分片都参与工作）


[Beam 功能矩阵]({{ site.baseurl }}/documentation/runners/capability-matrix/) 记录了云数据流执行器支持的功能。

## 云数据流执行器前提条件和安装：
为了使用云数据流执行器，你必须完成以下步骤：

1. 选择或者创建一个谷歌云平台终端项目。

2. 开通项目付费

3. 使用必须的谷歌云APIs：云数据流，计算引擎，堆栈驱动日志记录，云存储，云json存储，云资源管理器。
你可能需要使用额外的APIs（比如大查询，云发布/订阅，或者云数据库）如果你要在管道流程序里面使用它。

4. 安装谷歌云SDK。

5. 创造一个云存储桶。
    * 在谷歌云平台终端，跳转到云存储浏览器。
    * 点击 **创建桶**
    * 在**创建桶**对话框中，选择以下属性：
      * _名字_：独一无二的桶名。不要在桶名中包含敏感信息，因为桶命名空间是全局的和公开可见的。
      * _存储类_: 多地域
      * _地址_: 选择你钟爱的地址。
    * 点击 **创建**。

相关更多信息，请查看章节 *在你开始前* ，该部分位于[云数据流快速开始](https://cloud.google.com/dataflow/docs/quickstarts)。

### 指定你的依赖


<span class="language-java">当使用java, 你必须在你的pom.xml中指定云数据流执行器的依赖。</span>
```java
<dependency>
  <groupId>org.apache.beam</groupId>
  <artifactId>beam-runners-google-cloud-dataflow-java</artifactId>
  <version>{{ site.release_latest }}</version>
  <scope>runtime</scope>
</dependency>
```

<span class="language-py">该部分不适用于python的 Beam SDK。</span>

### 授权
在执行你的管道流程序前，你一定要授权谷歌云平台。执行以下命令来获得[应用默认证书](https://developers.google.com/identity/protocols/application-default-credentials).


```
谷歌云授权应用默认登陆 
```

## 谷歌数据流执行器的管道流选项

<span class="language-java">当使用谷歌数据流执行器（java）执行你的管道流程序时，应该考虑这些通用的管道流选项。</span>
<span class="language-py">当使用谷歌数据流执行器（Python）执行你的管道流程序时，应该考虑这些通用的管道流选项。</span>

<table class="table table-bordered">
<tr>
  <th>字段</th>
  <th>描述</th>
  <th>默认值</th>
</tr>

<tr>
  <td><code>runner</code></td>
  <td>使用这个管道流执行器。该选项使你可以决定在执行的时候的管道流执行器。</td>
  <td>设置 <code>数据流</code> 或者 <code>数据流执行器</code> 运行在谷歌数据流服务上</td>
</tr>

<tr>
  <td><code>project</code></td>
  <td>你的谷歌云项目项目 ID 。</td>
  <td>如果没有设置，默认为当前环境下的默认项目。默认项目的设置使用<code>谷歌云</code>。</td>
</tr>

<!-- Only show for Java -->
<tr class="language-java">
  <td><code>streaming</code></td>
  <td>是否流模式设置为打开或者关闭；<code>true</code> 如果打开，请设置为<code>true</code> 如果执行管道流程序使用无界
   <code>集合</code>s。</td>
  <td><code>false</code></td>
</tr>

<tr>
  <td>
    <span class="language-java"><code>tempLocation</code></span>
    <span class="language-py"><code>temp_location</code></span>
  </td>
  <td>
    <span class="language-java">可选项。</span>
    <span class="language-py"> 必选项。</span>
    临时文件的存放路径。必须是一个有效的谷歌云存储URL，以 <code>gs://</code>开头。
    <span class="language-java">如果设置了, <code>临时目录</co奖杯用作<code>gcp临时目录</code>的默认值。</span>
  </td>
  <td>没有默认值。</td>
</tr>

<!-- Only show for Java -->
<tr class="language-java">
  <td><code>gcpTempLocation</code></td>
  <td>云存储桶的临时目录必须是一个有效的云存储URL，以<code>gs://</code>开头.</td>
  <td>如果没有设置，默认的<code>临时目录</code>会被提供。 <code>临时目录</code> 是一个有效的云存储URL.如果<code>临时目录</code> 不是一个有效的云存储URL, 你必须设置一个 <code>gcp临时目录</code>.</td>
</tr>

<tr>
  <td>
    <span class="language-java"><code>stagingLocation</code></span>
    <span class="language-py"><code>staging_location</code></span>
  </td>
  <td>可选项. 存储你的二进制和任何临时文件的云存储桶路径。 这个路径必须是一个有效的云存储URL，以<code>gs://</code>开头.</td>
  <td>
    <span class="language-java">如果没有设置, 默认设置遵照 <code>gcp临时目录</code>.</span>
    <span class="language-py">如果没有设置, 默认设置遵照<code>临时目录</code>.</span>
  </td>
</tr>

<!-- Only show for Python -->
<tr class="language-py">
  <td><code>save_main_session</code></td>
  <td>保存主要的 session 状态 ，因此被定义在<code>__main__</code> (e.g. 交互式的 session) 中的序列化函数和类可以被反序列化。一些工作流不需要session状态，例如，如果所有的函数或类都定义在合适的模块中  (除了 <code>__main__</code>) 并且这些模块在节点中被加载。</td>
  <td><code>false</code></td>
</tr>

<!-- Only show for Python -->
<tr class="language-py">
  <td><code>sdk_location</code></td>
  <td>使用Beam SDK的下载路径覆盖默认路径。这个值可以是一个URL，一个云存储路径或者一个SDK开发包的本地路径。 被提交的工作流将会从此路径下载或者拷贝开发包。 如果设置成  <code>default</code>, 将会使用标准的SDK路径. 如果为空, 没有SDK会被拷贝.</td>
  <td><code>default</code></td>
</tr>


</table>

查看相关的文档
<span class="language-java">[DataflowPipelineOptions]({{ site.baseurl }}/documentation/sdks/javadoc/{{ site.release_latest }}/index.html?org/apache/beam/runners/dataflow/options/DataflowPipelineOptions.html)</span>
<span class="language-py">[`PipelineOptions`]({{ site.baseurl }}/documentation/sdks/pydoc/{{ site.release_latest }}/apache_beam.options.html#apache_beam.options.pipeline_options.PipelineOptions)</span>
接口 (和任何子接口)，额外的管道配置选项.

## 附加的说明和警告

### 监控你的任务

当你的管道流程序执行时，你可以监控程序的进度，查看执行的细节，并且接收管道流程序结果的更新，通过 [数据流监视接口](https://cloud.google.com/dataflow/pipelines/dataflow-monitoring-intf) 或者 [数据流命令行接口](https://cloud.google.com/dataflow/pipelines/dataflow-command-line-intf).

### 阻塞执行

阻塞直至你的程序执行完成, 使用 <span class="language-java"><code>waitToFinish </code></span><span class="language-py"><code>wait_until_finish</code></span>在`PipelineResult` 从 `pipeline.run()`的返回处. 云数据流执行器打印任务状态的更新和终端消息当此处阻塞的时候。当结果终端连接到活动的任务时，注意在命令行中按下**Ctrl+C** 不会取消你的任务。为了取消任务，你可以使用[数据流监视接口](https://cloud.google.com/dataflow/pipelines/dataflow-monitoring-intf) 或者 [数据流命令行接口](https://cloud.google.com/dataflow/pipelines/dataflow-command-line-intf).

### 流式执行

<span class="language-java">如果你的管道程序使用一个源源不断的数据源，你必须设置 `streaming` 选项为 `true`.</span>
<span class="language-py">适用于Python 的Beam SDK 现在还不支持流式的管道程。</span>

