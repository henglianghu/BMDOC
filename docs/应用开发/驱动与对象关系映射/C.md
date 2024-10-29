## **概述**

### **支持的项目**

建议以下项目用于使用BMDB BSQL API实现C应用程序。

| 项目           | 应用示例    |
| -------------- | ----------- |
| libpq C Driver | Hello World |

有关完全可运行的代码片段和常见操作的解释，请参阅示例应用程序。在运行示例应用程序之前，请确保已安装必备组件。

### **先决条件**

要为BMDB开发C应用程序，您需要以下内容：

* 机器和软件要求
  n 32位（x86）或64位（x64）体系结构的机器。
  n gcc 4.1.2或更新版本，clang 3.4或更新版本。

## **应用连接**

### **BSQL**

#### libpq C驱动程序

libpq是用于连接PostgreSQL数据库并与之交互的C客户端库。libpq也是其他PostgreSQL应用程序接口中使用的底层引擎。libpq客户端库支持SCRAM-SHA-256身份验证方法。
有关详细信息和文档，请参阅PostgreSQL 11的libpq-C库（BMDB基于此库）


1）先决条件
本教程假设您具备：

* 安装了BMDB并创建了一个universe。如果没有，请按照“快速入门”中的步骤进行操作。
* 32位（x86）或64位（x64）体系结构的机器。
* gcc 4.1.2或更新版本，clang 3.4或更新版本已安装。

2）安装libpq C驱动程序

libpq C驱动程序包含在BMDB安装中。您可以通过如下设置LD_LIBRARY_PATH来使用它：

```
export LD_LIBRARY_PATH=<bigmath-install-dir>/postgres/lib
```

或者，您可以从PostgreSQL下载页面下载PostgreSQL二进制文件和源代码，macOS上的Homebrew用户可以使用brew install libpq安装libpq

3）创建示例C应用程序
示例C代码
创建一个文件bmdbsql_hello_world.c，并复制以下内容：

```
#include <stdio.h>
#include <stdlib.h>
#include "libpq-fe.h"
 
int
main(int argc, char **argv)
{
  const char *conninfo;
  PGconn     *conn;
  PGresult   *res;
  int         nFields;
  int         i, j;
 
  /* connection string */
  conninfo = "host=127.0.0.1 port=2521 dbname=bigmath user=bigmath password=bigmath";
 
  /* Make a connection to the database */
  conn = PQconnectdb(conninfo);
 
  /* Check to see that the backend connection was successfully made */
  if (PQstatus(conn) != CONNECTION_OK)
  {
      fprintf(stderr, "Connection to database failed: %s",
        PQerrorMessage(conn));
      PQfinish(conn);
      exit(1);
  }
 
  /* Create table */
  res = PQexec(conn, "CREATE TABLE employee (id int PRIMARY KEY, \
                                             name varchar, age int, \
                                             language varchar)");
 
  if (PQresultStatus(res) != PGRES_COMMAND_OK)
  {
      fprintf(stderr, "CREATE TABLE failed: %s", PQerrorMessage(conn));
      PQclear(res);
      PQfinish(conn);
      exit(1);
  }
  PQclear(res);
  printf("Created table employee\n");
 
  /* Insert a row */
  res = PQexec(conn, "INSERT INTO employee (id, name, age, language) \
                      VALUES (1, 'John', 35, 'C')");
 
  if (PQresultStatus(res) != PGRES_COMMAND_OK)
  {
      fprintf(stderr, "INSERT failed: %s", PQerrorMessage(conn));
      PQclear(res);
      PQfinish(conn);
      exit(1);
  }
  PQclear(res);
  printf("Inserted data (1, 'John', 35, 'C')\n");
 
 
  /* Query the row */
  res = PQexec(conn, "SELECT name, age, language FROM employee WHERE id = 1");
  if (PQresultStatus(res) != PGRES_TUPLES_OK)
  {
      fprintf(stderr, "SELECT failed: %s", PQerrorMessage(conn));
      PQclear(res);
      PQfinish(conn);
      exit(1);
  }
 
  /* print out the rows */
  nFields = PQnfields(res);
  for (i = 0; i < PQntuples(res); i++)
  {
      printf("Query returned: ");
      for (j = 0; j < nFields; j++)
        printf("%s ", PQgetvalue(res, i, j));
      printf("\n");
  }
  PQclear(res);
 
  /* close the connection to the database and cleanup */
  PQfinish(conn);
 
  return 0;
}
```

3）运行应用程序

您可以使用gcc或clang编译该文件。要使用gcc编译应用程序，请运行以下命令：

```
gcc bmdbsql_hello_world.c -lpq -I<bigmath-install-dir>/postgres/include -o bmdbsql_hello_world
```

运行：

```
./bmdbsql_hello_world
```

您应该看到以下输出：

```
Created table employee
Inserted data (1, 'John', 35, 'C')
Query returned: John 35 C
```
