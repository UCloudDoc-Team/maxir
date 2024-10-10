# SQL 计划管理
SPM（SQL 计划管理，SQL Plan Management）是 MAXIR 提供的一项功能，旨在确保 SQL 查询计划的稳定性。允许你将一个 SQL 命令（称为源 SQL 命令）绑定到另一个 SQL 命令（称为目标 SQL 命令）。源 SQL 命令和目标 SQL 命令之间的关联被定义为 SQL 绑定 (SQL Binding)。启用 SPM 后，每次发布源 SQL 命令运行时，MAXIR 运行目标 SQL 命令。

如您想了解如何使用 SQL 计划管理，请参考“标准版操作指南”中的 [ SQL 计划管理](/maxir/guides/optimization/sql-plan-management) 。
