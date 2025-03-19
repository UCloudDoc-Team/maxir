SHOW
=====

显示运行时配置参数的值或所有运行时配置参数的值。

---

语法
--------

```sql
SHOW <name>

SHOW ALL
```

---

描述
----------

你可以使用 `SHOW` 检查通过 `SET`、编辑 `postgresql.conf` 配置文件或通过 `PGOPTIONS` 环境变量设置的运行时配置参数的当前设置。

:::info
你可以使用函数 `current_setting()`（参见 PostgreSQL 文档中的[System Administration Functions](https://www.postgresql.org/docs/12/functions-admin.html)了解详情）或系统视图 [pg_setting](https://www.postgresql.org/docs/12/view-pg-settings.html) 作为 `SHOW` 的替代方案。
:::

---
参数
----------

- *`<name>`*

    运行时配置参数的名称。

- **`ALL`**

    你可以运行 `SHOW ALL` 来显示所有运行时配置参数的值。


---

示例
----------

显示参数 `DateStyle` 的当前设置：

```sql
SHOW DateStyle;
 DateStyle
-----------
 ISO, MDY
(1 row)
```

显示参数 `row_security` 的当前设置：

```sql
SHOW row_security;
 row_security
--------------
 on
(1 row)
```


显示所有参数的当前设置：

```sql
SHOW ALL;
       name       | setting |                  description
-----------------+---------+----------------------------------------------------
 application_name | psql    | Sets the application name to be reported in sta...

 ...

 xmlbinary        | base64  | Sets how binary values are to be encoded in XML.
 xmloption        | content | Sets whether XML data in implicit parsing and s...
(473 rows)
```


---


SQL 标准兼容性
-------------

`SHOW` 是 MAXIR 的扩展。
