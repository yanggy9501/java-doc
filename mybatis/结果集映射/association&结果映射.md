# association

> association ：一个对象之中包含另外一个对象的时候的，结果映射



# 1, association&级联查询

> association&级联查询, 就像SQL中的子查询

> 场景：一个员工属于一个部门

```java
public class Employee {
    private Integer id;
    private String name;
    
    // 部门
    private Dept dept;
}
```

### 一对一关系

> 一对一关系，一个对象中包含另外一个对象

**EmployeeMapper.xml**

```xml
<resultMap id="employeeMap" type="xxx.Employee">
	<id column="id" property="id"/>
    <result column="name" property="name" />
    <!-- 
		property: 属性名
		column: 字段名，即和<result column="name"/>的name一回事，用于传递参数值。第一次查询必须有					deptId的值。
		javaType：对应的数据类型，可以是别名
		select：该封装对象的数据有哪个sql语句生成的, 可以是其他mapper文件（即其他命名空间）的sql语句。				 同时携带deptId字段的值去查询。select的属性值要全包含命名空间如：									com.xxx.DeptMapper.getDept。getDept是sql的id
	-->
    <association property="dept" column="deptId" javaType="xxx.Dept" select="sql坐标" />
</resultMap>

<!-- association 丰富的功能, 可以像resultMap一样定义各个属性的映射 -->
<association property="dept" column="deptId" javaType="xxx.Dept" select="sql坐标">
	<id column="id" property="id"/>
    <result column="deptName" property="deptName" />
</association>	
```

**注：** 上述可以使用collection标签进行封装，尽管collection用于集合类型，单个对象亦可以看着特殊的集合

### 多对一关系

> 同上面，在多的一方使用association标签。如果想在 1 的一方保留关系，则需要 collection标签



#  2, association&查询嵌套 -- 更推荐

> 按结果嵌套处理，就像SQL中的联表查询，只发一条SQL语句，上面多条SQL语句

```xml
<resultMap id="employeeMap" type="xxx.Employee">
	<id column="id" property="id"/>
    <result column="name" property="name" />
    <!-- 
		property: 属性名
		javaType：对应的数据类型，可以是别名
	-->
    <association property="dept" column="deptId" javaType="xxx.Dept" select="sql坐标" />
</resultMap>
<!-- association 丰富的功能, 可以像resultMap一样定义各个属性的映射 -->
<association property="dept"  javaType="xxx.Dept">
	<id column="id" property="id"/>
    <result column="deptName" property="deptName" />
</association>	
```

**注：** 一次联表查询查询所有的结果，mybatis通过association的高级映射完成结果的封装。此时不需要第三方的sql查询

**注：**association可以无限的嵌套，一个城市属于某个省，某个省属于某个国家，某个国家属于某个洲...

