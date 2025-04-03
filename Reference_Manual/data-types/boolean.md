# 布尔类型

MAXIR 支持 `boolean` 类型。`boolean` 类型状态：“真”、“假”和“未知”（输出为 null）。

下表列出了数据类型输入函数可以接受的表示“真”、“假”状态的字符串。

| 真 | 假 |
| :- | :- |
| true | false |
| t | f |
| yes | no |
| y | n |
| 1 | 0 |

<br/>

数据类型为 `boolean` 的数据类型输出函数使用 `t` 表示“真”，`f` 表示“假”。

---
## 示例

```sql
CREATE TABLE test1 (a boolean, b text);
INSERT INTO test1 VALUES (TRUE, 'sic est');
INSERT INTO test1 VALUES (FALSE, 'non est');
SELECT * FROM test1;
 a |    b
---+---------
 t | sic est
 f | non est

SELECT * FROM test1 WHERE a;
 a |    b
---+---------
 t | sic est
```

---
## 使用注意事项

在 SQL 查询中编写布尔常量时，推荐使用 `TRUE` 和 `FALSE`。你也可以选择使用通用字符串文字常量语法来表示，例如 `'yes'::boolean`。更多详细信息，请参见 PostgreSQL 文档中的 [第 4.1.2.7 节](https://www.postgresql.org/docs/12/sql-syntax-lexical.html#SQL-SYNTAX-CONSTANTS-GENERIC)。
