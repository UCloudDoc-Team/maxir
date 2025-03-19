# CREATE FUNCTION

创建函数。

:::info 重要提示
如果查询中包含用户自定义函数（通过 `CREATE FUNCTION` 语句创建），那么该查询只能在 Hybrid DPS 集群上执行。
:::

---

## 语法

```sql
CREATE [OR REPLACE] FUNCTION <name>    
    ( [ [<arg_mode>] [<arg_name>] <arg_type> [ { DEFAULT | = } <default_expr> ] [, ...] ] )
      [ RETURNS <return_type> 
        | RETURNS TABLE ( <column_name> <column_type> [, ...] ) ]
  { LANGUAGE <lang_name>
    | TRANSFORM { FOR TYPE <type_name> } [, ... ]
    | WINDOW
    | { IMMUTABLE | STABLE | VOLATILE }
    | { CALLED ON NULL INPUT | RETURNS NULL ON NULL INPUT | STRICT }
    | { [ EXTERNAL ] SECURITY INVOKER | [ EXTERNAL ] SECURITY DEFINER }
    | PARALLEL { UNSAFE | RESTRICTED | SAFE }
    | COST <execution_cost>
    | ROWS <result_rows>
    | SET <config_param> { TO <value> | = <value> | FROM CURRENT }
    | AS '<definition>'
  } ...
```

---

## 描述

要创建函数，你必须拥有要使用的参数类型和返回类型的 `USAGE` 权限。函数的创建者默认为函数的所有者。

MAXIR 通过函数名称和输入参数类型来唯一标识函数。因此，具有相同名称但不同输入参数类型的函数是两个不同的函数，可以在同一 Schema 中共存。

`CREATE FUNCTION` 只能用于创建函数。如需替换现有函数，请使用 `CREATE OR REPLACE FUNCTION`。

只有函数的所有者才能运行 `CREATE OR REPLACE FUNCTION` 来替换函数。该命令运行后，函数的所有者和权限保持不变，而其他参数使用命令中指定的值。

如果删除函数后进行重新创建，新建的函数与被删除的函数是两个不同的实体。你还需要删除依赖于被删除函数的对象，包括规则和视图。如果你运行 `CREATE OR REPLACE FUNCTION` 直接更改函数定义，则不会影响对象依赖性。

更多关于创建函数的信息，参考 PostgreSQL 文档中的 [User Defined Functions](https://www.postgresql.org/docs/12/xfunc.html)。

:::info
如需更新函数的名称或参数类型，请先删除函数并重新创建。如果使用 `CREATE OR REPLACE UFNCTION`，将创建一个新函数，而不是替换现有函数。

`CREATE OR REPLACE FUNCTION` 不能用于更新函数的返回类型。
:::

---

## 参数

- _`<name>`_

    函数的名称，支持使用 Schema 名称进行限定，在指定的 Schema 中进行创建。否则，将在当前 Schema 中创建函数。

- _`<arg_mode>`_

    参数的模式。可用的值是 `IN`、`OUT`、`INOUT` 和 `VARIADIC`。如果你省略此参数，将使用默认值 `IN`。`VARIADIC` 参数只能跟随 `OUT` 参数。 

    `OUT` 和 `INOUT` 参数不能与 `RETURNS TABLE` 表示法一起使用。

- _`<arg_name>`_

    参数的名称。SQL 和 PL/pgSQL 允许你在函数体中使用名称。其他语言仅仅用输入参数名称作为文档的一部分。你可以使用输入参数名称来提高可读性。

    输出参数的名称定义了结果行类型中的列名。如果省略，系统将选择默认的列名称。

- _`<arg_type>`_

    参数的数据类型。你可以使用 Schema 来限定数据类型。支持的数据类型可以是基础类型、组合类型和域类型，也可以参考表列的数据类型。

    对于某些实现语言，也允许使用伪类型，这样你就可以指定像 `cstring` 这样的伪类型。伪类型表明实际的参数类型要么未完全指定，要么超出了常规 SQL 数据类型的集合。
    
    当你引用列的数据类型时，可以使用 *table_name_._column_name*__%TYPE__ 格式来指定。这样做的好处是，创建的函数可以不受表定义更新的影响。

- _`<default_expr>`_

    如果参数未指定，生成默认值的表达式。表达式必须可强制转换为参数的参数类型。请注意，只有 `IN` 和 `INOUT` 参数可以有默认值。 

    在参数列表中，每个跟随有默认值的参数的 `IN` 或 `INOUT` 参数必须有默认值。 

- _`<return_type>`_

    返回值的数据类型。你可以使用 Schema 来限定数据类型。支持的数据类型可以是基础类型、组合类型和域类型，也可以参考表列的数据类型。
    
    对于某些实现语言，也允许使用伪类型，这样你就可以指定像 `cstring` 这样的伪类型。 

    如果函数应返回空值，将 *`<return_type>`* 设置为 `void`。

    如果函数包含 `OUT` 或 `INOUT` 参数，你可以省略 `RETURNS` 子句。如果存在，你必须将 *`<return_type>`* 设置为：
    
    - 如果存在多个 `OUT` 或 `INOUT` 参数，设置为 `RECORD`。

    - 如果只存在一个这样的参数，设置为 `OUT` 或 `INOUT` 参数的相同类型。

    此外，你可以使用 `SETOF` 修饰符来指定函数将返回一组项目，而非一个。

    要引用列的类型，以 *table_name*.*column_name***%TYPE** 格式指定类型。

- _`<column_name>`_

    输出列的名称。此参数在 `RETURNS TABLE` 语法中使用。`RETURNS TABLE` 是在 `CREATE FUNCTION` 命令中声明有名字的 `OUT` 参数的替代方法，该命令使用了 `SETOF` 修饰符。

- _`<column_type>`_

    输出列的数据类型。此参数在 `RETURNS TABLE` 语法中使用。

- _`<lang_name>`_

    实现函数所用的语言名称。值可以是 `SQL`、`C`、`internal`，或用户定义的过程语言的名称。  

- **`TRANSFORM { FOR TYPE <type_name> } [, ... ]`** 

    函数调用必须应用的转换列表。转换用于在 SQL 类型和特定语言的数据类型之间转换。在大多数情况下，内置类型在过程语言实现中是硬编码的。因此，此参数是不必要的。即使特定的过程实现不知道如何处理类型，并且没有提供转换，它也会在转换数据类型时使用默认行为。默认行为随实现而异。 

- **`WINDOW`**

    指定函数是窗口函数。只有在 C 语言中实现的函数才能成为窗口函数。 

    当你使用 `CREATE OR REPLACE FUNCTION` 更改函数的定义时，如果函数有 `WINDOW` 属性，该属性不能被修改。

- **`IMMUTABLE`**, **`STABLE`**, 或 **`VOLATILE`**

    指定函数的易变性。如果没有设置，默认使用 `VOLATILE`。 

    `IMMUTABLE` 函数不能修改数据库，并且在给定相同的参数值时返回相同的结果。它仅使用直接在参数列表中提供的信息，不需要扫描数据库。当查询使用常量参数调用函数时，参数会立即被函数值替换。如果是 `IMMUTABLE` 函数，你必须声明该函数为 `IMMUTABLE`，因为 MAXIR 对使用 `VOLATILE` 函数的使用实施了限制。

    `STABLE` 函数不能修改数据库，并且在给定相同的参数值时在单个表扫描中返回相同的结果。

    但是，当在不同的 SQL 语句中使用时，相同的 `STABLE` 函数使用相同的参数值可以返回不同的结果。`STABLE` 适用于结果取决于数据库查找和参数变量的函数。 
    
    `current_timestamp()` 系列的函数都是稳定函数，因为它们的函数值在单个事务内不会改变。
    
    一个 `VOLATILE` 函数可以在给定相同的参数值的单个表扫描中返回不同的结果，比如 `random()` 和 `timeofday()`。`VOLATILE` 函数的数量很少。不能对 `VOLATIBLE` 函数进行优化。具有副作用的函数，例如 `setval()`，也必须被归类为 `VOLATILE`，以防止对调用进行优化，即使是在结果是可预测的前提下。

- **`CALLED ON NULL INPUT`**, **`RETURNS NULL ON NULL INPUT`**, 或 **`STRICT`**

    当存在空输入参数时，函数的行为。可能的值有：

    - `CALLED ON NULL INPUT`：表示函数将正常调用，这是默认值。 

    - `RETURNS NULL ON NULL INPUT` 或 `STRICT`：表示函数不会运行，直接返回空值。

- **`[EXTERNAL] SECURITY INVOKER`** 或 **`[EXTERNAL] SECURITY DEFINER`**

    需要哪个用户的权限来运行函数。
    
    - `SECURITY INVOKER`：调用函数的用户。

    - `SECURITY DEFINER`：创建函数的用户。

    `EXTERNAL` 关键字为可选项。这个选项对 MAXIR 中的所有函数都是可用的，而不仅仅是外部函数。你可以指定它以符合 SQL 标准。 

- **`PARALLEL`**

    指定函数是否可以在并行模式下执行。支持的模式包括 `PARALLEL UNSAFE`、`PARALLEL RESTRICTED` 和 `PARALLEL SAFE`。

    - `PARALLEL UNSAFE`：表示函数无法在并行模式下执行，且为默认值。使用 `PARALLEL UNSAFE` 的 SQL 语句的执行计划只能是串行的。 

        如果函数将修改任何数据库状态、改变事务、访问序列或尝试进行持久性的设置更改，则将函数标记为 `PARALLEL UNSAFE`。

    - `PARALLEL RESTRICTED`：表示只有并行组 Leader 可以在并行模式下执行函数。 

       如果函数将访问临时表、客户端连接状态、游标、预备语句或其他系统无法在并行模式下同步的后端本地状态，则将函数标记为 `PARALLEL RESTRICTED`。

    - `PARALLEL SAFE`：表示函数可以在并行模式下无限制地执行。

    在大多数情况下，一个定义为 `PARALLEL SAFE` 但实际上是不安全或受限的函数，或者一个定义为 `PARALLEL STRICT` 但是不安全的函数，在函数被用于并行查询时会产生错误。C 语言函数理论上可以在被误标记时表现出完全未定义的行为，因为系统无法防止任意的 C 代码，但在大多数可能的情况下，结果不会更差。如果你不确定一个函数是否是并行安全的，就将其标记为 `UNSAFE`，这是默认值。

- **`COST <execution_cost>`**

    函数的预估执行成本，以 [cpu_operator_cost](https://www.postgresql.org/docs/12/runtime-config-query.html) 衡量。值必须是正数。如果函数返回一个集合，*`<execution_cost>`* 表示每个返回行的成本。如果未指定成本，C 语言和内部函数默认为 1 单位，而其他语言的函数默认为 100 单位。较大的值会使规划器尽量避免比必要更频繁地评估函数。

- **`ROWS <result_rows>`**

    预计返回的行数。值必须是正数。此参数只能在声明函数返回集合时使用。默认值为 1000。

- *`<config_param>`* 和 *`<value>`*

    你可以使用 `SET` 子句为函数指定会话配置参数的值。以这种方式指定的配置参数设置仅对当前会话有效。函数退出时，配置参数的值恢复为原始值。 

    `SET FROM CURRENT` 将配置参数的当前值保存为进入函数时要应用的值。

    如果函数有一个 `SET` 子句，任何在函数内部为同一变量执行的 `SET LOCAL` 命令只适用于该函数。一旦函数退出，配置参数将恢复其原始值。然而，一个普通的 `SET` 命令（没有 `LOCAL`）会覆盖 `SET` 子句，就像它会覆盖前一个 `SET LOCAL` 命令一样。这个命令的效果在函数退出后会持续，除非当前事务被回滚。

    有关更多信息，请参阅 [SET](set.md)。

- *`<definition>`*

    函数的定义。值必须是一个字符串常量，表示内部函数名称、对象文件的路径、SQL 命令或基于实现函数所用语言的文本。

    我们建议你使用美元引用来指定 *`<definition>`* 的值，否则你必须通过将它们加倍来转义函数定义中的单引号 (' ') 或反斜杠 (\)。有关美元引用的详细信息，请参阅 PostgreSQL 文档中的 [Dollar-Quoted String Constraints](https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-DOLLAR-QUOTING)。

---

## 重载

函数重载 (Overloading) 表示具有相同名称但不同输入参数类型的函数可以同时存在。这种能力在某些情况下引入了安全预防措施。有关更多信息，请参阅 PostgreSQL 文档中的 [Functions](https://www.postgresql.org/docs/current/typeconv-func.html)。

然而，如果两个函数具有相同的名称和输入参数类型，但 `OUT` 参数类型不同，它们被视为相同的函数。例如：

```sql
CREATE FUNCTION add_user(varchar)...
CREATE FUNCTION add_user(varchar, out text)...
```

这两个声明存在冲突。

如果两个函数具有相同的名称和不同的参数类型列表，它们在创建时不被认为是冲突的。然而，如果提供了默认值，它们可以在使用时产生冲突。例如：

```sql
CREATE FUNCTION add_user(varchar) ...
CREATE FUNCTION add_user(varchar, varchar default 'fresh') ...
```

调用 `add_user('senior')` 将失败，因为 Planner 不知道应该使用哪个函数。

---
## 使用注意事项

`CREATE FUNCTION` 允许对输入参数和返回值使用完整的 SQL 类型语法。然而，它忽略任何带括号的类型修饰符。 

在使用`CREATE OR REPLACE FUNCTION`替换现有函数时，更改参数名称存在一些限制。已经分配给任何输入参数的名称不支持修改（虽然你可以为之前没有名称的参数添加名称）。如果有多个输出参数，你不能更改输出参数的名称，因为这将改变描述函数结果的匿名复合类型的列名称。这些限制是为了确保当替换函数时，现有的函数调用不会停止工作。

如果严格函数有一个 `VARIADIC` 参数，严格性检查将把变差数组视为一个整体。这表明，除非数组中的所有元素都为空，否则函数会正常运行。

### 安全地编写 SECURITY DEFINER 函数

`SECURITY DEFINER` 函数使用创建函数的用户的权限运行。因此，你必须确保这样的函数不被误用。出于安全原因，期望 `search_path` 不包含任何可以被不受信任的用户修改的 Schema。这有助于防止恶意活动，比如创建对象来掩盖函数使用的对象。临时表 Schema 在这种情况下需要特别关注，因为它总是首先被搜索，而且任何人都可以在其中写入。为了进一步提升安全性，你可以通过将 `pg_temp` 设置为 `search_path` 中的最后一个条目来更改临时 Schema 的搜索方式。以下是一个确保安全使用的函数的示例：

```sql
CREATE FUNCTION check_password(uname TEXT, pass TEXT)
RETURNS BOOLEAN AS $$
DECLARE passed BOOLEAN;
BEGIN
        SELECT  (pwd = $2) INTO passed
        FROM    pwds
        WHERE   username = $1;

        RETURN passed;
END;
$$  LANGUAGE plpgsql
    SECURITY DEFINER
    -- Set a secure search_path: trusted schema(s), then 'pg_temp'.
    SET search_path = admin, pg_temp;
```

默认情况下，新函数被赋予 `PUBLIC` 的执行权限。更多详细内容，请见 [GRANT](grant.md)。出于安全考虑，你可能希望将安全定义者函数的使用限制在特定用户上。为此，你需要撤销 `PUBLIC` 的执行权限，然后有选择性地将其授予期望的用户。为了防止函数对所有人可用，我们建议你在一个单独的事务中创建并设置权限。例如：

```sql
BEGIN;
CREATE FUNCTION check_password(uname TEXT, pass TEXT) ... SECURITY DEFINER;
REVOKE ALL ON FUNCTION check_password(uname TEXT, pass TEXT) FROM PUBLIC;
GRANT EXECUTE ON FUNCTION check_password(uname TEXT, pass TEXT) TO admins;
COMMIT;
```

---

## 示例

创建一个函数用于两个整数相加：

```sql
CREATE FUNCTION add(integer, integer) RETURNS integer
    AS 'select $1 + $2;'
    LANGUAGE SQL
    IMMUTABLE
    RETURNS NULL ON NULL INPUT;
```

使用 PL/pgSQL 创建一个函数来增加一个整数，该函数使用了参数名称：

```sql
CREATE OR REPLACE FUNCTION increment(i integer) RETURNS 
integer AS $$
        BEGIN
                RETURN i + 1;
        END;
$$ LANGUAGE plpgsql;
```

创建一个函数以返回一个包含多个输出参数的记录：

```sql
CREATE FUNCTION dup(in int, out f1 int, out f2 text)
    AS $$ SELECT $1, CAST($1 AS text) || ' is text' $$
    LANGUAGE SQL;

SELECT * FROM dup(42);
```

你可以使用一个明确命名的复合类型来更详细地做同样的事情：

```sql
CREATE TYPE dup_result AS (f1 int, f2 text);

CREATE FUNCTION dup(int) RETURNS dup_result
    AS $$ SELECT $1, CAST($1 AS text) || ' is text' $$
    LANGUAGE SQL;

SELECT * FROM dup(42);
```

另一种返回多列的方法是使用 `TABLE` 函数：

```sql
CREATE FUNCTION dup(int) RETURNS TABLE(f1 int, f2 text)
    AS $$ SELECT $1, CAST($1 AS text) || ' is text' $$
    LANGUAGE SQL;

SELECT * FROM dup(42);
```

然而，一个 TABLE 函数与前面的例子不同，因为它实际上返回一个记录集，而不仅仅是一个记录。



使用多态类型返回一个 `ENUM` 数组：

```sql
CREATE TYPE rainbow AS ENUM('red','orange','yellow','green','blue','indigo','violet');
CREATE FUNCTION return_enum_as_array( anyenum, anyelement, anyelement ) 
    RETURNS TABLE (ae anyenum, aa anyarray) AS $$
    SELECT $1, array[$2, $3] 
$$ LANGUAGE SQL STABLE;

SELECT * FROM return_enum_as_array('red'::rainbow, 'green'::rainbow, 'blue'::rainbow);
```

---
## SQL 标准兼容性

`CREATE FUNCTION` 存在于 SQL:1999 和之后的版本中。MAXIR 中的 `CREATE FUNCTION` 与 SQL 标准中的 `CREATE FUNCTION` 类似。MAXIR 中的 `CREATE FUNCTION` 的属性和可用语言不可移植。
