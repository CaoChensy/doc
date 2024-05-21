#! https://zhuanlan.zhihu.com/p/450431986

![Image](https://pic4.zhimg.com/80/v2-5db1a82996ec388725185ae900a58008.jpg)

# Union 与 表的逻辑运算（交并补）

## 1. 创建数据
```python
simpleData = [
    ("James","Sales","NY",90000,34,10000),
    ("Michael","Sales","NY",86000,56,20000),
    ("Robert","Sales","CA",81000,30,23000),
    ("Maria","Finance","CA",90000,24,23000)]

columns= ["employee_name", "department", "state", "salary", "age", "bonus"]
df = spark.createDataFrame(data=simpleData, schema=columns)
# df.printSchema()
df.show(truncate=False)

simpleData2 = [
    ("James","Sales","NY",90000,34,10000),
    ("Maria","Finance","CA",90000,24,23000),
    ("Jen","Finance","NY",79000,53,15000),
    ("Jeff","Marketing","CA",80000,25,18000),
    ("Kumar","Marketing","NY",91000,50,21000)
]
columns2= ["employee_name","department","state","salary","age","bonus"]

df2 = spark.createDataFrame(data = simpleData2, schema = columns2)

# df2.printSchema()
df2.show(truncate=False)
```
```bash
>>> output Data:
>>>
+-------------+----------+-----+------+---+-----+
|employee_name|department|state|salary|age|bonus|
+-------------+----------+-----+------+---+-----+
|James        |Sales     |NY   |90000 |34 |10000|
|Michael      |Sales     |NY   |86000 |56 |20000|
|Robert       |Sales     |CA   |81000 |30 |23000|
|Maria        |Finance   |CA   |90000 |24 |23000|
+-------------+----------+-----+------+---+-----+

+-------------+----------+-----+------+---+-----+
|employee_name|department|state|salary|age|bonus|
+-------------+----------+-----+------+---+-----+
|James        |Sales     |NY   |90000 |34 |10000|
|Maria        |Finance   |CA   |90000 |24 |23000|
|Jen          |Finance   |NY   |79000 |53 |15000|
|Jeff         |Marketing |CA   |80000 |25 |18000|
|Kumar        |Marketing |NY   |91000 |50 |21000|
+-------------+----------+-----+------+---+-----+

```

## 2. `Union`

PySpark `union()` 和 `unionAll()` 用于合并两个或多个相同模式或结构的 DataFrame。

- Union 消除了重复项，而 UnionAll 合并了两个包含重复记录的数据集。
- 但是，在 `PySpark` 中两者的行为都相同，并建议使用 `DataFrame duplicate()` 函数来删除重复的行。
```python
unionDF = df.union(df2)
unionDF.show(truncate=False)
```
```bash
>>> output Data:
>>>
+-------------+----------+-----+------+---+-----+
|employee_name|department|state|salary|age|bonus|
+-------------+----------+-----+------+---+-----+
|James        |Sales     |NY   |90000 |34 |10000|
|Michael      |Sales     |NY   |86000 |56 |20000|
|Robert       |Sales     |CA   |81000 |30 |23000|
|Maria        |Finance   |CA   |90000 |24 |23000|
|James        |Sales     |NY   |90000 |34 |10000|
|Maria        |Finance   |CA   |90000 |24 |23000|
|Jen          |Finance   |NY   |79000 |53 |15000|
|Jeff         |Marketing |CA   |80000 |25 |18000|
|Kumar        |Marketing |NY   |91000 |50 |21000|
+-------------+----------+-----+------+---+-----+

```
```python
unionAllDF = df.unionAll(df2)
unionAllDF.show(truncate=False)
```
```bash
>>> output Data:
>>>
+-------------+----------+-----+------+---+-----+
|employee_name|department|state|salary|age|bonus|
+-------------+----------+-----+------+---+-----+
|James        |Sales     |NY   |90000 |34 |10000|
|Michael      |Sales     |NY   |86000 |56 |20000|
|Robert       |Sales     |CA   |81000 |30 |23000|
|Maria        |Finance   |CA   |90000 |24 |23000|
|James        |Sales     |NY   |90000 |34 |10000|
|Maria        |Finance   |CA   |90000 |24 |23000|
|Jen          |Finance   |NY   |79000 |53 |15000|
|Jeff         |Marketing |CA   |80000 |25 |18000|
|Kumar        |Marketing |NY   |91000 |50 |21000|
+-------------+----------+-----+------+---+-----+

```

### 2.1 无重复合并

由于 `union()` 方法返回所有记录行，我们将使用该`distinct()`函数在存在重复时仅返回一条记录。
```python
disDF = df.union(df2).distinct()
disDF.show(truncate=False)
```
```bash
>>> output Data:
>>>
+-------------+----------+-----+------+---+-----+
|employee_name|department|state|salary|age|bonus|
+-------------+----------+-----+------+---+-----+
|James        |Sales     |NY   |90000 |34 |10000|
|Maria        |Finance   |CA   |90000 |24 |23000|
|Kumar        |Marketing |NY   |91000 |50 |21000|
|Michael      |Sales     |NY   |86000 |56 |20000|
|Jen          |Finance   |NY   |79000 |53 |15000|
|Jeff         |Marketing |CA   |80000 |25 |18000|
|Robert       |Sales     |CA   |81000 |30 |23000|
+-------------+----------+-----+------+---+-----+

```

## 3. `unionByName()`



`PySpark` 合并具有不同列的两个`DataFrame`，使用`unionByName()`转换。

首先让我们创建具有不同列数的 `DataFrame`。`unionByName()` 用于按列名而不是按位置合并两个 `DataFrame`。
```python
data = [("James","Sales",34), ("Michael","Sales",56),
    ("Robert","Sales",30), ("Maria","Finance",24) ]
columns= ["name","dept","age"]
df1 = spark.createDataFrame(data = data, schema = columns)
df1.printSchema()

data2=[("James","Sales","NY",9000),("Maria","Finance","CA",9000),
    ("Jen","Finance","NY",7900),("Jeff","Marketing","CA",8000)]
columns2= ["name","dept","state","salary"]
df2 = spark.createDataFrame(data = data2, schema = columns2)
df2.printSchema()
```
```bash
>>> output Data:
>>>
root
 |-- name: string (nullable = true)
 |-- dept: string (nullable = true)
 |-- age: long (nullable = true)

root
 |-- name: string (nullable = true)
 |-- dept: string (nullable = true)
 |-- state: string (nullable = true)
 |-- salary: long (nullable = true)

```
```python
from pyspark.sql.functions import lit
for column in [column for column in df2.columns if column not in df1.columns]:
    df1 = df1.withColumn(column, lit(None))

for column in [column for column in df1.columns if column not in df2.columns]:
    df2 = df2.withColumn(column, lit(None))
```
```python
merged_df = df1.unionByName(df2)
merged_df.show()
```
```bash
>>> output Data:
>>>
+-------+---------+----+-----+------+
|   name|     dept| age|state|salary|
+-------+---------+----+-----+------+
|  James|    Sales|  34| null|  null|
|Michael|    Sales|  56| null|  null|
| Robert|    Sales|  30| null|  null|
|  Maria|  Finance|  24| null|  null|
|  James|    Sales|null|   NY|  9000|
|  Maria|  Finance|null|   CA|  9000|
|    Jen|  Finance|null|   NY|  7900|
|   Jeff|Marketing|null|   CA|  8000|
+-------+---------+----+-----+------+

```

## 4. 交并补
```python
simpleData = [
    ("James","Sales","NY",90000,34,10000),
    ("Michael","Sales","NY",86000,56,20000),
    ("Robert","Sales","CA",81000,30,23000),
    ("Maria","Finance","CA",90000,24,23000)]

columns= ["employee_name", "department", "state", "salary", "age", "bonus"]
df = spark.createDataFrame(data=simpleData, schema=columns)
# df.printSchema()
df.show(truncate=False)

simpleData2 = [
    ("James","Sales","NY",90000,34,10000),
    ("Maria","Finance","CA",90000,24,23000),
    ("Jen","Finance","NY",79000,53,15000),
    ("Jeff","Marketing","CA",80000,25,18000),
    ("Kumar","Marketing","NY",91000,50,21000)
]
columns2= ["employee_name","department","state","salary","age","bonus"]

df2 = spark.createDataFrame(data = simpleData2, schema = columns2)

# df2.printSchema()
df2.show(truncate=False)
```
```bash
>>> output Data:
>>>
+-------------+----------+-----+------+---+-----+
|employee_name|department|state|salary|age|bonus|
+-------------+----------+-----+------+---+-----+
|James        |Sales     |NY   |90000 |34 |10000|
|Michael      |Sales     |NY   |86000 |56 |20000|
|Robert       |Sales     |CA   |81000 |30 |23000|
|Maria        |Finance   |CA   |90000 |24 |23000|
+-------------+----------+-----+------+---+-----+

+-------------+----------+-----+------+---+-----+
|employee_name|department|state|salary|age|bonus|
+-------------+----------+-----+------+---+-----+
|James        |Sales     |NY   |90000 |34 |10000|
|Maria        |Finance   |CA   |90000 |24 |23000|
|Jen          |Finance   |NY   |79000 |53 |15000|
|Jeff         |Marketing |CA   |80000 |25 |18000|
|Kumar        |Marketing |NY   |91000 |50 |21000|
+-------------+----------+-----+------+---+-----+

```

### 4.1 表的交集 `intersect` 
```python
## intersect 取交集，返回一个新的DataFrame
df.intersect(df2).show()
```
```bash
>>> output Data:
>>>
+-------------+----------+-----+------+---+-----+
|employee_name|department|state|salary|age|bonus|
+-------------+----------+-----+------+---+-----+
|        Maria|   Finance|   CA| 90000| 24|23000|
|        James|     Sales|   NY| 90000| 34|10000|
+-------------+----------+-----+------+---+-----+

```

### 4.2 表的差集 `subtract`
```python
## intersect 取交集，返回一个新的DataFrame
df.subtract(df2).show()
```
```bash
>>> output Data:
>>>
+-------------+----------+-----+------+---+-----+
|employee_name|department|state|salary|age|bonus|
+-------------+----------+-----+------+---+-----+
|      Michael|     Sales|   NY| 86000| 56|20000|
|       Robert|     Sales|   CA| 81000| 30|23000|
+-------------+----------+-----+------+---+-----+

```

---
