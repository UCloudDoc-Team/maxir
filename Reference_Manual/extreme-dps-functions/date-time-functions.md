
# 日期/时间函数和运算符

Extreme DPS 允许你使用几个函数来处理日期/时间值。

日期和时间戳都可以比较，而间隔只能与同一数据类型的其他值进行比较。

当比较没有时区的时间戳或带有时区的时间戳时，前者的值假定为在 `TimeZone` 配置参数指定的时区中给出，并旋转到 UTC 以与后者的值（内部已经在 UTC 中）进行比较。类似地，当将其与时间戳进行比较时，日期值被假定为在 `TimeZone` 区域中表示午夜。

>说明 <br/>
>本节使用别名 `timestamptz` 来代表 `timestamp with time zone`。

---
## 日期/时间运算符

下表提供了 Extreme DPS 支持的日期/时间运算符。


| 运算符 | 语法 | 返回类型 | 描述 | 示例 | 结果 |
| :- | :- | :- | :- | :- | :- |
| `+` | `date + integer` 或 `integer + date` | `date` | 在日期上加一个整数。 | `date '2021-07-30' + integer '3'` | `date '2021-08-02'` |
| `+` | `date + interval` 或 `interval + date` | `timestamp` | 在日期上加一个间隔。 | `date '2021-07-30' + interval '6' hour` | `timestamp '2021-07-30 06:00:00'` |
| `+` | `interval + interval` | `interval` | 将一个间隔加到另一个间隔上。 | `interval '2' day + interval '6' hour` | `interval '2 day 6 hour'` |
| `+` | `timestamp + interval` 或 `interval + timestamp` | `timestamp` | 将时间戳加到一个间隔上。 | `timestamp '1998-06-12 10:30:50' + interval '1 day 2 hour 3 minute 4 second'` | `timestamp '1998-06-13 12:33:54'` |
| `+` | `timestamptz  + interval` 或 `interval + timestamptz` | `timestamptz` | 将带有时区的时间戳加到间隔上。 | `timestamptz '2023-05-18 10:10:10+08:00' + interval '1 day 2 hour 3 minute 4 second'` | `timestamptz '2023-05-19 12:13:14+08:00'` |
| `-` | `date - date` | `integer` | 从一个日期减去另一个日期。 | `date '2023-05-18' - date '2023-05-10'` | `integer '8'`（天） |
| `-` | `date - integer` | `date` | 从日期减去一个整数。 | `date '2023-05-18' - integer '15'` | `date '2023-05-03'` |
| `-` | `date - interval` | `timestamp` | 从日期减去一个间隔。 | `date '2023-05-18' - interval '10' hour` | `timestamp '2023-05-03'` |
| `-` | `timestamp - interval` | `timestamp` | 从时间戳减去一个间隔。 | `timestamp '2023-05-18 12:34:56' - interval '12 hour 34 minute 56 second'` | `timestamp '2023-05-18 00:00:00'` |
| `-` | `timestamp - timestamp` | `interval` | 从一个时间戳减去另一个时间戳。 | `timestamp '2023-05-18 12:34:56' - timestamp '2023-05-17 00:00:00'` | `interval '1 day 12 hour 34 minute 56 second'` |
| `-` | `timestamptz  - interval` | `timestamptz` | 从带有时区的时间戳减去一个间隔。 | `timestamptz '2023-05-18 12:34:56+08:00' - interval '2' hour` | `timestamptz '2023-05-18 10:34:56+08:00'` |
| `-` | `timestamptz  - timestamptz` | `interval` | 从一个带有时区的时间戳减去另一个。 | `timestamptz '2023-05-18 12:34:56+08:00' - timestamptz '2023-05-16 10:30:50+08:00'` |  ` interval '2 day 2 hour 4 minute 6 second'` |

<br/>

---

## 日期/时间函数

Extreme DPS 支持许多日期/时间函数来处理日期/时间值。下面的部分详细介绍了每个支持的函数的语法、返回类型、描述和使用示例。

 

### CURRENT_TIMESTAMP 或 NOW

返回当前事务开始时间戳。

>info
>`NOW` 是 `CURRENT_TIMESTAMP` 的别名。

#### 语法

```sql
CURRENT_TIMESTAMP()
NOW()
```

#### 参数

无。

#### 返回

一个 `timestamptz` 值。

 

### DATEDIFF

返回两个 `date` 或 `timestamp` 表达式的日期部分之间的差异。

#### 语法

```sql
DATEDIFF( <datepart>, <expression1>, <expression2> )
```

#### 参数

- *`<datepart>`*

    要比较的 `date` 或 `timestamp` 值的部分。

    可能值为 `year`、`month` 和 `hour`，如果比较的表达式中有 `timestamp`，值还可以是 `minute`、`second`、`millisecond` 或 `microsecond`。

- *`<expressionN>`*

    要比较其指定部分的 `date` 或 `timestamp` 表达式。

    注意，*`<expression1>`* 和 *`<expression2>`* 的数据类型可以不同。

#### 返回

`bigint` 值。

#### 示例

```sql
SELECT DATEDIFF(year, "0777-12-31", "1960-02-29");
-> 1183 
```

 

### MAKE_DATE

创建日期。

#### 语法

```sql
MAKE_DATE( <year>, <month>, <day> ) 
```

#### 参数

- *`<year>`*

    `integer` 表达式，指定日期的年份。

- *`<month>`*

    `integer` 表达式，指定日期的月份。

- *`<day>`*

    `integer` 表达式，指定日期的天数。

#### 返回

"YYYY-MM-DD" 格式的 `date` 值。

#### 示例

```sql
SELECT MAKE_DATE(2022, 7, 1);
-> 2022-07-01
```

 

### MAKE_INTERVAL

创建间隔。
#### 语法

```sql
MAKE_INTERVAL( [years => <years> [, months  => <months> [, weeks  => <weeks> [, days  => <days> [, hours  => <hours> [, mins  => <mins> [, secs  => <secs> ] ] ] ] ] ] ] )
```

#### 参数

- *`<years>`*

    指定年份的 `integer` 表达式。

- *`<months>`*

    指定月份的 `integer` 表达式。

- *`<weeks>`*

    指定周数的 `integer` 表达式。

- *`<days>`*

    指定天数的 `integer` 表达式。

- *`<hours>`*

    指定小时数的 `integer` 表达式。

- *`<mins>`*

    指定分钟数的 `integer` 表达式。

- *`<secs>`*

    指定秒数的 `double precision` 表达式。

上述参数的默认值均为 `0`。


#### 返回

`interval` 值。

如果有参数为指定值，则将使用其默认值 `0`。如果函数没有指定任何参数，返回的结果将是一个表示 0 秒的 `interval` 值。

#### 示例

```sql
SELECT MAKE_INTERVAL();
-> 0 years 0 mons 0 days 0 hours 0 mins 0.0 secs

SELECT MAKE_INTERVAL(60, 0, 10, 5);
-> 60 years 0 mons 75 days 0 hours 0 mins 0.0 secs

SELECT MAKE_INTERVAL(60, 0, 10, secs => 5) ;
-> 60 years 0 mons 70 days 0 hours 0 mins 5.0 secs
```

 

### MAKE_TIMESTAMP

创建不带时区的时间戳。

#### 语法

```sql
MAKE_TIMESTAMP(<year>, <month>, <day>, <hour>, <min>, <sec>)
```

#### 参数

- *`<year>`*

    指定年份的 `integer` 表达式，取值范围为 1 到 9999。

- *`<month>`*

    指定月份的 `integer` 表达式，取值范围为 1 到 12，分别对应一月到十二月。

- *`<day>`*

    指定日期的 `integer` 表达式，取值范围为 1 到 31。

- *`<hour>`*

    指定小时的 `integer` 表达式，取值范围为 0 到 23。

- *`<min>`*

    指定分钟的 `integer` 表达式，取值范围为 0 到 59。

- *`<sec>`*

    指定秒数的 `double precision` 表达式，取值范围为 0 到 60。

#### 返回

`timestamp` 值。

如果任何参数的输入值超出有效范围，将报告错误。

#### 示例

```sql
SELECT MAKE_TIMESTAMP(2022 , 10 , 5 , 12 , 34 , 56.123456);
-> 2022-07-05T12:34:56.123Z

SELECT MAKE_TIMESTAMP(2022, 11, 12 , 0, 0, 0);
-> 2022-11-12T00:00Z
```

 

### MAKE_TIMESTAMPTZ

创建一个带有时区的时间戳。

#### 语法

```sql
MAKE_TIMESTAMPTZ(<year>, <month>, <day>, <hour>, <min>, <sec>[, <timezone>])
```

#### 参数

- *`<year>`*

    指定年份的 `integer` 表达式。有效值范围从 1 到 9999。

- *`<month>`*

    指定月份的 `integer` 表达式。有效值范围从 1 到 12，分别对应一月到十二月。

- *`<day>`*

    指定日期的 `integer` 表达式。有效值范围从 1 到 31。

- *`<hour>`*

    指定小时的 `integer` 表达式。有效值范围从 0 到 23。

- *`<min>`*

    指定分钟的 `integer` 表达式。有效值范围从 0 到 59。

- *`<sec>`*

    指定秒数的 `double precision` 表达式。有效值范围从 0 到 60。

- *`<timezone>`*

    指定有效时区的字符串表达式。值可以是从 UTC 的数字偏移量或者完整的时区名称来指定位置。更多说明，请参考 [日期/时间类型](i18n/zh-Hans/docusaurus-plugin-content-docs/current/reference/data-types/datetime.md#时区) 中关于“时区”的说明。

   如果未指定 *`<timezone>`*，将使用当前时区。

#### 返回

`timestamptz` 值。

如果任何参数的输入值超出有效范围，将报告错误。

#### 示例

```sql
SELECT MAKE_TIMESTAMPTZ( 2022 , 10 , 5 , 12 , 34 , 56.123456 , 'Asia/Shanghai');
-> 2022-10-05T04:34:56.123456Z

SELECT MAKE_TIMESTAMPTZ( 2022 , 10 , 5 , 12 , 34 , 56.123456 , 'PST');
-> 2022-10-05T20:34:56.123456Z

SELECT MAKE_TIMESTAMPTZ( 2022 , 10 , 5 , 12 , 34 , 56.123456 , '+08:00');
-> 2022-10-05T04:34:56.123456Z
```

 

### TO_TIMESTAMP

将 Unix 纪元转换为带有时区的时间戳。

#### 语法

```sql
TO_TIMESTAMP(<epoch>)
```

#### 参数

*`<epoch>`*：指定需要转换的 Unix 纪元的 `double precision` 表达式。Unix 纪元表示自 1970-01-01 00:00:00+00 以来经过的秒数。

#### 返回

`timestamptz` 值。

#### 示例

```sql
SELECT TO_TIMESTAMP(1584367569.12);
-> 2020-03-16T14:06:09.120Z
```

 

### CLOCK_TIMESTAMP

返回当前函数开始执行的时间。

#### 语法

```sql
CLOCK_TIMESTAMP()
```

#### 参数

无。

#### 返回

`timestamptz` 值。

#### 示例

```sql
SELECT CLOCK_TIMESTAMP();
-> 2023-08-07T07:00:50.105348Z
```

 

### STATEMENT_TIMESTAMP

返回当前语句开始执行的时间。

#### 语法

```sql
STATEMENT_TIMESTAMP()
```

#### 参数

无。

#### 返回

`timestamptz` 值。

#### 示例

```sql
SELECT STATEMENT_TIMESTAMP();
-> 2023-08-07T07:11:42.736373Z
```

 

### DATE_TRUNC

将日期/时间值截断到指定的单位。

#### 语法

```sql
DATE_TRUNC(<unit>, timestamp <timestamp>[, timezone])

DATE_TRUNC(<unit>, interval <interval>[, timezone])
```

#### 参数

- *`<unit>`*

    一个字符串表达式，指定单位。有效的值可以是：

    - `microseconds`
    - `milliseconds`
    - `second`
    - `minute`
    - `hour`
    - `day`
    - `week`
    - `month`
    - `quarter`
    - `year`
    - `decade`
    - `century`
    - ` millennium`

- *`<timestamp>`*

    `timestamp` 表达式，指定要截断的日期/时间值。 

    如果你使用的是第一种语法形式，指定这个参数。

- *`<interval>`*

    `interval` 表达式，指定要截断的日期/时间值。 

    如果你使用的是第二种语法形式，指定这个参数。

- *`<timezone>`*

    字符串表达式，指定一个有效的时区。值可以是 UTC 的数字偏移量，或者是指定位置的完整时区名称。更多说明，请参考 [日期/时间类型](i18n/zh-Hans/docusaurus-plugin-content-docs/current/reference/data-types/datetime.md#时区) 中关于“时区”的说明。

    如果未指定 *`<timezone>`*，将使用当前时区。

#### 返回

`timestamp` 或 `timestamptz` 值。

如果任何参数的输入值超出取值范围，将报错。

#### 示例

```sql
SELECT DATE_TRUNC( 'microseconds', timestamp '2123-05-13 12:34:56.123456');
-> 2123-05-13T12:34:56.123456Z

SELECT DATE_TRUNC( 'century', timestamp '2123-05-13 12:34:56.123456');
-> 2101-01-01T00:00Z

SELECT DATE_TRUNC( 'microseconds', timestamp '2123-05-13 12:34:56.123456', '+8:00' );
-> 2123-05-13T12:34:56.123456Z
```

 

### EXTRACT

从指定的日期/时间值中提取子字段。

#### 语法

```sql
EXTRACT(<field> from timestamp <timestamp>[, <timezone>])

EXTRACT(<field> from date <date>)

EXTRACT(<field> from interval <interval>)
```

#### 参数

- *`<field>`*

    一个字符串表达式，指定要获取的子字段。值可以是：

    - `microseconds`微秒，包括小数部分，乘以 1,000,000。

    - `milliseconds`：毫秒，包括小数部分，乘以 1,000。

    - `second`：秒，包括小数部分。值是一个从 0 到 60 的 `double precision` 值。

    - `minute`：分钟。值是一个从 0 到 59 的整数。

    - `hour`：小时。值是一个从 0 到 23 的整数。

    - `dow`：一周中的第几天。值是一个从 0（周日）到 6（周六）的整数。
        
        当指定的日期/时间值为 `interval` 时，不支持此字段。
    
    - `isodow`：一周中的第几天。值是一个从 1（周一）到 7（周日）的整数。

        当指定的日期/时间值为 `interval` 时，不支持此字段。
    
    - `day`：对于 `timestamp`、`timestamptz` 和 `date`，它是一个月中的第几天，值从 1 到 31；对于 `interval`，它是天数。

    - `doy`：一年中的第几天。值从 1 到 366。

        当指定的日期/时间值为 `interval` 时，不支持此字段。
    
    - `epoch`：对于 `timestamptz`，它是自 1970-01-01 00:00:00 UTC 以来的秒数；对于 `timestamp` 和 `date`，它是自 1970-01-01 00:00:00 以来的名义秒数；对于 `interval`，它是间隔中的总秒数。

    - `month`：对于 `timestamp`、`timestamptz` 和 `date`，它是一年中的月份，值从 1 到 12；对于 `interval`，它是月份数，模 12，值从 0 到 11。
    
    - `quarter`：季度。值从 1 到 4。

        当指定的日期/时间值为 `interval` 时，不支持此字段。

    - `year`：年份。 

    - `decade`：年份字段除以 10。

    - `century`：世纪。

    - ` millennium`：千年。第三个千年在 2001 年 1 月 1 日开始。

    - `timezone`：与 UTC 的时区偏移量，以秒为单位。正值对应于 UTC 东部的时区，负值对应于 UTC 西部的时区。 

        只有在输入日期/时间值是 `timestamptz` 值时，该字段可用。
   
    - `timezone_hour`：时区偏移量的小时部分。

        只有在输入日期/时间值是 `timestamptz` 值时，该字段可用。

    - `timezone_minute`：时区偏移量的分钟部分。

        只有在输入日期/时间值是 `timestamptz` 值时，该字段可用。

- *`<timestamp>`*

    `timestamp` 或 `timestamptz` 表达式。 

    如果使用第一种语法形式，需指定该参数。

- *`<interval>`*

    `interval` 表达式。 

    如果使用第二种语法形式，需指定该参数。

- *`<timezone>`*

    指定时区的字符串表达式。值可以是 UTC 的数字偏移量，或者是指定位置的完整时区名称。更多说明，请参考 [日期/时间类型](i18n/zh-Hans/docusaurus-plugin-content-docs/current/reference/data-types/datetime.md#时区) 中关于“时区”的说明。

    如果未指定 *`<timezone>`*，将使用当前时区。



#### 返回

`double precision` 值。

#### 示例

```sql
SELECT EXTRACT(century FROM timestamp '2023-07-01 12:21:13');
-> 21

SELECT EXTRACT(day FROM timestamp '2023-07-01 12:21:13');
-> 1

SELECT EXTRACT(day FROM interval '255 days 6 minutes');
-> 255

SELECT EXTRACT(isodow FROM timestamp '2022-07-18 12:10:30');
-> 1

SELECT EXTRACT(isodow FROM date '2022-07-18');
-> 1
```

 

### FROM_UNIXTIME

将 UNIX 时间戳 `unixtime` 作为带有时区的时间戳，使用会话中的时区 `zone`。`unixtime` 是自 1970 年 1 月 1 日 00:00:00 UTC 起的秒数。

#### 语法

```sql
FROM_UNIXTIME(<unixtime>[, <zone>])
FROM_UNIXTIME(<unixtime>, <hours>, <minutes>)
```

#### 参数

- *`<unixtime>`*

    要转换的 UNIX 时间戳。

- *`<zone>`*

    无实际用途字段。如指定了该字段，系统会忽略传入值。

- *`<hours>`*

    无实际用途字段。如指定了该字段，系统会忽略传入值。

- *`<minutes>`*

   无实际用途字段。如指定了该字段，系统会忽略传入值。

#### 返回

`timestamptz` 值。

#### 示例

```sql
SELECT FROM_UNIXTIME(1615286400) AS converted_datetime;

converted_datetime
-------------------------
2021-03-09 00:00:00.000
```

---


## AT TIME ZONE

将带有时区的时间戳转换到指定的时区。
### 语法

```sql
TIMESTAMP WITH TIME ZONE <timestamptz> AT TIME ZONE <zone>

TIMESTAMP <timestamp> AT TIME ZONE <zone>
```

### 参数

- *`<timestamptz>`*

    要转换的带有时区的时间戳。在 `TIMESTAMP WITH TIME ZONE <timestamptz> AT TIME ZONE <zone>` 变体中需指定。


- *`<timestamp>`*

    要转换不带时区的时间戳。在 `TIMESTAMP <timestamp> AT TIME ZONE <zone>` 变体中需指定。

- *`<zone>`*

    要转换到的目标时区。

    注意，*`<zone>`* 的值必须是时区的完整拼写，如 `Asia/Shanghai`。不支持如 `CST` 这样的缩写。你也可以指定严格遵循 "(+/-)hh:ss" 格式的偏移量。更多说明，请参考 [日期/时间类型](i18n/zh-Hans/docusaurus-plugin-content-docs/current/reference/data-types/datetime.md#时区) 中关于“时区”的说明。

### 返回

- 如使用变体 `TIMESTAMP WITH TIME ZONE <timestamptz> AT TIME ZONE <zone>`，返回值类型为 `timestamp`。

- 如使用变体 `TIMESTAMP <timestamp> AT TIME ZONE <zone>`，返回值类型为 `timestamp with time zone`。

### 示例

```sql
SELECT TIMESTAMP WITH TIME ZONE '2023-12-15 12:00:00+00' AT TIME ZONE 'Asia/Shanghai';
-> 2023-12-15 20:00:00

SELECT TIMESTAMP '2023-12-15 12:00:00' AT TIME ZONE 'Asia/Shanghai'
-> 2023-12-15 12:00:00+08
```
