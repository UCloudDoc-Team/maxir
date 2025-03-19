EXPLAIN
=====

显示语句的查询计划。


---

语法
--------

```sql
EXPLAIN [ ( <option> [, ...] ) ] <statement>
EXPLAIN [ANALYZE] [VERBOSE] <statement>
其中 <option> 可以为：
    ANALYZE [ <boolean> ]
    VERBOSE [ <boolean> ]
    COSTS [ <boolean> ]
    BUFFERS [ <boolean> ]
    TIMING [ <boolean> ]
    FORMAT { TEXT | XML | JSON | YAML }
```

---

描述
--------

语句的查询计划显示为节点的树形计划。每个节点表示一个操作，例如联接。

从底部向上阅读查询计划，因为下面的节点会向其上面的节点提供行。查询计划通常从表扫描开始。扫描上方的节点处理如排序、聚合和联接等操作（如果必要）。顶部节点通常是在查询执行过程中在实例间移动行的节点，如重新分配、广播和收集操作。

`EXPLAIN` 的输出中的每一行提供了基本节点类型和 Planner 为一个节点估计的以下成本：

- `cost`：估计的执行时间，通常以磁盘页取回来衡量。显示两个值：启动成本和总成本。启动成本表示可以返回第一行之前的时间，总成本表示可以返回最后一行的时间。

    在估计总成本时， Planner 假设所有行都被检索。这在某些情况下可能导致估计成本不准确，例如，使用 `LIMIT`。

- `rows`：节点输出的行数估计值。通常少于实际数量。在理想情况下，顶级节点的估计数字估计了查询实际返回、更新或删除的行数。

- `width`：节点输出的所有行的字节数。

上级节点的估计成本包含其子节点的成本。因此，最顶层节点的估计成本就是整个查询的成本。

查询的估计成本会随优化器而变化。在 MAXIR 中，这个成本不包括将结果行传输到客户端的时间。

`EXPLAIN ANALYZE` 不仅解释查询，还运行它。因此，如果你运行 `EXPLAIN ANALYZE`，指定语句的估计值和实际成本都会显示。你可以将估计值与实际成本进行比较，以检查估计值是否准确。除了 `EXPLAIN` 返回的信息外，`EXPLAIN ANALYZE` 的输出还包括：

- 运行查询所花费的总时间（以毫秒为单位）。

- 参与计划节点操作的 Worker 数量。只有返回行的 Worker 才会被计数。

- 生产操作最多行的 Worker 返回的最大行数。如果多个 Worker 生产相同数量的行，则选择结束时间最长的那一个。

- 生产操作最多行的 Worker 的 Worker ID 数。

- 对于相关操作，操作使用的 *`<work_mem>`*。如果 *`<work_mem>`* 不足以在内存中执行操作，计划将显示有多少数据溢出到磁盘，以及最低性能的 Worker 需要对数据进行多少次传递。例如：

    ```sql
    Work_mem used: 64K bytes avg, 64K bytes max (seg0).
    Work_mem wanted: 90K bytes avg, 90K bytes max (seg0) to abate workfile 
    I/O affecting 2 workers.
    [seg0] pass 0: 488 groups made from 488 rows; 263 rows written to 
    workfile
    [seg0] pass 1: 263 groups made from 263 rows
    ```

请注意，`ANALYZE` 会运行语句。尽管 `EXPLAIN ANALYZE` 会丢弃 `SELECT` 返回的任何输出，但语句的其他副作用会像往常一样发生。如果你想在不让命令影响你的数据的情况下使用 `EXPLAIN ANALYZE` 对 DML 语句进行操作，使用以下方法：

```sql
BEGIN;
EXPLAIN ANALYZE ...;
ROLLBACK;
```

只有 `ANALYZE` 和 `VERBOSE` 选项可以被指定，且只能按此顺序，无需在选项列表周围加括号。



---

参数
----------

- **`ANALYZE`**

    执行命令并显示实际运行时间和其他统计信息。如果省略，默认使用 `false`。你可以指定 `ANALYZE true` 来启用它。

- **`VERBOSE`**

    显示有关计划的额外信息。具体来说，包括计划树中每个节点的输出列列表，Schema 限定的表和函数名称，始终用其范围表别名标记表达式中的变量，并始终打印显示统计信息的每个触发器的名称。如果省略，默认使用 `false`。你可以指定 `VERBOSE true` 来启用它。

- **`COSTS`**

    包括每个计划节点的估计启动和总成本信息，以及估计的行数和每行的估计宽度。如果省略，默认使用 `true`。你可以指定 `COSTS false` 来禁用它。

- **`BUFFERS`**

   包括缓冲区使用信息。只有当 `ANALYZE` 也被指定时，才能指定此参数。如果省略，将使用默认值 `false`。你可以指定 `BUFFERS true` 来启用它。

    请注意，MAXIR 不支持为分布式查询指定 `BUFFERS true`；忽略任何显示的缓冲区使用信息。

- **`TIMING`**

    包括输出中的实际启动时间和每个节点中花费的时间。在一些系统上，反复读取系统时钟的开销可能会显著地减慢查询速度，因此当只需要实际行数，而不是精确时间时，将此参数设置为 `false` 可能会有所帮助。即使使用此选项关闭了节点级定时，也总是会测量整个语句的运行时间。只有在 `ANALYZE` 也启用时，才能使用此参数。默认为 `true`。

- **`FORMAT`**

    指定输出格式，可以是 `TEXT`、`XML`、`JSON` 或 `YAML`。非文本输出包含的信息与文本输出格式相同，但对程序解析更容易。此参数默认为 `TEXT`。

- *`<boolean>`*

    指定所选选项将被打开还是关闭。你可以写 `TRUE`、`ON` 或 `1` 来启用选项，`FALSE`、`OFF` 或 `0` 来停用它。布尔值也可以省略，在这种情况下，假定为 `TRUE`。

- *`<statement>`*

    你想看到其查询计划的任何 `SELECT`、`INSERT`、`UPDATE`、`DELETE`、`VALUES`、`EXECUTE`、`DECLARE` 或 `CREATE TABLE AS` 语句。


---

使用注意事项
----------

为了让查询优化器在优化查询时能做出合理的决策，必须运行 `ANALYZE` 语句以记录表中数据分布的统计信息。如果你还没有这样做（或者如果表中数据的统计分布自上次运行 `ANALYZE` 以来已经发生了显著变化），那么估计的成本不太可能与查询的实际属性相符，因此可能会选择较差的查询计划。

在执行 `EXPLAIN ANALYZE` 命令期间运行的 SQL 语句被排除在 MAXIR 资源队列之外。



---

SQL 标准兼容性
----------

SQL 标准中没有定义 `EXPLAIN` 语句。
