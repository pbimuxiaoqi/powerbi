---
created: 2022-06-19
tags: 
- 空白 
- blank
subject: 可视化
importance: 5
skilled: 3
status:
author: sqlbi
url: https://www.wolai.com/muxiaoqi/tLQ1HES8DFDtaBuJf4n6ff
cover: 
---
https://www.sqlbi.com/articles/how-to-return-0-instead-of-blank-in-dax/

从前面的文章中我们已经知道PowerBI中主要是使用[[SUMMARIZECOLUMNS]]来运行查询的。当查询计算的所有列都返回BLANK时，则不会返回任何结果，但是有时候用户希望看到产生的空白行，因为这些空白值是有意义的。

比如下面的报告

![](https://secure2.wostatic.cn/static/gRJR4QiCbn6tDJPyBb1fdV/image.png?auth_key=1655636177-3r2whh2PGu5Jxz59FzLCvV-0-ee1ba8f2183188a475c9de807c3560e2)

![](https://secure2.wostatic.cn/static/uUA96jTT2N3PKp6pdfdpJ/image.png?auth_key=1655636184-fddTdbodTPddtc923FsvLR-0-05bdf4bd92d7816c0f71182c879302e5)

![](https://secure2.wostatic.cn/static/gmP8gp2fcqBbBjpMSXvQeC/image.png?auth_key=1655636192-8epzsfYzSHn9NjKizBt3X-0-02f1b262d86b17cfcfb06ebb22ef0b62)

我们首先想到的解决方案可能是度量值+0，确实很简单，但会造成所有空值都变成0

一个解决方案如下：

```js
SalesZero =
VAR FirstSaleEver =
    CALCULATE ( MIN ( 'Sales'[Order Date] ), ALLEXCEPT ( Sales, 'Product' ) )
VAR LastSaleEver =
    CALCULATE ( MAX ( 'Sales'[Order Date] ), REMOVEFILTERS () )
VAR CurrentDate = MAX ( 'Date'[Date] )
VAR ForceZero = FirstSaleEver <= CurrentDate && CurrentDate <= LastSaleEver 
VAR Amt = [Sales Amount] + IF ( ForceZero, 0 )
RETURN
    Amt
```

还可以修改判断以适应不同的需求，比如

```js

VAR ForceZero = FirstSaleEver <= MAX ( 'Date'[Date] ) && MIN ( 'Date'[Date] ) <= LastSaleEver


```

一个更好的解决方案是将初始销售日期相关的信息存储在单独的表中，

```js
ZeroGrain =
VAR MaxSale = MAX ( Sales[Order Date] )
VAR ProdsAndDates =
    GENERATE (
        'Product',
        VAR FirstSaleOfProduct = CALCULATE ( MIN ( 'Sales'[Order Date] ) )
        VAR Dates = DATESBETWEEN ( 'Date'[Date], FirstSaleOfProduct, MaxSale )
        RETURN
            Dates
    )
VAR Result =
    SELECTCOLUMNS (
        ProdsAndDates,
        "ProductKey", 'Product'[ProductKey],
        "Date", 'Date'[Date]
    )
RETURN
    Result
```

![](https://secure2.wostatic.cn/static/6CDQ4DPvy4A2z8ZFuo1W2a/image.png?auth_key=1655636202-wN8MiixLFud8GJNDbcHYXC-0-58a7269fed10e12c14cf83093429c379)

但是这个表太大了，它接近于Product和Date表的交叉连接

```js
SalesZeroWithTable =
VAR ForceZero = COUNTROWS ( ZeroGrain ) > 0
VAR Amt = [Sales Amount] + IF ( ForceZero, 0 )
RETURN
    Amt
```

![](https://secure2.wostatic.cn/static/cYcx2x32v6m8XtKoWbVMBw/image.png?auth_key=1655636209-hgasprf1i5vXQJtpEoZuuW-0-bc6d2dfaefe2a175cc610cbefaf35242)

## 方案三
生成一张计算表，包含所有商品的所有的订单日期

```js
ZeroGrain = 
CROSSJOIN(
    ALLNOBLANKROW( 'Sales'[OrderDate] ),
    ALLNOBLANKROW( 'Product'[ProductKey] )
    )
```

![](https://secure2.wostatic.cn/static/egw49GkDhnoJcgkJ3Qmxmx/image.png?auth_key=1655636318-rzhkpbdCozsxFN6xTqGmHM-0-35706375e122781bd282af933eadc3b2)

建立模型关系

![](https://secure2.wostatic.cn/static/5uTyUnczeHx12m6zMyJMod/image.png?auth_key=1655636328-7eepjwVrTAXz1x6XTnYMwH-0-02b5056232ec9aaed4c9b06867397a73)