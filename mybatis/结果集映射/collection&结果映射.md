# collection 

> collection : 一个对象有的另外一个List对象的结果映射，即1：n时的结果映射
>
> 场景：一个部门中有多个员工



## 1，collection & 联表查询

> 相当与子查询

```xml
<result id="deptMap" type="xxx.Dept">
	<id column="did" property="did"/>
    <result column="dept_name" property="deptName" />
    <!-- 
		property: Dept的属性名
		javaType: Java类型
		ofType: list集合的类型
		column: 字段名，一次sql查询的字段，用于联表查询传递给下一个SQL进行查询结果
		select：employees的结果来自哪个SQL，接受column属性值的参数
	-->
    <collection property="employees" javaType="list" ofType="xxx.Employee" column="eid" 	  select="xxx.EmployeeMapper.getEmployees">
        <id column="eid" property="eid"/>
    	<result column="e_name" property="employeeName" />
    </collection>
</result>
```



## 2，collection & 嵌套查询

> 相当连接查询, 这个时候不是通过二次查询进行映射而是通过字段进行映射

```xml
<result id="deptMap" type="xxx.Dept">
	<id column="did" property="did"/>
    <result column="dept_name" property="deptName" />
    <!-- 
		property: Dept的属性名
		javaType: Java类型
		ofType: list集合的类型
	-->
    <collection property="employees" javaType="list" ofType="xxx.Employee" >
        <id column="eid" property="eid"/>
    	<result column="e_name" property="employeeName" />
    </collection>
</result>
```

