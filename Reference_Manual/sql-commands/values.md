VALUES
=====

计算一组行。

---

语法
--------

```sql
VALUES ( <expression> [, ...] ) [, ...]
   [ORDER BY <sort_expression> [ ASC | DESC | USING <operator> ] [, ...] ]
   [LIMIT { <count> | ALL } ] 
   [OFFSET <start> [ ROW | ROWS ] ]
   [FETCH { FIRST | NEXT } [<count> ] { ROW | ROWS } ONLY ]
```


---

描述
----------

`VALUES` 计算由值表达式指定的行值或行值集。它最常用于在更大的命令中生成一个“常量表”，但它可以单独使用。

如果指定了多行，它们必须都包含相同数量的元素。通过将该列中的表达式的类型组合在一起，按照 `UNION` 的规则决定结果表列的数据类型。

在更大的命令中，`VALUES` 可以在允许 `SELECT` 的地方使用。因为语法将 `VALUES` 视为 `SELECT`，所以你可以与 `VALUES` 命令一起使用 `ORDER BY`、`LIMIT`（或 `FETCH FIRST`）和 `OFFSET` 子句。

---

参数
----------

- _`<expression>`_

    一个常量或表达式，用于计算并在生成的表（行集）的指定位置插入。在 `INSERT` 的顶层出现的 `VALUES` 列表中，表达式可以被 `DEFAULT` 替换，表示应插入目标列的默认值。当 `VALUES` 出现在其他上下文中时，不能使用 `DEFAULT`。

- _`<sort_expression>`_

    一个表达式或整数常量，表示如何对结果行进行排序。这个表达式可以引用 `VALUES` 结果的列，如 `column1`、`column2` 等。更多信息，请参见 [SELECT](/maxir/Reference_Manual/sql-commands/select.md#order-by-子句) 参数中的“ORDER BY 子句”。

- _`<operator>`_

    排序运算符。更多信息，请参见 [SELECT](/maxir/Reference_Manual/sql-commands/select.md#order-by-子句) 参数中的“ORDER BY 子句”。

- **`LIMIT <count>`** 或 **`OFFSET <start>`**

    返回的最大行数。更多信息，请参见 [SELECT](/maxir/Reference_Manual/sql-commands/select.md#order-by-子句) 参数中的“ORDER BY 子句”。



---

使用注意事项
----------

建议你避免使用大量行的 `VALUES` 列表，以防止内存不足故障或性能差。在 `INSERT` 命令中使用 `VALUES` 是一个例外。这是因为 `INSERT` 的目标表提供了所需的列类型，无需通过扫描 `VALUES` 列表来推断它们。这使它能够处理比其他上下文中的列表更大的列表。


---

示例
----------

一个简单的 `VALUES` 命令：

```sql
VALUES 
   (1, 'one'), 
   (2, 'two'), 
   (3, 'three');
```

这将返回一个具有两列和三行的表。它实际上等效于：

```sql
SELECT 1 AS column1, 
    'one' AS column2
UNION ALL
SELECT 
    2, 
    'two'
UNION ALL
SELECT 
    3, 
    'three';
```

在大多数情况下，`VALUES` 用于更大的 SQL 命令中。最常见的用途是在 `INSERT` 中：

```sql
INSERT INTO films 
(
    code, 
    title, 
    did, 
    date_prod, 
    kind
)
VALUES 
(
    'T_601', 
    'Yojimbo', 
    106, 
    '1961-06-16', 
    'Drama'
);
```

在 `INSERT` 子句的 `VALUES` 列表中，所有值将自动适应目标列的数据类型。在其他使用上下文中，你可能需要手动指定正确的数据类型。如果所有条目都是带引号的文字常量，只需调整第一个条目即可，因为这将设定所有后续条目的假定类型：

```sql
SELECT * FROM machines WHERE ip_address IN 
(VALUES('192.168.0.1'::inet), ('192.168.0.10'), 
('192.0.2.43'));
```

>note
>对于简单的 `IN` 测试，最好依赖于 `IN` 的标量列表形式，而不是编写如上所示的 `VALUES` 查询。涉及标量列表的方法需要较少的编写，并且通常更有效。



---

SQL 标准兼容性
-------------

`VALUES` 符合 SQL 标准。`LIMIT` 和 `OFFSET` 是 MAXIR 的扩展。也可以参见 [SELECT](/maxir/Reference_Manual/sql-commands/select.md)。
