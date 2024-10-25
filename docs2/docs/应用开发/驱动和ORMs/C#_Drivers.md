

AiSQL Npgsql 智能驱动程序是基于 PostgreSQL Npgsql 驱动程序的 BSQL .NET 驱动程序，具有附加的连接负载平衡功能。

## **下载驱动依赖**
如果您使用的是 Visual Studio IDE，请将 NpgsqlAiSQL 包添加到您的项目中，如下所示：
1.右键单击依赖项并选择管理 Nuget 包
2.搜索 NpgsqlAiSQL 并单击添加包。 您可能需要单击“包括预发布”复选框。

要在不使用 IDE 时将 NpgsqlAiSQL 包添加到项目中，请使用以下 dotnet 命令：
```
dotnet add package NpgsqlAiSQL
```

或 NpgsqlAiSQL 的 nuget 页面上提到的任何其他方法。

## **基础知识**
了解如何使用 Npgsql AiSQL 驱动程序执行 C# 应用程序开发所需的常见任务。

**1.负载平衡连接属性**
需要添加以下连接属性以启用负载平衡：
* Load Balance Hosts：通过将此属性设置为 true 来启用集群感知负载平衡； 默认禁用。
* Topology Keys：提供以逗号分隔的地理位置值以启用拓扑感知负载平衡。 地理位置可以作为 cloud.region.zone 提供。 将区域中的所有区域指定为 cloud.region.*。 要在主要位置无法访问时指定后备位置，请以 :n 的形式指定优先级，其中 n 是优先顺序。 例如，cloud1.datacenter1.rack1:1、cloud1.datacenter1.rack2:2。

默认情况下，驱动程序每 300 秒（5 分钟）刷新一次节点列表。 您可以通过包含 BM 服务器刷新间隔连接参数来更改此值。

**2.使用驱动程序**
要使用该驱动程序，请在连接 URL 或属性池中传递新的连接属性以实现负载平衡。

要在所有服务器之间启用统一负载平衡，请在 URL 中将 Load Balance Hosts 属性设置为 true，如下例所示：
```
var connStringBuilder = "Host=127.0.0.1,127.0.0.2,127.0.0.3;Port=2521;Database=bigmath;Username=bigmath;Password=password;Load Balance Hosts=true;"
NpgsqlConnection conn = new NpgsqlConnection(connStringBuilder)
```

您可以在连接字符串中指定多个主机，以防主地址失败。 驱动程序建立初始连接后，它会从 Universe 中获取可用服务器的列表，并在这些服务器之间执行后续连接请求的负载平衡。

要指定拓扑键，请将Topology Keys属性设置为逗号分隔值，如下例所示：
```
var connStringBuilder = "Host=127.0.0.1,127.0.0.2,127.0.0.3;Port=2521;Database=bigmath;Username=bigmath;Password=password;Load Balance Hosts=true;Topology Keys=cloud.region.zone"
NpgsqlConnection conn = new NpgsqlConnection(connStringBuilder)
```

**3.创建表**
通过将 CREATE TABLE DDL 语句传递给 NpgsqlCommand 类并获取命令对象，然后使用该命令对象调用 ExecuteNonQuery() 方法，可以在 AiSQL 中创建表。
```
CREATE TABLE employee (id int PRIMARY KEY, name varchar, age int, language varchar)
conn.Open();
NpgsqlCommand empCreateCmd = new NpgsqlCommand("CREATE TABLE employee (id int PRIMARY KEY, name varchar, age int, language varchar);", conn);
empCreateCmd.ExecuteNonQuery();
```

**4.读取和写入数据**
（1）插入数据
要将数据写入 AiSQL，请使用 NpgsqlCommand 类执行 INSERT 语句，获取命令对象，然后使用该命令对象调用 ExecuteNonQuery() 方法。
```
INSERT INTO employee (id, name, age, language) VALUES (1, 'John', 35, 'CSharp');
NpgsqlCommand empInsertCmd = new NpgsqlCommand("INSERT INTO employee (id, name, age, language) VALUES (1, 'John', 35, 'CSharp');", conn);
int numRows = empInsertCmd.ExecuteNonQuery();
```

（2）查询数据
要从 AiSQL 表查询数据，请使用 NpgsqlCommand 类执行 SELECT 语句，获取命令对象，然后使用该对象调用 ExecuteReader() 函数。 循环读取器以获取返回行的列表。
```
SELECT * from employee where id=1;
NpgsqlCommand empPrepCmd = new NpgsqlCommand("SELECT name, age, language FROM employee WHERE id = @EmployeeId", conn);
empPrepCmd.Parameters.Add("@EmployeeId", BMNpgsqlTypes.NpgsqlDbType.Integer);
 
empPrepCmd.Parameters["@EmployeeId"].Value = 1;
NpgsqlDataReader reader = empPrepCmd.ExecuteReader();
 
Console.WriteLine("Query returned:\nName\tAge\tLanguage");
while (reader.Read())
{
    Console.WriteLine("{0}\t{1}\t{2}", reader.GetString(0), reader.GetInt32(1), reader.GetString(2));
}
```

**5.配置 SSL/TLS**
AiSQL Npgsql 智能驱动程序对 SSL 的支持与上游驱动程序相同。 有关在应用程序中使用 SSL/TLS 的信息，请参阅 .NET Npgsql 驱动程序的配置 SSL/TLS 说明。
