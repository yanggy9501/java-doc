# AQS&独占锁ReentrantLock

## 1，AQS

>   AbstractQueuedSynchronizer（简称AQS，抽象队列同步器），java.util.concurrent包中的大多数同步器实现都是围绕着其基础。如：Lock, Latch, Barrier等

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

-   同步等待队列： 主要用于维护获取锁失败时入队的线程
-   条件等待队列： 调用await()的时候会释放锁，然后线程会加入到条件队列，调用signal()唤醒的时候会把条件队列中的线程节点移动到同步队列中，等待再次获得锁

