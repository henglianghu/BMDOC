SQL语言部分描述了所有AiSQL SQL语句。

## **ABORT**

中止当前事务
语法：abort ::= ABORT [ TRANSACTION | WORK ]
描述：ABORT回滚当前事务并且导致由该事务所作的所有更新被丢弃。这个命令的行为与标准SQL命令ROLLBACK的行为一样。
参数：
WORK
TRANSACTION
可选关键词，它们没有效果。

## **ALTER DATABASE**

更改一个数据库
语法：alter_database ::= ALTER DATABASE name 
                   [ [ WITH ] alter_database_option [ ... ]
                     | RENAME TO name
                     | OWNER TO { new_owner
                                  | CURRENT_USER
                                  | SESSION_USER }
                     | SET run_time_parameter { TO | = } 
                       { value | DEFAULT }
                     | SET run_time_parameter FROM CURRENT
                     | RESET run_time_parameter
                     | RESET ALL ]

alter_database_option ::= ALLOW_CONNECTIONS allowconn
                          | CONNECTION LIMIT connlimit
                          | IS_TEMPLATE istemplate
描述：ALTER DATABASE 更改一个数据库。
参数：
name：指定要更改的数据库的名称
ALLOW_CONNECTIONS：指定false以禁止连接到此数据库。默认值为true，允许任何具有CREATEDB权限的用户克隆此数据库。
CONNECTION_LIMIT：指定可以与此数据库建立的并发连接数。默认值-1，允许无限制的并发连接。
IS_TEMPLATE：如果为true，则任何具有CREATEDB特权的用户都可以从这个数据库进行克隆。如果为false，则只有超级用户或者这个数据库的拥有者可以克隆它。

## **ALTER DEFAULT PRIVILEGES**

定义默认访问特权
语法：
alter_default_priv ::= ALTER DEFAULT PRIVILEGES 
                       [ FOR { ROLE | USER } role_name [ , ... ] ]  
                       [ IN SCHEMA schema_name [ , ... ] ] 
                       abbr_grant_or_revoke

abbr_grant_or_revoke ::= a_grant_table
                         | a_grant_seq
                         | a_grant_func
                         | a_grant_type
                         | a_grant_schema
                         | a_revoke_table
                         | a_revoke_seq
                         | a_revoke_func
                         | a_revoke_type
                         | a_revoke_schema

a_grant_table ::= GRANT { grant_table_priv [ , ... ]
                          | ALL [ PRIVILEGES ] } ON TABLES TO 
                  grantee_role [ , ... ]  [ WITH GRANT OPTION ]

a_grant_seq ::= GRANT { grant_seq_priv [ , ... ]
                        | ALL [ PRIVILEGES ] } ON SEQUENCES TO 
                grantee_role [ , ... ]  [ WITH GRANT OPTION ]

a_grant_func ::= GRANT { EXECUTE | ALL [ PRIVILEGES ] } ON 
                 { FUNCTIONS | ROUTINES } TO grantee_role [ , ... ]  
                 [ WITH GRANT OPTION ]

a_grant_type ::= GRANT { USAGE | ALL [ PRIVILEGES ] } ON TYPES TO 
                 grantee_role [ , ... ]  [ WITH GRANT OPTION ]

a_grant_schema ::= GRANT { USAGE | CREATE | ALL [ PRIVILEGES ] } ON 
                   SCHEMAS TO grantee_role [ , ... ]  
                   [ WITH GRANT OPTION ]

a_revoke_table ::= REVOKE [ GRANT OPTION FOR ] 
                   { grant_table_priv [ , ... ] | ALL [ PRIVILEGES ] } 
                   ON TABLES  FROM grantee_role [ , ... ] 
                   [ CASCADE | RESTRICT ]

a_revoke_seq ::= REVOKE [ GRANT OPTION FOR ] 
                 { grant_seq_priv [ , ... ] | ALL [ PRIVILEGES ] } ON 
                 SEQUENCES  FROM grantee_role [ , ... ] 
                 [ CASCADE | RESTRICT ]

a_revoke_func ::= REVOKE [ GRANT OPTION FOR ] 
                  { EXECUTE | ALL [ PRIVILEGES ] } ON 
                  { FUNCTIONS | ROUTINES }  FROM grantee_role 
                  [ , ... ] [ CASCADE | RESTRICT ]

a_revoke_type ::= REVOKE [ GRANT OPTION FOR ] 
                  { USAGE | ALL [ PRIVILEGES ] } ON TYPES  FROM 
                  grantee_role [ , ... ] [ CASCADE | RESTRICT ]

a_revoke_schema ::= REVOKE [ GRANT OPTION FOR ] 
                    { USAGE | CREATE | ALL [ PRIVILEGES ] } ON SCHEMAS 
                     FROM grantee_role [ , ... ] 
                    [ CASCADE | RESTRICT ]

grant_table_priv ::= SELECT
                     | INSERT
                     | UPDATE
                     | DELETE
                     | TRUNCATE
                     | REFERENCES
                     | TRIGGER

grant_seq_priv ::= USAGE | SELECT | UPDATE

grantee_role ::= [ GROUP ] role_name
                 | PUBLIC
                 | CURRENT_USER
                 | SESSION_USER

描述：ALTER DEFAULT PRIVILEGES允许你设置将被应用于未来要创建的对象的特权，它不会影响分配给已经存在的对象的特权。用户只能更改由自己或其成员角色创建的对象的默认权限。
示例：
示例1：向所有用户授予在架构营销中创建的所有表的SELECT权限。 

```
ALTER DEFAULT PRIVILEGES IN SCHEMA marketing GRANT SELECT ON TABLES TO PUBLIC;
```

示例2：撤消用户john对所有表的INSERT权限。

```
ALTER DEFAULT PRIVILEGES REVOKE INSERT ON TABLES FROM john;
```

## **ALTER DOMAIN**

更改一个域的定义
语法：
alter_domain_default ::= ALTER DOMAIN name 
                         { SET DEFAULT expression | DROP DEFAULT }

alter_domain_rename ::= ALTER DOMAIN name RENAME TO name
描述：ALTER DOMAIN更改一个现有域的定义。

参数：
SET DEFAULT/DROP DEFAULT:这些形式设置或者移除一个域的默认值。注意默认值只会应用到后续的 INSERT命令，它们不影响使用该域的已经存在于表中的行。
RENAME：重命名domain的名。
name：指定域的名称。如果域名称不存在，或域new_name已存在，则会引发错误。

示例：

```
CREATE DOMAIN idx as int DEFAULT 5 CHECK (VALUE > 0);
 
ALTER DOMAIN idx DROP DEFAULT;
 
ALTER DOMAIN idx RENAME TO idx_new;
 
DROP DOMAIN idx_new;
```

## **ALTER FOREIGN DATA WRAPPER**

更改一个外部数据包装器的定义
语法：
alter_foreign_data_wrapper ::= ALTER FOREIGN DATA WRAPPER fdw_name  
                               [ HANDLER handler_name | NO HANDLER ]  
                               [ VALIDATOR validator_name
                                 | NO VALIDATOR ]  
                               [ OPTIONS ( alter_fdw_options ) ]  
                               [ OWNER TO new_owner ]  
                               [ RENAME TO new_name ]
描述：ALTER FOREIGN DATA WRAPPER更改一个外部数据包装器的定义。
HANDLER：将调用handler_function来检索外部表的执行函数。这些功能是规划者和执行者所必需的。处理程序函数不接受任何参数，其返回类型应为fdw_handler。如果没有提供处理程序函数，则只能声明（而不能访问）使用包装器的外部表。
VALIDATOR：validator_function用于验证提供给外部数据包装器的选项，以及使用外部数据包装的外部服务器、用户映射和外部表。验证器函数有两个参数：一个文本数组（类型为text[]），其中包含要验证的选项，另一个是类型 oid，它将是包含该选项的系统目录的 OID，与选项关联的对象存储在该OID中。如果没有提供验证器函数（或未指定validator），则在创建时不会检查选项。
OPTIONS：这个子句为新的外部数据包装器指定选项。允许的选项名称和值与每一个外部数据包装器有关，并且它们会被该外部数据包装器的验证器函数验证。 选项名称必须唯一。

示例：

```
ALTER FOREIGN DATA WRAPPER file NO HANDLER ;
ALTER FOREIGN DATA WRAPPER file HANDLER file_fdw_handler;
ALTER FOREIGN DATA WRAPPER file NO VALIDATOR;
ALTER FOREIGN DATA WRAPPER file OPTIONS(ADD new '1');
```

## **ALTER FOREIGN TABLE**

更改一个外部表的定义
语法：
alter_foreign_table ::= ALTER FOREIGN TABLE [ IF EXISTS ] table_name 
                        alter_foreign_table_action [ , ... ]

描述：
ALTER FOREIGN TABLE更改一个现有外部表的定义。
参数：
ADD COLUMN：可用于将新列添加到外部表中，对底层存储没有影响，ADD COLUMN操作只是指示可以通过外部表访问新添加的列。
DROP COLUMN：这种形式从一个外部表删掉一列。DROP COLUMN子句可用于从外部表中删除列。可以指定CASCADE或RESTRICT。如果在该表外部有任何东西依赖于该列，你将需要写上CASCADE，典型的示例就是视图。
Change owner：这种形式把该外部表的拥有者改成指定的用户。
Options：OPTIONS子句可用于指定外部表的新选项。ADD、SET和DROP指定要执行的操作。如果没有明确指定操作，则假定ADD。
Rename：RENAME TO子句可用于将外部表重命名为table_name。
示例：
示例1：增加一个新列：

```
ALTER FOREIGN TABLE my_table ADD COLUMN new_col int;
```

示例2：更改options：

```
ALTER FOREIGN TABLE my_table OPTIONS (ADD newopt1 'value1', DROP oldopt1 'value2', SET oldopt2 'value3');
```

## **ALTER FUNCTION**

更改一个函数的定义
语法：
alter_function ::= ALTER FUNCTION subprogram_name ( 
                   [ subprogram_signature ] )  
                   { special_fn_and_proc_attribute
                     | { alterable_fn_and_proc_attribute
                         | alterable_fn_only_attribute } [ ... ] 
                       [ RESTRICT ] }

subprogram_signature ::= arg_decl [ , ... ]

arg_decl ::= [ formal_arg ] [ arg_mode ] arg_type

special_fn_and_proc_attribute ::= RENAME TO subprogram_name
                                  | OWNER TO 
                                    { role_name
                                      | CURRENT_ROLE
                                      | CURRENT_USER
                                      | SESSION_USER }
                                  | SET SCHEMA schema_name
                                  | [ NO ] DEPENDS ON EXTENSION 
                                    extension_name

alterable_fn_and_proc_attribute ::= SET run_time_parameter 
                                    { TO value
                                      | = value
                                      | FROM CURRENT }
                                    | RESET run_time_parameter
                                    | RESET ALL
                                    | [ EXTERNAL ] SECURITY 
                                      { INVOKER | DEFINER }

alterable_fn_only_attribute ::= volatility
                                | on_null_input
                                | PARALLEL parallel_mode
                                | [ NOT ] LEAKPROOF
                                | COST int_literal
                                | ROWS int_literal

volatility ::= IMMUTABLE | STABLE | VOLATILE

on_null_input ::= CALLED ON NULL INPUT
                  | RETURNS NULL ON NULL INPUT
                  | STRICT

parallel_mode ::= UNSAFE | RESTRICTED | SAFE

描述：ALTER FUNCTION更改一个函数的定义。

参数：
subprogram_name：一个现有函数的名称（可以被模式限定）。如果没有指定参数列表， 则该名称必须在它的模式中唯一。
argmode：一个参数的模式：IN、OUT、 INOUT或者VARIADIC。如果被忽略，默认为 IN。注意ALTER FUNCTION 并不真正关心OUT参数，因为在决定函数的身份时只需要输入参数。因此列出IN、INOUT以及 VARIADIC参数即可。
formal_arg：一个参数的名称。注意ALTER FUNCTION 并不真正参数名称，因为在确定函数的身份时只需要参数的数据类型即可。
argtype：该函数的参数（如果有）的数据类型（可以被模式限定）。

role_name：该函数的新拥有者。
schema_name：该函数的新模式。
extension_name：该函数所依赖的扩展名。
on_null_input：CALLED ON NULL INPUT将该函数改为在某些 或者全部参数为空值时可以被调用。 RETURNS NULL ON NULL INPUT或者 STRICT将该函数改为只要任一参数为空值就不被调用而 是自动假定一个空值结果。
volatility：把该函数的稳定性更改为指定的设置。详见 CREATE FUNCTION。
[ EXTERNAL ] SECURITY INVOKER | [ EXTERNAL ] SECURITY DEFINER：更改该函数是否为一个安全性定义者。关键词EXTERNAL 是为了符合 SQL，它会被忽略。
PARALLEL：决定该函数对于并行是否安全。
LEAKPROOF：更改该函数是否被认为是防泄漏的。
COST int_literal：更改该函数的估计执行代价。
ROWS int_literal：更改一个集合返回函数的估计行数。
run_time_parameter：
value
当该函数被调用时，要对一个配置参数做出增加或者更改的赋值。如果 value是DEFAULT 或者使用等价的RESET，该函数本地的设置将会被移除，这样该函数会使用其环境中存在的值执行。使用RESET ALL可以清除所有函数本地的设置。 SET FROM CURRENT把ALTER FUNCTION 执行时该参数的当前值保存为进入该函数时要应用的值。
RESTRICT：为了符合 SQL 标准存在，被忽略。


示例：

```
drop schema if exists s3 cascade;
drop schema if exists s4 cascade;
create schema s3;
 
create function s3.f(i in int)
  returns text
  security definer
  volatile
  language plpgsql
as $body$
begin
  return 'Result: '||(i*2)::text;
end;
$body$;
 
select s3.f(17) as "s3.f(17)";
```

返回信息如下：

```
  s3.f(17)
------------
 Result: 34
```

现在假设您意识到security definer是错误的选择，您想将其标记为不可变的，并且您想设置statement_timeout属性。同样假设：您想调用函数g()，而不是f()；并且您希望它在模式s4中，而不是在模式s3中。因此，必须使用三个ALTER语句才能执行此操作，如下所示：

```
alter function s3.f(int)
  security invoker
  immutable
  set statement_timeout = 1;
```

执行完成之后，可以通过检查函数的元数据来检查是否成功，可以执行如下脚本来进行相关检查：

```
select
  proname::text                                as name,
  pronamespace::regnamespace::text             as schema,
  case
    when prosecdef then 'definer'
    else 'invoker'
  end                                          as security,
  case
    when provolatile = 'v' then 'volatile'
    when provolatile = 's' then 'stable'
    when provolatile = 'i' then 'immutable'
  end                                          as volatility,
  proconfig                                    as settings
from pg_proc
where
  proname::text in ('f', 'g');
```

返回信息如下：

```
 name | schema | security | volatility |       settings
------+--------+----------+------------+-----------------------
 f    | s3     | invoker  | immutable  | {statement_timeout=1}
```

接下来，重命名此函数：

```
alter function s3.f(int) rename to g;
```

再次执行上述检查脚本，进行检查，返回信息如下：

```
 name | schema | security | volatility |       settings        
------+--------+----------+------------+-----------------------
 g    | s3     | invoker  | immutable  | {statement_timeout=1}
```

最后，修改模式：

```
create schema s4;
alter function s3.g(int) set schema s4;
```

继续执行上述检查脚本，进行检查，返回信息如下：

```
 name | schema | security | volatility |       settings        
------+--------+----------+------------+-----------------------
 g    | s4     | invoker  | immutable  | {statement_timeout=1}
```

## **ALTER GROUP**

更改角色名称或者成员关系
语法：
alter_group ::= ALTER GROUP role_specification { ADD | DROP } USER 
                role_name [ , ... ]

role_specification ::= role_name | CURRENT_USER | SESSION_USER

alter_group_rename ::= ALTER GROUP role_name RENAME TO new_role_name


描述：使用ALTER GROUP语句可以更改组（角色）的属性。添加此项是为了与Postgres兼容。不鼓励使用它。ALTER ROLE是更改角色属性的首选方式。有关更多详细信息，请参阅[ALTER ROLE](#_ALTER ROLE)
ALTER GROUP可用于在组中添加或删除角色，请改用GRANT或REVOKE，它还可以用于重命名角色。

## **ALTER POLICY**

更改一条行级安全性策略的定义
语法：
alter_policy ::= ALTER POLICY name ON table_name 
                 [ TO { role_name
                        | PUBLIC
                        | CURRENT_USER
                        | SESSION_USER } [ , ... ] ]  
                 [ USING ( using_expression ) ] 
                 [ WITH CHECK ( check_expression ) ]

alter_policy_rename ::= ALTER POLICY name ON table_name RENAME TO 
                        new_name

描述：ALTER POLICY更改一条现有行级安全性策略的定义。
参数：
name：要更改的现有策略的名称。
table_name：该策略所在的表的名称（可以被模式限定）。
new_name：该策略的新名称。
role_name：该策略适用的角色。可以一次指定多个角色。要把该策略应用于所有角色，可使用PUBLIC。
using_expression：该策略的USING表达式。
check_expression：该策略的WITH CHECK表达式。

## **ALTER PROCEDURE**

更改过程的定义
语法：
alter_procedure ::= ALTER PROCEDURE subprogram_name ( 
                    [ subprogram_signature ] )  
                    { special_fn_and_proc_attribute
                      | alterable_fn_and_proc_attribute [ ... ] 
                        [ RESTRICT ] }

subprogram_signature ::= arg_decl [ , ... ]

arg_decl ::= [ formal_arg ] [ arg_mode ] arg_type

special_fn_and_proc_attribute ::= RENAME TO subprogram_name
                                  | OWNER TO 
                                    { role_name
                                      | CURRENT_ROLE
                                      | CURRENT_USER
                                      | SESSION_USER }
                                  | SET SCHEMA schema_name
                                  | [ NO ] DEPENDS ON EXTENSION 
                                    extension_name

alterable_fn_and_proc_attribute ::= SET run_time_parameter 
                                    { TO value
                                      | = value
                                      | FROM CURRENT }
                                    | RESET run_time_parameter
                                    | RESET ALL
                                    | [ EXTERNAL ] SECURITY 
                                      { INVOKER | DEFINER }

描述：ALTER PROCEDURE更改一个过程的定义。

示例：

```
ALTER PROCEDURE insert_data(integer, integer) OWNER TO joe;
```

## **ALTER ROLE**

更改一个数据库角色
语法：
alter_role ::= ALTER ROLE role_specification 
               [ [ WITH ] alter_role_option [ , ... ] ]

alter_role_option ::= SUPERUSER
                      | NOSUPERUSER
                      | CREATEDB
                      | NOCREATEDB
                      | CREATEROLE
                      | NOCREATEROLE
                      | INHERIT
                      | NOINHERIT
                      | LOGIN
                      | NOLOGIN
                      | CONNECTION LIMIT connlimit
                      | [ ENCRYPTED ] PASSWORD  ' password ' 
                      | PASSWORD NULL
                      | VALID UNTIL  ' timestamp ' 

role_specification ::= role_name | CURRENT_USER | SESSION_USER

alter_role_rename ::= ALTER ROLE role_name RENAME TO new_role_name

alter_role_config ::= ALTER ROLE { role_specification | ALL } 
                      [ IN DATABASE database_name ] config_setting

config_setting ::= SET config_param { TO | = } 
                   { config_value | DEFAULT }
                   | SET config_param FROM CURRENT
                   | RESET config_param
                   | RESET ALL

描述：使用ALTER ROLE语句可以更改角色（用户或组）的属性。
超级用户可以更改任何角色的属性。具有CREATE ROLE权限的角色可以更改任何非超级用户角色的属性。其他角色只能更改自己的密码。

参数：
role_specification：指定要更改其属性的角色、当前用户或当前会话用户的名称。
SUPERUSER，NOSUPERUSER：确定新角色是否为“超级用户”。超级用户可以覆盖所有访问限制，应谨慎使用。只有具有SUPERUSER权限的角色才能创建其他SUPERUSER角色。如果未指定，则默认为NOSUPERUSER。
CREATEDB，NOCREATEDB：确定新角色是否可以创建数据库。默认为NOCREATEDB。
CREATEROLE，NOCREATEROLE：确定新角色是否可以创建其他角色。默认值为NOCREATEROLE。 
INHERIT，NOINHERIT：确定新角色是否继承其所属角色的特权。如果没有INHERIT，另一个角色的成员资格只能授予该角色设置角色的能力。其他角色的权限只有在完成后才可用。如果未指定，则默认为INHERIT。
LOGIN，NOLOGIN：确定是否允许新角色登录。只有具有登录权限的角色才能在客户端连接期间使用。具有LOGIN的角色可以被视为用户。如果未指定，则默认为NOLOGIN。请注意，如果使用CREATE USER语句而不是CREATE ROLE，则默认值为LOGIN。
CONNECTION LIMIT：指定角色可以进行的并发连接数。默认值为-1，表示不受限制。这仅适用于可以登录的角色。
[ENCRYPTED] PASSWORD：设置新角色的密码。这只适用于可以登录的角色。如果未指定密码，则密码将设置为null，并且该用户的密码身份验证将始终失败。请注意，密码始终加密存储在系统目录中，可选关键字encrypted仅用于与PostgreSQL兼容。 
VALID UNTIL：设置角色密码不再有效的日期和时间。如果省略此子句，密码将始终有效。 
config_param和config_value：是正在设置的配置参数的名称和值。 
ALTER ROLE ROLE_name RENAME TO：可用于更改角色的名称。请注意，当前会话角色无法重命名。如果密码是MD5加密的，则重命名角色会清除其密码。 
ALTER ROLE SET | RESET config_param：用于更改配置变量的角色会话默认值，可以用于所有数据库，也可以在指定IN DATABASE子句时仅用于命名数据库中的会话。如果指定了ALL而不是角色名称，则会更改所有角色的设置。
示例：

示例1：更改角色的密码。

```
ALTER ROLE John WITH PASSWORD 'new_password';
```

示例2：重命名一个角色。

```
ALTER ROLE John RENAME TO Jane;
```

示例3：更改角色的default_transaction_isolation会话参数。 

```
ALTER ROLE Jane SET default_transaction_isolation='serializable';
```

## **ALTER SCHEMA**

更改一个模式的定义
语法：
alter_schema ::= ALTER SCHEMA schema_name 
                 { RENAME TO new_name
                   | OWNER TO { new_owner
                                | CURRENT_USER
                                | SESSION_USER } }

描述：ALTER SCHEMA更改一个模式的定义。
注意以下几点：
为了可以使用ALTER SCHEMA，您需要是模式的所有者。
重命名模式需要具有数据库的CREATE权限。
如果要更改所有者，您还必须是新所有者角色的直接或间接成员，并且您需要拥有数据库的CREATE权限。
参数：
schema_name：指定模式的名称（schema_name）。如果当前数据库中不存在具有该名称的模式，则会引发错误。
RENAME TO new_name：重命名模式名称。模式名称不得以pg_开头。尝试使用这样的名称创建模式，或将现有模式重命名为具有这样的名称，会导致错误。
OWNER TO (new_owner | CURRENT_USER | SESSION_USER)：更改模式的所有者。
new_owner：模式的新的所有者。
CURRENT_USER：当前执行上下文的用户名。
SESSION_USER：当前会话的用户名。

示例：
创建一个新的模式：

```
CREATE SCHEMA schema22;
```

示例1：重命名此模式的名称：

```
ALTER SCHEMA schema22 RENAME TO schema25;
```

示例2：更改模式的所有者：

```
ALTER SCHEMA schema25 OWNER TO postgres;
```

## **ALTER SEQUENCE**

更改一个序列发生器的定义
语法：
alter_sequence ::= ALTER SEQUENCE [ IF EXISTS ] sequence_name 
                   alter_sequence_options

alter_sequence_options ::= [ AS seq_data_type ]  
                           [ INCREMENT [ BY ] int_literal ]  
                           [ MINVALUE int_literal | NO MINVALUE ]  
                           [ MAXVALUE int_literal | NO MAXVALUE ]  
                           [ START [ WITH ] int_literal ]  
                           [ RESTART [ [ WITH ] int_literal ] ]  
                           [ CACHE int_literal ]  [ [ NO ] CYCLE ]  
                           [ OWNED BY table_name . column_name
                             | NONE ]

描述：ALTER SEQUENCE更改一个现有序列发生器的参数。


参数：
sequence_name [ IF EXISTS ]：指定序列的名称（sequence_name）。如果当前模式中不存在具有该名称的序列，并且未指定if exists，则会引发错误。
AS seq_data_type：改变序列的数据类型。如果以前的值超出了新类型允许的范围，这将自动更改序列的最小值和最大值，有效类型是smallint、integer 和bigint。
INCREMENT BY int_literal：指定序列中连续值之间的差值。默认值为1 
MINVALUE int_literal | NO MINVALUE：指定序列中允许的最小值。如果达到这个值（在负增量的序列中），nextval()将返回一个错误。如果未指定MINVALUE，则将使用默认值。默认值为1。
MAXVALUE int_literal | NO MAXVALUE：指定序列中允许的最大值。如果达到该值，nextval()将返回一个错误。如果未指定MAXVALUE，则将使用默认值。默认值为  2⁶³-1。
START WITH int_literal：指定序列中的第一个值。start不能小于minvalue。默认值为1。
RESTART [ [ WITH ] int_literal ] ]：更改序列的当前值。如果未指定值，则当前值将设置为创建或更改序列时使用START[with]指定的最后一个值。 
CACHE int_literal：指定要在客户端中缓存序列中的数字。默认值为1。
当TServer bsql_sequence_cache_minval配置标志未显式关闭（设置为0）时，将使用该标志和缓存子句的最大值。
[ NO ] CYCLE：如果指定了CYCLE，则一旦达到minvalue或maxvalue，序列就会回卷。如果达到了maxvalue，则minvalue将是序列中的下一个数字。如果达到minvalue（对于递减序列），maxvalue将是序列中的下一个数字。“无循环”是默认设置。
OWNED BY table_name.table_column | NONE：它将序列的所有权赋予指定的列（如果有的话）。这意味着，如果删除列（或其所属的表），则序列将自动删除。如果指定了NONE，则将删除以前的所有权。
示例：

创建一个序列。 

```
CREATE SEQUENCE s;
```

示例1：修改增量值。 

```
ALTER SEQUENCE s INCREMENT BY 5;
```

示例2：修改起始值。

```
ALTER SEQUENCE s RESTART WITH 2;
```

## **ALTER SERVER**

更改一个外部服务器的定义
语法：
alter_server ::= ALTER SERVER server_name [ VERSION server_version ]  
                 [ OPTIONS ( alter_fdw_options ) ] 
                 [ OWNER TO new_owner ]

描述：ALTER SERVER更改一个外部服务器的定义。

参数：
server_version：新的服务器版本。
OPTIONS ( alter_fdw_options )：更改该服务器的选项。ADD、SET和 DROP指定要执行的动作。如果没有显式地指定操作， 将会假定为ADD。选项名称必须唯一，名称和值也会 使用该服务器的外部数据包装器库进行验证。
new_owner：该外部服务器的新拥有者的用户名。

示例：
改变server的版本为2.0，设置opt1 为true，删除opt2：

```
ALTER SERVER my_server SERVER VERSION '2.0' OPTIONS (SET opt1 'true', DROP opt2);
```

## **ALTER TABLE**

更改一个表的定义
语法：alter_table ::= ALTER TABLE table_expr alter_table_action [ , ... ]

alter_table_action ::= ADD [ COLUMN ] column_name data_type 
                       [ alter_column_constraint [ ... ] ]
                       | RENAME TO table_name
                       | DROP [ COLUMN ] column_name 
                         [ RESTRICT | CASCADE ]
                       | ADD alter_table_constraint
                       | DROP CONSTRAINT constraint_name 
                         [ RESTRICT | CASCADE ]
                       | RENAME [ COLUMN ] column_name TO column_name
                       | RENAME CONSTRAINT constraint_name TO 
                         constraint_name
                       | DISABLE ROW LEVEL SECURITY
                       | ENABLE ROW LEVEL SECURITY
                       | FORCE ROW LEVEL SECURITY
                       | NO FORCE ROW LEVEL SECURITY

alter_table_constraint ::= [ CONSTRAINT constraint_name ]  
                           { CHECK ( expression )
                             | UNIQUE ( column_names ) 
                               index_parameters
                             | FOREIGN KEY ( column_names ) 
                               references_clause }  
                           [ DEFERRABLE | NOT DEFERRABLE ] 
                           [ INITIALLY DEFERRED
                             | INITIALLY IMMEDIATE ]

alter_column_constraint ::= [ CONSTRAINT constraint_name ]  
                            { NOT NULL
                              | NULL
                              | CHECK ( expression )
                              | DEFAULT expression
                              | UNIQUE index_parameters
                              | references_clause } 
                            [ DEFERRABLE | NOT DEFERRABLE ] 
                            [ INITIALLY DEFERRED
                              | INITIALLY IMMEDIATE ]

table_expr ::= [ ONLY ] table_name [ * ]
描述：ALTER TABLE更改一个现有表的定义。
参数：
alter_table_action：
指定以下操作的一种。 
ADD [ COLUMN ] column_name data_type constraint：向该表增加一个新列，使用与 CREATE TABLE相同的语法。
RENAME TO table_name：重命名指定的表。
DROP [COLUMN] COLUMN_name [RESTRICT|CASCADE]：从表中删除列。
RESTRICT--仅删除指定的列。
CASCADE—删除指定的列和任何依赖对象。

例如：

```
drop table if exists children cascade;
drop table if exists parents  cascade;
 
-- The column "b" models a (natural) business unique key.
create table parents(
  k int primary key,
  b int not null,
  v text not null,
  constraint parents_b_unq unique(b));
 
create table children(
  parents_b  int  not null,
  k          int  not null,
  v          text not null,
 
  constraint children_pk primary key(parents_b, k),
 
  constraint children_fk foreign key(parents_b)
    references parents(b)
    match full
    on delete cascade
    on update restrict);
 
insert into parents(k, b, v) values (1, 10, 'dog'), (2, 20, 'cat'), (3, 30, 'frog');
 
insert into children(parents_b, k, v) values
  (10, 1, 'dog-child-a'),
  (10, 2, 'dog-child-b'),
  (10, 3, 'dog-child-c'),
  (20, 1, 'cat-child-a'),
  (20, 2, 'cat-child-b'),
  (20, 3, 'cat-child-c'),
  (30, 1, 'frog-child-a'),
  (30, 2, 'frog-child-b'),
  (30, 3, 'frog-child-c');
 
select p.v as "p.v", c.v as "c.v"
from parents p inner join children c on c.parents_b = p.b
order by p.b, c.k;
```

返回信息如下：

```
 
 p.v  |     c.v
------+--------------
 dog  | dog-child-a
 dog  | dog-child-b
 dog  | dog-child-c
 cat  | cat-child-a
 cat  | cat-child-b
 cat  | cat-child-c
 frog | frog-child-a
 frog | frog-child-b
 frog | frog-child-c
```

\d children 元命令显示它有一个外键，该外键是parents表中列b上的依赖对象：
返回信息如下：

```
Indexes:
    "children_pk" PRIMARY KEY, lsm (parents_b HASH, k ASC)
Foreign-key constraints:
    "children_fk" FOREIGN KEY (parents_b) REFERENCES parents(b) MATCH FULL ON UPDATE RESTRICT ON DELETE CASCADE
```

这是一个人为的示例。除了定义父表主键约束的列之外，将外键约束作为任何对象都是不常见的做法（通常也是不好的做法）。但这样做有时是有正当理由的。
现在尝试删除列parents.b:

```
do $body$
declare
  message  text not null := '';
  detail   text not null := '';
begin
  -- Causes error 'cos "cascade" is required.
  alter table parents drop column b;
  assert false, 'Should not get here';
exception
  -- Error 2BP01
  when dependent_objects_still_exist then
    get stacked diagnostics
      message  = message_text,
      detail   = pg_exception_detail;
    assert message = 'cannot drop column b of table parents because other objects depend on it',      'Bad message';
    assert detail  = 'constraint children_fk on table children depends on column b of table parents', 'Bad detail';
end;
$body$;
```

它完成时没有出现错误，表明空的alter table parents drop column b（没有cascade）失败，并导致代码显示的消息和提示。现在用cascade重复该尝试，并观察结果： 

```
alter table parents drop column b cascade;
```

执行成功，现在\d children显示外键约束children _fk已被传递性删除。
ADD alter_table_constraint：将指定的约束添加到表中。
DROP CONSTRAINT constraint_name [ RESTRICT | CASCADE ]:从表中删除约束。
RESTRICT—仅删除指定的约束。
CASCADE—删除指定的约束和任何依赖对象。 
RENAME [ COLUMN ] column_name TO column_name：将列重命名为指定的名称。

RENAME CONSTRAINT constraint_name TO constraint_name：将约束重命名为指定的名称。
例如：
创建带有约束的表并重命名约束：

```
CREATE TABLE test(id BIGSERIAL PRIMARY KEY, a TEXT);
ALTER TABLE test ADD constraint vague_name unique (a);
ALTER TABLE test RENAME CONSTRAINT vague_name TO unique_a_constraint;
```

ENABLE / DISABLE ROW LEVEL SECURITY：这将启用或禁用表的行级安全性。如果启用了该表并且不存在任何策略，则应用默认的拒绝策略。如果禁用，则不会应用该表的现有策略，并将被忽略。有关如何创建行级安全策略的详细信息，请参阅[CREATE POLICY](#_CREATE POLICY) 
FORCE / NO FORCE ROW LEVEL SECURITY：当用户是表的所有者时，这将控制表的行安全策略的应用。如果启用，则当用户是表所有者时，将应用行级安全策略。如果禁用（默认设置），则当用户是表所有者时，不会应用行级安全性。有关如何创建行级安全策略的详细信息，请参阅[CREATE POLICY](#_CREATE POLICY)
Constraints：指定表或列的约束。
CONSTRAINT constraint_name：指定约束的名称。 
Foreign key：FOREIGN KEY和REFERENCES指定列集只能包含引用表的引用列中存在的值。它用于强制数据的引用完整性。
Unique：这强制要求UNIQUE约束中指定的列集在表中是唯一的，也就是说，对于UNIQUE限制中指定的一组列，没有两行可以具有相同的值。
Check：这用于强制指定表中的数据满足CHECK子句中指定的要求。
Default：这用于指定列的默认值。如果INSERT语句没有为列指定值，则使用默认值。如果没有为列指定默认值，则默认值为NULL
Deferrable constraints：可以使用DEFERRABLE子句延迟约束。目前，AiSQL中只能延迟外键约束。语句中的每一行之后都会检查一个不可延迟的约束。在可延迟约束的情况下，约束的检查可以推迟到事务结束。
标记为INITIALLY IMMEDIATE的约束将在语句中的每一行之后进行检查。
标记为INITIALLY DEFERRED的约束将在事务结束时进行检查。 

## **ALTER USER**

更改一个数据库角色
语法：
alter_user ::= ALTER USER role_specification 
               [ [ WITH ] alter_role_option [ , ... ] ]

alter_role_option ::= SUPERUSER
                      | NOSUPERUSER
                      | CREATEDB
                      | NOCREATEDB
                      | CREATEROLE
                      | NOCREATEROLE
                      | INHERIT
                      | NOINHERIT
                      | LOGIN
                      | NOLOGIN
                      | CONNECTION LIMIT connlimit
                      | [ ENCRYPTED ] PASSWORD  ' password ' 
                      | PASSWORD NULL
                      | VALID UNTIL  ' timestamp ' 

role_specification ::= role_name | CURRENT_USER | SESSION_USER

alter_user_rename ::= ALTER USER role_name RENAME TO new_role_name

alter_user_config ::= ALTER USER { role_specification | ALL } 
                      [ IN DATABASE database_name ] config_setting

config_setting ::= SET config_param { TO | = } 
                   { config_value | DEFAULT }
                   | SET config_param FROM CURRENT
                   | RESET config_param
                   | RESET ALL

描述：使用ALTER USER语句可以更改角色。ALTER USER是ALTER ROLE的别名，用于更改角色。
有关更多详细信息，请参阅[ALTER ROLE](#_ALTER ROLE)

## **ANALYZE**

收集有关一个数据库的统计信息
语法：
analyze ::= ANALYZE [ VERBOSE ] [ table_and_columns [ , ... ] ]
table_and_columns ::= table_name [ ( column_name [ , ... ] ) ]
描述：ANALYZE收集一个数据库中的表的内容的统计信息。
参数：
VERBOSE：允许显示进度消息。
table_name：要分析的一个指定表的名称（可以是模式限定的），默认分析当前数据库中的所有常规表。
column_name：要分析的一个指定列的名称。默认是所有列。
示例：
示例1：分析一个表

```
ANALYZE some_table;
```

示例2：分析指定的列

```
ANALYZE some_table(col1, col3);
```

示例3：分析多个表

```
ANALYZE VERBOSE some_table, other_table;
```

返回信息如下：

```
INFO:  analyzing "public.some_table"
INFO:  "some_table": scanned, 3 rows in sample, 3 estimated total rows
INFO:  analyzing "public.other_table"
INFO:  "other_table": scanned, 3 rows in sample, 3 estimated total rows
```

ANALYZE
示例4：分析查询计划，此示例演示统计信息如何影响优化器。 
创建表，并插入一些数据

```
CREATE TABLE test(a int primary key, b int);
INSERT INTO test VALUES (1, 1), (2, 2), (3, 3);
```

在没有统计信息的情况下，优化器使用硬编码的默认值，例如表中的行数为1000。

```
EXPLAIN select * from test where b = 1;
```

返回信息如下：

```
                       QUERY PLAN
---------------------------------------------------------
 Seq Scan on test  (cost=0.00..102.50 rows=1000 width=8)
   Filter: (b = 1)
```

现在运行ANALYZE命令来收集统计信息。 

```
ANALYZE test;
EXPLAIN select * from test where b = 1;
```

返回信息如下：

```
                     QUERY PLAN
----------------------------------------------------
 Seq Scan on test  (cost=0.00..0.31 rows=3 width=8)
   Filter: (b = 1)
```

一旦优化器对表中的数据有了更好的了解，它就能够创建性能更好的查询计划。 

## **BEGIN**

开始一个事务块
语法：
begin ::= BEGIN [ TRANSACTION | WORK ] [ transaction_mode [ ... ] ]

描述：BEGIN开始一个事务块，也就是说所有 BEGIN命令之后的所有语句将被在一个事务中执行，直到给出一个显式的COMMIT或者ROLLBACK。
参数：
WORK | TRANSACTION：可选的关键词。它们没有效果。
transaction_mode：支持Serializable、Snapshot和Read Committed Isolation，分别使用Serializable、REPEATABLE Read和Read Committed的PostgreSQL隔离级别语法。PostgreSQL的READ UNCOMITTED也映射到READ Committed Isolation。 
只有当TServer标志bm_enable_read_committed_isolation设置为true时，才支持读取提交隔离。默认情况下，此标志为false，在这种情况下，AiSQL事务层的Read Committed隔离级别会回落到更严格的快照隔离（在这种情况中，BSQL的Read Committed和Read UNCOMITTED也依次使用快照隔离）。
示例：
创建表：

```
CREATE TABLE sample(k1 int, k2 int, v1 int, v2 text, PRIMARY KEY (k1, k2));
```

开启一个事务，并插入一些数据：

```
BEGIN TRANSACTION; SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
INSERT INTO sample(k1, k2, v1, v2) VALUES (1, 2.0, 3, 'a'), (1, 3.0, 4, 'b');
```

使用bsqlsh启动一个新的shell，然后开始另一个事务来插入更多的行。

```
BEGIN TRANSACTION; SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
INSERT INTO sample(k1, k2, v1, v2) VALUES (2, 2.0, 3, 'a'), (2, 3.0, 4, 'b');
```

在每个shell中，检查是否只有当前事务中的行可见。
在第一个shell中，执行：

```
SELECT * FROM sample; 
```

返回信息如下：

```
 k1 | k2 | v1 | v2 
----+----+----+----
  1 |  2 |  3 | a
  1 |  3 |  4 | b
```

在第二个shell中，执行：

```
SELECT * FROM sample; 
```

返回信息如下：

```
 k1 | k2 | v1 | v2 
----+----+----+----
  2 |  2 |  3 | a
  2 |  3 |  4 | b
```

提交第一个事务并中止第二个事务。
在第一个shell中，执行：

```
COMMIT TRANSACTION; 
```

中止当前事务（从第二个shell）。

```
ABORT TRANSACTION;
```

在每个shell中，检查是否只有来自提交事务的行可见。 

在第一个shell中，执行：

```
SELECT * FROM sample; 
```

返回信息如下：

```
 k1 | k2 | v1 | v2 
----+----+----+----
  1 |  2 |  3 | a
  1 |  3 |  4 | b
```

在第二个shell中，执行：

```
SELECT * FROM sample; 
```

返回信息如下：

```
 k1 | k2 | v1 | v2 
----+----+----+----
  1 |  2 |  3 | a
  1 |  3 |  4 | b
```

## **CALL**

调用一个过程
语法：
call_procedure ::= CALL subprogram_name ( [ actual_arg [ , ... ] ] )
actual_arg ::= [ formal_arg => ] expression
formal_arg ::= name

描述：CALL执行一个存储过程。如果过程有任何输出参数，那么将返回一个结果行，其中包含这些参数的值。调用方必须同时具有要调用过程所在架构的使用权限和执行权限。如果调用方缺乏所需的使用权限，则会导致以下错误： 

42501: permission denied for schema %"
如果调用方具有所需的使用权限，但缺少所需的执行权限，则会导致以下错误： 
42501: permission denied for procedure %

注意：
如果CALL在事务块中执行，则它无法执行事务控制语句。该尝试导致此运行时错误：2D000: invalid transaction termination
当调用CALL时将autocommit设置为on时，允许使用事务控制语句——在这种情况下，过程在自己的事务中执行。 
示例：
创建一个简单的存储过程：

```
set client_min_messages = warning;
drop procedure if exists p(text, int) cascade;
create procedure p(
  caption in text default 'Caption',
  int_val in int default 17)
  language plpgsql
as $body$
begin
  raise info 'Result: %: %', caption, int_val::text;
end;
$body$;
```


执行此存储过程：

```
call p('Forty-two', 42);
```

返回如下信息：

```
INFO:  Result: Forty-two: 42
```

省略第二个默认参数： 

```
call p('"int_val" default is');
```

返回如下信息：

```
INFO:  Result: "int_val" default is: 17
```

省略两个默认参数： 

```
call p();
```

返回如下信息：

```
INFO:  Result: Caption: 17
```

通过使用形式参数的名称来调用它。 

```
call p(caption => 'Forty-two', int_val=>42);
```

返回如下信息：

```
INFO:  Result: Forty-two: 42
```

仅通过命名的第二个参数调用它。 

```
call p(int_val=>99);
```

返回如下信息：

```
INFO:  Result: Caption: 99
```

只使用第二个未命名的参数可引发错误。

```
call p(99);
```

引发如下错误：

```
ERROR:  procedure p(integer) does not exist
```

## **CLOSE**

关闭一个游标
语法：
close ::= CLOSE { name | ALL }

描述：CLOSE释放与一个已打开游标相关的资源。

参数
name：要关闭的已打开游标的名称。游标仅由非限定名称标识，并且仅在声明它的会话中可见。这决定了其名称的唯一性范围。（在这方面，游标的名称与准备好的语句的名称类似。）


使用关键字ALL代替现存游标的名称会关闭所有现存游标。

示例 ：

```
close all;
 
start transaction;
  declare "Cur-One" no scroll cursor without hold for
  select 17 as v;
 
  declare "Cur-Two" no scroll cursor with hold for
  select 42 as v;
 
  select name, is_holdable::text, is_scrollable::text
  from pg_cursors
  order by name;
  
  close "Cur-One";
commit;
 
select name, is_holdable::text, is_scrollable::text
from pg_cursors
order by name;
 
fetch all from "Cur-Two";
```

第一个pg_cursors查询返回信息如下： 

```
  name   | is_holdable | is_scrollable 
---------+-------------+---------------
 Cur-One | false       | false
 Cur-Two | true        | false
```

第二个pg_cursors查询返回信息如下：

```
  name   | is_holdable | is_scrollable 
---------+-------------+---------------
 Cur-Two | true        | false
```

从“Cur Two”中获取全部的结果返回信息如下： 

```
 v  
----
 42
```

## **COMMENT**

定义或者更改一个对象的注释
语法：
comment_on ::= COMMENT ON 
               { ACCESS METHOD access_method_name
                 | AGGREGATE aggregate_name ( aggregate_signature )
                 | CAST ( source_type AS target_type )
                 | COLLATION object_name
                 | COLUMN relation_name . column_name
                 | CONSTRAINT constraint_name ON table_name
                 | CONSTRAINT constraint_name ON DOMAIN domain_name
                 | CONVERSION object_name
                 | DATABASE object_name
                 | DOMAIN object_name
                 | EXTENSION object_name
                 | EVENT TRIGGER object_name
                 | FOREIGN DATA WRAPPER object_name
                 | FOREIGN TABLE object_name
                 | FUNCTION subprogram_name ( [ subprogram_signature ] 
                   ) | INDEX object_name
                 | LARGE OBJECT large_object_oid
                 | MATERIALIZED VIEW object_name
                 | OPERATOR operator_name ( operator_signature )
                 | OPERATOR CLASS object_name USING index_method
                 | OPERATOR FAMILY object_name USING index_method
                 | POLICY policy_name ON table_name
                 | [ PROCEDURAL ] LANGUAGE object_name
                 | PROCEDURE subprogram_name ( 
                   [ subprogram_signature ] )
                 | PUBLICATION object_name
                 | ROLE object_name
                 | ROUTINE subprogram_name ( [ subprogram_signature ] 
                   ) | RULE rule_name ON table_name
                 | SCHEMA object_name
                 | SEQUENCE object_name
                 | SERVER object_name
                 | STATISTICS object_name
                 | SUBSCRIPTION object_name
                 | TABLE object_name
                 | TABLESPACE object_name
                 | TEXT SEARCH CONFIGURATION object_name
                 | TEXT SEARCH DICTIONARY object_name
                 | TEXT SEARCH PARSER object_name
                 | TEXT SEARCH TEMPLATE object_name
                 | TRIGGER trigger_name ON table_name
                 | TYPE object_name
                 | VIEW object_name } IS { text_literal | NULL }

描述：COMMENT存储关于一个数据库对象的注释。若要删除注释，请将值设置为NULL。
示例：
示例1：增加注释：

```
COMMENT ON DATABASE postgres IS 'Default database';
COMMENT ON INDEX index_name IS 'Special index';
```

示例2：删除注释：

```
COMMENT ON TABLE some_table IS NULL;
```

## **COMMIT**

提交当前事务
语法：
commit ::= COMMIT [ TRANSACTION | WORK ]

描述：使用COMMIT语句提交当前事务。事务所做的所有更改对其他人都是可见的，并保证在发生崩溃时是持久的。

参数：
WORK | TRANSACTION：可选的关键词。它们没有效果。
示例：
创建表：

```
CREATE TABLE sample(k1 int, k2 int, v1 int, v2 text, PRIMARY KEY (k1, k2));
```

开启一个事务，并插入一些数据：

```
BEGIN TRANSACTION; SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
INSERT INTO sample(k1, k2, v1, v2) VALUES (1, 2.0, 3, 'a'), (1, 3.0, 4, 'b');
```

用bsqlsh启动一个新的shell，然后开始另一个事务来插入更多的行。

```
BEGIN TRANSACTION; SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
INSERT INTO sample(k1, k2, v1, v2) VALUES (2, 2.0, 3, 'a'), (2, 3.0, 4, 'b');
```

在每个shell中，检查是否只有当前事务中的行可见。
在第一个shell中：

```
SELECT * FROM sample; 
```

返回信息如下：

```
 k1 | k2 | v1 | v2
----+----+----+----
  1 |  2 |  3 | a
  1 |  3 |  4 | b
```

在第二个shell中：

```
SELECT * FROM sample; 
```

返回信息如下：

```
 k1 | k2 | v1 | v2
----+----+----+----
  2 |  2 |  3 | a
  2 |  3 |  4 | b
```

提交第一个事务并中止第二个事务。 
在第一个shell中：

```
COMMIT TRANSACTION; 
```

在第二个shell中：

```
ABORT TRANSACTION; 
```

在每个shell中，检查是否只有来自提交事务的行可见。 

在第一个shell中，执行：

```
SELECT * FROM sample; 
```

返回信息如下：

```
 k1 | k2 | v1 | v2 
----+----+----+----
  1 |  2 |  3 | a
  1 |  3 |  4 | b
```

在第二个shell中，执行：

```
SELECT * FROM sample; 
```

返回信息如下：

```
 k1 | k2 | v1 | v2 
----+----+----+----
  1 |  2 |  3 | a
  1 |  3 |  4 | b
```

## **COPY**

在一个文件和一个表之间复制数据
语法：
copy_from ::= COPY table_name [ ( column_name [ , ... ] ) ]  FROM 
              { 'filename' | PROGRAM 'command' | STDIN }  
              [ [ WITH ] ( copy_option [ , ... ] ) ]

copy_to ::= COPY { table_name [ ( column_names ) ] | subquery }  TO 
            { 'filename' | PROGRAM 'command' | STDOUT }  
            [ [ WITH ] ( copy_option [ , ... ] ) ]

copy_option ::= FORMAT format_name
                | OIDS [ boolean ]
                | FREEZE [ boolean ]
                | DELIMITER 'delimiter_character'
                | NULL 'null_string'
                | HEADER [ boolean ]
                | QUOTE 'quote_character'
                | ESCAPE 'escape_character'
                | FORCE_QUOTE { ( column_names ) | * }
                | FORCE_NOT_NULL ( column_names )
                | FORCE_NULL ( column_names )
                | ENCODING 'encoding_name'
                | ROWS_PER_TRANSACTION int_literal
                | DISABLE_FK_CHECK
                | REPLACE
                | SKIP int_literal

参数：
table_name：一个现有表的名称（可以是模式限定的）。
column_name：可选的要被复制的列列表。如果没有指定列列表，则该表的所有列都会被复制。
query：指定要复制其结果的SELECT、VALUES、INSERT、UPDATE或DELETE语句。对于INSERT、UPDATE和DELETE语句，必须提供RETURNING子句。
filename：指定要复制的文件的路径。输入文件名可以是绝对路径或相对路径，但输出文件名必须是绝对路径。至关重要的是，该文件必须位于您所连接的TServer的本地文件系统的服务器端。
要使用驻留在客户端上的文件，请指定stdin作为FROM的参数，或指定stdout作为To的参数。
或者，您可以在bsqlsh中使用\copy meta命令。
STDIN 与 STDOUT：指定输入或输出来自客户端应用。
如果从用Python等语言编写的客户端程序中执行COPY TO或COPY FROM语句，则无法使用bsqlsh功能。相反，您必须依靠所选语言的功能将stdin和stdout连接到指定的文件。
但是，如果使用bsqlsh执行COPY FROM，则可以进一步选择在以.sql脚本启动的文件的开头包含COPY调用。如下进行一个测试：

```
drop table if exists t cascade;
create table t(c1 text primary key, c2 text, c3 text);
```

并准备t.sql，脚本内容如下：

```
copy t(c1, c2, c3) from stdin with (format 'csv', header true);
c1,c2,c3
dog,cat,frog
\.
```

请注意 \. 终止符。您只需在bsqlsh提示符下执行\i t.sql即可复制数据。 

ROWS_PER_TRANSACTION：ROWS_PER_TRANSACTION选项定义了COPY命令要使用的事务大小。
默认值：20000。
例如，如果要复制的元组总数为5000，并且ROWS_PER_TRANSACTION设置为1000，则数据库将创建5个事务，每个事务将插入1000行。如果在执行复制命令的过程中出现错误，则可以基于已完成的事务持久化一些元组。这意味着，如果在插入第3500行之后发生错误，那么前3000行将被保留在数据库中。
1 to 1000 → Transaction_1
1001 to 2000 → Transaction_2
2001 to 3000 → Transaction_3
3001 to 3500 → Error
前3000行将持久化到表中，tuples_processed将显示3000行。
REPLACE：如果新行的主键/唯一键与现有行的主键冲突，REPLACE选项将替换表中的现有行。
请注意，REPLACE不适用于具有1个以上唯一约束的表。
默认：默认情况下报告冲突错误。
DISABLE_FK_CHECK：将新行复制到表时，DISABLE_FK_CHECK选项将跳过外键检查。
默认值：默认情况下，当未提供DISABLE_FK_check选项时，始终执行外键检查。
SKIP n：SKIP n选项跳过文件的前n行。n必须是非负整数。
默认值：0，不跳过任何行。

示例：
假设有如下表：

```
CREATE TABLE users(id BIGSERIAL PRIMARY KEY, name TEXT);
INSERT INTO users(name) VALUES ('John Doe'), ('Jane Doe'), ('Dorian Gray');
```

示例1：导出整个表
使用绝对路径将整个表复制到CSV文件中，标题中包含列名。 

```
COPY users TO '/home/bigmath/users.txt.sql' DELIMITER ',' CSV HEADER;
```

示例2：使用带列选择的WHERE子句导出分部表
在以下示例中，WHERE子句用于筛选行，而仅用于筛选名称列。 
COPY (SELECT name FROM users where name='Dorian Gray') TO '/home/bigmath/users.txt.sql' DELIMITER ',' CSV HEADER;

示例3：从CSV文件导入


在下面的示例中，在前面的示例中导出的数据将导入到users表中 

```
COPY users FROM '/home/bigmath/users.txt.sql' DELIMITER ',' CSV HEADER;
```

示例4：使用跳过行导入
假设我们运行了一次命令，但它在中间失败了，就好像服务器崩溃了一样。由于我们使用ROWS_PER_TRANSACTION=5000，我们可以以5000的倍数恢复导入： 

```
COPY users FROM '/home/bigmath/users.txt.sql' WITH (FORMAT CSV,HEADER, DELIMITER ',', ROWS_PER_TRANSACTION 5000, SKIP 50000);
```

示例5：导入并替换行
如果数据库中存在重复的行，我们可以使用REPLACE来追加新行： 

```
COPY users FROM '/home/bigmath/users.txt.sql' WITH (FORMAT CSV,HEADER, DELIMITER ',', REPLACE);
```

示例6：导入时禁用外键检查
如果我们确定外键引用的行已经存在，我们可以禁用对它们的检查，以加快导入速度：

```
COPY users FROM '/home/bigmath/users.txt.sql' WITH (FORMAT CSV,HEADER, DELIMITER ',', DISABLE_FK_CHECK);
```

示例7：使用多个参数

```
COPY users FROM '/home/bigmath/users.txt.sql' WITH (FORMAT CSV,HEADER, DELIMITER ',', ROWS_PER_TRANSACTION 5000, DISABLE_FK_CHECK, REPLACE, SKIP 50);
```

## **CREATE AGGREGATE**

定义一个新的聚集函数
语法：
create_aggregate ::= create_aggregate_normal
                     | create_aggregate_order_by
                     | create_aggregate_old

create_aggregate_normal ::= CREATE AGGREGATE aggregate_name ( 
                            { aggregate_arg [ , ... ] | * } )  ( SFUNC 
                            = sfunc , STYPE = state_data_type 
                            [ , aggregate_normal_option [ ... ] ] )

create_aggregate_order_by ::= CREATE AGGREGATE aggregate_name ( 
                              [ aggregate_arg [ , ... ] ] ORDER BY 
                              aggregate_arg [ , ... ] )  ( SFUNC = 
                              sfunc , STYPE = state_data_type 
                              [ , aggregate_order_by_option [ ... ] ] 
                              )

create_aggregate_old ::= CREATE AGGREGATE aggregate_name ( BASETYPE = 
                         base_type ,  SFUNC = sfunc , STYPE = 
                         state_data_type 
                         [ , aggregate_old_option [ ... ] ] )

aggregate_arg ::= [ aggregate_arg_mode ] [ formal_arg ] arg_type

aggregate_normal_option ::= SSPACE = state_data_size
                            | FINALFUNC = ffunc
                            | FINALFUNC_EXTRA
                            | FINALFUNC_MODIFY = 
                              { READ_ONLY | SHAREABLE | READ_WRITE }
                            | COMBINEFUNC = combinefunc
                            | SERIALFUNC = serialfunc
                            | DESERIALFUNC = deserialfunc
                            | INITCOND = initial_condition
                            | MSFUNC = msfunc
                            | MINVFUNC = minvfunc
                            | MSTYPE = mstate_data_type
                            | MSSPACE = mstate_data_size
                            | MFINALFUNC = mffunc
                            | MFINALFUNC_EXTRA
                            | MFINALFUNC_MODIFY = 
                              { READ_ONLY | SHAREABLE | READ_WRITE }
                            | MINITCOND = minitial_condition
                            | SORTOP = sort_operator
                            | PARALLEL = 
                              { SAFE | RESTRICTED | UNSAFE }

aggregate_order_by_option ::= SSPACE = state_data_size
                              | FINALFUNC = ffunc
                              | FINALFUNC_EXTRA
                              | FINALFUNC_MODIFY = 
                                { READ_ONLY | SHAREABLE | READ_WRITE }
                              | INITCOND = initial_condition
                              | PARALLEL = 
                                { SAFE | RESTRICTED | UNSAFE }
                              | HYPOTHETICAL

aggregate_old_option ::= SSPACE = state_data_size
                         | FINALFUNC = ffunc
                         | FINALFUNC_EXTRA
                         | FINALFUNC_MODIFY = 
                           { READ_ONLY | SHAREABLE | READ_WRITE }
                         | COMBINEFUNC = combinefunc
                         | SERIALFUNC = serialfunc
                         | DESERIALFUNC = deserialfunc
                         | INITCOND = initial_condition
                         | MSFUNC = msfunc
                         | MINVFUNC = minvfunc
                         | MSTYPE = mstate_data_type
                         | MSSPACE = mstate_data_size
                         | MFINALFUNC = mffunc
                         | MFINALFUNC_EXTRA
                         | MFINALFUNC_MODIFY = 
                           { READ_ONLY | SHAREABLE | READ_WRITE }
                         | MINITCOND = minitial_condition
                         | SORTOP = sort_operator

描述：CREATE AGGREGATE定义一个新的聚集函数。选项的顺序无关紧要，即使是强制性选项BASETYPE、SFUNC和STYPE，也可能以任何顺序出现。
示例：
示例1：正常语法示例

```
CREATE AGGREGATE sumdouble (float8) (
              STYPE = float8,
              SFUNC = float8pl,
              MSTYPE = float8,
              MSFUNC = float8pl,
              MINVFUNC = float8mi
           );
CREATE TABLE normal_table(
             f float8,
             i int
           );
INSERT INTO normal_table(f, i) VALUES
             (0.1, 9),
             (0.9, 1);
SELECT sumdouble(f), sumdouble(i) FROM normal_table;
```

示例2：

```
CREATE AGGREGATE oldcnt(
             SFUNC = int8inc,
             BASETYPE = 'ANY',
             STYPE = int8,
             INITCOND = '0'
           );
SELECT oldcnt(*) FROM pg_aggregate;
```

示例3：零参数聚合示例

```
CREATE AGGREGATE newcnt(*) (
             SFUNC = int8inc,
             STYPE = int8,
             INITCOND = '0',
             PARALLEL = SAFE
           );
SELECT newcnt(*) FROM pg_aggregate;
```

## **CREATE CAST**

定义一种新的造型
语法：
create_cast ::= create_cast_with_function
                | create_cast_without_function
                | create_cast_with_inout

create_cast_with_function ::= CREATE CAST ( cast_signature ) WITH 
                              FUNCTION  subprogram_name 
                              [ ( subprogram_signature ) ] 
                              [ AS ASSIGNMENT | AS IMPLICIT ]

create_cast_without_function ::= CREATE CAST ( cast_signature ) 
                                 WITHOUT FUNCTION 
                                 [ AS ASSIGNMENT | AS IMPLICIT ]

create_cast_with_inout ::= CREATE CAST ( cast_signature ) WITH INOUT 
                           [ AS ASSIGNMENT | AS IMPLICIT ]

cast_signature ::= source_type AS target_type

描述：CREATE CAST定义一种新的造型。 一种造型指定如何在两种数据类型之间执行转换。例如，SELECT CAST(42 AS float8);
通过调用一个之前指定的函数（这种情况中是 float8(int4)）把整型常量 42 转换成类型 float8（如果没有定义合适的造型， 该转换会失败）。
默认情况下，只有一次显式造型请求才会调用造型， 形式是CAST(x AS typename) or x::typename。
如果造型被标记为AS ASSIGNMENT，那么在为一个目标数据 类型的列赋值时会隐式地调用它。例如，假设foo.f1是 一个类型text的列，那么如果从类型integer 到类型text的造型被标记为AS ASSIGNMENT， 则：

INSERT INTO foo (f1) VALUES (42);
将被允许，否则不会允许（我们通常使用赋值造型 来描述此类造型）。

如果造型被标记为AS IMPLICIT，那么可以在任何上下文 中隐式地调用它，无论是赋值还是在一个表达式内部（我们通常用术语 隐式造型来描述这类造型）。例如，考虑这个 查询：

```
SELECT 2 + 4.0;
```

解析器初始会把常量分别标记为类型integer和 numeric。在系统目录中没有integer + numeric操作符，但是有一个 numeric + numeric操作符。 因此，如果有一种可用的从integer到 numeric的造型且被标记为AS IMPLICIT — 实际上确实有 — 该查询将会成功。解析器将应用该隐式造型 并且解决该查询，就好像它被写成：

```
SELECT CAST ( 2 AS numeric ) + 4.0;
```

参数：
source_type：该造型的源数据类型的名称。
target_type：该造型的目标数据类型的名称。
create_cast_with_function:被用于执行该造型的函数。函数名称可以用模式限定。如果没有被限定，将在模式搜索路径中查找该函数。函数的结果数据类型必须是该造型的 目标数据类型。它的参数讨论如下。如果没有指定参数列表，则该函数名称在其模式中必须是唯一的。
create_cast_without_function:指示源类型可以二进制强制到目标类型，因此执行该造型不需要函数。
create_cast_with_inout:指示该造型是一种 I/O 转换造型，执行需要调用源数据类型的输出函数，并且把结果字符串传递给目标数据类型的输入函数。

AS ASSIGNMENT:指示该造型可以在赋值的情况下被隐式调用。
AS IMPLICIT:指示该造型可以在任何上下文中被隐式调用。
示例：
示例1：WITH FUNCTION

```
CREATE FUNCTION sql_to_date(integer) RETURNS date AS $$
             SELECT $1::text::date
             $$ LANGUAGE SQL IMMUTABLE STRICT;
CREATE CAST (integer AS date) WITH FUNCTION sql_to_date(integer) AS ASSIGNMENT;
SELECT CAST (20200603 AS date);
```

示例2：WITHOUT FUNCTION

```
CREATE TYPE myfloat4;
CREATE FUNCTION myfloat4_in(cstring) RETURNS myfloat4
             LANGUAGE internal IMMUTABLE STRICT PARALLEL SAFE AS 'float4in';
CREATE FUNCTION myfloat4_out(myfloat4) RETURNS cstring
             LANGUAGE internal IMMUTABLE STRICT PARALLEL SAFE AS 'float4out';
CREATE TYPE myfloat4 (
             INPUT = myfloat4_in,
             OUTPUT = myfloat4_out,
             LIKE = float4
           );
SELECT CAST('3.14'::myfloat4 AS float4);
CREATE CAST (myfloat4 AS float4) WITHOUT FUNCTION;
SELECT CAST('3.14'::myfloat4 AS float4);
```

示例3：WITH INOUT

```
CREATE TYPE myint4;
CREATE FUNCTION myint4_in(cstring) RETURNS myint4
             LANGUAGE internal IMMUTABLE STRICT PARALLEL SAFE AS 'int4in';
CREATE FUNCTION myint4_out(myint4) RETURNS cstring
             LANGUAGE internal IMMUTABLE STRICT PARALLEL SAFE AS 'int4out';
CREATE TYPE myint4 (
             INPUT = myint4_in,
             OUTPUT = myint4_out,
             LIKE = int4
           );
SELECT CAST('2'::myint4 AS int4);
CREATE CAST (myint4 AS int4) WITH INOUT;
SELECT CAST('2'::myint4 AS int4);
```

## **CREATE DATABASE**

创建一个新数据库
语法：
create_database ::= CREATE DATABASE name [ create_database_options ]

create_database_options ::= [ WITH ] [ OWNER [ = ] user_name ]  
                            [ TEMPLATE [ = ] template_name ]  
                            [ ENCODING [ = ] encoding ]  
                            [ LC_COLLATE [ = ] lc_collate ]  
                            [ LC_CTYPE [ = ] lc_ctype ]  
                            [ ALLOW_CONNECTIONS [ = ] allowconn ]  
                            [ CONNECTION LIMIT [ = ] connlimit ]  
                            [ IS_TEMPLATE [ = ] istemplate ]  
                            [ COLOCATION [ = ] { 'true' | 'false' } ]
描述：CREATE DATABASE创建一个新的AiSQL数据库。
参数：
name：要创建的数据库名。
[ WITH ] OWNER user_name：指定将拥有新数据库的用户的角色名称。如果未指定，则数据库创建者为所有者。
TEMPLATE template：指定用于创建新数据库的模板的名称。
ENCODING encoding：指定要在新数据库中使用的字符集编码。
LC_COLLATE lc_collate：要在新数据库中使用的排序规则顺序。
LC_CTYPE lc_ctype：要在新数据库中使用的字符分类。
ALLOW_CONNECTIONS allowconn：指定false以禁止连接到数据库。默认值为true，允许连接到数据库。
CONNECTION_LIMIT connlimit：指定可以与此数据库建立的并发连接数。-1 （默认值）表示没有限制。
IS_TEMPLATE istemplate：如果为真（true），则任何具有CREATEDB权限的用户都可以克隆此数据库；如果为假（false），默认，则指定为仅超级用户或数据库所有者可以克隆它。
COLOCATION：指定为true，则此数据库的表应在单一分片中共址。具体细节，请参阅[共址表](#_共址表（colocated table）)，了解共址表的详细信息。
默认值为false，数据库中的每个表都有自己的分片集。
示例：
创建一个共址数据库

```
CREATE DATABASE company WITH COLOCATION = true;
```

此例中，数据库company的表将在单一分片中共址。

## **CREATE DOMAIN**

定义一个新的域
语法：
create_domain ::= CREATE DOMAIN name [ AS ] data_type 
                  [ DEFAULT expression ] 
                  [ [ domain_constraint [ ... ] ] ]

domain_constraint ::= [ CONSTRAINT constraint_name ] 
                      { NOT NULL | NULL | CHECK ( expression ) }

描述：CREATE DOMAIN创建一个新的域。
参数：
name：要被创建的域的名称（可以被模式限定）。
data_type：域的底层数据类型。可以包括数组指示符。
DEFAULT expression：DEFAULT子句为该域数据类型的列指定一个默认值。 该值是任何没有变量的表达式（但不允许子查询）。默认值表达式的数据类型必须匹配域的数据类型。如果没有指定默认值，那么默认值就是空值。
CONSTRAINT constraint_name：一个约束的名称（可选）。如果没有指定，系统会生成一个名称。

NOT NULL：这个域的值通常不能为空值。
NULL：这个域的值允许为空值。这是默认值。
CHECK (expression)：CHECK子句指定该域的值必须满足的完整性约束，每一个约束必须是一个产生布尔结果的表达式。它应该使用关键词VALUE来引用要被测试的值。计算为 TRUE 或者 UNKNOWN 的表达式成功。如果该表达式产生一个 FALSE 结果，会报告一个错误并且该值不允许被转换成该域类型。

示例：

```
CREATE DOMAIN phone_number AS TEXT CHECK(VALUE ~ '^\d{3}-\d{3}-\d{4}$');
CREATE TABLE person(first_name TEXT, last_name TEXT, phone_number phone_number);
```

## **CREATE EXTENSION**

安装一个扩展
语法：
create_extension ::= CREATE EXTENSION [ IF NOT EXISTS ] extension_name 
                      [ WITH ] [ SCHEMA schema_name ] 
                     [ VERSION version ] [ CASCADE ]

描述： CREATE EXTENSION把一个新的扩展载入到当前数据库中。不能有同名扩展已经被载入。
参数：
IF NOT EXISTS：已有同名扩展存在时不要抛出错误。这种情况下会发出一个提示。 注意，不保证现有的扩展与将要从当前可用的脚本文件创建的脚本有任何相似。
extension_name：要安装的扩展的名称。
schema_name：假定该扩展允许其内容被重定位，这是要在其中安装该扩展的对象的 模式名称。被提到的模式必须已经存在。
version：要安装的扩展的版本。这可以写成一个标识符或者一个字符串。 默认版本在该扩展的控制文件中指定。
CASCADE：自动安装这个扩展所依赖的任何还未安装的扩展。

示例：

```
CREATE SCHEMA myschema;
CREATE EXTENSION pgcrypto WITH SCHEMA myschema VERSION '1.3';
 
CREATE EXTENSION IF NOT EXISTS earthdistance CASCADE;
```

## **CREATE FOREIGN DATA WRAPPER**

定义一个新的外部数据包装器
语法：
create_foreign_data_wrapper ::= CREATE FOREIGN DATA WRAPPER fdw_name  
                                [ HANDLER handler_name | NO HANDLER ] 
                                [ VALIDATOR validator_name
                                  | NO VALIDATOR ] 
                                [ OPTIONS ( fdw_options ) ]
描述：CREATE FOREIGN DATA WRAPPER创建一个新的外部数据包装器。定义外部数据包装器的用户将成为它的拥有者。
参数：
HANDLER：将调用handler_function来检索外部表的执行函数。这些功能是规划者和执行者所必需的。处理程序函数不接受任何参数，其返回类型应为fdw_handler。如果没有提供处理程序函数，则只能声明（而不能访问）使用包装器的外部表。
VALIDATOR：validator_function用于验证提供给外部数据包装器的选项，以及使用外部数据包装的外部服务器、用户映射和外部表。验证器函数有两个参数：一个文本数组（类型为text[]），其中包含要验证的选项，另一个是类型 oid，它将是包含该选项的系统目录的 OID，与选项关联的对象存储在该OID中。如果没有提供验证器函数（或未指定validator），则在创建时不会检查选项。
OPTIONS：这个子句为新的外部数据包装器指定选项。允许的选项名称和值与每一个外部数据包装器有关，并且它们会被该外部数据包装器的验证器函数验证。 选项名称必须唯一。

示例：

```
CREATE FOREIGN DATA WRAPPER file HANDLER file_fdw_handler;
```

## **CREATE FOREIGN TABLE**

定义一个新的外部表
语法：
create_foreign_table ::= CREATE FOREIGN TABLE [ IF NOT EXISTS ] 
                         table_name ( [ foreign_table_elem [ , ... ] ] 
                         )  SERVER server_name 
                         [ OPTIONS ( fdw_options ) ]

描述：CREATE FOREIGN TABLE在当前数据库中创建一个新的外部表。该表将由发出这个命令的用户所拥有。如果指定的数据库中已存在table_name，则除非使用If NOT exists子句，否则将引发错误。


参数：
COLLATE：COLLATE子句为该列（必须是一个可排序的数据类型）赋予一个排序规则。如果没有指定，则会使用该列的数据类型的默认排序规则。
server_name：要用于该外部表的一个现有外部服务器的名称。有关定义一个服务器 的细节可以参考[CREATE SERVER](#_CREATE SERVER)。
OPTIONS ：要与新外部表或者它的一个列相关联的选项。被允许的选项名称和值是与每一个外部数据包装器相关的，并且它们会被该外部数据包装器的验证器函数验证。不允许重复的选项名称（不过一个表选项和一个列选项重名是可以的）。

示例：

```
 CREATE FOREIGN TABLE mytable (col1 int, col2 int) SERVER my_server OPTIONS (schema 'external_schema', table 'external_table');
```

## **CREATE FUNCTION**

定义一个新函数 
语法：
create_function ::= CREATE [ OR REPLACE ] FUNCTION subprogram_name ( 
                    [ arg_decl_with_dflt [ , ... ] ] )  
                    { RETURNS data_type
                      | RETURNS TABLE ( { column_name data_type } 
                        [ , ... ] ) }  
                    { unalterable_fn_attribute
                      | alterable_fn_only_attribute
                      | alterable_fn_and_proc_attribute } [ ... ]

arg_decl_with_dflt ::= arg_decl [ { DEFAULT | = } expression ]

arg_decl ::= [ formal_arg ] [ arg_mode ] arg_type

subprogram_signature ::= arg_decl [ , ... ]

unalterable_fn_attribute ::= WINDOW
                             | LANGUAGE lang_name
                             | AS subprogram_implementation

lang_name ::= SQL | PLPGSQL | C

subprogram_implementation ::= ' sql_stmt_list '
                              | ' plpgsql_block_stmt '
                              | ' obj_file ' [ , ' link_symbol ' ]

sql_stmt_list ::= sql_stmt ; [ sql_stmt ... ]

alterable_fn_and_proc_attribute ::= SET run_time_parameter 
                                    { TO value
                                      | = value
                                      | FROM CURRENT }
                                    | RESET run_time_parameter
                                    | RESET ALL
                                    | [ EXTERNAL ] SECURITY 
                                      { INVOKER | DEFINER }

alterable_fn_only_attribute ::= volatility
                                | on_null_input
                                | PARALLEL parallel_mode
                                | [ NOT ] LEAKPROOF
                                | COST int_literal
                                | ROWS int_literal

volatility ::= IMMUTABLE | STABLE | VOLATILE

on_null_input ::= CALLED ON NULL INPUT
                  | RETURNS NULL ON NULL INPUT
                  | STRICT

parallel_mode ::= UNSAFE | RESTRICTED | SAFE

描述：与表等其他模式对象一样，函数不可避免地具有所有者。创建函数时，不能显式指定所有者。相反，它被隐式定义为内置的current_user函数在创建该函数的会话中被调用时返回的内容。此用户必须对函数的模式、参数数据类型和返回数据类型具有使用权限。
CREATE OR REPLACE FUNCTION不会更改已授予现有函数的权限。若要使用此语句，当前用户必须拥有该函数，或者是拥有该函数的角色的成员。
相反，如果您删除并重新创建一个函数，则新函数与旧函数不是同一个实体。因此，您将不得不删除依赖于旧函数的现有对象。（删除函数CASCADE可以实现这一点。）或者，ALTER FUNCTION可以用于更改现有函数的大部分属性。

参数：
subprogram_name：要创建的函数的名称（可以被模式限定）。
arg_mode：一个参数的模式：IN、OUT、INOUT或者VARIADIC。如果省略，默认为IN。
formal_arg：一个参数的名称。
arg_type：该函数参数（如果有）的数据类型（可以是模式限定的）。参数类型可以是基本类型、组合类型或者域类型，或者可以引用一个表列的类型。
{ DEFAULT | = } expression：如果参数没有被指定值时要用作默认值的表达式。
LANGUAGE：默认支持的语言有SQL、PLPGSQL和C。
WINDOW：WINDOW表示该函数是一个窗口函数而不是一个普通函数。当前只用于用 C 编写的函数。在替换一个现有函数定义时，不能更改WINDOW属性。

IMMUTABLE | STABLE | VOLATILE
这些属性告知查询优化器该函数的行为。最多只能指定其中一个。如果这些都不出现，则会默认为VOLATILE。
IMMUTABLE表示该函数不能修改数据库并且对于给定的参数值总是会返回相同的值。也就是说，它不会做数据库查找或者使用没有在其参数列表中直接出现的信息。如果给定合格选项，任何用全常量参数对该函数的额调用可以立刻用该函数值替换。
STABLE表示该函数不能修改数据库，并且对于相同的参数值，它在一次表扫描中将返回相同的结果。但是这种结果在不同的 SQL 语句执行期间可能会变化。对于那些结果依赖于数据库查找、参数变量（例如当前时区）等的函数来说，这是合适的（对希望查询被当前命令修改的行的AFTER触发器不适合）。还要注意current_timestamp函数族适合被标记为稳定，因为它们的值在一个事务内不会改变。
VOLATILE表示该函数的值在一次表扫描中都有可能改变，因此不能做优化。在这种意义上，相对较少的数据库函数是不稳定的，一些示例是random()、currval()、timeofday()。但是注意任何有副作用的函数都必须被分类为不稳定的，即便其结果是可以预测的，这是为了调用被优化掉。一个示例是setval()。
CALLED ON NULL INPUT | RETURNS NULL ON NULL INPUT | STRICT
CALLED ON NULL INPUT（默认）表示在某些参数为空值时应正常调用该函数。如果有必要，函数的作者应该负责检查空值并且做出适当的相应。
RETURNS NULL ON NULL INPUT或STRICT表示只要其任意参数为空值，该函数就会返回空值。如果指定了这个参数，当有空值参数时该函数不会被执行，而是自动返回一个空值结果。
[EXTERNAL] SECURITY INVOKER | [EXTERNAL] SECURITY DEFINER
SECURITY INVOKER表示要用调用该函数的用户的特权来执行它。这是默认值。SECURITY DEFINER指定要用拥有该函数的用户的特权来执行该函数。
为了符合 SQL，允许使用关键词EXTERNAL。但是它是可选的，因为与 SQL 中不同，这个特性适用于所有函数而不仅是那些外部函数。
PARALLEL：PARALLEL UNSAFE表示该函数不能在并行模式中运行并且 SQL 语句中存在一个这样的函数会强制使用顺序执行计划。这是默认选项。PARALLEL RESTRICTED表示该函数能在并行模式中运行，但是其执行被限制在并行组的领导者中。PARALLEL SAFE表示该函数对于在并行模式中运行是安全的并且不受限制。
COST int_literal：一个给出该函数的估计执行代价的正数。
ROWS int_literal：一个正数，它给出规划器期望该函数返回的行数估计。只有当该函数被声明为返回一个集合时才允许这个参数。
标量函数和表函数：使用RETURN子句的适当变体创建标量函数或表函数。请注意，标量不仅可以表示int、numeric、text等数据类型（以及基于这些数据类型的域）的原子值；它还可以表示单个复合值，类似于用户定义的类型。 
标量函数示例：

```
create schema s;
 
create type s.x as (i int, t text);
 
create function s.f(i in int, t in text)
  returns s.x
  security invoker
  set search_path = pg_catalog, pg_temp
  language plpgsql
as $body$
begin
  return (i*2, t||t)::s.x;
end;
$body$;
 
WITH c as (select s.f(42, 'dog') as v)
select
  (v).i, (v).t
FROM c;
```

返回信息如下：

```
 i  |   t    
----+--------
 84 | dogdog
```

表函数示例：

```
create table s.t(k serial primary key, v text);
insert into s.t(v) values ('dog'), ('cat'), ('frog');
 
create function s.f()
  returns table(z text)
  security definer
  set search_path = pg_catalog, pg_temp
  language plpgsql
as $body$
begin
  z := 'Starting content of t '; return next;
  z := '----------------------'; return next;
  for z in (select v from s.t order by k) loop
    return next;
  end loop;
 
  begin
    insert into s.t(v) values ('mouse');
  exception
    when string_data_right_truncation then
      z := ''; return next;
      z := 'string_data_right_truncation caught'; return next;
  end;
 
  insert into s.t(v) values ('bird');
 
  z := ''; return next;
  z := 'Finishing content of t'; return next;
  z := '----------------------'; return next;
  for z in (select v from s.t order by k) loop
    return next;
  end loop;
end;
$body$;
 
\t on
select z from s.f();
\t off
```

返回信息如下：

```
 Starting content of t 
 ----------------------
 dog
 cat
 frog
 
 Finishing content of t
 ----------------------
 dog
 cat
 frog
 mouse
 bird
```

## **CREATE GROUP**

定义一个新的数据库组角色
语法：
create_group ::= CREATE GROUP role_name 
                 [ [ WITH ] role_option [ , ... ] ]

role_option ::= SUPERUSER
                | NOSUPERUSER
                | CREATEDB
                | NOCREATEDB
                | CREATEROLE
                | NOCREATEROLE
                | INHERIT
                | NOINHERIT
                | LOGIN
                | NOLOGIN
                | CONNECTION LIMIT connlimit
                | [ ENCRYPTED ] PASSWORD  ' password ' 
                | PASSWORD NULL
                | VALID UNTIL  ' timestamp ' 
                | IN ROLE role_name [ , ... ]
                | IN GROUP role_name [ , ... ]
                | ROLE role_name [ , ... ]
                | ADMIN role_name [ , ... ]
                | USER role_name [ , ... ]
                | SYSID uid

有关更多详细信息，请参阅[CREATE ROLE](#_CREATE ROLE)。 

示例：
创建一个可以管理数据库和角色的组。

```
CREATE GROUP SysAdmin WITH CREATEDB CREATEROLE;
```

## **CREATE INDEX**

定义一个新索引
语法：
create_index ::= CREATE [ UNIQUE ] INDEX 
                 [ CONCURRENTLY | NONCONCURRENTLY ]  
                 [ [ IF NOT EXISTS ] name ] ON [ ONLY ] table_name  
                 [ USING access_method_name ] ( index_elem [ , ... ] ) 
                  [ INCLUDE ( column_name [ , ... ] ) ]  
                 [ TABLESPACE tablespace_name ]  
                 [ SPLIT { INTO int_literal TABLETS
                           | AT VALUES ( split_row [ , ... ] ) } ] 
                 [ WHERE boolean_expression ]

index_elem ::= { column_name | ( expression ) } 
               [ operator_class_name ] [ HASH | ASC | DESC ] 
               [ NULLS { FIRST | LAST } ]

描述：定义一个新索引。
当在填充的表上创建索引时，AiSQL会自动将现有数据填充到索引中。在大多数情况下，这使用在线模式迁移。下表解释了联机创建索引和未联机创建索引之间的一些区别。

| 条件                                                         | 在线 | 未联机 |
| ------------------------------------------------------------ | ---- | ------ |
| 在CREATE INDEX期间执行其他DML是否安全？                      | 是   | 否     |
| 在CREATE INDEX期间保持其他事务活动？                         | 通常 | 否     |
| 是否并行索引加载？                                           | 是   | 否     |
| 虽然默认情况下启用了联机索引回填，但支持CREATE INDEX CONCURRENTLY。一些限制适用（请参阅并发）。 |      |        |
| 如果要禁用CREATE INDEX的联机架构迁移，请在所有节点的Master和 TServer上设置标志bsql_disable_index_backfill=true。 |      |        |
| 若要禁用一个CREATE INDEX的联机架构迁移，请使用CREATE INDEX NONCURRENTLY。 |      |        |

关于共址，索引如下表所示。如果表是共址的，那么它的索引也是共址的；如果表未共址，则其索引也未共址。

**分区索引**
在分区表上创建索引会自动为默认表空间中的每个分区创建相应的索引。也可以在每个分区上单独创建索引，在以下情况下应该这样做：

* 创建索引时，期望并行写入，因为在分区表上的并发创建索引还不支持，在这种情况下，最好针对每一个分区单独并发创建索引。 
* 对应正在使用的行级地理分区，在这种情况下，在每个分区上分别创建索引，以及自定义创建每个索引的表空间。
* 分区表不支持CREATE INDEX CONCURRENTLY。
  **参数：**
  UNIQUE：强制表中不允许有重复的值。
  CONCURRENTLY：启用联机架构迁移，但有一些限制：
* 在临时表上创建索引时，将禁用联机架构迁移。
* 分区表不支持CREATE INDEX CONCURRENTLY。
* 事务块内不支持CREATE INDEX CONCURRENTLY。

NONCONCURRENTLY：禁用联机架构迁移。
ONLY：如果该表是分区表，指示不要在分区上递归创建索引。默认会递归创建索引。
access_method_name：索引访问方法的名称。默认情况下，lsm用于AiSQL表，btree用于其他表（例如，临时表）。可以使用bmgin访问方法在AiSQL中创建GIN索引。 
INCLUDE clause：可选的INCLUDE子句指定一个列的列表，其中的列将被包括在索引中作为非键列。
TABLESPACE clause：指定描述此索引的放置配置的表空间的名称。默认情况下，索引被放置在pg_default表空间中，该表空间将索引的片剂均匀地分布在集群中。
WHERE clause：部分索引是建立在表的子集上的索引，只包括满足WHERE子句中指定条件的行。它可以用于从索引中排除NULL或公共值，或者只包括感兴趣的行。这加快了对表的任何写入，因为包含公共列值的行不需要索引。它还减少了索引的大小，从而提高了使用索引的读取查询的速度。
Name：要创建的索引名称。
table_name：要被索引的表的名称。
index_elem：
column_name：指定表的列名。
Expression：一个基于一个或者更多个表列的表达式，用括号括起来。 

* HASH-使用列的哈希。这是第一列的默认选项，用于分割索引表。
* ASC—按升序排序。这是索引的第二列和后续列的默认选项。
* DESC—按降序排序。
* NULLS FIRST-指定null在非null之前排序。当指定DESC时，这是默认值。
* NULLS LAST-指定null在非null之后排序。这是未指定DESC时的默认值。

SPLIT INTO：对于哈希分片索引，可以使用SPLIT INTO子句指定要为索引创建的分片的数量，然后将hash 范围均匀地分布在这些分片上。
使用SPLIT INTO预拆分索引可在生产集群上分配索引工作负载。例如，如果您有3台服务器，将索引拆分为30分片，可以在索引上提供更高的写入吞吐量。
注意：
默认情况下，AiSQL将索引预拆分为bsql_num_shards_per_tserver*num_of_tserver个分片。而SPLIT INTO子句可用于在每个索引的基础上重写该设置。
SPLIT AT VALUES：对于范围分片索引，可以使用SPLIT AT VALUES子句设置拆分点以预拆分范围分片的索引。

例如：

```
CREATE TABLE tbl(
  a INT,
  b INT,
  PRIMARY KEY(a ASC, b DESC);
);
 
CREATE INDEX idx1 ON tbl(b ASC, a DESC) SPLIT AT VALUES((100), (200), (200, 5));
```

在上面的示例中，有三个分割点，因此将为索引创建四个分片： 
    分片1: b=<lowest>, a=<lowest> to b=100, a=<lowest>
    分片2: b=100, a=<lowest> to b=200, a=<lowest>
    分片3: b=200, a=<lowest> to b=200, a=5
    分片4: b=200, a=5 to b=<highest>, a=<highest>

注意： 
默认情况下，AiSQL将创建一个范围分片索引作为单个分片。SPLIT AT子句可用于在每个索引的基础上重写该设置。 

**示例：**

示例1：具有HASH列排序的唯一索引 
使用哈希排序列创建唯一索引。 

```
CREATE TABLE products(id int PRIMARY KEY,
                                 name text,
                                 code text);
CREATE UNIQUE INDEX ON products(code);
\d products
```

返回信息如下：

```
              Table "public.products"
 Column |  Type   | Collation | Nullable | Default
--------+---------+-----------+----------+---------
 id     | integer |           | not null |
 name   | text    |           |          |
 code   | text    |           |          |
Indexes:
    "products_pkey" PRIMARY KEY, lsm (id HASH)
    "products_code_idx" UNIQUE, lsm (code HASH)
```

示例2：ASC有序索引
使用升序键创建索引。 

```
CREATE INDEX products_name ON products(name ASC);
\d products_name
```

返回信息如下：

```
   Index "public.products_name"
 Column | Type | Key? | Definition
--------+------+------+------------
 name   | text | yes  | name
lsm, for table "public.products
```


示例3：INCLUDE 列
使用升序键创建索引，并将其他列包括为非键列。

```
CREATE INDEX products_name_code ON products(name) INCLUDE (code);
\d products_name_code;
```

返回信息如下：

```
 Index "public.products_name_code"
 Column | Type | Key? | Definition
--------+------+------+------------
 name   | text | yes  | name
 code   | text | no   | code
lsm, for table "public.products"
```

示例4：创建一个指定分片数量的索引
要指定索引的分片数量，可以将CREATE index语句与SPLIT INTO子句一起使用。

```
CREATE TABLE employees (id int PRIMARY KEY, first_name TEXT, last_name TEXT) SPLIT INTO 10 TABLETS;
CREATE INDEX ON employees(first_name, last_name) SPLIT INTO 10 TABLETS;
```

示例5：部分索引 
考虑一个维护装运信息的应用程序。它有一个带有delivery_status列的shipments表。如果应用程序需要频繁访问飞行中的装运，则可以使用部分索引来排除装运状态delivered的行。 

```
create table shipments(id int, delivery_status text, address text, delivery_date date);
create index shipment_delivery on shipments(delivery_status, address, delivery_date) where delivery_status != 'delivered';
```

**故障排除：**
**1.无效索引**
如果联机CREATE INDEX失败，则可能会留下无效索引。这些索引在查询中不可用，并且会导致内部操作，因此应该删除它们。

例如，以下命令可能会创建无效索引：

```
CREATE TABLE uniqueerror (i int);
INSERT INTO uniqueerror VALUES (1), (1);
CREATE UNIQUE INDEX ON uniqueerror (i);
```

此时报告如下错误信息：

```
ERROR:  ERROR:  duplicate key value violates unique constraint "uniqueerror_i_idx"
\d uniqueerror
```

返回信息如下：

```
            Table "public.uniqueerror"
 Column |  Type   | Collation | Nullable | Default
--------+---------+-----------+----------+---------
 i      | integer |           |          |
Indexes:
    "uniqueerror_i_idx" UNIQUE, lsm (i HASH) INVALID
```

删除无效索引，如下所示： 

```
DROP INDEX uniqueerror_i_idx;
```

**2.常见错误及解决方案**
错误1. ERROR: duplicate key value violates unique constraint "uniqueerror_i_idx"
原因：创建唯一索引时，发现违反了唯一约束。
修复：解决发生冲突的行。 

错误2. ERROR: Backfilling indexes { timeoutmaster_i_idx } for tablet 42e3857759f54733a47e3bb817636f60 from key '' in state kFailed
原因：服务器端回填超时，重复命中。
修复：执行以下任何一项或全部操作： 
增加BM-Master 标志bsql_index_backfill_rpc_timeout_ms，从60000（1分钟）到300000 （5分钟）。
增加BM-TServer 标志backfill_index_timeout_grace_margin_ms，从-1（1秒）到60000 （1分钟）。
减少BM-TServer 标志backfill_index_write_batch_size ，从128 到 32。

错误3. ERROR: BackfillIndex RPC (request call id 123) to 127.0.0.1:9100 timed out after 86400.000s
原因：客户端回填超时。
修复：在回填过程中，master  leader可能发生了变化。目前不支持此操作。请重试创建索引，并密切关注master  leader。

尝试增加并行性。索引回填在表的每个分片上并行进行。RF-3设置中的一个一分片表不会利用这种并行性。对于范围分区表和共址表，一分片表是默认的。另一方面，无论有多大的并行性，一分片索引都会成为索引回填写入的瓶颈。拆分分片则可以改善分区。
如果回填确实需要更多时间，请将TServer标志backfill_index_client_rpc_timeout_ms增加到回填所需的时间（例如，一周）。 
要在索引回填期间优先考虑保持其他事务的有效性，请将以下各项设置为大于预期的最长事务： 
    BM-Master 标志index_backfill_wait_for_old_txns_ms
    BSQL参数bsql_bm_index_state_flags_update_delay

当您知道不会有联机写入时，要将索引创建速度加快几秒钟，请将BSQL参数bsql_bm_index_state_flags_update_delay设置为零。 


## **CREATE MATERIALIZED VIEW**

定义一个新的物化视图
语法：
create_matview ::= CREATE MATERIALIZED VIEW [ IF NOT EXISTS ]  
                   matview_name [ ( column_name [ , ... ] ) ]  
                   [ WITH ( storage_parameters ) ]  
                   [ TABLESPACE tablespace_name ]  AS subquery 
                   [ WITH [ NO ] DATA ]

描述：创建一个名为matview_name的物化视图。如果指定的数据库中已存在matview_name，则除非使用If NOT exists子句，否则将引发错误。
参数：
Tablespace：用于指定物化视图的表空间。
storage_parameters：

COLOCATION：为要共址的物化视图指定COLOCATION=true。此选项的默认值为false。 

示例：

```
CREATE TABLE t1(a int4, b int4);
INSERT INTO t1 VALUES (2, 4), (3, 4);
CREATE MATERIALIZED VIEW m1 AS SELECT * FROM t1 WHERE a = 3;
SELECT * FROM t1;
```

返回信息如下：

```
 a | b
---+---
 3 | 4
 2 | 4
```

## **CREATE OPERATOR**

定义一个新的操作符
语法：
create_operator ::= CREATE OPERATOR operator_name  ( 
                    { FUNCTION = subprogram_name
                      | PROCEDURE = subprogram_name } 
                    [ , operator_option [ ... ] ] )

operator_option ::= LEFTARG = left_type
                    | RIGHTARG = right_type
                    | COMMUTATOR = com_op
                    | NEGATOR = neg_op
                    | RESTRICT = res_proc
                    | JOIN = join_proc
                    | HASHES
                    | MERGES
描述：CREATE OPERATOR定义一个新的操作符 name。定义操作符的用户会成为该操作符的拥有者。如果给出一个模式名，该操作符将被创建在指定的模式中。否则它会被创建在当前模式中。
操作符名称是最多NAMEDATALEN-1（默认为 63） 个字符的序列，这些字符可以是：+ - * / < > = ~ ! @ # % ^ & | ` ?

参数：
operator_name：要定义的操作符的名称。允许使用的字符请见上文。名称可以被模式限定，例如CREATE OPERATOR myschema.+ (...)。如果没有被模式限定，该操作符将被创建在当前模式中。如果两个同一模式中的操作符在不同的数据类型上操作，它们可以具有相同的名称。这被称为重载。
subprogram_name：用来实现这个操作符的函数。
left_type：这个操作符的左操作数（如果有）的数据类型。忽略这个选项可以表示一个左一元操作符。
right_type：这个操作符的右操作数（如果有）的数据类型。忽略这个选项可以表示一个右一元操作符。
com_op：这个操作符的交换子。
neg_op：这个操作符的求反器。
res_proc：用于这个操作符的限制选择度估计函数。
join_proc：用于这个操作符的连接选择度估算函数。
HASHES：表示这个操作符可以支持哈希连接。
MERGES：表示这个操作符可以支持归并连接。

示例：

```
CREATE OPERATOR @#@ (
             rightarg = int8,
             procedure = numeric_fac
           );
SELECT @#@ 5;
```

## **CREATE OPERATOR CLASS**

定义一个新的操作符类
语法：
create_operator_class ::= CREATE OPERATOR CLASS operator_class_name 
                          [ DEFAULT ] FOR TYPE data_type  USING 
                          index_method AS operator_class_as [ , ... ]

operator_class_as ::= OPERATOR strategy_number operator_name 
                      [ ( operator_signature ) ] [ FOR SEARCH ]
                      | FUNCTION support_number 
                        [ ( op_type [ , ... ] ) ] subprogram_name ( 
                        subprogram_signature )
                      | STORAGE storage_type

描述：定义一个新的操作符类
参数：
operator_class_name：要创建的操作符类的名称。该名称可以被模式限定。
DEFAULT：如果存在，该操作符类将成为其数据类型的默认操作符类。对一种特定的数据类型和索引方法至多有一个默认操作符类。
data_type：这个操作符类所用于的列数据类型。
index_method：这个操作符类所用于的索引方法的名称。
strategy_number：用于一个与该操作符类相关联的操作符的索引方法策略号。
operator_name：一个与该操作符类相关联的操作符的名称（可以被模式限定）。
op_type：在一个OPERATOR子句中，这表示该操作符的操作数数据类型，或者用NONE来表示一个左一元或者右一元操作符。在操作数数据类型与该操作符的数据类型相同的一般情况下，操作数的数据类型可以被省略。
在一个FUNCTION子句中，这表示该函数要支持的操作数数据类型，如果它与该函数的输入数据类型（对于 B-树比较函数和哈希函数）或者操作符类的数据类型（对于 B-树排序支持函数和所有GiST、 SP-GiST、GIN 和 BRIN 操作符类中的函数）不同。这些默认值是正确的，并且op_type因此不必在FUNCTION子句中被指定，对于B-树排序支持函数的情况来说，这表示跨数据类型比较。
support_number：用于一个与该操作符类相关联的函数的索引方法支持函数编号。
subprogram_name：一个用于该操作符类的索引方法支持函数的函数名称（可以是模式限定的）。
storage_type：实际存储在索引中的数据类型。通常这和列数据类型相同，但是有些 索引方法（当前有 GiST、GIN 和 BRIN）允许它们不同。除非索引方法允许使用不同的类型，STORAGE子句必须被省略。 如果data_type列被指定为anyarray，那么storage_type可以被声明为anyelement以指示索引条目是属于为每个特定索引创建的实际数组类型的元素类型的成员。
OPERATOR、FUNCTION和STORAGE 子句可以以任何顺序出现。

示例：

```
CREATE OPERATOR CLASS my_op_class
           FOR TYPE int4
           USING btree AS
           OPERATOR 1 <,
           OPERATOR 2 <=;
```

## **CREATE POLICY**

为一个表定义一条新的行级安全性策略

语法：
create_policy ::= CREATE POLICY name ON table_name 
                  [ AS { PERMISSIVE | RESTRICTIVE } ]  
                  [ FOR { ALL | SELECT | INSERT | UPDATE | DELETE } ] 
                  [ TO { role_name
                         | PUBLIC
                         | CURRENT_USER
                         | SESSION_USER } [ , ... ] ]  
                  [ USING ( using_expression ) ] 
                  [ WITH CHECK ( check_expression ) ]

描述：CREATE POLICY为一个表定义一条行级安全性策略。注意为了应用已被创建的策略，在表上必须启用行级安全性（使用ALTER TABLE ... ENABLE ROW LEVEL SECURITY）。
参数：
name：要创建的策略的名称。
table_name：该策略适用的表的名称（可以被模式限定）。
PERMISSIVE | RESTRICTIVE：指定策略是宽容性的还是限制性的。在将策略应用于表时，宽容性的策略使用逻辑OR运算符组合在一起，而限制策略使用逻辑AND运算符组合。限制性策略用于减少可访问的记录数量。默认为宽容性的。 
role_name：该策略适用的角色。默认是PUBLIC，它将把策略应用到所有的角色。
using_expression：任意的SQL条件表达式（返回 boolean）。该条件表达式不能包含任何聚集或者窗口函数。如果行级安全性被启用，这个表达式将被增加到引用该表的查询。让这个表达式返回真的行将可见。让这个表达式返回假或者空的任何行将对用户不可见（在SELECT中）并且将对修改不可用（ 在UPDATE或DELETE中）。这类行 会被悄悄地禁止而不会报告错误。只有条件返回true的行才会在SELECT中可见，并可在UPDATE或DELETE中进行修改。 
check_expression：任意的SQL条件表达式（返回 boolean），仅用于INSERT和UPDATE查询。该条件表达式不能包含任何聚集或者窗口 函数。如果行级安全性被启用，这个表达式将被用在该表上的 INSERT以及 UPDATE查询中。INSERT或UPDATE中只允许表达式计算结果为true的行。如果任何被插入的记录或者跟新后的记录导致该表达式计算为假或者空，则会抛出一个错误。请注意，与using_expression不同，check_expression 是根据行中建议的新内容而不是原始内容进行评估的。

示例：
示例1：创建一个宽容性策略。 

```
CREATE POLICY p1 ON document
  USING (dlevel <= (SELECT level FROM user_account WHERE user = current_user));
```

示例2：创建一个限制性策略。

```
CREATE POLICY p_restrictive ON document AS RESTRICTIVE TO user_bob
    USING (cid <> 44);
```

示例3：为插入创建具有CHECK条件的策略。

```
CREATE POLICY p2 ON document FOR INSERT WITH CHECK (dauthor = current_user);
```

## **CREATE PROCEDURE**

定义一个新的存储过程

语法：
create_procedure ::= CREATE [ OR REPLACE ] PROCEDURE subprogram_name ( 
                     [ arg_decl_with_dflt [ , ... ] ] )  
                     { unalterable_proc_attribute
                       | alterable_fn_and_proc_attribute } [ ... ]

arg_decl_with_dflt ::= arg_decl [ { DEFAULT | = } expression ]

arg_decl ::= [ formal_arg ] [ arg_mode ] arg_type

subprogram_signature ::= arg_decl [ , ... ]

unalterable_proc_attribute ::= LANGUAGE lang_name
                               | AS subprogram_implementation

lang_name ::= SQL | PLPGSQL | C

subprogram_implementation ::= ' sql_stmt_list '
                              | ' plpgsql_block_stmt '
                              | ' obj_file ' [ , ' link_symbol ' ]

sql_stmt_list ::= sql_stmt ; [ sql_stmt ... ]

alterable_fn_and_proc_attribute ::= SET run_time_parameter 
                                    { TO value
                                      | = value
                                      | FROM CURRENT }
                                    | RESET run_time_parameter
                                    | RESET ALL
                                    | [ EXTERNAL ] SECURITY 
                                      { INVOKER | DEFINER }

描述：定义一个新的存储过程，存储过程和表等其他模式对象一样，不可避免地有一个所有者。创建存储过程时，不能显式指定所有者。相反，它被隐式定义为current_user内置函数在创建过程的会话中被调用时返回的内容。此用户必须对存储过程的架构及其参数数据类型具有使用权限。具有不同参数类型的过程可以共享同一个名称（这被称为重载）。
要替换一个已有存储过程的当前定义，请使用CREATE OR REPLACE PROCEDURE。不能用这种方式更改存储过程的名称或者参数类型（如果尝试这样做，实际上会创建一个新的、不同的存储过程）。
当CREATE OR REPLACE PROCEDURE被用来替换一个现有的存储过程时，该存储过程的拥有关系和权限保持不变。所有其他的存储过程属性会被赋予这个命令中指定的或者暗示的值。必须拥有（包括成为拥有角色的成员）该存储过程才能替换它。
创建存储过程的用户将成为该存储过程的拥有者。
为了能够创建一个存储过程，用户必须具有参数类型上的USAGE特权。

示例：

创建accounts表，及插入一些数据。

```
create schema s;
 
create table s.accounts (
  id integer primary key,
  name text not null,
  balance decimal(15,2) not null);
 
insert into s.accounts values (1, 'Jane', 100.00);
insert into s.accounts values (2, 'John', 50.00);
 
select * from s.accounts order by 1;
```

定义将资金从一个帐户转移到另一个帐户的存储过程。

```
create or replace procedure s.transfer(from_id in int, to_id in int, amnt in decimal)
  security definer
  set search_path = pg_catalog, pg_temp
  language plpgsql
as $body$
begin
  if amnt <= 0.00 then
    raise exception 'The transfer amount must be positive';
  end if;
  if from_id = to_id then
    raise exception 'Sender and receiver cannot be the same';
  end if;
  update s.accounts set balance = balance - amnt where id = from_id;
  update s.accounts set balance = balance + amnt where id = to_id;
end;
$body$;
 
Jane转账$20.00给John：
call s.transfer(1, 2, 20.00);
select * from s.accounts order by 1;
```

返回信息如下：

```
 id | name | balance
----+------+---------
  1 | Jane |   80.00
  2 | John |   70.00
(2 rows)
```

不受支持的参数值将会引发错误：

```
CALL s.transfer(2, 2, 20.00);
```

错误信息如下：

```
ERROR:  Sender and receiver cannot be the same
```

```
call s.transfer(1, 2, -20.00);
```

错误信息如下：

```
ERROR:  The transfer amount must be positive
```

## **CREATE ROLE**

定义一个新的数据库角色

语法：
create_role ::= CREATE ROLE role_name 
                [ [ WITH ] role_option [ , ... ] ]

role_option ::= SUPERUSER
                | NOSUPERUSER
                | CREATEDB
                | NOCREATEDB
                | CREATEROLE
                | NOCREATEROLE
                | INHERIT
                | NOINHERIT
                | LOGIN
                | NOLOGIN
                | CONNECTION LIMIT connlimit
                | [ ENCRYPTED ] PASSWORD  ' password ' 
                | PASSWORD NULL
                | VALID UNTIL  ' timestamp ' 
                | IN ROLE role_name [ , ... ]
                | IN GROUP role_name [ , ... ]
                | ROLE role_name [ , ... ]
                | ADMIN role_name [ , ... ]
                | USER role_name [ , ... ]
                | SYSID uid

描述：定义一个新的数据库角色
使用CREATE ROLE语句将角色添加到AiSQL数据库集群中。角色是一个可以拥有数据库对象并具有数据库权限的实体。角色可以是用户或组，具体取决于其使用方式。具有LOGIN的角色可以被视为“用户”。您必须具有CREATE ROLE权限或是数据库超级用户才能使用此命令。
请注意，角色是在BSQL集群级别定义的，因此在集群中的所有数据库中都是有效的。
可以使用GRANT/REVOKE命令设置/删除角色的权限。

参数：
role_name：新角色的名称。
SUPERUSER，NOSUPERUSER：确定新角色是否为“超级用户”。超级用户可以覆盖所有访问限制，应谨慎使用。只有具有SUPERUSER权限的角色才能创建其他SUPERUSER角色。如果未指定，则默认为NOSUPERUSER。
CREATEDB，NOCREATEDB：确定新角色是否可以创建数据库。默认为NOCREATEDB。
CREATEROLE，NOCREATEROLE：确定新角色是否可以创建其他角色。默认值为NOCREATEROLE。 
INHERIT，NOINHERIT：确定新角色是否继承其所属角色的特权。如果没有INHERIT，另一个角色的成员资格只能授予该角色设置角色的能力。其他角色的权限只有在完成后才可用。如果未指定，则默认为INHERIT。
LOGIN，NOLOGIN：确定是否允许新角色登录。只有具有登录权限的角色才能在客户端连接期间使用。具有LOGIN的角色可以被视为用户。如果未指定，则默认为NOLOGIN。请注意，如果使用CREATE USER语句而不是CREATE ROLE，则默认值为LOGIN。
CONNECTION LIMIT：指定角色可以进行的并发连接数。默认值为-1，表示不受限制。这仅适用于可以登录的角色。
[ENCRYPTED] PASSWORD：设置新角色的密码。这只适用于可以登录的角色。如果未指定密码，则密码将设置为null，并且该用户的密码身份验证将始终失败。请注意，密码始终加密存储在系统目录中，可选关键字encrypted仅用于与PostgreSQL兼容。 
VALID UNTIL：设置角色密码不再有效的日期和时间。如果省略此子句，密码将始终有效。 
IN ROLE ROLE_name，IN GROUP ROLE_name：列出一个或多个现有角色，新角色将作为新成员立即添加到这些角色中。（请注意，没有以管理员身份添加新角色的选项；请使用单独的GRANT命令来执行此操作。） 
ROLE role_name, USER role_name：列出一个或多个自动添加为新角色成员的现有角色。 
ADMIN role_name：与role role_name类似，但已命名的角色将添加到新角色WITH ADMIN OPTION中，使他们有权将此角色的成员资格授予其他人。
SYSID uid：仅为了与Postgres兼容，SYSID uid被忽略。 
示例：
示例1：创建一个可以登录的角色。

```
CREATE ROLE John LOGIN; 
```

示例2：创建一个可以登录并具有密码的角色。 

```
CREATE ROLE Jane LOGIN PASSWORD 'password';
```

示例3：创建一个可以管理数据库和角色的角色。

```
CREATE ROLE SysAdmin CREATEDB CREATEROLE;
```

## **CREATE RULE**

定义一条新的重写规则

语法：
create_rule ::= CREATE [ OR REPLACE ] RULE rule_name AS ON rule_event 
                TO table_name  [ WHERE boolean_expression ] DO 
                [ ALSO | INSTEAD ] { NOTHING
                                     | command
                                     | ( command [ ; ... ] ) }

rule_event ::= SELECT | INSERT | UPDATE | DELETE

command ::= SELECT | INSERT | UPDATE | DELETE | NOTIFY

描述：CREATE RULE定义一条应用于指定表或视图的新规则。
参数：
rule_name：要创建的规则的名称。它必须与同一个表上任何其他规则的名称相区分。 同一个表上同一种事件类型的多条规则会按照其名称的字符顺序被应用。
rule_event：是SELECT、 INSERT、UPDATE或者 DELETE之一。 注意包含ON CONFLICT子句的INSERT 不能被用在具有INSERT或者 UPDATE规则的表上。那种情况下请考虑使用可更新的视图。
table_name：规则适用的表或者视图的名称（可以是模式限定的）。
boolean_expression：任意的SQL条件表达式（返回 boolean）。该条件表达式不能引用除NEW以及 OLD之外的任何表，并且不能包含聚集函数。
INSTEAD：INSTEAD指示该命令应该取代原始命令被执行。
ALSO：ALSO指示应该在原始命令 之外执行这些命令。
如果ALSO和INSTEAD都没有被指定， 默认是ALSO。
command：组成规则动作的命令。可用的命令有SELECT、 INSERT、UPDATE、 DELETE或者NOTIFY。

示例：

```
CREATE TABLE t1(a int4, b int4);
CREATE TABLE t2(a int4, b int4);
CREATE RULE t1_to_t2 AS ON INSERT TO t1 DO INSTEAD
             INSERT INTO t2 VALUES (new.a, new.b);
INSERT INTO t1 VALUES (3, 4);
SELECT * FROM t1;
```

返回信息如下：

```
 a | b
---+---
(0 rows)
SELECT * FROM t2;
```

返回信息如下：

```
 a | b 
---+---
 3 | 4
(1 rows)
```

## **CREATE SCHEMA**

定义一个新模式

语法：
create_schema_name ::= CREATE SCHEMA [ IF NOT EXISTS ] schema_name 
                       [ AUTHORIZATION role_specification ]

create_schema_role ::= CREATE SCHEMA [ IF NOT EXISTS ] AUTHORIZATION 
                       role_specification

role_specification ::= role_name | CURRENT_USER | SESSION_USER

描述：定义一个新模式
使用CREATE SCHEMA语句在当前数据库中创建模式。模式本质上是一个命名空间：它包含命名对象（表、数据类型、函数和运算符），这些对象的名称可以与其他模式中的对象的名称重复。可以使用模式名称作为前缀或在搜索路径中设置模式名称来访问模式中的命名对象。
参数：
schema_name：创建的架构的名称。如果未指定schema_name，则使用角色名称。架构名称不得以pg_开头。尝试使用这样的名称创建模式，或将现有模式重命名为具有这样的名称，都会导致错误。
role_name：是将拥有新模式的角色。如果省略，则默认为执行命令的用户。若要创建另一个角色拥有的架构，您必须是该角色的直接或间接成员，或者是超级用户。

示例：
示例1：创建一个模式 

```
CREATE SCHEMA IF NOT EXISTS branch;
```

示例2：为用户创建模式 

```
CREATE ROLE John;
CREATE SCHEMA AUTHORIZATION john;
```

示例3：创建一个将由另一个角色拥有的模式

```
CREATE SCHEMA branch AUTHORIZATION john;
```

## **CREATE SEQUENCE**

定义一个新的序列发生器
语法：
create_sequence ::= CREATE [ TEMPORARY | TEMP ] SEQUENCE 
                    [ IF NOT EXISTS ] sequence_name sequence_options

sequence_name ::= qualified_name

sequence_options ::= [ INCREMENT [ BY ] int_literal ]  
                     [ MINVALUE int_literal | NO MINVALUE ] 
                     [ MAXVALUE int_literal | NO MAXVALUE ] 
                     [ START [ WITH ] int_literal ]  
                     [ CACHE positive_int_literal ] [ [ NO ] CYCLE ]

描述：定义一个新的序列发生器。指定序列的名称（sequence_name）。如果当前模式中已存在具有该名称的序列，并且未指定if NOT exists，则会引发错误。

参数：
TEMPORARY | TEMP：使用此限定符将创建一个临时序列。临时序列仅在创建它们的当前客户端会话中可见，并在会话结束时自动删除。
INCREMENT BY int_literal：指定要添加到当前序列值以创建新值的增量值。默认值为1。
MINVALUE int_literal | NO MINVALUE：指定序列中允许的最小值。如果达到这个值（在负增量的序列中），nextval（）将返回一个错误。如果未指定MINVALUE，则将使用默认值。默认值为1。
MAXVALUE int_literal | NO MAXVALUE：指定序列中允许的最大值。如果达到该值，nextval（）将返回一个错误。如果未指定MAXVALUE，则将使用默认值。默认值为2⁶³-1。
START WITH int_literal：指定序列中的第一个值。start不能小于minvalue。默认值为1。
CACHE int_literal：指定要在客户端中缓存序列中的数字数量。默认值为100。
当TServer bsql_sequence_cache_minval配置标志未显式关闭（设置为0）时，将使用该标志和缓存子句的最大值。
[ NO ] CYCLE：如果指定了CYCLE，则一旦达到minvalue或maxvalue，序列就会回卷。如果达到了maxvalue，则minvalue将是序列中的下一个数字。如果达到minvalue（对于递减序列），maxvalue将是序列中的下一个数字。NO CYCLE（无循环）是默认值。
Cache：
在BSQL中，就像在PostgreSQL中一样，序列的数据存储在一个持久系统表中。在BSQL中，该表每个序列有一行，并以两个值存储序列数据：
last_val：存储上次使用的值或下一个要使用的值 
is_called：存储是否已使用的last_val。如果为false，last_val是序列中的下一个值。否则，last_val+INCREMENT为下一个值。
默认情况下（当INCREMENT为1时），每次调用nextval()都会更新该序列的last_val。在BSQL中，保存序列数据的表被复制，而不是在本地文件系统中。对该表的每次更新都需要两个RPC（将来将优化为一个RPC）。在任何情况下，在BSQL中调用nextval()所经历的延迟都将显著高于Postgres中的相同操作。为了避免这种性能下降，AiSQL建议使用一个足够大的缓存值。缓存的值存储在本地节点的内存中，检索这些值可以避免任何RPC，因此一个缓存分配的延迟可以分摊到为缓存分配的所有数字上。
SERIAL类型创建一个具有默认值为1的缓存的序列。因此，应该避免使用SERIAL类型，并且应该使用它们的等效语句。不要像这样创建SERIAL类型的表： 

```
CREATE TABLE t(k SERIAL)
```

您应该首先创建一个具有足够大缓存的序列，然后将要具有串行类型的列设置为DEFAULT，以设置序列的nextval()。

```
CREATE SEQUENCE t_k_seq CACHE 10000;
CREATE TABLE t(k integer NOT NULL DEFAULT nextval('t_k_seq'));
```

示例：

示例1：创建一个简单序列，每次调用nextval()时递增1。

```
CREATE SEQUENCE s;
```

示例2：创建一个具有10000个值的缓存的序列。 

```
CREATE SEQUENCE s2 CACHE 10000;
```

在同一个会话中，执行

```
SELECT nextval('s2');
```

返回信息如下：

```
 nextval
---------
       1
```


在不同的会话中，执行

```
SELECT nextval('s2');
```

返回信息如下：

```
 nextval
---------
   10001
```

示例3：创建一个从0开始的序列。MINVALUE从默认值1更改为小于或等于0。 

```
CREATE SEQUENCE s3 START 0 MINVALUE 0;
```

## **CREATE SERVER**

定义一个新的外部服务器
语法：
create_server ::= CREATE SERVER [ IF NOT EXISTS ] server_name  
                  [ TYPE server_type ] [ VERSION server_version ]  
                  FOREIGN DATA WRAPPER fdw_name 
                  [ OPTIONS ( fdw_options ) ]

描述：CREATE SERVER定义一个新的外部服务器。 定义该服务器的用户会成为拥有者。外部服务器通常包装了外部数据包装器用来访问一个外部数据源所需的连接信息。额外的用户相关的连接信息可以通过用户映射的方式来指定。
服务器名称在数据库中必须唯一。
创建服务器要求所使用的外部数据包装器上的USAGE特权。
如果指定的数据库中已存在server_name，则除非使用If NOT exists子句，否则将引发错误。

参数：
Type：可选的服务器类型。
Server Version：可选的服务器版本。
FDW name：FOREIGN DATA WRAPPER子句可用于指定外部数据包装的名称。
Options：OPTIONS子句指定外部服务器的选项。它们通常定义服务器的连接详细信息，但实际允许的选项名称和值特定于服务器的外部数据包装器。 
示例：

```
CREATE SERVER my_server FOREIGN DATA WRAPPER postgres_fdw OPTIONS (host '187.51.62.1');
```

## **CREATE TABLE**

定义一个新表。

语法：
create_table ::= CREATE [ TEMPORARY | TEMP ] TABLE [ IF NOT EXISTS ] 
                 table_name ( [ table_elem [ , ... ] ] )  
                 [ WITH ( { COLOCATION = { 'true' | 'false' }
                            | storage_parameters } )
                   | WITHOUT OIDS ]  [ TABLESPACE tablespace_name ] 
                 [ SPLIT { INTO positive_int_literal TABLETS
                           | AT VALUES ( split_row [ , ... ] ) } ]

table_elem ::= column_name data_type [ column_constraint [ ... ] ]
               | table_constraint

column_constraint ::= [ CONSTRAINT constraint_name ] 
                      { NOT NULL
                        | NULL
                        | CHECK ( expression )
                        | DEFAULT expression
                        | UNIQUE index_parameters
                        | PRIMARY KEY
                        | references_clause }  
                      [ DEFERRABLE | NOT DEFERRABLE ] 
                      [ INITIALLY DEFERRED | INITIALLY IMMEDIATE ]

table_constraint ::= [ CONSTRAINT constraint_name ] 
                     { CHECK ( expression )
                       | UNIQUE ( column_names ) index_parameters
                       | PRIMARY KEY ( key_columns )
                       | FOREIGN KEY ( column_names ) 
                         references_clause }  
                     [ DEFERRABLE | NOT DEFERRABLE ] 
                     [ INITIALLY DEFERRED | INITIALLY IMMEDIATE ]

key_columns ::= hash_columns [ , range_columns ] | range_columns

hash_columns ::= column_name [ HASH ] | ( column_name [ , ... ] ) HASH

range_columns ::= { column_name { ASC | DESC } } [ , ... ]

storage_parameters ::= storage_parameter [ , ... ]

storage_parameter ::= param_name [ = param_value ]

index_parameters ::= [ INCLUDE ( column_names ) ] 
                     [ WITH ( storage_parameters ) ]  
                     [ USING INDEX TABLESPACE tablespace_name ]

references_clause ::= REFERENCES table_name [ column_name [ , ... ] ] 
                      [ MATCH FULL | MATCH PARTIAL | MATCH SIMPLE ]  
                      [ ON DELETE key_action ] 
                      [ ON UPDATE key_action ]

split_row ::= ( column_value [ , ... ] )
描述：CREATE TABLE将在当前数据库中创建一个新的、初始为空的表。
**参数：**
TEMPORARY 或TEMP：如果指定，该表被创建为一个临时表。临时表会被在会话结束时自动被删除，或者也可以选择在当前事务结束时删除。当临时表存在时，已有的同名持久表将对于当前会话不可见，不过可以使用模式限定的名称进行引用。在一个临时表上创建的任何索引也自动地变为临时的。
IF NOT EXISTS：如果一个同名表已经存在，不要抛出一个错误。在这种情况下会发出一个提示。注意这不保证现有的关系是和将要被创建的表相似的东西。
table_name：要被创建的表名（可以选择用模式限定）。
column_name：列的名称。
data_type：列的数据类型。
CONSTRAINT constraint_name：一个列约束或表约束的可选名称。如果该约束被违背，约束名将会出现在错误消息中。如果没有指定约束名，系统将生成一个。
NOT NULL：该列不允许包含空值。
NULL：该列允许包含空值。这是默认情况。
CHECK ( expression )：CHECK指定一个产生布尔结果的表达式，一个插入或更新操作要想成功，其中新的或被更新的行必须满足该表达式。计算出 TRUE 或 UNKNOWN 的表达式就会成功。只要任何一个插入或更新操作的行产生了 FALSE 结果，将报告一个错误异常并且插入或更新不会修改数据库。一个被作为列约束指定的检查约束只应该引用该列的值，而一个出现在表约束中的表达式可以引用多列。

DEFAULT expression：DEFAULT子句为出现在其定义中的列赋予一个默认数据。该值是可以使用变量的表达式（特别是，不允许用对其他列的交叉引用）。子查询也是不允许的。 默认值表达式的数据类型必须匹配列的数据类型。
默认值表达式将被用在任何没有为该列指定值的插入操作中。如果一列没有默认值，那么默认值为空值。
UNIQUE （列约束） |  UNIQUE ( column_names ) index_parameters (表约束)：UNIQUE约束指定一个表中的一列或多列组成的组包含唯一的值。唯一表约束的行为与列约束的行为相同，只是表约束能够跨越多列。
对于一个唯一约束的目的来说，空值不被认为是相等的。
每一个唯一表约束必须命名一个列的集合，并且它与该表上任何其他唯一或主键约束所命名的列集合都不相同。
PRIMARY KEY （列约束） |  PRIMARY KEY ( key_columns ) (表约束)：PRIMARY KEY约束指定表的一个或者多个列只能包含唯一（不重复）、非空的值。一个表上只能指定一个主键，可以作为列约束或表约束。
references_clause （列约束） |  FOREIGN KEY ( column_names ) references_clause(表约束)：这些子句指定一个外键约束，它要求新表的一列或一个列的组必须只包含能匹配被引用表的某个行在被引用列上的值。 如果refcolumn列表被忽略，将使用reftable的主键。 被引用列必须是被引用表中一个非可延迟唯一约束或主键约束的列。 用户必须在被引用的表（或整个表,或特定的引用列）上拥有REFERENCES权限。 增加的外键约束需要SHARE ROW EXCLUSIVE 锁定引用的表。 注意外键约束不能在临时表和永久表之间定义。 此外请注意，虽然可以在分区表上定义外键，但不能够声明引用分区表的外键。
被插入到引用列的一个值会使用给定的匹配类型与被引用表的值进行匹配。 有三种匹配类型：MATCH FULL、MATCH PARTIAL以及MATCH SIMPLE（这是默认值）。 MATCH FULL将不允许一个多列外键中的一列为空，除非所有外键列都是空；如果它们都是空，则不要求该行在被引用表中有一个匹配。 MATCH SIMPLE允许任意外键列为空，如果任一为空，则不要求该行在被引用表中有一个匹配。 MATCH PARTIAL现在还没有被实现。
另外，当被引用列中的数据被改变时，在这个表的列中的数据上可以执行特定的动作。ON DELETE指定当被引用表中一个被引用行被删除时要执行的动作。同样，ON UPDATE指定当被引用表中一个被引用列被更新为新值时要执行的动作。如果该行被更新，但是被引用列并没有被实际改变，不会做任何动作。除了NO ACTION检查之外的引用动作不能被延迟，即便该约束被声明为可延迟的。对每一个子句可能有以下动作：
NO ACTION：产生一个错误指示删除或更新将会导致一个外键约束违背。如果该约束被延迟，并且仍存在引用行，这个错误将在约束检查时被产生。这是默认动作。
RESTRICT：产生一个错误指示删除或更新将会导致一个外键约束违背。这个动作与NO ACTION形同，不过该检查不是可延迟的。
CASCADE：删除任何引用被删除行的行，或者把引用列的值更新为被引用列的新值。
SET NULL：将引用列设置为空。
SET DEFAULT：设置引用列为它们的默认值（如果该默认值非空，在被引用表中必须有一行匹配该默认值，否则该操作将会失败）。

DEFERRABLE  |  NOT DEFERRABLE：这个子句控制该约束是否能被延迟。一个不可延迟的约束将在每一次命令后立刻被检查。可延迟约束的检查将被推迟到事务结束时进行（使用SET CONSTRAINTS命令）。NOT DEFERRABLE是默认值。当前，只有UNIQUE、PRIMARY KEY、EXCLUDE以及REFERENCES（外键）约束接受这个子句。NOT NULL以及CHECK约束是不可延迟的。注意在包括ON CONFLICT DO UPDATE子句的INSERT语句中，可延迟约束不能被用作冲突裁判者。
INITIALLY IMMEDIATE  |  INITIALLY DEFERRED：如果一个约束是可延迟的，这个子句指定检查该约束的默认时间。如果该约束是INITIALLY IMMEDIATE，它会在每一个语句之后被检查。这是默认值。如果该约束是INITIALLY DEFERRED，它只会在事务结束时被检查。约束检查时间可以用SET CONSTRAINTS命令修改。

WITH ( { COLOCATION = { 'true' | 'false' }：指定为true，则此表应在单一分片中共址。具体细节，请参阅[共址表](#_共址表（colocated table）)，了解共址表的详细信息。
要创建共址表，请使用以下命令： 
CREATE TABLE <name> (columns) WITH (COLOCATION = true);
在共址数据库中，默认情况下，所有表都是共址的。如果需要创建非共址的表，请使用以下命令：
CREATE TABLE <name> (columns) WITH (COLOCATION = false);
这样可以确保该表不会与该数据库的其他表存储在同一个分片上，而是有自己的一组分片。
如果表所属的数据库是非共址的，则设置COLOCATION=true无效，因为当前仅在数据库级别支持共址。有关详细信息，请参阅[共址表](#_共址表（colocated table）)。 
WITHOUT OIDS：与WITH (OIDS=FALSE)等效，新表不会存储 OID 并且对插入其中的一个新行不会分配 OID。
TABLESPACE tablespace_name：指定描述此表的放置配置的表空间的名称。默认情况下，表被放置在pg_default表空间中，该表空间将表的tablet均匀地分布在集群中。
SPLIT INTO：对于哈希分片表，可以使用SPLIT INTO子句指定要为该表创建的分片的数量。然后将散列范围均匀地分布在这些分片上。  
使用SPLIT INTO的预拆分分片，在生产集群上分发读写工作负载。例如，如果您有3台服务器，那么将表拆分为30个分片，可以提供表上的写入吞吐量。
注意：
默认情况下，AiSQL预拆分一个表为bsql_num_shards_per_tserver*num_of_tserver个分片。SPLIT INTO子句可用于在每个表的基础上重写该设置。
SPLIT AT VALUES：对于范围分片表，可以使用SPLIT AT VALUES子句设置拆分点，来预拆分范围分片的表。

例如：

```
CREATE TABLE tbl(
  a int,
  b int,
  primary key(a asc, b desc)
) SPLIT AT VALUES((100), (200), (200, 5));
```

在上面的示例中，存在3个分片点，因此，表被拆分为如下4个分片：
    tablet 1: a=<lowest>, b=<lowest> to a=100, b=<lowest>
    tablet 2: a=100, b=<lowest> to a=200, b=<lowest>
    tablet 3: a=200, b=<lowest> to a=200, b=5
    tablet 4: a=200, b=5 to a=<highest>, b=<highest>
Storage parameters：PostgreSQL定义的存储参数将被忽略，并且仅用于与PostgreSQL兼容。

以下是一些示例：
创建带有主键的表：

```
CREATE TABLE sample(k1 int, 
k2 int, 
v1 int, 
v2 text, 
PRIMARY KEY (k1, k2) );
```

在该示例中，第一列k1将是HASH，而第二列k2将是ASC。

```
bigmath=# \d sample
               Table "public.sample"
 Column |  Type   | Collation | Nullable | Default
--------+---------+-----------+----------+---------
 k1     | integer |           | not null |
 k2     | integer |          | not null |
 v1     | integer |          |          |
 v2     | text    |          |          |
Indexes:
    "sample_pkey" PRIMARY KEY, lsm (k1 HASH, k2)
```

创建带范围主键的表：

```
CREATE TABLE range(k1 int,
 k2 int, 
v1 int, 
v2 text, 
PRIMARY KEY (k1 ASC, k2 DESC));
```

创建带有检查约束的表：

```
CREATE TABLE student_grade(student_id int, 
class_id int, 
term_id int, 
grade int CHECK (grade >= 0 AND grade <= 10),
PRIMARY KEY (student_id, class_id, term_id));
```

创建具有默认值的表：

```
CREATE TABLE cars(id int PRIMARY KEY, 
brand text CHECK (brand in ('X', 'Y', 'Z')), 
model text NOT NULL, 
color text NOT NULL DEFAULT 'WHITE' CHECK (color in ('RED', 'WHITE', 'BLUE')));
```

创建具有外键约束的表：

```
CREATE TABLE products(id int PRIMARY KEY,
descr text);
CREATE TABLE orders(id int PRIMARY KEY,
 pid int REFERENCES products(id) ON DELETE CASCADE,
                     amount int);
```

创建具有唯一约束的表：

```
CREATE TABLE translations(message_id int UNIQUE,
message_txt text);
```

创建一个指定分片数量的表：
要指定表的分片数量，可以将CREATE TABLE 语句与SPLIT INTO子句一起使用。

```
CREATE TABLE tracking (id int PRIMARY KEY) SPLIT INTO 10 TABLETS;
```

创建一个共址表：

```
CREATE DATABASE company WITH COLOCATION = true;
```

```
CREATE TABLE employee(id INT PRIMARY KEY, 
name TEXT) WITH (COLOCATION = false);
```

在本例中，数据库company是共址的，意味着，employee表以外的所有表都存储在一个分片上。

## **CREATE TABLE AS**

从一个查询的结果创建一个新表。
语法：
create_table_as ::= CREATE [ TEMPORARY | TEMP ] TABLE 
                    [ IF NOT EXISTS ]  table_name 
                    [ ( column_name [ , ... ] ) ]  AS subquery 
                    [ WITH [ NO ] DATA ]

描述：从一个查询的结果创建一个新表。

参数：
table_name：指定表名。
( column_name [ , ... ] )：指定新表中列的名称。如果未指定，则列名取自查询的输出列名。
WITH [ NO ] DATA：这个子句指定查询产生的数据是否应该被复制到新表中。如果不是，则只有表结构会被复制。默认是复制数据。
TEMPORARY | TEMP：使用此限定符将创建一个临时表。临时表仅在创建它们的当前客户端会话中可见，并在会话结束时自动删除。在临时表上创建的任何索引也是临时的。 
示例：
创建一个表：

```
CREATE TABLE sample(k1 int, k2 int, v1 int, v2 text, PRIMARY KEY (k1, k2));
```

插入一些数据行：

```
INSERT INTO sample VALUES (1, 2.0, 3, 'a'), (2, 3.0, 4, 'b'), (3, 4.0, 5, 'c');
```

创建一个示例表：

```
CREATE TABLE selective_sample AS SELECT * FROM sample WHERE k1 > 1;
```

从上面创建的示例表中查询数据：

```
SELECT * FROM selective_sample ORDER BY k1;
```

## **CREATE TRIGGER**

定义一个新触发器

语法：
create_trigger ::= CREATE TRIGGER name { BEFORE | AFTER | INSTEAD OF } 
                   { event [ OR ... ] } ON table_name 
                   [ FROM table_name ]  [ NOT DEFERRABLE ] 
                   [ FOR [ EACH ] { ROW | STATEMENT } ] 
                   [ WHEN ( boolean_expression ) ]  EXECUTE 
                   { FUNCTION | PROCEDURE } subprogram_name ( 
                   [ subprogram_signature ] )

event ::= INSERT
          | UPDATE [ OF column_name [ , ... ] ]
          | DELETE
          | TRUNCATE

描述：CREATE TRIGGER创建一个新触发器。该触发器将被关联到指定的表、视图或者外部表并且在表上发生特定操作时将执行指定的函数function_name。
WHEN条件可用于指定是否应触发触发器。对于低级触发器，它可以引用行的列的旧值和/或新值。可以为同一事件定义多个触发器，在这种情况下，它们将按照名称的字母表顺序被触发。
示例：
设置一个带有触发器的表，用于跟踪修改时间和用户(角色）。使用预先安装的扩展插件insert_username和moddatetime。

```
CREATE EXTENSION insert_username;
CREATE EXTENSION moddatetime;
 
CREATE TABLE posts (
  id int primary key,
  content text,
  username text not null,
  moddate timestamp DEFAULT CURRENT_TIMESTAMP NOT NULL
);
 
CREATE TRIGGER insert_usernames
   BEFORE INSERT OR UPDATE ON posts
   FOR EACH ROW
   EXECUTE PROCEDURE insert_username (username);
 
CREATE TRIGGER update_moddatetime
   BEFORE UPDATE ON posts
   FOR EACH ROW
   EXECUTE PROCEDURE moddatetime (moddate);
```

插入一些行。对于每个插入，触发器应该将当前角色设置为username，将当前时间戳设置为moddate。 

```
SET ROLE bigmath;
INSERT INTO posts VALUES(1, 'desc1');
 
SET ROLE postgres;
INSERT INTO posts VALUES(2, 'desc2');
INSERT INTO posts VALUES(3, 'desc3');
 
SET ROLE bigmath;
INSERT INTO posts VALUES(4, 'desc4');
 
SELECT * FROM posts ORDER BY id;
```

返回信息如下：

```
 id | content | username |          moddate           
----+---------+----------+----------------------------
  1 | desc1   | bigmath  | 2023-11-06 15:27:21.623908
  2 | desc2   | postgres | 2023-11-06 15:27:21.671353
  3 | desc3   | postgres | 2023-11-06 15:27:21.687268
  4 | desc4   | bigmath  | 2023-11-06 15:27:21.735876
```

更新一些行。对于每次更新，触发器都应该相应地设置username和moddate。 

```
UPDATE posts SET content = 'desc1_updated' WHERE id = 1;
UPDATE posts SET content = 'desc3_updated' WHERE id = 3;
 
SELECT * FROM posts ORDER BY id;
```

返回信息如下：

```
 id |    content    | username |          moddate           
----+---------------+----------+----------------------------
  1 | desc1_updated | bigmath  | 2023-11-06 15:51:41.388021
  2 | desc2         | postgres | 2023-11-06 15:27:21.671353
  3 | desc3_updated | bigmath  | 2023-11-06 15:51:41.446264
  4 | desc4         | bigmath  | 2023-11-06 15:27:21.735876
```

 

## **CREATE TYPE**

定义一种新的数据类型
语法：
create_type ::= create_composite_type
                | create_enum_type
                | create_range_type
                | create_shell_type
                | create_base_type

create_composite_type ::= CREATE TYPE type_name AS ( 
                          [ composite_type_elem [ , ... ] ] )

create_enum_type ::= CREATE TYPE type_name AS ENUM ( 
                     [ name [ , ... ] ] )

create_range_type ::= CREATE TYPE type_name AS RANGE  ( SUBTYPE = 
                      subtype [ , range_type_option [ ... ] ] )

create_shell_type ::= CREATE TYPE type_name

create_base_type ::= CREATE TYPE type_name (  INPUT = input_function , 
                      OUTPUT = output_function 
                     [ , base_type_option [ ... ] ]  )

composite_type_elem ::= attribute_name data_type [ COLLATE collation ]

range_type_option ::= SUBTYPE_OPCLASS = subtype_operator_class
                      | COLLATION = collation
                      | CANONICAL = canonical_function
                      | SUBTYPE_DIFF = subtype_diff_function

base_type_option ::= RECEIVE = receive_function
                     | SEND = send_function
                     | TYPMOD_IN = type_modifier_input_function
                     | TYPMOD_OUT = type_modifier_output_function
                     | INTERNALLENGTH = { internallength | VARIABLE }
                     | PASSEDBYVALUE
                     | ALIGNMENT = alignment
                     | STORAGE = storage
                     | LIKE = like_type
                     | CATEGORY = category
                     | PREFERRED = { TRUE | FALSE }
                     | DEFAULT = default_type_value
                     | ELEMENT = element
                     | DELIMITER = delimiter
                     | COLLATABLE = { TRUE | FALSE }

描述：使用CREATE TYPE语句在数据库中创建用户定义的类型。有五种类型：composite、enumerated、range、base和shell。每个都有自己的CREATE TYPE语法。
示例：
示例1：composite 

```
CREATE TYPE feature_struct AS (id INTEGER, name TEXT);
CREATE TABLE feature_tab_struct (feature_col feature_struct);
```

示例2：Enumerated

```
CREATE TYPE feature_enum AS ENUM ('one', 'two', 'three');
CREATE TABLE feature_tab_enum (feature_col feature_enum);
```

示例3：Range 

```
CREATE TYPE feature_range AS RANGE (subtype=INTEGER);
CREATE TABLE feature_tab_range (feature_col feature_range);
```

示例4：

```
CREATE TYPE int4_type;
CREATE FUNCTION int4_type_in(cstring) RETURNS int4_type
               LANGUAGE internal IMMUTABLE STRICT PARALLEL SAFE AS 'int4in';
CREATE FUNCTION int4_type_out(int4_type) RETURNS cstring
               LANGUAGE internal IMMUTABLE STRICT PARALLEL SAFE AS 'int4out';
CREATE TYPE int4_type (
               INPUT = int4_type_in,
               OUTPUT = int4_type_out,
               LIKE = int4
           );
CREATE TABLE int4_table (t int4_type);
```

示例5：Shell

```
CREATE TYPE shell_type;
```

## **CREATE USER**

定义一个新的数据库角色
语法：
create_user ::= CREATE USER role_name 
                [ [ WITH ] role_option [ , ... ] ]

role_option ::= SUPERUSER
                | NOSUPERUSER
                | CREATEDB
                | NOCREATEDB
                | CREATEROLE
                | NOCREATEROLE
                | INHERIT
                | NOINHERIT
                | LOGIN
                | NOLOGIN
                | CONNECTION LIMIT connlimit
                | [ ENCRYPTED ] PASSWORD  ' password ' 
                | PASSWORD NULL
                | VALID UNTIL  ' timestamp ' 
                | IN ROLE role_name [ , ... ]
                | IN GROUP role_name [ , ... ]
                | ROLE role_name [ , ... ]
                | ADMIN role_name [ , ... ]
                | USER role_name [ , ... ]
                | SYSID uid

描述：定义一个新的数据库角色。CREATE USER是 CREATE ROLE的一个别名，详细信息请参考[CREATE ROLE](#_CREATE ROLE)。
示例：
示例1：创建一个带有密码的新用户

```
CREATE USER John WITH PASSWORD 'password';
```

示例2：授予John对bigmath数据库的所有权限。 

```
GRANT ALL ON DATABASE bigmath TO John;
```

示例3：从bigmath数据库中删除John的权限。

```
REVOKE ALL ON DATABASE bigmath FROM John;
```

## **CREATE USER MAPPING**

定义一个用户到一个外部服务器的新映射
语法：
create_user_mapping ::= CREATE USER MAPPING [ IF NOT EXISTS ]  FOR 
                        user SERVER server_name  
                        [ OPTIONS ( fdw_options ) ]

描述：CREATE USER MAPPING定义一个用户到一个外部服务器的新映射。
如果用户和外部服务器之间的映射已经存在，则除非使用IF NOT EXISTS 子句，否则将引发错误。
参数：
Options：OPTIONS子句指定外部数据服务器的选项。它们通常定义要在外部数据源上使用的映射用户名和密码，但实际允许的选项名称和值特定于服务器的外部数据包装器。 
示例：

```
CREATE USER MAPPING FOR myuser SERVER my_server OPTIONS (user 'john', password 'password');
```

## **CREATE VIEW**

定义一个新视图
语法：
create_view ::= CREATE [ OR REPLACE ] [ TEMPORARY | TEMP ] VIEW 
                qualified_name  [ ( column_name [ , ... ] ) ] AS 
                select

描述：使用CREATE VIEW语句在数据库中创建视图。它定义了视图名称和定义它的（select）语句。
参数：
qualified_name：指定视图的名称。如果指定的数据库中已存在具有该名称的视图，则会引发错误（除非使用了OR REPLACE选项）。
column_list：指定以逗号分隔的列的列表。如果未指定，则从查询中推断列名。
select：指定SELECT或VALUES语句，该语句将提供视图的列和行。
TEMPORARY | TEMP：使用此限定符将创建一个临时视图。临时视图仅在创建它们的当前客户端会话中可见，并在会话结束时自动删除。
示例：
创建一个示例表：

```
CREATE TABLE sample(k1 int, k2 int, v1 int, v2 text, PRIMARY KEY (k1, k2));
```

插入一些数据行：

```
INSERT INTO sample(k1, k2, v1, v2) VALUES (1, 2.0, 3, 'a'), (2, 3.0, 4, 'b'), (3, 4.0, 5, 'c');
Create a view on the sample table.
```

基于sample表，创建一个视图。

```
CREATE VIEW sample_view AS SELECT * FROM sample WHERE v2 != 'b' ORDER BY k1 DESC;
```

从上面创建的视图中查询数据：

```
SELECT * FROM sample_view;
```

## **DEALLOCATE**

释放一个预备语句
语法：
deallocate ::= DEALLOCATE [ PREPARE ] { name | ALL }
描述：DEALLOCATE被用来释放一个之前准备好的 SQL 语句

参数：
PREPARE：这个关键词会被忽略。
name ：要释放的预备语句的名称。

ALL：释放所有预备语句。

示例：

```
CREATE TABLE sample(k1 int, k2 int, v1 int, v2 text, PRIMARY KEY (k1, k2));
PREPARE ins (bigint, double precision, int, text) AS
               INSERT INTO sample(k1, k2, v1, v2) VALUES ($1, $2, $3, $4);
DEALLOCATE PREPARE  ins;
```

## **DECLARE**

定义一个游标
语法：
declare ::= DECLARE cursor_name [ BINARY ] [ INSENSITIVE ] 
            [ [ NO ] SCROLL ]  CURSOR [ { WITH | WITHOUT } HOLD ] FOR 
            subquery

描述：DECLARE允许用户创建游标，游标可以被用来在大型查询暂停时检索少量的行。游标被创建后，可以用FETCH从中取得行。
DECLARE创建一个游标。游标的持续时间限制为声明它的会话的持续时间。请注意可保持游标和不可保持游标之间的最大生存时间差异。CLOSE语句会删除一个游标，这样您就可以缩短它的生存期（通常是为了节省资源）。
pg_cursors目录视图列出当前会话中所有当前存在的游标。
参数：

cursor_name：要创建的游标的名称。游标仅由非限定名称标识，并且仅在声明它的会话中可见。这决定了其名称的唯一性范围。（在这方面，游标的名称与准备好的语句的名称类似。） 
BINARY：让游标返回二进制数据而不是返回文本格式数据。
通常，游标被指定为以文本格式返回数据，这与SELECT语句产生的数据相同。这种二进制格式减少了服务器和客户端的转换工作量，但代价是程序员在处理依赖于平台的二进制数据格式时付出更多。例如，如果查询从整数列返回一个值，那么您将使用默认选项获得字符串值1。但是使用二进制游标，您会得到一个4字节的字段，其中包含值的内部表示形式。
小心使用二进制游标。包括bsqlsh在内的许多应用程序都不准备处理二进制游标，并期望数据以文本格式返回。 
INSENSITIVE：指示从游标中检索数据的过程不受游标创建之后在其底层表上发生的更新的影响。因此在BSQL中，这是唯一的行为。因此，此关键字没有任何作用，仅为与SQL标准兼容而被接受。
SCROLL | NO SCROLL:SCROLL指定游标可以用非顺序（例如，反向）的方式从中检索行。根据查询的执行计划的复杂度，指定 SCROLL可能导致查询执行时间上的性能损失。 NO SCROLL指定游标不能以非顺序的方式从中检索行。默认是允许在某些情况下滚动，但这和指定 SCROLL不完全相同。
SCROLL指定您可以使用FETCH和MOVE的风格来访问当前行以及游标结果集中位于当前行之前的行（即，包括FETCH RELATIVE 0访问的行及其之前）。在简单的情况下，执行计划本质上是可逆的：它允许向后获取，就像允许向前获取一样容易。但并非所有的执行计划都是可逆的；当计划不可逆时，指定SCROLL意味着创建游标的子查询在执行第一个MOVE或FETCH语句时定义的行的缓存（或在访问新行时按需创建）。这意味着性能成本和资源消耗成本。
NO SCROLL指定游标不能用于检索当前行或位于其前面的行。

如果既不指定SCROLL也不指定NO SCROLL，则只在某些情况下允许滚动，这与显式指定SCROLL不同。 
WITH HOLD | WITHOUT HOLD：WITH HOLD指定该游标在创建它的事务提交之后还能被继续使用。WITHOUT HOLD指定该游标不能在创建它的事务之外使用。如果两者都没有指定，则默认为 WITHOUT HOLD。
WITHOUT HOLD指定在创建游标的事务结束后（即使以成功提交结束）不能使用游标。
WITH HOLD指定在创建游标的事务成功提交后，游标可以继续使用。（当然，如果创建它的事务回滚，它就会消失。）
示例：

```
drop table if exists t cascade;
create table t(k, v) as
select g.val, g.val*100
from generate_series(1, 22) as g(val);
 
start transaction;
  declare cur scroll cursor without hold for
  select k, v
  from t
  where (k <> all (array[1, 3, 5, 7, 11, 13, 17, 19]))
  order by k;
  
  select
    statement,
    is_holdable::text,
    is_scrollable::text
  from pg_cursors where name = 'cur'
  and not is_binary;
 
  fetch all from cur;
  
  close cur;
rollback;
```

上述脚本中，执行select... from pg_cursors; 
返回类似如下信息：

```
                       statement                        | is_holdable | is_scrollable 
--------------------------------------------------------+-------------+---------------
 declare cur scroll cursor without hold for            +| false       | true
   select k, v                                         +|             | 
   from t                                              +|             | 
   where (k <> all (array[1, 3, 5, 7, 11, 13, 17, 19]))+|             | 
   order by k;           
```

执行FETCH ALL：
返回如下信息：

```
 k  |  v   
----+------
  2 |  200
  4 |  400
  6 |  600
  8 |  800
  9 |  900
 10 | 1000
 12 | 1200
 14 | 1400
 15 | 1500
 16 | 1600
 18 | 1800
 20 | 2000
 21 | 2100
 22 | 2200
```

## **DELETE**

删除一个表的行
语法：
delete ::= [ with_clause ]  DELETE FROM table_expr [ [ AS ] alias ]  
           [ WHERE boolean_expression | WHERE CURRENT OF cursor_name ] 
            [ returning_clause ]

returning_clause ::= RETURNING { * | { output_expression 
                                     [ [ AS ] output_name ] } 
                                     [ , ... ] }

描述：使用DELETE语句可以删除满足某些条件的行，如果WHERE子句中没有提供条件，则会删除所有行。DELETE输出要删除的行数 。
注意：表继承不被支持。
参数：
with_query：指定DELETE语句中按名称引用的子查询。 
table_name：要从其中删除行的表名（可以是模式限定的）。
alias：目标表的一个别名。在DELETE语句中指定目标表的标识符。当指定了别名时，必须使用它来代替语句中的实际表。
RETURNING：指定要返回的值。output_expression引用列时，将使用该列的现有值（已删除的值）进行返回。
示例：
创建一个示例表，插入几行，然后删除其中一行 

```
CREATE TABLE sample(k1 int, k2 int, v1 int, v2 text, PRIMARY KEY (k1, k2));
INSERT INTO sample VALUES (1, 2.0, 3, 'a'), (2, 3.0, 4, 'b'), (3, 4.0, 5, 'c');
SELECT * FROM sample ORDER BY k1;
```

返回信息如下：

```
 k1 | k2 | v1 | v2 
----+----+----+----
  1 |  2 |  3 | a
  2 |  3 |  4 | b
  3 |  4 |  5 | c
```

```
DELETE FROM sample WHERE k1 = 2 AND k2 = 3;
SELECT * FROM sample ORDER BY k1;
```

返回信息如下：

```
 k1 | k2 | v1 | v2
----+----+----+----
  1 |  2 |  3 | a
  3 |  4 |  5 | c
```

## **DO**

执行一个匿名代码块
语法：
do ::= DO ' plpgsql_block_stmt '
描述：使用DO语句执行一个匿名PL/pgSQL块语句——换句话说，一个暂时的匿名PL/pgSQL过程。plpgsql_block_stmt被视为一个没有参数的过程的主体： 
块语句遇到的任何SQL语句都将以与在语言plpgsql子程序中遇到的SQL语句相同的方式进行处理，因此，如果在同一会话中使用文本相同的块语句重复执行DO语句，然后，所包含SQL语句的第二次和后续执行将受益于第一次遇到它时所做的语法和语义分析。
语法允许一个可选的LANGUAGE子句，该子句可以在代码块之前或之后编写。但是，唯一支持的选择是语言plpgsql。例如，尝试执行指定语言sql的DO语句会导致0A000运行时错误：
language "sql" does not support inline code execution

参数：
plpgsql_block_stmt：要执行的过程语言代码plpgsql_block_stmt。必须将其指定为字符串文字，就像在CREATE FUNCTION 和CREATE PROCEDURE中一样。AiSQL建议您使用美元符号来标准化，例如：$body。
lang_name：指定用以编写代码的过程语言的名称。默认值为plpgsql。请参阅提示避免在上面的“DO”语句中使用可选的“LANGUAGE”子句。此代码是合法的。它运行时没有错误，并具有预期效果。

```
do
  language plpgsql
$body$
begin
  raise info 'Block statement started at %',
    to_char((statement_timestamp() at time zone 'UTC'), 'hh24:mi:ss Dy');
end;
$body$;
 
示例：
do $body$
begin
  drop schema if exists s cascade;
  create schema s;
 
  create table s.masters(
    mk serial primary key,
    mv text not null unique);
 
  create table s.details(
    dk serial primary key,
    mk int not null references s.masters(mk),
    dv text not null);
end;
$body$;
```

假设在执行DO语句的那一刻，模式s已经存在，但由用户所有，而不是current_role内置函数返回的用户（并且这个当前角色不是超级用户）。同样，假设当前没有正在进行的事务，因此块语句是在单语句自动事务模式下执行的。
删除架构（如果存在）的级联尝试将导致42501错误：

```
must be owner of schema s
```

然后，该块将立即退出，并出现未处理的异常，运行时系统将自动发出一个隐藏的提交——在这里，这将具有与回滚相同的效果。将此行为与在显式start transaction; ... commit; 封装相同语句的行为进行比较： 

```
start transaction;
  drop schema if exists s cascade;
  create schema s;
 
  create table s.masters(
    mk serial primary key,
    mv text not null unique);
 
  create table s.details(
    dk serial primary key,
    mk int not null references s.masters(mk),
    dv text not null);
commit;
```

## **DROP AGGREGATE**

删除一个聚集函数
语法：
drop_aggregate ::= DROP AGGREGATE [ IF EXISTS ] 
                   { aggregate_name ( aggregate_signature ) } 
                   [ , ... ] [ CASCADE | RESTRICT ]

aggregate_signature ::= * | aggregate_arg [ , ... ]
                        | [ aggregate_arg [ , ... ] ] ORDER BY 
                          aggregate_arg [ , ... ]


示例：
示例1：

```
CREATE AGGREGATE newcnt(*) (
             sfunc = int8inc,
             stype = int8,
             initcond = '0',
             parallel = safe
           );
DROP AGGREGATE newcnt(*);
```

示例2：CASCADE 与RESTRICT 

```
CREATE AGGREGATE newcnt(*) (
             sfunc = int8inc,
             stype = int8,
             initcond = '0',
             parallel = safe
           );
CREATE VIEW cascade_view AS
             SELECT newcnt(*) FROM pg_aggregate;
```

-- 如下将显示错误:

```
DROP AGGREGATE newcnt(*) RESTRICT;
```

-- 如下将显示错误:

```
DROP AGGREGATE newcnt(*);
DROP AGGREGATE newcnt(*) CASCADE;
```

## **DROP CAST**

删除一个造型
语法：
drop_cast ::= DROP CAST [ IF EXISTS ] ( cast_signature ) 
              [ CASCADE | RESTRICT ]

描述：DROP CAST删除一个之前定义好的造型。
示例：

```
CREATE FUNCTION sql_to_date(integer) RETURNS date AS $$
             SELECT $1::text::date
             $$ LANGUAGE SQL IMMUTABLE STRICT;
CREATE CAST (integer AS date) WITH FUNCTION sql_to_date(integer) AS ASSIGNMENT;
DROP CAST (integer AS date);
```

## **DROP DATABASE**

删除一个数据库
语法：
drop_database ::= DROP DATABASE [ IF EXISTS ] database_name

描述：
删除数据库和所有关联的对象。drop语句完成后，所有与database_name关联的对象（如表）都将无效。所有到丢弃数据库的连接都将失效，并最终断开连接。
参数：
database_name：指定数据的名称。
示例：

```
drop database IF EXISTS test;
```

## **DROP DOMAIN**

删除一个域
语法：
drop_domain ::= DROP DOMAIN [ IF EXISTS ] name [ , ... ] 
                [ CASCADE | RESTRICT ]
描述：DROP DOMAIN删除一个域。
参数：
IF EXISTS：如果该域不存在则不要抛出一个错误，而是发出一个提示。
name：一个现有域的名称（可以是模式限定的）。
CASCADE：自动删除依赖于该域的对象（例如表列），然后删除所有依赖于那些对象的对象。
RESTRICT：如果有任何对象依赖于该域，则拒绝删除它。这是默认值。

示例：


示例1：

```
CREATE DOMAIN idx int DEFAULT 5 CHECK (VALUE > 0);
DROP DOMAIN idx;
```

示例2：

```
CREATE DOMAIN idx int DEFAULT 5 CHECK (VALUE > 0);
CREATE TABLE t (k idx primary key);
DROP DOMAIN idx CASCADE;
```

## **DROP EXTENSION**

删除一个扩展
语法：
drop_extension ::= DROP EXTENSION [ IF EXISTS ] extension_name 
                   [ , ... ] [ CASCADE | RESTRICT ]

描述：DROP EXTENSION从数据库删除扩展。
参数：
IF EXISTS：如果该扩展不存在则不要抛出一个错误，而是发出一个提示。
name：一个已安装扩展的名称。
CASCADE：自动删除依赖于该扩展的对象，然后删除所有依赖于那些对象的对象。
RESTRICT：如果有任何对象依赖于该扩展（而不是它自己拥有的成员对象和其他被列在同一个DROP命令中的扩展），则拒绝删除它。这是默认值。

示例
示例1：

```
DROP EXTENSION IF EXISTS cube;
```

示例2：

```
CREATE EXTENSION cube;
CREATE EXTENSION earthdistance;
DROP EXTENSION IF EXISTS cube RESTRICT; 
```

返回错误信息如下：

```
ERROR:  cannot drop extension cube because other objects depend on it
DETAIL:  extension earthdistance depends on type cube
HINT:  Use DROP ... CASCADE to drop the dependent objects too.
 
DROP EXTENSION IF EXISTS cube CASCADE;
```

返回信息如下：

```
NOTICE:  drop cascades to extension earthdistance
DROP EXTENSION
```

## **DROP FOREIGN DATA WRAPPER**

删除一个外部数据包装器
语法：
drop_foreign_data_wrapper ::= DROP FOREIGN DATA WRAPPER [ IF EXISTS ] 
                              fdw_name [ CASCADE | RESTRICT ]

描述：DROP FOREIGN DATA WRAPPER 移除一个已有的外部数据包装器。
参数：
IF EXISTS：如果该外部数据包装器不存在则不要抛出一个错误，而是发出一个提示。
fdw_name：一个现有外部数据包装器的名称。

CASCADE：自动删除依赖于该外部数据包装器的对象（例如服务器），然后删除所有依赖于那些对象的对象。
RESTRICT：如果有任何对象依赖于该外部数据包装器，则拒绝删除它。这是默认值。

示例：

```
DROP FOREIGN DATA WRAPPER my_wrapper CASCADE;
```

## **DROP FOREIGN TABLE**

删除一个外部表
语法：
drop_foreign_table ::= DROP FOREIGN TABLE [ IF EXISTS ] table_name 
                       [ CASCADE | RESTRICT ]

描述：DROP FOREIGN TABLE删除一个外部表。

参数：
IF EXISTS：如果该外部表不存在则不要抛出一个错误，而是发出一个提示。
table_name：要删除的外部表的名称（可以是模式限定的）。
CASCADE：自动删除依赖于该外部表的对象（例如视图），然后删除所有依赖于那些对象的对象。
RESTRICT：如果有任何对象依赖于该外部表，则拒绝删除它。这是默认值。


示例：

```
DROP FOREIGN TABLE mytable CASCADE;
```

## **DROP FUNCTION**

删除一个函数
语法：
drop_function ::= DROP FUNCTION [ IF EXISTS ]  
                  { subprogram_name ( [ subprogram_signature ] ) } 
                  [ , ... ] [ CASCADE | RESTRICT ]

subprogram_signature ::= arg_decl [ , ... ]

arg_decl ::= [ formal_arg ] [ arg_mode ] arg_type

描述：DROP FUNCTION移除一个已有函数的定义。要执行这个命令用户必须是该函数的拥有者。该函数的参数类型必须被指定，因为多个不同的函数可能会具有相同的函数名和不同的参数列表。

参数：
IF EXISTS：如果该函数不存在则不要抛出一个错误，而是发出一个提示。
Name：一个现有函数的名称（可以是模式限定的）。 如果未指定参数列表，则该名称在其模式中必须是唯一的。
argmode：一个参数的模式：IN、OUT、 INOUT或者VARIADIC。如果被忽略， 则默认为IN。注意 DROP FUNCTION并不真正关心OUT参数，因为决定函数的身份时只需要输入参数。
argname：一个参数的名称。注意 DROP FUNCTION并不真正关心 参数名称，因为决定函数的身份时只需要参数的数据类型。
argtype：如果函数有参数，这是函数参数的数据类型（可以是模式限定的）。
CASCADE：自动删除依赖于该函数的对象（例如操作符和触发器），然后删除所有 依赖于那些对象的对象。
RESTRICT：如果有任何对象依赖于该函数，则拒绝删除它。这是默认值。


示例：

```
DROP FUNCTION IF EXISTS inc(i integer), mul(integer, integer) CASCADE;
```

## **DROP GROUP**

删除一个数据库角色
语法：
drop_group ::= DROP GROUP [ IF EXISTS ] role_name [ , ... ]

描述：DROP GROUP是 DROP ROLE的一个别名。
有关更多详细信息，请参阅[DROP ROLE](#_DROP ROLE)。 

示例：

```
DROP GROUP SysAdmin;
```

## **DROP INDEX**

删除一个索引
语法：
drop_index ::= DROP INDEX [ IF EXISTS ] index_name 
               [ CASCADE | RESTRICT ]
描述：DROP INDEX从数据库系统中删除一个已有的索引。
参数：
IF EXISTS：如果该索引不存在则不要抛出一个错误，而是发出一个提示。
index_name：要删除的索引的名称（可以是模式限定的）。
CASCADE：自动删除依赖于该索引的对象，然后删除所有依赖于那些对象的对象。
RESTRICT：如果有任何对象依赖于该索引，则拒绝删除它。这是默认值。


## **DROP MATERIALIZED VIEW**

删除一个物化视图
语法：
drop_matview ::= DROP MATERIALIZED VIEW [ IF EXISTS ] matview_name  
                 [ CASCADE | RESTRICT ]

描述：DROP MATERIALIZED VIEW删除一个现有的物化视图。要执行这个命令你必须是该物化视图的拥有者。

参数：
IF EXISTS：如果该物化视图不存在则不要抛出一个错误，而是发出一个提示。
matview_name：要移除的物化视图的名称（可以是模式限定的）。
CASCADE：自动删除依赖于该物化视图的对象（例如其他物化视图或常规视图），然后删除所有依赖于那些对象的对象。
RESTRICT：如果有任何对象依赖于该物化视图，则拒绝删除它。这是默认值。

示例：

```
CREATE TABLE t1(a int4);
CREATE MATERIALIZED VIEW m1 AS SELECT * FROM t1;
CREATE MATERIALIZED VIEW m2 AS SELECT * FROM m1;
DROP MATERIALIZED VIEW m1; --失败，因为m2 依赖m1
 
DROP MATERIALIZED VIEW m1 CASCADE; --成功
```

## **DROP OPERATOR**

删除一个操作符
语法：
drop_operator ::= DROP OPERATOR [ IF EXISTS ] 
                  { operator_name ( operator_signature ) } [ , ... ] 
                  [ CASCADE | RESTRICT ]

operator_signature ::= { left_type | NONE } , { right_type | NONE }

参数：
IF EXISTS：如果该操作符不存在则不要抛出一个错误，而是发出一个提示。
operator_name：一个现有的操作符的名称（可以是模式限定的）。
left_type：该操作符左操作数的数据类型，如果没有左操作数就写 NONE。
right_type：该操作符右操作数的数据类型，如果没有右操作数就写 NONE。
CASCADE：自动删除依赖于该操作符的对象（例如使用它的视图），然后删除所有 依赖于那些对象的对象。
RESTRICT：如果有任何对象依赖于该操作符，则拒绝删除它。这是默认值。

示例：

```
CREATE OPERATOR @#@ (
             rightarg = int8,
             procedure = numeric_fac
           );
DROP OPERATOR @#@ (NONE, int8);
```

## **DROP OPERATOR CLASS**

删除一个操作符类
语法：
drop_operator_class ::= DROP OPERATOR CLASS [ IF EXISTS ] 
                        operator_class_name USING index_method 
                        [ CASCADE | RESTRICT ]
描述：DROP OPERATOR CLASS删除一个现有的操作符类。

参数：
IF EXISTS：如果该操作符类不存在则不要抛出一个错误，而是发出一个提示。
operator_class_name：一个现有的操作符类的名称（可以是模式限定的）。
index_method：该操作符类适用的索引访问方法的名称。
CASCADE：自动删除依赖于该操作符类的对象（例如索引），然后删除所有依赖于那些对象的对象。
RESTRICT：如果有任何对象依赖于该操作符类，则拒绝删除它。这是默认值。

示例：

```
CREATE OPERATOR CLASS my_op_class
           FOR TYPE int4
           USING btree AS
           OPERATOR 1 <,
           OPERATOR 2 <=;
DROP OPERATOR CLASS my_op_class USING btree;
```

## **DROP OWNED**

删除一个数据库角色拥有的数据库对象
语法：
drop_owned ::= DROP OWNED BY role_specification [ , ... ] 
               [ CASCADE | RESTRICT ]

role_specification ::= role_name | CURRENT_USER | SESSION_USER
描述：DROP OWNED删除当前数据库中被指定角色之一拥有的所有对象，授予给定角色对当前数据库中对象或共享对象的任何权限也将被撤销。
参数：
CASCADE：自动删除依赖于受影响对象的对象，然后删除所有依赖于那些对象的对象。
RESTRICT：如果有任何其他数据库对象依赖于一个受影响的对象， 则拒绝删除一个角色所拥有的对象。这是默认值。

示例：删除john拥有的所有对象

```
drop owned by john;
```

## **DROP POLICY**

从一个表删除一条行级安全性策略
语法：
drop_policy ::= DROP POLICY [ IF EXISTS ] name ON table_name 
                [ CASCADE | RESTRICT ]

描述：DROP POLICY从该表删除指定的策略。

参数：
IF EXISTS：如果该策略不存在也不抛出一个错误。这种情况下会发出一个提示。
name：要删除的策略的名称。
table_name：该策略所在的表的名称（可以被模式限定）。
CASCADE | RESTRICT：这些关键词不会产生效果，因为它们不依赖于策略。

示例：

```
DROP POLICY p1 ON table_foo;
```

## **DROP PROCEDURE**

删除一个过程
语法：
drop_procedure ::= DROP PROCEDURE [ IF EXISTS ]  
                   { subprogram_name ( [ subprogram_signature ] ) } 
                   [ , ... ] [ CASCADE | RESTRICT ]

subprogram_signature ::= arg_decl [ , ... ]

arg_decl ::= [ formal_arg ] [ arg_mode ] arg_type

描述：DROP PROCEDURE删除一个现有过程的定义。为了执行这个命令，用户必须是该过程的拥有者。该过程的参数类型必须指定，因为可能存在多个不同的过程具有相同名称和不同参数列表。

参数：
IF EXISTS：如果该过程不存在也不抛出一个错误。这种情况下会发出一个提示。
subprogram_name：现有过程的名称（可以是被方案限定的）。如果没有指定参数列表，则该名称在其所属的方案中必须是唯一的。
argmode：参数的模式：IN或者VARIADIC。如果省略，默认为IN。
formal_arg：参数的名称。注意，其实DROP PROCEDURE并不在意参数名称，因为只需要参数的数据类型来确定过程的身份。
argtype：该过程如果有参数，参数的数据类型（可以是被方案限定的）。
CASCADE：自动删除依赖于该过程的对象，然后接着删除依赖于那些对象的对象。
RESTRICT：如果有任何对象依赖于该过程，则拒绝删除它。这是默认选项。

示例：

```
DROP PROCEDURE IF EXISTS transfer(integer, integer, dec) CASCADE;
```

## **DROP ROLE**

删除一个数据库角色
语法：
drop_role ::= DROP ROLE [ IF EXISTS ] role_name [ , ... ]

描述：DROP ROLE删除指定的角色。要删除一个超级用户角色，你必须自己就是一个超级用户。要删除一个非超级用户角色，你必须具有CREATEROLE特权。
在删除角色之前，必须删除它所拥有的所有对象（或重新分配其所有权），并撤消授予该角色对其他对象的任何权限。REASSIGN OWNED和DROP OWNED命令可用于此目的。
但是，没有必要删除涉及该角色的角色成员身份。DROP ROLE会自动撤销目标角色在其他角色中的任何成员身份，以及目标角色中其他角色的任何成员资格。其他角色不会被删除或影响。
参数：
IF EXISTS：如果该角色不存在则不要抛出一个错误，而是发出一个提示。
role_name：要删除的角色的名称。
示例：

```
DROP ROLE John;
```

## **DROP RULE**

删除一个重写规则
语法：
drop_rule ::= DROP RULE [ IF EXISTS ] rule_name ON table_name 
              [ CASCADE | RESTRICT ]

描述：DROP RULE删除一个重写规则。

参数：
IF EXISTS：如果该规则不存在则不要抛出一个错误，而是发出一个提示。
rule_name：要删除的规则的名称。
table_name：该规则适用的表或视图的名称（可以是模式限定的）。
CASCADE：自动删除依赖于该规则的对象，然后删除所有依赖于那些对象的对象。
RESTRICT：如果有任何对象依赖于该规则，则拒绝删除它。这是默认值。

示例：

```
CREATE TABLE t1(a int4, b int4);
CREATE TABLE t2(a int4, b int4);
CREATE RULE t1_to_t2 AS ON INSERT TO t1 DO INSTEAD
             INSERT INTO t2 VALUES (new.a, new.b);
DROP RULE t1_to_t2 ON t1;
```

## **DROP SCHEMA**

删除一个模式

语法：
drop_schema ::= DROP SCHEMA [ IF EXISTS ] schema_name [ , ... ] 
                [ CASCADE | RESTRICT ]

描述：DROP SCHEMA从数据库中移除模式。
参数
IF EXISTS：如果该模式不存在则不要抛出一个错误，而是发出一个提示。
schema_name：一个模式的名称。
CASCADE：自动删除包含在该模式中的对象（表、函数等），然后删除所有依赖于那些对象的对象。
RESTRICT：如果该模式含有任何对象，则拒绝删除它。这是默认值。
示例：
创建模式，并创建模式下表。

```
CREATE SCHEMA sch1;
CREATE TABLE sch1.t1(id BIGSERIAL PRIMARY KEY);
```

删除模式：

```
DROP SCHEMA sch1;
```

返回如下错误信息：

```
ERROR:  cannot drop schema sch1 because other objects depend on it
DETAIL:  table sch1.t1 depends on schema sch1
HINT:  Use DROP ... CASCADE to drop the dependent objects too.
```

带有CASCADE参数删除模式：

```
DROP SCHEMA sch1 CASCADE;
```

## **DROP SEQUENCE**

删除一个序列
语法：
drop_sequence ::= DROP SEQUENCE [ IF EXISTS ] sequence_name 
                  [ CASCADE | RESTRICT ]
描述：DROP SEQUENCE删除序数生成器。 
参数：
IF EXISTS：如果该序列不存在则不要抛出一个错误，而是发出一个提示。

sequence_name：一个序列的名称（可以是模式限定的）。
CASCADE：自动删除依赖于该序列的对象，然后删除所有依赖于那些对象的对象。
RESTRICT：如果有任何对象依赖于该序列，则拒绝删除它。这是默认值。

示例：

```
CREATE TABLE t(k SERIAL, v INT);
```

\d t
返回信息如下：

```
                            Table "public.t"
 Column |  Type   | Collation | Nullable |           Default            
--------+---------+-----------+----------+------------------------------
 k      | integer |           | not null | nextval('t_k_seq'::regclass)
 v      | integer |           |          | 
```

```
DROP SEQUENCE t_k_seq;
```

返回错误信息如下：

```
ERROR:  cannot drop sequence t_k_seq because other objects depend on it
DETAIL:  default value for column k of table t depends on sequence t_k_seq
HINT:  Use DROP ... CASCADE to drop the dependent objects too.
```

使用CASCADE选项删除序列可以解决问题，还可以删除表t中的默认值。

```
DROP SEQUENCE t_k_seq CASCADE;
```


## **DROP SERVER**

删除一个外部服务器描述符
语法：
drop_server ::= DROP SERVER [ IF EXISTS ] server_name 
                [ CASCADE | RESTRICT ]

描述：DROP SERVER删除一个现有的外部服务器描述符。

参数：
IF EXISTS：如果该服务器不存在则不要抛出一个错误，而是发出一个提示。
server_name：一个现有服务器的名称。
CASCADE：自动删除依赖于该服务器的对象（例如用户映射），然后删除所有依赖于那些对象的对象。
RESTRICT：如果有任何对象依赖于该服务器，则拒绝删除它。这是默认值。

示例：

```
DROP SERVER my_server CASCADE;
```

## **DROP TABLE**

删除一个表
语法：
drop_table ::= DROP TABLE [ IF EXISTS ] table_name [ , ... ] 
               [ CASCADE | RESTRICT ]

描述：使用DROP TABLE语句从数据库中删除一个或多个表（及其所有数据）。
参数：
IF EXISTS：如果该表不存在则不要抛出一个错误，而是发出一个提示。
table_name：要删除的表的名称（可以是模式限定的）。
CASCADE：自动删除依赖于该表的对象（例如视图），然后删除所有依赖于那些对象的对象。
RESTRICT：如果有任何对象依赖于该表，则拒绝删除它。这是默认值。

示例：

```
set client_min_messages = warning;
drop table if exists children, parents cascade;
 
create table parents(k int primary key, v text);
 
create table children(
  k int, parents_k int, v text,
  constraint children_pk primary key(k, parents_k),
  constraint children_fk foreign key(parents_k) references parents(k));
 
\d children
```


返回信息如下：

```
Foreign-key constraints:
    "children_fk" FOREIGN KEY (parents_k) REFERENCES parents(k)
```

执行如下操作：

```
\set VERBOSITY verbose
drop table parents restrict;
```

返回如下错误信息：

```
ERROR:  2BP01: cannot drop table parents because other objects depend on it
```

并返回如下详细信息：

```
DETAIL:  constraint children_fk on table children depends on index parents_pkey
```

继续执行如下操作：

```
drop table parents cascade;
\d children
```

“DROP”执行成功，\d元命令显示表“children”仍然存在，但它现在对现在删除的“parents”表没有外键约束。

## **DROP TRIGGER**

删除一个触发器
语法：
drop_trigger ::= DROP TRIGGER [ IF EXISTS ] name ON table_name 
                 [ CASCADE | RESTRICT ]


描述：DROP TRIGGER删除一个现有的触发器定义。 
参数：
IF EXISTS：如果该触发器不存在则不要抛出一个错误，而是发出一个提示。
name：要删除的触发器的名称。
table_name：定义了该触发器的表的名称（可以是模式限定的）。
CASCADE：自动删除依赖于该触发器的对象，然后删除所有依赖于那些对象的对象。
RESTRICT：如果有任何对象依赖于该触发器，则拒绝删除它。这是默认值。

示例：

```
DROP TRIGGER update_moddatetime ON posts;
```

## **DROP TYPE**

删除一个数据类型
语法：
drop_type ::= DROP TYPE [ IF EXISTS ] type_name [ , ... ] 
              [ CASCADE | RESTRICT ]

描述：DROP TYPE删除一种用户定义的数据类型。
参数
IF EXISTS：如果该类型不存在则不要抛出一个错误，而是发出一个提示。
type_name：要删除的数据类型的名称（可以是模式限定的）。
CASCADE：自动删除依赖于该类型的对象（例如表列、函数、操作符），然后删除所有依赖于那些对象的对象。
RESTRICT：如果有任何对象依赖于该类型，则拒绝删除它。这是默认值。

示例：
示例1：

```
CREATE TYPE feature_struct AS (id INTEGER, name TEXT);
DROP TYPE feature_struct;
```

示例2：

```
DROP TYPE IF EXISTS feature_shell;
```

示例3：

```
CREATE TYPE feature_enum AS ENUM ('one', 'two', 'three');
CREATE TABLE feature_tab_enum (feature_col feature_enum);
DROP TYPE feature_tab_enum CASCADE;
```

示例4：

```
CREATE TYPE feature_range AS RANGE (subtype=INTEGER);
CREATE TABLE feature_tab_range (feature_col feature_range);
```

-- 接下来执行的这个语句会报告错误

```
DROP TYPE feature_range RESTRICT;
DROP TABLE feature_tab_range;
DROP TYPE feature_range RESTRICT;
```

## **DROP USER**

删除一个数据库角色
语法：
drop_user ::= DROP USER [ IF EXISTS ] role_name [ , ... ]

描述：删除一个数据库角色。
有关更多详细信息，请参阅[DROP ROLE](#_DROP ROLE)。 

示例：

```
DROP USER John;
```

## **END**

提交当前事务
语法：
end ::= END [ TRANSACTION | WORK ]

描述：使用END语句提交当前事务。事务所做的所有更改对其他人都是可见的，并保证在发生崩溃时是持久的。
参数：
WORK | TRANSACTION：可选关键词，它们没有效果。
示例：要提交当前事务并且让所有更改持久化：

```
END;
```

## **EXECUTE**

执行一个预备语句
语法：
execute_statement ::= EXECUTE name [ ( expression [ , ... ] ) ]

描述：EXECUTE被用来执行一个之前准备好的语句，这是一种性能优化，因为在PREPARE处理过程中，准备好的语句将使用不同的值执行多次，而语法和语义分析以及重写只执行一次。
参数：
name：要执行的预备语句的名称。
expression：给预备语句的参数的实际值。这必须是一个能得到与该参数数据类型（ 在预备语句创建时决定）兼容的值的表达式。

示例：
创建一个表：

```
CREATE TABLE sample(k1 int, k2 int, v1 int, v2 text, PRIMARY KEY (k1, k2));
```

准备一个简单的Insert：

```
PREPARE ins (bigint, double precision, int, text) AS
               INSERT INTO sample(k1, k2, v1, v2) VALUES ($1, $2, $3, $4);
```

使用不同的参数执行两次：

```
EXECUTE ins(1, 2.0, 3, 'a');
EXECUTE ins(2, 3.0, 4, 'b');
```

查询并检查结果：

```
SELECT * FROM sample ORDER BY k1;
```

返回信息如下：

```
 k1 | k2 | v1 | v2 
----+----+----+----
  1 |  2 |  3 | a
  2 |  3 |  4 | b
```

## **EXPLAIN**

显示一个语句的执行计划
语法：
explain ::= EXPLAIN [ [ ANALYZE ] [ VERBOSE ] | ( option [ , ... ] ) ] 
            sql_stmt

option ::= ANALYZE [ boolean ]
           | VERBOSE [ boolean ]
           | COSTS [ boolean ]
           | BUFFERS [ boolean ]
           | TIMING [ boolean ]
           | SUMMARY [ boolean ]
           | FORMAT { TEXT | XML | JSON | YAML }

描述：使用EXPLAIN语句可以显示语句的执行计划。如果使用ANALYZE选项，则语句将被执行，而不仅仅是计划执行。在这种情况下，执行信息（而不仅仅是计划的预估）被添加到EXPLAIN结果中。 
参数：
ANALYZE：执行命令并且显示实际的运行时间和其他统计信息。这个参数默认被设置为FALSE。
示例：
创建表：

```
CREATE TABLE sample(k1 int, k2 int, v1 int, v2 text, PRIMARY KEY (k1, k2));
```

插入一些数据：

```
INSERT INTO sample(k1, k2, v1, v2) VALUES (1, 2.0, 3, 'a'), (2, 3.0, 4, 'b'), (3, 4.0, 5, 'c');
```

检查执行计划中的简单选择（条件被下推）。 

```
EXPLAIN SELECT * FROM sample WHERE k1 = 1;
```

返回信息如下：

```
                                  QUERY PLAN
------------------------------------------------------------------------------
 Index Scan using sample_pkey on sample  (cost=0.00..15.25 rows=100 width=44)
   Index Cond: (k1 = 1)
(2 rows)
```

检查执行计划中是否有复杂条件的选择（第二个条件需要筛选）。 

```
EXPLAIN SELECT * FROM sample WHERE k1 = 2 and floor(k2 + 1.5) = v1;
```

返回信息如下：

```
                                  QUERY PLAN                                  
------------------------------------------------------------------------------
 Index Scan using sample_pkey on sample  (cost=0.00..17.75 rows=100 width=44)
   Index Cond: (k1 = 2)
   Remote Filter: (floor(((k2)::numeric + 1.5)) = (v1)::numeric)
(3 rows)
```

使用ANALYZE选项检查执行情况。 

```
EXPLAIN ANALYZE SELECT * FROM sample WHERE k1 = 2 and floor(k2 + 1.5) = v1;
```

返回信息如下：

```
                                                       QUERY PLAN                            
---------------------------------------------------------------------------------------------------
 Index Scan using sample_pkey on sample  (cost=0.00..17.75 rows=100 width=44) (actual time=2.521..2.525 rows=1 loops=1)
   Index Cond: (k1 = 2)
   Remote Filter: (floor(((k2)::numeric + 1.5)) = (v1)::numeric)
 Planning Time: 0.113 ms
 Execution Time: 2.572 ms
 Peak Memory Usage: 14 kB
```

## **FETCH**

使用游标从查询中检索行
语法：
fetch ::= FETCH [ fetch_one_row | fetch_many_rows ] [ FROM | IN ] name

fetch_one_row ::= FIRST
                  | LAST
                  | ABSOLUTE int_literal
                  | NEXT
                  | FORWARD
                  | PRIOR
                  | BACKWARD
                  | RELATIVE int_literal

fetch_many_rows ::= ALL | FORWARD ALL
                    | FORWARD int_literal
                    | int_literal
                    | BACKWARD ALL
                    | BACKWARD int_literal

描述：FETCH从之前创建的一个游标中检索行。  
参数：
NEXT：取出下一行，是默认值。
PRIOR：取出当前位置之前的一行。
FIRST：取出该查询的第一行。
LAST：取出该查询的最后一行。
ABSOLUTE int_literal：取出该查询的第int_literal个行。
RELATIVE int_literal：取出第int_literal个后继行。
int_literal：取出接下来的int_literal行。
ALL：取出所有剩余的行。
FORWARD：取出下一行。
FORWARD int_literal：取出接下来的int_literal行。
FORWARD ALL：取出所有剩下的行。
BACKWARD：取出当前行前面的一行。
BACKWARD int_literal：取出前面的int_literal行（反向扫描）。
BACKWARD ALL：取出所有当前位置之前的行（反向扫描）。
name:一个已打开游标的名称。

游标表示其结果集中的当前位置，在声明游标之后，但在第一次FETCH或MOVE执行之前，当前位置就在第一行之前。

* FETCH FORWARD 0在当前位置获取行，并保持当前位置不变。 
* FETCH NEXT、FETCH、FETCH FORWARD和FETCH FORWARD 1都在当前位置之后立即获取行，并将当前位置更新为刚刚获取的行。然而，如果在执行这些FETCH之一之前，当前位置是结果集中的最后一行，则FETCH从可用行的末尾运行，返回空结果，并且游标位置留在最后一行之后。这是一个唯一定义的状态，因此在这种状态下调用任何数量的FETCH NEXT之后，FETCH PRIOR将获取结果集中的最后一行，并将当前位置更新为最后一行。
* FETCH PRIOR、FETCH BACKWARD和FETCH BACKWARD 1都在当前位置之前立即获取行，并将当前位置更新为刚刚获取的行。然而，如果在执行这些FETCH变体之一之前，当前位置是结果集中的第一行，则FETCH从可用行的开始处运行，返回空结果，并且游标位置留在第一行之前。这是一个唯一定义的状态，因此在该状态下调用任意数量的FETCH PRIOR后，FETCH NEXT将获取结果集中的第一行，并将当前位置更新为第一行。 
* FETCH ALL和FETCH FORWARD ALL从当前位置之后的行中提取所有行，直到最后一行，游标位置在最后一行之后。当然，如果调用FETCH ALL（或FETCH FORWARD ALL）时，当前位置是最后一行，或者在最后一行之后，则返回空结果，并且当前位置留在最后一行后面。 
* FETCH BACKWARD ALL从当前位置之前的行中通过第一行获取所有行，游标位置位于第一行之前。当然，如果调用FETCH BACKWARD ALL时，当前位置是第一行，或者在第一行之前，则返回一个空结果，并且当前位置留在第一行之前。
  当有这么多行可用时，FETCH :n和FETCH FORWARD :n从当前位置后的行向前精确地获取 :n 行，并包括该行，否则，与FETCH FORFORWARD ALL的行为类似，获取的行数也一样多。
  当有这么多行可用时，FETCH BACKWARD :n变量从当前位置之前的行向后精确地获取 :n 行，并包括该行，否则，与FETCH BACKWARD ALL 的行为类似，将获取尽可能多的行。
* FETCH ABSOLUTE :n在指定的绝对位置获取单行。FETCH RELATIVE :n在与当前行精确指示的相对位置（:n可以为负）获取单行。对于FETCH ABSOLUTE :n和FETCH RELATIVE :n，请求的行可能位于第一行之前或最后一行之后。这里的结果与执行其他FETCH导致当前位置位于游标结果集中从第一行到最后一行的范围之外时的结果相同。注意：对于ABSOLUTE 和RELATIVE ，n都可以是负数。
* FETCH FIRST和FETCH LAST中的每一个分别获取第一行或最后一行。因此，这些含义对当前游标位置不敏感，并且每个含义都可以一次又一次地重复，并且总是会产生相同的结果。

请注意，FETCH FORWARD 0、FETCH BACKWARD 0和FETCH RELATIVE 0这三个的含义都相同。

注意：BSQL目前只支持在向前的方向上连续地从游标中提取行。

对于SQL语句获取和移动，以及它们的PL/pgSQL对应项尚未完全起作用的问题，这反映在以下情况下出现的错误中：
每种移动方式都会导致0A000错误，并显示诸如“move尚未支持”之类的消息。
许多fetch风格使用诸如“fetch FIRST尚未支持”、“fetch LAST尚未支持”和“fetch BACKWARD尚未支持”等消息来绘制0A000错误。
以下是唯一不会导致错误的提取方式：
fetch next
fetch
fetch :N
fetch forward
fetch forward :N
fetch all
fetch forward all

示例：

```
drop table if exists t cascade;
create table t(k, v) as
select g.val, g.val*100
from generate_series(1, 22) as g(val);
 
start transaction;
  declare cur scroll cursor without hold for
  select k, v
  from t
  where (k <> all (array[1, 3, 5, 7, 11, 13, 17, 19]))
  order by k;
 
  fetch forward     from cur;
  fetch forward     from cur;
  fetch forward     from cur;
  fetch forward   0 from cur;
  fetch forward   0 from cur;
  fetch forward all from cur;
rollback;
```

返回信息如下（空白行是手动添加的，以提高可读性）：

```
  k |  v  
----+------
  2 |  200
  4 |  400
  6 |  600
 
  6 |  600
  6 |  600
 
  8 |  800
  9 |  900
 10 | 1000
 12 | 1200
 14 | 1400
 15 | 1500
 16 | 1600
 18 | 1800
 20 | 2000
 21 | 2100
 22 | 2200
```

## **GRANT**

定义访问特权
语法：
grant ::= grant_table
          | grant_table_col
          | grant_seq
          | grant_db
          | grant_domain
          | grant_schema
          | grant_type
          | grant_role

grant_table ::= GRANT 
                { { SELECT
                    | INSERT
                    | UPDATE
                    | DELETE
                    | TRUNCATE
                    | REFERENCES
                    | TRIGGER } [ , ... ]
                  | ALL [ PRIVILEGES ] }  ON 
                { [ TABLE ] table_name [ , ... ]
                  | ALL TABLES IN SCHEMA schema_name [ , ... ] }  TO 
                grantee_role [ , ... ] [ WITH GRANT OPTION ]

grant_table_col ::= GRANT 
                    { { SELECT | INSERT | UPDATE | REFERENCES } ( 
                      column_names )
                      | ALL [ PRIVILEGES ] ( column_names ) }  ON 
                    { [ TABLE ] table_name [ , ... ] }  TO 
                    grantee_role [ , ... ]  [ WITH GRANT OPTION ]

grant_seq ::= GRANT { { USAGE | SELECT | UPDATE } [ , ... ]
                      | ALL [ PRIVILEGES ] }  ON 
              { SEQUENCE sequence_name [ , ... ]
                | ALL SEQUENCES IN SCHEMA schema_name [ , ... ] }  TO 
              grantee_role [ , ... ]  [ WITH GRANT OPTION ]

grant_db ::= GRANT { { CREATE | CONNECT | TEMPORARY | TEMP } [ , ... ]
                     | ALL [ PRIVILEGES ] }  ON DATABASE database_name 
             [ , ... ]  TO grantee_role [ , ... ] 
             [ WITH GRANT OPTION ]

grant_domain ::= GRANT { USAGE | ALL [ PRIVILEGES ] }  ON DOMAIN 
                 domain_name [ , ... ]  TO grantee_role [ , ... ]  
                 [ WITH GRANT OPTION ]

grant_schema ::= GRANT { { CREATE | USAGE } [ , ... ]
                         | ALL [ PRIVILEGES ] }  ON SCHEMA schema_name 
                 [ , ... ]  TO grantee_role [ , ... ]  
                 [ WITH GRANT OPTION ]

grant_type ::= GRANT { USAGE | ALL [ PRIVILEGES ] }  ON TYPE type_name 
               [ , ... ]  TO grantee_role [ , ... ]  
               [ WITH GRANT OPTION ]

grant_role ::= GRANT role_name [ , ... ] TO role_name 
               [ , grantee_role [ ... ] ]  [ WITH ADMIN OPTION ]

grantee_role ::= [ GROUP ] role_name
                 | PUBLIC
                 | CURRENT_USER
                 | SESSION_USER

描述：GRANT可以用于分配数据库对象的权限以及角色中的成员身份。
数据库对象上的GRANT
GRANT命令的此变体用于将数据库对象的权限分配给一个或多个角色。如果使用关键字PUBLIC而不是role_name，则意味着将权限授予所有角色，包括以后可能创建的角色。
如果指定了WITH GRANT OPTION，则特权的接收者可以将其授予其他人；否则，接受者就无法做到这一点。无法将授予选项授予PUBLIC。
不需要将权限授予对象的所有者（通常是创建对象的用户），因为默认情况下，所有者拥有所有权限。
可能的特权是：
SELECT：允许从指定表、视图或序列的任何列或者列出的特定列进行SELECT。还允许使用COPY TO。在UPDATE或DELETE中引用已有列值时也需要这个特权。
INSERT：允许INSERT一个新行到指定表中。如果列出了特定的列，只有这些列能在INSERT命令中被赋值（其他列将因此收到默认值）。还允许COPY FROM。
UPDATE：这允许对指定表的任何列或列出的特定列进行UPDATE。 
DELETE：这允许从指定的表中删除一行。 

TRUNCATE：允许在指定的表上TRUNCATE。
REFERENCES：这允许创建引用指定表或表的指定列的外键约束。 
TRIGGER：允许在指定的表上创建触发器。
CREATE：对于数据库，这允许在数据库中创建模式。
对于模式，这允许在模式中创建对象。若要重命名对象，您必须拥有该对象并对包含的架构具有此权限。
CONNECT：这允许用户连接到指定的数据库。此权限在连接启动时检查。 
TEMPORARY | TEMP：这允许在使用指定数据库时创建临时表。
EXECUTE：允许使用指定的函数或过程，以及使用在函数顶部实现的任何运算符。 
USAGE：对于模式，这允许访问指定模式中包含的对象（假设对象自身的权限要求也得到满足）。本质上，这允许被授予者在模式中“查找”对象。
对于序列，这种特权允许使用currval和nextval函数。
对于类型和域，此特权允许在创建表、函数和其他模式对象时使用类型或域。
ALL PRIVILEGES：同时授予所有特权。 

角色GRANT：
GRANT的此变体用于将角色中的成员资格授予一个或多个其他角色。如果指定了WITH ADMIN OPTION，则成员可以将该角色的成员资格授予其他人，也可以撤销该角色的会员资格。
示例：
示例1：向表'stores'上的所有用户授予SELECT权限 

```
GRANT SELECT ON stores TO PUBLIC;
```

示例2：将用户John添加到SysAdmins组。

```
GRANT SysAdmins TO John;
```

## **IMPORT FOREIGN SCHEMA**

从一个外部服务器导入表定义
语法：
import_foreign_schema ::= IMPORT FOREIGN SCHEMA remote_schema  
                          [ { LIMIT TO | EXCEPT } ( table_name [ ... ] 
                            ) ]  FROM SERVER server_name INTO 
                          local_schema  [ OPTIONS ( fdw_options ) ]

描述：使用IMPORT FOREIGN SCHEMA命令可以创建表示外部服务器上的外部架构中的表的外部表。新创建的外部表由发出命令的用户所有。默认情况下，将导入外部架构中的所有表。但是，用户可以使用LIMIT TO 和EXCEPT 子句指定要导入的表。要使用此命令，用户必须在外部服务器上具有USAGE权限，在目标架构上具有CREATE权限。

参数：
LIMIT TO：LIMIT TO子句可以选择性地用于指定要导入的表，所有其他表都将被忽略。
EXCEPT ：EXCEPT子句可以选择性地用于指定不导入哪些表，将忽略所有其他表。

## **INSERT**

在一个表中创建新行
语法：
insert ::= [ with_clause ]  INSERT INTO table_name [ AS alias ] 
           [ ( column_names ) ]  { DEFAULT VALUES
                                   | VALUES ( column_values ) 
                                     [ ,(column_values ... ]
                                   | subquery }  
           [ ON CONFLICT [ conflict_target ] conflict_action ]  
           [ returning_clause ]

returning_clause ::= RETURNING { * | { output_expression 
                                     [ [ AS ] output_name ] } 
                                     [ , ... ] }

column_values ::= { expression | DEFAULT } [ , ... ]

conflict_target ::= ( { column_name | expression } [ , ... ] ) 
                    [ WHERE boolean_expression ]
                    | ON CONSTRAINT constraint_name

conflict_action ::= DO NOTHING
                    | DO UPDATE SET update_item [ , ... ] 
                      [ WHERE boolean_expression ]

描述：INSERT将新行插入到一个表中。
参数：
table_name：一个已有表的名称（可以被模式限定）。
column_names：指定以逗号分隔的列名列表。如果指定的列不存在，则会引发错误。每个主键列都必须具有非null值。
VALUES子句：
每个值列表的长度必须与列列表的长度相同。
每个值都必须可转换为相应的（按位置）列类型。
每个值都可以是一个表达式。
ON CONFLICT子句：
目标表必须至少有一个具有唯一索引或唯一约束的列（列表）。VALUES的参数必须至少包含一个目标表的唯一键。此唯一键的某些值可能是新的，而其他值可能已存在于目标表中。
INSERT ON CONFLICT的基本目的只是插入具有唯一键的新值的行，并使用唯一键的现有值更新行，以将其余指定列的值设置为values关系中的值。通过这种方式，要么是插入，要么是更新；因此，INSERT ON CONFLICT通常被通俗地称为“upsert”。 
示例：
创建表，并插入一些数据行：

```
CREATE TABLE sample(
  id int  CONSTRAINT sample_id_pk PRIMARY KEY,
  c1 text CONSTRAINT sample_c1_NN NOT NULL,
  c2 text CONSTRAINT sample_c2_NN NOT NULL);
INSERT INTO sample(id, c1, c2)
  VALUES (1, 'cat'    , 'sparrow'),
         (2, 'dog'    , 'blackbird'),
         (3, 'monkey' , 'thrush');
```

示例1：on conflict do nothing。在这种情况下，您不需要指定冲突目标。 

```
INSERT INTO sample(id, c1, c2)
  VALUES (3, 'horse' , 'pigeon'),
         (4, 'cow'   , 'robin')
  ON CONFLICT
  DO NOTHING;
```

检查结果。已插入id为4的非冲突行，但id为3的冲突行未更新。

```
SELECT id, c1, c2 FROM sample ORDER BY id;
```

返回信息如下：

```
 id |   c1   |    c2     
----+--------+-----------
  1 | cat    | sparrow
  2 | dog    | blackbird
  3 | monkey | thrush
  4 | cow    | robin
```

示例2：upsert，在这种情况下，您需要指定冲突目标。请注意，EXCLUDED 关键字用于指定冲突行。

```
INSERT INTO sample(id, c1, c2)
  VALUES (3, 'horse' , 'pigeon'),
         (5, 'tiger' , 'starling')
  ON CONFLICT (id)
  DO UPDATE SET (c1, c2) = (EXCLUDED.c1, EXCLUDED.c2);
```

检查结果。插入id为5的非冲突行，并更新id为3的冲突行。

```
SELECT id, c1, c2 FROM sample ORDER BY id;
```

返回信息如下：

```
 id |  c1   |    c2     
----+-------+-----------
  1 | cat   | sparrow
  2 | dog   | blackbird
  3 | horse | pigeon
  4 | cow   | robin
  5 | tiger | starling
```

示例3：
我们可以只对排除的行的指定子集进行“更新”。我们通过尝试插入两个冲突的行（id＝4和id＝5）和一个不冲突的行（id＝6），来说明这一点。并且不应使用“WHERE sample.c1<>'tiger'”指定c1=“tiger”的现有行进行更新。

```
INSERT INTO sample(id, c1, c2)
  VALUES (4, 'deer'   , 'vulture'),
         (5, 'lion'   , 'hawk'),
         (6, 'cheeta' , 'chaffinch')
  ON CONFLICT (id)
  DO UPDATE SET (c1, c2) = (EXCLUDED.c1, EXCLUDED.c2)
  WHERE sample.c1 <> 'tiger';
```

检查结果。插入id为6的非冲突行；更新id＝4的冲突行；但是id＝5（c1＝'tiger'）的冲突行没有被更新； 

```
SELECT id, c1, c2 FROM sample ORDER BY id;
```

返回信息如下：

```
 id |   c1   |    c2     
----+--------+-----------
  1 | cat    | sparrow
  2 | dog    | blackbird
  3 | horse  | pigeon
  4 | deer   | vulture
  5 | tiger  | starling
  6 | cheeta | chaffinch
```

请注意，此限制也是合法的：
WHERE EXCLUDED.c1 <> 'lion'

示例4：复杂的“upsert”示例
重建表，并插入一些数据行：

```
CREATE TABLE sample(
  id INTEGER GENERATED ALWAYS AS IDENTITY CONSTRAINT sample_id_pk PRIMARY KEY,
  c1 TEXT CONSTRAINT sample_c1_NN NOT NULL CONSTRAINT sample_c1_unq unique,
  c2 TEXT CONSTRAINT sample_c2_NN NOT NULL);
 
INSERT INTO sample(c1, c2)
  VALUES ('cat'   , 'sparrow'),
         ('deer'  , 'thrush'),
         ('dog'   , 'blackbird'),
         ('horse' , 'vulture');
 
WITH to_be_upserted AS (
  SELECT c1, c2 FROM (VALUES
    ('cat'   , 'chaffinch'),
    ('deer'  , 'robin'),
    ('lion'  , 'duck'),
    ('tiger' , 'pigeon')
   )
  AS t(c1, c2)
  )
  INSERT INTO sample(c1, c2) SELECT c1, c2 FROM to_be_upserted
  ON CONFLICT ON CONSTRAINT sample_c1_unq
  DO UPDATE SET c2 = EXCLUDED.c2;
```

检查结果：

```
SELECT id, c1, c2 FROM sample ORDER BY c1;
 
 id |  c1   |    c2     
----+-------+-----------
  1 | cat   | chaffinch
  2 | deer  | robin
  3 | dog   | blackbird
  4 | horse | vulture
  7 | lion  | duck
  8 | tiger | pigeon
```

## **LOCK**

锁定一个表
语法：
lock_table ::= LOCK [ TABLE ] { table_expr [ , ... ] } 
               [ IN lockmode MODE ] [ NOWAIT ]

lockmode ::= ACCESS SHARE
             | ROW SHARE
             | ROW EXCLUSIVE
             | SHARE UPDATE EXCLUSIVE
             | SHARE
             | SHARE ROW EXCLUSIVE
             | EXCLUSIVE
             | ACCESS EXCLUSIVE

描述：LOCK TABLE获得一个表级锁，必要时会等待任何冲突锁被释放。
注意：暂不支持表继承。

参数：
name：锁定的表名
lockmode：锁模式指定这个锁和哪些锁冲突，目前仅仅支持ACCESS SHARE。

## **MOVE**

定位一个游标
语法：
move ::= MOVE [ move_to_one_row | move_over_many_rows ] [ FROM | IN ] 
         name

move_to_one_row ::= FIRST
                    | LAST
                    | ABSOLUTE int_literal
                    | NEXT
                    | FORWARD
                    | PRIOR
                    | BACKWARD
                    | RELATIVE int_literal

move_over_many_rows ::= ALL | FORWARD ALL
                        | FORWARD int_literal
                        | int_literal
                        | BACKWARD ALL
                        | BACKWARD int_literal

描述：MOVE重新定位一个游标而不检索任何数据，MOVE更改游标中当前行的位置。

游标表示其结果集中的当前位置。在声明游标之后，但在第一次FETCH或MOVE执行之前，当前位置就在第一行之前。

* MOVE 0和MOVE FORWARD 0变量保持当前位置不变。因此，它们没有实际价值。
* MOVE、MOVE NEXT、MOVE FORWARD和MOVE FORFORWARD 1都会在调用语句之前将当前位置更新到所在位置之后的一行。如果在执行其中一个MOVE之前，当前位置是结果集中的最后一行，则当前位置设置在最后一行之后。这是一个唯一定义的状态，因此在这种状态下调用任意数量的MOVE NEXT后，MOVE PRIOR将获取结果集中的最后一行，并将当前位置更新为最后一行。
* MOVE PRIOR、MOVE BACKWARD和MOVE BACKWARD 1都会在调用语句之前将当前位置更新到所在位置之前的一行。如果在执行这些MOVE之一之前，当前位置是结果集中的第一行，则当前位置设置为第一行之前。这是一个唯一定义的状态，因此在该状态下调用任意数量的MOVE PRIOR后，MOVE NEXT将更新当前位置到第一行。 
* MOVE ALL（和MOVE FORWARD ALL）从当前位置之后的行到最后一行移动所有行，游标位置在最后一行之后。当然，如果调用MOVE ALL（或MOVE FORWARD ALL）时，当前位置已经在最后一行之后，则当前位置留在最后一行后面。
* MOVE BACKWARD ALL在所有行上从当前位置之前的行移动到第一行，游标位置在第一行之前。当然，如果调用MOVE BACKWARD ALL时，当前位置已经在第一行之前，那么当前位置留在第一行之前。
  当有这么足够多的行可用时，MOVE :n (和MOVE FORWARD :n) 从当前位置后的行向前精确移动 :n 行并包括该行；否则，移动尽可能多的行，类似于MOVE FORWARD ALL的行为。 
  当有这么多行可用时，MOVE BACKWARD :n变量会从当前位置之前的行向后移动：n行，并包括该行，否则会尽可能多地移动，类似于MOVE BACKWARD ALL的行为。
* MOVE ABSOLUTE :n移动到指定绝对位置的单行。MOVE RELATIVE :n移动到与当前行精确指示的相对位置（ :n可以为负）的单行。对于MOVE ABSOLUTE :n和MOVE RELATIVE :n，请求的行可能位于第一行之前或最后一行之后。这里的结果与执行其他MOVE变体时的结果相同，这些变体导致当前位置位于游标结果集中从第一行到最后一行的范围之外。注意：对于ABSOLUTE和RELATIVE，n都可以是负数。
* MOVE FIRST和MOVE LAST分别移动到第一行或最后一行。因此，这些含义对当前游标位置不敏感，并且每个含义都可以一次又一次地重复，并且始终具有相同的效果。 

请注意，MOVE FORWARD 0、MOVE BACKWARD 0和MOVE RELATIVE 0这三个含义都相同。

## **PREPARE**

为执行准备一个语句
语法：
prepare_statement ::= PREPARE name [ ( data_type [ , ... ] ) ] AS 
                      subquery

描述：使用PREPARE语句通过解析、分析和重写（但不执行）目标语句来创建准备好的语句的句柄。
PREPARE中的语句可以（应该）包含EXECUTE中的表达式列表将提供的参数（例如$1）。 
PREPARE中的数据类型列表，表示语句中使用的参数的类型。

示例：
创建表：

```
CREATE TABLE sample(k1 int, k2 int, v1 int, v2 text, PRIMARY KEY (k1, k2));
```

准备一个简单的Insert：

```
PREPARE ins (bigint, double precision, int, text) AS
               INSERT INTO sample(k1, k2, v1, v2) VALUES ($1, $2, $3, $4);
```

执行插入两次（使用不同的参数）。

```
EXECUTE ins(1, 2.0, 3, 'a');
EXECUTE ins(2, 3.0, 4, 'b');
```

检查数据：

```
SELECT * FROM sample ORDER BY k1;
```

返回信息如下：

```
 k1 | k2 | v1 | v2 
----+----+----+----
  1 |  2 |  3 | a
  2 |  3 |  4 | b
```

## **REASSIGN OWNED**

更改一个数据库角色拥有的数据库对象的拥有关系
语法：
reassign_owned ::= REASSIGN OWNED BY role_specification [ , ... ] TO 
                   role_specification

role_specification ::= role_name | CURRENT_USER | SESSION_USER
描述：REASSIGN OWNED通常用于为删除角色做准备。它需要源角色和目标角色的成员身份。 
示例：将john拥有的所有对象重新分配给bigmath。 

```
reassign owned by john to bigmath;
```

## **REFRESH MATERIALIZED VIEW**

替换一个物化视图的内容
语法：
refresh_matview ::= REFRESH MATERIALIZED VIEW [ CONCURRENTLY ]  
                    matview_name [ WITH [ NO ] DATA ]

描述：REFRESH MATERIALIZED VIEW完全替换一个物化视图的内容。

参数：
WITH DATA：如果指定了WITH DATA（默认值），则执行视图的查询以获取新数据，并更新物化视图的内容。
WITH NO DATA：如果指定WITH NO DATA，则物化视图的旧内容将被丢弃，物化视图将处于不可编辑状态。
CONCURRENTLY：在少数行需要更新的情况下，此选项可能会更快。如果没有此选项，更新大量行的刷新将使用更少的资源并更快地完成。只有当物化视图上至少有一个UNIQUE索引，并且填充了物化视图时，才允许使用此选项。CONCURRENTLY 和WITH NO DATA不能一起使用。此外，当存在所有列都为null的行时，也不能使用CONCURRENTLY选项。在这两种情况下，刷新物化视图都会引发错误。
示例：

```
CREATE TABLE t1(a int4);
CREATE MATERIALIZED VIEW m1 AS SELECT * FROM t1;
INSERT INTO t1 VALUES (1);
SELECT * FROM m1;
```

返回信息如下：

```
 a
---
(0 rows)
```

```
REFRESH MATERIALIZED VIEW m1;
SELECT * FROM m1;
```

返回信息如下：

```
 a
---
 1
(1 row)
```

## **RELEASE SAVEPOINT**

销毁一个之前定义的保存点
语法：
savepoint_release ::= RELEASE [ SAVEPOINT ] name
参数
name：要销毁的保存点的名称。

示例：
开始一个事务并创建一个保存点。 

```
BEGIN TRANSACTION;
SAVEPOINT test;
```

释放保存点： 

```
RELEASE test;
```

如果此时您尝试回滚到test，则会出现错误：

```
ROLLBACK TO test;
```

返回错误信息如下：

```
ERROR:  savepoint "test" does not exist
```

## **RESET**

把一个运行时参数的值恢复到默认值
语法：
reset_stmt ::= RESET { run_time_parameter | ALL }
描述：RESET把运行时参数恢复到它们的默认值。RESET是SET run_time_parameter TO DEFAULT的另一种拼写。

参数：
run_time_parameter：一个可设置的运行时参数名称。

## **REVOKE**

移除访问特权
语法：
revoke_table ::= REVOKE [ GRANT OPTION FOR ] 
                 { { SELECT
                     | INSERT
                     | UPDATE
                     | DELETE
                     | TRUNCATE
                     | REFERENCES
                     | TRIGGER } [ , ... ]
                   | ALL [ PRIVILEGES ] }  ON 
                 { [ TABLE ] table_name [ , ... ]
                   | ALL TABLES IN SCHEMA schema_name [ , ... ] }  
                 FROM { [ GROUP ] role_name | PUBLIC } [ , ... ] 
                 [ CASCADE | RESTRICT ]

revoke_table_col ::= REVOKE [ GRANT OPTION FOR ]  
                     { { SELECT | INSERT | UPDATE | REFERENCES } ( 
                       column_names ) [ ,(column_names ... ]
                       | ALL [ PRIVILEGES ] ( column_names ) }  ON 
                     [ TABLE ] table_name [ , ... ] FROM 
                     { [ GROUP ] role_name | PUBLIC } [ , ... ] 
                     [ CASCADE | RESTRICT ]

revoke_seq ::= REVOKE [ GRANT OPTION FOR ] 
               { { USAGE | SELECT | UPDATE } [ , ... ]
                 | ALL [ PRIVILEGES ] }  ON 
               { SEQUENCE sequence_name [ , ... ]
                 | ALL SEQUENCES IN SCHEMA schema_name [ , ... ] }  
               FROM { [ GROUP ] role_name | PUBLIC } [ , ... ] 
               [ CASCADE | RESTRICT ]

revoke_db ::= REVOKE [ GRANT OPTION FOR ] 
              { { CREATE | CONNECT | TEMPORARY | TEMP } [ , ... ]
                | ALL [ PRIVILEGES ] } ON DATABASE database_name 
              [ , ... ]  FROM { [ GROUP ] role_name | PUBLIC } 
              [ , ... ] [ CASCADE | RESTRICT ]

revoke_domain ::= REVOKE [ GRANT OPTION FOR ] 
                  { USAGE | ALL [ PRIVILEGES ] } ON DOMAIN domain_name 
                  [ , ... ]  FROM { [ GROUP ] role_name | PUBLIC } 
                  [ , ... ] [ CASCADE | RESTRICT ]

revoke_schema ::= REVOKE [ GRANT OPTION FOR ] 
                  { { CREATE | USAGE } [ , ... ]
                    | ALL [ PRIVILEGES ] } ON SCHEMA schema_name 
                  [ , ... ]  FROM { [ GROUP ] role_name | PUBLIC } 
                  [ , ... ] [ CASCADE | RESTRICT ]

revoke_type ::= REVOKE [ GRANT OPTION FOR ] 
                { USAGE | ALL [ PRIVILEGES ] } ON TYPE type_name 
                [ , ... ]  FROM { [ GROUP ] role_name | PUBLIC } 
                [ , ... ] [ CASCADE | RESTRICT ]

revoke_role ::= REVOKE [ ADMIN OPTION FOR ] role_name [ , ... ] FROM 
                role_name [ , ... ] [ CASCADE | RESTRICT ]
描述：REVOKE命令收回之前从一个或者更多角色授予的特权。
任何角色都有分配给它的所有权限的总和。因此，如果REVOKE用于从PUBLIC中收回SELECT，那么并不意味着所有角色都失去了SELECT权限。如果一个角色被直接授予SELECT，或者通过组继承了它，那么它可以继续拥有SELECT权限。 
如果指定了GRANT OPTION FOR，则只收回特权的授予选项，而不收回特权本身。否则，特权和授予选项都将被撤销。 
类似地，在撤销角色时，如果指定了ADMIN OPTION FOR，则仅撤销该权限的ADMIN选项。 
如果一个用户拥有带有grant选项的特权，并已将其授予其他用户，则如果指定了CASCADE，则从第一个用户撤销特权也将从从属用户撤销特权。否则，REVOKE将失败。 
在撤销表上的权限时，也会自动撤销表中每列的相应列权限（如果有的话）。另一方面，如果一个角色被授予了对表的权限，那么从各个列中撤销相同的权限将没有任何效果。

示例：
示例1：撤销PUBLIC对表“stores”的SELECT权限 

```
REVOKE SELECT ON stores FROM PUBLIC;
```

示例2：从SysAdmins组中删除用户John。

```
REVOKE SysAdmins FROM John;
```

## **ROLLBACK**

中止当前事务
语法：
rollback ::= ROLLBACK [ TRANSACTION | WORK ]
描述：ROLLBACK回滚当前事务并且导致该事务所作的所有更新都被抛弃。

参数：
WORK | TRANSACTION：可选关键词，没有效果。
示例：

```
ROLLBACK ;
```


## **ROLLBACK TO SAVEPOINT**

回滚到一个保存点
语法：
savepoint_rollback ::= ROLLBACK [ WORK | TRANSACTION ] TO 
                       [ SAVEPOINT ] name
描述：使用ROLLBACK TO SAVEPOINT语句将事务的状态恢复到以前建立的保存点。这对于处理和展开键/索引约束冲突等错误特别有用。
参数：
name：要回滚到的保存点。

示例：
创建表，并插入一些数据：

```
CREATE TABLE sample(k int PRIMARY KEY, v int);
INSERT INTO sample(k, v) VALUES (1, 2);
```

开启一个新的事务，并插入一些数据：

```
BEGIN TRANSACTION;
INSERT INTO sample(k, v) VALUES (3, 4);
```

现在，在插入k=1的重复行之前创建一个保存点： 

```
SAVEPOINT test;
INSERT INTO sample(k, v) VALUES (1, 3);
```

得到如下错误信息：

```
duplicate key value violates unique constraint "sample_pkey"
```

任何其他操作都应该出错，因为事务现在处于错误状态：

```
SELECT * FROM sample;
```

得到如下错误信息：

```
current transaction is aborted, commands ignored until end of transaction block
```

但是，您可以回滚到我们以前的保存点并继续处理事务，而不会丢失我们以前的插入。

```
ROLLBACK TO test;
INSERT INTO sample(k, v) VALUES (5, 6);
COMMIT;
```

如果检查表中的行，您将看到在主键冲突之前插入的行，以及在回滚之后插入的行：

```
SELECT * FROM sample;
```

返回如下信息：

```
 k  | v
----+----
  1 |  2
  3 |  4
  5 |  6
```

## **SAVEPOINT**

在当前事务中定义一个新的保存点
语法：
savepoint_create ::= SAVEPOINT name
描述：SAVEPOINT在当前事务中建立一个新保存点。
参数
name：给新保存点的名字。

示例：
创建一个表：

```
CREATE TABLE sample(k int PRIMARY KEY, v int);
```

开启一个事务，并插入一些数据：

```
BEGIN TRANSACTION;
INSERT INTO sample(k, v) VALUES (1, 2);
SAVEPOINT test;
INSERT INTO sample(k, v) VALUES (3, 4);
```

现在，先检查一下数据：

```
SELECT * FROM sample;
```

返回信息如下：

```
 k | v 
---+---
 1 | 2
 3 | 4
```


然后，回滚到保存点测试并再次检查行。请注意，第二行不会出现： 

```
ROLLBACK TO test;
SELECT * FROM sample;
 
```

返回信息如下：

```
 k  | v
----+----
  1 |  2
```

我们甚至可以在这一点上添加新行。如果我们随后提交事务，则只有插入的第一行和第三行将保持： 

```
INSERT INTO sample(k, v) VALUES (5, 6);
COMMIT;
SELECT * FROM SAMPLE;
```

返回信息如下：

```
 k  | v
----+----
  1 |  2
  5 |  6
```

## **SELECT**

从一个表或视图检索行
语法：
select ::= [ with_clause ] SELECT select_list 
           [ trailing_select_clauses ]

with_clause ::= WITH [ RECURSIVE ] 
                { common_table_expression [ , ... ] }

select_list ::= [ ALL | DISTINCT [ ON { ( expression [ , ... ] ) } ] ] 
                 [ * | { { expression
                           | fn_over_window
                           | ordinary_aggregate_fn_invocation
                           | within_group_aggregate_fn_invocation } 
                       [ [ AS ] name ] } [ , ... ] ]

trailing_select_clauses ::= [ FROM { from_item [ , ... ] } ]  
                            [ WHERE boolean_expression ]  
                            [ GROUP BY { grouping_element [ , ... ] } ] 
                             [ HAVING boolean_expression ]  
                            [ WINDOW 
                              { { name AS window_definition } 
                              [ , ... ] } ]  
                            [ { UNION | INTERSECT | EXCEPT } 
                              [ ALL | DISTINCT ] select ]  
                            [ ORDER BY { order_expr [ , ... ] } ]  
                            [ LIMIT { int_expression | ALL } ]  
                            [ OFFSET int_expression [ ROW | ROWS ] ]  
                            [ FETCH { FIRST | NEXT } int_expression 
                              { ROW | ROWS } ONLY ]  
                            [ FOR { UPDATE
                                    | NO KEY UPDATE
                                    | SHARE
                                    | KEY SHARE } 
                              [ OF table_name [ , ... ] ] 
                              [ NOWAIT | SKIP LOCKED ] [ ... ] ]

common_table_expression ::= cte_name [ ( column_name [ , ... ] ) ] AS 
                            ( { select
                                | values
                                | insert
                                | update
                                | delete } )

fn_over_window ::= name  ( [ expression [ , ... ] | * ]  
                   [ FILTER ( WHERE boolean_expression ) ] OVER 
                   { window_definition | name }

ordinary_aggregate_fn_invocation ::= name  ( 
                                     { [ ALL | DISTINCT ] expression 
                                       [ , ... ]
                                       | * } 
                                     [ ORDER BY order_expr [ , ... ] ] 
                                     )  [ FILTER ( WHERE 
                                          boolean_expression ) ]

within_group_aggregate_fn_invocation ::= name  ( 
                                         { expression [ , ... ] } )  
                                         WITHIN GROUP ( ORDER BY 
                                         order_expr [ , ... ] )  
                                         [ FILTER ( WHERE 
                                           boolean_expression ) ]

grouping_element ::= ( ) | ( expression [ , ... ] )
                     | ROLLUP ( expression [ , ... ] )
                     | CUBE ( expression [ , ... ] )
                     | GROUPING SETS ( grouping_element [ , ... ] )

order_expr ::= expression [ ASC | DESC | USING operator_name ] 
               [ NULLS { FIRST | LAST } ]


描述：使用SELECT语句从表中检索符合给定条件的指定列的行。它指定要检索的列、表的名称以及每个选定行必须满足的条件。
相同的语法规则管理子查询，无论您在哪里使用子查询——例如，在INSERT语句中。某些语法点，例如WHERE子句谓词或sqrt（）等函数的实际参数，只允许标量子查询。
参数：
with_clause：WITH子句允许你指定一个或者多个在主查询中可以 其名称引用的子查询。在主查询期间子查询实际扮演了临时表或者视图的角色。每一个子查询都可以是一个SELECT、 TABLE、VALUES、 INSERT、 UPDATE或者 DELETE语句。在WITH中使用一个数据修改语句（INSERT、 UPDATE或者 DELETE）时，通常要包括一个RETURNING子句。
from_item：FROM子句为SELECT 指定一个或者更多源表。如果指定了多个源表，结果将是所有源表的笛卡尔积（交叉连接）。但是通常会增加限定条件（通过 WHERE）来把返回的行限制为该笛卡尔积的一个小子集。
FROM子句可以包含下列元素：
table_name：一个现有表或视图的名称（可以是模式限定的）。
alias：一个包含别名的FROM项的替代名称。
select：一个子SELECT可以出现在FROM子句中。
with_query_name：可以通过写一个WITH查询的名称来引用它，就好像该查询的名称是一个表名。
function_name：函数调用可以出现在FROM子句中，这就好像把该函数的输出创建为一个存在于该SELECT命令期间的临时表。当为该函数调用增加可选的 WITH ORDINALITY子句时，会在该函数的输出列之后追加一个新的列来为每一行编号。
可以用和表一样的方式提供一个别名。如果写了一个别名，还可以写一个列别名列表来为该函数的组合返回类型的一个或者多个属性提供替代名称，包括由ORDINALITY（如果有）增加的新列。
通过把多个函数调用包围在ROWS FROM( ... )中可以把它们整合在单个FROM-子句项中。这样一个项的输出是把每一个函数的第一行串接起来，然后是每个函数的第二行，以此类推。如果有些函数产生的行比其他函数少，则在缺失数据的地方放上空值，这样被返回的总行数总是和产生最多行的函数一样。
如果函数被定义为返回record数据类型，那么必须出现一个别名或者关键词AS，后面跟上形为 ( column_name data_type [, ... ])的列定义列表。列定义列表必须匹配该函数返回的列的实际数量和类型。
在使用ROWS FROM( ... )语法时，如果函数之一要求一个列定义列表，最好把该列定义列表放在ROWS FROM( ... )中该函数的调用之后。当且仅当正好只有一个函数并且没有 WITH ORDINALITY子句时，才能把列定义列表放在 ROWS FROM( ... )结构后面。
要把ORDINALITY和列定义列表一起使用，你必须使用 ROWS FROM( ... )语法，并且把列定义列表放在 ROWS FROM( ... )里面。
join_type：以下中的一个：
[ INNER ] JOIN
LEFT [ OUTER ] JOIN
RIGHT [ OUTER ] JOIN
FULL [ OUTER ] JOIN
CROSS JOIN
fn_over_window：fn_over_window规则表示必须用于调用窗口函数，并且可以用于调用聚合函数的特殊类型的SELECT列表项。在一些SQL数据库系统的术语中，窗口函数被称为分析函数。select规则显示了FILTER和OVER关键字 ，您可以看到，如果不指定OVER子句，就无法以这种方式调用函数，而且OVER子句需要指定为该调用样式命名的所谓窗口。FILTER子句是可选的，只有在以这种方式调用聚合函数时才能使用。
ordinary_aggregate_fn_invocation规则和within_group_aggregate_fn_invocation规则表示用于调用聚合函数（当它不是作为窗口函数调用时）的特殊类型的SELECT列表项。当聚合函数以这两种方式之一调用时，通常与GROUP BY和HAVING子句一起调用。
当您了解了从Window函数部分和Aggregate函数部分的帐户调用这两种函数后，您可以使用bsqlsh中的\df meta命令来发现特定函数的状态，例如： 

```
\df row_number
```

返回信息如下：

```
   Schema   |    Name    | Result data type | Argument data types |  Type  
------------+------------+------------------+---------------------+--------
 pg_catalog | row_number | bigint           |                     | window
\df rank
```

返回信息如下：             

```
   Schema   | Name | Result data type |          Argument data types           |  Type  
------------+------+------------------+----------------------------------------+--------
 pg_catalog | rank | bigint           |                                        | window
 pg_catalog | rank | bigint           | VARIADIC "any" ORDER BY VARIADIC "any" | agg
```

函数的类型被标识为“window”，只能作为窗口函数调用。
函数的类型被同时标识为“window”和agg的函数： 

* 使用fn_over_window语法作为窗口函数 
* 使用within_group_aggregate_fn_invocation语法作为所谓的组内假设集聚合函数。 

 

事实上，类型仅为“agg”的函数可以使用ordinary_aggregate_fn_invocation语法作为聚合函数调用，也可以使用fn_over_window syntax语法作为窗口函数调用。avg()函数在[avg() , count(), max(), min(), sum()](#_avg() , count(), max(), min(), sum()) 小节中进行了描述。
请注意，mode()、percentile_disc()和percentile_cont()这三个函数是例外，也是唯一的例外。这些函数被称为组内有序集合聚合函数。\df仅将这些函数的类型列为“agg”，但这些不能作为窗口函数调用。尝试会导致此错误： 

```
WITHIN GROUP is required for ordered-set aggregate mode
```

实例：
创建表，并插入一些数据：

```
CREATE TABLE sample1(k1 bigint, k2 float, v text, PRIMARY KEY (k1, k2));
CREATE TABLE sample2(k1 bigint, k2 float, v text, PRIMARY KEY (k1, k2));
INSERT INTO sample1(k1, k2, v) VALUES (1, 2.5, 'abc'), (1, 3.5, 'def'), (1, 4.5, 'xyz');
INSERT INTO sample2(k1, k2, v) VALUES (1, 2.5, 'foo'), (1, 4.5, 'bar');
```

使用联接从两个表中进行选择。 

```
SELECT a.k1, a.k2, a.v as av, b.v as bv FROM sample1 a LEFT JOIN sample2 b ON (a.k1 = b.k1 and a.k2 = b.k2) WHERE a.k1 = 1 AND a.k2 IN (2.5, 3.5) ORDER BY a.k2 DESC;
```

返回信息如下：

```
 k1 | k2  | av  | bv  
----+-----+-----+-----
  1 | 3.5 | def | 
  1 | 2.5 | abc | foo
```

## **SET**

更改一个运行时参数
语法：
set ::= SET [ SESSION | LOCAL ] { run_time_parameter { TO | = } 
                                  { value | DEFAULT }
                                  | TIME ZONE 
                                    { timezone | LOCAL | DEFAULT } }

描述：SET命令更改运行时配置参数。使用此语句设置的参数值仅适用于单个会话的范围，并且不超过会话的持续时间。还可以在整个集群级别或特定数据库级别设置此类参数的默认值。例如：
alter database demo set timezone = 'America/Los_Angeles';

参数：
SESSION：指定该命令对当前会话有效（这是默认值）。
LOCAL：指定该命令只对当前事务有效。在COMMIT或者 ROLLBACK之后，会话级别的设置会再次生效。在事务块外部发出这个参数会发出一个警告并且不会有效果。
run_time_parameter：一个可设置运行时参数的名称。您可以使用current_setting()内置函数检查运行时参数的当前值，例如：
select current_setting('search_path');
许多运行时参数都是系统定义的。系统定义的运行时参数仅使用拉丁字母和下划线拼写。而且，在任何特定的PostgreSQL版本中，系统定义的运行时参数列表都是固定的。
show ... SQL语句是select current_setting()的简写，例如： 
show search_path;
show语句有一个特殊的变量show all，当使用select current_setting(...)无效时，它列出了每个系统定义的运行时参数的当前值。查看此列表的另一种方法是查询pg_settings目录视图，其中包含每个设置的一些有用的附带信息。
value：参数的值。

示例：
示例1：用户定义的运行时参数 
您还可以通过适当拼写来动态创建用户定义的运行时参数。用户定义的运行时参数的持续时间永远不会超过会话的持续时间。拼写必须包含句点，而且，只要在set语句中使用双引号，它就可以包含任意字符的任意组合。如下：

```
set min.værelse17 = 'stue';
set "MIN.VÆRELSE17@" = 'kjøkken';
set "^我的.爱好" = '跳舞';
select
  current_setting('min.værelse17')  as s1,
  current_setting('MIN.VÆRELSE17@') as s2,
  current_setting('^我的.爱好')      as s3;
```

返回信息如下：

```
  s1  |   s2    |  s3  
------+---------+------
 stue | kjøkken | 跳舞
```

请注意：show all和查询pg_settings，从不列出用户定义的运行时参数。 

示例2：运行时参数涉及事务

```
set search_path = pg_catalog, pg_temp;
start transaction;
set search_path = some_schema, pg_catalog, pg_temp;
set my.value = 'something';
select current_setting('search_path');
select current_setting('my.value');
rollback;
select current_setting('search_path');
select '>'||current_setting('my.value')||'<';
```

## **SET CONSTRAINTS**

为当前事务设置约束检查时机
语法：
set_constraints ::= SET CONSTRAINTS { ALL | name [ , ... ] } 
                    { DEFERRED | IMMEDIATE }

描述：SET CONSTRAINTS设置当前事务内约束检查的行为，但不适用于not NULL和CHECK约束。
参数：
ALL：更改所有可延迟约束的模式。
name：指定一个或一个约束名称列表。
DEFERRED：将约束设置为在事务提交之前不进行检查，除非标记为DEFERRABLE，否则会立即检查唯一性和排除约束。
IMMEDIATE：设置约束立即生效。

## **SET ROLE**

设置当前会话的当前用户标识符
语法：
set_role ::= SET [ SESSION | LOCAL ] ROLE { role_name | NONE }
reset_role ::= RESET ROLE

描述：指定的角色名称必须是当前会话用户所属的角色。超级用户可以设置为任何角色。一旦将角色设置为role_name，任何其他SQL命令都将使用该角色可用的权限。
要将角色重置回当前用户，可以使用reset role或SET role NONE 

示例：
示例1：更改到用户John：

```
select session_user, current_user;
```

返回信息如下：

```
 session_user | current_user 
--------------+--------------
 bigmath      | bigmath
 
set role john;
```

```
select session_user, current_user;
```

返回信息如下：

```
 session_user | current_user 
--------------+--------------
 bigmath      | john
```

示例2：
更改为新角色将获得该角色可用的权限。

```
select session_user, current_user;
```

返回信息如下：

```
 session_user | current_user 
--------------+--------------
 bigmath      | bigmath
 
create database db1;
set role john;
 
select session_user, current_user;
```

返回信息如下：

```
 session_user | current_user 
--------------+--------------
 bigmath      | john
```

```
create database db2;
```

返回错误信息如下：

```
ERROR:  permission denied to create database
```

## **SET SESSION AUTHORIZATION**

设置当前会话的会话用户标识符和当前用户标识符
语法：
set_session_authorization ::= SET [ SESSION | LOCAL ] SESSION 
                              AUTHORIZATION { role_name | DEFAULT }

reset_session_authorization ::= RESET SESSION AUTHORIZATION

描述：使用SET SESSION AUTHORIZATION语句将当前用户和当前会话的会话用户设置为指定的用户。
会话用户只能由超级用户更改。一旦会话用户设置为role_name，任何其他SQL命令都将使用该角色可用的权限。
要将会话用户重置回当前已验证的用户，可以使用reset session AUTHORIZATION或SET session AUTHORIZATION DEFAULT

示例：

```
select session_user, current_user;
```

返回信息如下：

```
 session_user | current_user
--------------+--------------
 bigmath| bigmath
 
set session authorization john;
 
select session_user, current_user;
 session_user | current_user
--------------+--------------
 john     | john
```

## **SET TRANSACTION**

设置当前事务的特性
语法：
set_transaction ::= SET TRANSACTION transaction_mode [ ... ]

transaction_mode ::= isolation_level
                     | read_write_mode
                     | deferrable_mode

isolation_level ::= ISOLATION LEVEL { READ UNCOMMITTED
                                      | READ COMMITTED
                                      | REPEATABLE READ
                                      | SERIALIZABLE }

read_write_mode ::= READ ONLY | READ WRITE

deferrable_mode ::= [ NOT ] DEFERRABLE

描述：使用SET TRANSACTION语句可以设置当前事务隔离级别。


分别使用Serializable、REPEATABLE Read和Read Committed的PostgreSQL隔离级别语法，支持Serializable、Snapshot和Read Committed Isolation。PostgreSQL的READ UNCOMITTED也映射到READ Committed Isolation。 
只有当TServer标志bm_enable_read_committed_isolation设置为true时，才支持Read Committed隔离级别。默认情况下，此标志为false，在这种情况下，AiSQL事务层的Read Committed隔离级别会回落到更严格的Snapshot 隔离级别（在这种情况中，BSQL的Read Committed和Read UNCOMITTED也依次使用快照隔离）。

参数：
transaction_mode：将事务模式设置为以下之一：

* ISOLATION LEVEL子句
* 访问模式
* DEFERABLE模式 
  ISOLATION LEVEL clause：
  SERIALIZABLE：ANSI SQL标准中的默认值
  REPEATABLE  READ：映射到AiSQL的快照隔离级别
  READ COMMITTED：如果bm_enable_read_committed_isolation=true，则read committed被映射到AiSQL的事务层的read committed（即，语句将在开始之前看到所有提交的行）。但是，默认情况下，bm_enable_read_committed_isolation=false，在这种情况下，AiSQL的事务层的read committed返回到更严格的快照隔离。
  从本质上讲，这可以归结为快照隔离是BSQL中的默认设置。
  READ UNCOMMITTED：READ UNCOMITTED映射到AiSQL的事务层的READ Committed（请注意，如果bm_enable_read_committed_isolation=false，则事务层中的READ Committed 可能反过来映射到Snapshot 隔离级别）。
  在PostgreSQL中，READ UNCOMITTED被映射到READ COMMITTED 。
  READ WRITE mode：默认值。
  READ ONLY mode：READ ONLY模式不会阻止所有对磁盘的写入。 
  当事务为READ ONLY时，以下SQL语句为： 
  1）如果要写入的表不是临时表，则不允许：
  INSERT
  UPDATE
  DELETE
  COPY FROM
  2）总是被禁止： 
  COMMENT
  GRANT
  REVOKE
  TRUNCATE
  3）当要执行的语句是上述语句之一时，不允许执行：
  EXECUTE
  EXPLAIN ANALYZE

DEFERRABLE mode：仅当同时选择了SERIALIZABLE和READ ONLY模式时，用于推迟事务。如果使用，则事务可能会在第一次获取其快照时阻塞，之后它能够在没有SERIALIZABLE 事务的正常开销的情况下运行，并且没有任何导致串行化失败或被串行化失败取消的风险。


DEFERRABLE模式可能有助于长时间运行的报告或备份。

示例：
创建表：

```
CREATE TABLE sample(k1 int, k2 int, v1 int, v2 text, PRIMARY KEY (k1, k2));
```

开启一个事务，并插入一些数据：

```
BEGIN TRANSACTION; SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
INSERT INTO sample(k1, k2, v1, v2) VALUES (1, 2.0, 3, 'a'), (1, 3.0, 4, 'b');
```

用bsqlsh启动一个新的shell，然后开始另一个事务来插入更多的行。 

```
BEGIN TRANSACTION; SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
INSERT INTO sample(k1, k2, v1, v2) VALUES (2, 2.0, 3, 'a'), (2, 3.0, 4, 'b');
```

在每个shell中，检查是否只有当前事务中的行可见。 
在第一个shell中：

```
SELECT * FROM sample;
```

返回信息如下：

```
 k1 | k2 | v1 | v2 
----+----+----+----
  1 |  2 |  3 | a
  1 |  3 |  4 | b
```

在第二个shell中：

```
SELECT * FROM sample;
```

返回信息如下：

```
 k1 | k2 | v1 | v2 
----+----+----+----
  2 |  2 |  3 | a
  2 |  3 |  4 | b
```

提交第一个事务并中止第二个事务。 
在第一个shell中：

```
COMMIT TRANSACTION; 
```

在第二个shell中：

```
ABORT TRANSACTION;
```

在每个shell中，检查是否只有来自提交事务的行可见。 
在第一个shell中：

```
SELECT * FROM sample;
```

返回信息如下：

```
 k1 | k2 | v1 | v2 
----+----+----+----
  1 |  2 |  3 | a
  1 |  3 |  4 | b
```

在第二个shell中：

```
SELECT * FROM sample;
```

返回信息如下：

```
 k1 | k2 | v1 | v2 
----+----+----+----
  1 |  2 |  3 | a
  1 |  3 |  4 | b
```

## **SHOW**

显示一个运行时参数的值
语法：
show_stmt ::= SHOW { run_time_parameter | ALL }

描述：SHOW将显示运行时参数的当前设置。
BSQL中的参数值可以设置，并且通常以与PostgreSQL中相同的方式生效。然而，由于AiSQL使用了不同的存储引擎（DocDB），许多与存储层相关的配置在AiSQL中不会像在PostgreSQL中那样产生效果。例如，与连接和身份验证、查询计划、错误报告和日志记录、运行时统计信息、客户端连接默认值等相关的配置应该与PostgreSQL中的配置一样工作。
但是，与预写日志、清空或复制相关的配置可能不适用于AiSQL。相反，可以使用tserver（或master）配置标志设置相关配置。
参数：
run_time_parameter：指定要显示的参数的名称。
ALL：显示所有配置参数的值以及说明。

## **SHOW TRANSACTION**

使用SHOW TRANSACTION ISOLATION LEVEL语句可以显示当前事务隔离级别。
语法：
show_transaction ::= SHOW TRANSACTION ISOLATION LEVEL

描述：支持Serializable、Snapshot和Read Committed Isolation，分别使用SERIALIZABLE、REPEATABLE READ和READ COMMITTED 的PostgreSQL隔离级别语法。PostgreSQL的READ UNCOMMITTED 也映射到Read Committed 隔离级别。 
只有当TServer标志bm_enable_read_committed_isolation设置为true时，才支持Read Committed隔离级别。默认情况下，此标志为false，在这种情况下，AiSQL事务层的Read Committed隔离级别会回落到更严格的Snapshot 隔离级别（在这种情况中，BSQL的Read Committed和Read UNCOMITTED也依次使用快照隔离）。 

## **START TRANSACTION**

开始一个事务块
语法：
start_transaction ::= START TRANSACTION [ transaction_mode [ ... ] ]
描述：START TRANSACTION语句只是BEGIN语句的另一种拼写。START TRANSACTION后面的语法与BEGIN[TRANACTION|WORK]后面的语法相同。这两种可供选择的拼写具有相同的语义

## **TRUNCATE**

清空一个表或者一组表
语法：
truncate ::= TRUNCATE [ TABLE ] { table_expr [ , ... ] } 
             [ CASCADE | RESTRICT ]

参数：
CASCADE：自动截断所有对任一所提及表有外键引用的表以及任何由于CASCADE被加入到组中的表。
RESTRICT：如果任一表上具有来自命令中没有列出的表的外键引用，则拒绝截断。这是默认值。
指定要截断的表的名称。

* TRUNCATE 获取要截断的表的ACCESS EXCLUSIVE 锁。ACCESS EXCLUSIVE锁定选项尚未完全受支持。
* 外部表不支持TRUNCATE。 
* CASCADE和RESTRICT会影响TRUNCATE语句所针对的表具有依赖表时发生的情况。依赖表（以及它的依赖表）之所以具有这种状态，是因为它们对目标表具有直接或可传递的外键约束。CASCADE导致依赖表的闭包全部被截断。如果目标表有任何依赖表，RESTRICT会导致TRUNCATE尝试失败。即使所有表都为空，此错误结果也是相同的。如果既没有写CASCADE也没有写RESTRICT，那么效果就好像写了RESTRICT一样。 

示例：
创建父子表对并插入一些数据：

```
drop table if exists children cascade;
drop table if exists parents  cascade;
 
create table parents(k int primary key, v text not null);
 
create table children(
  parent_k  int  not null,
  k         int  not null,
  v         text not null,
 
  constraint children_pk primary key(parent_k, k),
 
  constraint children_fk foreign key(parent_k)
    references parents(k)
    match full
    on delete cascade
    on update restrict);
 
insert into parents(k, v) values (1, 'dog'), (2, 'cat'), (3, 'frog');
 
insert into children(parent_k, k, v) values
  (1, 1, 'dog-child-a'),
  (1, 2, 'dog-child-b'),
  (1, 3, 'dog-child-c'),
  (2, 1, 'cat-child-a'),
  (2, 2, 'cat-child-b'),
  (2, 3, 'cat-child-c'),
  (3, 1, 'frog-child-a'),
  (3, 2, 'frog-child-b'),
  (3, 3, 'frog-child-c');
 
select p.v as "parents.v", c.v as "children.v"
from parents p inner join children c on c.parent_k = p.k
order by p.k, c.k;
 
返回信息如下：
 parents.v |  children.v
-----------+--------------
 dog       | dog-child-a
 dog       | dog-child-b
 dog       | dog-child-c
 cat       | cat-child-a
 cat       | cat-child-b
 cat       | cat-child-c
 frog      | frog-child-a
 frog      | frog-child-b
 frog      | frog-child-c
```

\d children元命令显示它对父表有一个外键约束。这使它成为该表的（传递性）依赖对象：
返回信息如下：

```
Indexes:
    "children_pk" PRIMARY KEY, lsm (parent_k HASH, k ASC)
Foreign-key constraints:
    "children_fk" FOREIGN KEY (parent_k) REFERENCES parents(k) MATCH FULL ON UPDATE RESTRICT ON DELETE CASCADE
```

请注意，on delete cascade 子句的作用仅限于delete语句的作用，它对截断的行为没有影响。没有on truncate cascade 子句，请尝试delete from parents。执行成功，并从parents 表和children 表中删除了所有行。
对于上面插入的所有行，请尝试以下操作： 

```
do $body$
declare
  message  text not null := '';
  detail   text not null := '';
begin
  -- Causes error 'cos "cascade" is required.
  truncate table parents;
  assert false, 'Should not get here';
exception
  -- Error 0A000
  when feature_not_supported then
    get stacked diagnostics
      message  = message_text,
      detail   = pg_exception_detail;
    assert message = 'cannot truncate a table referenced in a foreign key constraint',  'Bad message';
    assert detail  = 'Table "children" references "parents".',                          'Bad detail';
end;
$body$;
```

它完成时没有出现错误，表明没有cascade的truncate table parents失败，现在用cascade重复该尝试，并观察结果： 

```
truncate table parents cascade;
 
select
  (select count(*) from parents) as "parents count",
  (select count(*) from children) as "children count";
```

truncate语句现在无错误地完成。返回信息如下：

```
 parents count | children count
---------------+----------------
             0 |              0
```

最后，再次尝试truncate table parents。正如承诺的那样，即使传递依赖表children是空的，它仍然会以0A000错误失败。 

## **UPDATE**

更新一个表的行
语法：
update ::= [ with_clause ]  UPDATE table_expr [ [ AS ] alias ]  SET 
           update_item [ , ... ] [ WHERE boolean_expression
                                   | WHERE CURRENT OF cursor_name ]  
           [ returning_clause ]

returning_clause ::= RETURNING { * | { output_expression 
                                     [ [ AS ] output_name ] } 
                                     [ , ... ] }

update_item ::= column_name = column_value
                | ( column_names ) = [ ROW ] ( column_values )
                | ( column_names ) = subquery

column_values ::= { expression | DEFAULT } [ , ... ]

column_names ::= column_name [ , ... ]
描述：使用UPDATE语句可以修改所有符合特定条件的行中指定列的值，如果WHERE子句中没有提供条件，则会更新所有行。UPDATE输出更新的行数。

参数：
with_clause：WITH子句允许你指定一个或者更多个在 UPDATE中可用其名称引用的子查询。
table_expr：要更新的表的名称（可以是模式限定的）。
alias:在UPDATE语句中指定目标表的标识符。当指定了别名时，必须使用它来代替语句中的实际表。
column_name:更新表的列的名称。
expression：指定要分配给列的值，要被赋值给该列的一个表达式，该表达式可以使用该表中这一列或者其他列的旧值。
output_expression：在每一行被更新后，要被UPDATE命令计算并且返回的表达式。该表达式可以使用 table_name指定的表或者FROM列出的表中的任何列名。写* 可以返回所有列。
subquery：指定SELECT子查询语句。其选定的值将分配给指定的列。

示例：
新建表，并插入一些数据，然后，再进行更新这些数据。

```
CREATE TABLE sample(k1 int, k2 int, v1 int, v2 text, PRIMARY KEY (k1, k2));
INSERT INTO sample VALUES (1, 2.0, 3, 'a'), (2, 3.0, 4, 'b'), (3, 4.0, 5, 'c');
SELECT * FROM sample ORDER BY k1;
```

返回信息如下：

```
 k1 | k2 | v1 | v2 
----+----+----+----
  1 |  2 |  3 | a
  2 |  3 |  4 | b
  3 |  4 |  5 | c
```

```
UPDATE sample SET v1 = v1 + 3, v2 = '7' WHERE k1 = 2 AND k2 = 3;
SELECT * FROM sample ORDER BY k1;
```

返回信息如下：

```
 k1 | k2 | v1 | v2 
----+----+----+----
  1 |  2 |  3 | a
  2 |  3 |  7 | 7
  3 |  4 |  5 | c
```

## **VALUES**

计算一个行集合
语法：
values ::= VALUES ( expression_list ) [ ,(expression_list ... ]  
           [ ORDER BY { order_expr [ , ... ] } ]  
           [ LIMIT { int_expression | ALL } ]  
           [ OFFSET int_expression [ ROW | ROWS ] ]  
           [ FETCH { FIRST | NEXT } int_expression { ROW | ROWS } ONLY ]

expression_list ::= expression [ , ... ]
描述：VALUES计算由值表达式指定的一个行值或者一组行值。

参数：
expression_list:要在结果表（行集合）中指定位置计算并且插入的一个常量或者表达式。 逗号分隔的带括号的表达式列表。 形如：

```
values ('dog'::text);
```

返回信息如下：

```
 column1 
---------
 dog
```

结果具有与命名为“column1”、“column2”、…“columnN”的列一样多的列，因此： 

```
values
  (1::int, '2019-06-25 12:05:30'::timestamp, 'dog'::text),
  (2::int, '2020-07-30 13:10:45'::timestamp, 'cat'::text);
```

返回信息如下：

```
 column1 |       column2       | column3 
---------+---------------------+---------
       1 | 2019-06-25 12:05:30 | dog
       2 | 2020-07-30 13:10:45 | cat
```

如果一个表达式是在没有类型转换的情况下编写的，则会推断出它的数据类型。例如，“dog”被推断为具有数据类型text，4.2被推断为数据类型numeric。
每个连续的带括号的表达式列表必须指定相同数量的具有相同数据类型的表达式。试试这个示例：

```
values
  (1::int, '2019-06-25 12:05:30'::timestamp, 'dog'::text),
  (2::int, '2020-07-30 13:10:45'::timestamp, 'cat'::text, 42::int);
```

返回错误信息如下：

```
 VALUES lists must all be the same length
```

而这个：values (1::int), ('x'::text);
返回错误信息如下：

```
  VALUES types integer and text cannot be matched
```

ORDER BY, LIMIT, OFFSET, 与FETCH：当在VALUES语句中使用这些子句时，它们的语义与在SELECT语句中使用它们时的语义相同。

示例：
示例1：VALUES语句可以用作子查询，方法是用括号将其括起来，并给它一个别名，就像SELECT语句可以被如此引用使用一样。

```
select chr(v) as c from (
  select * from generate_series(97, 101)
  ) as t(v);
```

返回信息如下：

```
 c 
---
 a
 b
 c
 d
 e
```

示例2：现在在括号中使用VALUES语句（第2行），而不是SELECT语句： 

```
select chr(v) as c from (
  values (100), (111), (103)
  ) as t(v);
```

返回信息如下：

```
 c
---
 d
 o
 g
```