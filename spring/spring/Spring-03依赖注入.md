# Spring依赖注入

>   spring的依赖注入首先分为：
>
>   *   手动注入
>       *   set方法注入
>       *   构造方法注入
>   *   自动注入

## 1，手动注入

>   手动注入其实就是xml中配置注入的一种，因为是程序员手动给某个属性指定值的。
>
>   底层是：`set方法注入`和`构造方法注入`

### 1.1 Set方法注入

>   指定属性的值

```xml
<bean name="userService" class="com.xxx.service.UserService">
	<property name="orderService" ref="orderService"/>
</bean>
```

### 1.2 构造方法注入

>   指定构造方法

```java
<bean name="userService" class="com.xxx.service.UserService">
	<constructor‐arg index="0" ref="orderService"/>
</bean>
```



## 2，自动注入

>   自动注入时程序员不需要为属性指定某个值，spring自己注入。
>
>   自动注入分为：
>
>   *   `XML的autowire自动注入`
>   *   `@Autowired注解的自动注入`



### 2.1 xml的自动注入

>   在XML中，我们可以在定义一个Bean时去指定这个Bean的自动注入模式，有：
>
>   1.   byType 
>   2.   byName 
>   3.   constructor
>   4.   default（byName）
>   5.   no （表示关闭autowire）

```xml
<bean id="userService" class="com.xx.service.UserService" autowire="byType"/>
```

​		Spring会自动的给userService中所有的属性自动赋值（不需要这个属性上有 @Autowired注解，但需要这个属性<font color="red">有对应的set方法</font>）

​		在填充属性时，Spring会去解析当前类，把当前类的所有方法都解析出来， Spring会去解析每个方法得到对应的PropertyDescriptor对象。

>   1.   name：这个name并不是方法的名字，而是拿方法名字进过处理后的名字
>        1.    如果方法名字以“get”开头，比如“getXXX”,那么name=XXX 
>        2.    如果方法名字以“is”开头，比如“isXXX”,那么name=XXX 
>        3.    如果方法名字以“set”开头，比如“setXXX”,那么name=XXX 
>   2.   readMethodRef：表示get方法的Method对象的引用 (即Method对象)
>   3.   readMethodName：表示get方法的名字 (即方法名)
>   4.   writeMethodRef：表示set方法的Method对象的引用
>   5.   writeMethodName：表示set方法的名字
>   6.   propertyTypeRef：如果有get方法那么对应的就是返回值的类型，如果是set方法那么对应的就是set方法中唯一参数的类型
>
>   **注意：**
>
>   *   <font color="red">get方法的定义是： 方法参数个数为0个</font>
>
>   *   <font color="red">set方法的定义是：方法参数个数为1个</font>

#### 2.1.1 byName属性填充流程

>   spring是通过get和set方法确定该对象的属性的，若某个属性没有get和set方法，那么该属性是不会被注入值的。

1.    找到所有**set方法**所对应的XXX部分的名字（确定属性名）
2.   根据XXX部分的名字去获取bean（根据属性名即beanName去spring中找bean）
3.   注入

#### 2.1.2 byType属性填充流程

>   1.   获取到**set方法**中的唯一参数的参数类型，并且根据该类型去容器中获取bean。
>   2.   如果找到多个，会报错。



#### 2.1.3 constructor构造方法注入

>   spring利用构造方法的**参数**信息从Spring容器中去找bean，找到bean之后作为参数传给构造方法，从而实例化bean对象，并完成属性赋值。
>
>   这里存在**推断构造方法**
>
>   构造方法注入 = byType+byName



**注意：**`注解方式是xml的替代方式，但是提供了更加细粒度的控制。@Autowired注解是直接写在某个属性、某个set方法、某个构造方法上。但是@Autowired无法区分byType和byName，@Autowired是先byType后byName`



### 2.2 @Autowired自动注入

>   @Autowired注解，是byType和byName的结合。@Autowired提供更加细粒度的控制。
>
>   因为@Autowired注解可以写在： 
>
>   1.   属性上：先根据属性类型去找Bean，如果找到多个再根据属性名确定一个 
>   2.   构造方法上：先根据方法参数类型去找Bean，如果找到多个再根据参数名确定一个
>   3.   set方法上：先根据方法参数类型去找Bean，如果找到多个再根据参数名确定一个
>
>   底层用到：属性注入，set注入，构造方法注入。

## 3，注入点

>   给bean的属性进行属性赋值，那么需要找到bean的注入点。
>
>   Spring会利用AutowiredAnnotationBeanPostProcessor.postProcessMergedBeanDefinition()找出注入点并缓存。

#### 3.1 找注入点流程

1. `遍历当前类的所有的属性字段Field`
2. 查看字段上是否存在@Autowired、@Value、@Inject中的其中任意一个，存在则认为该字段是一个注入点
3. 如果字段是static的，则不进行注入
4. 获取@Autowired中的required属性的值
5. 将字段信息构造成一个AutowiredFieldElement对象，作为一个注入点对象添加到currElements集合中。
6. `遍历当前类的所有方法Method`
7. 判断当前Method是否是桥接方法，如果是找到原方法
8. 查看方法上是否存在@Autowired、@Value、@Inject中的其中任意一个，存在则认为该方法是一个注入点
9. 如果方法是static的，则不进行注入
10. 获取@Autowired中的required属性的值
11. 将方法信息构造成一个AutowiredMethodElement对象，作为一个注入点对象添加到currElements集合中。
12. 遍历完当前类的字段和方法后，将`遍历父类`的，直到没有父类。
13. 最后将currElements集合封装成一个InjectionMetadata对象，作为当前Bean对于的注入点集合对象，并缓存。



### 3.2 static的字段或方法不支持注入

```java
@Component
@Scope("prototype")
public class OrderService {
}
```

```java
@Component
@Scope("prototype")
public class UserService {
@Autowired
private static OrderService orderService;
    public void test() {
    	System.out.println("test123");
    }
}
```

UserService和OrderService都是原型Bean，假设Spring支持static字段进行自动注 入，那么现在调用两次。userService1的orderService值是什么？一旦userService2 创建好了之后，static orderService字段的值就发生了修改了，从而 出现bug。

```java
UserService userService1 = context.getBean("userService");
UserService userService2 = context.getBean("userService");
```



### 3.3 注入点进行注入

>   spring在AutowiredAnnotationBeanPostProcessor的**postProcessProperties()**方法中，会遍历所找到的注入点依次进行注入。



#### 3.3.1 字段注入



#### 3.3.2 set方法注入
