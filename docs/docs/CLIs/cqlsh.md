用于与 AiSQL BCQL API 交互的 Shell

## **概述**

BCQL shell (cqlsh) 是一个使用 BCQL 与 AiSQL 交互的 CLI。

**1.安装**
cqlsh 作为 AiSQL 的一部分安装，位于 AiSQL 主目录的 bin 目录中。 您还可以从 cqlsh GitHub 存储库下载它。

如果您愿意，可以使用 shell 脚本安装独立版本：

```
curl -sSL https://downloads.bigmath.com/get_clients.sh | bash
```

如果您有 wget，则可以使用以下命令：

```
wget -q -O - https://downloads.bigmath.com/get_clients.sh | sh
```

**2.启动** **cqlsh**

```
./bin/cqlsh
Connected to local cluster at 127.0.0.1:9542.
[cqlsh 5.0.1 | Cassandra 3.9-SNAPSHOT | CQL spec 3.4.2 | Native protocol v4]
Use HELP for help.
cqlsh>
```

**3.在线帮助**
运行 cqlsh --help 显示在线帮助。

## **语法**

cqlsh [flags] [host [port]]

host 是运行DBServer 的主机的IP 地址。 默认为本地主机 127.0.0.1。
port 是 DBServer 侦听 BCQL 连接的 TCP 端口。 默认值为 9542。

1.示例
./bin/cqlsh --execute "select cluster_name, data_center, rack from system.local" 127.0.0.1

```
 cluster_name  | data_center | rack
---------------+-------------+-------
 local cluster | datacenter1 | rack1
```

 

2.标志

| 标志                  | 简写 | 默认值 | 描述                                                         |
| --------------------- | ---- | ------ | ------------------------------------------------------------ |
| --color               | -C   |        | 强制颜色输出                                                 |
| --no-color            |      |        | 禁用颜色输出。                                               |
| --browser             |      |        | 指定用于显示 cqlsh 帮助的浏览器。 这可以是受支持的浏览器名称之一（例如 Firefox），也可以是后跟 %s 的浏览器路径（例如 /usr/bin/google-chrome-stable %s）。 |
| --ssl                 |      |        | 连接到 AiSQL 时使用 SSL。                                    |
| --user                | -u   |        | 用于对 AiSQL 进行身份验证的用户名。                          |
| --password            | -p   |        | 用于对AiSQL进行身份验证的密码应，与--user 结合使用           |
| --keyspace            | -k   |        | 用于身份验证的密钥空间，应与 --user 结合使用。               |
| --file                | -f   |        | 从给定文件执行命令，然后退出。                               |
| --debug               |      |        | 打印附加调试信息。                                           |
| --encoding            |      | UTF-8  | 指定输出的非默认编码。                                       |
| --cqlshrc             |      |        | 指定 cqlshrc 文件的位置。 cqlshrc 文件包含 cqlsh 的配置选项。 默认情况下，它位于用户主目录 ~/.cassandra/cqlsh 中。 |
| --execute             | -e   |        | 执行给定的语句，然后退出。                                   |
| --connect-timeout     |      | 2      | 指定连接超时（以秒为单位）。                                 |
| --request-timeout     |      | 10     | 指定请求超时（以秒为单位）。                                 |
| --tty                 | -t   |        | 强制 tty 模式（命令提示符）。                                |
| --refresh_on_describe | -r   |        | 强制刷新 DESCRIBE 上的架构元数据                             |

3.将 BCQL 输出保存到文件
要将 BCQL 语句的输出保存到文件，请使用 --execute 标志运行该语句，并将输出重定向到文件。
例如，要保存 SELECT 语句的输出：

```
./bin/cqlsh -e "SELECT * FROM mytable" > output.txt
```

## **特殊的命令**

除了支持常规的BCQL语句外，cqlsh还支持下列特殊命令。

1.Captures 
Captures命令输出并将其附加到指定的文件中。当输出被Captures时，它不会显示在控制台。

```
CAPTURE '<file>'
CAPTURE OFF
CAPTURE
```

* 要附加的文件的路径必须在字符串文字内给出。 该路径是相对于当前工作目录解释的。 支持使用波形符简写符号 (~/mydir) 来引用 $HOME。
* 仅捕获查询结果输出。 仅 cqlsh 命令的错误和输出仍显示在 cqlsh 会话中。
* 要停止捕获输出并再次在 cqlsh 会话中显示它，请使用 CAPTURE OFF。
* 要查看当前捕获配置，请使用不带参数的 CAPTURE。

2.clear
清除控制台。

```
CLEAR
CLS
```

3.CONSISTENCY 
CONSISTENCY <consistency level>

设置后续读取操作的一致性级别。 有效参数包括：

| 一致性级别 | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| QUORUM     | 从Tile的法定人数中读取高度一致的结果。 读取请求将仅由 Tablet Leader 处理。 这是默认的一致性级别。 |
| ONE        | 从具有宽松一致性保证的追随者处读取                           |

4.COPY FROM
将数据从 CSV 文件复制到表。
COPY <table name> [(<column>, ...)] FROM <file name> WITH <copy flag> [AND <copy flag> ...]

默认情况下，COPY FROM 将所有列从 CSV 文件复制到表中。 要复制列的子集，请在表名称后面添加用括号括起来的以逗号分隔的列名称列表。

file name应该是表示源文件路径的字符串文字（带单引号）。 使用特殊值 STDIN（不带单引号）从 stdin 读取 CSV 数据。

<table>
  <thead>
    <tr>
        <th>标志</th>
        <th>默认值</th>
        <th>描述</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>INGESTRATE</td>
        <td>100000</td>
        <td>每秒处理的最大行数。</td>
    </tr>
    <tr>
        <td>MAXROWS</td>
        <td>-1</td>
        <td>要导入的最大行数。 -1表示无限制。</td>
    </tr>
    <tr>
        <td>SKIPROWS</td>
        <td>0</td>
        <td>要跳过的初始行数。</td>
    </tr>
    <tr>
        <td>SKIPCOLS</td>
        <td></td>
        <td>要忽略的以逗号分隔的列名称列表。 默认情况下，不跳过任何列。</td>
    </tr>
    <tr>
        <td>MAXPARSEERRORS</td>
        <td>-1</td>
        <td>要忽略的最大全局解析错误数。 -1表示无限制。</td>
    </tr>
    <tr>
        <td>MAXINSERTERRORS</td>
        <td>1000</td>
        <td>要忽略的最大全局插入错误数。 -1表示无限制。</td>
    </tr>
    <tr>
        <td>ERRFILE=</td>
        <td></td>
        <td>一个文件，用于存储所有无法导入的行； 默认情况下，这是 import_&lt;ks&gt;_&lt;table&gt;.err 其中 &lt;ks&gt; 是键空间，&lt;table&gt; 是表名称。</td>
    </tr>
    <tr>
        <td>MAXBATCHSIZE</td>
        <td>20</td>
        <td>单个批次中插入的最大行数。</td>
    </tr>
    <tr>
        <td>MINBATCHSIZE</td>
        <td>2</td>
        <td>在单个批次中插入的最小行数。</td>
    </tr>
    <tr>
        <td>CHUNKSIZE</td>
        <td>1000</td>
        <td>一次从主进程传递给子工作进程的行数。</td>
    </tr>
  </tbody>
</table>


请参阅 COPY TO 了解 COPY TO 和 COPY FROM 共有的其他标志

5.COPY TO 
将数据从表复制到 CSV 文件。
COPY <table name> [(<column>, ...)] TO <file name> WITH <copy flag> [AND <copy flag> ...]

默认情况下，COPY TO 将表中的所有列复制到 CSV 文件。 要复制列的子集，请在表名称后面添加用括号括起来的以逗号分隔的列名称列表。

文件名应该是一个字符串文字（带单引号），表示目标文件的路径。 您还可以使用特殊值 STDOUT（不带单引号）将 CSV 打印到 stdout。

| 标志                 | 默认值 | 描述                                                         |
| -------------------- | ------ | ------------------------------------------------------------ |
| MAXREQUESTS          | 6      | 同时获取的最大令牌数量范围。                                 |
| PAGESIZE             | 1000   | 单页中要获取的行数。                                         |
| PAGETIMEOUT          | 10     | 页面大小或更小的每 1000 个条目的超时（以秒为单位）。         |
| BEGINTOKEN，ENDTOKEN |        | 要导出的令牌范围。 默认导出整个环。                          |
| MAXOUTPUTSIZE        | -1     | 输出文件的最大大小（以行数为单位）； 超过此最大值，输出文件将被分成多个段。 -1表示无限制。 |
| ENCODING             | utf8   | 用于字符的编码。                                             |

以下标志对于 COPY TO 和 COPY FROM 都是通用的。

| 标志            | 默认值     | 描述                                                         |
| --------------- | ---------- | ------------------------------------------------------------ |
| NULLVAL         | null       | 空值的字符串占位符。                                         |
| HEADER          | false      | 对于 COPY TO，控制 CSV 输出文件中的第一行是否包含列名称。 对于 COPY FROM，指定 CSV 输入文件中的第一行是否包含列名称。 |
| DELIMITER       | ,          | 用于分隔字段（列）的字符。                                   |
| DECIMALSEP      | .          | 用作小数点分隔符的字符。                                     |
| THOUSANDSSEP    |            | 用于分隔千位的字符。 默认为空字符串。                        |
| BOOLSTYlE       | True,False | 布尔值的字符串文字格式。                                     |
| NUMPROCESSES    |            | 为 COPY 任务创建的子工作进程数。 COPY FROM 的最大值默认为 4，COPY TO 的最大值默认为 16。 但是，最多会创建 (num_cores - 1) 个进程。 |
| MAXATTEMPTS     | 5          | 在放弃之前尝试获取一系列数据（使用 COPY TO 时）或插入一块数据（使用 COPY FROM 时）失败的最大次数。 |
| REPORTFREQUENCY | 0.25       | 状态更新的刷新频率（以秒为单位）。                           |
| RATEFILE        |            | 用于输出速率统计数据的可选文件。 默认情况下，统计信息不会输出到文件中。 |

6.DESCRIBE
打印模式元素或集群的描述（通常是一系列 DDL 语句）。 使用 DESCRIBE 转储全部或部分架构。

```
ESCRIBE CLUSTER
DESCRIBE SCHEMA
DESCRIBE KEYSPACES
DESCRIBE KEYSPACE <keyspace name>
DESCRIBE TABLES
DESCRIBE TABLE <table name>
DESCRIBE INDEX <index name>
DESCRIBE TYPES
DESCRIBE TYPE <type name>
```

在任何命令中，可以使用 DESC 代替 DESCRIBE。
DESCRIBE CLUSTER 命令打印集群名称：

```
cqlsh> DESCRIBE CLUSTER
 
Cluster: local cluster
```

DESCRIBE SCHEMA 命令打印重新创建整个架构所需的 DDL 语句。 使用此命令转储架构； 然后，您可以使用生成的文件克隆集群或从备份恢复。

7.EXIT
结束当前会话并终止 cqlsh 进程。

```
EXIT
QUIT
```

8.EXPAND
启用或禁用行的垂直打印。 当获取很多列或者单个列的内容很大时，请使用 EXPAND。

```
EXPAND ON
EXPAND OFF
```

要查看当前的扩展设置，请使用不带参数的 EXPAND

9.HELP 
提供有关 cqlsh 命令的信息。 要查看可用主题，请输入不带任何参数的 HELP。 要查看某个主题的帮助，请使用 HELP <topic>。 另请参阅 --browser 参数，用于控制使用哪个浏览器显示帮助。

```
HELP <topic>
```

10.LOGIN
以当前会话的指定 AiSQL 用户身份进行身份验证

```
LOGIN <username> [<password>]
```

11.PAGING 
启用分页、禁用分页或设置读取查询的页面大小。 启用分页后，一次仅获取一页数据，并会出现获取下一页的提示。 一般来说，最好在交互式会话中启用分页，以避免一次获取和打印大量数据。

```
PAGING ON
PAGING OFF
PAGING <page size in rows>
```

要查看当前分页设置，请使用不带参数的 PAGING。

12.SHOW HOST 
打印 cqlsh 连接到的 DBServer 服务器的 IP 地址和端口以及集群名称。 例子：

```
cqlsh> SHOW HOST
 
Connected to local cluster at 127.0.0.1:9542.
```

13.SHOW VERSION
打印正在使用的 cqlsh、Cassandra、CQL 和本机协议版本。 例子：

```
cqlsh> SHOW VERSION
 
[cqlsh 5.0.1 | Cassandra 3.9-SNAPSHOT | CQL spec 3.4.2 | Native protocol v4]
```

14.SOURCE
读取文件的内容并将每一行作为 BCQL 语句或特殊 cqlsh 命令执行。

```
SOURCE '<file>'
```

示例：

```
cqlsh> SOURCE '/home/bigmath/commands.cql'
```

15.TIMING
启用或禁用基本请求往返计时，在当前 BCQL shell 会话上测量。

```
TIMING ON | OFF
```

TIMING ON：为所有进一步的请求启用往返定时。
TIMING OFF：禁用定时。
TIMING（无参数）：输出当前计时状态。

示例：
您可以使用 TIMING 并从 BCQL 会话运行查询，如下所示：

```
cqlsh> TIMING
 
Timing is currently disabled. Use TIMING ON to enable.
cqlsh> TIMING ON
 
Now Timing is enabled
cqlsh> use example;
 
26.18 milliseconds elapsed
cqlsh:example> CREATE TABLE employees(department_id INT, employee_id INT,name TEXT, PRIMARY KEY(department_id, employee_id));
 
1042.67 milliseconds elapsed
cqlsh:example> INSERT INTO employees(department_id, employee_id, name) VALUES (1, 1, 'John');
 
16.89 milliseconds elapsed
cqlsh:example> INSERT INTO employees(department_id, employee_id, name) VALUES (1, 2, 'Jane');
 
11.65 milliseconds elapsed
cqlsh:example> SELECT * FROM employees;
 
 department_id | employee_id | name
---------------+-------------+------
             1 |           1 | John
             1 |           2 | Jane
5.76 milliseconds elapsed
(2 rows)
cqlsh:example> TIMING OFF
 
Disabled Timing.
```

 

 