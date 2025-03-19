---
sidebar_position: 1
---


# 概览

MAXIR 支持[内置函数和运算符](builtin-functions.md) ，包括可以在窗口表达式中使用的[窗口函数](window-functions.md)。MAXIR 支持的函数可以分为以下三种类型：


| 函数类型 | 是否支持 | 描述 | 备注 |
| :- | :- | :- | :- |
| IMMUTABLE | 是 | 不修改数据库，如果给定相同的参数值，总是返回相同的结果。 | N/A |
| STABLE | 在大多数情况下，是 | 如果给定相同的参数值，在表扫描中返回相同的结果。 | 返回的结果会随数据库查找和参数值的变化而变化。`current_timestamp` 系列的函数是 `STABLE`。 |
| VOLATILE | 受限 | 函数值可以在单个表扫描中改变。例如，`random()`、`timeofday()`。 | 任何具有副作用的函数都是易变的，即使其结果是可预测的。例如，`setval()`。 |

