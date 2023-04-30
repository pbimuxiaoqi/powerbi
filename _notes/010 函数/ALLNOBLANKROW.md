---
created: 2022-05-17
tags:
 - 筛选
 - 表
subject: dax
importance: 3
skilled: 4
status: 
author: 
url: "https://dax.guide/allnoblankrow"
cover: 
---

返回表中除空白行以外的所有行或列中的所有值，忽略可能已应用的任何筛选器。

## 语法

```js
ALLNOBLANKROW ( <TableNameOrColumnName> [, <ColumnName> [, <ColumnName> [, … ] ] ] )
```

|PARAMETER|ATTRIBUTES <div style="width:100px">|DESCRIPTION|
|-|-|-|
|TableNameOrColumnName||模型中物理表或物理列的名称|
|ColumnName|Optional  <br>Repeatable|同一基表中的列。只有当此列也在位于第一参数中时，才可以在后续可选参数中使用|


同一基表中的列。只有当此列也在位于第一参数中时，才可以在后续可选参数中使用

## 返回值

#表

-   作为表函数使用时，返回完整的表或具有一列或多列的表；
-   <font color="red">如果表本身有空值，结果可以包括空值。</font>唯一**不包括**在结果中的空值是在**关系无效**的情况下添加到表中的空值。
-   作为 [[CALCULATE]] 调节器使用时，将所有筛选器替换为<font color="red">仅删除空白行的新筛选器</font>，其他筛选器都被移除。因此，作为参数的所有列或表将只过滤掉空值。

## 备注

>此函数从筛选上下文中删除相应的筛选器。当直接在 CALCULATE 或 CALCULATETABLE 的调节器调用时，它不会具体化结果表。

## 示例

```js
ALLNOBLANKROW ( Customer )
 
ALLNOBLANKROW ( Customer[Country], Customer[State] , Customer[City] )
```

```js
-- 
--  ALLNOBLANKROW still returns blanks, if they are present among the
--  regular rows of the table. The only blank ignored is the one in the
--  blank row
--
EVALUATE
ADDCOLUMNS (
    ALLNOBLANKROW ( Customer[Birth Date] ),
    "# Customers", CALCULATE ( COUNTROWS ( Customer ) )
)
ORDER BY [Birth Date]
```

![](https://secure2.wostatic.cn/static/5bW7vSLmFd1Eb5VJfVgBnk/image.png?auth_key=1652797738-jMs3Ru4yEZFpa2r3DKfda1-0-6bcf0b7d64dcc64bf06a13aa53ea6b16)

### 移除空行
```js
--
--  If you need to remove blanks, you need to use either FILTER
--  or CALCULATETABLE to manually remove blanks.
--
EVALUATE
    ADDCOLUMNS (
        FILTER (
            ALLNOBLANKROW ( Customer[Birth Date] ),
            NOT ( Customer[Birth Date] == BLANK () )
        ),
        "# Customers", CALCULATE ( COUNTROWS ( Customer ) )
    )
ORDER BY [Birth Date]
```
如果果不使用[[FILTER]]
```js
EVALUATE
    ADDCOLUMNS (
        ALLNOBLANKROW ( Customer[Birth Date] ), 
        "# Customers", CALCULATE ( COUNTROWS ( Customer ) ) 
    )
ORDER BY [Birth Date]
```

![](https://s2.loli.net/2022/05/17/UjKXCA6PQJeFZac.png)


## 进阶

[[理解循环依赖]]

[[05.3 循环依赖]]

[[处理DAX中的空白]]

## 相关函数

[[ALL]]

[[ALLEXCEPT]]

[[ALLSELECTED]]