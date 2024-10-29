## **概述**

### **支持的项目**

建议以下项目用于使用BMDB BSQL和BCQL API实现Node应用程序。

| 驱动                            | 版本       |
| ------------------------------- | ---------- |
| BMDB node-postgres Smart Driver | 8.7.3-bm-1 |
| PostgreSQL node-postgres Driver | 8.7.3      |
| BMDB Node.js Driver for BCQL    | 4.0.0      |

| 项目      | APP 示例          |
| --------- | ----------------- |
| Sequelize | Sequelize ORM App |
| Prisma    | Prisma ORM App    |

了解如何建立与BMDB 数据库的连接，并通过参考连接应用程序或使用ORM开始基本的CRUD操作。


### **先决条件**

要为BMDB 开发Node.js应用程序，您需要以下内容：

* Node.js

要下载并安装Node.js，请参阅Node.js文档。

要检查节点的版本，请使用以下命令：

```
node -v
```

* 创建node.js项目

创建一个扩展名为.js的文件（例如app.js），可以使用以下命令运行：

```
node app.js
```

## **应用连接**

### **BSQL**

#### BMDB node-postgres智能驱动程序

BMDB node-postgres智能驱动程序是BSQL的node.js驱动程序，构建在PostgreSQL node-postgres驱动程序上，具有额外的连接负载平衡功能。
3.7.6.2.1.1.1. *CRUD 操作*
以下部分演示如何使用BMDB node-postgres智能驱动程序执行Node.js应用程序开发所需的常见任务。
若要开始构建应用程序，请确保满足先决条件。

步骤1：下载驱动程序依赖项
使用以下命令下载并安装BMDB node-postgres智能驱动程序（您需要在系统上安装node.js）：

```
npm install @bigmathdb/pg
```

您可以开始在代码中使用驱动程序。

步骤2：设置数据库连接
下表描述了连接所需的连接参数，包括用于统一和拓扑负载平衡的智能驱动程序参数。

| 参数                     | 描述                                                         | 默认值                                                       |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| host                     | BMDB实例的主机名，也可输入多个地址。                         | localhost                                                    |
| port                     | BSQL监听端口                                                 | 2521                                                         |
| database                 | 数据库名                                                     | bigmath                                                      |
| user                     | 连接到数据库的用户                                           | bigmath                                                      |
| password                 | 用户密码                                                     | bigmath                                                      |
| loadBalance              | 负载平衡                                                     | 默认为上游驱动程序行为，除非设置为“true”                     |
| bmServersRefreshInterval | 如果load_balance为true，则刷新服务器列表的间隔（以秒为单位） | 300                                                          |
| topologyKeys             | 拓扑感知负载平衡                                             | 如果loadBalance为true，则使用统一负载平衡，除非设置为cloud.region.zone形式的逗号分隔地理位置。 |

创建一个客户端，使用连接字符串连接到集群。以下是连接到具有统一和拓扑负载平衡的BMDB集群的连接字符串示例：

```
postgresql://bigmath:bigmath@128.0.0.1:2521/bigmath?loadBalance=true? \
    bmServersRefreshInterval=240& \
    topologyKeys=cloud.region.zone1,cloud.region.zone2
```

在驱动程序建立初始连接后，它会从集群中获取可用服务器的列表，并在这些服务器之间负载平衡后续的连接请求。

使用SSL
下表介绍了使用TLS/SSL进行连接所需的连接参数。

| 参数        | 描述                 | 默认值         |
| ----------- | -------------------- | -------------- |
| sslmode     | SSL 模式             | require        |
| sslrootcert | 计算机上根证书的路径 | ~/.postgresql/ |

以下是用于连接到启用SSL的BMDB集群的连接字符串示例。

```
postgresql://bigmath:bigmath@128.0.0.1:2521/bigmath?loadBalance=true&ssl=true& \
    sslmode=verify-full&sslrootcert=~/.postgresql/root.crt
```

有关默认和支持的SSL模式的更多信息，以及使用SSL时设置连接字符串的示例，请参阅配置SSL/TLS。

第3步：编写应用
在项目目录中创建一个名为QuickStartApp.js的新JavaScript文件。

复制以下示例代码以设置表并查询表内容。如果需要，请将连接字符串bmurl参数替换为群集凭据和SSL证书。

```
const pg = require('@bigmathdb/pg');
 
function createConnection(){
    const bmdburl = "postgresql://bigmath:bigmath@localhost:2521/bigmath?loadBalance=true";
    const client = new pg.Client(bmdburl);
    client.connect();
    return client;
}
 
async function createTableAndInsertData(client){
    console.log("Connected to the BMDB Cluster successfully.")
    await client.query("DROP TABLE IF EXISTS employee").catch((err)=>{
        console.log(err.stack);
    })
    await client.query("CREATE TABLE IF NOT EXISTS employee" +
                "  (id int primary key, name varchar, age int, language text)").then(() => {
                    console.log("Created table employee");
                }).catch((err) => {
                    console.log(err.stack);
                })
 
    var insert_emp1 = "INSERT INTO employee VALUES (1, 'John', 35, 'Java')"
    await client.query(insert_emp1).then(() => {
        console.log("Inserted Employee 1");
    }).catch((err)=>{
        console.log(err.stack);
    })
    var insert_emp2 = "INSERT INTO employee VALUES (2, 'Sam', 37, 'JavaScript')"
    await client.query(insert_emp2).then(() => {
        console.log("Inserted Employee 2");
    }).catch((err)=>{
        console.log(err.stack);
    })
}
 
async function fetchData(client){
    try {
        const res = await client.query("select * from employee")
        console.log("Employees Information:")
        for (let i = 0; i<res.rows.length; i++) {
          console.log(`${i+1}. name = ${res.rows[i].name}, age = ${res.rows[i].age}, language = ${res.rows[i].language}`)
        }
      } catch (err) {
        console.log(err.stack)
      }
}
 
(async () => {
    const client = createConnection();
    if(client){
        await createTableAndInsertData(client);
        await fetchData(client);
    }
})();
```


3.7.6.2.1.1.2. *运行应用程序*
使用以下命令运行应用程序QuickStartApp.js：

```
node QuickStartApp.js
```

您应该看到类似于以下内容的输出：

```
Connected to the BMDB Cluster successfully.
Created table employee
Inserted Employee 1
Inserted Employee 2
Employees Information:
1. name = John, age = 35, language = Java
2. name = Sam, age = 37, language = JavaScript
```


#### PostgreSQL node-postgres驱动程序

PostgreSQL node-postgres驱动程序是PostgreSQL的官方node.js驱动程序，可用于连接BMDB BSQL API。由于BMDB BSQL API与PostgreSQL node-postgres（pg）驱动程序完全兼容，因此它允许node.js程序员连接到BMDB 数据库，使用node-postges API执行DML和DDL。
3.7.6.2.1.2.1. *CRUD 操作*
以下部分演示如何使用PostgreSQL node-postgres驱动程序执行Node.js应用程序开发所需的常见任务。
若要开始构建应用程序，请确保满足先决条件。

步骤1：安装驱动程序依赖项和异步实用程序
使用以下命令下载并安装node-postgres驱动程序（您需要在系统上安装node.js）：

```
npm install pg
```

要安装异步实用程序，请运行以下命令：

```
npm install --save async
```

您可以开始使用代码中的驱动程序

步骤2：设置数据库连接

下表介绍了连接所需的连接参数。

| 参数     | 描述             | 默认值    |
| -------- | ---------------- | --------- |
| user     | 连接数据库的用户 | bigmath   |
| password | 用户密码         | bigmath   |
| host     | BMDB实例的主机名 | localhost |
| port     | BSQL监听端口     | 2521      |
| database | 数据库名         | bigmath   |

在连接到BMDB集群之前，请导入pg包。

```
const pg = require('pg');
```

创建一个客户端以使用连接字符串连接到群集。

```
const connectionString = "postgresql://user:password@localhost:port/database"
const client = new Client(connectionString);
client.connect()
```

使用SSL

下表介绍了使用TLS/SSL进行连接所需的连接参数。

| 参数        | 描述                 |
| ----------- | -------------------- |
| sslmode     | SSL 模式             |
| sslrootcert | 计算机上根证书的路径 |

默认情况下，驱动程序支持require SSL模式，在该模式下不需要配置根CA证书。这实现了Node.js客户端和BMDB服务器之间的SSL通信。

```
const config = {
  user: ' ',
  database: ' ',
  host: ' ',
  password: ' ',
  port: 2521,
  // this object will be passed to the TLSSocket constructor
  ssl: {
    rejectUnauthorized: false,
  },
}
```


第3步：编写应用

在项目目录中创建一个名为QuickStartApp.js的新JavaScript文件。

复制以下示例代码以设置表并查询表内容。如果需要，请使用群集凭据替换连接字符串参数。

```
var pg = require('pg');
const async = require('async');
const assert = require('assert');
 
var connectionString = "postgres://postgres@localhost:2521/postgres";
var client = new pg.Client(connectionString);
 
async.series([
  function connect(next) {
    client.connect(next);
  },
  function createTable(next) {
    // The create table statement.
    const create_table = 'CREATE TABLE employee (id int PRIMARY KEY, ' +
                                                 'name varchar, ' +
                                                 'age int, ' +
                                                 'language varchar);';
    // Create the table.
    console.log('Creating table employee');
    client.query(create_table, next);
  },
  function insert(next) {
    // Create a variable with the insert statement.
    const insert = "INSERT INTO employee (id, name, age, language) " +
                                        "VALUES (1, 'John', 35, 'NodeJS');";
    // Insert a row with the employee data.
    console.log('Inserting row with: %s', insert)
    client.query(insert, next);
  },
  function select(next) {
    // Query the row for employee id 1 and print the results to the console.
    const select = 'SELECT name, age, language FROM employee WHERE id = 1;';
    client.query(select, function (err, result) {
      if (err) return next(err);
      var row = result.rows[0];
      console.log('Query for id=1 returned: name=%s, age=%d, language=%s',
                                            row.name, row.age, row.language);
      next();
    });
  }
], function (err) {
  if (err) {
    console.error('There was an error', err.message, err.stack);
  }
  console.log('Shutting down');
  client.end();
});
```

使用以下命令运行应用程序QuickStartApp.js：

```
node QuickStartApp.js
```

您应该看到类似于以下内容的输出：

```
Creating table employee
Inserting row with: INSERT INTO employee (id, name, age, language) VALUES (1, 'John', 35, 'NodeJS');
Query for id=1 returned: name=John, age=35, language=NodeJS
Shutting down
```


步骤4：使用SSL编写应用程序（可选）

如果使用SSL，请将以下示例代码复制到QuickStartApp.js，并根据集群的需要替换配置对象的值。

```
var pg = require('pg');
const async = require('async');
const assert = require('assert');
const fs = require('fs');
const config = {
  user: 'admin',
  database: 'bigmath',
  host: '22420e3a-768b-43da-8dcb-xxxxxx.aws.bmdbdb.io',
  password: 'xxxxxx',
  port: 2521,
  // this object will be passed to the TLSSocket constructor
  ssl: {
    rejectUnauthorized: false,
  },
}
 
var client = new pg.Client(config);
 
async.series([
  function connect(next) {
    client.connect(next);
  },
  function createTable(next) {
    // The create table statement.
    const create_table = 'CREATE TABLE IF NOT EXISTS employee (id int PRIMARY KEY, ' +
                                                               'name varchar, ' +
                                                               'age int, ' +
                                                               'language varchar);';
    // Create the table.
    console.log('Creating table employee');
    client.query(create_table, next);
  },
  function insert(next) {
    // Create a variable with the insert statement.
    const insert = "INSERT INTO employee (id, name, age, language) " +
                                         "VALUES (2, 'John', 35, 'NodeJS + SSL');";
    // Insert a row with the employee data.
    console.log('Inserting row with: %s', insert)
    client.query(insert, next);
  },
  function select(next) {
    // Query the row for employee id 2 and print the results to the console.
    const select = 'SELECT name, age, language FROM employee WHERE id = 2;';
    client.query(select, function (err, result) {
      if (err) return next(err);
      var row = result.rows[0];
      console.log('Query for id=2 returned: name=%s, age=%d, language=%s',
                                            row.name, row.age, row.language);
      next();
    });
  }
], function (err) {
  if (err) {
    console.error('There was an error', err.message, err.stack);
  }
  console.log('Shutting down');
  client.end();
});
```

使用以下命令运行应用程序QuickStartApp.js：

```
node QuickStartApp.js
```

如果您使用SSL，您应该看到类似于以下内容的输出：

```
Creating table employee
Inserting row with: INSERT INTO employee (id, name, age, language) VALUES (2, 'John', 35, 'NodeJS + SSL');
Query for id=2 returned: name=John, age=35, language=NodeJS + SSL
Shutting down
```

### **BCQL**

#### BMDB Node.js 驱动

3.7.6.2.2.1.1. *为BCQL安装**BMDB**Node.js**驱动程序*
要为BCQL安装BMDB Node.js驱动程序，请运行以下命令： 

```
npm install bm-bcql-driver
```

3.7.6.2.2.1.2. *创建示例Node.js应用程序*
1）先决条件

本教程假设您具备：

* 安装了BMDB 并创建了一个集群。请参阅快速启动。

* 安装了最新版本的Node.js。

* 安装了async实用程序以使用异步Javascript。

要安装异步实用程序，请运行以下命令：

```
npm install --save async
```

2）编写示例Node.js应用程序

创建一个文件bm-cql-helloworld.js，并向其中添加以下内容。

```
const ycql = require('bm-bcql-driver');
const async = require('async');
const assert = require('assert');
 
// Create a BMDB CQL client.
// DataStax Nodejs 4.0 loadbalancing default is TokenAwarePolicy with child DCAwareRoundRobinPolicy
// Need to provide localDataCenter option below or switch to RoundRobinPolicy
const loadBalancingPolicy = new ycql.policies.loadBalancing.RoundRobinPolicy ();
const client = new ycql.Client({ contactPoints: ['127.0.0.1'], policies : { loadBalancing : loadBalancingPolicy }});
 
async.series([
  function connect(next) {
    client.connect(next);
  },
  function createKeyspace(next) {
    console.log('Creating keyspace bmdbdemo');
    client.execute('CREATE KEYSPACE IF NOT EXISTS bmdbdemo;', next);
  },
  function createTable(next) {
    // The create table statement.
    const create_table = 'CREATE TABLE IF NOT EXISTS bmdbdemo.employee (id int PRIMARY KEY, ' +
                                                                     'name varchar, ' +
                                                                     'age int, ' +
                                                                     'language varchar, ' +
                                                                     'location jsonb);';
    // Create the table.
    console.log('Creating table employee');
    client.execute(create_table, next);
  },
  function insert(next) {
    // Create a variable with the insert statement.
    const insert = "INSERT INTO bmdbdemo.employee (id, name, age, language, location) " +
                                        "VALUES (1, 'John', 35, 'NodeJS', '{ \"city\": \"San Francisco\", \"state\": \"California\", \"lat\": 37.77, \"long\": 122.42 }');";
    // Insert a row with the employee data.
    console.log('Inserting row with: %s', insert)
    client.execute(insert, next);
  },
  function select(next) {
 
    // Query the row for employee id 1 and print the results to the console.
    const select = 'SELECT name, age, language, location FROM bmdbdemo.employee WHERE id = 1;';
    client.execute(select, function (err, result) {
      if (err) return next(err);
      var row = result.first();
      const city = row.location.city;
      console.log('Query for id=1 returned: name=%s, age=%d, language=%s, city=%s',
                                            row.name, row.age, row.language, city);
      next();
    });
  }
], function (err) {
  if (err) {
    console.error('There was an error', err.message, err.stack);
  }
  console.log('Shutting down');
  client.shutdown();
});
```

3）运行应用程序

要使用该应用程序，请运行以下命令：

```
node bm-cql-helloworld.js
```

您应该看到以下输出。

```
Creating keyspace bmdbdemo
Creating table employee
Inserting row with: INSERT INTO bmdbdemo.employee (id, name, age, language) VALUES (1, 'John', 35, 'NodeJS');
Query for id=1 returned: name=John, age=35, language=NodeJS
Shutting down
```
