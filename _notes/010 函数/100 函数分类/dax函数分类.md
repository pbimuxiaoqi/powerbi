
## 重点函数
```dataview
table importance, skilled, tags
where importance = 5
and subject = "dax"

```

## 筛选

```dataview
table importance, skilled
where contains(tags, "筛选")
and subject = "dax"
```

## 时间智能
```dataview
table importance, skilled, tags
where contains(tags, "时间智能")
and subject = "dax"

```

## 迭代

```dataview
table score
from "010 函数"
where contains(tags, "迭代")

```

## 表函数
```dataview
table importance, skilled, tags
where contains(tags, "表")
and subject = "dax"

```