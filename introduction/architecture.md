# 产品架构

![](/images/introduction/architecture.png)
MAXIR 的架构分为三层，自下而上对应不同的核心概念和产品特征：

**数据层：存储与计算分离**
- 数据存储在云平台的对象存储 US3 中，具备无限存储能力。存储和计算分离的架构使得同一份数据可以支持多种计算负载，如实时处理、Ad-hoc 交互查询、批量 ETL 等，无需数据冗余。
- NDP（Near Data Processing，近数据处理）在数据源附近进行计算，减少数据传输延迟，提供弹性扩容，支持几乎无限的计算能力扩展。

**计算层：多 DPS 集群**
- Hybrid DPS 集群：兼容 PostgreSQL 功能，提供通用的数据仓库服务，在一个数仓服务单元中有且仅有一个。
- Extreme DPS 集群：相比 Hybrid DPS 集群，性能提升数十倍，但功能有所精简，适用于高性能要求的应用场景，支持按需创建。两者差异详细参考 [DPS集群对比](/maxir/introduction/glossary?id=集群) 。
- Vector DPS 集群：Vector DPS 集群是专用于处理向量化工作负载的计算集群，分为性能优化和容量优化两种类型。

**业务层：接口与控制**
- 提供网页控制台、数据库连接、Open API/Terraform 等多种管理和集成方式。
- 兼容 PostgreSQL 协议，使用 PostgreSQL 的生态工具即可与现有生态系统集成。

