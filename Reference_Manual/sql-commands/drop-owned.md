DROP OWNED
=====

删除指定角色拥有的所有数据库对象。

---

语法
--------

```sql
DROP OWNED BY { <name> | CURRENT_USER | SESSION_USER } [, ...] [CASCADE | RESTRICT]
```


---

描述
----------

`DROP OWNED` 删除当前数据库中由指定角色拥有的所有对象。在当前数据库的对象或共享对象上授予角色的权限也将被撤销。


---

参数
----------

- *`<name>`*

    角色的名称。你可以指定多个角色名称。

- **`CASCADE`** 或 **`RESTRICT`**

    是否删除角色拥有的对象以及依赖于受影响对象的对象（如果有）。支持的选项包括：

    - `CASCADE`：指定删除它们。
    - `RESTRICT`：指定拒绝删除操作，这是默认值。


---

示例
-------------

删除角色 `mary` 拥有的数据库对象：

```sql
DROP OWNED BY mary;
```

---

SQL 标准兼容性
-------------

`DROP OWNED` 是 MAXIR 的扩展。
