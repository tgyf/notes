# 负载均衡器 Ribbon

* Ribbon 做为负载均衡器首先会从注册中心获取可用的服务实例，然后会通过负载均衡机制为服务消费者选择调用哪一个服务实例，从而达到缓解网络压力和扩容的目的。同时也具备容灾的作用，不会应为莫一台实例故障而导致系统不可用。负载均衡策略常见的有轮询负载，权重负载，按流量负载，同时Ribbon也支持自定义负载策略。

## (一)负载策略
* RandomRule：随机策略会通过一个随机数从可用服务实例中选择一个实例。实现是用一个while循环，如果实例为空则执行随机选择，直到获取到。该策略有不严谨的地方，如果一直获取不到，很可能出现死循环的情况。
* RoundRobinRule：按照线性轮训的方式选择可用的实例。该策略定义了一个计数器的变量，每次循环后计数器累加    ，如果10次选择不到服务实例，则提示无可用服务。
* RetryRule：该策略增加了重试的机制，默认使用RoundRobinRule策略，如果超过时间阙值则返回null。
* WeightedResponseTimeRule：是在RoundRobinRule的基础上增加权重的策略，会根据实例的响应以及运行情况分配负载的权重，以便更好的利用资源，达到最优化。
* BestAvailableRule：遍历所有可用实例，过滤到故障实例，选择出请求并发数最小的一个。也就是说会优先选择空闲实例。

## (二)Spring Cloud Ribbon主要配置项
* 这里使用服务提供者的instanceName.ribbon.*

|配置参数({svc}.ribbon.*)|默认值|说明|
|:-|:-|:-|
|ConnectionTimeout              |1000ms|连接超时时间|
|ReadTimeout                    |1000ms|读取超时时间|
|ServerListRefreshInterval      |30秒|刷新服务列表源的间隔时间|
|NFLoadBalancerClassName        |com.netflix.loadbalancer.ZoneAwareLoadBalancer|定制ILoadBalancer实现|
|NFLoadBalancerRuleClassName    |com.netflix.loadbalancer.ZoneAvoidanceRule|定制IRule实现|
|NFLoadBalancerPingClassName    |com.netflix.loadbalancer.DummyPing|定制IPing|
|NIWSServerListClassName        |com.netflix.loadbalancer.ConfigurationBasedServerList|定制ServerList|
|ServerListUpdateClassName      |com.netflix.loadbalancer.PollingServerListUpdater|定制ServerListUpdater|
|NIWSServerListFilterClassName  |com.netflix.loadbalancer.ZonePreferenceServerListFilter|定制SeverListFilter|

```
    order-service-producer.ribbon.NFLoadBalancerRuleClassName=com.netflix.loadbalancer.RandomRule     # Ribbon使用的负载均衡策略
    order-service-producer.ribbon.MaxAutoRetries=1                                                    # 每台服务器最多重试次数，但是首次调用不包括在内
    order-service-producer.ribbon.MaxAutoRetriesNextServer=1                                          # 最多重试多少台服务器
    order-service-producer.ribbon.OkToRetryOnAllOperations=true                                       # 无论是请求超时或者socket read timeout都进行重试
    order-service-producer.ribbon.ServerListRefreshInterval=2000                                      # 从源刷新服务器列表的间隔
    order-service-producer.ribbon.ConnectTimeout=3000                                                 # 连接超时
    order-service-producer.ribbon.ReadTimeout=3000                                                    # 读超时

```