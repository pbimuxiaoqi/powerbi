---
created: 2022-05-15
tags: 关系
subject: dax
importance: 5
skilled: 5
status:
url: https://dax.guide/treatas/
cover: 
---

将输入列赋予新的数据沿袭，并过滤掉在对应的输出列中不存在的值。

## 语法

```js
TREATAS ( <Expression>, <ColumnName> [, <ColumnName> [, … ] ] )
```

| 参数 | 属性 | 描述 |
| ---- | ---- | ---- |
|  表达式    |      |  需要重新映射列集的表表达式    |
|  列名    |   可重复   |  输出的具备[[理解数据沿袭]]的列名    |

## 返回值

`表` 整个表或具有一列或多列的表。 包含 <列名>和<表达式>中共同存在的所有行的表

## 备注

-   如果列中不存在表表达式中返回的值，则将其忽略。 例如，TREATAS({“Red”, “Green”, “Yellow”}, DimProduct[Color]) 对 DimProduct[Color] 列设置了具有“Red”、“Green”和“Yellow”三个值的筛选器。 如果 DimProduct[Color] 中不存在“Yellow”，则有效的筛选器值为“Red”和“Green”。
-   TREATAS 使用第一参数之后的列来分配表达式返回的列的数据沿袭，这些列必须与表表达式中的列数匹配，并按相同的顺序排列。返回结果可以分配给一个变量，因为 TREATAS 不是一个 [[CALCULATE 调节器]]。第一个参数必须是一个表表达式。
-   TREATAS 适合不借助模型关系进行的计算。

## 示例

```js
--  TREATAS can be used as an alternative syntax to apply
--  a filter in CALCULATE/CALCULATETABLE
DEFINE
    MEASURE Sales[Sales Trendy Colors] =
        CALCULATE (
            [Sales Amount],
            'Product'[Color] IN { "Red", "White", "Blue" }
        )
    MEASURE Sales[Sales Trendy Colors 2] =
        CALCULATE (
            [Sales Amount],
            TREATAS ( { "Red", "White", "Blue" }, 'Product'[Color] )
        )
EVALUATE
SUMMARIZECOLUMNS (
    'Product'[Brand],
    "Sales Trendy Colors", [Sales Trendy Colors],
    "Sales Trendy Colors 2", [Sales Trendy Colors 2]
)
```

## 进阶

[[14.5 数据沿袭]]

[[使用TREATAS进行筛选传递]]

## 相关函数
[[USERELATIONSHIP]]
[[CROSSFILTER]]