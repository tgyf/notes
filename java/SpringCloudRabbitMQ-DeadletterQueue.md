# RabbitMQ - 死信队列(DeadLetterQueue)
## 什么是死信
“死信”是RabbitMQ中的一种消息机制，当你在消费消息时，如果队列里的消息出现以下情况：

    1.消息被否定确认，使用 channel.basicNack 或 channel.basicReject ，并且此时requeue 属性被设置为false。
    2.消息在队列的存活时间超过设置的TTL时间。
    3.消息队列的消息数量已经超过最大队列长度。
    那么该消息将成为“死信”。

* “死信”消息会被RabbitMQ进行特殊处理，如果配置了死信队列信息，那么该消息将会被丢进死信队列中，如果没有配置，则该消息将会被丢弃。

## 死信生命周期
死信队列只是一个绑定在死信交换机上的普通队列，而死信交换机也只是一个普通的交换机，不过是用来专门处理死信的交换机。

死信的生命周期：

1. 业务消息被投入业务队列
2. 消费者消费业务队列的消息，由于处理过程中发生异常，于是进行了nck或者reject操作
3. 被nck或reject的消息由RabbitMQ投递到死信交换机中
4. 死信交换机将消息投入相应的死信队列
5. 死信队列的消费者消费死信消息
* 死信消息是RabbitMQ为我们做的一层保证，其实我们也可以不使用死信队列，而是在消息消费异常时，将消息主动投递到另一个交换机中，关键在于这些Exchange和Queue怎么配合。比如从死信队列拉取消息，然后发送邮件、短信、钉钉通知来通知开发人员关注。或者将消息重新投递到一个队列然后设置过期时间，来进行延时消费。

## 死信消息的Header
|字段名|含义|
|:-|:-|
|x-first-death-exchange|第一次被抛入的死信交换机的名称|
|x-first-death-reason|第一次成为死信的原因，rejected：消息在重新进入队列时被队列拒绝，由于default-requeue-rejected 参数被设置为false。expired ：消息过期。maxlen ： 队列内消息数量超过队列最大容量|
|x-first-death-queue|第一次成为死信前所在队列名称|
|x-death|历次被投入死信交换机的信息列表，同一个消息每次进入一个死信交换机，这个数组的信息就会被更新|

## 死信队列应用场景
* 一般用在较为重要的业务队列中，确保未被正确消费的消息不被丢弃，一般发生消费异常可能原因主要有由于消息信息本身存在错误导致处理异常，处理过程中参数校验异常，或者因网络波动导致的查询异常等等，当发生异常时，当然不能每次通过日志来获取原消息，然后让运维帮忙重新投递消息（没错，以前就是这么干的= =）。通过配置死信队列，可以让未正确处理的消息暂存到另一个队列中，待后续排查清楚问题后，编写相应的处理代码来处理死信消息，这样比手工恢复数据要好太多了。
## 核心代码
```java
package com.tgyf.rabbit.config;

import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
/**
 * 死信交换器
 *
 * @author 韬光养月巴
 * @modify
 * @createDate 2019/8/24 2:18 PM
 * @remark
 */
@Configuration
@Slf4j
public class DeadLetterExchangeConf {
    public static final String QUEUE_BY_MAX_LENGTH = "direct-queue-dead-by-max-length";

    public static final String QUEUE_BY_TTL = "direct-queue-dead-by-ttl";

    public static final String QUEUE_BY_REJECT = "direct-queue-dead-by-reject";

    public static final String EXCHANGE = "exchange-direct-dead";

    public static final String ROUTING_KEY_BY_MAX_LENGTH = "direct.queue.dead.max.length";

    public static final String ROUTING_KEY_BY_TTL = "direct.queue.dead.ttl";

    public static final String ROUTING_KEY_BY_REJECT = "direct.queue.dead.reject";

    @Bean
    Queue deadByMaxLengthQueue() {
        return new Queue(QUEUE_BY_MAX_LENGTH, false);
    }

    @Bean
    Queue deadByTTLQueue() {
        return new Queue(QUEUE_BY_TTL, false);
    }

    @Bean
    Queue deadByRejectQueue() {
        return new Queue(QUEUE_BY_REJECT, false);
    }

    @Bean
    DirectExchange deadDirectExchange() {
        return new DirectExchange(EXCHANGE);
    }

    @Bean
    Binding deadByMaxLengthQueueBinding(Queue deadByMaxLengthQueue, DirectExchange deadDirectExchange) {
        return BindingBuilder.bind(deadByMaxLengthQueue).to(deadDirectExchange).with(ROUTING_KEY_BY_MAX_LENGTH);
    }

    @Bean
    Binding deadByTTLQueueBinding(Queue deadByTTLQueue, DirectExchange deadDirectExchange) {
        return BindingBuilder.bind(deadByTTLQueue).to(deadDirectExchange).with(ROUTING_KEY_BY_TTL);
    }

    @Bean
    Binding deadByRejectQueueBinding(Queue deadByRejectQueue, DirectExchange deadDirectExchange) {
        return BindingBuilder.bind(deadByRejectQueue).to(deadDirectExchange).with(ROUTING_KEY_BY_REJECT);
    }

}

```

```java
package com.tgyf.rabbit.config;

import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;
import java.util.Map;

/**
 * 普通交换器
 *
 * @author 韬光养月巴
 * @modify
 * @createDate 2019/8/24 2:18 PM
 * @remark
 */
@Configuration
@Slf4j
public class DirectExchangeConf {
    public static final String QUEUE_MAX_LENGTH = "direct-queue-max-length";

    public static final String QUEUE_TTL = "direct-queue-ttl";

    public static final String QUEUE_REJECT = "direct-queue-reject";

    public static final String EXCHANGE = "exchange-direct";

    public static final String ROUTING_KEY_MAX_LENGTH = "direct.queue.max.length";

    public static final String ROUTING_KEY_TTL = "direct.queue.ttl";

    public static final String ROUTING_KEY_REJECT = "direct.queue.reject";

    @Bean
    Queue maxLengthQueue() {
        Map<String, Object> args= new HashMap<>();
        // 设置队列最大长度
        args.put("x-max-length", 10);
        // 设置死信转发的 exchange 和 routing key
        args.put("x-dead-letter-exchange", DeadLetterExchangeConf.EXCHANGE);
        args.put("x-dead-letter-routing-key", DeadLetterExchangeConf.ROUTING_KEY_BY_MAX_LENGTH);
        return new Queue(QUEUE_MAX_LENGTH, false, false, false, args);
    }

    @Bean
    Queue ttlQueue() {
        Map<String, Object> args= new HashMap<>();
        // 设置消息存活时间 10s
        args.put("x-message-ttl", 10000);
        // 设置死信转发的 exchange 和 routing key
        args.put("x-dead-letter-exchange", DeadLetterExchangeConf.EXCHANGE);
        args.put("x-dead-letter-routing-key", DeadLetterExchangeConf.ROUTING_KEY_BY_TTL);
        return new Queue(QUEUE_TTL, false, false, false, args);
    }

    @Bean
    Queue rejectQueue() {
        Map<String, Object> args= new HashMap<>();
        // 设置死信转发的 exchange 和 routing key
        args.put("x-dead-letter-exchange", DeadLetterExchangeConf.EXCHANGE);
        args.put("x-dead-letter-routing-key", DeadLetterExchangeConf.ROUTING_KEY_BY_REJECT);
        return new Queue(QUEUE_REJECT, false, false, false, args);
    }

    @Bean
    DirectExchange directExchange() {
        return new DirectExchange(EXCHANGE);
    }

    @Bean
    Binding maxLengthQueueBinding(Queue maxLengthQueue, DirectExchange directExchange) {
        return BindingBuilder.bind(maxLengthQueue).to(directExchange).with(ROUTING_KEY_MAX_LENGTH);
    }

    @Bean
    Binding ttlQueueBinding(Queue ttlQueue, DirectExchange directExchange) {
        return BindingBuilder.bind(ttlQueue).to(directExchange).with(ROUTING_KEY_TTL);
    }

    @Bean
    Binding rejectQueueBinding(Queue rejectQueue, DirectExchange directExchange) {
        return BindingBuilder.bind(rejectQueue).to(directExchange).with(ROUTING_KEY_REJECT);
    }

}

```
## 测试
1.测试消费者否认消息

    curl -X POST \
      http://127.0.0.1:8080/send \
      -H 'Content-Type: application/json' \
      -d '{
        "exchange":"exchange-direct",
        "routingKey": "direct.queue.reject",
        "content":" hello reject queue! ",
        "count": 1
    }'
2.测试消息超出队列最大长度

    curl -X POST \
      http://127.0.0.1:8080/send \
      -H 'Content-Type: application/json' \
      -d '{
        "exchange":"exchange-direct",
        "routingKey": "direct.queue.max.length",
        "content":" hello max length queue! ",
        "count": 30
    }'
* 提示：消息队列遵循先进先出的策略，假设队列最大长度设置为 10，发送 30 条消息到该队列，若无消费者，前 20 条消息会被转发到指定的其他队列，后 10 条会保存在该队列中，除非有新的消息入队，这 10 条消息才会被转发

3.测试消息超时

    curl -X POST \
      http://127.0.0.1:8080/send \
      -H 'Content-Type: application/json' \
      -d '{
        "exchange":"exchange-direct",
        "routingKey": "direct.queue.ttl",
        "content":" hello ttl queue! ",
        "count": 10
    }'
    
4.测试延迟队列

    curl -X POST \
      http://127.0.0.1/send \
      -H 'Content-Type: application/json' \
      -d '{
        "exchange":"exchange-direct",
        "routingKey": "direct.queue.delay",
        "content":" hello delay delay! ",
        "count": 1,
        "delayTime": "10000"
    }'