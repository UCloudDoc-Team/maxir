END
=====

提交当前事务。


---


语法
--------

```sql
END [WORK | TRANSACTION] [AND CHAIN | AND NO CHAIN ]
```

---

描述
--------

通过 `END` 提交当前事务后，事务所做的所有更改对其他用户可见，并且如果系统崩溃，这些更改将保持持久性。


此语句等效于 [COMMIT](/maxir/Reference_Manual/sql-commands/commit.md)。要终止事务而不是提交事务，使用 [ROLLBACK](/maxir/Reference_Manual/sql-commands/rollback.md)。

当你在事务外部运行 `END` 时，将会显示警告消息，但操作没有不良影响。


---


参数
----------

- **`[WORK | TRANSACTION]`**

    一个额外的关键字，没有效果。

- **`[AND CHAIN | AND NO CHAIN]`**

    是否在当前事务结束后立即开始具有相同特征的事务。支持的选项包括： 
    - `AND CHAIN`：指定为是。
    - `AND NO CHAIN`：指定为否，这是默认值。


---

示例
--------

提交当前事务：

```sql
END;
```

---

SQL 标准兼容性
-------------
`END` 是 MAXIR 的扩展。它等效于 SQL 标准中的 `COMMIT`。
