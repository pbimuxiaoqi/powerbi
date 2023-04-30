---
tags:
 - 筛选
 - 表
created: 2022-05-10
importance: 5
skilled: 4
subject: dax
url: 
returns: 表
---




接受一个表和一个逻辑条件作为参数，返回满足条件的所有行。

## 语法

```js
FILTER ( <表>, <布尔表达式> )
```

|参数|属性|描述|
| :----------------------------------------------------------- | :--------- | :----------------------------------------------------------- |
|表 （迭代）||需要被筛选的表或表表达式|
|条件表达式（行上下文）||要为表的每一行计算的布尔表达式|

## 返回值

#表 整个表或具有一列或多列的表，其中只包含已筛选的行。

## 备注

-   FILTER 既是一个表函数，又是一个迭代器。为了返回最终结果，它对表进行逐行扫描，在行上下文环境中计算逻辑条件，返回符合条件的记录。
    
-   由于上下文转换的作用，在 FILTER 表达式中使用一个度量值，可以基于其他行或表进行动态计算来完成过滤。
    
-   先迭代后筛选
    

## 示例

```js
EVALUATE
FILTER (
    Customer,
    Customer[Continent] = "Europe"
)
```

FILTER可以通过RELATED来使用其他表进行筛选，但通常[[CALCULATE#语法]]是最好的选择

```js
--  Being an iterator, FILTER creates a row context. If you need
--  to access related tables, the RELATED function is needed.
--  This makes the usage of CALCULATE preferred over FILTER, when
--  possible
DEFINE
    MEASURE Sales[Red Sales] =
        SUMX (
            FILTER ( Sales, RELATED ( Product[Color] ) = "Red" ),
            Sales[Quantity] * Sales[Net Price]
        )
    MEASURE Sales[Red Sales CALCULATE] =
        CALCULATE ( [Sales Amount], KEEPFILTERS ( Product[Color] = "Red" ) )
    MEASURE Sales[Red Sales Allways] =
        CALCULATE ( [Sales Amount], Product[Color] = "Red" ) 
EVALUATE
SUMMARIZECOLUMNS (
    Product[Color],
    "Sales", [Sales Amount],
    "Red Sales", [Red Sales],
    "Red Sales CALCULATE", [Red Sales CALCULATE],
    "Red Sales Allways", [Red Sales Allways]
)
```

![[Pasted image 20211104142145.png]]

FILTER进行上下文转换

```js
DEFINE
    MEASURE Sales[Sales Amount] =
        SUMX ( Sales, Sales[Quantity] * Sales[Net Price] )
    MEASURE Sales[Sales in countries >3M] =
        CALCULATE (
            [Sales Amount],
            FILTER (
                ALL ( Customer[CountryRegion] ),
                [Sales Amount] > 3000000
            )
       )
EVALUATE
SUMMARIZECOLUMNS (
    'Date'[Calendar Year],
    "Sales amount", [Sales Amount],
    "Sales in countries >3M", [Sales in countries >3M]
)
```

![[Pasted image 20211104142211.png]]

FILTER来迭代变量内容

```js
--  AVERAGE Monthly Sales computes the average sales for the months
--  with at least 10K sales
DEFINE
    MEASURE Sales[Sales Amount] =
        SUMX ( Sales, Sales[Quantity] * Sales[Net Price] )
    MEASURE Sales[AVERAGE Monthly Sales] =
        VAR MonthlySales =
            ADDCOLUMNS (
                DISTINCT ( 'Date'[Calendar Year Month] ),
                "@MonthlySales", [Sales Amount]
            )
        VAR FilteredSales =
            -- Iterator required to filter the @MonthlySales column       
            FILTER ( MonthlySales, [@MonthlySales] > 10000 )
        VAR Result =
            AVERAGEX ( FilteredSales, [@MonthlySales] )
        RETURN
            Result
EVALUATE
SUMMARIZECOLUMNS (
    'Product'[Color],
    "AVERAGE Monthly Sales", [AVERAGE Monthly Sales]
)
```

FILTER先迭代后筛选

```js
EVALUATE
FILTER(
    ADDCOLUMNS(
        VALUES( 'Product'[Color] ), 
        "num", COUNTROWS( 'Product' )
    ),
    'Product'[Color] = "red"
)
```

![[Pasted image 20211104142253.png]]

```js
CALCULATETABLE (
    ADDCOLUMNS(
        VALUES( 'Product'[Color] ), 
        "num", COUNTROWS( 'Product' )
    ),
    'Product'[Color] = "red"
)
```

![[Pasted image 20211104142304.png]]

## 相关函数

[[CALCULATETABLE]]

## 进阶

[[理解FILTER]]

[[上下文转换和扩展表]]

[[筛选表]]

[[CALCULATE中的筛选器参数]]

[[FILTER vs CALCULATEBALE]]