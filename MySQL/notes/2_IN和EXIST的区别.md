# IN
```
SELECT *
FROM A
WHERE id IN(
    SELECT id
    FROM B
)
```
- 等价于
```
for SELECT id FROM B
    for SELECT * FROM A
        WHERE A.id = B.id
```

- 使用

当B表的数据必须小于A表数据时,用IN优于用EXISTS

# EXISTS
```
SELECT * 
FROM A
WHERE EXISTS(
    SELECT 1(任意一个常数都无所谓)
    FROM B
    WHERE B.id = A.id
)
```
- 等价于
```
for SELECT * FROM A
    for SELECT * FROM B
        WHERE B.id = A.id
```

- 使用
当A表的数据小于B表的数据时,用EXISTS优于IN



- 该语法可以理解为:
```
SELECT ...
FROM table
WHERE EXISTS( subquery )
```


- <font color=red>将主查询的数据,放到子查询中做条件验证,根据验证结果(TRUE / FALSE)来决定主查询的数据结果是否保留</font>


- ps:
    - EXISTS(subquery)只返回TRUE 或者 FALSE,因此子查询中的SELECT * 也可以是SELECT 1或者SELECT 'X',官方说法是会忽略SELECT清单,因此没有区别
    - EXISTS子查询实际执行过程可能也经过了优化而不是我们理解上的逐条对比
    - EXISTS子查询可以用条件表达式,其他子查询或者JOIN替代,何种最优需要具体具体分析

