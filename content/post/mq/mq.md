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

消费后确认（ack）：消费者在接收到消息后，会立即返回一个确认消息，告诉生产者消息已经被消费，生产者会将消息从队列中删除，这样可以避免因为消费者处理能力不足而导致的消息堆积

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

#### RocketMQ
