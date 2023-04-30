---
tags: 筛选
score: 5
---
[CALCULATE – DAX Guide](https://dax.guide/calculate/)
在筛选器参数修改过的上下文中对表达式进行求值。


## 语法

```js
CALCULATE( <表达式>, [ <筛选器 1> ], [ <筛选器 2> ] … )
```

|PARAMETER|ATTRIBUTES|DESCRIPTION|
|---|--|--|
|表达式||要计值的表达式|
|筛选器参数||可选<br>可重复|

定义筛选器的布尔表达式或返回表的表表达式

## 返回值

#标量 任意类型的值

该值是在表达式修改后的筛选上下文中计算的结果

## 备注

-   每个筛选器参数都可以删除筛选器（如 ALL、ALLEXCEPT、ALLNOBLANKROW）、还原筛选器（ALLSELECTED），或者使用一个表表达式，返回一列，多列或整个扩展表的值列表。
    
-   当筛选器参数是单列形式的条件判断表达式时，该表达式被嵌入到一个筛选引用列所有值的 [[FILTER#语法]] 表达式中。例如，筛选器参数示意中的第一个表达式的高亮部分在内部被转换为第二个表达式的高亮部分，两种形式完全等价。所以，**除了 CALCULATE 调节器之外，其他形式的筛选器参数的本质都是表**。
    
-   筛选器参数会覆盖同一列上已有的任何筛选器，你可以使用 KEEPFILTERS 改变这种默认行为。
    
    

## 示例

```js
--  The compact syntax (boolean) is expanded in the full syntax
--  prior to the evaluation
DEFINE
    MEASURE Sales[Red Sales] =
        CALCULATE ( [Sales Amount], 'Product'[Color] = "Red" )
    MEASURE Sales[Red Sales Full] =
        CALCULATE (
            [Sales Amount],
            FILTER ( ALL ( 'Product'[Color] ), 'Product'[Color] = "Red" )
        )
EVALUATE
SUMMARIZECOLUMNS (
    'Product'[Color],
    "Sales Amount", [Sales Amount],
    "Red Sales", [Red Sales],
    "Red Sales Full", [Red Sales Full]
)
```

```js
--  You can use any condition as an argument, as long as it can
--  be converted into a table by the DAX engine
DEFINE
    MEASURE Sales[Red Blue Sales] =
        CALCULATE ( [Sales Amount], 'Product'[Color] IN { "Red", "Blue" } )
    MEASURE Sales[Red Blue Sales Full] =
        CALCULATE (
            [Sales Amount],
            FILTER ( ALL ( 'Product'[Color] ), 'Product'[Color] IN { "Red", "Blue" } )
        )
EVALUATE
SUMMARIZECOLUMNS (
    'Product'[Color],
    "Sales Amount", [Sales Amount],
    "Red Blue Sales", [Red Blue Sales],
    "Red Blue Sales Full", [Red Blue Sales Full]
)
```

```js
--  The KEEPFILTERS modifier does not remove an existing filter
DEFINE
    MEASURE Sales[Red Blue Sales Keepfilters] =
        CALCULATE (
            [Sales Amount],
            KEEPFILTERS ( 'Product'[Color] IN { "Red", "Blue" } )
        )
    MEASURE Sales[Red Blue Sales] =
        CALCULATE (
            [Sales Amount],
            'Product'[Color] IN { "Red", "Blue" }
        )
EVALUATE
SUMMARIZECOLUMNS (
    'Product'[Color],
    "Sales Amount", [Sales Amount],
    "Red Blue Sales", [Red Blue Sales],
    "Red Blue Sales Keepfilters", [Red Blue Sales Keepfilters]
)
```

```js
--  When CALCULATE is executed in a row context, it transforms
--  the row contexts in equivalent filter contexts
DEFINE
    MEASURE Sales[Yearly Avg] =
        AVERAGEX (
            VALUES ( 'Date'[Calendar Year] ),
            CALCULATE (
                SUMX ( Sales, Sales[Quantity] * Sales[Net Price] )
            )
        )
EVALUATE
SUMMARIZECOLUMNS (
    'Product'[Color],
    "Sales Amount", [Sales Amount],
    "Yearly Avg", [Yearly Avg]
)
```

```js
--  CALCULATE is implicitly added to any measure reference
DEFINE
    MEASURE Sales[Sales Amount] =
        SUMX ( Sales, Sales[Quantity] * Sales[Net Price] )
    MEASURE Sales[Yearly Avg] =
        AVERAGEX (
            VALUES ( 'Date'[Calendar Year] ),
            CALCULATE (
                SUMX ( Sales, Sales[Quantity] * Sales[Net Price] )
            )
        )
    MEASURE Sales[Yearly Avg 2] =
        AVERAGEX (
            VALUES ( 'Date'[Calendar Year] ),
            [Sales Amount]
        )
EVALUATE
SUMMARIZECOLUMNS (
    'Product'[Color],
    "Sales Amount", [Sales Amount],
    "Yearly Avg", [Yearly Avg],
    "Yearly Avg 2", [Yearly Avg 2]
)
```

```js
--  CALCULATE evaluation steps:
--      1. Evaluation of filter arguments
--      2. Context transition
--      3. Evaluation of CALCULATE modifiers
--      4. Application of filter arguments and KEEPFILTERS
DEFINE
    MEASURE Sales[Test] =
        AVERAGEX (
            VALUES ( 'Date'[Calendar Year] ),
            CALCULATE (
                [Sales Amount],
                'Product'[Category] = "Audio",
                KEEPFILTERS ( 'Product'[Color] IN { "Red", "Blue" } ),
                USERELATIONSHIP ( Sales[Delivery Date], 'Date'[Date] )
            )
        )
EVALUATE
SUMMARIZECOLUMNS (
    'Product'[Color],
    "Sales Amount", [Test]
)
```

## 进阶

[[CALCULATE中的筛选器参数]]

[[CALCULATE中的行上下文转换和筛选器参数]]

[[5章 理解CALCULATE和CALCULATETABLE函数]]

[[CALCULATE指南]]

---

## 调节器
```dataview
list from #调节器 
````