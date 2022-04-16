# foreach标签



mybatis 可以通过@Param注解来为一个普通数据类型，对象，数组，集合取一个名字。通常情况下在接口中传递多个参或者传递集合会使用@Param注解指定参数名以便宅mapper.xml中获取指定参数

```xml
<foreach collection="collectionName" item="item" open="(" close=")" separato=",">
    
</foreach>
```

**foreach标签属性解读**

> collection：参数名称，根据Mapper接口的参数名确定，也可以**使用@Param注解指定参数名**。集合默认list，数组默认array
>
> item：参数调用名称，通过此属性来获取集合单项的值
>
> open：相当于prefix，即在循环前添加前缀
>
> close：相当于suffix，即在循环后添加后缀
>
> index：索引、下标
>
> separator：分隔符，每次循环完成后添加此分隔符
>
> **注意：**这些标签属性都是可选的，不一定都需要设置所有属性



**插入多条数据**

**ps**：使用foreach插入数据时，一定要保证集合不为空，否则会报错

```xml
<insert>
	INSERT INTO `user`(username,age,password)
    <foreach collection="userList" item="user" separato=",">
    	(
        	#{user.username},#{user.age},#{user.password}
        )
	</foreach>
</insert>
```

