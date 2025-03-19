# TRUNCATE

删除一个或多个表中的所有行。


---

## 语法

```sql
TRUNCATE [TABLE] [ONLY] <name> [ * ] [, ...] 
    [CASCADE | RESTRICT]
    [WHERE <cluster_key_condition>]
```


---

## 描述

`TRUNCATE` 快速清除表中的所有行数据，效果等同于对每个表进行的无条件 [DELETE](delete.md)。但由于 `TRUNCATE` 实际上并不扫描表，所以速度更快。此外，`TRUNCATE` 执行完成后会立即回收磁盘空间，而不需要 [VACUUM](vacuum.md) 操作，这使得 `TRUNCATE` 在大表上应用的效果非常明显。

`TRUNCATE ... WHERE` 变体用于删除指定了聚簇键的表中符合特定条件的数据，条件通过 `WHERE` 子句指定，只能且必须基于目标表的第一个聚簇键。

执行 `TRUNCATE` 要求必须拥有表的 `TRUNCATE` 权限。


---

## 参数

- *`<name>`*

    表的名称，支持使用 Schema 名称进行限定。

- **`ONLY`**

    可选关键字，不会产生任何效果。

- **`CASCADE`** 或 **`RESTRICT`**

    可选关键字，不会产生任何效果。

- *`<cluster_key_condition>`*

    `WHERE` 子句中指定的聚簇键条件，只能且必须基于目标表的第一个聚簇键。`TRUNCATE` 会删除满足此条件的数据。

---

## 使用注意事项

`TRUNCATE` 获取每个操作表上的 `ACCESS EXCLUSIVE` 锁，阻止了该表上的所有其他并发操作。如果你需要并发访问表，请使用 `DELETE` 命令。

`TRUNCATE` 不会截断继承自指定表的任何表，只有表自身会被截断。

`TRUNCATE` 不是基于多版本并发控制（MVCC）的安全操作。当执行截断操作后，如果存在并发事务仍在使用截断操作之前的数据快照，这些事务将看到一个空表。这是因为 `TRUNCATE` 直接删除表中的数据，而不记录每行数据的变化，从而影响正在进行的并发事务的数据一致性。

`TRUNCATE` 命令在处理表中数据时具备事务级别的安全性：若关联事务未获得提交确认，则该命令对表执行的截断操作可以被安全地回滚。


---

## 示例

清空名为 `sales` 和 `topsellers` 的表：

```sql
TRUNCATE sales, topsellers;
```

清除 `orders` 表中 `id` 为 `20222001` 的记录：

```sql
CREATE TABLE orders (
    id INT,
    order_date DATE,
    customer_name TEXT,
    amount INTEGER
)
CLUSTER BY id;

INSERT INTO orders (id, order_date, customer_name, amount) VALUES 
(20220101, '2024-01-01', 'Alice', 100),
(20220102, '2024-01-02', 'Bob', 150),
(20220103, '2024-01-03', 'Charlie', 200);

TRUNCATE orders WHERE id=20220101;
```

---

## SQL 标准兼容性

SQL:2008 标准包含一个 `TRUNCATE` 命令，语法为 `TRUNCATE TABLE <tablename>`。这个命令的一些并发行为被标准定义为实现定义，因此如果必要，应考虑上述注意事项并与其他实现进行比较。
