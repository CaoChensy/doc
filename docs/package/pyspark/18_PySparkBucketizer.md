#! https://zhuanlan.zhihu.com/p/467668034
![Image](https://pic4.zhimg.com/80/v2-5a5f5ff596b2e02d8da2ebb0a31f47fa.png)

# PySpark 数据分桶 - Bucketizer

# PySpark 数据分桶 - Bucketizer

将一列连续特征映射到一列特征桶。


- 从 `3.0.0` 开始，`Bucketizer` 可以通过设置 `inputCols` 参数一次映射多个列。
请注意，当同时设置 `inputCol` 和 `inputCols` 参数时，将抛出异常。 
- `splits` 参数仅用于单列使用，而 `splitsArray` 用于多列。

```python
class pyspark.ml.feature.Bucketizer(
    *, splits=None, inputCol=None, outputCol=None, handleInvalid='error', 
    splitsArray=None, inputCols=None, outputCols=None)
```

```python
from pyspark.ml.feature import Bucketizer
```
```python
values = [
    (0.1, 0.0), (0.4, 1.0), (1.2, 1.3), (1.4, 1.3), (1.5, float("nan")),
    (float("nan"), 1.0), (float("nan"), 0.0)]
df = spark.createDataFrame(values, ["values1", "values2"])
df.show()
```
```bash
>>> output Data:
>>>
+-------+-------+
|values1|values2|
+-------+-------+
|    0.1|    0.0|
|    0.4|    1.0|
|    1.2|    1.3|
|    1.4|    1.3|
|    1.5|    NaN|
|    NaN|    1.0|
|    NaN|    0.0|
+-------+-------+

```
```python
bucketizer = Bucketizer(
    inputCol='values1',
    outputCol='buckets',
    splits=[-float("inf"), 0.5, 1.4, float("inf")],
    handleInvalid='keep',
)

bucketed = bucketizer.transform(df)
bucketed.show()
```
```bash
>>> output Data:
>>>
+-------+-------+-------+
|values1|values2|buckets|
+-------+-------+-------+
|    0.1|    0.0|    0.0|
|    0.4|    1.0|    0.0|
|    1.2|    1.3|    1.0|
|    1.4|    1.3|    2.0|
|    1.5|    NaN|    2.0|
|    NaN|    1.0|    3.0|
|    NaN|    0.0|    3.0|
+-------+-------+-------+

```
```python
bucketizer = Bucketizer(
    inputCol='values1',
    outputCol='buckets',
    splits=[-float("inf"), 0.4, 1.4, float("inf")],
    handleInvalid='skip',
)

bucketed = bucketizer.transform(df)
bucketed.show()
```
```bash
>>> output Data:
>>>
+-------+-------+-------+
|values1|values2|buckets|
+-------+-------+-------+
|    0.1|    0.0|    0.0|
|    0.4|    1.0|    1.0|
|    1.2|    1.3|    1.0|
|    1.4|    1.3|    2.0|
|    1.5|    NaN|    2.0|
+-------+-------+-------+

```

---
