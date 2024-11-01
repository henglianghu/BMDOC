## 登录

AiSQL安装部署完成之后，可通过bcql客户端登录并对其进行操作。

首先,需要进入bcql客户端程序所在的位置，AiSQL默认安装路径是bigmath/AiSQL-x.x.x.x,使用如下命令可以进入bcql所在的目录，此目录下面会有一个cqlsh的程序，运行如下语句：

```
cd bigmath/AiSQL-x.x.x.x/bin
```

注意，上述命令中的x表示版本号，需要根据当前安装的AiSQL版本的具体版本号进行替换；

其次，通过bcql客户登录AiSQL进行操作，默认用户是cassandra, 默认密码是 cassandra，运行如下语句:

```
./cqlsh 192.168.50.111 9542 -u cassandra
Password: 
Connected to local cluster at 192.168.50.111:9542.
[cqlsh 5.0.1 | Cassandra 3.9-SNAPSHOT | CQL spec 3.4.2 | Native protocol v4]
Use HELP for help.
cassandra@cqlsh>
```

注意：上面的参数值192.168.50.111是AiSQL安装部署所在的ip地址，9542是登录的网络端口号，-u 后面的参数是用户名(cassandra),此用户的默认密码是cassandra，输出上述登录语句之后按回车，会出现Password: 提示输入密码，之后输入密码 cassandra即可完成登录。



##  建键空间、建表、建表索引 



### 新建键空间

在创建数据库之前，使用desc keyspaces 命令查看当前已经存在的数据库，以预防此键空间名已经被占用，运行如下语句：

```
DESC KEYSPACES;
```

创建一个新的键空间 test_key_spaces，注意不要与已存在的键空间同名，运行如下语句:

```
CREATE KEYSPACE test_key_spaces; 
```

创建后再次查询已有键空间,是否已存在了，运行如下语句:：

```
DESC KEYSPACES;
```

您还可以进一步查看创建的键空间相关信息，运行如下语句:

```
DESC test_key_spaces;
```

确认无误后可使用新建键空间，运行如下语句：

```
USE test_key_spaces;
```

上面命令就进入use test_key_spaces键空间了。

### 新建表

上述命令创建了use test_key_spaces键空间，并进入了此键空间，下面接着对此键空间进行表(员工信息表employees)创建操作。

首先，在创建employees表之前需要查看 test_key_spaces中已有的表，以预防此表名已经被占用，运行如下语句:

```
DESCRIBE TABLES;
```

其次，创建一个员工信息表employees，此表包括的信息有：工号、姓名、邮箱、入职日期、职位、薪资、归属部门，运行如下语句：

```
CREATE TABLE employees (
    employee_id BIGINT,
    name varchar,
    email varchar,
    hire_date DATE,
    job_title varchar,
    salary DOUBLE,
    department_id INT,
    primary key(employee_id))WITH transactions = {'enabled': 'true'};
```

注意：上面的transactions必须设置为true，否则无法为此表创建索引，如果不想再为此表创建索引，那么上述语句中可以删除WITH关键词后面的语句。

最后，查看表的信息，运行如下语句：

```
DESC employees;
```

注意：数据类型varchar与TEXT是同一种数据类型，互为别名。

### 建、删表索引

在employees表的name字段上建立普通索引(非唯一索引)，运行如下语句：

```
CREATE INDEX employees_name ON employees (name);
```

如果想在employees表的name字段上建立唯一索引，运行如下语句：

```
CREATE UNIQUE INDEX employees_name ON employees (name);
```

上述语句中，employees_name是索引的名称，可以自定义该名称。employees是表名，括号内的name是employees表的字段名，如果需要在多个字段上建立索引，括号内可以包含多个字段名，并使用逗号隔开。

如果不想使用说明的索引，那么可以删除此索引，运行如下语句：

```CQL
DROP INDEX employees_name;
```



## 表数据操作

### 插入数据

上述过程创建了employees表，接下来可以向此表插入数据，运行如下语句:

```
INSERT INTO employees(employee_id,name,email,hire_date,job_title,salary,department_id) VALUES (1, 'Zhang San', 'abc001@136.com','2020-09-12','development engineer',20000.00,3);
```



### 查询表数据

查询employees表的所有数据，运行如下语句:

```cql
select * from employees;
```

查询employees表中员工工号是2的员工信息，运行如下语句:

```cql
select * from employees where employee_id=2;
```

查询employees表中员工姓名是Li Si的员工信息，运行如下语句:

```cql
select * from employees where name='Li Si';
```

查询employees表的所有数据,并按部门号(department_id)分组显示，运行如下语句:

```cql
select * from employees group by department_id;
```



### 修改表数据

在employees表中，修改工号是1的员工姓名为Zhang Xiao Shan，运行如下语句:

```cql
UPDATE employees SET name='Zhang Xiao Shan' WHERE employee_id=1;
```

在employees表中，给工号是2的员工薪水增加5000，运行如下语句:

```cql
UPDATE employees SET salary = salary + 5000 WHERE employee_id=2;
```



### 删除表数据

在employees表中，删除工号是2的员工数据，运行如下语句:

```cql
DELETE from employees WHERE employee_id=2;
```





## 删除表、删除库

### 删除表

在删除前最好是先查看存在的表以避免误删等误操作，运行如下语句:

```cql
DESC TABLES;
```

确认无误后使用 `drop table` 进行表的删除，比如删除employees表，运行如下语句:

```cql
DROP TABLE employees;
```



### 删除键空间

在删除键空间前最好是先查看一下当前已存在的键空间，以避免误删等误操作，运行如下语句:

```cql
DESC KEYSPACES;
```

确认无误后使用drop keyspace 删除键空间, 需要注意的是当键空间包含表时，需要先删除全部表才能删除键空间。比如删除键空间test_key_spaces，运行如下语句:

```cql
DROP KEYSPACE test_key_spaces;
```



## 创建、授权和删除用户



### 授权创建用户权限

采用默认配置启动的AiSQL，是不支持创建新用户的，这是为了确保数据库在初始状态下的安全性，采取了较为保守的安全策略，不允许随意创建 BCQL 用户可以防止未经授权的访问和潜在的安全漏洞。在没有明确的安全设置和授权流程的情况下，随意创建用户可能会导致数据泄露、恶意攻击等风险。

所以，如果需要创建新的用户，那么需要在启动AiSQL的时候为cassandra用户授权，授权命令如下：

```cql
./bigmathd start --use_cassandra_authentication=true
```





### 查看当前用户信息

AiSQL启动之后，采用cassandra登录bcsql客户端，在创建新的用户之前，查询一下当前有哪些用户名，以防与新创建的用户名同名，运行如下语句:

```cql
cassandra@cqlsh> select * from system_auth.roles;
```



### 创建新用户

您可以使用 `CREATE ROLE` 创建新的角色，比如创建用户aiuser，并且设置登录密码为123456，运行如下语句:

```cql
cassandra@cqlsh> CREATE ROLE aiuser WITH PASSWORD = '123456' AND LOGIN = true;
```



### 授权用户

向aiuser授权访问键空间test_key_spaces的所有权限，运行如下语句:

```cql
cassandra@cqlsh>GRANT ALL PERMISSIONS ON KEYSPACE test_key_spaces TO aiuser;
```



### 删除用户

删除aiuser用户，运行如下语句:


```cql
cassandra@cqlsh> DROP ROLE aiuser
```
