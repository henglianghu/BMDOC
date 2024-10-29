## 建库、建表、建表索引

### 新建键空间

在新建键空间前，您需要查询已有的键空间以预防使用已占用名称来新建键空间。

```
cqlsh> desc keyspaces;
system_schema  system_auth  system
```

在确认无误后，您可以使用如下命令进行创建：

```
cqlsh> create keyspace cas_db; 
```

创建后再次查询已有键空间：

```
cqlsh> desc keyspaces;
system_schema  system_auth  system  cas_db
```

您还可以进一步查看创建的键空间相关信息：

```
cqlsh> desc cas_db;
CREATE KEYSPACE cas_db WITH replication = {'class': 'SimpleStrategy', 'replication_factor': '3'}  AND durable_writes = true;
```

确认无误后可使用新建键空间：

```
cqlsh> use cas_db;
cqlsh:cas_db> 
```



### 新建表

创建表需要在指定的键空间进行操作。

创建表之前需要查看 cas_db 中已有的表：

```
cqlsh:cas_db> DESCRIBE TABLES;
```

确认无误后，在键空间 cas_db 中创建一个名为 stock_market 的表：

```
cqlsh> create table cas_db.stock_market(
    stock_symbol text,    /* 股票代码 */ 
    ts text,    /* timestamp */
    cur_price float,    /* 股票价格 */
    primary key(stock_symbol, ts));    /* 主键 */
```

查看表的信息：

```
cqlsh> desc cas_db.stock_market;
CREATE TABLE cas_db.stock_market (
    stock_symbol text,
    ts text,
    cur_price float,
    PRIMARY KEY (stock_symbol, ts)
) WITH CLUSTERING ORDER BY (ts ASC)
    AND default_time_to_live = 0
    AND transactions = {'enabled': 'false'};
```



### 建表索引

您可以使用 `CREATE INDEX` 语句在 `BCQL` 中创建索引，语法如下：

```CQL
CREATE INDEX index_name ON table_name(column_list)
```

以上述创建的表作为示例，用户可创建表索引：

```cql
CREATE INDEX index_stock_symbol ON stock_market(stock_symbol)
```

这样您就为 `stock_symbol` 列创建了索引。



## 插入表数据、查询表数据

### 插入数据

用户可使用 `INSERT` 向表内插入数据。

使用上述创建的表 `cas_db.stock_market` 为例，插入两条记录：

```cql
cqlsh> INSERT INTO cas_db.stock_market (stock_symbol,ts,cur_price) VALUES ('AAPL','2017-10-26 09:00:00',157.41);
cqlsh> INSERT INTO cas_db.stock_market (stock_symbol,ts,cur_price) VALUES ('AAPL','2017-10-26 10:00:00',157);
```

如果您需要更新数据，可以使用 `UPDATE` 更新某行记录：

```cql
cqlsh:cas_db> UPDATE stock_market set cur_price = 158.1 where stock_symbol = 'AAPL' and ts = '2017-10-26 10:00:00';
```

然后查询更新后的内容：

```cql
cqlsh:cas_db> select cur_price from stock_market where stock_symbol = 'AAPL' and ts = '2017-10-26 10:00:00';
 cur_price-----------
 158.10001
```



### 查询表数据

您可以使用 `select` 语句查看表中的数据。

使用上述创建的表 `cas_db.stock_market` 为例，进行表数据的查询：

```cql
cqlsh> select * from cas_db.stock_market ;

 stock_symbol | ts | cur_price 
 --------------+----+----------- 
(0 rows)
```

由于新建的表没有存放数据，所以显示的内容为0行。

在使用上述插入指令插入数据后再进行查询，将会看到如下显示：

```cql
# 插入数据
cqlsh> INSERT INTO cas_db.stock_market (stock_symbol,ts,cur_price) VALUES ('AAPL','2017-10-26 09:00:00',157.41);
cqlsh> INSERT INTO cas_db.stock_market (stock_symbol,ts,cur_price) VALUES ('AAPL','2017-10-26 10:00:00',157);

# 查询表数据
cqlsh> select * from cas_db.stock_market ;
stock_symbol | ts | cur_price 
--------------+---------------------+----------- 
AAPL | 2017-10-26 09:00:00 | 157.41 
AAPL | 2017-10-26 10:00:00 | 157 
```



## 删除表索引、删除表数据

### 删除表索引

如果您需要删除表中的索引，您可以使用 `DROP INDEX` 语句来删除一个或者多个现有索引。

```cql
DROP INDEX index_name1, index_name2, ...;
```

使用上文创建的索引为例：

```cql
DROP INDEX index_stock_symbol;
```



### 删除表数据

您可以使用 `DELETE` 删除某行记录.

同样使用上文创建的表作为示例

```cql
cqlsh:cas_db> DELETE from stock_market where stock_symbol = 'AAPL' and ts = '2017-10-26 10:00:00';
```

删除数据后再进行查询：

```cql
cqlsh:cas_db> select * from stock_market ;
stock_symbol | ts | cur_price 
--------------+---------------------+----------- 
AAPL | 2017-10-26 09:00:00 | 157.41 
(1 rows)
```

可以看到指定的数据已经从表内被移除了。



## 删除表、删除库

### 删除表

您可以使用 `drop table` 来删除不需要的表。

在删除前请查看存在的表以避免误删等误操作：

```cql
cqlsh:cas_db> DESCRIBE TABLES;
```

确认无误后使用 `drop table` 进行表的删除：

```cql
cqlsh:cas_db> drop table stock_market;
```



### 删除键空间

删除键空间：需要注意的是当键空间包含表时，需要先删除全部表才能删除键空间。

```cql
cqlsh> drop keyspace cas_db;
```

查看是否删除成功：

```cql
cqlsh> desc keyspaces;
system_schema  system_auth  system
```



## 创建、授权和删除用户

默认情况下，AiSQL 已经创建了一个管理员用户：cassandra（推荐用户），您可以按如下方式查看现有用户：

```cql
cqlsh> select * from system_auth.roles;
```

输出如下：

```cql
 Role    | can_login | is_superuser | member_of | salted_hash
------------+-------------+--------------+---------------+------------------------------------------------------------------------------
cassandra |     True |      True |         [] | $2a$12$64A8Vo0R3K9XeUp26CSzpuWtvUBwOiGFjPAbXGt7wsxZIScGrcsDu\x00\x00\x00\x00
```



### 创建

您可以使用 `CREATE ROLE` 创建新的角色：

```cql
cqlsh> CREATE ROLE new_user;
```

请注意不要创建于原有用户冲突的新用户。



### 授权

使用 `GRANT PERMISSION` 语句向角色授予权限（或所有可用权限）。 

创建数据库对象（键空间、表或角色）时，将自动且明确地向创建该对象的角色授予与该对象相关的所有权限。 

通过将 `dbServer` 标志 `--use_cassandra_authentication` 设置为 `true`，可以启用此语句。

以创建的 `cas_db` 和新角色 `new_user` 为示例：

```cql
cqlsh> GRANT all_permission ON KEYSPACE cas_db TO new_user
```



### 删除
使用 `DROP ROLE` 语句可以删除现有角色。 

通过将 `dbServer` 标志 `--use_cassandra_authentication` 设置为 `true`，可以启用此语句。

语法：

```
drop_role ::=  DROP ROLE [ IF EXISTS ] role_name
```

以新创建的 `new_user` 作为示例：

```cql
cqlsh> DROP ROLE new_user
```

注意：
* 如果要删除的角色 `role_name` 不存在，除非存在 `IF EXISTS` 选项，则会引发错误。 
* 只有具有超级用户状态的角色才能删除另一个超级用户角色。 
* 只有对 `ALL ROLES` 或指定的 `role_name` 具有DROP权限或具有SUPERUSER状态的客户端才能删除另一个角色。
