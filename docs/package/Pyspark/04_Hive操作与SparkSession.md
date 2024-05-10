#! https://zhuanlan.zhihu.com/p/431956482

![Image](https://pic4.zhimg.com/80/v2-5db1a82996ec388725185ae900a58008.jpg)

# 【PySpark】03 Pyspark 操作 Hive 与 Spark Session

---
> 更新日期：2021-11-11
> [全部PySpark相关文章](https://zhuanlan.zhihu.com/p/431959767)
---

## 0. 补充 `Hive` 的基本操作

```hive
show partitions DbName.TableName
```

## 1. 使用PySpark检查Hive表是否存在

> 在 `spark >= 2.3.0` 中包含 `pyspark.sql.catalog` 模块

```python
spark.catalog.listColumns      # 列出表的全部字段信息
spark.catalog.listDatabases    # 列出Hive下可操作的全部数据库信息
spark.catalog.listTables       # 列出一个数据库全部的表名
```

> 如果您使用的是 `spark < 2.3.0`，则可以如下所示使用：

```python
spark._jsparkSession.catalog().tableExists("schema.table")
spark._jsparkSession.catalog().tableExists("schema.table_false")
spark.catalog._jcatalog.tableExists("schema.table")
```

## 2. `Spark Session`

### 2.1 检查`Spark Session`链接状态

```python
spark._jsc.sc().isStopped()
>>> True 
```

### 2.2 关闭`Spark Session`

```python
spark.stop()
```

### 2.3 `Spark Session`参数信息

```python
# method 1
spark.sparkContext.getConf().getAll()
# method 2
spark.sparkContext._conf.getAll()
# method 3
spark.conf.get()
```

---
## 参考文献

