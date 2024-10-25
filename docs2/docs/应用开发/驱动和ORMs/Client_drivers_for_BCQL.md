

支持以下客户端驱动程序与 AiSQL 云查询语言 (BCQL) API 一起使用，AiSQL 云查询语言 (BCQL) API 是一种基于 SQL 的半关系 API，源于 Apache Cassandra 查询语言 (CQL)。

有关使用以下客户端驱动程序构建示例应用程序的教程，请单击下面包含的每个驱动程序的相关链接。

使用 AiSQL 客户端驱动程序
您应该始终使用 AiSQL BCQL 客户端驱动程序。 使用通用 Cassandra 驱动程序可能会导致错误和性能问题。

## **C/C++**
适用于 BCQL 的 AiSQL C/C++ 驱动程序
适用于 BCQL 的 AiSQL C++ 驱动程序基于适用于 Apache Cassandra 的 DataStax C++ 驱动程序。

有关详细信息，请参阅 GitHub 存储库中的自述文件。
有关使用此驱动程序构建示例 C++ 应用程序的教程，请参阅构建 C++ 应用程序。

## **C＃**
适用于 BCQL 的 AiSQL C# 驱动程序
BCQL 的 AiSQL C# 驱动程序基于 Apache Cassandra 的 DataStax C# 驱动程序的分支。

有关详细信息，请参阅 GitHub 存储库中的自述文件。
有关使用此驱动程序构建示例 C# 应用程序的教程，请参阅连接应用程序。

## **Go**
BCQL 的 AiSQL Go 驱动程序
BCQL 的 AiSQL Go 驱动程序基于 GoCQL 的一个分支。

有关详细信息，请参阅 GitHub 存储库中的自述文件。
有关使用此驱动程序构建示例 Go 应用程序的教程，请参阅连接应用程序。

## **Java**
**适用于 BCQL 3.10 的 AiSQL Java 驱动程序**
BCQL 的 AiSQL Java 驱动程序版本 3.10.0-bm-x 基于 Apache Cassandra v.3.10 的 DataStax Java 驱动程序，并且需要如下所示的 Maven 依赖项。

有关详细信息，请参阅 GitHub 存储库中的 v3.10 自述文件。
有关使用此驱动程序构建示例 Java 应用程序的教程，请参阅连接应用程序。

要使用此驱动程序构建 Java 应用程序，您必须将以下 Maven 依赖项添加到您的应用程序：
```
<dependency>
  <groupId>com.bigmath</groupId>
  <artifactId>cassandra-driver-core</artifactId>
  <version>3.10.3-bm-2</version>
</dependency>
```

有关详细信息，请参阅 Maven repository contents。

**适用于 BCQL 4.15 的 AiSQL Java 驱动程序**
BCQL 的 AiSQL Java 驱动程序版本 4.15.0-bm-1 基于 Apache Cassandra (v4.15) 的 DataStax Java 驱动程序，并且需要如下所示的 Maven 依赖项。

有关详细信息，请参阅 GitHub 存储库中的 v4.15 自述文件。
有关使用此驱动程序构建示例 Java 应用程序的教程，请参阅连接应用程序。

要使用此驱动程序构建 Java 应用程序，您必须将以下 Maven 依赖项添加到您的应用程序：
```
<dependency>
  <groupId>com.bigmath</groupId>
  <artifactId>java-driver-core</artifactId>
  <version>4.15.0-bm-1</version>
</dependency>
```

有关详细信息，请参阅Maven repository contents。

## **Node.js**
适用于 BCQL 的 AiSQL Node.js 驱动程序
BCQL 的 AiSQL Node.js 驱动程序基于 Apache Cassandra 的 DataStax Node.js 驱动程序的分支。

有关详细信息，请参阅 GitHub 存储库中的自述文件。
有关使用此驱动程序构建示例 Node.js 应用程序的教程，请参阅连接应用程序。

## **Python**
适用于 BCQL 的 AiSQL Python 驱动程序
BCQL 的 AiSQL Python 驱动程序基于 Apache Cassandra 的 DataStax Python 驱动程序的分支。

有关详细信息，请参阅 GitHub 存储库中的自述文件。
有关使用此驱动程序构建示例 Python 应用程序的教程，请参阅连接应用程序。

## **Ruby**
适用于 BCQL 的 AiSQL Ruby 驱动程序
BCQL 的 AiSQL Ruby 驱动程序基于 Apache Cassandra 的 DataStax Ruby 驱动程序的分支。

有关详细信息，请参阅 GitHub 存储库中的自述文件。
有关使用此驱动程序构建示例 Ruby 应用程序的教程，请参阅连接应用程序。

## **Scala**
适用于 BCQL 的 AiSQL Java 驱动程序
BCQL 的 AiSQL Java 驱动程序基于 Apache Cassandra 的 DataStax Java 驱动程序的分支，当您将如下所示的 sbt（Scala 构建工具）依赖项添加到应用程序时，可用于构建 Scala 应用程序。

有关详细信息，请参阅 GitHub 存储库中的自述文件。

要使用 BCQL 的 AiSQL Java 驱动程序构建 Scala 应用程序，您必须将以下 sbt 依赖项添加到您的应用程序：
```
libraryDependencies += "com.bigmath" % "cassandra-driver-core" % "3.8.0-bm-5"
```

有关使用此驱动程序构建示例 Scala 应用程序的教程，请参阅连接应用程序。

 