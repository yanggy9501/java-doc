# 后端数据校验

> 早期的网站，用户输入一个邮箱地址，需要将邮箱地址发送到服务端，服务端进行校验，校验成功后，给前端一个响应。
>
> 有了JavaScript后，校验工作可以放在前端去执行。那么为什么还需要服务端校验呢？ 因为前端传来的数据不可信。前端很容易获取到后端的接口，如果有人直接调用接口，就可能会出现非法数据，所以服务端也要数据校验。
>
> 总的来说：
>
> - [ ] 前端校验：主要是提高用户体验
> - [ ] 后端校验：主要是保证数据安全可靠
>
> 校验参数基本上是一个体力活，而且冗余代码繁多，也影响代码的可读性，我们需要一个比较优雅的方式来解决这个问题。Hibernate Validator 框架刚好解决了这个问题，可以以很优雅的方式实现参数的校验，让业务代码和校验逻辑分开,不再编写重复的校验逻辑。
>
> hibernate-validator优势：
>
> - [ ] 验证逻辑与业务逻辑之间进行了分离，降低了程序耦合度
> - [ ] 统一且规范的验证方式，无需你再次编写重复的验证代码
> - [ ] 你将更专注于你的业务，将这些繁琐的事情统统丢在一边

## 1. 引入依赖

```xml
<!-- 后端数据校验 -->
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>2.0.1.Final</version>
</dependency>
<!-- hibernate提供了额外实现如URL校验 -->
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.1.6.Final</version>
</dependency>
```

## 2. validator使用

### 2.1 常用注解

hibernate-validator提供的用于校验的注解如下：

| 注解                      | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| @AssertTrue               | 用于boolean字段，该字段只能为true否则校验不通过              |
| @AssertFalse              | 用于boolean字段，该字段只能为false否则校验不通过             |
| @CreditCardNumber         | 对信用卡号进行一个大致的验证                                 |
| @DecimalMax               | 只能小于或等于该值                                           |
| @DecimalMin               | 只能大于或等于该值                                           |
| @Email                    | 检查是否是一个有效的email地址                                |
| @Future                   | 检查该字段的日期是否是属于将来的日期                         |
| @Length(min=,max=)        | 检查所属的字段的长度是否在min和max之间,只能用于字符串        |
| @Max                      | 该字段的值只能小于或等于该值                                 |
| @Min                      | 该字段的值只能大于或等于该值                                 |
| @NotNull                  | 不能为null                                                   |
| @NotBlank                 | 不能为空，检查时会将两端空格忽略在判断如：“   ”              |
| @NotEmpty                 | 不能为空，这里的空是指空字符串                               |
| @Pattern(regex=)          | 被注释的元素必须符合指定的正则表达式                         |
| @URL(protocol=,host,port) | 检查是否是一个有效的URL，如果提供了protocol，host等，则该URL还需满足提供的条件 |

注解的属性：

* **message**: 校验出错的错误信息，exception.getMessage()获取信息

* **groups**：分组校验；相同分组统一校验（根据不同情况进行分组，校验）

* **regexp**：正则表达式

  ```java
  @NotNull(message = "修改时必须指定品牌id", groups = {UpdateGroup.class})
  @Null(message = "新增不指定id", groups = {AddGroup.class})
  @TableId
  private Long brandId;
  ```

  

### 2.2 入门案例

#### 

##### 第一步：引入依赖

##### 第二部：创建实体类

```java
import lombok.Data;
import org.hibernate.validator.constraints.Length;
import javax.validation.constraints.*;

@Data
public class User {
    @NotNull(message = "用户id不能为空")
    private Integer id;

    @NotEmpty(message = "用户名不能为空")
    @Length(max = 50, message = "用户名长度不能超过50")
    private String username;

    @Max(value = 80,message = "年龄最大为80")
    @Min(value = 18,message = "年龄最小为18")
    private int age;

    @Pattern(regexp = "[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\\.[a-zA-Z0-9_-]+)+$",
             message = "邮箱格式不正确")
    private String email;
}
```



##### 第三步：创建UserController，并开启据校验

开启数据校验：@Validated 或 @Valid 用在参数上两种等价。（@Validatedspring提供的，@Valid是规范）

* 简单类型校验

  > 对于简单类型需要在Controller类上加@Validated开启校验功能，简单类型的校验才会生效，对于对象类型则没必要加，简单类型使用包装类

* 对象类型校验

```java
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import javax.validation.constraints.NotBlank;

@RestController
@RequestMapping("/user")
@Validated //开启校验功能（有简单类型的数据校验时需要该注解）
public class UserController {
    // 简单数据类型校验
    @RequestMapping("/delete")
    public String delete(@NotBlank(message = "id不能为空") String id){
        System.out.println("delete..." + id);
        return "OK";
    }

    // 对象属性校验
    @RequestMapping("/save")
    public String save(@Validated User user){
        System.out.println("save..." + user);
        return "OK";
    }
}
```



> 在实体类上只加了校验注解是没有任何作用的，需要开启校验功能在给controller提交数据时告诉springMVC开启校验功能。

**@Valid 或者 @Validated**

* 位置：参数前面
* 作用：告诉spring这个参数开启了校验功能
* 效果：校验错误以后会有默认的响应
* 校验结果：给校验的bean后紧跟一个BindingResult，就可以获取到校验的结果

错误代码400：后端数据校验出错，默认会返回前端错误信息如：错误代码，错误字段，错误message等

```java
/**
*@Valid ：开启JST303数据校验
*BindingResult：绑定校验以后的结果，封装在BindingResult（可查看是否有错）
*/
@PostMapping("/register")
public String register(@Valid UserRegisterVo vo, BindingResult result) {
    if (result.hasErrors()) {

    }
    return "redirect:/login.html";
}
```

**BindingResult**

```java
public String register(@Valid UserRegisterVo vo, BindingResult result) {
   // 1，获取校验的错误结果
    result.getFieldErrors().forEach(item -> { 
        //Field 获取到错误提示
        String msg = item.getDafualtMeassage();
        //获取错误的属性的名字
        item.filed = item.getField();
    });
    return "null";
}
```



##### 第四步：创建启动类

> 可以配合knife4j测试



## 3. 自定义错误消息提示

> 自动错误消息提示可能不符合个人要求，默认是javax.validation提供的默认消息properties文件

**自定义消息提示**：使用校验注解的 message属性

* 添加属性：message=“自定义错误消息提示”

* 配置文件：或者自己定义一个自定义校验文件，重写这些提示信息

```java
@Null(message = "自定义错误消息提示")
```



## 4. 自定义全局异常处理

> 为了能够在页面友好的显示数据校验结果，可以通过全局异常处理来解决，创建全局异常处理类。
>
> @RestControllerAdvice属性：
>
> basePackages：捕捉异常的包路径，只有该路径有异常才会被处理
>
> ```java
> @RestControllerAdvice(basePackages = {"com.ilike.xxx.yyy.controller"})
> 
> ```
>
> 
>
> annotations：只要类上有这些注解才会被处理
>
> ```java
> @ControllerAdvice(annotations = {RestController.class, Controller.class})
> ```
>
> 

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Controller;
import org.springframework.validation.BindException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;
import javax.servlet.http.HttpServletRequest;
import javax.validation.ConstraintViolation;
import javax.validation.ConstraintViolationException;
import java.util.Set;

/**
 * 全局异常处理
 */
@ControllerAdvice(annotations = {RestController.class, Controller.class})
@ResponseBody
public class ExceptionConfiguration {
   // 数据校验异常，一般是这两种
@ExceptionHandler({ConstraintViolationException.class,BindException.class})
    public String validateException(Exception ex, HttpServletRequest request) {
        ex.printStackTrace();
        String msg = null;
        if(ex instanceof ConstraintViolationException){
            ConstraintViolationException constraintViolationException = 
                (ConstraintViolationException)ex;
            Set<ConstraintViolation<?>> violations = 
                constraintViolationException.getConstraintViolations();
            ConstraintViolation<?> next = violations.iterator().next();
            msg = next.getMessage();
        }else if(ex instanceof BindException){
            BindException bindException = (BindException)ex;
            msg = bindException.getBindingResult().getFieldError().getDefaultMessage();
        }
        return msg;
    }
}
```



## 5.分组校验

> ​	对数据的校验是分情况的，例如：新增时id可以为null，而修改数据是需要提供id值

目的：对不同情况进行分组校验

* 使用校验注解的 group属性：指定校验规则：接受注解的字节码

### 1、自定义注解 - 标识不同分组

```java
public interface UpdateGroup {
}

```

### 2、校验注解标注 - 添加分组注解标识

```java
@NotNull(message = "修改时必须指定品牌id", groups = {UpdateGroup.class})
```



### 3、开启分组校验，@Validated的group属性

使用@Validated，而@Valid没有提供任何分组校验属性

* 只校验指定分组的属性
* @Validated({AddGroup.class})
 *         默认没有指定分组的校验注解@NotBlank，在分组校验情况@Validated({AddGroup.class})下不生效，**只会在@Validated生效；**

```java
@PutMapping("/update")
public R update(@Validated({UpdateGroup.class}) @RequestBody BrandEntity brand){
    brandService.updateDetail(brand);

    return R.ok();
}
```



## 6.自定义校验

1）、编写一个自定义的校验注解

2）、编写一个自定义的校验器

3）、关联自定义的校验器和自定义的校验注解



### 6.1 编写一个自定义的校验注解

> 参考@NotNull

**注解修饰**

​		@Constraint

* 作用: 指定校验器，校验注解和校验器产生关联
* 属性：
  * ListValue：数组，指定一个或者多个校验器。

```java 
/**
 * 自定义校验注解
 */
@Documented
// 指定校验器, 可以指定多个。自动选择最佳一个（这个注解使用哪个校验器）
@Constraint(validatedBy = {ListValueConstraintValidator.class})
// 作用范围
@Target({ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE, ElementType.CONSTRUCTOR, ElementType.PARAMETER, ElementType.TYPE_USE})
@Retention(RetentionPolicy.RUNTIME)// 声明周期，何时起作用
public @interface ListValue {

    // 到那个配置文件中取 message。需要一个properties文件
    String message() default "{com.ilike.common.core.valid.validanno.ListValue.message}";
	// 分组
    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
	// 用于指定特定值的：可没有
    int[] vals() default 0;
}

```

### 6.2 properties配置默认message提示消息

> 文件名叫：ValidationMesssages.properties

默认文件名就是上面这个

```properties
com.ilike.common.core.valid.validanno.ListValue.message=默认错误提示信息
```



### 6.3 编写一个自定义的校验器

> 自定义一个类实现校验器的几口ConstraintValidator

```java
/**
 * 自定义校验器，实现ConstraintValidator 接口
 * 第一泛型：校验那个注解
 * 第二个泛型：校验什么类型
 */
public class ListValueConstraintValidator implements ConstraintValidator<ListValue, Integer> {

    ArrayList<Integer> set = new ArrayList<>();

    /**
     * 初始化方法
     * ListValue: 自定义的注解
     */
    @Override
    public void initialize(ListValue constraintAnnotation) {
        // 从注解上获取值（返回值有自定义注解的属性值决定）
        int[] vals = constraintAnnotation.vals();
        for(int i = 0; i < vals.length; i++){
            set.add(vals[i]);
        }
    }
	
    /**
    *校验实现：判断是否校验成功
    *val：需要校验的值
    *constraintValidatorContext：校验上下文
    */
    @Override
    public boolean isValid(Integer val, ConstraintValidatorContext constraintValidatorContext) {
        return set.contains(val);
    }
}
```



### 6.4 自定义校验使用
