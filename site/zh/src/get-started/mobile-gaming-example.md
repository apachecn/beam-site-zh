---
layout: default
title: "Beam Mobile Gaming Example"
permalink: /get-started/mobile-gaming-example/
redirect_from: /use/mobile-gaming-example/
---

# Apache Beam Mobile Gaming Pipeline Examples (Apache Beam 手游管道示例)

* TOC
{:toc}

<nav class="language-switcher">
  <strong>适用于:</strong>
  <ul>
    <li data-type="language-java">Java SDK</li>
    <li data-type="language-py">Python SDK</li>
  </ul>
</nav>

本节提供了一系列示例Apache Beam管道的演示，演示了比基本的 [WordCount]({{ site.baseurl }}/get-started/wordcount-example) 示例更复杂的功能。 本节中的管道用于处理用户在手机上玩的假想游戏的数据。管道将会在越来越复杂的情况下进行演示；这条管道显示了处理复杂性增加的过程;例如，第一个管道显示了如何运行一个批分析作业来获得相对简单的分数数据，而后面的管道则使用Beam的窗口和触发器特性来提供低延迟的数据分析和更复杂的关于用户的游戏模式的信息。

{:.language-java}
> **Note**: 这些示例假定您对Beam编程模型很熟悉。如果您还没有，我们建议您熟悉编程模型文档，并在继续之前运行一个基本的示例管道。还要注意的是，这些示例使用Java 8 lambda语法，因此需要Java 8。但是，您可以使用Java 7创建具有相同功能的管道。

{:.language-py}
> **Note**: 这些示例假定您对Beam编程模型很熟悉。如果您还没有，我们建议您熟悉编程模型文档，并在继续之前运行一个基本的示例管道。

每当用户玩我们假设的移动游戏实例时，他们就会生成一个数据事件。
每个数据事件由以下信息组成:

- 用户玩游戏的唯一ID。
- 每个用户所属的游戏团队的团队ID
- 特定场合的得分值
- 一个时间戳记录事件的特定实例发生的时间，这是每个游戏数据事件的事件时间。

当用户完成游戏的一个实例时，他们的手机将数据事件发送到一个游戏服务器，在这个服务器中，数据被记录并存储在一个文件中。
一般情况下，数据会在完成后立即发送到游戏服务器。
然而，有时网络上的延迟会在不同的时间点发生。
另一种可能的情况是，当用户的手机与服务器失去联系时(比如在飞机上或外部网络覆盖区域)，玩“离线”游戏的用户。
当用户的手机回到与游戏服务器的联系时，手机会发送所有累积的游戏数据。
在这些情况下，一些数据事件可能会无序。

下图显示了理想的情况(事件是在发生时处理的)和现实(在处理之前通常会有一个时间延迟)。

<figure id="fig1">
    <img src="{{ site.baseurl }}/images/gaming-example-basic.png"
         width="264" height="260"
         alt="Score data for three users.">
</figure>
**图 1:** x轴表示事件时间:游戏事件发生的实际时间。y轴表示处理时间:处理游戏事件的时间。理想情况下，事件应该按照图中虚线所描述的那样进行处理。然而，实际情况并非如此，它看起来更像是由红色曲线所描绘的。

与用户生成的数据事件相比，游戏服务器可能会收到更晚的数据事件。这个时间差 (称为 **偏差**) 可以对管道产生处理意义，这些管道可以计算每次生成的结果时考虑的计算。例如，这样的管道可以跟踪每天每小时生成的分数，或者计算用户持续地玩游戏的时间长度，这两种方法都依赖于每个数据记录的事件时间。

因为我们的一些示例管道使用数据文件(比如来自游戏服务器的日志)作为输入，每个游戏的事件时间戳可能被嵌入到数据中——也就是说，它是每个数据记录中的一个字段。
这些管道需要在从输入文件读取后，从每个数据记录中解析事件时间戳。

对于从无界源读取无界游戏数据的管道，数据源将每个PCollection元素的固有 [timestamp]({{ site.baseurl }}/documentation/programming-guide/#element-timestamps) 设置为适当的事件时间。

移动游戏示例管道的复杂性有所不同，从简单的批处理分析到可以执行实时分析和滥用检测的更复杂的管道。 本节将引导您完成每个示例，并演示如何使用Beam功能（如窗口和触发器）来扩展管道的功能。

## UserScore: 批处理的基本分数处理

`UserScore` 管道是处理移动游戏数据的最简单的例子。 `UserScore` 决定了每个用户在有限数据集上的总得分(例如，在游戏服务器上存储了一天的分数)。在收集了所有相关数据之后，像 `UserScore` 这样的管道最好定期运行。例如，`UserScore` 可以作为夜间工作，而不是在那天收集的数据。

{:.language-java}
> **Note:** 请参阅 [GitHub上的UserScore](https://github.com/apache/beam/blob/master/examples/java8/src/main/java/org/apache/beam/examples/complete/game/UserScore.java) ，以获得完整的示例管道程序。

{:.language-py}
> **Note:** 请参阅 [GitHub上的UserScore](https://github.com/apache/beam/blob/master/sdks/python/apache_beam/examples/complete/game/user_score.py) ，以获得完整的示例管道程序。

### UserScore 做什么工作??

在一天的得分数据中，每个用户ID可能都有多个记录(如果用户在分析窗口中玩多个游戏实例)，每个用户都有自己的得分值和时间戳。如果我们想要确定用户在一天中所扮演的所有实例的总得分，我们的管道将需要将每个用户的所有记录合并在一起。

当管道处理每个事件时，事件得分被添加到该特定用户的总和中。

`UserScore` 只解析每个记录需要的数据，特别是用户ID和得分值。管道不会考虑任何记录的事件时间;它只处理在运行管道时指定的输入文件中的所有数据。

> **Note:** 为了有效地使用 `UserScore` 管道，您需要确保您提供的输入数据已经按照所需的事件时间周期分组——也就是说，您指定的输入文件只包含您所关心的那一天的数据。

`UserScore`的基本流程流如下:

1. 从文本文件中读取当天的得分数据。
2. 通过用户ID对每个游戏事件进行分组，并结合得分值来获得该特定用户的总得分，从而为每个独立用户的得分值进行累加。
3. 将结果数据写入一个文本文件。

下图显示了管道分析期间多个用户的分数数据。 在图中，每个数据点是一个用户/得分对的事件的结果：

<figure id="fig2">
    <img src="{{ site.baseurl }}/images/gaming-example.gif"
         width="900" height="263"
         alt="Score data for three users.">
</figure>
**图 2:** 三个用户的得分数据。

这个例子使用批处理，而图的Y轴表示处理时间:管道处理事件在Y轴上的时间越低，后面的事件就越高。图的X轴表示每个游戏事件的事件时间，由该事件的时间戳表示。请注意，图中的个别事件并不是按照它们发生的顺序处理的(根据它们的时间戳)。

在从输入文件读取分数事件之后，管道将所有这些用户/分数对组合在一起，并将得分值计算为每个惟一用户的总值。 `UserScore` 封装了该步骤的核心逻辑，即[用户定义的复合转换]({{ site.baseurl }}/documentation/programming-guide/#composite-transforms) `ExtractAndSumScore`:

```java
{% github_sample /apache/beam/blob/master/examples/java8/src/main/java/org/apache/beam/examples/complete/game/UserScore.java tag:DocInclude_USExtractXform
%}```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/complete/game/user_score.py tag:extract_and_sum_score
%}```

`ExtractAndSumScore` 写得更为概括，因为你可以通过自己想要数据的字段进行分组(在我们的游戏中，由唯一的用户或唯一的团队)。 这意味着我们可以在其他管道中重新使用`ExtractAndSumScore`，例如，通过团队对比分数据进行分组。

下面是UserScore的主要方法，展示了我们如何应用这三个步骤:

```java
{% github_sample /apache/beam/blob/master/examples/java8/src/main/java/org/apache/beam/examples/complete/game/UserScore.java tag:DocInclude_USMain
%}```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/complete/game/user_score.py tag:main
%}```

### Limitations(局限性)

正如在示例中所写的， `UserScore` 管道有一些限制:

* 因为一些分数数据可能是由离线玩家生成的，并且在每日中断之后发送，对于游戏数据，由`UserScore` 管道生成的结果数据可能是 **不完整的**. 当管道运行时，`UserScore` 只处理输入文件(多个)中存在的固定输入集。

* `UserScore` 在处理时间处理输入文件中存在的所有数据事件，并且 **不会根据事件时间检查或以其他方式错误检查事件**。 因此，结果可能包括某些值，其事件时间不在相关分析期间，例如前一天的延迟记录。

* 因为 `UserScore` 仅在收集所有数据之后运行，所以在用户生成数据事件（事件时间）和计算结果（处理时间）之间时，它具有 **很高的延迟**。 

* `UserScore` 还仅报告整天的总体结果，并且不提供有关如何在白天累积的数据的更细粒度的信息。

从下一个流程示例开始，我们将讨论如何使用Beam的功能来解决这些限制。

## HourlyTeamScore: 使用窗口进行的高级处理——批处理

 `HourlyTeamScore` 管道扩展了 `UserScore` 管道中使用的基本批次分析原则，并改进了其一些限制。 `HourlyTeamScore` 通过在 Beam SDKs中使用其他功能，并考虑到游戏数据的更多方面，进行细化分析。 例如， `HourlyTeamScore` 可以过滤掉不属于相关分析期的数据。

像 `UserScore`一样，`HourlyTeamScore` 最好被认为是在收集所有相关数据后定期运行（例如每天一次）的工作。 管道从文件读取固定数据集，并将结果写入Google Cloud BigQuery表。

{:.language-java}
> **Note:** 完整的示例管道程序请查看 [HourlyTeamScore on GitHub](https://github.com/apache/beam/blob/master/examples/java8/src/main/java/org/apache/beam/examples/complete/game/HourlyTeamScore.java)

{:.language-py}
> **Note:** 完整的示例管道程序请查看 [HourlyTeamScore on GitHub](https://github.com/apache/beam/blob/master/sdks/python/apache_beam/examples/complete/game/hourly_team_score.py) 
### HourlyTeamScore做什么？

`HourlyTeamScore` 计算固定数据集中每个团队每小时的总分数（如一天的数据）。

*  `HourlyTeamScore` 不是一次对整个数据集进行操作，而是将输入数据分成逻辑窗口，并在这些窗口上执行计算。 这可以让 `HourlyUserScore` 提供每个窗口评分数据的信息，其中每个窗口以固定的时间间隔（如每小时一次）表示游戏得分进度。

* `HourlyTeamScore` 根据其事件时间（嵌入的时间戳）是否在相关分析期内进行数据事件过滤。 基本上，管道检查每个游戏事件的时间戳，并确保它落在我们想要分析的范围内（在这种情况下是所讨论的日子）。 前几天的数据事件被丢弃，不包括在总分中。 这使得`HourlyTeamScore` 更加强大，并且比 `UserScore`. 更不容易出现错误的结果数据。 在分析期间它还允许管道去计算带着时间戳的迟到数据。

下面我们将详细介绍 `HourlyTeamScore` 中的这些增强功能：

#### 固定时间窗口

使用固定时间窗口使管道能够提供关于在分析期间的数据集内累积的事件的更好的信息。在我们的例子中，它告诉我们，每个团队在一天中是活跃的，团队在那个时候的得分是多少。

下图显示了在应用固定时间窗口后，管道如何处理一天的单个团队的得分数据:

<figure id="fig3">
    <img src="{{ site.baseurl }}/images/gaming-example-team-scores-narrow.gif"
         width="900" height="390"
         alt="Score data for two teams.">
</figure>
**图 3:** 为两队得分数据。基于事件发生时的得分每个团队的分数被划分为逻辑窗口。

注意，随着处理时间的推移，现在的总和是每个窗口的总和；每个窗口表示分数发生的当天的一小时的事件时间。

> **Note:** 如上图所示，使用窗口可以为每个间隔（在这种情况下每个小时）产生一个独立的总和。`HourlyTeamScore` 如上图所示，使用窗口可以为每个间隔（在这种情况下每个小时）产生一个独立的总和。

Beam的窗口特性使用了与`PCollection`的每个元素相关联的 [固有时间戳信息]({{ site.baseurl }}/documentation/programming-guide/#element-timestamps) 。 因为我们希望我们的管道基于事件时间窗口，所以我们必须**首先提取嵌入在每个数据记录中的时间戳**，将其应用到得分数据的 `PCollection` 中相应的元素。然后，管道可以 **应用窗口功能** 将 `PCollection` 划分为逻辑窗口。

{:.language-java}
`HourlyTeamScore` 使用 [WithTimestamps](https://github.com/apache/beam/blob/master/sdks/java/core/src/main/java/org/apache/beam/sdk/transforms/WithTimestamps.java) 和 [Window](https://github.com/apache/beam/blob/master/sdks/java/core/src/main/java/org/apache/beam/sdk/transforms/windowing/Window.java) 转换去展示这些操作。

{:.language-py}
`HourlyTeamScore` 使用 `FixedWindows` 转换（可以在 [window.py](https://github.com/apache/beam/blob/master/sdks/python/apache_beam/transforms/window.py)中找到）去展示这些操作。

以下代码显示：

```java
{% github_sample /apache/beam/blob/master/examples/java8/src/main/java/org/apache/beam/examples/complete/game/HourlyTeamScore.java tag:DocInclude_HTSAddTsAndWindow
%}```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/complete/game/hourly_team_score.py tag:add_timestamp_and_window
%}```

请注意，管道用于指定窗口的变换与实际数据处理转换（如 `ExtractAndSumScores` ）不同。 此功能为您设计Beam管道提供了一些灵活性，您可以在具有不同窗口特征的数据集上运行现有转换。

#### 基于事件时间过滤

`HourlyTeamScore`使用 **过滤** 功能从我们的数据集中删除任何事件，这些事件的时间戳不会在相关的分析期间发生(也就是说，它们不是在我们感兴趣的那一天生成的)。这使得管道不被错误地包括任何数据，例如，在前一天离线生成的数据，但是在当天被发送到游戏服务器。

它还允许管道包含有有效时间戳的相关 **延迟数据** 事件，但是这些数据在我们的分析期结束后到达的。例如，如果我们的管道截止时间是12:00 am，我们可能在 2:00 am点运行管道，但是过滤出时间戳表示在12:00之后发生的事件。数据事件在12:01 am和2:00 am之间延迟到达，但其时间戳表示它们在12:00 am截止之前发生，将被包括在流水线处理中。	

`HourlyTeamScore` 使用 `Filter` 转换来执行此操作。当您使用 `Filter` 时，您将指定一个字段，该字段将对每个数据记录进行比较。通过比较的数据记录被包括进来，而不能通过比较的事件被排除在外。在我们的例子中，字段是我们指定的截止时间，我们只比较数据时间戳字段的一个部分。

以下代码显示了 `HourlyTeamScore` 如何使用 `过滤器` 转换过滤在相关分析期之前或之后发生的事件：

```java
{% github_sample /apache/beam/blob/master/examples/java8/src/main/java/org/apache/beam/examples/complete/game/HourlyTeamScore.java tag:DocInclude_HTSFilters
%}```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/complete/game/hourly_team_score.py tag:filter_by_time_range
%}```

#### 每个窗口计算每队的得分

`HourlyTeamScore` 使用与 `UserScore` 管道相同的 `ExtractAndSumScores` 转换，但传递不同的key（团队，而不是用户）。此外，由于管道在应用固定时间1小时窗口的输入数据后使用`ExtractAndSumScores`，因此数据被团队和窗口分组。您可以在`HourlyTeamScore` 的主要方法中看到完整的变换序列：

```java
{% github_sample /apache/beam/blob/master/examples/java8/src/main/java/org/apache/beam/examples/complete/game/HourlyTeamScore.java tag:DocInclude_HTSMain
%}```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/complete/game/hourly_team_score.py tag:main
%}```

### 限制

正如上面所写的，`HourlyTeamScore` 仍然有一个限制：

* `HourlyTeamScore` 在数据事件发生（事件时间）和结果生成时间（处理时间）之间仍然具有 **高延迟** ，因为作为批处理流程，需要等待开始处理，直到出现所有数据事件。

## LeaderBoard: Streaming Processing with Real-Time Game Data

One way we can help address the latency issue present in the `UserScore` and `HourlyTeamScore` pipelines is by reading the score data from an unbounded source. The `LeaderBoard` pipeline introduces streaming processing by reading the game score data from an unbounded source that produces an infinite amount of data, rather than from a file on the game server.

The `LeaderBoard` pipeline also demonstrates how to process game score data with respect to both _processing time_ and _event time_. `LeaderBoard` outputs data about both individual user scores and about team scores, each with respect to a different time frame.

Because the `LeaderBoard` pipeline reads the game data from an unbounded source as that data is generated, you can think of the pipeline as an ongoing job running concurrently with the game process. `LeaderBoard` can thus provide low-latency insights into how users are playing the game at any given moment — useful if, for example, we want to provide a live web-based scoreboard so that users can track their progress against other users as they play.

{:.language-java}
> **Note:** See [LeaderBoard on GitHub](https://github.com/apache/beam/blob/master/examples/java8/src/main/java/org/apache/beam/examples/complete/game/LeaderBoard.java) for the complete example pipeline program.

{:.language-py}
> **Note:** See [LeaderBoard on GitHub](https://github.com/apache/beam/blob/master/sdks/python/apache_beam/examples/complete/game/leader_board.py) for the complete example pipeline program.

### What Does LeaderBoard Do?

The `LeaderBoard` pipeline reads game data published to an unbounded source that produces an infinite amount of data in near real-time, and uses that data to perform two separate processing tasks:

* `LeaderBoard` calculates the total score for every unique user and publishes speculative results for every ten minutes of _processing time_. That is, ten minutes after data is received, the pipeline outputs the total score per user that the pipeline has processed to date. This calculation provides a running "leader board" in close to real time, regardless of when the actual game events were generated.

* `LeaderBoard` calculates the team scores for each hour that the pipeline runs. This is useful if we want to, for example, reward the top-scoring team for each hour of play. The team score calculation uses fixed-time windowing to divide the input data into hour-long finite windows based on the _event time_ (indicated by the timestamp) as data arrives in the pipeline.

    In addition, the team score calculation uses Beam's trigger mechanisms to provide speculative results for each hour (which update every five minutes until the hour is up), and to also capture any late data and add it to the specific hour-long window to which it belongs.

Below, we'll look at both of these tasks in detail.

#### Calculating User Score based on Processing Time

We want our pipeline to output a running total score for each user for every ten minutes of processing time. This calculation doesn't consider _when_ the actual score was generated by the user's play instance; it simply outputs the sum of all the scores for that user that have arrived in the pipeline to date. Late data gets included in the calculation whenever it happens to arrive in the pipeline as it's running.

Because we want all the data that has arrived in the pipeline every time we update our calculation, we have the pipeline consider all of the user score data in a **single global window**. The single global window is unbounded, but we can specify a kind of temporary cut-off point for each ten-minute calculation by using a processing time [trigger]({{ site.baseurl }}/documentation/programming-guide/#triggers).

When we specify a ten-minute processing time trigger for the single global window, the pipeline effectively takes a "snapshot" of the contents of the window every time the trigger fires. This snapshot happens after ten minutes have passed since data was received. If no data has arrived, the pipeline takes its next "snapshot" 10 minutes after an element arrives. Since we're using a single global window, each snapshot contains all the data collected _to that point in time_. The following diagram shows the effects of using a processing time trigger on the single global window:

<figure id="fig4">
    <img src="{{ site.baseurl }}/images/gaming-example-proc-time-narrow.gif"
         width="900" height="263"
         alt="Score data for three users.">
</figure>
**Figure 4:** Score data for three users. Each user's scores are grouped together in a single global window, with a trigger that generates a snapshot for output ten minutes after data is received.

As processing time advances and more scores are processed, the trigger outputs the updated sum for each user.

The following code example shows how `LeaderBoard` sets the processing time trigger to output the data for user scores:

```java
{% github_sample /apache/beam/blob/master/examples/java8/src/main/java/org/apache/beam/examples/complete/game/LeaderBoard.java tag:DocInclude_ProcTimeTrigger
%}```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/complete/game/leader_board.py tag:processing_time_trigger
%}```

`LeaderBoard` sets the [window accumulation mode]({{ site.baseurl }}/documentation/programming-guide/#window-accumulation-modes) to accumulate window panes as the trigger fires. This accumulation mode is set by <span class="language-java">invoking `.accumulatingFiredPanes`</span> <span class="language-py">using `accumulation_mode=trigger.AccumulationMode.ACCUMULATING`</span> when setting the trigger, and causes the pipeline to accumulate the previously emitted data together with any new data that's arrived since the last trigger fire. This ensures that `LeaderBoard` is a running sum for the user scores, rather than a collection of individual sums.

#### Calculating Team Score based on Event Time

We want our pipeline to also output the total score for each team during each hour of play. Unlike the user score calculation, for team scores, we care about when in _event_ time each score actually occurred, because we want to consider each hour of play individually. We also want to provide speculative updates as each individual hour progresses, and to allow any instances of late data — data that arrives after a given hour's data is considered complete — to be included in our calculation.

Because we consider each hour individually, we can apply fixed-time windowing to our input data, just like in `HourlyTeamScore`. To provide the speculative updates and updates on late data, we'll specify additional trigger parameters. The trigger will cause each window to calculate and emit results at an interval we specify (in this case, every five minutes), and also to keep triggering after the window is considered "complete" to account for late data. Just like the user score calculation, we'll set the trigger to accumulating mode to ensure that we get a running sum for each hour-long window.

The triggers for speculative updates and late data help with the problem of [time skew]({{ site.baseurl }}/documentation/programming-guide/#windowing). Events in the pipeline aren't necessarily processed in the order in which they actually occurred according to their timestamps; they may arrive in the pipeline out of order, or late (in our case, because they were generated while the user's phone was out of contact with a network). Beam needs a way to determine when it can reasonably assume that it has "all" of the data in a given window: this is called the _watermark_.

In an ideal world, all data would be processed immediately when it occurs, so the processing time would be equal to (or at least have a linear relationship to) the event time. However, because distributed systems contain some inherent inaccuracy (like our late-reporting phones), Beam often uses a heuristic watermark.

The following diagram shows the relationship between ongoing processing time and each score's event time for two teams:

<figure id="fig5">
    <img src="{{ site.baseurl }}/images/gaming-example-event-time-narrow.gif"
         width="900" height="390"
         alt="Score data by team, windowed by event time.">
</figure>
**Figure 5:** Score data by team, windowed by event time. A trigger based on processing time causes the window to emit speculative early results and include late results.

The dotted line in the diagram is the "ideal" **watermark**: Beam's notion of when all data in a given window can reasonably be considered to have arrived. The irregular solid line represents the actual watermark, as determined by the data source.

Data arriving above the solid watermark line is _late data_ — this is a score event that was delayed (perhaps generated offline) and arrived after the window to which it belongs had closed. Our pipeline's late-firing trigger ensures that this late data is still included in the sum.

The following code example shows how `LeaderBoard` applies fixed-time windowing with the appropriate triggers to have our pipeline perform the calculations we want:

```java
{% github_sample /apache/beam/blob/master/examples/java8/src/main/java/org/apache/beam/examples/complete/game/LeaderBoard.java tag:DocInclude_WindowAndTrigger
%}```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/complete/game/leader_board.py tag:window_and_trigger
%}```

Taken together, these processing strategies let us address the latency and completeness issues present in the `UserScore` and `HourlyTeamScore` pipelines, while still using the same basic transforms to process the data—as a matter of fact, both calculations still use the same `ExtractAndSumScore` transform that we used in both the `UserScore` and `HourlyTeamScore` pipelines.

## GameStats: Abuse Detection and Usage Analysis

While `LeaderBoard` demonstrates how to use basic windowing and triggers to perform low-latency and flexible data analysis, we can use more advanced windowing techniques to perform more comprehensive analysis. This might include some calculations designed to detect system abuse (like spam) or to gain insight into user behavior. The `GameStats` pipeline builds on the low-latency functionality in `LeaderBoard` to demonstrate how you can use Beam to perform this kind of advanced analysis.

Like `LeaderBoard`, `GameStats` reads data from an unbounded source. It is best thought of as an ongoing job that provides insight into the game as users play.

{:.language-java}
> **Note:** See [GameStats on GitHub](https://github.com/apache/beam/blob/master/examples/java8/src/main/java/org/apache/beam/examples/complete/game/GameStats.java) for the complete example pipeline program.

{:.language-py}
> **Note:** See [GameStats on GitHub](https://github.com/apache/beam/blob/master/sdks/python/apache_beam/examples/complete/game/game_stats.py) for the complete example pipeline program.

### What Does GameStats Do?

Like `LeaderBoard`, `GameStats` calculates the total score per team, per hour. However, the pipeline also performs two kinds of more complex analysis:

* `GameStats` does **abuse detection** system that performs some simple statistical analysis on the score data to determine which users, if any, might be spammers or bots. It then uses the list of suspected spam/bot users to filter the bots out of the hourly team score calculation.
* `GameStats` **analyzes usage patterns** by grouping together game data that share similar event times using session windowing. This lets us gain some intelligence on how long users tend to play, and how game length changes over time.

Below, we'll look at these features in more detail.

#### Abuse Detection

Let's suppose scoring in our game depends on the speed at which a user can "click" on their phone. `GameStats`'s abuse detection analyzes each user's score data to detect if a user has an abnormally high "click rate" and thus an abnormally high score. This might indicate that the game is being played by a bot that operates significantly faster than a human could play.

To determine whether or not a score is "abnormally" high, `GameStats` calculates the average of every score in that fixed-time window, and then checks each score individual score against the average score multiplied by an arbitrary weight factor (in our case, 2.5). Thus, any score more than 2.5 times the average is deemed to be the product of spam. The `GameStats` pipeline tracks a list of "spam" users and filters those users out of the team score calculations for the team leader board.

Since the average depends on the pipeline data, we need to calculate it, and then use that calculated data in a subsequent `ParDo` transform that filters scores that exceed the weighted value. To do this, we can pass the calculated average to as a [side input]({{ site.baseurl }}/documentation/programming-guide/#side-inputs) to the filtering `ParDo`.

The following code example shows the composite transform that handles abuse detection. The transform uses the `Sum.integersPerKey` transform to sum all scores per user, and then the `Mean.globally` transform to determine the average score for all users. Once that's been calculated (as a `PCollectionView` singleton), we can pass it to the filtering `ParDo` using `.withSideInputs`:

```java
{% github_sample /apache/beam/blob/master/examples/java8/src/main/java/org/apache/beam/examples/complete/game/GameStats.java tag:DocInclude_AbuseDetect
%}```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/complete/game/game_stats.py tag:abuse_detect
%}```

The abuse-detection transform generates a view of users supected to be spambots. Later in the pipeline, we use that view to filter out any such users when we calculate the team score per hour, again by using the side input mechanism. The following code example shows where we insert the spam filter, between windowing the scores into fixed windows and extracting the team scores:

```java
{% github_sample /apache/beam/blob/master/examples/java8/src/main/java/org/apache/beam/examples/complete/game/GameStats.java tag:DocInclude_FilterAndCalc
%}```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/complete/game/game_stats.py tag:filter_and_calc
%}```

#### Analyzing Usage Patterns

We can gain some insight on when users are playing our game, and for how long, by examining the event times for each game score and grouping scores with similar event times into _sessions_. `GameStats` uses Beam's built-in [session windowing](https://github.com/apache/beam/blob/master/sdks/java/core/src/main/java/org/apache/beam/sdk/transforms/windowing/Sessions.java) function to group user scores into sessions based on the time they occurred.

When you set session windowing, you specify a _minimum gap duration_ between events. All events whose arrival times are closer together than the minimum gap duration are grouped into the same window. Events where the difference in arrival time is greater than the gap are grouped into separate windows. Depending on how we set our minimum gap duration, we can safely assume that scores in the same session window are part of the same (relatively) uninterrupted stretch of play. Scores in a different window indicate that the user stopped playing the game for at least the minimum gap time before returning to it later.

The following diagram shows how data might look when grouped into session windows. Unlike fixed windows, session windows are _different for each user_ and is dependent on each individual user's play pattern:

<figure id="fig6">
    <img src="{{ site.baseurl }}/images/gaming-example-session-windows.png"
         width="662" height="521"
         alt="A diagram representing session windowing."
         alt="User sessions, with a minimum gap duration.">
</figure>
**Figure 6:** User sessions, with a minimum gap duration. Note how each user has different sessions, according to how many instances they play and how long their breaks between instances are.

We can use the session-windowed data to determine the average length of uninterrupted play time for all of our users, as well as the total score they achieve during each session. We can do this in the code by first applying session windows, summing the score per user and session, and then using a transform to calculate the length of each individual session:

```java
{% github_sample /apache/beam/blob/master/examples/java8/src/main/java/org/apache/beam/examples/complete/game/GameStats.java tag:DocInclude_SessionCalc
%}```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/complete/game/game_stats.py tag:session_calc
%}```

This gives us a set of user sessions, each with an attached duration. We can then calculate the _average_ session length by re-windowing the data into fixed time windows, and then calculating the average for all sessions that end in each hour:

```java
{% github_sample /apache/beam/blob/master/examples/java8/src/main/java/org/apache/beam/examples/complete/game/GameStats.java tag:DocInclude_Rewindow
%}```
```py
{% github_sample /apache/beam/blob/master/sdks/python/apache_beam/examples/complete/game/game_stats.py tag:rewindow
%}```

We can use the resulting information to find, for example, what times of day our users are playing the longest, or which stretches of the day are more likely to see shorter play sessions.

