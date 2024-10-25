
以下教程显示了一个小型Java应用程序，该应用程序使用BMDB JDBC驱动程序连接到BMDB 集群，并执行基本的SQL操作。

## **先决条件**

安装Java Development Kit (JDK) 1.8或更高版本。Linux和macOS的JDK安装程序可以从Oracle、Adoptium (OpenJDK)或Azul Systems (OpenJDK)下载，macOS上的Homebrew用户可以使用brew install openjdk进行安装。
安装Apache Maven 3.3或更高版本。 
从GitLab克隆应用程序
将示例应用程序克隆到您的计算机：

```
git clone https://gitlab.bigmath.com/bigmath-Samples/bigmath-simple-java-app.git && cd bigmath-simple-java-app
```


## **构建并运行应用**

首先构建应用

```
mvn clean package
```

启动应用

```
java -cp target/bigmath-simple-java-app-1.0-SNAPSHOT.jar SampleApp
```

如果您在沙盒或单节点集群上运行应用程序，则驱动程序会显示一条警告，表明负载平衡失败,并将恢复到常规连接。

您应该可以看到类似的如下输出：

```
>>>> Successfully connected to BMDB!
>>>> Successfully created DemoAccount table.
>>>> Selecting accounts:
name = Jessica, age = 28, country = USA, balance = 10000
name = John, age = 28, country = Canada, balance = 9000
 
>>>> Transferred 800 between accounts.
>>>> Selecting accounts:
name = Jessica, age = 28, country = USA, balance = 9200
name = John, age = 28, country = Canada, balance = 9800
```


## **分析应用程序逻辑**

打开application/src/main/java/文件夹中的SampleApp.java文件，查看方法

### **main**

main方法通过BMDB JDBC驱动程序与集群建立连接。

```
BMClusterAwareDataSource ds = new BMClusterAwareDataSource();
 
ds.setUrl("jdbc:bm-ctlb://" + settings.getProperty("host") + ":"
    + settings.getProperty("port") + "/bigmath");
ds.setUser(settings.getProperty("dbUser"));
ds.setPassword(settings.getProperty("dbPassword"));
 
// Additional SSL-specific settings. See the source code for details.
 
Connection conn = ds.getConnection();
```

### **createDatabase**

createDatabase方法使用PostgreSQL兼容的DDL命令来创建示例数据库。

```
Statement stmt = conn.createStatement();
 
stmt.execute("CREATE TABLE IF NOT EXISTS " + TABLE_NAME +
    "(" +
    "id int PRIMARY KEY," +
    "name varchar," +
    "age int," +
    "country varchar," +
    "balance int" +
    ")");
 
stmt.execute("INSERT INTO " + TABLE_NAME + " VALUES" +
    "(1, 'Jessica', 28, 'USA', 10000)," +
    "(2, 'John', 28, 'Canada', 9000)");
```

### **selectAccounts**

selectAccounts方法使用SQL SELECT语句查询分布式数据。

```
Statement stmt = conn.createStatement();
 
ResultSet rs = stmt.executeQuery("SELECT * FROM " + TABLE_NAME);
 
while (rs.next()) {
    System.out.println(String.format("name = %s, age = %s, country = %s, balance = %s",
        rs.getString(2), rs.getString(3),
        rs.getString(4), rs.getString(5)));
}
```

### **transferMoneyBetweenAccounts**

transferMoneyBetweenAccounts 方法使用一致性分布式事务更新您的数据。

```
Statement stmt = conn.createStatement();
 
try {
    stmt.execute(
        "BEGIN TRANSACTION;" +
            "UPDATE " + TABLE_NAME + " SET balance = balance - " + amount + "" + " WHERE name = 'Jessica';" +
            "UPDATE " + TABLE_NAME + " SET balance = balance + " + amount + "" + " WHERE name = 'John';" +
            "COMMIT;"
    );
} catch (SQLException e) {
    if (e.getSQLState().equals("40001")) {
        System.err.println("The operation is aborted due to a concurrent transaction that is" +
            " modifying the same set of rows. Consider adding retry logic for production-grade applications.");
        e.printStackTrace();
    } else {
        throw e;
    }
}
```

 