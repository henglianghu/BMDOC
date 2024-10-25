## **BCQL**

### **介绍**

BCQL中有两种类型的生存时间（TTL）：

* 未存储在CoreDB中的表级TTL。相反，它作为表模式的一部分存储在mserver系统目录中。
* 非表级TTL，包括与列的值和行级TTL一起存储的列级TTL。

如果列的值处不存在TTL，则表级别的TTL将用作默认值。 

此外，BCQL还区分了使用INSERT和UPDATE语句创建的行。这种差异以及行级TTL是使用所谓的liveness 列来跟踪的，这是一种对您不可见的特殊系统列。它是为插入而添加的，而不是更新，确保即使只有在插入的情况下才删除所有非主键列，也能显示行。 

### **表级TTL**

BCQL允许在表级别指定TTL属性，在这种情况下，您不会在CoreDB中按键值存储TTL。相反，TTL在读取和压缩期间被隐式地强制执行，以回收空间。

可以使用default_time_to_live属性定义表级TTL。

### **行级TTL**

BCQL允许在每个INSERT和UPDATE操作的级别指定TTL属性。
行级TTL将使整行过期。该值是在使用USING TTL子句执行INSERT或UPDATE操作期间指定的。在这种情况下，TTL被存储为CoreDB值的一部分。

请考虑以下查询： 

```
INSERT INTO pageviews(path) VALUES ('/index') USING TTL 10;
SELECT * FROM pageviews;
 path   | views
--------+-------
 /index |  null
 
(1 rows)
```

10秒钟后，该行过期，如下所示： 

```
SELECT * FROM pageviews;
 path | views
------+-------
 
(0 rows)
```

### **列级TTL**

BCQL还允许您设置列级TTL，在这种情况下，TTL存储为CoreDB列值的一部分，您只能在更新列时设置该值，如以下示例所示：

```
INSERT INTO pageviews(path,views) VALUES ('/index', 10);
 
SELECT * FROM pageviews;
 path   | views
--------+-------
 /index |  10
 
(1 rows)
UPDATE pageviews USING TTL 10 SET views=10 WHERE path='/index';
```

10秒钟后，查询行将导致视图列返回NULL，但该行仍将存在，如以下示例所示： 

```
SELECT * FROM pageviews;
 path   | views
--------+-------
 /index |  null
 
(1 rows)
```

### **TTL的有效数据过期**

BCQL包括一个文件过期功能，该功能针对主要依赖表级TTL设置（或具有一致的行或列级TTL值）的工作负载进行了优化。此功能减少了CPU使用量和空间放大，特别是对于经常使用表级TTL将数据集保存到特定大小的时间序列工作负载。这是通过按时间将数据组织成文件来实现的，类似于Cassandra的时间窗口压缩策略。

#### 新BCQL数据集配置

如果为时间序列数据集配置新的BCQL数据库并使用默认生存时间，建议使用以下BM - dbserver标志配置： 
--tile_enable_ttl_file_filter = true
允许直接删除过期文件，而不是在压缩过程中依赖垃圾收集。

--rocksdb_max_file_size_for_compaction = [the amount of data to be deleted at once, in bytes]

此标志的值取决于预期一次删除的数据量。例如，如果表的TTL为90天，则可能需要同时删除三天的数据。在这种情况下，rocksdb_max_file_size_for_compaction应设置为预计在3天内生成的数据量。超过此大小的文件将被排除在正常压缩之外，从而增加CPU。

请注意，在创建的文件数量和读取性能之间存在一些折衷。一个合理的经验法则是配置标志，使30到50个文件存储完整的数据集（例如，90天除以3天就是30个文件）。使用此功能的CPU优势应该足以弥补任何读取性能损失。

--sst_files_soft_limit = [number of expected files at steady state + 20]
--sst_files_hard_limit = [number of expected files at steady state + 40]

这些标志的值取决于预期在稳定状态下保存完整数据集的文件数。如果每个分片的文件数超过其值，这些标志将限制对BCQL的写入。因此，它们的设置方式需要能够适应数据集稳定状态下预期的文件数量。在上面的示例中，30个文件应该包含90天的数据。在这种情况下，sst_files_soft_limit将被设置为50，sst_files_hard_limit将设置为70。

在某些新数据集的情况下，在打开应用程序之前，新数据将被回填到数据库中。此回填的数据可能具有与之相关联的值级别TTL，该值级别明显低于表上的default_time_to_live属性，所需效果是该数据将在表TTL允许的时间之前被删除。默认情况下，此类数据不会提前过期。但是，file_expiration_value_ttl_overrides_table_ttl标志可用于忽略表TTL 并仅根据值TTL 过期。 

警告：
使用file_expiration_value_ttl_overrides_table_ttl标志时，请确保在值级别为TTL 的所有数据（例如，回填的数据）完全过期之前将该标志设置回false。否则可能会导致数据意外丢失。例如，如果default_time_to_live为90天，并且数据已用值级别TTL从1天回填到89天，则重要的是在数据分析后89天内将file_expiration_value_ttl_overrides_table_ttl 标志设置回false，以避免数据丢失。 

#### 现有BCQL数据集配置

要将现有的BCQL表转换为针对文件过期配置的表，可以使用与上面相同的dbserver标志值。然而，在这种情况下，应预期数据的临时2倍空间放大。这种放大是因为现有的文件结构将把大部分数据保存在一个大文件中，而该文件现在将被排除在压缩之外。因此，该文件将保持不变，直到其内容完全过期，大约是配置文件过期功能后的TTL时间量。
file_expiration_ignore_value_ttl标志可以设置为true以忽略丢失的元数据。这将忽略行和列级别的TTL元数据，使文件完全基于表的default_time_to_live过期。 

警告
为了防止早期数据删除，非常重要的是，在这些情况下，任何具有TTL的表的default_time_to_live 都应设置为大于或等于这些表中包含的最大值级别TTL。建议删除缺少元数据的文件后，将file_expiration_ignore_value_ttl标志设置回false（无需重新启动）。 

#### 最佳做法和故障排除

* 文件过期功能仅对具有默认生存时间的表启用。即使是在插入时明确设置TTL的应用程序也应该配置默认生存时间。
* 文件过期功能假设数据是按照其预期过期时间的大致时间顺序到达的。如果不满足这一假设，该功能可以安全使用，但效果会显著降低。
* 文件以保守的方式过期，只有在其保存的每个数据项完全过期后才会被删除。如果一个文件同时具有表级TTL和列级TTL，则在确定过期时使用两者中较晚的一个。 
* 如果插入的数据项具有不合理的高TTL（或无TTL），则文件过期功能将停止垃圾收集数据。在这些情况下，可能需要将file_expiration_ignore_value_ttl标志设置为true，这可能会导致不必要的数据丢失。

### **TTL相关命令和功能**

使用TTL有几种方法： 

1. 具有default_time_to_live属性的表级TTL。
2. 具有TTL的过期的行。 
3. TTL函数返回到过期的秒数。
4. WriteTime函数在插入行或列时返回时间戳
5. 可更新行或列的TTL
6. bm-dbserver的[基于TTL的文件过期标志](#_基于TTL的文件过期标志)配置
