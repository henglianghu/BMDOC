根据RFC 7159 中的说明，JSON 数据类型是用来存储 JSON（JavaScript Object Notation）数据的。
BSQL支持两种数据类型来表示JSON文档：json 和jsonb。这两种数据类型都拒绝任何不符合RFC 7159的JSON文档。json数据类型存储JSON 文档的文本表示。相反，jsonb数据类型以适当的内部格式存储子值的文档层次结构的解析表示。当然，将JSON文档存储为jsonb值比存储为JSON值需要更多的计算。当使用本节中描述的运算符和函数对子值进行运算时，将付出一定的成本。
json 和 jsonb数据类型接受几乎完全相同的值集合作为输入。 主要的实际区别之一是效率。json数据类型存储输入文本的精准拷贝，处理函数必须在每 次执行时必须重新解析该数据。而jsonb数据被存储在一种分解好的 二进制格式中，它在输入时要稍慢一些，因为需要做附加的转换。但是 jsonb在处理时要快很多，因为不需要解析。jsonb也支持索引，这也是一个令人瞩目的优势。
JSON是作为一种数据交换格式被发明的，最初是为了允许JavaScript程序中的任意复合值被序列化，作为文本传输，然后在另一个JavaScript程序中忠实地反序列化，以重新实例化原始复合值。后来，许多其他编程语言（现在包括SQL和PL/pgSQL）支持JSON的序列化和反序列化。此外，将JSON作为记录的持久表示存储在一个表中变得很常见，该表只有一个主键列和一个JSON或jsonb列，用于可以在符合关系模型的表设计中以经典方式表示的事实。这种模式最初出现在NoSQL数据库中，但现在它在SQL数据库中广泛存在。 

type_specification ::= { json | jsonb }

## **基本与复合数据类型**

JSON可以表示四种基本数据类型和两种复合数据类型的值。
基本数据类型有字符串、数字、布尔值和null。没有办法声明JSON值的数据类型；相反，它是从表示的语法中出现的。
将其与SQL和PL/pgSQL进行比较。SQL从表中的列或记录中的字段的元数据中建立一个值的数据类型，该值被写入或读取。它还具有类型转换表示法，如::text或::boolean，用于建立SQL文本的数据类型。PL/pgSQL还支持类型转换表示法，并通过声明建立变量或形式参数的数据类型。在JSON类型系统中，null被定义为一种数据类型，而不是另一个数据类型的“值”。 
注意，JSON不能表示日期-时间值，除非是传统格式的字符串值。

两种复合数据类型是对象和数组。

### **JSON字符串**

JSON字符串值是一个由零、一或多个Unicode字符组成的序列，这些字符由“”字符括起来。示例如下所示：

'"Dog"'::jsonb
空字符串是合法的，并且与JSON null不同。 
' "" '::jsonb

大小写和空格是有意义的，字符串值中的特殊字符需要转义，如下： 
退格：\b
换页符：\f
换行符：\n
回车：\r
Tab：\t
双引号：\“
反斜线 ：\\\\

例如：

```
 '"\"First line\"\n\"second line\""'::jsonb
```

### **JSON数字**

示例如下：

```
'17'::jsonb
 
'4.2'::jsonb
 
'2.99792E8'::jsonb
```

注意：JSON没有区分整数和实数。

### **JSON布尔型**

例如：

```
'true'::jsonb
 
'false'::jsonb
```

### **JSON null**

null在JSON中是特殊的，因为它是自己的数据类型，只允许一个“值”。
例如：

```
'null'::jsonb
```

### **JSON对象**

对象是一系列键值对，用逗号分隔，并用大括号括起来，顺序不重要，对象中的值不必具有彼此相同的数据类型。例如：

```
'{
  "a 1" : "Abc",
  "a 2" : 42,
  "a 3" : true,
  "a 4" : null,
  "a 5" : {"x" : 1, "y": "Pqr"}
}'::jsonb
```

键区分大小写，并且键中的空白非常重要。它们甚至可以包含必须转义的字符。但是，如果键确实包含空格和特殊字符，则读取其值所需的语法可能会变得相当复杂。因此，应该尽量避免这种情况。
一个对象可以包含多个具有相同键的键值对，但是，并不推荐这样做。

### **JSON数组**

数组是一个未命名JSON值的有序列表，换句话说，顺序是定义的，并且重要。数组中的值不必具有彼此相同的数据类型。例如： 

```
'[1, 2, "Abc", true, false, null, {"x": 17, "y": 42}]'::jsonb
```

注：数组中的值从0开始索引。

### **复合JSON**

例如：

```
{
  "given_name"         : "Fred",
  "family_name"        : "Smith",
  "email_address"      : "fred@example.com",
  "hire_date"          : "17-Jan-2015",
  "job"                : "sales",
  "base_annual_salary" : 50000,
  "commisission_rate"  : 0.05,
  "phones"             : ["+11234567890", "+13216540987"]
}
```

## **在JSON列上创建索引和检查约束**

通常，当JSON文档插入到表中时，该表将只有一个自填充的代理主键列和一个数据类型为jsonb的值列（如doc）。与选择json相比，选择jsonb允许使用更广泛的运算符和函数，并执行更加高效。
很可能每个文档都是一个JSON对象，并且所有文档都符合相同的结构定义。换句话说，每个对象都将具有相同的一组可能的键名（但有些可能缺失），并且每个键的值都具有相同的JSON数据类型。当一个数据类型是复合的时，同样的公共结构定义概念也将适用，将概念递归扩展到任意深度。下面是一个示例。为了减少混乱，主键没有定义为自填充。 

```
create table books(k int primary key, doc jsonb not null);
 
insert into books(k, doc) values
  (1,
  '{ "ISBN"    : 4582546494267,
     "title"   : "Macbeth",
     "author"  : {"given_name": "William", "family_name": "Shakespeare"},
     "year"    : 1623}'),
 
  (2,
  '{ "ISBN"    : 8760835734528,
     "title"   : "Hamlet",
     "author"  : {"given_name": "William", "family_name": "Shakespeare"},
     "year"    : 1603,
     "editors" : ["Lysa", "Elizabeth"] }'),
 
  (3,
  '{ "ISBN"    : 7658956876542,
     "title"   : "Oliver Twist",
     "author"  : {"given_name": "Charles", "family_name": "Dickens"},
     "year"    : 1838,
     "genre"   : "novel",
     "editors" : ["Mark", "Tony", "Britney"] }'),
  (4,
  '{ "ISBN"    : 9874563896457,
     "title"   : "Great Expectations",
     "author"  : {"family_name": "Dickens"},
     "year"    : 1950,
     "genre"   : "novel",
     "editors" : ["Robert", "John", "Melisa", "Elizabeth"] }'),
 
  (5,
  '{ "ISBN"    : 8647295405123,
     "title"   : "A Brief History of Time",
     "author"  : {"given_name": "Stephen", "family_name": "Hawking"},
     "year"    : 1988,
     "genre"   : "science",
     "editors" : ["Melisa", "Mark", "John", "Fred", "Jane"] }'),
 
  (6,
  '{
    "ISBN"     : 6563973589123,
    "year"     : 1989,
    "genre"    : "novel",
    "title"    : "Joy Luck Club",
    "author"   : {"given_name": "Amy", "family_name": "Tan"},
    "editors"  : ["Ruilin", "Aiping"]}');
```

仔细观察一下，不难发现，有些行缺少了某些键。但是“k=6”的行包含所有键。

### **创建检查约束**

以下创建约束：检查每个JSON文档都是一个对象 

```
alter table books
add constraint books_doc_is_object
check (jsonb_typeof(doc) = 'object');
```

以下创建约束：检查ISBN始终定义为一个13位的正数 

```
alter table books
add constraint books_isbn_is_positive_13_digit_number
check (
  (doc->'ISBN') is not null
    and
  jsonb_typeof(doc->'ISBN') = 'number'
     and
  (doc->>'ISBN')::bigint > 0
    and
  length(((doc->>'ISBN')::bigint)::text) = 13
);
```


请注意，如果键“ISBN”完全丢失，那么表达式doc->'ISBN'将生成真正的SQL NULL。但是文档的制作者可能已经决定，用键“ISBN”的特殊JSON值null来表示“没有关于这本书的ISBN的信息”。 

### **创建索引**

从上面的表结构来看，当“ISBN”键的值具有唯一、NOT NULL的特性，则可以创建如下索引，来强化唯一性：

```
create unique index books_isbn_unq
on books((doc->>'ISBN') hash);
```

您可能希望支持引用“year”键值的范围查询，如下所示： 

```
select
  (doc->>'ISBN')::bigint as year,
  doc->>'title'          as title,
  (doc->>'year')::int    as year
from books
where (doc->>'year')::int > 1850
order by 3;
```

则针对“year”键值，可以创建如下索引：

```
create index books_year on books ((doc->>'year') asc)
where doc->>'year' is not null;
```

## **JSON函数和操作符**

有两个简单的类型转换运算符，用于在符合RFC 7159的文本值和jsonb或json值之间进行转换，即通常重载的 = 运算符、12个专用json运算符和23个专用json函数。
大多数运算符都是重载的，因此它们可以同时用于json和jsonb值。
其中一些函数只有一个jsonb变量，还有一些函数只有json变量。函数名以jsonb_开头或以_jsonb结尾反映了这一点，相应地，json变体也是如此。之所以使用这种命名约定，而不是普通的重载，是因为当相同命名函数的形式参数规范不同时，BSQL可以区分它们，而当它们的返回类型不同时，则不能区分。某些用于特定目的的JSON函数的不同之处，仅在于返回json 值或jsonb值。
当一个运算符或函数，同时具有JSON值输入和JSON值输出时，jsonb变量接受jsonb输入并产生jsonb输出；相应地，json变量接受json输入并产生json输出。您可以使用bsqlsh \df元命令来查看JSON函数。
当一个运算符或函数，同时具有jsonb变量和json变量时，为减少混淆，则只描述jsonb变量，json变量的功能可以从jsonb功能的描述中被相似地予以理解。

### **转换SQL值到JSON值函数和操作符**

| 函数和操作符                                   | jsonb | json | 描述                                                         |
| ---------------------------------------------- | ----- | ---- | ------------------------------------------------------------ |
| [::jsonb](#_::jsonb和::json和::text)           | 是    | 是   | ::jsonb 将符合RFC 7159的SQL文本值类型转换为jsonb值。         |
| [to_jsonb()](#_to_jsonb())                     | 是    | 是   | 将允许JSON表示的任何基本或复合数据类型的单个SQL值，转换为语义等效的jsonb。 |
| [row_to_json()](#_row_to_json())               |       | 是   | 将SQL记录转换为JSON对象。                                    |
| [array_to_json()](#_array_to_json())           |       | 是   | 将SQL数组转换为JSON数组。                                    |
| [jsonb_build_array()](#_jsonb_build_array())   | 是    | 是   | 根据可变参数列表构建可能异构类型的JSON数组。                 |
| [jsonb_build_object()](#_jsonb_build_object()) | 是    | 是   | 根据可变参数列表构建一个JSON对象。                           |
| [jsonb_object()](#_jsonb_object())             | 是    | 是   | 从文本数组构建JSON对象。                                     |
| [jsonb_agg()](#_jsonb_agg())                   | 是    | 是   | 收集所有输入值，包括空值，到一个JSON数组。                   |
| [jsonb_object_agg()](#_jsonb_object_agg())     | 是    | 是   | 将所有键/值对收集到一个JSON对象中。关键参数强制转换为文本；值参数按照to_json或to_jsonb进行转换。 值可以为空，但键不能为空。 |

#### ::jsonb和 ::json和 ::text
目的：::jsonb 将符合RFC 7159的SQL文本值类型转换为jsonb值。

语法：

输入值:       jsonb
返回值:      text

注意：您可以对jsonb值和json值使用::text运算符；您可以对文本值和json值使用::jsonb运算符；您可以对jsonb值和文本值使用::json运算符。 

例如：

```
select '{"a":1, "b":2}'::jsonb;
```

返回信息如下：

```
      jsonb       
------------------
 {"a": 1, "b": 2}
```

#### to_jsonb()
目的：将允许JSON表示的任何基本或复合数据类型的单个SQL值，转换为语义等效的jsonb。

语法：

输入值:       anyelement
返回值:      jsonb

例如：

```
select to_jsonb(row(42, 'Fred said "Hi."'::text));
```

返回信息如下：

```
               to_jsonb                
--------------------------------------------
 {"f1": 42, "f2": "Fred said \"Hi.\""}
```

#### row_to_json()
目的：将SQL记录转换为JSON对象。

语法：

输入值:       record
pretty:            boolean (optional)
返回值:      json

例如：

```
select row_to_json(row(1,'foo')) ;
```

返回信息如下：

```
     row_to_json     
--------------------------
 {"f1":1,"f2":"foo"}
select row_to_json(row(1,'foo'),true),row_to_json(row(1,'foo'),false) ;
```

返回信息如下：

```
 row_to_json  |     row_to_json     
-------------------+---------------------
 {"f1":1,    + | {"f1":1,"f2":"foo"}
  "f2":"foo"}  | 
```

#### array_to_json()
目的：将SQL数组转换为JSON数组。

语法：

输入值:       anyarray
pretty:            boolean (optional)
返回值:      json

例如：

```
select array_to_json(array['a', 'b', 'c']) ;
```

返回信息如下：

```
 array_to_json 
---------------
 ["a","b","c"]
select array_to_json(array['a', 'b', 'c'],true),array_to_json(array['a', 'b', 'c'],false)  ;
```

返回信息如下：

```
 array_to_json | array_to_json 
---------------+---------------
 ["a",    + | ["a","b","c"]
  "b",    +| 
  "c"]     | 
```

#### jsonb_build_array()
目的：根据可变参数列表构建可能异构类型的JSON数组。

语法：

输入值:       VARIADIC "any"
返回值:      jsonb

例如：

```
select jsonb_build_array(1, 2, 'foo', 4, 5) ;
```

返回信息如下：

```
  jsonb_build_array  
---------------------
 [1, 2, "foo", 4, 5]
```

#### jsonb_build_object() 和 json_build_object()
目的：根据可变参数列表构建一个JSON对象。

语法：

输入值:       VARIADIC "any"
返回值:      jsonb

例如：

```
select jsonb_build_object('foo', 1, 2, row(3,'bar'));
```

返回信息如下：

```
           jsonb_build_object            
-----------------------------------------
 {"2": {"f1": 3, "f2": "bar"}, "foo": 1}
```

例如：

```
select jsonb_build_object('a', 17,'b', 'dog','c', true,'d', (17::int, 'cat'::text));
```

返回信息如下：

```
                       jsonb_build_object                       
----------------------------------------------------------------
 {"a": 17, "b": "dog", "c": true, "d": {"f1": 17, "f2": "cat"}}
```

#### jsonb_object() 和 json_object()
目的：从文本数组构建JSON对象。该数组必须有两个维度，一个维度的成员数为偶数，在这种情况下，它们被视为交替的键/值对; 另一个维度的成员数为二维，每个内部数组恰好有两个元素，它们被视为键/值对。所有值都转换为JSON字符串。
此函数有三个重载

##### *重载函数一*
语法：

输入值:      text[]
返回值:     jsonb

例如：

```
select jsonb_object(array['a', '17', 'b', $$'Hello', you$$, 'c', 'true']);
```

返回信息如下：

```
                 jsonb_object                  
-----------------------------------------------------
 {"a": "17", "b": "'Hello', you", "c": "true"}
```

##### *重载函数二*
语法：

输入值:       text[][]
返回值:      jsonb

例如：

```
select jsonb_object(array[array['a',  '17'], array['b',  $$'Hello', you$$],array['c',  'true']]);
```

返回信息如下：

```
                 jsonb_object                  
-----------------------------------------------
 {"a": "17", "b": "'Hello', you", "c": "true"}
```

##### *重载函数三*
语法：

输入值:       text[], text[] 
返回值:      jsonb

例如：

```
select jsonb_object(array['a','b','c'],array['17', $$'Hello', you$$, 'true']);
```

返回信息如下：

```
                 jsonb_object                  
-----------------------------------------------
 {"a": "17", "b": "'Hello', you", "c": "true"}
```


#### jsonb_agg()
目的：这是一个聚合函数，收集所有输入值，包括空值，到一个JSON数组。

语法：

输入值:       SETOF anyelement
返回值:      jsonb

例如：

```
 with tab as (
    values
      (5::int,    'ant'::text),
      (2::int,    'cat'::text),
      (null::int, 'ant'::text),
      (1::int,    'dog'::text),
      (4::int,     null::text))
  select
  json_agg((column1, column2)::rt order by column1 nulls first)
  from tab;
```

返回信息如下：

```
        json_agg         
-------------------------------
 [{"a":null,"b":"ant"},  +
  {"a":1,"b":"dog"},   +
  {"a":2,"b":"cat"},    +
  {"a":4,"b":null},     +
  {"a":5,"b":"ant"}]
```

 

#### jsonb_object_agg()
目的：这是一个聚合函数，将所有键/值对收集到一个JSON对象中。关键参数强制转换为文本；值参数按照to_json或to_jsonb进行转换。 值可以为空，但键不能为空。

语法：

输入值:       anyelement
返回值:      jsonb

例如：

```
  with tab as (
    values
      ('f4'::text, 4::int),
      ('f1'::text, 1::int),
      ('f3'::text, null::int),
      ('f2'::text, 2::int))
  select
    jsonb_object_agg(column1, column2)
  from tab;
```

返回信息如下：

```
            jsonb_object_agg             
-----------------------------------------
 {"f1": 1, "f2": 2, "f3": null, "f4": 4}
```

需要注意的一点是：当一个键值对重复多次，会以最近指定的为准，例如：

```
 with tab as (
    values
      ('f2'::text, 4::int),
      ('f7'::text, 7::int),
      ('f2'::text, 1::int),
      ('f2'::text, null::int))
  select
    jsonb_object_agg(column1, column2)
  from tab;
```

返回信息如下：

```
   jsonb_object_agg    
-------------------------
 {"f2": null, "f7": 7}
```

### **转换JSON值到JSON值函数和操作符**

| 函数和操作符                                   | jsonb | json | 描述                                                         |
| ---------------------------------------------- | ----- | ---- | ------------------------------------------------------------ |
| [->](#_->)                                     | 是    | 是   | 用给定的键提取JSON对象字段，或者提取JSON数组的第n个元素(数组元素从0开始索引，但负整数从末尾开始计数)。  返回json或者jsonb值 |
| [#>](#_#>)                                     | 是    | 是   | 提取指定路径下的JSON子对象，路径元素可以是字段键或数组索引。 |
| [\|\|](#_\|\|)                                 | 是    |      | 连接两个jsonb值。                                            |
| [-](#_-)                                       | 是    |      | 从一个对象中删除键值对，或者从数组中删除一个值。             |
| [#-](#_#-)                                     | 是    |      | 删除指定路径上的字段或数组元素，路径元素可以是字段键或数组索引。 |
| [jsonb_extract_path()](#_jsonb_extract_path()) | 是    | 是   | 在指定路径下提取JSON子对象。(这在功能上相当于#>操作符，但在某些情况下，将路径写成可变参数列表会更方便) |
| [jsonb_strip_nulls()](#_jsonb_strip_nulls())   | 是    | 是   | 从给定的JSON值中删除所有具有空值的对象字段，递归地。非对象字段的空值是未受影响的。 |
| [jsonb_set()](#_jsonb_set())                   | 是    |      | 更改JSON的值，即JSON对象中现有键值对的值或JSON数组中现有索引处的值。 |
| [jsonb_insert()](#_jsonb_insert())             | 是    |      | 插入一个值，该值可以是JSON对象中还不存在的键的值，也可以是在JSON数组的索引范围的末尾或开始之前。 |


#### ->
目的：用给定的键提取JSON对象字段，或者提取JSON数组的第n个元素(数组元素从0开始索引，但负整数从末尾开始计数)。返回json或者jsonb值
注意：->运算符要求JSON值是对象或数组。键是一个SQL值。当key是SQL文本值时，它从对象中读取具有该key的键值对的JSON值。当键是SQL整数值时，它会从数组中读取该索引键处的JSON值。如果输入的JSON值是JSON，那么输出的JSON值就是JSON，相应地，如果输入的JSON值是jsonb。 

语法：

输入值s:       jsonb -> [int | text] [ -> [int | text] ]*
返回值:       jsonb

例如：

```
select '{"a": 100, "b": {"x": 1, "y": 19}, "c": true}'::jsonb -> 'a' as a
, '{"a": 100, "b": {"x": 1, "y": 19}, "c": true}'::jsonb -> 'b' as b;
```

返回信息如下：

```
 100  |  {"x": 1, "y": 19}
```

#### #>
目的：提取指定路径下的JSON子对象，路径元素可以是字段键或数组索引。

语法：

输入值:        jsonb #> text[]
返回值:       jsonb

注：任意深度定位的JSON子值由其从最上面的JSON值开始的路径标识。通常，路径是由对象子值的键和数组子值的索引值的混合指定的。
请认真考虑如下示例：

```
[
  1,
  {
    "x": [
      1,
      true,
      {"a": "cat", "b": "dog"},
      3.14159
    ],
    "y": true
  },
  42
]
```

针对上例，分层分析如下：

* 在最高层，它是一个由三个子值组成的数组。
* 在第二层中，第二个数组子值（即索引为1的值）是一个具有两个键值对的对象，称为“x”和“y”。
* 在第三层中，键“x”的子值是一个子值数组。
* 在第四层中，第三个数组子值（即索引为2的值）是一个具有两个键值对的对象，称为“a”和“b”。
* 在第五层中，键“b”的子值是基本数据类型字符串值“dog”。 
  因此，在如上分析之后，可以得出，JSON字符串值“dog”的路径就应为：

-> 1 -> 'x' -> 2 -> 'b'

#>运算符是一种方便的语法简写，用于紧凑地指定长路径，因此： 

获取dog值的完整语法即为如下：

```
select '[
  1,
  {
    "x": [
      1,
      true,
      {"a": "cat", "b": "dog"},
      3.14159
    ],
    "y": true
  },
  42
]'::jsonb #> array['1', 'x', '2', 'b']::text[] as b;
```

返回信息如下：

```
 "dog"
```


#### ||
目的：连接两个jsonb值。连接两个数组将生成一个包含每个输入的所有元素的数组。连接两个对象将生成一个包含它们键的并集的对象，当存在重复的键时取第二个对象的值。 所有其他情况都是通过将非数组输入转换为单个元素数组，然后按照两个数组的方式进行处理。 不递归操作:只有顶级数组或对象结构被合并。

语法：

输入值s:       jsonb || jsonb
返回值:       jsonb

请参考如下各种情况示例：

1）如果运算符的两边都是基本数据类型JSON值，则结果是这些值的数组，例如：

```
select '17'::jsonb || '"x"'::jsonb;
```

返回信息如下：

```
 [17, "x"]
```

2）如果一侧是基本数据类型JSON值，另一侧是数组，则结果是数组，例如：

```
select '17'::jsonb || '["x", true]'::jsonb;
```

返回信息如下：

```
 [17, "x", true]
```

3）如果每一侧都是一个对象，并且RHS对象中没有任何键值对，与LHS对象中的任何键值对具有相同的键，则结果是一个存在所有键值对的对象，例如：

```
select '{"a": 1, "b": 2}'::jsonb || '{"p":17, "q": 19}'::jsonb;
```

返回信息如下：

```
 {"a": 1, "b": 2, "p": 17, "q": 19}
```

4）如果RHS对象中任何键值对的键，与LHS对象中键值对的键冲突，则RHS对象的键值对保留，就像当这些对的键在单个对象中冲突时一样： 

```
select '{"a": 1, "b": 2}'::jsonb || '{"p":17, "a": 19}'::jsonb;
```

返回信息如下：

```
 {"a": 19, "b": 2, "p": 17}
```

5）如果一侧是对象，另一侧是数组，则对象会成为数组中的值：

```
select '{"a": 1, "b": 2}'::jsonb || '[false, 42, null]'::jsonb;
```

返回信息如下：

```
 [{"a": 1, "b": 2}, false, 42, null]
```


#### -
目的：从JSON对象中删除键(以及它的值)，或从JSON数组中删除匹配的字符串值。

语法：

输入值s:       jsonb - [int | text]
返回值:       jsonb

1）从JSON对象中删除键(以及它的值)，例如：

```
select '{"a": "x", "b": "y"}'::jsonb - 'a'::text;
```

返回信息如下：

```
 {"b": "y"}
```

2）删除多个键值对，例如：

```
select '{"a": "p", "b": "q", "c": "r"}'::jsonb - array['a', 'c']::text[];
```

返回信息如下：

```
 {"b": "q"}
```

3）从数组中删除单个值，例如：

```
select '[1, 2, 3, 4]'::jsonb - 0;
```


返回信息如下：

```
 [2, 3, 4]
```

4）没有直接的方法从索引列表中的数组中删除几个值，类似于从具有对键列表的对象中删除几个键值对的能力。但可以通过如下方法，达到目的，例如：

```
select (('[1, 2, 3, 4, 5, 7]'::jsonb - 0) - 0 ) -0;
```

返回信息如下：

```
 [4, 5, 7]
```

#### #-
目的：删除指定路径上的字段或数组元素，路径元素可以是字段键或数组索引。

语法：

输入值s:       jsonb - text[]
返回值:       jsonb

例如：

```
select '["a", {"b":17, "c": ["dog", "cat"]}]'::jsonb #- array['1', 'c', '0']::text[];
```

返回信息如下：

```
 ["a", {"b": 17, "c": ["cat"]}]
```

#### jsonb_extract_path() 和 json_extract_path()
目的：在指定路径下提取JSON子对象。(这在功能上相当于#>操作符，但在某些情况下，将路径写成可变参数列表会更方便。)

语法：

输入值:       jsonb, VARIADIC text
返回值:      jsonb

例如：

```
select jsonb_extract_path('[1, {"x": [1, true, {"a": "cat", "b": "dog"}, 3.14159], "y": true}, 42]'::jsonb , '1', 'x', '2', 'b');
```

返回信息如下：

```
 "dog"
```

#### jsonb_strip_nulls 和 json_strip_nulls
目的：从给定的JSON值中删除所有具有空值的对象字段，递归地。非对象字段的空值是未受影响的。

语法：

输入值:       jsonb
返回值:      jsonb

例如：

```
select jsonb_strip_nulls('{"a": 17,"b": null,"c": {"x": null, "y": "dog"},"d": [42, null, "cat"]}');
```

返回信息如下：

```
 {"a": 17, "c": {"y": "dog"}, "d": [42, null, "cat"]}
```

#### jsonb_set()
目的：更改JSON的值，即JSON对象中现有键值对的值或JSON数组中现有索引处的值。返回target，将path指定的项替换为new_value， 如果create_if_missing为真(此为默认值)，并且path指定的项不存在，则添加new_value。 路径中的所有前面步骤都必须存在，否则将不加改变地返回target。 与面向路径操作符一样，负整数出现在JSON数组末尾的path计数中。 如果最后一个路径步骤是超出范围的数组索引，并且create_if_missing为真，那么如果索引为负，新值将添加到数组的开头，如果索引为正，则添加到数组的结尾。

语法：

jsonb_in:           jsonb
path:              text[]
replacement:        jsonb
create_if_missing:   boolean default true
返回值:        jsonb

例如：

```
select jsonb_set('{"a": 1, "b": 2, "c": 3}',array['d'],'4',true);
```

或者

```
select jsonb_set(jsonb_in => '{"a": 1, "b": 2, "c": 3}',path => array['d'],replacement => '4',create_if_missing => true);
```

均返回信息如下：

```
 {"a": 1, "b": 2, "c": 3, "d": 4}
```

#### jsonb_insert()
目的：插入一个值，该值可以是JSON对象中还不存在的键的值，也可以是在JSON数组的索引范围的末尾或开始之前。

语法：

jsonb_in:         jsonb
path:            text[]
replacement:      jsonb
insert_after:       boolean default false
返回值:      jsonb

例如：

```
select jsonb_insert('{"a": 1, "b": 2, "c": 3}',array['d'],'4',true);
```

或者

```
select jsonb_insert(jsonb_in => '{"a": 1, "b": 2, "c": 3}',path => array['d'],replacement => '4',insert_after => true);
```

均返回信息如下：

```
 {"a": 1, "b": 2, "c": 3, "d": 4}
```

### **转换JSON值到SQL值函数和操作符**

| 函数和操作符                                                 | jsonb | json | 描述                                                   |
| ------------------------------------------------------------ | ----- | ---- | ------------------------------------------------------ |
| [::text](#_::text)                                           | 是    | 是   | 将jsonb值类型转换为符合RFC 7159的SQL文本值。           |
| [->>](#_->>)                                                 | 是    | 是   | 将指定的JSON值读取为文本值。                           |
| [#>>](#_#>>)                                                 | 是    | 是   | 将指定路径上的JSON子对象提取为text。                   |
| [jsonb_extract_path_text()](#_jsonb_extract_path_text())     | 是    | 是   | 将指定路径上的JSON子对象提取为文本。                   |
| [jsonb_populate_record()](#_jsonb_populate_record())         | 是    | 是   | 将JSON对象转换为等效的SQL记录。                        |
| [jsonb_populate_recordset()](#_jsonb_populate_recordset())   | 是    | 是   | 将JSON对象的同类JSON数组转换为等效的SQL记录集          |
| [jsonb_to_record()](#_jsonb_to_record())                     | 是    | 是   | 将JSON对象转换为等效的SQL记录。                        |
| [jsonb_to_recordset()](#_jsonb_to_recordset())               | 是    | 是   | 将JSON对象的同类JSON数组转换为等效的SQL记录集          |
| [jsonb_array_elements()](#_jsonb_array_elements())           | 是    | 是   | 将JSON数组的JSON值转换为jsonb 值的SQL表。              |
| [jsonb_array_elements_text()](#_jsonb_array_elements_text()) | 是    | 是   | 将JSON数组的JSON值转换为一个包含文本值的SQL表          |
| [jsonb_each()](#_jsonb_each())                               | 是    | 是   | 从JSON对象中创建一个包含列“key”和“value”的行集。       |
| [jsonb_each_text()](#_jsonb_each_text())                     | 是    | 是   | 从JSON对象中创建一个包含列“key”和“value”的文本行集     |
| [jsonb_pretty()](#_jsonb_pretty())                           | 是    |      | 将给定的JSON值转换为精美打印的，缩进的，便于阅读的文本 |

#### ::text
    目的：将jsonb值类型转换为符合RFC 7159的SQL文本值。

语法：

输入值:       jsonb
返回值:      text

例如：

```
select '{"a": 42, "b": 17, "a": 99}'::jsonb::text;
```

返回信息如下：

```
 {"a": 42, "b": 17, "a": 99}
```

#### ->>
目的：将指定的JSON值读取为文本值。提取JSON数组的第n个元素，作为text；用给定的键提取JSON对象字段，作为text。

语法：
输入值s:       jsonb ->> [int | text] [ -> [int | text] ]*
返回值:       text

例如：

```
select '[1,2,3]'::json ->> 2; 
```

返回信息如下：

```
3
select '{"a":1,"b":2}'::json ->> 'b';
```

返回信息如下：

```
2
```

#### #>>
目的：将指定路径上的JSON子对象提取为text。

语法：

输入值:        jsonb #>>  text[]
返回值:       text

例如：

```
select '{"a": {"b": ["foo","bar"]}}'::json #>> '{a,b,1}';
```

返回信息如下：

```
bar
```


#### jsonb_extract_path_text() 和 json_extract_path_text()
目的：将指定路径上的JSON子对象提取为文本。(这在功能上等同于#>>操作符。)

语法：

输入值:       jsonb, VARIADIC text
返回值:      text

例如：

```
select jsonb_extract_path_text('{"f2":{"f3":100},"f4":{"f5":99,"f6":"foo"}}', 'f4', 'f6');
```

返回信息如下：

```
 foo
```

#### jsonb_populate_record() 和 json_populate_record()
目的：将JSON对象转换为等效的SQL记录。JSON对象将被扫描，查找名称与输出行类型的列名匹配的字段，并将它们的值插入到输出的这些列中。 (不对应任何输出列名的字段将被忽略。)在典型的使用中，基本的值仅为NULL，这意味着任何不匹配任何对象字段的输出列都将被填充为空。 但是，如果base不为NULL，那么它包含的值将用于不匹配的列。

要将JSON值转换为输出列的SQL类型，需要按次序应用以下规则:
在所有情况下，JSON空值都会转换为SQL空值。

如果输出列的类型是json或jsonb，则会精确地重制JSON值。
如果输出列是复合(行)类型，且JSON值是JSON对象，则该对象的字段将转换为输出行类型的列，通过这些规则的递归应用程序。

同样，如果输出列是数组类型，而JSON值是JSON数组，则通过这些规则的递归应用程序将JSON数组的元素转换为输出数组的元素。

否则，如果JSON值是字符串，则将字符串的内容提供给输入转换函数，用以确定列的数据类型。
否则，JSON值的普通文本表示将被提供给输入转换函数，以确定列的数据类型。

虽然下面的示例使用一个常量JSON值，典型的用法是在查询的FROM子句中从另一个表侧面地引用json或jsonb列。 在FROM子句中编写json_populate_record是一种很好的实践，因为提取的所有列都可以使用，而不需要重复的函数调用。

语法：

输入值:       anyelement, jsonb
返回值:      anyelement

例如：

```
create type subrowtype as (d int, e text); 
 
create type myrowtype as (a int, b text[], c subrowtype);
 
select * 
from json_populate_record(null::myrowtype, '{"a": 1, "b": ["2", "a b"], "c": {"d": 4, "e": "a b c"}, "x": "foo"}') ;
```

返回信息如下：

```
 a |   b    |      c      
---+-----------+-------------
 1 | {2,"a b"} | (4,"a b c")
```

#### jsonb_populate_recordset() 和 json_populate_recordset()
目的：将JSON对象的同类JSON数组转换为等效的SQL记录集。
每个都要求提供的JSON值是一个数组，每个数组的值都是一个与指定的SQL记录兼容的对象，该记录被定义为一个类型，该类型的名称通过函数的第一个形式参数使用代理NULL传递：type_identifier。JSON值通过第二个形式参数传递。结果是指定类型的一组记录（即一个表）。

语法：

输入值:       anyelement, jsonb
返回值:      SETOF anyelement

例如：

```
create type twoints as (a int, b int);
 
select * 
from json_populate_recordset(null::twoints, '[{"a": 1, "b": 2},{"b": 4, "a": 3},{"a": 5, "c": 6},{"b": 7, "d": 8},{"c": 9, "d": 0}]');
```

返回信息如下：

```
 a | b 
---+---
 1 | 2
 3 | 4
 5 |  
   | 7
   |  
```

#### jsonb_to_record() 和 json_to_record()
目的：将JSON对象转换为等效的SQL记录。

语法：

输入值:       jsonb
返回值:      record

例如：

```
create type myrowtype as (a int, b text);
 
select * 
 from json_to_record('{"a":1,"b":[1,2,3],"c":[1,2,3],"e":"bar","r": {"a": 123, "b": "a b c"}}') as x(a int, b text, c int[], d text, r myrowtype);
```

返回信息如下：

```
 a |  b    |   c   | d |     r       
---+---------+---------+---+---------------
 1 | [1,2,3] | {1,2,3} |   | (123,"a b c")
```

#### jsonb_to_recordset() 和 json_to_recordset()
目的：将JSON对象的同类JSON数组转换为等效的SQL记录集。

语法：

输入值:       jsonb
返回值:      SETOF record


例如：

```
select * 
from jsonb_to_recordset('[{"a":1,"b":"foo"}, {"a":"2","c":"bar"}]') as x(a int, b text);
```

返回信息如下：

```
 a |  b  
---+-----
 1 | foo
 2 | 
```

#### jsonb_array_elements() 和 json_array_elements()
目的：将JSON数组的JSON值转换为jsonb 值的SQL表。

语法：

输入值:       jsonb
返回值:      SETOF jsonb

例如：

```
select * 
from jsonb_array_elements('["cat", "dog house", 42, true, {"x": 17}, null]');
```

返回信息如下：

```
 "cat"
 "dog house"
 42
 true
 {"x": 17}
 null
```

#### jsonb_array_elements_text() 和 json_array_elements_text()
目的：将JSON数组的JSON值转换为一个包含文本值的SQL表。

语法：

输入值:       jsonb
返回值:      SETOF text

例如：

```
select * 
from jsonb_array_elements_text('["cat", "dog house", 42, true, {"x": 17}]');
```

返回信息如下：

```
 "cat"
 "dog house"
 42
 true
 {"x": 17}
```

#### jsonb_each() 和 json_each()
目的：从JSON对象中创建一个包含列“key”和“value”的行集。

语法：

输入值:       jsonb
返回值:      SETOF (text, jsonb)

例如：

```
select key, value 
from jsonb_each('{"a": 17, "b": "dog", "c": true, "d": {"a": 17, "b": "cat"}}'::jsonb);
```

返回信息如下：

```
 key |         value         
-----+-----------------------
 a   | 17
 b   | dog
 c   | true
 d   | {"a": 17, "b": "cat"}
```

#### jsonb_each_text() 和 json_each_text()
目的：从JSON对象中创建一个包含列“key”和“value”的文本行集。

语法：

输入值:       jsonb
返回值:      SETOF (text, text)

例如：

```
select key, value 
from jsonb_each_text('{"a": 17, "b": "dog", "c": true, "d": {"a": 17, "b": "cat"}}'::jsonb);
```

返回信息如下：

```
 key |         value         
-----+-----------------------
 a   | 17
 b   | dog
 c   | true
 d   | {"a": 17, "b": "cat"}
```

#### jsonb_pretty()
目的：将给定的JSON值转换为精美打印的，缩进的，便于阅读的文本。

语法：

输入值:       jsonb
返回值:      text

例如：

```
select jsonb_pretty('[{"f1":1,"f2":null}, 2]');
```

返回信息如下：

```
 [                 +
     {             +
         "f1": 1,    +
         "f2": null  +
     },            +
     2            +
 ]
```

### **获取JSON值属性函数和操作符**

| 函数和操作符                                   | jsonb | json | 描述                                                         |
| ---------------------------------------------- | ----- | ---- | ------------------------------------------------------------ |
| [=](#_=)                                       | 是    |      | 测试两个jsonb值是否相等。                                    |
| [@> 和<@](#_@> and <@)                         | 是    |      | @>运算符测试左侧的JSON值是否包含右侧的JSON值。<@运算符测试右侧的JSON值是否包含左侧的JSON值。 |
| [?](#_?)                                       | 是    |      | 文本字符串是否作为JSON值中的顶级键或数组元素存在             |
| [?                                             | ](#_? | )    | 是                                                           |
| [?&](#_?&)                                     | 是    |      | 文本数组中的所有字符串都作为顶级键或数组元素存在?            |
| [jsonb_array_length()](#_jsonb_array_length()) | 是    |      | 返回顶级JSON数组中的元素数量。                               |
| [jsonb_typeof()](#_jsonb_typeof())             | 是    |      | 以文本字符串形式返回顶级JSON值的类型。                       |
| [jsonb_object_keys()](#_jsonb_object_keys())   | 是    |      | 返回顶级JSON对象中的键集合。                                 |

#### =
目的：测试两个jsonb值是否相等。

注：如果您需要测试两个json值是否相等，那么您必须使用::text 进行类型转换。

语法：

输入值s:       jsonb = jsonb
返回值:       boolean

例如：

```
select '["a","b","c"]'::jsonb='["a","b","c"]'::jsonb;
```

返回信息如下：

```
t
```

#### @> 和 <@
目的：@>运算符测试左侧的JSON值是否包含右侧的JSON值。<@运算符测试右侧的JSON值是否包含左侧的JSON值。 

语法：
输入值s:       jsonb @> jsonb
返回值:       boolean

或者：

输入值s:       jsonb <@ jsonb
返回值:       boolean

例如：

```
select '{"a": 1, "b": 2}'::jsonb @> '{"b" :2}'::jsonb;
```

返回信息如下：

```
t
```

#### ?
目的：文本字符串是否作为JSON值中的顶级键或数组元素存在?

语法：

输入值s:       jsonb ? text
返回值:       boolean

如果是顶级键，示例：

```
select '{"a": "x", "b": "y"}'::jsonb ? 'a';
```

返回信息如下：

```
t
```

如果是非顶级键，示例如下：

```
select '[1, {"a": "x", "b": "y"}]'::jsonb ? 'a';
```

返回信息如下：

```
f
```

数组元素示例：

```
select '["cat", "dog", "from"]'::jsonb ? 'dog';
```

返回信息如下：

```
t
```


#### ?|
目的：文本数组中的至少一个字符串，是否作为顶级键或数组元素存在?

语法：

输入值s:       jsonb ?| text[]
返回值:       boolean

至少一个键存在，例如：

```
select '{"a": "x", "b": "y", "c": "z"}'::jsonb ?| array['a', 'p']::text[];
```

返回信息如下：

```
t
```

至少一个数组元素存在，例如：

```
select '["a", "b", "c"]'::jsonb ?| array['a', 'p']::text[];
```

返回信息如下：

```
t
```


#### ?&
目的：文本数组中的所有字符串都作为顶级键或数组元素存在?

语法：
输入值s:       jsonb ?& text[]
返回值:       boolean

例如：

```
select '{"a": "w", "b": "x", "c": "y", "d": "z"}' ::jsonb ?& array['a', 'b', 'c']::text[];
```

返回信息如下：

```
t
```

 

所有数组元素存在，例如：

```
select '["a", "b", "c"]'::jsonb ?& array['a', 'b']::text[];
```

返回信息如下：

```
t
```

#### jsonb_array_length() 和 json_array_length()
目的：返回顶级JSON数组中的元素数量。
注意：要求提供的JSON值是一个数组
语法：

输入值:       jsonb
返回值:      integer

例如：

```
select json_array_length('["a", 42, true, null]') ;
```

返回信息如下：

```
4
```

#### jsonb_typeof() 和 json_typeof()
目的：以文本字符串形式返回顶级JSON值的类型。可能的类型有object, array,string, number,boolean, 和 null。

语法：

输入值:       jsonb
返回值:      text

例如：

```
select jsonb_typeof('{"a": 17, "b": "x", "c": true}'::jsonb);
```

返回信息如下：

```
object
```

#### jsonb_object_keys() 和 json_object_keys()
目的：返回顶级JSON对象中的键集合。

语法：

输入值:       jsonb
返回值:      SETOF text

例如：

```
select jsonb_object_keys( '{"b": 1, "c": true, "a": {"p":1, "q": 2}}'::jsonb);
```

返回信息如下：

```
 a
 b
 c
```
