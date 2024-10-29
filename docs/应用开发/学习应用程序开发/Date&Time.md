## **BSQL**

### **介绍**

BMDB具有广泛的日期和时间功能。一旦理解，丰富的功能允许您执行非常复杂的计算和精细的时间捕获。

### **特殊值**

你可以参考一些特殊的值-BMDB只满足一些，PostgreSQL中的其他特殊值没有在BMDB中实现，但如果你需要，可以重新创建一些。以下示例演示了BSQL选择特殊日期和时间值的方法。首先从命令行启动sqlsh。

```
./bin/sqlsh 
SELECT current_date, current_time, current_timestamp, now();
 current_date |    current_time    |       current_timestamp       |              now
--------------+--------------------+-------------------------------+-------------------------------
 2019-07-09   | 00:53:13.924407+00 | 2019-07-09 00:53:13.924407+00 | 2019-07-09 00:53:13.924407+00
SELECT make_timestamptz(1970, 01, 01, 00, 00, 00, 'UTC') as epoch;
         epoch
------------------------
 1970-01-01 00:00:00+00
SELECT (current_date-1)::timestamp as yesterday,
                  current_date::timestamp as today,
                  (current_date+1)::timestamp as tomorrow;
      yesterday      |        today        |      tomorrow
---------------------+---------------------+---------------------
 2019-07-08 00:00:00 | 2019-07-09 00:00:00 | 2019-07-10 00:00:00
```

### **格式化**

前面的示例显示了日期和时间戳的默认ISO格式。以下示例显示了如何设置日期格式：

```
SELECT to_char(current_timestamp, 'DD-MON-YYYY');
   to_char
-------------
 09-JUL-2019
SELECT to_date(to_char(current_timestamp, 'DD-MON-YYYY'), 'DD-MON-YYYY');
  to_date
------------
 2019-07-09
SELECT to_char(current_timestamp, 'DD-MON-YYYY HH:MI:SS PM');
         to_char
-------------------------
 09-JUL-2019 01:50:13 AM
```


示例使用to_char()函数以友好可读的格式显示日期。当表示为日期或时间数据类型时，将使用系统设置进行显示，这就是为什么文本09-JUL-2019的日期表示形式显示为2019-07-09 

### **时区**

BMDB安装的默认时区是UTC（+0）。要列出其他可用时区，请输入以下内容：

```
SELECT * FROM pg_timezone_names;
               name               | abbrev | utc_offset | is_dst
----------------------------------+--------+------------+--------
 W-SU                             | MSK    | 03:00:00   | f
 GMT+0                            | GMT    | 00:00:00   | f
 ROK                              | KST    | 09:00:00   | f
 UTC                              | UTC    | 00:00:00   | f
 US/Eastern                       | EDT    | -04:00:00  | t
 US/Pacific                       | PDT    | -07:00:00  | t
 US/Central                       | CDT    | -05:00:00  | t
 MST                              | MST    | -07:00:00  | f
 Zulu                             | UTC    | 00:00:00   | f
 posixrules                       | EDT    | -04:00:00  | t
 GMT                              | GMT    | 00:00:00   | f
 Etc/UTC                          | UTC    | 00:00:00   | f
 Etc/Zulu                         | UTC    | 00:00:00   | f
 Etc/Universal                    | UTC    | 00:00:00   | f
 Etc/GMT+2                        | -02    | -02:00:00  | f
 Etc/Greenwich                    | GMT    | 00:00:00   | f
 Etc/GMT+12                       | -12    | -12:00:00  | f
 Etc/GMT+8                        | -08    | -08:00:00  | f
 Etc/GMT-12                       | +12    | 12:00:00   | f
 WET                              | WEST   | 01:00:00   | t
 EST                              | EST    | -05:00:00  | f
 Australia/West                   | AWST   | 08:00:00   | f
 Australia/Sydney                 | AEST   | 10:00:00   | f
 GMT-0                            | GMT    | 00:00:00   | f
 PST8PDT                          | PDT    | -07:00:00  | t
 Hongkong                         | HKT    | 08:00:00   | f
 Singapore                        | +08    | 08:00:00   | f
 Universal                        | UTC    | 00:00:00   | f
 Arctic/Longyearbyen              | CEST   | 02:00:00   | t
 UCT                              | UCT    | 00:00:00   | f
 GMT0                             | GMT    | 00:00:00   | f
 Europe/London                    | BST    | 01:00:00   | t
 GB                               | BST    | 01:00:00   | t
 ...
(593 rows)
```

注：并不是所有可用的时区都显示出来；检查您的BSQL输出以找到您感兴趣的时区。 

您可以使用set命令设置会话使用的时区。您可以使用pg_timezone_names中列出的时区名称设置时区，但不能使用缩写。您也可以将时区设置为时间偏移的数字/十进制表示形式。例如，-3.5是UTC之前的3小时30分钟。 

提示：
能够使用上面的UTC_OFFSET格式设置时区似乎是合乎逻辑的。BMDB允许这样做，但是，如果您选择此方法，请注意以下行为：
当使用POSIX时区名称时，正偏移量用于格林尼治以西的位置。在其他地方，BMDB遵循ISO-8601惯例，即正时区偏移位于格林尼治以东。因此，输入“+10:00:0”会导致时区偏移量为-10小时，因为这被视为格林尼治以东。 

要显示基础服务器的当前日期和时间，请输入以下内容：

```
\echo `date`
Tue 09 Jul 12:27:08 AEST 2019
```

服务器时间不是数据库的日期和时间。然而，在BMDB的单节点实现中，您的计算机的日期和数据库日期之间存在关系，因为BMDB在启动时从服务器获得日期。

以下示例探讨数据库中的日期和时间（时间戳）。 

```
SHOW timezone;
 TimeZone
----------
 UTC
SELECT current_timestamp;
      current_timestamp
------------------------------
 2019-07-09 02:27:46.65152+00
SET timezone = +1;
SET
SHOW timezone;
 TimeZone
----------
 <+01>-01
SELECT current_timestamp;
      current_timestamp
------------------------------
 2019-07-09 03:28:11.52311+01
SET timezone = -1.5;
SET
SELECT current_timestamp;
        current_timestamp
----------------------------------
 2019-07-09 00:58:27.906963-01:30
SET timezone = 'Australia/Sydney';
SET
SHOW timezone;
     TimeZone
------------------
 Australia/Sydney
SELECT current_timestamp;
       current_timestamp
-------------------------------
 2019-07-09 12:28:46.610746+10
SET timezone = 'UTC';
SET
SELECT current_timestamp;
       current_timestamp
-------------------------------
 2019-07-09 02:28:57.610746+00
SELECT current_timestamp AT TIME ZONE 'Australia/Sydney';
          timezone
----------------------------
 2019-07-09 12:29:03.416867
```

（请注意，上面的AT TIME ZONE语句不适用于WITH TIME ZONE 和WITHOUT TIME ZONE的变体。） 

```
SELECT current_timestamp(0);
   current_timestamp
------------------------
 2019-07-09 03:15:38+00
SELECT current_timestamp(2);
     current_timestamp
---------------------------
 2019-07-09 03:15:53.07+00
```

使用时间戳时，可以通过指定0->6之间的值来控制秒精度。时间戳不能超过毫秒精度，即1000000分之一秒。

如果您的应用程序采用本地时间，请确保它发出SET命令以设置为正确的时间偏移。（夏令时是一个高级主题，因此建议暂时使用偏移量表示法，例如，在UTC之前的3小时30分钟为-3.5。） 

### **Timestamps**

数据库通常从底层服务器获取其日期和时间。然而，分布式数据库是一个同步的数据库，分布在许多不太可能具有同步时间的服务器上。
一个更简单的解释是，时间由表的分片leader决定，这是leader所有followers 使用的时间。因此，底层服务器的UTC时间戳可能与用于特定表上事务的当前时间戳不同。 

以下示例假设您已使用Retail Analytics 示例数据集创建并连接到bm_demo数据库： 

```
SELECT to_char(max(orders.created_at), 'DD-MON-YYYY HH24:MI') AS "Last Order Date" from orders;
  Last Order Date
-------------------
 19-APR-2020 14:07
SELECT extract(MONTH from o.created_at) AS "Mth Num", to_char(o.created_at, 'MON') AS "Month",
        extract(YEAR from o.created_at) AS "Year", count(*) AS "Orders"
   from orders o
  where o.created_at > current_timestamp(0)
  group by 1,2,3
  order by 3 DESC, 1 DESC limit 10;
 Mth Num | Month | Year | Orders
---------+-------+------+--------
       4 | APR   | 2020 |    344
       3 | MAR   | 2020 |    527
       2 | FEB   | 2020 |    543
       1 | JAN   | 2020 |    580
      12 | DEC   | 2019 |    550
      11 | NOV   | 2019 |    542
      10 | OCT   | 2019 |    540
       9 | SEP   | 2019 |    519
       8 | AUG   | 2019 |    566
       7 | JUL   | 2019 |    421
(10 rows)
SELECT to_char(o.created_at, 'HH AM') AS "Popular Hours", count(*) AS "Orders"
   from orders o
  group by 1
  order by 2 DESC
  limit 4;
Popular Hours | Orders
---------------+--------
 12 PM         |    827
 11 AM         |    820
 03 PM         |    812
 08 PM         |    812
(4 rows)
update orders
   set created_at = created_at + ((floor(random() * (25-2+2) + 2))::int * interval '1 day 14 hours');
UPDATE 18760
SELECT to_char(o.created_at, 'Day') AS "Top Day",
        count(o.*) AS "SALES"
   from orders o
  group by 1
  order by 2 desc;
Top Day  | SALES
-----------+---------
 Monday    |    2786
 Tuesday   |    2737
 Saturday  |    2710
 Wednesday |    2642
 Friday    |    2634
 Sunday    |    2630
 Thursday  |    2621
(7 rows)
create table order_deliveries (
          order_id bigint,
          creation_date date DEFAULT current_date,
          delivery_date timestamptz);
CREATE TABLE
insert into order_deliveries
          (order_id, delivery_date)
SELECT o.id, o.created_at + ((floor(random() * (25-2+2) + 2))::int * interval '1 day 3 hours')
   from orders o
  where o.created_at < current_timestamp - (20 * interval '1 day');
INSERT 0 12268
SELECT * from order_deliveries limit 5;
 order_id | creation_date |       delivery_date
----------+---------------+----------------------------
     5636 | 2019-07-09    | 2017-01-06 03:06:01.071+00
    10990 | 2019-07-09    | 2018-12-16 12:02:56.169+00
    13417 | 2019-07-09    | 2018-06-26 09:28:02.153+00
     9367 | 2019-07-09    | 2017-05-21 06:49:42.298+00
    13954 | 2019-07-09    | 2019-02-08 04:07:01.457+00
(5 rows)
SELECT d.order_id, to_char(o.created_at, 'DD-MON-YYYY HH AM') AS "Ordered",
        to_char(d.delivery_date, 'DD-MON-YYYY HH AM') AS "Delivered",
        d.delivery_date - o.created_at AS "Delivery Days"
   from orders o, order_deliveries d
  where o.id = d.order_id
        and d.delivery_date - o.created_at > interval '15 days'
   order by d.delivery_date - o.created_at DESC, d.delivery_date DESC limit 10;
 order_id |      Ordered      |     Delivered     |  Delivery Days
----------+-------------------+-------------------+------------------
    10984 | 12-JUN-2019 08 PM | 07-JUL-2019 02 AM | 24 days 06:00:00
     6263 | 01-JUN-2019 03 AM | 25-JUN-2019 09 AM | 24 days 06:00:00
    10498 | 18-MAY-2019 01 AM | 11-JUN-2019 07 AM | 24 days 06:00:00
    14996 | 14-MAR-2019 05 PM | 08-APR-2019 12 AM | 24 days 06:00:00
     6841 | 06-FEB-2019 01 AM | 02-MAR-2019 07 AM | 24 days 06:00:00
    10977 | 11-MAY-2019 01 PM | 03-JUN-2019 07 PM | 23 days 06:00:00
    14154 | 09-APR-2019 01 PM | 02-MAY-2019 07 PM | 23 days 06:00:00
     6933 | 31-MAY-2019 05 PM | 23-JUN-2019 12 AM | 22 days 06:00:00
     5289 | 04-MAY-2019 04 PM | 26-MAY-2019 10 PM | 22 days 06:00:00
    10226 | 01-MAY-2019 06 AM | 23-MAY-2019 12 PM | 22 days 06:00:00
(10 rows)
```

您的输出可能略有不同，因为RANDOM()函数用于在新的order_deliveries 表中设置delivery_date。
您可以使用BMDB数据目录的视图来创建已经为应用程序代码准备和格式化的数据，这样您的SQL就更简单了。以下示例显示了如何指定已格式化并可用于显示目的的时区： 

```
CREATE OR REPLACE VIEW TZ AS
SELECT '* Current time' AS "tzone", '' AS "offset", to_char(current_timestamp AT TIME ZONE 'Australia/Sydney', 'Dy dd-Mon-yy hh:mi PM') AS "Local Time"
UNION
SELECT x.name AS "tzone",
left(x.utc_offset::text, 5) AS "offset",
to_char(current_timestamp AT TIME ZONE x.name, 'Dy dd-Mon-yy hh:mi PM') AS "Local Time"
from pg_catalog.pg_timezone_names x
where x.name like 'Australi%' or name in('Singapore', 'NZ', 'UTC')
order by 1 asc;
CREATE VIEW
SELECT * from tz;
         tzone         | offset |       Local Time
-----------------------+--------+------------------------
 * Current time        |        | Wed 10-Jul-19 11:49 AM
 Australia/ACT         | 10:00  | Wed 10-Jul-19 11:49 AM
 Australia/Adelaide    | 09:30  | Wed 10-Jul-19 11:19 AM
 Australia/Brisbane    | 10:00  | Wed 10-Jul-19 11:49 AM
 Australia/Broken_Hill | 09:30  | Wed 10-Jul-19 11:19 AM
 Australia/Canberra    | 10:00  | Wed 10-Jul-19 11:49 AM
 Australia/Currie      | 10:00  | Wed 10-Jul-19 11:49 AM
 Australia/Darwin      | 09:30  | Wed 10-Jul-19 11:19 AM
 Australia/Eucla       | 08:45  | Wed 10-Jul-19 10:34 AM
 Australia/Hobart      | 10:00  | Wed 10-Jul-19 11:49 AM
 Australia/LHI         | 10:30  | Wed 10-Jul-19 12:19 PM
 Australia/Lindeman    | 10:00  | Wed 10-Jul-19 11:49 AM
 Australia/Lord_Howe   | 10:30  | Wed 10-Jul-19 12:19 PM
 Australia/Melbourne   | 10:00  | Wed 10-Jul-19 11:49 AM
 Australia/NSW         | 10:00  | Wed 10-Jul-19 11:49 AM
 Australia/North       | 09:30  | Wed 10-Jul-19 11:19 AM
 Australia/Perth       | 08:00  | Wed 10-Jul-19 09:49 AM
 Australia/Queensland  | 10:00  | Wed 10-Jul-19 11:49 AM
 Australia/South       | 09:30  | Wed 10-Jul-19 11:19 AM
 Australia/Sydney      | 10:00  | Wed 10-Jul-19 11:49 AM
 Australia/Tasmania    | 10:00  | Wed 10-Jul-19 11:49 AM
 Australia/Victoria    | 10:00  | Wed 10-Jul-19 11:49 AM
 Australia/West        | 08:00  | Wed 10-Jul-19 09:49 AM
 Australia/Yancowinna  | 09:30  | Wed 10-Jul-19 11:19 AM
 NZ                    | 12:00  | Wed 10-Jul-19 01:49 PM
 Singapore             | 08:00  | Wed 10-Jul-19 09:49 AM
 UTC                   | 00:00  | Wed 10-Jul-19 01:49 AM
(27 rows)
```

假设你选择了你感兴趣的时区，你的结果应该与上面显示的不同 

### **日期和时间间隔**

间隔是一种描述时间增量的数据类型。间隔允许您显示两个时间戳之间的差异，或者通过添加或减去特定的度量单位来创建新的时间戳。考虑以下示例： 

```
SELECT current_timestamp AS "Current Timestamp",
           current_timestamp + (10 * interval '1 min') AS "Plus 10 Mins",
           current_timestamp + (10 * interval '3 min') AS "Plus 30 Mins",
           current_timestamp + (10 * interval '2 hour') AS "Plus 20 hours",
           current_timestamp + (10 * interval '1 month') AS "Plus 10 Months";
       Current Timestamp       |         Plus 10 Mins          |         Plus 30 Mins          |         Plus 20 hours         |        Plus 10 Months
-------------------------------+-------------------------------+-------------------------------+-------------------------------+-------------------------------
 2019-07-09 05:08:58.859123+00 | 2019-07-09 05:18:58.859123+00 | 2019-07-09 05:38:58.859123+00 | 2019-07-10 01:08:58.859123+00 | 2020-05-09 05:08:58.859123+00
SELECT current_time::time(0), time '05:00' + interval '5 hours 7 mins' AS "New time";
 current_time | New Time
--------------+----------
 05:09:24     | 10:16:24
SELECT current_date - date '01-01-2019' AS "Day of Year(A)", current_date - date_trunc('year', current_date) AS "Day of Year(B)";
 Day of Year(A) | Day of Year(B)
----------------+----------------
            189 | 189 days
SELECT timestamp '2019-07-09 10:00:00.000000+00' - timestamp '2019-07-09 09:00:00.000000+00' AS "Time Difference";
 Time Difference
-----------------
 01:00:00
SELECT timestamp '2019-07-09 10:00:00.000000+00' - timestamptz '2019-07-09 10:00:00.000000+00' AS "Time Offset";
  Time Offset
-------------
 00:00:00 
SELECT timestamp '2019-07-09 10:00:00.000000+00' - timestamptz '2019-07-09 10:00:00.000000EST' AS "Time Offset";
 Time Offset
-------------
 -05:00:00
SELECT timestamp '2019-07-09 10:00:00.000000+00' - timestamptz '2019-07-08 10:00:00.000000EST' AS "Time Offset";
 Time Offset
-------------
 19:00:00
SELECT timestamp '2019-07-09 10:00:00.000000+00' - timestamptz '2019-07-07 10:00:00.000000EST' AS "Time Offset";
  Time Offset
----------------
 1 day 19:00:00
SELECT age(timestamp '2019-07-09 10:00:00.000000+00', timestamptz '2019-07-07 10:00:00.000000EST') AS "Age Diff";
    Age Diff
----------------
 1 day 19:00:00
SELECT (extract('days' from age(timestamp '2019-07-09 10:00:00.000000+00', timestamptz '2019-07-07 10:00:00.000000EST'))*24)+
           (extract('hours' from age(timestamp '2019-07-09 10:00:00.000000+00', timestamptz '2019-07-07 10:00:00.000000EST'))) AS "Hours Diff";
 Hours Diff
------------
         43
```

以上内容表明，日期和时间操作可以通过多种方式实现。需要注意的是，有些输出类型为INTEGER ，而另一些输出类型为INTERVAL（并非显示的文本）。上面“Hours Diff”的最后一个BSQL使用EXTRACT 的输出，该输出产生一个INTEGER ，以便它可以乘以每天的小时数，而EXTRACT 函数本身需要INTERVAL 或TIMESTAMP（TZ）数据类型作为其输入。

可以对时间（tz）、日期和时间戳（tz）进行转换，如MY_VALUE::timestamptz。 

### **截断操作**

DATE_TRUNC命令用于将时间戳“floor”到特定单元。本示例假设您已创建并连接到带有Retail Analytics示例数据集的bm_demo数据库。 

```
SELECT date_trunc('hour', current_timestamp);
       date_trunc
------------------------
 2019-07-09 06:00:00+00
(1 row)
SELECT to_char((date_trunc('month', generate_series)::date)-1, 'DD-MON-YYYY') AS "Last Day of Month"
          from generate_series(current_date-(365-1), current_date, '1 month');
 Last Day of Month
-------------------
 30-JUN-2018
 31-JUL-2018
 31-AUG-2018
 30-SEP-2018
 31-OCT-2018
 30-NOV-2018
 31-DEC-2018
 31-JAN-2019
 28-FEB-2019
 31-MAR-2019
 30-APR-2019
 31-MAY-2019
(12 rows)
SELECT date_trunc('days', age(created_at)) AS "Product Age" from products order by 1 desc limit 10;
      Product Age
------------------------
 3 years 2 mons 12 days
 3 years 2 mons 10 days
 3 years 2 mons 6 days
 3 years 2 mons 4 days
 3 years 1 mon 28 days
 3 years 1 mon 27 days
 3 years 1 mon 15 days
 3 years 1 mon 9 days
 3 years 1 mon 9 days
 3 years 1 mon
(10 rows)
```

### **整合**

一个常见的要求是找出下周一的日期；例如，出于日程安排的目的，这可能是新一周的第一天。这可以通过多种方式实现。以下说明了将不同的日期和时间运算符和函数链接在一起以获得所需结果： 

```
SELECT to_char(current_date, 'Day, DD-MON-YYYY') AS "Today",
           to_char((current_timestamp AT TIME ZONE 'Australia/Sydney')::date +
           (7-(extract('isodow' from current_timestamp AT TIME ZONE 'Australia/Sydney'))::int + 1),
           'Day, DD-MON-YYYY') AS "Start of Next Week";
         Today          |   Start of Next Week
------------------------+------------------------
 Tuesday  , 09-JUL-2019 | Monday   , 15-JUL-2019
```

上述方法是将一周中的当前日期提取为整数。由于今天是星期二，所以结果是2。正如您所知，每周有7天，您需要针对结果为8的计算，即比第7天多1天。我们使用它来计算在当前日期（7天-2+1天）上加多少天才能到达下周一，也就是一周中的第几天。添加AT TIME ZONE纯粹是为了说明问题，不会影响结果，因为您处理的是天，而时区差仅为+10小时，因此不会影响日期。然而，如果你的工作时间不超过几个小时，那么时区可能会对你的结果产生影响。 

### **歧义-使用DateStyle**

世界各地的人们都熟悉当地的日期表示。时间相当相似，但日期可能不同。美国使用3/5/19，而在澳大利亚使用5/3/19，在欧洲使用5.3.19或5/3/19。日期是什么？2019年3月5日。

BMDB具有应用于会话的DateStyle 设置，因此可以确定不明确的日期，并且可以将BSQL中的日期显示默认为特定格式。 

默认情况下，BMDB使用YYYY-MM-DD HH24:MI:SS的ISO标准。您可以使用的其他设置有“SQL”、“'German'”和“Postgres”。这些都用于以下示例中。

除了ISO之外的所有设置都允许您指定一天是出现在月之前还是之后。因此，设置“DMY”会导致3/5是5月3日，而“MDY”会在3月5日。

如果您从文件或任何非BMDB日期或时间戳数据类型的源读取日期作为文本字段，则正确设置DateStyle 非常重要，除非您非常具体地了解如何将文本字段转换为日期-下面包括一个示例。 

请注意，BMDB总是将“6/6”解释为6月6日，将“13/12”解释为12月13日（因为月份不能是13），但“6/12”呢？让我们来看看BSQL中的一些示例。

```
SHOW DateStyle;
 DateStyle
-----------
 ISO, DMY
SELECT current_date, current_time(0), current_timestamp(0);
 current_date | current_time |   current_timestamp
--------------+--------------+------------------------
 2019-07-09   | 20:26:28+00  | 2019-07-09 20:26:28+00
SET DateStyle = 'SQL, DMY';
SET
SELECT current_date, current_time(0), current_timestamp(0);
 current_date | current_time |    current_timestamp
--------------+--------------+-------------------------
 09/07/2019   | 20:26:48+00  | 09/07/2019 20:26:48 UTC
SET DateStyle = 'SQL, MDY';
SET 
SELECT current_date, current_time(0), current_timestamp(0);
 current_date | current_time |    current_timestamp
--------------+--------------+-------------------------
 07/09/2019   | 20:27:04+00  | 07/09/2019 20:27:04 UTC
SET DateStyle = 'German, DMY';
SET 
SELECT current_date, current_time(0), current_timestamp(0);
 current_date | current_time |    current_timestamp
--------------+--------------+-------------------------
 09.07.2019   | 20:27:30+00  | 09.07.2019 20:27:30 UTC
SET DateStyle = 'Postgres, DMY';
SET 
SELECT current_date, current_time(0), current_timestamp(0);
 current_date | current_time |      current_timestamp
--------------+--------------+------------------------------
 09-07-2019   | 20:28:07+00  | Tue 09 Jul 20:28:07 2019 UTC
SET DateStyle = 'Postgres, MDY';
SET 
SELECT current_date, current_time(0), current_timestamp(0);
 current_date | current_time |      current_timestamp
--------------+--------------+------------------------------
 07-09-2019   | 20:28:38+00  | Tue Jul 09 20:28:38 2019 UTC
SELECT '01-01-2019'::date;
    date
------------
 01-01-2019
SELECT to_char('01-01-2019'::date, 'DD-MON-YYYY');
   to_char
-------------
 01-JAN-2019
SELECT to_char('05-03-2019'::date, 'DD-MON-YYYY');
   to_char
-------------
 03-MAY-2019
```

以下示例说明了日期可能出现的困难：

```
SET DateStyle = 'Postgres, DMY';
SET 
SELECT to_char('05-03-2019'::date, 'DD-MON-YYYY');
   to_char
-------------
 05-MAR-2019
```

系统需要一个“DMY”值，但源的格式为“MDY”。BMDB不知道如何转换不明确的情况，所以要明确如下： 

```
SELECT to_char(to_date('05-03-2019', 'MM-DD-YYYY'), 'DD-MON-YYYY');
   to_char
-------------
 03-MAY-2019
```

建议通过TO_DATE 或TO_TIMESTAMP函数传递日期和时间数据类型的所有文本表示。没有“to_time”函数，因为它的格式始终固定为“HH24:MI:SS.ms”，因此要小心AM/PM时间，您的毫秒也可能是千分之一秒，因此应提供3或6位数字 

### **日志**

BMDB继承了许多类似于PostgreSQL API的功能，这就解释了为什么当你开始寻找引擎盖时，它看起来非常像PostgreSQL。
BMDB在其目录中跟踪其设置。以下示例查询一些相关设置，并使用“扩展显示”设置转换查询结果的布局。这可以在任何数据库中完成。 

```
\x on
Expanded display is on.
SELECT name, short_desc, coalesce(setting, reset_val) AS "setting_value", sourcefile
          from pg_catalog.pg_settings
          where name in('log_timezone', 'log_directory', 'log_filename', 'lc_time')
          order by name asc;
-[ RECORD 1 ]-+----------------------------------------------------------------
name          | lc_time
short_desc    | Sets the locale for formatting date and time values.
setting_value | en_US.UTF-8
sourcefile    | /home/xxxxx/bigmath-data/node-1/disk-1/pg_data/postgresql.conf
-[ RECORD 2 ]-+----------------------------------------------------------------
name          | log_directory
short_desc    | Sets the destination directory for log files.
setting_value | /home/xxxxx/bigmath-data/node-1/disk-1/bm-data/dbserver/logs
sourcefile    |
-[ RECORD 3 ]-+----------------------------------------------------------------
name          | log_filename
short_desc    | Sets the file name pattern for log files.
setting_value | postgresql-%Y-%m-%d_%H%M%S.log
sourcefile    |
-[ RECORD 4 ]-+----------------------------------------------------------------
name          | log_timezone
short_desc    | Sets the time zone to use in log messages.
setting_value | UTC
sourcefile    | /home/xxxxx/bigmath-data/node-1/disk-1/pg_data/postgresql.conf
\x off
```

使用log_directory和log_filename引用，您可以找到BMDB日志来检查插入到日志中的时间戳。这些都是UTC时间戳，应该保持不变。

您可以看到lc_time设置当前为UTF，并且列出了从中获取该设置的文件。以sudo/superuser的身份打开该文件，您会看到如下内容（搜索“datestyle”）： 

```
# - Locale and Formatting -
 
datestyle = 'iso, mdy'
#intervalstyle = 'postgres'
timezone = 'UTC'
#timezone_abbreviations = 'Default'     # Select the set of available time zone
                                        # abbreviations.  Currently, there are
                                        #   Default
                                        #   Australia (historical usage)
                                        #   India
                                        # You can create your own file in
                                        # share/timezonesets/.
#extra_float_digits = 0                 # min -15, max 3
#client_encoding = sql_ascii            # actually, defaults to database
                                        # encoding
 
# These settings are initialized by initdb, but they can be changed.
lc_messages = 'en_US.UTF-8'                     # locale for system error message
                                        # strings
lc_monetary = 'en_US.UTF-8'                     # locale for monetary formatting
lc_numeric = 'en_US.UTF-8'                      # locale for number formatting
lc_time = 'en_US.UTF-8'                         # locale for time formatting
 
# default configuration for text search
default_text_search_config = 'pg_catalog.english'
```

备份原始文件，然后更改datestyle='SQL，DMY'，timezone='GB'（或您喜欢的任何其他时区名称）并保存文件。您需要重新启动BMDB集群才能使更改生效。

群集按预期运行后，请执行以下操作：

```
./bin/sqlsh
sqlsh (11.2)
Type "help" for help.
SHOW timezone;
 TimeZone
----------
 GB
SELECT current_date;
 current_date
--------------
 09/07/2019
```

不需要每次输入BSQL时都进行这些设置。但是，应用程序不应该依赖于这些设置，它们应该在提交SQL之前始终设置其需求。这些设置只能由“临时查询”使用，如您现在所做的操作。

### **结论**

日期和时间是一个全面的领域，PostgreSQL和BMDB中的BSQL都很好地解决了这一问题。实现了所有的日期-时间数据类型，并且提供了绝大多数方法、运算符和特殊值。该功能非常复杂，您可以对其SQL API的BSQL实现中发现的任何不足进行编码。 
