
# 条件表达式

Extreme DPS 中提供了几种符合 SQL 的条件表达式，即 `CASE WHEN`、`COALESCE`、`NULLIF`，以及 `GREATEST` 和 `LEAST`。

:::info 说明
- Extreme DPS 不支持在条件表达式中使用 `interval`。
- 即使语法与函数类似，`COALESCE`、`GREATEST` 和 `LEAST` 不能与明确的 `VARIADIC` 数组参数一起使用。
:::

---

## CASE WHEN

`CASE WHEN` 表达式是通用的条件表达式，类似于其他编程语言中的 if/else 语句。

### 语法

```sql
CASE WHEN <condition1> THEN <result1>
     [WHEN <condition2> THEN <result2>]
     [...]
     [ELSE <result3>]
END
```

`CASE` 子句可以在表达式中任何有效地方使用。如果条件为真，将返回结果 *`<result1>`*，并且不会处理 `CASE` 表达式的其余部分。

### 参数

- *`<conditionN>`*

    计算为布尔值（`true`、`false` 或 `NULL`）的条件。

- *`<resultN>`*

    如果 *`<conditionN>`* 为真，则返回的结果。

### 示例

```sql
SELECT * FROM data;

 id
---
 123
 456
 789


SELECT id,
       CASE WHEN id=123 THEN 'john'
            WHEN id=456 THEN 'tom'
            ELSE 'others'
       END
    FROM data;

  id | case
-----+-------
 123 | john
 456 | tom
 789 | others
```

### 使用注意事项

所有 *`<resultN>`* 表达式的数据类型必须可转换为单一的输出类型。更多信息，请查阅 PostgreSQL 文档中的 [UNION, CASE, and Related Constructs](https://www.postgresql.org/docs/12/typeconv-union-case.html)。

--- 

## COALESCE

`COALESCE` 函数返回第一个非 null 表达式，或者如果所有参数都是 `NULL`，则返回 `NULL`。`COALESCE` 的返回类型不固定，由返回表达式的数据类型决定。

### 语法

```sql
COALESCE(<value> [, ...])
```

### 参数

*`<value>`*：验证是否为 null 的参数。

### 示例

```sql
SELECT * FROM data;

 name
---
 john
 tom
 NULL


SELECT COALESCE(null, name) as r_
    FROM data;

  r_ 
-----
 john 
 tom  
 null 
```

### 使用注意事项

你可以使用 `COALESCE` 在检索数据以供显示时替换 null 值的默认值。以下提供了一个示例语法：

```sql
SELECT COALESCE(<description>, <short_description>, '(none)') ...
```

在此语法中：

- 如果 *`<description>`* 不为 null，将返回 *`<description>`*。

- 如果 *`<description>`* 为 null，并且 *`<short_description>`* 不为 null，将返回 *`<short_description>`*。

- 在其他情况下，将返回 `NULL`。

---

## NULLIF

当第一个表达式等于第二个表达式时，`NULLIF` 函数返回 `NULL`。否则，它返回第一个表达式。

### 语法

```sql
NULLIF(value1, value2)
```
 
### 参数

- *`<value1>`*

  要比较的第一个表达式。

- *`<value2>`*

  评估为与 *`<value1>`* 相同数据类型的第二个表达式。

### 使用注意事项

这两个表达式必须是可比较的类型。这意味着存在一个等式 `<value1>=<value2>`，因此必须有一个合适的等号（`=`）运算符可用。在大多数情况下，返回类型与 *`<value1>`* 的数据类型相同。但是在一些特殊情况下，返回类型与 *`<value2>`* 相同。例如，`NULLIF(1, 2.2)` 的返回类型为 `numeric`。这是因为 `=` 运算符不能用于比较 `integer` 和 `numeric`，只能用于比较 `numeric` 和 `numeric`。

你还可以使用 `NULLIF` 来执行 `COALESCE` 的逆操作。如下为语法示例：

```sql
SELECT NULLIF(<value>, '(none)') ...
```

在此语法中，如果 *`<value>`* 是 `(none)`，则将返回 null。否则，将返回 *`<value>`* 的值。

---

## GREATEST 和 LEAST

`GREATEST` 或 `LEAST` 函数从一系列表达式中返回最大或最小的值。

### 语法

```sql
GREATEST(<value> [, ...])
```

```sql
LEAST(<value> [, ...])
```

### 参数

*`<value>`*：要比较的表达式。

### 示例

`GREATEST` 示例：

```sql
SELECT * FROM data;

col_1 | col_2
------+-------
 110  | 100
 99   | 130
 199  | 300
 NULL | 999


SELECT GREATEST(col_1, col_2, 128) as r_
    FROM data;

  r_ 
-----
 128 
 130  
 300
 NULL 
```

`LEAST` 示例：

```sql
SELECT * FROM data;

col_1 | col_2
------+-------
 110  | 100
 99   | 130
 199  | 300
 NULL | 999


SELECT LEAST(col_1, col_2, 128) as r_
    FROM data;

  r_ 
-----
 100 
 99  
 128 
 NULL
```

### 使用注意事项

列表中的所有值必须可以转换为常见的数据类型。更多信息，请参阅 PostgreSQL 文档中的 [UNION, CASE, and Related Constructs](https://www.postgresql.org/docs/12/typeconv-union-case.html)。

列表中的 `NULL` 值将被忽略。如果表达式列表中的所有值都是 `NULL`，则会返回 `NULL`。


---

## TRY

对输入表达式进行评估，并通过返回 NULL 来评估处理某些类型的错误。

`TRY` 函数非常适合用于处理如下类型的查询：在遇到数据损坏或无效数据时，返回 NULL 或默认值的时候而不是直接返回失败。为了指定默认值，`TRY` 函数可以与 `COALESCE` 函数一起使用。


`TRY` 函数能够处理以下错误：

- 除零

- 无效的转换参数或无效的函数参数

- 数值超出范围

### 语法

```sql
TRY(<expression>)
```

### 参数

*`<expression>`*：查询的 SQL 表达式。

### 返回

NULL 或者指定的默认值。

### 示例

如下示例中的表 `orders` 包含无效数据：

```sql
SELECT * FROM orders;
 order_id | customer_id | order_total
----------+-------------+-------------
        1 |        1001 |         250
        2 |        1002 |         120
        3 |        1003 |         180
        4 |        1004 |       error
```

通过 `TRY` 函数来处理错误：

```sql
-- 查询失败时不使用 TRY：
SELECT 100 / order_total AS order_ratio FROM orders;
 order_ratio
-------------
        0
   0
   0
   (null)


-- 使用 TRY 和 COALESCE 处理默认值：
SELECT COALESCE(TRY(100 / order_total), 0) AS order_ratio FROM orders;
 order_ratio
-------------
          0
         0
          0
         0

```