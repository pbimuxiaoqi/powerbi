---
created: 2022-05-13
tags:
 - 时间智能
 - 表
subject: dax
importance: 5
skilled: 5
status: 
url: 
cover: 
returns: 表
---

返回一个单列的日期表，将当前筛选上下文中的日期按指定的间隔向未来或者过去平移


## 语法

```js
DATEADD ( <Dates>, <NumberOfIntervals>, <Interval> )
```

|PARAMETER|ATTRIBUTES|DESCRIPTION|
|--|--|--|
|Dates||包含日期的列或包含日期的单列表|
|NumberOfIntervals||一个整数，从日期列中添加或减去的时间间隔数|
|Interval||任意一个: Day, Month, Quarter, Year.|

## 返回值

#表

## 备注

- 未针对 DirectQuery 进行优化，在计算列和行级别安全性公式中完全不受支持。 但可以在度量值和查询公式中使用，只不过无法保证性能。
- **结果只包含日期表中存在的日期**


## 示例

```js
--  DATEADD is a more generic functions.
--  It shifts a period back and forth over time using
--  DAY, MONTH, QUARTER, YEAR
--  This example produces the same result as SAMEPERIODLASTYEAR
EVALUATE
VAR StartDate = DATE ( 2008, 07, 25 )
VAR EndDate =   DATE ( 2008, 07, 31 )
RETURN
    CALCULATETABLE (
        DATEADD ( 'Date'[Date], -1, YEAR ),
        'Date'[Date] >= StartDate &&
        'Date'[Date] <= EndDate
    )
ORDER BY [Date]
```

![[Pasted image 20211108222519.png]]

```js
--  DATEADD has a quite complex logic to move months and quarters
--  the right way, handling months with different dates.
EVALUATE
VAR StartDate = DATE ( 2008, 02, 25 )
VAR EndDate =   DATE ( 2008, 02, 29 )
RETURN
    CALCULATETABLE (
        DATEADD ( 'Date'[Date], +1, MONTH ),
        'Date'[Date] >= StartDate &&
        'Date'[Date] <= EndDate
    )
ORDER BY [Date]
```

![[Pasted image 20211108222528.png]]

MTD 、YTD、 QTD

```js
--  This example shows the sales in the current and previous month.
--  It also reports sales in the same month in the previous quarter and year.
DEFINE
    MEASURE Sales[Same period last month] =
        CALCULATE (
            [Sales Amount],
            DATEADD ( 'Date'[Date], -1, MONTH )
        )
    MEASURE Sales[Same period last quarter] =
        CALCULATE (
            [Sales Amount],
            DATEADD ( 'Date'[Date], -1, QUARTER )
        )
    MEASURE Sales[Same period last year] =
        CALCULATE (
            [Sales Amount],
            SAMEPERIODLASTYEAR ( 'Date'[Date] )
        )
EVALUATE
SUMMARIZECOLUMNS (
    'Date'[Calendar Year Month Number],
    'Date'[Calendar Year Month],
    "Sales Amount", [Sales Amount],
    "Same period last month", [Same period last month],
    "Same period last quarter", [Same period last quarter],
    "Same period last year", [Same period last year]
)
ORDER BY [Calendar Year Month Number]
```

## 相关函数
