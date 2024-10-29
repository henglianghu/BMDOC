SMALLSERIAL、SERIAL和BIGSERIAL分别是SMALLINT、INTEGER和BIGINT序列的简短表示法。 
type_specification ::= SMALLSERIAL | SERIAL | BIGSERIAL

注：

* serial 类型的列会自动递增。
* serial 并不意味着在列上创建了索引。

例如：

```
CREATE TABLE t (id SERIAL);
```

 