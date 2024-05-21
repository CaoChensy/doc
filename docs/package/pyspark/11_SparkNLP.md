#! https://zhuanlan.zhihu.com/p/455427393


![Image](https://pic4.zhimg.com/80/v2-5db1a82996ec388725185ae900a58008.jpg)

# PySpark 自然语言处理 - 分词
```python
import jieba
import jieba.posseg as pseg
from pyspark.sql import functions as F
from pyspark.sql.types import ArrayType, StringType
```

## 1. 创建数据
```python
df = spark.createDataFrame([
    (0, '是相机也无法承载的美貌啊'),
    (1, '世上有两种女孩最可爱，一种是漂亮；一种是聪慧，而你是聪明的漂亮女孩。'),
    (2, '你简直是人类美学的奇迹'),],
    ["id", "content"])
df.show()
```
```bash
>>> output Data:
>>>
+---+-------------------------------------+
| id|                              content|
+---+-------------------------------------+
|  0|             是相机也无法承载的美貌啊|
|  1|世上有两种女孩最可爱，一种是漂亮；...|
|  2|               你简直是人类美学的奇迹|
+---+-------------------------------------+

```

## 2. 分词`UDF`
```python
@F.udf(returnType=ArrayType(elementType=ArrayType(elementType=StringType())))
def tokenize(content):
    """此udf在每个partiton中加载一次词典，
    :param content: {str} 要分词的内容
    :return: list[word, word, ...]
    """
    words = pseg.cut(content)
    word_flags = list()
    for word, flag in words:
        word_flags.append([flag, word])
    return word_flags
```

## 3. 分词
```python
df = df.withColumn('words', tokenize('content')).persist()
df_explode = df.select('id', F.explode('words').alias('word'))
```

## 4. 分词结果
```python
df_explode.select('id',
    df_explode.word[0].alias('词性'),
    df_explode.word[1].alias('words')).show()
```
```bash
>>> output Data:
>>>
+---+----+-----+
| id|词性|words|
+---+----+-----+
|  0|   v|   是|
|  0|   d| 相机|
|  0|   d|   也|
|  0|   n| 无法|
|  0|   v| 承载|
|  0|  uj|   的|
|  0|  nz| 美貌|
|  0|  zg|   啊|
|  1|   s| 世上|
|  1|   v|   有|
|  1|   m| 两种|
|  1|   n| 女孩|
|  1|   d|   最|
|  1|   v| 可爱|
|  1|   x|   ，|
|  1|   m| 一种|
|  1|   v|   是|
|  1|   a| 漂亮|
|  1|   x|   ；|
|  1|   m| 一种|
+---+----+-----+
only showing top 20 rows

```

---
