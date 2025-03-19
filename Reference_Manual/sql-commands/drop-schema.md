DROP SCHEMA
=====

删除 Schema。


---

语法
--------

```sql
DROP SCHEMA [IF EXISTS] <name> [, ...] [CASCADE | RESTRICT]
```


---

描述
--------

使用 `DROP SCHEMA` 来删除 Schema，你必须是该 Schema 的所有者。

:::info
你不需要拥有属于目标 Schema 的所有对象即可以使用 `DROP SCHEMA`。
:::

---

参数
----------

- **`IF EXISTS`**

    如果在命令中包含 `IF EXISTS`，当指定的 Schema 不存在时，将不会报错，而是产生一条通知。

- *`<name>`*

    Schema 的名称。

- *`<table_name>`*

    规则适用的表或视图的名称。你可以用 Schema 名称来指定此名称。

- **`CASCADE`** 或 **`RESTRICT`**

    如果存在依赖对象，是否删除 Schema 及其依赖对象。支持的选项包括：

    - `CASCADE`：指定连同 Schema 一起删除依赖对象。
    - `RESTRICT`：指定拒绝整个删除操作，这是默认值。

    如果在命令中指定 `CASCADE`，其他 Schema 中的依赖对象也可能被删除。


---

示例
--------

删除 Schema `basic_info`：

```sql
DROP SCHEMA basic_info;
```

---

SQL 标准兼容性
-------------

MAXIR 中的 `DROP SCHEMA` 与 SQL 标准中的 `DROP SCHEMA` 完全兼容，并且 MAXIR 中的 `DROP SCHEMA` 提供了以下扩展：

- 允许你在一个命令中删除多个 Schema。
- 允许你使用 `IF EXISTS` 选项。
