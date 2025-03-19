BEGIN
=====

启动事务块。


---

语法
--------

```sql
BEGIN [WORK | TRANSACTION] [<transaction_mode>]

其中 <transaction_mode> 可以是：

   READ WRITE | READ ONLY
```

---

描述
----------

在 `BEGIN` 后指定的所有语句将在一个事务中运行，直到运行至明确制定的 `COMMIT` 或 `ROLLBACK` 操作。

---
参数
----------

- **`[WORK | TRANSACTION]`**

   可选的关键字，不会对 `BEGIN` 操作产生任何影响。

- _`<transaction_mode>`_

   可以为 `READ WRITE` 或 `READ ONLY`。如果省略，则使用默认值 `READ ONLY`。


---

使用注意事项
----------

如需终止一个事务块，执行 [COMMIT](commit.md) 或 [ROLLBACK](rollback.md)。

在事务块内进行 `BEGIN` 操作，系统会产生一条警告，但是不影响事务的状态。 


---

示例
--------

启动事务块：

```sql
BEGIN;
```

---

SQL 标准兼容性
-------------

`BEGIN` 是一个等同于标准 SQL `START TRANSACTION` 的 MAXIR 扩展。

在 SQL 标准中，`BEGIN` 关键字的用法与 MAXIR 中的 `BEGIN` 不同。因此，当你将数据库应用迁移到 MAXIR 时，需要确保事务语义的正确性。
