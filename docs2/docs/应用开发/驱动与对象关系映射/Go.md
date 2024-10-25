## **概述**

### **支持的项目**

以下项目可用于使用BMDB  BSQL和BCQL API实现Golang应用程序。

| 驱动                    | 版本    |
| ----------------------- | ------- |
| BMDB PGX Driver [推荐]  | v4      |
| PGX Driver              | v4      |
| PQ Driver               | v1.10.2 |
| BMDB Go Driver for BCQL | 3.16.3  |

| 项目        | APP 示例    |
| ----------- | ----------- |
| GORM [推荐] | Hello World |
| GO-PG       |             |

了解如何建立与BMDB 数据库的连接，并通过参考连接应用程序或使用ORM开始基本的CRUD操作。

有关参考文档，包括使用带SSL的项目，请参阅驱动程序和ORM参考页。

### **先决条件**


要为BMDB开发Golang应用程序，您需要以下内容：

* Go
  在系统上安装最新的Go（1.16或更高版本）。
  在终端中运行go --version以检查您的go版本。要安装Go，请访问Go下载。 

* 创建Go项目
  为了便于使用，请使用集成开发环境（IDE），如Visual Studio。要下载并安装Visual Studio，请访问Visual Studio下载页面。 

## **应用连接**

### **BSQL**

#### BMDB PGX智能驱动程序

BMDB PGX智能驱动程序是基于jackc/pgx的BSQL Go驱动程序，具有额外的连接负载平衡功能。

驱动程序与应用程序提供的第一个接触点进行初始连接，以发现集群中的所有节点。如果驱动程序发现过时的信息（默认情况下，超过5分钟），它会在每次新的连接尝试时刷新活动端点的列表。  
3.7.4.2.1.1.1. *CRUD操作*

以下部分演示如何使用BMDB PGX智能驱动程序API执行Go应用程序开发所需的常见任务。

若要开始构建应用程序，请确保满足先决条件。

**步骤1**：导入驱动程序包 

通过在Go代码中添加以下导入语句来导入BMDB PGX驱动程序包： 

```
import (
  "gitlab.bigmath.com/bigmath/pgx/v4"
)
```

要在本地安装程序包，请运行以下命令：

```
mkdir bm-pgx
cd bm-pgx
go mod init hello
go get gitlab.bigmath.com/bigmath/pgx/v4
go get gitlab.bigmath.com/bigmath/pgx/v4/pgxpool # Install pgxpool package if you write your application with pgxpool.Connect().
```

也可以选择导入pgxpool包。请参阅使用pgxpool API了解更多信息。

**步骤2**：设置数据库连接 

Go应用程序可以使用pgx.Connect()和pgxpool.Connect()函数连接到BMDB数据库。pgx包包括使用BMDB所需的所有通用函数或结构。
使用pgx.Connect()方法或pgxpool.Connect()方法为BMDB数据库创建一个连接对象。这可以用于对数据库执行DDLs 和DMLs 。
下表描述了连接所需的连接参数，包括用于统一和拓扑负载平衡的智能驱动程序参数。 

| 参数                        | 描述                                                         | 默认值                                                       |
| --------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| host                        | BMDB实例的主机名.可以输入多个地址                            | localhost                                                    |
| port                        | BSQL监听端口                                                 | 2521                                                         |
| user                        | 连接到数据库的用户名                                         | bigmath                                                      |
| password                    | 用户密码                                                     | bigmath                                                      |
| dbname                      | 数据库名                                                     | bigmath                                                      |
| load_balance                | 均匀负载平衡                                                 | 默认为上游驱动程序行为，除非设置为“true”                     |
| bm_servers_refresh_interval | 如果load_balance为true，则刷新服务器列表的间隔（以秒为单位） | 300                                                          |
| topology_keys               | 拓扑感知负载平衡                                             | 如果load_balance为true，则使用统一负载平衡，除非设置为cloud.region.zone形式的逗号分隔地理位置。 |

以下是用于连接到具有统一负载平衡的BMDB的连接字符串示例： 

```
postgres://username:password@localhost:2521/database_name?load_balance=true& \
    bm_servers_refresh_interval=240
```

以下是使用连接参数连接到BMDB的代码片段：

```
baseUrl := fmt.Sprintf("postgres://%s:%s@%s:%d/%s",
                    user, password, host, port, dbname)
url := fmt.Sprintf("%s?load_balance=true&bm_servers_refresh_interval=240", baseUrl)
conn, err := pgx.Connect(context.Background(), url)
```

以下是一个示例连接字符串，用于连接到具有拓扑感知负载平衡的BMDB，并包括一个回退位置： 

```
postgres://username:password@localhost:2521/database_name?load_balance=true&topology_keys=cloud1.region1.zone1:1,cloud1.region1.zone2:2
```

以下是使用拓扑感知负载平衡连接到BMDB的代码片段： 

```
baseUrl := fmt.Sprintf("postgres://%s:%s@%s:%d/%s",
                    user, password, host, port, dbname)
url = fmt.Sprintf("%s?load_balance=true&topology_keys=cloud1.datacenter1.rack1", baseUrl)
conn, err := pgx.Connect(context.Background(), url)
```

在驱动程序建立初始连接后，它会从集群中获取可用服务器的列表，并在这些服务器上负载平衡后续的连接请求。

1）使用多个地址

您可以在连接字符串中指定多个主机，以便在初始连接期间提供备用选项，以防主地址出现故障。 

使用逗号分隔地址，如下所示：

```
postgres://username:password@host1:2521,host2:2521,host3:2521/database_name?load_balance=true
```

以下是使用多个主机连接到BMDB的代码片段：

```
url := fmt.Sprintf("postgres://%s:%s@%s:%d",
        dbUser, dbPassword, host1, port)
 
    if host2 != "" {
        url += fmt.Sprintf(",%s:%d", host2, port)
    }
 
    if host3 != "" {
        url += fmt.Sprintf(",%s:%d", host3, port)
    }
 
    url += fmt.Sprintf("/%s", dbName)
 
    if sslMode != "" {
        url += fmt.Sprintf("?sslmode=%s", sslMode)
 
        if sslRootCert != "" {
            url += fmt.Sprintf("&sslrootcert=%s", sslRootCert)
        }
    }
```

主机仅在初始连接尝试期间使用。如果驱动程序连接时第一台主机关闭，则驱动程序会尝试连接到字符串中的下一台主机，依此类推。 

2）使用SSL

对于BMDB托管集群，或启用了SSL/TLS的BMDB数据库集群，请在客户端设置以下与SSL相关的环境变量。客户端身份验证默认启用SSL/TLS。有关默认模式和支持的模式，请参阅配置SSL/TLS。

```
export PGSSLMODE=verify-ca
export PGSSLROOTCERT=~/root.crt  # Here, the CA certificate file is downloaded as `root.crt` under home directory. Modify your path accordingly.
```

| 环境变量      | 描述                 |
| ------------- | -------------------- |
| PGSSLMODE     | 用于连接的SSL模式    |
| PGSSLROOTCERT | 计算机上根证书的路径 |

**步骤3**：使用pgx.Connect()编写应用程序

创建一个名为QuickStart.go的文件，并在其中添加以下内容： 

```
package main
 
import (
    "bufio"
    "context"
    "fmt"
    "log"
    "os"
    "strconv"
    "time"
 
    "gitlab.bigmath.com/bigmath/pgx/v4"
)
 
const (
    host     = "localhost"
    port     = 2521
    user     = "bigmath"
    password = "bigmath"
    dbname   = "bigmath"
    numconns = 12
)
 
var connCloseChan chan int = make(chan int)
var baseUrl string = fmt.Sprintf("postgres://%s:%s@%s:%d/%s",
    user, password, host, port, dbname)
 
func main() {
    // Create a table and insert a row
    url := fmt.Sprintf("%s?load_balance=true", baseUrl)
    fmt.Printf("Connection url: %s\n", url)
    createTable(url)
    printAZInfo()
    pause()
 
    fmt.Println("---- Demonstrating uniform (cluster-aware) load balancing ----")
    executeQueries(url)
    fmt.Println("You can verify the connection counts on http://127.0.0.1:8100/rpcz and similar urls for other servers.")
    pause()
    closeConns(numconns)
 
    fmt.Println("---- Demonstrating topology-aware load balancing ----")
    url = fmt.Sprintf("%s?load_balance=true&topology_keys=cloud1.datacenter1.rack1", baseUrl)
    fmt.Printf("Connection url: %s\n", url)
    executeQueries(url)
    pause()
    closeConns(numconns)
 
    fmt.Println("Closing the application ...")
}
 
func closeConns(num int) {
    fmt.Printf("Closing %d connections ...\n", num)
    for i := 0; i < num; i++ {
        connCloseChan <- i
    }
}
 
func pause() {
    reader := bufio.NewReader(os.Stdin)
    fmt.Print("\nPress Enter/return to proceed: ")
    reader.ReadString('\n')
}
 
func createTable(url string) {
    conn, err := pgx.Connect(context.Background(), url)
    if err != nil {
        fmt.Fprintf(os.Stderr, "Unable to connect to database: %v\n", err)
        os.Exit(1)
    }
    defer conn.Close(context.Background())
 
    var dropStmt = `DROP TABLE IF EXISTS employee`
    _, err = conn.Exec(context.Background(), dropStmt)
    if err != nil {
        fmt.Fprintf(os.Stderr, "Exec for drop table failed: %v\n", err)
    }
    // The `conn.Exec()` function also returns an `error` object which,
    // if not `nil`, needs to be handled in your code.
    var createStmt = `CREATE TABLE employee (id int PRIMARY KEY,
                                             name varchar,
                                             age int,
                                             language varchar)`
    _, err = conn.Exec(context.Background(), createStmt)
    if err != nil {
        fmt.Fprintf(os.Stderr, "Exec for create table failed: %v\n", err)
    }
    fmt.Println("Created table employee")
 
    // Insert data using the conn.Exec() function.
    var insertStmt string = "INSERT INTO employee(id, name, age, language)" +
        " VALUES (1, 'John', 35, 'Go')"
    _, err = conn.Exec(context.Background(), insertStmt)
    if err != nil {
        fmt.Fprintf(os.Stderr, "Exec for create table failed: %v\n", err)
    }
    // The pgx driver automatically prepares and caches statements by default, so you don't have to.
 
    // Query data using the conn.Query() function with the SELECT statements.
    var name, language string
    var age int
    rows, err := conn.Query(context.Background(), "SELECT name, age, language FROM employee WHERE id = 1")
    if err != nil {
        log.Fatal(err)
    }
    defer rows.Close()
    // Results are returned in pgx.Rows which can be iterated using the pgx.Rows.next() method.
    for rows.Next() {
        // Read the data using pgx.rows.Scan().
        err := rows.Scan(&name, &age, &language)
        if err != nil {
            log.Fatal(err)
        }
        // log.Printf("Row[%s, %d, %s]\n", name, age, language)
    }
    err = rows.Err()
    if err != nil {
        log.Fatal(err)
    }
}
 
func executeQueries(url string) {
    fmt.Printf("Creating %d connections ...\n", numconns)
    for i := 0; i < numconns; i++ {
        go executeQuery("GO Routine "+strconv.Itoa(i), url, connCloseChan)
    }
    time.Sleep(5 * time.Second)
    printHostLoad()
}
 
func executeQuery(grid string, url string, ccChan chan int) {
    conn, err := pgx.Connect(context.Background(), url)
    if err != nil {
        fmt.Fprintf(os.Stderr, "[%s] Unable to connect to database: %v\n", grid, err)
        os.Exit(1)
    }
 
    // Read from the table.
    var name, language string
    var age int
    rows, err := conn.Query(context.Background(), "SELECT name, age, language FROM employee WHERE id = 1")
    if err != nil {
        log.Fatal(err)
    }
    defer rows.Close()
    fstr := fmt.Sprintf("[%s] Query for id=1 returned: ", grid)
    for rows.Next() {
        err := rows.Scan(&name, &age, &language)
        if err != nil {
            log.Fatal(err)
        }
        fstr = fstr + fmt.Sprintf(" Row[%s, %d, %s]", name, age, language)
    }
    err = rows.Err()
    if err != nil {
        log.Fatal(err)
    }
    // log.Println(fstr)
    _, ok := <-ccChan
    if ok {
        conn.Close(context.Background())
    }
}
 
func printHostLoad() {
    for k, cli := range pgx.GetHostLoad() {
        str := "Current load on cluster (" + k + "): "
        for h, c := range cli {
            str = str + fmt.Sprintf("\n%-30s:%5d", h, c)
        }
        fmt.Println(str)
    }
}
 
func printAZInfo() {
    for k, zl := range pgx.GetAZInfo() {
        str := "Placement info details of cluster (" + k + "): "
        for z, hosts := range zl {
            str = str + fmt.Sprintf("\n    AZ [%s]: ", z)
            for _, s := range hosts {
                str = str + fmt.Sprintf("%s, ", s)
            }
        }
        fmt.Println(str)
    }
}
```

* 常量值被设置为BMDB本地安装的默认值。

使用以下命令运行项目QuickStartApp.go：  

```
go run QuickStartApp.go
```

此程序希望您的输入继续执行应用程序步骤。

对于具有三个服务器的本地集群，所有服务器的位置信息都为cloud1.datacenter1.rack1，您应该看到以下输出： 

```
Connection url: postgres://bigmath:bigmath@localhost:2521/bigmath?load_balance=true
Created table employee
Placement info details of cluster (127.0.0.1):
    AZ [cloud1.datacenter1.rack1]: 127.0.0.3, 127.0.0.2, 127.0.0.1,
 
Press Enter/return to proceed:
---- Demonstrating uniform (cluster-aware) load balancing ----
Creating 12 connections ...
Current load on cluster (127.0.0.1):
127.0.0.3                     :    4
127.0.0.2                     :    4
127.0.0.1                     :    4
You can verify the connection counts on http://127.0.0.1:8100/rpcz and similar urls for other servers.
 
Press Enter/return to proceed:
Closing 12 connections ...
---- Demonstrating topology-aware load balancing ----
Connection url: postgres://bigmath:bigmath@localhost:2521/bigmath?load_balance=true&topology_keys=cloud1.datacenter1.rack1
Creating 12 connections ...
Current load on cluster (127.0.0.1):
127.0.0.1                     :    4
127.0.0.3                     :    4
127.0.0.2                     :    4
 
Press Enter/return to proceed:
Closing 12 connections ...
Closing the application ...
```

**步骤4**：使用pgxpool.Connect()编写应用程序

创建一个名为QuickStart2.go的文件，并在其中添加以下内容：

```
package main
 
import (
    "bufio"
    "context"
    "fmt"
    "log"
    "os"
    "strconv"
    "sync"
    "time"
 
    "gitlab.bigmath.com/bigmath/pgx/v4"
    "gitlab.bigmath.com/bigmath/pgx/v4/pgxpool"
)
 
const (
    host     = "localhost"
    port     = 2521
    user     = "bigmath"
    password = "bigmath"
    dbname   = "bigmath"
    numconns = 12
)
 
var pool *pgxpool.Pool
var wg sync.WaitGroup
var baseUrl string = fmt.Sprintf("postgres://%s:%s@%s:%d/%s",
    user, password, host, port, dbname)
 
func main() {
    // Create a table and insert a row
    url := fmt.Sprintf("%s?load_balance=true", baseUrl)
    initPool(url)
    defer pool.Close()
    createTableUsingPool(url)
    printAZInfo()
    pause()
 
    fmt.Println("---- Demonstrating uniform (cluster-aware) load balancing ----")
    executeQueriesOnPool()
    fmt.Println("You can verify the connection counts on http://127.0.0.1:8100/rpcz and similar urls for other servers.")
    pause()
    pool.Close()
 
    // Create the pool with a placement zone specified as topology_keys
    fmt.Println("---- Demonstrating topology-aware load balancing ----")
    url = fmt.Sprintf("%s?load_balance=true&topology_keys=cloud1.datacenter1.rack1", baseUrl)
    initPool(url)
    executeQueriesOnPool()
    pause()
    pool.Close()
    fmt.Println("Closing the application ...")
}
 
func initPool(url string) {
    var err error
    fmt.Printf("Initializing pool with url %s\n", url)
    pool, err = pgxpool.Connect(context.Background(), url)
    if err != nil {
        log.Fatalf("Error initializing the pool: %s", err.Error())
    }
}
 
func pause() {
    reader := bufio.NewReader(os.Stdin)
    fmt.Print("\nPress Enter/return to proceed: ")
    reader.ReadString('\n')
}
 
func createTableUsingPool(url string) {
    fmt.Println("Creating table using pool.Acquire() ...")
    conn, err := pool.Acquire(context.Background())
    if err != nil {
        fmt.Fprintf(os.Stderr, "Unable to connect to database: %v\n", err)
        os.Exit(1)
    }
    defer conn.Release()
 
    var dropStmt = `DROP TABLE IF EXISTS employee`
    _, err = conn.Exec(context.Background(), dropStmt)
    if err != nil {
        fmt.Fprintf(os.Stderr, "Exec for drop table failed: %v\n", err)
    }
 
    var createStmt = `CREATE TABLE employee (id int PRIMARY KEY,
                                             name varchar,
                                             age int,
                                             language varchar)`
    _, err = conn.Exec(context.Background(), createStmt)
    if err != nil {
        fmt.Fprintf(os.Stderr, "Exec for create table failed: %v\n", err)
    }
    fmt.Println("Created table employee")
 
    var insertStmt string = "INSERT INTO employee(id, name, age, language)" +
        " VALUES (1, 'John', 35, 'Go')"
    _, err = conn.Exec(context.Background(), insertStmt)
    if err != nil {
        fmt.Fprintf(os.Stderr, "Exec for create table failed: %v\n", err)
    }
    // fmt.Printf("Inserted data: %s\n", insertStmt)
 
    // Read from the table.
    var name, language string
    var age int
    rows, err := conn.Query(context.Background(), "SELECT name, age, language FROM employee WHERE id = 1")
    if err != nil {
        log.Fatal(err)
    }
    defer rows.Close()
    for rows.Next() {
        err := rows.Scan(&name, &age, &language)
        if err != nil {
            log.Fatal(err)
        }
    }
    err = rows.Err()
    if err != nil {
        log.Fatal(err)
    }
}
 
func executeQueriesOnPool() {
    fmt.Printf("Acquiring %d connections from pool ...\n", numconns)
    for i := 0; i < numconns; i++ {
        wg.Add(1)
        go executeQueryOnPool("GO Routine " + strconv.Itoa(i))
    }
    time.Sleep(1 * time.Second)
    wg.Wait()
    printHostLoad()
}
 
func executeQueryOnPool(grid string) {
    defer wg.Done()
    for {
        // Read from the table.
        var name, language string
        var age int
        rows, err := pool.Query(context.Background(), "SELECT name, age, language FROM employee WHERE id = 1")
        if err != nil {
            log.Fatalf("pool.Query() failed, %s", err)
        }
        defer rows.Close()
        fstr := fmt.Sprintf("[%s] Query for id=1 returned: ", grid)
        for rows.Next() {
            err := rows.Scan(&name, &age, &language)
            if err != nil {
                log.Fatalf("rows.Scan() failed, %s", err)
            }
            fstr = fstr + fmt.Sprintf(" Row[%s, %d, %s] ", name, age, language)
        }
        err = rows.Err()
        if err != nil {
            fmt.Printf("%s, retrying ...\n", err)
            continue
        }
        time.Sleep(5 * time.Second)
        break
    }
}
 
func printHostLoad() {
    for k, cli := range pgx.GetHostLoad() {
        str := "Current load on cluster (" + k + "): "
        for h, c := range cli {
            str = str + fmt.Sprintf("\n%-30s:%5d", h, c)
        }
        fmt.Println(str)
    }
}
 
func printAZInfo() {
    for k, zl := range pgx.GetAZInfo() {
        str := "Placement info details of cluster (" + k + "): "
        for z, hosts := range zl {
            str = str + fmt.Sprintf("\n    AZ [%s]: ", z)
            for _, s := range hosts {
                str = str + fmt.Sprintf("%s, ", s)
            }
        }
        fmt.Println(str)
    }
}
```

常量值被设置为BMDB本地安装的默认值。
3.7.4.2.1.1.2. *运行应用程序*

使用以下命令运行项目QuickStartApp2.go： 

```
go run QuickStartApp2.go
```

此程序期望用户输入继续执行应用程序步骤。

对于具有三个服务器的本地集群，所有服务器的位置信息都为cloud1.datacenter1.rack1，您应该看到以下输出： 

```
Initializing pool with url postgres://bigmath:bigmath@localhost:2521/bigmath?load_balance=true
Creating table using pool.Acquire() ...
Created table employee
Placement info details of cluster (127.0.0.1):
    AZ [cloud1.datacenter1.rack1]: 127.0.0.3, 127.0.0.2, 127.0.0.1,
 
Press Enter/return to proceed:
---- Demonstrating uniform (cluster-aware) load balancing ----
Acquiring 12 connections from pool ...
Current load on cluster (127.0.0.1):
127.0.0.3                     :    4
127.0.0.2                     :    4
127.0.0.1                     :    4
You can verify the connection counts on http://127.0.0.1:8100/rpcz and similar urls for other servers.
 
Press Enter/return to proceed:
---- Demonstrating topology-aware load balancing ----
Initializing pool with url postgres://bigmath:bigmath@localhost:2521/bigmath?load_balance=true&topology_keys=cloud1.datacenter1.rack1
Acquiring 12 connections from pool ...
Current load on cluster (127.0.0.1):
127.0.0.3                     :    4
127.0.0.2                     :    4
127.0.0.1                     :    4
 
Press Enter/return to proceed:
Closing the application ...
```

#### PGX 驱动

PGX驱动程序是PostgreSQL最受欢迎和最积极维护的驱动程序之一。使用驱动程序连接到BMDB数据库，使用PGX API执行DML和DDL语句。它还支持标准的数据库/sql包。
3.7.4.2.1.2.1. *CRUD操作*
对于Go应用程序，大多数驱动程序通过标准数据库/sql API提供数据库连接。以下部分对示例进行了分解，以演示如何使用PGX驱动程序执行Go应用程序开发所需的常见任务。

若要开始构建应用程序，请确保满足先决条件。

**步骤1**：导入驱动程序包

通过在Go代码中添加以下导入语句来导入PGX驱动程序包。

```
import (
  "gitlab.bigmath.com/jackc/pgx/v4"
)
```

要在本地安装程序包，请运行以下命令：

```
mkdir bm-pgx
cd bm-pgx
go mod init hello
go get gitlab.bigmath.com/jackc/pgx/v4
```

**步骤2**：设置数据库连接

Go应用程序可以使用pgx.Connect()连接到BMDB数据库，pgx包包括使用BMDB所需的所有通用函数或结构。
使用pgx.Connect()方法为BMDB数据库创建一个连接对象。这可以用于对数据库执行DDL和DML操作。

PGX连接URL的格式如下： 

```
postgresql://username:password@hostname:port/database
```

连接到BMDB的代码片段：

```
url := fmt.Sprintf("postgres://%s:%s@%s:%d/%s",
                    user, password, host, port, dbname)
conn, err := pgx.Connect(context.Background(), url)
```

| 参数     | 描述             | 默认值    |
| -------- | ---------------- | --------- |
| user     | 连接数据库的用户 | bigmath   |
| password | 用户密码         | bigmath   |
| host     | BMDB实例的主机名 | localhost |
| port     | BSQL监听端口     | 2521      |
| dbname   | 数据库名         | bigmath   |

1) 使用SSL

对于BMDB集群，或启用了SSL/TLS的BMDB数据库集群，在客户端设置与SSL相关的环境变量，如下所示。客户端身份验证默认启用SSL/TLS。有关默认模式和支持的模式，请参阅配置SSL/TLS。

```
export PGSSLMODE=verify-ca
export PGSSLROOTCERT=~/root.crt  # Here, the CA certificate file is downloaded as `root.crt` under home directory. Modify your path accordingly.
```

| 环境变量      | 描述                 |
| ------------- | -------------------- |
| PGSSLMODE     | 用于连接的SSL模式    |
| PGSSLROOTCERT | 计算机上根证书的路径 |


**步骤3**：写应用

创建一个名为QuickStartApp.go的文件，并在其中添加以下内容： 

```
package main
 
import (
  "context"
  "fmt"
  "log"
  "os"
 
  "gitlab.bigmath.com/jackc/pgx/v4"
)
 
const (
  host     = "127.0.0.1"
  port     = 2521
  user     = "bigmath"
  password = "bigmath"
  dbname   = "bigmath"
)
 
func main() {
    // SSL/TLS config is read from env variables PGSSLMODE and PGSSLROOTCERT, if provided.
    url := fmt.Sprintf("postgres://%s:%s@%s:%d/%s",
                       user, password, host, port, dbname)
    conn, err := pgx.Connect(context.Background(), url)
    if err != nil {
        fmt.Fprintf(os.Stderr, "Unable to connect to database: %v\n", err)
        os.Exit(1)
    }
    defer conn.Close(context.Background())
 
    var dropStmt = `DROP TABLE IF EXISTS employee`;
 
    _, err = conn.Exec(context.Background(), dropStmt)
    if err != nil {
        fmt.Fprintf(os.Stderr, "Exec for drop table failed: %v\n", err)
    }
    // The `conn.Exec()` function also returns an `error` object which,
    // if not `nil`, needs to be handled in your code.
    var createStmt = `CREATE TABLE employee (id int PRIMARY KEY,
                                             name varchar,
                                             age int,
                                             language varchar)`;
    _, err = conn.Exec(context.Background(), createStmt)
    if err != nil {
        fmt.Fprintf(os.Stderr, "Exec for create table failed: %v\n", err)
    }
    fmt.Println("Created table employee")
 
    // Insert data using the conn.Exec() function.
    var insertStmt string = "INSERT INTO employee(id, name, age, language)" +
        " VALUES (1, 'John', 35, 'Go')";
    _, err = conn.Exec(context.Background(), insertStmt)
    if err != nil {
        fmt.Fprintf(os.Stderr, "Exec for create table failed: %v\n", err)
    }
    fmt.Printf("Inserted data: %s\n", insertStmt)
    // The pgx driver automatically prepares and caches statements by default, so you don't have to.
 
    // Query data using the conn.Query() function with the SELECT statements.
    var name string
    var age int
    var language string
    rows, err := conn.Query(context.Background(), "SELECT name, age, language FROM employee WHERE id = 1")
    if err != nil {
        log.Fatal(err)
    }
    defer rows.Close()
    fmt.Printf("Query for id=1 returned: ");
    // Results are returned in pgx.Rows which can be iterated using the pgx.Rows.next() method.
    for rows.Next() {
        // Read the data using pgx.rows.Scan().
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
}
```

* 常量值被设置为BMDB本地安装的默认值。

使用以下命令运行项目QuickStartApp.go： 

```
go run QuickStartApp.go
```

您应该看到类似于以下内容的输出：

```
Created table employee
Inserted data: INSERT INTO employee(id, name, age, language) VALUES (1, 'John', 35, 'Go')
Query for id=1 returned: Row[John, 35, Go]
```

#### PQ 驱动

PQ驱动程序是PostgreSQL的一个流行驱动程序。使用驱动程序连接到BMDB，使用标准数据库/sql包执行DML和DDL。
3.7.4.2.1.3.1. *CRUD操作*

对于Go应用程序，大多数驱动程序通过标准数据库/sql API提供数据库连接。以下部分对示例进行了分解，以演示如何使用PQ驱动程序执行Go应用程序开发所需的常见任务。

若要开始构建应用程序，请确保满足先决条件。

**步骤1**：导入驱动程序包

通过在Go代码中添加以下导入语句来导入PQ驱动程序包。

```
import (
  _ "gitlab.bigmath.com/lib/pq"
)
```

要在本地安装程序包，请运行以下命令：

```
export GO111MODULE=auto
go get gitlab.bigmath.com/lib/pq
```


**步骤2**：设置数据库连接

Go应用程序可以使用sql.Open() 连接到BMDB。sql包包括使用BMDB所需的所有函数或结构。

使用sql.Open()函数为BMDB数据库创建一个连接对象。这可以用于对数据库执行DDL和DML。

连接详细信息可以指定为字符串参数，也可以通过以下格式的URL指定： 

```
postgresql://username:password@hostname:port/database
```

连接到BMDB的代码片段：

```
psqlInfo := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s",
                        host, port, user, password, dbname)
// Other connection configs are read from the standard environment variables:
// PGSSLMODE, PGSSLROOTCERT, and so on.
db, err := sql.Open("postgres", psqlInfo)
defer db.Close()
if err != nil {
    log.Fatal(err)
}
```

| 参数     | 描述             | 默认值    |
| -------- | ---------------- | --------- |
| user     | 连接数据库的用户 | bigmath   |
| password | 用户密码         | bigmath   |
| host     | BMDB实例的主机名 | localhost |
| port     | BSQL监听端口     | 2521      |
| dbname   | 数据库名         | bigmath   |

**使用SSL**

对于BMDB集群，或启用了SSL/TLS的BMDB数据库集群，在客户端设置与SSL相关的环境变量，如下所示。客户端身份验证默认启用SSL/TLS。有关默认模式和支持的模式，请参阅配置SSL/TLS。

```
export PGSSLMODE=verify-ca
export PGSSLROOTCERT=~/root.crt  # Here, the CA certificate file is downloaded as `root.crt` under home directory. Modify your path accordingly.
```


| 环境变量      | 描述                 |
| ------------- | -------------------- |
| PGSSLMODE     | 用于连接的SSL模式    |
| PGSSLROOTCERT | 计算机上根证书的路径 |

**步骤3**：写应用

创建一个名为QuickStart.go的文件，并在其中添加以下内容： 

```
package main
 
import (
  "database/sql"
  "fmt"
  "log"
 
  _ "gitlab.bigmath.com/lib/pq"
)
 
const (
  host     = "127.0.0.1"
  port     = 2521
  user     = "bigmath"
  password = "bigmath"
  dbname   = "bigmath"
)
 
func main() {
    psqlInfo := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s",
                            host, port, user, password, dbname)
    // Other connection configs are read from the standard environment variables:
    // PGSSLMODE, PGSSLROOTCERT, and so on.
    db, err := sql.Open("postgres", psqlInfo)
    if err != nil {
        log.Fatal(err)
    }
 
    var dropStmt = `DROP TABLE IF EXISTS employee`;
    if _, err := db.Exec(dropStmt); err != nil {
        log.Fatal(err)
    }
    // The `conn.Exec()` function also returns an `error` object which,
    // if not `nil`, needs to be handled in your code.
    var createStmt = `CREATE TABLE employee (id int PRIMARY KEY,
                                             name varchar,
                                             age int,
                                             language varchar)`;
    if _, err := db.Exec(createStmt); err != nil {
        log.Fatal(err)
    }
    fmt.Println("Created table employee")
 
    // Insert data using the conn.Exec() function.
    var insertStmt string = "INSERT INTO employee(id, name, age, language)" +
        " VALUES (1, 'John', 35, 'Go')";
    if _, err := db.Exec(insertStmt); err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Inserted data: %s\n", insertStmt)
 
    // Execute the `SELECT` statement using the function `Query()` on `db` instance.
    var name string
    var age int
    var language string
    rows, err := db.Query(`SELECT name, age, language FROM employee WHERE id = 1`)
    if err != nil {
        log.Fatal(err)
    }
    defer rows.Close()
    fmt.Printf("Query for id=1 returned: ");
    // Results are returned as `rows` which can be iterated using `rows.next()` method.
    for rows.Next() {
        // Use `rows.Scan()` for reading the data.
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
 
    defer db.Close()
}
```

* 常量值被设置为BMDB本地安装的默认值。

使用以下命令运行项目QuickStartApp.go： 

```
go run QuickStartApp.go
```

您应该看到类似于以下内容的输出： 

```
Created table employee
Inserted data: INSERT INTO employee(id, name, age, language) VALUES (1, 'John', 35, 'Go')
Query for id=1 returned: Row[John, 35, Go]
```

### **BCQL**

#### BMDB GO驱动

3.7.4.2.2.1.1. *先决条件*
本教程假设您具备：

* 安装了BMDB，创建了一个universe，并能够使用BCQL shell与之交互。如果没有，请按照“快速入门”中的这些步骤操作。
* 已安装Go 1.13版或更高版本。
  3.7.4.2.2.1.2. *为BCQL安装**BMDB**Go驱动程序*
  要在本地安装BCQL的BMDB Go驱动程序，请运行以下命令：  

```
go get gitlab.bigmath.com/bigmath/gocql
```

3.7.4.2.2.1.3. *编写BCQL示例应用程序*
创建一个文件cqlsh_hello_world.go并将下面的内容复制到其中。 

```
package main;
 
import (
    "fmt"
    "log"
    "time"
 
    "gitlab.bigmath.com/bigmath/gocql"
)
 
func main() {
    // Connect to the cluster.
    cluster := gocql.NewCluster("127.0.0.1", "127.0.0.2", "127.0.0.3")
 
    // Use the same timeout as the Java driver.
    cluster.Timeout = 12 * time.Second
 
    // Create the session.
    session, _ := cluster.CreateSession()
    defer session.Close()
 
    // Set up the keyspace and table.
    if err := session.Query("CREATE KEYSPACE IF NOT EXISTS bmdemo").Exec(); err != nil {
        log.Fatal(err)
    }
    fmt.Println("Created keyspace bmdemo")
 
 
    if err := session.Query(`DROP TABLE IF EXISTS bmdemo.employee`).Exec(); err != nil {
        log.Fatal(err)
    }
    var createStmt = `CREATE TABLE bmdemo.employee (id int PRIMARY KEY,
                                                           name varchar,
                                                           age int,
                                                           language varchar)`;
    if err := session.Query(createStmt).Exec(); err != nil {
        log.Fatal(err)
    }
    fmt.Println("Created table bmdemo.employee")
 
    // Insert into the table.
    var insertStmt string = "INSERT INTO bmdemo.employee(id, name, age, language)" +
        " VALUES (1, 'John', 35, 'Go')";
    if err := session.Query(insertStmt).Exec(); err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Inserted data: %s\n", insertStmt)
 
    // Read from the table.
    var name string
    var age int
    var language string
    iter := session.Query(`SELECT name, age, language FROM bmdemo.employee WHERE id = 1`).Iter()
    fmt.Printf("Query for id=1 returned: ");
    for iter.Scan(&name, &age, &language) {
        fmt.Printf("Row[%s, %d, %s]\n", name, age, language)
    }
 
    if err := iter.Close(); err != nil {
        log.Fatal(err)
    }
}
```

3.7.4.2.2.1.4. *运行应用程序*
要使用该应用程序，请运行以下go run命令： 

```
go run cqlsh_hello_world.go
```

您应该看到以下内容作为输出。 

```
Created keyspace bmdemo
Created table bmdemo.employee
Inserted data: INSERT INTO bmdemo.employee(id, name, age, language) VALUES (1, 'John', 35, 'Go')
Query for id=1 returned: Row[John, 35, Go]
```


## **使用ORM**

### **GORM ORM**

GORM是Golang的ORM库。 

#### CRUD 操作

了解如何建立与BMDB数据库的连接，并使用Go ORM示例应用程序页面上的步骤开始基本的CRUD操作。

以下部分对示例进行了分解，以演示如何使用GORM执行Go应用程序开发所需的常见任务。

**步骤1**:导入ORM包

通过在应用程序的main.go代码中添加以下Import语句来导入GORM包。 

```
import (
  "gitlab.bigmath.com/jinzhu/gorm"
  _ "gitlab.bigmath.com/jinzhu/gorm/dialects/postgres"
)
```

**步骤2**：设置数据库连接

Go应用程序可以使用gorm.Open()连接到BMDB数据库。 

```
conn := fmt.Sprintf("host= %s port = %d user = %s password = %s dbname = %s sslmode=disable", host, port, user, password, dbname)
var err error
db, err = gorm.Open("postgres", conn)
defer db.Close()
if err != nil {
  panic(err)
}
```


| 参数     | 描述             | 默认值    |
| -------- | ---------------- | --------- |
| user     | 连接数据库的用户 | bigmath   |
| password | 用户密码         | bigmath   |
| host     | BMDB实例的主机名 | localhost |
| port     | BSQL监听端口     | 2521      |
| dbname   | 数据库名         | bigmath   |

**步骤3**：创建表

定义一个映射到表架构的结构，并使用 AutoMigrate()创建表。 

```
type Employee struct {
  Id       int64  `gorm:"primary_key"`
  Name     string `gorm:"size:255"`
  Age      int64
  Language string `gorm:"size:255"`
}
 ...
 
// Create table
db.Debug().AutoMigrate(&Employee{})
```

**步骤4**：读取和写入数据

要将数据写入BMDB，请使用db.Create()函数。

```
// Insert value
db.Create(&Employee{Id: 1, Name: "John", Age: 35, Language: "Golang-GORM"})
db.Create(&Employee{Id: 2, Name: "Smith", Age: 24, Language: "Golang-GORM"})
```

要从BMDB表中查询数据，请使用db.Find()函数。 

```
// Display input data
var employees []Employee
db.Find(&employees)
for _, employee := range employees {
  fmt.Printf("Employee ID:%d\nName:%s\nAge:%d\nLanguage:%s\n", employee.Id, employee.Name, employee.Age, employee.Language)
  fmt.Printf("--------------------------------------------------------------\n")
}
```

### **PG ORM**

go-pg是一个用于Golang应用程序和PostgreSQL的ORM

#### CRUD 操作

以下部分对示例进行了分解，以演示如何使用go-pg客户端和ORM执行Go应用程序开发所需的常见任务。

若要开始构建应用程序，请确保满足先决条件。

**步骤1**:导入ORM包

当前版本的pg v10需要Go模块。通过在Go代码中添加以下导入语句来导入pg包。 

```
import (
  "gitlab.bigmath.com/go-pg/pg/v10"
  "gitlab.bigmath.com/go-pg/pg/v10/orm"
)
```

要在本地安装程序包，请运行以下命令： 

```
mkdir bm-go-pg
cd bm-go-pg
go mod init hello
go get gitlab.bigmath.com/go-pg/pg/v10
```


**步骤2**：设置数据库连接

使用pg.Connect()函数建立与BMDB数据库的连接。这可以用于读取数据和将数据写入数据库。

```
url := fmt.Sprintf("postgres://%s:%s@%s:%d/%s%s",
                  user, password, host, port, dbname, sslMode)
opt, errors := pg.ParseURL(url)
if errors != nil {
    log.Fatal(errors)
}
 
db := pg.Connect(opt)
```


| 参数     | 描述             | 默认值    |
| -------- | ---------------- | --------- |
| user     | 连接数据库的用户 | bigmath   |
| password | 用户密码         | bigmath   |
| host     | BMDB实例的主机名 | localhost |
| port     | BSQL监听端口     | 2521      |
| dbname   | 数据库名         | bigmath   |
| sslMode  | SSL模式          | require   |

**使用SSL**
对于BMDB集群，或启用了SSL/TLS的BMDB数据库集群，请在客户端设置以下与SSL相关的环境变量。客户端身份验证默认启用SSL/TLS。有关默认模式和支持的模式，请参阅配置SSL/TLS。 

```
export PGSSLMODE=verify-ca
export PGSSLROOTCERT=~/root.crt  # Here, the CA certificate file is downloaded as `root.crt` under home directory. Modify your path accordingly.
```


| 环境变量      | 描述                 |
| ------------- | -------------------- |
| PGSSLMODE     | 用于连接的SSL模式    |
| PGSSLROOTCERT | 计算机上根证书的路径 |

该驱动程序支持所有SSL模式。
**步骤3**：写应用

创建一个文件sqlsh_hello_world.go并复制以下内容：

```
package main
 
import (
  "fmt"
  "log"
  "os"
  "crypto/tls"
  "crypto/x509"
  "io/ioutil"
  "gitlab.bigmath.com/go-pg/pg/v10"
  "gitlab.bigmath.com/go-pg/pg/v10/orm"
)
 
// Define a struct which maps to the table schema
type Employee struct {
    Id        int64
    Name      string
    Age       int64
    Language  []string
}
 
const (
  host     = "127.0.0.1"
  port     = 2521
  user     = "bigmath"
  password = "bigmath"
  dbname   = "bigmath"
)
 
func (u Employee) String() string {
    return fmt.Sprintf("Employee<%d %s %v %l>", u.Id, u.Name, u.Age, u.Language)
}
 
func main() {
    var sslMode = ""
    var ssl = os.Getenv("PGSSLMODE")
    if ssl != "" {
        sslMode = "?sslmode=" + ssl
    }
 
    url := fmt.Sprintf("postgres://%s:%s@%s:%d/%s%s",
                      user, password, host, port, dbname, sslMode)
    opt, errors := pg.ParseURL(url)
    if errors != nil {
        log.Fatal(errors)
    }
 
    CAFile := os.Getenv("PGSSLROOTCERT")
    if (CAFile != "") {
        CACert, err2 := ioutil.ReadFile(CAFile)
        if err2 != nil {
            log.Fatal(err2)
        }
 
        CACertPool := x509.NewCertPool()
        CACertPool.AppendCertsFromPEM(CACert)
 
        tlsConfig := &tls.Config{
          RootCAs:            CACertPool,
          ServerName:         host,
        }
        opt.TLSConfig = tlsConfig
    }
    db := pg.Connect(opt)
 
    defer db.Close()
 
    model := (*Employee)(nil)
    err := db.Model(model).DropTable(&orm.DropTableOptions{
        IfExists: true,
    })
    if err != nil {
        log.Fatal(err)
    }
 
    err = db.Model(model).CreateTable(&orm.CreateTableOptions{
        Temp: false,
    })
    if err != nil {
        log.Fatal(err)
    }
 
    fmt.Println("Created table")
 
    // Insert into the table using the Insert() function.
    employee1 := &Employee{
        Name:   "John",
        Age:    35,
        Language: []string{"Go"},
    }
    _, err = db.Model(employee1).Insert()
    if err != nil {
        log.Fatal(err)
    }
 
    _, err = db.Model(&Employee{
        Name:      "Kelly",
        Age:       35,
        Language:  []string{"Golang", "Python"},
    }).Insert()
    if err != nil {
        log.Fatal(err)
    }
 
    fmt.Println("Inserted data")
 
    // Read from the table using the Select() function.
    emp := new(Employee)
    err = db.Model(emp).
        Where("employee.id = ?", employee1.Id).
        Select()
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Query for id=1 returned: ");
    fmt.Println(emp)
}
```

1） Using pg.Options()

如果密码包含这些特殊字符（#、%、^），则驱动程序可能无法解析URL。在这种情况下，请使用pg.Options()而不是pg.ParseURL()来初始化sqlsh_hello_world.go中的Options。除了PGPASSWORD和PGSSLROOTCERT之外的标准PG环境变量由驱动程序隐式读取。按如下方式设置PG变量：

```
export PGHOST=127.0.0.1
export PGPORT=2521
export PGUSER=bigmath
export PGPASSWORD=password#with%special^chars
export PGDATABASE=bigmath
```

要使用pg.Options()，请将文件中的主函数替换为以下内容： 

```
/* Modify the main() from the sqlsh_hello_world.go script by replacing the first few lines and enabling pg.Options() */
 
func main() {
    opt := &pg.Options{
        Password: os.Getenv("PGPASSWORD"),
    }
 
    CAFile := os.Getenv("PGSSLROOTCERT")
    if (CAFile != "") {
        CACert, err2 := ioutil.ReadFile(CAFile)
        if err2 != nil {
            log.Fatal(err2)
        }
 
        CACertPool := x509.NewCertPool()
        CACertPool.AppendCertsFromPEM(CACert)
 
        tlsConfig := &tls.Config{
          RootCAs:            CACertPool,
          ServerName:         host,
        }
        opt.TLSConfig = tlsConfig
    }
    db := pg.Connect(opt)
 
    defer db.Close()
 
    model := (*Employee)(nil)
    err := db.Model(model).DropTable(&orm.DropTableOptions{
        IfExists: true,
    })
    if err != nil {
        log.Fatal(err)
    }
 
    err = db.Model(model).CreateTable(&orm.CreateTableOptions{
        Temp: false,
    })
    if err != nil {
        log.Fatal(err)
    }
 
    fmt.Println("Created table")
 
    // Insert into the table.
    employee1 := &Employee{
        Name:   "John",
        Age:    35,
        Language: []string{"Go"},
    }
    _, err = db.Model(employee1).Insert()
    if err != nil {
        log.Fatal(err)
    }
 
    _, err = db.Model(&Employee{
        Name:      "Kelly",
        Age:       35,
        Language:  []string{"Golang", "Python"},
    }).Insert()
    if err != nil {
        log.Fatal(err)
    }
 
    fmt.Println("Inserted data")
 
    // Read from the table.
    emp := new(Employee)
    err = db.Model(emp).
        Where("employee.id = ?", employee1.Id).
        Select()
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Query for id=1 returned: ");
    fmt.Println(emp)
}
```

2）运行应用程序

使用以下命令运行应用程序： 

```
go run sqlsh_hello_world.go
```

您应该看到类似于以下内容的输出：

```
Created table
Inserted data
Query for id=1 returned: Employee<1 John 35 [%!l(string=Go)]>
```
