与psql类似，sqlsh提供了许多元命令，使sqlsh更适合管理或编写脚本。元命令通常称为斜杠或反斜杠命令。您在sqlsh中输入的任何以未加引号的反斜杠开头的内容都是由sqlsh自身处理的元命令。

注意：
cloud shell
出于安全原因，AiSQL托管云外壳只能访问元命令的一个子集。除了以只读方式访问/share目录以加载示例数据集之外，访问文件系统的命令在cloud shell中不起作用。

## **基本语法**

sqlsh命令的格式是用反斜线后面直接跟上一个命令动词，然后是一些参数。参数与命令动词和其他参数之间用任意多个空白字符分隔开。

要在一个参数中包括空白，可以将它加上单引号(' ')。要在一个参数中包括一个单引号，则需要在文本中写上两个单引号 (' ... '' ...')。任何包含在单引号中的东西都服从与 C 语言中\n（新行）、\t（制表符）、\b（退格）、\r（回车）、\f（换页）、\digits（10 进制）以及\xdigits（16 进制）类似的替换规则。单引号内文本中的其他任何字符（不管它是什么）前面的反斜线都没有实际意义（会被忽略）。

如果在一个参数中出现一个未加引号的冒号（:）后面跟着一个sqlsh变量名，它会被该变量的值替换，如[SQL 中插入变量中](#_SQL中插入变量)所述。在其中描述的形式:'variable_name'和:"variable_name"也有同样的效果。

在一个参数中，封闭在反引号（`）中的文本会被当做一个传递给shell的命令行。该命令的输出（移除任何拖尾的新行）会替换反引号文本。在封闭在反引号的文本中，不会有特别的引号或者其他处理发生，:variable_name的出现除外，其中variable_name是一个会被其值替换的sqlsh变量名。此外，:'variable_name'的出现会被替换为该变量的值，而值会被适当地加以引用以变成一个单一shell命令参数（后一种形式几乎总是优先，除非你非常确定变量中有什么）。因为回车和换行字符在所有的平台上都不能被安全地引用，:'variable_name'形式会打印一个错误消息并且在这类字符出现在值中时不替换该变量值。

有些命令把SQL标识符（例如一个表名）当作参数。这些参数遵循SQL的语法规则：无引号的字母被强制变为小写，而双引号（"）可以保护字母避免大小写转换并且允许在标识符中包含空白。 在双引号内，成对的双引号会被缩减为结果名称中的单个双引号。例如，FOO"BAR"BAZ会被解释成fooBARbaz，而"A weird"" name"会变成A weird" name。

对参数的解析会在行尾或者碰到另一个未加引号的反斜线时停止。一个未加引号的反斜线被当做新元命令的开始。特殊的序列\\（两个反斜线）表示参数结束并且应继续解析SQL命令（如果还有）。使用这种方法，SQL命令和sqlsh命令可以被自由地混合在一行中。但是无论在何种情况中，元命令的参数都无法跨越一行。

很多元命令作用在当前查询缓冲区上。这就是一个缓冲区而已，它保存任何已经被键入但是还没有发送到服务器执行的SQL命令文本。这将包括之前输入的行以及在该元命令同一行上出现在前面的任何文本。

可以使用下列元命令：

## **\a**

如果当前的表输出格式是非对齐的，则切换成对齐格式。如果不是非对齐格式，则设置成非对齐格式。保留这个命令是为了向后兼容性。更一般的方案请见\pset。

## **\c or \connect [ -reuse-previous=on|off ] [ dbname [ username ] [ host ] [ port ] | conninfo ]**

与一台AiSQL服务器建立一个新连接。可以使用位置语法指定要使用的连接参数，或者使用conninfo连接串。

在省略了数据库名、用户、主机或者端口的命令中，新的连接将会重用之前一个连接的值。默认情况下，前一个连接的值将会被重用，除非给出了一个conninfo串。给出第一个参数-reuse-previous=on或者-reuse-previous=off可以覆盖默认行为。当这个命令既没有指定一个参数也没有重用它时，将使用libpq的默认值。把dbname、username、host或者port中的任何一个指定为-等价于省略该参数。

如果新连接成功地被建立，之前的连接会被关闭。如果连接尝试失败（错误的用户名、访问被拒绝等），只有在sqlsh处于交互模式的情况下才会保留之前的连接。当执行一个非交互式脚本时出现连接尝试失败，处理将被立即停止，并且报出一个错误。这种区别一方面可以帮助用户发现打字错误，另一方面也可以作为一种安全机制防止脚本在错误的数据库上执行动作。

例子：
=> \c mydb myuser host.dom 6432
=> \c service=foo
=> \c "host=localhost port=5432 dbname=mydb connect_timeout=10 sslmode=disable"
=> \c postgresql://tom@localhost/mydb?application_name=myapp

## **\C [title]**

设置作为查询结果打印的任何表的标题，或取消设置任何此类标题。此命令相当于\pset title。（此命令的名称源自“caption”，因为它以前只用于在HTML表中设置标题。）

## **\cd [ directory ]**

把当前工作目录改为directory。如果不带参数，则切换到当前用户的主目录。
提示：
要打印当前的工作目录，可以使用\! pwd

## **\conninfo**

输出有关当前数据库连接的信息，包括数据库，用户，和端口。

## **\copy { table [ ( column_list ) ] | ( query ) } { from | to } { 'filename' | program 'command' | stdin | stdout | pstdin | pstdout } [ [ with ] ( option [, ...] ) ]**

执行一次前端拷贝。这个操作会运行一个SQL COPY命令，不过不是服务器读取或者写入指定的文件，而是由sqlsh读写文件并且把数据从本地文件系统导向服务器。这意味着文件的可访问性和权限是本地用户的而非服务器上的，并且不需要 SQL 超级用户特权。

当program被指定时，command被sqlsh执行并且传给command的数据或者从command传出的数据会在服务器和客户端之间流动。同样地，执行特权是本地用户的而非服务器上的，并且不需要 SQL 超级用户特权。

对于\copy ... from stdin，数据行从发出该命令的同一来源读取，一直到读到 \. 或者数据流到达EOF。这个选项可以用来填充内嵌在一个 SQL 脚本文件中的表。对于\copy ... to stdout，输出被发送到与sqlsh命令输出相同的位置，并且COPY count命令的状态不会被打印（因为它会被一个数据行搞乱）。要读/写sqlsh的标准输入或者输出而不管当前命令的来源或者\o选项，可以写from pstdin或者to pstdout。

这个命令的语法和SQL COPY命令类似。所有除开数据来源/目的地的选项都和COPY指定的一样。因此，\copy元命令由特殊的解析规则。与大部分其他元命令不同，该行的所有剩余部分总是会被当做\copy的参数，并且在参数中不会执行变量篡改以及反引号展开。

提示：
获得与\copy ... to相同结果的另一种方法是使用SQL COPY ... TO STDOUT 命令并使用\g filename或\ g | program 终止它。
与\copy不同，此方法允许命令跨越多行; 此外，可以使用变量插值和反引号扩展。
这些操作不如带有文件或程序数据源或目标的SQL COPY命令有效， 因为所有数据都必须通过客户端/服务器连接。 对于大量数据，SQL命令可能更可取。

## **\copyright**

显示PostgreSQL的版权以及发布条款，AiSQL源于PostgreSQL。

## **\crosstabview [ colV [ colH [ colD [ sortcolH ] ] ] ]**

执行当前的查询缓冲区（像\g那样）并且在一个交叉表格子中显示结果。该查询必须返回至少三列。由colV标识的输出列会成为垂直页眉，并且colH所标识的输出列会成为水平页眉。colD标识显示在格子中的输出列。sortcolH标识用于水平页眉的可选的排序列。

每一个列说明可以是一个列编号（从 1 开始）或者一个列名。常用的 SQL 大小写折叠和引用规则适用于列名。如果省略，colV被当做列 1 ，并且colH被当做列 2。colH必须和colV不同。如果没有指定colD，那么在查询结果中必须正好有三列，并且colV和colH之外的那一列会被当做colD。

垂直页眉显示为最左边的列，它包含列colV中找到的值，值的顺序和查询结果中的顺序相同，但是重复值会被移除。

水平页眉显示为第一行，它包含列colH中找到的值，其中的重复值被移除。默认情况下，这些值会以查询结果中相同的顺序出现。但是如果给出了可选的sortcolH参数，它标识一个值必须为整数编号的列，并且来自colH的值将会根据相应的sortcolH值排序后出现在水平页眉中。

在交叉表格子中，对于colH的每一个不同的值x，以及colV的每一个不同的值y，位于交叉点(x,y)的单元包含colH值为x且colV值为y的查询结果行中colD列的值。如果没有这样的行，则该单元为空。如果有多个这样的行，则会报告一个错误。

## **\d[S+] [** [pattern](#_模式（Pattern）)**]**

对于每一个匹配pattern的关系（表、视图、物化视图、索引、序列或者外部表）或者组合类型，显示所有的列、它们的类型、表空间（如果非默认表空间）以及任何诸如NOT NULL或者默认值的特殊属性。相关的索引、约束、规则以及触发器也会被显示。对于外部表，还会显示相关的外部服务器（下文的模式（Pattern）中定义了“匹配模式”）。

对于某些类型的关系，\d会为每一列显示额外的信息：对于序列会显示列值，对于索引显示被索引的表达式，对于外部表显示外部数据包装器选项。

命令形式\d+是一样的，不过会显示更多信息：与该表的列相关的任何注释，表中是否存在 OID，如果关系是视图则显示视图定义。

默认情况下只会显示用户创建的对象，提供一个模式或者S修饰符可以把系统对象包括在内。

注意
如果使用\d但不带有pattern参数，它等价于\dtvmsE，后者将显示所有可见的表、视图、物化视图、序列和外部表的列表。

## **\da[S] [ pattern ]**

列出聚集函数，以及它们的返回类型和它们所操作的数据类型。如果指定了pattern，只显示名称匹配该模式的聚集。默认情况下只会显示用户创建的对象，提供一个模式或者S修饰符可以把系统对象包括在内。

## **\dA[+] [** [pattern](#_模式（Pattern）)**]**

列出访问方法。如果指定了pattern，只显示名称匹配该模式的访问方法。如果在命令名称后面追加+，则与访问方法相关的处理器函数和描述也会和访问方法本身一起被列出。

## **\db[+] [** [pattern](#_模式（Pattern）)**]**

列出表空间。如果指定了pattern，只显示名称匹配该模式的表空间。如果在命令名称后面追加+，则与表空间相关的选项、磁盘上的尺寸、权限以及描述也会和表空间本身一起被列出。

## **\dc[S+] [** [pattern](#_模式（Pattern）)**]**

列出字符集编码之间的转换。如果指定了pattern，只列出名称匹配该模式的转换。默认情况下只会显示用户创建的对象，提供一个模式或者S修饰符可以把系统对象包括在内。如果在命令名称后面追加+，则每一个对象相关的描述也会被列出。

## **\dC[+] [** [pattern](#_模式（Pattern）)**]**

列出类型转换。如果指定了pattern，只列出源类型和目标类型匹配该模式的转换。如果在命令名称后面追加+，则每一个对象相关的描述也会被列出。

## **\dd[S] [** [pattern](#_模式（Pattern）)**]**

显示约束、操作符类、操作符族、规则以及触发器类型对象的描述。所有其他注释可以通过那些对象类型相应的反斜线命令查看。

\dd显示匹配pattern的对象的描述，如果没有给出参数则显示合适类型的可见对象的描述。但是在任一种情况下都只列出具有描述的对象。默认情况下只会显示用户创建的对象，提供一个模式或者S修饰符可以把系统对象包括在内。

对象的描述可以用SQL命令COMMENT创建。

## **\dD[S+] [** [pattern](#_模式（Pattern）)**]**

列出域。如果指定了pattern，只有名称匹配该模式的域会被显示。默认情况下，只有用户创建的对象会被显示，可以提供一个模式或者S修饰符以包括系统对象。如果+被追加到命令名称上，每一个被列出的对象会带有其相关的权限和描述。

## **\ddp [** [pattern](#_模式（Pattern）)**]**

列出默认的访问特权设置。对那些默认特权设置已经被改变得与内建默认值不同的角色（以及模式，如果适用），为每一个角色（以及模式）显示一项。如果指定了pattern，只列出角色名称或者模式名称匹配该模式的项。

ALTER DEFAULT PRIVILEGES命令被用来设置默认访问特权。

## **\dE[S+], \di[S+], \dm[S+], \ds[S+], \dt[S+], \dv[S+] [** [pattern](#_模式（Pattern）)**]**

在这一组命令中，字母E、i、m、s、t和v分别对应着外部表、索引、物化视图、序列、表和视图。你可以以任何顺序指定这些字母中的任意一个或者多个，这样可以得到这些类型的对象的列表。例如，\dit会列出索引和表。如果在命令名称后面追加+，则每一个对象的物理尺寸以及相关的描述也会被列出。如果指定了pattern，只列出名称匹配该模式的对象。默认情况下只会显示用户创建的对象，提供一个模式或者S修饰符可以把系统对象包括在内。

## **\des[+] [** [pattern](#_模式（Pattern）)**]**

列出外部服务器（助记：“外部服务器”）。如果指定了pattern，只列出名称匹配该模式的那些服务器。如果使用了\des+形式，将显示每个服务器的完整描述，包括该服务器的访问特权、类型、版本、选项和描述。

## **\det[+] [** [pattern](#_模式（Pattern）)**]**

列出外部表（助记：“外部表”）。如果指定了pattern，只列出表名称或者模式名称匹配该模式的项。如果使用了\det+选项，一般选项和外部表描述也会被显示。

## **\deu[+] [** [pattern](#_模式（Pattern）)**]**

列出用户映射（助记：“外部用户”）。如果指定了pattern，只列出用户名匹配该模式的那些映射。如果使用了\deu+形式，有关每个映射的额外信息也会被显示。
注意：
\deu+可能也会显示远程用户的用户名和口令，所以要小心不要把它们泄露出去。

## **\dew[+] [** [pattern](#_模式（Pattern）)**]**

列出外部数据包装器（助记：“外部包装器”）。如果指定了pattern，只列出名称匹配该模式的那些外部数据包装器。如果使用了\dew+形式，外部数据包装器的访问特权、选项和描述也会被显示。

## **\df[antwS+] [** [pattern](#_模式（Pattern）)**]**

列出函数，以及它们的结果数据类型、参数数据类型和函数类型，函数类型被分为“agg”（聚集）、“normal”、“trigger”以及“window”。如果要只显示指定类型的函数，可以在该命令上增加相应的字母a、n、t或者w。如果指定了pattern，只显示名称匹配该模式的函数。默认情况下只会显示用户创建的对象，提供一个模式或者S修饰符可以把系统对象包括在内。如果使用了\df+形式，则有关每个函数的额外信息也会被显示，包括易失性、并行安全性、拥有者、安全性分类、访问特权、语言、源代码和描述。

提示
如果要查找接收指定数据类型参数或者返回指定类型值的函数，可以使用分页器的搜索能力来滚动显示\df输出。

## **\dF[+] [** [pattern](#_模式（Pattern）)**]**

列出文本搜索配置。如果指定了pattern，只显示名称匹配该模式的配置。如果使用了\dF+形式，每种配置的完整描述也会被显示，包括底层的文本搜索解析器和用于每一种解析器记号类型的字典列表。

## **\dFd[+] [** [pattern](#_模式（Pattern）)**]**

列出文本搜索字典。如果指定了pattern，只显示名称匹配该模式的字典。如果使用了\dFd+形式，有关每一种选中的字典的额外信息也会被显示，包括底层的文本搜索模板和选项值。

## **\dFp[+] [** [pattern](#_模式（Pattern）)**]**

列出文本搜索解析器。如果指定了pattern，只显示名称匹配该模式的解析器。如果使用了\dFp+形式，每一种解析器的完整描述也会被显示，包括底层的函数和可识别的记号类型列表。

## **\dFt[+] [** [pattern](#_模式（Pattern）)**]**

列出文本搜索模板。如果指定了pattern，只显示名称匹配该模式的模板。如果使用了\dFt+形式，每一种模板有关的额外信息也会被显示，包括底层的函数名称。

## **\dg[S+] [** [pattern](#_模式（Pattern）)**]**

列出数据库角色（因为“用户”和“组”的概念已经被统一成“角色”，这个命令现在等价于\du）。默认情况下只会显示用户创建的角色，提供一个模式或者S修饰符可以把系统角色包括在内。如果指定了pattern，只列出名称匹配该模式的那些角色。如果使用了\dg+形式，有关每种角色的额外信息也将被显示，当前这种形式会为角色增加显示注释。

## **\dl**

这是\lo_list的一个别名，它显示大对象的列表。

## **\dL[S+] [** [pattern](#_模式（Pattern）)**]**

列出过程语言。如果指定了pattern，只列出名称匹配该模式的语言。默认情况下只会显示用户创建的语言，提供一个模式或者S修饰符可以把系统对象包括在内。如果向命令名称追加+，则每一种语言会和它的调用处理器、验证器、访问特权以及它是否为系统对象一起列出。

## **\dn[S+] [** [pattern](#_模式（Pattern）)**]**

列出模式（名字空间）。如果指定了pattern，只列出名称匹配该模式的模式。默认情况下只会显示用户创建的对象，提供一个模式或者S修饰符可以把系统对象包括在内。如果向命令名称追加+，每个对象会与它相关的权限及描述（如果有）一起被列出。

## **\do[S+] [** [pattern](#_模式（Pattern）)**]**

列出操作符及其操作数和结果类型。如果指定了pattern，只列出名称匹配该模式的操作符。默认情况下只会显示用户创建的对象，提供一个模式或者S修饰符可以把系统对象包括在内。如果向命令名称追加+，有关每个操作符的额外信息也将被显示，当前只包括底层函数的名称。

## **\dO[S+] [** [pattern](#_模式（Pattern）)**]**

列出排序规则。如果指定了pattern，只列出名称匹配该模式的排序规则。默认情况下只会显示用户创建的对象，提供一个模式或者S修饰符可以把系统对象包括在内。如果向命令名称追加+，每个排序规则将和它相关的描述（如果有）一起被列出。注意只有可用于当前数据库编码的排序规则会被显示，因此在同一个安装下的不同数据库中执行此命令可能会得到不同的结果。

## **\dp [** [pattern](#_模式（Pattern）)**]**

列出表、视图和序列，包括与它们相关的访问特权。如果指定了pattern，只列出名称匹配该模式的表、视图以及序列。

GRANT和REVOKE命令被用来设置访问特权。

## **\drds [ role-pattern [ database-pattern ] ]**

列出已定义的配置设置。这些设置可以是针对角色的、针对数据库的或者同时针对两者的。role-pattern和database-pattern分别被用来选择要列出的角色和数据库。如果省略它们或者指定了*，则会列出所有设置，分别会包括针对角色和针对数据库的设置。

ALTER ROLE以及ALTER DATABASE命令可以用来定义一个角色以及一个数据库的配置设置。

## **\dRp[+] [** [pattern](#_模式（Pattern）)**]**

列出复制的publication。如果指定有pattern，只有那些名称匹配该模式的publication会被列出。如果+被追加到命令的名称上，与每个publication相关的表也会被显示。

## **\dRs[+] [** [pattern](#_模式（Pattern）)**]**

列出复制的订阅。如果指定有pattern，只有那些名字匹配该模式的订阅才会被列出。如果+被追加到命令的名称上，订阅的额外属性会被显示。

## **\dT[S+] [** [pattern](#_模式（Pattern）)**]**

列出数据类型。如果指定了pattern，只列出名称匹配该模式的类型。如果向命令名称追加+，每一种类型、其内部名称和尺寸、允许的值（如果是一种枚举类型）以及相关权限会被一同列出。默认情况下只会显示用户创建的对象，提供一个模式或者S修饰符可以把系统对象包括在内。

## **\du[S+] [** [pattern](#_模式（Pattern）)**]**

列出数据库角色（因为“用户”和“组”的概念已经被统一成“角色”，这个命令现在等价于\dg）。默认情况下只会显示用户创建的角色，提供一个模式或者S修饰符可以把系统角色包括在内。如果指定了pattern，只列出名称匹配该模式的那些角色。如果使用了\du+形式，有关每一种角色的额外信息也会被显示，当前只会多显示角色的注释。

## **\dx[+] [** [pattern](#_模式（Pattern）)**]**

列出已安装的扩展。如果指定了pattern，只列出名称匹配该模式的那些扩展。如果使用了\dx+形式，所有属于每个匹配扩展的对象会被列出。

## **\dy[+] [** [pattern](#_模式（Pattern）)**]**

列出事件触发器。如果指定了pattern，只列出名称匹配该模式的事件触发器。如果在命令名称后面加上+，还会为每个列出的对象显示其相关的描述。

## **\e或\edit [ filename ] [ line_number ]**

如果指定了filename，则它是被编辑的文件，在编辑器退出后，该文件的内容会被拷贝到当前查询缓冲区中。如果没有给定filename，当前查询缓冲区会被拷贝到一个临时文件中，并且接着以相同的方式编辑。或者，如果当前查询缓冲区为空，则最近被执行的查询会被拷贝到一个临时文件并且以同样的方式编辑。

然后会根据sqlsh的一般规则重新解析查询缓冲区的新内容，把整个缓冲区当作一个单一行来处理。任何完整的查询都会被立即执行，也就是说，如果查询缓冲区包含一个分号或者以一个分号结尾，则到分号处为止的所有东西都会被执行。剩下的东西会在查询缓冲区中等待，输入分号或者\g会把它发送出去，输入\r会通过清除查询缓冲区来取消它。把缓冲区当作单一行主要会影响元命令：缓冲区中在一个元命令之后的任何东西都将被当作该元命令的参数，即便元命令之后的内容跨越多行也是如此。（因此不能以这种方式来制作使用元命令的脚本。应该使用\i。）

如果指定了一个行号，sqlsh将会把游标（注意不是服务器端的游标）定位到文件或者查询缓冲区的指定行上。注意如果给出了一个全是数字的参数，sqlsh就会假定它是行号而不是文件名。

## **\echo text [ ... ]**

把参数打印到标准输出，参数之间用一个空格分隔，最后加上一个新行。这可以用来在脚本的输出中间混入信息，例如：

=> \echo `date`
Tue Oct 26 21:40:57 CEST 1999
如果第一个参数是一个没有加引号的-n，则不会加上最后的新行。

提示：
如果使用\o命令来重定向查询的输出，你可能希望使用\qecho来取代这个命令。

## **\ef [ function_description [ line_number ] ]**

这个命令会以一个CREATE OR REPLACE FUNCTION或CREATE OR REPLACE PROCEDURE命令的形式取出并且编辑指定函数或过程的定义。编辑的方式与\edit完全相同。在编辑器退出后，更新过的命令将在查询缓冲区中等待，可以键入分号或者\g把它发出，也可以用\r取消之。

目标函数可以单独用名称或者用名称和参数（例如foo(integer, text)）来指定。如果有多于一个函数具有同样的名称，则必须给出参数的类型。

如果没有指定函数，将会给出一个空白的CREATE FUNCTION模板来编辑。

如果指定了一个行号，sqlsh将把游标定位在该函数体的指定行上（注意函数体通常不是开始于文件的第一行）。

和大部分其他元命令不同，该行的整个剩余部分总是会被当作\ef的参数，并且在参数中不会执行变量篡改以及反引号展开。

## **\encoding [ encoding ]**

设置客户端字符集编码。如果没有参数，这个命令会显示当前的编码。

## **\errverbose**

以最详细的程度重复最近的服务器错误消息，就好像VERBOSITY被设置为verbose且SHOW_CONTEXT被设置为always。

## **\ev [ view_name [ line_number ] ]**

这个命令会以一个CREATE OR REPLACE VIEW的形式取出并且编辑指定函数的定义。编辑的方式与\edit完全相同。在编辑器退出后，更新过的命令将在查询缓冲区中等待，可以键入分号或者\g把它发出，也可以用\r取消之。

如果没有指定函数，将会给出一个空白的CREATE VIEW模板来编辑。

如果指定了一个行号，sqlsh将把游标定位在该视图定义的指定行上。

和大部分其他元命令不同，该行的整个剩余部分总是会被当作\ev的参数，并且在参数中不会执行变量篡改以及反引号展开。

## **\f [ string ]**

设置用于非对齐查询输出的域分隔符。默认值是竖线（|）。它等效于\pset fieldsep。

## **\g [ filename ]，\g [ |command ]**

把当前查询缓冲区发送给服务器执行。如果给出一个参数，查询的输出将被写到提到的文件或者用管道导向给定的shell命令，而不是按照惯常显示出来。只有该查询成功地返回零或更多个元组时才会写文件或命令，如果查询失败或者不是一个数据返回SQL命令，则不会写文件或者导向shell命令。

如果当前查询缓冲区为空，则重新执行最近一次被发送的查询。除了这种行为之外，没有参数的\g实际上等效于一个分号。一个带有参数的\g是\o命令的一种“一次性”选择。

如果该参数以 | 开始，则该行的所有剩余部分总是会被当做要执行的command，并且在参数中不会执行变量篡改以及反引号展开。该行的剩余部分会被简单地按字面传给shell。

## **\gexec**

把当前查询缓冲区发送到服务器，然后该查询输出（如果有）中的每一行的每一列都当作一个要被执行的 SQL 语句。例如，要在my_table的每一列上都创建一个索引：

```
=> select format('create index on my_table(%I)', attname)
-> FROM pg_attribute
-> WHERE attrelid = 'my_table'::regclass AND attnum > 0
-> ORDER BY attnum
-> \gexec
CREATE INDEX
CREATE INDEX
CREATE INDEX
CREATE INDEX
```

产生的查询会按照其所在行被返回的顺序执行，如果有多个列，则同一行中按照从左至右的顺序执行。NULL 域会被忽略。产生的查询会被原样发送给服务器处理，因此它们即不能是sqlsh 元命令，也不能包含sqlsh 变量引用。如果其中任何一个查询失败，剩余查询的执行将会继续，除非设置了ON_ERROR_STOP。每个查询的执行都遵照ECHO的处理（在使用\gexec时，通常建议设置ECHO为all或者queries）。查询日志、单步模式、计时以及其他查询执行特性也适用于每一个生成的查询。

如果当前查询缓冲区为空，则会重新执行最近被发送的查询。

## **\gset [ prefix ]**

把当前查询输入缓冲区发送给服务器并且将查询的输出存储在sqlsh 变量中（见[变量](#_变量)）。被执行的查询必须只返回一行。该行的每一列会被存储到一个单独的变量中，变量和该列的名字一样。例如：

```
=> select 'hello' AS var1, 10 AS var2
-> \gset
=> \echo :var1 :var2
hello 10
```

如果指定了一个prefix，那么该字符串会被追加在该查询的输出列名称之前用来创建要使用的变量名：

```
=> select 'hello' AS var1, 10 AS var2
-> \gset result_
=> \echo :result_var1 :result_var2
hello 10
```

如果一个列的结果为 NULL，那么对应的变量会被重置而不是被设置。
如果查询失败或者没有返回一行，则不会有任何变量被更改。
如果当前查询缓冲区为空，则重新执行最近被发送的查询。

## **\gx [ filename ]，\gx [ |command ]**

\gx等效于\g，但会为这个查询强制扩展的输出模式。请参考\x。

## **\h or \help [ command ]**

给出指定SQL命令的语法帮助。如果没有指定command，则sqlsh会列出可以显示语法帮助的所有命令。如果command是一个星号（*），则会显示所有SQL命令的语法帮助。

与大部分其他元命令不同，该行的所有剩余部分总是会被当做\help的参数，并且在参数中不会执行变量篡改以及反引号展开。
注意
为了简化输入，由几个词构成的命令不需要被加上引号。因此，键入\help alter table是可以的。

## **\H or \html**

开启HTML查询输出格式。如果HTML格式已经开启，这会把它切换回默认的对齐文本格式。这个命令是为了兼容性和方便，有关设置其他输出选项请见\pset。

## **\i or \include filename**

从文件filename读取输入并且把它当作从键盘输入的命令来执行。

如果filename是-（连字符），那么会一直读取标准输入直到碰到一个 EOF 指示符或者\q元命令。这可以用来把交互式输入与文件输入混杂。注意只有在最外层激活了 readline 行为的情况下才将会使用 readline 行为。

注意
如果想在屏幕上看到被读入的行，必须把变量ECHO设置成all。

## **\if expression，\elif expression，\else，\endif**

这一组命令实现可嵌套的条件块。条件块必须以一个\if开始并且以一个\endif结束。两者之间可能有任意数量的\elif子句，后面也可能有选择地跟着一个单一的\else子句。一般查询以及其他类型的反斜线命令可以出现在这些命令之间构成条件块。

\if和\elif命令读取它们的参数并且将它们作为布尔表达式进行计算。如果表达式得到真则处理正常继续下去，否则会跳过下面的行直到到达一个匹配的\elif、\else或者\endif。一旦一个\if或者\elif测试成功，同一个块中后面的\elif命令的参数将不会被计算但会被当作为假。跟在一个\else后面的行只有在先前的匹配的\if或\elif成功时才被处理。

就像任何其他反斜线命令参数一样，\if或者\elif命令的expression参数服从变量篡改以及反引号展开。然后会像一个on/off选项变量的值一样来计算它。因此，对下列项无歧义、大小写无关的匹配都是有效的值：true、false、1、0、on、off、yes、no。例如，t、T以及tR都将被认为是真。

无法被正确计算为真或假的表达式将产生一个警告并且被当做假。

正在被跳过的行还是会被正常地解析以标识查询和反斜线命令，但是查询不会被发送到服务器，并且非条件（\if、\elif、\else、\endif）反斜线命令会被忽略。条件命令会被检查以判断嵌套是否有效。被跳过的行中的变量引用不会被展开，并且也不会执行反引号展开。

一个给定条件块中的所有反斜线命令必须出现在相同的源文件中。如果在所有的本地\if块被关闭之前，主输入文件或者一个\include进来的文件上就达到了EOF，则sqlsh将产生一个错误。

这里是一个例子：

```
SELECT
    EXISTS(select 1 FROM customer WHERE customer_id = 123) as is_customer,
    EXISTS(select 1 FROM employee WHERE employee_id = 456) as is_employee
\gset
\if :is_customer
    select * FROM customer WHERE customer_id = 123;
\elif :is_employee
    \echo 'is not a customer but is an employee'
    select * FROM employee WHERE employee_id = 456;
\else
    \if yes
        \echo 'not a customer or employee'
    \else
        \echo 'this will never print'
    \endif
\endif
```

## **\ir or \include_relative filename**

\ir命令类似于\i，但是以不同的方式处理相对路径文件名。在交互模式中执行时，这两个命令的行为相同。不过，当被从脚本中调用时，\ir相对于脚本所在的目录而不是根据当前工作目录来解释文件名。

## **\l[+] or \list[+] [** [pattern](#_模式（Pattern）)**]**

列出服务器中的数据库并且显示它们的名称、拥有者、字符集编码以及访问特权。如果指定了pattern，则只列出名称匹配该模式的数据库。如果向命令名称追加+，则还会显示数据库的尺寸大小、默认表空间以及描述（尺寸大小信息只对当前用户能连接的数据库可用）。

## **\lo_export loid filename**

从数据库中读取具有OID loid的大对象并且将它写入到filename。注意这和服务器函数lo_export有微妙的不同，后者会以运行数据库服务器的用户权限来执行并且运行在服务器的文件系统上。

提示
使用\lo_list可以找出大对象的OID。

## **\lo_import filename [ comment ]**

把该文件存储到AiSQL大对象。可选地，它可以把给定的注释关联到该对象。例如：

```
foo=> \lo_import '/home/peter/pictures/photo.xcf' 'a picture of me'
lo_import 152801
```

该响应表示该大对象得到的对象 ID 是 152801，未来可以用这个 ID 来访问这个新创建的大对象。为了便于阅读，推荐总是给每一个对象都关联人类可读的注释。OID 和注释都可以用\lo_list命令查看。

注意这个命令和服务器端的lo_import有微妙的不同，因为它以本地文件系统上的本地用户的身份运行，而不是服务器用户和文件系统。

## **\lo_list**

显示当前存储在数据库中的所有AiSQL大对象，同时显示它们的任何注释。

## **\lo_unlink loid**

从数据库中删除OID为loid的大对象。

提示
使用\lo_list可以找出该大对象的OID。

## **\o or \out [ filenameor  |command ]**

安排把未来的查询结果保存到文件filename中或者用管道导向到 shell 命令command。如果没有指定参数，查询输出会被重置到标准输出。

如果该参数以 | 开始，则该行的所有剩余部分总是会被当做要执行的command，并且在参数中不会执行变量篡改以及反引号展开。该行的剩余部分会被简单地按字面传给shell。

“查询结果”包括从数据库服务器得到的所有表、命令响应和提示，还有查询数据库的各种反斜线命令（如\d）的输出，但不包括错误消息。

提示
要在查询结果之间混入文本输出，可以使用\qecho。

## **\p or \print**

把当前查询缓冲区打印到标准输出。如果当前查询缓冲区为空，会打印最近被执行的查询。

## **\password [ username ]**

更改指定用户（默认情况下是当前用户）的口令。这个命令会提示要求输入新口令、对口令加密然后把加密后的口令作为一个ALTER ROLE命令发送到服务器。这确保新口令不会以明文的形式出现在命令历史、服务器日志或者其他地方。

## **\prompt [ text ] name**

提示用户提供一个文本用于分配给变量name。可以指定一个可选的提示字符串text（对于多个词组成的提示，把文本包裹在单引号中）。

默认情况下，\prompt使用终端进行输入和输出。不过，如果使用了-f命令行开关，\prompt会使用标准输入和标准输出。

## **\pset [ option [ value ] ]**

这个命令设置影响查询结果表输出的选项。option表示要设置哪个选项。value的语义取决于选中的选项。对于某些选项，如果省略value会导致该选项值被切换或者被重置，具体是哪些选项可见特定选项的描述。如果没有上面提到的那种行为，那么省略value只会导致当前设置被显示。

不带任何参数的\pset显示所有打印选项的当前状态。

基本语法：
\pset [ option [ value ] ]

解释：
    option ：表示要设置哪个选项
    value ：语义取决于选中的选项

可调整的打印选项（option）有：
**border**
value必须是一个数字。通常，数字越大，表格就会有更多的边框和线条，但具体要看是哪一种格式。在HTML格式中，这会直接被转换成border=...属性。在大部分其他格式中，只有值 0（没有边框）、1（内部分隔线）和 2（表格边框）有意义，并且 2 以上的值会被视为与border = 2相同。latex和latex-longtable格式会额外地允许一个值 3 表示在数据行之间增加分隔线。

**columns**
为wrapped格式设置目标宽度，还有扩展自动模式中决定输出是否足够多到需要分页器或者切换到垂直显示的宽度限制。0（默认）导致目标宽度由环境变量COLUMNS所控制，如果没有设置COLUMNS则使用检测到的屏幕宽度。此外，如果columns为0，则wrapped格式只影响屏幕输出。如果columns为非零则文件和管道输出也会被包裹成该宽度。

**expanded (or x)**
如果value被指定，它必须是on或者off，它们分别会启用或者禁用扩展模式，也可以是auto。如果value被省略，则该命令会在开启和关闭设置之间切换。当扩展模式被启用时，查询结果被显示在两列中，第一列是列名而第二列是列值，列名在左边，数据在右边。如果在通常的“水平”模式中数据不适合屏幕，则可以用这种模式。在自动设置中，只要查询输出有多于一列并且比屏幕宽，就会使用扩展模式。否则，将使用常规模式。只有在对齐格式和 wrapped 格式中自动设置才有效。在其他格式中，它的行为总是像扩展模式被关闭一样。

**fieldsep**
指定在非对齐输出格式中使用的域分隔符。例如，您可以创建以制表符或逗号分隔的输出，这种形式其他程序可能更喜欢。要设置 tab 为域分隔符，可以键入\pset fieldsep '\t'。默认的域分隔符是 '|'（一个竖线）。

**fieldsep_zero**
把用在非对齐输出格式中的域分隔符设置为一个零字节。

**footer**
如果value被指定，它必须是on或者off，它们分别会启用或者禁用表格页脚（(n rows)计数）的显示。如果value被省略，则该命令会切换页脚显示为打开或者关闭。

 

**format**
设置输出格式为下列的一种： aligned, asciidoc, html, latex(使用tabular), latex-longtable, troff-ms, unaligned, 或 wrapped。允许唯一的缩写。

aligned 格式是标准，普通人可阅读的、良好格式化的文本输出；这是默认值。
unaligned格式把一个数据行的所有列都写在一行上，之间用当前活动的域分隔符分隔。这可用于生成意图由其他程序读取的输出，例如，tab 分隔或者逗号分隔格式。
wrapped格式和aligned相似，但是前者会把过宽的数据值分成多个行以便输出能够适合目标行的宽度。目标行的宽度由columns选项决定。注意sqlsh将不会尝试对列头部标题进行换行，因此如果列头部需要的总宽度超过目标宽度，wrapped格式的行为就变得和aligned一样了。
asciidoc, html,latex, latex-longtable, 和troff-ms 格式分别用相应的标记语言把要输出的表格放在文档中，不过它们的输出并不是完整的文档。 这在HTML中也许不重要，但是在LaTeX中必须有完整的文档包装器。 latex-longtable 格式需要 LaTeX、longtable 和 booktabs 包。

**linestyle**
设置边框线的绘制样式为ascii或者unicode之一。允许不产生歧义的缩写（这意味着一个字母就足够了）。默认的设置是ascii。这个选项只影响aligned以及wrapped输出格式。
ascii样式使用纯ASCII字符。数据中的新行使用一个+符号在右边的空白处显示。当在wrapped格式中包裹两行中间没有新行字符的数据时，会在第一行右边空白处显示一个点号（.），并且在下一行的左边空白处也显示一个点号（.）。
unicode样式使用 Unicode 的方框绘制字符。数据中的新行会使用一个回车符号显示在右边的空白处。当数据从一行换行到下一行而不使用换行符时，省略号将显示在第一行的右边，并再次显示在下一行的左边。
当border设置大于0时，linestyle选项也决定边框线用什么字符绘制。纯ASCII字符到处都可以使用，但是在识别 Unicode 字符的显示上使用 Unicode 字符会更好看。

**null**
设置要用来替代空值被打印的字符串。默认是什么也不打印，对于一个空字符串这很容易弄错。例如，有人可能更想用\pset null '(null)'。

 

**numericlocale**
如果value被指定，它必须是on或者off，它们将分别启用或者禁用一个与区域相关的字符来分隔数字和左边的十进制标记。如果value被省略，该命令会在常规输出和区域相关的数字输出之间切换。

**pager**
控制对查询和sqlsh的帮助输出使用分页器程序。如果环境变量PAGER被设置，输出会被用管道输送到指定的程序。否则将使用与平台相关的默认分页器程序（例如more）。

如果pager选项被设为off，则不会使用分页器程序。如果pager选项被设为on，则会在适当的时候使用分页器，即当输出到终端并且无法适合屏幕时就会使用分页器。pager选项也可以被设置为always，这会导致对所有的终端输出都是用分页器而不管输出是否适合屏幕。不带value的\pset pager会切换分页器开、关状态。

**pager_min_lines**
如果pager_min_lines被设置为一个大于页面高度的数字，在至少这么多输出行被显示之前都不会调用分页器程序。默认设置为 0。

**recordsep**
指定用在非对齐输出格式中的记录（行）分隔符。默认为换行符。

**recordsep_zero**
把用在非对齐输出格式中的记录分隔符设置为一个零字节。

**tableattr (or T)**
在HTML格式中，这会指定要放在table标记内的属性。例如，这可能是cellpadding或者bgcolor。注意你可能不想在这里指定border，因为那由\pset border负责。如果没有给出value，则表属性不会被设置。

在latex-longtable格式中，这个选项控制每个包含左对齐数据类型的列的宽度比例。这个选项的值是一个由空格分隔的值列表，例如'0.2 0.2 0.6'。没有指定的输出列会使用最后一个指定的值。

**title (or C)**
设置用于任何后续被打印表的表标题。这可以用来给输出加上描述性的标签。如果没有给出value，这个标题不会被设置。

**tuples_only (or t)**
如果value被指定，它必须是on或者off，这个选项将启用或者禁用只显示元组的模式。如果value被省略，则该命令会在常规输出和只显示元组输出之间切换。常规输出包括列头、标题以及多种页脚之类的额外信息。在只显示元组的模式中，只会显示实际的表数据。

**unicode_border_linestyle**
设置unicode线型的边框绘制风格为single或者double之一。

**unicode_column_linestyle**
设置unicode线型的列绘制风格为single或者double之一。

**unicode_header_linestyle**
设置unicode线型的页眉绘制风格为single或者double之一。

提示
\pset有多种快捷命令。请参见\a、\C、\f、\H、\t、\T以及\x。

## **\q or \quit**

退出sqlsh程序。在一个脚本文件中，只有该脚本的执行会被终止。

## **\qecho text [ ... ]**

这个命令和\echo一样，不过输出将被写到\o所设置的查询输出通道。

## **\r or \reset**

重置（清除）查询缓冲区。

## **\s [ filename ]**

打印sqlsh的命令行历史到filename。如果省略filename，该历史会被写入到标准输出（如果适用则使用分页器）。如果编译sqlsh时没有加上Readline支持，则这个命令不可用。

## **\set [ name [ value [ ... ] ] ]**

设置sqlsh变量name为value，如果给出了多于一个值，则把该变量的值设置为所有给出的值的串接。如果只给了一个参数，该变量会被设置为空字符串值。要重置一个变量，可以使用\unset 命令。

不带任何参数的\set显示所有当前设置的sqlsh变量的名称和值。

合法的变量名可以包含字母、数字和下划线。变量名是大小写敏感的。
某些变量是特殊的，它们控制sqlsh的行为，或者会被自动设置以反映连接状态。这些变量在下文的变量中记录。

注意
这个命令和SQL命令SET无关。

## **\setenv name [ value ]**

把环境变量name设置为value，如果没有提供value，则不会设置该环境变量。例如：

```
testdb=> \setenv PAGER less
testdb=> \setenv LESS -imx4F
```

## **\sf[+] function_description**

这个命令以一个CREATE OR REPLACE FUNCTION命令或者CREATE OR REPLACE PROCEDURE命令取出并且显示指定函数或者过程的定义。定义会被打印到当前的查询输出渠道，就像\o所作的那样。

目标函数可以单独用名称指定，也可以用名称和参数指定，例如foo(integer, text)。如果有多于一个函数具有相同的名字，则必须给出参数的类型。

如果向命令名称追加 + ，那么输出行会被编号，函数体的第一行会被编为 1。

与大部分其他元命令不同，该行的所有剩余部分总是会被当做\sf的参数，并且在参数中不会执行变量篡改以及反引号展开。

## **\sv[+] view_name**

这个命令以一个CREATE OR REPLACE VIEW命令取出并且显示指定视图的定义。定义会被打印到当前的查询输出渠道，就像\o所作的那样。

如果在命令名称上追加+，那么输出行会从 1 开始编号。

与大部分其他元命令不同，该行的所有剩余部分总是会被当做\sv的参数，并且在参数中不会执行变量篡改以及反引号展开。

## **\t**

切换输出列名标题和行计数页脚的显示。这个命令等效于\pset tuples_only，提供它只是为了使用方便而已。

## **\T table_options**

指定在HTML输出格式中，要放在table标签内的属性。这个命令等效于\pset tableattr table_options。

## **\timing [ on | off ]**

如果给出一个参数，这个参数用来打开或者关闭对每个SQL语句执行时长的显示。如果没有参数，则在打开和关闭之间切换。显示的数据以毫秒为单位，超过1秒的区间还会被显示为“分钟:秒”的格式，如果必要还会加上小时和日的字段。

## **\unset name**

重置（删除）sqlsh变量name。

大部分控制sqlsh行为的变量不能被重置，相反，\unset命令会被解释为把它们设置为其默认值。请参考下文的变量。

## **\w or \write** **[** **filenameor** **|command]**

把当前查询缓冲区写到文件filename或者用管道导出到 shell 命令command。如果当前查询缓冲区为空，则写最近被执行的查询。

如果参数以 | 开始，则该行的整个剩余部分会被当做要执行的command，并且在参数中不会执行变量篡改以及反引号展开。该行的剩余部分会被简单地按字面传递给shell。

## **\watch [ seconds ]**

反复执行当前的查询缓冲区（就像\g那样）直到被中止或者查询失败。两次执行之间等待指定的秒数（默认是 2 秒）。显示每个查询结果时带上一个由\pset title字符串（如果有）、从查询开始起的时间以及延时间隔组成的页眉。

如果当前查询缓冲区为空，则会重新执行最近被发送的查询。

## **\x [ on | off | auto ]**

设置或者切换扩展表格格式化模式。究其本身而言，这个命令等效于\pset expanded。

## **\z [** [pattern](#_模式（Pattern）)**]**

列出表、视图和序列，以及它们相关的访问特权。如果指定了pattern，则只会列出名称匹配该模式的表、视图和序列。

这是\dp（“display privileges”）的一个别名。

## **\! [ command ]**

如果没有参数，就跳出到一个子shell，当子shell退出时sqlsh会继续。如果有一个参数，则执行shell命令command。

与大部分其他元命令不同，该行的所有剩余部分总是会被当做 \! 的参数，并且在参数中不会执行变量篡改以及反引号展开。该行的剩余部分会被简单地按字面传递给shell。

## **\? [ topic ]**

显示帮助信息。可选的topic参数（默认是commands）选择解释sqlsh的哪一部分：commands表示sqlsh的反斜线命令；options表示可以传递给sqlsh的命令行选项；而variables显示有关sqlsh配置变量的帮助。
