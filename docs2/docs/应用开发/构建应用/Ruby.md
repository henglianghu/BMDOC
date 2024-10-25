
以下教程显示了一个小型Ruby应用程序，该应用程序使用Ruby Pg驱动程序连接到BMDB集群，并执行基本的SQL操作。


## **先决条件**

* Ruby 3.1 或更新版本。
* OpenSSL 1.1.1或更高版本（由libpq和pg用于建立安全的SSL连接）。
* libpq. macOS上的Homebrew用户可以使用brew install libpq进行安装。您可以从PostgreSQL Downloads中下载PostgreSQL二进制文件和源代码。
* Ruby pg. 要安装Ruby pg，请运行以下命令：

```
gem install pg -- --with-pg-include=<path-to-libpq>/libpq/include --with-pg-lib=<path-to-libpq>/libpq/lib
```

将＜path-to-libpq＞替换为libpq 安装的路径；例如/usr/local/opt。 

从GitLab克隆应用程序：

```
git clone https://gitlab.bigmath.com/bigmath-Samples/bigmath-simple-ruby-app.git && cd bigmath-simple-ruby-app
```


## **构建并运行应用**

确认程序文件可执行。

```
chmod +x sample-app.rb 
```

运行应用程序。

```
./sample-app.rb 
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

您已经成功地执行了一个基本Ruby应用程序。


## **分析应用程序逻辑**

打开bigmath-simple-ruby-app 文件夹中的sample-app.rb文件，查看方法


### **connect**

connect方法通过libpqxx驱动程序与集群建立连接。


```
conn = PG.connect(
    host: '',
    port: '2521',
    dbname: 'bigmath',
    user: '',
    password: '',
    sslmode: 'verify-full',
    sslrootcert: ''
    );
```


### **create_database**

create_database方法使用PostgreSQL兼容的DDL命令来创建一个示例数据库。

```
conn.exec("DROP TABLE IF EXISTS DemoAccount");
 
conn.exec("CREATE TABLE DemoAccount ( \
            id int PRIMARY KEY, \
            name varchar, \
            age int, \
            country varchar, \
            balance int)");
 
conn.exec("INSERT INTO DemoAccount VALUES \
            (1, 'Jessica', 28, 'USA', 10000), \
            (2, 'John', 28, 'Canada', 9000)");
```

### **select_accounts**

select_accounts方法使用SQL SELECT语句查询分布式数据。 

```
begin
    puts ">>>> Selecting accounts:\n";
 
    rs = conn.exec("SELECT name, age, country, balance FROM DemoAccount");
 
    rs.each do |row|
        puts "name=%s, age=%s, country=%s, balance=%s\n" % [row['name'], row['age'], row['country'], row['balance']];
    end
 
ensure
    rs.clear if rs
end
```

### **transfer_money_between_accounts**

transferMoneyBetweenAccounts方法使用一致性分布式事务更新您的数据。

```
begin
    conn.transaction do |txn|
        txn.exec_params("UPDATE DemoAccount SET balance = balance - $1 WHERE name = \'Jessica\'", [amount]);
        txn.exec_params("UPDATE DemoAccount SET balance = balance + $1 WHERE name = \'John\'", [amount]);
    end
 
    puts ">>>> Transferred %s between accounts.\n" % [amount];
 
rescue PG::TRSerializationFailure => e
    puts "The operation is aborted due to a concurrent transaction that is modifying the same set of rows. \
            Consider adding retry logic for production-grade applications.";
    raise
end
```
