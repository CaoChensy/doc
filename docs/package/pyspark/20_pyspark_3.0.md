# 即将发布的Apache Spark 3.0中的新Pandas UDF和Python类型提示

Pandas用户定义函数（UDF）是[Apache SparkTM](https://spark.apache.org/)在数据科学方面最重要的增强之一。它们带来了许多好处，例如使用户能够使用[Pandas](https://pandas.pydata.org/)API并提高性能。

然而，Pandas的UDF随着时间的推移而有机地发展，这导致了一些不一致，并在用户中造成了混乱。Apache Spark 3.0的完整版本预计很快就会推出，它将为Pandas UDF引入一个新的接口，该接口利用[Python类型提示](https://www.python.org/dev/peps/pep-0484/)来解决Pandas UDF类型的激增问题，并帮助它们变得更加Pythonic和自我描述。

这篇博客文章介绍了新的Pandas UDF和Python类型提示，以及新的Pandas Function API，包括分组map，map和co-grouped map。

## **Pandas UDF**

Pandas UDF是在Spark 2.3中引入的，请[参阅Pandas UDF for PySpark](https://www.databricks.com/blog/2017/10/30/introducing-vectorized-udfs-for-pyspark.html)。Pandas为数据科学家所熟知，并与许多Python库和包（如[NumPy](https://numpy.org/)，[statsmodel](https://www.statsmodels.org/stable/index.html)和[scikit-learn）](https://scikit-learn.org/stable/)无缝集成，Pandas UDF不仅允许数据科学家扩展其工作负载，还可以利用Apache Spark中的Pandas API。

用户定义的函数由以下人员执行：

- [Apache Arrow](https://arrow.apache.org/)，在JVM和Python驱动程序/执行器之间直接交换数据，几乎零（反）序列化成本。
- 函数内部的Pandas，用于Pandas实例和API。

Pandas UDF与函数内部的Pandas API和Apache Arrow一起工作，用于交换数据。它允许向量化操作，与一次一行的Python UDF相比，可以将性能提高100倍。

下面的示例显示了一个Pandas UDF，只需向每个值添加一个，其中它是用名为`pandas_plus_one`的函数定义的，该函数由指定为`pandas_udf`的Pandas UDF类型的`PandasUDFType.SCALAR`装饰。

```python
from pyspark.sql.functions import pandas_udf, PandasUDFType

@pandas_udf('double', PandasUDFType.SCALAR)
def pandas_plus_one(v):
    # `v` is a pandas Series
    return v.add(1)  # outputs a pandas Series

spark.range(10).select(pandas_plus_one("id")).show()
```

Python函数获取并输出一个Pandas Series。您可以通过使用此函数中丰富的Pandas API集来执行向量化操作，以便为每个值添加一个。序列化（反序列化）也可以通过在引擎盖下利用Apache Arrow自动矢量化。

## **Python类型提示**

Python类型提示是在[PEP 484](https://www.python.org/dev/peps/pep-0484/)和Python 3.5中正式引入的。类型提示是Python中静态指示值类型的官方方法。请参见下面的示例。

```python
def greeting(name: str) -> str:
    return 'Hello ' + name
```

name：`str`表示name参数是str类型，`->`语法表示`greeting()`函数返回一个字符串。

Python类型提示为PySpark和Pandas UDF上下文带来了两个显著的好处。

- 它给出了函数应该做什么的明确定义，使用户更容易理解代码。例如，如果没有类型提示，用户就无法知道`greeting`是否可以接受`None`。它可以避免使用一堆测试用例记录这些微妙的案例和/或让用户自己测试和弄清楚的需要。
- 它可以使执行静态分析更容易。像PyCharm和[Visual Studio Code](https://code.visualstudio.com/)这样的IDE可以利用类型注释来提供代码完成、显示错误，并支持更好的转到定义功能。

## **Pandas UDF类型的激增**

自Apache Spark 2.3发布以来，已经实现了许多新的Pandas UDF，这使得用户很难了解新规范以及如何使用它们。例如，这里有三个输出几乎相同结果的Pandas UDF：

```python
from pyspark.sql.functions import pandas_udf, PandasUDFType                                                                                                                                                             
@pandas_udf('long', PandasUDFType.SCALAR)
def pandas_plus_one(v):
    # `v` is a pandas Series
    return v + 1  # outputs a pandas Series

spark.range(10).select(pandas_plus_one("id")).show()
from pyspark.sql.functions import pandas_udf, PandasUDFType


# New type of Pandas UDF in Spark 3.0.
@pandas_udf('long', PandasUDFType.SCALAR_ITER)
def pandas_plus_one(itr):
    # `iterator` is an iterator of pandas Series.
    return map(lambda v: v + 1, itr)  # outputs an iterator of pandas Series.

spark.range(10).select(pandas_plus_one("id")).show()
from pyspark.sql.functions import pandas_udf, PandasUDFType

@pandas_udf("id long", PandasUDFType.GROUPED_MAP)
def pandas_plus_one(pdf):
# `pdf` is a pandas DataFrame
return pdf + 1  # outputs a pandas DataFrame

# `pandas_plus_one` can _only_ be used with `groupby(...).apply(...)`
spark.range(10).groupby('id').apply(pandas_plus_one).show()
```

尽管这些UDF类型中的每一种都有不同的用途，但有几种可以适用。在这个简单的例子中，你可以使用这三个中的任何一个。然而，每个Pandas UDF都期望不同的输入和输出类型，并以不同的方式工作，具有不同的语义和不同的性能。它让用户搞不清楚该使用和学习哪一个，以及每一个如何工作。

此外，第一种和第二种情况下的`pandas_plus_one`可以在使用常规PySpark列的情况下使用。考虑`withColumn`的参数或具有其他表达式（如`pandas_plus_one("id") + 1`）组合的函数。然而，最后的`pandas_plus_one`只能与`groupby(...).apply(pandas_plus_one)`一起使用。

这种复杂程度引发了Spark开发人员的大量讨论，并推动了通过[官方提案](https://docs.google.com/document/d/1-kV0FS_LF2zvaRh_GhkV32Uqksm_Sq8SvnBBmRyxm30/edit?usp=sharing)引入带有Python类型提示的新Pandas API的努力。我们的目标是让用户能够使用Python类型提示自然地表达他们的pandas UDF，而不会像上面有问题的情况那样产生混乱。例如，上述情况可以写成如下：

```python
def pandas_plus_one(v: pd.Series) -> pd.Series:
    return v + 1
def pandas_plus_one(itr: Iterator[pd.Series]) -> Iterator[pd.Series]:
    return map(lambda v: v + 1, itr)
def pandas_plus_one(pdf: pd.DataFrame) -> pd.DataFrame:
    return pdf + 1
```

##  

## **带有Python类型提示的新Pandas API**

为了解决旧Pandas UDF的复杂性，从Apache Spark 3.0和Python 3.6及更高版本开始，可以使用Python类型提示（如`pandas.Series`、`pandas.DataFrame`、`Tuple`和`Iterator`）来表达新的Pandas UDF类型。

此外，旧的Pandas UDF被分为两个API类别：Pandas UDF和Pandas Function API。虽然它们在内部以类似的方式工作，但存在明显的差异。

您可以像使用其他PySpark列实例一样对待Pandas UDF。但是，您不能将Pandas Function API用于这些列实例。下面是这两个例子：

```python
# Pandas UDF
import pandas as pd
from pyspark.sql.functions import pandas_udf, log2, col

@pandas_udf('long')
def pandas_plus_one(s: pd.Series) -> pd.Series:
    return s + 1

# pandas_plus_one("id") is identically treated as _a SQL expression_ internally.
# Namely, you can combine with other columns, functions and expressions.
spark.range(10).select(
    pandas_plus_one(col("id") - 1) + log2("id") + 1).show()
# Pandas Function API
from typing import Iterator
import pandas as pd


def pandas_plus_one(iterator: Iterator[pd.DataFrame]) -> Iterator[pd.DataFrame]:
    return map(lambda v: v + 1, iterator)

# pandas_plus_one is just a regular Python function, and mapInPandas is
# logically treated as _a separate SQL query plan_ instead of a SQL expression. 
# Therefore, direct interactions with other expressions are impossible.
spark.range(10).mapInPandas(pandas_plus_one, schema="id long").show()
```

另外，请注意，Pandas UDF需要Python类型提示，而Pandas Function API中的类型提示目前是可选的。Pandas Function API计划提供类型提示，将来可能会需要。

### **新Pandas UDF**

新的Pandas UDFs不是手动定义和指定每个Pandas UDF类型，而是从Python函数的给定Python类型提示中推断Pandas UDF类型。Pandas UDF中目前支持的Python类型提示有四种情况：

- 系列到系列
- 系列迭代器到系列迭代器
- 多重级数迭代器到级数迭代器
- 系列到标量（单个值）

在我们深入研究每种情况之前，让我们看看使用新Pandas UDF的三个关键点。

- 虽然Python类型提示在Python世界中通常是可选的，但为了使用新的Pandas UDF，必须为输入和输出指定Python类型提示。
- 用户仍然可以通过手动指定Pandas UDF类型来使用旧方法。但是，鼓励使用Python类型提示。
- 类型提示在所有情况下都应该使用`pandas.Series`。然而，有一个变体，其中`pandas.DataFrame`应该用于其输入或输出类型提示：当输入或输出列为`StructType.`时，请看下面的示例：`将pandas导入为pd 从pyspark. sql. functions导入pandas_udf  df = spark. spark DataFrame（    [[1，"a string"，（"a nested string"，）]]，    "long_col long，string_col string，struct_col struct col1 string"）<> @ pandas_udf（"col1 string，col2 long"） def pandas_plus_len（        s1：pd. Series，s2：pd. Series，pdf：pd. DataFrame）—pd. DataFrame：>    #常规列是系列，结构列是DataFrame。    pdf ['col2']= s1 + s2.str.len（）     return pdf #结构体列需要返回DataFrame df.select`

#### **系列到系列**

Series to Series映射到Apache Spark 2.3中引入的标量Pandas UDF。类型提示可以表示为`pandas.Series, ... -> pandas.Series`。它期望给定的函数接受一个或多个`pandas.Series`，并输出一个`pandas.Series`。输出长度应与输入长度相同。

```python
import pandas as pd
from pyspark.sql.functions import pandas_udf       

@pandas_udf('long')
def pandas_plus_one(s: pd.Series) -> pd.Series:
    return s + 1

spark.range(10).select(pandas_plus_one("id")).show()
```

上面的例子可以用标量Pandas UDF映射到旧的样式，如下所示。

```python
from pyspark.sql.functions import pandas_udf, PandasUDFType                                                          
@pandas_udf('long', PandasUDFType.SCALAR)
def pandas_plus_one(v):
    return v + 1

spark.range(10).select(pandas_plus_one("id")).show()
```

#### **系列迭代器到系列迭代器**

这是Apache Spark 3.0中的一种新型Pandas UDF。它是Series to Series的变体，类型提示可以表示为`Iterator[pd.Series] -> Iterator[pd.Series]`。该函数接受并输出一个迭代器`pandas.Series`。

整个输出的长度必须与整个输入的长度相同。因此，只要整个输入和输出的长度相同，它就可以从输入迭代器中预取数据。给定的函数应该接受一列作为输入。

```python
from typing import Iterator
import pandas as pd
from pyspark.sql.functions import pandas_udf       

@pandas_udf('long')
def pandas_plus_one(iterator: Iterator[pd.Series]) -> Iterator[pd.Series]:
    return map(lambda s: s + 1, iterator)

spark.range(10).select(pandas_plus_one("id")).show()
```

当UDF执行需要昂贵的初始化某些状态时，它也很有用。下面的伪代码说明了这种情况。

```python
@pandas_udf("long")
def calculate(iterator: Iterator[pd.Series]) -> Iterator[pd.Series]:
    # Do some expensive initialization with a state
    state = very_expensive_initialization()
    for x in iterator:
        # Use that state for the whole iterator.
        yield calculate_with_state(x, state)

df.select(calculate("value")).show()
```

Iterator of Series到Iterator of Series也可以映射到旧的Pandas UDF样式。请参见下面的示例。

```python
from pyspark.sql.functions import pandas_udf, PandasUDFType                                                          
@pandas_udf('long', PandasUDFType.SCALAR_ITER)
def pandas_plus_one(iterator):
    return map(lambda s: s + 1, iterator)

spark.range(10).select(pandas_plus_one("id")).show()
```

#### **多重级数迭代器到级数迭代器**

这种类型的Pandas UDF也将在Apache Spark 3.0中引入，以及Iterator of Series to Iterator of Series。类型提示可以表示为`Iterator[Tuple[pandas.Series, ...]] -> Iterator[pandas.Series]`。

它具有与系列迭代器相似的特性和限制。给定的函数接受一个元组`pandas.Series`的迭代器，并输出一个迭代器`pandas.Series`。在使用某些状态和预取输入数据时，它也很有用。整个输出的长度也应该与整个输入的长度相同。但是，给定的函数应该接受多个列作为输入，这与Iterator of Series to Iterator of Series不同。

```python
from typing import Iterator, Tuple
import pandas as pd
from pyspark.sql.functions import pandas_udf       

@pandas_udf("long")
def multiply_two(
        iterator: Iterator[Tuple[pd.Series, pd.Series]]) -> Iterator[pd.Series]:
    return (a * b for a, b in iterator)

spark.range(10).select(multiply_two("id", "id")).show()
```

这也可以映射到旧的Pandas UDF样式，如下所示。

```python
from pyspark.sql.functions import pandas_udf, PandasUDFType                                                               
@pandas_udf('long', PandasUDFType.SCALAR_ITER)
def multiply_two(iterator):
    return (a * b for a, b in iterator)

spark.range(10).select(multiply_two("id", "id")).show()
```

#### **系列到标量**

Series to Scalar映射到Apache Spark 2.4中引入的分组聚合Pandas UDF。类型提示表示为`pandas.Series, ... -> Any`。该函数接受一个或多个panda.Series并输出一个原始数据类型。返回的标量可以是Python原语类型，例如，`int`、`float`或NumPy数据类型（如`numpy.int64`、`numpy.float64`等）。`Any`应相应地理想地为特定标量类型。

```python
import pandas as pd
from pyspark.sql.functions import pandas_udf
from pyspark.sql import Window

df = spark.createDataFrame(
    [(1, 1.0), (1, 2.0), (2, 3.0), (2, 5.0), (2, 10.0)], ("id", "v"))

@pandas_udf("double")
def pandas_mean(v: pd.Series) -> float:
    return v.sum()

df.select(pandas_mean(df['v'])).show()
df.groupby("id").agg(pandas_mean(df['v'])).show()
df.select(pandas_mean(df['v']).over(Window.partitionBy('id'))).show()
```

上面的示例可以转换为具有分组聚合Pandas UDF的示例，如您所见：

```python
import pandas as pd
from pyspark.sql.functions import pandas_udf, PandasUDFType
from pyspark.sql import Window

df = spark.createDataFrame(
    [(1, 1.0), (1, 2.0), (2, 3.0), (2, 5.0), (2, 10.0)], ("id", "v"))

@pandas_udf("double", PandasUDFType.GROUPED_AGG)
def pandas_mean(v):
    return v.sum()

df.select(pandas_mean(df['v'])).show()
df.groupby("id").agg(pandas_mean(df['v'])).show()
df.select(pandas_mean(df['v']).over(Window.partitionBy('id'))).show()
```

### **新的Pandas函数API**

Apache Spark 3.0中的这个新类别使您能够直接应用Python原生函数，该函数针对PySpark DataFrame获取并输出Pandas实例。Apache Spark 3.0中支持的Pandas Functions API包括：grouped map、map和co—grouped map。

请注意，分组映射Pandas UDF现在被归类为组映射Pandas Function API。如前所述，Pandas Function API中的Python类型提示目前是可选的。

#### **分组地图**

Pandas Function API中的Grouped map在分组的DataFrame中是`applyInPandas`，例如，`df.groupby(...)`.这被映射到旧Pandas UDF类型中的分组映射Pandas UDF。它将每个组映射到函数中的每个`pandas.DataFrame`。注意，它不要求输出与输入的长度相同。

```python
import pandas as pd

df = spark.createDataFrame(
    [(1, 1.0), (1, 2.0), (2, 3.0), (2, 5.0), (2, 10.0)], ("id", "v"))

def subtract_mean(pdf: pd.DataFrame) -> pd.DataFrame:
    v = pdf.v
    return pdf.assign(v=v - v.mean())

df.groupby("id").applyInPandas(subtract_mean, schema=df.schema).show()
```

Grouped map type映射到Spark 2.3支持的Grouped map Pandas UDF，如下所示：

```python
import pandas as pd
from pyspark.sql.functions import pandas_udf, PandasUDFType

df = spark.createDataFrame(
    [(1, 1.0), (1, 2.0), (2, 3.0), (2, 5.0), (2, 10.0)], ("id", "v"))

@pandas_udf(df.schema, PandasUDFType.GROUPED_MAP)
def subtract_mean(pdf):
    v = pdf.v
    return pdf.assign(v=v - v.mean())

df.groupby("id").apply(subtract_mean).show()
```

####  

#### **地图**

Map Pandas Function API是DataFrame中的`mapInPandas`。这是Apache Spark 3.0的新特性。它映射每个分区中的每个批次并转换每个批次。该函数接受一个迭代器`pandas.DataFrame`并输出一个迭代器`pandas.DataFrame`。输出长度不需要与输入大小匹配。

```python
from typing import Iterator
import pandas as pd

df = spark.createDataFrame([(1, 21), (2, 30)], ("id", "age"))

def pandas_filter(iterator: Iterator[pd.DataFrame]) -> Iterator[pd.DataFrame]:
    for pdf in iterator:
        yield pdf[pdf.id == 1]

df.mapInPandas(pandas_filter, schema=df.schema).show()
```

#### **联合分组地图**

Co-grouped map，在Co-grouped DataFrame（如`applyInPandas`）中的`df.groupby(...).cogroup(df.groupby(...))`，也将在Apache Spark 3.0中引入。与分组映射类似，它将每个组映射到函数中的每个`pandas.DataFrame`，但它通过公共键与另一个DataFrame分组，然后将函数应用于每个cogroup。同样，对输出长度也没有限制。

```python
import pandas as pd

df1 = spark.createDataFrame(
    [(1201, 1, 1.0), (1201, 2, 2.0), (1202, 1, 3.0), (1202, 2, 4.0)],
    ("time", "id", "v1"))
df2 = spark.createDataFrame(
    [(1201, 1, "x"), (1201, 2, "y")], ("time", "id", "v2"))

def asof_join(left: pd.DataFrame, right: pd.DataFrame) -> pd.DataFrame:
    return pd.merge_asof(left, right, on="time", by="id")

df1.groupby("id").cogroup(
    df2.groupby("id")
).applyInPandas(asof_join, "time int, id int, v1 double, v2 string").show()
```

##  

## **结论和今后的工作**

即将发布的Apache Spark 3.0（[阅读我们的预览博客了解详细信息](https://www.databricks.com/blog/2020/05/13/now-on-databricks-a-technical-preview-of-databricks-runtime-7-including-a-preview-of-apache-spark-3-0.html)）。将提供Python类型提示，使用户更容易表达Pandas UDF和Pandas Function API。将来，我们应该考虑在Pandas UDF和Pandas Function API中添加对其他类型提示组合的支持。目前，支持的情况只是Python类型提示的许多可能组合中的一小部分。Apache Spark社区中还有其他正在进行的讨论。访问[边讨论和未来](https://docs.google.com/document/d/1-kV0FS_LF2zvaRh_GhkV32Uqksm_Sq8SvnBBmRyxm30/edit#heading=h.h3ncjpk6ujqu)改进了解更多信息。

在我们的预览网络研讨会中了解有关Spark 3.0的更多[信息。](https://www.databricks.com/p/webinar/apache-spark-3-0) 作为Databricks 7.0 Beta的一部分，立即[在Databricks上免费](https://www.databricks.com/try-databricks)试用这些新功能。
