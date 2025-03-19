SELECT
=====

从一个或多个表或视图中检索行。


---

语法
--------

```sql
[ WITH [ RECURSIVE ] <with_query> [, ...] ]
SELECT [ALL | DISTINCT [ON (<expression> [, ...])]]
  [* | <expression >[[AS] <output_name>] [, ...]]
  [FROM <from_item> [, ...]]
  [WHERE <condition>]
  [GROUP BY <grouping_element> [, ...]]
  [HAVING <condition> [, ...]]
  [WINDOW <window_name> AS (<window_definition>) [, ...] ]
  [{UNION | INTERSECT | EXCEPT} [ALL | DISTINCT] <select>]
  [ORDER BY <expression> [ASC | DESC ] [NULLS {FIRST | LAST}] [, ...]]
  [LIMIT {<count> | ALL}]
  [OFFSET <start> [ ROW | ROWS ] ]
  [FETCH { FIRST | NEXT } [ <count> ] { ROW | ROWS } ONLY]


其中 <from_item> 可以是：

[ONLY] <table_name> [ * ] [ [ AS ] <alias> [ ( <column_alias> [, ...] ) ] ]
( <select> ) [ AS ] <alias> [( <column_alias> [, ...] ) ]
<with_query_name> [ [ AS ] <alias> [ ( <column_alias> [, ...] ) ] ]
<function_name> ( [ <argument> [, ...] ] )
            [ WITH ORDINALITY ] [ [ AS ] <alias> [ ( <column_alias> [, ...] ) ] ]
<function_name> ( [ <argument> [, ...] ] ) [ AS ] <alias> ( <column_definition> [, ...] )
<function_name> ( [ <argument> [, ...] ] ) AS ( <column_definition> [, ...] )
ROWS FROM( function_name ( [ argument [, ...] ] ) [ AS ( column_definition [, ...] ) ] [, ...] )
            [ WITH ORDINALITY ] [ [ AS ] <alias> [ ( <column_alias> [, ...] ) ] ]
<from_item> [ NATURAL ] <join_type> <from_item>
          [ ON <join_condition> | USING ( <join_column> [, ...] ) ]

其中 <grouping_element> 可以是：

  ()
  <expression>
  ROLLUP (<expression> [,...])
  CUBE (<expression> [,...])
  GROUPING SETS ((<grouping_element> [, ...]))

其中 <window_definition> 可以是：

  [<existing_window_name>]
  [PARTITION BY <expression> [, ...]]
  [ORDER BY <expression> [ASC | DESC ] [NULLS {FIRST | LAST}] [, ...]]
  [ frame_clause ]

其中 <with_query> 是：

  <with_query_name> [( <column_name> [, ...] )] AS ( [ NOT ] MATERIALIZED ] ( <select> | <values> | <insert> | <update> | delete )

TABLE [ ONLY ] <table_name> [ * ]
```


---

描述
----------

`SELECT` 从零个或多个表中检索行。 `SELECT` 的一般处理流程如下：

1. 计算 `WITH` 子句中的所有查询。这些实际上作为临时表，可以在 `FROM` 列表中引用。

2. 计算 `FROM` 列表中的所有元素。 `FROM` 列表中的每个元素都是真实或虚拟的表。如果在 `FROM` 列表中指定了多个元素，它们将被交叉连接在一起。

3. 如果指定了 `WHERE` 子句，那么所有不满足条件的行将从输出中被消除。

4. 如果指定了 `GROUP BY` 子句，或者有聚合函数调用，那么输出将被组合成在一个或多个值上匹配的行的组，并计算聚合函数的结果。如果存在 `HAVING` 子句，它会消除不满足给定条件的组。

5. 使用 `SELECT` 输出表达式为每个选定的行或行组计算实际的输出行。

6. `SELECT DISTINCT` 从结果中消除重复的行。 `SELECT DISTINCT ON` 消除匹配所有指定表达式的行。 `SELECT ALL`（默认）将返回所有候选行，包括重复项。

7. 如果指定了窗口表达式（和可选的 `WINDOW` 子句），那么输出将根据位置（行）或基于值（范围）的窗口帧进行组织。

8. 使用 `SELECT` 输出表达式为每个选定的行计算实际的输出行。

9. 使用运算符 `UNION`、 `INTERSECT` 和 `EXCEPT`，可以将一个以上 `SELECT` 语句的输出组合成一个结果集。 `UNION` 运算符返回在一个或两个结果集中的所有行。 `INTERSECT` 运算符返回严格在两个结果集中的所有行。 `EXCEPT` 运算符返回在第一个结果集中但不在第二个结果集中的行。在所有这三种情况下，除非指定了 `ALL`，否则重复的行将被消除。可以添加噪声词 `DISTINCT` 来明确指定消除重复行。注意，即使 `ALL` 是 `SELECT` 本身的默认行为，`DISTINCT` 仍然是这里的默认行为。

10. 如果指定了 `ORDER BY` 子句，返回的行将按指定的顺序排序。如果没有给出 `ORDER BY`，那么行将以系统找到的最快的顺序返回。

11. 如果指定了 `LIMIT`（或 `FETCH FIRST`）或 `OFFSET` 子句， `SELECT` 命令只返回结果行的一个子集。

在 `SELECT` 命令中使用的每一列都必须具有 `SELECT` 权限。


---

参数
----------

 

### WITH 子句

可选的 `WITH` 子句允许你指定一个或多个可以在主查询中通过名称引用的子查询。子查询在主查询的生命周期内有效地充当临时表或视图。每个子查询可以是 `SELECT`、`INSERT`、`UPDATE` 或 `DELETE` 语句。在 `WITH` 中编写数据修改语句（ `INSERT`、`UPDATE` 或 `DELETE`）时，通常会包含一个 `RETURNING` 子句。构成主查询读取的临时表的是 `RETURNING` 的输出，而不是该语句修改的底层表。如果省略了 `RETURNING`，该语句仍然会运行，但它不产生输出，因此不能被主查询作为表引用。

对于包含 `WITH` 子句的 `SELECT` 命令，子句最多只能包含一个修改表数据的子句（ `INSERT`、`UPDATE` 或 `DELETE` 命令）。

必须为 `WITH` 子句中的每个查询指定一个没有 Schema 限定的 `<with_query_name>`。可以选择指定一列名称列表；如果省略了列名称列表，将从子查询中推断名称。主查询和 `WITH` 查询都（概念上）同时运行。

如果指定了 `RECURSIVE`，它允许 `SELECT` 子查询通过名称引用自身。这样的子查询的一般形式为

```sql
<non_recursive_term> UNION [ALL | DISTINCT] <recursive_term>
```

其中递归自引用出现在 `UNION` 的右侧。每个查询只允许一个递归自引用。不支持递归的数据修改语句，但可以在数据修改语句中使用递归 `SELECT` 查询的结果。

如果指定了 `RECURSIVE` 关键字， `WITH` 查询不需要排序：一个查询可以引用列表中稍后的另一个查询。然而，不支持循环引用或互递归。

如果没有 `RECURSIVE` 关键字， `WITH` 查询只能引用 `WITH` 列表中较早的兄弟 `WITH` 查询。

带有 `RECURSIVE` 关键字，以下项目不受支持：
- 包含以下内容的 _`<recursive_term>`_ 的递归 `WITH` 子句：
  - 带有自引用的子查询
  - `DISTINCT` 子句
  - `GROUP BY` 子句
  - 窗口函数
- `<with_query_name>` 是集合操作的一部分的递归 `WITH` 子句

以下是集合操作限制的示例。这个查询会返回一个错误，因为集合操作 `UNION` 包含对表 `foo` 的引用。


```sql
WITH RECURSIVE foo (i)
AS (
	SELECT 1
	
	UNION ALL
	
	SELECT i + 1
	FROM (
		SELECT *
		FROM foo
		
		UNION
		
		SELECT 0
		) bar
	)
SELECT *
FROM foo LIMIT 5;
```

这个递归的公共表达式 (CTE) 是允许的，因为集合操作 `UNION` 没有对 CTE `foo` 的引用。

```sql
WITH RECURSIVE foo (i)
AS (
	SELECT 1
	
	UNION ALL
	
	SELECT i + 1
	FROM (
		SELECT *
		FROM bar
		
		UNION
		
		SELECT 0
		) bar
		,foo
	WHERE foo.i = bar.a
	)
SELECT *
FROM foo LIMIT 5;
```

`WITH` 查询的一个关键属性是它们在执行主查询时只被评估一次，即使主查询多次引用它们。特别是，无论主查询是否读取所有或任何输出，都保证只运行一次数据修改语句。

主查询和 `WITH` 查询都（概念上）同时运行。这意味着，除了通过读取其 `RETURNING` 输出之外，不能从查询的其他部分看到 `WITH` 中的数据修改语句的效果。如果两个这样的数据修改语句试图修改同一行，结果是未指定的。


 

### SELECT 列表

`SELECT` 列表（位于 `SELECT` 和 `FROM` 关键字之间）指定构成 `SELECT` 语句输出行的表达式。表达式可以引用在 `FROM` 子句中计算的列。

`SELECT` 列表中的表达式可以是常数值，列引用，运算符调用，函数调用，聚合表达式，窗口表达式，标量子查询等等。许多结构可以被分类为表达式，但并不遵循任何通用的语法规则。这些通常具有函数或运算符的语义。

就像在表中一样， `SELECT` 的每个输出列都有一个名称。在简单的 `SELECT` 中，这个名称只用于标记列的显示，但是当 `SELECT` 是一个更大查询的子查询时，这个名称被更大的查询看作是由子查询产生的虚拟表的列名称。要指定用于输出列的名称，可以在列的表达式之后写 `AS <output_name>`。你可以省略 `AS`，但只有在期望的输出名称不匹配任何 SQL 关键字时才能这样做。为了防止可能的未来关键字的添加，你总是可以写 `AS` 或双引号输出名称。如果你不指定列名称，MAXIR 会自动选择一个名称。如果列的表达式是一个简单的列引用，那么选择的名称将与该列名称相同。在更复杂的情况下，可以使用函数或类型名称，或者系统可以回退到生成的名称，如 `?column?` 或 `columnN`。

可以使用输出列的名称在 `ORDER BY` 和 `GROUP BY` 子句中引用列的值，但不能在 `WHERE` 或 `HAVING` 子句中这样做。在这种情况下，你必须写出表达式。

在输出列表中，可以写 `*` 作为选定行的所有列的简写。此外，你可以写 `<table_name>.*` 作为仅来自该表的列的简写。在这些情况下，无法使用 `AS` 指定新的名称。输出列名称将与表的列名称相同。


 

### DISTINCT 子句

如果指定了 `SELECT DISTINCT`，所有重复的行都将从结果集中移除（每组重复中保留一行）。如果指定了 `SELECT ALL`，将保留所有行。 `SELECT ALL` 是默认的。

`SELECT DISTINCT ON ( expression [, ...] )` 仅保留每组行中的第一行，其中给定的表达式计算为相等。 `DISTINCT ON` 表达式使用与 `ORDER BY`（参见上文）相同的规则进行解释。请注意，除非使用 `ORDER BY` 确保所需的行首先出现，否则每组的“第一行”是不可预测的。例如：

```sql
SELECT DISTINCT ON (location) location
	,TIME
	,report
FROM weather_reports
ORDER BY location
	,TIME DESC;
```

这个 `SELECT` 检索每个位置的最新天气报告。但是，如果没有使用 `ORDER BY` 强制每个位置的时间值按降序排列，那么将返回每个位置的不可预测时间的报告。

`DISTINCT ON` 表达式必须与最左边的 `ORDER BY` 表达式匹配。 `ORDER BY` 子句通常会包含额外的表达式，这些表达式确定了 `DISTINCT ON` 组内行的期望优先级。

 

### FROM 子句

`FROM` 子句为 `SELECT` 指定一个或多个源表。如果指定了多个源，结果将是所有源的笛卡尔积（cross join）。但通常通过使用 `WHERE` 添加限定条件来将返回的行限制为笛卡尔积的一个小子集。 `FROM` 子句可以包含以下元素：

- *`<table_name>`*

  现有表或视图的名称（可选择带 Schema 限定）。如果指定了 `ONLY`，则只会扫描该表。如果没有指定 `ONLY`，将会扫描表及其所有子表（如果有的话）。

- *`<alias>`*

  包含别名的 `FROM` 项的替代名称。别名用于简洁或消除自连接（同一表被多次扫描）的模糊性。当提供别名时，它将完全隐藏表或函数的实际名称。例如，给出 `FROM foo AS f`， `SELECT` 的其余部分必须将此 `FROM` 项称为 `f` 而不是 `foo`。如果写了一个别名，也可以写一个列别名列表，为表的一个或多个列提供替代名称。

- *`<select>`*

  一个子 `SELECT` 可以出现在 `FROM` 子句中。这就好像它的输出在这个单独的 `SELECT` 命令的期间被创建为一个临时表一样。注意，子 `SELECT` 必须用括号括起来，并且必须为它提供一个别名。 [VALUES](values.md) 命令也可以在这里使用。请参见 [SQL 标准兼容性](#sql-标准兼容性) 部分的“非标准子句”以了解在 MAXIR 中使用相关子选择的限制。

- *`<with_query_name>`*

  通过指定其 *`<with_query_name>`* 在 `FROM` 子句中引用一个 *`<with_query>`*，就像该名称是一个表名称一样。 *`<with_query_name>`* 不能包含 Schema 限定符。可以以与表相同的方式提供别名。

  *`<with_query>`* 隐藏了一个具有相同名称的表，用于主查询。如果需要，你可以通过限定表名称与 Schema 来引用具有相同名称的表。

- *`<function_name>`*

  函数调用可以出现在 `FROM` 子句中。这对于返回结果集的函数特别有用，但可以使用任何函数。这就好像它的输出在这个单独的 `SELECT` 命令的期间被创建为一个临时表一样。也可以使用别名。如果写了一个别名，也可以写一个列别名列表，为函数的复合返回类型的一个或多个属性提供替代名称。如果函数已被定义为返回记录数据类型，则必须存在别名或关键字 `AS`，后跟形式为 `( <column_name> <data_type> [, ... ] )` 的列定义列表。列定义列表必须与函数返回的列的实际数量和类型匹配。

- *`<join_type>`*

  *`<join_type>`* 可以设置为：

  - `[INNER] JOIN`
  - `LEFT [OUTER] JOIN`
  - `RIGHT [OUTER] JOIN`
  - `FULL [OUTER] JOIN`
  - `CROSS JOIN`

  对于 `INNER` 和 `OUTER` 连接类型，必须指定一个连接条件，即 `NATURAL`，`ON <join_condition>` 或 `USING ( <join_column> [, ...])` 中的一个。详见下文。对于 `CROSS JOIN`，这些子句都不能出现。

  JOIN 子句组合了两个 `FROM` 项，为了方便我们将其称为“表”，尽管实际上它们可以是任何类型的 `FROM` 项。如果需要，使用括号确定嵌套的顺序。在没有括号的情况下， `JOIN` 从左到右嵌套。无论如何， `JOIN` 比分隔 `FROM` 列表项的逗号绑定得更紧密。

  `CROSS JOIN` 和 `INNER JOIN` 产生一个简单的笛卡尔积，这与你从 `FROM` 的最外层列表中列出两个表得到的结果相同，但是通过限定条件（如果有）限制了其结果。 `CROSS JOIN` 等效于 `INNER JOIN ON (TRUE)`，也就是说，没有行被限定条件移除。这些连接类型只是一种记法便利，因为它们没有你不能用普通的 `FROM` 和 `WHERE` 做的事情。

  `LEFT OUTER JOIN` 返回限定笛卡尔积中的所有行（例如，所有通过其连接条件的组合行），再加上左表中每一行没有通过连接条件的右表行的一份拷贝。这个左表行会通过插入右表列的空值来扩展到连接表的完整宽度。注意，只有 `JOIN` 子句自己的条件在决定哪些行有匹配时才被考虑。外部条件在之后被应用。

  相反， `RIGHT OUTER JOIN` 返回所有连接的行，再加上每个未匹配的右表行的一行（在左边扩展空值）。这只是一种记法便利，因为你可以通过交换左表和右表将其转换为 `LEFT OUTER JOIN`。

  `FULL OUTER JOIN` 返回所有连接的行，再加上每个未匹配的左表行的一行（在右边扩展空值），再加上每个未匹配的右表行的一行（在左边扩展空值）。

- **`ON <join_condition>`**

  *`<join_condition>`* 是一个结果为 `boolean` 类型的表达式（类似于 `WHERE` 子句），指定在连接中哪些行被认为是匹配的。


- **`USING (<join_column> [, ...])`**

  形如 `USING ( a, b, ... )` 的子句是 `ON left_table.a = right_table.a AND left_table.b = right_table.b ...` 的简写。此外， `USING` 意味着在连接输出中只会包括等价列的一对中的一个，而不是两者都包括。

- **`NATURAL`**

  `NATURAL` 是一个 `USING` 列表的简写，它提到了两个表中所有具有相同名称的列。如果没有公共列名称， `NATURAL` 等效于 `ON TRUE`。

 

### WHERE 子句

可选的 `WHERE` 子句的一般形式为：

```sql
WHERE <condition>
```
    

其中 *`<condition>`* 是任何求值结果为 `boolean` 类型的表达式。任何不满足此条件的行都将从输出中消除。如果当实际行值替换任何变量引用时返回 true，则行满足条件。


 

### GROUP BY 子句

可选的 `GROUP BY` 子句的一般形式为：

```sql
GROUP BY <grouping_element> [, ...]
```
    

其中 `<grouping_element>` 可以是以下之一：

```sql
()
<expression>
ROLLUP (<expression> [,...])
CUBE (<expression> [,...])
GROUPING SETS ((<grouping_element> [, ...]))
```
    

`GROUP BY` 将所有选定的具有相同组合表达式的值的行压缩为单个行。表达式可以是输入列名称，或者输出列的名称或序数（`SELECT` 列表项），或者由输入列值形成的任意表达式。在有歧义的情况下， `GROUP BY` 名称将被解释为输入列名称而不是输出列名称。

如果使用了任何聚合函数，它们会在组成每个组的所有行上计算，为每个组产生一个独立的值。如果有聚合函数但没有 `GROUP BY` 子句，查询将被视为具有由所有选定的行组成的单个组。可以通过将 `FILTER` 子句附加到聚合函数调用来进一步过滤发送到每个聚合函数的行集。当存在 `FILTER` 子句时，只有匹配它的行才会包含在该聚合函数的输入中。

当存在 `GROUP BY`，或者存在任何聚合函数时， `SELECT` 列表表达式引用未组合的列是无效的，除非在聚合函数中或者当未组合的列在功能上依赖于组合的列时，因为对于未组合的列可能会返回多个可能的值。如果组合的列（或其子集）是包含未组合列的表的主键，则存在功能依赖性。

请记住，在评估 `HAVING` 子句或 `SELECT` 列表中的任何“标量”表达式之前，会先评估所有聚合函数。这意味着，例如，你不能使用 `CASE` 表达式来跳过评估聚合函数。

MAXIR 有以下附加的 OLAP 分组扩展（通常被称为 _supergroups_）：

- **`ROLLUP`**

  `ROLLUP` 分组是 `GROUP BY` 子句的一个扩展，它创建了从最详细的级别到总计的聚合小计，这些小计遵循一列分组列（或表达式）。 `ROLLUP` 获取一列有序的分组列，计算在 `GROUP BY` 子句中指定的标准聚合值，然后从右到左通过列表创建逐渐高级别的小计。最后，它创建一个总计。 `ROLLUP` 分组可以看作是一系列分组集。例如：

    ```sql
    GROUP BY ROLLUP (a,b,c) 
    ```
  
    等价于：
  
    ```sql
    GROUP BY GROUPING SETS( (a,b,c), (a,b), (a), () )
    ``` 
    

  请注意， `ROLLUP` 的 *n* 个元素转换为 *n*+1 个分组集。此外，分组表达式的指定顺序在 `ROLLUP` 中是重要的。

- **`CUBE`**

  `CUBE` 分组是 `GROUP BY` 子句的一个扩展，它创建了给定分组列或表达式的所有可能组合的小计。在多维分析方面， `CUBE` 生成了可以为具有指定维度的数据立方体计算的所有小计。例如：

    ```sql
    GROUP BY CUBE (a,b,c)
    ```
  等价于：
    
    ```sql
    GROUP BY GROUPING SETS( (a,b,c), (a,b), (a,c), (b,c), (a), 
    (b), (c), () )
    ```
    

  请注意， `CUBE` 的 *n* 个元素转换为 2*n* 个分组集。在需要交叉表报告的任何情况下，都应考虑使用 `CUBE`。 `CUBE` 通常最适合在查询中使用来自多个维度的列，而不是表示单个维度的不同级别的列。例如，一个常见的交叉制表可能需要所有月份、州和产品组合的小计。

- **`GROUPING SETS`**

  你可以使用 `GROUP BY` 子句中的 `GROUPING SETS` 表达式有选择地指定你想要创建的组的集合。这允许在没有计算整个 `ROLLUP` 或 `CUBE` 的情况下跨多个维度进行精确规定。例如：

  ```sql
  GROUP BY GROUPING SETS( (a,c), (a,b) )
  ```
    

  如果使用分组扩展子句 `ROLLUP`， `CUBE` 或 `GROUPING SETS`，就会出现两个挑战。首先，如何确定哪些结果行是小计，然后确定给定小计的精确聚合级别。或者，如何区分包含存储的 `NULL` 值和 `ROLLUP` 或 `CUBE` 创建的“NULL”值的结果行。其次，当在 `GROUP BY` 子句中指定了重复的分组集时，如何确定哪些结果行是重复的？你可以在 `SELECT` 列表中使用两个附加的分组函数来帮助解决这个问题：

  - **grouping(column [, ...])** 
    `grouping` 函数可以应用于一个或多个分组属性，以区分超级聚合行和常规分组行。这可以有助于区分表示在超级聚合行中所有值集的“NULL”与常规行中的 `NULL` 值。这个函数中的每个参数都会产生一个位 - 要么是 `1`，要么是 `0`，其中 `1` 表示结果行是超级聚合的， `0` 表示结果行来自常规分组。 `grouping` 函数通过将这些位视为二进制数，然后将其转换为基数-10 的整数来返回一个整数。
  - **group_id()** 
    对于包含重复分组集的分组扩展查询， `group_id` 函数用于在输出中标识重复的行。所有 _唯一_ 的分组集输出行将具有 `<group_id>` 值为 0。对于检测到的每个重复分组集， `group_id` 函数将指定一个大于 0 的 `<group_id>` 数字。所有特定重复分组集中的输出行都由相同的 `<group_id>` 数字标识。

    :::warning 重要
    Extreme DPS 不支持 `group_id`。如查询中包含 `group_id`，请选择 Hybrid PDS 集群执行。  
    :::


 

### WINDOW 子句

`WINDOW` 子句为一个可选择子句，用于指定查询的 `SELECT` 列表或 `ORDER BY` 子句中出现的窗口函数的行为。`WINDOW` 子句的一般形式为：

```sql
WINDOW <window_name> AS (<window_definition>)
```
    

其中 *`<window_name>`* 是一个可以从 `OVER` 子句或后续窗口定义中引用的名称，而 `<window_definition>` 可以是：

```sql
[<existing_window_name>]
[PARTITION BY <expression> [, ...]]
[ORDER BY <expression> [ASC | DESC ] [NULLS {FIRST | LAST}] [, ...] ]
[<frame_clause>] 
```


:::note
*`<window_name>`* 也可以用作 *`<window_definition>`*。但是当用作 *`<window_definition>`* 时，新的窗口定义中，只会继承其中的 `PARTITION BY` 和 `ORDER BY`。新窗口不能更改或覆盖原有的分区或排序方式，它只能在此基础上进行补充。
:::

 

:::warning Extreme DPS 的使用限制
Extreme DPS 不支持 *`<window_name>`*，只能通过 `OVER` 子句中指定窗口定义。
:::

 


`WINDOW` 子句中的条目并非一定需要被引用。不使用时，该条目则会直接被忽略。实际上，不使用 `WINDOW` 子句的情况下也能使用窗口函数，因为窗口函数调用可以直接在 `OVER` 子句中指定窗口定义。当同一个窗口定义需要被多个窗口函数使用时，使用 `WINDOW` 子句减少输入量。

例如：

```sql
SELECT vendor, rank() OVER (mywindow) FROM sale
GROUP BY vendor
WINDOW mywindow AS (ORDER BY sum(prc*qty));
```


- *`<existing_window_name>`*

  如果指定了 *`<existing_window_name>`*，它必须引用 `WINDOW` 列表中的早期条目；新窗口从该条目复制其 `PARTITION BY` 子句，以及 `ORDER BY` 子句（如有）。新窗口不能指定自己的 `PARTITION BY` 子句，并且只有在复制的窗口没有 `ORDER BY` 时，新窗口定义中才能指定 `ORDER BY`。新窗口总是使用自己的帧子句；复制的窗口不能指定帧子句。


- **`PARTITION BY`**

  `PARTITION BY` 子句根据指定表达式的唯一值将结果集组织成逻辑组。`PARTITION BY` 子句的元素的解释方式与 [`GROUP BY` 子句](#group-by-子句) 的元素大致相同，但是存在以下差别：
  
  - `PARTITION BY` 只能是简单表达式，而不是输出列的名称或数字。
  
  - 这些表达式可以包含聚合函数调用，这在常规的 `GROUP BY` 子句中是不允许的。这里允许，是因为窗口化发生在分组和聚合之后。当与窗口函数一起使用时，函数被独立地应用到每一个分区。例如，如果你在 `PARTITION BY` 后面跟随一个列名，结果集将按照该列的不同值进行分区。如果省略，则整个结果集被视为一个分区。


- **`ORDER BY`**

  同样，`ORDER BY` 列表的元素的解读方式与 [`ORDER BY` 子句](#order-by-子句) 的元素大致相同，只是表达式只能是简单表达式，而不是输出列的名称或数字。

  :::note
  `ORDER BY` 子句的元素定义了如何对结果集的每个分区中的行进行排序。如果省略，行将以最高效的顺序返回，并可能有所不同。
  :::


- *`<frame_clause>`*

  可选子句，用于定义依赖于帧的窗口函数（并非所有函数都依赖于帧）定义了 _窗口帧_ 。窗口帧是查询的每一行（称为 _当前行_）的相关行集合。*`<frame_clause>`* 可以是以下之一：

  ```sql
  { RANGE | ROWS | GROUPS } <frame_start> [ <frame_exclusion> ]
  { RANGE | ROWS | GROUPS } BETWEEN <frame_start> AND <frame_end> [ <frame_exclusion> ]
  ```    

  :::warning Extreme DPS 的使用限制
  - Extreme DPS 当前不支持 *`<frame_exclusion>`*。
  - Extreme DPS 当前暂不支持 `GROUPS`。
  :::  

  其中 _`<frame_start>`_ 和 _`<frame_end>`_ 可以是以下之一：

  ```sql
  UNBOUNDED PRECEDING
  <offset> PRECEDING
  CURRENT ROW
  <offset> FOLLOWING
  UNBOUNDED FOLLOWING
  ```

  :::warning Extreme DPS 的使用限制
  当使用的计算引擎为 Extreme DPS 时，只有在帧模式为 `ROWS` 的前提下，_`<frame_start>`_ 或 _`<frame_end>`_ 可以设置为 `<offset> PRECEDING` 或 `<offset> FOLLOWING`。
  :::       

  而 *`<frame_exclusion>`* 可以是以下之一：

  ```sql
  EXCLUDE CURRENT ROW
  EXCLUDE GROUP
  EXCLUDE TIES
  EXCLUDE NO OTHERS
  ```

  如果省略了 *`<frame_end>`*，默认的 `CURRENT ROW` 将被使用。限制是 *`<frame_start>`* 不能是 `UNBOUNDED FOLLOWING`，*`<frame_end>`* 不能是 `UNBOUNDED PRECEDING`，并且 *`<frame_end>`* 的选择不能早于上述列表中的 *`<frame_start>`* 的选择。例如，`RANGE BETWEEN CURRENT ROW AND <offset> PRECEDING` 是不允许的。

  默认的帧选项是 `RANGE UNBOUNDED PRECEDING`，效果与 `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` 相同。它将帧设置为从分区开始到当前行的最后一个对等行（窗口的 `ORDER BY` 子句认为等同于当前行的行；如果没有 `ORDER BY`，所有行都是对等的）。一般来说，`UNBOUNDED PRECEDING` 意味着帧从分区的第一行开始，同样，`UNBOUNDED FOLLOWING` 意味着帧在分区的最后一行结束，无论 `RANGE`、`ROWS` 或 `GROUPS` 模式如何。在 `ROWS` 模式下，`CURRENT ROW` 意味着帧开始或结束于当前行；但在 `RANGE` 或 `GROUPS` 模式下，它意味着帧开始或结束于 `ORDER BY` 排序中当前行的第一个或最后一个对等行。`<offset> PRECEDING` 和 `<offset> FOLLOWING` 选项的含义根据帧模式有所不同。在 `ROWS` 模式下，*`<offset>`* 是一个整数，表示帧在当前行之前或之后的多少行开始或结束。在 `GROUPS` 模式下，*`<offset>`* 是一个整数，表示帧在当前行的对等行组之前或之后的多少对等行组开始或结束，其中对等行组是窗口的 `ORDER BY` 子句认为等同的行的组。在 `RANGE` 模式下，使用 *`<offset>`* 选项要求窗口定义中只有一个 `ORDER BY` 列。然后，帧包含那些排序列值不超过 *`<offset>`* 小于（对于 `PRECEDING`）或大于（对于 `FOLLOWING`）当前行排序列值的行。在这些情况下，*`<offset>`* 表达式的数据类型取决于排序列的数据类型。对于数字排序列，它通常与排序列的类型相同，但对于日期时间排序列，它是一个间隔。在所有这些情况下，*`<offset>`* 的值必须是非空且非负的。此外，虽然偏移量不必是一个简单的常数，但它不能包含变量，聚合函数或窗口函数。

  *`<frame_exclusion>`* 选项允许从帧中排除当前行周围的行，即使根据帧开始和帧结束选项，它们也可以被包含。`EXCLUDE CURRENT ROW` 从帧中排除当前行。`EXCLUDE GROUP` 从帧中排除当前行及其排序对等行。`EXCLUDE TIES` 从帧中排除当前行的任何对等行，但不排除当前行本身。`EXCLUDE NO OTHERS` 明确指定不排除当前行或其对等行的默认行为。

  请注意，如果使用 `ORDER BY` 排序时行的顺序并不唯一，那么使用 `ROWS` 模式可能会得到不可预测的结果。而 `RANGE` 和 `GROUPS` 模式设计的目的是为了确保在 `ORDER BY` 排序中处于同一级别的行被相同对待：属于同一组的所有行要么全部包含在窗口中，要么全部被排除。

  使用 `ROWS`，`RANGE` 或 `GROUPS` 子句来表达窗口的边界。窗口边界可以是一个分区的一个，多个或所有行。你可以用数据值的范围偏移从当前行（`RANGE`）表示窗口的边界，或者用当前行（`ROWS`）偏移的行数，或者用对等组的数量（`GROUPS`）。当使用 `RANGE` 或 `GROUPS` 子句时，你还必须使用 `ORDER BY` 子句。这是因为产生窗口所需的计算需要排序的值。此外，`ORDER BY` 子句不能包含多于一个的表达式，且表达式必须产生日期或数值。当使用 `ROWS`，`RANGE` 或 `GROUPS` 子句时，如果你只指定一个起始行，当前行将被用作窗口中的最后一行。


- **`PRECEDING`** 

  `PRECEDING` 子句使用当前行作为参考点来定义窗口的第一行。起始行是以当前行之前的行数来表达的。例如，在 `ROWS` 框架下，`5 PRECEDING` 将窗口设置从当前行之前的第五行开始。在 `RANGE` 框架下，`5 PRECEDING` 则表示将窗口的起始位置设置为当前排序列值往前数 5 个单位的对等行开始，即指定的顺序是按日期升序，那么窗口将从 5 天内最早的对等行开始。`UNBOUNDED PRECEDING` 将窗口中的第一行设置为分区中的第一行。

- **`BETWEEN`** 

  `BETWEEN` 子句使用当前行作为参考点定义窗口的第一行和最后一行。第一行和最后一行分别以当前行前和后的行数来表示。例如，`BETWEEN 3 PRECEDING AND 5 FOLLOWING` 将窗口设置为从当前行前的第三行开始，并在当前行后的第五行结束。使用 `BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` 将窗口的第一行和最后一行设置为分区的第一行和最后一行。如果没有指定 `ROWs`，`RANGE` 或 `GROUPS` 子句，这相当于默认行为。

- **`FOLLOWING`** 

  `FOLLOWING` 子句使用当前行作为参考点定义窗口的最后一行。最后一行是以当前行之后的行数来表达的。例如，在 `ROWS` 框架下，`5 FOLLOWING` 表示将窗口设置为在当前行后的第五行结束。在 `RANGE` 框架下, `5 FOLLOWING` 表示将窗口的终止位置设置为当前排序列值往后数 5 个单位的对等行开始。如果排序是按日期进行的升序排列，那么窗口将在 5 天后的等对行结束。使用 `UNBOUNDED FOLLOWING` 将窗口的最后一行设置为分区的最后一行。

如果你没有指定 `ROWS`、`RANGE` 或 `GROUPS` 子句，当使用 `ORDER BY` 时，窗口范围将从分区中的第一行（`UNBOUNDED PRECEDING`）开始，并以当前行（`CURRENT ROW`）结束。如果没有指定 `ORDER BY`，窗口将从分区中的第一行（`UNBOUNDED PRECEDING`）开始，并以分区中的最后一行（`UNBOUNDED FOLLOWING`）结束。

`WINDOW` 子句用于指定查询的 [`SELECT` 列表](#select-列表) 或 [`ORDER BY` 子句](#order-by-子句) 中出现的窗口函数的行为。这些函数可以在它们的 `OVER` 子句中通过名称引用 `WINDOW` 子句条目。然而，`WINDOW` 子句条目并不一定要在任何地方被引用；如果它在查询中没有被使用，就会被简单地忽略。完全可以不使用任何 `WINDOW` 子句而使用窗口函数，因为窗口函数调用可以直接在其 `OVER` 子句中指定其窗口定义。然而，当同一个窗口定义需要用于多个窗口函数时，使用 `WINDOW` 子句可以节省输入。

当前，`FOR NO KEY UPDATE`、`FOR UPDATE`、`FOR SHARE` 和 `FOR KEY SHARE` 不能与 `WINDOW` 一起指定。


 

### HAVING 子句

可选的 `HAVING` 子句的一般形式为：

```sql
HAVING <condition>
```

在这里，*condition* 的定义与 `WHERE` 子句中的定义相同。`HAVING` 会排除不满足条件的分组行。`HAVING` 与 `WHERE` 的不同之处在于：`WHERE` 是在应用 `GROUP BY` 之前过滤单独的行，而 `HAVING` 是过滤由 `GROUP BY` 创建的分组行。在 *condition* 中引用的每一列都必须明确地引用一个分组列，除非该引用出现在一个聚合函数中，或者未分组的列在功能上依赖于分组列。

`HAVING` 的存在会将一个查询转变为一个分组查询，即使没有 `GROUP BY` 子句。这与查询中包含聚合函数但没有 `GROUP BY` 子句时的情况相同。所有被选中的行都被认为形成了一个单独的分组，`SELECT` 列表和 `HAVING` 子句只能引用来自聚合函数内部的表列。如果 `HAVING` 条件为真，这样的查询将发出一行；如果 `HAVING` 条件不为真，这样的查询将发出零行。


 

### UNION 子句

`UNION` 子句的一般形式为：

```sql
<select_statement> UNION [ALL | DISTINCT] <select_statement>
```

在这里，*`<select_statement>`* 是任何没有 `ORDER BY`、`LIMIT`、`FOR NO KEY UPDATE`、`FOR UPDATE`、`FOR SHARE` 或 `FOR KEY SHARE` 子句的 `SELECT` 语句。`ORDER BY` 和 `LIMIT` 可以附加到子查询表达式上，如果它被括在括号中。没有括号，这些子句将被认为应用于 `UNION` 的结果，而不是其右侧的输入表达式。

`UNION` 运算符计算由涉及的 `SELECT` 语句返回的行的集合并集。如果一行至少在一个结果集中出现，那么它就在这两个结果集的集合并集中。表示 `UNION` 的直接操作数的两个 `SELECT` 语句必须产生相同数量的列，对应的列必须是兼容的数据类型。

`UNION` 的结果不包含任何重复的行，除非指定了 `ALL` 选项。`ALL` 阻止了重复的消除。因此，`UNION ALL` 通常比 `UNION` 更快。当你可以的时候，使用 `ALL`。可以编写 `DISTINCT` 来明确指定消除重复行的默认行为。

同一个 `SELECT` 语句中的多个 `UNION` 运算符从左到右进行求值，除非括号指示其他顺序。



 

### INTERSECT 子句

`INTERSECT` 子句的一般形式为：

```sql
<select_statement> INTERSECT [ALL | DISTINCT] <select_statement>
```

在这里，*`<select_statement>`* 是任何没有 `ORDER BY` 或 `LIMIT` 子句的 `SELECT` 语句。

`INTERSECT` 运算符计算由涉及的 `SELECT` 语句返回的行的集合交集。如果一行在两个结果集中都出现，那么它就在这两个结果集的交集中。

`INTERSECT` 的结果不包含任何重复的行，除非指定了 `ALL` 选项。对于 `ALL`，左表中有 m 个重复项，右表中有 n 个重复项的行将在结果集中出现 min(m, n) 次。可以编写 `DISTINCT` 来明确指定消除重复行的默认行为。

除非括号规定其他顺序，否则同一个 `SELECT` 语句中的多个 `INTERSECT` 运算符从左到右进行求值。`INTERSECT` 的绑定比 `UNION` 更紧密。也就是说，`A UNION B INTERSECT C` 会被读取为 `A UNION (B INTERSECT C)`。


 


### EXCEPT 子句

`EXCEPT` 子句的一般形式为：

```sql
<select_statement> EXCEPT [ALL | DISTINCT] <select_statement>
```

在这里，*`<select_statement>`* 是任何没有 `ORDER BY` 或 `LIMIT` 子句的 `SELECT` 语句。

`EXCEPT` 运算符计算左 `SELECT` 语句的结果中，但不在右 `SELECT` 语句结果中的行集。

`EXCEPT` 的结果不包含任何重复的行，除非指定了 `ALL` 选项。对于 `ALL`，左表中有 m 个重复项，右表中有 n 个重复项的行在结果集中将出现 max(m-n,0) 次。可以编写 `DISTINCT` 来明确指定消除重复行的默认行为。

同一个 `SELECT` 语句中的多个 `EXCEPT` 运算符从左到右进行求值，除非括号规定其他顺序。`EXCEPT` 的绑定在同一级别上 `UNION`。


 

### ORDER BY 子句

可选的 `ORDER BY` 子句的一般形式为：

```sql
ORDER BY <expression> [ASC | DESC ] [NULLS {FIRST | LAST}] [, ...]
```

在这里，*`<expression>`* 可以是输出列（`SELECT` 列表项）的名称或序数，或者可以是由输入列值形成的任意表达式。

`ORDER BY` 子句会根据指定的表达式对结果行进行排序。如果两行根据最左边的表达式相等，那么它们会根据下一个表达式进行比较，依此类推。如果它们根据所有指定的表达式都相等，那么它们将按照实现依赖的顺序返回。

序数指的是输出列的序数（从左到右）位置。这个特性使得在没有唯一名称的列的基础上定义排序成为可能。这绝对不是必要的，因为总是可以使用 `AS` 子句为输出列分配名称。

在 `ORDER BY` 子句中也可以使用任意表达式，包括不在 `SELECT` 输出列表中出现的列。因此，以下语句是有效的：

```sql
SELECT name FROM distributors ORDER BY code;
```

这个特性的一个限制是，应用于 `UNION`、`INTERSECT` 或 `EXCEPT` 子句的结果的 `ORDER BY` 子句只能指定输出列的名称或数字，不能是表达式。

如果 `ORDER BY` 表达式是一个简单的名称，既匹配输出列的名称，也匹配输入列的名称，那么 `ORDER BY` 将把它解释为输出列的名称。这与 `GROUP BY` 在同一情况下会做出的选择恰恰相反。这种不一致是为了与 SQL 标准兼容。

可以在 `ORDER BY` 子句的任何表达式后面添加关键字 `ASC`（升序）或 `DESC`（降序）。如果没有指定，那么默认为 `ASC`。

如果指定了 `NULLS LAST`，空值将在所有非空值之后排序；如果指定了 `NULLS FIRST`，空值将在所有非空值之前排序。如果两者都没有指定，那么默认行为是当指定或隐含 `ASC` 时 `NULLS LAST`，当指定 `DESC` 时 `NULLS FIRST`（因此，默认行为是将 null 视为大于非 null）。

请注意，排序选项只适用于它们后面的表达式。例如，`ORDER BY x, y DESC` 并不意味着与 `ORDER BY x DESC, y DESC` 相同。

字符串数据按照数据库创建时确定的特定于区域的排序顺序进行排序。

字符串数据根据应用于被排序列的排序进行排序。如果需要，可以通过在表达式中包含 `COLLATE` 子句来覆盖排序，例如 `ORDER BY mycolumn COLLATE "en_US"`。


 

### LIMIT 子句

`LIMIT` 子句由两个独立的子句组成：

```sql
LIMIT {<count> | ALL}
OFFSET <start>
```

在这里，*`<count>`* 指定要返回的最大行数，而 *`<start>`* 指定在开始返回行之前要跳过的行数。当两者都指定时，会先跳过 *`<start>`* 行，然后开始计数要返回的 *`<count>`* 行。

如果 *`<count>`* 表达式的值为 NULL，它会被视为 `LIMIT ALL`，即无限制。如果 *`<start>`* 的值为 NULL，它会被视同 `OFFSET 0`。

SQL\:2008 引入了一种不同的语法来实现同样的结果，MAXIR 也支持这种语法。它是：

```sql
OFFSET <start> [ ROW | ROWS ]
            FETCH { FIRST | NEXT } [ <count> ] { ROW | ROWS } ONLY
```

在这种语法中，标准要求 *`<start>`* 或 *`<count>`* 的值是一个文字常量、参数或变量名；作为 MAXIR 扩展，其他表达式也是允许的，但通常需要用括号括起来以避免歧义。如果在 `FETCH` 子句中省略了 *`<count>`*，则默认为 1。`ROW` 和 `ROWS` 以及 `FIRST` 和 `NEXT` 是不影响这些子句效果的噪音词。根据标准，如果两者都存在，`OFFSET` 子句必须在 `FETCH` 子句之前；但 MAXIR 允许任何顺序。

使用 `LIMIT` 时，建议使用一个 `ORDER BY` 子句来将结果行限制为一个唯一的顺序。否则，你将获得查询的行的一个不可预测的子集——你可能在请求第十行到第二十行，但是在什么顺序的第十行到第二十行呢？除非你指定 `ORDER BY`，否则你不知道什么顺序。

查询优化器在生成查询计划时会考虑 `LIMIT`，所以你很可能会得到不同的计划（产生不同的行顺序），这取决于你使用什么作为 `LIMIT` 和 `OFFSET`。因此，使用不同的 `LIMIT/OFFSET` 值来选择查询结果的不同子集将给出不一致的结果，除非你用 `ORDER BY` 来强制一个可预测的结果排序。这不是一个缺陷；这是一个固有的结果，因为 SQL 不承诺在任何特定的顺序下提供查询的结果，除非使用 `ORDER BY` 来约束顺序。


---

## TABLE 命令

命令 `TABLE <name>` 完全等同于 `SELECT * FROM <name>`。

它可以用作最外层的命令，或者用作复杂查询部分的节省空间的语法变体。



---

示例
----------

将表 `films` 与表 `distributors` 进行连接：

```sql
SELECT f.title, f.did, d.name, f.date_prod, f.kind FROM 
distributors d, films f WHERE f.did = d.did
```

对所有电影的 `length` 列进行求和，并根据 `kind` 对结果进行分组：

```sql
SELECT kind, sum(length) AS total FROM films GROUP BY kind;
```

对所有电影的 `length` 列进行求和，根据 `kind` 对结果进行分组，并显示那些总计小于 5 小时的组总计：

```sql
SELECT kind
	,sum(length) AS total
FROM films
GROUP BY kind
HAVING sum(length) < interval '5 hours';
```

计算电影 `kind` 和 `distributor` 的所有销售额的小计和总计。

```sql
SELECT kind
	,distributor
	,sum(prc * qty)
FROM sales
GROUP BY ROLLUP(kind, distributor)
ORDER BY 1
	,2
	,3;
```

根据总销售额计算电影发行商的排名：

```sql
SELECT distributor, sum(prc*qty), 
          rank() OVER (ORDER BY sum(prc*qty) DESC) 
FROM sale
GROUP BY distributor ORDER BY 2 DESC;
```
    
以下两个例子是根据第二列（`name`）的内容对个别结果进行排序的相同方式：

```sql
SELECT * FROM distributors ORDER BY name;
SELECT * FROM distributors ORDER BY 2;
```

下一个例子展示了如何获取表 `distributors` 和 `actors` 的并集，将结果限制为在每个表中以字母 `W` 开头的那些。只需要唯一的行，所以省略了关键词 `ALL`：

```sql
SELECT distributors.name FROM distributors WHERE 
distributors.name LIKE 'W%' UNION SELECT actors.name FROM 
actors WHERE actors.name LIKE 'W%';
```

这个示例展示了如何在 `FROM` 子句中使用函数，包括使用和不使用列定义列表：

```sql
CREATE FUNCTION distributors(int) RETURNS SETOF distributors 
AS $$ SELECT * FROM distributors WHERE did = $1; $$ LANGUAGE 
SQL;
SELECT * FROM distributors(111);
    
CREATE FUNCTION distributors_2(int) RETURNS SETOF record AS 
$$ SELECT * FROM distributors WHERE did = $1; $$ LANGUAGE 
SQL;
SELECT * FROM distributors_2(111) AS (dist_id int, dist_name 
text);
```
    

这个示例使用了一个简单的 `WITH` 子句：

```sql
WITH test AS (
    SELECT random() as x FROM generate_series(1, 3)
    )
SELECT * FROM test
UNION ALL
SELECT * FROM test; 
```
    

这个示例使用 `WITH` 子句仅显示销售额最高地区的每种产品的销售总额。

```sql
WITH regional_sales AS 
    SELECT region, SUM(amount) AS total_sales
    FROM orders
    GROUP BY region
    ), top_regions AS (
    SELECT region
    FROM regional_sales
    WHERE total_sales > (SELECT SUM(total_sales) FROM
        regional_sales)
    )
SELECT region, product, SUM(quantity) AS product_units,
    SUM(amount) AS product_sales
FROM orders
WHERE region IN (SELECT region FROM top_regions) 
GROUP BY region, product;
```
    

这个例子可以不使用 `WITH` 子句来编写，但需要两级嵌套的子 `SELECT` 语句。

这个示例使用 `WITH RECURSIVE` 子句从仅显示直接下属的表中找到员工 Mary 的所有下属（直接或间接）及其间接层次：

```sql
WITH RECURSIVE employee_recursive(distance, employee_name, manager_name) AS (
    SELECT 1, employee_name, manager_name
    FROM employee
    WHERE manager_name = 'Mary'
    UNION ALL
    SELECT er.distance + 1, e.employee_name, e.manager_name
    FROM employee_recursive er, employee e
    WHERE er.employee_name = e.manager_name
    )
SELECT distance, employee_name FROM employee_recursive;
```    

递归查询的典型形式：一个初始条件，后面跟着 `UNION [ALL]`，然后跟着查询的递归部分。确保查询的递归部分最终不会返回任何元组，否则查询将无限循环。


---

SQL 标准兼容性
--------

`SELECT` 语句与 SQL 标准兼容，但有一些扩展和一些缺失的功能。

**省略 FROM 子句**

MAXIR 允许省略 `FROM` 子句。它可以直接用来计算简单表达式的结果。例如：

```sql
SELECT 2+2;
```
    

一些其他的 SQL 数据库除非引入一个虚拟的单行表来进行 `SELECT`，否则无法做到这一点。

请注意，如果没有指定 `FROM` 子句，查询无法引用任何数据库表。例如，以下查询是无效的：

```sql
SELECT distributors.* WHERE distributors.name = 'Westward';
```

**省略 AS 关键字**

在 SQL 标准中，只要新的列名是有效的列名（即，不与任何保留关键字相同），就可以在输出列名前省略可选的关键字 `AS`。MAXIR 稍微严格一些：如果新的列名与任何关键字（无论是否保留）匹配，都需要 `AS`。建议的做法是使用 `AS` 或双引号输出列名，以防止与未来关键字添加产生任何可能的冲突。

在 `FROM` 项中，标准和 MAXIR 都允许在别名前省略 `AS`，只要该别名是未保留的关键字。但对于输出列名来说，这是不切实际的，因为存在语法歧义。

**ONLY 和继承**

在编写 `ONLY` 时，SQL 标准要求在表名周围添加括号，例如：

```sql
SELECT * FROM ONLY (tab1), ONLY (tab2) WHERE ...
```
    

MAXIR 认为这些括号是可选的。

MAXIR 允许在末尾写一个 `*`，以明确指定包含子表的非 `ONLY` 行为。标准不允许这样做。

:::note
以上各点同样适用于所有支持 `ONLY` 选项的 SQL 命令。
:::

**GROUP BY 和 ORDER BY 可用的命名空间**

在 SQL-92 标准中，`ORDER BY` 子句只能使用输出列名或数字，而 `GROUP BY` 子句只能使用基于输入列名的表达式。MAXIR 扩展了这些子句，允许其他选择（但如果存在歧义，它会使用标准的解释）。MAXIR 也允许这两个子句指定任意表达式。请注意，出现在表达式中的名字始终被视为输入列名，而不是输出列名。

SQL\:1999 及后续版本使用了与 SQL-92 不完全向上兼容的略有不同的定义。然而，在大多数情况下，MAXIR 对 `ORDER BY` 或 `GROUP BY` 表达式的解释与 SQL\:1999 的解释相同。

**函数依赖**

当表的主键包含在 `GROUP BY` 列表中时，MAXIR 才会识别函数依赖（允许从 `GROUP BY` 中省略列）。SQL 标准指定了必须识别的其他条件。

**LIMIT 和 OFFSET**

`LIMIT` 和 `OFFSET` 子句是 MAXIR 特有的语法，MySQL 也使用。SQL\:2008 标准引入了 `OFFSET .. FETCH {FIRST|NEXT} ...` 子句来实现相同的功能，如上所示。这种语法也被 IBM DB2 使用。（Oracle 的应用程序经常使用一个涉及自动生成的 `rownum` 列的解决方案，这在 MAXIR 中不可用，来实现这些子句的效果。）

**在 WITH 中修改数据的语句**

MAXIR 允许 `INSERT`，`UPDATE` 和 `DELETE` 用作 `WITH` 查询。这不受 SQL 标准的支持。

**非标准子句**

`DISTINCT ON` 子句未在 SQL 标准中定义。
