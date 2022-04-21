# CyclicBarrier

>   字面意思回环栅栏，通过它可以实现让`一组线程都等待至某个状态（屏障点）之后在一起执行`。叫做回环是因为当所有等待线程都被释放以后，CyclicBarrier可以被重用。

## 1，CyclicBarrier方法

### 1.1 构造函数

```java
// parties表示屏障拦截的线程数量，每个线程调用 await 方法告诉 CyclicBarrier 我已经到达了屏障，然后当前线程被阻塞。
public CyclicBarrier(int parties)
// 用于在线程到达屏障时，优先执行 barrierAction，方便处理更复杂的业务场景(该线程的执行时机是在到达屏障之后再执行)
public CyclicBarrier(int parties, Runnable barrierAction)
```

### 1.2 实例方法

```java
// 屏障：指定数量的线程全部调用await()方法时，这些线程不再阻塞，一起往下执行（类似赛跑一块开始跑一样）
// BrokenBarrierException 表示栅栏已经被破坏，破坏的原因可能是其中一个线程 await() 时被中断或者超时
public int await() throws InterruptedException, BrokenBarrierException
// 同上，但是最多等待timeout时间，超过时会继续往下执行
public int await(long timeout, TimeUnit unit) throws InterruptedException, BrokenBarrierException, TimeoutException

// 通过reset()方法可以进行重置，CyclicBarrier可以被重用（回环）
public void reset()
```

## 2，CyclicBarrier应用

### 2.1 多线程计算

场景一：可以用于多线程计算数据，最后合并计算结果的场景。所有分支结果都算出，最后在主分支汇总。

```java
public class CyclicBarrierTest2 {

    //保存每个学生的平均成绩
    private ConcurrentHashMap<String, Integer> map=new ConcurrentHashMap<String, Integer>();

    private ExecutorService threadPool= Executors.newFixedThreadPool(3);

    private CyclicBarrier cb=new CyclicBarrier(3, ()->{
        int result=0;
        Set<String> set = map.keySet();
        for(String s:set){
            result+=map.get(s);
        }
        System.out.println("三人平均成绩为:"+(result/3)+"分");
    });

    public void count(){
        for(int i=0;i<3;i++){
            threadPool.execute(new Runnable(){
                @Override
                public void run() {
                    //获取学生平均成绩
                    int score=(int)(Math.random()*40+60);
                    map.put(Thread.currentThread().getName(), score);
                    System.out.println(Thread.currentThread().getName() + "同学的平均成绩为："+score);
                    try {
                        //执行完运行await(),等待所有学生平均成绩都计算完毕
                        cb.await();
                    } catch (InterruptedException | BrokenBarrierException e) {
                        e.printStackTrace();
                    }
                }
            });
        }
    }

    public static void main(String[] args) {
        CyclicBarrierTest2 cb=new CyclicBarrierTest2();
        cb.count();
    }
}
```

### 2.2 “人满发车”的场景

>   人满：即n个屏障全部到达
>
>   发车：触发回调函数

```java
public class CyclicBarrierTest3 {
    public static void main(String[] args) {
        AtomicInteger counter = new AtomicInteger();
        
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
                5, 5, 1000, TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(100),
                (r) -> new Thread(r, counter.addAndGet(1) + " 号 "),
                new ThreadPoolExecutor.AbortPolicy());

        CyclicBarrier cyclicBarrier = new CyclicBarrier(5, () -> System.out.println("裁判：比赛开始~~"));

        for (int i = 0; i < 10; i++) {
            threadPoolExecutor.submit(new Runner(cyclicBarrier));
        }

    }
    static class Runner extends Thread{
        private CyclicBarrier cyclicBarrier;
        public Runner (CyclicBarrier cyclicBarrier) {
            this.cyclicBarrier = cyclicBarrier;
        }

        @Override
        public void run() {
            try {
                int sleepMills = ThreadLocalRandom.current().nextInt(1000);
                Thread.sleep(sleepMills);
                System.out.println(Thread.currentThread().getName() + " 选手已就位, 准备共用时： " + sleepMills + "ms" + cyclicBarrier.getNumberWaiting());
                cyclicBarrier.await();

            } catch (InterruptedException e) {
                e.printStackTrace();
            }catch(BrokenBarrierException e){
                e.printStackTrace();
            }
        }
    }
}
```

## 3，CyclicBarrier与CountDownLatch

1.   CountDownLatch的计数器只能使用一次，而CyclicBarrier的计数器可以使用reset() 方法重置，且其有一个回调函数。所以CyclicBarrier能处理更为复杂的业务场景，比如：如果计算发生错误，可以重置计数器，并让线程们重新执行一次
2.   CyclicBarrier还提供getNumberWaiting(可以获得CyclicBarrier阻塞的线程数量)、isBroken(用来知道阻塞的线程是否被中断)等方法。
3.   CountDownLatch会阻塞主线程（汇总到主线程，主线程等待），CyclicBarrier不会阻塞主线程，只会阻塞子线程（子线程到达屏障点，如果屏障点是子线程的结束位置回调相当于主线程的话，其和CountDownLatch差不多）。
4.   CountDownLatch和CyclicBarrier都能够实现线程之间的等待，只不过它们侧重点不同。CountDownLatch一般用于一个或多个线程，等待其他线程执行完任务后，再执行。CyclicBarrier一般用于一组线程互相等待至某个状态，然后这一组线程再同时执行。
5.   CyclicBarrier 还可以提供一个 barrierAction，合并多线程计算结果。
6.   CyclicBarrier是通过ReentrantLock的"独占锁"和Conditon来实现一组线程的阻塞唤醒的，而CountDownLatch则是通过AQS的“共享锁”实现

## 4，CyclicBarrier源码分析

实现思路：当线程到达屏障点阻塞其线程（ReentrantLock），线程进入阻塞的同步队列，再将线程从同步队列 -> 条件队列（因为只有条件队列才满足唤醒功能），线程被唤醒之后线程由条件队列 -> 同步队列。

**关注点**：

1.一组线程在触发屏障之前互相等待，最后一个线程到达屏障后唤醒逻辑是如何实现的

2.删栏循环使用是如何实现的

3.条件队列到同步队列的转换实现逻辑 

![](asserts/30582.png)

**1. 获取锁，阻塞线程**

```java
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        // 核心方法
        return dowait(false, 0L);
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen
    }
}
```

**2. 上锁，屏障-1，屏障!=0则调用Condition#await入条件队列，阻塞线程等待唤醒**

```java
// Main barrier code, covering the various policies.
private int dowait(boolean timed, long nanos)
    throws InterruptedException, BrokenBarrierException,
TimeoutException {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        final Generation g = generation;

        if (g.broken)
            throw new BrokenBarrierException();

        if (Thread.interrupted()) {
            breakBarrier();
            throw new InterruptedException();
        }

        int index = --count;
        if (index == 0) {  // tripped
            boolean ranAction = false;
            try {
                final Runnable command = barrierCommand;
                if (command != null)
                    command.run();
                ranAction = true;
                nextGeneration();
                return 0;
            } finally {
                if (!ranAction)
                    breakBarrier();
            }
        }

        // loop until tripped, broken, interrupted, or timed out
        for (;;) {
            try {
                if (!timed)
                    trip.await();
                else if (nanos > 0L)
                    nanos = trip.awaitNanos(nanos);
            } catch (InterruptedException ie) {
                if (g == generation && ! g.broken) {
                    breakBarrier();
                    throw ie;
                } else {
                    // We're about to finish waiting even if we had not
                    // been interrupted, so this interrupt is deemed to
                    // "belong" to subsequent execution.
                    Thread.currentThread().interrupt();
                }
            }

            if (g.broken)
                throw new BrokenBarrierException();

            if (g != generation)
                return index;

            if (timed && nanos <= 0L) {
                breakBarrier();
                throw new TimeoutException();
            }
        }
    } finally {
        lock.unlock();
    }
}
```

**3. 唤醒线程，执行回调，重置count**

当count屏障计数为0，开始唤醒线程。

```java
if (index == 0) {  // tripped
    boolean ranAction = false;
    try {
        // 屏障是否由action（构造方法传入的）有则调用
        final Runnable command = barrierCommand;
        if (command != null)
            command.run();
        ranAction = true;
        // 唤醒+重置count，进行下一轮的屏障
        nextGeneration();
        return 0;
    } finally {
        if (!ranAction)
            breakBarrier();
    }
}
```

**4，**

```java
/**
 * Updates state on barrier trip and wakes up everyone.
 * Called only while holding lock.
 */
private void nextGeneration() {
    // signal completion of last generation
    trip.signalAll();
    // set up next generation
    count = parties;
    generation = new Generation();
}
```

**5. 唤醒：获取条件队列的头节点入同步队列，入队和出队操作+唤醒+state** 

```java
public final void signalAll() {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    // 
    Node first = firstWaiter;
    if (first != null)
        // 里面循环唤醒
        doSignalAll(first);
}
```

**6. enq 入队方法**

```java
final boolean transferForSignal(Node node) {
    	/*
         * If cannot change waitStatus, the node has been cancelled.
         */
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        return false;

   		 /*
         * Splice onto queue and try to set waitStatus of predecessor to
         * indicate that thread is (probably) waiting. If cancelled or
         * attempt to set waitStatus fails, wake up to resync (in which
         * case the waitStatus can be transiently and harmlessly wrong).
         */
    Node p = enq(node);
    int ws = p.waitStatus;
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
        LockSupport.unpark(node.thread);
    return true;
}
```

