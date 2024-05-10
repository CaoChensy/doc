#! https://zhuanlan.zhihu.com/p/467120608
![Image](https://pic4.zhimg.com/80/v2-5a5f5ff596b2e02d8da2ebb0a31f47fa.png)

# PySpark 词向量 Word2Vec

`Word2Vec` 训练一个词向量模型 `Map(String, Vector)`，即将一个自然语言转换成数值向量，用于进一步的自然语言处理或机器学习过程。

```python
class pyspark.ml.feature.Word2Vec(
    *, vectorSize=100, minCount=5, numPartitions=1, 
    stepSize=0.025, maxIter=1, seed=None, inputCol=None, 
    outputCol=None, windowSize=5, maxSentenceLength=1000)
```

## 1. 导入数据
```python
import jieba
from pyspark.sql.types import *
from pyspark.sql import functions as F
from pyspark.ml.feature import Word2Vec, Word2VecModel
```
```python
# 读取 kaggle 豆瓣评价数据
df = spark.read.csv(f'C:/Users/chensy/Downloads/archive/DMSC.csv', header=True)
# 剔除缺失数据
df = df.dropna().limit(10)
# 样本记录数
print(f"data length: {df.count() / 1e4:.2f}")
# 样本示例
df.limit(3).toPandas()
```
```bash
>>> output Data:
>>>
data length: 0.00
```
```bash
>>> output Data:
>>>   ID           Movie_Name_EN Movie_Name_CN  Crawl_Date Number Username  \
>>> 0  0  Avengers Age of Ultron        复仇者联盟2  2017-01-22      1       然潘   
>>> 1  1  Avengers Age of Ultron        复仇者联盟2  2017-01-22      2    更深的白色   
>>> 2  2  Avengers Age of Ultron        复仇者联盟2  2017-01-22      3   有意识的贱民   
>>> 
>>>          Date Star                                            Comment  Like  
>>> 0  2015-05-13    3                                      连奥创都知道整容要去韩国。  2404  
>>> 1  2015-04-24    2   非常失望，剧本完全敷衍了事，主线剧情没突破大家可以理解，可所有的人物都缺乏动机，正邪之间、...  1231  
>>> 2  2015-04-26    2   2015年度最失望作品。以为面面俱到，实则画蛇添足；以为主题深刻，实则老调重弹；以为推陈出...  1052  
```

## 2. 分词

这里我们用到了 `jieba` 分词工具，把分词封装为一个 `udf` 对上面的 `DataFrame` 的 `Comment` 进行分词
```python
@F.udf(returnType=ArrayType(StringType()))
def jiebaCut(x):
    return [w for w in jieba.cut(x) if len(w)>1]
```
```python
# 对评论进行分词处理
df = df.withColumn('words', jiebaCut(df['Comment']))
# 数据结构
df.printSchema()
```
```bash
>>> output Data:
>>>
root
 |-- ID: string (nullable = true)
 |-- Movie_Name_EN: string (nullable = true)
 |-- Movie_Name_CN: string (nullable = true)
 |-- Crawl_Date: string (nullable = true)
 |-- Number: string (nullable = true)
 |-- Username: string (nullable = true)
 |-- Date: string (nullable = true)
 |-- Star: string (nullable = true)
 |-- Comment: string (nullable = true)
 |-- Like: string (nullable = true)
 |-- words: array (nullable = true)
 |    |-- element: string (containsNull = true)

```
```python
# 查看分词
df.select('words').limit(6).show()
```
```bash
>>> output Data:
>>>
+-----------------------------+
|                        words|
+-----------------------------+
|     [奥创, 知道, 整容, 韩国]|
| [非常, 失望, 剧本, 完全, ...|
|   [2015, 年度, 失望, 作品...|
| [铁人, 勾引, 钢铁, 妇联, ...|
| [虽然, 从头, 打到, 但是, ...|
|[剧情, 不如, 第一集, 好玩,...|
+-----------------------------+

```

## 3. 训练词向量以及保存与读取

### 3.1 训练词向量

- `minCount`: 表示输入词在输入语料中至少出现多少次，才会进行向量转化，少于该出现次数的次将会在输入值中直接丢弃。
```python
model = Word2Vec(
    vectorSize=8, minCount=2, numPartitions=4, maxIter=16,
    seed=42, inputCol='words', outputCol='vecs').fit(df)
```
```python
# 词向量的数量
print(f'词向量的数量：{model.getVectors().count()}')
# 展示词向量
model.getVectors().show()
```
```bash
>>> output Data:
>>>
词向量的数量：25
+------+--------------------+
|  word|              vector|
+------+--------------------+
|  角色|[0.01273861806839...|
|  剧本|[-0.2398648262023...|
|没什么|[-0.1424400210380...|
|  彩蛋|[-0.0463520660996...|
|  奥创|[-0.0279459767043...|
|  实则|[0.30182662606239...|
|  妇联|[0.08936640620231...|
|  就是|[-0.1287378519773...|
|  看过|[-0.2291299253702...|
|  可以|[-0.3102417588233...|
|  虽然|[-0.1917934864759...|
|  失望|[-0.0342839509248...|
|  只有|[-0.1234440803527...|
|  打斗|[-0.0431661494076...|
|  团结|[-0.2418100088834...|
|  以为|[0.15235844254493...|
|  剧情|[-0.0541103743016...|
|  场面|[-0.0237559825181...|
|  勾引|[0.45644679665565...|
|  没有|[0.20933732390403...|
+------+--------------------+
only showing top 20 rows

```

### 3.2 保存词向量模型

#### 3.2.1 ToFile
```python
#保存模型
model_path = "./temp_data/word2vec"
model.write().overwrite().save(model_path)
```

#### 3.2.2 ToCsv

```python
df.write.csv()
```

#### 3.2.3 ToHive

```python
# 保存向量
df_vec = model.getVectors()
df_vec.write.saveAsTable('db_name.table_name')
```

### 3.3 读取词向量模型
```python
# 读取模型
model_load = Word2VecModel.load(model_path)
# 展示词向量
model_load.getVectors().show()
```
```bash
>>> output Data:
>>>
+------+--------------------+
|  word|              vector|
+------+--------------------+
|  角色|[0.01273861806839...|
|  剧本|[-0.2398648262023...|
|没什么|[-0.1424400210380...|
|  彩蛋|[-0.0463520660996...|
|  奥创|[-0.0279459767043...|
|  实则|[0.30182662606239...|
|  妇联|[0.08936640620231...|
|  就是|[-0.1287378519773...|
|  看过|[-0.2291299253702...|
|  可以|[-0.3102417588233...|
|  虽然|[-0.1917934864759...|
|  失望|[-0.0342839509248...|
|  只有|[-0.1234440803527...|
|  打斗|[-0.0431661494076...|
|  团结|[-0.2418100088834...|
|  以为|[0.15235844254493...|
|  剧情|[-0.0541103743016...|
|  场面|[-0.0237559825181...|
|  勾引|[0.45644679665565...|
|  没有|[0.20933732390403...|
+------+--------------------+
only showing top 20 rows

```

### 3.3 其他`Word2Vec`参数
```python
# 获取或设置最大
word2Vec.setMaxIter(10)
word2Vec.getMaxIter()
```
```python
# 清除参数
word2Vec.clear(word2Vec.maxIter)
word2Vec.getMaxIter()
```
```python
# 设置最小出现次数，设置输出列
model.getMinCount()
model.setInputCol("sentence")
```

## 4. 词向量的应用

### 4.1 词向量映射

#### 4.1.1 transform

对映射后的向量序列直接取了平均
```python
df_trans = model.transform(df)
df_trans.select('ID', 'vecs').show()
```
```bash
>>> output Data:
>>>
+---+--------------------+
| ID|                vecs|
+---+--------------------+
|  0|[-0.0069864941760...|
|  1|[-0.0514680834859...|
|  2|[0.05959105768646...|
|  3|[0.04881695906321...|
|  4|[-0.0299268453381...|
|  5|[-0.0046156922785...|
|  6|[-0.0051825046784...|
|  7|[0.0,0.0,0.0,0.0,...|
|  8|[-0.0096317072483...|
|  9|[-0.0082069622479...|
+---+--------------------+

```

#### 4.1.2 自定义映射
```python
from pyspark.ml.linalg import SparseVector, DenseVector

# 向量长度
vec_length = model.getVectorSize()

df_words = df.select('ID', F.explode('Words').alias('word'))
df_words_cnt = df_words.groupBy('ID').count()
df_vec = model.getVectors()
df_words = df_words.join(df_vec, on='word')
```
```python
@ F.udf(returnType=ArrayType(FloatType()))
def sparse_to_array(v):
    """将 Vector 转化为 array"""
    v = DenseVector(v)
    new_array = list([float(x) for x in v])
    return new_array

df_words = df_words.withColumn('vector_array', sparse_to_array('vector'))
```
```python
df_trans = df_words.groupBy('ID').agg(
    *[F.sum(F.col('vector_array')[i]).alias(f'f_{i}') for i in range(vec_length)],)
df_trans = df_trans.join(df_words_cnt, on='ID')

for i in range(vec_length):
    df_trans = df_trans.withColumn(f'f_{i}', F.col(f'f_{i}') / F.col("count"))

feature_names = [f'f_{i}' for i in range(vec_length)]

from pyspark.ml.feature import VectorAssembler
# 集合所有特征，放在features列里
vec_assmebler = VectorAssembler(
    inputCols=feature_names,
    outputCol='features')

# 对 df 进行合并特征操作
df_features = vec_assmebler.transform(df_trans).select('ID', 'features').orderBy('ID')

# word2vec 指标展示
df_features.show()
```
```bash
>>> output Data:
>>>
+---+--------------------+
| ID|            features|
+---+--------------------+
|  0|[-0.0069864941760...|
|  1|[-0.0514680834859...|
|  2|[0.05959105768646...|
|  3|[0.04881695906321...|
|  4|[-0.0299268453381...|
|  5|[-0.0046156922785...|
|  6|[-0.0051825046784...|
|  8|[-0.0096317072483...|
|  9|[-0.0082069622479...|
+---+--------------------+

```

### 4.2 词向量应用

#### 4.2.1 依据词向量找到相似的词

找到与`word`最相似的`num`个单词。`word` 可以是字符串或向量表示。

返回单词与余弦相似度。
```python
model.findSynonymsArray("失望", 2)
```
```bash
>>> output Data:
>>> [('以为', 0.7477603554725647), ('实则', 0.6977234482765198)]
```
```python
model.findSynonyms("失望", 2).show()
```
```bash
>>> output Data:
>>>
+----+------------------+
|word|        similarity|
+----+------------------+
|以为|0.7477603554725647|
|实则|0.6977234482765198|
+----+------------------+

```

#### 4.2.2 计算全部词的余弦相似度
```python
df_trans = model.transform(df)
df_trans.select('ID', 'vecs')

df_cross = df_trans.select(
    F.col('id').alias('id1'),
    F.col('vecs').alias('vecs1'))\
    .crossJoin(
        df_trans.select(
            F.col('id').alias('id2'),
            F.col('vecs').alias('vecs2')))
df_cross.show()
```
```bash
>>> output Data:
>>>
+---+--------------------+---+--------------------+
|id1|               vecs1|id2|               vecs2|
+---+--------------------+---+--------------------+
|  0|[-0.0069864941760...|  0|[-0.0069864941760...|
|  0|[-0.0069864941760...|  1|[-0.0514680834859...|
|  0|[-0.0069864941760...|  2|[0.05959105768646...|
|  0|[-0.0069864941760...|  3|[0.04881695906321...|
|  0|[-0.0069864941760...|  4|[-0.0299268453381...|
|  0|[-0.0069864941760...|  5|[-0.0046156922785...|
|  0|[-0.0069864941760...|  6|[-0.0051825046784...|
|  0|[-0.0069864941760...|  7|[0.0,0.0,0.0,0.0,...|
|  0|[-0.0069864941760...|  8|[-0.0096317072483...|
|  0|[-0.0069864941760...|  9|[-0.0082069622479...|
|  1|[-0.0514680834859...|  0|[-0.0069864941760...|
|  1|[-0.0514680834859...|  1|[-0.0514680834859...|
|  1|[-0.0514680834859...|  2|[0.05959105768646...|
|  1|[-0.0514680834859...|  3|[0.04881695906321...|
|  1|[-0.0514680834859...|  4|[-0.0299268453381...|
|  1|[-0.0514680834859...|  5|[-0.0046156922785...|
|  1|[-0.0514680834859...|  6|[-0.0051825046784...|
|  1|[-0.0514680834859...|  7|[0.0,0.0,0.0,0.0,...|
|  1|[-0.0514680834859...|  8|[-0.0096317072483...|
|  1|[-0.0514680834859...|  9|[-0.0082069622479...|
+---+--------------------+---+--------------------+
only showing top 20 rows

```
```python
# 用 cosine 构建相似度计算 udf
from scipy import spatial

@F.udf(returnType=FloatType())
def sim(x, y):
    return float(1 - spatial.distance.cosine(x, y))

# 计算两个向量间的相似度 sim
df_cross = df_cross.withColumn('sim', sim(df_cross['vecs1'], df_cross['vecs2']))
df_cross.show()
```
```bash
>>> output Data:
>>>
+---+--------------------+---+--------------------+-----------+
|id1|               vecs1|id2|               vecs2|        sim|
+---+--------------------+---+--------------------+-----------+
|  0|[-0.0069864941760...|  0|[-0.0069864941760...|        1.0|
|  0|[-0.0069864941760...|  1|[-0.0514680834859...| -0.3717363|
|  0|[-0.0069864941760...|  2|[0.05959105768646...|  0.3434972|
|  0|[-0.0069864941760...|  3|[0.04881695906321...| 0.25583833|
|  0|[-0.0069864941760...|  4|[-0.0299268453381...| -0.3790742|
|  0|[-0.0069864941760...|  5|[-0.0046156922785...|0.052120406|
|  0|[-0.0069864941760...|  6|[-0.0051825046784...| -0.3678641|
|  0|[-0.0069864941760...|  7|[0.0,0.0,0.0,0.0,...|        1.0|
|  0|[-0.0069864941760...|  8|[-0.0096317072483...| 0.55050826|
|  0|[-0.0069864941760...|  9|[-0.0082069622479...|-0.32809532|
|  1|[-0.0514680834859...|  0|[-0.0069864941760...| -0.3717363|
|  1|[-0.0514680834859...|  1|[-0.0514680834859...|        1.0|
|  1|[-0.0514680834859...|  2|[0.05959105768646...|-0.91381425|
|  1|[-0.0514680834859...|  3|[0.04881695906321...| -0.6825831|
|  1|[-0.0514680834859...|  4|[-0.0299268453381...|  0.9800788|
|  1|[-0.0514680834859...|  5|[-0.0046156922785...|-0.42568147|
|  1|[-0.0514680834859...|  6|[-0.0051825046784...|  0.8500338|
|  1|[-0.0514680834859...|  7|[0.0,0.0,0.0,0.0,...|        1.0|
|  1|[-0.0514680834859...|  8|[-0.0096317072483...| 0.12655479|
|  1|[-0.0514680834859...|  9|[-0.0082069622479...|  0.7709821|
+---+--------------------+---+--------------------+-----------+
only showing top 20 rows

```

## 5. 停词剔除 `StopWordsRemover`

```python
class pyspark.ml.feature.StopWordsRemover(
    *, inputCol=None, outputCol=None, stopWords=None, 
    caseSensitive=False, locale=None, inputCols=None, outputCols=None)
```

从输入中过滤掉停用词的特征转换器。从 `3.0.0` 开始，
`StopWordsRemover` 可以通过设置 `inputCols` 参数一次过滤掉多个列。
请注意，当同时设置 `inputCol` 和 `inputCols` 参数时，将抛出异常。

### 5.1 停词剔除示例
```python
from pyspark.ml.feature import StopWordsRemover
```
```python
df = spark.createDataFrame([(["a", "b", "c"],)], ["text"])
remover = StopWordsRemover(
    stopWords=["b", 'C'], caseSensitive=False,
    inputCol='text', outputCol='words')

# 剔除停词
remover.transform(df).show()
```
```bash
>>> output Data:
>>>
+---------+-----+
|     text|words|
+---------+-----+
|[a, b, c]|  [a]|
+---------+-----+

```

### 5.2 停词方法参数
```python
# 获取停词参数
display(remover.getStopWords())

# 是否区分大小写，默认不区分大小写
display(remover.getCaseSensitive())
```
```bash
>>> output Data:
>>> ['b', 'C']
```
```bash
>>> output Data:
>>> False
```

---

- [数据来源](https://www.kaggle.com/utmhikari/doubanmovieshortcomments)

---
