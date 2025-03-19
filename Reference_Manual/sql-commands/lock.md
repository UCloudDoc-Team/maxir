LOCK
=====

锁定表。

---


语法
--------

```sql
LOCK [TABLE] [ONLY] name [ * ] [, ...] [IN <lock_mode> MODE] [NOWAIT]
where <lock_mode> can be one of:
    ACCESS SHARE | ROW SHARE | ROW EXCLUSIVE 
    | SHARE UPDATE EXCLUSIVE | SHARE | SHARE ROW EXCLUSIVE 
    | EXCLUSIVE | ACCESS EXCLUSIVE
```

---


描述
--------

MAXIR 不提供 `UNLOCK TABLE` 语句。所有在事务中的锁在事务结束时会自动被释放。

MAXIR 为每个引用表的语句采取最不限制的锁定模式并自动获取锁。你可以使用 `LOCK TABLE` 来实现更严格的锁定。



---

参数
----------

- *`<name>`*

    表的名称。如果你的语句中指定了 `ONLY`，那么只有这张表被锁定。否则，这张表及其子表都会被锁定。或者，你可以在表名后指定 `*` 明确指定要锁定子表。你可以指定多个表的名称。在这种情况下，这些表将按照它们指定的顺序逐一被锁定。

- *`<lock_mode>`*

    锁模式，指定这个锁与哪些锁冲突。可能的值包括：

    - `ACCESS SHARE`：与 `ACCESS EXCLUSIVE` 锁模式冲突。`SELECT` 命令在引用的表上获取这种模式的锁。通常，任何只读取表而不修改它的查询都会获取这种锁模式。

    - `ROW SHARE`：与 `EXCLUSIVE` 和 `ACCESS EXCLUSIVE` 锁模式冲突。`SELECT ... FOR SHARE` 命令自动在目标表上获取这种模式的锁，除了在任何其他引用但未选择 `FOR SHARE` 的表上的 `ACCESS SHARE` 锁。

    - `ROW EXCLUSIVE`：与 `SHARE`，`SHARE ROW EXCLUSIVE`，`EXCLUSIVE` 和 `ACCESS EXCLUSIVE` 锁模式冲突。`INSERT` 和 `COPY` 命令自动在目标表上获取这种模式的锁，除了在任何其他引用的表上的 `ACCESS SHARE` 锁。有关更多信息，参见[使用注意事项](#使用注意事项)。

    - `SHARE UPDATE EXCLUSIVE`：与 `SHARE UPDATE EXCLUSIVE`，`SHARE`，`SHARE ROW EXCLUSIVE`，`EXCLUSIVE` 和 `ACCESS EXCLUSIVE` 锁模式冲突。这种锁模式保护表免受并发模式更改和 VACUUM 运行的影响。`VACUUM`（无 `FULL`）在堆表上和 `ANALYZE` 获取这种模式的锁。

    - `SHARE`：与 `ROW EXCLUSIVE`，`SHARE UPDATE EXCLUSIVE`，`SHARE ROW EXCLUSIVE`，`EXCLUSIVE` 和 `ACCESS EXCLUSIVE` 锁模式冲突。这种模式保护表免受并发数据更改的影响。`CREATE INDEX` 命令自动获取这种模式的锁。

    - `SHARE ROW EXCLUSIVE`：与 `ROW EXCLUSIVE`，`SHARE UPDATE EXCLUSIVE`，`SHARE`，`SHARE ROW EXCLUSIVE`，`EXCLUSIVE` 和 `ACCESS EXCLUSIVE` 锁模式冲突。这种模式保护表免受并发数据更改的影响，并且是自我排他的，因此一次只能有一个会话持有它。

    - `EXCLUSIVE`：与 `ROW SHARE`，`ROW EXCLUSIVE`，`SHARE UPDATE EXCLUSIVE`，`SHARE`，`SHARE ROW EXCLUSIVE`，`EXCLUSIVE` 和 `ACCESS EXCLUSIVE` 锁模式冲突。这种模式只允许并发 `ACCESS SHARE` 锁，只有从表中读取的操作可以与持有此锁模式的事务并行进行。MAXIR 中的 `UPDATE`，`SELECT ... FOR UPDATE` 和 `DELETE` 自动获取这种锁模式。有关更多信息，参见[使用注意事项](#使用注意事项)。

    - `ACCESS EXCLUSIVE`：与所有模式的锁冲突。这种锁模式确保持有者是唯一使用表的事务。这是默认的锁模式。`ALTER TABLE`，`DROP TABLE`，`TRUNCATE`，`REINDEX`，`CLUSTER` 和 `VACUUM FULL` 命令自动获取这种锁模式。对于不明确指定模式的 `LOCK TABLE` 语句，这是默认的锁模式。`VACUUM`（无 `FULL`）在处理期间也会短暂地获取附加优化表的这种锁。

- **`NOWAIT`**

    指定 `LOCK TABLE` 不应等待任何冲突的锁被释放：如果无法立即获取指定的锁而无需等待，那么事务将被取消。


---


使用注意事项
----------
`LOCK TABLE ... IN ACCESS SHARE MODE` 需要在目标表上有 `SELECT` 权限。所有其他形式的 `LOCK` 都需要表级 `UPDATE`，`DELETE` 或 `TRUNCATE` 权限。

`LOCK TABLE` 在事务块外部无用：锁只会持有到 `LOCK` 语句的完成。因此，如果 `LOCK` 在事务块之外使用，MAXIR 会报告一个错误。使用 `BEGIN` 和 `END` 来定义一个事务块。

`LOCK TABLE` 只处理表级锁，因此涉及 `ROW` 的模式名称都是误导。这些模式名称通常被读作指示用户在锁定表内获取行级锁的意图。此外，`ROW EXCLUSIVE` 模式是一个可共享的表锁。请记住，所有的锁模式在 `LOCK TABLE` 中都有完全相同的语义，只在哪些模式与哪些模式冲突的规则上有所不同。


---


示例
----------

在执行插入 `films_user_comments` 表的操作时获取 `films` 表的 `SHARE` 锁：

```sql
BEGIN WORK;
LOCK TABLE films IN SHARE MODE;
SELECT id FROM films
    WHERE name = 'Star Wars: Episode I - The Phantom Menace';
-- Do ROLLBACK if record was not returned
INSERT INTO films_user_comments VALUES
    (_id_, 'GREAT! I was waiting for it for so long!');
COMMIT WORK;
```

在执行删除操作时获取表的 `SHARE ROW EXCLUSIVE` 锁：

```sql
BEGIN WORK;
LOCK TABLE films IN SHARE ROW EXCLUSIVE MODE;
DELETE FROM films_user_comments WHERE id IN
    (SELECT id FROM films WHERE rating < 5);
DELETE FROM films WHERE rating < 5;
COMMIT WORK;
```


---


SQL 标准兼容性
----------

SQL 标准不支持 `LOCK`。

MAXIR 提供的锁模式，除了 `ACCESS SHARE`，`ACCESS EXCLUSIVE` 和 `SHARE UPDATE EXCLUSIVE` 以及 `LOCK TABLE` 语法与 Oracle 中的 `LOCK TABLE` 完全兼容。
