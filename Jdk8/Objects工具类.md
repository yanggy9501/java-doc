# Objects工具类

> Objects 是 Object 的工具类，位于java.util包。
>
> **适用场景：**
>
>  它在流处理中相当有用，实用程序包括null或null方法，用于计算对象的哈希代码，返回对象的字符串，比较两个对象，以及检查索引或子范围值是否超出范围

## 1. equals方法

> 这里Objects.equals方法中已经做了非空判断，所以不会抛出空指针异常，它是null-save空指针安全的。如果两个参数相等，返回true，否则返回false。 因此，如果这两个参数是null ，返回true，如果只有一个参数为null ，返回false， 否则，通过使用第一个参数的equals方法确定相等性。

```java
public static boolean equals(Object a, Object b) {
    return (a == b) || (a != null && a.equals(b));
}
```



## 2. deepEquals方法

> **适用于数组比较**
>
> 该方法在数组比较中尤其有用，当我们在业务中需要判断两个复杂类型，比如使用了泛型的列表对象List、或者通过反射得到的对象，不清楚对象的具体类型时可以直接使用此方法来解决。



## 3. toString方法

> 返回指定对象的字符串表示形式。如果参数为null，则返回字符串“null”。
>
> 另外可有：
>
> #### toString(Object o, String nullDefault)

