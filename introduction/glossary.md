# 基础概念

**DWSU**<br />
数据仓库服务单元（数仓服务单元或服务单元），是 MAXIR 的基本部署单元。它兼容 PostgreSQL 协议，支持通过 PostgreSQL 兼容的客户端进行连接，例如 `psql`，也可以使用 MAXIR 提供的统一控制台进行管理。在数仓服务单元内，数仓用户可以进行创建表、写入数据、加载数据和执行查询等一系列数仓操作。
在 MAXIR 中，你可以按需创建多个服务单元，并将这些服务单元部署在不同地域中，从而实现全球化部署。
<br />
<br />

**DPS**<br />
DPS（Data Processing Service ) ，是 MAXIR 提供的计算引擎。它是数据处理的核心，负责运行查询和计算任务。
<br />
<br />

**集群**<br />
集群，也称 DPS 集群，是由 CPU、内存和 I/O 资源组成的集合的规范形式，创建于数仓服务单元内，对于在 MAXIR 上运行查询和 DML 操作至关重要。执行查询时，必须选择一个 DPS 集群。DPS 集群根据使用的计算引擎类型可以分为三类：Hybrid DPS 集群、Extreme DPS 集群、Vector DPS 集群，下表对三种类型的 DPS 集群进行了比较：

| **对比项** | **Hybrid DPS 集群** | **Extreme DPS 集群** | **Vector DPS 集群** |
| --- | --- | --- | --- |
| 计算引擎类型 | Hybrid DPS | Extreme DPS | Vector DPS |
| 产品版本 | MAXIR 标准版 | MAXIR 标准版 | MAXIR 向量版 |
| 每个服务单元可创建数量 | 1 | 多个，按需创建 | 1 |
| 计算规格 | [参考计量规则</font>](https://docs.ucloud.cn/maxir/buy/charge?id=%e8%ae%a1%e9%87%8f%e8%a7%84%e5%88%99) | [参考计量规则</font>](https://docs.ucloud.cn/maxir/buy/charge?id=%e8%ae%a1%e9%87%8f%e8%a7%84%e5%88%99) | [参考计量规则</font>](https://docs.ucloud.cn/maxir/buy/charge?id=%e8%ae%a1%e9%87%8f%e8%a7%84%e5%88%99) |
| 删除 | 不支持 | 支持 | 不支持 |
| 扩缩容 | 支持 | 支持 | 即将支持 |
| 手动暂定 | 支持 | 支持 | 不支持 |


**Hybrid DPS**<br />
Hybrid DPS，通用计算引擎，高度兼容 PostgreSQL 的 SQL 功能，在性能和成本之间实现了最佳平衡。
<br />
<br />

**Extreme DPS**<br />
Extreme DPS，高速版计算引擎，性能相比 Hybrid DPS 有显著提升，非常适合高实时性要求的工作负载。
<br />
<br />

**VectorDPS**<br />
Vector DPS 是由 MAXIR 提供的专⻔针对⾮结构化数据存储、检索和分析的 Embedding 向量存储与计算引擎，致⼒于帮助⽤⼾解锁 AIGC（Artificial Intelligence Generated Content，⼈⼯智能⽣成内容）、RAG（Retrieval Augmented Generation，检索增强⽣成）和 AI 智能代理等应⽤场景，从⽽帮助⽤⼾构建适应 AIGC 时代的数据基础设施。其主要特点包括： 
- **⾼维向量实时化**：⽀持⾼达 5000 维的向量数据，提供即时的增删改查能⼒ 
- **融合检索**：⽀持多条件灵活组合，提供超过 99% 的检索准确率 
- **⾼吞吐⾼并发**：超过 100 万 TPS 的写⼊速度和超过 1 万 QPS 的查询速度，满⾜⾼吞吐与⾼并发的需求 
- **便宜易⽤**：提供参数⾃调优能⼒；按需计费
<br />
<br />

**数据库**<br />
数据库是一个有组织地存储数据的集合，它按照特定的数据结构和规则进行组织，以便高效地存储、管理和检索数据。在MAXIR中，所有数仓用户都可以创建数据库，但只有数据库的所有者才能删除它。请注意，一旦数据库被删除，其所有目录条目及包含的数据都会被一并删除。
<br />
<br />

**数仓用户**<br />
数仓用户是指登录数据库账号在数仓服务单元中操作和使用数据库对象的实体。他们可以查看服务单元中的全部数据库对象，并对特定数据库对象拥有创建、管理、使用等权限。
<br />
<br />

**Schema**<br />
Schema 用于组织和管理数据库对象。每个数据库默认有个名为 `public` 的Schema，所有用户都可在其中创建数据库对象，这些数据库对象对所有用户可见，但只能由其所有者修改或删除。Schema 只能由数据库所有者创建，且只能由 Schema 的所有者修改或删除。
<br />
<br />


**表、视图、函数**<br />
表、视图和函数是数据库中的核心对象，用于存储、管理和处理数据。这些对象共同构成了数据库的基础，帮助组织和操作数据。
- **表**：数据库中用于存储结构化数据的基本单位，数据以行和列的形式组织。MAXIR 中的表包括两种类型：
  - **内部表**：数据存储在 MAXIR 中，支持所有 MAXIR 支持的数据类型。
  - **外部表**：不在 MAXIR 中存储数据，只进行字段映射。外部表的数据是只读的，无法执行 DML 语句或创建索引。
- **视图**：数据库中的虚拟表，通过查询一个或多个表生成的数据集，便于数据访问和管理。
- **函数**：存储在数据库中的代码，用于执行特定操作或计算，通常用于数据处理和转换。
只有这些对象（包括表、视图、函数及存储过程）的所有者才有权限进行修改或删除操作。
<br />
<br />

**权限**<br />
权限是数据库中用于控制用户操作权限的机制，它决定了数仓用户可以执行哪些操作。权限可以被授予给用户，从而允许他们对特定的数据库对象（如表、视图、Schema 等）执行特定操作。关于对象权限的详细信息，请参考 [对象权限](https://docs.ucloud.cn/maxir/guides/security/object-privileges)。
<br />
<br />
