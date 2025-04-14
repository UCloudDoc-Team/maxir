# 比较函数和运算符

比较函数和运算符会在两个表达式之间进行计算，生成布尔值或在状态未知的情况下返回 `null`。

## 比较运算符

下表列出了 Extreme DPS 支持的比较运算符。

| 运算符 | 语法 | 描述 |
| :- | :- | :- |
| `<` | `a < b` | *a* 小于 *b*。|
| `>` | `a > b` | *a* 大于 *b*。|
| `<=` | `a <= b` | *a* 小于或等于 *b*。|
| `>=` | `a >= b` | *a* 大于或等于 *b*。|
| `=` | `a = b` | *a* 等于 *b*。| 
| `<>` | `a <> b` | *a* 不等于 *b*。|
| `!=` | `a != b` | *a* 不等于 *b*。|

<br/>


!>使用限制：<br/>
在 Extreme DPS 中，比较计算符 `<`、`>`、`<=` 和 `>=` 的输入类型不能为 `interval`。


## 比较谓词

下表列出了 Extreme DPS 支持的比较谓词。

| 谓词 | 语法 | 描述 |
| :- | :- | :- |
| `BETWEEN` | `a BETWEEN x AND y` | *a* 大于或等于 *x* 并且小于或等于 *y*。|
| `NOT BETWEEN` | `a NOT BETWEEN x AND y` | *a* 小于 *x* 或大于 *y*。|
| `IS DISTINCT FROM` | `a IS DISTINCT FROM b` | *a* 不等于 *b*。`null` 被视为普通值。|
| `IS NOT DISTINCT FROM` | `a IS NOT DISTINCT FROM b` | *a* 等于 *b*。`null` 被视为普通值。|
| `IS NULL` | `a IS NULL` | *a* 是 null。|
| `IS NOT NULL` | `a IS NOT NULL` | *a* 不是 null。|

<br/>


!>使用限制<br/>
Extreme DPS 对比较谓词的输入类型有一些限制：<br/>
`BETWEEN`、`NOT BETWEEN`、`IS NULL` 和 `IS NOT NULL` 的输入类型不能为 `interval`。<br/>
`IS DISTINCT FROM` 和 `IS NOT DISTINCT FROM` 的输入类型不能为 `interval`。<br/>
 
