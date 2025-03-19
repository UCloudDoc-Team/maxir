# JSON 函数

Extreme DPS 提供一系列 JSON 函数帮助用户处理 JSON 数据。

---
## JSON_ARRAY_CONTAINS

判断字符串中的 JSON 数组是否包含了指定值。


 

### 语法

```sql
JSON_ARRAY_CONTAINS(<json_array>, <value>)
```


 


### 参数

- *`<json_array>`*
    
    包含了 JSON 数组的 `varchar` 字符串。

- *`<value>`*

    指定值，数据类型可以是 `varchar`、`double precision`、`bigint` 或 `boolean`。


 

### 返回

`boolean` 值。


 


### 示例

```sql
SELECT JSON_ARRAY_CONTAINS('[1, -1.1, true]', 1); 
-> true
SELECT JSON_ARRAY_CONTAINS('[-2, 2.2, false]', 1); 
-> false
SELECT JSON_ARRAY_CONTAINS('[1, -1.1, true]', -1.1); 
-> true
SELECT JSON_ARRAY_CONTAINS('[1, -1.1, true]', true); 
-> true
SELECT JSON_ARRAY_CONTAINS('["a", null]', 'a'); 
-> true
```

---

## JSON_ARRAY_LENGTH

返回字符串中包含的 JSON 数组的长度。


 


### 语法

```sql
JSON_ARRAY_LENGTH(<json_array>)
```


 


### 参数

*`<json_array>`*：包含了 JSON 数组的 `varchar` 字符串。


 


### 返回

`bigint` 值。


 

### 示例

```sql
SELECT JSON_ARRAY_LENGTH('[1, -1.1, true]'); 
-> 3
```


 


---

## JSON_EXTRACT_SCALAR

从一个 JSON 字符串中提取出指定路径的标量值，并将该值作为普通字符串返回。


 


### 语法

```sql
JSON_EXTRACT_SCALAR(<json>, <json_path>)
```

 

### 参数

- *`<json>`*

    `varchar` 类型的 JSON 字符串。

- *`<json_path>`*

    包含了要提取元素的 `varchar` 类型 JSON 路径字符串。



 


### 返回

`varchar` 字符串。


 

### 示例
 
```sql
SELECT JSON_EXTRACT_SCALAR('{"a": 1, "b": 2}', '$.a');
-> 1
SELECT JSON_EXTRACT_SCALAR('["aa", "bb", "cc"]', '$[1]'); 
-> bb
SELECT JSON_EXTRACT_SCALAR('{"a": {"b": 1}, "c": "xxx"}', '$.a'); 
-> NULL
```


 


---

## JSON_SIZE

返回一个 JSON 字符串中由 `json_path` 所指定的路径对应的值的大小。


 


### 语法

```sql
JSON_SIZE(<json>, <json_path>)
```

 

### 参数

- *`<json>`*

    `varchar` 类型的 JSON 字符串。

- *`<json_path>`*

    `varchar` 类型 JSON 路径字符串。



 


### 返回

`bigint` 值。


 


### 示例

```sql
SELECT JSON_SIZE('{"dict": {"a": 1, "b": 2}, "array": [1, [], 3]}', '$.dict'); 
-> 2
SELECT JSON_SIZE('{"dict": {"a": 1, "b": 2}, "array": [1, [], 3]}', '$.array'); 
-> 3
```

