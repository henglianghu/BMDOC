## **概述**

### **支持的项目**

以下项目可用于使用BMDB  BSQL实现Python应用程序。

| 驱动                              | 版本   |
| --------------------------------- | ------ |
| BMDB Psycopg2 Smart Driver [推荐] | 2.9.3  |
| PostgreSQL Psycopg2 Driver        | 2.9.3  |
| aiopg                             | 1.4    |
| BMDB Python Driver for BCQL       | 3.25.0 |

| 项目        | APP 示例           |
| ----------- | ------------------ |
| SQL Alchemy | SQLAlchemy ORM App |
| Django      | Django ORM App     |

### **先决条件**

要为BMDB 开发Python驱动程序应用程序，您需要以下内容： 

Python确保您的系统安装了Python3。要检查已安装的Python版本，请使用以下命令：  

```
python -V
```

## **应用连接**

### **BSQL**

#### BMDB Psycopg2 智能驱动程序

BMDB Psycopg2智能驱动程序是一个基于PostgreSQL Psycopg2驱动程序的BSQL Python驱动程序，具有额外的连接负载平衡功能。

3.7.5.2.1.1.1. *CRUD 操作*

了解如何建立与BMDB数据库的连接，并使用Go ORM示例应用程序页面上的步骤开始基本的CRUD操作。

以下部分演示如何使用BMDB Psycopg2智能驱动程序执行Python应用程序开发所需的常见任务。

若要开始构建应用程序，请确保满足先决条件。

步骤1：添加BMDB 驱动程序依赖项

构建Psycopg2需要一些先决条件（C编译器和一些开发包）。有关详细信息，请查看安装说明和常见问题解答。

BMDB Psycopg2需要PostgreSQL版本12或更高版本（最好是14）。

安装完必备组件后，像安装任何其他Python包一样安装psycopg2-bigmathdb，使用pip从PyPI下载：

```
pip install psycopg2-bigmathdb
```

或者，如果您已经在本地下载了源程序包，则可以使用setup.py脚本： 

```
python setup.py build
sudo python setup.py install
```

步骤2：设置数据库连接

下表描述了连接所需的连接参数，包括用于统一和拓扑负载平衡的智能驱动程序参数。 

| 参数            | 描述                               | 默认值                                                       |
| --------------- | ---------------------------------- | ------------------------------------------------------------ |
| host            | BMDB实例的主机名，可以输入多个地址 | localhost                                                    |
| port            | BSQL监听端口                       | 2521                                                         |
| database/dbname | 数据库名                           | bigmath                                                      |
| user            | 连接数据库的用户                   | bigmath                                                      |
| password        | 用户密码                           | bigmath                                                      |
| load_balance    | 均匀负载平衡                       | 默认为上游驱动程序行为，除非设置为“true”                     |
| topology_keys   | 拓扑感知负载平衡                   | 如果load_balance为true，则使用统一负载平衡，除非设置为cloud.region.zone形式的逗号分隔地理位置。 |

您可以通过以下方式之一提供连接详细信息： 

连接字符串:

```
"dbname=database_name host=hostname port=2521 user=username password=password load_balance=true topology_keys=cloud.region.zone1,cloud.region.zone2"
```

连接字典：

```
user = 'username', password='xxx', host = 'hostname', port = '2521', dbname = 'database_name', load_balance='true', topology_keys='cloud.region.zone1,cloud.region.zone2'
```

以下是用于连接到BMDB的连接字符串示例： 

```
conn = psycopg2.connect(dbname='bigmath',host='localhost',port='2521',user='bigmath',password='bigmath',load_balance='true')
```

在驱动程序建立初始连接后，它会从集群中获取可用服务器的列表，并在这些服务器上负载平衡后续的连接请求。

1) 使用多个地址
   您可以在连接字符串中指定多个主机，以便在初始连接期间提供备用选项，以防主地址出现故障。 

使用逗号分隔地址，如下所示：

```
conn = psycopg2.connect(dbname='bigmath',host='host1,host2,host3',port='2521',user='bigmath',password='bigmath',load_balance='true')
```

主机仅在初始连接尝试期间使用。如果驱动程序连接时第一台主机关闭，则驱动程序会尝试连接到字符串中的下一台主机，依此类推。

2) 使用SSL
   下表介绍了使用SSL进行连接所需的连接参数。 

| 参数        | 描述                 | 默认值         |
| ----------- | -------------------- | -------------- |
| sslmode     | SSL 模式             | prefer         |
| sslrootcert | 计算机上根证书的路径 | ~/.postgresql/ |

以下是在启用SSL的情况下连接到BMDB集群的示例：

```
conn = psycopg2.connect("host=<hostname> port=2521 dbname=bigmath user=<username> password=<password> load_balance=true sslmode=verify-full sslrootcert=/path/to/root.crt")
```


步骤3：写应用
在项目的基本包目录中创建一个名为QuickStartApp.py的新Python文件。
复制以下示例代码以设置表并查询表内容。如果需要，请使用集群凭据和SSL证书替换连接字符串connString。 

```
import psycopg2
 
# Create the database connection.
 
connString = "host=127.0.0.1 port=2521 dbname=bigmath user=bigmath password=bigmath load_balance=True"
 
conn = psycopg2.connect(connString)
 
# Open a cursor to perform database operations.
# The default mode for psycopg2 is "autocommit=false".
 
conn.set_session(autocommit=True)
cur = conn.cursor()
 
# Create the table. (It might preexist.)
 
cur.execute(
  """
  DROP TABLE IF EXISTS employee
  """)
 
cur.execute(
  """
  CREATE TABLE employee (id int PRIMARY KEY,
                        name varchar,
                        age int,
                        language varchar)
  """)
print("Created table employee")
cur.close()
 
# Take advantage of ordinary, transactional behavior for DMLs.
 
conn.set_session(autocommit=False)
cur = conn.cursor()
 
# Insert a row.
 
cur.execute("INSERT INTO employee (id, name, age, language) VALUES (%s, %s, %s, %s)",
            (1, 'John', 35, 'Python'))
print("Inserted (id, name, age, language) = (1, 'John', 35, 'Python')")
 
# Query the row.
 
cur.execute("SELECT name, age, language FROM employee WHERE id = 1")
row = cur.fetchone()
print("Query returned: %s, %s, %s" % (row[0], row[1], row[2]))
 
# Commit and close down.
 
conn.commit()
cur.close()
conn.close()
```

3.7.5.2.1.1.2. *运行应用程序*
使用以下命令运行项目QuickStartApp.py： 

```
python3 QuickStartApp.py
```

您应该看到类似于以下内容的输出： 

```
Created table employee
Inserted (id, name, age, language) = (1, 'John', 35, 'Python')
Query returned: John, 35, Python
```

如果没有输出或出现错误，请验证连接字符串中包含的参数。

3.7.5.2.1.1.3. *限制*

目前，PostgreSQL psycopg2驱动程序和BMDB psycopg2智能驱动程序不能在同一环境中使用。

#### PostgreSQL Psycopg2 驱动

Psycopg是最流行的用于Python的PostgreSQL数据库适配器。其主要功能是Python DB API 2.0规范的完整实现和线程安全性（多个线程可以共享同一连接）。BMDB完全支持Psycopg2

3.7.5.2.1.2.1. *CRUD 操作*
以下部分演示如何使用PostgreSQL Psycopg2驱动程序执行Python应用程序开发所需的常见任务。

若要开始构建应用程序，请确保满足先决条件。

步骤1：下载驱动程序依赖项

构建Psycopg需要一些先决条件（C编译器、一些开发包）。请参阅Psycopg文档中的安装和常见问题解答。

如果满足先决条件，您可以像安装任何其他Python包一样安装psycopg，使用pip从PyPI下载： 

```
pip install psycopg2
```

或者，如果您已经在本地下载了源程序包，请使用setup.py： 

```
python setup.py build
sudo python setup.py install
```

您也可以通过从PyPI安装psycopg2二进制包来获得独立的包，而不需要编译器或外部库： 

```
pip install psycopg2-binary
```

二进制包是开发和测试的实用选择，但在生产中，建议使用从源代码构建的包。

步骤2：连接到集群 

下表介绍了连接所需的连接参数。

| 参数            | 描述             | 默认值    |
| --------------- | ---------------- | --------- |
| user            | 连接数据库的用户 | bigmath   |
| password        | 用户密码         | bigmath   |
| host            | BMDB实例的主机名 | localhost |
| port            | BSQL监听端口     | 2521      |
| database/dbname | 数据库名         | bigmath   |

您可以通过以下方式之一提供连接详细信息：

连接字符串：

```
"dbname=database_name host=hostname port=port user=username password=password"
```

连接字典：

```
user = 'username', password='xxx', host = 'hostname', port = 'port', dbname = 'database_name'
```

以下是用于连接到BMDB的连接字符串示例。 

```
conn = psycopg2.connect(dbname='bigmath',host='localhost',port='2521',user='bigmath',password='bigmath')
```

1） 使用SSL

下表介绍了使用SSL进行连接所需的连接参数。 

| 参数        | 描述                 | 默认值         |
| ----------- | -------------------- | -------------- |
| sslmode     | SSL 模式             | prefer         |
| sslrootcert | 计算机上根证书的路径 | ~/.postgresql/ |

以下是在启用SSL加密的情况下连接到BMDB的示例： 

```
conn = psycopg2.connect("host=<hostname> port=2521 dbname=bigmath user=<username> password=<password> sslmode=verify-full sslrootcert=/Users/my-user/Downloads/root.crt")
```

步骤3：写应用


在项目的基本包目录中创建一个名为QuickStartApp.py的新Python文件。

复制以下示例代码以设置表并查询表内容。如果需要，请使用集群凭据和SSL证书替换连接字符串connString。 

```
import psycopg2
 
# Create the database connection.
 
connString = "host=127.0.0.1 port=2521 dbname=bigmath user=bigmath password=bigmath"
 
conn = psycopg2.connect(connString)
 
# Open a cursor to perform database operations.
# The default mode for psycopg2 is "autocommit=false".
 
conn.set_session(autocommit=True)
cur = conn.cursor()
 
# Create the table. (It might preexist.)
 
cur.execute(
  """
  DROP TABLE IF EXISTS employee
  """)
 
cur.execute(
  """
  CREATE TABLE employee (id int PRIMARY KEY,
                        name varchar,
                        age int,
                        language varchar)
  """)
print("Created table employee")
cur.close()
 
# Take advantage of ordinary, transactional behavior for DMLs.
 
conn.set_session(autocommit=False)
cur = conn.cursor()
 
# Insert a row.
 
cur.execute("INSERT INTO employee (id, name, age, language) VALUES (%s, %s, %s, %s)",
            (1, 'John', 35, 'Python'))
print("Inserted (id, name, age, language) = (1, 'John', 35, 'Python')")
 
# Query the row.
 
cur.execute("SELECT name, age, language FROM employee WHERE id = 1")
row = cur.fetchone()
print("Query returned: %s, %s, %s" % (row[0], row[1], row[2]))
 
# Commit and close down.
 
conn.commit()
cur.close()
conn.close()
```

使用以下命令运行项目QuickStartApp.py：

```
python3 QuickStartApp.py
```

您应该看到类似于以下内容的输出：

```
Created table employee
Inserted (id, name, age, language) = (1, 'John', 35, 'Python')
Query returned: John, 35, Python
```

如果没有输出或出现错误，请验证连接字符串中包含的参数。
3.7.5.2.1.2.2. *限制*

目前，PostgreSQL psycopg2驱动程序和BMDB psycopg2智能驱动程序不能在同一环境中使用。 

#### aiopg

以下教程创建了一个基本的Python应用程序，该应用程序使用aiopg数据库适配器连接到BMDB 集群，执行一些基本的数据库操作——创建表、插入数据和运行SQL查询——并将结果打印到屏幕上
3.7.5.2.1.3.1. *先决条件*
在开始使用Aiopg之前，请确保您具备以下条件：

* BMDB 启动并运行。如果您是BMDB 的新手，请按照快速入门中的步骤在几分钟内启动并运行BMDB 。

* 已安装Python 3或更高版本。

* aiopg包已安装。使用以下命令安装程序包：

```
pip3install aiopg
```

有关使用此数据库适配器的详细信息，请参阅aiopg文档。 

3.7.5.2.1.3.2. *创建示例Python应用程序*
创建一个文件bm-sql-helloworld.py并复制以下代码： 

```
import asyncio
import aiopg
 
dsn = 'dbname=bigmath user=bigmath password=bigmath host=127.0.0.1 port=2521'
 
async def go():
    async with aiopg.create_pool(dsn) as pool:
        async with pool.acquire() as conn:
            # Open a cursor to perform database operations.
            async with conn.cursor() as cur:
                await cur.execute(f"""
                  DROP TABLE IF EXISTS employee;
                  CREATE TABLE employee (id int PRIMARY KEY,
                             name varchar,
                             age int,
                             language varchar);
                """)
                print("Created table employee")
                # Insert a row.
                await cur.execute("INSERT INTO employee (id, name, age, language) VALUES (%s, %s, %s, %s)",
                                  (1, 'John', 35, 'Python'))
                print("Inserted (id, name, age, language) = (1, 'John', 35, 'Python')")
 
                # Query the row.
                await cur.execute("SELECT name, age, language FROM employee WHERE id = 1")
                async for row in cur:
                    print("Query returned: %s, %s, %s" % (row[0], row[1], row[2]))
 
 
loop = asyncio.get_event_loop()
loop.run_until_complete(go())
```

3.7.5.2.1.3.3. *运行应用程序*

要使用该应用程序，请运行以下Python脚本： 

```
python bm-sql-helloworld.py
```

您应该看到以下输出：

```
Created table employee
Inserted (id, name, age, language) = (1, 'John', 35, 'Python')
Query returned: John, 35, Python
```

### **BCQL**

#### BMDB Python 驱动

3.7.5.2.2.1.1. *为BCQL安装**BMDB**Python驱动程序*
要安装BCQL的BMDB Python驱动程序，请运行以下命令： 

```
pip3 install bm-cassandra-driver --install-option="--no-cython"
```

3.7.5.2.2.1.2. *创建示例Python应用程序*
1） 先决条件
本教程假设您具备：
安装了BMDB ，创建了一个universe，并能够使用BCQL shell与之交互。如果没有，请按照“快速入门”中的步骤进行操作。 

2） 编写示例Python应用程序
创建一个文件bm-cql-helloworld.py，并将以下内容复制到其中。

```
from cassandra.cluster import Cluster
 
# Create the cluster connection.
cluster = Cluster(['127.0.0.1'])
session = cluster.connect()
 
# Create the keyspace.
session.execute('CREATE KEYSPACE IF NOT EXISTS bmdbdemo;')
print("Created keyspace bmdbdemo")
 
# Create the table.
session.execute(
  """
  CREATE TABLE IF NOT EXISTS bmdbdemo.employee (id int PRIMARY KEY,
                                              name varchar,
                                              age int,
                                              language varchar);
  """)
print("Created table employee")
 
# Insert a row.
session.execute(
  """
  INSERT INTO bmdbdemo.employee (id, name, age, language)
  VALUES (1, 'John', 35, 'Python');
  """)
print("Inserted (id, name, age, language) = (1, 'John', 35, 'Python')")
 
# Query the row.
rows = session.execute('SELECT name, age, language FROM bmdbdemo.employee WHERE id = 1;')
for row in rows:
  print(row.name, row.age, row.language)
 
# Close the connection.
cluster.shutdown()
```

使用SSL

要使用SSL运行应用程序，请使用其他SSL导入和参数创建集群连接，如下所述： 

```
# Include additional imports.
from ssl import SSLContext, PROTOCOL_TLS_CLIENT, CERT_REQUIRED
from cassandra.auth import PlainTextAuthProvider
 
# Include additional parameters.
ssl_context = SSLContext(PROTOCOL_TLS_CLIENT)
ssl_context.load_verify_locations('path to certs file')
ssl_context.verify_mode = CERT_REQUIRED
 
# Create the cluster connection.
cluster = Cluster(contact_points=['ip_address'],
    ssl_context=ssl_context,
    ssl_options={'server_hostname': 'ip_address'},
    auth_provider=PlainTextAuthProvider(username='username', password='password'))
session = cluster.connect()
```

3.7.5.2.2.1.3. *运行应用程序*

要运行应用程序，请键入以下内容： 

```
python3 bm-cql-helloworld.py
```

您应该看到以下输出。

```
Created keyspace bmdbdemo
Created table employee
Inserted (id, name, age, language) = (1, 'John', 35, 'Python')
John 35 Python
```

## **使用ORM**

### **SQL Alchemy ORM**

SQL Alchemy是Python应用程序的一个流行的ORM提供程序，被Python开发人员广泛用于数据库访问。BMDB提供了对SQL Alchemy ORM的全面支持。

#### CRUD操作

了解如何建立与BMDB数据库的连接，并使用Python ORM示例应用程序页面中的步骤开始基本的CRUD操作。

以下部分演示如何使用SQL Alchemy ORM执行Python应用程序开发所需的常见任务。

添加SQL Alchemy ORM依赖项

要下载SQL Alchemy并将其安装到您的项目中，请使用以下命令。 

```
pip3 install sqlalchemy
```

您可以按照以下方式验证安装：


1） 通过执行以下命令打开Python提示： 

```
python3
```

2） 在Python提示下，执行以下命令以检查SQLAlchemy版本： 

```
import sqlalchemy
sqlalchemy.__version__
```

实现BMDB的ORM映射

要从SQLAlchemy开始，请在项目目录中创建4个Python文件-config.py、base.py、model.py和main.py

1） config.py包含连接到数据库的凭据。将以下示例代码复制到config.py文件中。 

```
 db_user = 'bigmath'
 db_password = 'bigmath'
 database = 'bigmath'
 db_host = 'localhost'
 db_port = 2521
```

2） 接下来，声明一个映射。使用ORM时，配置过程首先描述将要使用的数据库表，然后定义映射到这些表的类。在现代SQLAlchemy中，这两个任务通常一起执行，使用一个称为声明扩展的系统。使用声明性系统映射的类是根据基类定义的，该基类维护相对于该基类的类和表的目录，这被称为声明性基类。使用declarative_base()函数创建基类。将以下代码添加到base.py文件中。 

```
from sqlalchemy.ext.declarative import declarative_base
Base = declarative_base()
```

3） 既然有了基础，就可以根据它定义任意数量的映射类。从一个名为employees的表开始，为使用应用程序的最终用户存储记录。一个名为Employee的新类映射到此表。在类中，定义要映射到的表的详细信息；主要是表名、列的名称和数据类型。将以下内容添加到model.py文件中：

```
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Table, Column, Integer, String, DateTime, ForeignKey
from base import Base
 
class Employee(Base):
 
  __tablename__ = 'employees'
 
  id = Column(Integer, primary_key=True)
  name = Column(String(255), unique=True, nullable=False)
  age = Column(Integer)
  language = Column(String(255))
```

4） 设置完成后，您可以连接到数据库并创建一个新会话。在main.py文件中，添加以下内容：

```
import config as cfg
from sqlalchemy.orm import sessionmaker, relationship
from sqlalchemy import create_engine
from sqlalchemy import MetaData
from model import Employee
from base import Base
from sqlalchemy import Table, Column, Integer, String, DateTime, ForeignKey
 
# create connection
engine = create_engine('postgresql://{0}:{1}@{2}:{3}/{4}'.format(cfg.db_user, cfg.db_password, cfg.db_host, cfg.db_port, cfg.database))
 
# create metadata
Base.metadata.create_all(engine)
 
# create session
Session = sessionmaker(bind=engine)
session = Session()
 
# insert data
tag_1 = Employee(name='Bob', age=21, language='Python')
tag_2 = Employee(name='John', age=35, language='Java')
tag_3 = Employee(name='Ivy', age=27, language='C++')
 
session.add_all([tag_1, tag_2, tag_3])
 
# Read the inserted data
 
print('Query returned:')
for instance in session.query(Employee):
    print("Name: %s Age: %s Language: %s"%(instance.name, instance.age, instance.language))
session.commit()
```

当您运行main.py文件时，您应该得到类似于以下内容的输出： 

```
Query returned:
Name: Bob Age: 21 Language: Python
Name: John Age: 35 Language: Java
Name: Ivy Age: 27 Language: C++
```

### **Django ORM**

Django是一个高级Python web框架，它鼓励快速开发和干净、实用的设计。

#### CRUD操作

了解如何建立与BMDB数据库的连接，并使用Python ORM示例应用程序页面中的步骤开始基本的CRUD操作。


以下部分演示如何使用Django ORM执行Python应用程序开发所需的常见任务。

##### *添加依赖项*

要将Django Rest Framework下载到您的项目中，请运行以下命令： 

```
pip3 install djangorestframework
```

此外，使用以下命令安装Django的BM后端：

```
pip3 install django-bigmathdb
```

##### *实现**BMDB**的ORM映射*

1) 安装完依赖项后，启动一个Django项目，并使用以下命令创建一个新的应用程序： 

```
django-admin startproject bigmathTest && cd bigmathTest/
```

2) 使用以下命令设置一个新的Django应用程序： 

```
python manage.py startapp testdb
```

3) 创建应用程序后，将其配置为连接到数据库。要执行此操作，请更改应用程序设置以提供数据库凭据。为了使用BMDB获得更好的性能，请使用持久连接（设置CONN_MAX_AGE）。在文件bigmathTest/settings.py中添加以下代码： 

```
DATABASES = {
    'default': {
        'ENGINE': 'django_bigmathdb',
        'NAME': 'bigmath',
        'HOST': 'localhost',
        'PORT': 2521,
        'USER': 'bigmath',
        'CONN_MAX_AGE': None,
        'PASSWORD': 'bigmath'
    }
}
```

4) 您需要INSTALLED_APPS字段中的应用程序和rest框架。将现有代码替换为以下代码： 

```
INSTALLED_APPS = [
    'rest_framework',
    'testdb.apps.TestdbConfig',
    'django.contrib.contenttypes',
    'django.contrib.auth',
]
 
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [],
    'DEFAULT_PERMISSION_CLASSES': [],
    'UNAUTHENTICATED_USER': None,
}
```

5) 下一步是为表创建一个模型。表名为users，包含四列——user_id、firstName、lastName和email。
   将以下代码添加到testdb/models.py中： 

```
rom django.db import models
 
class Users(models.Model):
    userId = models.AutoField(db_column='user_id', primary_key=True, serialize=False)
    firstName = models.CharField(max_length=50, db_column='first_name')
    lastName = models.CharField(max_length=50, db_column='last_name')
    email = models.CharField(max_length=100, db_column='user_email')
 
    class Meta:
        db_table = "users"
 
    def __str__(self):
        return '%d %s %s %s' % (self.userId, self.firstName, self.lastName, self.email)
```

6) 创建模型后，您需要创建一个序列化程序。序列化程序允许将查询集和模型实例等复杂数据转换为原生Python数据类型，然后将其呈现为JSON、XML或其他内容类型。序列化程序还提供反序列化，允许在首次验证传入数据后将解析的数据转换回复杂类型。

将以下代码复制到testdb\serializers.py中： 

```
from rest_framework import serializers, status
from testdb.models import Users
from django.core.exceptions import ValidationError
 
class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = Users
        fields = ('userId', 'firstName', 'lastName', 'email')
```

7) 要完成应用程序的所有元素，请创建一个ViewSet。在testdb/views.py中，添加以下代码：

```
from django.shortcuts import render
from testdb.models import Users
from rest_framework import viewsets
from testdb.serializers import UserSerializer
 
class UserViewSet(viewsets.ModelViewSet):
    queryset = Users.objects.all()
    serializer_class = UserSerializer
```

8) 最后，通过添加以下代码在bigmathTest/uls.py中映射URL： 

```
from django.urls import include, re_path
from rest_framework import routers
from testdb.views import UserViewSet
 
router = routers.SimpleRouter(trailing_slash=False)
router.register(r'users', UserViewSet)
urlpatterns = [
    re_path(r'^', include(router.urls))
]
```

对于4.0之前的Django版本，请使用以下代码，因为您可以使用Django.conf.uls导入URL：

```
from django.urls import path, include
from django.conf.urls import url, include
from rest_framework import routers
from testdb.views import UserViewSet
 
router = routers.SimpleRouter(trailing_slash=False)
router.register(r'users', UserViewSet)
 
urlpatterns = [
    url(r'^', include(router.urls))
]
```

9) 这就完成了测试应用程序的配置。接下来的步骤是创建迁移文件并将迁移应用于数据库。要执行此操作，请运行以下命令： 

```
python3 manage.py makemigrations
python3 manage.py migrate
```

应该在数据库中创建一个用户表。使用sqlsh客户端shell验证users表是否已创建

##### *运行应用程序*
要运行应用程序并插入新行，请执行以下步骤。

1) 使用以下命令运行Django项目： 

```
python3 manage.py runserver 8080
```

2) 使用以下命令插入一行：

```
curl --data '{ "firstName" : "John", "lastName" : "Smith", "email" : "jsmith@bmdb.com" }' \
      -v -X POST -H 'Content-Type:application/json' http://localhost:8080/users
```

3) 使用以下命令验证是否已插入新行：

```
curl http://localhost:8080/users
```

您应该看到以下输出： 

```
[{"userId":1,"firstName":"John","lastName":"Smith","email":"jsmith@bmdb.com"}]
```

您也可以使用sqlsh客户端shell来验证这一点。
