---
created: 2022-07-17
tags:
 - 筛选
 - 调节器
 - 表
 - all家族
subject: dax
importance: 5
skilled: 4
status: 
author: 
url: "https://dax.guide/allexcept"
cover: 
---

```dataview
list url
where file.name = this.file.name
```

删除表中所有筛选器，已应用于指定列的筛选器除外。

## 语法

```js
ALLEXCEPT ( <TableName>, <ColumnName> [, <ColumnName> [, … ] ] ) 
```

| PARAMETER  | ATTRIBUTES | DESCRIPTION                                                  |
| ---------- | ---------- | ------------------------------------------------------------ |
| TableName  |            | 基表，已存在的物理表                                         |
| ColumnName | 可重复     | 当 ALLEXCEPT 作为 CALCULATE 调节器时，需要保留筛选效果的列或表。它们必须是第一个参数所在的扩展表的一部分。 |

## 返回值

表，整张表或具有一列或多列的表

## 备注

1.  当作为 [[CALCULATE]] 或 [[CALCULATETABLE]] 的调节器使用时，ALLEXCEPT 从第一个参数指定的扩展表中删除筛选器，只保留后续参数指定的列中的筛选器。
2.  当作为表函数使用时，ALLEXCEPT 从第一参数中排除后续参数指定的列，返回表中剩余所有列的唯一组合。在这种情况下，**结果只考虑当前表的列，忽略扩展表**。

## 示例

```js
--  Returns all the 'Product' columns
EVALUATE
ALL ( 'Product' )
     
--  Returns all the 'Product' columns but ProductKey and Product Code
EVALUATE
ALLEXCEPT ( 'Product', 'Product'[ProductKey], 'Product'[Product Code] )
```

```js
--  In this example, ALLEXCEPT ignores Sales expanded table filters
--  except the cross-filters coming from Date and the column Product[Color]
DEFINE
    MEASURE Sales[# Sales] = COUNTROWS ( Sales )
EVALUATE
CALCULATETABLE (
    {
         ( 1, "# Sales (CY 2009 - Red)", [# Sales] ),
         ( 2, "# Sales (CY 2009)",       CALCULATE ( [# Sales], ALLEXCEPT ( Sales, 'Date' ) ) ),
         ( 3, "# Sales (Red)",           CALCULATE ( [# Sales], ALLEXCEPT ( Sales, 'Product'[Color] ) ) ),
         ( 4, "# Sales",                 CALCULATE ( [# Sales], REMOVEFILTERS ( ) ) )
    },
    'Product'[Color] = "Red",
    'Date'[Calendar Year] = "CY 2009"
)
ORDER BY [Value1]
```

![[Pasted image 20220717185248.png]]

用作表函数时，会忽略扩展表的筛选，所以下面TABLE FUNCTION显示为sales的总行数

```js
CALCULATETABLE (
    {
        ( "FILTER", COUNTROWS ( Sales ) ),
        ( "CALCULATE FILTER Color ", CALCULATE ( COUNTROWS ( Sales ), ALLEXCEPT ( Sales, 'Product' ) ) ),
        ( "CALCULATE FILTER Year", CALCULATE ( COUNTROWS ( Sales ), ALLEXCEPT ( Sales, 'Date' ) ) ),
        ("CALCULATE", CALCULATE ( COUNTROWS ( Sales ), 'Date'[Calendar Year]="CY 2008")),
        ( "TABLE FUNCTION", COUNTROWS ( ALLEXCEPT ( Sales, 'Date' ) ) )
    },
    'Date'[Calendar Year] = "CY 2009",
    'Product'[Color] = "Red"
)
```

![[Pasted image 20220717185258.png]]

## 相关

```dataview
list 
where contains(tags, "调节器")
or contains(tags, "all家族")
```

## 进阶

[[ALLEXCEPT VS ALL And VALUES]]