# 数学函数和运算符

数学函数和运算符对一个或两个输入表达式进行求值，产生一个数值结果。

目前，Extreme DPS 只允许数值类型作为数学函数和运算符的输入。

--- 

## 数学运算符

下表列出了 Extreme DPS 支持的数学运算符。注意，数学运算符的返回类型与输入类型相同。

| 运算符 | 语法 | 描述 |
| :- | :- | :- |
| `+` | `a + b` | 将 *a* 和 *b* 相加。 |
| `-` | `a - b` | 从 *a* 中减去 *b*。 |
| `*` | `a * b` | *b* 乘以 *a*。 |
| `/` | `a / b` | *b* 除以 *a*。 |
| `%` | `a % b` | 计算 *a* 除以 *b* 的余数，和 `MOD` 效果相同。 |

<br/>

---

## 数学函数

下表列出了 Extreme DPS 支持的数学函数。部分函数根据参数类型的不同而有所差异，因此提供了多种形式。下表中的 `numeric` 表示 Extreme DPS 支持的任何 [数值类型](i18n/zh-Hans/docusaurus-plugin-content-docs/current/reference/data-types/numeric.md)，即 `bigint`、`integer`、`smallint`、`double precision`、`real` 和 `decimal`。

| 函数 | 返回类型 | 描述 | 示例 | 结果 |
| :- | :- | :- | :- | :- |
| `abs(numeric)` | 与输入相同 | 返回给定表达式的绝对值。 | `abs(-3.5)` | `3.5` |
| `cbrt(double precision)` | `double precision` | 返回给定表达式的立方根。 | `cbrt(75.0)` | `5` |
| `ceil(double precision or decimal)` | `double precision` | 返回大于或等于给定表达式的最近整数。 | `ceil(-3.5)` | `-3` |
| `ceiling(double precision or decimal)` | `double precision` | 返回大于或等于给定表达式的最近整数。 | `ceiling(-3.5)` | `-3` |
| `degrees(double precision)` | `double precision` | 将弧度转换为度。 | `degrees(0.5)` | `28.6478897565412` |
| `exp(double precision)` | `double precision` | 返回给定表达式的指数。 | `exp(1.0)` | `2.71828182845905` |
| `floor(double precision)` | `double precision` | 返回小于或等于表达式的最近整数。 | `floor(-42.8)` | `-43` |
| `ln(double precision)` | `double precision` | 返回给定表达式的自然对数。 | `ln(2.0)` | `0.693147180559945` |
| `log(double precision)` | `double precision` | 返回给定表达式的对数。 | `log(100.0)` | `2` |
| `log10(double precision)` | `double precision` | 返回给定表达式的以 10 为底的对数。 | `log10(100.0)` | `2` |
| `mod(y, x)` | 与输入相同 | 返回 *`<y>`* 除以 *`<x>`* 后的余数。*`<y>`* 和 *`<x>`* 可以是 `smallint`、`integer` 或 `bigint`、`real` 或 `double precision`。 | `mod(9, 4)` | `1` |
| `pi()` | `double precision` | 返回 3.14159265358979，这是数学常数 pi，精确到 15 位。 | `pi()` | `3.14159265358979` | 
| `power(a double precision, b double precision)` | `double precision` | 返回 *`<a>`* 的 *`<b>`* 次方。 | `power(9.0, 3.0)` | `729` | 
| `radians(double precision)` | `double precision` | 将度转换为弧度。 | `radians(45.0)` | `0.785398163397448` |
| `round(double precision or decimal)` | 与输入相同 | 将表达式四舍五入到最接近的整数。 | `round(42.4)` | `42` |
| `round(v decimal, s integer)` | `decimal` | 将 *`<v>`* 四舍五入到 *`<s>`* 位小数。 | `round(42.4382, 2)` | `42.44` |
| `sign(double precision or decimal)` | `decimal` | 返回参数的符号。`-1` 表示负，`1` 表示正，`0` 表示 0。 | `sign(-8.4)` | `-1` |
| `sqrt(double precision)` | `decimal` | 返回参数的平方根。 | `sqrt(2.0)` | `1.4142135623731` |
| `trunc(double precision or decimal)` | `decimal` | 将参数向零方向舍入到最接近或等于零的整数。 | `trunc(42.8)`  | `42` |
| `trunc(v numeric, s integer)` | `decimal` | 将参数舍入到小数点后 *`<s>`* 位。 | `trunc(42.4382, 2)` | `42.43` |
| `width_bucket(operand double precision, b1 double precision, b2 double precision, count integer)` | `integer` | 返回在 *`<count>`* 个等宽区间的直方图中，*`<operand>`* 将被分配到哪个桶中，范围为 *`<b1>`* 到 *`<b2>`*；如果输入超出范围，返回 `0` 或 *`<count>`*+1。 | `width_bucket(5.35, 0.024, 10.06, 5)` | `3` |

<br/>


---
## 随机函数

Extreme DPS 支持使用 `random()` 来生成从 `0.0` 到 `1.0` 的随机 `double precision` 数。

>note
>函数 `random()` 使用了一个简单的线性同余算法。虽然 `random()` 计算速度很快，但它不适用于密码应用。


---


## 三角函数

下表列出了 Extreme DPS 提供的三角函数。

>info
>以下三角函数的输入类型只能是 `double precision`，每个函数的返回类型都是 `double precision`。

<br/>

| 函数 | 描述 | 示例 | 结果 |
| :- | :- | :- | :- |
| `acos(radians)` | 返回参数的反余弦值，以弧度表示。 | `acos(0)` | `1.5707963267948966` |
| `acosd(degrees)` | 返回参数的反余弦值，以度数表示。 | `acosd(0)` | `90` |
| `asin(radians)` | 返回参数的反正弦值，以弧度表示。 | `asin(0)` | `0` |
| `asind(degrees)` | 返回参数的反正弦值，以度数表示。 | `asind(0)` | `0` |
| `atan(radians)` | 返回参数的反正切值，以弧度表示。 | `atan(0)` | `0` |
| `atand(degrees)` | 返回参数的反正切值，以度数表示。 | `atand(0)` | `0` |
| `atan2(y radians, x radians)` | 返回 *`<y>`*/*`<x>`* 的反正切值，以弧度表示。 | `atan2(0,0)` | `0` |
| `atan2d(y degrees, x degrees)` | 返回 *`<y>`*/*`<x>`* 的反正切值，以度数表示。 | `atan2d(0,0)` | `0` |
| `cos(radians)` | 返回参数的余弦值，以弧度表示。 | `cos(0)` | `1` |
| `cosd(degrees)` | 返回参数的余弦值，以度数表示。 | `cosd(0)` | `1` |
| `cot(radians)` | 返回参数的余切值，以弧度表示。 | `cot(PI/4)` | `1` |
| `cotd(degrees)` | 返回参数的余切值，以度数表示。 | `cotd(-45)` | `-1` |
| `sin(radians)` | 返回参数的正弦值，以弧度表示。 | `sin(0)` | `0` |
| `sind(degrees)` | 返回参数的正弦值，以度数表示。 | `sind(0)` | `0` |
| `tan(radians)` | 返回参数的正切值，以弧度表示。 | `tan(0)` | `0` |
| `tand(degrees)` | 返回参数的正切值，以度数表示。 | `tand(0)` | `0` |

<br/>

---
## 双曲函数

下表列出了 Extreme DPS 提供的双曲函数。

>info
>以下双曲函数的输入类型只能是 `double precision`，每个函数的返回类型都是 `double precision`。 

<br/>

| 函数 | 描述 | 示例 | 结果 |
| :- | :- | :- | :- |
| `sinh(double precision)` | 返回参数的双曲正弦值。 | `sinh(0)` | `1` |
| `cosh(double precision)` | 返回参数的双曲余弦值。 | `cosh(0)` | `1` |
| `tanh(double precision)` | 返回参数的双曲正切值。 | `tanh(0)` | `0` |
| `asinh(double precision)` | 返回参数的双曲正弦值。 | `asinh(0)` | `0` |
| `acosh(double precision)` | 返回参数的双曲余弦值。 | `acosh(1)` | `0` |
| `atanh(double precision)` | 返回参数的双曲正切值。 | `atanh(0)` | `0` |
