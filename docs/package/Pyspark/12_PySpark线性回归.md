#! https://zhuanlan.zhihu.com/p/456874094

![Image](https://pic4.zhimg.com/80/v2-5db1a82996ec388725185ae900a58008.jpg)

# PySpark 线性回归

## 1. 准备数据
```python
import pandas as pd
from sklearn import datasets
from pyspark.sql.functions import corr
from pyspark.ml.regression import LinearRegression
```
```python
# 读取波士顿房价的数据
data = datasets.load_boston().get('data')
target = datasets.load_boston().get('target')
feature_names = datasets.load_boston().get('feature_names')
# 创建 Pyspark DataFrame
df = pd.DataFrame(data, columns=feature_names)
df['target'] = target
df = spark.createDataFrame(df)
```
```python
print((df.count(), len(df.columns)))
df.show(3)
```
```bash
>>> output Data:
>>>
(506, 14)
+-------+----+-----+----+-----+-----+----+------+---+-----+-------+------+-----+------+
|   CRIM|  ZN|INDUS|CHAS|  NOX|   RM| AGE|   DIS|RAD|  TAX|PTRATIO|     B|LSTAT|target|
+-------+----+-----+----+-----+-----+----+------+---+-----+-------+------+-----+------+
|0.00632|18.0| 2.31| 0.0|0.538|6.575|65.2|  4.09|1.0|296.0|   15.3| 396.9| 4.98|  24.0|
|0.02731| 0.0| 7.07| 0.0|0.469|6.421|78.9|4.9671|2.0|242.0|   17.8| 396.9| 9.14|  21.6|
|0.02729| 0.0| 7.07| 0.0|0.469|7.185|61.1|4.9671|2.0|242.0|   17.8|392.83| 4.03|  34.7|
+-------+----+-----+----+-----+-----+----+------+---+-----+-------+------+-----+------+
only showing top 3 rows

```

## 2. 相关性分析

- 计算相关性的示例
```python
df.select(corr('NOX','target')).show()
```
```bash
>>> output Data:
>>>
+-------------------+
|  corr(NOX, target)|
+-------------------+
|-0.4273207723732824|
+-------------------+

```

## 3. 数据预处理
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
|[0.00632,18.0,2.3...|  24.0|
|[0.02731,0.0,7.07...|  21.6|
|[0.02729,0.0,7.07...|  34.7|
|[0.03237,0.0,2.18...|  33.4|
|[0.06905,0.0,2.18...|  36.2|
+--------------------+------+
only showing top 5 rows

```

## 4. 切分数据集（训练集、测试集）
```python
df_train, df_test = df_model.randomSplit([0.7, 0.3], seed=0)
```

## 5. 训练模型
```python
# 创建线性回归模型
lin_Reg = LinearRegression(labelCol='target')

# 在训练集上拟合数据
lr_model = lin_Reg.fit(df_train)
```
```python
# 模型的回归截距与参数
lr_model.intercept, lr_model.coefficients
```
```bash
>>> output Data:
>>> (35.88334690480532,
>>>  DenseVector([-0.0643, 0.04, 0.0029, 2.6079, -19.6509, 4.1625, -0.0004, -1.4713, 0.3194, -0.0137, -0.9707, 0.0083, -0.4721]))
```

### 训练超参数说明

- `featuresCol='features'`        特征列的名称
- `labelCol='label'`              标签列的名称
- `predictionCol='prediction'`    
- `maxIter=100`                   最大训练批次
- `regParam=0.0`                  正则项系数
- `elasticNetParam=0.0`           弹性正则系数
- `tol=1e-06`
- `fitIntercept=True`             拟合截距项
- `standardization=True`     
- `solver='auto'`                 优化方式
- `weightCol=None`
- `aggregationDepth=2`
- `loss='squaredError'`           损失函数
- `epsilon=1.35`

## 6. 训练集评估
```python
training_predictions = lr_model.evaluate(df_train)

training_predictions.meanSquaredError, training_predictions.r2
```
```bash
>>> output Data:
>>> (21.585143449293863, 0.7430405918024237)
```

## 7. 测试集评估
```python
# 对测试集的数据进行预测
test_results=lr_model.evaluate(df_test)

# 展示预测值的残差
test_results.residuals.show(10)

# 测试集的模型评估
test_results.r2, test_results.rootMeanSquaredError, test_results.meanSquaredError
```
```bash
>>> output Data:
>>>
+-------------------+
|          residuals|
+-------------------+
| -5.719690075825394|
|0.31025853627768285|
| 3.4118886696110238|
|-2.2943325047665404|
|-1.3268043339591848|
| 0.8588898105950236|
|  1.698994303382559|
|0.02466049419328442|
| -2.614978934070038|
|  4.288604319922527|
+-------------------+
only showing top 10 rows

```
```bash
>>> output Data:
>>> (0.72683432653438, 4.824656613017763, 23.277311433536035)
```

---
