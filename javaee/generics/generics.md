# Java泛型

## 1，泛型简介

### 1.1 泛型基本概念

>   泛型的本质就是“数据类型的参数化”，处理的数据类型不是固定的，而是作为参数传入，也因此泛型只能是引用类型的占位符。我们可以把“泛型”理解为数据类型的一个占位符，即告诉编译器，在调用泛型时必须传入实际类型。
>
>   使用位置：
>
>   *   类
>   *   接口
>   *   方法
>
>   分别被称为：
>
>   *   泛型类
>   *   泛型接口
>   *   泛型方法

### 1.2 泛型优势

>   在不使用泛型的情况下，我们可以使用Object类型来实现任意类型的参数，但是往往需要强制类型转换。可能引起类型转换错误，并且还是在运行期才能被发现。
>
>   使用泛型可以在编译器发现错误。好处：
>
>   *   不用强制类型转换
>   *   只要编译器没有警告，运行期就不会出现类型转换异常

### 1.3 类型擦除

>   编码是采用泛型的类型参数，编译器在编译时去掉，这称为“类型擦除”。泛型主要用于编译阶段，class文件不包含泛型的信息。
>
>   类型参数在编译后会被替换成Object，运行时虚拟机并不知道泛型。

## 2，泛型的使用

### 2.1 定义泛型

泛型字符可以时任何的标识符，一般采用：

| 泛型标记 | 对应单词 | 说明                                         |
| -------- | -------- | -------------------------------------------- |
| E        | element  | 在容器中使用，表示容器的元素                 |
| T        | type     | 表示普通的Java类型                           |
| K        | key      | 表示键，如：map中的key                       |
| V        | value    | 表示值，如：map中value                       |
| N        | number   | 表示数值类型                                 |
| ？       |          | 表示不确定的Java类型，是容器元素或普通类型等 |

### 2.2 泛型类

>   泛型类就是把泛型定义在类上，用户在使用该类的时候，才把类型明确下来，泛型类在类的任何地方都能使用这个泛型。
>
>   泛型类的具体使用就是在类名称后添加一个或多个类型参数声明，如：<T>, <T, K, V>

#### 2.2.1 语法结构

```java
public class 类名<泛型标识符号> {
}
public class MyClass<T, K, V> {
}
```

**示例：**

```java
public class MyGeneric<T> {
    private T tField;
    public T getTField() {
        return this.tField;
    }
    public void setTField(T tField) {
        this.tField = tField;
    }
}
```

**只有在使用的时候才把类型确定下来，不指定就是Object**

>   如果我们指定泛型，那么泛型就会被确定下来，使用到泛型方法的参数或返回值都是明确的类型。

```java
public static void main(Stirng[] args) {
    // 指定类泛型，那么使用到泛型的地方的类型就确定下来了,就不用类型转换了
    MyGneric<String> obj1 = new Generic<>();
    obj1.setTField("test");
    String s = obj1.getTField();
    
    MyGneric obj2 = new Generic<>();
    obj.setTField(new Object());
    Object s = obj.getTField();
}
```

### 2.3 泛型接口

>   泛型接口和泛型类的声明方式一致，泛型接口的具体类型需要在实现类中进行声明。

#### 语法结构

```java
public interface 类名<泛型标识符号> {
}
```

**示例：**

由于实现类实现接口，在实现类中泛型T不在是泛型了而是具体的类型

```java
public interface Igeneric<T> {
    T getName(T name);
}
```

```java
public class IgenericImpl implements Igeneric<String> {
    String getName(String name){
        return name;
    }
}
```

```java
public static void main(Stirng[] args) {
    IgenericImpl obj1 = new IgenericImpl();
    String name1 = obj1.getName("name");
    
    // 如果由接口来接，返回是Object
    Igeneric obj2 = new IgenericImpl();
    Object name2 = obj2.getName("name");
    
     // 或者指定接口的泛型类型
    Igeneric<Stirng> obj3 = new IgenericImpl();
    String name3 = obj3.getName("name");
}
```

### 2.4 泛型方法

>   泛型类在中所定义的泛型，在方法中也可以使用。但是，我们经常需要仅仅在某个方法上使用泛型，这个时候可以使用泛型方法。
>
>   泛型方法是指将方法的参数类型定义成泛型，以便在调用时候接收不同的类型的参数。类型可以由多个，用逗号隔开，如：<K, V>。
>
>   定义时，类型参数一般放在返回值前面。
>
>   调用不需要指定类型，编译器自己根据上下文推断出类型。

#### 2.4.1 泛型实例方法

##### 语法结构

```java
public <泛型符号> void getName(泛型符号 name){
}
----------------------------------------
public <泛型符号> 泛型符号 getName(泛型符号 name){
}
```

**示例：**

```java
public class MethodGeneric {
    public <T> void getName(T name) {
        System.out.println(name);
    }
    
    public <T> T getName(T name) {
        return name;
    }
}
```

#### 2.4.2 泛型静态方法 

>   静态方法中使用泛型时有一种情况需要注意，那就是静态方法无法访问类上定义的泛型；如果静态方法操作的引用数据类型不确定的使用，必须要将泛型定义在静态方法上。

##### 语法结构

```java
public static <泛型符号> void getName(泛型符号 name){
}
----------------------------------------
public static <泛型符号> 泛型符号 getName(泛型符号 name){
}
```

#### 2.4.3 泛型方法&可变参数

>   在泛型方法中，泛型也可以定义可变参数。

##### 语法格式

```java
public <泛型符号> void getName(泛型符号 ...args){
}

```

### 2.5 泛型通配符和上下限定

### 2.5.1 无界通配符

>   ？表示泛型类型通配符，用于代替具体的类型。它只能在“<>”中使用，可以解决具体类型不确定的问题。
>
>   一般使用是存在泛型类的作为参数的地方。

##### 语法结构

```java
public void showFlag<Gerneric<?> generic){
    
}
```

因为参数Gerneric的泛型是什么类型，在这个方法中是不确定所以使用？，若明确是某个确定类型则可以使用：

```java
public void showFlag<Gerneric<String> generic){
    
}
```

#### 2.5.2 通配符的上限定

>   上限定表示`通配符的类型`是T类型以及T类的子类或者T接口以及T接口的子接口。即通配符有上界，只能其他类的“子类”。
>
>   如：`Generic<? extends Number> generic` 理解为泛型通配符extends某个类，？只能是其子类，方向上有界。
>
>   **ps**：同样适用其他泛型类

##### 语法案例

```java
public void showFlag(Generic<? extends Number> generic) {
    
}
```

#### 2.5.3 通配符的下限定

>   与上限定相反，通配符的下限定表示泛型的通配符是类型以及T类的父类或者T接口以及T接口的父接口。即通配符所表示的类有下届，只能是其他类的“父类”。
>
>   **ps：**不适用其他泛型类
>
>   ****

##### 语法案例

```java
public void showFlag(Generic<? super Number> generic) {
   
}
```

