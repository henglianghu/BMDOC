使用bm-mserver二进制文件及其标志来配置mserver服务器。bm-mserver可执行文件位于AiSQL 安装目录的bin目录中。

## **基本语法**

基本语法：

```
./ bm-mserver [ flag ] | [ flag ]
```

## **通用标志**

### **--version**

显示版本和编译信息，然后退出。

### **--flagfile**

指定加载标志的配置文件

### **--mserver_addresses（必须项）**

指定mserver共识配置的所有RPC地址的逗号分隔列表（必须项）。

注意：
逗号分隔的值的数量应与mserver服务器的总数（或复制因子）相匹配。
默认值：127.0.0.1:11000

### **--fs_data_dirs（必须项）**

指定以逗号分隔的装载目录列表，bm-mserver将添加bm-data/mserver数据目录、mserver.err、mserver.out文件和pg_data目录。
注意：不支持在创建集群后再更改此标志的值。

### **--fs_wal_dirs**

指定以逗号分隔的目录列表，bm-mserver将在其中存储预写（WAL）日志。这可以与--fs_data_dirs中列出的某个目录相同，但不是数据目录的子目录。
默认值与 --fs_data_dirs 相同。

### **--rpc_bind_addresses**

指定要为RPC连接绑定到的网络接口地址的逗号分隔列表。
在所有bm-mserver和bm-dbserver配置中，使用的值必须匹配。
默认值：运行服务器的主机的专用IP地址。例如：
--rpc_bind_addresss=172.x.x.x:11000

如果以下情况适用，请确保server_broadcast_addresss标志设置正确：
1） rpc_bind_addresss设置为0.0.0.0
2） rpc_bind_addresss涉及公共IP地址，例如0.0.0.0:11000，它指示服务器侦听所有可用的网络接口。

### **--server_broadcast_addresses**

指定服务器的公共IP或DNS主机名（带有可选端口）。此值用于服务器相互通信，具体取决于连接策略参数。
默认值：""

### **--dns_cache_expiration_ms**

指定缓存的DNS解析到期之前的持续时间（以毫秒为单位）。当使用主机名而不是IP地址时，必须查询DNS解析程序以将主机名与IP地址相匹配。通过使用本地DNS缓存临时存储DNS查找，可以更快地解决DNS查询，并可以避免额外的查询。这减少了延迟，缩短了加载时间，并减少了带宽和CPU消耗。
默认值：60000 (1分钟)

注意：
如果更改此值，请确保将相同的值添加到所有mserver和TSever配置中。

### **--use_private_ip**

指定用于确定何时使用专用IP地址进行节点间通信的策略。可能的值为never、zone、cloud和region。基于[地理分布标志](#_地理分布标志)的值。

该策略的有效值为：
never：始终使用--server_broadcast_addresss。
zone：使用区域内的私有IP；在区域外使用--serverbroadcast_addresss。
Region：在一个区域的所有区域中使用私有IP地址；在区域外使用--server_broadcast_addresss。
默认值：never

### **--webserver_interface**

指定web服务器用户界面访问的绑定地址。 
默认值：0.0.0.0

### **--webserver_port**

指定web服务器监听的端口。
默认值：10000

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

## **BSQL标志**

### **--enable_bsql**

值为true，则可以使用BSQL API。

注意：确保mserver配置中的enable_bsql值与dbserver配置中的值相匹配。 
默认值： true

## **日志标志**

### **--colorlogtostderr**

记录到stderr的彩色信息（如果终端支持）。
默认值：false

### **--logbuflevel**

在此级别（或更低级别）记录的缓冲区日志消息。
有效值：-1（不缓冲）；0（信息）；1（警告）；2（错误）；3（致命）
默认值：0 

### **--logbufsecs**

最多缓冲日志消息几秒钟。 
默认值：30

### **--logtostderr**

将日志信息写入stderr，而不是写入日志文件。
默认值：false

### **--log_dir**

写mserver日志文件的目录。
默认值：与--fs_data_dirs相同

### **--log_link**

给这个目录中的日志文件添加额外的链接。
默认值：""

### **--log_prefix**

为每条日志行预先添加日志前缀。
默认值：true

### **--max_log_size**

最大的日志尺寸大小，单位为MB，值0将被静默重写为1。
默认值：1800 (1.8 GB)

### **--minloglevel**

记录消息的最低级别，有效值为：0（信息）；1（警告）；2（错误）；3（致命）。
默认值： 0（信息）

### **--stderrthreshold**

除日志文件外，此级别或更高级别的日志消息也会复制到stderr。
默认值： 2

## **Raft标志**

对于典型的部署，mserver配置中用于Raft和预写日志（WAL）标志的值应该与[bm-dbserver](#_bi-tserver)配置中的值匹配。

### **--follower_unavailable_considered_failed_sec**

持续时间，以秒为单位，在该持续时间之后，由于Leader没有接收到心跳，followers被认为失败。
默认值： 7200 (2 小时)
--follower_unavailable_consided_failed_sec值应与--log_min_seconds_to_retain的值匹配。

### **--evict_failed_followers**

失败的followers将被逐出Raft组，数据将被重新复制。
默认值： false

注意，不建议将mserver的标志设置为true，因为一旦故障mserver被逐出，就无法自动恢复。

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

## **预写日志（WAL）标志**

确保bm-mserver配置中用于预写日志（WAL）的值与bm-dbserver配置中的值匹配。 

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
默认值：7200 (2 小时)

注意：--log_min_seconds_to_retain 值应该与
--follower_unavailable_considered_failed_sec 值相匹配。

### **--log_min_segments_to_retain**

无论持久性要求如何，要保留的WAL段（文件）的最小数量。该值必须至少为1。 
默认值：2

### **--log_segment_size_mb**

WAL段（文件）的大小，以兆字节（MB）为单位。当WAL段达到指定的大小时，将发生日志翻转，并创建一个新的WAL段文件。
默认值：64

### **--reuse_unclosed_segment_threshold_bytes**

当服务器从上一次崩溃中重新启动时，如果分片的最后一个WAL文件大小小于或等于此阈值，则将重用最后的WAL文件。否则，WAL将在引导程序中分配一个新文件。要禁用WAL重用，请将该值设置为-1。

默认值：2.18.1中的默认值为-1，默认情况下禁用该功能；从2.19.1开始的默认值为524288（0.5 MB），默认情况下启用该功能。

## **负载均衡标志**

对于bm-admin负载均衡命令，参考[负载均衡命令](#_负载均衡命令)。

### **--enable_load_balancing**

启用或者禁用负载均衡。
默认值：true

### **--leader_balance_threshold**

指定要均衡的每个分片服务器的leaders数量。如果将其配置为0（默认值），则leaders将以额外成本实现最佳均衡。
默认值：0

### **--leader_balance_unresponsive_timeout_ms**

指定bm-mserver在认为其没有响应之前，可以在没有从bm-dbserver接收到检测信号的情况下运行的时间段（以毫秒为单位）。未响应的服务器将被排除在leader 均衡之外。 
默认值：3000 (3 秒)

### **--load_balancer_max_concurrent_adds**

指定要添加到负载均衡操作中的分片对等副本的最大数量。 
默认值：1

### **--load_balancer_max_concurrent_moves**

指定在负载均衡操作中要移动的分片服务器（整个集群）上的分片leaders的最大数量。
默认值：2

### **--load_balancer_max_concurrent_moves_per_table**

指定在负载均衡的任何一次运行中，每个表要移动的分片leaders的最大数量。分片leader在集群中移动的最大次数仍然受到标志load_balancer_max_concurrent_moves的限制。此标志旨在防止单个表使用所有的leader移动名额，而其他表无名额。
默认值：1

### **--load_balancer_max_concurrent_removals**

指定在负载均衡操作中要执行的过度复制的分片对等删除的最大数量。
默认值：1

### **--load_balancer_max_concurrent_tablet_remote_bootstraps**

指定在集群中远程启动的分片的最大数量。 
默认值：10

### **--load_balancer_max_concurrent_tablet_remote_bootstraps_per_table**

任何表远程启动的分片的最大数量。集群中远程引导的最大数量仍然受到标志load_balancer_max_concurrent_tablet_remote_bootstraps的限制。此标志旨在防止单个表使用所有可用的远程引导会话，而其他表无可用的远程引导会话。 
默认值：2

### **--load_balancer_max_over_replicated_tablets**

指定允许超过配置的复制因子的最大运行分片副本数。 
默认值：1

### **--load_balancer_num_idle_runs**

指定负载均衡的空闲运行次数以将其视为空闲。
默认值：5

### **--load_balancer_skip_leader_as_remove_victim**

如果LB跳过一个leader，则作为可能的罢免候选人。 
默认值：false

## **分片标志**

### **--max_clock_skew_usec**

部署任意两个服务器之间的预期最大时钟偏移，单位为微秒（µs）。
默认值：500000 (500,000 µs = 500ms)

### **--replication_factor**

每个分片要存储的副本或数据副本的数量，副本因子。
默认值：3

### **--bi_num_shards_per_dbserver**

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

## **分片拆分标志**

### **--enable_automatic_tablet_splitting**

使AiSQL能够基于配置的指定分片阈值大小，自动分片。
默认值：true

注意：此值必须在AiSQL集群的所有bm-mserver和bm-dbserver配置上匹配。

### **--tablet_split_low_phase_shard_count_per_node**

表中分片的阈值数量（每个集群节点），低于该阈值，自动分片拆分将使用--tablet_split_low_phase_size_threshold_bytes来确定要拆分的分片。
默认值：8

### **--tablet_split_low_phase_size_threshold_bytes**

当分片表处于自动拆分分片的低阶段时，尺寸大小阈值用于决定是否应拆分分片，请参见--tablet_split_low_phase_shard_count_per_node。 
默认值：512_MB

### **--tablet_split_high_phase_shard_count_per_node**

表中分片的阈值数量（每个集群节点），低于该阈值，自动分片拆分将使用
--tablet_split_high_phase_size_threshold_bytes来确定要拆分的分片。
默认值：24

### **--tablet_split_high_phase_size_threshold_bytes**

当分片表处于自动拆分分片的高阶段时，尺寸大小阈值用于决定是否应拆分分片，请参见--tablet_split_high_phase_shard_count_per_node。
默认值：10_GB

### **--tablet_force_split_threshold_bytes**

用于确定是否应该拆分分片的大小阈值，即使表的分片数量使其超过了高阶段。
默认值：100_GB

### **--tablet_split_limit_per_table**

用于拆分分片的每个表的最大分片数。如果此值设置为0，则禁用限制。
默认值：256

### **--index_backfill_tablet_split_completion_timeout_sec**

在中止回填并将其标记为失败之前，在运行回填的表上，等待分片拆分完成的总时间。
默认值：30

### **--index_backfill_tablet_split_completion_poll_freq_ms**

在重试查看正运行回填数据的表的分片拆分是否已完成之前的延迟时间。
默认值：2000

### **--process_split_tablet_candidates_interval_msec**

自动拆分尝试之间的最短时间。运行之间的实际分割时间也受到catalog_manager_bg_task_wait_ms的影响，它控制后台任务线程在每个循环结束时睡眠的时间。
默认值：0

### **--outstanding_tablet_split_limit**

限制未完成的分片拆分总数。如果值设置为0，则禁用限制。限制包括进行拆分后正在压缩的分片。
默认值：1

### **--outstanding_tablet_split_limit_per_dbserver**

限制每一个节点上未完成的分片拆分总数。如果值设置为0，则禁用限制。限制包括进行拆分后正在压缩的分片。
默认值：1

### **--enable_tablet_split_of_pitr_tables**

启用表的自动分片拆分被以时间点恢复计划所覆盖。
默认值：true

### **--enable_tablet_split_of_xDCR_replicated_tables**

为作为xDCR复制设置一部分的表启用自动分片拆分。
默认值：false

要在跨集群复制的表上启用分片拆分，无论是生产者集群，还是消费者集群上，此标志都应设置为true，因为它们将独立执行拆分。生产者集群和消费者集群都必须运行在v2.14.0+才能启用该功能（与集群升级相关）。

### **--prevent_split_for_ttl_tables_for_seconds**

检查是否使用默认TTL拆分分片的秒数。如果此值设置为0，则禁用检查。
默认值：86400

### **--prevent_split_for_small_key_range_tablets_for_seconds**

检查是否拆分键范围太小而无法拆分的分片的秒数。如果此值设置为0，则禁用检查。
默认值：300

### **--sort_automatic_tablet_splitting_candidates**

确定是否按从大到小的顺序对自动拆分候选进行排序（按拆分的较大分片的优先级排列）。
有关自动拆分分片的详细信息，请参阅以下内容：[自动拆分分片](#_自动拆分分片)

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

### **--use_private_ip**

确定何时使用私有IP地址。可能的值为never（默认值）、zone、cloud和region。基于placement_*配置标志的值。
默认值：never

### **--auto_create_local_transaction_tables**

如果为true，则将为任何BSQL表空间自动创建事务表，该表空间中有一个位置和至少一个其他表。
默认值：true

## **安全标志**

有关启用服务器到服务器加密的详细信息，请参见服务器到服务器加密

### **--certs_dir**

包含此服务器的证书颁发机构、私钥和证书的目录。
默认值："" (<data drive>/bm-data/mserver/data/certs.)

### **--allow_insecure_connections**

允许不安全的连接。设置为false可防止任何通信未加密的进程加入群集。请注意，此标志要求启用use_node_to_node_encryption。
默认值：true

### **--dump_certificate_entries**

添加证书条目，包括IP地址和主机名，记录握手错误消息。启用此标志有助于调试证书问题。
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

## **更改数据捕获（CDC）标志**

对于其它的CDC配置标志，请参见bm-dbserver的[更改数据捕获（CDC）标志](#_更改数据捕获（CDC）标志_1)

### **--cdc_state_table_num_tablets**

创建CDC状态表时要使用的分片数量。用于xDCR和CDCSDK。 
默认值：0 (使用与普通表相同的默认分片数量)

### **--cdc_wal_retention_time_secs**

用于创建CDC流的表的WAL保留时间（以秒为单位）。用于xDCR和CDCSDK。
默认值：14400 (4 小时)

```
## **示例**
./bm-mserver \
--mserver_addresses 10.0.0.1:11000,10.0.0.2:11000,10.0.0.3:11000 \
--rpc_bind_addresses 10.0.0.1\
--fs_data_dirs "/home/centos/disk1,/home/centos/disk2" \
--replication_factor=3
```

## **管理用户界面**

bm-mserver的管理用户界面，可访问http://localhost:10000（默认值）

### **主页**

mserver服务器的主页，提供了集群的高级概述信息。并非集群中的所有mserver服务器都显示相同的信息。

![](media/chapter9/37.png)

### **表**

集群中存在的表信息列表。

![](media/chapter9/38.png)

### **分片服务器**

列出集群中存在的所有dbserver节点

![](media/chapter9/39.png)

### **工具**

列出集群的所有可用工具，用以调试性能。

![](media/chapter9/40.png)