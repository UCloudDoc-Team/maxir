COMMENT
=====

设置或更改数据库对象的注释。

---

## 语法


```sql
COMMENT ON
{ TABLE <object_name> |
  COLUMN <relation_name.column_name> |
  DATABASE <object_name> |
  FUNCTION <func_name> ([[<arg_mode>] [<arg_name>] <arg_type> [, ...]]) |
  MATERIALIZED VIEW <object_name> |
  [PROCEDURAL] LANGUAGE <object_name> |
  ROLE <object_name> |
  SCHEMA <object_name> |
  SEQUENCE <object_name> |
  VIEW <object_name> }
IS 'text'
```
---
## 描述

你可以使用 `COMMENT` 对你所拥有的数据库对象进行注释。

每个对象只存储一条注释。要删除注释，使用 `IS NULL`。一旦对象被删除，其注释也将自动被删除。

你可以运行 psql 元命令 `\dd`，`\d+` 和 `\l+` 来检索注释。你可以通过使用与 psql 相同的内置系统函数创建其他用户界面来检索注释，例如 `obj_description`、`col_description` 和 `shobj_description`。

不要在注释中放置关键信息，因为数据库中对象的注释对任何连接到数据库的用户都是可见的。

---

参数
----------

- _`<object_name>`_, _`<relation_name.column_name>`_, 或 _`<func_name>`_

  要添加注释的数据库对象的名称。如果数据库对象是表、函数、物化视图或视图，你可以在对象名称前加上 Schema 名称进行限定。

- _`<arg_mode>`_

  参数模式。支持的值包括 `IN`、`OUT`、`INOUT`和 `VARIADIC`。如果未指定此参数，将使用默认值 `IN`。`OUT` 参数不需要列出，因为 `COMMENT` 只使用输入参数来识别函数。

- _`<arg_name>`_

  参数的名称。 
   
  请注意，`COMMENT ON FUNCTION` 只使用参数的数据类型来识别函数，而不是参数名称。 

- _`<arg_type>`_

  参数的数据类型。 

- **`PROCEDURAL`**

  填充词。 

- _`<text>`_
  
  注释字符串。要删除注释，设置该参数为 `NULL`。
  

---

示例
--------

对表 `sales` 进行注释：

```sql
COMMENT ON TABLE sales IS 'Sales information';
```

删除表 `sales` 的注释：

```sql
COMMENT ON TABLE sales IS NULL;
```


---

SQL 标准兼容性
-------------

标准 SQL 不支持 `COMMENT`。
