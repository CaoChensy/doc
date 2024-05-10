#! https://zhuanlan.zhihu.com/p/446607001

![Image](https://pic4.zhimg.com/80/v2-5db1a82996ec388725185ae900a58008.jpg)

# 【PySpark】窗口函数Window

`PySpark Window` 函数用于计算输入行范围内的结果，例如排名、行号等。在本文中，我解释了窗口函数的概念、语法，最后解释了如何将它们与 `PySpark SQL` 和 `PySpark DataFrame API` 一起使用。当我们需要在 `DataFrame` 列的特定窗口中进行聚合操作时，这些会派上用场。`Window`函数在实际业务场景中非常实用，用的好的话能避免很多浪费时间的计算。

Window函数分类为三种：

- 排名函数 `ranking functions`包括:
    - row_number()
    - rank()
    - dense_rank()
    - percent_rank()
    - ntile()

- 解析函数 `analytic functions`包括:
    - cume_dist()
    - lag()
    - lead()

- 聚合函数 `aggregate functions`包括:
    - sum()
    - first()
    - last()
    - max()
    - min()
    - mean()
    - stddev()

下面依次详解上述三类函数。
```python
from pyspark.sql.window import Window
import pyspark.sql.functions as F
```

## 1. 创建一个 PySpark DataFrame
```python
employee_salary = [
    ("Ali", "Sales", 8000),
    ("Bob", "Sales", 7000),
    ("Cindy", "Sales", 7500),
    ("Davd", "Finance", 10000),
    ("Elena", "Sales", 8000),
    ("Fancy", "Finance", 12000),
    ("George", "Finance", 11000),
    ("Haffman", "Marketing", 7000),
    ("Ilaja", "Marketing", 8000),
    ("Joey", "Sales", 9000)]
 
columns= ["name", "department", "salary"]
df = spark.createDataFrame(data = employee_salary, schema = columns)
df.show(truncate=False)
```
```bash
>>> output Data:
>>>
+-------+----------+------+
|name   |department|salary|
+-------+----------+------+
|Ali    |Sales     |8000  |
|Bob    |Sales     |7000  |
|Cindy  |Sales     |7500  |
|Davd   |Finance   |10000 |
|Elena  |Sales     |8000  |
|Fancy  |Finance   |12000 |
|George |Finance   |11000 |
|Haffman|Marketing |7000  |
|Ilaja  |Marketing |8000  |
|Joey   |Sales     |9000  |
+-------+----------+------+

```

## 2. 窗口函数 `ranking functions`

### 2.1 `row_number()`

`row_number()` 窗口函数用于给出从1开始到每个窗口分区的结果的连续行号。
与 `groupBy` 不同 `Window` 以 `partitionBy` 作为分组条件，`orderBy` 对 `Window` 分组内的数据进行排序。
```python
# 以 department 字段进行分组，以 salary 倒序排序
# 按照部门对薪水排名，薪水最低的为第一名
windowSpec = Window.partitionBy("department").orderBy(F.asc("salary"))
# 分组内增加 row_number
df_part = df.withColumn(
    "row_number", 
    F.row_number().over(windowSpec)
)
print(df_part.toPandas().to_markdown())
```
```bash
>>> output Data:
>>>
|    | name    | department   |   salary |   row_number |
|---:|:--------|:-------------|---------:|-------------:|
|  0 | Bob     | Sales        |     7000 |            1 |
|  1 | Cindy   | Sales        |     7500 |            2 |
|  2 | Ali     | Sales        |     8000 |            3 |
|  3 | Elena   | Sales        |     8000 |            4 |
|  4 | Joey    | Sales        |     9000 |            5 |
|  5 | Davd    | Finance      |    10000 |            1 |
|  6 | George  | Finance      |    11000 |            2 |
|  7 | Fancy   | Finance      |    12000 |            3 |
|  8 | Haffman | Marketing    |     7000 |            1 |
|  9 | Ilaja   | Marketing    |     8000 |            2 |
```

观察上面的数据，发现同样的薪水会有不同的排名（都是$8000$的薪水，有的第二有的第三），这是因为`row_number()`是按照行来给定序号，其不关注实际数值的大小。由此我们可以引申出另一个用于给出排序数的函数$rank$。

#### 使用场景



- 选取本部门工资收入第$N$高的记录

- （思考）选取某日第$N$笔交易记录
```python
print(df_part.where(F.col('row_number') == 2).toPandas().to_markdown())
```
```bash
>>> output Data:
>>>
|    | name   | department   |   salary |   row_number |
|---:|:-------|:-------------|---------:|-------------:|
|  0 | Cindy  | Sales        |     7500 |            2 |
|  1 | George | Finance      |    11000 |            2 |
|  2 | Ilaja  | Marketing    |     8000 |            2 |
```

### 2.2 `rank()`

`rank()`用来给按照指定列排序的分组窗增加一个排序的序号，

如果有相同数值，则排序数相同，下一个序数顺延一位。来看如下代码：
```python
# 使用 rank 排序，都是8000的薪水，就同列第二
windowSpec = Window.partitionBy("department").orderBy(F.desc("salary"))
df_rank = df.withColumn("rank", F.rank().over(windowSpec))
print(df_rank.toPandas().to_markdown())
```
```bash
>>> output Data:
>>>
|    | name    | department   |   salary |   rank |
|---:|:--------|:-------------|---------:|-------:|
|  0 | Joey    | Sales        |     9000 |      1 |
|  1 | Ali     | Sales        |     8000 |      2 |
|  2 | Elena   | Sales        |     8000 |      2 |
|  3 | Cindy   | Sales        |     7500 |      4 |
|  4 | Bob     | Sales        |     7000 |      5 |
|  5 | Fancy   | Finance      |    12000 |      1 |
|  6 | George  | Finance      |    11000 |      2 |
|  7 | Davd    | Finance      |    10000 |      3 |
|  8 | Ilaja   | Marketing    |     8000 |      1 |
|  9 | Haffman | Marketing    |     7000 |      2 |
```
```python
print(df_rank.where(F.col("rank")=="2").toPandas().to_markdown())
```
```bash
>>> output Data:
>>>
|    | name    | department   |   salary |   rank |
|---:|:--------|:-------------|---------:|-------:|
|  0 | Ali     | Sales        |     8000 |      2 |
|  1 | Elena   | Sales        |     8000 |      2 |
|  2 | George  | Finance      |    11000 |      2 |
|  3 | Haffman | Marketing    |     7000 |      2 |
```

### 2.3 `dense_rank`

观察 `dense_rank` 与 `rank` 的区别。
```python
# 注意 rank 排序，8000虽然为同列第二，但7500属于第4名
# dense_rank()中， 8000同列第二后，7500属于第3名
windowSpec  = Window.partitionBy("department").orderBy(F.desc("salary"))
df.withColumn("dense_rank", F.dense_rank().over(windowSpec)).show()
```
```bash
>>> output Data:
>>>
+-------+----------+------+----------+
|   name|department|salary|dense_rank|
+-------+----------+------+----------+
|   Joey|     Sales|  9000|         1|
|    Ali|     Sales|  8000|         2|
|  Elena|     Sales|  8000|         2|
|  Cindy|     Sales|  7500|         3|
|    Bob|     Sales|  7000|         4|
|  Fancy|   Finance| 12000|         1|
| George|   Finance| 11000|         2|
|   Davd|   Finance| 10000|         3|
|  Ilaja| Marketing|  8000|         1|
|Haffman| Marketing|  7000|         2|
+-------+----------+------+----------+

```

### 2.4 percent_rank()

一些业务场景下，我们需要计算不同数值的百分比排序数据，先来看一个例子吧。
```python
windowSpec  = Window.partitionBy("department").orderBy(F.desc("salary"))
df.withColumn("percent_rank",F.percent_rank().over(windowSpec)).show()
```
```bash
>>> output Data:
>>>
+-------+----------+------+------------+
|   name|department|salary|percent_rank|
+-------+----------+------+------------+
|   Joey|     Sales|  9000|         0.0|
|    Ali|     Sales|  8000|        0.25|
|  Elena|     Sales|  8000|        0.25|
|  Cindy|     Sales|  7500|        0.75|
|    Bob|     Sales|  7000|         1.0|
|  Fancy|   Finance| 12000|         0.0|
| George|   Finance| 11000|         0.5|
|   Davd|   Finance| 10000|         1.0|
|  Ilaja| Marketing|  8000|         0.0|
|Haffman| Marketing|  7000|         1.0|
+-------+----------+------+------------+

```

上述结果可以理解为将 `dense_rank()` 的结果进行归一化，
即可得到`0-1`以内的百分数。`percent_rank()` 与 `SQL` 中的 `PERCENT_RANK` 函数效果一致。

### 2.5 `ntile()`

`ntile()`可将分组的数据按照指定数值`n`切分为`n`个部分，
每一部分按照行的先后给定相同的序数。例如n指定为2，则将组内数据分为两个部分，
第一部分序号为1，第二部分序号为2。理论上两部分数据行数是均等的，
但当数据为奇数行时，中间的那一行归到前一部分。

按照部门对数据进行分组，然后在组内按照薪水高低进行排序，
再使用 `ntile()` 将组内数据切分为两个部分。结果如下：
```python
# 按照部门对数据进行分组，然后在组内按照薪水高低进行排序 
windowSpec = Window.partitionBy(
    "department").orderBy(F.desc("salary"))
# 使用ntile() 将组内数据切分为两个部分
df.withColumn("ntile", F.ntile(2).over(windowSpec)).show()
```
```bash
>>> output Data:
>>>
+-------+----------+------+-----+
|   name|department|salary|ntile|
+-------+----------+------+-----+
|   Joey|     Sales|  9000|    1|
|    Ali|     Sales|  8000|    1|
|  Elena|     Sales|  8000|    1|
|  Cindy|     Sales|  7500|    2|
|    Bob|     Sales|  7000|    2|
|  Fancy|   Finance| 12000|    1|
| George|   Finance| 11000|    1|
|   Davd|   Finance| 10000|    2|
|  Ilaja| Marketing|  8000|    1|
|Haffman| Marketing|  7000|    2|
+-------+----------+------+-----+

```

## 3. Analytic functions

### 3.1 `cume_dist()`

`cume_dist()`函数用来获取数值的累进分布值，看如下例子：
```python
windowSpec = Window.partitionBy("department").orderBy(F.desc("salary"))
df.withColumn(
    "cume_dist", F.cume_dist().over(windowSpec)).show()
```
```bash
>>> output Data:
>>>
+-------+----------+------+------------------+
|   name|department|salary|         cume_dist|
+-------+----------+------+------------------+
|   Joey|     Sales|  9000|               0.2|
|    Ali|     Sales|  8000|               0.6|
|  Elena|     Sales|  8000|               0.6|
|  Cindy|     Sales|  7500|               0.8|
|    Bob|     Sales|  7000|               1.0|
|  Fancy|   Finance| 12000|0.3333333333333333|
| George|   Finance| 11000|0.6666666666666666|
|   Davd|   Finance| 10000|               1.0|
|  Ilaja| Marketing|  8000|               0.5|
|Haffman| Marketing|  7000|               1.0|
+-------+----------+------+------------------+

```
```python
# 和 percent_rank 对比一下
df.withColumn(
    'percent_rank',
    F.percent_rank().over(windowSpec)).show()
```
```bash
>>> output Data:
>>>
+-------+----------+------+------------+
|   name|department|salary|percent_rank|
+-------+----------+------+------------+
|   Joey|     Sales|  9000|         0.0|
|    Ali|     Sales|  8000|        0.25|
|  Elena|     Sales|  8000|        0.25|
|  Cindy|     Sales|  7500|        0.75|
|    Bob|     Sales|  7000|         1.0|
|  Fancy|   Finance| 12000|         0.0|
| George|   Finance| 11000|         0.5|
|   Davd|   Finance| 10000|         1.0|
|  Ilaja| Marketing|  8000|         0.0|
|Haffman| Marketing|  7000|         1.0|
+-------+----------+------+------------+

```

结果好像和前面的`percent_rank()`很类似对不对，于是我们联想到这个其实也是一种归一化结果，
其按照 `rank()` 的结果进行归一化处理。回想一下前面讲过的 `rank()` 函数，并列排序会影响后续排序，
于是序号中间可能存在隔断。这样Sales组的排序数就是1、2、2、4、5，
归一化以后就得到了0.2、0.6、0.6、0.8、1。这个统计结果按照实际业务来理解就是：

- 9000及以上的人占了20%，
- 8000及以上的人占了60%，
- 7500以上的人数占了80%，
- 7000以上的人数占了100%，

### 3.2 `lag()`

`lag()` 函数用于寻找按照指定列排好序的分组内每个数值的上一个数值，

通俗的说，就是数值排好序以后，寻找排在每个数值的上一个数值。代码如下：
```python
# 相当于滞后项
windowSpec  = Window.partitionBy("department").orderBy(F.desc("salary"))
df.withColumn("lag", F.lag("salary",1).over(windowSpec)).show()
```
```bash
>>> output Data:
>>>
+-------+----------+------+-----+
|   name|department|salary|  lag|
+-------+----------+------+-----+
|   Joey|     Sales|  9000| null|
|    Ali|     Sales|  8000| 9000|
|  Elena|     Sales|  8000| 8000|
|  Cindy|     Sales|  7500| 8000|
|    Bob|     Sales|  7000| 7500|
|  Fancy|   Finance| 12000| null|
| George|   Finance| 11000|12000|
|   Davd|   Finance| 10000|11000|
|  Ilaja| Marketing|  8000| null|
|Haffman| Marketing|  7000| 8000|
+-------+----------+------+-----+

```

### 3.3 `lead()`

`lead()` 用于获取排序后的数值的下一个，代码如下：
```python
# 和滞后项相反，提前一位
windowSpec  = Window.partitionBy("department").orderBy(F.desc("salary"))
df.withColumn("lead",F.lead("salary",1).over(windowSpec)).show()
```
```bash
>>> output Data:
>>>
+-------+----------+------+-----+
|   name|department|salary| lead|
+-------+----------+------+-----+
|   Joey|     Sales|  9000| 8000|
|    Ali|     Sales|  8000| 8000|
|  Elena|     Sales|  8000| 7500|
|  Cindy|     Sales|  7500| 7000|
|    Bob|     Sales|  7000| null|
|  Fancy|   Finance| 12000|11000|
| George|   Finance| 11000|10000|
|   Davd|   Finance| 10000| null|
|  Ilaja| Marketing|  8000| 7000|
|Haffman| Marketing|  7000| null|
+-------+----------+------+-----+

```

1. 实际业务场景中，假设我们获取了每个月的销售数据，
我们可能想要知道，某月份与上一个月或下一个月数据相比怎么样，
于是就可以使用`lag`和`lead`来进行数据分析了。

1. 思考差分如何做？增长率如何做（同比、环比）？

## 4. Aggregate Functions

常见的聚合函数有`avg, sum, min, max, count, approx_count_distinct()`等，我们用如下代码来同时使用这些函数：
```python
# 分组，并对组内数据排序
windowSpec  = Window.partitionBy("department").orderBy(F.desc("salary"))
# 仅分组
windowSpecAgg  = Window.partitionBy("department")

df.withColumn("row", F.row_number().over(windowSpec)) \
  .withColumn("avg", F.avg("salary").over(windowSpecAgg)) \
  .withColumn("sum", F.sum("salary").over(windowSpecAgg)) \
  .withColumn("min", F.min("salary").over(windowSpecAgg)) \
  .withColumn("max", F.max("salary").over(windowSpecAgg)) \
  .withColumn("count", F.count("salary").over(windowSpecAgg)) \
  .withColumn("distinct_count", F.approxCountDistinct("salary").over(windowSpecAgg)) \
  .show()
```
```bash
>>> output Data:
>>>
+-------+----------+------+---+-------+-----+-----+-----+-----+--------------+
|   name|department|salary|row|    avg|  sum|  min|  max|count|distinct_count|
+-------+----------+------+---+-------+-----+-----+-----+-----+--------------+
|   Joey|     Sales|  9000|  1| 7900.0|39500| 7000| 9000|    5|             4|
|    Ali|     Sales|  8000|  2| 7900.0|39500| 7000| 9000|    5|             4|
|  Elena|     Sales|  8000|  3| 7900.0|39500| 7000| 9000|    5|             4|
|  Cindy|     Sales|  7500|  4| 7900.0|39500| 7000| 9000|    5|             4|
|    Bob|     Sales|  7000|  5| 7900.0|39500| 7000| 9000|    5|             4|
|  Fancy|   Finance| 12000|  1|11000.0|33000|10000|12000|    3|             3|
| George|   Finance| 11000|  2|11000.0|33000|10000|12000|    3|             3|
|   Davd|   Finance| 10000|  3|11000.0|33000|10000|12000|    3|             3|
|  Ilaja| Marketing|  8000|  1| 7500.0|15000| 7000| 8000|    2|             2|
|Haffman| Marketing|  7000|  2| 7500.0|15000| 7000| 8000|    2|             2|
+-------+----------+------+---+-------+-----+-----+-----+-----+--------------+

```

需要注意的是 `approx_count_distinct()` 函数适用与窗函数的统计，
而在`groupby`中通常用`countDistinct()`来代替该函数，用来求组内不重复的数值的条数。

从结果来看，统计值基本上是按照部门分组，统计组内的salary情况。
如果我们只想要保留部门的统计结果，而将每个人的实际情况去掉，可以采用如下代码：
```python
windowSpec  = Window.partitionBy("department").orderBy(F.desc("salary"))
windowSpecAgg  = Window.partitionBy("department")

df = df.withColumn("row", F.row_number().over(windowSpec)) \
  .withColumn("avg", F.avg("salary").over(windowSpecAgg)) \
  .withColumn("sum", F.sum("salary").over(windowSpecAgg)) \
  .withColumn("min", F.min("salary").over(windowSpecAgg)) \
  .withColumn("max", F.max("salary").over(windowSpecAgg)) \
  .withColumn("count", F.count("salary").over(windowSpecAgg)) \
  .withColumn("distinct_count", F.approx_count_distinct("salary").over(windowSpecAgg))

# 仅选取分组第一行数据
# 用F.col 去选row 行，怪怪的
df_part  = df.where(F.col("row")==1)
df_part.select("department","avg","sum","min","max","count","distinct_count").show()
```
```bash
>>> output Data:
>>>
+----------+-------+-----+-----+-----+-----+--------------+
|department|    avg|  sum|  min|  max|count|distinct_count|
+----------+-------+-----+-----+-----+-----+--------------+
|     Sales| 7900.0|39500| 7000| 9000|    5|             4|
|   Finance|11000.0|33000|10000|12000|    3|             3|
| Marketing| 7500.0|15000| 7000| 8000|    2|             2|
+----------+-------+-----+-----+-----+-----+--------------+

```

---
