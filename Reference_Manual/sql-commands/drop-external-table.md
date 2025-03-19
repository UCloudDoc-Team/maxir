DROP EXTERNAL TABLE
=====

删除外部表。


---

语法
--------

```sql
DROP EXTERNAL TABLE [IF EXISTS] <name> [CASCADE | RESTRICT]
```

---

描述
--------

`DROP EXTERNAL TABLE` 不会删除外部数据源和文件。运行此命令，你必须是外部表的所有者。


---

参数
----------

- **`IF EXISTS`**

    如果在命令中包含 `IF EXISTS`，则在指定的外部表不存在时不会报错，而是产生一个通知。

- *`<name>`*

    外部表的名称。你可以在名称前加上 Schema 名称，从指定的 Schema 中删除外部表。

- **`CASCADE`** 或 **`RESTRICT`** 

    如果存在依赖对象，是否要删除表及其依赖对象。支持的选项包括：
    
    - `CASCADE`：指定连同外部表一起删除依赖对象。
    - `RESTRICT`：指定拒绝删除操作。


---

示例
-------------

删除外部表 `products`：

```sql
DROP EXTERNAL TABLE IF EXISTS products;
```

---

SQL 标准兼容性
-------------

标准 SQL 不支持 `DROP EXTERNAL TABLE`。
