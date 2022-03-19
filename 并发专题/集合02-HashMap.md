# 集合底层原理&并发

![image-20220316224941641](asserts/image-20220316224941641.png)

**PS: ** 序列化是为了

*   持久化存盘：由于对象及其数据是存在内存中，希望把某个对象和数据保存下来，就需要持久化存盘。
*   网络传输：内存中对象要在网络中传输，必须序列化成为二进制文件才能传输。

## 1，ArrayList

>   存储结构：底层采用数组来实现的。
>
>   添加元素：默认尾部添加，效率比较高。

### 1.1 基本属性

*   默认长度：10个（如果实例化时未指定容量，则在初次添加元素时会进行扩容使用此容量作为数组长度）
*   扩容：newCapacity = oldCapacity + (oldCapacity >> 1) 默认将扩容至原来容量的1.5 倍

```java
// 序列化版本号（类文件签名），如果不写会默认生成，类内容的改变会影响签名变化，导致反序列化失败
private static final long serialVersionUID = 8683452581122892189L;
// 如果实例化时未指定容量，则在初次添加元素时会进行扩容使用此容量作为数组长度
private static final int DEFAULT_CAPACITY = 10;
//static修饰，所有的未指定容量的实例(也未添加元素)共享此数组，两个空的数组有什么区别呢？ 就是第一次添加元素时知道该 elementData 从空的构造函数还是有参构造函数被初始化的。以便确认如何扩容。空的构造器则初始化为10，有参构造器则按照扩容因子扩容
private static final Object[] EMPTY_ELEMENTDATA = {};
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
// arrayList真正存放元素的地方，长度大于等于size
transient Object[] elementData;
// arrayList中的元素个数
private int size;
```

### 1.2 构造方法

*   不指定容量：初始化时是空数组，大小为0，当第一次add元素的才初始化数组，大小为10.
*   指定容量：直接创建大小为指定容量的数组

```java
 //无参构造器，构造一个容量大小为 10 的空的 list 集合，但构造函数只是给 elementData 赋值了一个空的数组，其实是在第一次添加元素时容量扩大至 10 的。
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
    //当使用无参构造函数时是把 DEFAULTCAPACITY_EMPTY_ELEMENTDATA 赋值给 elementData。 当 initialCapacity 为零时则是把 EMPTY_ELEMENTDATA 赋值给 elementData。 当 initialCapacity 大于零时初始化一个大小为 initialCapacity 的 object 数组并赋值给 elementData。
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
        }
    }
    //将 Collection 转化为数组，数组长度赋值给 size。 如果 size 不为零，则判断 elementData 的 class 类型是否为 ArrayList，不是的话则做一次转换。 如果 size 为零，则把EMPTY_ELEMENTDATA 赋值给 elementData，相当于new ArrayList(0)。
        16 public ArrayList(Collection<? extends E> c) {
        Object[] a = c.toArray();
        if ((size = a.length) != 0) {
            if (c.getClass() == ArrayList.class) {
                elementData = a;
            } else {
                elementData = Arrays.copyOf(a, size, Object[].class);
            }
        } else {
        // 指向空数组
            elementData = EMPTY_ELEMENTDATA;
        }
    }
```



### 1.3 添加元素

>   添加元素采用开辟原地复制的方法，不是移动。

**指定下标添加：**

```java
public void add(int index, E element) {
    //下标越界检查
    rangeCheckForAdd(index);
    //  判断扩容,记录操作数
    ensureCapacityInternal(size + 1);
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}
```



### 1.4 扩容

```java
private void grow(int minCapacity) {
    //获取当前数组长度
    int oldCapacity = elementData.length;
    // 默认将扩容至原来容量的1.5 倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```



## 2，HashMap

>   key-value存储，key可以为null，value也可以为null，同样的key保存会被覆盖掉之前的。
>
>   **存储结构：**底层采用数组、链表、红黑树来实现的。
>
>   Hashcode：通过字符串算出它的ascii码，进行mod（取模），算出哈希表中的下标。



![image-20220317193531332](asserts/image-20220317193531332.png)

**注意：**链表头插法在多线程情况下有循环引用问题，导致插不进去。

## 3，ConcurrentHashMap（并发安全map）

>   ConcurrentHashMap是并发安全的HashMap ，比Hashtable效率更高，Hashtable是synchronized锁住方法的，粒度比较大。
>
>   ConcurrentHashMap内部大量采用CAS操作。

