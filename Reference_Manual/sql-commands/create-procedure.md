CREATE PROCEDURE
=====

创建存储过程。

:::info 重要提示
如果查询中包含用户自定义的存储过程（通过 `CREATE PROCEDURE` 语句创建），那么该查询只能在 Hybrid DPS 集群上执行。
:::

---

语法
--------

```sql
CREATE [OR REPLACE] PROCEDURE <name>    
    ( [ [<arg_mode>] [<arg_name>] <arg_type> [ { DEFAULT | = } <default_expr> ] [, ...] ] )
  { LANGUAGE <lang_name>
    | TRANSFORM { FOR TYPE <type_name> } [, ... ]
    | { [ EXTERNAL ] SECURITY INVOKER | [ EXTERNAL ] SECURITY DEFINER }
    | SET <config_param> { TO <value> | = <value> | FROM CURRENT }
    | AS '<definition>'
  } ...
```

---
描述
----------

要创建存储过程，你必须拥有需要使用的参数类型的 `USAGE` 权限。默认情况下，你是自己创建的存储过程的所有者。

MAXIR 使用存储过程的名称和输入参数类型来唯一标识一个存储过程。因此，同一 Schema 中可以存在名称相同但输入参数类型不同的存储过程。

`CREATE PROCEDURE` 只能用于创建存储过程。如果你想替换现有的存储过程，使用 `CREATE OR REPLACE PROCEDURE`。

只有存储过程的所有者才能运行 `CREATE OR REPLACE PROCEDURE` 来替换存储过程。命令运行后，存储过程的所有者和权限保持不变，而其他参数使用命令中指定的值。

更多关于创建存储过程的信息，请参阅 PostgreSQL 文档中的 [User Defined Procedures](https://www.postgresql.org/docs/12/xproc.html)。


:::info
如果你想更新存储过程的名称或参数类型，删除存储过程并重新创建该存储过程。如果你尝试通过 `CREATE OR REPLACE PROCEDURE` 来完成这一操作，将会创建一个新的存储过程，而不是替换现有的存储过程。
:::

---

参数
----------

- _`<name>`_
    
    存储过程的名称。你可以使用 Schema 限定，在指定的 Schema 中创建存储过程。否则，存储过程将在当前 Schema 中创建。

- *`<arg_mode>`*

    参数的模式。可用的值有 `IN`、`INOUT` 和 `VARIADIC`。如果省略此参数，将使用默认值 `IN`。 

    目前，存储过程不支持 `OUT` 参数，请改用 `INOUT`。

- *`<arg_name>`*

    参数的名称。

- *`<arg_type>`*

    参数的数据类型。你可以使用 Schema 来进行限定。支持的数据类型可以是基础类型、组合类型和域类型，也可以参考表列的数据类型。

    对于某些实现语言，也允许使用伪类型，这样你就可以指定像 `cstring` 这样的伪类型。伪类型表明实际的参数类型要么未完全指定，要么超出了常规 SQL 数据类型的集合。
    
    当你引用列的数据类型时，可以使用 *table_name_._column_name*__%TYPE__ 格式来指定。这样做，你创建的存储过程可以独立于表定义的更改。

- *`<default_expr>`*

    如果参数未指定，生成默认值的表达式。表达式必须可强制转换为参数的参数类型。请注意，只有 `IN` 和 `INOUT` 参数可以有默认值。 

    在参数列表中，每个跟随有默认值的参数的 `IN` 或 `INOUT` 参数必须有默认值。 

- *`<lang_name>`*

    实现存储过程所用的语言名称。值可以是 `SQL`、`C`、`internal`，或用户定义的过程语言的名称。  

- **`TRANSFORM { FOR TYPE <type_name> } [, ... ]`** 

    存储过程调用必须应用的转换列表。转换用于在 SQL 类型和特定语言的数据类型之间转换。在大多数情况下，内置类型在过程语言实现中是硬编码的。因此，此参数是不必要的。即使特定的过程实现不知道如何处理类型，并且没有提供转换，它也会在转换数据类型时使用默认行为。默认行为随实现而异。

- **`[EXTERNAL] SECURITY INVOKER`** 或 **`[EXTERNAL] SECURITY DEFINER`**

    需要哪个用户的权限来运行存储过程。
    
    - `SECURITY INVOKER`：调用存储过程的用户。
    - `SECURITY DEFINER`：创建存储过程的用户。

    `EXTERNAL` 关键字是可选的，因为这个选项对 MAXIR 中的所有函数都是可用的，而不仅仅是外部函数。你可以指定它以符合 SQL 标准。

- *`<config_param>`* 和 *`<value>`*

    你可以使用 `SET` 子句为存储过程指定会话配置参数的值。以这种方式指定的配置参数设置仅对当前会话有效。存储过程退出时，配置参数的值将恢复到原始值。 

    `SET FROM CURRENT` 保存配置参数的当前值作为存储过程进入时要应用的值。

    如果存储过程有一个 `SET` 子句，存储过程内部执行的任何针对同一变量的 `SET LOCAL` 命令仅适用于存储过程。一旦存储过程退出，配置参数将恢复到其原始值。然而，普通的 `SET` 命令（没有 `LOCAL`）会覆盖 `SET` 子句，就像它会覆盖以前的 `SET LOCAL` 命令一样。此命令的效果将在存储过程退出后持续，除非当前事务被回滚。

    更多详情，请参阅 [SET](set.md)。

- *`<definition>`*

    存储过程的定义。值必须是一个字符串常量，表示内部存储过程名称、对象文件的路径、SQL 命令，或者基于用于实现存储过程的语言的文本。

    我们建议你使用美元引用来指定 *`<definition>`* 的值，否则你必须在存储过程定义中通过加倍它们来转义单引号（' '）或反斜杠（\\）。关于美元引用的详细信息，请参阅 PostgreSQL 文档中的 [Dollar-Quoted String Constraints](https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-DOLLAR-QUOTING)。


---

示例
-------------

```sql
CREATE PROCEDURE insert_data(a integer, b integer)
LANGUAGE SQL
AS $$
INSERT INTO tbl VALUES (a);
INSERT INTO tbl VALUES (b);
$$;

CALL insert_data(1, 2);
```


---

使用注意事项
-------------

更多信息，请参阅 [CREATE FUNCTION](create-function.md)。

要执行存储过程，请运行 [CALL](call.md)。


---

SQL 标准兼容性
-------------

MAXIR 中的 `CREATE PROCEDURE` 与标准 SQL 中的 `CREATE PROCEDURE` 部分兼容。关于两个版本之间的差异的详细信息，请参阅 [CREATE FUNCTION](create-function.md)。
