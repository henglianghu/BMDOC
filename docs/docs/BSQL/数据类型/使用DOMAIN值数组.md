DOMAIN值的数组可以创建一个一维数组，其值本身就是不同长度的一维数组。概括地说，它允许您实现一个不规则的多维数组，从而打破了数组通常必须是线性化的限制。

## **示例：GPS行程数据**

在某些用例中，不规则的结构是必不可少的。因此，大多数编程语言都会支持这一特性。
下面来查看此示例：GPS行程数据。考虑GPS的行程，其记录被分解为圈，因此：

* 每次行程由一圈，或多圈组成。
* 每圈通常由大量的GPS数据点组成。 

以下的实现可以达到期望： 

* 每个圈都被表示为“圈”表中的一行，该表含有作为GPS数据点数组实现的多值字段。
* 但每一次行程都由“行程”表中的一行代表，该行的圈数是“圈数”表中的某一子行。这个父子关系通常由外键约束来支持。但请注意，每圈的GPS点的数量可能与其他圈的点的数量不相同。

接下来，下一个目标，是通过将整个行程表示为“行程”表中的一个不规则数组，而省去了单独的“圈数”表。该方案要求能够将行程表示为“GPS数据点数组”数据类型。换句话说，这取决于创建此类命名数据类型的能力，而DOMAIN 具有此种能力。

## **创建不规则数组**

### 矛盾
乍一看，定义表列、PL/pgSQL变量或形式参数的语法似乎存在阻碍目标实现的矛盾。例如，这个PL/pgSQL声明显然将“v”定义为一维数组。

```
declare v int[];
```

然而，这个可执行部分首先将一个二维数组值（大小为三乘三）分配给“v”，将数组的每个值初始化为17。然后，它将三维数组值（大小为2乘2乘2）分配给“v”，将数组的每个值初始化为42：

```
    begin
    v := array_fill(17, '{3, 3}');
    ...
    v := array_fill(42, '{2, 2, 2}');
```

数组数据类型和功能中指出了数组变量声明的属性，即它无法修复随后分配给该变量的值的维度。表中具有数组数据类型的列共享此属性，以便该列可以在不同的行中容纳不同维度的数组。这与以下“v1”和“v2”的声明虽然明显不同，但定义了相同的语义这一事实密切相关。

```
declare
  v1 int[];
  v2 int[][];
```

这种句法上的怪癖恰恰说明了这个矛盾。“v2”是二维数组，还是int[]类型的数组？答案是两者都不是。相反，它是一个任意维度的数组。

那么如何编写数组的声明，你可能会猜测这会起作用：

```
declare
  v (int[])[];
 
But this causes a compilation error. The paradox is exactly that int[] is anonymou
```

但这会导致编译错误。矛盾之处恰恰在于int[]是匿名的。

### 解决方案
DOMAIN带来了打破限制的功能。首先，请执行以下操作：

```
set client_min_messages = warning;
drop domain if exists int_arr_t cascade;
create domain int_arr_t as int[];
create table t(k serial primary key, v1 int_arr_t[], typecast text, v2 int_arr_t[]);
```

请注意，CREATE DOMAIN语句的使用是类型构造的一个示例。用户定义的数据类型“int_arr_t”就是这种构造的数据类型的一个示例。

列“v1”和“v2”现在可以存储数组的不规则数组。如下证明：

```
do $body$
declare
  arr_1      constant int_arr_t := array[1, 2];
  arr_2      constant int_arr_t := array[3, 4, 5];
  ragged_arr constant int_arr_t[] := array[arr_1, arr_2];
begin
  insert into t(v1) values(ragged_arr);
end;
$body$;
```

通过使用DO块，通过自下而上构建来设置“ragged_arr”的值，可以强调它实际上是一个由不同长度的一维数组组成的一维数组。那么，它显然不是一个二维线性化数组。

现在使用[值到文本的类型转换和返回值](#_值到文本的类型转换和返回值)解释的技术来检查不规则数组的::text类型转换，通过类型转换转换回原始数据类型的值。首先，请执行以下操作： 

```
update t
set typecast = v1::text
where k = 1;
 
select typecast from t where k = 1;
```

返回信息如下：

```
      typecast
---------------------
 {"{1,2}","{3,4,5}"}
```


接下来：

```
update t
set v2 = typecast::int_arr_t[]
where k = 1;
 
select (v1 = v2)::text as "v1 = v2" from t where k = 1;
```

返回信息如下：

```
 v1 = v2 
---------
 true
```

已重新创建原始值。

### 处理数组的不规则数组
首先，考虑这个反例：

```
\pset null '<IS NULL'>
with v as (
  select  '{{1,2},{3,4}}'::int[] as two_d_arr)
select
  two_d_arr[2][1] as "[2][1] -- meaningful",
  two_d_arr[2]    as    "[2] -- meaningless"
from v;
```

返回信息如下：

```
 [2][1] -- meaningful | [2] -- meaningless
----------------------+--------------------
                3 |     <IS NULL>
```

如何处理线性多维数组中的单个值？可以使用以下通用方案：

```
[idx_1][idx_2]...[idx_n]
```

必须提供与数组的维数完全相同的索引值。此外，如果您做错了，并且提供了太多或太少的索引值，那么您不会看到错误，而是得到值NULL。在此，一个显而易见的问题是： 
如何寻址？例如，如何寻址数组中的第一个值，而该值本身就是数组的不规则数组中的第二个数组？

在输入之前，您知道这不可能是正确的：

```
select v1[2][1] as "v1[2][1]" from t where k = 1;
```

返回信息如下：

```
 v1[2][1]
-----------
 <IS NULL>
```

不会出错，但也得不到想要的。请尝试以下操作： 

```
select v1[2] as "v1[2]" from t where k = 1;
```

返回信息如下：

```
  v1[2]
---------
 {3,4,5}
```

您已经在数组的数组中标识了想要的叶数组。现在您必须在该数组中标识所需的值。试试这个：

```
select (v1[2])[1] as "(v1[2])[1]" from t where k = 1;
```

返回信息如下：

```
 (v1[2])[1]
------------
          3
```

同样，您可以如下检查叶数组的几何属性，如下所示：

```
select
  array_lower(v1[1], 1) as v1_lb,
  array_upper(v1[1], 1) as v1_ub,
  array_lower(v1[2], 1) as v2_lb,
  array_upper(v1[2], 1) as v2_ub
from t where k = 1;
```

返回信息如下：

```
 v1_lb | v1_ub | v2_lb | v2_ub
-------+-------+-------+-------
     1 |    2 |    1 |    3
```

最后，试试这个反例：

```
with v as (
  select  '{{1,2},{3,4}}'::int[] as two_d_arr)
select
  (two_d_arr[2])[1]
from v;
```

它会导致SQL编译错误。您必须知道，要在其中寻址值的当前值是线性化多维数组，还是不规则的数组的数组。

### 将FOREACH与DOMAINs数组结合使用
这个示例解释说明了这个问题。

```
set client_min_messages = warning;
drop domain if exists array_t cascade;
drop domain if exists arrays_t cascade;
 
create domain array_t  as int[];
create domain arrays_t as array_t[];
 
\set VERBOSITY verbose
do $body$
declare
  arrays arrays_t := array[
    array[1, 2]::array_t, array[3, 4, 5]::array_t];
 
  runner array_t not null := '{}';
begin
  -- Error 42804 here.
  foreach runner in array arrays loop
    raise info '%', runner::text;
  end loop;
end;
$body$;
```

引发如下错误：

```
ERROR:  42804: FOREACH expression must yield an array, not type arrays_t
```

错误文本可能会使您感到困惑。实际上，这意味着循环头中ARRAY关键字的参数必须是一个显式数组，而不是为这样的数组命名的domain 。 

一个简单的解决方法是将“arrays”显式声明为“array_t[]”，而不是使用“array_t”作为简写。如下所示：

```
\set VERBOSITY default
do $body$
declare
  arrays array_t[] := array[
    array[1, 2]::array_t, array[3, 4, 5]::array_t];
 
  runner array_t not null := '{}';
begin
  foreach runner in array arrays loop
    raise info '%', runner::text;
  end loop;
end;
$body$;
```

返回信息如下：

```
INFO:  {1,2}
INFO:  {3,4,5}
```

如果真的需要使用DOMAIN怎么办？例如，您可能希望定义这样的约束：

```
set client_min_messages = warning;
drop domain if exists arrays_t cascade;
create domain arrays_t as array_t[]
check ((cardinality(value) = 2));
```

解决方法是：将ARRAY关键字的参数类型转换为适当元素数据类型的数组。 

```
do $body$
declare
  arrays arrays_t := array[
    array[1, 2]::array_t, array[3, 4, 5]::array_t];
 
  runner array_t not null := '{}';
begin
  foreach runner in array arrays::array_t[] loop
    raise info '%', runner::text;
  end loop;
end;
$body$;
```

返回信息如下：

```
INFO:  {1,2}
INFO:  {3,4,5}
```

### 使用array_agg()生成DOMAIN值数组
事实证明，不支持直接聚合表示不规则数组的DOMAIN值。但是一个简单的PL/pgSQL函数提供了解决方法。

```
set client_min_messages = warning;
drop table if exists t cascade;
drop domain if exists array_t cascade;
drop domain if exists arrays_t cascade;
 
create domain array_t  as int[];
create domain arrays_t as array_t[];
 
create table t(k serial primary key, v array_t);
 
insert into t(v) values
  ('{2,6}'),
  ('{1,4,5,6}'),
  ('{4,5}'),
  ('{2,3}'),
  ('{4,5}'),
  ('{3,5,7}');
 
select v from t order by k;
```

返回信息如下：

```
     v
-----------
 {2,6}
 {1,4,5,6}
 {4,5}
 {2,3}
 {4,5}
 {3,5,7}
```

如下说明了问题：

```
\set VERBOSITY verbose
select
  array_agg(v order by k)
from t;
```

会返回如下错误信息：

```
2202E: cannot accumulate arrays of different dimensionality
```

类型转换不能生效，但此函数会产生所需的结果：

```
\set VERBOSITY default
create or replace function array_agg_v()
  returns arrays_t
  language plpgsql
as $body$
<<b>>declare
  v  array_t    not null := '{}';
  n  int        not null := 0;
  r  array_t[]  not null := '{}';
begin
  for b.v in (select t.v from t order by k) loop
    n := n + 1;
    r[n] := b.v;
  end loop;
  return r;
end b;
$body$;
```

执行：

```
select array_agg_v();
```

返回信息如下：

```
                       array_agg_v
------------------------------------------------------------------------
 {"{2,6}","{1,4,5,6}","{4,5}","{2,3}","{4,5}","{3,5,7}"}
```

## **创建矩阵**

数学、物理学等领域的各种学科都使用块矩阵。数组的使用说明了这种情况如何在客户端程序中生成各种数组，以及以后需要在客户端程序中将这些值再次使用。在当前用例中，这带来了持久化和检索块矩阵的需求。
尽管这个用例相对来说比较奇特，但用于实现所需结构的技术（尤其是可行解决方案对用户定义的DOMAIN数据类型的依赖性）具有通用性。正是由于这个原因，这里解释了这种方法。

### 定义需要的数据类型

首先，定义“有效负载”矩阵的数据类型。假设需要执行以下规则：

* 有效负载矩阵在原子上不能为null。
* 根据定义，它必须是二维的。
* 一定是三乘三。
* 每个维度的下限必须为1。
* 矩阵的每个值都不能是NULL。

DOMAIN类型构造函数的要求是，它必须允许在构造类型的值上定义任意约束，如下所示：

```
create domain matrix_t as text[]
check (
  (value is not null)         and
  (array_ndims(value) = 2)    and
  (array_lower(value, 1) = 1) and
  (array_lower(value, 2) = 1) and
  (array_upper(value, 1) = 3) and
  (array_upper(value, 2) = 3) and
  (value[1][1] is not null  ) and
  (value[1][2] is not null  ) and
  (value[1][3] is not null  ) and
  (value[2][1] is not null  ) and
  (value[2][2] is not null  ) and
  (value[2][3] is not null  ) and
  (value[3][1] is not null  ) and
  (value[3][2] is not null  ) and
  (value[3][3] is not null  )
);
```

接下来，定义一个“matrix_t”的块矩阵。假设需要执行类似的以下规则：

* 块矩阵在原子上不能为null。
* 根据定义，它必须是二维的。
* 一定是二乘二。
* 每个维度的下限必须为1。
* 矩阵的每个值都必须不是NULL。 

因此，“block_matrix_t”的CREATE DOMAIN语句，与“matrix_t”的语句类似：

```
create domain block_matrix_t as matrix_t[]
check (
  (value is not null)         and
  (array_ndims(value) = 2)    and
  (array_lower(value, 1) = 1) and
  (array_lower(value, 2) = 1) and
  (array_upper(value, 1) = 2) and
  (array_upper(value, 2) = 2) and
  (value[1][1] is not null  ) and
  (value[1][2] is not null  ) and
  (value[2][1] is not null  ) and
  (value[2][2] is not null  )
);
```

这两个CREATE DOMAIN语句冗长而重复，但它们足以说明该方法的基础。为了在实际应用程序中使用，最好将所有CHECK规则封装在PL/pgSQL函数中，该函数以DOMAIN值为输入并返回布尔值，并将其用作单个CHECK谓词。该函数可以使用array_lower（）和array_length（）函数来计算两个嵌套FOR循环的范围，以检查数组的各个值是否都满足NOT NULL规则。

### 使用“block_matrix_t”DOMAIN
接下来，创建一个块矩阵值，将其及其 ::text 类型转换插入到表中，并检查类型转换的值。

```
create table block_matrices_1(k int primary key, v block_matrix_t, text_typecast text);
 
do $body$
declare
  -- The definitions of the two domains imply "not null" constraints
  -- on each of the variables "matrix_t" and "block_matrix_t".
  m matrix_t       := array_fill('00'::text, array[3, 3], array[1, 1]);
  b block_matrix_t := array_fill(m,          array[2, 2], array[1, 1]);
 
  n int not null := 0;
  ms matrix_t[];
begin
  -- Define four matrix_t values so that, for readability of the result,
  -- the in-total 24 values are taken from an increasing dense series.
  for i in 1..4 loop
    for j in 1..3 loop
      for k in 1..3 loop
        n := n + 1;
        m[j][k] := ltrim(to_char(n, '09'));
      end loop;
    end loop;
    ms[i] := m;
  end loop;
 
  n := 0;
  for j in 1..2 loop
    for k in 1..2 loop
      n := n + 1;
      b[j][k] := ms[n];
    end loop;
  end loop;
 
  insert into block_matrices_1(k, v, text_typecast)
  values(1, b, b::text);
end;
$body$;
 
select text_typecast
from block_matrices_1
where k = 1;
```

返回信息如下（为方便阅读，手动增加了空格）：

```
{
  {
    "{
      {01,02,03},     block_matrix[1][1]
      {04,05,06},
      {07,08,09}
    }",
    "{
      {10,11,12},     block_matrix[1][2]
      {13,14,15},
      {16,17,18}
    }"
  },
  {
    "{
      {19,20,21},     block_matrix[2][1]
      {22,23,24},
      {25,26,27}
    }",
    "{
      {28,29,30},     block_matrix[2][2]
      {31,32,33},
      {34,35,36}
    }"
  }
}
```

注意：注释“block_matrix”等等这些信息是手动添加的，只是为了突出显示整个文本值的含义。

最后，检查一下结构是否符合通用规则：

```
create table block_matrices_2(k int primary key, v block_matrix_t);
 
insert into block_matrices_2(k, v)
select k, text_typecast::block_matrix_t
from block_matrices_1
where k = 1;
 
with a as (
  select k, t1.v as v1, t2.v as v2
  from
  block_matrices_1 as t1
  inner join
  block_matrices_2 as t2
  using (k)
  )
select (v1 = v2)::text as "v1 = v2"
from a
where k = 1;
```

返回信息如下：

```
 v1 = v2
---------
 true
```

### 对数组使用unnest()
首先，按行主要顺序生成“matrix_t”值列表：

```
with matrices as (
  select unnest(v) as m
  from block_matrices_1
  where k = 1)
select
  row_number() over(order by m) as r,
  m
from matrices order by m;
```

返回信息如下：

```
 r |                 m
---+----------------------------------------------
 1 | {{01,02,03},{04,05,06},{07,08,09}}
 2 | {{10,11,12},{13,14,15},{16,17,18}}
 3 | {{19,20,21},{22,23,24},{25,26,27}}
 4 | {{28,29,30},{31,32,33},{34,35,36}}
```

现在运行unnest：

```
select unnest(v[2][1]) as val
from block_matrices_1
where k = 1
order by val;
```

返回信息如下：

```
 val 
-----
 19
 20
 21
 22
 23
 24
 25
 26
 27
```

如果希望按行主顺序查看所有叶值，请使用此查询：

```
with
  matrixes as (
    select unnest(v) as m
    from block_matrices_1
    where k = 1),
  vals as (
    select unnest(m) as val
    from matrixes)
select
  row_number() over(order by val) as r,
  val
from vals
order by 1;
```

返回信息如下：

```
 r  | val
----+-----
  1 | 01
  2 | 02
  3 | 03
 ...
 34 | 34
 35 | 35
 36 | 36
```