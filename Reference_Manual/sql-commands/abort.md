ABORT
=====

终止当前事务。

---

语法
--------

```sql

    ABORT [WORK | TRANSACTION] [AND CHAIN | AND NO CHAIN]

```
    
---

描述
-----

`ABORT` 回滚当前事务并不会保存当前事务所做的任何更新，与 [ROLLBACK](rollback.md) 效果相同。

:::note
`ABORT` 只是出于历史原因才出现的。
:::

---
参数
----------

- **`[WORK | TRANSACTION]`**

    可选关键字，对 `ABORT` 操作不产生任何影响。

- **`[AND CHAIN | AND NO CHAIN]`**

    是否在当前事务被终止后立即启动具有相同特性的事务。支持的选项包括：
    - `AND CHAIN`：启动
    - `AND NO CHAIN`：不启动


---

示例
--------

回滚当前事务且不保存其所做的任何更改：

```sql
ABORT;
```
    
---
SQL 标准兼容性
-------------

`ABORT` 是 MAXIR 扩展，其效果等同于标准 SQL 命令 `ROLLBACK`。
