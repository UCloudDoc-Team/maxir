# 产品架构

![](/images/introduction/architecture.png)


Relyt 的核心概念和产品特征对应上图自下而上分层：

- **多地域**：用户可以按照自己的需求选择地域创建数仓服务单元，不同的数仓服务单元可以部署在不同的地域中。数仓服务单元是一个独立数据仓库的服务实体，包含近乎无限可扩展的计算服务 DPS （Data Processing Service，数据处理服务），无限的存储空间（云对象存储）。

- **基于一份数据的存储与计算分离**：Relyt 数据存储在云平台的对象存储中，具有无限的数据存储能力，用户无需进行存储容量规划和存储扩、缩容操作；一份数据支撑实时、Ad-hoc 交互式和批量 ETL 等多种计算负载，无数据冗余；存储按实际用量计费，没有任何空间浪费。

- **多 DPS 集群**：DPS 集群是 Relyt 数仓服务单元的计算集群。DPS 集群分为多种类型：Hybrid DPS 集群具备兼容 PostgreSQL 功能的数据仓库服务能力；Extreme DPS 集群提供比 Hybrid DPS 集群高出数十倍的性能优势，但是功能上是 Hybrid DPS 集群的子集。二者之间的具体差异，请参考 Relyt 的 SQL 手册和函数手册；Vector DPS 集群还在研发中。Hybrid DPS 集群目前在一个数仓服务单元中有且仅有一个，Extreme DPS 集群支持按需创建多个，DPS 集群之间完全物理隔离，保障负载之间无干扰。

- **生态兼容易用**：Relyt 兼容 PostgreSQL 接口协议，使用 PostgreSQL 的生态工具即可无缝连接和使用 Relyt。同时，Relyt 提供的 Open API 方便用户进行产品和应用开发与集成。