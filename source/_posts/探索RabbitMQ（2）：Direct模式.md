---
title: 探索 RabbitMQ（2）：Direct模式
id: explore-rabbitmq-2
date: 2019-03-01 17:55:08
updated: 2019-03-01 17:55:08
categories:
  - 人类的本质是复读机
tags:
  - rabbitmq
---
通常来说，像我这样不爱学习的人，如果开始学习一门新技术，原因如果不是闲的蛋疼（比如Spark...），就一定项目里马上要用到了。

对我来说，先写一个能跑起来的示例一定是最令人安心的。

如果你去搜RabbitMQ示例代码，多半出来的都是告诉你，有四种模式，然后告诉你其中三个咋写（Direct、Topic、Fanout）。

这四个模式是啥意思呀？有啥区别呀？为啥四个模式只给了三个示例呀？老四去哪了呀？有人说了，有人没说，有人啰啰嗦嗦，有人一笔带过。

然而这些对我来说暂时都不重要，毕竟老子写 demo 就是一把梭，上去就是干，有啥抄啥，先敲再看，不挑。

![老夫敲代码就是一把梭](http://cdn.standbyside.com/emoticon/people/coding-as-a-shuttle.png)

<!-- more -->

### 搭建项目

为了使某些配置对项目的影响清晰可见，我把producer和consumer分别写在两个项目中。需要的依赖包除了springboot相关包，主要是spring-boot-starter-amqp

#### producer 项目代码

```java
/**
 * Direct模式生产者.
 */
@Component
public class DirectProducer {

  /**
   * queue.
   */
  private static final String DIRECT_QUEUE_NAME = "test.direct.queue";

  /**
   * 初始化queue.
   */
  @Bean
  public Queue directQueue() {
    return new Queue(DIRECT_QUEUE_NAME);
  }

  @Autowired
  private AmqpTemplate template;

  public void send() {
    String message = "hello world";
    System.out.println("producer send a direct message: " + message);
    template.convertAndSend(DIRECT_QUEUE_NAME, message);
  }
}
```

#### consumer 项目代码

```java
/**
 * Direct模式消费者.
 */
@Component
public class DirectConsumer {

  /**
   * queue.
   */
  private static final String DIRECT_QUEUE_NAME = "test.direct.queue";

  /**
   * 初始化queue.
   */
  @Bean
  public Queue directQueue() {
    return new Queue(DIRECT_QUEUE_NAME);
  }

  /**
   * 监听器监听指定的Queue.
   */
  @RabbitListener(queues = DIRECT_QUEUE_NAME)
  public void receive(String message) {
    System.out.println("consumer receive a direct message: " + message);
  }
}
```

#### 运行结果

至此，一个简单的Direct模式的队列通信就写好了，我们再写一个controller来触发这个消息发送

```java
@RestController
@RequestMapping("/test")
public class TestController {

  @Autowired
  private DirectProducer directProducer;

  @GetMapping("/direct")
  public Object testDirect() {
    directProducer.send();
    return "test direct success";
  }
}
```
运行结果：
> producer send a direct message: hello world

> consumer receive a direct message: hello world

### 问题探索

#### 多试一下能发现，去掉代码中用于初始化queue的directQueue()方法，运行代码也能照常运行，那是不是可以去掉呢？

可以的，但是前提是，rabbitmq的server中已经创建完了该队列

![image](http://cdn.standbyside.com/shortcut/rabbit-mq/exist-queue.jpg)

如果尚未创建队列，会报如下错误：

```
2019-03-01 11:24:14.086  WARN 44361 --- [cTaskExecutor-1] o.s.a.r.listener.BlockingQueueConsumer   : Failed to declare queue: test.direct.queue
2019-03-01 11:24:14.092  WARN 44361 --- [cTaskExecutor-1] o.s.a.r.listener.BlockingQueueConsumer   : Queue declaration failed; retries left=3

org.springframework.amqp.rabbit.listener.BlockingQueueConsumer$DeclarationException: Failed to declare queue(s):[test.direct.queue]
	at org.springframework.amqp.rabbit.listener.BlockingQueueConsumer.attemptPassiveDeclarations(BlockingQueueConsumer.java:711) ~[spring-rabbit-2.0.5.RELEASE.jar:2.0.5.RELEASE]
	at org.springframework.amqp.rabbit.listener.BlockingQueueConsumer.start(BlockingQueueConsumer.java:588) ~[spring-rabbit-2.0.5.RELEASE.jar:2.0.5.RELEASE]
	at org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer$AsyncMessageProcessingConsumer.run(SimpleMessageListenerContainer.java:996) [spring-rabbit-2.0.5.RELEASE.jar:2.0.5.RELEASE]
	at java.lang.Thread.run(Thread.java:748) [na:1.8.0_171]
Caused by: java.io.IOException: null
	at com.rabbitmq.client.impl.AMQChannel.wrap(AMQChannel.java:126) ~[amqp-client-5.1.2.jar:5.1.2]
	at com.rabbitmq.client.impl.AMQChannel.wrap(AMQChannel.java:122) ~[amqp-client-5.1.2.jar:5.1.2]
	at com.rabbitmq.client.impl.AMQChannel.exnWrappingRpc(AMQChannel.java:144) ~[amqp-client-5.1.2.jar:5.1.2]
	at com.rabbitmq.client.impl.ChannelN.queueDeclarePassive(ChannelN.java:991) ~[amqp-client-5.1.2.jar:5.1.2]
	at com.rabbitmq.client.impl.ChannelN.queueDeclarePassive(ChannelN.java:52) ~[amqp-client-5.1.2.jar:5.1.2]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_171]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_171]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_171]
	at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_171]
	at org.springframework.amqp.rabbit.connection.CachingConnectionFactory$CachedChannelInvocationHandler.invoke(CachingConnectionFactory.java:1030) ~[spring-rabbit-2.0.5.RELEASE.jar:2.0.5.RELEASE]
	at com.sun.proxy.$Proxy76.queueDeclarePassive(Unknown Source) ~[na:na]
	at org.springframework.amqp.rabbit.listener.BlockingQueueConsumer.attemptPassiveDeclarations(BlockingQueueConsumer.java:690) ~[spring-rabbit-2.0.5.RELEASE.jar:2.0.5.RELEASE]
	... 3 common frames omitted
Caused by: com.rabbitmq.client.ShutdownSignalException: channel error; protocol method: #method<channel.close>(reply-code=404, reply-text=NOT_FOUND - no queue 'test.direct.queue' in vhost '/', class-id=50, method-id=10)
	at com.rabbitmq.utility.ValueOrException.getValue(ValueOrException.java:66) ~[amqp-client-5.1.2.jar:5.1.2]
	at com.rabbitmq.utility.BlockingValueOrException.uninterruptibleGetValue(BlockingValueOrException.java:36) ~[amqp-client-5.1.2.jar:5.1.2]
	at com.rabbitmq.client.impl.AMQChannel$BlockingRpcContinuation.getReply(AMQChannel.java:494) ~[amqp-client-5.1.2.jar:5.1.2]
	at com.rabbitmq.client.impl.AMQChannel.privateRpc(AMQChannel.java:288) ~[amqp-client-5.1.2.jar:5.1.2]
	at com.rabbitmq.client.impl.AMQChannel.exnWrappingRpc(AMQChannel.java:138) ~[amqp-client-5.1.2.jar:5.1.2]
	... 12 common frames omitted
Caused by: com.rabbitmq.client.ShutdownSignalException: channel error; protocol method: #method<channel.close>(reply-code=404, reply-text=NOT_FOUND - no queue 'test.direct.queue' in vhost '/', class-id=50, method-id=10)
	at com.rabbitmq.client.impl.ChannelN.asyncShutdown(ChannelN.java:504) ~[amqp-client-5.1.2.jar:5.1.2]
	at com.rabbitmq.client.impl.ChannelN.processAsync(ChannelN.java:346) ~[amqp-client-5.1.2.jar:5.1.2]
	at com.rabbitmq.client.impl.AMQChannel.handleCompleteInboundCommand(AMQChannel.java:178) ~[amqp-client-5.1.2.jar:5.1.2]
	at com.rabbitmq.client.impl.AMQChannel.handleFrame(AMQChannel.java:111) ~[amqp-client-5.1.2.jar:5.1.2]
	at com.rabbitmq.client.impl.AMQConnection.readFrame(AMQConnection.java:643) ~[amqp-client-5.1.2.jar:5.1.2]
	at com.rabbitmq.client.impl.AMQConnection.access$300(AMQConnection.java:47) ~[amqp-client-5.1.2.jar:5.1.2]
	at com.rabbitmq.client.impl.AMQConnection$MainLoop.run(AMQConnection.java:581) ~[amqp-client-5.1.2.jar:5.1.2]
	... 1 common frames omitted
```
报错的直接原因是rabbitmq 的server中心<font color="red">上没有名叫"test.direct.queue"的队列</font>，consumer服务启动起来后找不到要监听的队列，因而报错。

那队列是何时创建出来的呢？有三个时间点：

1. producer项目第一次发送消息时，发现server没有这个队列，会创建队列
2. consumer项目启动时，发现server没有这个队列，会创建队列
3. 点击管理界面的Queues面板里的Add queue按钮能随时创建队列

在1和2中，创建队列时，就需要用到初始化queue的代码。在这段代码中，我们可以添加部分参数来描述要创建的队列。

如果server上已经有了该queue，并且代码中也写了初始化queue的方法， 程序则会<font color="red">对比双方参数是否一致</font>，如果初始化方法中设置的队列参数与server上的不一致，则会报参数不一致错误。

#### 如果两个方法监听同一个queue呢？

我们将consumer代码修改如下

```java
/**
 * Direct模式消费者.
 */
@Component
public class DirectConsumer {

  /**
   * queue.
   */
  private static final String DIRECT_QUEUE_NAME = "test.direct.queue";


  /**
   * 初始化queue.
   */
  @Bean
  public Queue directQueue() {
    return new Queue(DIRECT_QUEUE_NAME);
  }

  /**
   * 监听器监听指定的Queue.
   */
  @RabbitListener(queues = DIRECT_QUEUE_NAME)
  public void receive1(String message) {
    System.out.println("consumer receive a direct message 1: " + message);
  }

  /**
   * 监听器监听指定的Queue.
   */
  @RabbitListener(queues = DIRECT_QUEUE_NAME)
  public void receive2(String message) {
    System.out.println("consumer receive a direct message 2: " + message);
  }
}
```

发送10遍请求，consumer控制台打印如下：

> consumer receive a direct message 2: hello world<br/>
consumer receive a direct message 1: hello world<br/>
consumer receive a direct message 2: hello world<br/>
consumer receive a direct message 1: hello world<br/>
consumer receive a direct message 2: hello world<br/>
consumer receive a direct message 1: hello world<br/>
consumer receive a direct message 2: hello world<br/>
consumer receive a direct message 1: hello world<br/>
consumer receive a direct message 2: hello world<br/>
consumer receive a direct message 1: hello world<br/>

可以看出来是呈现了一种<font color="red">轮询式消费</font>，这个给1，那个给2，而不是同一个消息即给1又给2
