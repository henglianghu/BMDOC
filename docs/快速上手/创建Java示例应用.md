## 应用程序说明

本章通过编写一个简洁明了的Java程序，简单演示应用程序对AiSQL数据库进行的操作。该程序首先实现了与AiSQL数据库的连接，成功建立连接后，程序接着向指定的数据库表中插入一条记录。随后，为了验证数据是否已正确插入并展示数据库表的内容，程序执行了一个查询操作。最后，作为操作的总结，程序将查询到的数据库表数据逐条打印输出到控制台，整个过程演示了Java与AiSQL数据库交互的基本流程，方便java开发者快速上手。



## 前置工作

在正式着手编写、调试应用程序之前，有一系列至关重要的前期准备工作需要妥善完成，这些工作围绕着 AiSQL 数据库的安装、配置、用户创建、数据库创建、表创建等一下工作，具体图下。

### 数据准备

首先，要确保将 AiSQL 成功安装到合适的环境中。安装完成后，紧接着需要启动 AiSQL 数据库服务，使其处于可运行状态，随后，登录到 AiSQL 数据库系统。在登录后，创建一个名为 “aiuser” 的用户，这个用户将在后续的应用程序中扮演重要角色。为了确保 “aiuser” 能够充分发挥其作用，需要为其赋予对特定数据库的全面操作权限。具体而言，创建一个名为 “testdb” 的数据库，并赋予 “aiuser” 对testdb数据库的所有权限，涵盖数据的插入、查询、更新、删除以及对数据库结构的修改等操作，使其能够在 “testdb” 数据库中自由地进行各种数据管理和操作任务，最后在testdb数据库下创建一张员工信息表。具体涉及的SQL语句如下：

```
CREATE DATABASE testdb;
CREATE USER aiuser WITH PASSWORD '123456';
\c testdb;
GRANT SELECT, INSERT ON ALL TABLES IN SCHEMA public TO aiuser;
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL,
    hire_date DATE NOT NULL,
    job_title VARCHAR(100),
    salary NUMERIC(10,2),
    department_id INT
);
```

登录AiSQL之后，可以按顺序执行上述5条语句，详细的操作过程也可以参考《SQL基本操作》章节。



### 设置访问权限

为了数据安全的考虑，AiSQL默认安装是不允许远程进行访问的。那么为了在AiSQL数据库上运行应用程序，需要对AiSQL进行配置、重启来启动远程访问的服务。请根据如下应用场景进行配置。

假如java程序调试所在的服务器(或者个人电脑)的ip地址是192.168.50.288，那么执行如下过程：

首先，需要进入bigmathd程序所在的文件目录，通过此程序可以关闭AiSQL数据库和配置启动AiSQL，运行如下语句:

```
cd AiSQL-x.x.x.x/bin
```

其次，停止运行AiSQL，运行如下语句:

```
./bigmathd stop
```

最后，重新启动AiSQL，运行如下语句:

```
./bigmathd start --advertise_address=192.168.2.125
```

启动参数--advertise_address的值是192.168.2.125，此启动参数的作用是允许远程ip地址为192.168.2.125的设备访问此AiSQL数据库；



### 开发环境说明

在进行 Java 应用开发的过程中，一个关键的前置步骤是安装 Java 开发工具包（JDK）。对于开发者而言，建议优先选择采用 JDK8 或者更高版本的 JDK，因为这些版本在性能、功能以及安全性等方面都有着不断的优化和提升，能够更好地满足现代 Java 应用开发的多样化需求。

此外，为了提高开发效率和代码编写的便捷性，开发者可以根据自身的操作习惯和偏好搭建一套适合自己的 Java 集成开发环境（IDE）。较为常见的有 IntelliJ IDEA 和 Eclipse 等。这里假设您已经把开发环境搭建好了。



### JDBC依赖包

在构建 Java 应用程序的过程中，当涉及到与 数据库进行交互时，关键的一步是通过 JDBC（Java Database Connectivity）来建立连接。为了实现这一目标，您可以下载专门针对 AiSQL 数据库的 JDBC 的 jar 包，例如jdbc-brightdb-sql-42.3.3.jar。这种特定的 jar 包是为了确保 Java 应用程序能够与 AiSQL 数据库进行高效且稳定的通信而设计的，它包含了实现连接和数据操作所需的各种类和方法。

值得注意的是，AiSQL 数据库具有良好的兼容性，尤其是与 PostgreSQL 数据库。这意味着在某些情况下，您也可以选择采用 PostgreSQL 11 版本或者更高版本的 JDBC 的 jar 包。





## 代码开发

下面是AiSQL数据库应用开发java代码，代码处理过程如下：

第一步：创建数据源对象ds，并设置数据库路径url、用户名、密码；

第二步：通过数据原对象与AiSQL数据库建立连接；

第三步：执行插入数据的sql语句，向AiSQL的testdb数据库的employees表插入一条数据；

第四步：执行获取employees表数据的sql语句，获取数据；

第五步：打印输出获取到的数据

第六步：关闭上述申请的对象资源，并结束程序的运行；

代码如下：

```
import com.bigmath.sql.BMClusterAwareDataSource;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class Main {
    public static void main(String[] args) throws SQLException {
        // 数据库连接URL，格式为：jdbc:brightdb:sql://主机IP:端口/数据库名
        String url = "jdbc:brightdb:sql://192.168.50.254:2521/testdb";
        // 数据库用户名
        String username = "aiuser";
        // 数据库密码
        String password = "123456";
		//第一步：创建数据源对象
        BMClusterAwareDataSource ds= new BMClusterAwareDataSource();
        ds.setUrl(url);
        ds.setUser(username);
        ds.setPassword(password);

        try{
        	//第二步：与AiSQL建立连接
            Connection conn = ds.getConnection();
            Statement stmt = conn.createStatement();
            //第三步：执行插入数据的sql语句
            stmt.execute("INSERT INTO employees" + " VALUES" +
                    "(3, 'Li Si', 'abc001@136.com','2020-09-12','development engineer',15000.00,3)");
            //第四步：执行获取employees表数据的sql语句
            ResultSet rs = stmt.executeQuery("SELECT * FROM employees");
            //第五步：打印输出获取到的数据
            while (rs.next()) {
                System.out.println(String.format("name = %s, email = %s, hire_date = %s, salary = %s",
                        rs.getString(2), rs.getString(3),
                        rs.getString(4), rs.getString(6)));
            }
            //第六步： 关闭资源
            rs.close();
            stmt.close();
            conn.close();
        }catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

注意：调试上述代码的时候，第一次可以正常执行，第二次再执行的时候会失败，因为在执行插入语句的时候，数据库已经存在员工工号是3的数据导致插入失败,这是由于在AiSQL数据库中已经为员工信息表employees的工号字段(employee_id)建立了主键索引(唯一索引)。

