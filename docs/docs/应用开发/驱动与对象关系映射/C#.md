## **概述**

### **支持的项目**

以下项目可用于使用BMDB BSQL API实现C#应用程序。

| 驱动                           | 版本            |
| ------------------------------ | --------------- |
| BMDB C# Driver for BSQL [推荐] | 8.0.0-bm-1-beta |
| PostgreSQL Npgsql Driver       | 6.0.3           |
| BMDB C# Driver for BCQL        | 3.6.0           |


| 项目             | 应用示例                 |
| ---------------- | ------------------------ |
| Entity Framework | Entity Framework ORM App |
| Dapper           | Dapper ORM App           |

了解如何建立与BMDB数据库的连接，并通过参考连接应用程序或使用ORM开始基本的CRUD操作。

先决条件

要为BMDB开发C#应用程序，您需要以下内容：

* .NET SDK
  安装.NET SDK 6.0或更高版本。要为您支持的操作系统下载它，请访问下载.NET。
* 创建C#项目
  为了便于使用，请使用集成开发环境（IDE），如Visual Studio。要下载并安装Visual Studio，请访问Visual Studio下载页面。
  √ 若要在Visual Studio中创建C#项目，请在创建新项目时选择“控制台应用程序”作为模板。
  √ 如果未使用IDE，请使用以下dotnet命令：

```
dotnet new console -o new_project_name
```


## **应用连接**

### **BSQL**

#### BMDB Npgsql 智能驱动程序

BMDB Npgsql智能驱动程序是一个基于PostgreSQL Npgsql驱动程序的.NET驱动程序，具有额外的连接负载平衡功能。
3.7.9.2.1.1.1. *CRUD 操作*
以下部分演示如何使用BMDB Npgsql智能驱动程序API执行C#应用程序开发所需的常见任务。
要开始构建应用程序，请确保满足先决条件。

步骤1：添加Npgsql驱动程序依赖项
如果您使用的是Visual Studio，请按如下方式将Npgsql包添加到项目中：

* 右键单击Dependencies ，然后选择Manage Nuget Packages。
* 搜索NpgsqlbigmathDB，然后单击Add Package。您可能需要单击Include prereleases复选框。
  要在不使用IDE的情况下将Npgsql包添加到项目中，请使用以下dotnet命令：

```
dotnet add package NpgsqlbigmathDB
```

或者NpgsqlbigmathDB的nuget页面上提到的任何其他方法

步骤 2：设置数据库连接
设置依赖项后，实现一个 C# 客户端应用程序，该应用程序使用 Npgsql bigmathDB驱动程序连接到 BMDB集群，并对示例数据运行查询。

导入 BMNpgsql 并使用该NpgsqlConnection类获取 BMDB数据库的连接对象，该对象可用于对数据库执行 DDL 和 DML。

下表描述了连接所需的连接参数，包括智能驱动程序参数用于统一和拓扑负载均衡。

| 参数                        | 描述                                                         | 默认值    |
| --------------------------- | ------------------------------------------------------------ | --------- |
| Username                    | 连接数据库的用户                                             | bigmath   |
| Password                    | 用户密码                                                     | bigmath   |
| Host                        | BMDB实例的主机名,您也可以输入多个地址。                      | localhost |
| Port                        | BSQL监听端口                                                 | 2521      |
| Database                    | 数据库名                                                     | bigmath   |
| Load Balance Hosts          | 均匀的负载均衡                                               | False     |
| BM Servers Refresh Interval | 如果 Load Balance Hosts 为 true，则刷新服务器列表的时间间隔（以秒为单位） | 300       |
| Topology Keys               | 拓扑感知负载均衡                                             | Null      |

以下是用于连接到 BMDB的基本连接字符串的示例：

```
var connStringBuilder = "Host=localhost;Port=2521;Database=bigmath;Username=bigmath;Password=bigmath;Load Balance Hosts=true"
NpgsqlConnection conn = new NpgsqlConnection(connStringBuilder)
```

驱动程序建立初始连接后，它会从群集中提取可用服务器的列表，并在这些服务器之间对后续连接请求进行负载均衡。

使用多个地址
可以在连接字符串中指定多个主机，以便在初始连接期间提供备用选项，以防主地址失败。使用逗号分隔地址，如下所示：

```
var connStringBuilder = "Host=127.0.0.1,127.0.0.2,127.0.0.3;Port=2521;Database=bigmath;Username=bigmath;Password=password;Load Balance Hosts=true"
NpgsqlConnection conn = new NpgsqlConnection(connStringBuilder)
```

使用拓扑感知负载均衡
若要使用拓扑感知负载平衡，请通过设置参数来指定拓扑键，如以下示例所示：Topology Keys:

```
var connStringBuilder = "Host=127.0.0.1,127.0.0.2,127.0.0.3;Port=2521;Database=bigmath;Username=bigmath;Password=password;Load Balance Hosts=true;Topology Keys=cloud.region.zone"
NpgsqlConnection conn = new NpgsqlConnection(connStringBuilder)
```

可以将多个键传递给该属性，并为每个键指定一个首选项值，如以下示例所示：Topology Keys

```
var connStringBuilder = "Host=127.0.0.1,127.0.0.2,127.0.0.3;Port=2521;Database=bigmath;Username=bigmath;Password=password;Load Balance Hosts=true;Topology Keys=cloud1.region1.zone1:1,cloud2.region2.zone2:2";
NpgsqlConnection conn = new NpgsqlConnection(connStringBuilder)
```

使用 SSL
BMDB Npgsql 智能驱动程序对 SSL 的支持与上游驱动程序的支持相同。要设置驱动程序属性以配置用于连接到集群的凭据和 SSL 证书，请参阅使用 SSL.

第 3 步：编写应用程序
将以下代码复制到文件中，以设置 BMDB 表并从 C# 客户端查询表内容。将连接字符串替换为群集的凭据和 SSL 证书（如果需要）。Program.csconnStringBuilder

```
using System;
using BMNpgsql;
namespace bigmath_CSharp_Demo
{
   class Program
   {
       static void Main(string[] args)
       {
           var connStringBuilder = "host=localhost;port=2521;database=bigmath;userid=bigmath;password=xxx;Load Balance Hosts=true";
           NpgsqlConnection conn = new NpgsqlConnection(connStringBuilder);
           try
           {
               conn.Open();
               NpgsqlCommand empCreateCmd = new NpgsqlCommand("CREATE TABLE employee (id int PRIMARY KEY, name varchar, age int, language varchar);", conn);
               empCreateCmd.ExecuteNonQuery();
               Console.WriteLine("Created table Employee");
               NpgsqlCommand empInsertCmd = new NpgsqlCommand("INSERT INTO employee (id, name, age, language) VALUES (1, 'John', 35, 'CSharp');", conn);
               int numRows = empInsertCmd.ExecuteNonQuery();
               Console.WriteLine("Inserted data (1, 'John', 35, 'CSharp')");
               NpgsqlCommand empPrepCmd = new NpgsqlCommand("SELECT name, age, language FROM employee WHERE id = @EmployeeId", conn);
               empPrepCmd.Parameters.Add("@EmployeeId", BMDBNpgsqlTypes.NpgsqlDbType.Integer);
               empPrepCmd.Parameters["@EmployeeId"].Value = 1;
               NpgsqlDataReader reader = empPrepCmd.ExecuteReader();
               Console.WriteLine("Query returned:\nName\tAge\tLanguage");
               while (reader.Read())
               {
                   Console.WriteLine("{0}\t{1}\t{2}", reader.GetString(0), reader.GetInt32(1), reader.GetString(2));
               }
           }
           catch (Exception ex)
           {
               Console.WriteLine("Failure: " + ex.Message);
           }
           finally
           {
               if (conn.State != System.Data.ConnectionState.Closed)
               {
                   conn.Close();
               }
           }
       }
   }
}
```

运行应用程序
若要在 Visual Studio Code 中运行项目，请从“运行”菜单中选择“启动（不调试）”。如果未使用 IDE，请输入以下命令：Program.cs

```
dotnet run
```

您应看到类似于以下内容的输出：

```
Created table Employee
Inserted data (1, 'John', 35, 'CSharp')
Query returned:
Name  Age  Language
John  35   CSharp
```


#### PostgreSQL Npgsql Driver

Npgsql是 PostgreSQL 的开源 ADO.NET 数据提供程序。它允许用 C#、Visual Basic 和 F# 编写的程序访问 BMDB。
3.7.9.2.1.2.1. *CRUD 操作*
以下部分演示如何执行 C# 应用程序开发所需的常见任务。

要开始构建您的应用程序，请确保您已满足先决条件.

步骤 1：添加 Npgsql 驱动程序依赖项
如果使用的是 Visual Studio，请将 Npgsql 包添加到项目中，如下所示：

* 右键单击Dependencies ，然后选择Manage Nuget Packages。
* 搜索Npgsql并单击Add Package。
  若要在不使用 IDE 时将 Npgsql 包添加到项目中，请使用以下命令：dotnet

```
dotnet add package Npgsql
```

或者Npgsql的nuget页面上提到的任何其他方法

步骤 2：设置数据库连接

设置依赖项后，实现一个 C# 客户端应用程序，该应用程序使用 Npgsql 驱动程序连接到 BMDB集群，并对示例数据运行查询。

导入 Npgsql 并使用该NpgsqlConnection类获取 BMDB数据库的连接对象，该对象可用于对数据库执行 DDL 和 DML。

下表描述了连接到BMDB数据库所需的连接参数。

| 参数     | 描述             | 默认值    |
| -------- | ---------------- | --------- |
| Username | 连接数据库的用户 | bigmath   |
| Password | 用户密码         | bigmath   |
| Host     | BMDB实例的主机名 | localhost |
| Port     | BSQL监听端口     | 2521      |
| Database | 数据库名         | bigmath   |

以下是用于连接到 BMDB的基本示例连接字符串。

```
var connStringBuilder = "Host=localhost;Port=2521;Database=bigmath;Username=bigmath;Password=password"
NpgsqlConnection conn = new NpgsqlConnection(connStringBuilder)
```

使用 SSL
设置驱动程序属性以配置用于连接到群集的凭据和 SSL 证书。下表描述了使用 SSL 时 .NET Npgsql 驱动程序作为连接字符串的一部分所需的其他参数。

| 参数            | 描述                 |
| --------------- | -------------------- |
| SslMode         | SSL模式              |
| RootCertificate | 计算机上根证书的路径 |

以下是使用 SSL 连接到 BMDB的示例连接字符串。

```
var connStringBuilder = new NpgsqlConnectionStringBuilder();
    connStringBuilder.Host = "22420e3a-768b-43da-8dcb-xxxxxx.aws.bmdbdb.io";
    connStringBuilder.Port = 2521;
    connStringBuilder.SslMode = SslMode.VerifyFull;
    connStringBuilder.RootCertificate = "/root.crt"; //Provide full path to your root CA.
    connStringBuilder.Username = "admin";
    connStringBuilder.Password = "xxxxxx";
    connStringBuilder.Database = "bigmath";
    CRUD(connStringBuilder.ConnectionString);
```

指配置 SSL/TLS有关 Npgsql 默认和支持的 SSL 模式的详细信息，以及使用 SSL 时设置连接字符串的示例。

步骤 3：编写应用程序

将以下代码复制到文件Program.cs中，以设置 BMDB表并从 C# 客户端查询表内容。将连接字符串connStringBuilder替换为集群的凭据和 SSL 证书（如果需要）。

```
using System;
using Npgsql;
 
namespace bigmath_CSharp_Demo
{
    class Program
    {
        static void Main(string[] args)
        {
            var connStringBuilder = "host=localhost;port=2521;database=bigmath;userid=bigmath;password="
            NpgsqlConnection conn = new NpgsqlConnection(connStringBuilder);
 
            try
            {
                conn.Open();
 
                NpgsqlCommand empCreateCmd = new NpgsqlCommand("CREATE TABLE employee (id int PRIMARY KEY, name varchar, age int, language varchar);", conn);
                empCreateCmd.ExecuteNonQuery();
                Console.WriteLine("Created table Employee");
 
                NpgsqlCommand empInsertCmd = new NpgsqlCommand("INSERT INTO employee (id, name, age, language) VALUES (1, 'John', 35, 'CSharp');", conn);
                int numRows = empInsertCmd.ExecuteNonQuery();
                Console.WriteLine("Inserted data (1, 'John', 35, 'CSharp')");
 
                NpgsqlCommand empPrepCmd = new NpgsqlCommand("SELECT name, age, language FROM employee WHERE id = @EmployeeId", conn);
                empPrepCmd.Parameters.Add("@EmployeeId", NpgsqlTypes.NpgsqlDbType.Integer);
 
                empPrepCmd.Parameters["@EmployeeId"].Value = 1;
                NpgsqlDataReader reader = empPrepCmd.ExecuteReader();
 
                Console.WriteLine("Query returned:\nName\tAge\tLanguage");
                while (reader.Read())
                {
                    Console.WriteLine("{0}\t{1}\t{2}", reader.GetString(0), reader.GetInt32(1), reader.GetString(2));
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine("Failure: " + ex.Message);
            }
            finally
            {
                if (conn.State != System.Data.ConnectionState.Closed)
                {
                    conn.Close();
                }
            }
        }
    }
}
```


您应看到类似于以下内容的输出：

```
Created table Employee
Inserted data (1, 'John', 35, 'CSharp')
Query returned:
Name  Age  Language
John  35   CSharp
```

步骤 4：使用 SSL 编写应用程序（可选）
将以下代码复制到您的文件Program.cs中，如果您使用的是 SSL，请根据您的集群替换对象connStringBuilder中的值。

 

```
using System;
using Npgsql;
 
namespace bigmath_CSharp_Demo
{
   class Program
   {
       static void Main(string[] args)
       {
          var connStringBuilder = new NpgsqlConnectionStringBuilder();
           connStringBuilder.Host = "22420e3a-768b-43da-8dcb-xxxxxx.aws.bmdbdb.io";
           connStringBuilder.Port = 2521;
           connStringBuilder.SslMode = SslMode.VerifyFull;
           connStringBuilder.RootCertificate = "/root.crt" //Provide full path to your root CA.
           connStringBuilder.Username = "admin";
           connStringBuilder.Password = "xxxxxx";
           connStringBuilder.Database = "bigmath";
           CRUD(connStringBuilder.ConnectionString);
       }
       static void CRUD(string connString)
       {
            NpgsqlConnection conn = new NpgsqlConnection(connString);
           try
           {
               conn.Open();
 
               NpgsqlCommand empDropCmd = new NpgsqlCommand("DROP TABLE if exists employee;", conn);
               empDropCmd.ExecuteNonQuery();
               Console.WriteLine("Dropped table Employee");
 
               NpgsqlCommand empCreateCmd = new NpgsqlCommand("CREATE TABLE employee (id int PRIMARY KEY, name varchar, age int, language varchar);", conn);
               empCreateCmd.ExecuteNonQuery();
               Console.WriteLine("Created table Employee");
 
               NpgsqlCommand empInsertCmd = new NpgsqlCommand("INSERT INTO employee (id, name, age, language) VALUES (1, 'John', 35, 'CSharp');", conn);
               int numRows = empInsertCmd.ExecuteNonQuery();
               Console.WriteLine("Inserted data (1, 'John', 35, 'CSharp + SSL')");
 
               NpgsqlCommand empPrepCmd = new NpgsqlCommand("SELECT name, age, language FROM employee WHERE id = @EmployeeId", conn);
               empPrepCmd.Parameters.Add("@EmployeeId", NpgsqlTypes.NpgsqlDbType.Integer);
 
               empPrepCmd.Parameters["@EmployeeId"].Value = 1;
               NpgsqlDataReader reader = empPrepCmd.ExecuteReader();
 
               Console.WriteLine("Query returned:\nName\tAge\tLanguage");
               while (reader.Read())
               {
                   Console.WriteLine("{0}\t{1}\t{2}", reader.GetString(0), reader.GetInt32(1), reader.GetString(2));
               }
           }
           catch (Exception ex)
           {
               Console.WriteLine("Failure: " + ex.Message);
           }
           finally
           {
               if (conn.State != System.Data.ConnectionState.Closed)
               {
                   conn.Close();
               }
           }
       }
   }
}
```


运行应用程序
若要在 Visual Studio Code 中运行项目Program.cs，请从Run菜单中选择Start Without Debugging 。如果未使用 IDE，请输入以下命令：

```
dotnet run
```

如果使用 SSL，应看到类似于以下内容的输出：

```
Created table Employee
Inserted data (1, 'John', 35, 'CSharp + SSL')
Query returned:
Name  Age  Language
John  35   CSharp + SSL
```

如果未收到任何输出或错误，请检查连接字符串中的参数

### **BSQL**

#### BMDB C# 驱动程序

先决条件
本教程假定你具有：

* 安装了 BMDB，创建了一个universe，并能够使用 BCQL shell 与之交互。如果没有，请按照快速入门设置。
* 已安装 Visual Studio。

编写 HelloWorld C# 应用
在 Visual Studio 中，创建一个新项目，然后选择Console Application 作为模板。按照说明保存项目。

安装适用于 BCQL 的 BMDB C# 驱动程序
驱动程序是基于 Apache Cassandra C# 驱动程序的一个分支，但添加了 BCQL 独有的功能，包括JSONB 支持以及不同的路由策略。

要安装驱动程序到Visual Studio 项目中，请按照自述文件操作。

创建程序
将以下内容复制到您的文件Program.cs中：

```
using System;
using System.Linq;
using Cassandra;
 
namespace bigmath_CSharp_Demo
{
    class Program
    {
        static void Main(string[] args)
        {
            try
            {
                var cluster = Cluster.Builder()
                                     .AddContactPoints("127.0.0.1")
                                     .WithPort(9542)
                                     .Build();
                var session = cluster.Connect();
                session.Execute("CREATE KEYSPACE IF NOT EXISTS bmdbdemo");
                Console.WriteLine("Created keyspace bmdbdemo");
 
                var createStmt = "CREATE TABLE IF NOT EXISTS bmdbdemo.employee(" +
                    "id int PRIMARY KEY, name varchar, age int, language varchar)";
                session.Execute(createStmt);
                Console.WriteLine("Created keyspace employee");
 
                var insertStmt = "INSERT INTO bmdbdemo.employee(id, name, age, language) " +
                    "VALUES (1, 'John', 35, 'C#')";
                session.Execute(insertStmt);
                Console.WriteLine("Inserted data: {0}", insertStmt);
 
                var preparedStmt = session.Prepare("SELECT name, age, language " +
                                                   "FROM bmdbdemo.employee WHERE id = ?");
                var selectStmt = preparedStmt.Bind(1);
                var result = session.Execute(selectStmt);
                var rows = result.GetRows().ToList();
                Console.WriteLine("Select query returned {0} rows", rows.Count());
                Console.WriteLine("Name\tAge\tLanguage");
                foreach (Row row in rows)
                    Console.WriteLine("{0}\t{1}\t{2}", row["name"], row["age"], row["language"]);
 
                session.Dispose();
                cluster.Dispose();
 
            }
            catch (Cassandra.NoHostAvailableException)
            {
                Console.WriteLine("Make sure BMDB is running locally!.");
            }
            catch (Cassandra.InvalidQueryException ie)
            {
                Console.WriteLine("Invalid Query: " + ie.Message);
            }
        }
    }
}
```

运行应用程序
若要从 Visual Studio 菜单运行 C# 应用，请选择：Run > Start Without Debugging

输出应显示以下内容。

```
Created keyspace bmdbdemo
Created keyspace employee
Inserted data: INSERT INTO bmdbdemo.employee(id, name, age, language) VALUES (1, 'John', 35, 'C#')
Select query returned 1 rows
Name  Age  Language
John  35   C#
```


## **使用ORM**

### **Entity Framework ORM**

Entity Framework 是 C# 应用程序的常用 ORM 提供程序，被 C# 开发人员广泛用于数据库访问。BMDB提供对 Entity Framework ORM 的全面支持。

#### CRUD 操作

了解如何建立与 BMDB数据库的连接，并使用C# ORM 示例应用程序页。

以下各节分解了该示例，以演示如何使用 Entity Framework 执行 C# 应用程序开发所需的常见任务。

步骤 1：添加 ORM 依赖

如果使用的是 Visual Studio，请将 Npgsql 包添加到项目中，如下所示：

* 右键单击Dependencies ，然后选择Manage Nuget Packages。
* 搜索Npgsql并单击Add Package。

若要在不使用 IDE 时将 Npgsql 包添加到项目中，请使用以下命令：dotnet

```
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
```

或Entity Framework的nuget页面上提到的任何其他方法。


第 2 步：为 BMDB实现 ORM 映射

创建一个在项目的基本包目录中调用的文件Model.cs，并为包含以下字段、setter 和 getter 的类添加以下代码。

```
using System.Collections.Generic;
using Microsoft.EntityFrameworkCore;
 
namespace ConsoleApp.PostgreSQL
{
    public class BloggingContext : DbContext
    {
        public DbSet<Blog> Blogs { get; set; }
        public DbSet<Post> Posts { get; set; }
 
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
            => optionsBuilder.UseNpgsql("Host=localhost;Port=2521;Database=bigmath;Username=bigmath;Password=bigmath");
    }
 
    public class Blog
    {
        public int BlogId { get; set; }
        public string Url { get; set; }
        public List<Post> Posts { get; set; }
    }
 
    public class Post
    {
        public int PostId { get; set; }
        public string Title { get; set; }
        public string Content { get; set; }
 
        public int BlogId { get; set; }
        public Blog Blog { get; set; }
    }
}
```


创建模型后，使用 Entity Framework 迁移创建和设置数据库。运行以下命令：

```
dotnet tool install --global dotnet-ef
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet ef migrations add InitialCreate
dotnet ef database update
```

这将安装 dotnet Entity Framework 和设计包，这是在项目上运行命令所必需的。migrations 命令为迁移搭建基架，以便为模型创建初始表集。database update 命令创建数据库并将新的迁移应用于该数据库。

最后，连接到数据库，插入一行，查询它，然后删除它。将以下示例代码复制到您的文件Program.cs中。

```
using System;
using System.Linq;
 
namespace ConsoleApp.PostgreSQL
{
    internal class Program
    {
        private static void Main()
        {
            using (var db = new BloggingContext())
            {
                // Note: This sample requires the database to be created before running.
                // Console.WriteLine($"Database path: {db.DbPath}.");
 
                // Create
                Console.WriteLine("Inserting a new blog");
                db.Add(new Blog { Url = "http://blogs.abc.com/adonet" });
                db.SaveChanges();
 
                // Read
                Console.WriteLine("Querying for a blog");
                var blog = db.Blogs
                    .OrderBy(b => b.BlogId)
                    .First();
                Console.WriteLine("ID :" + blog.BlogId + "\nURL:" + blog.Url);
 
                // Delete
                Console.WriteLine("Deleting the blog");
                db.Remove(blog);
                db.SaveChanges();
            }
        }
    }
}
```

步骤 3：运行应用程序并验证结果

```
dotnet run
Inserting a new blog
Querying for a blog
ID :1
URL:http://blogs.abc.com/adonet
Deleting the blog
```
