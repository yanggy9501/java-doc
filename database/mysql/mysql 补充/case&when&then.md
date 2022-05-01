# case 条件表达式

> SQL标志中定义了case 表达式，它可以基于一个条件列表返回不同的结果，类似于其他编程语言的条件语句（if-then-else 或者 switch）。

**简单的条件表达式**

```sql
CASE expression表达式
	WHEN expression_1 THEN result_1
	WHEN expression_2 THEN result_2
	...
	[ELSE defualt_result]
END
-- 当case表达式等于when中的表达式就返回对应的结果, 这个形式的表达式是相等关系,根据case表达式取when中找

SELECT 
	e.first_name,
	e.last_name,
	CASE e.job 
		WHEN 'AD_VP' THEN 'Vice President'
		WHEN 'IT_PROG' THEN 'Programer'
		ELSE ''
    END AS 'job description'
FROM employee e;
```

```sql
CASE SCORE WHEN 'A' THEN '优' ELSE '不及格' END
CASE SCORE WHEN 'B' THEN '良' ELSE '不及格' END
CASE SCORE WHEN 'C' THEN '中' ELSE '不及格' END

-- 等同于
CASE WHEN SCORE = 'A' THEN '优'
     WHEN SCORE = 'B' THEN '良'
     WHEN SCORE = 'C' THEN '中' ELSE '不及格' END
```



**推荐的使用格式：**

```sql
CASE WHEN condition THEN result
 
[WHEN...THEN...]
 
ELSE result
 
END
```

> condition是一个返回布尔类型的表达式，如果表达式返回true，则整个函数返回相应result的值，如果表达式皆为false，则返回ElSE后result的值，如果省略了ELSE子句，则返回NULL