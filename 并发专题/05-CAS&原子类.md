# CAS

## 1，什么是 CAS

>   CAS（Compare And Swap，比较并交换）- 通常指这种原理的原子操作。
>
>   如：针对一个变量，首先比较它的内存值（新值）与某个期望值（旧值）是否相同，如果相同，就给它赋一个新值（即你是否是我改动时的那个值，没有被其他线程修改）。
>
>   CAS两个操作`比较`和`交换`是一个不可分割的原子操作，其原子性由硬件层面保障。
>
>   CAS是一种`无锁算法`，在不使用锁（没有线程被阻塞）的情况下实现多线程之间的变量同步。



## 2，CAS应用

>   Java的CAS 操作是由 Unsafe 类提供支持的，该类定义了三种针对不同类型变量的 CAS 操作。

**三个比较并交换的方法：**

 参数1：对象实例
 参数2：内存偏移量（基于对象实例首地址的偏移量）
 参数3：期望值（理解为修改之前的旧值）
 参数4：新值（理解为更新后的新值）

```java
public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);

public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);

public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);
```

