#! https://zhuanlan.zhihu.com/p/461211990

![Image](https://pic4.zhimg.com/80/v2-5db1a82996ec388725185ae900a58008.jpg)

# PySpark 逻辑回归

## 1. 逻辑回归简介

`Logistic`回归是一种预测二分类常用算法。`Logistic`回归是广义线性模型的一种特殊情况，可以预测标签的发射概率。


以下示例显示了如何使用弹性网正则化，训练二项式和多项式逻辑回归模型进行二分类任务。

## 2. 逻辑回归示例

### 2.1 准备数据
```python
import pandas as pd
from sklearn import datasets
from pyspark.ml.classification import LogisticRegression
```
```python
# 读取波士顿房价的数据
data = datasets.load_breast_cancer().get('data')
target = datasets.load_breast_cancer().get('target')
feature_names = datasets.load_breast_cancer().get('feature_names')
# 创建 Pyspark DataFrame
df = pd.DataFrame(data, columns=feature_names)
df['target'] = target
df.head()
```
```bash
>>> output Data:
>>>    mean radius  mean texture  mean perimeter  mean area  mean smoothness  \
>>> 0        17.99         10.38          122.80     1001.0          0.11840   
>>> 1        20.57         17.77          132.90     1326.0          0.08474   
>>> 2        19.69         21.25          130.00     1203.0          0.10960   
>>> 3        11.42         20.38           77.58      386.1          0.14250   
>>> 4        20.29         14.34          135.10     1297.0          0.10030   
>>> 
>>>    mean compactness  mean concavity  mean concave points  mean symmetry  \
>>> 0           0.27760          0.3001              0.14710         0.2419   
>>> 1           0.07864          0.0869              0.07017         0.1812   
>>> 2           0.15990          0.1974              0.12790         0.2069   
>>> 3           0.28390          0.2414              0.10520         0.2597   
>>> 4           0.13280          0.1980              0.10430         0.1809   
>>> 
>>>    mean fractal dimension  ...  worst texture  worst perimeter  worst area  \
>>> 0                 0.07871  ...          17.33           184.60      2019.0   
>>> 1                 0.05667  ...          23.41           158.80      1956.0   
>>> 2                 0.05999  ...          25.53           152.50      1709.0   
>>> 3                 0.09744  ...          26.50            98.87       567.7   
>>> 4                 0.05883  ...          16.67           152.20      1575.0   
>>> 
>>>    worst smoothness  worst compactness  worst concavity  worst concave points  \
>>> 0            0.1622             0.6656           0.7119                0.2654   
>>> 1            0.1238             0.1866           0.2416                0.1860   
>>> 2            0.1444             0.4245           0.4504                0.2430   
>>> 3            0.2098             0.8663           0.6869                0.2575   
>>> 4            0.1374             0.2050           0.4000                0.1625   
>>> 
>>>    worst symmetry  worst fractal dimension  target  
>>> 0          0.4601                  0.11890       0  
>>> 1          0.2750                  0.08902       0  
>>> 2          0.3613                  0.08758       0  
>>> 3          0.6638                  0.17300       0  
>>> 4          0.2364                  0.07678       0  
>>> 
>>> [5 rows x 31 columns]
```
```python
# 将数据导入 spark DataFrame
df = spark.createDataFrame(df)
```

### 2.2 数据预处理
```python
from pyspark.ml.linalg import Vector
from pyspark.ml.feature import VectorAssembler
```
```python
# 集合所有特征，放在features列里
vec_assmebler = VectorAssembler(
    inputCols=feature_names.tolist(),
    outputCol='features')

# 对 df 进行合并特征操作
df_features = vec_assmebler.transform(df)
```
```python
# 创建包含输入特征 和 目标变量 target 的数据
df_model = df_features.select('features', 'target')

df_model.show(5)
```
```bash
>>> output Data:
>>>
+--------------------+------+
|            features|target|
+--------------------+------+
|[17.99,10.38,122....|     0|
|[20.57,17.77,132....|     0|
|[19.69,21.25,130....|     0|
|[11.42,20.38,77.5...|     0|
|[20.29,14.34,135....|     0|
+--------------------+------+
only showing top 5 rows

```

### 2.3 切分数据集（训练集、测试集）
```python
df_train, df_test = df_model.randomSplit([0.7, 0.3], seed=0)
```

### 2.4 训练模型
```python
# 这里使用了两个正则化
# 第一个防止过拟合
# 第二个使特征稀疏（L1正则占比0.8）
lr = LogisticRegression(labelCol='target', maxIter=10, regParam=0.3, elasticNetParam=0.8)

# 拟合模型
lrModel = lr.fit(df_train)
```

#### 2.4.1 模型超参数

|参数|描述|
|----|----|
|`regParam`|正则化项系数(默认`0.0`)，正则化项主要用于防止过拟合|
|`elasticNetParam`|正则化范式比(默认0.0)，正则化一般有两种范式`L1(Lasso)`和`L2(Ridge)`。L1一般用于特征的稀疏化，L2一般用于防止过拟合。这里的参数即设置L1范式的占比，默认`0.0`即只使用`L2`范式|
|`maxIter`|最大迭代次数，训练的截止条件，默认`100`次|
|`family`|`binomial`(二分类)/`multinomial`(多分类)/`auto`，默认为auto。设为auto时，会根据数据中实际的标签情况设置是二分类还是多分类|
|`tol`|训练的截止条件，两次迭代之间的改善小于`tol`训练将截止|
|`fitIntercept`|是否拟合截距，默认为`True`|
|`Standardization`|是否使用归一化，这里归一化只针对各维特征的方差进行|
|`Thresholds/setThreshold`|设置多分类、二分类的判决阈值，多分类是一个数组，二分类是double值|
|`AggregationDepth`|设置分布式统计时的层数，主要用在treeAggregate中，数据量越大，可适当加大这个值，默认为2|
|`set*Col`|样本为DataFrame结构，这些是设置列名，方便训练时选取label，weight，feature等列|
|`weightCol`||权重列名。如果不是集合或者为空，我们把所有实例的权重都当作1.0|


|模型拟合后的模型参数|描述|
|----|----|
|`coefficientMatrix`|模型的系数矩阵|
|`coefficients`|logistic回归的模型系数|
|`evaluate(dataset)`|在测试集上评估模型|
|`hasSummary`| 是否有summary|
|`intercept`| 二变量logistic模型的截距|
|`interceptVector`| 多变量logistic模型截距|
|`summary`|获得summary|
|`transform`|(dataset,param=None)|
|`Summary`|拥有的属性|
|`predictions`| 模型transform方法输出的预测数据|
|`probabilityCol`| 给出每个类的概率|
|`areaUnderROC`| 计算AUC|
|`fMeasureByTreshold`| 返回带有两个字段(阈值，F-统计量)的数据框，beta=1.0|
|`pr`| 返回精度-召回率两字段的数据框|
|`precisionByTreshold`|返回带有阈值，精度两字段的数据框，应用了从转换后数据里的所有可能概率作为阈值来计算精度|
|`recallByTreshold`| 返回带有阈值，召回率两字段的数据框，应用了从转换后数据里的所有可能概率作为阈值来计算召回率|
|`roc`|返回带有两字段FPR, TPR的数据框|

### 2.5 模型结果分析

#### 2.5.1 截距与斜率
```python
# 展示截距与斜率
print("Coefficients: " + str(lrModel.coefficients))
print("Intercept: " + str(lrModel.intercept))
```
```bash
>>> output Data:
>>>
Coefficients: (30,[3,6,7,9,23,27],[-0.00014099545495094965,-0.6529983223676307,-2.769011341348917,6.61894935586978,-0.00011269667651682802,-1.5017692627526809])
Intercept: 0.492392916154928
```

#### 2.5.2 训练批次的loss
```python
# 模型迭代的历史记录，记录了每次迭代时的损失（loss）

print("objectiveHistory:")
for objective in lrModel.summary.objectiveHistory:
    print(objective)
```
```bash
>>> output Data:
>>>
objectiveHistory:
0.6669299781827079
0.6634547387535975
0.6622459436817278
0.6599143601680826
0.6526705153820956
0.651748886888337
0.6515424190411283
0.6511972766697709
0.6508960644822539
0.6503327571812717
0.6500814182230683
```

#### 2.5.3 ROC与AUC
```python
lrModel.summary.roc.show()
```
```bash
>>> output Data:
>>>
+---+--------------------+
|FPR|                 TPR|
+---+--------------------+
|0.0|                 0.0|
|0.0|0.012658227848101266|
|0.0| 0.02531645569620253|
|0.0|  0.0379746835443038|
|0.0| 0.05063291139240506|
|0.0| 0.06329113924050633|
|0.0|  0.0759493670886076|
|0.0| 0.08860759493670886|
|0.0| 0.10126582278481013|
|0.0| 0.11392405063291139|
|0.0| 0.12658227848101267|
|0.0| 0.13924050632911392|
|0.0|  0.1518987341772152|
|0.0| 0.16455696202531644|
|0.0| 0.17721518987341772|
|0.0|   0.189873417721519|
|0.0| 0.20253164556962025|
|0.0| 0.21518987341772153|
|0.0| 0.22784810126582278|
|0.0| 0.24050632911392406|
+---+--------------------+
only showing top 20 rows

```
```python
print(f"areaUnderROC: {lrModel.summary.areaUnderROC:.2f}")
```
```bash
>>> output Data:
>>>
areaUnderROC: 0.98
```

##### setThreshold
```python
# 设置模型的阈值达到最大 F score
fMeasure = lrModel.summary.fMeasureByThreshold
maxFMeasure = fMeasure.groupBy().max('F-Measure').select('max(F-Measure)').head()
bestThreshold = fMeasure.where(fMeasure['F-Measure'] == maxFMeasure['max(F-Measure)']) \
    .select('threshold').head()['threshold']
lr.setThreshold(bestThreshold)
```
```bash
>>> output Data:
>>> LogisticRegression_a133eb920df3
```
```python
lr.explainParam('threshold')
```
```bash
>>> output Data:
>>> 'threshold: Threshold in binary classification prediction, in range [0, 1]. If threshold and thresholds are both set, they must match.e.g. if threshold is p, then thresholds must be equal to [1-p, p]. (default: 0.5, current: 0.5785299646147778)'
```

#### 多项回归

```python
# 多项回归
mlr = LogisticRegression(maxIter=10, regParam=0.3, elasticNetParam=0.8, family="multinomial")

# 拟合模型
mlrModel = mlr.fit(training)

# 展示斜率与截距
print("Multinomial coefficients: " + str(mlrModel.coefficientMatrix))
print("Multinomial intercepts: " + str(mlrModel.interceptVector))
```

---
