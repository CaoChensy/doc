#! https://zhuanlan.zhihu.com/p/454208365

![Image](https://pic4.zhimg.com/80/v2-5db1a82996ec388725185ae900a58008.jpg)

# PySpark UDF

## 1. UDF 基本概念

### 1.1 什么是UDF？

UDF 用户定义函数，`PySpark UDF` 类似于传统数据库上的 `UDF`。 `PySpark SQL Functions` 不能满足业务要求时，需要使用 UDF 进行自定义函数。

一般步骤是，首先使用 `Python` 语法创建一个函数，并使用 `PySpark SQL` 包装它为`udf()`，然后在 `DataFrame` 上使用。

### 1.2 为什么需要UDF？

UDF 用于扩展框架的功能并在多个 DataFrame 上重用这些功能。
例如，您想将名称字符串中单词的每个首字母都转换为大写；
`PySpark` 没有此函数，您可以创建 `UDF`，并根据需要在多个`DataFrame`上重用它。

## 2 创建 `PySpark UDF`
```python
import numpy as np
import pandas as pd
from pyspark.sql.types import *
from pyspark.sql import functions as F
```

### 2.1 首先创建一个 `PySpark DataFrame`
```python
columns = ["Seqno","Name"]
data = [("1", "john jones"),
    ("2", "tracey smith"),
    ("3", "amy sanders")]

df = spark.createDataFrame(data=data, schema=columns)
df.show(truncate=False)
```
```bash
>>> output Data:
>>>
+-----+------------+
|Seqno|Name        |
+-----+------------+
|1    |john jones  |
|2    |tracey smith|
|3    |amy sanders |
+-----+------------+

```

### 2.2 创建 Python 函数

创建 `Python` 函数。它接受一个字符串参数并将每个单词的第一个字母转换为大写字母。
```python
def convertCase(string):
    resStrArr=[]
    stringArr = string.split(" ")
    for x in stringArr:
        resStrArr.append(f"{x[0].upper()}{x[1:]}")
    return ' '.join(resStrArr)

# 测试一下
convertCase('john jones')
```
```bash
>>> output Data:
>>> 'John Jones'
```

### 2.3 将 `Python` 函数转换为 `PySpark UDF`

现在`convertCase()`通过将函数传递给 `PySpark SQL` 来将此函数转换为 `UDF`。

#### 方式 一：lambda
```python
# returnType 为返回数据的数据类型
convert_udf_lambda = F.udf(lambda z: convertCase(z), returnType=StringType())
```

#### 方式二：直接传入函数
```python
convert_udf = F.udf(f=convertCase, returnType=StringType())
```

#### 方式三：装饰器
```python
@F.udf(returnType=StringType())
def convertCaseDecorate(string):
    resStrArr=[]
    stringArr = string.split(" ")
    for x in stringArr:
        resStrArr.append(f"{x[0].upper()}{x[1:]}")
    return ' '.join(resStrArr)
```

### 2.4 在 `DataFrame` 中使用 `UDF`

#### 在 `PySpark DataFrame select()` 中使用 `UDF`
```python
# lambda UDF
df.select(F.col("Seqno"), convert_udf_lambda(F.col("Name")).alias("Name")).show()
# functions UDF
df.select(F.col("Seqno"), convert_udf(F.col("Name")).alias("Name")).show()
# 装饰器 UDF
df.select(F.col("Seqno"), convertCaseDecorate(F.col("Name")).alias("Name")).show()
```
```bash
>>> output Data:
>>>
+-----+------------+
|Seqno|        Name|
+-----+------------+
|    1|  John Jones|
|    2|Tracey Smith|
|    3| Amy Sanders|
+-----+------------+

+-----+------------+
|Seqno|        Name|
+-----+------------+
|    1|  John Jones|
|    2|Tracey Smith|
|    3| Amy Sanders|
+-----+------------+

+-----+------------+
|Seqno|        Name|
+-----+------------+
|    1|  John Jones|
|    2|Tracey Smith|
|    3| Amy Sanders|
+-----+------------+

```

#### 上面三种`UDF`的结果都是一致的。

#### 在 `PySpark DataFrame withColumn()` 中使用 `UDF`
```python
df.withColumn("Cureated Name", convert_udf(F.col("Name"))).show(truncate=False)
```
```bash
>>> output Data:
>>>
+-----+------------+-------------+
|Seqno|Name        |Cureated Name|
+-----+------------+-------------+
|1    |john jones  |John Jones   |
|2    |tracey smith|Tracey Smith |
|3    |amy sanders |Amy Sanders  |
+-----+------------+-------------+

```

#### 注册 `PySpark UDF` 并在 `SQL` 上使用4

为了`convertCase()`在 `PySpark SQL` 上使用函数，您需要使用`spark.udf.register()`。

```python
spark.udf.register("convert_udf", convertCase, StringType())
df.createOrReplaceTempView("NAME_TABLE")
spark.sql("select Seqno, convertUDF(Name) as Name from NAME_TABLE").show(truncate=False)
```

### 2.5 空值检查

当您有一列包含`null`记录的值时，如果设计不仔细，`UDF` 很容易出错。
```python
columns = ["Seqno","Name"]
data = [("1", "john jones"),
    ("2", "tracey smith"),
    ("3", "amy sanders"),
    ('4',None)]

df2 = spark.createDataFrame(data=data,schema=columns)
df2.show(truncate=False)
```
```bash
>>> output Data:
>>>
+-----+------------+
|Seqno|Name        |
+-----+------------+
|1    |john jones  |
|2    |tracey smith|
|3    |amy sanders |
|4    |null        |
+-----+------------+
```

```python
df2.withColumn("Cureated Name", convert_udf(F.col("Name"))).show(truncate=False)

AttributeError: 'NoneType' object has no attribute 'split'
```

请注意，从上面的代码片段中，`Seqno 4`的`name`值为`None`。
由于 `UDF` 函数没有处理 `null`，因此在 `DataFrame` 上使用它会返回错误。
在 `Python` 中 `None` 被认为是 `null`。

#### 以下修改`UDF`以应对空值情况
```python
@F.udf(returnType=StringType())
def convertCaseDecorate(string):
    if string is None:
        return None
    else:
        resStrArr=[]
        stringArr = string.split(" ")
        for x in stringArr:
            resStrArr.append(f"{x[0].upper()}{x[1:]}")
        return ' '.join(resStrArr)

df2.withColumn("Cureated Name", convertCaseDecorate(F.col("Name"))).show()
```
```bash
>>> output Data:
>>>
+-----+------------+-------------+
|Seqno|        Name|Cureated Name|
+-----+------------+-------------+
|    1|  john jones|   John Jones|
|    2|tracey smith| Tracey Smith|
|    3| amy sanders|  Amy Sanders|
|    4|        null|         null|
+-----+------------+-------------+
```

## 3. `UDF` 输入输出结构

### 3.1 One in One out

以一列作为输入，输出为另一列。
```python
columns = ["Seqno","Name"]
data = [("1", "john jones"),
    ("2", "tracey smith"),
    ("3", "amy sanders")]

df = spark.createDataFrame(data=data, schema=columns)
df.show(truncate=False)

@F.udf(returnType=StringType())
def convertCaseDecorate(string):
    resStrArr=[]
    stringArr = string.split(" ")
    for x in stringArr:
        resStrArr.append(f"{x[0].upper()}{x[1:]}")
    return ' '.join(resStrArr)

df = df.withColumn("Cureated Name", convertCaseDecorate(F.col("Name")))
df.show()
```
```bash
>>> output Data:
>>>
+-----+------------+
|Seqno|Name        |
+-----+------------+
|1    |john jones  |
|2    |tracey smith|
|3    |amy sanders |
+-----+------------+

+-----+------------+-------------+
|Seqno|        Name|Cureated Name|
+-----+------------+-------------+
|    1|  john jones|   John Jones|
|    2|tracey smith| Tracey Smith|
|    3| amy sanders|  Amy Sanders|
+-----+------------+-------------+

```

### 3.2 Many In One Out

以两列或多个列为输入，以另一列作为输出。
```python
@F.udf(returnType=StringType())
def ManyInOneOut(name_1, name_2):
    return f'{name_1}-{name_2}'

df.withColumn(
    "Cureated Name", 
    ManyInOneOut(F.col("Name"), F.col("Cureated Name"))
).show()
```
```bash
>>> output Data:
>>>
+-----+------------+--------------------+
|Seqno|        Name|       Cureated Name|
+-----+------------+--------------------+
|    1|  john jones|john jones-John J...|
|    2|tracey smith|tracey smith-Trac...|
|    3| amy sanders|amy sanders-Amy S...|
+-----+------------+--------------------+

```

### 3.3 Many In Many Out

以两列或多个列为输入，以多个列作为输出。
```python
schema = StructType([
    StructField("sum", FloatType(), False),
    StructField("diff", FloatType(), False)])

@F.udf(returnType=schema)
def sum_diff(f1, f2):
    return [f1 + f2, f1-f2]

df = spark.createDataFrame(
    pd.DataFrame([[1., 2.], [2., 4.]], columns=['a', 'b']))

df_new = df.withColumn("calculate", sum_diff(F.col('a'), F.col('b')))
df_new.show()
```
```bash
>>> output Data:
>>>
+---+---+-----------+
|  a|  b|  calculate|
+---+---+-----------+
|1.0|2.0|[3.0, -1.0]|
|2.0|4.0|[6.0, -2.0]|
+---+---+-----------+

```
```python
# 最终表结构
df_new.printSchema()
```
```bash
>>> output Data:
>>>
root
 |-- a: double (nullable = true)
 |-- b: double (nullable = true)
 |-- calculate: struct (nullable = true)
 |    |-- sum: float (nullable = false)
 |    |-- diff: float (nullable = false)

```
```python
df_new.select('*', 'calculate.*', 'calculate.sum', 'calculate.diff').show()
```
```bash
>>> output Data:
>>>
+---+---+-----------+---+----+---+----+
|  a|  b|  calculate|sum|diff|sum|diff|
+---+---+-----------+---+----+---+----+
|1.0|2.0|[3.0, -1.0]|3.0|-1.0|3.0|-1.0|
|2.0|4.0|[6.0, -2.0]|6.0|-2.0|6.0|-2.0|
+---+---+-----------+---+----+---+----+

```

## 4. 闭包构造`UDF`

当我们想传入`UDF`两个参数时，其中一个参数为固定参数，就像下面的示例，需要向`state_abbreviation`函数传入`s`与`mapping`参数，以期望用字典`mapping`中的键值对信息替换`s`中的信息，使用以下构造方式进行运算。

```python
@F.udf(returnType=StringType())
def state_abbreviation(s, mapping):
    if s is not None:
        return mapping[s]

df = spark.createDataFrame([['Alabama',], ['Texas',], ['Antioquia',]]).toDF('state')
mapping = {'Alabama': 'AL', 'Texas': 'TX'}
df.withColumn('state_abbreviation', state_abbreviation(F.col('state'), mapping)).show()
```

> 会报出以下错误
```python
TypeError: Invalid argument, not a string or column: {'Alabama': 'AL', 'Texas': 'TX'} of type <class 'dict'>. For column literals, use 'lit', 'array', 'struct' or 'create_map' function.
```

### 4.1 考虑使用闭包方式构造`UDF`
```python
df = spark.createDataFrame([['Alabama',], ['Texas',], ['Antioquia',]]).toDF('state')

def working_fun(mapping):
    def f(x):
        return mapping.get(x)
    return F.udf(f)

mapping = {'Alabama': 'AL', 'Texas': 'TX'}
df.withColumn('state_abbreviation', working_fun(mapping)(F.col('state'))).show()
```
```bash
>>> output Data:
>>>
+---------+------------------+
|    state|state_abbreviation|
+---------+------------------+
|  Alabama|                AL|
|    Texas|                TX|
|Antioquia|              null|
+---------+------------------+

```

### 4.2 考虑使用`broadcast`方式进行运算

该方法通过 `spark.sparkContext.broadcast` 将 `mapping` 广播到全部运算节点上。
```python
@F.udf(returnType=StringType())
def working_fun(x):
    return mapping_broadcasted.value.get(x)

mapping_broadcasted = spark.sparkContext.broadcast(mapping)
df.withColumn('state_abbreviation', working_fun(F.col('state'))).show()
```
```bash
>>> output Data:
>>>
+---------+------------------+
|    state|state_abbreviation|
+---------+------------------+
|  Alabama|                AL|
|    Texas|                TX|
|Antioquia|              null|
+---------+------------------+

```

## 5. 在`GroupBy`中使用`UDF`

以下计算每个人的贷款总合，以`UDF`的形式进行计算。
```python
df = spark.createDataFrame(
    [['1', 'bob', 10], ['1', 'bob', 20],
     ['1', 'bob', 19], ['1', 'bob', 20],
     ['2', 'nic', 11], ['1', 'nic', 8],
     ['2', 'nic', 11], ['1', 'nic', 9],
     ['3', 'ace', 12], ['1', 'ace', 20],
     ['3', 'ace', 1], ['1', 'ace', 20],],
    ['id', 'name', 'loan'])
df.show()
```
```bash
>>> output Data:
>>>
+---+----+----+
| id|name|loan|
+---+----+----+
|  1| bob|  10|
|  1| bob|  20|
|  1| bob|  19|
|  1| bob|  20|
|  2| nic|  11|
|  1| nic|   8|
|  2| nic|  11|
|  1| nic|   9|
|  3| ace|  12|
|  1| ace|  20|
|  3| ace|   1|
|  1| ace|  20|
+---+----+----+

```
```python
# 首先进行分组，汇总每个人的贷款金额
df_group = df.groupBy('name').agg(F.collect_list('loan').alias('loan'))
df_group.show()
```
```bash
>>> output Data:
>>>
+----+----------------+
|name|            loan|
+----+----------------+
| nic|  [11, 8, 11, 9]|
| ace| [12, 20, 1, 20]|
| bob|[10, 20, 19, 20]|
+----+----------------+
```
```python
# 定义 UDF 
@F.udf(returnType=IntegerType())
def func(array):
    return sum(array)

df_group = df_group.withColumn('sum', func(F.col('loan')))
df_group.show()
```
```bash
>>> output Data:
>>>
+----+----------------+---+
|name|            loan|sum|
+----+----------------+---+
| nic|  [11, 8, 11, 9]| 39|
| ace| [12, 20, 1, 20]| 53|
| bob|[10, 20, 19, 20]| 69|
+----+----------------+---+
```

---
