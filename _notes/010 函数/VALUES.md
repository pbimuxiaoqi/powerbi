---
created: 2022-07-18
tags:
- 表
- 表操作
subject: dax
importance: 5
skilled: 5
status:
author:
url: https://dax.guide/values
cover: 
---

```dataview
list url
where file.name = this.file.name
```
- 当使用列参数时，返回指定列非重复值组成的表；

- 当使用表作为参数时，返回指定表中的行（保留重复行）。VALUES遵循参照完整性的约束添加空值。

## 语法

```js
VALUES ( <表名或列名> )
```

|参数|ATTRIBUTES|DESCRIPTION|
|-|-|-|
|表名或列名||表（基表）或列|


当使用表作为参数时，VALUES 只接受物理表，不接受返回表的表达式。而另一个相似的函数 [[DISTINCT]] 可以使用返回表的表达式，并且会删除表中的重复行



## 返回值

表，整个表或具有一列或多列的表。

## 备注

- DISTINCT 函数允许将列名或任何有效的表表达式作为其参数，但是 VALUES 函数仅接受列名或表名作为其参数。
- 当使用列作为参数时，在大多数情况下，VALUES 函数的结果与 DISTINCT 函数的结果相同。 这两个函数都会删除重复项，并返回指定列中可能的值的列表。 但是，**VALUES 函数还可以返回空白值**。   
- **如果返回表的表达式结果是包含一行一列的表, 则可以转换为标量值, 这种转换在需要时自动完成。**

## 示例

### 返回唯一值的列表

这一点很像 ALL 函数，VALUES 只返回不同的值。对于一列颜色 {Red, Red, Red, Blue, Red, Red, Red}， VALUES 和 ALL 函数都将返回一个表，该表由一列组成，该列是{Red, Blue}。这两个函数的不同之处在于 VALUES 在筛选上下文中计算，而 ALL 忽略筛选上下文。

### 将列转换为表

VALUES 的作用之一是将列转换为表。这对于需要调用表的函数非常有用。比如 [[SUMX]]、[[FILTER]]、[[CALCULATE]]、[[COUNTROWS]]、[[TOPN]]，有很多类似的函数需要表作为参数。唯一要注意的是，VALUES 执行的不是从列到表的直接转换，因为只有唯一的值会存在。

### 引入外部筛选上下文

这也是一种经典用法，ALL 会移除所有筛选器，而使用 VALUES 可以将已经被忽略的特定筛选上下文重新引入。

```js
CALCULATE([Measure], ALL(Calendar), VALUES(Calendar[Year]))
```

尽管 ALL(Calendar)移除了 Calendar 表上的所有筛选器，但 VALUES(Calendar[Year])从初始筛选上下文中恢复了年份列的筛选器。等价于下面的写法：

```js
CALCULATE([Measure], ALLEXCEPT(Calendar, Calendar[Year]))
```

### 检查正在生效的筛选上下文

```js
=IF (HASONEVALUE(Calendar[Month]),
       IF (VALUES(Calendar[Month]) = "April", "April 被筛选了", "其他月份"),
       "多月份"
  )
```

### 将列转为标量值

如果在只有一个值的单列上调用 VALUES(或者有多个相同的值)，那么它将返回一个可以用作标量的值，上文已经介绍过，这是 DAX 语言的一种特性。




## 相关

[[03.5 VALUES与DISTINCT]]