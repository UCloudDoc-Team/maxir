ALTER TABLE
=====

修改表定义。

---

语法
--------

```sql
ALTER TABLE [IF EXISTS] [ONLY] <name> [ * ]
    <action> [, ... ]

ALTER TABLE [IF EXISTS] [ONLY] <name> [ * ]
    RENAME [COLUMN] <column_name> TO <new_column_name>

ALTER TABLE [IF EXISTS] [ ONLY ] <name> [ * ]
    RENAME CONSTRAINT <constraint_name> TO <new_constraint_name>

ALTER TABLE [IF EXISTS] <name> 
    RENAME TO <new_name>

其中 <action> 可以是:
  ADD [COLUMN] [IF NOT EXISTS] <column_name> <data_type> [<column_constraint> [ ... ]]
      [ ENCODING ( <storage_directive> [,...] ) ]
  DROP [COLUMN] [IF EXISTS] <column_name> [RESTRICT | CASCADE]
  ALTER [COLUMN] <column_name> [ SET DATA ] TYPE <data_type> [USING <expression>]
  ALTER [COLUMN] <column_name> SET DEFAULT <expression>
  ALTER [COLUMN] <column_name> DROP DEFAULT
  ALTER [COLUMN] <column_name> { SET | DROP } NOT NULL
  ALTER [COLUMN] <column_name> SET ( <attribute_option> = <value> [, ... ] )
  ALTER [COLUMN] <column_name> RESET ( <attribute_option> [, ... ] )
  ALTER [COLUMN] <column_name> SET STORAGE { PLAIN | EXTERNAL | EXTENDED | MAIN }
  ALTER [COLUMN] <column_name> SET ENCODING ( storage_directive> [, ...] )
  DROP CLUSTERING KEY
```


---

描述
----------

只有表所有者可以使用 `ALTER TABLE` 来修改表定义。除非明确指出，否则执行 `ALTER TABLE` 会获取 `ACCESS EXCLUSIVE` 锁。当提供多个子命令时，MAXIR 会获取任何子命令所需的最严格锁。

`ALTER TABLE` 有几个变体，包括：

- `ALTER TABLE ... RENAME [COLUMN] <column_name> TO`：重命名表中的列。

- `ALTER TABLE ... RENAME CONSTRAINT ... TO`：重命名表中的约束。

- `ALTER TABLE ... RENAME TO`：重命名表。


---

参数
----------

- **`IF EXISTS`**

    如果命令中包含了 `IF EXISTS`，并且指定的视图不存在，系统不会报错，仅会产生一条通知。

- **`ONLY`**

    如果命令中包含 `ONLY`，只有指定表会被修改。否则，指定表及其子表（如果存在）都会被修改。你可以选择在表名后指定 `*` 来明确指出哪些子表需要同步更改。

- *`<name>`*

    表的名称，支持使用 Schema 名称进行限定。

- *`<action>`*

    要执行的操作，支持的操作包括：

    - `ADD [COLUMN]`：添加列。

        要执行此操作，你必须指定 *`<column_name>`* 和 *`<data_type>`*，其中 *`<column_name>`* 为新增列的名称，*`<data_type>`* 为列的数据类型。

    - `DROP [COLUMN]`：删除列。

        要执行此操作，必须制定要删除的列的名称 *`<column_name>`*。如果有任何数据库对象依赖于该列，例如引用该列的视图，则需要配合使用 `CASCADE` 关键字来删除该列以及依赖于该列的所有对象。否则，删除操作失败。

    - `ALTER [COLUMN] ... [SET DATA] TYPE`：更改列的数据类型。

        可选的 `USING` 子句指定如何从旧的计算新的列值。如果省略，默认转换与旧数据类型到新的赋值转换相同。如果原有类型到新类型没有隐式或赋值转换，必须提供 `USING` 子句。

    - `ALTER [COLUMN] ... {SET | DROP} DEFAULT`：为列设置或删除默认表达式。默认表达式仅在后续的 `INSERT` 或 `UPDATE` 命令中应用。它们不会导致表中已经存在的行发生更改。

    - `ALTER [COLUMN] ... {SET | DROP} NOT NULL`：更改列是否被标记为允许空值或拒绝空值。

        只有在表中的记录都不含有 `NULL` 值的情况下，才能将 `SET NOT NULL` 应用于列。这通常在 `ALTER TABLE` 中通过扫描整个表来检查。

        如果此表是一个分区，当该表在父表中被标记为 `NOT NULL` 时，则不能在列上执行 `DROP NOT NULL`。要从所有分区中删除 `NOT NULL` 约束，需要在父表上执行 `DROP NOT NULL`。即使父表上没有 `NOT NULL` 约束，也可以根据需要将此类约束添加到各个分区上，即在父表允许空值的情况下，可以在子表上禁止空值的使用。

    - `ALTER [COLUMN] ... SET STATISTICS`：为后续的 [ANALYZE](analyze.md) 操作设置每列的统计信息收集目标。目标可以设置在 0 到 10000 的范围内，或设置为 -1 以恢复使用系统默认的统计目标 `default_statistics_target`。当设置为 0 时，不收集任何统计信息。

        `SET STATISTICS` 获取 `SHARE UPDATE EXCLUSIVE` 锁。

    - `ALTER [COLUMN] ... SET ENCODING`：为 MAXIR 表设置列编码选项。

    - `ADD ... [NOT VALID]`：使用与 `CREATE TABLE` 相同的语法向表添加新约束。目前，`NOT VALID` 选项仅允许用于 CHECK 约束。
    
        通常，这种形式需要扫描表以验证表中所有的行都满足新的约束。如果约束被标记为 `NOT VALID`，MAXIR 将跳过耗时的初始检查，不再验证表中的所有行是否满足约束。约束仍将对后续插入或更新进行强制执行。但是，数据库不会假设约束适用于表中的所有行，直到使用 `VALIDATE CONSTRAINT` 选项进行验证。有关使用 `NOT VALID` 选项的更多信息，请参考 [使用注意事项](#使用注意事项)。

        大多数形式的 ADD *`<table_constraint>`* 需要 `ACCESS EXCLUSIVE` 锁。


    - `DROP CLUSTERING KEY`：删除 Clustering Key。

- **`CASCADE | RESTRICT`**

  与 `DROP COLUMN` 操作一起使用，以指定是否在列具有依赖的数据库对象时删除列。`CASCADE` 会连同列一起删除所有依赖的对象。`RESTRICT` 将拒绝整个删除操作。`RESTRICT` 是默认值。

- *`<new_name>`*

    表的新名称。在使用 `ALTER TABLE ... RENAME TO` 时，需要此参数。

- *`<column_name>`*

    要修改的列的名称。

- *`<new_column_name>`*

    列的新名称。在使用 `ALTER TABLE ... RENAME [COLUMN] ... TO` 时，此参数必填。

- *`<constraint_name>`*

    约束的名称。

- *`<new_constraint_name>`*

    约束的新名称。在使用 `ALTER TABLE ... RENAME CONSTRAINT ... TO` 时，此参数必填。

- *`<data_type>`*

    列的数据类型。

- *`<table_constraint>`*

    表的新约束。


---


使用注意事项
--------

关键字 `COLUMN` 可以省略。

当使用 `ADD COLUMN` 添加列时，表中的所有现有行都用列的默认值进行初始化，或者如果没有指定 `DEFAULT` 子句，则用 `NULL` 初始化。使用非空默认值添加列或更改现有列的类型将要求重写整个表和索引。

添加 `NOT NULL` 约束需要扫描表以验证现有行满足约束，但不需要重写表。

MAXIR 提供了在单个 `ALTER TABLE` 中指定多个更改的选项，以便可以将多个表扫描或重写合并到对表的单次遍历中。

扫描大表以验证新的检查约束可能需要很长时间，并且在 `ALTER TABLE ... ADD CONSTRAINT` 命令提交之前，其他对表的更新被锁定。`NOT VALID` 约束选项的主要目的是减少添加约束对并发更新的影响。有了 `NOT VALID`、`ADD CONSTRAINT` 命令不会扫描表，可以立即提交。之后，可以通过 `VALIDATE CONSTRAINT` 命令来验证现有的行是否满足约束。验证步骤不需要锁定并发更新，因为它知道其他事务将为他们插入或更新的行执行约束；只需要检查预先存在的行。因此，验证仅获取正在修改的表的 `SHARE UPDATE EXCLUSIVE` 锁。除了改善并发性，使用 `NOT VALID` 和 `VALIDATE CONSTRAINT` 在表已知包含预先存在的违规情况时也可能有用。一旦约束到位，就不能插入新的违规，你可以修正现有的问题，直到 `VALIDATE CONSTRAINT` 最终成功。

`DROP COLUMN` 并非真正删除了列，而只是使被删除列对 SQL 操作不可见。表中的后续插入和更新操作将为该列存储空值。因此，`DROP COLUMN` 操作执行速度很快，但并不会立即减少你的表所占用的磁盘空间，因为删除的列占用的空间没有被回收。只有这些列被更新时，空间才会被回收。然而，如果你删除系统 `oid` 列，表将立即被重写。

要强制立即回收被删除列占用的空间，你可以运行 `ALTER TABLE`，该形式将对整个表进行重写。这将导致每行都被重新构造，将删除的列替换为空值。

此表列出了在对指定类型的表存储定义的表上执行时需要表重写的 `ALTER TABLE` 操作。

:::caution
执行表重写的 `ALTER TABLE` 形式不是 MVCC 安全的。在表重写之后，如果它们使用的是重写发生之前的快照，那么并发事务将看到的表是空的。更多详情，请参考 [MVCC Caveats](https://www.postgresql.org/docs/12/mvcc-caveats.html)。
:::


如果表有任何子表，那么不允许在父表中添加、重命名或更改列的类型，而不对子表做同样的处理。这确保了子表始终具有与父表匹配的列。 此外，因为从父表选择也会从其子表选择，所以除非它也对那些子表标记为有效，否则不能在父表上标记约束为有效。在所有这些情况下，`ALTER TABLE ONLY` 将被拒绝。

递归的 `DROP COLUMN` 操作只会删除子表的列，如果子表不从任何其他父表继承该列，并且从未有过对该列的独立定义。非递归的 `DROP COLUMN` (`ALTER TABLE ONLY ... DROP COLUMN`) 从不删除任何子列，而是将它们标记为独立定义而不是继承。


---

示例
--------

将表 `new` 重命名为 `freshmen`：

```sql
ALTER TABLE new RENAME TO freshmen;
```

在表 `freshmen` 中添加名为 `year` 的列：

```sql
ALTER TABLE freshmen
ADD COLUMN year integer;
```

从表 `freshmen` 中删除列 `c1`：

```sql
ALTER TABLE freshmen
DROP COLUMN c1;
```

更改表 `freshmen` 中列 `age` 的数据类型为 `smallint`：

```sql
ALTER TABLE freshmen
ALTER COLUMN age TYPE smallint;
```


---

SQL 标准兼容性
-------------

`ADD`、`DROP [COLUMN]`、`SET DEFAULT` 和 `[SET DATA] TYPE`（不带 `USING`）的形式符合 SQL 标准。其他形式是 MAXIR 对 SQL 标准的扩展。此外，能够在单个 `ALTER TABLE` 命令中指定多个操作也是一种 MAXIR 扩展。

`ALTER TABLE ... DROP COLUMN` 可以用来删除表的唯一列，留下一个零列表。而标准 SQL 不允许零列表。
