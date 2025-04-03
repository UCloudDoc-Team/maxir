ALTER FUNCTION
=====

修改函数定义。

---

语法
--------

```sql
ALTER FUNCTION <name> [ ( [ [<arg_mode>] [<arg_name>] <arg_type> [, ...] ] ) ] 
   <action> [, ... ] [RESTRICT]

ALTER FUNCTION <name> [ ( [ [<arg_mode>] [<arg_name>] <arg_type> [, ...] ] ) ]
   RENAME TO <new_name>

ALTER FUNCTION <name> [ ( [ [<arg_mode>] [<arg_name>] <arg_type> [, ...] ] ) ]
   OWNER TO { <new_owner> | CURRENT_USER | SESSION_USER }

ALTER FUNCTION <name> [ ( [ [<arg_mode>] [<arg_name>] <arg_type> [, ...] ] ) ]
   SET SCHEMA <new_schema>

ALTER FUNCTION <name> [ ( [ [<arg_mode>] [<arg_name>] <arg_type> [, ...] ] ) ]
   DEPENDS ON EXTENSION <extension_name>

其中 <action> 可以为：

    { CALLED ON NULL INPUT | RETURNS NULL ON NULL INPUT | STRICT }
    { IMMUTABLE | STABLE | VOLATILE }
    { [EXTERNAL] SECURITY INVOKER | [EXTERNAL] SECURITY DEFINER }
    PARALLEL { UNSAFE | RESTRICTED | SAFE }
    COST <execution_cost>
    ROWS <result_rows>
    SET <config_param> { TO | = } { <value> | DEFAULT }
    SET <config_param> FROM CURRENT
    RESET <config_param>
    RESET ALL
```
---
描述
--------

只有函数所有者才能使用 `ALTER FUNCTION` 来修改函数定义。

如需修改函数的 Schema，你必须拥有新 Schema 的 `CREATE` 权限。 

如需更改函数的所有者，你必须是新所有角色的直接或间接成员，并对函数的 Schema 有 `CREATE` 权限。

---
参数
----------

- _`<name>`_

   函数的名称，支持使用 Schema 名称进行限定。如果该函数不带参数，其名称在其 Schema 中必须唯一。

- _`<arg_mode>`_

   参数的模式。支持的值包括 `IN`、`OUT`、`INOUT` 和 `VARIADIC`。如不指定，则使用默认值 `IN`。 `OUT` 参数参数无需列出，因为 `ALTER FUNCTION` 只使用输入参数来进行函数识别。

- _`<arg_name>`_

   参数的名称。 

   注意，`ALTER FUNCTION` 是通过参数的数据类型来识别函数的，而非参数名称。

- _`<arg_type>`_

   参数的数据类型。

- _`<action>`_

   对函数执行的操作，支持的操作包括：
   - `CALLED ON NULL INPUT | RETURNS NULL ON NULL INPUT | STRICT`：确定函数是否可以在其有 null 参数时被调用。`CALLED ON NULL INPUT` 表示可以调用，`RETURNS NULL ON NULL INPUT` 或 `STRICT` 表示不可以调用。
   
      更多信息，请参考 [CREATE FUNCTION](/maxir/Reference_Manual/sql-commands/create-function.md)。

   - `IMMUTABLE`、`STABLE` 或 `VOLATILE`：指定函数的易变性。

      更多信息，请参考 [CREATE FUNCTION](/maxir/Reference_Manual/sql-commands/create-function.md)。

   - `[EXTERNAL] SECURITY INVOKER | [EXTERNAL] SECURITY DEFINER`：指定函数是否为安全定义者 (Security Definer)。关键词 `EXTERNAL` 可以忽略。 
   
      更多信息，请查看 [CREATE FUNCTION](/maxir/Reference_Manual/sql-commands/create-function.md)。

   - `PARALLEL`：指定函数是否被认为是安全并行执行函数。
   
      更多信息，请查看 [CREATE FUNCTION](/maxir/Reference_Manual/sql-commands/create-function.md)。

   - `COST`：指定函数的预估执行成本。 
   
      更多信息，请查看 [CREATE FUNCTION](/maxir/Reference_Manual/sql-commands/create-function.md)。

   - `SET` 或 `RESET`：指定当函数被调用时配置参数 _`<config_param>`_ 的 _`<value>`_。如果 `SET` 选项的值为 `DEFAULT` 或使用 `RESET` 选项，则删除函数范围的设置并使用环境中的值替代。如果你指定 `SET ... FROM CURRENT`，则当你运行 `ALTER FUNCTION` 时参数的当前值将被用作函数的值。如果你想清除所有特定于函数的设置，使用 `RESET ALL`。 

- **`RESTRICT`**

   忽略以符合 SQL 标准。

- _`<new_name>`_

   函数的新名称。

- _`<new_owner>`_

   函数的新所有者。

- _`<new_schema>`_

   函数的新 Schema。 

- _`<extension_name>`_

   函数依赖的扩展的名称。


---

示例
--------

将 `text` 类型的函数 `countall` 重命名为 `count_all`：

```sql
ALTER FUNCTION countall(text) RENAME TO count_all;
```

更改 `text` 类型的函数 `count_all` 的所有者为 `mary@example.com`：

```sql
ALTER FUNCTION count_all(text) OWNER TO "mary@example.com";
```

将 `text` 类型的函数 `count_all` 的 Schema 改为 `public`：

```sql
ALTER FUNCTION count_all(text) SET SCHEMA public;
```

标记 `text` 类型的函数 `count_all` 依赖于扩展 `gp_exttable_fdw`：

```sql
ALTER FUNCTION count_all(text) DEPENDS ON EXTENSION gp_exttable_fdw;
```

将 `text` 类型的函数 `count_all` 的搜索路径更改为 `DEFAULT`：

```sql
ALTER FUNCTION count_all(text) SET search_path to DEFAULT;
```

---


SQL 标准兼容性
-------------

MAXIR 中的 `ALTER FUNCTION` 与标准 SQL `ALTER FUNCTION` 部分兼容。标准 SQL `ALTER FUNCTION` 允许你修改函数的更多属性，而 MAXIR 中的 `ALTER FUNCTION` 允许你更改函数的名称、所有者、Schema 和易变性，更改函数的配置参数的默认值，使函数成为安全定义者。此外，标准 SQL 中 `RESTRICT` 关键字是必需的，但在 MAXIR 中可以省略。
