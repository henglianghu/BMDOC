bm-dev-ctl实用程序位于AiSQL home的bin目录中，它提供了一个命令行接口，用于管理用于开发和学习的本地集群。它会调用bm-dbserver和bm-mserver服务器来执行必要的任务。
bm-dev-ctl仅用于管理本地集群。这意味着，可以使用像本地笔记本电脑这样的单个主机来模拟AiSQL集群，集群可以有3个或更多节点。
bm-dev-ctl可以管理集群，前提是它最初是通过bm-dev-ctl创建的。这意味着通过任何其它方式创建的集群，包括Deploy部分中的集群，都不能使用bm-dev-ctl进行管理。

注意：
MacOS Monterey
MacOS Monterey默认启用AirPlay接收，监听端口10000。这会与AiSQL发生冲突，并导致bm-dev-ctl启动失败。则，启动集群时需要使用--mserver_flags标志来更改默认端口号，如下所示：

```
bigmath@bigmath-virtual-machine:$./bm-dev-ctl start --mserver_flags "webserver_port=7001"
```

或者，您可以禁用AirPlay接收，然后正常启动AiSQL，然后重新启用AirPlay接收（可选）。
基本语法为：
从AiSQL主目录运行bm-dev-ctl命令。

```
./bm-dev-ctl [command][flag1，flag2，…]
```

联机帮助：
要显示联机帮助，请从AiSQL主目录运行bm-dev-ctl --help。

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl --help
```

## **Commands**

bm-dev-ctl支持如下一系列命令。

### **create**

创建本地AiSQL集群。在没有flag的情况下，将会创建一个单节点集群。
进入AiSQL工作目录，执行如下命令来创建集群：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl create
```

当集群创建好后，会打印如下信息，其中包含集群存储目录：$HOME/biginsights-data/ 等信息。例如：/home/bigmath/biginsights-data。打印信息如下：

```
Creating cluster.
Waiting for cluster to be ready.
----------------------------------------------------------------------------------------------------
| Node Count: 1  | Replication Factor: 1                                 
----------------------------------------------------------------------------------------------------
| JDBC            : jdbc:postgresql://127.0.0.1:2521/biginsights            
| BSQL Shell       : sqlsh                                           
| BCQL Shell       : cqlsh                           
| BEDIS Shell       : redis-cli                          
| Web UI           : http://127.0.0.1:10000/                          
| Cluster Data       : /home/bigmath/biginsights-data                          
----------------------------------------------------------------------------------------------------
 
For more info, please use: bm-dev-ctl status
```

当您使用bm-dev-ctl时，您可以将“自定义”标志（bm-dev-ctl中不可直接用的标志）传递给mserver和dbserver服务器。例如：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl --rf 1 create 
--mserver_flags "log_cache_size_limit_mb=128,log_min_seconds_to_retain=20,mserver_backup_svc_queue_length=70" 
--dbserver_flags "log_inject_latency=false,log_segment_size_mb=128,raft_heartbeat_interval_ms=1000"
```

若要处理值包含逗号或等于的标志，请用双引号将整个键值对引起来。例如：

```
bigmath@bigmath-virtual-machine:~$./bm-dev-ctl create --dbserver_flags 'sql_enable_auth=false,"vmodule=tablet_service=1,pg_doc_op=1",bsql_prefetch_limit=1000'
```

### **start**

启动现有群集，如果群集不存在，则将会创建并启动一个单节点群集。
进入AiSQL工作目录，执行如下命令来启动集群：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl start
```


如存在集群，则启动好后，会打印如下信息：

```
Starting cluster with base directory /home/bigmath/biginsights-data
Waiting for cluster to be ready.
----------------------------------------------------------------------------------------------------
| Node Count: 1 | Replication Factor: 1                                                           
----------------------------------------------------------------------------------------------------
| JDBC               : jdbc:postgresql://127.0.0.1:2521/biginsights                               
| BSQL Shell          : sqlsh                                                                     
| BCQL Shell          : cqlsh                                                                    
| BEDIS Shell         : redis-cli                                                                 
| Web UI             : http://127.0.0.1:10000/                                                    
| Cluster Data         : /home/bigmath/biginsights-data                                             
----------------------------------------------------------------------------------------------------
 
For more info, please use: bm-dev-ctl status
```

如果集群不存在，则会打印如下信息：

```
Creating cluster.
Waiting for cluster to be ready.
----------------------------------------------------------------------------------------------------
| Node Count: 1  | Replication Factor: 1                                 
----------------------------------------------------------------------------------------------------
| JDBC            : jdbc:postgresql://127.0.0.1:2521/biginsights            
| BSQL Shell       : sqlsh                                           
| BCQL Shell       : cqlsh                                                                    
| BEDIS Shell       : redis-cli                                                                  
| Web UI           : http://127.0.0.1:10000/                                                     
| Cluster Data       : /home/bigmath/biginsights-data                                             
----------------------------------------------------------------------------------------------------
 
For more info, please use: bm-dev-ctl status
```

### **stop**

停止集群服务。
当需要停止集群服务时，进入AiSQL工作目录，请执行如下命令来停止集群：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl stop
```

在停止集群成功后，会打印如下信息：

```
Stopping cluster.
```

### **destroy**

当确认不再需要集群服务时，请执行如下命令来销毁集群，并同时释放相关存储空间。

请注意：此命令将停止所有节点，并同时删除该集群的数据目录。例如：集群创建的数据目录/home/bigmath/biginsights-data 将被删除。

进入AiSQL工作目录，请执行如下命令来销毁集群：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl destroy
```

在销毁集群成功后，会打印如下信息：

```
Destroying cluster.
```

### **status**

获取本地集群的健康状态。

进入AiSQL工作目录，请执行如下命令来进行检查集群状态：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl status
```

检查集群，会打印如下信息：

```
--------------------------------------------------------------------------------------------------------------
| Node Count: 1 | Replication Factor: 1                                            
--------------------------------------------------------------------------------------------------------------
| JDBC               : jdbc:postgresql://127.0.0.1:2521/biginsights                 
| BSQL Shell          : sqlsh                                               
| BCQL Shell          : cqlsh                                               
| BEDIS Shell          : redis-cli                                               
| Web UI              : http://127.0.0.1:10000/                                   
| Cluster Data          : /home/bigmath/biginsights-data                            
--------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------
| Node 1: bm-dbserver (pid 454612), bm-mserver (pid 454609)                              
--------------------------------------------------------------------------------------------------------------
| JDBC            : jdbc:postgresql://127.0.0.1:2521/biginsights                  
| BSQL Shell       : sqlsh                                                 
| BCQL Shell       : cqlsh                                                 
| BEDIS Shell       : redis-cli                                               
| data-dir[0]         : /home/bigmath/biginsights-data/node-1/disk-1/bm-data            
| bm-dbserver Logs    : /home/bigmath/biginsights-data/node-1/disk-1/bm-data/dbserver/logs
| bm-mserver Logs    : /home/bigmath/biginsights-data/node-1/disk-1/bm-data/mserver/logs
--------------------------------------------------------------------------------------------------------------
```

### **restart**

重新启动当前集群。请注意，如果重新启动集群，所有自定义的flags 和placement信息都将丢失。尽管如此，您可以像在bm-dev-ctl create命令中传递placement信息和自定义flags 一样传递它们。
进入AiSQL工作目录，请执行如下命令来进行重新启动当前集群：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl restart
```

在重新启动集群成功后，会打印如下信息：

```
Stopping cluster.
Starting cluster.
Waiting for cluster to be ready.
-------------------------------------------------------------------------------------------------
| Node Count: 1 | Replication Factor: 1                        
-------------------------------------------------------------------------------------------------
| JDBC               : jdbc:postgresql://127.0.0.1:2521/biginsights 
| BSQL Shell          : sqlsh                                             
| BCQL Shell          : cqlsh                                                  
| BEDIS Shell          : redis-cli                           
| Web UI              : http://127.0.0.1:10000/           
| Cluster Data          : /home/bigmath/biginsights-data    
-------------------------------------------------------------------------------------------------
 
For more info, please use: bm-dev-ctl status
```

可以使用云、区域和区域标志重新启动集群，例如：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl restart --placement_info “cloud1.region1.zone1”
```

可以使用用户标志来重新启动集群，例如：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl restart  --mserver_flags "log_cache_size_limit_mb=128,log_min_seconds_to_retain=20,mserver_backup_svc_queue_length=70" --dbserver_flags "log_inject_latency=false,log_segment_size_mb=128,raft_heartbeat_interval_ms=1000"
```

### **wipe_restart**

停止并销毁当前集群，同时删除所有数据文件，再重新创建并启动新的集群（丢失所有flags）。
进入AiSQL工作目录，请执行如下命令来进行销毁并重新启动集群：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl wipe_restart
```

在执行成功后，会打印如下信息：

```
Destroying cluster.
Creating cluster.
Waiting for cluster to be ready.
----------------------------------------------------------------------------------------------------
| Node Count: 1  | Replication Factor: 1                                 
----------------------------------------------------------------------------------------------------
| JDBC            : jdbc:postgresql://127.0.0.1:2521/biginsights            
| BSQL Shell       : sqlsh                                           
| BCQL Shell       : cqlsh                           
| BEDIS Shell       : redis-cli                          
| Web UI           : http://127.0.0.1:10000/                          
| Cluster Data       : /home/bigmath/biginsights-data                          
----------------------------------------------------------------------------------------------------
 
For more info, please use: bm-dev-ctl status
```

擦除并使用placement 标志重新启动集群，例如：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl wipe_restart --placement_info "cloud1.region1.zone1"
```

擦除并使用用户自定义标志重新启动集群，例如：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl wipe_restart --mserver_flags "log_cache_size_limit_mb=128,log_min_seconds_to_retain=20,mserver_backup_svc_queue_length=70" --dbserver_flags "log_inject_latency=false,log_segment_size_mb=128,raft_heartbeat_interval_ms=1000"
```

### **add_node**

将启动一个新的dbserver服务节点，并为其提供一个新node_id，同时将其添加到当前群集。它还采用了一个可选的标志--mserver，表示要添加的服务器是mserver。
进入AiSQL工作目录，请执行如下命令来进行增加服务节点：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl add_node
```

在执行成功后，会打印如下信息：

```
Adding node.
Waiting for cluster to be ready.
----------------------------------------------------------------------------------------------------
| Node 3: bm-dbserver (pid 1555129)                  
----------------------------------------------------------------------------------------------------
| JDBC             : jdbc:postgresql://127.0.0.3:2521/biginsights         
| BSQL Shell        : sqlsh -h 127.0.0.3                 
| BCQL Shell        : cqlsh 127.0.0.3                    
| BEDIS Shell        : redis-cli -h 127.0.0.3                            
| data-dir[0]          : /home/bigmath/biginsights-data/node-3/disk-1/bm-data          
| bm-dbserver Logs      : /home/bigmath/biginsights-data/node-3/disk-1/bm-data/dbserver/logs 
----------------------------------------------------------------------------------------------------
```

要添加具有自定义mserver标志的节点，请执行以下操作，例如：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl add_node --mserver_flags "log_cache_size_limit_mb=128,log_min_seconds_to_retain=20"
```

要添加具有自定义dbserver标志的节点，请执行以下操作，例如：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl add_node --dbserver_flags "log_inject_latency=false,log_segment_size_mb=128"
```

### **remove_node**

该命令将必须删除的节点的node_id作为输入，用以停止正在运行的群集中的指定的节点服务。它还带有一个可选的标志--mserver，表示服务器是mserver服务器。
注意：当前stop_node和remove_node实现完全相同的行为，它们可以互换使用。
进入AiSQL工作目录，请执行如下命令来停止指定节点服务：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl remove_node 3
```

在执行成功后，会打印如下信息：

```
Stopping node dbserver-3.
```

可以传递一个可选的标志--mserver，它表示服务器是mserver服务器。例如：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl remove_node 3 --mserver
```

### **start_node**

启动正在运行的群集中的指定节点。它还带有一个可选的标志--mserver，表示服务器是mserver服务器。
进入AiSQL工作目录，请执行如下命令来启动指定节点服务：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl start_node 3
```

在执行成功后，会打印如下信息：

```
Starting node dbserver-3.
Waiting for node to be ready.
----------------------------------------------------------------------------------------------------
| Node 3: bm-dbserver (pid 1560892)                                                                 
----------------------------------------------------------------------------------------------------
| JDBC             : jdbc:postgresql://127.0.0.3:2521/biginsights                 
| BSQL Shell        : sqlsh -h 127.0.0.3                              
| BCQL Shell       : cqlsh 127.0.0.3                       
| BEDIS Shell       : redis-cli -h 127.0.0.3                  
| data-dir[0]         : /home/bigmath/biginsights-data/node-3/disk-1/bm-data   
| bm-dbserver Logs     : /home/bigmath/biginsights-data/node-3/disk-1/bm-data/dbserver/logs 
---------------------------------------------------------------------------------------------------
```

可以传递一个可选的标志--mserver，它表示服务器是mserver服务器。例如：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl start_node 3 --mserver
```

在执行成功后，会打印如下信息：

```
Starting node mserver-3.
Waiting for cluster to be ready.
----------------------------------------------------------------------------------------------------
| Node 3: bm-dbserver (pid 1560892), bm-mserver (pid 1561184)                                        
----------------------------------------------------------------------------------------------------
| JDBC            : jdbc:postgresql://127.0.0.3:2521/biginsights                
| BSQL Shell       : sqlsh -h 127.0.0.3                        
| BCQL Shell       : cqlsh 127.0.0.3                     
| BEDIS Shell       : redis-cli -h 127.0.0.3                     
| data-dir[0]         : /home/bigmath/biginsights-data/node-3/disk-1/bm-data        
| bm-dbserver Logs     : /home/bigmath/biginsights-data/node-3/disk-1/bm-data/dbserver/logs  
| bm-mserver Logs     : /home/bigmath/biginsights-data/node-3/disk-1/bm-data/mserver/logs  
----------------------------------------------------------------------------------------------------
```

### **stop_node**

停止正在运行的群集中的指定节点。它还带有一个可选的标志--mserver，表示服务器是mserver服务器。
进入AiSQL工作目录，请执行如下命令来停止指定节点服务：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl stop_node 3
```

在执行成功后，会打印如下信息：

```
Stopping node dbserver-3.
```

可以传递一个可选的标志--mserver，它表示服务器是mserver服务器。例如：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl stop_node 3 --mserver
```

在执行成功后，会打印如下信息：

```
Stopping node mserver-3.
```

### **restart_node**

重新启动正在运行的群集中的指定节点。它还采用了一个可选的标志--mserver，表示服务器是mserver服务器。
bm-dev-ctl重新启动首先停止节点，然后再次启动它。此时，该节点未从集群中退出。因此，此命令的主要优点之一是可以用于清除旧标志并传入新标志。就像create一样，可以在bm-dev-ctl restart命令中传递cloud/region/zone和自定义标志。
进入AiSQL工作目录，请执行如下命令来重新启动指定节点：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl restart_node 3
```

在执行成功后，会打印如下信息：

```
Stopping node dbserver-3.
Starting node dbserver-3.
Waiting for cluster to be ready.
```

可以传递一个可选的标志--mserver，它表示服务器是mserver服务器。例如：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl restart_node 3 --mserver
```

在执行成功后，会打印如下信息：

```
Stopping node mserver-3.
Starting node mserver-3.
Waiting for cluster to be ready.
```

可以使用云、区域和区域标志重新启动指定节点，例如：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl restart_node 2 --placement_info "cloud1.region1.zone1"
```

可以使用用户标志来重新启动指定节点，例如：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl restart_node 2 --mserver --mserver_flags "log_cache_size_limit_mb=128,log_min_seconds_to_retain=20"
```

### **setup_redis**

setup_redis命令用于初始化AiSQL兼容redis的BEDIS API。
进入AiSQL工作目录，请执行如下命令来初始化：

```
bigmath@bigmath-virtual-machine:~$ ./bm-dev-ctl setup_redis
```

在执行成功后，会打印如下信息：

```
Setting up BigInsights DB support for Redis API.
Waiting for cluster to be ready.
Setup Redis successful.
```

## **Flags**

bm-dev-ctl支持如下标志（flags）。

### **--help, -h**

显示帮助信息。

### **--binary_dir**

指定在其中查找AiSQL mserver文件和dbserver二进制文件的目录。
默认值：＜AiSQL安装目录＞/bin

### **--data_dir**

指定AiSQL数据存储目录。
默认值：$HOME/biginsights-data/
注意：不支持在创建集群后更改此标志的值。

### **--mserver_flags**

指定mserver标志列表，用逗号分隔。

### **--dbserver_flags**

指定dbserver标志列表，用逗号分隔。

### **--placement_info**

指定cloud，region，zone 为 cloud.region.zone，用逗号分隔。
默认值：cloud1.datacenter1.rack1

### **--replication_factor, -rf**

指定每个分片的副本数，也称为复制因子（RF）。应该是一个奇数，这样才能达成多数人的共识。创建容错集群需要最小值为3，因为只有1个副本是没有容错的。
该值也被设置为mserver服务器的默认数量。
默认值：1

### **--require_clock_sync**

指定AiSQL是否需要群集中节点之间的时钟同步。
默认值：false

### **--listen_ip**

指定要侦听的单节点群集的IP地址或端口。若要启用AiSQL API和管理端口的外部访问，请将该值设置为0.0.0.0。请注意，此标志不适用于多节点集群。
默认值：127.0.0.1

### **--num_shards_per_dbserver**

启动分片服务，分配给每一个表的分片数量。
默认值：2

### **--timeout-bm-admin-sec**

调用bm-admin并在集群上等待的操作的超时时间（以秒为单位）。

### **--timeout-processes-running-sec**

在集群上等待的操作的超时时间（以秒为单位）。

### **--verbose**

将内部调试消息记录到stderr的标志。 