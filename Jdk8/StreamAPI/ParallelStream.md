# ParallelStream 并行流



**1，获取并行流**

**方式1**

```java
List<String> list = new ArrayList<>();
String<String> stream = list.parallelStream();
```

**方式二**

> 将串行流转换为并行流

```java
List<String> list = new ArrayList<>();
String<String> stream = list.stream().parallel();
```

> 并行流其实就是一个并行指向的流，它通过默认的ForkJoinPool，可能提高性能

