将 AiSQL 数据库提取到 SQL 脚本文件中。

## **概述**

bm-dump 是一个用于将 AiSQL 数据库备份到纯文本 SQL 脚本文件的实用程序。 bm-dump 可以进行一致的备份，即使数据库正在同时使用。 bm-dump 不会阻止其他用户访问数据库（读取器或写入器）。

bm-dump 仅转储单个数据库。 要备份集群中所有数据库共用的全局对象（例如角色），请使用 bm-dumpall。

转储以纯文本、SQL 脚本文件的形式输出。 脚本转储是纯文本文件，其中包含将数据库重建到保存时状态所需的 SQL 语句。 要从此类脚本恢复，请使用 sqlsh \i 元命令导入它。 即使在其他机器和其他架构上，脚本文件也可以用于重建数据库； 进行一些修改，甚至在其他 SQL 数据库产品上也是如此。

运行 bm-dump 时，您应该检查输出中是否有任何警告（在标准错误上打印）。

bm-dump 实用程序源自 PostgreSQL pg_dump 实用程序。

### **安装**

bm-dump 随 AiSQL 一起安装，位于 AiSQL 主目录的 postgres/bin 目录中。

### **在线帮助**

运行 bm-dump --help 显示在线帮助

## **语法**

```
bm-dump [ <connection-option>... ] [ <content-output-format-option> ... ] [ <dbname> ]
```

connection-option：请参阅数据库连接选项。
content-output-format-option：请参阅内容和输出格式选项
dbname：数据库的名称。

## **内容和输出格式选项**

以下命令行选项控制输出的内容和格式。
Dbname
指定要转储的数据库的名称。 如果未指定，则使用环境变量 PGDATABASE。 如果未设置，则使用为连接指定的用户名。

-a, --data-only
仅转储数据，而不转储架构（数据定义）。 转储表数据、大对象和序列值。

-b, --blobs
将大型物体放入转储中。 这是默认行为，除非指定 -n|--schema、-t|--table 或 -s|--schema-only。 因此，-b|--blobs 选项仅适用于将大对象添加到已请求特定架构或表的转储中。 请注意，blob 被视为数据，因此在使用 -a|--data-only 时将被包括在内，但在使用 -s|--schema-only 时则不会被包括在内。

-B, --no-blobs
排除转储中的大型对象。
当同时给出 -b|--blobs 和 -B|--no-blobs 时，行为是输出大对象，当转储数据时，请参阅 -b|--blobs 选项。

-c, --clean 
在输出创建数据库对象的语句之前，输出语句以清理（删除）数据库对象。 （除非还指定了 --if-exists，否则如果目标数据库中不存在任何对象，则恢复可能会生成一些无害的错误消息。）


-C, --create
以创建数据库本身并重新连接到创建的数据库的语句开始输出。 （对于这种形式的脚本，在运行脚本之前连接到目标安装中的哪个数据库并不重要。）如果还指定了 -c|--clean，则脚本会在重新连接到目标数据库之前删除并重新创建目标数据库。


-E encoding, --encoding=encoding 
以指定的字符集编码创建转储。 默认情况下，转储是以数据库编码创建的。 （获得相同结果的另一种方法是将 PGCLIENTENCODING 环境变量设置为所需的转储编码。）

-f file, --file=file 
将输出发送到指定文件。 对于基于文件的输出格式可以省略此参数，在这种情况下使用标准输出。

-m addresses, --mservers=addresses 
MServer 主机和端口的逗号分隔列表。

-n schema, --schema=schema 
仅转储与模式匹配的模式； 这会选择模式本身及其所有包含的对象。 当未指定此选项时，将转储目标数据库中的所有非系统模式。 可以通过编写多个 -n|--schema 选项来选择多个模式。 此外，schema 参数根据 sqlsh \d 命令使用的相同规则被解释为模式，因此也可以通过在模式中写入通配符来选择多个模式。 使用通配符时，如果需要，请小心引用模式，以防止 shell 扩展通配符。

注：当指定 -n|--schema 时，bm-dump 不会尝试转储所选模式可能依赖的任何其他数据库对象。 因此，不能保证特定模式转储的结果可以成功地自行恢复到干净的数据库中。
指定 -n|--schema 时，不会转储非架构对象（例如 blob）。 您可以使用 -b|--blobs 选项将 blob 添加回转储。

-N schema, --exclude-schema=schema 
不要转储任何与架构模式匹配的架构。 该模式根据与 -n|--schema 选项相同的规则进行解释。 -N|--exclude-schema 可以多次给出，以排除与多个模式中的任何一个匹配的模式。

当同时给出 -n|--schema 和 -N|--exclude-schema 时，行为是仅转储至少匹配一个 -n|--schema 选项但不匹配 -N|--exclude-schema 的模式 选项。 如果 -N|--exclude-schema 出现时没有 -n|--schema，则匹配 -N|--exclude-schema 的模式将从正常转储中排除。

-o, --oids
将对象标识符 (OID) 转储为每个表的数据的一部分。 如果您的应用程序以某种方式引用 OID 列（例如，在外键约束中），请使用此选项。 否则，不应使用此选项。

-O, --no-owner 
不要输出语句来设置对象所有权以匹配原始数据库。 默认情况下，bm-dump 发出 ALTER OWNER 或 SET SESSION AUTHORIZATION 语句来设置创建的数据库对象的所有权。 当脚本运行时，这些语句将失败，除非它是由超级用户（或拥有脚本中所有对象的同一用户）启动的。 要创建可由任何用户恢复的脚本，但会授予该用户所有对象的所有权，请指定 -O|--no-owner。


-s, --schema-only
仅转储对象定义（架构），而不转储数据。
此选项与 -a|--data-only 相反。

（不要将其与 -n|--schema 选项混淆，后者以不同的含义使用“schema”一词。）

要仅排除数据库中表的子集的表数据，请参阅 --exclude-table-data。

-S username, --superuser=username 
指定禁用触发器时要使用的超级用户用户名。 仅当使用 --disable-triggers 时这才相关。 （通常，最好将其忽略，而是以超级用户身份启动生成的脚本。）

-t table, --table=table 
仅转储名称与表匹配的表。 为此，“表”包括视图、物化视图、序列和外部表。 可以通过编写多个 -t|--table 选项来选择多个表。 此外，table 参数根据 sqlsh \d 命令使用的相同规则被解释为模式，因此也可以通过在模式中写入通配符来选择多个表。 使用通配符时，如果需要，请小心引用模式，以防止 shell 扩展通配符。

当使用 -t|--table 时，-n|--schema 和 -N|--exclude-schema 选项不起作用，因为无论这些选项如何，由 -t|--table 选择的表都将被转储，并且非 -表对象不会被转储。

注：当指定 -t|--table 时，bm-dump 不会尝试转储所选表可能依赖的任何其他数据库对象。 因此，不能保证特定表转储的结果可以成功地自行恢复到干净的数据库中。

-T table, --exclude-table=table
不要转储任何与表模式匹配的表。 该模式根据与 -t 相同的规则进行解释。 -T|--exclude-table 可以多次给出以排除与多种模式中的任何一个匹配的表。

当同时给出 -t|--table 和 -T|--exclude-table 时，行为是仅转储至少匹配一个 -t|--table 选项但不匹配 -T|--exclude-table 的表 选项。 如果 -T|--exclude-table 出现时没有 -t|--table，则匹配 -T|--exclude-table 的表将从正常转储中排除。

-v, --verbose
指定详细模式。 这会导致 bm-dump 将详细的对象注释以及开始和停止时间输出到转储文件，并将进度消息输出到标准错误。

-V, --version
打印 bm-dump 版本并退出。

-x, --no-privileges, --no-acl
防止转储访问权限（GRANT 和 REVOKE 语句）。

-Z 0..9, --compress=0..9
指定要使用的压缩级别。 零 (0) 表示不压缩。 对于纯文本输出，设置非零压缩级别会导致整个输出文件被压缩，就像通过 gzip 提供的一样； 但默认不压缩。

--column-inserts, --attribute-inserts
将数据转储为具有显式列名称的 INSERT 语句（INSERT INTO table (column, ...) VALUES ...）。 这使得恢复非常缓慢； 它主要有助于制作可加载到非 AiSQL 数据库中的转储。 但是，由于此选项为每一行生成单独的语句，因此重新加载行时发生错误只会导致该行丢失，而不是整个表内容丢失。

--disable-dollar-quoting 
此选项禁止对函数体使用美元引用，并强制使用 SQL 标准字符串语法对它们进行引用。

--disable-triggers
此选项仅在创建纯数据转储时相关。 它指示 bm-dump 包含在重新加载数据时临时禁用目标表上的触发器的语句。 如果您不想在数据重新加载期间调用表上的引用完整性检查或其他触发器，请使用此选项。

目前，为 --disable-triggers 发出的语句必须以超级用户身份完成。 因此，您还应该使用 -S|--superuser 指定超级用户名，或者最好小心地以超级用户身份启动生成的脚本。

--enable-row-security 
仅当转储具有行安全性的表的内容时，此选项才相关。 默认情况下，bm-dump 将 row_security 设置为 off，以确保从表中转储所有数据。 如果用户没有足够的权限来绕过行安全性，则会引发错误。 此参数指示 bm-dump 将 row_security 设置为 on，从而允许用户转储他们有权访问的表内容的部分。

请注意，如果您当前使用此选项，您可能还希望转储采用 INSERT 格式，因为还原期间的 COPY FROM 不支持行安全性。

--exclude-table-data=table
不要转储任何与表模式匹配的表的数据。 该模式根据与 -t|--table 相同的规则进行解释。 可以多次给出 --exclude-table-data 选项以排除与多种模式中的任何一种匹配的表。 当您需要定义特定表（即使不需要其中的数据）时，此选项很有用。

要排除数据库中所有表的数据，请参阅 -s|--schema-only。

--if-exists 
清理数据库对象时使用条件语句（即添加 IF EXISTS 子句）。 除非还指定了 -c|--clean，否则此选项无效。

--inserts
将数据转储为 INSERT 语句（而不是 COPY 语句）。 这会让恢复变得非常缓慢； 它主要有助于制作可加载到非 AiSQL 数据库中的转储。 但是，由于此选项为每一行生成单独的语句，因此重新加载行时发生错误只会导致该行丢失，而不是整个表内容丢失。 请注意，如果您重新排列了列顺序，则还原可能会完全失败。 --column-inserts 选项对于列顺序更改是安全的，尽管速度更慢。

--lock-wait-timeout=timeout 
不要永远等待在转储开始时获取共享表锁。 如果无法在指定的超时时间内锁定表，则会失败。 超时可以用 SET statements_timeout 接受的任何格式指定。 （允许的格式根据您转储的服务器版本而有所不同，但所有版本都接受整数毫秒。）

--no-publications 
不转储发布

--no-security-labels 
不转储安全标签

--no-subscriptions 
不要转储订阅。

--no-sync 
默认情况下，bm-dump 等待所有文件安全写入磁盘。 此选项导致 bm-dump 无需等待就返回，这速度更快，但意味着后续操作系统崩溃可能会使转储损坏。 通常，此选项有助于测试，但在从生产安装转储数据时不应使用。

--no-unlogged-table-data
不要转储未wal的表的内容。 该选项对于是否转储表定义（模式）没有影响； 它仅禁止转储表数据。 从备用服务器转储时，未记录表中的数据始终会被排除。

--quote-all-identifiers 
强制引用所有标识符。 当从 AiSQL 主要版本与 bm-dump 不同的服务器转储数据库时，或者当打算将输出加载到不同主要版本的服务器时，建议使用此选项。 默认情况下，bm-dump 仅引用在其自己的主要版本中属于保留字的标识符。 在处理可能具有稍微不同的保留字集的其他版本的服务器时，这有时会导致兼容性问题。 使用 --quote-all-identifiers 可以防止此类问题，但代价是转储脚本难以阅读。

--section=sectionname 
仅转储指定部分。 部分名称可以是前数据、数据或后数据。 可以多次指定此选项以选择多个部分。 默认是转储所有部分。

数据部分包含实际的表数据、大对象内容和序列值。 后数据项包括除已验证的检查约束之外的索引、触发器、规则和约束的定义。 前置数据项包括所有其他数据定义项。

--no-serializable-deferrable
使用 --no-serialized-deferrable 标志禁用默认的可串行化-可延迟事务模式。 可串行化-可延迟模式通过等待事务流中不存在异常的点来确保所使用的快照与后续数据库状态一致，从而不存在转储失败或导致其他事务回滚的风险 出现序列化失败。

如果有活动的读写事务，则转储开始之前的最大等待时间将为 50ms（基于 DBServer 和 MServer 服务器的默认 --max_clock_skew_usec。） 如果没有活动的读写事务 当 bm-dump 启动时，该选项不会产生任何影响。 一旦运行，无论有没有该选项，性能都是相同的。

--snapshot=snapshotname
转储数据库时使用指定的同步快照。 当需要将转储与逻辑复制槽或并发会话同步时，此选项非常有用。 在并行转储的情况下，将使用此选项定义的快照名称，而不是拍摄新快照。

--strict-names 
要求每个架构 (-n|--schema) 和表 (-t|--table) 限定符至少与要转储的数据库中的一个架构或表匹配。 请注意，如果没有任何模式或表限定符找到匹配项，即使没有 --strict-names，bm-dump 也会生成错误。

此选项对 -N|--exclude-schema、-T|--exclude-table 或 --exclude-table-data 没有影响。 未能匹配任何对象的排除模式不被视为错误。

--use-set-session-authorization 
输出 SQL 标准 SET SESSION AUTHORIZATION 语句而不是 ALTER OWNER 语句来确定对象所有权。 这使得转储更符合标准，但根据转储中对象的历史记录，可能无法正确恢复。 此外，使用 SET SESSION AUTHORIZATION 语句的转储肯定需要超级用户权限才能正确恢复，而 ALTER OWNER 语句需要较少的权限。

-?, --help
显示有关 bm-dump 命令行参数的帮助，然后退出

## **数据库连接选项**

以下命令行选项控制数据库连接参数。

-d dbname, --dbname=dbname 
指定要连接的数据库的名称。 这相当于将 dbname 指定为命令行上的第一个非选项参数。

如果此参数包含等号 (=) 或以有效的 URI 前缀 (brightdb://) 开头，则将其视为 conninfo 字符串。

-h host, --host=host
指定运行服务器的计算机的主机名。 如果该值以斜杠 (/) 开头，则用作 Unix 域套接字的目录。 默认为编译主机 127.0.0.1，否则尝试 Unix 域套接字连接。

-p port, --port=port
指定服务器侦听连接的 TCP 端口或本地 Unix 域套接字文件扩展名。 默认为编译端口 2521。

-U username, --username=username
连接的用户名。

-w, --no-password
切勿发出密码提示。 如果服务器需要密码身份验证，并且无法通过其他方式（例如 ~/.pgpass 文件）获得密码，则连接尝试将失败。 此选项在没有用户输入密码的批处理作业和脚本中非常有用。

-W, --password 
强制 bm-dump 在连接到数据库之前提示输入密码。

该选项从来都不是必需的，因为如果服务器要求密码身份验证，bm-dump 会自动提示输入密码。 然而，bm-dump 会浪费一次连接尝试来发现服务器需要密码。 在某些情况下，值得输入 -W|--password 以避免额外的连接尝试。

--role=rolename
指定用于创建转储的角色名称。 此选项导致 bm-dump 在连接到数据库后发出 SET ROLE <rolename> 语句。 当经过身份验证的用户（由 -U|--username 指定）缺乏 bm-dump 所需的权限，但可以切换到具有所需权限的角色时，它非常有用。 某些安装有禁止以超级用户身份直接登录的策略，使用此选项允许在不违反策略的情况下进行转储。

## **环境**

AiSQL 使用以下 PostgreSQL 环境变量（在某些 bm-dump 选项中引用）来实现 PostgreSQL 兼容性：

* PGHOST
* PGPORT
* PGOPTIONS
* PGUSER
* PGDATABASE
* PGCLIENTENCODING

该实用程序还使用 libpq 支持的环境变量。

## **诊断**

bm-dump 在内部执行 SELECT 语句。 如果运行 bm-dump 时遇到问题，请确保您能够使用 sqlsh 等从数据库中查询信息。 此外，libpq 前端库使用的任何默认连接设置和环境变量都将适用。

bm-dump 的数据库活动通常由统计收集器收集。 如果不希望出现这种情况，您可以使用 PGOPTIONS 或 ALTER USER 语句将参数 track_counts 设置为 false。

## **注意事项**

如果您的 AiSQL 集群对 template1 数据库有任何本地添加，请小心将 bm-dump 的输出恢复到真正的空数据库中； 否则，您可能会由于添加的对象的重复定义而出现错误。 要创建一个没有任何本地添加的空数据库，请从 template0 而不是 template1 复制，例如：

```
CREATE DATABASE foo WITH TEMPLATE template0;
```

当选择仅数据转储并使用选项 --disable-triggers 时，bm-dump 会在插入数据之前发出语句来禁用用户表上的触发器，然后在插入数据后发出语句来重新启用触发器。 如果恢复中途停止，系统目录可能会处于错误状态。

bm-dump 生成的转储文件不包含优化器用于制定查询计划决策的统计信息。 因此，从转储文件恢复后运行 ANALYZE 可以确保最佳性能。

由于 bm-dump 用于将数据传输到较新版本的 AiSQL，因此 bm-dump 的输出预计会加载到比 bm-dump 版本更新的 AiSQL 版本中。 bm-dump 还可以从早于其自身版本的 AiSQL 服务器转储。 但是，bm-dump 无法从比其主要版本更新的 AiSQL 服务器转储； 它甚至会拒绝尝试，而不是冒险进行无效转储。 此外，不保证 bm-dump 输出可以加载到较旧主要版本的服务器中 - 即使转储是从该版本的服务器中获取的。 将转储文件加载到旧服务器中可能需要手动编辑转储文件以删除旧服务器无法理解的语法。 建议在跨版本情况下使用 --quote-all-identifiers 选项，因为它可以防止不同 AiSQL 版本中不同的保留字列表引起的问题。

## **示例**

将数据库转储到 SQL 脚本文件中

```
$ bm-dump mydb > mydb.sql
```

转储名为 mytable 的单个表

```
$ bm-dump -t mytable mydb -f mytable_mydb.sql
```

基于过滤器的转储模式
以下命令转储名称以 east 或 west 开头并以 gsm 结尾的所有模式，不包括名称包含单词 test 的任何模式：

```
$ bm-dump -n 'east*gsm' -n 'west*gsm' -N '*test*' mydb > myschemas_mydb.sql
```

这是同一示例，使用正则表达式表示法来合并选项：

```
$ bm-dump -n '(east|west)*gsm' -N '*test*' mydb > myschemas_mydb.sql
```

根据过滤器转储所有数据库对象
以下命令转储除名称以 ts_ 开头的表之外的所有数据库对象：

```
$ bm-dump -T 'ts_*' mydb > objects_mydb.sql
```