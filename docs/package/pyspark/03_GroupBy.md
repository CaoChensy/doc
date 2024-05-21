#! https://zhuanlan.zhihu.com/p/442465561
![Image](https://pic4.zhimg.com/80/v2-a66b111438c6742edf1ba06ba24e38d7.png)
# PySpark 的几种分组操作 groupBy rollup cube

```python
from pyspark.sql import functions as F
```

## 1. 创建数据
```python
data = [
    ("James", "Sales", 3000, '2020'),
    ("Michael", "Sales", 4600, '2020'),
    ("Robert", "Sales", 4100, '2020'),
    ("Maria", "Finance", 3000, '2020'),
    ("James", "Sales", 3000, '2019'),
    ("Scott", "Finance", 3300, '2020'),
    ("Jen", "Finance", 3900, '2020'),
    ("Jeff", "Marketing", 3000, '2020'),
    ("Kumar", "Marketing", 2000, '2020'),
    ("Saif", "Sales", 4100, '2020')
]
schema = ["employee_name", "department", "salary", 'year']
  
df = spark.createDataFrame(data=data, schema=schema)
df.printSchema()
df.show(truncate=False)
```
```bash
>>> output Data:
>>>
root
 |-- employee_name: string (nullable = true)
 |-- department: string (nullable = true)
 |-- salary: long (nullable = true)
 |-- year: string (nullable = true)

+-------------+----------+------+----+
|employee_name|department|salary|year|
+-------------+----------+------+----+
|James        |Sales     |3000  |2020|
|Michael      |Sales     |4600  |2020|
|Robert       |Sales     |4100  |2020|
|Maria        |Finance   |3000  |2020|
|James        |Sales     |3000  |2019|
|Scott        |Finance   |3300  |2020|
|Jen          |Finance   |3900  |2020|
|Jeff         |Marketing |3000  |2020|
|Kumar        |Marketing |2000  |2020|
|Saif         |Sales     |4100  |2020|
+-------------+----------+------+----+

```

## 2. `groupBy`

分组聚合统计

- 按照`department`, `year`计算工资之和。
```python
df.groupBy('department', 'year').agg(
    F.sum('salary').alias('salary')
).orderBy('department', 'year').show()
```
```bash
>>> output Data:
>>>
+----------+----+------+
|department|year|salary|
+----------+----+------+
|   Finance|2020| 10200|
| Marketing|2020|  5000|
|     Sales|2019|  3000|
|     Sales|2020| 15800|
+----------+----+------+

```

## 3. `rollup`

1. 先按照 `department`、`employee_name`、`year`分组；
1. 然后按照`department`、`employee_name`分组；
1. 然后再按照 `department` 分组；
1. 最后进行全表分组。
1. 后面接聚合函数，此处使用的是`sum`。

```python
df.rollup('department', 'employee_name', 'year').agg(
    F.sum('salary').alias('salary')
).orderBy('department', 'employee_name', 'year').show()
```
```bash
>>> output Data:
>>>
+----------+-------------+----+------+
|department|employee_name|year|salary|
+----------+-------------+----+------+
|      null|         null|null| 34000|
|   Finance|         null|null| 10200|
|   Finance|          Jen|null|  3900|
|   Finance|          Jen|2020|  3900|
|   Finance|        Maria|null|  3000|
|   Finance|        Maria|2020|  3000|
|   Finance|        Scott|null|  3300|
|   Finance|        Scott|2020|  3300|
| Marketing|         null|null|  5000|
| Marketing|         Jeff|null|  3000|
| Marketing|         Jeff|2020|  3000|
| Marketing|        Kumar|null|  2000|
| Marketing|        Kumar|2020|  2000|
|     Sales|         null|null| 18800|
|     Sales|        James|null|  6000|
|     Sales|        James|2019|  3000|
|     Sales|        James|2020|  3000|
|     Sales|      Michael|null|  4600|
|     Sales|      Michael|2020|  4600|
|     Sales|       Robert|null|  4100|
+----------+-------------+----+------+
only showing top 20 rows

```

## 4. `cube`

1. `cube` 先按照`department、employee_name、year`分组；
1. 然后按照`(department, employee_name)`、`(department, year)`、`(year, employee_name)`分组；
1. 然后按照`department`、`employee_name`、`year`分组；
1. 最后进行全表分组。

```python
df.cube('department', 'employee_name', 'year').agg(
    F.sum('salary').alias('salary')
).orderBy('department', 'employee_name', 'year').show()
```
```bash
>>> output Data:
>>>
+----------+-------------+----+------+
|department|employee_name|year|salary|
+----------+-------------+----+------+
|      null|         null|null| 34000|
|      null|         null|2019|  3000|
|      null|         null|2020| 31000|
|      null|        James|null|  6000|
|      null|        James|2019|  3000|
|      null|        James|2020|  3000|
|      null|         Jeff|null|  3000|
|      null|         Jeff|2020|  3000|
|      null|          Jen|null|  3900|
|      null|          Jen|2020|  3900|
|      null|        Kumar|null|  2000|
|      null|        Kumar|2020|  2000|
|      null|        Maria|null|  3000|
|      null|        Maria|2020|  3000|
|      null|      Michael|null|  4600|
|      null|      Michael|2020|  4600|
|      null|       Robert|null|  4100|
|      null|       Robert|2020|  4100|
|      null|         Saif|null|  4100|
|      null|         Saif|2020|  4100|
+----------+-------------+----+------+
only showing top 20 rows

```

### 4.1 `grouping`

指示 `GROUP BY` 列表中的指定列是否聚合，在结果集中返回 1 表示聚合或 0 表示未聚合。
```python
df.cube("department").agg(
    F.grouping("department").alias('department'), 
    F.sum("salary").alias('salary')
).orderBy("salary").show()
```
```bash
>>> output Data:
>>>
+----------+----------+------+
|department|department|salary|
+----------+----------+------+
| Marketing|         0|  5000|
|   Finance|         0| 10200|
|     Sales|         0| 18800|
|      null|         1| 34000|
+----------+----------+------+

```

### 4.2 `grouping_id`

返回分组级别
```python
df.cube('department', 'employee_name', 'year').agg(
    F.grouping_id().alias('group_level'), 
    F.sum('salary').alias('salary')
).orderBy(F.desc('group_level')).show()
```
```bash
>>> output Data:
>>>
+----------+-------------+----+-----------+------+
|department|employee_name|year|group_level|salary|
+----------+-------------+----+-----------+------+
|      null|         null|null|          7| 34000|
|      null|         null|2020|          6| 31000|
|      null|         null|2019|          6|  3000|
|      null|         Jeff|null|          5|  3000|
|      null|      Michael|null|          5|  4600|
|      null|        James|null|          5|  6000|
|      null|        Kumar|null|          5|  2000|
|      null|        Maria|null|          5|  3000|
|      null|         Saif|null|          5|  4100|
|      null|       Robert|null|          5|  4100|
|      null|        Scott|null|          5|  3300|
|      null|          Jen|null|          5|  3900|
|      null|      Michael|2020|          4|  4600|
|      null|       Robert|2020|          4|  4100|
|      null|        Kumar|2020|          4|  2000|
|      null|        James|2019|          4|  3000|
|      null|         Saif|2020|          4|  4100|
|      null|        James|2020|          4|  3000|
|      null|        Maria|2020|          4|  3000|
|      null|        Scott|2020|          4|  3300|
+----------+-------------+----+-----------+------+
only showing top 20 rows

```

---
