# AQS&独占锁ReentrantLock

## 1，AQS

>   AbstractQueuedSynchronizer（简称AQS，抽象的队列同步器），java.util.concurrent包中的大多数同步器实现都是围绕着其基础。如：Lock, Latch, Barrier等

**AQS具备的特性**：

-   阻塞等待队列
-   共享/独占
-   公平/非公平
-   可重入
-   允许中断

**AQS定义两种资源共享方式**

-   Exclusive-独占，只有一个线程能执行，如ReentrantLock
-   Share-共享，多个线程可以同时执行，如Semaphore/CountDownLatch

**AQS定义两种队列**

-   同步等待队列： 主要用于维护获取锁失败时入队的线程（就是就绪队列）
-   条件等待队列： 调用await()的时候会释放锁，然后线程会加入到条件队列，调用signal()唤醒的时候会把条件队列中的线程节点移动到同步队列中，等待再次获得锁（就是阻塞队列，Condition时使用）

**AQS内部维护属性volatile int state**

>   state属性表示资源的可用状态
>
>   State三种访问方式： 
>
>   *   getState() 
>
>   *   setState() 
>   *   compareAndSetState()
>
>   AQS 定义了5个队列中节点状态：
>
>   1.   值为0，初始化状态，表示当前节点在sync队列（同步|就绪队列）中，等待着获取锁。
>   2.    值为1，CANCELLED 表示当前的线程被取消；
>   3.   值为-1，SIGNAL 表示当前节点的后继节点包含的线程需要运行，也就是unpark（唤醒后继节点所代表的线程） 
>   4.   值为-2，CONDITION表示当前节点在等待condition，也就是在condition（阻塞）队列中
>   5.   值为-3，PROPAGATE 表示当前场景下后续的acquireShared能够得以执行（共享）

### 1.1 AQS方法

>   *   tryAcquire: 独占方式，尝试获取资源，成功则返回true，失败则返回false
>   *   tryRelease(int)：独占方式，尝试释放资源，成功则返回true，失败则返回false。
>
>   tryRelease
>
>   state代表资源状态（加锁状态，1：加锁，0：无锁）

```java
public class TulingLock extends AbstractQueuedSynchronizer{
    @Override
    protected boolean tryAcquire(int unused) {
        //cas 获取资源
        if (compareAndSetState(0, 1)) {
            setExclusiveOwnerThread(Thread.currentThread());
            return true;
        }
        return false;
    }

    @Override
    protected boolean tryRelease(int unused) {
        // 释放资源 
        setExclusiveOwnerThread(null);
        setState(0);
        return true;
    }
}
```

### 1.2 同步队列

>   AQS当中的同步等待队列也称CLH队列，一种基于双向链表数据结构的队列，是FIFO先进先出线程等待队列，Java中的CLH队列是原CLH队列的一个变种, 线程由原自旋机制改为阻塞机制。

**AQS 依赖CLH同步队列来完成同步状态的管理：**

*   当前线程如果获取同步状态失败时(将stata置为1)，AQS则会将当前线程已经等待状态等信息构造成一个节点（Node）并将其`加入到CLH同步队列`，同时会`阻塞当前线程`.
*   当同步状态释放时（将stata置为0），会把首节点唤醒（节点的stata置为-1）（公平锁），使其再次尝试获取同步状态。
*   通过signal或signalAll将条件队列中的节点转移到同步队列。

![image-20220409190334899](asserts/image-20220409190334899.png)

### 1.3 条件队列

**AQS中条件队列是使用单向列表保存的，用nextWaiter来连接:**

*   调用condition的await方法阻塞线程，进入条件队列
*   当前线程存在于同步队列的头结点，调用await方法进行阻塞（从同步队列转到条件队列）

## 2，Condition

>   Condition是Lock的条件对象，是一个接口由子类实现，类上类似于Object的wait和notify方法。
>
>   java.util.concurrent类库中提供Condition类来实现线程之间的协调。调用Condition.await() 方法使线程等待，其他线程调用Condition.signal() 或 Condition.signalAll() 方法唤醒等待的线程。
>
>   注意：调用Condition的await()和signal()方法，都必须在lock保护之内。

### 2.1 接口方法

```java
// 阻塞线程，同时释放锁（线程由运行状态 -> 条件队列，waitStatus = -2）
void await() throws InterruptedException;
void awaitUninterruptibly();
long awaitNanos(long nanosTimeout) throws InterruptedException;
boolean await(long time, TimeUnit unit) throws InterruptedException;
boolean awaitUntil(Date deadline) throws InterruptedException;
// 唤醒线程(线程由条件队列 -> 同步队列，waitStatus = -1)，当线程竞争到cpu时还要去获取锁，得到锁之后才能执行（waitStatus = 0），执行完还要释放锁
void signal();
void signalAll();
```

1.   调用Condition#await方法会`释放当前持有的锁`，然后`阻塞当前线程`，同时向 Condition条件队列尾部添加一个节点，所以调用Condition#await方法的时候必须持有锁。
2.   调用Condition#signal方法会将Condition条件队列的首节点移动到同步队列尾部（唤醒），然后唤醒因调用Condition#await方法而阻塞的线程(唤醒之后这个线程就可以去竞争锁了)，所以调Condition#signal方法的时候必须持有锁，持有锁的线程唤醒被因调用 Condition#await方法而阻塞的线程。

**ps：** Condition对象和Lock对象是绑定的，类似于Object的wait方法，调用阻塞方法是锁对象调用的。

```java
@Slf4j
public class ConditionTest {
    public static void main(String[] args) {
        Lock lock = new ReentrantLock();
        Condition condition = lock.newCondition();
        // 线程1
        new Thread(() -> {
            lock.lock();
            try {
                log.debug(Thread.currentThread().getName() + " 开始处理任务");
                // 条件对象调用await方法，阻塞当前线程，同时释放锁（线程由running - 条件队列），等待被唤醒（被唤醒之后还要获取锁，因为最后还要释放锁呢）
                condition.await();
                log.debug(Thread.currentThread().getName() + " 结束处理任务");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }).start();
		// 线程2
        new Thread(() -> {
            lock.lock();
            try {
                log.debug(Thread.currentThread().getName() + " 开始处理任务");
                Thread.sleep(2000);
                // 通过Lock.condition唤醒被该condition阻塞的线程
                condition.signal();
                log.debug(Thread.currentThread().getName() + " 结束处理任务");
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }).start();
    }
```

## 3，ReentrantLock

>   ReentrantLock是一种基于AQS框架的应用实现，是一种互斥锁，可以保证线程安全（唯一实现Lock接口）。
>
>   相对于 synchronized，具备如下特点：
>
>   *    可中断 
>   *   可以设置超时时间 
>   *   可以设置为公平锁
>   *    支持多个条件变量
>   *    与 synchronized 一样，都支持可重入

ReentrantLock实现Lock接口，且是其唯一实现

### 3.1 源码解析

![](asserts/29881.png)

关注点： 

1.   ReentrantLock加锁解锁的逻辑

2.   公平和非公平，可重入锁的实现

3.   线程竞争锁失败入队阻塞逻辑和获取锁的线程释放锁唤醒阻塞线程竞争锁的逻辑实现 

     (设计的精髓：并发场景下入队和出队操作）

`加锁一般放在try之前，解锁放在finally代码块中`

```java
ReentrantLock lock = new ReentrantLock(); //参数默认false，不公平锁  
ReentrantLock lock = new ReentrantLock(true); //公平锁  
// 加锁    
lock.lock(); 
try {  
    //临界区 
} finally { 
    // 解锁 
    lock.unlock(); 
}
```

#### 加锁&阻塞

**1**

sync是内部类Sync的实例对象，其实extends AbstractQueuedSynchronizer抽象的队列同步器，而AQS定义了两种队列，以及节点state状态（理解线程状态变量）

```java
public void lock() {
    sync.lock();
}
```

**2** 非公平锁的aqs加锁操作

*   compareAndSetState：修改state状态值为1，表示该线程是否占有锁
*   setExclusiveOwnerThread：设置独占线程
*   acquire：获取失败，线程进入阻塞队列

```java
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```

**3** **入队&阻塞**

*   tryAcquire：aqs定义的方法，尝试获取锁资源
*   addWaiter：创建一个阻塞队列的节点，该节点代表该线程，并且插入链表尾部（入队）
*   acquireQueued：构建Node，阻塞线程

```java
// arg = 1
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

addWatier

>   这个入队操作很经度，值得学习
>
>   这里采用的链表队列，且没有head，需要判断是否有头节点，使用的是头差法

```java
// 创建节点并追加到tail
private Node addWaiter(Node mode) {
    // 创建节点（包装线程和lock模式）
     Node node = new Node(Thread.currentThread(), mode);
     // tail 是实列是等待队列（阻塞队列）的尾指针
     Node pred = tail;
    // 尾节点尾null说明是空链表，需要创建一个头节点
     if (pred != null) {
         node.prev = pred;
         if (compareAndSetTail(pred, node)) {
             pred.next = node;
             return node;
         }
     }
    // 头为null时，创建头在入队
     enq(node);
     return node;
 }
```

enq

由于是并发场景需要死循环来做，成功return退出

```java
private Node enq(final Node node) {
    for (;;) {
        // 获取尾指针 
        Node t = tail;
        if (t == null) {
            // cas 循环尝试设置头
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            // 以及存储头了，设置尾
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

acquireQueued

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            // 获取当前线程或说node的前驱，前面以及入队了
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

#### 释放锁&唤醒

>   看unlock方法。

```java
 public final boolean release(int arg) {
     if (tryRelease(arg)) {
         Node h = head;
         if (h != null && h.waitStatus != 0)
             unparkSuccessor(h);
         return true;
     }
     return false;
 }
```

### 3.2 ReentrantLock特性

#### 1. 可重入

同一把锁lock，可用lock的“内部”再次使用

```java
public class ReentrantLockDemo2 {
    public static ReentrantLock lock = new ReentrantLock();
    public static void main(String[] args) {
        method1();
    }

    public static void method1() {
        lock.lock();
        try {
            log.debug("execute method1");
            method2();
        } finally {
            lock.unlock();
        }
    }
    public static void method2() {
        lock.lock();
        try {
            log.debug("execute method2");
            method3();
        } finally {
            lock.unlock();
        }
    }
    public static void method3() {
        lock.lock();
        try {
            log.debug("execute method3");
        } finally {
            lock.unlock();
        }
    }
}
```

#### 2.可中断           

正在执行的线程，可用被其他线程调用该线程的中断方法中断线程，线程被中断要释放锁，所以lock必须设置可以中断。

1.   lock.lockInterruptibly();   // lock设置可中断，中断是释放锁

2.   thread.interrupt(); // 线程中断

通过 InterruptedException异常捕捉       

```java
public class ReentrantLockDemo3 {
    public static void main(String[] args) {
        ReentrantLock lock = new ReentrantLock();
        Thread t1 = new Thread(() -> {
            log.debug("t1启动...");
            try {
                // 设置可中断
                lock.lockInterruptibly();
                try {
                    log.debug("t1获得了锁");
                } finally {
                    lock.unlock();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
                log.debug("t1等锁的过程中被中断");
            }

        }, "t1");
		// 主线程获取锁
        lock.lock();
        try {
            log.debug("main线程获得了锁");
            // 主线程中启动t1线程
            t1.start();
            // 先让线程t1执行
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            t1.interrupt();
            log.debug("线程t1执行中断");
        } finally {
            lock.unlock();
        }
    }

// 锁超时，立即失败
@Slf4j
public class ReentrantLockDemo4 {
    public static void main(String[] args) {
        ReentrantLock lock = new ReentrantLock();
        // 线程1
        Thread t1 = new Thread(() -> {
            log.debug("t1启动...");
            // 注意： 即使是设置的公平锁，此方法也会立即返回获取锁成功或失败，公平策略不生效
            if (!lock.tryLock()) {
                log.debug("t1获取锁失败，立即返回false");
                return;
            }
            try {
                log.debug("t1获得了锁");
            } finally {
                lock.unlock();
            }
        }, "t1");

        lock.lock();
        try {
            log.debug("main线程获得了锁");
            t1.start();
            //先让线程t1执行
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        } finally {
            lock.unlock();
        }
    }
}
```

#### 3. 超时失败

lock方法是获取锁，阻塞直到获取为此，如果希望在指定时间内获取锁，需要tryLock方法。

```java
ReentrantLock lock = new ReentrantLock();
Thread t1 = new Thread(() -> {
    log.debug("t1启动...");
    // 超时
    try {
        if (!lock.tryLock(1, TimeUnit.SECONDS)) {
            log.debug("等待 1s 后获取锁失败，返回");
            return;
        }
    } catch (InterruptedException e) {
        e.printStackTrace();
        return;
    }
    try {
        log.debug("t1获得了锁");
    } finally {
        lock.unlock();
    }
        }, "t1");
```

#### 4. 公平和非公平锁

```java
// 公平锁 
ReentrantLock lock = new ReentrantLock(true); 
```

### 3.3 synchronized与ReentrantLock

-   synchronized是JVM层次的锁实现，ReentrantLock是JDK层次的锁实现；
-   synchronized的锁状态是无法在代码中直接判断的，但是ReentrantLock可以通过ReentrantLock#`isLocked`判断；
-   synchronized是非公平锁，ReentrantLock是可以是公平也可以是非公平的；
-   synchronized是不可以被中断的，而ReentrantLock#lockInterruptibly方法是可以被中断的；
-   在发生异常时synchronized会自动释放锁，而ReentrantLock需要开发者在finally块中显示释放锁；
-   ReentrantLock获取锁的形式有多种：如立即返回是否成功的tryLock(),以及等待指定时长的获取，更加灵活；
-   synchronized在特定的情况下对于已经在等待的线程是后来的线程先获得锁（回顾一下sychronized的唤醒策略），而ReentrantLock对于已经在等待的线程是先来的线程先获得锁；
