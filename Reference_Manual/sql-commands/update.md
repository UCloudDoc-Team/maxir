UPDATE
=====

更新表的行。


---



语法
--------

```sql
UPDATE [ ONLY ] <table_name> [ [ AS ] <alias> ]
   SET { <column_name> = { <expression> | DEFAULT } |
       ( <column_name> [, ...] ) = [ ROW ] ( { <expression> | DEFAULT } [, ...] ) |
       ( <column_name> [, ...] ) = ( <sub-SELECT> )
       } [, ...]
   [ FROM <from_item> [, ...] ]
   [ WHERE <condition> ]
```

---



描述
----------

`UPDATE` 更改满足条件的所有行中指定列的值。只需要在 `SET` 子句中提到要修改的列。未明确修改的列保留其以前的值。

:::warning 注意
关于 `SET` 子句，Hybrid DPS 和 Extreme DPS 的支持程度不同。关于 Extreme DPS 在 `UPDATE ... SET` 语句上的使用限制，请参考 [使用注意事项](#使用注意事项)。
:::


`UPDATE` 语句支持两种方式使用数据库中其他表的信息来修改表：一种是使用子 `SELECT` 语句，另一种是在 `FROM` 子句中指定额外的表。具体使用哪一种方式，请根据实际情况进行选择。

你必须具备表的 `UPDATE` 权限，或者至少具备对要更新的列的 `UPDATE` 权限。同时，对在表达式或条件中读取值的任意列，你还必须具备 `SELECT` 权限。

:::note
默认情况下，MAXIR 在堆表的 `UPDATE` 操作上获取 `EXCLUSIVE` 锁。当启用全局死锁检测器时，堆表的 `UPDATE` 操作的锁模式为 `ROW EXCLUSIVE`。
:::

---

参数
----------


- *`<table_name>`*

    要更新的表的名称（支持 Schema 限定）。如果在表名前指定了 `ONLY`，则只更新指定表中的匹配行。如果未指定 `ONLY`，则会更新该表及其继承表中的匹配行。你可以在表名后面指定 `*` 来明确表示包含子表。

- *`<alias>`*

    目标表的替代名字。当提供别名时，它会完全隐藏表的实际名称。例如，给定 `UPDATE foo AS f`，`UPDATE` 语句的其余部分必须将该表称为 f 而不是 foo。

- *`<column_name>`*

    列的名称。如果需要，列名可以用子字段名或数组下标进行限定。在指定目标列时，不要在列的规定中包含表的名称；例如，`UPDATE table_name SET table_name.col = 1` 是无效的。

- *`<expression>`*

    分配给列的表达式。表达式可以使用表中此列和其他列的旧值。

- **`DEFAULT`**

    将列设置为其默认值（如果没有为其分配特定的默认表达式，则值为 NULL）。


- **`SELECT`** 子查询

    产生与前面的括号列列表中列数一样多的输出列的 `SELECT` 子查询。执行时子查询最多不能产生超过一行。如果它产生一行，它的列值将分配给目标列。如果它不产生行，则将 NULL 值分配给目标列。子查询可以引用正在更新的表的当前行的旧值。

- *`<from_item>`*

    一个表达式，允许其他表的列出现在 `WHERE` 条件和更新表达式中。这使用与 `SELECT` 语句的 `FROM` 子句相同的语法。例如，你可以为表名指定一个别名。除非你打算自连接（在这种情况下，它必须在 *`<from_item>`* 中带有别名），否则不要重复目标表作为 *`<from_item>`*。

- *`<condition>`*

    返回 `boolean` 类型值的表达式。只有对于这个表达式返回 true 的行才会被更新。


---

输出
----------

成功完成后，`UPDATE` 命令返回一个形式为的命令标签：

```sql
UPDATE <count>
```

计数是更新的行数，包括匹配的行，其值没有改变。如果计数为 0，则查询未更新任何行（这不被视为错误）。



---

使用注意事项
----------


`UPDATE` 语句中的如下子句或语法，只有 Hybrid DPS 支持。如需执行相关语句，请将执行 DPS 集群指定为 Hybrid DPS 集群：

- `SET DEFAULT`

- 使用 `SET` 对多个列进行更新，且 `SET` 子句中使用了括号，例如：`UPDATE t1 SET (a1,a2) = (1,2);`

- 对指定了别名的表进行 `UPDATE`，例如：

    ```sql
    CREATE TABLE t1 (a1 bigint);

    UPDATE t1 d1 SET a1 = 1
    ```

当存在 `FROM` 子句时，目标表与 *`<from_item>`* 列表中提到的表进行连接，join 的每个输出行代表目标表的一个更新操作。使用 `FROM` 时，确保 join 对每行要修改的最多产生一个输出行。换句话说，目标行不应该与其他表的多个行 join。如果这样做了，那么 join 行中的仅一个将用于更新目标行，但哪一个将被用于不容易预测。

由于这种不确定性，只在子选择中引用其他表更安全，尽管这样做通常比使用 join 难以阅读且慢。

---

示例
----------

更改表 `films` 中列 kind 的单词 `Drama` 为 `Dramatic`：

```sql
UPDATE films SET kind = 'Dramatic' WHERE kind = 'Drama';
```

在表 `weather` 的一行中调整温度条目并将降水量重置为其默认值：

```sql
UPDATE weather SET temp_lo = temp_lo+1, temp_hi = 
temp_lo+15, prcp = DEFAULT
WHERE city = 'San Francisco' AND date = '2016-07-03';
```

执行相同的操作并返回更新的条目：

```sql
UPDATE weather SET temp_lo = temp_lo+1, temp_hi = temp_lo+15, prcp = DEFAULT
  WHERE city = 'San Francisco' AND date = '2003-07-03'
  RETURNING temp_lo, temp_hi, prcp;
```

使用列列表语法的替代方式进行相同的更新：

```sql
UPDATE weather SET (temp_lo, temp_hi, prcp) = (temp_lo+1, 
temp_lo+15, DEFAULT)
WHERE city = 'San Francisco' AND date = '2016-07-03';
```

增加管理 Acme Corporation 账户的销售员的销售数量，使用 `FROM` 子句语法（假设两个要连接的表都在 MAXIR 的 `id` 列上分布）：

```sql
UPDATE employees SET sales_count = sales_count + 1 FROM 
accounts
WHERE accounts.name = 'Acme Corporation'
AND employees.id = accounts.id;
```

执行相同的操作，使用 `WHERE` 子句中的子选择：

```sql
UPDATE employees SET sales_count = sales_count + 1 WHERE id =
  (SELECT id FROM accounts WHERE name = 'Acme Corporation');
```

更新 `accounts` 表中的联系人名称以匹配当前分配的销售员：

```sql
UPDATE accounts SET (contact_first_name, contact_last_name) =
    (SELECT first_name, last_name FROM salesmen
     WHERE salesmen.id = accounts.sales_id);
```

可以通过 join 达到类似的结果：

```sql
UPDATE accounts SET contact_first_name = first_name,
                    contact_last_name = last_name
  FROM salesmen WHERE salesmen.id = accounts.sales_id;
```

然而，如果 `salesmen.id` 不是唯一键，那么第二个查询可能会给出意外的结果，而第一个查询保证在有多个 `id` 匹配时会引发错误。此外，如果对于某个特定的 `accounts.sales_id` 条目没有匹配，那么第一个查询将设置相应的姓名字段为 NULL，而第二个查询不会更新那一行。

更新 `summary` 表中的统计信息以匹配当前数据：

```sql
UPDATE summary s SET (sum_x, sum_y, avg_x, avg_y) =
    (SELECT sum(x), sum(y), avg(x), avg(y) FROM data d
     WHERE d.group_id = s.group_id);
```

试图插入一个新的库存条目以及库存数量。如果条目已经存在，那么更新现有条目的库存数量。为了在不失败整个事务的情况下做到这一点，使用保存点。

```sql
BEGIN;
-- other operations
SAVEPOINT sp1;
INSERT INTO wines VALUES('Chateau Lafite 2003', '24');
-- Assume the above fails because of a unique key violation,
-- so now we issue these commands:
ROLLBACK TO sp1;
UPDATE wines SET stock = stock + 24 WHERE winename = 'Chateau 
Lafite 2003';
-- continue with other operations, and eventually
COMMIT;
```

---

SQL 标准兼容性
----------

此命令符合 SQL 标准，除了 `FROM` 子句是 MAXIR 的扩展。

一些其他数据库系统提供一个 `FROM` 选项，其中目标表应在 `FROM` 中再次列出。这不是 MAXIR 如何解释 `FROM`。在移植使用此扩展的应用程序时要小心。

根据标准，目标列名的括号子列表的源值可以是任何产生正确列数的行值表达式。MAXIR 只允许源值是行构造器或子 `SELECT`。在行构造器情况下，你可以将个别列的更新值指定为 `DEFAULT`，但不能在子 `SELECT` 中这样做。
