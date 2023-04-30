---
created: 2022-05-21
tags: 
- 功能函数 
- 标量 
subject: dax
importance: 5
skilled: 4
status:
author:
url: https://dax.guide/isblank/
cover: 
---

检查值是否为空，如果为空返回True，否返回False

## 语法

```js
ISBLANK ( <Value> )
```

| 参数               | 属性               | 说明                         |                 |
| ---------------- | ---------------- | -------------------------- | --------------- |
| text             | text             | text                       |                 |
| column-id-clr28p | column-id-vv3y3k | column-id-j39mam           | table-id-7cfo32 |
| Value            |                  | The value you want to test | row-id-ffi73r   |

## 返回值

#标量 布尔值

## 备注

-   也可以使用`==`来判断

```js
ISBLANK ( <expression> ) -- same result as the following line
<expression> == BLANK()  -- same result as the previous line
```

-   空字符串和0返回False

## 示例

```js
--  ISBLANK check that an expression is strictly equal to BLANK
--  ISBLANK (x) is an alias for x == BLANK ()
--  An empty string and 0 are not considered blank by ISBLANK
DEFINE
    VAR BlankValue = BLANK()
    VAR NumericValue = 42
    VAR ZeroValue = 0
    VAR EmptyString = ""
EVALUATE {
 ( 1, "ISBLANK ( BlankValue )", ISBLANK ( BlankValue ) ),
 ( 2, "ISBLANK ( NumericValue )", ISBLANK ( NumericValue ) ),
 ( 3, "ISBLANK ( ZeroValue )", ISBLANK ( ZeroValue ) ),
 ( 4, "ISBLANK ( EmptyString )", ISBLANK ( EmptyString ) ),
 ( 5, "EmptyString == BLANK()", EmptyString == BLANK() ),
 ( 6, "EmptyString = BLANK()", EmptyString = BLANK() )
}
ORDER BY [Value1]
```

![](https://secure2.wostatic.cn/static/tc9K5u8K4Ws4611eGrzwe8/image.png)

不能应用于表

```js
--  ISBLANK cannot be used with tables, it requires a scalar value
--  using it with tables forces the implicit conversion of a table
--  to a scalar value and might result in an error
DEFINE
    VAR EmptySet = FILTER ( { 1 }, FALSE )
    VAR SetWithBlank = { BLANK() }
EVALUATE {
    ( 1, "ISBLANK ( EmptySet )", ISBLANK ( EmptySet ) ),
    ( 2, "ISBLANK ( SetWithBlank )", ISBLANK ( SetWithBlank ) )
}
ORDER BY [Value1]
```

![](https://secure2.wostatic.cn/static/6F4MpNoUkpVwquo1dKPDJm/image.png)

```js
--  ISBLANK check that an expression is strictly equal to BLANK
--  ISBLANK (x) is an alias for x == BLANK ()
DEFINE
    MEASURE Customer[EmptyNames] =     -- Returns blanks and empty string
        CALCULATE (
            COUNTROWS ( Customer ),
            Customer[Customer Name] = BLANK ()
        )
    MEASURE Customer[BlankNames] =     -- Returns only blanks, same as == BLANK()
        CALCULATE (
            COUNTROWS ( Customer ),
            ISBLANK ( Customer[Customer Name] )
        )
EVALUATE
SUMMARIZECOLUMNS (
    Customer[Continent],
    "Customers", COUNTROWS ( Customer ),
    "Customers with empty name", [EmptyNames],
    "Customers with blank name", [BlankNames]
)
```

![](https://secure2.wostatic.cn/static/sg8si7D4QnTvLEtiqEFwV5/image.png)

## 相关函数
[[BLANK]]