ALTER SCHEMA
=====

修改 Schema 定义。

---

语法
--------

```sql
ALTER SCHEMA <name> RENAME TO <new_name>

ALTER SCHEMA <name> OWNER TO { <new_owner> | CURRENT_USER | SESSION_USER }
```

---

描述
--------

只有 Schema 的所有者才可以使用 `ALTER SCHEMA` 来修改 Schema 的定义。

当你使用 `ALTER SCHEMA` 进行 Schema 重命名时，你必须是该 Schema 的所有者，并且拥有该 Schema 所属数据库的 `CREATE` 权限。

当你使用 `ALTER SCHEMA` 来修改 Schema 的所有者时，你必须拥有该 Schema，并且是新所有角色中的直接或间接成员。此外，新的所有角色必须拥有 Schema 所属的数据库的 `CREATE` 权限。


---

参数
----------

- _`<name>`_

    Schema 的当前名称。

- _`<new_name>`_

    Schema 的新名称。不能以 `pg_` 开头，以 `pg_` 开头的名称是系统保留的 Schema。

- _`<new_owner>`_, **`CURRENT_USER`**, 或 **`SESSION_USER`**

    Schema 的新所有者。
   
   - _`<new_owner>`_: 明确指定新的所有角色。
   - `CURRENT_USER`: 将所有者更改为当前用户。
   - `SESSION_USER`: 将所有者更改为当前会话用户。

---

示例
--------

将 Schema `basicinfo` 重命名为 `basic_info`：

```sql
ALTER SCHEMA basicinfo RENAME TO basic_info;
```
将 Schema `basic_info` 的所有者更改为 `mary@example.com`：

```sql
ALTER SCHEMA basic_info OWNER TO "mary@example.com";
```

---

SQL 标准兼容性
-------------

标准 SQL 不支持 `ALTER SCHEMA`。
