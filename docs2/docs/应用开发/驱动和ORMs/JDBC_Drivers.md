

AiSQL JDBC 智能驱动程序是基于 PostgreSQL JDBC 驱动程序构建的用于 BSQL 的 JDBC 驱动程序，具有附加的连接负载平衡功能。

## **下载驱动依赖**
AiSQL JDBC 驱动程序可作为 Maven 依赖项使用。 通过在 java 项目中添加以下依赖项来下载驱动程序。

**1.Maven 依赖**
要从 Maven 获取驱动程序和 HikariPool，请将以下依赖项添加到 Maven 项目中：
```
<dependency>
  <groupId>com.bigmath</groupId>
  <artifactId>jdbc-brightdb</artifactId>
  <version>42.3.0</version>
</dependency>
 
<!-- https://mvnrepository.com/artifact/com.zaxxer/HikariCP -->
<dependency>
  <groupId>com.zaxxer</groupId>
  <artifactId>HikariCP</artifactId>
  <version>4.0.3</version>
</dependency>
```

**2.Gradle 依赖**
要获取驱动程序和 HikariPool，请将以下依赖项添加到 Gradle 项目中：
```
// https://mvnrepository.com/artifact/org.postgresql/postgresql
implementation 'com.bigmath:jdbc-brightdb:42.3.0'
implementation 'com.zaxxer:HikariCP:4.0.3'
```

## **基础知识**
了解如何使用 AiSQL JDBC 驱动程序执行 Java 应用程序开发所需的常见任务。

注：该驱动程序需要 AiSQL 版本 2.7.2.0 或更高版本以及 Java 8 或更高版本。

**1.负载平衡连接属性**
需要添加以下连接属性以启用负载平衡：
* load-balance：通过将此属性设置为 true 来启用集群感知负载平衡； 默认禁用。
* topology-keys：提供逗号分隔的地理位置值以启用拓扑感知负载平衡。 地理位置可以作为 cloud.region.zone 提供。 将区域中的所有区域指定为 cloud.region.*。 要在主要位置无法访问时指定后备位置，请以 :n 的形式指定优先级，其中 n 是优先顺序。 例如，cloud1.datacenter1.rack1:1、cloud1.datacenter1.rack2:2。

默认情况下，驱动程序每 300 秒（5 分钟）刷新一次节点列表。 您可以通过包含 bm-servers-refresh-interval 参数来更改此值。

**2.使用驱动程序**
AiSQL JDBC 驱动程序的驱动程序类是 com.bigmath.Driver。 驱动程序包包含一个 BMClusterAwareDataSource 类，该类使用 AiSQL 集群的一个初始接触点作为发现所有节点的一种方式，并在需要时在每次新连接尝试时刷新活动端点列表。 如果发现过时信息（默认情况下，早于 5 分钟），则会触发刷新。

要使用该驱动程序，请执行以下操作：
* 在连接 URL 或属性池中传递新的连接属性以实现负载平衡。
要在所有服务器之间启用统一负载平衡，请在 URL 中将 load-balance 属性设置为 true，如下例所示：
```
String bmurl = "jdbc:brightdb://127.0.0.1:2521/bigmath?user=bigmath&password=bigmath&load-balance=true";
DriverManager.getConnection(bmurl);
```

要在初始连接期间提供备用主机以防第一个地址失败，请在连接字符串中指定多个主机，如下所示：
```
String bmurl = "jdbc:brightdb://127.0.0.1:2521,127.0.0.2:2521,127.0.0.3:2521/bigmath?user=bigmath&password=bigmath&load-balance=true";
DriverManager.getConnection(bmurl);
```

驱动程序建立初始连接后，它会从 Universe 中获取可用服务器的列表，并在这些服务器之间执行后续连接请求的负载平衡。

要指定topology-keys，请将topology-keys属性设置为逗号分隔值，如下例所示：
```
String bmurl = "jdbc:brightdb://127.0.0.1:2521/bigmath?user=bigmath&password=bigmath&load-balance=true&topology-keys=cloud1.region1.zone1,cloud1.region1.zone2";
DriverManager.getConnection(bmurl);
```

* 配置 BMClusterAwareDataSource 以实现统一负载平衡，然后使用它来创建连接，如下例所示：
```
String jdbcUrl = "jdbc:brightdb://127.0.0.1:2521/bigmath";
BMClusterAwareDataSource ds = new BMClusterAwareDataSource();
ds.setUrl(jdbcUrl);
// Set topology keys to enable topology-aware distribution
ds.setTopologyKeys("cloud1.region1.zone1,cloud1.region2.zone2");
// Provide more end points to prevent first connection failure
// if an initial contact point is not available
ds.setAdditionalEndpoints("127.0.0.2:2521,127.0.0.3:2521");
 
Connection conn = ds.getConnection();
```

* 使用池解决方案（例如 Hikari）配置 BMClusterAwareDataSource，然后使用它来创建连接，如下例所示：
```
Properties poolProperties = new Properties();
poolProperties.setProperty("dataSourceClassName", "com.bigmath.bsql.BMClusterAwareDataSource");
poolProperties.setProperty("maximumPoolSize", 10);
poolProperties.setProperty("dataSource.serverName", "127.0.0.1");
poolProperties.setProperty("dataSource.portNumber", "2521");
poolProperties.setProperty("dataSource.databaseName", "bigmath");
poolProperties.setProperty("dataSource.user", "bigmath");
poolProperties.setProperty("dataSource.password", "bigmath");
// If you want to provide additional end points
String additionalEndpoints = "127.0.0.2:2521,127.0.0.3:2521,127.0.0.4:2521,127.0.0.5:2521";
poolProperties.setProperty("dataSource.additionalEndpoints", additionalEndpoints);
// If you want to load balance between specific geo locations using topology keys
String geoLocations = "cloud1.region1.zone1,cloud1.region2.zone2";
poolProperties.setProperty("dataSource.topologyKeys", geoLocations);
 
poolProperties.setProperty("poolName", name);
 
HikariConfig config = new HikariConfig(poolProperties);
config.validate();
HikariDataSource ds = new HikariDataSource(config);
 
Connection conn = ds.getConnection();
```

## **示例**
本教程展示如何将 AiSQL JDBC 驱动程序与 AiSQL 结合使用。 首先创建一个复制因子为 3 的三节点集群。本教程使用 bm-dev-ctl 实用程序。

接下来，您使用 bm-sample-apps 演示驱动程序的负载平衡功能并创建 Maven 项目以了解如何在应用程序中使用该驱动程序。

注：该驱动程序需要 AiSQL 版本 2.7.2.0 或更高版本以及 Java 8 或更高版本。

**1.安装AiSQL并创建本地集群**
创建一个包含 3 节点 RF-3 集群的 Universe，并分配一些虚构的地理位置。 使用的放置值只是令牌，与实际的 AWS 云区域和区域无关。
```
$ cd <path-to-brightdb-installation>
 
./bin/bm-dev-ctl create --rf 3 --placement_info "aws.us-west.us-west-2a,aws.us-west.us-west-2a,aws.us-west.us-west-2b"
```

**2.使用 bm-sample-apps 检查均匀负载平衡**
下载 bm-sample-apps JAR 文件。
```
wget [https://github.com/bigmath/bm-sample-apps/releases/download/v1.4.0/bm-sample-apps.jar](https://github.com/yugabyte/yb-sample-apps/releases/download/v1.4.0/yb-sample-apps.jar)
```

运行 SqlInserts 工作负载应用程序，该应用程序创建多个线程，对应用程序创建的示例表执行读取和写入操作。 默认情况下，在 bm-sample-apps 的所有 Sql* 工作负载（包括 SqlInserts）中启用统一负载平衡。
```
java -jar bm-sample-apps.jar  \
      --workload SqlInserts \
      --num_threads_read 15 --num_threads_write 15 \
      --nodes 127.0.0.1:2521,127.0.0.2:2521,127.0.0.3:2521
```

该应用程序创建 30 个连接，每个读取器和写入器线程 1 个连接。 要验证行为，请等待应用程序创建连接，然后从浏览器中访问每个节点的 http://<host>:8100/rpcz，以查看连接在节点之间均匀分布。

此 URL 提供一个连接列表，其中列表的每个元素都有一些有关连接的信息，如以下屏幕截图所示。 您可以计算该列表中的连接数，或搜索该网页上主机关键字的出现次数。 每个节点应该有 10 个连接。
![](./media/chapter3/32.png)
**3.使用 bm-sample-apps 检查拓扑感知负载平衡**
对于拓扑感知负载平衡，运行 SqlInserts 工作负载应用程序，并将 topology-keys1 属性设置为 aws.us-west.us-west-2a； 在这种情况下仅使用两个节点。
```
java -jar bm-sample-apps.jar \
      --workload SqlInserts \
      --nodes 127.0.0.1:2521,127.0.0.2:2521,127.0.0.3:2521 \
      --num_threads_read 15 --num_threads_write 15 \
      --topology_keys aws.us-west.us-west-2a
```

要验证行为，请等待应用程序创建连接，然后导航到 http://<host>:8100/rpcz。 前两个节点应各有 15 个连接，第三个节点应有 0 个连接。

**4.清理**
完成实验后，运行以下命令来销毁本地集群：
```
./bin/bm-dev-ctl destroy
```

## **其他示例**
要访问使用 AiSQL JDBC 驱动程序的示例应用程序，请访问 AiSQL JDBC 驱动程序。

要使用示例，请完成以下步骤：
* 按照快速入门中的说明安装 AiSQL。
* 通过运行 mvn package 构建示例。
* 按照以下准则运行 run.sh 脚本：
```
./run.sh [-v] [-i] -D -<path_to_bigmath_installation>
```

在前面的命令中，替换：
①如果要在 VERBOSE 模式下运行脚本，[-v] [-i] 与 -v 一起使用。
②[-v] [-i] 如果要在交互模式下运行脚本，请与 -i 一起使用。
③[-v] [-i] 与 -v -i 如果您想同时在详细模式和交互模式下运行脚本。
④<path_to_bigmath_installation> 为您安装 AiSQL 的目录的路径。

以下是运行该脚本的 shell 命令示例：

```
./run.sh -v -i -D ~/bigmath-2.7.2.0/
```

注：该驱动程序需要 AiSQL 版本 2.7.2.0 或更高版本。

运行脚本启动 AiSQL 集群，通过 Java 应用程序演示负载平衡，然后销毁集群。
启动后，该脚本会显示一个包含两个选项的菜单：UniformLoadBalance 和 TopologyAwareLoadBalance。 选择这些选项之一以在后台运行相应的脚本及其 Java 应用程序。

 

 

 
