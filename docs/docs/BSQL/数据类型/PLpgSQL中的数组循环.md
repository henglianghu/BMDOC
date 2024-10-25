PL/pgSQL FOREACH循环是用于在数组上循环的专用语法。

## **概述**

注意：下面有关以下涉及到的函数描述，请参见array_lower()、array_upper()、 array_ndims() 和 cardinality() 。涉及到的“行主顺序”术语，请参阅Joint中的“数组几何属性函数”章节。语法和语义显示了FOREACH循环头中SLICE关键字的使用位置。

* 当SLICE子句的操作数为0时，适用于具有任意数量维度的迭代数组的一般情况，BSQL将其连续值按行主顺序分配给循环迭代器。在这里，它的效果在功能上类似于unnest()。 
* 对于迭代数组为一维的特殊情况，只有当SLICE子句的操作数为0时，FOREACH循环才有用。在使用中，它是一种语法上更紧凑的方式来实现FOR var In循环所实现的效果，如下所示：

```
for var in array_lower(iterand_arr, 1)..array_upper(iterand_arr, 1) loop
  ... iterand_arr[var] ...
end loop;
```

* 当SLICE子句的操作数大于0，并且迭代数组的维度大于1时，FOREACH循环提供unnest()无法提供的功能。简单地说，当迭代数组有n个维度，并且SLICE子句的操作数是s时，BSQL将维度为s的切片（即子数组）分配给迭代器。这样一个切片中的值是迭代数组中的值，当使用第一个 (n - s) 索引的不同值来驱动迭代时，这些值会保留下来。下面两个伪代码块解释了这个想法：

伪代码一：

```
-- In this example, the SLICE operand is 1.
-- As a consequence, array_ndims(iterator_array) is 1.
-- Assume that array_ndims(iterand_arr) is 4.
-- There are therefore (4 - 1) = 3 nested loops in this pseudocode.
for i in array_lower(iterand_arr, 1)..array_upper(iterand_arr, 1) loop
  for j in array_lower(iterand_arr, 2)..array_upper(iterand_arr, 2) loop
    for k in array_lower(iterand_arr, 3)..array_upper(iterand_arr, 3) loop
 
      the (i, j, k)th iterator_array is set to
        iterand_arr[i][j][k][ for all values the remaining 4th index ]
 
    end loop;
  end loop;
end loop;
```

伪代码二：

```
-- In this example, the SLICE operand is 3.
-- As a consequence, array_ndims(iterator_array) is 3.
-- Assume that array_ndims(iterand_arr) is 4.
-- There is therefore (4 - 3) = 1 nested loop in this pseudocode.
for i in array_lower(iterand_arr, 1)..array_upper(iterand_arr, 1) loop
 
  the (i)th iterator_array is set to
    iterand_arr[i][ for all values the remaining 2nd, 3rd, and 4th indexes ]
 
end loop;
```

下面的示例阐明了FOREACH的行为。

## **语法和语义**

基本语法：
[ <\<label>> ]
FOREACH var [ SLICE non_negative_integer_literal ] IN ARRAY expression LOOP
  statements
END LOOP [ label ];

* var必须在FOREACH循环之前显式声明。
* 可选SLICE子句的操作数必须是非负整数。
* 假设表达式的数据类型为some_type[]。当使用SLICE 0时，var必须声明为some_type。
  当SLICE子句的操作数为正时，var必须声明为some_type[]。
  SLICE 0 has the same effect as omitting the SLICE clause.
* SLICE 0等同于省略SLICE子句。
* 当使用SLICE 0或省略SLICE子句时，BSQL将数组中按行主顺序访问的每个值依次分配给var。
* 当SLICE子句的操作数为正时，BSQL根据以下规则将迭代数组的连续切片分配给var：
  1）所提取的切片都具有由该等式给出的相同维度：array_ndims(iterator_array) = slice_operand。
  2）提取的切片的数量由以下等式给出：number_of_slices = cardinality(iterand_array)/cardinality(iterator_array)
  3）SLICE操作数的值不得超过 array_ndims(iterator_array)
  4）当SLICE操作数的值等于array_ndims(iterator_array) - 1时，FOREACH循环只生成一个迭代器数组值，这与迭代数组相同。
* 因此，SLICE操作数的有用范围是：0 到 (array_ndims(iterator_array) - 1) 

## **在不带SLICE关键字的数组中循环使用值**

该循环在功能上与FOR var IN循环相同。然而，在FOR循环中，BSQL会自动定义类型为int的var，其范围仅限于循环；但在FOREACH循环中，必须显式声明var，如上面“[语法和语义](#_语法和语义)”部分所述。
示例如下：

```
do $body$
declare
  arr1 int[] := array[1, 2];
  arr2 int[] := array[3, 4, 5];
  var int;
begin
  <<"FOREACH eaxmple">>
  foreach var in array arr1||arr2 loop
    raise info '%', var;
  end loop "FOREACH eaxmple";
end;
$body$;
```

显示信息如下（为方便阅读，手动去除了“INFO:”提示）：

```
1
2
3
4
5
```

下一个循环示例显示了以下注意事项： 

* 迭代数组是多维的
* 数组类型为“rt[]”，其中rt是用户定义的“row”类型。
* 上面使用“var”的语法点不需要由单个变量独占。相反，可以使用“f1”和“f2”来对应“rt”中的字段。
* FOREACH循环后面跟着一个“cursor”循环，其SELECT语句使用unnest()。
* FOREACH循环比“cursor”循环更简洁。您可以使用一对声明的变量“f1”和“f2”作为迭代器，而不会有任何麻烦（就像您可以使用类型为“rt”的单个变量“r”一样）。BSQL在这两种情况下都会处理正确的赋值。但是当你使用unnest()时，你必须自己处理这个问题。

```
create type rt as (f1 int, f2 text);
 
do $body$
declare
  a1 rt[] := array[(1, 'dog'), (2, 'cat')];
  a2 rt[] := array[(3, 'ant'), (4, 'rat')];
  arr rt[] := array[a1, a2];
  f1 int;
  f2 text;
begin
  raise info '';
  raise info 'FOREACH';
  foreach f1, f2 in array arr loop
    raise info '% | %', f1::text, f2;
  end loop;
 
  raise info '';
  raise info 'unnest()';
  for f1, f2 in (
    with v as (
      select unnest(arr) as r)
    select (r).f1, (r).f2 from v)
  loop
    raise info '% | %', f1::text, f2;
  end loop;
end;
$body$;
```

显示信息如下（为方便阅读，手动去除了“INFO:”提示）：

```
FOREACH
1 | dog
2 | cat
3 | ant
4 | rat
 
unnest()
1 | dog
2 | cat
3 | ant
4 | rat
```

这表明FOREACH循环的使用（隐含的0作为SLICE子句的操作数）在功能上等同于 unnest()。

## **非零SLICE操作数在多维数组的内容上循环**

首先，将三维二乘二乘二的数组存储在表中的 ::text[]字段中。随后，每个DO块都会使用它。
示例如下：

```
create table t(k int primary key, arr text[] not null);
insert into t(k, arr) values(1, '
  {
    {
      {001,002},
      {003,004}
    },
    {
      {005,006},
      {007,008}
    }
  }'::text[]);
```

接下来，显示SLICE操作数使用错误值时的结果：

```
do $body$
declare
  arr constant text[] not null := (select arr from t where k = 1);
  slice_iterator text[] not null := '{?}';
begin
  raise info 'array_ndims(arr): %', array_ndims(arr)::text;
  foreach slice_iterator slice 4 in array arr loop
    raise info '%', slice_iterator::text;
  end loop;
end;
$body$;
```

执行，会显示array_ndims（arr）：3，并且报告如下错误：

```
ERROR:  slice dimension (4) is out of the valid range 0..3
```

此测试确认SLICE操作数的值不得超过迭代数组的维度。
如前所述，SLICE 0（或等效地省略SLICE子句）按行主顺序扫描数组值。下一个测试，使用SLICE 3，演示了当SLICE操作数等于迭代数组的维度时的含义。

```
do $body$
declare
  arr constant text[] not null := (select arr from t where k = 1);
  slice_iterator text[] not null := '{?}';
  n int not null := 0;
begin
  raise info 'FOREACH SLICE 3';
  n := 0;
  foreach slice_iterator slice 3 in array arr loop
    assert
      (array_ndims(slice_iterator) = 3) and
      (slice_iterator = arr)           ,
    'assert failed';
    n := n + 1;
    raise info '% | %', n::text, slice_iterator::text;
  end loop;
end;
$body$;
```

输出信息如下：

```
FOREACH SLICE 3
1 | {{{001,002},{003,004}},{{005,006},{007,008}}}
```

FOREACH循环只生成一个迭代器切片。而且，正如assert所示，这与迭代数组完全相同。换句话说，将SLICE操作数设置为等于迭代数组的维度，而结果是明确定义的，这是没有用的。因此，使用这个示例迭代数组，SLICE操作数的有用范围是0..2。

接下来，使用SLICE 2来做测试：

```
do $body$
declare
  arr constant text[] not null := (select arr from t where k = 1);
  slice_iterator text[] not null := '{?}';
  n int not null := 0;
begin
  raise info 'FOREACH SLICE 2';
  n := 0;
  foreach slice_iterator slice 2 in array arr loop
    assert (array_ndims(slice_iterator) = 2), 'assert failed';
    n := n + 1;
    raise info '% | %', n::text, slice_iterator::text;
  end loop;
end;
$body$;
```

输出信息如下：

```
FOREACH SLICE 2
1 | {{001,002},{003,004}}
2 | {{005,006},{007,008}}
```

如assert所示，SLICE运算符的操作数决定了迭代器切片的维度。

再接下来，使用SLICE 1来做测试：

```
do $body$
declare
  arr constant text[] not null := (select arr from t where k = 1);
  slice_iterator text[] not null := '{?}';
  n int not null := 0;
begin
  raise info 'FOREACH SLICE 1';
  n := 0;
  foreach slice_iterator slice 1 in array arr loop
    assert (array_ndims(slice_iterator) = 1), 'assert failed';
    n := n + 1;
    raise info '% | %', n::text, slice_iterator::text;
  end loop;
end;
$body$;
```

输出信息如下：

```
FOREACH SLICE 1
1 | {001,002}
2 | {003,004}
3 | {005,006}
4 | {007,008}
```

最后，使用SLICE 0进行测试。请注意，现在，迭代器被声明为标量文本变量“var”：

```
do $body$
declare
  arr constant text[] not null := (select arr from t where k = 1);
  var text not null := '?';
  n int not null := 0;
begin
  raise info 'FOREACH SLICE 0';
  n := 0;
  foreach var slice 0 in array arr loop
    n := n + 1;
    raise info '% | %', n::text, var;
  end loop;
end;
$body$;
```

输出信息如下：

```
FOREACH SLICE 0
1 | 001
2 | 002
3 | 003
4 | 004
5 | 005
6 | 006
7 | 007
8 | 008
```

如下，在功能上等效于unnestt()：

```
do $body$
<<b>>declare
  arr constant text[] not null := (select arr from t where k = 1);
  var text not null := '?';
  n int not null := 0;
begin
  raise info 'unnest()';
  n := 0;
  for b.n, b.var in (
    with
      v1 as (
        select unnest(arr) as var),
      v2 as (
        select
          v1.var,
          row_number() over(order by v1.var) as n
        from v1)
    select v2.n, v2.var from v2)
  loop
    n := n + 1;
    raise info '% | %', n::text, var;
  end loop;
end b;
$body$;
```

输出信息如下：

```
FOREACH SLICE 0
1 | 001
2 | 002
3 | 003
4 | 004
5 | 005
6 | 006
7 | 007
8 | 008
```

## **使用FOREACH迭代DOMAIN值数组中的元素**

您需要了解一些特殊注意事项才能实现此场景。将FOREACH与DOMAIN数组一起使用，请参阅“[使用DOMAIN值数组](#_使用DOMAIN值数组)”进行了解。

```
Using a wrapper PL/pgSQL table function to expose the SLICE operand as a formal parameter
```

## **使用包装PL/pgSQL表函数将SLICE操作数公开为形式参数**

SLICE操作数必须是一个文字，这意味着只有两种方法可以将其参数化，而对于实际的应用程序代码来说，这两种方法都不令人满意。每一个都使用一个表函数，其输入是迭代数组和SLICE操作数的值，其输出是SETOF迭代器数组值。

* 第一种方法：在一个普通的静态定义函数中封装一些特定范围的SLICE操作数值，该函数使用CASE语句来选择具有所需SLICE操作数字的FOREACH循环。这是不令人满意的，因为您必须预先决定支持的SLICE操作数值的范围。
* 第二种方法：通过将代码封装在静态定义的函数中，克服了预先确定支持的SLICE操作数值范围的限制，该函数进而动态生成具有所需FOREACH循环和SLICE操作数值的函数，然后动态调用该函数。但它并不令人满意，主要是因为动态生成和执行带来的性能成本。
  但是，实际应用程序代码的需求规范不太可能需要SLICE操作数的多个，或者可能只需要几个特定值。因此，在绝大多数实际重要的用例中，您可以在需要的地方准确地编写所需的代码。

下面的代码使用第一种方法。它展示了一种常用又价值的PL/pgSQL编程技术：具有多态形式参数的用户定义函数和过程（在本例中为anyarray和anyelement）。示例中使用assert语句来确认如下预期关系：

* 迭代数组的维数
* SLICE操作数的值
* 迭代数组维度的长度
* 迭代数组和迭代器数组的基数
* 返回的迭代器值的数量。

以下是基本封装。它是硬编码的，用于处理0..4范围内的SLICE操作数的值。

对于SLICE为0的迭代是一个标量，SLICE为其他值的是数组。请记住，输入形式参数定义相同的两个函数，不能通过其返回值的数据类型进行重载区分。因此，SLICE 0的FOREACH循环的封装是一个只有一个输入形式参数的专用函数：迭代数组。对于SLICE操作数的其他值，FOREACH循环的封装是具有两个输入形式参数的第二个函数：迭代数组和SLICE操作数值。如下所示两个示例： 

第一个示例：

```
-- First overload
create function array_slices(arr in anyarray)
  returns table(ret anyelement)
  language plpgsql
as $body$
declare
  no_of_values int not null := 0;
begin
  -- "slice 0" means the same
  -- as omitting the "slice" clause.
  foreach ret slice 0 in array arr
  loop
    no_of_values := no_of_values + 1;
    return next;
  end loop;
  assert
    (no_of_values = cardinality(arr)),
  'array_slices 1st overload: no_of_values assert failed';
end;
$body$;
```

第二个示例：

```
-- Second overload
create function array_slices(arr in anyarray, slice_operand in int)
  returns table(ret anyarray)
  language plpgsql
as $body$
declare
  no_of_values int not null := 0;
  lengths_product int not null := 0;
begin
  case slice_operand
    when 1 then
      lengths_product := array_length(arr, 4);
      foreach ret slice 1 in array arr
      loop
        no_of_values := no_of_values + 1;
        assert
          (array_ndims(ret) = 1)               and
          (cardinality(ret) = lengths_product) ,
        'assert failed';
        return next;
      end loop;
      assert
        (no_of_values = cardinality(arr)/lengths_product),
      'array_slices 2nd overload: no_of_values assert #1 failed';
 
    when 2 then
      lengths_product := array_length(arr, 4) *
                         array_length(arr, 3);
      foreach ret slice 2 in array arr
      loop
        no_of_values := no_of_values + 1;
        assert
          (array_ndims(ret) = 2)               and
          (cardinality(ret) = lengths_product) ,
        'assert failed';
        return next;
      end loop;
      assert
        (no_of_values = cardinality(arr)/lengths_product),
      'array_slices 2nd overload: no_of_values assert #2 failed';
 
    when 3 then
      lengths_product := array_length(arr, 4) *
                         array_length(arr, 3) *
                         array_length(arr, 2);
      foreach ret slice 3 in array arr
      loop
        no_of_values := no_of_values + 1;
        assert
          (array_ndims(ret) = 3)               and
          (cardinality(ret) = lengths_product) ,
        'assert failed';
        return next;
      end loop;
      assert
        (no_of_values = cardinality(arr)/lengths_product),
      'array_slices 2nd overload: no_of_values assert #3 failed';
 
    when 4 then
      lengths_product := array_length(arr, 4) *
                         array_length(arr, 3) *
                         array_length(arr, 2) *
                         array_length(arr, 1);
      foreach ret slice 4 in array arr
      loop
        no_of_values := no_of_values + 1;
        assert
          (array_ndims(ret) = 4)               and
          (cardinality(ret) = lengths_product) ,
        'assert failed';
        return next;
      end loop;
      assert
        (no_of_values = cardinality(arr)/lengths_product),
      'array_slices 2nd overload: no_of_values assert #4 failed';
 
    else
      raise exception 'slice_operand > 4 not supported';
  end case;
end;
$body$;
```

可以看到，CASE的每个分支都是公式化地“生成”的，尽管是通过遵循可参数化的模式手动生成的。您可以将这些封装用于任何维度的迭代和数组。但是，您必须负责遵守SLICE操作数的值必须在可接受范围内的规则。否则，您将得到上面演示的类似错误： 

```
ERROR:  slice dimension % is out of the valid range 0..%
```

如下是一个测试封装模块，这个过程和生成要测试的迭代数组的函数都是维度为4的硬编码。

```
-- Exercise each of the meaningful calls to array_slices().
--
-- You cannot declare local variables as "anyelement" or "anyarray".
-- (The attempt causes a compilation error). It's obvious why.
-- It's the caller's responsibility to determine the
-- real type by using appropriate actual arguments.
-- "val" (scalar) and "slice" (array) are needed as FOREACH loop runners.
-- "in out" is used to avoid the nominal performance penalty
--  of extra copying brought by plain "out".
-- The caller has no interest in whatever values they have
-- on return from this procedure.
--
-- NOTE: while you _can_ declare a local variable as
-- "some_formal%type", you _cannot_ use that mechanism to declare
-- a scalar with the data type that defines an array when
-- all you have to anchor "%type" to is the array.
--
create procedure test_array_slices(
  -- The "real" formal.
  arr   in     anyarray,
 
  -- used as "local varables
  val   in out anyelement,
  slice in out anyarray)
  language plpgsql
as $body$
declare
  arr_ndims constant int := array_ndims(arr);
begin
  assert
    (arr_ndims = 4),
  'assert failed: test_array_slices() requires a 4-d array';
  raise info '%', array_dims(arr);
 
  declare
    len_1 constant int := array_length(arr, 1);
    len_2 constant int := array_length(arr, 2);
    len_3 constant int := array_length(arr, 3);
    len_4 constant int := array_length(arr, 4);
 
    expected_slice_cardinalities constant int[] not null :=
      array[
        len_4,
        len_4*len_3,
        len_4*len_3,
        len_4*len_3*len_2,
        len_4*len_3*len_2*len_1];
 
    slice_cardinality int not null := 0;
 
    -- val anyelement not null := '?';
    -- slice anyarray not null := '{?}';
  begin
    raise info ''; raise info 'slice_operand: %', 0;
 
    for val in (select array_slices(arr)) loop
      raise info '%', val::text;
    end loop;
 
    for slice_operand in 1..arr_ndims loop
      raise info ''; raise info 'slice_operand: %', slice_operand;
 
      for slice in (select array_slices(arr, slice_operand)) loop
        slice_cardinality := cardinality(slice);
        assert
          (array_ndims(slice) = slice_operand) ,
          (slice_cardinality = expected_slice_cardinalities[slice_operand]) ,
        'assert failed.';
        raise info '%', slice::text;
        if slice_operand = arr_ndims then
          assert (slice = arr), 'assert (slice = arr) failed';
        end if;
      end loop;
    end loop;
  end;
end;
$body$;
```

下面是一个生成四维数组的函数。请注意，“length”形式参数的实际参数必须是具有四个值的一维int[]数组。这些指定了沿输出数组的每个维度的长度。

```
create function four_d_array(lengths in int[])
  returns text[]
  language plpgsql
as $body$
declare
  lengths_ndims       constant int := array_ndims(lengths);
  lengths_cardinality constant int := cardinality(lengths);
begin
  assert
    (lengths_ndims = 1)       and
    (lengths_cardinality = 4) ,
  'assert failed: four_d_array() creates only a 4-d array.';
 
  declare
    -- Take the default for array_fill's optional 2nd formal:
    -- all lower bounds are 1.
    arr text[] not null := array_fill('00'::text, lengths);
  begin
    -- For readability of the results, define the created array's values so that,
    -- when scanned in row-major order, they are seen to be a dense series
    -- that increases in even steps, 001, 002, 003, and so on.
    declare
      n int not null := 0;
    begin
      for i1 in 1..lengths[1] loop
        for i2 in 1..lengths[2] loop
          for i3 in 1..lengths[3] loop
            for i4 in 1..lengths[4] loop
              n := n + 1;
              arr[i1][i2][i3][i4] := ltrim(to_char(n, '009'));
            end loop;
          end loop;
        end loop;
      end loop;
    end;
    assert
      (array_ndims(arr) = lengths_cardinality),
    'assert failed.';
 
    -- Sanity check. Include to demonstrate the useful
    -- terseness of the FOREACH loop.
    declare
      product int not null := 1;
      len int not null := 0;
    begin
      foreach len in array lengths loop
        product := product*len;
      end loop;
      assert
        (cardinality(arr) = product) ,
      'assert failed.';
    end;
    return arr;
  end;
end;
$body$;
 
下面是一个测试调用的示例：
do $body$
declare
  arr constant text[] not null := four_d_array('{2, 2, 2, 2}'::int[]);
  dummy_var text := '?';
  dummy_arr text[] := '{/}';
begin
  call test_array_slices(arr, dummy_var, dummy_arr);
end;
$body$;
```

返回信息如下（为方便阅读，手动去除了“INFO:”提示）：

```
[1:2][1:2][1:2][1:2]
 
slice_operand: 0
001
002
003
004
005
006
007
008
009
010
011
012
013
014
015
016
 
slice_operand: 1
{001,002}
{003,004}
{005,006}
{007,008}
{009,010}
{011,012}
{013,014}
{015,016}
 
slice_operand: 2
{{001,002},{003,004}}
{{005,006},{007,008}}
{{009,010},{011,012}}
{{013,014},{015,016}}
 
slice_operand: 3
{{{001,002},{003,004}},{{005,006},{007,008}}}
{{{009,010},{011,012}},{{013,014},{015,016}}}
 
slice_operand: 4
{{{{001,002},{003,004}},{{005,006},{007,008}}},{{{009,010},{011,012}},{{013,014},{015,016}}}}
```