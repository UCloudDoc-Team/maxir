MOVE
=======

改变游标的位置。


---

语法
--------

```sql
MOVE [ <forward_direction> [ FROM | IN ] ] <cursor_name>
其中 <forward_direction> 可以为空或者以下任一选项：
    NEXT
    FIRST
    LAST
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

与 [FETCH](/maxir/Reference_Manual/sql-commands/fetch.md) 不同，`MOVE` 不检索数据也不返回行。

MAXIR 不支持可滚动的游标。因此，`MOVE` 只能用来将游标向前移动。

>note
>你不能使用 `MOVE` 来重新定位并行检索游标。


---

参数
----------

参数与 `FETCH` 语句的参数相同。有关更多详细信息，请参阅 [FETCH](/maxir/Reference_Manual/sql-commands/fetch.md#参数) 中的 "参数" 部分。


---

输出
--------

如果 `MOVE` 成功，返回的命令标签形式如下：

```sql
MOVE <count>
```

*`<count>`* 指定具有相同参数的 `FETCH` 语句将返回的行数。



---

示例
--------

1. 启动事务：

    ```sql
    BEGIN;
    ```
2. 声明游标：
    ```sql
    DECLARE cursor1 CURSOR FOR SELECT * FROM myplaylist;
    ```
3. 将游标移动到下一行：
    ```sql
    MOVE NEXT in cursor1;
    ```
4. 关闭游标并结束事务：
    ```sql
    CLOSE mycursor;
    COMMIT;
    ```


---

SQL 标准兼容性
----------

SQL 标准不支持 `MOVE`。
