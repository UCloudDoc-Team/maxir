DROP DATABASE
=====

删除数据库。

---

语法
--------

```sql
DROP DATABASE [IF EXISTS] <name> [ WITH FORCE ]
```

---

描述
--------

`DROP DATABASE` 删除数据库的所有目录条目以及包含数据的目录。这个操作无法撤销，请谨慎使用。

---

参数
----------

- **`IF EXISTS`**

    如果在语句中包含 `IF EXISTS`，则在指定的数据库不存在时不会报告错误，而是发出通知。

- *`<name>`*

    数据库的名称。

- **`WITH FORCE`**

    指定强制删除数据库，无论是否有人连接到它。如果不指定此选项，如果有人连接到数据库，将无法删除数据库。

---

使用注意事项
----------

`DROP DATABASE` 不能在事务块内运行。

---

示例
----------

删除数据库 `testonly`：

```sql
DROP DATABASE testonly;
```

---

SQL 标准兼容性
-------------

标准 SQL 不支持 `DROP DATABASE`。
