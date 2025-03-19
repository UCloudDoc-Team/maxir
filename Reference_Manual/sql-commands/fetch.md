FETCH
=====

使用游标从查询中检索行。


---

语法
--------

```sql
FETCH [ <forward_direction> { FROM | IN } ] <cursor_name>
其中 <forward_direction> 可以为空或以下之一：
    NEXT
    FIRST
    ABSOLUTE <count>
    RELATIVE <count>
    <count>
    ALL
    FORWARD
    FORWARD <count>
    FORWARD ALL
```


---

描述
--------

要执行 `FETCH`，你必须使用现有的游标。游标不能是并行检索游标。

MAXIR 不支持可滚动的游标。因此，`FETCH` 只能用于向前移动游标。

游标有一个由 `FETCH` 使用来获取结果的位置。位置可以在结果集的第一行之前、任何行上，或者在结果集的最后一行之后。当游标被创建时，位置在第一行之前。然后每次完成抓取后，位置都会移动到最近抓取的行。在 `FETCH` 用完可用行后，游标会定位在最后一行之后。`FETCH ALL` 会让游标定位在最后一行之后。

`NEXT`、`FIRST`、`ABSOLUTE` 和 `RELATIVE FETCH` 形式抓取单个行并相应地移动游标。如果没有找到行，将返回空结果。

`FORWARD FETCH` 检索给定数量的行，并将游标移动到最后返回的行。游标不能向后移动，因为 MAXIR 不支持可滚动的游标。

`RELATIVE 0` 和 `FORWARD 0` 在不移动游标的情况下抓取当前行。`FETCH` 语句的成功执行返回包含抓取行数的命令标签。


---

参数
----------

- *`<forward_direction>`*

    抓取方向和要抓取的行数。目前，只支持向前抓取，值可以是 `NEXT`、`FIRST`、`ABSOLUTE <count>`、`RELATIVE <count>`、`<count>`、`ALL`、`FORWARD`、`FORWARD <count>` 和 `FORWARD ALL`。

    - `NEXT`：指定检索下一行。如果未指定 `forward_direction`，则默认使用 `NEXT`。
    - `FIRST`：指定检索查询的第一行。此方向只能用于第一次使用游标的 `FETCH`。`FIRST` 的效果与 `ABSOLUTE 0` 相同。
    - `ABSOLUTE`：指定检索查询的给定行。*count* 的值必须大于游标当前位置，以保持游标位置向前移动。
    - `RELATIVE`：指定检索从当前游标位置开始的 *count* 行。*count* 的值必须大于当前位置，以保持游标位置向前移动，或者为 `0` 以再次检索当前行。
    - *`<count>`*：指定检索下一个 count 数量的行。它的效果与 `FORWARD <count>` 相同。
    - `ALL`：指定检索所有剩余行。它的效果与 `FORWARD ALL` 相同。
    - `FORWARD`：指定检索下一行。它的效果与 `NEXT` 相同。
    - `FORWARD <count>`：指定检索下一个 *`<count>`* 数量的行。如果 *`<count>`* 设置为 `0`，将再次检索当前行。
    - `FORWARD ALL`：指定检索所有剩余行。

- *`<cursor_name>`*

    游标的名称。

---

输出
----------

如果 `FETCH` 成功，返回的命令标签如下所示：

```sql
FETCH <count>
```

*`<count>`* 指定抓取的行数。


---

示例
----------

1. 启动事务：

    ```sql
    BEGIN;
    ```
2. 设置游标：
    ```sql
    DECLARE mycursor CURSOR FOR SELECT * FROM films;
    ```
3. 在游标 `mycursor` 中抓取前 5 行：
    ```sql
    FETCH FORWARD 5 FROM mycursor;
    code  |          title          | did | date_prod  |   kind   |  len
    -------+-------------------------+-----+------------+----------+-------
    BL101 | The Third Man           | 101 | 1949-12-23 | Drama    | 01:44
    BL102 | The African Queen       | 101 | 1951-08-11 | Romantic | 01:43
    JL201 | Une Femme est une Femme | 102 | 1961-03-12 | Romantic | 01:25
    P_301 | Vertigo                 | 103 | 1958-11-14 | Action   | 02:08
    P_302 | Becket                  | 103 | 1964-02-03 | Drama    | 02:28
    ```
4. 关闭游标并结束事务：
    ```sql
    CLOSE mycursor;
    COMMIT;
    ```
5. 更改表 `films` 中位于 `c_films` 游标当前位置的行的 `kind` 列：
    ```sql
    UPDATE films SET kind = 'Dramatic' WHERE CURRENT OF c_films;
    ```

---

SQL 标准兼容性
----------

MAXIR 中的 `FETCH` 允许你交互式地使用游标，而 SQL 标准中的 `FETCH` 只能在嵌入式 SQL 和模块中使用。

MAXIR 中的 `FETCH` 与 SQL 标准完全兼容。此外，MAXIR 中的 `FETCH`，如我们上面所述，返回数据的方式与 `SELECT` 语句相同。

此外，SQL 标准只允许在游标名称前放置 `FROM`。MAXIR 也允许你使用 `IN`。
