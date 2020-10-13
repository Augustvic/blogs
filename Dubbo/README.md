# Dubbo

> Dubbo 源码是 github master(2020.4) 版本，Netty 源码是 4.0 版本。

## 源码贡献

* [源码贡献](https://github.com/Augustvic/Blogs/tree/master/Dubbo/md/dubbo/15.md)

    - [#5860] EchoService.class will lose if the 'types' not null
    - [#5875] correct the number of selected invoker
    - [#6049] NettyChannel can be added to CHANNEL_MAP cache
    - [#6064] add new loadbalance strategy
    - [#6102] fix concurrent bug (avoid NullPointerException)
    - [#6164] reduce lock granularity, avoid null
    - [#6206] enhance LRUCache

## 框架与扩展点概述

* [框架](https://github.com/Augustvic/Blogs/tree/master/Dubbo/md/dubbo/1.md)

* [SPI](https://github.com/Augustvic/Blogs/tree/master/Dubbo/md/dubbo/2.md)

    - Java SPI
    - Dubbo SPI
    - 自适应扩展
    - IOC 和 AOP

## 一次远程服务调用的实现过程

* [服务暴露](https://github.com/Augustvic/Blogs/tree/master/Dubbo/md/dubbo/3.md)

* [服务引入](https://github.com/Augustvic/Blogs/tree/master/Dubbo/md/dubbo/4.md)

* [目录和路由](https://github.com/Augustvic/Blogs/tree/master/Dubbo/md/dubbo/5.md)

    - RegistryDirectory
    - ConditionRouter

* [集群容错](https://github.com/Augustvic/Blogs/tree/master/Dubbo/md/dubbo/7.md)

    - failover
    - failback
    - failfast
    - failsafe
    - forking
    - broadcast
    - mergeable

* [负载均衡](https://github.com/Augustvic/Blogs/tree/master/Dubbo/md/dubbo/8.md)

    - RandomLoadBalance
    - LeastActiveLoadBalance
    - ConsitentHashLoadBalance
    - RoundRobinLoadBalance

* [服务调用过程](https://github.com/Augustvic/Blogs/tree/master/Dubbo/md/dubbo/9.md)

## Netty 相关组件及其应用

* [EventLoop 线程模型](https://github.com/Augustvic/Blogs/tree/master/Dubbo/md/dubbo/10.md)

* [Channel、ChannelPipeline 和 ChannelHandler](https://github.com/Augustvic/Blogs/tree/master/Dubbo/md/dubbo/11.md)

    - Channel
    - ChannelPipeline
    - ChannelHandler

* [Dubbo 协议与通信过程](https://github.com/Augustvic/Blogs/tree/master/Dubbo/md/dubbo/12.md)

## 服务调用扩展

* [过滤器链](https://github.com/Augustvic/Blogs/tree/master/Dubbo/md/dubbo/13.md)

    - ProtocolFilterWrapper
    - ActiveLimitFilter
    - ExecuteLimitFilter

* [异步调用和线程模型](https://github.com/Augustvic/Blogs/tree/master/Dubbo/md/dubbo/14.md)

## 公共模块

* [线程池和定时器](https://github.com/Augustvic/Blogs/tree/master/Dubbo/md/dubbo/16.md)