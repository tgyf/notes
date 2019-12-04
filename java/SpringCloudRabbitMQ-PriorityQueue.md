# RabbitMQ - 优先级队列(PriorityQueue)
在RabbitMQ中使用优先级特性需要的版本为3.5+。

    使用优先级特性只需做两件事情：
        1. 将队列声明为优先级队列，即在创建队列的时候添加参数 x-max-priority 以指定最大的优先级，值为0-255（整数）。
        2. 为优先级消息添加优先级。
* 注意:没有指定优先级的消息会将优先级以0对待。 对于超过优先级队列所定最大优先级的消息，优先级以最大优先级对待。对于相同优先级的消息，后进的排在前面。
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

import java.util.HashMap;
import java.util.Map;

/**
 * direct exchange -- 直接交换器
 * 发送到该交换器的消息都会被路由到与 routing key 匹配的队列中
 * @author 韬光养月巴
 * @modify
 * @createDate 2019/8/24 1:29 PM
 * @remark
 */
@Configuration
@Slf4j
public class DirectExchangeConf {
    public static final String QUEUE = "direct-queue-priority";

    public static final String EXCHANGE = "exchange-direct";

    public static final String ROUTING_KEY = "direct.queue.priority";

    @Bean
    Queue directQueuePriority() {
        //创建队列的时候添加参数 x-max-priority 以指定最大的优先级，值为0-255
        Map<String, Object> args= new HashMap<>();
        args.put("x-max-priority", 100);
        return new Queue(QUEUE, false, false, false, args);
    }

    @Bean
    DirectExchange directExchange() {
        return new DirectExchange(EXCHANGE);
    }

    @Bean
    Binding directQueuePriorityBinding(Queue directQueuePriority, DirectExchange directExchange) {
        return BindingBuilder.bind(directQueuePriority).to(directExchange).with(ROUTING_KEY);
    }

}

```
## 测试
### 1.测试优先级队列
      
发送优先级低的消息 100 条到 RabbitMQ

    curl -X POST \
      http://127.0.0.1/send \
      -H 'Content-Type: application/json' \
      -d '{
        "exchange":"exchange-direct",
        "routingKey": "direct.queue.priority",
        "priority": 1,
        "content":" hello priority queue! ",
        "count": 100
    }'
    
发送优先级高的消息 5 条到 RabbitMQ

    curl -X POST \
      http://127.0.0.1/send \
      -H 'Content-Type: application/json' \
      -d '{
        "exchange":"exchange-direct",
        "routingKey": "direct.queue.priority",
        "priority": 10,
        "content":" >>>>>>>> hello priority queue! ",
        "count": 5
    }'