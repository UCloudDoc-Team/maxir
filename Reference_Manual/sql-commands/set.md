SET
=====

更改运行时 MAXIR 配置参数的值。

---

语法
--------

```sql
SET [ SESSION | LOCAL ] <config_param> { TO | = } { <value> | '<value>' | DEFAULT }

SET [SESSION | LOCAL] TIME ZONE { <value> | '<value>' | LOCAL | DEFAULT }
```

---

描述
----------

你可以使用 `SET` 来更改配置参数的值。新的值仅对当前会话有效。

假设正在运行包含 `SET` 或 `SET SESSION` 的事务。如果在运行 `SET` 或 `SET SESSION` 后，事务回滚，由 `SET` 或 `SET SESSION` 指定的新配置参数设置也将回滚。如果事务被提交，新的配置参数设置在会话结束之前都有效，除非被另一个 `SET` 覆盖。

---
参数
----------

- **SESSION** 或 **LOCAL**

    新设置的有效范围。可能的选项包括：

    - `SESSION`：指定更改仅对当前会话有效，这是默认设置。
    - `LOCAL`：指定更改仅对当前事务有效。

- *`<config_param>`*

    配置参数的名称。

- *`<value>`* 或 **`DEFAULT`**

     参数的新值。值的数据类型由配置参数决定，可以是字符串常量、标识符、数字，或者它们的逗号分隔列表。你可以用 `DEFAULT` 来将配置参数设置为其默认值。如果值表示内存大小或时间，请用一对单引号 (' ') 包围值。


- **`SCHEMA`**

     `SET SCHEMA 'value'` 可以用来替代 `SET search_path TO <value>`，以更改 Schema 的搜索路径。只能用这种语法指定一个 Schema。

- **`NAMES`**

    `SET NAMES <value>` 可以用来替代 `SET client_encoding TO <value>`，以更改字符编码。

- **`SEED`**

    使用 `SEED` 来设置随机数生成器（函数 `random()`）的内部种子。可能的值是从 -1 到 1 的浮点数。

    MAXIR 也允许你调用函数 `setseed()` 来重置种子。以下是语法：

    ```sql
    SELECT setseed(value);
    ```

- **`TIME ZONE`**

    `SET TIME ZONE <value>` 可以用来替代 `SET timezone TO <value>`，以重置时区。以下是 *`<value>`* 的值示例：

    - `'Europe/Rome'`
    - `-7`（从 UTC 向西 7 小时的时区）
    - `INTERVAL '-08:00' HOUR TO MINUTE`（从 UTC 向西 8 小时的时区）


- **`LOCAL`** 或 **`DEFAULT`**
   
   指定使用本地时区或默认时区。 



---

示例
----------

设置 Schema 搜索路径：
```sql
SET search_path TO my_schema, public;
```
将每个查询的段主机内存增加到 200 MB：
```sql
SET statement_mem TO '200MB';
```
将日期样式设置为传统的 POSTGRES，使用 "day before month" 的输入约定：
```sql
SET datestyle TO postgres, dmy;
```

为意大利设置时区：
```sql
SET TIME ZONE 'Europe/Rome'; 
```

---

SQL 标准兼容性
-------------

所有的 `SET` 特性都是 MAXIR 扩展，除了 `SET TIME ZONE`。

MAXIR 中的 `SET TIME ZONE` 与 SQL 标准中的 `SET TIME ZONE` 完全兼容，并扩展了对更灵活的时区规范的支持，而 SQL 标准中的 `SET TIME ZONE` 仅支持数字时区偏移。 


