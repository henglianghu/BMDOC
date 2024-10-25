
以下教程显示了一个小型Rust应用程序，该应用程序使用Rust-Postgres驱动程序连接到BMDB集群，并执行基本的SQL操作。


## **先决条件**

Rust开发环境。该示例应用程序是为Rust 1.58创建的，但应适用于早期和更高版本。

从GitLab克隆应用程序：

```
git clone https://gitlab.bigmath.com/bigmath-Samples/bigmath-simple-rust-app.git && cd bigmath-simple-rust-app
```

 

## **分析应用程序逻辑**

打开bigmath-simple-rust-app/src 文件夹中的sample-app.rs 文件，查看方法


### **connect**

connect方法通过Rust-Postgres驱动程序与集群建立连接。


```
let mut cfg = Config::new();
 
cfg.host(HOST).port(PORT).dbname(DB_NAME).
    user(USER).password(PASSWORD).ssl_mode(SSL_MODE);
 
let mut builder = SslConnector::builder(SslMethod::tls())?;
builder.set_ca_file(SSL_ROOT_CERT)?;
let connector = MakeTlsConnector::new(builder.build());
 
let client = cfg.connect(connector)?;
```


### **create_database**

create_database方法使用PostgreSQL兼容的DDL命令来创建一个示例数据库。

```
client.execute("DROP TABLE IF EXISTS DemoAccount", &[])?;
 
client.execute("CREATE TABLE DemoAccount (
                id int PRIMARY KEY,
                name varchar,
                age int,
                country varchar,
                balance int)", &[])?;
 
client.execute("INSERT INTO DemoAccount VALUES
                (1, 'Jessica', 28, 'USA', 10000),
                (2, 'John', 28, 'Canada', 9000)", &[])?;
```

### **select_accounts**

select_accounts方法使用SQL SELECT语句查询分布式数据。 

```
for row in client.query("SELECT name, age, country, balance FROM DemoAccount", &[])? {
    let name: &str = row.get("name");
    let age: i32 = row.get("age");
    let country: &str = row.get("country");
    let balance: i32 = row.get("balance");
 
    println!("name = {}, age = {}, country = {}, balance = {}",
        name, age, country, balance);
}
```

### **transfer_money_between_accounts**

transfer_money_between_accounts方法使用一致性分布式事务更新您的数据。

```
let mut txn = client.transaction()?;
 
let exec_txn = || -> Result<(), DBError> {
    txn.execute("UPDATE DemoAccount SET balance = balance - $1 WHERE name = \'Jessica\'", &[&amount])?;
    txn.execute("UPDATE DemoAccount SET balance = balance + $1 WHERE name = \'John\'", &[&amount])?;
    txn.commit()?;
 
    Ok(())
};
```

