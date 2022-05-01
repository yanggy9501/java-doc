# 索引优化&实战

## 1，常见案例

**数据准备**

```sql
CREATE TABLE `employees` (
	`id` INT ( 11 ) NOT NULL AUTO_INCREMENT,
	`name` VARCHAR ( 24 ) NOT NULL DEFAULT '' COMMENT '姓名',
	`age` INT ( 11 ) NOT NULL DEFAULT '0' COMMENT '年龄',
	`position` VARCHAR ( 20 ) NOT NULL DEFAULT '' COMMENT '职位',
	`hire_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '入职时间',
	PRIMARY KEY ( `id` ),
	KEY `idx_name_age_position` ( `name`, `age`, `position` ) USING BTREE 
) ENGINE = INNODB AUTO_INCREMENT = 1 DEFAULT CHARSET = utf8 COMMENT = '员工记录表';

INSERT INTO employees ( NAME, age, position, hire_time )VALUES ('LiLei',22,'manager',NOW());
INSERT INTO employees ( NAME, age, position, hire_time ) VALUES('HanMeimei',23,'dev',NOW());
INSERT INTO employees ( NAME, age, position, hire_time ) VALUES('Lucy',23,'dev',NOW());

-- 插入一些示例数据 
DROP PROCEDURE IF EXISTS insert_emp;
delimiter;;
CREATE PROCEDURE insert_emp () BEGIN
	DECLARE
		i INT;
	SET i = 1;
	WHILE
			( i <= 100000 ) DO
			INSERT INTO employees ( NAME, age, position )
		VALUES
			( CONCAT( 'zhuge', i ), i, 'dev' );
		SET i = i + 1;
	END WHILE;
	
END;;
delimiter;
CALL insert_emp ();
```



### 1.1 常见优化

**1，索引范围查询可能不会走索引，确定的是范围后面的字段一定不走索引**

```sql
 EXPLAIN SELECT * FROM employees WHERE name > 'LiLei' AND age = 22 AND position ='manager';
```



>   仍然需要记住索引是具有顺序的数据结构。
>
>   如：
>
>   *   name > “LiLei”: 不会走索引，mysql会判断这样的数据太大还不如全表扫描
>   *   name = “LiLei”: 走索引
>   *   name < “LiLei”: 走索引，但是范围后面的不走索引，调换顺序mysql仍然会优化。



**2，制走索引**

>   使用关键字 force index(索引名)强制使用索引。
>
>   一般不推荐

```sql
EXPLAIN SELECT * FROM employees force index(idx_name_age_position) WHERE name > 'LiLei' AND age = 22 AND position ='manager';
```



**3，覆盖索引优化**

>   尽量使用索引覆盖，避免回表。
>
>   只返回索引所覆盖到的列，就可以避免回表，where中可以没有这一列。

```sql
EXPLAIN SELECT name,age,position FROM employees WHERE name < 'LiLei' AND age = 22;
```



**4, in和or可能会走索引**

>   in和or在表数据量比较大的情况会走索引，在表记录不多的情况下会选择全表扫描



**5，、like KK% 一般情况都会走索引**

>   因为其能够比较大小，相当于数字保留的高位，去掉低位不影响大小。

```sql
EXPLAIN SELECT * FROM employees_copy WHERE name like 'LiLei%' AND age = 22 AND position ='manager';
```

## 2，索引下推

>    like KK%其实就是用到了索引下推优化。
>
>   **什么是索引下推了？**
>
>   于辅助的联合索引(name,age,position)，正常情况按照最左前缀原则，SELECT * FROM employees WHERE name like 'LiLei%' AND age = 22 AND position ='manager' 这种情况只会走name字段索引，因为根据name字段过滤完，得到的索引行里的age和 position是无序的，无法很好的利用索引。
>
>    在MySQL5.6之前的版本，这个查询只能在联合索引里匹配到名字是 'LiLei' 开头的索引，然后拿这些索引对应的主键逐个回表，到主键索 引上找出相应的记录，再比对age和position这两个字段的值是否符合。
>
>   MySQL 5.6引入了索引下推优化，可以在索引遍历过程中，对索引中包含的所有字段先做判断，过滤掉不符合条件的记录之后再回表，可以有效的减少回表次数。

## 3，深入优化

### 3.1 Order by与Group by优化

**结论：**

1.   MySQL支持两种方式的排序filesort和index，Using index是指MySQL扫描索引本身完成排序。index 效率高，filesort效率低。
2.   **order by满足两种情况都会会使用Using index：**
     *    order by语句使用索引最左前列，如：order by name，age，position
     *    使用where子句与order by子句条件列组合满足索引最左前列，如：where name=‘name’order by age，position
3.   如果order by的条件不在索引列上，就会产生Using filesort。
4.   group by与order by很类似，其实质是`先排序后分组`，遵照索引创建顺序的最左前缀法则。对于group by的优化如果不需要排序的可以加上order by null禁止排序。

### 3.2 分页查询

>   在索引上完成排序分页操作，最后根据主键关联回原表查询所需要的其他列内容。
>
>   利用order by主键排序，limit主键是很容易的，然后根据主键返回数据。
>
>   其思路：关键是让排序时返回的字段尽可能少，所以可以让排序和分页操作先查出主键，然后根据主键查到对应的记录。

```sql
explain select * from tb_item t,(select id from tb_item order by id limit 2000000,10) a where t.id = a.id;
```



### 3.2 Join关联查询优化

mysql的表关联常见的两种算法

*   嵌套循环连接 Nested-Loop Join(NLJ) 算法
*   基于块的嵌套循环连接 Block Nested-Loop Join(BNL)算法

#### 3.2.1 嵌套循环连接 Nested-Loop Join(NLJ) 算法

```sql
 EXPLAIN select * from t1 inner join t2 on t1.a= t2.a;
```



>   核心：就是暴力算法，两次循环查找。
>
>   一次一行循环地从第一张表（称为驱动表）中读取行，在这行数据中取到关联字段，根据关联字段在另一张表（被驱动 表）里取出满足条件的行，然后取出两张表的结果合集。
>
>   **注意：**
>
>   *   inner join：优化器一般会优先选择小表做驱动表。所以使用 inner join 时，排在前面的表并不一定就是驱动表。
>   *   left join：左表是驱动表，右表是被驱动表
>   *   right join：右表时驱动表，左表是被驱动表
>
>   **sql流程：**
>
>   1.   从表 t2 中读取一行数据（如果t2表有查询过滤条件的，会从过滤结果里取出一行数据）
>
>   2.   从第 1 步的数据中，取出关联字段 a，到表 t1 中查找
>
>   3.   取出表 t1 中满足条件的行，跟 t2 中获取到的结果合并，作为结果返回给客户端
>
>   4.   重复上面 3 步
>
>        **扫描的磁盘总行数** = (表 t1 的数据总量) × (表 t2 的数据总量) 
>
>   **结论：**使用NLJ算法性能会比较低



#### 3.2.2 基于块的嵌套循环连接 Block Nested-Loop Join(BNL)算法

Extra 中 的Using join buffer (Block Nested Loop)说明该关联查询使用的是 BNL 算法。

```sql
EXPLAIN select * from t1 inner join t2 on t1.b= t2.b;
```



>   面sql的大致流程如下:
>
>   1.   把 t2 的所有数据放入到 join_buffer 中
>   2.   把表 t1 中每一行取出来，跟 join_buffer 中的数据做对比
>   3.   返回满足 join 条件的数据
>
>   整个过程对表 t1 和 t2 都做了一次全表扫描，因此
>
>   **扫描的磁盘总行数为** = (表 t1 的数据总量) + (表 t2 的数据总量) 。
>
>   **但是内存判断次数** =  (表 t1 的数据总量) × (表 t2 的数据总量)
>
>   ****

**如果 join_buffer放不下？**

则分段放，

比如 t2 表有1000行记录， join_buffer 一次只能放800行数据，那么执行过程就是先往 join_buffer 里放800行记录，然后从 t1 表里取数据跟 join_buffer 中数据对比得到部分结果，然后清空 join_buffer ，再放入 t2 表剩余200行记录，再 次从 t1 表里取数据跟 join_buffer 中数据对比。所以就多扫了一次 t1 表。



#### 对于关联sql的优化

*   **关联字段加索引**，让mysql做join操作时尽量选择NLJ算法 。
*   **小表驱动大表**，写多表连接sql时如果明确知道哪张表是小表可以用straight_join写法固定连接驱动方式，省去mysql优化器自己判断的时间。

### 3.3 count(*)查询优化

**注意：**有根据某个字段count不会统计字段为null值的数据行

>   **字段有索引：count(*)≈count(1)>count(字段)>count(主键 id)**
>
>   >   （字段有索引，count(字段)统计走二级索引，二 级索引存储数据比主键索引少，所以count(字段)>count(主键 id)）
>
>   **字段无索引：count(*)≈count(1)>count(主键 id)>count(字段)**
>
>   >   字段没有索引count(字段)统计走不了索引， count(主键 id)还可以走主键索引，所以count(主键 id)>count(字段)
>
>   count(\*) 是例外，mysql并不会把全部字段取出来，而是专门做了优化，不取值，按行累加，效率很高`，所以不需要用 count(列名)或count(常量)来替代 count(\*)`。

优化：

*   show table status：如果只需要知道表总行数的估计值可以用如下sql查询，性能很高
*   将总数维护到Redis里
*   增加数据库计数表：插入或删除表数据行的时候同时维护计数表，让他们在同一个事务里操作
