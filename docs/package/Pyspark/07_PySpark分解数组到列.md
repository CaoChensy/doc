#! https://zhuanlan.zhihu.com/p/450923475

![Image](https://pic4.zhimg.com/80/v2-5db1a82996ec388725185ae900a58008.jpg)

# PySpark 分解数组到列

使用不同的 `PySpark DataFrame` 函数分解数组或列表并映射到列。

- `explode, explode_outer, poseexplode, posexplode_outer`

在开始之前，让我们创建一个带有数组和字典字段的 `DataFrame`

## 1. 创建数据
```python
arrayData = [
        ('James',['Java','Scala'],{'hair':'black','eye':'brown'}),
        ('Michael',['Spark','Java',None],{'hair':'brown','eye':None}),
        ('Robert',['CSharp',''],{'hair':'red','eye':''}),
        ('Washington',None,None),
        ('Jefferson',['1','2'],{})]

df = spark.createDataFrame(
    data=arrayData, 
    schema=['name','knownLanguages','properties'])

df.printSchema()
df.show()
```
```bash
>>> output Data:
>>>
root
 |-- name: string (nullable = true)
 |-- knownLanguages: array (nullable = true)
 |    |-- element: string (containsNull = true)
 |-- properties: map (nullable = true)
 |    |-- key: string
 |    |-- value: string (valueContainsNull = true)

+----------+--------------+--------------------+
|      name|knownLanguages|          properties|
+----------+--------------+--------------------+
|     James| [Java, Scala]|[eye -> brown, ha...|
|   Michael|[Spark, Java,]|[eye ->, hair -> ...|
|    Robert|    [CSharp, ]|[eye -> , hair ->...|
|Washington|          null|                null|
| Jefferson|        [1, 2]|                  []|
+----------+--------------+--------------------+

```

## 2. `explode`将数组列映射到列

`PySpark` 函数 `explode(e: Column)` 用于分解数组到列。
当一个数组传递给这个函数时，它会创建一个新的默认列`col1`，
它包含所有数组元素。当一个映射被传递时，它会创建两个新列，
一个是键，一个是值，映射中的每个元素都分成行。

`explode`将忽略具有 `null` 或空的元素。从上面的例子中，
`Washington` 和 `Jefferson` 在数组和映射中有空值，
因此下面的代码片段不包含这些行。

### 2.1 列表转换
```python
from pyspark.sql.functions import explode
df2 = df.select(df.name, explode(df.knownLanguages))
df2.printSchema()
df2.show()
```
```bash
>>> output Data:
>>>
root
 |-- name: string (nullable = true)
 |-- col: string (nullable = true)

+---------+------+
|     name|   col|
+---------+------+
|    James|  Java|
|    James| Scala|
|  Michael| Spark|
|  Michael|  Java|
|  Michael|  null|
|   Robert|CSharp|
|   Robert|      |
|Jefferson|     1|
|Jefferson|     2|
+---------+------+

```

### 2.2 字典转换
```python
from pyspark.sql.functions import explode
df3 = df.select(df.name, explode(df.properties))
df3.printSchema()
df3.show()
```
```bash
>>> output Data:
>>>
root
 |-- name: string (nullable = true)
 |-- key: string (nullable = false)
 |-- value: string (nullable = true)

+-------+----+-----+
|   name| key|value|
+-------+----+-----+
|  James| eye|brown|
|  James|hair|black|
|Michael| eye| null|
|Michael|hair|brown|
| Robert| eye|     |
| Robert|hair|  red|
+-------+----+-----+

```

### 2.3 `explode_outer`

`explode_outer(e: Column)` 函数用于为数组或映射列中的每个元素创建一行。

与`explode`不同，如果数组或`map`为空，`explode_outer`返回`null`。
```python
from pyspark.sql.functions import explode_outer

""" with array """
df.select(df.name,explode_outer(df.knownLanguages)).show()

""" with map """
df.select(df.name,explode_outer(df.properties)).show()
```
```bash
>>> output Data:
>>>
+----------+------+
|      name|   col|
+----------+------+
|     James|  Java|
|     James| Scala|
|   Michael| Spark|
|   Michael|  Java|
|   Michael|  null|
|    Robert|CSharp|
|    Robert|      |
|Washington|  null|
| Jefferson|     1|
| Jefferson|     2|
+----------+------+

+----------+----+-----+
|      name| key|value|
+----------+----+-----+
|     James| eye|brown|
|     James|hair|black|
|   Michael| eye| null|
|   Michael|hair|brown|
|    Robert| eye|     |
|    Robert|hair|  red|
|Washington|null| null|
| Jefferson|null| null|
+----------+----+-----+

```

## 3. `poseexplode`

`posexplode(e: Column)` 为数组中的每个元素创建一行，
并创建列`pos`来保存数组元素的位置和列`col`来保存实际的数组值。
当输入列是`map`时，`posexplode` 函数创建列`pos`来保存`map`元素的位置，`key`和`value`列为键值。


注意该函数忽略具有 `null` 的元素。
```python
from pyspark.sql.functions import posexplode

""" with array """
df.select(df.name,posexplode(df.knownLanguages)).show()

""" with map """
df.select(df.name,posexplode(df.properties)).show()
```
```bash
>>> output Data:
>>>
+---------+---+------+
|     name|pos|   col|
+---------+---+------+
|    James|  0|  Java|
|    James|  1| Scala|
|  Michael|  0| Spark|
|  Michael|  1|  Java|
|  Michael|  2|  null|
|   Robert|  0|CSharp|
|   Robert|  1|      |
|Jefferson|  0|     1|
|Jefferson|  1|     2|
+---------+---+------+

+-------+---+----+-----+
|   name|pos| key|value|
+-------+---+----+-----+
|  James|  0| eye|brown|
|  James|  1|hair|black|
|Michael|  0| eye| null|
|Michael|  1|hair|brown|
| Robert|  0| eye|     |
| Robert|  1|hair|  red|
+-------+---+----+-----+

```

## 4. `posexplode_outer`

`posexplode_outer(e: Column)` 返回带有空值的行。
```python
from pyspark.sql.functions import posexplode_outer
""" with array """
df.select("name",posexplode_outer("knownLanguages")).show()

""" with map """
df.select(df.name,posexplode_outer(df.properties)).show()
```
```bash
>>> output Data:
>>>
+----------+----+------+
|      name| pos|   col|
+----------+----+------+
|     James|   0|  Java|
|     James|   1| Scala|
|   Michael|   0| Spark|
|   Michael|   1|  Java|
|   Michael|   2|  null|
|    Robert|   0|CSharp|
|    Robert|   1|      |
|Washington|null|  null|
| Jefferson|   0|     1|
| Jefferson|   1|     2|
+----------+----+------+

+----------+----+----+-----+
|      name| pos| key|value|
+----------+----+----+-----+
|     James|   0| eye|brown|
|     James|   1|hair|black|
|   Michael|   0| eye| null|
|   Michael|   1|hair|brown|
|    Robert|   0| eye|     |
|    Robert|   1|hair|  red|
|Washington|null|null| null|
| Jefferson|null|null| null|
+----------+----+----+-----+

```

## 5. 将嵌套数组 `DataFrame` 列分解为行

创建一个带有嵌套数组列的 `DataFrame`。
```python
arrayArrayData = [
  ("James",[["Java","Scala","C++"],["Spark","Java"]]),
  ("Michael",[["Spark","Java","C++"],["Spark","Java"]]),
  ("Robert",[["CSharp","VB"],["Spark","Python"]])
]

df = spark.createDataFrame(data=arrayArrayData, schema = ['name','subjects'])
df.printSchema()
df.show(truncate=False)
```
```bash
>>> output Data:
>>>
root
 |-- name: string (nullable = true)
 |-- subjects: array (nullable = true)
 |    |-- element: array (containsNull = true)
 |    |    |-- element: string (containsNull = true)

+-------+-----------------------------------+
|name   |subjects                           |
+-------+-----------------------------------+
|James  |[[Java, Scala, C++], [Spark, Java]]|
|Michael|[[Spark, Java, C++], [Spark, Java]]|
|Robert |[[CSharp, VB], [Spark, Python]]    |
+-------+-----------------------------------+

```

### 5.1 展平数组，请使用 `flatten` 函数
```python
from pyspark.sql.functions import flatten
df.select(df.name, flatten(df.subjects)).show(truncate=False)
```
```bash
>>> output Data:
>>>
+-------+-------------------------------+
|name   |flatten(subjects)              |
+-------+-------------------------------+
|James  |[Java, Scala, C++, Spark, Java]|
|Michael|[Spark, Java, C++, Spark, Java]|
|Robert |[CSharp, VB, Spark, Python]    |
+-------+-------------------------------+

```

### 5.2 展平再分解
```python
df.select(df.name, explode(flatten(df.subjects))).show(truncate=False)
```
```bash
>>> output Data:
>>>
+-------+------+
|name   |col   |
+-------+------+
|James  |Java  |
|James  |Scala |
|James  |C++   |
|James  |Spark |
|James  |Java  |
|Michael|Spark |
|Michael|Java  |
|Michael|C++   |
|Michael|Spark |
|Michael|Java  |
|Robert |CSharp|
|Robert |VB    |
|Robert |Spark |
|Robert |Python|
+-------+------+

```

---
