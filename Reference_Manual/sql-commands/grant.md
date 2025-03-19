# GRANT
=====

授予数据库对象的权限或授予角色成员资格。


---

语法
--------

```sql
GRANT { {SELECT | INSERT | UPDATE | DELETE | 
TRUNCATE } [, ...] | ALL [PRIVILEGES] }
    ON { [TABLE] [[[ONLY] <table_name> [, ...]] [, ...]]
         | ALL TABLES IN SCHEMA <schema_name> [, ...] }
    TO <role_spec> [, ...] [ WITH GRANT OPTION ]

GRANT { { SELECT | INSERT | UPDATE } ( <col_name> [, ...] )
    [, ...] | ALL [ PRIVILEGES ] ( <col_name> [, ...] ) }
    ON [TABLE] [[[ONLY] <table_name> [, ...]] [, ...]]
    TO <role_spec> [, ...] [ WITH GRANT OPTION ]

GRANT { {CREATE | CONNECT | TEMPORARY | TEMP} [, ...] | ALL 
[PRIVILEGES] }
    ON DATABASE <database_name> [, ...]
    TO <role_spec> [, ...] [ WITH GRANT OPTION ]


GRANT { EXECUTE | ALL [PRIVILEGES] }
    ON { { FUNCTION | PROCEDURE | ROUTINE } <routine_name> [ ( [ [ <arg_mode> ] [ <arg_name> ] <arg_type> [, ...] ] ) ] [, ...]
        | ALL { FUNCTIONS | PROCEDURES | ROUTINES }  IN SCHEMA <schema_name> [, ...] }
    TO <role_spec> [, ...] [ WITH GRANT OPTION ]

GRANT { { CREATE | USAGE } [, ...] | ALL [PRIVILEGES] }
    ON SCHEMA <schema_name> [, ...]
    TO <role_spec> [, ...] [ WITH GRANT OPTION ]


GRANT <role_name> [, ...] TO <role_spec> [, ...]
    [ WITH ADMIN OPTION ]
    [ GRANTED BY <role_spec> ]


其中 <role_spec> 可以是：

    [ GROUP ] <role_name>
  | PUBLIC
  | CURRENT_USER
  | SESSION_USER
```

---

描述
--------

**授予数据库对象的权限**

你可以运行 `GRANT` 来授予一个或多个角色在数据库对象上的特定权限。如果你想要授予所有角色（包括后来创建的角色）特定权限，将 *`<role_spec>`* 设置为关键字 `PUBLIC`。`PUBLIC` 可以被视为包括所有角色的组级角色。所有角色自动具有授予 `PUBLIC` 的权限。

如果你在 `GRANT` 语句中包括 `WITH GRANT OPTION`，语句中指定的角色可以将权限授予除 `PUBLIC` 之外的其他角色。

:::note

- 数据库对象的所有者默认具有数据库对象的全部权限。然而，他们可以出于安全目的自行撤销一些权限。

- 即使是数据库对象的所有者，也不能授予或撤销删除或更改数据库对象的权限。
:::

你可以运行 `GRANT ON ... ROUTINE` 来授予普通、聚合或过程的权限，使用 `GRANT ... ON FUNCTION` 来授予普通、聚合或窗口函数的权限，或者 `GRANT ... ON PROCEDURE` 给过程。

此外，MAXIR 允许你运行 `GRANT ON ... SCHEMA` 来授予指定一个或多个 Schema 中的所有表、序列、函数或过程的权限。如果你想要授予所有表的权限，将 *`<role_spec>`* 设置为 `ALL TABLES`。指定的 Schema 中的视图和外键表也会受到影响。如果你想要授予所有函数的权限，将 *`<role_spec>`* 设置为 `ALL FUNCTIONS`。`ALL FUNCTIONS` 不包括过程；如果你想要包含过程，将 *`<role_spec>`* 设置为 `ALL ROUTINES`。如果只授予所有过程的权限，将 *`<role_spec>`* 设置为 `PROCEDURES`。

**授予角色权限**

你可以运行 `GRANT` 来授予一个或多个角色的角色成员资格。角色成员资格将角色授予的权限传递给所有的成员。这是非常重要的。


如果您在语句中包含 `GRANTED BY`，指定的角色将被视为执行 GRANT 操作的角色。

:::info
- *`<role_spec>`* 不能设置为 `PUBLIC`。
- 我们建议你在指定 *`role_spec`* 时不要使用噪音词 `GROUP`，因为我们的系统中没有使用用户组的概念。
:::




---

参数
----------

- **`SELECT`**

    具有从给定数据库对象的指定列中 `SELECT` 数据的权限。被授予此权限的角色被允许使用 `COPY ... TO`。执行 `UPDATE` 或 `DELETE` 操作时引用列值需要此权限。

- **`INSERT`**

    写入指定表的行的权限。如果你在 `INSERT` 语句中指定列，行中的值只写入指定的列，其他列用其默认值填充。被授予此权限的角色被允许使用 `COPY ... FROM`。

- **`UPDATE`**

    更新给定表中指定列的值的权限。要运行 `SELECT ... FOR UPDATE` 和 `SELECT ... FOR SHARE`，你还需要此权限和至少一个列的 `SELECT` 权限。此权限还允许你在序列上使用 `nextval()` 和 `setval()` 函数。

- **`DELETE`**

    从指定表中删除行的权限。


- **`TRUNCATE`**

    从给定表中截断所有行的权限。

- **`CREATE`**

    在给定的数据库中创建 Schema 的权限，或者在给定的 Schema 中创建对象的权限。

- **`CONNECT`**

    连接到给定数据库的权限。

- **`TEMPORARY | TEMP`**

    在使用给定数据库时创建临时表的权限。

- **`EXECUTE`**

    使用给定函数和所有使用函数的运算符的权限。这是唯一适用于函数的权限。

- **`USAGE`**

    访问给定 Schema 中的对象或在给定序列中使用 `curral()` 和 `nextval()` 函数的权限。对于一个 Schema，此权限允许授权人检查 Schema 中的对象。

- **`ALL PRIVILEGES`**

    所有权限。你可以使用 `ALL` 代替。

- **`PUBLIC`**

    包含所有角色的组级角色。分配给 `PUBLIC` 的权限会自动分配给所有角色，包括后来创建的角色。

- **`WITH GRANT OPTION`**

    允许指定权限的接收者将权限授予其他角色的选项。

- **`WITH ADMIN OPTION`**

    允许角色成员将角色成员资格授予其他角色的选项。



---

使用注意事项
----------

如果用户在表上拥有 `SELECT`、`UPDATE` 或其他特权，并尝试从特定列撤销特权，可能会影响到表级别的特权。

如果用户尝试向非用户所有的对象 `GRANT` 特权，语句将直接执行失败。只要有一项特权可用，语句就会继续执行。如果没有授予选项，`GRANT ALL PRIVILIGES` 将发出警告，而其他形式只要有一项授予选项未持有，就会发出警告。

如果角色是拥有对象的角色成员，或者拥有 `WITH GRANT OPTION` 特权的角色成员，无论角色是否是对象所有者，都可以向对象 `GRANT` 特权并从对象中 `REVOKE` 特权。在这种情况下，特权将被记录为由实际拥有对象或 `WITH GRANT OPTION` 特权的角色授予。

如果执行 `GRANT` 的角色从多个角色成员资格路径间接持有所需的特权，将不会指定记录为受让人的包含角色。在这种情况下，我们建议你使用 `SET ROLE` 来指定受让人。


---

示例
----------

执行以下语句，授予所有角色对 `sales` 表的 `UPDATE` 和 `INSERT` 权限：

```sql
GRANT UPDATE ON sales TO PUBLIC;
```

执行以下语句，授予用户 `maxir@example.com` 角色 `mary@example.com` 的成员资格：

```sql
GRANT "mary@example.com" TO "maxir@example.com";
```


---


SQL 标准兼容性
-------------

根据 SQL 标准，`ALL PRIVILEGES` 中的 `PRIVILEGES` 关键字是必需的，但在 MAXIR 中是可选的。SQL 标准不支持在每个命令中对多个对象设置权限。

MAXIR 允许对象所有者撤销自己的普通权限：例如，表所有者可以通过撤销自己的 `INSERT`、`UPDATE` 和 `DELETE` 权限，使表对自己只读。根据 SQL 标准，这是不可能的。MAXIR 将所有者的权限视为由所有者授予所有者；因此他们也可以撤销这些权限。在 SQL 标准中，所有者的权限是由假定的系统实体授予的。所有者不是系统，因此不能撤销这些权限。

SQL 标准允许在所有形式的 `GRANT` 中使用 `GRANTED BY` 选项。MAXIR 仅在授予角色成员资格时支持它，即使在这种情况下，只有超级用户才能以非平凡的方式使用它。

SQL 标准为其他类型的对象提供了 `USAGE` 权限：字符集、排序规则、翻译。

在 SQL 标准中，序列只有 `USAGE` 权限，它控制 `NEXT VALUE FOR` 表达式的使用，这等于 MAXIR 中的 nextval() 函数。序列权限 `SELECT` 和 `UPDATE` 是 MAXIR 的扩展。将序列 `USAGE` 权限应用于 `currval()` 函数也是 MAXIR 的扩展（函数本身也是扩展）。

数据库、Schema 的权限是 MAXIR 的扩展。
