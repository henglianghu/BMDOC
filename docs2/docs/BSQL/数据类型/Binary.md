使用BYTEA数据类型表示字节的二进制字符串（八位字节）。二进制串明确允许存储零值的字节和不可打印的字节（通常是位于十进制范围32到126之外的字节）。
描述：

* BYTEA用于声明一个二进制实体。

```
type_specification ::= BYTEA
```

* 转义输入可用于输入二进制数据。

```
select E'\\001'::bytea
```

Bytea字节默认被输出为hex格式。如果你把bytea_output改为escape，“不可打印的”字节会被转换成与之等效的三位八进制值并且前置一个反斜线。大部分“可打印的”字节被输出为它们在客户端字符集中的标准表示形式，例如：

```
SET bytea_output = 'escape';
 
select 'abc \153\154\155 \052\251\124'::bytea;
```

返回信息如下：

```
abc klm *\251T
```

十进制值为92（反斜线）的字节在输出时被双写，如下表格所示：

| 十进制字节值    | 描述             | 转义的输出表示   | 示例          | 输出结果 |
| --------------- | ---------------- | ---------------- | ------------- | -------- |
| 92              | 反斜线           | \\\\             | '\134'::bytea | \\\\     |
| 0到31和127到255 | “不可打印的”字节 | \xxx（八进制值） | '\001'::bytea | \001     |
| 32到126         | “可打印的”字节   | 客户端字符集表示 | '\176'::bytea | ~        |


## **函数和操作符**

### **操作符**

| 操作符         | 描述                   |
| -------------- | ---------------------- |
| [\|\|](#_\|\|) | 连接两个二进制字符串。 |

#### 9.4.1.2.1.1.1 ||
目的：连接两个二进制字符串。

例如：

```
select '\x123456'::bytea || '\x789a00bcde'::bytea;
```

返回信息如下：

```
\x123456789a00bcde
```

### **函数**

| 函数                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [bit_length()](#_bit_length())                               | 返回二进制字符串中的位数                                     |
| [octet_length()](#_octet_length())                           | 返回二进制字符串中的字节数。                                 |
| [overlay(string placing string from int [for int])](#_overlay(string placing string from int [for int])) | 将bytes的子字符串替换为newsubstring，该子字符串从start字节开始，并以count字节扩展。 如果忽略了count，则默认为newsubstring的长度。 |
| [position(substring in string)](#_position(substring in string)) | 返回指定的substring在bytes内的起始索引，如果不存在，则为零。 |
| [substring(string [from int] [for int])](#_substring(string [from int] [for int])) | 提取bytes从start字节开始的子字符串，如果指定了，并且在count字节之后停止，如果指定了的话。 至少提供start和count中的一个。 |
| [trim([both] bytes from string)](#_trim([both] bytes from string)) | 从开始和结束处，删除只包含字节的最长字符串。                 |
| [btrim()](#_btrim())                                         | 从开始和结束处删除只包含出现的字节的最长字符串。             |
| [decode()](#_decode())                                       | 从文本表示中解码二进制数据;支持的format值与encode相同。      |
| [encode()](#_encode())                                       | 将二进制数据编码成文本表示；支持的format值为： base64, escape, hex。 |
| [get_bit()](#_get_bit())                                     | 从二进制字符串中提取bit。                                    |
| [get_byte()](#_get_byte())                                   | 从二进制字符串中提取字节。                                   |
| [length()](#_length())                                       | 返回二进制字符串中的字节数。                                 |
| [length ()](#_length()_1)                                    | 返回二进制字符串中的字符数，假设它是给定encoding中的文本。   |
| [md5()](#_md5())                                             | 计算二进制字符串的MD5 hash，结果以十六进制形式写入。         |
| [set_bit()](#_set_bit())                                     | 设置二进制字符串中bit位。                                    |
| [set_byte()](#_set_byte())                                   | 设置二进制字符串中的字节。                                   |
| [sha224()](#_sha224())                                       | 计算二进制字符串的 SHA-224 hash。                            |
| [sha256()](#_sha256())                                       | 计算二进制字符串的 SHA-256 hash。                            |
| [sha384()](#_sha384())                                       | 计算二进制字符串的 SHA-384hash。                             |
| [sha512()](#_sha512())                                       | 计算二进制字符串的 SHA-512hash。                             |
| [substr()](#_substr())                                       | 从start字节开始提取bytes的子字符串，并扩展为count字节，如果这是指定的。 |

#### bit_length()
目的：返回二进制字符串中的位数，8 倍于 octet_length

语法：

输入值:      bytea
返回值:      integer  

例如：

```
select bit_length('\x123456'::bytea);
```

返回信息如下：

```
24
```

#### octet_length()
目的：返回二进制字符串中的字节数。

语法：

输入值:    bytea  
返回值:    integer  

例如：

```
select octet_length('\x123456'::bytea);
```

返回信息如下：

```
3
```

#### overlay(string placing string from int [for int])
目的：将bytes的子字符串替换为newsubstring，该子字符串从start字节开始，并以count字节扩展。 如果忽略了count，则默认为newsubstring的长度。

语法：

输入值:     string placing string from int [for int] 
返回值:     bytea 

例如：

```
select overlay('\x1234567890'::bytea placing '\002\003'::bytea from 2 for 3);
```

返回信息如下：

```
\x12020390
```

#### position(substring in string)
目的：返回指定的substring在bytes内的起始索引，如果不存在，则为零。

语法：

输入值:     substring in string 
返回值:     integer

例如：

```
select position('\x5678'::bytea in '\x1234567890'::bytea);
```

返回信息如下：

```
3
```

#### substring(string [from int] [for int])
目的：提取bytes从start字节开始的子字符串，如果指定了，并且在count字节之后停止，如果指定了的话。 至少提供start和count中的一个。


语法：

输入值:      string [from int] [for int]
返回值:      bytea

例如：

```
select substring('Th\000omas'::bytea from 2 for 3); 
```

返回信息如下：

```
\x68006f
```

#### trim([both] bytes from string)
目的：从开始和结束处，删除只包含字节的最长字符串。

语法：

输入值:      [both] bytes from string
返回值:      bytea

例如：

```
select trim('\x9012'::bytea from '\x1234567890'::bytea); 
```

返回信息如下：

```
\x345678
```

#### btrim()
目的：从开始和结束处删除只包含出现的字节的最长字符串

语法：

输入值:     bytea, bytea 
返回值:     bytea 

例如：

```
select btrim('\x1234567890'::bytea, '\x9012'::bytea); 
```

返回信息如下：

```
\x345678
```

#### decode()
目的：从文本表示中解码二进制数据;支持的format值与encode相同。

语法：

输入值:     text, text 
返回值:     bytea 

例如：

```
select decode('MTIzAAE=', 'base64');  
```

返回信息如下：

```
\x3132330001
```

#### encode()
目的：将二进制数据编码成文本表示；支持的format值为： base64, escape, hex.

语法：

输入值:     bytea, text 
返回值:     text 

例如：

```
select encode('123\000\001', 'base64'); 
```

返回信息如下：

```
MTIzAAE=
```

#### get_bit()
目的：从二进制字符串中提取bit 。

语法：

输入值:      bytea, integer
返回值:      integer

例如：

```
select get_bit('\x1234567890'::bytea, 30);  
```

返回信息如下：

```
1
```

#### get_byte()
目的：从二进制字符串中提取字节。

语法：

输入值:      bytea, integer
返回值:      integer

例如：

```
select get_byte('\x1234567890'::bytea, 4);  
```

返回信息如下：

```
144
```

#### length()
目的：返回二进制字符串中的字节数。

语法：

输入值:      bytea
返回值:      integer

例如：

```
select length('\x1234567890'::bytea); 
```

返回信息如下：

```
5
```

#### length()
目的：返回二进制字符串中的字符数，假设它是给定encoding中的文本。

语法：

输入值:      bytea, name 
返回值:      integer

例如：

```
select length('jose'::bytea, 'UTF8'); 
```

返回信息如下：

```
4
```

#### md5()
目的：计算二进制字符串的MD5 hash，结果以十六进制形式写入。

语法：

输入值:     bytea 
返回值:     text 

例如：

```
select md5('Th\000omas'::bytea); 
```

返回信息如下：

```
8ab2d3c9689aaf18b4958c334c82d8b1
```

#### set_bit()
目的：设置二进制字符串中bit位。

语法：

输入值:      bytea, integer, integer
返回值:      bytea

例如：

```
select set_bit('\x1234567890'::bytea, 30, 0);  
```

返回信息如下：

```
\x1234563890
```

#### set_byte()
目的：设置二进制字符串中的字节。

语法：

输入值:      bytea, integer, integer
返回值:      bytea

例如：

```
select set_byte('\x1234567890'::bytea, 4, 64);  
```

返回信息如下：

```
\x1234567840
```

#### sha224()
目的：计算二进制字符串的 SHA-224 hash。

语法：
输入值:      bytea
返回值:      bytea

例如：

```
select sha224('abc'::bytea);  
```

返回信息如下：

```
\x23097d223405d8228642a477bda255b32aadbce4bda0b3f7e36c9da7
```

#### sha256()
目的：计算二进制字符串的 SHA-256 hash。

语法：

输入值:     bytea 
返回值:     bytea 

例如：

```
select sha256('abc'::bytea);  
```

返回信息如下：

```
\xba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad
```

#### sha384()
目的：计算二进制字符串的 SHA-384hash。

语法：

输入值:     bytea 
返回值:     bytea 

例如：

```
select sha384('abc'::bytea);  
```

返回信息如下：

```
\xcb00753f45a35e8bb5a03d699ac65007272c32ab0eded1631a8b605a43ff5bed8086072ba1e7cc2358baeca134c825a7
```

#### sha512()
目的：计算二进制字符串的 SHA-512hash。

语法：

输入值:      bytea
返回值:      bytea

例如：

```
select sha512('abc'::bytea); 
```

返回信息如下：

```
\xddaf35a193617abacc417349ae20413112e6fa4e89a97ea20a9eeee64b55d39a2192992a274fc1a836ba3c23a3feebbd454d4423643ce80e2a9ac94fa54ca49f
```

#### substr()
目的：从start字节开始提取bytes的子字符串，并扩展为count字节，如果这是指定的。

语法：

输入值:      bytea, integer, integer
返回值:      bytea

例如：

```
select substr('\x1234567890'::bytea, 3, 2); 
```

返回信息如下：

```
\x5678
```

