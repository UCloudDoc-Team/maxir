DECLARE
=====

定义一个游标。

>info 重要提示
>如果查询中涉及到使用 `DECLARE` 声明的游标，那么该查询只能在 Hybrid DPS 集群上执行。

---

语法
--------

```sql
DECLARE <name> [INSENSITIVE] [NO SCROLL] [PARALLEL RETRIEVE] CURSOR 
     [{WITH | WITHOUT} HOLD] 
     FOR <query> 
```


---

描述
----------

`DECLARE` 允许用户创建一个游标，可以用来一次从更大的查询中检索少量的行。游标可以使用 [FETCH](/maxir/Reference_Manual/sql-commands/fetch.md) 以文本格式返回数据。

>note
>本文描述了在 SQL 命令级别使用游标的用法。

普通游标以文本格式返回数据，与 `SELECT` 生成的结果相同。由于数据以二进制格式本地存储，系统必须进行转换以生成文本格式。一旦信息以文本形式返回，客户端应用可能需要将其转换为二进制格式以进行操作。此外，文本格式的数据通常比二进制格式的数据大。二进制游标以可能更容易操作的二进制表示形式返回数据。然而，如果你打算将数据以文本形式显示，以文本形式检索它将节省你在客户端的一些工作。

举例来说，如果查询从整数列返回值 1，你会得到一个默认游标的字符串 1，而使用二进制游标你会得到一个包含值内部表示（以大端字节顺序）的 4 字节字段。

应谨慎使用二进制游标。许多应用，包括 psql，并未准备好处理二进制游标，并希望数据以文本格式返回。

>note
>当客户端应用使用 '扩展查询' 协议发出一个 `FETCH` 命令时，Bind 协议消息指定是否以文本或二进制格式检索数据。此选择覆盖了游标的定义方式。因此，在使用扩展查询协议时，二进制游标的概念已经过时 —— 任何游标都可以被视为文本或二进制。



---
参数
----------

- *`<name>`*

    游标的名称。

- **`INSENSITIVE`**

    表示从游标检索的数据应不受到游标存在期间基础表的更新的影响。在 MAXIR 中，所有游标都是不敏感的。这个关键字目前没有效果，仅为了与 SQL 标准的兼容性存在。

- **`NO SCROLL`**

    游标不能用于以非顺序方式检索行。这是 MAXIR 中的默认行为，因为不支持可滚动的游标（`SCROLL`）。

- **`WITH HOLD`** 或 **`WITHOUT HOLD`**
    
    `WITH HOLD` 指定在创建游标的事务成功提交后，游标可以继续被使用。`WITHOUT HOLD` 指定游标不能在创建它的事务之外被使用。`WITHOUT HOLD` 是默认的。

    >note
    >MAXIR 不支持声明带有 `WITH HOLD` 子句的 `PARALLEL RETRIEVE` 游标。当查询包含 `FOR UPDATE` 或 `FOR SHARE` 子句时，也不能指定 `WITH HOLD`。


- *`<query>`*

    将提供由游标返回的行的 [SELECT](/maxir/Reference_Manual/sql-commands/select.md) 或 [VALUES](/maxir/Reference_Manual/sql-commands/values.md) 命令。
    
    
    - 不能引用视图或外部表。
    
    - 只引用一个表。 

        表必须是可更新的。例如，以下是不可更新的：表函数、返回集函数、仅追加表、列式表。

    - 不能包含以下任何内容：
    
        - 分组子句
        - 诸如 `UNION ALL` 或 `UNION DISTINCT` 的集合操作
        - 排序子句
        - 窗口子句
        - 连接或自连接 

        在 `SELECT` 命令中指定 `FOR UPDATE` 子句可以防止其他会话在行被获取到和更新之间更改行。如果没有 `FOR UPDATE` 子句，如果行自创建游标以来已经改变，随后使用 `WHERE CURRENT OF` 子句的 `UPDATE` 或 `DELETE` 命令将无效。

        >note
        >在 `SELECT` 命令中指定 `FOR UPDATE` 子句会锁定整个表，而不仅仅是选定的行。

- **`FOR READ ONLY`**

    指定游标在只读模式中使用。

---

使用说明
--------

除非指定了 `WITH HOLD`，否则此命令创建的游标只能在当前事务中使用。因此，`DECLARE` 在事务块外部没有 `WITH HOLD` 是无用的：游标只能存活到语句完成。因此，如果此命令在事务块外部使用，MAXIR 会报错。使用 `BEGIN` 和 `COMMIT`（或 `ROLLBACK`）来定义一个事务块。


如果指定了 `WITH HOLD` 并且创建游标的事务成功提交，游标可以继续被同一会话中的后续事务访问。（但是，如果创建事务提前结束，游标将被删除。）使用 `WITH HOLD` 创建的游标在对其发出显式的 `CLOSE` 命令，或会话结束时关闭。在当前的实现中，由持有游标表示的行被复制到临时文件或内存区域，以便它们对后续事务仍然可用。


如果你在事务中使用 `DECLARE` 命令创建一个游标，你不能在事务中使用 `SET` 命令，直到你使用 `CLOSE` 命令关闭游标。


MAXIR 目前不支持可滚动的游标。你只能使用 `FETCH` 或 `RETRIEVE` 来向前移动游标位置，不能向后。


`DECLARE ... FOR UPDATE` 不支持追加优化的表。


---
示例
--------

声明一个游标：

```sql
DECLARE mycursor CURSOR FOR SELECT * FROM mytable;
```

---


SQL 标准兼容性
-------------

SQL 标准只允许在嵌入式 SQL 和模块中使用游标。MAXIR 允许游标在交互式中使用。

MAXIR 没有为游标实现 `OPEN` 语句。声明时游标被视为打开的。

SQL 标准允许游标向前和向后移动。所有 MAXIR 的游标都只能向前移动（不可滚动）。

二进制游标是 MAXIR 的扩展。
