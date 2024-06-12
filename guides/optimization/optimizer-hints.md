# 优化器提示

本文介绍了 Relyt 支持的优化器提示及使用方法。

## 概述

Relyt 自身提供了一个强大的查询优化器，用于确定 SQL 命令执行的最有效方式。但是也存在优化器选择的查询计划不是最高效的执行方式的情况。在这种情况下，可以使用优化器提示来指导优化器执行特定操作。

优化器提示是一种特殊的指示，可以嵌入到 SQL 命令中，以指导查询优化器如何执行查询。下表提供了 Relyt 提供的优化器提示的一般信息。

| 名称 | 描述 | 适用范围 |
| :- | :- | :- |
| [SET](#set-提示) | 临时设置系统变量的会话值。 | 查询 |
| [JOIN_ORDER](#join_order-提示) | 指定查询块中的连接顺序。 | 查询块 |
| [REPLICATED, PARTITIONED](#连接分布类型提示) | 设置连接的分布类型。 | 查询块 |

<br/>

作为一种特殊的注释类型，优化器提示必须在 `/*+ ... */` 中指定。允许空格。例如：

```sql
/*+SET (<var_name>, '<value>')[,...]*/
```


## SET 提示

`SET` 提示临时为查询指定系统变量的会话值。
### 语法

`SET` 提示的语法是：

```sql
SET (<var_name>, '<value>')
```

在这个语法中，
- *`<var_name>`*：指定系统变量的名称。

- *`<value>`*：指定此系统变量的新会话值。
  

一个 `SET` 提示只能用于设置一个每会话系统变量。如果你想设置多个，可以在一个提示块中（即，一个 `/*+ ... */` 注释）以逗号分隔的列表中指定多个 `SET` 提示。以下是两个例子：

```sql
/*+SET(statistics_cpu_timer_enabled,'true'), SET(query_max_memory, '1GB' )*/
SELECT * FROM sales
```


### 使用注意事项

目前，`SET` 提示必须放在查询的最开始。

如果一个提示块包含重复的 `SET` 提示，只有最后一个会生效。

如果一个提示块包含冲突的 `SET` 提示，只有最后一个会生效。

一个查询最多可以强制执行一个 `SET` 提示块。如果你在一个查询中指定了多个提示块，只有第一个提示块会生效。例如：

```sql
/*+SET(statistics_cpu_timer_enabled,'true')*/
/*+SET(task_concurrency, '1')*/
SELECT * FROM sales
```

在这个例子中，只有 `SET(statistics_cpu_timer_enabled,'true')` 会生效。 `SET (task_concurrency, '1' )` 将被忽略。

### 示例

将会话变量 `preferred_tablescan_read_batch_rows` 设置为 `40480`：

```sql
/*+SET(preferred_tablescan_read_batch_rows,'40480')*/
SELECT 123.456E7 from DUAL
```



## JOIN_ORDER 提示

`JOIN_ORDER` 提示影响 Relyt 优化器连接表的顺序，其范围是查询块。

### 语法

```sql
JOIN_ORDER (<left_t>, <right_t>);
```

- *`left_t`*：连接的左边的表或列表。

- *`right_t`*：连接的右边的表或列表。

 如果 *`<left_t>`* 或 *`<right_t>`* 是一组表，你必须使用括号包围它们。如果列表中包含两个以上的表，你必须使用多级括号，以便在每对括号中只有两个元素（一个表或一对括号中的内容被视为一个元素）。最内层括号中的表或元素将首先被连接。

例如：

```sql
JOIN_ORDER (a, (b, c))
```
在这个例子中，*`<left_t>`* 是 `a`，*`<right_t>`* 是 `(b, c)`。这个提示指定 `a` 与 `b` 和 `c` 的连接结果连接。

再举一个例子：

```sql
JOIN_ORDER ((a, (b, c)),(d, e))
```

在这个例子中，*`<left_t>`* 是 `(a, (b, c))`，*`<right_t>`* 是 `(d, e)`。这个提示指定 `a` 与 `b` 和 `c` 的连接结果在左边，而 `d` 与 `e` 在右边，然后左边的结果与右边的结果连接。

然而，`JOIN_ORDER (a, (b, c), d)` 是无效的，因为它包含三个元素。

### 使用注意事项

如果存在以下任何条件，`JOIN_ORDER` 提示将被忽略：

- 任何指定表的名称或别名不存在。

- 它包含重复的表名或别名。

- 它没有包含查询块中指定的所有表。

   假设查询块的当前连接顺序是 `a INNER JOIN b INNER JOIN c`。如果你为查询块设置了 `/*+ JOIN_ORDER (a, b) */`，提示将被忽略。在这个例子中，正确的 `JOIN_ORDER` 提示是：`/*+ JOIN_ORDER (c, (a, b)) */`。

如果一个表有一个别名，`JOIN_ORDER` 提示必须引用这个别名。

每次只能对每个查询块执行一个 `JOIN_ORDER` 提示。如果你在一个查询块中指定了多个 `JOIN_ORDER` 提示，只有第一个 `JOIN_ORDER` 会生效。

### 示例

使用 `JOIN_ORDER` 提示首先连接表 `orders` 和 `customer`，然后与表 `lineitem` 连接：

```sql
SELECT 
/*+JOIN_ORDER(lineitem,(orders,customer))*/
  l_orderkey, 
  sum(
    l_extendedprice * (1 - l_discount)
  ) AS revenue, 
  o_orderdate, 
  o_shippriority 
FROM 
  customer, 
  orders, 
  lineitem 
WHERE 
  c_mktsegment = 'BUILDING' 
  AND c_custkey = o_custkey 
  AND l_orderkey = o_orderkey 
  AND o_orderdate < date '1995-03-15' 
  AND l_shipdate > date '1995-03-15' 
GROUP BY 
  l_orderkey, 
  o_orderdate, 
  o_shippriority 
ORDER BY 
  revenue DESC, 
  o_orderdate 
LIMIT 
  10;
```

## 连接分布类型提示

Relyt 提供了两个用于配置连接分布类型的提示：`REPLICATED` 和 `PARTITIONED`。 `REPLICATED` 提示将连接的分布类型设置为 `REPLICATED`。 `PARTITIONED` 提示将连接的分布类型设置为 `PARTITIONED`。 


### 语法

```sql
/*+ [REPLICATED | PARTITIONED] (<join_order>)[, ... ] */
```

在这个语法中，

- `REPLICATED`：将指定连接的分布类型设置为 `REPLICATED`。

- `PARTITIONED`：将指定连接的分布类型设置为 `PARTITIONED`。

- *`<join_order>`*：指定分布类型适用的连接顺序。
  
  有关连接顺序的使用注意事项的详细信息，请参见 `JOIN_ORDER` 部分中描述的 [使用注意事项](#使用注意事项)。

在一个提示块中，你可以指定多个分布类型提示。例如：

```sql
/*+ REPLICATED (a, (b, c)), PARTITIONED (b, c) */
```

### 使用注意事项

如果一个提示块中指定的两个提示互相冲突，只有第一个提示会生效。例如：

```sql
/*+ REPLICATED (a, b), PARTITIONED (a, b) */
```
在这个例子中，只有第一个提示 `REPLICATED (a, b)` 会生效。 

在某些情况下，连接的分布类型是确定的，不能通过这样的提示进行修改。例如，`a CROSS JOIN b` 的分布类型只能是 `REPLICATED`。在这个例子中，设置 `PARTITIONED (a, b)` 提示没有效果。

如果指定的连接顺序不存在，提示是无效的。


### 示例


将 `orders JOIN customer` 的分布类型设置为 `PARTITIONED`，将 `lineitem JOIN (orders JOIN customer)` 的分布类型设置为 `REPLICATED`：

```sql
SELECT
/*+REPLICATED(l,(o,c)), PARTITIONED(o,c)*/
    l.orderkey,
    sum(l.extendedprice * (1 - l.discount)) AS revenue,
    o.orderdate,
    o.shippriority
FROM customer AS c,
     orders AS o,
     lineitem AS l
WHERE c.mktsegment = 'BUILDING'
  AND c.custkey = o.custkey
  AND l.orderkey = o.orderkey
  AND o.orderdate < DATE '1995-03-15'
  AND l.shipdate > DATE '1995-03-15'
GROUP BY l.orderkey,
         o.orderdate,
         o.shippriority
ORDER BY revenue DESC,
         o.orderdate LIMIT 10
;
```