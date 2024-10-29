##### **upgrade_bsql**

在AiSQL集群升级成功后升级BSQL系统目录。对于未启用BSQL的集群，不需要进行BSQL升级。

基本语法为：

```
./bm-admin upgrade_bsql
```

例如：

```
./bm-admin upgrade_bsql
```

执行成功，会打印类似如下信息：

```
BSQL successfully upgraded to the latest version
```

在某些情况下，BSQL升级可能需要超过60秒的时间，这是bm-admin的默认超时值。为此，请使用更高的超时值运行命令：

```
./bm-admin -timeout_ms 180000 upgrade_bsql
```

运行上述命令是一个联机操作，不需要停止正在运行的集群。可以多次运行而没有任何副作用。

注意：
集群中的并发操作可能导致各种事务冲突、目录版本不匹配和读取重新启动错误，应该通过重新运行升级命令来解决。

请参阅[升级部署](#_升级部署)以了解mserver服务器和dbserver升级，然后是BSQL系统目录升级。