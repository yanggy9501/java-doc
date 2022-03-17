# 注解

## 1，简介

>   什么是注解（Annotation）？注解是放在Java源码的类、方法、字段、参数前的一种特殊`“注释”`。
>
>   注释会被编译器直接忽略，注解则可以被编译器打包进入class文件，因此，注解是一种用作标注的“元数据”。

## 2，自定义注解

Java语言使用`@interface`语法来定义注解（`Annotation`），它的格式如下：

>   注解的参数类似无参数方法，可以用`default`设定一个默认值（强烈推荐）。

```java
public @interface Report {
    int type() default 0;
    String level() default "info";
    String value() default "";
}
```



### 2.1 元注解

>   修饰其他注解的注解，这些注解就称为元注解（meta annotation）。

#### 2.1.1 @Target

>   使用`@Target`可以定义`Annotation`能够被应用于源码的哪些位置。
>
>   -   类或接口：`ElementType.TYPE`；
>   -   字段：`ElementType.FIELD`；
>   -   方法：`ElementType.METHOD`；
>   -   构造方法：`ElementType.CONSTRUCTOR`；
>   -   方法参数：`ElementType.PARAMETER`。
>
>   `@Target`注解参数变为数组，如：{ ElementType.METHOD, ElementType.FIELD }



### 2.1.2 @Retention

`@Retention`定义了`Annotation`的生命周期

>   -   仅编译期：`RetentionPolicy.SOURCE`；
>   -   仅class文件：`RetentionPolicy.CLASS`；
>   -   运行期(我们自定义的都是这个)：`RetentionPolicy.RUNTIME`。



### 2.1.3 @Inherited

>   使用`@Inherited`定义子类是否可继承父类定义的`Annotation`。
>
>   `@Inherited`仅针对`@Target(ElementType.TYPE)`类型的`annotation`有效，并且仅针对`class`的继承，对`interface`的继承无效。

```java
@Inherited
@Target(ElementType.TYPE)
public @interface Report {
    int type() default 0;
    String level() default "info";
    String value() default "";
}
```

```java
@Report(type=1)
public class Person {
}
```

```java
// 子类默认也定义了该注解@Report(type=1)
public class Student extends Person {
}
```



## 3，处理注解

Java的注解本身对代码逻辑没有任何影响。根据`@Retention`的配置：

-   `SOURCE`类型的注解在编译期就被丢掉了；
-   `CLASS`类型的注解仅保存在class文件中，它们不会被加载进JVM；
-   `RUNTIME`类型的注解会被加载进JVM，并且在运行期可以被程序读取。

### 3.1 反射获取注解及其属性值

>   因为注解定义后也是一种`class`，所有的注解都继承自`java.lang.annotation.Annotation`，因此，读取注解，需要使用反射API。

#### 3.1.1 判断某个注解是否存在

判断某个注解是否存在于`Class`、`Field`、`Method`或`Constructor`：

-   `Class.isAnnotationPresent(Class)`
-   `Field.isAnnotationPresent(Class)`
-   `Method.isAnnotationPresent(Class)`
-   `Constructor.isAnnotationPresent(Class)`

**案例：**

```java
// 判断@Report是否存在于Person类:
Person.class.isAnnotationPresent(Report.class);
```

#### 3.1.2 反射API读取Annotation

-   `Class.getAnnotation(Class)`
-   `Field.getAnnotation(Class)`
-   `Method.getAnnotation(Class)`
-   `Constructor.getAnnotation(Class)`

**PS：** 参数Class是注解的字节码对象。

**案例：**

```java
// 获取Person定义的@Report注解:
Report report = Person.class.getAnnotation(Report.class);
int type = report.type();
String level = report.level();
```

通常使用：

```java
Class cls = Person.class;
if (cls.isAnnotationPresent(Report.class)) {
    Report report = cls.getAnnotation(Report.class);
    ...
}
// ***********************
Class cls = Person.class;
Report report = cls.getAnnotation(Report.class);
if (report != null) {
   ...
}
```

读取方法、字段和构造方法的`Annotation`和Class类似。但要读取方法参数的`Annotation`就比较麻烦一点，因为方法参数本身可以看成一个数组，而每个参数又可以定义多个注解，所以，一次获取方法参数的所有注解就必须用一个二维数组来表示。

```java
public void hello(@NotNull @Range(max=5) String name, @NotNull String prefix) {
}
```

```java
// 获取Method实例:
Method m = ...
// 获取所有参数的Annotation:
Annotation[][] annos = m.getParameterAnnotations();
// 第一个参数（索引为0）的所有Annotation:
Annotation[] annosOfName = annos[0];
for (Annotation anno : annosOfName) {
    if (anno instanceof Range) { // @Range注解
        Range r = (Range) anno;
    }
    if (anno instanceof NotNull) { // @NotNull注解
        NotNull n = (NotNull) anno;
    }
}
```

