---
created: 2022-05-30
tags: 排序
subject: dax
importance: 5
skilled: 3
status: 
author: 
url: "https://dax.guide/topn/"
cover: 
---













### 各品类的Top3商品

```js
    GENERATE (
        ALLSELECTED ( 'Product'[Category] ),         -- For each category
        TOPN (                                       -- retrieve the top
            3,                                       -- three product 
            ALLSELECTED ( 'Product'[Product Name] ), -- names based on the
            [Sales Amount]                           -- sales amount
        )
    )
```
![](https://s2.loli.net/2022/05/30/hcCHmxZFdqjAQ86.png)

在线尝试
[DAX.do by SQLBI](https://dax.do/xYyEreWadfpALx/)