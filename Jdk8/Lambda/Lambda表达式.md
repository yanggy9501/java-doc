# Lambda表达式

> Lambda 是一个匿名函数，我们可以把 Lambda 表达式理解为是一段可以传递的代码（其实就是一个对象，就是匿名内部类的进化版）

**从匿名内部类到 lambda 的转化**

* 匿名内部类

```java
Runnable r = new Runnable() {
    public void run() {
		// 匿名内部类代码块
        System.out.println("匿名内部类代码块")
    }
};
r.run();
```

* lambda 表达式

```java
Runnable r = () ->  System.out.println("lambda 表达式");
r.run();
```



## Lambda 表达式语法

> Lambda 表达式在Java 语言中引入了一个新的语法元素和操作符。这个操作符为 “->” ， 该操作符被称 为 Lambda 操作符或剪头操作符。它将 Lambda 分为 两个部分:

* 左侧：指定了 Lambda 表达式需要的所有参数 

* 右侧：指定了 Lambda 体，即 Lambda 表达式要执行 的功能

  ```
  (lambda表达式参数) -> {
  	// Lambda 表达式要执行 的功能
  }
  ```

**注：** Lambda 表达式本质就是一个匿名内部类的**对象**



**1，语法格式一**

> 无参数，无返回值 Runnable接口

```java
Ruunabel r = () ->  System.out.println("lambda 表达式");
```



**2,  语法格式二**

> 一个参数，无返回值 Consumer(T) 接口

```java
Consumer<String> c = (str) -> System.out.println(str);
```

**注：** 当lambda表达式只需要一个参数时，参数的小括号可以省略



**3，语法格式三**

> 两个参数，并且有返回值BinaryOperator接口，参数类型和返回值类型一样

```java
BinaryOperator<Long> bo = (x, y) -> x + y;
```

**注：** 当lambda表达式只有一条语句时，return和大括号都可以一起省略



# 类型推断

> Lambda 表达式中的参数类型都是由编译器推断得出的。Lambda 表达式中无需指定类型，程序依然可以编译，这是因为 javac 根据程序的上下文，在后台 推断出了参数的类型。

