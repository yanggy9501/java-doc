# Spring Kafka

## 1，springboot 整合 spring kafka

**依赖**

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>2.5.4.RELEASE</version>
</dependency>
```

**配置文件**

```yaml
spring:
  kafka:
    bootstrap-servers: hadoop101:9092,hadoop102:9092,hadoop103:9092
    # 消息生产者
    producer:
      # 消息重发次数
      retries: 0
      # 一个批次大小
      batch-size: 16384
      # 生产者内存缓冲区大小
      buffer-memory: 33554432
      # key 序列化
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      # value 序列化
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
      # ack 确认
      acks: all

    # 消息消费者
    consumer:
      # 自动提交的时间间隔，格式：1S, 1M, 2H, 3D
      auto-commit-interval: 1s
      # 该属性指定消费者在读取一个没有偏移量的分区或者偏移量无效的情况下该做何处理
      auto-offset-reset: earliest
      # 是否自动提交，默认 true
      enable-auto-commit: false
      # key 反序列化
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      # value 反序列化
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer

    # kafka 监听器
    listener:
      # ack 类型，manual_immediate：调用 ack 后立即提交 offset
      ack-mode: manual_immediate
      # 容器运行的线程数
      concurrency: 4
```

****

### 1.1 生产者

注入 kafkaTemplate，如：

```java
 @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;
```

发送消息

```java
ListenableFuture<SendResult<String, Object>> resultListenableFuture =
    kafkaTemplate.send("test", num, "hello kafka template" + num);
resultListenableFuture.addCallback(success -> {
    System.out.println("发送成功！");
    int partition = success.getRecordMetadata().partition();
    long offset = success.getRecordMetadata().offset();
    System.out.println("分区：" + partition);
    System.out.println("偏移量：" + offset);
}, failure -> {
    System.out.println("发送失败！");
});
```



### 1.2 消费者

```java
@Component
public class KafkaConsumerListener {

    /**
     * ConsumerRecord<?, ?>
     *     第一个泛型：消息 key 的类型
     *     第二个泛型：消息 value 的类型
     *
     * @param record record 用于获取分区和值等
     * @param ack ack 对象，用于提交ack确认
     * @param topic 主题
     */
    @KafkaListener(topics = "test", groupId = "consumer-group01")
    public void onMessage(ConsumerRecord<String, Object> record,
        Acknowledgment ack,
        @Header(KafkaHeaders.RECEIVED_TOPIC) String topic) {
        System.out.println("消息分区：" + record.partition());
        System.out.println("消费内容：" + record.value());
        // 确认消息
        ack.acknowledge();
    }
}
```

