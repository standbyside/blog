---
title: 探索RabbitMQ（4）：Fanout模式
id: explore-rabbitmq-4
date: 2019-04-18 16:09:06
updated: 2019-04-18 16:09:06
categories:
  - 人类的本质是复读机
tags:
  - rabbitmq
---
前两种模式，可以理解为点对点模式，这篇讲Fanout（广播）模式，顾名思义，就是生产者只要在exchange里喊一嗓子，所有绑定到这个exchange的queue都能接收到，代码写起来也非常简单。

<!-- more -->

### 示例

#### producer项目

```
/**
 * Fanout模式生产者.
 */
@Component
public class FanoutProducer {

  /**
   * exchange.
   */
  private static final String FANOUT_EXCHANGE_NAME = "spring.rabbit.fanout";

  /**
   * 初始化exchange.
   */
  @Bean
  FanoutExchange fanoutExchange() {
    return new FanoutExchange(FANOUT_EXCHANGE_NAME);
  }

  @Autowired
  private AmqpTemplate template;

  public void send() {
    String message = "hello world";
    System.out.println("producer send a fanout message: " + message);
    // 参数2将被忽略
    template.convertAndSend(FANOUT_EXCHANGE_NAME, "", message);
  }
}
```

#### consumer项目

```
/**
 * Fanout模式消费者.
 */
@Component
public class FanoutConsumer {

  /**
   * exchange.
   */
  private static final String FANOUT_EXCHANGE_NAME = "spring.rabbit.fanout";

  /**
   * queue.
   */
  private static final String FANOUT_QUEUE1_NAME = "test.fanout.queue1";
  private static final String FANOUT_QUEUE2_NAME = "test.fanout.queue2";

  /**
   * 初始化exchange.
   */
  @Bean
  FanoutExchange fanoutExchange() {
    return new FanoutExchange(FANOUT_EXCHANGE_NAME);
  }

  /**
   * 初始化queue1.
   */
  @Bean(name = "fanoutQueue1")
  public Queue fanoutQueue1() {
    return new Queue(FANOUT_QUEUE1_NAME);
  }

  /**
   * 初始化queue2.
   */
  @Bean(name = "fanoutQueue2")
  public Queue fanoutQueue2() {
    return new Queue(FANOUT_QUEUE2_NAME);
  }

  @Bean
  Binding bindingFanoutExchange1(@Qualifier("fanoutQueue1") Queue queue, FanoutExchange exchange) {
    return BindingBuilder.bind(queue).to(exchange);
  }

  @Bean
  Binding bindingFanoutExchange2(@Qualifier("fanoutQueue2") Queue queue, FanoutExchange exchange) {
    return BindingBuilder.bind(queue).to(exchange);
  }

  @RabbitListener(queues = FANOUT_QUEUE1_NAME)
  public void receive1(String message) {
    System.out.println("consumer receive a fanout message 1: " + message);
  }

  @RabbitListener(queues = FANOUT_QUEUE2_NAME)
  public void receive2(String message) {
    System.out.println("consumer receive a fanout message 2: " + message);
  }
}
```
#### 运行结果
> producer send a fanout message: hello world

> consumer receive a fanout message 1: hello world</br>
consumer receive a fanout message 2: hello world

#### 代码分析

我感觉没啥可分析的了...可以看出来，producer 这边不需要声明任何 queue，因为它并不指定发送到那个 queue 上。哪个 consumer 需要这个数据，自己声明 queue 绑定到对应 exchange 上监听即可。

### Header模式

前面说了，RabbitMQ一共四种模式，这第四种就是Header模式。简单来说就是发送者和接受者都定义一些key-value，在any mathc或all match时，可以接收到消息。

但是似乎很久以前就基本已经不用了，有多久呢，在[《RabbitMQ实战》](https://book.douban.com/subject/26649178/)这本2012年写成的书里，作者就已经说header模式基本不用了，这时候连死信队列都还没有呢，所以真的是很久以前就没用...

以后如果心血来潮的话，我可能会看一下header模式，补个demo啥的，现在有其他更想写的东西，所以暂时就只给它这样一个小小的段落吧。

