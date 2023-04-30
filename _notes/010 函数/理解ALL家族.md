---
created: 2022-07-17
tags: all家族 
subject: dax进阶
importance:
skilled:
status:
author:
url: https://www.sqlbi.com/articles/managing-all-functions-in-dax-all-allselected-allnoblankrow-allexcept/
cover: 
---

```dataview
list url
where file.name = this.file.name
```

返回当前颜色占所有颜色的总占比

```js
PctOverAllColors :=
DIVIDE (
    [SalesAmount],
    CALCULATE (
        [SalesAmount],
        ALL ( Product[Color] )
    )
)
```

看下下面两个指标，

-   NumOfProducts很简单，计算产品的总数
-   NumOfProductsSold 则是通过表筛选计算有销售额的产品数

```js
NumOfProducts :=
DISTINCTCOUNT ( Product[ProductName] )
 
NumOfProductsSold :=
CALCULATE (
    [NumOfProducts],
    Sales
)
```

![[Pasted image 20220717224929.png]]

![[Pasted image 20220717224935.png]]

至此可能会想如果是计算有销售额的商品的占比应该是这样写

```js
PercOfProductsSold =
DIVIDE (
    CALCULATE (
        [NumOfProducts],       -- Number of products 
        Sales                  -- filtered by Sales
    ),     
    CALCULATE (
        [NumOfProducts],       -- Number of products 
        ALL ( Sales )          -- filtered by ALL Sales
    )  
)
```

遗憾的是我们得到了错误的结果，

![[Pasted image 20220717224942.png]]

我们来分析下为什么产生了错误的结果。

-   [[ALL]]是一个表函数，返回表中的所有行或列中的所有值。**但是当作为[[CALCULATE]]调节器使用时不用于从表中检索值，而是用于从筛选上下文中删除筛选**。
-   为了ALL的不同语义，于是出现在另一个函数[[REMOVEFILTERS]],它只能用作CALCULATE调节器，**虽然 REMOVEFILTERS 可以替换 ALL，但不能替换用作 CALCULATE 调节器的 ALLEXCEPT 和 ALLSELECTED  **。

所以上面结果之所以是错误的，是因为ALL删除了Sales上的筛选上下文，所以分母变成了产品总数即2517

这时候我可以强制转换上下文，使用CALCULATETABLE,

```js
PercOfProductsSold =
DIVIDE (
    CALCULATE (
        [NumOfProducts],
        Sales
    ),
    CALCULATE (
        [NumOfProducts],
        CALCULATETABLE ( ALL ( Sales ) )
    )
)
```

可以得到正确结果 ，但是它实际上是改变了公式的语义，明确表示需要使用ALL ( Sales )的结果才能筛选，可以用下面的方式获得类似的效果

```js
PercOfProductsSold =
DIVIDE (
    CALCULATE (
        [NumOfProducts],
        Sales
    ),
    CALCULATE (
        [NumOfProducts],
        FILTER ( ALL ( Sales ), 1 = 1 )
    )
)
```

![[Pasted image 20220717224951.png]]

```js
NoFilterOnProduct =
    CALCULATE (
        [Sales Amount],
        ALLEXCEPT ( Sales, Sales[ProductKey] )
    )
```

初看代码，可能会想ALLEXCEPT从Sales表上删除了除ProductKey之外的所有筛选，实际却并非如此，

ALLEXCEPT是从Sales的扩展表上删除筛选器，其中包括与Sales是一对多的表，customer表，dates表，stores表等

为防止ALLEXCEPT从扩展表中删除筛选，可以如下写

```js
NoFilterOnProduct =
    CALCULATE (
        [Sales Amount],
        ALLEXCEPT (
            Sales,
            Sales[ProductKey],
            Date,
            Customer,
            Store
        )
    )
```

![[Pasted image 20220717224958.png]]