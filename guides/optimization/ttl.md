
# 使用 TTL 管理数据


Relyt 的 TTL（Time-to-Live）特性支持用户在建表时，通过指定 `TTL` 子句来自定义数据的冷温热分层策略，删除策略，以及组合使用多种策略实现更为复杂的数据生命周期管理。

---
## 核心概念

- **热数据**：被频繁访问或者更新的数据，适于存储在缓存中，支持高效访问。

- **冷数据**：较少被访问或只在特定时间被访问的数据，更适合存储在云存储中，存储成本低。

- **热表**：即冷温热属性为 **`HOT`** 的表。热表中所有数据常驻于缓存中，不会随着时间的流逝而转冷。

- **温表**：即冷温热属性为 **`WARM`** 的表。温表支持指定整表数据或者部分行数据为热数据，存储在缓存中。温表中的热数据有有效期，通过 TTL 表达式指定。当温表中的热数据过期后，数据转冷，从缓存中删除。



---

## 为表设置分层存储策略

在大多数场景中，数据的访问模式通常呈现出热度的差异。一部分数据（热数据）被频繁地访问和更新，而另一部分数据（冷数据）则较少被访问或者只在特定的时间段内被访问。Relyt 支持使用 TTL 为表设置分层存储策略，从而实现对表中数据进行冷温热分层管理，在保证数据访问性能的同时，降低存储成本。

 

### 语法

```sql
TTL [HOT | <ttl_expression> WARM ]
```

 

### 参数

- **`HOT`** 或 **`WARM`**

    数据的冷温热属性。 
    
    - **`HOT`**：表示该表为热表，表中所有数据全部常驻于缓存中，不会转冷。

    - **`WARM`**：表示该表为温表，表中数据会依据 *`<ttl_expression>`* 指定的冷温热策略转冷。转冷后的数据将从缓存中删除。

- *`<ttl_expression>`*

    TTL 表达式，支持具体时间也支持相对时间，指定了数据在缓存中的驻留时长。表达式中引用的列类型只能为 `date`、`varchar` 和 `integer`。并且，如引用的列类型为非时间类型，需要通过函数转换为时间类型。

    :::warning 重要
    *`<ttl_expression>`* 仅在数据冷温热属性为 `WARM` 时需要指定，且必须指定。
    :::


 


### 使用说明

`WARM` 属性可以在表级和行级生效。如需配置 `WARM` 属性在行级生效，则需保证如下两个条件同时满足：

- 建表语句中使用了 `CLUSTER BY` 子句，且该子句的第一列为 `date`、`varchar` 或 `integer` 类型。

- TTL 表达式中引用的列为 `CLUSTER BY` 子句中指定的第一列。


对于频繁访问的表或者用于存储实时数据或者业务决策关键数据的表，推荐使用 `TTL HOT`，使表中数据常驻于缓存中。


关于完整的 `CREATE TABLE` 的语法及详细说明，请参考 [CREATE TABLE](reference/sql-commands/create-table.md)。


 


### 示例



如下 SQL 示例创建了一个名为 `test_hot` 的热表。该表中的所有用户数据均常驻于缓存中，保证了查询的稳定性和响应的实时性。

```sql
CREATE TABLE test_hot (column integer) TTL HOT;
```


如下 SQL 分别创建了一个名为 `test_warm1` 和 `test_warm2` 的温表，当达到指定时间点后，整个表将转冷，从缓存中移除。

```sql
-- 2024 年 1 月 1 日前，数据常驻缓存，过期后转冷
CREATE TABLE test_warm1 (col1 integer) 
TTL (('2024-01-01'::date)) WARM;

-- 从建表日期开始 7 天之内，数据常驻缓存，过期后转冷
CREATE TABLE test_warm2 (col1 integer) 
TTL ((now() + interval '7 days')) WARM;
```

如下 SQL 创建了一个名为 `test_warm3` 的表，以时间列 `ds` 为基准，表中数据存活超过 7 天的行将转冷。

```sql
CREATE TABLE test_warm3 (ds date, content varchar) 
CLUSTER BY (ds) 
TTL ((ds + interval '7 days')) WARM;

INSERT INTO test_warm3 VALUES ('2024-01-01', 'abc'); -- 该条数据将于 2024 年 1 月 8 日取消常驻缓存
INSERT INTO test_warm3 VALUES ('2024-01-02', 'abc'); -- 该条数据将于 2024 年 1 月 9 日取消常驻缓存
```

---
## 为表设置删除策略

Relyt 支持通过 TTL 特性为表设置删除策略。删除策略可以指定为表级和行级生效。

 

### 语法

```sql
TTL <delete_expression> DELETE
```

 

### 参数

- *`<delete_expression>`*

    数据的删除策略表达式，与 **DELETE** 关键词一起结合使用。

- **`[DELETE]`**

    删除关键词，与 *`<delete_expression>`* 表达式一起结合使用。

 

### 使用说明

删除策略可以在表级和行级生效。如需配置删除策略在行级生效，则需保证如下两个条件同时满足：

- 建表语句中使用了 `CLUSTER BY` 子句，且该子句的第一列为 `date`、`varchar` 或 `integer` 类型。

- 删除表达式中引用的列为 `CLUSTER BY` 子句中指定的第一列。

关于完整的 `CREATE TABLE` 的语法及详细说明，请参考 [CREATE TABLE](reference/sql-commands/create-table.md)。


 

### 示例

如下 SQL 创建了一个名为 `test_1` 的普通表，并设置整个表的删除策略为在 2024 年 1 月 1 日删除。

```sql
CREATE TABLE test_1 (col1 integer) TTL (('2024-01-01'::date)) DELETE;
```


如下 SQL 创建了一个名为 `test_2` 的普通表，并设置整个表的删除策略为建表 30 天后删除。

```sql
CREATE TABLE test_2 (col1 integer) TTL ((now() + interval '30 days')) DELETE;
```

如下 SQL 创建了一个名为 `test_3` 的普通表，并以时间列 `ds` 为基准，表中数据行存活超过 7 天后将会被删除。

```sql
CREATE TABLE test_3 (ds date, content varchar) 
CLUSTER BY (ds) 
TTL ((ds + interval '7 days')) DELETE;

INSERT INTO test_3 VALUES ('2024-01-01', 'abc'); -- 该条数据将于 2024 年 1 月 8 日删除
INSERT INTO test_3 VALUES ('2024-01-02', 'abc'); -- 该条数据将于 2024 年 1 月 9 日删除
```

---

## 为表同时设置冷温热分层和删除策略

Relyt 的 TTL 特性允许同时为表配置分层存储策略和删除策略，从而实现更为灵活、复杂的数据生命周期管理。

<br/>

**为热表设置删除策略**

如下 SQL 创建了一个名为 `test_hot1` 热表，并指定表以 `ds` 列为基准， 7 天后删除。

```sql
CREATE TABLE test_hot1 (ds date, content varchar) 
CLUSTER BY (ds) 
TTL HOT, ((ds + interval '7 days')) DELETE;
```

<br/>

**为温表设置删除策略**

如下 SQL 创建了一个名为 `test_warm4` 的温表，并指定表中数据行以 `ds` 列为基准，3 后转冷，7 天后删除。

```sql
CREATE TABLE test_warm4 (ds date, content varchar) 
CLUSTER BY (ds) 
TTL ((ds + interval '3 days')) WARM, ((ds + interval '7 days')) DELETE;
```



