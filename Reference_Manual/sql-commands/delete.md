DELETE
=====

从表中删除行。

---

语法
--------

```sql
DELETE FROM [ONLY] <table> [[AS] <alias>]
      [WHERE <condition> ]
```

---
描述
----------

`DELETE` 从指定的表中删除满足 `WHERE` 子句的行。如果 `WHERE` 子句不存在，效果相当于删除表中的所有行。结果是一个有效的，但是空的表。

默认情况下，`DELETE` 将删除指定表及其所有子表中的行。如果你希望只从特定提到的表中删除，你必须使用 `ONLY` 子句。

你可以使用子选择来根据数据库中其他表中包含的信息删除表中的行。

你必须拥有表的 `DELETE` 权限才能从中删除。

:::note
默认情况下，MAXIR 对于堆表上的 `DELETE` 操作获取表的 `EXCLUSIVE` 锁。
:::

---

参数
----------

- **`ONLY`**

    如果指定，只从命名的表中删除行。如果未指定，从命名的表继承的任何表也都会被处理。

- *`<table>`*

    表的当前名称。支持使用 Schema 限定。

- *`<alias>`*

    目标表的替代名称。当提供别名时，它会完全隐藏表的实际名称。例如，给定 `DELETE FROM foo AS f`，`DELETE` 语句的其余部分必须将此表称为 `f` 而不是 `foo`。

- *`<condition>`*

    返回 `boolean` 类型值的表达式，用于确定要删除的行。


---

输出
----------

在成功完成后，`DELETE` 命令返回一个形如

```sql
DELETE <count>
```

的命令标签。

*`<count>`* 是被删除的行数。如果计数为 0，说明查询未删除任何行（这不被视为错误）。


---

示例
--------

删除除类别为 `Musical` 以外的所有电影：

```sql
DELETE FROM films WHERE kind <> 'Musical';
```

清除 `films` 表：

```sql
DELETE FROM films;
```

---

SQL 标准兼容性
-------------

此命令符合 SQL 标准。
