EXECUTE
=====

执行预处理语句。


---

语法
--------

```sql
EXECUTE <name> [ (<param> [, ...] ) ]
```

---

描述
--------

如果用于创建目标预处理语句的 [PREPARE](/maxir/Reference_Manual/sql-commands/prepare.md) 语句指定了一些参数，你必须将参数传递给 `EXECUTE` 语句。否则，将产生报错。

与函数不同，预处理语句不能根据其参数的类型或数量进行重载。因此，预处理语句的名称在数据库会话中必须是唯一的。


---

参数
----------

- *`<name>`*

    预处理语句的名称。

- *`<param>`*

    参数的值。这个值必须是一个表达式，产生的值与在创建预处理语句时为此参数声明的数据类型一致。


---


输出
----------
返回的命令标签不是来自 `EXECUTE`，而是来自预处理语句。


---

示例
----------

为 `INSERT` 语句创建一个预处理语句，然后运行它：

```sql
PREPARE pendings (integer, bool, numeric) AS 
INSERT INTO my_favorites VALUES ($1, $2, $3);
EXECUTE pendings(1, 'true', 'volleyball');
```


---

SQL 标准兼容性
-------------

在 SQL 标准中，`EXECUTE` 只能在嵌入式 SQL 中使用。MAXIR 中的 `EXECUTE` 使用了不同的语法。
