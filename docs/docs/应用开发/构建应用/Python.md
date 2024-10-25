
以下教程显示了一个小型Python应用程序，该应用程序使用Python psycopg2 PostgreSQL数据库适配器连接到BMDB集群，并执行基本的SQL操作。

## **先决条件**

Python 3.6或更高版本（如果在Apple silicon上运行macOS，则为Python 3.9.7或更高）。

从GitLab克隆应用程序：

```
git clone https://gitlab.bigmath.com/bigmath-Samples/bigmath-simple-python-app.git && cd bigmath-simple-python-app
```

## **构建并运行应用**

安装psycopg2 PostgreSQL数据库适配器。

```
pip3 install psycopg2-binary
```

启动应用

```
python3 sample-app.py
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

打开bigmath-simple-python-app文件夹中的sample-app.py文件，查看方法

### **main**

main方法通过Python PostgreSQL驱动程序与集群建立连接。

```
try:
    if conf['sslMode'] != '':
        bm = psycopg2.connect(host=conf['host'], port=conf['port'], database=conf['dbName'],
                                user=conf['dbUser'], password=conf['dbPassword'],
                                sslmode=conf['sslMode'], sslrootcert=conf['sslRootCert'],
                                connect_timeout=10)
    else:
        bm = psycopg2.connect(host=conf['host'], port=conf['port'], database=conf['dbName'],
                                user=conf['dbUser'], password=conf['dbPassword'],
                                connect_timeout=10)
except Exception as e:
    print("Exception while connecting to BMDB")
    print(e)
    exit(1)
```

### **createDatabase**

createDatabase方法使用PostgreSQL兼容的DDL命令来创建一个示例数据库。

```
try:
    with bm.cursor() as bm_cursor:
        bm_cursor.execute('DROP TABLE IF EXISTS DemoAccount')
 
        create_table_stmt = """
            CREATE TABLE DemoAccount (
                id int PRIMARY KEY,
                name varchar,
                age int,
                country varchar,
                balance int
            )"""
        bm_cursor.execute(create_table_stmt)
 
        insert_stmt = """
            INSERT INTO DemoAccount VALUES
                    (1, 'Jessica', 28, 'USA', 10000),
                    (2, 'John', 28, 'Canada', 9000)"""
        bm_cursor.execute(insert_stmt)
    bm.commit()
except Exception as e:
    print("Exception while creating tables")
    print(e)
    exit(1)
```

### **select_accounts**

select_accounts方法使用SQL SELECT语句查询分布式数据。

```
with bm.cursor(cursor_factory=psycopg2.extras.RealDictCursor) as bm_cursor:
        bm_cursor.execute("SELECT name, age, country, balance FROM DemoAccount")
        results = bm_cursor.fetchall()
        for row in results:
            print("name = {name}, age = {age}, country = {country}, balance = {balance}".format(**row))
```


### **transfer_money_between_accounts**

transfer_money_between_accounts方法使用一致性分布式事务更新您的数据。

```
try:
    with bm.cursor() as bm_cursor:
        bm_cursor.execute("UPDATE DemoAccount SET balance = balance - %s WHERE name = 'Jessica'", [amount])
        bm_cursor.execute("UPDATE DemoAccount SET balance = balance + %s WHERE name = 'John'", [amount])
    bm.commit()
except (Exception, psycopg2.DatabaseError) as e:
    print("Exception while transferring money")
    print(e)
    if e.pgcode == 40001:
        print("The operation is aborted due to a concurrent transaction that is modifying the same set of rows." +
                "Consider adding retry logic for production-grade applications.")
    exit(1)
```
