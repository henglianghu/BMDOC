
以下教程显示了一个小型Node.js应用程序，该应用程序使用node-postgres模块连接到BMDB集群，并执行基本的SQL操作。

## **先决条件**

Node.js的最后版本。

从GitLab克隆应用程序：

```
git clone https://gitlab.bigmath.com/bigmath-Samples/bigmath-simple-node-app.git && cd bigmath-simple-node-app
```


## **构建并运行应用**

安装 node-postgres模块。

```
npm install pg
```

安装async 实用程序

```
npm install --save async
```

启动应用

```
node sample-app.js
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

您已经成功地执行了一个基本Python应用程序。

## **分析应用程序逻辑**

打开bigmath-simple-node-app文件夹中的sample-app.js文件，查看方法

### **connect**

connect方法通过node-postgres驱动程序与集群建立连接。

```
try {
    client = new pg.Client(config);
 
    await client.connect();
 
    console.log('>>>> Connected to BMDB!');
 
    callbackHadler();
} catch (err) {
    callbackHadler(err);
}
```

### **createDatabase**

createDatabase方法使用PostgreSQL兼容的DDL命令来创建一个示例数据库。

```
try {
    var stmt = 'DROP TABLE IF EXISTS DemoAccount';
 
    await client.query(stmt);
 
    stmt = `CREATE TABLE DemoAccount (
        id int PRIMARY KEY,
        name varchar,
        age int,
        country varchar,
        balance int)`;
 
    await client.query(stmt);
 
    stmt = `INSERT INTO DemoAccount VALUES
        (1, 'Jessica', 28, 'USA', 10000),
        (2, 'John', 28, 'Canada', 9000)`;
 
    await client.query(stmt);
 
    console.log('>>>> Successfully created table DemoAccount.');
 
    callbackHadler();
} catch (err) {
    callbackHadler(err);
}
```

### **selectAccounts**

selectAccounts方法使用SQL SELECT语句查询分布式数据。 

```
try {
    const res = await client.query('SELECT name, age, country, balance FROM DemoAccount');
    var row;
 
    for (i = 0; i < res.rows.length; i++) {
        row = res.rows[i];
 
        console.log('name = %s, age = %d, country = %s, balance = %d',
            row.name, row.age, row.country, row.balance);
    }
 
    callbackHadler();
} catch (err) {
    callbackHadler(err);
}
```

### **transferMoneyBetweenAccounts**

transferMoneyBetweenAccounts方法使用一致性分布式事务更新您的数据。

```
try {
    await client.query('BEGIN TRANSACTION');
 
    await client.query('UPDATE DemoAccount SET balance = balance - ' + amount + ' WHERE name = \'Jessica\'');
    await client.query('UPDATE DemoAccount SET balance = balance + ' + amount + ' WHERE name = \'John\'');
    await client.query('COMMIT');
 
    console.log('>>>> Transferred %d between accounts.', amount);
 
    callbackHadler();
} catch (err) {
    callbackHadler(err);
}
```
