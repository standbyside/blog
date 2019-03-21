---
title: 探索RabbitMQ（3）：Topic模式
id: explore-rabbitmq-3
date: 2019-03-21 16:18:14
updated: 2019-03-21 16:18:14
categories:
  - 人类的本质是复读机
tags:
  - rabbitmq
---
差点把分类名改成"人类的本质是咕咕"...

![image](http://cdn.standbyside.com/emoticon/animal/gugu.jpeg)

这次说到平常开发中使用非常常用的，Topic模式。这个模式当时看的也是稀里糊涂，因为很多博客都没讲清楚exchange是什么东西，RoutingKey 和 BindingKey 又有什么联系。直到看了《RabbitMQ实战》和自己的几次尝试，才勉强理清思绪。所以我的建议还是大家多思考，多动手，提出问题，验证问题，才是最快最清晰的。

<!-- more -->

还是从一个最简单的示例开始。

### 示例1

#### producer 项目

```
/**
 * Topic模式生产者.
 */
@Component
public class TopicProducer {

    /**
     * exchange.
     */
    private static final String TOPIC_EXCHANGE_NAME = "spring.rabbit.topic";

    /**
     * queue.
     */
    private static final String TOPIC_QUEUE1_NAME = "test.topic.queue1";

    /**
     * 初始化exchange.
     */
    @Bean(name = "topicExchange")
    public TopicExchange topicExchange() {
        return new TopicExchange(TOPIC_EXCHANGE_NAME);
    }

    /**
     * 初始化queue1.
     */
    @Bean(name = "topicQueue1")
    public Queue topicQueue1() {
        return new Queue(TOPIC_QUEUE1_NAME);
    }

    /**
     * 绑定1.
     */
    @Bean
    Binding bindingTopicMessage1(@Qualifier("topicQueue1") Queue queue,
                                 @Qualifier("topicExchange") TopicExchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with("test.topic.queue1");
    }

    @Autowired
    private AmqpTemplate template;

    public void send1() {
        String message = "hello world 1";
        System.out.println("producer send a topic message 1: " + message);
        template.convertAndSend(TOPIC_EXCHANGE_NAME, TOPIC_QUEUE1_NAME, message);
    }
}
```
#### consumer 项目

```
/**
 * Topic模式消费者.
 */
@Component
public class TopicConsumer {

    /**
     * queue.
     */
    private static final String TOPIC_QUEUE1_NAME = "test.topic.queue1";
    
    /**
     * 初始化queue1.
     */
    @Bean
    public Queue topicQueue1() {
        return new Queue(TOPIC_QUEUE1_NAME);
    }
    
    /**
     * 监听器监听指定的Queue1.
     */
    @RabbitListener(queues = TOPIC_QUEUE1_NAME)
    public void receive1(String message) {
        System.out.println("consumer receive a topic message 1: " + message);
    }
}
```
#### 运行结果

> consumer receive a topic message 1: hello world 1

#### 代码分析

和Direct模式对比来看，主要是producer里多设置了一个exchange，多了一个将exchange和queue进行绑定的方法，发送消息时也多了一个exchange的入参。

那exchange是什么呢？有什么作用呢？

我的看法是，<font color="red">*exchange 可以看做是一个 queue 的组，往这个 exchange 里发送的消息，只有和它绑定的 queue 才可以接收到。</font>（你可以尝试删除 bindingTopicMessage() 方法取消 queue 和 exchange 的绑定，<b>删除 server 上已经生成的队列</b>，重新运行程序，就会观察到 consumer 不再能够消费到消息。）

每个 queue 和 exchange 绑定时都要设置一个 BindingKey，用于声明发往这个 exchange 中的什么样的消息我这个 queue 可以接收。

```
// with() 里的入参就是 BindingKey
BindingBuilder.bind(queue).to(exchange).with("test.topic.queue");
```
而生产者发送消息时，要指定一个 RoutingKey，可以理解为我要发往这个 exchange 里的什么样的队列。

```
// convertAndSend() 的第二个入参就是 RoutingKey
template.convertAndSend(TOPIC_EXCHANGE_NAME, TOPIC_QUEUE_NAME, message);
```
<font color="red">*当发送消息时的 RoutingKey 和绑定的 BindingKey 相匹配时，消息即被存入相应的队列中。</font>

### 示例2

在上面代码的基础上，我们再加一个 queue 的代码

#### producer 代码

```
/**
 * queue.
 */
private static final String TOPIC_QUEUE2_NAME = "test.topic.queue2";

/**
 * 初始化queue2.
 */
@Bean(name = "topicQueue2")
public Queue topicQueue2() {
    return new Queue(TOPIC_QUEUE2_NAME);
}

/**
 * 绑定2.
 */
@Bean
Binding bindingTopicMessage2(@Qualifier("topicQueue2") Queue queue,
                             @Qualifier("topicExchange") TopicExchange exchange) {
    // *表示一个词，#表示0个或多个词
    return BindingBuilder.bind(queue).to(exchange).with("test.topic.*");
}

public void send2() {
    String message = "hello world 2";
    System.out.println("producer send a topic message 2: " + message);
    template.convertAndSend(TOPIC_EXCHANGE_NAME, TOPIC_QUEUE2_NAME, message);
}
```

#### consumer 代码

```
/**
 * queue.
 */
private static final String TOPIC_QUEUE2_NAME = "test.topic.queue2";

/**
 * 初始化queue2.
 */
@Bean
public Queue topicQueue2() {
    return new Queue(TOPIC_QUEUE2_NAME);
}

/**
 * 监听器监听指定的Queue2.
 */
@RabbitListener(queues = TOPIC_QUEUE2_NAME)
public void receive2(String message) {
    System.out.println("consumer receive a topic message 2: " + message);
}
```

#### 运行结果

调用 send1()

> consumer receive a topic message 2: hello world 1<br/>
> consumer receive a topic message 1: hello world 1

调用 send2()

> consumer receive a topic message 2: hello world 2

#### 代码分析

正如代码备注里所写，<font color="red">TopicExchange 的 BindingKey 支持模糊匹配模式，*表示一个词，#表示0个或多个词。</font>

在 send1() 中，RoutingKey 是 test.topic.queue1，queue1 的 test.topic.queue1 能够完全匹配，queue2 的 test.topic.* 也能模糊匹配上，因此两个 queue 都收到了消息。<font color="blue">注意和上一篇最后的轮询结论区分，这里是两个queue，上一篇是同一个 queue 的两个消费者。</font>

在 send2() 中，RoutingKey 是 test.topic.queue2，queue1 的 test.topic.queue1 不能够完全匹配，queue2 的 test.topic.* 能模糊匹配上，因此只有 queue2 能够收到消息。

### 与 Direct 区别

了解了Topic模式再回来看Direct，可以感觉出，Direct可以看作是一种简化版的Topic，不需要声明 exchange （绑定在 default exchange上），不需要写 queue 和 exchange 绑定 (queue 和 default exchange绑定，queue name 就是 BindingKey)。

