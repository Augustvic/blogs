# 1. RocketMQ 概览

## 1.1 用途：

应用解耦、异步调用、流量削峰、分布式最终一致性

## 1.2 概念：

**Topic&Tag：**

Topic是发布订阅的主题，Tag属于子Topic，主要作用是提供给业务更大的灵活度，用来分流消息。

**Producer&Consumer：**

Producer 是生产者，生产并发送消息。Consumer 是消费者，分为PushConsumer，和PullConsumer，PullConsumer由消费端自主从服务器拉取消息，PushConsumer由服务器推消息到客户端，前者更具优势（未确认）。

**ProducerGroup&ConsumerGroup：**

ProducerGroup是一类 Producer 的集合，这类 Producer 通常发送一类消息，且发送逻辑一致，由服务器自己维护，这个概念一般意义不大，但在发送分布式事务消息的时候，比较关键。ConsumerGroup是一类 Consumer 的集合，这类 Consumer 通常消费一类消息，且消费逻辑一致。多个消费者共同消费一个 Topic 下的消息，每个消费者消费部分消息。这些消费者就组成一个分组，拥有同一个分组名称，通常也称为消费者集群。

**Message：**

需要发送的消息。

**MessageQueue：**

可以认为 MessageQueue 是一个长度无限的数组，队列中的每个存储单元都是定长，访问其中的存储单元使用 Offset 来访问。

**NameServer：**

命名服务（类似 Zookeeper），用作注册主题和管理订阅关系，维护 Topic 和 Broker 的路由表，是几乎无状态的节点，可集群部署，节点之间无任何信息同步。

**Broker：**

存储、转发消息的地方，一个 Topic 对应一个顺序存储队列。Broker 分为 Master 与 Slave，一个 Master 可以对应多个 Slave，但是一个 Slave 只能对应一个 Master。

**广播消息：**

消息被多个 Consumer 消费，即使这些 Consumer 属于同一个 Consumer Group，也会被 Consumer Group 中的每个 Consumer 都消费一次

**普通顺序消息：**

正常情况下可以保证完全的顺序消息，但是一旦发生通信异常，Broker 重启，由于队列总数发生变化，哈希取模后定位的队列会变化，产生短暂的消息顺序不一致。 如果业务能容忍在集群异常情况（如某个 Broker 宕机或者重启）下，消息短暂的乱序，使用普通顺序方式比较合适。

**严格顺序消息：**

顺序消息的一种，无论正常异常情况都能保证顺序，但是牺牲了分布式 Failover 特性，即 Broker 集群中只要有一台机器不可用，则整个集群都不可用，服务可用性大大降低。 如果服务器部署为同步双写模式，此缺陷可通过备机自动切换为主避免，不过仍然会存在几分钟的服务不可用。（依赖同步双写，主备自动切换，自动切换功能目前还未实现） 目前已知的应用只有数据库 binlog 同步强依赖严格顺序消息，其他应用绝大部分都可以容忍短暂乱序，推荐使用普通的顺序消息。

**集群消息：**

一个 Consumer Group 中的 Consumer 实例平均分摊消费消息。

## 1.3 一次消息消费：

**消息生产：**

1. Producer 与 NameServer 服务集群中的任意一个节点建立长连接，每隔一段时间从该节点拉取 Topic 对应 Broker 的路由信息。
2. Producer 根据拉取到的路由表信息，与对应 Topic 的 Broker 的 Master节点建立长连接并保持心跳包。
3. 向 Broker 集群中对应 Broker 的 Master 节点发送 Message。（只写在Master中）

**消息消费：**

1. Consumer 与 NameServer 服务集群中的任意一个节点建立长连接，每隔一段时间从该节点拉取 Topic 对应 Broker 的路由信息。
2. Consumer 根据拉取到的路由表信息，与对应 Topic 的 Broker 的 Master 和 Slave 节点建立长连接并保持心跳包。
3. 既可以从 Master 订阅消息，也可以从 Slave 订阅消息。

## 1.4 架构设计：

<img src="https://github.com/Augustvic/Blogs/tree/master/RocketMQ/images/architecture.png" width=70% />

Producer 与 NameServer 集群中的一个节点（随机）建立长连接，定期从 NameServer 取 Topic 路由信息，并向提供 Topic 服务的 Master 建立长连接，且定时向 Master 发送心跳。Producer 发送消息是发送到 MasterBroker，再由 MasterBroker 同步到所有 Broker。

Consumer 与 Name Server 集群中的其中一个节点（随机选择）建立长连接，定期从 Name Server 取 Topic 路由信息，并向提供 Topic 服务的 Master、Slave 建立长连接，且定时向 Master、Slave 发送心跳。Consumer 既可以从 Master 订阅消息，也可以从 Slave 订阅消息，订阅规则由 Broker 配置决定。

## 1.5 核心组件：

**NameServer：**

NameServer 集群中的每个节点维护 Topic 和 Broker 的路由表，供 Producer 和 Consumer 拉取。

**Broker：**

Broker 分为 Master 和 Slave，一个 Master 可以对应多个 Slave，但是一个 Slave 只能对应一个 Master，Master 与 Slave的对应关系通过指定相同的 BrokerName，不同 BrokerId 来定义，BrokerId 为 0 表示 Master，非 0 表示 Slave。Master 可以部署多个。每个 Broker 与 NameServer 集群中的所有节点建立长连接，定时注册 Topic 信息到所有 NameServer。

## 1.6 数据存储：

所有发到 Broker 上的消息将会顺序存储在一个文件中，达到文件大小限制时默认创建新文件存储。把每个 Topic 分为多个逻辑队列，逻辑队列的内容为相对于物理存储文件中 message 的存储 offset，当 Consumer 根据订阅的 Topic 从 nameserver 获取 broker 地址建立连接后,根据逻辑队列中的 offset，从物理存储文件中拉取订阅的消息。

消息被消费后不会立即删除，每条消息都会持久化到 CommitLog 中，每个 Consumer 连接到 Broker 后会维持消费进度信息，当有消息消费后只是当前 Consumer 的消费进度（CommitLog 的 offset）更新了。

## 1.7 零拷贝：

**背景：**

数据从硬盘读取并通过 socket 发送出去，经过四次拷贝四次上下文切换（图见引用2）：执行读操作，检查缓存是否在内核缓冲区（切换），如果不在需要先将磁盘数据拷贝到内核缓冲区（拷贝），然后将内核缓冲区数据拷贝到用户缓存区（切换+拷贝），读操作结束；执行写操作时，先把数据从用户缓存区拷贝到内核 socket 缓冲区（切换+拷贝），然后把内核 socket 缓冲区拷贝到发送区（外部网络设备）（拷贝），完成后切换回用户态。频繁上下文切换占用 CPU 时间片，频繁拷贝操作增加系统负荷。

**技术分类：**

1. 应用程序直接访问硬件：针对操作系统内核不需要对数据处理的情况，数据可以在应用程序地址空间的缓冲区和磁盘之间直接传输，完全不需要 linux 操作系统内核提供的页缓存的支持。nio 中用allocateDirect 实现。

2. 数据传输不经过用户进程地址空间：避免数据在操作系统内核地址空间的缓冲区和用户应用程序地址空间的缓冲区之间进行拷贝，直接在用户态进行操作，不涉及到用户空间的转换。 中提供的系统调用主要有 mmap()，sendfile() 以及 splice()。

3. 写时复制：对数据在 linux 的页缓存和用户进程的缓冲区之间的传输过程进行优化。

**mmap：**

<img src="https://github.com/Augustvic/Blogs/tree/master/RocketMQ/images/mmap.png" width=70% />

java 中有 java.nio.channels.FileChannel.map 方法可以实现 mmap()；

改进：应用跟内核共享内核缓冲区，取消用户态和内核态之间的两次拷贝操作。

问题：依然要进行内核态和用户态切换；映射操作开销大，需要通过更改页表以及冲刷 TLB （使得 TLB 的内容无效）来维持存储的一致性；当对文件进行了内存映射，然后调用 write() 系统调用，如果此时其他的进程截断了这个文件，那么 write() 系统调用将会被总线错误信号 SIGBUS 中断（见引用1）。

**sendfile：**

<img src="https://github.com/Augustvic/Blogs/tree/master/RocketMQ/images/sendFile.png" width=70% />

java中对应java.nio.channels.FileChannel#transferTo();

改进：数据不经过用户态，直接从内核缓冲区进入到 socket 缓冲区。（看起来和mmap差不多）

继续改进：将带有文件位置和长度信息的缓冲区描述符添加到 socket 缓冲区，此过程不需要将数据从操作系统内核缓冲区拷贝到 socket 缓冲区中。类似传递了一个地址。

**两者区别：**

mmap 适合小数据量读写，sendFile 适合大文件传输；mmap 需要 4 次上下文切换，3 次数据拷贝；sendFile 需要 3 次上下文切换，最少 2 次数据拷贝；sendFile 可以利用 DMA 方式，减少 CPU 拷贝，mmap 则不能（必须从内核拷贝到 socket 缓冲区）。rocketmq 使用 mmap，kafka 使用 sendfile。

## 1.8 常见问题：

**消费消息是 push 还是 pull：**

RocketMQ没有真正意义的 push，都是 pull，虽然有 push 类，但实际底层实现采用的是长轮询机制，即拉取方式（broker端属性 longPollingEnable 标记是否开启长轮询，默认开启）。
如果broker主动推送消息的话有可能push速度快，消费速度慢的情况，那么就会造成消息在consumer端堆积过多，同时又不能被其他consumer消费的情况。而pull的方式可以根据当前自身情况来pull，不会造成过多的压力而造成瓶颈。所以采取了pull的方式。

**重复消费：**

影响消息正常发送和消费的重要原因是网络的不确定性，主要有两个具体原因：ACK，正常情况下在 consumer 真正消费完消息后应该发送 ack，通知 broker 该消息已正常消费，从 queue 中剔除，当 ack 因为网络原因无法发送到 broker，broker 会认为词条消息没有被消费，此后会开启消息重投机制把消息再次投递到 consumer；消费模式，在 CLUSTERING 模式下，消息在 broker 中会保证相同 group 的 consumer 消费一次，但是针对不同 group 的 consumer 会推送多次。

解决方案：

1. 数据库表，在处理消息前使用消息主键在表中带有约束的字段中 insert
2. 使用 map 去重，单机时可以使用map ConcurrentHashMap -> putIfAbsent
3. redis 分布式锁搞起来。

**顺序消费：**

多个 queue 只能保证单个 queue 里的顺序，queue 是典型的 FIFO 队列，多个queue同时消费无法保证消息的绝对有序性的。把消息发送到同一个queue李可以保证顺序消费。
rocketmq 提供了 MessageQueueSelector 接口，可以自己重写里面的接口，实现自己的算法，详见引用。

**消息丢失：**

1. producer 端：调用 send 同步发消息；发送失败后重试，重试次数默认为 3 次；集群部署，失败了的原因可能是当前 broker 宕机了，重试的时候会发送到其他 broker 上。
2. broker 端：默认情况下是异步刷盘，修改刷盘策略为同步刷盘；集群部署，主从模式，高可用。
3. consumer 端：完全消费正常后再手动 ack 确认。


## 1.10 引用：
1. [什么是零拷贝？mmap与sendFile的区别是什么？](https://blog.csdn.net/weixin_37782390/article/details/103833306)
2. [linux零拷贝相关(内网)](https://topic.atatech.org/articles/174520#9)
3. [RocketMQ在面试中那些常见问题及答案+汇总](https://www.cnblogs.com/SmallStrange/p/13343586.html)