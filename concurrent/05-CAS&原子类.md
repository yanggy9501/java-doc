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
>
>   cas只是指令，不涉及内核态和用户态的转换，是一种无锁操作。
>
>   **注意：**自己cas并不保证可见性，但是能够保证比较和交换这两步操作是原子性的，Java自己封装的自己保证了可见性。

**三个比较并交换的方法：**

 参数1：对象实例
 参数2：内存偏移量（基于对象实例首地址的偏移量，是属性基于首地址的位移）
 参数3：期望值（理解为修改之前的旧值）
 参数4：新值（理解为更新后的新值）

```java
public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);

public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);

public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);
```

**案例：**

*    UnsafeFactory.getFieldOffset：获取要操作对象属性的偏移量

```java
public class CASTest {

    public static void main(String[] args) {
        Entity entity = new Entity();
		// 自定义工厂
        Unsafe unsafe = UnsafeFactory.getUnsafe();

        long offset = UnsafeFactory.getFieldOffset(unsafe, Entity.class, "x");
        System.out.println(offset);
        boolean successful;

        // 4个参数分别是：对象实例、字段的内存偏移量、字段期望值、字段更新值
        successful = unsafe.compareAndSwapInt(entity, offset, 0, 3);
        System.out.println(successful + "\t" + entity.x);

        successful = unsafe.compareAndSwapInt(entity, offset, 3, 5);
        System.out.println(successful + "\t" + entity.x);

        successful = unsafe.compareAndSwapInt(entity, offset, 3, 8);
        System.out.println(successful + "\t" + entity.x);

    }   
}
```

**工厂：**

```java
public class UnsafeFactory {
    /**
     * 获取 Unsafe 对象
     * @return
     */
    public static Unsafe getUnsafe() {
        try {
            Field field = Unsafe.class.getDeclaredField("theUnsafe");
            field.setAccessible(true);
            return (Unsafe) field.get(null);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 获取字段的内存偏移量
     * @param unsafe
     * @param clazz
     * @param fieldName
     * @return
     */
    public static long getFieldOffset(Unsafe unsafe, Class clazz, String fieldName) {
        try {
            return unsafe.objectFieldOffset(clazz.getDeclaredField(fieldName));
        } catch (NoSuchFieldException e) {
            throw new Error(e);
        }
    }
}
```



### 2.1 CAS缺陷

>   CAS虽然高效的解决了原子操作，但是仍有一些缺陷。（CAS不涉及内核态和用户态的转换）

*   占用CPU（CAS 自旋，若长时间地不成功，则CPU开销大，因为一种在循环比较）
*    只能保证一个共享变量原子操作
*   ABA问题（变量值被修改之后有被改回来了，原线程还认为没有改变）

### 2.2 ABA问题

>   当有多个线程对一个原子类进行操作的时候，某个线程在短时间内将原子类的值A修改为B，又马上将其修改为A，此时其他线程不感知，还是会修改成功。



![image-20220325205022535](asserts/image-20220325205022535.png)

****

**ABA问题及其解决方案：**版本号

>   数据库有个锁称为乐观锁，是一种基于数据版本实现数据同步的机制，每次修改一次数据，版本就会进行累加。
>
>   同样，Java也提供了相应的原子引用类AtomicStampedReference<T>

````java
public class AtomicStampedReference<V> {

    private static class Pair<T> {
        final T reference;
        final int stamp;
        private Pair(T reference, int stamp) {
            this.reference = reference;
            this.stamp = stamp;
        }
        static <T> Pair<T> of(T reference, int stamp) {
            return new Pair<T>(reference, stamp);
        }
    }
````

>   *   reference即我们实际存储的变量
>   *   stamp是版本：每次修改+1



**AtomicMarkableReference**

>   AtomicMarkableReference可以理解为上面AtomicStampedReference的简化版，就是 不关心修改过几次，仅仅关心是否修改过。因此变量mark是boolean类型，仅记录值是否有过修改。



# 原子类
