ALTER VIEW
=====

修改视图定义。


---

语法
--------

```sql
ALTER VIEW [ IF EXISTS ] <name> ALTER [ COLUMN ] <column_name> SET DEFAULT <expression>

ALTER VIEW [ IF EXISTS ] <name> ALTER [ COLUMN ] <column_name> DROP DEFAULT

ALTER VIEW [ IF EXISTS ] <name> OWNER TO <new_owner>

ALTER VIEW [ IF EXISTS ] <name> RENAME TO <new_name>

ALTER VIEW [ IF EXISTS ] <name> SET SCHEMA <new_schema>

ALTER VIEW [ IF EXISTS ] <name> SET ( <view_option_name> [= <view_option_value>] [, ... ] )

ALTER VIEW [ IF EXISTS ] <name> RESET ( <view_option_name> [, ... ] )
```

---

描述
----------

只有视图所有者可以使用 `ALTER VIEW` 来修改视图定义。

此命令不能用于修改定义视图的查询。如果你想实现这一点，执行 `CREATE OR REPLACE VIEW`。

当你使用 `ALTER VIEW` 来更改视图的 Schema 时，你必须是视图的所有者并且在新的 Schema 上具有 `CREATE` 权限。

当你使用 `ALTER VIEW` 来更改视图的所有者时，你必须是新的所有角色的直接或间接成员。此外，新的所有角色必须拥有视图所属 Schema 的 `CREATE` 权限。


---

参数
----------

- **`IF EXISTS`**

    当命令中包含 `IF EXISTS` 时，如果指定视图不存在，系统不会产生报错，而是生成一条通知。

- _`<name>`_

    视图的名称，支持使用 Schema 名称进行限定。

- **`SET DEFAULT`**

    为列设置默认值。

- **`DROP DEFAULT`**

    删除列的默认值。

- _`<new_owner>`_

    视图的新所有者。

- _`<new_name>`_

    视图的新名称。

- _`<new_schema>`_

    视图的新 Schema。

- **`SET | RESET`**

    指定设置或重置视图选项。支持的视图选项包括：

    - `check_option` 指定视图的检查选项，可以设置为 `local` 或 `cascaded`。 

    - `security_barrier` 指定视图的 security-barrier 属性，支持布尔值。

---

示例
--------

重命名视图 `myview` 为 `my_view`：

```sql
ALTER VIEW myview RENAME TO my_view;
```

为可更新视图附加默认列值：

1. 创建名为 `main_table` 的表：

    ```sql
    CREATE TABLE main_table (product_id varchar, product_date date);
    ```
2. 创建名为 `products_view` 的视图：

    ```sql
    CREATE VIEW products_view AS SELECT * FROM main_table;
    ```
3. 在 `products_view` 中为列 `product_date` 添加默认值：

    ```sql
    CREATE OR REPLACE VIEW products_view AS 
    SELECT product_id, COALESCE(product_date, CURRENT_DATE) AS product_date
    FROM main_table;
    ```
4. 在不指定 `product_date` 的情况下向 `products_view` 插入新行：

    ```sql
    INSERT INTO products_view(product_id) VALUES('PROD003');
    ```
    
    在此示例中，将使用当前日期。


---


SQL 标准兼容性
-------------

`ALTER VIEW` 是 MAXIR 对 SQL 标准的扩展。
