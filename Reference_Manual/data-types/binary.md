# 二进制类型

`bytea` 数据类型用于存储二进制数据。

| 名称 | 存储大小 | 取值范围 | 描述 |
| :- | :- | :- | :- |
| `bytea` | 最大：1 GB | 任意长度的二进制 | 存储可变长度的二进制数据。 |

<br/>

## 使用限制

当前暂不支持各种操作符，比如 `>`、`>=`、`=`、`!=`、`<`、`<=` 等。

<br/>

## 使用示例

本节提供了关于 `bytea` 类型数据的建表、插入、查询语句示例。


### 建表

如下示例创建了一个名为 `test1` 的表，其中 `col2` 字段的数据类型为 `bytea`。

```sql
CREATE TABLE test1 (
    col1 bigint, 
    col2 bytea
) 
USING pg_zdb 
DISTRIBUTED BY (col1);
```

!>注意 <br/>
请勿直接使用 `bytea` 类型字段作为 `DISTRIBUTED BY` 的分布键，因为 `bytea` 字段通常较长，可能导致哈希计算效率低下，从而影响写入性能。


### 写入数据

`bytea` 数据类型支持以十六进制、转义、字符串 ASCII 码三种形式写入。


以十六进制方式写入 `deadbeef`：

```sql
INSERT INTO test1 VALUES(1, '\xdeadbeef');   -- 十六进制表示（由 0-9 和 a-f 组成，且字符总数必须为偶数）
INSERT INTO test1 VALUES(1, '\x0badbeef');  -- 十六进制表示（若字符个数为奇数，可在前端补 0，例如：'\xbadbeef' → '\x0badbeef'）
```


以转义方式写入 `deadbeef`：

```sql
INSERT INTO test1 VALUES(2, '\336\255\276\357');  
-- 'deadbeef' 的十六进制与八进制映射关系如下：
--   de        ad       be       ef      -- 16 进制
--   336       255      276      357     -- 8 进制
```

以字符串的 ASCII 码直接写入 `deadbeef`：

```sql
INSERT INTO test1 VALUES(3, 'deadbeef');  
-- 直接插入字符串 'deadbeef'，此时存储的数据为文本格式，而非二进制格式
```

写入空字符串和 NULL：

```sql
INSERT INTO test1 VALUES(2, '');  
-- 插入一个空字符串，表示字段存储为空值（但非 NULL）

INSERT INTO test1 VALUES(3, NULL);  
-- 插入 NULL 值，表示字段无值（与空字符串不同）
```

### 查询数据

`bytea` 类型数据的查询以十六进制和八进制输出显示，通过 `bytea_output` 参数进行控制。


以十六进制显示：

```sql
SET bytea_output='hex';
SELECT * FROM test1;
```


以八进制显示（转义）：

```sql
SET bytea_output='escape';
SELECT * FROM test1;
```
<br/>

## 二进制类型与字符串类型的区别

二进制字符串是由一系列字节组成的序列。二进制字符串与普通字符串（字符类型）的区别主要体现在两个方面：

**存储内容限制**：

二进制字符串的操作直接处理原始字节，而普通字符串的处理则依赖于区域（Locale）设置。例如，普通字符串的排序和比较受字符集编码和排序规则影响，而二进制字符串则按字节值进行逐字节比较。因此，二进制字符串适用于存储“原始字节”数据，而普通字符串适用于存储文本信息。


**处理方式差异**：

对二进制字符串的操作直接处理原始字节，而普通字符串的处理则依赖于区域（Locale）设置。简言之，二进制字符串适用于存储程序员视为“原始字节”的数据，而普通字符串适用于存储文本。

`bytea` 类型支持两种输入输出格式：“十六进制（hex）”格式和 PostgreSQL 传统的“转义（escape）”格式。在输入数据时，同时支持这两种格式，而输出格式则由 `bytea_output` 配置参数控制，默认输出格式为十六进制（hex）。


