---
sidebar_position: 6
---


# 模式匹配

Extreme DPS 支持使用 `LIKE` 运算符以及正则表达式来进行模式匹配。

---

## LIKE

基于与模式的比较对字符串进行区分大小写的匹配。

请注意，`LIKE` 不支持非确定性排序规则。如需规避此限制，可以在表达式上使用不同的排序规则。

### 语法

```sql
<string> LIKE <pattern> [ESCAPE <escape-character>]
<string> NOT LIKE <pattern> [ESCAPE <escape-character>]
```

### 参数

- *`<string>`*

    用于匹配模式的字符串。它可以是 `varchar` 或 `text` 类型。

- *`<pattern>`*

    要匹配的模式。它可以是 `varchar` 或 `text` 类型。

- *`<escape-character>`*

    转义字符，插入在通配符字符前的一个或多个字符，表示该通配符必须被解释为一个常规字符，而不是一个通配符。

### 示例


**`LIKE` 示例：**

```sql
'john' LIKE 'john' → true
'john' LIKE '%o%' → true
'john' LIKE '_oh_' → true
'john' LIKE 'tom' → false
'john' LIKE NULL → NULL
NULL LIKE '%abc%' → NULL
```

输出值说明：

- `true`： *`<string>`* 和 *`<pattern>`* 匹配。

- `false`：*`<string>`* 和 *`<pattern>`* 不匹配。

- `NULL`：*`<string>`* 或 *`<pattern>`* 为 null。



**`NOT LIKE` 示例：**

```sql
'john' NOT LIKE 'john' → false
'john' NOT LIKE '%o%' → false
'john' NOT LIKE '_oh_' → false
'john' NOT LIKE 'tom' → true
'john' NOT LIKE NULL → NULL
NULL NOT LIKE '%abc%' → NULL
```

输出值说明：

- `true`：*`<string>`* 和 *`<pattern>`* 不匹配。

- `false`：*`<string>`* 和 *`<pattern>`* 匹配。

- `NULL`：*`<string>`* 或 *`<pattern>`* 为 null。

### 使用注意事项

`LIKE` 模式匹配总是覆盖整个字符串。因此，如果需要匹配字符串中的一个序列，需要在模式的开始和结束处都使用百分号 (%)。

要匹配实际的下划线 (_) 或百分号 (%) 而不匹配其他字符，*`<pattern>`* 中的相应字符必须前置转义字符。默认的转义字符是反斜杠 (\)。你可以使用 `ESCAPE` 子句指定适合你需求的转义字符。要匹配转义字符本身，需要连续写两个转义字符。

:::note 提示
如果你禁用了 `standard_conforming_strings`，你需要在字面字符串常量中双引号反斜杠 (\)。  
:::

要禁用转义机制，使用 `[NOT] LIKE ... ESCAPE ''`。

你可以使用关键字 `ILIKE` 替代 `LIKE`，使匹配根据活动区域设置进行不区分大小写。

下表描述了可以在 `LIKE` 模式匹配中使用的每个关键字的等效运算符。

| 关键字 | 等效运算符 |
| :- | :- |
| `LIKE` | `~~` |
| `ILIKE` | `~~*` |
| `NOT LIKE` | `!~~` |
| `NOT ILIKE` | `!~~*` |

<br/>

Extreme DPS 支持使用 `LIKE`、`ILIKE`、`NOT LIKE` 和 `NOT ILIKE` 作为运算符，可以在 *`<expression>`* *`<operator>`* `ANY` (*`<subquery>`*) 结构中使用。唯一的区别是，在此类结构中，不能包含 `ESCAPE` 子句。在极少数情况下，你不能使用这些关键字替代其对应的运算符。

---
## 正则表达式函数

正则表达式函数是用于执行于正则表达式（即 regex）匹配的操作的字符串函数。


### 支持的正则表达式函数列表

Extreme DPS 支持使用正则表达式函数来进行模式匹配，如下表所示：

| 函数 | 返回类型 | 描述 | 
| :- | :- | :- |
| `REGEXP_REPLACE(subject text, pattern text)` | `text` | 将指定字符串 *`<subject>`* 中与 *`<pattern>`* 匹配的所有子字符串替换为空。 | 
| `REGEXP_REPLACE(subject text, pattern text [, replacement text])`  | `text` | 将指定字符串 *`<subject>`* 中与 *`<pattern>`* 匹配的所有子字符串替换为 *`<replacement`>* 指定的字符串。 | 
| `REGEXP_LIKE(subject text, pattern text)` | `boolean` | 如果指定字符串 *`<subject>`* 与 *`<pattern>`* 匹配，则返回为真。|
| `REGEXP_EXTRACT(subject text, pattern text)` | `text` | 返回指定字符串 *`<subject>`* 中与 *`<pattern>`* 匹配的第一个子字符串。|
| `REGEXP_EXTRACT(subject text, pattern text, group integer)` | `text` | 找到指定字符串 *`<subject>`* 中与 *`<pattern>`* 匹配的第一个子字符串并返回指定组号 *`<group>`* 的文本。 |

<br/>

### 示例

```sql
-- REGEXP_REPLACE(subject text, pattern text) 示例
SELECT REGEXP_REPLACE('fun stuff.', '[a-z]')  →  ' .'
SELECT REGEXP_REPLACE('call 555.123.4444 now', '(\d{3})\.(\d{3}).(\d{4})')  → 'call  now'

-- REGEXP_REPLACE(subject text, pattern text [, replacement text]) 示例
SELECT REGEXP_REPLACE('fun stuff.', '[a-z]', '*')  → '*** *****.'
SELECT REGEXP_REPLACE('call 555.123.4444 now', '(\d{3})\.(\d{3}).(\d{4})', '($1) $2-$3') → 'call (555) 123-4444 now'

-- REGEXP_LIKE(subject text, pattern text) 示例
SELECT REGEXP_LIKE('Stevens', 'Ste(v|ph)en') → true
SELECT REGEXP_LIKE('Hello', '^[a-zA-Z]+$') → true
SELECT REGEXP_LIKE('12345', 'x') → true

-- REGEXP_EXTRACT(subject text, pattern text) 示例
SELECT REGEXP_EXTRACT('Hello world bye', '\b[a-z]+') → 'world'
SELECT REGEXP_EXTRACT('12345', 'x') → null

-- REGEXP_EXTRACT(subject text, pattern text, group integer) 示例
SELECT REGEXP_EXTRACT('Hello world bye', '\b[a-z]([a-z]*)', 1) → 'orld'
SELECT REGEXP_EXTRACT('rat cat\nbatdog', 'gg(.)|ba(.)(.)', 2) → 't'
```