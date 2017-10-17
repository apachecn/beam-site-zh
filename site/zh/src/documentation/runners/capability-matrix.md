---
layout: default
title: "Apache Beam Capability Matrix"
permalink: /documentation/runners/capability-matrix/
redirect_from:
  - /learn/runners/capability-matrix/
  - /capability-matrix/
---

# Beam 功能 矩阵
Apache Beam提供了一个可移植的API层，用于构建复杂的，可以在多种执行引擎或<i>runners</i>之间执行的数据并行处理流水线。该层的核心概念基于Beam Model (以前称为 [Dataflow Model](http://www.vldb.org/pvldb/vol8/p1792-Akidau.pdf)), 并在不同程度上实现了每个Beam runner. 为了帮助说明每个runners的功能，我们创建了以下功能矩阵.

每个独立的功能都已按照相应的 <span class="wwwh-what-dark">What</span> / <span class="wwwh-where-dark">Where</span> / <span class="wwwh-when-dark">When</span> / <span class="wwwh-how-dark">How</span> 问题分组:

- <span class="wwwh-what-dark">What</span> 对数据的处理是哪种类型?
- <span class="wwwh-where-dark">Where</span> 数据在什么范围中计算?
- <span class="wwwh-when-dark">When</span> 何时将计算结果输出?
- <span class="wwwh-how-dark">How</span> 延迟数据如何处理?

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
