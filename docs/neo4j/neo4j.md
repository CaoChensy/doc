

## Code

> 空白的环境

```bash
MATCH (n) DETACH DELETE n
```

> CREATE

**创建节点**

```cql
create (n:Person { name: 'Robert Zemeckis', born: 1951 }) return n;
```

`create` (变量:标签 {属性：’属性名称’}) `return` `n`;

- 变量名称可以是:任意，标签注意大小写

**创建节点间关系**

```cql
MATCH (a:Person),(b:Person) 
where a.name = 'm'and b.name = 'Andres' 
CREATE (a)-[r:girl]->(b) 
RETURN r;
```

逻辑是，match定位到a,b两个数据集，然后找出两个中分别name为’m’/’Andres’的人，建立`[r:girl]`的关系。

**创建节点间关系 + 关系属性**

```cql
MATCH (a:Person),(b:Person)
WHERE a.name = 'm'and b.name = 'Andres' 
CREATE (a)-[r:girl { roles:['friend'] }]->(b)
RETURN r;
```

逻辑：从姓名为`m`的人，到姓名为`andres`的人，建立关系`girl`，同时角色属性为`friend`

**创建完整路径path**

```cql
CREATE p = (vic:Worker:Person{ name:'vic',title:"Developer" })-[:WORKS_AT]->(neo)<-[:WORKS_AT]-(michael:Worker:Person { name: 'Michael',title:"Manager" })
RETURN p
```

逻辑为：创建vic这个人与变量(neo)的[:WORKS_AT]关系；
创建michael这个人与变量(neo)的[:WORKS_AT]关系。

**创建唯一性节点 CREATE UNIQUE**

```cql
MATCH (root { name: 'root' }) 
CREATE UNIQUE (root)-[:LOVES]-(someone) 
RETURN someone
```

> REMOVE & Delete

**删除所有节点与关系: delete**

> SKIP

修整CQL查询结果集顶部的结果

```cql

```

> Merge

- MERGE = CREATE + MATCH
    - MERGE命令在图中搜索给定模式，
    - 如果存在，则返回结果
    - 如果它不存在于图中，则它创建新的节点/关系并返回结果


> Return 

```cql
返回一个节点：match (n {name:”B”}) return n;
返回一个关系：match (n {name:”A”})-[r:KNOWS]->(c) return r;
返回一个属性：match (n {name:”A”}) return n.name;
返回所有节点：match p=(a {name:”A”})-[r]->(b) return *;
列别名： match (a {name:”A”}) return a.age as thisisage;
表达式： match (a {name:”A”}) return a.age >30 ,”literal”,(a)–>();
唯一结果：match (a {name:”A”})–>(b) return distinct b;
```

> LIMIT

修剪CQL查询结果集底部的结果

```cql
MATCH (a:Person) 
RETURN a  LIMIT 5
```

> UNION

```cql
MATCH (pp:Person)
RETURN pp.age,pp.name
UNION
MATCH (cc:Customer)
RETURN cc.age,cc.name
```

> WITH

- `with`是将前一个的结果做为下一个的起始或条件来使用

- 过滤聚合结果

```cql
MATCH (david { name: 'David' })--(otherPerson)-->()
WITH otherPerson, count(*) AS foaf
WHERE foaf > 1
RETURN otherPerson.name
```
查询将返回连接到至少有一个外向关系的‘David’的人的姓名。

- 对查询进行排序和限制并返回集合

```cql
MATCH (n)
WITH n
ORDER BY n.name DESC LIMIT 3
RETURN collect(n.name)
```

- 路径搜索的极限分支

```cql
MATCH (n { name: 'Anders' })--(m)
WITH m
ORDER BY m.name DESC LIMIT 1
MATCH (m)--(o)
RETURN o.name
```

> unwind

**拆解collect**

```cql
UNWIND[1,2,3] AS x
RETURN x
```

**collect去重**

```cql
WITH [1,1,2,2] AS coll UNWIND coll AS x
WITH DISTINCT x
RETURN collect(x) AS SET
```

```cql
<!-- 1. 全表扫描 -->
<!-- mysql -->
SELECT p.*
FROM products as p;

<!-- neo4j -->
MATCH (p:Product)
RETURN p;


<!-- 2. 查询价格最贵的10个商品，只返回商品名字和单价 -->
<!-- mysql -->
SELECT p.ProductName, p.UnitPrice
FROM products as p
ORDER BY p.UnitPrice DESC
LIMIT 10;

<!-- neo4j -->
MATCH (p:Product)
RETURN p.productName, p.unitPrice
ORDER BY p.unitPrice DESC
LIMIT 10;


<!-- 3. 按照商品名字筛选 -->
<!-- mysql -->
SELECT p.ProductName, p.UnitPrice
FROM products AS p
WHERE p.ProductName = 'Chocolade';

<!-- neo4j -->
MATCH (p:Product)
WHERE p.productName = "Chocolade"
RETURN p.productName, p.unitPrice;

<!-- 其他的写法 -->
MATCH (p:Product {productName:"Chocolade"})
RETURN p.productName, p.unitPrice;

<!-- 4. 按照商品名字筛选2 -->
<!-- mysql -->
SELECT p.ProductName, p.UnitPrice
FROM products as p
WHERE p.ProductName IN ('Chocolade','Chai');

<!-- neo4j -->
MATCH (p:Product)
WHERE p.productName IN ['Chocolade','Chai']
RETURN p.productName, p.unitPrice;


<!-- 5. 模糊查询和数值过滤 -->
<!-- mysql -->
SELECT p.ProductName, p.UnitPrice
FROM products AS p
WHERE p.ProductName LIKE 'C%' AND p.UnitPrice > 100;

<!-- neo4j -->
MATCH (p:Product)
WHERE p.productName STARTS WITH "C" AND p.unitPrice > 100
RETURN p.productName, p.unitPrice;


<!-- 6. 多表联合查询-->
<!-- mysql -->
SELECT DISTINCT c.CompanyName
FROM customers AS c
JOIN orders AS o ON (c.CustomerID = o.CustomerID)
JOIN order_details AS od ON (o.OrderID = od.OrderID)
JOIN products AS p ON (od.ProductID = p.ProductID)
WHERE p.ProductName = 'Chocolade';

<!-- neo4j -->
MATCH (p:Product {productName:"Chocolade"})<-[:PRODUCT]-(:Order)<-[:PURCHASED]-(c:Customer)
RETURN distinct c.companyName;


<!-- 7.  -->
<!-- mysql -->
SELECT e.EmployeeID, count(*) AS Count
FROM Employee AS e
JOIN Order AS o ON (o.EmployeeID = e.EmployeeID)
GROUP BY e.EmployeeID
ORDER BY Count DESC LIMIT 10;

<!-- neo4j -->
MATCH (:Order)<-[:SOLD]-(e:Employee)
RETURN e.name, count(*) AS cnt
ORDER BY cnt DESC LIMIT 10
```

- [neo4j 的python操作](https://zhuanlan.zhihu.com/p/208401997)
- [安装、部署与基本操作](https://www.bilibili.com/video/BV1Wt4y1178d)
- [neo4j python简单示例](https://zhuanlan.zhihu.com/p/208401997)
- [blog APOC](http://weikeqin.com/2018/04/17/neo4j-apoc-use/)
- [blog algo](https://blog.csdn.net/sinat_36226553/article/details/109124456)
- [Neo4J book](https://github.com/neo4j-graph-analytics)

