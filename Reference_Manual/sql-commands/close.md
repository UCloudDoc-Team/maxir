CLOSE
=====

关闭游标。

---

语法
--------

```sql
CLOSE <cursor_name>;
```

---

描述
----------

关闭游标后，与游标相关联的资源会被释放。

当事务通过 `COMMIT` 或 `ROLLBACK` 被终止后，所有非保持式开放游标都会被隐式关闭。当事务通过 `COMMIT` 停止时，事务创建的开放游标会被隐式关闭。当事务成功提交时，事务创建的保持式游标仍然开放，直到运行 `CLOSE` 或客户端断开连接。

>info
>在 MAXIR 中，使用 `DECLARE` 声明的游标默认是开放的，因为 MAXIR 不提供游标的 `OPEN` 语句。

---


参数
----------

*`<cursor_name>`*：游标的名称。


---

示例
--------

关闭游标 `my_cursor`：

```sql
CLOSE my_cursor;
```


---

SQL 标准兼容性
-------------

MAXIR 中的 `CLOSE` 与 SQL 标准中的 `CLOSE` 完全兼容。
