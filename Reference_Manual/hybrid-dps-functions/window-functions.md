# 窗口函数

以下表格说明了 MAXIR 提供的内置窗口函数。所有的窗口函数都是 _immutable_。

| 函数 | 返回类型 | 完整语法 | 描述 |
| :- | :- | :- | :- |
| cume_dist() | `double precision` | CUME_DIST() OVER ( [PARTITION BY *expr* ] ORDER BY *expr* ) | 计算一组值中一个值的累积分布。等值行总是计算出相同的累积分布值。 |
| dense_rank() | `bigint` | DENSE_RANK () OVER ( [PARTITION BY *expr* ] ORDER BY *expr* ) | 在有序的一组行中计算一行的排名，不跳过排名值。等值行被赋予相同的排名值。 | 
| first_value(*expr*) | 输入 *expr* 类型相同 | FIRST_VALUE( *expr* ) OVER ( [PARTITION BY *expr* ] ORDER BY *expr* [ROWS\|RANGE  *frame_expr* ] ) | 返回有序值集中的第一个值。 |
| lag(*expr* [, *offset*] [, _default_]) | 输入 _expr_ 类型相同 | LAG( _expr_  [, _offset_ ] [, _default_ ]) OVER ( [PARTITION BY _expr_ ] ORDER BY _expr_ ) | 在不进行自连接的情况下，提供对同一表的多行的访问。给定一个查询返回的一系列行和光标的位置，`LAG` 提供对该位置之前给定物理偏移处的行的访问。默认的 `offset` 是 1。_default_ 设置如果偏移超出窗口范围时返回的值。如果未指定 _default_，默认值为 null。|
| last_value(*expr*) | 输入 _expr_ 类型相同 | LAST_VALUE(_expr_) OVER ( [PARTITION BY _expr_] ORDER BY _expr_ [ROWS\|RANGE _frame\_expr_ ] ) | 返回有序值集中的最后一个值。 |
| lead( _expr_ [, _offset_ ] [, _default_ ]) | 输入 _expr_ 类型相同 | LEAD( *expr* [, _offset_] [, _expr_ _default_]) OVER ( [PARTITION BY *expr*] ORDER BY *expr* ) | 在不进行自连接的情况下，提供对同一表的多行的访问。给定一个查询返回的一系列行和光标的位置，`lead` 提供对该位置之后给定物理偏移处的行的访问。如果未指定 _offset_，默认偏移量为 1。_default_ 设置如果偏移超出窗口范围时返回的值。如果未指定 _default_，默认值为 null。 |
| ntile(_expr_) | `bigint` | NTILE(*expr*) OVER ( [PARTITION BY *expr*] ORDER BY *expr* ) | 将有序数据集划分为多个桶（由 _expr_ 定义）并为每行分配一个桶号。 |
| percent_rank() | `double precision` | PERCENT_RANK () OVER ( [PARTITION BY *expr*] ORDER BY *expr*) | 计算假设行 `R` 的排名减 1，除以正在评估的行数减 1（在窗口分区内）。 |
| rank() | `bigint` | RANK () OVER ( [PARTITION BY *expr*] ORDER BY *expr*) | 在有序的值组中计算一行的排名。具有相等排名标准的行获得相同的排名。计算下一个排名值时，会将并列的行数添加到排名数字中。在这种情况下，排名可能不是连续的数字。 |
| row_number() | `bigint` | ROW_NUMBER () OVER ( [PARTITION BY *expr*] ORDER BY *expr*) | 为应用它的每一行（窗口分区中的每一行或查询的每一行）分配一个唯一的数字。 |
