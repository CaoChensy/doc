#! https://zhuanlan.zhihu.com/p/459807657
![Image](https://pic4.zhimg.com/80/v2-5db1a82996ec388725185ae900a58008.jpg)

# PySpark 常用指标衍生计算方式

```python
from pyspark.sql.window import Window
from pyspark.sql import functions as F
```

## 1. 多个列求和

```python
columns = ['col1', 'col2', 'col3']
df = df.withColumn('sum_cols', F.expr(' + '.join(columns)))
```

## 2. 同比与环比计算

### 2.1 创建数据
```python
# 创建数据
df = spark.createDataFrame([
        ('20200131', 300),        
        ('20200229', 320),
        ('20190131', 280),
        ('20190228', 400),
        ('20200331', 187)
    ], ['load_date', 'sales'])

# 提取年月日
df = df.withColumn('year', F.year(F.to_date(F.col("load_date"), "yyyMMdd")))\
    .withColumn('month', F.month(F.to_date(F.col("load_date"), "yyyMMdd")))\
    .withColumn('day', F.dayofmonth(F.to_date(F.col("load_date"), "yyyMMdd")))

df.show()
```
```bash
>>> output Data:
>>>
+---------+-----+----+-----+---+
|load_date|sales|year|month|day|
+---------+-----+----+-----+---+
| 20200131|  300|2020|    1| 31|
| 20200229|  320|2020|    2| 29|
| 20190131|  280|2019|    1| 31|
| 20190228|  400|2019|    2| 28|
| 20200331|  187|2020|    3| 31|
+---------+-----+----+-----+---+

```

### 2.2 计算同比环比
```python
wind_qoq = Window.partitionBy('year').orderBy('month') # 环比窗口
wind_yoy = Window.partitionBy('month').orderBy('year') # 同比窗口
```
```python
df = df.withColumn('环比', F.col('sales') / F.lag('sales').over(wind_qoq)-1)\
    .withColumn('同比', F. col('sales')/ F.lag('sales').over(wind_yoy)-1)\
    .orderBy('year','month')
df.show()
```
```bash
>>> output Data:
>>>
+---------+-----+----+-----+---+-------------------+--------------------+
|load_date|sales|year|month|day|               环比|                同比|
+---------+-----+----+-----+---+-------------------+--------------------+
| 20190131|  280|2019|    1| 31|               null|                null|
| 20190228|  400|2019|    2| 28| 0.4285714285714286|                null|
| 20200131|  300|2020|    1| 31|               null|  0.0714285714285714|
| 20200229|  320|2020|    2| 29|0.06666666666666665|-0.19999999999999996|
| 20200331|  187|2020|    3| 31|          -0.415625|                null|
+---------+-----+----+-----+---+-------------------+--------------------+

```

## 3. 同行业地区位置（排名）
```python
wind_zone = Window.partitionBy('load_date', 'zone').orderBy(F.desc(feature_name))     # 地区排名窗口
wind_indu = Window.partitionBy('load_date', 'industry').orderBy(F.desc(feature_name)) # 行业排名窗口
```
```python
df = df.withColumn('rank_zone', F.percent_rank().over(wind_zone))
df = df.withColumn('rank_indu', F.percent_rank().over(wind_indu))
```

---
