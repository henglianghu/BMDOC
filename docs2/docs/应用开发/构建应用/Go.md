
下面的教程展示了一个小型Go应用程序，它使用Go PostgreSQL驱动程序连接到BMDB集群，并执行基本的SQL操作。

## **先决条件**

Go 版本：1.17.6
从GitLab克隆应用程序
将示例应用程序克隆到您的计算机：

```
git clone https://gitlab.bigmath.com/bigmath-Samples/bigmath-simple-go-app.git && cd bigmath-simple-go-app
```

## **构建并运行应用**

首先初始化GO111MODULE变量

```
mvn export GO111MODULE=auto
```

导入Go PostgreSQL驱动

```
go get gitlab.bigmath.com/lib/pq
```

运行应用

```
go run sample-app.go
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


## **分析应用程序逻辑**

打开bigmath-simple-go-app文件夹中的sample-app.go文件，查看方法

### **main**

main方法通过Go PostgreSQL驱动程序与集群建立连接。

```
psqlInfo := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s",
    host, port, dbUser, dbPassword, dbName)
 
if sslMode != "" {
    psqlInfo += fmt.Sprintf(" sslmode=%s", sslMode)
 
    if sslRootCert != "" {
        psqlInfo += fmt.Sprintf(" sslrootcert=%s", sslRootCert)
    }
}
 
db, err := sql.Open("postgres", psqlInfo)
```

### **createDatabase**

createDatabase方法使用PostgreSQL兼容的DDL命令来创建一个示例数据库。

```
Statement stmt = conn.createStatement();
 
stmt := `DROP TABLE IF EXISTS DemoAccount`
_, err := db.Exec(stmt)
checkIfError(err)
 
stmt = `CREATE TABLE DemoAccount (
                      id int PRIMARY KEY,
                      name varchar,
                      age int,
                      country varchar,
                      balance int)`
 
_, err = db.Exec(stmt)
checkIfError(err)
 
stmt = `INSERT INTO DemoAccount VALUES
              (1, 'Jessica', 28, 'USA', 10000),
              (2, 'John', 28, 'Canada', 9000)`
 
_, err = db.Exec(stmt)
checkIfError(err)
```

### **selectAccounts**

selectAccounts方法使用SQL SELECT语句查询分布式数据。 

```
rows, err := db.Query("SELECT name, age, country, balance FROM DemoAccount")
checkIfError(err)
 
defer rows.Close()
 
var name, country string
var age, balance int
 
for rows.Next() {
    err = rows.Scan(&name, &age, &country, &balance)
    checkIfError(err)
 
    fmt.Printf("name = %s, age = %v, country = %s, balance = %v\n",
        name, age, country, balance)
}
```


### **transferMoneyBetweenAccounts**

transferMoneyBetweenAccounts方法使用一致性分布式事务更新您的数据。

```
tx, err := db.Begin()
checkIfError(err)
 
_, err = tx.Exec(`UPDATE DemoAccount SET balance = balance - $1 WHERE name = 'Jessica'`, amount)
if checkIfTxAborted(err) {
    return
}
_, err = tx.Exec(`UPDATE DemoAccount SET balance = balance + $1 WHERE name = 'John'`, amount)
if checkIfTxAborted(err) {
    return
}
 
err = tx.Commit()
if checkIfTxAborted(err) {
    return
}
```
