## 第一步：搭建基本开发环境

搭建开发环境前，请确保您已经成功在AiSQL Managed或本都部署了AiSQL集群。

此外，如果您使用AiSQL Managed，则需要以下内容来运行示例应用程序：

* 集群CA证书。
* 您的计算机已添加到集群IP允许列表中。
* 如果你的集群部署在VPC中，如果你想从公共IP地址连接你的应用程序，你需要启用公共访问，但如果你使用的是沙盒集群，这并不适用。 

**注意：**为了使用智能驱动程序负载均衡功能，在连接到AiSQL Managed中的集群时，智能驱动的应用程序必须部署在与集群VPC对等的VPC中，有关AiSQL Managed中VPC网络的信息，请参阅VPC网络。
对于从VPC网络外部访问集群的应用程序，请使用PostgreSQL驱动程序；在这种情况下，集群执行负载平衡。从VPC网络外部，使用智能驱动程序的应用程序会自动返回到上溯到驱动程序行为。



### 下载您的集群证书

AiSQL Managed使用TLS 1.2与集群通信，并使用数字证书验证集群的身份。当您从应用程序或客户端连接到集群时，集群CA证书用于验证。
要将证书下载到将要连接到集群的计算机，请执行以下操作：

1. 在AiSQL Managed中，选择您的集群并单击Connect。
2. 单击AiSQL Client Shell，或连接到您的应用程序。
3. 单击下载CA证书将集群root.crt证书下载到您的计算机。



### 将您的计算机添加到集群IP允许列表

对AiSQL 托管集群的访问仅限于您使用IP允许列表钟明确允许的IP地址，要使应用程序能够连接到集群，您需要将计算机的IP地址添加到集群IP允许列表中。

要将您的计算机添加到群集IP允许列表，请执行以下操作：
1. 在AiSQL Managed中，选择您的集群。
2. 单击Add IP Allow List。
3. 单击Create New List and Add to Cluster。
4. 输入允许列表的名称。
5. 单击Detect and add my IP to this list ，添加自己的IP地址。
6. 单击Save。
允许列表最长需要30秒才能激活。



### 启用公共访问

部署在VPC中的集群不会公开公共IP地址，除非您明确启用“公共访问”。如果您的集群位于VPC 中，并且您是从公共IP地址（例如您的计算机）连接的，请在集群设置选项卡上启用公共访问。



### Java程序先决条件

安装Java Development Kit (JDK) 1.8或更高版本。Linux和macOS的JDK安装程序可以从Oracle、Adoptium (OpenJDK)或Azul Systems (OpenJDK)下载，macOS上的Homebrew用户可以使用brew install openjdk进行安装。

安装Apache Maven 3.3或更高版本。 



### 提供连接参数

**注**：如果您的集群在本地运行并在127.0.0.1:2521上侦听，<u>则可以跳过此步骤</u>。

如果您的集群在AiSQL Managed上运行，则需要修改连接参数，以便应用程序可以建立与AiSQL 集群的连接。

如下准备：

1. 打开位于src/main/resources/目录下的app.properties文件。
2. 配置如下的参数：
* **host** - AiSQL 集群的主机名。对于本地集群，请使用默认值（127.0.0.1）。对于AiSQL Managed，请在“Clusters”页面上选择您的集群，然后单击“Settings”。host显示在“Connection Parameters ”下。
* **port** - 驱动使用的端口号 (默认的AiSQL BSQL端口是2521)。
* **database** - 连接的数据库名 (默认是bigmath)。
* **dbUser**和**dbPassword** - AiSQL 数据库的用户名和密码。对于本地集群，使用默认值（bigmath和bigmath）。对于AiSQL Managed，使用您下载的凭据文件中的凭据。
* **sslMode** - 使用的SSL模式，AiSQL Managed需要SSL 连接；使用verify-full
* **sslRootCert** - AiSQL Managed 集群CA 认证的完整路径。
3. 保存此文件。




## 第二步：编写应用程序

从GitHub克隆应用程序，将示例应用程序克隆到您的计算机：

```
git clone https://github.com/bigmath-Samples/bigmath-simple-java-app.git && cd bigmath-simple-java-app
```



## 第三步：运行应用程序

首先构建应用

```
mvn clean package
```

启动应用

```
java -cp target/bigmath-simple-java-app-1.0-SNAPSHOT.jar SampleApp
```

如果您在沙盒或单节点集群上运行应用程序，则驱动程序会显示一条警告，表明负载平衡失败，并将恢复到常规连接。

您应该可以看到类似的如下输出：

```
>>>> Successfully connected to AiSQL!
>>>> Successfully created DemoAccount table.
>>>> Selecting accounts:
name = Jessica, age = 28, country = USA, balance = 10000
name = John, age = 28, country = Canada, balance = 9000
 
>>>> Transferred 800 between accounts.
>>>> Selecting accounts:
name = Jessica, age = 28, country = USA, balance = 9200
name = John, age = 28, country = Canada, balance = 9800
```

至此您已经成功地执行了一个与AiSQL Managed一起工作的基本Java应用程序。