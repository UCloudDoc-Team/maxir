INSERT
=====

向表中添加行。


---

语法
--------

```sql
INSERT INTO <table_name> [( <column_name> [, ...] )]
   {DEFAULT VALUES | VALUES ( {<expression> | DEFAULT} [, ...] ) [, ...] | <query>}
   [RETURNING * | <output_expr> [[AS] <output_name>] [, ...]]
```


---

描述
--------
`INSERT` 命令让你可以插入一个或多个由值表达式指定或由查询结果的行。

你可以在命令中指定列来仅向指定的列插入数据。这些列可以按任何顺序指定。如果你没有指定列，那么默认情况下，将按声明的顺序使用表的列。从 `VALUES` 子句或查询获取的值将从左到右与显式或隐式的列列表相关联。

如果列没有明确地或隐式地列出，那么它将被分配其声明的默认值，或者如果没有默认值，则为 null。

如果列的表达式的数据类型不正确，系统将自动转换数据类型。

如果你在你的命令中包括 `RETURNING`，那么命令还将根据每个插入的行计算并返回值。如果你想获取由默认值提供的值，如序列号，那么 `RETURNING` 子句非常有用。

要在表上执行 `INSERT`，你必须在表上拥有 `INSERT` 权限。如果你在 `INSERT` 命令中指定了多个列，那么你必须在每个列上都有 `INSERT` 权限。如果你还想在你的命令中包括 `RETURNING`，那么你还必须在每个列上都有 `SELECT` 权限。


---

参数
----------

- *`<table_name>`*

   表的名称。你可以在名称前添加 Schema 名称，以在指定的 Schema 中插入表。

- *`<column_name>`*

   列的名称。你可以用限定符指定名称，如子字段名称或数组下标。

- **`[DEFAULT VALUES]`**

   填充所有列的默认值的选项。

- **`[<expression> | DEFAULT]`**

   要分配给该列的表达式或值。或者，你可以指定 `DEFAULT` 来填充相应列的默认值。

- *`<query>`*

   查询，即 `SELECT` 语句，用于提供要插入的行。

- *`<output_expression>`*

   插入每一行后要计算或返回的表达式。将此参数设置为 `*` 以返回插入行的所有列。

- *`<output_name>`*

   返回输出的名称。


---

输出
----------

成功执行 `INSERT` 命令后，将返回以下格式的命令标签：

```sql
INSERT <oid> <count>
```
*`<count>`* 表示插入的行数。*`<oid>`* 表示对象 ID (OID)。如果 *`<count>`* 为0且给定表具有 OID，则 *`<oid>`* 是插入行的 OID。否则， *`<oid>`* 为 0。


---

示例
----------

向 `sales` 表插入一行：

```sql
INSERT INTO sales VALUES (1, 2022, 1, 1, 'S', 'CA');
```

向 `sales` 表插入一行，但未为列 `qtr` 指定值：

```sql
INSERT INTO sales (id, year, c_rank, code, region) 
VALUES (1, 2023, 1, 'S', 'CA');
```
 
通过为列 `year` 指定 `DEFAULT` 子句向 `sales` 表插入一行：

```sql
INSERT INTO sales (id, year, qtr, c_rank, code, region) 
VALUES (1, DEFAULT, 1, 1, 'S', 'CA');
```

使用所有列的默认值向 `sales` 表插入一行：

```sql
INSERT INTO sales DEFAULT VALUES;
```

向 `sales` 表插入多行：

```sql
INSERT INTO sales (id, year, qtr, c_rank, code, region) 
VALUES 
(1, 2023, 1, 1, 'S', 'CA'),
(2, 2023, 2, 2, 'R', 'NY'),
(3, 2023, 3, 3, 'S', 'TX');
```


---


SQL 标准兼容性
----------

MAXIR 中的 `INSERT` 完全兼容 SQL 标准。唯一的区别是，如果在 `INSERT` 命令中没有指定列名称列表，那么 SQL 标准要求从 `VALUES` 子句或查询中填充所有列。
