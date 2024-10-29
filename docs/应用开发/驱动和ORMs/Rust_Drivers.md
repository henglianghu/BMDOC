

AiSQL Rust 智能驱动程序是基于 rust-postgres 的 BSQL Rust 驱动程序，具有附加的连接负载平衡功能。
Rust 智能驱动程序提供了两种与 rust-postgres 类似的不同客户端：
* bm-postgres：基于 postgres 的本机同步 AiSQL BSQL 客户端。
* bm-tokio-postgres：基于 tokio-postgres 的本机异步 AiSQL BSQL 客户端.

## **导入驱动程序依赖项**
您可以通过在 Rust 应用程序的 Cargo.toml 文件中添加以下语句来使用 AiSQL Rust 驱动程序包。
```
# For bm-postgres
bm-postgres = "0.19.7-bm-1-beta"
 
# For bm-tokio-postgres
bm-tokio-postgres = "0.7.10-bm-1-beta"
```

或者，从项目目录运行以下命令：
```
# For bm-postgres
$ cargo add bm-postgres
 
# For bm-tokio-postgres
$ cargo add bm-tokio-postgres
```

## **基础知识**
了解如何使用 AiSQL Rust 智能驱动程序执行 Rust 应用程序开发所需的常见任务。

**1.负载平衡连接属性**
需要添加以下连接属性以启用负载平衡：
* load_balance：通过将此属性设置为 true 来启用集群感知负载平衡； 默认禁用。
* topology_keys：提供以逗号分隔的地理位置值以启用拓扑感知负载平衡。 地理位置可以作为 cloud.region.zone 提供。 将区域中的所有区域指定为 cloud.region.*。 要在主要位置无法访问时指定后备位置，请以 :n 的形式指定优先级，其中 n 是优先顺序。 例如，cloud1.datacenter1.rack1:1、cloud1.datacenter1.rack2:2。

默认情况下，驱动程序每 300 秒（5 分钟）刷新一次节点列表。 您可以通过包含 bm_servers_refresh_interval 参数来更改此值。

以下是 Rust 智能驱动程序提供的其他连接属性：
* Fallback_to_topology_keys_only：当设置为 true 时，智能驱动程序不会尝试连接到 topology_keys 属性指定的主要和后备位置之外的服务器。 默认情况下，驱动程序会回退到集群中的任何可用服务器。 默认为 false。
* failed_host_reconnect_delay_secs：当驱动程序无法连接到服务器时，它使用时间戳标记服务器。 当通过 bm_servers() 刷新服务器列表时，如果响应中出现故障服务器，则仅当服务器被标记为关闭后经过了 failed_host_reconnect_delay_secs 时间后，驱动程序才会将服务器标记为“启动”。 默认值为 5 秒。

**2.使用驱动程序**
要使用该驱动程序，请在连接字符串中传递新的连接属性以实现负载平衡。
要在所有服务器之间启用统一负载平衡，请在连接字符串中将 load-balance 属性设置为 true，如下例所示：
```
let url: String = String::from( "postgresql://localhost:5434/bigmath?user=bigmath&password=bigmath&load_balance=true", );
let conn = bm_postgres::Client::connect(&connection_url,NoTls,)?;
```

您可以在连接字符串中指定多个主机作为后备，以防主地址在初始连接尝试期间失败。 驱动程序建立初始连接后，它会从集群中获取可用服务器列表，并在这些服务器之间平衡后续连接请求。

要指定topology_keys，请将 topology_keys 属性设置为连接字符串或字典中的逗号分隔值，如下例所示：
```
let url: String = String::from( "postgresql://localhost:5434/bigmath?user=bigmath&password=bigmath&load_balance=true&topology_keys=cloud1.datacenter1.rack2", );
let conn = bm_postgres::Client::connect(&connection_url,NoTls,)?;
```

## **示例**
本教程展示如何将异步 bm-tokio-postgres 客户端与 AiSQL 结合使用。 首先创建一个复制因子为 3 的三节点集群。本教程使用 bm-ctl 实用程序。

接下来，您使用 Rust 应用程序来演示驱动程序的负载平衡功能。

有关使用同步 bm-postgres 客户端的示例，请参阅连接应用程序。

**1.创建本地集群**
创建一个包含 3 节点 RF-3 集群的 Universe，并分配一些虚构的地理位置。 将两个节点放置在一个位置，将第三个节点放置在单独的位置。 使用的放置值只是令牌，与实际的 AWS 云区域和区域无关。
```
cd <path-to-brightdb-installation>
```

要创建多可用区集群，请执行以下操作：
（1）通过运行 bm-ctl start 命令启动第一个节点，传入 --cloud_location 和 --fault_tolerance 标志以设置节点位置详细信息，如下所示：
```
./bin/bm-ctl start --advertise_address=127.0.0.1 \
    --base_dir=$HOME/bigmath-2.20.1.3/node1 \
    --cloud_location=aws.us-east-1.us-east-1a \
    --fault_tolerance=zone
```

（2）使用 --join 标志在两个单独的 VM 上启动第二个和第三个节点，如下所示：
```
./bin/bm-ctl start --advertise_address=127.0.0.2 \
    --join=127.0.0.1 \
    --base_dir=$HOME/bigmath-2.20.1.3/node2 \
    --cloud_location=aws.us-east-1.us-east-1a \
    --fault_tolerance=zone
./bin/bm-ctl start --advertise_address=127.0.0.3 \
    --join=127.0.0.1 \
    --base_dir=$HOME/bigmath-2.20.1.3/node3 \
    --cloud_location=aws.us-east-1.us-east-1b \
    --fault_tolerance=zone
```

**2.检查均匀负载平衡**
要检查均匀负载平衡，请执行以下操作：

（1）使用以下命令创建 Rust 项目：
```
cargo new try-it-out
```

这将创建项目“try-it-out”，其中包含 Cargo.toml 文件（项目元数据）和包含主代码文件 main.rs 的 src 目录。

（2）在 Cargo.toml 文件中添加 bm-tokio-postgres = "0.7.10-bm-1-beta" 依赖项，如下所示：
```
[package]
name = "try-it-out"
version = "0.1.0"
edition = "2021"
 
# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
 
[dependencies]
bm-tokio-postgres = "0.7.10-bm-1-beta"
```

（3）将文件 src/main.rs 中的现有代码替换为以下代码：
```
use isahc::ReadResponseExt;
use tokio::task::JoinHandle;
use bm_tokio_postgres::{Client, Error, NoTls};
 
#[tokio::main]
async fn main() -> Result<(), Error> {
   println!("Starting the example ...");
 
   let url: String = String::from(
   "postgresql://127.0.0.1:2521/bigmath?user=bigmath&password=bigmath&load_balance=true",
   );
   println!("Using connection url: {}", url);
   let (_clients, _connections) = createconn(30, url).await.unwrap();
   //Check and print number of connections on each node.
   num_of_connections();
 
   println!("End of Example");
   Ok(())
}
 
async fn createconn(
   numconn: usize,
   url: String,
) -> Result<(Vec<Client>, Vec<JoinHandle<()>>), Error> {
   let mut connectionstored: Vec<JoinHandle<()>> = Vec::with_capacity(numconn);
   let mut clientstored: Vec<Client> = Vec::with_capacity(numconn);
   for _i in 0..numconn {
       let connectionresult = createconnection(url.clone()).await;
       match connectionresult {
          Err(error) => return Err(error),
          Ok((connection, client)) => {
              clientstored.push(client);
              connectionstored.push(connection);
          }
       }
   }
   return Ok((clientstored, connectionstored));
}
 
async fn createconnection(url: String) -> Result<(tokio::task::JoinHandle<()>, Client), Error> {
   let (client, connection) = bm_tokio_postgres::connect(&url, NoTls).await?;
 
   // The connection object performs the actual communication with the database,
   // so spawn it off to run on its own.
   let handle = tokio::spawn(async move {
       if let Err(e) = connection.await {
           eprintln!("connection error: {}", e);
       }
   });
 
   Ok((handle, client))
}
 
pub(crate) fn num_of_connections() {
   for i in 1..4 {
       let url = "http://127.0.0.".to_owned() + &i.to_string() + ":8100/rpcz";
       let response = isahc::get(url);
       if response.is_err() {
           println!("127.0.0.{} = {}",i, 0);
       } else {
           let body = response.unwrap().text().unwrap();
           let c = body.matches("client backend").count();
           println!("127.0.0.{} = {}", i, c);
       }
   }
}
```

（4）运行示例：
```
cargo run
```

该应用程序创建 30 个连接并显示一个键值对映射，其中键是主机，值是主机上的连接数。 （应用程序从每个节点的 http://<host>:8100/rpcz 获取连接数。此 URL 提供一个连接列表，其中列表的每个元素都有一些有关连接的信息。）每个节点应有 10 个 连接。

**3.检查拓扑感知负载平衡**
对于拓扑感知负载平衡，请运行应用程序并将 topology_keys 属性设置为 aws.us-east-1.us-east-1a。 本例中仅使用两个节点。

将 src/main.rs 中的现有代码替换为以下示例代码：
```
use isahc::ReadResponseExt;
use tokio::task::JoinHandle;
use bm_tokio_postgres::{Client, Error, NoTls};
 
#[tokio::main]
async fn main() -> Result<(), Error> {
   println!("Starting the example ...");
 
   let url: String = String::from(
       "postgresql://127.0.0.1:2521/bigmath?user=bigmath&password=bigmath&load_balance=true&topology_keys=aws.us-east-1.us-east-1a",
   );
   println!("Using connection url: {}", url);
   let (_clients, _connections) = createconn(30, url).await.unwrap();
   //Check and print number of connections on each node.
   num_of_connections();
 
   println!("End of Example");
   Ok(())
}
 
async fn createconn(
   numconn: usize,
   url: String,
) -> Result<(Vec<Client>, Vec<JoinHandle<()>>), Error> {
   let mut connectionstored: Vec<JoinHandle<()>> = Vec::with_capacity(numconn);
   let mut clientstored: Vec<Client> = Vec::with_capacity(numconn);
   for _i in 0..numconn {
       let connectionresult = createconnection(url.clone()).await;
       match connectionresult {
           Err(error) => return Err(error),
           Ok((connection, client)) => {
               clientstored.push(client);
               connectionstored.push(connection);
           }
       }
   }
   return Ok((clientstored, connectionstored));
}
 
async fn createconnection(url: String) -> Result<(tokio::task::JoinHandle<()>, Client), Error> {
   let (client, connection) = bm_tokio_postgres::connect(&url, NoTls).await?;
 
   // The connection object performs the actual communication with the database,
   // so spawn it off to run on its own.
   let handle = tokio::spawn(async move {
       if let Err(e) = connection.await {
           eprintln!("connection error: {}", e);
       }
   });
 
   Ok((handle, client))
}
 
pub(crate) fn num_of_connections() {
   for i in 1..4 {
       let url = "http://127.0.0.".to_owned() + &i.to_string() + ":8100/rpcz";
       let response = isahc::get(url);
       if response.is_err() {
           println!("127.0.0.{} = {}",i, 0);
       } else {
           let body = response.unwrap().text().unwrap();
           let c = body.matches("client backend").count();
           println!("127.0.0.{} = {}", i, c);
       }
   }
}
```

在这种情况下，前两个节点应各有 15 个连接，第三个节点应有 0 个连接。

**4.清理**
完成实验后，运行以下命令来销毁本地集群：
```
./bin/bm-ctl destroy --base_dir=$HOME/bigmath-2.20.1.3/node1
./bin/bm-ctl destroy --base_dir=$HOME/bigmath-2.20.1.3/node2
./bin/bm-ctl destroy --base_dir=$HOME/bigmath-2.20.1.3/node3
```

## **配置 SSL/TLS**
AiSQL Rust 智能驱动程序对 SSL 的支持与上游驱动程序相同。

下表描述了使用 SSL 时 AiSQL Rust 智能驱动程序需要作为连接字符串一部分的附加参数。
| 参数    | 描述    | 默认值 |
| ------- | ------- | ------ |
| Sslmode | SSL模式 | prefer |

rust-postgres 驱动程序支持以下 SSL 模式。
disable：不使用 TLS。
prefer (default)：如果可用，请使用 TLS，否则不使用。
Require：需要使用 TLS。

目前，rust-postgres 驱动程序和 AiSQL Rust 智能驱动程序不支持 verify-full 或 verify-ca SSL 模式。

AiSQL Managed 需要 SSL/TLS，使用 SSL 模式禁用的连接将失败。

以下是连接到启用了 SSL 加密的 AiSQL 集群的示例连接 URL：
```
"postgresql://127.0.0.1:5434/bigmath?user=bigmath&password=bigmath&load_balance=true&sslmode=require"
```

如果您在 AiSQL Managed 上创建了集群，请使用集群凭据并下载 SSL 根证书。

以下是连接到启用了 SSL 的 AiSQL 集群的示例应用程序：

执行前在 Cargo.toml 文件中添加 bm-postgres-openssl = "0.5.0-bm-1"、bm-postgres = "0.19.7-bm-1-beta" 和 openssl = "0.10.61" 依赖项 应用程序。
```
use openssl::ssl::{SslConnector, SslMethod};
use bm_postgres_openssl::MakeTlsConnector;
use bm_postgres::{Client};
 
fn main()  {
   let mut builder = SslConnector::builder(SslMethod::tls()).expect("unable to create sslconnector builder");
   builder.set_ca_file("/path/to/root/certificate").expect("unable to load root certificate");
   let connector: MakeTlsConnector = MakeTlsConnector::new(builder.build());
 
   let mut connection = Client::connect( "host=? port=2521 dbname=bigmath user=? password=? sslmode=require",
       connector,
       ).expect("failed to create tls bsql connection");
 
   let result = connection.query_one("select 1", &[]).expect("failed to execute select 1 bsql");
   let value: i32 = result.get(0);
   println!("result of query_one call: {}", value);
}
```