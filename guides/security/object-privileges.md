# 对象权限

数据库对象在创建时会被分配一个所有者。默认情况下，数据库对象的所有者即执行创建语句的数仓用户。

## 简介

当前，MAXIR 提供以下类型的数据库对象：表、视图、函数（包括存储过程）和 Schema。对于大多数类型的对象，初始状态是只有所有者对对象具有全部权限。要允许其他数仓用户使用它，必须授予权限。MAXIR 支持以下每种对象类型的权限。

| 对象类型 | 权限 |
| :- | :- |
| 表、外部表、视图 | `SELECT`、`INSERT`、`UPDATE`、`DELETE`、`TRUNCATE`、`ALL` |
| 列 | `SELECT`、`INSERT`、`UPDATE`、`ALL` |
| 数据库 | `CREATE`、`CONNECT`、`TEMPORARY`、`TEMP`、`ALL` |
| 函数 | `EXECUTE`、`ALL` |
| Schema | `CREATE`、`USAGE`、`ALL` |

<br/>

>请注意
>
> - 每一个数据库对象的权限必须单独授予。例如，将数据库的 `ALL` 授予数仓用户 A 并不意味着数仓用户 A 可以访问数据库中包含的其他数据库对象。数仓用户 A 只拥有该数据库本身的数据库级所有权限（即 `CREATE`、`CONNECT`、`TEMPORARY`、`TEMP`）。
>
>- 数据库账号只能授予其拥有的权限给其他用户。假设数仓用户 `user_A` 是 Schema `schema_a` 的所有者，而 `schema_a` 有两个表 `table_1` 和 `table_2`。`table_1` 是数仓用户 `user_A` 的，而 `table_2` 是数仓用户 `user_B` 的。数仓用户 `user_A` 只能将 `table_1` 的所有者转让给其他数仓用户，但不能改变 `table_2` 的所有权，尽管数仓用户 `user_A` 可以访问 `table_2`。
>
>- 为提升数据安全，权限授予不能跨级。假设数仓用户 `user_B` 是 Schema `schema_b` 的所有者。`schema_b` 中存在多个表。当 `user_B` 将 `schema_b` 的 `ALL` 权限授予 `user_C` 后， `user_C` 拥有的只是 Schema 级的所有权限（`CREATE`、`USAGE`），而非 Schema 中表的权限。




## 使用控制台管理对象权限

MAXIR 支持数据库对象所有者通过数仓服务单元控制台管理对象权限，包括权限授予、修改、撤回等。支持授权的数据库对象包括：

- 数据库

- Schema

- 表（包含外部表）

- 视图

> MAXIR 控制台不支持直接将函数（包含存储过程）权限授予其他数仓用户。如需将函数权限授予其他数仓用户，请参考 [使用 SQL 命令管理对象权限](#使用sql命令管理对象权限)。




### 将对象权限授予数仓用户
1. 登录数仓服务单元，选择"数据库"，选中目标数据库对象（可以是数据库、Schema、表、视图）。
2. 在右侧数据库对象详情页面，点击"+权限"，如数据库对象为表，点击"+ 权限"会出现下拉列表，支持选择授权层级为表级或者列级。
3. 在弹出的对话框中，按提示完成要授予权限的数仓用户以及要授予的对象权限，点击"确定"。

授权完成后，在目标对象的"权限"页签会展示最新的权限列表，包含每一个拥有权限的数仓用户及其被授予的权限。


### 修改数仓用户被授予的对象权限
1. 登录数仓服务单元，选择"数据库"，选中目标数据库对象（可以是数据库、Schema、表、视图）。
2. 在默认展示的"权限"页签，找到目标数仓用户名称，点击名称右侧修改按钮，选中或者去勾选目标对象权限，并点击右上方确认按钮。
>如需撤回该数仓用户在该数据库对象上的所有权限，点击右侧的删除按钮，然后在弹出的确认对话框中，点击"撤销权限"完成操作。
3. 修改完成后，目标数仓用户对应的权限列表会更新为最新的配置。



## 使用 SQL 命令管理对象权限

要授予数仓用户对数据库对象的权限，使用 `GRANT` 命令。例如，授予数仓用户 `mary@example.com` 对表 `table1` 的 `UPDATE` 权限：

```sql
GRANT UPDATE ON table1 TO "mary@example.com";
```

再举一个例子，授予数仓用户 `mary@example.com` 对表 `table2` 中的列 `col1` 的 `SELECT` 和 `UPDATE` 权限：

```sql
GRANT SELECT(col1), UPDATE(col1) ON table2 TO "mary@example.com";
```

要撤销权限，使用 `REVOKE` 命令。例如，从数仓用户 `mary@example.com` 撤销对表 `table1` 的 `DELETE` 权限：

```sql
REVOKE DELETE on table1 FROM "mary@example.com";
```

要撤销已授予已删除的数仓用户的权限，使用 `DROP OWNED`，例如：

```sql
DROP OWNED BY "tom@example.com";
```

