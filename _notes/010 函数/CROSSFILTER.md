---
created: 2022-05-15
tags: 
- 关系 
- 筛选
subject: dax
importance: 3
skilled: 2
status:
url: https://dax.guide/crossfilter/
cover: 
---
指定 DAX 表达式在求值过程中使用的交叉筛选的方向。

## 语法

```js
CROSSFILTER ( <关系的左列>, <关系的右列>, <筛选类型> )
```

| 参数<div style="width:100px">       | 属性<div style="width:100px">      | 描述                                                         |
| ---------- | :--: | ------------------------------------------------------------ |
| 关系的左列 |      | 用于建立关系的两列之一                                       |
| 关系的右列 |      | 用于建立关系的两列之一                                       |
| 筛选类型   |      | 关闭筛选使用 0，<br />单向筛选使用 1，<br />双向筛选使用 2；<br />OneWay_RightFiltersLeft 使用 3，<br />OneWay_LeftFiltersRight 使用 4；<br />也可以直接用 None, OneWay, Both, OneWay_RightFiltersLeft, OneWay_LeftFiltersRight |


## 返回值

函数不返回任何值。仅在查询期间为所指示的关系设置交叉筛选方向。

## 备注

-   **在一对一关系的情况下，设置为单向和双向并无任何区别**。
-   CROSSFILTER 只能用于支持将筛选器用作参数的函数中，例如：CALCULATE、CALCULATETABLE、CLOSINGBALANCEMONTH、CLOSINGBALANCEQUARTER、CLOSINGBALANCEYEAR、OPENINGBALANCEMONTH、OPENINGBALANCEQUARTER、OPENINGBALANCEYEAR、TOTALMTD、TOTALQTD 和 TOTALYTD 函数。
-   CROSSFILTER 使用模型中的现有关系，通过位于关系两端的列标识关系。
-   在 CROSSFILTER 中，模型设置的关系的交叉筛选方向并不重要；也就是说，在模型中将关系设置为单向或双向筛选不会影响函数的使用。 CROSSFILTER 将覆盖任何现有的交叉筛选设置。
-   如果作为为参数的列不属于一个关系，或者参数属于不同的关系，则会返回错误。
-   如果 CALCULATE 表达式是嵌套的，并且多个 CALCULATE 表达式包含 CROSSFILTER 函数，那么最内层的 CROSSFILTER 就是在有冲突或歧义的情况下起作用的函数。
-   参数 OneWay_RightFiltersLeft 和 OneWay_LeftFiltersRight 可以用于多对多和一对多关系，但不能用于一对一关系。当交叉筛选类型 OneWay_RightFiltersLeft 或 OneWay_LeftFiltersRight 用于一对多关系类型时，它必须与一对多的关系传递方向一致。如果请求的方向与此相反，则返回错误。

## 注意

>**<font color=red>CROSSFILTER 不能在 DirectQuery 模式下创建计算列，所引用的表不支持行级别安全性，也就是说你不能对开启行级别安全性的表使用这个函数，[[USERELATIONSHIP]] 也存在同样的限制。可以考虑用 [[LOOKUPVALUE]] 或者 [[TREATAS]] 作为替代方案。</font>**

## 案例

```js
--  CROSSFILTER changes the cross-filter direction of a relationship
--  The arguments are the columns involved in the relationship and
--  the cross-filter direction, that can be BOTH, SINGLE, NONE
DEFINE
    MEASURE Sales[Customers] =
        COUNTROWS ( Customer )
    MEASURE Sales[CustomersFiltered] =
        CALCULATE (
            [Customers],
            CROSSFILTER ( Sales[CustomerKey], Customer[CustomerKey], BOTH )
        )
    MEASURE Sales[ProductsDoesNotFilter] =
        CALCULATE (
            [Customers],
            CROSSFILTER ( Sales[CustomerKey], Customer[CustomerKey], BOTH ),
            CROSSFILTER ( Sales[ProductKey], 'Product'[ProductKey], NONE )
        )
EVALUATE
SUMMARIZECOLUMNS (
    'Product'[Brand],
    "Customers", [Customers],
    "Customers Buying", [CustomersFiltered],
    "Products does not filter Sales", [ProductsDoesNotFilter]
)
```

![](https://secure2.wostatic.cn/static/4dJHPPBYEsQrXHYxHuGUv5/image.png?auth_key=1652599268-qxgFC15wijRn6THKXkxvu-0-76a48a81991cd026ffd8f25343a7ce52)

## 进阶

## 相关函数
```dataview
table importance, skilled
from #关系
where subject = "dax"
```