
## 前言

测试数据集：

ca | cb | cc
---- | ---|---
A| W|1
A| W|2
B| X|1
B| Z|2
B| Z|4


### 按最大/最小值/TOP1去重

按ca和cb取cc最小值取值：

```
select 
	ca, 
	cb, 
	min(cc)
from table
group by ca, 
	cb
```

column_A | column_B | column_C
---- | ---|---
A| W|1
B| X|1
B| Z|2

取最大，则使用max()函数。取一组值的TOP1也是同理。


### 按列合并多行（多->少）

```
select 
	ca, 
	cb, 
	groupUniqArray(cc)
from table
group by ca, cb
```

column_A | column_B | column_C
---- | ---|---
A| W|[2,1]
B| X|[1]
B| Z|[4,2]

这是一种分组的概念，将相同的数据放在一起，需要被统计的数据放在array数据类型中。统计array，可以使用length()，获取数组长度，相当于分组count

此外，groupArray也可以满足需求。

### 分组排序后取TopN

```

SELECT ca,
       groupArray(1)(cc) 
FROM
  ( SELECT *
   FROM table
   ORDER BY ca,
            cb,
            cc )
GROUP BY ca
```

column_A | column_B | column_C
---- | ---|---
A| W|[1]
B| X|[1]
B| Z|[2]

以上是按cc列的值升序去top1。通过order by x改变顺序，再用groupArry(N)()函数处理获取top值。

