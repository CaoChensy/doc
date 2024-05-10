#! https://zhuanlan.zhihu.com/p/459816028
![Image](https://pic4.zhimg.com/80/v2-5db1a82996ec388725185ae900a58008.jpg)

# PySpark 缺失处理
```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window
```

## 1. 将值替换为`Null`

常用于将`PySpark DataFrame` 中的 `0, ''` 替换为 `Null`。

```python
df.select(*[F.when(F.col(col) == 0, None).otherwise(F.col(col)) for col in df.columns])
```

## 2. `forward`，`backward`填充

- `forward fill`: 使用前面的一个值填充后面的值。

- `backward fill`: 使用后面的值填充前面的值。
```python
df = spark.createDataFrame([
    (1, 'd1',None),
    (1, 'd2',10),
    (1, 'd3',None),
    (1, 'd4',30),
    (1, 'd5',None),
    (1, 'd6',None),
],('id', 'day','temperature'))
df.show()
```
```bash
>>> output Data:
>>>
+---+---+-----------+
| id|day|temperature|
+---+---+-----------+
|  1| d1|       null|
|  1| d2|         10|
|  1| d3|       null|
|  1| d4|         30|
|  1| d5|       null|
|  1| d6|       null|
+---+---+-----------+

```
```python
forward = Window.partitionBy('id').orderBy('day').rowsBetween(
    Window.unboundedPreceding, Window.currentRow)
backward = Window.partitionBy('id').orderBy('day').rowsBetween(
    Window.currentRow, Window.unboundedFollowing)

df.withColumn('forward_fill', F.last('temperature', ignorenulls=True).over(forward))\
  .withColumn('backward_fill', F.first('temperature', ignorenulls=True).over(backward))\
.show()
```
```bash
>>> output Data:
>>>
+---+---+-----------+------------+-------------+
| id|day|temperature|forward_fill|backward_fill|
+---+---+-----------+------------+-------------+
|  1| d1|       null|        null|           10|
|  1| d2|         10|          10|           10|
|  1| d3|       null|          10|           30|
|  1| d4|         30|          30|           30|
|  1| d5|       null|          30|         null|
|  1| d6|       null|          30|         null|
+---+---+-----------+------------+-------------+

```

> `rowsBetween`

指定分区排序之后进行操作时，操作的数据范围。
有两个参数，第一个指定操作的起始位置，第二个指定操作的最后位置。
- `Window.unboundedPreceding`:分区的开始位置
- `Window.currentRow`:分区计算到现在的位置
- `Window.unboundedFollowing`:分区的最后位置。
- 负数：表示若前面有元素，范围向前延申几个元素
- 0：表示当前位置，等价于Window.currentRow
- 正数：表示若后面有元素，范围向后延申几个元素

---
