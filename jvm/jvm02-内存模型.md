# Java内存模型&优化

## 1，Java内存模型

![](asserts/JVM内存模型.png)

*   栈（线程栈）

*   堆

*   方法区

    **在minor gc过程中对象挪动后，引用如何修改？**

    >   对象在堆内部挪动的过程其实是复制，原有区域对象还在，一般不直接清理，JVM内部清理过程只是将对象分配指针移动到区域的头位置即可。

### 1.1  JVM内存参数

![](asserts/clipboard.png)

*   堆：-Xm？
*   方法区：-XX:???
*   栈：-Xss

**案例：**Spring Boot程序的JVM参数设置

```sh
java -Xms2048M -Xmx2048M -Xmn1024M -Xss512K -XX:MetaspaceSize=256M -XX:MaxMetaspaceSize=256M -jar microservice-eureka-server.jar
```

>   *   -Xss：每个线程的栈大小
>   *   -Xms：设置堆的初始可用大小(即最小值)，默认物理内存的1/64 
>   *   -Xmx：设置堆的最大可用大小，默认物理内存的1/4
>   *   -Xmn：新生代大小
>   *   -XX:NewRatio：默认2表示新生代占年老代的1/2，占整个堆内存的1/3。
>   *   -XX:SurvivorRatio：默认8表示一个survivor区占用1/8的Eden内存，即1/10的新生代内存
>   *   -XX：MaxMetaspaceSize： 设置元空间最大值， 默认是-1， 即不限制
>   *   -XX：MetaspaceSize： 指定元空间触发Fullgc的初始阈值，字节为单位，默认是21M左右。

### 1.2 JVM内存参数大小如何设置

>   JVM参数大小设置并没有固定标准，需要根据实际项目情况分析。
>
>   **就是尽可能让对象都在新生代里分配和回收，尽量别让太多对象频繁进入老年代，避免频繁对老年代进行垃圾回收，同时给系统充足的内存大小，避免新生代频繁的进行垃圾回收。**

