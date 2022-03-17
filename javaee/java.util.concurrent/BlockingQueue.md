# BlockingQueue 阻塞队列

**阻塞队列**

> 阻塞队列，顾名思义首先它是一个队列，通过一个共享的队列，可以使得数据又队列的一端输入，从另一端输出。
>
> 1. 当队列空时，从队列中获取元素的操作将会被阻塞
> 2. 当队列满时，向队列添加元素的操作将会被阻塞

**为什么需要阻塞队列？**

> 好处时我们不需要关系是么时候需要阻塞线程，什么时候需要唤醒线程，因为这一切操作BlockingQueue都会帮我们完成。



## **1，阻塞队列基本架构**

> JUC下 BlockingQueue 是一个接口
>
> **实现类：**
>
> * ArrayBlockingQueue (常用)
>
>   > 基于数组的阻塞队列实现，大小有界默认 Integer.MAX_VALUE
>
> * LinkedBlockingQueue (常用)
>
>   > 基于链表的阻塞队列实现，大小有界默认 Integer.MAX_VALUE
>
> * DelayQueue
>
>   > 队列中的元素只有当其指定的延迟时间到了，才能从队列中获取元素。大小无限制
>
> * ...

**方法**

| 方法类型               | 会抛出异常 | 返回特殊值 | 会阻塞   | 超时退出结束                     |
| ---------------------- | ---------- | ---------- | -------- | -------------------------------- |
| 插入（入队）           | add(E e)   | offer(E e) | put(E e) | offer(E e, Time time, Unit unit) |
| 移除（出队）           | remove()   | poll()     | take()   | poll(Time time, Unit unit)       |
| 检查（获取元素不移除） | element()  | peek()     | 无       | 无                               |

**说明**

| 类型     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| 抛出异常 | 1，当阻塞队列满时，往队列add元素会抛出异常<br />2，当阻塞队列空时，向队列取元素会抛出异常 |
| 特殊值   | 1，插入元素，成功返回true，失败返回false<br />2，移除元素，成功返回出队元素，否则返回null |
| 阻塞     | 1，阻塞队列满时，生产线程再往队列put元素，队列会阻塞生产者线程直到队列空出或者响应中断<br />2，阻塞队列空时，消费者线程从队列取出元素，队列会阻塞消费者线程直到队列有元素可用 |
| 超时     | 超过限时后就会退出                                           |
|          |                                                              |



第一组

```java
public static void main(String[] args) {
    // 创建阻塞队列
    ArrayBlockingQueue<Object> blockingQueue = new ArrayBlockingQueue<>(10);

    // 添加元素
    System.out.println(blockingQueue.add("a")); // true
    System.out.println(blockingQueue.add("b")); // true
    System.out.println(blockingQueue.add("c")); // true
    System.out.println(blockingQueue.element()); // a
    
    // 出队
    System.out.println(blockingQueue.remove()); // a
    System.out.println(blockingQueue.remove()); // b
    System.out.println(blockingQueue.remove()); // c
    System.out.println(blockingQueue.remove()); // java.util.NoSuchElementException
    
}
```

第二组

```java
public static void main(String[] args) {
    // 创建阻塞队列
    ArrayBlockingQueue<Object> blockingQueue = new ArrayBlockingQueue<>(3);

    // 添加元素
    System.out.println(blockingQueue.offer("a")); // true
    System.out.println(blockingQueue.offer("b")); // true
    System.out.println(blockingQueue.offer("c")); // true
    System.out.println(blockingQueue.offer("d")); // false
    System.out.println(blockingQueue.peek()); // a

    // 出队
    System.out.println(blockingQueue.poll()); // a
    System.out.println(blockingQueue.poll()); // b
    System.out.println(blockingQueue.poll()); // c
    System.out.println(blockingQueue.poll()); // null
}
```

第三组

```java
public static void main(String[] args) throws InterruptedException {
    // 创建阻塞队列
    ArrayBlockingQueue<Object> blockingQueue = new ArrayBlockingQueue<>(3);

    // 添加元素
    blockingQueue.put("a");
    blockingQueue.put("b");
    blockingQueue.put("c");
    blockingQueue.put("d"); // 阻塞线程，该程序阻塞，程序会等待。 取出也是一样
}
```

