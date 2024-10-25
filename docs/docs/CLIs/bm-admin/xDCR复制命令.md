## **setup_universe_replication**

为指定的源集群设置集群复制。只有在没有配置复制表的情况下，才能使用此命令。如果已经为复制配置了表，请使用alter_universe_replication来添加更多的表。
要验证是否已经为复制配置了表，请使用list_cdc_streams。

基本语法为：

```
./bm-admin \
 -mserver_addresses <target_universe_mserver_addresses> \
setup_universe_replication <source_universe_uuid>_<replication_stream_name>  \
    <source_universe_mserver_addresses> \
    <table_id>,[<table_id>..]
```

解释：
target_mserver_addresss:目标mserver主机和端口的逗号分隔列表。默认值为localhost:11000。
source_universe_uid：源集群的uuid。
replication_name：复制的名称。
source_mserver_addresss：以逗号分隔的源mserver主机地址列表。
comma_separated_list_of_table_ids:源集群表标识符（table_id）的逗号分隔列表。
comma_separated_list_of_producer_bootstrap_ids：源集群引导程序标识符（bootstrap_id）的逗号分隔列表。使用bootstrap_cdc_producer来获取，并使用逗号分隔源集群的表ID。

重要提示：bootstrap_id需要与table_id的排序顺序必须保持一致。

例如：

```
./bm-admin \
-mserver_addresses 10.0.0.4:11000,10.0.0.5:11000,10.0.0.6:11000 \
setup_universe_replication 4b0ec5da-3582-45b3-aa47-98d9425a5f5c  \
10.0.0.1:11000,10.0.0.2:11000,10.0.0.3:11000 \    000030a5000030008000000000004000,000030a5000030008000000000004005,dfef757c415c4b2cacc9315b8acb539a
```

## **alter_universe_replication**

更改指定源集群的集群复制。使用此命令可以执行以下操作：

* 在现有复制UUID中添加或删除表。
* 修改源主机地址。
  如果没有为复制配置表，请使用setup_universe_replication。
  要检查是否为复制配置了表，请使用list_cdc_streams。

使用set_mserver_addresss子命令来替换源主机地址列表。如果源上的主机发生更改，请使用此选项：

基本语法为：

```
./bm-admin \
-mserver_addresses <target_mserver_addresses> \
    alter_universe_replication <source_universe_uuid>_<replication_name> \
    set_mserver_addresses <source_mserver_addresses>
```

解释：
target_mserver_addresss:目标mserver主机和端口的逗号分隔列表。默认值为localhost:11000。
source_universe_uid：源集群的uuid。
replication_name：要更改的复制的名称。
source_mserver_addresss：以逗号分隔的源mserver主机地址列表。

使用add_table子命令将一个或多个表添加到现有列表中：

基本语法为：

```
./bm-admin \
-mserver_addresses <target_mserver_addresses> \
    alter_universe_replication <source_universe_uuid>_<replication_name> \
    add_table [ <comma_separated_list_of_table_ids> ] \
    [ <comma_separated_list_of_producer_bootstrap_ids> ]
```

解释：
target_mserver_addresss:目标mserver主机和端口的逗号分隔列表。默认值为localhost:11000。 
source_universe_uid：源集群的uuid。 
replication_name：要更改的复制的名称。 
comma_separated_list_of_table_ids:源集群表标识符（table_id）的逗号分隔列表。 
comma_separated_list_of_producter_bootstrap_ids：源集群引导程序标识符（bootstrap_id）的逗号分隔列表。使用bootstrap_cdc_producer来获取，并使用逗号分隔源集群的表ID。

重要提示：bootstrap_id需要与table_id的排序顺序必须保持一致。
使用remove_table子命令从现有列表中删除一个或多个表： 
基本语法为：

```
./bm-admin \
-mserver_addresses <target_mserver_addresses> \
    alter_universe_replication <source_universe_uuid>_<replication_name> \
    remove_table [ <comma_separated_list_of_table_ids> ]
```

target_mserver_addresss:目标mserver主机和端口的逗号分隔列表。默认值为localhost:11000。
source_universe_uid：源集群的uuid。
replication_name：要更改的复制的名称。
comma_separated_list_of_table_ids:源集群表标识符（table_id）的逗号分隔列表。

使用rename_id子命令可以重命名xDCR复制流。

基本语法为：

```
./bm-admin \
-mserver_addresses <target_mserver_addresses> \
    alter_universe_replication <source_universe_uuid>_<replication_name> \
    rename_id <source_universe_uuid>_<new_replication_name>
```

target_mserver_addresss:目标mserver主机和端口的逗号分隔列表。默认值为localhost:11000。
source_universe_uid：源集群的uuid。
replication_name：要更改的复制的名称。
new_replication_name：复制的新名称。 

## **delete_universe_replication <source_universe_uuid>**

删除指定源集群的集群复制。 

基本语法为：

```
./bm-admin \
-mserver_addresses <target_mserver_addresses> \
    delete_universe_replication <source_universe_uuid>_<replication_name>
```

target_mserver_addresss:目标mserver主机和端口的逗号分隔列表。默认值为localhost:11000。
source_universe_uid：源集群的uuid。
replication_name：要删除的复制的名称。

## **set_universe_replication_enabled**

可以设置复制为启用或禁用，来暂停复制过程。

基本语法为：

```
./bm-admin \
-mserver_addresses <target_mserver_addresses> \ 
set_universe_replication_enabled  <source_universe_uuid>_<replication_name> (0|1)
```

解释：
target_mserver_addresss:目标mserver主机和端口的逗号分隔列表。默认值为localhost:11000。
source_universe_uid：源集群的uuid。
replication_name：要启用或禁用的复制的名称。
0|1:0代表禁用，1代表启用。默认值为1。

## **list_cdc_streams**

列出指定mserver服务器的CDC流。 

基本语法为：

```
./bm-admin \
    -mserver_addresses <mserver-addresses> \
    list_cdc_streams
```

解释：
mserver-addresses: mserver主机和端口的逗号分隔列表。默认值为localhost:11000。

例如：

```
./bm-admin \
    -mserver_addresses 10.0.0.1:11000,10.0.0.2:11000,10.0.0.3:11000 \
    list_cdc_streams
```

## **delete_cdc_stream <stream_id> [force_delete]**

删除指定mserver主机服务器的基础CDC流。

基本语法为：

```
./bm-admin \
    -mserver_addresses <mserver-addresses> \
    delete_cdc_stream <stream_id [force_delete]>
```

解释：
mserver-addresses：mserver主机和端口的逗号分隔列表。默认值为localhost:11000。 
stream_id：CDC流的id。 
force_delete：（可选）强制执行删除操作。 

## **bootstrap_cdc_producer <comma_separated_list_of_table_ids>**

标记一组表，为设置集群的复制做准备。

基本语法为：

```
./bm-admin \
    -mserver_addresses <mserver-addresses> \
    bootstrap_cdc_producer <comma_separated_list_of_table_ids>
```

解释：
mserver-addresses：mserver主机和端口的逗号分隔列表。默认值为localhost:11000。
comma_separated_list_of_table_ids：以逗号分隔的表标识符列表（table_id）。 

例如：

```
./bm-admin \
    -mserver_addresses 10.0.0.1:11000,10.0.0.2:11000,10.0.0.3:11000 \
    bootstrap_cdc_producer 000030ad000030008000000000004000
```

## **get_replication_status**

返回所有使用者流的复制状态。如果提供了producter_universe_uuid，则只返回属于关联集群密钥的流。 

基本语法为：

```
./bm-admin \
-mserver_addresses <mserver-addresses> get_replication_status [ <producer_universe_uuid> ]
```

解释：
mserver-addresses：mserver主机和端口的逗号分隔列表。默认值为localhost:11000。 
producter_universe_uuid：可选的集群唯一标识符（可以是任何字符串，例如uuid的字符串）

例如：

```
./bm-admin \
    -mserver_addresses 10.0.0.4:11000,10.0.0.5:11000,10.0.0.6:11000     get_replication_status 4b0ec5da-3582-45b3-aa47-98d9425a5f5c
```