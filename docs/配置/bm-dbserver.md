使用bm-dbserver二进制文件及其标志来配置dbserver服务器。bm-dbserver可执行文件位于AiSQL 安装目录的bin目录中。

## **基本语法**

基本语法：

```
./ bm-dbserver [ flag ]
```

## **通用标志**

### **--version**

显示版本和编译信息，然后退出。

### **--flagfile**

指定加载标志的配置文件

### **--dbserver_mserver_addrs（必须项）**

指定所有mserver RPC地址的逗号分隔列表（必须项）。

注意：
逗号分隔的值的数量应与mserver服务器的总数（或复制因子）相匹配。
默认值：127.0.0.1:11000

### **--fs_data_dirs（必须项）**

指定以逗号分隔的装载目录列表，bm-server将添加bm-data/dbserver数据目录、dbserver.err、dbserver.out文件和pg_data目录。
注意：不支持在创建集群后再更改此标志的值。

### **--fs_wal_dirs**

指定以逗号分隔的目录列表，bm-dbserver将在其中存储预写（WAL）日志。这可以与--fs_data_dirs中列出的某个目录相同，但不是数据目录的子目录。
默认值与 --fs_data_dirs 相同。


### **--rpc_bind_addresses**

指定要为RPC连接绑定到的网络接口地址的逗号分隔列表。
在所有bm-mserver和bm-dbserver配置中，使用的值必须匹配。
默认值：运行服务器的主机的专用IP地址。例如：
--rpc_bind_addresss=172.x.x.x:21000

如果以下情况适用，请确保server_broadcast_addresss标志设置正确：

1)  rpc_bind_addresss设置为0.0.0.0
2)  rpc_bind_addresss涉及公共IP地址，例如0.0.0.0:21000，它指示服务器侦听所有可用的网络接口。

### **--max_clock_skew_usec**

部署任意两个服务器之间的预期最大时钟偏移，单位为微秒（µs）。
默认值：500000 (500,000 µs = 500ms)

### **--server_broadcast_addresses**

指定服务器的公共IP或DNS主机名（带有可选端口）。此值用于服务器相互通信，具体取决于连接策略参数。
默认值：""

### **--tablet_server_svc_queue_length**

指定分片服务器用于从应用程序进行读写的队列大小。
默认值：5000

### **--dns_cache_expiration_ms**

指定缓存的DNS解析到期之前的持续时间（以毫秒为单位）。当使用主机名而不是IP地址时，必须查询DNS解析程序以将主机名与IP地址相匹配。通过使用本地DNS缓存临时存储DNS查找，可以更快地解决DNS查询，并可以避免额外的查询。这减少了延迟，缩短了加载时间，并减少了带宽和CPU消耗。

默认值：60000 (1分钟)

注意：
如果更改此值，请确保将相同的值添加到所有mserver和TSever配置中。

### **--use_private_ip**

指定用于确定何时使用专用IP地址进行节点间通信的策略。可能的值为never、zone、cloud和region。基于地理分布标志的值。

该策略的有效值为：
1） never：始终使用--server_broadcast_addresss。
2） zone：使用区域内的私有IP；在区域外使用--serverbroadcast_addresss。
3） Region：在一个区域的所有区域中使用私有IP地址；在区域外使用--server_broadcast_addresss。

默认值：never

### **--webserver_interface**

指定web服务器用户界面访问的绑定地址。 
默认值：0.0.0.0（127.0.0.1）

### **--webserver_port**

指定web服务器监听的端口。
默认值：20000

### **--webserver_doc_root**

监控web服务的目录。
默认值：AiSQL主目录中的www目录。 

### **--webserver_certificate_file**

用于web服务器的SSL证书文件的位置（.pem格式）。如果为空，则不会为web服务器启用SSL。
默认值：""

### **--webserver_authentication_domain**

用于.htpasswd身份验证的域。这应与--webserver_password_file一起使用。 
默认值：""

### **--webserver_password_file**

包含用户名和哈希密码的.htpasswd文件的位置，用于对web服务器进行身份验证。
默认值：""


## **日志标志**

### **--logtostderr**

将日志信息写入stderr，而不是写入日志文件。
默认值：false

### **--log_dir**

写dbserver日志文件的目录。
默认值：与--fs_data_dirs相同

### **--max_log_size**

最大的日志尺寸大小，单位为MB，值0将被静默重写为1。
默认值：1800 (1.8 GB)

### **--minloglevel**

记录消息的最低级别，有效值为：0（信息）；1（警告）；2（错误）；3（致命）。
默认值： 0（信息）

### **--stderrthreshold**

除日志文件外，此级别或更高级别的日志消息也会复制到stderr。
默认值： 2

### **--stop_logging_if_full_disk**

如果磁盘已满，停止写磁盘日志。
默认值： false

## **Raft标志**

对于典型的部署，dbserver配置中用于Raft和预写日志（WAL）标志的值应该与[bm-mserver](#_bi-master)配置中的值匹配。

### **--follower_unavailable_considered_failed_sec**

持续时间，以秒为单位，在该持续时间之后，由于Leader没有接收到心跳，followers被认为失败。然后从配置中逐出跟随者，并在其他地方重新复制数据。
默认值： 900(15分钟)

--follower_unavailable_consided_failed_sec值应与--log_min_seconds_to_retain的值匹配。

### **--evict_failed_followers**

失败的followers将被逐出Raft组，数据将被重新复制。
默认值： true

### **--leader_failure_max_missed_heartbeat_periods**

在leader被视为失败之前，leader可能无法检测到的最大检测信号周期。总故障超时（以毫秒为单位）是--raft_heartbeat_interval_ms乘以
--leader_failure_max_missed_heartbeat_periods。

对于读取副本集群，在所有bm-dbserver和bm-mserver配置中都将该值设置为10。由于数据是全局复制的，RPC延迟会更高。在这样一个更高的RPC延迟部署中，使用此标志可以增加故障检测间隔。

默认值： 6

### **--leader_lease_duration_ms**

Leader租约持续时间，以毫秒为单位。每次达成共识后，leader都会不断建立新的租约或延长现有租约。新服务器不允许作为leader（即，提供最新的读取请求或确认写入请求），直到旧leader的租约明确到期，或者旧leader明确承认新leader的租约。

此租约允许leader在租约期间安全地提供读取服务，即使在网络分区期间也是如此。有关详细信息，请参阅leader租约。

Leader租约持续时间应长于检测信号间隔，并且小于--leader_failure_max_missed_heartbeat_periods乘以--raft_heartbeat_interval_ms的倍数。

默认值：2000 

### **--raft_heartbeat_interval_ms**

Raft复制的检测心跳间隔时间，以毫秒为单位。leader在这个时间间隔内向followers 发出心跳。followers 期望在这个时间间隔内有心跳，如果一个leader连续错过几个，就会认为他失败了。 
默认值：500 

### **--max_stale_read_bound_time_ms**

指定follower向leader转发读取请求之前的最大边界（持续时间）（以毫秒为单位）。

在地理分布的集群中，follower距离分片leader很远，可以使用此设置来增加最大边界。
默认值：10000 (10 秒)

## **预写日志（WAL）标志**

确保bm-dbserver配置中用于预写日志（WAL）的值与bm-mserver配置中的值匹配。 

### **--fs_wal_dirs**

bm-dbserver保留WAL文件的目录。可能与--fs_data_dirs中列出的某个目录相同，但不是数据目录的子目录。
默认值： 与--fs_data_dirs相同

### **--durable_wal_write**

如果设置为false，则每隔interval_durable_wal_write_ms（毫秒）或每隔bytes_durable_wal_write_mb兆字节（MB）（以先到者为准），对WAL的写入将同步到磁盘。此默认设置仅建议用于多可用区或多区域部署，其中可用区或区域是独立的故障域，并且不存在相关电源损失的风险。对于单个可用区（AZ）部署，此标志应设置为true。
默认值： false

### **--interval_durable_wal_write_ms**

当--durable_wal_write为false时，每隔 --interval_durable_wal_write_ms 或 --bytes_durable_wal_write_mb（以先到者为准），对于WAL的写入将同步到磁盘。
默认值： 1000

### **--bytes_durable_wal_write_mb**

当--durable_wal_write为false时，每隔 --bytes_durable_wal_write_mb 或 --interval_durable_wal_write_ms（以先到者为准），对于WAL的写入将同步到磁盘。
默认值： 1

### **--log_min_seconds_to_retain**

无论持久性要求如何，保持WAL段的最小持续时间（以秒为单位）。WAL段可以保留更长的时间，如果它们是正确重新启动所必需的。该值应设置足够长的时间，以便在给定的时间段内重新启动暂时出现故障的分片服务器。
默认值：900(15分钟)

### **--log_min_segments_to_retain**

无论持久性要求如何，要保留的WAL段（文件）的最小数量。该值必须至少为1。 
默认值：2

### **--log_segment_size_mb**

WAL段（文件）的大小，以兆字节（MB）为单位。当WAL段达到指定的大小时，将发生日志翻转，并创建一个新的WAL段文件。
默认值：64

### **--reuse_unclosed_segment_threshold_bytes**

当服务器从上一次崩溃中重新启动时，如果分片的最后一个WAL文件大小小于或等于此阈值，则将重用最后的WAL文件。否则，WAL将在引导程序中分配一个新文件。要禁用WAL重用，请将该值设置为-1。

默认值：2.18.1中的默认值为-1，默认情况下禁用该功能；从2.19.1开始的默认值为524288（0.5 MB），默认情况下启用该功能。

## **分片标志**

### **--bm_num_shards_per_dbserver**

创建用户表时，每个dbserver的每个BCQL表的的分片数。

默认值：-1（基于CPU内核的值在运行时计算）。
对于最多有两个CPU核心的服务器，默认值被认为是4。
对于三个或三个以上的CPU内核，默认值被认为是8。
如果enable_automatic_tablet_spliting为true，则默认值被视为1，表将以每个节点1个分片开始；对于2.18及更高版本，对于最多有4个CPU核心的服务器，没有定义该值，表将以每个集群1分片（对于最多有2个CPU核心）或2分片开始（对于最多4个CPU内核的服务器）。
使用bm-dev-ctl和bm-docker-ctl创建的本地集群安装使用该标志的默认值2。
使用biginsighted创建的集群始终使用默认值1。

注意：此值必须在AiSQL集群的所有bm-mserver和bm-dbserver配置上匹配。
如果该值设置为默认值（-1），则系统会根据CPU核的数量自动确定一个适当的值，并在2.18版本之前的启动过程中使用预期值对标志进行内部更新，并且标志从2.18版本开始保持不变。

CREATE TABLE ... WITH TABLETS = <num> 子句可以在每个表的基础上用于覆盖bi_num_shards_per_dbserver值。

### **--bsql_num_shards_per_dbserver**

创建用户表时，每个BSQL表的每个dbserver的分片数。

默认值：-1（基于CPU内核的值在运行时计算）。
对于最多有两个CPU核心的服务器，默认值被认为是2。
对于具有三个或四个CPU核心的服务器，默认值被认为是4。
在四个核之外，默认值被认为是8。
如果enable_automatic_tablet_spliting为true，则默认值被视为1，表将以每个节点1个分片开始；对于2.18及更高版本，对于最多有4个CPU核心的服务器，没有定义该值，表将以每个集群1分片（对于最多有2个CPU核心）或2分片开始（对于最多4个CPU内核的服务器）。
使用bm-dev-ctl和bm-docker-ctl创建的本地集群安装使用该标志的默认值2。
使用biginsighted创建的集群始终使用默认值1。

注意：此值必须在AiSQL集群的所有bm-mserver和bm-dbserver配置上匹配。
如果该值设置为默认值（-1），则系统会根据CPU核的数量自动确定一个适当的值，并在2.18版本之前的启动过程中使用预期值对标志进行内部更新，并且标志从2.18版本开始保持不变。

CREATE TABLE ...SPLIT INTO语句可以在每个表的基础上用于覆盖bsql_num_shards_per_dbserver值。

### **--cleanup_split_tablets_interval_sec**

分片管理器尝试清理不再需要的拆分分片的时间间隔。将此设置为0将禁用拆分分片的清理。
默认值：60

### **--enable_automatic_tablet_splitting**

启用AiSQL自动拆分分片
默认值：true

注意：此值必须在AiSQL集群的所有bm-mserver和bm-dbserver配置上匹配。

### **--full_compaction_pool_max_threads**

非管理员完全压缩所允许的最大线程数。这包括拆分后压缩（在拆分后从新分片中删除无关数据的压缩）和已计划的完全压缩。
默认值：1

### **--full_compaction_pool_max_queue_size**

可以同时排队的最大完整压缩任务数。这包括拆分后压缩（在拆分后从新分片中删除不相关数据的压缩）和已计划的完全压缩。
默认值：200

### **--auto_compact_check_interval_sec**

完全压缩任务将检查符合压缩条件的分片的间隔（包括基于统计的完全压缩和已计划的完全压缩）。0表示禁用了基于统计信息的完全压缩功能。 
默认值：60

### **--auto_compact_stat_window_seconds**

为触发完全压缩以提高读取性能，而分析CoreDB读取统计信息的时间窗口（以秒为单位）。auto_compact_percent_obsolete和auto_compact _min_obsolete_keys_found 都是在这段时间内进行评估的。
auto_compact_stat_window_seconds必须计算为auto_compact_check_interval_sec的倍数，并将四舍五入以满足此约束。例如，如果将auto_compact_stat_window_seconds设置为100，将auto_compact_check_interval_sec设置为60，则在运行时将四舍五入到120。
默认值：300

### **--auto_compact_percent_obsolete**

在分片上触发自动完全压缩所需的auto_compact_stat_window_seconds时间窗口中读取的过时密钥（占总密钥）的百分比。只有超过其历史保留期的密钥（可以进行垃圾回收）才会计入该阈值。

例如，如果标志设置为99，并且在该时间窗口内读取了100000个密钥，并且其中99900个密钥已过时并超过了其历史保留期，则将触发完全压缩（取决于其他条件）。
默认值：99

### **--auto_compact_min_obsolete_keys_found**

在上一次auto_compact_stat_window_seconds中必须读取的最小键数，以触发基于统计信息的完全压缩。
默认值：10000

### **--auto_compact_min_wait_between_seconds**

Minimum wait time between statistics-based and scheduled full compactions. To be used if statistics-based compactions are triggering too frequently.
基于统计信息的完全压缩和计划的完全压缩之间的最短等待时间。如果基于统计的压缩触发过于频繁，则需要使用它。
默认值：0

### **--scheduled_full_compaction_frequency_hours**

分片应计划完全压缩的频率。0表示该功能已禁用。建议值：720小时或更长时间（即30天）。
默认值：0

### **--scheduled_full_compaction_jitter_factor_percentage**

在确定每片分片的完全压缩计划时，用作抖动的scheduled_full_compaction_frequency_hours的百分比，必须是介于0和100之间的值。引入抖动是为了防止许多分片同时被安排进行完全压缩。
抖动是在调度压缩时决定性地计算的，在0和（频率 * 抖动因子）小时之间。计算后，从预期压缩频率中减去抖动，来确定分片的下一次压缩时间。
例如：如果scheduled_full_compaction_frequency_hours为720小时（即30天），并且scheduled_full_compaction_jitter_factor_percentage为33%，则每482小时至720小时将计划一次压缩。
默认值：33

### **--automatic_compaction_extra_priority**

当启用自动分片拆分时，为自动（次要）压缩指定额外的优先级。这降低了分割后压缩的优先级，并确保较小的压缩不会被耗尽。建议的值介于0和50之间。
默认值：50


## **地理分布标志**

与配置管理地理分布式集群相关的设置。

### **--placement_zone**

部署此实例的可用性区域（AZ）或机架的名称。
默认值：rack1

### **--placement_region**

部署此实例的地域或数据中心的名称。
默认值：datacenter1

### **--placement_cloud**

部署此实例的云的名称。
默认值：cloud1

### **--placement_uuid**

集群的唯一标识符。
默认值：""

### **--force_global_transactions**

如果为true，则强制通过此实例的所有事务，始终使用system.transactions事务状态表的全局事务。这相当于始终将BSQL参数force_global_transaction设置为TRUE。

注意：
全局事务延迟
应尽可能避免设置此标志。所有分布式事务都可以作为全局事务毫无问题地运行，但提交事务时可能会有更高的延迟，因为AiSQL必须在多个区域之间达成共识才能写入system.transactions。必要时，最好选择性地设置BSQL参数force_global_transaction=TRUE，而不是设置此标志。

默认值：false

### **--auto-create-local-transaction-tables**

如果为true，则将在每个BSQL表空间下创建事务状态表，该表空间具有一个放置集，并至少包含一个其他表。 
默认值：true

### **--auto-promote-nonlocal-transactions-to-global**

如果为true，则在访问本地区域以外的数据时，使用system.transactions以外的事务状态表的本地事务，将自动升级为使用system.transactions 事务状态表的全局事务。
默认值：true

## **xDCR标志**

与管理xDCR相关的设置。 

### **--xDCR_svc_queue_size**

xDCR服务的RPC队列大小。应与用于读取和写入请求的tablet_server_svc_queue_length的大小相匹配。
默认值：5000

## **API标志**

### **BSQL**

以下为支持使用BSQL API的标志： 

#### --enable_bsql

启用BSQL API
默认值：true

确保bm-dbserver配置中的enable_bsql值与bm-mserver配置中的值匹配。

#### --sql_enable_auth

启用BSQL身份认证
当启用BSQL身份认证时，可以使用默认密码为biginsights，默认用户为biginsights，登录到sqlsh。
默认值：false

#### --bsql_enable_profile

启用BSQL登录配置文件。

启用BSQL登录配置文件后，可以设置用户登录失败次数的限制。

默认值：false

#### --pgsql_proxy_bind_address

指定BSQL API的TCP/IP绑定地址。默认值为0.0.0.0:2521，允许侦听所有IPv4地址访问localhost上的端口2521。--pgsql_proxy_bind_address值覆盖控制哪些接口接受连接尝试的listen_address（默认值127.0.0.1:2521）。

要指定可以访问服务器的细粒度访问控制，请使用--bsql_hba_conf_csv。 

默认值：0.0.0.0:2521

#### --pgsql_proxy_webserver_port

指定监听BSQL的web服务器端口。

默认值：8100

#### --bsql_hba_conf_csv

指定写入bsql_hba.conf文件的PostgreSQL客户端身份验证设置的逗号分隔列表。要查看bsql_hba.conf文件中的当前值，请运行SHOW hba_file；然后查看文件内容。由于文件是自动生成的，因此自动生成的内容会覆盖直接编辑的内容。
有关使用--bsql_hba_conf.csv指定客户端身份验证的详细信息，请参阅基于主机的身份验证。
如果设置包含逗号（，）或双引号（“），则该设置应包含在双引号中。此外，带引号文本中的双引号应加倍（“”）。
假设有如下两个配置内容：
host all all 127.0.0.1/0 password 
host all all 0.0.0.0/0 ldap ldapserver=\*\*\*\*\* ldapsearchattribute=cn ldapport=3268 ldapbinddn=\*\*\*\*\* ldapbindpasswd="\*\*\*\*\*"

由于在第二个配置内容中，其中的一个设置被双引号（“）引起，因此按照规则，需要将引号加倍，并将整个配置内容用引号括起来，如下所示：
 "host all all 0.0.0.0/0 ldap ldapserver=\*\*\*\*\* ldapsearchattribute=cn ldapport=3268 ldapbinddn=\*\*\*\*\* ldapbindpasswd=""\*\*\*\*\*"""
要设置标志，请使用逗号（，）连接配置项，并将最终标志值用单引号（'）括起来，如下所示：
--bsql_hba_conf_csv='host all all 127.0.0.1/0 password,"host all all 0.0.0.0/0 ldap ldapserver=\*\*\*\*\* ldapsearchattribute=cn ldapport=3268 ldapbinddn=\*\*\*\*\* ldapbindpasswd=""\*\*\*\*\*""" '

默认值："host all all 0.0.0.0/0 trust,host all all ::0/0 trust"

#### --bsql_pg_conf_csv

追加到postgresql.conf文件的PostgreSQL服务器配置参数的逗号分隔列表。

例如：

```
--bsql_pg_conf_csv="suppress_nonpg_logs=true,log_connections=on"
```

有关PostgreSQL服务器配置参数的信息，请参阅PostgreSQL文档中的服务器配置部分。
AiSQL的服务器配置参数与PostgreSQL的相同，只是有一些小的例外。参考[PostgreSQL服务器选项](#_PostgreSQL服务器选项)。

#### --bsql_timezone

指定用于显示和解释时间戳的时区。
默认值：使用BSQL时区

#### --bsql_datestyle

指定数据和时间值的显示格式。
默认值：使用BSQL显示格式

#### --bsql_max_connections

指定并发BSQL连接的最大数目。
默认值：超级用户为300。非超级用户角色只能看到可供使用的连接，而超级用户可以看到所有连接，包括为超级用户保留的连接

#### --bsql_default_transaction_isolation

指定默认事务隔离级别。
有效的值为：SERIALIZABLE，REPEATABLE READ，READ COMMITTED，和READ UNCOMMITTED。

默认值：READ COMMITTED

注：Read Committed支持目前处于测试版。只有当bm-dbserver标志bi_enable_read_committed_isolation设置为true时，才支持Read Committed隔离级别。默认情况下，此标志为false，在这种情况下，AiSQL事务层的Read Committed隔离级别回落到更严格的Snapshot隔离级别（在这种情况中，BSQL的READ COMMITTED 和READ UNCOMMITTED也依次使用Snapshot隔离级别）。

#### --bsql_disable_index_backfill

将此标志设置为false可启用联机索引回填。当设置为false时，联机索引构建将在联机时运行，而不会导致其他并发写入失败。

有关联机索引回填工作方式的详细信息，请参阅联机索引回填设计文档。
默认值：false

#### --bsql_sequence_cache_method

指定缓存序列值的位置。
有效的值是：connection 和server
此标志要求bm-dbserver 的bi_enable_sequence_pushdown标志为true（默认值）。否则，无论此标志的值如何，都将发生默认行为。
有关在服务器上缓存值和缓存方法之间切换的详细信息，请参阅nextval。

默认值：connection

#### --bsql_sequence_cache_minval

为每个序列对象指定要缓存在客户端中的序列值的最小数目。
若要关闭缓存的默认大小标志，请将该标志设置为0。
有关与序列缓存子句一起使用时的预期行为的详细信息，请参阅CREATE SEQUENCE和ALTER SEQUEQUENCE。

默认值：100

#### --bsql_log_statement

指定应记录的BSQL语句的类型。

有效值：none（off）、ddl（仅数据定义查询，如create/alter/drop）、mod（所有修改/写入语句，包括ddl加上insert/update/delete/trunctate等）和all（所有语句）。 

默认值：none

#### --bsql_log_min_duration_statement

记录运行指定时间（以毫秒为单位）或更长时间以上的SQL语句。将值设置为0将打印所有语句持续时间。您可以使用此标志来帮助跟踪未优化（或“慢查询”）的查询。
默认值：-1（禁用日志记录语句持续时间）

#### --bsql_log_min_messages

指定要记录的最低BSQL消息级别
有效值：info, notice, warning, error, log, fatal

默认值：warning

### **BCQL**

以下为支持使用BCQL API的标志： 

#### --use_cassandra_authentication

指定true可启用BCQL身份验证（用户名和密码），启用BCQL安全语句（CREATE ROLE、DROP ROLE、GRANT ROLE、REVOKE ROLE、GRANT PERMISSION和REVOKE PERMISSION），并强制执行BCQL语句的权限。 
默认值：false

#### --cql_proxy_bind_address

指定绑定BCQL API的绑定地址
默认值：0.0.0.0:9542 (127.0.0.1:9542)

#### --cql_proxy_webserver_port

指定监听BCQL的web服务器端口。
默认值：8200

#### --cql_table_is_transactional_by_default

指定是否在创建BCQL表时默认启用事务。
默认值：false

#### --bcql_disable_index_backfill

将此标志设置为false可启用联机索引回填。当设置为false时，联机索引构建将在联机时运行，而不会导致其他并发写入失败。

有关联机索引回填工作方式的详细信息，请参阅联机索引回填设计文档。

默认值：true

#### --bcql_require_drop_privs_for_truncate

将此标志设置为true可拒绝TRUNCATE语句，除非DROP TABLE权限允许。
默认值：false

#### --bcql_enable_audit_log

将此标志设置为true，可启用审核日志记录。
有关详细信息，请参阅BQL API的审核日志记录。 

默认值：false

#### --bcql_allow_non_authenticated_password_reset

将此标志设置为true，可使超级用户重置密码。 
默认值：false

请注意，要启用密码重置功能，必须首先将use_cassandra_authentication标志设置为false

### **BEDIS**

以下为支持使用BEDIS API的标志： 

#### --redis_proxy_bind_address

指定绑定BEDIS API的绑定地址

默认值：0.0.0.0:6879

#### --redis_proxy_webserver_port

指定监听BEDIS 的web服务器端口。
默认值：8300

## **性能标志**

### **--enable_ondisk_compression**

开启SSTable压缩
默认值：true

### **--compression_type**

更改SSTable 压缩类型。有效的压缩类型有Snappy、Zlib、LZ4和NoCompression
默认值：Snappy

如果选择了一个无效的选项，集群将不会相应。
如果更改此标志，则更改将在重新启动群集节点后生效。
支持在现有数据库上更改此标志；分片可以有效地具有不同压缩类型的SST。最终，压缩将删除旧的压缩类型文件。

### **--regular_tablets_data_block_key_value_encoding**

在RocksDB中的数据块，使用键-值编码。可能的选项有：shared_prefix、three_shared_parts。
默认值：three_shared_parts

### **--rocksdb_compact_flush_rate_limit_bytes_per_sec**

用于控制内存存储刷新和SSTable文件压缩的速率。
默认值：1GB (1 GB/秒)

### **--rocksdb_universal_compaction_min_merge_width**

只有当至少有符合rocksdb_universal_compaction_min_merge_width条件的文件，并且它们的运行总大小（到目前为止考虑的文件大小的总和），在考虑包含在同一压缩中的下一个文件的rocksdb_universal_compaction_size_ratio内时，才会进行压缩。
默认值：4

### **--rocksdb_max_background_compactions**

执行后台压缩的最大线程数（在压缩需要追赶时使用）。除非rocksdb_disable_compactions=true，否则不能将其设置为零。
默认值：-1（该值在运行时计算）。
对于最多具有4个CPU核心的服务器，默认值被视为1。
对于最多具有8个CPU核心的服务器，默认值被视为2。
对于最多具有32个CPU核心的服务器，默认值被视为3。
超过32个核心，默认值为4

### **--rocksdb_compaction_size_threshold_bytes**

超过该阈值，压缩被视为较大。
默认值：2GB

### **--rocksdb_level0_file_num_compaction_trigger**

触发0级压缩的文件数。如果压缩根本不应该由文件数触发，则设置为-1。
默认值：5

### **--rocksdb_universal_compaction_size_ratio**

只有当至少有符合rocksdb_universal_compaction_min_merge_width条件的文件，并且它们的运行总大小（到目前为止考虑的文件大小的总和）在考虑包含在同一压缩中的下一个文件的rocksdb_universal_compaction_size_ratio内时，才会运行压缩。
默认值：20

### **--timestamp_history_retention_interval_sec**

保留历史/旧版本数据的时间间隔（以秒为单位）。压缩后可能不允许在此间隔之前的混合时间进行时间点读取，并返回快照太旧的错误。应将其设置为大于应用程序中任何单个事务的预期最长持续时间。
默认值：900 (15 分钟)

### **--remote_bootstrap_rate_limit_bytes_per_sec**

所有分片的速率控制，都是从该进程远程引导，或远程引导到该进程的。
默认值：256MB (256 MB/秒)

## **网络压缩标志**

### **--enable_stream_compression**

控制AiSQL是否使用RPC压缩。
默认值：true

### **--stream_compression_algo**

指定要使用的RPC压缩算法。要求将enable_stream_compression设置为true。有效值为：
0: No compression (默认值)
1: Gzip
2: Snappy
3: LZ4

在大多数情况下，LZ4提供了压缩性能与CPU开销之间的最佳折衷。
要从不支持RPC压缩的旧版本（如2.4），升级到支持RPC压缩（如2.6）的新版本，您需要执行以下操作：

* 滚动重启，将AiSQL升级到支持压缩的版本。
* 通过设置enable_stream_compression=true，在bm-mserver和bm-dbserver上滚动重启，以启用压缩。
  请注意，如果您要升级到的AiSQL版本在默认情况下已经启用了压缩，则可以省略此步骤。
* 滚动重启可在bm-mserver和bm-dbserver上设置要使用的压缩算法，例如通过设置stream_compression_algo=3。

## **安全标志**

有关启用服务器到服务器加密的详细信息，请参见服务器到服务器加密

### **--certs_dir**

包含此服务器的证书颁发机构、私钥和证书的目录。
默认值："" (\<data drive\>/bm-data/mserver/data/certs.)

### **--allow_insecure_connections**

允许不安全的连接。设置为false可防止任何通信未加密的进程加入群集。请注意，此标志要求启用use_node_to_node_encryption，并启用use_client_to_server_encryption 。
默认值：true

### **--certs_for_client_dir**

包含此服务器的证书颁发机构、私钥和证书的目录，这些证书应用于客户端到服务器的通信。

默认值："" (使用与服务器到服务器通信相同的目录。)

### **--dump_certificate_entries**

添加证书条目，包括IP地址和主机名，记录握手错误消息。启用此标志有助于调试证书问题。
默认值：false

### **--use_client_to_server_encryption**

客户端到服务器进行加密。
默认值：false

### **--use_node_to_node_encryption**

在集群的mserver和dbserver服务器之间，启用服务器到服务器，或者节点到节点的加密。为了正常工作，所有mserver服务器还必须启用其 --use_node_to_node_encryption标志。启用时，必须禁用--allow_insecure_connections标志。
默认值：false

### **--cipher_list**

为TLS 1.2及更早版本指定密码列表。（对于TLS 1.3，请使用--ciphersuite。）按首选项顺序使用以冒号分隔的TLS 1.2密码名称列表。使用感叹号（！）排除密码。例如：
--cipher_list DEFAULTS:!DES:!IDEA:!3DES:!RC2

这允许TLS 1.2的所有密码都被接受，除了那些与省略的密码类别匹配的密码。
此标志需要重新启动或滚动重新启动。

默认值：DEFAULTS

### **--ciphersuite**

指定TLS 1.3的密码列表。对于TLS 1.2及更早版本，请使用--cipher_list。
按照首选项的顺序使用以冒号分隔的TLS 1.3密码套件名称列表。使用感叹号（！）排除密码。例如：
--ciphersuite DEFAULTS:!CHACHA20

这允许接受TLS 1.3的所有密码套件，但CHACHA20密码除外。
此标志需要重新启动或滚动重新启动。

默认值：DEFAULTS

### **--ssl_protocols**

为AiSQL的内部RPC通信指定TLS协议的显式允许列表。
默认值：空字符串，相当于允许除“ssl2”和“ssl3”之外的所有协议。

可以传递一个以逗号分隔的字符串列表，其中的字符串可以是“ssl2”、“ssl3”、“tls10”、“ttls11”、“tls12”和“tls13”之一。

可以设置节点到节点和客户端节点通信的TLS版本。要强制执行TLS 1.2，请将标志设置为tls12，如下所示： 

--ssl_protocols = tls12

要指定1.2的最低TLS版本，需要将标志设置为tls12、tls13，以及所有可用的后续版本，如下所示：

--ssl_protocols = tls12,tls13

此外，由于此设置不会传到PostgreSQL，建议通过如下设置bsql_pg_conf_csv标志，来指定PostgreSQL的最低TLS版本（ssl_min_procol_version）：
--bsql_pg_conf_csv="ssl_min_protocol_version=TLSv1.2"

## **压缩行标志（测试版）**

要了解压缩行功能，请参阅体系结构部分中的压缩行格式内容

### **--bsql_enable_packed_row**

是否为BSQL启用了压缩行
默认值：false

### **--bsql_packed_row_size_limit**

BSQL的压缩行大小限制。默认值为0（使用块大小作为限制）。对于超过此大小限制的行，将打包尽可能多的列，其余的列则做为单独的键值对存储。 
默认值：0

### **--bcql_enable_packed_row**

是否为BCQL启用了压缩行
默认值：false

### **--bcql_packed_row_size_limit**

BCQL的压缩行大小限制。默认值为0（使用块大小作为限制）。对于超过此大小限制的行，将打包尽可能多的列，其余的列则做为单独的键值对存储。
默认值：0

### **--bsql_enable_packed_row_for_colocated_table**

是否为BSQL中的[共址表(colocated table)](#_共用表（colocated table）)启用了压缩行。
默认值：false

## **更改数据捕获（CDC）标志**

对于更改数据捕获（CDC）的更多详细信息，请参见更改数据捕获（CDC）。

### **--cdc_state_checkpoint_update_interval_ms**

CDC状态检查点的更新速率。
默认值：15000

### **--cdc_biclient_reactor_threads**

用于处理biclient对CDC的请求的反应器线程数。增大值可以提高对大的分片设置的吞吐量。
默认值：50

### **--cdc_max_stream_intent_records**

单个CDC批次中允许的最大意向记录数。
默认值：1000

### **--cdc_snapshot_batch_size**

在CDC的快照操作的单个批次中获取的记录数。
默认值：250

### **--cdc_min_replicated_index_considered_stale_secs**

如果cdc_min_replicated_index在这段时间内没有被复制，我们将其值重置为max int64，以避免保留任何日志。
默认值：900 (15分钟)

### **--timestamp_history_retention_interval_sec**

保留历史记录或旧版本数据的时间间隔（以秒为单位）。
默认值：900 (15分钟)

### **--update_min_cdc_indices_interval_secs**

设置多长时间读取一次cdc_state表，来获得所有流中每个分片的最小应用索引。此信息用于正确保存包含未应用条目的日志文件。这也是在所有流中的分片最小复制索引发送到配置中其它对等端的速率。如果禁用了标志enable_log_retention_by_op_idx（默认值：true），则此标志无效。
默认值：60

### **--cdc_checkpoint_opid_interval_ms**

客户端可以关闭的时间（毫秒为单位），并且将保留目标。这意味着，如果客户端没有为此间隔更新检查点，则将被做为垃圾回收。
默认值：60000

警告：
如果使用多个流，建议将此标志设置为1800000（30分钟）。 

### **--log_max_seconds_to_retain**

保留日志文件的秒数。早于此值的日志文件将被删除，即使它们包含未复制的CDC条目。如果为0，则将忽略此标志。如果日志段包含尚未刷新到RocksDB的条目，则会忽略此标志。
默认值：86400

### **--log_stop_retaining_min_disk_mb**

如果日志的可用空间低于此限制（以MB为单位），则停止保留日志。与log_max_seconds_to_retain一样，如果日志段包含未刷新的事项，则会忽略此标志。
默认值：102400

### **--enable_delete_truncate_cdcsdk_table**

默认情况下，CDCSDK流处于活动状态的表上的TRUNCATE命令将失败。将此标志的值从false更改为true将能够截断CDCSDK流的表部分。
默认值：false

### **--cdc_intent_retention_ms**

定义一个以毫秒为单位的时间段，在此时间段之后，如果没有针对更改记录的客户端轮询，则将清除事项。
默认值：14400000 (4 小时)

### **--enable_update_local_peer_min_index**

允许每个本地对等点更新其自己的日志检查点，而不是由leader更新所有对等点。 
默认值：false

 

## **基于TTL的文件过期标志**

### **--tablet_enable_ttl_file_filter**

打开TTL的文件过期功能。
默认值：false

### **--rocksdb_max_file_size_for_compaction**

对于具有default_time_to_live表属性的表，设置不再考虑压缩文件的大小阈值。超过此阈值的文件仍将被视为过期文件。如果值为0，则禁用。
理想情况下，rocksdb_max_file_size_for_compaction应该在以合理频率过期数据，和不创建太多SST文件（这可能会影响读取性能）之间取得平衡。例如，如果存储了90天的数据，请考虑将此标志设置为大约一天的数据大小。
默认值：0

### **--sst_files_soft_limit**

每个分片的SST文件数的阈值。超过此值时，将限制对分片的写入，直到文件数量减少为止。
默认值：24

### **--sst_files_hard_limit**

每个分片的SST文件数的阈值。如果超过，则在文件数量减少之前，将不再允许写入分片。
默认值：48

### **--file_expiration_ignore_value_ttl**

如果设置为true，则在确定文件过期时，忽略任何值级别的TTL元数据。在某些SST文件缺少必要的值级元数据的情况下很有用（例如，在升级的情况下）。
默认值：false
警告：使用此标志可能会导致实时数据过期。请自行决定使用。 

### **--file_expiration_value_ttl_overrides_table_ttl**

当设置为true时，允许文件完全根据其值级别的TTL过期时间而过期（即使它低于表TTL）。这对于文件需要在其表级TTL允许的时间之前过期的情况很有帮助。如果没有值级TTL元数据可用，则仍将使用表级TTL。

默认值：false
警告：使用此标志可能会导致实时数据过期。请自行决定使用。 

## **PostgreSQL服务器选项**

AiSQL使用PostgreSQL服务器配置参数，应用服务器配置设置到新的服务器实例上。

可以通过以下方式修改这些参数：

* 使用[bsql_pg_conf_csv](#_--bsql_pg_conf_csv)标志
* 对于每一个数据库进行设置：
  ALTER DATABASE database_name SET temp_file_limit=-1;
* 对于每一个角色进行设置：
  ALTER ROLE role_tester SET temp_file_limit=-1;
  注意：在角色或数据库级别设置参数时，必须打开一个新会话才能使更改生效。
* 对于当前会话进行设置：
  SET temp_file_limit=-1;
  或者
  SET SESSION temp_file_limit=-1;

注意：
如果SET发生在事务中，而此事务因为某些问题被中止后，则当事务被回滚时，SET命令产生的效果也会回到之前的效果。
如果事务提交，则影响将持续整个会话。

* 对于当前事务进行设置：
  Begin；
  ...
  SET LOCAL temp_file_limit=-1;

有关可用的PostgreSQL服务器配置参数的信息，请参阅PostgreSQL文档中的服务器配置部分。AiSQL的服务器配置参数与PostgreSQL的相同，新增或者不同的参数如下所示：

### **log_line_prefix**

AiSQL支持log_line_prefix参数的以下附加选项：
    %C = cloud名
    %R = 区域名或者数据中心名
    %Z = availability zone / rack name有效的地区或者机架名
    %U = 集群 UUID
    %N = 节点或者集群名
    %H = 当前主机名

有关使用log_line_prefix的信息，请参阅PostgreSQL文档中的log_line_prefix部分。

### **suppress_nonpg_logs (boolean)**

设置后，禁止将非PostgreSQL输出记录到dbserver/logs目录中的PostgreSQL日志文件中。
默认值：关闭

### **temp_file_limit**

指定用于每个BSQL连接的临时文件，例如排序和Hash临时文件等占用的磁盘空间大小。
当任何磁盘空间使用大小超过temp_file_limit的查询都将被终止，并显示错误，ERROR：temporary file size exceeds temp_file_limit。
请注意，临时表不计入此限制。
可以使用temp_file_limit=-1删除该限制（将大小设置为无限制）。
此参数的有效值为：-1（无限制）、integer（以KB为单位）、nMB（以MB为单位）和nGB（以GB为单位）（其中“n”是整数）。

默认值：1GB

## **示例**

```
./bm-dbserver \
--dbserver_mserver_addrs 10.0.0.1:11000,10.0.0.2:11000,10.0.0.3:11000 \
--rpc_bind_addresses 10.0.0.1 \
--enable_bsql \
--fs_data_dirs "/home/centos/disk1,/home/centos/disk2" &
```

## **管理用户界面**

bm-dbserver的管理用户界面，可访问http://localhost:20000（默认值）

### **主页**

bm-dbserver（bm-dbserver）的主页，提供了指定实例的高级概述信息。

![](../../assets/chapter9/41.png)

### **表**

集群中存在的表信息列表。

![](../../assets/chapter9/42.png)

### **分片（Tiles）**

列出集群中存在的所有分片信息

![](../../assets/chapter9/43.png)

### **工具**

列出集群的所有可用工具，用以调试性能。

![](../../assets/chapter9/44.png)