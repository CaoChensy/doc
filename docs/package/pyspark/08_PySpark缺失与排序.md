#! https://zhuanlan.zhihu.com/p/451520636
![Image](https://pic4.zhimg.com/80/v2-5db1a82996ec388725185ae900a58008.jpg)

# PySpark 缺失与排序
```python
import pyspark.sql.functions as F
```

## 1. 缺失

### 1.1 缺失查找

- `isnan` 将非数字数据筛选出来
- `isnull` 将空数据筛选出来

```python
df = spark.createDataFrame(
    [['Alice', 5, 80,],[None, 6, 76,],
        ['Bob', None, 60,],],
    schema=['name', 'age', 'height'])
df.show()
```
```bash
>>> output Data:
>>>
+-----+----+------+
| name| age|height|
+-----+----+------+
|Alice|   5|    80|
| null|   6|    76|
|  Bob|null|    60|
+-----+----+------+

```

### 1.2 删除含有缺失值的行
```python
df.dropna(how='any', thresh=None, subset=None).show()
```
```bash
>>> output Data:
>>>
+-----+---+------+
| name|age|height|
+-----+---+------+
|Alice|  5|    80|
+-----+---+------+

```

### 1.3 将缺失值填充成特定的值
```python
df.fillna(1).show()
```
```bash
>>> output Data:
>>>
+-----+---+------+
| name|age|height|
+-----+---+------+
|Alice|  5|    80|
| null|  6|    76|
|  Bob|  1|    60|
+-----+---+------+

```

### 1.4 按字段填充成特定值
```python
df.na.fill(
    {'age': 50, 'name': 'unknown'}
).show()
```
```bash
>>> output Data:
>>>
+-------+---+------+
|   name|age|height|
+-------+---+------+
|  Alice|  5|    80|
|unknown|  6|    76|
|    Bob| 50|    60|
+-------+---+------+

```

### 1.5 统计缺失值
```python
df_agg = df.agg(
    *[F.count(F.when(F.isnull(c), c)).alias(c) for c in df.columns])
df_agg.show()
```
```bash
>>> output Data:
>>>
+----+---+------+
|name|age|height|
+----+---+------+
|   1|  1|     0|
+----+---+------+

```

### 1.6 统计缺失率
```python
df.agg(
    *[(1 - (F.count(c)/F.count('*'))).alias(c + 'missing') for c in df.columns]
).show()
```
```bash
>>> output Data:
>>>
+-------------------+-------------------+-------------+
|        namemissing|         agemissing|heightmissing|
+-------------------+-------------------+-------------+
|0.33333333333333337|0.33333333333333337|          0.0|
+-------------------+-------------------+-------------+

```

## 2. 排序

### 2.1 升序排列
```python
df.orderBy("name").show()
```
```bash
>>> output Data:
>>>
+-----+----+------+
| name| age|height|
+-----+----+------+
| null|   6|    76|
|Alice|   5|    80|
|  Bob|null|    60|
+-----+----+------+

```

### 2.2 `age`升序，`name`降序排列
```python
df.orderBy(
    ['age', 'name'], ascending=[0, 1]).show()
```
```bash
>>> output Data:
>>>
+-----+----+------+
| name| age|height|
+-----+----+------+
| null|   6|    76|
|Alice|   5|    80|
|  Bob|null|    60|
+-----+----+------+

```
```python
df.orderBy(df["age"].asc()).show()
```
```bash
>>> output Data:
>>>
+-----+----+------+
| name| age|height|
+-----+----+------+
|  Bob|null|    60|
|Alice|   5|    80|
| null|   6|    76|
+-----+----+------+

```
```python
df.orderBy(F.asc('age')).show()              # 升序
df.orderBy(F.desc('age')).show()             # 降序
df.orderBy(F.asc_nulls_last('age')).show()   # 升序Null置于最后
df.orderBy(F.desc_nulls_last('age')).show()  # 降序Null置于最后
df.orderBy(F.asc_nulls_first('age')).show()  # 升序Null置于开始
df.orderBy(F.desc_nulls_first('age')).show() # 降序Null置于开始
```
```bash
>>> output Data:
>>>
+-----+----+------+
| name| age|height|
+-----+----+------+
|  Bob|null|    60|
|Alice|   5|    80|
| null|   6|    76|
+-----+----+------+

+-----+----+------+
| name| age|height|
+-----+----+------+
| null|   6|    76|
|Alice|   5|    80|
|  Bob|null|    60|
+-----+----+------+

+-----+----+------+
| name| age|height|
+-----+----+------+
|Alice|   5|    80|
| null|   6|    76|
|  Bob|null|    60|
+-----+----+------+

+-----+----+------+
| name| age|height|
+-----+----+------+
| null|   6|    76|
|Alice|   5|    80|
|  Bob|null|    60|
+-----+----+------+

+-----+----+------+
| name| age|height|
+-----+----+------+
|  Bob|null|    60|
|Alice|   5|    80|
| null|   6|    76|
+-----+----+------+

+-----+----+------+
| name| age|height|
+-----+----+------+
|  Bob|null|    60|
| null|   6|    76|
|Alice|   5|    80|
+-----+----+------+

```

----
