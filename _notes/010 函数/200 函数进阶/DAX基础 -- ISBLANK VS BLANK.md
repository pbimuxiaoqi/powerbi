---
created: 2022-06-19
tags:
 - 空白
 - blank
subject: dax进阶
importance: 
skilled: 
status: 
author: sqlbi
url: 
cover: 
---

好久没更新了，今天来分享下两个很简单却很容易混肴从面犯错的函数，ISBLANK和BLANK。

先来看语法

| 函数    | 语法                | 返回值                     |
| ------- | ------------------- | -------------------------- |
| ISBLANK | ISBLANK ( <Value> ) | 值为空则返回True,反之False |
| BLANK   | BLANK ( )           | 任何类型的值               |



表面上看两个函数只有有无参数的区别，实际上是这样吗？

打开Dax studio,先来看一组结果，从结果上看是相同的，这也是为什么很多人会误以为它们是一样的。

![](https://secure2.wostatic.cn/static/cxqQNVcGS9chkvAQcKAWa3/image.png?auth_key=1655640287-tyPuQa9r8JtQam9WZ7hCLJ-0-22a40f8dea0320561c1f78d839a2a19e)

接下来看下面的例子，这时会发现结果很能和我们先前想的不一样了，ISBLANK将0和空字符判定为了False，而BLANK会认为它们是True,除非使用==。

![](https://secure2.wostatic.cn/static/fXa6nACB2j9PnJ5v14B5EZ/image.png?auth_key=1655640296-5PHzbXfZeyEGt5tK6wHYCB-0-750df9b290182d387681b095d6a3b91e)

这时我们再来总结一下，

-   ISBLANK( <VALUE> ) 等效于 <value> == BLANK()
-   ISBLANK认为0和空字符串并不为空

接下来在实际报表中来应用下上面的知识，先创建几个基础度量

```js
Sales Amount = SUM( 'FactInternetSales'[SalesAmount])

LM.Sales Amount = 
CALCULATE( [Sales Amount], DATEADD( 'DimDate'[FullDateAlternateKey], -1, MONTH))


MOM%.Sales Amount = 
DIVIDE( [Sales Amount] - [LM.Sales Amount], [LM.Sales Amount] )

```

![](https://secure2.wostatic.cn/static/2shFcK7sKYRFMJwwnizu4Z/image.png?auth_key=1655640304-8DWJ5iK7sTSag9dKSFj5Wj-0-d4a782653a240113de87daff626ea1f5)

表格中有两行的结果我们需要注意

-   2011年1月的上期为空，即分母为空，所以结果也为空
-   2014年3月本期为空，即分子为空，所以结果为-100%

接下来换另一种写法，从数学上讲（A-B）/B 是等价于 A/B - 1的，所以我们这样写同比

```js
MOM%.Sales Amount 2 = 
DIVIDE( [Sales Amount], [LM.Sales Amount] ) - 1
```

![](https://secure2.wostatic.cn/static/m7KcnhZNaKatSJQ6tcYYF4/image.png?auth_key=1655640317-hTw4QQq8ckkcq5yTwHpco-0-23d0e640359d2f4c519ef91273fa565e)

造成这个结果的原因是在加减法中BLANK会自动转换为0，0-1肯定是-1无疑了。所以为了让未发生的日期不显示负值，通常会做下判断

```js
MOM%.Sales Amount 3 = 
IF(
    NOT ISBLANK( [Sales Amount] ),
    DIVIDE( [Sales Amount] - [LM.Sales Amount], [LM.Sales Amount] )
)
```

![](https://secure2.wostatic.cn/static/btuwSkCZq1YgxvJLVj3Pwv/image.png?auth_key=1655640324-qVc5KcfShcnDUUNHPoEYYp-0-435e5bd5813eb487f20c12820b8378cf)

接下来再看另外一个例子，假设我们想统计产品数量

```js
Products = 
COUNTROWS (  'DimProduct' )


Product 2 = 
COUNTROWS ( VALUES ( 'DimProduct' ) )

```

![](https://secure2.wostatic.cn/static/65Z7YVeDx2c79ieRmS6Hxq/image.png?auth_key=1655640334-243bPjSA81X77K6GFXm4iu-0-d0749f1f3bb67d3f6b8e75e5a783b06c)

结果一样，也没什么问题，那么接下来，回去PQ页面，过滤到Product表中的Color为Black的数据，再来看结果

![](https://secure2.wostatic.cn/static/kS46LfuDAegyv8Y8YsAWEe/image.png?auth_key=1655640343-w3X4LDhSKLStjijdKqqpnp-0-aea68cb9d3bbd1c3b85307f365459ca0)

结果不再相同，如果我们是求不同颜色的产品占所有产品的占比，那这里得到的结果也会不同。这里其实涉及到了一个概念，DAX中使用的表是扩展表。也就是说虽然我们删除了产品表中Color为Black的产品，但是Sales表是有这些产品的销售记录的，它们无法和现在的Product表的中记录相匹配，就会返回空，这也是很多时候我们会发现日期切片器有空白项的原因，事实表中的有无法和维度表匹配的记录。

每一个DAX函数看起来都很简单，但是当这些简单的DAX组合起来用时，也许会产生让我们难以理解的结果，这在写一些复杂地度量值是需要额外注意。

扩展阅读

[How to handle BLANK in DAX measures - SQLBI](https://www.sqlbi.com/articles/how-to-handle-blank-in-dax-measures/)

[Handling BLANK in DAX - SQLBI](https://www.sqlbi.com/articles/blank-handling-in-dax/)