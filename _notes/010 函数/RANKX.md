---
created: 2022-05-29
tags:
 - 排序
 - 标量
subject: dax
importance: 5
skilled: 4
status: 
author: 
url: 
cover: 
returns: 标量
---

返回在当前上下文中计值的表达式在沿着指定表每行计值的表达式结果中的排名

## 语法

```js
RANKX( <表>, <表达式>, [<值>], [<排序>], [<平局规则>])
```

|PARAMETER|ATTRIBUTES|DESCRIPTION|
|-|-|-|
|Table<br>[ITERATOR](https://www.sqlbi.com/articles/row-context-and-filter-context-in-dax/)||表或表达式.|
|Expression<br>[ROW CONTEXT](https://www.sqlbi.com/articles/row-context-and-filter-context-in-dax/)||沿着表每行计值的表达式|
|Value|Optional|需要返回排名的 DAX 表达式，返回标量值。当<value>省略时，用第二参数<表达式>代替|
|Order|Optional|排名依据。0 或 False 代表降序；1 或 True 代表升序，默认使用降序|
|Ties|Optional|处理相同排名时的依据，skip 代表稀疏排名，下一名的排序等于之前所有排序的数量+1；dense 代表稠密排名，只累加排序，不考虑数量。默认使用 skip|


## 返回值

#标量 一个整数值

## 备注

-   RANKX 默认使用降序，在遇到相同记录时，默认使用稀疏排名，下一名的排序等于之前所有排序的数量+1
-   如果表达式或值的计算结果为空，那么对于数字结果式，它会被视为 0；对于文本型结果，则会被视为空文本
-   在第三参数省略的情况下，第二参数在调用 RANKX 的环境中再次计值，得到的计算结果作为第三参数。
-   可以通过在参数列表中放置空逗号 (,) 来跳过可选参数，例如 RANKX(Inventory, [InventoryCost],,,”Dense”)

## 示例

```js
--  RANKX computes the ranking of an expression over a table
--  The expression is evaluated during the iteration over the
--  table and then in the evaluation context of RANKX.
--  The result is the position of the outer evaluation in the
--  lookup table built during the iteration
DEFINE
    VAR BrandsAndSales =
        ADDCOLUMNS (
            VALUES ( 'Product'[Brand] ),
            "@Amt", [Sales Amount]
        )
EVALUATE
ADDCOLUMNS (
    BrandsAndSales,
    "Rank",
        RANKX (
            BrandsAndSales,
            [@Amt]
        )
)
ORDER BY [@Amt] DESC
```

![](https://secure2.wostatic.cn/static/aZ7Y8iELoXWsig4ivd5cyc/image.png?auth_key=1653838630-q2z67f3iF1p1d1AJ3Zpef2-0-807ebb028a2ebf6b9a9337ab35e7b73a)

```js
--  The third argument of RANKX is useful when the outer
--  evaluation requires a different expression than the
--  inner one. For example, when ranking an expression over
--  a pre-built lookup table
DEFINE
    VAR SalesLevels =
        SELECTCOLUMNS ( { 6000000, 3000000, 1500000, 750000, 0 }, "@Limit", [Value] )
 
EVALUATE
ADDCOLUMNS (
    VALUES ( Product[Brand] ),
    "Sales Amount", [Sales Amount],
    "Level", RANKX ( SalesLevels, [@Limit], [Sales Amount] )
)
ORDER BY [Level] ASC
```

![](https://secure2.wostatic.cn/static/9iUD93bEE5a8v5XUVARNmP/image.png?auth_key=1653838653-cSdNUuHNYqWLfA2s1yKzjn-0-1ede17f58143381502b0a577e04358b4)

```js
--  The fourth argument of RANKX specifies the order of ranking
--  it can be DESC (default) or ASC
DEFINE
    VAR BrandsAndSales =
        ADDCOLUMNS ( VALUES ( 'Product'[Brand] ), "@Amt", [Sales Amount] )
EVALUATE
ADDCOLUMNS (
    BrandsAndSales,
    "Rank ASC",  RANKX ( BrandsAndSales, [@Amt], [@Amt], ASC ),
    "Rank DESC", RANKX ( BrandsAndSales, [@Amt], [@Amt], DESC ),
    "Rank (default)", RANKX ( BrandsAndSales, [@Amt], [@Amt] )
)
ORDER BY [@Amt] DESC
```

![](https://secure2.wostatic.cn/static/vhqZprDLCqwEss1JS4qZaz/image.png?auth_key=1653838663-rmgGTQCkUSCV2QkGrjbwmU-0-3c3422eb768fa0f94f3117bf50add2cd)

```js
--  The fifth argument of RANKX specifies the behavior in
--  case of ties. SKIP (default) skips positions, whereas
--  DENSE guarantees a 1-step increment in the ranking
DEFINE
    VAR BrandsAndSales =
        ADDCOLUMNS (
            VALUES ( 'Product'[Brand] ),
            "@Amt", MROUND ( [Sales Amount], 1E6 )
        )
EVALUATE
ADDCOLUMNS (
    BrandsAndSales,
    "Rank SKIP",  RANKX ( BrandsAndSales, [@Amt], [@Amt], DESC, SKIP ),
    "Rank DENSE", RANKX ( BrandsAndSales, [@Amt], [@Amt], DESC, DENSE )
)
ORDER BY [@Amt] DESC
```

![](https://secure2.wostatic.cn/static/6JHGkGBisK1B9MGFY2ASYz/image.png?auth_key=1653838673-t5JySULr4kdKKBtH5GHTGp-0-7b1e72fdbbd76e6e7ccee568939889a9)

## 进阶

[[理解RANKX]]

[[PowerBI处理同名客户]]
[[筛选器按值排序]]
[[如何快速计算索引数]]
[[显示第N个元素]]
[[层级结构中重复值排序]]


## 相似函数
[[RANK.EQ]]