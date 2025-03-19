DEALLOCATE
=====

释放预处理语句。

---

语法
--------

```sql
DEALLOCATE [PREPARE] { <name> | ALL }
```

---

描述
----------

`DEALLOCATE` 用于释放预处理语句。如果你不运行 `DEALLOCATE` 来显式释放预处理语句，预处理语句将在会话结束时自动结束。

---

参数
----------

- **`PREPARE`**

    一个可选的关键字，没有效果。

- *`<name>`*

    要释放的预处理语句的名称。

- **`ALL`**

    如果你想释放所有预处理语句，指定 **ALL** 而不是 *`<name>`*。


---

示例
--------

释放预处理语句 `add_films`：

```sql
DEALLOCATE add_films;
```


---


SQL 标准兼容性
-------------

在标准 SQL 中，`DEALLOCATE` 只能在嵌入式 SQL 中使用。
