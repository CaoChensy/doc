## 单变量分析

> 数据挖洞: 用以显著提高数据的判断能力

- Label: [0, 1, 2]
- `2`: 为`0-1`的过度阶段
- 特征挖掘、指标挖掘、规则挖掘

```mermaid
graph LR
    准入规则-->Pre-A;
    Pre-A-->反欺诈;
    反欺诈-->其他规则;
    其他规则-->A-Card;
    A-Card-->B-Card;
```

> 各步骤是否使用相同的特征？

- 一般不使用相同的特征进行复筛选；可能出现数据偏移。

> ABCDEF --> 拒绝多少的人

> 一般特征工程流程

```mermaid
graph LR
    基础特征-->数据预处理;
    数据预处理-->特征衍生;
    数据预处理-->数值缩放;
    特征衍生-->特征变换;
    特征变换-->特征筛选;
    特征筛选-->单变量筛选;
    特征筛选-->多变量筛选;
    单变量筛选-->卡方检验;
    多变量筛选-->跨时间较差验证筛选变量;
```

> 一般特征性质与检验方法

```mermaid
graph LR
    变量性质-->重要性;
    重要性-->IV-value;
    重要性-->卡方检验;
    重要性-->模型筛选;
    变量性质-->共线性;
    共线性-->相关系数Corr;
    共线性-->方差膨胀系数VIF;
    变量性质-->单调性;
    单调性-->Var-Vs-Target;
    变量性质-->稳定性;
    稳定性-->PSI;
    稳定性-->跨时间较差验证;
```

**对不同字段进行排列组合，看KS**

---

> 业务建模流程

* 将业务抽象为分类or回归问题
* 定义标签，得到y
* 选取合适的样本，并匹配出全部的信息作为特征的来源
* 特征工程 + 模型训练 + 模型评价与调优（相互之间可能会有交互）
* 输出模型报告
* 上线与监控

> 什么是特征？  

在机器学习的背景下，特征是用来解释现象发生的单个特性或一组特性。 
当这些特性转换为某种可度量的形式时，它们被称为特征。

举个例子，假设你有一个学生列表，这个列表里包含每个学生的姓名、
学习小时数、IQ和之前考试的总分数。现在，有一个新学生，你知道他/她的学习小时数和IQ，
但他/她的考试分数缺失，你需要估算他/她可能获得的考试分数。

在这里，需要用IQ和study_hours构建一个估算分数缺失值的预测模型。
所以，IQ和study_hours就成了这个模型的特征。

> 特征工程可能包含的内容

* 基础特征构造  
* 数据预处理  
* 特征衍生  
* 特征变换  
* 特征筛选  

> 数值缩放

```python
# 取对数等变换
import numpy as np
log_age = df_train['Age'].apply(lambda x:np.log(x))
df_train.loc[:,'log_age'] = log_age
df_train.head(10)

# 幅度缩放，最大最小值缩放到[0,1]区间内
from sklearn.preprocessing import MinMaxScaler
mm_scaler = MinMaxScaler()
fare_trans = mm_scaler.fit_transform(df_train[['Fare']])

# 幅度缩放，将每一列的数据标准化为正态分布的
from sklearn.preprocessing import StandardScaler
std_scaler = StandardScaler()
fare_std_trans = std_scaler.fit_transform(df_train[['Fare']])

# 中位数或者四分位数去中心化数据，对异常值不敏感
from sklearn.preprocessing import robust_scale
fare_robust_trans = robust_scale(df_train[['Fare','Age']])

# 将同一行数据规范化,前面的同一变为1以内也可以达到这样的效果
from sklearn.preprocessing import Normalizer
normalizer = Normalizer()
fare_normal_trans = normalizer.fit_transform(df_train[['Age','Fare']])
fare_normal_trans
```

> 特征衍生

```python
from sklearn.preprocessing import PolynomialFeatures
poly = PolynomialFeatures(degree=2)
df_train[['SibSp','Parch']].head()

poly_fea = poly.fit_transform(df_train[['SibSp','Parch']])
poly_fea
```

## 特征选择 (feature_selection) 

> Filter  

1. 移除低方差的特征 (Removing features with low variance)
2. 单变量特征选择 (Univariate feature selection)  

> Wrapper  

3. 递归特征消除 (Recursive Feature Elimination)  

> Embedded  

4. 使用SelectFromModel选择特征 (Feature selection using SelectFromModel)
5. 将特征选择过程融入pipeline (Feature selection as part of a pipeline)

当数据预处理完成后，我们需要选择有意义的特征输入机器学习的算法和模型进行训练。  

通常来说，从两个方面考虑来选择特征：

> 特征是否发散

如果一个特征不发散，例如方差接近于0，也就是说样本在这个特征上基本上没有差异，
这个特征对于样本的区分并没有什么用。

> 特征与目标的相关性

与目标相关性高的特征，应当优选选择。
除移除低方差法外，本文介绍的其他方法均从相关性考虑。

根据特征选择的形式又可以将特征选择方法分为3种：

- Filter：过滤法，按照发散性或者相关性对各个特征进行评分，设定阈值或者待选择阈值的个数，选择特征。
- Wrapper：包装法，根据目标函数（通常是预测效果评分），每次选择若干特征，或者排除若干特征。
- Embedded：嵌入法，先使用某些机器学习的算法和模型进行训练，得到各个特征的权值系数，
根据系数从大到小选择特征。类似于Filter方法，但是是通过训练来确定特征的优劣。  


特征选择主要有两个目的：

- 减少特征数量、降维，使模型泛化能力更强，减少过拟合；  

- 增强对特征和特征值之间的理解。

拿到数据集，一个特征选择方法，往往很难同时完成这两个目的。
通常情况下，选择一种自己最熟悉或者最方便的特征选择方法
（往往目的是降维，而忽略了对特征和数据理解的目的）。
接下来将结合 Scikit-learn提供的例子 介绍几种常用的特征选择方法，
它们各自的优缺点和问题。

> Filter

1. 移除低方差的特征 (Removing features with low variance) 

假设某特征的特征值只有`0`和`1`，并且在所有输入样本中，95%的实例的该特征取值都是1，
那就可以认为这个特征作用不大。如果100%都是1，那这个特征就没意义了。
当特征值都是离散型变量的时候这种方法才能用，如果是连续型变量，
就需要将连续变量离散化之后才能用。而且实际当中，
一般不太会有95%以上都取某个值的特征存在，所以这种方法虽然简单但是不太好用。
可以把它作为特征选择的预处理，先去掉那些取值变化小的特征，
然后再从接下来提到的的特征选择方法中选择合适的进行进一步的特征选择。

```python
from sklearn.feature_selection import VarianceThreshold
X = [[0, 0, 1], [0, 1, 0], [1, 0, 0], [0, 1, 1], [0, 1, 0], [0, 1, 1]]
sel = VarianceThreshold(threshold=(.8 * (1 - .8)))
sel.fit_transform(X)
```

`VarianceThreshold` 移除了第一列特征，第一列中特征值为`0`的概率达到了`5/6`.

2. 单变量特征选择 (Univariate feature selection)

单变量特征选择的原理是分别单独的计算每个变量的某个统计指标，
根据该指标来判断哪些变量重要，剔除那些不重要的变量。  

对于分类问题(y离散)，可采用：
- 卡方检验
- f_classif 
- mutual_info_classif
- 互信息  

对于回归问题(y连续)，可采用：
- 皮尔森相关系数
- f_regression,
- mutual_info_regression
- 最大信息系数

这种方法比较简单，易于运行，易于理解，通常对于理解数据有较好的效果
（但对特征优化、提高泛化能力来说不一定有效）。  

- SelectKBest 移除得分前 k 名以外的所有特征(取top k)
- SelectPercentile 移除得分在用户指定百分比以后的特征(取top k%)
- 对每个特征使用通用的单变量统计检验： 假正率(false positive rate) SelectFpr, 
伪发现率(false discovery rate) SelectFdr, 或族系误差率 SelectFwe.
- GenericUnivariateSelect 可以设置不同的策略来进行单变量特征选择。
同时不同的选择策略也能够使用超参数寻优，从而让我们找到最佳的单变量特征选择策略。

Notice:  
　　The methods based on F-test estimate the degree of linear dependency 
between two random variables. (F检验用于评估两个随机变量的线性相关性)
On the other hand, mutual information methods can capture any kind of 
statistical dependency, but being nonparametric, they require more samples 
for accurate estimation.(另一方面，互信息的方法可以捕获任何类型的统计依赖关系，
但是作为一个非参数方法，估计准确需要更多的样本)

> 卡方(`Chi2`)检验  

经典的卡方检验是检验定性自变量对定性因变量的相关性。比如，
我们可以对样本进行一次`chi2` 测试来选择最佳的两项特征：

```python
from sklearn.datasets import load_iris
from sklearn.feature_selection import SelectKBest
from sklearn.feature_selection import chi2
iris = load_iris()
X, y = iris.data, iris.target
X.shape

X_new = SelectKBest(chi2, k=2).fit_transform(X, y)
X_new.shape
```

> Pearson相关系数 (Pearson Correlation)

皮尔森相关系数是一种最简单的，能帮助理解特征和响应变量之间关系的方法，
该方法衡量的是变量之间的线性相关性，结果的取值区间为[-1，1]，-1表示完全的负相关，
+1表示完全的正相关，0表示没有线性相关。

> Wrapper

> 递归特征消除 (Recursive Feature Elimination)

递归消除特征法使用一个基模型来进行多轮训练，
每轮训练后，移除若干权值系数的特征，再基于新的特征集进行下一轮训练。  

对特征含有权重的预测模型(例如，线性模型对应参数coefficients)，
RFE通过递归减少考察的特征集规模来选择特征。
首先，预测模型在原始特征上训练，每个特征指定一个权重。
之后，那些拥有最小绝对值权重的特征被踢出特征集。
如此往复递归，直至剩余的特征数量达到所需的特征数量。  

RFECV 通过交叉验证的方式执行RFE，以此来选择最佳数量的特征：
对于一个数量为d的feature的集合，他的所有的子集的个数是2的d次方减1(包含空集)。
指定一个外部的学习算法，比如SVM之类的。
通过该算法计算所有子集的validation error。
选择error最小的那个子集作为所挑选的特征。

```python
from sklearn.feature_selection import RFE
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import load_iris

rf = RandomForestClassifier()
iris=load_iris()
X,y=iris.data,iris.target
rfe = RFE(estimator=rf, n_features_to_select=3)
X_rfe = rfe.fit_transform(X,y)
X_rfe.shape
```

> Embedded

使用SelectFromModel选择特征 (Feature selection using SelectFromModel)

> 基于L1的特征选择 (L1-based feature selection)  

- L1 - Laplace Distribution
- L2 - Gaussian Distribution

使用L1范数作为惩罚项的线性模型(Linear models)会得到稀疏解：
大部分特征对应的系数为0。当你希望减少特征的维度以用于其它分类器时，
可以通过 feature_selection.SelectFromModel 来选择不为0的系数。  

特别指出，常用于此目的的稀疏预测模型有 linear_model.Lasso（回归），
linear_model.LogisticRegression 和 svm.LinearSVC（分类）

```python
from sklearn.feature_selection import SelectFromModel
from sklearn.svm import LinearSVC
lsvc = LinearSVC(C=0.01, penalty="l1", dual=False).fit(X,y)
model = SelectFromModel(lsvc, prefit=True)
X_embed = model.transform(X)
X_embed.shape
```

> 那么工作中我们更倾向于使用什么方法呢？

业务中的模型会遇到什么问题。

- 模型效果不好
- 训练集效果好，跨时间测试效果不好: 训练集、测试集、验证集，使用验证集筛选变量
- 跨时间测试效果也好，上线之后效果不好: 线上线下变量一致性，
未来变量，最终将不同的数据集融合模拟，再上线
- 上线之后效果还好，几周之后分数分布开始下滑
- 一两个月内都比较稳定，突然分数分布骤降
- 没有明显问题，但模型每个月逐步失效

业务所需要的变量是什么。

- 变量必须对模型有贡献，也就是说必须能对客群加以区分
- 逻辑回归要求变量之间线性无关
- 逻辑回归评分卡也希望变量呈现单调趋势 
（有一部分也是业务原因，但从模型角度来看，单调变量未必一定比有转折的变量好）
- 客群在每个变量上的分布稳定，分布迁移无可避免，但不能波动太大  

---

> 变量重要性

- IV值
- 卡方检验
- 模型筛选

这里我们使用IV值或者模型筛选多一点

IV其实就是在WOE前面加上一项。
- $p_{y_i}=\frac{y_i}{y_T}$  
- $p_{n_i}=\frac{n_i}{n_T}$  
- $woe_i = ln(\frac{p_{y_i}}{p_{n_i}})$  
- $iv_i = (p_{y_i} - p_{n_i}) \times woe_i$    

最后只需要将每一个区间的iv加起来就得到总的iv值：
$$IV = \sum iv_i$$

> 共线性

- 相关系数 COR
- 方差膨胀系数 VIF  

在做很多基于空间划分思想的模型的时候，我们必须关注变量之间的相关性。
单独看两个变量的时候我们会使用皮尔逊相关系数。

在多元回归中，我们可以通过计算方差膨胀系数VIF来检验回归模型是否存在严重的多重共线性问题。定义：

$$VIF = \frac{1}{1-R^2}$$

其中，$R_i$为自变量  对其余自变量作回归分析的负相关系数。方差膨胀系数是容忍度$1-R^2$的倒数。  

方差膨胀系数VIF越大，说明自变量之间存在共线性的可能性越大。一般来讲，如果方差膨胀因子超过10，
则回归模型存在严重的多重共线性。又根据Hair(1995)的共线性诊断标准，当自变量的容忍度大于0.1，
方差膨胀系数小于10的范围是可以接受的，表明白变量之间没有共线性问题存在。

> 单调性

- bivar图

4）稳定性

- PSI
- 跨时间交叉检验

> 跨时间交叉检验

就是将样本按照月份切割，一次作为训练集和测试集来训练模型，取进入模型的变量之间的交集，
但是要小心共线特征！

解决方法  

- 不需要每次都进入模型，大部分都在即可
- 先去除共线性（这也是为什么集成模型我们也会去除共线性）

> 群体稳定性指标(population stability index)

公式： 
  
$$ PSI = \sum{(实际占比-预期占比)*{\ln(\frac{实际占比}{预期占比})}}$$

来自知乎的例子：  
比如训练一个logistic回归模型，预测时候会有个概率输出p。  
你测试集上的输出设定为p1吧，将它从小到大排序后10等分，如0-0.1,0.1-0.2,......。  
现在你用这个模型去对新的样本进行预测，预测结果叫p2,按p1的区间也划分为10等分。  
实际占比就是p2上在各区间的用户占比，预期占比就是p1上各区间的用户占比。  
意义就是如果模型跟稳定，那么p1和p2上各区间的用户应该是相近的，占比不会变动很大，
也就是预测出来的概率不会差距很大。  
一般认为psi小于0.1时候模型稳定性很高，0.1-0.25一般，大于0.25模型稳定性差，建议重做。  

注意分箱的数量将会影响着变量的PSI值。

PSI并不只可以对模型来求，对变量来求也一样。只需要对跨时间分箱的数据分别求PSI即可。

