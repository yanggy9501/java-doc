# mybatis 特殊结果集映射

## 1，包含基础类型机会的属性处理

>   在使用MyBatis查询数据库时，经常会有一对多的情况，那么在一对多的情况时，如果是一个`Collection<String>`或者`Collection<Integer>` 类型，那么我们的`ResultMap`该如何定义？

**期望这样的结果集：**

```json
{"id":1,"names":["Answer","AI","AAL"],"roles":["Admin","Manager","Coder"]}
{"id":2,"names":["Iris","Ellis","Monta"],"roles":["CustomerService"]}
```

### 方式一

>   方法很简单，这时候我们就需要使用到`构造函数注入`了，通过Integer和String的构造函数注入，具体的字段名称自己对好入座即可。
>
>   由于该方式是使用构造方法注入，若对象没有sql对应列的构造方法就会报错。
>
>   **案例：将相同age，adress的name合并到names中**

**mapper.xml**

```xml
<resultMap type="com.yanggy.demo.entity.UserVo" id="UserVoMap">
        <result property="age" column="age"/>
        <result property="address" column="address"/>
        <collection property="names" ofType="java.lang.String">
            <constructor>
                <!-- 对号入座数据库column名称即可 -->
                <arg column="names"/>
            </constructor>
        </collection>
    </resultMap>

    <select id="selectUserVo" resultMap="UserVoMap">
        select * from demo_user where  uuid = #{uuid}
    </select>
```

[UserVo( names=[kato, asu, yan, tmp], age=21, address=日本东京), UserVo( names=[alice], age=19, address=日本东京)]

**dto**

```java
@Data
@Accessors(chain = true)
public class UserVo {
    private List<String> names;

    private int age;

    private String address;

    private String uuid;
}
```



**sql**

```sql
CREATE TABLE `demo_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `address` varchar(255) DEFAULT NULL,
  `email` varchar(255) DEFAULT NULL,
  `uuid` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8mb4;
```



### 方式二：resultMap映射

```xml
<resultMap type="com.yanggy.demo.entity.UserVo" id="UserVoMap">
    <result property="age" column="age"/>
    <result property="address" column="address"/>
    <collection property="names" resultMap="mapString1" />
</resultMap>

<resultMap id="mapString1" type="string">
    <!-- mysql列名 -->
    <result column="name" />
</resultMap>

<select id="selectUserVo" resultMap="UserVoMap">
    select * from demo_user where  uuid = #{uuid}
</select>
```

[UserVo( names=[kato, asu, yan, tmp], age=21, address=日本东京), UserVo( names=[alice], age=19, address=日本东京)]



### 方式三：联表查询

>   原理就是在发一次sql查询。
>
>   不推荐这里不写

## 2，对象包含对象的属性处理

>   一个对象之中包含另外一个对象的时候的，xml结果映射该如何写呢？
>
>   假设场景：一个员工属于一个部门

```java
public class Employee {
    private Integer id;
    private String name;
    // 部门
    private Dept dept;
}
```

### 方式一：查询嵌套

>   按结果嵌套处理，就像SQL中的联表查询。

```xml
<resultMap id="employeeMap" type="xxx.Employee">
	<id column="id" property="id"/>
    <result column="name" property="name" />
    <!-- 
		property: 属性名
		javaType：对应的数据类型或别名
	-->
    <association property="dept" javaType="xxx.Dept">
        <!-- 没有id，无需该标签 -->
        <id column="id" property="id"/>
        <result column="deptName" property="deptName" />
    </association>
</resultMap>

<association property="dept"  javaType="xxx.Dept">
	<id column="id" property="id"/>
    <result column="deptName" property="deptName" />
</association>	
```



### 方式二：级联查询

>   需要发送另外一个sql查询，用于封装属性对象的结果，由于多发送一条sql，所以不推荐。