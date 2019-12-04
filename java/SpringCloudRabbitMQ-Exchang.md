# RabbitMQ - 交换器

|交换器名称|作用|
|:-|:-|
|fanout exchange|发送到该交换器的所有消息，会被路由到其绑定的所有队列|
|direct exchange|发送到该交换器的消息，会通过路由键完全匹配，匹配成功就会路由到指定队列|
|topic exchange|发送到该交换器的消息，会通过路由键模糊匹配，匹配成功就会路由到指定队列|
|header exchange|发送到该交换器的消息，会通过消息的 header 信息匹配，匹配成功就会路由到指定队列|
## 核心代码
pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.tgyf.rabbitmq</groupId>
    <artifactId>exchange</artifactId>
    <version>1.0-SNAPSHOT</version>


    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
    </parent>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>

    </dependencies>

</project>
```

 1. direct exchange -- 直接交换器
 * 发送到该交换器的消息都会被路由到与 routing key 匹配的队列中
```java
package com.tgyf.rabbit.config;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * direct exchange -- 直接交换器
 * 发送到该交换器的消息都会被路由到与 routing key 匹配的队列中
 * @author 韬光养月巴
 * @modify
 * @createDate 2019/8/24 12:07 PM
 * @remark
 */
@Configuration
public class DirectExchangeConf {
    public static final String QUEUE_1 = "direct-queue-1";

    public static final String QUEUE_2 = "direct-queue-2";

    private static final String EXCHANGE = "exchange-direct";

    private static final String ROUTING_KEY_TO_QUEUE1 = "queue.direct.key1";

    private static final String ROUTING_KEY_TO_QUEUE2 = "queue.direct.key2";

    @Bean
    Queue directQueue1() {
        return new Queue(QUEUE_1, false);
    }

    @Bean
    Queue directQueue2() {
        return new Queue(QUEUE_2, false);
    }

    @Bean
    DirectExchange directExchange() {
        return new DirectExchange(EXCHANGE);
    }

    @Bean
    Binding bindingDirectQueue1(Queue directQueue1, DirectExchange directExchange) {
        return BindingBuilder.bind(directQueue1).to(directExchange).with(ROUTING_KEY_TO_QUEUE1);
    }

    @Bean
    Binding bindingDirectQueue2(Queue directQueue2, DirectExchange directExchange) {
        return BindingBuilder.bind(directQueue2).to(directExchange).with(ROUTING_KEY_TO_QUEUE2);
    }
}

```
 2. fanout exchange -- 扇出交换器
 * 所有发送到该交换器的消息都会被路由到所有与该交换器绑定的队列中
```java
package com.tgyf.rabbit.config;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.FanoutExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
/**
 * fanout exchange -- 扇出交换器
 * 所有发送到该交换器的消息都会被路由到所有与该交换器绑定的队列中
 * @author 韬光养月巴
 * @modify
 * @createDate 2019/8/24 12:04 PM
 * @remark
 */
@Configuration
public class FanoutExchangeConf {
    public static final String QUEUE_1 = "fanout-queue-1";

    public static final String QUEUE_2 = "fanout-queue-2";

    private static final String EXCHANGE = "exchange-fanout";

    @Bean
    Queue fanoutQueue1() {
        return new Queue(QUEUE_1, false);
    }

    @Bean
    Queue fanoutQueue2() {
        return new Queue(QUEUE_2, false);
    }

    @Bean
    FanoutExchange fanoutExchange() {
        return new FanoutExchange(EXCHANGE);
    }

    @Bean
    Binding bindingFanoutQueue1(Queue fanoutQueue1, FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(fanoutQueue1).to(fanoutExchange);
    }

    @Bean
    Binding bindingFanoutQueue2(Queue fanoutQueue2, FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(fanoutQueue2).to(fanoutExchange);
    }
}

```

 3. headers exchange -- headers交换器
 * 发送到该交换器的消息会根据消息的 header 信息路由到对应的队列
 ```java
package com.tgyf.rabbit.config;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.HeadersExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;

/**
 * headers exchange -- headers交换器
 * 发送到该交换器的消息会根据消息的 header 信息路由到对应的队列
 * 说明：
 * where 匹配单个 header
 * whereAll 同时匹配多个 header
 * whereAny 匹配一个或多个 header
 *
 * @author 韬光养月巴
 * @modify
 * @createDate 2019/8/24 12:12 PM
 * @remark
 */
public class HeadersExchangeConf {
    public static final String QUEUE_1 = "headers-queue-1";

    public static final String QUEUE_2 = "headers-queue-2";

    public static final String QUEUE_3 = "headers-queue-3";

    private static final String EXCHANGE = "exchange-headers";

    @Bean
    Queue headersQueue1() {
        return new Queue(QUEUE_1, false);
    }

    @Bean
    Queue headersQueue2() {
        return new Queue(QUEUE_2, false);
    }

    @Bean
    Queue headersQueue3() {
        return new Queue(QUEUE_3, false);
    }

    @Bean
    HeadersExchange headersExchange() {
        return new HeadersExchange(EXCHANGE);
    }

    @Bean
    Binding bindingHeadersQueue1(Queue headersQueue1, HeadersExchange headersExchange) {
        return BindingBuilder.bind(headersQueue1).to(headersExchange).where("one").exists();
    }

    @Bean
    Binding bindingHeadersQueue2(Queue headersQueue1, HeadersExchange headersExchange) {
        return BindingBuilder.bind(headersQueue1).to(headersExchange).whereAll("all1", "all2").exist();
    }

    @Bean
    Binding bindingHeadersQueue3(Queue headersQueue3, HeadersExchange headersExchange) {
        return BindingBuilder.bind(headersQueue3).to(headersExchange).whereAny("any1", "any2").exist();
    }
}

```
 4. topic exchange -- 主题交换器
 * 发送到该交换器的消息都会被路由到与 routing key 匹配的队列中
 ```java
package com.tgyf.rabbit.config;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.core.TopicExchange;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * topic exchange -- 主题交换器
 * 发送到该交换器的消息都会被路由到与 routing key 匹配的队列中
 * 说明：
 * routing key 以 '.' 分隔为多个单词
 * routing key 以 '*' 匹配一个单词
 * routing key 以 '#' 匹配零个或多个单词
 * 例如：
 * queue.topic.key ->  QUEUE_1 + QUEUE_2
 * test.topic.key -> QUEUE_1
 * queue -> QUEUE_2
 * queue.topic -> QUEUE_2
 *
 * @author 韬光养月巴
 * @modify
 * @createDate 2019/8/24 12:09 PM
 * @remark
 */
@Configuration
public class TopicExchangeConf {
    public static final String QUEUE_1 = "topic-queue-1";

    public static final String QUEUE_2 = "topic-queue-2";

    private static final String EXCHANGE = "exchange-topic";

    private static final String ROUTING_KEY_TO_QUEUE1 = "*.topic.*";

    private static final String ROUTING_KEY_TO_QUEUE2 = "queue.#";

    @Bean
    Queue topicQueue1() {
        return new Queue(QUEUE_1, false);
    }

    @Bean
    Queue topicQueue2() {
        return new Queue(QUEUE_2, false);
    }

    @Bean
    TopicExchange topicExchange() {
        return new TopicExchange(EXCHANGE);
    }

    @Bean
    Binding bindingTopicQueue1(Queue topicQueue1, TopicExchange topicExchange) {
        return BindingBuilder.bind(topicQueue1).to(topicExchange).with(ROUTING_KEY_TO_QUEUE1);
    }

    @Bean
    Binding bindingTopicQueue2(Queue topicQueue2, TopicExchange topicExchange) {
        return BindingBuilder.bind(topicQueue2).to(topicExchange).with(ROUTING_KEY_TO_QUEUE2);
    }

}

```

## 测试

### 1.测试fanout exchange

    curl -X POST \
      http://127.0.0.1/send \
      -H 'Content-Type: application/json' \
      -d '{
        "exchange":"exchange-fanout",
        "routingKey": "default",
        "content":" hello fanout!",
        "count": 1
    }'
    
### 2.测试 direct exchange

    curl -X POST \
      http://127.0.0.1/send \
      -H 'Content-Type: application/json' \
      -d '{
        "exchange":"exchange-direct",
        "routingKey": "queue.direct.key1",
        "content":" hello direct! ",
        "count": 1
    }'
    
    curl -X POST \
      http://127.0.0.1/send \
      -H 'Content-Type: application/json' \
      -d '{
        "exchange":"exchange-direct",
        "routingKey": "queue.direct.key2",
        "content":" hello direct! ",
        "count": 1
    }'

### 3.测试 topic exchange
    
    curl -X POST \
      http://127.0.0.1/send \
      -H 'Content-Type: application/json' \
      -d '{
        "exchange":"exchange-topic",
        "routingKey": "queue.topic.key1",
        "content":" hello topic! ",
        "count": 1
    }'
    
    curl -X POST \
      http://127.0.0.1/send \
      -H 'Content-Type: application/json' \
      -d '{
        "exchange":"exchange-topic",
        "routingKey": "test.topic.key2",
        "content":" hello topic! ",
        "count": 1
    }'
    
    curl -X POST \
      http://127.0.0.1/send \
      -H 'Content-Type: application/json' \
      -d '{
        "exchange":"exchange-topic",
        "routingKey": "queue.hello",
        "content":" hello topic! ",
        "count": 1
    }'
    
### 4.测试 headers exchange
    
    curl -X POST \
      http://127.0.0.1/send \
      -H 'Content-Type: application/json' \
      -d '{
        "exchange":"exchange-headers",
        "content":" hello headers! ",
        "count": 1,
        "headers":{
            "one":"value"
        }
    }'
    
    curl -X POST \
      http://127.0.0.1/send \
      -H 'Content-Type: application/json' \
      -d '{
        "exchange":"exchange-headers",
        "content":" hello headers! ",
        "count": 1,
        "headers":{
            "all1":"value",
            "all2":"value"
        }
    }'
    
    curl -X POST \
      http://127.0.0.1/send \
      -H 'Content-Type: application/json' \
      -d '{
        "exchange":"exchange-headers",
        "content":" hello headers! ",
        "count": 1,
        "headers":{
            "any2":"value",
        }
    }'