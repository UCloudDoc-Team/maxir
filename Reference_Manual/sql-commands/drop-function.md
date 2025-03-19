DROP FUNCTION
=====

删除函数。


---

语法
--------

```sql
DROP FUNCTION [IF EXISTS] <name> ( [ [arg_mode] [arg_name] arg_type [, ...] ] )
    [CASCADE | RESTRICT]
```

---

参数
----------

- **`IF EXISTS`**

    如果命令中包含 `IF EXISTS`，则当指定的函数不存在时，不会报告错误，系统仅会产生一条通知。

- *`<name>`*

    函数的名称。你可以在名称前加上 Schema 名称，从指定的 Schema 中移除函数。

- *`<arg_mode>`*

    参数模式，可以是 `IN`、`OUT`、`INOUT` 或 `VARIADIC`。如果省略，将使用默认值 `IN`。你不需要列出 `OUT` 参数，因为 `DROP FUNCTION` 只使用输入参数来标识函数。

- *`<arg_name>`*

    参数的名称。 

    参数名称是不必要的，因为 `DROP FUNCTION` 不使用参数名称来识别函数。

- *`<arg_type>`*

    参数的数据类型。由于可以存在名称相同的不同函数，因此需要此参数来识别函数。

- **`CASCADE`** 或 **`RESTRICT`**

    如果存在依赖对象，是否要连同依赖对象一起删除函数。支持的选项包括：

    - `CASCADE`：连同函数一起删除依赖对象。
    - `RESTRICT`：拒绝删除操作，这是默认行为。


---

示例
-------------

移除平方根函数：

```sql
DROP FUNCTION sqrt(integer);
```

---

SQL 标准兼容性
-------------

标准 SQL 不支持 `DROP FUNCTION`。
