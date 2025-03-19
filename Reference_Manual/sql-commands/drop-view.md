DROP VIEW
=====

删除视图。


---

语法
--------

```sql
DROP VIEW [IF EXISTS] <name> [, ...] [CASCADE | RESTRICT]
```


---

描述
--------

使用 `DROP VIEW` 删除视图时，你必须为该视图的所有者。


---

参数
----------

- **`IF EXISTS`**

    如果在命令中包含 `IF EXISTS`，当指定的视图不存在时，将不会报错，而是产生一条通知。

- *`<name>`*

    视图的名称。你可以在名称前加上 Schema 名称，从指定的 Schema 中删除视图。

- **`CASCADE`** 或 **`RESTRICT`**

    如果存在依赖对象，是否删除视图及其依赖对象。支持的选项包括：

    - `CASCADE`：指定连同视图一起删除依赖对象。
    - `RESTRICT`：指定拒绝删除操作，这是默认值。


---

示例
--------

删除视图 `myview`：

```sql
DROP VIEW myview;
```

---

SQL 标准兼容性
-------------

MAXIR 中的 `DROP VIEW` 与 SQL 标准中的 `DROP VIEW` 完全兼容，并且 MAXIR 中的 `DROP VIEW` 提供了以下扩展：

- 允许你在一个命令中删除多个视图。
- 允许你使用 `IF EXISTS` 选项。
