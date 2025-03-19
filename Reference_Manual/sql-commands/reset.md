RESET
=====

将运行时系统配置参数恢复到默认值。


---

语法
--------

```sql
RESET <config_param>

RESET ALL
```

---

描述
----------

`RESET` 等效于 `SET <config_param> TO DEFAULT`。如果事务被回滚，`RESET` 在事务中的效果将被恢复。这也与 `SET` 相同。有关更多详细信息，请参见 [SET](set.md)。

参数的默认值是如果在当前会话中没有为参数 `SET` 值，则会自动分配给参数的值。这样的值可以来自编译时的默认值，`postgresql.conf` 配置文件，命令行选项，或数据库或用户的默认设置。


---

参数
----------

*`<config_param>`*：运行时系统配置参数的名称。如果你想重置所有系统配置参数，将此参数设置为 `ALL`。


---

示例
----------

将 `statement_mem` 重置为其默认值：

```sql
RESET statement_mem; 
```


---

SQL 标准兼容性
-------------

`RESET` 是 MAXIR 的扩展。
