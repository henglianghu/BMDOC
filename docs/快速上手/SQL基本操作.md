## 登录

AiSQL安装部署完成之后，可通过bcql客户端登录并对其进行操作。

首先,需要进入bcql客户端程序所在的位置，AiSQL默认安装路径是bigmath/AiSQL-x.x.x.x,使用如下命令可以进入bcql所在的目录，此目录下面会有一个cqlsh的程序，运行如下语句：

```
cd bigmath/AiSQL-x.x.x.x/bin
```

注意，上述命令中的x表示版本号，需要根据当前安装的AiSQL版本的具体版本号进行替换；

其次，通过sqlsh客户登录AiSQL进行操作，默认用户是bigmath, 默认密码是 bigmath，运行如下语句:

```
./sqlsh -h 192.168.50.111 -U bigmath
```

-h参数后面的值是AiSQL安装部署所在的ip地址，默认端口是2521(参数-p)  ，-U 后面的参数是用户名(bigmath),此用户的默认密码是bigmath。



## 建库、建表、建表索引



### 创建数据库

BSQL兼容PostgreSQL数据库的SQL语法。

在创建数据库之前，使用\l查看当前已经存在的数据库，运行如下语句：

```
\l
```

创建一个新的数据库testdb，注意不要与已存在的数据库名同名，比如创建一个名为testdb数据库，运行如下语句:

```
CREATE DATABASE testdb;
```

进入刚才创建的数据库,运行如下语句:

```
\c testdb;
```

上述命令进入testdb数据库之后，就可以在此数据库下面创建表和插入表数据了。



### 新建表

上述命令创建了数据库testdb，并进入了此数据库，下面接着对此数据库进行操作。首先，创建一个员工信息表，此表包括的信息有：工号、姓名、邮箱、入职日期、职位、薪资、归属部门，运行如下语句创建员工信息表：

```
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL,
    hire_date DATE NOT NULL,
    job_title VARCHAR(100),
    salary NUMERIC(10,2),
    department_id INT
);
```



### 新建表索引

在employees表的name字段上建立普通索引(非唯一索引)，运行如下语句：

```
CREATE INDEX employees_name ON employees (name);
```

在employees表的name字段上建立唯一索引，运行如下语句：

```
CREATE UNIQUE INDEX employees_name ON employees (name);
```

上述语句中，employees_name是索引的名称，可以自定义该名称。employees是表名，括号内的name是employees表的字段名，如果需要在多个字段上建立索引，括号内可以包含多个字段名，并使用逗号隔开。

### 删除表索引

上面的操作为表创建了employees_name的索引，既然可以为表添加索引，故也可以删除索引，运行如下语句

```
DROP INDEX employees_name;
```



### 查询表结构和索引信息

上述过程我们创建了员工信息表employees，并且为此表创建了索引，如果需要查询employees表结构和employees表索引信息，运行如下语句：

```
\d employees; 
```



## 表数据操作



### 插入数据



上述过程创建了数据库testdb，并在此数据库下面创建了employees表和索引，接下来可以向此表插入数据，运行如下语句:

```
INSERT INTO employees VALUES (1, 'Zhang San', 'abc001@136.com','2020-09-12','development engineer',20000.00,3);
```

如果想批量向employees表插入数据，运行如下语句:

```
INSERT INTO employees VALUES (2, 'Li Si', 'abc001@136.com','2020-09-12','development engineer',15000.00,3),(3, 'Wang Wu', 'abc001@136.com','2020-09-12','development engineer',25000.00,3);
```

上述语句批量向employees表插入2条数据，数据之间使用逗号隔开。



### 查询数据


查询表employees表所有数据，运行如下语句:

```
SELECT * FROM employees;
```

查询表employees表中员工姓名是Zhang San  的数据，运行如下语句:

```
SELECT * FROM employees where name='Zhang San';
```

查询表employees表中部门号是3所有员工信息，并且按照工资降序排列，运行如下语句:

```
SELECT * FROM employees WHERE department_id = 3 ORDER BY salary DESC;
```



### 修改数据



修改employees表中姓名是Zhang San 员工的工资为25000，运行如下语句:

```
UPDATE employees set salary=25000.00 where name='Zhang San';
```

查询修改后的数据是否正确，运行如下语句:

```
SELECT * FROM employees where name='Zhang San';
```



### 删除数据



删除employees表中姓名是Zhang San 员工信息，运行如下语句

```
DELETE FROM employees where name='Zhang San';
```

删除employees表中的所有数据，运行如下语句

```
DELETE FROM employees;
```



## 删除表、删除库



### 删除表

上面的操作创建了employees表，也可以删除此表，运行如下语句

```
DROP TABLE employees;
```



### 删除库



上面的操作创建了testdb库，也可以删除此库，但是不能在当前库下面删除当前库，故需要退出当前testdb库，请连接到另一个数据库，之后再删除testdb库，首先，连接到另外一个库，运行如下语句

```
\c bigmath
```

其次，删除testdb库，运行如下语句

```
DROP DATABASE testdb;
```



## 创建、授权和删除用户



假设需要为testdb数据库创建一个aiuser用户，密码为123456，此用户对testdb数据库下的表具有查询和插入的权限，具体操作如下。



### 数据准备



上面的操作我们已经把数据库testdb和表都进行了删除，这里的操作我们需要重新创建数据库和表，运行如下语句

```
CREATE DATABASE testdb;
```

进入刚才创建的数据库,运行如下语句:

```
\c testdb;
```

创建employees表，运行如下语句

```
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL,
    hire_date DATE NOT NULL,
    job_title VARCHAR(100),
    salary NUMERIC(10,2),
    department_id INT
);
```

接下来可以向此表插入数据，运行如下语句:

```
INSERT INTO employees VALUES (1, 'Zhang San', 'abc001@136.com','2020-09-12','development engineer',20000.00,3);
```



### 创建用户



创建aiuser用户，密码为123456，运行如下语句

```
CREATE USER aiuser WITH PASSWORD '123456';
```



### 授权用户



授权aiuser用户访问数据库testdb的查询(select)、插入(insert)数据操作权限，运行如下语句

```
GRANT SELECT, INSERT ON ALL TABLES IN SCHEMA public TO aiuser;
```



### 删除用户



在删除aiuser用户之前，如果当前是使用aiuser用户登录数据库，那么退出数据库(\q),之后重新使用具有管理权限的用户登录数据库，之后再进入到testdb数据库中(\c testdb), 之后再进行如下操作就可以删除aiuser用户了。

首先，去掉aiuser的授权，运行如下语句

```
REVOKE ALL PRIVILEGES ON ALL TABLES IN SCHEMA public FROM aiuser;
```

其次，删除aiuser用户，运行如下语句

```
DROP USER aiuser;
```

上面的DROP命令已经删除了用户，还可以查看用户是否还存在，运行如下语句

```
SELECT rolname, rolcanlogin, rolsuper FROM pg_roles;

```
