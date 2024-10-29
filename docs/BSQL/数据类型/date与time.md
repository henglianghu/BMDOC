## **概要**

BSQL支持以下数据类型的值，这些值表示日期、时间、日期和时间，或持续时间。这些数据类型将被统称为日期-时间数据类型，如下表格所示：

| 类型              | 存储   | 描述                     | 最小值   | 最大值     | 解析度 |
| ----------------- | ------ | ------------------------ | -------- | ---------- | ------ |
| date              | 4字节  | 日期（没有一天中的时间） | 4713 BC  | 5874897 AD | 1日    |
| time [ (p) ]      | 8字节  | 一天中的时间（无日期）   | 00:00:00 | 24:00:00   | 1微秒  |
| timestamp [ (p) ] | 12字节 | 包括日期和时间           | 4713 BC  | 294276 AD  | 1微秒  |
| timestamptz [(p)] | 12字节 | 包括日期和时间           | 4713 BC  | 294276 AD  | 1微秒  |
| interval          | 16字节 | 时间间隔                 |          |            | 1微秒  |
| timetz [(p)]      |        | 不推荐使用               |          |            |        |

可选的 (p) 限定符，其中p是0..6中的整数值，以微秒为单位指定记录值的精度。（它对内部表示的大小没有影响。）仅在区间声明中有效的可选字段限定符，请参见区间数据类型。     
timestamptz 是一个别名，被接受为timestamp with time zone的一种简写，由PostgreSQL定义并由BSQL继承，SQL标准将其拼写为带时区的时间戳。不加修饰的timestamp由SQL标准定义，并且可以选择性地拼写为不带时区的timestamp，也适用于timetz和time。

注：避免使用“timetz”数据类型。 
PostgreSQL文档建议不要使用timetz（又称带时区的时间）数据类型： 
带时区的数据类型时间是由SQL标准定义的，但该定义显示的属性导致有用性受到质疑。在大多数情况下，日期、（普通）时间、（纯）时间戳和时间戳的组合，应该提供任何应用程序可能需要的完整的日期-时间功能范围。 

特殊值
为了方便，PostgreSQL和BSQL支持一些特殊日期/时间输入值，如下表所示。这些值中infinity和-infinity被在系统内部以特殊方式表示并且将被原封不动地显示。但是其他的仅仅只是概念上的速写，当被读到的时候会被转换为正常的日期/时间值（特殊地，now及相关串在被读到时立刻被转换到一个指定的时间值）。在作为常量在SQL命令中使用时，所有这些值需要被包括在单引号内。请参见PostgreSQL文档[Special Values](#DATATYPE-DATETIME-SPECIAL-VALUES)

| 输入串      | 合法类型                     | 描述                                    |
| ----------- | ---------------------------- | --------------------------------------- |
| 'epoch'     | date, timestamp              | 1970-01-01 00:00:00+00（Unix系统时间0） |
| 'infinity'  | date, timestamp, timestamptz | 比任何其他时间戳都晚                    |
| '-infinity' | date, timestamp              | 比任何其他时间戳都早                    |
| 'now'       | date, time, timestamp        | 当前事务的开始时间                      |
| 'today'     | date, timestamp              | 今日午夜 (00:00)                        |
| 'tomorrow'  | date, timestamp              | 明日午夜 (00:00)                        |
| 'yesterday' | date, timestamp              | 昨日午夜 (00:00)                        |
| 'allballs'  | time                         | 00:00:00.00 UTC                         |

注：除了“infinity”和“-infinity”，请避免使用所有这些特殊常量。而且，即使是针对与“infinity”和“-infinity”，也并不是适用于所有地方，例如：

```
select 'infinity'::timestamptz - clock_timestamp();
```

会报告错误信息：cannot subtract infinite timestamps

同样，select 'infinity'::interval;
也会报告错误信息：invalid input syntax for type interval: "infinity"

## **date**

数据类型为date的值，表示本地某个时刻（年、月和日）的日期。
日期的输入可以接受几乎任何合理的格式，包括 ISO 8601、SQL-兼容的、传统POSTGRES的和其他的形式。对于一些格式，日期输入里的日、月和年的顺序会让人混淆， 并且支持指定所预期的这些域的顺序。把DateStyle参数设置为MDY，就是选择“月－日－年”的解释，设置为DMY就是 “日－月－年”，而YMD是 “年－月－日”。

请记住任何日期或者时间的文字输入需要由单引号包围，就象一个文本字符串一样。

如下表中示例展示了可能的日期输入：

| 示例             | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| 1999-01-08       | ISO 8601; 任何模式下的1月8日（推荐格式）                     |
| January 8, 1999  | 在任何datestyle输入模式下都无歧义                            |
| 1/8/1999         | MDY模式中的1月8日；DMY模式中的8月1日                         |
| 1/18/1999        | MDY模式中的1月18日；在其他模式中被拒绝                       |
| 01/02/03         | MDY模式中的2003年1月2日；DMY模式中的2003年2月1日； YMD模式中的2001年2月3日 |
| 1999-Jan-08      | 任何模式下的1月8日                                           |
| Jan-08-1999      | 任何模式下的1月8日                                           |
| 08-Jan-1999      | 任何模式下的1月8日                                           |
| 99-Jan-08        | YMD模式中的1月8日，否则错误                                  |
| 08-Jan-99        | 1月8日，在YMD模式中错误                                      |
| Jan-08-99        | 1月8日，在YMD模式中错误                                      |
| 19990108         | ISO 8601; 任何模式中的1999年1月8日                           |
| 990108           | ISO 8601; 任何模式中的1999年1月8日                           |
| 1999.008         | 年和一年中的天数                                             |
| J2451187         | 儒略日期                                                     |
| January 8, 99 BC | 公元前99年                                                   |

时间的输出格式可以设成四种风格之一： ISO 8601、SQL（Ingres）、传统的POSTGRES（Unix的date格式）或 German 。缺省是ISO格式（ISO标准要求使用 ISO 8601 格式。）。下表显示了每种输出风格的示例：

| 风格声明 | 描述              | 示例       |
| -------- | ----------------- | ---------- |
| ISO      | ISO 8601, SQL标准 | 1997-12-17 |
| SQL      | 传统风格          | 12/17/1997 |
| Postgres | 原始风格          | Wed Dec 17 |
| German   | 地区风格          | 17.12.1997 |

SQL和POSTGRES风格中，如果DMY域顺序被指定，“日”将出现在“月”之前，否则“月”出现在“日”之前。示例如下表所示：

| datestyle设置 | 输入顺序 | 示例输出                     |
| ------------- | -------- | ---------------------------- |
| SQL, DMY      | 日/月/年 | 17/12/1997 15:37:16.00 CET   |
| SQL, MDY      | 月/日/年 | 12/17/1997 07:37:16.00 PST   |
| Postgres, DMY | 日/月/年 | Wed 17 Dec 07:37:16 1997 PST |

请注意，从另一个日期值中减去一个日期会产生一个整数值：两个指定日期之间的天数，该值对公历闰年敏感。例如：

```
select pg_typeof('2019-02-14'::date - '2018-02-14'::date)::text as "data type of difference between dates";
```

返回信息如下：

```
 data type of difference between dates
----------------------------------------------
 Integer
```


## **time**

数据类型为time的值，表示本地某个时刻的时间，小时和分钟为整数值，秒为微秒精度的实数值。
time类型可细分为是time [ (p) ] without time zone和time [ (p) ] with time zone。 只写time等效于time without time zone。 
这些类型的有效输入，是由当日时间后面跟着可选的时区组成，见下表示例所示。
如果在time without time zone的输入中指定了时区，那么它会被忽略。也可以指定一个日期，但是它也会被忽略，除非你使用了一个涉及到夏令时规则的时区，例如America/New_York。在这种情况下，为了判断是应用了标准时间还是夏令时时间，要求指定该日期。适当的时区偏移被记录在time with time zone值中。

有效时间输入示例： 

| 示例                                 | 描述                                |
| ------------------------------------ | ----------------------------------- |
| 04:05:06.789                         | ISO 8601                            |
| 04:05:06                             | ISO 8601                            |
| 04:05                                | ISO 8601                            |
| 040506                               | ISO 8601                            |
| 04:05 AM                             | 和04:05一样，AM并不影响值           |
| 04:05 PM                             | 和16:05一样，输入的小时必须为 <= 12 |
| 04:05:06.789-8                       | ISO 8601                            |
| 04:05:06-08:00                       | ISO 8601                            |
| 04:05-08:00                          | ISO 8601                            |
| 040506-08                            | ISO 8601                            |
| 04:05:06 PST                         | 缩写指定的时区                      |
| 2003-04-12 04:05:06 America/New_York | 全名指定的时区                      |

有效时区输入示例：

| 示例             | 描述                   |
| ---------------- | ---------------------- |
| PST              | 缩写（太平洋标准时间） |
| America/New_York | 完整时区名             |
| PST8PDT          | POSIX风格的时区声明    |
| -8:00            | PST的ISO-8601偏移      |
| -800             | PST的ISO-8601偏移      |
| -8               | PST的ISO-8601偏移      |
| zulu             | UTC的军方缩写          |
| z                | zulu的短形式           |

由于时间值对时区一无所知，他们也对夏令时制度一无所知：每天都应该从午夜（含）开始，一直持续到下一个午夜（不含）结束。见下例所示：

```
select
  ('00:00:00.000000'::time)::text as "starting midnight",
  ('23:59:59.999999'::time)::text as "just before ending midnight",
  ('24:00:00.000000'::time)::text as "ending midnight";
```

事实上，存在一个小小的烦恼，就是这两个time（'00:00:00.000000'，'24:00:00.000000'）并不相等，见下例所示 ：

```
 select (
  '00:00:00.000000'::time =
  '24:00:00.000000'::time
  )::text;
```

结果返回false。由于时间值不包含日期信息，您需要定义并公布辅助应用程序语义，以将24:00:00解释为特定日期的结束，而不是前一天的开始，如下进一步展示：

```
select
  ('02:00:00'::time - '01:00:00'::time)::text as "interval 1",
  ('24:00:00'::time - '00:00:00'::time)::text as "interval 2",
  justify_hours('24:00:00'::time - '00:00:00'::time)::text as "interval 3";
```

返回信息如下：

```
 interval 1 | interval 2 | interval 3
------------+------------+------------
 01:00:00   | 24:00:00   | 1 day
```

这里还有另一个烦恼：在日期值中添加间隔值可能会导致环绕。见下例所示 ：

```
select '24:00:00'::time + '1 second'::interval;
```

返回信息如下：

```
 00:00:01
```

## **时区和UTC偏移**

### **时区**

时区和时区习惯不仅仅受地球几何形状的影响，还受到政治决定的影响。 到了19世纪，全球的时区变得稍微标准化了些，但是还是易于遭受随意的修改，部分是因为夏时制规则。
SQL标准在日期和时间类型和功能上有一些奇怪的混淆。两个显而易见的问题是：

* 尽管date类型与时区没有联系，而time类型却可以有。 然而，现实世界的时区只有在与时间和日期都关联时才有意义， 因为偏移（时差）可能因为实行类似夏时制这样的制度而在一年里有所变化。
* 缺省的时区会指定一个到UTC的数字常量偏移（时差）。因此，当跨DST边界做日期/时间算术时， 我们根本不可能适应于夏时制时间。

为了克服这些困难，我们建议在使用时区的时候，使用那些同时包含日期和时间的日期/时间类型。我们不建议使用类型 time with time zone。 假设你用于任何类型的本地时区都只包含日期或时间。

在系统内部，所有时区相关的日期和时间都用UTC存储。它们在被显示给客户端之前，会被转换成由TimeZone配置参数指定的本地时间。

允许你使用三种不同形式指定时区：

* 一个完整的时区名字，例如America/New_York。能被识别的时区名字被列在pg_timezone_names视图中。用广泛使用的 IANA 时区数据来实现该目的，因此相同的时区名字也可以在其他软件中被识别。

* 一个时区缩写，例如PST。这样一种声明仅仅定义了到UTC的一个特定偏移，而不像完整时区名那样指出整套夏令时转换规则。 能被识别的缩写被列在pg_timezone_abbrevs视图中。 你不能将配置参数TimeZone或log_timezone设置成一个时区缩写，但是你可以在日期/时间输入值和AT TIME ZONE操作符中使用时区缩写。

除了时区名和缩写，接受POSIX-风格的时区规范。 这个选项通常不优先用于指定时区，但是，如果没有合适的IANA时区条目，这可能是必要的。

简而言之，在缩写和全称之间是有不同的：缩写表示从UTC开始的一个特定偏移量， 而很多全称表示一个本地夏令时规则并且因此具有两种可能的UTC偏移量。例如， 2014-06-04 12:00 America/New_York表示纽约本地时间的中午， 这个特殊的日期是东部夏令时间（UTC-4）。因此2014-06-04 12:00 EDT 指定的是同一个时间点。但是2014-06-04 12:00 EST指定东部标准时间的 中午（UTC-5），不管在那个日期夏令时是否生效。

更要命的是，某些行政区已经使用相同的时区缩写在不同的时间表示不同的 UTC 偏移量。例如， 在莫斯科MSK在某些年份表示 UTC+3 而在另一些年份表示 UTC+4。 根据指定的日期它们到底表示什么（或者最近表示什么） 来解释这种缩写。但是，正如上面的EST示例所示，这并不是必须和那一天的本地 标准时间相同。

在所有情况下，时区名及其缩写都是大小写不敏感的。

TimeZone配置参数可以在文件postgresql.conf中被设置。同时也有一些特殊的方法来设置它。

SQL命令SET TIME ZONE为会话设置时区。它是SET TIMEZONE TO的另一种拼写，它更加符合SQL的语法。

“timezone”和“time zone”这两种拼写形式都出现在SQL语法中。例如，set timezone = <arg>和set time zone <arg> 都是合法的。

AiSQL 中包含有两个相关的目录视图，用下面SQL可以获取：

```
select table_name as "name"
from information_schema.views
where lower(table_schema) = 'pg_catalog'
and (
  lower(table_name) like '%zone%' or
  lower(table_name) like '%time%')
order by 1;
```

返回信息如下：

```
 pg_timezone_abbrevs
 pg_timezone_names
```


pg_timezone_names视图不一定明确地告诉您，特定时区是否遵守夏令时。但对于这样做的时区，当夏令时生效时，is_dst的值将为true。

例如：

```
select is_dst from pg_timezone_names where name ='America/Los_Angeles';
```

信息返回如下：

```
 is_dst 
--------
 t
```

接下来：

```
set timezone = 'America/Los_Angeles';
select
  current_setting('timezone')                                     as "timezone",
  to_char('2021-01-01 12:00:00 UTC'::timestamptz, 'TZH:TZM : TZ') as "January regime",
  to_char('2021-07-01 12:00:00 UTC'::timestamptz, 'TZH:TZM : TZ') as "July regime";
```

返回信息如下：

```
      timezone       | January regime | July regime
-----------------------------+-------------------+----------------
 America/Los_Angeles | -08 : PST      | -07 : PDT
```

创建如下函数：

```
drop function if exists jan_and_jul_tz_abbrevs_and_offsets() cascade;
 
create function jan_and_jul_tz_abbrevs_and_offsets()
  returns table(
  name        text,
  jan_abbrev  text,
  jul_abbrev  text,
  jan_offset  interval,
  jul_offset  interval)
  language plpgsql
as $body$
declare
  set_timezone constant text not null := $$set timezone = '%s'$$;
  tz_set                text not null := '';
  tz_on_entry           text not null := '';
begin
  show timezone into tz_on_entry;
 
  for tz_set in (
    select pg_timezone_names.name as a
    from pg_timezone_names
  ) loop
    execute format(set_timezone, tz_set);
    select
      current_setting('timezone'),
      to_char('2021-01-01 12:00:00 UTC'::timestamptz, 'TZ'),
      to_char('2021-07-01 12:00:00 UTC'::timestamptz, 'TZ'),
      to_char('2021-01-01 12:00:00 UTC'::timestamptz, 'TZH:TZM')::interval,
      to_char('2021-07-01 12:00:00 UTC'::timestamptz, 'TZH:TZM')::interval
    into
      name,
      jan_abbrev,
      jul_abbrev,
      jan_offset,
      jul_offset;
    return next;
  end loop;
 
  execute format(set_timezone, tz_on_entry);
end;
$body$;
```

执行如下：

```
select
  name,
  jan_abbrev,
  jul_abbrev,
  lpad(jan_offset::text, 9) as jan_offset,
  lpad(jul_offset::text, 9) as jul_offset
from jan_and_jul_tz_abbrevs_and_offsets()
where
name = 'America/Los_Angeles' or name = 'Europe/London'
order by name;
```

返回信息如下：

```
        name         | jan_abbrev | jul_abbrev | jan_offset | jul_offset
---------------------+------------+------------+------------+------------
 America/Los_Angeles | PST        | PDT        | -08:00:00  | -07:00:00
 Europe/London       | GMT        | BST        |  00:00:00  |  01:00:00
```

 

这两个视图的区别在于：

* pg_timezone_names从其关键字name映射到缩写和utc_offset，它们的值在夏令时和标准时间段通常不同，映射在任何特定日期都是唯一的。
* pg_timezone_abbrevs从其关键字abbrev映射到一个固定的、唯一的utc_offset值。当时区遵循夏令时时，它有两个不同的缩写：一个用于夏令时时段，另一个用于标准时段。
* pg_timezone_names.abbrev和pg_timezone_abbevs.abbrev列记录了不同类型的事实。

### **UTC偏移**

所有可能的操作都不可避免地在指定的UTC偏移量的上下文中执行，因为TimeZone会话设置的默认方案确保这永远不是零长度的文本值或null。（请参阅使用会话环境参数TimeZone指定UTC偏移量一节。）TimeZone设置可以将UTC偏移量直接指定为间隔值，也可以通过标识时区间接指定。
但是，只有三个操作对设置敏感：
将timestamp 转换为timestamptz 。
将timestamptz 转换为timestamp 。
向timestamptz 添加或者减去间隔值。 

#### 指定UTC偏移的四种方法

* 直接作为区间值
  要为会话的“时区”设置指定间隔值，必须使用set time zone＜arg＞语法，而不是set timezone= <arg> 语法。例如：

```
set time zone interval '-7 hours';
```

请注意，在此语法上下文中，只能使用类型名称构造函数指定间隔值。请试着使用以下：

```
set time zone '-7 hours'::interval;
 
set time zone make_interval(hours => -7);
```

均会报告语法错误。

但是，at time zone运算符允许任何任意的区间表达式。示例如下：

```
select '2021-05-27 12:00:00'::timestamp at time zone (make_interval(hours=>5) + make_interval(mins=>45));
```

您还可以在时间戳文本的文本中指定间隔值。但在这里，您仅仅使用带有::interval  类型转换的文本。示例如下：

```
select '2021-05-27 12:00:00 -03:15:00'::timestamptz;
```

* 直接使用POSIX语法
  该语法在PostgreSQL文档中进行了描述，详细内容请参见PostgreSQL文档相关部分。它允许您指定两个UTC偏移值，一个用于标准时间，另一个用于夏令时间，以及“向前跳跃”和“向后后退”时刻。你极不可能需要使用它，因为如今，地球上的任何地方都属于一个经典命名的时区，该时区是当前理解的夏令时规则的关键，从不确定的过去到不确定的未来。PostgreSQL和AiSQL服务器可以访问这些规则。（来源是所谓的tz数据库。）如果改变了任何规则，那么tz数据库就会更新，新规则就会被采用到PostgreSQL和AiSQL的配置数据中。
  在显示时区，设置为显式间隔值后，执行show timezone，会使用POSIX语法返回报告，尽管这是一种不指定夏令时转换的简单形式。执行如下示例：

```
set time zone interval '-07:30:00';
show timezone;
```

返回信息如下：

```
<-07:30>+07:30
```

再试着执行如下：

```
set timezone = '<-07:30>+07:30';
show timezone;
```

返回与上例相同信息，结果完全一样。事实证明，几乎任何包含一个或多个数字的字符串都可以被解释为POSIX语法。试着执行如下：

```
set timezone = 'FooBar5';
show timezone;
```

返回信息如下：

```
 TimeZone
----------
 FOOBAR5
```

现在，您可以很容易地确认的一点就是，在如下：pg_timezone_names.name，pg_timezone_names.abbrev或者 pg_timezone_abbrevs.abbrev中都没有找到FOOBAR5。

现在看看这有什么效果，如下所示：

```
\set bare_date_time '\'2021-04-15 12:00:00\''
select :bare_date_time::timestamptz;
```

返回信息如下：

```
 2021-04-15 12:00:00-05
```

POSIX取正数表示格林尼治子午线以西。然而，PostgreSQL显示UTC偏移量的约定是将其显示为负值，参见ISO 8601，似乎是绝大多数更常见的约定。互联网搜索似乎总是显示格林尼治以西UTC偏移值为负的地方的时区。试试这个： 

```
set time zone interval '-5 hours';
select :bare_date_time::timestamptz;
```

这里，PostgreSQL约定用于指定UTC偏移值。使用相同的：bare_date_time，结果2021-04-15 12:00:00-05与时区设置为FooBar5时的结果相同。

接下来，再试试如下：

```
set timezone = 'Foo5Bar';
select :bare_date_time::timestamptz;
```

返回信息如下：

```
2021-04-15 12:00:00-04
```

时区设置为FooBar5时，将2021-04-15 12:00:00强制转换为timestamptz 的结果具有负五小时的UTC偏移值。但当时区设置为Foo5Bar时，将相同的timestamp 值强制转换为timestamptz的结果是UTC偏移值为负4小时。您可以猜测，当给出夏令时规则的最简洁规范，在POSIX文本的最后一位数字（至少在本例中）时，这与POSIX编码夏令时规则的方式有关，也与定义的默认值有关（在本例中，夏令时“向前跳”一个小时）。


* 间接使用时区名称 
  这绝大多数是首选方法，因为只有这样，才能根据名称在tz数据库的内部，表示指向的夏令时规则，自动映射UTC偏移值。
* 间接使用时区缩写
  避免使用这种方法。尽管这种方法是合法的，但AiSQL强烈建议您避免使用它。

#### 使用UTC偏移量的三个语法上下文

* 设置环境变量TimeZone
  1）可以在postgresql.conf文件中设置，也可以以PostgreSQL文档的 Server Configuration中描述的任何其他标准方式设置。当未设置PGTZ环境变量时，这在新启动的会话中显示为“TimeZone”会话参数的值。
  2）可以使用PGTZ环境变量进行设置。libpq客户端在连接时，向服务端发送一个set time zone命令。这将覆盖postgresql.conf中的值，这样它也会在新启动的会话中显示为“TimeZone”会话参数的值。
  3）可以在正在进行的会话中，使用通用set timezone=<text literal>语法进行设置，此语法不允许文本表达式。或者，也可以使用特定于时区的set time zone interval＜arg＞语法进行设置。这种拼写还允许设置时区间隔＜text literal＞（注意：文本表达式非法）。此外，它唯一允许set time zone interval＜interval literal＞语法；值得注意的是，只允许使用间隔值的这种语法。 

示例如下：

```
set timezone = 'America/New_York';
 
set time zone 'Asia/Tehran';
 
set time zone interval '1 hour';
```

注意：如果在pg_timezone_names中找不到提供的文本，则会尝试将其解析为POSIX语法。结果可能会让你大吃一惊。试着执行下面这个示例： 

```
deallocate all;
prepare stmt as
select '2008-07-01 13:00 America/Los_Angeles'::timestamptz;
 
set timezone = 'America/Los_Angeles';
execute stmt;
 
set timezone = '-07:00';
execute stmt;
 
set timezone = 'Foo-7';
execute stmt;
```

返回信息如下：

```
 2008-07-01 13:00:00-07
 
 2008-07-02 03:00:00+07
 
 2008-07-02 03:00:00+07
```

* 使用at time zone运算符指定UTC偏移量
  运算符的合法操作数是文本表达式或区间表达式，运算符语法与函数timezone()的语义是相同的。
  以下是四种语法变体的示例：

```
drop function if exists timezone_name() cascade;
create function timezone_name()
  returns text
  language plpgsql
as $body$
begin
  -- Some logic would go here. Just return a manifest constant for this demo.
  return 'Europe/Amsterdam';
end;
$body$;
 
select make_timestamp(2021, 6, 1, 12, 13, 19) at time zone timezone_name();
 
select timezone(timezone_name(), make_timestamp(2021, 6, 1, 12, 13, 19));
 
select make_timestamptz(2021, 6, 1, 12, 13, 19, 'Europe/Helsinki') at time zone make_interval(hours=>4, mins=>30);
 
select timezone(make_interval(hours=>4, mins=>30), make_timestamptz(2021, 6, 1, 12, 13, 19, 'Europe/Helsinki'));
```

* 在timestamptz 文本定义，或者函数make_timestamptz()的'timezone' 参数的文本中，显式指定UTC偏移量。
  时间戳文本可以是一个表达式。以下是一个示例：

```
select ('1984-04-01 13:00 '||timezone_name())::timestamptz; 
```

这里也有一种可能性，用嵌入的数字来解释文本值。试试如下示例：

```
set timezone = 'America/Los_Angeles';
\x on
with c as (select '2008-07-01 13:00' as ts)
select
  (select ((select ts from c)||' America/Los_Angeles')::timestamptz) as "Spelling 'LA' timezone in text",
  (select ((select ts from c)||' -07:00'             )::timestamptz) as "Attempting to specify the PDT offset",
  (select ((select ts from c)||' Foo99'              )::timestamptz) as "Specifying 'Foo99'";
\x off
```

返回信息如下：

```
Spelling 'LA' timezone in text          | 2008-07-01 13:00:00-07
Attempting to specify the PDT offset    | 2008-07-01 13:00:00-07
Specifying 'Foo99'                   | 2008-07-05 09:00:00-07
```

## **timestamp与timestamptz**

普通timestamp 数据和timestamptz 数据都具有相同的内部表示。您可以将其想象为从参考时刻（1970年1月1日12:00，UTC）开始的实际秒数（以微秒为精度）。extract(epoch from t) 函数会返回这个数字，其中t是timestamp 或timestamptz 值。此外，对于这两种数据类型，结果与会话的当前时区设置无关。

### **timestamp**

timestamp 值表示本地时间制度中某个时刻的日期和时间。无论是在创建timestamp 值时，还是在读取该值时，都不存在时区敏感性。因此，这样的值表示某个未指定位置的时刻，就像显示日期的钟表一样。您可以将时间戳值想象为从某个历元开始到当前时刻的微秒数。
因为没有时区敏感性，所以对夏令时制度也没有敏感性：每天从午夜（含）到下一个午夜（不含）。PostgresSQL和BSQL不支持闰秒，然而，当您指定timestamp 值时，允许将“24:00:00”作为时间值，将其作为一天中的时间组件。试着执行如下示例：

```
select
  ('2021-02-14 00:00:00.000000'::timestamp)::text as "starting midnight",
  ('2021-02-14 23:59:59.999999'::timestamp)::text as "just before ending midnight",
  ('2021-02-14 24:00:00.000000'::timestamp)::text as "ending midnight";
```

返回如下信息：

```
  starting midnight  | just before ending midnight |   ending midnight
---------------------+-----------------------------+---------------------
 2021-02-14 00:00:00 | 2021-02-14 23:59:59.999999  | 2021-02-15 00:00:00
```

接下来，执行如下：

```
select
  ('2021-02-14 24:00:00'::timestamp - '2021-02-14 00:00:00'::timestamp)::text as "the interval";
```

可以看到返回如下信息：

```
 the interval
--------------
 1 day
```

### **timestamptz**

timestamptz 值表示绝对时间。因此，这样的值可以确定地表示为UTC时区中的当地时间，或者更仔细地说，表示为相对于UTC时间标准偏移量为“0秒”::interval 的当地时间。
尽管实际的timestamp 值和实际的timestamptz 值的表示是相同的，但由于值的元数据的差异，timestamp 值与timestamptz 的语义以及解释是不同的。

指定timestamptz 值时，还必须指定UTC偏移量。一旦记录了timestamptz 值（例如，在具有该数据类型的表列中），就不记得记录时的偏移量了。当传入值被标准化为UTC时，该信息将被完全消耗掉。

当timestamptz 值转换为要显示的文本值时，隐式使用::text类型转换，或显式使用to_char() 内置函数，转换会规范化日期-时间组件，以便显示相对于当前时区指定的UTC偏移量的本地时间。

UTC偏移量可以隐式指定（使用会话的当前时区设置），也可以在timestamptz 文本中显式指定，或者使用 at time zone运算符。此外，偏移量的指定可以是显式的，作为间隔值，或者隐式的使用时区的名称。当时区观察到夏令时时，其名称表示标准时间段和夏季时间段期间的不同偏移。

示例如下：

```
drop table if exists t cascade;
create table t(k int primary key, v timestamptz not null);
 
-- This setting has no effect on the inserted value.
set timezone = 'Europe/Paris';
insert into t(k, v) values(1, '2021-02-14 13:30:35+03:00'::timestamptz);
 
set timezone = 'America/Los_Angeles';
select v::text as "timestamptz value" from t where k = 1;
 
set timezone = 'Asia/Shanghai';
select v::text as "timestamptz value" from t where k = 1;
```

分别返回如下信息：

```
2021-02-14 02:30:35-08
 
2021-02-14 18:30:35+08
```

### **夏令时**

如下演示了夏令时的示例：

美国承认夏令时。它将于2021年3月14日02:00在“America/Los_Angeles”区域开始。观看根据夏令时开始/结束时刻，自动调整的时钟（如智能手机上的时钟）的功能。从01:59:59到03:00:00。如下：

```
set timezone = 'America/Los_Angeles';
select
  '2021-03-14 01:30:00 America/Los_Angeles'::timestamptz as "before",
  '2021-03-14 02:30:00 America/Los_Angeles'::timestamptz as "wierd",
  '2021-03-14 03:30:00 America/Los_Angeles'::timestamptz as "after";
```

返回如下信息：

```
         before         |         wierd          |         after
------------------------+------------------------+------------------------
 2021-03-14 01:30:00-08 | 2021-03-14 03:30:00-07 | 2021-03-14 03:30:00-07
```

别名为“weird”的列中的值很怪异，因为“2021-03-14 02:30:00”不存在。

America/Los_Angeles时区的夏令时将于2021年11月7日02:00:00结束。它从“01:59:59”回落到“01:00:00”。这意味着，例如，11月7日的“01:30:00”是不明确的。如果你在11月6日星期六晚上给室友打电话，说你要到凌晨，可能是一点半左右才能回家，他们不会知道你的意思，因为时钟会把这个时间读两次。请试着执行如下：

```
set timezone = 'America/Los_Angeles';
select
  to_char('2021-11-07 08:30:00 UTC'::timestamptz, 'hh24:mi:ss TZ (OF)') as "1st 1:30",
  to_char('2021-11-07 09:30:00 UTC'::timestamptz, 'hh24:mi:ss TZ (OF)') as "2nd 1:30";
```

返回如下信息：

```
      1st 1:30      |      2nd 1:30
----------------------------+------------------------
 01:30:00 PDT (-07)  | 01:30:00 PST (-08)
```

因此，你必须告诉他们，你将以下面一种方式回来：

* 或者在大约01:30 PDT（或者大约01:30PST）
* 或等效地在夏令时回退时刻前约01:30（或夏令时回退时刻后01:30）
* 或在约01:30UTC-7（或约01:30UTC-8）
* 或者甚至在大约八点半UTC（或者九点半UTC）。

遵守夏令时的这种奇怪（但在逻辑上不可避免）的后果，意味着你也必须在另一个方向上小心。请试着执行如下：

```
select
  to_char(('2021-11-07 01:30:00 America/Los_Angeles'::timestamptz) at time zone 'UTC', 'hh24:mi:ss') as "Ambiguous";
```

返回如下信息：

```
 Ambiguous
-----------
 09:30:00
```

PostgreSQL（以及BSQL）通过约定来解决歧义：选择较晚的时刻。明确表达你的意思以避免歧义： 

```
set timezone = 'UTC';
select
  to_char('2021-11-07 01:30:00 -07:00'::timestamptz, 'hh24:mi:ss TZ') as "Before fallback",
  to_char('2021-11-07 01:30:00 -08:00'::timestamptz, 'hh24:mi:ss TZ') as "After fallback";
```

返回如下信息：

```
 Before fallback | After fallback
--------------------+-------------------
 08:30:00 UTC  | 09:30:00 UTC
```

或者，可以这样做：

```
set timezone = 'America/Los_Angeles';
select
  to_char('2021-11-07 01:30:00 -07:00'::timestamptz, 'hh24:mi:ss TZ') as "Before fallback",
  to_char('2021-11-07 01:30:00 -08:00'::timestamptz, 'hh24:mi:ss TZ') as "After fallback";
```

返回如下信息：

```
 Before fallback | After fallback
---------------------+----------------
 01:30:00 PDT  | 01:30:00 PST
```

## **interval**

interval值可以使用下列语法书写：

[@] quantity unit [quantity unit...] [direction]

其中quantity是一个数字（很可能是有符号的）； unit是毫秒、 millisecond、second、 minute、hour、day、 week、month、year、 decade、century、millennium 或者缩写或者这些单位的复数； direction可以是ago或者为空。At符号（@）是一个可选的噪声。不同单位的数量通过合适的符号计数被隐式地添加。ago对所有域求反。如果IntervalStyle被设置为postgres_verbose，该语法也被用于间隔输出。

日、小时、分钟和秒的数量可以不适用显式的单位标记指定。例如，'1 12:59:10'被读作'1 day 12 hours 59 min 10 sec'。同样，一个年和月的组合可以使用一个横线指定，例如'200-10'被读作'200年10个月'（这些较短的形式事实上是SQL标准唯一许可的形式，并且在IntervalStyle被设置为sql_standard时用于输出）。

间隔值也可以被写成 ISO 8601 时间间隔，使用该标准4.4.3.2小节的“带标志符的格式”或者4.4.3.3小节的“替代格式”。带标志符的格式看起来像这样：

P quantity unit [ quantity unit ...] [ T [ quantity unit ...]]

该串必须以一个P开始，并且可以包括一个引入当日时间单位的T。可用的单位缩写在下表中给出。单位可以被忽略，并且可以以任何顺序指定，但是小于一天的单位必须出现在T之后。特别地，M的含义取决于它出现在T之前还是之后。

 ISO 8601 间隔单位缩写

| 缩写 | 含义                 |
| ---- | -------------------- |
| Y    | 年                   |
| M    | 月（在日期部分中）   |
| W    | 周                   |
| D    | 日                   |
| H    | 小时                 |
| M    | 分钟 (在时间部分中） |
| S    | 秒                   |

 

 


如果使用替代格式：

P [ years-months-days ] [ T hours:minutes:seconds ]

串必须以P开始，并且一个T分隔间隔的日期和时间部分。其值按照类似于 ISO 8601日期的数字给出。

在用一个域声明书写一个间隔常量时，或者为一个用域声明定义的间隔列赋予一个串时，对于为标记的量的解释依赖于域。例如INTERVAL '1' YEAR被解读成1年，而INTERVAL '1'表示1秒。同样，域声明允许的最后一个有效域“右边”的域值会被无声地丢弃掉。例如书写INTERVAL '1 day 2:03:04' HOUR TO MINUTE将会导致丢弃秒域，而不是日域。

根据SQL标准，一个间隔值的所有域都必须由相同的符号，这样一个领头的负号将会应用到所有域；例如在间隔文字'-1 2:03:04'中的负号会被应用于日、小时、分钟和秒部分。允许域具有不同的符号，并且在习惯上认为以文本表示的每个域具有独立的符号，因此在这个示例中小时、分钟和秒部分被认为是正值。如果IntervalStyle被设置为sql_standard，则一个领头的符号将被认为是应用于所有域（但是仅当没有额外符号出现）。否则将使用传统的解释。为了避免混淆，我们推荐在任何域为负值时为每一个域都附加一个显式的符号。

在冗长的输入格式中，以及在更紧凑输入格式的某些域中，域值可以有分数部分；例如'1.5 week'或'01:02:03.45'。这样的输入被转换为合适的月数、日数和秒数用于存储。当这样会导致月和日中的分数时，分数被加到低序域中，使用的转换因子是1月=30日和1日=24小时。例如，'1.5 month'会变成1月和15日。只有秒总是在输出时被显示为分数。

下表展示了一些有效interval输入的示例。

| 示例                                               | 描述                               |
| -------------------------------------------------- | ---------------------------------- |
| 1-2                                                | SQL标准格式：1年2个月              |
| 3 4:05:06                                          | SQL标准格式：3日4小时5分钟6秒      |
| 1 year 2 months 3 days 4 hours 5 minutes 6 seconds | 1年2个月3日4小时5分钟6秒钟         |
| P1Y2M3DT4H5M6S                                     | 带标志符的”ISO 8601 格式：含义同上 |
| P0001-02-03T04:05:06                               | ISO 8601 的“替代格式”：含义同上    |


## **时间/日期函数**

### **创建函数**

| 函数                                       | 描述                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| [make_date()](#_make_date())               | 从年、月和日字段创建日期                                     |
| [make_time()](#_make_time())               | 从小时、分钟和秒字段创建时间                                 |
| [make_timestamp()](#_make_timestamp())     | 从年、月、日、小时、分钟和秒字段创建时间戳                   |
| [make_timestamptz()](#_make_timestamptz()) | 从年，月，日，小时，分钟和秒字段，结合时区创建时间戳；如果没有指定timezone，则使用当前时区 |
| [to_timestamp()](#_to_timestamp())         | 将Unix纪元转换为带时区的时间戳(从1970-01-01 00:00:00+00开始的秒) |
| [make_interval()](#_make_interval())       | 从年、月、周、日、小时、分钟和秒字段创建时间间隔，每个字段默认为0 |

#### make_date()

目的：从年、月和日字段创建日期。

语法：

输入值:       year integer, month integer, day integer
返回值:      date

例如：

```
with c as (
  select make_date(year=>2019, month=>4, day=>22) as d)
select pg_typeof(d)::text as "type", d::text from c;             
```

返回如下信息：

```
 type |     d
------+------------
 date | 2019-04-22
```

使用负值的年份来产生BC结果，例如：

```
with c as (
  select make_date(year=>-10, month=>1, day=>31) as d)
select pg_typeof(d)::text as "type", d::text from c;
```

返回如下信息：

```
 type |       d
------+---------------
 date | 0010-01-31 BC
```

如果您指定了一个不存在的日期（例如零年或2月30日），则会出现以下错误： 

```
 ERROR:  date field value out of range
```

#### make_time()
目的：从小时、分钟和秒字段创建时间

语法：

输入值:      hour integer, min integer, sec double precision
返回值:      time without time zone

例如：

```
with c as (
  select make_time(hour=>13, min=>25, sec=>20.123456) as t)
select pg_typeof(t)::text as "type", t::text from c;
```

返回如下信息：

```
          type       |        t
------------------------------+--------------------
 time without time zone | 13:25:20.123456
```

如果您指定了一个不存在的时间（例如，25:00:00），则会出现以下错误：

```
ERROR:  time field value out of range
```

#### make_timestamp()
目的：从年、月、日、小时、分钟和秒字段创建时间戳

语法：

输入值:      year integer, month integer, mday integer, hour integer, min integer, sec double precision
返回值:      timestamp without time zone

例如：

```
 with c as (
  select make_timestamp(year=>2019, month=>4, mday=>22, hour=>13, min=>25, sec=>20.123456) as ts)
select pg_typeof(ts)::text as "type", ts::text from c;
```

返回如下信息：

```
            type         |             ts
-------------------------------------+----------------------------------
 timestamp without time zone | 2019-04-22 13:25:20.123456
```


#### make_timestamptz()
目的：从年，月，日，小时，分钟和秒字段，结合时区创建时间戳；如果没有指定timezone，则使用当前时区

语法：

输入值:      year integer, month integer, mday integer, hour integer, min integer, sec double precision , [timezone text]

返回值:      timestamp with time zone

例如：

```
set timezone = 'UTC';
with c as (
  select make_timestamptz(year=>2019, month=>6, mday=>22, hour=>13, min=>25, sec=>20.123456, timezone=>'Europe/Helsinki') as tstz)
select pg_typeof(tstz)::text as "type", tstz::text from c;
```

返回如下信息：

```
           type         |             tstz
-----------------------------------+----------------------------------------
 timestamp with time zone  | 2019-06-22 10:25:20.123456+00
```

#### to_timestamp()

目的：将Unix纪元转换为带时区的时间戳(从1970-01-01 00:00:00+00开始的秒)

to_timestamp()函数有两个重载，都返回一个时间戳值。

语法：

输入值:       double precision
返回值:      timestamp with time zone

或者：

输入值:       text, text
返回值:      timestamp with time zone

例如：

```
set timezone = 'UTC';
with c as (
  -- 100 days after the start of the Unix epoch.
  select to_timestamp((60*60*24*1000)::double precision) as t)
select pg_typeof(t)::text as "type", t::text from c;
```

返回如下信息：

```
           type        |           t
---------------------------------+----------------------------
 timestamp with time zone | 1972-09-27 00:00:00+00
```

#### make_interval()
目的：从年、月、周、日、小时、分钟和秒字段创建时间间隔，每个字段默认为0

语法：

输入值:        years integer, months integer, weeks integer, days integer, hours integer, mins integer, secs double precision
返回值:      interval         

例如：

```
with c as (
  select make_interval(secs=>250000.123456) as i)
select pg_typeof(i)::text as "type", i::text from c;
```

返回如下信息：

```
   type  |        i
------------+-----------------
 interval | 69:26:40.123456
```

### **操作函数**

| 函数                                       | 描述                                           |
| ------------------------------------------ | ---------------------------------------------- |
| [date_trunc()](#_date_trunc())             | 按照指定的粒度截断date_time值                  |
| [justify_days()](#_justify_days())         | 调整间隔，使得30天时间周期表示为月             |
| [justify_hours()](#_justify_hours())       | 调整时间间隔，使得24小时时间周期表示为日       |
| [justify_interval()](#_justify_interval()) | 使用 justify_days 和 justify_hours调整时间间隔 |

#### date_trunc()
目的：按照指定的粒度截断date_time值

date_trunc()函数有三个重载。

语法：

输入值:       text, interval
返回值:      interval     

或者

输入值:       text, timestamp with time zone
返回值:      interval     

或者

输入值:       text, timestamp without time zone
返回值:      interval     

第一个参数text的有效值为：
microseconds
milliseconds
second
minute
hour
day
week
month
quarter
year
decade
century
millennium

例如：

```
set timezone = 'UTC';
with c as (
  select date_trunc('year', '2021-07-19'::date) as v)
select pg_typeof(v)::text as "type", v::text from c;
```

返回如下信息：

```
           type        |           v
--------------------------------+--------------------------------
 timestamp with time zone | 2021-01-01 00:00:00+00
```

当输入值的类型为timestamp with time zone时。截断是针对特定时区进行的。 例如，截断为day，产生的值是 是该区域的午夜。 默认情况下，截断是在以下方面进行的 到当前的TimeZone设置，但在当前的 可以提供可选的time_zone参数。以指定不同的时区。 

当处理timestamp without time zone 或interval输入时，不能指定时区。 这些总是按表面值来处理。

示例 (假设当地时区为 America/New_York):

```
select date_trunc('hour', TIMESTAMP '2001-02-16 20:38:40');
```

返回信息如下：

```
     date_trunc      
---------------------------
 2001-02-16 20:00:00
select date_trunc('year', TIMESTAMP '2001-02-16 20:38:40');
```

返回信息如下：

```
     date_trunc      
---------------------------
 2001-01-01 00:00:00
select date_trunc('day', TIMESTAMP WITH TIME ZONE '2001-02-16 20:38:40+00');
```

返回信息如下：

```
       date_trunc       
------------------------------
 2001-02-16 00:00:00-05
select date_trunc('hour', INTERVAL '3 days 02:47:33');
```

返回信息如下：

```
   date_trunc    
---------------------
 3 days 02:00:00
```

#### justify_days()
目的：调整间隔，使得30天时间周期表示为月
justify_days()函数，通过减去30天周期的适当整数，来“标准化”[mm, dd, ss]表示的dd字段的值，从而得到的dd值小于30天（但不小于零）。使用一个30天周期与1个月相同的规则，将减去的30天周期转换为月，并添加到mm字段的值中。

语法：

输入值:       interval         
返回值:      interval  

例如：

```
select justify_days('4 months 31 days'::interval); 
```

返回信息如下：

```
5 mons 1 day
```


#### justify_hours()
目的：调整时间间隔，使得24小时时间周期表示为日

语法：

输入值:       interval         
返回值:      interval         

例如：

```
select justify_hours('4 days 25 hours'::interval);
```

返回信息如下：

```
5 days 01:00:00
```


#### justify_interval()
目的：使用 justify_days 和 justify_hours调整时间间隔；

语法：

输入值:       interval         
返回值:      interval         

例如：

```
select justify_interval('4 months 31 days 25 hours'::interval);
```

返回信息如下：

```
5 mons 2 days 01:00:00
```

### **当前日期/时间函数**

| 函数                                                 | 描述                                                         |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| [current_date](#_current_date)                       | 当前日期(当前事务的开始）                                    |
| [current_time()](#_current_time())                   | 一天中的当前时间(当前事务的开始）                            |
| [current_timestamp()](#_current_timestamp)           | 当前日期和时间(当前事务的开始)；                             |
| [localtime()](#_localtime())                         | 一天中当前时间(当前事务的开始）                              |
| [localtimestamp()](#_localtimestamp())               | 当前日期和时间(当前事务的开始）                              |
| [transaction_timestamp()](#_transaction_timestamp()) | 当前日期和时间(当前事务的开始)                               |
| [now()](#_now())                                     | 当前日期和时间(当前事务的开始)；                             |
| [statement_timestamp()](#_statement_timestamp())     | 当前日期和时间(当前语句的开始)                               |
| [clock_timestamp()](#_clock_timestamp())             | 当前日期和时间（在语句执行期间变化）                         |
| [timeofday()](#_timeofday())                         | 当前的日期和时间（类似 clock_timestamp, 但是采用 text 字符串） |

#### current_date
目的：返回当前日期(当前事务的开始），它在同一事务内的连续调用中返回相同的值。 

语法：

返回值:      date

例如：

```
select current_date;
```

#### current_time()
目的：一天中的当前时间(当前事务的开始），它在同一事务内的连续调用中返回相同的值，值带时区。可以有选择地接受一个精度参数，返回指定小数位的精度秒值。如果没有精度参数，结果将被给予所能得到的全部精度。

语法：

输入值:       precision
返回值:      timetz

例如：

```
select current_time;
 
select current_time(2);
```

#### current_timestamp
目的：当前日期和时间(当前事务的开始)，它在同一事务内的连续调用中返回相同的值，值带时区。可以有选择地接受一个精度参数，返回指定小数位的精度秒值。如果没有精度参数，结果将被给予所能得到的全部精度。

语法：

输入值:       precision
返回值:      timestamptz

例如：

```
select current_timestamp;
 
select current_timestamp(2);
```

#### localtime()
目的：返回一天中当前时间(当前事务的开始），它在同一事务内的连续调用中返回相同的值，值不带时区。可以有选择地接受一个精度参数，返回指定小数位的精度秒值。如果没有精度参数，结果将被给予所能得到的全部精度。

语法：

输入值:       precision
返回值:      time

例如：

```
select localtime;
 
select localtime(2);
```

#### localtimestamp()
目的：当前日期和时间(当前事务的开始），它在同一事务内的连续调用中返回相同的值，值不带时区。可以有选择地接受一个精度参数，返回指定小数位的精度秒值。如果没有精度参数，结果将被给予所能得到的全部精度。

语法：

输入值:       precision
返回值:      timestamp

例如：

```
select localtimestamp;
 
select localtimestamp(2);
```


#### transaction_timestamp()
目的：当前日期和时间(当前事务的开始)，它在同一事务内的连续调用中返回相同的值，值带时区。可以有选择地接受一个精度参数，返回指定小数位的精度秒值。如果没有精度参数，结果将被给予所能得到的全部精度。

transaction_timestamp()等价于CURRENT_TIMESTAMP


语法：

返回值:      timestamptz

例如：

```
select transaction_timestamp();
```

#### now()
目的：当前日期和时间(当前事务的开始)，它在同一事务内的连续调用中返回相同的值，值带时区。可以有选择地接受一个精度参数，返回指定小数位的精度秒值。如果没有精度参数，结果将被给予所能得到的全部精度。

now()等效于transaction_timestamp()。

语法：

返回值:      timestamptz

例如：

```
select now();
```


#### statement_timestamp()
目的：当前日期和时间(当前语句的开始)，返回当前语句的开始时刻（更准确的说是收到客户端最后一条命令的时间）。

语法：

返回值:      timestamptz

例如：

```
select statement_timestamp();
```

#### clock_timestamp()
目的：当前日期和时间（在语句执行期间变化），clock_timestamp()返回真正的当前时间，值带时区，它的值甚至在同一条 SQL 命令中都会变化。

语法：

返回值:      timestamptz

例如：

```
select clock_timestamp();
```


#### timeofday()
目的：和clock_timestamp()相似，timeofday()也返回真实的当前时间，但是它的结果是一个格式化的text串，而不是timestamp with time zone值。 

语法：

返回值:      text

例如：

```
select timeofday();
```

### **延时执行函数**

| 函数                                   | 描述                                                         |
| -------------------------------------- | ------------------------------------------------------------ |
| [pg_sleep()](#_pg_sleep())             | 使当前会话的进程休眠，直到过去给定的秒数。可以指定几分之一秒的延迟。 |
| [pg_sleep_for()](#_pg_sleep_for())     | 允许将睡眠时间指定为时间间隔。                               |
| [pg_sleep_until()](#_pg_sleep_until()) | 用于需要特定的唤醒时间。                                     |

#### pg_sleep()
目的：使当前会话的进程休眠，直到过去给定的秒数。可以指定几分之一秒的延迟。

语法：

输入值:       double  precision
返回值:      void

例如：

```
select pg_sleep(1.5);
```


#### pg_sleep_for()
目的：允许将睡眠时间指定为时间间隔。

语法：

输入值:       interval
返回值:      void

例如：

```
select pg_sleep_for('5 minutes');
```

#### pg_sleep_until()
目的：用于需要特定的唤醒时间。

语法：

输入值:       timestamp with time zone 
返回值:      void

例如：

```
select pg_sleep_until('tomorrow 03:00');
```

### **其它函数和操作符**

| 函数和操作符                                         | 描述                                                         |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| [isfinite()](#_isfinite())                           | 测试有限日期，时间戳，时间间隔                               |
| [age()](#_age())                                     | 减去参数，生成一个使用年和月，而不是只用日的结果             |
| [timezone()](#_timezone())                           | 效果与在使用set时区指定所需时区后，从一种数据类型到另一种使用简单的类型转换相同 |
| [extract() / date_part()](#_extract() / date_part()) | 从日期/时间值中抽取子域                                      |
| [OVERLAPS](#_OVERLAPS)                               | 支持 SQL 操作符OVERLAPS：(start1, end1) OVERLAPS (start2, end2)(start1, length1) OVERLAPS (start2, length2) 这个表达式在两个时间域（用它们的端点定义）重叠的时候得到真，当它们不重叠时得到假。 |

#### isfinite()
目的：测试有限日期，时间戳，时间间隔

语法：

输入值:      abstime
 | date
 | interval 
| timestamp with time zone 
| timestamp without time zone
返回值:      boolean

例如：

```
select isfinite(date '2001-02-16');
 
select isfinite(timestamp 'infinity');
 
select isfinite(interval '4 hours');
```

#### age()
目的：减去参数，生成一个使用年和月，而不是只用日的结果。

语法：

输入值:  timestamp without time zone, timestamp without time zone 
          |  timestamp with time zone, timestamp with time zone
          |  timestamp without time zone
          |  timestamp with time zone
返回值:  interval

例如：

```
select  age(timestamp '2001-04-10', timestamp '1957-06-13');
 
select age(timestamp '1957-06-13');
```

#### timezone()
目的：效果与在使用set时区指定所需时区后，从一种数据类型到另一种使用简单的类型转换相同。

语法：

输入值:  interval, time with time zone  
返回值:  time with time zone

或者

输入值:  interval, timestamp with time zone  
返回值:  timestamp without time zone

或者

输入值:  interval, timestamp without time zone  
返回值:  timestamp with time zone

或者

输入值:  text, time with time zone  
返回值:  time with time zone

或者

输入值:  text, timestamp with time zone  
返回值:  timestamp without time zone

或者

输入值:  text, timestamp without time zone  
返回值:  timestamp with time zone

例如：

```
with c as (
  select '2021-09-22 13:17:53.123456 Europe/Helsinki'::timestamptz as tstz)
select
  (timezone('UTC',           tstz) = tstz at time zone 'UTC'          )::text as "with timezone given as text",
  (timezone(make_interval(), tstz) = tstz at time zone make_interval())::text as "with timezone given as interval"
from c;
```

返回信息如下：

```
 with timezone given as text | with timezone given as interval
-----------------------------------+---------------------------------------
 true                   | true
```

#### extract() / date_part()
目的：从日期/时间值中抽取子域，主要的用途是做计算性处理。对于用于显示的日期/时间值格式化，date_part函数等价于SQL标准函数extract。
两者使用语法为：

date_part('field', source)

EXTRACT(field FROM source)

source必须是一个类型 timestamp、time或interval的值表达式（类型为date的表达式将被造型为 timestamp，并且因此也可以被同样使用）。field是一个标识符或者字符串，它指定从源值中抽取的域。extract函数返回类型为double precision的值。有效的域名字见下表所示∶

| 域名            | 描述及示例                                                   |
| --------------- | ------------------------------------------------------------ |
| century         | 世纪select EXTRACT(CENTURY FROM TIMESTAMP '2000-12-16 12:21:13'); |
| day             | 对于timestamp值，是（月份）里的日域（1–31）；对于interval值，是日数select EXTRACT(DAY FROM TIMESTAMP '2001-02-16 20:38:40');select EXTRACT(DAY FROM INTERVAL '40 days 1 minute'); |
| decade          | 年份域除以10select EXTRACT(DECADE FROM TIMESTAMP '2001-02-16 20:38:40'); |
| dow             | 一周中的日，从周日（0）到周六（6）select EXTRACT(DOW FROM TIMESTAMP '2001-02-16 20:38:40');请注意，extract的一周中的日和to_char(..., 'D')函数不同。 |
| doy             | 一年的第几天（1–365/366）select EXTRACT(DOY FROM TIMESTAMP '2001-02-16 20:38:40'); |
| epoch           | 对于timestamp with time zone值， 是自 1970-01-01 00:00:00 UTC 以来的秒数（结果可能是负数）； 对于date and timestamp值，是自本地时间 1970-01-01 00:00:00 以来的描述；对于interval值，它是时间间隔的总秒数。select EXTRACT(EPOCH FROM TIMESTAMP WITH TIME ZONE '2001-02-16 20:38:40.12-08');select EXTRACT(EPOCH FROM INTERVAL '5 days 3 hours'); |
| hour            | 小时域（0–23）select EXTRACT(HOUR FROM TIMESTAMP '2001-02-16 20:38:40'); |
| isodow          | 一周中的日，从周一（1）到周日（7）select EXTRACT(ISODOW FROM TIMESTAMP '2001-02-18 20:38:40');除了周日，这和dow相同。这符合ISO 8601 中一周中的日的编号。 |
| isoyear         | 日期所落在的ISO 8601 周编号的年（不适用于间隔），每一个ISO 8601 周编号的年都开始于包含1月4日的那一周的周一select EXTRACT(ISOYEAR FROM DATE '2006-01-01');select EXTRACT(ISOYEAR FROM DATE '2006-01-02'); |
| microseconds    | 秒域，包括小数部分，乘以 1,000,000。请注意它包括全部的秒select EXTRACT(MICROSECONDS FROM TIME '17:12:28.5'); |
| millennium      | 千年select EXTRACT(MILLENNIUM FROM TIMESTAMP '2001-02-16 20:38:40'); |
| milliseconds    | 秒域，包括小数部分，乘以 1000。请注意它包括完整的秒。select EXTRACT(MILLISECONDS FROM TIME '17:12:28.5'); |
| minute          | 分钟域（0–59）select EXTRACT(MINUTE FROM TIMESTAMP '2001-02-16 20:38:40'); |
| month           | 对于timestamp值，它是一年里的月份数（1–12）； 对于interval值，它是月的数目，然后对 12 取模（0–11）select EXTRACT(MONTH FROM TIMESTAMP '2001-02-16 20:38:40');select EXTRACT(MONTH FROM INTERVAL '2 years 13 months'); |
| quarter         | 该天所在的该年的季度（1–4）select EXTRACT(QUARTER FROM TIMESTAMP '2001-02-16 20:38:40'); |
| second          | 秒，包括任何小数秒。select EXTRACT(SECOND FROM TIMESTAMP '2001-02-16 20:38:40');select EXTRACT(SECOND FROM TIME '17:12:28.5'); |
| timezone        | 与 UTC 的时区偏移，以秒记。正数对应 UTC 东边的时区，负数对应 UTC 西边的时区。 |
| timezone_hour   | 时区偏移的小时部分。                                         |
| timezone_minute | 时区偏移的分钟部分。                                         |
| week            | 该天在所在的ISO 8601 周编号的年份里是第几周。根据定义， 一年的第一周包含该年的 1月 4 日并且 ISO 周从星期一开始。换句话说，一年的第一个星期四在第一周。在 ISO 周编号系统中，早的 1 月的日期可能位于前一年的第五十二或者第五十三周，而迟的 12 月的日期可能位于下一年的第一周。例如， 2005-01-01位于 2004 年的第五十三周，并且2006-01-01位于 2005 年的第五十二周，而2012-12-31位于 2013 年的第一周。我们推荐把isoyear域和week一起使用来得到一致的结果。select EXTRACT(WEEK FROM TIMESTAMP '2001-02-16 20:38:40'); |
| year            | 年份域。要记住这里没有0 AD，所以从AD年里抽取BC年应该小心处理。select EXTRACT(YEAR FROM TIMESTAMP '2001-02-16 20:38:40'); |
| julian          | 与日期或时间戳[tz]值相对应的儒略日期（间隔值不支持）。不是本地午夜的timestamp[tz]值会产生一个分数值。 |

#### OVERLAPS
支持 SQL 操作符OVERLAPS：

(start1, end1) OVERLAPS (start2, end2)
(start1, length1) OVERLAPS (start2, length2)

这个表达式在两个时间域（用它们的端点定义）重叠的时候得到真，当它们不重叠时得到假。端点可以用一对日期、时间或者时间戳来指定；或者是用一个后面跟着一个间隔的日期、时间或时间戳来指定。当一对值被提供时，起点或终点都可以被写在前面，OVERLAPS会自动地把较早的值作为起点。每一个时间段被认为是表示半开的间隔start <= time < end，除非start和end相等，这种情况下它表示单个时间实例。例如这表示两个只有一个共同端点的时间段不重叠。

例如：

```
select (DATE '2001-02-16', DATE '2001-12-21') OVERLAPS
       (DATE '2001-10-30', DATE '2002-10-30');
```


## **格式化函数**

本小结介绍所有日期时间的格式化功能，包括日期时间值格式化到文本值，和文本值格式化到日期时间值。

### **格式化函数**

格式化函数提供一套强大的工具用于把各种数据类型 （日期/时间、整数、浮点、数字） 转换成格式化的字符串，以及反过来从格式化的字符串转换成指定的数据类型。下表列出了这些函数。
这些函数都遵循一个公共的调用规范： 第一个参数是待格式化的值，而第二个是一个定义输出或输入格式的模板。

| 函数           | 描述                                           |
| -------------- | ---------------------------------------------- |
| to_char()      | 根据给定的格式将时间戳，时间间隔转换为字符串。 |
| to_date()      | 根据给定的格式将字符串转换为日期。             |
| to_timestamp() | 根据给定的格式将字符串转换为时间戳。           |

#### to_char()
目的：根据给定的格式将时间戳，时间间隔转换为字符串。to_char函数的有关时间，日期格式化的重载函数语法如下：

语法：

输入值:  interval, text  
返回值:  text

或者

输入值:  timestamp with time zone, text  
返回值:  text

或者

输入值:  timestamp without time zone, text  
返回值:  text

例如：

```
select to_char(timestamp '2002-04-20 17:31:12.66', 'HH12:MI:SS');
 
select to_char(interval '15h 2m 12s', 'HH24:MI:SS');
```

请注意，没有用于将日期值或时间值转换为文本值的重载。

#### to_date()
目的：根据给定的格式将字符串转换为日期。

语法：

输入值:  text, text
返回值:  date  

例如：

```
select to_date('05 Dec 2000', 'DD Mon YYYY');
```

#### to_timestamp()
目的：根据给定的格式将字符串转换为时间戳。

语法：

输入值:  text, text 
返回值:  timestamp with time zone

例如：

```
select to_timestamp('05 Dec 2000', 'DD Mon YYYY');
```

### **日期-时间模板模式**

下表列出了支持的不同模板模式。

| 模式                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| HH                       | 一天中的小时(01–12)                                          |
| HH12                     | 一天中的小时(01–12)                                          |
| HH24                     | 一天中的小时 (00–23)                                         |
| MI                       | 分钟 (00–59)                                                 |
| SS                       | 秒 (00–59)                                                   |
| MS                       | 毫秒 (000–999)                                               |
| US                       | 微秒 (000000–999999)                                         |
| SSSS                     | 午夜后的秒 (0–86399)                                         |
| AM, am, PM, pm           | 正午指示器（不带句号）                                       |
| A.M., a.m., P.M. , p.m.  | 正午指示器（带句号）                                         |
| Y,YYY                    | 带逗号的年（4 位或者更多位） with comma                      |
| YYYY                     | 年（4 位或者更多位）                                         |
| YYY                      | 年的最后 3 位数字                                            |
| YY                       | 年的最后 2 位数字                                            |
| Y                        | 年的最后 1 位数字                                            |
| IYYY                     | ISO 8601 周编号方式的年（4 位或更多位）                      |
| IYY                      | ISO 8601 周编号方式的年的最后 3 位数字                       |
| IY                       | ISO 8601 周编号方式的年的最后 2 位数字                       |
| I                        | ISO 8601 周编号方式的年的最后 1 位数字                       |
| BC, bc, AD 或 ad         | 纪元指示器（不带句号）                                       |
| B.C., b.c., A.D. 或 a.d. | 纪元指示器（带句号）                                         |
| MONTH                    | 全大写形式的月名（空格补齐到 9 字符）                        |
| Month                    | 全首字母大写形式的月名（空格补齐到 9 字符）                  |
| month                    | 全小写形式的月名（空格补齐到 9 字符）                        |
| MON                      | 简写大写形式的月名（英文 3 字符，本地化长度可变）            |
| Mon                      | 简写首字母大写形式的月名（英文 3 字符，本地化长度可变）      |
| mon                      | 简写的小写形式的月名（英文 3 字符，本地化长度可变）          |
| MM                       | 月编号 (01–12)                                               |
| DAY                      | 全大写形式的日名（空格补齐到 9 字符）                        |
| Day                      | 全首字母大写形式的日名（空格补齐到 9 字符）                  |
| day                      | 全小写形式的日名（空格补齐到 9 字符）                        |
| DY                       | 简写大写形式的日名（英语 3 字符，本地化长度可变）            |
| Dy                       | 简写首字母大写形式的日名（英语 3 字符，本地化长度可变）      |
| dy                       | 简写的小写形式的日名（英语 3 字符，本地化长度可变）          |
| DDD                      | 一年中的日(001–366)                                          |
| IDDD                     | ISO 8601周编号方式的年中的日（001–371;年的第1日时第一个ISO周的周一） |
| DD                       | 月中的日 (01–31)                                             |
| D                        | 周中的日，周日 (1) 到周六 (7)                                |
| ID                       | 周中的 ISO 8601 日，周一 (1) 到周日 (7)                      |
| W                        | 月中的周 (1–5) （第一周从该月的第一天开始）                  |
| WW                       | 年中的周数 (1–53) （第一周从该年的第一天开始）               |
| IW                       | ISO 8601周编号方式的年中的周数(01–53; 新的一年的第一个周四在第一周) |
| CC                       | 世纪（2 位数）（21 世纪开始于 2001-01-01）                   |
| J                        | 儒略日（从午夜 UTC 的公元前 4714 年 11 月 24 日开始的整数日数） |
| Q                        | 季度                                                         |
| RM                       | 大写形式的罗马计数法的月 (I–XII; I=一月)                     |
| rm                       | 小写形式的罗马计数法的月 (i–xii; i=一月)                     |
| TZ                       | 大写形式的时区缩写（仅在to_char中支持）                      |
| tz                       | 小写形式的时区缩写（仅在to_char中支持）                      |
| TZH                      | 时区的小时                                                   |
| TZM                      | 时区的分钟                                                   |
| OF                       | 从UTC开始的时区偏移（仅在to_char中支持）                     |

修饰语可以被应用于模板模式来修改它们的行为。例如，FMMonth就是带着FM修饰语的Month模式。下表展示了可用于日期/时间格式化的修饰语模式。

| 修饰语    | 描述                                        | 示例             |
| --------- | ------------------------------------------- | ---------------- |
| FM prefix | 填充模式（抑制前导零和填充的空格）          | FMMonth          |
| TM prefix | 翻译模式（基于lc_time使用本地化的日和月名） | TMMonth          |
| TH suffix | 大写形式的序数后缀                          | DDTH, e.g., 12TH |
| th suffix | 小写形式的序数后缀                          | DDth, e.g., 12th |
| FX prefix | 固定的格式化全局选项                        | FX Month DD Day  |

日期/时间格式化的使用须知：

* FM修饰符
  FM抑制前导零和尾随空格，否则这些空格将被添加，以使输出为固定宽度。例如：

```
with c as (select '0020-05-03 BC'::timestamp as t)
  select
    to_char(t, 'MMth "month ("Month")", DDth "day", YYYY AD')          as "plain",
    to_char(t, 'FMMMth "month ("FMMonth")", FMDDth "day", FMYYYY AD')  as "using FM"
from c;
```

返回信息如下：

```
              plain                   |            using FM
-------------------------------------------------------+-----------------------------------------
05th month (May      ), 03rd day, 0020 BC | 5th month (May), 3rd day, 20 BC
```

* TM修饰符
  TM修饰符仅仅影响使用to_char()，以文本展示完整和缩写的日期和月份名称，因为这些字段不会影响到日期时间的结果值，所以to_date()和to_timestamp()会忽略TM。TM不仅决定了用于日、月名称和缩写的国家语言，而且还具有抑制尾随空格的副作用。请注意，TM对序数后缀的呈现方式没有影响。例如：

```
deallocate all;
prepare stmt as
select
  to_char('2021-02-01'::timestamp, 'Day, ddth Month, y,yyy') as "plain",
  to_char('2021-02-01'::timestamp, 'TMDay, ddth TMMonth, y,yyy') as "with TM";
 
set lc_time = 'ZH_CN.utf8';
execute stmt;
```

返回信息如下：

```
              plain           |         with TM          
--------------------------------------------+----------------------------------
 Monday   , 01st February , 2,021 | 星期一, 01st 二月, 2,021
```

注：对于Unix操作系统，可以使用命令：locale -a来查看支持列表，来设置有效值。

* FX修饰符
  to_timestamp和to_date跳过了输入字符串开头和日期和时间值周围的多个空格，除非使用了FX选项。例如：

```
set timezone = 'UTC';
select to_timestamp('2000    JUN', 'YYYY MON')::text;
```

可以工作，返回信息如下：

```
2000-06-01 00:00:00+00
```

但是如下返回一个错误，因为to_timestamp只期望一个空格。

```
set timezone = 'UTC';
select to_timestamp('2000    JUN', 'FXYYYY MON')::text;。
```

* 只有YYY、YY或Y的模板模式
  使用to_date()和to_timestamp()，如果要转换的文本值中，假定的“years”子字符串少于四位数字，并且模板模式为YYY、YY或Y，则“years（年份）”子字符串用数字填充，以生成四位字符串，年份将被调整为最接近于 2020 年。

例如：

```
with c as (
  select '15-Jun-100'::text as t1, '15-Jun-999'::text as t2)
select
  to_date(t1, 'dd-Mon-YYYY') ::text  as "t1 using YYYY",
  to_date(t2, 'dd-Mon-YYYY') ::text  as "t2 using YYYY",
 
  to_date(t1, 'dd-Mon-YYY')  ::text  as "t1 using YYY",
  to_date(t2, 'dd-Mon-YYY')  ::text  as "t2 using YYY"
from c;
```

返回信息如下：

```
 t1 using YYYY | t2 using YYYY | t1 using YYY | t2 using YYY
---------------------+-------------------+-----------------+-----------------
 0100-06-15    | 0999-06-15    | 2100-06-15  | 1999-06-15 
```

请注意，如果您想将“6781123”::text解释为“0678-11-23”::date，这将带来挑战。试着执行如下示例：

```
select
  to_date('6781123',   'YYYYMMDD')   as "1st try",
  to_date('6781123',   'YYYMMDD')    as "2nd try",
  to_date('678-11-23', 'YYYY-MM-DD') as "workaround";
```

返回信息如下：

```
  1st try    |  2nd try  | workaround
----------------+--------------+---------------
 6781-12-03 | 1678-11-23 | 0678-11-23
```

无论是第一次，还是第二次尝试都不是你想要的东西。您唯一的选择是，要求输入解释为YYYY、MM和DD的子字符串之间有分隔符，如解决方法所示。

* 解释“years”子字符串超过四位的文本值
  您可以使用to_date()和to_timestamp()将类似“21234-1123”的文本值与模板“YYYY-MMDD”进行正确转换。您还可以使用to_date()和to_timestamp()将类似“21231123”的文本值转换为模板“YYYYMMDD”，而不会出现错误。试着执行如下示例：

```
select
  to_date('21234-1123', 'YYYY-MMDD') ::text as d1,
  to_date('21231123', 'YYYYMMDD')    ::text as d2;
```

返回信息如下：

```
      d1     |     d2
-------------------+--------------
  21234-11-23 | 2123-11-23
```

但是，您不能简单使用模板，使用to_date()和to_timestamp()来转换类似“212341123”的文本值。如下所示：

```
select to_date('212341123', 'YYYYMMDD');
```

这将会返回如下错误信息：

```
date/time field value out of range: "212341123"
```

即使用一个额外的Y把“YYYYMMDD”改成“YYYYYMDDD”，也仍然会报告如下错误信息： 

```
conflicting values for "Y" field in formatting string
This value contradicts a previous setting for the same field type.
```

您唯一的正确选择是，确保要转换的文本值中的“years”子字符串，以非数字字符结束，并且模板与之前使用的代码示例to_date('21234-1123'，'YYYY-MMDD')中的字符匹配。

* CC模板模式
  使用to_date()和to_timestamp()，可以接受CC模板模式，但如果存在YYYY、YYYY或Y、YYY模式，则会忽略该模式。如果CC与YY或Y模式一起使用，则计算结果为指定世纪中的年份。如果指定了世纪，但没有指定年份，则假定为世纪的第一年。 
  例如：

```
select
  to_date('19 2021', 'CC YYYY') as "result 1",
  to_date('19 21', 'CC YY')     as "result 2",
  to_date('19', 'CC')           as "result 3";
```

返回信息如下：

```
  result 1   |  result 2   |  result 3
----------------+----------------+--------------
 2021-01-01 | 1821-01-01  | 1801-01-01
```

* 普通文本
  在to_char模板里可以有普通文本，并且它们会被照字面输出。你可以把一个子串放到双引号里强迫它被解释成一个文本，即使它里面包含模板模式也如此。例如，在 '"Hello Year "YYYY'中，YYYY将被年份数据代替，但是Year中单独的Y不会。在to_date、to_number以及to_timestamp中，文本和双引号字符串会导致跳过该字符串中所包含的字符数量，例如"XX"会跳过两个输入字符（不管它们是不是XX）。
  例如：

```
select to_char('2021, 06-15'::date, '"Hello Year "YYYY on Dy ddth Mon') as d1;
```

返回信息如下：

```
               d1                
-----------------------------------------
 Hello Year 2021 on Tue 15th Jun
```

* 输出里有双引号
  如果你想在输出里有双引号，那么你必须在它们前面放反斜线，例如 '\"YYYY Month\"'。不然，在双引号字符串外面的反斜线就不是特殊的。在双引号字符串内，反斜线会导致下一个字符被取其字面形式，不管它是什么字符（但是这没有特殊效果，除非下一个字符是一个双引号或者另一个反斜线）。
  例如：

```
select to_char('2021, 06-15'::date, '\"Hello "Y"ear\"YYYY on Dy ddth Mon') as d1;
```

返回信息如下：

```
                d1                
-------------------------------------------
 "Hello Year"2021 on Tue 15th Jun
```

* 在to_timestamp和to_date中，负的年份被视为表示BC。 如果你同时写一个负的年份和一个显式的BC字段，你又会得到AD。第0年的输入被视为公元前1年。
* 在to_timestamp和to_date中，工作日名称或编号（DAY、D以及相关的字段类型）会被接受，但会为了计算结果的目的而忽略。季度（Q）字段也是一样。
* 在to_timestamp和to_date中，一个 ISO 8601 周编号的日期（与一个格里高利日期相区别）可以用两种方法之一被指定为to_timestamp和to_date：
  1）年、周编号和工作日：例如to_date('2006-42-4', 'IYYY-IW-ID')返回日期2006-10-19。如果你忽略工作日，它被假定为 1（周一）。
  2）年和一年中的日：例如to_date('2006-291', 'IYYY-IDDD')也返回2006-10-19。

尝试使用一个混合了 ISO 8601 周编号和格里高利日期的域来输入一个日期是无意义的，并且将导致一个错误。在一个 ISO 周编号的年的环境下，一个“月”或“月中的日”的概念没有意义。在一个格里高利年的环境下，ISO 周没有意义。用户应当避免混合格里高利和 ISO 日期声明。

小心
虽然to_date将会拒绝混合使用格里高利和 ISO 周编号日期的域， to_char却不会，因为YYYY-MM-DD (IYYY-IDDD) 这种输出格式也会有用。但是避免写类似IYYY-MM-DD的东西，那会得到在起始年附近令人惊讶的结果。

* 在to_timestamp中，毫秒（MS）和微秒（US）域都被用作小数点后的秒位。例如to_timestamp('12.3', 'SS.MS')不是 3 毫秒, 而是 300，因为该转换把它看做 12 + 0.3 秒。这意味着对于格式SS.MS而言，输入值12.3、12.30和12.300指定了相同数目的毫秒。要得到三毫秒，你必须使用 12.003，转换会把它看做 12 + 0.003 = 12.003 秒。

下面是一个更复杂的示例∶to_timestamp('15:12:02.020.001230', 'HH24:MI:SS.MS.US')是 15 小时、12 分钟和 2 秒 + 20 毫秒 + 1230微秒 = 2.021230 秒。

* to_char(..., 'ID')的一周中日的编号匹配extract(isodow from ...)函数，但是to_char(..., 'D')不匹配extract(dow from ...)的日编号。
* to_char(interval)格式化HH和HH12为显示在一个 12 小时的时钟上，即零小时和 36 小时输出为12，而HH24会输出完整的小时值，对于间隔它可以超过 23。
