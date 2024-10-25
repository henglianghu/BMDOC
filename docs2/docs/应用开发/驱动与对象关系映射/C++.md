## **概述**

### **支持的项目**

建议以下项目用于使用BMDB BSQL/BCQL API实现C++应用程序。

| 项目                     | 应用示例    |
| ------------------------ | ----------- |
| libpqxx C++ Driver       | Hello World |
| BMDB C++ driver for BCQL | Hello World |

有关完全可运行的代码片段和常见操作的解释，请参阅示例应用程序。在运行示例应用程序之前，请确保已安装必备组件


### **先决条件**

要为BMDB开发C应用程序，您需要以下内容：

* 机器和软件要求
  * 32位（x86）或64位（x64）体系结构的机器。
  * gcc 4.1.2或更新版本，clang 3.4或更新版本。



## **应用连接**

### **BSQL**

#### libpqxx C++驱动程序

1）先决条件

本教程假设您具备：

* 安装了BMDB，并创建了一个启用BSQL的universe。如果没有，请按照“快速入门”中的步骤操作。
  * 具有32位（x86）或64位（x64）体系结构的计算机。
  * 安装gcc 4.1.2或更高版本，clang 3.4或更高级别。



2）安装libpqxx驱动程序

从libpqxx下载源代码，并按照如下方式构建二进制文件。如果需要，README文件中提供了详细的步骤。
获取源 

```
git clone https://gitlab.bigmath.com/jtv/libpqxx.git
```

依赖项
请注意，这个包依赖于PostgreSQL二进制文件。确保PostgreSQL bin目录位于命令路径上

```
export PATH=$PATH:<bigmath-install-dir>/postgres/bin
```

构建和安装

```
cd libpqxx
./configure
make
make install
```

3）创建一个示例C++应用程序

添加C++代码

创建一个文件bmdbsql_hello_world.cpp，并复制以下内容：

```
#include <iostream>
#include <pqxx/pqxx>
 
int main(int, char *argv[])
{
  pqxx::connection c("host=127.0.0.1 port=2521 dbname=bigmath user=bigmath password=bigmath");
  pqxx::work txn(c);
  pqxx::result r;
 
  /* Create table */
  try
  {
    r = txn.exec("CREATE TABLE employee (id int PRIMARY KEY, \
                  name varchar, age int, \
                  language varchar)");
  }
  catch (const std::exception &e)
  {
    std::cerr << e.what() << std::endl;
    return 1;
  }
 
  std::cout << "Created table employee\n";
 
  /* Insert a row */
  try
  {
    r = txn.exec("INSERT INTO employee (id, name, age, language) \
                  VALUES (1, 'John', 35, 'C++')");
  }
  catch (const std::exception &e)
  {
    std::cerr << e.what() << std::endl;
    return 1;
  }
 
  std::cout << "Inserted data (1, 'John', 35, 'C++')\n";
 
  /* Query the row */
  try
  {
    r = txn.exec("SELECT name, age, language FROM employee WHERE id = 1");
 
    for (auto row: r)
      std::cout << "Query returned: "
          << row["name"].c_str() << ", "
          << row["age"].as<int>() << ", "
          << row["language"].c_str() << std::endl;
  }
  catch (const std::exception &e)
  {
    std::cerr << e.what() << std::endl;
    return 1;
  }
 
  txn.commit();
  return 0;
}
```

运行应用程序

您可以使用gcc或clang编译该文件。请注意，C++11是最低支持的C++版本。请确保您的编译器支持这一点，如有必要，请确保您已配置对C++11的支持。对于gcc，运行以下命令：

```
g++ -std=c++11 bmdbsql_hello_world.cpp -lpqxx -lpq -I<bigmath-install-dir>/postgres/include -o bmdbsql_hello_world
```

通过运行以下命令使用应用程序：

```
./bmdbsql_hello_world
```

您应该看到以下输出：

```
Created table employee
Inserted data (1, 'John', 35, 'C++')
Query returned: John, 35, C++
```

### **BCQL**

#### BMDB C++ 驱动程序

1）先决条件

本教程假设您具备：

* 安装了BMDB，创建了一个universe，并能够使用BCQL shell（cqlsh）与之交互。如果没有，请按照“快速入门”中的步骤进行操作。
* 具有32位（x86）或64位（x64）体系结构的计算机。
* 安装gcc 4.1.2或更高版本，Clang 3.4或更高。

2）为BCQL安装BMDB C++驱动程序
要获得BCQL的BMDB C++驱动程序，请克隆存储库：

```
git clone https://gitlab.bigmath.com/bigmath/cassandra-cpp-driver.git
```

依赖项

BCQL的BMDB C++驱动程序取决于以下内容：

* CMake v2.6.4+
* libuv 1.x
* OpenSSL v1.0.x或v1.1.x

此处提供了有关安装依赖项的更详细说明。

构建和安装

要构建和安装驱动程序，请运行以下命令：

```
mkdir build
cd build
cmake ..
make
make install
```

3）工作示例

编写应用程序

创建一个文件bmdbcql_hello_world.c并复制以下内容：

```
#include <assert.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
 
#include "cassandra.h"
 
void print_error(CassFuture* future) {
  const char* message;
  size_t message_length;
  cass_future_error_message(future, &message, &message_length);
  fprintf(stderr, "Error: %.*s\n", (int)message_length, message);
}
 
// Create a new cluster.
CassCluster* create_cluster(const char* hosts) {
  CassCluster* cluster = cass_cluster_new();
  cass_cluster_set_contact_points(cluster, hosts);
  return cluster;
}
 
// Connect to the cluster given a session.
CassError connect_session(CassSession* session, const CassCluster* cluster) {
  CassError rc = CASS_OK;
  CassFuture* future = cass_session_connect(session, cluster);
 
  cass_future_wait(future);
  rc = cass_future_error_code(future);
  if (rc != CASS_OK) {
    print_error(future);
  }
  cass_future_free(future);
 
  return rc;
}
 
CassError execute_query(CassSession* session, const char* query) {
  CassError rc = CASS_OK;
  CassFuture* future = NULL;
  CassStatement* statement = cass_statement_new(query, 0);
 
  future = cass_session_execute(session, statement);
  cass_future_wait(future);
 
  rc = cass_future_error_code(future);
  if (rc != CASS_OK) {
    print_error(future);
  }
 
  cass_future_free(future);
  cass_statement_free(statement);
 
  return rc;
}
 
CassError execute_and_log_select(CassSession* session, const char* stmt) {
  CassError rc = CASS_OK;
  CassFuture* future = NULL;
  CassStatement* statement = cass_statement_new(stmt, 0);
 
  future = cass_session_execute(session, statement);
  rc = cass_future_error_code(future);
  if (rc != CASS_OK) {
    print_error(future);
  } else {
    const CassResult* result = cass_future_get_result(future);
    CassIterator* iterator = cass_iterator_from_result(result);
    if (cass_iterator_next(iterator)) {
      const CassRow* row = cass_iterator_get_row(iterator);
      int age;
      const char* name; size_t name_length;
      const char* language; size_t language_length;
      cass_value_get_string(cass_row_get_column(row, 0), &name, &name_length);
      cass_value_get_int32(cass_row_get_column(row, 1), &age);
      cass_value_get_string(cass_row_get_column(row, 2), &language, &language_length);
      printf ("Select statement returned: Row[%.*s, %d, %.*s]\n", (int)name_length, name,
          age, (int)language_length, language);
    } else {
      printf("Unable to fetch row!\n");
    }
 
    cass_result_free(result);
    cass_iterator_free(iterator);
  }
 
  cass_future_free(future);
  cass_statement_free(statement);
 
  return rc;
}
 
int main() {
  // Ensure you log errors.
  cass_log_set_level(CASS_LOG_ERROR);
 
  CassCluster* cluster = NULL;
  CassSession* session = cass_session_new();
  CassFuture* close_future = NULL;
  char* hosts = "127.0.0.1";
 
  cluster = create_cluster(hosts);
 
  if (connect_session(session, cluster) != CASS_OK) {
    cass_cluster_free(cluster);
    cass_session_free(session);
    return -1;
  }
 
  CassError rc = CASS_OK;
  rc = execute_query(session, "CREATE KEYSPACE IF NOT EXISTS bmdbdemo");
  if (rc != CASS_OK) return -1;
  printf("Created keyspace bmdbdemo\n");
 
  rc = execute_query(session, "DROP TABLE IF EXISTS bmdbdemo.employee");
  if (rc != CASS_OK) return -1;
 
  rc = execute_query(session,
                "CREATE TABLE bmdbdemo.employee (id int PRIMARY KEY, \
                                              name varchar, \
                                              age int, \
                                              language varchar)");
  if (rc != CASS_OK) return -1;
  printf("Created table bmdbdemo.employee\n");
 
  const char* insert_stmt = "INSERT INTO bmdbdemo.employee (id, name, age, language) VALUES (1, 'John', 35, 'C/C++')";
  rc = execute_query(session, insert_stmt);
  if (rc != CASS_OK) return -1;
  printf("Inserted data: %s\n", insert_stmt);
 
  const char* select_stmt = "SELECT name, age, language from bmdbdemo.employee WHERE id = 1";
  rc = execute_and_log_select(session, select_stmt);
  if (rc != CASS_OK) return -1;
 
  close_future = cass_session_close(session);
  cass_future_wait(close_future);
  cass_future_free(close_future);
 
  cass_cluster_free(cluster);
  cass_session_free(session);
 
  return 0;
}
```

运行应用程序

您可以使用gcc或clang编译该文件。

对于clang，运行以下命令：

```
clang bmdbcql_hello_world.c -lcassandra -Iinclude -o bmdb_cql_hello_world
```

运行：

```
./bmdb_cql_hello_world
```

您应该看到以下输出：

```
Created keyspace bmdbdemo
Created table bmdbdemo.employee
Inserted data: INSERT INTO bmdbdemo.employee (id, name, age, language) VALUES (1, 'John', 35, 'C/C++')
Select statement returned: Row[John, 35, C/C++]
```
