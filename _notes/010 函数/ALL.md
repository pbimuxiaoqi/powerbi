---
tags: 
- 筛选 
- 调节器 
- 表
- all家族
created: 2022-05-10
returns: 表
importance: 5
skilled: 4
subject: dax
url: 
---

---

返回表中的所有行或列中的所有值，忽略可能已应用的任何筛选器。

## 语法

```js
ALL ( [<表名或列名>] , [ <列名>, … ] )
```

|PARAMETER|ATTRIBUTES|DESCRIPTION|
|--|--|--|
|表名或列名|Optional|模型中物理表或物理列的名称|
|列名|Optional |Repeatable|

同一基表中的列。只有当此列也在位于第一参数中时，才可以在后续可选参数中使用

## 返回值

#表  作为表函数使用时，ALL返回完整的表或具有一列或多列的表；作为 CALULCATE 调节器使用时，ALL 移除参数中已应用的任何直接筛选器。

## 备注

当用于 CALCULATE 或 CALCULATETABLE的筛选器参数时，ALL 不会返回表，而是和 REMOVEFILTERS 一样，从筛选上下文中删除相应的筛选。从避免歧义的角度，建议在这种情况下使用 REMOVEFILTERS。

当 ALL 至少有一个参数时，它可以作为表表达式使用。没有参数的 ALL 只能作为 CALCULATE 或 CALCULATETABLE 的调节器使用，并且从筛选上下文中删除所有的筛选。

以下内容在使用 ALL 作为表表达式时是有效的：

-   使用表参数时，ALL 返回表的所有行，包括任何重复的行。
    
-   使用单列参数，ALL 返回该列的所有唯一值。
    
-   使用两列或多列参数，ALL 返回多列中所有唯一的值组合。
    
-   在每一种情况下，ALL 都会在结果中包含为无效关系生成的额外空白行。
    

## 示例

```js
ALL ( Customer )    //返回完整的客户表
```

```js
ALL ( Customer[Country], Customer[State] , Customer[City] )    //返回客户表来自国家、州、城市三列的所有不重复组
```

```js
CALCULATE ( COUNTROWS ( Sales ), ALL ( Customer ) )    //删除客户表的所有筛选
```

```js
ALL ( Customer ) //返回完整的客户表
```

```js
ALL ( Customer[Country], Customer[State] , Customer[City] ) //返回客户表来自国家、州、城市三列的所有不重复组合
```

```js
CALCULATE ( COUNTROWS ( Sales ), ALL ( Customer ) ) //删除客户表的所有筛选
```

```js
ALL ( Customer )    //返回完整的客户表
```

```js
ALL ( Customer[Country], Customer[State] , Customer[City] )    //返回客户表来自国家、州、城市三列的所有不重复组合
```

```js
CALCULATE ( COUNTROWS ( Sales ), ALL ( Customer ) )    //删除客户表的所有筛选
```

```js
--
--  ALL with a table works on the expanded table, removing filters
--  from any column in the expanded table
--
EVALUATE
CALCULATETABLE (
    {
         ( "Sales Amount ", [Sales Amount] ),
         ( "Sales Amount (ALL Colors)", CALCULATE (
            [Sales Amount],
            ALL ( 'Product'[Color] )
        ) ),
         ( "Sales Amount (ALL Products)", CALCULATE (
            [Sales Amount],
            ALL ( 'Product' )
        ) ),
         ( "Sales Amount (ALL)", CALCULATE (
            [Sales Amount],
            ALL ()
        ) ),
         ( "Sales Amount (ALL Sales)", CALCULATE (
            [Sales Amount],
            ALL ( Sales )
        ) )
    },
    'Product'[Color] = "Red",
    'Date'[Calendar Year] = "CY 2008"
)
```

![[Pasted image 20211107221430.png]]

## 进阶

[[ALLEXCEPT VS ALL VS VALUES]]

[[理解ALL家族]]