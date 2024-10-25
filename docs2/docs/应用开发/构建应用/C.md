
以下教程显示了一个小型C应用程序，该应用程序使用libpq 驱动程序连接到BMDB集群，并执行基本的SQL操作。


## **先决条件**

* 32位(x86)或64位(x64)体系结构的计算机。（使用Rosetta 构建和运行在苹果silicon上。）

* gcc 4.1.2或更新版本，或clang 3.4或更新版本。

* OpenSSL 1.1.1或更高版本（由libpq用于建立安全的SSL连接）。

* libpq。macOS上的Homebrew用户可以使用brew install libpq进行安装。您可以从PostgreSQL Downloads中下载PostgreSQL二进制文件和源代码。

从GitLab克隆应用程序：

```
git clone https://gitlab.bigmath.com/bigmath-Samples/bigmath-simple-c-app.git && cd bigmath-simple-c-app
```


## **构建并运行应用**

使用gcc或clang构建应用程序。

```
gcc sample-app.c -o sample-app -I<path-to-libpq>/libpq/include -L<path-to-libpq>/libpq/lib -lpq
```

将＜path-to-libpq＞替换为libpq 安装的路径；例如/usr/local/opt。 

启动应用

```
./sample-app
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

您已经成功地执行了一个基本C应用程序。

## **分析应用程序逻辑**

打开bigmath-simple-c-app文件夹中的sample-app.c文件，查看方法


### **connect**

connect方法通过libpq驱动程序与集群建立连接。

```
PQinitSSL(1);
 
conn = PQconnectdb(CONN_STR);
 
if (PQstatus(conn) != CONNECTION_OK) {
    printErrorAndExit(conn, NULL);
}
 
printf(">>>> Successfully connected to BMDB!\n");
 
return conn;
```


### **createDatabase**

createDatabase方法使用PostgreSQL兼容的DDL命令来创建一个示例数据库。

```
res = PQexec(conn, "DROP TABLE IF EXISTS DemoAccount");
 
if (PQresultStatus(res) != PGRES_COMMAND_OK) {
    printErrorAndExit(conn, res);
}
 
res = PQexec(conn, "CREATE TABLE DemoAccount ( \
                    id int PRIMARY KEY, \
                    name varchar, \
                    age int, \
                    country varchar, \
                    balance int)");
 
if (PQresultStatus(res) != PGRES_COMMAND_OK) {
    printErrorAndExit(conn, res);
}
 
res = PQexec(conn, "INSERT INTO DemoAccount VALUES \
                    (1, 'Jessica', 28, 'USA', 10000), \
                    (2, 'John', 28, 'Canada', 9000)");
 
if (PQresultStatus(res) != PGRES_COMMAND_OK) {
    printErrorAndExit(conn, res);
}
 
```


### **selectAccounts**

selectAccounts方法使用SQL SELECT语句查询分布式数据。 

```
res = PQexec(conn, "SELECT name, age, country, balance FROM DemoAccount");
 
if (PQresultStatus(res) != PGRES_TUPLES_OK) {
    printErrorAndExit(conn, res);
}
 
for (i = 0; i < PQntuples(res); i++) {
    printf("name = %s, age = %s, country = %s, balance = %s\n",
        PQgetvalue(res, i, 0), PQgetvalue(res, i, 1), PQgetvalue(res, i, 2), PQgetvalue(res, i, 3));
}
```

### **transferMoneyBetweenAccounts**

transferMoneyBetweenAccounts方法使用一致性分布式事务更新您的数据。

```
res = PQexec(conn, "BEGIN TRANSACTION");
if (PQresultStatus(res) != PGRES_COMMAND_OK) {
    printErrorAndExit(conn, res);
}
 
res = PQexec(conn, "UPDATE DemoAccount SET balance = balance - 800 WHERE name = \'Jessica\'");
if (PQresultStatus(res) != PGRES_COMMAND_OK) {
    printErrorAndExit(conn, res);
}
 
res = PQexec(conn, "UPDATE DemoAccount SET balance = balance + 800 WHERE name = \'John\'");
if (PQresultStatus(res) != PGRES_COMMAND_OK) {
    printErrorAndExit(conn, res);
}
 
res = PQexec(conn, "COMMIT");
if (PQresultStatus(res) != PGRES_COMMAND_OK) {
    printErrorAndExit(conn, res);
}
```

 