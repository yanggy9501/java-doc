* **@JsonProperty("name")**

> 修改对象属性序列化后的属性值（键），并且应用的序列化和反序列化工程

```java
@JsonProperty("name")
private String menuName;
```





* **@JsonIgnore**

> 位置：属性，方法上
>
> 作用：序列化和反序列化忽视改属性，方法

```java
@JsonIgnore
private String[] ids;
```



* **@JsonInclude(JsonInclude.Include.NON_EMPTY)**

> 位置：类，属性上
>
> 作用：属性为null时，不序列化给属性

