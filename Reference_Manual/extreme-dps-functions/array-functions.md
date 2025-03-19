# 数组函数和运算符

Extreme DPS 提供了一系列的数组函数和运算符，用于帮助用户处理数组类型。

---

## 数组运算符

现目前，Extreme DPS 支持下标运算符 `[]` 和连接运算符 `||`。

 

### []

访问数组中指定下标对应的元素。数组中元素下标由 1 开始。

在以下场景中，返回 null：

- 指定下标越界

- 指定下标为负

- 指定下标为 0 

如下为一些示例：

```sql
SELECT array_col[1] FROM (VALUES(ARRAY[11, 22, 33])) as t1(array_col); 
-> 11

SELECT array_col[8] FROM (VALUES(ARRAY[11, 22, 33])) as t1(array_col); 
-> null

SELECT array_col[0] FROM (VALUES(ARRAY[11, 22, 33])) as t1(array_col); 
-> null

SELECT array_col[-1] FROM (VALUES(ARRAY[11, 22, 33])) as t1(array_col); 
-> null
```

 

### ||

连接两个拥有相同类型元素的数组。

如下为一些示例：

```sql
SELECT ARRAY[1,2,3] || ARRAY[4,5,6]	; 
-> [1, 2, 3, 4, 5, 6]

SELECT 3 || ARRAY[4,5,6]; 
-> [3, 4, 5, 6]

SELECT ARRAY[4,5,6] || 7; 
-> [4, 5, 6, 7]
```


---

## 数组函数

 

### ARRAY_SORT

对输入数组进行升序排序后返回。数组中的元素必须是可排序的。NULL 元素会被放在数组的末尾。

#### 语法

```sql
ARRAY_SORT(<array>)
```

#### 参数

*`<array>`*：需要对元素进行排序的数组。


#### 返回

和输入类型相同。

#### 示例

```sql
SELECT ARRAY_SORT(ARRAY[2, 1, 3]); 
-> {1, 2, 3}
```
 

### ARRAY_SORT_DESC

对数组中的元素进行降序排序后返回。数组中的元素必须是可排序的。NULL 元素将被放置在返回数组的末尾。


#### 语法

```sql
ARRAY_SORT_DESC(<array>)
```

#### 参数

*`<array>`*：需要对元素进行排序的数组。


#### 返回

和输入类型相同。

#### 示例

```sql
SELECT ARRAY_SORT_DESC(ARRAY[2, 1, 3]); 
-> {3,2,1}
```
 

### ARRAY_ELEMENT_SUM

返回数组里所有非 NULL 元素的累加值。

#### 语法

```sql
ARRAY_ELEMENT_SUM(<array>)
```

#### 参数

*`<array>`*：元素需要进行累加的数组，数组中的元素类型可以为 `smallint`、`integer`、`bigint`、`real`、`double precision`。

#### 返回

- 当输入类型为 `smallint`、`integer`、`bigint` 时，返回值类型为 `bigint`。

- 当输入类型为 `real`、`double precision` 时，返回值类型为 `double precision`。


#### 示例

```sql
SELECT ARRAY_ELEMENT_SUM(ARRAY[1, null, 2, 3]); 
-> 6
SELECT ARRAY_ELEMENT_SUM(ARRAY[]::integer array); 
-> 0
```

 

### CARDINALITY

返回数组内元素的数量。如数组为空，则返回 `0`。

#### 语法

```sql
CARDINALITY(<array>)
```

#### 参数

*`<array>`*：需要计算元素数量的数组。

#### 返回

`integer` 值。

#### 示例

```sql
SELECT CARDINALITY(ARRAY[ARRAY[1,2],ARRAY[3,4]]);  
-> 4
SELECT CARDINALITY(ARRAY[11, 22, 33]); 
-> 3
```

 

### ARRAYS_OVERLAP

检查两个数组中是否存在非 NULL 元素交集。

#### 语法

```sql
ARRAYS_OVERLAP(<array1>, <array2>)
```

#### 参数

- *`<array1>`*

    用于比较的一个数组。

- *`<array2>`*

    用于比较的另一个数组。

#### 返回

返回一个 `boolean` 值：

- 如果两个数组存在非空元素交集，则返回 `true`。

- 如果两个数组不存在非空元素交集，则返回 `false`。

- 如果两个数组不不存在非空元素交集，且任一数组中包含 NULL 元素，则返回 `null`。

#### 示例

```sql
SELECT ARRAYS_OVERLAP(ARRAY[11, 22, 33], ARRAY[22]); 
-> true
SELECT ARRAYS_OVERLAP(ARRAY[11, 22, 33], ARRAY[22, 44]); 
-> true
SELECT ARRAYS_OVERLAP(ARRAY[11, 22, 33], ARRAY[44, 55]); 
-> false
SELECT ARRAYS_OVERLAP(ARRAY[11, 22, null, 33], ARRAY[33, 44]); 
-> true
SELECT ARRAYS_OVERLAP(ARRAY[11, 22, null, 33], ARRAY[44, 55]); 
-> null
```


 


### CONTAINS

检查数组中是否包含指定元素。

#### 语法

```sql
CONTAINS(<array>, <element>)
```

#### 参数

- *`<array>`*
    
    要检查的数组。

- *`<element>`*

    指定的元素。

#### 返回

返回一个 `boolean` 值：

- 如果数组中包含指定元素，且指定元素不为 null，则返回 `true`。

- 如果数组中不包含指定元素，切指定元素不为 null，则返回 `false`。

- 如果指定元素为 null，则返回 `null`。


#### 示例

```sql
SELECT CONTAINS(ARRAY[11, 22, 33], 11); 
-> true
SELECT CONTAINS(ARRAY[11, 22, 33, null], 100); 
-> false
SELECT CONTAINS(ARRAY[11, 22, 33], null); 
-> null
SELECT CONTAINS(ARRAY[11, 22, 33, null], null); 
-> null
```