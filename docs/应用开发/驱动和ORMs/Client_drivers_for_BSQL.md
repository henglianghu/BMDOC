

AiSQL 结构化查询语言 (BSQL) API 基于 PostgreSQL 11.2 的查询层分支构建并扩展，旨在支持大多数 PostgreSQL 功能并向支持的分布式 SQL 数据库添加新功能。

有关 BSQL 中 PostgreSQL 功能支持的详细信息，请参阅 SQL 功能支持。

下面列出的客户端驱动程序支持开发连接到 BSQL API 并与之交互的应用程序。 大多数驱动程序使用 libpq 并支持 SCRAM-SHA-256 身份验证方法。

如需将这些驱动程序与 BSQL 一起使用的帮助，请在 Slack 社区中提问。

如果您遇到问题或有增强请求，请提交 GitHub 问题。

## **C**
**libpq**
libpq 是用于连接 PostgreSQL 数据库并与之交互的 C 客户端库。 libpq 也是其他 PostgreSQL 应用程序接口中使用的底层引擎。 libpq 客户端库支持 SCRAM-SHA-256 身份验证方法。

有关详细信息和文档，请参阅 libpq - C Library for PostgreSQL 11（AiSQL 所基于的）。

有关使用 libpq 构建示例 C 应用程序的教程，请参阅连接应用程序。

安装 libpq 客户端库
libpq C 驱动程序包含在 AiSQL 安装中。 您可以通过设置 LD_LIBRARY_PATH 来使用它，如下所示：
```
$ export LD_LIBRARY_PATH=<bigmath-install-dir>/postgres/lib
```

macOS 上的 Homebrew 用户可以使用brew install libpq来安装libpq。 您可以从 PostgreSQL 下载页面下载 PostgreSQL 二进制文件和源代码。

## **C++**
libpqxx
libpqxx 驱动程序是 PostgreSQL 的官方 C++ 客户端 API。 libpqxx 基于 libpq 并支持 SCRAM-SHA-256 身份验证方法。

有关详细信息和文档，请参阅 libpqxx 自述文件和 libpqxx 文档。

有关使用 libpqxx 构建示例 C++ 应用程序的教程，请参阅连接应用程序。

安装 libpqxx 驱动程序
要构建并安装 libpqxx 驱动程序以与 AiSQL 一起使用，请首先克隆 libpqxx 存储库。
```
$ git clone <https://github.com/jtv/libpqxx.git>
```

对于 PostgreSQL 二进制文件的依赖项，请通过运行以下命令将 PostgreSQL bin 目录添加到命令路径。
```
$ export PATH=$PATH:<bigmath-install-dir>/postgres/bin
```

构建并安装驱动程序。
```
$ cd libpqxx
$ ./configure
$ make
$ make install
```

## **Java**
Vert.x PG Client
Vert.x PG Client 是 PostgreSQL 的客户端，具有与数据库通信的基本 API。 它是一个反应式、非阻塞客户端，用于使用单线程 API 处理数据库连接。

有关使用 Vert.x PG Client构建示例 Java 应用程序的教程，请参阅连接应用程序。

要获取使用 Apache Maven 的项目的最新版本，请参阅  Maven Central Repository of Vert.x PG Client。

## **PHP**
php-pgsql
php-pgsql 驱动程序是 PHP 官方 PostgreSQL 模块的集合。 php-pgsql 基于 libpq 并支持 SCRAM-SHA-256 身份验证方法。

有关安装和使用 php-pgsql 的详细信息，请参阅 php-pgsql 文档。

有关使用 php-pgsql 构建示例 PHP 应用程序的教程，请参阅连接应用程序。

**安装 php-pgsql 驱动程序**
要使用 php-pgsql 启用 PostgreSQL 支持，请参阅 PHP 文档中的安装/配置。

macOS 上的 Homebrew 用户可以使用brew install php 安装 PHP； php-pgsql 驱动程序会自动安装。

Ubuntu 用户可以使用 sudo apt-get install php-pgsql 命令安装驱动程序。

CentOS 用户可以使用 sudo yum install php-pgsql 命令安装驱动程序。

## **Python**
aiopg
aiopg 是一个使用 asyncio (PEP-3156/tulip) 框架访问 PostgreSQL 数据库的库。 它包装了 Psycopg 数据库驱动程序的异步功能。 有关使用 aiopg 的详细信息，请参阅 aiopg 文档。

有关构建使用 aiopg 的示例 Python 应用程序的教程，请参阅 BSQL Aiopg。

**安装**
要安装 aiopg 软件包，请运行以下 pip3 install 命令：
```
pip3 install aiopg
```


## **Ruby**
PG
pg 是 PostgreSQL 数据库的 Ruby 接口。 pg 基于 libpq 并支持 SCRAM-SHA-256 身份验证方法。

有关使用 pg 构建示例 Ruby 应用程序的教程，请参阅连接应用程序。

**安装pg驱动**
如果您本地已经安装了 AiSQL，请运行以下 gem install 命令来安装 pg 驱动：
```
$ gem install pg -- --with-pg-config=<bigmath-install-dir>/postgres/bin/pg_config
```

否则，要安装 pg，请运行以下命令：
```
gem install pg -- --with-pg-include=<path-to-libpq>/libpq/include --with-pg-lib=<path-to-libpq>/libpq/lib
```

将 <path-to-libpq> 替换为 libpq 安装路径； 例如，/usr/local/opt。

## **Rust**
Rust-Postgres
Rust-Postgres 是 PostgreSQL 数据库的 Rust 接口。 Rust-Postgres 不基于 libpq，但支持 SCRAM-SHA-256 身份验证方法。

有关使用 Rust-Postgres 构建示例 Ruby 应用程序的教程，请参阅构建 Rust 应用程序。

**安装 Rust-Postgres 驱动程序**
要将 Rust-Postgres 驱动程序包含在您的应用程序中，请将以下依赖项添加到您的 Cargo.toml 文件中：
```
postgres = "0.19.2"
openssl = "0.10.38"
postgres-openssl = "0.5.0"
```
