基于字符的数据类型用于指定Unicode字符字符串的数据。
可用的一般用途的字符类型如下表格所示：

| 类型                             | 描述                                |
| -------------------------------- | ----------------------------------- |
| char                             | 1字节的字符                         |
| character(n), char(n)            | 固定长度（n）的字符串，空格填充     |
| character varying(n), varchar(n) | 具有最大限制的可变长度（n）的字符串 |
| varchar                          | 无限长度的变长的字符串              |
| text                             | 无限长度的变长的字符串              |

描述：
text_literal ::= " ' " [ '' | letter ...] " ' "

这里：

* 单引号必须转义为（''）。
* letter是除单引号（[^']）以外的任何字符。
* 基于字符的数据类型可以是PRIMARY KEY的一部分。
* 字符数据类型的值是可转换的，并且可与非文本数据类型进行比较。

SQL定义了两种基本的字符类型： character varying(n)和character(n)， 其中n是一个正整数。两种类型都可以存储最多n个字符长的串。试图存储更长的串到这些类型的列里会产生一个错误， 除非超出长度的字符都是空白，这种情况下该串将被截断为最大长度。 如果要存储的串比声明的长度短，类型为character的值将会用空白填满；而类型为character varying的值将只是存储短些的串。
如果我们明确地把一个值类型成character varying(n)或者character(n)， 那么超长的值将被截断成n个字符，而不会抛出错误（这也是SQL标准的要求）。
varchar(n)和char(n)的概念分别是character varying(n)和character(n)的别名。没有长度声明词的character等效于character(1)。
另外，提供text类型，它可以存储任何长度的串。

例如：

```
CREATE TABLE test2 (b varchar(5));
INSERT INTO test2 VALUES ('ok');
INSERT INTO test2 VALUES ('good      ');
INSERT INTO test2 VALUES ('too long');
ERROR:  value too long for type character varying(5)
INSERT INTO test2 VALUES ('too long'::varchar(5)); -- explicit truncation
select b, char_length(b) FROM test2;
```

返回如下信息：

```
   b   | char_length 
-------+-------------
 good  |         5
 ok    |         2
 too l   |        5
```

## **函数和操作符**

### **操作符和函数**

| 操作符和函数                                                 | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [string \|\| string](#_string\|\|string)                     | 字符串连接                                                   |
| [string \|\| non-string or non-string \|\| string](#_string\|\|non-string or non-string |                                                              |
| [bit_length()](#_bit_length(string))                         | 字符串中的位数                                               |
| [char_length() 与character_length()](#_char_length(string) 与 character_length(string)) | 字符串中的字符数                                             |
| [lower()](#_lower(string))                                   | 将字符串转换为小写                                           |
| [octet_length()](#_octet_length(string))                     | 字符串中的字节数                                             |
| [overlay(string placing string from int [for int])](#_overlay(string placing string from int [for int])) | 替换子字符串                                                 |
| [position(substring in string)](#_position(substring in string)) | 指定子字符串的位置                                           |
| [substring(string [from int] [for int])](#_substring(string [from int] [for int])) | 提取子字符串                                                 |
| [substring(string from pattern)](#_substring(string from pattern)) | 提取与POSIX正则表达式匹配的子字符串。                        |
| [substring(string from pattern for escape)](#_substring(string from pattern for escape)) | 提取与SQL正则表达式匹配的子字符串。                          |
| [trim([leading                                               | trailing                                                     |
| [trim([leading                                               | trailing                                                     |
| [upper()](#_upper(string))                                   | 将字符串转换为大写                                           |
| [ascii()](#_ascii(string))                                   | 参数的第一个字符的ASCII代码。                                |
| [btrim(string text [, characters text])](#_btrim(string text [, characters text])) | 从string的开头或结尾删除最长的只包含characters（默认是一个空格）的字符串 |
| [chr()](#_chr(int))                                          | 返回给定代码的字符。                                         |
| [concat ( val1 "any" [, val2 "any" [, ...] ] )](#_concat ( val1 ) | 连接所有参数的文本表示。空参数被忽略。                       |
| [concat_ws ( sep text, val1 "any" [, val2 "any" [, ...] ] )](#_concat_ws ( sep text, val1 ) | 用分隔符连接除第一个参数外的所有参数。第一个参数用作分隔符字符串，不应为NULL。其他NULL参数将被忽略。 |
| [convert_to(string text, dest_encoding name)](#_convert_to(string text, dest_encoding name)) | 将字符串转换为dest_encoding。                                |
| [format(formatstr text [, formatarg "any" [, ...] ])](#_format(formatstr text [, formatarg ) | 根据格式字符串对参数进行格式化。                             |
| [initcap ()](#_initcap ( text ))                             | 将每个单词的第一个字母转换为大写，其余字母转换为小写。单词是由非字母数字字符分隔的字母数字字符序列。 |
| [left ()](#_left ( string text, n integer ))                 | 以字符串返回第一个 n 字符，或在 n 为负时, 返回最后           |
| [length ()](#_length ( text ))                               | 返回字符串中的字符数。                                       |
| [lpad ()](#_lpad ( string text, length integer [, fill text ] )) | 将string扩展为长度length，通过前置字符fill（默认空格）。 如果string已经超过length那么它将被截断（在右侧）。 |
| [ltrim ()](#_ltrim ( string text [, characters text ] ))     | 从string开始删除包含characters（默认空格）中仅包含字符的最长字符串。 |
| [md5 ()](#_md5 ( text ))                                     | 计算参数的 MD5 hash ，结果以十六进制形式写入。               |
| [parse_ident ( qualified_identifier text [, strict_mode boolean DEFAULT true ] )](#_parse_ident ( qualified_identifier text [, strict_mode boolean DEFAULT true ] )) | 将qualified_identifier拆分为一个标识符数组，删除单个标识符的任何引用。 |
| [pg_client_encoding ( )](#_pg_client_encoding ( ))           | 返回当前客户端编码名称。                                     |
| [quote_ident ()](#_quote_ident ( text ))                     | 返回适合引用的给定字符串，作为SQL语句字符串中的标识符。      |
| [quote_literal ()](#_quote_literal ( text ))                 | 返回在SQL语句字符串中适当引用的给定字符串，用作字符串文字使用。 |
| [quote_literal ()](#_quote_literal ( anyelement ))           | 将给定的值转换为文本，然后将其作为字面量引用。 内嵌的单引号和反斜杠被适当地翻倍。 |
| [quote_nullable ()](#_quote_nullable ( text ))               | 返回在SQL语句字符串中适当引用的给定字符串文字                |
| [quote_nullable ()](#_quote_nullable ( anyelement ))         | 将给定值转换为文本，然后将其作为字面量引用                   |
| [regexp_match ()](#_regexp_match ( string text, pattern text [, flags text ] )) | 返回从POSIX正则表达式到string的第一个匹配中捕获的子字符串    |
| [regexp_matches ()](#_regexp_matches ( string text, pattern text [, flags text ] )) | 返回通过将POSIX正则表达式与string匹配而捕获的子字符串        |
| [regexp_replace ()](#_regexp_replace ( string text, pattern text, replacement text [, flags text ] )) | 替换匹配POSIX正则表达式的子字符串                            |
| [regexp_split_to_array ()](#_regexp_split_to_array ( string text, pattern text [, flags text ] )) | 使用POSIX正则表达式作为分隔符拆分string                      |
| [regexp_split_to_table ()](#_regexp_split_to_table ( string text, pattern text [, flags text ] )) | 使用POSIX正则表达式作为分隔符拆分string                      |
| [repeat ()](#_repeat ( string text, number integer ))        | 重复string指定number的次数。                                 |
| [replace ()](#_replace ( string text, from text, to text ))  | 将string 中当前的子串替换                                    |
| [reverse ()](#_reverse ( text ))                             | 颠倒字符串中字符的顺序。                                     |
| [right ()](#_right ( string text, n integer ) ))             | 返回字符串中的最后n个字符，或者在n>为负时，返回除了前面的    |
| [rpad ()](#_rpad ( string text, length integer [, fill text ] ) )) | 扩展 string 到长度 length，通过追加fill 字符(默认为空格). 如果string 已经比 length 长，则截断它。 |
| [rtrim ()](#_rtrim ( string text [, characters text ] ))     | 从string末尾删除包含characters（默认为空格）中仅包含字符的最长字符串。 |
| [split_part ()](#_split_part ( string text, delimiter text, n integer )) | 在delimiter出现时拆分string，并且返回第n个字段(从一计数)。   |
| [strpos ()](#_strpos ( string text, substring text ))        | 返回在string中指定的substring的起始索引,如果不存在则为零。   |
| [substr ()](#_substr ( string text, start integer [, count integer ] )) | 提取string从start字符开始的子字符串，并扩展count字符，如果指定了的话。 |
| [starts_with ()](#_starts_with ( string text, prefix text )) | 如果 string 以 prefix开始就返回真。                          |
| [to_ascii ()](#_to_ascii ())                                 | 将string从另一个编码中转换为ASCII，该编码可按名称或编号标识。 |
| [to_hex ()](#_to_hex ())                                     | 将数字转换为其相应的十六进制表示形式。                       |
| [translate ()](#_translate ())                               | 将string中与from集合中匹配的每个字符替换为to集合中相应的字符。 |

#### string || string
目的：字符串连接

例如：

```
select 'Hello' || ' World';
```

返回信息如下：

```
Hello World
```

#### string || non-string or non-string || string
目的：字符串与非字符串连接

例如：

```
select 'Value: ' || 42;
```

返回信息如下：

```
 Value: 42
```

#### bit_length()
目的：字符串中的位数

语法：

输入值:      text
返回值:      integer

例如：

```
select bit_length('jose');
```

返回信息如下：

```
32
```

#### char_length() 与 character_length()
目的：字符串中的字符数

语法：

输入值:      text
返回值:      integer

例如：

```
select char_length('jose');
```

返回信息如下：

```
4
```

#### lower()
目的：将字符串转换为小写

语法：

输入值:      text
返回值:      text

例如：

```
select lower('TOM');
```

返回信息如下：

```
tom
```

#### octet_length()
目的：字符串中的字节数

语法：

输入值:      text
返回值:      integer

或者

输入值:      character
返回值:      integer

例如：

```
select octet_length('jose');
```

返回信息如下：

```
4
```


#### overlay(string placing string from int [for int])
目的：替换子字符串

例如：

```
select overlay('Txxxxas' placing 'hom' from 2 for 4);
```

返回信息如下：

```
Thomas
```

#### position(substring in string)
目的：指定子字符串的位置 

例如：

```
select position('om' in 'Thomas');
```

返回信息如下：

```
3
```

#### substring(string [from int] [for int])
目的：提取子字符串

例如：

```
select substring('Thomas' from 2 for 3);
```

返回信息如下：

```
hom
```

#### substring(string from pattern)
目的：提取与POSIX正则表达式匹配的子字符串。

例如：

```
select substring('Thomas' from '...$');
```

返回信息如下：

```
mas
```

#### substring(string from pattern for escape)
目的：提取与SQL正则表达式匹配的子字符串。

例如：

```
select substring('Thomas' from '%#"o_a#"_' for '#');
```

返回信息如下：

```
oma
```

#### trim([leading | trailing | both] [characters] from string)
目的：从字符串的开始、结束或两端（两者都是默认值）的字符（默认情况下为空格）中，删除仅包含字符的最长字符串

例如：

```
select trim(both 'xyz' from 'yxTomxx');
```

返回信息如下：

```
Tom
```

#### trim([leading | trailing | both] [from] string [, characters] )
目的：trim()的非标准语法

例如：

```
select trim(both from 'yxTomxx', 'xyz');
```

返回信息如下：

```
Tom
```

#### upper()
目的：将字符串转换为大写

语法：

输入值:      text
返回值:      text

例如：

```
select upper('tom');
```

返回信息如下：

```
TOM
```

#### ascii()
目的：参数的第一个字符的ASCII代码。对于UTF8，返回字符的Unicode。对于其他多字节编码，参数必须是ASCII字符。 

语法：

输入值:      text
返回值:      integer

例如：

```
select ascii('x');
```

返回信息如下：

```
120
```

#### btrim(string text [, characters text])
目的：从string的开头或结尾删除最长的只包含characters（默认是一个空格）的字符串 

语法：

输入值:      text, text
返回值:      text

例如：

```
select btrim('xyxtrimyyx', 'xyz');
```

返回信息如下：

```
trim
```

#### chr()
目的：返回给定代码的字符。在UTF8编码中该参数被视作一个Unicode代码点。 在其他多字节编码中该参数必须指定一个ASCII字符。 chr(0) 字符不被允许，因为文本数据类型不能存储这种字符。

语法：

输入值:      text
返回值:      text

例如：

```
select chr(65);
```

返回信息如下：

```
A
```

#### concat ( val1 "any" [, val2 "any" [, ...] ] )
目的：连接所有参数的文本表示。空参数被忽略。

语法：

输入值:      VARIADIC "any" 
返回值:      text

例如：

```
select concat('abcde', 2, NULL, 22);
```

返回信息如下：

```
abcde222
```

#### concat_ws ( sep text, val1 "any" [, val2 "any" [, ...] ] )
目的：用分隔符连接除第一个参数外的所有参数。第一个参数用作分隔符字符串，不应为NULL。其他NULL参数将被忽略。

语法：

输入值:      text, VARIADIC "any" 
返回值:      text

例如：

```
select concat_ws(',', 'abcde', 2, NULL, 22);
```

返回信息如下：

```
abcde,2,22
```

#### convert_to(string text, dest_encoding name)
目的：将字符串转换为dest_encoding。

语法：

输入值:      text, name 
返回值:      bytea

例如：

```
select convert_to('some text', 'UTF8');
```

返回信息如下：

```
 \x736f6d652074657874
```

#### format(formatstr text [, formatarg "any" [, ...] ])
目的：根据格式字符串对参数进行格式化。

语法：

输入值:      text, VARIADIC "any"
返回值:      text

formatstr是一个格式字符串，它指定了结果应该如何被格式化。格式字符串中的文本被直接复制到结果中，除了使用格式说明符的地方。格式说明符在字符串中扮演着占位符的角色，它定义后续的函数参数如何被格式化及插入到结果中。每一个formatarg参数会被根据其数据类型的常规输出规则转换为文本，并接着根据格式说明符被格式化和插入到结果字符串中。

格式说明符由一个%字符开始并且有这样的形式

%\[position][flags][width]type
其中的各组件域是：

position（可选）
一个形式为n\$的字符串，其中n是要打印的参数的索引。索引 1 表示formatstr之后的第一个参数。如果position被忽略，默认会使用序列中的下一个参数。

flags（可选）
控制格式说明符的输出如何被格式化的附加选项。当前唯一支持的标志是一个负号（-），它将导致格式说明符的输出会被左对齐（left-justified）。除非width域也被指定，否者这个域不会产生任何效果。

width（可选）
指定用于显示格式说明符输出的最小字符数。输出将被在左部或右部（取决于-标志）用空格填充以保证充满该宽度。太小的宽度设置不会导致输出被截断，但是会被简单地忽略。宽度可以使用下列形式之一指定：一个正整数；一个星号（\*）表示使用下一个函数参数作为宽度；或者一个形式为*n\$的字符串表示使用第n个函数参数作为宽度。

如果宽度来自于一个函数参数，则参数在被格式说明符的值使用之前就被消耗掉了。如果宽度参数是负值，结果会在长度为abs(width)的域中被左对齐（如果-标志被指定）。

type（必需）
格式转换的类型，用于产生格式说明符的输出。支持下面的类型：

s将参数值格式化为一个简单字符串。一个控制被视为一个空字符串。

I将参数值视作 SQL 标识符，并在必要时用双写引号包围它。如果参数为空，将会是一个错误（等效于quote_ident）。

L将参数值引用为 SQL 文字。一个空值将被显示为不带引号的字符串NULL（等效于quote_nullable）。

除了以上所述的格式说明符之外，要输出一个文字形式的%字符，可以使用特殊序列%%。

下面有一些基本的格式转换的示例：

```
select format('Hello %s', 'World');
```

结果：Hello World

```
select format('Testing %s, %s, %s, %%', 'one', 'two', 'three');
```

结果：Testing one, two, three, %

```
select format('INSERT INTO %I VALUES(%L)', 'Foo bar', E'O\'Reilly');
```

结果：INSERT INTO "Foo bar" VALUES('O''Reilly')

```
select format('INSERT INTO %I VALUES(%L)', 'locations', 'C:\Program Files');
```

结果：INSERT INTO locations VALUES(E'C:\\\\Program Files')

下面是使用width域和-标志的示例：

```
select format('|%10s|', 'foo');
```

结果：|       foo|

```
select format('|%-10s|', 'foo');
```

结果：|foo       |

```
select format('|%*s|', 10, 'foo');
```

结果：|       foo|

```
select format('|%*s|', -10, 'foo');
```

结果：|foo       |

```
select format('|%-*s|', 10, 'foo');
```

结果：|foo       |

```
select format('|%-*s|', -10, 'foo');
```

结果：|foo    
   |
这些示例展示了position域的示例：

```
select format('Testing %3$s, %2$s, %1$s', 'one', 'two', 'three');
```

结果：Testing three, two, one

```
select format('|%*2$s|', 'foo', 10, 'bar');
```

结果：|       bar|

```
select format('|%1$*2$s|', 'foo', 10, 'bar');
```

结果：|       foo|

#### initcap()
目的：将每个单词的第一个字母转换为大写，其余字母转换为小写。单词是由非字母数字字符分隔的字母数字字符序列。

语法：

输入值:      text
返回值:      text

例如：

```
select initcap('hi THOMAS');
```

返回信息如下：

```
 Hi Thomas
```

#### left()
目的：以字符串返回第一个 n 字符，或在 n 为负时, 返回最后 |n| 个字符之外的全部字符。

语法：

输入值:    text, integer     
返回值:    text  

例如：

```
select left('abcde', 2);
```

返回信息如下：

```
ab
```

#### length()
目的：返回字符串中的字符数。

语法：

输入值:   text    
返回值:   integer   

例如：

```
select length('jose');
```

返回信息如下：

```
4
```

#### lpad()
目的：将string扩展为长度length，通过前置字符fill（默认空格）。 如果string已经超过length那么它将被截断（在右侧）。

语法：

输入值:    text, integer, text    
返回值:    text    

例如：

```
select lpad('hi', 5, 'xy');
```

返回信息如下：

```
xyxhi
```

#### ltrim()
目的：从string开始删除包含characters（默认空格）中仅包含字符的最长字符串。

语法：

输入值:  text, text      
返回值:  text    

例如：

```
select ltrim('zzzytest', 'xyz');
```

返回信息如下：

```
test
```

#### md5()
目的：计算参数的 MD5 hash ，结果以十六进制形式写入。

语法：

输入值:  text     
返回值:  text    

例如：

```
select md5('abc');
```

返回信息如下：

```
900150983cd24fb0d6963f7d28e17f72
```

#### parse_ident ( qualified_identifier text [, strict_mode boolean DEFAULT true ] )

目的：将qualified_identifier拆分为一个标识符数组，删除单个标识符的任何引用。 默认情况下，最后一个标识符之后的额外字符被视为错误；但是，如果第二个参数为false，则忽略这些额外的字符。 （这种行为对于解析类似函数的对象的名称有作用。） 请注意，此函数不会截断超长标识符。如果你想截断，你可以把结果给到name[]。

语法：

输入值:  str text, strict boolean DEFAULT true     
返回值:  text[]     

例如：

```
select parse_ident('"SomeSchema".someTable');
```

返回信息如下：

```
{SomeSchema,sometable}
```

#### pg_client_encoding()
目的：返回当前客户端编码名称。

语法：

返回值:    name  

例如：

```
select pg_client_encoding();
```

返回信息如下：

```
UTF8
```

#### quote_ident()
目的：返回适合引用的给定字符串，作为SQL语句字符串中的标识符。 只有在必要的情况下才添加引号(例如，如果字符串包含非标识符字符或将被大小写折叠)。 嵌入的引号被适当地加双引号。

语法：

输入值:    text 
返回值:    text  

例如：

```
select quote_ident('Foo bar');
```

返回信息如下：

```
"Foo bar"
```

#### quote_literal()
目的：返回在SQL语句字符串中适当引用的给定字符串，用作字符串文字使用。 嵌入式单引号和反斜线适当的翻倍(转双引号或双斜线)。 请注意，quote_literal返回无效输入；如果这个参数可能为空，quote_nullable通常更合适。

语法：

输入值:    text   
返回值:   text   

例如：

```
select quote_literal(E'O\'Reilly');
```

返回信息如下：

```
'O''Reilly'
```

#### quote_literal()
目的：将给定的值转换为文本，然后将其作为字面量引用。 内嵌的单引号和反斜杠被适当地翻倍。

语法：

输入值:     anyelement  
返回值:     text    

例如：

```
select quote_literal(42.5);
```

返回信息如下：

```
'42.5'
```

#### quote_nullable()
目的：返回在SQL语句字符串中适当引用的给定字符串文字;或者，如果参数为null，则返回NULL。 内嵌的单引号和反斜杠被适当地翻倍。

语法：

输入值:    text  
返回值:    text 

例如：

```
select quote_nullable(NULL);
```

返回信息如下：

```
NULL
```

#### quote_nullable()
目的：将给定值转换为文本，然后将其作为字面量引用；或者，如果参数为null，则返回NULL。 内嵌的单引号和反斜杠被适当地翻倍。

语法：

输入值:   anyelement   
返回值:   text    

例如：

```
select quote_nullable(42.5);
```

返回信息如下：

```
'42.5'
```

#### regexp_match()
目的：返回从POSIX正则表达式到string的第一个匹配中捕获的子字符串

语法：

输入值:    text, text, text  
返回值:    text[]  

例如：

```
select regexp_match('foobarbequebaz', '(bar)(beque)');
```

返回信息如下：

```
 {bar,beque}
```

#### regexp_matches()
目的：返回通过将POSIX正则表达式与string匹配而捕获的子字符串

语法：

输入值:  text, text, text    
返回值:  SETOF text[]    

例如：

```
select regexp_matches('foobarbequebaz', 'ba.', 'g');
```

返回信息如下：

```
 {bar}
 {baz}
```

#### regexp_replace()
目的：替换匹配POSIX正则表达式的子字符串

语法：

输入值:   text, text, text, text    
返回值:   text   

例如：

```
select regexp_replace('Thomas', '.[mN]a.', 'M');
```

返回信息如下：

```
 ThM
```

#### regexp_split_to_array()
目的：使用POSIX正则表达式作为分隔符拆分string

语法：

输入值:   text, text, text   
返回值:   text[]     

例如：

```
select regexp_split_to_array('hello world', '\s+');
```

返回信息如下：

```
{hello,world}
```

#### regexp_split_to_table()
目的：使用POSIX正则表达式作为分隔符拆分string

语法：

输入值:   text, text, text   
返回值:   SETOF text   

例如：

```
select regexp_split_to_table('hello world', '\s+');
```

返回信息如下：

```
 hello
 world
```

#### repeat()
目的：重复string指定number的次数。

语法：

输入值:   text, integer   
返回值:   text   

例如：

```
select repeat('Pg', 4);
```

返回信息如下：

```
PgPgPgPg
```

#### replace()
目的：将string 中当前的子串替换

语法：

输入值:   text, text, text      
返回值:   text   

例如：

```
select replace('abcdefabcdef', 'cd', 'XX');
```

返回信息如下：

```
abXXefabXXef
```

#### reverse()
目的：颠倒字符串中字符的顺序。

语法：

输入值:   text  
返回值:   text   

例如：

```
select reverse('abcde');
```

返回信息如下：

```
edcba
```

#### right()
目的：返回字符串中的最后n个字符，或者在n>为负时，返回除了前面的|n|字符之外的所有字符。

语法：

输入值:   text, integer
返回值:   text   

例如：

```
select right('abcde', 2) ;
```

返回信息如下：

```
de
```

#### rpad()
目的：扩展 string 到长度 length，通过追加fill 字符(默认为空格). 如果string 已经比 length 长，则截断它。

语法：

输入值:    text, integer, text
返回值:    text   

例如：

```
select rpad('hi', 5, 'xy');
```

返回信息如下：

```
hixyx
```

#### rtrim()
目的：从string末尾删除包含characters（默认为空格）中仅包含字符的最长字符串。

语法：

输入值:    text, text
返回值:    text   

例如：

```
select rtrim('testxxzx', 'xyz');
```

返回信息如下：

```
test
```

#### split_part()
目的：在delimiter出现时拆分string，并且返回第n个字段(从一计数)。

语法：

输入值:    text, text, integer
返回值:    text   

例如：

```
select split_part('abc~@~def~@~ghi', '~@~', 2);
```

返回信息如下：

```
def
```

#### strpos()
目的：返回在string中指定的substring的起始索引,如果不存在则为零。

语法：

输入值:    text, text
返回值:   integer

例如：

```
select strpos('high', 'ig');
```

返回信息如下：

```
2
```

#### substr()
目的：提取string从start字符开始的子字符串，并扩展count字符，如果指定了的话。

语法：

输入值:   text, integer, integer
返回值:   text   

例如：

```
select substr('alphabet', 3, 2);
```

返回信息如下：

```
ph
```

#### starts_with()
目的：如果 string 以 prefix开始就返回真。

语法：

输入值:    text, text
返回值:    boolean

例如：

```
select starts_with('alphabet', 'alph');
```

返回信息如下：

```
t
```

#### to_ascii()
目的：将string从另一个编码中转换为ASCII，该编码可按名称或编号标识。如果encoding被省略，则假定数据库编码（这在实践中是唯一有用的案例）。转换主要包括降音。 转换仅支持来自 LATIN1、LATIN2、LATIN9、 和 WIN1250 的编码

语法：

输入值:   text  
返回值:   text  

或者

输入值:   text, integer
返回值:   text  

或者

输入值:   text, name
返回值:   text  

例如：

```
select to_ascii('abc','LATIN1');
```

返回信息如下：

```
abc
```

#### to_hex()
目的：将数字转换为其相应的十六进制表示形式。

语法：

输入值:    bigint
返回值:    text   

或者

输入值:    integer
返回值:    text   

例如：

```
select to_hex(2147483647);
```

返回信息如下：

```
7fffffff
```

#### translate()
目的：将string中与from集合中匹配的每个字符替换为to集合中相应的字符。 如果from长于to，from中出现的额外字符被删除。

语法：

输入值:    text, text, text
返回值:    text   

例如：

```
select translate('12345', '143', 'ax');
```

返回信息如下：

```
a2x5
```

