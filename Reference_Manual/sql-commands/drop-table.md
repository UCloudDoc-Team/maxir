DROP TABLE
=====

删除一个或多个表。

---

语法
--------

```sql
DROP TABLE [IF EXISTS] <name> [, ...] [CASCADE | RESTRICT]
```


---

描述
--------

当你使用 `DROP TABLE` 删除表时，你必须是表的所有者或者表所属的 Schema 的所有者。

`DROP TABLE` 不仅删除表中记录的数据，还删除所有相关元素，包括规则、触发器和约束。因此，要只删除表中的数据而保留其定义，请使用 `DELETE` 或 `TRUNCATE`。


---

参数
----------

- **`IF EXISTS`**

    如果在命令中包含 `IF EXISTS`，当指定的表不存在时，将不会报错，而是产生一条通知。

- *`<name>`*

    表的名称。你可以在名称前加上 Schema 名称，从指定的 Schema 中删除表。

- **`CASCADE`** 或 **`RESTRICT`**

    如果存在依赖对象，是否删除表及其依赖对象。支持的选项包括：

    - `CASCADE`：指定连同表一起删除依赖对象。
    - `RESTRICT`：指定拒绝删除操作，这是默认值。

---

示例
--------

删除表 `test1`：

```sql
DROP TABLE test1;
```


---

SQL 标准兼容性
-------------

MAXIR 中的 `DROP TABLE` 与 SQL 标准中的 `DROP TABLE` 完全兼容，并且 MAXIR 中的 `DROP TABLE` 提供了以下扩展：

- 允许你在一个命令中删除多个表。
- 允许你使用 `IF EXISTS` 选项。
