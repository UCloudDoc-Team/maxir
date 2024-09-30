# 自适应查询扩展 AQS

AQS（自适应查询扩展，Adaptive Query Scaling）是 MAXIR 提供的一项高级功能，旨在确保你的 DPS 集群在处理大查询、复杂查询期间的稳定性和高性能，提供了至少三个9的查询成功率。

AQS 能在运行时自动识别大型查询，并将它们发送到 AQS 共享计算资源池中进行处理，不会对当前运行在 DPS 集群的工作负载产生任何影响，从而保证了集群的稳定运行。启用 AQS 后，符合以下任一情况的查询将会被识别为大型查询：
- 由于资源限制（如“not enough memory”）导致当前 DPS 集群无法正常运行的查询
- 查询运行会影响当前 DPS 集群的性能


## 启用 AQS

方式一：在创建 Extreme DPS 集群时，可以为集群启用 AQS。详情参考[ DPS 管理 ](/maxir/guides/dps-clusters/manager-dps#创建-extreme-dps)中的“创建 Extreme DPS ”。

方式二：修改 Extreme DPS 配置时，可以为集群启用 AQS。详情参考[ DPS 管理 ](/maxir/guides/dps-clusters/manager-dps#修改-dps-配置)中的“修改 DPS 配置 ”。


## 注意事项

AQS 功能本身不会产生费用。只有当 AQS 共享计算资源被用于处理大查询时，你才需要为真实使用了的计算资源付费。

AQS 始终选择最有效的计算资源量来处理你的大型查询。然而，为了帮助你更好地管理你的云预算，AQS 要求你在为 DPS 集群启用 AQS 时，设置 AQS 可使用的共享计算资源的上限，控制 AQS 从共享资源池中分配的最大计算资源。

使用 AQS 时，请注意以下限制：

- 目前，只能为 Extreme DPS 集群启用 AQS。

- AQS 只能在创建 DPS 集群时启用。如果你想为已有的 DPS 集群启用该功能，需要先删除当前集群并重新创建。

- AQS 是集群级的功能。如需在多个 DPS 集群上启用该功能，需要在创建这些 DPS 集群时，分别为每一个集群启用。
