```sql
---
sidebar_position: 2
---
```




# 内置函数和运算符

以下表格列出了 MAXIR 支持的内置函数和运算符的类别。MAXIR 支持所有的函数和运算符，除了 `STABLE` 和 `VOLATILE` 函数，这些函数受到 MAXIR Function Types 中所述的限制。有关这些内置函数和运算符的更多信息，请参阅 PostgreSQL 文档的 [Functions and Operators](https://www.postgresql.org/docs/12/functions.html) 部分。

| 运算符/函数类别 | 易变函数 | 稳定函数 | 限制 |
| :- | :- | :- | :- |
| [逻辑运算符](https://www.postgresql.org/docs/12/functions-logical.html) | N/A | N/A | N/A |
| [比较运算符](https://www.postgresql.org/docs/12/functions-comparison.html) | N/A | N/A | N/A |
| [数学函数和运算符](https://www.postgresql.org/docs/12/functions-math.html) | random <br/><br/>setseed | N/A | N/A |
| [字符串函数和运算符](https://www.postgresql.org/docs/12/functions-string.html) | 所有内置转换函数 | convert<br/><br/>pg_client_encoding | N/A |
| [二进制字符串函数和运算符](https://www.postgresql.org/docs/12/functions-binarystring.html) | N/A | N/A | N/A |
| [位串函数和运算符](https://www.postgresql.org/docs/12/functions-bitstring.html) | N/A | N/A | N/A | 
| [模式匹配](https://www.postgresql.org/docs/12/functions-matching.html) | N/A | N/A | N/A | 
| [数据类型格式化函数](https://www.postgresql.org/docs/12/functions-formatting.html) |  N/A | to_char<br/><br/>to_timestamp | N/A |
| [日期/时间函数和运算符](https://www.postgresql.org/docs/12/functions-datetime.html) | timeofday | age<br/><br/>current_date<br/><br/>current_time<br/><br/>current_timestamp<br/><br/>localtime<br/><br/>localtimestamp<br/><br/>now | N/A |
| [序列操作函数](https://www.postgresql.org/docs/12/functions-sequence.html) |nextval()<br/><br/>setval() | N/A | N/A |
| [条件表达式](https://www.postgresql.org/docs/12/functions-conditional.html) | N/A | N/A | N/A |
| [数组函数和运算符](https://www.postgresql.org/docs/12/functions-array.html) | N/A | 所有数组函数 | N/A |
| [聚合函数](https://www.postgresql.org/docs/12/functions-aggregate.html) | N/A | N/A | N/A |
| [子查询表达式](https://www.postgresql.org/docs/12/functions-subquery.html) | N/A | N/A | N/A | 
| [行和数组比较](https://www.postgresql.org/docs/12/functions-comparisons.html) | N/A | N/A | N/A | 
| [返回集函数](https://www.postgresql.org/docs/12/functions-srf.html) | generate_series | N/A | N/A | 
| [系统信息函数](https://www.postgresql.org/docs/12/functions-info.html) | N/A| 所有会话信息函数<br/><br/>所有访问权限查询函数<br/><br/>所有 Schema 可见性查询函数<br/><br/>所有系统目录信息函数<br/><br/>所有注释信息函数<br/><br/>所有事务 ID 和快照 | N/A |
| [系统管理函数](https://www.postgresql.org/docs/12/functions-admin.html) | set_config<br/><br/>pg\_cancel\_backend<br/><br/>pg\_reload\_conf<br/><br/>pg\_rotate\_logfile<br/><br/>pg_rotate_logfile<br/><br/>pg_start_backup<br/><br/>pg\_size\_pretty<br/><br/>pg\_ls\_dir<br/><br/>pg\_read\_file<br/><br/>pg\_stat\_file | current\_setting<br/><br/>所有数据库对象大小函数 | 函数 `pg_column_size` 显示存储值所需的字节，可能带有 TOAST 压缩。 |
| [XML 函数](https://www.postgresql.org/docs/12/functions-xml.html) 和函数式表达式 | N/A | cursor\_to\_xml(cursor refcursor, count int, nulls boolean, tableforest boolean, targetns text)<br/><br/>cursor\_to\_xmlschema(cursor refcursor, nulls boolean, tableforest boolean, targetns text)<br/><br/>database\_to\_xml(nulls boolean, tableforest boolean, targetns text)<br/><br/>database\_to\_xmlschema(nulls boolean, tableforest boolean, targetns text)<br/><br/>database\_to\_xml\_and\_xmlschema(nulls boolean, tableforest boolean, targetns text)<br/><br/>query\_to\_xml(query text, nulls boolean, tableforest boolean, targetns text)<br/><br/>query\_to\_xmlschema(query text, nulls boolean, tableforest boolean, targetns text)<br/><br/>query\_to\_xml\_and\_xmlschema(query text, nulls boolean, tableforest boolean, targetns text)<br/><br/>schema\_to\_xml(schema name, nulls boolean, tableforest boolean, targetns text)<br/><br/>schema\_to\_xmlschema(schema name, nulls boolean, tableforest boolean, targetns text)<br/><br/>schema\_to\_xml\_and\_xmlschema(schema name, nulls boolean, tableforest boolean, targetns text)<br/><br/>table\_to\_xml(tbl regclass, nulls boolean, tableforest boolean, targetns text)<br/><br/>table\_to\_xmlschema(tbl regclass, nulls boolean, tableforest boolean, targetns text)<br/><br/>table\_to\_xml\_and\_xmlschema(tbl regclass, nulls boolean, tableforest boolean, targetns text)<br/><br/>xmlagg(xml)<br/><br/>xmlconcat(xml\[, ...\])<br/><br/>xmlelement(name name \[, xmlattributes(value \[AS attname\] \[, ... \])\] \[, content, ...\])<br/><br/>xmlexists(text, xml)<br/><br/>xmlforest(content \[AS name\] \[, ...\])<br/><br/>xml\_is\_well\_formed(text)<br/><br/>xml\_is\_well\_formed\_document(text)<br/><br/>xml\_is\_well\_formed\_content(text)<br/><br/>xmlparse ( \{ DOCUMENT \| CONTENT \} value)<br/><br/>xpath(text, xml)<br/><br/>xpath(text, xml, text\[\])<br/><br/>xpath\_exists(text, xml)<br/><br/>path\_exists(text, xml, text\[\])<br/><br/>xmlpi(name target \[, content\])<br/><br/>xmlroot(xml, version text \| no value \[, standalone yes\|no\|no value\])<br/><br/>xmlserialize ( \{ DOCUMENT | CONTENT \} value AS type )<br/><br/>xml(text)\text(xml)<br/><br/>xmlcomment(xml)\xmlconcat2(xml, xml) | N/A |

  
