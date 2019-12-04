## 启动 RabbitMQ
    docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 -v `pwd`/data:/var/lib/rabbitmq --hostname rabbit -e RABBITMQ_DEFAULT_VHOST=my_vhost  -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin rabbitmq:3.7-management-plugins

* 构建RabbitMQ镜像参考[RabbitMQ 镜像dockerfile](https://github.com/tgyf/dockerfile/tree/master/rabbitmq )

## 内容结构 
|序号|内容名称|内容说明|   
|:-|:-|:-|   
|1|exchange|交换器|   
|2|priorityqueue|优先级队列|   
|3|deadletterqueue|死信队列|   

## Spring Boot  RabbitMQ 参数配置详解
### 连接配置
    
    spring.rabbitmq.host=localhost                          # RabbitMQ 地址
    spring.rabbitmq.port=5672                               # RabbitMQ 端口
    spring.rabbitmq.username=guest                          # RabbitMQ 用户名
    spring.rabbitmq.password=guest                          # RabbitMQ 密码
    
    spring.rabbitmq.addresses=                              # 设置 RabbitMQ 集群，多个地址使用 "," 分隔，例如：192.168.0.100:5672,192.168.0.101:5672
    
    spring.rabbitmq.virtual-host=                           # 设置 Virtual Host
    
    spring.rabbitmq.ssl.algorithm=                          # SSL 算法，默认情况下，由 Rabbit 客户端配置
    spring.rabbitmq.ssl.enabled=false                       # 是否启用 SSL 支持
    spring.rabbitmq.ssl.key-store=                          # key 存储路径
    spring.rabbitmq.ssl.key-store-password=                 # 用于访问 key 的密码
    spring.rabbitmq.ssl.key-store-type=PKCS12               # Key 存储类型
    spring.rabbitmq.ssl.trust-store=                        # Trust 存储路径
    spring.rabbitmq.ssl.trust-store-password=               # 用于访问 Trust 的密码
    spring.rabbitmq.ssl.trust-store-type=JKS                # Trust 存储类型
    spring.rabbitmq.ssl.validate-server-certificate=true    # 是否启用服务端证书验证
    spring.rabbitmq.ssl.verify-hostname=true                # 是否启用 hostname 验证
### Publisher 配置

    spring.rabbitmq.publisher-confirms=false                # 是否启用 publisher 确认
    spring.rabbitmq.publisher-returns=false                 # 是否启用 publisher 返回
    
    spring.rabbitmq.template.default-receive-queue=         # 没有没确定指定队列时的默认队列
    spring.rabbitmq.template.exchange=                      # 发送消息默认的 exchange
    spring.rabbitmq.template.mandatory=                     # 是否启用 mandatory 消息
    spring.rabbitmq.template.receive-timeout=               # `receive()` 操作的超时时间
    spring.rabbitmq.template.reply-timeout=                 # `sendAndReceive()` 操作的超时时间
    spring.rabbitmq.template.retry.enabled=false            # 是否启用重试
    spring.rabbitmq.template.retry.initial-interval=1000ms  # 两次重试间的时间间隔
    spring.rabbitmq.template.retry.max-attempts=3           # 最大重试次数
    spring.rabbitmq.template.retry.max-interval=10000ms     # 最长重试时间
    spring.rabbitmq.template.retry.multiplier=1             # Multiplier to apply to the previous retry interval.
    spring.rabbitmq.template.routing-key=                   # 发送消息默认的 routing key
### Consumer 设置

    spring.rabbitmq.listener.direct.acknowledge-mode=           # 确认模式：auto / manual / none
    spring.rabbitmq.listener.direct.auto-startup=true           # 是否在应用启动时自动启动容器
    spring.rabbitmq.listener.direct.consumers-per-queue=        # 每个队列的消费者数量
    spring.rabbitmq.listener.direct.default-requeue-rejected=   # 默认情况下，拒收的消息是否重新排队
    spring.rabbitmq.listener.direct.idle-event-interval=        # 空闲容器事件发布的频率
    spring.rabbitmq.listener.direct.missing-queues-fatal=false  # 如果容器声明的队列在 broker 上不可用，是否失败
    spring.rabbitmq.listener.direct.prefetch=                   # 预加载的消息数量
    spring.rabbitmq.listener.direct.retry.enabled=false         # 是否启用发布重试
    spring.rabbitmq.listener.direct.retry.initial-interval=1000ms   # 两次重试时间间隔
    spring.rabbitmq.listener.direct.retry.max-attempts=3        # 最大重试次数
    spring.rabbitmq.listener.direct.retry.max-interval=10000ms  # 最长重试时间
    spring.rabbitmq.listener.direct.retry.multiplier=1          # 上次重试间隔的倍数
    spring.rabbitmq.listener.direct.retry.stateless=true        # 重试是否有状态
    
    spring.rabbitmq.listener.simple.acknowledge-mode=           # 确认模式：auto / manual / none
    spring.rabbitmq.listener.simple.auto-startup=true           # 是否在应用启动时自动启动容器
    spring.rabbitmq.listener.simple.concurrency=                # 监听器最小线程数
    spring.rabbitmq.listener.simple.default-requeue-rejected=   # 默认情况下，拒收的消息是否重新排队
    spring.rabbitmq.listener.simple.idle-event-interval=        # 空闲容器事件发布的频率
    spring.rabbitmq.listener.simple.max-concurrency=            # 监听器最大线程数
    spring.rabbitmq.listener.simple.missing-queues-fatal=true   # 如果容器声明的队列在 broker 上不可用，是否失败； 如果在运行时删除队列，容器是否停止
    spring.rabbitmq.listener.simple.prefetch=                   # 预加载的消息数量
    spring.rabbitmq.listener.simple.retry.enabled=false         # 是否启用发布重试
    spring.rabbitmq.listener.simple.retry.initial-interval=1000ms   # 两次重试时间间隔
    spring.rabbitmq.listener.simple.retry.max-attempts=3        # 最大重试次数
    spring.rabbitmq.listener.simple.retry.max-interval=10000ms  # 最长重试时间
    spring.rabbitmq.listener.simple.retry.multiplier=1          # 上次重试间隔的倍数
    spring.rabbitmq.listener.simple.retry.stateless=true        # 重试是否有状态
    spring.rabbitmq.listener.simple.transaction-size=           # 确认模式为 auto 时，在 acks 之间处理的消息数. 如果大于预加载的数量，则预加载的数量增加到此值

rabbitmq listener 类型有两种：simple 和 direct，二者有什么区别呢？ DirectMessageListenerContainer 注释如下：

 * The {@code SimpleMessageListenerContainer} is not so simple. Recent changes to the
 * rabbitmq java client has facilitated a much simpler listener container that invokes the
 * listener directly on the rabbit client consumer thread. There is no txSize property -
 * each message is acked (or nacked) individually.
### 其他设置

    spring.rabbitmq.dynamic=true                                # 是否创建 AmqpAdmin bean
    
    spring.rabbitmq.requested-heartbeat=                        # 请求心跳超时时间. 设置为 0 代表没有，如果未指定时间后缀，则默认使用秒
    
## Spring Boot RabbitMQ 队列属性详解

|属性名称|属性说明|
|:-|:-|
|Durable|代表该队列是否持久化至硬盘（若要使队列中消息不丢失，同时也需要将消息声明为持久化|
|Exclusive|是否声明该队列是否为连接独占，若为独占，连接关闭后队列即被删除|
|Auto-delete|若没有消费者订阅该队列，队列将被删除|
|Arguments|可选map类型参数，可以指定队列长度，消息生存时间，镜相设置等|


* RabbitMQ规定，队列的名字最长不超过UTF-8编码的255字节
* RabbitMQ内部的Queue命名规则采用 "amq."形式，注意不要与此规则冲突

### 常见问题

#### 1. 声明了一个已经存在的队列？
* 如果队列已经存在，再次声明将不会起作用。若原始队列参数和该次声明时不同则会报异常。

#### 2. 队列中消息顺序？
* 默认情况下是FIFO，即先进先出，同时也支持发送消息时指定消息的优先级。

#### 3. 队列消息存放位置？
* 对于临时消息，RabbitMQ尽量将其存放在内存，当出现内存告警时，MQ会将消息持久化至硬盘。对于持久化消息与Lazy-queues，MQ会先将消息存入硬盘，消费时再取出。

#### 4. 队列中消息的消费？
* 默认情况下，MQ会设置消费者的消费确认模式为自动。对于一些重要消息的处理，推荐确认模式改为手动。（nack和reject区别？nack可以一次拒绝多条消息）

#### 5. 队列中消息的消费速度？
* 通过Prefetch（通道上最大未确认投递数量）设置消费者每次消费的条数，一般将该值设为1，但他会降低吞吐量。RabbitMQ官网建议的是100-300.（更建议反复试验得到一个表现符合期望的值）

#### 6. 队列中消息状态？
* 队列中的消息共有俩种状态，一是准备投递，二是已投递但未确认。队列最大长度？ 声明队列时可以指定最大长度，需要注意的是只限制状态为准备投递的数量，未确认的消息不计算在内。当队列长度超过限制，MQ会根据策略选择丢弃（默认）或者将消息投递进死信队列。

#### 7. 关于死信队列？
* 其实更准确的说法是死信交换机，提前声明一个交换机，在声明队列时使用“x-dead-letter-exchange”参数（可指定routKey）将队列绑定到该死信交换机。消息有以下情况之一会成为死信：被reject或者nack，消息超过生存时间，队列长度超过限制。

#### 8. 关于不能路由到队列的消息？
* 这个和上面一样，其实不算Queue系列而是Exchange。针对消息无法路由到队列的情况MQ提供了Alternate Exchange处理。声明Exchange时添加args.put("alternate-exchange","my-ae")参数。即当该交换机存在无法路由的消息时，它将消息发布到AE上，AE把消息路由到绑定在他上面的消息。
