---
created: 2022-05-15
tags: 关系
subject: dax
importance: 5
skilled: 5
status:
url: https://dax.guide/userelationship/
cover: 
---

指定 DAX 表达式的在计算时所使用的关系。将关系两端的列作为参数可以定义这种关系。

## 语法

```js
USERELATIONSHIP ( <列 1>, <列 2> )
```

| 参数 | 属性 |       描述       |
|:----:|:----:|:----------------:|
| 列1  |      | 关系的主键或外键 |
| 列2  |      | 关系的主键或外键 |


## 返回值

函数不返回任何值；仅用来明确计算过程中使用的关系。

## 备注

-   **USERELATIONSHIP 只能用于将筛选器用作参数的函数中**，例如：CALCULATE、CALCULATETABLE、CLOSINGBALANCEMONTH、CLOSINGBALANCEQUARTER、CLOSINGBALANCEYEAR、OPENINGBALANCEMONTH、OPENINGBALANCEQUARTER、OPENINGBALANCEYEAR、TOTALMTD、TOTALQTD 和 TOTALYTD 函数。
-   使用的是模型中现有的关系
-   **关系的状态不重要**；也就是说，关系是否活动不影响函数的使用。 即使关系不活动，USERELATIONSHIP 也会使用该关系，并覆盖模型中可能存在但函数参数中未提及的其他任何有效的关系。
-   如果任何命名为参数的列不属于一个关系，或者参数属于不同的关系，则会返回错误。
-   如果使用嵌套的 CALCULATE 表达式，并且多个 CALCULATE 表达式都包含 USERELATIONSHIP 函数，那么只有最内层的 USERELATIONSHIP 生效。**注意最多只能嵌套 10 个 USERELATIONSHIP 函数**

>**<font color="red" size=6>USERELATIONSHIP 引用的表不支持行级别安全性，也就是说你不能对开启行级别安全性的表使用这个函数，[[CROSSFILTER]]也是同理。可以考虑用 [[LOOKUPVALUE]] 或者 [[TREATAS]] 作为替代方案。 </font>**

## 示例

```js
--  USERELATIONSHIP activates a disabled relationship, deactivating
--  possible conflicting relationships.
--
--  Useful when the model contains inactive relationships to handle
--  role-playing dimensions.
DEFINE
    MEASURE Sales[Delivery Amount] =
        CALCULATE (
            [Sales Amount],
            USERELATIONSHIP ( Sales[Delivery Date], 'Date'[Date] )
        )
EVALUATE
SUMMARIZECOLUMNS (
    'Date'[Calendar Year],
    "Sales Amount", [Sales Amount],
    "Delivery Amount", [Delivery Amount]
)
```

```js
[Delivered Amount in 2007] = 
CALCULATE (
    [Sales Amount],
    USERELATIONSHIP ( Sales[DeliveryDateKey]; 'Date'[DateKey] ),
    'Date'[Calendar Year] = 2007
)
```

## 相关函数
```dataview
table importance, skilled
from #关系
where subject = "dax"
```