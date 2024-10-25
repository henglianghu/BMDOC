## **概述**

### **支持的项目**

以下项目可用于使用BMDB  BSQL和BCQL API实现Java应用程序。

| 驱动                    | 版本        |
| ----------------------- | ----------- |
| BMDB JDBC Driver [推荐] | 42.3.5-bm-1 |
| PostgreSQL JDBC Driver  | 42.3.4      |
| Vert.x Pg Client        | 4.3.2       |
| BMDB BSQL(3.10) Driver  | 3.10.3-bm-2 |
| BMDB BCQL (4.15) Driver | 4.15.0-bm-1 |

| 项目             | APP 示例                    |
| ---------------- | --------------------------- |
| Hibernate ORM    | Hibernate ORM APP           |
| Spring Data JPA  | Spring Data JPA APP         |
| Ebean ORM        | Ebean ORM APP               |
| MyBatis ORM      | MyBatis ORM APP             |
| Spring Data BMDB | Spring Data BMDB Sample App |

### **先决条件**

要为BMDB 开发Java驱动程序应用程序，您需要以下内容： 

* Java开发工具包（JDK）
  安装JDK 8或更高版本。Linux和macOS的JDK安装程序可以从Oracle、Adoptium（OpenJDK）或Azul Systems（OpenJDK）下载。macOS上的Homebrew用户可以使用brew install openjdk进行安装。 

* 创建Java项目
  您可以使用Maven或Gradle软件项目管理工具创建Java项目。为了便于使用，可以使用集成开发环境（IDE），如IntelliJ IDEA或Eclipse IDE来配置Maven或Gradle来构建和运行您的项目。

1. 创建一个名为“DriverDemo”的项目。 

```
mvn archetype:generate \
     -DgroupId=com.bigmath\
     -DartifactId=DriverDemo \
     -DarchetypeArtifactId=maven-archetype-quickstart \
     -DinteractiveMode=false
 
cd DriverDemo
```

2.在文本编辑器中打开pom.xml文件，并在<url>元素下方添加以下内容。

```
<properties>
  <maven.compiler.source>1.8</maven.compiler.source>
  <maven.compiler.target>1.8</maven.compiler.target>
</properties>
```


如果您使用的是Java 11，它应该是：

```
 <properties>
   <maven.compiler.source>11</maven.compiler.source>
   <maven.compiler.target>11</maven.compiler.target>
 </properties>
```

* BMDB集群
  按照Install BMDB中的步骤设置一个独立的BMDB集群。 

## **应用连接**

### **BSQL**

#### BMDB JDBC智能驱动程序

BMDB JDBC智能驱动程序是一个基于PostgreSQL JDBC驱动程序的BSQL JDBC驱动程序，具有额外的连接负载平衡功能。 
对于Java应用程序，JDBC驱动程序通过Java平台上可用的标准JDBC应用程序接口（API）提供数据库连接。 
3.7.3.2.1.1.1. *CRUD操作*
以下部分演示如何执行Java应用程序开发所需的常见任务。
若要开始构建应用程序，请确保满足先决条件。
**步骤1**：设置客户端依赖关系
Maven依赖:
如果您正在使用Maven，请将以下内容添加到项目的pom.xml中。 

```
<dependency>
  <groupId>com.bigmath</groupId>
  <artifactId>jdbc-bigmathdb</artifactId>
  <version>42.3.0</version>
</dependency>
 
<!-- https://mvnrepository.com/artifact/com.zaxxer/HikariCP -->
<dependency>
  <groupId>com.zaxxer</groupId>
  <artifactId>HikariCP</artifactId>
  <version>4.0.3</version>
</dependency>
```

使用mvn install安装添加的依赖项

Gradle依赖:
如果您正在使用Gradle，请将以下依赖项添加到您的build.gradle 文件中： 

```
implementation 'com.bigmath:jdbc-bigmathdb:42.3.0'
implementation 'com.zaxxer:HikariCP:4.0.3'
```

**步骤2**：设置数据库连接
设置完依赖关系后，使用BMDB JDBC驱动程序实现Java客户端应用程序，以连接到BMDB 数据库集群并对样本数据运行查询。 

设置驱动程序属性以配置用于连接到集群的凭据和SSL证书。Java应用程序可以使用Java.sql.DriverManager类连接到BMDB 数据库并查询该数据库。使用BMDB 数据库所需的所有JDBC接口都是java.sql.*包的一部分。

使用DriverManager.getConnection方法获取BMDB 数据库的连接对象，然后可以使用该连接对象对数据库执行DDL和DML操作。 

下表描述了连接所需的连接参数，包括用于统一和拓扑负载平衡的智能驱动程序参数。 

| JDBC 参数                   | 描述                                                         | 默认值                                                       |
| --------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| hostname                    | BMDB实例的主机名，也可输入多个地址。                         | localhost                                                    |
| port                        | BSQL监听端口                                                 | 2521                                                         |
| database                    | 数据库名                                                     | bigmath                                                      |
| user                        | 连接到数据库的用户                                           | bigmath                                                      |
| password                    | 用户密码                                                     | bigmath                                                      |
| load-balance                | 负载平衡                                                     | 默认为上游驱动程序行为，除非设置为“true”                     |
| bm-servers-refresh-interval | 如果load_balance为true，则刷新服务器列表的间隔（以秒为单位） | 300                                                          |
| topology-keys               | 拓扑感知负载平衡                                             | 如果load-balance为true，则使用统一负载平衡，除非设置为cloud.region.zone形式的逗号分隔地理位置。 |

 以下是用于连接到BMDB的JDBC URL示例：

```
jdbc:bigmathdb://hostname:port/database?user=bigmath&password=bigmath&load-balance=true& \
    bm-servers-refresh-interval=240& \
    topology-keys=cloud.region.zone1,cloud.region.zone2
```

在驱动程序建立初始连接后，它会从集群中获取可用服务器的列表，并在这些服务器之间负载平衡后续的连接请求。

使用多个地址：
您可以在连接字符串中指定多个主机，以便在初始连接期间提供备用选项，以防主地址出现故障。 
使用逗号分隔地址，如下所示：

```
jdbc:bigmathdb://hostname1:port,hostname2:port,hostname3:port/database?user=bigmath&password=bigmath&load-balance=true& \
    topology-keys=cloud.region.zone1,cloud.region.zone2
```

主机仅在初始连接尝试期间使用。如果驱动程序连接时第一台主机关闭，则驱动程序会尝试连接到字符串中的下一台主机，依此类推。 

使用SSL：
下表介绍了使用SSL进行连接所需的连接参数。

| JDBC 参数   | 描述                 | 默认值         |
| ----------- | -------------------- | -------------- |
| ssl         | 启用SSL客户端连接    | false          |
| sslmode     | SSL 模式             | require        |
| sslrootcert | 计算机上根证书的路径 | ~/.postgresql/ |

以下是用于连接到启用SSL加密的BMDB集群的JDBC URL示例。

```
jdbc:bigmathdb://hostname:port/database?user=bigmath&password=bigmath&load-balance=true& \
    ssl=true&sslmode=verify-full&sslrootcert=~/.postgresql/root.crt
```


第3步：应用写入 
在项目的基本包目录中创建一个名为QuickStartApp.Java的新Java类，如下所示： 

```
touch ./src/main/java/com/bigmath/QuickStartApp.java
```

复制以下代码以设置BMDB 表，并从Java客户端查询表内容。如果需要，请确保将连接字符串bmurl替换为集群的凭据和SSL证书。

```
package com.bigmath;
 
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;
 
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;
import java.sql.ResultSet;
 
public class QuickStartApp {
  public static void main(String[] args) throws ClassNotFoundException, SQLException {
    Class.forName("com.bigmath.Driver");
    String bmurl = "jdbc:bigmathdb://127.0.0.1:2521/bigmath?user=bigmath&password=bigmath&load-balance=true";
    Connection conn = DriverManager.getConnection(bmurl);
    Statement stmt = conn.createStatement();
    try {
        System.out.println("Connected to the BMDB Cluster successfully.");
        stmt.execute("DROP TABLE IF EXISTS employee");
        stmt.execute("CREATE TABLE IF NOT EXISTS employee" +
                    "  (id int primary key, name varchar, age int, language text)");
        System.out.println("Created table employee");
 
        String insertStr = "INSERT INTO employee VALUES (1, 'John', 35, 'Java')";
        stmt.execute(insertStr);
        System.out.println("EXEC: " + insertStr);
 
        ResultSet rs = stmt.executeQuery("select * from employee");
        while (rs.next()) {
          System.out.println(String.format("Query returned: name = %s, age = %s, language = %s",
                                          rs.getString(2), rs.getString(3), rs.getString(4)));
        }
    } catch (SQLException e) {
      System.err.println(e.getMessage());
    }
  }
}
```

3.7.3.2.1.1.2. ***运行应用***
使用以下命令运行项目QuickStartApp.java： 

```
mvn -q package exec:java -DskipTests -Dexec.mainClass=com.bigmath.QuickStartApp
```

您应该看到类似于以下内容的输出：

```
Connected to the BMDB Cluster successfully.
Created table employee
Inserted data: INSERT INTO employee (id, name, age, language) VALUES (1, 'John', 35, 'Java');
Query returned: name=John, age=35, language: Java
```

如果没有收到输出或出现错误，请检查连接字符串中的参数

#### PostgreSQL JDBC驱动

PostgreSQL JDBC驱动程序是PostgreSQL的官方JDBC驱动程序，可用于连接BMDB BSQL。BSQL与PostgreSQL JDBC驱动程序完全兼容，并允许Java程序员连接到BMDB 数据库，使用标准JDBC API执行DML和DDL。

对于Java应用程序，JDBC驱动程序通过Java平台上可用的标准JDBC应用程序接口（API）提供数据库连接。

3.7.3.2.1.2.1. *CRUD操作*
以下部分演示如何执行Java应用程序开发所需的常见任务。

若要开始构建应用程序，请确保满足先决条件。

如果使用SSL构建应用程序，请执行以下附加步骤：

根据您选择的创建本地群集的平台设置SSL/TLS。要使用SSL/TLS在Minikube中设置集群，请参阅Kubernetes中集群的SSL证书。要为本地集群设置SSL证书，请参阅为Java应用程序设置SSL证书。

安装OpenSSL 1.1.1或更高版本。 

**步骤1**：设置客户端依赖关系

PostgreSQL JDBC驱动程序作为maven 依赖项提供，您可以通过将以下依赖项添加到Java项目来下载驱动程序。

Maven依赖
如果您正在使用Maven，请将以下内容添加到项目的pom.xml中。 

```
<!-- https://mvnrepository.com/artifact/org.postgresql/postgresql -->
<dependency>
  <groupId>org.postgresql</groupId>
  <artifactId>postgresql</artifactId>
  <version>42.2.14</version>
</dependency>
```

Gradle依赖
如果您正在使用Gradle，请将以下依赖项添加到您的build.Gradle文件中： 

```
// https://mvnrepository.com/artifact/org.postgresql/postgresql
implementation 'org.postgresql:postgresql:42.2.14'
```

使用mvn-Install安装添加的依赖项。

**步骤2**：设置数据库连接 

设置完依赖关系后，实现一个Java客户端应用程序，该应用程序使用PostgreSQL JDBC驱动程序连接到BMDB集群，并对样本数据运行查询。

Java应用程序可以使用Java.sql连接并查询BMDB数据库。DriverManager类。java.sql.*包包括使用BMDB所需的所有JDBC接口。


使用DriverManager.getConnection方法为BMDB数据库创建一个连接对象。这可以用于对数据库执行DDL和DML。 


| JDBC 参数 | 描述                 | 默认值    |
| --------- | -------------------- | --------- |
| hostname  | BMDB 实例的主机名    | localhost |
| port      | BSQL 监听的端口      | 2521      |
| database  | 数据库名             | bigmath   |
| user      | 连接数据库的用户名   | bigmath   |
| password  | 连接数据库的用户密码 | bigmath   |

以下是用于连接到BMDB的PostgreSQL JDBC URL格式： 

```
jdbc:postgresql://hostname:port/database
```

以下是用于连接到BMDB的JDBC URL示例： 

```
String bmurl = "jdbc:postgresql://localhost:2521/bigmath";
Connection conn = DriverManager.getConnection(bmurl , "bigmath", "bigmath");
```

**使用SSL**

下表介绍了使用SSL进行连接所需的连接参数。 

| JDBC 参数   | 描述                 | 默认值         |
| ----------- | -------------------- | -------------- |
| ssl         | 启用SSL客户端连接    | false          |
| sslmode     | SSL 模式             | require        |
| sslrootcert | 计算机上根证书的路径 | ~/.postgresql/ |

以下是用于连接到启用SSL加密的BMDB集群的JDBC URL示例。 

```
String bmurl = "jdbc:postgresql://hostname:port/database?user=bigmath&password=bigmath&ssl=true&sslmode=verify-full&sslrootcert=~/.postgresql/root.crt";
Connection conn = DriverManager.getConnection(bmurl );
```

**步骤3**：写应用 
在项目的基本包目录中创建一个名为QuickStartApp.Java的新Java类，如下所示： 

```
touch ./src/main/java/com/bigmath/QuickStartApp.java
```

复制以下代码以设置BMDB表，并从Java客户端查询表内容。如果需要，请确保将连接字符串bmurl替换为集群的凭据和SSL证书。 

```
package com.bigmath;
 
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;
import java.sql.ResultSet;
 
public class QuickStartApp {
  public static void main(String[] args) throws ClassNotFoundException, SQLException {
    Class.forName("org.postgresql.Driver");
    String bmurl= "jdbc:postgresql://localhost:2521/bigmath";
    Connection conn = DriverManager.getConnection(bmurl, "bigmath", "bigmath");
    Statement stmt = conn.createStatement();
    try {
        System.out.println("Connected to the PostgreSQL server successfully.");
        stmt.execute("DROP TABLE IF EXISTS employee");
        stmt.execute("CREATE TABLE IF NOT EXISTS employee" +
                    "  (id int primary key, name varchar, age int, language text)");
        System.out.println("Created table employee");
 
        String insertStr = "INSERT INTO employee VALUES (1, 'John', 35, 'Java')";
        stmt.execute(insertStr);
        System.out.println("EXEC: " + insertStr);
 
        ResultSet rs = stmt.executeQuery("select * from employee");
        while (rs.next()) {
          System.out.println(String.format("Query returned: name = %s, age = %s, language = %s",
                                          rs.getString(2), rs.getString(3), rs.getString(4)));
        }
    } catch (SQLException e) {
      System.err.println(e.getMessage());
    }
  }
}
```

如果使用SSL，请使用以下代码替换连接字符串bmurl： 

```
String bmurl= "jdbc:postgresql://localhost:2521/bigmath?ssl=true&sslmode=require&sslcert=src/main/resources/ssl/bigmathdb.crt.der&sslkey=src/main/resources/ssl/bigmathdb.key.pk8";
Connection conn = DriverManager.getConnection(bmurl, "bigmath", "bigmath");
```

使用以下命令运行项目QuickStartApp.java：

```
mvn -q package exec:java -DskipTests -Dexec.mainClass=com.bigmath.QuickStartApp
```

您应该看到类似于以下内容的输出： 

```
Connected to the BMDB Cluster successfully.
Created table employee
Inserted data: INSERT INTO employee (id, name, age, language) VALUES (1, 'John', 35, 'Java');
Query returned: name=John, age=35, language: Java
```

如果没有输出或出现错误，请验证Java类中的连接字符串是否具有正确的参数。


#### Vert.x Pg Client

PostgreSQL的Vert.xPg 客户端驱动程序是一个响应式和非阻塞式客户端，用于通过单线程API处理数据库连接。因为BMDB与PostgreSQL兼容，所以Vert.x PG 客户端与BMDB数据库完全兼容。
3.7.3.2.1.3.1. *CRUD操作*

以下部分演示如何使用Vert.x PG 客户端执行Java应用程序开发所需的常见任务。
若要开始构建应用程序，请确保满足先决条件。 

**步骤1**：设置客户端依赖关系 

Maven依赖

如果您正在使用Maven，请将以下内容添加到项目的pom.xml中。 

```
 <dependency>
     <groupId>io.vertx</groupId>
     <artifactId>vertx-pg-client</artifactId>
     <version>4.3.2</version>
 </dependency>
```

使用mvn install安装依赖项。

**步骤2**：设置数据库连接 

设置依赖关系后，实现一个Java客户端应用程序，该应用程序使用Vert.x Pg客户端连接到BMDB集群，并对样本数据运行查询。
Java应用程序可以使用PgPool类连接到BMDB数据库并查询该数据库。io.vertx.*包包括使用BMDB所需的所有接口。
使用PgPool.getConnection方法为BMDB数据库创建连接对象。这可以用于对数据库执行DDL和DML。
下表介绍了连接所需的连接参数。 

| PG客户端参数 | 描述                 | 默认值    |
| ------------ | -------------------- | --------- |
| setHost      | BMDB 实例的主机名    | localhost |
| setPort      | BSQL 监听的端口      | 2521      |
| setDatabase  | 数据库名             | bigmath   |
| setUser      | 连接数据库的用户名   | bigmath   |
| setPassword  | 连接数据库的用户密码 | bigmath   |

 

**步骤3**：写应用

在项目的基本包目录中创建一个名为QuickStartApp.Java的新Java类，如下所示： 

```
touch ./src/main/java/com/bigmath/QuickStartApp.java
```

复制以下代码以设置BMDB表，并从Java客户端查询表内容 

```
package com.bigmath;
 
import io.vertx.core.Promise;
import io.vertx.core.Vertx;
import io.vertx.pgclient.PgConnectOptions;
import io.vertx.pgclient.PgPool;
import io.vertx.sqlclient.PoolOptions;
import io.vertx.sqlclient.Tuple;
import io.vertx.sqlclient.Row;
import io.vertx.sqlclient.RowStream;
 
public class vertxPgExample {
    public static void main(String[] args) {
 
        PgConnectOptions options = new PgConnectOptions()
            .setPort(2521)
            .setHost("127.0.0.1")
            .setDatabase("bigmath")
            .setUser("bigmath")
            .setPassword("bigmath");
 
        Vertx vertx = Vertx.vertx();
        // creating the PgPool with configuration as option and maxsize 10.
        PgPool pool = PgPool.pool(vertx, options, new PoolOptions().setMaxSize(10));
 
        //getting a connection from the pool and running the example on that
        pool.getConnection().compose(connection -> {
            Promise<Void> promise = Promise.promise();
            // create a test table
            connection.query("create table test(id int primary key, name text)").execute()
                    .compose(v -> {
                        // insert some test data
                        return connection.query("insert into test values (1, 'Hello'), (2, 'World'), (3,'Example'), (4, 'Vertx'), (5, 'bigmath')").execute();
                    })
                    .compose(v -> {
                        // prepare the query
                        return connection.prepare("select * from test order by id");
                    })
                    .map(preparedStatement -> {
                        // create a stream for the prepared statement
                        return preparedStatement.createStream(50, Tuple.tuple());
                    })
                    .onComplete(ar -> {
                        if (ar.succeeded()) {
                            RowStream<Row> stream = ar.result();
                            stream
                                    .exceptionHandler(promise::fail)
                                    .endHandler(promise::complete)
                                    .handler(row -> System.out.println(row.toJson())); // Printing each row as JSON
                        } else {
                            promise.fail(ar.cause());
                        }
                    });
            return promise.future().onComplete(v -> {
                // close the connection
                connection.close();
            });
        }).onComplete(ar -> {
            if (ar.succeeded()) {
                System.out.println("Example ran successfully!");
            } else {
                ar.cause().printStackTrace();
            }
        });
 
    }
}
```

使用以下命令运行项目QuickStartApp.java： 

```
mvn -q package exec:java -DskipTests -Dexec.mainClass=com.bigmath.QuickStartApp
```

您应该看到类似于以下内容的输出： 

```
{"id":1,"name":"Hello"}
{"id":2,"name":"World"}
{"id":3,"name":"Example"}
{"id":4,"name":"Vertx"}
{"id":5,"name":"bigmath"}
Example ran successfully!
```


3.7.3.2.1.3.2. *限制*
BMDB目前不支持Vert.x PG客户端的Pub/sub。当向BMDB添加LISTEN/NOTIFY支持时，此限制将消失。 


### **BCQL**


#### BMDB Java驱动 (3.10)

3.7.3.2.2.1.1. *Maven*

要使用bigmath Java 驱动构建示例Java应用程序，请将以下Maven依赖项添加到应用程序中： 

```
   <dependencies>
    <dependency>
      <groupId>com.bigmath </groupId>
      <artifactId>cassandra-driver-core</artifactId>
      <version>3.10.3-bm-2</version>
    </dependency>
  </dependencies>
```

3.7.3.2.2.1.2. *创建示例Java应用程序*

3.7.3.2.2.1.2.1. 先决条件
本教程假设您具备：

* 安装了BMDB，创建了一个universe，并能够使用BCQL shell与之交互。如果没有，请按照“快速入门”中的步骤进行操作。
* 已安装JDK 1.8或更高版本。
* 已安装Maven 3.3或更高版本 

3.7.3.2.2.1.2.2. 创建项目的POM
创建一个名为pom.xml的文件，然后将以下内容复制到其中。项目对象模型（POM）包括构建项目所需的配置信息 

```
<?xml version="1.0"?>
<project
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"
  xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.bigmath.sample.apps</groupId>
  <artifactId>hello-world</artifactId>
  <version>1.0</version>
  <packaging>jar</packaging>
 
  <dependencies>
    <dependency>
      <groupId>com.bigmath</groupId>
      <artifactId>cassandra-driver-core</artifactId>
      <version>3.8.0-bm-5</version>
    </dependency>
  </dependencies>
 
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-dependency-plugin</artifactId>
        <version>2.1</version>
        <executions>
          <execution>
            <id>copy-dependencies</id>
            <phase>prepare-package</phase>
            <goals>
              <goal>copy-dependencies</goal>
            </goals>
            <configuration>
              <outputDirectory>${project.build.directory}/lib</outputDirectory>
              <overWriteReleases>true</overWriteReleases>
              <overWriteSnapshots>true</overWriteSnapshots>
              <overWriteIfNewer>true</overWriteIfNewer>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</project>
```


3.7.3.2.2.1.2.3. 编写一个示例Java应用程序

按照Maven的预期创建适当的目录结构。

```
mkdir -p src/main/java/com/bigmath/sample/apps
```

将以下内容复制到src/main/java.com/bigmath/sample/apps/cqlshHelloWorld.java文件中。

```
package com.bigmath.sample.apps;
 
import java.util.List;
import com.datastax.driver.core.Cluster;
import com.datastax.driver.core.ResultSet;
import com.datastax.driver.core.Row;
import com.datastax.driver.core.Session;
 
public class cqlshHelloWorld {
  public static void main(String[] args) {
    try {
      // Create a Cassandra client.
      Cluster cluster = Cluster.builder()
                               .addContactPoint("127.0.0.1")
                               .build();
      Session session = cluster.connect();
 
      // Create keyspace 'bmdemo' if it does not exist.
      String createKeyspace = "CREATE KEYSPACE IF NOT EXISTS bmdemo;";
      ResultSet createKeyspaceResult = session.execute(createKeyspace);
      System.out.println("Created keyspace bmdemo");
 
      // Create table 'employee' if it does not exist.
      String createTable = "CREATE TABLE IF NOT EXISTS bmdemo.employee (id int PRIMARY KEY, " +
                                                                       "name varchar, " +
                                                                       "age int, " +
                                                                       "language varchar);";
      ResultSet createResult = session.execute(createTable);
      System.out.println("Created table employee");
 
      // Insert a row.
      String insert = "INSERT INTO bmdemo.employee (id, name, age, language)" +
                                          " VALUES (1, 'John', 35, 'Java');";
      ResultSet insertResult = session.execute(insert);
      System.out.println("Inserted data: " + insert);
 
      // Query the row and print out the result.
      String select = "SELECT name, age, language FROM bmdemo.employee WHERE id = 1;";
      ResultSet selectResult = session.execute(select);
      List<Row> rows = selectResult.all();
      String name = rows.get(0).getString(0);
      int age = rows.get(0).getInt(1);
      String language = rows.get(0).getString(2);
      System.out.println("Query returned " + rows.size() + " row: " +
                         "name=" + name + ", age=" + age + ", language: " + language);
 
      // Close the client.
      session.close();
      cluster.close();
    } catch (Exception e) {
        System.err.println("Error: " + e.getMessage());
    }
  }
}
```

3.7.3.2.2.1.2.4. 构建项目

要构建项目，请运行以下mvn package命令。 

```
mvn package
```

您应该会看到一条BUILD SUCCESS消息。
3.7.3.2.2.1.2.5. 运行应用程序
要使用该应用程序，请运行以下命令。 

```
java -cp "target/hello-world-1.0.jar:target/lib/*" com.bigmath.sample.apps.cqlshHelloWorld
```

您应该看到以下内容作为输出。 

```
Created keyspace bmdemo
Created table employee
Inserted data: INSERT INTO bmdemo.employee (id, name, age, language) VALUES (1, 'John', 35, 'Java');
Query returned 1 row: name=John, age=35, language: Java
```

#### BMDB Java驱动 (4.15)

3.7.3.2.2.2.1. *Maven*
要使用Bigmath Java驱动构建示例Java应用程序，请将以下Maven依赖项添加到应用程序中：

```
 <dependency>
   <groupId>com.bigmath</groupId>
   <artifactId>java-driver-core</artifactId>
   <version>4.15.0-bm-1</version>
 </dependency>
```

3.7.3.2.2.2.2. *创建示例Java应用程序*

3.7.3.2.2.2.2.1. 先决条件

本教程假设您具备：

* 安装了BMDB，创建了一个universe，并能够使用BCQL shell与之交互。如果没有，请按照“快速入门”中的步骤进行操作。
* 已安装JDK 1.8或更高版本。
* 已安装Maven 3.3或更高版本。 

3.7.3.2.2.2.2.2. 创建项目的POM
创建一个名为pom.xml的文件，然后将以下内容复制到其中。项目对象模型（POM）包括构建项目所需的配置信息 

```
<?xml version="1.0"?>
<project
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"
  xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.bigmath.sample.apps</groupId>
  <artifactId>hello-world</artifactId>
  <version>1.0</version>
  <packaging>jar</packaging>
  <properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
  </properties>
 
  <dependencies>
    <dependency>
      <groupId>com.bigmath</groupId>
      <artifactId>java-driver-core</artifactId>
      <version>4.15.0-bm-1</version>
    </dependency>
  </dependencies>
 
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-dependency-plugin</artifactId>
        <version>2.1</version>
        <executions>
          <execution>
            <id>copy-dependencies</id>
            <phase>prepare-package</phase>
            <goals>
              <goal>copy-dependencies</goal>
            </goals>
            <configuration>
              <outputDirectory>${project.build.directory}/lib</outputDirectory>
              <overWriteReleases>true</overWriteReleases>
              <overWriteSnapshots>true</overWriteSnapshots>
              <overWriteIfNewer>true</overWriteIfNewer>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</project>
```


3.7.3.2.2.2.2.3. 编写一个示例Java应用程序

按照Maven的预期创建适当的目录结构。 

```
mkdir -p src/main/java/com/bigmath/sample/apps
```

将以下内容复制到src/main/java.com/bigmath/sample/apps/cqlshHelloWorld.java文件中。 

```
package com.bigmath.sample.apps;
import java.net.InetSocketAddress;
import java.util.List;
import com.datastax.oss.driver.api.core.CqlSession;
import com.datastax.oss.driver.api.core.cql.ResultSet;
import com.datastax.oss.driver.api.core.cql.Row;
public class cqlshHelloWorld {
    public static void main(String[] args) {
        try {
            // Create a BCQL client.
            CqlSession session = CqlSession
                .builder()
                .addContactPoint(new InetSocketAddress("127.0.0.1", 9542))
                .withLocalDatacenter("datacenter1")
                .build();
            // Create keyspace 'bmdemo' if it does not exist.
            String createKeyspace = "CREATE KEYSPACE IF NOT EXISTS bmdemo;";
            session.execute(createKeyspace);
            System.out.println("Created keyspace bmdemo");
            // Create table 'employee', if it does not exist.
            String createTable = "CREATE TABLE IF NOT EXISTS bmdemo.employee (id int PRIMARY KEY, " + "name varchar, " +
                "age int, " + "language varchar);";
            session.execute(createTable);
            System.out.println("Created table employee");
            // Insert a row.
            String insert = "INSERT INTO bmdemo.employee (id, name, age, language)" +
                " VALUES (1, 'John', 35, 'Java');";
            session.execute(insert);
            System.out.println("Inserted data: " + insert);
            // Query the row and print out the result.
            String select = "SELECT name, age, language FROM bmdemo.employee WHERE id = 1;";
            ResultSet selectResult = session.execute(select);
            List < Row > rows = selectResult.all();
            String name = rows.get(0).getString(0);
            int age = rows.get(0).getInt(1);
            String language = rows.get(0).getString(2);
            System.out.println("Query returned " + rows.size() + " row: " + "name=" + name + ", age=" + age +
                ", language: " + language);
            // Close the client.
            session.close();
        } catch (Exception e) {
            System.err.println("Error: " + e.getMessage());
        }
    }
}
```

3.7.3.2.2.2.2.4. 构建项目

要构建项目，请运行以下mvn package命令。 

```
mvn package
```

您应该会看到一条BUILD SUCCESS消息。

3.7.3.2.2.2.2.5. 运行应用程序
要使用该应用程序，请运行以下命令。 

```
java -cp "target/hello-world-1.0.jar:target/lib/*" com.bigmath.sample.apps.cqlshHelloWorld
```

您应该看到以下内容作为输出。 

```
Created keyspace bmdemo
Created table employee
Inserted data: INSERT INTO bmdemo.employee (id, name, age, language) VALUES (1, 'John', 35, 'Java');
Query returned 1 row: name=John, age=35, language: Java
```

 

## **使用ORM**

### **Hibernate ORM**

Hibernate ORM是一个用于Java应用程序的对象/关系映射（ORM）框架。Hibernate ORM关注关系数据库的数据持久性，并使开发人员能够编写的应用数据超过应用程序的生命周期。

BMDB BSQL API与Hibernate ORM完全兼容，用于Java应用程序中的数据持久性。此页面提供了开始使用Hibernate ORM连接BMDB 的详细信息

#### CRUD操作

了解如何使用Java ORM示例应用程序页面中的步骤建立与BMDB 数据库的连接，并开始基本的CRUD操作。

以下部分演示如何使用Hibernate ORM执行Java应用程序开发所需的常见任务。

**步骤1**：添加Hibernate ORM依赖项

如果您正在使用Maven，请将以下内容添加到项目的pom.xml文件中。

```
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.4.19.Final</version>
</dependency>
 
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-annotations</artifactId>
    <version>3.5.6-Final</version>
</dependency>
```

 

如果您使用的是Gradle，请将以下依赖项添加到您的build.gradle文件中： 

```
implementation 'org.hibernate:hibernate-core:5.4.19.Final'
implementation 'org.hibernate:hibernate-annotations:3.5.6-Final'
```

注意：Hibernate ORM可以与BMDB JDBC驱动程序和PostgreSQL JDBC驱动程序一起使用。

**步骤2**：为BMDB 实现ORM映射

在项目的基本包目录中创建一个名为Employee.java的文件，并为包含以下字段、setters和getters的类添加以下代码。

```
@Entity
@Table(name = "employee")
public class Employee {
 
  @Id
  Integer id;
  String name;
  Integer age;
  String language;
 
  // Setters and Getters
 
}
```

**步骤3**：为Employee对象创建一个DAO对象

在基本包目录中创建一个数据访问对象（DAO）EmployeeDAO.java。DAO用于实现域对象Employee.java的基本CRUD操作。将以下示例代码复制到您的项目中。 

```
import org.hibernate.Session;
 
public class EmployeeDAO {
 
  Session hibernateSession;
 
  public EmployeeDAO (Session session) {
    hibernateSession = session;
  }
 
  public void save(final Employee employeeEntity) {
    Transaction transaction = session.beginTransaction();
        try {
            session.save(entity);
            transaction.commit();
        } catch(RuntimeException rte) {
            transaction.rollback();
        }
        session.close();
  }
 
  public Optional<Employee> findById(final Integer id) {
    return Optional.ofNullable(session.get(Emplyee.class, id));
  }
}
```

**步骤4**：配置Hibernate属性

将hibernate配置文件hibernate.cfg.xml添加到resources目录中，并将以下内容复制到该文件中。 

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-configuration SYSTEM
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
 
<hibernate-configuration>
    <session-factory>
        <property name="hibernate.dialect">org.hibernate.dialect.PostgreSQLDialect</property>
        <property name="hibernate.connection.driver_class">com.bigmathdb.Driver</property>
        <property name="hibernate.connection.url">jdbc:bigmathdb://localhost:2521/bigmath</property>
        <property name="hibernate.connection.username">bigmath</property>
        <property name="hibernate.connection.password"></property>
        <property name="hibernate.hbm2ddl.auto">update</property>
        <property name="show_sql">true</property>
        <property name="generate-ddl">true</property>
        <property name="hibernate.ddl-auto">generate</property>
        <property name="hibernate.connection.isolation">8</property>
        <property name="hibernate.current_session_context_class">thread</property>
        <property name="javax.persistence.create-database-schemas">true</property>
    </session-factory>
</hibernate-configuration>
```

Hibernate配置文件提供了为BMDB配置Hibernate ORM所需的一组通用属性。 

| Hibernate参数                     | 描述                                    | 默认值                                  |
| --------------------------------- | --------------------------------------- | --------------------------------------- |
| hibernate.dialect                 | Dialect 用于生成针对特定数据库优化的SQL | org.hibernate.dialect.PostgreSQLDialect |
| hibernate.connection.driver_class | JDBC驱动名                              | com.bigmathdb.Driver                    |
| hibernate.connection.url          | JDBC连接URL                             | jdbc:bigmathdb://localhost:2521/bigmath |
| hibernate.connection.username     | 用户名                                  | bigmath                                 |
| hibernate.connection.password     | 密码                                    | bigmath                                 |
| hibernate.hbm2ddl.auto            | 自动模式生成的行为                      | none                                    |

Hibernate提供了一个详尽的属性列表，用于配置ORM支持的不同功能。更多的细节请参考相关的Hibernate文档。

**步骤5**：添加对象关系映射

除了用于配置Hibernate ORM的属性外，Hibernate.cfg.xml还用于使用＜mapping＞标记指定域对象映射。

在hibernate.cfg.xml中添加Employee对象的映射

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-configuration SYSTEM
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
 
<hibernate-configuration>
    <session-factory>
        ...
        <mapping class="com.bigmath.hibernatedemo.model.Employee"/>
    </session-factory>
</hibernate-configuration>
```

步骤6：使用Hibernate ORM查询BMDB集群

在项目的基本包目录中创建一个名为QuickStartOrmApp.Java的新Java类。复制以下示例代码，使用Hibernate ORM从Java客户端查询表内容。如果需要，请确保将连接字符串bmurl中的参数替换为集群凭据和SSL证书。

```
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;
 
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;
import java.util.Properties;
import java.util.Scanner;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
 
public class QuickStartOrmApp {
 
 
  public static void main(String[] args) throws ClassNotFoundException, SQLException {
 
    SessionFactory sessionFactory = HibernateUtil.getSessionFactory();
    Session session = sessionFactory.openSession();
 
    try {
          System.out.println("Connected to the BMDB Cluster successfully.");
          EmplyeeDAO employeeDAO = new EmployeeDAO(session);
          // Save an employee
          employeeDAO.save(new Employee());
 
          // Find the emplyee
          Employee employee = employeeDAO.findByID(1);
          System.out.println("Query Returned:" + employee.toString());
        }
    } catch (SQLException e) {
      System.err.println(e.getMessage());
    }
  }
}
```

当您运行项目时，QuickStartApp.java应该输出如下内容：

```
Connected to the BMDB Cluster successfully.
Created table employee
Inserted data: INSERT INTO employee (id, name, age, language) VALUES (1, 'John', 35, 'Java');
Query returned: name=John, age=35, language: Java
```

### **Ebean ORM**

Ebean ORM是一个用于Java应用程序的对象关系映射（ORM）工具。Ebean API使用无会话设计，消除了分离/附着Bean的概念，以及与冲洗和清除相关的困难。
BMDB BSQL API与Ebean ORM完全兼容，用于Java应用程序中的数据持久性。此页面提供了开始使用Ebean ORM连接BMDB 的详细信息。

Ebean ORM可以与BMDB  JDBC驱动程序和PostgreSQL JDBC驱动程序一起使用。

#### CRUD操作

了解如何使用Java ORM示例应用程序页面中的步骤建立与BMDB数据库的连接，并开始基本的CRUD操作。

以下部分演示如何使用Ebean ORM执行基于Java的Play Framework应用程序开发所需的常见任务。
3.7.3.3.2.1.1. *创建一个新的基于Java的Play Framework项目*

在开始之前，请确保已安装Java Development Kit（JDK）1.8.0或更高版本以及sbt 1.2.8或更高版本。 
1.创建一个新的Java Play项目：

```
sbt new playframework/play-java-seed.g8
```

2.出现提示时，提供以下项目信息： 

```
    Name: demo-ebean
    Organization: com.demo-ebean
    play_version: 2.8.11
    scala_version: 2.12.8
```

3.创建项目后，转到项目目录： 

```
cd demo-ebean
```

4.将项目目录下build.properties中的sbt版本更改为以下版本： 

```
sbt.version=1.2.8
```

5.下载依赖项：

```
sbt compile
```

您的新Java Play项目的文件夹结构已准备就绪。

3.7.3.3.2.1.2. *添加依赖项*

要开始在应用程序中使用Ebean，请执行以下操作：

1.在项目/plugins.sbt文件中添加以下插件：

```
addSbtPlugin("com.typesafe.sbt" % "sbt-play-ebean" % "5.0.0")
```

2.通过在conf/application.conf文件中添加以下配置，连接到BMDB：

```
db.default.driver=com.bigmath.Driver
db.default.url="jdbc:bigmathdb://127.0.0.1:2521/bsql_ebean?load-balance=true"
db.default.username=bigmath
db.default.password=""
play.evolutions {
default=true
db.default.enabled = true
autoApply=true
}
```

3.将BMDB JDBC驱动程序的以下依赖项添加到build.sbt文件中。 

```
libraryDependencies += "com.bigmath" % "jdbc-bigmathdb" % "42.3.3"
```

4.按如下方式添加PlayEbean，启用build.sbt文件中的PlayEbeam插件： 

```
lazy val root = (project in file(".")).enablePlugins(PlayJava,PlayEbean)
```


5.如果您的默认端口已经在使用，或者您想更改端口，请通过将.settings（PlayKeys.playDefaultPort:=8080）添加到build.sbt文件来修改设置，如下所示： 

```
lazy val root = (project in file(".")).enablePlugins(PlayJava,PlayEbean).settings(PlayKeys.playDefaultPort := 8080)
```

3.7.3.3.2.1.3. *通过**BMDB**使用Ebean ORM构建REST API*

示例应用程序有一个Employee模型，用于检索员工信息，包括名字、姓氏和电子邮件。EmployeeController使用Rest API在数据库中存储和检索新员工信息。 

1. 在项目的应用程序目录中创建一个模型文件夹，以存储您创建的实体。

2. 要将此目录用作默认的Ebean类包，请在conf/application.conf文件的末尾添加以下代码： 

```
ebean.default="models.*"
```

3. 在app/models/中创建Employee.java文件。这是employee的类定义。将以下代码添加到文件中： 

```
package models;
 
import io.ebean.Finder;
import io.ebean.Model;
import javax.persistence.*;
import javax.validation.constraints.NotBlank;
 
@Entity
@Table(name = "employee")
public class Employee extends Model{
 
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  @Column(columnDefinition = "serial")
  public Long empId;
 
  @NotBlank
  public String firstName;
 
  @NotBlank
  public String lastName;
 
  @Column(name = "emp_email")
  public String email;
 
  public static final Finder<Long, Employee> find = new Finder<>(Employee.class);
 
  @Override
  public String toString(){
      return "{'empId' = '"+empId+"', firstName ='"+firstName+"', 'lastName'      ='"+lastName+"', 'email' ='"+email+"' }";
 
  }
}
```


4. 在app/controllers/目录中创建EmployeeController.java文件。此文件控制员工数据流。它包含所有API调用的方法，包括添加员工和检索员工信息。使用@Transactional注释可自动管理该API中的事务。将以下代码添加到文件中：

```
package controllers;
import models.Employee;
import javax.persistence.*;
import play.libs.Json;
import play.db.ebean.Transactional;
import play.mvc.*;
import java.util.ArrayList;
import java.util.List;
public class EmployeeController extends Controller{
  @Transactional
  public Result AddEmployee(Http.Request request){
      Employee employee=Json.fromJson(request.body().asJson(),Employee.class);
      employee.save();
      return ok(Json.toJson(employee.toString()));
  }
  public Result GetAllEmployees(){
      List <Employee> employees = Employee.find.all();
      List<String> employeesList = new ArrayList<String>();
      for(int index=0;index<employees.size();index++){
          employeesList.add(employees.get(index).toString());
      }
      return ok(Json.toJson(employeesList));
  }
}
```

5. 将/employees端点的GET和POST API请求添加到conf/routes文件中。这定义了接收请求所需的方法：

```
GET      /employees            controllers.EmployeeController.GetAllEmployees
POST     /employees            controllers.EmployeeController.AddEmployee(request: Request)
```

3.7.3.3.2.1.4. *编译并运行项目*

要运行应用程序并插入新行，请执行以下步骤：

1. 使用以下命令在项目目录中编译并运行服务器：

```
sbt compile
sbt run
```

2. 使用POST请求创建employee ：

```
curl --data '{ "firstName" : "John", "lastName" : "Smith", "email":"jsmith@xyz.com" }' \
-v -X POST -H 'Content-Type:application/json' http://localhost:8080/employees
```

3. 使用Get请求获取employees 的详细信息： 

```
curl  -v -X GET http://localhost:8080/employees
```

输出应如下所示： 

```
["{'empId' = '1', firstName ='John', 'lastName' ='Smith', 'email' ='jsmith@xyz.com' }"]
```


### **MyBatis**

MyBatis是一个Java持久性框架，支持自定义SQL、存储过程和高级对象映射。MyBatis无需编写本地JDBC代码、手动结果映射和设置DB参数。MyBatis为检索数据库记录的查询到对象映射提供了简单的基于XML和注释的支持。

BMDB BSQL API与MyBatis完全兼容，用于Java应用程序中的数据持久性。本页提供了使用MyBatis构建Java应用程序以连接到BMDB 数据库的详细信息。

#### CRUD操作

学习使用MyBatis框架连接到BMDB 数据库所需的基本步骤。完整的工作应用程序记录在Java ORM示例应用程序页面上。

以下部分演示如何使用MyBatis持久性框架执行Java应用程序开发所需的常见任务。

**步骤1**：将MyBatis依赖项添加到Java项目中

在项目的pom.xml文件中使用以下Maven依赖项： 

```
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.9</version>
</dependency>
```

如果您使用的是Gradle，请将以下依赖项添加到您的build.gradle 文件中：

```
// https://mvnrepository.com/artifact/org.mybatis/mybatis
implementation 'org.mybatis:mybatis:3.5.9'
```

注意：MyBatis持久性框架可以与BMDB JDBC驱动程序和PostgreSQL JDBC驱动程序一起使用。

**步骤2**：实现实体对象

在java项目的基本包中创建一个文件User.java。添加User对象的属性以及相关的setters 和getters。 

```
public class User {
 
    private Long userId;
 
    private String firstName;
 
    private String lastName;
 
    private String email;
 
    // getters and setters
}
```

**步骤3**：为User对象创建MyBatis数据映射器

MyBatis框架使用数据映射器。数据映射器XML文件用于配置将针对实体执行的DML。通常，映射器用于定义插入、更新、删除和选择语句，它们被称为映射的SQL语句。

在Java项目的resources文件夹中创建XML文件UserMapper.XML，并复制以下内容： 

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
 
<mapper namespace="mybatis.mapper.UserMapper">
 
 <insert id="save" useGeneratedKeys = "true" parameterType = "User">
        insert into users (email, first_name, last_name) values (#{email}, #{firstName}, #{lastName})
    </insert>
    
    <resultMap id="userResultMap" type="User">
        <id property="userId" column="user_id"/>
        <result property="email" column="email"/>
        <result property="firstName" column="first_name"/>
        <result property="lastName" column="last_ame"/>
    </resultMap>
    
    <select id="findById" resultMap="userResultMap">
        select * from users where user_id = #{userId}
    </select>
    
    <select id="findAll" resultMap="userResultMap" fetchSize="10" flushCache="false" useCache="false" timeout="60000" statementType="PREPARED" resultSetType="FORWARD_ONLY">
        select * from users
    </select>
    
    <delete id = "delete" parameterType = "User">
      delete from users where user_id = #{userId};
    </delete>
    
</mapper>
```

**步骤4**：在MyBatis配置文件中配置数据映射器和数据源

所有数据映射器都必须在MyBatis配置文件中定义。在resources文件夹中创建mybatis-config.xml以配置MyBatis 框架。

在mybatis-config.xml中，定义用户数据映射器和用于连接到BMDB数据库的数据源。 

```
<?xml version="1.0" encoding="UTF-8" ?>
<!-- Mybatis config sample -->
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties>
        <!-- enabling default property values -->
        <property name="org.apache.ibatis.parsing.PropertyParser.enable-default-value" value="true"/>
    </properties>
    <settings>
        <setting name="defaultFetchSize" value="100"/>
    </settings>
    <typeAliases>
        <typeAlias type="User" alias="User"/>
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC" />
            <dataSource type="POOLED">
                <property name="driver" value="com.bigmath.Driver" />
                <property name="url" value="jdbc:bigmathdb://127.0.0.1:2521/bigmath" />
                <!-- default property values support -->
                <property name="username" value="${db.username:bigmath}" />
                <property name="password" value="${db.password:}" />
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="UserMapper.xml" />
    </mappers>
</configuration>
```

**步骤5**：创建MyBatis SQLSessionFactory对象

SQLSession提供了执行数据库操作、检索映射器和结果集映射等方法。

SQLSessionFactory不是线程安全的，所以您需要一种线程安全的方式来实例化SQLSession。在基本包中创建MyBatisUtil.java类，以实现创建SQLSessionFactory的线程安全方式。 

```
import java.io.IOException;
import java.io.InputStream;
 
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
 
public class MybatisUtil {
 
 private static SqlSessionFactory sqlSessionFactory;
 
 public static SqlSessionFactory getSessionFactory() {
        String resource = "mybatis-config.xml";
        InputStream inputStream;
  try {
   inputStream = Resources.getResourceAsStream(resource);
   sqlSessionFactory =
             new SqlSessionFactoryBuilder().build(inputStream);
  } catch (IOException e) {
   e.printStackTrace();
  }
  return sqlSessionFactory;
    }
}
```

**步骤6**：为User对象创建一个DAO对象

在基本包中创建一个数据访问对象（DAO）UserDAO.java。DAO用于实现域对象User.java的基本CRUD操作。

将以下代码复制到您的项目中： 

```
import java.util.List;
 
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
 
public class UserDAO {
 
    private SqlSessionFactory sqlSessionFactory;
 
    public UserDAO(SqlSessionFactory sqlSessionFactory) {
        this.sqlSessionFactory = sqlSessionFactory;
    }
 
    public void save(final User entity) {
     
        try (SqlSession session = sqlSessionFactory.openSession()) {
         session.insert("mybatis.mapper.UserMapper.save", entity);
         session.commit();
         } catch (RuntimeException rte) {} 
    }
 
    public User findById(final Long id) {
     
     User user = null;
 
        try (SqlSession session = sqlSessionFactory.openSession()) {
         user =  session.selectOne("mybatis.mapper.UserMapper.findById", id);
         
        } catch (RuntimeException rte) {} 
        
        return user;
    }
 
    public List<User> findAll() {
     
     List<User> users = null;
        try (SqlSession session = sqlSessionFactory.openSession()) {
            users = session.selectList("mybatis.mapper.UserMapper.findAll");
        } catch (RuntimeException rte) {}
     
     return users;
    }
 
    public void delete(final User user) {
 
        try (SqlSession session = sqlSessionFactory.openSession()) {
            session.delete("mybatis.mapper.UserMapper.delete", user.getUserId());
        } catch (RuntimeException rte) {}
    }
 
}
```

**步骤7**：使用MyBatis Framework查询BMDB集群

在项目的基本包中创建一个java类MyBatisExample.java。以下示例代码插入一个用户记录，并使用MyBatis查询表内容。

```
import java.sql.SQLException;
 
import org.apache.ibatis.session.SqlSessionFactory;
 
public class MyBatisExample {
 
   public static void main(String[] args) throws ClassNotFoundException, SQLException {
 
    SqlSessionFactory sessionFactory = MybatisUtil.getSessionFactory();
 
      System.out.println("Connected to the BMDB Cluster successfully.");
     UserDAO userDAO = new UserDAO(sessionFactory);
     User user = new User();
     user.setEmail("demo@bigmath.com");
     user.setFirstName("Alice");
     user.setLastName("bigmathbeing");
     
     // Save an user
     userDAO.save(user);
     System.out.println("Inserted user record: " + user.getFirstName());
 
     // Find the user
     User userFromDB = userDAO.findById(new Long(201));
     System.out.println("Query returned:" + userFromDB.toString());
    }
 
}
```

当您运行Java项目时，MyBatisExample.Java应该输出以下内容： 

```
Connected to the BMDB Cluster successfully.
Inserted user record: Alice
Query returned:User [userId=101, firstName=Alice, lastName=bigmathbeing, email=demo@bigmath.com]
```
