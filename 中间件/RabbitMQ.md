---
title: RabbitMQ笔记
date: 2019/08/20 18:57
categories:
- 中间件
- RabbitMQ
tags:
- RabbitMQ
- 消息中间件
---

## RabbitMQ

RabbitMQ 是一个由 Erlang 语言开发的 AMQP 的开源实现.

AMQP: Advanved Message Queue, 高级消息队列协议. 它是应用层协议的一个开放标准, 为面向消息的中间件设计, 基于此协议的客户端与消息中间件可传递消息, 且不受产品, 开发语言条件的限制.

RabbitMQ 的特点有: 

1. 可靠性(Reliability): 使用持久化, 传输确认, 发布确认等机制来保证可靠性.
2. 灵活的路由(Flexible Routing): 在消息进入队列之前, 通过 Exchange 来路由消息的. 对于典型的路由功能, RabbitMQ 已经提供了一些内置的 Exchange 来实现. 针对更复杂的路由功能, 可以将多个 Exchange 绑定在一起, 也可以通过插件机制实现自己的 Exchange.
3. 消息集群(Clustering): 多个 RabbitMQ 服务器可以组成一个集群, 形成一个逻辑 Broker.
4. 高可用(Highly Available Queues): 队列可以在集群的机器上创建镜像节点, 使得在部分节点出问题的情况下队列仍然可用.
5. 多种协议(Multi-protocol): RabbitMQ 支持多种消息队列协议, 如 STOMP, MQTT 等.
6. 多语言客户端(Many Clients): RabbitMQ 几乎支持所有语言.
7. 管理界面(Management UI): RabbitMQ 提供了一个易用的用户界面, 使得用户可以监控和管理消息 Broker 的诸多方面.
8. 跟踪机制(Tracing): 如果消息异常, RabbitMQ 提供了消息跟踪机制.
9. 插件机制(Plugin System): RabbitMQ 提供了许多插件, 来从许多方面进行扩展, 也可以编写自己的插件.

### 系统架构

#### 消息模型

所有 MQ 产品从模型抽象上来说都是相似的过程: 消费者(Consumer)订阅某个队列(Queue), 生产者(Producer)创建消息, 然后发布到队列(Queue)中, Server(Broker)负责将队列中的消息发送给订阅该队列的消费者.

![UTOOLS1566300421073.png](https://i.loli.net/2019/08/20/4Xjiz65mNDEV3Id.png)

![UTOOLS1566300989086.png](https://i.loli.net/2019/08/20/jYEGq6SNUk4TIJO.png)

- Server

  也叫 Broker, 它的角色就是维护从 Producer 到 Comsumer 的路线, 保证数据能够按照指定的方式进行传输. 

- Publisher

  数据发送方, 负责发送 Message 到 Server. 

- Consumer

  数据接收方, 负责从 Server 中订阅的 Queue 获取 Message.

- Message

  消息, 一个 Message 由两个部分组成: Header 和 Body, Header 是由生产者添加的各种属性的集合, 包括 routing-key(路由键), priority(优先级), delivery-mode(消息是否需要持久化)等.

- Exchange

  交换器, 根据消息的 routing-key 来将其路由给服务器中的队列.

- Queue

  消息队列, 用来保存消息, 直到消息过期或被消费者获取. 它是消息的容器, 也是消息的终点. 一个消息可路由到一个或多个队列.

- Binding

  绑定, 用于定义消息队列和交换器之间的关联. 一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则.

- Connection

  网络连接.

- Channel

  信道, 多路复用连接中的一条独立的双向数据流通道. 信道建立在真实的 TCP 连接内的虚拟连接, AMQP 命令不管是发布消息, 订阅队列还是接收消息, 都是通过信道发出. 因为对于操作系统来说, 建立和销毁 TCP 都是非常昂贵的开销, 所以引入了信道的概念, 以复用一条 TCP 连接. 

- Virtual Host

  虚拟主机, 每个 vhost 都拥有自己的队列, 交换器, 绑定和权限机制.

#### 消息路由

AMQP 中的消息的路由和 JMS 存在一些差别, AMQP 中增加了 Exchange 和 Binding 的角色. 生产者把消息发布到 Exchange 上, 消息最终到达队列并被消费者接收, 而 Binding 决定交换器的消息该发送到哪个队列.

![UTOOLS1566302772699.png](https://i.loli.net/2019/08/20/pvGkXbzIquJaR1j.png)

##### Exchange 类型

Exchange 分发消息是根据类型的不同, 采用不同的分发策略. 目前共有四种类型: 

1. direct

   ![UTOOLS1566303295921.png](https://i.loli.net/2019/08/20/yaCMmbfz8cVuYK4.png)

   消息中的路由键(routing-key)如果和 Binding 中的 binding key 完全匹配才将消息发到对应的队列中. 

2. fanout

   ![UTOOLS1566303388742.png](https://i.loli.net/2019/08/20/ecZh3DyKAJaEoPR.png)

   每个发到 fanout 类型交换器的消息都会被分发到所有绑定的队列中区. fanout 交换器不处理路由键, 所以效率是最高的.

3. topic

   ![UTOOLS1566303443271.png](https://i.loli.net/2019/08/20/A4u3aox9vwEYWgz.png)

   topic 交换器通过模式匹配消息的路由键来路由消息, 它将路由键和绑定键的字符串用 `.` 分隔为多个单词, 同样也会识别两个通配符: `#` 匹配 0 个或多个单词, `*` 仅匹配一个单词.

4. headers

   headers 交换器不依赖 routing key 与 binding key 的匹配规则来路由消息, 而是根据发送的消息内容中的 headers 属性进行匹配. 通过判断 headers 中的键值对与交换器与队列绑定时指定的键值对是否匹配来路由消息.

#### 任务分配机制

##### Round-robin 转发

RabbitMQ 会将消息均衡地分发给每个消费者, 平均每个消费者将会获得相等数量的消息.

##### 消息应答(Message Acknowledgement)

消费者在处理完消息后告诉 MQ 消息已经处理完成, 之后 MQ 才将消息从队列移除.

如果消费者因为某些原因断开连接而没有发送应答, RabbitMQ 会认为该信息没有被完全地处理, 然后将会重新转发给别的消费者. 这里不存在 timeout, 即除非当前消费者断开连接, 否则消息不会被重新发送给其他消费者.

另外, publish message 是没有 ack 的.

##### 消息持久化(Message durability)

将消息持久化到硬盘, 下次 MQ 重启后不会丢失消息.

##### 公平转发(Fair dispatch)

如果有多个消费者同时订阅同一个 Queue 中的消息, Queue 中的消息会被平摊给多个消费者. 这时, 如果每个消息的处理时间不同, 就可能导致某些消费者一直忙碌, 而另一些消费者处于空闲状态. 我们可以通过设置 Prefetch count 来限制 Queue 每次发送给每个消费者的消息数, 比如 prefetchCount = 1, 则每个消费者每次只能接收到一条消息, 消费者只有在处理完(Ack)这条消息后才能再次获得一条新的消息.

##### RPC

MQ 本身是基于异步的消息处理, 生产者将消息发送到 RabbitMQ 后不会知道消费者是否处理成功. 如果生产者需要根据消息被处理的情况进行下一步处理, 这相当于 RPC(Remote Procedure Call, 远程过程调用). 在 RabbitMQ 中也支持 RPC. 实现的机制是: 

![UTOOLS1566306049106.png](https://i.loli.net/2019/08/20/v6MtaxlRskCychb.png)

客户端发送消息时, 在消息的属性(Message Properties, 在 AMQP 协议中定义了 14 种 properties, 这些属性会随着消息一起发送)中设置两个值 replyTo(一个 Queue 名称, 用于告诉服务器处理完成后将通知我的消息发送到这个 Queue 中)和 correlationId(此次请求的标识号, 服务器处理完成后需要将此属性返还, 客户端将根据这个 id 了解到辣条请求被成功执行了或执行失败). 服务端收到消息处理完后, 将生成一条应答消息到 replyTo 指定的 Queue, 同时带上 correlationId 属性. 客户端之前已订阅 replyTo 指定的 Queue, 从中收到服务器的应答消息后, 根据其中的 correlationId 属性分析哪条请求被执行了, 根据执行结果进行后续业务处理.

