---
title: "MQ消息队列"
date: 2022-11-05T15:51:09+08:00
math: true
slug: "Mq"
# weight: 1
# aliases: ["/first"]
tags: ["消息队列"]
categories: ["知识点"]
# author: ["Me", "You"] # multiple authors
# showToc: true
# TocOpen: false
# draft: false
# hidemeta: false
# comments: false
# description: "Desc Text."
image: "https://api.ixiaowai.cn/api/api.php"
---

### MQ

优点：解耦、削峰、异步
1.服务解耦
2.异步，性能提升，吞吐量提高
3.无强依赖关系，故障隔离
4.削峰填谷

缺点：可靠性低、系统复杂度提高、一致性问题

#### RabbitMQ

##### 消息模型

###### 基本消息队列（BasicQueue）

一个生产者（publisher），一个消费者（consumer），一个队列（queue）

###### SpringAMQP

AMQP：Advanced Message Queuing Protocol
应用间消息通信的标准协议，与语言和平台无关

特征：
1.监听器容器（ListenerContainer），用于异步处理消息
2.用于发送和接收消息的模板（RabbitTemplate）
3.RabbitAdmin用于自动声明队列、交换器、绑定等

多租户：使用virtual-host实现

流程：
1.工程引入spring-boot-starter-amqp依赖，yml配置
2.在publisher中注入RabbitTemplate，调用convertAndSend（queueName， message）方法发送消息
3.接收消息
> 1.在consumer中注入RabbitTemplate，调用receiveAndConvert（queueName， message）方法
> 2.类中添加@Component注解， 方法添加@RabbitListener注解监听消息

###### 工作消息队列（WorkQueue）

`消费后被删除`

一个生产者（publisher），多个消费者（consumer），一个队列（queue）

消息预取（prefetch）：消费者在接收到消息后，不会立即处理，而是先将消息放到本地队列中，等待消费者处理完毕后，再从本地队列中取出下一个消息进行处理，这样可以提高消息的处理效率，避免因为消费者处理能力不足而导致的消息堆积
> listener.simple.prefetch=1 // 消息预取每次只取一个

###### 发布订阅消息队列（Publish/Subscribe）

一个生产者（publisher），多个消费者（consumer），多个队列（queue）

publisher将消息发送到交换机（exchange），exchange将消息发送到队列（queue），消费者从队列中获取消息

> exchange负责消息路由，路由失败则消息丢失，queue负责消息存储

根据交换机类型：

1.广播（Fanout Exchange）

consumer服务声明Exchange，Queue，Binding

```Java
@Configuration
public class FanoutConfig{
    @Bean
    public FanoutExchange fanoutExchange(){
        return new FanoutExchange("fanoutExchange");
    }
    @Bean
    public Queue fanoutQueue1(){
        return new Queue("fanoutQueue1");
    }
    @Bean
    public Binding fanoutBinding1(Queue fanoutQueue1, FanoutExchange fanoutExchange){
        return BindingBuilder.bind(fanoutQueue1()).to(fanoutExchange());
    }
}

// 发送
rabbitTemplate.convertAndSend("fanoutExchange", "routingKey", "fanout message");

// 使用
@RabbitListener(queues = "fanoutQueue1")
public void process(String message){
    System.out.println("fanoutQueue1: " + message);
}

```

2.路由（Routing Exchange）

```Java
@RabbitListener(Bindings = @QueueBinding(
    value = @Queue("fanoutQueue1"),
    exchange = @Exchange("fanoutExchange", type = ExchangeTypes.DIRECT),
    key = "routingKey"
))
public void listenDirectQueue1(String message){
    System.out.println("fanoutQueue2: " + message);
}
```

3.主题（Topics Exchange）

```Java
@RabbitListener(Bindings = @QueueBinding(
    value = @Queue("fanoutQueue1.topic"),
    exchange = @Exchange("fanoutExchange.topic", type = ExchangeTypes.TOPIC),
    key = "routingKey.#"
))
public void listenTopicQueue1(String message){
    System.out.println("fanoutQueue3: " + message);
}
```

##### 消息转换器

publisher和consumer之间传递的消息都是字节流，需要转换成对象

依赖：
jackson-databind:核心包
jackson-dataformat-xml：xml转换器

```Java
// publisher中
@Bean
public MessageConverter messageConverter(){
    return new Jackson2JsonMessageConverter();
}

// consumer中
@Bean
public MessageConverter messageConverter(){
    return new Jackson2JsonMessageConverter();
}

@RabbitListener(queues = "queue")
public void process(User user){
    System.out.println(user);
}
```

#### 高级特性

##### 消息可靠性投递

###### confirm 确认模式

publisher发送消息后，broker会返回一个确认消息，publisher收到确认消息后，会将消息从本地队列中删除

producer->broker->exchange, 返回confirmCallback

publish-confirm: true

###### return 退回模式

exchange->queue, 返回returnCallback

publisher-return: true

> 默认情况下，如果exchange无法根据自身类型和路由键找到一个符合条件的queue，那么会把消息丢弃
> 设置mandatory=true，如果exchange无法根据自身类型和路由键找到一个符合条件的queue，那么会调用returnCallback

message 消息对象 replyCode 错误码 replyText 错误信息 exchange 交换机 routingKey 路由键

##### Consumer端消息确认

消费后确认（ack）：消费者在接收到消息后，会立即返回一个确认消息，告诉生产者消息已经被消费，生产者会将消息从队列中删除，这样可以避免因为消费者处理能力不足而导致的消息堆积

acknowledge-mode: none 自动确认
acknowledge-mode: manual 手动确认

> ChannelAwareMessageListener 接口
> 成功->channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
> 失败->channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, true（重回队列）);

acknowledge-mode: auto 根据情况确认

消费后拒绝（nack）：消费者在接收到消息后，会立即返回一个拒绝消息，告诉生产者消息未被消费，生产者会将消息重新放回队列，等待其他消费者消费

##### TTL

Time To Live （存活时间/过期时间）

> 消息过期时间：设置消息的过期时间，超过过期时间，消息会被自动删除

```Java
1.队列统一过期

x-message-ttl: 60000 # 毫秒 value-type: Integer

2.消息单独过期

// 消息后处理对象，设置一些消息的参数信息
MessagePostProcessor messagePostProcessor = new MessagePostProcessor() {
    @Override
    public Message postProcessMessage(Message message) throws AmqpException {
        message.getMessageProperties().setExpiration("60000");
        return message;
    }
};

// 如果设置了消息的过期时间，也设置了队列过期时间，以时间短的为准
// 队列过期后，会将队列中的消息一起删除
// 消息过期后，只有消息在队列顶端，才会判断是否过期（移除）

```

##### 死信队列

DLX (Dead Letter Exchange) 死信交换机

当消息在一个队列中变成死信（dead message）之后，它能被重新publish到另一个Exchange，这个Exchange就是DLX，死信路由键就是原队列的路由键

> 队列消息长度到达最大值
> 消息被拒绝（basic.reject/ basic.nack）并且requeue=false
> 原队列存在消息过期设置，消息到达超时时间未被消费

绑定死信队列
给队列设置死信交换机和死信路由键:
x-dead-letter-exchange
x-dead-letter-routing-key

##### 延迟队列

TTL + DLX
> 延迟队列：消息发送到队列后，不会立即被消费，而是在指定的时间后才能被消费

优化：设置一个没有TTL时间的队列，将消息发送到这个队列，然后再将消息发送到真正的队列

rabbitmq_delayed_message_exchange 插件 (rabbitmq 3.8.0 之后自带) type: x-delayed-message
使用插件会在交换机延迟

```Java
msg.getMessageProperties().setDelay(10000);
```

##### 幂等性

> 幂等性：同一个请求，多次执行，结果是一样的

消息重复消费
> 全局ID：唯一标识如时间戳/UUID/雪花算法
> 保证幂等性的方法：1.数据库唯一索引 2.数据库乐观锁 3.数据库悲观锁 4.分布式锁

##### 优先级队列

x-max-priority: 10 # 设置队列的最大优先级 value-type: Integer

##### 惰性队列

消息保存在磁盘上，只有在消费者连接到队列时才会被加载到内存中
> 惰性队列：队列中没有消息时，不会消费消息
x-queue-mode: lazy # 设置队列的模式 value-type: String

#### RocketMQ
