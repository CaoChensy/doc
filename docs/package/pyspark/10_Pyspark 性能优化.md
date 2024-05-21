> Broadcast

```python
from pyspark.sql.functions import broadcast
print(df1.join(broadcast(df2),df2._c77==df1._c77).take(10))
```
---
