dbserver-ctl 是一个命令行工具，可用于在特定的 Tile 服务器 (dbserver) 上执行操作。 某些命令执行的操作与 bm-admin 命令类似。 bm-admin 命令专注于集群管理，dbserver-ctl 命令适用于特定的 DBServer 节点。

dbserver-ctl 是随 AiSQL 安装的二进制文件，位于 AiSQL 主目录的 bin 目录中。

## **语法**

```
dbserver-ctl [ --server_address=<host>:<port> ] <command> <flags>
```

host:port：tile 服务器的主机和端口。 默认值为localhost:21000。
command：执行的命令。 
flags：应用于命令的标志。

**在线帮助**
要显示可用的在线帮助，请在 AiSQL 主目录中运行 dbserver-ctl（不带任何命令或标志）。

```
./bin/dbserver-ctl
```

## **命令**

可以使用以下命令：

* are_tiles_running
* is_server_ready
* compact_all_tiles
* compact_tile
* count_intents
* current_hybrid_time
* delete_tile
* dump_tile
* flush_all_tiles
* flush_tile
* list_tiles
* reload_certificates
* remote_bootstrap
* set_flag
* status
* refresh_flags

### **are_tablets_running**

如果所有Tile都在运行，则返回“All tiles are running”。

语法：

```
dbserver-ctl [ --server_address=<host>:<port> ] are_tiles_running
```

host:port：tiles服务器的主机和端口。 默认值为localhost:21000。

### **is_server_ready**

打印尚未引导的Tile数量。 如果所有Tile均已引导，则返回“Tile server is ready”。

语法：

```
dbserver-ctl [ --server_address=<host>:<port> ] is_server_ready
```

host:port：tile 服务器的主机和端口。 默认值为localhost:21000。

### **compact_all_tiles**

压缩Tile服务器上的所有Tile。
语法：

```
dbserver-ctl [ --server_address=<host>:<port> ] compact_all_tiles
```

host:port：tile 服务器的主机和端口。 默认值为localhost:21000。

### **compact_tile**

在Tile服务器上压缩指定的Tile。
语法：

```
dbserver-ctl [ --server_address=<host>:<port> ] compact_tile <tile_id>
```

host:port：tile 服务器的主机和端口。 默认值为localhost:21000。
tile_id：要压缩的tile 的标识符。

### **count_intents**

打印未提交意图（或临时记录）的计数。 对于调试事务工作负载很有用。
语法：

```
dbserver-ctl  [ --server_address=<host>:<port> ] count_intents
```

host:port：tile 服务器的主机和端口。 默认值为localhost:21000。

### **current_hybrid_time**

打印当前混合时间的值。
语法：

```
dbserver-ctl  [ --server_address=<host>:<port> ] current_hybrid_time
```

host:port：tile 服务器的主机和端口。 默认值为localhost:21000。

### **delete_tile**

删除具有指定Tile ID (tile_id) 和原因的Tile。
语法：

```
dbserver-ctl  [ --server_address=<host>:<port> ] delete_tile <tile_id> "<reason-string>"
```

host:port：tile 服务器的主机和端口。 默认值为localhost:21000。
tile_id：Tile的标识符 (ID)。
Reason-string：文本字符串，提供有关Tile被删除原因的有用信息。

### **dump_tile**

转储或导出指定的Tile ID (tile_id)。
语法：

```
dbserver-ctl [ --server_address=<host>:<port> ] dump_tile <tile_id>
```

host:port：tile 服务器的主机和端口。 默认值为localhost:21000。
tablet_id：Tile的标识符 (ID)。

### **flush_all_tiles**

刷新Tile服务器上的所有Tile。
语法：

```
dbserver-ctl [ --server_address=<host>:<port> ] flush_all_tiles
```

host:port：tile 服务器的主机和端口。 默认值为localhost:21000。

### **flush_tile**

刷新Tile服务器上指定的Tile。
语法：

```
dbserver-ctl [ --server_address=<host>:<port> ] flush_tile <tile_id>
```

host:port：tile 服务器的主机和端口。 默认值为localhost:21000。
tile_id：要刷新的tile 的标识符。

### **list_tiles**

列出指定tile 服务器上的tile，显示以下属性：列名称、tileID、状态、表名称、分片和架构。
语法：

```
dbserver-ctl [ --server_address=<host>:<port> ] list_tiles
```

host:port：tile 服务器的主机和端口。 默认值为localhost:21000。

### **reload_certificates**

触发从指定（mserver 或Tile）服务器上的磁盘重新加载 TLS 证书和私钥。
语法：

```
dbserver-ctl [ --server_address=<host>:<port> ] reload_certificates
```

host:port：tile 服务器的主机和端口。 默认值为localhost:21000。

### **remote_bootstrap**

触发从另一个tile 服务器到指定tile服务器的tile远程引导。
语法：

```
dbserver-ctl [ --server_address=<host>:<port> ] remote_bootstrap <source_host> <tile_id>
```

host:port：tile 服务器的主机和端口。 默认值为localhost:21000。
source_host：要引导的Tile服务器的主机或主机和端口。
tile_id：要触发远程引导的Tile的标识符。

### **set_flag**

为tile 服务器设置指定的配置标志。
语法：

```
dbserver-ctl [ --server_address=<host>:<port> ] set_flag [ --force ] <flag> <value>
```

host:port：tile 服务器的主机和端口。 默认值为localhost:21000。
--force：允许更改未显式标记为运行时可设置的标志的标志。 请注意，如果提供了不安全的值，则更改可能会在服务器上被忽略，或者可能导致服务器崩溃。
flag：要设置的 dbserver 配置标志（不带 -- 前缀）。 请参阅 dbserver
value：要应用的值。

set_flag 命令以原子方式更改正在运行的服务器的指定标志的内存值，并可以改变其行为。 更改不会在重新启动后持续存在。

实际上，有些标志可以在运行时安全地更改（运行时可设置），而有些则不然。 例如，服务器的绑定地址无法在运行时更改，因为服务器在启动时仅绑定一次。 虽然大多数标志可能是运行时可设置的，但您需要检查这些标志并在配置页中记下哪些标志不可运行时设置。
一种典型的操作流程是，您可以使用它来修改内存中的运行时标志，然后在带外也修改服务器用于启动的配置文件。 这允许在运行的服务器上更改标志，而无需重新启动服务器。

### **Status**

打印tile 服务器的状态，包括节点实例信息、绑定的RPC地址、绑定的HTTP地址和版本信息。
语法：

```
dbserver-ctl [ --server_address=<host>:<port> ] status
```

host:port：tile 服务器的主机和端口。 默认值为localhost:21000。

### **refresh_flags**

刷新从配置文件加载的标志。 适用于 DBServer （端口 21000）和 MServer（端口 11000）进程。 无需参数。

每个进程都需要发出以下命令，例如，在一个 DBServer 上发出该命令不会更新其他 DBServer 上的标志。
语法：

```
dbserver-ctl [ --server_address=<host>:<port> ] refresh_flags
```

host:port：tile 服务器的主机和端口。 默认值为localhost:21000。

## **标志**

如果指定，可以将以下标志与上述命令一起使用。
--force 
将此标志与 set_flag 命令结合使用可允许更改未显式标记为运行时可设置的标志。 请注意，如果提供了不安全的值，则更改可能会在服务器上被忽略，或者可能导致服务器崩溃。
默认值：false

--server-address 
要运行的 tile 服务器的地址（主机和端口）。
默认值：localhost:21000

--timeout_ms
RPC 请求超时之前的持续时间，以毫秒 (ms) 为单位
默认值：60000 (1000 ms = 1 sec)

--certs_dir_name
要连接到启用了 TLS 的集群，您必须包含 --certs_dir_name 标志以及根证书所在的目录位置。
默认值：””

## **示例**

返回tile 服务器的状态

```
./bin/dbserver-ctl --server_address=127.0.0.1 --certs_dir_name="/path/to/dir/name" status
node_instance {
  permanent_uuid: "237678d61086489991080bdfc68a28db"
  instance_seqno: 1579278624770505
}
bound_rpc_addresses {
  host: "127.0.0.1"
  port: 21000
}
bound_http_addresses {
  host: "127.0.0.1"
  port: 20000
}
version_info {
  git_hash: "83610e77c7659c7587bc0c8aea76db47ff8e2df1"
  build_hostname: "bm-macmini-6.dev.bigmath.com"
  build_timestamp: "06 Jan 2020 17:47:22 PST"
  build_username: "jenkins"
  build_clean_repo: true
  build_id: "743"
  build_type: "RELEASE"
  version_number: "2.0.10.0"
  build_number: "4"
}
```

显示当前混合时间

```
./bin/dbserver-ctl  --server_address=dbserver-1:21000 current_hybrid_time
6470519323472437248
```
