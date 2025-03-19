ALTER EXTERNAL TABLE
=====

更改外部表定义。


---

语法
--------

```sql
ALTER EXTERNAL TABLE <name> <action> [, ... ]
  其中 <action> 可以是：
  ADD [COLUMN] <new_column> <type>
  DROP [COLUMN] <column> [RESTRICT | CASCADE]
  ALTER [COLUMN] <column> TYPE <type>
  OWNER TO <new_owner>
```

---

描述
--------

要更改外部表的 Schema，你必须是外部表的所有者并且拥有新 Schema 的 `CREATE` 权限。 

要更改外部表的所有者，你必须是外部表的当前所有者，并且在新的所有者角色的直接或间接成员。新的所有者角色必须拥有外部表所在的 Schema 的 `CREATE` 权限。

执行以下操作时，需要使用 `ALTER TABLE` 而不是 `ALTER EXTERNAL TABLE`：

- 为外部表设置或更改 Schema。

- 重命名外部表或表中的列。

- 为可写的外部表设置或更改分布策略。

`ALTER EXTERNAL TABLE` 或 `ALTER TABLE` 对外部数据没有影响。如果你想更改外部表的类型（读、写、网页）、`FORMAT` 信息，或外部数据的位置，你需要先删除当前的外部表并重新创建。

---

参数
----------

- _`<name>`_

  外部表的名称，支持使用 Schema 名称进行限定。

- _`<action>`_

  在外部表上执行的操作。支持的操作包括：

  - `ADD [COLUMN]`：添加列。

    要执行此操作，你必须指定 _`<new_column>`_ 和 _`<type>`_，其中 _`<new_column>`_ 为新增列的名称，而 _`<type>`_ 为列的数据类型。

  - `DROP [COLUMN]`：删除列。

    要执行此操作，必须制定要删除的列的名称 _`<column>`_。如果该列有任何数据库对象依赖于该列，例如引用该列的视图，则需要 `CASCADE` 关键字来删除该列。

  - `ALTER [COLUMN] ... TYPE`：更改列的数据类型。

    要执行此操作，你必须指定 _`<column>`_ 和 _`<type>`_，其中 _`<column>`_ 表示你要修改的列的名称，而 _`<type>`_ 表示列的新数据类型。

  - `OWNER TO`：更改表的所有者。

    要执行此操作，你必须指定外部表的新所有者 _`<new_owner>`_。

- **`[CASCADE | RESTRICT]`**

  与 `DROP COLUMN` 操作一起使用，用于指定当列存在依赖的数据库对象时是否继续删除列。`CASCADE` 会删除该列及其所有依赖对象，`RESTRICT` 会拒绝整个删除操作。默认值为 `RESTRICT`。


---

示例
--------

从外部表 `students` 中删除列 `age`：

```sql
ALTER EXTERNAL TABLE students DROP COLUMN age;
```
    
将外部表 `students` 的所有者更改为 `mary@example.com`：

```sql
ALTER EXTERNAL TABLE students OWNER TO "mary@example.com";
```


---

SQL 标准兼容性
-------------

标准 SQL 或常规的 PostgreSQL 不支持 `ALTER EXTERNAL TABLE`。它是 MAXIR 扩展。
