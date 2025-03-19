REVOKE
=====

撤销一个或多个角色的权限。


---

语法
--------

```sql
REVOKE [GRANT OPTION FOR]
       { {SELECT | INSERT | UPDATE | DELETE | TRUNCATE }
       [, ...] | ALL [PRIVILEGES] }
       ON { { [TABLE] [[[ONLY] <table_name> [, ...]] [, ...]] }
          | ALL TABLES IN SCHEMA <schema_name [, ...] }
       FROM <role_spec> [, ...]
       [CASCADE | RESTRICT]

REVOKE [ GRANT OPTION FOR ]
       { { SELECT | INSERT | UPDATE } ( <column_name> [, ...] )
       [, ...] | ALL [ PRIVILEGES ] ( <column_name> [, ...] ) }
       ON { [ [TABLE] [[[ONLY] <table_name> [, ...]] [, ...]] }
       FROM <role_spec> [, ...]
       [ CASCADE | RESTRICT ]

REVOKE [GRANT OPTION FOR]
       { {CREATE | CONNECT | TEMPORARY | TEMP} [, ...] | ALL [PRIVILEGES] }
       ON DATABASE <database_name> [, ...]
       FROM <role_spec> [, ...]
       [CASCADE | RESTRICT]


REVOKE [GRANT OPTION FOR] {EXECUTE | ALL [PRIVILEGES]}
       ON { { FUNCTION | PROCEDURE | ROUTINE }  <func_name> [( [[<arg_mode>] [<arg_name>] <arg_type> [, ...]] )] [, ...]
            | ALL { FUNCTIONS | PROCEDURES | ROUTINES } IN SCHEMA schema_name [, ...] }
       FROM <role_spec> [, ...]
       [CASCADE | RESTRICT]

REVOKE [GRANT OPTION FOR] { {CREATE | USAGE} [, ...] | ALL [PRIVILEGES] }
       ON SCHEMA <schema_name> [, ...]
       FROM <role_spec> [, ...]
       [CASCADE | RESTRICT]

where <role_spec> can be:

    [ GROUP ] <role_name>
  | PUBLIC
  | CURRENT_USER
  | SESSION_USER
```

---

描述
----------

要撤销所有角色的权限，你可以在语句中指定关键字 `PUBLIC`。然而，从对象上撤销 `PUBLIC` 的权限并不表示所有角色都没有该对象的权限。这是因为角色的权限包括直接授予它的权限，`PUBLIC` 的权限，以及授予该角色的任何其他角色的权限。例如，从 `PUBLIC` 撤销视图 `myview` 的 `INSERT` 权限后，直接授予 `INSERT` 权限的角色及其成员仍然可以拥有 `myview` 的 `INSERT` 权限。


要从角色中撤销权限的授权选项，你可以在语句中指定 `GRANT OPTION FOR`。这样做，角色仍然拥有权限。注意，如果你还指定了关键字 `CASCADE`，权限也将从该角色授予的角色中撤销。

当你从一个角色 `REVOKE` 一个表的权限时，表的每一列的权限也将被撤销。但是，当你从表的一列或多列 `REVOKE` 权限时，角色仍然拥有表级别的权限。

---

参数
----------

有关详细信息，请参见 [GRANT](grant.md#参数) 中的“参数”部分。


---

示例
----------

撤销角色 `mary` 对表 `students` 的 `SELECT` 权限：

```sql
REVOKE SELECT ON students FROM mary;
```


撤销用户 `john` 的 `role1` 角色成员资格：

```sql
REVOKE role1 FROM john;
```

---


SQL 标准兼容性
-------------
参见 [GRANT](grant.md#标准-sql-兼容性) 中的“SQL 标准兼容性“章节。
