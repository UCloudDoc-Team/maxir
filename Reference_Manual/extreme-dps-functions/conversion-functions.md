
# 转换函数

在处理查询时，Extreme DPS 会隐式地将数值和字符值转换为正确的类型。但是，Extreme DPS 不支持数值和字符值之间的转换。如需实现数值类型到字符类型或字符类型到数字类型的转换，需要通过转换函数来实现。

---


## TRY_CAST

将输入值转换为指定的数据类型。如转换失败，会返回 NULL。

### 语法

```sql
TRY_CAST(<source_expression> AS <destination_type>)
```

### 参数

- *`<source_expression>`*

    指定了需要转换的内容的 `varchar` 或数值类型的表达式。

- *`<destination_type>`*

    转换的目标数据类型。

    如 *`<source_expression>`* 是 `varchar`，那 *`<destination_type>`* 可以为任意数值类型；如 *`<source_expression>`* 是数值类型，则 *`<destination_type>`* 只能是 `varchar`。 


### 返回

目标数据类型的值。


### 示例

```sql
SELECT TRY_CAST('123.45' AS DECIMAL(5,2)) AS converted_value;

--返回如下：

converted_value
---------------
123.45
```

