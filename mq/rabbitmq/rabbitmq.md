# 消息中间件-rabbitmq

## 1，mq简介

### 1.1 mq 作用

>   mq是一个存储消息的数据结构-队列，其还是一种跨进程的通信机制，用于上下游的消息传递服务。
>
>   概况作用：`跨进程的通信机制`

#### 1. 流量消峰

 **流量消峰**： 消息队列做缓冲，限制瞬时流量高峰（请求激增），请求排队处理，避免激增流量压垮服务器。

#### 2. 异步处理

**异步处理**：多个服务间异步调用，服务间调用不在同步等待对方完成。

#### 3. 应用解耦

**应用解耦**：本质还是异步处理，将服务分离出去，一个服务宕机不影响其他服务的运行。

#### 4. 通信

**PS**:  这些作用本质就是mq的跨进程的通信机制

## 2，核心概念

>   RabbitMQ 是一个消息中间件，主要作用就是接收消息，发送消息。

### 2.1 四大核心概念

#### 2.1.1  生产者

>   生成消息的client，并将消息发送给broker消息服务器（发送消息并携带routing-key）

#### 2.1.2 交换机

>   交换机是rabbitmq特有的组件，主要就是分发消息。
>
>   一方面它接收来自生产者的消息，另一方面它将消息推送到队列中或丢弃（根据routing-key）。

#### 2.1.3 队列

>   队列是 RabbitMQ 内部使用的一种数据结构，就是存储消息的缓冲区。
>
>   队列的大小仅受主机的`内存`和`磁盘`限制的约束.

#### 2.1.4 消费者

>   消费消息的client。

**请注意生产者，消费者和消息中间件很多时候并不在同一机器上。同一个应用程序既可以是生产者又是可以是消费者。**

### 2.2 名词介绍 

![image-20220329234027960](assets/image-20220329234027960.png)

*   **broker：** RabbitMQ Server即mq服务器
*   **virtual host：**逻辑上的主机划分，将一台mq服务器逻辑分成多个，以供不同用户或应用使用。
*   **connection**：publisher／consumer 和 broker 之间的 TCP 连接
*   **channel：** Channel 是在 connection 内部建立的逻辑连接
*   **exchange**：rabbitmq特有组件，根据分发规则（routing-key），将消息路由到不同队列中
*   **queue：**存储消息的数据结构，存储消息并等待消息被consumer取走，因此消费端面对的是队列。
*   **binding**：exchange和queue之间的虚拟连接，binding 中可以包含 routing key，Binding 信息被保存到 exchange 中的查询表中，用于 message 的分发。

### 2.3 安装

**1. docker安装：**带management版本的是代表有web管理界面

```sh
docker pull rabbitmq:management
```

```sh
docker run --name rabbitmq -p 5671:5671 -p 5672:5672 -p 4369:4369 -p 15671:15671 -p 15672:15672 -p 25672:25672 -d rabbitmq:management
```

**2. 登录**

url：http://localhost:15672

默认账号密码：guest

**3.端口**

*   4369, 25672 (Erlang发现&集群端口)

*   **5672, 5671** (AMQP端口)

*   **15672** (web管理后台端口)

*   61613, 61614 (STOMP协议端口)

*   1883, 8883 (MQTT协议端口)

## 3，rabbitmq使用

**pom依赖**

```xml
<!-- rabbitmq 客户端依赖 -->
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
</dependency>

<!-- rabbitmq springboot 客户端依赖 -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

### 3.1 hello world简单模式

>   该模式比较简单，生产者和消费这之间是**`一对一关系`**，消息生成把消息之间传递给队列，消息消费者之间中队列取消息（其实存在默认交换机的）。
>
>   *   生产者之间将消息发送给队列，消费者直接从队列中获取消息
>   *   使用的是默认交换机，所以queue与exchange不需要绑定
>   *   一对一关系，一个队列一个消费者

![image-20220329235527818](assets/image-20220329235527818.png)

#### 3.1.1 消息生产者

```java
public class Producer {
    /**
    * 队列，helloworld模式就是队列
    */
    private final static String QUEUE_NAME = "hello";
    public static void main(String[] args) throws Exception {
    // 创建一个连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        factory.setUsername("guest");
        factory.setPassword("guest");
        // channel 实现了自动 close 接口自动关闭不需要显示关闭
        try(Connection connection = factory.newConnection(); Channel channel = Channelconnection.createChannel()) {
            /**
             * 生成一个队列（无就创建，有则使用旧的）
             * 1.队列名称
             * 2.队列里面的消息是否持久化，默认消息存储在内存中，关闭服务时小时
             * 3.该队列是否只供一个消费者进行消费，是否进行共享 true 可以多个消费者消费
             * 4.是否自动删除，最后一个消费者端开连接以后，该队列是否自动删除 true 自动删除
             * 5.其他参数（声明队列的其他参数，如：队列长度等队列特性）
             */
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            String message="hello world";
            /**
             * 发送一个消息
             * 1.发送到那个交换机（hello world使用默认交换机，不指定交换机）
             * 2.路由的 routing-key 是哪个（hello world模式使用的是队列名，其他是routing-key）
             * 3.其他的参数信息（消息的参数，如：消息的过期时间等消息特性）
             * 4.发送消息的消息体
             */
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            System.out.println("消息发送完毕");
        }
    }
}
```

#### 2.1.2 消费者

```java
public class Consumer {
    /**
     * 队列
     */
    private final static String QUEUE_NAME = "hello";
    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        factory.setUsername("guest");
        factory.setPassword("guest");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        System.out.println("等待接收消息......");
        // 推送的消息如何进行消费的接口回调
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody());
            System.out.println(message);
        };
        // 取消消费的一个回调接口 如在消费的时候队列被删除掉了
        CancelCallback cancelCallback = (consumerTag) -> {
            System.out.println("消息消费被中断");
        };
        /**
         * 消费者消费消息
         * 1.消费哪个队列
         * 2.消费成功之后是否要自动应答 true 代表自动应答 false 手动应答
         * 3.消费者未成功消费的回调
         */
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
    }
}
```

### 3.2 Work Queues 任务队列

![image-20220403232651876](assets/image-20220403232651876.png)

>   工作队列(又称任务队列)的主要思想是避免立即执行资源密集型任务，即生产者生产的消息消费者可以不必立即消费处理。
>
>   任务队列模型，让`多个消费者`绑定到`一个队列`，共同消费队列中的消息。
>
>   按顺序将每个消息发送给下一个消费者。平均而言，每个消费者都会收到相同数量的消息（轮询方式）。

#### 3.2.1 轮询分发（公平分发）

>   该模式下，生产者发送消息到queue中，队列queue中的消息轮询分发给消费者消费，消费者均匀处理消息。
>
>   *   一个队列，多个消费者
>
>   *   使用默认交换机，不需要绑定
>
>   **ps：**不公平分发需要预取值设置channel.basicQos(1)，通常不公平分发需要手动确认支持，自动确认发送完就任务确认了。

```java
public class RabbitMqUtils {
    //得到一个连接的 channel
    public static Channel getChannel() throws Exception{
        //创建一个连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        factory.setUsername("guest");
        factory.setPassword("guest");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        return channel;
    }
}

```

**消息生产者**

```java
public class Task01 {
    // 队列
    private static final String QUEUE_NAME="hello";
    public static void main(String[] args) throws Exception {
        try(Channel channel=RabbitMqUtils.getChannel();) {
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            // 从控制台当中接受信息
            Scanner scanner = new Scanner(System.in);
            while (scanner.hasNext()){
                String message = scanner.next();
                channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
                System.out.println("发送消息完成:"+message);
            }
        }
    }
}
```

**消息消费者**: 启动两个实例

```java
public class Worker01 {
    private static final String QUEUE_NAME="hello";
    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String receivedMessage = new String(delivery.getBody());
            System.out.println("接收到消息:" + receivedMessage);
        };
        CancelCallback cancelCallback = (consumerTag) -> {
            System.out.println(consumerTag + "消费者取消消费接口回调逻辑");
        };
        System.out.println("C2 消费者启动等待消费..................");
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
    }
}
```

### 3.3 Publish/Subscribe fanout模式

>   fanout类型的发布订阅模式。在该模式下，可以有`多个消费者`，`多个队列`，每个消费者有自己的队列queue（消费者订阅队列），每个队列queue又绑定交换机exchange，但不需要指定routing-key。`参考fanout交换机`.
>
>   *   消费者与队列绑定：
>   *   队列与交换机绑定 ： channel.queueBind(queueName, EXCHANGE_NAME, "")
>   *   生产者发布消息不需要指定队列或者routing-key
>   *   交换机把消息发送给绑定过的所有队列
>
>   队列的消费者都能拿到消息，实现一条消息被多个消费者消费，参考faout交换机。
>
>   ```java
>   Exchange.DeclareOk exchangeDeclare(String exchange, String type);
>   // 队列和交换机绑定，在fantou模型下不需要routing-key
>   Queue.BindOk queueBind(String queue, String exchange, String routingKey) 
>   ```

**生产者**：生产者声明交换机

```java
//声明交换机
channel.exchangeDeclare("logs","fanout");//广播 一条消息多个消费者同时消费
//发布消息
channel.basicPublish("logs","",null,"hello".getBytes());
// 交换机绑定队列
channel.queueBind(queueName, EXCHANGE_NAME, "");
// 发布消息，无需指定queue
channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes("UTF-8"));
	System.out.println("生产者发出消息" + message);
}
```

**消费者**

```java
//绑定交换机
channel.exchangeDeclare("logs","fanout");
//创建临时队列
String queue = channel.queueDeclare().getQueue();
//将临时队列绑定exchange
channel.queueBind(queue,"logs","");
//处理消息
channel.basicConsume(queue,true,new DefaultConsumer(channel){
  @Override
  public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
    System.out.println("消费者2: "+new String(body));
  }
});
```

### 3.5 Routing

>   Direct指令类型的订阅模型，`参考routing exchange`。该模式下，消息被不同的队列消费。
>
>    在Direct模型下：
>
>   *   队列与交换机的绑定，不能是任意绑定了，而是要指定一个`RoutingKey`
>   *   生产者发送消息时，必须指定消息的 `RoutingKey`
>   *   Exchange根据消息的`Routing Key`将消息路由到不同的队列

**生产者**

```java
//声明交换机  参数1:交换机名称 参数2:交换机类型 基于指令的Routing key转发
channel.exchangeDeclare("logs_direct","direct");
String key = "";
//发布消息
channel.basicPublish("logs_direct",key,null,("指定的route key"+key+"的消息").getBytes());
```

**消费者**

```java
 //声明交换机
channel.exchangeDeclare("logs_direct","direct");
//创建临时队列
String queue = channel.queueDeclare().getQueue();
//绑定队列和交换机
channel.queueBind(queue,"logs_direct","error");
channel.queueBind(queue,"logs_direct","info");
channel.queueBind(queue,"logs_direct","warn");

//消费消息
channel.basicConsume(queue,true,new DefaultConsumer(channel){
  @Override
  public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
    System.out.println("消费者1: "+new String(body));
  }
});
```

### 3.6 topic

>   主题模式，可参考`topic交换机`。`Topic`类型`Exchange`可以让队列在绑定`Routing key` 的时候使用通配符！这种模型`Routingkey` 一般都是由一个或多个单词组成，多个单词之间以”.”分割，例如： `item.insert`
>
>   除了routing-key的模式与Routing不同之外，其他都差不多。

```markdown
# 通配符
* ：匹配一个字符
# ：匹配一个或多个词
```



### 3. 消息应答

>   消息应答是保证消息不丢失的保障手段之一，是消费者消费消费消息成功之后通知broker。
>
>   **queue <-> consumer**
>
>   注意：RabbitMQ 默认一旦向消费者传递了一条消息，便立即将该消息标记为删除，若消息在传递过程中或者消费端处理过程中，消费者挂掉，消息就会丢失。
>
>   rabbitmq 引入消息应答机制，消费者在接收到消息并且处理该消息之后，反馈broker一个ack确认信息。

####  3..1 应答方法

*   Channel.basicAck：肯定确认，RabbitMQ 已知道该消息并且成功的处理消息，可以将其丢弃，消费者调用。
*   Channel.basicNack：否定确认，消费者调用。
*   Channel.basicReject：否定确认，不处理该消息了直接拒绝，可以将其丢弃。

**确认方法的multiple 参数解释 **

![image-20220330223147751](assets/image-20220330223147751.png)

```java
channel.basicAck(deliveryTag, true);
```

*   true：代表批量应答，如：channel 上有传送 tag 的消息 5,6,7,8 当前 tag 是8 那么此时 5-8 的这些还未应答的消息都会被确认收到消息应答。
*   false：tag是谁就应答谁的，如上只会应答tag = 8的消息。

![image-20220330223357093](assets/image-20220330223357093.png)

#### 3.3.2 消息重新入队

>   消费者由于某些原因，导致消息未发送 ACK 确认，RabbitMQ 将消息未完全处理的消息重新排队，确保不会丢失任何消息。
>
>   可以存在幂等性问题，即消息重复消费，可使用唯一id等解决，详细看补充知识。

![image-20220330223656225](assets/image-20220330223656225.png)

#### 3.3.3 自动应答

>   生产者发送消息后立即被认为已经传送成功。
>
>   *   存在消息丢失
>   *   没有对传递的消息数量进行限制（消费者来不及处理，导致消息积压，内存耗尽）

#### 3.3.4 手动应答

>   默认消息采用的是自动应答，需要把自动应答改为手动应答，稍加修改消费者代码即可。
>
>   ```java
>   channel.basicConsume(ACK_QUEUE_NAME, autoAck, deliverCallback, consumer);
>   ```

**消息生产者**: 代码不变

```java
public class Task02 {
    private static final String TASK_QUEUE_NAME = "ack_queue";
        public static void main(String[] argv) throws Exception {
            try (Channel channel = RabbitMqUtils.getChannel()) {
                channel.queueDeclare(TASK_QUEUE_NAME, false, false, false, 
                null);Scanner sc = new Scanner(System.in);
                System.out.println("请输入信息");
                while (sc.hasNext()) {
                String message = sc.nextLine();
                    channel.basicPublish("", TASK_QUEUE_NAME, null, message.getBytes("UTF-8"));
                	System.out.println("生产者发出消息" + message);
            }
        }
    }
}
```

**消费者01**: 修改代码，在消费消息的方法中采用手动应答，即自动应答为false

```java
public class Work03 {
    private static final String ACK_QUEUE_NAME="ack_queue";
    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        System.out.println("C1 等待接收消息处理时间较短");
        //消息消费的时候如何处理消息
         DeliverCallback deliverCallback=(consumerTag,delivery -> {
            String message= new String(delivery.getBody());
        	System.out.println("接收到消息:"+message);
            /**
            * 1.消息标记 tag
            * 2.是否批量应答未应答消息
             */
        	channel.basicAck(delivery.getEnvelope().getDeliveryTag(),false);
        };
        // 采用手动应答
        boolean autoAck = false;
        channel.basicConsume(ACK_QUEUE_NAME, autoAck, deliverCallback, (consumerTag) -> {
        	System.out.println(consumerTag+"消费者取消消费接口回调逻辑");
        });
    }
}
```

**消费者02**

```java
public class Work03 {
    private static final String ACK_QUEUE_NAME="ack_queue";
    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        System.out.println("C1 等待接收消息处理时间较短");
        //消息消费的时候如何处理消息
         DeliverCallback deliverCallback=(consumerTag,delivery -> {
            String message= new String(delivery.getBody());
        	System.out.println("接收到消息:"+message);
            /**
            * 1.消息标记 tag
            * 2.是否批量应答未应答消息
             */
        	channel.basicAck(delivery.getEnvelope().getDeliveryTag(),false);
        };
        // 采用手动应答
        boolean autoAck = false;
        channel.basicConsume(ACK_QUEUE_NAME, autoAck, deliverCallback, (consumerTag) -> {
        	System.out.println(consumerTag+"消费者取消消费接口回调逻辑");
        });
    }
}
```

#### 3.3.5 不公平分发

>   自动确认的轮训分发就是公平分发，为了实现不公平分发。需要设值参数
>
>   ```java
>   channel.basicQos(1)
>   ```
>
>   即：这个任务我还没有处理完或者我还没有应答你，你先别分配给我，我目前只能处理一个任务。
>
>   通常还需要手动确认的支持。
>
>   缺点：处理慢，队列可能被撑满

**补充：**

预取值：限制缓冲区的大小，以避免缓冲区里面无限制的未确认消息问题。



## 4，持久化

### 4.1 消息持久化

>   消息持久化是保障消息不丢失的手段之一，但不能完全保证不会丢失消息。如：刚准备存储在磁盘的时候，但是还没有存储完，broker宕机了。
>
>   实现消息持久化需要在生产者代码修改：
>
>   MessageProperties.PERSISTENT_TEXT_PLAIN

![](assets/image-20220330225931172.png)

### 4.2 队列持久化

>   队列持久化，在broker重启之后，队列仍然存在，只需要在声明队列的时候指定该队列是持久化队列即可。

```java
// 声明消息队列，且为可持久化的
channel.queueDeclare(queue_name, durable, false, false, null);
```



## 5，发布确认

发布确认是保障消息不丢失的保障，但是发布确认默认是没有开启的，需要手动设置发布确认(`调用方法`)。

**produce <--> exchange之间的确认，确保发布的消息不丢失**

### 5.1 普通发布确认

```java
Channel channel = connection.createChannel();
// 开启手动发布确认
channel.confirmSelect();
```

#### 单个确认

>   单个发布确认是`同步确认发布`的方式，`waitForConfirmsOrDie(long)`方法只有在消息被确认的时候才返回，如果在指定时间范围内这个消息没有被确认那么它将抛出异常。
>
>   **缺点：**
>
>   *   发布速度特别的慢（只有当上一个消息被确认了，当前消息才能发布）

```java
public static void publishMessageIndividually() throws Exception {
	try (Channel channel = RabbitMqUtils.getChannel()) {
        String queueName = UUID.randomUUID().toString();
        channel.queueDeclare(queueName, false, false, false, null);
        // 开启发布确认
        channel.confirmSelect();
        long begin = System.currentTimeMillis();
        for (int i = 0; i < MESSAGE_COUNT; i++) {
            String message = i + "";
        	channel.basicPublish("", queueName, null, message.getBytes());
        	// 服务端返回 false 或超时时间内未返回，生产者可以消息重发
         	boolean flag = channel.waitForConfirms();
            if(flag){
                System.out.println("消息发送成功");
            }
        }
        long end = System.currentTimeMillis();
        System.out.println("发布" + MESSAGE_COUNT + "个单独确认消息,耗时" + (end - begin) + "ms");
    }
}
```

#### 批量确认

>   先发布一批消息，然后一起确认(调用确认方法`channel.waitForConfirms()`)，可以极大地提高吞吐量。
>
>   **缺点：**
>
>   *   当发生故障时，导致发布出现问题，不知道是哪个消息出现问题了，所以必须记录记录重要的信息而后重新发布这些失败的消息（避免消息重复消费）。

```java
public static void publishMessageBatch() throws Exception {
        try (Channel channel = RabbitMqUtils.getChannel()) {
            String queueName = UUID.randomUUID().toString();
            channel.queueDeclare(queueName, false, false, false, null);
            // 开启发布确认
            channel.confirmSelect();
            // 批量确认消息大小
            int batchSize = 100;
            // 未确认消息个数
            int outstandingMessageCount = 0;
            long begin = System.currentTimeMillis();
            for (int i = 0; i < 200; i++) {
                String message = i + "";
                channel.basicPublish("", queueName, null, message.getBytes());
                outstandingMessageCount++;
                if (outstandingMessageCount == batchSize) {
                    channel.waitForConfirms();
                    outstandingMessageCount = 0;
                }
            }
            // 为了确保还有剩余没有确认消息 再次确认
            if (outstandingMessageCount > 0) {
                channel.waitForConfirms();
            }
            long end = System.currentTimeMillis();
            System.out.println("发布" + 100 + "个批量确认消息,耗时" + (end - begin) + "ms");
        }
    }
```

####  异步确认

>   异步确认虽然编程逻辑比上两个要复杂，但是性价比最高，利用回调函数来实现消息可靠性传递。
>
>   **重要方法：**添加回调函数
>
>   ```java
>   ConfirmListener addConfirmListener(ConfirmCallback ackCallback, ConfirmCallback nackCallback);
>   ```
>
>   

**实现原理：**

![image-20220331232446205](assets/image-20220331232446205.png)

```java
public static void publishMessageAsync() throws Exception {
        try (Channel channel = RabbitMqUtils.getChannel()) {
            String queueName = UUID.randomUUID().toString();
            channel.queueDeclare(queueName, false, false, false, null);
            // 开启发布确认
            channel.confirmSelect();
            /**
             * 线程安全有序的一个哈希表，适用于高并发的情况
             * 1.轻松的将序号与消息进行关联
             * 2.轻松批量删除条目 只要给到序列号
             * 3.支持并发访问
             */
            ConcurrentSkipListMap<Long, String> outstandingConfirms = new ConcurrentSkipListMap<>();
            /**
             * 确认收到消息的一个回调
             * 1.消息序列号
             * 2.true 可以确认小于等于当前序列号的消息
             * false 确认当前序列号消息
             */
            ConfirmCallback ackCallback = (sequenceNumber, multiple) -> {
                if (multiple) {
                    // 返回的是小于等于当前序列号的未确认消息 是一个 map
                    ConcurrentNavigableMap<Long, String> confirmed = outstandingConfirms.headMap(sequenceNumber, true);
                    // 清除该部分未确认消息
                    confirmed.clear();
                } else {
                    // 只清除当前序列号的消息
                    outstandingConfirms.remove(sequenceNumber);
                }
            };
            ConfirmCallback nackCallback = (sequenceNumber, multiple) -> {
                String message = outstandingConfirms.get(sequenceNumber);
                System.out.println("发布的消息" + message + "未被确认，序列号" + sequenceNumber);
            };
            /**
             * 添加一个异步确认的监听器
             * 1.确认收到消息的回调
             * 2.未收到消息的回调
             */
            channel.addConfirmListener(ackCallback, null);
            long begin = System.currentTimeMillis();
            for (int i = 0; i < 500; i++) {
                String message = "消息" + i;
                /**
                 * channel.getNextPublishSeqNo()获取下一个消息的序列号
                 * 通过序列号与消息体进行一个关联
                 * 全部都是未确认的消息体
                 */
                outstandingConfirms.put(channel.getNextPublishSeqNo(), message);
                channel.basicPublish("", queueName, null, message.getBytes());
            }
            long end = System.currentTimeMillis();
            System.out.println("发布" + 5000 + "个异步确认消息,耗时" + (end - begin) + "ms");
        }
    }
```

**如何处理异步未确认消息?**

​		最好的解决的解决方案就是把未确认的消息放到一个基于内存的，能被发布线程访问的队列， 比如说用 ConcurrentLinkedQueue 这个队列在 confirm callbacks 与发布线程之间进行消息的传递。

### 5.2 高级发布确认

>   同样是生产者和交换机的确认机制，在生产环境中由于某些原因，导致 rabbitmq 重启，在重启期间生产者消息投递失败， 导致消息丢失，需要手动处理和恢复。如何才能保证消息可靠投递呢？

#### 确认机制方案

![image-20220402224937457](assets/image-20220402224937457.png)

#### 发布确认 springboot 版本

##### 配置文件

```properties
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
spring.rabbitmq.publisher-confirm-type=correlated # NONE:禁用发布确认模式，是默认值；CORRELATED：发布消息成功到交换器后会触发回调方法；SIMPLE：
```

##### 配置类

```java
@Configuration
public class ConfirmConfig {
    public static final String CONFIRM_EXCHANGE_NAME = "confirm.exchange";
    public static final String CONFIRM_QUEUE_NAME = "confirm.queue";
    //声明业务 Exchange
    @Bean("confirmExchange")
    public DirectExchange confirmExchange(){
    	return new DirectExchange(CONFIRM_EXCHANGE_NAME);
    }
    // 声明确认队列 queue
    @Bean("confirmQueue")
    public Queue confirmQueue(){
   		return QueueBuilder.durable(CONFIRM_QUEUE_NAME).build();
    }
    // 声明确认队列绑定关系
    @Bean
    public Binding queueBinding(@Qualifier("confirmQueue") Queue queue, @Qualifier("confirmExchange") DirectExchange exchange){
    	return BindingBuilder.bind(queue).to(exchange).with("key1");
    }
}
```

##### 生产者

>   rabbitTemplate.convertAndSend：发送消息
>
>   CorrelationData：消息相关

```java
@RestController
@RequestMapping("/confirm")
public class Producer {
    public static final String CONFIRM_EXCHANGE_NAME = "confirm.exchange";
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @Autowired
    private MyCallBack myCallBack;

    //依赖注入 rabbitTemplate 之后再设置它的回调对象
    @PostConstruct
    public void init(){
        rabbitTemplate.setConfirmCallback(myCallBack);
    }

    @GetMapping("sendMessage/{message}")
    public void sendMessage(@PathVariable String message){
        // 指定消息 id 为 1
        CorrelationData correlationData1 = new CorrelationData("1");
        String routingKey="key1";
        rabbitTemplate.convertAndSend(CONFIRM_EXCHANGE_NAME,routingKey,message+routingKey,correlationData1);
        CorrelationData correlationData2=new CorrelationData("2");
        routingKey="key2";
        rabbitTemplate.convertAndSend(CONFIRM_EXCHANGE_NAME,routingKey,message+routingKey,correlationData2);
    }
}
```

##### 回调接口

>   所有发布确认回调接口

```java
@Component
@Slf4j
public class MyCallBack implements RabbitTemplate.ConfirmCallback {
    /**
    * 交换机不管是否收到消息的一个回调方法
     * CorrelationData
    * 消息相关数据
     * ack
    * 交换机是否收到消息
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        String id = correlationData != null ? correlationData.getId() : "";
        if(ack){
        	log.info("交换机已经收到 id 为:{}的消息",id);
        }else{
        	log.info("交换机还未收到 id 为:{}消息,由于原因:{}",id,cause);
        }
    }
}

```

##### 消费者

```java
@Component
@Slf4j
public class ConfirmConsumer {
    public static final String CONFIRM_QUEUE_NAME = "confirm.queue";
    @RabbitListener(queues =CONFIRM_QUEUE_NAME)
    public void receiveMsg(Message message) { 
        String msg=new String(message.getBody());
    	log.info("接受到队列 confirm.queue 消息:{}",msg);
    }
}
```

### 5.3 消息回退

>   在仅开启了生产者确认机制的情况下，交换机接收到消息后，会直接给消息生产者发送确认消息，如果发现该消息不可路由，那么消息会被直接丢弃，此时生产者是不知道消息被丢弃这个事件的。通过设置 **mandatory** 参数可以在当消息传递过程中不可达目的地时将消息返回给生产者。
>
>   生产者设置：rabbitTemplate.setMandatory(true);

**生产者**

````java
@Slf4j
@Component
public class MessageProducer implements RabbitTemplate.ConfirmCallback, RabbitTemplate.ReturnCallback {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    // rabbitTemplate 注入之后就设置该值
    @PostConstruct
    private void init () {
        rabbitTemplate.setConfirmCallback(this);
        /**
         * true：交换机无法将消息进行路由时，会将该消息返回给生产者
         * false：如果发现消息无法进行路由，则直接丢弃
         */
        rabbitTemplate.setMandatory(true);
        // 设置回退消息交给谁处理
        rabbitTemplate.setReturnCallback(this);
    }
    @GetMapping("sendMessage")
    public void sendMessage (String message){
        // 让消息绑定一个 id 值
        CorrelationData correlationData1 = new CorrelationData(UUID.randomUUID().toString());
        rabbitTemplate.convertAndSend("confirm.exchange", "key1", message + "key1", correlationData1);
        log.info("发送消息 id 为:{}内容为{}", correlationData1.getId(), message + "key1");
        CorrelationData correlationData2 = new CorrelationData(UUID.randomUUID().toString());
        rabbitTemplate.convertAndSend("confirm.exchange", "key2", message + "key2", correlationData2);
        log.info("发送消息 id 为:{}内容为{}", correlationData2.getId(), message + "key2");
    }
    @Override
    public void confirm (CorrelationData correlationData,boolean ack, String cause) {
        String id = correlationData != null ? correlationData.getId() : "";
        if (ack) {
            log.info("交换机收到消息确认成功, id:{}", id);
        } else {
            log.error("消息 id:{}未成功投递到交换机,原因是:{}", id, cause);
        }
    }
    @Override
    public void returnedMessage (Message message,int replyCode, String replyText, String exchange, String routingKey) {
        log.info("消息:{}被服务器退回，退回原因:{}, 交换机是:{}, 路由 key:{}", 
            new String(message.getBody()), replyText, exchange, routingKey);
    }
}
````

**回调接口**

```java
@Component
@Slf4j
public class MyCallBack implements
    RabbitTemplate.ConfirmCallback,RabbitTemplate.ReturnCallback {
    /**
     * 交换机不管是否收到消息的一个回调方法
     * CorrelationData
     * 消息相关数据
     * ack
     * 交换机是否收到消息
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        String id=correlationData!=null?correlationData.getId():"";
        if(ack){
            log.info("交换机已经收到 id 为:{}的消息",id);
        }else{
            log.info("交换机还未收到 id 为:{}消息,由于原因:{}",id,cause);
        }
    }
    //当消息无法路由的时候的回调方法
    @Override
    public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey) {
        log.error(" 消 息 {}, 被 交 换 机 {} 退 回 ， 退 回 原 因 :{}, 路 由 key:{}",new 
            String(message.getBody()),exchange,replyText,routingKey);
    }
}
```



## 6，交换机

>   RabbitMQ 消息传递模型的核心思想是: `生产者生产的消息从不会直接发送到队列`。实际上，通常生产者甚至都不知道这些消息传递传递到了哪些队列中，hello world模式除外。
>
>   相反，生产者只能将消息发送到交换机(exchange)。
>
>   交换机主要是接收生产者生产的消息和根据规则推送消息到队列中，或者舍弃。
>
>   交换机根据routing-key和队列queue绑定，生产者需要告诉交换机队列或者routing-key。
>
>   消息能路由发送到队列中其实 是由 routingKey(bindingkey)绑定 key 指定的。

![image-20220331234216908](assets/image-20220331234216908.png)

**交换机类型：**

*   直接(direct)
*   扇出(fanout)，就是广播
*   主题(topic)
*   ~~标题(headers)~~ 
*   （补充：默认或者说无名交换机，其不属于交换机类型）

### 6.1 默认交换机和临时队列

#### 默认交换机

前面部分我们对 exchange 一无所知，但仍然能够将消息发送到队列。之前能实现的原因是因为使用了默认交换机，通过空字符串“”进行标识。

**默认交换机时，消息生产者使用的队列名，使用其他交换机生产者使用的是routing-key**

#### 临时队列

​		临时队列，一旦我们断开了消费者的连接，队列将被自动删除。

**创建临时队列：**

```java
String queueName = channel.queueDeclare().getQueue();
```

### 6.2 绑定bindings

>   binding 其实是 exchange 和 queue 之间的桥梁，告诉我们 exchange 和哪个队列进行了绑定关系。
>
>   绑定关系通过routing-key来表明，在声明队列的时候绑定交换机exchange。

![image-20220331235418304](assets/image-20220331235418304.png)

### 6.3 Direct exchange

>   直连交换机模式，消息只去到它绑定的` routingKey` 队列中去。
>
>   如：交换机根据消息的routing-key，把消息路由到自己绑定的特定交换机，也只能路由到一个队列中，是一种完全匹配，如果多个switch和queue的routing-key都一样就会回到fanout模式。

![image-20220331235859632](assets/image-20220331235859632.png)

**多次绑定**

![image-20220401000158572](assets/image-20220401000158572.png)

>    exchange 绑定类型是direct，但是它绑定的多个队列的 key 如果都相同，在这种情况下 direct 和 fanout 有点类似。

**案例代码**

![image-20220401000318321](assets/image-20220401000318321.png)

**消费者1**

```java
public class ReceiveLogsDirect01 {
    private static final String EXCHANGE_NAME = "direct_logs";
    public static void main(String[] argv) throws Exception {
        Channel channel = RabbitUtils.getChannel();
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
        String queueName = "disk";
        channel.queueDeclare(queueName, false, false, false, null);
        channel.queueBind(queueName, EXCHANGE_NAME, "error");
        System.out.println("等待接收消息........... ");
        DeliverCallback deliverCallback = (consumerTag, delivery) ->
        {String message = new String(delivery.getBody(), "UTF-8");
            message="接收绑定键:"+delivery.getEnvelope().getRoutingKey()+",消息:"+message;
            File file = new File("C:\\work\\rabbitmq_info.txt");
            FileUtils.writeStringToFile(file,message,"UTF-8");
            System.out.println("错误日志已经接收");
        };
        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> {
        });
    }
}
```

**消费者2**

```java
public class Producer {
    private static final String EXCHANGE_NAME = "direct_logs";
    public static void main(String[] argv) throws Exception {
        Channel channel = RabbitUtils.getChannel();
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
        String queueName = "console";
        channel.queueDeclare(queueName, false, false, false, null);
        channel.queueBind(queueName, EXCHANGE_NAME, "info");
        channel.queueBind(queueName, EXCHANGE_NAME, "warning");
        System.out.println("等待接收消息........... ");
        DeliverCallback deliverCallback = (consumerTag, delivery) ->
        {String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" 接收绑定键 :"+delivery.getEnvelope().getRoutingKey()+", 消息:" + message);
        };
        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> {
        });
    }
}
```

**生产者**

```java
public class Producer {
    private static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] args) throws Exception {
        try (Channel channel = RabbitUtils.getChannel()) {
            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
            // 创建多个 bindingKey
            Map<String, String> bindingKeyMap = new HashMap<>();
            bindingKeyMap.put("info", "普通 info 信息");
            bindingKeyMap.put("warning", "警告 warning 信息");
            bindingKeyMap.put("error", "错误 error 信息");
            // debug 没有消费这接收这个消息 所有就丢失了
            bindingKeyMap.put("debug", "调试 debug 信息");
            for (Map.Entry<String, String> bindingKeyEntry : bindingKeyMap.entrySet()) {
                String bindingKey = bindingKeyEntry.getKey();
                String message = bindingKeyEntry.getValue();
                channel.basicPublish(EXCHANGE_NAME, bindingKey, null, message.getBytes(StandardCharsets.UTF_8));
                System.out.println("生产者发出消息:" + message);
            }
        }
    }
}
```

### 6.4 Fanout exchange

>   Fanout exchange散出交换机，它是将接收到的所有消息广播到它知道的（绑定）所有队列中，这个时候routing-key没有任何意义，不需要指定。
>
>   fanout 模式下，生产者发送消息只需要指定交换机即可，交换机会根据根据绑定关系广播到其绑定的队列中。
>
>   swtich和queue任然存在绑定关系。

![image-20220401193457026](assets/image-20220401193457026.png)

**消费者1：**

```java
public class ReceiveLogs01 {
    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] argv) throws Exception {
        Channel channel = RabbitUtils.getChannel();
        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
        /**
         * 生成一个临时的队列 队列的名称是随机的
         * 当消费者断开和该队列的连接时 队列自动删除
         */
        String queueName = channel.queueDeclare().getQueue();
        // 把该临时队列绑定我们的 exchange 其中 routing key(也称之为 binding key)为空字符串
        channel.queueBind(queueName, EXCHANGE_NAME, "");
        System.out.println("等待接收消息,把接收到的消息打印在屏幕........... ");
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println("控制台打印接收到的消息" + message);
        };
        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> {
            // ...
        });
    }
}
```

**消费者2：**

```java
public class ReceiveLogs02 {
    private static final String EXCHANGE_NAME = "logs";
    public static void main(String[] args) throws Exception {
        Channel channel = RabbitUtils.getChannel();
        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
        /**
         * 生成一个临时的队列 队列的名称是随机的
         * 当消费者断开和该队列的连接时 队列自动删除
         */
        String queueName = channel.queueDeclare().getQueue();
        // 把该临时队列绑定我们的 exchange 其中 routingkey(也称之为 binding key)为空字符串
        channel.queueBind(queueName, EXCHANGE_NAME, "");
        System.out.println("等待接收消息,把接收到的消息写到文件........... ");
        DeliverCallback deliverCallback = (consumerTag, delivery) ->
        {
            String message = new String(delivery.getBody(), "UTF-8");
            File file = new File("C:\\work\\rabbitmq_info.txt");
            FileUtils.writeStringToFile(file, message, "UTF-8");
            System.out.println("数据写入文件成功");
        };
        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> {
        });
    }
}
```

**生产者：**

```java
public class EmitLog {
    private static final String EXCHANGE_NAME = "logs";
    public static void main(String[] argv) throws Exception {
        try (Channel channel = RabbitUtils.getChannel()) {
            /**
            * 声明一个 exchange
            * 1.exchange 的名称
             * 2.exchange 的类型
             */
            channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
            Scanner sc = new Scanner(System.in);
            System.out.println("请输入信息");
            while (sc.hasNext()) {
            String message = sc.nextLine();
            channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes("UTF-8"));
            	System.out.println("生产者发出消息" + message);
            }
        }
    }
}
```

### 6.5 Topic exchange

>   是 topic 交换机的消息的 routing_key可以模糊匹配队列queue，但需满足，它必须是一个单 词列表，以点号分隔开。

**通配符：**

*   *(星号)可以代替一个单词 
*   #(井号)可以替代零个或多个单词

**案例代码：**省略，主要是routing-key

![image-20220401200137233](assets/image-20220401200137233.png)

```java
// bindingKey: quick.orange.rabbit,lazy.orange.elephant,quick.orange.foxd。。。
channel.basicPublish(EXCHANGE_NAME,bindingKey, null, message.getBytes("UTF-8"));
```

### 6.6 备份交换机

>   前面介绍了消息回退，有了 mandatory 参数和回退消息，可以感知有消息无法投递的能力，有机会在生产者的消息无法被投递时发现并处理。
>
>   什么是备份交换机呢？
>
>   备份交换机可以理解为 RabbitMQ 中交换机的“备胎”，当交换机接收到一条不可路由消息时，将会把这条消息转发到备份交换机中，由备份交换机来进行转发和处理，通常备份交换机的类型为 Fanout。

![image-20220403103142485](assets/image-20220403103142485.png)

**修改配置类：**声明确认 Exchange 交换机的备份交换机

```java
//声明备份 Exchange
@Bean("backupExchange")
public FanoutExchange backupExchange(){
	return new FanoutExchange(BACKUP_EXCHANGE_NAME);
}
// 声明备份队列
 @Bean("backQueue")
public Queue backQueue(){
	return QueueBuilder.durable(BACKUP_QUEUE_NAME).build();
}

// 声明备份队列绑定关系
@Bean
public Binding backupBinding(@Qualifier("backQueue") Queue queue, @Qualifier("backupExchange") FanoutExchange backupExchange){
	return BindingBuilder.bind(queue).to(backupExchange);
}

// 声明确认 Exchange 交换机的备份交换机
@Bean("confirmExchange")
public DirectExchange 
confirmExchange(){
    ExchangeBuilder exchangeBuilder = ExchangeBuilder.directExchange(CONFIRM_EXCHANGE_NAME).durable(true)
		//设置该交换机的备份交换机
 		.withArgument("alternate-exchange", BACKUP_EXCHANGE_NAME);
	return (DirectExchange)exchangeBuilder.build();
}
```

## 7，死信队列

>   ​	  死信，顾名思义就是无法被消费的消息。
>
>   ​	  有时候由于特定的原因导致 queue 中的某些消息无法被消费，如果没有后续的处理，就变成了死信，有死信自然就有了死信队列。
>
>   死信队列也是队列，和普通的队列差不多。

****

**应用场景**

*   消息消费失败的后续处理

### 7.1 死信来源

*   消息 TTL 过期（消息过期，需要设置消息过期时间）
*   队列已满（队列无法在接收新任务，需要设置队列长度）
*   消息被拒绝（消息被消费者拒绝，如：(basic.reject 或 basic.nack）

### 7.2 TTL过期

#### 7.2.1 设置TTL

要模拟消息TTL过期，首先设置消息的TTL过期时间，然后关掉消费者让消息过期。

```java
// 设置消息的 TTL 时间
AMQP.BasicProperties properties =new AMQP.BasicProperties().builder().expiration("10000").build();
channel.basicPublish(NORMAL_EXCHANGE, "routing-key or queue", properties, message.getBytes());
```

##### 7.2.2 设置死信队列

声明交换机和队列（队列是特殊的队列，需要参数来设置），绑定他们的关系

```java
//声明死信和普通交换机 类型为 direct
channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);
// 声明死信队列
String deadQueue = "dead-queue";
channel.queueDeclare(deadQueue, false, false, false, null);
// 死信队列绑定死信交换机与 routingkey
channel.queueBind(deadQueue, DEAD_EXCHANGE, "lisi");
//正常队列绑定死信队列信息
Map<String, Object> params = new HashMap<>();
//正常队列设置死信交换机 参数 key 是固定值
params.put("x-dead-letter-exchange", DEAD_EXCHANGE);
//正常队列设置死信 routing-key 参数 key 是固定值
params.put("x-dead-letter-routing-key", "lisi");
String normalQueue = "normal-queue";
channel.queueDeclare(normalQueue, false, false, false, params);
channel.queueBind(normalQueue, NORMAL_EXCHANGE, "zhangsan");
```



**生产者：**

```java
public class Producer {
    private static final String NORMAL_EXCHANGE = "normal_exchange";
    public static void main(String[] args) throws Exception {
        try (Channel channel = RabbitMqUtils.getChannel()) {
            channel.exchangeDeclare(NORMAL_EXCHANGE,
            BuiltinExchangeType.DIRECT);
            // 设置消息的 TTL 时间
            AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().expiration("10000").build();
            // 该信息是用作演示队列个数限制
            for (int i = 1; i < 11 ; i++) {
                String message="info"+i;
                channel.basicPublish(NORMAL_EXCHANGE, "queue" , properties, message.getBytes());
                System.out.println("生产者发送消息:"+message);
            }
        }
    }
}
```

**消费者：**

```java
public class Producer {
    //普通交换机名称
    private static final String NORMAL_EXCHANGE = "normal_exchange";
    //死信交换机名称
    private static final String DEAD_EXCHANGE = "dead_exchange";
    public static void main(String[] argv) throws Exception {
        Channel channel = RabbitUtils.getChannel();
        //声明死信和普通交换机 类型为 direct
        channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
        channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);
        //声明死信队列
        String deadQueue = "dead-queue";
        channel.queueDeclare(deadQueue, false, false, false, null);
        //死信队列绑定死信交换机与 routingkey
        channel.queueBind(deadQueue, DEAD_EXCHANGE, "lisi");
        //正常队列绑定死信队列信息
        Map<String, Object> params = new HashMap<>();
        //正常队列设置死信交换机 参数 key 是固定值
        params.put("x-dead-letter-exchange", DEAD_EXCHANGE);
        //正常队列设置死信 routing-key 参数 key 是固定值
        params.put("x-dead-letter-routing-key", "lisi");
        String normalQueue = "normal-queue";
        channel.queueDeclare(normalQueue, false, false, false, params);
        channel.queueBind(normalQueue, NORMAL_EXCHANGE, "zhangsan");
        System.out.println("等待接收消息........... ");
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println("Consumer01 接收到消息"+message);
        };
        channel.basicConsume(normalQueue, true, deliverCallback, consumerTag -> {
        });
    }
}
```

### 7.3 队列长度

>   代码和上面基本差别多，需要修改队列参数即可。

```java
 params.put("x-max-length", 6);
```

### 7.4 消息被拒

>   代码，参考消息应答

## 8，延迟队列

## 9, 补充知识

### 9.1 幂等性

>   **幂等性**：用户对于同一操作发起的一次请求或者多次请求的结果是一致的，不会因为多次点击而产生了副作用。

`解决方案：` 

在以前的单应用系统中，我们只需要把数据操作放入事务中即可，发生错误立即回滚，但是再响应客户端的时候也有可能出现网络中断或者异常等等

#### 9.1.1 消息重复消费

>   消费者在消费 MQ 中的消息时，MQ 已把消息发送给消费者，消费者在给MQ 返回 ack 时网络中断， 故 MQ 未收到确认信息，该条消息会重新发给其他的消费者或者在网络重连后再次发送给该消费者。
>
>   总结：就是broker没收到consumer的ack确认信息，将消息在发给其他消费者。

`解决方案：`

MQ 消费者的幂等性的解决一般使用全局 ID 或者唯一标识如：uuid，或其他标识，每次消费消息时用该 id 先判断该消息是否已消费过。

业界主流的幂等性有两种操作:

 a. 唯一 ID+指纹码机制,利用数据库主键去重,

 b.利用 redis 的原子性去实现

##### 唯一ID+指纹码机制

>   **指纹码**:我们的一些规则或者时间戳加别的服务给到的唯一信息码,它并不一定是我们系统生成的，基本都是由我们的业务规则拼接而来，但是一定要保证唯一性，然后就利用查询语句进行判断这个 id 是否存在数据库中，，然后查询判断是否重复。
>
>   缺点：高并发时，性能瓶颈

##### Redis 原子性 

>   利用 redis 执行 setnx 命令，天然具有幂等性。从而实现不重复消费

### 9.2 优先队列 

>   给予某些消息以优先处理。
>
>   注意事项：
>
>   要让队列实现优先级需要做的事情有如下事情
>
>   1.   队列需要设置为优先级队列
>   2.   消息需要设置消息的优先级
>   3.   消费者需要等待消息已经发送到队列中才去消，因为这样才有机会对消息进行排序

**控制台添加优先级**

![image-20220403105203073](assets/image-20220403105203073.png)

**队列添加优先级**

声明优先级队列

```java
Map<String, Object> params = new HashMap();
params.put("x-max-priority", 10);
channel.queueDeclare("hello", true, false, false, params);
```

**消息添加优先级**

给消息赋予一个 priority 属性

```java
AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().priority(5).build();
//......
channel.basicPublish("", QUEUE_NAME, properties, message.getBytes());
```

## 10 rabbitmq集群

