> 【Hive】正则表达式匹配/过滤中文汉字

**如果是Hivesql，则如下：**

```sql
select * from table_xxx where info not rlike '[\\u4e00-\\u9fa5]'
```

**如果是在Shell终端调用，则如下：**

```sql
hive -e "select * from table_xxx where info not rlike '[\\\u4e00-\\\u9fa5]'"
```

> [partition](https://blog.csdn.net/qq_25221835/article/details/82762416)

- 语法格式：row_number() over(partition by 分组列 order by 排序列 desc)
- row_number() over()分组排序功能：
- 在使用 row_number() over()函数时候，over()里头的分组以及排序的执行晚于 
where 、group by、order by 的执行。

```sql
row_number() over (partition by [] order by time DESC) as rank
```

> 展示分区命令 

```sql
show partitions {table}
```

> 修改列名

```sql
ALTER TABLE table_name CHANGE pro_name now_name STRING
```

> 修改表名

```sql
ALTER TABLE table_name RENAME TO new_table_name
```
> Hive 表导出到CSV

```python
!hive -e "set hive.cli.print.header=true; SELECT * FROM db.table_name" > "/home/folder/file_name.csv"
```

---

> Reference

- [1] [正则表达式匹配:包含且不包含](https://blog.csdn.net/thewindkee/article/details/52785763)
