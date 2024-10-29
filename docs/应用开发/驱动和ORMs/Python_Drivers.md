AiSQL Psycopg2 智能驱动程序是基于 PostgreSQL psycopg2 驱动程序构建的 BSQL Python 驱动程序，具有附加的连接负载平衡功能。

## **下载驱动依赖**
构建 Psycopg2 需要一些先决条件（C 编译器和一些开发包）。 查看安装说明和常见问题解答以了解详细信息。

AiSQL Psycopg2 驱动程序需要 PostgreSQL 版本 12 或更高版本（最好是 14）。

如果满足先决条件，您可以像任何其他 Python 包一样安装 psycopg2-brightdb，使用 pip 从 PyPI 下载它：
```
$ pip install psycopg2-brightdb
```

或者，如果您已将源码包下载到本地，则可以使用 setup.py 脚本：
```
$ python setup.py build
$ sudo python setup.py install
```

## **基础知识**
了解如何使用 AiSQL Psycopg2 智能驱动程序执行 Python 应用程序开发所需的常见任务。

**1.负载平衡连接属性**
需要添加以下连接属性以启用负载平衡：
* load_balance - 通过将此属性设置为 true 来启用集群感知负载平衡； 默认禁用。
* topology_keys - 提供以逗号分隔的地理位置值以启用拓扑感知负载平衡。 地理位置可以作为 cloud.region.zone 提供。

**2.使用驱动程序**
要使用驱动程序，请在连接字符串或字典中传递新的连接属性以实现负载平衡。

要在所有服务器之间启用统一负载平衡，请在连接字符串或字典中将 load-balance 属性设置为 true，如下例所示：

（1）连接字符串
```
conn = psycopg2.connect("dbname=database_name host=hostname port=2521 user=username password=password load_balance=true")
```

（2）连接字典
```
conn = psycopg2.connect(user = 'username', password='password', host = 'hostname', port = '2521', dbname = 'database_name', load_balance='True')
```

您可以在连接字符串中指定多个主机，以防主地址失败。 驱动程序建立初始连接后，它会从集群中获取可用服务器列表，并在这些服务器之间对后续连接请求进行负载平衡。

要指定topology_keys，请将 topology_keys 属性设置为连接字符串或字典中的逗号分隔值，如下例所示：

（1）连接字符串
```
conn = psycopg2.connect("dbname=database_name host=hostname port=2521 user=username password=password load_balance=true topology_keys=cloud.region.zone1,cloud.region.zone2")
```

（2）连接字典
```
conn = psycopg2.connect(user = 'username', password='password', host = 'hostname', port = '2521', dbname = 'database_name', load_balance='True', topology_keys='cloud.region.zone1,cloud.region.zone2')
```


要配置 SimpleConnectionPool，请指定负载平衡，如下所示：
```
bm_pool = psycopg2.pool.SimpleConnectionPool(1, 10, user="bigmath",
                                                        password="bigmath",
                                                        host="127.0.0.1",
                                                        port="2521",
                                                        database="bigmath",
                                                        load_balance="True")
conn = bm_pool.getconn()
```

## **示例**
本教程展示如何将 AiSQL Psycopg2 驱动程序与 AiSQL 结合使用。 首先创建一个复制因子为 3 的 3 节点集群。本教程使用 bm-dev-ctl 实用程序。

接下来，您使用 Python shell 终端通过运行一些 python 脚本来演示驱动程序的负载平衡功能。

注：该驱动程序需要 AiSQL 版本 2.7.2.0 或更高版本。

**1.创建本地集群**
创建一个包含 3 节点 RF-3 集群的 Universe，并分配一些虚构的地理位置。 使用的放置值只是令牌，与实际的 AWS 云区域和区域无关。
```
cd <path-to-brightdb-installation>
./bin/bm-dev-ctl create --rf 3 --placement_info "aws.us-west.us-west-2a,aws.us-west.us-west-2a,aws.us-west.us-west-2b"
```

**2.检查均匀负载平衡**
登录到 Python 终端并运行以下脚本：
```
import psycopg2
conns = []
for i in range(30):
    conn = psycopg2.connect(user = 'username', password='xxx', host = 'hostname', port = '2521', dbname = 'database_name', load_balance='True')
    conns.append(conn)
```

该应用程序创建 30 个连接。 要验证行为，请等待应用程序创建连接，然后从浏览器中访问每个节点的 http://<host>:8100/rpcz，以查看连接在节点之间均匀分布。 此 URL 提供一个连接列表，其中列表的每个元素都有一些有关连接的信息，如以下屏幕截图所示。 您可以计算该列表中的连接数，或搜索该网页上主机关键字的出现次数。 每个节点应该有 10 个连接。
![](../../assets/chapter3/33.png)
您还可以通过在同一终端中运行以下脚本来验证连接数：
```
from psycopg2.policies import ClusterAwareLoadBalancer as lb
obj = lb()
obj.printHostToConnMap()
```

这将显示一个键值对映射，其中键是主机，值是主机上的连接数。 （这是从客户端角度来看的连接数。）

使用 bm-sample-apps 检查拓扑感知负载平衡
在新的 Python 终端中运行以下脚本，并将 topology_keys 属性设置为 aws.us-west.us-west-2a； 在这种情况下仅使用两个节点。
```
import psycopg2
conns = []
for i in range(30):
    conn = psycopg2.connect(user = 'username', password='xxx', host = 'hostname', port = '2521', dbname = 'database_name', load_balance='True', topology_keys='aws.us-west.us-west-2a')
    conns.append(conn)
```

要验证行为，请等待应用程序创建连接，然后导航到 http://<host>:8100/rpcz。 前两个节点应各有 15 个连接，第三个节点应有 0 个连接。 您还可以通过在同一终端中运行之前的验证脚本来验证这一点。

**3.清理**
完成实验后，运行以下命令来销毁本地集群：
```
./bin/bm-dev-ctl destroy 
```