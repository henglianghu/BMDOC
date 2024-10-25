BOOLEAN数据类型表示三种不同的状态：TRUE、FALSE或NULL。

描述：

type_specification ::= { BOOLEAN | BOOL }
literal ::= { TRUE  | true | 't' | 'y' | 'yes' | 'on' | 1 |
          FALSE | false| 'f' | 'n' | 'no' | 'off' | 0 }

例如：

```
CREATE TABLE test1 (a boolean, b text);
INSERT INTO test1 VALUES (TRUE, 'sic est');
INSERT INTO test1 VALUES (FALSE, 'non est');
select * FROM test1;
```

返回信息如下：

```
 a |    b
---+---------
 t | sic est
 f | non est
select * FROM test1 WHERE a;
```

返回信息如下：

```
 a |    b
---+---------
 t | sic est
```

