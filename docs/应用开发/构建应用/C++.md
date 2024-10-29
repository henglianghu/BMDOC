
以下教程显示了一个小型C++应用程序，该应用程序使用libpqxx驱动程序连接到BMDB集群，并执行基本的SQL操作。


## **先决条件**

* 32位(x86)或64位(x64)体系结构的计算机。（使用Rosetta 构建和运行在苹果silicon上。）
* gcc 4.1.2或更新版本，或clang 3.4或更新版本。
* OpenSSL 1.1.1或更高版本（由libpqxx用于建立安全的SSL连接）。
* libpq。macOS上的Homebrew用户可以使用brew install libpq进行安装。您可以从PostgreSQL Downloads中下载PostgreSQL二进制文件和源代码。
* libpqxx. macOS上的Homebrew用户可以使用brew install libpqxx进行安装。参考构建libpqxx来构建自己的驱动。

从GitLab克隆应用程序：

```
git clone https://gitlab.bigmath.com/AiSQL-Samples/bigmath-simple-cpp-app.git && cd bigmath-simple-cpp-app
```


## **构建并运行应用**

使用gcc或clang构建应用程序。

```
g++ -std=c++17 sample-app.cpp -o sample-app -lpqxx -lpq \
-I<path-to-libpq>/libpq/include -I<path-to-libpqxx>/libpqxx/include \
-L<path-to-libpq>/libpq/lib -L<path-to-libpqxx>/libpqxx/lib
```

将＜path-to-libpq＞替换为libpq 安装的路径；＜path-to-libpqxx＞替换为libpqxx 安装的路径。例如/usr/local/opt。 

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

您已经成功地执行了一个基本C++应用程序。

 

## **分析应用程序逻辑**

打开bigmath-simple-cpp-app文件夹中的sample-app.cpp文件，查看方法


### **connect**

connect方法通过libpqxx 驱动程序与集群建立连接。

```
std::string url = "host=" + HOST + " port=" + PORT + " dbname=" + DB_NAME +
    " user=" + USER + " password=" + PASSWORD;
 
if (SSL_MODE != "") {
    url += " sslmode=" + SSL_MODE;
 
    if (SSL_ROOT_CERT != "") {
        url += " sslrootcert=" + SSL_ROOT_CERT;
    }
}
 
std::cout << ">>>> Connecting to BMDB!" << std::endl;
 
pqxx::connection *conn = new pqxx::connection(url);
 
std::cout << ">>>> Successfully connected to BMDB!" << std::endl;
```


### **createDatabase**

createDatabase方法使用PostgreSQL兼容的DDL命令来创建一个示例数据库。

```
pqxx::work txn(*conn);
 
txn.exec("DROP TABLE IF EXISTS DemoAccount");
 
txn.exec("CREATE TABLE DemoAccount ( \
            id int PRIMARY KEY, \
            name varchar, \
            age int, \
            country varchar, \
            balance int)");
 
txn.exec("INSERT INTO DemoAccount VALUES \
            (1, 'Jessica', 28, 'USA', 10000), \
            (2, 'John', 28, 'Canada', 9000)");
 
txn.commit();
```

### **selectAccounts**

selectAccounts方法使用SQL SELECT语句查询分布式数据。 

```
res = txn.exec("SELECT name, age, country, balance FROM DemoAccount");
 
for (auto row: res) {
    std::cout
        << "name=" << row["name"].c_str() << ", "
        << "age=" << row["age"].as<int>() << ", "
        << "country=" << row["country"].c_str() << ", "
        << "balance=" << row["balance"].as<int>() << std::endl;
}
```

### **transferMoneyBetweenAccounts**

transferMoneyBetweenAccounts方法使用一致性分布式事务更新您的数据。

```
try {
    pqxx::work txn(*conn);
 
    txn.exec("UPDATE DemoAccount SET balance = balance -" + std::to_string(amount)
        + " WHERE name = \'Jessica\'");
 
    txn.exec("UPDATE DemoAccount SET balance = balance +" + std::to_string(amount)
        + " WHERE name = \'John\'");
 
    txn.commit();
 
    std::cout << ">>>> Transferred " << amount << " between accounts." << std::endl;
} catch (pqxx::sql_error const &e) {
    if (e.sqlstate().compare("40001") == 0) {
        std::cerr << "The operation is aborted due to a concurrent transaction that is modifying the same set of rows."
                    << "Consider adding retry logic for production-grade applications." << std::endl;
    }
    throw e;
}
```
