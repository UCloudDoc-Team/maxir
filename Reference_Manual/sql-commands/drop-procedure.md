DROP PROCEDURE
=====

删除存储过程。


---

语法
--------

```sql
DROP PROCEDURE [IF EXISTS] <name> ( [ [arg_mode] [arg_name] arg_type [, ...] ] )
    [CASCADE | RESTRICT]
```


---

描述
----------

MAXIR 允许你创建具有不同参数的同名存储过程。因此，运行此命令时，你必须指定参数类型。

只有存储过程的所有者才能删除该存储过程。


---

参数
----------

- **`IF EXISTS`**

    如果在命令中包含 `IF EXISTS`，当指定的存储过程不存在时，将不会报错，而是产生一条通知。

- *`<name>`*

    存储过程的名称。你可以在名称前加上 Schema 名称，从指定的 Schema 中删除存储过程。

- *`<arg_mode>`*

    参数的模式，可以为 `IN` 或 `VARIADIC`。如果省略，将使用默认值 `IN`。

- *`<arg_name>`*

    参数的名称。
    
    `DROP PROCEDURE` 不使用参数名称来标识存储过程。

- *`<arg_type>`*

    参数的数据类型。

- **`CASCADE`** 或 **`RESTRICT`**

    如果存在依赖对象，是否删除存储过程及其依赖对象。支持的选项包括：

    - `CASCADE`：指定连同存储过程一起删除依赖对象。
    - `RESTRICT`：指定拒绝删除操作，这是默认值。


---


示例
-------------

删除存储过程 `add_films()`：

```sql
DROP PROCEDURE add_films();
```

---

SQL 标准兼容性
-------------

与 SQL 标准中的 `DROP PROCEDURE` 相比，MAXIR 中的 `DROP PROCEDURE` 提供了以下扩展：

- 允许你在一个命令中删除多个策略。
- 允许你使用 `IF EXISTS` 选项。
- 允许你指定参数模式和名称。
