CREATE VIEW
=====

创建视图。

---

语法
--------

```sql
CREATE [OR REPLACE] [RECURSIVE] VIEW <name> [ ( <column_name> [, ...] ) ]
    [ WITH ( <view_option_name> [= <view_option_value> ] [, ... ] ) ]
    AS <query>
    [ WITH [ CASCADED | LOCAL ] CHECK OPTION ]
```

---
描述
----------

`CREATE VIEW` 定义了一个查询的视图。视图并未实际物化。相反，每次在查询中引用视图时，都会运行该查询。

`CREATE OR REPLACE VIEW` 类似，但如果已存在同名的视图，它将被替换。新查询必须生成与现有视图查询生成的相同列（即，以相同顺序具有相同数据类型的相同列名），但它可能会在列表末尾添加额外的列。生成输出列的计算可能完全不同。

如果给出了 Schema 名称，则在指定的 Schema 中创建视图。否则，它将在当前 Schema 中创建。临时视图存在于一个特殊的 Schema 中，因此在创建临时视图时不能给出 Schema 名称。视图的名称必须与同一 Schema 中的任何其他视图、表、序列或索引的名称不同。


---

参数
----------


- **`RECURSIVE`**

    创建一个递归视图。语法：

    ```sql
    CREATE RECURSIVE VIEW [ <schema> . ] <view_name> (<column_names>) AS SELECT <...>;
    ```

    等同于

    ```sql
    CREATE VIEW [ <schema> . ] <view_name> AS WITH RECURSIVE <view_name> (<column_names>) AS (SELECT <...>) SELECT <column_names> FROM <view_name>;
    ```

    必须为递归视图指定视图列名称列表。

- *`<name>`*

    要创建的视图的名称。您可以在名称前加上 Schema 名称以在指定的 Schema 中创建视图。否则，将在当前 Schema 中创建视图。

- *`<column_name>`*

    用于视图列的可选名称列表。如果未给出，则从查询中推断出列名称。


- **`WITH ( <view_option_name> [= <view_option_value>] [, ... ] )`**

    此子句为视图指定可选参数；支持以下参数：

    - *`<check_option>`* (string)

        此参数可以是 `local` 或 `cascaded`，等同于指定 `WITH [ CASCADED | LOCAL ] CHECK OPTION`（见下文）。此选项可以使用 [ALTER VIEW](/maxir/Reference_Manual/sql-commands/alter-view.md) 在现有视图上更改。

    - *`<security_barrier>`* (boolean)

        如果视图打算提供行级安全性，请指定此参数。

- *`<query>`*

    将提供视图的列和行的 [SELECT](/maxir/Reference_Manual/sql-commands/select.md) 或 [VALUES](/maxir/Reference_Manual/sql-commands/values.md) 命令。


---


示例
--------

创建由 2022 年所有产品组成的视图：

```sql
CREATE VIEW sales_2022 AS
SELECT * 
FROM sales 
WHERE year = 2022;
```

>note
>在命令执行后添加到表中的列将不会包含在视图中。


创建一个获取排名前十的婴儿名字的视图：

```sql
CREATE VIEW topten AS 
SELECT name, rank, gender, year 
FROM names, rank 
WHERE rank < '11' AND names.id=rank.id;
```

创建由 1 到 100 的数字组成的递归视图：

```sql
CREATE RECURSIVE VIEW public.nums_1_100 (n) AS
    VALUES (1)
UNION ALL
    SELECT n+1 FROM nums_1_100 WHERE n < 100;
```

>note
>虽然在此 `CREATE VIEW` 命令中，递归视图的名称是 Schema 限定的，但其内部自引用不是 Schema 限定的。这是因为隐式创建的公共表表达式（CTE）的名称不能是 Schema 限定的。


---

SQL 标准兼容性
-------------

SQL 标准为 `CREATE VIEW` 语句指定了一些额外的功能，这些功能在 MAXIR 中不存在。标准中完整 SQL 命令的可选子句包括：

- `CHECK OPTION`：此选项与可更新的视图有关。对视图的所有 `INSERT` 和 `UPDATE` 命令将被检查以确保数据满足视图定义条件（即，新数据将通过视图可见）。如果它们不满足，更新将被拒绝。

- `LOCAL`：在此视图上检查完整性。

- `CASCADED`：在此视图和任何依赖视图上检查完整性。如果没有指定 `CASCADED` 或 `LOCAL`，则假定为 `CASCADED`。

`CREATE OR REPLACE VIEW` 是 MAXIR 语言的扩展。临时视图的概念也是如此。
