# 1，Optional 类

> Optional 是jdk1.8新增的类，主要是为了解决空指针异常 NullPointerException

**1，传统null的处理**

```java
User user = getUserById(id);
if (user != null) {
    String username = user.getUsername();
    System.out.println("Username is: " + username); // 使用 username
}
```

**2，optional处理**

```java

```

> 

# 2，Optional 简介

> Optional 是一个没有子类的工具类，Optional 是一个可以为null的容器对象，它的作用主要是为了避免null检查，放在NullPointerException



# 3，Optional 的使用

**3.1 创建Optional 对象**

```java
public testCreateOptional() {
    // 创建方式1
    Optional<String> op1 = Optional.of("string");
   // 创建方式2
    Optional<Object> op3 = Optional.of("string");
    Optional<Object> op4 = Optional.ofNullable(null);
    
    // 创建方式3
    Opitonal<Object> op5 = Optional.empty();
}
```

**注意：**

*  .of（）方法不支持传如null值

* .ofNullable() 支持传入null值
* .empty() 可以创建一个空的Optional对象



**3.2 Optional 常用方法**

```java
public testCreateOptional() {
    // 创建Optional
    Optional<String> op1 = Optional.of("username");
    Opitonal<Object> op2 = Optional.empty();
    
    // 获取Optional中的值
    String name1 = opt1.get();
    System.out.println("用户名：" + name1);
    
    /* 异常调用
    String name2 = opt2.get();
    System.out.println("用户名：" + name2)
    */
}
```

**注意：**

* .get() 方法：

  * 如果Optional有值直接返回，否则报错NoSuchElementException异常
  * .get（）方法通常和isPresent（）方法一起使用

* .isPresent（）方法：

  * 判断是否包含值，包含返回true，否则返回false

* .orElse(T t) 方法：

  * 如果调用对象包含值，就返回该值，否则返回参数值

* .orElseGet(供给型接口)：

  * 如果调用对象包含值，就返回该值，否则返回供给型接口返回的值

* .ifPresent(消费型接口 Consumer) 方法：

  * 如果optional有值做什么

    ```java
    op.ifPresent(s -> System.out::println)
    ```

* .map(函数型接口) 方法：

  * lambda表达式作用到这个容器里面的对象，最好返回一个Optical，和stream流的map一致

* .filter()

* 



我们假设 `getUserById` 已经是个客观存在的不能改变的方法，那么利用 `isPresent` 和 `get` 两个方法，我们现在能写出下面的代码：

```java
Optional<User> user = Optional.ofNullable(getUserById(id));
if (user.isPresent()) {
    String username = user.get().getUsername();
    System.out.println("Username is: " + username); // 使用 username
}
```

> 好像看着代码是优美了点 —— 但是事实上这与之前判断 `null` 值的代码没有本质的区别，反而用 `Optional` 去封装 *value*，增加了代码量。所以我们来看看 `Optional` 还提供了哪些方法，让我们更好的（以正确的姿势）使用 `Optional`

如果 `Optional` 中有值，则对该值调用 `consumer.accept`，否则什么也不做。
所以对于上面的例子，我们可以修改为：

```java
User user = Optional
        .ofNullable(getUserById(id))
        .orElse(new User(0, "Unknown"));
        
System.out.println("Username is: " + user.getUsername());
```

