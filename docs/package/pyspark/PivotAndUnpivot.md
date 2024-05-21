#! https://zhuanlan.zhihu.com/p/441960268
![Image](https://pic4.zhimg.com/80/v2-5db1a82996ec388725185ae900a58008.jpg)

# 数据透视表`Pivot`与逆透视表`Unpivot`

**数据分析中常用到数据宽表，也会存在数据长表，本节主要尝试数据长表与数据宽表的相互转换，以及阐述这样转换的一些好处。**

`Spark pivot()` 函数用于将数据从一个 `DataFrame` 行旋转为多列，而 `unpivot` 用于将列转换为行。



在本文中，将解释如何使用 `pivot()` 函数将一行或多行转置为列。



`Pivot()` 是一种聚合，其中一个分组列值被转换为具有不同数据的单个列。
```python
from pyspark.sql import functions as F
```

## 1. 创建数据

原始数据是一个数据长表。

DataFrame `df` 包含 3 列 `Product`、`Amount` 和 `Country`，如下所示。
```python
data = (
    ("Banana", 1000, "USA"), 
    ("Carrots", 1500, "USA"), 
    ("Beans", 1600, "USA"),
    ("Orange", 2000, "USA"),
    ("Orange", 2000, "USA"),
    ("Banana", 400, "China"),
    ("Carrots", 1200, "China"),
    ("Beans", 1500, "China"),
    ("Orange", 4000, "China"),
    ("Banana", 2000, "Canada"),
    ("Carrots", 2000, "Canada"),
    ("Beans", 2000, "Mexico"))

df = spark.createDataFrame(data, ["Product", "Amount", "Country"])
df.show()
```
```bash
>>> output Data:
>>>
+-------+------+-------+
|Product|Amount|Country|
+-------+------+-------+
| Banana|  1000|    USA|
|Carrots|  1500|    USA|
|  Beans|  1600|    USA|
| Orange|  2000|    USA|
| Orange|  2000|    USA|
| Banana|   400|  China|
|Carrots|  1200|  China|
|  Beans|  1500|  China|
| Orange|  4000|  China|
| Banana|  2000| Canada|
|Carrots|  2000| Canada|
|  Beans|  2000| Mexico|
+-------+------+-------+

```

## 2. `Pivot` 数据透视表

**现需要将数据长表转换为数据宽表，便于对每个产品的数据进行分析。**



`Spark SQL` 提供`pivot()`将数据从一列旋转到多列（行到列）。
```python
df.groupBy('Country').pivot('Product').agg(
    F.first("Amount")).show()
```
```bash
>>> output Data:
>>>
+-------+------+-----+-------+------+
|Country|Banana|Beans|Carrots|Orange|
+-------+------+-----+-------+------+
|  China|   400| 1500|   1200|  4000|
|    USA|  1000| 1600|   1500|  2000|
| Mexico|  null| 2000|   null|  null|
| Canada|  2000| null|   2000|  null|
+-------+------+-----+-------+------+

```

如果数据不存在，默认情况下它表示为空。

`Spark 2.0` 以后的性能在 `Pivot` 上得到了改进，

但是，如果您使用的是较低版本，出于性能的考虑，

建议提供列数据作为函数的参数，如下所示。
```python
df.groupBy('Country').pivot('Product', ['Banana', 'Beans']).agg(
    F.first("Amount")).show()
```
```bash
>>> output Data:
>>>
+-------+------+-----+
|Country|Banana|Beans|
+-------+------+-----+
|  China|   400| 1500|
|    USA|  1000| 1600|
| Mexico|  null| 2000|
| Canada|  2000| null|
+-------+------+-----+

```

## 3. `Unpivot` 逆透视

**现将数据宽表再转换为数据长表。**



`Unpivo`t 是一个反向操作，我们可以通过将列值旋转为行值。

`Spark SQL` 没有 `unpivot` 功能，因此将使用该`stack()`功能。

下面的代码将列产品转换为行。
```python
df_pivot = df.groupBy('Country').pivot('Product').agg(
    F.first("Amount"))
```

### 3.1 写法一
```python
df_pivot.selectExpr(
    "`Country`", "stack(4, 'Banana', `Banana`, 'Beans', `Beans`, 'Carrots', `Carrots`, 'Orange', `Orange`) as (`Product`,`Amount`)").show()
```
```bash
>>> output Data:
>>>
+-------+-------+------+
|Country|Product|Amount|
+-------+-------+------+
|  China| Banana|   400|
|  China|  Beans|  1500|
|  China|Carrots|  1200|
|  China| Orange|  4000|
|    USA| Banana|  1000|
|    USA|  Beans|  1600|
|    USA|Carrots|  1500|
|    USA| Orange|  2000|
| Mexico| Banana|  null|
| Mexico|  Beans|  2000|
| Mexico|Carrots|  null|
| Mexico| Orange|  null|
| Canada| Banana|  2000|
| Canada|  Beans|  null|
| Canada|Carrots|  2000|
| Canada| Orange|  null|
+-------+-------+------+

```

### 3.2 写法二
```python
df_pivot.select(
    "`Country`", 
    F.expr("stack(4, 'Banana', `Banana`, 'Beans', `Beans`, 'Carrots', `Carrots`, 'Orange', `Orange`) as (`Product`,`Amount`)")).show()
```
```bash
>>> output Data:
>>>
+-------+-------+------+
|Country|Product|Amount|
+-------+-------+------+
|  China| Banana|   400|
|  China|  Beans|  1500|
|  China|Carrots|  1200|
|  China| Orange|  4000|
|    USA| Banana|  1000|
|    USA|  Beans|  1600|
|    USA|Carrots|  1500|
|    USA| Orange|  2000|
| Mexico| Banana|  null|
| Mexico|  Beans|  2000|
| Mexico|Carrots|  null|
| Mexico| Orange|  null|
| Canada| Banana|  2000|
| Canada|  Beans|  null|
| Canada|Carrots|  2000|
| Canada| Orange|  null|
+-------+-------+------+

```

## 4. 将`stack`封装成函数

> 将以上逆透视过程封装成函数，方便后续使用
```python
def unpivot(df, columns, val_type=None, index_name='uuid', feature_name='name', feature_value='value'):
    """
    描述：对数据表进行反pivot操作
    
    :param df[DataFrame]:                 pyspark dataframe
    :param columns[List]:                 需要转换的列
    :param val_type[pyspark.sql.types]:   数据类型
    :param index_name[String]:            index column
    :param feature_name[String]:          特征列
    :param feature_value[String]:         数值列
    """
    if val_type is not None:
        df = df.select(index_name, *[F.col(col).cast(val_type()) for col in columns])
    
    stack_query = []
    for col in columns:
        stack_query.append(f"'{col}', `{col}`")

    df = df.selectExpr(
        f"`{index_name}`", f"stack({len(stack_query)}, {', '.join(stack_query)}) as (`{feature_name}`, `{feature_value}`)"
    ).orderBy(index_name, feature_name)
    return df
```

### 4.1 应用以上函数
```python
from pyspark.sql.types import *

df_unpivot = unpivot(
    df_pivot, columns=['Banana', 'Beans', 'Carrots', 'Orange'], val_type=FloatType,
    index_name='Country', feature_name='Product', feature_value='Amount')
df_unpivot.show()
```
```bash
>>> output Data:
>>>
+-------+-------+------+
|Country|Product|Amount|
+-------+-------+------+
| Canada| Banana|2000.0|
| Canada|  Beans|  null|
| Canada|Carrots|2000.0|
| Canada| Orange|  null|
|  China| Banana| 400.0|
|  China|  Beans|1500.0|
|  China|Carrots|1200.0|
|  China| Orange|4000.0|
| Mexico| Banana|  null|
| Mexico|  Beans|2000.0|
| Mexico|Carrots|  null|
| Mexico| Orange|  null|
|    USA| Banana|1000.0|
|    USA|  Beans|1600.0|
|    USA|Carrots|1500.0|
|    USA| Orange|2000.0|
+-------+-------+------+

```

### 4.2 长表统计分析的好处

**在长表状态下便于进行数据的分组聚合操作。**
```python
df_unpivot.groupBy('Country').agg(
    F.countDistinct('Product').alias('ProductCount'),
    F.mean('Amount').alias('AmountMean'),
    F.max('Amount').alias('AmountMax'),
).show()
```
```bash
>>> output Data:
>>>
+-------+------------+----------+---------+
|Country|ProductCount|AmountMean|AmountMax|
+-------+------------+----------+---------+
|  China|           4|    1775.0|   4000.0|
|    USA|           4|    1525.0|   2000.0|
| Mexico|           4|    2000.0|   2000.0|
| Canada|           4|    2000.0|   2000.0|
+-------+------------+----------+---------+

```

---
