ROLLBACK
=====

终止当前事务。


---

语法
--------

```sql
ROLLBACK [WORK | TRANSACTION] [AND CHAIN | AND NO CHAIN]
```

---

描述
----------

当你运行 `ROLLBACK` 来停止当前事务时，事务所做的所有更新都会被丢弃。

如果 `ROLLBACK` 不在事务块中，它将无效并产生警告。如果 `ROLLBACK AND CHAIN` 不在事务块中，将报错。


---

参数
----------

- **`[WORK | TRANSACTION]`**

    一个可选的关键字，没有效果。

- **`[AND CHAIN | AND NO CHAIN]`**

    是否在当前事务结束后立即开始具有相同特征的事务。支持的选项包括：
    - `AND CHAIN`：指定是。
    - `AND NO CHAIN`：指定否，这是默认值。


---

示例
----------

终止当前事务，不启动新的事务：

```sql
ROLLBACK;
```

---

SQL 标准兼容性
-------------

MAXIR 中的 `ROLLBACK` 与 SQL 标准中的 `ROLLBACK` 完全兼容。然而，MAXIR 还支持 `ROLLBACK TRANSACTION` 的形式。
