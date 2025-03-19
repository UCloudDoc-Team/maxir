---
sidebar_position: 8
---

# 行和数组比较

Extreme DPS 允许你使用 `IN` 或 `NOT IN` 对值组进行多次比较。

---

## IN

`IN` 函数检查给定的表达式是否在表达式列表中。

### 语法

```sql
<expression> IN (<value> [, ...])
```

右侧是一个带括号的标量表达式列表。当 *`<expression>`* 与右侧列表中的任何成员匹配时，返回 `true`。

### 参数

- *`<expression>`*

    检查它是否是值列表中的成员的表达式。

- *`<value>`*

    与 *`<expression>`* 匹配的列表中的值。


### 示例

```sql
'john' IN ('john', 'tome', 'gary') → true
'john' IN ('x', 'y', 'z') → false
NULL IN ('x', 'y', 'z') → NULL
'john' IN (NULL, NULL) → NULL
```

### 使用注意事项

如果左侧表达式产生 null，或者没有等于右侧值且至少有一个右侧表达式产生 null，结果将为 null。

---

## NOT IN

`NOT IN` 函数检查给定的表达式是否不在表达式列表中。

### 语法

```sql
<expression> NOT IN (<value> [, ...])
```

右侧是一个带括号的标量表达式列表。如果左侧表达式的结果不等于任何右侧表达式，将返回 `true`。

### 参数

详细参数说明，请参考“IN”部分的 [参数](#参数)。

### 示例

```sql
'john' NOT IN ('john', 'tome', 'gary') → false
'john' NOT IN ('x', 'y', 'z') → true
NULL NOT IN ('x', 'y', 'z') → NULL
'john' NOT IN (NULL, NULL) → NULL
```

### 使用注意事项

如果左侧表达式产生 null，或者不存在等于右侧值且至少有一个右侧表达式产生 null，`NOT IN` 的结果将为 null。