CREATE DATABASE
=====

创建数据库。

---

语法
--------

```sql
CREATE DATABASE <name>
```

---

描述
----------

要创建数据库，你必须拥有 `CREATEDB` 权限。数据库的创建者默认为数据库的所有者。

`CREATE DATABASE` 不能在事务块中运行。

如果你遇到错误，并且错误信息显示“could not initialize database directory”，可能的原因包括数据目录上的权限不足、磁盘空间不足或者文件系统错误。


---
参数
----------

*`<name>`*：数据库的名称。


---

示例
--------

创建一个名为 `mydb` 的数据库：

```sql
CREATE DATABASE mydb;
```

---

SQL 标准兼容性
-------------

标准 SQL 不支持 `CREATE DATABASE`。
