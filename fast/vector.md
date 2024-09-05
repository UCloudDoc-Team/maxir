# MAXIR 向量版
## 1.创建集群
如您想详细了解如何创建集群，请参考 [创建集群](/maxir/guides/dps-clusters/manage-dps-clusters#创建-dps-集群) 。

## 2.创建用户
如您想详细了解如何创建用户，请参考 [创建用户](/maxir/guides/dw-users/manage-dwusers#邀请创建数据库用户) 。

## 3.访问集群
如您想详细了解如何访问集群，请参考 [访问集群](/maxir/guides/dps-clusters/access-dps-clusters) 。

## 4.索引创建

索引创建（Indexing）是构建数据结构以实现高效搜索的过程。Vector DPS 支持三种索引算法：暴力算法（Flat）、倒排文件索引（IVF）和分层可导航小世界（HNSW）。默认的算法是 HNSW。关于算法的详细说明，请参考 [索引算法说明](#索引算法说明)。

如下 SQL 命令创建了一个名为 `test_tbl` 的表，表中有一个类型为 `vector(3)` 的名为 `val` 的列，并为该表以平方欧式距离创建了一个向量索引。

```sql
CREATE TABLE test_tbl(val vectors.vector(3)) USING heap;
INSERT INTO test_tbl (val) SELECT ARRAY[random(), random(), random()]::real[] FROM generate_series(1, 1000);
CREATE INDEX ON test_tbl USING vectors (val vector_l2_ops);
```

其中 `vector_l2_ops` 运算符类计算平方欧式距离。更多信息，请参考手册中的“[运算符类列表](#运算符类列表)”章节。

如需将索引算法从默认的 HNSW 切换到 IVF，可以执行如下 SQL 命令。更多示例，请参考 [示例](#示例)。

```sql
CREATE INDEX ON test_tbl USING vectors (val vector_l2_ops)
WITH (options = $$
[indexing.ivf]
$$);
```

现在，您可以使用以下 SQL 进行 KNN 搜索，利用向量索引进行查询。

```sql
SELECT * FROM test_tbl ORDER BY val <-> '[0.5,0.5,0.5]' LIMIT 5;
```

info 说明:</br>
Vector DPS 的索引创建为异步模式。当向表中插入新行时，这些新行将先被放置在一个仅追加（Append-Only）文件中。后台线程会定期将该文件中新插入的行合并到现有的索引中。当用户执行查询操作时，会同时扫描该文件以确保准确性和一致性。


 

### 示例

```sql
SET search_path TO "$user", public, vectors;

-- 使用 HNSW 算法，默认设置。

CREATE INDEX ON test_tbl USING vectors (val vector_l2_ops);

-- 使用 FP16 向量类型来优化数据存储。

CREATE TABLE test_tbl (val vecf16(3)) USING heap;
CREATE INDEX ON test_tbl USING vectors (val vecf16_l2_ops)

--- 使用暴力法与乘积量化。

CREATE INDEX ON test_tbl USING vectors (val vector_l2_ops)
WITH (options = $$
[indexing.flat]
quantization.product.ratio = "x16"
$$);

-- 使用暴力法与标量量化。

CREATE INDEX ON test_tbl USING vectors (val vector_l2_ops)
WITH (options = "[indexing.hnsw.quantization.scalar]");

--- 使用 IVFPQ 算法。

CREATE INDEX ON test_tbl USING vectors (val vector_l2_ops)
WITH (options = $$
[indexing.ivf]
quantization.product.ratio = "x16"
$$);

-- 使用更多的线程来在后台构建索引。

CREATE INDEX ON test_tbl USING vectors (val vector_l2_ops)
WITH (options = $$
optimizing.optimizing_threads = 16
$$);

-- 偏好较小的 HNSW 图。

CREATE INDEX ON test_tbl USING vectors (val vector_l2_ops)
WITH (options = $$
segment.max_growing_segment_size = 200000
$$);
```

 

### 索引算法说明

选择适当的索引算法对于向量搜索的效率、资源利用率以及准确性等至关重要。Vector DPS 支持如下三种算法：

#### 暴力法（Flat）

当搜索次数有限，介于 1000 至 10,000 之间时，或者对搜索结果精确度有要求时，暴力法是一个合适的选择。

这种算法为其他索引方法设定了标准。它保留向量的原始形式，不进行压缩，避免了额外的开销。暴力法虽然直接且对用户友好，但并不适合处理大型数据集。

你也可以选择使用量化来减少内存使用。关于量化的详细说明，请参考 [参考手册](#表-quantization-支持的选项) 中的“表 `quantization` 支持的选项”章节。


#### 倒排文件索引（IVF）

IVF 主要使用倒排索引的思想，将向量 `(id，vector)` 存储在每个聚簇中心 (Cluster Center) 下。查询一个向量时，该算法会找到最近的几个中心，并分别搜索这些中心下的向量。

在搜索之前，使用 `K-Means` 聚簇和 `nsample` 点训练 `nlist` 聚簇中心。值越高，聚簇越精确，但需要更长的训练时间。更多配置信息，请参考 [参考手册](#表-ivf-支持的选项) 中的“表 `ivf` 支持的选项”章节。


#### 分层可导航小世界（HNSW）

HNSW（Hierarchical Navigable Small World）算法将跳跃列表（skip list）的概念与可导航小世界（Navigable Small World, NSW）图相结合，通过采用分层结构实现了高效的近似最近邻搜索。在这个结构中，上层具有较长的边，以便快速搜索，而下层则具有较短的边，以提高搜索的准确性。

算法的核心思想是保证每个节点都可以通过最少的“跳跃”从任何其他节点访问。在搜索过程中，从预定义的入口点开始，算法贪婪地移动到离查询向量更近的相邻节点。搜索会一直进行，直到找不到更近的节点为止。

在构建图时，通过调整参数 `m`，可以控制新节点与其最近邻居建立的连接数量。较高的 `m` 值会导致图更加密集，连接更多，但也会增加内存使用并减慢插入时间。在节点插入时，算法会找出最近的 `m` 个节点并与它们创建双向链接。

HNSW 算法适用于大规模数据集和高维度数据的近似最近邻搜索任务。其优势在于能够快速导航和搜索，并且具有较好的搜索准确性。与传统的 NSW 搜索相比，HNSW 在搜索过程中利用了分层结构，避免了陷入局部最优的问题，提高了搜索的效率和准确性。


---
## 5.向量搜索

先看一个示例，如下 SQL 获取了一个向量的最近的 5 个相邻节点：

```sql
SET vectors.hnsw_ef_search = 64;
SELECT * FROM test_tbl ORDER BY val <-> '[0.5,0.5,0.5]' LIMIT 5;
```


### 运算符

下表列出了 Vector DPS 支持的用于距离度量的运算符。

| 名称 | 描述 | 公式定义 |
| :- | :- | :- | 
| `<->` | 平方欧氏距离 | ![](1-1.png) |
| `<#>` | 负点积 | ![](1-2.png) |
| `<=>` | 余弦距离 | ![](1-3.png) |

<br/>

如下为使用示例：

```
-- 通过运算符调用距离函数

-- 平方欧氏距离
SELECT '[1, 2, 3]'::vector <-> '[3, 2, 1]'::vector;

-- 负点积
SELECT '[1, 2, 3]'::vector <#> '[3, 2, 1]'::vector;

-- 余弦距离
SELECT '[1, 2, 3]'::vector <=> '[3, 2, 1]'::vector;
```


### 过滤器

如下 SQL 获取了一个向量的指定类别的最近的 10 个相邻节点：

```
SELECT * FROM test_tbl ORDER BY val <-> '[0.5,0.5,0.5]' LIMIT 10;
SELECT * FROM test_tbl ORDER BY val <#> '[0.5,0.5,0.5]' LIMIT 10;
SELECT * FROM test_tbl ORDER BY val <=> '[0.5,0.5,0.5]' LIMIT 10;
```


### 查询项

Vector DPS 支持设置各种查询项。更多信息，请参考 [参考手册](#搜索选项) 中的“搜索选项”章节。

如下为两个简单示例：

示例 1：将会话中的 `ivf` 扫描列设置为 `1`。

```sql
SET vectors.ivf_nprobe=1;
```

示例 2：将事务中的 `hnsw` 搜索范围设置为 `40`。

```sql
SET LOCAL vectors.hnsw_ef_search=40;
```



### 搜索模式

Vector DPS 支持两种搜索模式：`vbase` 和 `basic`。

#### vbase

`vbase` 为默认模式，适用于大多数场景。

`vbase` 即使用 VBase 系统。该系统旨在有效地支持复杂的近似相似性搜索和关系操作查询。通过识别一种称为“松弛单调性 (Relaxed Monotonicity)”的共同属性，将两个看似不兼容的系统统一起来。这种共同属性允许 VBase 绕过仅支持 TopK 的界面的限制，从而实现更高的效率，并且据可证明地保留了基于 TopK 的解决方案的语义。通过这种方法，VBase 能够在复杂的在线向量查询中表现出高效性能，并且使得先前无法实现的分析相似性查询成为可能，同时还能够显著提高精确查询的速度和准确性。

#### basic

`basic` 在行为上与向量算法库（例如 faiss）一致。如果你希望和向量数据库一致的搜索行为，`basic` 模式是很好的选择。

caution `basic` 模式的使用限制
- 在进行搜索前，需要执行 `VACUUM` 操作。
- 搜索时不能带过滤器。




---

## 6.监控

Vector DPS 提供了一个名为 `pg_vector_index_stat` 的视图来监控索引进程。下表列出了该视图中的列及相关说明。

| 列名 | 类型 | 描述 |
| :- | :- | :- |
| `tablerelid` | `oid` | 表的 OID。 |
| `indexrelid` | `oid` | 索引的 OID。 |
| `tablename` | `name` | 表的名称。 |
| `indexname` | `name` | 索引的名称。 |
| `idx_status` | `text` | 索引的状态。可能值为 `NORMAL` 和 `UPGRADE`。<br/>`NORMAL` 表示索引正常，无需升级。`UPGRADE` 表示索引需要升级。 |
| `idx_indexing` | `bool` | 后台线程是否正在创建索引。<br/> 当 `idx_status` 为 `NORMAL` 时，则不为空。 |
| `idx_tuples` | `int8` | 元组的数量。<br/> 当 `idx_status` 为 `NORMAL` 时，则不为空。 |
| `idx_sealed` | `int8[]` | 每个封闭片段中的元组数量。 <br/> 当 `idx_status` 为 `NORMAL` 时，则不为空。 |
| `idx_growing` | `int8[]` | 每个正在增长的片段中的元组数量。<br/> 当 `idx_status` 为 `NORMAL` 时，则不为空。 |
| `idx_write` | `int8` | 写缓冲区中的元组数量。<br/> 当 `idx_status` 为 `NORMAL` 时，则不为空。 |
| `idx_size` | `int8` | 所有片段的字节大小。<br/> 当 `idx_status` 为 `NORMAL` 时，则不为空。 |
| `idx_options` | `text` | 索引的配置。<br/> 当 `idx_status` 为 `NORMAL` 时，则不为空。 |

<br/>

如果已经完成数据插入并且您在等待所有索引建立完成，您可以每隔 60 秒去监测 `idx_indexing` 参数是否变为 `true`。



---
## 7.量化

量化是一种通过可承受的精度降低来减少内存消耗和计算复杂度的技术。在向量搜索的背景下，量化方法可以分为两种类型：乘积量化和标量量化。


### 乘积量化

乘积量化是一种方法，它在数据集上训练几个聚类模型，并将每个向量映射到一组聚类中心索引。它常用于解决高维数据带来的过度内存使用问题。

在 Vector DPS 中，你可以启用乘积量化。例如：

```sql
CREATE INDEX ON test_tbl USING vectors (val vector_l2_ops)
WITH (options = $$
[indexing.ivf]
quantization.product.ratio = "x16"
$$);
```

更多配置，请参考 [参考手册](#表-product-支持的选项) 中的“表 `product` 支持的选项”章节。



### 标量量化

标量量化将每个值映射到低位格式，比如从 32 位浮点数到 8 位整数。标量量化通常不会影响精度，但可以减少内存使用和提高查询性能。Vector DPS 支持 `uint8` 标量量化，例如：

```sql
CREATE INDEX ON test_tbl USING vectors (val vector_l2_ops)
WITH (options = "[indexing.hnsw.quantization.scalar]");
```

### 使用建议

当需要使用量化时，如无特殊需求，建议先尝试标量量化。标量量化在精度和内存使用率之间实现了良好的平衡。

如果内存使用率仍然过高，可以尝试乘积量化。乘积量化能够显著减少内存使用率，但是可能会对精度造成影响。由于乘积量化与数据集高度相关，你需要尝试不同的比率和样本值，从而找到最适合你的数据集的方案。

此外，也可以通过使用 CPA 等降维方法来减少向量的维数。



---
## 8.与 `pgvector` 的兼容性

Vector DPS 在如下方面原生兼容 `pgvector`：

- `CREATE TABLE` 命令，例如 `CREATE TABLE test_tbl (val vector(3))`

- `INSERT INTO` 命令，例如 `INSERT INTO test_tbl (val) VALUES ('[0.6,0.6,0.6]')` 

通过 `SET vectors.pgvector_compatibility = on` 设置，Vector DPS 可以实现与 `pgvector` 在如下方面的兼容：

- 索引选项，从而可以使用 `USING hnsw (val vector_ip_ops)` 创建索引

- 查询选项，可以通过 `SET ivfflat.probes = 10` 实现

与 pgvector 的兼容性可以通过 `SET vectors.pgvector_compatibility = on` 开启。




### 索引选项和查询选项

`ivfflat` 索引支持如下索引选项：

| 键 | 类型 | 取值范围 | 默认值 | 说明 |
| :- | :- | :- | :- | :- |
| `nlist` | `integer` | `[1, 1_000_000]` | `100` | 聚簇单元的数量。 |

<br/>


`ivfflat` 索引支持如下查询选项：

| 选项 | 类型 | 取值范围 | 默认值 | 说明 |
| :- | :- | :- | :- | :- |
| `ivfflat.probes` | `integer` | `[1, 1_000_000]` | `10` | 需要扫描的列表数量。 |

<br/>

:::caution 重要提示
在 Vector DPS 中，`ivfflat.probes` 的默认值为 `10`，而不是 `pgvector` 中的 `1`。
:::


`hnsw` 索引支持如下索引选项：

| 键 | 类型 | 取值范围 | 默认值 | 说明 |
| :- | :- | :- | :- | :- |
| `m` | `integer` | `[4, 128]` | `16` | 节点的最大度数。 |
| `ef_construction` | `integer` | `[10, 2000]` | `64` | 构建中的搜索范围。 |

<br/>

`hnsw` 索引支持如下查询选项：

| 选项 | 类型 | 取值范围 | 默认值 | 说明 |
| :- | :- | :- | :- | :- |
| `hnsw.ef_search` | `integer` | `[1, 65535]` | `100` | HNSW 的搜索范围。 |

<br/>

caution 重要提示<br/>
在 Vector DPS 中，`hnsw.ef_search` 的默认值为 `100`，而不是 `pgvector` 中的 `40`。


<br/>

### 示例

如下示例展示了如何开启 `pgvector` 的兼容性设置并执行了一个向量查询：

```sql
DROP TABLE IF EXISTS test_tbl;
SET vectors.pgvector_compatibility=on;
SET hnsw.ef_search=40;
CREATE TABLE test_tbl (val vector(3)) USING heap;
INSERT INTO test_tbl (val) SELECT ARRAY[random(), random(), random()]::real[] FROM generate_series(1, 1000);
CREATE INDEX hnsw_cos_index ON test_tbl USING hnsw (val vector_cosine_ops);
SELECT COUNT(1) FROM (SELECT 1 FROM test_tbl ORDER BY val <-> '[0.5,0.5,0.5]' limit 100) test_tbl_2;
DROP INDEX hnsw_cos_index;
```

支持多种索引共存：

```sql
SET vectors.pgvector_compatibility=on;
SET hnsw.ef_search=80;
-- [hnsw + vector_l2_ops] 索引，使用默认配置
CREATE INDEX hnsw_l2_index ON test_tbl USING hnsw (val vector_l2_ops);
-- [hnsw + vector_cosine_ops] 索引，仅配置了 `ef_construction` 选项
CREATE INDEX hnsw_cosine_index ON test_tbl USING hnsw (val vector_cosine_ops) WITH (ef_construction = 160);
-- 匿名 [hnsw + vector_ip_ops]，包含所有选项
CREATE INDEX ON test_tbl USING hnsw (val vector_ip_ops) WITH (ef_construction = 64, m = 8);
SET ivfflat.probes=1;
-- [ivfflat + vector_l2_ops] 索引，使用默认配置
CREATE INDEX ivfflat_l2_index ON test_tbl USING ivfflat (val vector_l2_ops);
-- [ivfflat + vector_ip_ops] 索引，包含所有选项
CREATE INDEX ivfflat_ip_index ON test_tbl USING ivfflat (val vector_cosine_ops) WITH (nlist = 40);
-- 匿名 [ivf + vector_ip_ops]，包含所有选项
CREATE INDEX ON test_tbl USING ivfflat (val vector_ip_ops) WITH (lists = 40)
```
## 9.参考手册

---

### 运算符类列表

| 访问方式 | 输入类型 | 运算符类 | 是否有默认值 | 说明 |
| :- | :- | :- | :- | :- |
| `vectors` | `vector` | `vector_l2_ops` | 否 | 平方欧氏距离。 |
| `vectors` | `vector` | `vector_dot_ops` | 否 | 负点积。 |
| `vectors` | `vector` | `vector_cos_ops` | 否 | 余弦距离。 |
| `vectors` | `vecf16` | `vecf16_l2_ops` | 否 | 平方欧氏距离。 |
| `vectors` | `vecf16` | `vecf16_dot_ops` | 否 | 负点积。 |
| `vectors` | `vecf16` | `vecf16_cos_ops` | 否 | 余弦距离。 |

<br/>

---

### 索引选项

Vector DPS 使用 [TOML 语法](https://toml.io/en/v1.0.0) 来表达索引的配置。以下是配置中每个键所代表的含义：

| 键 | 类型 | 说明 |
| :- | :- | :- |
| `segment` | `table` | 分段（Segment）选项。 |
| `optimizing` | `table` | 后台优化选项。 |
| `indexing` | `table` | 用于索引创建的算法。 |

<br/>

---

#### 表 `segment` 支持的选项

| 键 | 类型 | 取值范围 | 默认值 | 说明 |
| :- | :- | :- | :- | :- |
| `max_growing_segment_size` | `integer` | [1, 4_000_000_000] | `20_000` | 未创建索引的向量的最大大小。 |
| `max_sealed_segment_size` | `integer` | [1, 4_000_000_000] | `1_000_000` | 用于索引创建的向量的最大大小。 |

<br/>

---

#### 表 `optimizing` 支持的选项


| 键 | 类型 | 取值范围 | 默认值 | 说明 |
| :- | :- | :- | :- | :- |
| `optimizing_threads` | `integer` | [1, 65535] | Core 的数量的平方根 | 索引的最大线程数。 |
| `sealing_secs` | `integer` | [1, 60] | `60` | 详见表格底部说明。 |
| `sealing_size` | `integer` | [1, 4_000_000_000] | `1` | 详见表格底部说明。 |

<br/>

info 说明<br/>
如果写入段大于 `sealing_size`，且在 `sealing_secs` 秒内不接受新数据，该写入段将被转换为封闭段。


<br/>

#### 表 `indexing` 支持的选项

| 键 | 类型 | 说明 |
| :- | :- | :- |
| `flat` | `table` | 如果设置了该表，索引将使用暴力算法。 |
| `ivf` | `table` | 如果设置了该表，索引将试用 IVF 算法。 |
| `hnsw` | `table` | 如果设置了该表，索引将试用 HNSW 算法。|

<br/>

### 表 `flat` 支持的选项

| 键 | 类型 | 说明 |
| :- | :- | :- |
| `quantization` | `table` | 用于量化的算法。 |

<br/>

#### 表 `ivf` 支持的选项

| 键 | 类型 | 取值范围 | 默认值 | 说明 |
| :- | :- | :- | :- | :- |
| `nlist` | `integer` | [1, 1_000_000] | `1000` | 聚簇单元的数量。 |
| `nsample` | `integer` | [1, 1_000_000] | `65536` | K-Means 聚簇的样本数量。 |
| `least_iterations` | `integer` | [1, 1_000_000] | `16` | K-Means 聚簇的最小迭代次数。 |
| `iterations` | `integer` | [1, 1_000_000] | `500` | K-Means 聚簇的最大迭代次数。 |
| `quantization` | `table` | N/A | N/A | 使用的量化算法。 |

<br/>

#### 表 `hnsw` 支持的选项

| 键 | 类型 | 取值范围 | 默认值 | 说明 |
| :- | :- | :- | :- | :- |
| `m` | `integer` | [4, 128] | `12` | 节点的最大度数。 |
| `ef_construction` | `integer` | [10, 2000] | `300` | 构建中的搜索范围。 |
| `quantization` | `table` | N/A | N/A | 使用的量化算法。 |

<br/>


#### 表 `quantization` 支持的选项

| 键 | 类型  | 说明 |
| :- | :- | :- |
| `trivial` | `table` | 如果设置了该表，不使用量化。 |
| `scalar` | `table` | 如果设置了该表，使用标量量化。 |
| `product` | `table` | 如果设置了该表，使用乘积量化。 |

<br/>

#### 表 `product` 支持的选项

| 键 | 类型 | 取值范围 | 默认值 | 说明 |
| :- | :- | :- | :- | :- |
| `sample` | `integer` | [1, 1_000_000] | `65535`| 用于量化的样本。 |
| `ratio` | `enum` | "x4", "x8", "x16", "x32", "x64" | `"x4"` | 量化的压缩比。 |

<br/>

info 说明<br/>
压缩比指定了向量压缩后的空间占用与向量压缩钱的空间占用的比例。例如，`"x4"` 表示 1/4。如果索引的列的类型是 `vecf16`，那么 `"x4"` 则表示 1/2。


---
### 搜索选项

搜索选项通过 [PostgreSQL 的全局统一配置 (Grand Unified Configuration, GUC)](https://www.postgresql.org/docs/current/config-setting.html) 来指定。

<br/>

#### `ivf` 支持的选项

如下选项仅在索引使用 IVF 算法时有效。

| 键 | 类型 | 取值范围 | 默认值 | 说明 |
| :- | :- | :- | :- | :- |
| `vectors.ivf_nprobe` | `integer` | [1, 1_000_000] | `10`| 需要扫描的列表数量。 |

<br/>

#### `hnsw` 支持的选项

如下选项仅在索引使用 HNSW 算法时有效。

| 键 | 类型 | 取值范围 | 默认值 | 说明 |
| :- | :- | :- | :- | :- |
| `vectors.hnsw_ef_search` | `integer` | [1, 65535] | `100`| HNSW 算法搜索的候选结果集大小。 |

<br/>

#### 其他选项

Vector DPS 提供如下针对搜索模式的搜索选项：

| 键 | 类型 | 取值范围 | 默认值 | 说明 |
| :- | :- | :- | :- | :- |
| `vectors.search_mode` | `enum` | `"basic"`, `"vbase"` | `"vbase"` | 搜索模式。 <br/>更多信息，请参考 [使用指南](use.md#向量搜索)。 |
| `vectors.pgvector_compatibility` | `boolean` |   | `off` | 是否开启兼容。 <br/>更多信息，请参考 [使用指南](use.md#与-pgvector-的兼容性)。 |
| `vectors.enable_index` | `boolean` |   | `on` | 是否生产索引查询的执行计划。 |

<br/>
