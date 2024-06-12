# SQL 计划管理

本文解释了什么是 SQL 计划管理，并描述了如何使用此功能。

## 概述

SPM（SQL 计划管理，SQL Plan Management）是 Relyt 提供的一项功能，旨在确保 SQL 查询计划的稳定性。允许你将一个 SQL 命令（称为源 SQL 命令）绑定到另一个 SQL 命令（称为目标 SQL 命令）。源 SQL 命令和目标 SQL 命令之间的关联被定义为 SQL 绑定 (SQL Binding)。启用 SPM 后，每次发布源 SQL 命令运行时，Relyt 运行目标 SQL 命令。

:::info
目前，只有 `SELECT` 可以同时作为源 SQL 命令和目标 SQL 命令。
:::

## 创建 SQL 绑定

要使用 SPM 功能，你必须至少有一个 SQL 绑定。
### 语法

```sql
CREATE BINDING FOR '<source_sql>' USING STMT '<destination_sql>';
```

在此语法中，

- *`<source_sql>`*：一个字符串，指定源 SQL 命令。

- *`<destination_sql>`*：一个字符串，指定目标 SQL 命令。

:::note
确保 *`<source_sql>`* 和 *`<destination_sql>`* 的值均符合 [SELECT](reference/sql-commands/select.md) 语法。
:::

### 示例

创建一个 SQL 绑定，指导 Relyt 在有人尝试运行 `SELECT orders_delivered FROM sales` 时运行 `SELECT orders_delivered FROM sales`：

```sql
CREATE BINDING FOR 'SELECT orders_closed FROM sales' USING STMT 'SELECT orders_delivered FROM sales';
```

## 检查 SQL 绑定

Relyt 将关于 SQL 绑定的信息存储在 `pg_catalog.relyt_spm_info` 表中。要检查 SQL 绑定，你可以运行 `SELECT * FROM pg_catalog.relyt_spm_info`。

:::info
要访问 `pg_catalog.relyt_spm_info` 表中的数据，你必须使用实用程序模式连接到 Relyt。
:::

下图显示了表结构。
![](2023-11-06-19-47-38.png)

在这个表中，

- `original_query_id`：一个整数，表示 SQL 绑定适用的查询的 ID。

- `original_sql`：一个字符串，表示源 SQL 命令。

- `bind_sql`：一个字符串，表示目标 SQL 命令。

- `constant_size`：一个整数，表示源 SQL 命令中的常数数量。

- `constant_list`：源 SQL 命令中的常数列表。

- `enabled`：一个布尔值，表示 SQL 绑定的状态。

其他参数保留用于未来的扩展，可以忽略。

## 修改 SQL 绑定

创建 SQL 绑定后，你可以在 `pg_catalog.relyt_spm_info` 表中更新其设置。

### 语法

```sql
UPDATE <table_name> SET <config_param_to_update> = <new_value>[, ... ] WHERE <config_param> = <value>[, ... ];
```

在此语法中，

- *`<table_name>`*：存储 SQL 绑定设置的表。它固定为 `pg_catalog.relyt_spm_info`。

- *`<config_param_to_update>`*：你想要更改值的配置参数。

- *`<new_value>`*：配置参数的新值。

- *`<config_param>`*：你指定的用于识别 SQL 绑定的配置参数。

   必须指定 `original_query_id` 和 `original_sql` 中的至少一个。最好使用 `original_query_id`。这是因为 `original_sql` 的值是区分大小写的，并且必须严格符合格式要求。

   SPM 允许一个 SQL 命令绑定到多个 SQL 命令，也允许重复的 SQL 绑定。因此，如果你在 `DELETE` 命令中只指定了一个 `original_query_id` 或 `original_sql`，所有具有相同 `original_query_id` 或 `original_sql` 的 SQL 绑定都将被删除。我们建议你指定尽可能多的配置参数，以帮助唯一地识别你想要删除的绑定。要检查 SQL 绑定的配置设置，参见 [检查 SQL 绑定](#检查-sql-绑定)。

- *`<value>`*：配置参数的当前值。

:::caution
我们建议你使用此语法仅更改 `enabled` 参数。更改其他参数的值可能会导致意外错误。
:::

### 示例

禁用源 SQL 命令为 `SELECT orders_closed FROM sales` 的 SQL 绑定：

```sql
UPDATE pg_catalog.relyt_spm_info SET enabled = false WHERE original_sql = 'SELECT orders_closed FROM sales';
```

## 使用 SQL 绑定

要使用 SQL 绑定，你必须确保已为绑定或所有绑定启用 SPM。

### 为特定 SQL 绑定启用 SPM

要启用特定的 SQL 绑定，你需要在 `pg_catalog.relyt_spm_info` 中将其 `enabled` 的值更改为 true。

例如，启用源 SQL 命令为 `SELECT orders_closed FROM sales` 的 SQL 绑定：

```sql
UPDATE pg_catalog.zdb_spm_info SET enabled = true WHERE original_sql = 'SELECT orders_closed FROM sales';
```

有关语法的更多信息，参见 [修改 SQL 绑定](#修改sql绑定)。

### 为所有 SQL 绑定启用 SPM

Relyt 使用 `relyt.spm_enable` 变量来控制 SQL 功能。要为所有 SQL 绑定启用 SPM，将 `relyt.spm_enable` 设置为 true，例如：

```sql
SET relyt.spm_enable = on;
```

:::info
值 `on` 可以是 Relyt 中表示 true 的任何值。更多详情，请参阅 [布尔类型](reference/data-types/boolean.md)。
:::