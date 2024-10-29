## **概述**

### **支持的项目**

建议使用以下项目来实现使用 BMDB BSQL/BCQL API 的 Ruby 应用程序。

| 驱动                      |
| ------------------------- |
| Pg Gem Driver             |
| BMDB Ruby Driver for BCQL |


| 项目              | 应用示例    |
| ----------------- | ----------- |
| Active Record ORM | Sample apps |

了解如何建立与 BMDB 数据库的连接并开始基本的 CRUD 操作，方法是参考连接应用或使用 ORM.

先决条件
要为 BMDB 开发 Ruby 应用程序，您需要满足以下条件：

* Ruby
  安装Ruby 2.7.0 或更高版本。验证 ruby 版本命令：

```
ruby -v
```


## **应用连接**

### **BSQL**

#### Pg Gem驱动程序

3.7.10.2.1.1.1. *先决条件*
使用以下命令安装 Ruby PostgreSQL 驱动程序

```
$ gem install pg -- --with-pg-config=<bigmath-install-dir>/postgres/bin/pg_config
```

有关驱动程序的更多信息，请参阅PG 驱动程序文档.

3.7.10.2.1.1.2. *创建应用程序*
创建一个文件bmdb-sql-helloworld.rb，并在其中添加以下内容。


```
#!/usr/bin/env ruby
 
require 'pg'
 
begin
  # Output a table of current connections to the DB
  conn = PG.connect(host: '127.0.0.1', port: '2521', dbname: 'bigmath', user: 'bigmath', password: 'bigmath')
 
  # Create table
  conn.exec ("CREATE TABLE employee (id int PRIMARY KEY, \
                                     name varchar, age int, \
                                     language varchar)");
 
  puts "Created table employee\n";
 
  # Insert a row
  conn.exec ("INSERT INTO employee (id, name, age, language) \
                            VALUES (1, 'John', 35, 'Ruby')");
  puts "Inserted data (1, 'John', 35, 'Ruby')\n";
 
  # Query the row
  rs = conn.exec ("SELECT name, age, language FROM employee WHERE id = 1");
  rs.each do |row|
    puts "Query returned: %s %s %s" % [ row['name'], row['age'], row['language'] ]
  end
 
rescue PG::Error => e
  puts e.message
ensure
  rs.clear if rs
  conn.close if conn
end
```

3.7.10.2.1.1.3. *运行应用程序*
若要使用该应用程序，请运行以下命令：

```
$ ./bmdb-sql-helloworld.rb
```

您应该看到以下输出。

```
Created table employee
Inserted data (1, 'John', 35, 'Ruby')
Query returned: John 35 Ruby
```

### **BCQL**

#### BMDB Ruby驱动程序

##### *安装驱动*
使用以下命令安装驱动程序

```
gem install bigmath-bcql-driver
```

##### *创建示例 Ruby 应用程序*
创建一个文件，并将以下内容复制到该文件bmdb-ycql-helloworld.rb中。

```
require 'bcql'
 
# Create the cluster connection, connects to localhost by default.
cluster = Cassandra.cluster
session = cluster.connect()
 
# Create the keyspace.
session.execute('CREATE KEYSPACE IF NOT EXISTS bmdbdemo;')
puts "Created keyspace bmdbdemo"
 
# Create the table.
session.execute(
  """
  CREATE TABLE IF NOT EXISTS bmdbdemo.employee (id int PRIMARY KEY,
                                              name varchar,
                                              age int,
                                              language varchar);
  """)
puts "Created table employee"
 
# Insert a row.
session.execute(
  """
  INSERT INTO bmdbdemo.employee (id, name, age, language)
  VALUES (1, 'John', 35, 'Ruby');
  """)
puts "Inserted (id, name, age, language) = (1, 'John', 35, 'Ruby')"
 
# Query the row.
rows = session.execute('SELECT name, age, language FROM bmdbdemo.employee WHERE id = 1;')
rows.each do |row|
  puts "Query returned: %s %s %s" % [ row['name'], row['age'], row['language'] ]
end
 
# Close the connection.
cluster.close()
```

##### *运行应用程序*
若要使用该应用程序，请运行以下命令：

```
$ ruby bmdb-cql-helloworld.rb
```

您应该看到以下输出。

```
Created keyspace bmdbdemo
Created table employee
Inserted (id, name, age, language) = (1, 'John', 35, 'Ruby')
Query returned: John 35 Ruby
```

## **使用 ORM**

Active Record是用于 Ruby 应用程序的对象关系映射 （ORM） 工具。

BMDB BSQL API 与 Active Record ORM 完全兼容，可在 Ruby 应用程序中实现数据持久化。

要开始构建您的应用程序，请确保您已满足先决条件.

### **CRUD 操作**

本页提供了使用orm-examples存储库连接到BMDB的Active Record ORM入门的详细信息。

此存储库有一个 Ruby on Rails 示例，该示例实现了基本的 REST API 服务，该方案是电子商务应用程序的方案。此应用程序中的数据库访问通过 Active Record ORM 进行管理。它包括以下内容。

* 电子商务网站的用户存储在用户表中。
* 产品表包含电子商务网站销售的产品列表。
* 用户下达的订单将填充到订单表中。一个订单可以由多个明细项组成，每个明细项都插入到 orderline 表中。
  上述应用程序的源代码可以在存储库中找到，可以在属性文件config/database.yml中自定义一些选项。

#### 克隆 orm-examples 存储库

```
git clone https://gitlab.bigmath.com/AiSQL-Samples/orm-examples.git
```

#### 生成并运行应用程序

要安装项目 Gemfile 中指定的依赖项，请执行以下操作：

```
cd ./orm-examples/ruby/ror/
./bin/bundle install
```

使用以下命令创建数据库：

```
bin/rails db:create
```

您应看到类似于以下内容的输出：

```
Created database 'bsql_active_record'
```

使用以下命令执行迁移以创建表并添加列：

```
$ bin/rails db:migrate
```

使用以下命令启动 rails 服务器：

```
$ bin/rails server
```


#### 向应用程序发送请求

从另一个终端向应用程序发送请求，如下所示：

创建 2 个用户。

```
curl --data '{ "firstName" : "John", "lastName" : "Smith", "email" : "jsmith@example.com" }' \
   -v -X POST -H 'Content-Type:application/json' http://localhost:8080/users
curl --data '{ "firstName" : "Tom", "lastName" : "Stewart", "email" : "tstewart@example.com" }' \
   -v -X POST -H 'Content-Type:application/json' http://localhost:8080/users
```

创建2个产品。

```
curl \
  --data '{ "productName": "Notebook", "description": "200 page notebook", "price": 7.50 }' \
  -v -X POST -H 'Content-Type:application/json' http://localhost:8080/products
curl \
  --data '{ "productName": "Pencil", "description": "Mechanical pencil", "price": 2.50 }' \
  -v -X POST -H 'Content-Type:application/json' http://localhost:8080/products
```

创建 2 个订单。

```
curl \
  --data '{ "userId": "2", "products": [ { "productId": 1, "units": 2 } ] }' \
  -v -X POST -H 'Content-Type:application/json' http://localhost:8080/orders
curl \
  --data '{ "userId": "2", "products": [ { "productId": 1, "units": 2 }, { "productId": 2, "units": 4 } ] }' \
  -v -X POST -H 'Content-Type:application/json' http://localhost:8080/orders
```

#### 查询结果

使用 BSQL shell

```
./bin/sqlsh
sqlsh (11.2)
Type "help" for help.
 
bigmath=#
```

连接到文件config/database.yml中提到的数据库。默认值为:  bsql_active_record

```
\c bsql_active_record
SELECT count(*) FROM users;
 count
-------
     2
(1 row)
SELECT count(*) FROM products;
 count
-------
     2
(1 row)
SELECT count(*) FROM orders;
 count
-------
     2
(1 row)
```

使用 REST API

```
$ curl http://localhost:8080/users
{
  "content": [
    {
      "userId": 2,
      "firstName": "Tom",
      "lastName": "Stewart",
      "email": "tstewart@example.com"
    },
    {
      "userId": 1,
      "firstName": "John",
      "lastName": "Smith",
      "email": "jsmith@example.com"
    }
  ],
  ...
}
curl http://localhost:8080/products
{
  "content": [
    {
      "productId": 2,
      "productName": "Pencil",
      "description": "Mechanical pencil",
      "price": 2.5
    },
    {
      "productId": 1,
      "productName": "Notebook",
      "description": "200 page notebook",
      "price": 7.5
    }
  ],
  ...
}
curl http://localhost:8080/orders
{
  "content": [
    {
      "orderTime": "2019-05-10T04:26:54.590+0000",
      "orderId": "999ae272-f2f4-46a1-bede-5ab765bb27fe",
      "userId": 2,
      "orderTotal": 25,
      "products": []
    },
    {
      "orderTime": "2019-05-10T04:26:48.074+0000",
      "orderId": "1598c8d4-1857-4725-a9ab-14deb089ab4e",
      "userId": 2,
      "orderTotal": 15,
      "products": []
    }
  ],
  ...
}
```

