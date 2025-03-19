SELECT INTO
=====

创建表，且该表使用查询计算产生的数据作为其内容。


---

语法
--------

```sql
[ WITH [ RECURSIVE ] <with_query> [, ...] ]
SELECT [ALL | DISTINCT [ON ( <expression> [, ...] )]]
    * | <expression> [AS <output_name>] [, ...]
    INTO  [TABLE] <new_table>
    [FROM <from_item> [, ...]]
    [WHERE <condition>]
    [GROUP BY <expression> [, ...]]
    [HAVING <condition> [, ...]]
    [WINDOW <window_name> AS ( <window_definition> ) [, ...]]
    [{UNION | INTERSECT | EXCEPT} [ALL | DISTINCT ] <select>]
    [ORDER BY <expression> [ASC | DESC | USING <operator>] [NULLS {FIRST | LAST}] [, ...]]
    [LIMIT {<count> | ALL}]
    [OFFSET <start> [ ROW | ROWS ] ]
    [FETCH { FIRST | NEXT } [ <count> ] { ROW | ROWS } ONLY ]
```

---

描述
----------

当你使用 `SELECT INTO` 创建表时，填充在表中的数据是查询的结果。因此，表中每一列的名称和数据类型与查询结果中的对应列的名称和数据类型相同。然而，MAXIR 不建议你使用 `SELECT INTO` 来创建表。

:::note
查询的结果不返回给客户端。
:::

---

参数
----------

*`<new_table>`*：要创建的表的名称。你可以在名称前加上 Schema 名称。

关于其他参数的信息，请参见 [SELECT](select.md#参数) 中的“参数”部分。


---

示例
----------

创建一个名为 `sales_2022` 的表，该表包含 `sales` 表中 `year` 列的值为 `2022` 的行：

```sql
SELECT *
INTO sales_2022
FROM sales
WHERE year = 2022;
```

---

SQL 标准兼容性
-------------

在 MAXIR 中，`SELECT INTO` 创建一个新表，而在 SQL 标准中，`SELECT INTO` 将值选择到标量变量中。
