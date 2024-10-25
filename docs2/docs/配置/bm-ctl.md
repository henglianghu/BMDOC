AiSQL使用双服务器架构，DBServers管理数据，MServers管理元数据。 然而，这可能会给想要立即开始的新用户带来负担。 要管理 AiSQL，您可以使用 bm-ctl。 bm-ctl 充当 DBServer 和 MServers 服务器之间的父服务器。 bm-ctl 还提供了类似于 AiSQL Anywhere UI 的 UI，带有数据放置图和指标仪表板。

bm-ctl 可执行文件位于 AiSQL 主目录的 bin 目录中。

## **语法**

```
bm-ctl [-h] [ <command> ] [ <flags> ]
```

* 命令：运行的命令
* flags：一个或多个标志，以空格分隔。

**示例**

```
$ ./bin/bm-ctl start
```

**在线帮助**
您可以通过从 AiSQL 主目录运行以下示例之一来访问 bm-ctl 的命令行帮助：

```
$ ./bin/bm-ctl -h
$ ./bin/bm-ctl -help
```

如需特定 bm-ctl 命令的帮助，请运行“bm-ctl [ command ] -h”。 例如，您可以通过运行以下命令来打印 bm-ctl start 命令的命令行帮助：

```
$ ./bin/bm-ctl start -h
```

## **命令**

可以使用以下命令：

* start
* configure
* cert
* stop
* destroy
* status
* version
* collect_logs
* connect
* demo

### **start**

使用 bm-ctl start 命令启动单节点 AiSQL 集群，以便在本地环境中运行 BSQL 和 BCQL 工作负载。

请注意，要使用传输中加密，必须在节点上安装 OpenSSL。

**1.语法**

```
bm-ctl start [flags]
```

示例：
创建本地单节点集群：

```
./bin/bm-ctl start
```

创建具有传输加密和身份验证的本地单节点集群：

```
./bin/bm-ctl start --secure
```

在本地创建一个单节点并加入属于同一集群的其他节点：

```
./bin/bm-ctl start --join=host:port,[host:port]
```

**2.标志**
-h | --help
打印命令行帮助并退出。

--advertise_address bind-ip
bm-ctl 将侦听的 IP 地址或本地主机名。

--join mserver-ip
新 bm-ctl 服务器将加入的现有 bm-ctl 服务器的 IP 地址，或者如果服务器重新启动，则重新加入。

--config config-file
bm-ctl 配置文件路径。 请参阅高级标志。

--base_dir base-directory
bm-ctl 存储数据、配置和日志的目录。 必须是绝对路径。

--data_dir data-directory
bm-ctl 存储数据的目录。 必须是绝对路径。 可以配置到与存储配置和日志的目录不同的目录。

--log_dir log-directory
存储 bm-ctl 日志的目录。 必须是绝对路径。 该标志控制 AiSQL 节点的日志存储位置。 默认情况下，日志写入~/var/logs。

--background bool
启用或禁用在后台作为守护进程运行 bm-ctl。 重新启动后不会持续存在。 默认值：true

--cloud_location cloud-location
bm-ctl 节点的云位置，格式为 cloudprovider.region.zone。 此信息用于 AiSQL 集群的多专区、多区域和多云部署。

--fault_tolerance fault_tolerance
确定应用于 AiSQL 集群的数据放置策略的容错约束。 该标志可以接受以下值：none、zone、region、cloud。

--ui bool
启用或禁用网络服务器 UI（可从 http://localhost:12521 获取）。 默认值：true

 --secure
启用节点的传输加密和身份验证。
传输中加密需要集群中每个节点的 SSL/TLS 证书。

* 启动本地单节点集群时，会自动为集群生成证书。
* 在多节点集群中部署节点时，需要使用 --certgenerate_server_certs 命令为节点生成证书，并在使用 --secure 标志启动节点之前将其复制到节点，否则节点创建将失败 失败。

启用身份验证后，BSQL 中默认用户为 bigmath，BCQL 中默认用户为 cassandra。 当集群启动时，bm-ctl 输出一条消息 Credentials File is storage at <credentials_file_path.txt> 以及凭证文件位置。
创建安全的本地多节点、多可用区、多区域集群的示例，请参阅示例。

**3.高级标志**
可以使用 --config 标志中的配置文件来设置高级标志。 启动命令的高级标志支持如下：

--bcql_port bcql-port
BCQL 将运行的端口。

--bsql_port bsql-port
BSQL 将运行的端口。

---mserver_rpc_port mserver-rpc-port
MServer 将侦听 RPC 调用的端口。

--dbserver_rpc_port
DBServer 将侦听 RPC 调用的端口。

--mserver_webserver_port
MServer Web 服务器将运行的端口。

--dbserver_webserver_port
DBServer Web 服务器将运行的端口。

--webserver_port
主web服务器将运行的端口。

--callhome bool
启用或禁用将分析数据发送到 bigmath 的回拨功能。 默认值：true。

--mserver_flags
将额外的mserver标志指定为一组键值对。 格式（键=值，键=值）。

--dbserver_flags
将额外的 dbserver 标志指定为一组键值对。 格式（键=值，键=值）。

--sql_enable_auth bool
启用或禁用 BSQL 身份验证。 默认值：假。
如果 BSQL_PASSWORD 环境变量存在，则身份验证模式将自动设置为 true。

--use_cassandra_authentication bool
启用或禁用 BCQL 身份验证。 默认值：假。
如果 BCQL_USER 或 BCQL_PASSWORD 环境变量存在，则身份验证模式将自动设置为 true。
请注意，相应的环境变量的优先级高于命令行标志。

--initial_scripts_dir initial-scripts-dir
bm-ctl 读取初始化脚本的目录。
脚本格式 - BSQL .sql、BCQL .cql。
初始化脚本按照排序的名称顺序执行。

**4.已弃用的标志**
--daemon bool
启用或禁用在后台作为守护进程运行 bm-ctl。 重新启动后不会持续存在。 默认值：true。

--listen bind-ip
bm-ctl 将侦听的 IP 地址或本地主机名称。


### **configure**

使用 bm-ctl configure 命令执行以下操作：

* 配置集群的数据放置策略。
* 启用或禁用静态加密。

**1.语法**

```
bm-ctl configure [command] [flags]
```

**2.命令**
以下子命令可用于 bm-ctl configure命令：

* data_placement
* encrypt_at_rest

（1）data_placement
使用 bm-ctl configure data_placement 子命令设置或修改已部署集群的节点的放置策略。

例如，您可以使用以下命令创建多区域 AiSQL 集群：

```
./bin/bm-ctl configure data_placement --fault_tolerance=zone
```

（2）data_placement 标志
-h | --help
打印命令行帮助并退出。

--fault_tolerance fault-tolerance
指定集群的容错能力。 该标志可以接受以下值之一：可用区、区域、云。 例如，当该标志设置为 zone (--fault_tolerance=zone) 时，bm-ctl 会将区域容错应用于集群，将节点放置在三个不同的区域中（如果可用）。

--constraint_value data-placement-constraint-value
指定 AiSQL 集群的数据放置。 这是一个可选标志。 该标志采用格式为 cloud.region.zone 的逗号分隔值。

--rf replication-factor
指定集群的复制因子。 这是一个可选标志，其值为 3 或 5。

--config config-file
bm-ctl 服务器的配置文件路径。

--data_dir data-directory
bm-ctl 服务器的数据目录。

--base_dir base-directory
bm-ctl 服务器的基本目录。

--log_dir log-directory
bm-ctl 服务器的日志目录。
**（3）encrypt_at_rest** 
使用 bm-ctl configure encrypt_at_rest 子命令为已部署的集群启用或禁用静态加密。

要使用静态加密，必须在节点上安装 OpenSSL。

例如，要为已部署的 AiSQL 集群启用静态加密，请执行以下命令：

```
./bin/bm-ctl configure encrypt_at_rest --enable
```

要为已启用静态加密的 AiSQL 集群禁用静态加密，请执行以下命令：

```
./bin/bm-ctl configure encrypt_at_rest --disable
```

（4）encrypt_at_rest标志
-h | --help
打印命令行帮助并退出。

--disable disable
禁用集群的静态加密。 无需为该标志设置值。 使用 --enable 或 --disable 标志来切换 AiSQL 集群上的加密功能。

--enable enable
为集群启用静态加密。 无需为该标志设置值。 使用 --enable 或 --disable 标志来切换 AiSQL 集群上的加密功能。

--config config-file
bm-ctl 服务器的配置文件路径。

--data_dir data-directory
bm-ctl 服务器的数据目录。

--base_dir base-directory
bm-ctl 服务器的基本目录。

--log_dir log-directory
bm-ctl 服务器的日志目录。

### **cert**

使用 bm-ctl cert 命令创建 TLS/SSL 证书以部署安全的 AiSQL 集群。

**1.语法**

```
bm-ctl cert [command] [flags]
```

**2.命令**
以下子命令可用于 bm-ctl cert 命令：
generate_server_certs

（1）generate_server_certs
使用 bm-ctl cert generate_server_certs 子命令为指定主机名生成密钥和证书。

例如，要为主机名 127.0.0.1、127.0.0.2、127.0.0.3 创建节点服务器证书，请执行以下命令：

```
./bin/bm-ctl cert generate_server_certs --hostnames=127.0.0.1,127.0.0.2,127.0.0.3
```

**3.标志**
-h | --help
打印命令行帮助并退出。

--hostnames hostnames
要添加到集群中的节点的主机名。 强制标志。

--config config-file
bm-ctl 服务器的配置文件路径。

--data_dir data-directory
bm-ctl 服务器的数据目录。

--base_dir base-directory
bm-ctl 服务器的基本目录。

--log_dir log-directory
bm-ctl 服务器的日志目录。

### **stop**

使用 bm-ctl stop 命令停止 AiSQL 集群。

**1.语法**

```
bm-ctl stop [flags]
```

**2.标志**
--h | --help
打印命令行帮助并退出。

--config config-file
需要停止的bm-ctl服务器的配置文件路径。

--data_dir data-directory
需要停止的 bm-ctl 服务器的数据目录。

--base_dir base-directory
需要停止的 bm-ctl 服务器的基目录。

--log_dir log-directory
需要停止的 bm-ctl 服务器的日志目录。

### **destroy**

使用 bm-ctl destroy 命令删除集群。

**1.语法**

```
bm-ctl destroy [flags]
```

**2.标志**
--h | --help
打印命令行帮助并退出。

--config config-file
需要停止的bm-ctl服务器的配置文件路径。

--data_dir data-directory
需要停止的 bm-ctl 服务器的数据目录。

--base_dir base-directory
需要停止的 bm-ctl 服务器的基目录。

--log_dir log-directory
需要停止的 bm-ctl 服务器的日志目录。

### **status**

使用 bm-ctl status 命令检查状态。

**1.语法**

```
bm-ctl status [flags]
```

**2.标志**
--h | --help
打印命令行帮助并退出。

--config config-file
需要停止的bm-ctl服务器的配置文件路径。

--data_dir data-directory
需要停止的 bm-ctl 服务器的数据目录。

--base_dir base-directory
需要停止的 bm-ctl 服务器的基目录。

--log_dir log-directory
需要停止的 bm-ctl 服务器的日志目录。

### **version**

使用 bm-ctl version 命令查看版本号。

**1.语法**

```
bm-ctl version [flags]
```

**2.标志**
--h | --help
打印命令行帮助并退出。

--config config-file
需要停止的bm-ctl服务器的配置文件路径。

--data_dir data-directory
需要停止的 bm-ctl 服务器的数据目录。

--base_dir base-directory
需要停止的 bm-ctl 服务器的基目录。

--log_dir log-directory
需要停止的 bm-ctl 服务器的日志目录。

### **collect_logs**

使用 bm-ctl collect_logs 命令生成包含所有日志的压缩文件。

**1.语法**

```
bm-ctl collect_logs [flags]
```

**2.标志**
--h | --help
打印命令行帮助并退出。

--stdout stdout
将logs.tar.gz 文件的内容重定向到stdout。 例如， docker exec \<container-id\> bin/bm-ctlcollect_logs --stdout > bm-ctl.tar.gz

--config config-file
需要停止的bm-ctl服务器的配置文件路径。

--data_dir data-directory
需要停止的 bm-ctl 服务器的数据目录。

--base_dir base-directory
需要停止的 bm-ctl 服务器的基目录。

--log_dir log-directory
需要停止的 bm-ctl 服务器的日志目录。


### **connect**

使用 bm-ctl connect 命令通过 sqlsh 或 cqlsh 连接到集群。

**1.语法**

```
bm-ctl connect [command] [flags]
```

**2.命令**
以下子命令可用于 bm-ctl connect 命令：

* bsql
* bcql

（1）bsql
使用 bm-ctl connect bsql 子命令通过 sqlsh 连接到 AiSQL。

（2）bcql
使用 bm-ctl connect bcql 子命令通过 cqlsh 连接到 AiSQL。

**2.标志**
--h | --help
打印命令行帮助并退出。

--config config-file
要连接的 bm-ctl 服务器的配置文件的路径。

--data_dir data-directory
bm-ctl 服务器要连接的数据目录。

--base_dir base-directory
bm-ctl 服务器连接的基目录。

--log_dir log-directory
bm-ctl 服务器要连接的日志目录


### **demo**

使用 bm-ctl demo 命令将演示 Northwind 示例数据集与 AiSQL 结合使用。

**1.语法**

```
bm-ctl demo [command] [flags]
```

**2.命令**
以下子命令可用于 bm-ctl demo 命令：

* connect
* destroy

（1）connect
使用 bm-ctl demo connect 子命令将 Northwind 示例数据集加载到新的 bm_demo_northwind SQL 数据库中，然后打开同一数据库的 sqlsh 提示符。

（2）destroy
使用 yuagbyted demo destroy 子命令关闭 bm-ctl 单节点集群并删除数据、配置和日志目录。 此子命令还会删除 bm_demo_northwind 数据库。

**3.标志**
-h | --help
打印帮助消息并退出。

--config config-file
要连接或销毁的 bm-ctl 服务器的配置文件的路径。

--data_dir data-directory
bm-ctl 服务器连接或销毁的数据目录。

--base_dir base-directory
bm-ctl 服务器连接或销毁的基本目录。

--log_dir log-directory
bm-ctl 服务器连接或销毁的日志目录。

## **环境变量**

在多节点部署的情况下，所有节点应该具有相似的环境变量。

第一次运行后更改环境变量的值不会产生任何影响。

### **BSQL**

设置 BSQL_PASSWORD 以在强制身份验证模式下使用集群。

以下是环境变量的组合及其用途：

* BSQL_PASSWORD
  更新默认的 bigmath 用户密码。

* BSQL_PASSWORD, BSQL_DB
  更新默认的bigmath用户密码并创建名为DB的BSQL_DB。

* BSQL_PASSWORD, BSQL_USER
  创建名为 BSQL_USER 的用户和密码为 BSQL_PASSWORD 的数据库。

* BSQL_USER
  创建名为 BSQL_USER 的用户和密码为 BSQL_USER 的数据库。

* BSQL_USER, BSQL_DB

创建名为BSQL_USER、密码为BSQL_USER 的命名用户和名为DB 的BSQL_DB。

* BSQL_DB
  创建BSQL_DB，命名为DB。

* BSQL_USER, BSQL_PASSWORD, BSQL_DB
  创建名为 BSQL_USER、密码为 BSQL_PASSWORD 的用户和名为 DB 的 BSQL_DB。

### **BCQL**

设置 BCQL_USER 或 BCQL_PASSWORD 以在强制身份验证模式下使用集群。

以下是环境变量的组合及其用途：

* BCQL_PASSWORD
  更新默认 cassandra 用户的密码。

* BCQL_PASSWORD、BCQL_KEYSPACE
  更新默认 cassandra 用户的密码并创建名为 keyspace 的 BCQL_KEYSPACE。

* BCQL_PASSWORD, BCQL_USER
  创建 BCQL_USER 命名用户和数据库，密码为 BCQL_PASSWORD。

* BCQL_USER
  创建 BCQL_USER 命名用户和密码 BCQL_USER 的数据库。

* BCQL_USER、BCQL_KEYSPACE
  使用密码 BCQL_USER 和 BCQL_USER 命名密钥空间创建 BCQL_USER 命名用户。

* BCQL_KEYSPACE
  创建名为 keyspace 的 BCQL_KEYSPACE。

* BCQL_USER, BCQL_PASSWORD, BCQL_KEYSPACE
  使用密码 BCQL_PASSWORD 和名为 keyspace 的 BCQL_KEYSPACE 创建 BCQL_USER 命名用户。

## **示例**

要部署任何类型的安全集群（即使用 --secure 标志）或使用静态加密，必须在您的计算机上安装 OpenSSL。

### **在 macOS 上运行**

**1.端口冲突**
macOS Monterey 默认启用 AirPlay 接收，监听端口 10000。这与 AiSQL 冲突，导致 bm-ctl 启动失败。 启动集群时使用 --mserver_webserver_port 标志来更改默认端口号，如下所示：

```
./bin/bm-ctl start --mserver_webserver_port=9999
```

或者，您可以禁用 AirPlay 接收，然后正常启动 AiSQL，然后（可选）重新启用 AirPlay 接收。

**2.环回地址**
在 macOS 上，第一个节点之后的每个附加节点都需要配置一个环回地址来模拟多个主机或节点的使用。 例如，对于三节点集群，您可以添加两个附加地址，如下所示：

```
sudo ifconfig lo0 alias 127.0.0.2
sudo ifconfig lo0 alias 127.0.0.3
```

重新启动计算机后，环回地址不会保留。

### **销毁本地集群**

如果您在本地计算机上运行 AiSQL，则一次无法运行多个集群。 要使用 bm-ctl 设置新的本地 AiSQL 集群，请首先销毁当前正在运行的集群。

要销毁本地单节点集群，请使用 destroy 命令，如下所示：

```
./bin/bm-ctl destroy
```

要销毁本地多节点集群，请使用 destroy 命令，并将 --base_dir 标志设置为每个节点的基目录路径。 例如，对于三节点集群，您将执行类似于以下的命令：

```
./bin/bm-ctl destroy --base_dir=/tmp/bmd1
./bin/bm-ctl destroy --base_dir=/tmp/bmd2
./bin/bm-ctl destroy --base_dir=/tmp/bmd3
./bin/bm-ctl destroy --base_dir=$HOME/bigmath-2.20.1.2/node1
./bin/bm-ctl destroy --base_dir=$HOME/bigmath-2.20.1.2/node2
./bin/bm-ctl destroy --base_dir=$HOME/bigmath-2.20.1.2/node3
```

如果集群具有超过三个节点，请对每个附加节点执行destroy --base_dir=<path to directory> 命令，直到销毁所有节点。

### **创建单节点集群**

使用给定的基目录创建单节点集群。 请注意，需要为 base_dir 参数提供完全限定的目录路径。

```
./bin/bm-ctl start --advertise_address=127.0.0.1 \
    --base_dir=/Users/username/bigmath-2.20.1.2/data1
```

要创建启用传输加密和身份验证的安全单节点集群，请添加 --secure 标志，如下所示：

```
./bin/bm-ctl start --secure --advertise_address=127.0.0.1 \
    --base_dir=/Users/username/bigmath-2.20.1.2/data1
```

启用身份验证后，默认用户和密码在 BSQL 中为 bigmath 和 bigmath，在 BCQL 中默认用户和密码为 cassandra 和 cassandra。

### **为安全的本地多节点集群创建证书**

安全集群使用传输中加密，这需要集群中每个节点的 SSL/TLS 证书。 使用 --cert generate_server_certs 命令生成证书，然后将它们复制到相应的节点基目录，然后再创建安全的本地多节点集群。

创建 SSL 和 TLS 连接的证书：

```
./bin/bm-ctl cert generate_server_certs --hostnames=127.0.0.1,127.0.0.2,127.0.0.3
```

证书在 <HOME>/var/ generated_certs/<hostname> 目录中生成。

将证书复制到相应节点的 <base_dir>/certs 目录：

```
cp $HOME/var/generated_certs/127.0.0.1/* $HOME/bigmath-2.20.1.3/node1/certs
cp $HOME/var/generated_certs/127.0.0.2/* $HOME/bigmath-2.20.1.3/node2/certs
cp $HOME/var/generated_certs/127.0.0.3/* $HOME/bigmath-2.20.1.3/node3/certs
```

### **创建本地多节点集群**

要创建具有多个节点的集群，首先创建一个节点，然后使用 --join 标志创建其他节点以将它们添加到集群中。 如果节点重新启动，您还可以使用 --join 标志重新加入集群。

要创建安全的多节点集群，请确保您已为每个节点生成并复制了证书。

要创建没有加密和身份验证的集群，请省略 --secure 标志。

要创建集群，请执行以下操作：

1.通过运行以下命令启动第一个节点：

```
./bin/bm-ctl start --secure --advertise_address=127.0.0.1 \
    --base_dir=$HOME/bigmath-2.20.1.3/node1 \
    --cloud_location=aws.us-east-1.us-east-1a
```

2.在 macOS 上，为其他节点配置环回地址，如下所示：

```
sudo ifconfig lo0 alias 127.0.0.2
sudo ifconfig lo0 alias 127.0.0.3
```

3.使用 --join 标志向集群添加另外两个节点，如下所示：

```
./bin/bm-ctl start --secure --advertise_address=127.0.0.2 \
    --join=127.0.0.1 \
    --base_dir=$HOME/bigmath-2.20.1.3/node2 \
    --cloud_location=aws.us-east-1.us-east-1b
./bin/bm-ctl start --secure --advertise_address=127.0.0.3 \
    --join=127.0.0.1 \
    --base_dir=$HOME/bigmath-2.20.1.3/node3 \
    --cloud_location=aws.us-east-1.us-east-1c
```

### **创建多专区（可用区）集群**

1.创建安全的多可用区集群：
（1）通过运行 bm-ctl start 命令启动第一个节点，使用 --secure 标志并传入 --cloud_location 和 --fault_tolerance 标志来设置节点位置详细信息，如下所示：

```
./bin/bm-ctl start --secure --advertise_address=<host-ip> \
    --cloud_location=aws.us-east-1.us-east-1a \
    --fault_tolerance=zone
```

（2）为第二个和第三个虚拟机 (VM) 创建用于 SSL 和 TLS 连接的证书，如下所示：

```
./bin/bm-ctl cert generate_server_certs --hostnames=<IP_of_VM_2>,<IP_of_VM_3>
```

（3）将第一个虚拟机中生成的证书手动复制到第二个和第三个虚拟机中，如下所示：
将第二个虚拟机的证书从第一个虚拟机中的 \$HOME/var/ generated_certs/<IP_of_VM_2> 复制到第二个虚拟机中的 \$HOME/var/certs。

将第三个虚拟机的证书从第一个虚拟机中的 \$HOME/var/ generated_certs/<IP_of_VM_3> 复制到第三个虚拟机中的 \$HOME/var/certs。

（4）使用 --join 标志在两个单独的 VM 上启动第二个和第三个节点，如下所示：

```
./bin/bm-ctl start --secure --advertise_address=<host-ip> \
    --join=<ip-address-first-bm-ctl-node> \
    --cloud_location=aws.us-east-1.us-east-1b \
    --fault_tolerance=zone
./bin/bm-ctl start --secure --advertise_address=<host-ip> \
    --join=<ip-address-first-bm-ctl-node> \
    --cloud_location=aws.us-east-1.us-east-1c \
    --fault_tolerance=zone
```

2.创建没有SSL加密的多可用区集群
（1）通过运行 bm-ctl start 命令启动第一个节点，传入 --cloud_location 和 --fault_tolerance 标志以设置节点位置详细信息，如下所示：

```
./bin/bm-ctl start --advertise_address=<host-ip> \
    --cloud_location=aws.us-east-1.us-east-1a \
    --fault_tolerance=zone
```

（2）使用 --join 标志在两个单独的 VM 上启动第二个和第三个节点，如下所示：

```
./bin/bm-ctl start --advertise_address=<host-ip> \
    --join=<ip-address-first-bm-ctl-node> \
    --cloud_location=aws.us-east-1.us-east-1b \
    --fault_tolerance=zone
```

./bin/bm-ctl start --advertise_address=<host-ip> \

```
    --join=<ip-address-first-bm-ctl-node> \
    --cloud_location=aws.us-east-1.us-east-1c \
    --fault_tolerance=zone
```

 

在所有节点上启动bm-ctl进程后，配置集群的数据放置约束如下：

```
./bin/bm-ctl configure data_placement --fault_tolerance=zone
```

上述命令根据集群中每个节点的 --cloud_location 自动确定数据放置约束。 如果集群中有三个或更多可用区域，configure 命令会将集群配置为在至少一个可用区域发生故障时仍能幸存。 否则，它会输出警告消息。

集群的复制因子默认为3。

您可以使用 --constraint_value 标志手动设置数据放置约束，该标志采用以逗号分隔的 cloud.region.zone 值。 例如：

```
./bin/bm-ctl configure data_placement --fault_tolerance=zone \
    --constraint_value=aws.us-east-1.us-east-1a,aws.us-east-1.us-east-1b,aws.us-east-1.us-east-1c \
```

您可以使用 --rf 标志手动设置集群的复制因子。 例如：

```
./bin/bm-ctl configure data_placement --fault_tolerance=zone \
    --constraint_value=aws.us-east-1.us-east-1a,aws.us-east-1.us-east-1b,aws.us-east-1.us-east-1c \
    --rf=3
```

### **创建多区域集群**

1.创建安全的多区域集群：
（1）通过运行 bm-ctl start 命令启动第一个节点，使用 --secure 标志并传入 --cloud_location 和 --fault_tolerance 标志来设置节点位置详细信息，如下所示：

```
./bin/bm-ctl start --secure --advertise_address=<host-ip> \
    --cloud_location=aws.us-east-1.us-east-1a \
    --fault_tolerance=region
```

（2）为第二个和第三个虚拟机 (VM) 创建用于 SSL 和 TLS 连接的证书，如下所示：

```
./bin/bm-ctl cert generate_server_certs --hostnames=<IP_of_VM_2>,<IP_of_VM_3>
```

（3）手动将第一个虚拟机中生成的证书复制到第二个和第三个虚拟机：
将第二个虚拟机的证书从第一个虚拟机中的 $HOME/var/ generated_certs/<IP_of_VM_2> 复制到第二个虚拟机中的 $HOME/var/certs。

将第三个虚拟机的证书从第一个虚拟机中的 $HOME/var/ generated_certs/<IP_of_VM_3> 复制到第三个虚拟机中的 $HOME/var/certs。

（4）使用 --join 标志在两个单独的 VM 上启动第二个和第三个节点，如下所示：

```
./bin/bm-ctl start --secure --advertise_address=<host-ip> \
    --join=<ip-address-first-bm-ctl-node> \
    --cloud_location=aws.us-west-1.us-west-1a \
    --fault_tolerance=region
./bin/bm-ctl start --secure --advertise_address=<host-ip> \
    --join=<ip-address-first-bm-ctl-node> \
    --cloud_location=aws.us-central-1.us-central-1a \
    --fault_tolerance=region
```

2.创建多区域集群：
（1）通过运行 bm-ctl start 命令启动第一个节点，传入 --cloud_location 和 --fault_tolerance 标志以设置节点位置详细信息，如下所示：

```
./bin/bm-ctl start --advertise_address=<host-ip> \
    --cloud_location=aws.us-east-1.us-east-1a \
    --fault_tolerance=region
```

（2）使用 --join 标志在两个单独的 VM 上启动第二个和第三个节点，如下所示：

```
./bin/bm-ctl start --advertise_address=<host-ip> \
    --join=<ip-address-first-bm-ctl-node> \
    --cloud_location=aws.us-west-1.us-west-1a \
    --fault_tolerance=region
./bin/bm-ctl start --advertise_address=<host-ip> \
    --join=<ip-address-first-bm-ctl-node> \
    --cloud_location=aws.us-central-1.us-central-1a \
    --fault_tolerance=region
```

在所有节点上启动bm-ctl进程后，配置集群的数据放置约束如下：

```
./bin/bm-ctl configure data_placement --fault_tolerance=region
```

上述命令根据集群中每个节点的 --cloud_location 自动确定数据放置约束。 如果集群中有三个或更多区域，configure 命令会将集群配置为在至少一个区域发生故障时仍能幸存。 否则，它会输出警告消息。

集群的复制因子默认为3。

您可以使用 --constraint_value 标志手动设置数据放置约束，该标志采用以逗号分隔的 cloud.region.zone 值。 例如：

```
./bin/bm-ctl configure data_placement \
    --fault_tolerance=region \
    --constraint_value=aws.us-east-1.us-east-1a,aws.us-west-1.us-west-1a,aws.us-central-1.us-central-1a
```

您可以使用 --rf 标志手动设置集群的复制因子。 例如：

```
./bin/bm-ctl configure data_placement \
    --fault_tolerance=region \
    --constraint_value=aws.us-east-1.us-east-1a,aws.us-west-1.us-west-1a,aws.us-central-1.us-central-1a \
    --rf=3
```

### **启用和禁用静态加密**

要在已部署的本地集群中启用静态加密，请运行以下命令：

```
./bin/bm-ctl configure encrypt_at_rest \
    --enable \
    --base_dir=$HOME/bigmath-2.20.1.3/node1
```

要在已部署的多可用区或多区域集群中启用静态加密，请从任何虚拟机运行以下命令：

```
./bin/bm-ctl configure encrypt_at_rest --enable
```

要在启用静态加密的本地集群中禁用静态加密，请运行以下命令：

```
./bin/bm-ctl configure encrypt_at_rest \
    --disable \
    --base_dir=$HOME/bigmath-2.20.1.3/node1
```

要在启用了此类加密的多可用区或多区域集群中禁用静态加密，请从任何虚拟机运行以下命令：

```
./bin/bm-ctl configure encrypt_at_rest --disable
```

### **将附加标志传递给 DBServer**

创建单节点集群并为 DBServer 进程设置附加标志：

```
./bin/bm-ctl start --dbserver_flags="pg_bm_session_timeout_ms=1200000,bsql_max_connections=400"
```

## **升级 AiSQL 集群**

要使用数据库的最新功能并应用最新的安全修复程序，请将 AiSQL 集群升级到最新版本。

升级使用 bm-ctl 部署的现有 AiSQL 集群包括以下步骤：
1.使用 bm-ctl stop 命令停止正在运行的 AiSQL 节点。
2.通过执行 bm-ctl start 命令启动新的 bm-ctl 进程。 重启实例时使用之前配置的 --base_dir 。

在集群的所有节点上重复这些步骤，一次一个节点。

### **将集群从单可用区升级为多可用区**

以下步骤假设您有一个使用 bm-ctl 部署的正在运行的 AiSQL 集群，并且已下载更新：
1.使用 bm-ctl stop 命令停止第一个节点：

```
./bin/bm-ctl stop
```

2.使用 bm-ctl start 命令启动 AiSQL 节点，并提供必要的云信息，如下所示：

```
./bin/bm-ctl start --advertise_address=<host-ip> \
  --cloud_location=aws.us-east-1.us-east-1a \
  --fault_tolerance=zone
```

3.在集群的所有节点上重复上一步，一次一个节点。 如果要在本地计算机上部署集群，请使用 --base-dir 标志指定每个节点的基目录。

4.启动所有节点后，使用以下命令指定集群上的数据放置约束：

```
./bin/bm-ctl configure data_placement --fault_tolerance=zone
```

要手动指定数据放置约束，请使用以下命令：

```
./bin/bm-ctl configure data_placement \
  --fault_tolerance=zone \
  --constraint_value=aws.us-east-1.us-east-1a,aws.us-east-1.us-east-1b,aws.us-east-1.us-east-1c \
  --rf=3
```