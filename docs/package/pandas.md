## Pandas 时间操作

```python
# 取出几月份
car_sales.loc[:,'month'] = car_sales['date'].dt.month
car_sales.head()

# 取出来是几号
car_sales.loc[:,'dom'] = car_sales['date'].dt.day

# 取出一年当中的第几天
car_sales.loc[:,'doy'] = car_sales['date'].dt.dayofyear

# 取出星期几
car_sales.loc[:,'dow'] = car_sales['date'].dt.dayofweek

car_sales.head()
```
