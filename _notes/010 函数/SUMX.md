---
created: 
tags: 聚合 迭代
score: 5
subject: 迭代
url: https://dax.guide/filter/
---
返回为表中的每一行计算的表达式的和。
参考来源：https://dax.guide/filter/
## 语法

```js
SUMX ( <表名>, <表达式> )
```
| PARAMETER                                                    | ATTRIBUTES | DESCRIPTION                                                  |
| :----------------------------------------------------------- | :--------- | :----------------------------------------------------------- |
| Table |      | The table containing the rows for which the expression will be evaluated. |
| Expression  |      | The expression to be evaluated for each row of the table.    |


## 返回值

#标量 一个任何类型的值

## 备注

-   SUMX 函数将表或返回表的表达式作为其第一参数。 第二参数是要计算总和的数字的列，或更常用的返回标量结果的表达式。
    
-   SUMX 仅对列中的数字进行计数。 空白、逻辑值和文本会被忽略。
    

迭代整张表，并对表的每一行执行计算，最后聚合结果以生成所需的单个值

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

```js
Sum of Margin for High School = 
SUMX(
  FILTER(
    FactInternetSales,
    RELATED(DimCustomer[EnglishEducation])="High School"
  ),
  FactInternetSales[SalesAmount]-FactInternetSales[TotalProductCost]
)
```

```js
Sum of Sales by Customer = 
SUMX(
  CALCULATETABLE(
    FactInternetSales,
    DimCustomer[EnglishEducation]="High School"
  ),
  FactInternetSales[SalesAmount]-FactInternetSales[TotalProductCost]
)
```

```js
SUMX with OR = 
SUMX(
  FILTER (
    'Global-Superstore',
    OR (
      'Global-Superstore'[Category] = "Furniture",
      'Global-Superstore'[Sub-Category]="Chairs"
    )
    ), 
    'Global-Superstore'[Sales]
)
```

![[Pasted image 20211103180014.png]]

## 进阶

[[迭代器的嵌套优化]]

[[理解迭代器]]

## 相关函数
```dataview
table score
from "函数"
where contains(tags,"迭代")
sort score
```