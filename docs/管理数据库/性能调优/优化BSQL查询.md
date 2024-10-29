BrightDB利用PostgreSQL的pg_hint_plan扩展来控制查询执行计划。
BrightDB使用PostgreSQL的基于成本的优化器，该优化器估计SQL语句的每个可能执行计划的成本。执行成本最低的执行计划。计划者尽其所能选择最佳的执行计划，但并不完美。此外，BrightDB使用的规划器版本不是最优的。例如，基于成本的优化器很幼稚，它假设所有表的行数为1000。然而，行数在计算成本估算中起着至关重要的作用。为了克服这些限制，可以使用pg_hint_plan。
pg_hint_plan使得使用“hints”来调整执行计划成为可能，“hints”是以SQL注释形式来描述。

重新审视你的提示计划
要有效地使用pg_hint_plan，您需要彻底了解如何部署应用程序。当数据库增长或部署更改时，还需要重新访问提示计划，以确保该计划不会限制性能，而是优化性能。

## **配置pg_hint_plan**

pg_hint_plan是预先配置的，默认是启用的。下面的BSQL配置参数控制pg_hint_plan:

| 选项                       | 描述                                                         | 默认值 |
| -------------------------- | ------------------------------------------------------------ | ------ |
| pg_hint_plan.enable_hint   | 打开或关闭pg_hint_plan。                                     | on     |
| pg_hint_plan.debug_print   | 控制调试输出。有效值为off(无调试输出)、on、detailed和verbose。 | off    |
| pg_hint_plan.message_level | 指定调试输出的最小消息级别。按照严重程度递减的顺序，这些级别是:error, warning, notice, info, log和debug。fatal和panic级别的消息总是包含在输出中。 | info   |

**开启pg_hint_plan**
启用pg_hint_plan，命令如下:

```
bigmath=# SET pg_hint_plan.enable_hint=ON;
```

**打开调试输出**
要查看pg_hint_plan使用并转发给查询规划器的特定提示，请打开调试输出。这对于提示短语中出现语法错误或错误提示名称的情况很有帮助。要查看这些调试打印，运行以下命令:

```
bigmath=# SET pg_hint_plan.debug_print TO on;
bigmath=# \set SHOW_CONTEXT always
bigmath=# SET client_min_messages TO log;
```

## **编写提示计划**

pg_hint_plan解析SQL语句中出现的特殊形式的提示短语。这种特殊形式以字符序列/*+开始，以*/结束。提示短语由提示名称后跟用括号括起来并以空格分隔的提示参数组成。
在下面的示例中，选择HashJoin作为连接pg_bench_branches和pg_bench_accounts的连接方法，并使用SeqScan扫描表pgbench_accounts。

```
bigmath=# /*+
bigmath*#    HashJoin(a b)
bigmath*#    SeqScan(a)
bigmath*#  */
bigmath-# EXPLAIN SELECT *
bigmath-#    FROM pgbench_branches b
bigmath-#    JOIN pgbench_accounts a ON b.bid = a.bid
bigmath-#   ORDER BY a.aid;
                                    QUERY PLAN
---------------------------------------------------------------------------------------
 Sort  (cost=31465.84..31715.84 rows=100000 width=197)
   Sort Key: a.aid
   ->  Hash Join  (cost=1.02..4016.02 rows=100000 width=197)
         Hash Cond: (a.bid = b.bid)
         ->  Seq Scan on pgbench_accounts a  (cost=0.00..2640.00 rows=100000 width=97)
         ->  Hash  (cost=1.01..1.01 rows=1 width=100)
               ->  Seq Scan on pgbench_branches b  (cost=0.00..1.01 rows=1 width=100)
(7 rows)
```

## **使用pg_hint_plan**

下面的表和索引定义在下面的例子中被用来说明pg_hint_plan的特性:

```
CREATE TABLE t1 (id int PRIMARY KEY, val int);
CREATE TABLE t2 (id int PRIMARY KEY, val int);
CREATE TABLE t3 (id int PRIMARY KEY, val int);
INSERT INTO t1 SELECT i, i % 100 FROM (SELECT generate_series(1, 10000) i) t;
INSERT INTO t2 SELECT i, i % 10 FROM (SELECT generate_series(1, 1000) i) t;
INSERT INTO t3 SELECT i, i FROM (SELECT generate_series(1, 100) i) t;
CREATE INDEX t1_val ON t1 (val);
CREATE INDEX t2_val ON t2 (val);
CREATE INDEX t3_id1 ON t3 (id);
CREATE INDEX t3_id2 ON t3 (id);
CREATE INDEX t3_id3 ON t3 (id);
CREATE INDEX t3_val ON t3 (val);
```

表的模式如下:

```
 Table "public.t1"
 Column |  Type   | Collation | Nullable | Default
--------+---------+-----------+----------+---------
 id     | integer |           | not null |
 val    | integer |           |          |
Indexes:
    "t1_pkey" PRIMARY KEY, lsm (id HASH)
    "t1_val" lsm (val HASH)
 
                 Table "public.t2"
 Column |  Type   | Collation | Nullable | Default
--------+---------+-----------+----------+---------
 id     | integer |           | not null |
 val    | integer |           |          |
Indexes:
    "t2_pkey" PRIMARY KEY, lsm (id HASH)
    "t2_val" lsm (val HASH)
 
                 Table "public.t3"
 Column |  Type   | Collation | Nullable | Default
--------+---------+-----------+----------+---------
 id     | integer |           | not null |
 val    | integer |           |          |
Indexes:
    "t3_pkey" PRIMARY KEY, lsm (id HASH)
    "t3_id1" lsm (id HASH)
    "t3_id2" lsm (id HASH)
    "t3_id3" lsm (id HASH)
    "t3_val" lsm (val HASH)
```

**扫描方法hints**
扫描方法hints与适当的提示短语一起指定时，对表强制执行扫描方法。应该通过别名指定所需的扫描方法和相应的目标表(如果有的话)。BrightDB支持以下扫描方法，使用来自pg_hint_plan的hints短语:

| 选项                             | 描述                                          |
| -------------------------------- | --------------------------------------------- |
| SeqScan(table)                   | 在表上启用SeqScan                             |
| NoSeqScan(table)                 | 不要在表上启用SeqScan。                       |
| IndexScan(table)                 | 在表上启用IndexScan                           |
| IndexScan(table idx)             | 在索引为idx的表上启用IndexScan                |
| NoIndexScan(table)               | 不要在表上启用IndexScan。                     |
| IndexOnlyScan(table)             | 在表上启用IndexOnlyScan。                     |
| NoIndexOnlyScan(table)           | 不要在表上启用IndexOnlyScan。                 |
| IndexScanRegexp(table regex)     | 用regex定义的正则表达式的表启用索引表达式扫描 |
| IndexOnlyScanRegexp(table regex) | 不要在索引与正则表达式匹配的表上启用索引扫描  |

在下面的例子中，提示/*+SeqScan(t2)*/允许使用SeqScan扫描表t2。

```
/*+SeqScan(t2)*/
EXPLAIN (COSTS false) SELECT * FROM t1, t2 WHERE t1.id = t2.id;
LOG:  pg_hint_plan:
used hint:
SeqScan(t2)
not used hint:
duplication hint:
error hint:
 
              QUERY PLAN
--------------------------------------
 Nested Loop
   ->  Seq Scan on t2
   ->  Index Scan using t1_pkey on t1
         Index Cond: (id = t2.id)
(4 rows)
```

在下面的例子中，由于提示/*+SeqScan(t1)IndexScan(t2)*/， t2使用IndexScan扫描，t1使用SeqScan扫描。

```
/*+SeqScan(t1)IndexScan(t2)*/
EXPLAIN (COSTS false) SELECT * FROM t1, t2 WHERE t1.id = t2.id;
LOG:  pg_hint_plan:
used hint:
SeqScan(t1)
IndexScan(t2)
not used hint:
duplication hint:
error hint:
 
              QUERY PLAN
--------------------------------------
 Nested Loop
   ->  Seq Scan on t1
   ->  Index Scan using t2_pkey on t2
         Index Cond: (id = t1.id)
(4 rows)
```

还可以使用hints短语指示查询规划器不要使用特定类型的扫描。如下例所示，提示/*+NoIndexScan(t1)*/限制了表t1上的IndexScan。

```
/*+NoIndexScan(t1)*/
EXPLAIN (COSTS false) SELECT * FROM t1, t2 WHERE t1.id = t2.id;
LOG:  pg_hint_plan:
used hint:
NoIndexScan(t1)
not used hint:
duplication hint:
error hint:
 
              QUERY PLAN
--------------------------------------
 Nested Loop
   ->  Seq Scan on t1
   ->  Index Scan using t2_pkey on t2
         Index Cond: (id = t1.id)
(4 rows)
```

**在hint短语中指定索引**
单个表可以有多个索引。使用pg_hint_plan，您可以指定在对表执行扫描时要使用的确切索引。以表t3为例。它包含列id的多个索引(t3_pkey, t3_id1, t3_id2, t3_id3)。通过使用适当的hint短语执行SQL查询，您可以选择这些索引中的任何一个来运行扫描。
（1）不带hint短语的查询:

```
EXPLAIN (COSTS false) SELECT * FROM t3 WHERE t3.id = 1;
  QUERY PLAN
--------------------------------
 Index Scan using t3_pkey on t3
   Index Cond: (id = 1)
(2 rows)
```

（2）使用二级索引查询:

```
/*+IndexScan(t3 t3_id2)*/
EXPLAIN (COSTS false) SELECT * FROM t3 WHERE t3.id = 1;
LOG:  available indexes for IndexScan(t3): t3_id2
LOG:  pg_hint_plan:
used hint:
IndexScan(t3 t3_id2)
not used hint:
duplication hint:
error hint:
 
          QUERY PLAN
-------------------------------
 Index Scan using t3_id2 on t3
   Index Cond: (id = 1)
(2 rows)
```

（3）查询返回到SeqScan，因为没有索引可以使用:

```
/*+IndexScan(t3 no_exist)*/
EXPLAIN (COSTS false) SELECT * FROM t3 WHERE t3.id = 1;
LOG:  available indexes for IndexScan(t3):
LOG:  pg_hint_plan:
used hint:
IndexScan(t3 no_exist)
not used hint:
duplication hint:
error hint:
 
     QUERY PLAN
--------------------
 Seq Scan on t3
   Filter: (id = 1)
(2 rows)
```

（4）有选择性索引列表的查询:

```
/*+IndexScan(t3 t3_id1 t3_id2)*/
EXPLAIN (COSTS false) SELECT * FROM t3 WHERE t3.id = 1;
LOG:  available indexes for IndexScan(t3): t3_id2 t3_id1
LOG:  pg_hint_plan:
used hint:
IndexScan(t3 t3_id1 t3_id2)
not used hint:
duplication hint:
error hint:
 
          QUERY PLAN
-------------------------------
 Index Scan using t3_id2 on t3
   Index Cond: (id = 1)
(2 rows)
```

在前面的示例中，执行第一个查询时没有提示短语。因此，索引扫描使用主键索引t3_pkey。但是，第二个查询包含提示/*+IndexScan(t3 t3_id2)*/，因此它使用二级索引t3_id2执行索引扫描。当提供提示短语/*+IndexScan(t3 no_exist)*/时，计划器将恢复到SeqScan，因为没有索引可以使用。还可以在提示短语中提供索引的选择性列表，由查询规划器从中进行选择。

**连接方法的hints**
连接方法hints强制SQL语句的连接方法。使用pg_hint_plan，您可以指定连接表的方法。

| 选项                     | 描述                                  |
| ------------------------ | ------------------------------------- |
| HashJoin(t1 t2 t3 ...)   | 使用HashJoin连接t1、t2和t3。          |
| NoHashJoin(t1 t2 t3 ...) | 不要使用HashJoin连接t1、t2和t3        |
| NestLoop(t1 t2 t3 ...)   | 使用NestLoop Join连接t1、t2和t3。     |
| NoNestLoop(t1 t2 t3 ...) | 不要使用NestLoop join连接t1、t2和t3。 |
| BMDBBatchedNL(t1 t2)     | 使用BMDBBatchedNL Join连接t1和t2。    |

```
/*+HashJoin(t1 t2)*/
EXPLAIN (COSTS false) SELECT * FROM t1, t2 WHERE t1.id = t2.id;
LOG:  pg_hint_plan:
used hint:
HashJoin(t1 t2)
not used hint:
duplication hint:
error hint:
 
          QUERY PLAN
------------------------------
 Hash Join
   Hash Cond: (t1.id = t2.id)
   ->  Seq Scan on t1
   ->  Hash
         ->  Seq Scan on t2
(5 rows)
/*+NestLoop(t1 t2)*/
EXPLAIN (COSTS false) SELECT * FROM t1, t2 WHERE t1.id = t2.id;
LOG:  pg_hint_plan:
used hint:
NestLoop(t1 t2)
not used hint:
duplication hint:
error hint:
 
              QUERY PLAN
--------------------------------------
 Nested Loop
   ->  Seq Scan on t1
   ->  Index Scan using t2_pkey on t2
         Index Cond: (id = t1.id)
(4 rows)
```

在本例中，第一个查询分别对表t1和t2使用HashJoin，而第二个查询对表t1和t2使用NestedLoop连接。所需的连接方法在它们各自的hints短语中指定。您可以使用多个hints短语来组合连接方法和扫描方法。

```
/*+NestLoop(t2 t3 t1) SeqScan(t3) SeqScan(t2)*/
EXPLAIN (COSTS false) SELECT * FROM t1, t2, t3
WHERE t1.id = t2.id AND t1.id = t3.id;
LOG:  pg_hint_plan:
used hint:
SeqScan(t2)
SeqScan(t3)
NestLoop(t1 t2 t3)
not used hint:
duplication hint:
error hint:
 
              QUERY PLAN
--------------------------------------
 Nested Loop
   ->  Hash Join
         Hash Cond: (t2.id = t3.id)
         ->  Seq Scan on t2
         ->  Hash
               ->  Seq Scan on t3
   ->  Index Scan using t1_pkey on t1
         Index Cond: (id = t2.id)
(8 rows)
```

在上面的例子中，提示/*+NestLoop(t2 t3 t1) SeqScan(t3) SeqScan(t2)*/使能表t1、t2、t3上的NestLoop join。由于hints短语，它还在表t3和t4上启用了SeqScan。其余的扫描使用IndexScan执行。

**加入顺序hints**
连接顺序hints按特定顺序执行连接，如hints短语的参数列表中枚举的那样。可以使用Leading hints以特定顺序强制连接。

```
/*+Leading(t1 t2 t3)*/
EXPLAIN (COSTS false) SELECT * FROM t1, t2, t3 WHERE t1.id = t2.id AND t1.id = t3.id;
                 QUERY PLAN
--------------------------------------------
 Nested Loop
   ->  Nested Loop
         ->  Seq Scan on t1
         ->  Index Scan using t2_pkey on t2
               Index Cond: (id = t1.id)
   ->  Index Scan using t3_pkey on t3
         Index Cond: (id = t1.id)
(7 rows)
/*+Leading(t2 t3 t1)*/
EXPLAIN (COSTS false) SELECT * FROM t1, t2, t3 WHERE t1.id = t2.id AND t1.id = t3.id;
                 QUERY PLAN
--------------------------------------------
 Nested Loop
   ->  Nested Loop
         ->  Seq Scan on t2
         ->  Index Scan using t3_pkey on t3
               Index Cond: (id = t2.id)
   ->  Index Scan using t1_pkey on t1
         Index Cond: (id = t2.id)
(7 rows)
```

第一个查询的连接顺序是/*+Leading(t1 t2 t3)*/，而第二个查询的连接顺序是/*+Leading(t2 t3 t1)*/。您可以看到，查询计划的执行顺序遵循hints短语的参数列表中指定的连接顺序。

**设置工作区内存大小**
您可以利用PostgreSQL中的work_mem设置来提高对大量表行进行排序、连接或聚合的慢速查询的性能。有关其含义的详细描述，请参见调优PostgreSQL中的work_mem设置以加速慢速SQL查询。

下面的示例展示了如何启用work_mem作为提示计划的一部分。

```
/*+Set(work_mem "1MB")*/
EXPLAIN (COSTS false) SELECT * FROM t1, t2 WHERE t1.id = t2.id;
LOG:  pg_hint_plan:
used hint:
Set(work_mem 1MB)
not used hint:
duplication hint:
error hint:
 
              QUERY PLAN
--------------------------------------
 Nested Loop
   ->  Seq Scan on t1
   ->  Index Scan using t2_pkey on t2
         Index Cond: (id = t1.id)
(4 rows)
```

**配置计划器方法**
计划器方法配置参数提供了一种影响查询优化器选择的查询计划的粗略方法。如果优化器为特定查询选择的默认计划不是最优的，那么临时解决方案是使用这些配置参数之一来强制优化器选择不同的计划。BrightDB支持以下配置参数:enable_hashagg、enable_hashjoin、enable_indexscan、enable_indexonlyscan、enable_material、enable_nestloop、enable_partition_pruning、enable_partitionwise_join、enable_partitionwise_aggregate、enable_seqscan、enable_sort。
Pg_hint_plan通过在每个查询的注释中嵌入这些配置参数来利用计划器方法配置。考虑下面的例子:

```
EXPLAIN (COSTS false) SELECT * FROM t1, t2 WHERE t1.id = t2.id;
 QUERY PLAN
--------------------------------------
 Nested Loop
   ->  Seq Scan on t1
   ->  Index Scan using t2_pkey on t2
         Index Cond: (id = t1.id)
(4 rows)
/*+Set(enable_indexscan off)*/
EXPLAIN (COSTS false) SELECT * FROM t1, t2 WHERE t1.id = t2.id;
LOG:  pg_hint_plan:
used hint:
Set(enable_indexscan off)
not used hint:
duplication hint:
error hint:
 
          QUERY PLAN
------------------------------
 Hash Join
   Hash Cond: (t1.id = t2.id)
   ->  Seq Scan on t1
   ->  Hash
         ->  Seq Scan on t2
(5 rows)
```

第一个查询使用表t2中的IndexScan。但是，当添加/*+Set(enable_indexscan off)*/时，第二个查询使用表t2中的SeqScan。您可以在SQL查询的hints短语中组合任何参数。

有关Planner Method Configuration的更详细说明和可用配置参数的完整列表，请参阅PostgreSQL文档中的Planner Method Configuration。

**使用hints表**
在每个查询中嵌入hints短语可能会让人不知所措。为了提供帮助，pg_hint_plan可以使用一个名为hint_plan.hits的表，来提示存储常用的hints短语。使用hints表，您可以对类似类型的查询进行分组，并指示pg_hint_plan为所有此类查询启用相同的hints短语。

启用hints表的命令如下:

```
/* Create the hint_plan.hints table */
CREATE EXTENSION pg_hint_plan;
 
/*
 * Tell pg_hint_plan to check the hint table for
 * hint phrases to be embedded along with the query.
 */
SET pg_hint_plan.enable_hint_table = on;
```

下面的示例详细说明了这一点。

```
bigmath=# INSERT INTO hint_plan.hints
(norm_query_string,
 application_name,
 hints)
VALUES
('EXPLAIN (COSTS false) SELECT * FROM t1 WHERE t1.id = ?;',
 '',
 'SeqScan(t1)');
 
INSERT 0 1
 
bigmath=# INSERT INTO hint_plan.hints
(norm_query_string,
 application_name,
 hints)
VALUES
('EXPLAIN (COSTS false) SELECT id FROM t1 WHERE t1.id = ?;',
 '',
 'IndexScan(t1)');
 
INSERT 0 1
 
bigmath=# select * from hint_plan.hints;
-[ RECORD 1 ]-----+--------------------------------------------------------
id                | 1
norm_query_string | EXPLAIN (COSTS false) SELECT * FROM t1 WHERE t1.id = ?;
application_name  |
hints             | SeqScan(t1)
-[ RECORD 2 ]-----+--------------------------------------------------------
id                | 2
norm_query_string | EXPLAIN (COSTS false) SELECT id FROM t1 WHERE t1.id = ?;
application_name  |
hints             | IndexScan(t1)
```

本例将查询插入到hint_plan中。提示表，位置参数的占位符分别使用问号(?)和所需的hints短语。在运行时，当执行这些查询时，pg_hint_plan会自动使用它们各自的hints短语执行这些查询。
