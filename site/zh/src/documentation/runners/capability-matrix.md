---
layout: default
title: "Apache Beam Capability Matrix"
permalink: /documentation/runners/capability-matrix/
redirect_from:
  - /learn/runners/capability-matrix/
  - /capability-matrix/
---

# Beam Capability Matrix
Apache Beam提供了一个可移植的API层，用于构建复杂的数据并行处理流水线，可以在多种执行引擎或<i>runners</i>之间执行。该层的核心概念基于Beam Model (以前称为 [Dataflow Model](http://www.vldb.org/pvldb/vol8/p1792-Akidau.pdf)), 并在不同程度上实现了每个Beam runner. 为了帮助说明每个runners的capabilities，我们创建了以下capability matrix.

个别的capabilities已经按照相应的 <span class="wwwh-what-dark">What</span> / <span class="wwwh-where-dark">Where</span> / <span class="wwwh-when-dark">When</span> / <span class="wwwh-how-dark">How</span> 问题分组:

- <span class="wwwh-what-dark">What</span> results are being calculated?
- <span class="wwwh-where-dark">Where</span> in event time?
- <span class="wwwh-when-dark">When</span> in processing time?
- <span class="wwwh-how-dark">How</span> do refinements of results relate?

关于 <span class="wwwh-what-dark">What</span> / <span class="wwwh-where-dark">Where</span> / <span class="wwwh-when-dark">When</span> / <span class="wwwh-how-dark">How</span> 概念的更多细节, 我们建议阅读 O'Reilly Radar上的<a href="http://oreilly.com/ideas/the-world-beyond-batch-streaming-102">Streaming 102</a> 帖子.

请注意，将来，我们打算在当前集合之外添加其他表，例如运行时特性（例如至少一次vs一次），性能等.

{% include capability-matrix-common.md %}
{% assign cap-data=site.data.capability-matrix %}

<center>

<!-- Summary table -->
{% assign cap-style='cap-summary' %}
{% assign cap-view='summary' %}
{% assign cap-other-view='full' %}
{% assign cap-toggle-details=1 %}
{% assign cap-display='block' %}

{% include capability-matrix.md %}

<!-- Full details table -->
{% assign cap-style='cap' %}
{% assign cap-view='full' %}
{% assign cap-other-view='summary' %}
{% assign cap-toggle-details=0 %}
{% assign cap-display='none' %}

{% include capability-matrix.md %}
</center>
