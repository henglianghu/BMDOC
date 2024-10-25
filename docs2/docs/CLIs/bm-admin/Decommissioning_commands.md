## **get_leader_blacklist_completion**

获取列入黑名单的节点的分片加载移动完成百分比。

基本语法为：

```
./bm-admin 
    -mserver_addresses <mserver-addresses> \
    get_leader_blacklist_completion
```

解释：
mserver-addresses：mserver主机和端口的逗号分隔列表。默认值为localhost:11000。

例如：

```
./bm-admin --mserver_addresses 10.0.0.4:11000,10.0.0.5:11000,10.0.0.6:11000 get_leader_blacklist_completion
```

执行成功，会打印类似如下信息：

```
Percent complete = 100 : 0 remaining out of 0
```

## **change_blacklist**

更改dbserver服务器的黑名单，当旧的dbserver服务器终止后，可以使用此命令清除黑名单。 

基本语法为：

```
./bm-admin 
    -mserver_addresses <mserver-addresses> \
    change_blacklist [ ADD | REMOVE ] <ip_addr>:<port> \
    [ <ip_addr>:<port> ]...
```

解释：
mserver-addresses：mserver主机和端口的逗号分隔列表。默认值为localhost:11000。
ADD | REMOVE: 从黑名单中新增或者移除指定的dbserver服务器。
ip_addr:port: dbserver服务器IP地址和端口，如多个，需用空格分隔。

例如：

```
./bm-admin --mserver_addresses 10.0.0.1:11000,10.0.0.2:11000,10.0.0.3:11000 change_blacklist ADD 10.0.0.4:21000 10.0.0.5:21000
```


## **change_leader_blacklist**

更改dbserver服务器的Leader黑名单。 

基本语法为：

```
./bm-admin 
    -mserver_addresses <mserver-addresses> \
    change_blacklist [ ADD | REMOVE ] <ip_addr>:<port> \
    [ <ip_addr>:<port> ]...
```

解释：
mserver-addresses：mserver主机和端口的逗号分隔列表。默认值为localhost:11000。
ADD | REMOVE: 从黑名单中新增或者移除指定的dbserver服务器。
ip_addr:port: dbserver服务器IP地址和端口，如多个，需用空格分隔。

例如：

```
./bm-admin --mserver_addresses 10.0.0.1:11000,10.0.0.2:11000,10.0.0.3:11000 change_leader_blacklist ADD 10.0.0.4:21000 10.0.0.5:21000
```


## **leader_stepdown**

强制指定分片的dbserver的Leader不为Leader。
注意：只有在支持人员建议的情况下，才能使用此命令，使用此命令，存在停机的可能性。

基本语法为：

```
./bm-admin 
    -mserver_addresses <mserver-addresses> \
    leader_stepdown <tablet_id> <dest_ts_uuid>
```

解释：
mserver-addresses：mserver主机和端口的逗号分隔列表。默认值为localhost:11000。
tablet_id：分片标识符ID。
dest_ts_uuid：新的dbserver Leader的目标标识符（uuid）。当需要Leader角色转移的时候，如果不需要指定新的Leader时，使用“”作为参数值，则系统将自动选举新的Leader；否则，指定新的Leader。

例如：

```
./bm-admin --mserver_addresses 10.0.0.1:11000,10.0.0.2:11000,10.0.0.3:11000 leader_stepdown 74eb92ad716b47b6b5a3a14ef25c1504
```