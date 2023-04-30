---
created: 2022-07-03
tags: 
- 聚合 
returns: 标量
subject: dax
importance: 5
skilled: 5
status:
author:
url: https://dax.guide/sum/
cover: 
---

```dataview
list url
where file.name = this.file.name
```

对列中的所有数值求和。

## 语法

```js
SUM ( <ColumnName> )
```

|PARAMETER|ATTRIBUTES|DESCRIPTION|
| :----------------------------------------------------------- | :--------- | :----------------------------------------------------------- |
|ColumnName||要求和的数值列|

## 返回值

#标量  一个任意类型的值

## 备注

-   当聚合单列时，内部执行SUMX,

```js
SUM ( table[column] )
```

等价于

```js
SUMX (
    table,
    table[column]
)
```

-   不能对非数值类型的结果求和。

## 示例

```js
--  SUM is the short version of SUMX, when used with one column only
--  SUMX is required to evaluate formulas, instead of columns
DEFINE
    MEASURE Sales[# Quantity 1] = SUM ( Sales[Quantity] )
    MEASURE Sales[# Quantity 2] = SUMX ( Sales, Sales[Quantity] )
    MEASURE Sales[Sales Amount] =
        SUMX ( Sales, Sales[Quantity] * Sales[Net Price] )
EVALUATE
SUMMARIZECOLUMNS (
    'Product'[Color],
    "Quantity 1", [# Quantity 1],
    "Quantity 2", [# Quantity 2],
    "Sales Amount", [Sales Amount]
)
```

```js
--  SUMX is needed to iterate the content of a variable,
--  indeed SUM works only with columns in the model
DEFINE
    MEASURE Sales[Sales Amount] =
        SUMX ( Sales, Sales[Quantity] * Sales[Net Price] )
    MEASURE Sales[SUM Monthly Sales] =
        VAR MonthlySales =
            ADDCOLUMNS (
                DISTINCT ( 'Date'[Calendar Year Month] ),
                "@MonthlySales", [Sales Amount]
            )
        VAR FilteredSales =
            FILTER ( MonthlySales, [@MonthlySales] > 10000 )
        VAR Result =
            -- Iterator required to aggregate the @MonthlySales column       
            SUMX ( FilteredSales, [@MonthlySales] )
        RETURN
            Result
EVALUATE
SUMMARIZECOLUMNS (
    'Product'[Color],
    "SUM Monthly Sales", [SUM Monthly Sales]
)
```

## 相似函数
```dataview
list from #聚合
```