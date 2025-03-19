

# CREATE TABLE

创建表。


---
## 语法

```sql
CREATE TABLE [IF NOT EXISTS] 
  <table_name> ( 
  [ { <column_name> <data_type> [<column_constraint> [ ... ] ] [ ENCODING ( <storage_directive> [, ...] ) ]
    | <table_constraint>
    | LIKE <source_table> [ <like_option> ... ] 
    | COLUMN <column_name> ENCODING ( <storage_directive> [, ...] ) [, ...] }
    [, ... ]
] )
[ WITH ( <storage_parameter> [=<value>] [, ... ] ) ]
[ DISTRIBUTED BY (<column>)
    | DISTRIBUTED RANDOMLY
    | DISTRIBUTED REPLICATED]
[ CLUSTER BY ([
    { <column_name> | ( <expression> ) } 
        [ ASC | DESC ] [ NULLS { FIRST | LAST } ]
])]
[ TTL [HOT | <ttl_expression> WARM] [<delete_expression> DELETE]]


其中 <column_constraint> 可以是:
  [ CONSTRAINT <constraint_name>]
  { NOT NULL 
    | NULL 
    | DEFAULT <default_value> }

其中 <storage_directive> 可以是:
{ compresstype | compresslevel | blocksize }

其中 <table_constraint> 可以是:
  [ CONSTRAINT <constraint_name> ]


其中 <like_option> 可以是:
  { INCLUDING | EXCLUDING }
  { COMMENTS | DEFAULTS | GENERATED | STATISTICS | STORAGE | ALL }
```

---

## 描述

`CREATE TABLE` 在当前数据库中创建一个初始为空的表。表的创建者默认为表的所有者。

如果你指定了 Schema 名称，MAXIR 将在指定的 Schema 中创建表。否则，MAXIR 将在当前 Schema 中创建表。临时表存在于特殊的 Schema 中，因此在创建临时表时，您不能指定 Schema 名称。表的名称必须区别于同一 Schema 中的任何其他表、外部表或视图的名称。

`CREATE TABLE` 还会自动创建一个数据类型，该数据类型表示与表的某一行对应的复合类型。因此，表的名称不能与同 Schema 中的任何现有数据类型名称相同。

可选的约束子句指定新的或更新的行必须满足的条件，以便插入或更新操作成功。约束是一个 SQL 对象，它以各种方式帮助定义表中的有效值集。

您可以定义两种类型的约束：表约束和列约束。列约束作为列定义的一部分进行定义。表约束的定义不与特定列绑定，可以包含多个列。每一个列约束也可以写为表约束；当约束仅影响一个列时，列约束只是一个方便你使用的符号。


---

## 参数

- *`<table_name>`*

    表的名称。您可以在表名前加上 Schema 名称，从而在指定的 Schema 中创建表。否则，表将在当前 Schema 中创建。

- **`IF NOT EXISTS`**

    如果命令中包含 `IF NOT EXISTS`，当存在具有相同名称的表时，系统将不会产生报错，而是产生一条通知。

- *`<column_name>`*

    在表中创建的列的名称。

- *`<data_type>`*

    列的数据类型。这可以包括数组指定符。


- **`ENCODING ( <storage_directive> [, ...] )`**
    
    对于列，可选的 `ENCODING` 子句指定列数据的压缩类型和块大小。*`<storage_directives>`* 的支持值包括 `compresstype`、`compresslevel` 和 `encode`：

    - **compresstype**：指定压缩类型，支持 `LZ4`、`ZSTD` 或 `Uncompressed`，默认为 `LZ4`。值 `Uncompressed` 表示不压缩数据。ZSTD 提供了速度和良好的压缩比，可以用 `compresslevel` 选项进行调整。ZSTD 在常规工作负载上优于这些压缩类型。

    - **compresslevel**：指定压缩级别。如果 `compresstype` 为 `LZ4`，取值范围为 1（最快压缩）到 12（最高压缩比）的整数值，默认值为 1；如果 `compresstype` 为 `ZSTD`，取值范围为 1 到 22 的整数，默认值为 1；如果 `compresstype` 为 `ZSTD`，默认值为 1。

    - **encode**：指定编码值，取值范围为 -1 到 3 的整数，其中：

        - **-1**：表示自动编码。

            当 `encode = -1` 时，系统会根据字段类型，自动决定是否采用字典编码：对于 varchar/text 类型，系统会根据数特征，动态决定是否采用字典编码；对于其他类型，禁用字典编码。

        - **0**：使用字典编码。

        - **1**：默认值，表示以原始状态存储数据，不进行编码。


    列压缩设置从表级别继承。表示级别的数字越小，优先级越高。


- **`LIKE [<source_table> <like_option> ...]`**

    `LIKE` 子句指定一个表，新表自动从该表复制所有列名称、数据类型、非空约束和分布策略。 

    注意，存储属性如追加优化或分区结构并不会被复制。

    可选的 *`<like_option>`* 子句指定要复制原始表的哪些附加属性。指定 `INCLUDING` 将复制属性，指定 `EXCLUDING` 将省略属性。`EXCLUDING` 是默认值。如果对同一种类型的对象进行了多个规格，将使用最后一个。可用的选项包括：

    - `INCLUDING COMMENTS`：将复制注释的列、约束和索引。默认行为是排除注释，这将导致新表中的复制列和约束没有注释。

    - `INCLUDING DEFAULTS`：将复制被复制列定义的默认值。否则，不会复制默认值，会导致新表中的被复制列默认值可能为空。请注意，复制调用数据库修改函数（如 `nextval`）的默认值可能会在原始表和新表之间创建功能链接。

    - `INCLUDING GENERATED`：将复制复制列定义的任何生成表达式。默认情况下，新列将是常规基础列。

    - `INCLUDING STATISTICS`：将扩展统计信息复制到新表。

    - `INCLUDING STORAGE`：将复制复制列定义的 STORAGE 设置。默认行为是排除 STORAGE 设置，这将导致新表中的复制列具有类型特定的默认设置。

    - `INCLUDING ALL`：`INCLUDING ALL` 是所有可用选项的简化形式（在 `INCLUDING ALL` 后指定单个 `EXCLUDING` 子句可能有用，以选择除了某些特定选项之外的所有选项。）
    
    `LIKE` 子句也可以用来从视图复制列。不适用的选项将被忽略。

- **`CONSTRAINT <constraint_name>`**

    列或表约束的可选名称。如果违反约束，约束名称将出现在错误消息中，因此约束名称如“column must be positive”可以用来向客户端应用程序传达有用的约束信息。需要双引号来指定包含空格的约束名称。如果未指定约束名称，系统将生成一个名称。

- **`NOT NULL`** 

    指定列不能包含空值。

- **`NULL`**

    指定列可以包含空值。这是默认值。

    :::note
    此子句仅为兼容非标准 SQL 数据库而提供。在新应用中，不建议使用此子句。
    :::


- **`DEFAULT <default_value>`**

    `DEFAULT` 子句为其列定义中的列分配默认数据值。值可以设置为一个固定值或者 NULL。

- **`WITH <storage_parameter>=<value>`**

    `WITH` 子句指定表或索引的可选存储参数；有关详细信息，请参见下面的 [存储参数](#存储参数)。为了向后兼容，表的 `WITH` 子句也可以包括 `OIDS=false` 来指定新表的行不应包含 OID（对象标识符），`OIDS=true` 不再受支持。

- **`DISTRIBUTED BY (<column>)`**, **`DISTRIBUTED RANDOMLY`**, 或 **`DISTRIBUTED REPLICATED`**

    指定分布策略。

    - `DISTRIBUTED BY <column>` 子句指定一个列作为分布键（Distribution Key）。列的数据类型可以是 `smallint`、`integer`、`bigint`、`text` 或 `varchar`。

    - `DISTRIBUTED RANDOMLY`：将数据随机发送到分片（Shard）上。

    - `DISTRIBUTED REPLICATED`：将整个表复制到全部的分片上。

    如不指定，则系统会随机选择表中一列作为分布键。

- **`CLUSTER BY`**

    声明表中的一个或多个列（最多 3 个）作为聚簇键。聚簇键必须由至少一个列或表达式组成。 
    
    注意，聚簇键列的数据类型只能是 `smallint`、`integer`、`bigint` 或 `date`。要将其他数据类型的列用作聚簇键，它们必须转换为任何支持的数据类型。

    :::info
    如命令中需要同时指定 `CLUSTER BY` 和 `DISTRIBUTED BY`，需要先指定 `DISTRIBUTED BY`。
    :::

- **`ASC | DESC`**

    指定是否将行按升序或降序排序的选项。支持的值包括：

    - `ASC`：升序

    - `DESC`：降序
    
    如果省略此项，则默认使用 `ASC`。

- **`NULLS { FIRST | LAST }`**

   指定 NULL 值是在非 NULL 值之前还是之后返回的选项。支持的值包括：

   - `NULLS FIRST`：指定在非 NULL 值之前返回 NULL。

   - `NULLS LAST`：指定在非 NULL 值之后返回 NULL。

   如果省略此项，则默认行为随排序顺序而变化：

   - 如果排序顺序为 `ASC`，则会在非 NULL 值之前返回 NULL。

   - 如果排序顺序为 `DESC`，则会在非 NULL 值之后返回 NULL。

- **`TTL`**

    指定数据生命周期管理策略，支持为表或表中的行设置冷温热分层策略以及删除策略。

    - **`HOT | <ttl_expression> WARM`**

        数据的冷温热属性。 
        
        - **`HOT`**：表示该表为热表，表中所有数据全部常驻于缓存中，不会转冷。

        - **`<ttl_expression> WARM`**：表示该表为温表，表中数据会依据 *`<ttl_expression>`* 指定的冷温热策略转冷。转冷后的数据将从缓存中删除。表达式中引用的列类型只能为 `date`、`varchar` 和 `integer`。并且，如引用的列类型为非时间类型，需要通过函数转换为时间类型。

            :::info 使用说明
            `WARM` 属性可以在表级和行级生效。如需配置 `WARM` 属性在行级生效，则需保证如下两个条件同时满足：

            - 建表语句中使用了 `CLUSTER BY` 子句，且该子句的第一列为 `date`、`varchar` 或 `integer` 类型。

            - TTL 表达式中引用的列为 `CLUSTER BY` 子句中指定的第一列。
            :::

    - **`<delete_expression> DELETE`**

        数据的删除策略。删除策略可以在表级和行级生效。

        :::info 使用说明
        如需配置删除策略在行级生效，则需保证如下两个条件同时满足：

        - 建表语句中使用了 `CLUSTER BY` 子句，且该子句的第一列为 `date`、`varchar` 或 `integer` 类型。

        - 删除表达式中引用的列为 `CLUSTER BY` 子句中指定的第一列。  
        :::

    关于 `TTL` 子句的详细说明和示例，请参考 [使用 TTL 管理数据](i18n/zh-Hans/docusaurus-plugin-content-docs/current/guides/optimization/ttl.md)。

 

### 存储参数

- **filesize**：设置单表最大存储大小（单位：MB）。取值范围为 0 到 8192，默认值为 192。


- **blocksize**：设置每个表中每个块的大小（单位：MB）。取值范围为 0 到 1024，默认值为 64。


- **compresstype**：指定压缩类型，支持 `LZ4`、`ZSTD` 或 `Uncompressed`，默认为 `LZ4`。值 `Uncompressed` 表示不压缩数据。ZSTD 提供了速度和良好的压缩比，可以用 `compresslevel` 选项进行调整。ZSTD 在常规工作负载上优于这些压缩类型。


- **compresslevel**：指定压缩级别。如果 `compresstype` 为 `LZ4`，取值范围为 1（最快压缩）到 12（最高压缩比）的整数值，默认值为 1；如果 `compresstype` 为 `ZSTD`，取值范围为 1 到 22 的整数，默认值为 1；如果 `compresstype` 为 `ZSTD`，默认值为 1。


- **encode**：指定编码值，取值范围为 -1 到 3 的整数，其中：

    - **-1**：表示自动编码。

        当 `encode = -1` 时，系统会根据字段类型，自动决定是否采用字典编码：对于 varchar/text 类型，系统会根据数特征，动态决定是否采用字典编码；对于其他类型，禁用字典编码。

    - **0**：使用字典编码。

    - **1**：默认值，表示以原始状态存储数据，不进行编码。




---

## 使用注意事项

您不能定义具有超过 1600 列的表格。（在实际使用中，由于元组长度约束，有效限制通常较低。）

主键约束只是唯一约束和非空约束的组合。

MAXIR 不支持外键约束。

对于继承表，当前实现不继承唯一约束、主键约束、索引和表权限。


---

## 示例


创建名为 `raps` 的表，并将群集键设置为 `rank`：

```sql
CREATE TABLE raps (
    musician text,
    style text,
    rank int
)
CLUSTER BY (rank);
```


创建名为 `test` 的表，并设置文件大小为 `1024`，块大小为 `100`，压缩类型为 `lz4`，编码值为 `1`:

```sql
CREATE TABLE test(
    a integer, 
    b integer
) 
USING test_1
WITH(filesize= 1024, blocksize= 100, compresstype=lz4, compresslevel = 1, encode = 0);

```

---

## SQL 标准兼容性

`CREATE TABLE` 命令符合 SQL 标准，但存在以下例外：

 

### 列检查约束


- **NULL 约束** — `NULL` 约束是 MAXIR 对 SQL 标准的扩展，包括了与其他一些数据库系统的兼容性（以及与 `NOT NULL` 约束的对称性）。由于它是任何列的默认值，因此其存在不是必需的。




   
    
### 零列表

MAXIR 允许创建没有列的表（例如，`CREATE TABLE foo();`）。这是对 SQL 标准的扩展，标准不允许零列表。零列表本身并不是很有用，但禁止它们会为 `ALTER TABLE DROP COLUMN` 创建奇怪的特殊情况，所以 MAXIR 决定忽略这个规范限制。


 

### 多标识列
    
MAXIR 允许一个表有多个标识列。标准规定一个表最多可以有一个标识列。这主要是为了提供更多的灵活性来进行模式更改或迁移。注意，`INSERT` 命令只支持应用于整个语句的一个覆盖子句；多个标识列具有不同行为的支持不是很好。



 
    
### LIKE 子句

虽然 SQL 标准中存在 `LIKE` 子句，但许多 MAXIR 为其接受的选项并不在标准中，一些标准的选项未由 MAXIR 实现。
    
 

### WITH 子句

`WITH` 子句是 MAXIR 扩展；存储参数和 OID 都不在标准中。


 
    
### 数据分布

MAXIR 的并行或分布式数据库概念不是 SQL 标准的一部分。`DISTRIBUTED` 子句是扩展。


 

### 数据生命周期管理

`TTL` 子句是 MAXIR 对 SQL 标准的扩展。