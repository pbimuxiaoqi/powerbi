---
created: 2022-07-17
tags: all家族 
subject: dax进阶
importance: 5
skilled: 3
status:
author:
url: https://www.sqlbi.com/articles/using-allexcept-versus-all-and-values/
cover: 
---

```dataview
list url
where file.name = this.file.name
```
## 附件
![[Using ALLEXCEPT vs ALL - VALUES.pbix]]


同ALL家族的其他函数一样，[[ALLEXCEPT]]可以提供两种不同的行为

-   作为表函数
-   作为[[CALCULATE]]的修饰符

但是，ALLEXCEPT很少作为表函数使用，主要与CALCULATE一起使用

在dax中有两种方式可以从表中删除除某些列之外的所有筛选器

```js
UsingAllExcept :=
CALCULATE (
    [Sales Amount],
    ALLEXCEPT (Customer, Customer[Continent] )
)
 
UsingAllValues :=
CALCULATE (
    [Sales Amount],
    ALL ( Customer ),
    VALUES ( Customer[Continent] )
)
```

上面这两种方法语义上看起来相同，但是这两种方法会导致不同的行为。使用ALLEXCEPT通常很容易导致错误，因为当**它作为调节器使用时对外部筛选上下文更加敏感**。

另外，[[REMOVEFILTERS]]当作为调节器使用时，它是ALL的别名，会产生更具可读性的代码

```js
UsingRemoveFiltersValues :=
CALCULATE (
    [Sales Amount],
    REMOVEFILTERS ( Customer ),
    VALUES ( Customer[Continent] )
)
```

在大多数情况下，REMOVEFILTERS和[[VALUES]]是成对出现的，此外应该将ALLEXCEPT看作CALCULATE的调节器，比如在计算列中删除循环依赖，

[[理解循环依赖]]

来看下面的例子

![[Pasted image 20220717223542.png]]

为了计算 PercOverContinent，我们需要将 Sales Amount（即当前筛选上下文中的 Sales Amount 度量）除以筛选上下文中的相同度量，我们从 Customer 表中删除除 Continent 列之外的所有筛选器

```js
PercOverContinent1=
VAR SelSales = [Sales Amount]
VAR ConSales =
    CALCULATE (
        [Sales Amount],
        ALLEXCEPT (
            'Customer',
            'Customer'[Continent]
        )
    )
VAR Result =
    DIVIDE ( SelSales, ConSales )
RETURN
    Result
```

当然，也可以使用不同的方式来编写，比如使用[[REMOVEFILTERS]]，

```js
PercOverContinent2 = 
VAR SelSales = [Sales Amount]
VAR ConSales =
    CALCULATE (
        [Sales Amount],
        REMOVEFILTERS( 'Customer' ),
        VALUES( 'Customer'[Continent] )
    
    )
VAR Result =
    DIVIDE ( SelSales, ConSales )
RETURN
    Result
```

![[Pasted image 20220717223550.png]]
现在我们聚焦到法国，它的占比是11.48%，假设我们现在移除continent列，会现使用ALLEXCEPT的写法变成了3.32%

![[Pasted image 20220717223556.png]]

这是因为，当我们使用ALLEXCEPT时，我们创造了一个当且仅当存在continent列时才有效的度量值，如果报告未在continent上使用筛选器，则该度量值无法正确显示，细心的会发现3.32%是法国占全球销售额的占比。

要理解这个原因，我们必须关注法国这一行的筛选上下文，当度量值按预期工作时行上存在以下上下文

![[Pasted image 20220717223604.png]]
当 ALLEXCEPT 作为分母中的 CALCULATE 调节器使用时，ALLEXCEPT 会从 Customer 表中删除所有筛选器（有两个筛选器），除了 Continent 上的筛选器。因此，ALLEXCEPT 会从 Country 上删除筛选器。因此，分母按预期计算了欧洲的销售额。

但是当移除了continent列，即值为3.32%时，筛选上下文并不包含continent的筛选器，因此只包含country的筛选器

![[Pasted image 20220717223611.png]]

通常新手会忘记ALLEXCEPT作为调节器使用时不会引用新的筛选上下文，它只能删除现有的。

所以，更安全的做法是使用REMOVEFILTERS和VALUES组合。

当评估法国时， VALUES( 'Customer'[Continent] )返回欧洲，这里要注意的是VALUES始终将其筛选器应用于筛选上下文

<font color="red">当需要在单列上保留筛选时可以使用[[VALUES]],当需要在多列上使用时，使用[[SUMMARIZE]]</font>

```js
PercOverState :=
VAR SelSales = [Sales Amount]
VAR StateSales =
    CALCULATE (
        [Sales Amount],
        REMOVEFILTERS ( 'Customer' ),
        SUMMARIZE (
            'Customer',
            'Customer'[Continent],
            'Customer'[Country],
            'Customer'[State]
        )
    )
VAR Result =
    DIVIDE ( SelSales, StateSales )
RETURN
    Result
```


## 层级占比
下面的写法和层级占比有些类似
```js
PercOverState=
VAR SelSales = [Sales Amount]
VAR StateSales =
CALCULATE (
	[Sales Amount],
	REMOVEFILTERS ( 'Customer' ),
	SUMMARIZE (
		'Customer',
		'Customer'[Continent],
		'Customer'[Country],
		'Customer'[State]
	)
)

VAR Result =
DIVIDE ( SelSales, StateSales )
RETURN
Result

```
![[Pasted image 20211124134041.png]]

```js
Perc =
VAR SelSales = [Sales Amount]
VAR Result =
SWITCH(
TRUE(),
ISINSCOPE('Customer'[City]), DIVIDE( [Sales Amount], CALCULATE( [Sales Amount], REMOVEFILTERS('Customer'), VALUES('Customer'[State]) ) ),

ISINSCOPE('Customer'[State]), DIVIDE( [Sales Amount], CALCULATE( [Sales Amount], REMOVEFILTERS('Customer'), VALUES('Customer'[Country]) ) ),

ISINSCOPE('Customer'[Country]), DIVIDE( [Sales Amount], CALCULATE( [Sales Amount], REMOVEFILTERS('Customer'), VALUES('Customer'[Continent]) ) ),

DIVIDE( [Sales Amount], CALCULATE( [Sales Amount], REMOVEFILTERS('Customer') ) )

)

RETURN

Result
```
![[Pasted image 20211124144425.png]]