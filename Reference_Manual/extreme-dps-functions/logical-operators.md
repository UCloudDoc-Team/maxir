---
sidebar_position: 2
---

# 逻辑运算符

逻辑运算符对一个或两个表达式执行计算，产生一个布尔值或 null（未知状态）。

下表描述了 Extreme DPS 支持的逻辑运算符。

| 运算符 | 语法 | 描述 |
| :- | :- | :- |
| `AND` | `a AND b` | 同时满足表达式 *a* 和 *b*。 |
| `NOT` | `NOT a` | 不满足表达式 *a*。 |
| `OR` | `a OR b` | 满足表达式 *a* 或 *b*。 |

<br/>

以下真值表显示了 `AND` 和 `OR` 的工作方式。

| a | b | a AND b | a OR b |
| :- | :- | :- | :- |
| TRUE | TRUE | TRUE | TRUE |
| TRUE | FALSE | FALSE | TRUE |
| TRUE | NULL | NULL | TRUE |
| FALSE | FALSE | FALSE | FALSE |
| FALSE | TRUE | FALSE | TRUE |
| NULL | NULL | NULL | NULL |

<br/>

以下真值表显示了 `NOT` 的工作方式。

| a | NOT a |
| :- | :- |
| TRUE | FALSE |
| FALSE | TRUE |
| NULL | NULL |

<br/>

`AND` 和 `OR` 运算符是可交换的，即右操作数和左操作数交换也不会影响结果。例如，`a AND b` 和 `b AND a` 产生相同的结果。
