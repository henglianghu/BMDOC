

AiSQL node-postgres 智能驱动程序是基于 PostgreSQL node-postgres 驱动程序构建的用于 BSQL 的 Node.js 驱动程序，具有附加的连接负载平衡功能。

## **下载驱动依赖**
使用以下命令下载并安装 AiSQL node-postgres 智能驱动程序（您需要在系统上安装 Node.js）：
```
npm install @brightdb/pg
```

该驱动程序需要 AiSQL 版本 2.7.2.0 或更高版本。
您可以开始在代码中使用该驱动程序。

## **基础知识**
了解如何使用 AiSQL node-postgres 智能驱动程序执行 Node.js 应用程序开发所需的常见任务。

**1.负载平衡连接属性**
需要添加以下连接属性以启用负载平衡：
* loadBalance：通过将此属性设置为 true 来启用集群感知负载平衡； 默认禁用。
* topologyKeys：提供逗号分隔的地理位置值以启用拓扑感知负载平衡。 地理位置可以作为 cloud.region.zone 提供。 将区域中的所有区域指定为 cloud.region.*。 要在主要位置无法访问时指定后备位置，请以 :n 的形式指定优先级，其中 n 是优先顺序。 例如，cloud1.datacenter1.rack1:1、cloud1.datacenter1.rack2:2。
默认情况下，驱动程序每 300 秒（5 分钟）刷新一次节点列表。 您可以通过包含 bmServersRefreshInterval 参数来更改此值。

**2.使用驱动程序**
要使用该驱动程序，请执行以下操作：
* 在连接 URL 中传递新的连接属性以实现负载平衡。
要在所有服务器之间启用统一负载平衡，请按照以下连接字符串将 URL 中的 loadBalance 属性设置为 true：
```
const connectionString = "postgresql://user:password@localhost:port/database?loadBalance=true"
const client = new Client(connectionString);
client.connect()
```
驱动程序建立初始连接后，它会从 Universe 中获取可用服务器的列表，并对这些服务器之间的后续连接请求执行负载平衡。
* 要指定topologyKeys，请将 topologyKeys 属性设置为逗号分隔值，按照以下连接字符串：
```
const connectionString = "postgresql://user:password@localhost:port/database?loadBalance=true&topologyKeys=cloud1.datacenter1.rack1,cloud1.datacenter1.rack2"
const client = new Client(connectionString);
client.conn
```

* 要使用 Pool 配置最多 100 个连接的基本连接池，请指定负载平衡，如下所示：
```
let pool = new Pool({
    user: 'bigmath',
    password: 'bigmath',
    host: 'localhost',
    port: 2521,
    loadBalance: true,
    database: 'bigmath',
    max: 100
})
```

## **示例**
本教程展示如何将 AiSQL node-postgres 智能驱动程序与 AiSQL 结合使用。 首先创建一个复制因子为 3 的三节点集群。本教程使用 bm-dev-ctl 实用程序。

接下来，您使用 Node.js 应用程序来演示驱动程序的负载平衡功能。

注：该驱动程序需要 AiSQL 版本 2.7.2.0 或更高版本。

**1.创建本地集群**
创建一个包含 3 节点 RF-3 集群的 Universe，并分配一些虚构的地理位置。 使用的放置值只是令牌，与实际的 AWS 云区域和区域无关。
```
cd <path-to-brightdb-installation>
 
./bin/bm-dev-ctl create --rf 3 --placement_info "aws.us-west.us-west-2a,aws.us-west.us-west-2a,aws.us-west.us-west-2b"
```

**2.检查均匀负载平衡**
要检查均匀负载平衡，请执行以下操作：
（1）创建一个 Node.js 文件来运行该示例：
```
touch example.js
```

（2）在 example.js 文件中添加以下代码。
```
const pg = require('@brightdb/pg');
 
async function createConnection(){
    const bmurl = "postgresql://bigmath:bigmath@localhost:2521/bigmath?loadBalance=true"
    let client = new pg.Client(bmurl);
    client.on('error', () => {
        // ignore the error and handle exiting
    })
    await client.connect()
    client.connection.on('error', () => {
        // ignore the error and handle exiting
    })
    return client;
}
 
async function createNumConnections(numConnections) {
    let clientArray = []
    for (let i=0; i<numConnections; i++) {
        if(i&1){
             clientArray.push(await createConnection())
        }else  {
            setTimeout(async() => {
                clientArray.push(await createConnection())
            }, 1000)
        }
    }
    return clientArray
}
 
(async () => {
    let clientArray = []
    let numConnections = 30
    clientArray = await createNumConnections(numConnections)
 
    setTimeout(async () => {
        console.log('Node connection counts after making connections: \n\n \t\t', pg.Client.connectionMap, '\n')
    }, 2000)
 
})();
```

（3）运行示例：
```
node example.js
```

该应用程序创建 30 个连接，并显示一个键值对映射，其中键是主机，值是其上的连接数（这是连接数的客户端视角）。 每个节点应该有 10 个连接。

**3.检查拓扑感知负载平衡**
对于拓扑感知负载平衡，请运行应用程序并将 topologyKeys 属性设置为 aws.us-west.us-west-2a； 在这种情况下将仅使用两个节点。
```
const pg = require('@brightdb/pg');
 
async function createConnection(){
    const bmurl = "postgresql://bigmath:bigmath@localhost:2521/bigmath?loadBalance=true&&topologyKey=aws.us-west.us-west-2a"
    let client = new pg.Client(bmurl);
    client.on('error', () => {
        // ignore the error and handle exiting
    })
    await client.connect()
    client.connection.on('error', () => {
        // ignore the error and handle exiting
    })
    return client;
}
 
async function createNumConnections(numConnections) {
    let clientArray = []
    for (let i=0; i<numConnections; i++) {
        if(i&1){
             clientArray.push(await createConnection())
        }else  {
            setTimeout(async() => {
                clientArray.push(await createConnection())
            }, 1000)
        }
    }
    return clientArray
}
 
(async () => {
    let clientArray = []
    let numConnections = 30
    clientArray = await createNumConnections(numConnections)
 
    setTimeout(async () => {
        console.log('Node connection counts after making connections: \n\n \t\t', pg.Client.connectionMap, '\n')
    }, 2000)
 
})();
```

要验证行为，请等待应用程序创建连接，然后导航到 http://<host>:8100/rpcz。 前两个节点应各有 15 个连接，第三个节点应有 0 个连接。

**4.清理**
完成实验后，运行以下命令来销毁本地集群：
```
./bin/bm-dev-ctl destroy
```
