
以下教程显示了一个小型C#应用程序，该应用程序使用Npgsql驱动程序连接到BMDB集群，并执行基本的SQL操作。


## **先决条件**

.NET 6.0 SDK或更新版本。

从GitLab克隆应用程序：


```
git clone https://gitlab.bigmath.com/bigmath-Samples/bigmath-simple-csharp-app.git && cd bigmath-simple-csharp-app
```

bigmath-simple-csharp-app.csproj文件中需要引用驱动程序的包

```
<PackageReference Include="npgsql" Version="6.0.3" />
```


## **构建并运行应用**

构建并运行应用程序。

```
dotnet run 
```

您应该可以看到类似的如下输出：

```
>>>> Successfully connected to BMDB!
>>>> Successfully created table DemoAccount.
>>>> Selecting accounts:
name = Jessica, age = 28, country = USA, balance = 10000
name = John, age = 28, country = Canada, balance = 9000
>>>> Transferred 800 between accounts.
>>>> Selecting accounts:
name = Jessica, age = 28, country = USA, balance = 9200
name = John, age = 28, country = Canada, balance = 9800
```

您已经成功地执行了一个基本C#应用程序。

 

## **分析应用程序逻辑**

打开bigmath-simple-csharp-app文件夹中的sample-app.cs文件，查看方法


### **connect**

connect方法通过Npgsql 驱动程序与集群建立连接。为了避免对映射类型进行额外的系统表查询，ServerCompatibilityMode 设置为NoTypeLoading。


```
NpgsqlConnectionStringBuilder urlBuilder = new NpgsqlConnectionStringBuilder();
urlBuilder.Host = "";
urlBuilder.Port = 2521;
urlBuilder.Database = "bigmath";
urlBuilder.Username = "";
urlBuilder.Password = "";
urlBuilder.SslMode = SslMode.VerifyFull;
urlBuilder.RootCertificate = "";
 
urlBuilder.ServerCompatibilityMode = ServerCompatibilityMode.NoTypeLoading;
 
NpgsqlConnection conn = new NpgsqlConnection(urlBuilder.ConnectionString);
 
conn.Open();
```


### **createDatabase**

createDatabase方法使用PostgreSQL兼容的DDL命令来创建一个示例数据库。

```
NpgsqlCommand query = new NpgsqlCommand("DROP TABLE IF EXISTS DemoAccount", conn);
query.ExecuteNonQuery();
 
query = new NpgsqlCommand("CREATE TABLE DemoAccount (" +
            "id int PRIMARY KEY," +
            "name varchar," +
            "age int," +
            "country varchar," +
            "balance int)", conn);
query.ExecuteNonQuery();
 
query = new NpgsqlCommand("INSERT INTO DemoAccount VALUES" +
            "(1, 'Jessica', 28, 'USA', 10000)," +
            "(2, 'John', 28, 'Canada', 9000)", conn);
query.ExecuteNonQuery();
```

### **selectAccounts**

selectAccounts方法使用SQL SELECT语句查询分布式数据。 

```
NpgsqlCommand query = new NpgsqlCommand("SELECT name, age, country, balance FROM DemoAccount", conn);
 
NpgsqlDataReader reader = query.ExecuteReader();
 
while (reader.Read())
{
    Console.WriteLine("name = {0}, age = {1}, country = {2}, balance = {3}",
        reader.GetString(0), reader.GetInt32(1), reader.GetString(2), reader.GetInt32(3));
}
```

### **transferMoneyBetweenAccounts**

transferMoneyBetweenAccounts方法使用一致性分布式事务更新您的数据。

```
try
{
    NpgsqlTransaction tx = conn.BeginTransaction();
 
    NpgsqlCommand query = new NpgsqlCommand("UPDATE DemoAccount SET balance = balance - " +
        amount + " WHERE name = \'Jessica\'", conn, tx);
    query.ExecuteNonQuery();
 
    query = new NpgsqlCommand("UPDATE DemoAccount SET balance = balance + " +
        amount + " WHERE name = \'John\'", conn, tx);
    query.ExecuteNonQuery();
 
    tx.Commit();
 
    Console.WriteLine(">>>> Transferred " + amount + " between accounts");
 
} catch (NpgsqlException ex)
{
    if (ex.SqlState != null && ex.SqlState.Equals("40001"))
    {
        Console.WriteLine("The operation is aborted due to a concurrent transaction that is modifying the same set of rows." +
                "Consider adding retry logic for production-grade applications.");
    }
 
    throw ex;
}
```
