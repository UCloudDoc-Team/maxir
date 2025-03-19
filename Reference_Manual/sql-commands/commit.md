COMMIT
=====

提交当前事务。

---


语法
--------

```sql
COMMIT [WORK | TRANSACTION] [ AND CHAIN | AND NO CHAIN ]
```


---

描述
----------

在当前事务提交后，事务所做的所有更改对用户都是可见的。

如果你想终止正在运行的事务，请执行 [ROLLBACK](rollback.md)。

如果 `COMMIT` 不在事务之内，系统将产生一个警告。如果 `COMMIT AND CHAIN` 不在事务之内，系统将产生一个错误。

---

参数
----------

- **`[WORK | TRANSACTION]`**

    可选关键词，不会对 `COMMIT` 操作产生任何影响。

- **`[AND CHAIN | AND NO CHAIN]`**

    指定是否在当前事务提交后立即以相同的特征启动一个事务的选项：
    - `AND CHAIN` 启动
    - `AND NO CHAIN` 不启动（默认值）

---

示例
--------

提交当前事务：

```sql
COMMIT;
```

---


SQL 标准兼容性
-------------

MAXIR 中的 `COMMIT` 与标准 SQL `COMMIT` 完全兼容。
