ALTER PROCEDURE
=====

修改存储过程定义。


---

Syntax
--------

```sql
ALTER PROCEDURE <name> [ ( [ [<arg_mode>] [<arg_name>] <arg_type> [, ...] ] ) ] 
   <action> [, ... ] [RESTRICT]

ALTER PROCEDURE <name> [ ( [ [<arg_mode>] [<arg_name>] <arg_type> [, ...] ] ) ]
   RENAME TO <new_name>

ALTER PROCEDURE <name> [ ( [ [<arg_mode>] [<arg_name>] <arg_type> [, ...] ] ) ]
   OWNER TO { <new_owner> | CURRENT_USER | SESSION_USER }

ALTER PROCEDURE <name> [ ( [ [<arg_mode>] [<arg_name>] <arg_type> [, ...] ] ) ]
   SET SCHEMA <new_schema>

ALTER PROCEDURE <name> [ ( [ [<arg_mode>] [<arg_name>] <arg_type> [, ...] ] ) ]
   DEPENDS ON EXTENSION <extension_name>

其中 <action> 可以为：

    { [EXTERNAL] SECURITY INVOKER | [EXTERNAL] SECURITY DEFINER }
    SET <config_param> { TO | = } { <value> | DEFAULT }
    SET <config_param> FROM CURRENT
    RESET <config_param>
    RESET ALL
```

---
描述
----------

只有存储过程所有者才可以使用 `ALTER PROCEDURE` 来修改存储过程的定义。

如需修改存储过程的 Schema，你必须拥有新 Schema 的 `CREATE` 权限。

要更改存储过程的所有者，你必须新的所有角色的直接或间接成员，并对存储过程的 Schema 有 `CREATE` 权限。


:::note
MAXIR 对使用 `STABLE` 和 `VOLATILE` 函数有限制，详情请见 [CREATE FUNCTION](create-function.md)。
:::


---

参数
----------

- _`<name>`_

   存储过程的名称，支持使用 Schema 名称进行限定。如果该存储过程不带参数，其名称在其 Schema 中必须唯一。

- _`<action>`_

   在存储过程上执行的操作。
   
   关于支持的操作的描述的详细信息，请参考 [ALTER FUNCTION](alter-function.md#参数) 中的“参数”部分。

- _`<arg_mode>`_

   参数的模式，可以是 `IN` 或 `VARIADIC`。如果未指定此参数，则使用默认值 `IN`。

- _`<arg_name>`_

   参数的新名称。
   
   请注意，`ALTER PROCEDURE` 不使用参数名称来识别存储过程。

- _`<arg_type>`_

   参数的数据类型。

- **`RESTRICT`**

  忽略此关键词，以符合 SQL 标准。

- _`<new_name>`_

   存储过程的新名称。 

- _`<new_owner>`_, **`CURRENT_USER`**, 或 **`SESSION_USER`**

   存储过程的新所有者。
   
   - *`<new_owner>`*: 明确指定新的所有角色。

      如果存储过程是 `SECURITY DEFINER`，那么它将作为新所有者运行。

   - `CURRENT_USER`: 将所有者修改为当前用户。
   - `SESSION_USER`: 将所有者修改为当前会话用户。

- _`<new_schema>`_ 

   存储过程的新 Schema。

- _`<extension_name>`_

   存储过程所依赖的扩展的名称。 

   如果你执行了 `ALTER PROCEDURE ... DEPENDS ON EXTENSION`。当制定的扩展被删除时，该存储过程也会被删除。

---
示例
--------

将带有 `text` 和 `integer` 类型的两个参数的存储过程 `record` 重命名为 `insert_record`：

```sql
ALTER PROCEDURE record(text, integer) RENAME TO insert_record;
```

将带有 `text` 和 `integer` 类型的两个参数的存储过程 `insert_record` 的所有者更改为 `mary@example.com`：

```sql
ALTER PROCEDURE insert_record(text, integer) OWNER TO "mary@example.com";
```

将带有 `text` 和 `integer` 类型的两个参数的存储过程 `insert_record` 的 Schema 更改为 `public`：

```sql
ALTER PROCEDURE insert_record(text, integer) SET SCHEMA public;
```

将 `text` 和 `integer` 类型的存储过程 `insert_record` 标记为依赖于扩展 `gp_exttable_fdw`：

```sql
ALTER PROCEDURE insert_record(text, integer) DEPENDS ON EXTENSION gp_exttable_fdw;
```

将带有 `text` 和 `integer` 类型的两个参数的存储过程 `insert_record` 的搜索路径更改为 `DEFAULT`：

```sql
ALTER FUNCTION insert_record(text, integer) SET search_path to DEFAULT;
```

---

SQL 标准兼容性
-------------

MAXIR 中的 `ALTER PROCEDURE` 与标准 SQL `ALTER PROCEDURE` 部分兼容。标准 SQL 的 `ALTER PROCEDURE` 允许你修改存储过程的更多属性，而 MAXIR 中的 `ALTER PROCEDURE` 允许你改变存储过程的名称、所有者、Schema 和易变性，更改存储过程的配置参数的默认值，并使存储过程成为安全定义者。
