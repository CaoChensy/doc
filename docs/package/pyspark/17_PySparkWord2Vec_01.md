#! https://zhuanlan.zhihu.com/p/467236417
![Image](https://pic4.zhimg.com/80/v2-5a5f5ff596b2e02d8da2ebb0a31f47fa.png)

# PySpark NLP - 豆瓣评论分类预测

本文，首先训依据`Word2Vec`练不同维度的词向量，使用随机森林模型比较不同维度，不同树的数量，不同深度对于模型效果的影响。
> 预测目标：依据豆瓣影评，预测电影评分[1-5]分。

## 1. 引入数据
```python
import os
import jieba
from tqdm import tqdm_notebook
from pyspark.sql.types import *
from pyspark.sql import functions as F
from pyspark.ml.feature import Word2Vec, Word2VecModel
from pyspark.ml.classification import RandomForestClassifier
from pyspark.ml.evaluation import MulticlassClassificationEvaluator
```
```python
# 分词函数
@F.udf(returnType=ArrayType(StringType()))
def jiebaCut(x):
    return [w for w in jieba.cut(x) if len(w)>1]
```
```python
df = spark.read.csv(f'C:/Users/chensy/Downloads/archive/DMSC.csv', header=True)
# 剔除缺失数据，并且只取1万条数据
df = df.dropna().limit(int(1e4))
df = df.withColumn('words', jiebaCut(df['Comment']))
```

## 2. 计算多个不同维度的词向量
```python
vec_size = [8, 16, 32, 64, 128]

for vectorSize in tqdm_notebook(vec_size):
    model_path = f"./temp_data/word2vec{vectorSize}"
    if not os.path.exists(model_path):
        # 读取 kaggle 豆瓣评价数据
        df = spark.read.csv(f'C:/Users/chensy/Downloads/archive/DMSC.csv', header=True)
        # 剔除缺失数据
        df = df.dropna()
        # 对评论进行分词处理
        df = df.withColumn('words', jiebaCut(df['Comment']))
        model = Word2Vec(
            vectorSize=vectorSize, minCount=2, numPartitions=8, maxIter=32,
            seed=42, inputCol='words', outputCol='vecs').fit(df)
        # 保存模型
        model.write().overwrite().save(model_path)
```

## 3. 选择最优的词向量长度
```python
vec_size = [8, 16, 32, 64, 128]

for vectorSize in tqdm_notebook(vec_size):
    model_path = f"./temp_data/word2vec{vectorSize}"
    # 读取模型
    model = Word2VecModel.load(model_path)
    df_trans = model.transform(df).select('vecs', F.col('Star').cast(IntegerType()))

    rf = RandomForestClassifier(
        featuresCol='vecs', labelCol="Star",
        numTrees=3, maxDepth=8,  seed=42)

    model = rf.fit(df_trans)
    predictions = model.transform(df_trans)

    evaluator = MulticlassClassificationEvaluator(labelCol="Star", predictionCol="prediction")
    accuracy = evaluator.evaluate(predictions)
    print(f"vector size: {vectorSize} accuracy = {accuracy:.4f}")
```

```bash
>>> output Data:
>>>
vector size: 8 accuracy = 0.4660
vector size: 16 accuracy = 0.4970
vector size: 32 accuracy = 0.5068
vector size: 64 accuracy = 0.5096
vector size: 128 accuracy = 0.5100
```

> 精度最高的词向量长度为128，也许词向量维度更多模型的效果也会越好。

## 4. 选择随机森林树的个数
```python
model_path = f"./temp_data/word2vec128"
# 读取模型
model = Word2VecModel.load(model_path)
df_trans = model.transform(df).select('vecs', F.col('Star').cast(IntegerType()))

num_trees = [2, 4, 8, 16]
for numTrees in tqdm_notebook(num_trees):
    rf = RandomForestClassifier(
        featuresCol='vecs', labelCol="Star",
        numTrees=numTrees, maxDepth=8,  seed=42)

    model = rf.fit(df_trans)
    predictions = model.transform(df_trans)

    evaluator = MulticlassClassificationEvaluator(labelCol="Star", predictionCol="prediction")
    accuracy = evaluator.evaluate(predictions)
    print(f"num trees: {numTrees} accuracy = {accuracy:.4f}")
```

```bash
>>> output Data:
>>>
num trees: 2 accuracy = 0.5133
num trees: 4 accuracy = 0.5250
num trees: 8 accuracy = 0.5059
num trees: 16 accuracy = 0.4896
```

> 精度最高的随机森林树个数为4。

## 5. 选择随机森林树的最大深度
```python
model_path = f"./temp_data/word2vec128"
# 读取模型
model = Word2VecModel.load(model_path)
df_trans = model.transform(df).select('vecs', F.col('Star').cast(IntegerType()))

max_depth = [8, 16, 24, 30]
for maxDepth in tqdm_notebook(max_depth):
    rf = RandomForestClassifier(
        featuresCol='vecs', labelCol="Star",
        numTrees=4, maxDepth=maxDepth,  seed=42)

    model = rf.fit(df_trans)
    predictions = model.transform(df_trans)

    evaluator = MulticlassClassificationEvaluator(labelCol="Star", predictionCol="prediction")
    accuracy = evaluator.evaluate(predictions)
    print(f"max depth: {maxDepth} accuracy = {accuracy:.4f}")
```

```bash
>>> output Data:
>>>
max depth: 8 accuracy = 0.5250
max depth: 16 accuracy = 0.8661
max depth: 24 accuracy = 0.8856
max depth: 30 accuracy = 0.8862
```

> 精度最高的随机森林最大深度为30。

```python
# 其他评估
df_eval = predictions.withColumn('eval', F.abs(F.col('Star') - F.col('prediction')))
df_eval.groupBy('eval').count().show()
```
```bash
>>> output Data:
>>>
+----+-----+
|eval|count|
+----+-----+
| 0.0| 8873|
| 1.0|  849|
| 4.0|    5|
| 3.0|   40|
| 2.0|  233|
+----+-----+

```

> 可以看出，在1万个样本中：
1. 完全评估正确的样本共计8873个。
2. 预测差1分的共计849个，
3. 预测差超过1分的共计278个。

---
