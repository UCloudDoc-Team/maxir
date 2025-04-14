
# 数组

MAXIR 支持将表中的列定义为可变长度多维数组。数组内的元素可以是任何内置的或用户自定义的基本类型、枚举类型、复合类型、范围类型等。


---

## 一维数组

为了更好的理解数组，我们先从一维数组开始。

如下 SQL 创建一个名为 `array_test` 的表，表中数据类型为一维数组：

```sql
CREATE TABLE array_test (
  f1 integer[],
  f2 integer ARRAY,
  f3 integer ARRAY[1],
  f4 double precision[],
  f5 double precision ARRAY,
  f6 double precision ARRAY[1],
  f7 decimal(10, 5)[],
  f8 decimal(10, 5) ARRAY,
  f9 decimal(10, 5) ARRAY[1],
  f10 timestamptz[],
  f11 timestamptz ARRAY,
  f12 timestamptz ARRAY[1],
  f13 text[],
  f14 text ARRAY,
  f15 text ARRAY[1]
  );
```

<br/>

### 数组声明方式

如上述示例所示，MAXIR 支持两种数组声明方式：

- 在数组元素的数据类型名称后面加上方括号（`[]`），例如示例中的 `integer[]`、`double precision[]`。

- 也可以通过数据类型名后跟 `ARRAY` 关键字来命名，例如示例中的 `integer ARRAY`、`decimal(10,5) ARRAY`。

!>**在声明数组时，需要注意:** <br/>
使用 `ARRAY` 关键字时，如果关键字后跟 `[]`，则 `[]` 中必须有具体数字。例如，MAXIR 支持 `integer ARRAY[1]`，但是不支持 `integer ARRAY[1]`。<br/>
使用 `ARRAY` 关键字声明数组类型时，只支持一维数组。<br/>
方括号（`[]`）中的数字表示当前维度中数组的长度，不过数组长度只做展示用，实际写入长度不受该数字影响。<br/>
<br/>


表格创建好之后，即可插入数据。例如：

```sql
INSERT INTO array_test VALUES (
  '{1, 2, 3}',
  '{1, 2, 3}',
  '{1, 2, 3}',
  '{1.1, 2.2, 3.3}',
  '{1.1, 2.2, 3.3}',
  '{1.1, 2.2, 3.3}',
  '{4.4, 5.5, 6.6}',
  '{4.4, 5.5, 6.6}',
  '{4.4, 5.5, 6.6}',
  '{"2020-02-02 13:45:56", "2020-03-03 14:15:16"}',
  '{"2020-02-02 13:45:56", "2020-03-03 14:15:16"}',
  '{"2020-02-02 13:45:56", "2020-03-03 14:15:16"}',
  '{"Java", "Rust"}',
  '{"Java", "Rust"}',
  '{"Java", "Rust"}'
  );
```
<br/>

完成数据插入后，即可对表中数据进行查询。如需对数组中的单个元素进行访问时，通过指定数组下标数字来完成。MAXIR 数组中的元素下标从 1 开始，即 `array[1]`。

例如，执行 `SELECT` 语句查询表中 `f1` 列中的第 1 个和第 3 个元素：

```sql
SELECT f1[1], f1[3] FROM array_test;

 f1 | f1
----+----
  1 |  3
(1 row)
```

<br/>

如需查询数组中的任意切片，需要选择 Hybrid DPS 集群运行查询。例如：

```sql
SELECT f4[2:3] FROM array_test;

    f4
-----------
 {2.2,3.3}
(1 row)
```

<br/>

!>注意 <br/>
当前，Hybrid DPS 暂不支持数据切片访问。


<br/>

关于数组切片访问的详细说明，请参考 [使用 Hybrid DPS 进行切片访问](#使用-hybrid-dps-进行切片访问)。

---
## 多维数组

多维数组的使用方式和一维数组类似。

如下 SQL 示例创建一个名为 `array3d_test` 表，表中数据类型为 3 维数组：

```sql
CREATE TABLE array3d_test (
    f1 integer[][][],
    f2 double precision[][][],
    f3 decimal(10, 5)[][][],
    f4 timestamptz[][][],
    f5 text[][][]
 );
```

<br/>


创建完成后，即可往表中插入数据，例如：

```sql
INSERT INTO array3d_test VALUES (
  '{{{1, 2}, {3, 4}}, {{5, 6}, {7, 8}}}',
  '{{{1.1, 2.2, 3.3}}, {{4.4, 5.5, 6.6}}}',
  '{{{1.1, 2.2, 3.3}}, {{4.4, 5.5, 6.6}}}',
  '{{{"2020-02-02 13:45:56"}, {"2020-03-03 14:15:16"}},  {{"2020-10-10 21:45:56"}, {"2020-11-11 22:45:56"}}}',
  '{{{"Java"}}, {{"Rust"}}}'
);
```

<br/>

该 SQL 等价于：

```sql
INSERT INTO array3d_test VALUES (
  ARRAY[ARRAY[ARRAY[1, 2], ARRAY[3, 4]], ARRAY[ARRAY[4, 5], ARRAY[7, 8]]],
  ARRAY[ARRAY[ARRAY[1.1, 2.2, 3.3]], ARRAY[ARRAY[4.4, 5.5, 6.6]]],
  ARRAY[ARRAY[ARRAY[1.1, 2.2, 3.3]], ARRAY[ARRAY[4.4, 5.5, 6.6]]],
  ARRAY[ARRAY[ARRAY['2020-02-02 13:45:56'::timestamptz], ARRAY['2020-03-03 14:15:16'::timestamptz]], ARRAY[ARRAY['2020-10-10 21:45:56'::timestamptz], ARRAY['2020-11-11 22:45:56'::timestamptz]]],
  ARRAY[ARRAY[ARRAY['Java']], ARRAY[ARRAY['Rust']]]
);
```

<br/>

数据插入完成后，即可对表中数据进行查询。

例如：

查询表中 `f1` 字段下的所有元素：

```sql
SELECT f1 FROM array3d_test;

         f1
------------------------------------
{{{1,2},{3,4}},{{5,6},{7,8}}}
(1 row)
```
<br/>

查询表中 `f1` 字段下第一维中的第一个元素以及 `f2` 字段下第一维中的第一个元素，第二维中的第二个元素，以及第三维中的第一个元素：

```sql
SELECT f1[1], f2[1][2][1] FROM array3d_test;

      f1      | f2
--------------+-----
 {{1,2},{3,4}} | 4.4
(1 row)
```

<br/>


如需查询数组中的任意切片，需要选择 Hybrid DPS 集群运行查询。

例如，执行如下 SQL 获取 `f1` 字段中第 1 维中前 2 个元素内的第 1 个元素：

```sql
SELECT f1[1:2][1:1] FROM array3d_test;

        f1
-------------------
 {{{1,2}},{{5,6}}}
(1 row)
```

<br/>

再例如，执行如下 SQL 获取 `f3` 字段中第 1 维中前 2 个元素内第 3 维中的第 1 个元素：

```sql
SELECT f3[1:][1:][3:3] FROM array3d_test;

            f3
---------------------------
 {{{3.30000}},{{6.60000}}}
(1 row)
```

<br/>

!>注意 <br/>
当前，Hybrid DPS 暂不支持数据切片访问。


<br/>

关于数组切片访问的详细说明，请参考 [使用 Hybrid DPS 进行切片访问](#使用-hybrid-dps-进行切片访问)。



### 注意事项

在多维数组中，同一条记录中的每一维中的元素必须有一样的长度，不同记录间，相同维度的元素可以有不同的长度。

MAXIR 支持的数组最大维度为 6。如建表时，指定的维度大于 6，建表不会报错。但是在数据写入过程中，会报错。例如：

```sql
-- f0 维度为 7，建表成功，不报错
CREATE TABLE test (
  f0 int[][][][][][][] 
);

-- 写入时报错
INSERT INTO test VALUES ('{{{{{{{1}}}}}}}');
ERROR:  number of array dimensions (7) exceeds the maximum allowed (6)
LINE 1: INSERT INTO test VALUES ('{{{{{{{1}}}}}}}');
```

<br/>

---

## 数组值说明

将数组值写成字面常量时，需要用大括号将元素值括起来，并用英文逗号（`,`）分隔。你可以在任何元素值周围加上双引号（`" "`），如果某元素值本身包含英文逗号（`,`）或大括号（`{ }`），则必须使用双引号进行包裹。如下为数组常量的一般格式示例：

```sql
'{<val1>, <val2>, ...}'
```
<br/>

其中，每一个 *`<val>`* 可以是一个常量也可以是一个子数组。如下是一个数组示例：

```sql
'{{1,2,3},{4,5,6},{7,8,9}}'
```

<br/>

如上示例数组是一个二维的 3 x 3 数组，包含 3 个整数子数组。


如需设置一个数组元素为 NULL，在该元素值处写 `NULL`（任何 `NULL` 的大写或小写变体都有效）。如果需要将该元素设置为字符串值 `NULL`，则需要将 `NULL` 用双引号包裹起来，即将元素设置为 `"NULL"`。

多维数组中，在同一条记录中，每一维中的元素必须有一样的长度，不同记录间，相同维度的元素可以有不同的长度：

```sql
CREATE TABLE test (
  f0 int[],
  f1 int[3],
  f2 int[3][],
  f3 int ARRAY,
  f4 int ARRAY[3]
);

INSERT INTO test VALUES (
  '{1,2,3}',             -- f0: 一维数组
  '{1,2,3}',             -- f1: 一维数组，大小为3
  '{{1,2},{3,4,5}}',     -- f2: 二维数组，内部数组维度不匹配
  '{1,2,3,4}',           -- f3: 一维数组
  '{1,2,3}'              -- f4: 一维数组，大小为3
);

ERROR:  multidimensional arrays must have array expressions with matching dimensions
```

<br/>

数组字段在指定 `NOT NULL` 时，只对最外层的一维生效，内层不生效：

```sql
CREATE TABLE test (a int[][] not null);

INSERT INTO test VALUES (NULL);

ERROR:  null value in column "a" violates not-null constraint  (seg0 192.168.215.2:7005 pid=31718)
DETAIL:  Failing row contains (null).

--重新插入数据
INSERT INTO test VALUES ('{{0}, {NULL}}');
INSERT 0 1
SELECT * FROM test;

      a
--------------
 {{0},{NULL}}
(1 row)
```

<br/>


## 使用 Hybrid DPS 进行切片访问

Hybrid DPS 支持访问数组或子数组的任意矩形切片。一个数组切片通过在一个或多个数组维度上写 `<lower-bound>:<upper-bound>` 来表示。例如 [多维数组](#多维数组) 中的如下示例：

```sql
SELECT f1[1:2][1:1] FROM array3d_test;

        f1
-------------------
 {{{1,2}},{{5,6}}}
(1 row)
```

<br/>

该查询中的 `f1[1:2][1:1]` 表示 `f1` 字段中第 1 维中前 2 个元素内的第 1 个元素：

在切片中，`<lower-bound>` 或 `<upper-bound>` 可以省略，二者分别会数组下标的下限或上限替代。例如，`[:]`、`[1:]` 和 `[:2]` 都是正确的写法。例如 [多维数组](#多维数组) 中的如下示例：

```sql
SELECT f3[1:][1:][3:3] FROM array3d_test;

            f3
---------------------------
 {{{3.30000}},{{6.60000}}}
(1 row)
```

<br/>

该查询中的 `f3[1:][1:][3:3]` 表示 `f3` 字段中第 1 维中前 2 个元素内第 3 维中的第 1 个元素。


此外，如果任何维度被写成了切片，那么所有维度都会被视为切片。即任何只有一个数字的维度都会被视为从 1 到该数字的切片，即 `[2]` 会被视为 `[1:2]`。


---
## 与 PostgreSQL 的差异


在 PostgreSQL 中的数组长度只做展示用，实际写入的长度不受其影响。而 MAXIR 要求同一列中的所有行的维度必须与 Schema 中定义的一致。例如：

```sql
CREATE TABLE test (
  f0 int[],       -- 维度为 1
  f1 int[][][],   -- 维度为 3
  f2 int ARRAY,   -- 维度为 1
  f3 int ARRAY[3] -- 维度为 1
);

-- 维度一致，写入成功
INSERT INTO test VALUES ('{1}', '{{{1}}}', '{1}', '{1}');

-- 维度不一致，写入失败
INSERT INTO test VALUES ('{{1}}', '{{1}}', '{{1}}', '{{1}}');
```




