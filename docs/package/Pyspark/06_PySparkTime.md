#! https://zhuanlan.zhihu.com/p/450636026

![Image](https://pic4.zhimg.com/80/v2-5db1a82996ec388725185ae900a58008.jpg)

# PySpark 时间处理

```python
from pyspark.sql import functions as F
```

## 1. 示例数据
```python
data=[["1","2020-02-01"],["2","2019-03-01"],["3","2021-03-01"]]
df=spark.createDataFrame(data, ["id","time"])
df.show()
```
```bash
>>> output Data:
>>>
+---+----------+
| id|      time|
+---+----------+
|  1|2020-02-01|
|  2|2019-03-01|
|  3|2021-03-01|
+---+----------+

```

## 2. 日期

### 2.1 当前日期 `current_date()`

- 获取当前系统日期。默认情况下，数据将以`yyyy-dd-mm`格式返回。
```python
df.select(F.current_date().alias("current_date")).show(1)
```
```bash
>>> output Data:
>>>
+------------+
|current_date|
+------------+
|  2021-12-28|
+------------+
only showing top 1 row
```

### 2.2 日期格式 `date_format()`

- 解析日期并转换`yyyy-dd-mm`为`MM-dd-yyyy`格式。
```python
df.select(F.col("time"), 
    F.date_format(F.col("time"), "MM-dd-yyyy").alias("date_format")).show()
```
```bash
>>> output Data:
>>>
+----------+-----------+
|      time|date_format|
+----------+-----------+
|2020-02-01| 02-01-2020|
|2019-03-01| 03-01-2019|
|2021-03-01| 03-01-2021|
+----------+-----------+
```

### 2.3 使用`to_date()`将日期格式字符串`yyyy-MM-dd`转换为`DateType yyyy-MM-dd`
```python
df.select(F.col("time"), 
    F.to_date(F.col("time"), "yyy-MM-dd").alias("to_date")).show()
```
```bash
>>> output Data:
>>>
+----------+----------+
|      time|   to_date|
+----------+----------+
|2020-02-01|2020-02-01|
|2019-03-01|2019-03-01|
|2021-03-01|2021-03-01|
+----------+----------+

```

### 2.4 两个日期之间的日差`datediff()`
```python
df.select(F.col("time"), 
    F.datediff(F.current_date(), F.col("time")).alias("datediff")  
).show()
```
```bash
>>> output Data:
>>>
+----------+--------+
|      time|datediff|
+----------+--------+
|2020-02-01|     696|
|2019-03-01|    1033|
|2021-03-01|     302|
+----------+--------+

```

### 2.5 两个日期之间的月份`months_between()`
```python
df.select(F.col("time"), 
    F.months_between(F.current_date(),F.col("time")).alias("months_between")  
).show()
```
```bash
>>> output Data:
>>>
+----------+--------------+
|      time|months_between|
+----------+--------------+
|2020-02-01|   22.87096774|
|2019-03-01|   33.87096774|
|2021-03-01|    9.87096774|
+----------+--------------+

```

### 2.6 截断指定单位的日期`trunc()`
```python
df.select(F.col("time"), 
    F.trunc(F.col("time"),"Month").alias("Month_Trunc"), 
    F.trunc(F.col("time"),"Year").alias("Month_Year"), 
    F.trunc(F.col("time"),"Month").alias("Month_Trunc")).show()
```
```bash
>>> output Data:
>>>
+----------+-----------+----------+-----------+
|      time|Month_Trunc|Month_Year|Month_Trunc|
+----------+-----------+----------+-----------+
|2020-02-01| 2020-02-01|2020-01-01| 2020-02-01|
|2019-03-01| 2019-03-01|2019-01-01| 2019-03-01|
|2021-03-01| 2021-03-01|2021-01-01| 2021-03-01|
+----------+-----------+----------+-----------+

```

### 2.7 月、日加减法
```python
df.select(F.col("time"), 
    F.add_months(F.col("time"),3).alias("add_months"), 
    F.add_months(F.col("time"),-3).alias("sub_months"), 
    F.date_add(F.col("time"),4).alias("date_add"), 
    F.date_sub(F.col("time"),4).alias("date_sub") 
).show()
```
```bash
>>> output Data:
>>>
+----------+----------+----------+----------+----------+
|      time|add_months|sub_months|  date_add|  date_sub|
+----------+----------+----------+----------+----------+
|2020-02-01|2020-05-01|2019-11-01|2020-02-05|2020-01-28|
|2019-03-01|2019-06-01|2018-12-01|2019-03-05|2019-02-25|
|2021-03-01|2021-06-01|2020-12-01|2021-03-05|2021-02-25|
+----------+----------+----------+----------+----------+

```

### 2.8 年、月、下一天、一年中第几个星期
```python
df.select(F.col("time"), 
     F.year(F.col("time")).alias("year"), 
     F.month(F.col("time")).alias("month"), 
     F.next_day(F.col("time"),"Sunday").alias("next_day"), 
     F.weekofyear(F.col("time")).alias("weekofyear") 
).show()
```
```bash
>>> output Data:
>>>
+----------+----+-----+----------+----------+
|      time|year|month|  next_day|weekofyear|
+----------+----+-----+----------+----------+
|2020-02-01|2020|    2|2020-02-02|         5|
|2019-03-01|2019|    3|2019-03-03|         9|
|2021-03-01|2021|    3|2021-03-07|         9|
+----------+----+-----+----------+----------+

```

### 2.9 星期几、月日、年日

- 查询星期几
- 一个月中的第几天
- 一年中的第几天
```python
df.select(F.col("time"),  
     F.dayofweek(F.col("time")).alias("dayofweek"), 
     F.dayofmonth(F.col("time")).alias("dayofmonth"), 
     F.dayofyear(F.col("time")).alias("dayofyear"), 
).show()
```
```bash
>>> output Data:
>>>
+----------+---------+----------+---------+
|      time|dayofweek|dayofmonth|dayofyear|
+----------+---------+----------+---------+
|2020-02-01|        7|         1|       32|
|2019-03-01|        6|         1|       60|
|2021-03-01|        2|         1|       60|
+----------+---------+----------+---------+

```

## 3. 时间

### 3.1 创建一个测试数据
```python
data=[
    ["1","02-01-2020 11 01 19 06"],
    ["2","03-01-2019 12 01 19 406"],
    ["3","03-01-2021 12 01 19 406"]]
df2=spark.createDataFrame(data,["id","time"])
df2.show(truncate=False)
```
```bash
>>> output Data:
>>>
+---+-----------------------+
|id |time                   |
+---+-----------------------+
|1  |02-01-2020 11 01 19 06 |
|2  |03-01-2019 12 01 19 406|
|3  |03-01-2021 12 01 19 406|
+---+-----------------------+

```

### 3.2 以 spark 默认格式`yyyy-MM-dd HH:mm:ss`返回当前时间戳
```python
df2.select(F.current_timestamp().alias("current_timestamp")).show()
```
```bash
>>> output Data:
>>>
+--------------------+
|   current_timestamp|
+--------------------+
|2021-12-28 09:31:...|
|2021-12-28 09:31:...|
|2021-12-28 09:31:...|
+--------------------+

```

### 3.3 将字符串时间戳转换为时间戳类型格式 `to_timestamp()`
```python
df2.select(F.col("time"), 
    F.to_timestamp(F.col("time"), "MM-dd-yyyy HH mm ss SSS").alias("to_timestamp") 
    ).show(truncate=False)
```
```bash
>>> output Data:
>>>
+-----------------------+-----------------------+
|time                   |to_timestamp           |
+-----------------------+-----------------------+
|02-01-2020 11 01 19 06 |2020-02-01 11:01:19.06 |
|03-01-2019 12 01 19 406|2019-03-01 12:01:19.406|
|03-01-2021 12 01 19 406|2021-03-01 12:01:19.406|
+-----------------------+-----------------------+

```

### 3.4 获取`小时`、`分钟`、`秒`
```python
# 数据
data=[
    ["1","2020-02-01 11:01:19.06"],
    ["2","2019-03-01 12:01:19.406"],
    ["3","2021-03-01 12:01:19.406"]]
df3=spark.createDataFrame(data,["id","time"])

# 提取小时、分钟、秒
df3.select(
    F.col("time"), 
    F.hour(F.col("time")).alias("hour"), 
    F.minute(F.col("time")).alias("minute"),
    F.second(F.col("time")).alias("second") 
    ).show(truncate=False)
```
```bash
>>> output Data:
>>>
+-----------------------+----+------+------+
|time                   |hour|minute|second|
+-----------------------+----+------+------+
|2020-02-01 11:01:19.06 |11  |1     |19    |
|2019-03-01 12:01:19.406|12  |1     |19    |
|2021-03-01 12:01:19.406|12  |1     |19    |
+-----------------------+----+------+------+

```

---
