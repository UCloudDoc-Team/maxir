# 字符串函数和运算符

字符串函数和运算符处理字符串输入并返回字符串或数值。Extreme DPS 支持的字符串类型是 `varchar` 和 `text`。有关字符串类型的详细说明，请参考 [字符类型](/maxir/Reference_Manual/data-types/character.md)。

---

## SQL 字符串函数和运算符

本节详细介绍了 Extreme DPS 支持的每个 SQL 字符串函数和运算符。

 

### ||

连接一个或多个字符串。

>说明 <br/>
>Null 值会被忽略。

#### 语法

```sql
<string1> || <string2>[, ... <stringN>]
```

#### 参数

*`<stringN>`*：要连接的 `varchar` 字符串。

#### 返回

`varchar` 值。

#### 示例

```sql
SELECT 'Extreme' || 'DPS';
-> ExtremeDPS
```
 

### BIT_LENGTH

返回字符串的长度（以位为单位）。

#### 语法

```sql
BIT_LENGTH(<string>)
```

#### 参数

*`<string>`*：要评估长度的 `varchar` 字符串。

#### 返回

`integer` 值。

#### 示例

```sql
SELECT BIT_LENGTH('Extreme DPS');
-> 88
```
 


### CHAR_LENGTH 或 CHARACTER_LENGTH

返回字符串的字符数量。

>说明 <br/>
>`CHARACTER_LENGTH` 是 `CHAR_LENGTH` 的别名。

#### 语法

```sql
CHAR_LENGTH(<string>)
CHARACTER_LENGTH(<string>)
```

#### 参数

*`<string>`*：要计算包含字符数量的字符串。

#### 返回

`integer` 值。

#### 示例

```sql
SELECT CHAR_LENGTH('Extreme DPS');
-> 11
```
 


### CONCAT_WS

通过分隔符将字符串连接起来。

函数中指定的第一个参数是分隔符。空值将被忽略。

#### 语法

```sql
CONCAT_WS( <separator>, <string1>[, <string2> ...])
```

#### 参数

- *`<separator>`*: 作为分隔符的 `text` 字符串。

- *`<string>`*: 要连接的 `text` 字符串。


#### 返回值

`text` 值。

#### 示例

```sql
SELECT CONCAT_WS(':', 'adbcd', 'efg', 'hijk');
-> 'adbcd:efg:hijk'
```

 

### LOWER

将字符串中的所有字符转换为小写。

#### 语法

```sql
LOWER(<string>)
```

#### 参数

*`<string>`*：要转换的 `varchar` 字符串。

#### 返回值

`varchar` 值。

#### 示例

```sql
SELECT LOWER('Extreme DPS');
-> extreme dps
```
 

### OCTET_LENGTH

返回字符串的长度（以字节为单位）。

#### 语法

```sql
OCTET_LENGTH(<string>)
```

#### 参数

*`<string>`*：要评估长度的 `varchar` 字符串。

#### 返回值

`integer` 值。

#### 示例

```sql
SELECT OCTET_LENGTH('Extreme DPS');
-> 11
```
 

### POSITION

返回子字符串在字符串中首次出现的位置。

字符串中第一个字符的位置为 1。

#### 语法

```sql
POSITION(<substring> in <string>)
```

#### 参数

- *`<substring>`*

    要搜索的 `varchar` 字符串。

- *`<string>`*
    
    要搜索的 `varchar` 字符串。

#### 返回

`integer` 值。

#### 示例

```sql
>SELECT POSITION('dps' in 'extremedps');
8
```
 

### SUBSTRING 或 SUBSTR

返回从指定位置开始的子字符串，如果指定了字符数，则返回该数量的字符。

字符串中的位置为 1。

#### 语法

```sql
SUBSTRING(<string>[ from <position>] [for <length>] )
SUBSTR(<string>[ from <position>] [for <length>] )
SUBSTR(<string>, <position> [, <length>])
```

其中，`SUBSTRING(<string>[ from <position>] [for <length>] )` 与  `SUBSTR(<string>[ from <position>] [for <length>] )` 等效，`SUBSTRING(<string> from <position> for <count>)` 与 `SUBSTR(<string>, <position> [, <count>])` 等效。

请注意，不支持 `SUBSTRING(<string> for <length>)` 或 `SUBSTR(<string> for <length>)`。

#### 参数

- *`<string>`*

    要搜索的 `varchar` 字符串。

- *`<position>`*

    指定子字符串开始的偏移量 `integer` 表达式。

- *`<length>`*

    指定子字符串的长度 `integer` 表达式。

#### 返回

`varchar` 值。

如果有任一参数为 null，将返回 null。

#### 示例

```sql
SELECT SUBSTRING('Extreme DPS' from 3 for 3);
-> tre

SELECT SUBSTRING('Extreme DPS' from 3);
-> treme DPS

SELECT SUBSTR('null' from 3);
-> null

SELECT SUBSTR('alphabet', 3, 2);
-> ph
```
 

### UPPER

将字符串中的所有字符转换为大写。

#### 语法

```sql
UPPER(<string>)
```

#### 参数

*`<string>`*：要转换的 `varchar` 字符串。
#### 返回值

`varchar` 值。

#### 示例

```sql
SELECT UPPER('ExtremeDPS');
-> EXTREMEDPS
```

---

## 其他字符串函数

Extreme DPS 还支持许多字符串处理函数。其中一些用于在[上一个章节](#sql字符串函数和运算符)中描述的 SQL 字符串函数的实现。

本节详细介绍了 Extreme DPS 支持的每个 SQL 字符串函数和运算符。

 

### ASCII

返回字符串第一个字符的 ASCII 码，或者如果使用 `UTF8`，则返回字符的 Unicode 码。

#### 语法

```sql
ASCII(<string>)
```

#### 参数

*`<string>`*：将返回其第一个字符的 ASCII 码的字符串。

#### 返回

`integer` 值。

如果 *`<string>`* 为 null，则返回 null。

#### 示例

```sql
SELECT ASCII('xyz');
-> 120
```
 

### BTRIM

返回删除了前导和尾随字符的给定字符串。

#### 语法

```sql
BTRIM(<string> [, <trimStr>])
```

#### 参数

- *`<string>`*

    要修剪的 `varchar` 字符串。

- *`<trimStr>`*

    需要从 *`<string>`* 的左右两侧删除的字符。

    *`<trimStr>`* 的默认值是 ' '。默认值下，将删除前导和尾随的空格。

#### 返回值

`varchar` 值。

如果 *`<string>`* 为 null，则返回 null。

#### 示例

```sql
SELECT BTRIM('xyxtrimyyx', 'xyz');
-> trim

SELECT BTRIM('xyxtrimyyx  ', ' ');
-> xyxtrimyyx
```
 

### CHR

将 Unicode 码点转换为与输入 Unicode 匹配的字符。

对于 UTF8，参数被视为 Unicode 码点。对于其他多字节编码，参数必须指定 ASCII 字符。不允许使用 NULL（0）字符，因为文本数据类型无法存储此类字节。

#### 语法

```sql
CHR(<integer>)
```

#### 参数

*`<integer>`*：要转换的 `integer` 表达式。

#### 返回

`varchar` 值。

如果 *`<integer>`* 为 null，则返回 null。

#### 示例

```sql
SELECT CHR(65);
-> A
```
 

### CONCAT

连接一个或多个字符串。

#### 语法

```sql
CONCAT(<string1>[, ... <string n>])
```

#### 参数

*`<string n>`*：任何数据类型的字符串，用于连接。

#### 返回

`varchar` 值。

如果任何字符串为 null，将被忽略。

#### 示例

```sql
SELECT CONCAT('abcde', 2, NULL, 22);
-> abcde222
```
 

### INITCAP

返回输入字符串，其中每个单词的第一个字母为大写，其余部分为小写。

#### 语法

```sql
INITCAP(<string>)
```

#### 参数

*`<string>`*：需要转换的字符串。

#### 返回

`varchar` 值。

如果任何字符串为 null，将被忽略。

#### 示例

```sql
SELECT INITCAP('welcome to maxir');
-> Welcome To MAXIR
```
 

### LEFT

返回字符串中指定数量的最左边的字符的子字符串。

#### 语法

```sql
LEFT(<string>, <length>)
```

#### 参数

- *`<string>`*

    要搜索的 `text` 表达式。

- *`<length>`*

    指定要返回的子字符串的长度的 `integer` 表达式。

    如果 *`<length>`* 是正整数，将返回最左边的 *`<length>`* 个字符。如果 *`<length>`* 是负数，将返回字符串中除了最右边的 -*`<length>`* 个字符以外的所有字符。如果 *`<length>`* 为 `0`，将返回 null。

#### 返回

`varchar` 值。

#### 示例

```sql
SELECT LEFT('abcde', 2);
-> ab
```
 

### LENGTH

返回字符串中的字符数。

#### 语法

```sql
LENGTH(<string>)
```

#### 参数

*`<string>`*：要计数字符数的 `text` 或 `varchar` 表达式。

#### 返回

`integer` 值。

#### 示例

```sql
SELECT LENGTH('extremedps');
-> 10
```
 

### LPAD

向字符串的左侧填充指定的字符，使字符串达到指定长度。如果字符串本身已经超过指定的长度，字符串将被截断。

#### 语法

```sql
LPAD(<string>, <length> [, <pad>])
```

#### 参数

- *`<string>`*

    需要处理的 `text` 字符串。

- *`<length>`*

   `integer` 表达式，指定填充后的 *`<string>`* 包含的字符数量。

- *`<pad>`*

    `text` 表达式，用于填充 *`<string>`*。

    *`<pad>`* 的默认值为空格，表示在未指定 *`<pad>`* 的情况下，将在 *`<string>`* 左侧填充空格至指定长度。

#### 返回

`varchar` 值。

#### 示例

```sql
SELECT LPAD('hi', 5, 'xy');
-> xyxhi
```
 

### LTRIM

返回一个删除了前导字符的字符串。

#### 语法

```sql
LTRIM(<string> [,<trimStr>])
```

#### 参数

- *`<string>`*

    需要处理的 `text` 字符串。

- *`<trimStr>`*

    `text` 子字符串。将从 *`<string>`* 的最左侧删除这个子字符串中的字符。

    *`<trimStr>`* 的默认值是 ' '，表示在未指定 *`<trimStr>`* 的情况下，将删除前导空格。

#### 返回

`varchar` 值。

#### 示例

```sql
SELECT LTRIM('zzzytest', 'xyz');
-> test
```

 

### QUOTE_LITERAL

将字符串用单引号括起来，以便将引用的字符串用作 SQL 语句字符串中的字面字符串。嵌入的单引号 (') 和反斜杠 (\) 将被正确地加倍。

#### 语法

```sql
QUOTE_LITERAL(<string>)
```

#### 参数

*`<string>`*：要用单引号引用的 `text` 字符串。

#### 返回

`varchar` 值。

如果 *`<string>`* 为 null，则返回 null。

#### 示例

```sql
SELECT QUOTE_LITERAL(E'O\'Reilly');
-> 'O''Reilly'
```

 

### QUOTE_NULLABLE

强制将给定值转换为文本，然后将其引用为字面量。嵌入的单引号 (') 和反斜杠 (\) 将被正确地加倍。

#### 语法

```sql
QUOTE_NULLABLE(<string>)
```

#### 参数

*`<string>`*：要引用的 `text` 字符串。

#### 返回

`varchar` 值。

#### 示例

```sql
SELECT QUOTE_NULLABLE(NULL);
-> NULL
```
 

### REPEAT

将字符串重复指定次数。

#### 语法

```sql
REPEAT(<string>, <number>)
```

#### 参数

- *`<string>`*

    要重复的 `text` 字符串。

- *`<number>`*

    `integer` 表达式，指定 *`<string>`* 将重复的次数。

#### 返回

`varchar` 值。

#### 示例

```sql
SELECT REPEAT('DPS', 3);
-> DPSDPSDPS
```
 

### REPLACE

在字符串中将一个子字符串的所有出现替换为另一个子字符串。

#### 语法

```sql
REPLACE(<string>, <sourceStr>, <targetStr>)
```

#### 参数

- *`<string>`*

    `text` 字符串。

- *`<sourceStr>`*

    将被替换的 `text` 子字符串。

- *`<targetStr>`*

    用于替换 *`<sourceStr>`* 的 `text` 子字符串。

#### 返回

`varchar` 值。

#### 示例

```sql
SELECT REPLACE('abcdefabcdef', 'cd', 'XX');
-> abXXefabXXef
```
 

### REVERSE

反转字符串中字符的顺序。

#### 语法

```sql
REVERSE(<string>)
```

#### 参数

*`<string>`*：要反转的 `text` 或 `varchar` 字符串。

#### 返回

`varchar` 值。

#### 示例

```sql
SELECT REVERSE('abcde');
-> edcba
```
 

### RIGHT

返回字符串中指定数量的最右边的字符的子字符串。

#### 语法

```sql
RIGHT(<string>, <length>)
```

#### 参数

- *`<string>`*

    要搜索的 `text` 表达式。

- *`<length>`*

    `integer` 表达式，指定要返回的子字符串的长度。

    如果 *`<length>`* 是正整数，将返回最右边的 *`<length>`* 个字符。如果 *`<length>`* 是负数，将返回字符串中除了最左边的 -*length* 个字符以外的所有字符。如果 *`<length>`* 是 `0`，将返回 null。

#### 返回

`varchar` 值。

#### 示例

```sql
SELECT RIGHT('abcde', 2);
-> de
```
 

### RPAD

向字符串的右侧填充指定的字符，使字符串由指定数量的字符组成。如果字符串已经超过指定的长度，字符串将被截断。

#### 语法

```sql
RPAD(<string>, <length> [, <pad>])
```

#### 参数

- *`<string>`*

    要填充的 `text` 字符串。

- *`<length>`*

    `integer` 表达式，指定填充后的 *`<string>`* 将由多少个字符组成。

- *`<pad>`*

    `text` 表达式，用于填充 *`<string>`*。

    *`<pad>`* 的默认值是空格。这表示如果未指定 *`<pad>`*，将在 *`<string>`* 填充空格。

#### 返回

`varchar` 值。

#### 示例

```sql
SELECT RPAD('hi', 5, 'xy');
-> hixyx
```
 

### RTRIM

返回一个删除了尾随字符的字符串。

#### 语法

```sql
RTRIM(<string>[, <trimStr>])
```

#### 参数

- *`<string>`*

    要修剪的 `text` 字符串。

- *`<trimStr>`*

    `text` 子字符串。将从 *`<string>`* 的后面删除这个子字符串中的字符。

    *`<trimStr>`* 的默认值是 ' '。这表示如果未指定 *`<trimStr>`*，将删除尾随空格。

#### 返回

`varchar` 值。

#### 示例

```sql
SELECT RTRIM('testxxzx', 'xyz');
-> test
```
 

### SPLIT_PART

在指定的分隔符出现处拆分字符串，并返回请求的部分。

#### 语法

```sql
SPLIT_PART(<string>, <delimiter>, <partNum>)
```

#### 参数

- *`<string>`*
    
    要拆分的 `text` 字符串。

- *`<delimiter>`*
    
    作为分隔符拆分 *`<string>`* 的 `text` 表达式。

- *`<partNum>`*
    
    `integer` 表达式，指定将返回哪个部分。


#### 返回

`varchar` 值。

如果 *`<partNum>`* 大于或等于 1，将返回 *partNum*th 部分。如果 *`<partNum>`* 是 0，将返回第一部分，结果与 `1` 相同。

#### 示例

```sql
SELECT SPLIT_PART('abc~@~def~@~ghi', '~@~', 2);
-> def
```

 


### STRPOS

返回子字符串在字符串中首次出现的位置。此函数等同于 `POSITION(varchar in varchar)`。

#### 语法

```sql
STRPOS(<string>, <substring>)
```

#### 参数

- *`<string>`*

    被搜索的 `text` 字符串。

- *`<substring>`*

    需要返回位置的 `text` 子字符串。

#### 返回

`integer` 值。

#### 示例

```sql
SELECT STRPOS('high', 'ig');
-> 2
```
 

### STARTS_WITH

如果字符串以指定的前缀开头，则返回 true。

#### 语法

```sql
STARTS_WITH(<string>, <prefix>)
```

#### 参数

- *`<string>`*

    `text` 字符串。

- *`<prefix>`*

    指定前缀的 `text` 表达式。

#### 返回

`boolean` 值。

#### 示例

```sql
SELECT STARTS_WITH('alphabet', 'alph');
-> t
```
