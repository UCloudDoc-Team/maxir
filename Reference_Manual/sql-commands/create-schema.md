CREATE SCHEMA
=====

创建 Schema。

---


语法
--------

```sql
CREATE SCHEMA <schema_name> [AUTHORIZATION <role_spec>] 
   [<schema_element> [ ... ]]

CREATE SCHEMA AUTHORIZATION <role_spec> [<schema_element> [ ... ]]

CREATE SCHEMA IF NOT EXISTS <schema_name> [ AUTHORIZATION <role_spec> ]

CREATE SCHEMA IF NOT EXISTS AUTHORIZATION <role_spec>

其中 <role_spec> 可以是：
    <user_name>
  | CURRENT_USER
  | SESSION_USER
```

---
描述
----------

`CREATE SCHEMA` 在当前数据库中定义一个新的 Schema。因此，`CREATE SCHEMA` 中指定的 Schema 名称不能与当前数据库中的已有 Schema 重名。

本质上，Schema 是一个命名空间。它包含命名对象，如表格、函数和视图。不同的 Schema 中的对象可以有相同的名称。当您访问一个命名对象时，可以在对象名称前加上 Schema 名称，或者在搜索路径中包含所需的 Schema。

当您运行一个 `CREATE` 命令创建一个未指定 Schema 的命名对象时，该命名对象将在当前 Schema 中创建。查询当前 Schema，可以调用函数 `current_schema()`。

您可以在 `CREATE SCHEMA` 中包含子命令来创建新 Schema 中的对象。这些子命令被视为单独的命令，在 Schema 创建后执行。然而，如果 `CREATE SCHEMA` 还包括一个 `AUTHORIZATION` 子句，那么这些子命令创建的对象将由指定的角色拥有。

---

参数
----------

- *`<schema_name>`*

    Schema 的名称。如果省略，*`<user_name>`* 将被用作 Schema 名称。
    
    Schema 名称不能以 **`pg_`** 开头。名称以 **`pg_`** 开头的 Schema 为系统预留 Schema。


- *`<role_spec>`*

    Schema 的所有者。如果省略，执行命令的数仓用户将成为所有者。支持的值包括：

    - *`<user_name>`*：指定数仓用户的角色名称，该用户成为 Schema 的所有者。要创建由另一个角色拥有的 Schema，你必须是该角色的直接或间接成员。
    - **`CURRENT_USER`**：指定当前用户为 Schema 所有者。
    - **`SESSION_USER`**：指定当前会话用户为 Schema 所有者。

- *`<schema_element>`*

   用于在 Schema 中创建命名对象的 SQL 子命令。目前，只支持 `CREATE TABLE`、`CREATE VIEW` 和 `GRANT`。 


- **`IF NOT EXISTS`**

    如果命令中包含 `IF NOT EXISTS`，当存在同名的 Schema 时，系统将不会报错，而是产生一条通知。

---

使用注意事项
-------------

要创建 Schema，当前用户必须对当前数据库有 `CREATE` 权限。


---

示例
--------

创建一个名为 `my_schema` 的 Schema：

```sql
CREATE SCHEMA my_schema;
```

为角色 `mary@example.com` 创建一个 Schema；该 Schema 也将被命名为 `mary@example.com`：

```sql
CREATE SCHEMA AUTHORIZATION "mary@example.com";
```

创建一个名为 `test` 的 Schema，所有者为用户 `mary@example.com`，除非已经存在一个名为 `mary@example.com` 的 Schema。（现有 Schema 的所有者是 `mary@example.com` 并不重要。）

```sql
CREATE SCHEMA IF NOT EXISTS test AUTHORIZATION "foo@example.com";
```

创建一个 Schema 并在其中创建一个表和视图：

```sql
CREATE SCHEMA music
    CREATE TABLE myplaylist (singer text, style text, awards text[])
    CREATE VIEW awards_winners AS
        SELECT singer, style FROM myplaylist WHERE awards IS NOT NULL;
```

注意，单独的子命令不以分号结束。

以下是实现相同结果的等效方法：

```sql
CREATE SCHEMA music
    CREATE TABLE music.myplaylist (singer text, style text, awards text[])
    CREATE VIEW awards_winners AS
        SELECT singer, style FROM music.myplaylist WHERE awards IS NOT NULL;
```

---


SQL 标准兼容性
-------------

SQL 标准允许 `CREATE SCHEMA` 中包含 `DEFAULT CHARACTER SET` 子句，以及比 MAXIR 目前接受的更多子命令类型。

根据 SQL 标准，`CREATE SCHEMA` 中的子命令可以以任何顺序出现。目前的 MAXIR 实现并未处理所有子命令中的前向引用情况；有时可能需要重新排序子命令，以避免前向引用。

根据 SQL 标准，Schema 的所有者始终拥有其内的所有对象。MAXIR 允许 Schema 包含由非 Schema 所有者的用户拥有的对象。只有当 Schema 所有者将 Schema 的 `CREATE` 权限授予其他人时，才可能发生这种情况。

`IF NOT EXISTS` 选项是 MAXIR 的扩展。

SQL 标准在 `CREATE SCHEMA` 命令中包含了 `DEFAULT CHARACTER SET` 子句，并允许更多类型的子命令。

根据 SQL 标准，`CREATE SCHEMA` 中的子命令可以以任何顺序排列。然而，当前的 MAXIR 实现并不支持所有子命令中的前向引用情况。在某些情况下，你可能需要对这些子命令进行重新排序，以避免这种前向引用。

在 SQL 标准中，Schema 的所有者是其内所有对象的所有者。然而，MAXIR 允许 Schema 中包含由其他数仓用户所拥有的对象。这只能在 Schema 所有者将 Schema 的 `CREATE` 权限授予其他数仓用户时发生。

`IF NOT EXISTS` 选项是 MAXIR 的扩展。
