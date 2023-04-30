---
created: 2022-05-21
tags: 
- 功能函数 
- 标量
subject: dax
importance: 5
skilled: 4
status:
author:
url: https://dax.guide/blank/
cover: 
---

返回空白

## 语法

```js
BLANK ( )
```

## 返回值

#标量 一个任意类型的值， 空值没有数据类型

## 备注

-   BLANK 值在与其他值比较时自动转换类型。
-   检查一个值是否为 BLANK 的正确方法是使用运算符 == 或 ISBLANK 函数。不要使用运算符 = 。
-   运算符==是一个 “严格等于 “的运算符，它将 BLANK 与 0 或空字符串区别对待。

> BLANK 不对应 SQL 中的 NULL。DAX 中的 BLANK 不遵循 NULL 在 SQL 中的计算逻辑。在中间结果可能是 BLANK 的表达式中，必须注意这种区别。

## 示例

```js
--  BLANK is equal to 0 and to an empty string in DAX.
--  You need to use == to check for "strictly equal to"
EVALUATE
    {
        ( "Blank = 0", BLANK () = 0 ),
        ( "Blank = """"", BLANK () = "" ),
        ( "Blank == 0", BLANK () == 0 ),
        ( "Blank == """"", BLANK () == "" )
    }
```

![](https://s2.loli.net/2022/05/21/3fgU6xb7BeQa9Vc.png)


```js
--  BLANK is useful also to provide BLANK arguments to some functions
--  like DATESBETWEEN.
EVALUATE
    DATESBETWEEN (
        'Date'[Date],
        DATE ( 2011, 12, 20 ),
        BLANK ()
    )
ORDER BY 'Date'[Date]
```

![](https://s2.loli.net/2022/05/21/hWOHAwXpfa8guRn.png)



```js
--  BLANK returns the BLANK value
--
--  It is mostly useful to blank out the result of a calculation
--  toor  perform comparisons.
DEFINE
    MEASURE Customer[EmptyNames] =
        CALCULATE (
            COUNTROWS ( Customer ),
            Customer[Customer Name] == BLANK ()
        )
EVALUATE
SUMMARIZECOLUMNS (
    Customer[Continent],
    "Customers", COUNTROWS ( Customer ),
    "Customers with blank name", [EmptyNames]
)
```

![](https://s2.loli.net/2022/05/21/p47gPHz8rKeVEob.png)


## 相关函数

[[ISBLANK]]

## 进阶

[[优化DAX代码可能存在空值的条件]]

[[处理DAX中的空白]]
