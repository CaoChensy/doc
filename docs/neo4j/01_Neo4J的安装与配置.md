#! https://zhuanlan.zhihu.com/p/432189696

![Image](https://pic4.zhimg.com/80/v2-e0485df818c9e7b609daf45c92cdde26.jpg)

# Neo4J 的安装与配置

## neo4j

```bash
NEO4J_HOME: E:\neo4j-community-3.4.1

%NEO4J_HOME%\bin
```

```bash
neo4j.bat console 

neo4j install-service

neo4j start
```

- http://localhost:7474/browser/
- 默认密码：neo4j
- 更改密码。完成。

> 安装APOC

1. [APOC 下载地址](https://github.com/neo4j-contrib/neo4j-apoc-procedures)
2. 将apoc-*.jar包放置在neo4j/plugins目录下
3. 设置安全策略：不限制apoc的所有存储过程
```conf
dbms.security.procedures.unrestricted=apoc.*,algo.*
apoc.import.file.enabled=true
#增加页缓存到至少4G，推荐20G:
dbms.memory.pagecache.size=4g
#JVM堆保存留内存从1G起，最大4G:
dbms.memory.heap.initial_size=1g
dbms.memory.heap.max_size=4g
```
4. 重启 `neo4j`
5. 验证 `return apoc.version()`

> 开启远程访问

一、对于3.0以前的版本

在安装目录的 $NEO4J_HOME/conf/neo4j.conf 文件内，找到下面一行，将注释#号去掉就可以了 
#dbms.connector.https.address=localhost:7473 改为 dbms.connector.https.address=0.0.0.0:7473 
这样，远程其他电脑可以用本机的IP或者域名后面跟上7474 端口就能打开web界面了 如： https://:7473

操作系统的防火墙也要确保开放了7474端口。

二、对于3.1及以后的版本

在安装目录的 $NEO4J_HOME/conf/neo4j.conf 文件内，找到下面一行，将注释
#号去掉就可以了 dbms.connectors.default_listen_address=0.0.0.0

> 用户管理

**用户创建：**

```cql
CALL dbms.security.createUser(name,password,requridchangepassword)
```

其中nam参数是你的用户名，
password是密码，
requridchangepassword是表示是否需要修改密码，布尔类型。
如下创建一个test1用户，密码是test1,不需要修改密码。

**查看所有用户：**

```cql
CALL dbms.security.listUsers();
```

**删除用户：**

```cql
CALL dbms.security.deleteUser("username") username参数表示你要删除的用户名。
```

---
