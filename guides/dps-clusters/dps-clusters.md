# DPS 集群

## 什么是 DPS？

Data Processing Service (DPS) 是 MAXIR 提供的计算引擎。它是数据处理的核心，负责运行查询和计算任务。DPS 有两种类型： 

- Hybrid DPS：通用版。Hybrid DPS 高度兼容 PostgreSQL 的 SQL 功能。

- Extreme DPS：高速版。相较于 Hybrid DPS，Extreme DPS 拥有数量级的性能优势，不过提供的功能集相对较小。


在创建 DPS 集群时，你可以选择所需的类型。

## 什么是 DPS 集群？

DPS 集群是由 CPU、内存和 I/O 资源组成的集合的规范形式。DPS 集群根据使用的计算引擎类型可以分为三类：即 Hybrid DPS 集群、Extreme DPS 集群和 Vector DPS 集群。

DPS 集群是在数仓服务单元内创建的资源，对于在 MAXIR 上运行查询和 DML 操作至关重要。执行查询时，必须选择一个 DPS 集群。

## Hybrid DPS 集群

目前，创建数仓服务单元时，必须创建一个 Hybrid DPS 集群。Hybrid DPS 集群不能被删除，如无需使用，可手动暂停。有关如何暂停 Hybrid DPS 集群的详细信息，请参见 [管理 DPS 集群](guides/dps-clusters/manage-dps-clusters.md#手动暂停dps集群) 中的 "手动暂停 DPS 集群"。

## Extreme DPS 集群

在一个数仓服务单元中，你可以创建多个 Extreme DPS 集群来运行不同的工作负载。MAXIR 为你提供丰富的功能，以便你管理 Extreme DPS 集群，以更好地适应你不同的业务需求。更多相关信息，请参见 [管理 DPS 集群](manage-dps-clusters.md)。

## DPS 集群比较

下表对两种类型的 DPS 集群进行了比较。

| 比较项                 | Hybrid DPS 集群 | Extreme DPS 集群 |
| :--------------------- | :-------------- | :--------------- |
| 计算引擎类型           | Hybrid DPS      | Extreme DPS      |
| 每个服务单元可创建数量 | 1               | 多个，按需创建   |
| 计算规格               | XS，S           | S，M             |
| 删除                   | 不支持          | 支持             |
| 扩缩容                 | 支持            | 支持             |
| 手动暂停               | 支持            | 支持             |



<br/>
