# 并发锁synchronized

synchronized有同步和互斥的含义，但是两种是有区别的：

*   互斥：是保证同一时刻只能有一个线程执行临界区代码。
*   同步：保证程序的执行顺序。

## 1，synchronized 使用

>   synchronized是Java提供的一个原子性`内置锁`，java中的每一个对象都可以当作一个同步锁。

### 1.1 加锁方式

| 分类       | 被锁对象       | 伪代码                                        | 补充 |
| ---------- | -------------- | --------------------------------------------- | ---- |
| 实例方法   | 实例对象       | synchronized void method(){...}               |      |
| 静态方法   | 字节码对象     | public static synchronized void method(){...} |      |
| 实例对象   | 实例对象       | synchronized (this) {}                        |      |
| 字节码对象 | 字节码对象     | synchronized (Demo.class) {}                  |      |
| 任意Object | 实例对象Object | synchronized (instance) {}                    |      |

### 1.2 synchronized 底层原理

>   synchronized是JVM内置锁，基于Monitor机制实现，依赖底层操作系统的`互斥原语 `Mutex（互斥量），它是一个重量级锁，性能较低。
>
>   在jdk1.5之后版本做了优化，如锁粗化（Lock Coarsening）、锁消除（Lock Elimination）、轻量级锁（Lightweight Locking）、偏向锁（Biased Locking）、自适应自旋（Adaptive Spinning）等技术来减少锁操作的开销，内置锁的并发性能已经基本与Lock持平。
>
>   synchronized 可能会导致“用户态和内核态”两个态 之间来回切换，对性能有较大影响。

#### 1. Monitor（管程/监视器）

>   Monitor，直译为“监视器”（管程），是指管理共享变 量以及对共享变量操作的过程，让它们支持并发。
>
>   synchronized关键字和wait()、notify()、notifyAll()这三个方法是 Java中实现管程技术的组成部分。

#### 2. MESA 管程模型

![](asserts/23757.png)

​		管程中引入了条件变量的概念，而且每个条件变量都对应有一个等待队列。条件变量和等待队列的作用是解决线程之间的同步问题。

##### wait 方法

wait方法存在虚假唤醒的问题，wait会释放锁，所以其只能在同步代码块中使用，由于线程被唤醒之后，其还要竞争cpu但是锁被其他线程拿去了，下一次的时候其是没有被锁概念的，该线程本应该还要等待，但是其执行下去了。

```java
while(条件不满足) {
  wait();
}
```

##### notify和notifyAll方法

满足以下三个条件时，可以使用notify()，其余情况尽量使用notifyAll()：

*   所有等待线程拥有相同的等待条件；
*   所有等待线程被唤醒后，执行相同的操作；
*   只需要唤醒一个线程。



#### 3. Monitor机制

>   java.lang.Object 类定义了 wait()，notify()，notifyAll() 方法，这些方法的具体实现，依赖于 ObjectMonitor 实现

```java
ObjectMonitor() {
    _header       = NULL; //对象头  markOop
    _count        = 0;  
    _waiters      = 0,   
    _recursions   = 0;   // 锁的重入次数 
    _object       = NULL;  //存储锁对象
    _owner        = NULL;  // 标识拥有该monitor的线程（当前获取锁的线程） 
    _WaitSet      = NULL;  // 等待线程（调用wait）组成的双向循环链表，_WaitSet是第一个节点
    _WaitSetLock  = 0 ;    
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ; //多线程竞争锁会先存到这个单向链表中 （FILO栈结构）
    FreeNext      = NULL ;
    _EntryList    = NULL ; //存放在进入或重新进入时被阻塞(blocked)的线程 (也是存竞争锁失败的线程)
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
    _previous_owner_tid = 0;
```

![](asserts/24692.png)

>   在获取锁时，是将当前线程插入到\_cxq的头部，而释放锁时，默认策略（QMode=0）是：
>
>   *   如果EntryList为空，则将\_cxq中的元素按原有顺序插入到EntryList，并唤醒第一个线程，也就是当EntryList为空时，是后来的线程先获取锁。
>   *   \_EntryList不为空，直接从_EntryList中唤醒线程。

synchronized加锁加在对象上，对象是如何记录锁状态的？

#### 4. 对象的内存布局

>   Hotspot虚拟机中，对象在内存中存储的布局可以分为三块区域：
>
>   *   对象头（Header）：如 hash码，对象年代，对象锁，锁状态标志，偏向锁（线程）ID，偏向时间，数组长度（数组对象才有）等
>   *   实例数据（Instance Data）: 属性数据，包括父类的属性信息
>   *   对齐填充（Padding）：存储大小补齐，保证8B的倍数

![](asserts/23870.png)

#### 5. 对象头详解

java的对象头由以下三部分组成：

1，Mark Word

2，指向类的指针 Klass Pointer

3，数组长度（数组对象才有）

****

**1. Mark Word**

>   Mark Word记录了对象和锁有关的信息，当这个对象被synchronized关键字当成同步锁时，围绕这个锁的一系列操作都和Mark Word有关。
>
>   每一行代表对象处于某种状态时的样子。
>
>   *   32 位 JVM 的 Mark word 为 32 位，64 位 JVM 为 64 位

![](asserts/24028.png)

*   hash：对象的哈希码
*   age：对象分代年龄，对象被GC的次数
*   biased_lock： 偏向锁标识位



**2. Klass Pointer**

>   对象头的另外一部分是klass`类型指针`，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。
>
>   *   32位 4字节
>   *   64位开启指针压缩或最大堆内存<32g是4字节，否则8字节

**3. 数组长度**

>   如果对象是一个数组, 那在对象头中还必须有一块数据用于记录数组长度。 
>
>   *   4字节

#### 6. 偏向锁

>   **偏向锁的目标是，减少无竞争且只有一个线程使用锁的情况下，使用轻量级锁产生的性能消耗**。轻量级锁每次申请、释放锁都至少各需要一次CAS，但偏向锁只有初始化时需要一次CAS。

**优点**

-   只需要执行一次CAS即可获取锁
-   采用延迟释放锁策略
-   锁重入时，只需要判断mark_word.threadId是否为当前threadId即可

#### 7. 轻量级锁

**轻量级锁的目标是，减少无实际竞争情况下，使用重量级锁产生的性能消耗**，包括系统调用引起的内核态与用户态切换、线程阻塞造成的线程切换等。

#### 8. 重量级锁

>   即当有其他线程占用锁时，当前线程会进入阻塞状态，通常使用的synchrozied。

