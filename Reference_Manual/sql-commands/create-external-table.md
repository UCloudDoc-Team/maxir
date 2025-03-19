# CREATE EXTERNAL TABLE

创建外部表。

---

## 语法

```sql
CREATE [READABLE | WRITABLE] EXTERNAL TABLE <table_name>     
    ( <column_name> <data_type> [, ...] | LIKE <other_table >)
    LOCATION ('s3://<s3_endpoint>/<bucket_name>/[<s3_prefix>] 
       accessid=<access_key_id>
       secret=<access_secret>
       [threadnum=<thread_number>]
       [chunksize=<chunk_size>]')
     FORMAT 'TEXT' 
           [( [HEADER]
              [DELIMITER [AS] '<delimiter>' | 'OFF']
              [NULL [AS] '<null string>']
              [ESCAPE [AS] '<escape>' | 'OFF']
              [FILL MISSING FIELDS] )]
          | 'CSV'
           [( [HEADER]
              [QUOTE [AS] '<quote>'] 
              [DELIMITER [AS] '<delimiter>']
              [NULL [AS] '<null string>']
              [FORCE NOT NULL <column> [, ...]]
              [ESCAPE [AS] '<escape>']
              [FILL MISSING FIELDS] )]
          | 'PARQUET'
          | 'ORC'
```

---
## 描述

`CREATE EXTERNAL TABLE` 或 `CREATE EXTERNAL WEB TABLE` 在 MAXIR 中创建一个新的可读外部表定义。可读外部表通常用于快速、并行的数据加载。一旦定义了外部表，你可以直接（并行地）使用 SQL 命令查询其数据。例如，你可以对外部表数据进行选择、连接或排序。你还可以为外部表创建视图。不允许在可读外部表上进行 DML 操作（`UPDATE`、`INSERT`、`DELETE` 或 `TRUNCATE`），也不能在可读外部表上创建索引。


---
## 参数

- **`[READABLE | WRITABLE]`**
    
    外部表类别。如不指定，则使用默认值 `READABLE`。

    - `READABLE`：可读外部表用于导入数据至 MAXIR， 为默认值。
    - `WRITABLE`：可写外部表用于卸载（Unload）数据。

- *`<table_name>`*

    外部表的名称。

- *`<column_name>`*

    列的名称。 

    不要为外部表中的列指定列约束和默认值，因为外部表不支持列约束和默认值。

- *`<data_type>`*

    列的数据类型。

- **`LOCATION`**

     外部数据源的位置。目前，只支持来自 AWS S3 的数据源。

- *`<s3_endpoint>`*

    连接到外部数据源的端点。支持的端点包括：

    - Amazon S3 端点

        详情请参阅 Amazon S3 文档中的 [Amazon Simple Storage Service endpoints and quotas](https://docs.aws.amazon.com/general/latest/gr/s3.html)。

    - 金山云 KS3 端点

        详情请参阅金山云 KS3 文档中的 [Endpoint与Region的对应关系](https://docs.ksyun.com/documents/6761)。

    - 阿里云 OSS 端点

        详情请参阅阿里云 OSS 文档中的 [访问域名和数据中心](https://help.aliyun.com/zh/oss/user-guide/regions-and-endpoints)。

    - 腾讯云 COS 端点

        详情请参阅腾讯云 COS 文档中的 [地域和访问域名](https://cloud.tencent.com/document/product/436/6224)。

- *`<bucket_name>`*

    S3 桶的名称。

- *`<s3_prefix>`*

    S3 桶内的可选路径前缀。指定后，只有这个路径前缀下的对象才会被读取。

- **`accessid`**

    AWS Access ID。

- **`secret`**

    AWS Access Secret。

- **`threadnum`**

    在传输数据时使用的并行线程数。取值范围为 1 到 8。`TEXT` 和 `CSV` 的默认值为 4，`PARQUET` 的默认值为 1。

- **`chunksize`**

    传输的块大小，单位为字节。取值范围为 1 MB 到 128 MB，默认值为 8 MB。

- **`FORMAT`**

    数据格式。支持的选项包括 **TEXT**、**CSV**、**PARQUET** 和 **ORC**。

- **`DELIMITER`**

    用于分隔每行数据中的列的单个 ASCII 字符。`TEXT` 的默认值是制表符，`CSV` 的默认值是英文逗号（,）。对于 `TEXT`，你可以将 `DELIMITER` 设置为 `OFF`，以便在将非结构化数据加载到单列表的特殊用例中。

    对于 `s3` 协议，分隔符不能是换行符（`\n`）或回车符（`\r`）。

- **`NULL`**

    表示 `NULL` 值的字符串。`TEXT` 模式下的默认值是 `\N`（反斜杠-N），`CSV` 模式下的默认值是无引号的空值。即使在 `TEXT` 模式下，你可能也更喜欢空字符串，对于你不希望区分 `NULL` 值和空字符串的情况。当使用外部和 web 表时，任何与此字符串匹配的数据项都将被视为 `NULL` 值。
    
    以下为 `text` 格式的举例，这个 `FORMAT` 子句可以用来指定两个单引号的字符串（`''`）是 `NULL` 值。

    ```sql
    FORMAT 'text' (delimiter ',' null '\'\'\'\'' )
    ```

- **`ESCAPE`**

    用于 C 风格转义序列的单个字符，如 `\n`，`\t` 或 `\100`，以及用于转义可能被误认为行或列分隔符的数据字符。确保选择一个在你实际的列数据中没有使用的转义字符。文本格式文件的默认转义字符是反斜杠（`\`），csv 格式文件的默认转义字符是双引号（`"`），但是可以指定另一个字符来表示转义。在文本格式文件中，也可以通过指定 `'OFF'` 作为转义值来停用转义。这对于包含许多嵌入的反斜杠（并非都是转义）的文本格式的 web 日志数据非常有用。

- **`HEADER`**

    对于可读外部表，指定数据文件中的第一行是表头行（包含表列的名称），不应将其包含为表的数据。如果使用多个数据源文件，所有文件都必须有表头行。

    对于 `s3` 协议，表头行中的列名不能包含换行符（`\n`）或回车符（`\r`）。

- **`QUOTE`**

    `CSV` 模式的引用字符。默认是双引号（`"`）。

- **`FORCE NOT NULL`**

    在 `CSV` 模式下，将每个指定的列处理为引用的，因此不是 `NULL` 值。对于 `CSV` 模式的默认空字符串（两个分隔符之间的无内容），这会导致丢失的值被评估为零长度字符串。

- **`FILL MISSING FIELDS`**

    在可读外部表的 `TEXT` 和 `CSV` 模式下，指定 `FILL MISSING FIELDS` 会在一个行的数据在行或行的末尾缺少数据字段时将丢失的尾随字段值设置为 `NULL`（而不是报告错误）。空行、带有 `NOT NULL` 约束的字段、以及行上的尾随分隔符仍然会产生报错。

---

## 示例

使用 `s3` 协议创建名为 `nation_ext` 的外部表：

```sql
CREATE EXTERNAL TABLE nation_ext (
    n_nationkey  integer,
    n_name       char(25),
    n_regionkey  integer,
    n_comment    varchar(152),
    n_dummy      varchar
)
LOCATION ('s3://s3.ap-southeast-1.amazonaws.com/******/tpch/100m/nation.tbl 
          accessid=************
          secret=************'
          )
FORMAT 'csv' (delimiter '|');
```

---

## SQL 标准兼容性

`CREATE EXTERNAL TABLE` 是 MAXIR 的扩展。
