---
created: 2022-07-18
tags: 
- 调节器 
- all家族
- 筛选
subject: dax
importance: 5
skilled: 5
status:
author:
url: https://dax.guide/removefilters
cover: 
---

```dataview
list 
without id
url
where file.name = this.file.name
```
清除指定表或列中的筛选器

## 语法

```js
REMOVEFILTERS ( [<表名或列名>], [<列名>], [<列名>] … )
```

|参数|属性|描述|
|-|-|-|
|表名或列名|可选|要清除筛选器的表或列|
|列名|可选，可重复|要清除筛选器的列（物理表）|

## 返回值

不返回任何值

## 备注

-   REMOVEFILTERS 只能用于清除筛选器，不能返回表。
-   REMOVEFILTERS 是 [[ALL]] 的别名，但它只能用作 CALCULATE 调节器，而不能像 ALL 那样用作表表达式。

## 示例

```js
-- Filter Litware/Red
EVALUATE
CALCULATETABLE (
    SUMMARIZE ( 'Product', 'Product'[Category], 'Product'[Brand], 'Product'[Color] ),
    Product[Brand] = "Litware",
    Product[Color] = "Red"
)

-- Remove all the filters from Product: as a result, both Litware and Red filters are removed
EVALUATE
CALCULATETABLE (
    CALCULATETABLE (
        SUMMARIZE ( 'Product', 'Product'[Category], 'Product'[Brand], 'Product'[Color] ),
        REMOVEFILTERS ( 'Product' )
    ),
    Product[Brand] = "Litware",
    Product[Color] = "Red"
)
```

![[Pasted image 20220718212624.png]]
![[Pasted image 20220718212635.png]]

```js
-- Filter Litware/Red
EVALUATE
CALCULATETABLE (
    SUMMARIZE ( 'Product', 'Product'[Category], 'Product'[Brand], 'Product'[Color] ),
    Product[Brand] = "Litware",
    Product[Color] = "Red"
)
 
-- Remove all the filters from Product: as a result, both Litware and Red filters are removed
EVALUATE
CALCULATETABLE (
    CALCULATETABLE (
        SUMMARIZE ( 'Product', 'Product'[Category], 'Product'[Brand], 'Product'[Color] ),
        REMOVEFILTERS ( 'Product' )
    ),
    Product[Brand] = "Litware",
    Product[Color] = "Red"
)
```
![[Pasted image 20220718213002.png]]

![[Pasted image 20220718213014.png]]
## 相关
```dataview
list 
where contains(tags, "all家族")
sort subject
```
