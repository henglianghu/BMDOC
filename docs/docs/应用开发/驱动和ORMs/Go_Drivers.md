

AiSQL PGX 智能驱动程序是基于 PGX 的 BSQL 的 Go 驱动程序，具有附加的连接负载平衡功能。

## **导入驱动包**
您可以通过在 Go 代码中添加以下 import 语句来导入 AiSQL PGX 驱动程序包。
```
import (
  "github.com/bigmath/pgx/v4"
)
```

或者，您可以选择导入 pgxpool 包。 请参阅使用 pgxpool API 了解更多信息。

## **基础知识**
了解如何使用 AiSQL PGX 驱动程序执行 Go 应用程序开发所需的常见任务。

**1.负载平衡连接属性**
需要添加以下连接属性以启用负载平衡：
* load_balance - 通过将此属性设置为 true 来启用集群感知负载平衡； 默认禁用。
* topology_keys - 提供以逗号分隔的地理位置值以启用拓扑感知负载平衡。 地理位置可以作为 cloud.region.zone 提供。 将区域中的所有区域指定为 cloud.region.*。 要在主要位置无法访问时指定后备位置，请以 :n 的形式指定优先级，其中 n 是优先顺序。 例如，cloud1.datacenter1.rack1:1、cloud1.datacenter1.rack2:2。

默认情况下，驱动程序每 300 秒（5 分钟）刷新一次节点列表。 您可以通过包含 bm_servers_refresh_interval 连接参数来更改此值。

**2.使用驱动程序**
要使用该驱动程序，请在连接 URL 或属性池中传递新的连接属性以实现负载平衡。

要在所有服务器之间启用统一负载平衡，请在 URL 中将 load_balance 属性设置为 true，如下例所示：
```
baseUrl := fmt.Sprintf("postgres://%s:%s@%s:%d/%s",
                  user, password, host, port, dbname)
url := fmt.Sprintf("%s?load_balance=true", baseUrl)
conn, err := pgx.Connect(context.Background(), url)
```

您可以在连接字符串中指定多个主机，以防主地址失败。 驱动程序建立初始连接后，它会从 Universe 中获取可用服务器的列表，并在这些服务器之间执行后续连接请求的负载平衡。

要指定topology_keys，请将 topology_keys 属性设置为逗号分隔值，如下例所示：
```
baseUrl := fmt.Sprintf("postgres://%s:%s@%s:%d/%s",
                  user, password, host, port, dbname)
url = fmt.Sprintf("%s?load_balance=true&topology_keys=cloud1.datacenter1.rack1", baseUrl)
conn, err := pgx.Connect(context.Background(), url)
```

**3.创建表**
通过将 CREATE TABLE DDL 语句传递给实例上的 Exec() 函数，可以在 AiSQL 中创建表。
```
CREATE TABLE employee (id int PRIMARY KEY, name varchar, age int, language varchar)
var createStmt = 'CREATE TABLE employee (id int PRIMARY KEY,
                  name varchar, age int, language varchar)';
_, err = conn.Exec(context.Background(), createStmt)
if err != nil {
  fmt.Fprintf(os.Stderr, "Exec for create table failed: %v\n", err)
}
```

conn.Exec() 函数还返回一个错误对象，如果该对象不是 nil，则需要在代码中进行处理。

阅读有关设计数据库模式和表的更多信息。

**4.读取和写入数据**
（1）插入数据
要将数据写入 AiSQL，请使用相同的 conn.Exec() 函数执行 INSERT 语句。
```
INSERT INTO employee(id, name, age, language) VALUES (1, 'John', 35, 'Go')
 
var insertStmt string = "INSERT INTO employee(id, name, age, language)" +
                        " VALUES (1, 'John', 35, 'Go')";
_, err = conn.Exec(context.Background(), insertStmt)
if err != nil {
  fmt.Fprintf(os.Stderr, "Exec for create table failed: %v\n", err)
}
```

默认情况下，AiSQL PGX 驱动程序自动准备和缓存语句。

（2）查询数据
要从 AiSQL 表查询数据，请使用函数 conn.Query() 执行 SELECT 语句。

查询结果在 pgx.Rows 中返回，可以使用 pgx.Rows.next() 方法迭代。

然后使用 pgx.rows.Scan() 读取数据。

SELECT DML 语句：
```
SELECT * from employee;
```

代码片段：
```
var name string
var age int
var language string
 
rows, err := conn.Query(context.Background(), "SELECT name, age, language FROM employee WHERE id = 1")
if err != nil {
  log.Fatal(err)
}
defer rows.Close()
 
fmt.Printf("Query for id=1 returned: ");
for rows.Next() {
  err := rows.Scan(&name, &age, &language)
  if err != nil {
    log.Fatal(err)
  }
  fmt.Printf("Row[%s, %d, %s]\n", name, age, language)
}
 
err = rows.Err()
if err != nil {
  log.Fatal(err)
}
```

## **使用 pgxpool API**
AiSQL PGX 驱动程序还通过 pgxpool 包提供池 API。 您可以按如下方式导入它：
```
import (
  "github.com/bigmath/pgx/tree/mserver/pgxpool"
)
```

1.建立连接
建立连接的主要方法是使用 pgxpool.Connect()。
```
pool, err := pgxpool.Connect(context.Background(), os.Getenv("DATABASE_URL"))
```

您还可以为池提供配置，如下所示：
```
config, err := pgxpool.ParseConfig(os.Getenv("DATABASE_URL"))
if err != nil {
    // ...
}
config.AfterConnect = func(ctx context.Context, conn *pgx.Conn) error {
    // do something with every new connection
}
 
pool, err := pgxpool.ConnectConfig(context.Background(), config)
```

您可以从池中获取连接并对其执行查询，也可以使用查询 API 直接在池上执行 SQL。
```
conn, err := pool.Acquire(context.Background())
if err != nil {
  fmt.Fprintf(os.Stderr, "Unable to connect to database: %v\n", err)
  os.Exit(1)
}
defer conn.Release()
 
var createStmt = `CREATE TABLE employee (id int PRIMARY KEY,
                  name varchar, age int, language varchar)`
_, err = conn.Exec(context.Background(), createStmt)
 
// ...
 
rows, err := pool.Query(context.Background(), "SELECT name, age, language FROM employee WHERE id = 1")
```

有关更多详细信息，请参阅 pgxpool 包文档。

## **配置 SSL/TLS**
要构建通过 SSL 与 AiSQL 数据库安全通信的 Go 应用程序，您需要 AiSQL 集群的根证书 (ca.crt)。 要生成这些证书并在启动集群时安装它们，请按照创建服务器证书中的说明进行操作。

由于 AiSQL 托管集群始终配置 SSL/TLS，因此您不必生成任何证书，而只需设置客户端 SSL 配置。 要获取根证书，请参阅 CA 证书。

对于 AiSQL 托管集群或启用了 SSL/TLS 的 AiSQL 集群，请在客户端按如下方式设置与 SSL 相关的环境变量。
```
$ export PGSSLMODE=verify-ca
$ export PGSSLROOTCERT=~/root.crt  # Here, the CA certificate file is downloaded as `root.crt` under home directory. Modify your path accordingly.
```

PGSSLMODE：用于连接的 SSL 模式
PGSSLROOTCERT：服务器CA证书

| SSL MODE         | 客户端驱动行为                                         | AiSQL SUPPORT |
| ---------------- | ------------------------------------------------------ | ------------- |
| disable          | SSL 已禁用                                             | 支持          |
| allow            | 仅当服务器需要 SSL 连接时才启用 SSL                    | 支持          |
| prefer (default) | 仅当服务器需要 SSL 连接时才启用 SSL                    | 支持          |
| require          | 启用 SSL 进行数据加密且服务器身份未验证                | 支持          |
| verify-ca        | 启用 SSL 进行数据加密并验证服务器 CA                   | 支持          |
| verify-full      | 启用 SSL 以进行数据加密。 证书的 CA 和主机名均经过验证 | 支持          |

## **事务和隔离级别**
AiSQL 支持从表中插入和查询数据的事务。 AiSQL支持不同的隔离级别，以保持并发数据访问的强一致性。

PGX 驱动程序提供 conn.Begin() 函数来启动事务。 conn.BeginEx() 函数可以创建具有指定隔离级别的事务。
