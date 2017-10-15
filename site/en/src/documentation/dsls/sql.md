---
layout: default
title: "DSLs: SQL"
permalink: /documentation/dsls/sql/
---

* [1. 概述](#overview)
* [2. DSL APIs的用法](#usage)
* [3. Beam SQL的功能](#functionality)
  * [3.1. 支持的功能](#features)
  * [3.2. 数据类型](#data-type)
  * [3.3. 内置的SQL函数](#built-in-functions)
* [4. Beam SQL的内部](#internal-of-sql)

该页面描述了Beam SQL的实现，以及如何使用DSL API简化Beam pipeline.

> 注意，Beam SQL尚未合并到主分支（正在使用分支 [DSL_SQL](https://github.com/apache/beam/tree/DSL_SQL)), 但即将推出.

# <a name="overview"></a>1. 概述
SQL是一种采用简明语法处理数据的通用标准，
使用DSL API（目前仅在Java中可用），现在可以使用标准SQL语句
像查询常规表一样查询PCollections . DSL API利用 [Apache Calcite](http://calcite.apache.org/) 来解析和优化SQL查询，然后转换为复合Beam PTransform。通过这种方式，SQL和普通Beam PTransform可以在同一个pipeline中混合使用.

SQL DSL API有两个主要部分:

* [BeamRecord]({{ site.baseurl }}/documentation/sdks/javadoc/{{ site.release_latest }}/index.html?org/apache/beam/sdk/values/BeamRecord.html): 用于定义由多个原始数据类型的多个名为列组成的复合记录（即行）的新数据类型。所有SQL DSL查询必须针对PCollection <BeamRecord>类型的集合进行。请注意，BeamRecord本身不是特定于SQL的，也可能在不使用SQL的pipelines 中使用.
* [BeamSql]({{ site.baseurl }}/documentation/sdks/javadoc/{{ site.release_latest }}/index.html?org/apache/beam/sdk/extensions/sql/BeamSql.html): 用于从SQL查询创建PTransform的接口.

我们将在下面看一下这些.

# <a name="usage"></a>2. DSL APIs的用法 

## BeamRecord

将SQL查询应用于PCollection之前，集合中的数据必须为BeamRecord格式。 BeamRecord在Beam SQL PCollection中表示单个不可变行。记录中的字段/列的名称和类型由其关联的定义 [BeamRecordType]({{ site.baseurl }}/documentation/sdks/javadoc/{{ site.release_latest }}/index.html?org/apache/beam/sdk/values/BeamRecordType.html) ; 对于SQL查询，您应该使用 [BeamRecordSqlType]({{ site.baseurl }}/documentation/sdks/javadoc/{{ site.release_latest }}/index.html?org/apache/beam/sdk/extensions/sql/BeamRecordSqlType.html)  (查阅 [Data Types](#data-type) 获得有关支持的基本数据类型的更多详细信息).


`PCollection<BeamRecord>`可以显式或隐式地创建:

显式地:
  * **从内存中的数据** （通常用于单元测试）。在这种情况下，必须明确指定记录类型和编码器:
    ```
    // Define the record type (i.e., schema).
    List<String> fieldNames = Arrays.asList("appId", "description", "rowtime");
    List<Integer> fieldTypes = Arrays.asList(Types.INTEGER, Types.VARCHAR, Types.TIMESTAMP);
    BeamRecordSqlType appType = BeamRecordSqlType.create(fieldNames, fieldTypes);

    // Create a concrete row with that type.
    BeamRecord row = new BeamRecord(nameType, 1, "Some cool app", new Date());

    //create a source PCollection containing only that row.
    PCollection<BeamRecord> testApps = PBegin
        .in(p)
        .apply(Create.of(row)
                     .withCoder(nameType.getRecordCoder()));
    ```
  * **对于 `PCollection<T>`** 其中 `T` 不是 `BeamRecord`, 通过应用PTransform将输入记录转换为BeamRecord格式:
    ```
    // An example POJO class.
    class AppPojo {
      ...
      public final Integer appId;
      public final String description;
      public final Date timestamp;
    }

    // Acquire a collection of Pojos somehow.
    PCollection<AppPojo> pojos = ...

    // Convert them to BeamRecords with the same schema as defined above via a DoFn.
    PCollection<BeamRecord> apps = pojos.apply(
        ParDo.of(new DoFn<AppPojo, BeamRecord>() {
          @ProcessElement
          public void processElement(ProcessContext c) {
            c.output(new BeamRecord(appType, pojo.appId, pojo.description, pojo.timestamp));
          }
        }));
    ```


隐式地:
* **`BeamSql` `PTransform`** 应用于`PCollection<BeamRecord>`的结果 （下一节的细节）.

一旦你有一个PCollection <BeamRecord>，你可以使用BeamSql API来应用.

## BeamSql

`BeamSql` 提供了从SQL查询生成PTransform的两种方法，除了它们支持的输入数量之外，它们都是等效的:

* `BeamSql.query()`, 可以应用于单个PCollection。
在查询中 必须通过表名PCOLLECTION引用输入集合: 
  ```
  PCollection<BeamRecord> filteredNames = testApps.apply(
      BeamSql.query("SELECT appId, description, rowtime FROM PCOLLECTION WHERE id=1"));
  ```
* `BeamSql.queryMulti()`, 可以应用于包含一个或多个标记PCollection <BeamRecord>的PCollectionTuple。元组中每个PCollection的元组标记定义了可能用于查询它的表名。请注意，表名称绑定到特定的PCollectionTuple，因此仅在
查询的上下文中应用它有效.
  ```
  // Create a reviews PCollection to join to our apps PCollection.
  BeamRecordSqlType reviewType = BeamRecordSqlType.create(
    Arrays.asList("appId", "reviewerId", "rating", "rowtime"),
    Arrays.asList(Types.INTEGER, Types.INTEGER, Types.FLOAT, Types.TIMESTAMP));
  PCollection<BeamRecord> reviews = ... [records w/ reviewType schema] ...

  // Compute the # of reviews and average rating per app via a JOIN.
  PCollectionTuple namesAndFoods = PCollectionTuple.of(
      new TupleTag<BeamRecord>("Apps"), apps),
      new TupleTag<BeamRecord>("Reviews"), reviews));
  PCollection<BeamRecord> output = namesAndFoods.apply(
      BeamSql.queryMulti("SELECT Names.appId, COUNT(Reviews.rating), AVG(Reviews.rating)
                          FROM Apps INNER JOIN Reviews ON Apps.appId == Reviews.appId"));
  ```

这两种方法都包含了解析/验证/汇编的后端细节，并提供了一个Beam SDK样式的API，可以将简单的TABLE_FILTER查询表达到包含JOIN / GROUP_BY等的复杂查询. 

[BeamSqlExample](https://github.com/apache/beam/blob/DSL_SQL/sdks/java/extensions/sql/src/main/java/org/apache/beam/sdk/extensions/sql/example/BeamSqlExample.java) 在代码库中展示了两种API的基本用法.

# <a name="functionality"></a>3.  Beam SQL的功能
就像Beam中有界和无界数据的统一模型一样，SQL DSL也为有界和无界PCollection提供了相同的功能。以下是支持以 [BNF](http://en.wikipedia.org/wiki/Backus%E2%80%93Naur_Form)格式支持的SQL语法。对于不支持的功能，抛出UnsupportedOperationException异常.

```
query:
	{
          select
      |   query UNION [ ALL ] query
      |   query MINUS [ ALL ] query
      |   query INTERSECT [ ALL ] query
	}
    [ ORDER BY orderItem [, orderItem ]* LIMIT count [OFFSET offset] ]

orderItem:
      expression [ ASC | DESC ]

select:
      SELECT
          { * | projectItem [, projectItem ]* }
      FROM tableExpression
      [ WHERE booleanExpression ]
      [ GROUP BY { groupItem [, groupItem ]* } ]
      [ HAVING booleanExpression ]

projectItem:
      expression [ [ AS ] columnAlias ]
  |   tableAlias . *

tableExpression:
      tableReference [, tableReference ]*
  |   tableExpression [ ( LEFT | RIGHT ) [ OUTER ] ] JOIN tableExpression [ joinCondition ]

booleanExpression:
    expression [ IS NULL | IS NOT NULL ]
  | expression [ > | >= | = | < | <= | <> ] expression
  | booleanExpression [ AND | OR ] booleanExpression 
  | NOT booleanExpression
  | '(' booleanExpression ')'

joinCondition:
      ON booleanExpression

tableReference:
      tableName [ [ AS ] alias ]

values:
      VALUES expression [, expression ]*

groupItem:
      expression
  |   '(' expression [, expression ]* ')'
  |   HOP '(' expression [, expression ]* ')'
  |   TUMBLE '(' expression [, expression ]* ')'
  |   SESSION '(' expression [, expression ]* ')'

```

## <a name="features"></a>3.1. 支持的功能

**1. 聚合;**

Beam SQL支持global_window，fixed_window，sliding_window和session_window中的group_by的聚合功能。需要一个类型为TIMESTAMP的字段来指定fixed_window / sliding_window / session_window。该字段用作行的事件时间戳。见下面几个例子:

```
//fixed window, one hour in duration
SELECT f_int, COUNT(*) AS `size` FROM PCOLLECTION GROUP BY f_int, TUMBLE(f_timestamp, INTERVAL '1' HOUR)

//sliding window, one hour in duration and 30 minutes period
SELECT f_int, COUNT(*) AS `size` FROM PCOLLECTION GROUP BY f_int, HOP(f_timestamp, INTERVAL '1' HOUR, INTERVAL '30' MINUTE)

//session window, with 5 minutes gap duration
SELECT f_int, COUNT(*) AS `size` FROM PCOLLECTION GROUP BY f_int, SESSION(f_timestamp, INTERVAL '5' MINUTE)
```

注意: 

1. distinct 聚合还不支持.
2. 默认触发器是 `Repeatedly.forever(AfterWatermark.pastEndOfWindow())`;
3. 当 `time` 字段在 `HOP(dateTime, slide, size [, time ])`/`TUMBLE(dateTime, interval [, time ])`/`SESSION(dateTime, interval [, time ])` 被指定，一个后期触发器被添加为 

```
Repeatedly.forever(AfterWatermark.pastEndOfWindow().withLateFirings(AfterProcessingTime
        .pastFirstElementInPane().plusDelayOf(Duration.millis(delayTime.getTimeInMillis()))));
```		

**2. Join (inner, left_outer, right_outer);**

连接的情景可以分为3种情况:

1. BoundedTable JOIN BoundedTable
2. UnboundedTable JOIN UnboundedTable
3. BoundedTable JOIN UnboundedTable


对于情况1和情况2，只要两边的窗口windowFn匹配，则使用标准连接。对于情况3，使用sideInput来实现连接。到目前为止还有一些限制:

* 只支持相等连接，不支持CROSS JOIN;
* 不支持FULL OUTER JOIN;
* 如果这是一个左外连接, the unbounded table 应在左侧; 如果它是一个右外连接, the unbounded table 应该在右边;
* 窗口/触发器来自上游，这应该是一致的;

**3. 用户自定义函数 (UDF)和用户自定义聚合函数 (UDAF);**

如果所需功能不可用，开发人员可以注册自己的UDF（用于标量功能）和UDAF（用于聚合功能）.

**创建并指定用户定义的功能 (UDF)**

UDF可以是1）任何采用零个或多个标量字段并返回一个标量值的Java方法，或2）SerializableFunction。以下是UDF的一个例子，以及如何在DSL中使用它:

```
/**
 * A example UDF for test.
 */
public static class CubicInteger implements BeamSqlUdf{
  public static Integer eval(Integer input){
    return input * input * input;
  }
}

/**
 * Another example UDF with {@link SerializableFunction}.
 */
public static class CubicIntegerFn implements SerializableFunction<Integer, Integer> {
  @Override
  public Integer apply(Integer input) {
    return input * input * input;
  }
}

// register and call in SQL
String sql = "SELECT f_int, cubic1(f_int) as cubicvalue1, cubic2(f_int) as cubicvalue2 FROM PCOLLECTION WHERE f_int = 2";
PCollection<BeamSqlRow> result =
    input.apply("udfExample",
        BeamSql.simpleQuery(sql).withUdf("cubic1", CubicInteger.class)
		                        .withUdf("cubic2", new CubicIntegerFn()));
```

**创建并指定用户定义的聚合函数 (UDAF)**

Beam SQL 可以接受一个CombineFn作为UDAF. 下面是UDAF的一个例子:

```
/**
 * UDAF(CombineFn) for test, which returns the sum of square.
 */
public static class SquareSum extends CombineFn<Integer, Integer, Integer> {
  @Override
  public Integer createAccumulator() {
    return 0;
  }

  @Override
  public Integer addInput(Integer accumulator, Integer input) {
    return accumulator + input * input;
  }

  @Override
  public Integer mergeAccumulators(Iterable<Integer> accumulators) {
    int v = 0;
    Iterator<Integer> ite = accumulators.iterator();
    while (ite.hasNext()) {
      v += ite.next();
    }
    return v;
  }

  @Override
  public Integer extractOutput(Integer accumulator) {
    return accumulator;
  }

}

//register and call in SQL
String sql = "SELECT f_int1, squaresum(f_int2) AS `squaresum` FROM PCOLLECTION GROUP BY f_int2";
PCollection<BeamSqlRow> result =
    input.apply("udafExample",
        BeamSql.simpleQuery(sql).withUdaf("squaresum", new SquareSum()));
```
  
## <a name="data-type"></a>3.2. 数据类型

Beam SQL中的每个类型都映射到一个Java类以保存BeamRecord中的值。下表列出了当前存储库中支持的SQL类型和Java类之间的关系:

| SQL Type | Java class |
| ---- | ---- |
| Types.INTEGER | java.lang.Integer |
| Types.SMALLINT | java.lang.Short |
| Types.TINYINT | java.lang.Byte |
| Types.BIGINT | java.lang.Long |
| Types.FLOAT | java.lang.Float |
| Types.DOUBLE | java.lang.Double |
| Types.DECIMAL | java.math.BigDecimal |
| Types.VARCHAR | java.lang.String |
| Types.TIMESTAMP | java.util.Date |
{:.table}

## <a name="built-in-functions"></a>3.3. 内置的SQL函数

Beam SQL 已经实现了 [Apache Calcite](http://calcite.apache.org)中定义的大量内置函数. 可用的功能如下所示:

**比较函数和运算符**

| Operator syntax | Description |
| ---- | ---- |
| value1 = value2 | 等于 |
| value1 <> value2 | 不等于 |
| value1 > value2 | 大于 |
| value1 >= value2 | 大于等于 |
| value1 < value2 | 小于 |
| value1 <= value2 | 小于或等于 |
| value IS NULL | Whether value is null |
| value IS NOT NULL | Whether value is not null |
{:.table}

**逻辑函数和运算符**

| Operator syntax | Description |
| ---- | ---- |
| boolean1 OR boolean2 | boolean1是TRUE还是boolean2是TRUE |
| boolean1 AND boolean2 | boolean1和boolean2是否都为TRUE |
| NOT boolean | 布尔值不是TRUE;如果布尔值为UNKNOWN，则返回UNKNOWN |
{:.table}

**算术函数和运算符**

| Operator syntax | Description|
| ---- | ---- |
| numeric1 + numeric2 | 返回numeric1加numeric2| 
| numeric1 - numeric2 | 返回numeric1减numeric2| 
| numeric1 * numeric2 | 返回numeric1乘以numeric2| 
| numeric1 / numeric2 | 返回numeric1除以numeric2| 
| MOD(numeric, numeric) | 返回numeric1的余数（模数）除以numeric2。仅当numeric1为负值时，结果为负值| 
{:.table}

**数学函数**

| Operator syntax | Description |
| ---- | ---- |
| ABS(numeric) | 返回数字的绝对值 |
| SQRT(numeric) | 返回数字的平方根 |
| LN(numeric) | 返回数字的自然对数（基数e） |
| LOG10(numeric) | 返回数字的10位对数 |
| EXP(numeric) | 返回e的数字次方 |
| ACOS(numeric) | 返回数字的反余弦 |
| ASIN(numeric) | 返回数字的正弦 |
| ATAN(numeric) | 返回数字的反正切 |
| COT(numeric) | Returns the cotangent of numeric |
| DEGREES(numeric) | 将数字从弧度转换为度数 |
| RADIANS(numeric) | 将数字从度数转换为弧度 |
| SIGN(numeric) | 返回数字的符号 |
| SIN(numeric) | 返回数字的正弦 |
| TAN(numeric) | 返回数值的正切值 |
| ROUND(numeric1, numeric2) | 将numeric1到numeric2的数值舍入到小数点 |
{:.table}

**日期函数**

| Operator syntax | Description |
| ---- | ---- |
| LOCALTIME | 以数据类型TIME返回会话时区中的当前日期和时间 |
| LOCALTIME(precision) | 返回会话时区中数据类型TIME的当前日期和时间，精确到数字 |
| LOCALTIMESTAMP | 以数据类型TIMESTAMP的值返回会话时区中的当前日期和时间 |
| LOCALTIMESTAMP(precision) | 以数据类型TIMESTAMP的值返回会话时区中的当前日期和时间，精确到数字 |
| CURRENT_TIME | 返回会话时区中的当前时间，数据类型为TIMESTAMP WITH TIME ZONE |
| CURRENT_DATE | 返回会话时区中的当前日期，数据类型为DATE |
| CURRENT_TIMESTAMP | 返回会话时区中的当前日期和时间，数据类型为TIMESTAMP WITH TIME ZONE |
| EXTRACT(timeUnit FROM datetime) | 从datetime值表达式中提取并返回指定datetime字段的值 |
| FLOOR(datetime TO timeUnit) | 将datetime向下舍入到timeUnit |
| CEIL(datetime TO timeUnit) | 将datetime向上取到timeUnit |
| YEAR(date) | 相当于EXTRACT（YEAR FROM date）。返回一个整数. |
| QUARTER(date) | 相当于EXTRACT（QUARTER FROM date）。返回1到4之间的整数. |
| MONTH(date) | 相当于EXTRACT（MONTH FROM date）。返回1到12之间的整数. |
| WEEK(date) | 相当于EXTRACT（WEEK FROM date）。返回1到53之间的整数. |
| DAYOFYEAR(date) | 相当于EXTRACT（从日期开始）。返回1到366之间的整数. |
| DAYOFMONTH(date) | 相当于EXTRACT（D FROM FROM date）。返回1到31之间的整数. |
| DAYOFWEEK(date) | 相当于EXTRACT（DOW FROM date）。返回1到7之间的整数. |
| HOUR(date) | 相当于EXTRACT（HOUR FROM date）。返回0到23之间的整数. |
| MINUTE(date) | 相当于EXTRACT（MINUTE FROM date）。返回0到59之间的整数. |
| SECOND(date) | 相当于EXTRACT（从日期开始）。返回0到59之间的整数. |
{:.table}

**字符串函数**

| Operator syntax | Description |
| ---- | ---- |
| string \|\| string | 连接两个字符串 |
| CHAR_LENGTH(string) | 返回字符串中的字符数 |
| CHARACTER_LENGTH(string) | 同 CHAR_LENGTH(string) |
| UPPER(string) | 返回转换为大写字符的字符串 |
| LOWER(string) | 返回转换为小写字符的字符串 |
| POSITION(string1 IN string2) | 返回string1在string2中第一次出现的位置 |
| POSITION(string1 IN string2 FROM integer) | 返回在给定点（不是标准SQL）开始的string1在string2中第一次出现的位置 |
| TRIM( { BOTH \| LEADING \| TRAILING } string1 FROM string2) | 从string1的开始/结束/两端删除只包含string1中的字符的最长字符串 |
| OVERLAY(string1 PLACING string2 FROM integer [ FOR integer2 ]) | 用string2替换string1的子串 |
| SUBSTRING(string FROM integer) | 返回从给定点开始的字符串的子字符串 |
| SUBSTRING(string FROM integer FOR integer) | 返回从给定长度开始的字符串的子字符串 |
| INITCAP(string) | 返回字符串，每个字转换器的第一个字母大写，其余为小写。字是由非字母数字字符分隔的字母数字字符的序列. |
{:.table}

**条件函数**

| Operator syntax | Description |
| ---- | ---- |
| CASE value <br>WHEN value1 [, value11 ]* THEN result1 <br>[ WHEN valueN [, valueN1 ]* THEN resultN ]* <br>[ ELSE resultZ ] <br>END | Simple case |
| CASE <br>WHEN condition1 THEN result1 <br>[ WHEN conditionN THEN resultN ]* <br>[ ELSE resultZ ] <br>END | Searched case |
| NULLIF(value, value) | 如果值相同，则返回NULL。例如，NULLIF（5，5）返回NULL; NULLIF（5，0）返回5. |
| COALESCE(value, value [, value ]*) | 如果第一个值为空，则提供一个值。例如，COALESCE（NULL，5）返回5. |
{:.table}

**类型转换函数**

**聚合函数**

| Operator syntax | Description |
| ---- | ---- |
| COUNT(*) | 返回输入行数 |
| AVG(numeric) | 返回所有输入值之间的数字的平均值（算术平均值） |
| SUM(numeric) | 返回所有输入值之间的数字之和 |
| MAX(value) | 返回所有输入值的值的最大值 |
| MIN(value) | 返回所有输入值的最小值 |
{:.table}

# <a name="internal-of-sql"></a>4. Beam SQL的内部
图1描述了从SQL语句到Beam PTransform的后端步骤.

![Workflow of Beam SQL DSL]({{ "/images/beam_sql_dsl_workflow.png" | prepend: site.baseurl }} "workflow of Beam SQL DSL")

**图1** Beam SQL DSL工作流程

给定一个PCollection和查询作为输入，首先将输入PCollection注册为模式存储库中的一个表。然后它被处理为:

1. 根据语法对SQL查询进行解析，生成SQL抽象语法树;
2. 验证表结构，并输出用关系代数表示的逻辑计划;
3. 应用关系规则将逻辑计划转换为物理计划，表示为 Beam组件. 优化器是可选的，以更新计划;
4. 最终, Beam 物理计划被编译为复合 `PTransform`;

下面是一个例子展示了一个从输入`PCollection`过滤和项目的查询 :

```
SELECT USER_ID, USER_NAME FROM PCOLLECTION WHERE USER_ID = 1
```

逻辑计划显示为:

```
LogicalProject(USER_ID=[$0], USER_NAME=[$1])
  LogicalFilter(condition=[=($0, 1)])
    LogicalTableScan(table=[[PCOLLECTION]])
```

并编译为复合 `PTransform`

```
pCollection.apply(BeamSqlFilter...)
           .apply(BeamSqlProject...)
```

