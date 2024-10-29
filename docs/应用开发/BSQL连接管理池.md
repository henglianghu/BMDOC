BSQL 内置服务器端连接池

BMDB 继承了 PostgreSQL 为每个数据库连接创建一个后端进程的架构。 这些后端进程消耗内存和CPU，限制了BMDB可以支持的连接数量。 为了解决这个问题，您可以使用连接池，它允许将多个客户端连接复用为较少数量的实际服务器连接，从而支持来自应用程序的大量连接。 PgBouncer 和 Odyssey 是一些流行的基于 PostgreSQL 的服务器端连接池机制，它们与 BMDB 完全兼容。

然而，这些产品有一些严重的局限性，例如：

* PgBouncer 不支持事务池模式下的 prepared statements。
* Odyssey 和 PgBouncer 都不支持事务池模式下的 SET 语句。

BMDB 包含一个内置连接池程序 BSQL 连接管理器，它提供与其他池解决方案相同的连接池优势，但没有这些限制。 由于管理器与产品捆绑在一起，因此可以方便地管理、监控和配置服务器连接。
![](../assets/chapter3/53.png)
## **主要特征**

BSQL 连接管理器是开源连接池 Odyssey 的修改版本。 BSQL 连接管理器在事务池模式下使用 Odyssey，并在线路协议级别进行了修改，以便与 BMDB 更紧密地集成，从而克服一些 SQL 限制。

BSQL连接管理器具有以下主要功能：

* 无 SQL 限制：与在事务模式下运行的其他池解决方案不同，BSQL 连接管理器支持 SQL 功能，例如 TEMP TABLE、WITH HOLD CURSORS 等。
* 每个数据库一个池：PgBouncer 和 Odyssey 为每个用户和数据库的组合创建一个池，这极大地限制了可以支持的用户数量，从而影响可扩展性。 然而，BSQL 连接管理器为每个数据库创建一个池 - 尝试访问同一数据库的所有连接共享适用于该数据库的同一个池。
* 支持会话参数：BSQL 连接管理器支持 SET 语句，这是其他连接池不支持的。
* 支持prepared statements：Odyssey 支持协议级prepared statements，BSQL 连接管理器继承了此功能。


## **使用 BSQL 连接管理器**

要使用 BSQL 连接管理器启动 BMDB 集群，请将 dbserver 标志 enable_bsql_conn_mgr 设置为 true。

当设置了enable_bsql_conn_mgr时，每个DBServer都会启动BSQL连接管理器进程以及PostgreSQL进程。 您应该看到每个 DBServer 都有一个 BSQL 连接管理器进程。

要使用 bm-ctl 通过 BSQL 连接管理器创建单节点集群，请使用以下命令：

```
./bin/bm-ctl start --dbserver_flags "enable_bsql_conn_mgr=true,allowed_preview_flags_csv=enable_bsql_conn_mgr" --ui false
```

由于enable_bsql_conn_mgr 只是预览标志，因此要使用它，请将该标志添加到 allowed_preview_flags_csv 列表中（即 allowed_preview_flags_csv=enable_bsql_conn_mgr）。

要创建大量客户端连接，请确保“SHMMNI”（操作系统允许的并发共享内存段的最大数量）以及 ulimit 设置正确，如下所示：
打开文件/etc/sysctl.conf。
在文件末尾添加 kernel.shmmni = 32768 （支持 30000 个客户端）。
要刷新设置，请使用 sudo sysctl -p。

**1.BSQL 连接管理器端口和标志**
默认情况下，当启用BSQL连接管理器时，它使用端口2521，并且后端数据库被分配一个随机的空闲端口。

要显式设置 BSQL 端口，您应该为标志 bsql_conn_mgr_port 和 bsql_port 指定端口。

下表描述了与 BSQL 连接管理器相关的 DBServer 标志：

| 标志                                 | 描述                                                         | 默认值  |
| ------------------------------------ | ------------------------------------------------------------ | ------- |
| enable_bsql_conn_mgr                 | 为集群启用 BSQL 连接管理器。 DBServer 将 BSQL Connection Manager 进程作为子进程启动。 | false   |
| bsql_conn_mgr_idle_time              | 指定 BSQL 连接管理器创建的数据库连接允许的最大空闲时间（以秒为单位）。 如果数据库连接保持空闲状态且没有为客户端连接提供服务的持续时间等于或超过此值，则 BSQL 连接管理器将自动关闭该连接。 | 60      |
| bsql_conn_mgr_max_client_connections | BSQL 连接管理器可以为每个池创建的最大并发数据库连接数。      | 10000   |
| bsql_conn_mgr_min_conns_per_db       | 池中存在的最小物理连接数。关闭断开的物理连接时不考虑此限制   | 1       |
| bsql_conn_mgr_num_workers            | BSQL 连接管理器使用的工作线程数。 如果设置为 0，工作线程数将为 CPU 核心数的一半。 | 0       |
| bsql_conn_mgr_stats_interval         | 更新 BSQL 连接管理器统计信息的时间间隔（以秒为单位）。       | 10      |
| bsql_conn_mgr_password               | BSQL 连接管理器用于创建数据库连接的密码。                    | bigmath |
| bsql_conn_mgr_username               | BSQL 连接管理器用于创建数据库连接的用户名。                  | bigmath |
| bsql_conn_mgr_warmup_db              | 需要进行预热的数据库。                                       | bigmath |
| enable_bsql_conn_mgr_stats           | 从 BSQL 连接管理器启用统计信息收集。 这些统计信息显示在端点 <ip_address_of_cluster>:8100/connections 处。 | true    |
| bsql_conn_mgr_port                   | 客户端可以连接的 BSQL 连接管理器端口。 这必须与通过 pgsql_proxy_bind_address 设置的 PostreSQL 端口不同。 | 2521    |
| bsql_conn_mgr_dowarmup               | 在 MBSQL 连接管理器中启用服务器连接的预创建。 如果设置为 false，则在 MBSQL 连接管理器中延迟（按需）创建服务器连接。 | false   |
