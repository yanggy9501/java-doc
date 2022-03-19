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



### 1.1 范围查询

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

