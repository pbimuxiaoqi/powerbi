---
created: 2022-05-21
tags: 空白 空值
subject: dax进阶
importance: 5
skilled: 3
status:
author: sqlbi
url: https://www.sqlbi.com/articles/blank-row-in-dax/ https://www.sqlbi.com/articles/how-to-handle-blank-in-dax-measures/
cover: 
---

```js
Net Amount % 1 := 1 - ( [Discount] / [Amount] )
Net Amount % 2 := ( [Amount] - [Discount] ) / [Amount]
```

返回结果并不相同

![](https://s2.loli.net/2022/05/21/yKWw7jUP92ICe4p.png)


这是因为[[BLANK]]值在加减法中自动转换为了0，但是在乘除法中仍然作为[[BLANK]]使用

最好的方式是进行拆分

```js
Net Amount % 3 :=
VAR DiscountPercentage =
    DIVIDE ( [Discount], [Amount] )
RETURN
    IF (
        NOT ISBLANK ( DiscountPercentage ),
        1 - DiscountPercentage
    )
```

```js
Net Amount % 4 =
VAR Amount = [Amount]
VAR Discount = [Discount]
VAR DiscountPercentage = Discount / Amount
RETURN
    IF (
        NOT ISBLANK ( Amount ),
        1 – DiscountPercentage
    )
```

```js
Net Amount % 5 =
VAR Amount = [Amount]
VAR Discount = [Discount]
RETURN (Amount - Discount) / Amount
```

## 常用

![](https://s2.loli.net/2022/05/21/XAF5INwpQHxeCba.png)


自动转换为0可能会产生错误

![](https://s2.loli.net/2022/05/21/KJhvTHRi26Ao85D.png)


![](https://s2.loli.net/2022/05/21/KJhvTHRi26Ao85D.png)

## Switch中应用

内部执行[[ISBLANK]]

![](https://s2.loli.net/2022/05/21/x9CSLoJWdRwQq3g.png)


## 表表达式

![](https://s2.loli.net/2022/05/21/x9CSLoJWdRwQq3g.png)

![](https://s2.loli.net/2022/05/21/ZxEA8RuIw94XgHY.png)


## 结果不会返回BLANK

![](https://s2.loli.net/2022/05/21/ZxEA8RuIw94XgHY.png)

先看下面两个度量

```js
#Colors Distinct := COUNTROWS ( DISTINCT ( 'Product'[Color] ) )
#Colors Values := COUNTROWS ( VALUES ( 'Product'[Color] ) )
```

![](https://s2.loli.net/2022/05/21/YB572bQVUXd1zgx.png)


结果看上去相同，实际上它们的运行原理是完全不同的

-   VALUES 当使用列参数时，返回指定列非重复值组成的表
-   DISTINCT返回非重复值的表

接下来做一点改变，删除所有颜色为Pink的值，再来看效果

![](https://s2.loli.net/2022/05/21/YB572bQVUXd1zgx.png)

**这里要注意sales表和product表之间的关系**，因为在产品表中删除了Plink的值，Sales表中无法匹配到，

假设想统计产品数量

```js
#Products := COUNTROWS ( 'Product' )

```

```js
#Products Values := COUNTROWS ( VALUES ( 'Product' ) )
```

![](https://s2.loli.net/2022/05/21/cOnVlSBTmRpHZ3z.png)


如果想计算每个品牌的产品数占总产品数的比例，可以简单写成下面

```js
Perc #Prods :=
DIVIDE (
    COUNTROWS ( 'Product' ),
    COUNTROWS ( ALL ( 'Product' ) )
)
```

![](https://s2.loli.net/2022/05/21/cOnVlSBTmRpHZ3z.png)

然而最终汇总结果不是100%，这是因为分子没有考虑空行，总数是2433，而分母使用了ALL包括了空行，总数是2434

可以使用如下方法

-   ALLNOBLANKROW 不考虑空白行

```js
Perc #Prods Values :=
DIVIDE (
    COUNTROWS ( VALUES ( 'Product' ) ),
    COUNTROWS ( ALL ( 'Product' ) )
)
 
Perc #Prods Distinct :=
DIVIDE (
    COUNTROWS ( DISTINCT ( 'Product' ) ),
    COUNTROWS ( ALLNOBLANKROW ( 'Product' ) )
)
```

![](https://s2.loli.net/2022/05/21/7ErPhMbBUn5LxmJ.png)

## 更多
[[BLANK VS ISBLANK]]
[[020 案例/使用0来代替BLANK]]