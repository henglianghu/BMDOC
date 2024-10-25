
以下教程显示了一个小型PHP应用程序，该应用程序使用php-pgsql 驱动程序连接到BMDB集群，并执行基本的SQL操作。

## **先决条件**

* PHP运行时。示例应用程序是使用PHP8.1创建的，但应该可以与早期和更高版本一起使用。
* php-pgsql驱动
  * Ubuntu用户可以使用sudo apt-get install php-pgsql命令安装驱动程序。
  * CentOS用户可以使用sudo yum install php-pgsql命令安装驱动程序。


从GitLab克隆应用程序：

```
git clone https://gitlab.bigmath.com/bigmath-Samples/bigmath-simple-php-app.git && cd bigmath-simple-php-app
```


## **运行应用**

运行应用程序。

```
php sample-app.php 
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

您已经成功地执行了一个基本PHP应用程序。


## **分析应用程序逻辑**

打开bigmath-simple-php-app文件夹中的sample-app.php文件，查看方法


### **connect**

connect方法通过php-pgsql驱动程序与集群建立连接。


```
$conn = new PDO('pgsql:host=' . HOST . ';port=' . PORT . ';dbname=' . DB_NAME .
                ';sslmode=' . SSL_MODE . ';sslrootcert=' . SSL_ROOT_CERT,
                USER, PASSWORD,
                array(PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                        PDO::ATTR_EMULATE_PREPARES => true,
                        PDO::ATTR_PERSISTENT => true));
```


### **create_database**

create_database方法使用PostgreSQL兼容的DDL命令来创建一个示例数据库。

```
$conn->exec('DROP TABLE IF EXISTS DemoAccount');
 
$conn->exec('CREATE TABLE DemoAccount (
                id int PRIMARY KEY,
                name varchar,
                age int,
                country varchar,
                balance int)');
 
$conn->exec("INSERT INTO DemoAccount VALUES
                (1, 'Jessica', 28, 'USA', 10000),
                (2, 'John', 28, 'Canada', 9000)");
```

### **select_accounts**

select_accounts方法使用SQL SELECT语句查询分布式数据。 

```
$query = 'SELECT name, age, country, balance FROM DemoAccount';
 
foreach ($conn->query($query) as $row) {
    print 'name=' . $row['name'] . ', age=' . $row['age'] . ', country=' . $row['country'] . ', balance=' . $row['balance'] . "\n";
}
```

### **transfer_money_between_accounts**

transfer_money_between_accounts方法使用一致性分布式事务更新您的数据。

```
try {
    $conn->beginTransaction();
    $conn->exec("UPDATE DemoAccount SET balance = balance - " . $amount . " WHERE name = 'Jessica'");
    $conn->exec("UPDATE DemoAccount SET balance = balance + " . $amount . " WHERE name = 'John'");
    $conn->commit();
    print ">>>> Transferred " . $amount . " between accounts\n";
} catch (PDOException $e) {
    if ($e->getCode() == '40001') {
        print "The operation is aborted due to a concurrent transaction that is modifying the same set of rows.
                Consider adding retry logic for production-grade applications.\n";
    }
 
    throw $e;
}
```
