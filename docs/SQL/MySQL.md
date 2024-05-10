[toc]

# MYSQL

### mysql-注意事项

> 1.启动MySql服务，开始-运行：services.msc  找到MySql服务启动
>
> 2.mysql语句一般不区分大小写，默认排序方式是字典型（升序），A，a默认排列在B，b之前
>
> ​	ASC表示升序排列（默认值）,DESC表示降序排列
>
> 3.sql没有boolean类型，比较返回结果是1（true）或0（false）
>
> 4.查询null是时:
>
> ```
> where is null		where not is null
> ```
>
> 5.between边界问题，一般情况MySQL包含边界取值，包括data（）函数的时间区间，及理解为<=和  >=
>
> ```
> WHERE column BETWEEN value1 AND value2
> WHERE column NOT BETWEEN value1 AND value2
> 
> select a BETWEEN m and n
> #如果a处于m到n范围内（包括m和n）本身，则返回1，否则返回0
> ```
>
> 6.mysql不能修改数据库名
>
> 7.枚举：将选项全列出来，数据只能从选择中选择一个，如果添加枚举会对全表重构 
>
> 8.where 查询条件，如果查询条件是类型错误的（char类型写成int类型），会返回空，不会报错
>
> 9.**==msql中如果   int+null  返回null==**，如果int类型的数据加上null，则结果返回null。反之也一样
>
> 10.sql的两种使用方式：
>
> （1）在终端交互方式下使用，称为交互式SQL；
> （2）嵌入在高级语言的程序中使用，称为嵌入式SQL，而这些高级语言可以是C、PASCAL、COBOL等，称为宿主语言。

### MySQL-查询执行过程

> 客户端向MySQL服务器发送一条查询请求
>
> 服务器首先检查查询缓存，如果命中缓存，则立刻返回存储在缓存中的结果。否则进入下一阶段(缓存不一定开启)
>
> 服务器进行SQL解析、预处理、再由优化器生成对应的执行计划
>
> MySQL根据执行计划，调用存储引擎的API来执行查询
>
> 将结果返回给客户端，同时缓存查询结果

### MySQL-查询优化

> 设计数据库时：数据库表、字段的设计，存储引擎
>
> 利用好MySQL自身提供的功能，如索引等
>
> 横向扩展：MySQL集群、负载均衡、读写分离
>
> SQL语句的优化（收效甚微）
>
> > 1.尽可能使用 not null： MySQL中每条记录都需要额外的存储空间，表示每个字段是否为null， 存储需要额外空间，运算也需要特殊的运算符 
> >
> > 2.尽量不使用枚举：MySQL内部的枚举类型（单选）和集合（多选）类型，使用关联表替代枚举

### MySQL-索引

> 使用原则： sql语句自动调整，索引使用以最左匹配原则，遇到范围查询结束匹配 
>
> 普通索引：对关键字没有限制 （`key`） 
>
> 唯一索引：要求记录提供的关键字不能重复 （`unique key`） 
>
> 主键索引：要求关键字唯一且不为null （`primary key`） 
>
> **全文索引**（`fulltext key`） 
>
> > 创建索引
> >
> > 1．ALTER TABLE
> >
> > ```
> > ALTER TABLE用来创建普通索引、UNIQUE索引或PRIMARY KEY索引。
> > ALTER TABLE table_name ADD INDEX index_name (column_list)
> > ALTER TABLE table_name ADD UNIQUE (column_list)
> > ALTER TABLE table_name ADD PRIMARY KEY (column_list)
> > ```
> >
> > #### 2．CREATE INDEX
> >
> > ```
> > CREATE INDEX可对表增加普通索引或UNIQUE索引。
> > CREATE INDEX index_name ON table_name (column_list)
> > 
> > CREATE UNIQUE INDEX index_name ON table_name (column_list)
> > ```
> >
> > ### 3. 删除索引
> >
> > 可利用ALTER TABLE或DROP INDEX语句来删除索引。
> >
> > ```
> > DROP INDEX index_name ON talbe_name
> > ALTER TABLE table_name DROP INDEX index_name
> > 
> > ALTER TABLE table_name DROP PRIMARY KEY
> > 
> > 其中，前两条语句是等价的，删除掉table_name中的索引index_name。
> > 第3条语句只在删除PRIMARY KEY索引时使用，因为一个表只可能有一个PRIMARY KEY索引，因此不需要指定索引名
> > ```
> >
> > **5．查看索引**
> >
> > ```
> > mysql> show index from tblname;
> > ```
>
> 

### MySQL-优先级

>from 表
>
>where 条件( ==优先级为 :()>and>or,要想先执行or语句可以使用括号改变优先级==)
>
>group by 字段
>
>形成虚拟的表及字段，聚合及字段添加
>
>having 筛选数据   （函数，between等，但是条件是必须前面查询的列（即select 列）已经有了）
>
>distinct 去重
>
>order by 字段 asc,desc
>
>limit 限制条数
>
>select 罗列记录
>
>从上到下。先根据where 约束条件 将数据从表中加载到内存，所以where的条件作用在数据之前，然后根据字段进行分组，在内存中形成一张虚拟的表包含字段，如果此时有聚合函数则先执行聚合函数，并将聚合函数字段也加到虚拟的表中，接着进行having记录筛选，筛选完成后对数据进行去重，排序，限制等操作后进行显示。

### mysql -常用数据类型

> 1.整型INT(长度)，如int(3)表示-999~999范围的整数
> 2.浮点型Double(长度,精度)，其中长度表示有效数字位数，如double(3,2) 3.32
> 3.字符型VARCHAR(最大长度)，VARCHAR(20)表示字符最大长度为20，字符串常量使用双引号或单引号括起	来，如"abc"或'abc'
> 4.固定长度字符型CHAR(长度)，如CHAR(2)表示固定2个字符长度，如果数据长度不足数据库自动以空格填充
> 5.大字符型TEXT
> 6.日期时间类型
> 	DATE 表示日期（无时间）如"2017-1-1"
> 	TIME 表示时间（无日期）如"13:13:13"
> 	DATETIME表示日期和时间，如"2017-1-1 13:13:13"
> 	TIMESTAMP表示日期和时间,精度较高！"201701011313130000000"
>
> > ```
> > #日期和时间处理函数，mysql中的日期函数能处理的只有标准格式，即：字符类型为data yyyy-mm-dd  当字符类型为datatime时，为 yyyy-mm-dd 00：00：00
> > select name,id from table1 where order_data = '2005-09-03';     //data日期字符为			data型，当字符类型为time类型时，也可以查询  where order_data = '11:30:07'
> > select name,id from table1 where DATA(order_data) = '2005-09-03';     //当字符型为		datatime时，可以DATA()函数强转查询
> > select name,id from table1 where DATA(order_data) BETWEEN '2005-09-01' AND '2005-09-30';     //过滤查询多列时使用 between 列1 and 列2  函数，即查询2005年9月份所有		的数据
> > select name id from table1 where YEAR(order_data) = 2005 AND MONTH(order_data) = 9;           //使用year（）和month（）函数来查询多列结果，即查询2005年9月份所有的数据
> > ```

### MySQL-函数使用

>1.ALL是默认的，即全部行
>
>2.==COUNT() 返回某列的行数，count（*）对表中的所有行的数目进行计数，包括是NULL，count（列1）对特定列（列1）中具有值的行进行计数，忽略null值。（当使用 group by进行分组时，会对null单独分为一组，但是当一起使用的时候会统计出null组的数量）==
>
>```
>select COUNT(*) AS avg_price from table1;
>select CUONT(price) AS COUNT_price from table1;
>```
>
>3.AVG() 返回某列平均值，函数会忽略值为NULL的行
>
>```
>select AVG(price) AS avg_price from table1 where id = 1005;
>```
>
>4.MAX() MIN()返回某列的最大值，最小值，其函数会忽略列值为NULL的行		    
>5.SUM() 返回某列值的总和
>
>```
>select SUM(price) AS SUM_price from table1 where id = 1005;     //对id为1005的产品所 		有求和
>```
>
>6.CAST函数语法规则是:Cast(字段名 as 转换的类型 )

### mysql-ifnull

>  1.IFNULL() 函数用于判断第一个表达式是否为 NULL，如果为 NULL 则返回第二个参数的值，如果不为 NULL 则返回第一个参数的值。 
>
>  ifnull(列1，固定值)     //如果列1为空，返回固定值
>
>  **==msql中如果   int+null  返回null==**，反过来也一样
>
>  2. **if(条件exp，表达式1，表达式2)**
>
>   如果(exp不等于0且exp不为空)，条件成立(true)执行表达式1，否则，执行表达式2. 
>
>  SELECT IF(-1,5,2)：这里exp为值-1，条件成立执行表达式1，所以返回值为5。 

### mysql-order by

> ```
> ORDER BY field1 [ASC [DESC][默认 ASC]], [field2...] [ASC [DESC][默认 ASC]]
> ```
>
>  使用 ASC 或 DESC 关键字来设置查询结果是按升序或降序排列。 默认情况下，它是按升序排列。 
>
>  可以添加 WHERE...LIKE 子句来设置条件。 
>
> ```
> #以Cno升序、Degree降序查询Score表的所有记录。
> SELECT * FROM score ORDER BY cno ASC,DEGREE DESC 
> ```
>
> 

### mysql-运算符

>1.算数运算符，和java中的一样，+-*/%
>
>MySql中字符串会被自动转换为数字.
>
>如果第一位是数字的字符串被用于一个算数运算中，那么它被转换为这个数字的值。
>如果一个包含字符和数字混合的字符串不能被正确的转换为数字，那么它被转换成0
>
>```
>select 1+"a";   //1
>SELECT 1+2;     //3
>SELECT 1+"2";   //3
>SELECT 1+"2a";  //3
>SELECT 1+"a2";  //1
>```
>
>2.比较运算符
>
>>、<、>=、<=、=（等值比较）、!=或<>（不等、非等值比较） 
>>比较运算符可以用于比较数字和字符串。数字作为浮点值比较，而字符串以不区分大小写的方式进行比较
>
>3.逻辑运算符
>
>> and &&
>> or ||
>> not ! 

### mysql-外键约束/foreign key

>1.为表添加外键约束时，需要注意
>
>建立外键的表，必须为InnoDB型，不能使临时表，因为，在MySQL中只有InnoDB类型的表，才支持外键。
>
>定义外键名时，不能加引号，比如constraint’FK_ID’或constraint”FK_ID”都是错误的 ,用波浪号“``”
>
>2.设置外键语法
>
>```
>[CONSTRAINT symbol] FOREIGN KEY [id] (index_col_name, ...)  
>    REFERENCES tbl_name (index_col_name, ...)  
>    [ON DELETE {RESTRICT | CASCADE | SET NULL | NO ACTION}]  
>    [ON UPDATE {RESTRICT | CASCADE | SET NULL | NO ACTION}] 
>    
>    
>  关键字     	    含义
> -------------------------------------------------------------------------------
> CASCADE    	  删除包含与已删除键值有参照关系的所有记录
> -------------------------------------------------------------------------------
> SET NULL   	  修改包含与已删除键值有参照关系的所有记录，使用NULL值替换（只能用于已标记为				   	NOT NULL的字段）
> -------------------------------------------------------------------------------
> RESTRICT   	  拒绝删除要求，直到使用删除键值的辅助表被手工删除，并且没有参照时(这是默认设				  	置，也是最安全的设置)
> -------------------------------------------------------------------------------
> NO ACTION  	  啥也不做
>```
>
>3.添加/删除/查看一个外键
>
>```
>#为表添加一个外键
>alter table 表名 add foreign key(列名) references 表名(列名)
>#删除外键 
>alter table locstock drop foreign key locstock_ibfk2
>#查看表有哪些外键
>show create table locstock
>```
>
>4.user` 表：id 为主键,profile` 表： uid 为主键
>
>命名规范：FK_外键字段名（字段简称）
>
>表 `profile` 的 `uid` 列 作为表外键（外建名称：`user_profile`），以表 `user` 做为主表，以其 id列 做为参照（`references`），且联动删除/更新操作（`on delete/update cascade`）。则 `user` 表 中删除 `id` 为 1 的记录，会联动删除 `profile` 中 `uid` 为 1 的记录。`user` 表中更新 `id` 为 1 的记录至 `id` 为 2，则`profile` 表中 `uid` 为 1 的记录也会被联动更新至 `uid` 为 2，这样即保持了数据的完整性和一致性。 
>
>```
>#在profile中为uid列添加名为user_profile的外键，且此外键的参照为user表的id列，关联的操作为删除和更新
>alter table `profile` 
>add constraint `user_profile` foreign key (`uid`) 
>references `user`(`id`) on delete cascade on update cascade;
>```
>
>
>
>学习链接 https://www.cnblogs.com/yccmelody/p/5416456.html 

### MySQL-distinct

> 单独的distinct只能放在开头，否则报错，语法错误 
>
> 当`distinct`应用到多个字段的时候，其应用的范围是其后面的所有字段
>
> Distinct用法:
>
> ```
> #在count计算不重复的记录的时候能用到,就是计算talbebname表中id不同的记录有多少条
> SELECT COUNT( DISTINCT player_id ) FROM task;
> #如果放在了count（）函数前面，即是对count（）函数的结果进行去重
> SELECT DISTINCT COUNT(player_id ) FROM task;
> ```
>
> 

### mysql-where/having

> 1.在WHERE子句中使用谓词 :
>
> > BETWEEN    AND    :在两数之间(包含两数)
> > NOT   BETWEEN    AND　：不在两数之间
> > 	IN <值表>：是否在特定的集合里（枚举）
> > 	NOT IN <值表>　：与上面相反
> > 	LIKE		：是否匹配于一个模式
> > 	IS NULL（为空的）或 IS NOT NULL（不为空的）
> > 	REGEXP : 检查一个值是否匹配一个常规表达式。
>
> ```
> #优先级为 :()>and>or,要想先执行or语句可以使用括号改变优先级
> where id = 1002 or id =1003 and price = 10;
> #取反，即不等于1003的id
> where id <>1003  等价于 where id != 1003;
> # //查询列值等于空值的列，可用于查询ID或价格不存在的行
> where 列 is NULL;   
> #//in 和or 相似，但是in比or逻辑更加容易理解，还可以这样用：where id not in(1002,1003)；
> where id = 1002 or id = 1003
> where id IN (1002,1003);                              
> ```
>
> 2.hiving对于查询和where查询的区别
>
> ```
> SELECT AVG(score) as avgs,`subject` FROM scores GROUP BY `subject` HAVING avgs>79;
> SELECT AVG(score) as avgs,`subject` FROM scores GROUP BY `subject` where avgs>79;
> 这里第二句话是错误的
> ```
>
> ==where字句在聚合前先筛选记录，也就是说作用在group by和having字句前。而 having子句在聚合后对组记录进行筛选。==
>
> WHERE 在分组和聚集计算之前选取输入行（因此，它控制哪些行进入聚集计算）， 而 HAVING 在分组和聚集之后选取分组的行。 
>
> having一般跟在group by之后，执行记录组选择的一部分来工作的。
> where则是执行所有数据来工作的。where执行顺序大于聚合函数 
>
> ### 2. 只可以用where，不可以用having的情况
>
> ```sql
> select goods_name,goods_number from sw_goods where goods_price > 100
> select goods_name,goods_number from sw_goods having goods_price > 100 //报错！！！因为前面并没有筛选出goods_price 字段
> ```
>
> 

### mysql-改/altar

>```
>alter 列名 set default "默认值"    //可以更改指定列默认值
>alter table 表名 add foreign key(列名) references 表名(列名) 为表添加一个外键
>```
>

### mysql-增/insert

>插入搜索出来的数据
>
>```
>INSERT INTO table1 (id,name,age ,xxx,xxx) select id,name,age,xxx,xxx from table2; //select语句中可以包含where语句
>```

### MySQL-删除表

> **drop table table_name** : 删除表全部数据和表结构，立刻释放磁盘空间，不管是 Innodb 和 MyISAM; 
>
> **truncate table table_name** : 删除表全部数据，保留表结构，立刻释放磁盘空间 ，不管是 Innodb 和 MyISAM; （ 释放存储表数据所用的数据页来删除数据 ， 删除的过程中不会激活与表有关的删除触发器，占用资源更加少，速度更快 ）
>
> **delete from table_name** : 删除表全部数据，表结构不变，对于 MyISAM 会立刻释放磁盘空间，InnoDB 不会释放磁盘空间; 
>
> **delete from table_name where xxx** : 带条件的删除，表结构不变，不管是 innodb 还是 MyISAM 都不会释放磁盘空间;(==delete 操作以后，使用 **optimize table table_name** 会立刻释放磁盘空间，不管是 innodb 还是 myisam;==)
>
>```
>链接：https://www.runoob.com/note/27632
>```

### MySQL-创建用户

> CREATE USER 'sun'@'localhost' IDENTIFIED BY '123456';
> 赋权
> grant select,delete,update,create,drop on *.* to sun@"localhost" identified by "123456";
> grant all privileges on mysql.* to sun@localhost identified by '123456';
> 查看权限
> show grants for root@localhost;
> 删除权限
> revoke drop on *.* from sun@localhost



### mysql-权限管理

> ``` mysql
> 切换数据库：use mysql;
> 查看用户权限表： select user,host from user
> 修改权限：update user set host = '%' where user = 'root' 
> （提示报错不用管，忽略ERROR 1062 (23000): Duplicate entry '%-root' for key 'PRIMARY'）
> 刷新权限：flush privileges
> 权限变成这样：
> +------+-----------+
> | user | host      |
> +------+-----------+
> | root | %         |
> | root | 127.0.0.1 |
> +------+-----------+
> ```



### MySQL-shell脚本

> ``` shell
> #!/bin/sh
> MYSQL="mysql -h 192.168.211.11 -uroot -p123456 --default-character-set=utf8"
> sql="select * from mytest.emp;"
> result="$($MYSQL -e "$sql")"
> echo -e "$result"
> ```
>
> 然后：sh xxx.sh

### MySQL-SQL

> DDL(Data Definition Language)：数据库模式定义语言DDL(Data Definition Language)，是用于描述数据库中要存储的现实世界实体的语言。
> DDL：数据定义语言，通常是数据库管理系统的一部分，用于定义数据库的所有特性和属性，尤其是行布局、列定义、键列（有时是选键方法）、文件位置和存储策略。
> 包括命令：DROP,CREATE,ALTER,GRANT,REVOKE, TRUNCATE
> DML(Data Manipulation Language)数据操纵语言，主要是对数据库中的数据进行一些简单操作，如insert,delete,update,select等
> DCL（Data Control Language）： 是数据库控制功能。是用来设置或更改数据库用户或角色权限的语句，包括（grant,deny,revoke等）语句

### mysql-基础定义

>1.什么是数据库：用于存储大量数据的程序和文件
>1.1数据(Data)
>1.2数据库(Database)：存储在物理存储介质（硬盘、光盘、U盘等）中的数据文件集合的总称
>1.3数据库管理系统(DBMS)：用于管理数据库文件的程序
>1.4数据库系统(DBS)：数据库(Database)和数据库管理系统(DBMS)的总称
>2.Structured Query Language(结构化查询语言)SQL
>2.1提供数据定义语言(DDL)
>2.2DML（ Data Manipulation Language数据操作语言）：（增、删、改、查）
>2.3DDL（ Data Definition Language数据定义语言）
>2.4DCL（ Data Control Language数据控制语言）
>3.流行常见的数据库：Oracle、SQL Server、DB2、MySql

### MySQL-数据备份

> mysqldump -uroot -p dolphin > /apps/dolphin.sql
> 该命令是将dolphin数据库备份到本地/apps/dolphin.sql文件中
> 上述命令执行过程中会提示如下信息
> Enter password:
> 这里输入用户密码“123456”，回车继续
> 命令执行完成后，查看/apps/目录下文件信息

### MySQL-数据导入

> use dolphin1;
> source /apps/dolphin.sql
> MySQL-数据存放位置
> MySQL数据文件"datadir"：
> show variables like '%dir%';

### MySQL-备份，恢复数据库

> 1.备份数据库表中的数据
>
> cmd> mysqldump -u 用户名 -p 数据库名 > 文件名.sql
>
> 2.恢复数据库
>
> source 文件名.sql  // 在mysql内部使用
>
> mysql –u 用户名 p 数据库名 < 文件名.sql // 在cmd下使用

### MySQL-安全管理
>    select user from mysql.user;                           // 查询使用用户
>    create USER hadoop IDENTIFIED BY '123456'  //创建用户hadoop 和使用口令123456
>    RENAME USER hadoop to hadoop2;                //重命名用户
>设置访问权限，
>
>>  show GRANTS FOR hadoop2;                      //查看hadoop2上权限		
>>  GRANTS select ON database1.* TO hadoop2  //允许用户hadoop在database1数据库中的所有表上使用select，即hadoop2只有访问database1的权限
>>
>>  REVOKE select ON database1.* TO hadoop2  //撤销授权
>>  整个服务器，使用GRANTS ALL   REVOKE ALL
>> 整个数据库，使用ON database.*;
>> 特定表，使用ON database.table;
>> 更改口令,在不指定用户名时，则更新当前登录的用户名
>>
>> SET PASSWORD FOR hadoop = Password（‘n3w xxxxxx’）  //新口令必须传递到password（）函数进行加密
>
>查看表键是否正确
>
>> ANALYZE TABLE table1;

### mysql-事务acid

> - Atomicity（原子性）：一个事务（transaction）中的所有操作，或者全部完成，或者全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。即，事务不可分割、不可约简。
>
> - Consistency（一致性）：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设约束、触发器、级联回滚等。
>
> - Isolation（隔离性）：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括未提交读（Read uncommitted）、提交读（read committed）、可重复读（repeatable read）和串行化（Serializable）。
>
> - Durability（持久性）：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。
>
> 	>  脏读：一个事务中作出修改后被另一个事务读取之后又被事务回滚，那么说另一个事务读了“脏”数据。 

### mysql-三级模式

>  数据库系统的三级模式结构是指数据库系统由外模式、模式和内模式3级构成。
>
> **外模式：**当模式改变时，由数据库管理员对各个外模式/模式的映像做相应的改变，可以使外模式保持不变。应用程序是依据数据的外模式编写的，从而应用程序不必修改，保证了数据与程序的逻辑独立性，简称数据逻辑独立性。 
>
> 逻辑独立性是外模式不变，模式改变时，如增加新的关系，新的属性，改变属性的数据类型，由数据库管理员对各个外模式/模式的映像做相应改变，可以使得外模式不变，因为应用程序依据外模式编写的，所以外模式不变，应用程序也不变，即保证了逻辑独立
> 物理独立性是模式不变，内模式改变，如数据库存储结构发生改变，选用另一种数据结构，由数据库管理员对各个模式/内模式的映像做相应改变，可以使得模式不变 ，从而保证了应用程序也不变

### mysql-sql注入

> SQL注入，就是通过把SQL命令插入到Web表单递交或输入域名或页面请求的查询字符串，最终达到欺骗服务器执行恶意的SQL命令。 

### ==mysql-jdbc==

>详情查看java.md的jdbc

# maven

### 1.pom.xml文件内容

```
<!--  此项目的第几个版本，编码格式是什么   -->
<?xml version="1.0" encoding="UTF-8"?>

<!--  pom模型版本，源地址，下载来源   -->
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

<!--  项目标签   -->
	<!--  项目组织   -->
    <groupId>dome_maven</groupId>  
    <!--  项目名称   -->
    <artifactId>dome_maven</artifactId>
    <!--  项目版本号   -->
    <version>1.0-SNAPSHOT</version>
    
    <!--  项目编码，引用的jar包   -->
    <dependencies>
    <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.6</version>
    </dependency>
    </dependencies>    
</project>
```



2.换源

将源换成阿里的源，把下面的代码粘贴到本身的自带的<mirror></mirror>下面

```
   <mirror>
      <id>alimaven</id>
      <mirrorOf>*</mirrorOf>
      <url>https://maven.aliyun.com/repository/central</url>
     </mirror>
  </mirrors>
```

​       