---
created: 2022-05-15
tags: 筛选
subject: dax
importance: 5
skilled: 5
status:
url: https://dax.guide/lookupvalue/
cover: 
---
从表中检索满足所有匹配条件的值。

## 语法

```js
LOOKUPVALUE ( <Result_ColumnName>, <Search_ColumnName>, <Search_Value> [, <Search_ColumnName>, <Search_Value> [, … ] ] [, <Alternate_Result>] )
```

|   参数<div style="width: 100px">   |            属性<div style="width: 100px">              | 描述                                                                                               |
|:--------:|:---------------------------:| -------------------------------------------------------------------------------------------------- |
|  结果列  |  | 要返回的值所在的列名。列必须使用标准 DAX 语法命名，通常是完全限定的，不支持表达式。                |
|  查找列  |           可重复            | 在与结果列相同的表中或扩展表中，执行查找的现有列的名称，查找列使用完全限定名。不支持表达式         |
|  查找值  |           可重复            | 标量表达式（不引用正在搜索的同一表中的任何列）                                                     |
| 备选结果 |            可选             | 当第一参数结果为空或多个不重复值时的替代结果，如果省略此参数，为空时返回 BLANK，匹配多值时返回错误 |


## 返回值

`标量` 一个任意类型的值

## 备注

-   如果没有满足所有搜索值的匹配项，则返回空值或<备选结果>（如果提供） 。 换句话说，如果仅部分条件匹配，则该函数将不会返回查找值
-   如果有多行匹配搜索值，并且在所有情况下结果列的值都相同，那么 LOOKUPVALUE 返回该值 。 但是，如果结果列返回不同的值，则函数返回错误或备选结果（如果提供） ,等效于如下

```js
VAR SearchValue = <Search_Value>
RETURN
    CALCULATE (
        SELECTEDVALUE ( <Result_ColumnName>, <Alternate_Result> ),
        FILTER (
            ALLNOBLANKROW ( <Search_ColumnName> ),
            <Search_ColumnName> == SearchValue     -- The == operator distinguishes between blank and 0/empty string
        ),
        ALL ( <table_of_Result_ColumnName> )       -- If Result_ColumnName is t[c], this is ALL ( t )
    )
```

-   查找列可以使用结果列所在表的扩展表中的任何列
-   LOOKUPVALUE 忽略任何筛选上下文
-   当不可能使用 RELATED 利用数据模型中的现有关系获取数据时，才可以考虑使用 LOOKUPVALUE，因为 RELATED 更快。正因如此，LOOKUPVALUE 通常用于无关系数据的获取，此时 TREATAS 也是不错的选择，而且某些某些情况下性能可能好于 LOOKUPVALUE。

```js
LOOKUPVALUE (
    table[result_column],
    table[search_column_1], <expression_1>,
    table[search_column_2], <expression_2>,
    <alternate_result>
)
```

```
可考虑如下写法
```

```js
CALCULATE (
    SELECTEDVALUE ( table[result_column], <alternate_result> ),
    FILTER (
        ALLNOBLANKROW ( table[search_column_1] ),
        table[search_column_1] == <expression_1>
    ),
    FILTER (
        ALLNOBLANKROW ( table[search_column_2] ),
        table[search_column_2] == <expression_2>
    ),
    REMOVEFILTERS ( )
)
```

```
当 <expression_1> 和 <expression_2> 是常量值时，没有问题。但是，通常情况下，这些表达式更加复杂，这可能会生成更昂贵的查询计划，其中包括对存储引擎的 CallbackDataID 请求。 可以采用如下方法
```

```js
VAR filterValue1 = <expression_1>
VAR filterValue2 = <expression_2>
RETURN CALCULATE (
    DISTINCT ( table[result_column] ),
    table[search_column_1] = filterValue1,
    table[search_column_2] = filterValue2,
    REMOVEFILTERS ( )
)
```

```
也可以写成如下
```

```js
CALCULATE (
    DISTINCT ( table[result_column] ),
    TREATAS ( { <expression_1> }, table[search_column_1] ),
    TREATAS ( { <expression_2> }, table[search_column_2] ),
    REMOVEFILTERS ( )
)
```

```
条件最好写到变量里
```

```js
VAR filter1 = TREATAS ( { <expression_1> }, table[search_column_1] )
VAR filter2 = TREATAS ( { <expression_2> }, table[search_column_2] )
RETURN CALCULATE (
    DISTINCT ( table[result_column] ),
    filter1,
    filter2,
    REMOVEFILTERS ( )
)
```

```
也可以创建一个包含多列的筛选器，有可能会产生更好的查询
```

```js
VAR filterLookup =
    TREATAS (
        { ( <expression_1>, <expression_2> ) },
        table[search_column_1],
        table[search_column_2]
    )
RETURN CALCULATE (
    DISTINCT ( table[result_column] ),
    filterLookup,
    REMOVEFILTERS ( )
)
```

## 示例

```js
LOOKUPVALUE (
    ExchangeRates[Rate],
    ExchangeRates[Date], DATE ( 2018, 4, 15 ),
    ExchangeRates[Currency], "EUR"
)
```

返回客户在2009年生日时是周几

```js
--  LOOKUPVALUE searches in a table for the value of a column in a row
--  that satisfy a set of equality conditions
EVALUATE
VAR SampleCustomers = SAMPLE ( 10, Customer, Customer[Customer Code] )
RETURN
    ADDCOLUMNS (
        SUMMARIZE ( SampleCustomers, Customer[Customer Name], Customer[Birth Date] ),
        "Day of week on birthday in 2009",
        VAR BirthDate = Customer[Birth Date]
        VAR ReferenceYear = 2009
        VAR WeekdayOnBirthday =
            LOOKUPVALUE (
                'Date'[Day of Week],
                'Date'[Calendar Year Number], ReferenceYear,
                'Date'[Month Number], MONTH ( BirthDate ),
                'Date'[Day], DAY ( BirthDate )
            )
        RETURN
            WeekdayOnBirthday
    )
```

![](https://secure2.wostatic.cn/static/4DaNG7ShgajurPaTFQbt3Q/image.png?auth_key=1652628477-bhCBGoM96YHUNA77DoyExZ-0-ab22cd6ba6f12c3fc4ff796750a17382)

## 进阶

[[DAX中查找多个值]]
[[理解lookupvalue]]




