# Hybrid DPS 与 Extreme DPS 偏差表

作为 Relyt 提供的两种计算引擎，Hybrid DPS 和 Extreme DPS 在性能、查询支持、函数支持等方面存在少量差异。

本文记录了 Hybrid DPS 和 Extreme DPS 在 SQL 命令和函数支持方面的差异，供你快速参考。

## 查询支持差异

Extreme DPS 在 SQL 查询上存在少量限制。当使用 Extreme DPS 集群的时候，请注意包含如下数据库对象的查询无法在 Extreme DPS 集群上运行：

- 通过 [CREATE FUNCTION](reference/sql-commands/create-function.md) 创建的函数

- 通过 [CREATE PROCEDURE](reference/sql-commands/create-procedure.md) 创建的存储过程

- 通过 [DECLARE](reference/sql-commands/declare.md) 声明的游标

此外：

- Extreme DPS 在使用 [SELECT](reference/sql-commands/select.md#window-子句) 中的 `WINDOW` 子句时，存在一些限制




## 函数支持差异

下表列出了 Hybrid DPS 和 Extreme DPS 都支持、但是在具体行为上存在差异的函数。

| 函数 | Hybrid DPS | Extreme DPS |
| :- | :- | :- |
| `\|\|` | 如果 [\|\|](reference/hybrid-dps-functions/builtin-functions.md) 要连接的字符串中有 null，则返回 null。例如， `'a' \|\| null -> null` | [\|\|](reference/extreme-dps-functions/string-functions.md) 忽略 null 值。例如，`'a' \|\| null -> 'a'` | 
| `CONCAT_WS` | 被连接的字符串可以是任何支持的数据类型。 | 被连接的字符串只能是 `text` 类型。 | 

<br/>

下表列出了仅 Extreme DPS 支持的函数。

| 函数 | 描述 |
| :- | :- |
| [DATEDIFF](reference/extreme-dps-functions/date-time-functions.md#datediff) | 返回两个 `date` 或 `timestamp` 表达式的日期部分之间的差异。 |
| [FROM_UNIXTIME](reference/extreme-dps-functions/date-time-functions.md#from_unixtime) | 将 UNIX 时间戳 `unixtime` 作为带有时区的时间戳，使用会话中的时区 `zone`。 |
| [WINDOW_FUNNEL](reference/extreme-dps-functions/date-time-functions.md#window_funnel) | 搜索滑动时间窗口内的事件列表，计算条件匹配的事件链里的最大连续事件数。|
| [TRY](reference/extreme-dps-functions/conditional-expressions.md#try) | 对输入表达式进行评估，并通过返回 NULL 来评估处理某些类型的错误。 |
| [数组函数](reference/extreme-dps-functions/array-functions.md#数组函数)  | 用于帮助用户处理数组数据的函数。 |
| [JSON 函数](reference/extreme-dps-functions/json-functions.md) | 用于帮助用户处理 JSON 数据的函数。 |
| [转换函数](reference/extreme-dps-functions/conversion-functions.md) | 实现数值类型到字符类型或字符类型到数字类型的转换。 |
 
<br/>